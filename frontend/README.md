# NoteNest Frontend

The frontend is planned with HTML5, CSS3, and JavaScript, using the Firebase Web SDK for authentication.

## Responsibilities
- Provide student and admin user interfaces.
- Use Firebase Authentication for sign-up, sign-in, and password reset.
- Obtain Firebase ID tokens after sign-in.
- Send ID tokens to protected Flask API endpoints in the Authorization Bearer header.
- Display academic catalog, approved resources, upload forms, and moderation screens according to the user's role.
- Never treat frontend role checks as security enforcement.

## Firebase Client Setup
- Register the frontend app in the NoteNest Firebase project.
- Add the Firebase Web SDK and initialize it using the project's web app configuration.
- Enable only the Authentication providers selected by the team.
- Configure authorized domains.
- Keep privileged service-account credentials out of frontend files. Firebase web configuration is intended for client use; secure data with Firestore/Storage Security Rules and backend authorization.

## Backend Integration
Use the approved contract in [../docs/API-documentation.md](../docs/API-documentation.md). For protected calls, send:

```http
Authorization: Bearer <FIREBASE_ID_TOKEN>
```

The Flask backend verifies the token. Do not send a user ID or role and expect the backend to trust it.

## Current Status
Planning only. No pages, styles, scripts, Firebase client initialization, or build tooling have been implemented.

## Setup
Local setup instructions will be added after deciding whether to use plain static files or a frontend build tool.
