# RB-SUB-01 · Subscription State Inconsistency

> **Runbook ID:** RB-SUB-01
> **Severity:** P0
> **Owner:** billing-owner · reviewed by: platform-engineer · compliance-officer
> **Last reviewed:** 2026-06-08
> **SOC 2 TSC:** CC7.2, PI1.2, PI1.5
> **Cross-refs:** docs/OBSERVABILITY.md §24.2, §24.5 (AL-SUB-01); docs/AUDIT_LOG_SCHEMA.md DEC-030; docs/DATA_MODEL.md §24.7; docs/INCIDENT_RESPONSE.md §1.2

---

## 1. Scenario Summary

| Field | Value |
|---|---|
| ID | RB-SUB-01 |
| Severity | P0 |
| Description | Impossible state transition detected in `subscription_events`, OR `users.subscription_tier` does not match the latest terminal `subscription_events` row for one or more `user_id` values. Either condition indicates the billing state machine has been bypassed, a webhook was processed incorrectly, or direct DB manipulation has occurred. |
| Affected tiers | Consumer (RevenueCat), Enterprise (Stripe) |
| Estimated resolution time | 30–90 min (root cause determines path) |

---

## 2. Detection Signals

**How the alert fires:**
- **Real-time path:** The webhook receiver (`workers/billing/webhook-receiver.ts`) validates every `(from_state, to_state)` pair against the valid-transitions table in §24.2 before persisting the `subscription_events` row. On an invalid pair, the worker emits `billing.subscription_state_changed` with `invalid_transition = true` (DEC-030 CRITICAL) and does NOT update `users.subscription_tier`. PagerDuty P0 fires within seconds.
- **Consistency-check path:** The daily pg_cron `billing-consistency-check` (03:00 UTC, §24.9 Check A/B) detects users where `users.subscription_tier` does not match the terminal state inferred from `subscription_events`. This path catches manual DB edits that bypassed the worker pipeline.
- **Signature-failure escalation:** If `billing.webhook.signature_invalid` > 10 events in any 5-minute window, AL-SUB-01 fires with additional routing to security-engineer.

**Key metrics/signals to check:**
- Grafana / Cloudflare Analytics Engine: `billing.webhook.signature_invalid` counter spike
- Metabase "Subscription Health" panel 2 (state distribution) — unexpected `invalid_transition_detected` rows
- PagerDuty incident body: includes `user_id_hash`, `from_state`, `to_state`, `idempotency_key`, `source`

---

## 3. Pre-Flight Checklist

- [ ] Confirm the PagerDuty alert is not a drill (`alert_body.is_drill = false`)
- [ ] Open a Linear incident ticket — P0, tag `billing-state-machine`, link to PagerDuty alert
- [ ] Page billing-owner immediately; page security-engineer if `billing.webhook.signature_invalid` is elevated
- [ ] Retrieve `user_id_hash` and `idempotency_key` from the alert body — these identify the affected event(s)
- [ ] Check whether the alert is an isolated event or a pattern (run query in §5 Step 1)
- [ ] Confirm RevenueCat / Stripe status pages are clean:
  - https://status.revenuecat.com
  - https://status.stripe.com
- [ ] Check Cloudflare Workers status: https://www.cloudflarestatus.com

---

## 4. Step-by-Step Procedure

**1. Determine scope — isolated event vs. mass inconsistency** (on-call; T+0)

```sql
-- Run as compliance_officer or form_system role
-- Count invalid transitions in last 2 hours
SELECT
    COUNT(*)                      AS invalid_count,
    MIN(created_at)               AS first_seen,
    MAX(created_at)               AS last_seen,
    array_agg(DISTINCT source)    AS sources,
    array_agg(DISTINCT from_state || '->' || to_state) AS transitions
FROM subscription_events
WHERE event_type = 'invalid_transition_detected'
  AND created_at > now() - interval '2 hours';
```

- **Single isolated event:** Proceed to Step 2 (identify root cause).
- **Multiple events / pattern:** Escalate to billing-owner + platform-engineer immediately; treat as possible systemic failure. Check if `billing.webhook.signature_invalid` counter is elevated (possible spoofing/replay attack — escalate to security-engineer).

**2. Identify the affected user(s) and their current state** (on-call; T+5)

```sql
-- Retrieve the inconsistent event(s)
-- Use user_id_hash from alert body to locate the event
SELECT
    se.id,
    se.user_id,
    se.tenant_id,
    se.event_type,
    se.source,
    se.from_state,
    se.to_state,
    se.subscription_tier,
    se.valid_from,
    se.valid_to,
    se.idempotency_key,
    se.created_at
FROM subscription_events se
WHERE se.event_type = 'invalid_transition_detected'
  AND se.created_at > now() - interval '24 hours'
ORDER BY se.created_at DESC
LIMIT 20;
```

