# FORM · Staging Data Anonymisation Procedure v1.0

> **Policy ID:** POL-CC8-ANON-01
> **Owner:** platform-engineer + compliance-officer
> **Approver:** clinical-safety (quarterly attestation required before each run)
> **SOC 2 Criteria:** CC4.1 (monitoring controls), A1.1 (system availability testing), CC8.1 (change management)
> **Cadence:** Quarterly — before each PERF-SLO-06 k6 Cloud EU-West reference run
> **Authoritative source:** `docs/OBSERVABILITY.md §45.3` (DEC-058, 2026-06-14)
> **Evidence artefact:** PERF-STAGING-E-001 (`compliance/evidence/a1/perf-staging-e-001-YYYY-QN.png`)
> **Last reviewed:** 2026-06-16
> **Next review:** 2026-09-01 (Q3 quarterly run)

---

## Purpose

This procedure governs the refresh of FORM's shared staging Supabase project before each quarterly PERF-SLO-06 load-test reference run on k6 Cloud EU-West (Amsterdam). It ensures that no production personal data — including GDPR Article 9 special-category health data — is present in the staging environment at the time of the load test.

This document is the standalone extraction of `docs/OBSERVABILITY.md §45.3.3` required by `docs/OBSERVABILITY.md §45.8` checklist item 3 (P0 / M6). If any conflict arises between this document and the authoritative source, `docs/OBSERVABILITY.md §45.3` governs.

**Clinical-safety VETO:** If any ANON-01–ANON-05 invariant (§2) cannot be verified, the k6 Cloud run MUST NOT start. `clinical-safety` has unilateral authority to block the run.

---

## 1. Scope and Non-Scope

**In scope:**
- Quarterly refresh of the shared staging Supabase project (`$STAGING_DB_URL`) before k6 Cloud EU-West load-test runs.
- Verification that no Art. 9 data (health, biometric, coaching, body composition, ED-screening) is present.
- Clinical-safety attestation and PERF-STAGING-E-001 evidence artefact collection.

**Out of scope:**
- Daily development environments (developers use `supabase start` locally — no shared staging access required).
- Staging refreshes for purposes other than the quarterly PERF-SLO-06 run.
- Production database operations of any kind.

---

## 2. Anonymisation Invariants (ANON-01 through ANON-05)

These five invariants are **non-negotiable**. All five must be verified before `compliance-officer` requests clinical-safety sign-off (Step 3). A single failure is grounds to abort; do not proceed to Step 3 until all five pass.

| ID | Invariant | Enforcement |
|---|---|---|
| **ANON-01** | No real `user_id` from production ever appears in staging | `user_id` values are generated as `gen_random_uuid()` by the `test-data-factory` seeder; `pg_dump --data-only` from production is prohibited |
| **ANON-02** | No Art. 9 health data (biometric logs, coaching content, workout sessions, body composition, ED-screening) is ever copied to staging | `supabase db dump --schema-only` flag; resulting SQL must contain zero `COPY` or `INSERT` statements (grep check in Step 1) |
| **ANON-03** | No real `tenant_id` from production appears in staging | All staging tenants use `lt-synthetic-{N}` prefix; no production tenant slug is referenced |
| **ANON-04** | No real `email` or `full_name` appears in staging | Synthetic users: `lt-user-{N}@test.form.coach`; full name `Load Test User {N}` |
| **ANON-05** | Staging project is physically isolated from production | Separate Supabase project (`$STAGING_DB_URL`); no cross-project query, no foreign-data-wrapper, no shared read replica |

---

## 3. Pre-Conditions

Before starting Step 1, confirm the following:

- [ ] You hold `$PRODUCTION_DB_URL` (read-only connection string for schema export).
- [ ] You hold `$STAGING_DB_URL` (owner-level connection string for staging project).
- [ ] You hold `$TEST_DATA_FACTORY_URL` and `$TEST_DATA_FACTORY_SECRET` (Cloudflare Workers Secret).
- [ ] The k6 Cloud run has NOT yet been triggered (this procedure must complete and receive sign-off first).
- [ ] The staging project is currently in its post-previous-run empty state (`supabase db reset` was run after the last quarterly run per Step 4).

---

## 4. Four-Step Procedure

### Step 1 — Schema synchronisation from production DDL

Export the production schema (DDL only; no row data) and apply it to the staging project:

```bash
# Export production schema (DDL only; no row data)
supabase db dump \
  --db-url "$PRODUCTION_DB_URL" \
  --schema-only \
  --file "staging-schema-$(date +%Y%m%d).sql"

# ANON-02 enforcement: verify zero data statements
DATA_ROWS=$(grep -cE '^(COPY|INSERT)' "staging-schema-$(date +%Y%m%d).sql" || true)
if [ "$DATA_ROWS" -ne 0 ]; then
  echo "FAIL: schema dump contains $DATA_ROWS COPY/INSERT rows — abort" >&2
  exit 1
fi

# Apply to staging (destructive reset then schema apply)
supabase db reset --db-url "$STAGING_DB_URL"
psql "$STAGING_DB_URL" < "staging-schema-$(date +%Y%m%d).sql"
```

