# FORM · Sub-Processor Register & Enterprise DPA Framework · v1.0

> Canonical list of all third-party sub-processors handling personal data on FORM's behalf.
> Also: enterprise Data Processing Agreement (DPA) template, SIEM export DPA addendum, and self-service DPA acceptance protocol.
> Owner: `compliance-officer` + `security-engineer` + `enterprise-architect`. Reviewed quarterly (§8).
> Cross-references: `docs/SOC2_READINESS.md §17` (vendor review), `docs/SOC2_READINESS.md §59` (VRM tier classification), `docs/GDPR_DPIA.md`, `docs/AUDIT_LOG_SCHEMA.md` (DEC-030 chain), `docs/OBSERVABILITY.md §27` (SIEM export / OQ-SIEM-02), `docs/COST_MODEL.md §25.3` (per-deal legal, OQ-35), DEC-030 (HMAC chain audit log).

---

## TL;DR

- **11 active sub-processors** processing personal data on FORM's behalf under GDPR Art. 28
- **5 DPAs pending execution** — VRM-GAP-001 🔴 HIGH, blocking SOC 2 CC9.2 and enterprise onboarding
- **Enterprise DPA template** in §5 — full Art. 28(3)(a)–(h) clauses, SCCs Module 2, TOMs Annex
- **SIEM Export DPA Addendum** in §6 — resolves OQ-SIEM-02 (P1): standard enterprise DPA covers customer-controlled endpoints; addendum required only for third-party-controlled endpoints
- **Self-service DPA acceptance flow** in §7 — targets 60% standard-term adoption, reduces blended per-deal legal from $6,850 → ~$3,200 (COST_MODEL OQ-35)
- Privacy floor: sub-processors never receive `user_id` linked to Art. 9 health categories; pseudonymous identifiers only

---

## 1. GDPR Framework & Dual-Role Context

### 1.1 FORM as Data Controller (B2C)

For consumer users accessing FORM via the iOS/Android app, FORM is the **data controller** — it determines the purposes and means of processing. All third parties that process personal data on FORM's instructions are **sub-processors** under GDPR Art. 28, requiring executed DPAs.

### 1.2 FORM as Data Processor (B2B Enterprise)

For enterprise customers on annual contracts, the enterprise organisation is the **data controller** (its employees are the data subjects) and FORM acts as a **data processor** governed by the enterprise DPA (§5). In this role:

- Sub-processors in §2 are sub-processors *to FORM*, which is itself a processor — GDPR Art. 28.4 applies
- GDPR Art. 28.2 requires FORM to obtain the enterprise customer's prior consent before onboarding new sub-processors that process their employees' data
- FORM's 30-day advance change notification procedure (§8.2) satisfies Art. 28.2

### 1.3 UK GDPR Applicability

This register serves as the Art. 28 processor record under UK GDPR post-Brexit. All Art. 28 references encompass UK GDPR Art. 28 where applicable. Transfer mechanisms for UK data subjects use the UK IDTA or UK Addendum to EU SCCs, as appropriate.

---

## 2. Current Sub-Processor Register

**Publication URL:** `form.coach/legal/sub-processors`
**Last updated:** 2026-06-03
**Next scheduled review:** 2026-09-03 (quarterly)
**Change notification lead-time:** 30 days advance notice to enterprise customers (§8.2)

> This register lists only GDPR Art. 28 sub-processors — third parties that process EU/UK personal data on FORM's behalf. Infrastructure tooling that processes no user personal data (GitHub, 1Password, Better Stack, Expo/EAS) is excluded; those appear in `docs/SOC2_READINESS.md §17` vendor register.

| ID | Sub-Processor | Service | Data Categories (GDPR_DPIA ref) | Processing Location | Transfer Mechanism | DPA Status | SOC 2 / Cert | Tier | Review |
|---|---|---|---|---|---|---|---|---|---|
| SP-01 | **Supabase, Inc.** (DigitalOcean-hosted) | Database, Authentication, File Storage | User account, workout history, health profile, CV metadata, audit log, meal log (PA-01–PA-10) | EU: Frankfurt `de-fra1`; US: `us-east-1` | EU adequacy (Frankfurt); SCCs Module 2 (US tenants) | 🔴 **Pending** (VRM-GAP-001 🔴) | SOC 2 Type I (2024) | T1 Critical | 2027-01 |
| SP-02 | **Cloudflare, Inc.** | CDN, Edge Workers, R2 Object Storage, Access Zero Trust, WAF, Email Workers | Usage metadata (IP, request), cached content, encrypted compliance-vault objects | Global (EU data residency for R2 via jurisdictional restriction — confirm OQ-VRM-03) | SCCs Module 2; Cloudflare GDPR DPA accepted via enterprise ToS | 🟡 **ToS DPA active** | SOC 2 Type II (2025), ISO 27001 | T1 Critical | 2027-01 |
| SP-03 | **Resend, Inc.** | Transactional email delivery | Email address, non-health notification content, DSAR encrypted payload delivery | US (`us-east-1`) | SCCs Module 2 | 🔴 **Pending** (VRM-GAP-001 🔴) | No SOC 2 | T2 Important | 2027-01 |
| SP-04 | **RevenueCat, Inc.** | Subscription lifecycle management | Subscription status, entitlement events, platform receipt (iOS/Android). No card data — payment handled natively by Apple/Google. | US | SCCs Module 2 | 🔴 **Pending** (VRM-GAP-001 🔴) | SOC 2 Type I (2024) | T2 Important | 2027-01 |
| SP-05 | **Apple, Inc.** | Push notification delivery (APNs) | Device token, notification payload (non-health text only — no health values in push content) | US/Global | Apple Developer Program License Agreement — no standalone Art. 28 DPA commercially available (see OQ-VRM-01 in `docs/SOC2_READINESS.md §59.11`; compensating controls documented in §11 item 14) | 🟡 **Apple DPA gap** — compensating control documented | No standalone SOC 2 | T1 Special | 2027-01 |
| SP-06 | **Sentry, Inc.** | Error monitoring, crash reporting | Error context (stack traces, breadcrumbs); PII scrubbed at SDK level via `beforeSend` filter per `docs/OBSERVABILITY.md §28.3` deny-list | US (`us-east-1`) | SCCs Module 2 | 🔴 **Pending** (VRM-GAP-001 🔴) | SOC 2 Type II (2025) | T2 Important | 2027-01 |
| SP-07 | **Anthropic, PBC** | AI inference — Victor coaching (LLM completion API) | Pseudonymised exercise context only — no PII, no health values in prompts (`docs/GDPR_DPIA.md §4` constraint PA-05); coaching response text | US | SCCs Module 2; Anthropic enterprise DPA (prohibits training on customer data) | 🟢 **Enterprise DPA active** | SOC 2 Type II (2025) | T1 Critical | 2027-01 |
| SP-08 | **ElevenLabs, Inc.** | Text-to-speech voice synthesis (Victor voice) | Coaching text cues only — no user PII in payload (PA-06 constraint) | US | SCCs Module 2 | 🔴 **Pending** (OQ-SP-01 — see §12) | No SOC 2 (startup; security questionnaire required, §4.3 of SOC2 §59) | T2 Important | 2027-01 |
| SP-09 | **PostHog, Inc.** (EU Cloud) | Product analytics event stream | Anonymised event stream (no health metrics — `docs/OBSERVABILITY.md §25.7` constraint); pseudonymous `distinct_id` (SHA-256 user UUID) | EU: Frankfurt | EU adequacy (EU Cloud, Frankfurt); PostHog GDPR DPA available | 🔴 **Pending** (P1 M4 per `OBSERVABILITY §25.8`) | SOC 2 Type I (2024) | T2 Important | 2027-01 |
| SP-10 | **Stripe, Inc.** | Subscription billing events (webhook notifications to FORM) | Subscription status, payment event metadata only — no card data (PCI-DSS scope is Stripe-only) | US + EU | SCCs Module 2; Stripe Data Protection Addendum (auto-signed via Business ToS) | 🟢 **Stripe DPA active** | SOC 2 Type I (2025), PCI-DSS Level 1 | T2 Important | 2027-01 |
| SP-11 | **WorkOS, Inc.** | Enterprise SSO (SAML 2.0/OIDC) and SCIM 2.0 directory sync | Enterprise employee identity (email, display name, group membership) for SSO-provisioned tenants only — no consumer user data | US | SCCs Module 2 | 🟡 **Pending verification** (see §11 item 6) | SOC 2 Type II (2024) | T1 Critical (enterprise tier only) | 2027-01 |

