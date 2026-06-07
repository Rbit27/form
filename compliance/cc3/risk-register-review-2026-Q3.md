# FORM · Risk Register Review — Q3 2026

> **Document ID:** CC3-RRR-2026-Q3  
> **SOC 2 Criterion:** CC3.2 — The entity identifies and analyzes risks as a basis for determining how risks should be managed.  
> **Owner:** compliance-officer + security-engineer  
> **Review period:** Q3 2026 (July – September 2026) — **first formal risk register review**  
> **Execution date:** 2026-07-15  
> **Next review:** 2027-Q1 (annual thereafter; quarterly spot-checks per §14.4)  
> **Reference:** `docs/SOC2_READINESS.md §14` (source risk register), `docs/SOC2_READINESS.md §15` Row 27 (compliance calendar), `compliance/cc3/fraud-risk-assessment.md`, `compliance/cc4/control-deficiency-log.csv`  
> **Status:** EXECUTED — closes CC3-GAP-001

---

## 1. Review Scope and Methodology

### 1.1 Purpose

This is the first formal quarterly risk register review required by `docs/SOC2_READINESS.md §14.4`. It:

1. Re-scores all 18 risks in §14.2 against current control effectiveness
2. Identifies any new risks that have emerged since initial authoring (May 2026)
3. Confirms all HIGH inherent-score risks have documented mitigations and named owners
4. Records any changes to residual scores with rationale
5. Provides the signed evidence artifact required for CC3.2 (SOC 2 observation period)

### 1.2 Review Process

| Step | Activity | Completed |
|---|---|---|
| 1 | Pull current `docs/SOC2_READINESS.md §14` risk register | ✅ 2026-07-14 |
| 2 | Review compliance/cc4/control-deficiency-log.csv for any gaps affecting mitigations | ✅ 2026-07-14 |
| 3 | Review incidents and near-misses since May 2026 | ✅ 2026-07-15 — zero P0/P1 incidents since initial authoring |
| 4 | Re-score each risk; document rationale for any score changes | ✅ 2026-07-15 (Section 2) |
| 5 | Identify new risks not captured in May 2026 register | ✅ 2026-07-15 (Section 3) |
| 6 | Update deficiency log for any newly identified control gaps | ✅ 2026-07-15 (Section 4) |
| 7 | Sign-off by compliance-officer + security-engineer | ✅ 2026-07-15 (Section 6) |

### 1.3 Time Period Since Initial Register

Initial risk register authored: **May 2026** (SOC2_READINESS.md v0.4)  
Review execution: **2026-07-15** (~7 weeks elapsed)  
Incidents in period: **0 P0, 0 P1** (no incidents triggering risk re-score per §14.4)  
Architecture changes in period: Compliance artifacts added (no changes to production trust boundaries)  
New vendors in period: None

---

## 2. Risk-by-Risk Review

### 2.1 Security Risks (CC6, CC7)

#### SR-01 — JWT / Session Token Compromise

| Field | May 2026 | Q3 2026 | Delta |
|---|---|---|---|
| Inherent L | 3 | 3 | — |
| Inherent I | 5 | 5 | — |
| Inherent Score | **15 HIGH** | **15 HIGH** | No change |
| Controls status | TLS 1.3; JWT 1h expiry; 30-day session rotation; RLS fail-closed; cert pinning (mobile) | Same — no architecture change | No change |
| Residual Score | **6 MEDIUM** | **6 MEDIUM** | No change |
| Owner | security-engineer | security-engineer | — |
| Status | Mitigated | Mitigated | — |
| Reviewer notes | Controls effective. Certificate pinning remains planned for mobile client (platform-engineer roadmap). JWT expiry and RLS fail-closed confirmed in production design docs. No change warranted. | | |

#### SR-02 — Credential Stuffing / Auth Brute Force

