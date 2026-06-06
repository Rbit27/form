# FORM-DR-001 · Supabase Primary Region Outage

> **Runbook ID:** FORM-DR-001
> **Severity:** P0
> **Owner:** devops-lead · reviewed by: compliance-officer · enterprise-architect
> **Last reviewed:** 2026-06-06
> **SOC 2 TSC:** A1.1, A1.2, A1.3
> **Cross-refs:** docs/SOC2_READINESS.md §53, §53.4, §53.6.3; docs/DATA_MODEL.md §10; docs/INCIDENT_RESPONSE.md §IR-1; docs/OBSERVABILITY.md §OBS-1

---

## 1. Scenario Summary

| Field | Value |
|---|---|
| **ID** | FORM-DR-001 |
| **Severity** | P0 |
| **Description** | Supabase PostgreSQL primary region (us-east-1) is completely unavailable. All DB reads and writes fail. PITR is inaccessible from the primary region endpoint. |
| **Affected tiers** | Enterprise, Growth, Consumer — all authenticated functionality fails |
| **RTO (PITR path)** | 60–90 min (satisfies Enterprise ≤2h) |
| **RTO (cold backup path)** | 3–5h (breaches Enterprise RTO; trigger `system.sla_breach_recorded`) |

---

## 2. RTO/RPO Targets

| Tier | RTO | RPO | SLA Enforcement |
|---|---|---|---|
| Enterprise | ≤2h | ≤15min | Contractual; breach triggers MSA penalty clause |
| Growth | ≤4h | ≤1h | Operational SLA |
| Consumer | ≤8h | ≤4h | Best effort |

PITR granularity: 1-minute. Retention: 7 days (Supabase Pro).
Cold backup RPO: up to 24h (nightly pg_dump; AES-256-GCM encrypted in R2 `form-cold-backups/`).

---

## 3. Detection Signals

- **Sentry alert FORM-DR-DB-001** — Supabase health-check failing >2 consecutive checks
- **`https://status.supabase.com`** — incident banner present on DB or API component
- **API 503/500 responses** — all authenticated endpoints returning DB connection errors
- **PostHog** — sharp drop in `session_started` events (leading indicator)
- **Better Stack / Grafana Cloud** — `supabase_db_reachable` metric = 0 for >2min

Escalate to founder within 15min of P0 declaration (IC: devops-lead; backup IC: security-engineer).

---

## 4. Pre-Flight Checklist

- [ ] Confirm outage is Supabase-side: check `https://status.supabase.com`
- [ ] Confirm it is not a FORM misconfiguration: verify `DATABASE_URL` Workers Secret is unchanged (`wrangler secret list --env production`)
- [ ] Confirm Sentry alert FORM-DR-DB-001 is active (not a false-positive test)
- [ ] Open Linear incident ticket — P0, tag `db-outage`, link to Supabase status page
- [ ] Page backup IC (security-engineer) if devops-lead unavailable
- [ ] Confirm founder is reachable (≤15min from P0 declaration)
- [ ] Confirm compliance-officer is reachable (for SLA breach assessment)
- [ ] Identify current PITR restore window: last known good timestamp before outage

---

## 5. Step-by-Step Recovery Procedure

### Path A — PITR Restore (Primary; target RTO 60–90min)

Use if Supabase primary region outage exceeds 60min with no ETA, or immediately if PITR is accessible from another region.

**1. Check Supabase status and set timer**

```bash
# Every 15min during outage — log each check in the Linear ticket
curl -s https://status.supabase.com/api/v2/summary.json | jq '.components[] | select(.name | test("Database|API"))'
```

Responsible: devops-lead | Time constraint: continuous until decision at T+60min

**2. Create new Supabase project in secondary region**

- Region: `eu-west-1` (or nearest non-us-east-1 region)
- Plan: Pro (required for PITR)
- Name: `form-production-dr-YYYYMMDD`

Responsible: devops-lead | Time constraint: complete by T+65min

**3. Initiate PITR restore**

```bash
# Via Supabase dashboard: Project Settings → Backups → Point in Time Recovery
# Or via Supabase Management API:
curl -X POST "https://api.supabase.com/v1/projects/${DR_PROJECT_REF}/database/backups/restore" \
  -H "Authorization: Bearer ${SUPABASE_ACCESS_TOKEN}" \
  -H "Content-Type: application/json" \
  -d '{"recovery_time_target_unix": <LAST_GOOD_UNIX_TS>}'
```

Target `recovery_time_target_unix`: last clean timestamp before incident (from Sentry breadcrumbs or PostHog event stream). Responsible: devops-lead | Time constraint: initiated by T+70min

**4. Update `DATABASE_URL` Workers Secret**

```bash
# After new project is healthy and accepting connections
wrangler secret put DATABASE_URL --env production
# Paste new Supabase connection string when prompted
```

Responsible: devops-lead | Time constraint: T+80min

**5. Redeploy Cloudflare Workers**

```bash
wrangler deploy --env production
```

Responsible: devops-lead | Time constraint: T+85min

**6. Verify RLS enforcement**

```bash
supabase test db --filter=rls --db-url "${NEW_DATABASE_URL}"
```

All RLS tests must pass before declaring recovery. Responsible: devops-lead + platform-engineer | Time constraint: T+90min

**7. Verify HMAC audit-log chain continuity**

```sql
-- Run against new DB; confirm no gap in sequence numbers
SELECT
  id,
  occurred_at,
  hmac_sha256(prev_hash || event_type || actor_uuid || occurred_at::text, current_setting('app.audit_hmac_key')) AS expected_hash,
  chain_hash
FROM audit_log_events
ORDER BY id DESC
LIMIT 20;
```

