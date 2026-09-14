---
name: setup-environments-workflow
description: Complete guide for the setup-environments reusable GitHub Actions workflow
disable-model-invocation: false
metadata:
  tools: ["Read", "Bash", "Edit", "Write"]
  model: claude-sonnet-5
---

# Setup Environments Workflow Guide

This is a **reusable GitHub Actions workflow** that automatically sets up CI, development, test, and production environments in GitHub by creating environments with required secrets and variables. It integrates with AWS using OIDC authentication.

## Project Overview

- **Type**: GitHub Actions Reusable Workflow
- **Language**: YAML workflows + JavaScript (semantic-release plugins)
- **Primary Use**: Bootstrap GitHub environments with AWS account integration
- **Release Strategy**: Semantic versioning via `semantic-release` with custom plugins

## Repository Structure

```text
.github/
├── workflows/
│   ├── setup-environments.yaml  # Main reusable workflow (core logic)
│   ├── release.yaml             # Automated semantic release trigger
│   └── create-branch.yaml       # Feature branch creation helper
└── CODEOWNERS                   # Repository ownership

scripts/plugins/                 # Custom semantic-release plugins
├── release.config.js            # Semantic-release configuration
├── verify-conditions.js         # Pre-release validation
├── analyze-commits.js           # Commit analysis logic
├── prepare.js                   # Pre-publish steps
├── publish.js                   # Release publication
└── generate-notes.js            # Release notes generation
```

## Workflow Behavior

The `setup-environments.yaml` workflow:

1. **Validates Environment Configuration File** - Checks if `.env/environments.yaml` exists and is in the correct format
   - If validation fails, the workflow halts with a detailed error message displaying the expected format in the GitHub Step Summary
   - Expected format is documented in the [Configuration Format](#configuration-format) section

2. **Parses Environment File** - Reads environment configuration and regions from the validated YAML file

3. **Parses CODEOWNERS** - Extracts GitHub usernames/teams from the CODEOWNERS file

4. **Adds Collaborators** - Grants push access to all CODEOWNERS entries

5. **Deletes Existing Environments** - Clears old GitHub environments from the repository

6. **Creates New Environments** - Creates CI environment (single instance) and regional environments (one per region) for devl, test, and prod

7. **Configures Environment Secrets & Variables**:
   - Secrets: `AWS_ROLE_ARN`, `TF_STATE_BUCKET_NAME`
   - Variables: `AWS_REGION`, `S3_KMS_KEY_ALIAS`, `TF_STATE_BUCKET_NAME`, `AWS_ROLE_ARN`

## Branch Protection Rules Job

After the `setup-environments` job completes successfully, the `setup-branch-protection` job runs to configure branch protection rules for the repository.

### Job Behavior

The `setup-branch-protection` job:

1. **Applies Branch Protection Rules** - Configures ruleset for feature and bug branches
2. **Enforces Code Review** - Requires 1 approval from code owners before merging
3. **Validates Branch Naming** - Enforces naming pattern: `feature/*` or `bug/*`
4. **Prevents Force Pushes** - Blocks non-fast-forward pushes on protected branches
5. **Dismisses Stale Reviews** - Automatically dismisses outdated reviews when new commits are pushed

### Branch Protection Configuration

The branch protection rules are automatically applied to branches matching the pattern:

- **Include**: `refs/heads/feature/*`, `refs/heads/bug/*`
- **Exclude**: None

### Rules Applied

```json
{
  "name": "Branch Protection - Feature & Bug ",
  "target": "branch",
  "enforcement": "active",
  "conditions": {
    "ref_name": {
      "include": [
        "refs/heads/feature/*",
        "refs/heads/bug/*"
      ],
      "exclude": []
    }
  },
  "rules": [
    {
      "type": "creation",
      "parameters": {}
    },
    {
      "type": "non_fast_forward",
      "parameters": {}
    },
    {
      "type": "pull_request",
      "parameters": {
        "required_approving_review_count": 1,
        "require_code_owner_reviews": true,
        "dismiss_stale_reviews": true,
        "require_last_push_approval": false
      }
    },
    {
      "type": "branch_name_pattern",
      "parameters": {
        "operator": "regex",
        "pattern": "^(feature|bug)\\/[a-zA-Z0-9\\-_.]+)$"
      }
    }
  ]
}
```

**Rule Details:**

| Rule | Purpose |
| --- | --- |
| `creation` | Allow branch creation on protected branches |
| `non_fast_forward` | Prevent force pushes and rewrites on protected branches |
| `pull_request` | Require pull request reviews before merging with code owner approval |
| `branch_name_pattern` | Enforce naming convention: branches must start with `feature/` or `bug/` followed by alphanumeric characters, hyphens, underscores, or dots |

**Pull Request Requirements:**

- Minimum 1 approving review required
- Code owner review required
- Stale reviews dismissed automatically
- Last push approval not required

## Main Branch Protection Job

After the `setup-branch-protection` job completes successfully, the `setup-main-branch-protection` job runs to configure stringent protection rules for the main branch.

### Job Behavior

The `setup-main-branch-protection` job:

1. **Applies Main Branch Protection Rules** - Configures ruleset specifically for the main production branch
2. **Prevents Deletion** - Blocks deletion of the main branch
3. **Prevents Force Pushes** - Blocks non-fast-forward pushes to maintain history integrity
4. **Requires Pull Request Review** - Requires at least 1 approval before merging
5. **Requires Deployment Environments** - Ensures code is deployed to CI and development environments before main branch integration

### Branch Protection Configuration

The branch protection rules are applied to the main branch:

- **Include**: `refs/heads/main`
- **Exclude**: None

### Rules Applied

```json
{
  "name": "Main Branch Protection",
  "target": "branch",
  "enforcement": "active",
  "conditions": {
    "ref_name": {
      "include": [
        "refs/heads/main"
      ],
      "exclude": []
    }
  },
  "rules": [
    {
      "type": "deletion",
      "parameters": {}
    },
    {
      "type": "non_fast_forward",
      "parameters": {}
    },
    {
      "type": "pull_request",
      "parameters": {
        "required_approving_review_count": 1
      }
    },
    {
      "type": "required_deployments",
      "parameters": {
        "required_deployment_environments": [
          "ci","devl"
        ]
      }
    }
  ]
}
```

**Rule Details:**

| Rule | Purpose |
| --- | --- |
| `deletion` | Prevent accidental or intentional deletion of the main branch |
| `non_fast_forward` | Prevent force pushes and rewrites on the main branch to preserve commit history |
| `pull_request` | Require pull request reviews before merging changes to main |
| `required_deployments` | Ensure code is deployed to CI and development environments before main branch integration |

**Pull Request Requirements:**

- Minimum 1 approving review required
- Code must be successfully deployed to: `ci`, `devl` environments

**Key Differences from Feature/Bug Branch Protection:**

- Main branch cannot be deleted
- Requires successful deployment to CI and development before merging
- Only requires 1 approval (no code owner requirement)
- No branch naming pattern enforcement (main is protected by name directly)

## Configuration Format

### Environment File Location and Validation

The workflow expects the environment configuration file at:

```text
.env/environments.yaml
```

**Important**: The file must:

- Be located at exactly `.env/environments.yaml` in the repository root
- Follow the exact YAML format specified below
- Be valid YAML syntax
- Contain the `environments` section with the `ci` environment as mandatory

If the file is missing, invalid, or doesn't follow the specified format, the workflow will:

- Fail immediately
- Display the validation error in the GitHub Step Summary
- Show the correct format for reference

### Environment File Structure

The configuration file at `.env/environments.yaml` must follow this exact format:

```yaml
---
environments:
  - ci: ci-account-alias
  - devl: devl-account-alias
  - test: test-account-alias
  - prod: prod-account-alias
regions:
  - us-east-1
  - us-east-2
```

**Requirements:**

- `ci` - **Mandatory** - Must always be specified
- `devl`, `test`, `prod` - **Optional** - Include only the environments you need
- `regions` - **Optional** - List of AWS regions (if omitted, no region-specific environments are created)

### JSON Schema Definition

For validation purposes, the configuration file must conform to the following schema:

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "GitHub Environments Configuration",
  "type": "object",
  "required": ["environments"],
  "properties": {
    "environments": {
      "type": "array",
      "items": {
        "type": "object",
        "oneOf": [
          { "properties": { "ci": { "type": "string", "minLength": 1 } }, "required": ["ci"], "additionalProperties": false },
          { "properties": { "devl": { "type": "string", "minLength": 1 } }, "required": ["devl"], "additionalProperties": false },
          { "properties": { "test": { "type": "string", "minLength": 1 } }, "required": ["test"], "additionalProperties": false },
          { "properties": { "prod": { "type": "string", "minLength": 1 } }, "required": ["prod"], "additionalProperties": false }
        ]
      }
    },
    "regions": {
      "type": "array",
      "items": {
        "type": "string",
        "minLength": 1,
        "pattern": "^[a-z]{2}-[a-z]+-\\d{1}$"
      },
      "minItems": 1,
      "default": ["us-east-1"]
    }
  },
  "additionalProperties": false,
  "allOf": [
    {
      "properties": {
        "environments": {
          "contains": {
            "required": ["ci"]
          }
        }
      },
      "required": ["environments"]
    }
  ]
}
```

**Validation Rules:**

- The `ci` environment is **mandatory** and must always be present
- `devl`, `test`, `prod` environments are **optional** - include only the ones you need
- Each environment must be a separate object with exactly one key-value pair
- Environment values (account aliases) must be non-empty strings
- `regions` array is **mandatory** - must contain at least one region
- If `regions` is omitted, it defaults to `["us-east-1"]`
- Region format must be valid AWS region identifiers (e.g., `us-east-1`, `eu-west-2`, `us-west-2`)

### How Environments Are Created

Each account alias must correspond to a key in the `AWS_ACCOUNT_ID_MAP` organization variable.

- **CI Environment**: Single environment named `ci` (not replicated across regions)
- **Regional Environments** (devl, test, prod): One environment per region, named as `{environment}-{account-alias}-{region}`
  - Example: `devl-my-devl-account-us-east-1`, `devl-my-devl-account-us-east-2`

### Configuration Examples

**Full configuration (all environments + multiple regions):**

```yaml
---
environments:
  - ci: my-ci-account
  - devl: my-devl-account
  - test: my-test-account
  - prod: my-prod-account
