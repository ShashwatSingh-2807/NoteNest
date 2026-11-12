# System Architecture

**Status:** Proposed direction. Detailed design pending.

## Architecture Style
The initial proposal is a modular web application with a separately developed frontend and Flask backend. Keep module boundaries clear without introducing unnecessary microservices.

## Main Components
- **Frontend:** HTML, CSS, and JavaScript; communicates with the backend through documented HTTP interfaces.
- **Backend:** Python and Flask; organized into focused modules after boundaries are agreed.
- **Database:** SQLite for the initial project version.
- **File storage:** decision pending. Do not assume uploaded files can safely live on an ephemeral deployment filesystem.
- **Delivery:** GitHub, GitHub Actions, Docker, and Render, subject to persistence review.

## Module Boundaries
1. Authentication and user management.
2. Subject and semester management.
3. Resource upload and management.
4. Search and download.
5. UI/UX and DevOps.

## Design Principles
- Agree on API contracts before frontend/backend integration.
- Validate input and enforce authorization on the backend.
- Keep secrets outside source control.
- Keep configuration environment-specific.
- Test modules independently and together.

## Decisions Still Required
Database schema, relationships, authentication details, file storage, API conventions, deployment topology, and production persistence. This is not a finalized architecture.
