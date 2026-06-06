# FORM Vendor Security Questionnaire — Template

**File reference:** VRM-QT-001
**SOC 2 criteria:** CC9.1, CC9.2, CC9.3, P8.1, C1.1
**Owner:** compliance-officer (review); security-engineer (technical sections)
**Status:** Active
**Version:** 1.0 — 2026-06-06
**Next review:** January 2027 (aligned with annual vendor review per `docs/SOC2_READINESS.md §15.1`)

---

## Purpose and Scope

This questionnaire is the primary evidence artefact for FORM's vendor security assessment process defined in `docs/SOC2_READINESS.md §17.3`. It documents the security posture of vendors that access or process FORM user data before a production connection is established, and annually thereafter.

**When this questionnaire is required:**

| Vendor tier | Requirement |
|---|---|
| **T1 Critical** — processes GDPR Art. 9 Restricted data or is a platform single-point-of-failure | Questionnaire mandatory. No waiver. SOC 2 Type II report does not replace it; both are required. |
| **T2 Important** — processes Confidential personal data; outage degrades (not kills) platform | Questionnaire or current SOC 2 Type II / ISO 27001 cert accepted. If cert exists and is current, use the cert and skip the questionnaire. |
| **T3 Standard** — limited personal data; meaningful but recoverable impact if vendor fails | Vendor self-attestation accepted in lieu. Do not use this template. |
| **T4 Peripheral** — no personal data processed | No questionnaire required. |

**Response SLA:** 10 business days from date of send. If no response within 15 business days, escalate to compliance-officer. Unresponsive vendors must not receive production data until questionnaire is completed.

**Filing completed responses:**
```
compliance/vendor-review/questionnaires/VENDOR-NAME-YYYY.md
```
For PDF responses: `compliance/vendor-review/questionnaires/VENDOR-NAME-YYYY.pdf`

**Audit event:** Upon receiving a completed response, emit an HMAC-chained audit log entry per `docs/AUDIT_LOG_SCHEMA.md` (DEC-030):
- `event_type: admin.vendor_questionnaire_received`
- Include: vendor name, tier, reviewer, date received, VRM evidence reference

---

## How to Use This Template

1. Copy this file to `compliance/vendor-review/questionnaires/VENDOR-NAME-YYYY.md`
2. Fill in vendor details at the top of the copy
3. Email the questionnaire to the vendor's security or legal contact
4. When the vendor returns responses, paste them under each question in the copy
5. Compliance-officer reviews responses against acceptance criteria (§2 below)
6. File the completed copy to `compliance/vendor-review/questionnaires/` and reference in the Vendor Risk Registry (`docs/VENDOR_REGISTRY.md`)
7. Onboarding approval is blocked until questionnaire review is completed and approval is documented

---

## Questionnaire

> **Instructions to vendor:** Please complete all sections in full. Incomplete responses will be returned. This questionnaire is confidential and is held under the terms of the NDA or DPA in place between your organisation and FORM. Respond within 10 business days of receipt. Questions: security@form.coach.

---

**Vendor organisation:** ___________________________________________________

**Completed by (full name + title):** _________________________________________

**Date of response:** _______________________________________________________

**FORM contact:** ___________________________________________________________

**Assessment type:** ☐ Onboarding — new vendor  ☐ Annual renewal  ☐ Triggered review

---

### Q1. Encryption

**(a)** What encryption standard do you apply to data at rest?

> **Expected:** AES-256 or equivalent. Describe the algorithm and key management approach (e.g., AWS KMS, customer-managed keys).

**Vendor response:**

---

**(b)** What TLS version do you enforce for data in transit? Is TLS 1.0 or 1.1 still supported on any endpoint?

> **Expected:** TLS 1.2 minimum; TLS 1.3 preferred. TLS 1.0 and 1.1 must be disabled.

**Vendor response:**

---

**(c)** How are encryption keys rotated, and what is your key rotation cadence for data at rest?

> **Expected:** Annual rotation minimum; automatic rotation preferred. Customer-managed key (CMK) option noted if available.

**Vendor response:**

---

### Q2. Access Control

**(a)** Describe your role-based access control (RBAC) model and explain how access to FORM data is scoped to only those employees who require it.

> **Expected:** Least-privilege enforcement; access scoped per customer / tenant; no shared credentials.

**Vendor response:**

---

**(b)** Is multi-factor authentication (MFA) enforced for all employees with access to production systems or customer data?

> **Expected:** MFA required for all production access. Hardware token or TOTP minimum; SMS MFA alone is not acceptable for production access.

**Vendor response:**

---

**(c)** How is privileged / admin access managed, reviewed, and revoked? What is your access review cadence?

> **Expected:** Privileged access is separately role-gated; just-in-time (JIT) access preferred. Quarterly or more frequent access reviews.

**Vendor response:**

---

### Q3. Incident Notification

**(a)** What is your contractual SLA for notifying FORM of a security incident or personal data breach that affects FORM data?

