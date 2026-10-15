---
name: phygital-mcp
description: How an agent works with the Phygital+ portal MCP server (/api/mcp) - pick a model, build its input from the JSON Schema, get a file into storage (yours to read, or the person's to hand over), start a generation, wait for the result, and handle a job that failed. Use on any call to list_models, get_model_schema, generate, get_job, cancel_job, upload_file, start_upload, upload_chunk, finish_upload, request_upload, claim_upload, get_upload_url.
---

# Running Phygital+ generations over MCP

## What this server is

Twelve tools on `POST /api/mcp` (Streamable HTTP). The server keeps no history: no job
list, no "list my generations". Everything you start exists only under the `job_id`
returned by `generate` — **lose it and the job cannot be reached over MCP again**.

The one thing it does hold between calls is a chunked upload in progress: `start_upload`
opens an assembly that lives in the server's memory, belongs to your key, and is dropped
after its idle timeout. Finish it in the same run of calls; do not park a `session_id`
and come back to it later.

`generate` spends the user's credits. Run only what you were asked for, one job at a
time, and never "just to see what happens".

## The order of operations (do not shortcut)

1. `list_models` — find the model, take its `id`.
2. `get_model_schema` — get the `inputSchema` and `capabilities`
   **of that exact model**.
3. If the input includes a file: get it into storage first — see **Files**, below.
4. `generate` — get the `job_id`.
5. `get_job` — poll until a terminal status.

Step 2 is never skipped. Parameter names and allowed values come from the live node
definition in the main backend: they differ between models and change over time. Input
assembled from memory, or by analogy with another model, is a guaranteed validation
error — not a saved call.

## Tools

| Tool | Arguments | Returns | Token |
|---|---|---|---|
| `list_models` | `category?`: `image` \| `video` \| `audio` \| `text` \| `3d` \| `other` | `{models: [{id, id_global, name, category, description, deprecated}]}` | `tasks:read` |
| `get_model_schema` | `model`: slug or `id_global` | `{model, id_global, name, category, description, deprecated, averageTimeInSeconds, inputSchema, capabilities: {rules}, outputs}` | `tasks:read` |
| `generate` | `model`, `input` (flat object) | `{job_id, status: "queued", model}` | `tasks:write` |
| `get_job` | `jobId` | `{job_id, status, position, error?, result?, outputs?}` plus `resource_link` blocks for media | `tasks:read` |
| `cancel_job` | `jobId` | `{job_id, canceled: true}` | `tasks:write` |
| `get_upload_url` | — | `{upload_url, method, content_type, file_field, auth, result_field}` | `files:write` |
| `upload_file` | `filename`, `mime_type`, `data_base64` | `{storage_object_id, filename, mime_type, bytes}` | `files:write` |
| `start_upload` | `filename`, `mime_type`, `total_bytes?` | `{session_id, chunk_max_bytes, max_total_bytes, expires_in_seconds, next_step}` | `files:write` |
| `upload_chunk` | `session_id`, `index`, `data_base64` | `{session_id, status, received_chunks, received_bytes, next_index, next_step}` | `files:write` |
| `finish_upload` | `session_id` | `{storage_object_id, filename, mime_type, bytes}` | `files:write` |
| `request_upload` | — | `{status, upload_url, ticket, expires_in_seconds, max_bytes, next_step}` | `files:write` |
| `claim_upload` | `code` | `{storage_object_id, filename, mime_type, bytes}` | `files:write` |

Every tool, the catalog included, needs the user's token: `list_models` and
`get_model_schema` ask for the same `tasks:read` as `get_job`. Without a token nothing
answers, so an authorization error on the very first `list_models` means the client is
not signed in yet — not that the catalog is empty.

Not every portal model is in the MCP catalog: some are REST-only. If a model is not in
`list_models`, passing its slug directly will not run it either.

## Building `input`

`input` is **one flat object**: its keys are the `properties` of `inputSchema`. The
portal splits them into inputs and params itself; `x-phygital-group` in the schema is
only there in case you want to show that split to the user.

Rules enforced before anything reaches the backend:

- **No extra keys.** A name that is not in the schema → `unknown field for this model`.
- **`required`** lists the fields that have no default. Everything else can be omitted —
  the portal fills in the node's defaults. Do not restate a default "to be safe".
- **`null` means "not supplied".** The field falls back to its default; if it has no
  default and is required, you get `is required`.
- **`enum`** — strictly one of the listed values. These are technical codes
  (`sec_5`, `r_16_9`, `kling_v2_master`), not what a human is shown; never translate
  them into "5 seconds" and back.
- **Numbers** — respect `minimum` / `maximum` / `multipleOf` and the type
  (`integer` vs `number`).
- **File fields** (`anyOf: [integer, string/uri]`) accept either the
  `storage_object_id` of an uploaded file or a public `http(s)` URL. A local path,
  `file://`, or `data:` is not accepted.
- **Arrays** — respect `maxItems`; each item of a file list follows the rule above.
- File fields deliberately ship no `default`: the node definition holds someone else's
  sample file there. Do not substitute anything of your own.

### Dynamic capabilities

`get_model_schema` returns dynamic `capabilities.rules` alongside the static
`inputSchema`. Evaluate the rules before building `input`; `generate` validates the
same contract before spending credits.

A rule applies when every condition in `when` matches. Its effects may:

- disable or require a field;
- restrict `allowedValues` (active restrictions are intersected);
- provide a `defaultValue` when the field is omitted;
- set a `forcedValue`, which should be omitted from `input`.

For file conditions, `connected`/`any_connected` means a non-empty file value was
supplied; `not_connected`/`none_connected` means it was not. If `rules` is empty,
follow `inputSchema` alone. Only submit fields present in `inputSchema`; do not invent
product-UI preset selectors.

## Files

A file input takes a public `http(s)` URL or the `storage_object_id` of a file already in
the person's storage. Which tool puts it there depends on **who actually holds the file**.

### The file goes up as it is

**Never re-encode, resize, crop, or re-compress a file to make it fit a limit.** Not a
JPEG quality pass, not a scale-down, not a convert "just to be safe". The model generates
from what you send, the loss is silent and permanent, and the person sees only a worse
result they will blame on the model. If you already have a file that someone else
shrank, say so rather than passing it off as the original.

Size picks the channel, never the quality:

| Size | What to do |
|---|---|
| small enough for one call, bytes in hand | `upload_file` — base64 in a single call |
| too big for that, bytes still in hand | `start_upload` → `upload_chunk` → `finish_upload` |
| you can make an HTTP request yourself | `get_upload_url`, then POST the file |
| the file is the person's, not yours | `request_upload` — the upload panel |
| bigger than every route here | ask for a public `http(s)` URL and pass that straight into the input |

Do not carry these thresholds in your head: the server states them. `[FILE_TOO_LARGE]`
names the limit it just hit, `start_upload` answers with `chunk_max_bytes` and
`max_total_bytes`, `get_upload_url` and `request_upload` with `max_bytes`. A file that
does not fit the route you took moves to the next route down this table — it does not get
smaller.

### You hold it

A path you can read, bytes you produced yourself:

```
upload_file(filename, mime_type, data_base64)  →  {storage_object_id}
```

Small files only — the base64 passes through your context and is a third larger than the
file. Above the server's inline limit you get `[FILE_TOO_LARGE; retryable=false]`, and
there are two ways on.

**In pieces, over MCP.** Same bytes, same storage, just split across calls:

```
start_upload(filename, mime_type, total_bytes?)  →  {session_id, chunk_max_bytes}
upload_chunk(session_id, index: 0, data_base64)  →  {next_index: 1, ...}
upload_chunk(session_id, index: 1, data_base64)  →  {next_index: 2, ...}
finish_upload(session_id)                        →  {storage_object_id}
```

Cut the **file** into pieces of at most `chunk_max_bytes` and base64 each piece on its
own. Slicing the base64 text of the whole file instead produces a broken file and nothing
will say so. Indexes run from `0` with no gap; sending the same index again replaces that
piece, so a chunk that failed is simply repeated. Nothing is stored until `finish_upload`
returns the `storage_object_id`, and the assembly is dropped after
`expires_in_seconds` without a new piece.

Errors say what to do next: `[INCOMPLETE_UPLOAD]` — a piece is missing, send it and
finish again; `[FILE_TOO_LARGE]` — cut smaller, or take the route below;
`[TOO_MANY_UPLOADS]` — finish or drop an assembly you already have open;
`[UNKNOWN_UPLOAD_SESSION]` — the assembly is gone, start over with `start_upload`;
`[UPLOAD_IN_FLIGHT]` — a `finish_upload` for this session is still running, wait for its
answer instead of calling again. A `finish_upload` that fails on the storage side keeps
the pieces: call it again with the same `session_id` rather than re-sending the file.

**Or outside MCP**, which does not spend your context at all — use `get_upload_url` and
POST the file yourself:

```
get_upload_url  →  {upload_url, method: POST, content_type: multipart/form-data,
                    file_field: "file", ticket, expires_in_seconds, max_bytes,
                    result_field: "storage_object_id"}
```

The `ticket` authorizes the upload on its own, so no header and no key of yours are
needed:

```
curl -F file=@<path> -F ticket=<ticket> <upload_url>   →  {"storage_object_id": …}
```

It is good for `expires_in_seconds` (minutes, not hours) and works only on `upload_url`;
fetch a fresh one when it runs out. Do not print it in your answer. On a portal with no
pass to give, `ticket` is null and `auth` says to send the same `Authorization` header you
use for MCP — that route needs a key you can actually read.

### The person holds it

A picture they dropped into the chat, something on their desk, anything you were only
told about. **You cannot forward that file.** A chat attachment reaches you as pixels in
your context, not as bytes you can pass on, and base64 you write from looking at an image
is a broken file, not a copy of it. Do not try.

```
request_upload()  →  {status: "waiting_for_the_person", upload_url, ticket, max_bytes}
```

This opens an upload panel in the conversation: the person picks the file there and it
goes straight into their storage. Say the panel is open and wait — nothing is uploaded
yet, and `generate` has nothing to work with until it is.

Then wait. Where the client allows it, the panel speaks for itself: the
`storage_object_id` arrives as a message from the person, and there is nothing left to
call — use the number as the file input. Where it does not, the panel shows them a short
code instead and they send that:

```
claim_upload(code)  →  {storage_object_id, filename, mime_type, bytes}
```

Never assume the file arrived because the panel looks busy or the person says they picked
one. Until one of those two things happens, you have no id — and an id you guessed is
someone else's file.

`ticket` is not for you: it is the panel's one-purpose pass. Never print it, never put it
in a link, never send it anywhere except `upload_url`.

### Already public

If the file has a public `http(s)` URL, upload nothing — pass the URL straight into the
input.

## Waiting for the result

`generate` returns immediately — the job is asynchronous. `get_job` statuses:

| Status | Meaning | What to do |
|---|---|---|
| `queued` | in the queue; `position` is the place in it | wait |
| `running` | being generated | wait |
| `completed` | done; `result` = `{output_name: [links] or a plain value}` | pass links on as they are; a text value is the answer itself — give it to the user |
| `failed` | failed; `error.code` + `error.message` | see below |
| `canceled` | canceled | report it |

`averageTimeInSeconds` from the schema sets the pace: first `get_job` after roughly half
that time, then every 5–10 seconds. Polling more often than once every 3 seconds is
pointless — neither the queue nor the generation moves that fast.

Budget roughly 3–5 × `averageTimeInSeconds` for waiting. Past that, do not stay silent:
tell the user where the job stands (`status`, `position`) and ask whether to keep waiting
or cancel. Never lose the `job_id` — it still reaches the job in a later session.

The keys in `result` are the `outputs[].name` values from the schema, so you know them
in advance.

## Result media

When a job finishes with a media output, `get_job` returns three things at once:

- **the JSON** — `result` as before plus `outputs`:
  `[{name, type, url, mime, expires_at}]`, one entry per file;
- **`resource_link` blocks** — one per file, with a `mimeType` and the address. A client
  that can show attachments renders them on its own;
- **a text hint** carrying a ready-made markdown line for images.

On top of that the portal ships a result viewer app (the MCP Apps extension): a client
that negotiated it shows the image with an `img` tag and video or audio in a player, right
in the chat. There you need to do nothing — the result is already visible. The tool result
cannot tell you whether that happened, so behave as if there is no app.

Files never pass through the portal in any of these paths: the link is signed by the
backend and the client fetches it directly. So you have no bytes — you have an address and
a type.

**How to show an image in the chat:** put it into your own reply as markdown —
`![name](url)`. The ready-made line is already in the hint from `get_job`, so copying it
is enough. What renders the image is your message, not the tool result.

**Never claim you displayed a file when you did not.** Not "as a card", not "as a widget",
not "the image above": if all you passed was a link, a link is what the user sees. Do not
explain what they are looking at with rendering that does not exist.

Video, audio, and 3D cannot be embedded as markdown — for those, the `resource_link` and a
plain link are all there is. Say exactly that: it can be viewed at the link.

The links are signed and time-limited — the exact moment is in `expires_at` (in the URL
itself it is `Expires=<unix-time>`, about a week on the dev stand). If the file is needed
for anything lasting, download it now rather than "later, from this link".

## Tool errors

They arrive as text. Read the text rather than reacting to the mere fact of an error:
half of them are not retryable, and a blind retry either repeats the same error or
spends the credits twice.

| Text | Cause | Action |
|---|---|---|
| `Unknown model: X. Call list_models…` | slug/id is not in the MCP catalog | call `list_models`, take the `id` from there |
| `Model is not enabled for MCP: X` | the model is REST-only | do not retry, tell the user |
| `Input does not match the model schema — field: message; …` | input failed the portal's validation | fix **the listed** fields against `inputSchema` and active `capabilities`; no credits were spent — the request never reached the backend |
| `[MODEL_UNAVAILABLE; retryable=false]` | the model is temporarily unavailable through MCP | do not retry; use another model or report that this one is currently unavailable |
| `This tool runs on your own account: send your Phygital+ API key…` | the client sent no token | ask the user to authorize (OAuth login) or put a key in the client config; until then no tool answers, the catalog included |
| `Your authorization does not cover this action. Missing scope: …` | there is a token, but not that scope | re-authorize with the required scope; retrying with the same token is pointless |
| `Job id must be a number: X` | the `job_id` got mangled | pass the value from the `generate` response verbatim |
| `Phygital+ is unreachable: …` | network or timeout to the backend | `list_models`, `get_model_schema`, `get_job` — retry with backoff. `generate` — **do not retry blindly**: the job may have been created, and a second start spends credits again. MCP has no job list — ask the user whether they can see the job in the UI |

Backend errors (out of credits, revoked key, rate limit, 5xx) are passed through with the
backend's own text. Read the message and relay what it means: "out of credits" and "no
access to this model" are fixed by the user, not by calling again.

## When a job fails (`status: "failed"`)

The response carries `error.code` — the backend's raw status (it starts with `error`) —
and `error.message`, the reason.

1. **Read `message`.** If the cause is the input (format, size, content, unsupported
   value), it is not a transient glitch: fix the input against the schema.
2. **Retry only for a clearly transient cause** (timeout, provider-side internal error),
   and **only once**. A repeat `generate` is a new job and a new charge, so it is better
   to ask the user first.
3. **Two failures in a row for the same reason — stop.** Show the user `error.code`,
   `error.message`, and the `job_id`; the call is theirs from there.
4. Do not invent a cause when `message` is empty: say exactly that — the job failed
   without an explanation, and here is its `job_id`.

Cancel a job stuck in `queued`/`running` with `cancel_job` — the credits spent on it are
returned. `cancel_job` on an already finished job is pointless.

## What not to do

- Call `generate` unprompted, in batches, or "to see what comes out".
- Repeat `generate` after an error without changing the input.
- Invent parameter names, enum values, or a `storage_object_id`.
- Carry one model's schema over to another, including a neighbouring version.
- Wait past a sensible budget in silence instead of reporting back to the user.

## Connecting (for reference)

```json
{
  "mcpServers": {
    "phygital": { "type": "http", "url": "https://dev.phygital.plus/api/mcp" }
  }
}
```

When OAuth is enabled on the portal, the client opens a browser itself: the user signs in
with their normal Phygital+ login and approves access — no key to obtain anywhere. Where
there is no browser redirect (CI, server-side integrations), a personal API key in the
`Authorization: Bearer <key>` header works.
