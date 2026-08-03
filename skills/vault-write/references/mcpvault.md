# mcpvault — Vault Operations

## Primary

Use mcpvault MCP tools for all vault operations:
- `search_notes` — full-text search
- `read_note` — read pages
- `write_note` — create/overwrite pages (mode="append" appends)
- `update_frontmatter` — update frontmatter
- `get_vault_stats` — health probe (verifies vault binding)
- `list_directory` / `get_notes_info` — list pages
- `list_all_tags` — list tags

Backlinks: no dedicated tool. Find references via
`search_notes(query="<title>", searchContent=true)` or file I/O grep for
`[[<basename>]]` over the vault root.

## Vault Path Resolution

mcpvault is bound to a specific vault at MCP startup. The agent does **not**
need to know the absolute vault path — all mcpvault paths are relative to the
vault root, which is `wiki/` inside the obsidian-vault repository.

If the vault location must be resolved manually (e.g., file I/O fallback):
1. **Ask mcpvault** — call `get_vault_stats()`; a successful response means
   MCP is bound, paths are vault-root-relative
2. **Default macOS path** — `~/Development/obsidian-vault/wiki/`
3. **Ask user** — if neither works

Never hardcode `/srv/cloud/obsidian` (Linux-legacy) anywhere.

## Fallback (mcpvault unavailable)

Use file I/O against the resolved vault root:
- `read <vault-root>/projects/<p>/Entities/foo.md` — read files
- `write <vault-root>/projects/<p>/Entities/foo.md` — write files
- `glob <vault-root>/**/*.md` — list pages

Note in output: "mcpvault unavailable — using file I/O fallback"

## Vault Path Convention

All wikilinks and mcpvault paths are **vault-root-relative**, with `wiki/` as
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

`<project>` mapping: `surgut` → `projects/surgut/`, `infrastructure` → `projects/infrastructure/`, `library/<topic>` → `projects/library/<Topic>/`.

Wikilinks: vault-root-relative, no `.md`, no `wiki/` prefix.
