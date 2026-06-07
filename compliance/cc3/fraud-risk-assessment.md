# FORM · CC3.3 Fraud Risk Assessment

> **Document ID:** CC3-FRA-001  
> **SOC 2 Criterion:** CC3.3 — The entity considers the potential for fraud in assessing risks to the achievement of objectives.  
> **Owner:** compliance-officer (primary) · security-engineer (technical review)  
> **Effective date:** 2026-06-07  
> **Next review:** 2027-Q1 (annual) · triggered earlier on: any detected internal control bypass, any new privileged role created, any material architecture change affecting access boundaries  
> **Status:** IN_FORCE — closes CC3-GAP-002  
> **Related documents:** `docs/SOC2_READINESS.md §14`, `docs/INCIDENT_RESPONSE.md R-20`, `docs/AUDIT_LOG_SCHEMA.md`, `docs/SSO_SCIM_IMPLEMENTATION.md §24`, `compliance/cc4/control-deficiency-log.csv`

---

## 1. Purpose and Scope

This document provides the standalone fraud risk assessment required by SOC 2 Type II CC3.3. It supplements the formal risk register at `docs/SOC2_READINESS.md §14`, which covers fraud scenarios within a broader risk taxonomy but does not apply explicit CC3.3 framing.

**What this document adds:**
- Explicit enumeration of fraud scenarios using CC3.3 categories (misappropriation of assets, fraudulent reporting, management override, vendor/supply-chain fraud)
- Fraud-specific Likelihood × Severity scoring aligned with the §14 methodology
- Explicit separation of **preventive** vs. **detective** controls per COSO 2013 fraud risk guidance
- Named fraud risk owners distinct from general operational risk owners
- Annual review and attestation mechanism

**Scope:** All FORM systems, personnel, contractors, and sub-processors as of the effective date. FORM is a pre-launch SaaS product with zero employees (solo-founder stage); all references to "personnel" apply to the founder and any contractors onboarded before the next annual review.

**Not in scope:** External threat actors (credential stuffing, API exploitation) — these are covered as Security Risks SR-01 through SR-05 in §14. External fraud means threat actors with no legitimate access to FORM systems.

---

## 2. Fraud Risk Framework

### 2.1 Scoring Methodology

Consistent with `docs/SOC2_READINESS.md §14.1`:

| Dimension | 1 | 2 | 3 | 4 | 5 |
|---|---|---|---|---|---|
| **Likelihood** | Rare (<once/5yr) | Unlikely (<once/yr) | Possible (1×/yr) | Likely (several/yr) | Almost Certain (monthly+) |
| **Impact** | Negligible | Minor | Moderate (user-visible, recoverable) | Major (SLA breach, data exposure) | Catastrophic (breach, regulatory, existential) |

Score = L × I. Rating: 1–5 LOW · 6–11 MEDIUM · 12–19 HIGH · 20–25 CRITICAL.

**Fraud-specific impact amplifier:** any fraud involving GDPR Art. 9 special-category health data automatically receives +1 to Impact before scoring. FORM's primary asset is user health data — misuse carries regulatory and reputational consequence beyond typical SaaS breach.

### 2.2 CC3.3 Fraud Categories

COSO 2013 and SOC 2 CC3.3 guidance identifies the following fraud risk categories. Each is assessed below.

| Category | COSO Label | Applies to FORM? |
|---|---|---|
| Misappropriation of assets | Physical / digital asset theft by insiders | Yes — health data is the primary asset |
| Fraudulent financial reporting | Manipulating financial statements | Limited — pre-revenue, no external financial reporting obligation yet |
| Fraudulent performance reporting | Manipulating KPIs, product metrics, AI output | Yes — wellness aggregates, CV accuracy claims |
| Management override of controls | Bypassing designed controls | Yes — founder has full admin access |
| Vendor / supply-chain fraud | Sub-processor misuse or malicious dependency | Yes — Anthropic, Supabase, Sentry, WorkOS |

---

## 3. Fraud Risk Register

### FR-01 — Health Data Misappropriation (Insider)

