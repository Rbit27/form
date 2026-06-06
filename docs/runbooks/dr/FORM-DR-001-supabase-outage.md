# FORM-DR-001 · Supabase Primary Region Total Outage

> **Runbook ID:** FORM-DR-001
> **Severity:** P0
> **Owner:** devops-lead · reviewed by: compliance-officer · enterprise-architect
> **Last reviewed:** 2026-06-06
> **SOC 2 TSC:** A1.1, A1.2, A1.3
> **Cross-refs:** docs/SOC2_READINESS.md §53; docs/INCIDENT_RESPONSE.md §IR-1; docs/OBSERVABILITY.md §OBS-3; docs/DATA_MODEL.md §10

---

## 1. Scenario Summary

| Field | Value |
|---|---|
| ID | FORM-DR-001 |
| Severity | P0 |
| Description | Supabase PostgreSQL primary region (us-east-1) total outage. All DB reads and writes fail. PITR inaccessible from primary region endpoint. |
| Affected tiers | Enterprise, Growth, Consumer — all authenticated functionality fails |
| Estimated RTO — PITR path | 60–90 min (satisfies Enterprise ≤2 h) |
| Estimated RTO — Cold backup path | 3–5 h (breaches Enterprise RTO; trigger `system.sla_breach_recorded`) |

---

## 2. RTO/RPO Targets

| Tier | RTO | RPO | SLA Enforcement |
|---|---|---|---|
| Enterprise | ≤ 2 h | ≤ 15 min | Contractual; breach triggers MSA penalty clause |
| Growth | ≤ 4 h | ≤ 1 h | Operational SLA |
| Consumer | ≤ 8 h | ≤ 4 h | Best effort |

PITR granularity: 1-minute. PITR retention: 7 days (Supabase Pro). Cold backup RPO: up to 24 h (nightly pg_dump). Cold backup retention: 90 days in R2 `form-cold-backups/`.

---

## 3. Detection Signals

- **Sentry alert FORM-DR-DB-001** — Supabase health-check failing > 2 consecutive checks; pages devops-lead immediately
- **https://status.supabase.com** — incident declared on DB or API component
- **All authenticated API calls returning 503 / 500** with `DB_CONNECTION_ERROR` in response body
- **PostHog** — sharp drop in `session_started` events (leading indicator)
- **Better Stack / Grafana Cloud** — `supabase_db_reachable` metric = 0 for > 2 min

Escalate to founder within 15 min of P0 declaration. IC: devops-lead; backup IC: security-engineer.

---

## 4. Pre-Flight Checklist

- [ ] Confirm outage is Supabase-side: check https://status.supabase.com
- [ ] Confirm this is not a FORM misconfiguration: verify `DATABASE_URL` Workers Secret is unchanged via `wrangler secret list --env production`
- [ ] Confirm Sentry alert FORM-DR-DB-001 is active (not a false-positive drill)
- [ ] Open Linear incident ticket — P0, tag `db-outage`, link to Supabase status page
- [ ] Page backup IC (security-engineer) if devops-lead is unavailable
- [ ] Confirm founder is reachable within 15 min of P0 declaration
- [ ] Confirm compliance-officer is reachable (for SLA breach assessment)
- [ ] Identify PITR restore target timestamp: last clean DB write before first error (from Sentry breadcrumbs or PostHog event stream)

---

## 5. Step-by-Step Recovery Procedure

### Path A — PITR Restore (Primary; target RTO 60–90 min)

Use if Supabase primary region outage exceeds 60 min with no ETA, or immediately if PITR is accessible from another region.

**1. Monitor Supabase status and set decision timer** (devops-lead; T+0, repeat every 15 min)

```bash
curl -s https://status.supabase.com/api/v2/summary.json \
  | jq '.components[] | select(.name | test("Database|API"))'
```

Log each check result in the Linear ticket. Decision gate at T+60 min: if no ETA from Supabase, initiate PITR restore.

**2. Create new Supabase project in secondary region** (devops-lead; T+60 min)

- Via Supabase dashboard: New Project → Region: `eu-west-1` (or nearest non-us-east-1)
- Plan: Pro (required for PITR)
- Name: `form-production-dr-YYYYMMDD`
- Record new project `ref` in Linear ticket

