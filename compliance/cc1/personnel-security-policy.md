# FORM · Personnel Security, Background Check & Confidentiality Onboarding Policy

| Field | Value |
|---|---|
| **Policy ID** | POL-014 |
| **Version** | v1.0 |
| **Status** | IN_FORCE |
| **Effective date** | 2026-06-07 |
| **Owner** | `compliance-officer` + `security-engineer` |
| **Next review** | 2027-01-01 (§15 compliance calendar Q1-Jan; accelerated on any hire or contractor engagement) |
| **Gap closures** | CC1-GAP-003 · PRE-04 |
| **SOC 2 criteria** | CC1.1 · CC1.4 · CC6.1 · CC6.3 · P3.1 |
| **Evidence artifacts** | CC1-E-003a · CC1-E-003b · CC1-E-003c · CC1-E-005 · CC1-E-006 · CC1-E-007a · CC1-E-007b |
| **Source section** | `docs/SOC2_READINESS.md §42` |

> **Standalone auditor exhibit.** This document extracts the personnel security policy from `docs/SOC2_READINESS.md §42` into a discrete, auditor-deliverable artefact. Cross-reference §42 for the full implementation narrative and gap-closure history.
>
> **Privacy floor:** No employee personal data, no background check finding detail, no health data may appear in any evidence artefact produced under this policy. Evidence identifiers use `employee_id` (internal pseudonym); no names, dates of birth, or criminal record details are stored in git-tracked files.

---

## 1. Scope

This policy applies to:

| Role category | Coverage | First trigger |
|---|---|---|
| **Full-time employees** (Founding Engineer, Design Lead, future hires) | Full background check + NDA before Day 1 system access | Founding Engineer conditional offer |
| **Part-time contractors** with production access (code, infra, data) | Identity verification + abbreviated background check + NDA before access granted | First contractor engagement |
| **Advisory board members** | NDA before any non-public product or financial information shared | First advisory session |
| **Penetration test firms** | Mutual NDA (MNDA) before credentials or architecture diagrams are shared | Scope kick-off meeting |
| **Solo-founder (current phase)** | Self-attestation; background check not applicable | Continuous |

**Out of scope:** End users of the FORM app; enterprise tenant admin users (governed by Enterprise Agreement and `docs/ENTERPRISE.md`); sub-processors (governed by `docs/SUBPROCESSORS.md §3` and `docs/SOC2_READINESS.md §28`).

---

## 2. Background Check Policy

### 2.1 Trigger and Timing

Background check is **mandatory** before any individual receives:

- GitHub push access to `rbit27/form` or any production repository
- Supabase production project access (any role above read-only)
- 1Password production vault access
- Cloudflare account access (any permission level)
- Direct access to Art. 9 health data (workout records, coaching turns, CV sessions, biometric readings)

**Timing:** Background check initiated **before** a conditional offer is extended. No production access granted until the check result is received and reviewed by `compliance-officer`.

### 2.2 Provider

**Primary:** Checkr (EU entity; GDPR DPA available; SOC 2 Type II certified). Lower friction for < 5 hires/year; automated workflows; 1–5 business day turnaround.

**Fallback:** Sterling. If Checkr is unavailable or the candidate's jurisdiction requires deeper local criminal record verification, Sterling is the approved alternative. Document provider selection rationale in `compliance/evidence/cc1/background-check-provider-selection.md` (CC1-E-003a) before contracting.

### 2.3 Check Scope

| Check type | Mandatory | Rationale |
|---|---|---|
| Identity verification (government ID) | Yes | Confirms candidate identity before any access |
| Criminal record check (country of residence + global watchlist) | Yes | Access to Art. 9 health data per GDPR Art. 9 sensitivity; SOC 2 CC1.1 |
| Employment history verification (last 5 years / 3 employers) | Yes | CC1.4 competence; confirms claimed credentials |
| Education verification (highest degree claimed) | Conditional — only if degree is cited as a qualification criterion in the job description | Avoids scope creep |
| Credit check | No | FORM does not handle direct financial transactions |
| Drug screen | No | Remote-first; not material to role |

### 2.4 GDPR Legal Basis for Background Check Processing

Legal basis: **GDPR Art. 6(1)(b)** (pre-contractual steps) and, for criminal record data, **GDPR Art. 10** under national law of the candidate's country of residence.

