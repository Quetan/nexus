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
Deduplicate overlapping facts, resolve conflicts by date, list remaining gaps.

## Input

- MemPalace findings (drawer IDs + dates)
- Wiki findings (wikilink citations + dates)

## Process

1. **Merge** — combine both result sets
2. **Deduplicate** — remove overlapping facts (same entity, same claim)
3. **Conflict resolution** — for contradictory claims:
   - Compare dates (frontmatter `date`, `valid_from`, drawer timestamp)
   - **Newer record wins**
   - If dates equal or both missing → flag for user
4. **Gap analysis** — list topics searched in either source with no record
5. **Priority ordering** — sort by relevance to task scope

## Output

```markdown
### Synthesized Context
- <fact> — source: palace drawer `<id>` / wiki `[[path]]`

### Conflicts
- <claim A> (date: YYYY-MM-DD) vs <claim B> (date: YYYY-MM-DD) — newer: ...

### Gaps
- <topic> — not found in palace or wiki
```

## Constraints

- Never silently pick a winner when dates are equal — always flag for user
- Do not blend or interpolate conflicting facts
- Preserve source traceability for every claim

## Gotchas

- Dates may be in different formats (DD.MM.YYYY vs YYYY-MM-DD) — normalize before comparing
- Equal dates or missing dates → ask user, never silently pick
