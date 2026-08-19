---
name: research-synthesize
description: >
  Use when merging structured findings from MemPalace and the wiki into unified,
  plan-relevant context. Called by nexus after Phase 2a. Standalone use: after
  collecting multiple research sources that need deduplication and conflict
  handling.
---

# research-synthesize

## Input

- MemPalace contract: `{status, findings[], gaps[]}`
- Wiki contract: `{status, findings[], gaps[]}`
- Task scope and proposed plan steps, when known

## Process

1. Combine `findings[]` directly; do not parse prose.
2. Deduplicate claims about the same entity and fact.
3. Retain only implementation-relevant facts: schema, endpoints, configuration,
   decisions, and constraints.
4. Resolve contradictions only across sources (MemPalace versus wiki): normalize
   dates and retain the newer record.
5. Never select a winner for two wiki claims. Also preserve both claims when
   cross-source dates are equal or missing.
6. Promote an uncertainty to a blocker when a conflicting or missing
   plan-relevant fact changes implementation choice, scope, or verification. Map it
   to the affected step; retain all other uncertainty as a gap.

## Output contract

```json
{
  "status": "success|partial|failed",
  "findings": [
    {
      "fact": "<plan-relevant fact>",
      "source": "drawer:<id>|wiki:<path>",
      "date": "YYYY-MM-DD"
    }
  ],
  "gaps": ["<non-blocking gap>"],
  "blockers": [
    {
      "step": "<blocked plan step>",
      "reason": "<unresolved conflict or required fact>",
      "claims": [{"fact": "...", "source": "...", "date": "YYYY-MM-DD"}],
      "question": "<focused requester question>"
    }
  ]
}
```

## Constraints

- Cap `findings[]` at 15; merge related facts before dropping plan-relevant ones.
- Preserve source and date traceability for every retained claim.
- `failed` means both required tracks are unavailable. Propagate `partial` when
  either track is partial or coverage is materially incomplete. An empty result is
  `success` with gaps.
- Every unresolved within-wiki conflict and every equal- or missing-date
  cross-source conflict that affects implementation requires a `blockers[]` entry
  with a focused requester question.
- Never blend conflicting facts, invent a date, or narrate searches.
- `research-brief` localizes nothing; `research-plan` alone localizes the final
  requester-facing output.
