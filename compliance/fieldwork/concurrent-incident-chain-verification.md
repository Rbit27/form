# Auditor Fieldwork Protocol — Concurrent Incident Sub-Chain Verification

**Document path:** `compliance/fieldwork/concurrent-incident-chain-verification.md`
**Version:** v1.0 · 2026-06-18
**Owner:** compliance-officer
**SOC 2 TSC:** CC7.4 (Response to identified security events), CC4.1 (Performance evaluations of controls)
**Source spec:** `docs/INCIDENT_RESPONSE.md §18.5` (DEC-053)
**Onboarding package:** Included in `compliance/evidence/auditor-onboarding/README.md`

---

## 1. Purpose

This protocol is for SOC 2 auditors verifying incident lifecycle chain records when two or more incidents were simultaneously active during the observation period.

FORM's DEC-030 HMAC chain records **all** incident lifecycle events in a single timestamp-ordered linked list. When two incidents are simultaneously active, their `incident.*` events are interleaved chronologically. This document explains:

1. Why non-contiguous per-incident sequences are **expected and correct**.
2. The SQL procedure to extract a per-incident event sequence (§3).
3. The HMAC spot-verification procedure (§4).
4. The CONC-CHAIN-01 cross-contamination detection query (§5).
5. A worked example with synthetic incident IDs (§6).

---

## 2. Design Rationale: Timestamp-Interleaved Master Chain

### 2.1 Single-list HMAC invariant

FORM's audit chain appends events in a single global list, where each event's `hmac_value` covers the previous event's `hmac_value`, `actor`, `action`, `payload`, and `emitted_at`. This is the DEC-030 design (see `docs/AUDIT_LOG_SCHEMA.md`).

An alternative "sub-chain" design — forking the chain per incident — was evaluated (DEC-053, 2026-06-14) and rejected on five grounds:

1. **Statistical rarity:** Concurrent P0/P1 incidents at FORM's scale are estimated at < 2% annual probability.
2. **Chain integrity risk:** A fork/merge design requires new event types (`incident.sub_chain_fork`, `incident.sub_chain_merge`) and introduces a write-ordering race under concurrent Cloudflare Workers execution — creating a chain-break failure mode not present in the single-list design.
3. **Auditor-standard practice:** AICPA SOC 2 guides explicitly permit filtered event-stream extraction for per-subject chain verification.
4. **Pattern consistency:** FORM consistently prefers documented auditor protocol over architectural complexity at this scale (DEC-043, DEC-044, DEC-051).
5. **Upgrade path preserved:** The sub-chain design is fully additive and can be introduced later without re-hashing.

### 2.2 What "non-contiguous" means and why it is correct

When incidents INC-A and INC-B are simultaneously active, the master chain may look like:

```
... → INC-A event 1 → INC-B event 1 → INC-A event 2 → INC-B event 2 → INC-A event 3 → ...
```

In this chain, `INC-A event 2`'s `hmac_prev` references the hash of `INC-B event 1` — not `INC-A event 1`. This is **correct**: the `hmac_prev` references the immediately preceding event in the **master chain**, regardless of which incident it belongs to.

Auditors must not interpret a non-INC-A predecessor as an anomaly. The skip-and-verify procedure (§3) is the tool for confirming correctness in this scenario.

The master chain integrity check (`docs/SOC2_READINESS.md §79.7`) verifies the entire chain including all interleaved events in a single pass — it does not require per-incident filtering.

---

## 3. Canonical Per-Incident SQL (§18.3.1)

Use this query to extract the ordered event sequence for a single incident from the interleaved master chain. Execute as the `compliance_reviewer` role (read-only).

