# Lint Protocol — Wiki Health Check

Wikilinks/Dataview: wiki-semantic (`projects/`, `library/`, …). MCP explicit
paths: `wiki/...`. File I/O: under `wiki/`.

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
- Pages not listed in vault `index.md` or project `00_Index.md`
- Compare actual page files with index entries

### Broken Wikilinks
- `[[links]]` that don't resolve to existing vault files under `wiki/`
- Wikilinks must be wiki-semantic — no `wiki/` prefix, no `../`

### Legacy Wiki Purity (Convention 0.3)
- Scan all Markdown files under `wiki/` for `[[raw/...]]`, `## Raw Sources`,
  `⏳ pending`, and `✅ ingested`.
- Every match is an existing artifact, not a current lifecycle state.
- Report each match as:
  `[LEGACY] Wiki Purity: <path> line <n> — <matched artifact>`
- Severity: WARN. Never report this check as FAIL or ERROR, and do not alter
  pages to remediate it.

### Stale `wiki/` Prefix
- Content wikilinks: `[[projects/...]]` not `[[wiki/projects/...]]` (MCP args may use `wiki/`)
- Scan `.md` for `[[wiki/` and `](wiki/` → `[ERROR] Stale wiki/ prefix in link: <path> line <n>`
- Severity: ERROR

### Misplaced repo-root projects/ (path sanity)
- If `list_directory("projects")` finds paths outside `wiki/projects/`:
  - `[ERROR] Misplaced pages outside wiki/: projects/...`
- Fix: rewrite under `wiki/projects/...`, delete orphan
- Severity: ERROR

### Archived Page Path (Convention 0.4)
- Pages with `status: archived | deprecated` must be physically in `archive/`.
- Find all `.md` files with frontmatter `status: archived` OR `status: deprecated`
- Verify path contains `archive/` at any level
- If not → `[FAIL] Misplaced archived page: <path> (status=<status>)`
- Severity: FAIL

### Missing Hubs (Convention 0.5)
- Entities with at least five backlinks from Concept pages need `00_Index.md`.
- Extract all `[[Entity]]` wikilinks from `library/*/Concepts/**/*.md` and `projects/*/Concepts/**/*.md`
- Count backlinks per Entity (how many different Concept files mention it)
- For each Entity with ≥5 backlinks:
  - Check existence of `00_Index.md`:
    - `entities/<Entity>/00_Index.md` OR
    - `projects/*/Entities/<Entity>/00_Index.md`
  - If missing → `[WARN] Hub needed: [[Entity]] (<N> backlinks, no 00_Index.md)`
- Severity: WARN
- Action: Manual hub creation (not automatic)

### Dataview Coverage (Convention 0.6)
- Every top-level namespace must be covered by a dashboard `FROM` clause.
- Discover top-level namespaces:
  ```bash
  ls -d */ | grep -v -E '(.llm-wiki|.obsidian|.git)'
  ```
  (run inside vault root) Example: `projects/`, `library/`
- Read `index.md`, extract all dataview blocks with `FROM`
- For each namespace, check inclusion in at least one `FROM`
- Report uncovered: `[WARN] Uncovered namespace: <name>/ not in any FROM clause`
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
- Legacy Wiki Purity: <N> — PASS/WARN
- Stale `wiki/` prefix: <N> — PASS/WARN/FAIL
- Misplaced repo-root projects/: <N> — PASS/WARN/FAIL
- Archived page path: <N> — PASS/WARN/FAIL
- Missing hubs: <N> — PASS/WARN/FAIL
- Dataview coverage: <N> — PASS/WARN/FAIL

### Details (WARN/FAIL only)
- <path> — <issue>
```

## Log Update

Append entry to `wiki/log.md` (MCP) using wiki-semantic wikilinks:

```
## [YYYY-MM-DD] HH:MM — lint — <summary>
```
