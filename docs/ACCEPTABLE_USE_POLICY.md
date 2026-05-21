# FORM Acceptable Use Policy

| Field | Value |
|---|---|
| **Document title** | Acceptable Use Policy |
| **Version** | v1.0 |
| **Status** | AUTHORED — PENDING EXTERNAL COUNSEL REVIEW |
| **Effective date** | TBD — takes effect on founder signature and completion of external counsel review |
| **Owner** | compliance-officer |
| **Evidence artifact ID** | CC1-E-004 |
| **Gap IDs closed** | CC1-GAP-001, CC5-GAP-001 |
| **Cross-references** | `docs/SOC2_READINESS.md` §1, §22.5, §26, §27; `docs/INCIDENT_RESPONSE.md`; `docs/AUDIT_LOG_SCHEMA.md`; `docs/SECURITY.md`; `docs/GDPR_DPIA.md` |
| **Replaces** | `docs/SOC2_READINESS.md` §27 Annex A (v0.1-draft) |
| **Last reviewed** | 2026-05-21 |
| **Next review due** | 2027-05-21 |

> **NOTICE.** This document is authored and filed as a compliance artifact per the SOC 2 gap registry. It is NOT yet in legal effect. It takes effect only upon (a) completion of external counsel review by qualified privacy/employment counsel covering EU, US, and Ukrainian jurisdictions, and (b) founder signature on the signatory block in Section 13. Until both conditions are met, this document is a design artifact, not an enforceable policy.

---

## 1. Purpose

This Acceptable Use Policy (AUP) defines the permitted and prohibited uses of FORM information systems, infrastructure, data, and tooling by any individual granted access to those systems. Its objectives are:

1. To protect the confidentiality, integrity, and availability (CIA) of FORM systems and services.
2. To protect the personal data of FORM users — in particular health data constituting special-category data under GDPR Article 9 — from unauthorized access, misuse, and unlawful processing.
3. To satisfy the AICPA SOC 2 Trust Service Criteria CC1.1 (Commitment to Competence) and CC5.3 (Deployment of Control Activities through Policies), and to serve as primary evidence artifact CC1-E-004.
4. To establish unambiguous behavioral standards against which access decisions, disciplinary actions, and regulatory assessments can be made.

This policy does not replace other FORM security and compliance documents. It operates alongside the Access Control Policy (`docs/SOC2_READINESS.md` §26), Cryptography Policy (pending — CC5-GAP-002), Incident Response Plan (`docs/INCIDENT_RESPONSE.md`), and GDPR DPIA (`docs/GDPR_DPIA.md`). Where those documents impose stricter requirements, those requirements prevail.

---

## 2. Scope

### 2.1 Persons in scope

This policy applies to all individuals granted access to any FORM system, including but not limited to:

- Founders and co-founders
- Employees (full-time and part-time)
- Contractors, consultants, and freelancers
- Advisors with any system access
- Third-party vendors with delegated access under a signed Data Processing Agreement (DPA)

Access to any FORM system, in any capacity, constitutes acceptance of this policy. Any person who cannot accept this policy must not be granted access and must immediately notify compliance-officer.

### 2.2 Systems in scope

| System | Category |
|---|---|
| Cloudflare dashboard, Workers, R2, WAF, Access, Pages, WARP | Cloud edge infrastructure |
| Supabase dashboard, Postgres, Auth, Vault, Storage | Database and identity |
| GitHub organisation (`form-app` org and all repositories) | Version control and CI/CD |
| PostHog | Product analytics |
| Sentry | Error monitoring and alerting |
| 1Password (all vaults) | Credential and secret storage |
| AWS (any account, present or future) | Cloud infrastructure |
| ElevenLabs | Text-to-speech subprocessor |
| Anthropic | LLM inference subprocessor |
| Stripe | Payment processing |
| Linear | Project and issue tracking |
| Slack (company workspace) | Internal communications |
| Apple Developer account | Mobile distribution |
| Google Play account | Mobile distribution |
| Any MDM-enrolled device used to access the above | End-user device |
| Any personal device approved in writing by security-engineer for FORM access | End-user device (BYOD exception) |
| Internal developer tooling, scripts, and automation accounts | Developer tooling |

