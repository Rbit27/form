# FORM · Open-Source Licence Register

> Weak-copyleft OSS dependency register. Required by `docs/SOC2_READINESS.md §54.7.1`.
> Owner: `compliance-officer` + `security-engineer`. Review: quarterly + on every new entry.
> Process governed by `docs/SOC2_READINESS.md §54.7` (Snyk licence scanning programme).

**Register ID:** OSS-LIC-001
**Schema version:** 1.0 (2026-06-07)
**Status:** Active — pre-launch baseline; no weak-copyleft production dependencies registered as of 2026-06-07.

---

## Purpose

This register tracks all production dependencies under weak-copyleft licences (MPL-2.0, LGPL-2.1, LGPL-3.0, EUPL-1.2). For each registered package, the compliance-officer documents the specific use pattern and confirms that FORM's use does not trigger the copyleft licence conditions.

Packages under permissive licences (MIT, Apache-2.0, BSD-2-Clause, BSD-3-Clause, ISC, CC0-1.0, 0BSD, Unlicense) require **no registration**. Packages under strong-copyleft or commercial-restriction licences (GPL-2.0, GPL-3.0, AGPL-3.0, SSPL, Commons Clause, BSL) are **prohibited without legal sign-off** and must not appear here — they follow a separate legal-review process with written approval from `security-engineer`, `compliance-officer`, and founder (see §6 below).

SOC 2 controls supported: CC9.2 (vendor/supply-chain risk management), CC6.8 (malicious or unauthorised software prevention), CC5.2 (software inventory).

---

## Licence Classification Reference

| Category | SPDX Identifiers | Policy |
|---|---|---|
| **Permitted — no registration** | MIT, Apache-2.0, BSD-2-Clause, BSD-3-Clause, ISC, CC0-1.0, Unlicense, 0BSD | May be added to any production dependency without review |
| **Permitted — register and review** (this document) | MPL-2.0, LGPL-2.1, LGPL-3.0, EUPL-1.2 | Permitted if FORM's use does not trigger copyleft conditions. Dynamic linking acceptable; static linking is not. Register below before adding to `package.json` or any platform manifest. |
| **Prohibited without legal sign-off** | GPL-2.0, GPL-3.0, AGPL-3.0, SSPL, Commons Clause, Business Source Licence (BSL) | Do not add without written legal opinion. Three-person approval required (see §6). |
| **Prohibited absolutely** | No SPDX identifier (no licence) | Cannot determine terms. Do not use in any environment. |

*Copyleft trigger rule:* A weak-copyleft licence (LGPL/MPL/EUPL) is triggered when FORM distributes a **modified** copy of the licensed component, or — for LGPL — when the component is **statically linked** into a FORM executable. Dynamic linking, API/HTTP calls, and use as a build-time tool do not trigger copyleft obligations in the vast majority of cases. When in doubt, obtain a legal opinion before proceeding.

---

## Dependency Layers

FORM's production dependency surface spans four distinct layers. Each layer has different licence exposure:

| Layer | Runtime | Package Manager | Notes |
|---|---|---|---|
| **iOS mobile app** | Swift 6 + SwiftUI | Swift Package Manager (SPM) | Apple-framework dependencies have Apple-specific licences; OSS packages via SPM |
| **Cloudflare Workers (backend)** | TypeScript / Node 18 compat | npm / pnpm | Primary OSS dependency surface; Snyk scans this layer |
| **Android mobile (Phase 2)** | Kotlin + Jetpack Compose | Gradle | Activates M4+; Gradle dependency audits required at first build |
| **Build tooling (dev-only)** | TypeScript | npm / pnpm | Dev-only dependencies with copyleft licences are lower risk; register here as best practice |

---

## Current Register — Weak-Copyleft Dependencies

*As of 2026-06-07 (pre-launch baseline): **zero weak-copyleft dependencies registered.***

The Cloudflare Workers dependency tree (Hono, Supabase JS client, WorkOS SDK, Stripe Node, PostHog Node, Sentry Node) consists entirely of MIT or Apache-2.0 packages. The iOS SPM dependency tree (Apple frameworks, no third-party OSS that is LGPL/MPL) is similarly clean.

This table will be populated as each weak-copyleft dependency is added. Add an entry **before** merging the `package.json` or `Package.swift` change.

| # | Package Name | Version | Layer | Licence | Copyleft Trigger Analysis | Date Added | Added By | Reviewed By (compliance-officer) | Review Date | Status |
|---|---|---|---|---|---|---|---|---|---|---|
| — | *none at baseline* | — | — | — | — | — | — | — | — | — |

