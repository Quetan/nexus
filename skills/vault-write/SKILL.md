---
name: vault-write
description: >
  Use when preserving session work, external sources, or agent outputs directly
  into the Obsidian LLM Wiki — even if the user doesn't explicitly say "save"
  or "write". Triggers on "save session", "write note", "сохрани", "запиши
  результаты". Requires mcpvault MCP (or file I/O fallback). Automatically
  calls vault-index for log/index update after save. Do NOT use for MemPalace
  storage, codegraph output, or non-wiki markdown editing.
  Requires: vault-index.
---

# vault-write

## What it does

Evidence-gated capture of session work **directly into `wiki/`** — no raw
staging. Every claim tagged `user:`, `vault:`, or `UNVERIFIED` before writing.
Produces typed wiki pages (entity / concept / decision / task / topic).

For integrating pre-existing manual notes from `raw/`, use `vault-ingest`.

## References

Load when needed:
- [Write protocol](references/write-protocol.md) — before drafting (resolution, evidence, save flow, page types)
- [Consistency rules](references/consistency-rules.md) — frontmatter, paths, dates
- [Vault conventions](references/vault-conventions.md) — before creating any wiki page
- [mcpvault dual-root](references/mcpvault.md) — before any MCP path write/read; file I/O fallback

## Process

1. **Identify project** — explicit → session → cwd/git → ask user
2. **Evidence pass** — tag every claim: `user:` / `vault:` / `UNVERIFIED`
3. **Search wiki for overlap** — avoid duplicate pages
4. **Pick page type** — entity / concept / decision / task / topic (write-protocol)
5. **Draft** — wiki-semantic path (`projects/...`) + content + Sources table
6. **Wait for user** — `save` / `edit` / `skip`
7. **On save** — MCP `wiki/projects/...` → `vault-index` → read back under `wiki/`

## Sources Table Format

| Claim | Tag | Evidence |
|-------|-----|----------|
| <claim> | user: | direct user statement |
| <claim> | vault: | [[projects/<p>/<Bucket>/<name>]] / palace drawer `<id>` |
| <claim> | UNVERIFIED | — |

## Constraints

- Draft before write. No silent writes.
- No `UNVERIFIED` claim ever asserted as fact.
- User session facts override vault for write content.
- Frontmatter dates: `YYYY-MM-DD`; Decision/Task filenames: `DD-MM-YYYY`.
- Wikilinks: wiki-semantic (no `wiki/` prefix). MCP wiki paths: `wiki/...` — see [mcpvault.md](references/mcpvault.md).

## Gotchas

- Concepts auto-get `date`; Topics never carry `date` (convention 0.1)
- Decisions and dated Concepts default `lifecycle: frozen` (convention 0.2)
- File I/O fallback: `~/Development/obsidian-vault/wiki/...` (never cwd-relative `projects/...`)
- SHA256 cache belongs to `vault-ingest` — this skill skips `raw/`
