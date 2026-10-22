# Специфікація вимог: MGS Lore Database

## 1. EARS Requirements

| ID | Патерн | Синтаксис вимоги |
| :--- | :--- | :--- |
| **REQ-UB-01** | Ubiquitous | The system SHALL provide read-only access for Archivist without manual data mutation capabilities. |
| **REQ-UB-02** | Ubiquitous | The system SHALL maintain a logical many-to-many ($M:N$) relation between `Character`/`Snake` and `Game`. |
| **REQ-UB-03** | Ubiquitous | The system SHALL enforce strict uniqueness on `chron_order` in `Game`. |
| **REQ-EV-01** | Event-driven | WHEN an update arrives from `Konami Archive API`, the system SHALL verify local `final_status` against incoming canonical data. |
| **REQ-EV-02** | Event-driven | WHEN a status mismatch is detected, the system SHALL apply API values and raise `Flag Non-Canon Conflict`. |
| **REQ-EV-03** | Event-driven | WHEN Archivist filters by `role_type`, `final_status`, and `is_clone`, the system SHALL return only entities matching all criteria. |
| **REQ-EV-04** | Event-driven | WHEN Archivist requests timeline view, the system SHALL sort games ascending by `chron_order` regardless of `release_year`. |
| **REQ-ST-01** | State-driven | WHILE entity synchronization with API is active, the system SHALL lock the corresponding record against concurrent mutations. |
| **REQ-ST-02** | State-driven | WHILE `Flag Non-Canon Conflict` is active on an entity, the system SHALL display a conflict warning banner in the character dossier. |
| **REQ-OP-01** | Optional | WHERE `Konami Archive API` is unreachable, the system SHALL serve cached data with a `Stale Data` indicator. |
| **REQ-OP-02** | Optional | WHERE a text search query is submitted, the system SHALL perform partial matching on `Character.name` or `Snake.codename`. |
| **REQ-OOS-01** | Out-of-Scope | WHEN timeline/lore paradoxes occur, the system SHALL automatically resolve narrative plot holes via generative AI. |

---

## 2. User Stories & Gherkin Scenarios

### US-01: Хронологічний таймлайн
**As an** Archivist,  
**I want to** переглядати ігри за внутрішнім сюжетом `chron_order`,  
**So that** відстежувати події канону незалежно від року релізу.

```gherkin
Scenario: Відображення сюжетної хронології
  Given у базі є "MGS3" (chron_order: 1, release_year: 2004) та "MG1" (chron_order: 2, release_year: 1987)
  When Archivist запитує таймлайн
  Then першою у списку повертається "MGS3", другою — "MG1"
```

### US-02: Каталогізація клонів Снейка
**As an** Archivist,  
**I want to** фільтрувати список за прапорцем `is_clone` та статусом,  
**So that** швидко ідентифікувати живих клонів Big Boss.

```gherkin
Scenario: Фільтрація клонів за статусом
  Given у системі є Snake "Solid Snake" (is_clone: true, final_status: "Retired")
  And є Snake "Liquid Snake" (is_clone: true, final_status: "Deceased")
  When встановлено фільтр is_clone = true та final_status = "Retired"
  Then повертається лише "Solid Snake"
```

### US-03: Синхронізація канону та фіксація конфлікту
**As the** Konami Archive API,  
**I want to** пушити оновлені статуси сутностей,  
**So that** локальна база підтримувала актуальний канон франшизи.

```gherkin
Scenario: Звірка статусу з пріоритетом API
  Given персонаж має локальний final_status "MIA"
  When Konami Archive API надсилає оновлення final_status = "Deceased"
  Then локальний final_status замінюється на "Deceased"
  And системний атрибут встановлює Flag Non-Canon Conflict = true
```

### US-04: Офлайн-режим читача
**As an** Archivist,  
**I want to** читати каталог за відсутності мережі,  
**So that** мати безперервний доступ до досьє.

```gherkin
Scenario: Завантаження закешованих даних
  Given Konami Archive API недоступний
  When Archivist відкриває досьє персонажа
  Then система завантажує закешований запис
  And виводить банер "Stale Data"
```

---

## 3. Матриця трасування (Traceability Matrix)

| EARS ID | Use Case(s) | Покриття |
| :--- | :--- | :---: |
| **REQ-UB-01** | `UC-1` (Дослідження каталогу), `UC-1.1`, `UC-1.2`, `UC-1.3`, `UC-1.4` | In-Scope |
| **REQ-UB-02** | `UC-1.1`, `UC-1.2`, `UC-1.4` | In-Scope |
| **REQ-UB-03** | `UC-1.1`, `UC-2` (Синхронізація метаданих) | In-Scope |
| **REQ-EV-01** | `UC-2`, `UC-4` (Блокування запису) | In-Scope |
| **REQ-EV-02** | `UC-2`, `UC-3` (Фіксація конфлікту канону), `UC-1.4` | In-Scope |
| **REQ-EV-03** | `UC-1`, `UC-1.2` (Фільтрація персонажів) | In-Scope |
| **REQ-EV-04** | `UC-1`, `UC-1.1` (Перегляд таймлайну) | In-Scope |
| **REQ-ST-01** | `UC-2`, `UC-4` | In-Scope |
| **REQ-ST-02** | `UC-1.4` (Перегляд досьє), `UC-3` | In-Scope |
| **REQ-OP-01** | `UC-1.4`, `UC-5` (Попередження про застарілість) | In-Scope |
| **REQ-OP-02** | `UC-1`, `UC-1.3` (Пошук за назвою/кодовим ім'ям) | In-Scope |
| **REQ-OOS-01**| *Не покрито прецедентами (0 зв'язків)* | **Out-of-Scope** |

> **Примітка щодо Out-of-Scope (REQ-OOS-01):**  
> Автоматичне виправлення сюжетних дір та канонічних парадоксів за допомогою ШІ свідомо виключено з архітектури. Система обмежується фіксацією конфлікту (`Flag Non-Canon Conflict`), а інтерпретація розбіжностей залишається виключно за читачем.