**DPA status summary:**

| Status | Count | Sub-Processors |
|---|---|---|
| 🟢 Active | 2 | SP-07 Anthropic, SP-10 Stripe |
| 🟡 Partial / ToS-accepted | 2 | SP-02 Cloudflare, SP-05 Apple (compensating control) |
| 🟡 Pending verification | 1 | SP-11 WorkOS |
| 🔴 Pending execution | 6 | SP-01 Supabase, SP-03 Resend, SP-04 RevenueCat, SP-06 Sentry, SP-08 ElevenLabs, SP-09 PostHog |

**VRM-GAP-001 🔴 HIGH (from `docs/SOC2_READINESS.md §59`):** DPAs for SP-01 Supabase, SP-03 Resend, SP-04 RevenueCat, and SP-06 Sentry not yet executed. Blocking for SOC 2 CC9.2 and all enterprise onboarding. Target: M6. Checklist items in §11 rows 1–4 close this gap; evidence artefacts VRM-E-001 through VRM-E-004.

---

## 3. Sub-Processor Tier Classification

Consistent with the four-tier framework in `docs/SOC2_READINESS.md §59.2`:

| Tier | Definition | Annual Review Requirements | DPA Terms Required | Active Sub-Processors |
|---|---|---|---|---|
| **T1 Critical** | SOC 2 in-scope processing; Art. 9 data or primary data store; outage = platform failure | Full security questionnaire + SOC 2 report review + right-to-audit clause | Full Art. 28(3)(a)–(h) + SCCs Module 2 + breach notification ≤ 24h to FORM | SP-01 Supabase, SP-02 Cloudflare, SP-05 Apple, SP-07 Anthropic, SP-11 WorkOS |
| **T2 Important** | Operational dependency; processes personal data but not Art. 9 health data directly | SOC 2 bridge letter or 8-item security questionnaire (SOC2 §59.4) | Standard Art. 28(3) clauses + SCCs Module 2 | SP-03 Resend, SP-04 RevenueCat, SP-06 Sentry, SP-08 ElevenLabs, SP-09 PostHog, SP-10 Stripe |
| **T3 Standard** | Limited personal data exposure (IP addresses, metadata only) | Annual compliance-officer review | Standard DPA via ToS acceptable | (none currently) |
| **T4 Peripheral** | No personal data processed; infrastructure or tooling only | Biennial review | No DPA required | GitHub, 1Password, Better Stack, Expo/EAS |

**Apple special case (T1 Special):** Meets T1 criticality criteria (APNs processes device tokens for all users) but no standalone GDPR Art. 28 DPA is commercially available from Apple. Compensating controls: (a) no personal data or health identifiers in push payload — device token + non-health text only; (b) processing scope governed by Apple Developer Program License Agreement; (c) formal risk acceptance documented in `compliance/evidence/vendors/apple/no-dpa-compensating-control-{date}.md` (§11 item 14). Cross-reference: SOC2 §59.11 OQ-VRM-01.

---

## 4. Transfer Mechanism Summary

FORM processes data from EU/EEA and UK users. For US-based sub-processors:

| Transfer Mechanism | Basis | Applies To |
|---|---|---|
| **EU Adequacy** | GDPR Art. 45 — data does not leave EU/EEA | SP-01 Supabase EU (`de-fra1`), SP-09 PostHog EU (Frankfurt) |
| **SCCs Module 2** (controller-to-processor) | EU Commission Decision 2021/914 | SP-03 Resend, SP-04 RevenueCat, SP-06 Sentry, SP-07 Anthropic, SP-08 ElevenLabs, SP-10 Stripe, SP-11 WorkOS |
| **Cloudflare DPA** | Cloudflare GDPR DPA + Data Privacy Framework + SCCs Module 2 | SP-02 Cloudflare |
| **UK IDTA / UK Addendum** | UK ICO-approved IDTA or UK Addendum to EU SCCs | UK data subjects to all US sub-processors listed above |
| **Apple Developer Agreement** | No Art. 28 DPA — compensating control documented | SP-05 Apple |

**Transfer Impact Assessments (TIAs):** Required under SCCs Module 2 for T1 US processors. Priority TIAs: SP-07 Anthropic (T1, health-adjacent coaching context) and SP-11 WorkOS (T1, enterprise employee identity). See §11 items 11.