**3. Initiate PITR restore** (devops-lead; T+65 min)

```bash
# Via Supabase Management API
curl -X POST "https://api.supabase.com/v1/projects/${DR_PROJECT_REF}/database/backups/restore" \
  -H "Authorization: Bearer ${SUPABASE_ACCESS_TOKEN}" \
  -H "Content-Type: application/json" \
  -d "{\"recovery_time_target_unix\": ${LAST_GOOD_UNIX_TS}}"
```

See `docs/DATA_MODEL.md §10` for full restore command reference. Target timestamp: 5 min before first detected error to guarantee RPO margin.

**4. Verify RLS enforcement on restored cluster** (devops-lead + platform-engineer; T+80 min)

```bash
supabase test db --filter=rls --db-url "${NEW_DATABASE_URL}"
```

All RLS tests must pass before routing any traffic. If any test fails: halt, do not update `DATABASE_URL`, escalate to platform-engineer.

**5. Verify HMAC audit-log chain continuity** (devops-lead; T+82 min)

```sql
-- Run against the new restored cluster
SELECT
  id,
  occurred_at,
  hmac_sha256(prev_hash || event_type || actor_uuid || occurred_at::text,
              current_setting('app.audit_hmac_key')) AS expected_hash,
  chain_hash
FROM audit_log_events
ORDER BY id DESC
LIMIT 20;
-- All expected_hash values must match chain_hash; any mismatch must be reported to compliance-officer immediately
```

**6. Update DATABASE_URL Workers Secret** (devops-lead; T+85 min)

```bash
wrangler secret put DATABASE_URL --env production
# Paste new Supabase connection string when prompted
```

**7. Redeploy Cloudflare Workers** (devops-lead; T+87 min)

```bash
wrangler deploy --env production
```

**8. Smoke-test critical user paths** (platform-engineer; T+90 min)

```bash
# Health endpoint
curl -sf https://api.formcoach.app/health | jq .

# Authenticated endpoint — expect non-empty array
curl -sf -H "Authorization: Bearer ${TEST_TOKEN}" \
  https://api.formcoach.app/api/v1/sessions/recent | jq 'length'
```

Paths to verify: authenticated login, workout log write, CV session thumbnail read, coaching turn insert.

---

### Path B — Cold Backup Restore (Fallback; target RTO 3–5 h)

Use only if PITR is unavailable. Enterprise RTO will be breached — emit `system.sla_breach_recorded` immediately.

**1. Emit SLA breach event** (devops-lead; at decision point to use Path B)

```sql
INSERT INTO audit_log_events (event_type, severity, actor_type, payload, occurred_at)
VALUES (
  'system.sla_breach_recorded',
  'HIGH',
  'system',
  jsonb_build_object(
    'breach_type',       ARRAY['enterprise_rto'],
    'runbook',           'FORM-DR-001',
    'path',              'cold_backup',
    'expected_rto_min',  120,
    'projected_rto_min', 300,
    'reason',            'PITR unavailable; falling back to cold backup restore',
    'incident_ref',      '<LINEAR_TICKET_ID>'
  ),
  now()
);
```

**2. Retrieve cold backup manifest from R2** (devops-lead)

```bash
wrangler r2 object get \
  "form-cold-backups/manifests/$(date +%Y/%m)/backup-manifest-$(date +%Y-%m-%d).json" \
  --file /tmp/manifest.json

cat /tmp/manifest.json
# Fields: backup_date, backup_file, sha256_checksum, file_size_bytes
```

**3. Download and decrypt cold backup** (devops-lead)

```bash
BACKUP_FILE=$(jq -r '.backup_file' /tmp/manifest.json)
wrangler r2 object get "form-cold-backups/${BACKUP_FILE}" --file /tmp/backup.sql.enc

# COLD_BACKUP_ENC_KEY retrieved from Workers Secret (pre-exported to secure session env)
openssl enc -d -aes-256-gcm -in /tmp/backup.sql.enc -out /tmp/backup.sql \
  -pass env:COLD_BACKUP_ENC_KEY -pbkdf2

# Verify checksum before restore
EXPECTED=$(jq -r '.sha256_checksum' /tmp/manifest.json)
ACTUAL=$(sha256sum /tmp/backup.sql | awk '{print $1}')
[ "$EXPECTED" = "$ACTUAL" ] && echo "Checksum OK" || { echo "CHECKSUM MISMATCH — HALT"; exit 1; }
```

