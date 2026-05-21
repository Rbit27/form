# Health Data Retention Decision Record

**File reference:** P1-RET-001  
**SOC 2 criterion:** P4.2 — Retains Personal Information  
**GDPR articles:** Art. 5(1)(e) storage limitation · Art. 17 right to erasure  
**Remediates:** P-GAP-004 (P1) — workout/health data retention periods not formally decided  
**Input document:** `docs/SOC2_READINESS.md §35.6.3` (proposed schedule)  
**Owner:** compliance-officer  
**Decision authority:** compliance-officer + founder + outside counsel sign-off  
**Status:** ✅ Decided — pending legal sign-off before privacy policy publication  
**Effective date:** 2026-05-21  
**Next review:** Annually — or upon material product change affecting data categories  
**Artifact path:** `compliance/p1/retention-decisions.md`

---

## Purpose

FORM processes Art. 9 special-category health data. GDPR Art. 5(1)(e) requires that personal data is "kept in a form which permits identification of data subjects for no longer than is necessary" for the purpose for which it was collected.

This document records the formal per-category retention decisions required to:

1. Close **P-GAP-004** in the SOC 2 Type II gap register
2. Provide the authoritative input for the privacy policy retention schedule
3. Enable implementation of Supabase TTL migration policies for active-coaching tables
4. Satisfy AICPA P4.2 — that the entity retains personal information only for as long as necessary to fulfil the purposes for which it was collected

All decisions in this document supersede the "TBD" entries in `docs/SOC2_READINESS.md §35.6.3`. Any future change to a decided retention period requires a new version of this document with updated sign-off.

---

## Scope

This record covers the following Supabase (Postgres) tables that hold health-adjacent personal data:

| Table | Data category | GDPR Art. 9 applicable? |
|---|---|---|
| `workout_sessions` | Physical activity records | Yes — inference of health status possible |
| `sets` | Per-exercise repetition data | Yes — in aggregate with sessions |
| `coaching_turns` | AI coaching dialogue | Yes — contains health goals, injury context, coaching history |
| `cv_sessions` | Computer-vision form-check metadata | Yes — biometric-adjacent (movement signature) |
| `user_profile` (health fields) | Age, sex, height, weight goal, injury flags, goal category | Yes — explicit health data |
| `meal_log` | Food intake log | Yes — explicit health/dietary data |
| `wearable_readings` | HRV, resting HR, sleep, steps (Apple Health / Health Connect) | Yes — explicit health data |

**Out of scope for this document** (retention already decided via other mechanisms):

| Table / store | Retention | Decision source |
|---|---|---|
| ClickHouse `fact_events`, `fact_session_metrics` | 2 years — TTL enforced in DDL | ClickHouse schema DDL |
| `audit_log` (DEC-030 HMAC chain) | 7 years — WORM bucket | DEC-030; SOC 2 evidence requirement; GDPR Art. 30 |
| Cloudflare R2 video frames | In-session only; purged post-inference | `docs/SECURITY.md §2` |
| Stripe payment data | Per Stripe DPA + PCI DSS requirements | Stripe DPA |

---

## Decision Inputs

The following inputs were used to determine appropriate retention periods:

1. **Product purpose:** FORM is a long-term fitness coaching application. Users accumulate workout history over months and years. Coaching context improves with history length. Shorter retention reduces product value; longer retention increases privacy risk.

2. **GDPR Art. 5(1)(e) storage limitation:** Data must not be retained longer than necessary for the stated purpose. "Necessary" is assessed against each category's coaching utility horizon, not the maximum technically possible period.

3. **User expectation baseline:** Consumer fitness apps with history features (Garmin Connect, Strava, MyFitnessPal) retain data indefinitely unless deleted by the user. FORM's position: shorter default retention with user-controllable extension is more privacy-respecting than indefinite retention.

4. **Coaching utility horizon:** Assessed by sports-scientist input — workout history beyond 3 years has marginal training-adaptation value for most recreational users; coaching_turns beyond 2 years rarely inform current programming; CV session metadata beyond 1 year rarely improves form-feedback calibration.

5. **Minimum anonymity period:** Cohort analytics in ClickHouse retain aggregates for 2 years; individual-level data should not outlast aggregate cohort data without strong justification.

6. **P-GAP-004 proposed periods:** `docs/SOC2_READINESS.md §35.6.3` proposed initial periods which this document formally adopts with one amendment (see §4.1 note on `cv_sessions`).

---

## Formal Retention Decisions

### 4.1 Decision Table

All periods run from the **data creation timestamp** (i.e., the date the record was inserted), not from last access or account creation. Account deletion or consent withdrawal triggers erasure immediately, regardless of whether the scheduled retention period has expired — GDPR Art. 17 takes precedence.

