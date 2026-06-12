# FORM · Personnel Offboarding Checklist Template

> **Evidence artifact ID:** CC6-E-009 (per departure instance)
> **SOC 2 criteria:** CC6.3 (Termination of Access) · CC6.5 (Removal of Access) · C1.2 (Disposal of Confidential Information)
> **Policy source:** `compliance/c1/device-disposal-policy.md §3` (POL-013) · `docs/SOC2_READINESS.md §40.5`
> **Owner:** compliance-officer + security-engineer

**USAGE:** Copy this template to `compliance/offboarding/{full-name}-{YYYY-MM-DD}/offboarding-checklist.md` on each departure. Replace all `[PLACEHOLDERS]` before use. Do NOT edit this template file — append a new copy per departure.

---

## Offboarding Record

| Field | Value |
|---|---|
| **Individual** | `[DISPLAY NAME ONLY — no email in this file]` |
| **Role** | `[Job title]` |
| **Last day** | `[YYYY-MM-DD]` |
| **Offboarding initiated by** | `[compliance-officer name]` |
| **Initiated at** | `[ISO 8601 timestamp]` |
| **Reason** | `[resignation / termination / contract-end / other]` |
| **Emergency offboarding** | `[yes (R-12 triggered) / no]` |

---

## Phase 1 — Access Revocation (SLA: within 24 hours of last day)

All Phase 1 steps must complete within **24 hours** of the departure confirmation. Record completion time for each step.

| # | System | Action | Owner | Completed at | Evidence |
|---|---|---|---|---|---|
| 1 | **GitHub** | Remove from `rbit27` org; revoke all PATs and GitHub Apps tokens; remove from any team with production-branch push access | security-engineer | `[YYYY-MM-DD HH:MM UTC]` | GitHub audit log export URL or screenshot saved to this folder |
| 2 | **1Password** | Remove from all shared vaults (especially `FORM-Production-Secrets` and `FORM-Infra`); remove device trust for all individual's devices | security-engineer | `[YYYY-MM-DD HH:MM UTC]` | 1Password audit log screenshot |
| 3 | **Cloudflare** | Revoke all API tokens scoped to the individual; remove from Cloudflare account members; revoke any Cloudflare Access policies granting personal access | devops-lead | `[YYYY-MM-DD HH:MM UTC]` | Cloudflare audit log screenshot |
| 4 | **Supabase** | Revoke Supabase dashboard access for the individual's email; rotate any service role key or JWT secret the individual had access to | security-engineer | `[YYYY-MM-DD HH:MM UTC]` | Supabase access log; key rotation record in `docs/KEY_ROTATION.md` |
| 5 | **Sentry** | Remove from Sentry `form-prod` and `form-staging` organizations | security-engineer | `[YYYY-MM-DD HH:MM UTC]` | Sentry admin screenshot |
| 6 | **PostHog EU** | Remove from PostHog organization; revoke any personal API keys | security-engineer | `[YYYY-MM-DD HH:MM UTC]` | PostHog admin screenshot |
| 7 | **Better Stack / Uptime** | Remove from Better Stack account | devops-lead | `[YYYY-MM-DD HH:MM UTC]` | Screenshot |
| 8 | **PagerDuty** | Deactivate PagerDuty user; remove from on-call schedules | devops-lead | `[YYYY-MM-DD HH:MM UTC]` | PagerDuty admin screenshot |
| 9 | **Slack** | Deactivate Slack account; remove from `#security-*` and `#compliance-*` channels | compliance-officer | `[YYYY-MM-DD HH:MM UTC]` | Slack admin screenshot |
| 10 | **Stripe** | Revoke Stripe restricted key if held; remove from Stripe dashboard team | devops-lead | `[YYYY-MM-DD HH:MM UTC]` | Stripe dashboard screenshot |
| 11 | **Anthropic** | Revoke Anthropic API key if the individual held a personal key (not shared Workers Secret) | devops-lead | `[YYYY-MM-DD HH:MM UTC]` | Anthropic dashboard screenshot |
| 12 | **Linear** | Deactivate Linear team member; revoke API keys | compliance-officer | `[YYYY-MM-DD HH:MM UTC]` | Linear admin screenshot |
| 13 | **Google Workspace / Email** | Archive mailbox; disable login; revoke OAuth scopes | compliance-officer | `[YYYY-MM-DD HH:MM UTC]` | Google Admin console screenshot |

