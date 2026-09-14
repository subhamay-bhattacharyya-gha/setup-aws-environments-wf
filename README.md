# Setup Environments – GitHub Reusable Workflow

![Built with Claude Code](https://img.shields.io/badge/Built_with-Claude_Code-D97757?logo=anthropic&logoColor=white)&nbsp;![Release](https://github.com/subhamay-bhattacharyya-gha/setup-aws-environments-wf/actions/workflows/release.yaml/badge.svg)&nbsp;![Commit Activity](https://img.shields.io/github/commit-activity/t/subhamay-bhattacharyya-gha/setup-aws-environments-wf)&nbsp;![Last Commit](https://img.shields.io/github/last-commit/subhamay-bhattacharyya-gha/setup-aws-environments-wf)&nbsp;![Release Date](https://img.shields.io/github/release-date/subhamay-bhattacharyya-gha/setup-aws-environments-wf)&nbsp;![Repo Size](https://img.shields.io/github/repo-size/subhamay-bhattacharyya-gha/setup-aws-environments-wf)&nbsp;![File Count](https://img.shields.io/github/directory-file-count/subhamay-bhattacharyya-gha/setup-aws-environments-wf)&nbsp;![Open Issues](https://img.shields.io/github/issues/subhamay-bhattacharyya-gha/setup-aws-environments-wf)&nbsp;![Top Language](https://img.shields.io/github/languages/top/subhamay-bhattacharyya-gha/setup-aws-environments-wf)&nbsp;![Monthly Commit Activity](https://img.shields.io/github/commit-activity/m/subhamay-bhattacharyya-gha/setup-aws-environments-wf)&nbsp;![Custom Endpoint](https://img.shields.io/endpoint?url=https://gist.githubusercontent.com/bsubhamay/d581133d043d5125e0b78bf39eb55380/raw/setup-aws-environments-wf.json?)

## 📝 About the Reusable Workflow

This reusable GitHub Action sets up CI, development, test, and production environments by creating GitHub environments with required secrets and variables.

## 🛠️ Usage

Call this workflow from another workflow using `workflow_call`. Example:

---

## 🚀 Example: Using This Reusable Workflow

To invoke this workflow from another workflow in the same or external repo, read configuration from `environments.yaml`:

```yaml
name: Setup Environments
run-name: Setup Environments in ${{ github.ref_name }}

on:
  push:
    paths:
      - 'environments.yaml'
      - '.github/workflows/setup-environments.yaml'
    branches:
      - main
  workflow_dispatch:

permissions:
  contents: read
  id-token: write
  actions: write
  deployments: write

jobs:
  parse-config:
    name: Parse Configuration
    runs-on: ubuntu-latest
    outputs:
      ci-environment: ${{ steps.config.outputs.ci-environment }}
      devl-environment: ${{ steps.config.outputs.devl-environment }}
      test-environment: ${{ steps.config.outputs.test-environment }}
      prod-environment: ${{ steps.config.outputs.prod-environment }}
      aws-region: ${{ steps.config.outputs.aws-region }}
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Read environments.yaml
        id: config
        run: |
          ci_env=$(yq -r '.ci.account_name' environments.yaml)
          devl_env=$(yq -r '.devl.account_name' environments.yaml)
          test_env=$(yq -r '.test.account_name' environments.yaml)
          prod_env=$(yq -r '.prod.account_name' environments.yaml)
          region=$(yq -r '.aws_region' environments.yaml)
          
          echo "ci-environment=$ci_env" >> $GITHUB_OUTPUT
          echo "devl-environment=$devl_env" >> $GITHUB_OUTPUT
          echo "test-environment=$test_env" >> $GITHUB_OUTPUT
          echo "prod-environment=$prod_env" >> $GITHUB_OUTPUT
          echo "aws-region=$region" >> $GITHUB_OUTPUT

  setup:
    name: Setup
    needs: parse-config
    uses: subhamay-bhattacharyya-gha/setup-aws-environments-wf/.github/workflows/setup-environments.yaml@main
    with:
      ci-environment: ${{ needs.parse-config.outputs.ci-environment }}
      devl-environment: ${{ needs.parse-config.outputs.devl-environment }}
      test-environment: ${{ needs.parse-config.outputs.test-environment }}
      prod-environment: ${{ needs.parse-config.outputs.prod-environment }}
      aws-region: ${{ needs.parse-config.outputs.aws-region }}
    secrets:
      GH_PAT: ${{ secrets.GH_PAT }}

```

## 📋 Sample Caller Workflow

Here's a complete example of how to use this reusable workflow in your own project:

**File: `.github/workflows/setup-environments.yaml`**

```yaml
name: Setup AWS Environments
run-name: Setup AWS Environments in ${{ github.ref_name }}

on:
  push:
    paths:
      - 'environments.yaml'
      - '.github/workflows/setup-environments.yaml'
    branches:
      - main
  workflow_dispatch:

permissions:
  contents: read
  id-token: write
  actions: write
  deployments: write

jobs:
  parse-config:
    name: Parse Configuration
    runs-on: ubuntu-latest
    outputs:
      ci-environment: ${{ steps.config.outputs.ci-environment }}
      devl-environment: ${{ steps.config.outputs.devl-environment }}
      test-environment: ${{ steps.config.outputs.test-environment }}
      prod-environment: ${{ steps.config.outputs.prod-environment }}
      aws-region: ${{ steps.config.outputs.aws-region }}
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Read environments.yaml
        id: config
        run: |
          ci_env=$(yq -r '.ci.account_name' environments.yaml)
          devl_env=$(yq -r '.devl.account_name' environments.yaml)
          test_env=$(yq -r '.test.account_name' environments.yaml)
          prod_env=$(yq -r '.prod.account_name' environments.yaml)
          region=$(yq -r '.aws_region' environments.yaml)
          
          echo "ci-environment=$ci_env" >> $GITHUB_OUTPUT
          echo "devl-environment=$devl_env" >> $GITHUB_OUTPUT
          echo "test-environment=$test_env" >> $GITHUB_OUTPUT
          echo "prod-environment=$prod_env" >> $GITHUB_OUTPUT
          echo "aws-region=$region" >> $GITHUB_OUTPUT

  setup-environments:
    name: Setup Environments
    needs: parse-config
    uses: subhamay-bhattacharyya-gha/setup-aws-environments-wf/.github/workflows/setup-environments.yaml@v1
    with:
      ci-environment: ${{ needs.parse-config.outputs.ci-environment }}
      devl-environment: ${{ needs.parse-config.outputs.devl-environment }}
      test-environment: ${{ needs.parse-config.outputs.test-environment }}
      prod-environment: ${{ needs.parse-config.outputs.prod-environment }}
      aws-region: ${{ needs.parse-config.outputs.aws-region }}
    secrets:
      GH_PAT: ${{ secrets.GH_PAT }}

  verify-setup:
    name: Verify Setup
    needs: setup-environments
    runs-on: ubuntu-latest
    strategy:
      matrix:
        environment: [ci, devl, test, prod]
    environment: ${{ matrix.environment }}
    steps:
      - name: Verify Environment Variables
        run: |
          echo "Environment: ${{ matrix.environment }}"
          echo "AWS Region: ${{ vars.AWS_REGION }}"
          echo "✓ Environment setup verified"
```

**File: `environments.yaml`**

```yaml
aws_region: us-east-1

ci:
  account_name: ci-ou-a

devl:
  account_name: devl-ou-a

test:
  account_name: test-ou-a

prod:
  account_name: prod-ou-a
```

### Key Points for Caller Workflow

1. **Configuration as Code**: Store environment configuration in `environments.yaml` instead of manual inputs
2. **Automated Execution**: Workflow runs automatically on:
   - Push to `environments.yaml` (config changes)
   - Manual dispatch via `workflow_dispatch`
3. **Parse Configuration Job**: The `parse-config` job reads the YAML file and outputs values using `yq`
4. **Version Tags**: Replace `@v1` with a specific version tag like `@v1.1.0` for production use
5. **Permissions**: Ensure your caller workflow has the required permissions:
   - `id-token: write` for OIDC
   - `actions: write` for environment management
   - `deployments: write` for deployment environment management
6. **Secrets Management**: The `GH_PAT` token must have:
   - `repo` scope for repository access
   - `workflow` scope for workflow management
   - `admin:repo_hook` scope for webhook management
7. **Repository Configuration**: Your repository must define:
   - `AWS_ACCOUNTS`: JSON map of account names to AWS Account IDs
   - `AWS_OIDC_ROLE`: Name of the IAM role for OIDC authentication

## Inputs

| Name               | Description                                        | Required |
|--------------------|----------------------------------------------------|----------|
| `ci-environment`   | AWS Account Name for CI environment                | ✅       |
| `devl-environment` | AWS Account Name for Development environment       | ✅       |
| `test-environment` | AWS Account Name for Test environment              | ✅       |
| `prod-environment` | AWS Account Name for Production environment        | ✅       |
| `aws-region`       | AWS region where services will be deployed         | ✅       |

## Secrets

| Name     | Description                                      | Required |
|----------|--------------------------------------------------|----------|
| `GH_PAT` | GitHub Personal Access Token with `repo` `workflow` and `admin:repo_hook` scopes   | ✅       |

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

- Checks out the repository
- Installs `gh` CLI and `jq`
- For each environment (`ci`, `devl`, `test`, `prod`):
  - Retrieves AWS account ID from the `AWS_ACCOUNTS` repository variable (must be a JSON object)
  - Creates the environment if it doesn’t exist
  - Sets a secret `AWS_ROLE_ARN` with the OIDC role ARN
  - Sets a variable `AWS_REGION` with the target region

## Requirements

Make sure your repository has the following variables defined:

- `AWS_ACCOUNTS`: JSON map of environment account names to AWS Account IDs. Example:

  ```json
  {
    "ci-account-name": "111122223333",
    "devl-account-name": "444455556666",
    "test-account-name": "777788889999",
    "prod-account-name": "000011112222"
  }
  ```

- `AWS_OIDC_ROLE`: Name of the IAM role to assume via OIDC in each AWS account

---

## License

 MIT
