# FORM · CC3 Risk Assessment — Evidence Index

> SOC 2 Type II · Trust Service Criteria CC3 (Risk Assessment).
> Owner: compliance-officer + security-engineer. Review: annual (Q1) + on any material risk event or architecture change.
> Reference: `docs/SOC2_READINESS.md §14 (Risk Register)`, `docs/SOC2_READINESS.md §15 (Compliance Calendar)`, `docs/GDPR_DPIA.md`, `compliance/cc4/control-deficiency-log.csv`, `compliance/cc8/change-management-policy.md`.

---

## CC3 Criteria Coverage

| Criterion | Description | Status |
|---|---|---|
| CC3.1 | The entity specifies objectives with sufficient clarity to enable the identification and assessment of risks relating to objectives. | 🟢 Done — Objectives specified across four dimensions: Availability (99.9% SLA in `docs/ENTERPRISE_SLA.md`), Security (CIA triad in `docs/SECURITY.md`), Privacy (data minimisation + health data protections in `docs/PRIVACY_POLICY.md` and `docs/GDPR_DPIA.md`), Processing Integrity (CV accuracy + coaching output completeness in `docs/PRODUCT_SPEC.md`). Formal OKR structure in `docs/OKRS_2026.md` anchors Security and Compliance objectives. |
| CC3.2 | The entity identifies risks to the achievement of its objectives across the entity and analyzes risks as a basis for determining how the risks should be managed. | 🟡 Partial — 18 risks formally registered in `docs/SOC2_READINESS.md §14` across 6 categories (Authentication, Data, AI, Regulatory/Privacy, Availability, Confidentiality). Inherent and residual scoring on 1–5 Likelihood × Severity matrix. **Gap:** first formal annual review not yet executed (scheduled Q3 2026 in `docs/SOC2_READINESS.md §15`). |
| CC3.3 | The entity considers the potential for fraud in assessing risks to the achievement of objectives. | 🟡 Partial — Insider threat explicitly modelled (R-20 in `docs/INCIDENT_RESPONSE.md`; CR-01 RLS bypass in §14 risk register). Two-person break-glass authorization (DEC-030 enforced). HMAC-chained audit log (tamper detection). Segregation of duties documented as compensating control. **Gap:** no standalone fraud risk assessment section in risk register; CC3-GAP-002. |
| CC3.4 | The entity identifies and assesses changes that could significantly impact the system of internal control. | 🟢 Done — Change impact assessment integrated into `compliance/cc8/change-management-policy.md` (POL-011 §4): significant architecture changes require Security Review tag + compliance-officer notification. `docs/DECISION_LOG.md` provides auditor-legible record of all material architecture decisions (DEC-001 through DEC-030+). Emergency change process requires retroactive 24h post-hoc review. |

---

## Evidence Files

