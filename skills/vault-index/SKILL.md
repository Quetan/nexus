---
name: vault-index
compatibility: mcpvault MCP, or an existing local vault with requester consent.
description: >
  Use when updating vault index.md, log.md, or projects/{project}/00_Index.md after
  vault-write operations. Called automatically by vault-write. Standalone use:
  when manually fixing index drift or log entries. Wikilinks omit `wiki/`
  prefix; MCP paths use `wiki/log.md`, `wiki/index.md`,
  `wiki/projects/{project}/00_Index.md`.
---

# vault-index

## Vault access

Reuse `vault_access` (mode, absolute wiki root, consent); otherwise use MCP without pings.
On missing tools or connection failure, read the bundled [local fallback](references/vault-access.md)
once per agent context. Inherited `local` reuses its approved root; `skip` records
the reason and skips vault operations. Under nexus, the parent owns local consent.

## What it does

Maintain the wiki catalog and chronology:
- Append entries to `log.md`, `index.md`, `projects/<project>/00_Index.md`
- Preserve Wiki Purity in every new index entry

## Process

1. **Update indexes** — MCP targets: `wiki/log.md`, `wiki/index.md`,
   `wiki/projects/{project}/00_Index.md` (never bare `log.md` / `projects/...` at MCP root)
2. **Classify pages** — append typed wiki-page links under the matching section
   of `00_Index.md`; never create source queues or lifecycle markers.
3. **Verify** — own the index/log read-back through the selected backend;
   return verified wiki-semantic paths and failed/unverified updates to the caller.
   Read each affected target once after its final append. Repeat only for an
   unresolved failure or a later mutation. On uncertainty, load bundled
   [write recovery](references/vault-access.md#write-recovery) before retrying.

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
missing. Read before append. MCP path: `wiki/projects/{project}/00_Index.md`.

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

- Command, description, created/updated wiki-semantic page paths + display names,
  project slug, and optional `vault_access` inherited from the caller

## Constraints

- Use the caller's backend/root for every read and append; in local mode resolve
  wiki-semantic paths under the approved absolute wiki root.
- Append only
- Check duplicates before append
- Wikilinks: wiki-semantic. MCP wiki files: `wiki/...`
- Reject new index content containing external staging links, `Raw Sources`, or
  ingest-status markers.

## Gotchas

- Read existing index before append
- Existing legacy source-queue content is left unchanged; vault-lint reports it.
