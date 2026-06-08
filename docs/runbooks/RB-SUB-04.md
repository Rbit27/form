# RB-SUB-04 · Enterprise Seat Drift

> **Runbook ID:** RB-SUB-04
> **Severity:** P1
> **Owner:** billing-owner · reviewed by: platform-engineer · customer-success · compliance-officer
> **Last reviewed:** 2026-06-08
> **SOC 2 TSC:** PI1.2, CC7.2, CC6.1, CC6.2
> **Cross-refs:** docs/OBSERVABILITY.md §24.5 (AL-SUB-04), §24.8; docs/AUDIT_LOG_SCHEMA.md DEC-030; docs/DATA_MODEL.md §24.7, §24.8; docs/ENTERPRISE_SLA.md; docs/INCIDENT_RESPONSE.md §1.2

---

## 1. Scenario Summary

| Field | Value |
|---|---|
| ID | RB-SUB-04 |
| Severity | P1 |
| Description | `tenant_seats.seat_count` (FORM's local count) does not match `subscription.quantity` on the authoritative Stripe subscription object for one or more enterprise `tenant_id` values, persisting for more than 5 minutes. Any persistent delta means either FORM is licensing more seats than the customer contracted (revenue loss) or fewer seats than the customer paid for (access denial). |
| Affected tiers | Enterprise (Stripe) only |
| Estimated resolution time | 20–60 min |

---

## 2. Detection Signals

**How the alert fires:**
- The pg_cron job `seat-reconciliation` runs every 5 minutes. It compares `tenant_seats.seat_count` against the Stripe `subscription.quantity` via a Worker proxy (`GET /internal/stripe/seat-quantities`). Drift rows are inserted into `seat_reconciliation_log` with `resolved = false`. If any row has `checked_at < now() - interval '5 minutes'` and `resolved = false`, the cron emits `billing.seat_drift_detected` (DEC-030 CRITICAL) and triggers PagerDuty P1.
- Metabase "Subscription Health" panel 5 (Enterprise Seat Drift Count) shows count > 0.
- PagerDuty alert body includes `tenant_id`, `local_count`, `stripe_count`, `delta`, `drift_duration_minutes`.

**Key signals to check:**
- `seat_reconciliation_log WHERE resolved = false`: full list of drifting tenants
- Stripe Dashboard → Customers → [tenant's Stripe customer] → Subscriptions: authoritative seat count
- Recent `subscription_events` for the affected `tenant_id`: look for `enterprise_seat_added` or `enterprise_seat_removed` events that may have failed to update `tenant_seats`
- `cron.job_run_details` for `seat-reconciliation`: confirm the job is running on schedule

---

## 3. Pre-Flight Checklist

- [ ] Open a Linear incident ticket — P1, tag `enterprise-seat-drift`, link to PagerDuty alert
- [ ] Notify customer-success immediately — an enterprise tenant is potentially affected (Step 1 tells you which one)
- [ ] Retrieve `tenant_id`, `local_count`, `stripe_count`, `delta` from PagerDuty alert body
- [ ] Verify Stripe status: https://status.stripe.com (Stripe API outage can cause `seat-reconciliation` to fail silently)
- [ ] Confirm `seat-reconciliation` cron job is running: check `cron.job_run_details`
- [ ] Check whether the drift is FORM-under-counted (revenue loss risk) or FORM-over-counted (access denial risk)

---

## 4. Step-by-Step Procedure

**1. Identify all drifting tenants and the direction of drift** (on-call; T+0)

```sql
-- Run as compliance_officer or form_system role
SELECT
    srl.tenant_id,
    srl.local_count,
    srl.stripe_count,
    srl.delta,
    srl.checked_at,
    EXTRACT(EPOCH FROM (now() - srl.checked_at)) / 60 AS drift_age_minutes,
    t.name AS tenant_name  -- for internal reference; do not log externally
FROM seat_reconciliation_log srl
JOIN tenants t ON t.id = srl.tenant_id
WHERE srl.resolved = false
ORDER BY srl.checked_at ASC;
```

Classify each drifting tenant:

| Condition | Risk |
|---|---|
| `local_count > stripe_count` | FORM is licensing more seats than Stripe says were paid for — revenue loss. Stripe is authoritative. |
| `local_count < stripe_count` | FORM is denying access to seats the customer paid for — access denial. Immediate remediation required. |

**2. Pull the authoritative seat count directly from Stripe** (billing-owner; T+5)

```bash
# For each affected tenant_id, retrieve the Stripe subscription
# STRIPE_SUBSCRIPTION_ID is stored in tenant_contracts.stripe_subscription_id
STRIPE_SUB_ID=$(psql "${DATABASE_URL}" -t -A \
  -c "SELECT stripe_subscription_id FROM tenant_contracts WHERE tenant_id = '<TENANT_ID>' AND status = 'active'")

curl -s "https://api.stripe.com/v1/subscriptions/${STRIPE_SUB_ID}" \
  -u "${STRIPE_SECRET_KEY}:" \
  | jq '{id: .id, quantity: .quantity, status: .status, current_period_end: .current_period_end}'
```

Record the Stripe `quantity`. This is the authoritative value.

**3. Check for unprocessed seat-change webhook events** (platform-engineer; T+10)

```sql
-- Look for seat-change events in subscription_events for the affected tenant
-- in the time window leading up to the drift detection
SELECT
    se.id,
    se.event_type,
    se.source,
    se.from_state,
    se.to_state,
    se.created_at,
    se.processed_at,
    se.idempotency_key
FROM subscription_events se
WHERE se.tenant_id = :affected_tenant_id
  AND se.event_type IN ('enterprise_seat_added', 'enterprise_seat_removed', 'subscription_created')
  AND se.created_at > now() - interval '24 hours'
ORDER BY se.created_at DESC;
```

If seat-change events exist with `to_state` mismatch vs. `tenant_seats.seat_count`: the `tenant_seats` update failed or was not applied after the event was written. Proceed to Step 4a.

If no seat-change events exist but Stripe shows a different quantity: a webhook was missed or failed. Proceed to Step 4b.

**4a. Remediation — seat_seats table not updated after valid seat-change event** (billing-owner; T+20)

Update `tenant_seats.seat_count` to match the authoritative Stripe value and insert a correcting `subscription_events` row:

```sql
BEGIN;

-- Update local seat count to match Stripe authoritative count
UPDATE tenant_seats
SET seat_count = :stripe_quantity
WHERE tenant_id = :affected_tenant_id
  AND subscription_tier = 'enterprise';

-- Insert a correcting subscription_events row for audit trail
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
    NULL,  -- seat changes are tenant-level, no single user_id
    :affected_tenant_id,
    'correction',
    'manual_admin',
    'active',
    'active',
    encode(digest('seat-drift-correction-' || :incident_linear_id, 'sha256'), 'hex'),
    'enterprise',
    now(),
    NULL,
    'seat-correction-' || :incident_linear_id,
    :admin_actor_id,
    0
);

COMMIT;
```

**Note:** `user_id` has a NOT NULL FK constraint on `subscription_events`. If the correction requires a `user_id`, use the tenant's primary admin user ID. If the table schema permits NULL for seat-level events (check the DDL in §24.7), use NULL. Confirm which is valid before executing.

**4b. Remediation — missing seat-change webhook** (platform-engineer; T+20)

If no `enterprise_seat_added` / `enterprise_seat_removed` event exists but Stripe shows a quantity change, replay the Stripe webhook event:

```bash
# Identify the missing event in Stripe Dashboard → Webhooks → Event log
# Filter by customer ID for the affected tenant, event type: customer.subscription.updated
# Replay the event:
stripe events resend <evt_STRIPE_EVENT_ID> --webhook-endpoint=<endpoint_id>
```

Confirm the replayed event is processed (check `subscription_events` for the new row and `tenant_seats.seat_count` for the update).

**5. Mark the drift as resolved and verify** (on-call; T+30)

```sql
-- Mark unresolved drift rows as resolved
UPDATE seat_reconciliation_log
SET
    resolved = true,
    resolved_at = now(),
    resolved_by = :admin_actor_id
WHERE tenant_id = :affected_tenant_id
  AND resolved = false;
```

```sql
-- Verify tenant_seats now matches Stripe count
SELECT
    ts.tenant_id,
    ts.seat_count AS local_count,
    :stripe_quantity AS stripe_count,
    ts.seat_count = :stripe_quantity AS is_resolved
FROM tenant_seats ts
WHERE ts.tenant_id = :affected_tenant_id
  AND ts.subscription_tier = 'enterprise';
-- is_resolved must be true
```

**6. Assess SLA credit applicability** (billing-owner + customer-success; T+30)

Review docs/ENTERPRISE_SLA.md to determine whether the drift caused a service impact qualifying for an SLA credit:

- If `local_count < stripe_count` (access denial): users were blocked from seats they paid for. Duration of access denial is the credit window. Calculate per docs/ENTERPRISE_SLA.md §3.4 credit schedule. If credit applies: emit `sla.credit_calculated` DEC-030 event and notify the tenant's billing contact.
- If `local_count > stripe_count` (revenue loss, no access denial): no credit applies to the tenant. Document internally.
- Customer-success notifies the tenant account manager if any user was denied access.

**Privacy floor:** Do not share individual user IDs, names, or session data with customer-success or the tenant contact. Share only: tenant name, drift duration, seat count discrepancy, whether access was affected.

---

## 5. Rollback / Post-Resolution Steps

- Do NOT delete from `seat_reconciliation_log` or `subscription_events`. Both are append-only for audit purposes.
- After correction: wait for the next `seat-reconciliation` cron run (within 5 min) and confirm no new drift rows are inserted for the affected tenant.
- Resolve PagerDuty after: `seat_reconciliation_log` has zero unresolved rows for the tenant AND the 5-min cron run passes cleanly.
- Update Linear ticket: tenant (by ID, not name), drift direction, duration, seat delta, root cause, credit decision.

---

## 6. Escalation Path

| Condition | Escalate to | Timeline |
|---|---|---|
| `seat-reconciliation` cron job has been failing | platform-engineer | Immediately |
| Access denial confirmed (users blocked from paid seats) | customer-success + billing-owner; initiate SLA credit review | Immediately |
| Drift duration > 30 min | billing-owner + customer-success | T+30 |
| Stripe API is returning errors for seat queries | devops-lead (Stripe status) | T+15 |
| Delta > 20 seats or > 10% of the tenant's seat count | billing-owner + enterprise-architect | Immediately |
| Multiple tenants drifting simultaneously | billing-owner + platform-engineer (systemic failure) | Immediately |

---

## 7. Evidence and DEC-030 Events to Emit

All evidence uploaded to R2 `form-cold-backups/incidents/<LINEAR_TICKET_ID>/`.

**DEC-030 events that must exist before closing:**

1. `billing.seat_drift_detected` (CRITICAL) — emitted automatically by `seat-reconciliation` cron. Confirm:

```sql
SELECT event_type, severity, payload, created_at
FROM audit_log
WHERE event_type = 'billing.seat_drift_detected'
  AND payload->>'tenant_id' = :affected_tenant_id::text
  AND created_at > :incident_start_time
ORDER BY created_at DESC
LIMIT 5;
```

2. `billing.seat_count_changed` (STANDARD) — should have been emitted when the seat-change event was originally processed. If missing (because the webhook was missed), confirm it is emitted after webhook replay.
3. If an SLA credit was calculated: `sla.credit_calculated` (HIGH) — emitted by the month-close Worker or manually if the credit must be applied out of cycle.

**Evidence checklist:**
- [ ] Terminal output: Step 1 query showing drifting tenants with counts and duration
- [ ] Stripe API output (Step 2) confirming authoritative seat count
- [ ] Terminal output: Step 3 query showing subscription_events for the drift window
- [ ] Terminal output: verification query (Step 5) showing `is_resolved = true`
- [ ] DEC-030 `billing.seat_drift_detected` event confirmed in `audit_log`
- [ ] SLA credit assessment documented in Linear ticket (even if credit does not apply)
- [ ] Customer-success notification sent (if access denial occurred)
- [ ] PagerDuty alert body (save as JSON)

---

## 8. Post-Incident Checklist

- [ ] Root cause documented: webhook missed, processing failure, or direct DB edit?
- [ ] If `seat-reconciliation` cron was failing: fix deployed and cron confirmed running
- [ ] If webhook missed: confirm Stripe webhook retry delivered and processed; confirm idempotency key in `subscription_events`
- [ ] SOC 2 PI1.2 evidence: `billing.seat_drift_detected` CRITICAL event and `seat_reconciliation_log` export are the PI1.2 artefacts
- [ ] SLA credit decision documented and applied to next invoice if applicable (per docs/ENTERPRISE_SLA.md)
- [ ] If > 5% seat delta or > 30 min drift: post-mortem required within 48 h; owner: billing-owner
- [ ] Confirm next 3 consecutive `seat-reconciliation` cron runs return zero drift for the affected tenant

---

## 9. SOC 2 TSC Cross-Reference

| TSC | How this runbook satisfies it |
|---|---|
| PI1.2 | Seat count accuracy is a processing-integrity control; `seat-reconciliation` cron and Stripe-as-authoritative-source enforce accuracy; `billing.seat_drift_detected` CRITICAL event is the PI1.2 audit artefact |
| CC7.2 | 5-min reconciliation with drift escalation detects seat count anomalies in near-real-time; DEC-030 CRITICAL event on persistent drift |
| CC6.1 | Enterprise seat provisioning is gated on `tenant_seats.seat_count`; drift correction ensures the access control gate reflects the contracted quantity |
| CC6.2 | `tenant_contracts.seats_contracted` is the authorization basis; seat count must reflect this; drift is a logical access control deviation |

---

*v1.0 · 2026-06-08 · billing-owner + platform-engineer + customer-success*
