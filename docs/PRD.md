# Product Requirements Document

**Status:** Initial draft for team review.  
**Product:** NoteNest — Academic Notes & Resource Sharing Platform.

## 1. Product Summary
NoteNest is a college-focused platform where students can upload, organize, discover, access, and download academic resources such as lecture notes, lab manuals, previous-year question papers, assignments, and reference materials.

## 2. Goals
- Centralize academic resources.
- Organize resources by semester, subject, module, and type.
- Allow authenticated students to contribute resources.
- Make resource discovery and downloads straightforward.
- Require administrator approval before a submitted resource is published.
- Provide a maintainable platform suitable for collaborative development.

## 3. User Roles
- **Student:** create an account/sign in, view and update their profile, browse/search approved resources, download approved resources, and submit resources for review.
- **Admin:** perform authorized administrative tasks, manage academic catalog data, review submissions, and manage user access where permitted.

The public registration flow must never let a user grant themselves the admin role. Admin privileges must be assigned through a trusted server-side process.

## 4. Confirmed Technology Direction
- Frontend: HTML, CSS, JavaScript, Firebase Authentication Web SDK.
- Backend: Python and Flask.
- Server-side Firebase integration: Firebase Admin SDK.
- Database: Cloud Firestore.
- File storage: Firebase Storage.
- Testing: Pytest and Postman.
- DevOps: GitHub Actions and Docker; hosting details to be finalized.

### Service responsibilities
- Firebase Authentication owns account credentials and sign-in identity.
- Firestore stores application profiles/roles, academic catalog documents, resource metadata/moderation state, and download activity.
- Firebase Storage stores uploaded file contents.
- Flask verifies Firebase ID tokens and enforces application permissions before accessing protected data or files.

## 5. Functional Scope

### 5.1 Authentication & User Management
- Student sign-up and sign-in using Firebase Authentication.
- Password reset through Firebase Authentication.
- Send Firebase ID tokens with protected API requests.
- Verify tokens in Flask using Firebase Admin SDK.
- Create/maintain application profile documents in Firestore.
- Enforce student/admin permissions on the backend.
- Admin role assignment through a trusted process only.

### 5.2 Academic Catalog
- Maintain semesters, subjects, and modules.
- A subject belongs to one semester.
- A module belongs to one subject.
- Students can read the academic catalog.
- Only authorized admins can create, update, or remove catalog entries.

### 5.3 Resource Upload & Management
- Authenticated students can submit a file with title, description, resource type, subject, and optional module.
- Validate file type, size, and content on the server.
- Store file contents in Firebase Storage.
- Store resource metadata and a Storage object path in Firestore.
- New submissions always begin with `pending` status.
- Students can view their own submissions.
- Resource editing/deletion permissions require a final team decision.

### 5.4 Mandatory Admin Moderation
- Admins can list and inspect pending submissions.
- Admins can approve or reject submissions.
- Rejection requires a reason.
- Record reviewer UID and review timestamp.
- Only approved resources appear in student search and can be downloaded.
- Only admins may change moderation status.

### 5.5 Search & Download
- Search and filter approved resources by title, semester, subject, module, and resource type.
- Support pagination and sorting.
- Authenticated students can download approved resources.
- Validate resource status and authorization before serving/download URL creation.
- Record successful download activity in Firestore.

### 5.6 Admin & Platform Management
- View moderation queue and basic platform counts.
- Manage semesters, subjects, and modules.
- View/manage user access where authorized.
- Provide basic resource and download statistics.

## 6. Non-Functional Requirements
- Backend authorization must not rely on frontend checks.
- Never expose Firebase service-account credentials to the browser or commit them to Git.
- Validate all user input and uploaded files.
- Use least-privilege Firebase Security Rules and server-side access controls.
- Keep API contracts and Firestore document shapes documented.
- Provide understandable error responses and automated tests.
- Design file access so pending/rejected resources are not publicly accessible.

## 7. Out of Scope for Initial Release
AI recommendations/summarization, comments/discussion, ratings, real-time notifications, advanced analytics, payments, and multi-college support.

## 8. Open Decisions
- Allowed file formats and maximum upload size.
- College email verification requirement.
- Whether students may edit/delete resources, and how approved edits trigger re-review.
- Exact admin provisioning process and whether Firebase custom claims will be used.
- Whether file uploads/downloads are proxied through Flask or use short-lived signed/download URLs generated after backend authorization.
- Firestore indexes and query patterns after search requirements are finalized.
- Hosting provider and production configuration.
