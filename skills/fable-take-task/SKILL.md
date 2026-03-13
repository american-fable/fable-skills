---
name: fable-take-task
description: "Take assigned tasks from Fable. Use '/fable-take-task' for one task, '/fable-take-task --loop' to poll continuously."
user-invocable: true
---

# Take Task

Fetch the next assigned task from the Fable storage API and execute it.

**IMPORTANT**: This skill runs non-interactively. Do NOT ask the user any questions or narrate what you're doing. No preamble, no "let me fetch the task", no warnings about environment variables or self-signed certs. Just silently execute each step and only print a brief summary after each task.

If the argument `--loop` is provided, repeat the steps below until there are no more tasks. Otherwise, run them once.

## Environment

- `FABLE_JWT` (required) — authentication token
- `FABLE_URL` (optional) — base URL, defaults to `https://fable.is`
- `FABLE_COOKIES` (optional) — passed to curl via `-b`
- `FABLE_INSECURE` (optional) — if set to `1`, adds `--insecure` to curl

## Curl

All HTTP requests use `curl -s` with these additions:
- Base URL: `FABLE_URL` if set, otherwise `https://fable.is`
- If `FABLE_INSECURE` is `1`, add `--insecure`
- If `FABLE_COOKIES` is set, add `-b` with its value

Construct the full curl command accordingly and reuse it for all requests below.

## Steps

1. Run a curl command to fetch the task (do not print anything before running it):
   ```
   curl -s [insecure/cookie flags] "$FABLE_URL/storage/agent/task-take?jwt=$FABLE_JWT"
   ```
   - If `FABLE_JWT` is empty, say "FABLE_JWT not set." and stop.
   - If the response is 404: in `--loop` mode, sleep 10 seconds and go back to step 1. Without `--loop`, say "No tasks available." and stop.
   - The endpoint returns JSON: `{"taskId": "...", "strings": ["parent", "child", ...]}`

2. Extract the `strings` array. Join them with " > " to form the task path. The last string is the task to do; preceding strings are context.

3. Launch an Agent subagent with a prompt that includes:
   - The full task path for context
   - The last string as the specific task to complete
   - Instructions to complete the task in the codebase
   - A reminder to work autonomously without asking questions — use best judgment for any ambiguity

4. After the subagent completes, post the result back to Fable. URL-encode the taskId JSON and the reply text:
   ```
   curl -s [insecure/cookie flags] -G "$FABLE_URL/storage/agent/task-update?jwt=$FABLE_JWT" --data-urlencode "taskId=<taskId JSON from step 1>" --data-urlencode "reply=<summary of what the subagent did>"
   ```

5. Print a one-line summary of the completed task.