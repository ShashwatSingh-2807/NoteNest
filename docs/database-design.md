# Database Design

**Status:** Proposed v1.0 — ready for team review.  
**Database:** SQLite with Flask-SQLAlchemy.  
**Scope:** Initial NoteNest release.

This design supports student accounts, academic organization, resource uploads with mandatory administrator approval, resource discovery, and download tracking. It intentionally avoids features that are not in the current PRD.

## 1. Design assumptions

The following assumptions make the first version concrete. The team should review them before implementation:

1. A user has one role: `student` or `admin`.
2. A subject belongs to one semester in the initial release.
3. A module belongs to one subject. A resource must belong to a subject; its module is optional because some notes cover an entire subject.
4. Every new resource submission starts with `pending` status. Only an administrator can approve or reject it.
5. Only approved resources appear in student search results and can be downloaded.
6. Only authenticated students can submit resources and download them.
7. Uploaded file bytes are stored in file/object storage, not inside SQLite. The database stores a storage key and file metadata.
8. College-email verification is not included in this schema yet because the verification policy is still an open product decision.
9. Admin accounts should be provisioned through a controlled process; public registration must not allow a user to choose the admin role.

## 2. Entity overview

| Entity | Purpose |
|---|---|
| `users` | Student and administrator accounts |
| `semesters` | Academic semester catalog |
| `subjects` | Subjects offered in a semester |
| `modules` | Units/modules within a subject |
| `resources` | Uploaded academic resources and their moderation state |
| `resource_downloads` | Download activity records |

## 3. Table definitions

### 3.1 `users`

| Column | SQLite type | Rules | Description |
|---|---|---|---|
| `id` | INTEGER | PK, autoincrement | Unique user ID |
| `full_name` | TEXT | NOT NULL | User's display name |
| `email` | TEXT | NOT NULL, UNIQUE | Login email; compare case-insensitively |
| `password_hash` | TEXT | NOT NULL | Secure password hash; never store plain-text passwords |
| `role` | TEXT | NOT NULL, default `student`; CHECK | `student` or `admin` |
| `is_active` | BOOLEAN/INTEGER | NOT NULL, default true | Whether the account can sign in |
| `created_at` | DATETIME/TEXT | NOT NULL | Account creation timestamp (UTC) |
| `updated_at` | DATETIME/TEXT | NOT NULL | Last account update timestamp (UTC) |

**Constraints and notes**
- Normalize email addresses to lowercase before saving, and enforce uniqueness.
- Public student registration must always assign the `student` role on the server.
- Passwords must be hashed with a suitable password-hashing library such as Werkzeug's password hashing utilities.

### 3.2 `semesters`

| Column | SQLite type | Rules | Description |
|---|---|---|---|
| `id` | INTEGER | PK, autoincrement | Unique semester ID |
| `semester_number` | INTEGER | NOT NULL, UNIQUE | Semester number, e.g. 1–8 |
| `name` | TEXT | NOT NULL | Display name, e.g. Semester 1 |
| `created_at` | DATETIME/TEXT | NOT NULL | Creation timestamp (UTC) |

The initial catalog can contain the semesters required by the college. Adjust the number range if the academic program requires it.

### 3.3 `subjects`

Each subject belongs to one semester.

| Column | SQLite type | Rules | Description |
|---|---|---|---|
| `id` | INTEGER | PK, autoincrement | Unique subject ID |
| `semester_id` | INTEGER | NOT NULL, FK → `semesters.id` | Parent semester |
| `subject_code` | TEXT | NOT NULL, UNIQUE | College subject code |
| `name` | TEXT | NOT NULL | Subject name |
| `description` | TEXT | NULL | Optional subject description |
| `created_at` | DATETIME/TEXT | NOT NULL | Creation timestamp (UTC) |

### 3.4 `modules`

Stores modules/units within a subject.

| Column | SQLite type | Rules | Description |
|---|---|---|---|
| `id` | INTEGER | PK, autoincrement | Unique module ID |
| `subject_id` | INTEGER | NOT NULL, FK → `subjects.id` | Parent subject |
| `module_number` | INTEGER | NOT NULL | Position/number within subject |
| `name` | TEXT | NOT NULL | Module title |
| `created_at` | DATETIME/TEXT | NOT NULL | Creation timestamp (UTC) |

**Constraint:** `(subject_id, module_number)` must be unique so a subject cannot have two modules with the same number.

### 3.5 `resources`

Stores resource metadata, file-storage reference, ownership, and moderation details. The actual PDF/document file is stored outside the relational database.