This list is not exhaustive. When in doubt, a system should be treated as in scope.

---

## 3. Definitions

| Term | Definition |
|---|---|
| **FORM Systems** | All hardware, software, network infrastructure, cloud services, SaaS platforms, and data stores listed in Section 2.2 or otherwise used to build, operate, or support FORM products and services. |
| **Health Data** | Any data constituting special-category personal data under GDPR Article 9(1), including but not limited to: fitness metrics, heart rate, workout history, body composition data, sleep data, injury or recovery notes, and any inferences derived from the foregoing. Health Data is classified as `data_class: restricted` in the DEC-030 audit schema. |
| **Credential** | Any authenticator granting access to a FORM System, including passwords, API keys, bearer tokens, OAuth refresh tokens, SSH private keys, Cloudflare API tokens, Supabase service role keys, and hardware security key recovery codes. |
| **Authorized User** | A natural person explicitly listed in the Access Control Policy (`docs/SOC2_READINESS.md` §26) with a named role, defined access scope, and a signed AUP acknowledgment on file at `/evidence/cc1/aup/`. |
| **Break-Glass Access** | Emergency escalated access to production systems or restricted data outside the normal least-privilege access path. Break-Glass Access requires a pre-existing open incident record (`incident_id`) and is subject to mandatory DEC-030 audit logging before query execution. |
| **DEC-030 Audit Chain** | FORM's HMAC-chained tamper-evident audit log, defined in `docs/AUDIT_LOG_SCHEMA.md`. Every access event, data operation, and system action emits a structured record into this chain. Tampering with, suppressing, or forging DEC-030 records is a P0 security incident. |
| **Operational Need** | A documented, current business reason for accessing a specific system or data category, traceable to an active support ticket, Linear issue, incident record, DSAR request, or written approval from compliance-officer. Operational Need is not self-assessed; it must be externally verifiable in the audit record. |
| **Material Change** | Any revision to this policy that alters the scope of permitted uses, prohibited uses, health data handling rules, or enforcement consequences. Version number increments and clarifying edits that do not change behavioral requirements are not Material Changes. |

---

## 4. Acceptable Uses

The following uses of FORM Systems are permitted. All permitted uses are subject to the constraints and requirements in Sections 5 through 9. Items not listed here require written pre-approval from compliance-officer before access proceeds.

### 4.1 Assigned duties

Accessing FORM Systems for the purpose of performing assigned job responsibilities as defined in the Access Control Policy role matrix (`docs/SOC2_READINESS.md` §26.2). Access must not exceed the permissions defined for the assigned role.

### 4.2 Development, testing, and deployment

Using FORM development and production infrastructure to build, test, deploy, and operate FORM products and services, subject to the change management procedures in `docs/SOC2_READINESS.md` §21. All deployments to production must pass CI/CD gates and must not bypass merge protection rules on the `main` branch.

### 4.3 Health Data access with documented Operational Need

Accessing user Health Data where a specific, documented Operational Need exists. Acceptable triggering events include: an active user support ticket referencing the affected user account; a DSAR request formally logged in the DSAR register; an open incident record (`incident_id`) in `docs/INCIDENT_RESPONSE.md`; or written pre-authorization from compliance-officer for a defined investigation scope. Every Health Data access event must emit a `data.read_individual` DEC-030 audit event with the `operational_need_ref` field populated.

### 4.4 GDPR Data Subject Access Request (DSAR) fulfillment

Downloading, assembling, and delivering user data exports in response to a valid DSAR, within the 30-day statutory SLA. DSAR operations must log a `data.export_initiated` event to the DEC-030 audit chain. DSAR fulfillment is a permitted export exception; the export must be delivered only to the verified data subject and not retained internally beyond the delivery confirmation.

### 4.5 Business communication on approved channels

Using FORM-approved communication channels — Slack, Linear, GitHub Issues, and GitHub Discussions — for business purposes. Personal use of approved channels is permitted at low volume provided it does not transmit FORM confidential data, user personal data, or system credentials to parties outside the authorized roster.

