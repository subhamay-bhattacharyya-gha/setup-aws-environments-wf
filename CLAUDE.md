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

## Bucket Name Variables

The workflow (`.github/workflows/setup-environments.yaml`) derives S3 bucket names per environment from base-name inputs:

| Input (repository variable)        | Derived environment variable     |
| ---------------------------------- | -------------------------------- |
| `CFN_TEMPLATE_BUCKET_BASE_NAME`    | `CFN_TEMPLATE_S3_BUCKET_NAME`    |
| `LAMBDA_PACKAGES_BUCKET_BASE_NAME` | `LAMBDA_PACKAGES_S3_BUCKET_NAME` |
| `GLUE_SCRIPTS_BUCKET_BASE_NAME`    | `GLUE_SCRIPTS_S3_BUCKET_NAME`    |
| `STATE_MACHINE_BUCKET_BASE_NAME`   | `STATE_MACHINE_S3_BUCKET_NAME`   |

- Derived names follow `<base name>-<account ID>-<region>`.
- When adding a bucket, set both the base-name input and the derived `*_S3_BUCKET_NAME` variable in the `ci` block and the regional loop of the workflow.
- Keep the `*_S3_BUCKET_NAME` suffix so names state that they refer to S3 buckets.

## Bash Style

Workflows use strict mode:

```bash
set -euo pipefail
```

- `-e`: Exit on any error
- `-u`: Error on undefined variables
- `-o pipefail`: Pipeline fails if any command fails
