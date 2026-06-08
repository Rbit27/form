# RB-SUB-02 · Grace Period Overshoot

> **Runbook ID:** RB-SUB-02
> **Severity:** P0
> **Owner:** billing-owner · reviewed by: platform-engineer · compliance-officer
> **Last reviewed:** 2026-06-08
> **SOC 2 TSC:** PI1.5, CC7.2, CC6.1
> **Cross-refs:** docs/OBSERVABILITY.md §24.2, §24.5 (AL-SUB-02), §24.8; docs/AUDIT_LOG_SCHEMA.md DEC-030; docs/DATA_MODEL.md §24.7; docs/INCIDENT_RESPONSE.md §1.2

---

## 1. Scenario Summary

| Field | Value |
|---|---|
| ID | RB-SUB-02 |
| Severity | P0 |
| Description | One or more users in `past_due` or `canceled` state have `subscription_events.valid_to < now() - interval '72 hours'` AND `users.subscription_tier = 'premium'`. These users are receiving premium feature access beyond the contractual 72-hour grace period. This is a billing integrity and access-control failure. |
| Affected tiers | Consumer (RevenueCat). Enterprise billing uses invoice-payment flow; grace period applies to consumer tier only. |
| Estimated resolution time | 15–45 min |

---

## 2. Detection Signals

**How the alert fires:**
- The pg_cron job `grace-period-enforcer` runs every 30 minutes. It queries for users in `past_due` state whose latest `subscription_events.valid_to < now() - interval '72 hours'` and `users.subscription_tier = 'premium'`. On detection, it emits a `billing.consistency_check_failed` DEC-030 CRITICAL event and triggers PagerDuty P0.
- The daily `billing-consistency-check` (03:00 UTC) also catches any overshoot that `grace-period-enforcer` missed (e.g., if the cron job itself was failing).

**Key signals to check:**
- PagerDuty incident body: includes `user_id_hash` list, `valid_to` timestamps, `overshoot_hours`
- Metabase "Subscription Health" panel 4 (Grace-Period Users): count > 0 where `valid_to < now() - interval '72 hours'`
- `cron.job_run_details` in Supabase: verify `grace-period-enforcer` has been running on schedule
- Cloudflare Analytics Engine: `billing.webhook.processed{outcome="error"}` — elevated errors may explain why the enforcer did not fire `billing.access_revoked` on time

---

## 3. Pre-Flight Checklist

- [ ] Confirm PagerDuty alert is not a drill (`alert_body.is_drill = false`)
- [ ] Open a Linear incident ticket — P0, tag `billing-grace-period`, link to PagerDuty alert
- [ ] Retrieve `user_id_hash` list and `valid_to` values from alert body
- [ ] Check `cron.job_run_details` for `grace-period-enforcer` — confirm it ran in the last 30 minutes
- [ ] Check `cron.job_run_details` for `billing-consistency-check` — confirm last run result
- [ ] Confirm RevenueCat status: https://status.revenuecat.com (upstream outage may have delayed cancellation events)
- [ ] Confirm Cloudflare Workers status: https://www.cloudflarestatus.com

---

## 4. Step-by-Step Procedure

**1. Identify all affected users** (on-call; T+0)

```sql
-- Run as compliance_officer or form_system role
-- Identify users with expired grace period still on premium
SELECT
    u.id                          AS user_id,
    u.subscription_tier           AS current_tier,
    se.to_state                   AS subscription_state,
    se.valid_to                   AS grace_expired_at,
    EXTRACT(EPOCH FROM (now() - se.valid_to)) / 3600 AS hours_overdue,
    se.source,
    se.idempotency_key
FROM users u
JOIN LATERAL (
    SELECT to_state, valid_to, source, idempotency_key
    FROM subscription_events
    WHERE user_id = u.id
    ORDER BY created_at DESC
    LIMIT 1
) se ON true
WHERE u.subscription_tier = 'premium'
  AND se.to_state IN ('past_due', 'canceled')
  AND se.valid_to < now() - interval '72 hours'
ORDER BY se.valid_to ASC;
```