### 4.6 Authorized backup and audit evidence operations

Downloading or copying FORM data for authorized backup procedures, DSAR fulfillment, or audit evidence packaging, provided that the operation is logged in the DEC-030 audit chain (`data.export_initiated` or `system.backup_initiated` as applicable) and that exported data is stored only in locations listed in the subprocessor register (`docs/SECURITY.md §5`) or the designated private compliance repository.

### 4.7 Break-Glass emergency access

Accessing production systems or restricted data via the Break-Glass procedure when an active incident requires it and normal access paths are unavailable or insufficient. Break-Glass Access requires: (a) an open `incident_id` in `docs/INCIDENT_RESPONSE.md`; (b) a DEC-030 `access.break_glass_initiated` event emitted with the `incident_id` populated before any query executes; (c) compliance-officer notification at the time of access (not retroactively); and (d) an `access.break_glass_expired` event at the close of the access window. Break-Glass keys must be rotated immediately after use.

### 4.8 Coach-assigned data access within consent scope

Accessing user workout, progress, and coaching data for the purpose of delivering the AI coaching service, where the user has provided explicit GDPR Article 9(2)(a) consent through the FORM onboarding consent gate. Coaching context access must be scoped to directional and bucketed metrics only; raw biometric records must not be accessed outside a documented support or investigation workflow. Access is governed by the RLS policy and tenant isolation architecture defined in `docs/SOC2_READINESS.md` §26.

### 4.9 Approved third-party integrations

Configuring and maintaining integrations with third-party services listed in the subprocessor register (`docs/SECURITY.md §5`) where a signed DPA is in place. Adding a new integration to the FORM stack requires compliance-officer approval and DPA signature before any data flows to the new processor.

### 4.10 Security engineering and incident response operations

Performing authorized security testing, penetration testing (within a defined scope and schedule), log review, incident investigation, and control effectiveness assessment, when carried out by authorized personnel and logged to the DEC-030 audit chain. Penetration testing scopes must be formally defined in writing by compliance-officer and security-engineer before testing begins.

---

## 5. Prohibited Uses

The following uses of FORM Systems are strictly and unconditionally prohibited. No business justification, management instruction, or time pressure overrides these prohibitions. Any person who believes a prohibition must be overridden in an emergency must contact compliance-officer before acting, not after.

### 5.1 Curiosity access to Health Data

Accessing, querying, copying, or transmitting user Health Data for any purpose other than a documented Operational Need as defined in Section 4.3. Accessing Health Data to satisfy personal curiosity, to demonstrate system capabilities in sales or demo contexts using real user data, or for any purpose not traceable to an active operational trigger is prohibited. Demo environments must use synthetic data only.

### 5.2 Credential sharing

Sharing FORM Credentials — including passwords, API keys, tokens, or SSH keys — with any person not individually listed in the Access Control Policy as an Authorized User. Each Authorized User must hold their own, uniquely issued Credentials. Service accounts shared for technical reasons must be approved in writing by security-engineer and documented in the access roster. All Credentials must be stored exclusively in 1Password, Cloudflare Workers Secrets, or Supabase Vault. Storage of Credentials in any other location — including local text files, Slack messages, email, Notion, Linear, or shared documents — is prohibited.

### 5.3 Committing secrets to version control

Committing secrets, Credentials, API keys, bearer tokens, PII, or any data classified as `restricted` in the DEC-030 schema to any git repository, including private repositories, forks, and CI/CD configuration files. This prohibition extends to `.env` files, hardcoded values in test fixtures, and commented-out credential strings. TruffleHog pre-commit scanning (CC5-GAP-003) and `git-secrets` hooks (CC6-GAP-006) are deployed as technical controls; these controls do not replace the behavioral obligation.

### 5.4 Bypassing or disabling security controls

Disabling, modifying, weakening, or circumventing any security control without prior written approval from compliance-officer and a filed exception ticket in Linear. Prohibited actions include but are not limited to: disabling MFA on any admin surface; modifying or disabling Cloudflare WAF rules; weakening or disabling Row-Level Security (RLS) policies in Supabase; removing Content Security Policy (CSP) headers; bypassing HMAC audit log writes; or suppressing Sentry error events that would otherwise surface to the monitoring pipeline. Exception approvals must specify a time-bounded scope and a remediation commitment.

