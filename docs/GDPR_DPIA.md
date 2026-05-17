# FORM · GDPR Data Protection Impact Assessment v0.1

> **Document type:** Data Protection Impact Assessment (DPIA) per GDPR Art. 35  
> **Owner:** `compliance-officer` (DPO-equivalent) + `security-engineer`. Co-reviewed: `enterprise-architect`, `clinical-safety`.  
> **Status:** Draft — requires legal review before EU enterprise pilot launch.  
> **References:** DEC-030 (HMAC audit log), DEC-018 (ED-screening), `docs/AUDIT_LOG_SCHEMA.md`, `docs/ENTERPRISE.md`, `docs/DATA_MODEL.md`, `docs/SECURITY.md`, `docs/SOC2_READINESS.md`.  
> **Review cadence:** Annual or on material processing change, whichever comes first.

---

## Table of Contents

1. [Why This DPIA Is Mandatory](#1-why-this-dpia-is-mandatory)
2. [Processing Activities Inventory](#2-processing-activities-inventory)
3. [Data Flows](#3-data-flows)
4. [Legal Bases for Processing](#4-legal-bases-for-processing)
5. [Necessity and Proportionality](#5-necessity-and-proportionality)
6. [Risk Assessment](#6-risk-assessment)
7. [Technical and Organisational Measures](#7-technical-and-organisational-measures)
8. [Residual Risk Statement](#8-residual-risk-statement)
9. [Supervisory Authority Consultation](#9-supervisory-authority-consultation)
10. [Open Items and Conditions of Approval](#10-open-items-and-conditions-of-approval)
11. [Review Log](#11-review-log)

---

## 1. Why This DPIA Is Mandatory

GDPR Art. 35 requires a DPIA before beginning processing that is "likely to result in a high risk to the rights and freedoms of natural persons." The EDPB guidelines (WP248 rev. 01) enumerate nine criteria; the presence of **two or more triggers a mandatory DPIA**. FORM satisfies five.

| Criterion | FORM situation | Triggered? |
|---|---|---|
| **Special category data** (Art. 9) | Health data: workout performance, body composition trends, HRV baseline, meal logs, ED-screening responses | ✅ Yes |
| **Large-scale processing** | Intended scale: tens of thousands of users; enterprise tenants with hundreds of seats | ✅ Yes |
| **Systematic profiling** | Victor AI coach generates adaptive fitness and nutrition programmes from continuous user data | ✅ Yes |
| **Innovative technology** | On-device computer vision pose estimation; generative AI coaching; voice synthesis | ✅ Yes |
| **Vulnerable data subjects** | Includes users who may have EDs or body-image sensitivity (DEC-018 mitigates; clinical-safety VETO active) | ✅ Yes |
| Data subjects unable to opt out | Consumer tier: voluntary; Enterprise tier: employer-sponsored (potential coercion risk) | 🟡 Partial |
| Data matching or combining | Multi-source aggregation (wearables + CV + nutrition) but not cross-system profiling beyond FORM | 🟡 Partial |
| Physical location tracking | No GPS / location data collected | ❌ No |
| Credit or financial data | No financial data processed (Stripe handles billing; FORM receives only plan status) | ❌ No |

**Conclusion: DPIA is mandatory under GDPR Art. 35.** This document constitutes FORM's formal DPIA.

---

## 2. Processing Activities Inventory

### 2.1 Processing activity table

| Activity ID | Activity | Data categories | Subjects | Processor(s) | Lawful basis |
|---|---|---|---|---|---|
| PA-01 | **Account creation and identity** | Email, display name, provisioning method | All users | Supabase Auth | Contract (Art. 6(1)(b)) |
| PA-02 | **Health profile collection** | Date of birth, biological sex, height, resting HR, HRV baseline, goals, restrictions/injuries | All users (voluntary fields) | Supabase | Explicit consent (Art. 9(2)(a)) |
| PA-03 | **CV-based workout analysis** | Joint angles, rep count, form score — computed on-device; only form_score + metadata stored | All users | On-device ML; Supabase (metadata only) | Contract (Art. 6(1)(b)); consent (Art. 9(2)(a)) for health inference |
| PA-04 | **Meal logging** | Food items, macros (kcal, protein, carbs, fat), meal type, timestamp | Pro and Enterprise users | Supabase | Explicit consent (Art. 9(2)(a)) — food log is health-adjacent |
| PA-05 | **AI coaching (Victor)** | Pseudonymised exercise context: sets, reps, RPE, programme phase. No PII in prompts. | All users | Anthropic (sub-processor) | Contract (Art. 6(1)(b)) |
| PA-06 | **Voice synthesis** | Coaching text cues (no user PII in payload) | Users who enable voice | ElevenLabs (sub-processor) | Contract (Art. 6(1)(b)) |
| PA-07 | **Wearable integration** | HRV, resting HR, sleep stages, step count (HealthKit / Health Connect) | Users who opt in | On-device only (HealthKit / Health Connect APIs) | Explicit consent (Art. 9(2)(a)) |
| PA-08 | **ED-screening** | SCOFF-light responses (3 questions). Result is a UX flag only — not stored as a diagnostic label. | All users at onboarding | Supabase (UX flag only) | Explicit consent + legitimate interest (clinical-safety obligation) |
| PA-09 | **Enterprise aggregate reporting** | Aggregated and k-anonymised tenant wellness summaries (active users, workout count, avg form score, avg duration) | Enterprise tenants | Supabase materialized view | Contract + legitimate interest of enterprise customer (Art. 6(1)(f)) |
| PA-10 | **Audit logging** | Actor ID, action, resource type, IP address, outcome — all privileged actions | All users; FORM employees | Supabase (append-only HMAC chain — DEC-030) | Legal obligation (Art. 6(1)(c)) + legitimate interest (Art. 6(1)(f)) |
| PA-11 | **Product analytics** | Anonymous event stream, opt-out flag, device type; no health event content | All users | PostHog (sub-processor) | Legitimate interest (Art. 6(1)(f)); analytics_opt_out honoured |
| PA-12 | **Error monitoring** | Anonymised crash reports, stack traces; PII scrubbed at SDK level pending formal DPA | All users | Sentry (sub-processor) | Legitimate interest (Art. 6(1)(f)) |

### 2.2 Special category data under Art. 9

The following data elements are classified as special category (health data) and receive heightened protection throughout this DPIA:

| Data element | Activity | Why Art. 9 applies |
|---|---|---|
| HRV baseline | PA-02, PA-07 | Reveals cardiovascular health status |
| Body composition trends | PA-02 (goals field) | Can reveal health conditions |
| Workout performance and form score | PA-03 | Reflects physical capacity and health trajectory |
| Meal log items and macros | PA-04 | Reveals dietary health patterns; ED-adjacent |
| ED-screening UX flag | PA-08 | Mental health adjacent; requires heightened care |
| Sleep stages (wearable) | PA-07 | Mental and physical health indicator |
| Injury restrictions | PA-02 (restrictions field) | Disability-adjacent health data |

---

## 3. Data Flows

### 3.1 Consumer tier data flow

```
User device (iOS / Android)
  │
  ├── CV inference (on-device, Neural Engine)
  │     └── Joint angles computed locally
  │           → form_score (0–100) + rep_count → Supabase [PA-03]
  │           → Raw camera frames: NEVER uploaded
  │
  ├── User input: profile, goals, meal log, wearable opt-in
  │     → Supabase (encrypted at rest, per-tenant KMS for sensitive cols) [PA-02, PA-04, PA-07]
  │
  ├── Coaching request → Cloudflare Worker
  │     → Pseudonymised context assembled (no PII, no raw health values)
  │           → Anthropic API [PA-05]
  │           ← Coaching response → ElevenLabs [PA-06] (if voice enabled)
  │
  └── Anonymous events → PostHog [PA-11]
        └── analytics_opt_out → no events sent
```

### 3.2 Enterprise tier additional data flow

```
Enterprise user (authenticates via SSO → WorkOS/Auth0)
  │
  ├── All consumer flows above, scoped to tenant_id via RLS
  │
  ├── Nightly materialized view refresh (form_admin role, BYPASSRLS)
  │     → tenant_wellness_summary: aggregated, k-anon floor = 5 [PA-09]
  │           → HR/People admin reads ONLY from this MV via admin dashboard
  │           → Individual health rows: ZERO access for any admin role
  │
  └── Audit log: every privileged action with HMAC chain [PA-10]
        → Tenant admin can read own tenant's audit log (action taxonomy only)
        → FORM compliance-officer reads via form_audit role for SOC 2 evidence
```

### 3.3 Data residency

| Tenant region | Primary DB | Backups | Sub-processors |
|---|---|---|---|
| `us-east-1` (default) | AWS N. Virginia | S3 us-east-1 | Supabase US, Anthropic US, ElevenLabs US |
| `eu-central-1` (EU opt-in) | AWS Frankfurt | S3 eu-central-1 | Supabase EU, PostHog EU; Anthropic SCC-covered |
| `eu-west-1` (UK/IE) | AWS Ireland | S3 eu-west-1 | Supabase EU, SCC-covered |

EU enterprise tenants must select a region at contract signing. Region cannot be changed post-deployment without a paid migration engagement and re-execution of data processing addendum.

**Transfer mechanism for US sub-processors (EU tenants):** Standard Contractual Clauses (SCCs) Module 2 (controller→processor). Anthropic, ElevenLabs, and Cloudflare SCCs are in place. See `docs/SOC2_READINESS.md` Sub-Processor Register.

---

## 4. Legal Bases for Processing

### 4.1 Consumer tier

| Processing | Legal basis | Notes |
|---|---|---|
| Account and identity | Art. 6(1)(b) — contract | Necessary to provide the service |
| Health profile | Art. 9(2)(a) — explicit consent | Granular consent per data category at onboarding; withdrawable in Settings |
| CV workout analysis | Art. 9(2)(a) + Art. 6(1)(b) | Health inference from movement requires Art. 9 consent; service delivery requires Art. 6(1)(b) |
| Meal logging | Art. 9(2)(a) — explicit consent | Food logs are health-adjacent; treated as special category |
| Wearable data | Art. 9(2)(a) — explicit consent | Per-device, per-data-type consent (HealthKit/Health Connect model) |
| AI coaching via Anthropic | Art. 6(1)(b) — contract | No health data in prompt payload; pseudonymised context only |
| Voice via ElevenLabs | Art. 6(1)(b) — contract | Coaching text cues only; no user data transmitted |
| Analytics (PostHog) | Art. 6(1)(f) — legitimate interest | Anonymous events; opt-out available; LIA passed (see §5.3) |
| Error monitoring (Sentry) | Art. 6(1)(f) — legitimate interest | PII scrubbed at SDK; necessary for service reliability |
| Audit logging | Art. 6(1)(c) — legal obligation | SOC 2 / GDPR Art. 30 requirement |

### 4.2 Enterprise tier additional bases

| Processing | Legal basis | Notes |
|---|---|---|
| SSO/SCIM provisioning | Art. 6(1)(b) — contract (controller-to-controller with employer) | Joint controller arrangement with enterprise customer; addressed in DPA |
| Aggregate wellness reporting to HR | Art. 6(1)(f) — legitimate interest of enterprise customer | Strictly k-anonymised; individual data never exposed |
| ED-screening | Art. 9(2)(a) — explicit consent + clinical-safety obligation | DEC-018; consent captured before onboarding; result is UX flag only, not a clinical record |

### 4.3 Consent record design

Per DEC-030, consent events are logged to the HMAC-chained audit log with the following schema:

```json
{
  "action": "privacy.consent_granted",
  "resource_type": "consent_record",
  "changes": {
    "consent_version": "v1.2",
    "categories": ["health_profile", "meal_log", "wearable_hrv"],
    "granted_at": "2026-05-17T10:00:00Z",
    "mechanism": "in_app_onboarding_screen"
  }
}
```

Withdrawal is logged as `privacy.consent_withdrawn` with the same structure. Withdrawal triggers immediate cessation of that category's processing; retroactive deletion of previously processed data is offered for health profile and meal logs.

---

## 5. Necessity and Proportionality

### 5.1 Is each data category necessary?

| Data category | Why necessary | Alternative considered | Conclusion |
|---|---|---|---|
| Email address | Account identity; breach notification | Pseudonymous-only account | Email necessary for enterprise SSO matching and legal notification obligations |
| HRV baseline | Calibrates workout intensity recommendations | Remove from initial profile | HRV is the primary readiness signal; removing it degrades coaching quality materially; retained |
| Meal log items | Enables nutritional coaching | Aggregate-only (total kcal, not items) | Item-level needed for macro balance coaching; retained but not aggregated to enterprise dashboard |
| CV form_score | Core product value (movement quality) | Manual self-report | Manual report is inaccurate; CV is the product differentiator; retained |
| Raw CV keypoints | Debug and model improvement | Never upload | **Raw keypoints are NOT uploaded.** Only derived form_score is stored. This is a hard architectural decision. |
| ED-screening responses | Clinical safety (DEC-018) | Skip screen entirely | Skipping creates harm risk; DEC-018 mandates non-skippable screen; result stored as UX flag only (not as clinical label) |
| IP address in audit log | Forensic traceability (DEC-030) | Omit from audit log | Required for SOC 2 CC7 and GDPR Art. 30; retained with 30-day log retention cap for high-frequency events |

### 5.2 Data minimisation evidence

1. **No PII in Anthropic prompts.** Prompts contain only pseudonymised exercise context: `{programme_phase, sets_completed, rpe, equipment}`. User name, email, age, and all health values are excluded. See `docs/SECURITY.md §5`.

2. **CV frames never leave device.** Neural Engine inference runs on-device. Only the scalar form_score and rep_count are transmitted. This is enforced at the architecture level, not just policy.

3. **Meal logs excluded from enterprise aggregates.** The `tenant_wellness_summary` materialized view deliberately omits meal log data. ED-adjacent nutrition data has no legitimate use in employer wellness dashboards.

4. **Body composition excluded from aggregates.** BMI, body fat percentage, and weight trend data are not included in any field exposed to admin-tier roles. See `docs/DATA_MODEL.md §6`.

5. **k-anonymity floor = 5.** Enterprise aggregate views suppress cohort groups with fewer than 5 members. Implemented in SQL (`HAVING COUNT(DISTINCT user_id) >= 5`) and validated at the application response layer before serving to the admin dashboard.

### 5.3 Legitimate interest assessment (LIA) — analytics

| Test | Assessment |
|---|---|
| **Purpose test** — is the interest legitimate? | Yes — product improvement, crash detection, and activation funnel analysis are necessary for a viable business |
| **Necessity test** — is processing necessary for the purpose? | Yes — anonymous event data is the minimum viable instrumentation; PII not required for funnel analysis |
| **Balancing test** — does user's interest override? | No — events are anonymous; opt-out is surfaced at first launch and in Settings; users retain control |
| **Safeguards** | analytics_opt_out flag stops all event transmission; logged in audit log; PostHog configured to exclude user-identifiable fields |

**LIA result: legitimate interest is justified for anonymous product analytics.** Health event *content* (e.g., specific meal items, form_score values) is never included in PostHog events.

---

## 6. Risk Assessment

Risks are scored on **Likelihood × Severity** using a 1–5 scale. Residual risk assessed after mitigations in §7.

### 6.1 Risk register

| Risk ID | Risk description | Likelihood (1–5) | Severity (1–5) | Inherent score | Primary mitigation | Residual likelihood | Residual severity | Residual score |
|---|---|---|---|---|---|---|---|---|
| R-01 | Unauthorised access to individual health data by HR/People admin | 2 | 5 | 10 | RLS `health_profile_owner_only` policy + `workouts_no_admin_peek`; admin API has no health data routes | 1 | 5 | 5 |
| R-02 | Cross-tenant data leakage (one tenant reads another's data) | 2 | 5 | 10 | RLS isolation with `SET LOCAL app.current_tenant_id`; fail-closed (no context = no rows); CI RLS isolation test on every PR | 1 | 5 | 5 |
| R-03 | Security breach exposing health data at rest | 2 | 5 | 10 | AES-256 volume encryption (RDS); per-tenant CMK (KMS) for sensitive columns; HMAC audit chain detects tampering | 1 | 5 | 5 |
| R-04 | Re-identification of individuals from enterprise aggregate data | 2 | 4 | 8 | k-anonymity floor = 5 in SQL + application layer validation; no meal/body data in aggregates | 1 | 4 | 4 |
| R-05 | AI model training on user health data (Anthropic) | 1 | 5 | 5 | Anthropic enterprise DPA prohibits training use; no PII/health in prompts by design | 1 | 4 | 4 |
| R-06 | ED-screening result misused or exposed | 2 | 5 | 10 | DEC-018: result stored as UX flag only (not clinical label); flag excluded from all export APIs and enterprise aggregates; clinical-safety VETO on any feature touching this data | 1 | 4 | 4 |
| R-07 | Wellness-as-punishment: employer uses aggregate data to disadvantage employees | 3 | 5 | 15 | Privacy floor contractually bound (ENTERPRISE.md); aggregate-only; k-anon floor = 5; individual identification not possible; FORM terms prohibit this use | 2 | 4 | 8 |
| R-08 | Employee coercion in employer-sponsored enterprise tier | 3 | 4 | 12 | Voluntary participation stated in employer onboarding materials (CUEC-07); employee can disconnect at any time (Settings → Privacy); technical enforcement of disconnect | 2 | 3 | 6 |
| R-09 | Data breach during international transfer (EU → US sub-processors) | 2 | 4 | 8 | SCCs in place for all US sub-processors; EU data residency option eliminates transfer for EU tenants; Anthropic SCCs in force | 1 | 3 | 3 |
| R-10 | GDPR DSAR not completed within 30-day deadline | 3 | 3 | 9 | `data.export_initiated` logged; JSON export available programmatically; 48-hour secure link delivery | 2 | 3 | 6 |
| R-11 | Inaccurate AI coaching output causes physical harm | 2 | 4 | 8 | CV confidence thresholds — uncertain outputs defer to safe templates; medical disclaimer in ToS; no medical claims; sports-scientist review of exercise programming | 1 | 4 | 4 |
| R-12 | Vendor (sub-processor) breach | 2 | 4 | 8 | DPA with all sub-processors; vendor security review annual; DPAs with Anthropic/ElevenLabs/Supabase/PostHog signed | 2 | 3 | 6 |

### 6.2 Critical risks requiring special attention

**R-07 (Wellness-as-punishment) — highest inherent risk at 15:** Even with contractual prohibitions, an enterprise customer could attempt to use aggregate wellness data in employment decisions. FORM's mitigations are technical (k-anonymity prevents individual identification) and contractual (ToS prohibit the use). The residual risk of 8 is accepted as unavoidable given the nature of employer-sponsored wellness programmes; it cannot be reduced to zero while the enterprise product exists.

**R-01, R-02, R-03 (health data access, cross-tenant leakage, breach) — all 10:** These are the primary GDPR harms (data subject unable to control their health data). Mitigations are substantial and layered (RLS, encryption, CI tests, HMAC chain). Residual risk of 5 remains because no technical system is zero-risk; the mitigation is depth-of-defence.

### 6.3 Risks not in scope

- **Physical harm from exercise not covered here** — this is a product liability and clinical-safety scope concern, not a GDPR privacy risk.
- **Commercial misuse of aggregate data** — governed by ToS and addressed in ENTERPRISE.md no-go customer policy.
- **Cookie consent for marketing site** — covered by GDPR cookie law separately; out of scope for this DPIA which covers health data processing in the product.

---

## 7. Technical and Organisational Measures

### 7.1 Technical measures summary

| Measure | Implementation | Addresses |
|---|---|---|
| **Row-Level Security (fail-closed)** | Every table with user data has RLS. No context = no rows. `form_api` role cannot `BYPASSRLS`. See `docs/DATA_MODEL.md §3`. | R-01, R-02 |
| **Per-tenant encryption at column level** | AWS KMS Customer Managed Key per tenant. `raw_cv_data`, biometric columns encrypted with `pgcrypto`. Annual key rotation. | R-03 |
| **On-device CV inference** | Neural Engine inference; form_score scalar uploaded, not keypoints. Enforced at architecture level (no upload API exists for raw frames). | R-03 (eliminates this vector entirely) |
| **No PII in AI prompts** | Prompt assembly layer strips all PII; uses pseudonymised context only. Validated in CI (prompt content tests). | R-05 |
| **k-anonymity floor** | `HAVING COUNT(DISTINCT user_id) >= 5` in MV SQL + API response validator. Groups < 5 → HTTP 204 (no content), not aggregate. | R-04 |
| **ED-screening isolation** | UX flag only; excluded from all admin APIs, export formats, and the MV. clinical-safety VETO required for any feature touching this flag. | R-06 |
| **HMAC-chained audit log** | Append-only, tamper-evident (DEC-030). Weekly chain verification cron. Break → immediate security alert. | R-03 (tamper detection) |
| **GDPR erasure implementation** | Hard delete across health tables within 30-day grace; backup purge within 60 days; audit log anonymised (user_id removed, action retained). See `docs/DATA_MODEL.md §8.3`. | R-10 |
| **DSAR export (programmatic)** | `data.export_initiated` event triggers background job → JSON archive → 48-hour secure link. | R-10 |
| **TLS 1.3 + HSTS + cert pinning** | Mobile clients pin certificates. All API traffic TLS 1.3 minimum. | R-03, R-09 |
| **CI RLS isolation test** | Every PR runs cross-tenant isolation test. Tenant A user cannot see Tenant B data. Failure blocks merge. | R-02 |

### 7.2 Organisational measures summary

| Measure | Detail | Addresses |
|---|---|---|
| **Privacy-by-design principle** | Health data exclusion from enterprise aggregates is a design constraint, not a configuration option. Cannot be toggled by customer or support. | R-01, R-07 |
| **7 non-negotiable privacy rules** | Documented in `docs/ENTERPRISE.md`. HR never sees individual user data. Any customer request to bypass is declined; deal lost if necessary. | R-07, R-08 |
| **No-go customer policy** | Insurance risk-scoring, government backdoors, wellness-as-punishment use cases are contractually excluded. Deal qualification checklist enforced. | R-07 |
| **clinical-safety VETO** | clinical-safety agent has VETO on any feature touching ED-screening, body image, or individual health display. This is an org-level control. | R-06, R-11 |
| **Complementary User Entity Controls (CUECs)** | Enterprise customers are contractually obligated to: designate a tenant_owner, enforce IdP MFA, notify FORM within 24h of offboarding, inform employees of disconnect right. See `docs/SOC2_READINESS.md`. | R-08 |
| **Sub-processor DPAs** | All sub-processors have signed DPAs before receiving any user data. SCC Module 2 for US transfers. Sentry DPA in progress (PII scrubbed at SDK as compensating control). | R-12 |
| **30-day advance notice of new sub-processors** | Enterprise customers notified before any new sub-processor processes their tenant data. Right to object per GDPR Art. 28(2). | R-12 |
| **Annual vendor security review** | Formal review of each sub-processor's security posture (SOC 2 report, pentest summary, or equivalent). | R-12 |
| **Breach notification: 72h supervisory authority / 1h tenant** | GDPR 72-hour obligation to supervisory authority. P0 tenant notification within 1 hour. See `docs/INCIDENT_RESPONSE.md`. | R-03, R-12 |

---

## 8. Residual Risk Statement

After applying all technical and organisational measures in §7, the residual risk levels are:

| Risk category | Residual score | Acceptability | Rationale |
|---|---|---|---|
| Unauthorised health data access (R-01) | 5 | **Accepted** | Fail-closed RLS + no admin health data routes reduces this to a theoretical vulnerability; routine penetration tests will validate |
| Cross-tenant leakage (R-02) | 5 | **Accepted** | CI isolation test catches regression; residual is theoretical exploit of RLS implementation bug |
| Health data breach at rest (R-03) | 5 | **Accepted** | Layered encryption (volume + CMK + HMAC chain); residual is infrastructure provider compromise |
| Re-identification from aggregates (R-04) | 4 | **Accepted** | k-anon floor = 5 is below EU norms (some DPAs require k ≥ 10 for health data); this is an **open item** — see §10.3 |
| AI training on user data (R-05) | 4 | **Accepted** | Anthropic DPA; no health data in prompts; residual is DPA breach by Anthropic (low, commercially catastrophic for Anthropic) |
| ED-screening misuse (R-06) | 4 | **Accepted** | UX flag only; no API exposure; clinical-safety VETO as org control |
| Wellness-as-punishment (R-07) | 8 | **Accepted with monitoring** | Highest residual risk; cannot be reduced to zero while enterprise product exists; mitigated by k-anon + contract + deal qualification |
| Employee coercion (R-08) | 6 | **Accepted with condition** | Requires CUEC-07 compliance (employer disclosure); FORM cannot enforce employer-side communication |

**Overall residual risk: MEDIUM-HIGH.** The primary residual risk is wellness-as-punishment (R-07) — an inherent tension in employer-sponsored wellness products. FORM's architectural and contractual mitigations reduce this meaningfully but cannot eliminate it. Processing proceeds with these residual risks accepted by the compliance-officer and documented here.

**The DPIA supports proceeding with processing** subject to the open items in §10 being resolved before EU enterprise launch.

---

## 9. Supervisory Authority Consultation

### 9.1 Consultation threshold

GDPR Art. 36 requires prior consultation with the supervisory authority when residual risk after mitigation remains HIGH. FORM's residual risk is assessed as **MEDIUM-HIGH**, not HIGH — specifically because:

1. Processing of special category data has explicit consent as legal basis
2. Technical mitigations (on-device CV, RLS, encryption, k-anonymity) are robust
3. The wellness-as-punishment risk (R-07) is the primary concern, and it is contractually constrained

**Current assessment: supervisory authority consultation is not mandatory.** This assessment should be re-evaluated if:
- k-anonymity floor is reduced below 5
- Body composition or mental health data is added to enterprise aggregates
- FORM begins processing genetic or biometric data for identification purposes

### 9.2 Proactive engagement

Prior to EU enterprise sales launch, FORM will proactively notify the lead supervisory authority (anticipated: Irish DPC, given Supabase EU hosting in `eu-west-1`) of FORM's processing activities and this DPIA. This is voluntary and establishes a good-faith compliance posture.

---

## 10. Open Items and Conditions of Approval

### 10.1 Blocking items (must resolve before EU enterprise launch)

| Item | Owner | Deadline |
|---|---|---|
| **10.1.1 Privacy policy published** at form.coach/privacy covering all data categories in §2.1 | compliance-officer | Month 1 |
| **10.1.2 Consent version management** — privacy policy version captured in `privacy.consent_granted` audit event | security-engineer | Month 1 |
| **10.1.3 In-app consent granularity** — separate consent collection per health data category (health profile, meal log, wearable) rather than a single consent screen | platform-engineer | Month 2 |
| **10.1.4 Sentry DPA signed** — PII scrubbing at SDK is a compensating control, not a permanent solution | compliance-officer | Month 1 |
| **10.1.5 DSAR handling runbook** — documented 30-day response procedure, tested end-to-end | compliance-officer + security-engineer | Month 2 |

### 10.2 Required before observation period (SOC 2 Month 3)

| Item | Owner | Deadline |
|---|---|---|
| **10.2.1 Records of Processing Activities (ROPA)** — formal Art. 30 register covering all activities in §2.1 | compliance-officer | Month 2 |
| **10.2.2 CUEC-07 template** — standard employer communication template for informing employees about FORM enrollment and their rights | customer-success + compliance-officer | Month 2 |
| **10.2.3 DPA template for enterprise customers** — controller-to-controller DPA addendum covering employer and FORM respective obligations | Legal review | Month 3 |
| **10.2.4 Proactive DPC notification** (if EU enterprise) | compliance-officer | Before first EU deal closes |

### 10.3 Under review (non-blocking)

| Item | Status |
|---|---|
| **k-anonymity floor at 5** — EDPB and some EU DPAs may expect k ≥ 10 for health data. Legal review required before EU enterprise sales. | 🟡 In review |
| **DPIA for any new AI features** — any new use of Victor AI that processes health data in a materially different way requires a DPIA update | 🟡 Ongoing obligation |
| **DPIA for wearable genetic markers** — if Oura/Whoop future data includes genetic markers, this DPIA must be re-run | 🟡 Future trigger |

---

## 11. Review Log

| Version | Date | Author | Summary |
|---|---|---|---|
| v0.1 | 2026-05-17 | compliance-officer (enterprise-builder cloud session) | Initial DPIA covering all processing activities for consumer and enterprise tier. Legal review pending. Not yet submitted to supervisory authority. |

**Next mandatory review:** Before EU enterprise first deal close, or within 12 months (May 2027), whichever is earlier.

---

**v0.1 · травень 2026**  
**Owner: compliance-officer + security-engineer**  
**Status: Draft — pending legal review before activation**

*This document is FORM's formal Data Protection Impact Assessment under GDPR Art. 35. It is an internal compliance document. It is disclosed to enterprise customers under NDA on request as part of security due diligence. It is not published publicly.*