---

## 5. Enterprise Data Processing Agreement Template

FORM's standard enterprise DPA template. When enterprise procurement requests a DPA, FORM first offers self-service acceptance (§7). Outside counsel is engaged only when a customer cannot accept the standard template.

**Template ID:** DPA-FORM-001 v1.0 (2026-06-03)
**Legal review status:** Drafted by compliance-officer — external counsel sign-off required before first enterprise signature (§11 item 13)
**Applicable to:** Enterprise customers ≥ 50 employees on annual contracts

### 5.1 Standard DPA Clauses — GDPR Art. 28(3)(a)–(h)

**Art. 28(3)(a) — Process only on documented instructions:**

> FORM shall process Customer Personal Data only on documented instructions from Customer, including with regard to international transfers, unless required to do so by Union or Member State law. In such a case, FORM shall inform Customer before processing, unless that law prohibits such information on public-interest grounds.

**Art. 28(3)(b) — Confidentiality of authorised personnel:**

> FORM shall ensure that persons authorised to process Customer Personal Data have committed themselves to confidentiality or are under an appropriate statutory obligation of confidentiality.

**Art. 28(3)(c) — Security measures (Art. 32):**

> FORM shall implement the technical and organisational measures specified in Annex II of this DPA. These measures ensure a level of security appropriate to the risk, including: (i) AES-256-GCM encryption at rest and TLS 1.2+ in transit; (ii) HMAC-chained immutable audit log for all privileged data operations (DEC-030 standard); (iii) role-based access control with Just-in-Time privilege escalation and dual-person authorisation for destructive operations (`docs/SSO_SCIM_IMPLEMENTATION.md §24`); (iv) SOC 2 Type II certification (report available to Customer under NDA upon request).

**Art. 28(3)(d) — Sub-processor restrictions:**

> FORM shall not engage another processor ("sub-processor") without Customer's prior written authorisation. Customer grants FORM general authorisation to engage the sub-processors listed at `form.coach/legal/sub-processors` as of the DPA effective date. FORM shall notify Customer of any intended addition or replacement of sub-processors by updating `form.coach/legal/sub-processors` and delivering in-app notification to Customer's admin users at least thirty (30) days before the effective date. Customer may object in writing within thirty (30) days; absent an objection, Customer is deemed to have accepted the change. FORM shall impose data protection obligations on sub-processors equivalent to those in this DPA.

**Art. 28(3)(e) — Assist with data subject rights:**

> Taking into account the nature of the processing, FORM shall assist Customer in fulfilling data subject rights requests (access, erasure, portability, objection) via the `/enterprise/v1/dsar` API endpoint documented in `docs/API.md §DSAR`.

**Art. 28(3)(f) — Assist with Art. 32–36 obligations:**

> FORM shall assist Customer with security obligations (Art. 32), breach notification (Art. 33/34), DPIAs (Art. 35), and prior consultation (Art. 36), taking into account the nature of processing and information available to FORM. FORM's breach notification SLA is eight (8) hours from FORM's awareness of a breach affecting Customer's employees (Template P2C-01 in `docs/INCIDENT_RESPONSE.md §15.8`).

**Art. 28(3)(g) — Deletion or return upon termination:**

> At Customer's choice, FORM shall delete or return all Customer Personal Data after the end of services, and delete existing copies unless required by law. Customer must submit a deletion or export request via the Admin Dashboard within ninety (90) days of contract termination. After ninety (90) days, FORM will delete all Customer Personal Data from production systems. Audit log entries subject to HMAC chain retention requirements (DEC-030, seven-year retention) will be retained in an access-controlled compliance vault for the legally required period, then irreversibly destroyed.

**Art. 28(3)(h) — Audit cooperation:**

> FORM shall make available to Customer all information necessary to demonstrate compliance with this Article and allow for audits, including inspections, by Customer or a mandated auditor. FORM's primary cooperation mechanism is the annual SOC 2 Type II report (available under NDA). On-site audits are available for Enterprise+ customers with ≥ 30 days' advance written notice, subject to scheduling constraints and confidentiality obligations.

### 5.2 DPA Annex I — Description of Processing

| Field | Value |
|---|---|
| Controller | Customer (enterprise organisation) |
| Processor | FORM (as identified in the Order Form) |
| Data subjects | Customer's employees and contractors who use the FORM application |
| Categories of personal data | Identity data (name, email — received via SSO/SCIM); fitness engagement data (workout session completion, program adherence, aggregated form scores); device data (device type, OS version — crash telemetry only, no biometrics sent to employer) |
| Special categories (Art. 9) | **None at employer level.** Health-adjacent data (individual workout metrics, CV session data, RPE) is accessible only to the individual user. Employer Admin Dashboard shows only k-anonymised aggregate metrics (n ≥ 5). FORM's privacy floor prohibits any export enabling the employer to identify individual health information. |
| Purpose of processing | Providing AI fitness coaching to Customer's employees; generating aggregate wellness engagement reports for HR/People teams |
| Duration | Contract term plus 90-day deletion window post-termination |
| Customer's legal basis | Art. 6(1)(b) performance of employment contract, or Art. 6(1)(a) consent — Customer's responsibility to establish and document per applicable member state law |

### 5.3 DPA Annex II — Technical and Organisational Measures (TOMs)

