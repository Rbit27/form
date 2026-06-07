# FORM-DR-003 · Supabase Regional Failure & Failover Runbook

> **Runbook ID:** FORM-DR-003 · **Version:** 1.0 · **Last tested:** _(pending first drill — see §7)_
> **Classification:** Internal — Enterprise Confidential. Share under NDA only.
> **Owner:** `devops-lead` + `platform-engineer`. Execution: on-call engineer.
> **SOC 2 controls:** CC7.4, CC7.5, A1.2, A1.3, CC9.1.
> **RTO target:** 4 hours (enterprise) · **RPO target:** 1 hour (enterprise).
> **Related:** `docs/BUSINESS_CONTINUITY.md §6.1`, `docs/INCIDENT_RESPONSE.md §5.2`, `docs/DATA_MODEL.md §10`.
> **DEC-030:** HMAC audit chain must remain intact across restore; see §6.

---

## Purpose

This runbook provides step-by-step recovery instructions when the Supabase Postgres primary is unreachable, degraded, or confirmed as a regional failure. It is designed to be executed by any competent engineer **without tribal knowledge from the author**.

**Activation triggers (any one sufficient):**

- `GET /api/health` returns non-2xx on 3 consecutive 1-minute Better Uptime checks → PagerDuty P1 alert.
- Sentry: database connection error spike > 50 errors / minute for > 3 consecutive minutes.
- Cloudflare Analytics: 5xx rate on any database-dependent endpoint > 20% for > 5 minutes.
- On-call engineer manually confirms Supabase dashboard shows project unavailable.

**Do NOT activate** if the failure is confirmed as a FORM Worker bug or migration error — use `docs/INCIDENT_RESPONSE.md §5.2` instead.

---

## Pre-Execution Checklist

Complete before any recovery steps. Estimated time: **< 5 minutes**.

```
[ ] I have acknowledged the PagerDuty alert and set myself as Incident Commander (IC).
[ ] I have access to 1Password → "Production / Supabase" vault entry (service key, PITR credentials).
[ ] I have access to Supabase dashboard (dashboard.supabase.com) and project console.
[ ] I have Cloudflare Workers wrangler CLI authenticated (`wrangler whoami` shows production org).
[ ] I have checked status.supabase.com — screenshot the current incident status (attach to post-mortem).
[ ] I have notified the founder via direct message: "Activating FORM-DR-003 — Supabase failure".
[ ] Severity is declared: [ ] P0 (enterprise + consumer affected) [ ] P1 (enterprise affected) [ ] P1 (consumer only).
[ ] Start a running notes document (Linear incident ticket, Notion page, or plain text file). Log every action with timestamp.
```

---

## §1 — Failure Classification (< 5 min)

Before executing any recovery steps, classify the failure type. This determines the recovery path.

| Check | Command / Source | Expected result for FORM-side | Expected result for Supabase platform |
|---|---|---|---|
| Supabase Status Page | `status.supabase.com` | All green | Active incident |
| Supabase dashboard | `dashboard.supabase.com` | Project shows "Healthy" | Project shows "Degraded" or unreachable |
| Supabase logs | Dashboard → Logs → API | FORM query errors | Supabase infra errors |
| Cloudflare Workers log | `wrangler tail --env production` | Database connection errors | Possibly connection-level timeouts |

**Decision:**

```
Supabase platform incident confirmed?
  ├── YES → Continue to §2 (Platform Incident Response).
  └── NO → FORM-side issue → invoke docs/INCIDENT_RESPONSE.md §5.2 (Database Incident).
            Do NOT continue this runbook.
```

---

## §2 — Platform Incident Response (T+5 to T+30 min)

When Supabase confirms a platform-side incident:

### Step 2.1 — Enable maintenance mode

```bash
# Set maintenance flag in Cloudflare KV (key: system:maintenance, value: 1)
wrangler kv key put --binding=SYSTEM_KV "system:maintenance" "1" --env production
```

Verify: `curl -s https://api.form.coach/api/health` should return HTTP 503 with body:
```json
{"status":"maintenance","retry_after":1800,"message":"Scheduled maintenance. Check form.coach/status."}
```

### Step 2.2 — Update status page

- Go to `form.coach/status` admin panel (Cloudflare Workers route `/admin/status`).
- Set status: **Degraded — Database maintenance**.
- ETA: Use Supabase's published ETA if available; otherwise write "Investigating".

### Step 2.3 — Notify enterprise tenants

Send within **10 minutes** of maintenance mode activation. Use template from `docs/INCIDENT_RESPONSE.md §8 E-02`:

