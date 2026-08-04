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

Return contract:
- Plain bullet list, MAX 10 bullets, one fact per bullet + source (drawer id / date).
- Only facts that affect an implementation plan (schema, endpoints, configs, decisions).
- No prose, no narration. A "not found" entry only if it blocks a plan step.
```

Rules: No code exploration. No inventing facts. Record gaps internally.

## Subagent B — research-wiki → vault-query

Prompt template:

```
You are Subagent B — Wiki research. Your task:

Task scope: {entities, topics, ticket IDs}
Project: {project}

1. Read vault index.md
2. Call vault-query for each entity/topic
3. Collect findings with wikilink citations

Return contract:
- Plain bullet list, MAX 10 bullets, one fact per bullet + [[wikilink]] source.
- Only facts that affect an implementation plan (schema, endpoints, configs, decisions).
- No prose, no narration. A "not found" entry only if it blocks a plan step.
```

Rules: Vault-only. No code. Record gaps internally.

**Note:** For Phase 3 code exploration, load `code-exploration-prompt.md` separately after Phase 2b synthesis.
