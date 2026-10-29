# Query Protocol — Medium Vault Research

## Flow

1. Resolve project scope (auto or ask)
2. Read vault index via the selected backend → list candidate pages
3. For each key term: MCP `search_notes(query, limit=10)` or approved local search
4. For 2-5 central pages: MCP title/content search or, in approved local
   mode, search `[[<basename>]]` under the selected local wiki root
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
- Cap `findings[]` at 10. Completed MCP/local coverage is `success`; limited
  coverage is `partial`; wholly unavailable/skipped access is `failed` with a
  reason in `gaps`. Empty results are `success` with topic gaps.
- Preserve both dated claims for a within-wiki conflict and add a gap requesting
  requester resolution. Do not emit a selected winner.
- Wikilinks omit `wiki/`; MCP paths use `wiki/...`.

## mcpvault Fallback

On MCP unavailability, use the bundled fallback linked from this skill's `SKILL.md`.
Forward `vault_access`; obtain task-scoped consent before local content access.
Under nexus, return approval needs to the parent, which asks and resumes the
track. Record local use/known limitations in `gaps` for JSON results, or briefly
in standalone prose. Reuse access decisions on subsequent calls.

## Source gate and conflict gate

If two vault sources disagree — do not pick a winner. Show both claims with dates and ask the user.

Remain vault-only unless the requester explicitly supplied `allow_external: true`.
That gate permits supplementary external evidence but never authorizes silently
resolving a within-wiki conflict.
