---
name: nexus
description: >
  Use when starting a new task that needs institutional context before
  implementation — any project with MemPalace + Obsidian LLM Wiki, even if the
  user doesn't name a specific project. Triggers on "/nexus", "nexus research",
  "собери контекст", "исследуй перед реализацией", or any task in a tracked
  project workspace. Do NOT use when only vault operations (write/ingest/query/lint) are needed
  without research flow. Requires: onboarding-harness, research-mempalace,
  research-wiki, research-synthesize, research-brief, research-plan, vault-query,
  vault-write, vault-lint, vault-index.
---

# nexus

## Why this exists

Tasks fail when agents guess schema, ports, or prior decisions from code or stale
markdown. This skill forces institutional memory lookup — MemPalace drawers and
Obsidian wiki — before any implementation plan. It delivers one compact document — a short
Контекст block plus a deterministic step-by-step plan — then stops. No code
unless the user explicitly approves implementation.

**Core principle:** Institutional memory before implementation; every session
writes its work back — MemPalace gets the full work log (continuous, verbatim),
the wiki gets curated key knowledge. On conflict — newer record wins (by date).

## Language Rule

- This skill file and references/: **English**
- Research Brief: **Russian**
- Implementation Plan: match user request language (default Russian)

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
- [ ] MemPalace checkpoint after Phase 3 — актуальное состояние → `documentation` [ref]

### Deliverables
- [ ] Phase 4: research-brief — compact Контекст block (≤8 bullets, Russian)
- [ ] Phase 5: research-plan — merged deliverable: Контекст + deterministic plan
- [ ] MemPalace checkpoint after Phase 5 — plan decisions → `decisions` [ref]
- [ ] Phase 6: Handoff — wait for user approval before coding

### Post-delivery
- [ ] Phase 7: Wiki capture if significant new knowledge
  - Delegate to `vault-write` (direct wiki write, no raw staging) with user approval
  - For manual external dumps only: `vault-ingest`
- [ ] MemPalace checkpoint after Phase 7 — final decisions + session close → `decisions` + diary [ref]

## Phase Details

### Phase 1 — Decompose Request

| Dimension | Output |
| --- | --- |
| Entities | services, tables, features, ticket ids |
| Repos | from project context |
| Task type | migration, bugfix, feature, cross-service |

### References

Load when needed:
- Read [references/subagent-prompts.md] **before dispatching Phase 2a sub-agents** — prompt templates for parallel vault research (A+B only)
- Read [references/code-exploration-prompt.md] **before Phase 3** — prompt for code exploration subagent
- Read [references/mempalace-logging.md] **before first checkpoint** — continuous session log contract

### Phase 2a — Vault Research (parallel)

Dispatch two sub-agents in parallel:

1. **research-mempalace** — palace queries for the entities/rooms
2. **research-wiki** → **vault-query** — wiki queries

No code exploration during Phase 2a (except 1-2 user-named peek files).

### Phase 2b — Synthesize

Delegate to `research-synthesize`:
- Deduplicate overlapping facts
- Flag conflicts — newer date wins
- List gaps (topics with no record)

### Phase 3 — Code Exploration

Guided by vault findings. Read key files (controllers, migrations, configs, routes).
Output notes for the Контекст block.

### Phase 4 — Research Brief (compact context)

Delegate to `research-brief`. Produces a Контекст block in Russian: max 8 bullets,
only facts that affect the plan. No findings narrative, no gaps/conflicts prose.

### Phase 5 — Implementation Plan (single merged deliverable)

Delegate to `research-plan`. Emits ONE user-facing document: the Контекст block
(≤8 bullets) followed by deterministic steps (file-path + command + verification)
and an optional Блокеры section. Phases are internal; this plan is the only output
the user sees.

### Phase 6 — Handoff

Wait for user approval before any coding.

### Phase 7 — Wiki Capture

If session produced durable knowledge: call `vault-write` (direct wiki write,
no raw staging) with user approval. Skip for routine bugfixes with no new
decisions. `vault-ingest` is reserved for manual/external source drops — it
is never chained after `vault-write`.

### MemPalace Logging (throughout)

After each checkpoint phase (1, 2b, 3, 5, 7), call `mempalace_checkpoint` —
decisions → `decisions` room, findings → `documentation` room, one diary entry
per checkpoint. MemPalace is the full work log; the wiki stays curated-only.
See [references/mempalace-logging.md].

## Pre-delivery Validation

- [ ] Harness check completed
- [ ] Vault research completed before code (except peek)
- [ ] Both palace and wiki tracks ran
- [ ] Контекст ≤ 8 bullets, every bullet cites a source (drawer id or wikilink)
- [ ] Every plan step has file-path + command + verification
- [ ] No findings narrative, no gaps/conflicts prose (only Блокеры if blocking)
- [ ] No implementation code (unless user approved)
- [ ] Plan is in user's language; Контекст is Russian
- [ ] Phase 7 noted — scheduled or skipped with reason
- [ ] Every checkpoint phase (1, 2b, 3, 5, 7) logged to MemPalace

## Gotchas

- Runtime ports, credentials, docker: palace manifest only — not markdown
- Wiki track is mandatory even if user says skip; record gap if vault empty
- Older record does not win — newer date wins (equal/missing dates → ask user)
- Never batch two checkpoint phases into one — a mid-session crash loses un-logged work

## Anti-patterns

| Rationalization | Reality |
| --- | --- |
| "I know the schema" | Palace and wiki first |
| "Faster to grep code" | Code after vault synthesis |
| "User said skip wiki" | Wiki is mandatory; record gap if empty |
| "I'll write migration now" | Plan first |
| "Need to show what I found" | User wants the plan, not the findings — emit compact Контекст + steps only |
| "Too small to log" | Every phase accrues decisions — checkpoint it, verbatim |

## Quick Reference

| Phase | Action | Delegates to |
| --- | --- | --- |
| 0 | Onboarding | `onboarding-harness` |
| 1 | Decompose | self |
| 2a | Vault research | `research-mempalace` + `research-wiki` |
| 2b | Synthesize | `research-synthesize` |
| 3 | Code | self |
| 4 | Brief | `research-brief` (compact Контекст) |
| 5 | Plan | `research-plan` (merged Контекст + steps) |
| 6 | Handoff | self |
| 7 | Wiki capture | `vault-write` (direct), or `vault-ingest` for manual raw drops |
| CP | MemPalace checkpoint (after 1, 2b, 3, 5, 7) | `mempalace_checkpoint` (self) |
