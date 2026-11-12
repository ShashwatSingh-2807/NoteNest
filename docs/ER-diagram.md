# Entity Relationship Diagram

**Status:** Proposed v1.0 — matches [database-design.md](database-design.md).  
**Database:** SQLite with Flask-SQLAlchemy.

## ER diagram

```mermaid
erDiagram
    USERS ||--o{ RESOURCES : uploads
    USERS o|--o{ RESOURCES : reviews
    USERS ||--o{ RESOURCE_DOWNLOADS : makes
    SEMESTERS ||--o{ SUBJECTS : contains
    SUBJECTS ||--o{ MODULES : contains
    SUBJECTS ||--o{ RESOURCES : classifies
    MODULES o|--o{ RESOURCES : groups
    RESOURCES ||--o{ RESOURCE_DOWNLOADS : has

    USERS {
        int id PK
        string full_name
        string email UK
        string password_hash
        string role
        boolean is_active
        datetime created_at
        datetime updated_at
    }

    SEMESTERS {
        int id PK
        int semester_number UK
        string name
        datetime created_at
    }

    SUBJECTS {
        int id PK
        int semester_id FK
        string subject_code UK
        string name
        string description
        datetime created_at
    }

    MODULES {
        int id PK
        int subject_id FK
        int module_number
        string name
        datetime created_at
    }

    RESOURCES {
        int id PK
        string title
        string description
        string resource_type
        string original_filename
        string storage_key UK
        string mime_type
        int file_size_bytes
        int subject_id FK
        int module_id FK
        int uploaded_by FK
        string status
        int reviewed_by FK
        datetime reviewed_at
        string rejection_reason
        datetime created_at
        datetime updated_at
    }

    RESOURCE_DOWNLOADS {
        int id PK
        int resource_id FK
        int user_id FK
        datetime downloaded_at
    }
```

## Relationship explanation

| Relationship | Cardinality | Meaning |
|---|---|---|
| Semester → Subject | 1 : many | A semester contains multiple subjects; each subject belongs to one semester. |
| Subject → Module | 1 : many | A subject can contain multiple modules; each module belongs to one subject. |
| Subject → Resource | 1 : many | Each resource is classified under one subject. |
| Module → Resource | 1 : many, optional on resource | A resource may be for a specific module or the whole subject. |
| User → Resource (uploader) | 1 : many | A student can submit multiple resources. |
| User → Resource (reviewer) | 1 : many, optional on resource | An admin can review multiple resources; pending resources have no reviewer yet. |
| User → Resource Download | 1 : many | A user can download many resources. |
| Resource → Resource Download | 1 : many | A resource can be downloaded many times. |

## Moderation lifecycle

```mermaid
flowchart TD
    A[Student submits resource] --> B[Pending]
    B --> C{Admin review}
    C -->|Approve| D[Approved and visible]
    C -->|Reject with reason| E[Rejected and not visible]
```

- New submissions always begin as `pending`.
- Only an administrator can approve or reject a submission.
- Only `approved` resources are searchable and downloadable by students.
- If the team adopts the proposed edit policy, editing an approved resource sends it back to `pending`.

## Important integrity rule

When a resource has both `subject_id` and `module_id`, the backend must ensure that the selected module belongs to that same subject. This cross-table rule should be validated by the application (or enforced with an additional composite foreign-key design if the team later chooses to add it).

The diagram and table definitions are proposed for team review. Do not create migrations until the team approves the schema and open decisions in [database-design.md](database-design.md).
