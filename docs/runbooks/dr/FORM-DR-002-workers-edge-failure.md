# FORM-DR-002 · Cloudflare Workers Global Edge Failure

> **Runbook ID:** FORM-DR-002
> **Severity:** P0
> **Owner:** devops-lead · reviewed by: compliance-officer · enterprise-architect
> **Last reviewed:** 2026-06-06
> **SOC 2 TSC:** A1.1, A1.2, A1.3
> **Cross-refs:** docs/SOC2_READINESS.md §53; docs/INCIDENT_RESPONSE.md §IR-2; docs/OBSERVABILITY.md §OBS-2

---

## 1. Scenario Summary

| Field | Value |
|---|---|
| ID | FORM-DR-002 |
| Severity | P0 |
| Description | Cloudflare Workers globally unavailable due to a Cloudflare platform incident. All FORM API requests fail. Supabase database is healthy but unreachable through the Workers proxy layer. |
| Affected tiers | Enterprise, Growth, Consumer — all API-dependent functionality fails |
| Estimated RTO — FORM deployment rollback | < 15 min (if FORM caused the failure) |
| Estimated RTO — Cloudflare platform incident | Vendor-dependent; no FORM recovery action possible |

**Critical constraint:** FORM has no control over Cloudflare platform recovery. If root cause is a Cloudflare platform incident, primary actions are to detect correctly, communicate correctly, and activate force-majeure provisions in the enterprise MSA. Do not consume IC time on recovery actions that cannot succeed.

---

## 2. RTO/RPO Targets

| Tier | RTO | RPO | SLA Enforcement |
|---|---|---|---|
| Enterprise | ≤ 2 h | ≤ 15 min | Contractual; force-majeure clause may apply if root cause is Cloudflare platform |
| Growth | ≤ 4 h | ≤ 1 h | Operational SLA |
| Consumer | ≤ 8 h | ≤ 4 h | Best effort |

If root cause is confirmed as Cloudflare platform: compliance-officer must assess force-majeure clause before any SLA credit is issued or denied.

---

## 3. Detection Signals

- **Sentry alert FORM-DR-EDGE-001** — 100% of Workers health-check probes failing across all regions
- **https://www.cloudflarestatus.com** — incident on Cloudflare Workers or core routing components
- **`wrangler tail --env production`** — zero invocations visible (no traffic reaching Workers)
- **Better Stack / Grafana Cloud** — `workers_health_probe_success_rate` = 0% for > 2 min
- **PostHog** — complete drop in all event ingestion
- **Cloudflare Analytics** — requests graph shows cliff-edge drop to zero

Escalate to founder within 15 min of P0 declaration. IC: devops-lead; backup IC: security-engineer.

---

## 4. Pre-Flight Checklist

- [ ] Confirm 100% failure (not partial): check `wrangler deployments list --env production` for current deployment hash
- [ ] Check https://www.cloudflarestatus.com for a Workers or platform-wide incident
- [ ] Rule out FORM deployment as root cause: compare last deploy timestamp against first error timestamp in Sentry
- [ ] Rule out DNS misconfiguration: `dig api.formcoach.app` — confirm CNAME points to correct `*.workers.dev` hostname
- [ ] Open Linear incident ticket — P0, tag `edge-outage`, link to cloudflarestatus.com incident URL with exact start timestamp
- [ ] Confirm backup IC (security-engineer) is reachable
- [ ] Confirm founder is reachable within 15 min of P0 declaration
- [ ] Confirm compliance-officer is reachable (force-majeure + SLA assessment)

---

## 5. Step-by-Step Recovery Procedure

### Step 1 — Confirm root cause: FORM deployment vs. Cloudflare platform (devops-lead; T+0 to T+10 min)

