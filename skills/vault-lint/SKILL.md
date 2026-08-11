---
name: vault-lint
description: >
  Use when checking Obsidian LLM Wiki health — orphan detection, staleness,
  contradictions, coverage gaps, index drift, broken wikilinks. Triggers on
  "check wiki health", "lint vault", "wiki health check", "проверь вики".
  Requires mcpvault MCP. Report only — never creates, edits, or deletes
  pages. Appends summary to vault log.md.
---

# vault-lint

## What it does

Run health checks on the Obsidian LLM Wiki. Reports issues without making
changes. Appends a summary entry to vault `log.md`.

## Checks

| Check | What it detects |
|-------|-----------------|
| Orphans | Pages with 0 backlinks (except intentional stubs) |
| Staleness | Pages not updated > 90 days |
| Contradictions | Pages claiming conflicting facts about the same entity |
| Coverage gaps | Topics mentioned in log but no dedicated page |
| Index drift | Pages not listed in index.md or project 00_Index.md |
| Broken wikilinks | `[[links]]` that don't resolve to existing pages |
| Raw refs in wiki | `[[raw/...]]` outside `## Raw Sources` (convention 0.3) |
| Raw Sources prefix | Missing ⏳ or `✅` pointing at raw (convention 0.3) |
| Stale `wiki/` prefix | `[[wiki/...]]` in **content** (not MCP tool args) |
| Misplaced repo-root projects/ | Bare `projects/` at MCP root outside `wiki/` |
| Archived page path | archived/deprecated outside /archive/ (convention 0.4) |
| Missing hubs | Entities with ≥5 backlinks, no 00_Index (convention 0.5) |
| Dataview coverage | Namespaces not in index.md FROM (convention 0.6) |

## References

Load when needed:
- [Lint protocol](references/lint-protocol.md) — check definitions and report format

## Process

1. Run each check sequentially
2. For each check: PASS / WARN / FAIL
3. On WARN/FAIL: list items with full vault paths
4. Append summary to MCP `wiki/log.md` (wikilinks in entry: no `wiki/` prefix)
5. Present report to user

## Output

```markdown
### Lint Report
- Orphans: <N> PASS/WARN/FAIL
- Staleness: <N> PASS/WARN/FAIL
- Contradictions: <N> PASS/WARN/FAIL
- Coverage gaps: <N> PASS/WARN/FAIL
- Index drift: <N> PASS/WARN/FAIL
- Broken wikilinks: <N> PASS/WARN/FAIL
- Raw refs in wiki: <N> PASS/WARN/FAIL
- Raw Sources prefix: <N> PASS/WARN/FAIL
- Stale `wiki/` prefix: <N> PASS/WARN/FAIL
- Misplaced repo-root projects/: <N> PASS/WARN/FAIL
- Archived page path: <N> PASS/WARN/FAIL
- Missing hubs: <N> PASS/WARN/FAIL
- Dataview coverage: <N> PASS/WARN/FAIL
```

## Constraints

- Report only — never create, edit, or delete pages
- Only append to `wiki/log.md` (no other writes)
- If mcpvault is down → report as unable to check
