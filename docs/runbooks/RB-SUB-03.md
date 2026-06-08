# RB-SUB-03 · Webhook Error Rate Elevated

> **Runbook ID:** RB-SUB-03
> **Severity:** P1
> **Owner:** billing-owner · reviewed by: platform-engineer · devops-lead
> **Last reviewed:** 2026-06-08
> **SOC 2 TSC:** PI1.1, PI1.3, CC7.2
> **Cross-refs:** docs/OBSERVABILITY.md §24.5 (AL-SUB-03), §24.6; docs/AUDIT_LOG_SCHEMA.md DEC-030; docs/INCIDENT_RESPONSE.md §1.2

---

## 1. Scenario Summary

| Field | Value |
|---|---|
| ID | RB-SUB-03 |
| Severity | P1 |
| Description | `billing.webhook.processed{outcome="error"}` / `billing.webhook.received` exceeds 2% over any 15-minute rolling window. Elevated error rates mean billing events are not being processed; subscription state changes are missing from `subscription_events`, which risks access-grant and access-revocation failures. |
| Affected tiers | Consumer (RevenueCat) and/or Enterprise (Stripe) depending on `source` label |
| Estimated resolution time | 20–60 min |

---

## 2. Detection Signals

**How the alert fires:**
- Cloudflare Analytics Engine evaluates `billing.webhook.processed{outcome="error"}` / `billing.webhook.received` every 60 seconds over a 15-minute rolling window. When the ratio crosses 2%, PagerDuty P1 fires.
- **Suppression:** AL-SUB-03 is suppressed for 10 minutes after a RevenueCat or Stripe announced incident. If the alert fired during a suppression window, verify the upstream incident is still active before proceeding.

**Key signals to check:**
- Cloudflare Analytics Engine: `billing.webhook.received` and `billing.webhook.processed` counters by `source` and `outcome`
- Cloudflare Workers logs: `workers/billing/webhook-receiver.ts` — look for exception types and stack traces
- Sentry: errors tagged `worker=billing-webhook-receiver` for structured exception messages
- Metabase "Subscription Health" panel 1 (Webhook Processing Rate & Error Rate): error % line crossing 2%
- PagerDuty alert body: `source` label tells you whether it is RevenueCat, Stripe, or both

---

## 3. Pre-Flight Checklist

- [ ] Confirm PagerDuty alert is not suppressed due to upstream outage (`billing.alert_suppressed` event absent from recent `audit_log`)
- [ ] Open a Linear incident ticket — P1, tag `billing-webhook`, link to PagerDuty alert
- [ ] Check upstream status pages:
  - RevenueCat: https://status.revenuecat.com
  - Stripe: https://status.stripe.com
  - Cloudflare Workers: https://www.cloudflarestatus.com
- [ ] Identify which `source` is affected: `revenuecat`, `stripe`, or both (from PagerDuty alert body)
- [ ] Retrieve current error count from Analytics Engine (Step 1 query)

---

## 4. Step-by-Step Procedure

**1. Measure the current error rate and identify the source** (on-call; T+0)

```bash
# Query Cloudflare Analytics Engine for error rate breakdown
curl -s "https://api.cloudflare.com/client/v4/accounts/${CF_ACCOUNT_ID}/analytics_engine/sql" \
  -H "Authorization: Bearer ${CF_API_TOKEN}" \
  -H "Content-Type: application/json" \
  -d '{
    "query": "SELECT blob1 AS source, blob2 AS outcome, sum(1) AS count FROM METRICSDATA WHERE index1 IN ('"'"'billing.webhook.received'"'"', '"'"'billing.webhook.processed'"'"') AND timestamp > NOW() - INTERVAL '"'"'15'"'"' MINUTE GROUP BY source, outcome ORDER BY source, outcome"
  }'
```

Calculate error rate per source: `errors / (successes + errors + idempotent)`. Confirm which source is failing.

**2. Check Cloudflare Workers logs for error type** (on-call; T+5)

```bash
# Pull recent error logs from billing webhook receiver
wrangler tail --env production --format json workers/billing/webhook-receiver 2>/dev/null \
  | grep '"level":"error"' \
  | head -30
```

Common error types and their causes:

| Error type | Likely cause |
|---|---|
| `SyntaxError: Unexpected token` | Malformed JSON body from upstream (RevenueCat/Stripe payload format change) |
| `RLS policy violation` | Supabase RLS misconfiguration; `form_system` role issue |
| `DB connection timeout` | Supabase connection pool exhausted |
| `duplicate key value violates unique constraint subscription_events_idempotency_key_unique` | Duplicate delivery not caught by idempotency check (race condition) |
| `invalid input value for enum subscription_source` | New source value not in `subscription_source` ENUM |
| `relation "subscription_events" does not exist` | DB migration not applied or wrong DB URL |

**3. Check Supabase for DB-side errors** (on-call; T+8)

```sql
-- Check for recent errors in pg_stat_activity or connection exhaustion
SELECT
    state,
    COUNT(*) AS connection_count,
    MAX(now() - state_change) AS max_duration
FROM pg_stat_activity
WHERE datname = current_database()
GROUP BY state;
```

If connection count is near the pool limit, this is a Supabase connection issue — see escalation path.

**4a. Remediation — malformed payload / upstream format change** (platform-engineer; T+15)

If error logs show payload parsing failures:

1. Retrieve a sample failing payload from Stripe or RevenueCat webhook dashboard.
2. Identify the changed field or new event type.
3. If a new event type is not in the handler's `switch` statement: add a `default` fallback that returns HTTP 200 with `outcome = 'unhandled'` (do not return 4xx — upstream will retry and worsen the error rate).
4. Deploy the fix:

