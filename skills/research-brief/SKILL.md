---
name: research-brief
description: >
  Use when generating internal, sourced context after research and optional code
  exploration are complete. Called by nexus during Phase 4. Standalone use:
  when structured findings need a terse current-state summary. Requires:
  research-synthesize.
---

# research-brief

## What it does

Reduce structured research into plan-relevant internal context. Phase 4 never
emits a user-facing response; `research-plan` localizes the final result once.

## Input

- Synthesis contract: `{status, findings[], gaps[]}`
- Optional code-exploration contract: `{status, findings[], gaps[]}`
- Task description
- Complexity tier

## Internal output contract

```json
{
  "status": "success|partial|failed",
  "context": [
    {"text": "<plan-relevant current-state fact>", "sources": ["drawer:<id>", "wiki:<path>"]}
  ],
  "blockers": [
    {"step": "<blocked plan step>", "reason": "<unresolved conflict or required fact>", "question": "<requester question>"}
  ],
  "gaps": ["<non-blocking gap>"]
}
```

## Constraints

- For medium/full, create 2-3 concise context entries. Each must be traceable to
  one or more sources and affect a plan step.
- For light, create only the facts needed for a compact 3-5-line final answer.
- Promote a gap or conflict to `blockers[]` only when it prevents a concrete step.
  Preserve non-blocking gaps but do not surface them in the final deliverable.
- Propagate `partial` when coverage is incomplete; use `failed` only when required
  input is unavailable. Do not invent facts or narrate research activity.

## Gotchas

- Prefer plain language; use technical terminology only when implementation needs it.
- `research-plan` owns localized labels and user-facing prose.