FORM obligations:
1. Provide candidate with a background check notice at initiation: legal basis, data categories processed, provider name, retention period, and right to dispute.
2. Instruct Checkr as a **GDPR Art. 28 data processor** under an executed DPA (filed as CC1-E-003b).
3. Retain only the pass/fail attestation (check ID + outcome) for 7 years; destroy the full report within 12 months of check completion.

### 2.5 Adverse Result Handling

| Result | Definition | Action |
|---|---|---|
| **Clear** | No disqualifying findings | Conditional offer converts to unconditional; access provisioning begins per §5 |
| **Consider** | Finding exists; adjudication required | `compliance-officer` applies §2.6 criteria; individual assessment before employment decision |
| **Suspended / Dispute** | Candidate disputes a finding | Access provisioning paused; dispute resolution per Checkr procedure |
| **Failed** | Disqualifying finding confirmed | Conditional offer withdrawn; outside counsel consulted before communication to candidate |

### 2.6 Adjudication Criteria

FORM applies **individualized assessment** per EEOC guidance and EU/UA employment law:
- Nature of the finding and relevance to the specific role duties
- Time elapsed since the incident
- Evidence of rehabilitation

**Automatic disqualifiers (no individualized assessment):** Conviction for fraud, identity theft, computer crimes, or data privacy violations within the last 7 years; any finding involving unauthorized access to computer systems; any finding involving a minor victim.

**Non-disqualifiers:** Minor traffic violations; incidents > 10 years ago with no pattern; offences unrelated to data access scope.

All adjudication decisions recorded in `compliance/evidence/cc1/adjudication-log.csv` (pseudonymized — check ID only) for 7-year retention.

---

## 3. Confidentiality Agreement (NDA) Policy

### 3.1 NDA Requirement

Every individual in scope must sign a confidentiality agreement **before** any of:
- Access to non-public product roadmap, pricing, or financial information
- Access to any production system
- Access to any Art. 9 health data
- Participation in a user research session where content is shared with FORM

### 3.2 NDA Template Structure

The master FORM NDA template is filed as `compliance/ndas/TEMPLATE-employee-nda.md` (CC1-E-005). Outside counsel review required before first hire. The template must cover:

| Clause | Required content |
|---|---|
| **1. Parties** | FORM (legal entity name once incorporated) and individual |
| **2. Confidential Information** | All non-public technical, financial, and user data; explicitly includes all Art. 9 data categories: workout records, coaching turns, CV sessions, wearable readings, meal logs, biometric data |
| **3. Obligations** | Hold in strict confidence; no third-party disclosure; use only for role duties; minimum reasonable care per GDPR Art. 32 |
| **4. Exclusions** | Publicly available; independently developed; received from third party lawfully; required by law (prior notice to FORM where permissible) |
| **5. Data protection** | Process personal data only as directed by FORM and consistent with `docs/SECURITY.md`; report suspected breach to `security@form.coach` within 1 hour |
| **6. Intellectual property** | All work product is work-for-hire assigned to FORM; explicitly includes ML artefacts (model weights, training data annotations, CV pipeline improvements, coaching prompt templates) |
| **7. Non-solicitation** | 12-month post-termination non-solicit of FORM employees and customers (review enforceability by jurisdiction) |
| **8. Term** | Effective from signing; survives termination for 5 years; health data obligations survive indefinitely |
| **9. Governing law** | Ukraine law (primary); EU GDPR takes precedence for data protection obligations |
| **10. Signature** | DocuSign qualified electronic signature or wet ink; date |

### 3.3 NDA Signing Workflow

```
Step 1 — Template preparation (compliance-officer, pre-offer):
  - Generate NDA from TEMPLATE-employee-nda.md
  - Fill: candidate name, role title, start date, effective date
  - Send via DocuSign to candidate email

Step 2 — Candidate signature:
  - Candidate signs via DocuSign qualified e-signature
  - DocuSign timestamps serve as binding execution evidence

Step 3 — Counter-signature:
  - Founder counter-signs in DocuSign

Step 4 — Filing:
  - Completed PDF → compliance/ndas/{YYYY}/{employee-id}-nda-signed.pdf
  - SHA-256 hash of PDF recorded in DEC-030 event personnel.nda_signed
  - Row added to compliance/ndas/nda-register.csv

Step 5 — Annual audit:
  - §15 compliance calendar Q1-Jan: NDA register audited
  - Re-sign required if template version changes materially
```

### 3.4 NDA Register

File: `compliance/ndas/nda-register.csv`

