---
name: research-wiki
description: >
  Use when extracting context from Obsidian LLM Wiki for a research task. Called
  by nexus during Phase 2a vault research. Delegates actual wiki queries to
  vault-query. Requires: vault-query, mcpvault MCP.
  Standalone use: when you need to look up project knowledge from the wiki
  without the full research flow. Do NOT use for MemPalace queries (use
  research-mempalace) or code exploration.
---

# research-wiki

## What it does

Extract project context from Obsidian LLM Wiki by delegating to `vault-query`.
Returns wikilink-cited findings for the research task.

## Input

- **Task scope**: `entities[]` (list) — all entities are processed within a single
  subagent dispatch, never one dispatch per entity

## Process

1. Read vault `index.md` → list candidate pages
2. Call `vault-query` ONCE with the full question list (batch mode) — one combined
   result for all entities, not one call per entity. The loop runs inside one
   subagent context; nexus dispatches exactly ONE research-wiki subagent per task,
   regardless of scope size.
3. For architectural/cross-cutting topics:
   - Follow backlinks from central pages (`search_notes(query="<title>", searchContent=true)` or file I/O grep for `[[<basename>]]`)
   - Read relevant sections adaptively

## Output — structured contract (JSON)

```json
{
  "status": "success" | "partial" | "failed",
  "findings": [
    {"fact": "<plan-relevant fact>", "source": "wiki:<vault-relative-path>", "date": "YYYY-MM-DD"}
  ],
  "gaps": ["<topic>", "<topic2>"]
}
```

- `status`: success = all entities searched; partial = some searches empty/failed;
  failed = source unavailable (mcpvault down). Empty result is NOT `failed` — it is
  `success` with the topics in `gaps`.
- Hard cap: **10 findings**. Only facts that affect the implementation plan
  (schema, endpoints, configs, decisions).
- No prose, no narration. A "not found" entry only if it blocks a plan step.

## Constraints

- Vault-only by default. External sources (web, codegraph) only if user explicitly allows.
- Every entity in scope gets the same per-entity depth as single-entity mode
  (candidate pages from index.md + search) — batching saves setup overhead, not depth.
- Conflicts found in wiki → pass to `research-synthesize` with both dates.
- If topic has no record, note it internally as a gap.

## Gotchas

- Never spawn one subagent invocation per entity — batch the full scope into a single dispatch per track
- vault-query is vault-only by default — don't supplement with web/codegraph unless allowed
- index.md may be incomplete after many ingests — also search directly