```bash
# Inspect current deployment and compare hash to last known good
wrangler deployments list --env production

# Attempt to tail logs — any output means Workers are partially reachable
wrangler tail --env production --format pretty

# Check Cloudflare component status
curl -s https://www.cloudflarestatus.com/api/v2/summary.json \
  | jq '.components[] | select(.name | test("Workers|Routing|DNS"))'
```

**Decision gate:**
- Last deploy timestamp matches first error timestamp → FORM deployment issue → proceed to Step 2.
- `cloudflarestatus.com` shows Workers incident, deploy timestamp does not match error start → Cloudflare platform issue → skip to Step 3.
- Errors are isolated to one region → potential partial failure → skip to Step 4.

---

### Step 2 — FORM deployment rollback (if FORM caused the failure; devops-lead; T+10–25 min)

```bash
# Identify last known good deployment hash
wrangler deployments list --env production

# Roll back to previous deployment
wrangler rollback --env production

# Confirm recovery
curl -sf https://api.formcoach.app/health | jq .
wrangler tail --env production --format pretty
```

Target: full recovery in < 15 min from diagnosis. If rollback does not restore service, re-evaluate as Cloudflare platform issue.

---

### Step 3 — Cloudflare platform incident: no direct recovery action (devops-lead + compliance-officer; ongoing)

This is a vendor outage. Do not attempt actions that cannot succeed.

Actions (execute in parallel):

1. Record Cloudflare incident URL and start timestamp in Linear ticket
2. Begin enterprise customer communication tree (see §8) — notify within 10 min of P0 declaration
3. Set 15-min status-check cadence on https://www.cloudflarestatus.com; log each check in Linear
4. Document outage start timestamp for SLA breach duration calculation
5. Notify Growth customers via email within 15 min; post Consumer status page update within 30 min

---

### Step 4 — Partial regional failure: attempt alternate-zone routing (devops-lead; T+20–65 min)

Use only if cloudflarestatus.com shows a regional (not global) Workers failure and an `alt-zone` Worker environment is configured.

```bash
# Deploy to alternate zone
wrangler deploy --env alt-zone

# Update DNS record to point api.formcoach.app at alt-zone worker
curl -X PATCH "https://api.cloudflare.com/client/v4/zones/${CF_ZONE_ID}/dns_records/${DNS_RECORD_ID}" \
  -H "Authorization: Bearer ${CF_API_TOKEN}" \
  -H "Content-Type: application/json" \
  -d '{"content": "<alt-zone-worker-hostname>.workers.dev"}'

# Verify
curl -sf https://api.formcoach.app/health | jq .
```

Time budget: 45 min total. If not resolved by T+65 min, abandon alt-zone attempt and treat as global outage (Step 3).

---

### Step 5 — Background job queue management (platform-engineer; within 30 min of P0)

- Cloudflare Queues: messages retained up to 4 days by default — no action required.
- Stripe webhooks: Stripe retries failed deliveries for 72 h automatically — no action required.
- Supabase async triggers: remain queued in DB — no action required.
- Log estimated queue depth in Linear ticket for post-recovery flush verification.

---

### Step 6 — Recovery confirmation (devops-lead; within 15 min of Cloudflare declaring resolved)

```bash
# Confirm Workers responding
curl -sf https://api.formcoach.app/health | jq .

# Confirm invocations resuming
wrangler tail --env production --format pretty

# Verify queue consumers are processing
wrangler queues consumer list <queue-name> --env production
```

If outage exceeded Enterprise RTO (2 h), emit `system.sla_breach_recorded`:

```sql
INSERT INTO audit_log_events (event_type, severity, actor_type, payload, occurred_at)
VALUES (
  'system.sla_breach_recorded',
  'HIGH',
  'system',
  jsonb_build_object(
    'breach_type',             ARRAY['enterprise_rto'],
    'runbook',                 'FORM-DR-002',
    'root_cause',              'cloudflare_platform_incident',
    'cloudflare_incident_url', '<CF_INCIDENT_URL>',
    'outage_duration_min',     '<ACTUAL_MINUTES>',
    'force_majeure_claimed',   true,
    'incident_ref',            '<LINEAR_TICKET_ID>'
  ),
  now()
);
```