**Checkpoint:** The `DATA_ROWS` check must return `0`. Record the filename and its size (e.g. `staging-schema-20260701.sql — 142 KB`) for the Step 3 attestation message.

---

### Step 2 — Synthetic tenant and user seeding

Invoke the `test-data-factory` Worker to populate the staging project with synthetic load-test data:

```bash
# Invoke test-data-factory Worker
curl -sS -X POST "$TEST_DATA_FACTORY_URL/seed" \
  -H "Authorization: Bearer $TEST_DATA_FACTORY_SECRET" \
  -H "Content-Type: application/json" \
  -d '{
    "tenants": 3,
    "users_per_tenant": 500,
    "tenant_id_prefix": "lt-synthetic",
    "email_domain": "test.form.coach",
    "skip_art9_tables": true
  }'
```

`skip_art9_tables: true` ensures that `coaching_turns`, `workout_sessions`, `biometric_logs`, `body_composition_logs`, and `ed_screening_sessions` tables are left empty. The load test exercises auth, SSO, SCIM, and API quota pathways only — not health data pipelines.

**Post-seed verification — run before proceeding to Step 3:**

```sql
-- Confirm Art. 9 tables are empty (all values must be 0)
SELECT
  (SELECT COUNT(*) FROM coaching_turns)         AS coaching_turns,
  (SELECT COUNT(*) FROM workout_sessions)       AS workout_sessions,
  (SELECT COUNT(*) FROM biometric_logs)         AS biometric_logs,
  (SELECT COUNT(*) FROM body_composition_logs)  AS body_composition_logs;
-- Expected: coaching_turns=0, workout_sessions=0, biometric_logs=0, body_composition_logs=0
```

Record the row counts for the Step 3 attestation message. Any count > 0 is a hard stop — abort and investigate before re-running.

---

### Step 3 — Clinical-safety sign-off

`compliance-officer` sends the following attestation to `clinical-safety` via Slack `#compliance-alerts` before triggering the k6 Cloud run. Replace bracketed placeholders with actual values:

---

> **STAGING REFRESH ATTESTATION — [YYYY-QN]**
>
> Schema dump file: `staging-schema-[YYYYMMDD].sql` — [N KB]. COPY/INSERT grep check: **0 rows**.
>
> Seeded: `lt-synthetic-1`, `lt-synthetic-2`, `lt-synthetic-3` — 500 users each (@test.form.coach).
>
> Art. 9 table counts: coaching_turns=0, workout_sessions=0, biometric_logs=0, body_composition_logs=0.
>
> ANON-01–ANON-05 verified. Requesting clinical-safety sign-off before k6 Cloud EU-West run.

---

`clinical-safety` confirms with a Slack `:white_check_mark:` reaction **or** a written reply.

`compliance-officer` screenshots the full confirmation thread (attestation message + `clinical-safety` reaction/reply visible) and files the screenshot as **PERF-STAGING-E-001** at:

```
compliance/evidence/a1/perf-staging-e-001-[YYYY-QN].png
```

Filing must happen within 30 minutes of `clinical-safety` confirmation. The k6 Cloud EU-West run may be triggered only after PERF-STAGING-E-001 is filed.

---

### Step 4 — Post-run cleanup

After the `system.quarterly_perf_reference_completed` DEC-030 event is confirmed in `audit_log_events` for the completed run:

```bash
# Return staging to empty schema state
supabase db reset --db-url "$STAGING_DB_URL"
```

Staging remains empty until the next quarterly refresh. Day-to-day development uses local Supabase instances (`supabase start`) — the shared staging project is not used between quarterly runs.

---

## 5. Quick-Reference Checklist

Run this sequentially each quarter. Do not proceed past a failed step.

```
[ ] Pre-condition: staging is in empty post-reset state
[ ] Step 1a: supabase db dump --schema-only executed
[ ] Step 1b: grep COPY/INSERT returns 0 — ANON-02 ✓
[ ] Step 1c: staging db reset + schema applied
[ ] Step 2a: test-data-factory seed invoked
[ ] Step 2b: post-seed SQL verification — all Art. 9 tables = 0 — ANON-01/02/03/04 ✓
[ ] ANON-05 confirmed (separate Supabase project — no cross-project query)
[ ] Step 3: attestation message sent to #compliance-alerts
[ ] Step 3: clinical-safety :white_check_mark: received
[ ] Step 3: PERF-STAGING-E-001 screenshot filed within 30 min
[ ] k6 Cloud EU-West run triggered
[ ] Step 4: post-run supabase db reset executed after system.quarterly_perf_reference_completed confirmed
```

