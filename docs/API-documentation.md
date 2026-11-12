# API Documentation

**Status:** Proposed v1 contract for team review.  
**Base path:** `/api/v1`  
**Backend:** Flask REST API.  
**Identity:** Firebase Authentication.  
**Data:** Cloud Firestore.  
**Files:** Firebase Storage.

This is the shared frontend/backend contract proposal. Endpoint details and unresolved permissions must be approved before implementation.

## 1. Authentication Convention

The frontend uses Firebase Authentication Web SDK for sign-up, sign-in, and password reset. After sign-in, it obtains a Firebase ID token and sends it to protected Flask endpoints:

```http
Authorization: Bearer <FIREBASE_ID_TOKEN>
```

Flask verifies the token using Firebase Admin SDK. The verified token UID is the identity for the request. Never trust a UID or role supplied in the request body.

Firebase Authentication handles credential operations; Flask does not implement password storage, password hashing, login, or JWT issuance.

### Common response format

Successful responses:

```json
{
  "success": true,
  "data": {},
  "message": "Request completed"
}
```

Error responses:

```json
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "A required field is missing"
  }
}
```

Common status codes:
- `200 OK`: successful read/update/action.
- `201 Created`: document/resource created.
- `400 Bad Request`: invalid request or validation failure.
- `401 Unauthorized`: missing, invalid, or expired Firebase ID token.
- `403 Forbidden`: authenticated user lacks permission or account is inactive.
- `404 Not Found`: requested resource/document does not exist or is not accessible.
- `409 Conflict`: duplicate or invalid state transition.
- `413 Payload Too Large`: upload exceeds configured limit.
- `415 Unsupported Media Type`: unsupported file type.
- `500 Internal Server Error`: unexpected server error.

## 2. Endpoint Summary

### Profile
| Method | Endpoint | Access | Purpose |
|---|---|---|---|
| POST | `/users/me/bootstrap` | Authenticated | Create profile after Firebase sign-up; role defaults to student |
| GET | `/users/me` | Authenticated | Get current user's application profile |
| PATCH | `/users/me` | Authenticated | Update allowed profile fields only |

The client creates/signs in the Firebase Auth account using the Firebase SDK. The bootstrap endpoint verifies the ID token, creates the matching `users/{uid}` Firestore profile if absent, and never accepts a client-selected admin role.

### Academic catalog
| Method | Endpoint | Access | Purpose |
|---|---|---|---|
| GET | `/semesters` | Public or authenticated (team decision) | List active semesters |
| GET | `/subjects?semesterId={id}` | Public or authenticated (team decision) | List/filter subjects |
| GET | `/subjects/{subjectId}` | Public or authenticated (team decision) | Get subject details |
| GET | `/modules?subjectId={id}` | Public or authenticated (team decision) | List modules |
| POST | `/semesters` | Admin | Create semester |
| PATCH | `/semesters/{semesterId}` | Admin | Update semester |
| DELETE | `/semesters/{semesterId}` | Admin | Deactivate/delete subject to reference checks |
| POST | `/subjects` | Admin | Create subject |
| PATCH | `/subjects/{subjectId}` | Admin | Update subject |
| DELETE | `/subjects/{subjectId}` | Admin | Deactivate/delete subject to reference checks |
| POST | `/modules` | Admin | Create module |
| PATCH | `/modules/{moduleId}` | Admin | Update module |
| DELETE | `/modules/{moduleId}` | Admin | Deactivate/delete subject to reference checks |

### Resources and discovery
| Method | Endpoint | Access | Purpose |
|---|---|---|---|
| POST | `/resources` | Student | Upload file and submit metadata (multipart form) |
| GET | `/resources` | Student | List approved resources with filters |
| GET | `/resources/{resourceId}` | Student | Get approved resource details |
| GET | `/users/me/resources` | Student | List current user's submissions, including moderation state |
| PATCH | `/resources/{resourceId}` | Owner (policy pending) | Edit permitted metadata |
| DELETE | `/resources/{resourceId}` | Owner/admin (policy pending) | Delete/deactivate resource |

Student list/detail endpoints must return only approved resources. A student may see their own pending/rejected submissions through the `/users/me/resources` endpoint.

Suggested query parameters for `GET /resources`:
- `q`: search term (exact/prefix strategy to be finalized; Firestore is not a general full-text search engine).
- `semesterId`, `subjectId`, `moduleId`, `resourceType`.
- `limit`: page size with a server-enforced maximum.
- `cursor`: opaque cursor for next page.
- `sort`: approved sort field/direction from an allowlist.