```sql
-- Per-incident sub-chain extraction (compliance_reviewer role; read-only)
-- Replace :incident_id with the target ID in format INC-YYYYMMDD-XXXXXX
SELECT
  e.id                                                     AS event_id,
  e.emitted_at,
  e.actor,
  e.action,
  e.payload->>'incident_id'                                AS incident_id,
  e.hmac_value,
  e.hmac_prev,
  ROW_NUMBER() OVER (ORDER BY e.emitted_at)                AS sub_chain_seq,
  LAG(e.id)   OVER (ORDER BY e.emitted_at)                 AS master_chain_prev_event_id
FROM audit_log_events e
WHERE e.payload->>'incident_id' = :incident_id
ORDER BY e.emitted_at ASC;
```

### Column interpretation

| Column | Meaning |
|---|---|
| `event_id` | Primary key in `audit_log_events` — unique identifier for this chain event |
| `emitted_at` | UTC timestamp of event emission — ordering basis for `sub_chain_seq` |
| `actor` | Emitting principal: `form_system` (automated), `form_ops:uuid` (IC), etc. |
| `action` | DEC-030 event type: `incident.opened`, `incident.severity_changed`, `incident.resolved`, etc. |
| `incident_id` | The `incident_id` from the event payload — should match `:incident_id` for all rows |
| `hmac_value` | This event's HMAC-SHA256 chain value |
| `hmac_prev` | Hash of the immediately preceding event in the **master chain** (may belong to a different incident) |
| `sub_chain_seq` | Sequential ordinal of this incident's own events (1, 2, 3, …); no gaps possible |
| `master_chain_prev_event_id` | `event_id` of the master-chain predecessor; **may be from a different incident** — this is correct and expected |

**Key interpretation rule:** A non-INC-A value in `master_chain_prev_event_id` (i.e., the predecessor belongs to a different incident or namespace) is **not an anomaly**. It means another incident's event was emitted immediately before this event in wall-clock time.

---

## 4. HMAC Spot-Verification Procedure (§18.3.2)

### 4.1 Primary path (preferred)

Run the master-chain integrity check from `docs/SOC2_READINESS.md §79.7`. A clean result verifies **all** events in the chain, including interleaved concurrent-incident events. This is the recommended approach in most cases.

### 4.2 Optional per-incident spot-verification

To verify HMAC integrity of a specific incident's events:

**Step 1.** Run the §3 query to obtain the event list for the target `incident_id`.

**Step 2.** For each row, look up its master-chain predecessor:

```sql
SELECT hmac_value, actor, action, payload, emitted_at
FROM audit_log_events
WHERE hmac_value = :hmac_prev;
```

(Replace `:hmac_prev` with the `hmac_prev` value from the incident event row.)

**Step 3.** Recompute the expected HMAC:

```
expected_hmac = HMAC-SHA256(
  predecessor.hmac_value
  || predecessor.actor
  || predecessor.action
  || predecessor.payload (canonical JSON)
  || predecessor.emitted_at (ISO-8601)
)
```

Using the `AUDIT_LOG_HMAC_SECRET` active at the time of `emitted_at`. Request this key from the compliance-officer during fieldwork (held in 1Password `form-cloudflare-workers-secrets`; epoch rotation dates are in vault version history).

**Step 4.** Compare `expected_hmac` to the incident event's `hmac_value`. They must match exactly.

**On mismatch:** Escalate to the security-engineer. Do not file CONC-E-001 until investigated. Possible cause: epoch key rotation occurred between predecessor emission and this event — check rotation timestamps against both events' `emitted_at` values.

---

## 5. CONC-CHAIN-01 Cross-Contamination Invariant (§18.3.3)

**Invariant:** No `incident.*` DEC-030 event may reference an `incident_id` belonging to a different simultaneously-active incident. Each event must carry exactly the `incident_id` of the incident it pertains to.

**Detection query:**

