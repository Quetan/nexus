---
name: vault-write
description: >
  Use when preserving session work, external sources, or agent outputs as source
  notes in the Obsidian LLM Wiki — even if the user doesn't explicitly say "save"
  or "write". Triggers on "save session", "write note", "сохрани", "запиши
  результаты". Requires user-seekstone MCP (or file I/O fallback). Automatically
  calls vault-index for log update after save. Do NOT use for MemPalace storage,
  codegraph output, or non-wiki markdown editing. Requires: vault-index.
---

# vault-write

## What it does

Evidence-gated raw capture of session work into `sources/` directory. Every
claim is tagged `user:`, `vault:`, or `UNVERIFIED` before writing.

Use for preserving session work, external sources, and agent outputs. Valuable
query answers from `vault-query` should also be captured — see `vault-query`
for the file-back step.

## References

Load these when needed:
- Read [references/write-protocol.md] **before drafting a source note** — project resolution, evidence rules, save flow
- Read [references/consistency-rules.md] **if unsure about date format or frontmatter fields** — naming, dates, frontmatter standards
- Read [references/seekstone.md] **if seekstone MCP returns errors** — file I/O fallback details

## Process

1. **Identify project** — explicit → session → cwd/git → ask user
2. **Evidence pass** — tag every claim:
   - `user:` — from user statement
   - `vault:` — from wiki or palace lookup
   - `UNVERIFIED` — agent inference, needs clarification
3. **Search wiki for overlap** — avoid duplicate sources
4. **Draft** — show path + content + Sources table
5. **Wait for user** — `save` / `edit` / `skip`
6. **On save**: create note → call `vault-index` (log update + index update)
7. **Verify** — read back created note

## Sources Table Format

| Claim | Tag | Evidence |
|-------|-----|----------|
| <claim> | user: | direct user statement |
| <claim> | vault: | [[wiki/path]] / palace drawer `<id>` |
| <claim> | UNVERIFIED | — |

## Constraints

- Draft before write. No silent writes.
- No `UNVERIFIED` claim ever asserted as fact.
- User session facts override vault for write content.
- Source dates use `DD.MM.YYYY` format.

## Gotchas

- Source dates use DD.MM.YYYY (match source directory), not YYYY-MM-DD
- seekstone may be unavailable — have file I/O fallback ready
- SHA256 computation failure → fallback to cache miss (never block)
