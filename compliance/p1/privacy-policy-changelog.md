# Privacy Policy Version Changelog

**File reference:** P1-CLP-001
**SOC 2 criteria:** P1.1 — Notice to Data Subjects · P2.1 — Choice and Consent
**GDPR articles:** Art. 5(1)(b) purpose limitation · Art. 5(1)(e) storage limitation · Art. 7 conditions for consent · Art. 13 information at point of collection · Art. 14 information not obtained directly (not in current scope)
**Remediates:** PRV-05 (`docs/SOC2_READINESS.md §35.3.1`) — privacy policy version management; `consent_version` field in `privacy.consent_granted` DEC-030 event must map to a formal, auditor-reviewable version registry
**Input documents:** `docs/PRIVACY_POLICY.md` · `docs/AUDIT_LOG_SCHEMA.md` (DEC-030) · `docs/SOC2_READINESS.md §35.3.1` (PRV-05, PRV-10) · `docs/SOC2_READINESS.md §74` (`form-consent-gate` Worker) · `docs/ENTERPRISE.md §Privacy Floor` · `docs/SUBPROCESSORS.md §8.2` (enterprise change notification)
**Owner:** compliance-officer
**Decision authority:** compliance-officer + outside counsel (EU-qualified) + founder sign-off
**Status:** 🟡 Authored — pending outside counsel review and founder signature
**Created:** 2026-06-12
**Next review:** Annually — or within 30 days of any material privacy policy change
**Artifact path:** `compliance/p1/privacy-policy-changelog.md`
**SOC 2 evidence:** PRE-35-E-002 (supporting) — documents that `consent_version` is managed through a formal, auditor-reviewable process

---

## Purpose

This changelog records every published and draft version of the FORM Privacy Policy ("Policy"), with classification of each change as material or non-material and whether a consent re-request was required.

It exists to satisfy three overlapping requirements:

1. **PRV-05 closure** — the `consent_version` field stamped on every `privacy.consent_granted` DEC-030 HMAC-chain event must map to a formal, auditor-reviewable version registry. Without this document, the version string in audit events is unverifiable and fails P1.1 at SOC 2 Type II fieldwork.
2. **SOC 2 P1.1 / P2.1 evidence** — auditors must confirm that FORM has a documented mechanism for managing privacy policy updates and for re-obtaining consent when material changes occur (PRV-10).
3. **GDPR Art. 7 / Art. 13 compliance** — consent obtained under a particular policy version is scoped to the purposes stated in that version. Any material change to purposes, data categories, or third-party disclosures invalidates prior consent for affected categories; re-consent is required before continued processing.

---

## Scope

This changelog governs the version lifecycle of:

| Artifact | Path / URL | Notes |
|---|---|---|
| Authoritative policy document | `docs/PRIVACY_POLICY.md` | Internal source of truth; inputs all other artifacts |
| Consumer-facing web page | `form.coach/privacy` (rendered by `privacy.html`) | Live page; must be deployed before any user onboarding begins |
| Consent gate version constant | `CONSENT_SCHEMA_VERSION` in `form-consent-gate` Cloudflare Worker | Source of truth for the in-app consent flow version |
| `consent_version` field in DEC-030 events | `privacy.consent_granted`, `privacy.consent_withdrawn`, `privacy.consent_version_bumped` | Stamped at event time; immutable once written to HMAC chain |
| `users.consent_version` column | Supabase `users` table | Compared to `CONSENT_SCHEMA_VERSION` on each app open to trigger re-consent |

---

## Version Format

Policy versions use the format `YYYY-MM-vN`:

| Component | Meaning | Example |
|---|---|---|
| `YYYY` | Four-digit year of effective date | `2026` |
| `MM` | Two-digit month of effective date | `05` |
| `vN` | Sequential revision within that year-month; resets to `v1` on each new month | `v1` |

**Example:** `2026-05-v1` — first policy version effective in May 2026.

This string is used identically across all artifacts in scope above to ensure a single canonical reference that auditors can trace from the DEC-030 HMAC-chain event through to the published policy text.

---

## Material Change Classification

### Material changes — require consent re-request

A Policy change is **material** if it meets one or more of the following criteria:

| # | Criterion | Examples |
|---|---|---|
| M-1 | Adds a new category of personal data collected | Adding biometric data not previously collected |
| M-2 | Adds a new purpose for processing existing data | Sharing workout data with employer for performance review — prohibited by privacy floor; listed here as illustrative of what triggers re-consent if ever permitted |
| M-3 | Adds a new sub-processor that processes Art. 9 health data directly | New medical analytics vendor with access to `workout_sessions` or `cv_sessions` |
| M-4 | Changes the legal basis for any processing activity | Switching from consent to legitimate interest for health data — not permissible under GDPR Art. 9; listed for completeness |
| M-5 | Extends the retention period for any data category | Extending `coaching_turns` retention from 2yr to 5yr |
| M-6 | Changes how data is disclosed to third parties | Adding disclosure to insurance providers — explicitly prohibited (ENTERPRISE.md no-go list) |
| M-7 | Removes or weakens a user right previously documented | Removing the right to withdraw consent for Art. 9 categories |
| M-8 | Any change to the government request handling provisions | Change to the user notification commitment in `compliance/p1/gov-request-policy.md §Principle 3` |

