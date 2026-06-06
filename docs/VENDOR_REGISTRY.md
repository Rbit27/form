# FORM · Vendor Risk Registry v1.0

> Standalone current-state registry of all vendors across all tiers (T1 Critical → T4 Peripheral).
> Supplements `docs/SUBPROCESSORS.md` (GDPR Art. 28 context + DPA template) and `docs/SOC2_READINESS.md §17` (review process programme).
> Owner: `compliance-officer` + `security-engineer`. Updated within 5 business days of any change.
> SOC 2 evidence: CC9.1 (vendor management programme), CC9.2 (security review of service providers), CC9.3 (vendor contract requirements).
> Cross-references: `docs/SUBPROCESSORS.md`, `docs/SOC2_READINESS.md §17` (process), `docs/SOC2_READINESS.md §14` (risk register), `docs/AUDIT_LOG_SCHEMA.md` (DEC-030 HMAC chain), `docs/ENTERPRISE.md`.

---

## TL;DR

| Metric | Current state |
|---|---|
| Total vendors in registry | 15 |
| T1 Critical | 5 (Supabase, Cloudflare, Apple, Anthropic, WorkOS) |
| T2 Important | 6 (Resend, RevenueCat, Sentry, ElevenLabs, PostHog, Stripe) |
| T3 Standard | 0 |
| T4 Peripheral | 4 (GitHub, 1Password, Better Stack, Expo/EAS) |
| DPAs executed | 3 of 11 sub-processors (Anthropic, Stripe, Cloudflare partial) |
| 🔴 Auto-escalated vendors | 5 (DPA factor = 3) |
| Open critical gaps | VRM-GAP-001 (5 pending DPAs), VRM-GAP-002 (ElevenLabs no cert), VRM-GAP-003 (Resend no cert) |
| Next full review | January 2027 (§9) |
| Registry version date | 2026-06-06 |

**Privacy floor:** sub-processors never receive `user_id` linked to GDPR Art. 9 health categories; pseudonymous identifiers only. Enforced by prompt construction controls (GDPR_DPIA §4), SDK `beforeSend` filters (OBSERVABILITY §28.3), and PostHog event schema (OBSERVABILITY §25.7).

---

## 1. Scope and Classification

This registry covers every third party that:
- processes personal data on FORM's behalf (GDPR Art. 28 sub-processors),
- has privileged access to FORM production systems (credentials, read access),
- or whose availability failure would directly affect platform uptime.

Open-source libraries and npm packages are out of scope — covered by Dependabot / `npm audit` per `docs/SOC2_READINESS.md §15.1 PRE-13`.

### Tier definitions

| Tier | Definition | DPA required | Review cadence | Min evidence |
|---|---|---|---|---|
| **T1 Critical** | Processes GDPR Art. 9 Restricted data OR single point of failure for platform availability | Yes — executed before data flows | Annual full + quarterly status check | SOC 2 Type II report + security questionnaire |
| **T2 Important** | Processes Confidential personal data; outage degrades (not kills) platform | Yes — executed before data flows | Annual full | SOC 2 bridge letter or 8-item questionnaire |
| **T3 Standard** | Operational tooling; limited personal data exposure (metadata, IP) | Not required if no direct data flows; ToS acceptable | Annual self-attestation; full review every 2 years | Vendor self-attestation letter |
| **T4 Peripheral** | No FORM user personal data processed; infrastructure or build tooling only | Not required | Biennial review | Compliance-officer acknowledgement |

Full classification criteria and review process: `docs/SOC2_READINESS.md §17`.

---

## 2. T1 Critical Vendors

> DPA execution is a hard gate before any production data flows. No waiver for T1.

### SP-01 · Supabase, Inc.

| Field | Value |
|---|---|
| **Service** | Database (PostgreSQL), Authentication (GoTrue), File Storage (S3-compatible) |
| **Data categories** | Restricted: health metrics, coaching sessions, CV analysis outputs, clinical flags. Confidential: user identity, tenant configuration, audit log, meal log. |
| **Processing location** | EU — Frankfurt `de-fra1` (EU tenants); US — `us-east-1` (US tenants) |
| **Transfer mechanism** | EU adequacy (Frankfurt); SCCs Module 2 (US tenants via DigitalOcean/AWS) |
| **DPA status** | 🔴 **Pending execution** — VRM-GAP-001 P0 M6 |
| **Certification** | SOC 2 Type I (2024); Type II audit in progress |
| **Subprocessors** | DigitalOcean (primary hosting), AWS (backup/storage), Fly.io (edge) — inherited via Supabase DPA chain |
| **Composite risk score** | 🔴 **Auto-escalated** — DPA factor = 3 (§6) |
| **Evidence target** | VRM-E-001 (signed DPA PDF → `compliance/evidence/vendors/supabase/`) |
| **Next review** | On DPA execution + January 2027 |
| **Owner** | `security-engineer` |

