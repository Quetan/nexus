# Query Protocol — Medium Vault Research

## Flow

1. Resolve project scope (auto or ask)
2. Read vault `wiki/index.md` (MCP path) → list candidate pages
3. For each key term: `search_notes(query, limit=10)`
4. For 2-5 central pages: `search_notes(query="<title>", searchContent=true)` or file I/O grep for `[[<basename>]]`
5. Adaptively read relevant notes (entities → concepts → decisions → sources)
   - Read at most 5 pages in full
   - Additional matches: title + first paragraph only
   - Need more depth? Note it as a gap and ask the user whether to expand budget
6. Synthesize answer with wikilink citations

## Output Format

Default: answer immediately (no draft approval for reads).

Optional structured findings when requested:

```markdown
### Findings
- <fact> — [[projects/<project>/Entities/<name>]]

### Prior Decisions
- <decision> — [[projects/<project>/Decisions/<slug>]]

### Gaps
- Topic "<topic>" — vault has no record

### Conflicts
- <claim A> (date) vs <claim B> (date) — awaiting user
```

Wikilinks: no `wiki/` prefix in citations. MCP paths: `wiki/...`.

## mcpvault Fallback

If mcpvault is down: file I/O under `~/Development/obsidian-vault/wiki/`. Note fallback in output.

## Conflict Gate

If two vault sources disagree — do not pick a winner. Show both claims with dates and ask the user.
