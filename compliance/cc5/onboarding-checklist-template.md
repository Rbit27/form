# FORM · New-Hire Security Onboarding Checklist

> SOC 2 Type II · CC5-P2-007 · Closes CC5-GAP onboarding evidence gap.
> Owner: compliance-officer. Review: annually + on any policy change.
> Reference: `docs/ACCEPTABLE_USE_POLICY.md`, `compliance/policy-approval-log.csv`, `docs/ONBOARDING_SECURITY.md`.
> Status: Template — complete per hire and archive signed copy.

---

## Instructions

1. Complete one copy of this checklist per new hire on their first day (or within 24 hours).
2. Both the new hire and the compliance-officer/founder must sign (git-commit SHA as signature for remote hires).
3. Archive completed copy to `compliance/evidence/cc1/onboarding/[name]-[YYYY-MM-DD]-onboarding.md`.
4. Update `compliance/cc1/README.md` evidence index with the new entry.
5. This checklist satisfies CC5.3 (policy communication) and CC1.4 (HR practices for new hires).

---

## New Hire Information

| Field | Value |
|---|---|
| **Name** | _(fill)_ |
| **Role** | _(fill)_ |
| **Start date** | _(fill)_ |
| **Checklist completed** | _(YYYY-MM-DD)_ |
| **Completed by** | _(compliance-officer name)_ |
| **Archive path** | `compliance/evidence/cc1/onboarding/[name]-[date]-onboarding.md` |

---

## Section 1: Identity and Access Provisioning

| # | Item | Done | Notes |
|---|---|---|---|
| 1.1 | GitHub account (personal) added to `rbit27/form` with minimum required role (Contributor unless otherwise specified in role JD) | ☐ | |
| 1.2 | GitHub 2FA (TOTP or hardware key) verified enabled on personal GitHub account before repo access granted | ☐ | |
| 1.3 | 1Password team invite sent and accepted; shared vault access confirmed | ☐ | |
| 1.4 | Supabase project access provisioned with role-appropriate permissions (read-only for non-engineers by default) | ☐ | |
| 1.5 | Cloudflare access: if applicable to role, sub-account access provisioned | ☐ | N/A if not applicable |
| 1.6 | WorkOS dashboard access: if applicable to role | ☐ | N/A if not applicable |
| 1.7 | Anthropic Console access: if applicable to role (ML/platform engineers only) | ☐ | N/A if not applicable |
| 1.8 | All credentials issued via 1Password shared vault, **not** via Slack/email plaintext | ☐ | |
| 1.9 | Principle of least privilege confirmed: access matches role requirements, no standing admin without documented justification | ☐ | |

---

## Section 2: Device and Endpoint Security

| # | Item | Done | Notes |
|---|---|---|---|
| 2.1 | Work device (company-issued or BYOD) confirmed with full-disk encryption enabled (FileVault / BitLocker / LUKS) | ☐ | |
| 2.2 | Auto-lock configured: screen lock after ≤5 minutes of inactivity | ☐ | |
| 2.3 | OS auto-updates enabled | ☐ | |
| 2.4 | Password manager (1Password) installed and active on work device | ☐ | |
| 2.5 | VPN policy briefed: when VPN required (public Wi-Fi, external API access from untrusted network) | ☐ | |
| 2.6 | MDM enrolment: if company MDM implemented at this stage | ☐ | N/A if pre-MDM |

---

## Section 3: Policy Acknowledgments (12 Policies — POL-001 through POL-011)

New hire has read and signed each policy. Signature = git commit SHA on this completed checklist file.

