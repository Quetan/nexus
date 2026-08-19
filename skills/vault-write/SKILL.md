---
name: vault-write
compatibility: Requires mcpvault MCP; supports absolute file I/O fallback.
description: >
  Use when preserving session work, pasted files, URLs, external materials, or
  agent outputs directly into the Obsidian LLM Wiki — even if the user does not
  explicitly say "save" or "write". Triggers on "save session", "write note",
  "сохрани", "запиши результаты". Automatically calls vault-index after save.
  Do NOT use for MemPalace storage,
  codegraph output, or non-wiki markdown editing.
  Requires: vault-index.
---

# vault-write

## What it does

Evidence-gated capture **directly into `wiki/`**. Curate session work and
external materials into typed pages: entity, concept, decision, task, or topic.
Every claim is tagged `user:`, `vault:`, or `UNVERIFIED` before drafting.

## References

Load when needed:
- [Write protocol](references/write-protocol.md) — before drafting (resolution, evidence, save flow, page types)
- [External-source protocol](references/external-source-protocol.md) — before curating pasted files, URLs, or external materials
- [Consistency rules](references/consistency-rules.md) — frontmatter, paths, dates
- [Vault conventions](references/vault-conventions.md) — before creating any wiki page
- [mcpvault dual-root](references/mcpvault.md) — before any MCP path write/read; file I/O fallback

## Process

1. **Identify input** — session output or external material; resolve project explicitly → session → cwd/git → ask.
2. **Extract evidence** — tag every claim: `user:` / `vault:` / `UNVERIFIED`.
3. **Search wiki for overlap** — avoid duplicates and identify updates or contradictions.
4. **Pick page types** — entity / concept / decision / task / topic.
5. **Draft** — use wiki-semantic paths and a Sources table; reject impure wiki content.
6. **Wait for user** — `save` / `edit` / `skip`.
7. **On save** — write to MCP `wiki/...`, call `vault-index`, then read back under `wiki/`.

## Sources Table Format

Use requester-localized column labels:

| <Localized claim> | <Localized tag> | <Localized evidence> |
|-------------------|-----------------|----------------------|
| <claim> | user: | direct user statement |
| <claim> | vault: | [[projects/<p>/<Bucket>/<name>]] / palace drawer `<id>` |
| <claim> | UNVERIFIED | — |

## Constraints

- Draft before write. No silent writes.
- No `UNVERIFIED` claim ever asserted as fact.
- User session facts override vault for write content.
- Frontmatter dates: `YYYY-MM-DD`; Decision/Task filenames: `DD-MM-YYYY`.
- Wikilinks: wiki-semantic (no `wiki/` prefix). MCP wiki paths: `wiki/...` — see [mcpvault.md](references/mcpvault.md).
- Wiki Purity is a hard gate: reject new `[[raw/...]]`, `Raw Sources` headings, and ingest-status markers.

## Gotchas

- Concepts auto-get `date`; Topics never carry `date` (convention 0.1)
- Decisions and dated Concepts default `lifecycle: frozen` (convention 0.2)
- File I/O fallback: `~/Development/obsidian-vault/wiki/...` (never cwd-relative `projects/...`)
