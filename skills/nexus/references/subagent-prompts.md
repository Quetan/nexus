# Subagent Prompts — Phase 2a Vault Research

Load this file when dispatching parallel sub-agents during **Phase 2a** (vault research only, no code).

## Subagent A — research-mempalace

Prompt template:

```
You are Subagent A — MemPalace research. Your task:

Task scope: {entities, repos, ticket IDs}
Palace rooms: {rooms}

1. Search MemPalace: mempalace_search for each entity/room
2. Query KG: mempalace_kg_query for each entity
3. Collect findings with drawer IDs and dates

Return: Markdown list of findings with sources.

Rules: No code exploration. No inventing facts. Record gaps.
```

## Subagent B — research-wiki → vault-query

Prompt template:

```
You are Subagent B — Wiki research. Your task:

Task scope: {entities, topics, ticket IDs}
Project: {project}

1. Read vault index.md
2. Call vault-query for each entity/topic
3. Collect findings with wikilink citations

Return: Markdown list of findings with [[wikilink]] sources.

Rules: Vault-only. No code. Record gaps.
```

**Note:** For Phase 3 code exploration, load `code-exploration-prompt.md` separately after Phase 2b synthesis.
