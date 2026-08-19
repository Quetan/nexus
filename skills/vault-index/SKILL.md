---
name: vault-index
compatibility: Requires mcpvault MCP.
description: >
  Use when updating vault index.md, log.md, or projects/<p>/00_Index.md after
  vault-write operations. Called automatically by vault-write. Standalone use:
  when manually fixing index drift or log entries. Wikilinks omit `wiki/`
  prefix; MCP paths use `wiki/log.md`, `wiki/index.md`,
  `wiki/projects/<p>/00_Index.md`.
---

# vault-index

## What it does

Maintain the wiki catalog and chronology:
- Append entries to `log.md`, `index.md`, `projects/<project>/00_Index.md`
- Preserve Wiki Purity in every new index entry

## Process

1. **Update indexes** — MCP targets: `wiki/log.md`, `wiki/index.md`,
   `wiki/projects/<p>/00_Index.md` (never bare `log.md` / `projects/...` at MCP root)
2. **Classify pages** — append typed wiki-page links under the matching section
   of `00_Index.md`; never create source queues or lifecycle markers.

### log.md entry

```markdown
## [YYYY-MM-DD] HH:MM — <command> — <description>
  - [[full/vault/root/relative/path|Display Name]]
```

Rules:
- One `## [YYYY-MM-DD]` heading per time group; tab-indented (`  -`) aliased wikilinks
- Wikilinks: full wiki-semantic paths, no `.md`, no `../`, no `wiki/` prefix
- MCP write: `wiki/log.md`
- Commands: `write` / `lint` / `query`
- Link only wiki pages in `log.md`

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

```

## Input

- Command, description, wiki-semantic page paths + display names, project slug

## Constraints

- Append only
- Check duplicates before append
- Wikilinks: wiki-semantic. MCP wiki files: `wiki/...`
- Reject new index content containing external staging links, `Raw Sources`, or
  ingest-status markers.

## Gotchas

- Read existing index before append
- Existing legacy source-queue content is left unchanged; vault-lint reports it.
