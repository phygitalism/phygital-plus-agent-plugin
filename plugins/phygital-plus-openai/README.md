# Phygital+ для ChatGPT

Отдельный плагин под OpenAI: смотрит на `https://dev.phygital.plus/api/mcp/openai`, а не
на `/api/mcp`.

## Почему отдельно

Правила подачи приложений в ChatGPT
([App submission guidelines](https://developers.openai.com/apps-sdk/app-submission-guidelines))
требуют отдельный инструмент на каждую операцию и запрещают discovery с универсальным
исполнителем:

> Expose each model-callable operation as a separate tool with a clear description,
> input schema, and annotations.
>
> Do not use discovery, operation selection, or schema fetching with a generic executor.

Основной плагин `phygital-plus` устроен ровно так: `list_models` → `get_model_schema` →
`generate`. Для Claude, Codex и остальных клиентов это удобно, и там он остаётся как
есть. На `/api/mcp/openai` вместо этих трёх инструментов у каждой модели свой
(`generate_image_gpt`, `generate_video_kling`, `upscale_image`, …) со схемой входа этой
модели. У всех инструментов явно выставлены `readOnlyHint` / `destructiveHint` /
`openWorldHint`.

## Что внутри

| Что | Зачем |
|---|---|
| `plugin.json` | манифест, витрина OpenAI — в `extensions."com.openai"` |
| `mcp.json` | сервер `/api/mcp/openai`, OAuth через DCR |
| `skills/phygital-openai/` | инструкция агенту под инструменты по моделям |

## Набор моделей

Список инструментов зафиксирован на сервере (`MODEL_TOOLS` в `app/mcp_openai.py`
портала). OpenAI сканирует метаданные инструментов при подаче версии, поэтому новая
модель появляется в ChatGPT только после правки этого списка и подачи новой версии
плагина. Хост сервера между версиями менять нельзя, только путь.