---

## Snyk CI Integration

Snyk licence scanning is enabled at the organisation level (`app.snyk.io`, `form-production` project). CI gate behaviour:

| Licence Category | CI Outcome | Action |
|---|---|---|
| Prohibited without legal sign-off (GPL/AGPL/SSPL) | **Block** — PR cannot merge | Remove dependency; if legal sign-off already obtained, whitelist in Snyk policy with written approval reference |
| Permitted — register and review (MPL/LGPL/EUPL) | **Warn** — CI passes with annotation | Add entry to this register before next sprint; Snyk warning resolves when annotation is acknowledged |
| Permitted — no registration (MIT/Apache/BSD/ISC) | **Pass silently** | No action required |

Snyk licence policy is reviewed by `compliance-officer` annually (Q1, §8 below). Policy YAML is stored in `snyk.yml` at repo root (added at M3 when Snyk organisation is provisioned).

---

## Local Audit with license-checker

Run this command locally to audit the current npm dependency tree. Output is filed to evidence quarterly (see §8).

```bash
# Prerequisite: npm install -g license-checker
license-checker \
  --production \
  --excludePackages "form" \
  --failOn "GPL;AGPL;SSPL" \
  --csv \
  --out compliance/evidence/licence-audits/licence-audit-$(date +%Y-%m-%d).csv
```

The `--failOn` flag exits non-zero if any package in the resolved production tree has an SPDX identifier matching GPL, AGPL, or SSPL — triggering a CI failure if run in a pipeline.

For iOS (SPM), use `swift package show-dependencies --format json` and review the licence files in each resolved package's source directory. There is no automated SPM licence scanner equivalent to license-checker; manual review is required until a dedicated tool is selected (open question: select an SPM licence scanner at M4 when iOS dependencies are first locked).

---

## Process: Adding a New Weak-Copyleft Dependency

Before opening a PR that adds a MPL-2.0, LGPL-2.1, LGPL-3.0, or EUPL-1.2 package:

1. **Identify the licence.** Confirm the SPDX identifier from the package's `package.json` `license` field or `LICENSE` file. If ambiguous, check `npmjs.com` or the upstream repository.

2. **Perform the copyleft trigger analysis.** Answer these questions in writing:
   - Is FORM distributing a **modified** copy of the package? (Yes = copyleft triggered)
   - For LGPL packages: Is the package **statically linked** into a compiled binary? (Yes = copyleft triggered)
   - Is the package accessed only at runtime via standard npm dynamic loading? (Yes = not triggered)
   - Is the package a build-time tool only (not shipped to production)? (Yes = not triggered, but register as best practice)

3. **Add a row to §4 above** with all required fields. Draft your analysis in the "Copyleft Trigger Analysis" column.

4. **Submit for compliance-officer review.** Assign the PR to `compliance-officer` for review and approval. The PR must not merge until the row in §4 has "Reviewed By" and "Review Date" filled in.

5. **Emit DEC-030 audit event** (see §7). The event is emitted automatically by the CI pipeline on the merge commit if Snyk detects a new weak-copyleft package; it can also be emitted manually via the audit-emit Worker endpoint.

6. **File the updated register to evidence.** On the quarterly evidence filing date (§8), the current state of this document is committed with a dated SHA-256 hash in `compliance/checksums.sha256`.

---

## Process: Legal Sign-Off for Prohibited Licences

If a GPL/AGPL/SSPL/BSL package is genuinely required (rare), follow this process before adding it:

1. **Open a Linear ticket** tagged `[licence-review]` describing the package, its licence, and why it is required.
2. **Obtain a written legal opinion** from external counsel confirming that FORM's specific use pattern does not create a copyleft obligation on FORM's source code.
3. **Three-person approval**: `security-engineer` + `compliance-officer` + founder must all approve the Linear ticket.
4. **Whitelist in Snyk** policy with the Linear ticket ID referenced in the policy comment.
5. **Do not add to this register.** Prohibited-licence approvals are tracked in a separate `compliance/prohibited-licence-approvals/` log (created on first occurrence).
6. **Emit DEC-030 `security.licence_violation_detected`** event with `policy_category: "prohibited_legal_review_required"` — even with approval, the detection event must be emitted and retained.

---

## DEC-030 HMAC-Chained Audit Events

All licence-related events follow the HMAC-chain standard (DEC-030, `docs/AUDIT_LOG_SCHEMA.md`). The following events are relevant to this register:

