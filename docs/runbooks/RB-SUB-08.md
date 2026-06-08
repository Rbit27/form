# RB-SUB-08 · Subscription State Orphan

> **Runbook ID:** RB-SUB-08
> **Severity:** P3
> **Owner:** billing-owner · reviewed by: platform-engineer
> **Last reviewed:** 2026-06-08
> **SOC 2 TSC:** PI1.1, CC6.1
> **Cross-refs:** docs/OBSERVABILITY.md §24.5 (AL-SUB-08), §24.9; docs/AUDIT_LOG_SCHEMA.md DEC-030; docs/DATA_MODEL.md §24.7; docs/INCIDENT_RESPONSE.md §1.2

---

## 1. Scenario Summary

| Field | Value |
|---|---|
| ID | RB-SUB-08 |
| Severity | P3 |
| Description | One or more `user_id` values exist in `subscription_events` with no matching row in the `users` table. These orphaned events represent subscription history for users who have since been hard-deleted or whose user record is missing due to a data integrity issue. Orphaned rows do not grant or revoke access (the `users.subscription_tier` column controls access), but they create inconsistencies in audit records and may indicate incomplete GDPR erasure or a data pipeline bug. |
| Affected tiers | Consumer and/or Enterprise |
| Estimated resolution time | 1–4 h investigation; remediation may span multiple days |

---

## 2. Detection Signals

**How the alert fires:**
- The daily pg_cron `billing-consistency-check` (03:00 UTC) runs a LEFT JOIN of `subscription_events.user_id` against `users.id` and counts unmatched rows. Any count > 0 posts a message to Slack `#observability`.
- No PagerDuty page is issued. The Slack message is the notification.

**Key signals to check:**
- Slack `#observability`: message from the daily check with orphan count and a sample of `user_id` values
- `subscription_events` for the orphaned `user_id` values: event types, dates, sources
- GDPR erasure logs: `privacy.erasure_completed` events in `audit_log` for the orphaned `user_id` values
- `users` table soft-delete columns: `deleted_at` (if soft-delete is used before hard-delete)

---

## 3. Pre-Flight Checklist

- [ ] Confirm Slack `#observability` message is from the daily check (not a test message)
- [ ] Note the orphan count and the sample `user_id` values from the Slack message
- [ ] This is a P3 — no immediate action required; investigate during normal business hours
- [ ] Check GDPR erasure pipeline for recent hard-deletes (Step 1 query)
- [ ] No PagerDuty alert; no Linear ticket auto-created — create one manually if investigation reveals a data pipeline bug

---

## 4. Step-by-Step Procedure

**1. Identify all orphaned subscription_events rows** (billing-owner or platform-engineer; next business day)

```sql
-- Run as compliance_officer or form_system role
-- Find user_ids in subscription_events with no matching users row
SELECT
    se.user_id,
    COUNT(*)                    AS event_count,
    MIN(se.created_at)          AS first_event,
    MAX(se.created_at)          AS last_event,
    array_agg(DISTINCT se.event_type ORDER BY se.event_type) AS event_types,
    array_agg(DISTINCT se.source ORDER BY se.source)         AS sources,
    se.tenant_id
FROM subscription_events se
LEFT JOIN users u ON u.id = se.user_id
WHERE u.id IS NULL
GROUP BY se.user_id, se.tenant_id
ORDER BY MAX(se.created_at) DESC;
```

**2. Classify the orphan type** (platform-engineer; same investigation session)

For each orphaned `user_id`, determine why the `users` row is absent:

**Case A — GDPR Art. 17 hard-delete (expected)**

```sql
-- Check for erasure events for the orphaned user_id
SELECT
    event_type,
    payload->>'user_id_hash'  AS user_id_hash,
    payload->>'erasure_type'  AS erasure_type,
    created_at
FROM audit_log
WHERE event_type IN ('privacy.erasure_completed', 'privacy.erasure_initiated')
  AND created_at > (
      SELECT MAX(created_at) FROM subscription_events WHERE user_id = :orphaned_user_id
  ) - interval '30 days'
ORDER BY created_at DESC;
```

Note: `privacy.erasure_completed` events use `user_id_hash` (SHA-256), not the plaintext `user_id`. Compute `encode(digest(:orphaned_user_id::text, 'sha256'), 'hex')` to match them.

If an erasure event exists: this is a known and acceptable orphan. The `subscription_events` rows for a hard-deleted user are intentionally retained for 7 years as financial records per `docs/DATA_MODEL.md §12` (GDPR Art. 17(3)(b) legal obligation exception). No action required.

**Case B — Missing user row (unexpected)**

If no erasure event exists: the `users` row was deleted without going through the proper GDPR erasure pipeline, or a data pipeline bug caused the user record to be lost while subscription events remain.

