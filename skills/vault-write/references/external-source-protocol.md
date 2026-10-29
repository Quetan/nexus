# External-Source Protocol — Direct Curation to Wiki

## Scope

Use this protocol for pasted notes, uploaded files, URLs, exports, and material
from another session. Curate the material directly into typed pages under
`wiki/`; do not stage, link, or queue source files in the vault.

## Pipeline

1. **Identify provenance** — record the source kind and any stable URL, title,
   author, or retrieval date available in the material.
2. **Extract claims** — distinguish direct source statements from inferences.
   Tag source-backed claims `user:` when supplied by the requester; otherwise
   use `UNVERIFIED` until corroborated. Tag wiki-derived claims `vault:` with a
   wikilink. Use `tool:` only for directly observed session actions or results
   under the [write protocol](write-protocol.md); retrieving an external claim
   with a tool does not by itself corroborate that claim.
3. **Decompose** — identify entities, concepts, decisions, tasks, data points,
   and relationships. Do not create a `type: source` page.
4. **Search for overlap** — find matching wiki pages before proposing changes.
   Update a living page only when the evidence is compatible. For a frozen page,
   show the diff and require explicit approval. Show conflicting wiki claims to
   the user; do not silently choose a newer source.
5. **Propose a change package** — list new pages, updates, skipped material,
   and contradictions. Use requester-localized prose for the proposal and
   drafts; ask once if their language is unclear.
6. **Wait for approval** — require explicit `save` for every new page and
   proposed update.
7. **Write and verify** — follow the [save flow](write-protocol.md#save-flow)
   with the selected `vault_access` backend. Read back changed pages and use
   `vault-index`'s verification summary for indexes/log; do not reread verified entries.
   If session work is included, apply the session capture sections in the write
   protocol. Reply with status and verified page links only.

## Page Type Mapping

| Material | Wiki page type |
|----------|----------------|
| Service, person, tool, team | Entity |
| Architecture, pattern, workflow | Concept |
| Choice with rationale | Decision |
| Actionable work item | Task |
| Stable cross-cutting grouping | Topic |

## Evidence and Purity

- Keep the usable provenance in each page's `## Sources` table.
- Do not include `[[raw/...]]`, a `## Raw Sources` section, `⏳ pending`,
  `✅ ingested`, or other ingest-status markers in a new draft.
- A source URL may be plain text in the evidence column when it is useful; it
  is not a wiki page link.
- Preserve `UNVERIFIED` claims as tentative or omit them from factual prose.
