# CLAUDE.md

Development conventions for this repository.

## Setup

```bash
npm ci  # Install dependencies
```

## Commit Conventions

Use conventional commits via Commitizen:

```bash
git cz  # Interactive conventional commit prompt
```

Commit types determine version bumps:

- `feat:` → Minor version bump (0.1.0 → 0.2.0)
- `fix:` → Patch version bump (0.2.0 → 0.2.1)
- `chore:` → No version bump

## Branch Strategy

- **Main Branch**: Production-ready code
- **Feature Branches**: `feature/name` or `bug/name` for new work

## Node.js Version

- Required: **24.10.0** (pinned in `release.yaml` and `package.json`)

## Bash Style

Workflows use strict mode:

```bash
set -euo pipefail
```

- `-e`: Exit on any error
- `-u`: Error on undefined variables
- `-o pipefail`: Pipeline fails if any command fails
