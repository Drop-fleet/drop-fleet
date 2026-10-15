# Drop Fleet / Drop Delivery — Technical Audit

> **Status: BLOCKED (partial audit).** Audit date: 2026-10-15. Read-only.
>
> The requested scope was all 29 repositories in the GitHub org
> [`Drop-fleet`](https://github.com/Drop-fleet). The credentials available to
> the auditor (a GitHub App installation token) can see **exactly one**
> repository: [`Drop-fleet/drop-fleet`](https://github.com/Drop-fleet/drop-fleet).
> Every other repo returns `404 Not Found` from both `git ls-remote` and the
> REST API, which is how GitHub responds when a private repo exists but the
> caller has no access. The org reports `public_repos: 1`, and no SSH keys,
> PATs, or other GitHub credentials were present in the environment.
>
> As a result, **no product source code (servers, apps, POS integration,
> ArgoCD manifests, etc.) was read.** This document records what *was*
> verified, labels everything else as unverified, and gives the exact steps to
> finish the audit once access is granted. Nothing here should be read as a
> finding about the Fleet or Delivery codebases unless it is marked
> **Verified**.

## Executive summary

1. **The only accessible repo, `drop-fleet`, is not product code.** It is the
   untouched AWS CodeStar "static HTML website" sample (one commit by `AWS`,
   2023-08-29) plus a side branch `pipeline-qa-master` holding 19 trial-and-error
   commits of an Azure DevOps pipeline (2024-08-21, user `azDevops2023`). It
   is public, deploys nothing Drop-specific, and is effectively dead. See
   [docs/audit/drop-fleet-repo.md](docs/audit/drop-fleet-repo.md).
2. **Access is itself the top takeover risk.** A new technical lead cannot
   take over a platform whose source is not reachable. Who holds org-owner
   rights, and whether code exists only in agency accounts (for example
   Atyantik, going by the `atyantik-*` repo names), needs to be settled before
   anything else. See [docs/audit/access-request.md](docs/audit/access-request.md).
3. **Signals worth following up, all unverified:** at least three CI/CD systems
   appear to be in play (AWS CodeStar/CodeDeploy, which is verified;
   Azure DevOps, verified on a branch; ArgoCD, inferred from repo names). The
   repo names `drop-fleet-thedropapp-server` and
   `drop-technologies-thedropapp-server` point to Fleet and Delivery being forks
   of one backend, which would mean fixes have to be made twice. Agency-prefixed
   repo names (`atyantik-*`) point to bus-factor risk sitting with an outside
   vendor.

## Sections

| # | Section | Status | Document |
|---|---------|--------|----------|
| 1 | Repo inventory, Fleet-vs-Delivery fork divergence | 1/29 verified | [docs/audit/repo-inventory.md](docs/audit/repo-inventory.md) |
| 2 | System architecture (mermaid) | Hypothesis only | [docs/audit/architecture.md](docs/audit/architecture.md) |
| 3 | Infrastructure and deployment | Partially verified | [docs/audit/infrastructure.md](docs/audit/infrastructure.md) |
| 4 | Data model and key flows | Not started (no access) | [docs/audit/audit-playbook.md#4-data-model-and-key-flows](docs/audit/audit-playbook.md#4-data-model-and-key-flows) |
| 5 | Cost drivers | Not started (no access) | [docs/audit/audit-playbook.md#5-cost-drivers](docs/audit/audit-playbook.md#5-cost-drivers) |
| 6 | Security | `drop-fleet` verified; rest not started | [docs/audit/security.md](docs/audit/security.md) |
| 7 | Code quality, tests, tech debt, bus factor | `drop-fleet` verified; rest not started | [docs/audit/drop-fleet-repo.md](docs/audit/drop-fleet-repo.md) |
| 8 | Takeover plan: top 10 risks, next steps, open questions | Done (on current knowledge) | [docs/audit/takeover-plan.md](docs/audit/takeover-plan.md) |

## How to finish this audit

1. An org owner grants read access to all repos (see
   [access-request.md](docs/audit/access-request.md)). The quickest way is to set
   the GitHub App installation on `Drop-fleet` to **All repositories**.
2. Re-run the audit. [audit-playbook.md](docs/audit/audit-playbook.md) has the
   clone script and, for each section, the exact searches and checks (geocoding
   call sites, POS polling loops, Telnyx usage, secret scanning, dependency
   audits, Fleet-vs-Delivery diffs).
3. Replace each "Unverified"/"Not started" section with findings that include
   file and line references.

## Method (what was actually done)

- `gh repo list Drop-fleet`, `gh api orgs/Drop-fleet/repos?type=all`,
  `gh api installation/repositories`: all return only `drop-fleet`
  (installation `total_count: 1`).
- `git ls-remote` and `gh api repos/Drop-fleet/<name>` on each repo named in
  the brief: all `404`.
- Public GitHub search for the repo names: no public mirrors or forks found.
  The `DropFleet` repos that do turn up belong to unrelated users.
- For `drop-fleet`: read every file on both branches, reviewed the full git
  history, and grepped all history for secrets (AWS keys, Google API keys,
  Slack and Stripe tokens, private keys, generic `password/secret/token`).
  Nothing found.
