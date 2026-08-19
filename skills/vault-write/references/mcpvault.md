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
`[[<basename>]]` under `wiki/`.

## Dual-root model

mcpvault is bound to the **vault repository root**, not to `wiki/` alone.
Obsidian wiki semantics use `wiki/` as the content root.

| Layer | Root | Used for | Example |
|-------|------|----------|---------|
| MCP / filesystem | vault **repo** | `write_note`, `read_note`, `patch_note`, file I/O | `wiki/projects/surgut/Tasks/foo.md` |
| Wiki semantic | `wiki/` | wikilinks, Dataview, drafts shown to user | `[[projects/surgut/Tasks/foo]]` |

**Hard rule for MCP tool paths:**
- Wiki pages → path **must** start with `wiki/`
- Never call `write_note` with bare `projects/...` — that creates files at
  repo root (`<repo>/projects/...`) instead of `<repo>/wiki/projects/...`

If onboarding-harness reports MCP mode `wiki-bound` (no top-level `wiki/`
dir), omit the `wiki/` prefix. Default live mode is `repo-bound`.

## Vault Path Resolution

If the vault location must be resolved manually (e.g., file I/O fallback):
1. **Ask mcpvault** — `get_vault_stats()` + `list_directory("/")`; confirm
   `wiki` appears as a top-level directory (`repo-bound`)
2. **Default macOS path** — `~/Development/obsidian-vault/` (repo);
   wiki content under `wiki/`
3. **Ask user** — if neither works

Do not use `/srv/cloud/obsidian` as a **Mac file I/O fallback** (remote host
path). The live remote mcpvault server may still bind there — that is fine;
clients talk over MCP, not local paths.

## Fallback (mcpvault unavailable)

Use absolute file I/O against the Mac clone:
- `read ~/Development/obsidian-vault/wiki/projects/<p>/Entities/foo.md`
- `write ~/Development/obsidian-vault/wiki/projects/<p>/Entities/foo.md`
- `glob ~/Development/obsidian-vault/wiki/**/*.md`

Never write cwd-relative `projects/...` (lands in the wrong repo).

Note in output: "mcpvault unavailable — using file I/O fallback"

## Path table

| Scope | Wikilink / draft path | MCP / filesystem path |
|-------|----------------------|------------------------|
| Global index | `index.md` | `wiki/index.md` |
| Log | `log.md` | `wiki/log.md` |
| Project index | `projects/<p>/00_Index.md` | `wiki/projects/<p>/00_Index.md` |
| Project Entity | `projects/<p>/Entities/<name>` | `wiki/projects/<p>/Entities/<name>.md` |
| Project Concept | `projects/<p>/Concepts/<name>` | `wiki/projects/<p>/Concepts/<name>.md` |
| Project Decision | `projects/<p>/Decisions/<slug-DD-MM-YYYY>` | `wiki/projects/<p>/Decisions/<slug-DD-MM-YYYY>.md` |
| Project Task | `projects/<p>/Tasks/<slug-DD-MM-YYYY>` | `wiki/projects/<p>/Tasks/<slug-DD-MM-YYYY>.md` |
`<project>` mapping: `surgut` → `projects/surgut/`, `infrastructure` →
`projects/infrastructure/`, `library/<topic>` → `library/<Topic>/`.

Wikilinks in page content: wiki-semantic, no `.md`, no `wiki/` prefix.