### Admin moderation
| Method | Endpoint | Access | Purpose |
|---|---|---|---|
| GET | `/admin/resources/pending` | Admin | List pending submissions |
| GET | `/admin/resources/{resourceId}` | Admin | Inspect submission details |
| PATCH | `/admin/resources/{resourceId}/review` | Admin | Approve or reject |

Review request example:

```json
{
  "decision": "rejected",
  "rejectionReason": "The uploaded document is not for the selected subject."
}
```

For approval, `decision` is `approved`; rejection requires a non-empty `rejectionReason`. The backend sets `reviewedBy` from the verified admin UID and `reviewedAt` using server time. The client cannot set these fields directly.

### Downloads and admin overview
| Method | Endpoint | Access | Purpose |
|---|---|---|---|
| GET | `/resources/{resourceId}/download` | Student | Authorize and deliver an approved file |
| GET | `/admin/dashboard/summary` | Admin | Basic user/resource/moderation counts |
| GET | `/admin/users` | Admin | List users (fields restricted) |
| PATCH | `/admin/users/{uid}/status` | Admin | Activate/deactivate application access |

Download endpoint must verify the ID token, check the resource is approved, then deliver the file or generate a short-lived authorized download mechanism. Record a download event in Firestore only after successful delivery initiation/confirmation according to the selected implementation. Never store public permanent download URLs in resource documents.

### Health
| Method | Endpoint | Access | Purpose |
|---|---|---|---|
| GET | `/health` | Public | API health check; do not expose secrets or detailed infrastructure data |

## 3. Resource Upload Contract

Use `multipart/form-data`:
- `file`: required file.
- `title`: required string.
- `description`: optional string.
- `resourceType`: required allowed enum.
- `semesterId`: required document ID.
- `subjectId`: required document ID.
- `moduleId`: optional document ID.

Backend responsibilities:
1. Verify Firebase ID token and active student profile.
2. Validate metadata and confirm academic references are consistent.
3. Validate file size, extension, MIME type, and content.
4. Generate resource ID and Storage object path.
5. Upload file to Firebase Storage.
6. Create Firestore resource metadata with `status: pending`, uploader UID from token, and server timestamps.
7. Clean up uploaded file if Firestore creation fails.

The client must not submit `status`, `uploadedBy`, `reviewedBy`, `reviewedAt`, `storagePath`, or arbitrary download URLs.

## 4. Example Resource Response

```json
{
  "success": true,
  "data": {
    "id": "resourceDocumentId",
    "title": "Data Structures - Unit 1 Notes",
    "resourceType": "lecture_notes",
    "semesterId": "semester3",
    "subjectId": "dataStructures",
    "moduleId": "module1",
    "uploadedBy": "firebaseAuthUid",
    "status": "pending",
    "createdAt": "server timestamp"
  }
}
```

Student-facing resource lists should omit non-approved resources and should not expose internal Storage credentials or unrestricted object URLs.

## 5. Authorization Requirements

- Public Firebase sign-up creates a student account/profile only.
- Every protected endpoint verifies the Firebase ID token.
- Admin endpoints require an admin role from the trusted server-side role source.
- Ownership checks use the verified UID.
- Only admins can moderate resources or change protected user roles/status.
- Only approved resources can be downloaded by students.
- The backend must enforce all rules even if the frontend hides controls.

## 6. Pagination and Search

Use cursor-based pagination with a bounded `limit` (for example, default 20 and maximum 100; finalize with the team). Return a `nextCursor` when more results exist. Firestore composite indexes may be required for combinations of status, category filters, and ordering.

Firestore does not provide arbitrary full-text search. Decide whether v1 uses prefix search, a limited client experience, or a dedicated search service before promising full-text behavior.

## 7. Decisions to Finalize

- Which catalog reads are public versus authenticated.
- Admin role source (custom claims, Firestore profile, or a defined combination).
- File size/type limits.
- Direct Storage SDK upload versus Flask-proxied upload.
- Download delivery and download-event timing.
- Resource edit/delete permissions and reapproval behavior.
- Exact search semantics and Firestore indexes.
- Pagination defaults and sorting allowlist.

This document is a proposed contract, not evidence that these endpoints have been implemented.
