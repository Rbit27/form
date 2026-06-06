# FORM · Security Onboarding Checklist v1.0

| Field | Value |
|---|---|
| **Document title** | Security Onboarding Checklist |
| **Version** | v1.0 |
| **Status** | IN FORCE — effective from date of first hire |
| **Effective date** | 2026-06-06 |
| **Owner** | security-engineer |
| **Evidence artifact ID** | CC1-E-002 |
| **Gap IDs closed** | CC5-P2-007 (new-hire onboarding checklist) |
| **Cross-references** | `docs/SOC2_READINESS.md` §22, §49; `docs/ACCEPTABLE_USE_POLICY.md`; `docs/INCIDENT_RESPONSE.md`; `docs/CRYPTOGRAPHY_POLICY.md`; `docs/SECURITY.md` |
| **Last reviewed** | 2026-06-06 |
| **Next review due** | 2027-06-06 |

> **Scope.** This document applies to every person granted access to any FORM production system — engineers, contractors, advisors, and future enterprise-tier operators alike. During the solo-founder phase it is a forward-authored template executed as written when the second team member joins. The MFA requirements and SSH key standards apply to the founder retroactively from document creation date.

---

## 1. Purpose

This checklist is the authoritative security onboarding procedure for all new FORM team members. Its objectives are:

1. Ensure every person with production access meets the same baseline security posture before credentials are provisioned.
2. Create auditable evidence (CC1-E-002) that each control step was completed and verified.
3. Communicate the MFA enforcement policy — in particular the prohibition on SMS 2FA and the requirement for hardware keys or TOTP on all production systems.
4. Provide a structured first exposure to FORM's incident response obligations, data classification rules, and privacy floor commitments.

This document does not replace `docs/ACCEPTABLE_USE_POLICY.md` (which the new hire also signs as Step 1), `docs/INCIDENT_RESPONSE.md` (which is the operational guide), or `docs/SOC2_READINESS.md §22` (which documents the training programme). It is the executable checklist; those are the policy and programme documents.

---

## 2. Prerequisites

Before the checklist is initiated, the following must be in place:

| # | Prerequisite | Owner |
|---|---|---|
| P-1 | Offer letter or contractor agreement signed | founder |
| P-2 | Role defined and least-privilege access scope documented (see §6) | founder + security-engineer |
| P-3 | `docs/ACCEPTABLE_USE_POLICY.md` current version confirmed in force | compliance-officer |
| P-4 | 1Password Team seat provisioned (invite pending acceptance) | security-engineer |
| P-5 | New hire's work device confirmed (own machine or company-issued) | security-engineer |

No credential provisioning may begin before all five prerequisites are verified.

---

## 3. Eight-Step Onboarding Checklist

Steps are sequential. No step may be skipped or deferred. **Step 3 (MFA) is a hard gate: production credentials are not issued until MFA is verified by the security-engineer, not merely reported complete by the new hire.**

### Step 1 — AUP Acknowledgment

| Item | Detail |
|---|---|
| **Action** | New hire reads and signs `docs/ACCEPTABLE_USE_POLICY.md` v1.0 (or current version). |
| **Method** | PDF export with wet-signature or DocuSign electronic signature. Verbal acknowledgment is not sufficient. |
| **Evidence** | Signed PDF filed to `compliance/evidence/cc1/onboarding/[name]-[YYYY-MM-DD]-aup-signed.pdf` |
| **Owner** | security-engineer (witness); new hire (signatory) |
| **Verify before proceeding** | security-engineer confirms signed PDF is filed. |

Key AUP obligations the new hire is acknowledging:
- No credential sharing under any circumstances.
- Mandatory incident reporting — a suspected breach must be reported to the founder within 1 hour.
- No production data on personal unmanaged devices outside the approved toolchain.
- Clinical safety floor: HR-visible enterprise aggregate data must never include individual user health values (enforced at infrastructure level, but the new hire must understand the obligation).

---

### Step 2 — Security Awareness Training

