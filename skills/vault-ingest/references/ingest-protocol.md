# Ingest Protocol — Source → Wiki Pages

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

## Page Type Mapping

| Source content → | Wiki page type |
|------------------|---------------|
| Service, tool, person, team | Entity |
| Architecture, pattern, workflow | Concept |
| Decision with rationale | Decision |
| Action item, tracked work | Task |
| Cross-cutting grouping | Topic |

## Cache Format

```json
{
  "raw/DD.MM.YYYY/<project>/<slug>.md": "sha256hex",
  ...
}
```