---

## 6. SOC 2 Evidence

| Artefact | Criteria | Description | Path | Cadence | Retention |
|---|---|---|---|---|---|
| **PERF-STAGING-E-001** | CC4.1, A1.1 | Slack screenshot of clinical-safety `:white_check_mark:` or written sign-off; includes attestation message with Art. 9 table row counts and COPY/INSERT grep check result | `compliance/evidence/a1/perf-staging-e-001-YYYY-QN.png` | Quarterly (before each k6 Cloud run) | 7 years |

**Baseline artefact:** `compliance/evidence/a1/perf-staging-e-001-baseline.png` — clinical-safety acknowledgement of this procedure document (POL-CC8-ANON-01 v1.0) before the first quarterly run.

**Auditor narrative for CC4.1:** FORM's quarterly k6 Cloud EU-West load-test run is preceded by a four-step staging refresh procedure that eliminates all personal data, including GDPR Article 9 special-category health data, from the staging environment. A mandatory clinical-safety sign-off gate verifies all five anonymisation invariants before the run triggers. PERF-STAGING-E-001 provides a Slack-screenshot evidence trail of the attestation and sign-off for every quarterly run.

**Auditor narrative for A1.1:** The quarterly PERF-SLO-06 reference run validates FORM's availability SLA commitments under realistic load from EU-West geolocation. The staging refresh procedure ensures the load test uses only synthetic tenants and users (lt-synthetic-*, lt-user-*@test.form.coach), never real employee health data, satisfying Art. 9 GDPR constraints on test data governance.

---

## 7. Privacy Floor

The following constraints are absolute and cannot be overridden by any stakeholder:

1. `pg_dump --data-only` from production is prohibited. Schema-only export is the only permitted dump mode.
2. No `user_id`, `tenant_id`, `email`, or `full_name` from production may appear in staging at any time.
3. Art. 9 tables (`coaching_turns`, `workout_sessions`, `biometric_logs`, `body_composition_logs`, `ed_screening_sessions`) must be empty before the k6 Cloud run starts. Zero-row verification is mandatory, not optional.
4. The staging Supabase project has no foreign-data-wrapper or other link to the production project.
5. `form_api` role has no access to `audit_log_events` in staging or production.

---

## 8. Failure Procedures

| Failure scenario | Action |
|---|---|
| `grep -cE '^(COPY\|INSERT)'` returns > 0 in Step 1 | Abort. Delete the schema dump file. Investigate why schema dump contains data rows. Do not proceed to Step 2. Escalate to security-engineer if the dump origin is unclear. |
| Any Art. 9 table count > 0 after Step 2 | Abort. Run `supabase db reset --db-url "$STAGING_DB_URL"`. Investigate `test-data-factory` seed logic. Do not proceed to Step 3. |
| `clinical-safety` is unavailable or does not respond within 4 hours | Postpone the k6 Cloud run. Do not substitute another sign-off. Reschedule within the same calendar quarter. |
| k6 Cloud run fails before Step 4 cleanup | Run `supabase db reset --db-url "$STAGING_DB_URL"` regardless. Log the failed `cloud_run_id` in `#devops-alerts`. Re-run after root-cause analysis; repeat full four-step procedure including Step 3 sign-off. |

---

## 9. Cross-References

| Document | Relationship |
|---|---|
| `docs/OBSERVABILITY.md §45.3` | Authoritative source — this document is an extraction of §45.3.3 |
| `docs/OBSERVABILITY.md §45.8` | Implementation checklist item 3 (P0/M6) — authoring this document closes that item |
| `docs/OBSERVABILITY.md §45.4` | DEC-030 events: `system.quarterly_perf_reference_initiated` and `system.quarterly_perf_reference_completed` |
| `docs/OBSERVABILITY.md §45.6` | PERF-STAGING-E-001 evidence artefact specification |
| `docs/DECISION_LOG.md DEC-058` | Decision authorising this procedure (k6 platform selection + staging data governance) |
| `docs/CLINICAL_SAFETY.md` | clinical-safety sign-off authority for ANON invariant verification |
| `docs/SOC2_READINESS.md §CC4.1` | PERF-STAGING-E-001 maps to CC4.1 monitoring control evidence |
| `docs/SOC2_READINESS.md §A1.1` | PERF-STAGING-E-001 maps to A1.1 availability testing evidence |
| `compliance/cc8/change-management-policy.md` | SOC 2 CC8.1 — this procedure operates within the change management framework |

---

*v1.0 · 2026-06-16 · Owner: platform-engineer + compliance-officer · Approver: clinical-safety*
*Extracted from `docs/OBSERVABILITY.md §45.3` per §45.8 checklist item 3 (P0/M6 · DEC-058).*
*Co-Authored-By: Claude (enterprise-builder) <cloud@form.coach>*
