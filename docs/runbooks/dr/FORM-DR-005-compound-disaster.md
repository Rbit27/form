# FORM-DR-005 · Compound Disaster — Simultaneous Multi-Component Failure

> **Runbook ID:** FORM-DR-005
> **Severity:** P0
> **Owner:** devops-lead · reviewed by: compliance-officer · enterprise-architect
> **Last reviewed:** 2026-06-06
> **SOC 2 TSC:** A1.1, A1.2, A1.3
> **Cross-refs:** docs/SOC2_READINESS.md §53; docs/INCIDENT_RESPONSE.md §IR-5; docs/OBSERVABILITY.md §OBS-6; FORM-DR-001; FORM-DR-002; FORM-DR-003; FORM-DR-004

---

## 1. Scenario Summary

| Field | Value |
|---|---|
| ID | FORM-DR-005 |
| Severity | P0 |
| Description | Simultaneous failure of two or more FORM system components. Examples: Supabase outage + R2 corruption; Workers failure + Anthropic outage. Requires prioritised triage across multiple parallel recovery tracks. |
| Affected tiers | Enterprise, Growth, Consumer — scope depends on which components have failed |
| Estimated RTO | Enterprise RTO ≤ 2 h almost certainly breached; SLA breach event expected |
| Primary obligation | Triage and communicate within 15 min; founder reachable within 15 min of P0 declaration |

---

## 2. RTO/RPO Targets

| Tier | RTO | RPO | SLA Enforcement |
|---|---|---|---|
| Enterprise | ≤ 2 h (contractual; likely breached in compound event) | ≤ 15 min | MSA penalty clause applies; force-majeure assessed per component |
| Growth | ≤ 4 h | ≤ 1 h | Operational SLA |
| Consumer | ≤ 8 h | ≤ 4 h | Best effort |

In a compound disaster, Enterprise RTO breach must be assumed from T+15 min. Emit `system.sla_breach_recorded` with all breached components listed. Do not wait to confirm breach — emit early and correct if unnecessary.

---

## 3. Detection Signals

Multiple concurrent alerts from two or more of the following:

- **FORM-DR-DB-001** — Supabase health-check failing (see FORM-DR-001 §3)
- **FORM-DR-EDGE-001** — Workers health-check probes all failing (see FORM-DR-002 §3)
- **FORM-DR-R2-001** — R2 object reads returning 404/500 at > 1% (see FORM-DR-003 §3)
- **FORM-DR-AI-001** — Anthropic API returning 5xx on coaching requests (see FORM-DR-004 §3)
- **Better Stack / Grafana Cloud** — multiple `*_reachable` metrics at 0 simultaneously
- **PostHog** — complete drop in all event types (distinguishes from AI-only outage)

Two or more concurrent P0/P1 alerts from distinct system components = compound disaster. Declare FORM-DR-005 and open a single compound incident ticket.

---

## 4. Pre-Flight Checklist

- [ ] Declare compound P0 and open single Linear incident ticket — tag all affected components (e.g., `db-outage`, `edge-outage`, `r2-data-loss`, `ai-outage`)
- [ ] Confirm founder is reachable — must be reached within 15 min of P0 declaration (not optional)
- [ ] Confirm all IC contacts are reachable before starting triage: devops-lead (IC), security-engineer (backup), platform-engineer
- [ ] Confirm compliance-officer is reachable (force-majeure + SLA assessment will be needed)
- [ ] Create sub-tasks in the Linear compound ticket — one per affected component — linked to parent
- [ ] Assign one responder per component track (do not have one person attempt multiple simultaneous recoveries)
- [ ] Emit `system.sla_breach_recorded` with `breach_type` array covering all affected components (see §5 Step 1)
- [ ] Assess force-majeure eligibility immediately if Cloudflare is one of the failed components
- [ ] Notify enterprise customers within 10 min of compound P0 declaration — even before scope is fully determined

---

## 5. Step-by-Step Recovery Procedure

### Step 1 — Declare compound P0 and emit SLA breach event (devops-lead; T+0 to T+5 min)

```sql
INSERT INTO audit_log_events (event_type, severity, actor_type, payload, occurred_at)
VALUES (
  'system.sla_breach_recorded',
  'HIGH',
  'system',
  jsonb_build_object(
    'breach_type',          ARRAY['enterprise_rto', 'enterprise_rpo'],
    'runbook',              'FORM-DR-005',
    'components_failed',    ARRAY['<component_1>', '<component_2>'],
    'expected_rto_min',     120,
    'projected_rto_min',    NULL,
    'force_majeure_scope',  '<cloudflare|none|tbd>',
    'incident_ref',         '<LINEAR_TICKET_ID>',
    'escalated_to_founder', true
  ),
  now()
);
```

Replace `components_failed` with the actual list (e.g., `ARRAY['supabase', 'r2']`). Update `projected_rto_min` when the triage gives a better estimate.

---

### Step 2 — Enterprise customer notification (compliance-officer or devops-lead; T+10 min)

