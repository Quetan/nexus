---
name: research-plan
description: >
  Use when generating a single merged deliverable — compact Контекст plus a
  deterministic Implementation Plan — after research is complete, even if the
  implementation seems straightforward. Called by nexus during Phase 5 (absorbs
  the research-brief block). Standalone use: when you have research findings and
  need a structured plan with concrete steps, file paths, commands, and
  verification for each step.
---

# research-plan

## What it does

Emit ONE deliverable: a terse Контекст block followed by a deterministic,
step-by-step implementation plan. This is the only user-facing output of the
research flow — phases are internal, the plan is the result.

## Input

- Контекст block from `research-brief`
- Task scope (repos, services)
- User language preference

## Output contract — use exactly, no other sections

```markdown
## План

### Контекст (максимум 8 буллетов)
- Схема: <1 строка> — источник: drawer <id> / [[wikilink]]
- Endpoints: <1 строка>
- Конфиги: <1 строка>
- Решения/факты: <1 буллет каждый>

### Шаги
1. `<file-path>` — <1 строка: что меняем>
   - Команда: `...`
   - Верификация: `...`
2. `<file-path>` — <1 строка: что меняем>
   - Команда: `...`
   - Верификация: `...`

### Блокеры (только если пробел/конфликт мешает конкретному шагу)
- <что мешает> — нужен ответ пользователя: ...
```

## Determinism rules

- A step is exactly three fields: file-path, Команда, Верификация. One line each.
- Small task → 3-5 steps. No artificial expansion.
- No prose after Шаги: no status summaries, no "исследование завершено", no handoff
  paragraph. Блокеры is the only optional section.
- If a gap or conflict does not block a step, it is omitted entirely.
- Empty Блокеры → single line ("Блокеры: нет"), never an empty table skeleton.
- Every step has a concrete file-path + command + verification — never a vague
  "fix the migration".
- Language: Russian or English (match user request; default Russian if unclear).
- No implementation code in this phase.
