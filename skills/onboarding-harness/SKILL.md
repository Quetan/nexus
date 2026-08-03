---
name: onboarding-harness
description: >
  Use at the start of any nexus research workflow to verify MCP availability and
  project manifest — even if the workflow was already run this session. Called
  by nexus during Phase 0. Standalone use: before any multi-step agent workflow
  that depends on MemPalace or mcpvault MCP. Do NOT use for simple single-command
  queries.
---

# onboarding-harness

## What it does

Verify that required MCP servers are available and the project manifest exists
in MemPalace. Reports pass/fail status per check.

## Process

1. `mempalace_status` — verify MemPalace MCP is available
2. `get_vault_stats` — verify mcpvault MCP is available
3. Determine project scope:
   - Check `mempalace_search(query="ONB manifest", wing=<project>, room="documentation", limit=1)`
   - Fallback: git remote or cwd
4. If manifest found: extract project rooms and manifest searches
5. Report:

```
Onboarding harness: mempalace ✓ | mcpvault ✓ | manifest ✓
```

(or name failed steps)

## Output

```markdown
### Onboarding Results
- mempalace: ✓ / ✗
- mcpvault: ✓ / ✗
- manifest: ✓ / ✗ (project: <name>)
- rooms: [room1, room2, ...]
```

## On MCP Failure

If mempalace or mcpvault is unavailable:
- Record the gap
- Continue with available sources
- Note limitations in research brief

## Gotchas

- MCP failure does not block the workflow — continue with gaps
- Manifest may be stale; cross-check with actual project structure