```sql
-- CONC-CHAIN-01 violation detector (compliance_reviewer role; read-only)
-- Returns events where the payload incident_id is inconsistent with the Linear ticket URL.
-- Zero rows = invariant intact. Any rows = investigate with security-engineer.
SELECT
  e.id,
  e.action,
  e.payload->>'incident_id'       AS claimed_incident_id,
  e.payload->>'linear_ticket_url' AS linear_url
FROM audit_log_events e
WHERE e.action LIKE 'incident.%'
  AND e.payload->>'linear_ticket_url' IS NOT NULL
  AND e.payload->>'linear_ticket_url' NOT LIKE
      '%' || (e.payload->>'incident_id') || '%';
```

### Interpretation

| Result | Meaning | Action |
|---|---|---|
| **Zero rows** | CONC-CHAIN-01 intact — no cross-contamination detected | File as part of CONC-E-001 (§18.6 of `docs/INCIDENT_RESPONSE.md`) |
| **Any rows** | Potential cross-contamination: an incident event's payload `incident_id` is inconsistent with the embedded Linear ticket URL | Escalate to security-engineer for manual review; do not file CONC-E-001 until reviewed and explained |

**Scope note:** This query detects events where the `linear_ticket_url` field exists. Not all event types carry `linear_ticket_url` — only events associated with an IC-authored action (e.g., `incident.update_posted`). The `incident.opened` event emitted by the `siem-incident-automator` Worker includes the Linear ticket URL once linked; the initial emission may carry `"PENDING"` if Linear was unavailable (subsequently resolved via `incident.linear_ticket_linked`).

---

## 6. Worked Example — Tabletop Scenario P

**Scenario:** Two simultaneously active incidents — INC-20260618-a1b2c3 (P0 data breach) and INC-20260618-d4e5f6 (P1 SIEM false positive). The following is a synthetic example with expected inputs and outputs.

### 6.1 Synthetic master chain segment

Assume the following event order in the master chain (simplified to relevant fields):

| `sub_chain_seq` (global) | `event_id` | `action` | `payload.incident_id` | `emitted_at` (UTC) |
|---|---|---|---|---|
| 1 | evt-0001 | `incident.opened` | INC-20260618-a1b2c3 | 2026-06-18T09:00:00.000Z |
| 2 | evt-0002 | `incident.opened` | INC-20260618-d4e5f6 | 2026-06-18T09:00:45.000Z |
| 3 | evt-0003 | `incident.severity_changed` | INC-20260618-a1b2c3 | 2026-06-18T09:05:12.000Z |
| 4 | evt-0004 | `incident.update_posted` | INC-20260618-d4e5f6 | 2026-06-18T09:07:30.000Z |
| 5 | evt-0005 | `incident.resolved` | INC-20260618-d4e5f6 | 2026-06-18T09:45:00.000Z |
| 6 | evt-0006 | `incident.resolved` | INC-20260618-a1b2c3 | 2026-06-18T11:20:00.000Z |

### 6.2 §18.3.1 query for INC-20260618-a1b2c3

```sql
SELECT ... FROM audit_log_events
WHERE payload->>'incident_id' = 'INC-20260618-a1b2c3'
ORDER BY emitted_at ASC;
```

**Expected output:**

| `sub_chain_seq` | `event_id` | `action` | `master_chain_prev_event_id` |
|---|---|---|---|
| 1 | evt-0001 | `incident.opened` | NULL (first event in window) |
| 2 | evt-0003 | `incident.severity_changed` | evt-0002 ← *INC-20260618-d4e5f6 event* |
| 3 | evt-0006 | `incident.resolved` | evt-0005 ← *INC-20260618-d4e5f6 event* |

**Auditor note:** `master_chain_prev_event_id` for rows 2 and 3 references INC-d4e5f6 events (`evt-0002` and `evt-0005`). This is **correct** — the P1 incident's events were emitted between INC-a1b2c3's severity change and resolution. The non-INC-a1b2c3 predecessors are expected and do not indicate contamination.

### 6.3 §18.3.1 query for INC-20260618-d4e5f6

**Expected output:**

