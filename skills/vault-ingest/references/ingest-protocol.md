# Ingest Protocol — Manual Source → Wiki Pages

## Scope

This protocol handles **manual/external source notes** that live in `raw/` —
files dropped by the user, external dumps, ad-hoc captures. New session work
never enters `raw/` — `vault-write` writes directly into `wiki/`.

## Pipeline

1. **Cache check** — compare SHA256 of source vs `.llm-wiki/ingest-cache.json`
   - Same → skip (no re-process)
   - Different → proceed
2. **Decompose** — extract from source:
   - Entities (services, people, tools)
   - Concepts (patterns, architectures)
   - Decisions (architectural choices with rationale)
   - Data points (config values, ports, endpoints)
   - Relationships (entity A uses entity B)
3. **Overlap scan** — search wiki for matching pages
4. **Propose** — list:
   - New pages to create
   - Existing pages to update
   - Contradictions to flag
5. **User approval** — new pages need explicit `save`
   - Contradictions: show both claims with dates, ask
6. **Execute** — create/update pages → vault-index → update cache
   - Vault-index removes the ⏳ line from `## Raw Sources` and adds
     ✅ wikilinks to the resulting wiki pages under their typed section
     (Entities / Concepts / Decisions / Tasks / Topics).
   - Vault-index never links back to the raw file from a wiki page.

## Page Type Mapping

| Source content → | Wiki page type |
|------------------|---------------|
| Service, tool, person, team | Entity |
| Architecture, pattern, workflow | Concept |
| Decision with rationale | Decision |
| Action item, tracked work | Task |
| Cross-cutting grouping | Topic |

## Linking Rules

- Wikilinks: `[[projects/<p>/Entities/foo]]` (no `wiki/` prefix in content)
- Wiki pages never link to `raw/`
- MCP: read `raw/...`; write `wiki/projects/...`

## Cache Format

```json
{
  "raw/DD.MM.YYYY/<project>/<slug>.md": "sha256hex",
  ...
}
```

## After Ingest — Raw Source Lifecycle

The source file stays on disk in `raw/DD.MM.YYYY/<project>/<slug>.md` as
provenance only. The wiki does not link back to it. Periodic archiving of
`raw/` is out of scope for this skill.
