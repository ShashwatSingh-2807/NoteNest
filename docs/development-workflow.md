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
5. For Firebase changes, update/test Firestore or Storage Security Rules and indexes as applicable.
6. Commit with a meaningful message.
7. Push and open a PR into `staging`.
8. Obtain review and pass required checks.
9. Merge after approval.
10. Periodically merge stable, reviewed `staging` into `main`.

Do not commit directly to `main` or `staging`. Coordinate changes to shared files and contracts.

## Firebase Development
- Use the agreed development Firebase project.
- Keep Admin SDK credentials out of the repository.
- Prefer Firebase Local Emulator Suite for local integration and Security Rules testing where practical.
- Review changes to Firestore collection shapes, indexes, and Firestore/Storage Security Rules.
- Never test destructive operations against production data.

See [CONTRIBUTING.md](../CONTRIBUTING.md) for commands and review conventions.
