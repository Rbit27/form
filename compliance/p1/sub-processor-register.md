# GDPR Art. 28 / SOC 2 CC9.2 Sub-Processor Register

**Document ID:** P1-SUB-001  
**Remediates:** P-GAP-005 — sub-processor register not formalised as standalone auditor exhibit  
**Regulatory basis:** GDPR Art. 28 (processor obligations) · GDPR Art. 13(1)(e) (transparency disclosure) · SOC 2 CC9.2 (vendor risk management)  
**Owner:** compliance-officer  
**Status:** 🟡 Active — Sentry (SP-06) DPA in progress; compensating control in place (see §6)  
**Effective date:** 2026-05-21  
**Review cycle:** Annual — Q1 January; or within 30 days of any addition, removal, or material change  
**Artifact path:** `compliance/p1/sub-processor-register.md`

---

## 1. Purpose and Scope

### 1.1 Purpose

GDPR Art. 28(1) requires that a controller engage only processors providing sufficient guarantees to implement appropriate technical and organisational measures such that processing meets GDPR requirements and ensures protection of data subject rights. Art. 28(3) requires that processing be governed by a binding contract (Data Processing Agreement). Art. 13(1)(e) requires that data subjects be informed of any recipients or categories of recipients of their personal data, including sub-processors.

SOC 2 CC9.2 requires that FORM assess and monitor the risks associated with third-party service providers who have access to, process, or could affect the security and availability of FORM systems and data.

This document serves four functions:

1. **Formal register** — the authoritative, version-controlled list of all entities that process personal data on FORM's behalf
2. **Auditor exhibit** — the primary evidence artifact for CC9.2 vendor management and GDPR Art. 28 processor compliance, filed as PRE-SUB-001 in the SOC 2 evidence library
3. **Enterprise disclosure** — provided to enterprise customers on request and referenced from the published sub-processor list at `security.form.coach/sub-processors`
4. **Gap closure record** — closes P-GAP-005 (sub-processor register not formalised), contributes to closure of PRE-02, PRE-08, and PRE-09

### 1.2 Scope

This register covers all natural or legal persons who:

- receive personal data from FORM's systems in the course of providing a service to FORM, or
- operate infrastructure on which FORM's personal data resides or transits, or
- process personal data on FORM's documented instruction

**Excluded from scope** (independent controllers — see §5.8):

- Apple Inc (iOS App Store payment processing — Apple is an independent controller for payment PII; FORM receives receipt tokens only)
- Google LLC (Android Play Store — same as Apple)

These entities are listed as SP-07 and SP-08 in the register for completeness and enterprise disclosure purposes, but they are not sub-processors within the meaning of GDPR Art. 4(8) with respect to FORM's processing activities.

### 1.3 Controller Identity

**Data Controller:** FORM (operated by [Legal entity name TBD — to be completed before enterprise DPA countersigning])  
**Privacy contact:** `privacy@form.coach`  
**Lead supervisory authority (anticipated):** Irish Data Protection Commission (DPC), given Supabase EU hosting in `eu-west-1`

---

## 2. FORM Data Classification Reference

The following table defines the personal data categories that FORM processes and that may be transmitted to or processed by sub-processors. The Art. 9 column determines which sub-processors require the highest-tier protective measures.

| Category ID | Category name | Description | GDPR Art. 9 special category? | Stored in |
|---|---|---|---|---|
| DC-01 | Identity data | Email, display name, account creation method, provisioning metadata | No | Supabase Auth |
| DC-02 | Health profile | Date of birth, biological sex, height, resting HR, HRV baseline, fitness goals, injury flags, dietary restrictions | **Yes — Art. 9(1) health data** | Supabase (`user_profile`) |
| DC-03 | Workout / exercise data | Workout sessions, sets, reps, RPE, session date, programme phase | **Yes — health status inference** | Supabase (`workout_sessions`, `sets`) |
| DC-04 | CV / pose data | Computer-vision form-check metadata: joint angles, rep count, form score; computed on-device; only form_score + metadata stored server-side | **Yes — biometric-adjacent; Art. 9 conservative classification** | Supabase (`cv_sessions`) |
| DC-05 | Wearable / sensor data | HRV, resting HR, sleep duration, steps, recovery score sourced from Apple Health / Health Connect | **Yes — Art. 9(1) health data** | Supabase (`wearable_readings`) |
| DC-06 | Coaching turns | AI coaching dialogue: exercise context, coaching responses, coaching history; pseudonymised in prompts (no PII in payload to Anthropic) | **Yes — contains health goals, injury context** | Supabase (`coaching_turns`) |
| DC-07 | Meal log | Food items, macros (kcal, protein, carbs, fat), meal type, timestamp | **Yes — Art. 9 health/dietary data** | Supabase (`meal_log`) |
| DC-08 | Payment metadata | iOS IAP receipt tokens, subscription tier, renewal status; no card data | No — payment metadata only; card data held by Apple/Google as independent controllers | Supabase (receipt tokens) |
| DC-09 | Error telemetry | Crash reports, stack traces, anonymised user IDs; health-adjacent fields scrubbed at SDK layer before transmission | No — PII scrubbed before leaving FORM's boundary; residual risk is compensating-controlled (see §6) | Sentry |
| DC-10 | Analytics events | Pseudonymised event stream, device type, feature interaction flags, `analytics_opt_out` flag; no health event content | No — no health data by configuration; opt-out honoured | PostHog |