### SP-02 · Cloudflare, Inc.

| Field | Value |
|---|---|
| **Service** | CDN, Edge Workers (API runtime), R2 Object Storage, Access Zero Trust, WAF, Email Workers |
| **Data categories** | Confidential: usage metadata (IP, request headers, timestamps). R2: encrypted compliance-vault objects (health data never in plaintext). |
| **Processing location** | Global CDN; R2 EU jurisdiction restriction pending confirmation (OQ-VRM-03) |
| **Transfer mechanism** | Cloudflare GDPR DPA + Data Privacy Framework + SCCs Module 2 |
| **DPA status** | 🟡 **ToS DPA active** — standalone executed DPA preferred; tracking OQ-VRM-03 (EU R2 data residency confirmation) |
| **Certification** | SOC 2 Type II (2025), ISO 27001 |
| **Subprocessors** | Cloudflare operates its network directly; limited sub-chain |
| **Composite risk score** | 🟡 **Medium** — score 9/18 (§6) |
| **Evidence target** | `compliance/evidence/vendors/cloudflare/` |
| **Next review** | January 2027 |
| **Owner** | `security-engineer` |

### SP-05 · Apple, Inc.

| Field | Value |
|---|---|
| **Service** | Push notification delivery (APNs) |
| **Data categories** | Device token (pseudonymous); notification payload — non-health text only (no health values or user PII in payload by architectural constraint) |
| **Processing location** | US/Global |
| **Transfer mechanism** | Apple Developer Program License Agreement — no standalone GDPR Art. 28 DPA commercially available; see OQ-VRM-01 |
| **DPA status** | 🟡 **Compensating control** — payload minimisation enforced at SDK level; risk acceptance filed at `compliance/evidence/vendors/apple/no-dpa-compensating-control-{date}.md` (VRM-E-011) |
| **Certification** | No standalone SOC 2; Apple operates under its own enterprise security programme |
| **Composite risk score** | 🟡 **Medium** — score 10/18; compensating controls reduce residual risk (§6) |
| **Evidence target** | VRM-E-011 (compensating control document) |
| **Next review** | January 2027 |
| **Owner** | `compliance-officer` |

### SP-07 · Anthropic, PBC

| Field | Value |
|---|---|
| **Service** | AI inference — Victor coaching LLM completions (Claude API) |
| **Data categories** | Confidential: pseudonymised exercise context (exercise type, rep counts, cues). No PII, no health values in prompts — enforced by prompt construction gate (GDPR_DPIA §4 constraint PA-05). |
| **Processing location** | US |
| **Transfer mechanism** | SCCs Module 2 (EU Commission Decision 2021/914); Anthropic enterprise DPA includes data-not-used-for-training clause |
| **DPA status** | 🟢 **Enterprise DPA active** |
| **Certification** | SOC 2 Type II (2025) |
| **Subprocessors** | AWS (inference infrastructure) — inherited via Anthropic DPA |
| **Composite risk score** | 🟢 **Low** — score 8/18 (§6) |
| **Evidence target** | `compliance/evidence/vendors/anthropic/dpa-active.pdf` |
| **Next review** | January 2027 |
| **Owner** | `security-engineer` |

### SP-11 · WorkOS, Inc.

| Field | Value |
|---|---|
| **Service** | Enterprise SSO (SAML 2.0 / OIDC) and SCIM 2.0 directory synchronisation |
| **Data categories** | Confidential: enterprise employee identity — email, display name, group membership, IdP subject identifier. Consumer user data is never routed through WorkOS. |
| **Processing location** | US |
| **Transfer mechanism** | SCCs Module 2 |
| **DPA status** | 🟡 **Pending verification** — WorkOS DPA available; execution not confirmed (SOC2 §11 item 6) |
| **Certification** | SOC 2 Type II (2024) |
| **Composite risk score** | 🟡 **Medium** — score 9/18; DPA pending verification (§6) |
| **Evidence target** | `compliance/evidence/vendors/workos/dpa-{date}.pdf` |
| **Next review** | On DPA verification + January 2027 |
| **Owner** | `enterprise-architect` + `compliance-officer` |

