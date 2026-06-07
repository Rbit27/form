# FORM · Security Awareness & Training Policy

> **POL-012 · v1.0**
> Owner: `compliance-officer` (programme governance) + `security-engineer` (delivery + evidence).
> Status: `AUTHORED_PENDING_APPROVAL` — in force upon compliance-officer approval; no external counsel dependency for technical policy.
> Review cadence: Annual (Q4 December, aligned to §15 compliance calendar) or on material change to team size, product scope, or regulatory environment.
> SOC 2 criteria addressed: **CC1.4** (Commitment to Competence), **CC2.2** (Communicates Internally), **CC1.2** (Management Accountability), **CC9.2** (Vendor Risk Awareness).
> Cross-references: `docs/SOC2_READINESS.md §22`, `docs/ACCEPTABLE_USE_POLICY.md` (POL-001), `docs/INCIDENT_RESPONSE.md`, `docs/SECURITY.md`, `docs/ENGINEERING_RUNBOOK.md`, `docs/AUDIT_LOG_SCHEMA.md` (DEC-030 chain).

---

## Table of Contents

1. [Policy Statement](#1-policy-statement)
2. [Purpose & SOC 2 Mapping](#2-purpose--soc-2-mapping)
3. [Scope](#3-scope)
4. [Roles & Responsibilities](#4-roles--responsibilities)
5. [Required Training Topics (8-Topic Curriculum)](#5-required-training-topics-8-topic-curriculum)
6. [Solo-Founder Annual Training Plan](#6-solo-founder-annual-training-plan)
7. [New Hire Security Onboarding Checklist](#7-new-hire-security-onboarding-checklist)
8. [Annual Refresher Training Cadence](#8-annual-refresher-training-cadence)
9. [Phishing Simulation Programme](#9-phishing-simulation-programme)
10. [Evidence Package & Artefact Registry](#10-evidence-package--artefact-registry)
11. [DEC-030 Audit Events](#11-dec-030-audit-events)
12. [Non-Compliance & Escalation](#12-non-compliance--escalation)
13. [Gap Closure Table](#13-gap-closure-table)
14. [Policy Version History](#14-policy-version-history)

---

## 1. Policy Statement

FORM requires that every person with access to production systems, customer data, or the private compliance repository completes documented security awareness training before credentials are provisioned and annually thereafter.

This policy is the **human-layer control** that ensures FORM's technical security controls (documented in `docs/SECURITY.md`) and its compliance obligations (documented in `docs/SOC2_READINESS.md`) are understood, practised, and retained by everyone who can affect the confidentiality, integrity, or availability of customer data.

Training is not optional. Completion is a hard gate for production access. Annual renewal is a hard gate for continued access. Missing the annual deadline does not temporarily suspend training obligations — it begins an escalation sequence that ends with access suspension if not remediated within 14 days.

---

## 2. Purpose & SOC 2 Mapping

FORM processes special-category health data under GDPR Article 9. Every person with production access is a potential risk vector for that data. Security awareness training converts documented policies into internalised behaviours.

| SOC 2 Criterion | Requirement addressed by this policy |
|---|---|
| **CC1.4 — Commitment to Competence** | Documented training programme with completion evidence; competency areas defined per role |
| **CC2.2 — Communicates Internally** | Structured internal communication of security policies, acceptable use, and data handling obligations |
| **CC1.2 — Management Accountability** | Founder/compliance-officer completes training and signs off annually; management models behaviour |
| **CC9.2 — Vendor Risk Awareness** | Vendor risk awareness training supports third-party risk management obligations |

This policy **does not replace**:
- `docs/SECURITY.md` — which specifies technical controls
- `docs/INCIDENT_RESPONSE.md` — the IR runbook (though IR orientation is a training topic)
- `docs/ACCEPTABLE_USE_POLICY.md` (POL-001) — the AUP that all persons in scope must sign

---

## 3. Scope

### 3.1 In scope — always

All persons who hold access to any of the following are subject to this policy with no exceptions:

| System | Access type |
|---|---|
| Supabase (production project) | Any role — service_role, postgres, or app-level |
| Cloudflare Workers / Pages / Dashboard | Any deploy or write permission |
| R2, KV, Durable Objects | Any write or admin permission |
| GitHub (`github.com/Rbit27/form`) | Write access to `main` or any protected branch |
| 1Password (production vaults) | Any vault containing production credentials or backup encryption keys |
| PostHog, Sentry, PagerDuty | Any role with access to user-identifiable data |
| Private compliance repository | Any read or write access |

Full-time employees, part-time employees, and contractors are treated identically. Employment status does not reduce training obligations.

### 3.2 Partial scope — contractors < 30 days

Short-term contractors with engagement length under 30 days and **no production system access** are out of scope for the full 8-topic curriculum but must:
1. Sign the Acceptable Use Policy (POL-001) before any access is granted.
2. Complete a 30-minute security briefing covering data classification (Topic 3) and incident reporting (Topic 4).

### 3.3 Out of scope

Third-party auditors, pentesters, and regulators accessing FORM systems under a signed engagement agreement with explicit time-bound access boundaries are out of scope. Their access is governed by the engagement agreement, which must include data handling obligations equivalent to this policy's obligations.

---

## 4. Roles & Responsibilities

| Role | Responsibilities |
|---|---|
| **compliance-officer** | Programme ownership; annual content review; evidence filing; gap tracking in Linear; escalation decisions; policy approval and version control |
| **security-engineer** | Training delivery (new hire orientations, tabletop exercises); MFA hard-gate verification; phishing simulation platform administration; evidence collection |
| **founder** | Completing training as a participant during solo-founder phase; modelling behaviour; signing AUP; approving budget for training platforms |
| **Every person in scope** | Completing all required training before deadline; filing evidence artefacts correctly; immediately reporting suspected phishing or security incidents regardless of training status |

The compliance-officer and security-engineer may be the same person at the solo-founder stage. During that phase, the founder acts as all three roles and the structural limitation is documented as a compensating control acceptance (see §6.3 below).

---

## 5. Required Training Topics (8-Topic Curriculum)

Every person in scope must complete training in all eight topics before production access is provisioned (new hires) or by 28 February of each calendar year (existing personnel, annual refresh). Topics are not ranked by priority — all eight are required.

| # | Topic | SOC 2 Criteria | Delivery method | Frequency |
|---|---|---|---|---|
| **1** | **OWASP Top 10** — web application vulnerability classes including injection (A01), broken access control (A01), cryptographic failures (A02), SSRF (A10), and insecure design (A04) | CC6.1, CC6.6, PI1.2 | Interactive course: OWASP WebGoat or equivalent; completion certificate required | Annual; Q3 refresher each year (see §8.2) |
| **2** | **Phishing & social engineering** — recognising and reporting phishing emails, smishing, pretexting, and vishing; indicators of compromise; correct reporting path | CC1.4, CC2.2 | KnowBe4 or GoPhish simulation (post-hire); self-directed module with scenario exercises (solo phase) | Annual training + quarterly simulations post-hire (see §9) |
| **3** | **Data classification & handling** — the four-tier classification scheme (`docs/SECURITY.md §13`), Restricted data obligations including GDPR Art. 9 health data, prohibited sharing patterns, and the FORM privacy floor enforcements | CC2.2, C1.1, P1.1 | Internal document review with signed attestation; founder-authored for solo phase | Annual; whenever classification scheme changes materially |
| **4** | **Incident response** — roles and responsibilities during a security incident, escalation paths, the five-phase IR process (`docs/INCIDENT_RESPONSE.md §18.1`), and notification obligations under GDPR Art. 33 (72-hour supervisory authority) and Art. 34 (data subject notification) | CC7.3, CC7.4, CC7.5 | Tabletop exercise (`docs/INCIDENT_RESPONSE.md §18.6`); scenario walkthrough | Annual tabletop; mandatory ad-hoc refresher after any real incident ≥ SEV-1 |
| **5** | **Credential hygiene** — 1Password mandatory for all credentials, MFA requirements on all production systems, SSH key rotation procedure (`docs/KEY_ROTATION.md`), prohibition on shared credentials, prohibition on credentials in code or git history | CC6.1, CC6.2, CC6.3 | Policy review + hands-on verification: MFA gate checked by security-engineer before credentials issued | Annual; immediate mandatory refresher on any credential exposure event |
| **6** | **Secure code review** — identifying security issues during pull request review: hardcoded secrets, missing input validation, RLS bypass patterns, insecure direct object references, injection vectors, and missing authentication checks | CC6.1, CC8.1, PI1.2 | Cloudflare Security Learning Path + internal PR review checklist (`docs/ENGINEERING_RUNBOOK.md`) | Annual; checklist enforced on every PR |
| **7** | **Privacy & GDPR** — Article 9 special-category health data obligations, lawful basis for each processing activity, data subject rights (DSAR, erasure, portability), DPIA triggers, breach notification timelines, and FORM's no-individual-data-to-employer privacy floor | P1.1, P2.1, P3.1, P4.1, P5.1, P6.1 | Supabase security and privacy documentation review + ICO GDPR controller self-assessment; external DPO review annually once user base exceeds 500 EU residents | Annual; whenever a new data category is introduced |
| **8** | **Vendor risk awareness** — recognising supply-chain risk, understanding DPA and SCC requirements, the process for onboarding a new sub-processor (`docs/SOC2_READINESS.md §6`), and red flags for vendor compromise | CC9.2, C1.2 | Internal sub-processor register review (`docs/SUBPROCESSORS.md`); vendor security questionnaire walkthrough (`docs/SECURITY_QUESTIONNAIRE.md`) | Annual; whenever a new sub-processor is added |

### 5.1 Curriculum coverage statement

These eight topics align with AICPA guidance for CC1.4 competency evidence and are calibrated to FORM's current risk surface: a health-data SaaS processing GDPR Art. 9 data, running on Cloudflare Workers + Supabase, with AI inference via third-party APIs (Anthropic, ElevenLabs).

If the product surface expands materially — native mobile app with on-device data, on-premise deployment, HIPAA-covered-entity customers, or AI model training on user data — the curriculum must be reviewed within 30 days of the material change and updated with a training impact assessment filed to `/evidence/cc1/training-impact-assessments/`.

---

## 6. Solo-Founder Annual Training Plan

During the solo-founder phase (before first employee hire), the following training activities constitute the complete annual programme. All must be completed by **28 February** of each calendar year, with evidence filed to the private compliance repository under `/evidence/cc1/training/YYYY/`.

### 6.1 Certifications and courses

| Course / Resource | Platform | Completion evidence | Topics covered |
|---|---|---|---|
| **OWASP WebGoat** — complete all modules for OWASP Top 10 2021 (A01–A10) | OWASP WebGoat (self-hosted or owasp.org) | Screenshot of module completion progress; exported completion record if available | 1, 6 |
| **Cloudflare Security Learning Path** — Workers security, zero-trust concepts, DDoS and bot protection | developers.cloudflare.com/learning-paths/security | Account dashboard screenshot showing completion; or notes document with module titles and completion dates | 5, 6 |
| **Supabase Security Documentation Review** — RLS, Auth configuration, API key scoping, connection pooling security | supabase.com/docs (Security section) | Signed internal attestation: "I have reviewed the Supabase security documentation as of [date] and confirm understanding of RLS policies, API key rotation, and Auth security configuration" | 3, 5, 6 |
| **GDPR Controller Self-Assessment** — Article 9 obligations, DPIA completion, DSAR workflow test | ICO self-assessment toolkit or equivalent | Completed self-assessment export or signed attestation referencing the assessment tool and date | 7 |
| **Phishing awareness module** — self-directed scenario exercises covering credential harvesting, spear-phishing, and pretexting | KnowBe4 free resources, Google Phishing Quiz, or equivalent | Screenshot of module completion or signed attestation with date and module name | 2 |
| **Vendor risk review** — review current sub-processor register and confirm all DPAs are in place | `docs/SUBPROCESSORS.md` + Cloudflare, Supabase, Anthropic DPA status | Signed attestation: "Sub-processor register reviewed on [date]. All listed processors have active DPAs. [Any gap noted]" | 8 |

### 6.2 Internal document review cadence

| Document | Review trigger | Evidence artefact |
|---|---|---|
| `docs/SECURITY.md` | Annual (Q1) + any material change | Signed attestation: "Reviewed [doc] on [date]. No policy gaps identified" or "Reviewed — gap: [description] — tracked in Linear as [ticket]" |
| `docs/INCIDENT_RESPONSE.md` | Annual (Q1) + after any real incident | Same format |
| `docs/SUBPROCESSORS.md` | Annual (Q1) + any sub-processor change | Same format |
| `docs/SOC2_READINESS.md` | Annual (Q1) + quarterly for gap tracking | Same format |
| This policy (`docs/SECURITY_AWARENESS_TRAINING_POLICY.md`) | Annual (Q1) | Signed attestation confirming policy reviewed and content is current |

### 6.3 Compensating control acceptance (solo founder)

The following structural limitations are formally accepted during the solo-founder phase. These acceptances expire immediately when the relevant second person is hired.

| Gap | Compensating control | Acceptance condition | Expiry |
|---|---|---|---|
| Independent training completion verification | Founder documents completion of equivalent external training; certificate or screenshot stored in `/evidence/cc1/training/YYYY/`; evidence must be verifiable by a third-party auditor without founder attestation alone | Evidence must include a date visible in the artefact itself, not just in the filename | Expires when first employee needs training — cohort delivery requires independence |
| Trainer ≠ trainee | Founder acts as both programme owner and sole participant; training content sourced from external providers (OWASP, ICO, Cloudflare) to provide third-party validation of content quality | All training uses external published curricula; custom internal content is supplementary only | Expires at first hire where independence is structurally possible |
| Phishing simulation (requires multiple participants) | Phishing awareness covered by Topic 2 self-directed training; simulation programme activates post-hire | External training module + signed attestation is accepted as equivalent for solo phase | Expires 30 days after first hire; KnowBe4 / GoPhish must be configured before simulation cohort launches |

---

## 7. New Hire Security Onboarding Checklist

Every person granted production system access must complete the following eight steps **in order** before credentials are provisioned. Steps are sequential — a step may not be skipped or deferred. **Step 3 (MFA verification) is a hard gate: no credentials are issued until the security-engineer has personally verified MFA is active on all required platforms.**

| Step | Action | Owner | Verification method | Evidence artefact |
|---|---|---|---|---|
| **1** | Review and sign the Acceptable Use Policy (AUP, POL-001) — covers permitted use, credential sharing prohibition, mandatory incident reporting, and data classification obligations | security-engineer (witness); new hire (signatory) | Signed PDF | `/evidence/cc1/onboarding/[name]-[YYYY-MM-DD]-aup-signed.pdf` |
| **2** | Complete all eight required training topics (§5) — may be completed over first five business days, but must be finished before solo production access is granted | new hire (self-directed); security-engineer (verifies completion) | Training completion evidence per topic | `/evidence/cc1/onboarding/[name]-[YYYY-MM-DD]-training-complete/` |
| **3** | **MFA hard gate** — enrol MFA on all provisioned accounts: GitHub (TOTP or hardware key), 1Password (hardware key strongly recommended), Supabase (TOTP), Cloudflare (TOTP or hardware key). Security-engineer verifies MFA active on each platform before credentials are considered live. | security-engineer | Screenshot of each platform's MFA settings page showing MFA enabled and enrolment date | Filed alongside AUP in `/evidence/cc1/onboarding/[name]-[YYYY-MM-DD]-mfa-confirmed.pdf` |
| **4** | 1Password onboarding — receive vault access; confirm password manager is installed on work machine; spot-check: no `~/.ssh/config` pointing to plain-text keys; no `.env` files in home directory with production values | security-engineer | Verbal confirmation + spot-check checklist signed by security-engineer | Signed checklist filed to onboarding evidence directory |
| **5** | SSH key generation — generate new Ed25519 key pair on work machine; register public key on GitHub; confirm private key is passphrase-protected and stored only on work machine (not in 1Password, cloud storage, or email) | new hire | Public key visible in GitHub account settings; security-engineer confirms Ed25519 or RSA-4096 minimum | Linear ticket with key fingerprint noted |
| **6** | Access provisioning review — security-engineer provisions access to only the systems required for the role (least privilege); all grants logged in the access control matrix | security-engineer | Access control matrix updated | Linear ticket listing all systems granted + role assigned |
| **7** | Incident response orientation — 30-minute walkthrough of `docs/INCIDENT_RESPONSE.md` with security-engineer; confirm new hire knows: how to report a suspected incident, the escalation path, and what they must not do during an active incident (no public disclosure before founder approval) | security-engineer (facilitates) | Signed attestation by new hire | `/evidence/cc1/onboarding/[name]-[YYYY-MM-DD]-ir-orientation.pdf` |
| **8** | 30-day check-in — security-engineer schedules follow-up to address questions, confirm no credential issues, and confirm access scope remains appropriate | security-engineer | Linear ticket with 30-day outcome documented | Access control matrix updated if any changes made |

### 7.1 Re-onboarding trigger

If a person's production access is suspended for more than 90 days (e.g., extended leave, contract hiatus), steps 3, 4, and 5 must be repeated before access is reinstated. Steps 1, 2, 6, 7, and 8 are waived only if no policy changes have occurred during the suspension period — if any policy changed, the affected topics must be re-reviewed and re-attested.

---

## 8. Annual Refresher Training Cadence

Training effectiveness degrades without reinforcement. The following cadence governs refresher training for all persons in scope.

### 8.1 Q1 annual refresh (February — all topics)

**Deadline: 28 February each calendar year.**

Every person in scope completes a full refresh of all eight training topics (§5). For courses with certificates (OWASP WebGoat, Cloudflare Security track), a fresh completion run is required — prior-year certificates do not satisfy the current-year obligation. For attestation-based topics, a new signed attestation with the current year's date is required.

The deadline is firm. An extension beyond 28 February requires written approval from the compliance-officer filed to the compliance repository. Extensions grant up to 14 additional days maximum. If training is not completed by the extended deadline, production access is suspended pending completion (see §12).

### 8.2 Q3 OWASP refresher (August)

**Deadline: 31 August each calendar year.**

OWASP publishes updates periodically. Even in years without a new Top 10 publication, the Q3 refresher requires:

1. Review of OWASP advisories published since the Q1 training run.
2. Review of CVEs affecting FORM's dependency stack (Cloudflare Workers runtime, Supabase/PostgreSQL, Node.js / TypeScript runtime) published since Q1.
3. A signed attestation confirming the review was completed and listing any CVEs identified as relevant to FORM's stack.

If a new OWASP Top 10 is published in the prior 12 months, the Q3 refresher must include a full re-run of OWASP WebGoat modules covering new or reclassified vulnerability categories.

### 8.3 Post-incident mandatory refresher

Whenever a security incident of severity SEV-1 or higher is resolved, the incident post-mortem (`docs/INCIDENT_RESPONSE.md §18.5`) must include an assessment of whether a training gap contributed to the incident. If a training gap is identified:

1. A targeted refresher on the relevant topic(s) is **mandatory** for all persons in scope within **14 days** of the post-mortem being completed.
2. The refresher is documented as a separate training artefact referencing the incident ticket ID.
3. The curriculum (§5) is reviewed for systemic weakness; if found, the topic is updated before the next Q1 annual cycle.

---

## 9. Phishing Simulation Programme

> **Applicability: post-hire only.** Phishing simulation requires a minimum of two participants to be meaningful. During the solo-founder phase, phishing awareness is covered by Topic 2 self-directed training (§5, §6). The simulation programme activates upon first hire.

### 9.1 Programme design

| Dimension | Specification |
|---|---|
| **Platform** | KnowBe4 (preferred — SAML integration; AICPA-recognised audit reports) or GoPhish (open-source self-hosted alternative if cost constraint at early stage) |
| **Frequency** | Quarterly — four simulations per calendar year |
| **Scenario rotation** | Each quarter uses a different scenario type, rotating across: (1) credential harvesting (fake login page), (2) spear-phishing (personalised sender spoofing), (3) attachment-based (malicious attachment lure), (4) SMS/voice (if mobile devices in scope) |
| **Difficulty progression** | Year 1: low-to-medium (obvious sender anomalies, generic lures). Year 2+: escalate progressively; add domain-lookalike attacks and internal-spoofing scenarios |
| **Results reporting** | To compliance-officer within 5 business days of simulation completion: number of targets, click rate, credential-submission rate, and reporting rate (users who flagged the simulation as suspicious) |

### 9.2 Failure protocol

> **No punitive action. Ever.** The phishing simulation programme is a training tool, not a disciplinary mechanism. Clicking a simulated phishing link is not a performance issue. Public identification of who clicked, manager notification for disciplinary purposes, or any compensation impact are **prohibited** responses to simulation failure. These outcomes destroy the psychological safety required for people to report real phishing attempts promptly — the opposite of the programme's intent.

| Outcome | Required response | Timeline |
|---|---|---|
| Clicked link (no credential submission) | Immediate in-platform training module (5–10 min micro-course on the technique used) | Delivered automatically by platform within 24 hours |
| Submitted credentials | In-platform training module + personal 1:1 debrief with security-engineer covering the specific indicators present in the simulation | Debrief within 5 business days |
| Reported simulation as suspicious | Positive acknowledgement from security-engineer; reporting rate tracked as a KPI (target: > 30% report rate by Year 2) | Acknowledgement within 2 business days |
| Aggregate click rate > 30% in any quarter | Programme-wide refresher on Topic 2 (phishing) for all persons in scope, regardless of individual performance | Completed within 30 days of simulation results |

### 9.3 Individual record handling

Individual-level simulation results (who clicked, who submitted credentials) are retained internally for training tracking but are **not** included in the SOC 2 evidence package. Auditors receive aggregate results only. Individual records are subject to data minimisation obligations (`docs/SECURITY.md §13`) and are retained for no longer than 3 years.

---

## 10. Evidence Package & Artefact Registry

### 10.1 Evidence artefact naming convention

All training evidence is filed to the private compliance repository using the format:

```
/evidence/cc1/training/YYYY/CC1-E-00X-YYYY-[topic-slug]-[type].[ext]
```

Examples:
- `/evidence/cc1/training/2026/CC1-E-001-2026-owasp-webgoat-screenshot.png`
- `/evidence/cc1/training/2026/CC1-E-002-2026-cloudflare-security-track-attestation.pdf`
- `/evidence/cc1/training/2026/CC1-E-003-2026-gdpr-self-assessment-export.pdf`
- `/evidence/cc1/onboarding/alice-smith-2026-07-01-aup-signed.pdf`
- `/evidence/cc1/phishing/2026-Q3-simulation-report.pdf`

The completion date must be **visible within the artefact itself** (not only in the filename) so an auditor can independently verify the timeline.

### 10.2 SOC 2 auditor evidence package (CC1-E-001 through CC1-E-005)

| Evidence ID | Description | Location | Refresh cadence | Owner |
|---|---|---|---|---|
| **CC1-E-001** | Annual training completion records — certificates, screenshots, and signed attestations for all eight topics (§5) for each person in scope | `/evidence/cc1/training/YYYY/` — one sub-directory per calendar year | Annual (Q1, deadline 28 February) | compliance-officer |
| **CC1-E-002** | New hire security onboarding completion record — all eight checklist steps (§7) with verification artefacts per hire | `/evidence/cc1/onboarding/[name]-[date]-complete/` | Per hire | security-engineer |
| **CC1-E-003** | Q3 OWASP refresher attestation — signed record of CVE review and any new OWASP guidance reviewed | `/evidence/cc1/training/YYYY/q3-owasp-refresher-YYYY.pdf` | Annual (Q3, deadline 31 August) | compliance-officer |
| **CC1-E-004** | Acceptable Use Policy — current signed version (POL-001) plus signed copies from all persons in scope | `/evidence/cc1/aup/aup-current-signed-[name]-[date].pdf` | Per hire; re-signed on material policy change | security-engineer |
| **CC1-E-005** | Phishing simulation quarterly reports — aggregate results (click rate, credential-submission rate, reporting rate) | `/evidence/cc1/phishing/YYYY-QN-simulation-report.pdf` | Quarterly (post-hire only; §9) | compliance-officer |

### 10.3 Solo-phase auditor note

During the solo-founder observation period, CC1-E-002 will be absent (no hires) and CC1-E-005 will be absent (phishing simulation requires multiple participants). This is expected and disclosed. The primary controls for the solo phase are CC1-E-001 and CC1-E-004. Audit firms experienced with pre-revenue SaaS companies (Prescient Assurance, Johanson Group) routinely accept this pattern as appropriate for the operating context, provided both artefacts are demonstrably complete and filed before the observation period closes.

---

## 11. DEC-030 Audit Events

All training completions, simulation results, and onboarding completions emit HMAC-SHA256 chained audit events per `docs/AUDIT_LOG_SCHEMA.md` (DEC-030). These events are append-only and tamper-evident. Retention: 7 years.

| Event type | Sensitivity | Trigger | Key fields | Retention |
|---|---|---|---|---|
| `training.annual_completion_recorded` | STANDARD | Compliance-officer files CC1-E-001 artefacts to evidence repository and commits to compliance repo | `person_id` (pseudonymised), `year`, `topics_completed` (array of 8 topic IDs), `evidence_artefact_paths`, `completion_date`, `compliance_officer_id` | 7 years |
| `training.new_hire_onboarding_completed` | HIGH | Security-engineer marks all 8 onboarding steps complete in Linear; access provisioned | `hire_id` (pseudonymised), `hire_start_date`, `onboarding_complete_date`, `mfa_verified_by`, `systems_provisioned` (array), `aup_signed_date` | 7 years |
| `training.mfa_gate_passed` | HIGH | Security-engineer verifies MFA active on all required platforms before credentials issued | `person_id`, `platforms_verified` (array), `verified_by`, `verified_at` | 7 years |
| `training.phishing_simulation_completed` | STANDARD | Compliance-officer files quarterly simulation report | `year`, `quarter`, `platform`, `target_count`, `click_rate_pct`, `credential_submission_rate_pct`, `reporting_rate_pct`, `report_path` | 7 years |
| `training.access_suspended_overdue` | HIGH | Compliance-officer suspends production access due to training overdue > 14 days | `person_id`, `suspension_reason`, `access_systems_suspended` (array), `suspended_by`, `suspended_at`, `reinstatement_condition` | 7 years |
| `training.access_reinstated` | HIGH | Compliance-officer reinstates access after training completion | `person_id`, `reinstated_by`, `reinstated_at`, `training_completion_evidence_path` | 7 years |

---

## 12. Non-Compliance & Escalation

Training non-compliance is treated as a security control failure, not an administrative oversight.

### 12.1 Annual training (deadline: 28 February)

| Days past deadline | Action | Owner |
|---|---|---|
| **Day 1–7** (1 March – 7 March) | Automated reminder via compliance calendar; compliance-officer sends individual reminder | compliance-officer |
| **Day 8–14** (8 March – 14 March) | Compliance-officer escalates to founder; person's manager notified if applicable; Linear ticket opened with P1 priority | compliance-officer + founder |
| **Day 15+** | Production access suspended pending completion. Access suspension is logged via `training.access_suspended_overdue` DEC-030 event. Suspension lifted within 24h of completing all outstanding training and filing evidence. | compliance-officer (suspension decision); security-engineer (access revocation) |

### 12.2 New hire onboarding (deadline: end of Day 5 post-start)

If onboarding steps are not complete by the end of the fifth business day, no production credentials may be provisioned. The hiring manager (or founder, pre-hire-team) must be notified. The access provisioning Linear ticket remains in "blocked" state until all steps are verified.

### 12.3 Emergency access during suspension

A person with suspended training access may request emergency production access under the break-glass procedure (`docs/AUDIT_LOG_SCHEMA.md §break-glass`). Break-glass access:
- Requires 2-person approval (founder + compliance-officer)
- Is time-bound: maximum 4 hours
- Does not waive the training obligation — the person must still complete outstanding training within 24 hours of emergency access expiry
- Generates a `security.break_glass_access` DEC-030 HIGH event that references the suspension context

---

## 13. Gap Closure Table

This policy formally closes the following gaps identified in `docs/SOC2_READINESS.md §22.9` and the compliance calendar.

| Control | Status before this policy | Status after this policy | Notes |
|---|---|---|---|
| Security awareness training programme | 🟡 Partial — documented in §22 but not standalone policy | 🟢 **Done** — POL-012 standalone policy; 8-topic curriculum; evidence artefacts CC1-E-001 through CC1-E-005 formally specified |  |
| New hire security onboarding | 🟡 Partial — checklist in §22.5 | 🟢 **Done** — formalised in §7 with MFA hard gate and 8-step sequential checklist | |
| Annual refresher training | 🟡 Partial — cadence in §22.6 | 🟢 **Done** — Q1/Q3/post-incident cadence formalised in §8 | |
| Phishing simulation | 🟡 Partial — programme designed but post-hire dependency | 🟡 **Partial** — programme formalised in §9; activates post-hire; CC1-E-005 defined | Remains partial until first hire and first simulation run |
| Policy register (POL-012) | 🔴 Missing — 11 policies in log; §22 extraction noted as pending | 🟢 **Done** — add POL-012 row to `compliance/policy-approval-log.csv` (post-approval) | |

**Impact on SOC 2 readiness:** CC1.4 and CC2.2 gaps reduced from 🟡 Partial to 🟢 Done for all components except phishing simulation (remains 🟡 pending first hire). Net readiness score improvement: ~+1.5% (estimate; full re-score at next quarterly review).

---

## 14. Policy Version History

| Version | Date | Author | Changes |
|---|---|---|---|
| **v1.0** | 2026-06-07 | compliance-officer (enterprise-builder) | Initial standalone policy — extracted and formalised from `docs/SOC2_READINESS.md §22`. Added: standalone policy statement (§1); formal scope table (§3); roles & responsibilities matrix (§4); DEC-030 audit event registry (§11); non-compliance escalation procedure with suspension SLAs (§12); policy version table (§14). Content preserves all substantive content from §22 with added governance structure. Closes CC5 gap: "Security Awareness Training standalone policy" noted in `compliance/policy-approval-log.csv` row 4. Cross-reference to SOC2_READINESS.md §22 preserved for audit trail continuity. |

---

*POL-012 · Security Awareness & Training Policy · v1.0 · 2026-06-07*
*compliance-officer + security-engineer · FORM*
*Co-Authored-By: Claude (enterprise-builder) <cloud@form.coach>*