| Item | Detail |
|---|---|
| **Action** | Complete all eight training topics per `docs/SOC2_READINESS.md §22.3`. |
| **Timeline** | May be spread across the first five business days, but must be complete before solo production access. |
| **Evidence** | Training completion artefacts (certificates, screenshots, signed attestations) filed to `compliance/evidence/cc1/onboarding/[name]-[YYYY-MM-DD]-training-complete/` |
| **Owner** | new hire (self-directed); security-engineer (verifies completion) |

Required training topics summary:

| # | Topic | Delivery |
|---|---|---|
| 1 | OWASP Top 10 | OWASP WebGoat completion screenshot |
| 2 | Phishing and social engineering | KnowBe4 or self-directed module with attestation |
| 3 | Data classification and handling | Internal document review + signed attestation |
| 4 | Incident response | Tabletop walkthrough with security-engineer |
| 5 | Credential hygiene | Policy review + hands-on MFA gate (Step 3) |
| 6 | Secure code review | Cloudflare Security track + PR checklist review |
| 7 | Privacy and GDPR | GDPR controller self-assessment + attestation |
| 8 | Vendor risk awareness | Subprocessor register review + attestation |

---

### Step 3 — MFA Enrollment (Hard Gate)

**No production credential is activated until this step is verified by the security-engineer.**

#### 3.1 MFA policy

| Rule | Detail |
|---|---|
| **SMS-based 2FA is prohibited** | SMS 2FA is subject to SIM-swap attacks. No `form-coach` GitHub organisation member may use `two_factor_method_sms`. The GitHub organisation audit log is checked for `two_factor_method_sms` events — any such event triggers immediate re-enrollment. |
| **Hardware keys are preferred** | YubiKey 5 NFC or equivalent FIDO2/WebAuthn hardware key is the recommended second factor for GitHub and 1Password. Passkeys (device-resident FIDO2) are also accepted. |
| **TOTP is the minimum** | Where a hardware key is not yet available, TOTP via an authenticator app (Authy, 1Password TOTP, Google Authenticator) is the minimum accepted second factor. TOTP secrets must be stored in 1Password — not as screenshots, not in cloud notes. |

#### 3.2 Per-platform enrollment procedure

**GitHub (`github.com/form-coach` organisation)**

```
1. Settings → Password and authentication → Two-factor authentication
2. Preferred: Add a security key (FIDO2 / WebAuthn) — YubiKey or Passkey
3. Acceptable fallback: Set up authenticator app (TOTP) — save secret to 1Password
4. DO NOT select "SMS": GitHub org policy will reject SMS 2FA
5. Save recovery codes to 1Password vault: "GitHub 2FA Recovery — [name]"
6. Security-engineer verifies: organisation member list shows 2FA enabled
   CLI: gh api orgs/form-coach/members --jq '.[] | select(.login=="[new-hire-login]")'
   Dashboard: org Settings → Members → 2FA column shows ✓
```

**1Password (Team plan)**

```
1. Accept vault invitation email
2. Install 1Password desktop app (required — browser extension only is insufficient)
3. Settings → Security → Two-Factor Authentication → Set up
4. Preferred: Hardware key (register YubiKey via WebAuthn)
5. Acceptable: Authenticator app (TOTP) — recovery codes saved to personal vault
6. Security-engineer verifies: Team admin console → Members → [name] shows MFA active
```

**Supabase (form-coach organisation)**

```
1. supabase.com → Account → Security
2. Enable Two-Factor Authentication → Authenticator app (TOTP)
3. Save TOTP secret to 1Password vault: "Supabase 2FA — [name]"
4. Save recovery codes to 1Password: "Supabase 2FA Recovery — [name]"
5. Security-engineer verifies: organisation member list shows MFA status
   (Supabase does not yet support hardware keys — TOTP is the maximum)
```

**Cloudflare (form.coach account)**

```
1. Cloudflare dashboard → My Profile → Authentication
2. Manage two-factor authentication → Enable
3. Preferred: Security key (FIDO2 / hardware key)
4. Acceptable: Authenticator app (TOTP)
5. Save backup codes to 1Password: "Cloudflare 2FA Backup — [name]"
6. Security-engineer verifies: Account members tab → 2FA column
```

#### 3.3 Verification evidence

After verifying MFA on all platforms, the security-engineer files the following:

