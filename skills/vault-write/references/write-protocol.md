# Write Protocol — Evidence-Gated Direct Wiki Capture

## Project Resolution Order

1. **Explicit** — user names the project
2. **Session** — already working in a project context
3. **cwd/git** — current directory or git remote
4. **Ask** — if ambiguous, ask the user

## Evidence Rules

| Tag | Meaning | Usage |
|-----|---------|-------|
| `user:` | Direct user statement | Safe to write as fact |
| `vault:` | From wiki lookup | Include source wikilink |
| `UNVERIFIED` | Agent inference | Never write as fact |

## Page Type Decision

Pick exactly one type per page before drafting:

| Type | When | Example slug |
|------|------|--------------|
| **Entity** | service, person, tool, library, team | `projects/<p>/Entities/cc-api-sng` |
| **Concept** | pattern, workflow, architecture insight | `projects/<p>/Concepts/auth-flow` |
| **Decision** | a choice + rationale (reversible-or-not) | `projects/<p>/Decisions/db-schema-v2-04-08-2026` |
| **Task** | tracked work item, fix, audit | `projects/<p>/Tasks/file-upload-fix-04-08-2026` |
| **Topic** | stable, dateless hub grouping concepts | `library/<Topic>/00_Index` |

Rules:
- One session may produce multiple pages — split by type, never mix.
- Decision/Task file names include date suffix `DD-MM-YYYY`; Entity/Concept/Topic do not.
- Concepts get `date:` in frontmatter (auto-add if missing). Topics never carry `date`.

## Draft Format (per page)

```markdown
---
title: "<short title>"
date: YYYY-MM-DD                # for Concept/Decision/Task/Entity
project: <project-slug>
type: entity|concept|decision|task|topic
status: active
lifecycle: frozen|living        # default: frozen for Decision & dated Concept, living otherwise
tags: [<type>, project/<slug>]
related: []
---

# <Title>

## Суть
<1-3 предложения на русском>

## Детали
<bullets, tables, code blocks as needed>

## Sources
| Claim | Tag | Evidence |
|-------|-----|----------|
| ... | user: | direct user statement |
| ... | vault: | [[projects/<p>/Entities/foo]] / palace drawer `<id>` |
| ... | UNVERIFIED | — |
```

## Save Flow

1. Show **wiki-semantic** path + full draft for every page
   (example draft path: `projects/<p>/Tasks/foo-11-08-2026`)
2. Wait for: `save` / `edit` / `skip`
3. On `save` (MCP mode `repo-bound`, the default):
   - MCP path = `wiki/` + draft path + `.md`
     → `write_note(path="wiki/projects/<p>/Tasks/foo-11-08-2026.md", ...)`
   - **Never** `write_note(path="projects/...")` — creates files at repo root
   - Call `vault-index` with: command=`write`, description, list of created
     pages (wiki-semantic paths), project slug
   - Read back via MCP path `wiki/...` to verify
   - If the note is missing under `wiki/` or appeared under bare `projects/`
     at MCP root → treat as failure; delete the misplaced file if present;
     rewrite with the `wiki/` prefix

## Anti-patterns

| Don't | Do instead |
|-------|-----------|
| `write_note("projects/<p>/...")` (no `wiki/` prefix) | `write_note("wiki/projects/<p>/...")` |
| Write source-style pages with `type: source` into wiki/ | Pick entity/concept/decision/task/topic |
| Cross-link from a wiki page to a `[[raw/...]]` file | Mention evidence tags inline; raw stays vault-external |
| Decompose one capture into 10+ identical small pages | Group by concept; minimum viable granularity |

## Post-save

- **No `raw/` step.** This skill never writes to `raw/` for session captures.
- For pre-existing manual notes already in `raw/`, defer to `vault-ingest`.
