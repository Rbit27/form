# CC1 — Control Environment Evidence

This directory contains evidence artifacts for SOC 2 CC1 (Control Environment) criteria.

## Evidence Index

| Artifact ID | Description | Status | Path |
|---|---|---|---|
| CC1-E-001 | Founder annual security training completion record | 🟡 AUTHORED — log template + workflow defined; evidence pending founder completion | `compliance/cc1/security-training-log.md` |
| CC1-E-002 | New hire onboarding checklist (signed) | 🔴 Pending — required on first hire | evidence/cc1/onboarding/ |
| CC1-E-003 | Code of conduct acknowledgment | 🟡 Partial — CONTRIBUTING.md in git history | git commit history |
| CC1-E-004 | Acceptable Use Policy — current version + signed copies | 🟡 In Progress — AUP authored v1.0; signing pending counsel review | aup/ |
| CC1-E-005 | Phishing simulation report | 🔴 Pending — requires first hire + KnowBe4/GoPhish (CC1-GAP-003) | evidence/cc1/phishing/ |

## AUP Acknowledgment Path

When the AUP (POL-001 in `compliance/policy-approval-log.csv`) is formally signed:

1. Founder signs `docs/ACCEPTABLE_USE_POLICY.md` via git commit tagged `aup-signed-[date]`
2. File PDF copy (or git commit SHA + date) to `compliance/cc1/aup/aup-v1.0-signed-[name]-YYYY-MM-DD.md`
3. Update `compliance/policy-approval-log.csv` POL-001: status → `IN_FORCE`, effective_date → signing date
4. Update `docs/SOC2_READINESS.md` P0-11 gate criterion status → PASS (after counsel review completed)

## CC1 Gap Status

| Gap ID | Description | Priority | Status |
|---|---|---|---|
| CC1-GAP-001 | AUP authored and signed | P0 | 🟡 AUTHORED — pending counsel review + founder signature |
| CC1-GAP-002 | Founder annual training completion | P0 | 🟡 AUTHORED — `compliance/cc1/security-training-log.md` (CC1-E-001, 2026-05-22); closes 🟢 upon founder self-attestation executed |
| CC1-GAP-003 | Phishing simulation programme (KnowBe4 / GoPhish) | P1 | 🔴 Open — requires first hire |
| CC1-GAP-004 | Background check procedure in onboarding checklist | P1 | 🔴 Open — requires first hire |

---

*Owner: compliance-officer · Updated: May 2026*