| File | Description | SOC 2 Criteria | Collection Cadence | Status |
|---|---|---|---|---|
| `docs/SOC2_READINESS.md §14` | **Formal Risk Register** — 18 risks across 6 categories (AR: Authentication, DR: Data, AIR: AI/ML, RR: Regulatory/Privacy, VR: Vendor, CR: Confidentiality/Integrity). Each risk: unique ID, description, Likelihood (1–5), Severity (1–5), inherent score, current mitigations, residual score, owner, status. | CC3.1, CC3.2, CC3.3 | Quarterly review (§15 compliance calendar); annual formal re-score (Q1); triggered on any HIGH residual score change | 🟢 Authored — 18 risks; first annual re-score Q1 2027 |
| `docs/OKRS_2026.md` | Organisational objectives for 2026 — Security & Compliance OKR cluster anchors CC3.1 objective specification. Key Results include SOC 2 Type II audit commencement, 99.9% uptime SLA, GDPR DPA before pilot, zero critical-unpatched-CVE-in-production policy. | CC3.1 | Annual OKR cycle; quarterly review | 🟢 Authored |
| `docs/ENTERPRISE_SLA.md` | Enterprise SLA — 99.9% monthly uptime objective; P0 < 1h / P1 < 4h / P2 < 24h response objectives; credit mechanism for SLA breach. Provides CC3.1 precision for availability risk scoping. | CC3.1 | Annual review; update on contract structure change | 🟢 Authored — IN_FORCE |
| `docs/PRODUCT_SPEC.md` | Product specification — CV accuracy requirements, coaching output completeness guardrails, clinical-safety veto power, fallback behavior on AI unavailability. Specifies processing integrity objectives for CC3.1 risk scoping. | CC3.1 | Update on major product change | 🟢 Authored |
| `docs/GDPR_DPIA.md` | Data Protection Impact Assessment (GDPR Art. 35) for health-category processing. Risk identification and mitigation analysis for Art. 9 special-category data processing. Feeds into CC3.2 regulatory risk register (RR-01 through RR-05). | CC3.2 | Update when new data category is introduced or processing purpose changes; annual review | 🟢 Authored |
| `compliance/cc4/control-deficiency-log.csv` | Control deficiency register — structured log of known control gaps with severity, root cause, owner, target remediation date, and closure date. All pre-launch deficiencies seeded; active during observation period. | CC3.2, CC3.4 | Updated on each new deficiency identification or remediation closure | 🟢 Active |
| `compliance/cc7/vuln-management-policy.md` | Vulnerability Management & Patching SLA Policy v1.0 (POL-010) — CVSS scoring with FORM-specific uplift rules (multiplier for enterprise tenant data exposure), binding SLAs (Critical 24h / High 7d / Medium 30d / Low 90d). Operationalises CC3.2 vulnerability risk management. | CC3.2 | Annual review; update on CVSS schema change | 🟢 Authored — IN_FORCE |
| `compliance/cc8/change-management-policy.md` | Change Management Policy (POL-011) — PR-based change control, CI gates, Security Review tag trigger, emergency change process, rollback capability. §4 requires compliance-officer notification for architecture changes affecting data flows or trust boundaries. | CC3.4 | Annual review | 🟢 Authored — IN_FORCE |
| `docs/DECISION_LOG.md` | Architecture Decision Record log — DEC-001 through DEC-030+. Auditor-legible record of all material architecture and security decisions: rationale, options considered, constraints, compliance impact. | CC3.4 | Updated on each material architecture decision | 🟢 Active |
| `docs/INCIDENT_RESPONSE.md §R-20` | Insider Threat runbook — covers rogue or compromised internal actors with legitimate credentials (form_admin, PAM service_role, Cloudflare Access admin). Risk model: access scope abuse, data exfiltration, audit log tampering. Two-person break-glass authorization, DEC-030 HMAC chain. Supports CC3.3 fraud risk consideration. | CC3.3 | Annual review; trigger on each break-glass event or elevated-access revocation | 🟢 Authored |
| `docs/AUDIT_LOG_SCHEMA.md` | DEC-030 audit log schema — HMAC-SHA256 chain; append-only; tamper detection via chain break; 7-year retention for CRITICAL events. Provides the fraud-detection infrastructure underlying CC3.3 controls. | CC3.3 | Reviewed on schema change; HMAC chain verified weekly (automated cron) | 🟢 Authored |
| `docs/SSO_SCIM_IMPLEMENTATION.md §24` | Privileged Access Management (PAM) architecture — JIT escalation, two-person authorization for break-glass, 4-hour time-bound role, DEC-030 emission on every escalation and action. Supports CC3.3 fraud risk (prevents single-person abuse of privileged access). | CC3.3 | Update on PAM architecture change | 🟢 Authored |
| `docs/SOC2_READINESS.md §22` | Compensating control acceptances — formal documentation of single-person SOD gaps at pre-hire stage with management attestation. SOC 2 auditors will review this section. | CC3.3 | Update on each new hire; compensating controls expire per role | 🟢 Authored |
| _(pending)_ `compliance/cc3/risk-register-review-2026-Q3.md` | First formal risk register review — re-score all 18 risks in §14; add new risks identified since initial authoring; confirm HIGH-residual risks have current mitigations; sign-off by compliance-officer + security-engineer. | CC3.2 | Annual (Q1 steady state); first execution Q3 2026 per SOC2_READINESS.md §15 | 🔴 First execution Q3 2026 (CC3-GAP-001) |
| _(pending)_ `compliance/cc3/fraud-risk-assessment.md` | Standalone fraud risk assessment section covering: misappropriation of assets (health data), fraudulent reporting (wellness aggregates), management override of privacy floor controls, vendor/supply-chain fraud. Supplements R-20 and addresses CC3.3 explicitly in risk register format. | CC3.3 | Annual (Q1); triggered on any detected internal control bypass | 🔴 Not yet authored (CC3-GAP-002) |
| _(pending)_ `compliance/cc3/risk-register-review-2027-Q1.md` | Annual steady-state risk register review for Q1 2027 — template identical to 2026-Q3 initial review. | CC3.2 | Annual (Q1) | 🔴 Template pending first execution |

