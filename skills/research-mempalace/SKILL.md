---
name: research-mempalace
description: >
  Use when extracting context, facts, or decisions from MemPalace for a research
  task. Called by nexus during Phase 2a vault research. Requires user-mempalace MCP.
  Standalone use: when you need to search MemPalace for project context. Do NOT
  use for Obsidian wiki queries (use research-wiki) or code exploration.
---

# research-mempalace

## What it does

Query MemPalace for institutional memory about entities, decisions, architecture,
and prior work. Returns structured findings with drawer IDs and date stamps.

## Input

- **Task scope**: entities, repos, ticket IDs
- **Rooms**: palace rooms mapped to the project (from manifest)
- **Query budget**: narrow (2-3), default (3-5), broad (4-6)

## Process

1. Determine palace rooms from project manifest or convention
   - Default rooms for any project: `documentation`, `architecture`, `decisions`
   - Project-specific rooms: from project manifest `searches` section
2. For each entity/symbol in task scope:
   - `mempalace_search(query=entity, wing=<project>, limit=3)`
   - `mempalace_kg_query(entity=<entity>)` — get relationships
3. For architectural/ticket queries:
   - `mempalace_search(query=topic, wing=<project>, room=relevant-room, limit=5)`
4. Collect all findings with:
   - Drawer ID or KG fact reference
   - Date stamp (`valid_from`, frontmatter `date`, or drawer timestamp)
   - Content summary

## Output

```markdown
### MemPalace Findings
- <fact> — drawer: `<id>` (date: YYYY-MM-DD)
- <entity relationship> — KG: `<subject>` → `<predicate>` → `<object>`
```

## Constraints

- Respect query budget — don't exceed the allocated count
- If a search returns nothing, record it as a gap (don't invent)
- Always return drawer IDs for traceability
- Never read code or grep files during this phase

## Gotchas

- Drawer IDs are required for traceability — never omit
- KG query may return empty for new entities — record as gap, don't assume non-existence
- Legacy palace rooms may not exist — use manifest rooms only
