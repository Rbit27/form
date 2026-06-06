# FORM-DR-002 · Cloudflare Workers Global Edge Failure

> **Runbook ID:** FORM-DR-002
> **Severity:** P0
> **Owner:** devops-lead · reviewed by: compliance-officer · enterprise-architect
> **Last reviewed:** 2026-06-06
> **SOC 2 TSC:** A1.1, A1.2, A1.3
> **Cross-refs:** docs/SOC2_READINESS.md §53, §53.4; docs/INCIDENT_RESPONSE.md §IR-2; docs/OBSERVABILITY.md §OBS-2

---

## 1. Scenario Summary

| Field | Value |
|---|---|
| **ID** | FORM-DR-002 |
| **Severity** | P0 |
| **Description** | Cloudflare Workers globally unavailable due to a Cloudflare platform incident. All FORM API requests fail. Supabase database is healthy but unreachable through the Workers proxy layer. |
| **Affected tiers** | Enterprise, Growth, Consumer — all API-dependent functionality fails |
| **RTO (FORM deployment rollback)** | <15min (if FORM caused the failure) |
| **RTO (Cloudflare platform incident)** | Vendor-dependent; no FORM recovery action possible |

**Critical constraint:** FORM has no control over Cloudflare platform recovery. If the root cause is a Cloudflare platform incident, the primary actions are to detect correctly, communicate correctly, and activate force-majeure provisions. Do not spend IC time on recovery actions that cannot succeed.

---

## 2. RTO/RPO Targets

| Tier | RTO | RPO | SLA Enforcement |
|---|---|---|---|
| Enterprise | ≤2h | ≤15min | Contractual; breach triggers MSA penalty clause; force-majeure clause may apply |
| Growth | ≤4h | ≤1h | Operational SLA |
| Consumer | ≤8h | ≤4h | Best effort |

Note: If the root cause is confirmed as a Cloudflare platform incident, compliance-officer must assess whether the MSA force-majeure clause applies before any SLA credit is issued.

---

## 3. Detection Signals

- **Sentry alert FORM-DR-EDGE-001** — 100% of Workers health-check probes failing across all regions
- **`https://www.cloudflarestatus.com`** — incident on Cloudflare Workers or core routing components
- **`wrangler tail --env production`** — shows zero invocations (no traffic reaching Workers)
- **Better Stack / Grafana Cloud** — `workers_health_probe_success_rate` = 0% for >2min
- **PostHog** — complete drop in all event ingestion (leading indicator; PostHog itself may be affected if EU-hosted via Cloudflare)
- **Cloudflare Analytics** — requests graph shows cliff-edge drop to zero

Escalate to founder within 15min of P0 declaration (IC: devops-lead; backup IC: security-engineer).

---

## 4. Pre-Flight Checklist

- [ ] Confirm 100% failure (not partial): check `wrangler deployments list --env production` for current deployment hash
- [ ] Confirm `https://www.cloudflarestatus.com` shows a Workers or platform-wide incident
- [ ] Rule out FORM deployment as root cause: compare last deploy timestamp with first error timestamp in Sentry
- [ ] Rule out DNS misconfiguration: `dig api.formapp.com` — confirm CNAME points to `*.workers.dev`
- [ ] Open Linear incident ticket — P0, tag `edge-outage`, link to cloudflarestatus.com incident URL
- [ ] Confirm backup IC (security-engineer) is reachable
- [ ] Confirm founder is reachable (≤15min from P0 declaration)
- [ ] Confirm compliance-officer is reachable (for force-majeure + SLA assessment)
- [ ] Note exact Cloudflare incident URL and start timestamp in Linear ticket

---

## 5. Step-by-Step Recovery Procedure

### Step 1 — Confirm root cause: FORM deployment vs. Cloudflare platform

```bash
# Check current deployment and compare hash to last known good
wrangler deployments list --env production

# Tail logs — if Workers are reachable at all, you will see output
wrangler tail --env production --format pretty

# Check Cloudflare platform status
curl -s https://www.cloudflarestatus.com/api/v2/summary.json \
  | jq '.components[] | select(.name | test("Workers|Routing|DNS"))'
```

Responsible: devops-lead | Time constraint: complete within 10min of alert

**Decision gate:**
- If last deploy timestamp matches first error timestamp → FORM deployment issue → proceed to Step 2
- If `cloudflarestatus.com` shows a Workers incident → Cloudflare platform issue → skip to Step 3

---

### Step 2 — FORM deployment rollback (if FORM caused the failure)

```bash
# List deployments and identify last known good hash
wrangler deployments list --env production

# Roll back to previous deployment
wrangler rollback --env production

# Verify recovery
wrangler tail --env production --format pretty
curl -f https://api.formapp.com/health
```

Target: full recovery in <15min from detection. If rollback does not restore service, re-evaluate as Cloudflare platform issue.

Responsible: devops-lead | Time constraint: complete within 15min of diagnosis

---

### Step 3 — Cloudflare platform incident: no direct recovery action

This is a vendor outage. Do not attempt actions that cannot succeed.

Actions in parallel:

- Record Cloudflare incident URL and start timestamp in Linear ticket
- Begin communication tree (see Section 8)
- Set 15min status-check cadence on `https://www.cloudflarestatus.com`
- Notify enterprise contacts within 10min of P0 declaration
- Document start timestamp for SLA breach calculation

Responsible: devops-lead (monitoring) + compliance-officer (SLA) | Time constraint: ongoing

---

### Step 4 — Partial regional failure: attempt alternate-zone routing

