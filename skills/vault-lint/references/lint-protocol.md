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

### Raw Sources Prefix (Convention 0.3)
- Каждая запись в `## Raw Sources` секции `00_Index.md` обязана иметь префикс `⏳` или `✅`
- Scan all `wiki/projects/*/00_Index.md` → extract `## Raw Sources` section
- For each line starting with `- [[` or `- wiki/`:
  - Check for `⏳ pending` or `✅ ingested` prefix
  - If missing → `[ERROR] Unprefixed Raw Sources entry: <line>`
- Severity: ERROR

### Archived Page Path (Convention 0.4)
- Страницы с `status: archived | deprecated` физически в `/archive/`
- Find all `.md` files with frontmatter `status: archived` OR `status: deprecated`
- Verify path contains `wiki/archive/` OR `wiki/projects/*/archive/`
- If not → `[FAIL] Misplaced archived page: <path> (status=<status>)`
- Severity: FAIL

### Missing Hubs (Convention 0.5)
- Entities с ≥5 backlinks из Concept-страниц нуждаются в `00_Index.md`
- Extract all `[[Entity]]` wikilinks from `wiki/library/*/Concepts/**/*.md`
- Count backlinks per Entity (how many different Concept files mention it)
- For each Entity with ≥5 backlinks:
  - Check existence of `00_Index.md`:
    - `wiki/entities/<Entity>/00_Index.md` OR
    - `wiki/projects/*/entities/<Entity>/00_Index.md`
  - If missing → `[WARN] Hub needed: [[Entity]] (<N> backlinks, no 00_Index.md)`
- Severity: WARN
- Action: Manual hub creation (not automatic)

### Dataview Coverage (Convention 0.6)
- Каждый top-level namespace покрыт хотя бы одним dashboard FROM clause
- Discover top-level namespaces:
  ```bash
  ls -d wiki/*/ | grep -v -E '(.llm-wiki|.obsidian)'
  ```
  Example: `wiki/projects/`, `wiki/library/`, `wiki/entities/`, `wiki/topics/`
- Read `wiki/index.md`, extract all dataview blocks with `FROM`
- For each namespace, check inclusion in at least one `FROM`
- Report uncovered: `[WARN] Uncovered namespace: wiki/<name>/ not in any FROM clause`
- Severity: WARN

## Report Format

```markdown
### Lint Report — <date>
- Orphans: <N> — PASS/WARN/FAIL
- Staleness: <N> — PASS/WARN/FAIL
- Contradictions: <N> — PASS/WARN/FAIL
- Coverage gaps: <N> — PASS/WARN/FAIL
- Index drift: <N> — PASS/WARN/FAIL
- Broken wikilinks: <N> — PASS/WARN/FAIL
- Raw Sources prefix: <N> — PASS/WARN/FAIL
- Archived page path: <N> — PASS/WARN/FAIL
- Missing hubs: <N> — PASS/WARN/FAIL
- Dataview coverage: <N> — PASS/WARN/FAIL

### Details (WARN/FAIL only)
- <path> — <issue>
```

## Log Update

Append entry to wiki/log.md:

```
## [YYYY-MM-DD] HH:MM — lint — <summary>
```
