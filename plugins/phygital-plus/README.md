# Phygital+ for agents

This plugin connects an agent to Phygital+ generative models: choose a model,
read its current input schema, upload an input file, start a generation, and
wait for the result.

Generation runs on the remote MCP server at
`https://dev.phygital.plus/api/mcp`. This is the production endpoint.
The plugin supplies connection settings and instructions for the agent;
it does not host or run the models locally.

## Package contents

| Path | Purpose |
| --- | --- |
| `.claude-plugin/plugin.json` | Claude Code manifest |
| `plugin.json` | Portable manifest, with OpenAI settings under `extensions."com.openai"` |
| `.mcp.json` | MCP connection in Claude Code format |
| `mcp.json` | Streamable HTTP connection with OAuth through DCR |
| `skills/phygital-mcp/` | Tool sequence, input schemas, uploads, job tracking, and errors |
| `skills/phygital-which-ai/` | Model selection guidance |
| `commands/` | Claude Code commands: `/phygital-plus:connect`, `:models`, `:generate`, and `:job` |

Skills are included in this directory. Keep their instructions aligned with
the tools and schemas exposed by the server.

## Installation

Claude Code:

```sh
claude plugin marketplace add phygitalism/phygital-plus-agent-plugin
claude plugin install phygital-plus@phygital-plus
```

Codex:

```sh
codex plugin marketplace add phygitalism/phygital-plus-agent-plugin
```

Then install the plugin from the client's plugin catalog.
The Codex marketplace is declared in `.agents/plugins/marketplace.json`
at the repository root.

## Versions

Keep `version` synchronized in `plugin.json` and `.claude-plugin/plugin.json`
when releasing an update. Claude Code uses the version to detect plugin updates.

## Authentication

**OAuth is the packaged default.** The client opens the sign-in flow when
connecting and stores the token. `.mcp.json` intentionally omits authorization
headers so the client can use OAuth discovery.

**Personal API keys** are available for development setups and clients without
OAuth. Obtain a key from the portal's API key section and configure your client
manually. The following example assumes the client supports environment-variable
substitution in MCP headers:

```json
{
  "phygital": {
    "type": "http",
    "url": "https://dev.phygital.plus/api/mcp",
    "headers": { "Authorization": "Bearer ${PHYGITAL_API_KEY}" }
  }
}
```

Store the key in `PHYGITAL_API_KEY`, not in the configuration file. Requests
consume the account owner's credits. Use `/phygital-plus:connect` in Claude Code
to check the connection.

## Other clients

Any client supporting remote MCP over Streamable HTTP can use
`https://dev.phygital.plus/api/mcp`. A plugin package is only needed by clients
that support that format.

Skills are not transferred automatically to clients without skill support.
Add the instructions from `skills/phygital-mcp/` manually; the portal also
provides skill instructions on its `/skills` page.

For the separate OpenAI variant, see
[Phygital+ OpenAI](../phygital-plus-openai/README.md).
