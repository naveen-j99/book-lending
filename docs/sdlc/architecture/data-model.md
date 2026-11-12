# Data Model

<!-- sdlc:auto:start entities -->
### users

| Field | Type | PK | FK | Nullable |
|---|---|---|---|---|
| id | uuid | True | — | False |
| email | citext (unique) | False | — | False |
| password_hash | text | False | — | False |
| role | varchar(16) (member\|admin) | False | — | False |
| created_at | timestamptz | False | — | False |

### books

| Field | Type | PK | FK | Nullable |
|---|---|---|---|---|
| id | uuid | True | — | False |
| title | varchar(500) | False | — | False |
| author | varchar(300) | False | — | False |
| total_copies | integer (>=0) | False | — | False |
| available_copies | integer (>=0, <=total_copies) | False | — | False |
| created_at | timestamptz | False | — | False |

### loans

| Field | Type | PK | FK | Nullable |
|---|---|---|---|---|
| id | uuid | True | — | False |
| user_id | uuid | False | users.id | False |
| book_id | uuid | False | books.id | False |
| borrowed_at | timestamptz | False | — | False |
| due_date | timestamptz | False | — | False |
| returned_at | timestamptz | False | — | True |

### sessions

| Field | Type | PK | FK | Nullable |
|---|---|---|---|---|
| token | varchar(64) (opaque) | True | — | False |
| user_id | uuid | False | users.id | False |
| created_at | timestamptz | False | — | False |
| expires_at | timestamptz | False | — | False |
<!-- sdlc:auto:end entities -->

<!-- sdlc:auto:start relationships -->
| Source | Target | Type |
|---|---|---|
| users | loans | one-to-many |
| books | loans | one-to-many |
| users | sessions | one-to-many |
<!-- sdlc:auto:end relationships -->

<!-- sdlc:auto:start er-diagram -->
```mermaid
erDiagram
  users {
    uuid id PK
    citext__unique_ email
    text password_hash
    varchar_16___member_admin_ role
    timestamptz created_at
  }
  books {
    uuid id PK
    varchar_500_ title
    varchar_300_ author
    integer____0_ total_copies
    integer____0____total_copies_ available_copies
    timestamptz created_at
  }
  loans {
    uuid id PK
    uuid user_id FK
    uuid book_id FK
    timestamptz borrowed_at
    timestamptz due_date
    timestamptz returned_at
  }
  sessions {
    varchar_64___opaque_ token PK
    uuid user_id FK
    timestamptz created_at
    timestamptz expires_at
  }
  users ||--o{ loans : "one-to-many"
  books ||--o{ loans : "one-to-many"
  users ||--o{ sessions : "one-to-many"
```
<!-- sdlc:auto:end er-diagram -->