regions:
  - us-east-1
  - us-east-2
```

**Created environments:** `ci`, `devl-my-devl-account-us-east-1`, `devl-my-devl-account-us-east-2`, `test-my-test-account-us-east-1`, `test-my-test-account-us-east-2`, `prod-my-prod-account-us-east-1`, `prod-my-prod-account-us-east-2`

**Minimal configuration (CI only):**

```yaml
---
environments:
  - ci: my-ci-account
```

**Selective configuration (specific environments + regions):**

```yaml
---
environments:
  - ci: my-ci-account
  - prod: my-prod-account
regions:
  - us-east-1
  - us-west-2
```

**Created environments:** `ci`, `prod-my-prod-account-us-east-1`, `prod-my-prod-account-us-west-2`

## Required and Optional Organization Variables

The calling repository can access these organization-level variables:

### AWS_ACCOUNT_ID_MAP (Required)

JSON object mapping account aliases → account IDs. Can contain any number of entries:

```json
{
  "ci-account-name": "111122223333",
  "devl-account-name": "444455556666",
  "test-account-name": "777788889999",
  "prod-account-name": "000011112222"
}
```

The workflow resolves account aliases from the environment configuration file to their corresponding AWS account IDs.

### Optional Organization Variables with Defaults

The following variables are read from organization variables. If not defined, they default to the values shown:

| Variable | Default Value | Description |
| --- | --- | --- |
| `AWS_ACCOUNT_ID_MAP` | ```{"ci-account-name": "111122223333", "devl-account-name": "444455556666", "test-account-name": "777788889999", "prod-account-name": "000011112222"} ``` | JSON object mapping account aliases → account IDs |
| `AWS_OIDC_ROLE` | `GitHubActionsOIDCRole` | Name of IAM role to assume via OIDC |
| `AWS_REGION` | `us-east-1` | Default AWS region |
| `TF_STATE_BUCKET_BASE_NAME` | `terraform-state-bucket` | Base name for Terraform state S3 buckets |
| `S3_KMS_KEY_ALIAS` | `SB-KMS` | KMS key alias for bucket encryption |

## How to Use This Workflow

### Step 1: Create an Environment Configuration File

In your repository, create `.env/environments.yaml` (note: the directory and filename are required):

```yaml
---
environments:
  - ci: my-ci-account
  - devl: my-devl-account
  - test: my-test-account
  - prod: my-prod-account
