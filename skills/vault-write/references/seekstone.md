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

All paths are relative to Obsidian vault root:
- `wiki/index.md`
- `wiki/log.md`
- `sources/DD.MM.YYYY/project/slug.md`
- `projects/<project>/Entities/<entity>.md`
- `projects/<project>/Concepts/<concept>.md`

Wikilinks in content: full vault-relative paths, no `.md`, no `../`.
