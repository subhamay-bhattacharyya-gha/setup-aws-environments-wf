# [1.4.0](https://github.com/subhamay-bhattacharyya-gha/setup-aws-environments-wf/compare/v1.3.0...v1.4.0) (2026-09-25)


### Bug Fixes

* add defaults for organization variables and debug logging for CFN_TEMPLATE_S3_BUCKET_NAME ([651afa2](https://github.com/subhamay-bhattacharyya-gha/setup-aws-environments-wf/commit/651afa20f75bc60852ff540d8b98e21227c863d9))
* handle single region case in environment creation output ([0901d1c](https://github.com/subhamay-bhattacharyya-gha/setup-aws-environments-wf/commit/0901d1cb5eed4c6fc2fc11585781b92890f35a5f))
* update CFN_TEMPLATE_S3_BUCKET_NAME format in workflow documentation ([b2c4249](https://github.com/subhamay-bhattacharyya-gha/setup-aws-environments-wf/commit/b2c424959719f180f3e86fa1aa086bc5a62ec9b7))


### Features

* add CFN_TEMPLATE_S3_BUCKET_NAME environment variable to workflow ([fd869a2](https://github.com/subhamay-bhattacharyya-gha/setup-aws-environments-wf/commit/fd869a20c71ecd818396e2101805a48b65d56c6e))
* add CloudFormation template S3 bucket name variable and update workflow configurations ([af10a24](https://github.com/subhamay-bhattacharyya-gha/setup-aws-environments-wf/commit/af10a24d5bc387a0d8e33fa5e53f237a08c1a099))

# [1.3.0](https://github.com/subhamay-bhattacharyya-gha/setup-aws-environments-wf/compare/v1.2.1...v1.3.0) (2026-09-15)


### Features

* bump version to 1.2.1 and update release configuration to include package.json ([4e537a7](https://github.com/subhamay-bhattacharyya-gha/setup-aws-environments-wf/commit/4e537a701c7b7b02ab75e821a5ab1c5eeba4b932))

## [1.2.1](https://github.com/subhamay-bhattacharyya-gha/setup-aws-environments-wf/compare/v1.2.0...v1.2.1) (2026-09-15)


### Bug Fixes

* update CODEOWNERS to correct team reference ([6754557](https://github.com/subhamay-bhattacharyya-gha/setup-aws-environments-wf/commit/6754557652614bfdfa25af5c978bb224499319c5))
* update CODEOWNERS to correct team reference ([36980b3](https://github.com/subhamay-bhattacharyya-gha/setup-aws-environments-wf/commit/36980b3546a0feae248a4e51d590edaa7c9a10ba))

# [1.2.0](https://github.com/subhamay-bhattacharyya-gha/setup-aws-environments-wf/compare/v1.1.0...v1.2.0) (2026-09-14)


### Bug Fixes

* correct pull_request rule parameter for GitHub rulesets ([cee98e6](https://github.com/subhamay-bhattacharyya-gha/setup-aws-environments-wf/commit/cee98e63cfcf8b13f34d057948d1c5e3e8a8050b))
* correct region extraction in step summary output ([67be70b](https://github.com/subhamay-bhattacharyya-gha/setup-aws-environments-wf/commit/67be70b0c1923e8a5645ccf38573d15564dca0b7))
* delete existing rulesets before creating new branch protection rules ([b9ff528](https://github.com/subhamay-bhattacharyya-gha/setup-aws-environments-wf/commit/b9ff52852f4999285af8cd756b1359ccad89bf51))
* parse regions as space-separated values for array expansion ([e7d04b8](https://github.com/subhamay-bhattacharyya-gha/setup-aws-environments-wf/commit/e7d04b88686085093acd7a894acd76e36b48b5c1))
* remove pull_request rule from feature/bug branch protection ruleset ([17e7c06](https://github.com/subhamay-bhattacharyya-gha/setup-aws-environments-wf/commit/17e7c06194c052fbf114c6aafb35e595c8479fd9))
* remove pull_request rule from main branch protection ruleset ([c96b920](https://github.com/subhamay-bhattacharyya-gha/setup-aws-environments-wf/commit/c96b92036285d2d0eb7fb5cd668729d9fde1b5d9))
* remove push trigger for main branch in workflow ([9abe9b5](https://github.com/subhamay-bhattacharyya-gha/setup-aws-environments-wf/commit/9abe9b5cbaec84c20a6f42bd9dd444293c364553))
* simplify pull_request rule parameters in branch protection rulesets ([44ed7af](https://github.com/subhamay-bhattacharyya-gha/setup-aws-environments-wf/commit/44ed7af0aafca11d622622cb175167a2dc29f037))
* update variable names from AWS_ACCOUNTS to AWS_ACCOUNT_ID_MAP in workflow and documentation ([f441fd6](https://github.com/subhamay-bhattacharyya-gha/setup-aws-environments-wf/commit/f441fd61b057753131c5c6eed92713684e8ded41))


### Features

* add AWS_ENVIRONMENT variable to all environments ([1b8d100](https://github.com/subhamay-bhattacharyya-gha/setup-aws-environments-wf/commit/1b8d1008c0fb2f3c3fd4513ed301109093ae5417))
* add GitHub step summary output for environments and branch protection rules ([02c13e0](https://github.com/subhamay-bhattacharyya-gha/setup-aws-environments-wf/commit/02c13e01279823a1fdfefcddf5683e4387abf9a6))
* add setup guide and conventions for CLAUDE and update environment workflow validation ([54bc898](https://github.com/subhamay-bhattacharyya-gha/setup-aws-environments-wf/commit/54bc898855fff8889a58f07174c3e628ef068ffa))
* enhance workflow configuration by adding environment parsing and automation ([8bbf1b4](https://github.com/subhamay-bhattacharyya-gha/setup-aws-environments-wf/commit/8bbf1b452c477defc3091f5bd598f74214b72409))
* make environment naming conditional based on region count ([e555a74](https://github.com/subhamay-bhattacharyya-gha/setup-aws-environments-wf/commit/e555a74c60069387dbf19de75c0833a005b5555a))
* refactor environment configuration parsing and update README for new structure ([1e94990](https://github.com/subhamay-bhattacharyya-gha/setup-aws-environments-wf/commit/1e949906469d5f9a3010a332d873cb5b1622936a))
* update README to include "Built with Claude Code" badge ([2032772](https://github.com/subhamay-bhattacharyya-gha/setup-aws-environments-wf/commit/2032772622501836248cf46b950b0d74d13b6d78))

# [1.1.0](https://github.com/subhamay-bhattacharyya-gha/setup-aws-environments-wf/compare/v1.0.0...v1.1.0) (2026-09-12)


### Bug Fixes

* Update Node.js version to 24.10.0 for semantic-release compatibility ([ae6ea60](https://github.com/subhamay-bhattacharyya-gha/setup-aws-environments-wf/commit/ae6ea60194856267c5890cc76d43c3a42a35045c))


### Features

* upgrade CI/CD tooling to support Node.js 24 ([5ead2ff](https://github.com/subhamay-bhattacharyya-gha/setup-aws-environments-wf/commit/5ead2ffe71d0f886013a888c20ac565d7567c11c))

# 1.0.0 (2025-05-20)


### Features

* Add reusable workflow for setting up AWS environments and update documentation ([fcedda2](https://github.com/subhamay-bhattacharyya-gha/setup-aws-environments-wf/commit/fcedda2f0e2887a952f5bc87bafbb8381f5c1be2))

# Changelog

All notable changes to this project will be documented in this file.