---

## Risk Register Summary (as of 2026-06-07)

Reference: `docs/SOC2_READINESS.md §14`. 18 risks across 6 categories.

| Category | Risk IDs | Highest Residual | Count |
|---|---|---|---|
| Authentication & Access (AR) | AR-01 through AR-04 | AR-03 (MFA bypass) · 9 MEDIUM | 4 |
| Data & Privacy (DR) | DR-01 through DR-04 | DR-01 (RLS bypass) · 9 MEDIUM | 4 |
| AI/ML Integrity (AIR) | AIR-01 through AIR-02 | AIR-01 (adversarial prompt injection) · 8 MEDIUM | 2 |
| Regulatory & Privacy (RR) | RR-01 through RR-05 | RR-01 (GDPR consent failure) · 6 MEDIUM | 5 |
| Vendor (VR) | VR-01 through VR-02 | VR-02 (Sentry DPA gap) · 6 MEDIUM | 2 |
| Confidentiality & Integrity (CR) | CR-01 through CR-02 | CR-01 (RLS cross-tenant) · 3 LOW | 2 |

**No risks at HIGH (≥ 12) or CRITICAL (≥ 16) residual score.** Highest residual: 9 MEDIUM (three risks at this level: AR-03, DR-01, AIR-01). All HIGH-inherent risks have documented mitigations that reduce residual to MEDIUM or LOW.

---

## Risk Review Cadence

Per `docs/SOC2_READINESS.md §15` compliance calendar:

| Event | Frequency | Owner | Evidence Artifact |
|---|---|---|---|
| Quarterly risk register check-in | Quarterly (Q1–Q4) | compliance-officer | Linear ticket `[COMPLIANCE] Quarterly risk register check-in YYYY-QN` + any new risk entries in §14 |
| Annual formal risk re-score | Annual (Q1) | compliance-officer + security-engineer | `compliance/cc3/risk-register-review-YYYY-Q1.md` |
| First formal review (inaugural) | Q3 2026 (one-time) | compliance-officer + security-engineer | `compliance/cc3/risk-register-review-2026-Q3.md` |
| Ad-hoc: new HIGH-residual risk | On identification | security-engineer | New entry in §14 + Linear `[COMPLIANCE]` ticket |
| Ad-hoc: architecture change with trust-boundary impact | On each change | compliance-officer (notified) | DEC-### in `docs/DECISION_LOG.md` + CC8 change record |

---

## Fraud Risk Considerations (CC3.3)

CC3.3 requires the entity to consider fraud risk explicitly. FORM's fraud risk surface, given the product (AI health coaching with biometric data), includes:

