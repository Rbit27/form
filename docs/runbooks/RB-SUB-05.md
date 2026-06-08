# RB-SUB-05 · Webhook Delivery Lag

> **Runbook ID:** RB-SUB-05
> **Severity:** P1
> **Owner:** billing-owner · reviewed by: platform-engineer · devops-lead
> **Last reviewed:** 2026-06-08
> **SOC 2 TSC:** PI1.3, CC7.2
> **Cross-refs:** docs/OBSERVABILITY.md §24.5 (AL-SUB-05), §24.6, §24.4 (SUB-SLO-01); docs/AUDIT_LOG_SCHEMA.md DEC-030; docs/INCIDENT_RESPONSE.md §1.2

---

## 1. Scenario Summary

| Field | Value |
|---|---|
| ID | RB-SUB-05 |
| Severity | P1 |
| Description | `billing.webhook.lag_ms` P95 exceeds 60,000 ms (60 seconds) over any 5-minute window. The SLO target is P95 < 30 seconds (SUB-SLO-01). A P95 > 60 s means the webhook receiver is severely backlogged or the Supabase write path is slow; subscription state changes are being delayed, which can result in users being in stale states (e.g., not yet activated after purchase, or not yet revoked after cancellation). |
| Affected tiers | Consumer (RevenueCat) and/or Enterprise (Stripe) |
| Estimated resolution time | 15–45 min |

---

## 2. Detection Signals

**How the alert fires:**
- Cloudflare Analytics Engine evaluates the `billing.webhook.lag_ms` histogram every 60 seconds. When P95 over any 5-minute window exceeds 60,000 ms, PagerDuty P1 fires.
- **Suppression:** AL-SUB-05 is suppressed for 10 minutes after a RevenueCat or Stripe announced incident.
- `billing.webhook.lag_ms` is the wall-clock time from the event's source timestamp (RevenueCat `purchasedAt` / Stripe `event.created`) to `subscription_events.processed_at`.

**Key signals to check:**
- Cloudflare Analytics Engine: `billing.webhook.lag_ms` histogram — P50, P95, P99 over 5-min windows
- Cloudflare Workers dashboard: CPU time, memory usage, request duration for `workers/billing/webhook-receiver`
- Supabase dashboard: DB query latency, connection pool utilisation, active connections
- Sentry: slow transaction traces tagged `worker=billing-webhook-receiver`
- PagerDuty alert body: `source` label indicates which upstream (RevenueCat, Stripe, or both) is lagging

---

## 3. Pre-Flight Checklist

- [ ] Check whether AL-SUB-05 suppression is active (verify `billing.alert_suppressed` in recent `audit_log`)
- [ ] Confirm upstream status:
  - RevenueCat: https://status.revenuecat.com
  - Stripe: https://status.stripe.com
  - Cloudflare Workers: https://www.cloudflarestatus.com
- [ ] Open a Linear incident ticket — P1, tag `billing-webhook-lag`, link to PagerDuty alert
- [ ] Identify the lag source from PagerDuty alert body (`source` label: `revenuecat` or `stripe`)
- [ ] Pull the current P95 lag value (Step 1 query)

---

## 4. Step-by-Step Procedure

**1. Measure current lag and identify the bottleneck** (on-call; T+0)

```bash
# Query P50/P95/P99 lag from Cloudflare Analytics Engine
curl -s "https://api.cloudflare.com/client/v4/accounts/${CF_ACCOUNT_ID}/analytics_engine/sql" \
  -H "Authorization: Bearer ${CF_API_TOKEN}" \
  -H "Content-Type: application/json" \
  -d '{
    "query": "SELECT blob1 AS source, quantilesMerge(0.5, 0.95, 0.99)(double1) AS lag_percentiles FROM METRICSDATA WHERE index1 = '"'"'billing.webhook.lag_ms'"'"' AND timestamp > NOW() - INTERVAL '"'"'5'"'"' MINUTE GROUP BY source"
  }'
```

Record the P95 value. Values > 60,000 ms confirm the alert is valid.

**2. Identify where lag is accumulating** (on-call; T+5)

