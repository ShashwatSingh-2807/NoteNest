# NoteNest
### Academic Notes & Resource Sharing Platform

NoteNest is a centralized platform for college students to upload, organize, discover, access, and download academic resources such as lecture notes, lab manuals, previous-year question papers, and reference documents.

> **Status:** Planning and repository setup. Application implementation has not started.

## Problem
Academic resources are scattered across messaging groups, personal drives, and informal networks. NoteNest aims to make these resources easier to organize, discover, and share.

## Objectives
- Centralize academic learning resources.
- Organize material by semester, subject, module, and resource type.
- Support student accounts and role-based administration.
- Enable search, filtering, and downloads.
- Practice collaborative Git, testing, CI, Docker, and deployment workflows.

## Planned Features
- Student registration, login, profiles, and role-based access.
- Semester, subject, and module organization.
- Resource upload, metadata, validation, and moderation.
- Search, filters, downloads, and download tracking.
- Responsive student/admin interface.

## Technology Stack
| Area | Planned technologies |
|---|---|
| Frontend | HTML5, CSS3, JavaScript |
| Backend | Python, Flask, Flask-SQLAlchemy, Flask-JWT-Extended |
| Database | SQLite |
| Testing | Pytest, Postman |
| DevOps | Git, GitHub, GitHub Actions, Docker, Render |

## Architecture Overview
The proposed application has a separately developed frontend and Flask backend, with SQLite for the initial version. Module boundaries and API contracts will be finalized before implementation. Deployment design must account for persistent database and uploaded-file storage.

## Module Division
1. **Authentication & User Management** — registration, login, profiles, roles, and admin privileges.
2. **Subject & Semester Management** — semesters, subjects, modules, and academic categorization.
3. **Resource Upload & Management** — uploads, metadata, validation, editing, deletion, and moderation.
4. **Search & Download** — search, filters, downloads, and download tracking.
5. **UI/UX & DevOps** — responsive frontend, CI, Docker, and deployment.

## Repository Structure
```text
NoteNest/
├── docs/                 # Product, architecture, API, and workflow documentation
├── backend/
│   ├── app/              # Backend application (implementation pending)
│   ├── tests/            # Backend tests (implementation pending)
│   ├── requirements.txt
│   └── README.md
├── frontend/             # Frontend workspace and guidance
├── .github/
│   ├── workflows/        # CI workflows (to be added after test setup)
│   └── PULL_REQUEST_TEMPLATE.md
├── CONTRIBUTING.md
├── .gitignore
└── README.md
```

## Getting Started
This repository currently contains planning documentation and scaffolding only. Setup instructions will be added when the application structure and dependencies are finalized.

## Contribution
Read [CONTRIBUTING.md](CONTRIBUTING.md). Feature branches should start from `staging` and PRs should target `staging`. Only stable, reviewed work should move to `main`.

## Team
Team member credits will be added after the team confirms names and GitHub usernames.

## License
No open-source license has been selected. All rights are reserved by default until the team agrees otherwise.
