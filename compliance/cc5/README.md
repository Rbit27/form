# FORM · CC5 Control Activities — Evidence Index

> SOC 2 Type II · Trust Service Criteria CC5 (Control Activities).
> Owner: compliance-officer + security-engineer. Review: quarterly.
> Reference: `docs/SOC2_READINESS.md §27`, `docs/CRYPTOGRAPHY_POLICY.md`, `docs/ACCEPTABLE_USE_POLICY.md`, `compliance/policy-approval-log.csv`.

---

## CC5 Criteria Coverage

| Criterion | Description | Status |
|---|---|---|
| CC5.1 | Control activities selected and developed to mitigate risks to the achievement of objectives | 🟡 Partial — risk-to-control matrix complete (§27.1.1); gaps: TruffleHog CI (CC5-P1-003), IaC drift detection (CC5-P1-005) |
| CC5.2 | General control activities over technology to support the achievement of objectives | 🟡 Partial — IaC enforcement table, configuration baseline, logical access table, and automated monitoring documented; gaps: CSP header (CC6-GAP-007), Dependabot CI gate (CC6-GAP-005), TruffleHog CI (CC5-P1-003), IaC drift (CC5-P1-005) |
| CC5.3 | Control activities deployed through policies and procedures | 🟡 Partial — 10 of 12 policies authored or in force; 1 policy pending counsel (AUP); 1 policy partial (Security Awareness Training standalone doc); policy approval log exists |

---

## Evidence Files

| File | Description | SOC 2 Criteria | Status |
|---|---|---|---|
| `docs/CRYPTOGRAPHY_POLICY.md` | Cryptography Policy v1.0 (POL-002) — cipher suites, key lengths, rotation SLA, key custody, SHA-1/MD5 prohibition | CC5.2, CC5.3, C1.1, C1.2 | 🟢 In Force · 2026-05-21 |
| `docs/ACCEPTABLE_USE_POLICY.md` | Acceptable Use Policy v1.0 (POL-001) — permitted uses, prohibited uses, consequences, review cadence | CC1.1, CC5.3, CC6.3 | 🟡 Authored · pending external counsel review and founder signature |
| `compliance/policy-approval-log.csv` | Policy approval register (POL-001 through POL-011) — policy name, version, status, approval date, approver, next review | CC5.3 | 🟢 Exists · 11 policies registered |
| `docs/SOC2_READINESS.md §27.1.1` | Risk-to-control mapping matrix — 10 risk categories mapped to named controls with owners, review cadences, and evidence artefacts | CC5.1 | 🟢 Complete |
| `docs/SOC2_READINESS.md §27.1.2` | Control ownership register — 12 controls with owners, authority, review cadence, and evidence artefact IDs | CC5.1 | 🟢 Complete |
| `docs/SOC2_READINESS.md §27.2.1` | IaC enforcement table — 9 deployment systems with change method, config location, drift detection status | CC5.2 | 🟡 Partial — drift detection gap (CC5-P1-005) |
| `docs/SOC2_READINESS.md §27.2.2` | Configuration baseline — 13 controls with current setting, compliant target, and enforcement method | CC5.2 | 🟡 Partial — CSP gap (CC6-GAP-007) |
| `docs/SOC2_READINESS.md §27.2.3` | Logical access table for 6 admin surfaces — MFA status, session policy, break-glass | CC5.2 | 🟡 Partial — MFA gaps CC6-GAP-001/002 |
| `docs/SOC2_READINESS.md §27.3.2` | Solo-founder compensating control narrative for policy communication evidence | CC5.3 | 🟢 Auditor-ready |
| `docs/SOC2_READINESS.md §27.3.3` | Control procedure deployment table — 12 policies mapped to implementing runbooks | CC5.3 | 🟡 Partial — AUP pending counsel |
| _(pending)_ `compliance/cc5/trufflehog-ci-results/` | TruffleHog CI scan results — per-PR log of verified credential scans | CC5.1, CC5.2 | 🔴 Gap — CC5-P1-003 not yet implemented |