```
Subject: FORM Service Degradation — [DATE TIME UTC]

We are currently experiencing a database infrastructure incident affecting FORM services.
Affected: All API endpoints requiring data reads/writes.
Status: Investigating. We are monitoring the Supabase platform incident.
ETA: [Insert Supabase ETA or "under investigation"].
Updates: form.coach/status (every 30 minutes).
```

Channels: Enterprise customer Slack Connect channels + CSM direct message. Log the notification event:

```sql
-- Emit to audit_events (DEC-030 HMAC-chained)
INSERT INTO audit_events (event_type, severity, actor_type, metadata)
VALUES ('system.maintenance_activated', 'HIGH', 'system',
  '{"reason": "supabase_platform_incident", "incident_ref": "FORM-DR-003-[DATE]",
    "runbook_version": "1.0", "maintenance_start": "[ISO8601_NOW]"}');
```

### Step 2.4 — Monitor and hold (T+10 to T+120 min)

- Check Supabase Status Page every **15 minutes**. Log status in running notes.
- If Supabase recovery is confirmed: proceed to §5 (Post-Recovery Validation).
- If no resolution after **2 hours**: escalate to founder; proceed to §3 (PITR Initiation).
- If outage exceeds **4 hours** (enterprise RTO): initiate PITR regardless of Supabase timeline. Proceed to §3.

---

## §3 — PITR Initiation (T+120 min, if applicable)

Initiate only if Supabase has **not** recovered within 2 hours OR the 4-hour enterprise RTO is at risk.

**Requires founder approval before initiating PITR.** Log approval with timestamp.

```
[ ] Founder approval received at: _____________ (timestamp)
[ ] Approval channel: [ ] Slack DM [ ] Phone [ ] In-person
[ ] Approval logged in running notes.
```

### Step 3.1 — Identify recovery point

Determine the last clean state before the failure:

```bash
# Check Supabase PITR timeline in dashboard → Database → Backups → Point-in-Time Recovery
# Identify the last timestamp before first error spike (from Sentry/Cloudflare logs)
# Target: T_failure - 5 minutes (stay within 1-hour RPO)

RECOVERY_TIMESTAMP="2026-06-07T14:30:00Z"  # Example — set from your incident timeline
```

**RPO check:** If `now() - RECOVERY_TIMESTAMP > 1 hour`, the enterprise RPO is breached. Log this:

```
[ ] RPO breach: YES / NO
[ ] If YES: SLA credit applies. Document breach window: from ________ to ________.
[ ] If YES: Notify compliance-officer immediately.
```

### Step 3.2 — Initiate PITR restore

1. In Supabase dashboard → Database → Backups → Point-in-Time Recovery.
2. Select the recovery timestamp from Step 3.1.
3. Confirm restore. Estimated restore duration: 30–90 minutes depending on database size.
4. **Do not interrupt** the restore once started.
5. Log restore initiation:

```sql
INSERT INTO audit_events (event_type, severity, actor_type, metadata)
VALUES ('system.pitr_restore_initiated', 'HIGH', 'system',
  '{"recovery_timestamp": "[ISO8601]", "initiated_by": "[engineer_user_id]",
    "rpo_breach": false, "incident_ref": "FORM-DR-003-[DATE]"}');
```

### Step 3.3 — Monitor restore progress

- Check Supabase dashboard → Restore Progress every 10 minutes.
- Estimated time to completion: ≈ 30–90 minutes.
- Do not attempt any writes to Supabase while restore is in progress.
- Update `form.coach/status` page: "Database restore in progress. ETA: [time]".

---

## §4 — HMAC Audit Chain Integrity (Post-Restore)

**DEC-030 requirement:** Before lifting maintenance mode, verify the HMAC audit chain has not been broken by the restore.

### Step 4.1 — Verify chain integrity

```sql
-- Run in Supabase SQL Editor post-restore
WITH chain_check AS (
  SELECT
    id,
    created_at,
    hmac_self,
    hmac_prev,
    LAG(hmac_self) OVER (ORDER BY created_at ASC) AS expected_prev
  FROM audit_events
  ORDER BY created_at DESC
  LIMIT 1000
)
SELECT
  COUNT(*) FILTER (WHERE hmac_prev != expected_prev) AS broken_links,
  MIN(created_at) FILTER (WHERE hmac_prev != expected_prev) AS first_break_at
FROM chain_check;
```

**Expected result:** `broken_links = 0`.

**If `broken_links > 0`:**

