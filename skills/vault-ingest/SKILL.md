---
name: vault-ingest
description: >
  Use when integrating source notes from sources/ into wiki entity, concept,
  decision, and task pages. Triggers on "ingest", "integrate source",
  "разложи по полкам". Requires user-seekstone MCP. Automatically calls
  vault-index for index/log updates. Do NOT use for initial session capture
  (use vault-write) or for vault queries (use vault-query). Requires: vault-index.
---

# vault-ingest

## What it does

Decompose source notes into wiki pages — entities, concepts, decisions, tasks.
Updates wiki/index.md, project 00_Index.md, and log.md through vault-index.

## Process

0. **Cache check** — if source SHA256 unchanged vs `.llm-wiki/ingest-cache.json` → skip
1. **Read source** — decompose into entities, concepts, decisions, data points
2. **Overlap scan** — check existing wiki pages for matching content
3. **Propose change package** — list new pages + updates + contradictions
4. **User approval** — new pages need explicit approval; contradictions → ask
5. **Execute** — create/update pages → call `vault-index` (all indexes + log)
6. **Cache** — update `.llm-wiki/ingest-cache.json` with source SHA256
7. **Verify** — read back created pages

## Constraints

- SHA256 computation failure → fallback to cache miss (never block)
- One source should yield multiple page updates across the wiki graph
- New pages need explicit user approval before creation
- Contradictions with existing wiki content → ask user (no silent newer-wins)
