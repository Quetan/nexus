---
name: research-wiki
description: >
  Use when extracting context from Obsidian LLM Wiki for a research task. Called
  by nexus during Phase 2a vault research. Delegates actual wiki queries to
  vault-query. Requires: vault-query, mcpvault MCP.
  Standalone use: when you need to look up project knowledge from the wiki
  without the full research flow. Do NOT use for MemPalace queries (use
  research-mempalace) or code exploration.
---

# research-wiki

## What it does

Extract project context from Obsidian LLM Wiki by delegating to `vault-query`.
Returns wikilink-cited findings for the research task.

## Input

- **Task scope**: entities, topics, ticket IDs

## Process

1. Read vault `index.md` → list candidate pages
2. For each entity/topic in task scope:
   - Call `vault-query` with the specific research question
   - Collect findings with full-path wikilink citations
3. For architectural/cross-cutting topics:
   - Follow backlinks from central pages (`search_notes(query="<title>", searchContent=true)` or file I/O grep for `[[<basename>]]`)
   - Read relevant sections adaptively

## Output — plan-relevant facts only

```markdown
### Wiki Findings
- <fact> — [[projects/<project>/Entities/<entity>|<Display>]]
- <decision> — [[projects/<project>/Decisions/<slug>|<Display>]]
```

- Hard cap: **10 bullets**. Only facts that affect the implementation plan
  (schema, endpoints, configs, decisions).
- No prose, no narration. A "not found" entry only if it blocks a plan step.

## Constraints

- Vault-only by default. External sources (web, codegraph) only if user explicitly allows.
- Conflicts found in wiki → pass to `research-synthesize` with both dates.
- If topic has no record, note it internally as a gap.

## Gotchas

- vault-query is vault-only by default — don't supplement with web/codegraph unless allowed
- index.md may be incomplete after many ingests — also search directly
