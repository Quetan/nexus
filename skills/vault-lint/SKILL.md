---
name: vault-lint
compatibility: mcpvault MCP, or an existing local vault with requester consent.
description: >
  Use when checking Obsidian LLM Wiki health — orphan detection, staleness,
  contradictions, coverage gaps, index drift, broken wikilinks. Triggers on
  "check wiki health", "lint vault", "wiki health check", "проверь вики".
  Report only — never creates, edits, or deletes pages. Appends summary to
  vault log.md.
---

# vault-lint

## Vault access

Reuse `vault_access` (mode, absolute wiki root, consent); otherwise use MCP without pings.
On missing tools or connection failure, read the bundled [local fallback](references/vault-access.md)
once per agent context. Inherited `local` reuses its approved root; `skip` records
the reason and skips vault operations. Under nexus, the parent owns local consent.

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
| Legacy Wiki Purity | Existing external-staging links, `Raw Sources` headings, or ingest-status markers |
| Stale `wiki/` prefix | `[[wiki/...]]` in **content** (not MCP tool args) |
| Misplaced repo-root projects/ | Bare `projects/` at MCP root outside `wiki/` |
| Archived page path | archived/deprecated outside /archive/ (convention 0.4) |
| Missing hubs | Entities with ≥5 backlinks, no 00_Index (convention 0.5) |
| Dataview coverage | Namespaces not in index.md FROM (convention 0.6) |

## References

Load when needed:
- [Lint protocol](references/lint-protocol.md) — check definitions and report format

## Process

1. Run each check sequentially through the selected backend
2. For each check: PASS / WARN / FAIL
3. On WARN/FAIL: list items with full vault paths
4. Append summary to `wiki/log.md` in MCP mode or `<local_root>/log.md` in
   approved local mode; read back to verify. Wikilinks omit the `wiki/` prefix.
5. Present report to user

On an uncertain log write, load bundled [write recovery](references/vault-access.md#write-recovery).

## Output

```markdown
### Lint Report
- Orphans: <N> PASS/WARN/FAIL
- Staleness: <N> PASS/WARN/FAIL
- Contradictions: <N> PASS/WARN/FAIL
- Coverage gaps: <N> PASS/WARN/FAIL
- Index drift: <N> PASS/WARN/FAIL
- Broken wikilinks: <N> PASS/WARN/FAIL
- Legacy Wiki Purity: <N> PASS/WARN
- Stale `wiki/` prefix: <N> PASS/WARN/FAIL
- Misplaced repo-root projects/: <N> PASS/WARN/FAIL
- Archived page path: <N> PASS/WARN/FAIL
- Missing hubs: <N> PASS/WARN/FAIL
- Dataview coverage: <N> PASS/WARN/FAIL
```

## Constraints

- Report only — never create, edit, or delete pages
- Only append to `wiki/log.md` (no other writes)
- If access is skipped, report checks as not performed with the reason; do not
  fabricate PASS results or attempt a log write. If only logging fails, retain
  the completed check report and explicitly report the unverified log update.
- Do not turn legacy Wiki Purity findings into failures or propose automatic
  migrations.
