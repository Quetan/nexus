# Subagent Prompts

Load this file when dispatching parallel sub-agents during Phase 2a.

## Subagent A — research-mempalace

Prompt template:

```
You are Subagent A — MemPalace research. Your task:

Task scope: {entities, repos, ticket IDs}
Palace rooms: {rooms}
Query budget: {N} queries

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
Query budget: {N} queries
Project: {project}

1. Read wiki/index.md
2. Call vault-query for each entity/topic
3. Collect findings with wikilink citations

Return: Markdown list of findings with [[wikilink]] sources.

Rules: Vault-only. No code. Record gaps.
```

## Subagent C — Code Exploration

Prompt template:

```
You are Subagent C — Code exploration. Guided by these vault findings:

{vault_findings}

Explore the codebase for:
1. Schema (tables, keys, nullable, indexes)
2. Endpoints (routes, methods, middleware)
3. Configs (env vars, ports)
4. Key files (controllers, migrations, routes)

Return: актуальное состояние for the research brief.
```
