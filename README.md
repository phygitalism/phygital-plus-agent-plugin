# Phygital+ для агентов

Плагин подключает генеративные модели [Phygital+](https://dev.phygital.plus) к
агенту: выбрать модель, прочитать её схему входа, загрузить файл, запустить
генерацию и дождаться результата. Работа идёт на MCP-сервере
`https://dev.phygital.plus/api/mcp`, вход через OAuth с обычным аккаунтом
Phygital+.

## Claude Code

```
claude plugin marketplace add <org>/phygital-plus-agent-plugins
claude plugin install phygital-plus@phygital-plus
```

## Codex

```
codex plugin marketplace add <org>/phygital-plus-agent-plugins
```

Дальше плагин ставится из каталога плагинов.

## ChatGPT

Для ChatGPT — отдельный плагин `plugins/phygital-plus-openai`: он подключается к
`https://dev.phygital.plus/api/mcp/openai`, где у каждой модели свой инструмент
вместо каталога и общего `generate`. Так требуют правила подачи приложений OpenAI.
Подробности — в [plugins/phygital-plus-openai/README.md](plugins/phygital-plus-openai/README.md).

## Другие клиенты

Любой клиент с поддержкой удалённых MCP-серверов (Streamable HTTP) подключается
по адресу `https://dev.phygital.plus/api/mcp`. Навыки из
`plugins/phygital-plus/skills/` в таком случае вставляются в инструкции агента
вручную.