---

## 3. Sub-Processor Tier Classification

All sub-processors are assigned a tier that governs the minimum protective measures required in the DPA and the frequency of security review.

| Tier | Criteria | DPA requirement | SCC requirement | Review frequency |
|---|---|---|---|---|
| **Tier 1** | Processes Art. 9 special-category health data (DC-02 through DC-07) on FORM's behalf | Full GDPR Art. 28 DPA; explicit Art. 9 provisions; sub-processor restriction clause; audit right clause | SCC Module 2 (controller→processor) mandatory | Annual + any material change |
| **Tier 2** | Processes personal data that is not Art. 9 (DC-01, DC-08, DC-09, DC-10) | GDPR Art. 28 DPA; standard data protection clauses | SCC Module 2 required for US-EU transfers | Annual |
| **Tier 3** | Payment passthrough or independent controller; FORM receives only encrypted tokens or anonymised data; no instruction relationship | Contractual terms sufficient; independent controller notice obligations | Not applicable (independent controller) | Annual review of independent controller terms |

---

## 4. Master Sub-Processor Register

The following table constitutes the authoritative register. Column definitions:

- **Art. 9?** — Does this sub-processor receive Art. 9 special-category health data from FORM?
- **Tier** — Classification per §3
- **DPA status** — Current executed agreement status
- **SCC status** — Standard Contractual Clauses (SCCs) Module 2 status for US-EU transfers
- **Processing location** — Primary region where FORM's data is processed
- **Legal basis** — GDPR lawful basis under which FORM transmits data to this processor

