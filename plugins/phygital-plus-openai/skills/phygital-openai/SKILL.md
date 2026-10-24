---
name: phygital-openai
description: How to run Phygital+ generation models from ChatGPT through the /api/mcp/openai server - one tool per model (generate_image_*, generate_video_*, upscale_*, generate_music_*, ...), each with its own input schema; get a file into the person's storage, start a job, wait for it with get_job and show the result. Use on any call to a Phygital+ model tool, get_job, cancel_job, upload_file, start_upload, upload_chunk, finish_upload, request_upload, claim_upload, get_upload_url.
---

# Running Phygital+ models from ChatGPT

## What this server is

`POST /api/mcp/openai` (Streamable HTTP). Every model is its own tool: there is no
catalog to browse and no schema to fetch. The tool's name says what it does, its input
schema is that model's own, and its description lists the rules that depend on other
fields.

| Tool family | Examples |
|---|---|
| images | `generate_image_gpt`, `generate_image_flux`, `generate_image_nano_banana`, `generate_image_seedream`, `generate_vector_image`, `vectorize_image`, `outpaint_image`, `remove_image_background` |
| upscaling | `upscale_image`, `upscale_image_magnific`, `upscale_image_topaz`, `upscale_video` |
| video | `generate_video_kling`, `generate_video_veo`, `generate_video_seedance`, `generate_video_runway`, `edit_video_runway_aleph`, `animate_character`, `remove_video_background` |
| audio | `generate_music_suno`, `generate_music_lyria`, `generate_speech`, `generate_sound`, `generate_dialogue`, `design_voice`, `clone_voice` |
| text | `generate_text_gpt`, `generate_text_claude`, `generate_text_gemini`, `describe_image`, `generate_prompt`, `extend_prompt` |
| other | `generate_3d_model`, `generate_presentation`, `create_pipeline` |
| shared | `get_job`, `cancel_job`, `upload_file`, `start_upload`, `upload_chunk`, `finish_upload`, `get_upload_url`, `request_upload`, `claim_upload` |

The server keeps no history: no job list, no "my generations". A job exists for you only
under the `job_id` a model tool returned. **Lose it and the job cannot be reached again.**

Every model tool runs on the person's Phygital+ account and **spends their credits**. Run
only what you were asked for, one job at a time, and never "just to see what comes out".

## The order of operations

1. Pick the tool whose name matches the request. When several models can do the job and
   the person did not name one, pick the one the description fits best, or ask.
2. If the input includes a file, put it into storage first (see **Files**).
3. Call the model tool and keep the `job_id`.
4. Wait for the result (see **Waiting for the result**), then show it.

## Building the arguments

The arguments are one flat object: the `properties` of the tool's input schema.

- **No extra keys.** The schema has `additionalProperties: false`; an unknown name is
  rejected.
- **`required`** lists the fields without a default. Everything else can be left out, and
  the server fills in the model's defaults. Do not restate a default "to be safe".
- **`enum`** values are technical codes (`sec_5`, `r_16_9`, `kling_v3`). Pass them as they
  are; do not translate them into "5 seconds" and back.
- **Numbers**: respect `minimum`, `maximum`, `multipleOf`, and `integer` vs `number`.
- **File fields** accept either the `storage_object_id` of an uploaded file or a public
  `http(s)` URL. A local path, `file://`, or `data:` is not accepted.
- **Arrays**: respect `maxItems`. Each item of a file list follows the rule above.

### Rules in the description

Some fields depend on others: a parameter that only applies when a start image is given,
a model version that allows fewer durations, and so on. The tool description spells them
out as `- if <condition>: <effect>` lines:

- `not available: a, b` means leave those fields out;
- `x is required` means `x` must be given;
- `x in [...]` means only those values are accepted;
- `x must be <value>` means the server sets it, so leave it out.

The server checks the same rules before anything is charged. A call that breaks them
comes back as `[INVALID_INPUT; retryable=false]` naming the field, and no credits are spent.

## Files

A file input takes a public `http(s)` URL or the `storage_object_id` of a file already in
the person's storage. Which tool puts it there depends on **who holds the file**.

**Never re-encode, resize, crop, or re-compress a file to make it fit a limit.** The size
picks the channel, never the quality:

| Situation | What to do |
|---|---|
| the file has a public `http(s)` URL | upload nothing, pass the URL |
| the person attached it to the chat, or only told you about it | `request_upload`: the upload panel |
| you hold the bytes and they fit one call | `upload_file` (base64) |
| you hold the bytes and they do not fit | `start_upload`, then `upload_chunk` for each piece, then `finish_upload` |
| you can make an HTTP request yourself | `get_upload_url`, then POST the file |

The limits come from the server. `[FILE_TOO_LARGE]` names the limit it just hit,
`start_upload` answers with `chunk_max_bytes`, and `get_upload_url` and `request_upload`
answer with `max_bytes`.

