# FORM · CC9 Risk Mitigation — Evidence Index

> SOC 2 Type II · Trust Service Criteria CC9 (Risk Mitigation).
> Owner: compliance-officer + devops-lead. Review: annual (January) + on any significant vendor change.
> Reference: `docs/SOC2_READINESS.md §17`, `docs/VENDOR_REGISTRY.md`, `docs/SUBPROCESSORS.md`, `docs/BUSINESS_CONTINUITY.md`.

---

## CC9 Criteria Coverage

| Criterion | Description | Status |
|---|---|---|
| CC9.1 | The entity identifies, selects, and develops risk mitigation activities for risks arising from potential business disruptions; considers the use of insurance, business continuity and disaster recovery plans, and vendor redundancy to mitigate identified risks | 🟡 Partial — Business Continuity Plan authored (`docs/BUSINESS_CONTINUITY.md`); DR runbooks FORM-DR-003–005 completed (v2.38.1); risk register §14 covers vendor/operational risks; first annual review Q1 2027 |
| CC9.2 | The entity assesses and manages risks associated with vendors and business partners; includes performance monitoring and review of relevant reports; includes impact on CC1.4 HR practices | 🟡 Partial — Vendor Registry authored (`docs/VENDOR_REGISTRY.md`); Sub-processor Register published (`docs/SUBPROCESSORS.md`); 5-step annual vendor security review programme defined (§17.5); DPAs in place for 6 of 8 sub-processors; Sentry DPA in progress; first full evidence cycle Q1 2027 |
| CC9.3 | The entity evaluates compliance with vendor obligations and, when applicable, whether contracts address the entity's service commitments and system requirements | 🟡 Partial — DPA review requirements defined (§17.3); SCCs in place for EU sub-processors; deletion confirmation workflow defined (§17.8); Sentry DPA pending (AR-2026-Q2-02 finding) |

---

## Evidence Files

| File | Description | SOC 2 Criteria | Collection Cadence | Status |
|---|---|---|---|---|
| `docs/VENDOR_REGISTRY.md` | Vendor risk registry — all sub-processors and service providers with tier classification (Critical/High/Standard), DPA status, SOC 2 cert status, data categories processed, annual review date | CC9.1, CC9.2, CC9.3 | Annual update; update on any vendor addition or removal | 🟢 Authored |
| `docs/SUBPROCESSORS.md` | Sub-processor disclosure list — published to enterprise customers; 30-day advance notice for additions; GDPR Art. 28 compliant | CC9.1, CC9.2 | Monthly currency check; publish refresh Q4 | 🟢 Authored |
| `docs/BUSINESS_CONTINUITY.md` | Business Continuity and Disaster Recovery Plan — RTO/RPO targets, incident tiers, escalation paths, vendor substitution procedures | CC9.1, A1.2 | Annual review (Q1); update on architecture change | 🟢 Authored |
| `enterprise/runbooks/FORM-DR-003.md` | DR Runbook: Supabase primary-region failure and failover to read replica | CC9.1, A1.2 | Test execution: semi-annual | 🟢 Authored · v2.38.1 |
| `enterprise/runbooks/FORM-DR-004.md` | DR Runbook: Cloudflare Workers service degradation and origin fallback | CC9.1, A1.2 | Test execution: semi-annual | 🟢 Authored · v2.38.1 |
| `enterprise/runbooks/FORM-DR-005.md` | DR Runbook: Total vendor outage — multi-provider failure scenario and manual operations | CC9.1, A1.2 | Test execution: annual | 🟢 Authored · v2.38.1 |
| `docs/SOC2_READINESS.md §17` | Vendor Management Programme — 5-step annual review process, tier classification, pre-approval requirements, risk scoring matrix, termination procedure | CC9.1, CC9.2, CC9.3 | Annual programme execution Q1 | 🟢 Programme designed |
| `docs/SOC2_READINESS.md §14` | Risk Register — vendor/operational risks: R-004 (Cloudflare dependency), R-005 (Supabase data residency), R-006 (Anthropic availability), R-007 (WorkOS SSO) | CC9.1 | Quarterly review | 🟡 Partial — risk entries exist; quarterly reviews not yet cadenced |
| _(pending)_ `compliance/vendor-review/2027/annual-vendor-review-2027.md` | Annual vendor security review memo — Q1 2027 first execution; questionnaire responses from Critical/High-tier vendors; SOC 2 cert review | CC9.2 | Annual (January) | 🔴 First execution Q1 2027 |
| _(pending)_ `compliance/vendor-review/2027/vendor-soc2-certs/` | SOC 2 / ISO 27001 reports from Critical-tier vendors (Supabase, Cloudflare, WorkOS, Anthropic, Stripe) | CC9.2 | Annual collection | 🔴 First collection Q1 2027 |
| _(pending)_ `compliance/vendor-review/2027/questionnaire-responses/` | Security questionnaire responses from vendors that do not publish SOC 2 reports | CC9.2 | Annual; or current bridge letter acceptable | 🔴 First collection Q1 2027 |

