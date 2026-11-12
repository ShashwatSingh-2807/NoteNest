# Firestore Database Design

**Status:** Proposed v1.0 for team review.  
**Database:** Cloud Firestore (Native mode).  
**File storage:** Firebase Storage.  
**Identity:** Firebase Authentication.

This document replaces the earlier SQLite/relational schema. NoteNest uses Firestore collections and documents; it does not require SQL tables, joins, or Flask-SQLAlchemy models.

## 1. Core Design Decisions

1. Firebase Authentication owns user credentials and identity. Do not store passwords or password hashes in Firestore.
2. Use the Firebase Authentication UID as the stable user identifier and as the document ID in `users/{uid}`.
3. Firestore stores application profiles, roles, academic catalog data, resource metadata/moderation state, and download records.
4. Firebase Storage stores actual uploaded files. Firestore stores a Storage object path and file metadata.
5. Every newly submitted resource starts with `pending` status.
6. Only an authorized admin can approve/reject a resource.
7. Student-facing search and download operations expose only `approved` resources.
8. Flask verifies Firebase ID tokens and performs server-side authorization before accessing protected data.
9. Admin privileges are assigned through a trusted server-side process, never through public registration.

## 2. Proposed Collection Structure

```text
users/{uid}
semesters/{semesterId}
subjects/{subjectId}
modules/{moduleId}
resources/{resourceId}
resource_downloads/{downloadId}
```

Use Firestore document IDs as identifiers. Store references as ID strings (for example `semesterId`, `subjectId`, `moduleId`, `uploadedBy`) rather than relying on SQL-style foreign keys.

## 3. Document Shapes

### 3.1 `users/{uid}`

Created or updated by a trusted backend process after Firebase Authentication sign-up.

| Field | Type | Required | Description |
|---|---|---|---|
| `uid` | string | yes | Firebase Authentication UID |
| `fullName` | string | yes | Display name |
| `email` | string | yes | Email from verified Firebase identity |
| `role` | string | yes | `student` or `admin`; server-controlled |
| `isActive` | boolean | yes | Application access status |
| `createdAt` | timestamp | yes | Profile creation time |
| `updatedAt` | timestamp | yes | Last profile update |

**Rules**
- Never store passwords, password hashes, refresh tokens, or service credentials in this document.
- Public registration always creates a `student` profile.
- Users must not be able to update `role` or `isActive` through profile endpoints.
- Use Firebase Authentication account state and the application profile status together when enforcing access.

### 3.2 `semesters/{semesterId}`

| Field | Type | Required | Description |
|---|---|---|---|
| `semesterNumber` | number | yes | Semester number |
| `name` | string | yes | Display name |
| `isActive` | boolean | yes | Whether shown in the catalog |
| `createdAt` | timestamp | yes | Creation time |
| `updatedAt` | timestamp | yes | Last update |

Semester numbers should be unique. Firestore does not provide relational unique constraints; enforce uniqueness in the backend (for example, with a deterministic document ID or a transaction).

### 3.3 `subjects/{subjectId}`

| Field | Type | Required | Description |
|---|---|---|---|
| `semesterId` | string | yes | Parent semester document ID |
| `subjectCode` | string | yes | College subject code |
| `name` | string | yes | Subject name |
| `description` | string/null | no | Optional description |
| `isActive` | boolean | yes | Catalog visibility |
| `createdAt` | timestamp | yes | Creation time |
| `updatedAt` | timestamp | yes | Last update |

A subject belongs to one semester. Enforce subject-code uniqueness in backend logic/transactions or a dedicated lookup document.

### 3.4 `modules/{moduleId}`

| Field | Type | Required | Description |
|---|---|---|---|
| `subjectId` | string | yes | Parent subject document ID |
| `moduleNumber` | number | yes | Position within subject |
| `name` | string | yes | Module title |
| `isActive` | boolean | yes | Catalog visibility |
| `createdAt` | timestamp | yes | Creation time |
| `updatedAt` | timestamp | yes | Last update |

A module belongs to one subject. Enforce uniqueness of `(subjectId, moduleNumber)` in backend logic/transactions or use a deterministic ID.

### 3.5 `resources/{resourceId}`

Stores metadata and moderation state. The file itself is stored in Firebase Storage.

| Field | Type | Required | Description |
|---|---|---|---|
| `title` | string | yes | Resource title |
| `description` | string/null | no | Optional description |
| `resourceType` | string | yes | `lecture_notes`, `lab_manual`, `previous_year_question`, `reference_material`, `assignment`, or `other` |
| `originalFilename` | string | yes | Sanitized display filename |
| `storagePath` | string | yes | Firebase Storage object path; never a user-supplied path |
| `mimeType` | string | yes | Server-validated content type |
| `fileSizeBytes` | number | yes | Validated size in bytes |
| `semesterId` | string | yes | Denormalized semester ID for filtering |
| `subjectId` | string | yes | Parent subject document ID |
| `moduleId` | string/null | no | Optional module document ID |
| `uploadedBy` | string | yes | Firebase Authentication UID |
| `status` | string | yes | `pending`, `approved`, or `rejected` |
| `reviewedBy` | string/null | no | Admin UID for latest decision |
| `reviewedAt` | timestamp/null | no | Latest review time |
| `rejectionReason` | string/null | no | Required when rejected |
| `createdAt` | timestamp | yes | Submission time |
| `updatedAt` | timestamp | yes | Last update time |

