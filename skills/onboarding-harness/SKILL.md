---
name: onboarding-harness
compatibility: Requires MemPalace and mcpvault MCPs.
description: >
  Use when starting any nexus research workflow to verify MCP availability and
  project manifest — even if the workflow was already run this session (a cached
  result is reused when fresh, see Session Cache). Called
  by nexus during Phase 0. Standalone use: before any multi-step agent workflow
  that depends on MemPalace or mcpvault MCP. Do NOT use for simple single-command
  queries.
---

# onboarding-harness

## What it does

Verify MemPalace + mcpvault availability and the project manifest. Detect whether
mcpvault is bound to the vault **repo** root or directly to `wiki/`.

## Session Cache

Reuse a successful earlier result in this session (mempalace ✓, mcpvault ✓,
manifest ✓, mcp-mode recorded) unless >30 minutes passed, user asks to re-check,
or a later MCP call fails.

## Process

1. `mempalace_status` — MemPalace available
2. `get_vault_stats` — mcpvault available
3. **MCP binding probe** — `list_directory("/")`:
   - `wiki` at top level → **`repo-bound`** (default): wiki MCP paths need `wiki/` prefix
   - `projects` / `index.md` and no `wiki` dir → **`wiki-bound`**: omit `wiki/` prefix
   - Record `mcp-mode` for the session
4. Project scope — `mempalace_search(query="ONB manifest", wing=<project>, room="documentation", limit=1)`; fallback git/cwd
5. If manifest found — extract rooms and searches
6. Report:

```
Onboarding harness: mempalace ✓ | mcpvault ✓ | mcp-mode=repo-bound | manifest ✓
```

## Output

```markdown
### Onboarding Results
- mempalace: ✓ / ✗
- mcpvault: ✓ / ✗
- mcp-mode: repo-bound | wiki-bound | unknown
- manifest: ✓ / ✗ (project: <name>)
- rooms: [room1, room2, ...]
```

## On MCP Failure

Record the gap, continue with available sources, note limits in the brief.

## Gotchas

- MCP failure does not block the workflow
- Manifest may be stale — cross-check project structure
- `repo-bound`: never `write_note("projects/...")` without `wiki/`