### 5.5 Prohibited cryptographic algorithms

Using SHA-1, MD5, RC4, DES, 3DES, or any other algorithm that does not meet the minimum standards defined in the FORM Cryptography Policy (CC5-GAP-002) for any security-relevant function, including but not limited to: hashing passwords, generating session tokens, signing audit log entries, or verifying data integrity. Passwords must never be transmitted in plaintext over any channel, internal or external.

### 5.6 Force-pushing to protected branches

Force-pushing (`git push --force` or `git push --force-with-lease`) to the `main` branch or any branch configured as protected in the GitHub organization. This prohibition also covers `git rebase` operations that rewrite commit history on protected branches. Legitimate history corrections require a pull request with compliance-officer approval and a documented justification.

### 5.7 Personal financial gain and unauthorized purposes

Using FORM Systems, data, or infrastructure for personal financial gain, competitive intelligence gathering, recruitment of FORM team members by a competing entity, or any purpose unrelated to FORM's business operations. This prohibition extends to use of FORM's Anthropic or ElevenLabs API capacity for personal projects.

### 5.8 Installation of unauthorized software

Installing, running, or connecting software, agents, browser extensions, or services on FORM-issued or MDM-enrolled devices that has not been approved by security-engineer. This includes file-sharing clients, personal cloud sync agents (other than approved backup tools), alternative password managers, VPN clients other than Cloudflare WARP, and any software flagged in the MDM software inventory as prohibited.

### 5.9 Transmitting data to unlisted subprocessors

Transmitting FORM user Health Data or personal data to any third party not listed in the current subprocessor register (`docs/SECURITY.md §5`) under a signed DPA. This prohibition applies regardless of whether the transmission is intentional or incidental. It explicitly prohibits: sending user data to employer HR systems in any aggregate or individual form; including identifiable user metrics in marketing or investor materials; and routing data through analytics or logging platforms not covered by the DPA register.

### 5.10 Audit log tampering

Deleting, modifying, forging, suppressing, or truncating any record in the DEC-030 HMAC-chained audit log, including the `audit_log` table, the HMAC chain integrity records, and the weekly cron chain-verification reports. Any detected chain break is a P0 incident per `docs/INCIDENT_RESPONSE.md` regardless of intent.

### 5.11 Accessing systems from non-MDM devices without approval

Accessing any FORM System from a device that is not enrolled in MDM (Jamf or equivalent) unless written approval has been granted by security-engineer for a specific device, a specific access scope, and a defined time window. BYOD access is permitted only under the conditions specified in the BYOD exception in `docs/SOC2_READINESS.md` §26.6 and requires full-disk encryption, screen lock, and Cloudflare WARP to be active before access is granted.

### 5.12 Misrepresentation to regulators or auditors

Providing false, misleading, or incomplete information to external auditors, regulatory authorities, law enforcement, or data protection authorities in connection with any audit, investigation, or regulatory inquiry involving FORM Systems or FORM data. Any request from a regulatory authority must be routed to compliance-officer within one business day of receipt.

---

## 6. Health Data Special Category Rules

Health Data is GDPR Article 9 special-category data. Processing of Health Data is subject to heightened requirements that apply in addition to all other sections of this policy.

### 6.1 Consent gate enforcement

Health Data may only be collected and processed where the user has provided explicit, informed, freely given, and specific consent under GDPR Article 9(2)(a) through the FORM onboarding consent flow. Consent records are logged to the DEC-030 audit chain as `privacy.consent_granted` events. Any attempt to process Health Data for a purpose not covered by the consent record requires a new consent event or a separate lawful basis approved in writing by compliance-officer. Consent records must not be modified or deleted outside of a documented right-of-withdrawal or erasure workflow.

### 6.2 Analytics pipeline exclusion