| Control Category | FORM Implementation | Evidence Reference |
|---|---|---|
| Encryption at rest | AES-256-GCM for health-adjacent columns; database-level encryption via Supabase | `docs/SOC2_READINESS.md §56` key management exhibit |
| Encryption in transit | TLS 1.2+ enforced on all endpoints; Cloudflare WAF enforces HTTPS-only | SOC 2 §26 CC6.7 evidence |
| Access control | RBAC (consumer / enterprise_admin / form_api / form_system / form_admin roles); JIT privilege escalation for `form_admin` (4h–15min windows by tier, dual-person auth for destructive); TOTP-gated admin surfaces | SOC 2 §43 credential hardening; `docs/SSO_SCIM_IMPLEMENTATION.md §24` PAM |
| Audit logging | HMAC-chained immutable audit log (DEC-030) for all privileged operations; 7-year retention; append-only with tamper-evident chain | `docs/AUDIT_LOG_SCHEMA.md`; SOC 2 §46 alert pipeline |
| Sub-processor controls | T1/T2 sub-processors hold Art. 28 DPAs; annual security review (SOC 2 §59.5); breach notification ≤ 24h to FORM | This document §2; SOC 2 §59 VRM |
| Vulnerability management | Monthly dependency scanning (Dependabot + Snyk); critical CVE patch ≤ 24h; annual external penetration test | SOC 2 §41 vulnerability management; §52 pen testing |
| Incident response | P0–P3 classification; 72h GDPR Art. 33 SLA; 8h B2B processor-to-controller notification (Template P2C-01) | `docs/INCIDENT_RESPONSE.md §15` |
| Data minimisation | No PII in LLM prompts (GDPR_DPIA §4); CV analysis on-device (no biometrics to cloud); PostHog event stream anonymised | `docs/OBSERVABILITY.md §25.7`; `docs/GDPR_DPIA.md §4` |
| Personnel security | Confidentiality agreements for all team members; annual security awareness training | SOC 2 §42 personnel security; SOC 2 §55 |
| Physical security | Cloud-native, no on-premises infrastructure; Supabase and Cloudflare data centre physical security inherited from their SOC 2 scopes | Supabase SOC 2 report; Cloudflare SOC 2 report |

### 5.4 DPA Annex III — Sub-Processor List

The current sub-processor list is maintained at `form.coach/legal/sub-processors` (§2 of this document). Annex III of any executed DPA incorporates the list as of the signing date. Post-execution changes are governed by Art. 28(3)(d) and §8.

---

## 6. SIEM Export DPA Addendum (Enterprise+)

*Resolves `docs/OBSERVABILITY.md §27.12 OQ-SIEM-02` (P1): whether the standard enterprise DPA covers tenant-specified SIEM endpoints as downstream controllers or processors, and whether separate consent is required.*

### 6.1 Legal Classification

When an Enterprise+ customer enables SIEM export via Admin Dashboard (`POST /enterprise/v1/siem-export/enable`), FORM transmits a redacted audit event stream to a SIEM endpoint specified by the customer. Legal classification:

| Scenario | Classification | DPA Requirement |
|---|---|---|
| Customer's SIEM endpoint is **customer-operated** (self-hosted Splunk, on-prem Elasticsearch, private Sentinel workspace) | Customer's own infrastructure — FORM acts on Art. 28(3)(a) documented instruction | **No addendum required** — covered by standard enterprise DPA |
| Customer's SIEM endpoint is a **customer-contracted SaaS** (Splunk Cloud, Datadog Cloud, Microsoft Sentinel SaaS under customer's subscription) | FORM transmits to customer's SaaS workspace — still acting on customer's instructions | **No addendum required** — customer maintains their own DPA with SIEM vendor |
| SIEM endpoint is **not under customer's direct control** (third-party MSSP, shared SOC endpoint) | New sub-processor chain — FORM would become a processor with an additional processor | **Addendum required** before export can be enabled |

**Resolution of OQ-SIEM-02:** The standard enterprise DPA Art. 28(3)(a) (process only on documented instructions) covers Scenarios 1 and 2, which represent the overwhelming majority of enterprise SIEM deployments. The Admin Dashboard "enable SIEM export" UI action constitutes sufficient documented instruction for these scenarios. The addendum in §6.2 is required only for Scenario 3 (MSSP or third-party SOC endpoint).

### 6.2 SIEM Export DPA Addendum Template

*Required only for Scenario 3 (§6.1). Template ID: DPA-FORM-SIEM-ADDENDUM-001 v1.0.*

> **SIEM Export Addendum to FORM Data Processing Agreement**
>
> This Addendum supplements the Data Processing Agreement between \[Customer\] and FORM. Capitalised terms have the meanings given in the DPA.
>
> **1. Scope.** Customer authorises FORM to transmit a redacted subset of Customer's HMAC-chained audit event stream ("SIEM Export Data") to the endpoint specified in Customer's Admin Dashboard SIEM configuration ("SIEM Endpoint").
>
> **2. Privacy Floor.** FORM shall apply the data redaction rules in `docs/OBSERVABILITY.md §27.5` to all SIEM Export Data regardless of Customer configuration. SIEM Export Data shall not contain individual `user_id`, untruncated IP addresses, or any Art. 9 data categories.
>
> **3. Third-Party SIEM Operator.** If the SIEM Endpoint is operated by a third party not under Customer's direct control ("SIEM Operator"), Customer warrants that Customer holds a valid DPA or processing agreement with the SIEM Operator covering receipt of SIEM Export Data, and that such processing is lawful. Customer shall provide FORM with written confirmation of this DPA within fourteen (14) days of FORM's reasonable request.
>
> **4. FORM's Role.** FORM's transmission of SIEM Export Data is an act of processing on Customer's documented instruction. FORM is not a party to any agreement between Customer and the SIEM Operator and accepts no liability for the SIEM Operator's processing of SIEM Export Data.
>
> **5. Suspension.** FORM may suspend SIEM export immediately if the SIEM Endpoint transmits SIEM Export Data outside the scope of this Addendum, or if Customer fails to provide DPA confirmation under §3 within fourteen (14) days.

### 6.3 Known Cloud SIEM SaaS — No Addendum Required

FORM maintains an allow-list of SIEM SaaS endpoints clearly operated by the customer under their own enterprise agreements. The compliance gate in Admin Dashboard auto-clears these domains (§11 item 12):

| SIEM Product | Endpoint Pattern |
|---|---|
| Splunk Cloud | `*.splunkcloud.com` |
| Microsoft Sentinel | `*.ods.opinsights.azure.com` |
| Datadog | `*.datadoghq.com`, `*.datadoghq.eu` |
| IBM QRadar SaaS | `*.qradar.ibmcloud.com` |
| Elastic Cloud | `*.elastic.co` |
| Sumo Logic | `*.sumologic.com` |
| Panther | `*.runpanther.io` |

For any endpoint not on this list, the compliance-officer must review and classify before SIEM export can be enabled. The review gate emits `admin.siem_export_enabled` with `compliance_officer_reviewed: true`.

---

## 7. Self-Service DPA Acceptance Flow

*Addresses `docs/COST_MODEL.md §25.10 OQ-35`: can standard DPA acceptance reduce per-deal legal overhead from $6,850 to ~$3,200?*

### 7.1 Design Rationale

