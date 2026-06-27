<!-- PURGE-INT-01 — Internal Compensating Control Memo Template
     Runbook: INCIDENT_RESPONSE.md R-30 (GDPR workout_data_purge pg_cron Stale)
     Version: 1.0 · 2026-06-27
     Owner: compliance-officer
     Storage path (completed): compliance/evidence/purge-cron/PURGE-CRON-COMP-E-001-INC-YYYYMMDD.md
     Privacy invariant: carries only aggregate counts + compliance metadata.
       No user_id, no email, no health values in any field.
       form_api must never route any R-30 event to any user-facing or tenant-facing endpoint.
-->

# FORM INTERNAL — PRIVILEGED & CONFIDENTIAL

**Compensating Control Memo — workout_data_purge Cron Failure**

| Field | Value |
|---|---|
| Date | YYYY-MM-DD |
| Incident ID | INC-YYYYMMDD |
| Initial severity | \[P1 / P0 / P0 escalated\] |
| Filing path | `compliance/evidence/purge-cron/PURGE-CRON-COMP-E-001-INC-YYYYMMDD.md` |

---

## 1. CONTROL FAILURE

`pg_cron` job 26 (`workout_data_purge`, schedule `0 2 * * *`) failed to run for
**[N]** consecutive days starting **[confirmed_stale_since ISO]** UTC.

- Last successful run: `[timestamp from pg_cron.job_run_details]`
- Root cause (H1–H5): `[hypothesis confirmed]`

---

## 2. IMPACT ASSESSMENT

| Metric | Count |
|---|---|
| Affected users with overdue workout data (R-30-C1) | [count] |
| `workout_sets` past retention | [count] |
| `workout_sessions` past retention | [count] |
| `body_metrics` past retention (R-30-C3) | [count] |
| Art. 9 risk flag (`body_metrics_count > 0`) | **YES / NO** |
| DPA notification assessment required (Art. 9 AND stale > 72h) | **YES / NO** |

> All counts are aggregates only — no `user_id`, no email, no health values recorded here.

---

## 3. COMPENSATING CONTROL EXECUTED

| Field | Value |
|---|---|
| Manual purge SQL executed at | [ISO timestamp] UTC |
| PAM session ID | [session UUID] |
| Users purged (count) | [count] |
| `workout_sets` deleted | [count] |
| `workout_sessions` deleted | [count] |
| `body_metrics` deleted | [count] |
| DEC-030 `system.purge_cron_manual_run_completed` emitted | **YES / NO** |
| Chain record `audit_log_events.event_id` | [UUID] |

---

## 4. RESTORATION

| Field | Value |
|---|---|
| Job 26 restored at | [ISO timestamp] UTC |
| Confirmation test run (`SELECT cron.run_job('workout_data_purge')`) | **SUCCESS / FAILURE** |
| R-30-C1 re-run post-restore (must return 0 rows) | [row count] |
| DEC-030 `system.purge_cron_restored` emitted | **YES / NO** |
| PagerDuty `purge-job-26-stale` dedup key auto-resolved | **YES / NO** |

---

## 5. ART. 33 DPA NOTIFICATION ASSESSMENT

> Complete this section **only if P0 escalated** (`body_metrics` affected AND stale > 72h).
> Outside counsel engagement is required before any notification determination.
> If P1 or P0 (no Art. 9 data, or stale ≤ 72h): mark all fields N/A.

| Field | Value |
|---|---|
| `body_metrics_count` | [N] |
| Stale duration at detection | [N hours] |
| Was retained data accessed by any FORM system beyond normal read path? | **YES / NO / UNKNOWN** |
| Outside counsel engaged | **YES / NO** — [name / firm] |
| Assessment outcome | Notification required / Not required / Pending counsel opinion |
| DPA notification filed | **YES / NO / N/A** |
| Filing reference (if applicable) | [reference] |
| Relevant supervisory authority | ICO (UK) / Irish DPC (EEA) / Other: [specify] |

> **No customer-facing notification template exists for R-30.** If outside counsel determines
> Art. 33 notification is required (P0 escalated), the DPA notification is filed directly with
> the relevant supervisory authority per FORM's GDPR DPA registry. Affected users are not
> directly notified unless the Art. 34 high-risk breach threshold is also met — a determination
> requiring outside counsel.

---

## 6. SIGN-OFF

| Role | Name | Date |
|---|---|---|
| Compliance-officer | [name] | [date] |
| Founder (required if P0 escalated) | [name or N/A] | [date or N/A] |

---

*Filed as evidence artefact PURGE-CRON-COMP-E-001 under TSC CC6.5 / PI1.2.
Retention: 7 years. R2 path: `compliance/evidence/purge-cron/r30-<incident_id>/`.*