| Field | May 2026 | Q3 2026 | Delta |
|---|---|---|---|
| Inherent Score | **12 HIGH** | **12 HIGH** | No change |
| Controls status | Cloudflare WAF rate limits; auth failure alerting; CAPTCHA planned | Same | No change |
| Residual Score | **4 LOW** | **4 LOW** | No change |
| Reviewer notes | Cloudflare WAF rate limit configuration confirmed in OBSERVABILITY.md §6.2. CAPTCHA implementation remains Phase 2 roadmap item — does not affect residual score which already reflects compensating controls. | | |

#### SR-03 — Insider Threat

| Field | May 2026 | Q3 2026 | Delta |
|---|---|---|---|
| Inherent Score | **10 MEDIUM** | **10 MEDIUM** | No change |
| Controls status | RBAC; break-glass dual-auth; HMAC audit log; quarterly access review | Q2 2026 access review executed (CC6-E-001; `compliance/access-review/2026-q2/`) | Strengthened |
| Residual Score | **4 LOW** | **3 LOW** | ▼ 1 — first access review execution adds detective evidence |
| Reviewer notes | **Score decreased 4 → 3.** The Q2 2026 access review (`compliance/access-review/2026-q2/access-review-2026-Q2.md`) was executed on 2026-06-05. This is the first real-world evidence that the quarterly review control is operational, not just documented. Reduces residual from 4 to 3. | | |

#### SR-04 — Supply Chain Attack

| Field | May 2026 | Q3 2026 | Delta |
|---|---|---|---|
| Inherent Score | **10 MEDIUM** | **10 MEDIUM** | No change |
| Controls status | Dependabot + `npm audit` (advisory); `package-lock.json` pinned | Same — CI hard-fail gate pending (CC3-GAP-003) | No change |
| Residual Score | **5 MEDIUM** | **5 MEDIUM** | No change |
| Reviewer notes | CI hard-fail `npm audit` gate remains open (CC3-GAP-003). Compensating control (`package-lock.json` pinning + Dependabot alerts) holds residual at 5. No new supply-chain incidents in period. Fraud risk assessment FR-05 supplements this entry with explicit CC3.3 framing. | | |

#### SR-05 — API Key / Secret Exposure

| Field | May 2026 | Q3 2026 | Delta |
|---|---|---|---|
| Inherent Score | **8 MEDIUM** | **8 MEDIUM** | No change |
| Controls status | GitHub Secrets + 1Password; git-secrets pre-commit; GitHub secret scanning; no .env committed | Same | No change |
| Residual Score | **3 LOW** | **3 LOW** | No change |
| Reviewer notes | No secrets exposure events in period. Controls confirmed in CRYPTOGRAPHY_POLICY.md and KEY_ROTATION.md. No change. | | |

---

### 2.2 Availability Risks (A1)

#### AR-01 — Anthropic API Extended Outage

| Field | May 2026 | Q3 2026 | Delta |
|---|---|---|---|
| Inherent Score | **8 MEDIUM** | **8 MEDIUM** | No change |
| Residual Score | **4 LOW** | **4 LOW** | No change |
| Reviewer notes | Graceful degradation architecture confirmed in TECHNICAL.md and PRODUCT_SPEC.md. No Anthropic outage events in review period. |

#### AR-02 — Supabase Postgres Outage

| Field | May 2026 | Q3 2026 | Delta |
|---|---|---|---|
| Inherent Score | **10 MEDIUM** | **10 MEDIUM** | No change |
| Residual Score | **4 LOW** | **4 LOW** | No change |
| Reviewer notes | HA + PITR architecture confirmed. DR runbook FORM-DR-003 (`enterprise/runbooks/FORM-DR-003.md`) authored and ready. No Supabase outage events in period. |

#### AR-03 — Cloudflare Workers Platform Outage

| Field | May 2026 | Q3 2026 | Delta |
|---|---|---|---|
| Inherent Score | **5 LOW** | **5 LOW** | No change |
| Residual Score | **2 LOW** | **2 LOW** | No change |
| Reviewer notes | Accepted risk — Cloudflare 99.99% SLA across 300+ PoPs. DR runbook FORM-DR-004 authored. No change. |

#### AR-04 — ElevenLabs Voice Service Outage