```sql
-- Compare users.subscription_tier against latest subscription_events terminal state
-- for any user with a recent invalid_transition_detected event
SELECT
    u.id                          AS user_id,
    u.subscription_tier           AS users_tier,
    se_latest.to_state            AS latest_event_state,
    se_latest.event_type          AS latest_event_type,
    se_latest.created_at          AS latest_event_at
FROM users u
JOIN LATERAL (
    SELECT to_state, event_type, created_at
    FROM subscription_events
    WHERE user_id = u.id
    ORDER BY created_at DESC
    LIMIT 1
) se_latest ON true
WHERE u.subscription_tier != CASE se_latest.to_state
    WHEN 'active'   THEN 'premium'
    WHEN 'trialing' THEN 'premium'
    WHEN 'past_due' THEN 'premium'
    WHEN 'canceled' THEN 'free'
    WHEN 'expired'  THEN 'free'
    ELSE 'free'
END
  AND u.id IN (
      SELECT DISTINCT user_id
      FROM subscription_events
      WHERE event_type = 'invalid_transition_detected'
        AND created_at > now() - interval '24 hours'
  );
```

**3. Determine root cause** (on-call + billing-owner; T+10)

Review the `source` column of the flagged rows:

| Source | Likely cause |
|---|---|
| `revenuecat` | RevenueCat delivered a duplicate/out-of-order notification; check RevenueCat dashboard for replay |
| `stripe` | Stripe webhook replay or event misrouting; check Stripe Dashboard → Webhooks → event log |
| `manual_admin` | Admin performed a direct state change bypassing validation; review `actor_id` in the event row |

Check for webhook signature failures in the last hour:
```bash
# Query Cloudflare Analytics Engine via API
curl -s "https://api.cloudflare.com/client/v4/accounts/${CF_ACCOUNT_ID}/analytics_engine/sql" \
  -H "Authorization: Bearer ${CF_API_TOKEN}" \
  -d "SELECT sum(1) as count FROM METRICSDATA WHERE index1 = 'billing.webhook.signature_invalid' AND timestamp > NOW() - INTERVAL '1' HOUR"
```

**4a. Remediation — state inconsistency (users.subscription_tier wrong)** (billing-owner; T+20)

Do NOT directly UPDATE `users.subscription_tier`. The correct fix is to insert a compensating event into `subscription_events` with `event_type = 'correction'`, `source = 'manual_admin'`, and `actor_id` set to the admin's user ID. The billing worker's re-processing or admin correction script then updates `users.subscription_tier`.

```sql
-- Verify the correct terminal state before inserting a correction
-- ONLY execute after billing-owner has reviewed and confirmed the correct state
-- This INSERT requires form_system or compliance_officer role

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
    :tenant_id_or_null,
    'correction',
    'manual_admin',
    :incorrect_state,
    :correct_state,
    encode(digest('manual-correction-' || :incident_linear_id, 'sha256'), 'hex'),
    :correct_tier,
    now(),
    NULL,
    'correction-' || :incident_linear_id,
    :admin_actor_id,
    0
);
```

Then update `users.subscription_tier` to match (requires form_system role or direct admin path):
```sql
UPDATE users
SET subscription_tier = :correct_tier
WHERE id = :affected_user_id;
```

**4b. Remediation — signature-based attack vector** (security-engineer; T+20)

If `billing.webhook_signature_invalid` count > 10 in 5 minutes, rotate webhook secrets immediately:

```bash
# Rotate RevenueCat webhook auth key via RevenueCat Dashboard, then update Workers Secret:
wrangler secret put REVENUECAT_WEBHOOK_AUTH_KEY --env production

# Rotate Stripe webhook signing secret via Stripe Dashboard → Webhooks, then:
wrangler secret put STRIPE_WEBHOOK_SIGNING_SECRET --env production

# Redeploy Workers to pick up new secrets
wrangler deploy --env production
```

**5. Verify HMAC chain integrity after any correction** (billing-owner; T+30)

```sql
-- Check that the chain is intact after the correction INSERT
-- Any mismatch → escalate to compliance-officer immediately
SELECT
    id,
    event_type,
    occurred_at,
    hmac_self IS NOT NULL AS has_hmac
FROM audit_log
WHERE event_type LIKE 'billing.%'
ORDER BY occurred_at DESC
LIMIT 30;
```

Confirm the DEC-030 `billing.subscription_state_changed` event for the correction is present in `audit_log` with `invalid_transition = true` on the original event and `actor_id` set on the correction event.

**6. Mark the seat-reconciliation / consistency check as resolved** (on-call; T+45)

