# Query Protocol — Medium Vault Research

## Flow

1. Resolve project scope (auto or ask)
2. Read vault `index.md` → list candidate pages
3. For each key term: `search_notes(query, limit=10)`
4. For 2-5 central pages: `search_notes(query="<title>", searchContent=true)` or file I/O grep for `[[<basename>]]`
5. Adaptively read relevant notes (entities → concepts → decisions → sources)
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

Wikilinks are vault-root-relative — never use the `wiki/` prefix.

## mcpvault Fallback

If mcpvault is down: use file I/O fallback. Read `.md` files directly from the vault directory. Note the fallback in output.

## Conflict Gate

If two vault sources disagree — do not pick a winner. Show both claims with dates and ask the user.
