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
question: {research question}
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
MemPalace research track. Batch the full scope in this one run. No code.

wing: {project}; question: {research question}; entities: [...]; repos: [...]; rooms: [...]

Choose the smallest fitting read: search short queries in the relevant room (limit=3
for an entity, 5 for a topic/ticket) for source text and decisions; KG (max 10
relations) for known relationships; diary
(last_n=3) only for recent continuity by a known agent. Do not call search and KG for
every entity. Search first; retry with candidate_strategy="union" for missed exact IDs,
filenames, or ticket numbers. If results are noisy, set max_distance explicitly; no
universal threshold is established. Verify manifest content before using it. Use the
confirmed source date only, and an empty date when unknown. For multiple known drawer
IDs, use mempalace_get_drawers when available. Make one extra targeted read only to
close a plan-relevant gap; if an optional parameter fails, retry default search once.

Return ONLY JSON:
{"status":"success|partial|failed","findings":[{"fact":"...","source":"drawer:<id>","date":"YYYY-MM-DD or empty string when unknown"}],"gaps":["..."]}
MAX 10 plan-relevant findings. Empty=success+gaps. MCP down=failed.
```

### B — Wiki

```
Wiki research via vault-query (batch once). No code. Vault-only unless allow_external=true.
mcp-mode: repo-bound (wiki/ MCP prefix).

project: {project}; entities: [...]; topics: [...]; ticket_ids: [...]; allow_external: {true|false}
vault_access: {mode, local_root, local_consent} (forward to vault-query)

Run vault-query once for the full question list. It owns note discovery and reads.

Return ONLY JSON:
{"status":"success|partial|failed","findings":[{"fact":"...","source":"wiki:<path>","date":"YYYY-MM-DD"}],"gaps":["..."]}
MAX 10. Within-wiki conflict: both dated claims + gap (no winner). Empty=success+gaps.
Follow shared vault access handling on MCP failure; parent asks about local use.
Approved local coverage can succeed. Limited coverage=partial; wholly unavailable/
skipped access=failed+reason in gaps. Return approval needs/access changes and
local-use/freshness limitations in gaps; never read local content before consent.
```

Phase 3: load `code-exploration-prompt.md`; required type `explore`.
