# Application Flow

**Status:** Conceptual flow for team review. Update after UI/UX finalization.

## Student Authentication Flow
1. Student opens NoteNest.
2. Student registers or signs in through Firebase Authentication in the frontend.
3. Frontend obtains a Firebase ID token.
4. Frontend calls the Flask profile/bootstrap endpoint with the ID token.
5. Flask verifies the token and creates or retrieves the student's Firestore profile.
6. Student enters the resource discovery area.

Firebase Authentication handles credentials and password reset; Flask does not implement a separate password/login system.

## Browse and Download Flow
1. Student browses the academic catalog.
2. Student searches or filters resources by semester, subject, module, or resource type.
3. Flask returns approved resource metadata from Firestore.
4. Student opens resource details.
5. Student requests a download.
6. Flask verifies identity, checks that the resource is approved, and authorizes delivery from Firebase Storage.
7. The successful download is recorded in Firestore.

## Resource Submission Flow
1. Authenticated student opens the upload form.
2. Student selects a file and enters resource metadata.
3. Frontend submits the file and metadata to Flask with the Firebase ID token.
4. Flask verifies identity, validates metadata and file content, and checks academic references.
5. Flask stores the file in Firebase Storage.
6. Flask creates a Firestore resource document with `pending` status.
7. Student can view the submission status in their own uploads.

## Admin Moderation Flow
1. Admin signs in through Firebase Authentication using an account provisioned with admin privileges.
2. Frontend obtains an ID token and calls Flask admin endpoints.
3. Flask verifies the token and checks the trusted admin role.
4. Admin views pending submissions and inspects resource details.
5. Admin approves or rejects the submission.
6. Flask records the decision, reviewer UID, timestamp, and rejection reason when applicable.
7. Approved resources become visible/downloadable to students; rejected resources remain unavailable.

## Admin Catalog and User Management
1. Admin accesses protected management screens.
2. Admin creates/updates/deactivates semesters, subjects, and modules.
3. Admin may view or deactivate user access where permitted.
4. Flask enforces all role and ownership rules.

Exact screen navigation, public/private catalog access, edit/delete permissions, and file delivery method remain subject to the approved PRD and API contract.