```
compliance/evidence/cc1/onboarding/[name]-[YYYY-MM-DD]-mfa-verification/
  github-mfa-enabled-screenshot.png
  1password-mfa-enabled-screenshot.png
  supabase-mfa-enabled-screenshot.png
  cloudflare-mfa-enabled-screenshot.png
  security-engineer-attestation.md   ← signed attestation of verification
```

Attestation template:

```
I, [security-engineer name], confirm that I have personally verified MFA enrollment for
[new hire name] on the following platforms on [YYYY-MM-DD]:

  ✓ GitHub — [hardware key / TOTP] — org member list shows 2FA: enabled
  ✓ 1Password — [hardware key / TOTP] — admin console shows MFA: active
  ✓ Supabase — TOTP — organisation member shows MFA: active
  ✓ Cloudflare — [hardware key / TOTP] — account member shows 2FA: active
  ✓ Confirmed: no two_factor_method_sms events in GitHub org audit log for this member

SMS 2FA: NOT active on any platform.
Hardware key enrolled: [YES / NO — TOTP only at this time]

Signed: [security-engineer name]
Date: [YYYY-MM-DD]
```

---

### Step 4 — 1Password Setup

| Item | Detail |
|---|---|
| **Action** | Confirm password manager is operational and no credentials are stored outside 1Password. |
| **Owner** | security-engineer (spot-check); new hire |

