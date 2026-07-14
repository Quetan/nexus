# Lint Protocol — Wiki Health Check

## Checks

### Orphans
- Pages with 0 backlinks
- Exception: intentional stubs (e.g., `00_Index.md`)
- Method: check backlinks for each page in project/ dirs

### Staleness
- Pages not updated > 90 days from `date` in frontmatter
- Mark as WARN (not FAIL — old pages may still be correct)

### Contradictions
- Two pages claiming conflicting facts about the same entity
- Requires semantic comparison — flag pairs for manual review

### Coverage Gaps
- Topics mentioned in log.md but no dedicated wiki page
- Scan log.md for topics, cross-reference with projects/

### Index Drift
- Pages not listed in wiki/index.md or project 00_Index.md
- Compare actual page files with index entries

### Broken Wikilinks
- `[[links]]` that don't resolve to existing vault files
- Scan all `.md` files for wikilinks, verify targets exist

## Report Format

```markdown
### Lint Report — <date>
- Orphans: <N> — PASS/WARN/FAIL
- Staleness: <N> — PASS/WARN/FAIL
- Contradictions: <N> — PASS/WARN/FAIL
- Coverage gaps: <N> — PASS/WARN/FAIL
- Index drift: <N> — PASS/WARN/FAIL
- Broken wikilinks: <N> — PASS/WARN/FAIL

### Details (WARN/FAIL only)
- <path> — <issue>
```

## Log Update

Append entry to wiki/log.md:

```
## [YYYY-MM-DD] HH:MM — lint — <summary>
```