---

## 3. T2 Important Vendors

> DPA execution required before data flows. SOC 2 bridge letter accepted in lieu of questionnaire where current cert exists.

| SP | Vendor | Service | Data | DPA status | Cert | Risk score | Owner |
|---|---|---|---|---|---|---|---|
| SP-03 | **Resend, Inc.** | Transactional email | Email address, non-health notification content, DSAR payload delivery | 🔴 Pending — VRM-GAP-001 | No SOC 2 — VRM-GAP-003 | 🔴 Auto (DPA=3) | `compliance-officer` |
| SP-04 | **RevenueCat, Inc.** | Subscription lifecycle | Subscription status, entitlement events, iOS/Android receipt | 🔴 Pending — VRM-GAP-001 | SOC 2 Type I (2024) | 🔴 Auto (DPA=3) | `compliance-officer` |
| SP-06 | **Sentry, Inc.** | Error monitoring | Error context (stack traces, breadcrumbs); health data scrubbed by `beforeSend` deny-list (OBSERVABILITY §28.3) | 🔴 Pending — VRM-GAP-001 | SOC 2 Type II (2025) | 🔴 Auto (DPA=3) | `security-engineer` |
| SP-08 | **ElevenLabs, Inc.** | Text-to-speech (Victor voice) | Coaching text cues only — no user PII in payload (PA-06 constraint) | 🔴 Pending — OQ-SP-01 | No SOC 2 — VRM-GAP-002 | 🔴 Auto (DPA=3) | `security-engineer` |
| SP-09 | **PostHog, Inc.** (EU Cloud) | Product analytics | Anonymised event stream — no health metrics (OBSERVABILITY §25.7); pseudonymous `distinct_id` (SHA-256 UUID) | 🔴 Pending — P1 M4 | SOC 2 Type I (2024) | 🔴 Auto (DPA=3) | `compliance-officer` |
| SP-10 | **Stripe, Inc.** | Subscription billing webhooks | Subscription status, payment event metadata. No card data — PCI-DSS scope is Stripe-only. | 🟢 Stripe DPA active via Business ToS | SOC 2 Type I (2025), PCI-DSS Level 1 | 🟡 Medium (score 9) | `compliance-officer` |

---

## 4. T3 Standard Vendors

None currently in scope. If Better Stack or PagerDuty expand to process personal data in the future, they will be promoted to T3 and this section populated. Reclassification triggers: access to Confidential or Restricted data, alerting payloads that include user-identifiable error context.

---

## 5. T4 Peripheral Vendors

> No FORM user personal data processed. Biennial compliance-officer review. No DPA required.

| Vendor | Service | Why T4 | Sensitivity note | Next review |
|---|---|---|---|---|
| **GitHub** | Source code hosting, Actions CI | Source code and CI artefacts — no user personal data | Do not commit secrets; enforce branch protection + Dependabot | Jan 2028 |
| **1Password** | Credential management for engineering team | Stores team credentials and API keys — no user personal data | Holds production secrets: treat access as privileged; MFA enforced; break-glass policy per SOC2 §23 | Jan 2028 |
| **Better Stack** (Uptime) | Uptime monitoring, status page | Pings public URLs and records HTTP status — no user data | Availability data only; no health payload in checks | Jan 2028 |
| **Expo / EAS** | iOS and Android build infrastructure | Build platform processes source code and app bundles — no user personal data | Builds contain compiled app binary; ensure no secrets baked into binary | Jan 2028 |

**1Password risk note:** While T4 by data-processing definition, 1Password has effective privileged access to all FORM production systems via stored API keys and service credentials. Access is governed by `docs/SOC2_READINESS.md §23` (access review) and `docs/KEY_ROTATION.md`. The compliance-officer must confirm 1Password MFA enforcement annually as part of the access review (PRE-23), not the vendor review.

**OQ-SP-02 open question:** Whether Expo/EAS should be reclassified T2 if it processes signed build artefacts that include HMAC signing keys or OTA update payloads. Target resolution: M10. Owner: `platform-engineer` + `compliance-officer`.

---

## 6. Composite Risk Score Summary

