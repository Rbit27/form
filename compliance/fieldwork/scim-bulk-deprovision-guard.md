# SCIM Bulk Deprovision Guard — SOC 2 Fieldwork Guide

**Version:** v1.0 · 2026-06-20  
**Owner:** compliance-officer  
**SOC 2 Criteria:** CC6.3 · A1.2 · CC7.2 · CC9.2  
**Evidence artefact:** GUARD-E-001  
**Evidence path:** `compliance/evidence/scim-guard/GUARD-E-001_<YYYY-QN>.csv`  
**Source design:** `docs/SSO_SCIM_IMPLEMENTATION.md §34` (DEC-066) + `§35` (DEC-069)

---

## §1 Purpose and Scope

This guide enables a SOC 2 auditor to independently verify the SCIM Bulk Deprovision Guard (BDG) during the observation window. It covers:

1. **BDG architecture** — what the guard does and how it operates
2. **Three-step audit traceability reconstruction** — reconstructing the guard limit at any historical block event from immutable records
3. **GUARD-E-001 collection SQL** — primary export + CC6.3 assertion + CC7.2 stale-advisory cross-check
4. **Zero-event attestation procedure** — for quarters where no guard events fired
5. **Privacy floor attestation** — confirming no employee PII in any collection output

The compliance-officer provides `compliance_reviewer` database access at the start of the fieldwork engagement. See `compliance/evidence/auditor-onboarding/README.md` for credential provisioning.

---

## §2 BDG Architecture (Summary)

The SCIM Bulk Deprovision Guard is a preventive control built into the SCIM Worker (`apps/scim-worker`). It operates before any `PATCH /Users/{id}` or `DELETE /Users/{id}` write reaches Postgres.

**Decision model (two-layer defence):**
- **Layer 1 (preventive — this guide):** BDG fires HTTP 422 `BULK_DEPROVISION_GUARD_TRIGGERED` when a single IdP sync would deactivate more than `threshold_pct` percent of a tenant's `contracted_seats` within a 5-minute rolling window.
- **Layer 2 (reactive):** `AL-SCIM-MASS-01` and `docs/INCIDENT_RESPONSE.md R-24` trigger after deprovisioning has occurred.

**Guard evaluation (`enforceDeprovisionGuard()`):**

```
1. Fetch guard config from KV key scim:guard_cfg:{tenantId} (3600s TTL)
   └─ Cache miss: JOIN enterprise_contracts + tenant_sso_configs via Supabase
2. threshold_pct = 100 → guard disabled (fast-path; no KV counter write)
3. Active override? → allow, increment counter, auto-revoke after first over-limit request
4. KV counter key: scim:deprov_guard:{tenantId}:{epoch5min} (600s TTL)
5. IF (increment counter) > ceil(contracted_seats × threshold_pct / 100)
   THEN HTTP 422 + emit scim.bulk_deprovision_blocked (HIGH, 7yr)
```

**Five DEC-030 events (all HMAC-chained, append-only):**

| Event | Severity | Retention | Trigger |
|---|---|---|---|
| `scim.bulk_deprovision_blocked` | HIGH | 7yr | Guard limit exceeded in 5-min window |
| `scim.bulk_deprovision_override_issued` | HIGH | 7yr | CSM countersigns tenant_owner override request |
| `scim.bulk_deprovision_override_used` | HIGH | 7yr | First over-limit request under active override (auto-revoke fires) |
| `scim.bulk_deprovision_threshold_updated` | STANDARD | 7yr | Tenant admin/owner changes threshold via Admin Dashboard |
| `system.scim_guard_repeated_trigger` | STANDARD | 1yr | ≥ 3 blocks in one calendar day (GUARD-CHAIN-01 advisory) |
| `system.scim_guard_cfg_cache_stale` | LOW | 1yr | `billing-event-relay` KV delete failed (missed invalidation) |

**Contracted-seats source:** `enterprise_contracts.contracted_seats` (authoritative billing record). Cached at `scim:guard_cfg:{tenantId}` with 3600s safety-net TTL + active invalidation on `billing.seats_expanded` / `billing.seats_reduced` DEC-030 events. See `docs/DECISION_LOG.md DEC-069`.