```bash
wrangler deploy --env production
```

5. Monitor error rate for 5 minutes post-deploy.

**4b. Remediation — DB connection exhaustion** (devops-lead; T+15)

If connection pool is at limit:

```bash
# Check Supabase connection pool status via Supabase dashboard
# Or via pgBouncer metrics if available

# Scale connection pool temporarily via Supabase dashboard if on Pro plan
# Or reduce worker concurrency via Cloudflare Worker settings
wrangler secret put DB_POOL_MAX --env production
# Set to lower value (e.g. 5 → 3) and redeploy
wrangler deploy --env production
```

**4c. Remediation — Supabase service degradation** (devops-lead; T+15)

If Supabase status page shows an active incident:

1. AL-SUB-03 suppression should have activated — check `audit_log` for `billing.alert_suppressed`.
2. If suppression did not activate: open a Linear ticket to fix the suppression mechanism.
3. No remediation available until Supabase recovers. Monitor error rate.
4. If degradation exceeds 30 minutes: consider draining the webhook queue by replaying failed events via the Stripe/RevenueCat dashboards after Supabase recovers.

**5. Replay failed webhooks after root cause is resolved** (platform-engineer; T+30)

```bash
# Stripe: replay failed events via Stripe CLI
stripe events resend <evt_XXXXXXXXXX> --webhook-endpoint=<endpoint_id>

# For bulk replay: use Stripe Dashboard → Webhooks → Failed events → Retry all
# Or query Stripe API for events in the failure window:
stripe events list \
  --created[gt]=$(date -d '1 hour ago' +%s) \
  --limit 100 \
  --type=customer.subscription.*
```

RevenueCat does not support programmatic webhook replay; retry via RevenueCat Dashboard → Project Settings → Webhooks → Test/Retry.

**6. Confirm error rate returns below 2%** (on-call; T+45)

```bash
# Re-run the Analytics Engine query from Step 1
# Confirm error rate < 2% over the last 15 minutes before resolving PagerDuty
```

---

## 5. Rollback / Post-Resolution Steps

- If a Worker code change was deployed: monitor error rate for 10 minutes. If errors persist or increase, run `wrangler rollback --env production` and escalate to platform-engineer.
- Confirm any missed `subscription_events` rows have been backfilled by replaying failed webhooks.
- Resolve PagerDuty only after error rate is confirmed below 2% for a full 15-minute window.
- Update Linear ticket: which source failed, root cause, number of events affected during the error window.

---

## 6. Escalation Path

| Condition | Escalate to | Timeline |
|---|---|---|
| Root cause not identified within 20 min | platform-engineer | T+20 |
| Supabase DB is unreachable | devops-lead (DR procedures) | Immediately |
| Error rate > 20% (wholesale failure) | billing-owner + platform-engineer | Immediately |
| Error window > 30 min — subscription state changes may be missing | billing-owner; run RB-SUB-01 scope check after recovery | T+30 |
| Evidence of malformed payloads matching a known attack pattern | security-engineer | Immediately |

---

## 7. Evidence and DEC-030 Events to Emit

All evidence uploaded to R2 `form-cold-backups/incidents/<LINEAR_TICKET_ID>/`.

**DEC-030 events to confirm or emit:**

- `billing.consistency_check_failed` will fire if the error window caused gaps in `subscription_events` that the daily check detects. Verify the next morning's check at 03:00 UTC passes.
- No mandatory DEC-030 emission is required solely for an elevated error rate; evidence is the Analytics Engine metric data.
- If webhook secrets were rotated during investigation: `system.credential_rotated` must be emitted.

```sql
-- After resolution, verify no subscription_events gaps exist for the error window
SELECT
    date_trunc('minute', created_at) AS minute_bucket,
    COUNT(*)                         AS events_processed
FROM subscription_events
WHERE created_at BETWEEN :error_window_start AND :error_window_end
GROUP BY 1
ORDER BY 1;
-- Look for gaps (zero-count minutes during the error window)
-- Any gap means webhooks were lost and must be replayed
```

**Evidence checklist:**
- [ ] Analytics Engine query output showing error rate timeline
- [ ] Cloudflare Workers error logs (Step 2 output)
- [ ] Root cause identified and documented
- [ ] Error rate < 2% confirmed for 15-min window post-resolution
- [ ] Webhook replay confirmation (if applicable)
- [ ] PagerDuty alert body (save as JSON)

---

## 8. Post-Incident Checklist

- [ ] Confirm all failed events during the error window were replayed and processed successfully
- [ ] Run RB-SUB-01 scope check to verify no subscription state inconsistencies resulted from the error window
- [ ] If error was caused by a Stripe/RevenueCat payload format change: update the webhook receiver handler and add a test for the new event type
- [ ] SOC 2 PI1.1 evidence: confirm `subscription_events` row count during the error window is complete (no gaps after replay)
- [ ] Post-mortem required if: error window > 30 min, or > 50 events were dropped

---

## 9. SOC 2 TSC Cross-Reference

| TSC | How this runbook satisfies it |
|---|---|
| PI1.1 | Webhook error rate monitoring ensures completeness of event capture; replay procedure closes gaps caused by transient failures |
| PI1.3 | SUB-SLO-02 error rate SLO ensures timely processing; AL-SUB-03 fires at 2% to provide margin before material gaps accumulate |
| CC7.2 | 15-min rolling error rate monitoring detects processing failures in near-real-time; DEC-030 events on downstream consistency failures |

---

*v1.0 · 2026-06-08 · billing-owner + devops-lead*
