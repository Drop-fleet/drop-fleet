# 8. Takeover plan

Each risk is marked **Verified** (seen directly) or **Inferred** (from repo
names or indirect evidence; confirm before acting on it).

## Top 10 risks (prioritized)

| # | Risk | Basis | Why it matters |
|---|------|-------|----------------|
| 1 | **The new lead's tooling can't reach the source code.** 28 of 29 repos are not shared | Verified | Nothing can be maintained, secured, or costed without it |
| 2 | **Code ownership may sit with an outside agency** (`atyantik-*` repos, a contractor-run Azure DevOps account) | Inferred | Vendor lock-in; access could be withdrawn; IP and contract status unclear |
| 3 | **Unknown owners of production accounts** (AWS, Azure DevOps, k8s/ArgoCD, Apple/Google developer accounts, DNS, Telnyx, payments, maps) | Inferred | If you lose an account you lose the product. App store accounts are the hardest to recover |
| 4 | **Fleet and Delivery as forked backends** (`drop-fleet-thedropapp-server` vs `drop-technologies-thedropapp-server`) | Inferred | Every fix has to land twice; drift leads to inconsistent behavior and security gaps |
| 5 | **Secrets in git**, especially plaintext k8s `Secret`s in `argocd-*` | Not assessed | Commonly found in setups like this; would require rotating everything |
| 6 | **Three overlapping CI/CD systems** (CodeStar, which is discontinued; Azure DevOps; ArgoCD) | Verified (2), Inferred (1) | Nobody is sure what deploys prod; orphaned resources keep billing |
| 7 | **Unbounded usage-based costs** (geocoding per ping or order, POS polling, SMS per status change) | Inferred | Typical big line items for delivery platforms; can grow faster than revenue |
| 8 | **End-of-life runtimes/OS** (Amazon Linux 2, Python 3.7, old CodeDeploy agent in `drop-fleet`; likely elsewhere too) | Verified for `drop-fleet` | Security exposure, and app store/SDK deadlines |
| 9 | **v1/v2 API coexistence** (`atyantik-drop-api.v2` vs the `*-thedropapp-server` repos) | Inferred | A half-finished migration doubles the surface to maintain |
| 10 | **No tests / no working CI**. The only CI found can't pass (missing `package.json`) | Verified for `drop-fleet` | Changes become risky, which slows the takeover |

## Recommended next steps

**First (blocking):**
1. Get org-owner access, and grant the audit tool read access to all repos
   ([access-request.md](access-request.md)). Re-run the
   [playbook](audit-playbook.md).
2. Build an account/credential register: every cloud, SaaS, store, and domain
   account, with owner, billing card, MFA holder, and recovery email. Move
   ownership to company-controlled identities.
3. Review the agency/contractor agreements (Atyantik? `azDevops2023`?) for IP
   assignment and handover duties, and rotate any credentials they hold
   once the handover is done.

**Next:**
4. Secret scan all repos and full history; rotate anything found.
5. Pin down which pipeline deploys prod; shut down CodeStar leftovers and
   unused Azure agent pools.
6. Pull the last 3 months of invoices (cloud, maps, SMS, POS vendors, payments)
   and map each line to code using the playbook's §5 call sites.
7. Measure the Fleet/Delivery fork divergence and decide whether to merge into
   one multi-tenant backend or formally split.

**Then:**
8. Add minimal CI to the backends (install, lint, test, build image) and smoke
   tests for the order lifecycle.
9. Plan runtime/OS/dependency upgrades, mobile SDK targets first because the
   stores enforce deadlines.
10. Archive dead repos (starting with `drop-fleet`, the CodeStar sample) and
    make anything that shouldn't be public private.

## Open questions for the team

1. Who are the GitHub org owners, and is any code hosted outside `Drop-fleet`
   (agency GitHub, Azure Repos)?
2. Is Atyantik still engaged? What does the contract say about IP and handover?
   Who is `azDevops2023`?
3. Which backend do the production apps call: the `*-thedropapp-server` repos
   or `atyantik-drop-api.v2`?
4. Are Fleet and Delivery deployed to the same cluster and database or separate ones?
   Which cloud(s)?
5. Which pipeline builds and deploys prod today? Is anything still on the
   CodeStar EC2 stack?
6. Which maps/geocoding, SMS (Telnyx?), payments, push, and POS vendors are
   live, and what does each cost per month?
7. Is POS sync webhook-based or polling? At what interval, and for how many merchants?
8. How are secrets managed in Kubernetes?
9. Who gets paged when production breaks, and where do alerts go?
10. Who holds the Apple Developer and Google Play accounts and the signing keys?
