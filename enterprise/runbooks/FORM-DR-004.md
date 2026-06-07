# FORM-DR-004 · Cloudflare Workers Degradation & Origin Fallback Runbook

> **Runbook ID:** FORM-DR-004 · **Version:** 1.0 · **Last tested:** _(pending first drill — see §6)_
> **Classification:** Internal — Enterprise Confidential. Share under NDA only.
> **Owner:** `devops-lead`. Execution: on-call engineer.
> **SOC 2 controls:** CC7.4, CC7.5, A1.2, A1.3.
> **RTO target:** Cloudflare-dependent (platform outage) / **15 min** (FORM Worker bug rollback).
> **Related:** `docs/BUSINESS_CONTINUITY.md §6.2`, `docs/INCIDENT_RESPONSE.md`, `docs/OBSERVABILITY.md`.
> **DEC-030:** Audit events must be emitted even when API is degraded; see §5.

---

## Purpose

This runbook covers two distinct failure modes:

1. **Cloudflare platform incident** — Cloudflare's global edge is degraded or unavailable. FORM has no secondary edge provider (documented risk R-004 in `docs/SOC2_READINESS.md §14`). Recovery is primarily wait-and-communicate.

2. **FORM Worker bug** — A deployment to production introduced a regression causing 5xx errors or incorrect behaviour. Recovery is a Worker rollback executed in < 15 minutes.

Both failure modes cause the same user-visible symptom (API errors) but require completely different responses. **Do not skip §1 failure classification.**

**Activation triggers (any one sufficient):**

- Better Uptime global probe checks fail from **multiple geographic locations simultaneously** → PagerDuty P1 alert.
- Cloudflare Analytics: 5xx rate on any Workers route > 20% for > 5 minutes.
- Enterprise tenant reports complete API unavailability (all endpoints failing).
- On-call engineer confirms `wrangler tail` shows unhandled exceptions or Worker boot failures.

---

## Pre-Execution Checklist

Estimated time: **< 3 minutes**.

```
[ ] I have acknowledged the PagerDuty alert and set myself as Incident Commander (IC).
[ ] I have wrangler CLI authenticated: `wrangler whoami` shows production org.
[ ] I have access to Cloudflare dashboard (dash.cloudflare.com) → Workers.
[ ] I have checked cloudflarestatus.com — screenshot current status (attach to post-mortem).
[ ] I have notified the founder via direct message: "Activating FORM-DR-004 — Workers failure".
[ ] I have a running notes document open. Log every action with timestamp.
```

---

## §1 — Failure Classification (< 5 min)

```bash
# Step 1a: Check Better Uptime probe locations
# Go to betteruptime.com → your monitor → check which probe locations are failing
# If ALL global probes fail simultaneously → likely Cloudflare platform
# If only SOME probes fail, or errors are logic errors (not connection) → likely FORM Worker bug

# Step 1b: Check Cloudflare Status Page
open https://cloudflarestatus.com
# Active incident? Screenshot it.

# Step 1c: Check Worker logs
wrangler tail --env production --format pretty 2>&1 | head -50
# Boot/uncaught errors → FORM Worker bug
# Connection timeout → Cloudflare platform OR Supabase (check status.supabase.com too)

# Step 1d: Check recent deployments
git log --oneline -5  # Was there a recent deploy to main?
wrangler deployments list --env production  # Last 5 deployments with timestamps
```

**Decision:**

```
Cloudflare Status Page shows active platform incident?
  ├── YES → Continue to §2 (Platform Incident Response).
  └── NO  → Check Worker logs for application errors.
             Worker bug confirmed?
               ├── YES → Continue to §3 (Worker Bug Rollback).
               └── NO  → Check Supabase status.supabase.com.
                          Supabase issue? → Use FORM-DR-003.
                          Unknown root cause → escalate to founder.
```

---

## §2 — Cloudflare Platform Incident Response

### Step 2.1 — Enable maintenance mode