| Fraud Risk Area | Current Control | Status |
|---|---|---|
| **Health data misappropriation** — internal actor exfiltrates workout, biometric, or coaching data for personal gain or sale | R-20 (Insider Threat runbook); DEC-030 HMAC audit chain; break-glass two-person auth (PAM §24); `audit_log_events.action = 'admin.data_export'` alert | 🟡 Controlled — break-glass API not yet implemented (OQ-INS-01 in INCIDENT_RESPONSE.md); interim: WorkOS magic-link manual path |
| **Aggregate wellness report manipulation** — admin inflates engagement metrics to justify contract renewal | Privacy floor enforced at RLS layer (not application layer); aggregation queries audited; `n=15` anonymity threshold hardcoded | 🟢 Controlled — RLS and hardcoded thresholds prevent manipulation without a schema change, which would require CI approval |
| **Privacy floor bypass** — enterprise customer or internal actor attempts to surface individual user data through admin dashboard | Admin API returns only aggregated data; RLS policy blocks per-user health queries from admin role; `is_admin` flag not a health data bypass (enforced at API layer) | 🟢 Controlled — tested in cross-tenant CI suite |
| **Management override** — founder/future executive overrides a privacy or security control without audit trail | DEC-030 HMAC chain records all schema migrations and control changes; git commit history; break-glass two-person authorization | 🟡 Partial — HMAC chain is the primary detective control; preventive controls are limited at solo-founder stage (compensating control accepted per §22) |
| **Vendor fraud / supply-chain compromise** — sub-processor or CI dependency introduces malicious code or data siphon | Dependency pinning + `npm audit` CI gate; SOC 2 reports required from Critical-tier vendors; CC8 change management controls any new dependency introduction | 🟡 Partial — CI gate exists; `npm audit` not yet enforced as hard-fail (CC3-GAP-003) |

A standalone **fraud risk assessment document** (`compliance/cc3/fraud-risk-assessment.md`) that maps each risk area to a formal L×S score and mitigation is planned as CC3-GAP-002.

---

## Gap Status

| Gap ID | Item | Priority | Owner | Status |
|---|---|---|---|---|
| **CC3-GAP-001** | First formal risk register review (`compliance/cc3/risk-register-review-2026-Q3.md`) | P1 — required for SOC 2 observation period; scheduled Q3 2026 per `docs/SOC2_READINESS.md §15` Row 27 | compliance-officer + security-engineer | 🔴 Open — scheduled; not yet executed |
| **CC3-GAP-002** | Standalone fraud risk assessment (`compliance/cc3/fraud-risk-assessment.md`) | P2 — CC3.3 explicit requirement; R-20 and break-glass controls exist but are not consolidated into a CC3.3-format risk assessment | compliance-officer | 🔴 Not yet authored — target Q1 2027 |
| **CC3-GAP-003** | `npm audit` enforced as hard-fail in CI (supply-chain risk) | P1 — currently advisory; hard-fail blocks compromise of CC3.4 change assessment | platform-engineer | 🔴 Open — `compliance/cc7/vuln-management-policy.md` §4.2 references this; implementation pending |
| **CC3-GAP-004** | Quarterly risk register check-in executed and logged | P1 — first quarterly check-in must occur before SOC 2 observation end | compliance-officer | 🔴 Open — first instance Q3 2026 |

---

## SOC 2 Auditor Notes

**Controls this section evidences:**
- CC3.1: Objective specification — `docs/OKRS_2026.md`, `docs/ENTERPRISE_SLA.md`, `docs/PRODUCT_SPEC.md`, `docs/GDPR_DPIA.md`
- CC3.2: Risk identification and analysis — `docs/SOC2_READINESS.md §14` (18 risks), `docs/GDPR_DPIA.md`, `compliance/cc4/control-deficiency-log.csv`
- CC3.3: Fraud risk consideration — `docs/INCIDENT_RESPONSE.md R-20`, `docs/AUDIT_LOG_SCHEMA.md`, `docs/SSO_SCIM_IMPLEMENTATION.md §24`, fraud risk table above
- CC3.4: Change impact assessment — `compliance/cc8/change-management-policy.md §4`, `docs/DECISION_LOG.md`

**Evidence gaps that prevent 🟢 status:**
1. First formal risk register review not yet executed (CC3-GAP-001) — this is the primary gap preventing CC3.2 from 🟢
2. Standalone fraud risk assessment not yet authored (CC3-GAP-002) — prevents CC3.3 from 🟢
3. Quarterly review cadence not yet initiated (CC3-GAP-004) — prevents auditor evidence of ongoing review

**Auditor expectation at Type I stage:** Point-in-time documentation of risk assessment design. Gaps above are acceptable for Type I if compensating controls are attested. For Type II (observation period), CC3-GAP-001 must be closed before observation start.

---

*Owner: compliance-officer + security-engineer · Created: 2026-06-07 · Next review: Q3 2026 (first formal risk register review)*