---

## Vendor Tier Summary

| Vendor | Tier | Data Access | DPA | SOC 2 | Annual Review |
|---|---|---|---|---|---|
| **Supabase** | Critical | User health data, tenant data, all production data | 🟢 Signed (DPA + SCC) | 🟢 SOC 2 Type II | Q1 2027 |
| **Cloudflare** | Critical | Network traffic, API requests, Worker secrets | 🟢 Signed (DPA + SCC) | 🟢 SOC 2 Type II | Q1 2027 |
| **WorkOS** | Critical | SSO tokens, enterprise tenant identity | 🟢 Signed | 🟢 SOC 2 Type II | Q1 2027 |
| **Anthropic** | Critical | User coaching prompts (no PII policy), response data | 🟢 Signed | 🟡 Bridge letter | Q1 2027 |
| **Stripe** | High | Payment method tokens, billing metadata | 🟢 Signed | 🟢 PCI DSS Level 1 + SOC 2 | Q1 2027 |
| **ElevenLabs** | High | Victor voice synthesis requests (no health data) | 🟢 Signed | 🟡 ISO 27001 pending | Q1 2027 |
| **PostHog** | Standard | Anonymised analytics events, session metadata | 🟢 Signed | 🟢 SOC 2 Type II | Q1 2027 |
| **Sentry** | Standard | Error reports, stack traces (no health data policy) | 🔴 In progress (AR-2026-Q2-02 finding) | 🟢 SOC 2 Type II | Q1 2027 |

---

## Gap Status

| Gap ID | Item | Priority | Status |
|---|---|---|---|
| **CC9-GAP-001** | Complete Sentry DPA | P1 — AR-2026-Q2-02 finding; escalation deadline 2026-06-12 | 🔴 Open → compliance-officer to contact Sentry legal |
| **CC9-GAP-002** | First annual vendor security review execution (Q1 2027) | P1 — programme defined; first evidence cycle not yet run | 🟡 Planned — Q1 2027 |
| **CC9-GAP-003** | Quarterly risk register review cadence | P1 — risk entries exist in §14 but no quarterly review record | 🔴 Open → add to §15 compliance calendar |
| **CC9-GAP-004** | Sub-processor list public publication at `form.coach/legal/sub-processors` | P0 — required before enterprise GA for GDPR Art. 13(1)(e) | 🔴 Open — authored in `docs/SUBPROCESSORS.md` but not yet published as a live page |
| **CC9-GAP-005** | DR runbook test execution — FORM-DR-003/004/005 | P1 — runbooks authored; first tabletop or live test not yet scheduled | 🔴 Open → schedule semi-annual test for FORM-DR-003/004; annual for FORM-DR-005 |

---

## Compliance Calendar