Cloudflare platform incidents mean the Workers runtime is unavailable. The maintenance flag in Cloudflare KV may itself be unreachable. If the KV write fails, skip to 2.2 — the 503 response from Cloudflare's own error page will serve as the maintenance signal.

```bash
# Attempt maintenance mode flag (may fail during Cloudflare outage — that is acceptable)
wrangler kv key put --binding=SYSTEM_KV "system:maintenance" "1" --env production 2>&1 \
  | tee /tmp/dr004-maint-flag.log || echo "KV write failed — Cloudflare platform outage confirmed"
```

### Step 2.2 — Update status page

If the Cloudflare Workers-served `form.coach/status` is also unreachable, update any out-of-band channel available:

- **Better Uptime status page** (betteruptime.com → Status Pages) — does not depend on FORM infrastructure.
- **Enterprise customer Slack Connect channels** — direct message to each tenant admin.

Use template (`docs/INCIDENT_RESPONSE.md §8 E-02`):

```
Subject: FORM API Unavailable — Cloudflare Platform Incident [DATE TIME UTC]

FORM API is currently unavailable due to a Cloudflare infrastructure incident.
This is a third-party platform issue outside FORM's control.
Status: https://cloudflarestatus.com
We are monitoring and will restore service immediately upon Cloudflare recovery.
Updates every 30 minutes.
```

### Step 2.3 — Monitor Cloudflare recovery

- Check `cloudflarestatus.com` every **10 minutes**. Log status in running notes.
- No FORM-side recovery action is possible during a Cloudflare platform incident.
- If incident persists > **4 hours** (enterprise contractual RTO): invoke `docs/ENTERPRISE_SLA.md §5` SLA credit procedure. Notify `compliance-officer`.

### Step 2.4 — Post-recovery validation

When Cloudflare confirms the incident is resolved, proceed to **§4 (Post-Recovery Validation)** before announcing recovery to customers.

---

## §3 — FORM Worker Bug Rollback (< 15 min)

This is the fast path. A FORM Worker bug is entirely within FORM's control and should be resolved quickly.

### Step 3.1 — Identify the offending deployment

```bash
# List recent deployments
wrangler deployments list --env production
# Output shows: deployment ID, timestamp, git commit SHA

# Identify last known-good deployment (before 5xx errors started)
# Cross-reference with Sentry first-seen timestamp for the error
```

### Step 3.2 — Rollback to previous version

```bash
# Get the deployment ID of the last known-good version from Step 3.1
GOOD_DEPLOYMENT_ID="<deployment-id>"

# Rollback
wrangler rollback "$GOOD_DEPLOYMENT_ID" --env production

# Verify rollback
wrangler tail --env production --format pretty 2>&1 | head -20
# Should show no boot errors
```

**If `wrangler rollback` is unavailable or fails:**

```bash
# Manual rollback: deploy the previous git commit directly
git log --oneline -5  # Find the last known-good commit SHA
GOOD_COMMIT="<sha>"

git stash  # Stash any local changes
git checkout "$GOOD_COMMIT" -- worker/  # Checkout only the worker directory
wrangler deploy --env production
git checkout main -- worker/  # Restore main branch worker files
git stash pop
```

### Step 3.3 — Verify rollback

```bash
# Confirm the Worker is running the expected version
wrangler tail --env production --format pretty 2>&1 | grep "deployment_id" | head -5

# Check health endpoint
curl -sf https://api.form.coach/api/health | jq '.version, .status'
# Expected: status "ok", version matches the rolled-back commit

# Quick smoke test
curl -sf https://api.form.coach/api/health | jq '.status' # Expected: "ok"
```

If health check passes after rollback: proceed to **§4 (Post-Recovery Validation)**.

### Step 3.4 — Root cause for the breaking deployment

While the service is recovering, note the offending commit for post-mortem:

```bash
# Get the diff that introduced the regression
git show <breaking-commit-sha> --stat
git show <breaking-commit-sha> -- worker/  # Full diff
```

File the finding in the running incident notes. The post-mortem action item must include a CI test that would have caught this regression.

---

## §4 — Post-Recovery Validation