**Override protocol:** Two-person gate — `tenant_owner` submits via Admin Dashboard; CSM countersigns via internal admin console (PAM `read_write` session, §24). TTL: 1h/2h/4h (CSM selects). Auto-revoked after first over-limit request under override. Override raw reason text never enters the chain; only `reason_hash` SHA-256.

---

## §3 Three-Step SOC 2 Audit Traceability Reconstruction

A CC6.3 auditor verifying a `scim.bulk_deprovision_blocked` event at time **T** can reconstruct the exact guard limit using immutable records without querying live `tenant_users` state:

### Step 1 — Contracted seats at T

```sql
SELECT contracted_seats
FROM enterprise_contracts
WHERE tenant_id = :tenant_id
  AND effective_from <= :T
ORDER BY effective_from DESC
LIMIT 1;
```

This returns the authoritative seat count from the billing record at block time T.

### Step 2 — Threshold at T

```sql
SELECT payload->>'new_threshold_pct' AS threshold_pct
FROM audit_log_events
WHERE event_type = 'scim.bulk_deprovision_threshold_updated'
  AND payload->>'tenant_id' = :tenant_id
  AND created_at <= :T
ORDER BY created_at DESC
LIMIT 1;
```

If no `threshold_updated` event exists before T, the default threshold is **20%** (DEC-066 decision).

### Step 3 — Guard limit

```
guard_limit = ceil(contracted_seats × threshold_pct / 100)
```

**Shortcut:** The `scim.bulk_deprovision_blocked` event `payload.contracted_seats` field records the value at block time directly. Auditor can verify this field against the §3 Step 1 reconstruction. A non-zero result from the §98.2 assertion SQL (§4.2 below) indicates a mismatch that requires compliance-officer review.

**Auditor narrative for CC6.3:** The guard limit at any block event is reconstructable from two immutable records: the `enterprise_contracts` billing row and the preceding DEC-030 `scim.bulk_deprovision_threshold_updated` event. The `billing.seats_reduced` event-driven KV invalidation ensures the guard reflected the current contract within seconds of any seat reduction; the 3600s TTL provides a documented maximum staleness bound.

---

## §4 GUARD-E-001 Collection SQL

### §4.1 Primary Export (run as `form_system`, role `compliance_reviewer`)

Replace `:quarter_start` and `:quarter_end` with UTC ISO-8601 quarter boundaries.

```sql
-- GUARD-E-001: SCIM Bulk Deprovision Guard quarterly export
-- Run as: form_system (compliance_reviewer SELECT)
SELECT
  event_id,
  event_type,
  payload->>'tenant_id'            AS tenant_id,
  payload->>'deprovisioned_count'  AS deprovisioned_count,
  payload->>'contracted_seats'     AS contracted_seats,
  payload->>'threshold_pct'        AS threshold_pct,
  payload->>'window_key'           AS window_key,
  payload->>'scim_source'          AS scim_source,
  payload->>'override_token_hash'  AS override_token_hash,
  payload->>'auto_revoked'         AS auto_revoked,
  payload->>'old_threshold_pct'    AS old_threshold_pct,
  payload->>'new_threshold_pct'    AS new_threshold_pct,
  payload->>'trigger_count'        AS trigger_count,
  hmac_chain_hash,
  created_at
FROM audit_log
WHERE event_type IN (
  'scim.bulk_deprovision_blocked',
  'scim.bulk_deprovision_override_issued',
  'scim.bulk_deprovision_override_used',
  'scim.bulk_deprovision_threshold_updated',
  'system.scim_guard_repeated_trigger'
)
  AND created_at >= :quarter_start
  AND created_at <  :quarter_end
ORDER BY created_at;
```

### §4.2 Contracted-Seats Traceability Assertion (CC6.3 · run alongside §4.1)

**Expected: 0 rows.** Non-zero rows require compliance-officer review before filing.

