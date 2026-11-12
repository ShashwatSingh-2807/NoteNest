# Product Requirements Document

**Status:** Initial draft. Confirm with the team before implementation.

## Product Summary
NoteNest is a college-focused platform for sharing and discovering academic resources.

## Goals
- Centralize academic resources.
- Organize resources by semester, subject, module, and type.
- Allow authenticated students to contribute resources.
- Make discovery and downloads straightforward.
- Provide administrative controls for platform quality.

## User Roles
- **Student:** register, sign in, manage profile, browse/search resources, and submit resources.
- **Admin:** perform authorized administrative tasks, including moderation and academic catalog management.

Final permissions must be agreed before authorization is implemented.

## Functional Scope
### 1. Authentication & User Management
Student registration/login, profiles, authentication, role-based authorization, and admin privileges.

### 2. Subject & Semester Management
Manage semesters, subjects, modules, and academic categorization.

### 3. Resource Upload & Management
Upload academic files and metadata; validate, edit, or delete permitted resources; support admin moderation.

### 4. Search & Download
Search resources; filter by semester, subject, module, and resource type; download accessible files; track downloads.

### 5. UI/UX & DevOps
Responsive interface, GitHub Actions checks, Docker workflow, and deployment.

## Non-Functional Requirements
- Clear, accessible interface.
- Secure credential and secret handling.
- Server-side validation and authorization.
- Maintainable separation of frontend and backend.
- Automated tests for important behavior.
- Understandable project documentation.

## Out of Scope for Initial Planning
Payments, public social networking, recommendation engines, and complex analytics.

## Open Decisions
Confirm file limits/types, college email verification, moderation states, exact permissions, storage strategy, and deployment persistence before implementation.