**Consequence of material change:**
- Increment the `vN` suffix (or roll to a new `YYYY-MM`) and deploy new `CONSENT_SCHEMA_VERSION`
- All users whose `users.consent_version` < new version see the consent re-request screen on next app open (PRV-10)
- Enterprise customers receive 30-day advance notice per DPA §8.2 if the change affects the sub-processor list
- File a `privacy.consent_version_bumped` DEC-030 event (HIGH severity, 7-year retention) with `old_version`, `new_version`, and `change_type = "material"` at the moment of deployment

### Non-material changes — no re-consent required

A Policy change is **non-material** if it:

| # | Criterion | Examples |
|---|---|---|
| N-1 | Corrects a typographical error without changing meaning | Fixing a misspelled word in a data category description |
| N-2 | Improves clarity without expanding scope | Rewording a sentence to be more understandable in plain language |
| N-3 | Updates a contact address or URL without changing the entity | New privacy@form.coach mailbox routing |
| N-4 | Adds explanatory language that does not expand data collection or processing scope | Adding a "what this means for you" plain-language sidebar |
| N-5 | Updates a reference link that was broken | Fixing a dead hyperlink to a sub-processor's privacy page |

**Consequence of non-material change:**
- Create a new version entry in this changelog for audit trail continuity
- Do NOT increment `CONSENT_SCHEMA_VERSION` — existing consents remain valid
- Do NOT trigger consent re-request
- Do NOT require 30-day advance enterprise notice (unless the non-material change is to the sub-processor list — even removing a sub-processor requires 30-day notice per DPA §8.2)
- File a `privacy.consent_version_bumped` DEC-030 event with `change_type = "non-material"` — this creates an audit record without triggering user-facing re-consent

---

## Deployment Process

When a new Policy version is ready to publish, the following steps must be executed in order:

| Step | Action | Owner | DEC-030 Event |
|---|---|---|---|
| 1 | Update `docs/PRIVACY_POLICY.md`: change `version:` front-matter, update `## Version History` footer, get compliance-officer and outside-counsel sign-off | compliance-officer | — |
| 2 | If material change: increment `CONSENT_SCHEMA_VERSION` in `form-consent-gate` Cloudflare Worker source | security-engineer | — |
| 3 | Rebuild `privacy.html` with updated policy text and `<meta name="policy-version" content="YYYY-MM-vN">` tag | devops-lead | — |
| 4 | Deploy `privacy.html` to Cloudflare Pages | devops-lead | — |
| 5 | Deploy updated `form-consent-gate` Worker | devops-lead | `privacy.consent_version_bumped` (HIGH, 7yr) with `old_version`, `new_version`, `change_type` — emitted by Worker on first request post-deploy |
| 6 | Add entry to this changelog (version history table below); commit to `main` | compliance-officer | — |
| 7 | If sub-processor list changed: update `docs/SUBPROCESSORS.md` and redeploy `subprocessors.html`; notify enterprise CSM for 30-day customer notice | customer-success | `enterprise.policy_change_notified` (HIGH, 7yr) — one event per notified tenant |
| 8 | If material change: run smoke test confirming consent re-request displays for a test account with `users.consent_version` < new version | qa-lead | — |
| 9 | File a copy of this changelog in `compliance/evidence/p1/` with date stamp at next quarterly SOC 2 evidence collection | compliance-officer | — |

**Privacy Floor constraint** (`docs/ENTERPRISE.md §Privacy Floor`): No policy version may introduce processing that grants employers visibility into individual employee health, coaching, or CV data. The privacy floor is non-negotiable and cannot be changed by any policy version, customer contract, or legal demand. compliance-officer has veto power on any Policy change that approaches the privacy floor boundary.

---

## Version History

### `2026-05-v1` — Initial Draft

| Field | Value |
|---|---|
| **Version ID** | `2026-05-v1` |
| **Source document** | `docs/PRIVACY_POLICY.md` v0.1.0-draft |
| **Draft date** | 2026-05-17 |
| **Effective date** | TBD — pending outside counsel review (EU/US/UA jurisdictions) and founder sign-off |
| **Change type** | N/A — initial version; no prior version to diff |
| **Material change** | N/A — initial version |
| **Consent re-request** | N/A — initial version; all new users receive the initial consent flow |
| **`CONSENT_SCHEMA_VERSION`** | `2026-05-v1` — set at first production deployment |
| **Status** | 🟡 Drafted — pending outside counsel review (EU/US/UA) + founder sign-off |

