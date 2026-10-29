---
name: vault-query
compatibility: mcpvault MCP, or an existing local vault with requester consent.
description: >
  Use when answering questions from the Obsidian LLM Wiki — even if the user
  doesn't name the wiki explicitly. Triggers on "спроси вики", "what does the
  wiki say", "look up in vault", "найди в базе знаний". Returns answers with
  full-path wikilink citations. Vault-only by default. Do NOT use for MemPalace
  queries, web searches, codegraph lookups, or non-wiki knowledge bases.
---

# vault-query

## Vault access

Reuse `vault_access` (mode, absolute wiki root, consent); otherwise use MCP without pings.
On missing tools or connection failure, read the bundled [local fallback](references/vault-access.md)
once per agent context. Inherited `local` reuses its approved root; `skip` records
the reason and skips vault operations. Under nexus, the parent owns local consent.

## What it does

Research the Obsidian LLM Wiki and return structured answers with wikilink
citations. Medium-depth vault research — not a full research workflow.

## Input

- Single question, OR
- Batch: `questions[]` (from research-wiki) plus `allow_external` — processed
  sequentially in one invocation and returned as one structured result
- Optional `vault_access` — shared task access context; forward it to nested calls

## References

Load when needed:
- [Query protocol](references/query-protocol.md) — structured findings output and conflict gate

## Process

1. **Resolve project scope** — auto (cwd/git/manifest) or ask if ambiguous
2. Read `wiki/index.md` through the selected backend → list candidate pages
3. For each key term: MCP `search_notes(query, limit=10)` or approved local search
4. For 2-5 central pages: MCP `search_notes(query="<title>", searchContent=true)`
   or, in approved local mode, search `[[<basename>]]` under the local wiki root
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

Cap batch findings at 10. Completed local or MCP coverage is `success`; limited
coverage is `partial`; wholly unavailable/skipped access is `failed` with the
reason in `gaps`. Empty results are `success` with topic gaps. Record local use
and known freshness limitations in `gaps`. For approval-pending delegated work,
follow the parent handoff in the shared vault access protocol.
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
