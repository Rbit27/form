# FORM Compliance Artifacts

This directory contains auditor-legible compliance artifacts for FORM's SOC 2 Type II programme. It is maintained by the `compliance-officer` role.

For the full SOC 2 readiness roadmap, gap analysis, and control mapping, see `docs/SOC2_READINESS.md`.

---

## Directory Structure

### `cc4/` — CC4 Monitoring Activities

Artifacts remediating the three CC4.2 monitoring gaps identified in `docs/SOC2_READINESS.md §32`.

| File | Description | Auditor Exhibit |
|---|---|---|
| `cc4/control-deficiency-log.csv` | Structured register of all known control deficiencies — mandatory columns include criterion, severity, root cause, owner, target date, and closure date. All pre-launch deficiencies seeded. | CC4-E-001 |
| `cc4/evidence-collection-plan.md` | Execution schedule for all CC4.1 ongoing and separate evaluation controls — what is collected, how often, where it is stored, and who is responsible. Collection begins at launch. | CC4-E-002 |
| `cc4/deficiency-communication-procedure.md` | Three-tier compensating control for CC4.2 board communication (P0 gap): Tier 1 founder monthly self-review, Tier 2 advisory board quarterly notification, Tier 3 investor observer quarterly update. Expires 90 days after Series A close. | CC4-E-003 |

### `evidence/` — Evidence Packages

Created at launch. Evidence packages are filed here by type and period as ongoing controls execute across the SOC 2 observation window. Naming conventions are defined in `cc4/evidence-collection-plan.md`.

| Path pattern | Content |
|---|---|
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
