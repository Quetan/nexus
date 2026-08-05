# Subagent Prompts — Phase 2a Vault Research

Load this file when dispatching parallel sub-agents during **Phase 2a** (vault research only, no code).

## Subagent A — research-mempalace

Prompt template:

```
You are Subagent A — MemPalace research. Your task:

Task scope: {entities: [...], repos: [...]}   # full list, not one dispatch per entity
Palace rooms: {rooms}

Dispatch ONE subagent per track — this subagent iterates over the full list internally.

1. Search MemPalace: mempalace_search for each entity/room
2. Query KG: mempalace_kg_query for each entity
3. Collect findings with drawer IDs and dates

Return contract (JSON):
- {"status": "success"|"partial"|"failed", "findings": [{"fact", "source": "drawer:<id>", "date"}], "gaps": ["topic"]}
- MAX 10 findings, only facts that affect an implementation plan (schema, endpoints, configs, decisions).
- status "failed" only if the source itself is unavailable; an empty result is status "success" with topics in gaps.
```

Rules: No code exploration. No inventing facts. Record gaps internally.

## Subagent B — research-wiki → vault-query

Prompt template:

```
You are Subagent B — Wiki research. Your task:

Task scope: {entities: [...], topics: [...], ticket IDs}   # full list, not one dispatch per entity
Project: {project}

Dispatch ONE subagent per track — this subagent iterates over the full list internally.

1. Read vault index.md
2. Call vault-query once (batch mode) with the full question list
3. Collect findings with wikilink citations

Return contract (JSON):
- {"status": "success"|"partial"|"failed", "findings": [{"fact", "source": "wiki:<path>", "date"}], "gaps": ["topic"]}
- MAX 10 findings, only facts that affect an implementation plan (schema, endpoints, configs, decisions).
- status "failed" only if the source itself is unavailable; an empty result is status "success" with topics in gaps.
```

Rules: Vault-only. No code. Record gaps internally.

**Note:** For Phase 3 code exploration, load `code-exploration-prompt.md` separately after Phase 2b synthesis.
