# Vault Conventions — Enforced Structural Rules

> Packaged copy for `vault-ingest` (one-level refs). Canonical source:
> `skills/vault-write/references/vault-conventions.md` — edit there and re-copy.

Шесть обязательных правил для структуры вики, жизненного цикла страниц и покрытия дашбордами.

## Применимость

- **Новые страницы:** правила применяются строго с момента внедрения
- **Существующие страницы:** нарушения репортятся vault-lint как `[LEGACY]`, миграция опциональна
- **Enforcement tiers:**
  - 🔴 **Hard gate:** vault-ingest/vault-write отклоняют операцию
  - 🟡 **Soft gate:** auto-fix with notification OR warning + confirmation prompt
  - 🔵 **Audit-only:** vault-lint репортит, не блокирует

---

## 0.1 Topic vs Concept

**Правило:**
- **Topic:** `type: topic`, **no `date` field**
- **Concept:** `type: concept`, **`date: YYYY-MM-DD` required**, must link from parent Topic

**Examples:**
- Topic: `wiki/library/MLOps/ml-configuration.md`
- Concept: `wiki/library/MLOps/Concepts/prometheus-custom-metrics.md`

**Enforcement:** 🔴 Hard gate
- vault-ingest при создании Concept: `date` отсутствует → auto-add current date
- vault-ingest при создании Topic: `date` присутствует → auto-remove with warning

**Rationale:** Topic = stable, dateless category. Concept = time-bound insight, requires date.
Symmetric remediation (auto-fix both directions) reduces friction while enforcing semantics.

---

## 0.2 lifecycle

**Правило:**
- **Значения:** `living | frozen`
- **Семантика:**
  - `frozen` — страница не подлежит автоматическому merge при vault-ingest (только с явным одобрением пользователя)
  - `living` — страница постоянно обновляется, автоматический merge разрешён

**Defaults:**
| Тип страницы | Default lifecycle |
|--------------|-------------------|
| Decision | `frozen` (всегда) |
| Concept (с датой) | `frozen` (если не указано `living`) |
| Topic | `living` (всегда) |
| Entity | `living` |
| Task | `living` |

**Enforcement:** 🟡 Soft gate
- vault-ingest перед merge:
  ```
  if page.lifecycle == "frozen":
      show_diff_to_user()
      if not user_approves():
          skip_merge()
  else:
      auto_merge()
  ```
- Ручное редактирование frozen-страниц **разрешено** всегда

---

## 0.3 Wiki Pages Never Reference Raw/

**Правило:**
- Хранилище `raw/` — это **staging area** для ручных/внешних заметок, ингрессируемых через `vault-ingest`
- **Wiki-страницы никогда не содержат ссылок** на `raw/` — ни в теле, ни в Related, ни в Sources
- Исключение — секция `## Raw Sources` проектного `00_Index.md`, которая является очередью инжеста:
  - `⏳ pending` — ссылка на файл в `raw/` (ещё не обработан, ждёт ручного ingest)
  - `✅ ingested` — **НЕ используется в этой секции для ссылки на raw**; вместо этого: `✅ [[projects/<p>/Entities/foo]]` указывает на итоговую wiki-страницу

**Примеры:**
```markdown
## Raw Sources
- ⏳ pending [[raw/12.07.2026/discovery-session]]         ← OK, ждёт ingest
- ingested → [[library/MLOps/Concepts/prometheus-setup]]  ← OK, после ingest: wiki-only
- ❌ [[raw/notes.md]] без префикса                        ← INVALID
- ❌ ✅ ingested [[raw/12.07.2026/foo]]                   ← INVALID, raw-links never ✅
- ❌ ✅ ingested [[raw/12.07.2026/foo]] → [[projects/...]] ← INVALID, no raw allowed even with arrow
```

