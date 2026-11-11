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

## Process

1. Route each saved page by its wiki-semantic path to the owning
   `projects/<project>/00_Index.md` or `library/<Topic>/00_Index.md`.
   Use the selected backend and its binding. MCP repo-bound paths start
   with `wiki/`; wiki-bound and local paths use the wiki root directly.
2. Read the owning index. Resolve existing targets to full paths before
   checking for duplicates. Insert a missing link in the matching
   section by type (such as Tasks or Задачи); create the section once if
   missing. Move an existing Task entry into the task section if needed.
   Preserve unrelated sections. If no index exists, create it and add
   its navigation link to the home index for a project or to
   `library/00_Index.md` for a library topic.
3. Add a log entry to `log.md`. The home `index.md` holds navigation to
   project indexes, the library, and the log. Change it only when those
   navigation targets change.
4. Read each changed index and the log once after the final write and
   return verified wiki-semantic paths and failed or unverified updates.
   On uncertainty, use the shared [write recovery](references/vault-access.md#write-recovery)
   before retrying.

### Profile index entry

```markdown
- [[projects/<p>/Tasks/<slug>|TICKET — Task title]] — Outcome or subject in one line
```

Use a full wiki-semantic target without `wiki/`, `.md`, or `../`.
Include the ticket ID if present, and a short description supported by
the note. Keep separate links to different notes about the same ticket.
Archived pages, when indexed, go in a separate archive section.

### Log entry

Append one heading per event and include only wiki page links:

```markdown
## [YYYY-MM-DD] HH:MM — <command> — <description>
  - [[full/wiki/semantic/path|Display Name]]
```

Use the time of the event, and avoid adding a duplicate entry on retry.

## Input

Command (`write`, `lint`, or `query`), description, created or updated
wiki-semantic page paths and titles, and inherited `vault_access` when
available. Keep existing legacy source material intact. Reject new index
entries containing external staging links, `Raw Sources`, or ingest-status
markers.