Lag in `billing.webhook.lag_ms` is measured as: time from event source timestamp → `processed_at` in Supabase. The lag can accumulate in three places:

| Segment | How to diagnose |
|---|---|
| **Upstream delivery delay** | Source timestamp was long before the webhook arrived at Cloudflare. Check Stripe Dashboard → Webhooks → recent event delivery time vs. event `created` timestamp |
| **Worker processing time** | Cloudflare Workers duration metric is high (> 5 s per request). Check Workers dashboard for P99 duration |
| **Supabase write latency** | DB query takes long. Check Supabase dashboard → Database → Query performance |

```bash
# Check Cloudflare Worker request duration (P95) for billing receiver
curl -s "https://api.cloudflare.com/client/v4/accounts/${CF_ACCOUNT_ID}/analytics_engine/sql" \
  -H "Authorization: Bearer ${CF_API_TOKEN}" \
  -H "Content-Type: application/json" \
  -d '{
    "query": "SELECT quantilesMerge(0.5, 0.95)(double1) AS duration_ms FROM METRICSDATA WHERE index1 = '"'"'worker.request_duration_ms'"'"' AND blob1 = '"'"'billing-webhook-receiver'"'"' AND timestamp > NOW() - INTERVAL '"'"'10'"'"' MINUTE"
  }'
```

**3a. Remediation — Supabase write latency** (devops-lead; T+10)

If Supabase query latency is elevated:

```sql
-- Check for slow queries in the last 10 minutes
SELECT
    query,
    calls,
    total_exec_time / calls AS avg_ms,
    max_exec_time AS max_ms
FROM pg_stat_statements
WHERE query ILIKE '%subscription_events%'
ORDER BY avg_ms DESC
LIMIT 10;
```

If `subscription_events` INSERT is slow:
- Check for lock contention: `SELECT * FROM pg_locks WHERE NOT granted;`
- Check index bloat: `VACUUM ANALYZE subscription_events;` (non-blocking)
- Check connection pool: if connections are exhausted, reduce worker concurrency

If Supabase itself is degraded (check status page): no remediation — monitor and wait. The lag will self-recover once Supabase performance normalises.

**3b. Remediation — Cloudflare Worker CPU saturation** (platform-engineer; T+10)

If Worker CPU time is > 50 ms per request (near the free tier limit) or request queue is growing:

```bash
# Check current Worker invocation count and error rate
curl -s "https://api.cloudflare.com/client/v4/accounts/${CF_ACCOUNT_ID}/workers/scripts/billing-webhook-receiver/schedules" \
  -H "Authorization: Bearer ${CF_API_TOKEN}"

# Review recent Worker analytics
# Via Cloudflare Dashboard → Workers → billing-webhook-receiver → Analytics
```

If CPU saturation is due to a code regression (e.g., unnecessary crypto operations per request): deploy a hotfix.

If saturation is due to spike in webhook volume (e.g., RevenueCat batch event delivery after an outage): volume is self-limiting; monitor until backlog clears.

**3c. Remediation — upstream delivery delay** (billing-owner; T+10)

If the lag is primarily upstream (Stripe or RevenueCat is delivering events late):

1. Check upstream status page for delivery queue delays.
2. If confirmed upstream issue: AL-SUB-05 suppression should have activated. If it did not, open a Linear ticket for the suppression mechanism.
3. No action is possible until upstream resolves. Monitor lag and confirm it normalises after upstream recovery.

**4. Verify lag returns below SLO threshold** (on-call; T+30)

```bash
# Re-run Step 1 query after remediation
# Confirm P95 < 60,000 ms over the most recent 5-minute window
# Ideally confirm P95 returns to < 30,000 ms (SUB-SLO-01 target)
```

**5. Check for stale subscription states caused by the lag window** (platform-engineer; T+35)

If lag window exceeded 5 minutes, run the RB-SUB-01 scope check to confirm no subscription state inconsistencies accumulated:

