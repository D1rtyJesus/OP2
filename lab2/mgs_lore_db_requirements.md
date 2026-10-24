# Специфікація вимог: MGS Lore Database

## 1. Схема даних та сутності

* **`Character`**: `id` (PK, string), `name` (string), `role_type` (string), `final_status` (string), `has_canon_conflict` (boolean, default: `false`).
* **`Snake`**: `id` (PK, string), `character_id` (FK -> `Character.id`), `codename` (string), `is_clone` (boolean).
* **`Game`**: `id` (PK, string), `title` (string), `release_year` (number), `chron_order` (number, unique).
* **`Character_Game`**: логічна асоціація багато-до-багатьох ($M:N$) між сутностями `Character` та `Game`.

---

## 2. EARS Requirements

| ID | Патерн | Синтаксис вимоги |
| :--- | :--- | :--- |
| **REQ-UB-01** | Ubiquitous | The system SHALL provide read-only access for `Reader` and `Archivist` without manual mutation capabilities. |
| **REQ-UB-02** | Ubiquitous | The system SHALL maintain a logical many-to-many ($M:N$) relation between `Character` and `Game`. |
| **REQ-UB-03** | Ubiquitous | The system SHALL enforce strict uniqueness on `chron_order` within the `Game` registry. |
| **REQ-EV-01** | Event-driven | WHEN an update payload arrives from `Konami Archive API`, the system SHALL verify incoming status against `Character.final_status`. |
| **REQ-EV-02** | Event-driven | WHEN `final_status` from API diverges from local state, the system SHALL apply incoming data and set `Character.has_canon_conflict = true`. |
| **REQ-EV-03** | Event-driven | WHEN an update targets a non-existent `character_id`, the system SHALL reject the operation and return HTTP 404 Not Found. |
| **REQ-EV-04** | Event-driven | WHEN an actor queries the catalog by `role_type`, `final_status`, or `Snake.is_clone`, the system SHALL return matching entries via relation `Snake.character_id = Character.id`. |
| **REQ-EV-05** | Event-driven | WHEN an actor requests the timeline view, the system SHALL sort records strictly by `Game.chron_order` ascending. |
| **REQ-ST-01** | State-driven | WHILE entity synchronization with API is in progress, the system SHALL reject concurrent modification requests with HTTP 409 Conflict. |
| **REQ-ST-02** | State-driven | WHILE `Character.has_canon_conflict == true`, the system SHALL render a canonical conflict notice on the character dossier view. |
| **REQ-OP-01** | Optional | WHERE `Konami Archive API` is unreachable, the system SHALL serve cached data with HTTP header `X-Data-Status: STALE`. |
| **REQ-OP-02** | Optional | WHERE search terms are entered, the system SHALL evaluate case-insensitive partial matches against `Character.name` and `Snake.codename`. |

---

## 3. Use Case Deep Dive: UC-2 «Синхронізація метаданих»

**Use Case ID:** UC-2  
**Назва:** Синхронізація метаданих персонажів та статусів канону  
**Актор:** Konami Archive API (зовнішня система)  
**Короткий опис:** Оновлення значень `final_status` персонажа на основі авторитетного джерела з виявленням розбіжностей та ізоляцією транзакцій.

```gherkin
Feature: Синхронізація метаданих персонажа через Konami Archive API

  Background:
    Given у базі даних зареєстровано Character з id "char-bb", name "Big Boss", final_status "MIA", has_canon_conflict false

  Scenario: 1. Happy path — статус збігається, успішна синхронізація
    Given зовнішнє API надсилає запит PUT /api/v1/characters/char-bb/sync з final_status "MIA"
    When система обробляє запит синхронізації
    Then система повертає HTTP-відповідь зі статусом 200 OK
    And значення Character.final_status залишається "MIA"
    And значення Character.has_canon_conflict дорівнює false

  Scenario: 2. Conflict path — статус розійшовся, оновлення з фіксацією розбіжності
    Given зовнішнє API надсилає запит PUT /api/v1/characters/char-bb/sync з final_status "Deceased"
    When система обробляє запит синхронізації
    Then система оновлює Character.final_status на "Deceased"
    And система встановлює Character.has_canon_conflict = true
    And система повертає HTTP-відповідь зі статусом 200 OK із прапорцем canon_conflict: true

  Scenario: 3. Lock / edge case — оновлення під час активного блокування запису
    Given запис Character з id "char-bb" наразі заблокований тривалою транзакцією оновлення
    When зовнішнє API надсилає запит PUT /api/v1/characters/char-bb/sync з final_status "Deceased"
    Then система відхиляє виконання операції
    And система повертає HTTP-відповідь зі статусом 409 Conflict
    And локальний запис Character залишається без змін

  Scenario: 4. Error path — неіснуючий character_id
    When зовнішнє API надсилає запит PUT /api/v1/characters/char-unknown/sync з final_status "Active"
    Then система фіксує відсутність сутності з вказаним ідентифікатором
    And система повертає HTTP-відповідь зі статусом 404 Not Found
    And жодні модифікації даних у системі не виконуються
```