```
[ ] THIS IS A P0 SECURITY INCIDENT. Do NOT lift maintenance mode.
[ ] Immediately notify compliance-officer and security-engineer.
[ ] Invoke docs/INCIDENT_RESPONSE.md §5.5 (Security Breach protocol).
[ ] Emit: privacy.floor_breach_detected with breach_source='audit_log_integrity'
[ ] Do NOT restore the audit log from backup without compliance-officer approval.
[ ] The audit log break is forensic evidence — preserve it.
```

### Step 4.2 — Log restore completion

```sql
INSERT INTO audit_events (event_type, severity, actor_type, metadata)
VALUES ('system.pitr_restore_completed', 'HIGH', 'system',
  '{"recovery_timestamp": "[ISO8601]", "chain_integrity_verified": true,
    "broken_links_found": 0, "incident_ref": "FORM-DR-003-[DATE]"}');
```

---

## §5 — Post-Recovery Validation

Execute before lifting maintenance mode. Estimated time: **30 minutes**.

### Step 5.1 — Privacy Floor verification

**This step is non-negotiable.** The Privacy Floor must hold in all restored states.

```sql
-- Test 1: HR admin query must return only aggregates, never individual rows
EXPLAIN ANALYZE
SELECT * FROM v_team_wellness_aggregate WHERE tenant_id = '[TEST_TENANT_UUID]';
-- Expected: view returns aggregate stats only. If raw user rows are returned → P0 privacy breach.

-- Test 2: RLS prevents cross-tenant read
SET LOCAL role = 'form_member_role';
SET LOCAL request.tenant_id = '[TENANT_A_UUID]';
SELECT COUNT(*) FROM user_profiles WHERE tenant_id = '[TENANT_B_UUID]';
-- Expected: 0 rows (RLS blocks cross-tenant). If > 0 → P0 security incident.

-- Test 3: Aggregate queries do not expose individual identifiers
SELECT user_id, workout_count FROM v_team_wellness_aggregate;
-- Expected: column does not exist in the view. If it does → Privacy Floor violation.
```

**If any test fails:**

```
[ ] Do NOT lift maintenance mode.
[ ] Emit: privacy.floor_breach_detected (CRITICAL, DEC-030 HMAC-chained)
[ ] Notify compliance-officer and clinical-safety immediately.
[ ] Invoke docs/INCIDENT_RESPONSE.md §R-22.
```

### Step 5.2 — Core smoke test (F1–F8 flows)

```bash
# Run smoke test suite against production (with maintenance mode still active)
# These are internal API checks, not user-facing flows

# F1: Health check
curl -sf https://api.form.coach/api/health | jq '.status' # Expected: "ok"

# F2: Auth — token refresh
curl -sf -H "Authorization: Bearer $TEST_REFRESH_TOKEN" \
  https://api.form.coach/api/auth/refresh | jq '.access_token' # Expected: non-null

# F3: Workout read (test tenant)
curl -sf -H "Authorization: Bearer $TEST_ACCESS_TOKEN" \
  https://api.form.coach/api/workouts?limit=1 | jq '.data | length' # Expected: >= 0

# F4: Workout write
curl -sf -X POST -H "Authorization: Bearer $TEST_ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"type":"strength","duration_sec":1}' \
  https://api.form.coach/api/workouts | jq '.id' # Expected: UUID

# F5: Enterprise tenant SSO health
curl -sf "https://api.form.coach/api/sso/health?tenant_id=$TEST_ENTERPRISE_TENANT" \
  | jq '.saml_configured' # Expected: true

# F6: Admin aggregate endpoint (enterprise)
curl -sf -H "Authorization: Bearer $TEST_ADMIN_TOKEN" \
  "https://api.form.coach/api/admin/wellness-aggregate?tenant_id=$TEST_ENTERPRISE_TENANT" \
  | jq 'has("user_id")' # Expected: false (Privacy Floor)

# F7: Audit log write
curl -sf -X POST -H "Authorization: Bearer $TEST_ACCESS_TOKEN" \
  https://api.form.coach/api/debug/audit-ping | jq '.hmac_valid' # Expected: true

# F8: SCIM health (enterprise)
curl -sf -H "Authorization: Bearer $TEST_SCIM_TOKEN" \
  "https://api.form.coach/scim/v2/ServiceProviderConfig" | jq '.schemas' # Expected: non-empty
```

Log any failures before proceeding. All F1–F8 must pass before lifting maintenance mode.

### Step 5.3 — Row count sanity check

```sql
-- Verify no unexpected data loss after restore
SELECT
  schemaname,
  tablename,
  n_live_tup AS row_count
FROM pg_stat_user_tables
WHERE schemaname = 'public'
ORDER BY n_live_tup DESC;
```

