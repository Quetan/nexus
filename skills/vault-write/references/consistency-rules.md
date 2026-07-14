# Consistency Rules

## Date Format

| Page type | Format | Example |
|-----------|--------|---------|
| Source | `DD.MM.YYYY` | `12.07.2026` |
| Wiki pages (entity, concept, decision, task) | `YYYY-MM-DD` | `2026-07-12` |

## File Naming

| Page type | Pattern | Example |
|-----------|---------|---------|
| Source | `<slug>.md` | `discovery-results.md` |
| Entity | `<entity-name>.md` | `FavoriteDir.md` |
| Concept | `<concept-name>.md` | `ServiceMesh.md` |
| Decision | `<slug-DD-MM-YYYY>.md` | `db-schema-v2-12-07-2026.md` |
| Index | `00_Index.md` | `00_Index.md` |

## Frontmatter

### Source
```yaml
---
title: "What was done"
date: DD.MM.YYYY
project: <project-slug>
type: source
topics: [kebab-case, keywords]
---
```

### Entity
```yaml
---
title: "<name>"
date: YYYY-MM-DD
project: <project-slug>
type: entity
entity_type: service | person | tool | team
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
tags: [decision, project/<slug>]
related: []
---
```

### General Rules
- `title`: always quoted, required
- `date`: always quoted, format per page type
- `project`: lowercase-dashes slug, required
- `type`: source/entity/concept/decision/task/index