---

## 4. Нефункціональні вимоги (NFR)

* **NFR-PLATFORM-01 (Платформи та браузери):** Веб-клієнт системи повинен коректно рендеритися та зберігати повну функціональність у сучасних веб-браузерах версій: Chromium (Chrome, Edge) $\ge$ 115, Mozilla Firefox $\ge$ 118, Apple Safari $\ge$ 17.
* **NFR-LOCALE-01 (Мова та локалізація):** Усі структури даних, службові REST API та заголовки передаються англійською мовою (`en-US`). Інтерфейс користувача та повідомлення про статус канону відображаються українською мовою (`uk-UA`).
* **NFR-PERF-01 (Час відгуку):** 95-й процентиль ($p95$) часу обробки запитів на читання (`GET /characters`, `GET /timeline`) не повинен перевищувати $150\text{ мс}$ при нормальному експлуатаційному навантаженні.
* **NFR-RATE-01 (Ліміти навантаження):** Шлюз системи повинен витримувати навантаження до $50\text{ RPS}$ (запитів/сек) на публічні ендпоінти. При перевищенні ліміту система повинна повертати статус HTTP 429 Too Many Requests із заголовком `Retry-After`.
* **NFR-CACHE-01 (Кешування та автономність):** Час життя локального кешу метаданих (TTL) становить строго $24\text{ години}$. У разі недоступності `Konami Archive API` відповіді формуються з кешу та маркуються заголовком `X-Data-Status: STALE`.

---

## 5. Межі системи: Out-of-Scope

Наступні можливості явно виключені з поточного релізу:

1. **Генеративні моделі розв'язання сюжетних суперечностей:** Відхилено застосування Generative AI для автоматичної інтерпретації сюжетних дір, парадоксів таймлайну чи ретконів Кодзіми. Системна роль обмежена фіксацією колізії (`has_canon_conflict = true`); інтерпретація залишається на розсуд читача.
2. **Неканонічні відгалуження (Spin-offs):** Ігри серій *Metal Gear Survive*, *Metal Gear Acid 1/2*, *Snake's Revenge* та мобільні версії виключені з моделі `Game` і не враховуються при побудові атрибута `chron_order`.
3. **Ручне редагування бази (CMS/Admin panel):** Кінцеві користувачі не мають інтерфейсу для внесення правок; єдиним джерелом змін є контракт із зовнішнім API.

---

## 6. Матриця трасування (Traceability Matrix)

| EARS ID | Прецедент (Use Case) | Сценарій Gherkin | Статус |
| :--- | :--- | :--- | :---: |
| **REQ-UB-01** | `UC-1: Дослідження каталогу` | Покривається загальними перевірками доступу Read-only | In-Scope |
| **REQ-UB-02** | `UC-1.1: Перегляд таймлайну`, `UC-1.4: Перегляд досьє` | Покривається валідацією асоціації `Character_Game` | In-Scope |
| **REQ-UB-03** | `UC-1.1: Перегляд таймлайну`, `UC-2: Синхронізація метаданих` | Покривається перевіркою унікальності `chron_order` | In-Scope |
| **REQ-EV-01** | `UC-2: Синхронізація метаданих` | `Scenario 1: Happy path` | In-Scope |
| **REQ-EV-02** | `UC-2: Синхронізація метаданих`, `UC-2.1: Фіксація конфлікту канону` | `Scenario 2: Conflict path` | In-Scope |
| **REQ-EV-03** | `UC-2: Синхронізація метаданих` | `Scenario 4: Error path (404)` | In-Scope |
| **REQ-EV-04** | `UC-1.2: Фільтрація персонажів`, `UC-1.3: Пошук за назвою` | Перевірка запитів із JOIN `Snake.character_id` | In-Scope |
| **REQ-EV-05** | `UC-1.1: Перегляд таймлайну ігор` | Перевірка впорядкування за `Game.chron_order` | In-Scope |
| **REQ-ST-01** | `UC-2: Синхронізація метаданих` | `Scenario 3: Lock / edge case (409)` | In-Scope |
| **REQ-ST-02** | `UC-1.4: Перегляд досьє`, `UC-2.1: Фіксація конфлікту канону` | `Scenario 2: Conflict path` | In-Scope |
| **REQ-OP-01** | `UC-1.4: Перегляд досьє` | Перевірка заголовка `X-Data-Status: STALE` | In-Scope |
| **REQ-OP-02** | `UC-1.3: Пошук за назвою / кодом` | Перевірка операцій підрядкового фільтрування | In-Scope |