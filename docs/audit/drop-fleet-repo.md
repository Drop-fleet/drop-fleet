# `Drop-fleet/drop-fleet`: full review (the only accessible repo)

- **Visibility:** public. Created 2023-08-29 (org created 2023-08-11).
- **Description:** "Repository created by AWS CodeStar."
- **Branches:**
  - `master`: 1 commit, `6e724fa`, author `AWS <noreply-aws@amazon.com>`,
    "Initial commit made by AWS during repository creation."
  - `pipeline-qa-master`: `master` plus 19 commits by `azDevops2023`
    (GitHub user created 2023-10-27) on 2024-08-21, 21:06 to 21:59 +07:00,
    nearly all titled "Update azure-pipeline-pipeline-qa.yaml for Azure
    Pipelines".
- **PRs/issues:** none.

## Contents

| Path | What it is |
|------|------------|
| `webpage/` | Stock CodeStar "Congratulations! You just created an HTML5 web application" page. Links to aws.amazon.com, with no Drop branding |
| `template.yml`, `template-configuration.json` | CloudFormation for one EC2 instance and a security group |
| `buildspec.yml`, `appspec.yml`, `scripts/` | CodeBuild/CodeDeploy hooks: Python 3.7, nginx, CodeStar SSH access |
| `azure/azure-pipeline-pipeline-qa.yaml` (branch only) | Node 20 + `npm install` + `npm run husky` on self-hosted agent pool `agentPoolScaleSet1` |

## Code quality, tests, tech debt

- No application code and no tests. Nothing to measure for coverage.
- Tech debt: deprecated platform (CodeStar), end-of-life OS/runtimes, and a CI
  definition that can't pass (no `package.json`; `npm install ./packages.json`
  is a typo or wrong path either way).
- The timezone (+07:00) of the Azure DevOps commits suggests a contractor in
  Southeast Asia (Vietnam, Thailand, or Indonesia). Worth confirming who
  `azDevops2023` is and whether they still have access.

## Recommendation

Archive the repo or make it private. Before archiving, check the AWS account
for a leftover CodeStar CloudFormation stack (`awscodestar-*`) and its EC2
instance, and delete them if they're unused.