Health Data must not be routed to PostHog or any other analytics pipeline in individually identifiable form. PostHog events may carry bucketed, directional metrics (e.g., `workouts_completed_range: "10-20"`) but must not carry raw biometric values, exact measurements, or any combination of fields that constitutes a unique identifier under the re-identification threshold of eleven unique field combinations. The Sentry PII scrubber must be active and configured to strip Health Data fields before any error event is transmitted. Configuration of the Sentry scrubber is a P0 implementation requirement and must not be disabled.

### 6.3 DEC-030 logging on every Health Data access

Every read, export, modification, or deletion of Health Data must emit a DEC-030 audit event with `data_class: restricted` and the `operational_need_ref` field populated. System-generated operations (e.g., nightly backup cron, PITR snapshots) use `actor_type: system` and are logged without alert. Human-initiated access with `actor_type: user` or `actor_type: break-glass` triggers an automated Slack alert to `#security-alerts` within 30 seconds. Any Health Data access event missing a valid `operational_need_ref` is treated as a potential unauthorized access event and must be investigated under `docs/INCIDENT_RESPONSE.md`.

### 6.4 No disclosure to employers or HR systems

Health Data must never be disclosed to a user's employer, HR platform, insurance provider, or any third party not explicitly authorized in the user's consent record, regardless of any commercial arrangement. This prohibition is enforced at the data layer via the `rls_restricted_no_hr` Row-Level Security policy in Supabase. Any request from a commercial partner to provide health metrics aggregated by employer or team is a prohibited disclosure and must be declined and escalated to compliance-officer immediately.

### 6.5 Right to erasure

Upon receipt of a valid right-to-erasure request, Health Data must be hard-deleted from the primary database within 30 days of request, and purged from all backup tiers within 60 days. The erasure sequence must follow the procedure in `docs/GDPR_DPIA.md` and must emit `data.deleted` events to the DEC-030 audit chain for each affected record category. Audit log records are anonymized (user_id replaced; action and timestamp retained) per the retention policy; they are not subject to erasure.

---

## 7. Device Security Requirements

Any device used to access FORM Systems must meet the following minimum security requirements before first use and continuously thereafter.

| Requirement | Standard | Verification Method |
|---|---|---|
| Full-disk encryption | macOS: FileVault 2 enabled. Windows: BitLocker enabled. | MDM compliance report (PRE-26-E-004) |
| MDM enrollment | Enrolled in Jamf Now or equivalent MDM before first use. | MDM device inventory |
| Screen lock | Auto-lock configured to trigger at 5 minutes of inactivity or less. | MDM configuration profile |
| Remote wipe capability | MDM remote wipe and macOS EACS (or Windows equivalent) enabled and tested. The device owner acknowledges that FORM may trigger a remote wipe in the event of device loss or offboarding. | MDM wipe test record |
| Approved software | All installed software must be approved per the MDM software inventory. P2P file-sharing clients, personal VPN clients (other than Cloudflare WARP), and prohibited software categories are blocked. | MDM software inventory |
| Credential management | 1Password agent installed. Credentials must not be stored in browser native password managers or any other credential store not approved by security-engineer. | MDM compliance check |
| Network security | Cloudflare WARP must be active when connecting from any untrusted network (public Wi-Fi, hotel, co-working space, coffee shop). Connecting to FORM Systems from an untrusted network without Cloudflare WARP active is prohibited. Home broadband on a private, password-protected network is permitted without WARP. | Policy acknowledgment; Cloudflare Access OIDC gate as compensating technical control |

Non-compliance with any device security requirement must be reported to security-engineer and remediated within one business day. During the remediation window, access to FORM Systems from the non-compliant device must be suspended.

---

## 8. Incident Reporting Obligations

### 8.1 Mandatory reporting

Every Authorized User is required to report the following events to compliance-officer and security-engineer on the same calendar day that the event is discovered or suspected. Suspicion is sufficient — confirmed knowledge is not required before reporting.

