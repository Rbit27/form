# Evidence: Vulnerability Management (CC7-E-001 through CC7-E-009)

This directory holds all evidence artefacts for FORM's Vulnerability Management & Patching SLA Programme. It is the evidence repository for SOC 2 CC7.1 and CC7.2 controls. The governing policy is `compliance/cc7/vuln-management-policy.md`.

Evidence is not filed here until the SOC 2 observation period begins. This README documents the expected structure and collection cadence so that the directory is auditor-ready on day one of the observation window.

---

## Directory Structure

```
compliance/evidence/vuln-management/
├── README.md                         — this file
├── advisory-subscriptions.md         — vendor advisory channel subscription confirmations (§2.4)
├── YYYY/
│   ├── linear-export-YYYY-QN.json   — CC7-E-001: Linear [SEC] ticket export
│   ├── ci-audit-gate-YYYY-QN.txt    — CC7-E-002: npm audit CI gate pass record
│   ├── dependabot-alerts-YYYY-QN.csv — CC7-E-003: Dependabot alert closure log
│   ├── cloudflare-waf-YYYY-QN.json  — CC7-E-004: Cloudflare WAF rule export
│   ├── dec030-vuln-chain-YYYY.json  — CC7-E-005: DEC-030 vuln.* chain extract
│   ├── responsible-disclosure-log.csv — CC7-E-007: Responsible disclosure inbox log
│   ├── vendor-advisory-log.csv      — CC7-E-008: Vendor advisory monitoring log
│   └── sla-adherence-YYYY.md        — CC7-E-009: SLA adherence summary
└── risk-acceptance/
    └── YYYY/
        └── YYYY-NNNN.md             — CC7-E-006: Risk acceptance memos (per exception)
```

---

## Artefact Descriptions

| Artefact ID | File pattern | Description | Collection owner | Cadence |
|---|---|---|---|---|
| **CC7-E-001** | `YYYY/linear-export-YYYY-QN.json` | Linear export of all `[SEC]`-labelled tickets during observation period — includes `vuln_discovered_at`, `patched_at`, `sla_met`, `cvss_base`, `final_severity` fields | compliance-officer | Quarterly + at audit fieldwork |
| **CC7-E-002** | `YYYY/ci-audit-gate-YYYY-QN.txt` | GitHub Actions workflow run log excerpts confirming `npm audit --audit-level=critical` passed on every merge to `main` during the observation period | devops-lead | Per audit period export |
| **CC7-E-003** | `YYYY/dependabot-alerts-YYYY-QN.csv` | GitHub Security tab → Dependabot alerts CSV export — all alerts opened and closed with timestamps | security-engineer | Quarterly |
| **CC7-E-004** | `YYYY/cloudflare-waf-YYYY-QN.json` | Cloudflare dashboard → Firewall → Rules export confirming OWASP CRS version and custom rule set active | devops-lead | At any rule change + quarterly |
| **CC7-E-005** | `YYYY/dec030-vuln-chain-YYYY.json` | `audit_log_events` query filtered on `event_type LIKE 'vuln.%'`, HMAC chain verified, covering full observation period | security-engineer | At audit fieldwork |
| **CC7-E-006** | `risk-acceptance/YYYY/YYYY-NNNN.md` | Risk acceptance memos for any SLA exceptions granted — one file per exception, includes CVE ID, approver signatures, compensating control, expiry date | compliance-officer | Per SLA exception granted |
| **CC7-E-007** | `YYYY/responsible-disclosure-log.csv` | Log of all reports received at `security@form.coach` — pseudonymised if reporter requests anonymity; includes receipt timestamp, triage outcome, Linear ticket reference | security-engineer | Quarterly |
| **CC7-E-008** | `YYYY/vendor-advisory-log.csv` | Record of vendor advisory channels reviewed per policy §2.4 — date reviewed, advisory ID if applicable, action taken | security-engineer | Monthly |
| **CC7-E-009** | `YYYY/sla-adherence-YYYY.md` | Per-severity tier summary: total CVEs discovered, patched within SLA, SLA exceptions granted, Won't Fix closures, open at period end; SLA adherence rate (%) | security-engineer | Quarterly summary + annual report |

---

## Collection Timeline (First SOC 2 Observation Year)

| Period | Artefacts due | Owner |
|---|---|---|
| Month O+0 (observation start) | advisory-subscriptions.md filed and up to date | security-engineer |
| Month O+3 (Q1 close) | CC7-E-001, CC7-E-002, CC7-E-003, CC7-E-004, CC7-E-007, CC7-E-008, CC7-E-009 (Q1) | compliance-officer + security-engineer |
| Month O+6 (Q2 close) | Same as Q1 + any CC7-E-006 risk acceptance memos | compliance-officer + security-engineer |
| Month O+9 (Q3 close) | Same as Q1 | compliance-officer + security-engineer |
| Month O+12 (annual) | CC7-E-005 (full year DEC-030 extract); CC7-E-009 annual summary; pentest artefacts (filed in `compliance/pentest/YYYY/`) | compliance-officer |
| At audit fieldwork | CC7-E-001, CC7-E-005 current through fieldwork date | compliance-officer |

---

## Evidence Retention

All artefacts in this directory are retained for **7 years** per the FORM data retention schedule (`docs/SOC2_READINESS.md §11.3`). DEC-030 `vuln.*` events in the audit log carry a 7-year retention tier.

---

*Owner: compliance-officer · Created: 2026-06-07 · Review: annually per §15.1 compliance calendar*
