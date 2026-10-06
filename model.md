```mermaid
erDiagram
    Character {
        string id PK
        string name
        string role_type
        string final_status
    }

    Snake {
        string id PK
        string codename
        boolean is_clone
    }

    Game {
        string id PK
        string title
        number release_year
        number chron_order
    }

    Character }o--o{ Game : "zjavlyayetsya_u"
    Snake }o--o{ Game : "protagonist_u"
    Character }o--o{ Snake : "vzaiemodiye_z"
```