Columns: `individual_id` (pseudonymized) · `role` · `nda_version` · `signed_date` · `docusign_envelope_id` · `pdf_sha256` · `dec030_event_id` · `status` (`active` / `superseded` / `terminated`) · `superseded_date` · `termination_date`

### 3.5 Advisory Board and Contractor NDA Variants

| Recipient | Template | Modifications |
|---|---|---|
| Advisory board members | `compliance/ndas/TEMPLATE-advisor-nda.md` | Clause 7 (non-solicitation) removed; Clause 6 (IP) scoped to advisory work product only |
| Short-term contractors < 30 days (no production access) | `compliance/ndas/TEMPLATE-contractor-confidentiality-rider.md` | Clauses 2, 3, 4, 5, 8 only |
| Penetration test firms | `compliance/ndas/TEMPLATE-pentest-mnda.md` | Mutual NDA; both parties bound |

---

## 4. Pre-Boarding Security Checklist

Executed by `compliance-officer` + `security-engineer`. All access revocation steps (1–6) within **24 hours** of termination notice; device return within **7 business days**.

| # | Day | Task | Owner |
|---|---|---|---|
| 1 | Day −14 | Background check initiated via Checkr; candidate notified with background check notice | compliance-officer |
| 2 | Day −10 | NDA sent via DocuSign; counter-signed by founder | compliance-officer |
| 3 | Day −7 | Background check result reviewed; adjudication completed; go/no-go decision documented | compliance-officer |
| 4 | Day −5 | 1Password Teams seat provisioned; Starter vault access granted (no production secrets vault) | security-engineer |
| 5 | Day −3 | GitHub account added as `member` to `rbit27` org; no push access to `main` before first code review approved | security-engineer |
| 6 | Day −2 | Role-specific access matrix reviewed per `docs/SOC2_READINESS.md §26`: permissions scoped, no over-provisioning | security-engineer |
| 7 | Day −1 | `personnel.hire_check_passed` DEC-030 event emitted | security-engineer |
| 8 | Day 0 | Production access granted in order: (a) Supabase read-only → (b) Cloudflare dev-only → (c) expand per role matrix. No write access to production until Day-30 probationary review | security-engineer |
| 9 | Day 0 | Security awareness training assigned per CC1.4 programme (`docs/SOC2_READINESS.md §22`); 30-day completion deadline | compliance-officer |
| 10 | Day 0 | AUP signed (`docs/ACCEPTABLE_USE_POLICY.md`); copy filed as CC1-E-002 | compliance-officer |
| 11 | Day 30 | First access review: confirm all granted permissions appropriate; expand to production write access if probationary review passes | security-engineer + compliance-officer |

---

## 5. DEC-030 Audit Events

All events use the standard HMAC-chained envelope (`event_id`, `event_type`, `actor_id`, `timestamp_utc`, `hmac_chain_value`). No health data, no coaching content, no `user_id` in any payload.

| Event type | Severity | Retention | Trigger |
|---|---|---|---|
| `personnel.background_check_initiated` | STANDARD | 7 yr | Step 1 of pre-boarding checklist |
| `personnel.background_check_passed` | HIGH | 7 yr | Step 3 — adjudication clear; written by compliance-officer via admin API |
| `personnel.nda_signed` | STANDARD | 7 yr | Step 2 — DocuSign completion |
| `personnel.hire_check_passed` | HIGH | 7 yr | After both `background_check_passed` and `nda_signed` confirmed |

**Chain break rule:** A `personnel.hire_check_passed` event with a broken HMAC chain is a **P0 incident** per `docs/INCIDENT_RESPONSE.md §R-05`. Do not grant production access until chain integrity is restored.

---

## 6. Evidence Artefacts for SOC 2 Auditors

| Artefact ID | Description | Location | SOC 2 criteria |
|---|---|---|---|
| **CC1-E-003a** | Background check provider selection rationale (Checkr vs. Sterling evaluation) | `compliance/evidence/cc1/background-check-provider-selection.md` | CC1.4 |
| **CC1-E-003b** | Checkr DPA (EU) — GDPR Art. 28 data processor agreement | `compliance/cc1/checkr-dpa.pdf` | CC1.4, P3.1 |
| **CC1-E-003c** | Anonymized background check completion log (check IDs, status, date — no names) | `compliance/evidence/cc1/background-check-log-{YYYY}.csv` | CC1.4 |
| **CC1-E-005** | NDA templates (all four variants) — counsel-reviewed versions with approval date | `compliance/ndas/TEMPLATE-*.md` + `compliance/policy-approval-log.csv` | CC1.1 |
| **CC1-E-006** | NDA register — all signed NDAs in force, pseudonymized | `compliance/ndas/nda-register.csv` | CC1.1 |
| **CC1-E-007a** | Pre-boarding security checklist completion record (one per hire) | `compliance/evidence/cc1/onboarding-checklist-{employee-id}-{YYYY-MM-DD}.md` | CC1.4, CC6.3 |
| **CC1-E-007b** | DEC-030 audit log filtered on `event_type LIKE 'personnel.%'`, HMAC chain verified | HMAC audit log query | CC1.1, CC1.4, CC6.3 |