| Data category | Table(s) | **Decided retention** | Legal basis for period | Disposal method | Supabase TTL mechanism |
|---|---|---|---|---|---|
| **Workout sessions** | `workout_sessions` | **3 years from session date** | GDPR Art. 5(1)(e) — 3-year window covers realistic training history for adaptive programming; beyond 3 years, marginal coaching utility | Hard delete + cascade to `sets`; `privacy.erasure_completed` DEC-030 event | Supabase `pg_cron` scheduled job; soft-delete flag + sweep at T+3yr |
| **Exercise sets** | `sets` | **3 years from session date** (cascades from `workout_sessions`) | Derived from parent record; same legal basis | Cascade delete on parent; no independent TTL needed | Cascade delete |
| **AI coaching turns** | `coaching_turns` | **2 years from creation** | GDPR Art. 5(1)(e) — coaching dialogue has shorter relevance window than workout history; 2-year window covers meaningful coaching continuity | Hard delete; `privacy.erasure_completed` DEC-030 event | Supabase `pg_cron` sweep; `created_at < NOW() - INTERVAL '2 years'` |
| **CV session metadata** | `cv_sessions` | **1 year from creation** | GDPR Art. 9 — CV keypoint data is biometric-adjacent; highest sensitivity category warrants shortest retention; 1-year window sufficient for form-progression coaching | Hard delete; `privacy.erasure_completed` DEC-030 event | Supabase `pg_cron` sweep; `created_at < NOW() - INTERVAL '1 year'` |
| **Health profile** | `user_profile` (health fields: `height_cm`, `weight_kg_goal`, `biological_sex`, `date_of_birth`, `fitness_goal`, `injury_flags`, `restrictions`) | **Until account deletion or consent withdrawal** | GDPR Art. 6(1)(b) + Art. 9(2)(a) — user-active data; cannot deliver core service without it; retained only while account active | Hard delete on account closure or erasure request; `privacy.erasure_completed` event | Account deletion cascade; no time-based TTL |
| **Meal log** | `meal_log` | **2 years from entry creation** | GDPR Art. 5(1)(e) — nutritional patterns have a 1-2 year analytical window; 2-year retention balances longitudinal nutrition coaching against minimisation obligation | Hard delete; `privacy.erasure_completed` DEC-030 event | Supabase `pg_cron` sweep; `created_at < NOW() - INTERVAL '2 years'` |
| **Wearable readings** | `wearable_readings` | **2 years from reading date** | GDPR Art. 5(1)(e) — HRV/sleep trend analysis yields diminishing returns beyond 2 years; aligns with ClickHouse analytics TTL; balances recovery-trend utility | Hard delete; `privacy.erasure_completed` DEC-030 event | Supabase `pg_cron` sweep; `reading_date < NOW() - INTERVAL '2 years'` |

**Amendment note on `cv_sessions`:** The §35.6.3 entry classified CV data as "biometric-adjacent." This document formally confirms that classification. The 1-year retention is the most restrictive in this table and reflects that CV keypoint data carries the highest re-identification risk of any data category in FORM's stack, even though FORM's on-device inference means raw video is never stored.

### 4.2 GDPR Legal Basis Summary

| Table | Legal basis for processing | Legal basis for retention period |
|---|---|---|
| `workout_sessions`, `sets` | Art. 6(1)(b) — performance of contract; Art. 9(2)(a) — explicit consent | Art. 5(1)(e) — storage limitation; Art. 17 erasure right |
| `coaching_turns` | Art. 6(1)(b) — performance of contract; Art. 9(2)(a) — explicit consent | Art. 5(1)(e) |
| `cv_sessions` | Art. 9(2)(a) — explicit consent (biometric-adjacent category) | Art. 5(1)(e); Art. 9 elevated protection |
| `user_profile` health fields | Art. 9(2)(a) — explicit consent | Art. 6(1)(b) — necessary for contract; deletion upon contract end or consent withdrawal |
| `meal_log` | Art. 9(2)(a) — explicit consent | Art. 5(1)(e) |
| `wearable_readings` | Art. 9(2)(a) — explicit consent (health data per Art. 9(1)) | Art. 5(1)(e) |

---

## User Controls

The retention periods above are **maximum default periods**. Users may shorten them at any time via in-app controls:

| User action | Effect | GDPR basis |
|---|---|---|
| **Delete account** | Triggers erasure queue for all categories; grace period 30 days; completed within 30 days of request | Art. 17(1)(b) — withdrawal of consent |
| **Withdraw health-data consent** (Settings → Privacy → Health Consent) | Triggers erasure queue for all Art. 9 categories; `privacy.consent_withdrawn` DEC-030 event; processing ceases immediately | Art. 7(3) — right to withdraw consent |
| **Delete individual record** (e.g. delete a workout session or meal log entry) | Hard delete of that record and its cascade dependencies; DEC-030 `data.record_deleted` event | Art. 17(1) — erasure on request |
| **Download My Data (DSAR)** | Exports all categories above in JSON format; retention periods included in export metadata | Art. 15 — right of access |