```sql
-- DEC-069 §35.7 CC6.3 traceability assertion
-- For every scim.bulk_deprovision_blocked event in the quarter,
-- verify payload.contracted_seats matches enterprise_contracts at block time.
-- Run as: form_system (compliance_reviewer SELECT on audit_log_events + enterprise_contracts)
SELECT
  ale.event_id,
  ale.created_at,
  ale.payload->>'tenant_id'               AS tenant_id,
  (ale.payload->>'contracted_seats')::int  AS payload_contracted_seats,
  ec.contracted_seats                      AS billing_contracted_seats
FROM audit_log_events ale
JOIN LATERAL (
  SELECT contracted_seats
  FROM enterprise_contracts
  WHERE tenant_id = (ale.payload->>'tenant_id')::uuid
    AND effective_from <= ale.created_at
  ORDER BY effective_from DESC
  LIMIT 1
) ec ON true
WHERE ale.event_type = 'scim.bulk_deprovision_blocked'
  AND ale.created_at >= :quarter_start
  AND ale.created_at <  :quarter_end
  AND (ale.payload->>'contracted_seats')::int <> ec.contracted_seats;
-- Zero rows = assertion passes.
-- Possible causes for non-zero: (a) contract amendment within 5-min rolling window,
-- (b) billing-event-relay KV miss lasting > 3600s TTL safety-net,
-- (c) enterprise_contracts effective_from history gap.
```

### §4.3 Stale-Advisory Cross-Check (CC7.2 · run alongside §4.1)

**Expected: 0 rows.** Non-zero indicates `billing-event-relay` KV delete failures.

```sql
-- CC7.2 stale-advisory count: billing-event-relay KV delete failures during the quarter
-- Run as: form_audit (read-only)
SELECT
  payload->>'tenant_id'  AS tenant_id,
  COUNT(*)               AS stale_events,
  MIN(created_at)        AS first_stale_at,
  MAX(created_at)        AS last_stale_at
FROM audit_log_events
WHERE event_type = 'system.scim_guard_cfg_cache_stale'
  AND created_at >= :quarter_start
  AND created_at <  :quarter_end
GROUP BY payload->>'tenant_id'
ORDER BY stale_events DESC;
-- Zero rows = advisory path operational, no missed invalidations.
-- Non-zero: review whether any scim.bulk_deprovision_blocked event fired during the ≤ 3600s
-- stale window; if blocked_event.contracted_seats < actual contracted_seats, note in filing.
```

---

## §5 Filing Procedure

### §5.1 Standard filing (events present)

1. Run §4.1 primary export SQL → save as `GUARD-E-001_<YYYY-QN>.csv`
2. Run §4.2 assertion SQL → confirm 0 rows (if non-zero, resolve before filing)
3. Run §4.3 stale-advisory SQL → document count; note in filing if non-zero
4. File at `compliance/evidence/scim-guard/GUARD-E-001_<YYYY-QN>.csv`
5. Compute SHA-256 of the CSV and append to `compliance/evidence/MASTER-INDEX`

### §5.2 Zero-event attestation (no guard events in quarter)

If §4.1 returns 0 rows, file an attestation JSON instead of a CSV:

```json
{
  "artefact": "GUARD-E-001",
  "quarter": "<YYYY-QN>",
  "query_run_at": "<ISO-8601 UTC>",
  "blocked_count": 0,
  "override_issued_count": 0,
  "override_used_count": 0,
  "repeated_trigger_count": 0,
  "attestation": "Zero SCIM Bulk Deprovision Guard events in quarter. Guard active — confirmed by most recent scim.bulk_deprovision_threshold_updated event prior to quarter start.",
  "monitoring_pipeline_reference": "<most recent guard event_id prior to quarter, or 'none' if pre-GA>"
}
```

File the JSON at `compliance/evidence/scim-guard/GUARD-E-001_<YYYY-QN>.json` and append its SHA-256 to MASTER-INDEX.

---

## §6 SOC 2 Criteria Mapping

