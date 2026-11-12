# NoteNest Backend

The backend is planned as a Python Flask REST API integrated with Firebase through the Firebase Admin SDK.

## Responsibilities
- Verify Firebase Authentication ID tokens.
- Enforce student/admin roles and ownership rules.
- Read and write application documents in Cloud Firestore.
- Upload and retrieve files in Firebase Storage.
- Validate resource metadata and file content.
- Enforce mandatory admin moderation before publication.
- Provide search, catalog, download, and admin APIs.

## Planned Dependencies
- Flask
- firebase-admin
- Pytest
- Additional validation, CORS, and environment/configuration packages to be selected during implementation.

**Not planned:** Flask-JWT-Extended, Flask-SQLAlchemy, SQLite, or a custom password store. Firebase Authentication owns credentials and identity; Firestore is the application database.

## Request Authentication
The frontend sends a Firebase ID token in `Authorization: Bearer <ID_TOKEN>`. Flask verifies it with Firebase Admin SDK. Use the verified UID as the request identity. Never trust client-supplied UID, role, moderation status, or Storage path.

## Suggested Backend Structure

```text
backend/
├── app/
│   ├── __init__.py
│   ├── config.py
│   ├── firebase/
│   │   └── admin.py
│   ├── middleware/
│   │   └── authentication.py
│   ├── routes/
│   ├── services/
│   ├── repositories/
│   └── utils/
├── tests/
├── requirements.txt
└── README.md
```

This is a suggested structure, not implemented code.

## Firebase Setup
1. Create or select the NoteNest Firebase project.
2. Enable Authentication providers selected by the team.
3. Create a Cloud Firestore database.
4. Enable Firebase Storage.
5. Configure Firebase Admin SDK credentials securely for local development and deployment.
6. Add environment configuration without committing secrets.
7. Configure Firestore and Storage Security Rules and test them.

Never commit a service-account JSON file or expose Admin SDK credentials to the frontend. Prefer a secure local credential mechanism and deployment secrets/workload identity.

## Current Status
Documentation and repository scaffolding only. Flask routes, Firebase initialization, Firestore collections, Storage integration, and tests have not yet been implemented.

See [API Documentation](../docs/API-documentation.md), [Database Design](../docs/database-design.md), and [System Architecture](../docs/system-architecture.md).