If a `billing_consistency_checks` failure row exists for today's run:
```sql
-- Verify the next daily run will pass by re-running the relevant check manually
-- Check A (active subscriber entitlement coverage):
SELECT COUNT(*) AS uncovered_active_subscribers
FROM users u
WHERE u.subscription_tier IN ('premium', 'enterprise')
  AND NOT EXISTS (
      SELECT 1 FROM subscription_events se
      WHERE se.user_id = u.id
        AND se.to_state = 'active'
        AND (se.valid_to IS NULL OR se.valid_to > now())
  );
-- Expected result: 0
```

---

## 5. Rollback / Post-Resolution Steps

- If the correction INSERT causes a downstream issue (e.g., a second webhook arrives for the same `idempotency_key`), insert another `correction` event to return to the pre-correction state. Do NOT DELETE from `subscription_events` — it is append-only.
- Resolve the PagerDuty alert only after Step 6 verification query returns 0.
- Update the Linear incident ticket: root cause, affected `user_id_hash` count, correction events inserted.

---

## 6. Escalation Path

| Condition | Escalate to | Timeline |
|---|---|---|
| On-call cannot determine root cause within 30 min | billing-owner + platform-engineer | T+30 |
| Signature-failure count > 10 in 5 min | security-engineer (in addition to above) | Immediately |
| Correction INSERT fails or HMAC chain breaks | compliance-officer + platform-engineer | Immediately |
| > 5 affected users or enterprise tenant involved | billing-owner + founder | T+15 |
| Manual DB manipulation suspected (`source = 'manual_admin'` without valid `actor_id`) | security-engineer + compliance-officer | Immediately |

---

## 7. Evidence and DEC-030 Events to Emit

All evidence uploaded to R2 `form-cold-backups/incidents/<LINEAR_TICKET_ID>/`.

**DEC-030 events that must exist in `audit_log` before this incident is closed:**

1. `billing.subscription_state_changed` with `invalid_transition = true` — emitted automatically by webhook receiver on detection. Confirm it exists.
2. `billing.consistency_check_failed` — emitted by daily pg_cron if the mismatch was caught there. Confirm payload lists the affected check name.
3. For correction path: `billing.subscription_state_changed` with `event_type = 'correction'` and `actor_id` set to the admin who performed the correction.
4. If secrets were rotated: `system.credential_rotated` for each rotated secret (emitted by rotation procedure).

```sql
-- Verify required audit events exist
SELECT event_type, severity, created_at, payload
FROM audit_log
WHERE event_type IN (
    'billing.subscription_state_changed',
    'billing.consistency_check_failed',
    'system.credential_rotated'
)
  AND created_at > :incident_start_time
ORDER BY created_at DESC;
```

**Evidence checklist:**
- [ ] Terminal output: scope-check query (Step 1) showing count and affected transitions
- [ ] Terminal output: correction INSERT query output (if applied)
- [ ] Terminal output: HMAC chain verification query (Step 5) — no missing HMACs
- [ ] Terminal output: post-correction Check A result = 0
- [ ] Screenshot: Stripe / RevenueCat webhook event log at time of incident
- [ ] PagerDuty alert body (save as JSON)

---

## 8. Post-Incident Checklist

- [ ] Root cause documented in Linear incident ticket
- [ ] If webhook replay: confirm idempotency key uniqueness constraint blocked the duplicate (verify via `billing.webhook.processed{outcome="idempotent"}` counter)
- [ ] If manual DB manipulation: access review initiated; access log exported from `audit_log WHERE actor_id = :suspect_actor_id`
- [ ] If systemic (> 1 user): platform-engineer reviews state machine validator logic for edge cases
- [ ] SOC 2 PI1.5 evidence: confirm `billing.subscription_state_changed` CRITICAL event with `invalid_transition = true` is in `audit_log` — this is the PI1.5 evidence artefact
- [ ] Post-mortem required if: > 5 users affected, enterprise tenant impacted, or secrets were rotated
- [ ] Post-mortem due: within 48 h of resolution; owner: billing-owner; template: docs/INCIDENT_RESPONSE.md §IR-1

---

## 9. SOC 2 TSC Cross-Reference

| TSC | How this runbook satisfies it |
|---|---|
| CC7.2 | AL-SUB-01 real-time invalid-transition detection; daily consistency check; both paths emit DEC-030 CRITICAL events |
| PI1.2 | State machine validator blocks impossible transitions before they reach `users.subscription_tier`; correction path is auditable via `actor_id` |
| PI1.5 | Invalid transitions are rejected at write time; any bypass is detected by daily consistency check; correction is append-only via compensating event |

---

*v1.0 · 2026-06-08 · billing-owner + platform-engineer*
