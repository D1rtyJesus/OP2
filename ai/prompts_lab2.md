Лаб2

Посилання на Gemini:
Чат 1: https://share.gemini.google/FzERzj1Qn3Zo
Чат 2: https://share.gemini.google/ZTqxmoHH6udG

Промпт 1(початковий):
 Main goal: створити структуровані вимоги, Use cases та матрицю трасування для системи бази даних лору MGS на основі сутностей з першої лаби (Character, Snake, Game).


Актори системи:

    Людина: Archivist (фанат лору / читач) — переглядає досьє персонажів, відстежує зв'язки між клонами та будує таймлайн ігор.
    Зовнішній сервіс: Konami Archive API (зовнішня система) — для синхронізації метаданих релізів та перевірки канонічності статусів. 


Головні цілі:

    Каталогізація та пошук персонажів за роллю, фінальним статусом та належністю до клонів Снейка.
    Побудова хронологічного порядку ігор (chron_order) незалежно від року релізу (release_year).
    Автоматична верифікація та синхронізація канонічних даних через зовнішнє API. 


Вимоги та критерії прийняття:

    Скласти EARS-вимоги з використанням 4 патернів: Ubiquitous (The system SHALL...), Event-driven (WHEN <тригер>, the system SHALL...), State-driven (WHILE <стан>, the system SHALL...), Optional (WHERE <умова>, the system SHALL...). Усі вимоги мають бути однозначними, без розмитих слів.
    Написати User Stories (As a... I want to... So that...) разом із приймальними тестами у форматі Gherkin (Given/When/Then).
    Побудувати Use Case діаграму в синтаксисі Mermaid або PlantUML. Обов'язково показати двох акторів (людина та зовнішнє API), використати зв'язки <<include>>, <<extend>> та узагальнення (generalization).
    Скласти матрицю трасування «вимога (EARS) - сценарій (UC)» у вигляді Markdown-таблиці. Зв'язки строго багато-до-багатьох (не 1:1). Обов'язково виділити мінімум 1 непокриту вимогу (Out-of-Scope) з поясненням. 



-Нижче наведено повну специфікацію вимог, поведінкові сценарії, діаграму використання та матрицю трасування для системи бази даних лору Metal Gear Solid (MGS Lore DB).1. Системні вимоги (EARS Patterns)IDПатерн EARSФормулювання вимогиREQ-01UbiquitousThe system SHALL maintain persistent records for entities Character, Snake, and Game, enforcing unique canonical identifiers for each entity.REQ-02UbiquitousThe system SHALL log every metadata update and synchronization attempt with a timestamp and the response status code.REQ-03Event-drivenWHEN an Archivist requests a timeline view, the system SHALL sort and render all recorded Game entities strictly by their chron_order attribute in ascending sequence.REQ-04Event-drivenWHEN an Archivist queries the clone lineage of Big Boss, the system SHALL return all Snake entities linked via the Les Enfants Terribles project relationship.REQ-05State-drivenWHILE a synchronization session with the Konami Archive API is active, the system SHALL block manual metadata write operations on entities flagged for update.REQ-06State-drivenWHILE the network connection to Konami Archive API is in a disconnected state, the system SHALL serve cached canonical statuses with an explicit stale_data: true flag.REQ-07OptionalWHERE a Character entity has the subtype Snake, the system SHALL expose attributes for genetic lineage, clone type (Solid, Liquid, Solidus), and cellular degradation rate.REQ-08OptionalWHERE a Game record is marked as non-canon (is_canon: false), the system SHALL exclude its linked character events from the primary in-universe timeline.REQ-09Ubiquitous (Out-of-Scope)The system SHALL automatically resolve narrative timeline paradoxes between contradictory game releases using generative AI summarization. (Виділена непокрита вимога).2. User Stories та приймальні тести (Gherkin)US-01: Хронологічний таймлайн операційAs an Archivist,I want to view games ordered by internal narrative chronology (chron_order),So that I can follow the story from Snake Eater (1964) onwards, regardless of original release dates.GherkinFeature: Chronological Timeline Rendering

  Scenario: Render game timeline ordered by in-universe timeline
    Given the database contains the following game records:
      | title                      | release_year | chron_order |
      | Metal Gear Solid           | 1998         | 3           |
      | Metal Gear Solid 3         | 2004         | 1           |
      | Metal Gear Solid: PW       | 2010         | 2           |
    When the Archivist requests the narrative timeline
    Then the system displays the games in the following order:
      | title                      |
      | Metal Gear Solid 3         |
      | Metal Gear Solid: PW       |
      | Metal Gear Solid           |
    And each entry shows both release_year and chron_order
US-02: Каталогізація клонів родини СнейкаAs an Archivist,I want to filter character records specifically for the Snake entity type,So that I can analyze clone attributes, original DNA sources, and final statuses.GherkinFeature: Snake Clone Entity Lookup

  Scenario: Retrieve Les Enfants Terribles clone profiles
    Given an entity "Liquid Snake" exists with subtype "Snake" and progenitor "Big Boss"
    When the Archivist filters records by entity type "Snake" and lineage "Les Enfants Terribles"
    Then the result set contains "Liquid Snake"
    And the record displays clone type "Liquid", progenitor "Big Boss", and status "Deceased"
