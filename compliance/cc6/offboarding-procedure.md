# Personnel Offboarding Procedure

**File reference:** CC6-E-001
**SOC 2 criterion:** CC6.3 (Prior to changing or removing an individual's physical or logical access, the entity considers and evaluates the business purpose for the access)
**Related criteria:** CC6.1 (Logical access security software), CC6.2 (New access provisioning)
**Owner:** compliance-officer (founder)
**Status:** AUTHORED — ready for execution at first hire; solo-founder compensating control documented in §7
**Document date:** 2026-05-22
**Artifact path:** `compliance/cc6/offboarding-procedure.md`
**Review cadence:** Annual (Q1), or immediately following any personnel departure
**Next review due:** May 2027

---

## 1. Purpose and Scope

This procedure defines the mandatory steps FORM must complete when any person's access to FORM systems, data, or repositories is terminated. It satisfies SOC 2 CC6.3 by documenting how FORM evaluates and removes access rights for departing personnel.

**Scope — applies to:**

- Full-time employees (current: none; document pre-dates first hire)
- Part-time employees
- Contractors and consultants
- Advisors with system access
- Interns with system access
- The founder (succession/incapacity protocol — §8)

**Out of scope:** Enterprise customer user offboarding. That process is documented in `docs/SSO_SCIM_IMPLEMENTATION.md §3` (SCIM deprovisioning) and `docs/INCIDENT_RESPONSE.md §11`.

**Mandatory SLA:** All credential revocation steps in §4 must be completed within **24 hours** of departure confirmation. This SLA is the CC6.3 operating commitment. Exceptions require written approval from the compliance-officer and must be logged in §9.

---

## 2. Triggering Events

This procedure activates on any of the following:

| Event | Trigger Point |
|---|---|
| Voluntary resignation | Date of final day of employment (last day in office/remote) |
| Involuntary termination | Immediately upon termination conversation (same-day completion required) |
| Contract end (contractor / consultant) | End-of-contract date, or immediately upon early termination |
| Performance-based termination for cause | Immediately upon notification — do not wait for end of notice period |
| Advisor access removal | Date of advisor relationship termination |
| Death or incapacity | See §8 for founder; for employees, immediate execution by compliance-officer |
| Security incident requiring access revocation | Immediately upon IC decision (see `docs/INCIDENT_RESPONSE.md §5`) |

**For involuntary termination or cause-based termination:** complete §4 (Credential Revocation) before or simultaneously with the termination conversation, not after. Departing personnel should not have system access after notification.

---

## 3. Responsible Parties

| Role | Responsibility |
|---|---|
| **compliance-officer** | Owns execution of this procedure; signs completion attestation |
| **security-engineer** | Verifies revocation is complete for all production systems; provides technical confirmation |
| **Departing person's manager** | Initiates the procedure by notifying compliance-officer; confirms access inventory |
| **customer-success** | Notifies enterprise customers if any enterprise-facing credentials are rotated (see §4.10) |

**Solo-founder note (current state):** Until first hire, compliance-officer = security-engineer = manager = founder. Procedure is maintained as a template and will be executed by the founder's designated successor or emergency contact per §8.

---

## 4. Credential Revocation Checklist (< 24h SLA)

Complete all rows. Mark each with completion timestamp. File the completed checklist in `compliance/evidence/cc6/offboarding/<YYYYMMDD-name-slug>/CC6-E-001-checklist.md`.

### 4.1 Identity and Access Management

| # | System | Action | SLA | Completed (UTC timestamp) | Completed by |
|---|---|---|---|---|---|
| 1.1 | GitHub (`github.com/Rbit27`) | Remove collaborator access; revoke any personal access tokens issued to the individual | < 24h | — | — |
| 1.2 | Supabase | Remove from organisation; revoke any service-role or anon keys created specifically for the individual | < 24h | — | — |
| 1.3 | Cloudflare | Remove from account team; revoke any API tokens issued to the individual | < 24h | — | — |
| 1.4 | 1Password Teams | Remove from relevant vaults; if individual was a vault admin, rotate all secrets in those vaults | < 24h | — | — |
| 1.5 | PostHog | Remove team member; revoke any personal API keys | < 24h | — | — |
| 1.6 | Sentry | Remove organisation member | < 24h | — | — |
| 1.7 | Google Workspace / email (if applicable) | Suspend account; set email forwarding to compliance-officer for 30 days; disable login | < 24h | — | — |
| 1.8 | Slack workspace (if applicable) | Deactivate account | < 24h | — | — |
| 1.9 | Linear (project management, if access granted) | Remove team member | < 24h | — | — |
| 1.10 | ElevenLabs (if API access granted) | Rotate any shared API keys the individual had knowledge of | < 24h | — | — |
| 1.11 | Anthropic / Claude API (if direct access granted) | Rotate any API keys the individual had knowledge of | < 24h | — | — |
| 1.12 | AWS / GCP / Azure (if applicable) | Disable IAM user; revoke access keys | < 24h | — | — |
| 1.13 | Stripe (if applicable) | Remove team member; revoke restricted keys | < 24h | — | — |

### 4.2 Physical and Device Access

| # | Action | SLA | Completed | Notes |
|---|---|---|---|---|
| 2.1 | Revoke any physical office key cards or access codes | < 24h | — | Current state: no physical office; N/A |
| 2.2 | Confirm recovery of any FORM-owned hardware (laptops, devices, USB keys) | < 5 business days | — | Log asset ID, return date, condition |
| 2.3 | Confirm that any personal devices with FORM source code or data have been wiped (see §5) | < 5 business days | — | Obtain written confirmation or witness wipe |
| 2.4 | Revoke any VPN or remote-access credentials | < 24h | — | Current state: no VPN; N/A |

### 4.3 Secrets Rotation Decision

The following secrets must be evaluated for rotation whenever an engineer or developer with production access departs. The decision to rotate must be documented even if rotation is not performed.

| Secret | Rotation Required? | Decision rationale | Rotated by | Timestamp |
|---|---|---|---|---|
| Supabase `service_role` JWT | Mandatory if individual had production DB access | — | — | — |
| Cloudflare Workers secrets (all) | Mandatory if individual had Workers deploy access | — | — | — |
| ElevenLabs API key | Mandatory if individual had AI voice access | — | — | — |
| Anthropic API key | Mandatory if individual had AI API access | — | — | — |
| Stripe restricted keys | Mandatory if individual had payment processing access | — | — | — |
| PostHog project API key | Mandatory if individual had analytics write access | — | — | — |
| GitHub deploy keys | Mandatory if individual had repository write access | — | — | — |
| HMAC chain signing key (Audit Log) | **Always rotate** — no exceptions (DEC-030 integrity requirement) | — | — | — |

**HMAC chain key rotation protocol:** Rotation of the audit log HMAC key requires an entry in the audit log (event type `security.hmac_key_rotated`) before the new key is activated. See `docs/AUDIT_LOG_SCHEMA.md §9` and `docs/INCIDENT_RESPONSE.md R-05`.

### 4.4 DEC-030 Audit Log Events

The following events must be logged to the HMAC-chained audit log upon completion of revocation:

```
security.access_revoked     — one event per system with {system, user_id, actor, reason}
security.secrets_rotated    — one event per rotated secret with {secret_name, actor, reason}
security.device_returned    — upon confirmed hardware return
security.device_wiped       — upon confirmed remote wipe or physical destruction evidence
```

All events are append-only. Log immediately upon action completion, not at end of procedure.

---

## 5. Personal Device and Data Handling

Individuals who accessed FORM production systems from personal devices (BYOD) must confirm one of the following before the 24h SLA expires:

**Option A — Remote wipe (preferred if MDM enrolled):**
- Compliance-officer initiates remote wipe via Jamf/MDM
- Wipe confirmation logged: `CC6-E-001-YYYYMMDD-<name>-device-wipe-confirmation.[ext]`

**Option B — Self-certification (acceptable if no MDM):**
- Individual signs the self-certification template (§10.1)
- Self-certification must specify: device type, make/model, FORM data categories confirmed deleted, date of deletion
- Filed in `compliance/evidence/cc6/offboarding/<YYYYMMDD-name-slug>/`

**Data categories to confirm deletion:** source code clones, `.env` files or secrets, Supabase dumps, PostHog exports, Sentry tokens, coaching session transcripts, user PII, any AI model artefacts or training data.

**Copies retained by FORM:** FORM retains all production data in Supabase and Cloudflare. The individual's personal copies must be deleted; FORM's own copies are governed by the retention schedule in `compliance/p1/retention-decisions.md`.

---

## 6. NDA and Confidentiality Reminder

At the time of departure:

1. Compliance-officer sends the **Post-Departure Confidentiality Reminder** (template §10.2) via email to the individual's personal email address (not the FORM work email — that account is suspended).
2. The reminder references the specific NDA version signed (stored in `compliance/ndas/<name>-nda-YYYYMMDD.pdf`) and confirms post-departure obligations:
   - Continued confidentiality of FORM IP, source code, user data, business strategy
   - Non-disclosure of enterprise customer identities and contract terms
   - No retention of FORM confidential data on personal devices
3. The reminder is not a new legal instrument — it references the existing NDA. It does not require the individual's signature. It creates a contemporaneous record that they were reminded.
4. Log sent timestamp and delivery confirmation in offboarding checklist.

**If NDA was not signed:** Flag immediately to compliance-officer and legal counsel. Document that NDA was not in place. Expedite obtaining a post-departure NDA if contractually available.

---

## 7. Solo-Founder Phase — Current State (as of 2026-05-22)

**Current team size:** 1 (founder only).

During the solo-founder phase, this procedure has no immediate execution trigger. It is maintained as a pre-drafted, tested template. The compensating control for SOC 2 CC6.3 during this phase is:

1. **This authored document** demonstrating the organisation has designed a CC6.3-compliant process.
2. **Access inventory** is maintained in `compliance/c1/data-asset-inventory.md` (PRE-34-E-001), which serves as the baseline for any future offboarding audit.
3. **Secrets management** is documented in `docs/CRYPTOGRAPHY_POLICY.md` — all secrets are stored in Cloudflare Workers Secrets and 1Password; no credentials exist only in a person's memory.
4. **No contractors or advisors currently have production system access.** If access is granted to any external party, this procedure applies and the 24h SLA becomes operative.

**Auditor disclosure:** FORM will disclose to the SOC 2 auditor that no employee offboarding has occurred during the observation period due to team size. The auditor will be asked to evaluate design effectiveness (documented procedure) rather than operating effectiveness (execution). SOC 2 standards permit this disclosure for solo-founder companies with an agreed-upon observation scope.

---

## 8. Founder Succession / Emergency Access Protocol

In the event of founder incapacity or death, the following protocol applies to ensure business continuity and data protection without creating an unauthorized-access risk:

### 8.1 Emergency Access Contacts

| Role | Contact |
|---|---|
| Designated emergency contact | [To be named: trusted colleague or legal counsel] |
| Legal representation | [To be named: Ukrainian corporate counsel] |
| 1Password Emergency Kit holder | [To be named: stored in a physical sealed envelope with designated individual] |

*Founder: populate this section before first enterprise contract. The emergency contact should have a sealed envelope with the 1Password Emergency Kit, not individual passwords.*

### 8.2 Emergency Access Protocol

1. Emergency contact verifies incapacity or death through appropriate documentation.
2. Emergency contact accesses 1Password via Emergency Kit (sealed physical envelope).
3. Emergency contact changes all master passwords immediately upon access.
4. Emergency contact logs all access events in the audit log (event type `security.emergency_access_invoked`).
5. Emergency contact engages legal counsel within 24 hours to determine disposition of assets, data, and enterprise customer obligations.
6. Enterprise customers are notified within 48 hours per `docs/INCIDENT_RESPONSE.md §12` (P0 event).

### 8.3 Data Preservation Obligations

In a founder incapacity/death scenario, GDPR Art. 17 deletion obligations and enterprise DPA obligations continue. The emergency contact or legal representative must:
- Not delete any Supabase health data without following the erasure protocol in `docs/INCIDENT_RESPONSE.md §12.5`
- Honour any pending DSAR requests (30-day SLA continues running)
- Notify the supervisory authority if the incapacity creates a risk to data processing (GDPR Art. 33/34)

---

## 9. Exception Log

| Date | Departing person | Exception type | SLA breach (hours) | Justification | Approved by | Follow-up completed |
|---|---|---|---|---|---|---|
| — | — | — | — | — | — | — |

*Exceptions must be approved by compliance-officer (or, if the compliance-officer is departing, by the founder or designated successor) before the procedure is considered complete.*

---

## 10. Templates

### 10.1 Personal Device Self-Certification

```
FORM — Personal Device Data Deletion Self-Certification

I, [Full Name], confirm that on [Date], I deleted all FORM confidential data,
source code, credentials, and user data from the following personal devices:

Device 1: [Make, model, OS]
Data deleted: [list categories]
Deletion method: [Trash + empty / Secure erase / Factory reset / Other]

I confirm I retain no copies of any FORM source code, user data, secrets,
API keys, or enterprise customer information on any personal device,
cloud storage, or external drive.

Signed: ______________________
Date: ________________________
```

### 10.2 Post-Departure Confidentiality Reminder Email

```
Subject: FORM — Post-departure confidentiality reminder

Hi [Name],

This is a brief note to confirm your ongoing confidentiality obligations
under the NDA signed on [Date] remain in effect following your departure
from FORM on [Departure Date].

These obligations include:
- Continued confidentiality of FORM source code, user data, and business strategy
- No retention of FORM confidential data on personal devices or cloud storage
- Non-disclosure of enterprise customer names, contract terms, and usage data
- Non-disclosure of any FORM trade secrets or proprietary AI model configurations

If you have any questions about these obligations, please contact
privacy@form.coach.

Thank you for your contribution to FORM.

[Founder name]
Founder, FORM
```

---

## 11. SOC 2 Evidence Package

Upon completion of any offboarding event, file the following in `compliance/evidence/cc6/offboarding/<YYYYMMDD-name-slug>/`:

| Artifact | File name convention | SOC 2 criterion |
|---|---|---|
| Completed revocation checklist (§4) | `CC6-E-001-checklist.md` | CC6.3 |
| Device self-certification or wipe confirmation (§5) | `CC6-E-001-device-cert.[ext]` | C1.2 |
| Post-departure NDA reminder (§6) — copy of email + delivery receipt | `CC6-E-001-nda-reminder.[ext]` | CC1.4 |
| Audit log export covering revocation events | `CC6-E-001-audit-log-extract.csv` | CC6.3, CC2.3 |
| Secrets rotation record (§4.3) | `CC6-E-001-secrets-rotated.md` | CC6.1 |
| Exception log entry (if any) (§9) | Referenced from `compliance/cc6/offboarding-procedure.md §9` | CC6.3 |

Commit all evidence files with commit message: `compliance: CC6 offboarding evidence — [name slug] YYYY-MM-DD`.

---

## 12. Gap Status

| Gap ID | Description | Status before this document | Status after this document | Closes when |
|---|---|---|---|---|
| PRE-05 | Offboarding procedure documented and tested end-to-end | 🔴 Open | 🟡 Authored | 🟢 Closed upon first real execution + evidence filing |
| CC6-GAP-001 | No formal offboarding procedure existed | 🔴 Open | 🟡 Authored | 🟢 Closed upon first real execution |

**Auditor note:** PRE-05 and CC6-GAP-001 advance from 🔴 Open to 🟡 Authored with this document. The gap closes to 🟢 upon the first real execution of the procedure with a filed evidence package. For a solo-founder company with no employees, the SOC 2 auditor will be asked to accept design effectiveness as the operating evidence for the observation period, with the commitment that first execution will be tested internally (see §15.1, PRE-27 in `docs/SOC2_READINESS.md`).

---

*v0.1 · 2026-05-22 · Owner: compliance-officer + security-engineer*
*Authoring agents: enterprise-architect, compliance-officer*
*Next review: May 2027 or immediately following any personnel departure.*
