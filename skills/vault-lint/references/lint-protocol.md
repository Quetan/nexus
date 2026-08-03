# Lint Protocol — Wiki Health Check

Read this file when running health checks. All paths are vault-root-relative
— `projects/`, `library/`, `index.md`, `log.md`. Never prefix wikilinks or
grep patterns with `wiki/` — vault root IS what was formerly `wiki/`.

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
- `[[links]]` that don't resolve to existing vault files
- Scan all `.md` files for wikilinks, verify targets exist
- Wikilinks must be vault-root-relative — no `wiki/` prefix, no `../`

### Raw Refs in Wiki (Convention 0.3, body part)
- Wiki pages never reference `raw/` files in body, Related, or Sources sections
- Scan all `.md` under vault root **excluding** lines inside `## Raw Sources` sections of `00_Index.md` files
- For each match of `[[raw/...]]` outside that context:
  - `[ERROR] Page references raw: <path> line <n>`
- Severity: ERROR
- Fix: remove the wikilink; if evidence needed, cite claim with `user:`/`vault:`/`UNVERIFIED` tag per evidence rules

### Raw Sources Prefix (Convention 0.3, queue part)
- Every line in the `## Raw Sources` section of `00_Index.md` must start with `⏳ pending` AND point at `raw/`, OR be a summary line of the form `- ingested → [[<wiki-page>]], ...` after the raw has been consumed
- Scan all `projects/*/00_Index.md` → extract `## Raw Sources` section
- For each line:
  - If it links to `raw/` without `⏳ pending` → `[ERROR] Unprefixed Raw Sources entry: <line>`
  - If it has `✅ ingested` AND links to `raw/` → `[ERROR] 'ingested' marker on raw link: <line> — replace with wiki links`
  - If it has no emoji prefix at all → `[ERROR] Unprefixed Raw Sources entry: <line>`
- Severity: ERROR

### Stale `wiki/` Prefix
- Wikilinks must be vault-root-relative — `[[projects/...]]` not `[[wiki/projects/...]]`
- Scan all `.md` files for `[[wiki/` and for markdown links of form `](wiki/`
- Each match → `[ERROR] Stale wiki/ prefix in link: <path> line <n>`
- Severity: ERROR

### Archived Page Path (Convention 0.4)
- Страницы с `status: archived | deprecated` физически в `/archive/`
- Find all `.md` files with frontmatter `status: archived` OR `status: deprecated`
- Verify path contains `archive/` at any level
- If not → `[FAIL] Misplaced archived page: <path> (status=<status>)`
- Severity: FAIL

### Missing Hubs (Convention 0.5)
- Entities с ≥5 backlinks из Concept-страниц нуждаются в `00_Index.md`
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
- Каждый top-level namespace покрыт хотя бы одним dashboard FROM clause
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
- Raw refs in wiki: <N> — PASS/WARN/FAIL
- Raw Sources prefix: <N> — PASS/WARN/FAIL
- Stale `wiki/` prefix: <N> — PASS/WARN/FAIL
- Archived page path: <N> — PASS/WARN/FAIL
- Missing hubs: <N> — PASS/WARN/FAIL
- Dataview coverage: <N> — PASS/WARN/FAIL

### Details (WARN/FAIL only)
- <path> — <issue>
```

## Log Update

Append entry to `log.md` using vault-root-relative wikilinks:

```
## [YYYY-MM-DD] HH:MM — lint — <summary>
```
