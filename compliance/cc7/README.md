# CC7 — System Operations Evidence

This directory contains evidence artifacts and policy documents for SOC 2 CC7 (System Operations) criteria. CC7 covers threat identification, vulnerability management, security event evaluation, and incident response.

## Policy Documents

| Document | Description | Status | Path |
|---|---|---|---|
| Vulnerability Management & Patching SLA Policy | Standalone policy for CC7.1/CC7.2/CC5.3 — discovery sources, CVSS classification, patching SLAs, triage workflow, risk acceptance, enterprise disclosure templates, DEC-030 audit events | 🟢 IN_FORCE | `compliance/cc7/vuln-management-policy.md` |

## Evidence Artefact Index

Evidence artefacts are filed to `compliance/evidence/vuln-management/YYYY/` per the schedule below. The `README.md` in that directory describes the full directory structure and collection cadence.

| Artefact ID | Description | Cadence | SOC 2 criteria |
|---|---|---|---|
| CC7-E-001 | Linear `[SEC]` ticket export — all vulnerabilities during observation period | Quarterly + audit fieldwork | CC7.1, CC7.2 |
| CC7-E-002 | `npm audit` CI gate pass record | Per audit period | CC7.1, CC7.2 |
| CC7-E-003 | Dependabot alert closure log | Quarterly | CC7.1 |
| CC7-E-004 | Cloudflare WAF rule configuration export | At rule change + quarterly | CC7.1 |
| CC7-E-005 | `audit_log_events` DEC-030 `vuln.*` chain extract, HMAC verified | At audit fieldwork | CC7.1, CC7.2 |
| CC7-E-006 | Risk acceptance memos (SLA exceptions) | Per exception | CC7.2 |
| CC7-E-007 | Responsible disclosure inbox log | Quarterly | CC7.1 |
| CC7-E-008 | Vendor advisory monitoring log | Monthly | CC7.1 |
| CC7-E-009 | SLA adherence summary — per-severity totals, SLA met rate | Quarterly + annual | CC7.1, CC7.2 |

## CC7 Gap Status

| Gap ID | Description | Priority | Status |
|---|---|---|---|
| PRE-15 | Patching SLA enforced and tracked — standalone policy and evidence path | P0 | 🟢 Done — `vuln-management-policy.md` v1.0 |
| CC7.1 | Vulnerability identification | — | 🟡 Advancing — advances to 🟢 after first annual pentest + first full quarter CC7-E-001 |
| CC7.2 | Remediation tracking | — | 🟡 Advancing — advances to 🟢 after first quarter of `vuln.*` DEC-030 events |
| CC5.3 | Vulnerability Management Policy (standalone document) | P0 | 🟢 Done — `vuln-management-policy.md` v1.0 |

---

*Owner: security-engineer + compliance-officer · Created: 2026-06-07*