| Event | Reporting Deadline | Reporting Path |
|---|---|---|
| Suspected or confirmed unauthorized access to any FORM System | Same day | #security-alerts Slack + security-engineer direct message |
| Suspected or confirmed Health Data breach or unauthorized disclosure | Same day | #security-alerts Slack + security-engineer + compliance-officer direct message |
| Loss or theft of any MDM-enrolled or BYOD device used for FORM access | Same day | #security-alerts Slack + security-engineer direct message |
| Suspected or confirmed compromise of any FORM Credential | Same day | #security-alerts Slack; rotate credential immediately before or simultaneous with notification |
| Discovery of a secret or Credential in a git repository | Same day | #security-alerts Slack; do not delete the commit without security-engineer guidance |
| Receipt of any communication from a regulatory authority (DPA, ICO, FTC, etc.) | One business day | compliance-officer direct message |
| Any security anomaly or behavior inconsistent with normal operations | Same day | #security-alerts Slack |

### 8.2 No-retaliation commitment

No Authorized User will face retaliation for making a good-faith report under this section. Failure to report a known security event is itself a policy violation and subject to the consequences in Section 9.

### 8.3 Incident Response procedure

All reported events are handled under `docs/INCIDENT_RESPONSE.md`. The Incident Response Plan defines severity classifications, notification timelines (including GDPR Article 33 72-hour supervisory authority notification for qualifying personal data breaches), and the DEC-030 evidence preservation requirements. Authorized Users must cooperate fully with any incident investigation.

---

## 9. Enforcement and Consequences

### 9.1 Enforcement actions

Violation of this policy, regardless of intent, may result in one or more of the following enforcement actions, applied at the discretion of the founder and compliance-officer in proportion to the severity and circumstances of the violation:

1. Immediate suspension of access to all FORM Systems pending investigation.
2. Permanent revocation of access to all FORM Systems.
3. Termination of employment contract, contractor agreement, or advisory engagement, for cause, without severance where permitted by applicable law.
4. Civil liability claim for damages resulting from the violation, including regulatory fines attributable to the violation.
5. Referral to relevant law enforcement or regulatory authorities where the violation constitutes a criminal offense or a regulatory breach.

### 9.2 Health Data violations

Any violation of Sections 5, 6, or 7 involving Health Data is classified as a P0 security incident under `docs/INCIDENT_RESPONSE.md` and triggers the following mandatory actions, in addition to the general enforcement actions in Section 9.1:

- Immediate isolation and preservation of evidence in the DEC-030 audit chain.
- Assessment within 24 hours of whether the violation constitutes a personal data breach requiring notification under GDPR Article 33 (supervisory authority, within 72 hours) or Article 34 (affected data subjects, without undue delay).
- Notification to the lead supervisory authority if the 72-hour threshold is triggered.
- Documentation in the FORM breach register, regardless of whether notification is required.
- GDPR Article 83 administrative fines may apply; FORM will not indemnify individuals whose deliberate or negligent conduct caused the breach.

### 9.3 Audit log tampering

Tampering with DEC-030 audit records is treated as a P0 incident with the maximum available enforcement response, given that audit log integrity is a foundation of the entire compliance program. Any detected HMAC chain break triggers an immediate investigation without exception.

---

## 10. Policy Review and Acknowledgment

### 10.1 Annual review

This policy is reviewed annually by compliance-officer, with reference to changes in FORM's system architecture, team structure, subprocessor roster, and applicable regulatory requirements. The annual review is scheduled in the Q2 (May) slot of the FORM Annual Compliance Calendar (`docs/SOC2_READINESS.md` §15.1). The review produces either a confirmed re-approval with no changes or a new version increment.

### 10.2 Material change re-acknowledgment

On any Material Change (as defined in Section 3), all Authorized Users must re-acknowledge the updated policy within five business days of the updated version being published. Access to FORM Systems will be suspended for any Authorized User who does not re-acknowledge within the five-day window.

### 10.3 Acknowledgment procedure

Every Authorized User must complete the following acknowledgment procedure:

1. Read the current version of this policy in full.
2. Sign the acknowledgment form (PDF) confirming: (a) they have read and understood the policy; (b) they accept the obligations it imposes; (c) they understand the consequences of violation.
3. File the signed PDF to the private compliance repository at `/evidence/cc1/aup/aup-current-signed-[name]-[date].pdf` using the exact naming convention specified.
4. Record the filing in `compliance/policy-approval-log.csv` with the columns: `person_name`, `role`, `policy_version`, `acknowledgment_date`, `evidence_path`.

