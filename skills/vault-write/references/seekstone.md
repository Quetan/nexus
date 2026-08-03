# Seekstone — Vault Operations

## Primary

Use seekstone MCP tools for all vault operations:
- `seekstone_search` — full-text search
- `seekstone_read_note` — read pages
- `seekstone_create_note` — create new pages
- `seekstone_patch_frontmatter` — update frontmatter
- `seekstone_get_backlinks` — find references
- `seekstone_append_note` — append to existing pages
- `seekstone_get_periodic_note` — health probe (verifies vault binding)

## Vault Path Resolution

Seekstone is bound to a specific vault at MCP startup. The agent does **not**
need to know the absolute vault path — all seekstone paths are relative to the
vault root, which is `wiki/` inside the obsidian-vault repository.

If the vault location must be resolved manually (e.g., file I/O fallback):
1. **Ask seekstone** — call `seekstone_get_periodic_note(period="daily")`; a
   successful response means MCP is bound, paths are vault-root-relative
2. **Default macOS path** — `~/Development/obsidian-vault/wiki/`
3. **Ask user** — if neither works

Never hardcode `/srv/cloud/obsidian` (Linux-legacy) anywhere.

## Fallback (seekstone unavailable)

Use file I/O against the resolved vault root:
- `read <vault-root>/projects/<p>/Entities/foo.md` — read files
- `write <vault-root>/projects/<p>/Entities/foo.md` — write files
- `glob <vault-root>/**/*.md` — list pages

Note in output: "seekstone unavailable — using file I/O fallback"

## Vault Path Convention

All wikilinks and seekstone paths are **vault-root-relative**, with `wiki/` as
the vault root. Paths **never** start with `wiki/`.

| Scope | Vault-root-relative path | Physical repo path |
|-------|--------------------------|---------------------|
| Global index | `index.md` | `wiki/index.md` |
| Log | `log.md` | `wiki/log.md` |
| Project index | `projects/<p>/00_Index.md` | `wiki/projects/<p>/00_Index.md` |
| Project Entity | `projects/<p>/Entities/<name>.md` | `wiki/projects/<p>/Entities/<name>.md` |
| Project Concept | `projects/<p>/Concepts/<name>.md` | `wiki/projects/<p>/Concepts/<name>.md` |
| Project Decision | `projects/<p>/Decisions/<slug-DD-MM-YYYY>.md` | `wiki/projects/<p>/Decisions/<slug-DD-MM-YYYY>.md` |
| Project Task | `projects/<p>/Tasks/<slug-DD-MM-YYYY>.md` | `wiki/projects/<p>/Tasks/<slug-DD-MM-YYYY>.md` |
| Project Topic | `projects/<p>/Topics/<slug>.md` | `wiki/projects/<p>/Topics/<slug>.md` |
| Library index | `library/00_Index.md` | `wiki/library/00_Index.md` |
| Library Entity | `library/<Topic>/Entities/<name>.md` | `wiki/library/<Topic>/Entities/<name>.md` |
| Library Concept | `library/<Topic>/Concepts/<name>.md` | `wiki/library/<Topic>/Concepts/<name>.md` |
| Archive (project) | `projects/<p>/archive/<...>` | `wiki/projects/<p>/archive/<...>` |
| Raw staging | n/a (managed by vault-ingest) | `raw/DD.MM.YYYY/<p>/<slug>.md` |

**Wikilinks in content:** full vault-root-relative path, no `.md`, no `../`, no `wiki/` prefix.

Path mapping follows `consistency-rules.md` — any `<project-slug>` maps to `projects/<project-slug>/`.
