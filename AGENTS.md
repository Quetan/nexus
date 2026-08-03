# AGENTS.md — nexus: atomic skills refactoring

## Project Overview

Refactor `surgut-research` and `obsidian-task-note` into a modular atomic skill system in the `~/nexus` repository.

**Goal:** Replace two monolithic skills (~565 combined lines) with 12 atomic single-purpose skills in one repo. Each skill follows the Agent Skills spec (agentskills.io), is <200 lines, uses JIT-loaded references for token economy.

**Current lifecycle state (2026-08):** vault-write performs direct wiki ingest (no raw staging for session work). vault-ingest is reserved for manual/external raw/ drops. All wikilinks are vault-root-relative — `wiki/` is the vault root, never used as a prefix.

## Architecture Decisions (from Q&A)

| Decision | Choice |
|----------|--------|
| Coupling level | Light — skills can call each other via `requires-skill` |
| Aggregator name | `nexus` (generic, not project-bound) |
| Read/write scope | `nexus` is read-only; vault write/ingest delegated to atomic skills |
| Wiki automation | Semi-automatic — agent proposes drafts, user approves (`save`/`edit`/`skip`) |
| Token priority | Balance — SKILL.md ~200-400 lines, references JIT-loaded |
| Project resolution | Auto-then-ask: git/cwd first, ask if ambiguous |
| Session capture path | Direct `vault-write` → `wiki/` (no `raw/` stage) |
| Manual source path | `raw/` → `vault-ingest` → `wiki/` |
| Wikilink format | Vault-root-relative: `[[projects/<p>/Entities/foo]]`, never `[[wiki/...]]` |
| Vault location | Resolved via mcpvault MCP; fallback `~/Development/obsidian-vault/wiki/` |

## Repo Structure

```
~/nexus/
├── AGENTS.md                   # This file
├── README.md                   # Install & usage overview
├── docs/superpowers/specs/
│   └── YYYY-MM-DD-nexus-atomic-skills-design.md
├── skills/
│   ├── nexus/                  # Orchestrator — coordinates flow
│   │   ├── SKILL.md            # Which atomic skills to load when
│   │   └── references/         # Phase templates, prompts
│   ├── research-mempalace/     # Extract context from MemPalace
│   ├── research-wiki/          # Extract context from Obsidian wiki
│   ├── research-synthesize/    # Merge sources, conflict resolution
│   ├── research-brief/         # Generate Research Brief (Russian)
│   ├── research-plan/          # Generate Implementation Plan
│   ├── vault-write/            # Direct wiki capture (typed pages, no raw staging)
│   ├── vault-ingest/           # Manual raw/ → wiki pages
│   ├── vault-query/            # Medium vault research
│   ├── vault-lint/             # Wiki health check
│   ├── vault-index/            # index.md + log.md + 00_Index management
│   └── onboarding-harness/     # MCP + manifest checks
```

## Atomic Skills Specification

### Frontmatter Template (each SKILL.md)
```yaml
---
name: kebab-case-name          # Must match parent directory
description: >                 # "Use when..." triggers; embed Requires: inline
  Use when (trigger-1), (trigger-2), ... (trigger-N).
  Requires: skill-1, skill-2 (if any)
---
```

Note: `Requires:` is embedded in the `description` field (not a standalone frontmatter key). This is consistent across all 12 skills. The optional `metadata` and `license` fields from agentskills.io spec are used only when needed.

### Rules per skill
- Single-purpose: does ONE thing well
- Imperative mood ("Validate input", not "You should consider validating")
- No context the model already has (assume high baseline capability)
- Progressive disclosure: metadata in frontmatter, core in SKILL.md, details in references/
- References: one level deep only. Loaded JIT when explicitly referenced.
- SKILL.md ≤ 200 lines (≤ 5000 tokens). References ≤ 200 lines each.

## Execution Plan (Phase 2 — Writing Plans)

### Phase A: Foundation
1. Write design doc to `docs/superpowers/specs/`
2. Write AGENTS.md (this file)
3. Write all 13 SKILL.md files
4. Write README.md

### Phase B: Progressive disclosure
5. Create references/ for skills that need them
   - nexus/references/ — subagent prompts, templates
   - vault-write/references/ — write-protocol, evidence rules, formatting
   - vault-ingest/references/ — ingest-protocol, cache
   - vault-query/references/ — query-protocol
   - vault-lint/references/ — lint-protocol

### Phase C: Verification
6. `npx skills add ~/nexus --list` — verify all 13 skills detected
7. `npx skills add ~/nexus -g -a claude-code -y` — global install test
8. README.md with install + usage

