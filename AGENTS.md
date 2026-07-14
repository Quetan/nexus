# AGENTS.md — nexus: atomic skills refactoring

## Project Overview

Refactor `surgut-research` and `obsidian-task-note` into a modular atomic skill system in the `~/nexus` repository.

**Goal:** Replace two monolithic skills (~565 combined lines) with ~13 atomic single-purpose skills in one repo. Each skill follows the Agent Skills spec (agentskills.io), is <200 lines, uses JIT-loaded references for token economy.

## Architecture Decisions (from Q&A)

| Decision | Choice |
|----------|--------|
| Coupling level | Light — skills can call each other via `requires-skill` |
| Aggregator name | `nexus` (generic, not project-bound) |
| Read/write scope | `nexus` is read-only; vault write/ingest delegated to atomic skills |
| Wiki automation | Semi-automatic — agent proposes drafts, user approves (`save`/`edit`/`skip`) |
| Token priority | Balance — SKILL.md ~200-400 lines, references JIT-loaded |
| Project resolution | Auto-then-ask: git/cwd first, ask if ambiguous |

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
│   ├── vault-write/            # Evidence-gated raw capture → sources/
│   ├── vault-ingest/           # Source → wiki pages
│   ├── vault-query/            # Medium vault research
│   ├── vault-lint/             # Wiki health check
│   ├── vault-index/            # index.md + log.md + 00_Index management
│   └── onboarding-harness/     # MCP + manifest checks
```

## Atomic Skills Specification

### Frontmatter Template (each SKILL.md)
```yaml
---
name: kebab-case-name
description: >
  Use when (trigger-1), (trigger-2), ... (trigger-N).
  Requires: skill-1, skill-2 (if any)
---
```

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
| Frontmatter fields | `name`, `description`, `requires-skill`, `metadata` |
| trigger phrases in description | Use when (1) ..., (2) ... |

## Source Materials

- Existing skills to extract logic from:
  - `~/.agents/skills/surgut-research/SKILL.md`
  - `~/.agents/skills/obsidian-task-note/SKILL.md`
  - `~/.agents/skills/obsidian-task-note/references/*.md`
