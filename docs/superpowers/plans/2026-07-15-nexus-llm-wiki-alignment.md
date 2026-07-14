# Nexus: LLM Wiki Alignment & Best Practices

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development or superpowers:executing-plans.

**Goal:** Align all 12 nexus skills with Karpathy's LLM Wiki pattern and agentskills.io best practices (gotchas, reference load conditions, "even if" edge cases).

**Architecture:** Edit all 12 SKILL.md files + vault-* references + AGENTS.md. Each task is a self-contained set of edits.

**Tech Stack:** Markdown, YAML frontmatter

## Global Constraints

- Every SKILL.md must remain ≤ 200 lines
- Every `name` must match parent directory
- Every `description` must start with "Use when..." and be ≤ 1024 chars
- All wikilinks: full vault-relative paths, no `.md`, no `../`

---

### Task 1: Gotchas — add to all 12 skills

**Files:** Modify all 12 `skills/*/SKILL.md`

**Interfaces:**
- Produces: `## Gotchas` section in every skill body

- [ ] **Edit each SKILL.md**: append `## Gotchas` section before `## Anti-patterns` or end of file.

**Gotchas content per skill:**

**nexus:**
```
## Gotchas
- Runtime ports, credentials, docker: palace manifest only — not markdown
- Wiki track is mandatory even if user says skip; record gap if vault empty
- Older record does not win — newer date always wins
```

**onboarding-harness:**
```
## Gotchas
- MCP failure does not block the workflow — continue with gaps
- Manifest may be stale; cross-check with actual project structure
```

**research-mempalace:**
```
## Gotchas
- drawer IDs are required for traceability — never omit
- KG query may return empty for new entities — record as gap, don't assume non-existence
- Legacy palace rooms may not exist — use manifest rooms only
```

**research-wiki:**
```
## Gotchas
- vault-query is vault-only by default — don't supplement with web/codegraph unless allowed
- wiki/index.md may be incomplete after many ingests — also search directly
```

**research-synthesize:**
```
## Gotchas
- Dates may be in different formats (DD.MM.YYYY vs YYYY-MM-DD) — normalize before comparing
- Equal dates or missing dates → ask user, never silently pick
```

**research-brief:**
```
## Gotchas
- Brief must be in Russian even if the conversation was in English
- Актуальное состояние section is mandatory — even if all fields are "not found"
```

**research-plan:**
```
## Gotchas
- Each step must have file-path + command + verify — no vague "fix the migration"
- Match language to user request; default Russian
```

**vault-write:**
```
## Gotchas
- Source dates use DD.MM.YYYY (match source directory), not YYYY-MM-DD
- seekstone may be unavailable — have file I/O fallback ready
- SHA256 computation failure → fallback to cache miss (never block)
```

**vault-ingest:**
```
## Gotchas
- One source should update 5-15 wiki pages — don't single-summary
- SHA256 cache prevents re-processing — check before reading source
- User must approve new pages — draft first
```

**vault-query:**
```
## Gotchas
- vault-only by default — orchestrator may override with allow_external
- If two vault sources disagree → ask user, don't pick newer
- If vault has no record — say so, don't invent
```

**vault-lint:**
```
## Gotchas
- Report only — never create, edit, or delete pages
- Only append to log.md (no other writes)
- seekstone down → report as unable to check
```

**vault-index:**
```
## Gotchas
- Read existing index content before append — avoid duplicates
- Never rewrite existing entries — append only
- Full vault paths only (no .md, no ../)
```

- [ ] **Verify** — `grep "## Gotchas" ~/nexus/skills/*/SKILL.md | wc -l` = 12
- [ ] **Commit**

---

### Task 2: Concrete reference load conditions

**Files:** Modify `skills/nexus/SKILL.md`, `skills/vault-write/SKILL.md`, `skills/vault-ingest/SKILL.md`, `skills/vault-query/SKILL.md`, `skills/vault-lint/SKILL.md`

- [ ] **Edit vault-write/SKILL.md**: Replace generic "Read: [references/write-protocol.md]" with conditional:
  - "Read [references/write-protocol.md] **before drafting a source note** — project resolution, evidence rules, save flow"
  - "Read [references/consistency-rules.md] **if unsure about date format or frontmatter fields**"
  - "Read [references/seekstone.md] **if seekstone MCP returns errors** — file I/O fallback details"

