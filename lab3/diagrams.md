- :warning: == Scenario 1: Happy path (`200 OK` — статус збігається) ==

```mermaid
sequenceDiagram
    actor API as Konami Archive API
    participant Ctrl as SyncController
    participant Svc as SyncService
    participant LCM as LockCacheManager
    participant Char as Character
    participant Snake as Snake
    participant Game as Game

    API->>+Ctrl: PUT /api/v1/characters/char-bb/sync (payload)
    Ctrl->>+Svc: syncCharacter("char-bb", payload)
    Svc->>+LCM: acquireLock("char-bb")
    LCM-->>-Svc: true

    Svc->>+Char: findById("char-bb")
    Char-->>-Svc: Character(id="char-bb", final_status="MIA", has_canon_conflict=false)

    opt status matches
        Svc->>Svc: verifyStatusMatch("MIA", "MIA")
    end

    opt has snakeProfile
        Svc->>+Snake: syncProfile("char-bb", payload.snakeProfile)
        Snake-->>-Svc: Snake(updated)
    end

    loop each game in payload.games
        Svc->>+Game: validateChronOrderUnique(game.id, game.chron_order)
        Game-->>-Svc: true
    end

    Svc->>+LCM: invalidateCache("char-bb")
    LCM-->>-Svc: ok

    Svc->>+LCM: releaseLock("char-bb")
    LCM-->>-Svc: ok

    Svc-->>-Ctrl: SyncResult(200, final_status="MIA", has_canon_conflict=false)
    Ctrl-->>-API: 200 OK {"final_status": "MIA", "has_canon_conflict": false}
```

---

- :warning: == Scenario 2: Conflict path (`200 OK` — розбіжність статусів, `has_canon_conflict = true`) ==

```mermaid
sequenceDiagram
    actor API as Konami Archive API
    participant Ctrl as SyncController
    participant Svc as SyncService
    participant LCM as LockCacheManager
    participant Char as Character
    participant Snake as Snake
    participant Game as Game

    API->>+Ctrl: PUT /api/v1/characters/char-bb/sync (payload)
    Ctrl->>+Svc: syncCharacter("char-bb", payload)
    Svc->>+LCM: acquireLock("char-bb")
    LCM-->>-Svc: true

    Svc->>+Char: findById("char-bb")
    Char-->>-Svc: Character(id="char-bb", final_status="MIA", has_canon_conflict=false)

    opt status diverges
        Svc->>+Char: updateStatus("char-bb", "Deceased", has_canon_conflict=true)
        Char-->>-Svc: Character(updated)
    end

    opt has snakeProfile
        Svc->>+Snake: syncProfile("char-bb", payload.snakeProfile)
        Snake-->>-Svc: Snake(updated)
    end

    loop each game in payload.games
        Svc->>+Game: validateChronOrderUnique(game.id, game.chron_order)
        Game-->>-Svc: true
    end

    Svc->>+LCM: invalidateCache("char-bb")
    LCM-->>-Svc: ok

    Svc->>+LCM: releaseLock("char-bb")
    LCM-->>-Svc: ok

    Svc-->>-Ctrl: SyncResult(200, final_status="Deceased", canon_conflict=true)
    Ctrl-->>-API: 200 OK {"final_status": "Deceased", "canon_conflict": true}
```

---

- :warning: == Scenario 3: Lock / edge case (`409 Conflict` — запис заблоковано) ==

```mermaid
sequenceDiagram
    actor API as Konami Archive API
    participant Ctrl as SyncController
    participant Svc as SyncService
    participant LCM as LockCacheManager

    API->>+Ctrl: PUT /api/v1/characters/char-bb/sync (payload)
    Ctrl->>+Svc: syncCharacter("char-bb", payload)
    Svc->>+LCM: acquireLock("char-bb")
    LCM-->>-Svc: false

    opt lock failed
        Svc-->>Ctrl: ConflictException("Entity char-bb is locked")
    end

    deactivate Svc
    Ctrl-->>-API: 409 Conflict {"error": "Entity synchronization already in progress"}
```

---

- :warning: == Scenario 4: Error path (`404 Not Found` — неіснуючий `character_id`) ==

```mermaid
sequenceDiagram
    actor API as Konami Archive API
    participant Ctrl as SyncController
    participant Svc as SyncService
    participant LCM as LockCacheManager
    participant Char as Character

    API->>+Ctrl: PUT /api/v1/characters/char-unknown/sync (payload)
    Ctrl->>+Svc: syncCharacter("char-unknown", payload)
    Svc->>+LCM: acquireLock("char-unknown")
    LCM-->>-Svc: true

    Svc->>+Char: findById("char-unknown")
    Char-->>-Svc: null

    opt character not found
        Svc->>+LCM: releaseLock("char-unknown")
        LCM-->>-Svc: ok
        Svc-->>Ctrl: NotFoundException("Character char-unknown not found")
    end

    deactivate Svc
    Ctrl-->>-API: 404 Not Found {"error": "Character not found"}
```