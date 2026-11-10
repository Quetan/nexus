# Subagent Prompts — Phase 2a Vault Research

Load **only** when dispatching medium/full Phase 2a (not for light inline).

## Dispatch rules

| Track         | Named role and model profile | Generic fallback | Forbidden |
| ------------- | ---------------------------- | ---------------- | --------- |
| A — MemPalace | `nexus-mempalace`             | `generalPurpose` | `explore` |
| B — Wiki      | `nexus-wiki`                  | `generalPurpose` | `explore` |

- Launch **two** named research agents in parallel for medium/full tiers.
- Use the runtime's native model profile for each named role. Codex uses
  `gpt-6-luna` with high reasoning effort, Cursor uses standard
  `composer-2.5[fast=false]`, and Claude Code uses `haiku` at its default effort.
- Keep synthesis, research brief, and final plan in the parent session so they
  use the model selected for the task.
- One Task per track; batch all entities inside that Task.
- Custom agents already know the skill contract — send **scope payload only**
  (below). Do not paste the full skill body into the Task prompt.
- If the named role is unavailable, use a generic agent only when the runtime
  lets the parent select a separate model. If no separate model can be selected
  or the launch fails, run only that missing track inline/on the parent model.
  Keep the completed track and record the fallback in the internal handoff.
- If the runtime reports that it substituted a model, keep usable findings and
  record the actual model in the internal handoff. Do not rerun that track.
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
vault_access: {mode: mcp|local|skip, local_root: null|absolute-wiki-root, local_consent: pending|approved|declined|unavailable}
```

On MCP unavailability, load this skill's bundled fallback (linked from `SKILL.md`).
The parent owns the local-fallback question. Children forward access changes or
approval needs in `gaps`; the parent retains the context and resumes only the wiki
track after answering, preserving completed work from the other track.

## Compact prompts (generalPurpose fallback only)

### A — MemPalace

```
MemPalace research track. Batch all entities in this one run. No code.

wing: {project}; entities: [...]; repos: [...]; rooms: [...]

For each entity: mempalace_search(limit=3) + mempalace_kg_query (max 10 relations).
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
vault_access: {mode, local_root, local_consent} (forward to vault-query)

1) Read index.md  2) vault-query once for full question list  3) backlinks if needed

Return ONLY JSON:
{"status":"success|partial|failed","findings":[{"fact":"...","source":"wiki:<path>","date":"YYYY-MM-DD"}],"gaps":["..."]}
MAX 10. Within-wiki conflict: both dated claims + gap (no winner). Empty=success+gaps.
Follow shared vault access handling on MCP failure; parent asks about local use.
Approved local coverage can succeed. Limited coverage=partial; wholly unavailable/
skipped access=failed+reason in gaps. Return approval needs/access changes and
local-use/freshness limitations in gaps; never read local content before consent.
```

Phase 3: load `code-exploration-prompt.md`; required type `explore`.
