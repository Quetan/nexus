# Subagent Prompts — Phase 2a Vault Research

Load **only** when dispatching medium/full Phase 2a (not for light inline).

## Dispatch rules

| Track | Prefer `subagent_type` | Fallback | Forbidden |
| --- | --- | --- | --- |
| A — MemPalace | `nexus-mempalace` (`.cursor/agents/` or `~/.cursor/agents/`) | `generalPurpose` | `explore` |
| B — Wiki | `nexus-wiki` | `generalPurpose` | `explore` |

- Launch **two** Task calls in **one** parent message.
- One Task per track; batch all entities inside that Task.
- Custom agents already know the skill contract — send **scope payload only**
  (below). Do not paste the full skill body into the Task prompt.
- Fallback `generalPurpose`: send the compact prompt template for that track.
- Never use `explore` for MCP research.

## Scope payload (custom agents)

**A — nexus-mempalace**

```
wing: {project}
entities: [...]
repos: [...]
rooms: [...]
allow_external: {true|false}   # record only; MemPalace-only track
```

**B — nexus-wiki**

```
project: {project}
entities: [...]
topics: [...]
ticket_ids: [...]
allow_external: {true|false}
mcp-mode: repo-bound
```

## Compact prompts (generalPurpose fallback only)

### A — MemPalace

```
MemPalace research track. Batch all entities in this one run. No code.

wing: {project}; entities: [...]; repos: [...]; rooms: [...]

For each entity: mempalace_search(limit=3) + mempalace_kg_query (max 5 relations).
Topic searches: room searches limit=5.

Return ONLY JSON:
{"status":"success|partial|failed","findings":[{"fact":"...","source":"drawer:<id>","date":"YYYY-MM-DD"}],"gaps":["..."]}
MAX 10 plan-relevant findings. Empty=success+gaps. MCP down=failed.
```

### B — Wiki

```
Wiki research via vault-query (batch once). No code. Vault-only unless allow_external=true.
mcp-mode: repo-bound (wiki/ MCP prefix).

project: {project}; entities: [...]; topics: [...]; ticket_ids: [...]; allow_external: {true|false}

1) Read index.md  2) vault-query once for full question list  3) backlinks if needed

Return ONLY JSON:
{"status":"success|partial|failed","findings":[{"fact":"...","source":"wiki:<path>","date":"YYYY-MM-DD"}],"gaps":["..."]}
MAX 10. Within-wiki conflict: both dated claims + gap (no winner). Empty=success+gaps. MCP down=failed.
```

Phase 3: load `code-exploration-prompt.md`; required type `explore`.
