# DevOps Plan

**Status:** Planned. Implementation follows once app structure is agreed.

## Version Control
Use Git and GitHub, with `main` and `staging` as long-lived branches and feature branches for isolated work.

## Continuous Integration
Add GitHub Actions after backend/frontend test commands are established. Pull-request workflows should validate changes without invoking commands that do not yet exist.

## Containerization
Add Docker configuration after local startup steps and required environment variables are defined.

## Deployment
Render is the planned host. Before deployment, verify database and uploaded-file persistence. Ephemeral filesystems are unsuitable for durable SQLite databases or user uploads without a persistent-storage design.

## Testing
- Pytest for backend automated tests.
- Postman for API exploration and manual integration checks.
- Add frontend checks after choosing its tooling.

## Secrets and Configuration
Store deployment secrets in provider environment settings. Never commit credentials, tokens, or production data.
