# Consistency Rules

## Date Format

⚠️ **Note:** Decision/Task pages use **two formats**:
- Frontmatter `date` field: `YYYY-MM-DD` (example: `"2026-07-12"`)
- Filename suffix: `DD-MM-YYYY` (example: `db-schema-12-07-2026.md`)

This is intentional — Entity/Concept/Topic don't use date in filename at all.

| Page type | Format | Example |
|-----------|--------|---------|
| Raw staging filename | `DD.MM.YYYY` (directory) | `raw/12.07.2026/surgut/<slug>.md` |
| Wiki pages (frontmatter) | `YYYY-MM-DD` | `2026-07-12` |

## File Naming

| Page type | Pattern | Example |
|-----------|---------|---------|
| Raw staging source | `<slug>.md` inside `raw/DD.MM.YYYY/<project>/` | `discovery-results.md` |
| Entity | `<entity-name>.md` | `FavoriteDir.md` |
| Concept | `<concept-name>.md` | `ServiceMesh.md` |
| Decision | `<slug-DD-MM-YYYY>.md` | `db-schema-v2-12-07-2026.md` |
| Topic | `<kebab-case-name>.md` | `ml-configuration.md` |
| Task | `<slug-DD-MM-YYYY>.md` | `server-audit-04-07-2026.md` |
| Index | `00_Index.md` | `00_Index.md` |
| Log | `log.md` | `log.md` |

## Frontmatter

### Entity
```yaml
---
title: "<name>"
date: YYYY-MM-DD
project: <project-slug>
type: entity
entity_type: service | person | tool | team | library
status: active
tags: [entity, project/<slug>]
related: []
---
```

### Concept
```yaml
---
title: "<name>"
date: YYYY-MM-DD
project: <project-slug>
type: concept
status: active
lifecycle: frozen   # default for dated concepts
tags: [concept, project/<slug>]
related: []
---
```

### Decision
```yaml
---
title: "<decision title>"
date: YYYY-MM-DD
project: <project-slug>
type: decision
status: active
lifecycle: frozen   # always for decisions
tags: [decision, project/<slug>]
related: []
---
```

### Topic
```yaml
---
title: "<topic name>"
project: <project-slug>
type: topic
status: active
lifecycle: living   # always for topics
tags: [topic, project/<slug>]
---
```

Note: Topic frontmatter has **no `date` field** (convention 0.1).

### Task
```yaml
---
title: "<task title>"
date: YYYY-MM-DD
project: <project-slug>
type: task
status: active
tags: [task, project/<slug>]
related: []
---
```

There is NO `type: source` template — sources no longer land in `wiki/` directly.

## General Rules
- `title`: always quoted, required
- `date`: always quoted, format per page type
- `project`: lowercase-dashes slug, required
- `type`: entity/concept/decision/task/index/topic
- `lifecycle`: frozen|living — defaults per convention 0.2

## Path Mapping

Vault root = `wiki/` directory inside the obsidian-vault repo. All wikilinks
are **vault-root-relative** — **never** use a `wiki/` prefix.

`project` field value → vault path:
- Any project slug `<s>` → `projects/<s>/`
- `library/<topic>` → `library/<Topic>/`

**Examples:** `surgut` → `projects/surgut/Entities/cc-api-sng`, `MLOps` → `library/MLOps/Concepts/prometheus-setup`

**Wikilink format (in content):**
- ✓ `[[projects/surgut/Entities/cc-api-sng]]`
- ✓ `[[library/MLOps/Concepts/prometheus-setup]]`
- ✓ `[[library/00_Index|Library]]`
- ✗ `[[wiki/projects/surgut/Entities/cc-api-sng]]` — never
- ✗ `[[./projects/surgut/...]]` — no relative `../` or `.` prefixes

**Physical file locations (relative to repo root):**
- Wiki pages: `wiki/projects/...`, `wiki/library/...`, `wiki/index.md`, `wiki/log.md`
- Raw staging: `raw/DD.MM.YYYY/<project>/<slug>.md` (managed only by `vault-ingest`)

## General Rules
- `title`: always quoted, required
- `date`: always quoted, format per page type
- `project`: lowercase-dashes slug, required
- `type`: source/entity/concept/decision/task/index/topic
