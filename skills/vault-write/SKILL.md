---
name: vault-write
description: >
  Use when preserving session work, external sources, or agent outputs directly
  into the Obsidian LLM Wiki — even if the user doesn't explicitly say "save"
  or "write". Triggers on "save session", "write note", "сохрани", "запиши
  результаты". Requires seekstone MCP (or file I/O fallback). Automatically
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

Use for preserving session work, external sources, and agent outputs. Valuable
query answers from `vault-query` should also be captured — see `vault-query`
for the file-back step.

## References

Load these when needed:
- Read [references/write-protocol.md] **before drafting a wiki page** — project resolution, evidence rules, save flow, page-type decision
- Read [references/consistency-rules.md] **when writing frontmatter, creating file paths, or formatting dates** — naming, dates, frontmatter standards
- Read [references/vault-conventions.md] **before creating any wiki page** — six enforced structural rules
- Read [references/seekstone.md] **if seekstone MCP returns errors** — file I/O fallback details

## Process

1. **Identify project** — explicit → session → cwd/git → ask user
2. **Evidence pass** — tag every claim:
   - `user:` — from user statement
   - `vault:` — from wiki or palace lookup
   - `UNVERIFIED` — agent inference, needs clarification
3. **Search wiki for overlap** — avoid duplicate pages
4. **Pick page type** — entity / concept / decision / task / topic (see write-protocol)
5. **Draft** — show full vault-relative path + content + Sources table
6. **Wait for user** — `save` / `edit` / `skip`
7. **On save**: create page → call `vault-index` (log update + index update) → verify read back

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
- Frontmatter dates use `YYYY-MM-DD` (file system dates for Decisions/Tasks use `DD-MM-YYYY`).
- Wikilinks always vault-root-relative — never with `wiki/` prefix.

## Gotchas

- Convention 0.1 auto-fix: Concepts auto-get `date`, Topics never carry `date`
- Convention 0.2 default: Decisions and dated Concepts default to `lifecycle: frozen`
- seekstone may be unavailable — have file I/O fallback ready
- SHA256 cache belongs to `vault-ingest`, not this skill — direct wiki write skips raw/ entirely