| ID | Sub-processor | Corporate entity | HQ | Purpose | Data categories | Art. 9? | Tier | DPA status | SCC status | Processing location | Legal basis |
|---|---|---|---|---|---|---|---|---|---|---|---|
| SP-01 | **Anthropic** | Anthropic PBC | San Francisco, US | LLM inference — Victor AI coach (coaching cue generation, exercise context classification) | DC-06 (pseudonymised exercise context; no PII in prompt by design — see `docs/SECURITY.md §5`) | **Yes** — coaching turns contain health goals and injury context | 1 | ✅ Signed | ✅ SCC Module 2 (controller→processor) | US (`api.anthropic.com`); no EU region currently — SCC-covered | Art. 6(1)(b) contract; Art. 9(2)(a) explicit consent for health-context inference |
| SP-02 | **Supabase** | Supabase Inc | San Francisco, US | Managed PostgreSQL database, Supabase Auth (sessions, SAML/OIDC SSO), Edge Functions, file storage | DC-01, DC-02, DC-03, DC-04, DC-05, DC-06, DC-07, DC-08 (all user and tenant data including all Art. 9 categories) | **Yes** — primary store for all Art. 9 data | 1 | ✅ Signed | ✅ SCC Module 2; EU data residency available (`eu-central-1`); enterprise tenants routed to EU region | US `us-east-1` (default) / EU `eu-central-1` (enterprise opt-in) | Art. 6(1)(b) contract; Art. 9(2)(a) explicit consent; Art. 6(1)(c) legal obligation (audit log) |
| SP-03 | **Cloudflare** | Cloudflare Inc | San Francisco, US | CDN, DDoS protection, DNS, edge compute (Cloudflare Workers) — API gateway layer | IP addresses, request metadata (no user health data in Workers; health data does not transit Cloudflare layer by design) | No — no Art. 9 data in Cloudflare boundary | 2 | ✅ Enterprise DPA signed | ✅ SCC Module 2 | Global edge; EU data does not exit EU PoPs before reaching Supabase `eu-central-1` | Art. 6(1)(b) contract; Art. 6(1)(f) legitimate interest (network security) |
| SP-04 | **ElevenLabs** | ElevenLabs Inc | New York, US | Text-to-speech voice synthesis for Victor coaching cues | Coaching cue text only (no user PII in payload by design — see `docs/GDPR_DPIA.md PA-06`) | No — coaching text cues only; no user identity or health data transmitted | 2 | ✅ Signed | ✅ SCC Module 2 | US | Art. 6(1)(b) contract |
| SP-05 | **PostHog** | PostHog Inc | San Francisco, US | Product analytics — pseudonymised event stream, feature flags, funnel analysis | DC-10 (pseudonymised events, `analytics_opt_out` flag; no health event content by configuration) | No — health data excluded by configuration; opt-out available | 2 | ✅ Signed | ✅ SCC Module 2; EU Cloud elected (Frankfurt) | EU Cloud (Frankfurt) — no SCC required for EU tenants; SCC applies for US tenants | Art. 6(1)(f) legitimate interest; LIA passed (see `docs/GDPR_DPIA.md §5.3`); analytics_opt_out honoured |
| SP-06 | **Sentry** | Functional Software Inc | San Francisco, US | Error monitoring, crash reporting | DC-09 (error stack traces, anonymised user IDs; health-adjacent fields scrubbed at SDK `beforeSend` hook — see §6 for compensating control detail) | **No** — compensating control prevents Art. 9 data from reaching Sentry; if scrubber fails, treated as potential Art. 33 breach (see §6.4) | 2 | 🟡 **In progress** — DPA under review; 60-day target from 2026-05-21 | 🟡 **Pending** — SCC Module 2 pending DPA signature | US (EU region available; migration planned on DPA execution) | Art. 6(1)(f) legitimate interest (service reliability) |
| SP-07 | **Apple** | Apple Inc | Cupertino, US | iOS App Store — in-app purchase processing | Receipt tokens only (Apple is independent controller for payment PII; FORM does not process cardholder data) | No | 3 (independent controller) | Apple Developer Program terms (independent controller agreement, not a DPA) | Not applicable | Apple servers | Art. 6(1)(b) contract (receipt validation only); Apple is independent controller for payment processing |
| SP-08 | **Google** | Google LLC | Mountain View, US | Android Play Store — in-app purchase processing | Receipt tokens only (Google is independent controller for payment PII; FORM does not process cardholder data) | No | 3 (independent controller) | Google Play Developer Distribution Agreement (independent controller agreement, not a DPA) | Not applicable | Google servers | Art. 6(1)(b) contract (receipt validation only); Google is independent controller for payment processing |

---

## 5. Art. 9 Processing Disclosure per Sub-Processor

GDPR Art. 9 processing requires explicit consent from the data subject (Art. 9(2)(a)) and heightened protective measures from any processor that handles the data. This section documents the Art. 9 exposure and protective measures for each sub-processor individually.

### 5.1 SP-01 Anthropic — Art. 9 Exposure

**Exposure:** Anthropic receives DC-06 coaching turns. These are pseudonymised at the application layer (user name, email, age, and explicit health values are excluded from prompts) but the content of coaching turns — exercise context, RPE, injury references, programme phase, coaching history — constitutes health-status-inferrable data within the meaning of GDPR Art. 9(1).

**Protective measures:**

- Anthropic enterprise DPA prohibits use of FORM's data for model training
- Prompt construction enforced in `docs/SECURITY.md §5` — explicit blocklist of fields forbidden in prompts
- No PII fields (`user_id`, `email`, `name`, `dob`, `height_cm`, `weight_kg_goal`) are included in Anthropic API calls by implementation design
- SCC Module 2 in place for EU-US transfer
- DPIA assessed in `docs/GDPR_DPIA.md R-05`: residual risk rated LOW (commercially catastrophic for Anthropic to breach DPA)

**GDPR legal basis:** Art. 9(2)(a) explicit consent; Art. 6(1)(b) performance of contract.

### 5.2 SP-02 Supabase — Art. 9 Exposure

**Exposure:** Supabase is the primary datastore and processes the full set of Art. 9 categories: DC-02 health profile, DC-03 workout data, DC-04 CV/pose data, DC-05 wearable data, DC-06 coaching turns, DC-07 meal log. This is the highest Art. 9 exposure of any sub-processor.

**Protective measures:**

- Supabase DPA signed; SOC 2 Type II certified
- EU data residency available at `eu-central-1` and `eu-west-1`; enterprise tenants routed to EU region to eliminate SCC transfer requirement
- Row-Level Security (RLS) enforced — each tenant's data is isolated at the database policy layer; see `docs/DATA_MODEL.md`
- Per-tenant KMS encryption for sensitive columns (health_profile fields); AES-256 at rest
- TLS 1.3 in transit
- Audit log (DEC-030 HMAC chain) stored in Supabase append-only table; retention 7 years
- PITR (point-in-time recovery) enabled on Team plan; backup procedures documented in `docs/DATA_MODEL.md §10`
- Supabase is contractually prohibited from accessing customer data except for operational support under explicit instruction
- SCC Module 2 applies for US-region tenants