```sql
-- Check for users where subscription_events was written but users.subscription_tier not yet updated
-- (indicates the webhook receiver processed the event but the Supabase update failed)
SELECT
    u.id           AS user_id,
    u.subscription_tier AS users_tier,
    se.to_state    AS latest_event_state,
    se.processed_at
FROM users u
JOIN LATERAL (
    SELECT to_state, processed_at
    FROM subscription_events
    WHERE user_id = u.id
    ORDER BY created_at DESC
    LIMIT 1
) se ON true
WHERE se.processed_at > now() - interval '2 hours'
  AND (
    (u.subscription_tier = 'premium' AND se.to_state = 'canceled') OR
    (u.subscription_tier = 'free'    AND se.to_state = 'active')
  );
```

Any rows returned indicate a stale state — escalate to RB-SUB-01 procedure.

---

## 5. Rollback / Post-Resolution Steps

- No direct rollback is available for lag — lag is a performance metric, not a data state.
- If a Worker code change was deployed: monitor lag for 5 minutes. If lag worsens, run `wrangler rollback --env production`.
- Resolve PagerDuty only after P95 lag < 60,000 ms for a full 5-minute window.
- Update Linear ticket: lag duration, peak P95 value, root cause, affected source(s).

---

## 6. Escalation Path

| Condition | Escalate to | Timeline |
|---|---|---|
| P95 lag > 120 s (2x alert threshold) | platform-engineer + devops-lead | Immediately |
| Supabase is unreachable (lag is infinite) | devops-lead (DR procedures, FORM-DR-001) | Immediately |
| Lag > 5 min and subscription states appear stale | billing-owner; initiate RB-SUB-01 | T+5 min of lag onset |
| Root cause not identified within 15 min | platform-engineer | T+15 |

---

## 7. Evidence and DEC-030 Events to Emit

All evidence uploaded to R2 `form-cold-backups/incidents/<LINEAR_TICKET_ID>/`.

**DEC-030 events:** No mandatory DEC-030 emission is required solely for a lag event. If the lag caused downstream state inconsistencies that triggered RB-SUB-01, the evidence requirements of that runbook apply.

```sql
-- After resolution, verify subscription_events processing_duration_ms during the lag window
SELECT
    date_trunc('minute', processed_at)   AS minute_bucket,
    COUNT(*)                             AS events,
    AVG(processing_duration_ms)          AS avg_ms,
    PERCENTILE_CONT(0.95) WITHIN GROUP (ORDER BY processing_duration_ms) AS p95_ms
FROM subscription_events
WHERE processed_at BETWEEN :lag_window_start AND :lag_window_end
GROUP BY 1
ORDER BY 1;
-- p95_ms should return to < 30,000 after the incident window
```

**Evidence checklist:**
- [ ] Analytics Engine query output showing lag P95 timeline (before, during, and after incident)
- [ ] Cloudflare Workers duration metrics showing Worker processing time
- [ ] Supabase query performance data during lag window
- [ ] Upstream status page state at time of alert
- [ ] Post-resolution verification that P95 < 60,000 ms
- [ ] PagerDuty alert body (save as JSON)

---

## 8. Post-Incident Checklist

- [ ] Root cause documented: upstream delivery delay, Worker saturation, or DB write latency?
- [ ] If Supabase write latency: run `VACUUM ANALYZE subscription_events` in a maintenance window; review index strategy
- [ ] Run RB-SUB-01 scope check to confirm no stale subscription states accumulated during the lag window
- [ ] SOC 2 PI1.3 evidence: lag histogram data and resolution timeline demonstrate timely-processing recovery
- [ ] Post-mortem required if: P95 lag exceeded 5 minutes, or stale subscription states were found

---

## 9. SOC 2 TSC Cross-Reference

| TSC | How this runbook satisfies it |
|---|---|
| PI1.3 | Webhook lag monitoring enforces the timely-processing commitment (SUB-SLO-01: P95 < 30 s); AL-SUB-05 fires at 60 s to provide margin; this runbook is the remediation procedure |
| CC7.2 | 5-min rolling lag monitoring detects processing performance degradation; escalation to RB-SUB-01 ensures downstream state integrity is verified after lag events |

---

*v1.0 · 2026-06-08 · billing-owner + devops-lead*
