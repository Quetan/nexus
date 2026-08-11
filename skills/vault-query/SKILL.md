---
name: vault-query
description: >
  Use when answering questions from the Obsidian LLM Wiki — even if the user
  doesn't name the wiki explicitly. Triggers on "спроси вики", "what does the
  wiki say", "look up in vault", "найди в базе знаний". Returns answers with
  full-path wikilink citations. Vault-only by default. Requires mcpvault
  MCP. Do NOT use for MemPalace queries, web searches, codegraph lookups, or
  non-wiki knowledge bases.
---

# vault-query

## What it does

Research the Obsidian LLM Wiki and return structured answers with wikilink
citations. Medium-depth vault research — not a full research workflow.

## Input

- Single question, OR
- Batch: `questions[]` (from research-wiki) — processed sequentially within one
  invocation, one combined structured output (`{status, findings[], gaps[]}`)

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

```markdown
<fact> — [[projects/<project>/Entities/<entity>|<Display>]]
<fact> — [[projects/<project>/Concepts/<concept>|<Display>]]
```

Wikilinks: no `wiki/` prefix. MCP paths: `wiki/...`.

## Conflict Handling

If two vault sources disagree → show both claims with frontmatter dates and ask
user. Do not silently pick newer.

**Rationale:** Within-wiki conflicts need manual resolution — the vault is
authoritative; contradictions mean stale content or legitimate divergence.

## Constraints

- Vault-only by default. External sources only if user explicitly allows.
- If vault has no record for a topic → say so explicitly.
- Propose `vault-write` only when accumulating durable knowledge.
- **When called from nexus workflow:** defer file-back to Phase 7; read only
  sections a plan needs (schema, endpoints, configs, decisions)

## Gotchas

- Vault-only by default — orchestrator may override with allow_external
- If two vault sources disagree → ask user, don't pick newer
- If vault has no record — say so, don't invent
