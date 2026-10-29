---
name: nexus
description: >
  Use when starting a new task that needs institutional context before
  implementation — any project with MemPalace + Obsidian LLM Wiki, even if the
  user doesn't name a specific project. Triggers on "/nexus", "nexus research",
  "собери контекст", "исследуй перед реализацией", or any task in a tracked
  project workspace. Do NOT use when only vault operations (write/query/lint) are needed
  without research flow. Requires: research-mempalace,
  research-wiki, research-synthesize, research-brief, research-plan, vault-query,
  vault-write, vault-lint, vault-index.
---

# nexus

## Vault access

Reuse `vault_access` (mode, absolute wiki root, consent); otherwise use MCP without pings.
On missing tools or connection failure, read the bundled [local fallback](references/vault-access.md)
once per agent context. Inherited `local` reuses its approved root; `skip` records
the reason and skips vault operations. Under nexus, the parent owns local consent.

## Why this exists

Research institutional memory before proposing implementation. Deliver one
requester-localized plan, then stop; do not write code without explicit approval.

## Language and output policy

- Keep all skill instructions, subagent prompts, and structured handoffs in English.
- Infer the requester's locale; if ambiguous, ask once before the final emission.
- Phase 5 (`research-plan`): localize labels and prose; write in plain, human-readable
  language. Do **not** include file paths, source citations, wikilinks, drawer IDs,
  or raw shell commands in the user-facing emission.
- Phases 2–4 retain full technical detail (paths, sources, commands) in structured
  internal handoffs only.
- Phase 4 is internal input only. Phase 5 is the sole user-facing research emission.

## Workflow Checklist

### Session bootstrap + decompose
- [ ] Phase 1: Session bootstrap + decompose — mcp-mode default, manifest, entities, tier
- [ ] MemPalace checkpoint after Phase 1 — scope → `decisions` [ref]

### Vault Research — NO code until synthesis
- [ ] Phase 2a: research-mempalace + research-wiki (via vault-query) — per subagent matrix
- [ ] Phase 2b: research-synthesize — merge, newer-wins, gaps
- [ ] MemPalace checkpoint after Phase 2b — facts + gaps → `documentation` [ref]

### Code Exploration — only after Phase 2b
- [ ] Phase 3: Code exploration guided by vault findings (`explore` subagent)
- [ ] MemPalace checkpoint after Phase 3 — current state → `documentation` [ref]

### Deliverables
- [ ] Phase 4: research-brief — internal, sourced context
- [ ] Phase 5: research-plan — one localized context-and-plan emission
- [ ] MemPalace checkpoint after Phase 5 — plan decisions → `decisions` [ref]
- [ ] Phase 6: Handoff — wait for user approval; implement from internal context (Phases 2–4), not Phase 5 text alone

### Post-delivery
- [ ] Phase 7: Wiki capture if significant new knowledge
  - Delegate to `vault-write` for direct wiki writing with user approval
- [ ] MemPalace checkpoint after Phase 7 — final decisions + session close → `decisions` + diary [ref]

### Phase 1 — Session Bootstrap & Classify Request

- Start with real task operations; do **not** call `mempalace_status` or
  `get_vault_stats` for availability. MemPalace failure → `status: failed` / gap.
  On mcpvault unavailability, use bundled fallback; retain `vault_access` for later phases.
- Default **`mcp-mode=repo-bound`**. Probe `list_directory("/")` only after a path
  error or before `vault-write` when binding is unknown (or optional
  `onboarding-harness`).
- Resolve **project** from git/cwd (then ask if unclear). Use default rooms
  (`documentation`, `architecture`, `decisions`) unless already known.
  Call `mempalace_search(query="ONB manifest", ...)` **only** when project or
  rooms/searches are still ambiguous — not on every run.

Record entities (services, tables, features, ticket IDs), repositories, rooms,
task type (migration, bugfix, feature, cross-service), and complexity tier.

**Tier rules:**
- **light** — single fact lookup, 1-2 known entities, no schema/endpoint change
  → **one** relevant track **inline** (no Task; wiki **or** MemPalace), skip Phase 3;
  compact `research-plan` (3–5 plain-language lines; blockers only if blocked)
- **medium** — known entities → **both** tracks + code exploration under budget
- **full** — migration / new architecture / cross-service / unclear scope → both
  tracks + broader exploration; ask on unresolved blockers

Default **medium**. User may force full ("сделай полное исследование" / "full nexus").

### References

Load when needed:
- [Subagent prompts](references/subagent-prompts.md) — before Phase 2a dispatch (A+B only)
- [Code exploration prompt](references/code-exploration-prompt.md) — before Phase 3
- [MemPalace logging](references/mempalace-logging.md) — before first checkpoint

### Phase 2a — Vault Research (parallel)