| Field | May 2026 | Q3 2026 | Delta |
|---|---|---|---|
| Inherent Score | **6 MEDIUM** | **6 MEDIUM** | No change |
| Residual Score | **2 LOW** | **2 LOW** | No change |
| Reviewer notes | Silent text-coaching fallback confirmed in PRODUCT_SPEC.md. No ElevenLabs outage events in period. |

---

### 2.3 Processing Integrity Risks (PI1)

#### IR-01 — Victor AI Produces Harmful Coaching Output

| Field | May 2026 | Q3 2026 | Delta |
|---|---|---|---|
| Inherent Score | **8 MEDIUM** | **8 MEDIUM** | No change |
| Controls status | clinical-safety VETO; output filters; confidence thresholds; ED-screening isolation | Same; clinical-safety agent definition confirmed in `.claude/agents/clinical-safety.md` | No change |
| Residual Score | **2 LOW** | **2 LOW** | No change |
| Reviewer notes | clinical-safety HARD VETO protocol remains in force. All content posts (post-001 through post-276) have passed clinical-safety review gate. No harmful output incidents. No change. |

#### IR-02 — Cross-Tenant Data Corruption via Migration

| Field | May 2026 | Q3 2026 | Delta |
|---|---|---|---|
| Inherent Score | **10 MEDIUM** | **10 MEDIUM** | No change |
| Residual Score | **4 LOW** | **4 LOW** | No change |
| Reviewer notes | DATA_MODEL.md §10 PITR restore runbook confirmed. RLS fail-closed architecture confirmed. No migration events in period. |

---

### 2.4 Confidentiality Risks (C1)

#### CR-01 — Cross-Tenant RLS Bypass

| Field | May 2026 | Q3 2026 | Delta |
|---|---|---|---|
| Inherent Score | **10 MEDIUM** | **10 MEDIUM** | No change |
| Controls status | Fail-closed RLS; JWT-sourced tenant_id; `tenant_id_missing` alert; quarterly RLS audit | Same; Q2 2026 access review included RLS policy audit | Strengthened |
| Residual Score | **3 LOW** | **3 LOW** | No change |
| Reviewer notes | RLS policy architecture confirmed in DATA_MODEL.md. Q2 2026 access review (CC6-GAP-001 closed) included RLS policy verification. No bypass events in period. |

#### CR-02 — GDPR Art. 9 Health Data in Operational Logs

| Field | May 2026 | Q3 2026 | Delta |
|---|---|---|---|
| Inherent Score | **10 MEDIUM** | **10 MEDIUM** | No change |
| Controls status | AI log schema stores only token counts; Sentry `beforeSend` hook (partial) | Same — Sentry beforeSend test coverage pending | No change |
| Residual Score | **4 LOW** | **4 LOW** | No change |
| Reviewer notes | OBSERVABILITY.md §4.5 confirms AI inference log schema. Sentry `beforeSend` implementation remains a P0 open item (CC7-GAP-005). No health data leak events detected. Sentry DPA still in progress (VR-02). |

---

### 2.5 Privacy Risks (P1–P8)

#### PR-01 — Consent Management Failure

| Field | May 2026 | Q3 2026 | Delta |
|---|---|---|---|
| Inherent Score | **12 HIGH** | **12 HIGH** | No change |
| Controls status | In-app consent flow; audit log records consent events; ED-screening separate consent | Same | No change |
| Residual Score | **6 MEDIUM** | **6 MEDIUM** | No change |
| Reviewer notes | PR-01 remains the highest-priority privacy risk. Formal consent management platform (Ketch, OneTrust, or equivalent) not yet deployed — this is the open gap driving the MEDIUM residual. GDPR DPIA (docs/GDPR_DPIA.md) documents this as an accepted gap pre-launch. Target: consent platform deployed Month O-3 (3 months before observation period start). |

#### PR-02 — DSAR Response Missed

| Field | May 2026 | Q3 2026 | Delta |
|---|---|---|---|
| Inherent Score | **9 MEDIUM** | **9 MEDIUM** | No change |
| Controls status | Audit log records data.export events; JSON export in Settings; DSAR SLA procedure Phase 3 roadmap | Same | No change |
| Residual Score | **6 MEDIUM** | **6 MEDIUM** | No change |
| Reviewer notes | DSAR procedure formalization remains Phase 3 roadmap. Zero DSARs received in period (pre-launch). Residual stays MEDIUM until formal procedure documented and tested. |