The $6,850 per-deal legal overhead (COST_MODEL §25.3.6 base) includes ~$4,500 for MSA redlines and DPA review. Customers who accept FORM's standard DPA without redlines reduce that cost to near-zero on the DPA portion; only MSA negotiation remains (~$2,350). At 60% standard-term adoption, the blended per-deal legal cost drops to approximately $3,200 — the COST_MODEL OQ-35 target scenario.

**Qualification by deal size:**

| Deal Size | Default Path | Notes |
|---|---|---|
| < 500 seats | Self-service DPA acceptance (no outside counsel) | Standard-terms expected |
| 500–2,000 seats | Self-service + compliance-officer DPA review at close | Outside counsel only if customer requests redlines |
| > 2,000 seats | Full MSA + DPA negotiation mandatory | Compliance-officer + outside counsel engaged from term-sheet stage |

### 7.2 Acceptance Flow Architecture

```
1. Enterprise admin navigates to Admin Dashboard → Legal → Data Processing Agreement

2. FORM presents DPA-FORM-001 v1.0 (§5) with sub-processor Annex (§2 snapshot)

3. Admin clicks "Accept DPA"

4. System records:
   - accepting_user_id (enterprise admin UUID, verified via Cloudflare Access session)
   - acceptance_timestamp_utc
   - dpa_version ("DPA-FORM-001-v1.0")
   - sub_processor_list_snapshot_hash (SHA-256 of form.coach/legal/sub-processors at acceptance time)
   - accepting_ip (browser IP — stored in audit event only, not in DPA PDF)

5. DEC-030 HMAC-chained event emitted: admin.dpa_accepted (§9)

6. PDF receipt generated with acceptance metadata embedded; stored in:
   - compliance/legal/enterprise-dpas/{tenant_id}/dpa-{version}-{date}.pdf  (FORM internal R2)
   - Admin Dashboard → Legal → DPA History  (customer-accessible, 7-year retention)

7. Confirmation email sent to accepting admin + Customer DPA contact
   (if specified in org settings → Admin Dashboard → Settings → Legal Contact)
```

### 7.3 DPA Versioning and Re-Acceptance

| Change Type | Notice Period | Re-Acceptance Required |
|---|---|---|
| Sub-processor-only changes (§8 change notification) | 30 days | No — right to object within 30 days; silence = acceptance |
| Material DPA template changes (new data categories, new legal mechanisms) | 30 days | Yes — required by contract anniversary date |
| Regulatory-driven changes (new SCCs issued by EU Commission) | 60 days | Yes — required within 60 days |

### 7.4 Customer Legal Review Gate

Customers who cannot accept the standard DPA:
1. Email `legal@form.coach` with subject "DPA Redline Request — [Company Name]"
2. Attach their internal DPA template or a marked-up version of FORM's template
3. Allow 10 business days for compliance-officer review; outside counsel engaged only if redlines affect FORM's core security architecture or sub-processor list

**OQ-35 tracking:** `admin.dpa_accepted` vs. `admin.dpa_redline_requested` event counts per quarter (with `deal_size_seats` covariate) provide the data to close OQ-35 after the 3rd enterprise deal. Target: ≥ 60% `dpa_accepted` rate.

---

## 8. Sub-Processor Change Management

### 8.1 Change Types and Notice Requirements

| Change Type | Lead-Time | Notification Channel | Customer Action |
|---|---|---|---|
| Adding a new T1/T2 sub-processor | 30 days advance | In-app notification (all enterprise admin users) + email to DPA contact | Right to object in writing within 30 days; silence = acceptance |
| Replacing a T1/T2 sub-processor (functionally equivalent) | 30 days advance | Same as above | Same as above |
| Removing a sub-processor | 7 days | In-app notification | Informational; no action required |
| Emergency replacement (vendor breach or termination) | Best effort; ≥ 48h | Emergency email + in-app + status page notice | Informational; customer may request written explanation |
| T3/T4 changes | No notice required | Annual sub-processor list update | N/A |

### 8.2 Change Notification Template

```
Subject: [FORM] Sub-Processor Change Notice — Effective [DATE]

Dear [Customer Name] Team,

FORM is notifying you of an upcoming sub-processor change per your Data Processing Agreement
(Art. 28(3)(d)) with thirty (30) days' advance notice.

Effective Date: [DATE — 30 days from this notice]
Change Type:   [Addition / Replacement / Removal]
Sub-Processor: [Name]
Service:       [Description]
Data Categories Affected: [List]
Processing Location: [Region]
Transfer Mechanism: [SCCs Module 2 / EU adequacy]
Reason for Change: [Brief description]

If you object to this change, notify us in writing at legal@form.coach within 30 days.
If no objection is received, you are deemed to have accepted this change.

Updated sub-processor list: form.coach/legal/sub-processors

FORM Compliance Team · compliance@form.coach
```

### 8.3 Internal Onboarding Approval

Before any new T1/T2 sub-processor receives FORM user data:

1. Complete T1/T2 due diligence questionnaire (`docs/SOC2_READINESS.md §59.4`)
2. Execute DPA; store signed copy in `compliance/evidence/vendors/{slug}/dpa-{date}.pdf`
3. Assess GDPR Art. 28 sub-processor impact (new transfer mechanism? new data category? TIA required?)
4. Compliance-officer formal approval (signed `compliance/evidence/vendors/{slug}/onboarding-approval-{date}.md`)
5. Emit `admin.sub_processor_added` DEC-030 event (§9)
6. Update `form.coach/legal/sub-processors` via Sub-Processor List Worker (`docs/SOC2_READINESS.md §44`)
7. Send 30-day advance notice to all enterprise customers with active DPAs (§8.2 template)

### 8.4 Sub-Processor Offboarding

1. Issue formal data deletion request per DPA Art. 28(3)(g) terms within 5 business days of contract end
2. Confirm deletion within vendor's stated SLA (typically 30–90 days); request written confirmation
3. Store deletion confirmation in `compliance/evidence/vendors/{slug}/deletion-confirmation-{date}.md`
4. Emit `admin.sub_processor_removed` DEC-030 event (§9)
5. Update `form.coach/legal/sub-processors`; notify enterprise customers per §8.1

---

## 9. DEC-030 HMAC-Chained Audit Events

All sub-processor lifecycle and DPA events are HMAC-chained per `docs/AUDIT_LOG_SCHEMA.md`. 7-year retention for all events in this section (GDPR Art. 5(2) accountability obligation + SOC 2 CC9.2 evidence).

