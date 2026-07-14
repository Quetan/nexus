---
name: vault-lint
description: >
  Use when checking Obsidian LLM Wiki health — orphan detection, staleness,
  contradictions, coverage gaps, index drift, broken wikilinks. Triggers on
  "check wiki health", "lint vault", "wiki health check", "проверь вики".
  Requires user-seekstone MCP. Report only — never creates, edits, or deletes
  pages. Appends summary to wiki/log.md.
---

# vault-lint

## What it does

Run health checks on the Obsidian LLM Wiki. Reports issues without making
changes. Appends a summary entry to `wiki/log.md`.

## Checks

| Check | What it detects |
|-------|-----------------|
| Orphans | Pages with 0 backlinks (except intentional stubs) |
| Staleness | Pages not updated > 90 days |
| Contradictions | Pages claiming conflicting facts about the same entity |
| Coverage gaps | Topics mentioned in log but no dedicated page |
| Index drift | Pages not listed in wiki/index.md or project 00_Index.md |
| Broken wikilinks | `[[links]]` that don't resolve to existing pages |

## Process

1. Run each check sequentially
2. For each check: PASS (no issues), WARN (minor issues), or FAIL (blocking)
3. On WARN/FAIL: list items with full vault paths
4. Append summary to `wiki/log.md`:

```
## [YYYY-MM-DD] HH:MM — lint — <summary>
```

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
```

## Constraints

- Report only — never create, edit, or delete pages
- Only append to wiki/log.md (no other writes)
- If seekstone is down → report as unable to check