Risk scoring methodology: `docs/SOC2_READINESS.md §17.6`. Six factors, each scored 1 (Low) to 3 (High). Total range 6–18.

**Auto-escalation rule:** Any vendor scoring 3 on the DPA factor (no DPA, processing personal data) is automatically 🔴 regardless of composite total.

| SP | Vendor | Data sensitivity | Certification | DPA status | Incident history | Data residency | Sub-chain depth | **Total** | **Rating** |
|---|---|---|---|---|---|---|---|---|---|
| SP-01 | Supabase | 3 | 2 | **3** | 1 | 1 | 2 | 12 | 🔴 Auto |
| SP-02 | Cloudflare | 2 | 1 | 2 | 1 | 2 | 1 | 9 | 🟡 Medium |
| SP-03 | Resend | 2 | 3 | **3** | 1 | 2 | 2 | 13 | 🔴 Auto |
| SP-04 | RevenueCat | 2 | 2 | **3** | 1 | 2 | 2 | 12 | 🔴 Auto |
| SP-05 | Apple | 2 | 2 | 2† | 1 | 2 | 1 | 10 | 🟡 Medium |
| SP-06 | Sentry | 2 | 1 | **3** | 1 | 2 | 1 | 10 | 🔴 Auto |
| SP-07 | Anthropic | 2 | 1 | 1 | 1 | 2 | 1 | 8 | 🟢 Low |
| SP-08 | ElevenLabs | 2 | 3 | **3** | 1 | 2 | 1 | 12 | 🔴 Auto |
| SP-09 | PostHog | 2 | 2 | **3** | 1 | 1 | 1 | 10 | 🔴 Auto |
| SP-10 | Stripe | 2 | 1 | 1 | 1 | 2 | 2 | 9 | 🟡 Medium |
| SP-11 | WorkOS | 2 | 1 | 2 | 1 | 2 | 1 | 9 | 🟡 Medium |

† Apple: DPA factor scored 2 (compensating control documented, not 3) because no personal data flows in push payload by architectural constraint.

**Score legend:**
| Total | Rating | Action |
|---|---|---|
| 6–8 | 🟢 Low | Annual review per §9 |
| 9–11 | 🟡 Medium | Compliance-officer quarterly status; remediation plan for any factor-3 within 90 days |
| 12–14 | 🟠 Elevated | Joint compliance-officer + security-engineer quarterly; remediation plan within 30 days |
| 15–18 | 🔴 High | Immediate founder escalation; continued use requires written risk acceptance; migrate within 6 months |
| Auto | 🔴 Auto | DPA=3 override — no waiver; remediation required before first data flow |

---

## 7. Open Gap Register

> Gaps that remain open after each annual review must be escalated and tracked here until closed.

### VRM-GAP-001 🔴 HIGH — Pending DPAs for 5 sub-processors

| Field | Value |
|---|---|
| **Affected vendors** | SP-01 Supabase, SP-03 Resend, SP-04 RevenueCat, SP-06 Sentry, SP-08 ElevenLabs, SP-09 PostHog |
| **SOC 2 criteria** | CC9.2, C1.1 |
| **Blocking for** | SOC 2 CC9.2 evidence, all enterprise onboarding |
| **Target close** | M6 (Supabase, Resend, RevenueCat, Sentry); M4 PostHog; M5 ElevenLabs (OQ-SP-01) |
| **Evidence on close** | VRM-E-001 (Supabase DPA), VRM-E-002 (Resend), VRM-E-003 (RevenueCat), VRM-E-004 (Sentry), VRM-E-005 (ElevenLabs), VRM-E-006 (PostHog) |
| **Owner** | `compliance-officer` |
| **Compensating control** | Each vendor has architectural data-minimisation controls in place that reduce risk during the gap period. Health data flows do not begin until DPA is executed; feature gates enforce this at the application layer. |

### VRM-GAP-002 🟠 ELEVATED — ElevenLabs has no SOC 2 certification

| Field | Value |
|---|---|
| **Affected vendor** | SP-08 ElevenLabs |
| **SOC 2 criteria** | CC9.2 |
| **Risk** | T2 vendor; startup stage; no third-party security certification |
| **Compensating control** | 8-item security questionnaire required before DPA execution (OQ-SP-01). No PII in TTS payloads (PA-06 architectural constraint). |
| **Target close** | ElevenLabs SOC 2 Type I expected Q3 2026 per vendor roadmap. If unavailable by M9, migrate to alternative TTS provider or downgrade Victor voice feature. |
| **Evidence on close** | VRM-E-007 (ElevenLabs SOC 2 report or completed questionnaire) |
| **Owner** | `security-engineer` |

