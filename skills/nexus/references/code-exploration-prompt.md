# Code Exploration Prompt

Load when dispatching Phase 3 (after Phase 2b). Skip for **light**.

## Dispatch

- **Required `subagent_type`:** `explore`
- Do **not** use `generalPurpose`
- Pass **synthesized** vault findings only — max 15 facts, one line each
  (`fact` + `source`). Do not dump raw MCP payloads or full page bodies.

## Prompt

```
Code exploration for nexus. Guided by these vault findings (already synthesized):

{vault_findings_max_15}

Find only plan-relevant current state:
1. Schema (tables, keys, nullable, indexes)
2. Endpoints (routes, methods, middleware)
3. Configs (env vars, ports)
4. Key files (controllers, migrations, routes)

Budget: ≤8 files, ~150 lines each (or the relevant block). If more needed →
status partial and list skipped areas.

Return ONLY JSON:
{"status":"success|partial|failed","findings":[{"fact":"...","source":"code:<path>:<line-or-symbol>"}],"gaps":["..."]}
MAX 15 findings. Empty=success+gaps. No narration.
```