Execute before announcing recovery to customers. Estimated time: **30 minutes**.

### Step 4.1 — Smoke test suite (F1–F8 flows)

```bash
# F1: Health
curl -sf https://api.form.coach/api/health | jq '.status'
# Expected: "ok"

# F2: Auth
curl -sf -H "Authorization: Bearer $TEST_REFRESH_TOKEN" \
  https://api.form.coach/api/auth/refresh | jq '.access_token'
# Expected: non-null JWT

# F3: Workout read
curl -sf -H "Authorization: Bearer $TEST_ACCESS_TOKEN" \
  https://api.form.coach/api/workouts?limit=1 | jq '.data'
# Expected: array (may be empty)

# F4: Workout write
curl -sf -X POST -H "Authorization: Bearer $TEST_ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"type":"strength","duration_sec":1}' \
  https://api.form.coach/api/workouts | jq '.id'
# Expected: UUID

# F5: Enterprise SSO health
curl -sf "https://api.form.coach/api/sso/health?tenant_id=$TEST_ENTERPRISE_TENANT" \
  | jq '.saml_configured'
# Expected: true

# F6: Admin aggregate (Privacy Floor check)
curl -sf -H "Authorization: Bearer $TEST_ADMIN_TOKEN" \
  "https://api.form.coach/api/admin/wellness-aggregate?tenant_id=$TEST_ENTERPRISE_TENANT" \
  | jq 'has("user_id")'
# Expected: false (Privacy Floor must hold)

# F7: Audit log write
curl -sf -X POST -H "Authorization: Bearer $TEST_ACCESS_TOKEN" \
  https://api.form.coach/api/debug/audit-ping | jq '.hmac_valid'
# Expected: true

# F8: SCIM ServiceProviderConfig
curl -sf -H "Authorization: Bearer $TEST_SCIM_TOKEN" \
  "https://api.form.coach/scim/v2/ServiceProviderConfig" | jq '.schemas'
# Expected: non-empty array
```

All 8 tests must pass. Log results with timestamps.

### Step 4.2 — Verify Worker version is expected

```bash
# Confirm the live Worker version matches what you intended to deploy/rollback to
curl -sf https://api.form.coach/api/health | jq '.git_sha'
# Compare with: git log --oneline -1 (for current deploy) or known-good SHA (for rollback)
```

### Step 4.3 — Check for queued background jobs

If FORM uses Cloudflare Queues for any async processing (e.g., HMAC audit chain writes during high load):

```bash
# Check queue depth (if applicable)
wrangler queues list --env production
# If any queue shows a backlog > expected: monitor consumer throughput before lifting maintenance
```

---

## §5 — Lift Maintenance Mode & Recovery Communication

Only after all §4 checks pass.

```bash
# Remove maintenance flag
wrangler kv key delete --binding=SYSTEM_KV "system:maintenance" --env production

# Verify API is live
curl -sf https://api.form.coach/api/health | jq '.status'
# Expected: "ok"
```

Update status page: **All Systems Operational**.

Send recovery notification to enterprise tenants (template `docs/INCIDENT_RESPONSE.md §8 E-04`):

```
Subject: FORM Service Restored — [DATE TIME UTC]

FORM services have been fully restored as of [TIME UTC].
Duration of impact: [X hours Y minutes].
Root cause: [Cloudflare platform incident (third-party) / FORM Worker regression (internal)].
Action taken: [Monitoring until Cloudflare resolved / Rollback to version [SHA] deployed].
Follow-up: Full post-incident report within 5 business days.
SLA credit: [Automatic per docs/ENTERPRISE_SLA.md §5 / Not applicable].
```

Emit audit events:

```sql
-- Emit via direct Supabase insert (Supabase must be available for this step)
INSERT INTO audit_events (event_type, severity, actor_type, metadata)
VALUES ('system.maintenance_lifted', 'HIGH', 'system',
  '{"incident_ref": "FORM-DR-004-[DATE]",
    "failure_type": "cloudflare_platform" | "worker_bug",
    "rto_minutes": [X], "smoke_tests_passed": 8,
    "rollback_deployed": false | true,
    "rollback_from_sha": "[breaking-sha]",
    "rollback_to_sha": "[good-sha]",
    "lifted_by": "[engineer_user_id]"}');
```