Compare against pre-incident baseline (stored in `compliance/evidence/cc7/baseline-row-counts.csv`). A drop > 5% in any key table requires investigation before proceeding.

---

## §6 — Lift Maintenance Mode

Only after ALL checks in §5 pass.

```bash
# Remove maintenance flag
wrangler kv key delete --binding=SYSTEM_KV "system:maintenance" --env production

# Verify API is live
curl -sf https://api.form.coach/api/health | jq '.status' # Expected: "ok"
```

Update `form.coach/status` page: **All Systems Operational**.

Send recovery notification to enterprise tenants (template `docs/INCIDENT_RESPONSE.md §8 E-04`):

```
Subject: FORM Service Restored — [DATE TIME UTC]

FORM services have been fully restored as of [TIME UTC].
Duration: [X hours Y minutes].
Data impact: None / [describe if RPO was breached].
Root cause: [Supabase platform incident / PITR restore — final RCA to follow in 5 business days].
SLA credit: [Automatic / None applicable].
```

Emit recovery audit event:

```sql
INSERT INTO audit_events (event_type, severity, actor_type, metadata)
VALUES ('system.maintenance_lifted', 'HIGH', 'system',
  '{"incident_ref": "FORM-DR-003-[DATE]", "rto_minutes": [X],
    "rpo_breach": false, "smoke_tests_passed": 8,
    "privacy_floor_verified": true, "lifted_by": "[engineer_user_id]"}');
```

---

## §7 — Evidence Filing & Post-Mortem

### Evidence to file

```
[ ] Screenshot: Supabase Status Page (captured during incident).
[ ] Screenshot: PagerDuty alert timeline.
[ ] Export: Running notes document (timestamped actions log).
[ ] Export: Sentry error spike chart (error rate vs time).
[ ] Export: Cloudflare Analytics 5xx rate chart.
[ ] PITR restore log (if initiated): Supabase restore confirmation page.
[ ] Audit log export: all events emitted during incident window.
[ ] Smoke test results (F1–F8 pass/fail log).
[ ] Privacy Floor verification SQL output.
```

File to: `compliance/evidence/cc7/incidents/FORM-DR-003-[DATE]/`

### Post-mortem

Write within **5 business days** using `docs/INCIDENT_RESPONSE.md §8` template. Required sections:
- Timeline (T+0 through maintenance lift, 5-minute resolution)
- Root cause (confirmed or suspected)
- Customer impact (enterprise tenants affected, duration, data gap if any)
- SLA credit calculation (if applicable per `docs/ENTERPRISE_SLA.md §5`)
- Action items (preventive measures, runbook updates)

### DR drill evidence (if this was a planned drill)

If this execution was a scheduled DR drill (not a real incident), additionally file:

```
[ ] Drill report to: enterprise/runbooks/test-reports/FORM-DR-003-drill-[DATE].md
[ ] Update §7 header: "Last tested: [DATE] — [Engineer name]"
[ ] Update compliance/cc9/README.md: CC9-GAP-005 status if drill is complete.
```

---

## §8 — Quick Reference Card

| Condition | Action |
|---|---|
| Supabase platform incident | §2 maintenance mode → monitor → §5 post-recovery |
| FORM Worker bug (not Supabase) | Stop. Use `docs/INCIDENT_RESPONSE.md §5.2` |
| > 2 hours, not recovered | Call founder → §3 PITR initiation |
| > 4 hours (enterprise RTO breach) | §3 PITR regardless + SLA credit notification |
| PITR complete, chain broken | **P0 — call compliance-officer. Do not lift maintenance.** |
| Privacy Floor test fails | **P0 — call compliance-officer + clinical-safety. Do not lift maintenance.** |
| All F1–F8 pass + chain intact | §6 lift maintenance → recovery notification |

---

## §9 — Contact Escalation Tree

| Role | Contact | When |
|---|---|---|
| **Founder** | 1Password → "Emergency Contacts" | Before PITR initiation; after 2-hour platform incident |
| **compliance-officer** | 1Password → "Emergency Contacts" | HMAC chain break; Privacy Floor failure; GDPR 72h clock starts |
| **Supabase Enterprise Support** | dashboard.supabase.com → Support | After 2 hours of Supabase platform incident |
| **Customer CSM leads** | Linear → "Customer Contacts" | On enterprise tenant notification |

---

*v1.0 · 2026-06-07 · Owner: devops-lead + platform-engineer*
*Approved by: compliance-officer · Next review: after first drill execution or any architecture change*
*SOC 2 evidence: CC7.4, CC7.5, A1.2, A1.3, CC9.1 · HMAC audit chain: DEC-030*
