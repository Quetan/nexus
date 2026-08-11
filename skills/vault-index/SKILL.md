---
name: vault-index
description: >
  Use when updating vault index.md, log.md, or projects/<p>/00_Index.md after
  vault-write or vault-ingest operations. Called automatically by vault-write
  and vault-ingest. Standalone use: when manually fixing index drift or log
  entries. Requires mcpvault MCP. Wikilinks omit `wiki/` prefix; MCP paths use
  `wiki/log.md`, `wiki/index.md`, `wiki/projects/<p>/00_Index.md`.
---

# vault-index

## What it does

Maintain the wiki catalog and chronology:
- Append entries to `log.md`, `index.md`, `projects/<project>/00_Index.md`
- Enforce Raw Sources prefix lifecycle (convention 0.3)

## Process

1. **Update indexes** — MCP targets: `wiki/log.md`, `wiki/index.md`,
   `wiki/projects/<p>/00_Index.md` (never bare `log.md` / `projects/...` at MCP root)
2. **Raw Sources lifecycle** (convention 0.3):
   - New raw file → add `⏳ pending [[raw/DD.MM.YYYY/<p>/<slug>]]` under `## Raw Sources`
   - After ingest → remove ⏳ line; add wikilinks under typed sections of `00_Index.md`
   - Optional summary: `- ingested → [[projects/<p>/<Bucket>/<name>]], ...`
   - Wiki pages never link to `raw/` outside that ingest-queue section

### log.md entry

```markdown
## [YYYY-MM-DD] HH:MM — <command> — <description>
  - [[full/vault/root/relative/path|Display Name]]
```

Rules:
- One `## [YYYY-MM-DD]` heading per time group; tab-indented (`  -`) aliased wikilinks
- Wikilinks: full wiki-semantic paths, no `.md`, no `../`, no `wiki/` prefix
- MCP write: `wiki/log.md`
- Commands: `write` / `ingest` / `lint` / `query`
- Never `[[raw/...]]` in log.md — link result wiki pages only

### index.md entry

```markdown
- [[projects/<p>/Entities/<entity>|<Display Name>]] — one-line summary
- [[projects/<p>/Concepts/<concept>|<Display Name>]] — one-line summary
- [[library/<Topic>/Concepts/<concept>|<Display Name>]] — one-line summary
```

MCP write: `wiki/index.md`

### 00_Index.md

Same format as `index.md`, scoped to project. Create `# <Project> Index` if
missing. Read before append. MCP path: `wiki/projects/<p>/00_Index.md`.

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

- Command, description, wiki-semantic page paths + display names, project slug
- For ingest: list of raw files consumed (to remove ⏳ lines)

## Constraints

- Append only (except removing consumed ⏳ lines on ingest)
- Check duplicates before append
- Wikilinks: wiki-semantic. MCP wiki files: `wiki/...`
- `[[raw/...]]` only under `## Raw Sources` in `00_Index.md`

## Gotchas

- Read existing index before append
- Unprefixed Raw Sources links → auto-add ⏳ pending
- After ingest: delete ⏳ line — do not replace with `✅ [[raw/...]]`
