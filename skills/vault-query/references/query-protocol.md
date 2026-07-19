# Query Protocol — Medium Vault Research

## Flow

1. Resolve project scope (auto or ask)
2. Read `wiki/index.md` → list candidate pages
3. For each key term: `seekstone_search(query, limit=10)`
4. For 2-5 central pages: `seekstone_get_backlinks(path)`
5. Adaptively read relevant notes (entities → concepts → decisions → sources)
6. Synthesize answer with wikilink citations

## Output Format

Default: answer immediately (no draft approval for reads).

Optional structured findings when requested:

```markdown
### Findings
- <fact> — [[wiki/projects/<project>/...]]

### Prior Decisions
- <decision> — [[wiki/projects/<project>/Decisions/...]]

### Gaps
- Topic "<topic>" — vault has no record

### Conflicts
- <claim A> (date) vs <claim B> (date) — awaiting user
```

## Seekstone Fallback

If seekstone is down: use file I/O fallback. Read `.md` files directly from the vault directory. Note the fallback in output.

## Conflict Gate

If two vault sources disagree — do not pick a winner. Show both claims with dates and ask the user.