| Event Type | Severity | Trigger | Retention |
|---|---|---|---|
| `security.licence_violation_detected` | HIGH | Snyk detects a prohibited-category licence in the production dependency tree (auto-emitted by CI) | 1 year |
| `security.licence_weak_copyleft_added` | STANDARD | A weak-copyleft (LGPL/MPL/EUPL) package is added to production dependencies and register entry is created | 1 year |
| `security.licence_register_reviewed` | LOW | Compliance-officer completes quarterly licence register review | 7 years (audit evidence) |
| `security.licence_audit_filed` | LOW | licence-checker CSV audit filed to `compliance/evidence/licence-audits/` | 7 years (audit evidence) |

All events include: `actor_id` (compliance-officer UUID or `ci-sca-worker`), `resource_type: "licence_register"`, `resource_id` (package name + version), `outcome: "success"`, `metadata.spdx_id`, `metadata.package_name`, `metadata.package_version`. No user PII is included.

---

## Annual Audit Procedure

Quarterly evidence filing (aligned with SOC 2 §15 compliance calendar):

| Quarter | Task | Artefact | Stored At |
|---|---|---|---|
| Q1 (Jan) | Run `license-checker` on production tree; review all entries in §4; update Snyk licence policy if new categories detected; compliance-officer signs off | `licence-audit-YYYY-01-15.csv` | `compliance/evidence/licence-audits/` |
| Q2 (Apr) | Run `license-checker`; confirm no new weak-copyleft additions since Q1; update register if any found | `licence-audit-YYYY-04-15.csv` | `compliance/evidence/licence-audits/` |
| Q3 (Jul) | Run `license-checker`; confirm register currency; review any copyleft trigger analyses added since Q2 | `licence-audit-YYYY-07-15.csv` | `compliance/evidence/licence-audits/` |
| Q4 (Oct) | Full annual review: re-check all existing entries against current versions (licence may change on major version bump); update register; compliance-officer signs off annual review | `licence-audit-YYYY-10-15.csv` + `licence-register-annual-review-YYYY.md` | `compliance/evidence/licence-audits/` |

SHA-256 hashes of all audit CSVs and the current state of this register are committed to `compliance/checksums.sha256` on each quarterly filing date.

The Snyk licence policy is reviewed by `compliance-officer` in Q1 alongside the general annual policy review (`compliance/policy-approval-log.csv` entry).

---

## SOC 2 Evidence Mapping

| Control | How This Register Contributes |
|---|---|
| **CC9.2** — Assess and manage risks associated with vendors and business partners | OSS licence register is the evidence artefact for software supply-chain licence risk management. Quarterly audit CSVs + this register constitute the CC9.2 licence risk evidence package. |
| **CC6.8** — Prevent or detect unauthorised or malicious software | Snyk licence scan (CI gate on prohibited licences) is the primary detection control. This register is the evidence of proactive review for all detected weak-copyleft packages. |
| **CC5.2** — Software asset inventory | This register, combined with SBOM (`docs/SOC2_READINESS.md §54.6`), constitutes the OSS inventory evidence. |

Auditor-accessible evidence artefacts:
- `compliance/licence-register.md` — this document (current state of approved weak-copyleft packages)
- `compliance/evidence/licence-audits/licence-audit-YYYY-MM-DD.csv` — quarterly `license-checker` output (one per quarter)
- `compliance/checksums.sha256` — SHA-256 hashes of all evidence files for tamper-evidence
- Snyk organisation licence policy configuration (screenshot or API export, filed in `compliance/evidence/cc9/snyk-licence-policy-YYYY.png`)
- Git history of this file — each entry addition is a dated, attributed commit

---

## Review Log

| Version | Date | Author | Changes |
|---|---|---|---|
| 1.0 | 2026-06-07 | compliance-officer | Initial register created. Pre-launch baseline: zero weak-copyleft production dependencies. Schema, classification reference, Snyk integration, process definitions, and SOC 2 mapping established. `compliance/evidence/licence-audits/` directory path defined; first quarterly audit due Q3 2026 (July). |

*Next review: Q3 2026 audit (July) — run `license-checker` on first locked production `package.json`.*
*Cross-ref: `docs/SOC2_READINESS.md §54.7` (licence scanning programme); `docs/AUDIT_LOG_SCHEMA.md` (DEC-030 event format); `docs/VENDOR_REGISTRY.md` (for vendor-level OSS posture); `compliance/checksums.sha256`.*