**GDPR legal basis:** Art. 9(2)(a) explicit consent; Art. 6(1)(b) performance of contract; Art. 6(1)(c) legal obligation (audit log retention).

### 5.3 SP-03 through SP-08 — No Art. 9 Exposure

| Sub-processor | Reason Art. 9 data does not reach this processor |
|---|---|
| SP-03 Cloudflare | Workers API gateway layer handles routing and auth tokens only; health data is not present in request headers or bodies at the Cloudflare layer; Workers proxy to Supabase without inspecting body content for health fields |
| SP-04 ElevenLabs | Receives coaching cue text strings only (e.g. "Good depth on that squat — keep your chest up for the next set"); user identity, health profile, and workout data are not included in TTS API payloads by design |
| SP-05 PostHog | Analytics events are constructed to exclude health event content; product events record feature interactions (screen views, button taps, feature flag evaluations), not health metric values; `analytics_opt_out` flag stops all transmission |
| SP-06 Sentry | `beforeSend` hook scrubs health-adjacent fields before any event leaves the mobile client or Worker (see §6 for full detail); if scrubber fails, Art. 33 notification obligation is assessed |
| SP-07 Apple | Independent controller; FORM receives only cryptographic receipt tokens from Apple IAP; payment PII is never transmitted to FORM's systems |
| SP-08 Google | Same as SP-07 — independent controller; FORM receives only receipt tokens |

---

## 6. SP-06 Sentry — Compensating Control for Pending DPA

### 6.1 Status

As of the effective date of this document (2026-05-21), the Sentry DPA (GDPR Art. 28 contract) and corresponding SCC Module 2 are in progress. DPA execution target: 60 days from 2026-05-21 (deadline: **2026-07-20**). This gap is tracked as VR-02 in `docs/SOC2_READINESS.md §14.2` and PRE-08 in the Pre-Observation Period Readiness Checklist.

Processing EU user error telemetry without a signed DPA constitutes a potential GDPR Art. 28 violation. The compensating control described in this section reduces the risk of personal data (and specifically Art. 9 data) leaving FORM's processing boundary and reaching Sentry in the absence of a DPA.

### 6.2 Compensating Control — `beforeSend` PII Scrubber

All Sentry SDK initialisations (React Native mobile client and Cloudflare Workers backend) must include a `beforeSend` hook that removes health-adjacent fields from every event payload before the Sentry SDK transmits it. This is the authoritative specification for the scrubber.

```typescript
import * as Sentry from "@sentry/react-native"; // or @sentry/cloudflare for Workers

/**
 * Health-adjacent field blocklist.
 * Any key matching these names is removed from Sentry event extras,
 * breadcrumb data, and exception context before transmission.
 *
 * Add to this list whenever a new health-adjacent field is introduced
 * to the data model. Requires a corresponding test in
 * __tests__/sentry-scrubber.test.ts.
 *
 * Maintained by: platform-engineer
 * Last reviewed: 2026-05-21
 * Compliance cross-reference: CR-02 (SOC2_READINESS.md §14.2), P1-SUB-001 §6
 */
const HEALTH_FIELD_BLOCKLIST: ReadonlySet<string> = new Set([
  "workout_data",
  "body_metrics",
  "coaching_context",
  "health_conditions",
  "clinical_flags",
  "coach_sessions",
  "hrv_value",
  "resting_hr",
  "sleep_duration",
  "wearable_readings",
  "meal_log",
  "form_score",
  "injury_flags",
  "health_profile",
]);

/**
 * Recursively scrub all health-adjacent keys from an object.
 * Returns a new object — does not mutate the original.
 */
function scrubHealthFields(
  obj: Record<string, unknown>
): Record<string, unknown> {
  const scrubbed: Record<string, unknown> = {};
  for (const [key, value] of Object.entries(obj)) {
    if (HEALTH_FIELD_BLOCKLIST.has(key)) {
      scrubbed[key] = "[scrubbed by FORM compliance P1-SUB-001]";
    } else if (value !== null && typeof value === "object" && !Array.isArray(value)) {
      scrubbed[key] = scrubHealthFields(value as Record<string, unknown>);
    } else {
      scrubbed[key] = value;
    }
  }
  return scrubbed;
}

Sentry.init({
  dsn: process.env.SENTRY_DSN,
  environment: process.env.APP_ENV,

  beforeSend(event) {
    // Scrub extras
    if (event.extra) {
      event.extra = scrubHealthFields(event.extra as Record<string, unknown>);
    }

    // Scrub breadcrumb data
    if (event.breadcrumbs?.values) {
      event.breadcrumbs.values = event.breadcrumbs.values.map((crumb) => ({
        ...crumb,
        data: crumb.data
          ? scrubHealthFields(crumb.data as Record<string, unknown>)
          : crumb.data,
      }));
    }

    // Scrub exception values (message strings, mechanism metadata)
    if (event.exception?.values) {
      event.exception.values = event.exception.values.map((exc) => ({
        ...exc,
        mechanism: exc.mechanism
          ? {
              ...exc.mechanism,
              data: exc.mechanism.data
                ? scrubHealthFields(exc.mechanism.data as Record<string, unknown>)
                : exc.mechanism.data,
            }
          : exc.mechanism,
      }));
    }

    return event;
  },
});
```

