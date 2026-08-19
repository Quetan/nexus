---
name: vault-query
description: >
  Use when answering questions from the Obsidian LLM Wiki — even if the user
  doesn't name the wiki explicitly. Triggers on "спроси вики", "what does the
  wiki say", "look up in vault", "найди в базе знаний". Returns answers with
  full-path wikilink citations. Vault-only by default. Do NOT use for MemPalace
  queries, web searches, codegraph lookups, or non-wiki knowledge bases.
compatibility: mcpvault MCP
---

# vault-query

## What it does

Research the Obsidian LLM Wiki and return structured answers with wikilink
citations. Medium-depth vault research — not a full research workflow.

## Input

- Single question, OR
- Batch: `questions[]` (from research-wiki) plus `allow_external` — processed
  sequentially in one invocation and returned as one structured result

## References

Load when needed:
- [Query protocol](references/query-protocol.md) — structured findings output and conflict gate

## Process

1. **Resolve project scope** — auto (cwd/git/manifest) or ask if ambiguous
2. Read `wiki/index.md` (MCP) → list candidate pages
3. For each key term: `search_notes(query, limit=10)`
4. For 2-5 central pages: `search_notes(query="<title>", searchContent=true)` or file I/O grep for `[[<basename>]]`
5. Adaptively read relevant notes (entities > concepts > decisions > sources)
   - Explicit MCP paths under `wiki/`; read ≤5 pages in full; else title + first paragraph
   - Need more depth? Record a gap and ask whether to expand budget
6. Synthesize answer with full-path wikilink citations
7. **File back** — durable knowledge → propose `vault-write`

## Output

For a standalone question, return concise requester-locale Markdown:

```markdown
<fact> — [[projects/<project>/Entities/<entity>|<Display>]]
<fact> — [[projects/<project>/Concepts/<concept>|<Display>]]
```

Wikilinks: no `wiki/` prefix. MCP paths: `wiki/...`.

For a batch call from `research-wiki`, return JSON only:

```json
{
  "status": "success|partial|failed",
  "findings": [
    {"fact": "<plan-relevant fact>", "source": "wiki:<vault-relative-path>", "date": "YYYY-MM-DD"}
  ],
  "gaps": ["<topic>"]
}
```

Cap batch findings at 10. `failed` means the vault source is unavailable;
`partial` means coverage was limited; an empty result is `success` with gaps.
For a within-wiki conflict, retain both dated claims in `findings[]`, add a gap
that requester resolution is required, and do not select a winner.

## Conflict Handling

If two vault sources disagree → show both claims with frontmatter dates and ask
user. Do not silently pick newer.

**Rationale:** Within-wiki conflicts need manual resolution — the vault is
authoritative; contradictions mean stale content or legitimate divergence.

## Constraints

- Vault-only by default. Consult external sources only when the requester
  explicitly authorized `allow_external: true`; record that gate in batch input.
- If vault has no record for a topic → say so explicitly.
- Propose `vault-write` only when accumulating durable knowledge.
- **When called from nexus workflow:** defer file-back to Phase 7; read only
  sections a plan needs (schema, endpoints, configs, decisions)

## Gotchas

- `allow_external` must be explicitly authorized; it is never inferred
- If two vault sources disagree → ask user, don't pick newer
- If vault has no record — say so, don't invent
