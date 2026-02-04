# Entity Relationship Diagram - Women's Soccer Hub

> **Auto-generated** from your idea analysis
> **Entities:** 2

---

## Visual Diagram

```mermaid
erDiagram
    profiles {
        uuid id PK
        text username UK
        text display_name
        text avatar_url
        timestamptz created_at
        timestamptz updated_at
    }

    users {
        uuid id PK
        uuid user_id FK
        uuid id
        timestamptz created_at
        timestamptz updated_at
        uuid user_id FK
        text username UK
        text email UK
        timestamptz created_at
        timestamptz updated_at
    }

    posts {
        uuid id PK
        uuid user_id FK
        uuid id
        timestamptz created_at
        timestamptz updated_at
        uuid user_id FK
        text title
        text content
        timestamptz created_at
        timestamptz updated_at
    }

    %% Relationships
    profiles ||--o{ users : owns
    profiles ||--o{ posts : owns
    users ||--o{ posts : "A user can create many posts"
    posts }o--|| users : "A post is created by one user"
```

---

## Entity Details

### User
> A registered user of the platform

**Fields:**
  - `id`: uuid (required) - Primary key
  - `created_at`: datetime (required) - Creation timestamp
  - `updated_at`: datetime (required) - Last update timestamp
  - `user_id`: uuid (required) - Owner user ID
  - `username`: string (required, unique, indexed) - Username chosen by the user
  - `email`: string (required, unique, indexed) - Email address of the user

**Relationships:**
  - one_to_many → **Post**: A user can create many posts

### Post
> A discussion post created by a user

**Fields:**
  - `id`: uuid (required) - Primary key
  - `created_at`: datetime (required) - Creation timestamp
  - `updated_at`: datetime (required) - Last update timestamp
  - `user_id`: uuid (required) - Owner user ID
  - `title`: string (required, indexed) - Title of the post
  - `content`: text (required) - Content of the post

**Relationships:**
  - many_to_one → **User**: A post is created by one user

---

## Notes

- All entities have standard fields: `id`, `user_id`, `created_at`, `updated_at`
- `PK` = Primary Key, `FK` = Foreign Key, `UK` = Unique Key
- Copy the Mermaid code block to visualize in any Mermaid-compatible tool
- Relationships: `||--o{` = one-to-many, `||--||` = one-to-one, `}o--o{` = many-to-many