For the founding team, a git commit to the compliance repository with a commit message referencing the policy version and the signer's name serves as an additional tamper-evident record of the acknowledgment during the pre-hire phase.

### 10.4 New Authorized User onboarding

No new Authorized User may be granted access to FORM Systems before completing policy acknowledgment per Section 10.3. AUP acknowledgment is Step 1 of the new-hire security onboarding checklist (`docs/SOC2_READINESS.md` §22.5) and is a prerequisite for all subsequent onboarding steps.

---

## 11. SOC 2 Evidence Mapping

| SOC 2 Criterion | Criterion Text (summary) | How this policy satisfies it | Evidence Artifacts |
|---|---|---|---|
| **CC1.1** | The entity demonstrates a commitment to integrity and ethical values through establishment of standards of conduct, evaluation against those standards, and remediation of deviations. | AUP establishes behavioral standards for all Authorized Users; acknowledgment procedure creates individual-level evidence of communication; enforcement section documents consequences for deviation. | CC1-E-004 (this document + signed copies); `compliance/policy-approval-log.csv` |
| **CC5.3** | The entity deploys control activities through policies that establish what is expected and procedures that put policies into action. | AUP is a formal policy document covering all FORM Systems; it cross-references procedural documents (Incident Response, Access Control, Cryptography) and specifies implementation requirements (DEC-030 logging, MFA, device security). | CC1-E-004; `docs/SOC2_READINESS.md` §27 policy inventory; gap registry entries CC1-GAP-001, CC5-GAP-001 marked closed |
| **CC6.3** | The entity authorizes, modifies, and removes access to data and information processing systems in a timely manner and in accordance with stated access policies. | Section 4 defines authorized access; Section 5 defines prohibited access; Section 10.4 requires acknowledgment before access is granted; enforcement in Section 9 covers access revocation. | CC1-E-004; access review records (`/evidence/cc6/`); DEC-030 `access.break_glass_initiated` events |

---

## 12. Signatory Block

This policy takes effect when both signatories below have signed and external counsel review is complete. Until then, refer to the status notice in the header.

| Role | Name | Signature | Date |
|---|---|---|---|
| Founder | | | TBD |
| Compliance Officer | | | TBD |

Upon signing, file a copy of this document with the signature block completed as:

```
compliance/cc1/aup/aup-v1.0-signed-[name]-[date].pdf
```

and record the approval in `compliance/policy-approval-log.csv`.

---

## 13. Document History

| Version | Date | Author | Description | Location |
|---|---|---|---|---|
| v0.1-draft | 2026-05-19 | compliance-officer | Initial draft — Annex A in `docs/SOC2_READINESS.md` §27. Not in force. Covered: purpose, scope, acceptable uses (6 items), prohibited uses (7 items), consequences (summary), review cadence. | `docs/SOC2_READINESS.md` lines 4606–4652 |
| v1.0 | 2026-05-21 | compliance-officer | Promoted to standalone policy document. Expanded to enterprise-grade AUP: 12 sections; acceptable uses expanded to 10 items; prohibited uses expanded to 12 items; standalone Health Data Special Category Rules section; Device Security Requirements table; Incident Reporting Obligations with mandatory same-day reporting; Enforcement and Consequences with GDPR Art. 83 triggers; SOC 2 evidence mapping table; acknowledgment procedure specifying `/evidence/cc1/aup/` path and `policy-approval-log.csv`; document history; signatory block. Closes CC1-GAP-001 and CC5-GAP-001 (P0). Filed as CC1-E-004. PENDING EXTERNAL COUNSEL REVIEW. | `docs/ACCEPTABLE_USE_POLICY.md` |

---

---

*v1.0 · May 2026 · Owner: compliance-officer · Status: AUTHORED — PENDING EXTERNAL COUNSEL REVIEW. This document is not yet in legal effect until the founder signs and external counsel review is completed. Upon signing, file a copy as `compliance/cc1/aup/aup-v1.0-signed-[name]-[date].pdf` and record in `compliance/policy-approval-log.csv`.*
