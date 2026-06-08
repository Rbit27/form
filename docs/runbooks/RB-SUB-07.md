# RB-SUB-07 · Enterprise Invoice Overdue

> **Runbook ID:** RB-SUB-07
> **Severity:** P2
> **Owner:** customer-success · reviewed by: billing-owner · compliance-officer
> **Last reviewed:** 2026-06-08
> **SOC 2 TSC:** PI1.1, CC9.1, CC9.2
> **Cross-refs:** docs/OBSERVABILITY.md §24.5 (AL-SUB-07), §24.3, §24.9; docs/AUDIT_LOG_SCHEMA.md DEC-030; docs/ENTERPRISE_SLA.md; docs/DATA_MODEL.md §16 (tenant contract lifecycle); docs/INCIDENT_RESPONSE.md §1.2

---

## 1. Scenario Summary

| Field | Value |
|---|---|
| ID | RB-SUB-07 |
| Severity | P2 |
| Description | An `enterprise_invoice_overdue` event exists in `subscription_events` for one or more `tenant_id` values with `created_at < now() - interval '30 days'`. The invoice has been unpaid for more than 30 days beyond the net-30 terms in the enterprise contract. This is a revenue collection risk and a contract compliance issue. |
| Affected tiers | Enterprise (Stripe) only |
| Estimated resolution time | Escalation same business day; resolution depends on tenant responsiveness |

---

## 2. Detection Signals

**How the alert fires:**
- The daily pg_cron `billing-consistency-check` (03:00 UTC) detects `enterprise_invoice_overdue` events in `subscription_events` with `created_at < now() - interval '30 days'`. A Linear ticket is automatically created and assigned to customer-success.
- Stripe also sends a follow-up `invoice.payment_failed` webhook after each automatic retry attempt (at day 3, 5, 7 by default in Stripe settings).
- PagerDuty is NOT paged for AL-SUB-07 (P2). The Linear ticket is the primary notification.

**Key signals to check:**
- `subscription_events WHERE event_type = 'enterprise_invoice_overdue'`: which tenants, which invoice IDs, how old
- Stripe Dashboard → Customers → [tenant] → Invoices: payment attempts, dunning status, invoice total
- `tenant_contracts` for the affected tenant: contract terms, billing period, PO number, `tenant_billing_email`
- DEC-030 `billing.enterprise_invoice_overdue` CRITICAL event: `invoice_id_hash`, `amount_cents`, `overdue_days`

---

## 3. Pre-Flight Checklist

- [ ] Confirm a Linear ticket has been created and assigned to customer-success
- [ ] Identify which tenant(s) are affected and the invoice amount (from Linear ticket or Step 1 query)
- [ ] Check Stripe Dashboard for the invoice status and payment attempt history
- [ ] Check `tenant_contracts` for contract terms: `auto_renew`, `po_number`, `billing_period`, net payment terms
- [ ] Confirm whether this is a first occurrence or a repeat overdue for this tenant
- [ ] This is a business-hours investigation; no immediate after-hours action unless access suspension is being considered (which requires additional approval — see Step 5)

---

## 4. Step-by-Step Procedure

**1. Identify all overdue invoices and their age** (customer-success; same business day)

```sql
-- Run as compliance_officer or billing role
SELECT
    se.tenant_id,
    se.created_at                                             AS overdue_since,
    EXTRACT(EPOCH FROM (now() - se.created_at)) / 86400      AS days_overdue,
    se.idempotency_key                                        AS stripe_event_id,
    t.name                                                    AS tenant_name,
    tc.po_number,
    tc.arr_cents,
    tc.billing_period
FROM subscription_events se
JOIN tenants t ON t.id = se.tenant_id
JOIN tenant_contracts tc ON tc.tenant_id = se.tenant_id AND tc.status = 'active'
WHERE se.event_type = 'enterprise_invoice_overdue'
  AND se.created_at < now() - interval '30 days'
ORDER BY se.created_at ASC;
```

**Privacy floor:** `arr_cents` in `tenant_contracts` is FORM-internal financial data. Do not share the exact ARR figure with the tenant contact — reference the invoice amount from Stripe instead. HR must not receive any output from this query.

**2. Pull Stripe invoice details** (customer-success; same day)