**Sign-off status:**

| Signatory | Status | Date |
|---|---|---|
| compliance-officer | Pending | — |
| Outside counsel (EU) | Pending | — |
| Outside counsel (US) | Pending | — |
| Founder | Pending | — |

**Scope of initial version — data categories:**

| Category | GDPR Art. 9? | Legal basis | Consent type |
|---|---|---|---|
| `health_profile` (age, sex, height, weight, injury flags, goal) | Yes | Art. 9(2)(a) explicit consent | Granular opt-in, default OFF |
| `meal_log` | Yes | Art. 9(2)(a) explicit consent | Granular opt-in, default OFF |
| `wearable_data` (HRV, sleep, resting HR, steps) | Yes | Art. 9(2)(a) explicit consent | Granular opt-in, default OFF |
| `cv_workout_analysis` (pose keypoints, rep count, form score) | Yes | Art. 9(2)(a) explicit consent | Granular opt-in, default OFF |
| `ed_screening_result` (SCOFF-light UX flag) | Yes | Art. 9(2)(a) explicit consent via DEC-018 gate | Standalone consent gate, required |
| `usage_analytics` | No | Art. 6(1)(f) legitimate interest | Analytics opt-in toggle, default OFF |
| `payment_metadata` (Stripe customer ID, subscription status) | No | Art. 6(1)(b) contract performance | Mandatory for paid subscription |

**Sub-processors disclosed in this version:** 11 — per `docs/SUBPROCESSORS.md §2` (SP-01 through SP-11)

**Key policy sections:** §1 Identity and contact · §2 Scope · §3 Data collected · §4 Legal basis · §5 Art. 9 special category data · §6 Consent management · §7 Data retention · §8 User rights (Art. 15–22) · §9 Wearable integrations · §10 Enterprise tier · §11 Sub-processors · §12 International transfers (SCCs Module 2) · §13 Version management · §14 Contact and complaints

---

## Consent Re-Request Log

Records all instances where a material Policy change triggered the PRV-10 consent re-request flow in the FORM app.

| Date | Old version | New version | Reason | Users affected | Re-request completion rate (30-day) | DEC-030 event batch ID |
|---|---|---|---|---|---|---|
| *(none — pre-launch; first entry to be added on first production release)* | — | — | — | — | — | — |

---

## SOC 2 Evidence Classification

| Evidence artifact | Location | SOC 2 criterion | Status |
|---|---|---|---|
| This document (P1-CLP-001) | `compliance/p1/privacy-policy-changelog.md` | P1.1, P2.1 | 🟡 Authored |
| DEC-030 `privacy.consent_granted` event sample (staging) | `audit_log` (staging) | P1.1, P2.1 | 🔴 Pending — requires staging `form-consent-gate` deployment |
| DEC-030 `privacy.consent_withdrawn` event sample (staging) | `audit_log` (staging) | P2.1 | 🔴 Pending — requires staging deployment |
| `privacy.consent_version_bumped` DEC-030 event (first publish) | `audit_log` (production) | P1.1 | 🔴 Pending — fires at first production policy deployment |

**Auditor note:** PRE-35-E-002 is a composite artifact. This changelog satisfies the "formal version management process" element. The DEC-030 event samples are the second element. Both must be provided before the first SOC 2 Type II observation period closes. The event samples are generated automatically at production deployment — no separate manual action required beyond staging deployment validation.

---

## Document Control

| Field | Value |
|---|---|
| **Document ID** | P1-CLP-001 |
| **Document version** | 1.0 |
| **Created** | 2026-06-12 |
| **Owner** | compliance-officer |
| **Review cadence** | Annually — or within 30 days of any material privacy policy change |
| **Approval required** | compliance-officer sign-off · outside counsel sign-off · founder sign-off |
| **Filing** | `compliance/p1/privacy-policy-changelog.md`; copy in `compliance/evidence/p1/` at each SOC 2 evidence collection |

---

*Cross-references: `docs/PRIVACY_POLICY.md` · `docs/AUDIT_LOG_SCHEMA.md` (DEC-030 `privacy.consent_granted`, `privacy.consent_withdrawn`, `privacy.consent_version_bumped`) · `docs/SOC2_READINESS.md §35.3.1` (PRV-05) · `docs/SOC2_READINESS.md §35.4.1` (PRV-10) · `docs/SOC2_READINESS.md §74` (`form-consent-gate` Cloudflare Worker) · `docs/ENTERPRISE.md §Privacy Floor` · `docs/SUBPROCESSORS.md §8.2` (enterprise change notification) · `compliance/p1/retention-decisions.md` (P4.2 — data retention periods per category) · `compliance/p1/gov-request-policy.md` (P6.5 — government request provisions) · DEC-030 (HMAC-chained audit log)*

*Co-Authored-By: Claude (enterprise-builder) <cloud@form.coach>*