Notify enterprise contacts within 10 min of P0 declaration. Message must include:

- The compound event is in progress (multiple components affected)
- The scope is still being determined
- Devops-lead is the IC and is actively triaging
- Next update will follow within 30 min
- Core data integrity is not at risk (clarify if DB is unaffected)

Do not wait until scope is fully known. Silence for > 10 min is a communication SLA breach in itself.

---

### Step 3 — Triage: identify which components are failed (devops-lead; T+0 to T+15 min)

Run all checks in parallel:

```bash
# Database
curl -sf "https://${SUPABASE_PROJECT_REF}.supabase.co/rest/v1/" \
  -H "apikey: ${SUPABASE_ANON_KEY}" | jq '.error // "DB OK"'

# Workers edge
curl -sf https://api.formcoach.app/health | jq . || echo "WORKERS FAILED"

# R2 bucket
wrangler r2 object head form-production/health-check-probe.txt && echo "R2 OK" || echo "R2 FAILED"

# Anthropic API
curl -sf https://status.anthropic.com/api/v2/summary.json \
  | jq '.components[] | select(.name | test("API|Claude")) | .status'

# Cloudflare platform
curl -sf https://www.cloudflarestatus.com/api/v2/summary.json \
  | jq '.components[] | select(.name | test("Workers|R2")) | {name, status}'
```

Document results in the Linear ticket with timestamps. This determines which component runbooks are activated.

---

### Step 4 — Execute component recoveries in priority order

Recovery must proceed in this sequence. Do not skip ahead. Assign a separate responder to each track if headcount allows.

#### Priority 1: Restore database access — execute FORM-DR-001

Database is the foundation. No other recovery is meaningful without DB access.

```bash
# IC devops-lead activates FORM-DR-001 protocol
# Sub-task owner: devops-lead
# Reference: docs/runbooks/dr/FORM-DR-001-supabase-outage.md
```

Target: DB accessible within 90 min. If PITR unavailable, emit updated `system.sla_breach_recorded` with `path: cold_backup`.

#### Priority 2: Restore API layer — execute FORM-DR-002

Only after DB access is confirmed or a concrete ETA is established.

```bash
# Sub-task owner: devops-lead or security-engineer (backup IC)
# Reference: docs/runbooks/dr/FORM-DR-002-workers-edge-failure.md
```

If root cause is Cloudflare platform: no direct recovery action; activate force-majeure assessment immediately.

#### Priority 3: Restore object storage — execute FORM-DR-003

Begin partial/full R2 restore from `form-cold-backups` after DB access is confirmed. Enterprise tenant objects restored first.

```bash
# Sub-task owner: platform-engineer
# Reference: docs/runbooks/dr/FORM-DR-003-r2-data-loss.md
```

#### Priority 4: Communicate AI degradation — execute FORM-DR-004 posture

If Anthropic API is also failed: activate `VICTOR_FALLBACK=true` and disable TTS. This is a 5-min action and can be parallelised with Priority 1.

```bash
# Can run in parallel with Priority 1 — does not depend on DB access
# Sub-task owner: platform-engineer
wrangler kv key put VICTOR_FALLBACK "true" --namespace-id "${KV_NAMESPACE_ID}" --env production
wrangler kv key put VICTOR_TTS_ENABLED "false" --namespace-id "${KV_NAMESPACE_ID}" --env production
```

---

### Step 5 — Rollback safety gate (devops-lead; continuous throughout)

Before each step in any component recovery, confirm that the action cannot harm another component.

Known conflict: Workers redeployment (FORM-DR-002 Step 2 or Step 4) will apply the current `DATABASE_URL` Workers Secret. If DB migration is in progress (FORM-DR-001 PITR restore is not yet complete), a Workers redeploy pointing at the partially-restored cluster may cause cascading errors.

**Rule:** Do not redeploy Workers until the DB is confirmed healthy and all RLS tests pass.

If recovery of one component causes harm to another:
1. Halt the offending action immediately
2. Document the failure mode in the Linear ticket
3. Escalate to founder immediately — do not attempt a second recovery without founder sign-off
4. Revert the harmful action (use `wrangler rollback` for Workers; restore `DATABASE_URL` to last-known-good)

---

### Step 6 — Compound recovery confirmation (devops-lead; after all component recoveries complete)

```bash
# Full system smoke test — run after all components are declared healthy by their respective owners
curl -sf https://api.formcoach.app/health | jq .

curl -sf -H "Authorization: Bearer ${TEST_TOKEN}" \
  https://api.formcoach.app/api/v1/sessions/recent | jq 'length'

curl -sf -H "Authorization: Bearer ${TEST_TOKEN}" \
  https://api.formcoach.app/api/v1/coaching/message \
  -d '{"message": "test"}' | jq '.fallback // "Victor OK"'
```

Declare recovery only when all four component checks pass. Update the Linear ticket and statuspage.io.

---

## 6. Rollback / Abort Procedure