### VRM-GAP-003 🟡 MEDIUM — Resend has no SOC 2 certification

| Field | Value |
|---|---|
| **Affected vendor** | SP-03 Resend |
| **SOC 2 criteria** | CC9.2 |
| **Risk** | Email delivery vendor processes user email addresses; no third-party cert |
| **Compensating control** | 8-item security questionnaire required at DPA execution. Payload limited to non-health notification content (architectural constraint). |
| **Target close** | Annual review Q1 2027. If Resend has not obtained SOC 2 Type I by then, migrate to Postmark or SendGrid (both hold SOC 2 Type II). |
| **Evidence on close** | VRM-E-008 (Resend SOC 2 report or completed questionnaire) |
| **Owner** | `compliance-officer` |

### VRM-GAP-004 🟡 MEDIUM — Cloudflare R2 EU jurisdiction unconfirmed (OQ-VRM-03)

| Field | Value |
|---|---|
| **Affected vendor** | SP-02 Cloudflare |
| **Impact** | R2 Object Storage jurisdictional restriction for EU-resident user data not yet confirmed via Cloudflare dashboard configuration |
| **Target close** | M6 — devops-lead to confirm R2 bucket policy `{ "jurisdictionCode": "EU" }` before EU tenant onboarding |
| **Evidence on close** | Screenshot of R2 jurisdiction setting + Cloudflare DPA confirmation in `compliance/evidence/vendors/cloudflare/` |
| **Owner** | `devops-lead` |

---

## 8. Security Questionnaire Template

> Used for T1 Critical vendors (mandatory) and T2 Important vendors without a current SOC 2 Type II report.
> Send to vendor's security or legal contact. Minimum response time: 10 business days.
> File completed responses to `compliance/vendor-review/questionnaires/VENDOR-NAME-YYYY.{pdf,md}`.

```
FORM Vendor Security Questionnaire — Annual / Onboarding
Vendor: ___________________________________________
Completed by (name + title): ______________________
Date: _____________________________________________

1. ENCRYPTION
   (a) What encryption standards do you apply to data at rest?
       (Expected: AES-256 or equivalent.)
   (b) What TLS version do you enforce for data in transit?
       (Expected: TLS 1.2 minimum; TLS 1.3 preferred.)

2. ACCESS CONTROL
   (a) Describe your RBAC model and how access to FORM data is scoped.
   (b) Is MFA enforced for all employees with access to production systems?
   (c) How is privileged / admin access managed and reviewed?

3. INCIDENT NOTIFICATION
   (a) What is your SLA for notifying FORM of a security incident or data breach
       that affects FORM data? (Expected: ≤24 hours from discovery.)
   (b) Who is the designated security contact for incident notification?

4. PENETRATION TESTING
   (a) How frequently do you conduct penetration testing?
   (b) Are you willing to provide an executive summary of your most recent
       penetration test report on request under NDA?

5. DATA DELETION
   (a) Describe your process for deleting FORM data upon contract termination.
   (b) What is your maximum time-to-deletion SLA?
       (Expected: ≤30 days, with written confirmation.)

6. SUBPROCESSORS
   (a) List all subprocessors that may receive or process FORM data.
   (b) Do you notify customers ≥30 days before adding or changing a subprocessor?

7. DATA RESIDENCY
   (a) In which countries or regions is FORM data processed and stored?
   (b) Do you offer an EU data residency option for EU-resident user data?
   (c) What transfer mechanism(s) do you rely on for transfers outside the EEA?
       (Expected: SCCs Module 2, EU adequacy decision, or UK IDTA.)

8. SECURITY POSTURE
   (a) Do you hold a current SOC 2 Type II report or ISO 27001 certificate?
       If yes, provide a copy or a link to your Trust Portal.
   (b) Have you had any security incidents in the last 12 months that involved
       customer data? If yes, summarise the nature and resolution.
   (c) Do you have a formal vulnerability disclosure / bug bounty programme?
```

---

## 9. Annual Review Evidence Checklist

Full review process: `docs/SOC2_READINESS.md §17.5`. This checklist summarises the evidence artefacts required for SOC 2 auditor fieldwork (CC9.2).

