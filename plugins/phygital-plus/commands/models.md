---
description: Show Phygital+ models suited to a task
argument-hint: "[what to generate]"
---

## Task

Find models matching `$ARGUMENTS`. If no arguments are supplied, show the full catalog.

## Steps

1. Call `list_models`. Pass `category` when the request specifies one.
2. Select matches by name and description. Do not suggest models marked
   `deprecated` unless the user explicitly asks for them.
3. Show a table with the model ID, name, category, and a one-line description.
4. If there is one clear candidate, call `get_model_schema` and list its required
   inputs. Use that exact model's current schema: parameters differ across models
   and change over time.

## Avoid

Do not describe inputs from memory or copy parameters from another model.
Fetch the schema to avoid validation errors.