US-03: Синхронізація канонічності через Konami Archive APIAs the Konami Archive API,I want to deliver upstream canonical status payloads to MGS Lore DB,So that non-canonical entries or retcons are flagged without manual intervention.GherkinFeature: Upstream Canon Synchronization

  Scenario: Automated verification flags non-canon release
    Given a local game entry "Metal Gear Survive" with status "Pending_Verification"
    When the Konami Archive API sends a sync payload with id "MG_SURVIVE" and "is_canon": false
    Then the system updates "Metal Gear Survive" attribute "is_canon" to false
    And the system logs the sync event with status 200 OK
3. Use Case діаграма (PlantUML)Діаграма відображає:Двох акторів: Archivist (людина) та Konami Archive API (зовнішня система).Узагальнення акторів/прецедентів: базовий пошук Search Dossier спеціалізується у Filter by Snake Lineage.Зв'язки <<include>>: перегляд деталей або синхронізація обов'язково включають валідацію ідентифікаторів сутностей.Зв'язки <<extend>>: розширення перегляду хронології за умови виявлення конфлікту канонічності (Flag Canon Conflict).Фрагмент коду@startuml
left to right direction
skinparam packageStyle rectangle

actor "Archivist" as User
actor "Konami Archive API" as API <<External System>>

rectangle "MGS Lore Database System" {
  ' Базові прецеденти
  usecase "Search Character Dossier" as UC_Search
  usecase "Filter by Snake Lineage" as UC_FilterSnake
  usecase "View Narrative Timeline" as UC_Timeline
  usecase "Inspect Character Profile" as UC_Inspect
  
  ' Прецеденти синхронізації
  usecase "Synchronize Release Metadata" as UC_SyncMeta
  usecase "Verify Canon Status" as UC_VerifyCanon
  
  ' Допоміжні / розширені прецеденти
  usecase "Validate Canonical Entity ID" as UC_ValidateID
  usecase "Flag Non-Canon Conflict" as UC_FlagConflict
  usecase "Log Sync Audit Entry" as UC_LogAudit
}

' Generalization (Узагальнення)
UC_FilterSnake -up-|> UC_Search

' Взаємодія користувача
User --> UC_Search
User --> UC_Timeline
User --> UC_Inspect

' Взаємодія зовнішнього API
API --> UC_SyncMeta
API --> UC_VerifyCanon

' Зв'язки <<include>>
UC_Inspect ..> UC_ValidateID : <<include>>
UC_SyncMeta ..> UC_ValidateID : <<include>>
UC_SyncMeta ..> UC_LogAudit : <<include>>
UC_VerifyCanon ..> UC_LogAudit : <<include>>

' Зв'язки <<extend>>
UC_FlagConflict ..> UC_Timeline : <<extend>> \n(condition: where is_canon == false)
UC_FlagConflict ..> UC_VerifyCanon : <<extend>> \n(condition: on upstream mismatch)

@enduml
4. Матриця трасування (Traceability Matrix)Матриця демонструє зв'язки багато-до-багатьох (M:N) між системними EARS-вимогами та Use Case сценаріями.ID вимогиКороткий опис вимогиUC_Search / UC_FilterSnakeUC_TimelineUC_InspectUC_SyncMetaUC_VerifyCanonСтатус покриттяREQ-01Сховище сутностей Character, Snake, GameXXXXПокритаREQ-02Аудит та логування змін APIXXПокритаREQ-03Сортування за chron_orderXXПокритаREQ-04Запит генеалогії клонів СнейкаXXПокритаREQ-05Блокування запису під час активної сесії APIXXПокритаREQ-06Обслуговування кешу зі статусом staleXXXXПокритаREQ-07Атрибути клонів для підтипу SnakeXXПокритаREQ-08Виключення неканонічних подій з таймлайнуXXПокритаREQ-09Автовирішення парадоксів через GenAIOut-of-Scope (Непокрита)Обґрунтування непокритої вимоги (Out-of-Scope Analysis)Вимога REQ-09: "The system SHALL automatically resolve narrative timeline paradoxes between contradictory game releases using generative AI summarization."Причина виключення: Вимога не покрита жодним сценарієм використання (Use Case) поточного релізу навмисно. Задача вирішення суперечностей канону (наприклад, реткони між Metal Gear 2: Solid Snake та Metal Gear Solid V) покладається на авторитетне джерело — Konami Archive API (прапорець is_canon) або експертне судження самого Archivist. Впровадження недетермінованих алгоритмів генеративного штучного інтелекту порушує цілісність реляційних зв'язків бази даних та виходить за межі сутностей першої лабораторної роботи (Character, Snake, Game).



Промпт 2(другий):
- привіт! почни з питаннь . + ось логіка данних в першій базі даних з якої бере основу ця 
+файди spec.md +  скриншот схеми з 1 лаби