### Phase D: Cleanup
9. Remove old `surgut-research` symlink/install (optional)
10. Commit to git

## Quality Gates

Before each commit:
- [ ] `npx skills add ~/nexus --list` — all 13 skills listed
- [ ] No file > 300 lines (references excluded)
- [ ] Every frontmatter has `name` + `description`
- [ ] No duplicate `name` across skills
- [ ] `nexus` SKILL.md references all 12 sub-skills by name
- [ ] Each sub-skill description contains exact trigger phrases

## Naming Conventions

| Pattern | Example |
|---------|---------|
| Skill directory | `kebab-case` |
| Skill name (frontmatter) | `kebab-case` |
| Reference files | `kebab-case.md` |
| Frontmatter fields | `name`, `description` |
| trigger phrases in description | Use when (1) ..., (2) ... |
| Requires-skill notation | Embedded in `description` as `Requires: skill-a, skill-b` |
| Optional fields | `metadata`, `license`, `compatibility` (per agentskills.io spec) |

## Vault Conventions — Enforced Structural Rules

Шесть обязательных правил для wiki-структуры, жизненного цикла и покрытия.  
Детали: `skills/vault-write/references/vault-conventions.md`

| # | Правило | Enforcement | Tier |
|---|---------|-------------|------|
| 0.1 | **Topic vs Concept**: Topic без даты, Concept с датой (auto-add) | vault-ingest auto-adds date to Concepts | 🔴 Hard |
| 0.2 | **lifecycle**: `living\|frozen` — Decisions/dated Concepts default frozen, merge требует одобрения для frozen | vault-ingest checks before auto-merge | 🟡 Soft |
| 0.3 | **Raw Sources prefix + no raw refs in wiki**: `⏳ pending` для новых raw; после ingest ⏳-линия удаляется, ✅-ссылки ведут на итоговые wiki-страницы; wiki-страницы никогда не ссылаются на raw/ вне секции Raw Sources | vault-write rejects drafts with raw links, vault-index auto-fixes prefix and rewrites post-ingest, vault-lint reports stray raw refs | 🔴 Hard |
| 0.4 | **Archive convention**: `status: archived\|deprecated` → физически в `/archive/` | vault-ingest rejects, vault-lint reports | 🔴 Hard |
| 0.5 | **Hub rule**: Entity с ≥5 backlinks нуждается в `00_Index.md` | vault-lint reports (manual fix) | 🔵 Audit |
| 0.6 | **Dataview coverage**: каждый namespace в `index.md` FROM (vault root) | vault-lint reports gaps | 🔵 Audit |

**Backward compatibility:** Правила применяются к новым страницам. Существующие нарушения репортятся как `[LEGACY]`.

## Vault State (canonical, 2026-08-03)

| Item | Value |
|------|-------|
| Vault repo | `~/Development/obsidian-vault/` (macOS) |
| Vault root (used by all wikilinks & its paths) | `wiki/` inside that repo |
| Raw staging | `raw/DD.MM.YYYY/<project>/<slug>.md` next to `wiki/` |
| Legacy Linux path | `/srv/cloud/obsidian` — DEPRECATED, do not reference |
| Session capture flow | `vault-write` → `wiki/<...>` (no `raw/` step) |
| Manual raw ingest flow | `raw/` → `vault-ingest` → `wiki/` |
| Wikilink format | `[[projects/<p>/<Bucket>/<name>]]` — no `wiki/` prefix, no `.md` |
| Vault access | mcpvault MCP preferred; file I/O fallback to `~/Development/obsidian-vault/wiki/` |
| Known legacy | `wiki/log.md` + some `00_Index.md`s contain historical `[[raw/...]]` wikilinks; treated as `[LEGACY]` by rule 0.3, no mass migration |

## Schema Co-evolution

AGENTS.md evolves with the wiki and the skills. After each significant ingest or after discovering a pattern that works (or doesn't), update this file:

- Add new conventions discovered during implementation
- Fix inconsistencies between declared rules and actual practice
- Document workflows that worked well for future sessions
- Remove assumptions that proved wrong

The agent should treat AGENTS.md as a living document — not immutable law, but the current best understanding. When in doubt, ask the user.

## Source Materials

- Existing skills to extract logic from:
  - `~/.agents/skills/surgut-research/SKILL.md`
  - `~/.agents/skills/obsidian-task-note/SKILL.md`
  - `~/.agents/skills/obsidian-task-note/references/*.md`
