---
description: Check the Phygital+ connection and explain authentication options
---

## Task

Check whether the Phygital+ MCP server responds. If it does not, explain the
available authentication options.

## Steps

1. Call `list_models` without arguments. The catalog requires a token, so this
   checks both server access and authentication.
2. **Success:** report how many models are available and stop.
3. **Authentication error** (`[UNAUTHORIZED; ...]`, HTTP 401, or a sign-in request):
   explain both supported options and let the user choose.

   **OAuth is the default.** The client opens the sign-in flow when connecting.
   If it does not appear, reconnect through `/mcp` in Claude Code or reinstall
   the plugin.

   **Personal API key:** for development setups and clients without OAuth,
   obtain a key from the portal's API key section. The user configures their
   MCP client manually, using environment-variable substitution where supported:

   ```json
   {
     "phygital": {
       "type": "http",
       "url": "https://dev.phygital.plus/api/mcp",
       "headers": { "Authorization": "Bearer ${PHYGITAL_API_KEY}" }
     }
   }
   ```

   Store the key in the `PHYGITAL_API_KEY` environment variable, not in a file.
4. **Other error:** show the returned message. Server errors use the prefix
   `[CODE; retryable=true|false|unknown]`. Retry automatically only when
   `retryable=true`.

## Avoid

Do not request an API key in chat or write it into files for the user. The key
spends the account owner's credits and belongs in an environment variable.