**Enforcement:** 🟡 Soft gate (auto-fix + audit)
- vault-write **отклонит** draft содержащий wikilink на `raw/` — raw — не ссылочный формат для wiki
- vault-index при обновлении Raw Sources:
  - Новая `raw/`-ссылка без префикса → автоматически добавить `⏳ pending`
  - После `vault-ingest`: **удалить** ⏳-линию целиком; в нужные секции (Entities / Concepts / Decisions / Tasks / Topics) добавить `✅`-free строки `- ingested → [[projects/<p>/...]], [[projects/<p>/...]]`. Raw-ссылки исчезают из wiki после ingest.
- vault-lint:
  - Сканирует все `00_Index.md`, репортит строки без префикса как `[ERROR]`
  - Сканирует все wiki-страницы на ссылку `[[raw/` вне секции Raw Sources → `[ERROR] Page references raw: <path>`

---

## 0.4 Archive Convention

**Правило:**
- Страница с `status: archived` ИЛИ `status: deprecated` **физически обязана** находиться в `/archive/`:
  - `wiki/archive/...`
  - `wiki/projects/<project>/archive/...`
- Тег `#archived` **не заменяет** физический перенос

**Enforcement:** 🔴 Hard gate
- vault-ingest/vault-write при создании страницы ИЛИ изменении `status` → `archived|deprecated`:
  - Если путь не содержит `/archive/`:
    - **Отклонить** операцию
    - Сообщить: "Archived pages must be in /archive/ directory"
  - Applies to: `write_note`, `update_frontmatter`
- vault-lint:
  - Найти все страницы с `status: archived | deprecated`
  - Проверить путь
  - Репортить несоответствия: `[FAIL] Misplaced archived page: <path>`

---

## 0.5 Hub Rule (N≥5)

**Правило:**
- Если **≥5 Concept-страниц** ссылаются на одну и ту же Entity, и для этой Entity **нет** `00_Index.md` → **это находка**, требующая создания хаба

**Threshold:** N=5 (hardcoded)

**Enforcement:** 🔵 Audit-only
- vault-lint проверка:
  1. Извлечь все `[[Entity]]` wikilinks из:
     - `wiki/library/*/Concepts/**/*.md`
     - `wiki/projects/*/Concepts/**/*.md`
  2. Подсчитать backlinks для каждой Entity
  3. Для каждой Entity с ≥5 backlinks:
     - Проверить существование `00_Index.md`:
       - `wiki/entities/<Entity>/00_Index.md` ИЛИ
       - `wiki/projects/*/entities/<Entity>/00_Index.md`
     - Если отсутствует → репортить: `[WARN] Hub needed: [[Entity]] (N backlinks, no 00_Index.md)`
- **Действие пользователя:** создать хаб вручную (не автоматически)

---

## 0.6 Dataview Coverage

**Правило:**
- Любой существующий top-level namespace (`wiki/projects/`, `wiki/library/`, `wiki/entities/`, `wiki/topics/`, любые будущие) **обязан** быть включён в `FROM`-условия хотя бы одного dashboard
- Dashboard-файл: `index.md` в корне vault
- Namespace, не входящий ни в один дашборд → **баг конфигурации**, а не "пока не нужно"

**Enforcement:** 🔵 Audit-only
- vault-lint проверка:
  1. Обнаружить все top-level namespaces:
     - `ls wiki/*/` (директории, содержащие `.md` файлы)
     - Исключить служебные: `.llm-wiki/`, `.obsidian/`
  2. Прочитать `index.md` (vault root), извлечь все dataview `FROM` clauses
  3. Проверить покрытие: каждый namespace упоминается в хотя бы одном `FROM`
  4. Репортить непокрытые: `[WARN] Uncovered namespace: wiki/<name>/ not in any FROM clause`

---

## Backward Compatibility

- **Новые страницы:** правила применяются строго
- **Существующие нарушения:** vault-lint репортит с пометкой `[LEGACY]`
- **Миграция:** опциональна, инициируется пользователем

## Vault Root Convention

- Wikilinks всегда от корня vault — без префикса `wiki/`
- `wiki/` является корнем vault: `[[projects/surgut/Entities/foo]]`, не `[[wiki/projects/surgut/Entities/foo]]`
- `raw/` — соседняя с `wiki/` директория; она не линкуется из wiki-страниц
