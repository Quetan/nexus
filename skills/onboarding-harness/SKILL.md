---
name: onboarding-harness
compatibility: Requires MemPalace and mcpvault MCPs.
description: >
  Use when diagnosing MemPalace or mcpvault MCP binding problems — broken wiki
  paths, unclear repo-bound vs wiki-bound mode, or missing project manifest.
  Optional diagnostic only; nexus research does NOT call this on every run.
  Do NOT use as a mandatory pre-flight health check. Do NOT use for routine
  vault-query or vault-write when MCP already works.
---

# onboarding-harness

## What it does

Diagnose mcpvault binding and optionally MCP reachability / project manifest.
Standalone troubleshooting — not part of the nexus mandatory checklist.

## When to run

- User asks to check MCP / fix vault paths
- First MCP path read/write fails (missing path / wrong root)
- Before `vault-write` when binding is unknown

Do **not** run at the start of every nexus research session.

## Modes

### Binding-only (default for path errors / pre-write)

1. `list_directory("/")` only:
   - `wiki` at top level → **`repo-bound`** (MCP paths need `wiki/` prefix)
   - `projects` / `index.md` and no `wiki` → **`wiki-bound`** (omit `wiki/`)
2. Report `mcp-mode=...` and stop.

### Full diagnostic (user asked to check MCP)

1. `mempalace_status`
2. `get_vault_stats`
3. Binding probe as above
4. Optional: `mempalace_search(query="ONB manifest", wing=<project>, room="documentation", limit=1)`; fallback git/cwd
5. Report:

```
Onboarding harness: mempalace ✓ | mcpvault ✓ | mcp-mode=repo-bound | manifest ✓
```

## Output

```markdown
### Onboarding Results
- mempalace: ✓ / ✗ / skipped
- mcpvault: ✓ / ✗ / skipped
- mcp-mode: repo-bound | wiki-bound | unknown
- manifest: ✓ / ✗ / skipped (project: <name>)
- rooms: [room1, room2, ...]
```

## Gotchas

- Default without this skill is **`repo-bound`**
- Prefer binding-only unless the user asked for a full check
- `repo-bound`: never `write_note("projects/...")` without `wiki/`
