# Vault Conventions — Enforced Structural Rules

These rules apply to new drafts. Existing violations are reported by
`vault-lint` as `[LEGACY]` only; do not mass-migrate them.

## Enforcement Tiers

- 🔴 **Hard gate:** `vault-write` rejects the draft.
- 🟡 **Soft gate:** show the change or diff and require approval.
- 🔵 **Audit-only:** `vault-lint` reports the issue without blocking.

## 0.1 Topic vs Concept

- **Topic:** `type: topic` and no `date`.
- **Concept:** `type: concept`, a `date: YYYY-MM-DD`, and a parent Topic link.

**Enforcement:** 🔴 `vault-write` adds the current date to a new Concept when
missing and removes `date` from a Topic with a warning.

## 0.2 Lifecycle

Allowed values: `living | frozen`.

| Page type | Default lifecycle |
|-----------|-------------------|
| Decision | `frozen` |
| Dated Concept | `frozen` unless explicitly `living` |
| Topic | `living` |
| Entity | `living` |
| Task | `living` |

**Enforcement:** 🟡 Show a diff and require explicit user approval before
changing a frozen page. Manual edits remain allowed after approval.

## 0.3 Wiki Purity

New wiki content must not contain:

- `[[raw/...]]` links;
- a `Raw Sources` heading;
- ingest-status markers such as `⏳ pending` or `✅ ingested`.

**Enforcement:** 🔴 `vault-write` rejects an impure draft. `vault-index` never
creates source queues. `vault-lint` reports existing instances as `[LEGACY]`
only.

## 0.4 Archive Convention

A page with `status: archived` or `status: deprecated` must be physically under
an `archive/` directory, such as `wiki/archive/...` or
`wiki/projects/<project>/archive/...`.

**Enforcement:** 🔴 Reject a new write or status update outside `archive/`.
`vault-lint` reports existing mismatches.

## 0.5 Hub Rule (N≥5)

When five or more Concept pages link to one Entity and that Entity lacks
`00_Index.md`, report a hub recommendation.

**Enforcement:** 🔵 `vault-lint` reports
`[WARN] Hub needed: [[Entity]] (<N> backlinks, no 00_Index.md)`. Users create
hubs manually.

## 0.6 Dataview Coverage

Every top-level wiki namespace containing pages must be covered by at least one
`FROM` clause in the vault-root `index.md`.

**Enforcement:** 🔵 `vault-lint` reports uncovered namespaces.

## Vault Root Convention

- Content wikilinks are relative to the wiki root:
  `[[projects/surgut/Entities/foo]]`, never `[[wiki/projects/...]]`.
- MCP paths for wiki pages start with `wiki/`.
