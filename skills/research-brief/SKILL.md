---
name: research-brief
description: >
  Use when generating a structured Research Brief in Russian after vault and code
  exploration are complete — even for small or obvious tasks. Called by nexus
  during Phase 4. Standalone use: when you have research findings and need a
  formatted brief document. Requires completed vault synthesis (research-synthesize)
  and code exploration.
---

# research-brief

## What it does

Format research findings into a structured Research Brief in Russian, including
Актуальное состояние (current state) section with schema, endpoints, config, code.

## Input

- Synthesized context from `research-synthesize`
- Code exploration findings (schema, endpoints, configs, key files)
- Task description

## Template

```markdown
## Research Brief

**Задача:** <одна строка>

### Контекст
- <факт> — источник: palace drawer `<id>` / wiki `[[path]]`

### Актуальное состояние
- **Схема БД:** (таблицы, ключи, nullable, индексы)
- **Endpoints:** (роуты, методы, middleware)
- **Конфиги:** (env vars, порты)
- **Код:** (ключевые файлы и их роль)
- **Зависимости:** (что ещё нужно изменить)

### Принятые решения
- <решение> — источник: ...

### Пробелы (не найдено)
- <тема поиска, записи нет>

### Конфликты
- <если есть, с указанием какая запись новее>
```

## Constraints

- Language: **Russian**
- Every fact must cite a source (drawer ID or wikilink)
- Gaps section is mandatory even if empty
- Conflicts section is mandatory even if empty

## Gotchas

- Brief must be in Russian even if the conversation was in English
- Актуальное состояние section is mandatory — even if all fields are "not found"
