erDiagram
    CHARACTER {
        string id PK
        string name
        string role_type
        string final_status
    }

    SNAKE {
        string id PK
        string codename
        boolean is_clone
    }

    GAME {
        string id PK
        string title
        number release_year
    }

    CHARACTER }o--o{ GAME : "з'являється у"
    SNAKE }o--o{ GAME : "головний протагоніст у"
    CHARACTER }o--o{ SNAKE : "взаємодіє з"