**4. Restore to ephemeral Postgres** (devops-lead)

```bash
# Provision a temporary Supabase project or Cloudflare Container
psql "${EPHEMERAL_DB_URL}" < /tmp/backup.sql
```

**5. Continue from Path A steps 4–8** — run RLS verification, HMAC chain check, update Workers Secret, redeploy, smoke-test.

---

## 6. Rollback / Abort Procedure

- If new Supabase project fails health checks after PITR restore: do not update `DATABASE_URL`; reassess cold backup path.
- If Workers redeploy after secret update causes cascading errors: run `wrangler rollback --env production` within 30 s; revert `DATABASE_URL` to original project ref for read-only partial service.
- If RLS tests fail: set `MAINTENANCE_MODE=true` in Cloudflare KV to halt all user traffic; escalate to platform-engineer immediately.
- Never destroy the DR Supabase project until primary is confirmed stable for ≥ 30 min.

---

## 7. Evidence Capture Instructions

Upload all evidence to R2 `form-cold-backups/incidents/<LINEAR_TICKET_ID>/`:

- [ ] Terminal output: full PITR restore API call and response showing new cluster endpoint
- [ ] Screenshot: Supabase dashboard showing restored project status = Healthy
- [ ] Terminal output: `supabase test db --filter=rls` — all tests passing
- [ ] Terminal output: HMAC chain verification SQL query — no mismatches
- [ ] Terminal output: `curl /health` confirming recovery
- [ ] Screenshot: https://status.supabase.com at time of detection and at time of recovery
- [ ] DEC-030 `system.sla_breach_recorded` SQL output (if Path B was used)
- [ ] DEC-030 `system.dr_drill_completed` event if this execution was a drill:

```sql
INSERT INTO audit_log_events (event_type, severity, actor_type, payload, occurred_at)
VALUES (
  'system.dr_drill_completed',
  'INFO',
  'system',
  jsonb_build_object(
    'runbook',             'FORM-DR-001',
    'drill_started_at',    '<ISO8601>',
    'drill_completed_at',  now()::text,
    'path_tested',         'pitr',
    'rto_achieved_min',    '<ACTUAL_MINUTES>'
  ),
  now()
);
```

Privacy floor: all payloads must use `actor_uuid` (not names), `client_ip_hash` (SHA-256 with `IP_HASH_SALT`). No plaintext health data, individual names, or raw IPs in any captured output or audit event.

---

## 8. Post-Incident Actions

### Customer Communication

| Tier | Channel | Timing | Frequency |
|---|---|---|---|
| Enterprise | Phone + email | Within 10 min of P0 declaration | Every 30 min until resolved |
| Growth | Email | Within 15 min of P0 declaration | Every 60 min until resolved |
| Consumer | Status page (statuspage.io) | Within 30 min of P0 declaration | On change |

Post-incident report for Enterprise: due within 48 h of resolution. Include root cause, timeline, RPO/RTO achieved vs. target, and remediation steps.

### SLA Breach Assessment

If Path B was used: compliance-officer reviews MSA penalty clause applicability for each Enterprise tenant. Breach event must be emitted as shown in Path B step 1. Document in Linear ticket.

### Post-Mortem

- Trigger: always for P0
- Timeline: within 48 h of resolution
- Owner: devops-lead
- Template: docs/INCIDENT_RESPONSE.md §IR-1
- Required: MTTD, MTTR, RTO/RPO achieved vs. target, action items with owners and due dates

---

## 9. SOC 2 TSC Cross-Reference

| TSC | How this runbook satisfies it |
|---|---|
| A1.1 | RTO/RPO commitments defined per tier; recovery capacity (PITR + cold backup) documented and scoped |
| A1.2 | Two recovery paths tested via scheduled DR drills; evidence uploaded to R2; post-mortem required |
| A1.3 | RLS verification and HMAC chain continuity check required before traffic cutover; audit events emitted per DEC-030 |

---

*v1.0 · 2026-06-06 · devops-lead + compliance-officer*