---

## 6. Rollback / Abort Procedure

- If FORM rollback (Step 2) does not restore service within 15 min: stop FORM-side actions; confirm Cloudflare platform involvement; shift entirely to communication mode (Step 3).
- If alt-zone routing (Step 4) causes DB connection errors (wrong `DATABASE_URL` in alt-zone environment): revert DNS record immediately via Cloudflare Dashboard; check alt-zone `DATABASE_URL` Workers Secret.
- Do not issue repeated `wrangler deploy` cycles during a confirmed Cloudflare platform incident — it will not help and consumes deploy budget.

---

## 7. Evidence Capture Instructions

Upload all evidence to R2 `form-cold-backups/incidents/<LINEAR_TICKET_ID>/`:

- [ ] Screenshot: https://www.cloudflarestatus.com showing incident, URL, and start/end timestamps
- [ ] Terminal output: `wrangler deployments list` at time of detection
- [ ] Sentry FORM-DR-EDGE-001 alert export showing zero invocations during outage window
- [ ] Terminal output: `wrangler rollback` result (if Step 2 was executed)
- [ ] Terminal output: `curl -sf https://api.formcoach.app/health` at time of recovery confirmation
- [ ] DEC-030 `system.sla_breach_recorded` SQL output (if RTO was breached)
- [ ] DEC-030 `system.dr_drill_completed` event if this execution was a drill:

```sql
INSERT INTO audit_log_events (event_type, severity, actor_type, payload, occurred_at)
VALUES (
  'system.dr_drill_completed',
  'INFO',
  'system',
  jsonb_build_object(
    'runbook',            'FORM-DR-002',
    'drill_started_at',   '<ISO8601>',
    'drill_completed_at', now()::text,
    'scenario_tested',    'cloudflare_platform_incident'
  ),
  now()
);
```

Privacy floor: all payloads must use `actor_uuid`, `client_ip_hash` (SHA-256 with `IP_HASH_SALT`). No plaintext health data, names, or raw IPs.

---

## 8. Post-Incident Actions

### Customer Communication

| Tier | Channel | Timing | Frequency |
|---|---|---|---|
| Enterprise | Phone + email | Within 10 min of P0 declaration | Every 30 min until resolved |
| Growth | Email | Within 15 min of P0 declaration | Every 60 min until resolved |
| Consumer | Status page (statuspage.io) | Within 30 min of P0 declaration | On change |

Email template note: include Cloudflare incident URL, state that recovery is vendor-dependent, confirm Supabase data is safe and unaffected.

### Force-Majeure Assessment

- If outage > 2 h and root cause is confirmed Cloudflare platform: compliance-officer documents Cloudflare incident URL, duration, and per-tenant impact in Linear ticket.
- Legal review required before any SLA credit denial based on force-majeure.
- Outcome must be documented in post-mortem.

### Post-Mortem

- Trigger: always for P0, regardless of root cause
- Timeline: within 48 h of resolution
- Owner: devops-lead
- Template: docs/INCIDENT_RESPONSE.md §IR-2
- Required: MTTD, MTTR, whether FORM-side rollback was possible, force-majeure assessment outcome, actions to improve alt-zone failover automation

---

## 9. SOC 2 TSC Cross-Reference

| TSC | How this runbook satisfies it |
|---|---|
| A1.1 | RTO targets defined per tier; vendor dependency and force-majeure constraints explicitly documented |
| A1.2 | Detection signals defined; communication tree activated within prescribed windows; DR drills include this scenario |
| A1.3 | Recovery verification via health endpoint; queue replay verified post-recovery; `system.sla_breach_recorded` emitted if thresholds breached |

---

*v1.0 · 2026-06-06 · devops-lead + compliance-officer*
