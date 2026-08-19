---
name: nexus
description: >
  Use when starting a new task that needs institutional context before
  implementation — any project with MemPalace + Obsidian LLM Wiki, even if the
  user doesn't name a specific project. Triggers on "/nexus", "nexus research",
  "собери контекст", "исследуй перед реализацией", or any task in a tracked
  project workspace. Do NOT use when only vault operations (write/query/lint) are needed
  without research flow. Requires: onboarding-harness, research-mempalace,
  research-wiki, research-synthesize, research-brief, research-plan, vault-query,
  vault-write, vault-lint, vault-index.
---

# nexus

## Why this exists

Research institutional memory before proposing implementation. Deliver one
requester-localized plan, then stop; do not write code without explicit approval.

## Language and output policy

- Keep all skill instructions, subagent prompts, and structured handoffs in English.
- Infer the requester's locale from the request. If it is ambiguous, ask once before
  the final emission.
- Localize all Phase 5 labels and prose to that locale. Preserve paths, identifiers,
  commands, and source citations unchanged.
- Phase 4 is internal input only. Phase 5 is the sole user-facing research emission.

## Workflow Checklist

### Pre-flight
- [ ] Phase 0: Onboarding harness — delegate to `onboarding-harness`
- [ ] Phase 1: Decompose request — entities, repos
- [ ] MemPalace checkpoint after Phase 1 — scope → `decisions` [ref]

### Vault Research — NO code until synthesis
- [ ] Phase 2a: Parallel research-mempalace + research-wiki (via vault-query)
- [ ] Phase 2b: research-synthesize — merge, newer-wins, gaps
- [ ] MemPalace checkpoint after Phase 2b — facts + gaps → `documentation` [ref]

### Code Exploration — only after Phase 2b
- [ ] Phase 3: Code exploration guided by vault findings
- [ ] MemPalace checkpoint after Phase 3 — current state → `documentation` [ref]

### Deliverables
- [ ] Phase 4: research-brief — internal, sourced context
- [ ] Phase 5: research-plan — one localized context-and-plan emission
- [ ] MemPalace checkpoint after Phase 5 — plan decisions → `decisions` [ref]
- [ ] Phase 6: Handoff — wait for user approval before coding

### Post-delivery
- [ ] Phase 7: Wiki capture if significant new knowledge
  - Delegate to `vault-write` for direct wiki writing with user approval
- [ ] MemPalace checkpoint after Phase 7 — final decisions + session close → `decisions` + diary [ref]

## Phase Details

Prose only where it adds beyond the Quick Reference table: Phase 1 (tier rules),
Phase 2a (parallel dispatch), Phase 7 (conditional capture). All other phases are
fully covered by the table below.

### Phase 1 — Decompose & Classify Request

| Dimension | Output |
| --- | --- |
| Entities | services, tables, features, ticket ids |
| Repos | from project context |
| Task type | migration, bugfix, feature, cross-service |
| **Complexity tier** | light / medium / full — see below |

**Tier rules:**
- **light** — single fact lookup, 1-2 known entities, no schema/endpoint change implied
  → run one targeted research track, skip Phase 3, and emit the localized compact
  format from `research-plan`: 3-5 sourced lines and blockers only when action is blocked
- **medium** — typical feature/bugfix touching known entities
  → run both tracks, code exploration under budget, then the standard final format
- **full** — migration, new architecture, cross-service change, or scope genuinely unclear
  → run the standard pipeline and ask focused questions for unresolved blockers

If unsure which tier applies, default to **medium**. User can always request full
explicitly ("сделай полное исследование" / "full nexus").

### References

Load when needed:
- [Subagent prompts](references/subagent-prompts.md) — before Phase 2a dispatch (A+B only)
- [Code exploration prompt](references/code-exploration-prompt.md) — before Phase 3
- [MemPalace logging](references/mempalace-logging.md) — before first checkpoint

### Phase 2a — Vault Research (parallel)

Dispatch two sub-agents in parallel:

1. **research-mempalace** — palace queries for the entities/rooms
2. **research-wiki** → **vault-query** — wiki queries

No code exploration during Phase 2a (except 1-2 user-named peek files).

All research and Phase 3 handoffs use:

```json
{"status":"success|partial|failed","findings":[],"gaps":[]}
```

`failed` means the source or required capability was unavailable. `partial` means
coverage was limited. Empty results remain `success` with gaps. Promote a gap to a
blocker only when it prevents a specific plan step. `research-synthesize` retains
the blocked step and the user question in the structured handoff.

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

### Phase 7 — Wiki Capture

If session produced durable knowledge: call `vault-write` for direct wiki writing
with user approval. Skip routine bugfixes with no new decisions.

### MemPalace Logging (throughout)

After each checkpoint phase (1, 2b, 3, 5, 7), call `mempalace_checkpoint` —
decisions → `decisions` room, findings → `documentation` room, one diary entry
per checkpoint. MemPalace is the full work log; the wiki stays curated-only.
See [MemPalace logging](references/mempalace-logging.md).

## Pre-delivery Validation

- [ ] Harness check completed
- [ ] Vault research completed before code (except peek)
- [ ] Research track(s) ran per tier (both for medium/full, target track for light)
- [ ] Medium/full context has 2-3 short, plain-language sourced paragraphs
- [ ] Plan has 3-7 one-line steps and one concise localized verification line
- [ ] Light output has 3-5 sourced lines; no artificial context or steps
- [ ] No findings narrative, gaps, or conflicts unless a blocker stops a step
- [ ] No implementation code (unless user approved)
- [ ] Phase 5 is the only user-facing output and matches requester locale
- [ ] Phase 7 noted — scheduled or skipped with reason
- [ ] Every checkpoint phase (1, 2b, 3, 5, 7) logged to MemPalace

## Gotchas

- Runtime ports, credentials, docker: palace manifest only — not markdown
- Wiki track is mandatory even if user says skip; record gap if vault empty
- Newer-wins applies only across sources; equal/missing dates → ask user
- Never batch two checkpoint phases into one — a mid-session crash loses un-logged work

## Anti-patterns

| Rationalization | Reality |
| --- | --- |
| "I know the schema" | Palace and wiki first |
| "Faster to grep code" | Code after vault synthesis |
| "User said skip wiki" | Wiki is mandatory; record gap if empty |
| "I'll write migration now" | Plan first |
| "Need to show what I found" | Emit only the tier-appropriate localized plan |
| "Too small to log" | Every phase accrues decisions — checkpoint it, verbatim |
| "Too small for brief+plan" | Tier-appropriate output is still required — light tier uses condensed format, it does not skip sourcing |

## Quick Reference

| Phase | Action | Delegates to |
| --- | --- | --- |
| 0 | Onboarding | `onboarding-harness` |
| 1 | Decompose | self |
| 2a | Vault research | `research-mempalace` + `research-wiki` |
| 2b | Synthesize | `research-synthesize` |
| 3 | Code | self |
| 4 | Brief | `research-brief` (internal context) |
| 5 | Plan | `research-plan` (single final emission) |
| 6 | Handoff | self |
| 7 | Wiki capture | `vault-write` (direct) |
| CP | MemPalace checkpoint (after 1, 2b, 3, 5, 7) | `mempalace_checkpoint` (self) |