Any mismatch must be reported to compliance-officer immediately. Responsible: devops-lead | Time constraint: T+90min

**8. Smoke-test critical user paths**

- Authenticated login (Supabase Auth)
- Workout log write
- CV session thumbnail read
- Coaching turn insert

Responsible: platform-engineer | Time constraint: T+95min

---

### Path B — Cold Backup Restore (Fallback; target RTO 3–5h)

Use only if PITR is unavailable. This path likely breaches Enterprise RTO.

**1. Emit `system.sla_breach_recorded`**

```sql
INSERT INTO audit_log_events (event_type, severity, actor_type, payload, occurred_at)
VALUES (
  'system.sla_breach_recorded',
  'HIGH',
  'system',
  jsonb_build_object(
    'breach_type', ARRAY['enterprise_rto'],
    'runbook', 'FORM-DR-001',
    'path', 'cold_backup',
    'expected_rto_min', 120,
    'projected_rto_min', 300,
    'incident_ref', '<LINEAR_TICKET_ID>'
  ),
  now()
);
```

**2. Download latest cold backup from R2**

```bash
# List available backups
wrangler r2 object get form-cold-backups/manifests/$(date +%Y/%m)/backup-manifest-$(date +%Y-%m-%d).json \
  --file manifest.json

# Download the backup file listed in the manifest
BACKUP_FILE=$(jq -r '.backup_file' manifest.json)
wrangler r2 object get "form-cold-backups/${BACKUP_FILE}" --file backup.sql.enc
```

**3. Decrypt backup**

```bash
# COLD_BACKUP_ENC_KEY is in Workers Secret — retrieve via wrangler or 1Password
openssl enc -d -aes-256-gcm -in backup.sql.enc -out backup.sql \
  -pass env:COLD_BACKUP_ENC_KEY -pbkdf2
```

**4. Restore to ephemeral Postgres**

Provision a Cloudflare Container or temporary Supabase project, then:

```bash
psql "${EPHEMERAL_DB_URL}" < backup.sql
```

**5. Update `DATABASE_URL` and redeploy Workers** — same as Path A steps 4–5.

**6. Verify RLS and HMAC chain** — same as Path A steps 6–7.

---

## 6. Rollback / Abort Procedure

- If new Supabase project fails health checks after PITR restore: do not update `DATABASE_URL`. Re-evaluate cold backup path.
- If Workers redeploy causes errors: `wrangler rollback --env production` within 30 seconds.
- If RLS tests fail: halt all user traffic by setting `MAINTENANCE_MODE=true` in Cloudflare KV; escalate to platform-engineer immediately.
- Never discard the ephemeral DR project until primary is confirmed stable for ≥30min.

---

## 7. Evidence Capture Instructions

Capture the following and upload to R2 `form-cold-backups/incidents/<LINEAR_TICKET_ID>/`:

- [ ] Terminal output: full PITR restore command and new cluster endpoint
- [ ] Screenshot: Supabase dashboard showing new project status = "Healthy"
- [ ] Terminal output: `supabase test db --filter=rls` — all tests passing
- [ ] Terminal output: HMAC chain verification query output (no mismatches)
- [ ] Screenshot: `https://status.supabase.com` at time of detection and at time of recovery
- [ ] DEC-030 event: emit `system.dr_drill_completed` if this execution was a drill

```sql
-- For live drills only
INSERT INTO audit_log_events (event_type, severity, actor_type, payload, occurred_at)
VALUES (
  'system.dr_drill_completed',
  'INFO',
  'system',
  jsonb_build_object(
    'runbook', 'FORM-DR-001',
    'drill_started_at', '<ISO8601>',
    'drill_completed_at', now()::text,
    'path_tested', 'pitr',
    'rto_achieved_min', <ACTUAL_MINUTES>
  ),
  now()
);
```

All audit log payloads must use `actor_uuid` (not name), `client_ip_hash` (SHA-256 with `IP_HASH_SALT`), and must not contain plaintext health data.

---

## 8. Post-Incident Actions

### Customer Communication

| Tier | Channel | Timing | Frequency |
|---|---|---|---|
| Enterprise | Phone + email | Within 10min of P0 declaration | Every 30min until resolved |
| Growth | Email | Within 15min of P0 declaration | Every 60min until resolved |
| Consumer | Status page | Within 30min of P0 declaration | On change |

Post-incident report for Enterprise: within 48h of resolution (root cause, timeline, remediation steps).

### SLA Breach Assessment

- If Path B was used: compliance-officer assesses MSA penalty clause applicability for each Enterprise tenant affected.
- Log breach in `audit_log_events` with `system.sla_breach_recorded` (see Path B step 1).

### Post-Mortem

- Trigger: always for P0
- Timeline: within 48h of resolution
- Owner: devops-lead
- Include: MTTD, MTTR, RTO achieved vs target, RPO achieved vs target, action items with owners and due dates

---

## 9. SOC 2 TSC Cross-Reference

| TSC | How this runbook satisfies it |
|---|---|
| A1.1 | RTO/RPO targets defined per tier; PITR and cold backup paths documented |
| A1.2 | Recovery procedures tested via scheduled DR drills; evidence captured to R2 |
| A1.3 | Restoration procedures verified via RLS tests and HMAC chain continuity checks; post-incident review required |

---

*v1.0 · 2026-06-06 · devops-lead + compliance-officer*