### 6.3 Implementation Status

| Component | Status | Target milestone | Owner |
|---|---|---|---|
| React Native mobile `beforeSend` hook installed | 🟡 Partial — hook specified; CI test coverage not yet verified | M2 | mobile-engineer |
| Cloudflare Workers backend Sentry SDK `beforeSend` hook | 🔴 Not yet implemented — Workers SDK not yet installed | M2 | platform-engineer |
| Automated CI test for scrubber (fields must not appear in mock Sentry payload) | 🔴 Not yet implemented — blocking for DPA compensating control evidence | M2 | platform-engineer |
| Sentry `FORM-HEALTH-LEAK-001` alert rule (fires if blocked field appears in any event) | 🔴 Not yet configured | M2 | security-engineer |
| EU Sentry data region migration (on DPA execution) | Planned — pending DPA signature | On DPA execution | compliance-officer + platform-engineer |

The `beforeSend` scrubber implementation and automated CI test (items 1–4 above) must be complete and evidence-filed before the compensating control can be relied upon by auditors as PRE-08 mitigation. Until then, the Sentry integration scope must not be expanded beyond its current footprint (mobile crash reporting only — no session replay, no PII-enriched contexts).

### 6.4 Scrubber Failure Protocol

If Sentry alert rule `FORM-HEALTH-LEAK-001` fires (a health-adjacent field key detected in a transmitted Sentry event payload), the following actions are required:

1. **Immediate (within 1 hour):** Disable Sentry DSN ingestion for the affected environment; contact `privacy@form.coach`
2. **Within 4 hours:** Assess whether the leaked field constitutes personal data and whether any EU data subject is affected
3. **Within 24 hours:** Determine whether GDPR Art. 33 breach notification to supervisory authority is triggered (threshold: risk to rights and freedoms of natural persons)
4. **Within 72 hours if Art. 33 triggered:** Notify lead supervisory authority (anticipated Irish DPC)
5. **Remediation:** Fix the scrubber; add the missed field to `HEALTH_FIELD_BLOCKLIST`; deploy; verify CI test passes; re-enable Sentry DSN
6. **Post-incident:** File incident report in `compliance/incidents/`; update this register version with scrubber incident note; DEC-030 `compliance.scrubber_failure` event

---

## 7. DPA and SCC Status Matrix

This table is the auditor-facing summary of Art. 28 contract status for all sub-processors. It is the evidence artifact for PRE-08 and PRE-09.

| ID | Sub-processor | DPA status | DPA date | SCC status | SCC module | Transfer mechanism | Certification |
|---|---|---|---|---|---|---|---|
| SP-01 | Anthropic PBC | ✅ Signed | [Confirm date — file in `compliance/dpa/anthropic-dpa-executed.pdf`] | ✅ In place | Module 2 (controller→processor) | SCCs + Anthropic EU DPA addendum | SOC 2 Type II |
| SP-02 | Supabase Inc | ✅ Signed | [Confirm date — file in `compliance/dpa/supabase-dpa-executed.pdf`] | ✅ In place | Module 2; EU region available | EU data residency eliminates transfer for EU tenants; SCCs for US-region tenants | SOC 2 Type II |
| SP-03 | Cloudflare Inc | ✅ Enterprise DPA | [Confirm date — file in `compliance/dpa/cloudflare-dpa-executed.pdf`] | ✅ In place | Module 2 + Cloudflare EU DPA | Cloudflare EU Addendum (UK IDTA equivalent also available) | SOC 2 Type II + ISO 27001 |
| SP-04 | ElevenLabs Inc | ✅ Signed | [Confirm date — file in `compliance/dpa/elevenlabs-dpa-executed.pdf`] | ✅ In place | Module 2 | SCCs | SOC 2 Type II |
| SP-05 | PostHog Inc | ✅ Signed | [Confirm date — file in `compliance/dpa/posthog-dpa-executed.pdf`] | ✅ In place (not required for EU tenants — EU Cloud hosting) | Module 2 for US tenants | EU Cloud Frankfurt eliminates transfer for EU tenants | SOC 2 Type II |
| SP-06 | Functional Software Inc (Sentry) | 🟡 **In progress** — target 2026-07-20 | — | 🟡 **Pending** — contingent on DPA execution | Module 2 (planned) | Compensating control active (§6); EU region migration planned on DPA execution | SOC 2 Type II |
| SP-07 | Apple Inc | Independent controller agreement (Apple Developer Program terms) | N/A | Not applicable | N/A — independent controller | Independent controller; no data transfer instruction relationship | PCI DSS; Apple App Store data protection terms |
| SP-08 | Google LLC | Independent controller agreement (Google Play Developer Distribution Agreement) | N/A | Not applicable | N/A — independent controller | Independent controller; no data transfer instruction relationship | PCI DSS; Google Play data protection terms |

