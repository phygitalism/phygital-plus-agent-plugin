---
description: Start a Phygital+ generation and wait for the result
argument-hint: <model or task description>
---

## Task

Run the generation described in `$ARGUMENTS` and follow it through to the result.

## Steps

1. Identify the model. If `$ARGUMENTS` does not name one, call `list_models`,
   suggest options, and wait for the user's choice. Generation consumes credits.
2. Call `get_model_schema` for the selected model. Build the input only from
   that schema.
3. For file inputs, choose the upload channel based on who has the file and
   its size. Keep the original file; do not recompress or resize it.
   - If you have the bytes and they fit one call, use `upload_file`.
   - For larger files, use `start_upload`, `upload_chunk`, and `finish_upload`.
     Split the file bytes, not the base64 string.
   - If you can make HTTP requests, use `get_upload_url`, then POST the file
     with the issued upload pass. No API key is needed for that upload.
   - If the user has the file, open `request_upload` and wait for the returned
     `storage_object_id` in their message. If the panel gives them a code instead,
     use `claim_upload` with that code. Never invent an ID.

   A chat image attachment arrives as an image in context, not as file bytes
   you can forward. Read upload limits from the server rather than assuming them.
4. Show the assembled input and wait for confirmation before calling `generate`.
5. Call `generate` and preserve the returned `job_id`. A lost ID cannot be recovered.
6. Wait for the result. When a generation panel is active, it tracks the job and
   sends one completion message; do not call `get_job` meanwhile. Without a panel,
   or if no message arrives for several minutes, call `get_job` with
   `waitSeconds=25`. While the job remains active, repeat immediately without
   adding delays or intermediate status messages. Do not use
   `averageTimeInSeconds` as a typical estimate: it represents the catalog's
   worst-case duration.
7. Return the result link and display images in the reply. Do not claim a file
   was displayed if you only provided a link.

## Avoid

Do not start several jobs at once or generate exploratory samples without a
request. Every generation consumes the user's credits.
