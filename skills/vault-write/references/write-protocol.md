# Write Protocol — Evidence-Gated Direct Wiki Capture

## Project Resolution

1. **Explicit** — the requester names the project.
2. **Session** — the session already has a project context.
3. **cwd/git** — infer from the current directory or git remote.
4. **Ask** — ask the requester if the result is ambiguous.

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

## Draft Format

Use headings and explanatory prose in the requester's language. If it cannot be
inferred, ask once before drafting.

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

## <Localized summary heading>
<1–3 requester-localized sentences>

## <Localized details heading>
<bullets, tables, code blocks as needed>

## <Localized sources heading>
| Claim | Tag | Evidence |
|-------|-----|----------|
| ... | user: | direct user statement |
| ... | vault: | [[projects/<p>/Entities/foo]] / palace drawer `<id>` |
| ... | UNVERIFIED | — |
```

## Save Flow

1. Validate the draft against Wiki Purity and archive placement before showing it.
2. Show the **wiki-semantic** path and full draft for every page
   (example: `projects/<p>/Tasks/foo-11-08-2026`).
3. Wait for `save`, `edit`, or `skip`.
4. On `save` (MCP mode `repo-bound`, the default):
   - MCP path = `wiki/` + draft path + `.md`
     → `write_note(path="wiki/projects/<p>/Tasks/foo-11-08-2026.md", ...)`
   - **Never** `write_note(path="projects/...")` — creates files at repo root
   - Call `vault-index` with: command=`write`, description, list of created
     pages (wiki-semantic paths), project slug
   - Read back via MCP path `wiki/...` to verify
   - If the note is missing under `wiki/` or appeared under bare `projects/`
     at MCP root → treat as failure; delete the misplaced file if present;
     rewrite with the `wiki/` prefix

## Rejections

Reject any new draft with a `[[raw/...]]` link, a `Raw Sources` heading, or an
ingest-status marker. Do not rewrite existing pages merely to remove legacy
artifacts.

## Anti-patterns

| Don't | Do instead |
|-------|-----------|
| `write_note("projects/<p>/...")` (no `wiki/` prefix) | `write_note("wiki/projects/<p>/...")` |
| Write source-style pages with `type: source` into wiki/ | Pick entity/concept/decision/task/topic |
| Link a new wiki page to external staging | Preserve provenance in the Sources table |
| Decompose one capture into 10+ identical small pages | Group by concept; minimum viable granularity |

## Post-save

- Update indexes only with typed wiki-page links.
- Do not create source queues, caches, or staging references.
