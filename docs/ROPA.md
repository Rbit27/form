# FORM · Records of Processing Activities (ROPA) · v0.1

> **Document type:** GDPR Art. 30 Records of Processing Activities  
> **Owner:** `compliance-officer` (DPO-equivalent). Co-reviewed: `enterprise-architect`, `security-engineer`, `data-engineer`.  
> **Triggers:** Required before SOC 2 observation period (Month 3); blocking item `docs/GDPR_DPIA.md §10.2.1`.  
> **Status:** Draft — legal review required before EU enterprise pilot launch.  
> **References:** `docs/GDPR_DPIA.md` (source processing inventory), `docs/SUBPROCESSORS.md` (sub-processor register), `docs/DATA_MODEL.md` (schema), `docs/AUDIT_LOG_SCHEMA.md` (DEC-030), `docs/SSO_SCIM_IMPLEMENTATION.md §15.9.4` (SCIM group processing), `docs/SOC2_READINESS.md §13` (DPA requirements), DEC-030.  
> **Confidentiality:** Internal compliance document. Disclosed to enterprise customers under NDA on request. Not published publicly. Disclosed to supervisory authority on request under Art. 30(4).  
> **Review cadence:** Annual, or on any material change to processing scope, whichever is earlier.

---

## Table of Contents