| # | Policy | Document | Version | Acknowledged |
|---|---|---|---|---|
| 3.1 | POL-001 · Acceptable Use Policy | `docs/ACCEPTABLE_USE_POLICY.md` | v1.0 | ☐ |
| 3.2 | POL-002 · Cryptography Policy | `docs/CRYPTOGRAPHY_POLICY.md` | v1.0 | ☐ |
| 3.3 | POL-003 · Data Classification Policy | `docs/DATA_MODEL.md §data-classification` | v1.0 | ☐ |
| 3.4 | POL-004 · Incident Response Policy | `docs/INCIDENT_RESPONSE.md` | v1.0 | ☐ |
| 3.5 | POL-005 · Access Control Policy | `docs/SSO_SCIM_IMPLEMENTATION.md §access-control` | v1.0 | ☐ |
| 3.6 | POL-006 · Vulnerability Management Policy | `compliance/cc7/vuln-management-policy.md` | v1.0 | ☐ |
| 3.7 | POL-007 · Change Management Policy | `compliance/cc8/change-management-policy.md` | v1.0 | ☐ |
| 3.8 | POL-008 · Business Continuity Policy | `docs/BUSINESS_CONTINUITY.md` | v1.0 | ☐ |
| 3.9 | POL-009 · Privacy Policy (internal awareness) | `docs/PRIVACY_POLICY.md` | v1.0 | ☐ |
| 3.10 | POL-010 · GDPR/Data Protection Obligations | `docs/GDPR_DPIA.md` | v1.0 | ☐ |
| 3.11 | POL-011 · Vendor Management Policy | `docs/VENDOR_REGISTRY.md §vendor-management` | v1.0 | ☐ |
| 3.12 | Code of Conduct | `CONTRIBUTING.md §code-of-conduct` | current | ☐ |

---

## Section 4: Security Awareness Briefing

Delivered verbally (or via async video for remote hires) by compliance-officer/founder. Topics covered:

| # | Topic | Covered | Notes |
|---|---|---|---|
| 4.1 | Phishing awareness: how to identify phishing attempts; what to do if suspected (report to founder immediately via Signal/1Password) | ☐ | |
| 4.2 | Credential hygiene: no password reuse, all credentials via 1Password, never in plaintext messages | ☐ | |
| 4.3 | Secret handling: no secrets in git commits, environment variables only via 1Password or CI secrets manager; TruffleHog CI will flag any committed credential | ☐ | |
| 4.4 | Data handling: what constitutes user health data and how it must be handled; no copying user data to local devices without explicit written approval | ☐ | |
| 4.5 | Incident reporting: what to escalate, to whom, within what timeframe (see POL-004 §escalation-path) | ☐ | |
| 4.6 | Break-glass procedure: what it is, who has it, when it's used | ☐ | N/A if not applicable to role |
| 4.7 | Social engineering awareness: do not disclose internal tooling, customer data or infrastructure to external parties without explicit approval | ☐ | |
| 4.8 | Clear desk policy: no sensitive data left on screen unattended in shared/public spaces | ☐ | |

---

## Section 5: Offboarding Pre-briefing (Acknowledge at Onboarding)

New hire acknowledges that upon separation:

| # | Item | Acknowledged |
|---|---|---|
| 5.1 | All company credentials and access will be revoked within 4 hours of separation notice per `compliance/cc6/offboarding-procedure.md` | ☐ |
| 5.2 | Any locally downloaded company data must be deleted and deletion confirmed | ☐ |
| 5.3 | 1Password company vault access will be revoked; personal vault unaffected | ☐ |
| 5.4 | GitHub org membership will be removed | ☐ |

---

## Section 6: Completion Sign-off

| Field | Value |
|---|---|
| **New hire signature** | _(git commit SHA of this completed file, or wet signature if in-person)_ |
| **Compliance officer signature** | _(git commit SHA or name)_ |
| **Date completed** | _(YYYY-MM-DD)_ |
| **Archived to** | `compliance/evidence/cc1/onboarding/[name]-[date]-onboarding.md` |
| **Checklist version** | v1.0 |

---

## Notes and Exceptions

_(Document any items marked N/A with rationale, or any items that could not be completed on day 1 with target completion date.)_

---

## Compliance Coverage

| SOC 2 Criterion | How this checklist satisfies it |
|---|---|
| CC1.4 | Demonstrates HR practices: new hires are briefed on security responsibilities and policies before system access |
| CC5.3 | Evidence that policies (POL-001 through POL-011) are communicated to all personnel |
| CC6.1 | Minimum access provisioned based on role; least privilege documented |
| CC6.2 | Authentication controls (GitHub 2FA, 1Password) confirmed before access granted |
| CC6.3 | AUP acknowledgment prior to system access |

---

*v1.0 · 2026-06-07 · Owner: compliance-officer*
*Closes: CC5-P2-007 (new-hire security onboarding checklist template)*
*Archive completed copies to: `compliance/evidence/cc1/onboarding/`*
