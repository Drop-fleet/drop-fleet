# Access request: what is needed to complete the audit

## Current state (verified 2026-10-15)

| Check | Result |
|-------|--------|
| `gh api installation/repositories` | `total_count: 1` → `Drop-fleet/drop-fleet` |
| `gh api orgs/Drop-fleet/repos?type=all` | only `drop-fleet` |
| `gh api orgs/Drop-fleet` | `public_repos: 1`; private repo count hidden from this token |
| `git ls-remote` on each named repo | `remote: Repository not found` |
| Branch protection / org settings | `403 Resource not accessible by integration` |

Repos in the brief that could not be accessed: `drop-fleet-thedropapp-server`,
`drop-technologies-thedropapp-server`, `atyantik-drop-api.v2`,
`atyantik-drop-tracking`, `pos-integration`, `alert`, `docker`, `argocd-df`,
`argocd-dd`, `argocd-monitoring`, plus the driver and consumer iOS/Android
apps, PWAs, Socket.IO chat, automation repos, and marketing/homepage repos.
Their exact names are unknown because the private repo list isn't visible.

## Options for granting access (pick one)

1. **Preferred:** an org owner opens *GitHub → Drop-fleet → Settings → GitHub
   Apps → (Cursor app) → Configure → Repository access* and picks **All
   repositories**, or selects all 29. Read access is enough.
2. Add a dedicated machine user with org-level **Read** to all repos and supply
   its fine-grained PAT (contents:read, metadata:read, actions:read) as a Cloud
   Agent secret, e.g. `DROP_GH_TOKEN`.
3. Export each repo as a `git bundle` (`git bundle create <repo>.bundle --all`)
   and supply the bundles.

## Things to confirm while doing this

- Who are the org **owners** today? (The audit token can't list members.)
- Are any repos actually hosted outside the org, for example in the Atyantik
  agency's GitHub or in Azure DevOps Repos (user `azDevops2023` pushed an
  `agentPoolScaleSet1` pipeline), with the `Drop-fleet` copies only being
  mirrors?
- Who controls the AWS account(s) behind the CodeStar project, the Azure DevOps
  org, the Kubernetes clusters/ArgoCD, the Apple and Google developer accounts,
  and the domain registrar/DNS?
