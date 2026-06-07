# FORM Compliance Artifacts

This directory contains auditor-legible compliance artifacts for FORM's SOC 2 Type II programme. It is maintained by the `compliance-officer` role.

For the full SOC 2 readiness roadmap, gap analysis, and control mapping, see `docs/SOC2_READINESS.md`.

---

## Directory Structure

### `cc1/` — CC1 Control Environment

Artifacts for SOC 2 CC1 (Control Environment) — security training, AUP acknowledgment, new-hire onboarding.

| File | Description | Auditor Exhibit |
|---|---|---|
| `cc1/README.md` | Evidence index for CC1 artifacts (CC1-E-001 through CC1-E-005) and gap status | — |
| `cc1/security-training-log.md` | Founder annual security training completion record — log template and workflow | CC1-E-001 |

### `cc4/` — CC4 Monitoring Activities

Artifacts remediating the three CC4.2 monitoring gaps identified in `docs/SOC2_READINESS.md §32`.

| File | Description | Auditor Exhibit |
|---|---|---|
| `cc4/control-deficiency-log.csv` | Structured register of all known control deficiencies — mandatory columns include criterion, severity, root cause, owner, target date, and closure date. All pre-launch deficiencies seeded. | CC4-E-001 |
| `cc4/evidence-collection-plan.md` | Execution schedule for all CC4.1 ongoing and separate evaluation controls — what is collected, how often, where it is stored, and who is responsible. Collection begins at launch. | CC4-E-002 |
| `cc4/deficiency-communication-procedure.md` | Three-tier compensating control for CC4.2 board communication (P0 gap): Tier 1 founder monthly self-review, Tier 2 advisory board quarterly notification, Tier 3 investor observer quarterly update. Expires 90 days after Series A close. | CC4-E-003 |

### `cc6/` — CC6 Logical and Physical Access

Artifacts for SOC 2 CC6 (Logical and Physical Access Controls).

| File | Description |
|---|---|
| `cc6/offboarding-procedure.md` | Employee and contractor offboarding checklist — credential revocation, NDA reminder, device return. Addresses CC6.3 access termination. |

### `cc7/` — CC7 System Operations

Policy documents and evidence index for SOC 2 CC7 (System Operations) — vulnerability management, patching, incident detection.

| File | Description | Auditor Exhibit |
|---|---|---|
| `cc7/README.md` | Evidence index for CC7 artifacts (CC7-E-001 through CC7-E-009) and gap status | — |
| `cc7/vuln-management-policy.md` | **Vulnerability Management & Patching SLA Policy v1.0** — standalone policy (POL-010). Closes PRE-15 and CC5.3. Four discovery sources, CVSS classification with FORM-specific uplift rules, binding patching SLAs (Critical 24 h / High 7 d / Medium 30 d / Low 90 d), DEC-030 audit events, enterprise disclosure templates. | CC5-E-003 |

### `c1/` — C1 Confidentiality

Artifacts for SOC 2 C1 (Confidentiality) — data asset inventory, access controls, encryption evidence.

| File | Description |
|---|---|
| `c1/README.md` | Evidence index and gap status for C1 criteria |
| `c1/data-asset-inventory.md` | Data asset inventory per C1-GAP-002 — all data categories, classification tier, storage location, access controls, retention schedule |

### `p1/` — P1 Privacy

Artifacts for SOC 2 P1 and GDPR compliance — sub-processor register, complaint intake, retention decisions.

| File | Description |
|---|---|
| `p1/sub-processor-register.md` | Vendor / sub-processor risk register (POL-007) — all processors with DPA status and data categories |
| `p1/retention-decisions.md` | Data retention decisions (POL-008) — all data categories with retention periods and legal basis |
| `p1/complaint-intake-procedure.md` | Formal complaint intake procedure (POL-009) — regulatory and user complaints |
| `p1/complaint-log.csv` | Running log of all complaints received |

### `access-review/` — Quarterly Access Review

Quarterly access review artifacts per §23 of `docs/SOC2_READINESS.md`.

| File | Description |
|---|---|
| `access-review/access-review-2026-Q2.md` | Q2 2026 access review — production system credentials, service account audit, principle of least privilege verification |
| `access-review/authorized-roster.md` | Current authorized personnel roster with access scope |

### `vendor-review/` — Vendor Security Review

Annual vendor security review artifacts per §17 of `docs/SOC2_READINESS.md`.

| File | Description |
|---|---|
| `vendor-review/questionnaire-template.md` | Security questionnaire template sent to sub-processors during annual vendor review (CC9.1) |

### `calendar/` — Compliance Calendar

Compliance calendar artifacts and scheduled review records.

| File | Description |
|---|---|
| `calendar/q3-2026-access-review.md` | Q3 2026 access review pre-scheduling and scope definition |

### `evidence/` — Evidence Packages

Evidence packages are filed here by type and period as ongoing controls execute across the SOC 2 observation window. Sub-directories are created at launch.

| Path pattern | Content |
|---|---|
| `evidence/vuln-management/` | **Vulnerability management evidence (CC7-E-001 through CC7-E-009)** — see `evidence/vuln-management/README.md` |
| `evidence/soc2-monthly-YYYY-MM.md` | Monthly SOC 2 dashboard memo (Tier 1 self-review) |
| `evidence/chain-check-YYYY-MM/` | Daily HMAC chain integrity check results (probe S-007) |
| `evidence/access-review-YYYY-QN.md` | Quarterly access review artifact (§23 procedure) |
| `evidence/supabase-YYYY-MM-DD.json` | Weekly Supabase database health export |
| `evidence/cloudflare-YYYY-MM.csv` | Monthly Cloudflare analytics snapshot |
| `evidence/pentest-YYYY/` | Annual penetration test package |
| `evidence/soc2-annual-YYYY.md` | Year-over-year SOC 2 readiness review |
| `evidence/advisory-brief-YYYY-QN.md` | Advisory board deficiency communication summary (Tier 2) |
| `evidence/investor-update-YYYY-QN-sent.txt` | Investor observer update confirmation (Tier 3) |

---

## SOC 2 Readiness Reference

Full control mapping, gap analysis, and pre-observation checklist: `docs/SOC2_READINESS.md`

Current SOC 2 readiness (self-assessed, as of May 2026): ~90%

Target: Type I report — Month 3 post-enterprise launch. Type II report — Month 12.