---

## §6 — Evidence Filing & Post-Mortem

### Evidence to file

```
[ ] Screenshot: cloudflarestatus.com status page (during incident).
[ ] Screenshot: Better Uptime probe failure map (global vs regional).
[ ] Export: PagerDuty alert timeline.
[ ] Export: wrangler tail output (error logs during incident).
[ ] Export: wrangler deployments list (for Worker bug scenarios).
[ ] Export: Running notes document.
[ ] Smoke test results (F1–F8 pass/fail timestamps).
[ ] Git diff of offending commit (for Worker bug scenarios).
```

File to: `compliance/evidence/cc7/incidents/FORM-DR-004-[DATE]/`

### Post-mortem

Write within **5 business days** using `docs/INCIDENT_RESPONSE.md §8` template. Required sections:
- Timeline (detection through restoration)
- Root cause (platform vs Worker bug; for Worker bug: what regression, what CI gap allowed it)
- Customer impact and SLA credit calculation if applicable
- Action items (for Worker bugs: mandatory CI test preventing regression recurrence)

### DR drill evidence

If this was a planned drill:

```
[ ] File: enterprise/runbooks/test-reports/FORM-DR-004-drill-[DATE].md
[ ] Update §1 header: "Last tested: [DATE] — [Engineer name]"
[ ] Update: compliance/cc9/README.md CC9-GAP-005 if drill is complete.
```

---

## §7 — Known Gaps & Limitations

| Gap ID | Description | Mitigation |
|---|---|---|
| **R-004** | No secondary edge/CDN provider. Cloudflare platform outage means full API unavailability. | Cloudflare's historical uptime > 99.99%; global anycast redundancy. Secondary provider evaluation: Post-Series-A. |
| **BCP-03** | Sole on-call engineer pre-Series A. | Written runbooks; all credentials in 1Password team vault. First hire receives full production access + runbook training within 30 days. |
| Worker rollback latency | Wrangler rollback typically deploys in < 60 seconds but can take up to 5 minutes in practice. | Pre-validation in staging before any production deploy reduces rollback probability. |
| Queue consumer catch-up | If Cloudflare Queues accumulated a large backlog during platform outage, consumer throughput may lag 10–30 minutes post-recovery. | Monitor queue depth post-recovery before announcing full normalisation. |

---

## §8 — Quick Reference Card

| Condition | Action |
|---|---|
| All global probes failing + Cloudflare incident active | §2 platform response — monitor + communicate |
| Worker boot errors in `wrangler tail` | §3 Worker bug rollback |
| Rollback succeeds, health passes | §4 smoke test → §5 lift maintenance |
| Cloudflare incident > 4 hours | SLA credit + notify compliance-officer |
| F6 Privacy Floor test fails | **Stop. Call compliance-officer. Do not lift maintenance.** |
| All F1–F8 pass | §5 lift maintenance → recovery notification |

---

## §9 — Contact Escalation Tree

| Role | Contact | When |
|---|---|---|
| **Founder** | 1Password → "Emergency Contacts" | Cloudflare incident > 2 hours; any P0 scenario |
| **compliance-officer** | 1Password → "Emergency Contacts" | Privacy Floor failure; SLA credit > $10k; GDPR breach risk |
| **Cloudflare Enterprise Support** | dash.cloudflare.com → Support | Cloudflare platform incident — get ETA; file support ticket |
| **Customer CSM leads** | Linear → "Customer Contacts" | Enterprise tenant impact > 30 minutes |

---

*v1.0 · 2026-06-07 · Owner: devops-lead*
*Approved by: compliance-officer · Next review: after first drill execution or any Cloudflare contract/architecture change*
*SOC 2 evidence: CC7.4, CC7.5, A1.2, A1.3 · Risk register: R-004 (`docs/SOC2_READINESS.md §14`)*