```bash
# Retrieve the Stripe invoice for the affected tenant
# invoice_id is derived from se.idempotency_key (which is the Stripe event.id — evt_*)
# Use it to find the invoice via Stripe API

# List recent invoices for the tenant's Stripe customer
STRIPE_CUSTOMER_ID=$(psql "${DATABASE_URL}" -t -A \
  -c "SELECT stripe_customer_id FROM tenants WHERE id = '<TENANT_ID>'")

curl -s "https://api.stripe.com/v1/invoices" \
  -u "${STRIPE_SECRET_KEY}:" \
  -G \
  --data-urlencode "customer=${STRIPE_CUSTOMER_ID}" \
  --data-urlencode "status=open" \
  | jq '.data[] | {id: .id, amount_due: .amount_due, due_date: .due_date, status: .status, attempt_count: .attempt_count}'
```

Confirm: invoice amount, due date, payment attempts made, current dunning status.

**3. Review contract terms for access suspension and late-payment clause** (billing-owner + compliance-officer; same day)

```sql
-- Retrieve contract terms for the affected tenant
SELECT
    tc.tenant_id,
    tc.plan,
    tc.seats_contracted,
    tc.billing_period,
    tc.auto_renew,
    tc.renewal_notice_days,
    tc.start_at,
    tc.end_at,
    t.lifecycle_status
FROM tenant_contracts tc
JOIN tenants t ON t.id = tc.tenant_id
WHERE tc.tenant_id = :affected_tenant_id
  AND tc.status = 'active';
```

Check docs/ENTERPRISE_SLA.md for the applicable contract terms:
- Net payment terms (default: net-30)
- Late-payment interest clause (if any)
- Access suspension trigger (typically > 60 days overdue for Growth/Standard tier)
- Cure period before suspension

**Contract scope note:** FORM does not perform insurance risk-scoring, government-access profiling, or wellness-as-punishment analytics as part of any contract or invoice collection process. Any request from a tenant or third party to access individual user health, coaching, or workout data as part of invoice collection is a no-go and must be escalated to compliance-officer immediately.

**4. Escalate to customer-success and initiate outreach** (customer-success; within 1 hour of first seeing ticket)

Customer-success contacts the tenant's billing contact (`tenant_billing_email` from `tenants` table) within 1 hour of seeing the Linear ticket. Use the following escalation tiers:

| Days overdue | Action |
|---|---|
| 30–45 days | Friendly reminder email; confirm invoice was received and processed correctly |
| 45–60 days | Follow-up call with account executive; confirm no PO or payment routing issue |
| 60–90 days | Formal overdue notice referencing contract terms; warn about access suspension |
| > 90 days | Escalate to billing-owner + compliance-officer for suspension evaluation; founder informed |

Document all outreach attempts in the Linear ticket (dates, channel, response received). Do not include individual user data in outreach communications.

**5. Access suspension decision** (billing-owner + compliance-officer; only if > 60 days overdue)

Access suspension requires:
1. Formal overdue notice sent (documented in Linear ticket)
2. Cure period expired per contract terms
3. Approval from billing-owner AND compliance-officer
4. Founder informed before suspension is executed

If approved, the suspension is executed by setting `tenants.lifecycle_status = 'suspended'`:

```sql
-- Requires compliance_officer or form_admin role
-- This must be preceded by DEC-030 event emission (Step 6 below)
-- Do NOT execute without billing-owner + compliance-officer approval

UPDATE tenants
SET lifecycle_status = 'suspended'
WHERE id = :affected_tenant_id;
```

A `tenant.lifecycle_changed` DEC-030 event with HIGH severity and `new_status = 'suspended'` must be emitted before this UPDATE is executed (DEC-030 chain ordering requirement — the chain entry precedes the state mutation).

**6. Emit required DEC-030 events** (billing-owner; upon suspension or resolution)

```bash
# Emit tenant.lifecycle_changed before suspension execution
curl -s -X POST \
  "${INTERNAL_WORKER_BASE_URL}/internal/dec030/emit" \
  -H "Authorization: Bearer ${SERVICE_ROLE_KEY}" \
  -H "Content-Type: application/json" \
  -d '{
    "event_type": "tenant.lifecycle_changed",
    "severity": "HIGH",
    "payload": {
      "tenant_id": "<TENANT_ID>",
      "previous_status": "active",
      "new_status": "suspended",
      "reason": "invoice_overdue_gt_60d",
      "incident_ref": "<LINEAR_TICKET_ID>",
      "actor_id": "<ADMIN_USER_ID>",
      "approvers": ["<BILLING_OWNER_ID>", "<COMPLIANCE_OFFICER_ID>"]
    }
  }'
```