**PRE-08 status:** 🟡 Partial — DPAs confirmed for SP-01 through SP-05; SP-06 in progress; SP-07 and SP-08 are independent controllers (DPA not applicable).

**PRE-09 status:** 🟡 Partial — SCCs in place for SP-01 through SP-05; SP-06 SCCs pending DPA execution.

---

## 8. Customer Disclosure Language for Enterprise DPAs

The following standardised language is for inclusion in FORM's enterprise MSA / DPA template. It constitutes FORM's Art. 28(2) disclosure obligation and objection mechanism.

---

**Sub-Processor Disclosure and Objection Procedure**

FORM uses the sub-processors listed at `https://security.form.coach/sub-processors` (the "Sub-Processor List") to provide the Service. By entering into this Agreement, Customer provides general written authorisation for FORM to engage sub-processors as listed.

**Changes to sub-processors.** FORM will give Customer not less than **30 days' prior written notice** before any new sub-processor begins processing Customer Personal Data. Notice will be delivered by: (a) email to the Customer's designated privacy contact on record; and (b) status page announcement at `https://status.form.coach`. The Sub-Processor List "last updated" date will be updated at the same time.

**Objection window.** Customer may object to a new sub-processor by sending written notice to `privacy@form.coach` within **10 days** of receiving the sub-processor addition notice. The notice must state the specific reasonable grounds for the objection (which must be limited to data protection concerns). If Customer objects and FORM cannot accommodate the objection through a technically and commercially reasonable alternative, either party may terminate the relevant Order Form(s) without penalty upon 30 days' written notice, solely with respect to the processing that requires the new sub-processor.

**Removal of sub-processors.** Removals do not require advance notice but FORM will update the Sub-Processor List promptly upon removal and notify Customer's privacy contact by email.

**FORM's obligations to sub-processors.** FORM imposes data protection obligations on each sub-processor equivalent to those imposed on FORM under this Agreement, including requirements to: (i) process personal data only on FORM's documented instructions; (ii) implement appropriate technical and organisational security measures; (iii) assist FORM in meeting its obligations regarding data subject rights; (iv) delete or return personal data upon termination; (v) permit audit by FORM or an authorised auditor.

---

## 9. Sub-Processor Addition Workflow

Adding a new sub-processor after FORM is in the SOC 2 observation period requires all seven steps below. For additions before the observation period, Steps 1–3 are minimum requirements; Steps 4–7 must be completed before the observation period opens.