---

## Phase 2 — Device Return (SLA: within 7 business days of last day)

| # | Action | Owner | Completed at | Notes |
|---|---|---|---|---|
| 1 | Issue prepaid return shipping label (for remote team member) or arrange in-person collection | compliance-officer | `[YYYY-MM-DD HH:MM UTC]` | Shipping tracking number or in-person confirmation |
| 2 | Receive device(s); log receipt in `compliance/assets/device-register.csv` (`date_returned`) | security-engineer | `[YYYY-MM-DD HH:MM UTC]` | |
| 3 | Initiate device sanitization per `compliance/c1/device-disposal-policy.md §2` (NIST SP 800-88 method appropriate to device class) | security-engineer | `[YYYY-MM-DD HH:MM UTC]` | Method used: `[Cryptographic Erase / 3-pass / Factory Reset / Physical destruction]` |
| 4 | Append disposal row to `compliance/media-disposal/device-disposal-log.csv` (C1-E-008) | security-engineer | `[YYYY-MM-DD HH:MM UTC]` | `disposal_id`: `[UUID]` |
| 5 | Emit `asset.device_disposal_logged` DEC-030 HMAC event | security-engineer | `[YYYY-MM-DD HH:MM UTC]` | DEC-030 event ID: `[event_id]` |

**BYOD exception:** If the individual used a personal device under BYOD and cannot physically return it, obtain and file `compliance/offboarding/[name]-[date]/byod-attestation-signed.pdf` confirming:
- No FORM production secrets retained on the device
- GitHub sessions revoked
- 1Password device removed
- Cloudflare browser sessions cleared

The attestation is a compensating control, not a substitute for physical sanitization. Disclose to auditors as such.

---

## Phase 3 — Completion

| # | Action | Owner | Completed at |
|---|---|---|---|
| 1 | All Phase 1 steps complete (verified by compliance-officer) | compliance-officer | `[YYYY-MM-DD HH:MM UTC]` |
| 2 | All Phase 2 steps complete or BYOD attestation filed | security-engineer | `[YYYY-MM-DD HH:MM UTC]` |
| 3 | Emit `personnel.offboarding_completed` DEC-030 HMAC event (STANDARD, 7yr) with `subject_name` (display name only), `offboarding_date`, `device_classes_sanitized` | security-engineer | `[YYYY-MM-DD HH:MM UTC]` |
| 4 | Update quarterly access review (`compliance/access-review/`) — confirm individual no longer appears in any system | compliance-officer | `[YYYY-MM-DD HH:MM UTC]` |
| 5 | File this completed checklist at `compliance/offboarding/[name]-[date]/offboarding-checklist.md` as CC6-E-009 | compliance-officer | `[YYYY-MM-DD HH:MM UTC]` |

---

## Completion Sign-Off

| Role | Name | Signature (git commit SHA of signed checklist) | Date |
|---|---|---|---|
| compliance-officer | `[Name]` | `[git commit SHA]` | `[YYYY-MM-DD]` |
| security-engineer | `[Name]` | `[git commit SHA]` | `[YYYY-MM-DD]` |

---

## Open Items / Exceptions

_List any step not completed within SLA, with reason and remediation plan:_

```
[None / describe exception]
```

---

*Template v1.0 · 2026-06-12 · Owner: compliance-officer + security-engineer*
*Source: `docs/SOC2_READINESS.md §40.5` · `compliance/c1/device-disposal-policy.md §3`*
