---
description: Check the status of a Phygital+ job by its ID
argument-hint: <job_id>
---

## Task

Show the status of the job identified by `$ARGUMENTS`.

## Steps

1. Call `get_job` with that `job_id`.
2. If the job has reached a terminal status, show the result or the failure reason.
3. If it is still running, report its status and queue position. If the user asked
   you to wait, call `get_job` with `waitSeconds=25` and repeat immediately until
   it finishes. Do not post intermediate status messages while waiting.
4. If `job_id` cannot be found, explain that the server has no job-history listing
   and cannot recover a lost ID.