| Event Type | Severity | Retention | Key Metadata Fields | Trigger |
|---|---|---|---|---|
| `admin.dpa_accepted` | HIGH | 7 years | `tenant_id`, `accepting_user_id`, `dpa_version`, `sub_processor_list_snapshot_hash`, `acceptance_ip` (stored in event only, not PDF), `deal_size_seats` | Enterprise admin accepts DPA via self-service flow (§7) |
| `admin.dpa_redline_requested` | MEDIUM | 7 years | `tenant_id`, `requesting_user_id`, `dpa_version`, `outside_counsel_engaged` (bool) | Customer requests DPA redlines via `legal@form.coach` |
| `admin.dpa_executed` | CRITICAL | 7 years | `tenant_id`, `dpa_version`, `execution_date`, `customer_signatory_email_hash` (SHA-256), `form_signatory`, `legal_ref`, `per_deal_legal_cost_usd`, `outside_counsel_engaged` (bool) | DPA fully executed (both parties signed) |
| `admin.sub_processor_added` | HIGH | 7 years | `sub_processor_id` (internal ref SP-NN), `sub_processor_name`, `service_description`, `data_categories[]`, `processing_location`, `transfer_mechanism`, `dpa_executed_date`, `tier`, `operator` | New T1/T2 sub-processor onboarded; 30-day notice sent |
| `admin.sub_processor_removed` | HIGH | 7 years | `sub_processor_id`, `sub_processor_name`, `removal_reason`, `deletion_request_date`, `deletion_confirmed_date`, `operator` | Sub-processor offboarded |
| `admin.sub_processor_change_notice_sent` | STANDARD | 7 years | `change_type` (added/replaced/removed/emergency), `sub_processor_name`, `notice_date`, `effective_date`, `tenant_ids_notified_count` (integer — not array, to avoid PII), `objection_deadline` | 30-day advance notice dispatched to enterprise customers |
| `admin.sub_processor_change_objection_received` | HIGH | 7 years | `tenant_id`, `sub_processor_name`, `objection_received_date`, `resolution` (change_cancelled / tenant_offboarded / exception_granted), `compliance_officer_reviewed` (bool) | Enterprise customer objects to sub-processor change |
| `admin.siem_export_enabled` | HIGH | 7 years | `tenant_id`, `siem_vendor`, `endpoint_domain_hash` (SHA-256 of domain — not full URL), `addendum_required` (bool), `addendum_signed` (bool), `enabled_by_user_id_hash` (SHA-256), `compliance_officer_reviewed` (bool), `allowlist_match` (bool) | Enterprise admin enables SIEM export in Admin Dashboard |
| `admin.dpa_re_acceptance_required` | STANDARD | 7 years | `tenant_id`, `dpa_old_version`, `dpa_new_version`, `change_reason`, `re_acceptance_deadline`, `notified_admin_count` (integer) | FORM issues material DPA template update requiring re-acceptance |

**Privacy invariant:** No personal data of individual employees (name, email address, workout data) appears in any of the above events. All `user_id` references are pseudonymous internal UUIDs. Customer signatory email is stored as SHA-256 hash in audit events only; plaintext version stored solely in the executed DPA PDF in the compliance vault. `tenant_ids_notified` is stored as a count, not an array, to avoid encoding customer enumeration data in the audit chain.

---

## 10. SOC 2 Mapping

| SOC 2 Criterion | Control | Evidence from This Document |
|---|---|---|
| **CC9.2 — Sub-processor risk management** | Sub-processor register (§2) with 4-tier classification (§3); DPA template (§5) imposes equivalent security obligations; annual review cadence; 30-day change notification with objection right (§8) | `admin.sub_processor_added` / `admin.sub_processor_removed` DEC-030 events; executed DPA PDFs in compliance vault; `form.coach/legal/sub-processors` live page; VRM-E-001 through VRM-E-010 artefacts |
| **CC9.2 — Vendor due diligence** | T1/T2 sub-processors reviewed annually with SOC 2 report or 8-item questionnaire (SOC2 §59.4 + §59.5) | Annual reassessment records in `compliance/evidence/vendors/{slug}/review-{year}.md`; SOC 2 bridge letters |
| **C1.1 — Confidentiality commitments** | DPA Art. 28(3)(b) requires sub-processor confidentiality; DPA Annex II TOMs (§5.3) specifies data minimisation and pseudonymisation controls; sub-processors never receive raw Art. 9 health data | `admin.dpa_executed` DEC-030 events; DPA executed PDFs; GDPR_DPIA §4 privacy constraints |
| **P3.2 — Sub-processor disclosure** | Sub-processor list published at `form.coach/legal/sub-processors`; changes notified ≥ 30 days in advance per DPA Art. 28(3)(d) | `admin.sub_processor_change_notice_sent` DEC-030 events; VRM-E-010 (publication confirmation) |
| **P6.1 — Processor disclosure to controller** | Enterprise DPA (§5) makes full sub-processor disclosure via Annex III and change notification; self-service acceptance records the snapshot hash at acceptance time | `admin.dpa_accepted` events with `sub_processor_list_snapshot_hash`; `admin.dpa_executed` events |
| **CC2.3 — External communication** | DPA template and sub-processor list are public-facing GDPR Art. 14 transparency disclosures | `form.coach/legal/sub-processors` live page; SOC 2 CC9-GAP-007 closure (Sub-Processor List Worker per SOC2 §44) |

**Evidence artefacts:**

| Artefact ID | Description | Location |
|---|---|---|
| SP-E-001 | Published sub-processor list page confirmation screenshot + URL verification | `compliance/evidence/cc9/sp-e-001-subprocessor-page.png` |
| SP-E-002 | Executed DPA PDFs for SP-01 through SP-11 (VRM-E-001 through VRM-E-004 cross-reference) | `compliance/evidence/vendors/{slug}/dpa-{date}.pdf` |
| SP-E-003 | `admin.dpa_executed` DEC-030 event exports (one per enterprise customer) | `compliance/evidence/cc9/sp-e-003-dpa-executions-{year}.jsonl` |
| SP-E-004 | Annual sub-processor security review records (SOC 2 reports or questionnaire responses) | `compliance/evidence/vendors/{slug}/review-{year}.md` |
| SP-E-005 | Self-service DPA acceptance rate report (OQ-35 tracking) | `compliance/evidence/cc9/sp-e-005-dpa-acceptance-rate-{quarter}.csv` |

