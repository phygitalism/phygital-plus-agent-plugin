# Phygital+ OpenAI plugin

Connects ChatGPT and Codex to the production MCP endpoint:
`https://dev.phygital.plus/api/mcp/openai`.

The `dev.phygital.plus` hostname is the production deployment for this plugin.
Authentication uses OAuth with Dynamic Client Registration (DCR).
Each generation runs on the connected Phygital+ account and consumes credits.
Users purchase subscriptions and credits on the Phygital+ website.

## Package

- `plugin.json`: listing, publisher, commerce, review cases, and demo URL.
- `mcp.json`: production MCP endpoint and OAuth configuration.
- `skills/phygital-openai/SKILL.md`: model selection, uploads, and job tracking.
- `assets/icon.png`: listing and composer icon.

Each model has a separate tool with its own input schema. This OpenAI variant
uses `/api/mcp/openai`; the other plugin in this repository uses `/api/mcp`.

## Demo

[English demo: FLUX, GPT Text, and unsupported history retrieval](https://github.com/phygitalism/phygital-plus-agent-plugin/releases/download/openai-v0.1.1/phygital-plus-demo.mp4)

## Submission status

Version 0.1.1 prepares the package for submission. A GitHub release is not an
OpenAI review approval or public directory publication.
See [submission preparation](../../docs/openai-submission.md) for evidence and open checks.
Build a ZIP from this directory only, excluding the sibling plugin and marketplace files.
Do not include credentials, OAuth tokens, or reviewer account details in source or archives.

After MCP server changes, use the portal's scan flow and check tool availability.
Package metadata and skill changes require an updated ZIP.
