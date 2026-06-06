# FORM-DR-004 · Anthropic Claude API Unavailability

> **Runbook ID:** FORM-DR-004
> **Severity:** P1
> **Owner:** devops-lead · reviewed by: compliance-officer · enterprise-architect
> **Last reviewed:** 2026-06-06
> **SOC 2 TSC:** A1.1, A1.2
> **Cross-refs:** docs/SOC2_READINESS.md §53; docs/INCIDENT_RESPONSE.md §IR-4; docs/OBSERVABILITY.md §OBS-5

---

## 1. Scenario Summary

| Field | Value |
|---|---|
| ID | FORM-DR-004 |
| Severity | P1 |
| Description | Anthropic Claude API unavailable for > 2 consecutive hours. Victor (AI coaching) is non-functional. Workout logging, CV analysis (on-device), Stripe billing, and all non-AI features remain fully operational. |
| Affected tiers | All tiers for AI coaching feature only |
| Estimated RTO for graceful degradation | ≤ 15 min (FORM obligation) |
| Estimated RTO for full Victor restoration | Vendor-dependent; no FORM recovery action possible |

**Critical constraint:** No FORM RTO applies to Victor restoration — this is a vendor dependency on Anthropic's infrastructure. FORM's sole obligation is to activate graceful degradation within 15 min of detection, and to communicate accurately to enterprise customers.

---

## 2. RTO/RPO Targets

| Tier | RTO (degradation active) | RPO | SLA Enforcement |
|---|---|---|---|
| Enterprise | ≤ 15 min (degradation) | N/A — coaching state preserved in DB | Contractual for degradation SLA; Victor restoration is vendor-dependent |
| Growth | ≤ 15 min (degradation) | N/A | Operational SLA |
| Consumer | ≤ 15 min (degradation) | N/A | Best effort |

RPO note: all workout logs and session data are written to Supabase independently of the Anthropic API. No user health data is lost during a Claude API outage.

---

## 3. Detection Signals

- **Sentry alert FORM-DR-AI-001** — Anthropic API returning 5xx for > 5 consecutive coaching requests
- **https://status.anthropic.com** — incident declared on Claude API or model inference components
- **Better Stack / Grafana Cloud** — `anthropic_api_success_rate` < 90% for > 3 min
- **PostHog** — sharp drop in `coaching_turn_completed` events while `session_started` remains healthy (distinguishes AI failure from general outage)
- **ElevenLabs TTS** calls may also fail downstream if Victor response pipeline is blocked

Escalate to devops-lead immediately. IC: devops-lead; platform-engineer as backup.

---

## 4. Pre-Flight Checklist

- [ ] Confirm Sentry alert FORM-DR-AI-001 is active (not a false-positive test spike)
- [ ] Check https://status.anthropic.com for an active incident
- [ ] Confirm the failure is Anthropic API, not a FORM Worker bug: check Sentry error details for `anthropic_api_error` vs. `worker_internal_error`
- [ ] Confirm `ANTHROPIC_API_KEY` Workers Secret is valid and unexpired via `wrangler secret list --env production`
- [ ] Open Linear incident ticket — P1, tag `ai-outage`, link to status.anthropic.com incident URL
- [ ] Confirm platform-engineer (backup IC) is reachable
- [ ] Note exact outage start timestamp for SLA duration calculation

---

## 5. Step-by-Step Recovery Procedure

### Step 1 — Activate Victor fallback flag (devops-lead; within 15 min of detection)

```bash
# Write fallback flag to Cloudflare KV — Workers read this on every request
wrangler kv key put VICTOR_FALLBACK "true" \
  --namespace-id "${KV_NAMESPACE_ID}" \
  --env production
```

Workers must check this flag before invoking the Anthropic API. When `VICTOR_FALLBACK=true`, return the static fallback message to users:

> *"Victor is temporarily unavailable. Your session has been saved and coaching will resume when the service is restored."*

Verify the flag is active and Workers are serving the fallback response:

```bash
curl -sf -H "Authorization: Bearer ${TEST_TOKEN}" \
  https://api.formcoach.app/api/v1/coaching/message \
  -d '{"message": "test"}' | jq '.fallback'
# Expect: true
```

---

### Step 2 — Queue user coaching requests for retry (platform-engineer; within 15 min of detection)

Coaching requests received while `VICTOR_FALLBACK=true` are enqueued in Cloudflare Queues with a 2 h retry window:

```bash
# Confirm queue consumer is running and depth is increasing as expected
wrangler queues consumer list coaching-requests --env production

# Monitor queue depth (check every 30 min)
wrangler queues stats coaching-requests --env production
```

The app UI must surface the queued state to the user (pending indicator on coaching turn). Confirm with platform-engineer that UI state is correct.

---

### Step 3 — Disable ElevenLabs TTS for fallback messages (devops-lead; within 15 min of detection)

Fallback messages must not trigger ElevenLabs TTS calls — there is no Victor response to speak.

```bash
# Disable TTS via Cloudflare KV flag
wrangler kv key put VICTOR_TTS_ENABLED "false" \
  --namespace-id "${KV_NAMESPACE_ID}" \
  --env production
```

This prevents wasted ElevenLabs API spend during the outage. Confirm TTS calls are suppressed in Sentry — expect zero `elevenlabs_request` events while flag is false.

---

### Step 4 — Monitor Anthropic status page (devops-lead; every 30 min)

```bash
# Automated status check — run every 30 min; log result in Linear ticket
curl -s https://status.anthropic.com/api/v2/summary.json \
  | jq '.components[] | select(.name | test("API|Claude|Inference"))'
```