Spot-check procedure performed by security-engineer (with new hire's consent and presence):

```bash
# Check for plain-text env files with production values in home directory
find ~ -name "*.env" -o -name ".env*" 2>/dev/null | head -20

# Check for SSH config pointing to plain-text key files outside ~/.ssh
cat ~/.ssh/config 2>/dev/null | grep IdentityFile

# Check for credentials in shell history (sanitised review — no scrolling through history)
grep -E "(ANTHROPIC|SUPABASE|STRIPE|CLOUDFLARE|SENTRY)_.*=.{10}" ~/.bash_history ~/.zsh_history 2>/dev/null | wc -l
# Expected: 0. If non-zero, rotate affected keys immediately and update 1Password.
```

If any finding is identified:
1. Rotate the exposed credential immediately (follow `docs/KEY_ROTATION.md` for the relevant key type).
2. File a low-severity incident per `docs/INCIDENT_RESPONSE.md §R-03` (Compromised Credentials).
3. Document the finding and rotation in the onboarding record.

Evidence filed: security-engineer spot-check attestation at `compliance/evidence/cc1/onboarding/[name]-[YYYY-MM-DD]-1password-spotcheck.md`.

---

### Step 5 — SSH Key Generation and Registration

| Item | Detail |
|---|---|
| **Action** | Generate a new SSH key pair on the work machine; register with GitHub. |
| **Owner** | new hire (generates); security-engineer (verifies algorithm) |

**Key generation:**

```bash
# Ed25519 is required minimum. RSA 4096 is acceptable if Ed25519 is unavailable.
# DO NOT generate RSA 2048 or ECDSA P-256 keys for production use.
ssh-keygen -t ed25519 -C "[name]@form.coach-[YYYY-MM-DD]" -f ~/.ssh/id_ed25519_form

# Set a strong passphrase (≥ 20 characters, stored in 1Password vault: "SSH Key Passphrase — form — [name]")
# The passphrase must be stored in 1Password only — not written down, not in notes
```

**Registration:**

```
GitHub → Settings → SSH and GPG keys → New SSH key
Title: "form-[machinename]-[YYYY-MM-DD]"
Key type: Authentication Key
Key: [contents of ~/.ssh/id_ed25519_form.pub]
```

**Verification by security-engineer:**
- Confirm key type is `ssh-ed25519` (or `ssh-rsa` with 4096-bit — acceptable but note in record).
- Confirm key is listed in GitHub under new hire's account.
- Confirm private key is passphrase-protected: `ssh-keygen -y -f ~/.ssh/id_ed25519_form` (will prompt for passphrase — if it doesn't, the key is unprotected).

Evidence: Screenshot of GitHub SSH keys page filed to `compliance/evidence/cc1/onboarding/[name]-[YYYY-MM-DD]-ssh-key.png`.

---

### Step 6 — Access Provisioning (Least Privilege)

| Item | Detail |
|---|---|
| **Action** | Provision access only to systems required for the role; log in access control matrix. |
| **Owner** | security-engineer |

**Role-based access scope:**

| Role | GitHub | Supabase | Cloudflare | 1Password | PostHog | Sentry | Anthropic |
|---|---|---|---|---|---|---|---|
| Founding Engineer | Write (main branch via PR only) | `anon` + `service_role` via env | Zone Editor (form.coach only) | Shared Engineering vault | Read (product analytics) | Member | API key (staging only until verified) |
| ML Engineer | Write (CV branches) | `anon` only | No access | Shared Engineering vault | Read | Member | API key (ML pipeline only) |
| Compliance Officer | Read | No direct DB access | No access | Shared Compliance vault | No access | No access | No access |
| Customer Success | No access | No access | No access | Shared CS vault | Customer-facing dashboards only | No access | No access |

**Access provisioning checklist:**

```
[ ] GitHub organisation invitation sent and accepted (role: Member, not Owner)
[ ] Supabase organisation invitation: appropriate role assigned
[ ] Cloudflare team invitation: role scoped to minimum required
[ ] 1Password vault(s) shared: only vaults required for role
[ ] PostHog invitation: appropriate project access
[ ] Sentry invitation: Member role
[ ] Linear invitation: role appropriate to function
[ ] Anthropic Console: project-scoped API key issued (not org admin)
```

Evidence: Access control matrix updated in `docs/SOC2_READINESS.md §26` + Linear ticket linking provisioning record.

---

### Step 7 — Incident Response Orientation

| Item | Detail |
|---|---|
| **Action** | 30-minute walkthrough of `docs/INCIDENT_RESPONSE.md` with security-engineer. |
| **Owner** | security-engineer (facilitates) |

**Walkthrough agenda:**

| # | Topic | Section |
|---|---|---|
| 1 | Severity classification (P0–P3) and what each level means for you | §2 |
| 2 | How to report a suspected incident — Slack channel, PagerDuty, founder direct | §3 |
| 3 | What NOT to do during an active incident | §4.2 "Do not" list |
| 4 | The two runbooks most likely to apply in your first 90 days | R-03 (credentials), R-08 (supply chain) |
| 5 | Privacy floor — why you must never attempt to read individual user health data, even to debug | `docs/ENTERPRISE.md` §Privacy floor |
| 6 | GDPR breach notification timeline — 72 hours to supervisory authority, 1 hour to founder | §10 |

At the end of the walkthrough, the new hire signs:

```
I, [name], confirm that I have reviewed docs/INCIDENT_RESPONSE.md v[version] on [YYYY-MM-DD]
with [security-engineer name] and understand:
  - How to classify and report a suspected security incident
  - The escalation path for P0 incidents
  - My obligations under GDPR Art. 33 breach notification rules
  - The enterprise privacy floor — no individual user health data access

Signed: [name]
Date: [YYYY-MM-DD]
```

Evidence filed: signed attestation at `compliance/evidence/cc1/onboarding/[name]-[YYYY-MM-DD]-ir-orientation.md`.

---

### Step 8 — 30-Day Check-in

| Item | Detail |
|---|---|
| **Action** | Security-engineer schedules a 30-day follow-up meeting. |
| **Trigger** | Calendar event created on Day 1, set for Day 30 ± 3 days. |
| **Owner** | security-engineer |

**30-day check-in agenda:**

1. Any credential questions or exposures discovered since onboarding?
2. Any security incidents observed or suspected (even if not reported)?
3. Access scope review — is the access granted still appropriate? Has the role changed?
4. MFA still active on all platforms? (Re-verify screenshots if any doubt.)
5. Open questions about IR runbooks, privacy floor, or data classification?

**Outcome logging:** Linear ticket with outcome summary, any access changes logged in access control matrix.

---

## 4. Completion Record

When all eight steps are complete, the security-engineer creates the completion record:

**File location:** `compliance/evidence/cc1/onboarding/[name]-[YYYY-MM-DD]-onboarding-complete.md`

**Template:**

```markdown
# Security Onboarding Completion Record

| Field | Value |
|---|---|
| **New team member** | [name] |
| **Role** | [role] |
| **Onboarding date** | [YYYY-MM-DD] |
| **Checklist version** | docs/ONBOARDING_SECURITY.md v1.0 |
| **Security-engineer** | [name] |

## Step Completion

| Step | Completed | Evidence filed | Notes |
|---|---|---|---|
| 1 — AUP acknowledgment | [YYYY-MM-DD] | `…-aup-signed.pdf` | |
| 2 — Security awareness training | [YYYY-MM-DD] | `…-training-complete/` | |
| 3 — MFA enrollment | [YYYY-MM-DD] | `…-mfa-verification/` | [hardware key / TOTP] |
| 4 — 1Password spot-check | [YYYY-MM-DD] | `…-1password-spotcheck.md` | |
| 5 — SSH key generation | [YYYY-MM-DD] | `…-ssh-key.png` | [ed25519 / rsa4096] |
| 6 — Access provisioning | [YYYY-MM-DD] | Linear ticket [link] | |
| 7 — IR orientation | [YYYY-MM-DD] | `…-ir-orientation.md` | |
| 8 — 30-day check-in | [YYYY-MM-DD] | Linear ticket [link] | Scheduled |

## MFA Summary

- GitHub: [hardware key / TOTP] — SMS: NOT active ✓
- 1Password: [hardware key / TOTP] ✓
- Supabase: TOTP ✓
- Cloudflare: [hardware key / TOTP] ✓
- Hardware key enrolled: [YES / NO]

## Access Granted

[List of systems and access level granted]

## Security-Engineer Sign-Off

I confirm all eight steps have been completed and verified.

Signed: [security-engineer name]  
Date: [YYYY-MM-DD]
```

---

## 5. MFA Enforcement Policy (Authoritative)

This section is the authoritative statement of FORM's MFA enforcement policy. It satisfies the requirement in `docs/SOC2_READINESS.md §49.2.2` that MFA policy "must be communicated in the onboarding checklist."

### 5.1 Mandatory second factors (in order of preference)

| Preference | Method | Notes |
|---|---|---|
| **1 (Best)** | FIDO2 hardware key (YubiKey 5 NFC, Passkey) | Phishing-resistant. Required for anyone with `admin:org` GitHub access or Cloudflare Super Admin access. |
| **2** | Passkey (device-resident FIDO2 / WebAuthn) | Phishing-resistant. Acceptable on platforms that support it. |
| **3** | TOTP authenticator app | Authy, 1Password TOTP, Google Authenticator — all acceptable. TOTP secret stored in 1Password. |
| **❌ Prohibited** | SMS-based 2FA | SIM-swap risk. GitHub org policy will reject this; if somehow active, it is a P1 finding. |
| **❌ Prohibited** | Email-based OTP | Phishing risk. Not accepted as a second factor on any FORM system. |

### 5.2 Enforcement and audit

GitHub organisation MFA audit (run at each quarterly access review, `docs/SOC2_READINESS.md §23`):

```bash
# List org members without 2FA enabled
gh api orgs/form-coach/members?filter=2fa_disabled --jq '.[].login'
# Expected: empty list. Any result is a P1 finding requiring same-day remediation.

# Verify no SMS 2FA in org audit log (last 90 days)
gh api orgs/form-coach/audit-log \
  --jq '.[] | select(.action == "org.update_member_repository_invitation_permission" or .action == "two_factor_disabled" or .action | startswith("two_factor"))' | head -50
```

### 5.3 Exceptions and break-glass

No permanent exceptions to the MFA requirement are permitted. Temporary exceptions (e.g., hardware key lost, TOTP device failure) follow:

1. New hire reports lost/broken second factor to security-engineer.
2. Security-engineer revokes production access immediately.
3. New hire re-enrolls a new second factor (new hardware key or new TOTP) — this must be completed before access is restored.
4. New second factor enrollment is documented as an addendum to the original onboarding completion record.
5. DEC-030 `system.credential_rotated` event emitted (if a key rotation was involved) per `docs/AUDIT_LOG_SCHEMA.md`.

---

## 6. Access Scope Principles

Access is provisioned on three principles, each non-negotiable:

**Least privilege.** No new hire receives more access than required for their stated role at the time of onboarding. Access that may be needed later is not provisioned speculatively.

**Time-bound review.** All access is reviewed at the quarterly access review (`docs/SOC2_READINESS.md §23`) and at the 30-day check-in. Unused access is revoked.

**Privacy floor.** No team member — regardless of role — is provisioned with access that would allow direct read of individual user health data outside of an approved incident response or compliance process. Even during an approved access, the privacy floor in `docs/ENTERPRISE.md §Privacy floor` governs what can be read and how. Any access to enterprise tenant user data requires a PAM session (`docs/SSO_SCIM_IMPLEMENTATION.md §24`).

---

## 7. Annual Refresh

After initial onboarding, each team member completes an annual security refresher per `docs/SOC2_READINESS.md §22.6`:

| Cadence | Requirement |
|---|---|
| **Q1 (February, annual)** | Full review of all eight training topics (§22.3); fresh certificates or attestations required |
| **Q3 (August, annual)** | OWASP refresher — review any new CVEs affecting FORM's stack; signed attestation |
| **Ad-hoc (post-incident)** | Targeted refresher within 14 days if any incident reveals a training gap |
| **On material policy change** | AUP re-signed whenever `docs/ACCEPTABLE_USE_POLICY.md` is updated materially |
| **On hardware key loss/replacement** | New key enrolled; documented as addendum to completion record |

---

## 8. SOC 2 Evidence Mapping

| SOC 2 Criterion | Control description | How this document satisfies it | Evidence artefact |
|---|---|---|---|
| **CC1.1** — Commitment to integrity and ethical values | AUP signed before access; no-credential-sharing policy enforced | Step 1 (AUP) + Step 4 (1Password spot-check) | CC1-E-004 (AUP signed), CC1-E-002 (completion record) |
| **CC1.4** — Competence and training | Security awareness training completed before production access | Step 2 (training completion) + Step 7 (IR orientation) | CC1-E-001 (training records), CC1-E-002 (completion record) |
| **CC5.3** — Policies deployed via procedures | Onboarding checklist is the deployment mechanism for AUP, IR policy, and data classification | This document as executed; each step references the governing policy document | CC1-E-002 |
| **CC6.1** — Logical access security controls | Least-privilege provisioning; MFA hard gate before credential issuance | Step 3 (MFA hard gate) + Step 6 (access provisioning) | CC1-E-002 + access control matrix |
| **CC6.2** — Authentication credentials | MFA policy: hardware key / TOTP only; SMS prohibited; TOTP secrets in 1Password | §5 MFA Enforcement Policy; Step 3 per-platform instructions | CC1-E-002 mfa-verification/ |
| **CC6.3** — Access removal on change | 30-day check-in for access scope review; access control matrix updated on any change | Step 8 (30-day check-in) + §6 (access scope principles) | Linear ticket + access matrix update |
| **CC6.8** — Controls protect against unauthorized changes | SSH key passphrase required; no plain-text credentials outside 1Password | Step 4 (spot-check) + Step 5 (SSH key + passphrase) | CC1-E-002 ssh-key.png + 1password-spotcheck.md |

---

## 9. DEC-030 Audit Events

The following DEC-030 HMAC-chained events are relevant to onboarding. They are registered in `docs/AUDIT_LOG_SCHEMA.md`.

| Event | Trigger | Severity | Retention | Notes |
|---|---|---|---|---|
| `system.access_review_completed` | Quarterly access review executed | STANDARD | 7 years | Covers new hire access in scope of review; quarterly, not per-hire |
| `system.credential_rotated` | SSH key, API key, or 2FA credential replaced | STANDARD | 7 years | Emitted whenever a credential is rotated as part of onboarding or 30-day check-in |

No onboarding-specific DEC-030 event exists yet. CC5-P2-007 implementation checklist item 4 (§27 of `docs/SOC2_READINESS.md`) tracks the decision on whether to register an `ops.team_member_onboarded` event.

---

## 10. Off-Boarding Reference

Off-boarding is the inverse of this checklist and is documented in `compliance/cc6/offboarding-procedure.md`. Key difference: off-boarding must be completed within **4 hours** of departure (same-day for any involuntary separation). Access revocation is not sequential — all platforms are revoked simultaneously. This document does not govern off-boarding; it is referenced here for awareness.

---

## 11. Implementation Checklist

| # | Task | Owner | Priority | Milestone | Definition of Done |
|---|---|---|---|---|---|
| 1 | Execute this checklist for the founder as a self-attestation (solo phase): complete Steps 1–7 as solo-founder; file evidence to `compliance/evidence/cc1/onboarding/founder-[YYYY-MM-DD]-*` | founder | **P0** | Before SOC 2 observation period | Founder completion record filed; CC1-E-002 partially satisfied (solo phase note documented) |
| 2 | Procure YubiKey 5 NFC (or equivalent FIDO2 hardware key) for the founder and all future hires | founder | **P1** | Before first hire | Receipt filed; key enrolled on GitHub + 1Password + Cloudflare; hardware key attestation in completion record |
| 3 | Execute this checklist in full for the first new hire | security-engineer | **P0** | Day 1 of first hire | Complete completion record at `compliance/evidence/cc1/onboarding/` |
| 4 | Create `compliance/evidence/cc1/onboarding/` directory in private compliance repository | security-engineer | **P0** | Before first hire | Directory exists; README explaining naming convention committed |
| 5 | Register `ops.team_member_onboarded` DEC-030 event type in `docs/AUDIT_LOG_SCHEMA.md` | platform-engineer + security-engineer | **P1** | M4 | Event registered with Zod schema; emittable from admin Worker; test event in staging |
| 6 | Add hardware key (YubiKey / Passkey) support to WorkOS tenant SSO for enterprise team members | platform-engineer | **P1** | M6 | Enterprise CSM can enforce hardware-key-only MFA for their tenant via admin dashboard policy |

---

## 12. Open Questions

| ID | Question | Owner | Priority | Resolution target |
|---|---|---|---|---|
| OQ-ONB-01 | **Should the 30-day check-in (Step 8) be formally registered as a DEC-030 event?** Recording check-in completion in the audit log would provide continuous evidence for CC6.3 (access review on change). Cost: additional DEC-030 event type. Benefit: auditor-visible cadence without manual evidence extraction. Recommendation: register `ops.access_scope_review_completed` event type alongside `ops.team_member_onboarded` (checklist item 5). | security-engineer + platform-engineer | **P2** | M5 |
| OQ-ONB-02 | **What is the hardware key procurement policy for contractors with access ≥ 30 days?** The current policy (§5) requires hardware keys for `admin:org` GitHub and Cloudflare Super Admin roles. But a contractor with `service_role` Supabase access and write access to `main` has a significant blast radius without a hardware key. Recommendation: require hardware key for any person with Supabase `service_role` or Cloudflare Zone Admin or higher, regardless of contractor status. | security-engineer + compliance-officer | **P1** | Before first contractor hire |
| OQ-ONB-03 | **Enterprise customer IT administrators: does this checklist apply to WorkOS-federated enterprise team members accessing only their own tenant's admin dashboard?** Enterprise tenant admins authenticate exclusively via the customer's IdP (SSO) and never receive a FORM-issued credential. The MFA requirement for them is enforced at the IdP level per `docs/SSO_SCIM_IMPLEMENTATION.md §25.1`. This checklist applies to internal FORM team members only. Confirmation needed before enterprise GA to ensure the customer CSM onboarding script does not inadvertently reference this internal checklist. | customer-success + security-engineer | **P2** | Before enterprise GA (M13) |

---

*v1.0 · 2026-06-06 · Closes `docs/SOC2_READINESS.md §49.2.2` cross-reference (hardware key + no-SMS MFA policy "must be communicated in the onboarding checklist") and `docs/SOC2_READINESS.md` CC5-P2-007 (new-hire onboarding checklist). Provides evidence artifact CC1-E-002 template. Owner: security-engineer + compliance-officer.*