| Criterion | Narrative | Evidence role |
|---|---|---|
| **CC6.3** | Preventive control blocking unauthorised bulk access revocation. `scim.bulk_deprovision_blocked` events confirm guard fired; override pair confirms two-person exception path is auditable. Verify `override_used_count ≤ override_issued_count` and `auto_revoked: true` on all `override_used` events. | §4.1 export + §4.2 assertion |
| **A1.2** | Environmental threat detection — guard blocks H3-class SCIM incidents before availability impact. Complementary to reactive AL-SCIM-MASS-01. Verify threshold values are within 5–100 range. | §4.1 export (blocked events) |
| **CC7.2** | Proactive monitoring via `system.scim_guard_repeated_trigger` (GUARD-CHAIN-01). Repeated triggers (≥ 3/day) route P2 alert to CSM for IdP investigation. Stale-advisory monitors billing-event-relay KV path. | §4.1 (advisory events) + §4.3 |
| **CC9.2** | Third-party IdP risk management. Guard treats IdP as untrusted upstream. Two-person override protocol (CSM PAM + tenant_owner) documents human authorisation. `auto_revoked: true` prevents override reuse. | §4.1 export (override events) |

---

## §7 Privacy Floor Attestation

All GUARD-E-001 collection outputs satisfy the following privacy floor:

| Field category | Status |
|---|---|
| `user_id` of any deprovisioned employee | **ABSENT** — payloads contain only aggregate counts |
| Employee name, email, or handle | **ABSENT** — `override_issued_by_email_hash` is SHA-256(lowercase(email)) only |
| Health data, coaching content | **ABSENT** — guard operates at the access-control layer, not the data layer |
| GDPR Art. 9 special category data | **ABSENT** |
| `tenant_id` | **PRESENT** — FORM-internal UUID; no individual-identity dimension |
| Aggregate integers | **PRESENT** — `contracted_seats`, `deprovisioned_count`, `threshold_pct`, `stale_events` |

`form_api` has zero-row RLS on `tenant_sso_configs.bulk_deprovision_*` columns (migration 0079). Stale-advisory events carry only `tenant_id` and a timestamp. Override events carry `reason_hash` SHA-256 only; raw reason text is never stored.

---

## §8 Cross-References

| Document | Relevance |
|---|---|
| `docs/SSO_SCIM_IMPLEMENTATION.md §34` | BDG full design, migration 0079 DDL, KV counter pattern, override protocol |
| `docs/SSO_SCIM_IMPLEMENTATION.md §35` | `getGuardConfig()` seat source (DEC-069), §35.7 three-step reconstruction protocol |
| `docs/SOC2_READINESS.md §91` | GUARD-E-001 primary registration, criteria mapping, filing checklist |
| `docs/SOC2_READINESS.md §98` | CC6.3 traceability supplement + stale-advisory CC7.2 cross-check |
| `docs/AUDIT_LOG_SCHEMA.md §SCIM Bulk Deprovision Guard` | Six DEC-030 event definitions (v2.19) |
| `docs/DECISION_LOG.md DEC-066` | Adoption decision: per-tenant threshold, 5-min window, CSM countersign |
| `docs/DECISION_LOG.md DEC-069` | Seat source: `enterprise_contracts.contracted_seats` + 3600s KV TTL |
| `docs/INCIDENT_RESPONSE.md R-24` | Reactive layer (AL-SCIM-MASS-01) — complementary to preventive BDG |

---

*v1.0 (2026-06-20): Initial authoring. Closes `docs/SSO_SCIM_IMPLEMENTATION.md §35.9` item 5 (P1/M13) and `docs/SOC2_READINESS.md §98.5` item 3. Sections: (a) BDG purpose and architecture (§2); (b) §35.7 three-step reconstruction protocol with psql queries (§3); (c) GUARD-E-001 full collection SQL — primary §4.1 + §98.2 contracted-seats assertion §4.2 + §98.3 stale-advisory §4.3 (§4); (d) zero-event attestation procedure (§5.2); (e) privacy floor attestation (§7). Indexed in `compliance/evidence/auditor-onboarding/README.md`. Owner: compliance-officer.*
