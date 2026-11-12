# NoteNest
### Academic Notes & Resource Sharing Platform

NoteNest is a centralized academic resource-sharing platform that helps college students upload, organize, discover, access, and download lecture notes, lab manuals, previous-year question papers, assignments, and reference materials.

> **Project status:** Planning and repository scaffolding. Application implementation has not started.

## Problem
Academic resources are often scattered across messaging groups, personal drives, and informal networks. NoteNest aims to make them easier to organize, discover, and share through a structured platform.

## Objectives
- Centralize academic learning resources.
- Organize material by semester, subject, module, and resource type.
- Allow students to contribute resources.
- Provide admin moderation before resources are published.
- Support search, filtering, secure access, and downloads.
- Practice collaborative development, testing, CI, containerization, and deployment.

## Planned Features
- Firebase-backed student authentication.
- Student and administrator roles with protected operations.
- Semester, subject, and module organization.
- Academic resource upload and metadata management.
- Mandatory admin approval before publication.
- Search, filters, resource downloads, and download tracking.
- Student and admin interfaces.

## Technology Stack

| Area | Technology |
|---|---|
| Frontend | HTML5, CSS3, JavaScript |
| Frontend authentication | Firebase Authentication Web SDK |
| Backend API | Python, Flask |
| Backend Firebase integration | Firebase Admin SDK |
| Database | Cloud Firestore |
| File storage | Firebase Storage |
| Testing | Pytest, Postman |
| DevOps | Git, GitHub, GitHub Actions, Docker |
| Deployment | To be finalized; Flask host and Firebase project configured separately |

## Firebase Service Responsibilities

- **Firebase Authentication:** student sign-up/sign-in, identity, password reset, and issuing Firebase ID tokens.
- **Cloud Firestore:** user profiles and roles, academic catalog, resource metadata and moderation state, and download records.
- **Firebase Storage:** actual uploaded PDF/document files.
- **Firebase Admin SDK (Flask):** verify ID tokens and securely access Firestore and Storage from the backend.

The frontend signs users in with Firebase Authentication and sends the resulting ID token to protected Flask endpoints as a Bearer token. Flask verifies the token and enforces authorization. The backend, not the browser, decides whether a resource is approved.

## Architecture Overview

The JavaScript frontend communicates with a modular Flask REST API. Firebase Authentication handles identity; Flask verifies Firebase ID tokens using the Admin SDK. Firestore stores structured application data, while Firebase Storage stores file contents. The backend enforces role checks, ownership, upload validation, and the resource moderation workflow.

See [System Architecture](docs/system-architecture.md), [Database Design](docs/database-design.md), and [API Documentation](docs/API-documentation.md).

## Main Modules
1. **Authentication & User Management** — Firebase sign-in integration, profile provisioning, role management, and backend token verification.
2. **Academic Catalog Management** — semesters, subjects, and modules.
3. **Resource Upload & Moderation** — file validation, Storage upload, metadata, and mandatory admin review.
4. **Search & Download** — approved-resource discovery, filtering, secure downloads, and download tracking.
5. **Frontend & DevOps** — responsive UI, testing, CI, Docker, and deployment.

## Repository Structure

```text
NoteNest/
├── docs/                 # Product, architecture, data, API, and workflow docs
├── backend/
│   ├── app/              # Flask application (implementation pending)
│   ├── tests/            # Backend tests (implementation pending)
│   ├── requirements.txt
│   └── README.md
├── frontend/             # HTML, CSS, JavaScript and Firebase client setup
├── .github/
│   ├── workflows/        # CI workflows (to be added)
│   └── PULL_REQUEST_TEMPLATE.md
├── CONTRIBUTING.md
├── .gitignore
└── README.md
```

## Getting Started
This repository currently contains planning documentation and scaffolding. Local setup instructions, Firebase project configuration, and environment-variable templates will be added alongside the initial implementation. Never commit Firebase service-account credentials or private keys.

## Contribution
Read [CONTRIBUTING.md](CONTRIBUTING.md). Create feature branches from `staging` and open pull requests into `staging`. Only stable, reviewed work should move to `main`.

## Team
Team credits will be added after the team confirms names and GitHub usernames.

## License
No open-source license has been selected. All rights are reserved by default until the team agrees otherwise.