The 30-day grace period for account deletion is disclosed in the privacy policy. Users who initiate deletion and change their mind within 30 days may contact `privacy@form.coach` to restore the account. After 30 days, erasure is irreversible.

---

## Implementation Requirements

Closing P-GAP-004 requires the following implementation steps. These must be completed before the privacy policy is published.

### 6.1 Supabase TTL Migrations

The following `pg_cron` jobs must be deployed. Each job runs daily at 02:00 UTC (off-peak). Each batch is capped at 1,000 rows per run to avoid replication lag spikes on the Supabase managed Postgres instance.

```sql
-- workout_sessions: 3-year TTL (sets cascade automatically via FK ON DELETE CASCADE)
SELECT cron.schedule(
  'retention-workout-sessions',
  '0 2 * * *',
  $$
    DELETE FROM workout_sessions
    WHERE session_date < NOW() - INTERVAL '3 years'
      AND id IN (
        SELECT id FROM workout_sessions
        WHERE session_date < NOW() - INTERVAL '3 years'
        LIMIT 1000
      );
  $$
);

-- coaching_turns: 2-year TTL
SELECT cron.schedule(
  'retention-coaching-turns',
  '0 2 * * *',
  $$
    DELETE FROM coaching_turns
    WHERE created_at < NOW() - INTERVAL '2 years'
      AND id IN (
        SELECT id FROM coaching_turns
        WHERE created_at < NOW() - INTERVAL '2 years'
        LIMIT 1000
      );
  $$
);

-- cv_sessions: 1-year TTL
SELECT cron.schedule(
  'retention-cv-sessions',
  '0 2 * * *',
  $$
    DELETE FROM cv_sessions
    WHERE created_at < NOW() - INTERVAL '1 year'
      AND id IN (
        SELECT id FROM cv_sessions
        WHERE created_at < NOW() - INTERVAL '1 year'
        LIMIT 1000
      );
  $$
);

-- meal_log: 2-year TTL
SELECT cron.schedule(
  'retention-meal-log',
  '0 2 * * *',
  $$
    DELETE FROM meal_log
    WHERE created_at < NOW() - INTERVAL '2 years'
      AND id IN (
        SELECT id FROM meal_log
        WHERE created_at < NOW() - INTERVAL '2 years'
        LIMIT 1000
      );
  $$
);

-- wearable_readings: 2-year TTL
SELECT cron.schedule(
  'retention-wearable-readings',
  '0 2 * * *',
  $$
    DELETE FROM wearable_readings
    WHERE reading_date < NOW() - INTERVAL '2 years'
      AND id IN (
        SELECT id FROM wearable_readings
        WHERE reading_date < NOW() - INTERVAL '2 years'
        LIMIT 1000
      );
  $$
);
```

Each sweep must emit a `data.retention_sweep_completed` DEC-030 audit event with:
- `action: "data.retention_sweep_completed"`
- `resource_type: "<table_name>"`
- `data.rows_deleted: <count>`
- `data.sweep_date: <ISO timestamp>`
- `data.retention_period_days: <days>`
- `data_class: "system"`

If `rows_deleted > 0`, the HMAC chain event confirms automated erasure to auditors without requiring a manual process.

### 6.2 DEC-030 Event for Scheduled Erasure

The `data.retention_sweep_completed` event is a new DEC-030 event type. It must be added to `docs/AUDIT_LOG_SCHEMA.md` in the data-lifecycle events section. Schema:

```jsonc
{
  "event_id": "<uuid-v7>",
  "tenant_id": "system",          // sweeps are cross-tenant; tenant_id = "system"
  "actor_id": "system:retention-worker",
  "actor_type": "service",
  "action": "data.retention_sweep_completed",
  "resource_type": "table",
  "resource_id": "<table_name>",
  "data": {
    "rows_deleted": 1247,
    "retention_period_days": 730,
    "oldest_deleted_timestamp": "2024-05-20T01:58:00Z",
    "sweep_date": "2026-05-21T02:00:04Z"
  },
  "data_class": "system",
  "timestamp": "2026-05-21T02:00:04.123Z",
  "prev_hmac": "<sha256-of-previous-event>",
  "hmac": "<sha256-of-this-event>"
}
```

### 6.3 Privacy Policy Update

Once Supabase TTL migrations are deployed and verified, add the following retention table to the privacy policy at `form.coach/privacy` (section: "How long we keep your data"):

