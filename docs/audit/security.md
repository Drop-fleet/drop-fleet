# 6. Security

## Verified findings: `drop-fleet`

No hardcoded secrets were found in any commit on any branch. Patterns checked:
AWS access keys, Google API keys, Slack/Stripe tokens, PEM private keys, and
generic `password|secret|token|api_key`.

| Severity | Finding | Location |
|----------|---------|----------|
| High (if deployed) | SSH (22) and HTTP (80) open to `0.0.0.0/0`, no HTTPS | `template.yml` → `WebAppSG` |
| High (if deployed) | OS/runtime past end of support: Amazon Linux 2 extras, Python 3.7 (EOL 2023-06), Ruby 2.6 (EOL 2022-03), pip 9/20.2.1 | `template.yml` UserData, `scripts/install_python3` |
| Medium | Outdated pinned deps: `pycryptodome==3.9.7` (2020, has known advisories), `awscli==1.27.84`, `boto3==1.26.84`, CodeDeploy agent 1.4.0 | `scripts/requirements.txt`, `template.yml` |
| Medium | Root-run scripts fetched from remote URLs with no integrity check (`get_authorized_keys` from S3, nvm `install.sh` piped to bash in CI) | `scripts/codestar_remote_access`, `azure/azure-pipeline-pipeline-qa.yaml` (branch) |
| Low | `CPUCredits: unlimited` on t3 can lead to surprise bills | `template.yml` |
| Low | Repo is **public**, which advertises infra layout and the org name. Make it private or archive it | repo settings |

## Not assessed (no access)

- Hardcoded secrets in the server/app/POS/ArgoCD repos. Method:
  [audit-playbook.md §6](audit-playbook.md#6-security).
- Auth design: JWT secret handling, token expiry, driver-vs-consumer-vs-admin
  role checks, Socket.IO handshake auth, and webhook signature verification
  (POS, payments, Telnyx).
- Dependency vulnerabilities (`npm audit`, `pip-audit`, Gradle/CocoaPods).
- Mobile apps: API keys in the bundle (Maps keys without app restrictions),
  certificate pinning, and whether location data is stored on the device.

**Rule for the follow-up audit:** report secret *locations* (file:line, commit)
only, never values. Rotate anything found, even in old history.
