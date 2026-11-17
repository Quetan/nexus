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
and prior work. Choose the read tool that fits the question and return sourced findings.

## Input

- **Task scope**: `question` and `entities[]` — process the full list in **one** invocation
  (never one Task/dispatch per entity)
- **Rooms**: from Phase 1 defaults or manifest when looked up
- **Invocation**: medium/full → one `nexus-mempalace` agent with its runtime's
  configured research model; use a generic agent only under the nexus fallback
  rules;
  light → parent runs this skill inline

## Process

1. Determine palace rooms from Phase 1 / manifest or convention
   - Default rooms: `documentation`, `architecture`, `decisions`
   - Project-specific: from manifest `searches` when available
2. Select the smallest read that can answer the supplied question:
   - Use `mempalace_search(query=<short query>, wing=<project>, room=<room>, limit=3)` for an entity lookup, or `limit=5` for a topic, ticket, or architecture search.
   - Use `mempalace_kg_query(entity=<known entity>)` for a question about its known relationships; cap at 10 relations.
   - Use `mempalace_diary_read(agent_name=<known agent>, wing=<project>, last_n=3)` for recent continuity by a known agent.
3. Search first in the relevant room for tickets or architecture. Retry with
   `candidate_strategy="union"` when exact IDs, filenames, or ticket numbers may
   have been missed by semantic search. If results are noisy, set `max_distance`
   explicitly and judge the returned text; do not assume a universal threshold.
4. Check matching source text before treating a search result as the project
   manifest. Use `mempalace_get_drawers(drawer_ids=[...])` to read several known
   sources in one call when available. Add a second read only to close a
   plan-relevant gap.
5. Cite drawer IDs for sourced claims. Use a date only when the source supports
   it, such as `content_date` with a known provenance; `filed_at` alone is the
   filing date, not evidence of when a decision was made. Use `"date": ""`
   when the source date is unknown. KG or diary findings without a drawer ID
   must be confirmed with a targeted search or recorded as a gap.
## Output — structured contract (JSON)

```json
{
  "status": "success" | "partial" | "failed",
  "findings": [
    {"fact": "<plan-relevant fact>", "source": "drawer:<id>", "date": "YYYY-MM-DD or empty string when unknown"}
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

- Choose tools from the question; do not call search and KG for every entity.
- Keep one track invocation for the full scope. Preserve limits of 3 for entity
  searches and 5 for topic/ticket searches; use the KG limit and 10-finding cap above.
- An empty result is success with a topic gap. If an optional search parameter
  is rejected, retry once with the same query and filters using default search.
- Always return drawer IDs for traceability
- Never read code or grep files during this phase

## Gotchas

- Never one Task per entity — one track invocation batches the full scope
- Drawer IDs required for traceability
- Empty KG → gap, not "does not exist"; do not query KG without a relationship question
- Legacy rooms may be missing — use Phase 1 / manifest rooms only
