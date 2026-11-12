# Development Workflow

## Branches
- `main`: stable project state.
- `staging`: integration branch.
- `feature/*`: isolated feature work created from `staging`.

## Workflow
1. Clone the repository.
2. Switch to and update `staging`.
3. Create a focused feature branch.
4. Implement and test the assigned change.
5. Commit with a meaningful message.
6. Push and open a PR into `staging`.
7. Obtain review and pass required checks.
8. Merge after approval.
9. Periodically merge stable, reviewed `staging` into `main`.

Do not commit directly to `main` or `staging`. Coordinate changes to shared files and contracts. See [CONTRIBUTING.md](../CONTRIBUTING.md) for commands and review conventions.
