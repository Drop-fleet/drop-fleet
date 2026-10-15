# 1. Repository inventory

The brief says the org has 29 repos. One is accessible. The rest are listed
using the names given in the brief. "Product" and "Purpose" for inaccessible
repos are **guesses from the name** and must be confirmed.

| Repo | Access | Product (guess unless verified) | Purpose (guess unless verified) | Lang/framework | Last activity | Active? |
|------|--------|-------------------------------|---------------------------------|----------------|---------------|---------|
| `drop-fleet` | **Verified** (public) | Neither (scaffold) | AWS CodeStar sample static site + Azure Pipelines experiment | HTML/CSS, CloudFormation, bash, Azure Pipelines YAML | `master`: 2023-08-29; `pipeline-qa-master`: 2024-08-21 | **Dead.** Never customized |
| `drop-fleet-thedropapp-server` | Inaccessible | Fleet | Main backend API | ? | ? | ? |
| `drop-technologies-thedropapp-server` | Inaccessible | Delivery | Main backend API, probably a fork of the Fleet server (or the reverse) | ? | ? | ? |
| `atyantik-drop-api.v2` | Inaccessible | ? (possibly shared) | v2 API built by agency Atyantik | ? | ? | ? |
| `atyantik-drop-tracking` | Inaccessible | ? | Driver location tracking service | ? | ? | ? |
| `pos-integration` | Inaccessible | Delivery? | Point-of-sale order ingestion/sync | ? | ? | ? |
| Driver iOS / Android | Inaccessible | Both? | Driver apps | ? | ? | ? |
| Consumer iOS / Android | Inaccessible | Delivery? | Customer ordering/tracking apps | ? | ? | ? |
| PWAs | Inaccessible | ? | Web clients (dispatcher/consumer?) | ? | ? | ? |
| Socket.IO chat | Inaccessible | Shared? | Real-time chat (driver to customer/dispatch) | Node.js (Socket.IO) | ? | ? |
| `docker` | Inaccessible | Shared | Container build definitions / compose | ? | ? | ? |
| Automation repos | Inaccessible | ? | Scripts/cron/QA automation | ? | ? | ? |
| Marketing / homepage | Inaccessible | ? | Public website | ? | ? | ? |
| `alert` | Inaccessible | Shared | Alerting service or alert rules | ? | ? | ? |
| `argocd-df` | Inaccessible | Fleet | ArgoCD app manifests (read from `origin/main`) | YAML/Helm/Kustomize? | ? | ? |
| `argocd-dd` | Inaccessible | Delivery | ArgoCD app manifests (read from `origin/main`) | YAML/Helm/Kustomize? | ? | ? |
| `argocd-monitoring` | Inaccessible | Shared | Monitoring stack (Prometheus/Grafana?) | YAML | ? | ? |

## Fleet vs Delivery fork divergence

**Not assessed.** It needs both `*-thedropapp-server` repos. The method is in
[audit-playbook.md §1](audit-playbook.md#1-inventory-and-fork-divergence):
find the merge base or common root commit, `git diff --stat` the two, and
classify each difference as config/branding, feature-only-in-one, or a bug fix
applied to one side only.