**A chat attachment is not yours to forward.** It reaches you as pixels, not bytes, and
base64 written from looking at an image is a broken file. Use `request_upload`, tell the
person the panel is open, and wait. The `storage_object_id` arrives as a message from
them. Where the client does not let the panel speak, they send a short code instead, and
`claim_upload(code)` turns it into the id. Until one of the two happens nothing has been
uploaded, so never guess an id.

For chunks, cut the **file** into pieces of at most `chunk_max_bytes` and base64 each
piece on its own. Indexes start at `0` with no gaps. Sending an index again replaces that
piece. Nothing is stored until `finish_upload` returns the id.

Never print an upload `ticket` and never send it anywhere except its `upload_url`.

## Waiting for the result

A model tool returns at once with `{job_id, status: "queued", ...}`.

- **The generation panel is shown** (ChatGPT and other clients with MCP Apps): the panel
  follows the job by itself, shows the queue and progress, and posts **one** message
  when the job is done, with the result or the error. Do not call `get_job` meanwhile;
  wait for that message and answer it with the result. Tell the person in one line
  that the panel is following the job: it can be closed at any time, and then no
  message comes, so they only need to ask you again.
- **No panel, or the person asks again:** call `get_job` with
  `waitSeconds=25`. The server answers the moment the job finishes, so while it is
  still `queued` or `running`, call again right away, with no pause of your own.

Do not post progress updates while the job runs: one message with the result, its
links and a short summary is enough.

| `status` | Meaning | What to do |
|---|---|---|
| `queued` | waiting in line; `position` is the place | keep waiting |
| `running` | being generated | keep waiting |
| `completed` | done; `outputs` lists the files, `result` holds text values | show the result |
| `failed` | `error.code`, `error.message`, `error.retryable` | see **When a job fails** |
| `canceled` | stopped | report it |

If a job runs much longer than expected, tell the person where it stands and ask whether
to keep waiting or cancel. Do not go silent.

## Showing the result

`get_job` returns the files three ways: `outputs` (`name`, `type`, `url`, `mime`,
`expires_at`), `resource_link` blocks, and a text hint with a ready markdown line for
images. In a client that supports it, a viewer shows the result in the chat, but the tool
result cannot tell you whether that happened.

- **Images:** put them into your reply as markdown, `![name](url)`. The hint already
  carries the line.
- **Video, audio, 3D:** give the link and say it opens there.
- **Never claim you displayed a file when you only passed its link.**
- The links are signed and expire (`expires_at`). Anything needed for longer should be
  downloaded now.
- A text model's answer is in `result`. Give it to the person as the answer.

## Errors

Every tool error starts with `[CODE; retryable=true|false|unknown]`. Retry automatically
only on `retryable=true`.

| Code | What to do |
|---|---|
| `INVALID_INPUT` | fix the named fields against the schema and the rules in the description; nothing was charged |
| `MODEL_UNAVAILABLE` | do not retry; pick another model for the same job or say this one is unavailable right now |
| `AUTH_REQUIRED`, `AUTH_ERROR`, `INSUFFICIENT_SCOPE` | the person needs to reconnect Phygital+ and approve access |
| `INSUFFICIENT_CREDITS` | tell the person their Phygital+ account is out of credits; do not retry |
| `RATE_LIMITED` | wait, then try once more |
| `FILE_TOO_LARGE` | move to the next upload channel; do not shrink the file |
| `INVALID_JOB_ID`, `JOB_NOT_FOUND` | pass the `job_id` exactly as the model tool returned it |
| `UPSTREAM_UNAVAILABLE`, `UPSTREAM_ERROR` | `get_job` can be retried. **Do not repeat a model tool blindly**: the job may already exist, and a second call charges again |

Other backend messages, such as having no access to a model, are passed through as they
are. Tell the person what the message says. These are fixed on their side, not by calling
again.

## When a job fails

1. Read `error.message`. If the input caused it (format, size, content), fix the input.
2. Retry only for a clearly transient cause (`retryable: true`), and only once. A retry is
   a new job and a new charge, so ask first when in doubt.
3. Two failures in a row for the same reason: stop and show the person `error.code`,
   `error.message` and the `job_id`.
4. If `message` is empty, say the job failed without an explanation and give the `job_id`.

`cancel_job` stops a job that is still `queued` or `running`. Report what its answer says
(`canceled: true/false`, `status`), not what you asked for.

## What not to do

- Start a model tool unprompted, in batches, or "to see what comes out".
- Repeat a model tool after an error without changing the input.
- Invent field names, enum values or a `storage_object_id`.
- Carry one model's fields over to another tool.
- Talk about prices, plans or buying credits. Point the person to their Phygital+
  account if they run out.
