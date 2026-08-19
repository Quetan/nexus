---
name: research-plan
description: >
  Use when generating the single requester-localized, sourced implementation
  plan after research is complete. Called by nexus during Phase 5. Standalone
  use: when internal context needs concise, actionable implementation steps.
  Requires: research-brief.
---

# research-plan

## What it does

Emit exactly one user-facing deliverable. Phase 5 owns that emission; do not
repeat Phase 4 context or expose internal handoffs.

## Input

- Internal context contract from `research-brief`
- Task scope (repos, services)
- Requester locale (inferred, or requested once if ambiguous)
- Complexity tier

## Output contract

```markdown
## <localized "Plan">

### <localized "Context">

<2-3 short sourced paragraphs for medium/full.>
<localized "Source">: drawer:<id>, [[path]].

### <localized "Steps">
1. `<file-path>` — <one-line action and purpose>.

<localized "Verification">: `<command>` or <short manual check>.

### <localized "Blockers"> (only when present)
- <blocked step>: <reason>. <localized requester question>
```

## Simplicity rules

- Medium/full: use exactly 2-3 short context paragraphs, each with a localized
  source label and drawer ID or wikilink. Then give 3-7 one-line steps.
- Light: omit the standard sections. Emit 3-5 concise sourced lines answering the
  request, plus a localized blockers section only if action is prevented.
- State file paths when known; do not invent them or add implementation details
  merely to fill the format. Use one verification line with only relevant commands.
- Omit non-blocking gaps and conflicts. Include every `blockers[]` item from the
  internal context as a localized blocker tied to its step and requester question.
- Translate all headings, labels, and prose to the requester locale. Keep paths,
  identifiers, commands, and citations unchanged. If locale remains ambiguous,
  ask once instead of defaulting to a language.
- Do not emit implementation code or any content after verification except blockers.