This is a data integrity issue. Create a Linear ticket tagged `data-integrity`, assign to platform-engineer.

**Case C — Soft-deleted user (transient)**

```sql
-- Check if the user exists in a soft-deleted state
-- (if soft-delete uses a deleted_at column rather than immediate hard-delete)
SELECT id, deleted_at, subscription_tier
FROM users
WHERE id = :orphaned_user_id;
-- If this returns a row with deleted_at IS NOT NULL: the user is in the 30-day grace period
-- The orphan will resolve when the nightly hard-delete job runs
```

If `deleted_at IS NOT NULL`: the orphan is transient (within 30-day erasure grace period). No action required; it will resolve naturally.

**3a. Remediation — Case B (unexpected missing user)** (platform-engineer)

Do NOT insert a placeholder `users` row or attempt to reconstruct the user. This would violate GDPR and create false data.

Do NOT delete the orphaned `subscription_events` rows. They are append-only financial records.

Correct action:
1. If the `user_id` corresponds to a real user who should still exist: identify the root cause (DB migration bug, accidental delete, RLS policy gap). Restore from PITR if within the PITR retention window and the deletion was recent.
2. Document in Linear: `user_id` (UUID), event count, event date range, source, root cause hypothesis.
3. Escalate to platform-engineer and compliance-officer if > 5 unexpected orphans are found in a single run.

**4. Monitor for recurrence** (billing-owner)

After classifying all orphans from the current alert:
- If all are Case A (expected erasure-related): no action beyond documentation; confirm count is consistent with the GDPR erasure volume for the period.
- If any are Case B: investigate and fix the root cause before closing. The daily check will re-alert if new Case B orphans appear.

---

## 5. Rollback / Post-Resolution Steps

- No data rollback is required for Case A orphans.
- For Case B: the root cause fix (stopping the erroneous deletion path) is the remediation. No data restoration without IC approval and PITR availability.
- After investigation: add a comment to the Slack thread with the classification result (case type, count, action taken).
- If a Linear ticket was opened: close it with root cause and resolution.

---

## 6. Escalation Path

| Condition | Escalate to | Timeline |
|---|---|---|
| > 20 orphans in a single daily run | platform-engineer + billing-owner | Same business day |
| Case B orphans (unexpected deletion) found | compliance-officer + platform-engineer | Same business day |
| Case B orphans from enterprise `tenant_id` | customer-success + compliance-officer | Same business day |
| Root cause appears to be an RLS policy gap | security-engineer + platform-engineer (treat as P1 uplift) | Immediately |

---

## 7. Evidence and DEC-030 Events to Emit

No mandatory DEC-030 events are emitted solely for P3 orphan detection. The daily `billing-consistency-check` may emit `billing.consistency_check_failed` (CRITICAL) if the orphan count crosses a separate threshold.

```sql
-- Verify the daily check result for the orphan detection
SELECT
    check_name,
    check_date,
    passed,
    failure_count,
    failure_sample
FROM billing_consistency_checks
WHERE check_name = 'subscription_event_orphan_check'
  AND check_date > now() - interval '25 hours'
ORDER BY check_date DESC
LIMIT 1;
```

**Evidence checklist (if Case B is found):**
- [ ] Step 1 query output: orphan count, user_id list, event date ranges
- [ ] Step 2 query output: erasure event search results (Case A confirmation or Case B confirmation)
- [ ] Linear ticket created with classification and root cause hypothesis
- [ ] Escalation to compliance-officer documented if Case B orphans found

---

## 8. Post-Incident Checklist

- [ ] All orphans classified (Case A, B, or C)
- [ ] Case A orphans: confirmed as expected GDPR hard-delete artefacts; no action
- [ ] Case B orphans: root cause investigated; Linear ticket open until fixed; recurrence monitoring in place
- [ ] Case C orphans: transient; resolved at next nightly hard-delete run
- [ ] If Case B: confirm the deletion pipeline has been fixed; verify no new unexpected orphans in the following daily check
- [ ] SOC 2 PI1.1 evidence: financial records (`subscription_events`) are retained even after user deletion per GDPR Art. 17(3)(b); orphan classification documents this is by design for Case A

---

## 9. SOC 2 TSC Cross-Reference

| TSC | How this runbook satisfies it |
|---|---|
| PI1.1 | `subscription_events` retention after user deletion is a deliberate financial-record completeness control (GDPR Art. 17(3)(b) exception); orphan classification procedure documents the distinction between expected and unexpected orphans |
| CC6.1 | Orphaned subscription events do not grant access (access gate is `users.subscription_tier`); this runbook confirms the access control layer is not affected by orphan rows |

---

*v1.0 · 2026-06-08 · billing-owner + platform-engineer*
