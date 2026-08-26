---
name: research-wiki
description: >
  Use when extracting context from Obsidian LLM Wiki for a research task. Called
  by nexus during Phase 2a vault research. Delegates actual wiki queries to
  vault-query. Requires: vault-query.
  Standalone use: when you need to look up project knowledge from the wiki
  without the full research flow. Do NOT use for MemPalace queries (use
  research-mempalace) or code exploration.
compatibility: mcpvault MCP
---

# research-wiki

## What it does

Extract project context from Obsidian LLM Wiki by delegating to `vault-query`.
Returns wikilink-cited findings for the research task.

## Input

- **Task scope**: `entities[]` — process the full list in **one** invocation
  (never one Task/dispatch per entity)
- **External-source gate**: `allow_external` false unless requester authorizes
- **Invocation**: medium/full → one Task (`nexus-wiki` or `generalPurpose`);
  light → parent runs this skill inline
- **mcp-mode**: default `repo-bound` (`wiki/` MCP prefix)

## Process

1. Read vault `index.md` → candidate pages
2. Call `vault-query` **once** (batch) for the full question list
3. For cross-cutting topics: backlinks via
   `search_notes(query="<title>", searchContent=true)` or grep `[[<basename>]]`
4. Collect findings with wiki path, date, content summary

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

- `status`: success = completed coverage; partial = coverage constrained by a
  failed search or budget; failed = source unavailable (mcpvault down). Empty
  results are `success` with the topics in `gaps`.
- Hard cap: **10 findings**. Only facts that affect the implementation plan
  (schema, endpoints, configs, decisions).
- No prose, no narration. A "not found" entry only if it blocks a plan step.

## Constraints

- Vault-only by default. Use web, codegraph, or other external evidence only when
  `allow_external: true` was explicitly authorized by the requester.
- Every entity in scope gets the same per-entity depth as single-entity mode
  (candidate pages from index.md + search) — batching saves setup overhead, not depth.
- For a within-wiki conflict, return both dated claims in `findings[]` and add a
  gap that requester resolution is required. Never select a newer winner.
- If topic has no record, note it internally as a gap.

## Gotchas

- Never one Task per entity — one track invocation batches the full scope
- vault-query is vault-only unless `allow_external: true`
- index.md may be incomplete — also search directly