> **Expected:** ≤ 24 hours from discovery of an incident affecting FORM data. This SLA must also appear in the DPA.

**Vendor response:**

---

**(b)** Who is the designated security contact at your organisation for breach notification? Provide name, role, and email/phone.

> **Expected:** A named individual or on-call security alias with 24/7 coverage, not a generic support queue.

**Vendor response:**

---

**(c)** Have you experienced any security incidents in the past 12 months that involved customer data (including FORM data if applicable)? If yes, summarise the nature, scope, and resolution.

> **Expected:** Full disclosure. A vendor's refusal to answer this question is itself a risk flag requiring compliance-officer escalation.

**Vendor response:**

---

### Q4. Penetration Testing

**(a)** How frequently do you conduct third-party penetration tests? Who conducts them?

> **Expected:** Annual minimum; semi-annual preferred for Critical-tier services. Testing firm is independent (not internal red team only).

**Vendor response:**

---

**(b)** Are you willing to provide an executive summary of your most recent penetration test report upon FORM's request, under NDA?

> **Expected:** Yes. Refusal is a risk flag. Full report is not required — executive summary covering scope, critical and high findings, and remediation status is sufficient.

**Vendor response:**

---

**(c)** Do you have a formal vulnerability disclosure or bug bounty programme? If yes, provide the URL.

> **Expected:** A public or private programme is a positive signal. Not required but noted in risk score.

**Vendor response:**

---

### Q5. Data Deletion

**(a)** Describe your process for deleting FORM data — including backups and derived data — upon contract termination or upon FORM's written request.

> **Expected:** Full deletion including backups within 30 days of written request. Process is documented and tested.

**Vendor response:**

---

**(b)** What is your maximum time-to-deletion SLA from the date of FORM's written termination notice?

> **Expected:** ≤ 30 days. Written deletion confirmation (certificate or equivalent) provided to FORM on completion.

**Vendor response:**

---

**(c)** Do you retain anonymised or aggregated derivatives of FORM data after deletion? If yes, describe what is retained and the legal basis.

> **Expected:** No health data or PII retained in any form. Statistical aggregates that cannot be attributed to FORM or FORM users may be disclosed with justification.

**Vendor response:**

---

### Q6. Subprocessors

**(a)** List all subprocessors (third-party services) that may receive, store, or process FORM data in the course of providing your service to FORM.

> **Expected:** Complete list including cloud infrastructure provider, CDN, support tooling, analytics. "None" is acceptable if technically accurate.

| Subprocessor name | Country of processing | Purpose | Data categories |
|---|---|---|---|
| | | | |

**Vendor response (use table above or attach):**

---

**(b)** Do you notify customers at least 30 days before adding or materially changing a subprocessor?

> **Expected:** Yes, with opt-out or objection mechanism available for enterprise customers. This aligns with GDPR Art. 28(2).

**Vendor response:**

---

### Q7. Data Residency

**(a)** In which countries or regions is FORM data processed and stored (including backups)?

> **Expected:** Explicit list. EU residency is required for EU-resident user data absent an adequate transfer mechanism.

**Vendor response:**

---

**(b)** Do you offer an EU data residency option that ensures FORM's EU-resident user data is stored and processed only within the EEA?

> **Expected:** EU region available or EU-only deployment option confirmed. If not available, document compensating control.

**Vendor response:**

---

**(c)** For transfers of personal data outside the EEA, which transfer mechanism(s) do you rely on?

> **Expected:** EU Standard Contractual Clauses (SCCs, Commission Decision 2021/914, Module 2 controller-to-processor); EU adequacy decision (e.g., Japan, Canada, UK); or UK IDTA for UK data. US-only privacy certifications (Privacy Shield successors) are not accepted alone.

**Vendor response:**

---

### Q8. Security Posture

**(a)** Do you hold a current SOC 2 Type II report or ISO 27001 certificate? If yes, provide a copy or a link to your Trust Portal.

> **Expected:** SOC 2 Type II preferred (issued within last 12 months, covering services FORM will use). ISO 27001 (current certificate from accredited body) accepted. SOC 2 Type I is acceptable for T3 Standard only.

**Vendor response:**

---

**(b)** Do you have a formal Security Development Lifecycle (SDLC) process? Does it include mandatory security code review and/or SAST/DAST tooling for changes affecting customer data?

> **Expected:** Yes for T1 Critical. Evidence: CI/CD gates with SAST, mandatory security review for high-risk changes.

**Vendor response:**

---

**(c)** Do you have a Business Continuity Plan (BCP) and Disaster Recovery (DR) plan? What is your stated RTO and RPO for the service FORM will consume?

> **Expected:** Documented BCP and DR with RTO ≤ 4 hours and RPO ≤ 1 hour for Critical-tier services. Annual DR drill performed.

**Vendor response:**

---

## Acceptance Criteria