| Step | Action | Deadline | Evidence artefact |
|---|---|---|---|
| 1 | Registry currency check — confirm all vendors current, tiers accurate, no unreviewed additions | January 5 | `compliance/vendor-review/YYYY/01-registry-currency-check.md` |
| 2 | Certification / SOC 2 report refresh for all T1 + T2 vendors | January 15 | `compliance/vendor-review/YYYY/02-cert-refresh/VENDOR-cert-YYYY.pdf` + summary |
| 3 | DPA status confirmation — re-execute if terms materially changed; confirm SCCs current | January 20 | `compliance/vendor-review/YYYY/03-dpa-status-check.md` |
| 4 | Questionnaire or self-attestation — T1 mandatory; T2 accept SOC 2 in lieu; T4 self-attestation | January 25 | `compliance/vendor-review/YYYY/04-questionnaire-responses/` |
| 5 | Annual review sign-off memo — risk score reassignment, open items, compliance-officer signature | January 31 | `compliance/vendor-review/YYYY/05-annual-review-memo.md` |

**SOC 2 CC9.2 evidence package:** Steps 1–5 artefacts jointly satisfy CC9.2 for the audit observation period. Auditors will typically request Steps 2 and 5 first; keep Steps 1–4 available for follow-up.

---

## 10. SOC 2 Evidence Mapping

| SOC 2 Criterion | Evidence provided by this registry |
|---|---|
| **CC9.1** — Risk mitigation activities for business disruptions, including vendor use | §2–5 (tier classification with defined review cadence), §7 (gap register with remediation plans) |
| **CC9.2** — Risk assessment and monitoring of third-party vendors | §6 (composite risk scores), §8 (questionnaire template), §9 (annual review evidence artefacts) |
| **CC9.3** — Evaluation of vendor compliance with obligations | §2–5 (DPA status column per vendor), §7 (VRM-GAP-001 DPA remediation tracking) |
| **P8.1** — Privacy enforcement with third parties | §2–3 (transfer mechanisms, DPA status); §5 (T4 no-data note); privacy floor note in TL;DR |
| **C1.1** — Confidentiality obligations imposed on processors | §2–3 (DPA terms required per tier); §8 questionnaire item 5 (deletion obligations) |

---

## 11. Audit Event References (DEC-030 HMAC Chain)

Vendor lifecycle events that must emit HMAC-chained audit log entries per `docs/AUDIT_LOG_SCHEMA.md` (DEC-030):

| Event | Trigger | Retention |
|---|---|---|
| `admin.sub_processor_added` | New vendor enters §2–3 | 7 years |
| `admin.sub_processor_removed` | Vendor removed from active registry | 7 years |
| `admin.sub_processor_change_notice_sent` | 30-day notice issued to enterprise tenants | 7 years |
| `admin.vendor_dpa_executed` | DPA signed by both parties; evidence filed | 7 years |
| `admin.vendor_offboarded` | Offboarding complete; deletion confirmation received | 7 years |

See `docs/SUBPROCESSORS.md §9` for full DEC-030 event schemas and `docs/AUDIT_LOG_SCHEMA.md` for the HMAC chain structure.

---

*v1.0 (2026-06-06): Initial standalone release. Closes the "formal vendor registry needed" gap in `docs/SOC2_READINESS.md` §5 (CC column) and §17.4 (embedded aspirational registry → now has standalone current-state counterpart). 15 vendors registered across T1–T4. Introduces VRM-GAP-002 (ElevenLabs no SOC 2) and VRM-GAP-003 (Resend no SOC 2) as new tracked gaps. Adds 8-item security questionnaire template previously referenced in SOC2 §17.3 as `compliance/vendor-review/questionnaire-template.md` but not yet authored. Adds annual review evidence checklist (Steps 1–5, SOC2 §17.5) as auditor-facing summary. Documents OQ-SP-02 (Expo/EAS T4 reclassification) and OQ-VRM-03 (Cloudflare R2 EU jurisdiction) as open questions. References: `docs/SUBPROCESSORS.md`, `docs/SOC2_READINESS.md §17`, `docs/AUDIT_LOG_SCHEMA.md`, `docs/ENTERPRISE.md`, DEC-030. SOC 2 mapping: CC9.1, CC9.2, CC9.3, P8.1, C1.1. Owner: compliance-officer + security-engineer.*
