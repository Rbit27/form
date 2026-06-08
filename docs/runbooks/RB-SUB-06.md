# RB-SUB-06 · Elevated Refund Rate

> **Runbook ID:** RB-SUB-06
> **Severity:** P2
> **Owner:** billing-owner · reviewed by: compliance-officer · fraud-review
> **Last reviewed:** 2026-06-08
> **SOC 2 TSC:** PI1.1, CC7.2
> **Cross-refs:** docs/OBSERVABILITY.md §24.5 (AL-SUB-06), §24.9; docs/AUDIT_LOG_SCHEMA.md DEC-030; docs/INCIDENT_RESPONSE.md §1.2

---

## 1. Scenario Summary

| Field | Value |
|---|---|
| ID | RB-SUB-06 |
| Severity | P2 |
| Description | Refund events in the trailing 7 days exceed 5% of renewal events in the same window. Elevated refund rates can indicate a product quality issue, a fraudulent refund campaign (chargeback fraud / friendly fraud), an onboarding or billing UX problem, or a downstream payment processor issue. P2 does not require immediate paging but requires same-business-day investigation. |
| Affected tiers | Consumer (RevenueCat primarily); Enterprise invoices may generate refunds via Stripe |
| Estimated resolution time | 1–4 h for investigation; remediation varies by root cause |

---

## 2. Detection Signals

**How the alert fires:**
- The daily pg_cron `billing-consistency-check` (03:00 UTC) calculates `refund_issued` count / `subscription_renewed` count over the trailing 7-day window from `subscription_events`. When this ratio exceeds 5%, a Linear ticket is automatically created and assigned to billing-owner and fraud-review.
- Metabase "Subscription Health" panel 7 (Refund Rate, rolling 7d): gauge in red band (> 5%).
- Linear ticket body includes: refund count, renewal count, refund rate, window start/end.

**Key signals to check:**
- `subscription_events WHERE event_type = 'refund_issued'` for the 7-day window: distribution by `source` (revenuecat vs. stripe) and over time
- RevenueCat Dashboard → Revenue → Refunds: breakdown by refund reason code
- Stripe Dashboard → Radar / Disputes: fraud signals, dispute rate
- PostHog: onboarding completion rate and session completion rate in the same 7-day window (product quality signal)
- `subscription_events WHERE event_type = 'subscription_renewed'`: confirm the denominator is accurate

---

## 3. Pre-Flight Checklist

- [ ] Confirm a Linear ticket has been created by the daily check
- [ ] Pull the exact refund count, renewal count, and rate from the Linear ticket or the query in Step 1
- [ ] Identify whether the rate is driven by a single day (spike) or a sustained trend (Step 1 query)
- [ ] Check RevenueCat Dashboard for refund reason breakdown
- [ ] Check Stripe Dashboard for dispute / chargeback rate (distinct from refund rate)
- [ ] No PagerDuty page is expected for P2 — this is business-hours investigation

---

## 4. Step-by-Step Procedure

**1. Quantify and characterise the elevated refunds** (billing-owner; same business day)

```sql
-- Run as compliance_officer or billing role
-- Breakdown of refunds by day and source over the trailing 7 days
SELECT
    date_trunc('day', created_at)   AS refund_day,
    source,
    COUNT(*)                        AS refund_count
FROM subscription_events
WHERE event_type = 'refund_issued'
  AND created_at > now() - interval '7 days'
GROUP BY 1, 2
ORDER BY 1 DESC, 2;
```

```sql
-- Renewal count in same window (denominator)
SELECT
    date_trunc('day', created_at)   AS renewal_day,
    source,
    COUNT(*)                        AS renewal_count
FROM subscription_events
WHERE event_type = 'subscription_renewed'
  AND created_at > now() - interval '7 days'
GROUP BY 1, 2
ORDER BY 1 DESC, 2;
```

Look for:
- Single-day spike (possible product issue, promo expiry, refund campaign)
- Gradual increase over multiple days (systemic issue, fraud campaign)
- RevenueCat vs. Stripe concentration (which channel is driving the rate)

**Privacy floor:** Do not join on `users.email` or any PII column. Work with `user_id` UUIDs and `user_id_hash` values from DEC-030 events. Do not share individual user data with fraud-review beyond UUIDs. HR must not receive any data from this investigation.

**2. Distinguish product-driven refunds from fraud-driven refunds** (billing-owner + fraud-review; same day)

| Signal | Interpretation |
|---|---|
| Refund reason codes clustered on "not as described" or "dissatisfied" | Product quality issue — investigate PostHog onboarding/session funnel |
| Refund reason "duplicate charge" | Billing bug — check for double-charge in RevenueCat/Stripe event log |
| High dispute rate in Stripe Radar alongside refunds | Friendly fraud / chargeback campaign — escalate to fraud-review immediately |
| Refunds concentrated in a narrow user cohort (same acquisition channel or signup date) | Onboarding or promo issue — investigate the specific cohort |
| Refunds evenly distributed across all users | Broad product dissatisfaction — escalate to product owner |

```bash
# Check Stripe Radar dispute rate via Stripe CLI
stripe balance_transactions list \
  --type=payment \
  --created[gt]=$(date -d '7 days ago' +%s) \
  --limit 100 \
  | jq '[.[] | select(.type == "dispute")] | length'
```

**3. If fraud-driven: initiate fraud-review investigation** (fraud-review; same day)