| Step | Action | Owner | Blocking gate | Evidence artifact |
|---|---|---|---|---|
| 1 | **Security review** — complete vendor assessment per `docs/SOC2_READINESS.md §17.3`; confirm SOC 2 / ISO 27001 certification or equivalent; assign tier classification per §3 of this document; document risk score | compliance-officer + security-engineer | Vendor must pass Tier risk threshold before DPA negotiation begins | Completed vendor assessment form filed to `compliance/vendor-review/YYYY/VENDOR-assessment.md` |
| 2 | **DPA execution** — negotiate and execute GDPR Art. 28 DPA; confirm DPA covers: purpose limitation, security obligations, sub-processor change notification (30 days), audit rights, data deletion on termination, breach notification ≤24 hours | compliance-officer (outside counsel for Tier 1 vendors) | DPA must be countersigned before vendor begins processing personal data; no exceptions | Executed DPA PDF filed to `compliance/dpa/VENDOR-dpa-executed.pdf`; date confirmed in this register |
| 3 | **SCC execution** — confirm SCC Module 2 (controller→processor) or EU adequacy mechanism; obtain Transfer Impact Assessment (TIA) for Tier 1 vendors | compliance-officer | Required for any US-based Tier 1 or Tier 2 processor receiving EU personal data | SCCs filed to `compliance/scc/VENDOR-scc-executed.pdf`; TIA filed if required |
| 4 | **Art. 9 classification** — determine whether the new processor will receive any DC-02 through DC-07 categories; if yes, promote to Tier 1; update this register's Art. 9 column; confirm DPA includes explicit Art. 9 provisions | compliance-officer | If Tier 1 and DPA does not contain Art. 9 provisions, re-negotiate before proceeding | This register updated; version incremented; git commit hash recorded |
| 5 | **Customer notification** — send 30-day advance written notice to all enterprise tenants per §8 customer disclosure language; log notification date, recipient list, and any objections to `compliance/subprocessor-notices/YYYY-MM-VENDOR.md` | compliance-officer | Vendor must not begin processing enterprise tenant data until 30-day notice window has elapsed; objections must be resolved | Notification log filed; no unresolved objections confirmed |
| 6 | **Register update** — add vendor to this document (new SPxx row) and to `security.form.coach/sub-processors` public list; update "last updated" date; update DPA/SCC status matrix in §7; version-increment this document | compliance-officer | Public list must be updated no later than the effective date of sub-processor addition | Git commit hash of updated register; published page screenshot filed as CC9.2 evidence |
| 7 | **Audit event** — emit DEC-030 `compliance.subprocessor_added` event with: vendor name, tier, DPA executed date, SCC status, Art. 9 exposure boolean, effective date, notification log path | compliance-officer (triggered by platform-engineer) | Audit event must be logged before vendor integration goes to production | DEC-030 event in HMAC audit chain; event ID recorded in this register's change log |

**Emergency exception:** If a critical operational need requires using a new sub-processor in less than 30 days, compliance-officer and founder must jointly approve an exception in writing. The exception must be filed to `compliance/subprocessor-notices/YYYY-MM-VENDOR-emergency.md` and includes: business justification, Art. 9 risk assessment, compensating control specification, and commitment to complete Steps 1–7 within 14 days.

---

## 10. Sub-Processor Removal Workflow

When a sub-processor is removed or replaced:

| Step | Action | Owner | Evidence artifact |
|---|---|---|---|
| 1 | **Data deletion confirmation** — obtain written confirmation from the departing processor that all FORM personal data has been deleted or returned per the DPA termination clause; confirm deletion timeline | compliance-officer | Deletion confirmation email or letter filed to `compliance/dpa/VENDOR-deletion-confirmation-YYYY-MM.pdf` |
| 2 | **Credential revocation** — revoke all FORM credentials and API keys held by the departing processor; confirm via access review checklist | security-engineer | Access review record updated; `compliance.access_revoked` DEC-030 event emitted |
| 3 | **Register update** — mark processor as removed in this document with removal date; update `security.form.coach/sub-processors`; remove from active DPA/SCC matrix in §7 | compliance-officer | Git commit of updated register; published page screenshot |
| 4 | **Customer notification** — notify enterprise tenant privacy contacts that the sub-processor has been removed; no advance notice required for removals, but prompt notification is required as a good-faith obligation | compliance-officer | Notification log filed to `compliance/subprocessor-notices/YYYY-MM-VENDOR-removal.md` |
| 5 | **Audit event** — emit DEC-030 `compliance.subprocessor_removed` event | compliance-officer | DEC-030 event in HMAC audit chain |
| 6 | **7-year record retention** — retain the executed DPA and SCCs for 7 years from the termination date per GDPR Art. 28(3)(h) record-keeping requirement and SOC 2 evidence retention policy | compliance-officer | DPA and SCCs moved to `compliance/dpa/archive/`; not deleted |

---

## 11. Annual Review Procedure

This register must be reviewed annually in Q1 January as part of the annual vendor security review (`docs/SOC2_READINESS.md §17.5`). The review confirms that the register remains accurate, all DPAs are current, and no new sub-processors have been added without completing the addition workflow.

