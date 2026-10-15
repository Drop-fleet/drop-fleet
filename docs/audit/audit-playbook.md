# Audit playbook: how to finish once access is granted

Everything below is read-only. Run it from an empty directory.

## 0. Clone everything

```bash
ORG=Drop-fleet
gh repo list "$ORG" --limit 200 --json name,isFork,isArchived,pushedAt,primaryLanguage,defaultBranchRef,diskUsage \
  > inventory.json
jq -r '.[].name' inventory.json | while read -r r; do
  gh repo clone "$ORG/$r" "$r" -- --no-single-branch || echo "$r" >> inaccessible.txt
done
for d in argocd-df argocd-dd argocd-monitoring; do git -C "$d" checkout origin/main; done
```

Per repo, record: `git log -1 --format=%ci --all`, `git shortlog -sne --all`
(for bus factor), branch count, and whether CI config exists (`.github/workflows`,
`azure-pipelines*.yml`, `buildspec.yml`, `Jenkinsfile`, `bitbucket-pipelines.yml`).

## 1. Inventory and fork divergence

```bash
A=drop-fleet-thedropapp-server; B=drop-technologies-thedropapp-server
git -C $A remote add other ../$B && git -C $A fetch other
git -C $A merge-base HEAD other/HEAD || echo "no shared history (copy-paste fork)"
git -C $A diff --stat HEAD other/HEAD | tail -1
git -C $A diff --name-status HEAD other/HEAD | awk '{print $2}' | cut -d/ -f1-2 | sort | uniq -c | sort -rn | head -30
```

If there's no shared history, use `diff -rq --exclude=node_modules --exclude=.git $A $B`.
Classify each difference as branding/config, a feature only on one side, or a
fix applied to only one side. Do the same for any driver/consumer app pairs.

## 2. Architecture

- Entry points: `package.json` `main`/`scripts`, `Dockerfile` `CMD`, `Procfile`, k8s `Deployment` images.
- Datastores: `rg -n "mongoose|sequelize|typeorm|prisma|knex|pg\b|mysql|redis|ioredis|DynamoDB|firestore"`.
- Queues/schedulers: `rg -n "bull|bullmq|agenda|node-cron|cron\.schedule|setInterval\(|SQS|amqplib|kafka"`.
- Third parties: `rg -n -i "googleapis|maps\.google|geocode|mapbox|here\.com|telnyx|twilio|stripe|square|clover|toast|olo|deliverect|firebase-admin|fcm|apns|onesignal|sendgrid|mailgun|sentry"`.
- Inter-service calls: `rg -n "axios|fetch\(|got\(|request\(" | rg -i "http"` and base URL env vars.

## 3. Infrastructure

- `argocd-*`: list every `Application`/`ApplicationSet`, `targetRevision`,
  destination cluster/namespace, image tags (pinned vs `latest`), resource
  requests/limits, HPA, and `Ingress` hosts (these map out the environments and domains).
- `kind: Secret` with `data:` or `stringData:` in git is a **critical** finding.
- `docker`: base images and their age, whether containers run as root, multi-stage builds.
- `alert`/`argocd-monitoring`: alert rules, receivers, on-call routing.

## 4. Data model and key flows

- Dump the schemas (Mongoose models, migrations, Prisma schema) into an ER diagram.
- Trace these flows end to end, from HTTP route to DB writes to outbound calls to
  socket emits:
  order created (app / POS) → priced → dispatched/assigned → driver
  accepts → pickup → en route (tracking) → delivered → payment
  captured/payout. Note where order status values are defined, and check
  whether those definitions are duplicated across services.

## 5. Cost drivers

- **Geocoding:** count call sites (`rg -n -i "geocode|distancematrix|directions|places"`),
  check for caching (lat/lng stored on address records? Redis?), and look for
  calls inside loops or on every location ping.
- **POS sync:** polling interval (`setInterval`/cron expressions in
  `pos-integration`), per-merchant fan-out, and whether webhooks are available
  but unused.
- **API polling:** client-side `setInterval`/`Timer`/`Handler.postDelayed` in apps
  and PWAs hitting status endpoints. These should be replaced by the existing
  Socket.IO channel.
- **Tracking:** location ping frequency from the driver apps, and whether every
  ping is a DB write.
- **SMS:** per-message Telnyx sends per order status change, and retry loops.
- **Infra:** replica counts and requests in `argocd-*`, idle environments,
  CodeStar EC2 leftovers, and the Azure VMSS agent pool size.

## 6. Security

```bash
docker run --rm -v "$PWD:/repo" zricethezav/gitleaks:latest detect -s /repo/<repo> --redact --log-opts="--all"
rg -n -i "(api[_-]?key|secret|passw(or)?d|token|private[_-]?key)\s*[:=]" --glob '!**/node_modules/**'
rg -n "AKIA[0-9A-Z]{16}|AIza[0-9A-Za-z_-]{35}|sk_live_|KEY[0-9A-F]{6,}_"   # AWS, Google, Stripe, Telnyx-style keys
```

Only record file paths, never values. Then check:
- JWT: where the secret comes from, `expiresIn`, `algorithms` pinned, refresh flow.
- Authorization: can driver A read or modify driver B's or another merchant's orders?
  (Look for ID-only lookups without an owner check.)
- Socket.IO: is the handshake authenticated? Are room joins authorized?
- Webhooks: is the signature verified (payments, POS, Telnyx)?
- Deps: `npm audit --omit=dev`, `npx npm-check-updates`, Node/RN/Gradle/Xcode
  SDK versions vs current store requirements.

## 7. Quality, tests, bus factor

- Test presence and `npm test` / CI coverage; lint configs; TypeScript vs JS.
- `git shortlog -sne --since=2.years` per repo. If one person (or one agency)
  wrote over 80% of the code, that's a bus-factor risk.
- Look for dead code, duplicated services, and commented-out credentials.
