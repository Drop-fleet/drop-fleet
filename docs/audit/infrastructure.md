# 3. Infrastructure and deployment

## Verified (from `drop-fleet`)

### AWS CodeStar → CodeBuild → CodeDeploy → single EC2 (master branch)

- `template.yml`: CloudFormation with the `AWS::CodeStar` transform. It creates
  one `AWS::EC2::Instance` (`WebApp01`) with a public IP and security group
  `WebAppSG`, which opens **TCP 80 and TCP 22 to `0.0.0.0/0`**. IMDSv2 is
  enforced (`HttpTokens: required`). On burstable (t3) instances it sets
  `CPUCredits: unlimited`, which can quietly add cost.
- `template.yml` UserData installs Ruby 2.6 (`amazon-linux-extras`), CodeDeploy
  agent **1.4.0-2218** (pinned and old), and `awslogs-1.1.4`, and ships CodeDeploy
  logs to CloudWatch Logs.
- `buildspec.yml`: CodeBuild with the `java: corretto11` runtime (not needed for
  a static site). It only rewrites placeholders in `template-configuration.json`.
- `appspec.yml` + `scripts/`: CodeDeploy copies `webpage/` to `/var/www`,
  installs Python 3.7.9, pip 9.0.3 → 20.2.1, `awscli==1.27.84`,
  `boto3==1.26.84`, `pycryptodome==3.9.7`, and nginx, then
  `service nginx start`.
- `scripts/codestar_remote_access`: downloads `get_authorized_keys` from a
  public AWS S3 bucket and sets it as sshd `AuthorizedKeysCommand`, run as
  root. SSH keys are managed through CodeStar team membership.
- **AWS CodeStar was discontinued by AWS on 2024-07-31**, and Amazon Linux 2
  `amazon-linux-extras` reached end of support on 2026-06-30. If this stack is
  still deployed it can't be managed through CodeStar any more, and the OS is
  past end of support.

### Azure DevOps Pipelines (`pipeline-qa-master` branch)

- `azure/azure-pipeline-pipeline-qa.yaml`: trigger `pipeline-qa-master`,
  self-hosted pool `agentPoolScaleSet1` (an Azure VM Scale Set agent pool), and
  variable group `PIPELINE_QA_DEV`.
- Steps: `curl … nvm/v0.40.0/install.sh | bash`, `nvm install 20`,
  `npm install`, `npm install ./packages.json`, then `npm run husky`.
- The branch contains **no `package.json`**, so this pipeline cannot succeed
  on this repo. It looks like a pipeline template being tested here before use
  elsewhere (20 commits in about 45 minutes on 2024-08-21 by `azDevops2023`).
  It confirms an **Azure DevOps org with self-hosted VMSS agents** exists and
  may be building the real repos.

## Unverified (needs `docker`, `argocd-*`, `alert`, server repos)

- Kubernetes clusters (provider: EKS/AKS/GKE/self-managed?) and how many
  (`argocd-df` and `argocd-dd` suggest per-product apps, maybe per-product
  clusters).
- Environments (dev/staging/prod), image registry, how images are built
  (Azure Pipelines? GitHub Actions? none?) and promoted into the ArgoCD repos.
- Monitoring/alerting stack (`argocd-monitoring`, `alert`): Prometheus,
  Grafana, Alertmanager, and where alerts are routed (Slack, SMS through Telnyx?).
- Secrets management in Kubernetes (Sealed Secrets, External Secrets, or
  plaintext `Secret` manifests in the ArgoCD repos?). Check this first once
  there's access.

## Open question

The same org shows **three generations of CI/CD**: CodeStar (2023), Azure
DevOps (2024), and ArgoCD (date unknown). Which one deploys production today,
and can the other two be switched off and their cloud resources deleted?
