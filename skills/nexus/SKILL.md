---
name: nexus
description: >
  Use when starting a new task that needs institutional context before
  implementation — any project with MemPalace + Obsidian LLM Wiki, even if the
  user doesn't name a specific project. Triggers on "/nexus", "nexus research",
  "собери контекст", "исследуй перед реализацией", or any task in a tracked
  project workspace. Do NOT use when only vault operations (write/ingest/query/lint) are needed
  without research flow. Requires: onboarding-harness, research-mempalace,
  research-wiki, research-synthesize, research-brief, research-plan, vault-query.
---

# nexus

## Why this exists

Tasks fail when agents guess schema, ports, or prior decisions from code or stale
markdown. This skill forces institutional memory lookup — MemPalace drawers and
Obsidian wiki — before any implementation plan. It delivers a research brief and
checklist plan, then stops. No code unless the user explicitly approves implementation.

**Core principle:** Institutional memory before implementation. On conflict —
newer record wins (by date).

## Language Rule

- This skill file and references/: **English**
- Research Brief: **Russian**
- Implementation Plan: match user request language (default Russian)

## Workflow Checklist

### Pre-flight
- [ ] Phase 0: Onboarding harness — delegate to `onboarding-harness`
- [ ] Phase 1: Decompose request — entities, repos

### Vault Research — NO code until synthesis
- [ ] Phase 2a: Parallel research-mempalace + research-wiki (via vault-query)
- [ ] Phase 2b: research-synthesize — merge, newer-wins, gaps

### Code Exploration — only after Phase 2b
- [ ] Phase 3: Code exploration guided by vault findings

### Deliverables
- [ ] Phase 4: research-brief — Russian, Актуальное состояние
- [ ] Phase 5: research-plan — steps with file-paths + verify commands
- [ ] Phase 6: Handoff — wait for user approval before coding

### Post-delivery
- [ ] Phase 7: Wiki capture if significant new knowledge
  - Delegate to `vault-write` (direct wiki write, no raw staging) with user approval
  - For manual external dumps only: `vault-ingest`

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
Output notes for Актуальное состояние.

### Phase 4 — Research Brief

Delegate to `research-brief`. Deliver in Russian.

### Phase 5 — Implementation Plan

Delegate to `research-plan`. Deliver in user's language.

### Phase 6 — Handoff

Wait for user approval before any coding.

### Phase 7 — Wiki Capture

If session produced durable knowledge: call `vault-write` (direct wiki write,
no raw staging) with user approval. Skip for routine bugfixes with no new
decisions. `vault-ingest` is reserved for manual/external source drops — it
is never chained after `vault-write`.

## Pre-delivery Validation

- [ ] Harness check completed
- [ ] Vault research completed before code (except peek)
- [ ] Both palace and wiki tracks ran
- [ ] Every fact has source (drawer id or wikilink)
- [ ] Gaps section exists
- [ ] Актуальное состояние section exists
- [ ] No implementation code (unless user approved)
- [ ] Brief is Russian; plan matches user language
- [ ] Phase 7 noted — scheduled or skipped with reason

## Gotchas

- Runtime ports, credentials, docker: palace manifest only — not markdown
- Wiki track is mandatory even if user says skip; record gap if vault empty
- Older record does not win — newer date wins (equal/missing dates → ask user)

## Anti-patterns

| Rationalization | Reality |
| --- | --- |
| "I know the schema" | Palace and wiki first |
| "Faster to grep code" | Code after vault synthesis |
| "User said skip wiki" | Wiki is mandatory; record gap if empty |
| "I'll write migration now" | Brief + plan first |
| "Too small for brief+plan" | Still deliver both; use the full workflow |

## Quick Reference

| Phase | Action | Delegates to |
| --- | --- | --- |
| 0 | Onboarding | `onboarding-harness` |
| 1 | Decompose | self |
| 2a | Vault research | `research-mempalace` + `research-wiki` |
| 2b | Synthesize | `research-synthesize` |
| 3 | Code | self |
| 4 | Brief | `research-brief` |
| 5 | Plan | `research-plan` |
| 6 | Handoff | self |
| 7 | Wiki capture | `vault-write` (direct), or `vault-ingest` for manual raw drops |
