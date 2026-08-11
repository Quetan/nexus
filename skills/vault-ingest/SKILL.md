---
name: vault-ingest
description: >
  Use when integrating manual/external source notes from raw/ into wiki entity,
  concept, decision, and task pages. Triggers on "ingest", "integrate source",
  "разложи по полкам", "обработай raw". Requires mcpvault MCP. Automatically
  calls vault-index for index/log updates. Do NOT use for new session capture
  (use vault-write — direct wiki write, no raw staging) or for vault queries
  (use vault-query). Requires: vault-index.
---

# vault-ingest

## What it does

Decompose **manually placed** source notes from `raw/` into wiki pages —
entities, concepts, decisions, tasks. Updates vault `index.md`,
`projects/<project>/00_Index.md`, and vault `log.md` through vault-index.

**New session work does not go through this skill.** `vault-write` writes
directly into `wiki/`. Use `vault-ingest` only when the user drops files into
`raw/` by hand (external dumps, notes from other sessions, ad-hoc captures).

## References

Load when needed:
- [Ingest protocol](references/ingest-protocol.md) — before decomposing a source
- [Vault conventions](references/vault-conventions.md) — before creating/updating pages (rules 0.1–0.4)

## Process

0. **Cache check** — if source SHA256 unchanged vs `.llm-wiki/ingest-cache.json` → skip
1. **Read source** — decompose into entities, concepts, decisions, data points
1.5. **Convention enforcement** (vault-conventions.md):
   - Rule 0.1: Concepts → auto-add date if missing
   - Rule 0.2: Decisions/dated Concepts → set lifecycle:frozen (default), frozen pages require user approval for merge
   - Rule 0.3: result wiki pages must NOT reference the raw file
   - Rule 0.4: Archived pages → verify /archive/ path, reject if outside
2. **Discuss takeaways** — discuss key findings with the user: what to emphasize, what to skip, what's surprising
3. **Overlap scan** — check existing wiki pages for matching content
4. **Propose change package** — list new pages + updates + contradictions
5. **User approval** — new pages need explicit approval; contradictions → ask
6. **Execute** — create/update at MCP `wiki/...`; read `raw/...` → call `vault-index`
7. **Cache** — update `.llm-wiki/ingest-cache.json` with source SHA256
8. **Verify** — read back created pages under `wiki/`

## Constraints

- SHA256 computation failure → fallback to cache miss (never block)
- One source typically updates 5-15 wiki pages — precision over count
- New pages need explicit user approval before creation
- Contradictions with existing wiki content → ask user (no silent newer-wins)
- Never wikilink to `raw/` inside a wiki page (rule 0.3)
- Wikilinks: no `wiki/` prefix. MCP writes: `wiki/projects/...` (never bare `projects/...`)

## Gotchas

- Typical range: 5-15 wiki pages; don't collapse into a single summary
- SHA256 cache prevents re-processing — check before reading source
- User must approve new pages — draft first
- After ingest, vault-index removes the ⏳ Raw Sources line and links result wiki pages
