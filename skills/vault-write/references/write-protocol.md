# Write Protocol — Evidence-Gated Raw Capture

## Project Resolution Order

1. **Explicit** — user names the project
2. **Session** — already working in a project context
3. **cwd/git** — current directory or git remote
4. **Ask** — if ambiguous, ask the user

## Evidence Rules

| Tag | Meaning | Usage |
|-----|---------|-------|
| `user:` | Direct user statement | Safe to write as fact |
| `vault:` | From wiki lookup | Include source wikilink |
| `UNVERIFIED` | Agent inference | Never write as fact |

## Draft Format

```markdown
---
title: "<what was done>"
date: DD.MM.YYYY
project: <project-slug>
type: source
tags: [source, project/<slug>]
topics: [kebab-case, keywords]
---

# <Title>

## Коротко

<1-3 sentence summary in Russian>

## Что сделали

<bullet points of what was done>

## Ключевые решения

| Решение | Обоснование |
|---------|-------------|

## Sources Table

| Claim | Tag | Evidence |
|-------|-----|----------|
```

## Save Flow

1. Show path + draft + Sources table
2. Wait for: `save` / `edit` / `skip`
3. On `save`: create file → call vault-index → verify read back