| Field | Detail |
|---|---|
| **Scenario** | A privileged insider (founder, future employee, or contractor with production access) accesses and exfiltrates user health data — workout logs, body metrics, AI coaching sessions — for personal gain, to sell to third parties, or to leverage against users. |
| **CC3.3 Category** | Misappropriation of assets |
| **Related §14 Risks** | SR-03 (Insider threat), CR-01 (Cross-tenant RLS bypass) |
| **Inherent Likelihood** | 1 — Rare (solo founder; currently no employees or contractors with independent production access) |
| **Inherent Impact** | 5 → **6** (5 Catastrophic + health-data amplifier) |
| **Inherent Score** | 6 LOW (1 × 6) |
| **Preventive Controls** | RLS fail-closed on all Supabase reads; `tenant_id` sourced from verified JWT claims only; RBAC with least privilege; break-glass dual-authorization (DEC-030 §24 of SSO doc) requires two-person authorization for any service-role-level operation; `docs/ONBOARDING_SECURITY.md` — AUP acknowledgment before any access granted |
| **Detective Controls** | HMAC-chained audit log (DEC-030) — every `data.read_individual`, `data.export_initiated`, `admin.break_glass_initiated` event emitted and chained; chain break detected by daily cron (`audit-chain-daily-check`); `#security-alerts` Slack channel alerts on audit chain break or `tenant_id_missing` counter >0; quarterly access review (`compliance/access-review/`) verifies no unauthorized access grants; 4-hour time-bound break-glass role automatically expires |
| **Residual Score** | **3 LOW** |
| **Fraud Risk Owner** | compliance-officer (founder self-attestation at pre-hire stage) |
| **Status** | Mitigated — controls documented; detection is automated; dual-auth enforced by architecture |

---

### FR-02 — Fraudulent Workout / Progress Reporting

| Field | Detail |
|---|---|
| **Scenario** | Internal actor manipulates backend data to falsify workout completion records, CV-detected rep counts, or wellness aggregate metrics. Could occur to: (a) inflate product performance claims for investor/enterprise demos; (b) suppress negative indicators before a funding round; (c) manipulate cohort data used in marketing claims. |
| **CC3.3 Category** | Fraudulent performance reporting |
| **Related §14 Risks** | IR-02 (Cross-tenant data corruption), CR-01 (RLS bypass) |
| **Inherent Likelihood** | 1 — Rare (pre-revenue; no investor pressure that typically drives this behavior; founder commitment to "no fabricated metrics" principle — see README §Principles) |
| **Inherent Impact** | 4 → **5** (Major reputational/regulatory harm + health-data amplifier if metrics derive from body data) |
| **Inherent Score** | 5 LOW (1 × 5) |
| **Preventive Controls** | Audit log schema (DEC-030) captures all write operations to `workouts`, `sets`, `health_metrics` tables; server-side CV pipeline generates rep counts — client cannot submit rep counts without corresponding pose-estimation metadata; RLS prevents cross-tenant writes; schema migration policy (CC8) requires PR review before any data-model change |
| **Detective Controls** | HMAC-chained audit log — any `data.write` or `data.delete` to health tables is chained; row-count-monitor cron (`*/15 * * * *`) detects anomalous bulk writes or deletes (OBSERVABILITY.md §6.3); audit log is append-only — deletion is architecturally impossible without breaking the HMAC chain (detected within 24h) |
| **Residual Score** | **2 LOW** |
| **Fraud Risk Owner** | product-manager (data integrity); compliance-officer (reporting integrity) |
| **Status** | Mitigated — append-only audit log and server-side CV validation prevent silent manipulation |

---

### FR-03 — Management Override of Privacy Floor Controls