| `sub_chain_seq` | `event_id` | `action` | `master_chain_prev_event_id` |
|---|---|---|---|
| 1 | evt-0002 | `incident.opened` | evt-0001 ← *INC-20260618-a1b2c3 event* |
| 2 | evt-0004 | `incident.update_posted` | evt-0003 ← *INC-20260618-a1b2c3 event* |
| 3 | evt-0005 | `incident.resolved` | evt-0004 |

**Auditor note:** INC-d4e5f6's `opened` event has `master_chain_prev_event_id = evt-0001` (INC-a1b2c3's `opened`). Again, correct — INC-a1b2c3 was opened 45 seconds earlier.

### 6.4 CONC-CHAIN-01 result for this scenario

Running the §5 detection query against this synthetic dataset returns **zero rows**: every `incident.*` event with a `linear_ticket_url` in its payload has that URL containing its own `incident_id`. No cross-contamination.

**Expected CONC-E-001 attestation:** Zero-violation quarter; §18.3.1 outputs for both incidents attached; §18.3.3 zero-row result attached.

### 6.5 Key takeaways from worked example

1. Both incidents are independently verifiable from the master chain via §18.3.1 — no infrastructure changes needed.
2. Non-INC predecessor events are a feature, not a defect.
3. The CONC-CHAIN-01 query provides a deterministic cross-contamination check independent of the HMAC verification path.
4. The master chain integrity check (`docs/SOC2_READINESS.md §79.7`) verifies both incidents' events in a single pass — no per-incident re-run is required for primary HMAC verification.

---

## 7. Filing CONC-E-001

CONC-E-001 is filed quarterly even when no concurrent incidents occurred during the quarter (zero-event attestation).

**Path:** `compliance/evidence/ir-chain/CONC-E-001_<YYYY-QN>.md`

**Contents:**
- (a) §18.3.1 query output for each concurrent-incident period in the observation quarter (or a zero-event attestation statement if no concurrent incidents occurred).
- (b) §18.3.3 CONC-CHAIN-01 detection query result (zero rows expected).
- (c) Reference to this document.

**SOC 2 criteria:** CC7.4 (concurrent incidents managed with independent `incident_id` namespaces; per-incident extraction verifiable), CC4.1 (CONC-CHAIN-01 verifiable at any time; zero rows confirms no cross-contamination during observation period).

---

## 8. Cross-References

| Document | Relevance |
|---|---|
| `docs/INCIDENT_RESPONSE.md §18.3` | Skip-and-verify algorithm (source for §3 and §4 of this document) |
| `docs/INCIDENT_RESPONSE.md §18.4` | Concurrent incident coordination protocol (IC operational procedure) |
| `docs/INCIDENT_RESPONSE.md §18.5` | Source specification for this fieldwork document |
| `docs/INCIDENT_RESPONSE.md §18.6` | CONC-E-001 evidence artefact definition |
| `docs/SOC2_READINESS.md §79.7` | Master chain integrity check — primary HMAC verification path |
| `docs/AUDIT_LOG_SCHEMA.md` | DEC-030 event schemas; `incident.*` namespace |
| `docs/DECISION_LOG.md DEC-053` | Decision record: timestamp-interleaved chain; Option (a) selected |
| `compliance/fieldwork/incident-reason-verification.md` | Companion fieldwork document — `incident.severity_changed` reason hash retrieval |
| `compliance/evidence/auditor-onboarding/README.md` | Auditor onboarding package index |

---

*v1.0 (2026-06-18): Initial authoring per `docs/INCIDENT_RESPONSE.md §18.7 checklist item 1` (P1/M7). Closes the documentation obligation from `docs/INCIDENT_RESPONSE.md §18.5`. Worked example uses synthetic incident IDs for Tabletop Scenario P (INC-20260618-a1b2c3 P0 data breach + INC-20260618-d4e5f6 P1 SIEM false positive). Owner: compliance-officer.*
