---
name: vault-query
description: >
  Use when answering questions from the Obsidian LLM Wiki — even if the user
  doesn't name the wiki explicitly. Triggers on "спроси вики", "what does the
  wiki say", "look up in vault", "найди в базе знаний". Returns answers with
  full-path wikilink citations. Vault-only by default. Requires seekstone
  MCP. Do NOT use for MemPalace queries, web searches, codegraph lookups, or
  non-wiki knowledge bases.
---

# vault-query

## What it does

Research the Obsidian LLM Wiki and return structured answers with wikilink
citations. Medium-depth vault research — not a full research workflow.

## References

Load when needed:
- Read [references/query-protocol.md] **if the answer requires structured findings output** — format template and conflict gate

## Process

1. **Resolve project scope** — auto (cwd/git/manifest) or ask if ambiguous
2. Read `wiki/index.md` → list candidate pages
3. For each key term: `seekstone_search(query, limit=10)`
4. For 2-5 central pages: `seekstone_get_backlinks(path)`
5. Adaptively read relevant notes (entities > concepts > decisions > sources)
6. Synthesize answer with full-path wikilink citations
7. **File back** — if the answer contains durable knowledge (a comparison, analysis, or connection worth keeping), propose `vault-write` to file it back into the wiki as a new page

## Output

Answer with inline citations:
```markdown
<fact> — [[wiki/projects/<project>/Entities/<entity>|<Display>]]
<fact> — [[wiki/projects/<project>/Concepts/<concept>|<Display>]]
```

## Conflict Handling

If two vault sources disagree → show both claims with frontmatter dates and ask
user. Do not silently pick newer.

**Rationale:** Within-wiki conflicts require manual resolution — the vault is the
authoritative source, and contradictions indicate either stale content or legitimate
divergence that needs human judgment.

## Constraints

- Vault-only by default. External sources only if user explicitly allows.
- If vault has no record for a topic → say so explicitly.
- Propose `vault-write` only when accumulating durable knowledge.
- **When called from nexus workflow:** defer file-back proposal to Phase 7 (wiki capture happens post-plan, with user approval)

## Gotchas

- Vault-only by default — orchestrator may override with allow_external
- If two vault sources disagree → ask user, don't pick newer
- If vault has no record — say so, don't invent