| Quarter/Event | Task | Artefact |
|---|---|---|
| **Q1 (Jan)** — Annual | Annual vendor security review — send questionnaire to all non-SOC-2 vendors; collect SOC 2 certs from Critical/High vendors; review DPA validity; generate sign-off memo | `compliance/vendor-review/YYYY/annual-vendor-review-YYYY.md` |
| **Q1 (Jan)** — Annual | Annual BCP/DR review — validate FORM-DR-003/004/005 runbooks against current architecture; update RTO/RPO targets if infrastructure changed | Updated `docs/BUSINESS_CONTINUITY.md` + dated sign-off commit |
| **Q1 (Jan)** — Annual | Annual penetration test — external firm; scope: API endpoints, auth flows, RLS bypass, Workers | Pentest report filed to `compliance/evidence/cc7/pentest/YYYY/` |
| **Monthly** | Sub-processor list currency check — confirm no new processors added without 30-day customer notice | Git commit timestamp on `docs/SUBPROCESSORS.md` |
| **Q4 (Dec)** | Sub-processor list publication refresh — update `form.coach/legal/sub-processors`; send 30-day advance notice for any new processors taking effect Q1 | Published page with "last updated" date |
| **Semi-annual** | DR runbook test — tabletop or live execution of FORM-DR-003 (Supabase failover) and FORM-DR-004 (Cloudflare fallback) | Test execution report filed to `enterprise/runbooks/test-reports/` |
| **Annual** | DR runbook test — live execution of FORM-DR-005 (multi-vendor failure) | Test execution report |
| **On any new vendor addition** | Pre-approval review (§17.3) — tier classification, DPA review, SOC 2 check, data minimisation sign-off | `docs/VENDOR_REGISTRY.md` updated row + Linear ticket |
| **On any vendor termination** | Termination procedure (§17.8) — confirm data deletion; revoke credentials; obtain deletion confirmation letter | `compliance/vendor-review/offboarded/VENDOR-YYYY-MM-DD.md` |
| **On Series A** | Upgrade vendor review programme — add independent security assessments for Critical-tier vendors; consider Vanta or Drata for automated evidence collection | Vendor review programme v2 |

---

## Open Action Items

| ID | Action | Owner | Priority | Target |
|---|---|---|---|---|
| CC9-P0-001 | Publish sub-processor list at `form.coach/legal/sub-processors` (static page rendered from `docs/SUBPROCESSORS.md`); required before enterprise GA for GDPR Art. 13(1)(e) | compliance-officer | P0 | Before enterprise GA (M13) |
| CC9-P1-002 | Escalate Sentry DPA to resolution (AR-2026-Q2-02): contact Sentry legal; obtain DPA + SCC; update `docs/VENDOR_REGISTRY.md`; update `compliance/policy-approval-log.csv` | compliance-officer | P1 | 2026-06-12 |
| CC9-P1-003 | Add quarterly risk register review to `docs/SOC2_READINESS.md §15` compliance calendar; create Linear recurring task for Q3/Q4 2026 first execution | compliance-officer | P1 | 2026-06-14 |
| CC9-P1-004 | Schedule semi-annual DR tabletop test for FORM-DR-003 (Supabase failover) and FORM-DR-004 (Cloudflare fallback); target: Q3 2026 | devops-lead | P1 | 2026-07-31 |
| CC9-P2-005 | Evaluate Vanta or Drata for automated SOC 2 evidence collection at Series A; projected time savings vs manual evidence management | compliance-officer | P2 | Series A due diligence |
| CC9-P2-006 | Author `docs/VENDOR_REGISTRY.md` update: add `anthropic_data_minimisation_policy` column confirming no PII in API prompts per FORM data handling spec | compliance-officer + platform-engineer | P2 | Before enterprise GA |

---

*v1.0 · 2026-06-07 · Owner: compliance-officer + devops-lead*
*Directory created: compliance/cc9/ — closes structural gap (CC2, CC3, CC9 were the only missing CC directories; CC9 is now initialised)*
*Gaps: CC9-GAP-001 (Sentry DPA, P1, escalation 2026-06-12); CC9-GAP-004 (sub-processor page, P0, before enterprise GA); CC9-GAP-005 (DR runbook tests, P1, Q3 2026)*
*Next milestone: Q1 2027 — first full annual vendor security review evidence cycle*
*Cross-ref: `docs/SOC2_READINESS.md §17` (vendor management programme); `docs/BUSINESS_CONTINUITY.md` (BCP); enterprise DR runbooks FORM-DR-003–005*