1. [Document Scope and Legal Basis](#1-document-scope-and-legal-basis)
2. [Controller Identity](#2-controller-identity)
3. [Art. 30(1) Register — FORM as Data Controller](#3-art-301-register--form-as-data-controller)
4. [Art. 30(2) Register — FORM as Data Processor](#4-art-302-register--form-as-data-processor)
5. [Sub-Processor Reference Table](#5-sub-processor-reference-table)
6. [International Transfer Summary](#6-international-transfer-summary)
7. [Retention Schedule](#7-retention-schedule)
8. [Technical and Organisational Measures (Summary)](#8-technical-and-organisational-measures-summary)
9. [DEC-030 Audit Events](#9-dec-030-audit-events)
10. [Open Questions and Gaps](#10-open-questions-and-gaps)
11. [Implementation Checklist](#11-implementation-checklist)
12. [Review Log](#12-review-log)

---

## 1. Document Scope and Legal Basis

### 1.1 Regulatory Obligation

GDPR Art. 30 requires both controllers and processors to maintain written records of processing activities. The obligation applies to organisations with ≥ 250 employees, or — regardless of size — where:

- processing is not occasional;
- processing involves special categories of data under Art. 9; or
- processing involves personal data relating to criminal convictions and offences.

FORM satisfies the second and third criteria: workout performance, biometric inference, HRV data, and meal logs are Art. 9 special category data. Processing is continuous (not occasional). The obligation therefore applies at launch, not at headcount 250.

### 1.2 Dual-Role Structure

FORM operates in two GDPR roles simultaneously:

| Role | Context | Register |
|---|---|---|
| **Data Controller** | FORM determines purposes and means for consumer-tier and enterprise aggregate analytics | Art. 30(1) register — §3 of this document |
| **Data Processor** | Enterprise customers (employers) are controllers; FORM processes employee data on their behalf | Art. 30(2) register — §4 of this document |

Both registers are maintained in this document. `docs/SUBPROCESSORS.md §1.1–1.2` provides the full legal analysis of the dual-role distinction.

### 1.3 UK GDPR Applicability

This ROPA serves simultaneously as the Art. 30 record under UK GDPR. All Art. 30 references in this document encompass UK GDPR Art. 30 where applicable to UK data subjects. Transfer mechanisms for UK data subjects use the UK IDTA or UK Addendum to EU SCCs.

### 1.4 Relationship to Other Documents

| Document | Relationship |
|---|---|
| `docs/GDPR_DPIA.md` | Source inventory for processing activities PA-01–PA-12; this ROPA formalises that inventory as an Art. 30-compliant register |
| `docs/SUBPROCESSORS.md` | Sub-processor register (Art. 28 DPAs); §5 contains the enterprise DPA template |
| `docs/DATA_MODEL.md` | Schema definitions for each data category and tenant isolation controls |
| `docs/SOC2_READINESS.md §35` | Privacy series (P1–P8) control mapping; P-series evidence artefacts |
| `docs/SSO_SCIM_IMPLEMENTATION.md §15.9.4` | SCIM group membership processing — incorporated as RA-13 in §3 below |

---

## 2. Controller Identity

### 2.1 Data Controller (Art. 30(1)(a))

| Field | Value |
|---|---|
| **Controller name** | FORM, Inc. (also trading as FORM Coach) |
| **Legal form** | Delaware corporation (pre-registration; interim: Ukrainian FOP — update when Delaware entity registered) |
| **Registered address** | `[FILL: registered address post-incorporation]` |
| **Contact** | `[FILL: DPO / compliance contact email — compliance@form.coach]` |
| **DPO / privacy contact** | `compliance-officer` agent (internal); no mandatory DPO appointment under Art. 37 at current scale; revisit at 10k+ enterprise seats |
| **Joint controller?** | No joint controllers at present. Enterprise tier introduces a controller-processor relationship (§4), not joint controllership. |

### 2.2 Representative in the EU/EEA (Art. 27)

FORM processes data of EU/EEA data subjects. As a non-EU establishment, FORM must designate an EU representative under Art. 27 before marketing to EU users.

| Status | Details |
|---|---|
| **Current status** | 🔴 Not yet designated — ROPA-GAP-001 (P0, pre-EU-launch) |
| **Planned mechanism** | Designate a representative in an EU member state (Ireland preferred, consistent with Supabase `eu-west-1` hosting) |
| **Deadline** | Before any EU marketing campaigns or EU enterprise deal close |

---

## 3. Art. 30(1) Register — FORM as Data Controller

Each row below constitutes a processing activity record per Art. 30(1)(b)–(h). Source: `docs/GDPR_DPIA.md §2.1` processing inventory.

### 3.1 Processing Activity Register

| ID | Activity Name | Purpose | Legal Basis | Data Subjects | Data Categories | Art. 9 Special Category? | Sub-Processors | Retention | Transfer Mechanism | Notes |
|---|---|---|---|---|---|---|---|---|---|---|
| **RA-01** | Account creation and identity | Create and authenticate user accounts | Art. 6(1)(b) — contract performance | All users | Email address, display name, hashed password (Supabase Auth), provisioning method (SSO/email), `user_id` UUID | ❌ No | SP-01 Supabase | Active account lifetime + 30-day deactivation window; hard-deleted on Art. 17 erasure request | EU: adequacy (Supabase `de-fra1`); US: SCCs Module 2 | Supabase Auth handles credential storage; FORM never stores plaintext passwords |
| **RA-02** | Health profile collection | Personalise training programme and Victor AI coaching | Art. 9(2)(a) — explicit consent | All users (voluntary fields) | Date of birth, biological sex, height, resting HR, HRV baseline, goals (free text), injury/restriction flags | ✅ Yes — HRV, injury flags, body composition goals | SP-01 Supabase | Active lifetime; soft-deleted on deactivation; hard-deleted on Art. 17 erasure or at 36 months inactivity (§7.2) | EU: adequacy; US: SCCs Module 2 | `consent_version` stamped at collection; separate consent from RA-01 (granular consent model) |
| **RA-03** | CV-based workout analysis (pose estimation) | Provide real-time form correction and rep counting | Art. 6(1)(b) — contract; Art. 9(2)(a) — consent for health inference | All users | **On-device only:** joint angles, rep count, form score (0–100). Only `form_score` + exercise metadata stored in Supabase. **Raw camera frames: never uploaded.** | ✅ Yes — physical capacity and health trajectory inferred from form score | SP-01 Supabase (metadata only; no video) | `cv_sessions` rows: 12 months from session; form scores: retained in `workout_sessions` per RA-05 retention | EU: adequacy; US: SCCs Module 2 | On-device inference via Core ML is the primary privacy control. No video, keypoints, or raw biometric data leave the device. |
| **RA-04** | Meal logging | Track nutrition for programme optimisation | Art. 9(2)(a) — explicit consent (food log is health-adjacent) | Pro and Enterprise users who opt in | Food items (free text / barcode scan), macros (kcal, protein, carbs, fat), meal type, timestamp | ✅ Yes — dietary health patterns; ED-adjacent | SP-01 Supabase | Meal log entries: 24 months from creation; hard-deleted on Art. 17 erasure | EU: adequacy; US: SCCs Module 2 | Separate consent gate from health profile (granular five-category consent model). `clinical-safety` VETO applies to any change to meal log features. |
| **RA-05** | Victor AI coaching | Deliver personalised coaching responses | Art. 6(1)(b) — contract performance | All users | Pseudonymised exercise context: sets, reps, RPE, programme phase, coaching turn ID. **No PII in Anthropic prompts.** **No raw health values.** | ⚠️ Indirectly — context infers health status, but no Art. 9 data transmitted to Anthropic | SP-07 Anthropic | `coaching_turns` rows: 24 months; Anthropic data-in-transit only, zero retention at Anthropic (contractual) | US: SCCs Module 2; Anthropic DPA prohibits training use | `stripPersonalProperties()` applied before every API call. `docs/OBSERVABILITY.md §25.7` constraint list enforced via CI lint. |
| **RA-06** | Voice synthesis (Victor voice) | Deliver spoken coaching cues | Art. 6(1)(b) — contract performance (voice is a delivery channel, not separate processing) | Users who enable voice coaching | Coaching text cues only; no user PII in payload | ❌ No (coaching text carries no health data) | SP-08 ElevenLabs | In-transit only; no retention at ElevenLabs (contractual SCC) | US: SCCs Module 2 | DPA with ElevenLabs pending (OQ-SP-01, `docs/SUBPROCESSORS.md §12`). Processing proceeds under SCC-only until DPA executed; compensating control: no PII in payload. |
| **RA-07** | Wearable integration (HealthKit / Health Connect / Whoop / Oura / Garmin) | Enrich coaching with physiological signals (HRV, sleep, resting HR) | Art. 9(2)(a) — explicit consent (separate opt-in per wearable source) | Users who opt in | HRV, resting HR, sleep stages, step count — processed on-device via HealthKit / Health Connect APIs. Aggregated summary metrics (not raw) stored in Supabase. | ✅ Yes — HRV, sleep stages are health data | SP-01 Supabase (aggregated metadata); HealthKit / Health Connect process on-device (not GDPR sub-processors — Apple/Google are platform APIs, not FORM sub-processors) | Wearable readings: 24 months; raw platform data is never held by FORM | EU: adequacy; US: SCCs Module 2 (Supabase only) | Stale data safety gate: Victor coaching confidence is downgraded if wearable data is > 48h old (`docs/OBSERVABILITY.md §41.7`; clinical-safety VETO on weakening this gate). |
| **RA-08** | ED screening (SCOFF-light) | Identify users who may be harmed by standard nutrition coaching and apply clinical-safety guardrails | Art. 9(2)(a) — explicit consent + Art. 6(1)(f) — legitimate interest (duty of care) | All users at onboarding | SCOFF-light responses (3 binary questions). Result stored as a UX flag only (not a diagnostic label). Response content not stored after flag is set. | ✅ Yes — mental health adjacent | SP-01 Supabase (UX flag only; responses discarded) | UX flag: active account lifetime; hard-deleted on Art. 17 erasure | EU: adequacy; US: SCCs Module 2 | DEC-018 governs ED-screening. `clinical-safety` has absolute VETO on any change to this activity. Flag stored as `BOOLEAN ed_flag`, not as questionnaire response — minimisation by design. |
| **RA-09** | Enterprise aggregate reporting | Provide HR/People leads with anonymised wellness programme metrics | Art. 6(1)(f) — legitimate interests of enterprise customer (evidence-based wellness programme management) | Enterprise users (employees of enterprise customer tenants) | Aggregated and k-anonymised tenant metrics: active user count, workout count, average form score, average session duration. **No individual health data.** k-anonymity floor: N < 5 per cohort → suppressed. | ❌ No (aggregated and anonymised; no individual data) | SP-01 Supabase (materialized view; `form_admin` role; BYPASSRLS) | Aggregated snapshots: 36 months | EU: adequacy (tenant data in EU region); US tenants: SCCs Module 2 | Privacy Floor: enterprise admin roles (`tenant_owner`, `tenant_admin`, `tenant_manager`) have **zero access** to individual user health data. `data.read_individual` event triggers PagerDuty P1 alert. See `docs/DATA_MODEL.md §3` and `docs/SOC2_READINESS.md §35.7`. |
| **RA-10** | Audit logging (DEC-030) | Maintain tamper-evident record of all privileged actions for SOC 2 evidence, GDPR Art. 30, and security incident investigation | Art. 6(1)(c) — legal obligation (SOC 2, Art. 30) + Art. 6(1)(f) — legitimate interest (security) | All users; FORM employees; enterprise tenant admins | Actor ID (user UUID or `[DELETED-{hash}]` post-erasure), action, resource type, IP address, outcome, HMAC chain fields. No health data content in log payloads — only action taxonomy. | ❌ No (action taxonomy only; no health field values logged) | SP-01 Supabase (primary HMAC chain); Cloudflare R2 (compliance vault archive, encrypted) | 7 years (DEC-030 immutable HMAC chain; WORM bucket; consistent with Art. 30 records retention and US litigation hold best practice) | EU: adequacy; US: SCCs Module 2 | Audit log is append-only; no deletion even on Art. 17 erasure — actor_id anonymised to `[DELETED-{hash}]` instead. This is the Art. 30 compliance record itself. |
| **RA-11** | Product analytics (PostHog) | Understand feature usage, improve product, measure activation and retention | Art. 6(1)(f) — legitimate interests (product improvement); `analytics_opt_out` honoured | All users | Anonymous event stream: event name, timestamp, anonymous `distinct_id` (SHA-256 of user UUID), device type, app version. **No health event content.** **No health field values.** | ❌ No (health events are blocklisted at SDK level — `docs/OBSERVABILITY.md §25.7`) | SP-09 PostHog (EU Cloud, Frankfurt) | Anonymous events: 24 months; PostHog auto-expires | EU: Frankfurt (adequacy — no international transfer for EU users) | CI lint rule enforces no health field content in PostHog events (`BLOCKED_HEALTH_EVENTS` array). `analytics_opt_out: true` → zero events sent. |
| **RA-12** | Error monitoring (Sentry) | Detect and diagnose application errors and crashes | Art. 6(1)(f) — legitimate interests (service reliability) | All users | Anonymised crash reports, stack traces, breadcrumbs. PII scrubbed at SDK level via `beforeSend` filter (`docs/OBSERVABILITY.md §28.3` deny-list). Health field values replaced with `[HEALTH_DATA_REDACTED]`. | ❌ No (PII and health data scrubbed at source) | SP-06 Sentry | Error events: 90 days (Sentry default; no extended retention configured) | US: SCCs Module 2 (pending DPA execution — VRM-GAP-001) | DPA with Sentry pending. Compensating control: `beforeSend` filter verified in CI. `health_field_detected` Sentry tag surfaces scrubber activations for audit. |
| **RA-13** | SCIM group membership (enterprise SSO tenants) | Access control and role assignment within FORM enterprise tier (maps IdP groups to FORM roles) | Art. 6(1)(f) — legitimate interests of employer controller (maintaining role-based access to a workplace tool) | Enterprise users on SCIM-provisioned tenants | Group membership relationships (user–group pairs); SCIM group `displayName`. No health data. | ❌ No | SP-01 Supabase; SP-11 WorkOS (SCIM provisioning source) | Active while SCIM provisioning active; hard-deleted on user erasure (Art. 17) immediately — no soft-delete period for access-control data (`docs/SSO_SCIM_IMPLEMENTATION.md §15.9.1`). Group records retained until tenant deletes group or tenant offboarding. | EU: adequacy; US: SCCs Module 2 (WorkOS) | Art. 9 group-name scanner blocks encoding of health categories in SCIM `displayName` (`docs/SSO_SCIM_IMPLEMENTATION.md §15.9.5`). Incorporated from `docs/SSO_SCIM_IMPLEMENTATION.md §15.9.4`. |

### 3.2 Consent Dependency Map

The following activities require explicit consent under Art. 9(2)(a). Withdrawal of consent triggers the downstream cascade:

```
consent.health_profile (RA-02)
  └── Withdrawal → stop RA-02 processing → offer Art. 17 erasure of health profile data

consent.meal_log (RA-04)
  └── Withdrawal → stop RA-04 processing → offer erasure of meal log

consent.wearable (RA-07)
  └── Withdrawal → stop RA-07 processing → delete wearable readings

consent.cv_coaching (RA-03)
  └── Withdrawal → stop RA-03 form-score storage; on-device inference continues until user revokes camera permission

consent.analytics (RA-11)
  └── analytics_opt_out = TRUE → zero PostHog events; can re-opt-in at any time

RA-08 (ED screening)
  └── Consent collected once at onboarding; cannot be withdrawn in isolation — tied to account creation
  └── Withdrawal = account deletion (Art. 17 erasure)

RA-05 (Victor AI coaching) — Art. 6(1)(b) contract basis
  └── No separate consent; pseudonymised processing; withdrawal = contract termination (deletion of account)

RA-01, RA-06, RA-10 — contract / legal obligation / legitimate interest bases
  └── Not consent-dependent; right to object (Art. 21) available for RA-06 legitimate interest basis
```

Consent events are HMAC-chained DEC-030 entries (`privacy.consent_granted`, `privacy.consent_withdrawn`). Consent version stamped at collection; `consent_version` column in `users` table tracks current version accepted.

---

## 4. Art. 30(2) Register — FORM as Data Processor

When enterprise customers (employers) deploy FORM to their employees, the employer is the **data controller** and FORM acts as a **data processor** under GDPR Art. 28. This register covers FORM's processing activities in that processor role.

### 4.1 Processor Register

| ID | Processing Activity | Controller (Customer) | Instruction Basis | Data Categories Processed on Behalf of Controller | Sub-Processors Used | Security Measures | Relevant DPA Clause |
|---|---|---|---|---|---|---|---|
| **RP-01** | Employee user account creation and SSO provisioning | Enterprise customer (employer) | Enterprise DPA Exhibit C (`docs/SUBPROCESSORS.md §5`); Order Form specifying provisioning method | Employee email, display name, SSO identity (`sub` claim from IdP), group memberships (SCIM) | SP-01 Supabase, SP-11 WorkOS | RLS tenant isolation (`tenant_id`); AES-256 at rest; TLS 1.3 in transit; HMAC audit log | Art. 28(3)(a) — processing only on documented instructions |
| **RP-02** | Employee health data processing (all RA-02–RA-08 activities, scoped to employer's tenant) | Enterprise customer (employer) | Enterprise DPA §3 — processor processing health data on customer's behalf for programme delivery | All Art. 9 categories from §3.1, scoped to `tenant_id` via RLS | SP-01 Supabase, SP-07 Anthropic (pseudonymised only), SP-08 ElevenLabs (coaching text only) | On-device CV inference; no Art. 9 data in Anthropic prompts; k-anonymity floor for aggregate reporting | Art. 28(3)(b) — confidentiality obligations on all authorised persons |
| **RP-03** | Aggregate wellness reporting (RA-09) | Enterprise customer (employer) | Enterprise DPA §4 — processor generating aggregate reports for controller's HR dashboard | k-anonymised aggregates only; no individual health data exposed | SP-01 Supabase | k-anonymity N < 5 → suppressed; materialized view access scoped to `form_admin` role; individual row access blocked at RLS layer | Art. 28(3)(c) — appropriate technical and organisational security measures |
| **RP-04** | Audit log and compliance evidence | Enterprise customer (employer) + FORM (independent legal obligation) | Enterprise DPA §6 — audit log as dual-purpose record (SOC 2 evidence + customer's Art. 30 processor records) | Action taxonomy (no health field values); actor_id (user UUID, anonymised post-erasure) | SP-01 Supabase, Cloudflare R2 (WORM archive) | HMAC chain (DEC-030); append-only; 7-year retention; tenant admin read-only view via admin dashboard | Art. 28(3)(h) — information to demonstrate compliance with Art. 28; audit rights |
| **RP-05** | Data Subject Access Request (DSAR) fulfilment for enterprise employees | Enterprise customer (employer) | Enterprise DPA §8 — processor assists controller in fulfilling DSAR obligations (Art. 28(3)(e)) | All personal data held in the customer's tenant scope (`tenant_id` filter) | SP-01 Supabase | DSAR export encrypted (Resend encrypted delivery); `dsar.data_provided` HMAC-chained event; 24-hour SLO (`docs/ENTERPRISE_SLA.md §19.5`) | Art. 28(3)(e) — processor assists controller with DSAR obligations |
| **RP-06** | Employee data deletion and offboarding | Enterprise customer (employer) | Enterprise DPA §9 — processor deletes all employee data on controller instruction or contract end | All personal data in customer's tenant scope; Art. 9 health data subject to zero-grace-period hard-delete | SP-01 Supabase, Cloudflare R2 (deletion propagation) | Art. 9 zero grace period (DEC-036); `data.individual_deletion` HMAC-chained event; deletion certificate issued; 90-day standard data deletion post-offboarding (§7.3) | Art. 28(3)(g) — deletion or return of data on controller instruction |

### 4.2 Processing Instruction Scope

FORM processes enterprise employee personal data **only** on the documented instructions of the enterprise customer (controller). Instructions are established via:

1. **Enterprise DPA** (`docs/SUBPROCESSORS.md §5`) — primary processor instruction document
2. **Order Form** — specifies seat count, enabled features, and tenant configuration
3. **Admin Dashboard settings** — runtime configuration by `tenant_owner` / `tenant_admin` roles

**Out-of-scope instructions:** FORM will not process employee data for purposes outside the service delivery scope (analytics training, cross-tenant comparison, employer performance scoring, insurance risk assessment). These are explicitly prohibited in the DPA §3.4 and `docs/ENTERPRISE.md §10` no-go list.

### 4.3 Sub-Processor Notification Obligation

FORM must provide enterprise customers 30 days advance notice before onboarding new sub-processors that process their employees' data (GDPR Art. 28(2); enterprise DPA §8.2). The procedure:

1. `compliance-officer` drafts notification to enterprise customers via `customer-success` channel
2. `vendor.sub_processor_added` DEC-030 HMAC-chained event emitted (with 30-day notice start timestamp)
3. Customer has 30 days to object; objection triggers review; if not resolved, customer may terminate without penalty
4. Sub-processor register at `docs/SUBPROCESSORS.md §2` updated on effective date

---

## 5. Sub-Processor Reference Table

Full register: `docs/SUBPROCESSORS.md §2`. Summary below for ROPA cross-reference.

| ID | Sub-Processor | Role | Art. 9 Exposure | DPA Status | Applicable Activities |
|---|---|---|---|---|---|
| SP-01 | Supabase, Inc. | Primary data store, Auth, File Storage | ✅ Yes (all Art. 9 data at rest) | 🔴 Pending (VRM-GAP-001) | RA-01–RA-13, RP-01–RP-06 |
| SP-02 | Cloudflare, Inc. | CDN, Edge Workers, R2 (WORM archive), WAF | ❌ No (encrypted transit / metadata only) | 🟡 ToS DPA active | RA-10, RP-04 (WORM archive) |
| SP-03 | Resend, Inc. | Transactional email (DSAR delivery) | ❌ No (email address only; no health content) | 🔴 Pending (VRM-GAP-001) | RP-05 (DSAR) |
| SP-04 | RevenueCat, Inc. | Subscription lifecycle | ❌ No | 🔴 Pending (VRM-GAP-001) | Consumer billing only |
| SP-05 | Apple, Inc. (APNs) | Push notifications | ❌ No (device token only; no health payload) | 🟡 Compensating control | RA-01 (notifications) |
| SP-06 | Sentry, Inc. | Error monitoring | ❌ No (PII scrubbed at SDK) | 🔴 Pending (VRM-GAP-001) | RA-12 |
| SP-07 | Anthropic, PBC | LLM inference (Victor) | ⚠️ Indirect (pseudonymised context only) | 🟢 Enterprise DPA active | RA-05, RP-02 |
| SP-08 | ElevenLabs, Inc. | Voice synthesis | ❌ No (coaching text only; no PII) | 🔴 Pending (OQ-SP-01) | RA-06, RP-02 |
| SP-09 | PostHog, Inc. (EU) | Product analytics | ❌ No (anonymous events; health blocklist) | 🔴 Pending | RA-11 |
| SP-10 | Stripe, Inc. | Billing events (webhooks) | ❌ No | 🟢 DPA active | Consumer/enterprise billing |
| SP-11 | WorkOS, Inc. | Enterprise SSO + SCIM | ❌ No (identity metadata only; no health data) | 🟡 Pending verification | RA-13, RP-01 |

---

## 6. International Transfer Summary

FORM is headquartered in Ukraine (Kyiv) with data infrastructure primarily in the EU and US. Data subjects are primarily in the EU/EEA, UK, and US.

| Transfer Route | Mechanism | Applies To | Status |
|---|---|---|---|
| **FORM → Supabase EU (Frankfurt)** | EU/EEA adequacy — no transfer | EU/EEA data subjects | ✅ In place |
| **FORM → Supabase US (`us-east-1`)** | SCCs Module 2 (controller → processor) | US data subjects | ✅ Pending DPA execution (VRM-GAP-001) |
| **FORM → Cloudflare (global)** | SCCs Module 2 + Cloudflare GDPR DPA | All data subjects | 🟡 ToS DPA active |
| **FORM → Anthropic (US)** | SCCs Module 2; enterprise DPA prohibits training use | All users (coaching context only) | ✅ In place |
| **FORM → PostHog EU (Frankfurt)** | EU/EEA adequacy — no transfer | EU/EEA data subjects | 🔴 Pending DPA execution |
| **FORM → Resend (US)** | SCCs Module 2 | All data subjects | 🔴 Pending DPA execution |
| **FORM → Sentry (US)** | SCCs Module 2 | All data subjects | 🔴 Pending DPA execution |
| **FORM → ElevenLabs (US)** | SCCs Module 2 | All data subjects | 🔴 Pending DPA execution |
| **FORM → WorkOS (US)** | SCCs Module 2 | Enterprise SSO tenants only | 🟡 Pending verification |
| **FORM → Stripe (US + EU)** | SCCs Module 2 + Stripe DPA | Billing events only | ✅ In place |

### 6.1 UK GDPR Transfers

For data subjects in the United Kingdom:
- US sub-processors: UK IDTA (International Data Transfer Agreement) or UK Addendum to EU SCCs required as separate transfer mechanism
- EU-based processing (Supabase Frankfurt, PostHog Frankfurt): UK adequacy decision covers EU/EEA transfers; no separate mechanism required for UK → EU transfers
- **Status:** 🔴 UK transfer mechanisms not yet verified for all US sub-processors — ROPA-GAP-002 (P1)

### 6.2 Transfer Impact Assessment (TIA)

A Transfer Impact Assessment for US transfers is required before first EU enterprise deal close. The TIA evaluates:
- US FISA §702 / EO 14086 risk for each US sub-processor
- Supplementary measures in place (encryption, minimisation, pseudonymisation)
- Whether the SCCs remain effective given the legal framework of the destination country

**Status:** 🔴 TIA not yet completed — ROPA-GAP-003 (P1, before EU enterprise sales)

---

## 7. Retention Schedule

Full retention decisions and legal basis analysis: `compliance/p1/retention-decisions.md` (P1-RET-001).

| Data Category | Table / Store | Retention Period | Legal Basis for Retention | Deletion Method | Implemented? |
|---|---|---|---|---|---|
| User account (email, display name) | `users` | Active lifetime + 30 days post-deactivation | Contract (Art. 6(1)(b)); Art. 17 erasure right applies | Hard-delete; email pseudonymised to `[DELETED-{hash}]` in audit log | 🔴 Migration pending |
| Health profile (HRV, goals, injury flags) | `user_health_profiles` | 36 months from last activity OR erasure request | Explicit consent (Art. 9(2)(a)); Art. 17 zero-grace-period for Art. 9 data | Hard-delete; zero grace period on Art. 17 request | 🔴 Supabase TTL migration pending (P-GAP-004) |
| Workout sessions (form score, rep count) | `workout_sessions` | 36 months from session date | Contract (Art. 6(1)(b)); legitimate interest (training history) | Hard-delete at 36m via pg_cron + WORM purge | 🔴 TTL migration pending |
| CV session metadata (form_score, joint metadata) | `cv_sessions` | 12 months from session | Contract (Art. 6(1)(b)) — shorter period justified by performance analytics only | Hard-delete at 12m | 🔴 TTL migration pending |
| Coaching turns (Victor conversation) | `coaching_turns` | 24 months from turn | Contract (Art. 6(1)(b)); coaching context window requirement | Hard-delete at 24m | 🔴 TTL migration pending |
| Meal log entries | `meal_log` | 24 months from entry | Explicit consent (Art. 9(2)(a)) | Hard-delete at 24m; hard-delete on consent withdrawal | 🔴 TTL migration pending |
| Wearable readings (aggregated) | `wearable_readings` | 24 months from reading | Explicit consent (Art. 9(2)(a)) | Hard-delete at 24m | 🔴 TTL migration pending |
| ED screening flag | `users.ed_flag` | Active account lifetime | Consent + legitimate interest (duty of care) | Hard-delete on Art. 17 erasure | 🔴 Migration pending |
| SCIM group memberships | `scim_group_members` | Active while provisioned; immediate on Art. 17 erasure | Art. 6(1)(f) — employer legitimate interest | Hard-delete; zero soft-delete period for access-control data | 🔴 Migration pending |
| Audit log (DEC-030) | `audit_log` + R2 WORM | 7 years (immutable HMAC chain) | Art. 6(1)(c) — legal obligation (SOC 2; Art. 30 records); Art. 6(1)(f) (security) | Actor_id anonymised on Art. 17; log entries NOT deleted (WORM) | 🟡 Partial (HMAC chain implemented; WORM bucket pending) |
| Product analytics (PostHog) | PostHog EU Cloud | 24 months (PostHog default) | Art. 6(1)(f) — legitimate interest | Auto-expiry at PostHog; no cross-reference to user_id | 🟡 PostHog DPA pending |
| Error monitoring (Sentry) | Sentry | 90 days | Art. 6(1)(f) — legitimate interest | Auto-expiry at Sentry | 🟡 Sentry DPA pending |

### 7.1 Retention Implementation Status

All health data retention periods are formally decided in `compliance/p1/retention-decisions.md` (P1-RET-001, effective 2026-05-21). Implementation of Supabase TTL migrations and pg_cron deletion jobs is **P0 before SOC 2 observation period**. See P-GAP-004 in `docs/SOC2_READINESS.md §35`.

### 7.2 Art. 17 Erasure Cascade

On receipt of a valid Art. 17 erasure request, the following cascade executes:

```
1. auth.users row deleted (Supabase Auth)
2. user_health_profiles hard-deleted (Art. 9 — zero grace period)
3. cv_sessions hard-deleted
4. workout_sessions hard-deleted
5. coaching_turns hard-deleted
6. meal_log hard-deleted
7. wearable_readings hard-deleted
8. scim_group_members hard-deleted (immediate — access-control data)
9. users.ed_flag set to NULL
10. audit_log: actor_id pseudonymised to [DELETED-{hash}]; entries NOT deleted
11. DEC-030 data.individual_deletion event emitted (group_memberships_deleted: true, group_count: N)
12. Deletion certificate issued to data subject via encrypted Resend delivery
```

DEC-030 events chaining this cascade: `dsar.deletion_soft`, `dsar.deletion_confirmed`, `data.individual_deletion` (`docs/AUDIT_LOG_SCHEMA.md`; `docs/ENTERPRISE_SLA.md §19.5`).

### 7.3 Enterprise Tenant Offboarding Data Deletion

On contract end, FORM deletes all enterprise customer data per enterprise DPA §9:
- Trigger: `tenant.lifecycle_status = 'offboarded'` (DEC-030 `tenant.lifecycle_status_changed`)
- Timeline: All user personal data deleted within 90 days of offboarding
- Art. 9 data: Hard-deleted within 30 days (zero grace period extends to contract-end offboarding)
- Aggregate snapshots (RA-09): Deleted within 90 days
- Audit log: Retained 7 years per legal obligation; actor_ids anonymised
- Deletion certificate: Issued to `tenant_owner` contact; `dsar.offboarding_export_available` event emitted prior to deletion

---

## 8. Technical and Organisational Measures (Summary)

Full TOMs: `docs/MSA_TEMPLATE.md Exhibit D`; `docs/SECURITY.md`; `docs/SOC2_READINESS.md §3`.

| Control Category | Measure | Reference |
|---|---|---|
| **Encryption at rest** | AES-256 (Supabase / AWS KMS) for all Supabase tables; per-tenant KMS-derived key for sensitive health columns | `docs/CRYPTOGRAPHY_POLICY.md` |
| **Encryption in transit** | TLS 1.3 minimum enforced at Cloudflare edge; HSTS preloading; certificate managed by Cloudflare | `docs/SECURITY.md §3` |
| **Tenant isolation** | Row-Level Security (RLS) on all Supabase tables; `tenant_id` partitioning; cross-tenant query blocked at Postgres policy layer | `docs/DATA_MODEL.md §3` |
| **Access control** | RBAC: `tenant_owner`, `tenant_admin`, `tenant_manager`; `form_audit` (compliance read-only); `form_admin` (aggregate view only). MFA enforced for all admin roles. | `docs/DATA_MODEL.md §6` |
| **On-device inference** | CV pose estimation runs on-device (Core ML); no raw video or keypoints transmitted to FORM servers | `docs/GDPR_DPIA.md §7` |
| **Data minimisation** | PII stripped from Anthropic prompts via `stripPersonalProperties()`; PostHog health-event blocklist enforced by CI lint; Sentry `beforeSend` scrubber | `docs/OBSERVABILITY.md §25.7, §28.3` |
| **Audit logging** | Append-only HMAC-chained audit log (DEC-030); every privileged action logged; tamper-evident; 7-year retention | `docs/AUDIT_LOG_SCHEMA.md` |
| **Vulnerability management** | Annual third-party penetration test; quarterly dependency scan (Dependabot); vulnerability findings tracked in Linear with SLA | `docs/PENETRATION_TESTING.md`; `docs/VULNERABILITY_MANAGEMENT.md` |
| **Incident response** | P0–P3 classification; P0 ≤ 4h resolution SLA; 72h GDPR breach notification; post-mortem within 48h | `docs/INCIDENT_RESPONSE.md` |
| **Key rotation** | AES-256 keys rotated quarterly; HMAC audit chain secret rotated quarterly; JWT secrets rotated on personnel change | `docs/KEY_ROTATION.md` |
| **Personnel security** | Background checks at hire; NDA; annual security awareness training; principle of least privilege | `docs/ONBOARDING_SECURITY.md`; `docs/SECURITY_AWARENESS_TRAINING_POLICY.md` |
| **Business continuity** | RPO 4 hours; RTO 12 hours; Supabase PITR 7-day window; Cloudflare R2 WORM for compliance vault | `docs/BUSINESS_CONTINUITY.md` |

---

## 9. DEC-030 Audit Events

The following DEC-030 HMAC-chained events are emitted in connection with ROPA-governed processing activities. Full schema: `docs/AUDIT_LOG_SCHEMA.md`.

| Event | Severity | Retention | Processing Activity | Purpose |
|---|---|---|---|---|
| `privacy.consent_granted` | HIGH | 7 years | RA-02, RA-04, RA-07, RA-08 | Records consent collection with version stamp; Art. 5(2) accountability |
| `privacy.consent_withdrawn` | HIGH | 7 years | RA-02, RA-04, RA-07 | Records consent withdrawal; triggers processing cessation cascade |
| `data.export_initiated` | HIGH | 7 years | RP-05, RA-10 | DSAR Art. 15 export start; anchors 30-day GDPR SLA |
| `dsar.data_provided` | HIGH | 7 years | RP-05 | DSAR fulfilment event; 24-hour SLO evidence |
| `dsar.deletion_soft` | HIGH | 7 years | RP-06, §7.2 | Art. 17 soft delete initiated |
| `dsar.deletion_confirmed` | HIGH | 7 years | RP-06, §7.2 | Art. 17 hard delete completed; deletion certificate issued |
| `dsar.portability_export_completed` | STANDARD | 7 years | RP-05 | Art. 20 data portability export |
| `dsar.offboarding_export_available` | STANDARD | 7 years | RP-06, §7.3 | Contract-end export available before deletion |
| `data.individual_deletion` | CRITICAL | 7 years | §7.2 | Complete Art. 17 cascade completion; includes `group_memberships_deleted` |
| `data.art9_hard_delete` | CRITICAL | 7 years | §7.2, §7.3 | Art. 9 data zero-grace-period hard delete; separate event for audit trail separation |
| `tenant.lifecycle_status_changed` | HIGH | 7 years | RP-06, §7.3 | Tenant offboarding trigger event |
| `ropa.review_completed` | STANDARD | 7 years | §12 (review log) | Annual ROPA review completion — Art. 30 compliance evidence |
| `ropa.processing_activity_added` | STANDARD | 7 years | All | New processing activity formally registered in ROPA |
| `ropa.processing_activity_modified` | HIGH | 7 years | All | Material change to existing processing activity — triggers DPIA re-assessment if Art. 9 data affected |
| `vendor.sub_processor_added` | HIGH | 7 years | §4.3 | Sub-processor change notification; 30-day notice start timestamp |
| `vendor.sub_processor_removed` | HIGH | 7 years | §4.3 | Sub-processor removed from register |
| `vendor.dpa_executed` | HIGH | 7 years | §5 | DPA execution with sub-processor; VRM-GAP-001 closure evidence |

### 9.1 ROPA Events Registration

`ropa.review_completed`, `ropa.processing_activity_added`, and `ropa.processing_activity_modified` are **new DEC-030 event types** not yet registered in `docs/AUDIT_LOG_SCHEMA.md`. Registration is a P0 action in §11.

---

## 10. Open Questions and Gaps

### 10.1 Blocking Gaps (P0 — before EU enterprise launch or SOC 2 observation period)

| Gap ID | Description | Owner | Deadline | Closes |
|---|---|---|---|---|
| **ROPA-GAP-001** | EU representative not designated (Art. 27) — required before marketing to EU users | compliance-officer | Before EU marketing launch | `docs/GDPR_DPIA.md §10.1` (open) |
| **ROPA-GAP-002** | UK GDPR transfer mechanisms not verified for all US sub-processors (UK IDTA / UK Addendum) | compliance-officer + security-engineer | Before UK user launch | — |
| **ROPA-GAP-003** | Transfer Impact Assessment (TIA) for US sub-processors not completed | compliance-officer | Before first EU enterprise deal close | — |
| **ROPA-GAP-004** | Supabase TTL migrations for health data retention not implemented (workout_sessions, cv_sessions, coaching_turns, meal_log, wearable_readings) | platform-engineer | Pre-launch / SOC 2 observation start | P-GAP-004 in `docs/SOC2_READINESS.md §35` |
| **ROPA-GAP-005** | DEC-030 events `ropa.review_completed`, `ropa.processing_activity_added`, `ropa.processing_activity_modified` not registered in `docs/AUDIT_LOG_SCHEMA.md` | security-engineer | Pre-launch | — |
| **ROPA-GAP-006** | Sub-processor DPAs pending for SP-01 Supabase, SP-03 Resend, SP-04 RevenueCat, SP-06 Sentry (VRM-GAP-001) | compliance-officer | M6 | VRM-GAP-001 in `docs/SUBPROCESSORS.md §3` |

### 10.2 High-Priority Gaps (P1 — before first EU enterprise deal)

| Gap ID | Description | Owner | Deadline |
|---|---|---|---|
| **ROPA-GAP-007** | Privacy policy not published at `form.coach/privacy` — P0 enterprise gate and prerequisite for valid Art. 9 consent | compliance-officer | Pre-launch | 
| **ROPA-GAP-008** | ROPA itself not tested with a mock supervisory authority request — Art. 30(4) requires production within 2 weeks of request | compliance-officer | Before first EU deal |
| **ROPA-GAP-009** | PostHog DPA not yet executed (P1 M4 per `docs/OBSERVABILITY.md §25.8`) | compliance-officer | M4 |
| **ROPA-GAP-010** | ElevenLabs DPA not yet executed (OQ-SP-01, `docs/SUBPROCESSORS.md §12`) | compliance-officer | Before voice feature in production |
| **OQ-ROPA-01** | **k-anonymity floor at N < 5 for RA-09 aggregate reporting.** EDPB and some EU DPAs may expect k ≥ 10 for health-category data. Legal review required before EU enterprise sales. Tracked in `docs/GDPR_DPIA.md §10.3`. | compliance-officer + legal | Before first EU enterprise deal |
| **OQ-ROPA-02** | **Art. 27 EU representative — jurisdiction selection.** Ireland (consistent with Supabase `eu-west-1` hosting) vs. another EU member state. Influences which supervisory authority is the lead SA. | compliance-officer + legal | Before EU marketing launch |

### 10.3 Closed Items

| Item | Resolution |
|---|---|
| **GDPR_DPIA §10.2.1 — ROPA required** | ✅ Closed by this document (v0.1, 2026-06-13) |
| **SSO_SCIM §15.9.4 — SCIM group processing not in ROPA** | ✅ Closed — incorporated as RA-13 in §3.1 |

---

## 11. Implementation Checklist

### P0 — Before SOC 2 Observation Period / EU Enterprise Launch

| # | Action | Owner | Priority | Status |
|---|---|---|---|---|
| 1 | Register `ropa.review_completed`, `ropa.processing_activity_added`, `ropa.processing_activity_modified` events in `docs/AUDIT_LOG_SCHEMA.md` | security-engineer | P0 | ☐ |
| 2 | Implement Supabase TTL migrations for all health data retention periods in §7 (closes P-GAP-004) | platform-engineer | P0 | ☐ |
| 3 | Execute DPAs with SP-01 Supabase, SP-03 Resend, SP-04 RevenueCat, SP-06 Sentry (closes VRM-GAP-001) | compliance-officer | P0 | ☐ |
| 4 | Publish privacy policy at `form.coach/privacy` covering all RA-01–RA-13 activities (closes P-GAP-001) | compliance-officer | P0 | ☐ |
| 5 | Designate EU Art. 27 representative (closes ROPA-GAP-001) | compliance-officer + founder | P0 | ☐ |
| 6 | Complete erasure cascade implementation and end-to-end DSAR test (closes P-GAP-005) | platform-engineer + compliance-officer | P0 | ☐ |

### P1 — Before First EU Enterprise Deal Close

| # | Action | Owner | Priority | Status |
|---|---|---|---|---|
| 7 | Complete Transfer Impact Assessment for US sub-processors (closes ROPA-GAP-003) | compliance-officer + legal | P1 | ☐ |
| 8 | Verify UK IDTA / UK Addendum for all US sub-processors (closes ROPA-GAP-002) | compliance-officer | P1 | ☐ |
| 9 | Execute PostHog DPA (closes ROPA-GAP-009) | compliance-officer | P1 | ☐ |
| 10 | Execute ElevenLabs DPA (closes ROPA-GAP-010) | compliance-officer | P1 | ☐ |
| 11 | Commission legal review of k-anonymity floor N < 5 for EU health data aggregates (OQ-ROPA-01) | compliance-officer + legal | P1 | ☐ |
| 12 | Mock supervisory authority ROPA request exercise (closes ROPA-GAP-008) | compliance-officer | P1 | ☐ |

### P2 — Ongoing Maintenance

| # | Action | Owner | Status |
|---|---|---|---|
| 13 | Annual ROPA review — emit `ropa.review_completed` DEC-030 event; update review log (§12) | compliance-officer | ☐ Annual (June) |
| 14 | Update ROPA within 30 days of any material change to processing scope | compliance-officer | ☐ Triggered |
| 15 | Update RA-13 entry if SCIM group processing scope expands (OQ-ROPA-03: future group provisioning for non-access-control purposes) | enterprise-architect + compliance-officer | ☐ Triggered |
| 16 | File `compliance/ropa/ropa-v{N}-{date}.pdf` rendered copy on each revision | compliance-officer | ☐ On each version |

---

## 12. Review Log

| Version | Date | Author | Summary of Changes |
|---|---|---|---|
| v0.1 | 2026-06-13 | compliance-officer (enterprise-builder cloud session) | Initial ROPA. Art. 30(1) register (RA-01–RA-13) covering all processing activities from `docs/GDPR_DPIA.md §2.1` plus SCIM group processing from `docs/SSO_SCIM_IMPLEMENTATION.md §15.9.4`. Art. 30(2) processor register (RP-01–RP-06) covering enterprise customer data processing. Transfer mechanism summary, retention schedule (cross-reference to P1-RET-001), TOMs summary, DEC-030 events, 6 P0 / 6 P1 / 4 P2 implementation checklist items. 6 ROPA-specific gaps opened. Closes `docs/GDPR_DPIA.md §10.2.1` (ROPA required before observation period) and `docs/SSO_SCIM_IMPLEMENTATION.md §15.9.4` (SCIM group processing in Art. 30 records). Legal review pending; not yet disclosed to supervisory authority. |

**Next mandatory review:** Before first EU enterprise deal close, or by 2027-06-13, whichever is earlier.

---

*v0.1 · 2026-06-13*  
*Owner: compliance-officer + enterprise-architect + security-engineer*  
*Status: Draft — pending legal review before EU enterprise pilot*  
*Filed: `compliance/ropa/ropa-v0.1-2026-06-13.md` (to be rendered to PDF)*

*Closed items: `docs/GDPR_DPIA.md §10.2.1` (ROPA open item — Month 2 requirement); `docs/SSO_SCIM_IMPLEMENTATION.md §15.9.4` (SCIM Art. 30 records note).*  
*Cross-references: `docs/GDPR_DPIA.md` (processing inventory source), `docs/SUBPROCESSORS.md` (sub-processor register + enterprise DPA template §5), `docs/DATA_MODEL.md §3` (RLS tenant isolation), `docs/SOC2_READINESS.md §35` (P-series privacy controls), `docs/AUDIT_LOG_SCHEMA.md` (DEC-030 event schema), `docs/INCIDENT_RESPONSE.md §10` (breach notification), `docs/ENTERPRISE_SLA.md §19.5` (DSAR SLOs), `docs/MSA_TEMPLATE.md Exhibit C` (enterprise DPA), `docs/KEY_ROTATION.md`, `docs/CRYPTOGRAPHY_POLICY.md`, `docs/BUSINESS_CONTINUITY.md`.*