| Step | Action | Owner | Month | Evidence artifact |
|---|---|---|---|---|
| 1 | **Currency check** — confirm no new processors added without 30-day notice; compare running git log of register against customer notification log | compliance-officer | January | Register git log comparison; notification log currency confirmation |
| 2 | **DPA review** — confirm all Tier 1 and Tier 2 DPAs remain current and have not expired; request updated DPA if any have lapsed; confirm Sentry DPA executed (PRE-08 closure) | compliance-officer | January | Updated §7 DPA/SCC matrix; executed DPA file dates confirmed |
| 3 | **SCC review** — confirm SCCs remain valid under current transfer mechanism rules; assess any new adequacy decisions or supervisory authority guidance that may affect transfer mechanisms | compliance-officer (outside counsel if required) | January | SCC review memo filed to `compliance/vendor-review/YYYY/scc-review-memo.md` |
| 4 | **Security certification review** — confirm all Tier 1 and Tier 2 processors have current SOC 2 Type II or equivalent certification; request bridge letter if report is more than 12 months old | security-engineer | January | Certification file dates; bridge letters filed to `compliance/vendor-review/YYYY/VENDOR-bridge-letter.pdf` |
| 5 | **Art. 9 exposure review** — confirm no new data categories have been added to any processor's scope without this register being updated; confirm `beforeSend` scrubber test coverage for Sentry remains active | compliance-officer + security-engineer | January | Art. 9 column confirmed accurate; Sentry scrubber CI test output filed |
| 6 | **Sign-off** — compliance-officer signs annual review memo confirming register is accurate and all items above are complete; memo references this document version and includes git commit hash | compliance-officer | January | Annual review memo filed to `compliance/vendor-review/YYYY/sub-processor-annual-review-YYYY.md`; git commit hash of reviewed register version |

**SOC 2 evidence package:** Steps 1–6 above, completed within the January window and cross-referencing each other, constitute the CC9.2 annual vendor management evidence package. Evidence must be filed to `compliance/vendor-review/YYYY/` by 31 January each year.

---

## 12. Gap Closure Status

This section records the formal gap closure status that this document contributes to.

| Gap ID | Gap description | Status before this document | Status after this document | Remaining actions to close fully |
|---|---|---|---|---|
| **P-GAP-005** | Sub-processor register not formalised as standalone auditor exhibit — register content existed within `docs/SOC2_READINESS.md §13` but was not a versioned, standalone compliance artifact with DPA/SCC status matrix, addition/removal workflows, and customer disclosure language | 🔴 Gap | 🟡 **Partially closed** — formal register created; DPA dates and Sentry DPA execution required to fully close | (1) Confirm and file executed DPA dates for SP-01 through SP-05 in §7; (2) Execute Sentry DPA (SP-06) and update §7; (3) File register as PRE-SUB-001 in SOC 2 evidence library |
| **PRE-02** | Sub-processor list not published at `security.form.coach/sub-processors`; GDPR Art. 13(1)(e) transparency obligation not met | 🟡 Partial (Sentry DPA pending; list not yet deployed) | 🟡 **Partial** — this document provides the authoritative source content for the published page; page deployment still required | Deploy `security.form.coach/sub-processors` Cloudflare Worker rendering this register in HTML + JSON (see `docs/SOC2_READINESS.md §20.8`); file deployment screenshot as CC9.2 evidence |
| **PRE-08** | DPAs not confirmed with all sub-processors — Sentry DPA pending | 🟡 Partial (Sentry pending) | 🟡 **Partial** — DPA status matrix in §7 is now the authoritative tracking artifact; no new closures from this document alone | Execute Sentry DPA by 2026-07-20; update §7; promote PRE-08 to ✅ Done |
| **PRE-09** | SCCs not in place for all US-EU transfers — Sentry SCCs pending DPA signature | 🟡 Partial | 🟡 **Partial** — SCC status column in §7 is now the authoritative tracking artifact | Execute Sentry SCCs concurrent with DPA execution; update §7; promote PRE-09 to ✅ Done |
| **CC9.2 SOC 2** | Vendor risk management — register defined but programme not yet evidenced through a full annual review cycle | 🟡 Partial (programme documented; first evidence cycle Q1 2027) | 🟡 **Partial** — this document provides the formal register and addition/removal workflows required for CC9.2 evidence; annual review procedure in §11 defines the evidence package | Complete first annual review in January 2027 per §11; file all six evidence steps to `compliance/vendor-review/2027/`; this document version at time of review is the auditor exhibit |

---

## 13. Sign-Off Record

| Role | Name | Date | Signature / Commit |
|---|---|---|---|
| compliance-officer | _[to be signed]_ | _[date]_ | _[git commit hash or DocuSign envelope]_ |
| Founder | _[to be signed]_ | _[date]_ | _[git commit hash]_ |
| Outside counsel (for Tier 1 DPA review) | _[to be signed — required before enterprise DPA countersigning]_ | _[date]_ | _[DocuSign envelope ID]_ |

> **Note:** Outside counsel sign-off is required before this register is referenced in any enterprise DPA or MSA. Estimated scope: 2–3 hours at current counsel rates. Route through the same counsel engaged for privacy policy review (P-GAP-001).

---

*v1.0 · May 2026 · compliance-officer + security-engineer · append-only for DPA history*
