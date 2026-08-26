# MemPalace Logging — Continuous Session Log

Load when running the nexus workflow. MemPalace is the complete work log
(every action, finding, decision, verbatim); the wiki is curated key knowledge
only. Wing is always the project (from the manifest).

## When to checkpoint

After each checkpoint phase, before starting the next: **1, 2b, 3, 5, 7**.
Never batch two phases into one checkpoint — a mid-session crash loses
un-logged work.

## What goes where

| Room | Content |
| --- | --- |
| `decisions` | decomposed scope, assumptions, chosen approach, plan decisions, user approvals, conflict resolutions |
| `documentation` | research findings, synthesis, gaps, current state |
| diary entry | running narrative — what happened, what was decided, what's next |

## Call contract

```
mempalace_checkpoint(
  items=[
    {wing: <project>, room: "decisions",    content: "<verbatim decisions>"},
    {wing: <project>, room: "documentation", content: "<verbatim findings>"}
  ],
  diary={
    agent_name: <self>,
    wing: <project>,
    topic: "nexus:<task>",
    entry: "<AAAK, 2-4 lines>"
  }
)
```

- Content is verbatim — exact words, never summarized (dedup needs originals).
- One topic per item; a checkpoint holds 1-3 items. Exactly one diary entry per checkpoint.
- Keep sources in content (drawer IDs, wikilinks) — same as research output.
- Runtime facts (ports, credentials, docker) go here only — never markdown.

## Per-phase map

| Checkpoint | File to palace |
| --- | --- |
| after 1 | project, tier, entities, repos, mcp-mode default, rooms source → `decisions` |
| after 2b | track findings + merged synthesis, gaps, conflict resolutions → `documentation` |
| after 3 | current state (code facts) → `documentation` |
| after 5 | chosen approach, plan decisions → `decisions` |
| after 7 | final decisions, wiki pages written, session close → `decisions` + diary |