**Rules**
- Backend always sets new resources to `pending`; ignore/reject any client-supplied status.
- Verify that the selected subject belongs to the supplied semester and that an optional module belongs to that subject.
- Generate a unique Storage path such as `resources/{resourceId}/{randomFileName}`.
- Validate actual file content, extension, MIME type, and size; never trust browser-provided metadata alone.
- Only admins can change moderation fields.
- Rejection requires a non-empty reason.
- Only approved resources are returned by student browse/search and download APIs.
- Proposed policy: edits to approved metadata or file return the resource to `pending` for re-review. Confirm this before implementing edit functionality.

### 3.6 `resource_downloads/{downloadId}`

One document per successful download.

| Field | Type | Required | Description |
|---|---|---|---|
| `resourceId` | string | yes | Downloaded resource ID |
| `userId` | string | yes | Downloading Firebase UID |
| `downloadedAt` | timestamp | yes | Download timestamp |
| `resourceTitle` | string | optional | Snapshot for reporting |
| `subjectId` | string | optional | Snapshot for reporting |

Use a new document for each successful download; do not restrict a user to one download per resource. Create the record only after the backend confirms that the resource is approved and can be delivered.

## 4. Logical Relationships

- One semester has many subjects; each subject references one semester.
- One subject has many modules; each module references one subject.
- One subject has many resources.
- A resource may optionally reference one module.
- One user may upload many resources.
- One admin may review many resources.
- One user may create many download records.
- One resource may have many download records.

These are logical relationships implemented with document ID fields, not SQL foreign-key constraints. The backend must validate references and prevent orphaned or mismatched references.

## 5. Firestore Query and Index Considerations

Expected queries include:
- Active semesters, subjects by semester, and modules by subject.
- A student's own resources filtered by `uploadedBy`.
- Admin moderation queue filtered by `status == pending`, ordered by `createdAt`.
- Student resource listing filtered by `status == approved`, optionally by `semesterId`, `subjectId`, `moduleId`, and `resourceType`.
- Download history by `userId` or `resourceId`.

Create composite indexes in response to actual Firestore query requirements. Commit the generated `firestore.indexes.json` to the repository once query patterns are finalized. Use cursor-based pagination (document snapshots) for scalable lists; avoid offset pagination for large collections.

Firestore does not support arbitrary full-text search across title and description. For the first version, use structured filters and a clearly scoped search approach (such as prefix search or a dedicated search service if required). Do not promise SQL-like full-text search without choosing an implementation.

## 6. Firebase Storage Layout

Proposed object path:

```text
resources/{resourceId}/{generatedFileName}
```

- Store file bytes in Storage, not Firestore.
- Do not make the bucket or resource objects public by default.
- Restrict client Storage access with Security Rules; if all file operations go through Flask Admin SDK, Flask must enforce authorization because Admin SDK bypasses Security Rules.
- Use generated object names and validate files server-side.
- Delete orphaned objects if metadata creation fails; handle deletion of both metadata and object deliberately.
- Do not store long-lived public download URLs in resource documents. Store the object path and generate an authorized delivery mechanism when needed.

## 7. Security and Roles

- Firebase Authentication verifies identity.
- Flask verifies ID tokens with Firebase Admin SDK.
- A trusted backend-controlled role source determines whether a user is an admin. Decide whether to use Firebase custom claims, the Firestore user profile, or both with a clearly defined source of truth.
- Never accept a role, UID, moderation status, or Storage path as authoritative just because the client sent it.
- Keep service-account credentials out of Git and frontend code.
- Configure Firestore and Storage Security Rules for the chosen access pattern.
- Admin SDK bypasses Firestore/Storage Security Rules; Flask must enforce all required checks.

## 8. Open Decisions Before Implementation

1. Allowed file types and maximum upload size.
2. Whether college email verification is required.
3. Admin role source and provisioning workflow.
4. Whether uploads/downloads go through Flask or use client SDK flows with strict Security Rules.
5. Exact download delivery mechanism.
6. Student edit/delete permissions and whether approved edits require reapproval.
7. Whether the initial academic catalog is seeded or admin-managed.
8. Search behavior beyond structured Firestore filters.
9. Final Firestore composite indexes and pagination requirements.

## 9. Implementation Guidance

Use Firebase Admin SDK from Flask for privileged server operations. Keep Firestore access behind a repository/service layer rather than scattering raw database calls across route handlers. Validate every referenced document and enforce roles/ownership in the backend. Update the ER/logical model and API contract whenever the collection shapes change.

This is a proposed Firestore model, not an implemented database or deployed Firebase project.