| Tier / phase | Mechanism | `subagent_type` |
| --- | --- | --- |
| light / one track | Parent skill inline — **no Task** | — |
| medium·full / 2a | 2× Task parallel | Prefer `nexus-mempalace` + `nexus-wiki`; fallback `generalPurpose`; never `explore` |
| medium·full / 3 | 1× Task | **`explore`** |
| 2b / 4 / 5 | Parent + skill — no Task | — |

Medium/full: dispatch A+B in one message. No code in Phase 2a (except 1–2 peek files).
Pass `vault_access` to the wiki track; resolve local consent in the parent and
resume only that track, preserving MemPalace results (see subagent prompts).
Reuse access decisions for capture/lint/index; record skips and continue independent steps.

All research and Phase 3 handoffs use:

```json
{"status":"success|partial|failed","findings":[],"gaps":[]}
```

Completed MCP/local coverage is `success`; limited coverage is `partial`;
unavailable/skipped access after fallback is `failed`. Record local use and known
limitations in `gaps`; empty results remain `success` with topic gaps. Promote a
gap to a blocker only if it prevents a specific plan step; research-synthesize
retains the blocked step and requester question in its handoff.

### External-source gate

Set `allow_external: true` only when the requester explicitly authorizes external
sources for this task. Without that value, research-wiki and vault-query are
vault-only; do not use web, codegraph, or other external evidence. Record the
gate value in every dispatched research prompt.

### Conflict policy

- Resolve contradictory records by newer date only when the records come from
  different sources (MemPalace versus wiki).
- Never choose a winner for a conflict within the wiki. Preserve both dated claims,
  mark the relevant step blocked, and ask the requester to resolve it.
- Equal or missing cross-source dates are also blockers; never infer a winner.

### Phase 6 — Handoff

Wait for explicit requester approval before writing code. On approval, implement
from the internal brief, synthesis, and code-exploration handoffs (Phases 2–4),
which retain technical paths and commands; Phase 5 is the human-readable summary.

### Phase 7 — Wiki Capture

If session produced durable knowledge: call `vault-write` for direct wiki writing
with user approval, forwarding `vault_access`. Session captures include changed
files, adopted decisions, and work results in a Task page under the write
protocol; the save reply contains status and page links only. Skip routine
bugfixes with no new durable knowledge unless the requester asks for capture.

### MemPalace Logging (throughout)

After each checkpoint phase (1, 2b, 3, 5, 7), call `mempalace_checkpoint` —
decisions → `decisions` room, findings → `documentation` room, one diary entry
per checkpoint. MemPalace is the full work log; the wiki stays curated-only.
See [MemPalace logging](references/mempalace-logging.md).

## Pre-delivery Validation

- [ ] Bootstrap done (default mcp-mode; no availability pings); vault before code
- [ ] Vault access propagated; parent resolves consent before local reads; skips/limits recorded
- [ ] Tracks and subagent types match tier matrix; Phase 3 = `explore`
- [ ] Medium/full: 2–3 plain-language Context paragraphs, 3–7 steps, verification (no paths/citations/commands in Phase 5)
- [ ] Light: 3–5 plain-language lines only; Phase 5 sole user-facing emission
- [ ] No code without approval; Phase 7 noted; checkpoints 1/2b/3/5/7 logged

## Gotchas

- Runtime ports/credentials/docker: palace manifest only — not markdown
- Medium/full: include the wiki track unless the requester declines it or shared
  access handling skips it; record the reason. Empty vault → topic gap.
- Newer-wins across sources only; equal/missing dates → ask user
- Never batch two checkpoint phases; no availability pings. MCP wiki failure
  follows shared fallback handling before the track is marked unavailable.

## Anti-patterns

| Rationalization | Reality |
| --- | --- |
| "I know the schema" / "grep first" | Palace and wiki before code |
| "Skip fallback handling after wiki MCP failure" | Parent resolves local access; record skip if unusable/declined |
| "Show findings" / "too small to log" | Emit tier plan only; checkpoint every phase |
| "Always onboarding-harness" | Diagnostic only until a real MCP call fails |
| "explore for wiki/MemPalace" | `explore` is Phase 3 codebase only |

## Quick Reference

| Phase | Action | Delegates to |
| --- | --- | --- |
| 1 | Bootstrap + decompose | self (optional `onboarding-harness` on path errors) |
| 2a | Vault research | `research-mempalace` + `research-wiki` (Task / inline per matrix) |
| 2b | Synthesize | `research-synthesize` |
| 3 | Code | Task `explore` |
| 4 | Brief | `research-brief` (internal context) |
| 5 | Plan | `research-plan` (single final emission) |
| 6 | Handoff (internal context for coding) | self |
| 7 | Wiki capture | `vault-write` (direct) |
| CP | MemPalace checkpoint (after 1, 2b, 3, 5, 7) | `mempalace_checkpoint` (self) |
