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

3. **Deletes Existing Environments** - Clears old GitHub environments from the repository

4. **Creates New Environments** - Creates CI environment (single instance) and regional environments (one per region) for devl, test, and prod
   - Single region: `ci`, `devl`, `test`, `prod`
   - Multiple regions: `ci`, `devl-{region}`, `test-{region}`, `prod-{region}`

5. **Configures Environment Variables**:
   - `AWS_ENVIRONMENT`: Environment type (ci, devl, test, prod)
   - `AWS_REGION`: Target AWS region
   - `AWS_ACCOUNT_ID`: AWS account ID for the environment
   - `S3_KMS_KEY_ALIAS`: KMS key alias for S3 bucket encryption
   - `TF_STATE_BUCKET_ARN`: S3 bucket ARN for Terraform state (in arn:aws:s3:::bucket-name format)
   - `CFN_TEMPLATE_S3_BUCKET_NAME`: Complete name for the CloudFormation template S3 bucket, formatted as `${env.CFN_TEMPLATE_BUCKET_BASE_NAME}-${env.AWS_ACCOUNT_ID}-${env.AWS_REGION}`

## Branch Protection

Branch protection rules (for `feature/*`, `bug/*`, and `main` branches) are **no longer managed by this workflow**. Instead, use **Organization repository rules** for consistent enforcement across all repositories in your organization.

## Collaborator Management

CODEOWNERS and team access are **no longer managed by this workflow**. Instead, manage teams and permissions at the **Organization level** for consistent access control across all repositories.

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
  ci: ci-account-alias
  devl: devl-account-alias
  test: test-account-alias
  prod: prod-account-alias
regions:
  - us-east-1
  - us-east-2
```

**Requirements:**

- `ci` - **Mandatory** - Must always be specified
- `devl`, `test`, `prod` - **Optional** - Include only the environments you need
- `regions` - **Mandatory** - List of AWS regions (if omitted, defaults to `us-east-1`)

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
- **Regional Environments** (devl, test, prod):
  - **Single region**: Environments named `{environment}` (e.g., `devl`, `test`, `prod`)
  - **Multiple regions**: One environment per region, named as `{environment}-{region}` (e.g., `devl-us-east-1`, `devl-us-east-2`)

### Configuration Examples

**Single Region (Minimal):**

```yaml
environments:
  ci: my-ci-account
  devl: my-devl-account
regions:
  - us-east-1
```

**Created environments:** `ci`, `devl`

**Single Region (Production):**

```yaml
environments:
  ci: my-ci-account
  devl: my-devl-account
  test: my-test-account
  prod: my-prod-account
regions:
  - us-east-1
```

**Created environments:** `ci`, `devl`, `test`, `prod`

**Multi-Region (Production):**

```yaml
environments:
  ci: my-ci-account
  devl: my-devl-account
  test: my-test-account
  prod: my-prod-account
regions:
  - us-east-1
  - us-east-2
```

**Created environments:** `ci`, `devl-us-east-1`, `devl-us-east-2`, `test-us-east-1`, `test-us-east-2`, `prod-us-east-1`, `prod-us-east-2`

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
| `CFN_TEMPLATE_BUCKET_BASE_NAME` | `subhamay-cfn-templates-bucket` | Base name for CloudFormation template S3 buckets |

## How to Use This Workflow

### Step 1: Create an Environment Configuration File

In your repository, create `.env/environments.yaml` (note: the directory and filename are required):

```yaml
environments:
  ci: my-ci-account
  devl: my-devl-account
  test: my-test-account
  prod: my-prod-account
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