The compliance-officer reviews completed responses against these thresholds before issuing onboarding approval. Items marked 🔴 are blocking; items marked 🟡 require a documented compensating control or risk acceptance.

| Item | Blocking threshold | Risk flag (non-blocking) |
|---|---|---|
| Q1 Encryption | Non-AES-256 at rest; TLS 1.0/1.1 not disabled | Key rotation cadence > 2 years |
| Q2 Access control | MFA not enforced on production access; no RBAC | Access review cadence > 6 months |
| Q3 Incident notification | Notification SLA > 72h; named contact refused | Generic support queue only |
| Q3(c) Prior incidents | Refusal to answer | Unremediated high/critical finding disclosed |
| Q4 Pen test | No third-party pentest in last 24 months | Refusal to share executive summary on request |
| Q5 Deletion | Deletion SLA > 90 days; no deletion confirmation | Backups excluded from deletion scope |
| Q6 Subprocessors | Undisclosed subprocessors discovered post-onboarding | No 30-day change notice process |
| Q7 Data residency | EU user data stored outside EEA without valid transfer mechanism | EU region not available (document compensating control) |
| Q8 Security posture | No SOC 2 or ISO cert and no acceptable questionnaire (T1 only) | BCP/DR not tested in last 24 months |

**Approval authority:** compliance-officer. For T1 Critical vendors with any 🔴 blocking item, founder co-approval is required.

**Compensating controls:** Any 🟡 risk flag that cannot be remediated before onboarding must be documented in the Vendor Risk Registry (`docs/VENDOR_REGISTRY.md §7`) as a named VRM-GAP with a remediation plan and target date.

---

## Evidence Filing Checklist

After completing the review, the compliance-officer confirms:

- [ ] Completed questionnaire filed at `compliance/vendor-review/questionnaires/VENDOR-NAME-YYYY.{md,pdf}`
- [ ] Vendor entry updated in `docs/VENDOR_REGISTRY.md` (DPA status, cert, risk score, next review date)
- [ ] Any blocking items documented as VRM-GAPs in `docs/VENDOR_REGISTRY.md §7`
- [ ] Onboarding approval recorded in the registry with reviewer name and date
- [ ] `admin.vendor_questionnaire_received` audit event emitted (DEC-030 HMAC chain)
- [ ] `admin.vendor_dpa_executed` audit event emitted upon DPA signature (if applicable)

---

## SOC 2 Evidence Mapping

| Criterion | How this questionnaire satisfies it |
|---|---|
| **CC9.1** — Vendor risk identification | Questionnaire establishes baseline risk data for every in-scope vendor |
| **CC9.2** — Vendor risk assessment and monitoring | Completed responses + annual re-assessment cycle demonstrates ongoing monitoring |
| **CC9.3** — Vendor obligations evaluated | Q3 (notification SLA), Q5 (deletion), Q6 (subprocessors) directly map to contractual obligations in the DPA |
| **P8.1** — Privacy with third parties | Q6 and Q7 address subprocessor chain and lawful transfer mechanisms required under GDPR Art. 28 |
| **C1.1** — Confidentiality obligations on processors | Q5 (deletion on termination) and Q2 (access scoping) evidence that processors are bound to confidentiality obligations equivalent to FORM's |

---

## Audit Event Reference (DEC-030)

Vendor lifecycle events emitted as HMAC-chained audit log entries per `docs/AUDIT_LOG_SCHEMA.md`:

| Event type | Trigger | Retention |
|---|---|---|
| `admin.vendor_questionnaire_received` | Completed questionnaire filed in `compliance/vendor-review/questionnaires/` | 7 years |
| `admin.vendor_dpa_executed` | DPA signed by both parties; evidence filed in `compliance/dpa/` | 7 years |
| `admin.sub_processor_added` | Vendor enters active sub-processor register | 7 years |
| `admin.vendor_offboarded` | Offboarding complete; deletion confirmation received | 7 years |

See `docs/AUDIT_LOG_SCHEMA.md` for the full event schema and HMAC chain structure. See `docs/VENDOR_REGISTRY.md §11` for the complete vendor lifecycle event reference.

---

*v1.0 (2026-06-06): Initial authoring. Closes VRM-GAP referenced in `docs/VENDOR_REGISTRY.md` v1.0 changelog and the "not yet authored" note in `docs/SOC2_READINESS.md §17.3`. Eight-item questionnaire (Encryption, Access Control, Incident Notification, Penetration Testing, Data Deletion, Subprocessors, Data Residency, Security Posture) with acceptance criteria, filing checklist, SOC 2 evidence mapping, and DEC-030 audit event reference. Tier applicability: T1 Critical (mandatory), T2 Important (accepted in lieu of SOC 2 cert). Owner: compliance-officer + security-engineer. References: `docs/SOC2_READINESS.md §17.3`, `docs/VENDOR_REGISTRY.md §8`, `docs/AUDIT_LOG_SCHEMA.md`, DEC-030.*
