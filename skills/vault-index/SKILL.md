---
name: vault-index
description: >
  Use when updating wiki/index.md, wiki/log.md, or project 00_Index.md after
  vault-write or vault-ingest operations. Called automatically by vault-write
  and vault-ingest. Standalone use: when manually fixing index drift or log
  entries. Requires user-seekstone MCP.
---

# vault-index

## What it does

Maintain the wiki catalog and chronology:
- Append entries to `wiki/log.md`
- Append entries to `wiki/index.md`
- Append entries to `projects/<project>/00_Index.md`

## Process

### log.md entry

```markdown
## [YYYY-MM-DD] HH:MM — <command> — <description>
  - [[full/vault/relative/path|Display Name]]
```

Rules:
- `## [YYYY-MM-DD]` — one heading per time group
- Tab-indented (`  -`) wikilinks with display aliases
- Full vault-relative paths, no `.md`, no `../`
- Command: `write`, `ingest`, `lint`, `query`

### index.md entry

```markdown
- [[projects/<project>/Entities/<entity>|<Display Name>]] — one-line summary
- [[projects/<project>/Concepts/<concept>|<Display Name>]] — one-line summary
```

### 00_Index.md

Same format as `wiki/index.md`, scoped to project. Create the file with
`# <Project> Index` heading if it doesn't exist. Read existing content before
append to avoid duplicates.

## Input

- Command: write / ingest / lint / query
- Description: what was done
- Page paths: list of created/updated pages with display names
- Project: project slug

## Constraints

- Append only — never rewrite existing entries
- Check for duplicates before appending to indexes
- Full vault paths only (no `.md`, no `../`)
