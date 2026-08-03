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

1. Show vault-relative path + full draft for every page to be written
2. Wait for: `save` / `edit` / `skip`
3. On `save`:
   - `write_note` (or file I/O)
   - Call `vault-index` with: command=`write`, description, list of created pages, project slug
   - Read back created page to verify

## Anti-patterns

| Don't | Do instead |
|-------|-----------|
| Write source-style pages with `type: source` into wiki/ | Pick entity/concept/decision/task/topic |
| Cross-link from a wiki page to a `[[raw/...]]` file | Mention evidence tags inline; raw stays vault-external |
| Decompose one capture into 10+ identical small pages | Group by concept; minimum viable granularity |

## Post-save

- **No `raw/` step.** This skill never writes to `raw/` for session captures.
- For pre-existing manual notes already in `raw/`, defer to `vault-ingest`.
