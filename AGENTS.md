# AGENTS.md — nexus atomic skills

## Project Overview

`nexus` is an eleven-skill system for researching institutional context before
implementation and curating durable knowledge into an Obsidian wiki. `nexus`
is the read-only orchestrator; it has ten peer skills.

The authoritative design is
[2026-08-19-nexus-atomic-skills-design.md](docs/superpowers/specs/2026-08-19-nexus-atomic-skills-design.md).

## Architecture

| Decision        | Current contract                                                                                |
| --------------- | ----------------------------------------------------------------------------------------------- |
| Skill count     | 11 total: `nexus` plus 10 peer skills                                                           |
| Coupling        | Light; dependencies are named in `Requires:`                                                    |
| Research        | MemPalace and wiki research precede code exploration                                            |
| Session start   | No MCP availability pings; default `mcp-mode=repo-bound`; manifest in Phase 1                   |
| Subagents       | Phase 2a: `nexus-mempalace`/`nexus-wiki` or `generalPurpose`; Phase 3: `explore`; light inline  |
| Capture         | `vault-write` is the only wiki capture path                                                     |
| Approval        | Draft first; write only after `save`, `edit`, or `skip`                                         |
| Output language | User-facing content uses the requester's locale; internal instructions and handoffs use English |
| Wiki semantics  | Wikilinks use `[[projects/...]]`; MCP wiki paths use `wiki/...` in repo-bound mode              |

## Skill Inventory

```text
skills/
├── nexus/                 # research orchestrator
├── onboarding-harness/    # optional MCP/binding diagnostic
├── research-mempalace/    # MemPalace findings
├── research-wiki/         # wiki findings via vault-query
├── research-synthesize/   # merge and conflict handling
├── research-brief/        # internal context
├── research-plan/         # single requester-localized final plan
├── vault-query/           # focused wiki research
├── vault-write/           # evidence-gated typed-page curation
├── vault-index/           # index and log maintenance
└── vault-lint/            # read-only wiki health checks
```

Cursor Task agents for Phase 2a: `.cursor/agents/nexus-mempalace.md` and
`nexus-wiki.md`. For cross-project use, copy them to `~/.cursor/agents/`.
Agent names differ from skill names on purpose (avoid skill/Task collision).## Skill Authoring Contract

- Keep each skill single-purpose and imperative.
- Main `SKILL.md` files must stay at or below 200 lines; references are loaded
  just in time and remain one level deep.
- Frontmatter requires `name` and `description`; `name` matches the directory.
- Use `Requires:` in `description` only for peer skill names. Put MCP
  prerequisites in `compatibility:`.
- Write instruction prose in English. Multilingual trigger phrases and
  requester-localized output examples are permitted.
- Pass research and code-exploration results as
  `{status, findings[], gaps[]}`. Add `blockers[]` only where the receiving
  contract defines it.

## Research Contract

Use the tier appropriate to the request:

| Tier   | Flow                                                      | Final emission (Phase 5, human-readable)        |
| ------ | --------------------------------------------------------- | ----------------------------------------------- |
| Light  | One relevant research track (inline); no code exploration | 3–5 plain-language lines                        |
| Medium | Both tracks, synthesis, scoped code exploration           | 2–3 Context paragraphs, 3–7 steps, verification |
| Full   | Medium flow with broader scope                            | 2–3 Context paragraphs, 3–7 steps, verification |

`research-plan` is the only Phase 5 user-facing emission — no paths, citations,
or commands. Phase 4 produces internal context with full technical detail for
Phase 6 implementation. Resolve conflicting records by newer date only across
MemPalace and wiki; keep within-wiki conflicts unresolved and ask the requester.
External evidence requires explicit `allow_external: true`.

## Token Budgets

| Boundary          | Budget                                                    | Escape hatch                                          |
| ----------------- | --------------------------------------------------------- | ----------------------------------------------------- |
| vault-query       | 5 pages in full; other matches title plus first paragraph | Record a gap and ask to expand                        |
| code exploration  | 8 files; roughly 150 relevant lines each                  | Stop, report partial coverage, identify skipped areas |
| MemPalace KG      | 10 relations per entity                                   | Record a gap                                          |
| research findings | 10 per track; 15 merged                                   | Merge related facts before dropping detail            |

## Vault Conventions

Details: [vault conventions](skills/vault-write/references/vault-conventions.md).

| #   | Rule                                                                                          | Enforcement                                                                                               | Tier  |
| --- | --------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------- | ----- |
| 0.1 | Topics have no date; Concepts have a date and parent Topic link                               | `vault-write` normalizes new drafts                                                                       | Hard  |
| 0.2 | `lifecycle` is `living` or `frozen`; frozen-page changes need approval                        | `vault-write` shows a diff                                                                                | Soft  |
| 0.3 | Wiki Purity: new pages contain no external-staging links, source queues, or lifecycle markers | `vault-write` rejects; `vault-index` avoids queues; `vault-lint` reports existing artifacts as `[LEGACY]` | Hard  |
| 0.4 | Archived/deprecated pages reside under `archive/`                                             | `vault-write` rejects misplaced writes; `vault-lint` reports existing mismatches                          | Hard  |
| 0.5 | An Entity with at least five Concept backlinks needs `00_Index.md`                            | `vault-lint` reports                                                                                      | Audit |
| 0.6 | Every top-level namespace is covered by an `index.md` Dataview `FROM` clause                  | `vault-lint` reports                                                                                      | Audit |

## Vault Paths

| Use                  | Value                                  |
| -------------------- | -------------------------------------- |
| Vault repo           | `~/Development/obsidian-vault/`        |
| Wiki semantic root   | `wiki/`                                |
| Wikilink             | `[[projects/<p>/Entities/<name>]]`     |
| MCP path, repo-bound | `wiki/projects/<p>/Entities/<name>.md` |
| macOS fallback       | `~/Development/obsidian-vault/wiki/`   |

Never use a bare `projects/...` MCP path in repo-bound mode. Existing legacy
Wiki Purity artifacts are lint-only findings and must not be mass-migrated.

## Validation

Before committing, run:

```bash
git diff --check
skills-ref validate skills/*/SKILL.md
npx skills add /Users/quetan/Development/nexus --list
```