Use only if `cloudflarestatus.com` shows a regional (not global) Workers failure.

```bash
# If alt-zone Worker environment is configured
wrangler deploy --env alt-zone

# Update Cloudflare DNS to point api.formapp.com at alt-zone worker
# Via Cloudflare Dashboard: DNS → CNAME record for api.formapp.com
# Or via API:
curl -X PATCH "https://api.cloudflare.com/client/v4/zones/${CF_ZONE_ID}/dns_records/${DNS_RECORD_ID}" \
  -H "Authorization: Bearer ${CF_API_TOKEN}" \
  -H "Content-Type: application/json" \
  -d '{"content": "<alt-zone-worker-url>"}'
```

Time budget: 45min to attempt re-routing. If not resolved within 45min, abandon and treat as global outage.

Responsible: devops-lead | Time constraint: decision to attempt by T+20min if regional

---

### Step 5 — Background job queue management

During outage, queue any deferrable background tasks for replay on recovery:

- Supabase DB triggers for async events remain queued in the database (no Workers action needed)
- Cloudflare Queues: messages are retained for up to 4 days by default; no action required
- Stripe webhooks: Stripe retries failed webhook deliveries for 72h automatically
- Log expected queue depth in Linear ticket for post-recovery flush verification

Responsible: platform-engineer | Time constraint: document within 30min of P0

---

### Step 6 — Recovery confirmation

When `cloudflarestatus.com` shows the incident resolved:

```bash
# Confirm Workers are responding
curl -f https://api.formapp.com/health

# Confirm no invocation gap artifacts
wrangler tail --env production --format pretty

# Flush any stalled Cloudflare Queue messages if needed
wrangler queues consumer list <queue-name> --env production
```

Emit `system.sla_breach_recorded` if outage exceeded Enterprise RTO (2h):

```sql
INSERT INTO audit_log_events (event_type, severity, actor_type, payload, occurred_at)
VALUES (
  'system.sla_breach_recorded',
  'HIGH',
  'system',
  jsonb_build_object(
    'breach_type', ARRAY['enterprise_rto'],
    'runbook', 'FORM-DR-002',
    'root_cause', 'cloudflare_platform_incident',
    'cloudflare_incident_url', '<CF_INCIDENT_URL>',
    'outage_duration_min', <ACTUAL_MINUTES>,
    'force_majeure_claimed', true,
    'incident_ref', '<LINEAR_TICKET_ID>'
  ),
  now()
);
```

Responsible: devops-lead | Time constraint: within 15min of Cloudflare declaring incident resolved

---

## 6. Rollback / Abort Procedure

- If FORM rollback (Step 2) does not restore service within 15min: confirm Cloudflare platform involvement; stop FORM-side actions and shift to communication mode.
- If alt-zone routing (Step 4) causes DB connection errors (wrong `DATABASE_URL`): revert DNS immediately via Cloudflare Dashboard; check alt-zone `DATABASE_URL` secret.
- Never attempt repeated `wrangler deploy` cycles during a Cloudflare platform incident — it will not help and may consume deploy budget.

---

## 7. Evidence Capture Instructions

Upload to R2 `form-cold-backups/incidents/<LINEAR_TICKET_ID>/`:

- [ ] Screenshot: `https://www.cloudflarestatus.com` showing incident with URL and timestamps
- [ ] `wrangler deployments list` output at time of detection
- [ ] Sentry trace export showing zero invocations during outage window (FORM-DR-EDGE-001)
- [ ] Terminal output: `wrangler rollback` result (if applicable)
- [ ] Terminal output: `curl -f https://api.formapp.com/health` at time of recovery confirmation
- [ ] DEC-030 `system.sla_breach_recorded` event (if RTO breached)
- [ ] DEC-030 `system.dr_drill_completed` event (if drill)

All audit log payloads must use `actor_uuid`, `client_ip_hash` (SHA-256 with `IP_HASH_SALT`), and must not contain plaintext health data.

---

## 8. Post-Incident Actions

### Customer Communication

| Tier | Channel | Timing | Frequency |
|---|---|---|---|
| Enterprise | Phone + email | Within 10min of P0 declaration | Every 30min until resolved |
| Growth | Email | Within 15min of P0 declaration | Every 60min until resolved |
| Consumer | Status page | Within 30min of P0 declaration | On change |

Email template note: include Cloudflare incident URL, state that recovery is vendor-dependent, confirm Supabase data is safe and unaffected.

### Force-Majeure Assessment

- If outage duration >2h and root cause is confirmed Cloudflare platform: compliance-officer documents Cloudflare incident URL, duration, and customer impact in Linear ticket.
- Compliance-officer assesses enterprise SLA credit applicability under MSA force-majeure clause.
- Legal review required before any credit denial based on force-majeure.

### Post-Mortem

- Trigger: always for P0, regardless of root cause
- Timeline: within 48h of resolution
- Owner: devops-lead
- Include: MTTD, MTTR, whether FORM-side rollback was possible, force-majeure assessment outcome, action items (e.g., alt-zone failover automation)

---

## 9. SOC 2 TSC Cross-Reference

| TSC | How this runbook satisfies it |
|---|---|
| A1.1 | RTO targets defined; vendor dependency and force-majeure constraints documented |
| A1.2 | Detection signals defined; communication tree activated within prescribed timeframes; DR drills include this scenario |
| A1.3 | Recovery verification via health endpoint; queue replay verified; SLA breach recorded with `system.sla_breach_recorded` event |

---

*v1.0 · 2026-06-06 · devops-lead + compliance-officer*