- [ ] **Edit vault-ingest/SKILL.md**: Replace "Read [references/ingest-protocol.md]" with:
  - "Read [references/ingest-protocol.md] **before decomposing a source** — cache check, overlap scan, page type mapping"

- [ ] **Edit vault-query/SKILL.md**: Replace "Read [references/query-protocol.md]" with:
  - "Read [references/query-protocol.md] **if the answer requires structured findings output** — format template and conflict gate"

- [ ] **Edit vault-lint/SKILL.md**: Replace "Read [references/lint-protocol.md]" with:
  - "Read [references/lint-protocol.md] **when running health checks** — check definitions and report format"

- [ ] **Edit nexus/SKILL.md**: Replace progressive disclosure table with triggers:
  - "Read `references/subagent-prompts.md` **before dispatching Phase 2a sub-agents**"
  - Keep existing trigger table but add "Read when..." phrasing

- [ ] **Commit**

---

### Task 3: LLM Wiki alignment fixes

**Files:** Modify `skills/vault-query/SKILL.md`, `skills/vault-ingest/SKILL.md`, `skills/vault-write/SKILL.md`, `AGENTS.md`

- [ ] **Edit vault-query/SKILL.md**: Add after synthesis step:
  > "After answering, if the answer contains durable knowledge (a comparison, analysis, or connection worth keeping), propose vault-write to file it back into the wiki as a new page."

- [ ] **Edit vault-ingest/SKILL.md**: Add before decomposition:
  > "After reading the source, discuss key takeaways with the user — what to emphasize, what to skip, what's surprising."

  And in constraints:
  > "One source should update 5-15 wiki pages across entities, concepts, decisions, and topics."

- [ ] **Edit vault-write/SKILL.md**: Add note:
  > "Use for preserving session work and external sources. Valuable query answers should also be captured — see vault-query."

- [ ] **Edit AGENTS.md**: Add schema co-evolution section:
  > "### Schema Co-evolution"
  > "AGENTS.md evolves with the wiki. After each significant ingest or after discovering a new pattern, update this file: add new conventions, fix inconsistencies, document workflows that worked."

- [ ] **Commit**

---

### Task 4: Fix AGENTS.md frontmatter field list

**Files:** Modify `AGENTS.md`

- [ ] **Edit AGENTS.md**: Replace the "Frontmatter Template" section:
  - Remove `requires-skill` and `metadata` from the list of required fields
  - Add note: `Requires:` can be embedded in `description` (current approach) or added as optional `metadata.requires` — consistent across all skills

- [ ] **Commit**

---

### Task 5: "even if" edge cases in descriptions

**Files:** Modify `skills/vault-write/SKILL.md`, `skills/vault-query/SKILL.md`, `skills/research-brief/SKILL.md`, `skills/research-plan/SKILL.md`, `skills/onboarding-harness/SKILL.md`, `skills/nexus/SKILL.md`

- [ ] **Edit vault-write description**: Append "even if the user doesn't explicitly say 'save' or 'write'"
- [ ] **Edit vault-query description**: Append "even if the user doesn't name the wiki explicitly"
- [ ] **Edit research-brief description**: Append "even for small or obvious tasks"
- [ ] **Edit research-plan description**: Append "even if the implementation seems straightforward"
- [ ] **Edit onboarding-harness description**: Append "even if the workflow was already run this session"
- [ ] **Edit nexus description**: Append "even if the user doesn't name a specific project"
- [ ] **Verify** — `grep "even if" ~/nexus/skills/*/SKILL.md` shows all 6
- [ ] **Commit**

---

### Task 6: Verify and final commit

- [ ] Run: `npx skills add ~/nexus --list` — expect 12 skills
- [ ] Run: `wc -l ~/nexus/skills/*/SKILL.md` — max under 200
- [ ] Run: `git -C ~/nexus status` — clean working tree
- [ ] Final commit: `git -C ~/nexus commit -m "feat: llm wiki alignment + best practices

- Add Gotchas sections to all 12 skills
- Add concrete reference load conditions
- Align with Karpathy LLM Wiki pattern
- Add 'even if' edge cases to descriptions
- Fix AGENTS.md frontmatter field list"`
