---
name: vault-index
description: >
  Use when updating vault index.md, log.md, or projects/<p>/00_Index.md after
  vault-write or vault-ingest operations. Called automatically by vault-write
  and vault-ingest. Standalone use: when manually fixing index drift or log
  entries. Requires seekstone MCP. All paths vault-root-relative — never use
  the `wiki/` prefix.
---

# vault-index

## What it does

Maintain the wiki catalog and chronology:
- Append entries to `log.md`
- Append entries to `index.md`
- Append entries to `projects/<project>/00_Index.md`
- Enforce Raw Sources prefix lifecycle (convention 0.3): ⏳ pending → ✅ wiki link, no raw refs

## Process

1. **Update indexes** — append to log.md, index.md, projects/<p>/00_Index.md
2. **Raw Sources lifecycle** (convention 0.3):
   - When a NEW raw file appears (manual user drop or vault-ingest pre-register):
     auto-add `⏳ pending [[raw/DD.MM.YYYY/<p>/<slug>]]` under `## Raw Sources`
   - When vault-ingest completes for a given raw file:
     - Remove the `⏳ pending [[raw/...]]` line entirely
     - Add `✅`-less wikilinks under appropriate **typed sections** (Entities / Concepts / Decisions / Tasks / Topics) of `00_Index.md`, pointing at the result wiki pages
     - Optionally keep one summary line under Raw Sources in the form
       `- ingested → [[projects/<p>/<Bucket>/<name>]], [[projects/<p>/<Bucket>/<other>]]` without prefix emoji, dated
   - Wiki pages never link to `raw/` — only the ingest-queue section may temporarily do so

### log.md entry

```markdown
## [YYYY-MM-DD] HH:MM — <command> — <description>
  - [[full/vault/root/relative/path|Display Name]]
```

Rules:
- `## [YYYY-MM-DD]` — one heading per time group
- Tab-indented (`  -`) wikilinks with display aliases
- Full vault-root-relative paths, no `.md`, no `../`, no `wiki/` prefix
- Command: `write`, `ingest`, `lint`, `query`
- **Never** put `[[raw/...]]` links into log.md — link to result wiki pages
- For `write`: list the created/updated wiki pages
- For `ingest`: list result wiki pages, optionally mention raw slug in prose (`— из raw/DD.MM.YYYY/<p>/<slug>`)

### index.md entry

```markdown
- [[projects/<p>/Entities/<entity>|<Display Name>]] — one-line summary
- [[projects/<p>/Concepts/<concept>|<Display Name>]] — one-line summary
- [[library/<Topic>/Concepts/<concept>|<Display Name>]] — one-line summary
```

### 00_Index.md

Same format as `index.md`, scoped to project. Create the file with
`# <Project> Index` heading if it doesn't exist. Read existing content before
append to avoid duplicates.

Structure (typed sections):

```markdown
# <Project> Index

## Entities
- [[projects/<p>/Entities/foo|Foo]] — ...

## Concepts
- [[projects/<p>/Concepts/bar|Bar]] — ...

## Decisions
- [[projects/<p>/Decisions/baz-01-08-2026|Baz]] — ...

## Tasks
- [[projects/<p>/Tasks/quux-01-08-2026|Quux]] — ...

## Topics
- [[projects/<p>/Topics/quuz|Quuz]] — ...

## Raw Sources  ← ingest queue only, ⏳ prefix for pending
- ⏳ pending [[raw/03.08.2026/<p>/ad-hoc-note|Ad hoc note]]
```

## Input

- Command: write / ingest / lint / query
- Description: what was done
- Page paths: list of created/updated pages with display names
- Project: project slug
- For ingest: list of raw files consumed (to remove ⏳ lines)

## Constraints

- Append only — never rewrite existing entries (except removing consumed ⏳ lines on ingest)
- Check for duplicates before appending to indexes
- Full vault-root-relative paths only (no `.md`, no `../`, no `wiki/` prefix)
- Never emit `[[raw/...]]` wikilinks outside the `## Raw Sources` section of `00_Index.md`
- Wikilinks in `## Raw Sources` are the ONLY place raw files may appear; once ingested, that line is replaced by wiki links

## Gotchas

- Read existing index before append — duplicate detection required
- Raw Sources links without ⏳/✅ → auto-add ⏳ pending (convention 0.3)
- After ingest: remove the ⏳ line — do NOT replace it with `✅ [[raw/...]]` (raw refs not allowed in wiki)
- Full vault-root-relative paths only (no `.md`, no `../`) — relative paths break indexes
