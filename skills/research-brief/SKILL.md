---
name: research-brief
description: >
  Use when generating a compact Контекст block in Russian after vault and code
  exploration are complete — even for small or obvious tasks. Called by nexus
  during Phase 4. Standalone use: when you have research findings and need a
  terse current-state block. Requires completed vault synthesis
  (research-synthesize) and code exploration.
---

# research-brief

## What it does

Reduce research findings to a compact Контекст block: only facts that affect the
implementation plan. No narrative, no "what was found / not found".

## Input

- Synthesized context from `research-synthesize`
- Code exploration findings (schema, endpoints, configs, key files)
- Task description

## Output contract

```markdown
### Контекст (максимум 8 буллетов — только то, что влияет на план)
- Схема: <1 строка> — источник: drawer <id> / [[wikilink]]
- Endpoints: <1 строка>
- Конфиги: <1 строка>
- Решения/факты: <1 буллет каждый>
```

## Constraints

- Language: **Russian**
- Hard cap: 8 bullets. Merge related facts to stay under it.
- Each bullet cites one source (drawer ID or wikilink), no exception.
- A gap or conflict is included ONLY if it blocks a plan step — otherwise omit.
- Never narrate searches, near-misses, or "не найдено" entries.
- Empty Gaps/Conflicts content → single line ("Пробелы: не обнаружено" /
  "Конфликты: не обнаружено"), never a full table skeleton.
- Do not add sections, headings, or prose beyond the block.

## Gotchas

- Bullet count wins over completeness — a plan that is missing an unused fact is fine.
- Brief must be in Russian even if the conversation was in English.