| Field | Detail |
|---|---|
| **Scenario** | The founder (sole admin) bypasses the `clinical-safety` VETO protocol, disables RLS policies, removes data minimization constraints, or modifies the HMAC audit log schema to cover tracks. This is the highest-severity internal fraud scenario because the founder has superuser access and no peer review at solo-founder stage. |
| **CC3.3 Category** | Management override of internal controls |
| **Related §14 Risks** | SR-03 (Insider threat), CR-02 (Health data in logs), PR-01 (Consent failure) |
| **Inherent Likelihood** | 1 — Rare (founder has strong incentive to maintain controls; FORM's brand and compliance posture depend on them) |
| **Inherent Impact** | 5 → **6** (Catastrophic — GDPR enforcement, loss of enterprise trust + health-data amplifier) |
| **Inherent Score** | 6 LOW (1 × 6) |
| **Preventive Controls** | Break-glass dual-authorization (DEC-030 §24) — even the founder cannot operate at `service_role` level without a documented second-party authorization (compensating control: emergency-access contact defined in 1Password vault; any break-glass event generates audit event FR-BREAKGLASS-001); all RLS policies version-controlled in GitHub (any change requires a PR commit — constitutes immutable evidence); clinical-safety VETO is enforced by the agent pipeline, not by a runtime switch; `docs/DECISION_LOG.md` is public and append-only — DEC entries cannot be deleted without a visible git revert (CC8 change management) |
| **Detective Controls** | HMAC-chained audit log — if audit log schema is modified to suppress events, the chain breaks on the next daily check; GitHub commit history is immutable (any force-push would require GitHub admin access and would be visible in `git reflog`); advisory board (docs/ADVISORY_BOARD.md) receives quarterly updates — anomalous operational behavior would be visible in investor updates; Cloudflare Access logs all admin-panel accesses with timestamp and account identity |
| **Residual Score** | **3 LOW** |
| **Fraud Risk Owner** | founder (self-attestation); advisory-board (external check) |
| **Status** | Mitigated — compensating controls accepted per `docs/SOC2_READINESS.md §22` (solo-founder SOD compensating control narrative). Post-first-hire: a second engineer must co-authorize break-glass operations, eliminating the compensating-control dependency. |
| **Compensating Control Note** | At solo-founder stage, true segregation of duties is architecturally impossible. The compensating control package is: (1) public GitHub repository — all code changes visible; (2) immutable HMAC audit log; (3) break-glass dual-auth requiring documented emergency-access contact; (4) advisory board + investor quarterly update. This package is documented and accepted in SOC2_READINESS.md §22 and will be superseded by peer review upon first engineering hire. |

---

### FR-04 — Vendor / Sub-Processor Data Misuse

| Field | Detail |
|---|---|
| **Scenario** | A contracted sub-processor (Anthropic, Supabase, Sentry, WorkOS, ElevenLabs, PostHog, Cloudflare, Stripe) uses FORM user health data beyond the scope of their DPA, trains models on user data without consent, or shares data with third parties in violation of GDPR Art. 28 obligations. |
| **CC3.3 Category** | Vendor / supply-chain fraud |
| **Related §14 Risks** | VR-02 (Sentry DPA), CR-02 (Health data in logs) |
| **Inherent Likelihood** | 2 — Unlikely (all Critical-tier vendors publish SOC 2 reports; GDPR penalties create strong contractual deterrent) |
| **Inherent Impact** | 4 → **5** (Major — GDPR Art. 83(4)/(5) penalties apply; user trust destroyed + health-data amplifier) |
| **Inherent Score** | 10 MEDIUM (2 × 5) |
| **Preventive Controls** | DPAs executed with 6 of 8 sub-processors (Sentry DPA in progress — VR-02); GDPR Standard Contractual Clauses (SCCs) for EU data transfers; data minimization: AI inference logs store token counts and latency only — no prompt or response content transmitted to Anthropic beyond the API call (OBSERVABILITY.md §4.5); Sentry `beforeSend` hook strips all health data fields before crash reports leave the device; Anthropic model training opt-out confirmed in API agreement; sub-processor disclosure list published (`docs/SUBPROCESSORS.md`); 30-day advance notice required before adding new sub-processors |
| **Detective Controls** | Annual vendor security review programme (§17 of SOC2_READINESS.md) — SOC 2 Type II reports from Critical-tier vendors reviewed annually; security questionnaire issued to Standard-tier vendors; `FORM-HEALTH-LEAK-001` Sentry alert fires if health-field keys detected in crash payloads; DPA register (`compliance/policy-approval-log.csv`) tracks DPA execution dates and expiry — gap triggers compliance-officer escalation |
| **Residual Score** | **6 MEDIUM** (primary open gap: Sentry DPA pending — VR-02) |
| **Fraud Risk Owner** | compliance-officer |
| **Status** | Partially mitigated — residual MEDIUM until Sentry DPA executed (VR-02 target: Month O-5 per pre-launch checklist §15.2). All other Critical-tier vendors have executed DPAs. |

---

### FR-05 — Supply-Chain Compromise via Malicious Dependency

| Field | Detail |
|---|---|
| **Scenario** | A malicious npm package, Cloudflare Worker dependency, or CI pipeline step is introduced — either via a typosquatting attack, a compromised maintainer account, or deliberate insider introduction — that exfiltrates health data or injects fraudulent coaching output. |
| **CC3.3 Category** | Vendor / supply-chain fraud |
| **Related §14 Risks** | SR-04 (Supply chain attack) |
| **Inherent Likelihood** | 2 — Unlikely (targeted supply-chain attacks against pre-launch SaaS are rare; FORM is not yet a high-value target) |
| **Inherent Impact** | 5 → **6** (Catastrophic — backdoor in health data pipeline is existential + health-data amplifier) |
| **Inherent Score** | 12 HIGH (2 × 6) |
| **Preventive Controls** | `package-lock.json` pinned (no floating dependencies); Dependabot enabled for automated dependency updates with PR-based review; `npm audit` in CI (advisory; hard-fail gate pending — CC3-GAP-003); Change Management Policy (CC8) requires PR review for any new dependency introduction; `git-secrets` pre-commit hook; GitHub secret scanning; no external Cloudflare Workers fetch beyond explicitly listed allowlist |
| **Detective Controls** | Dependabot alerts route to `#security-alerts`; `npm audit --audit-level=critical` CI report visible in every PR; HMAC audit log would detect anomalous data egress patterns (unexpected `data.export_*` events); pen test programme (SOC2_READINESS.md §16) includes dependency review |
| **Residual Score** | **5 MEDIUM** (hard-fail `npm audit` gate not yet enforced — CC3-GAP-003; reduces to LOW on CI gate implementation) |
| **Fraud Risk Owner** | devops-lead |
| **Status** | Partially mitigated — `npm audit` advisory in CI; hard-fail gate is the open remediation item (CC3-GAP-003). Compensating control: `package-lock.json` pinning limits new malicious versions from entering without a PR. |

---

### FR-06 — Billing / Subscription Fraud

| Field | Detail |
|---|---|
| **Scenario** | An insider modifies subscription status in FORM's database to grant enterprise features to non-paying customers, or to suppress churn signals for reporting purposes. |
| **CC3.3 Category** | Misappropriation of assets (revenue); fraudulent performance reporting |
| **Related §14 Risks** | Not previously in §14 risk register — new risk identified in this fraud assessment |
| **Inherent Likelihood** | 1 — Rare (pre-revenue; Stripe is the source of truth; local database subscription status is derived from Stripe webhooks, not independently managed) |
| **Inherent Impact** | 3 (Moderate — revenue impact; no health data involved) |
| **Inherent Score** | 3 LOW (1 × 3) |
| **Preventive Controls** | Stripe webhook signature validation (HMAC-SHA256) on every event; subscription status derived from Stripe, not from a locally mutable field without a corresponding Stripe event; no FORM admin UI grants subscription status changes outside Stripe dashboard; `billing.subscription_change` audit event emitted on every status transition |
| **Detective Controls** | `billing.subscription_change` events in HMAC audit log; Stripe dashboard provides independent ground truth; any divergence between Stripe status and FORM database is flagged by the nightly reconciliation job (planned — Phase 3 roadmap) |
| **Residual Score** | **2 LOW** |
| **Fraud Risk Owner** | founder (billing); compliance-officer (audit) |
| **Status** | Mitigated — Stripe as authoritative source eliminates insider-modification attack surface |

---

## 4. Fraud Risk Summary

| Risk ID | Scenario | Inherent Score | Residual Score | Status |
|---|---|---|---|---|
| FR-01 | Health data misappropriation (insider) | 6 LOW | **3 LOW** | Mitigated |
| FR-02 | Fraudulent workout/progress reporting | 5 LOW | **2 LOW** | Mitigated |
| FR-03 | Management override of privacy floor controls | 6 LOW | **3 LOW** | Mitigated (compensating controls per §22) |
| FR-04 | Vendor / sub-processor data misuse | 10 MEDIUM | **6 MEDIUM** | Partially mitigated — Sentry DPA pending (VR-02) |
| FR-05 | Supply-chain compromise via malicious dependency | 12 HIGH | **5 MEDIUM** | Partially mitigated — CI hard-fail gate pending (CC3-GAP-003) |
| FR-06 | Billing / subscription fraud | 3 LOW | **2 LOW** | Mitigated |

**Highest fraud residual risk:** FR-05 (5 MEDIUM) and FR-04 (6 MEDIUM). Both have remediation paths tracked in the deficiency log.

**No CRITICAL or HIGH residual fraud risks exist.** The highest inherent fraud risk is FR-05 (12 HIGH inherent), which is reduced to MEDIUM via `package-lock.json` pinning and Dependabot.

---

## 5. Relationship to §14 Risk Register

| FR ID | §14 Risk ID(s) | Delta |
|---|---|---|
| FR-01 | SR-03, CR-01 | No change to §14 scores; FR-01 provides CC3.3-explicit framing |
| FR-02 | IR-02, CR-01 | No change to §14 scores |
| FR-03 | SR-03, CR-02, PR-01 | No change to §14 scores; FR-03 adds management-override framing absent from §14 |
| FR-04 | VR-02, CR-02 | No change to §14 scores |
| FR-05 | SR-04 | No change to §14 scores |
| FR-06 | **New** — not previously in §14 | Add FR-06 to §14 as billing/subscription fraud risk at next quarterly review (LOW; does not change readiness scorecard) |

---

## 6. Gaps and Remediation

| Gap ID | Description | Severity | Target | Owner |
|---|---|---|---|---|
| CC3-GAP-002 | This document (fraud-risk-assessment.md) — **closed by this file** | — | 2026-06-07 ✅ | compliance-officer |
| CC3-GAP-003 | `npm audit --audit-level=critical` hard-fail CI gate not yet enforced (reduces FR-05 to LOW) | P1 | Month O-3 | devops-lead |
| VR-02 (cross-ref) | Sentry DPA not yet executed (reduces FR-04 to LOW) | P1 | Month O-5 | compliance-officer |

---

## 7. Annual Review and Attestation

This assessment must be reviewed annually (Q1) and any time:
- A new privileged role is created (engineer, contractor, admin)
- A material architecture change affects access boundaries or data flows
- A CC3.3-relevant incident is detected (internal control bypass, suspected insider threat)
- The sub-processor list changes (new vendor with health data access)

**Review checklist:**
- [ ] Re-score each FR risk given updated controls and threat landscape
- [ ] Add any new fraud scenarios identified since last review
- [ ] Confirm all open gaps from §6 are either closed or have updated remediation dates
- [ ] Confirm Sentry DPA executed (closes FR-04 open gap)
- [ ] Confirm CI hard-fail `npm audit` gate in place (closes FR-05 open gap)
- [ ] Cross-reference with §14 risk register quarterly review (`compliance/cc3/risk-register-review-*`)
- [ ] Founder / compliance-officer sign-off below

---

## 8. Sign-off

| Role | Name | Date | Signature |
|---|---|---|---|
| compliance-officer (primary) | [founder] | 2026-06-07 | _[founder signature on file]_ |
| security-engineer (technical review) | [founder] | 2026-06-07 | _[founder signature on file]_ |

_At the pre-hire solo-founder stage, the compliance-officer and security-engineer roles are held by the same individual. This is documented as a compensating control in `docs/SOC2_READINESS.md §22`. Upon first security hire, a separate security-engineer signature is required._

---

*CC3-GAP-002 closed · 2026-06-07 · v1.0 · append-only after effective date · next review Q1 2027*
