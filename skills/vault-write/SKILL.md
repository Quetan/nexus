---
name: vault-write
compatibility: mcpvault MCP, or an existing local vault with requester consent.
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

## Vault access

Reuse `vault_access` (mode, absolute wiki root, consent); otherwise use MCP without pings.
On missing tools or connection failure, read the bundled [local fallback](references/vault-access.md)
once per agent context. Inherited `local` reuses its approved root; `skip` records
the reason and skips vault operations. Under nexus, the parent owns local consent.

## What it does

Evidence-gated capture **directly into `wiki/`**. Curate session work and
external materials into typed pages: entity, concept, decision, task, or topic.
Every claim is tagged `user:`, `vault:`, `tool:`, or `UNVERIFIED` before drafting.

## References

Load when needed:
- [Write protocol](references/write-protocol.md) — before drafting (resolution, evidence, save flow, page types)
- [External-source protocol](references/external-source-protocol.md) — before curating pasted files, URLs, or external materials
- [Consistency rules](references/consistency-rules.md) — frontmatter, paths, dates
- [Vault conventions](references/vault-conventions.md) — before creating any wiki page
- [MCP paths](references/mcpvault.md) — binding/path diagnostics
- [Write recovery](references/vault-access.md#write-recovery) — failed or uncertain writes

## Process

1. **Identify input** — session output or external material; resolve project explicitly → session → cwd/git → ask.
2. **Extract evidence** — tag every claim: `user:` / `vault:` / `tool:` / `UNVERIFIED`.
3. **Search wiki for overlap** — avoid duplicates and identify updates or contradictions.
4. **Pick page types** — entity / concept / decision / task / topic.
5. **Draft** — use wiki-semantic paths and a Sources table; reject impure wiki content.
   For session work, include changed files, adopted decisions, and work results
   in a Task page, following the [write protocol](references/write-protocol.md).
6. **Wait for user** — `save` / `edit` / `skip`.
7. **On save** — write and read back pages through the selected backend; pass
   `vault_access` to `vault-index` and use its index/log verification result.
8. **Report** — return only save status and page links; keep work details in the note.

## Sources Table Format

Use requester-localized column labels:

| <Localized claim> | <Localized tag> | <Localized evidence> |
|-------------------|-----------------|----------------------|
| <claim> | user: | direct user statement |
| <claim> | vault: | [[projects/<p>/<Bucket>/<name>]] / palace drawer `<id>` |
| <claim> | tool: | observed file/diff, or command and result |
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
- Local fallback consent does not approve draft contents or override filesystem permissions.
