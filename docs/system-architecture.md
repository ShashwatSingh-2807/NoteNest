# System Architecture

**Status:** Proposed architecture for team review.  
**Backend:** Python + Flask.  
**Firebase services:** Authentication, Cloud Firestore, and Firebase Storage.

## 1. Architecture Style
A modular web application with a JavaScript frontend, Flask REST API, and Firebase-managed identity, database, and file storage. Keep the backend modular; microservices are not required for the initial version.

## 2. High-Level Components

### Frontend
- HTML, CSS, and JavaScript.
- Firebase Authentication Web SDK for sign-up, sign-in, and password reset.
- Sends Firebase ID tokens to protected Flask endpoints using the `Authorization: Bearer <ID_TOKEN>` header.
- Displays student/admin screens according to the authenticated user's role; UI role checks are for presentation only, not security.

### Flask Backend
- Exposes versioned REST endpoints.
- Uses Firebase Admin SDK.
- Verifies Firebase ID tokens on protected routes.
- Enforces role and ownership checks on every protected operation.
- Validates resource metadata and uploaded file content.
- Reads/writes Firestore documents and stores/retrieves objects in Firebase Storage.
- Controls moderation state transitions and download logging.

### Firebase Authentication
- Owns user identity and credentials.
- Handles sign-up, sign-in, password reset, and ID token issuance.
- The backend must verify ID tokens; it must not trust a UID or role sent in an ordinary request body.

### Cloud Firestore
Stores application data in collections such as:
- `users`
- `semesters`
- `subjects`
- `modules`
- `resources`
- `resource_downloads`

See [database-design.md](database-design.md) for proposed document fields and relationships.

### Firebase Storage
Stores actual uploaded files. Firestore stores only metadata and the Storage object path, not the file bytes.

## 3. Request Flow

1. A student signs in through Firebase Authentication in the frontend.
2. The frontend obtains the Firebase ID token.
3. The frontend calls a Flask endpoint and sends the token in the Authorization header.
4. Flask verifies the token with Firebase Admin SDK and obtains the trusted UID.
5. Flask checks the user's role/permissions using trusted server-side role data (and custom claims if adopted).
6. Flask validates the request and reads/writes Firestore or Firebase Storage.
7. Flask returns a response to the frontend.

## 4. Resource Submission and Moderation Flow

1. Authenticated student submits a file and metadata to Flask.
2. Flask verifies the ID token and confirms the user is allowed to upload.
3. Flask validates the file type, size, and content.
4. Flask stores the file in Firebase Storage using a generated, non-user-controlled object path.
5. Flask creates the Firestore resource document with `status: pending`.
6. Admin reviews the pending document.
7. Admin approval changes the status to `approved`; rejection changes it to `rejected` and stores a reason.
8. Student browse/search/download endpoints return only approved resources.

If a Firestore write fails after a Storage upload, the backend should clean up the orphaned Storage object or record it for retry.

## 5. Authorization Model
- Firebase Authentication proves identity; it does not by itself grant application admin permissions.
- Admin roles must be assigned by a trusted server-side process. The team may use Firebase custom claims, a protected Firestore role field, or both; decide and document one authoritative source before implementation.
- The backend must enforce permissions regardless of frontend visibility.
- Firebase Storage and Firestore Security Rules must not allow users to bypass moderation or access restricted files directly.
- If Flask uses the Admin SDK, remember that Admin SDK access bypasses Firebase Security Rules. Therefore, Flask must perform all required authorization checks itself.

## 6. File Access
Recommended initial approach: route upload and download authorization through Flask. Flask stores/retrieves objects using Firebase Admin SDK and only serves or generates a short-lived access mechanism after verifying identity, resource status, and permissions. Do not make the Storage bucket public to simplify downloads.

## 7. Security and Configuration
- Keep service-account credentials out of source control and frontend bundles.
- Use deployment secret/environment configuration or an approved workload identity mechanism.
- Use separate Firebase projects/configuration for development and production where practical.
- Apply least-privilege Firestore and Storage Security Rules for any direct client access.
- Validate all request bodies, file metadata, file contents, and ownership.
- Restrict CORS to the deployed frontend origins in production.

## 8. Deployment
Deploy the Flask API to a selected hosting provider (Render is a candidate). Firebase services are managed separately in the Firebase project. Configure server credentials securely and verify network access, CORS, upload limits, and file persistence. Do not store user uploads on the Flask host's ephemeral filesystem.

## 9. Decisions to Finalize
- Admin role source: custom claims, Firestore profile, or both with a defined source of truth.
- Direct client Storage upload versus Flask-proxied upload.
- Download delivery strategy.
- Firestore indexes and query pagination approach.
- File limits and allowed content types.
- Hosting provider and production Firebase project configuration.
