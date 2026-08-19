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

## Output format

For a standalone question, answer immediately in concise requester-locale
Markdown; read operations do not need draft approval.

For a batch call from `research-wiki`, return this JSON contract:

```json
{
  "status": "success|partial|failed",
  "findings": [
    {"fact": "<plan-relevant fact>", "source": "wiki:<path>", "date": "YYYY-MM-DD"}
  ],
  "gaps": ["<topic>"]
}
```

- Process all `questions[]` in one invocation; batch setup does not reduce
  per-topic depth.
- Cap `findings[]` at 10. Use `failed` only when the vault is unavailable and
  `partial` only when coverage is limited. Empty results are `success` with gaps.
- Preserve both dated claims for a within-wiki conflict and add a gap requesting
  requester resolution. Do not emit a selected winner.
- Wikilinks omit `wiki/`; MCP paths use `wiki/...`.

## mcpvault Fallback

If mcpvault is down: file I/O under `~/Development/obsidian-vault/wiki/`. Note fallback in output.

## Source gate and conflict gate

If two vault sources disagree — do not pick a winner. Show both claims with dates and ask the user.

Remain vault-only unless the requester explicitly supplied `allow_external: true`.
That gate permits supplementary external evidence but never authorizes silently
resolving a within-wiki conflict.
