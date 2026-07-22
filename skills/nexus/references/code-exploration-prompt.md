# Code Exploration Prompt

Load this file when dispatching code exploration subagent during **Phase 3** (after Phase 2b synthesis).

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