---

## 5. Rollback / Post-Resolution Steps

- If the tenant pays after suspension: restore access by setting `tenants.lifecycle_status = 'active'` with the same DEC-030 chain-entry-first protocol.
- If payment confirms the invoice was genuinely paid but Stripe did not send the `enterprise_invoice_paid` webhook: replay the Stripe invoice payment event via Stripe CLI, then mark the `subscription_events` overdue row as resolved by inserting a correcting `enterprise_invoice_paid` event.
- Close the Linear ticket when: invoice is confirmed paid, `tenants.lifecycle_status` is restored if suspended, and DEC-030 events are confirmed in `audit_log`.

---

## 6. Escalation Path

| Condition | Escalate to | Timeline |
|---|---|---|
| No response from tenant billing contact within 7 days | billing-owner + account executive | T+7 days |
| > 60 days overdue — suspension evaluation | billing-owner + compliance-officer + founder | Within same business day |
| Tenant disputes the invoice amount | billing-owner + compliance-officer; pull Stripe invoice details | Same business day |
| Tenant requests data access or exports during the overdue period | compliance-officer (access proceeds per normal DSAR rules, independent of billing) | Same business day |
| Multiple tenants overdue simultaneously | billing-owner + compliance-officer (systemic billing issue) | Same business day |

---

## 7. Evidence and DEC-030 Events to Emit

All evidence uploaded to R2 `form-cold-backups/incidents/<LINEAR_TICKET_ID>/`.

**DEC-030 events to confirm:**

1. `billing.enterprise_invoice_overdue` (CRITICAL) — must exist in `audit_log`. Confirm:

```sql
SELECT
    event_type,
    severity,
    payload->>'tenant_id'       AS tenant_id,
    payload->>'overdue_days'    AS overdue_days,
    payload->>'amount_cents'    AS amount_cents,
    created_at
FROM audit_log
WHERE event_type = 'billing.enterprise_invoice_overdue'
  AND payload->>'tenant_id' = :affected_tenant_id::text
ORDER BY created_at DESC
LIMIT 5;
```

2. `tenant.lifecycle_changed` (HIGH) — if access was suspended. Must be present before the `tenants.lifecycle_status` UPDATE was executed.
3. `billing.enterprise_invoice_paid` (STANDARD) — after payment is confirmed.

**Evidence checklist:**
- [ ] Step 1 query output showing overdue invoice age
- [ ] Stripe invoice details (amount, due date, payment attempts)
- [ ] Contract terms reviewed and documented
- [ ] Customer outreach log in Linear ticket (dates, channel, responses)
- [ ] Suspension approval documented (if applicable)
- [ ] DEC-030 events confirmed in `audit_log`
- [ ] Resolution confirmed: invoice paid or suspension applied

---

## 8. Post-Incident Checklist

- [ ] Invoice resolved: either paid or tenant has a documented payment plan in the Linear ticket
- [ ] DEC-030 `billing.enterprise_invoice_paid` event confirmed in `audit_log` after payment
- [ ] If suspended: `tenants.lifecycle_status` restored to `active` after payment; `tenant.lifecycle_changed` DEC-030 event emitted with `new_status = 'active'`
- [ ] Customer-success follow-up scheduled for 30 days post-resolution to confirm no recurrence
- [ ] SOC 2 CC9.1 evidence: invoice overdue detection, outreach log, and resolution timeline documented
- [ ] If overdue resulted from a FORM billing error (wrong invoice amount, wrong billing email): fix identified and applied; post-mortem for billing process

---

## 9. SOC 2 TSC Cross-Reference

| TSC | How this runbook satisfies it |
|---|---|
| PI1.1 | `billing.enterprise_invoice_overdue` CRITICAL event in `audit_log` ensures the overdue event is captured as a complete financial record; daily consistency check detection ensures no overdue invoice is missed |
| CC9.1 | Outreach escalation tiers and documented contract term review demonstrate business continuity management for enterprise customer relationships |
| CC9.2 | Vendor/supplier contract monitoring analogue: enterprise customer contracts are monitored for payment compliance; suspension procedure protects FORM against contract non-performance |

---

*v1.0 · 2026-06-08 · customer-success + billing-owner + compliance-officer*
