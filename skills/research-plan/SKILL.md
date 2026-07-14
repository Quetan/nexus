---
name: research-plan
description: >
  Use when generating an Implementation Plan after research is complete — even
  if the implementation seems straightforward. Called by nexus during Phase 5.
  Standalone use: when you have research findings and need a structured plan
  with concrete steps, file paths, commands, and verification for each step.
---

# research-plan

## What it does

Generate a structured Implementation Plan with concrete file paths, commands, and
verification for each step. Delivered in Russian or English (match user request).

## Input

- Research Brief from `research-brief`
- Task scope (repos, services)
- User language preference

## Template

```markdown
## План реализации

### Scope
- Репозитории: ...
- Сервисы: ...

### Шаги
1. `<file-path>` — описание изменения
   - Команда: `npm run ...`
   - Верификация: `npx jest ...`
2. `<file-path>` — описание изменения
   - Команда: `...`
   - Верификация: `...`

### Верификация
- [ ] `npx tsc --noEmit`
- [ ] `jest --testPathPattern=<pattern>`
- [ ] `curl <endpoint>` — smoke test

### Риски / открытые вопросы
- ...

### Handoff
Исследование завершено. Утвердите план или скажите «приступаем к реализации».
```

## Constraints

- No implementation code in this phase
- Every step must have: file-path, command, verification
- Language: Russian or English (match user request; default Russian if unclear)

## Gotchas

- Each step must have file-path + command + verify — no vague "fix the migration"
- Match language to user request; default Russian
