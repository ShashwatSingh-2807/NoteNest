# Contributing to NoteNest

## Before contributing
Read the root README and relevant documents in `docs/`. Coordinate changes to shared API contracts, Firestore document shapes, Firebase Security Rules, and module boundaries before implementation.

## Branching
- `main`: stable, reviewed project state.
- `staging`: integration branch.
- Create feature branches from the latest `staging`.
- Planned names: `feature/firebase-auth`, `feature/academic-management`, `feature/resource-management`, `feature/search-download`, `feature/frontend`, and `feature/ci-cd`.
- For small fixes, use a descriptive name such as `fix/upload-validation`.

Do not commit directly to `main` or `staging`.

## Typical workflow
```bash
git clone https://github.com/ShashwatSingh-2807/NoteNest.git
cd NoteNest
git switch staging
git pull origin staging
git switch -c feature/your-feature
# Make and test your changes
git add .
git commit -m "feat: describe the change"
git push -u origin feature/your-feature
```

Open a pull request from your feature branch into `staging`.

## Commit messages
Use a simple Conventional Commits-inspired format:
- `feat:` new functionality
- `fix:` bug fix
- `docs:` documentation
- `test:` tests
- `refactor:` restructuring without behavior change
- `chore:` maintenance/tooling

Keep commits focused and messages meaningful.

## Pull requests and reviews
Describe the problem, changes, and tests. Include screenshots for UI changes. Keep PRs focused, request teammate review, address feedback, resolve conversations, and ensure required checks pass before merging. Do not target `main` for ordinary feature work.

## Shared Firebase and API contracts
Discuss and document changes to Firestore collection shapes, Firebase Authentication behavior, role assignment, Firebase Security Rules, Storage paths, API request/response formats, and file delivery before parallel implementation. Keep frontend and backend aligned through the approved API documentation.

## Security
Never commit service-account JSON files, private keys, passwords, tokens, local environment files, personal data, production data, or uploaded user files. Do not expose Firebase Admin SDK credentials in frontend code. Use environment variables, secure deployment secrets, and approved secure channels for credentials.
