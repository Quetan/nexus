# Query Protocol — Medium Vault Research

## Flow

- Use a known page path directly. For a known project or library topic,
  restrict search to its subtree. For cross-project tickets, search
  `projects/` and preserve distinct matches from each project.
- MCP: `search_notes(query, pathPrefix, limit=10, searchFrontmatter=true)`.
  Use the active binding (`wiki/projects/…` or `projects/…`). Local: check
  filenames and bounded content matches under the equivalent directory.
- Prefer exact ticket IDs (case insensitive) and matches in note paths
  or titles. Use index and log hits as navigation clues. If coverage is
  thin, widen the subtree or try an alternate term once. A profile index
  can supply names or synonyms when search is inconclusive.
- For additional evidence, search references to the selected title or
  basename as needed. Keep the main skill's page budget.

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