- Count of results determines scope. Zero rows = alert was a false positive (proceed to Step 5 — false positive investigation).
- Any result requires immediate remediation (Step 2).

**2. Revoke access for all affected users** (billing-owner; T+5)

For each affected `user_id`, execute the following in a transaction:

```sql
-- Revoke premium access: update users table and insert access_revoked event
-- MUST be executed as form_system or compliance_officer role
-- Repeat for each affected user_id
BEGIN;

-- 1. Update subscription tier to free
UPDATE users
SET subscription_tier = 'free'
WHERE id = :affected_user_id
  AND subscription_tier = 'premium';

-- 2. Insert compensating subscription_events row
INSERT INTO subscription_events (
    user_id,
    tenant_id,
    event_type,
    source,
    from_state,
    to_state,
    raw_payload_hash,
    subscription_tier,
    valid_from,
    valid_to,
    idempotency_key,
    actor_id,
    processing_duration_ms
) VALUES (
    :affected_user_id,
    NULL,  -- consumer tier: tenant_id is NULL
    'subscription_canceled',
    'manual_admin',
    'past_due',
    'canceled',
    encode(digest('grace-overshoot-remediation-' || :incident_linear_id || '-' || :affected_user_id::text, 'sha256'), 'hex'),
    'free',
    now(),
    now(),
    'grace-revoke-' || :incident_linear_id || '-' || :affected_user_id::text,
    :admin_actor_id,
    0
);

COMMIT;
```

After committing, the DEC-030 `billing.access_revoked` event must be emitted. If the `grace-period-enforcer` did not emit it automatically, emit it manually via the internal Worker endpoint:

```bash
curl -s -X POST \
  "${INTERNAL_WORKER_BASE_URL}/internal/dec030/emit" \
  -H "Authorization: Bearer ${SERVICE_ROLE_KEY}" \
  -H "Content-Type: application/json" \
  -d '{
    "event_type": "billing.access_revoked",
    "severity": "CRITICAL",
    "payload": {
      "user_id_hash": "<SHA256_OF_USER_ID>",
      "tenant_id": null,
      "revocation_reason": "grace_period_expired",
      "actor_id": "form_system",
      "overshoot_hours": <HOURS_OVERDUE>,
      "incident_ref": "<LINEAR_TICKET_ID>"
    }
  }'
```

**3. Verify access revocation was applied** (on-call; T+15)

```sql
-- Confirm zero remaining overshoot users
SELECT COUNT(*) AS remaining_overshoot
FROM users u
JOIN LATERAL (
    SELECT to_state, valid_to
    FROM subscription_events
    WHERE user_id = u.id
    ORDER BY created_at DESC
    LIMIT 1
) se ON true
WHERE u.subscription_tier = 'premium'
  AND se.to_state IN ('past_due', 'canceled')
  AND se.valid_to < now() - interval '72 hours';
-- Expected: 0
```

**4. Diagnose why grace-period-enforcer did not fire** (platform-engineer; T+20)

```sql
-- Check cron job run history
SELECT
    jobname,
    runid,
    status,
    return_message,
    start_time,
    end_time
FROM cron.job_run_details
WHERE jobname = 'grace-period-enforcer'
ORDER BY start_time DESC
LIMIT 20;
```

| Return status | Action |
|---|---|
| `succeeded` | Enforcer ran but did not catch the overshoot — review the enforcer query logic for off-by-one in the 72h interval |
| `failed` | Enforcer was failing silently — open P1 follow-up to fix the cron job; review `cron_job_results` for recent failures |
| Gap in `start_time` sequence | Cron scheduler issue — check pg_cron configuration; escalate to platform-engineer |

**5. False-positive investigation** (on-call; T+5 if zero rows in Step 1)

```sql
-- Check whether billing_consistency_checks has a recent failed row
-- that triggered the alert without actual overshoot
SELECT
    check_name,
    check_date,
    passed,
    failure_count,
    failure_sample
FROM billing_consistency_checks
WHERE check_date > now() - interval '2 hours'
ORDER BY check_date DESC;
```