Fraud-review receives the following data (no individual user PII beyond UUIDs):
- `user_id` UUIDs of refunded users (from `subscription_events.user_id` — UUID only, no name or email)
- Refund timestamps
- `source` (revenuecat or stripe)
- Idempotency keys (for cross-referencing with RevenueCat/Stripe dashboards)

Fraud-review accesses RevenueCat and Stripe dashboards directly to obtain IP data, device fingerprints, and refund reason codes. This data is not pulled into FORM's Supabase database.

**No-go constraints:** FORM does not perform insurance risk-scoring, government-backdoor analysis, or wellness-as-punishment profiling. Fraud review is limited to identifying refund pattern anomalies to protect revenue integrity. Any proposed fraud-review methodology that would score users by health data, coaching content, or workout frequency is prohibited per docs/ENTERPRISE_SLA.md §13 (Privacy Floor) and must be escalated to compliance-officer for review.

**4. If product-driven: identify the failing funnel step** (billing-owner + product; same day)

```bash
# PostHog: query drop-off in the F8 (paywall/upgrade) flow and post-purchase onboarding
# Via PostHog API or PostHog dashboard → Funnels → F8 flow → 7-day window
curl -s "https://eu.posthog.com/api/projects/${POSTHOG_PROJECT_ID}/insights/funnel/" \
  -H "Authorization: Bearer ${POSTHOG_PERSONAL_API_KEY}" \
  -H "Content-Type: application/json" \
  -d '{"date_from": "-7d", "events": [{"id": "paywall.shown"}, {"id": "subscription.purchase_started"}, {"id": "onboarding.completed"}]}'
```

If onboarding completion rate is down during the refund window: escalate to product owner. A product issue driving refunds does not require fraud-review involvement.

**5. Remediation actions by root cause** (billing-owner; same/next business day)

| Root cause | Action |
|---|---|
| Product quality / onboarding issue | Create Linear bug ticket assigned to product owner; no billing action required |
| Billing bug (duplicate charge) | Identify affected `idempotency_key` values; confirm correct refunds were issued; fix the billing code causing the duplicate |
| Fraud campaign | fraud-review initiates appropriate controls in RevenueCat/Stripe (e.g., block specific payment methods or regions) |
| Promo / offer expiry driving "dissatisfied" refunds | Marketing / product review of offer terms |

---

## 5. Rollback / Post-Resolution Steps

- No data rollback required — `subscription_events` rows for refunds are already written and are correct.
- Refund events in `subscription_events` must NOT be deleted. They are append-only financial records.
- Close the Linear ticket when: root cause identified, corrective action documented, and rate is confirmed below 5% over the subsequent 7-day window.
- If fraud controls were applied in RevenueCat/Stripe: document the change in the Linear ticket (date, control type, applied by).

---

## 6. Escalation Path

| Condition | Escalate to | Timeline |
|---|---|---|
| Fraud-review confirms a coordinated fraud campaign | billing-owner + compliance-officer + security-engineer | Same business day |
| Refund rate > 15% | billing-owner + founder | Same business day |
| Evidence of a billing bug causing duplicate charges | platform-engineer; stop the charge if ongoing | Immediately (P1 uplift) |
| An enterprise tenant is involved in refund activity | customer-success + billing-owner | Same business day |
| Chargeback / dispute rate in Stripe > 0.5% | fraud-review + billing-owner (Stripe dispute risk threshold) | Same business day |

---

## 7. Evidence and DEC-030 Events to Emit

All evidence uploaded to R2 `form-cold-backups/incidents/<LINEAR_TICKET_ID>/`.

**DEC-030 events to confirm:**

1. `billing.refund_issued` (HIGH) — must exist for each refund event. Confirm the count in `audit_log` matches the count in `subscription_events` for the 7-day window:

```sql
SELECT COUNT(*) AS audit_log_refund_count
FROM audit_log
WHERE event_type = 'billing.refund_issued'
  AND created_at > now() - interval '7 days';

SELECT COUNT(*) AS subscription_events_refund_count
FROM subscription_events
WHERE event_type = 'refund_issued'
  AND created_at > now() - interval '7 days';
-- Both counts must match
```

2. `billing.consistency_check_failed` (CRITICAL) — only if the daily check determined the refund rate triggered a broader consistency failure; otherwise not required.

**Evidence checklist:**
- [ ] Step 1 query output: refund and renewal counts by day and source
- [ ] Root cause classification (product / fraud / billing bug)
- [ ] Fraud-review engagement confirmed if fraud-driven
- [ ] PostHog funnel data if product-driven
- [ ] Corrective action documented in Linear ticket
- [ ] `billing.refund_issued` DEC-030 event count verified

---

## 8. Post-Incident Checklist

- [ ] Rate returned to < 5% over the next 7-day window (confirm via Metabase panel 7)
- [ ] Root cause documented and closed in Linear
- [ ] If billing bug: code fix deployed and tested; no further duplicate charges occurring
- [ ] If fraud controls applied: document control type and date; schedule a 30-day review
- [ ] Privacy floor confirmed: no individual user health data, names, or emails shared with fraud-review

---

## 9. SOC 2 TSC Cross-Reference

| TSC | How this runbook satisfies it |
|---|---|
| PI1.1 | Refund event tracking ensures completeness of the financial event record; DEC-030 `billing.refund_issued` HIGH events must match `subscription_events` row count |
| CC7.2 | Daily consistency check detects revenue anomalies (elevated refund rate); Linear ticket creation triggers investigation before the rate causes payment processor risk |

---

*v1.0 · 2026-06-08 · billing-owner + compliance-officer*
