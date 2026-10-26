# Phygital+ agent plugins

Connect AI agents to [Phygital+](https://phygital.plus/) to generate images,
video, audio, 3D models, and text using your Phygital+ account.
Generations consume credits; subscriptions and credits are purchased on the
Phygital+ website.

## Choose a plugin

| Plugin | MCP endpoint | Workflow |
| --- | --- | --- |
| [Phygital+](plugins/phygital-plus/README.md) | `https://dev.phygital.plus/api/mcp` | Browse models, read a model's current input schema, then generate |
| [Phygital+ OpenAI](plugins/phygital-plus-openai/README.md) | `https://dev.phygital.plus/api/mcp/openai` | Use a separate tool for each model, with its own input schema |

`dev.phygital.plus` is the production hostname for these MCP integrations.
Both plugins include instructions for file uploads, job tracking, and error handling.

## Claude Code

Add this repository as a marketplace, then install the general MCP plugin:

```sh
claude plugin marketplace add phygitalism/phygital-plus-agent-plugin
claude plugin install phygital-plus@phygital-plus
```

## Codex

```sh
codex plugin marketplace add phygitalism/phygital-plus-agent-plugin
```

Then install the desired plugin from the client's plugin catalog.
The Codex marketplace is defined in `.agents/plugins/marketplace.json`.

## ChatGPT

Use the [OpenAI plugin](plugins/phygital-plus-openai/README.md), which connects
to `/api/mcp/openai` and exposes each model as a separate tool.
See [submission preparation](docs/openai-submission.md) for the recorded demo,
review evidence, and remaining checks. A GitHub release does not indicate
approval or publication in the OpenAI directory.

## Authentication

The packaged configuration uses OAuth with a Phygital+ account. Complete the
client's sign-in flow when prompted. Keep credentials and tokens out of source
files and chat messages.

## Other MCP clients

Clients supporting remote MCP servers over Streamable HTTP can connect to
`https://dev.phygital.plus/api/mcp` directly. If a client does not support
packaged skills, add the relevant instructions from
`plugins/phygital-plus/skills/` to the agent's instructions manually.

## Releases and support

- [Release packages and demo videos](https://github.com/phygitalism/phygital-plus-agent-plugin/releases)
- [Phygital+ website and support contact](https://phygital.plus/)
- [Privacy policy](https://phygital.plus/privacy-policy/)
- [Terms and conditions](https://library.phygital.plus/terms-and-conditions)