---

## Gap Status

| Gap ID | Item | Priority | Status | Closed by |
|---|---|---|---|---|
| **CC5-GAP-001** | Acceptable Use Policy | P0 — blocks CC5.3 and CC1.1 from 🟡 → 🟢 | 🟡 Authored · pending counsel | `docs/ACCEPTABLE_USE_POLICY.md` (POL-001) exists; pending external counsel review + founder signature |
| **CC5-GAP-002** | Cryptography Policy | P0 — blocks CC5.3 and CC5.2 | 🟢 Closed | `docs/CRYPTOGRAPHY_POLICY.md` (POL-002) in force · 2026-05-21 |
| **CC5-GAP-003** | TruffleHog CI secret scan | P1 | 🔴 Open | GitHub Actions workflow not yet implemented (CC5-P1-003) |
| **CC5-GAP-004** | `compliance/policy-approval-log.csv` creation | P1 | 🟢 Closed | `compliance/policy-approval-log.csv` exists with 11 policies (POL-001 through POL-011) |
| **CC5-GAP-005** | Terraform IaC drift detection in CI | P1 | 🔴 Open | CC5-P1-005 — remote state backend + GitHub Actions step not yet implemented |

---

## Compliance Calendar

| Quarter | Task | Artefact |
|---|---|---|
| Q1 (Jan) | Annual policy review — all 12 policies; update `compliance/policy-approval-log.csv` with new review date and version | Updated `policy-approval-log.csv` rows |
| Q1 (Jan) | Security awareness training completion — founder (and any hires) | `compliance/cc1/security-training-log.md` |
| Q3 (Jul) | Mid-year policy spot-check: CRYPTOGRAPHY_POLICY.md cipher suite review vs current `wrangler.toml` TLS config | Spot-check note in Linear |
| On new hire | Policy acknowledgment sign-off for all 12 policies | `compliance/evidence/cc1/onboarding/[name]-[date]-aup-signed.pdf` |
| On every CC5-P1-003 CI run | TruffleHog scan result archived | `compliance/cc5/trufflehog-ci-results/` _(pending implementation)_ |
| On any cipher suite change | Cryptography Policy updated + `policy-approval-log.csv` version bumped | Updated POL-002 row |

---

## Open Action Items

| ID | Action | Owner | Priority |
|---|---|---|---|
| CC5-P0-001 | Complete external counsel review of `docs/ACCEPTABLE_USE_POLICY.md`; obtain founder signature; update POL-001 row in `compliance/policy-approval-log.csv` to `IN_FORCE` | compliance-officer | P0 — before first hire |
| CC5-P1-003 | Implement TruffleHog CI GitHub Actions step (`trufflesecurity/trufflehog@main`, `--only-verified`); fail PR on any verified credential finding; archive results to `compliance/cc5/trufflehog-ci-results/` | security-engineer | P1 — before observation period |
| CC5-P1-005 | Add `terraform plan` CI check for `infra/cloudflare/`; fail on non-zero diff outside a planned-change PR; alert `#ops-alerts` | devops-lead | P1 — before observation period |
| CC5-P2-007 | Author new-hire security onboarding checklist at `compliance/onboarding-checklist-template.md`; include 12-policy acknowledgment sign-off, 1Password vault invite, MDM enrolment, GitHub 2FA, security training, break-glass briefing | compliance-officer | P2 — pre-hire |

---

*v1.0 · 2026-06-07 · Owner: compliance-officer + security-engineer*
*Gaps closed this version: CC5-GAP-002 (Cryptography Policy in force), CC5-GAP-004 (policy-approval-log.csv created)*
*Gaps downgraded this version: CC5-GAP-001 (AUP 🔴 → 🟡 authored, pending counsel)*
