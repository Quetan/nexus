# Vault Conventions — Enforced Structural Rules

Шесть обязательных правил для структуры вики, жизненного цикла страниц и покрытия дашбордами.

## Применимость

- **Новые страницы:** правила применяются строго с момента внедрения
- **Существующие страницы:** нарушения репортятся vault-lint как `[LEGACY]`, миграция опциональна
- **Enforcement tiers:**
  - 🔴 **Hard gate:** vault-ingest/vault-write отклоняют операцию
  - 🟡 **Soft gate:** предупреждение + запрос подтверждения
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
- vault-ingest при создании Topic: `date` присутствует → reject creation

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

## 0.3 Raw Sources Status Prefix

**Правило:**
- Каждая запись в секции `## Raw Sources` проектного `00_Index.md` **обязана** иметь префикс:
  - `⏳ pending` — ссылка на файл в `raw/` (ещё не обработан)
  - `✅ ingested` — ссылка на итоговую wiki-страницу (обработан)
- Запись без префикса — **невалидна**

**Примеры:**
```markdown
## Raw Sources
- ⏳ pending [[raw/12.07.2026-discovery-session.md]]
- ✅ ingested [[library/MLOps/Concepts/prometheus-setup.md]]
- ❌ [[raw/notes.md]]  ← INVALID
```

**Enforcement:** 🔵 Audit-only
- vault-index при обновлении Raw Sources:
  - Если ссылка на `raw/` без префикса → автоматически добавить `⏳ pending`
- vault-lint:
  - Сканирует все `00_Index.md`, репортит строки без префикса как `[ERROR]`

---

## 0.4 Archive Convention

**Правило:**
- Страница с `status: archived` ИЛИ `status: deprecated` **физически обязана** находиться в `/archive/`:
  - `wiki/archive/...`
  - `wiki/projects/<project>/archive/...`
- Тег `#archived` **не заменяет** физический перенос

**Enforcement:** 🔴 Hard gate
- vault-ingest при создании страницы:
  - Если `status == "archived" | "deprecated"` AND путь не содержит `/archive/`:
    - **Отклонить** создание
    - Сообщить: "Archived pages must be in /archive/ directory"
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
  1. Извлечь все `[[Entity]]` wikilinks из `wiki/library/*/Concepts/**/*.md`
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
- Dashboard-файл: `wiki/index.md`
- Namespace, не входящий ни в один дашборд → **баг конфигурации**, а не "пока не нужно"

**Enforcement:** 🔵 Audit-only
- vault-lint проверка:
  1. Обнаружить все top-level namespaces:
     - `ls wiki/*/` (директории, содержащие `.md` файлы)
     - Исключить служебные: `.llm-wiki/`, `.obsidian/`
  2. Прочитать `wiki/index.md`, извлечь все dataview `FROM` clauses
  3. Проверить покрытие: каждый namespace упоминается в хотя бы одном `FROM`
  4. Репортить непокрытые: `[WARN] Uncovered namespace: wiki/<name>/ not in any FROM clause`

---

## Backward Compatibility

- **Новые страницы:** правила применяются строго
- **Существующие нарушения:** vault-lint репортит с пометкой `[LEGACY]`
- **Миграция:** опциональна, инициируется пользователем
