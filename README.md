# Setup Environments – GitHub Reusable Workflow

![Release](https://github.com/subhamay-bhattacharyya-gha/setup-aws-environments-wf/actions/workflows/release.yaml/badge.svg)&nbsp;![Commit Activity](https://img.shields.io/github/commit-activity/t/subhamay-bhattacharyya-gha/setup-aws-environments-wf)&nbsp;![Last Commit](https://img.shields.io/github/last-commit/subhamay-bhattacharyya-gha/setup-aws-environments-wf)&nbsp;![Release Date](https://img.shields.io/github/release-date/subhamay-bhattacharyya-gha/setup-aws-environments-wf)&nbsp;![Repo Size](https://img.shields.io/github/repo-size/subhamay-bhattacharyya-gha/setup-aws-environments-wf)&nbsp;![File Count](https://img.shields.io/github/directory-file-count/subhamay-bhattacharyya-gha/setup-aws-environments-wf)&nbsp;![Open Issues](https://img.shields.io/github/issues/subhamay-bhattacharyya-gha/setup-aws-environments-wf)&nbsp;![Top Language](https://img.shields.io/github/languages/top/subhamay-bhattacharyya-gha/setup-aws-environments-wf)&nbsp;![Monthly Commit Activity](https://img.shields.io/github/commit-activity/m/subhamay-bhattacharyya-gha/setup-aws-environments-wf)&nbsp;![Custom Endpoint](https://img.shields.io/endpoint?url=https://gist.githubusercontent.com/bsubhamay/d581133d043d5125e0b78bf39eb55380/raw/setup-aws-environments-wf.json?)

## 📝 About the Reusable Workflow

This reusable GitHub Action sets up CI, development, test, and production environments by creating GitHub environments with required secrets and variables.

## 🛠️ Usage

Call this workflow from another workflow using `workflow_call`. Example:

---

## 🚀 Example: Using This Reusable Workflow

To invoke this workflow from another workflow in the same or external repo:

```yaml
name: Setup Environments
run-name: Setup Environments in ${{ github.ref_name }} by ${{ github.actor }}

on:
  workflow_dispatch:
    inputs:
      ci-environment:
        description: "AWS Account Name for CI Environment."
        required: true
        default: "devl-ou-a"
      devl-environment:
        description: "AWS Account Name for Development Environment."
        required: true
        default: "devl-ou-a"
      test-environment:
        description: "AWS Account Name for Test Environment."
        required: true
        default: "test-ou-a"
      prod-environment:
        description: "AWS Account Name for Production Environment."
        required: true
        default: "prod-ou-a"
      aws-region:
        description: "AWS region where the services will be deployed."
        required: true
        default: "us-east-1"

jobs:
  setup:
    name: Setup
    uses: subhamay-bhattacharyya-gha/setup-aws-environments-wf/.github/workflows/setup-environments.yaml@main
    with:
      ci-environment: ${{ github.event.inputs.ci-environment }}
      devl-environment: ${{ github.event.inputs.devl-environment }}
      test-environment: ${{ github.event.inputs.test-environment }}
      prod-environment: ${{ github.event.inputs.prod-environment }}
      aws-region: ${{ github.event.inputs.aws-region }}
    secrets:
      GH_PAT: ${{ secrets.GH_PAT }}

```

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
