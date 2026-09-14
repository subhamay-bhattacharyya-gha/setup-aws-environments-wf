# Setup Environments – GitHub Reusable Workflow

![Built with Claude Code](https://img.shields.io/badge/Built_with-Claude_Code-D97757?logo=anthropic&logoColor=white)&nbsp;![Release](https://github.com/subhamay-bhattacharyya-gha/setup-aws-environments-wf/actions/workflows/release.yaml/badge.svg)&nbsp;![Commit Activity](https://img.shields.io/github/commit-activity/t/subhamay-bhattacharyya-gha/setup-aws-environments-wf)&nbsp;![Last Commit](https://img.shields.io/github/last-commit/subhamay-bhattacharyya-gha/setup-aws-environments-wf)&nbsp;![Release Date](https://img.shields.io/github/release-date/subhamay-bhattacharyya-gha/setup-aws-environments-wf)&nbsp;![Repo Size](https://img.shields.io/github/repo-size/subhamay-bhattacharyya-gha/setup-aws-environments-wf)&nbsp;![File Count](https://img.shields.io/github/directory-file-count/subhamay-bhattacharyya-gha/setup-aws-environments-wf)&nbsp;![Open Issues](https://img.shields.io/github/issues/subhamay-bhattacharyya-gha/setup-aws-environments-wf)&nbsp;![Top Language](https://img.shields.io/github/languages/top/subhamay-bhattacharyya-gha/setup-aws-environments-wf)&nbsp;![Monthly Commit Activity](https://img.shields.io/github/commit-activity/m/subhamay-bhattacharyya-gha/setup-aws-environments-wf)&nbsp;![Custom Endpoint](https://img.shields.io/endpoint?url=https://gist.githubusercontent.com/bsubhamay/d581133d043d5125e0b78bf39eb55380/raw/setup-aws-environments-wf.json?)

## 📝 About the Reusable Workflow

This reusable GitHub Action sets up CI, development, test, and production environments by creating GitHub environments with required secrets and variables.

## 🛠️ Usage

Call this workflow from another workflow using `workflow_call`. Example:

---

## 🚀 Example: Using This Reusable Workflow

To invoke this workflow from another workflow, call it with the `GH_PAT` secret:

```yaml
name: Setup AWS Environments
run-name: Setup AWS Environments in ${{ github.ref_name }}

on:
  workflow_dispatch:

permissions:
  contents: write
  id-token: write
  actions: write
  deployments: write

jobs:
  setup:
    name: Setup AWS Environments
    uses: subhamay-bhattacharyya-gha/setup-aws-environments-wf/.github/workflows/setup-environments.yaml@v1
    secrets:
      GH_PAT: ${{ secrets.GH_PAT }}
```

## 📋 Configuration

Create a `.env/environments.yaml` file in your repository with the following structure:

```yaml
environments:
  ci: AWS-SCS-C03-DEVL
  devl: AWS-SCS-C03-DEVL
  test: AWS-SCS-C03-TEST
  prod: AWS-SCS-C03-PROD
regions:
  - us-east-1
  - us-east-2
```

**Environment Structure:**

- `ci` environment is required
- `devl`, `test`, `prod` are optional
- Specify one or more AWS regions (defaults to `us-east-1` if omitted)

### Key Points

1. **Configuration as Code**: Store environment configuration in `.env/environments.yaml`
2. **Automated Execution**: Workflow runs automatically on:
   - Push to `.env/environments.yaml` (config changes)
   - Push to `.github/workflows/setup-environments.yaml` (workflow changes)
   - Manual dispatch via `workflow_dispatch`
3. **Version Tags**: Use a specific version tag like `@v1.1.0` for production use
4. **Permissions**: Ensure your workflow has the required permissions:
   - `contents: write` to read/validate config files
   - `id-token: write` for OIDC
   - `actions: write` for environment management
   - `deployments: write` for deployment environment management
5. **Secrets Management**: The `GH_PAT` token must have:
   - `repo` scope for repository access
   - `workflow` scope for workflow management
6. **Repository Configuration**: Your repository must define:
   - `AWS_ACCOUNT_ID_MAP`: JSON map of account names to AWS Account IDs
   - `AWS_OIDC_ROLE`: Name of the IAM role for OIDC authentication
   - `TF_STATE_BUCKET_BASE_NAME`: Base name for Terraform state S3 buckets
   - `S3_KMS_KEY_ALIAS`: KMS key alias for S3 bucket encryption

## Secrets

| Name     | Description                                                    | Required |
|----------|----------------------------------------------------------------|----------|
| `GH_PAT` | GitHub Personal Access Token with `repo` and `workflow` scopes | ✅       |

## Permissions Required

This workflow requires the following permissions:

```yaml
permissions:
  contents: write
  id-token: write
  actions: write
  deployments: write
```

## What It Does

1. **Validates** the `.env/environments.yaml` configuration file
2. **Parses** environment and region configuration from YAML
3. **Manages CODEOWNERS**: Adds CODEOWNERS team members as repository collaborators
4. **Deletes** existing GitHub environments to ensure clean setup
5. **Creates GitHub Environments** with conditional naming:
   - **Single region**: `ci`, `devl`, `test`, `prod`
   - **Multiple regions**: `ci`, `devl-{region}`, `test-{region}`, `prod-{region}`
6. **Sets variables** in each environment:
   - `AWS_ENVIRONMENT`: Environment type (ci, devl, test, prod)
   - `AWS_REGION`: Target AWS region
   - `S3_KMS_KEY_ALIAS`: KMS key for S3 encryption
   - `TF_STATE_BUCKET_ARN`: S3 bucket ARN for Terraform state
   - `AWS_ROLE_ARN`: OIDC role ARN
7. **Sets up branch protection** rules for `feature/*` and `bug/*` branches
8. **Sets up main branch protection** with required deployments to `ci` and `devl`

## Requirements

Your repository must have the following variables defined:

- **`AWS_ACCOUNT_ID_MAP`**: JSON map of account names to AWS Account IDs. Example:

  ```json
  {
    "ci-account-alias": "111122223333",
    "devl-account-alias": "444455556666",
    "test-account-alias": "777788889999",
    "prod-account-alias": "000011112222"
  }
  ```

- **`AWS_OIDC_ROLE`**: Name of the IAM role to assume via OIDC in each AWS account (e.g., `GithubActionsRole`)

- **`TF_STATE_BUCKET_BASE_NAME`**: Base name for Terraform state S3 buckets (e.g., `myorg-tfstate`)

- **`S3_KMS_KEY_ALIAS`**: KMS key alias for S3 bucket encryption (e.g., `tfstate-encryption`)

---

## License

 MIT
