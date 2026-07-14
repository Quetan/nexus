---
name: vault-query
description: >
  Use when answering questions from the Obsidian LLM Wiki. Triggers on "спроси
  вики", "what does the wiki say", "look up in vault", "найди в базе знаний".
  Returns answers with full-path wikilink citations. Vault-only by default.
  Requires user-seekstone MCP. Do NOT use for MemPalace queries, web searches,
  codegraph lookups, or non-wiki knowledge bases.
---

# vault-query

## What it does

Research the Obsidian LLM Wiki and return structured answers with wikilink
citations. Medium-depth vault research — not a full research workflow.

## Process

1. **Resolve project scope** — auto (cwd/git/manifest) or ask if ambiguous
2. Read `wiki/index.md` → list candidate pages
3. For each key term: `seekstone_search(query, limit=10)`
4. For 2-5 central pages: `seekstone_get_backlinks(path)`
5. Adaptively read relevant notes (entities > concepts > decisions > sources)
6. Synthesize answer with full-path wikilink citations

## Output

Answer with inline citations:
```markdown
<fact> — [[projects/<project>/Entities/<entity>|<Display>]]
<fact> — [[projects/<project>/Concepts/<concept>|<Display>]]
```

## Conflict Handling

If two vault sources disagree → show both claims with frontmatter dates and ask
user. Do not silently pick newer.

## Constraints

- Vault-only by default. External sources only if user explicitly allows.
- If vault has no record for a topic → say so explicitly.
- Propose `vault-write` only when accumulating durable knowledge.
