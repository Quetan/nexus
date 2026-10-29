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
| `tool:` | Directly observed file, diff, or command result | Cite the repository/path or command and observed result |
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
- One session may produce multiple pages — use one type per page. A Task page
  may summarize work and link to independently significant Decision pages.
- Decision/Task file names include date suffix `DD-MM-YYYY`; Entity/Concept/Topic do not.
- Concepts get `date:` in frontmatter (auto-add if missing). Topics never carry `date`.

## Session work capture

For session work, create or update a Task page with these three sections in the
requester's language. Keep them in the note; the final reply contains status and
page links only.

- **Changed files** — repository identifier, repository-relative path, change
  kind (added/modified/deleted/renamed), and purpose. Use session actions and
  observed diffs/files to identify this task's changes. A dirty working tree
  alone does not establish authorship; exclude unrelated changes. When no files
  changed, say so explicitly; if evidence is unavailable, state that limitation.
- **Adopted decisions** — chosen approach and rationale. Label proposals and open
  questions separately. Say when no new decisions were adopted. Link to separate
  Decision pages for independently significant choices, avoiding duplicated detail.
- **Work results** — completed work, checks actually run and their observed
  outcomes, limitations, and remaining work. Distinguish passed, failed, and
  unrun checks; do not turn planned work into completed work.

Support observed changes and check outcomes with `tool:` evidence (file/diff or
command and result). Use `user:` for explicit requester decisions; an observed
implementation can establish the approach taken without implying user approval.
For pasted files, URLs, and other external material, require these sections only
when the capture also includes session work. Preserve the existing page types
for material-only captures.

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
| ... | tool: | repository/path + observed diff, or command + result |
| ... | UNVERIFIED | — |
```

## Save Flow

1. Validate the draft against Wiki Purity and archive placement before showing it.
2. Show the **wiki-semantic** path and full draft for every page
   (example: `projects/<p>/Tasks/foo-11-08-2026`).
3. Wait for `save`, `edit`, or `skip`.
4. On `save`, reuse the selected `vault_access` backend:
   - MCP `repo-bound` (default): `wiki/` + draft path + `.md`, e.g.
     `write_note(path="wiki/projects/<p>/Tasks/foo-11-08-2026.md", ...)`.
     Never use bare `projects/...` in this mode. Confirmed `wiki-bound` MCP
     omits the prefix; local mode uses `<absolute-local-wiki-root>/<draft-path>.md`.
   - Read existing targets before updating; preserve unrelated content.
   - Call `vault-index` with command=`write`, description, created/updated pages
     (wiki-semantic paths), project slug, and the same `vault_access` context.
   - Read back every created/updated page through the selected backend.
     `vault-index` owns index/log read-back; use its verified-path and failure
     summary without repeating those reads. Mark capture complete only when
     every expected page and index/log update is verified.
   - On a write failure, uncertain completion, or misplaced page, follow shared
     recovery rules before retrying; report partial/unverified work explicitly.

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

- Reply with save status and links to verified pages only. State partial or
  unverified saves honestly; do not repeat changed files, decisions, or results.
- Update indexes only with typed wiki-page links.
- Do not create source queues, caches, or staging references.