-Для формування специфікації вимог (EARS, User Stories, Use Cases та матриці трасування) необхідно уточнити поведінку системи та бізнес-правила:1. Зв'язки між сутностями (Data Relationships)Як сутність Snake співвідноситься з Character? Це відношення типу спадкування/спеціалізації (1:1 за спільним id), чи Snake має зовнішній ключ character_id?Чи потрібен зв'язок між персонажами та іграми (Character / Snake $\leftrightarrow$ Game, поява в іграх як M:N), чи сутності існують як автономні списки?Чи потрібно моделювати дерево клонування (хто чий донор/клон: Big Boss $\rightarrow$ Solid/Liquid/Solidus), чи для поточного скоупу достатньо прапорця is_clone: boolean?   
2. Інтеграція з Konami Archive APIЯкий тригер запускає синхронізацію: ручний запит Архіваріуса, зміна запису, чи фоновий процес за розкладом?   Яка стратегія вирішення конфліктів, якщо локальний final_status персонажа розходиться зі статусом від API (пріоритет API, ручний вибір користувача чи блокування оновлення)?   Як система має поводитися у разі недоступності API (робота з кешованими/локальними даними з попередженням чи блокування відповідних операцій)?
3. Хронологія та пошукЧи може значення chron_order дублюватися для кількох ігор (якщо події відбуваються одночасно), чи це строго унікальний порядковий номер?   Які критерії комбінованого пошуку: архіваріус фільтрує суворо за role_type + final_status + is_clone, чи потрібен також повнотекстовий пошук за name / codename?   
4. Права та межі системи (Scope)Чи має Архіваріус права на пряме додавання/редагування/видалення (CRUD) персонажів та ігор вручну, чи система на цьому етапі працює як каталог лише для читання (Read-only) з оновленням через API?   Чи є якісь специфічні вимоги, які заплановано винести в категорію Out-of-Scope (наприклад, експорт таймлайну в PDF, розрахунок точних внутрішньоігрових дат або графічне дерево родоводу)?

- як M:N (чистий логічний без додавання FK у базові сутності) . так потрібно, прямі зв'язки багатьох - до - багатьох . дерево не треба моделювати, вистачить просто прапорця is_clone: boolean та його codename
запускається зовнішньою системою ( запит від API або оновлення). при конфлікті пріоритет у офіц API (is_canon: false) , але система фіксує конфлікт (щось типу Flag Non-Canon Conflict) для уваги читача. Під час активної синхронки запис блокуєтсья. Якщо нема зв1язку видати закешованні дані (з позначкою застарілих даних)
строго юнік ордер (нема ігор де є одночасні події.пошук буде щось типу : фільтр за role_type , final_status + прапорець is_clone а також базовий перегляд за іменем/кож нейм.
для Архіваріуса система тільки в режимі каталогу(Read only, пошук + фільтрації) , а оновлення метаданих і канонічності приходять через API. непрокритою вимогою буде автоматичне вирішення сюжетних дір і а парадоксі через шішку. цього не робимо все на розсуд читача. 

- робота не пішла, тому будуть оновлення.

Необхідно доопрацювати такі деталі:

    Модель даних і звірка з Лабою 1: 

У Snake немає поля final_status (воно належить виключно Character). Зв'язок між ними - Snake.character_id = Character.id.

Додай до Character атрибут has_canon_conflict: boolean (дефолт false).

2.Один Use Case вглиб:

Замість чотирьох окремих поверхневих історій беремо UC-2: Синхронізація метаданих (із розширенням фіксації конфлікту) і розписуємо 4 сценарії Gherkin:

1. Happy path (статус збігся, успіх 200 OK).

Conflict path (статус розійшовся, оновлення + прапорець has_canon_conflict = true).

Lock / edge case (оновлення під час активного блокування запису, помилка 409).

Error path (неіснуючий character_id, помилка 404).

3.Діаграма Use Case:

Прибери псевдо-прецеденти «Блокування запису» та «Показ банера» (це внутрішня поведінка системи, а не цілі актора).

Додай узагальнення акторів: базовий актор Reader та його спеціалізація Archivist (Archivist -|> Reader).

4.Блок NFR:

Додай окремий розділ нефункціональних вимог із конкретними цифрами: платформи/браузери, мова, час відгуку (p95 ≤ 150 мс), навантаження (50 RPS, понад ліміт - 429), TTL кешу (24 години та заголовок X-Data-Status: STALE).

1.Out-of-Scope:

Прибери REQ-OOS-01 із таблиці зі словом SHALL. Винеси це в окремий розділ (відмова від GenAI для сюжетних дірок, відсікання неканонічних спін-офів типу Survive/Acid).

5.Матриця трасування:

Додай колонку зі сценаріями Gherkin, щоб показати шлях Use Case -> Сценарій -> Вимога.

Згенеруй оновлений повний текст для файлу вимог, оновлений код діаграми та текст для ADR-документа. 