Record each check result (timestamp + component status) in the Linear ticket.

---

### Step 5 — SLA breach assessment at 6 h (compliance-officer; at T+6 h if still down)

If Anthropic outage exceeds 6 h: compliance-officer assesses enterprise SLA credit applicability. Document in Linear ticket:
- Anthropic incident URL and status page timestamps
- Enterprise tenants affected and coaching request queue depth
- Whether force-majeure clause in enterprise MSA applies

---

### Step 6 — Recovery: restore Victor (devops-lead; on Anthropic recovery)

When https://status.anthropic.com shows the incident resolved:

**6a. Verify Anthropic API is responding**

```bash
curl -sf https://api.anthropic.com/v1/messages \
  -H "x-api-key: ${ANTHROPIC_API_KEY}" \
  -H "anthropic-version: 2023-06-01" \
  -H "Content-Type: application/json" \
  -d '{"model":"claude-sonnet-4-6","max_tokens":10,"messages":[{"role":"user","content":"ping"}]}' \
  | jq '.content[0].text'
```

**6b. Disable fallback flag and re-enable TTS**

```bash
wrangler kv key put VICTOR_FALLBACK "false" \
  --namespace-id "${KV_NAMESPACE_ID}" \
  --env production

wrangler kv key put VICTOR_TTS_ENABLED "true" \
  --namespace-id "${KV_NAMESPACE_ID}" \
  --env production
```

**6c. Flush queued coaching requests in priority order**

Process enterprise tenant queued requests first, then Growth, then Consumer. Verify coaching responses are valid before resuming normal queue processing:

```bash
# Spot-check first 3 responses from queue
wrangler queues consumer peek coaching-requests --count 3 --env production | jq .
```

---

## 6. Rollback / Abort Procedure

- If disabling fallback (`VICTOR_FALLBACK=false`) results in continued 5xx errors from Anthropic: immediately re-enable the fallback flag.
- If Anthropic API returns throttling errors (429) after recovery: stagger queue flush over 15 min; do not burst the full queue depth immediately.
- If TTS re-enablement causes ElevenLabs errors: re-disable TTS; investigate ElevenLabs status separately (ElevenLabs may have its own outage concurrent with Anthropic).

```bash
# Quick rollback: re-enable fallback
wrangler kv key put VICTOR_FALLBACK "true" --namespace-id "${KV_NAMESPACE_ID}" --env production
wrangler kv key put VICTOR_TTS_ENABLED "false" --namespace-id "${KV_NAMESPACE_ID}" --env production
```

---

## 7. Evidence Capture Instructions

Upload all evidence to R2 `form-cold-backups/incidents/<LINEAR_TICKET_ID>/`:

- [ ] Screenshot: https://status.anthropic.com showing incident timeline with start and end timestamps
- [ ] Sentry FORM-DR-AI-001 alert export showing 5xx event timeline
- [ ] Terminal output: `wrangler kv key get VICTOR_FALLBACK` showing flag state at activation and deactivation
- [ ] Cloudflare KV screenshot showing flag state during outage window
- [ ] Queue depth metrics during outage (from `wrangler queues stats` or Grafana)
- [ ] Terminal output: Anthropic API spot-check at recovery (Step 6a)
- [ ] DEC-030 `system.dr_drill_completed` event if this was a drill:

```sql
INSERT INTO audit_log_events (event_type, severity, actor_type, payload, occurred_at)
VALUES (
  'system.dr_drill_completed',
  'INFO',
  'system',
  jsonb_build_object(
    'runbook',               'FORM-DR-004',
    'drill_started_at',      '<ISO8601>',
    'drill_completed_at',    now()::text,
    'fallback_activated',    true,
    'tts_disabled',          true,
    'queue_depth_at_peak',   '<COUNT>'
  ),
  now()
);
```

Privacy floor: coaching queue messages must not contain plaintext health data in audit log payloads. Use `session_uuid` and `actor_uuid` only.

---

## 8. Post-Incident Actions

### Customer Communication

| Tier | Channel | Timing | Frequency |
|---|---|---|---|
| Enterprise | Email | Within 15 min if outage > 2 h | Every 60 min until Victor restored |
| Growth | In-app notice | Within 30 min (surfaced via fallback message) | At resolution |
| Consumer | In-app notice | Within 30 min (surfaced via fallback message) | At resolution |

Email to enterprise must: include Anthropic incident URL, confirm core features (logging, CV analysis, billing) are fully operational, state ETA from Anthropic status page if available, confirm all queued coaching requests will be processed on recovery.

### SLA Breach Assessment

- Graceful degradation must activate ≤ 15 min from detection. If this was not achieved: devops-lead documents root cause; compliance-officer reviews enterprise SLA applicability.
- Full Victor restoration is vendor-dependent and does not count against FORM RTO.

### Post-Mortem

- Trigger: if degradation did not activate within 15 min, or if outage exceeded 6 h
- Timeline: within 48 h of resolution
- Owner: devops-lead
- Include: MTTD, time-to-fallback-activation, queue depth and flush duration, TTS suppression effectiveness, actions to improve fallback automation

---

## 9. SOC 2 TSC Cross-Reference

| TSC | How this runbook satisfies it |
|---|---|
| A1.1 | Graceful degradation SLA (≤ 15 min) defined; vendor dependency documented; non-AI features confirmed unaffected |
| A1.2 | Fallback flag mechanism tested in DR drills; queue replay verified post-recovery; TTS cost suppression documented |

---

*v1.0 · 2026-06-06 · devops-lead + compliance-officer*