If the check row shows `passed = true` but PagerDuty fired: alert mis-routing or stale alert. Resolve PagerDuty and document in Linear ticket. Tune alert threshold if this recurs.

---

## 5. Rollback / Post-Resolution Steps

- Do NOT delete or UPDATE the `subscription_events` rows inserted as corrections. The table is append-only.
- If a revoked user contacts support claiming they paid successfully: verify via RevenueCat or Stripe dashboard before re-granting access. Any re-grant requires a `subscription_reactivated` event insert with `source = 'manual_admin'` and `actor_id` set.
- After all users are resolved: resolve PagerDuty alert.
- Confirm Metabase panel 4 shows count = 0.
- Update Linear ticket: how many users were affected, `user_id_hash` list (no plaintext UUIDs in Linear body — reference by count and incident ID), root cause.

---

## 6. Escalation Path

| Condition | Escalate to | Timeline |
|---|---|---|
| > 10 users affected | billing-owner + founder | Immediately |
| Enforcer cron job has been failing for > 24 h | platform-engineer + devops-lead | Immediately (treat as PI1.5 gap) |
| Evidence of deliberate access after cancellation | security-engineer + compliance-officer | Immediately |
| Affected users cannot be identified (hash mismatch) | compliance-officer | T+15 |
| Enterprise tenant user affected | customer-success + billing-owner | T+10 |

---

## 7. Evidence and DEC-030 Events to Emit

All evidence uploaded to R2 `form-cold-backups/incidents/<LINEAR_TICKET_ID>/`.

**DEC-030 events that must exist before closing this incident:**

1. `billing.access_revoked` (CRITICAL) — one per affected user, with `revocation_reason = 'grace_period_expired'`. Confirm via:

```sql
SELECT event_type, severity, payload->>'user_id_hash' AS user_id_hash, created_at
FROM audit_log
WHERE event_type = 'billing.access_revoked'
  AND created_at > :incident_start_time
ORDER BY created_at DESC;
```

2. `billing.consistency_check_failed` (CRITICAL) — emitted by the enforcer cron or daily check that detected the overshoot.
3. `system.cron_job_stale` (HIGH) — if the enforcer cron was found to have failed or missed runs.

**Evidence checklist:**
- [ ] Terminal output: scope query (Step 1) showing affected user count and overshoot hours
- [ ] Terminal output: verification query (Step 3) returning 0
- [ ] Terminal output: cron job history (Step 4) showing run status
- [ ] DEC-030 `billing.access_revoked` events for each affected user (confirm count matches Step 1 result)
- [ ] PagerDuty alert body (save as JSON)

---

## 8. Post-Incident Checklist

- [ ] Root cause documented: why did `grace-period-enforcer` not revoke access at the 72h mark?
- [ ] If cron job failed: platform-engineer fixes and tests the enforcer; redeploys pg_cron job
- [ ] If RevenueCat upstream event was delayed: confirm idempotency key was processed correctly once event arrived; no duplicate access granted
- [ ] SOC 2 PI1.5 evidence: confirm `billing.access_revoked` CRITICAL events are present in `audit_log` — this is the PI1.5 enforcement evidence artefact
- [ ] If > 5 users affected: post-mortem required within 48 h; owner: billing-owner; template: docs/INCIDENT_RESPONSE.md §IR-1
- [ ] Tune `grace-period-enforcer` if it missed the window due to a logic issue
- [ ] Confirm next enforcer run (T+30 min from resolution) returns zero overshoot rows

---

## 9. SOC 2 TSC Cross-Reference

| TSC | How this runbook satisfies it |
|---|---|
| PI1.5 | Grace period enforcement is a validity control; overshoot detection and revocation is the remediation path; `billing.access_revoked` CRITICAL event is the PI1.5 audit artefact |
| CC7.2 | AL-SUB-02 real-time detection via 30-min pg_cron enforcer; DEC-030 CRITICAL events on detection and remediation |
| CC6.1 | Access revocation procedure ensures only entitled users retain premium features; append-only correction event preserves audit trail |

---

*v1.0 · 2026-06-08 · billing-owner + platform-engineer*