---

## 11. Implementation Checklist

| # | Task | Owner | Priority | Milestone | Definition of Done |
|---|---|---|---|---|---|
| 1 | Execute Supabase DPA (SP-01). Store in `compliance/evidence/vendors/supabase/dpa-{date}.pdf`. File VRM-E-001. Emit `admin.sub_processor_added` DEC-030 event. Cross-reference: SOC2 §59.10 item 1. | compliance-officer | **P0** | M6 | DPA signed by both parties; PDF in compliance vault; DEC-030 event confirmed in HMAC chain |
| 2 | Execute Resend DPA (SP-03). Store in `compliance/evidence/vendors/resend/dpa-{date}.pdf`. File VRM-E-002. Emit DEC-030 event. SOC2 §59.10 item 2. | compliance-officer | **P0** | M6 | Same DoD as item 1 |
| 3 | Execute RevenueCat DPA (SP-04). Store in `compliance/evidence/vendors/revenuecat/dpa-{date}.pdf`. File VRM-E-003. Emit DEC-030 event. SOC2 §59.10 item 3. | compliance-officer | **P0** | M6 | Same DoD as item 1 |
| 4 | Execute Sentry DPA (SP-06) + confirm `beforeSend` PII scrubbing in staging (OBSERVABILITY §28.3 deny-list CI test). Store in `compliance/evidence/vendors/sentry/dpa-{date}.pdf`. File VRM-E-004. | security-engineer + compliance-officer | **P0** | M6 | DPA signed; `beforeSend` deny-list CI test green; PDF in compliance vault |
| 5 | Execute PostHog EU DPA (SP-09). Store in `compliance/evidence/vendors/posthog/dpa-{date}.pdf`. Emit DEC-030 event. Closes OBSERVABILITY §25.8 P1 checklist item. | compliance-officer | **P1** | M4 | DPA executed; PostHog entry on `form.coach/legal/sub-processors` updated |
| 6 | Verify WorkOS DPA (SP-11) scope covers FORM's enterprise SSO/SCIM use case. Confirm SCCs Module 2 for EU enterprise tenants. Store in `compliance/evidence/vendors/workos/dpa-{date}.pdf`. | compliance-officer + enterprise-architect | **P1** | M4 | WorkOS DPA scope confirmed; SCCs Module 2 confirmed for EU transfers; PDF filed |
| 7 | Execute ElevenLabs DPA (SP-08) with SCCs Module 2. If ElevenLabs cannot provide a standard Art. 28 DPA, document compensating controls (no user PII in TTS payload) and file risk acceptance memo signed by compliance-officer. Escalate OQ-SP-01 to founder if no resolution by M5 -2 weeks. | compliance-officer | **P1** | M5 | DPA executed OR risk acceptance memo filed with compliance-officer sign-off |
| 8 | Register all 9 DEC-030 events from §9 in `docs/AUDIT_LOG_SCHEMA.md`. Implement Zod schema for `admin.dpa_accepted` and `admin.sub_processor_added`. Validate HMAC chain includes new event types in staging. | security-engineer + platform-engineer | **P1** | M6 | All 9 events registered; Zod schemas passing; staging chain validation green |
| 9 | Deploy self-service DPA acceptance flow (§7.2) in Admin Dashboard → Legal. Wire `admin.dpa_accepted` DEC-030 event. Generate PDF receipt; store in compliance vault and Admin Dashboard DPA History. | platform-engineer + design-craft | **P1** | M8 | DPA acceptance UI live; PDF generation working; DEC-030 event in HMAC chain; 7-year retention confirmed |
| 10 | Publish `form.coach/legal/sub-processors` page via Sub-Processor List Worker (SOC2 §44). Confirm all SP-01 through SP-11 entries render accurately. File VRM-E-010. Closes CC9-GAP-007 and CC2-GAP-001 in SOC2 §51 consolidated gap register. | platform-engineer | **P1** | M7 | Page live; all entries accurate; VRM-E-010 filed; SOC2 §51 gap register updated |
| 11 | Complete Transfer Impact Assessments (TIAs) for SP-07 Anthropic and SP-11 WorkOS (T1 US processors handling EU user data under SCCs Module 2). File in `compliance/evidence/vendors/{slug}/tia-{date}.pdf`. | compliance-officer + external counsel | **P1** | M7 | TIAs completed, signed, and filed; SCCs Module 2 attached to each executed DPA |
| 12 | Implement SIEM export compliance gate in Admin Dashboard (§6.3 allow-list): before enabling export, check endpoint domain against allow-list; if no match, set `compliance_officer_reviewed: false` and block export pending review flag. Emit `admin.siem_export_enabled` DEC-030 event on enable. Closes OQ-SIEM-02 in OBSERVABILITY §27.12. | platform-engineer + compliance-officer | **P1** | M5 | SIEM export compliance gate live; allow-list hardcoded in Worker; DEC-030 event confirmed |
| 13 | Engage external EU-qualified privacy counsel to review DPA-FORM-001 v1.0 template (§5). Increment to v1.1-counsel-reviewed before first enterprise customer signature. File counsel sign-off memo in `compliance/legal/dpa-counsel-review-{date}.md`. | compliance-officer + external counsel | **P0** | M5 (before first enterprise pilot) | Counsel sign-off memo filed; DPA version incremented; template updated in Admin Dashboard |
| 14 | Document Apple compensating control (SP-05): no standalone DPA available. File `compliance/evidence/vendors/apple/no-dpa-compensating-control-{date}.md` with: (a) payload minimisation confirmation (no PII/health in push content), (b) Apple Developer Agreement scope, (c) compliance-officer risk acceptance signature. Close OQ-VRM-01 for this document. File VRM-E-011. | compliance-officer | **P2** | M7 | Compensating control document filed; risk acceptance signed; VRM-E-011 filed |

---

## 12. Open Questions

