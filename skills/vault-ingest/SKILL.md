---
name: vault-ingest
description: >
  Use when integrating source notes from raw/ into wiki entity, concept,
  decision, and task pages. Triggers on "ingest", "integrate source",
  "разложи по полкам". Requires seekstone MCP. Automatically calls
  vault-index for index/log updates. Do NOT use for initial session capture
  (use vault-write) or for vault queries (use vault-query). Requires: vault-index.
---

# vault-ingest

## What it does

Decompose source notes into wiki pages — entities, concepts, decisions, tasks.
Updates wiki/index.md, wiki/projects/<project>/00_Index.md, and wiki/log.md through vault-index.

## References

Load when needed:
- Read [references/ingest-protocol.md] **before decomposing a source** — cache check, overlap scan, page type mapping
- Read [references/vault-conventions.md] **before creating/updating pages** — enforcement of conventions 0.1, 0.2, 0.4

## Process

0. **Cache check** — if source SHA256 unchanged vs `.llm-wiki/ingest-cache.json` → skip
1. **Read source** — decompose into entities, concepts, decisions, data points
1.5. **Convention enforcement** (vault-conventions.md):
   - Rule 0.1: Concepts → auto-add date if missing
   - Rule 0.2: Decisions/dated Concepts → set lifecycle:frozen (default), frozen pages require user approval for merge
   - Rule 0.4: Archived pages → verify /archive/ path, reject if outside
2. **Discuss takeaways** — discuss key findings with the user: what to emphasize, what to skip, what's surprising
3. **Overlap scan** — check existing wiki pages for matching content
4. **Propose change package** — list new pages + updates + contradictions
5. **User approval** — new pages need explicit approval; contradictions → ask
6. **Execute** — create/update pages → call `vault-index` (all indexes + log)
7. **Cache** — update `.llm-wiki/ingest-cache.json` with source SHA256
8. **Verify** — read back created pages

## Constraints

- SHA256 computation failure → fallback to cache miss (never block)
- One source typically updates 5-15 wiki pages across entities, concepts, decisions, and topics — small/narrow sources may legitimately produce fewer; precision over count
- New pages need explicit user approval before creation
- Contradictions with existing wiki content → ask user (no silent newer-wins)

## Gotchas

- Typical range: 5-15 wiki pages. Real concern: don't collapse into single summary. Small sources producing 1-2 pages are fine if properly decomposed.
- SHA256 cache prevents re-processing — check before reading source
- User must approve new pages — draft first
