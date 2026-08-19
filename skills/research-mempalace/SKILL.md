---
name: research-mempalace
description: >
  Use when extracting context, facts, or decisions from MemPalace for a research
  task. Called by nexus during Phase 2a vault research.
  Standalone use: when you need to search MemPalace for project context. Do NOT
  use for Obsidian wiki queries (use research-wiki) or code exploration.
compatibility: mempalace MCP
---

# research-mempalace

## What it does

Query MemPalace for institutional memory about entities, decisions, architecture,
and prior work. Returns structured findings with drawer IDs and date stamps.

## Input

- **Task scope**: `entities[]` (list) — all entities are processed within a single
  subagent dispatch, never one dispatch per entity
- **Rooms**: palace rooms mapped to the project (from manifest)

## Process

1. Determine palace rooms from project manifest or convention
   - Default rooms for any project: `documentation`, `architecture`, `decisions`
   - Project-specific rooms: from project manifest `searches` section
2. For each entity/symbol in task scope:
   - `mempalace_search(query=entity, wing=<project>, limit=3)`
   - `mempalace_kg_query(entity=<entity>)` — get relationships (max 5 relations)
   This loop runs inside one subagent context. Nexus dispatches exactly ONE
   research-mempalace subagent per task, regardless of scope size.
3. For architectural/ticket queries:
   - `mempalace_search(query=topic, wing=<project>, room=relevant-room, limit=5)`
4. Collect all findings with:
   - Drawer ID or KG fact reference
   - Date stamp (`valid_from`, frontmatter `date`, or drawer timestamp)
   - Content summary

## Output — structured contract (JSON)

```json
{
  "status": "success" | "partial" | "failed",
  "findings": [
    {"fact": "<plan-relevant fact>", "source": "drawer:<id>", "date": "YYYY-MM-DD"}
  ],
  "gaps": ["<topic>", "<topic2>"]
}
```

- `status`: success = completed coverage; partial = coverage constrained by a
  failed search or budget; failed = source unavailable (MCP down). Empty results
  are `success` with the topics in `gaps`.
- Hard cap: **10 findings**. Only facts that affect the implementation plan
  (schema, endpoints, configs, decisions).
- No prose, no narration. A "not found" entry only if it blocks a plan step.

## Constraints

- Every entity in scope gets at least one search attempt with the same limit as
  single-entity mode (`mempalace_search` limit=3) — batching saves setup overhead,
  not per-entity depth
- If a search returns nothing, record it internally as a gap (don't invent)
- Always return drawer IDs for traceability
- Never read code or grep files during this phase

## Gotchas

- Never spawn one subagent invocation per entity — batch the full scope into a single dispatch per track
- Drawer IDs are required for traceability — never omit
- KG query may return empty for new entities — record as gap, don't assume non-existence
- Legacy palace rooms may not exist — use manifest rooms only