---

## 7. SOC 2 Criteria Mapping

| SOC 2 Criterion | How this policy satisfies it |
|---|---|
| **CC1.1 — Integrity and ethical values** | NDA template (CC1-E-005) binds all personnel to confidentiality and data protection obligations; AUP signing sets conduct standards; NDA register (CC1-E-006) provides evidence of coverage |
| **CC1.4 — Commitment to competence** | Background check (§2) verifies identity and criminal history before production access; pre-boarding checklist (§4) ensures security training assigned Day 0 |
| **CC6.1 — Logical access provisioning** | Background check confirms identity before any access; NDA signed before access; `personnel.hire_check_passed` DEC-030 event is the access-granting trigger |
| **CC6.3 — Logical access removal** | §4 pre-boarding checklist scopes access to minimum required; Day-30 access review confirms appropriateness; offboarding per `compliance/c1/device-disposal-policy.md §3` ensures revocation within 24h of departure |
| **P3.1 — Personal data collection** | §2.4 background check notice covers GDPR disclosure obligation for candidate personal data processed during screening |

---

## 8. Gap Closure Record

| Gap ID | Status | Closed by |
|---|---|---|
| **CC1-GAP-003** — Background check provider not contracted | 🟡 AUTHORED (closes to 🟢 on Checkr account creation + DPA execution + test check) | §42 of `docs/SOC2_READINESS.md`; this policy |
| **PRE-04** — NDA/confidentiality agreements not authored | 🟡 AUTHORED (closes to 🟢 on outside counsel review + first signing) | §42 of `docs/SOC2_READINESS.md`; this policy |

---

## 9. Implementation Checklist

| # | Task | Owner | Priority | Milestone |
|---|---|---|---|---|
| 1 | Create Checkr account; complete onboarding; execute EU DPA and file as CC1-E-003b | compliance-officer | **P0** | Before Founding Engineer offer |
| 2 | Run test background check on a willing non-employee volunteer; file anonymized result as CC1-E-003c | compliance-officer + security-engineer | **P0** | Before Founding Engineer offer |
| 3 | Commission outside counsel to review NDA template (§3.2); confirm enforceability in UA/EU/relevant contractor jurisdiction; file approval in `compliance/policy-approval-log.csv` | compliance-officer + outside counsel | **P0** | Before Founding Engineer offer |
| 4 | Create `compliance/ndas/` directory with four template files and `nda-register.csv` seed (header row only) | compliance-officer | **P0** | Before Founding Engineer offer |
| 5 | Create `compliance/evidence/cc1/background-check-provider-selection.md` (CC1-E-003a) from §2.2 table | compliance-officer | **P0** | On Checkr contract signing |
| 6 | Register four `personnel.*` DEC-030 event types in `docs/AUDIT_LOG_SCHEMA.md` event registry | security-engineer | **P0** | Before first hire |
| 7 | Add pre-boarding checklist template to `compliance/cc1/TEMPLATE-onboarding-checklist.md` per §4 step table | compliance-officer + security-engineer | **P0** | Before Founding Engineer offer |
| 8 | Commit `compliance/ndas/founder-self-attestation.md` as NDA equivalent for solo-founder phase | founder | **P1** | M3 |
| 9 | Implement `POST /internal/admin/personnel/events` admin API endpoint for manual DEC-030 emission | platform-engineer + security-engineer | **P1** | M3 |
| 10 | Wire Checkr and DocuSign webhooks to DEC-030 event emission (automated path) | platform-engineer | **P1** | M4 |

---

*Owner: compliance-officer + security-engineer · v1.0 · 2026-06-12 · Extracted from `docs/SOC2_READINESS.md §42` · Next review: 2027-01-01*

*Co-Authored-By: Claude (enterprise-builder) <cloud@form.coach>*
