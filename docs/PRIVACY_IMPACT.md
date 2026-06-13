# FORM · Privacy Impact Assessment Process v1.0

> **Document type:** Privacy Impact Assessment (PIA) process, constraint registry, and living log  
> **Owner:** `compliance-officer` (primary) · `clinical-safety` (veto authority) · `security-engineer` (technical controls) · `enterprise-architect` (data-layer enforcement)  
> **Status:** v1.0 — active  
> **Review cadence:** Annual (Q1 January, aligned with `docs/SOC2_READINESS.md §15` compliance calendar) plus ad-hoc on any proposed change to a privacy-hard-constraint listed in §2  
> **References:** `docs/ENTERPRISE.md`, `docs/GDPR_DPIA.md`, `docs/AUDIT_LOG_SCHEMA.md`, `docs/OBSERVABILITY.md §24–25`, `docs/SOC2_READINESS.md §35`, `docs/DATA_MODEL.md`, `docs/INCIDENT_RESPONSE.md`, DEC-030, DEC-018  
> **Gap closure:** `docs/SOC2_READINESS.md` P-GAP-008 (annual privacy review scope checklist · P8.1)

---

## Table of Contents

1. [Why This Document Exists](#1-why-this-document-exists)
2. [Privacy Constraint Registry](#2-privacy-constraint-registry)
3. [PIA Trigger Criteria](#3-pia-trigger-criteria)
4. [PIA Process](#4-pia-process)
5. [Clinical-Safety Veto Procedure](#5-clinical-safety-veto-procedure)
6. [PIA Template](#6-pia-template)
7. [DEC-030 Audit Events](#7-dec-030-audit-events)
8. [SOC 2 P-Series Mapping](#8-soc-2-p-series-mapping)
9. [Annual Privacy Review Integration](#9-annual-privacy-review-integration)
10. [PIA Register](#10-pia-register)
11. [Implementation Checklist](#11-implementation-checklist)
12. [Open Questions](#12-open-questions)

---

## 1. Why This Document Exists

`docs/GDPR_DPIA.md` is the one-time formal Data Protection Impact Assessment required by GDPR Art. 35 before high-risk processing begins. It is a static artefact that documents the initial risk assessment and is updated annually.

This document serves a different purpose: it is the **ongoing change management layer** for FORM's privacy-hard-constraints. When any engineer, product manager, enterprise customer, or founder proposes a change to how FORM collects, processes, exposes, or retains personal data — and that proposal touches a constraint listed in §2 — a Privacy Impact Assessment under this process is mandatory before the change ships.

**The relationship between the three privacy documents:**

| Document | Purpose | Trigger | Owner |
|---|---|---|---|
| `docs/GDPR_DPIA.md` | Initial statutory DPIA; lawful basis per processing activity | Required by GDPR Art. 35 before processing begins | compliance-officer |
| `docs/PRIVACY_POLICY.md` | User-facing notice of processing | Required by GDPR Art. 13/14 | compliance-officer |
| **`docs/PRIVACY_IMPACT.md` (this doc)** | Internal PIA process for changes to hard constraints; living log | Triggered by proposed product or architectural changes | compliance-officer + clinical-safety |

**Why the hard-constraint framing matters for enterprise.** FORM's enterprise privacy floor is a competitive differentiator and a legal commitment in customer DPAs. Enterprise customers sign agreements that include the privacy floor from `docs/ENTERPRISE.md §Privacy floor for enterprise`. If a change relaxes a hard constraint, it may constitute a material contract breach with every enterprise customer who signed a DPA referencing those commitments. The PIA process exists to make that consequence visible before code ships.

---

## 2. Privacy Constraint Registry

The following constraints are **non-negotiable hard requirements**, not guidelines. They are enforced at the data layer (RLS policies, materialized view definitions, API response filters) and repeated in customer DPAs. Any proposed relaxation requires a PIA (§4) and clinical-safety veto review (§5).

### 2.1 Enterprise Privacy Floor (from `docs/ENTERPRISE.md §Privacy floor`)

| Constraint ID | Constraint | Data-layer enforcement | Clinical-safety veto scope |
|---|---|---|---|
| **EF-01** | HR can never access individual user data — only opt-in aggregates | `tenant_aggregates` materialized view; `data.read_individual` DEC-030 alert; no individual-level API endpoint exposed to `tenant_admin` or `tenant_manager` roles | No |
| **EF-02** | Aggregate metrics require k-anonymity N ≥ 5 — groups below five are suppressed | Materialized view `WHERE count >= 5`; `data.aggregate_below_threshold` DEC-030 event on suppressed query | No |
| **EF-03** | Manager-direct-report drill-down is not a permitted feature — no "my team" individual view | RBAC: `tenant_manager` role has no `SELECT` on any individual user health table | No |
| **EF-04** | Employee can revoke company association at any time; revocation takes effect within 24 hours | `user_tenants.status = 'revoked'`; RLS excludes revoked rows from all tenant queries | No |
| **EF-05** | ED-screening responses (DEC-018) are never aggregated to the employer | `ed_screening` table excluded from all tenant-facing aggregation queries; `clinical-safety` hard veto | **Yes — clinical-safety hard veto** |
| **EF-06** | Body composition data (weight, body fat %, measurements) is never aggregated to the employer | Body composition columns excluded from `tenant_aggregates`; no employer-facing API field | **Yes — clinical-safety hard veto** |
| **EF-07** | Mental health and mood data is never aggregated to the employer | `mood_score`, `mental_health_flag`, subjective wellbeing fields excluded from tenant-facing views | **Yes — clinical-safety hard veto** |

### 2.2 Observability and Analytics Privacy Constraints (from `docs/OBSERVABILITY.md §24–25`)

| Constraint ID | Constraint | Enforcement point |
|---|---|---|
| **OC-01** | No raw webhook payloads stored — only SHA-256 hash | `subscription_events.raw_payload_hash` only; raw body discarded after hash + processing |
| **OC-02** | No payment card data or billing addresses in FORM's Supabase database | Stripe/RevenueCat handle all payment data; FORM stores only subscription state and plan identifiers |
| **OC-03** | DEC-030 billing events carry `user_id_hash` (SHA-256), not plaintext `user_id` | Audit log schema enforced at event emission layer |
| **OC-04** | No raw health metrics in PostHog — weight, load, body composition, RPE, body measurements never as event properties | `stripPersonalProperties()` called in mobile SDK layer before PostHog call; CI lint rule enforces field blocklist |
| **OC-05** | No free-text content (Victor conversation turns, custom notes, user input) in PostHog | Same `stripPersonalProperties()` gate; length-bucket proxies permitted |
| **OC-06** | No CV keypoints or pose data in PostHog — no frame-level coordinates, numerical form scores, or per-rep quality assessments | Boolean `cv_enabled` permitted; numerical scores prohibited |
| **OC-07** | PostHog `distinct_id` is `sha256(user_id)` — never the plaintext Supabase UUID | Computed in mobile app before SDK call; PostHog itself never receives plaintext UUID |
| **OC-08** | `readiness_score` (1–5) raw value: **prohibited** (PostHog). `readiness_bucket` (low\|medium\|high): **conditionally permitted** per OQ-33 clinical-safety ruling 2026-06-12 — see `docs/OBSERVABILITY.md §25.9 OQ-33 Ruling` for the full condition set; conditions are binding and non-negotiable. `mood_score` (1–5) and `mood_bucket` (low\|medium\|high): **prohibited — clinical-safety VETO** per OQ-33 ruling 2026-06-12; this is a permanent prohibition absent a new ruling. | SDK `stripPersonalProperties()` explicit allowlist for `readiness_bucket`; explicit blocklist for `mood_score`, `mood_bucket`, `readiness_score`; CI lint rule fails build on any PostHog `.capture()` call containing a prohibited property name |

### 2.3 Data Model Isolation Constraints (from `docs/DATA_MODEL.md`)

| Constraint ID | Constraint | Enforcement point |
|---|---|---|
| **DM-01** | Row-level security isolates all tenant data — no cross-tenant query without explicit grant | Postgres RLS policy on every table with `tenant_id`; CI cross-tenant tests block policy regressions |
| **DM-02** | Tenant feature flags are tenant-scoped — no global flag can enable a constraint relaxation for a subset of tenants | Feature flag schema requires `tenant_id`; global flag changes require PIA if they affect any constraint in §2 |

---

## 3. PIA Trigger Criteria

A PIA under this process is required **before merging** any change that:

| Trigger | Examples | Required PIA type |
|---|---|---|
| **T-1 Constraint relaxation proposal** | Proposal to expose individual user data to HR dashboard; proposal to add body composition to tenant_aggregates | Full PIA (§4) + clinical-safety veto (§5) where marked |
| **T-2 New data collection** | New biometric field added to health profile; new wearable metric streamed to Supabase; new question in ED-screening | Full PIA (§4); GDPR_DPIA.md delta review also required |
| **T-3 New sub-processor receiving Art. 9 data** | New AI inference provider receiving health context; new analytics vendor receiving event stream | Full PIA (§4); also triggers `docs/SOC2_READINESS.md §17` vendor review |
| **T-4 New enterprise feature touching aggregation** | New "team wellness report" feature; new HR dashboard widget; new export format | Full PIA (§4) |
| **T-5 Retention period change** | Shortening or extending any retention period in `docs/SOC2_READINESS.md §35.6.3` | Lightweight PIA (§6.2) |
| **T-6 Consent model change** | Adding a new consent category; changing granularity of an existing consent; deprecating a consent category | Full PIA (§4); GDPR_DPIA.md delta review also required |
| **T-7 Government or law enforcement request** | Any request to disclose user data beyond the four-principle policy in `docs/SOC2_READINESS.md §35.8.2` | Full PIA (§4); legal counsel required |

**What does NOT require a PIA:**
- Bug fixes that do not change data collection, retention, or exposure logic
- Refactors that preserve existing constraints at the data layer
- Adding a new PostHog event that does not include any constraint-listed field
- New UI features that do not touch any table or API endpoint containing personal data
- Performance improvements (index additions, query rewrites) that do not alter what data is returned

When in doubt, the trigger test is: *"Does this change alter what data FORM collects, how long it keeps it, who can see it, or what sub-processors receive it?"* If yes, file a PIA.

---

## 4. PIA Process

### 4.1 Five-step process

```
Step 1 → Proposal filed (Linear ticket + PIA-ID assigned)
Step 2 → Impact scoped (which constraints affected, GDPR_DPIA delta needed?)
Step 3 → Review conducted (compliance-officer + security-engineer)
Step 4 → Clinical-safety veto review (mandatory for EF-05, EF-06, EF-07)
Step 5 → Decision recorded (PIA Register §10 + DEC-030 event emitted)
```

### 4.2 Step-by-step detail

**Step 1 — File a PIA proposal**

- Open a Linear ticket with label `privacy-impact-assessment`
- Assign PIA ID in format `PIA-YYYY-NNN` (e.g., `PIA-2026-001`)
- Describe the proposed change: what data, what change, what user-facing impact
- Link to the constraint IDs from §2 that are affected
- Assign to `compliance-officer`

SLA for compliance-officer to acknowledge: **2 business days**

**Step 2 — Scope the impact**

`compliance-officer` produces a one-page impact memo covering:
- Which constraint(s) from §2 are affected and how
- Whether a `docs/GDPR_DPIA.md` delta review is required (required if a new processing activity or new data category is introduced)
- Whether a `docs/SOC2_READINESS.md §17` vendor review is required (required for T-3)
- Whether clinical-safety veto review is required (mandatory for EF-05, EF-06, EF-07; recommended for any change touching mental health or ED-adjacent data)
- Risk rating: **Low / Medium / High / Critical**

| Risk rating | Criteria |
|---|---|
| **Low** | Retention period change only; no new data collected; no new exposure |
| **Medium** | New PostHog event with borderline property; new wearable metric (non-Art. 9) |
| **High** | New Art. 9 data category; new sub-processor receiving health data; consent model change |
| **Critical** | Proposed relaxation of any EF-0x constraint; any government data access request |

**Step 3 — Conduct the review**

`compliance-officer` + `security-engineer` complete the PIA template (§6) within:
- Low: 3 business days
- Medium: 5 business days
- High: 10 business days
- Critical: Immediately (same-day escalation; change blocked until PIA completed and clinical-safety veto obtained)

The review assesses:
1. Necessity and proportionality — is the change required for the stated purpose?
2. Data minimisation — can the same outcome be achieved with less data or narrower exposure?
3. Risk to data subjects — likelihood and severity of harm if the change proceeds
4. Technical mitigations — RLS, hashing, aggregation, consent gates, audit events
5. Contractual exposure — does the change breach existing enterprise DPAs?
6. Regulatory exposure — does the change affect GDPR Art. 9 obligations or AICPA P-series controls?

**Step 4 — Clinical-safety veto review**

Mandatory for all changes touching EF-05 (ED-screening), EF-06 (body composition), EF-07 (mental health), and for any change that `compliance-officer` flags as touching vulnerable-user risk.

`clinical-safety` receives the Step 3 PIA template and has:
- **48 hours** to review and issue a decision: Approve / Approve with conditions / Veto

If `clinical-safety` issues a **Veto**, the change is blocked unconditionally. The veto is recorded in the PIA Register (§10) and in the DEC-030 audit log. A veto may not be overridden by the founder, the product roadmap, or an enterprise customer contract. The only path to reconsideration is a materially revised proposal that addresses the clinical-safety concern.

If `clinical-safety` **Approves with conditions**, the conditions become implementation requirements — they must be completed before the change can ship.

**Step 5 — Record the decision**

`compliance-officer` updates the PIA Register (§10) with:
- PIA ID, date, Linear ticket link
- Change description
- Constraints affected
- Risk rating
- Clinical-safety involvement (yes/no) and outcome
- Decision: Approved / Approved with conditions / Rejected
- Conditions (if any)
- GDPR_DPIA.md delta filed? (yes/no/not required)
- DEC-030 `privacy.pia_completed` event ID

---

## 5. Clinical-Safety Veto Procedure

`clinical-safety` holds unconditional veto authority over any change that affects EF-05, EF-06, EF-07, or that is referred by `compliance-officer` as touching vulnerable-user risk.

### 5.1 Veto trigger list

The following change types automatically trigger a clinical-safety veto review regardless of compliance-officer opinion:

1. Any proposal to aggregate ED-screening responses (DEC-018) to any employer-facing view
2. Any proposal to expose body composition trends, weight history, or body fat percentage to HR or manager roles
3. Any proposal to aggregate mental health scores, mood data, or stress indicators to employer-facing views
4. Any proposal to enable manager-level "my team" health reporting, even in aggregate
5. Any change to the ED-screening instrument (SCOFF-light questions, scoring, UX flag logic) that would alter what data is collected or how it is used
6. Any change to the consent flow for Art. 9 health data that reduces the clarity or specificity of consent
7. Any request from an enterprise customer to access individual user data as a contracted feature (automatic reject — cannot be approved even with clinical-safety approval)

### 5.2 Veto decision record

When `clinical-safety` issues a veto, the following must be recorded:

- Veto ID: `VETO-YYYY-NNN`
- PIA ID it references
- Date of veto decision
- Constraint(s) protected
- Specific harm prevented
- Whether founder acknowledged the veto (DEC-030 event required)

Enterprise customers who request a feature that would require relaxing a vetoed constraint must be told: *"This is not a feature we build."* Sales may not offer it as a roadmap item or future consideration. If a signed contract contains a clause that contradicts a clinical-safety constraint, the contract clause is unenforceable and must be corrected with outside counsel.

---

## 6. PIA Template

### 6.1 Full PIA template (for T-1 through T-4, T-6, T-7)

```
PIA ID: PIA-YYYY-NNN
Date filed: YYYY-MM-DD
Proposed by: [role or name]
Linear ticket: [URL]
Change description: [one paragraph]
Trigger type(s): [T-1 / T-2 / T-3 / T-4 / T-5 / T-6 / T-7]
Constraints affected: [list from §2]
Risk rating: [Low / Medium / High / Critical]

--- NECESSITY AND PROPORTIONALITY ---
1. What is the stated purpose of the change?
2. Is there a less privacy-invasive way to achieve the same outcome?
3. What happens if FORM does not make this change?

--- DATA SUBJECT IMPACT ---
4. Who are the affected data subjects? (consumers / enterprise employees / both)
5. What is the likelihood of harm to data subjects if the change proceeds?
   (1 = remote / 2 = possible / 3 = likely / 4 = near-certain)
6. What is the severity of harm if it occurs?
   (1 = minor inconvenience / 2 = reputational harm / 3 = financial harm / 4 = physical or psychological harm)
7. Residual risk score (likelihood × severity):

--- TECHNICAL MITIGATIONS ---
8. What RLS policy changes are required?
9. What new DEC-030 audit events are required?
10. What consent gates are required?
11. What data minimisation measures are applied?

--- CONTRACTUAL AND REGULATORY EXPOSURE ---
12. Does this change affect any existing enterprise DPA? If yes, which customers and how?
13. Does this change introduce or modify a GDPR Art. 9 processing activity?
    If yes: GDPR_DPIA.md delta review is REQUIRED before approval.
14. Does this change affect any AICPA P-series control?
    If yes: identify the control and document the impact.

--- CLINICAL-SAFETY REVIEW ---
15. Is clinical-safety veto review required? (mandatory for EF-05/06/07)
16. Clinical-safety decision: [Approve / Approve with conditions / Veto]
17. Conditions or veto rationale:

--- DECISION ---
18. Compliance-officer decision: [Approve / Approve with conditions / Reject]
19. Conditions (if any):
20. If rejected: what alternative is recommended?

--- EVIDENCE ---
21. DEC-030 privacy.pia_completed event ID:
22. Filed in compliance/evidence/privacy/YYYY/:
23. GDPR_DPIA.md delta filed: [yes / no / not required]
24. SOC2_READINESS.md §17 vendor review triggered: [yes / no / not required]
```

### 6.2 Lightweight PIA template (for T-5 retention period changes only)

```
PIA ID: PIA-YYYY-NNN (Lightweight)
Date: YYYY-MM-DD
Change: [which retention period, from X to Y]
Reason: [business or regulatory justification]
Data subject impact: [who is affected and how]
GDPR Art. 5(1)(e) storage limitation compliance: [still compliant because...]
Enterprise DPA impact: [none / describe]
Decision: [Approve / Reject]
DEC-030 event ID:
```

---

## 7. DEC-030 Audit Events

All PIA lifecycle events are HMAC-chained per `docs/AUDIT_LOG_SCHEMA.md §3`. Retention: 7 years (PIA decisions are governance evidence for SOC 2 Type II auditors and GDPR Art. 5(2) accountability).

| Event type | Severity | Key metadata fields | Trigger |
|---|---|---|---|
| `privacy.pia_filed` | STANDARD | `pia_id`, `trigger_type` (T-1…T-7), `proposed_by`, `constraints_affected[]`, `linear_ticket_url`, `filed_by` (compliance-officer user_id) | Emitted at Step 1 when PIA ticket is acknowledged |
| `privacy.pia_completed` | HIGH | `pia_id`, `risk_rating`, `clinical_safety_required` (bool), `clinical_safety_decision` (`approved`/`approved_with_conditions`/`vetoed`/`not_required`), `decision` (`approved`/`approved_with_conditions`/`rejected`), `conditions[]` (list of condition strings, empty if none), `gdpr_dpia_delta_required` (bool), `completed_by` (compliance-officer user_id) | Emitted at Step 5 on final decision |
| `privacy.pia_veto_issued` | CRITICAL | `veto_id`, `pia_id`, `constraints_protected[]`, `harm_prevented` (free text), `vetoed_by` (clinical-safety agent identifier), `acknowledged_by_founder` (bool) | Emitted by clinical-safety on veto decision; must be acknowledged by founder within 24 hours |
| `privacy.constraint_relaxation_rejected` | HIGH | `pia_id`, `constraint_id` (e.g., `EF-05`), `requester_type` (`internal`/`enterprise_customer`/`government`), `rejection_reason`, `rejected_by` | Emitted whenever a constraint relaxation proposal is formally rejected, regardless of whether a full PIA was filed |
| `privacy.annual_review_scope_drafted` | STANDARD | `review_year`, `scope_items[]`, `drafted_by` (compliance-officer user_id) | Emitted when annual review scope checklist is finalised (Q1 preparation) |
| `privacy.annual_review_completed` | HIGH | `review_year`, `dpia_version_reviewed`, `pias_in_scope[]`, `gaps_opened` (int), `gaps_closed` (int), `sign_off_by` (compliance-officer user_id) | Emitted on completion of annual privacy review; triggers P8.1 evidence filing |

**HMAC chain note:** `privacy.pia_veto_issued` at CRITICAL severity triggers an immediate PagerDuty page to `compliance-officer` and `founder` (alert `PRIV-VETO-001`). This ensures no veto can be silently buried in the audit log.

---

## 8. SOC 2 P-Series Mapping

| P criterion | Control | This document's contribution |
|---|---|---|
| **P1.1** — Privacy notice | FORM communicates its privacy practices | §2 constraint registry is the internal counterpart to the user-facing Privacy Policy; PIA process ensures notices stay accurate as the product changes |
| **P3.1** — Collection limitation | Personal information collected only as necessary | §3 trigger criteria + §4 Step 3 necessity and proportionality assessment enforce collection minimisation for every change |
| **P3.2** — Explicit consent | Art. 9 consent obtained before special-category processing | §3 T-6 trigger + §4 Step 3 item 13 require GDPR_DPIA.md delta before any consent model change |
| **P4.1** — Use limitation | Personal information used only for stated purposes | §2 constraint registry defines what purposes are authorised; §4 Step 3 items 1–3 assess whether proposed changes stay within stated purposes |
| **P6.1** — Sub-processor disclosure | Third-party disclosures consistent with privacy commitments | §3 T-3 trigger mandates a PIA + vendor review for every new sub-processor receiving Art. 9 data |
| **P8.1** — Monitoring and enforcement | Ongoing privacy programme; complaint process; annual review | §9 defines the annual review scope checklist; PIA Register (§10) is the ongoing evidence log; DEC-030 events provide the operating-effectiveness trail auditors require |

**P-GAP-008 closure note.** `docs/SOC2_READINESS.md P-GAP-008` identifies: *"Annual privacy review not calendared; review scope checklist not drafted."* This document closes the scope-checklist component of P-GAP-008. The calendar entry is in `docs/SOC2_READINESS.md §15` (Q1 January). The §9 table below is the scope checklist. P-GAP-008 advances from 🔴 Open → 🟡 Authored; it closes to 🟢 when the first annual review is executed and `privacy.annual_review_completed` is emitted.

---

## 9. Annual Privacy Review Integration

The annual privacy review (Q1 January, per `docs/SOC2_READINESS.md §15`) includes a mandatory PIA programme review. This is the scope checklist for that review:

| Item | Scope | Evidence artefact |
|---|---|---|
| **PR-1** | Review PIA Register (§10) — confirm all PIAs filed in the past year were completed; identify any change that shipped without a required PIA | PIA Register export + compliance-officer attestation |
| **PR-2** | Re-assess constraint registry (§2) — confirm all constraints are still enforced at the data layer; run a cross-tenant RLS test to verify EF-01/EF-03; check materialized view definition for EF-02 k-anonymity clause | CI cross-tenant test pass + materialized view DDL screenshot |
| **PR-3** | Review clinical-safety veto log — confirm no veto was overridden or silently superseded by a subsequent change | DEC-030 `privacy.pia_veto_issued` event list for the year; cross-reference with deployed features |
| **PR-4** | Review OC-04 through OC-08 PostHog constraints — run a spot-check of PostHog event schema for top 10 event types; confirm no health fields in production event properties | PostHog schema screenshot; `stripPersonalProperties()` code version hash |
| **PR-5** | Refresh `docs/GDPR_DPIA.md` — confirm processing activities PA-01–PA-12 still reflect production; add delta notes for any new activities triggered by T-2 PIAs in the year | DPIA annual review memo (1 page minimum) |
| **PR-6** | Confirm sub-processor list accuracy — cross-reference `docs/SUBPROCESSORS.md` with production SDK integrations; verify all new sub-processors added in the year had a T-3 PIA | Sub-processor registry diff vs. prior year |
| **PR-7** | Confirm consent model coverage — verify all Art. 9 data collected in production is covered by an explicit consent category in the five-category model | Consent category audit table |
| **PR-8** | Review open PIA questions — check §12 open questions; close or escalate any that are overdue | OQ status table update |
| **PR-9** | File review output — emit `privacy.annual_review_completed` DEC-030 event; file memo + evidence in `compliance/evidence/privacy/YYYY/` | DEC-030 event ID + R2 object listing |

---

## 10. PIA Register

Living log of all PIAs. Updated at Step 5 of the PIA process (§4.2). This section is the P8.1 operating-effectiveness evidence.

| PIA ID | Date | Change description | Trigger | Constraints affected | Risk | Clinical-safety | Decision | DEC-030 event ID |
|---|---|---|---|---|---|---|---|---|
| **PIA-2026-001** | 2026-06-12 | `readiness_bucket` (low\|medium\|high) transmitted to PostHog as event property on `checkin.submitted` only — first health-adjacent property in PostHog analytics pipeline | **T-6** — new health-adjacent property in external analytics sub-processor (PostHog); also T-3 (new data use for a previously collected field) | **OC-08** — partial relaxation: CONDITIONAL PASS for `readiness_bucket`; VETO maintained for `mood_bucket` and `readiness_score` (permanent prohibition) | **Medium** — 3-bucket collapse (`low`\|`medium`\|`high`) reduces individual-level inference precision vs. raw 1–5 integer; binding conditions (event-property scope, PostHog DPA review, SDK allowlist, no A/B segmentation) reduce residual risk to acceptable level per clinical-safety ruling | **CONDITIONAL PASS** (`readiness_bucket`); **VETO — permanent prohibition** (`mood_bucket`, `mood_score`, `readiness_score` raw value); clinical-safety ruling 2026-06-12 per `docs/OBSERVABILITY.md §25.9 OQ-33 Ruling`; DEC-046 in `docs/DECISION_LOG.md` | **Approved with conditions** — `readiness_bucket` CONDITIONALLY PERMITTED on `checkin.submitted` event-property only, subject to all conditions in `docs/OBSERVABILITY.md §25.9`. Pre-conditions before any PostHog emission: (1) PostHog DPA review completed by compliance-officer confirming `readiness_bucket` within agreed sub-processor scope; (2) `readiness_bucket` added to `stripPersonalProperties()` explicit allowlist by platform-engineer in a PR reviewed by compliance-officer; (3) CI lint rule updated to blocklist `mood_bucket`, `mood_score`, `readiness_score` raw integer. Ruling expires at SOC 2 observation period start (mandatory re-review). | `privacy.pia_completed` with `pia_id = "PIA-2026-001"` — **pending** platform-engineer deployment of `privacy.pia_*` event types to `emit-audit-event` Worker (PRIVACY_IMPACT.md §11 P0/M4 item 1; AUDIT_LOG_SCHEMA.md v2.0 schema now registered) |

---

## 11. Implementation Checklist

### P0 — Before first enterprise pilot (M4)

| # | Task | Owner | Milestone | Definition of Done |
|---|---|---|---|---|
| 1 | Register `privacy.pia_filed`, `privacy.pia_completed`, `privacy.pia_veto_issued`, `privacy.constraint_relaxation_rejected`, `privacy.annual_review_scope_drafted`, `privacy.annual_review_completed` as DEC-030 event types in `docs/AUDIT_LOG_SCHEMA.md`; validate Zod schema for each; deploy to `emit-audit-event` Worker endpoint | data-engineer + platform-engineer | M4 | Six event types in AUDIT_LOG_SCHEMA.md; events emittable from admin console; test events pass in staging |
| 2 | Configure PagerDuty alert `PRIV-VETO-001` — fires on `privacy.pia_veto_issued` (CRITICAL) → pages `compliance-officer` and `founder` | devops-lead | M4 | Alert fires on test emission; confirmation screenshot filed |
| 3 | Add `privacy-impact-assessment` label to Linear; create PIA issue template with §6.1 template text pre-filled | product-manager | M4 | Label and template live in Linear; compliance-officer confirms usability |
| 4 | Add annual privacy review entry to `docs/SOC2_READINESS.md §15` compliance calendar (Q1 January, scope = §9 table above); reference this document as scope checklist source | compliance-officer | M4 | §15 calendar updated; `docs/PRIVACY_IMPACT.md` linked from calendar entry; P-GAP-008 status updated to 🟡 Authored |

### P1 — Before SOC 2 Type II observation window start (M6)

| # | Task | Owner | Milestone | Definition of Done |
|---|---|---|---|---|
| 5 | File a retroactive PIA (PIA-2026-000 or equivalent) documenting the §2 constraint registry as it exists at audit start — this establishes the baseline constraint set as a HMAC-chained record | compliance-officer | M6 | PIA-2026-000 filed; DEC-030 event emitted; filed in `compliance/evidence/privacy/2026/` |
| 6 | Brief engineering team on PIA trigger criteria (§3) — ensure every engineer knows that changes touching §2 constraints require a PIA before merging; add check to PR template | compliance-officer + devops-lead | M6 | PR template updated with PIA checklist item; team briefing documented in `compliance/cc1/security-training-log.md` |

### P2 — Post-hire (first engineering hire with production access)

| # | Task | Owner | Milestone | Definition of Done |
|---|---|---|---|---|
| 7 | Add PIA process to new hire security onboarding checklist (`docs/SOC2_READINESS.md §22`) | compliance-officer | Post-hire | §22 onboarding checklist updated with PIA module |

---

## 12. Open Questions

**OQ-PIA-01: Should the PIA process include a data subject representative review for High/Critical changes?**

For High and Critical risk PIAs, GDPR Art. 35(9) recommends seeking the views of data subjects where appropriate. FORM currently has no formal mechanism for this — the closest proxy is the `clinical-safety` review (which represents user vulnerability concerns) and the privacy floor (which represents user autonomy commitments). As the user base grows, a data subject panel or user advisory group could formally fulfil this role.

**Recommended resolution:** For now, `clinical-safety` review serves as the data-subject-representative proxy for High/Critical PIAs. At ARR ≥ $500k or user base ≥ 10,000, revisit whether a formal user advisory process is needed. Owner: `compliance-officer` + `research-lead`. Priority: **P2**.

**OQ-PIA-02: How should the PIA process handle requests from enterprise customers to add new aggregate metrics to the admin dashboard?**

Enterprise customers regularly request new admin dashboard metrics (e.g., "can we see average sleep quality by department?"). Each such request may or may not trigger a PIA depending on whether the metric involves Art. 9 data and whether it meets the k-anonymity threshold. The current process (§3 trigger T-4) captures these, but the intake path is unclear — the request typically arrives via the CSM, not via a Linear ticket from an internal engineer.

**Recommended resolution:** Add a customer dashboard feature request intake step in `docs/B2B_PLAYBOOK.md` that routes through `compliance-officer` for a T-4 trigger assessment before any response is given to the customer. The CSM should never commit to a new metric without compliance-officer sign-off. Owner: `customer-success` + `compliance-officer`. Priority: **P1 — before first enterprise pilot is in active use**.

**OQ-PIA-03: What is the correct resolution for OC-08 (readiness_score and mood_score pending clinical-safety OQ-33 ruling)?**

`docs/OBSERVABILITY.md §25.7` treats `readiness_score` and `mood_score` as prohibited PostHog properties pending OQ-33, a clinical-safety ruling on whether these numerical values are health-adjacent enough to require prohibition. This PIA process is the natural home for documenting the OQ-33 resolution — if clinical-safety approves these fields, a Lightweight PIA (§6.2) should be filed to record the approval as a HMAC-chained event before the fields are enabled in any PostHog event.

**Recommended resolution:** When OQ-33 is resolved, file `PIA-YYYY-NNN` (Lightweight, T-5/T-6 hybrid) referencing OQ-33 clinical-safety decision; emit `privacy.pia_completed` event; update OC-08 constraint status in §2.2 to reflect approved, approved-with-conditions, or permanently-prohibited. Owner: `clinical-safety` (decision) + `compliance-officer` (PIA filing). Priority: **P1 — before any PostHog event includes these fields**.

---

*v1.0 (2026-06-06): Initial document. Establishes Privacy Impact Assessment process, constraint registry (§2: 7 enterprise privacy floor constraints EF-01–EF-07, 8 observability constraints OC-01–OC-08, 2 data model constraints DM-01–DM-02), trigger criteria (§3: T-1 through T-7), five-step PIA process (§4) with risk-rated SLAs (Low 3d / Medium 5d / High 10d / Critical same-day), clinical-safety veto procedure (§5) with unconditional veto authority and seven auto-trigger change types, full PIA template (§6.1) and lightweight retention-change template (§6.2), six DEC-030 HMAC-chained event types including CRITICAL `privacy.pia_veto_issued` with PagerDuty alert `PRIV-VETO-001` (§7), SOC 2 P-series mapping P1.1/P3.1/P3.2/P4.1/P6.1/P8.1 (§8), annual privacy review scope checklist PR-1 through PR-9 (§9) — closes scope-checklist component of P-GAP-008 (🔴 → 🟡 Authored), living PIA Register (§10), implementation checklist 7 items (4× P0 M4, 2× P1 M6, 1× P2 post-hire), three open questions OQ-PIA-01 through OQ-PIA-03. Distinct from `docs/GDPR_DPIA.md` (initial statutory DPIA) — this document is the ongoing change management layer for hard constraints. Clinical-safety veto is non-overridable by founder, product roadmap, or enterprise customer contract. References: `docs/ENTERPRISE.md`, `docs/GDPR_DPIA.md`, `docs/AUDIT_LOG_SCHEMA.md`, `docs/OBSERVABILITY.md §24–25`, `docs/SOC2_READINESS.md §35`, `docs/DATA_MODEL.md`, DEC-030, DEC-018. Owner: compliance-officer + clinical-safety + security-engineer + enterprise-architect. enterprise-builder cloud worker · 2026-06-06.*