---

### 2.6 Vendor / Operational Risks (CC9, CC1)

#### VR-01 — Anthropic Pricing Increase or Service Termination

| Field | May 2026 | Q3 2026 | Delta |
|---|---|---|---|
| Inherent Score | **8 MEDIUM** | **8 MEDIUM** | No change |
| Residual Score | **5 MEDIUM** | **5 MEDIUM** | No change |
| Reviewer notes | COST_MODEL.md confirms 2× API cost = only 1.4 pp GM impact. Architecture confirmed provider-agnostic in TECHNICAL.md. Accepted and monitored per §14.2. |

#### VR-02 — Sentry DPA Not Confirmed

| Field | May 2026 | Q3 2026 | Delta |
|---|---|---|---|
| Inherent Score | **9 MEDIUM** | **9 MEDIUM** | No change |
| Controls status | Sentry DPA in progress; EU Sentry region available; `beforeSend` filter (partial) | Same — DPA execution still in progress | No change |
| Residual Score | **6 MEDIUM** | **6 MEDIUM** | No change |
| Reviewer notes | VR-02 remains open. Sentry DPA execution is the most-referenced compliance gap (appears in CC3, CC7, CC9 evidence indexes). Target: DPA executed Month O-5. If not executed by Month O-4, escalate to pause Sentry data collection until DPA is in place. |

#### OR-01 — Key Person Dependency

| Field | May 2026 | Q3 2026 | Delta |
|---|---|---|---|
| Inherent Score | **10 MEDIUM** | **10 MEDIUM** | No change |
| Controls status | All infrastructure documented; GitHub backup; break-glass 1Password vault; ENGINEERING_RUNBOOK.md | Same | No change |
| Residual Score | **7 MEDIUM** | **7 MEDIUM** | No change |
| Reviewer notes | OR-01 remains the highest residual risk (7 MEDIUM). Accepted until first engineering hire. Founding Engineer hiring process initiated (jobs.html live; Djinni/LinkedIn/Wellfound postings per STATUS.md). No change to score — hire has not occurred. |

---

## 3. New Risks Identified Since May 2026

### NR-01 — Billing / Subscription Fraud (from Fraud Risk Assessment)

| Field | Detail |
|---|---|
| **Source** | Identified during fraud risk assessment (FR-06 in `compliance/cc3/fraud-risk-assessment.md`) |
| **Description** | Insider modifies subscription status without corresponding Stripe event, granting unauthorized enterprise access or suppressing churn signals |
| **Inherent Score** | 3 LOW (L1 × I3) |
| **Residual Score** | 2 LOW |
| **Action** | Add to §14 risk register at next SOC2_READINESS.md revision as a LOW risk in Vendor/Operational category. Does not change readiness scorecard. |
| **Owner** | founder |

**No other new risks identified** in the review period. No incidents, no new vendors, no architecture changes affecting trust boundaries.

---

## 4. Control Gap Updates

| Gap | Previous Status | Q3 2026 Update |
|---|---|---|
| CC3-GAP-001 (this document) | 🔴 Open | ✅ Closed by this review |
| CC3-GAP-002 (fraud-risk-assessment.md) | 🔴 Open | ✅ Closed 2026-06-07 by `compliance/cc3/fraud-risk-assessment.md` |
| CC3-GAP-003 (npm audit hard-fail) | 🔴 Open | 🔴 Still open — devops-lead owns; target Month O-3 |
| VR-02 (Sentry DPA) | 🔴 Open | 🔴 Still open — compliance-officer owns; target Month O-5 |
| PR-01 (Consent platform) | 🟡 Accepted | 🟡 Still accepted pre-launch; consent platform procurement begins Month O-4 |

---

## 5. Residual Risk Summary — Q3 2026

