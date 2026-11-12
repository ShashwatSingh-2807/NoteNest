# DevOps Plan

**Status:** Planned. Implementation follows once application structure and Firebase project configuration are finalized.

## 1. Version Control
Use Git and GitHub with `main` and `staging` as long-lived branches and feature branches for isolated work.

## 2. Firebase Environments
- Create/configure separate Firebase projects for development and production where practical.
- Enable Firebase Authentication providers required by the product.
- Create Cloud Firestore in Native mode.
- Enable Firebase Storage and configure bucket access.
- Keep frontend Firebase web configuration separate from privileged server credentials. Firebase web config is not a service-account secret, but access must still be protected with Security Rules and authorized domains.
- Never commit service-account JSON files, private keys, or production secrets.

## 3. Firebase Configuration as Code
- Add reviewed Firestore Security Rules and Storage Security Rules.
- Add `firestore.indexes.json` when query patterns require composite indexes.
- Consider Firebase Local Emulator Suite for Auth, Firestore, and Storage development/testing.
- Test rules with the Firebase Emulator Suite before production deployment.
- Deploy rules and indexes through a reviewed process.

## 4. Continuous Integration
Add GitHub Actions after backend test commands and project scripts are established. Pull-request workflows should:
- Install pinned backend dependencies.
- Run formatting/lint checks if adopted.
- Run Pytest.
- Validate frontend scripts/build if a build tool is selected.
- Avoid requiring production Firebase credentials in ordinary pull-request tests; use mocks/emulators or dedicated test credentials safely.

## 5. Containerization
Add a Dockerfile for the Flask API after local startup and environment requirements are defined. Docker Compose may be used for local orchestration, including emulator services if the team chooses.

## 6. Backend Deployment
Render is a candidate for hosting the Flask API. Configure:
- Firebase Admin credentials using a secure secret mechanism or workload identity where supported.
- Environment-specific Firebase project/bucket identifiers.
- CORS allowed origins.
- Request/file upload size limits.
- Production logging and health checks.

Do not store uploaded files or durable application data on the Flask host's ephemeral filesystem; files belong in Firebase Storage and application data in Firestore.

## 7. Testing
- Pytest for Flask route/service behavior.
- Firebase Emulator Suite for Auth, Firestore, and Storage integration/rules tests where practical.
- Postman for API exploration and manual integration.
- Test token verification, role enforcement, ownership, moderation, upload validation, approved-only downloads, and error cases.

## 8. Secrets and Configuration
- Keep service-account credentials and API secrets out of source control.
- Use deployment secret settings or workload identity.
- Restrict access to production Firebase project and service accounts.
- Rotate credentials if they are ever exposed.