| OQ | Status | Owner | Priority | Target |
|---|---|---|---|---|
| **OQ-SP-01** | 🟡 **Open.** ElevenLabs DPA gap — ElevenLabs may not have a standard GDPR Art. 28 DPA template available. Compensating controls if DPA unavailable: (a) no user PII or health data in TTS payload (coaching text cues only per PA-06 constraint in GDPR_DPIA §4); (b) ElevenLabs payload is generated by FORM Workers before calling the TTS API (user data never directly transmitted to ElevenLabs); (c) voice synthesis is opt-in only. If no DPA is available by M5, compliance-officer must sign a formal risk acceptance memo. | compliance-officer | **P1** | M5 |
| **OQ-SP-02** | 🟡 **Open.** Expo/EAS sub-processor classification — Currently T4 (no personal data processed). However, EAS builds the FORM app binary distributed to all users; a compromised EAS build pipeline could affect all user data. CC9-GAP-003 (SOC2 §28) tracks the Expo DPA negotiation. Question: should EAS be re-classified T4 → T2 based on supply chain risk alone, even if no user data flows through EAS at runtime? If T2, an Art. 28 DPA and annual review are required. Decision needed before enterprise GA. | security-engineer + compliance-officer | **P2** | M10 |
| **OQ-SP-03** | 🟡 **Open.** New AI model provider process — If FORM integrates additional LLM providers beyond Anthropic (SP-07), each new provider must complete T1 due diligence and execute a DPA before receiving any FORM user data. Product team must notify compliance-officer ≥ 60 days before any new AI model integration to allow DPA negotiation. This should be added as a standing gate in the feature launch checklist (`docs/LAUNCH_CHECKLIST.md`). | compliance-officer + product-manager | **P2** | Standing process |
| **OQ-SP-04** | 🟡 **Open.** Self-service DPA adoption rate (COST_MODEL OQ-35 closure) — The 60% standard-term adoption target can only be validated empirically. Data: `admin.dpa_accepted` vs. `admin.dpa_redline_requested` event counts per quarter with `deal_size_seats` as covariate. OQ-35 in `docs/COST_MODEL.md §25.10` closes when ≥ 3 enterprise deals have been observed; if adoption rate ≥ 60%, record blended per-deal legal ≈ $3,200 as confirmed. If < 40%, DPA template or UI requires simplification review. | compliance-officer + founder | **P1** | After 3rd enterprise deal |

---

*v1.0 (2026-06-03): Initial release. Closes the dead-reference gap across 6+ cross-document references to `docs/SUBPROCESSORS.md` that previously had no target. Addresses: OBSERVABILITY.md §25.8 P1 (PostHog DPA checklist item — §11 item 5); OBSERVABILITY.md §24.13 P2 (RevenueCat and Stripe DPA references — §2 SP-04/SP-10); OBSERVABILITY.md §27.11 P1 (SIEM export DPA addendum — §6, closes OQ-SIEM-02 with 3-scenario legal classification table: no addendum required for Scenarios 1/2, addendum required for Scenario 3 only); DATA_MODEL.md §coaching_turns Anthropic sub-processor reference — §2 SP-07 and §5.3 TOMs Annex; SOC2_READINESS.md §44 Sub-Processor List Worker publication URL — §10 SP-E-001 evidence artefact. 11 active sub-processors catalogued: SP-01 Supabase (T1, 🔴 DPA pending VRM-GAP-001), SP-02 Cloudflare (T1, 🟡 ToS DPA), SP-03 Resend (T2, 🔴 DPA pending VRM-GAP-001), SP-04 RevenueCat (T2, 🔴 DPA pending VRM-GAP-001), SP-05 Apple (T1 Special, 🟡 compensating control), SP-06 Sentry (T2, 🔴 DPA pending VRM-GAP-001), SP-07 Anthropic (T1, 🟢 enterprise DPA active), SP-08 ElevenLabs (T2, 🔴 pending OQ-SP-01), SP-09 PostHog EU (T2, 🔴 pending P1 M4), SP-10 Stripe (T2, 🟢 ToS DPA active), SP-11 WorkOS (T1, 🟡 pending verification). Enterprise DPA template DPA-FORM-001 v1.0 (§5): full Art. 28(3)(a)–(h) clauses, Annex I description of processing, Annex II TOMs (10 control categories with evidence references), Annex III sub-processor snapshot. SIEM export DPA addendum (§6, DPA-FORM-SIEM-ADDENDUM-001 v1.0): 3-scenario legal classification resolving OQ-SIEM-02; §6.3 allow-list of 7 known cloud SIEM SaaS domains (no addendum required); addendum template required only for non-customer-controlled endpoints. Self-service DPA acceptance (§7): Admin Dashboard → Legal → DPA History; 3-tier qualification by deal size (< 500 / 500–2,000 / > 2,000 seats); `admin.dpa_accepted` DEC-030 event with sub_processor_list_snapshot_hash; PDF generation to compliance vault; versioning and re-acceptance protocol. 9 DEC-030 HMAC-chained events (§9, all 7-year retention): admin.dpa_accepted (HIGH), admin.dpa_redline_requested (MEDIUM), admin.dpa_executed (CRITICAL), admin.sub_processor_added (HIGH), admin.sub_processor_removed (HIGH), admin.sub_processor_change_notice_sent (STANDARD), admin.sub_processor_change_objection_received (HIGH), admin.siem_export_enabled (HIGH), admin.dpa_re_acceptance_required (STANDARD). Privacy invariant: tenant_ids_notified stored as integer count (not array) in audit events; all user_id references pseudonymous; customer signatory email SHA-256 hashed in events. SOC 2 mapping: CC9.2 (sub-processor register + DPA template + change notification + annual review), C1.1 (confidentiality obligations imposed on sub-processors via DPA Annex II), P3.2 (sub-processor transparency via published list and change notices), P6.1 (processor disclosure to controller via DPA Annex III + snapshot hash in admin.dpa_accepted), CC2.3 (public sub-processor list = external communication). 5 evidence artefacts SP-E-001 through SP-E-005. 14-item implementation checklist: 3× P0 M6 (SP-01/03/04 DPA executions, VRM-GAP-001 items 1–3), 1× P0 M6 (SP-06 Sentry DPA + beforeSend CI test, VRM-GAP-001 item 4), 1× P0 M5 (external counsel DPA review — blocks first enterprise pilot), 7× P1 M4–M8, 2× P2 M7–M10. 4 open questions: OQ-SP-01 (ElevenLabs DPA gap — P1, M5), OQ-SP-02 (Expo/EAS T4 vs T2 re-classification — P2, M10), OQ-SP-03 (new AI provider gate — P2, standing process), OQ-SP-04 (self-service adoption tracking to close COST_MODEL OQ-35 — P1, after 3rd deal). Owner: compliance-officer + security-engineer + enterprise-architect.*
