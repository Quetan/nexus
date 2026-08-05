---
name: research-synthesize
description: >
  Use when merging research findings from multiple sources (MemPalace + Wiki)
  into a unified context. Called by nexus after Phase 2a parallel research.
  Standalone use: after collecting data from multiple sources that need
  deduplication and conflict resolution.
---

# research-synthesize

## What it does

Merge findings from MemPalace and Wiki into a single coherent context.
Deduplicate overlapping facts, resolve conflicts by date, keep only facts the
implementation plan needs.

## Input

- MemPalace findings — structured contract `{status, findings[], gaps[]}`
- Wiki findings — same contract; `source` = `wiki:<path>`

## Process

1. **Merge** — combine both `findings[]` arrays directly (no prose parsing).
   A track with `status: failed` becomes a gap ("source unavailable"), never
   confused with a legitimate empty result.
2. **Deduplicate** — remove overlapping facts (same entity, same claim)
3. **Conflict resolution** — for contradictory claims:
   - Compare dates (frontmatter `date`, `valid_from`, drawer timestamp)
   - **Newer record wins**
   - If dates equal or both missing → flag for user
4. **Plan-relevance filter** — drop facts that do not affect implementation
   (schema, endpoints, configs, decisions). Keep only what the plan needs.
5. **Priority ordering** — sort by relevance to task scope

## Output — facts for the plan only

```markdown
### Факты для плана
- <fact> — source: palace drawer `<id>` / wiki `[[path]]`
```

- Hard cap: **15 bullets**. Merge related facts to stay under it.
- Gaps and conflicts are surfaced ONLY when they block a plan step — otherwise
  omit. If one blocks, emit it as a single bullet ending with `— БЛОКЕР`.
- Never narrate what was searched or missing.

## Constraints

- Never silently pick a winner when dates are equal — always flag for user
  (as a БЛОКЕР bullet).
- Do not blend or interpolate conflicting facts
- Preserve source traceability for every claim
- **Cross-source conflict resolution:** MemPalace and Wiki are external memories;
  newer date wins by default (unlike within-wiki conflicts in vault-query, which
  always escalate to user)

## Gotchas

- Dates may be in different formats (DD.MM.YYYY vs YYYY-MM-DD) — normalize before comparing
- Equal dates or missing dates → ask user, never silently pick
