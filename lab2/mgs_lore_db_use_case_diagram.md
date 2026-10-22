flowchart LR
    %% Actors
    Archivist(["fa:fa-user Archivist<br/>(Reader)"])
    KonamiAPI(["fa:fa-server Konami Archive<br/>(External API)"])

    %% System Boundary
    subgraph System ["Система: MGS Lore Database"]
        direction TB

        %% Core & Generalization Use Cases
        UC_Explore(["UC-1: Дослідження каталогу"])
        UC_Timeline(["UC-1.1: Перегляд таймлайну ігор"])
        UC_Filter(["UC-1.2: Фільтрація персонажів"])
        UC_Search(["UC-1.3: Пошук за назвою / кодом"])
        UC_Dossier(["UC-1.4: Перегляд досьє"])

        %% Supporting & Extension Use Cases
        UC_Stale(["UC-5: Показ банера<br/>застарілих даних"])
        UC_Conflict(["UC-3: Фіксація<br/>конфлікту канону"])
        UC_Sync(["UC-2: Синхронізація метаданих"])
        UC_Lock(["UC-4: Блокування запису<br/>при оновленні"])

        %% Generalization hierarchy (UML triangle arrow: --|>)
        UC_Timeline --|> UC_Explore
        UC_Filter --|> UC_Explore
        UC_Search --|> UC_Explore
        UC_Dossier --|> UC_Explore

        %% Include / Extend relationships (UML dotted arrow: -.->)
        UC_Sync -.->|«include»| UC_Lock
        UC_Conflict -.->|«extend»<br/>(при розбіжності)| UC_Sync
        UC_Stale -.->|«extend»<br/>(при збої API)| UC_Dossier
    end

    %% Actor to System connections
    Archivist --> UC_Explore
    KonamiAPI --> UC_Sync