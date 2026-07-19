# Seekstone — Vault Operations

## Primary

Use seekstone MCP tools for all vault operations:
- `seekstone_search` — full-text search
- `seekstone_read_note` — read pages
- `seekstone_create_note` — create new pages
- `seekstone_patch_frontmatter` — update frontmatter
- `seekstone_get_backlinks` — find references
- `seekstone_append_note` — append to existing pages

## Fallback (seekstone unavailable)

Use file I/O directly:
- `read <vault-path>` — read files
- `write <vault-path>` — write files
- `glob <vault-path>/**/*.md` — list pages

Note in output: "seekstone unavailable — using file I/O fallback"

## Vault Path Convention

All paths are relative to vault root. Use `<project>/` subdir for known projects;
flat `raw/DD.MM.YYYY/<slug>.md` is also valid.

| Scope | Path pattern |
|-------|-------------|
| Index | `wiki/index.md` |
| Log | `wiki/log.md` |
| Raw source | `raw/DD.MM.YYYY/<project>/<slug>.md` |
| Project entity | `wiki/projects/<project>/Entities/<entity>.md` |
| Project concept | `wiki/projects/<project>/Concepts/<concept>.md` |
| Project decision | `wiki/projects/<project>/Decisions/<slug-DD-MM-YYYY>.md` |
| Library index | `wiki/library/00_Index.md` |
| Library entity | `wiki/library/<Topic>/Entities/<entity>.md` |
| Library concept | `wiki/library/<Topic>/Concepts/<concept>.md` |

`<project>` mapping: `surgut` → `wiki/projects/surgut/`, `infrastructure` → `wiki/projects/infrastructure/`, `library/<topic>` → `wiki/library/<Topic>/`.

Wikilinks in content: full vault-relative paths, no `.md`, no `../`.