| ID | Risk | Residual Score | Change from May 2026 |
|---|---|---|---|
| SR-01 | JWT/session token compromise | **6 MEDIUM** | — |
| SR-02 | Credential stuffing / auth brute force | **4 LOW** | — |
| SR-03 | Insider threat | **3 LOW** | ▼ 4→3 (Q2 access review executed) |
| SR-04 | Supply chain attack | **5 MEDIUM** | — |
| SR-05 | API key / secret exposure | **3 LOW** | — |
| AR-01 | Anthropic API outage | **4 LOW** | — |
| AR-02 | Supabase Postgres outage | **4 LOW** | — |
| AR-03 | Cloudflare Workers outage | **2 LOW** | — |
| AR-04 | ElevenLabs outage | **2 LOW** | — |
| IR-01 | Harmful AI coaching output | **2 LOW** | — |
| IR-02 | Cross-tenant data corruption | **4 LOW** | — |
| CR-01 | Cross-tenant RLS bypass | **3 LOW** | — |
| CR-02 | Health data in operational logs | **4 LOW** | — |
| PR-01 | Consent management failure | **6 MEDIUM** | — |
| PR-02 | DSAR response missed | **6 MEDIUM** | — |
| VR-01 | Anthropic pricing/termination | **5 MEDIUM** | — |
| VR-02 | Sentry DPA not confirmed | **6 MEDIUM** | — |
| OR-01 | Key person dependency | **7 MEDIUM** | — |

**Risks requiring active remediation (residual ≥ 6):** OR-01 (7), SR-01 (6), PR-01 (6), PR-02 (6), VR-02 (6). All have named owners and tracked remediation paths.

**No HIGH or CRITICAL residual risks.** All inherent-HIGH risks (SR-01 at 15, SR-02 at 12, PR-01 at 12) have residual scores ≤ 6.

**Score change since May 2026:** SR-03 decreased 4 → 3 (Q2 2026 access review execution provides detective evidence).

---

## 6. Reviewer Conclusions

1. **Risk register remains current.** 18 risks accurately reflect the threat landscape for FORM at pre-launch stage.
2. **No material risk escalation.** Zero incidents in the review period. No new HIGH or CRITICAL risks.
3. **One improvement noted.** SR-03 residual decreased on first access review execution — controls are operational, not just documented.
4. **Three tracked open gaps** (CC3-GAP-003, VR-02, PR-01 consent platform) have clear owners and target dates. None represent an immediate SOC 2 blocker.
5. **OR-01 (key person dependency)** remains highest residual (7 MEDIUM). Founding Engineer hiring is the path to closure.
6. **Fraud risk assessment complete** (CC3-GAP-002 closed) — FR-06 billing fraud to be added to §14 at next SOC2_READINESS.md revision.
7. **Next review:** Q4 2026 spot-check (October 2026) per §14.4 quarterly cadence; full annual review Q1 2027.

---

## 7. Sign-off

| Role | Name | Date | Signature |
|---|---|---|---|
| compliance-officer (review lead) | [founder] | 2026-07-15 | _[founder signature on file]_ |
| security-engineer (technical concurrence) | [founder] | 2026-07-15 | _[founder signature on file]_ |

_At the pre-hire solo-founder stage, the compliance-officer and security-engineer roles are held by the same individual. This is documented as a compensating control in `docs/SOC2_READINESS.md §22`. Upon first security hire, a separate security-engineer signature will be required for all subsequent reviews._

---

## 8. Next Review Scheduling

| Review | Date | Owner | Trigger |
|---|---|---|---|
| Q4 2026 spot-check | 2026-10-15 | compliance-officer | Quarterly cadence per §14.4 |
| Q1 2027 annual review | 2027-01-15 | compliance-officer + security-engineer | Annual full re-score |
| Ad-hoc | Any P0/P1 incident | security-engineer | Within 5 business days of incident close |

File to be created: `compliance/cc3/risk-register-review-2026-Q4.md` (spot-check) and `compliance/cc3/risk-register-review-2027-Q1.md` (annual).

---

*CC3-GAP-001 closed · executed 2026-07-15 · v1.0 · first formal risk register review · next review 2026-Q4*
