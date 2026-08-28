---
name: research-plan
description: >
  Use when generating the single requester-localized, human-readable implementation
  plan after research is complete. Called by nexus during Phase 5. Standalone
  use: when internal context needs a concise plan for the requester. Requires:
  research-brief.
---

# research-plan

## What it does

Emit exactly one user-facing deliverable in plain language. Phase 5 owns that
emission; do not repeat Phase 4 context, expose internal handoffs, or surface
paths, source citations, or raw commands to the requester.

## Input

- Internal context contract from `research-brief` (paths, sources, commands live here)
- Task scope (repos, services)
- Requester locale (inferred, or requested once if ambiguous)
- Complexity tier

## Output contract

```markdown
## <localized "Plan">

### <localized "Context">

<2-3 short plain-language paragraphs for medium/full — what matters and why.>

### <localized "Steps">
1. <what to do and why, in everyday language>.
2. ...

### <localized "Verification">
<how to confirm success — described in words, not shell commands.>

### <localized "Blockers"> (only when present)
- <blocked step in plain language>: <reason>. <localized requester question>
```

## Simplicity rules

- Write for a human reader, not an agent log. Use the requester's locale and
  everyday language; avoid internal jargon (drawer IDs, MCP modes, skill names).
- Medium/full: exactly 2-3 short context paragraphs, then 3-7 one-line steps.
  Each step states the action and purpose — no file paths, wikilinks, or
  `backtick` identifiers unless the requester named them in the task.
- Light: omit the standard sections. Emit 3-5 concise plain-language lines
  answering the request, plus a localized blockers section only if action is
  prevented.
- Verification: describe the check in words ("run the test suite", "confirm the
  endpoint returns 200") — do not paste commands or paths.
- Omit non-blocking gaps and conflicts. Include every `blockers[]` item from the
  internal context as a localized blocker tied to its step and requester question.
- Do not emit implementation code or any content after verification except blockers.
- Technical detail (paths, commands, drawer IDs, wiki paths) stays in Phase 4
  and earlier handoffs only. The orchestrator uses that internal context for
  Phase 6 implementation; Phase 5 is presentation for the requester.