General principles:

- Each component rollback follows its respective runbook (FORM-DR-001 §6, FORM-DR-002 §6, FORM-DR-003 §6, FORM-DR-004 §6).
- If any recovery action worsens the compound event: halt immediately; document; escalate to founder before retrying.
- If resource contention prevents parallel recovery (e.g., only one engineer available): database recovery takes absolute priority; all other tracks pause.
- The compound incident ticket must not be closed until all component sub-tasks are closed.
- Never destroy any DR environment (ephemeral DB, alt-zone Worker deployment) until compound recovery is confirmed for ≥ 30 min.

---

## 7. Evidence Capture Instructions

Upload all evidence to R2 `form-cold-backups/incidents/<LINEAR_TICKET_ID>/` with sub-folders per component:

- [ ] Linear compound incident ticket screenshot showing all sub-tasks and their owners
- [ ] DEC-030 `system.sla_breach_recorded` event export (from Step 1) — include `components_failed` array
- [ ] Evidence from each component recovery per its own runbook (FORM-DR-001 §7, FORM-DR-002 §7, FORM-DR-003 §7, FORM-DR-004 §7)
- [ ] Enterprise notification timestamps (email send receipts or Linear log entries)
- [ ] Triage check output from Step 3 (terminal output with timestamps for each component)
- [ ] Full system smoke test output from Step 6
- [ ] DEC-030 `system.dr_drill_started` event if this was a drill (emit at start):

```sql
INSERT INTO audit_log_events (event_type, severity, actor_type, payload, occurred_at)
VALUES (
  'system.dr_drill_started',
  'INFO',
  'system',
  jsonb_build_object(
    'runbook',          'FORM-DR-005',
    'drill_started_at', now()::text,
    'scenario',         'compound_disaster',
    'components',       ARRAY['<component_1>', '<component_2>']
  ),
  now()
);
```

- [ ] DEC-030 `system.dr_drill_completed` event at drill conclusion:

```sql
INSERT INTO audit_log_events (event_type, severity, actor_type, payload, occurred_at)
VALUES (
  'system.dr_drill_completed',
  'INFO',
  'system',
  jsonb_build_object(
    'runbook',             'FORM-DR-005',
    'drill_started_at',    '<ISO8601>',
    'drill_completed_at',  now()::text,
    'components_tested',   ARRAY['<component_1>', '<component_2>'],
    'rto_achieved_min',    '<ACTUAL_MINUTES>',
    'sla_breach_emitted',  true
  ),
  now()
);
```

Privacy floor: all audit event payloads use `actor_uuid` and `client_ip_hash` only. No plaintext health data, names, or raw IPs in any evidence artifact.

---

## 8. Post-Incident Actions

### Customer Communication

| Tier | Channel | Timing | Frequency |
|---|---|---|---|
| Enterprise | Phone + email | Within 10 min of P0 declaration | Every 30 min until all components restored |
| Growth | Email | Within 15 min of P0 declaration | Every 60 min until all components restored |
| Consumer | Status page | Within 30 min of P0 declaration | On each component restoration milestone |

Post-incident report for Enterprise: due within **72 h** of resolution (not the standard 48 h — complexity requires additional time). Report must cover all affected components, independent RTO/RPO analysis per component, and root-cause analysis for the compound event.

### Force-Majeure Assessment

- If Cloudflare is one of the failed components: compliance-officer assesses MSA force-majeure applicability separately from any FORM-caused components.
- Force-majeure can apply to Cloudflare-related components only; FORM-caused failures (e.g., bad deploy) are not eligible.
- Legal review required before any credit denial based on force-majeure in a compound event.

### SLA Breach Assessment

Compound events almost always breach Enterprise RTO. The `system.sla_breach_recorded` event emitted in Step 1 is the canonical record. Compliance-officer reviews penalty clause applicability for each Enterprise tenant per their MSA. Document in the compound Linear ticket.

### Post-Mortem

- Trigger: always for compound P0
- Timeline: within **72 h** of resolution (extended from standard 48 h)
- Owner: devops-lead
- Attendees: devops-lead, platform-engineer, security-engineer, compliance-officer, founder
- Template: docs/INCIDENT_RESPONSE.md §IR-5
- Required: MTTD per component, MTTR per component, triage decision timeline, rollback conflicts encountered, enterprise notification timestamps, action items for improving compound-failure detection and automated triage

---

## 9. SOC 2 TSC Cross-Reference

| TSC | How this runbook satisfies it |
|---|---|
| A1.1 | Triage priority order defined; enterprise RTO breach assumed and recorded early; founder escalation required within 15 min |
| A1.2 | Component runbooks activated in defined priority order; compound DR drills include this scenario with all components exercised |
| A1.3 | Rollback safety gate prevents recovery of one component from harming another; `system.sla_breach_recorded` emitted with all breached components; extended post-mortem required |

---

*v1.0 · 2026-06-06 · devops-lead + compliance-officer*