regions:
  - us-east-1
  - us-east-2
```

### Step 2: Call the Workflow

From another repository, create a workflow that calls this one:

```yaml
jobs:
  setup:
    uses: subhamay-bhattacharyya-gha/setup-aws-environments-wf/.github/workflows/setup-environments.yaml@main
    secrets:
      GH_PAT: ${{ secrets.GH_PAT }}
```

The workflow will automatically:

- Look for `.env/environments.yaml` in your repository
- Validate the file format (fails with detailed error if invalid)
- Parse the environment configuration
- Set up the specified environments with their corresponding AWS account configurations

## Workflow Permissions

The `setup-environments.yaml` workflow requires these permissions:

```yaml
permissions:
  contents: write     # Push tags and changelog
  id-token: write     # OIDC authentication
  actions: write      # Manage GitHub Actions
  deployments: write  # Manage deployments
```

## GitHub Token Requirements

The workflow requires a GitHub Personal Access Token (GH_PAT) with these scopes:

- `repo` - Full access to repositories
- `workflow` - Manage GitHub Actions workflows
- `admin:repo_hook` - Manage repository webhooks

## Semantic Release Configuration

The project uses `semantic-release` to automate versioning and releases.

### Release Plugins

1. `@semantic-release/commit-analyzer` - Analyzes commits to determine version bump
2. `@semantic-release/release-notes-generator` - Generates release notes
3. `@semantic-release/changelog` - Updates CHANGELOG.md
4. `@semantic-release/git` - Commits changelog updates
5. `@semantic-release/github` - Creates GitHub releases

### Custom Plugins

Custom plugins in `scripts/plugins/` extend semantic-release:

- `verify-conditions.js` - Pre-release validation
- `analyze-commits.js` - Custom commit analysis
- `prepare.js` - Pre-publication setup
- `publish.js` - Release publication logic
- `generate-notes.js` - Release notes customization

To extend the release process, modify or add plugins in `scripts/plugins/` and reference them in `release.config.js`.

## Testing Workflow Changes

Since this is a reusable workflow:

1. Push changes to a feature branch
2. Create a test workflow in another repo that calls your branch:

   ```yaml
   uses: your-fork/setup-aws-environments-wf/.github/workflows/setup-environments.yaml@your-branch
   ```

3. Run the workflow_dispatch trigger to test

## Triggering a Release

Releases are **automated via GitHub Actions** on push to `main`:

1. Semantic release analyzes commits since last tag
2. Custom plugins in `scripts/plugins/` extend the release process
3. CHANGELOG.md is updated with release notes
4. GitHub release is created with release notes

To test locally:

```bash
npm run release  # Requires GITHUB_TOKEN in environment
```

## Troubleshooting

### Semantic Release Fails

- Verify dependencies installed: `npm ci`
- Check `GITHUB_TOKEN` environment variable is set for local testing
- Review logs in `release.yaml` workflow run

### Setup Environments Workflow Fails

- Verify all required organization variables are set (AWS_ACCOUNT_ID_MAP, AWS_OIDC_ROLE, etc.)
- Check that the GH_PAT secret has required scopes
- Ensure the workflow has necessary permissions in the calling repository

### YAML Validation

- Use GitHub's workflow syntax validator (available in PR UI)
- Or validate locally: `npx github-workflow-validator .github/workflows/*.yaml`