| Column | SQLite type | Rules | Description |
|---|---|---|---|
| `id` | INTEGER | PK, autoincrement | Unique resource ID |
| `title` | TEXT | NOT NULL | Resource title |
| `description` | TEXT | NULL | Optional description |
| `resource_type` | TEXT | NOT NULL, CHECK | `lecture_notes`, `lab_manual`, `previous_year_question`, `reference_material`, `assignment`, or `other` |
| `original_filename` | TEXT | NOT NULL | Filename supplied by uploader, for display only |
| `storage_key` | TEXT | NOT NULL, UNIQUE | Opaque key/path used by configured storage |
| `mime_type` | TEXT | NOT NULL | Validated media type |
| `file_size_bytes` | INTEGER | NOT NULL | File size in bytes; positive and within configured limit |
| `subject_id` | INTEGER | NOT NULL, FK → `subjects.id` | Subject this resource belongs to |
| `module_id` | INTEGER | NULL, FK → `modules.id` | Optional module; must belong to the selected subject |
| `uploaded_by` | INTEGER | NOT NULL, FK → `users.id` | Student who submitted the resource |
| `status` | TEXT | NOT NULL, default `pending`, CHECK | `pending`, `approved`, or `rejected` |
| `reviewed_by` | INTEGER | NULL, FK → `users.id` | Administrator who made the latest moderation decision |
| `reviewed_at` | DATETIME/TEXT | NULL | Time of latest moderation decision (UTC) |
| `rejection_reason` | TEXT | NULL | Explanation provided when rejecting |
| `created_at` | DATETIME/TEXT | NOT NULL | Submission timestamp (UTC) |
| `updated_at` | DATETIME/TEXT | NOT NULL | Last metadata/status update (UTC) |

**Important rules**
- A resource is created with `pending` status. The server must not accept `approved` from a student upload request.
- Only an admin may change a pending resource to `approved` or `rejected`.
- A rejected resource should include a rejection reason.
- The application must verify that `module_id`, when supplied, belongs to the same `subject_id`.
- Only approved resources are returned by student-facing browse/search and download operations.
- Never trust the original filename or MIME type supplied by the browser. Validate file extension, actual content/signature where practical, MIME type, and size on the server. Generate safe storage keys rather than using user filenames as storage paths.
- If an approved resource's file or academic metadata is edited, the proposed v1 policy is to send it back to `pending` for another admin review. Confirm this policy with the team before implementing edits.

### 3.6 `resource_downloads`

Stores one record for each successful authenticated download.

| Column | SQLite type | Rules | Description |
|---|---|---|---|
| `id` | INTEGER | PK, autoincrement | Unique download record |
| `resource_id` | INTEGER | NOT NULL, FK → `resources.id` | Downloaded resource |
| `user_id` | INTEGER | NOT NULL, FK → `users.id` | Student who downloaded it |
| `downloaded_at` | DATETIME/TEXT | NOT NULL | Download timestamp (UTC) |

Each successful download creates a new row. Do not add a unique constraint to `(resource_id, user_id)`, because the same student may download a resource more than once. Record a download only after the server confirms that the resource is approved and the file can be served.

## 4. Relationships and cardinality

- One semester has many subjects; each subject belongs to one semester.
- One subject has many modules; each module belongs to one subject.
- One subject has many resources; each resource belongs to one subject.
- A module may have many resources; a resource may optionally refer to one module.
- One student can upload many resources; each resource has one uploader.
- One administrator can review many resources; a resource may have no reviewer while pending, or one latest reviewer after review.
- One user can have many download records; each download record belongs to one user.
- One resource can have many download records; each download record refers to one resource.

## 5. Keys, constraints, and deletion behavior

- Every table uses an integer primary key.
- Foreign keys must be enabled for every SQLite connection.
- Use foreign-key constraints to prevent orphan records.
- Prefer `RESTRICT` for deleting semesters, subjects, modules, or users that are still referenced by resources or download history. Decide on a deliberate admin archival/deactivation workflow rather than accidentally breaking references.
- Prefer deactivating users over deleting accounts that own resources or download history.
- Use `CASCADE` only for dependent records where deleting the parent should genuinely delete the child; do not cascade-delete a user's uploaded resources or download history by default.
- Use application-level validation in addition to database constraints.

## 6. Recommended indexes

Create indexes for common joins and filters:

- `subjects.semester_id`
- `modules.subject_id`
- `resources.subject_id`
- `resources.module_id`
- `resources.status`
- `resources.resource_type`
- `resources.uploaded_by`
- `resources.created_at`
- `resource_downloads.resource_id`
- `resource_downloads.user_id`
- `resource_downloads.downloaded_at`

A composite index on `resources(status, subject_id, resource_type)` can help approved-resource filtering. Add full-text search only if ordinary title/description search becomes insufficient.

## 7. File storage and deployment note

Do not store uploaded PDFs as large BLOB values in SQLite. Store the file in configured file/object storage and save only its `storage_key` and metadata in `resources`.

A local folder can be used during development. Before deployment, choose persistent object/file storage. Render service filesystems may be ephemeral unless a persistent disk or external storage service is configured. The database and file storage must both survive application restarts and redeployments.

## 8. Open decisions before implementation

The team must confirm:

1. Allowed file extensions, MIME types, and maximum upload size.
2. Whether users must verify a college email before using the platform.
3. Whether a student can edit/delete a resource after submission, approval, or rejection.
4. Whether the initial semester/subject/module catalog is seeded by developers or managed by admins.
5. Which persistent file-storage provider will be used in deployment.
6. Whether download access is restricted to signed-in students (proposed: yes).
7. Whether the project needs multiple departments/programs in a later version (not included in this v1 model).

## 9. Implementation guidance

Use Flask-SQLAlchemy models to represent these tables. Keep validation and permission checks in the backend service/routes layer; frontend checks are for user experience and are not a security boundary.

This is the proposed v1 database model, not a migration or implementation. Update the ER diagram and API contract alongside any approved schema change.