| Data type | How long we keep it | Deletion method |
|---|---|---|
| Workout sessions and exercise sets | 3 years from the date of activity | Automatically deleted; you can request earlier deletion |
| AI coaching conversations | 2 years from the date of each message | Automatically deleted; you can request earlier deletion |
| Movement analysis sessions | 1 year from the date of analysis | Automatically deleted; you can request earlier deletion |
| Nutrition log entries | 2 years from the date of entry | Automatically deleted; you can request earlier deletion |
| Wearable sensor data (heart rate, sleep, etc.) | 2 years from the date the data was recorded | Automatically deleted; you can request earlier deletion |
| Your health profile (height, fitness goals, injury notes) | Until you delete your account or withdraw consent | Deleted immediately on account closure |
| Service audit log (anonymised) | 7 years | Required for legal and compliance purposes; your identity is removed if you delete your account |

### 6.4 Monitoring and Alerting

- **Sweep success monitoring:** If a scheduled sweep does not emit a `data.retention_sweep_completed` event within 25 hours of its scheduled run time, PagerDuty alert fires to `compliance-officer`.
- **Overdue-data alert:** Monthly: query each table for rows that are past their retention period. If any row older than the retention period + 7-day grace window exists and no DEC-030 deletion event is found for it, escalate to `compliance-officer` as a control failure.
- **Annual audit sample:** During SOC 2 observation period, auditors may request a sample confirming no rows exist past their retention period. The monthly overdue-data query output serves as the ongoing evidence artifact (PRE-35-E-008).

---

## Legal Review Requirements

This document **must not be referenced in the published privacy policy** until outside counsel has:

1. Confirmed that the chosen retention periods satisfy GDPR Art. 5(1)(e) storage limitation for each category as used in FORM's product
2. Confirmed that the legal bases cited in §4.2 are appropriate for FORM's processing purposes and consent mechanism
3. Reviewed the user control mechanisms in §5 against Art. 17, Art. 7(3), and Art. 15 requirements
4. Signed and dated the sign-off record in §8

Estimated outside counsel review scope: 2–4 hours. Route through the same counsel engaged for MSA/DPA review.

---

## Annual Review Procedure

This document must be reviewed annually, or within 30 days of any of the following triggers:

| Trigger | Review scope |
|---|---|
| New data category added to FORM's product | Add retention decision for new category; obtain fresh legal sign-off |
| Existing data category materially changes its processing purpose | Re-assess retention period for that category |
| Relevant regulatory guidance published (e.g. EDPB opinion on fitness app data) | Assess whether decisions need updating |
| Enterprise customer DPA negotiation raises retention question | Note negotiated position; document whether it diverges from this record and why |
| Annual privacy review (P-GAP-008 calendar entry) | Full review of all categories; confirm periods remain appropriate; update version |

Annual review owner: `compliance-officer`. Output: updated version of this document with revised effective date. If no changes are needed, a dated "no change" confirmation entry in the sign-off record in §8 is sufficient.

---

## Gap Closure Checklist

This document closes P-GAP-004 when **all** of the following items are complete:

- [x] Formal retention decision recorded per category (this document)
- [ ] Outside counsel legal sign-off obtained (§8 sign-off record)
- [ ] Supabase TTL migrations deployed and verified in staging
- [ ] Supabase TTL migrations deployed to production
- [ ] `data.retention_sweep_completed` DEC-030 event type added to `docs/AUDIT_LOG_SCHEMA.md`
- [ ] Retention table added to privacy policy at `form.coach/privacy`
- [ ] Privacy policy reviewed by outside counsel and published
- [ ] Monitoring alerts configured (§6.4)
- [ ] First successful sweep confirmed via DEC-030 event chain
- [ ] Evidence artifact PRE-35-E-008 filed (monthly overdue-data query output)

P-GAP-004 status: 🟡 **Partially closed** — decision recorded; implementation and legal sign-off pending.

---

## Sign-Off Record

| Role | Name | Date | Signature / Commit |
|---|---|---|---|
| compliance-officer | _[to be signed]_ | _[date]_ | _[git commit hash or DocuSign envelope]_ |
| Founder | _[to be signed]_ | _[date]_ | _[git commit hash]_ |
| Outside counsel | _[to be signed]_ | _[date]_ | _[DocuSign envelope ID]_ |

> **Note:** "Signature / Commit" accepts either a DocuSign envelope ID (for counsel) or a git commit hash of the reviewed version (for internal signatories). The git commit hash approach is accepted by SOC 2 auditors as evidence of management sign-off when accompanied by the committer's identity in the git log.

---

**P1-RET-001 · v1.0 · 2026-05-21**

*Input: `docs/SOC2_READINESS.md §35.6.3` proposed periods — all formally adopted by this decision record with one amendment (cv_sessions confirmed as biometric-adjacent; 1-year period confirmed). Decisions are effective immediately for implementation planning. Legal sign-off required before privacy policy publication.*
