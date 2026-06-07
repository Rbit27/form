# FORM · Coordinated Vulnerability Disclosure Policy v1.0

> **Status:** IN FORCE — 2026-06-07
> **Classification:** Public. Intended audience: security researchers, enterprise customers, auditors.
> **Owner:** `security-engineer` + `compliance-officer`. Review: annually (Q1) or after any significant vulnerability disclosure event.
> **SOC 2 controls satisfied:** CC7.1 (vulnerability identification), CC7.2 (vulnerability remediation), CC2.3 (external security communication).
> **Cross-references:** `docs/SOC2_READINESS.md §16` (penetration test program), `docs/SOC2_READINESS.md §41` (vulnerability management policy), `docs/INCIDENT_RESPONSE.md R-08` (supply chain / external vulnerability incidents), `docs/AUDIT_LOG_SCHEMA.md` (DEC-030 event schema), `docs/ENTERPRISE.md` (enterprise security commitments), `docs/SECURITY.md`.
> **Decision log:** DEC-030 (HMAC-chained audit log — all disclosure intake events are logged).
> **Published at:** `security.form.coach/disclosure` · `security@form.coach`

---

## Table of Contents

1. [Our Commitment to Researchers](#1-our-commitment-to-researchers)
2. [Safe Harbor](#2-safe-harbor)
3. [Reporting Channel](#3-reporting-channel)
4. [PGP Encryption Key](#4-pgp-encryption-key)
5. [What to Include in Your Report](#5-what-to-include-in-your-report)
6. [Scope](#6-scope)
7. [Out of Scope](#7-out-of-scope)
8. [CVSS Severity Classification and Uplift Rules](#8-cvss-severity-classification-and-uplift-rules)
9. [Response SLAs](#9-response-slas)
10. [Coordinated Disclosure Timeline](#10-coordinated-disclosure-timeline)
11. [What Happens After You Report](#11-what-happens-after-you-report)
12. [Acknowledgement and Hall of Fame](#12-acknowledgement-and-hall-of-fame)
13. [Bug Bounty Programme](#13-bug-bounty-programme)
14. [Enterprise Customers and Partners](#14-enterprise-customers-and-partners)
15. [Things We Ask You Not to Do](#15-things-we-ask-you-not-to-do)
16. [DEC-030 Audit Events](#16-dec-030-audit-events)
17. [SOC 2 Evidence Mapping](#17-soc-2-evidence-mapping)
18. [Policy History](#18-policy-history)

---

## 1. Our Commitment to Researchers

FORM processes sensitive personal health data under GDPR Article 9. Security research that helps protect this data serves our users directly. We take external vulnerability reports seriously and treat security researchers as partners, not adversaries.

If you discover a security vulnerability in FORM's systems — in good faith, in line with this policy — we commit to:

- Acknowledging your report within **1 business day**.
- Keeping you informed of our remediation progress.
- Not initiating legal action against you for good-faith research conducted within the bounds of this policy.
- Crediting you publicly (with your consent) once the issue is remediated.
- Working with you on coordinated disclosure timing.

We follow the **ISO/IEC 29147:2018** and **ISO/IEC 30111:2019** standards for vulnerability disclosure and handling.

---

## 2. Safe Harbor

FORM will not pursue civil or criminal action against security researchers who:

1. Discover and report a security vulnerability in good faith.
2. Make a reasonable effort to avoid privacy violations, degradation of service, and harm to FORM users.
3. Do not access, modify, or delete user data beyond the minimum necessary to demonstrate the vulnerability.
4. Do not exfiltrate, share, or publish any personal data, health data, or confidential business information discovered during the research.
5. Follow the coordinated disclosure timeline in §10.
6. Act within the scope defined in §6 and avoid the prohibited actions in §15.

**Health data caveat.** FORM processes GDPR Art. 9 special category health data (exercise records, body composition, recovery data, CV pose keypoints). If your research causes you to observe, access, or incidentally receive this data, you are required to:

- Stop the research activity immediately upon realising health data is reachable.
- Report the finding to `security@form.coach` using the encrypted channel (§4).
- Delete any local copies of health data and certify deletion in your report.
- Not retain, share, or use the health data for any purpose.

Good-faith compliance with these requirements is part of this safe harbour. Exfiltration, retention, or further use of health data by a researcher — even for research purposes — removes safe harbour protections.

**Legal standing.** This safe harbour represents FORM's intentions. FORM cannot bind third parties (including law enforcement agencies, foreign governments, or other companies whose systems may be accessed during research). The safe harbour applies to FORM's own legal claims, not to third-party claims.

---

## 3. Reporting Channel

**Primary channel:** `security@form.coach`

Use this email address for all vulnerability reports. For sensitive reports (any finding that involves or may involve personal data, health data, authentication bypass, or tenant isolation failure), use the PGP key in §4 to encrypt your message before sending.

**Response commitment:** We acknowledge all reports within **1 business day** (Monday–Friday, excluding public holidays in Ukraine and the EU). You will receive a tracking ID (`SEC-YYYY-NNNN`) by return. All communications should reference this ID.

**Escalation.** If you do not receive acknowledgement within 2 business days, resend your encrypted report to `security@form.coach` with "ESCALATION — [original date]" in the subject line. FORM's founder receives escalated reports directly.

---

## 4. PGP Encryption Key

For sensitive reports, encrypt your email using the following PGP public key before sending to `security@form.coach`.

```
-----BEGIN PGP PUBLIC KEY BLOCK-----

[PGP public key to be generated and inserted at M4 deployment — see
SOC2_READINESS.md §39 implementation checklist item 4.
Interim: send sensitive reports to security@form.coach with
"[SENSITIVE]" in the subject line; founder will provide secure
channel within 2 hours.]

-----END PGP PUBLIC KEY BLOCK-----
```

**Key fingerprint:** `[To be published at deployment — verify against security.form.coach/disclosure]`

**Key ID:** `security@form.coach · 4096R`

We recommend verifying the fingerprint on our published trust center page before encrypting. Fingerprint will be published at `security.form.coach/disclosure` and in our GitHub repository's `SECURITY.md` at the time of first deployment.

---

## 5. What to Include in Your Report

A well-structured report allows us to triage and remediate faster. Include:

### Required

| Field | Content |
|---|---|
| **Vulnerability type** | e.g., IDOR, SSRF, SQL injection, broken authentication, RLS bypass, HMAC chain bypass |
| **Affected component** | e.g., `/api/v1/users`, `tenant_sso_configs`, Cloudflare Worker, iOS app |
| **CVSS 3.1 base score** | Your estimate — we will verify, but this helps prioritise |
| **Reproduction steps** | Minimal, numbered, unambiguous |
| **Expected behaviour** | What should happen |
| **Actual behaviour** | What actually happens |
| **Impact** | What an attacker could do with this vulnerability; which data or tenants are affected |
| **Evidence** | Screenshots, HTTP request/response logs (with personal data redacted), proof-of-concept code |

### Optional but helpful

| Field | Content |
|---|---|
| **Suggested fix** | Your recommendation for remediation |
| **References** | CVE IDs, CWE IDs, OWASP category |
| **Contact preferences** | Whether you want attribution in the acknowledgement; preferred name/handle |
| **Disclosure timeline preferences** | If you have a specific publication date in mind |

### Health data redaction

If your reproduction steps or evidence contain any user personal data or health data (even synthetic-looking data that happens to be real), redact it before sending. Replace values with `[REDACTED]`. Do not send raw health payloads, keypoint arrays, or identifiable user records.

---

## 6. Scope

The following assets are **in scope** for coordinated vulnerability disclosure:

### Production systems

| Asset | Description |
|---|---|
| `form.coach` | Marketing and web application |
| `api.form.coach` | REST API (Cloudflare Workers) |
| `auth.form.coach` | Authentication service (SAML / OIDC / SCIM endpoints) |
| `admin.form.coach` | Enterprise admin dashboard |
| `security.form.coach` | Security trust center |
| FORM iOS application | Available on Apple App Store |
| FORM Android application | Available on Google Play Store |
| Cloudflare Worker edge functions | All workers on `form.coach` routes |

### Priority targets (highest impact if compromised)

The following findings are automatically escalated to P0 regardless of CVSS base score and receive the Critical response SLA (§9):

- **Tenant isolation bypass** — any mechanism by which a user or admin of Tenant A can access, read, or modify data belonging to Tenant B
- **Row-Level Security bypass** — any SQL-level technique that defeats Supabase RLS policies
- **HMAC audit log chain break or forgery** — any mechanism that allows inserting, modifying, or deleting audit log records without detection
- **Authentication bypass** — gaining access to any user account, admin account, or API endpoint without valid credentials
- **GDPR Art. 9 health data exposure** — any path that exposes health data (exercise records, CV pose keypoints, body composition, recovery data) to unauthorized parties

---

## 7. Out of Scope

The following are **not in scope** and will not receive safe harbour protections:

### Infrastructure not operated by FORM

| Out of scope | Reason |
|---|---|
| Supabase infrastructure | Operated by Supabase, Inc. — report to `security@supabase.com` |
| Anthropic API endpoints | Operated by Anthropic — report via `anthropic.com/security` |
| ElevenLabs infrastructure | Operated by ElevenLabs — report directly to them |
| Cloudflare infrastructure | Operated by Cloudflare — report via `hackerone.com/cloudflare` |
| Apple App Store / Google Play | Report to Apple / Google directly |
| PostHog infrastructure | Operated by PostHog — report directly to them |

### Finding types not in scope

- **Denial-of-service attacks** — Any finding that requires actively degrading or disrupting service to reproduce. Describe the vulnerability class and we will test it ourselves in a sandboxed environment.
- **Social engineering** — Phishing, vishing, or impersonation attacks against FORM employees.
- **Physical access attacks** — Attacks requiring physical access to FORM infrastructure.
- **Volume-based brute force** — Brute-force attacks that require generating large numbers of requests without a bypass of rate limiting.
- **Known, unpatched third-party vulnerabilities** — CVEs in third-party dependencies that FORM has already acknowledged and is tracking for remediation. Check our published vuln status first.
- **Theoretical attacks without proof of concept** — We require a working reproduction to triage.
- **Self-XSS** — Cross-site scripting that requires the attacker to trick a user into executing malicious code themselves with no plausible exploitation path.
- **Missing security headers on non-sensitive pages** — CSP, X-Frame-Options, etc. on public marketing pages without demonstrated exploitability.
- **Clickjacking on non-authenticated pages.**
- **Rate limiting on non-sensitive endpoints.**
- **Report scanner output** without manual triage and confirmation of exploitability.
- **Subdomain takeover** on subdomains that FORM does not control (e.g., third-party CNAMEs that expired).

---

## 8. CVSS Severity Classification and Uplift Rules

FORM uses **CVSS v3.1** base scores to classify findings. All scores are verified by `security-engineer` within the acknowledgment window.

### Base severity bands

| CVSS v3.1 Base Score | Severity | Colour |
|---|---|---|
| 9.0 – 10.0 | **Critical** | 🔴 |
| 7.0 – 8.9 | **High** | 🟠 |
| 4.0 – 6.9 | **Medium** | 🟡 |
| 0.1 – 3.9 | **Low** | 🟢 |
| 0.0 | **Informational** | ⚪ |

### FORM-specific uplift rules

FORM applies the following severity uplift rules on top of the CVSS base score. These reflect the elevated risk of health data exposure in our platform.

**U-01 — GDPR Art. 9 health data exposure path (+1 severity tier)**

Any finding where the exploitation path exposes, or could expose, GDPR Art. 9 special category health data is uplifted by one severity tier. A Medium (CVSS 5.5) finding that exposes exercise records becomes **High**. A High (CVSS 7.8) finding that exposes body composition data becomes **Critical**.

*Health data in scope for U-01:* `workout_sessions`, `cv_sessions` (pose keypoints, `keypoints_enc`), `wearable_readings`, `meal_log`, `coaching_turns` (LLM session content), `health_profile`, body composition metrics, and any field tagged `data_class = 'Restricted'` per `docs/DATA_MODEL.md §2.5`.

**U-02 — Tenant isolation / RLS bypass (auto-elevates to Critical — no downgrade)**

Any finding that demonstrates a mechanism for bypassing Row-Level Security in Supabase, or any other mechanism that allows cross-tenant data access, is classified as **Critical** regardless of CVSS base score. RLS bypass findings are never eligible for Risk Acceptance and trigger P0 incident protocol (`docs/INCIDENT_RESPONSE.md R-01`).

**U-03 — HMAC audit log chain integrity (+1 severity tier)**

Any finding that affects the tamper-evidence properties of the DEC-030 HMAC audit log chain — including bypassing the chain verification, forging a chain link, or deleting entries without breaking the chain — is uplifted by one severity tier. This uplift reflects the SOC 2 and GDPR Art. 5(2) accountability dependency on audit log integrity.

### Uplift rule interaction

Uplifts are cumulative. A finding that triggers both U-01 and U-03 receives +2 tiers (capped at Critical). U-02 always results in Critical regardless of other rules.

---

## 9. Response SLAs

From receipt of a validated report, FORM commits to the following timelines.

| Severity | Acknowledgement | Initial Assessment | Patch Deployed to Production | Re-test Offered |
|---|---|---|---|---|
| **Critical** | 1 business day | 1 business day | 24 hours from validation | Within 48 hours of patch |
| **High** | 1 business day | 2 business days | 7 days from validation | Within 7 days of patch |
| **Medium** | 1 business day | 5 business days | 30 days from validation | Within 14 days of patch |
| **Low** | 1 business day | 10 business days | 90 days from validation | At researcher's request |
| **Informational** | 1 business day | 10 business days | At FORM's discretion | Not offered by default |

**SLA clock start:** The earlier of: (a) FORM's receipt of the email report, or (b) FORM's first awareness of the vulnerability through any other channel.

**SLA pause conditions.** The remediation clock may pause only in the following defined circumstances:
1. Awaiting an upstream dependency patch (max 14-day pause; must be documented with CVE ID and upstream issue link).
2. Researcher-requested extension to coordinate with a publication timeline.
3. FORM CI/CD outage preventing deployment (max 8 hours; FORM must communicate the outage).

**SLA transparency.** We will proactively communicate to the reporting researcher if we anticipate missing an SLA, with the reason and revised timeline. We do not silently miss SLAs.

---

## 10. Coordinated Disclosure Timeline

FORM operates on a **90-day coordinated disclosure window**. This means:

1. **Day 0:** You submit your report to `security@form.coach`.
2. **Day 1:** FORM acknowledges receipt and assigns a tracking ID.
3. **Day 1–7:** FORM validates the finding and classifies severity.
4. **Day 7–[SLA]:** FORM patches the vulnerability and deploys to production.
5. **Day [patch] – Day 90:** FORM and researcher agree on public disclosure content and timing.
6. **Day 90 (default):** If the vulnerability is patched, FORM publishes a security advisory. If the vulnerability is not yet patched, FORM will request an extension (see below).

### Extension requests

If FORM cannot remediate a Critical or High finding within the 90-day window, we will:

1. Notify the researcher before Day 85 with a documented reason and a new target date.
2. Disclose to affected enterprise customers under the `vuln.enterprise_notified` DEC-030 event (§16).
3. Accept a maximum 14-day extension per request, with a hard limit of 104 days total.

**If FORM fails to respond or patch within 104 days**, the researcher may disclose the vulnerability at their discretion. FORM agrees not to take legal action against researchers who disclose under these circumstances.

### Simultaneous disclosure

If you discover the same vulnerability is being actively exploited in the wild, notify us immediately. We will coordinate an emergency patch and may reduce the disclosure window to 24–48 hours for Critical findings that are under active exploitation.

### Coordinated disclosure with CERT/CSIRT

For vulnerabilities with systemic impact (e.g., a library-level vulnerability affecting many products), FORM will coordinate with CERT/CC, ENISA, or the relevant national CSIRT as appropriate.

---

## 11. What Happens After You Report

### Stage 1 — Intake (Day 0–1)

Your encrypted email arrives at `security@form.coach`. A DEC-030 `vuln.disclosure_received` event is emitted immediately (§16). You receive an automated acknowledgement within 1 hour (or the next business morning if received outside business hours), followed by a personal reply from `security-engineer` or `compliance-officer` within 1 business day with your tracking ID.

### Stage 2 — Triage (Day 1–[SLA start])

`security-engineer` reproduces the finding in a staging environment, applies CVSS v3.1 scoring with FORM-specific uplift rules (§8), and creates a Linear ticket labelled `disclosure-[tracking-ID]` with severity tag. If the finding involves health data or tenant isolation, the incident response protocol (`docs/INCIDENT_RESPONSE.md`) is activated in parallel.

### Stage 3 — Validation (within severity SLA)

`security-engineer` replies to the researcher with:
- Confirmed severity classification
- Whether the finding is accepted as valid or closed as out-of-scope/not-reproducible (with explanation)
- Estimated patch timeline
- Any questions or requests for additional detail

### Stage 4 — Remediation

A patch is written, reviewed, and deployed to production within the severity SLA. The patch commit links back to the Linear ticket. A DEC-030 `vuln.patch_deployed` event is emitted when the fix reaches production.

### Stage 5 — Re-test (offered for Critical and High)

For Critical and High findings, we offer the reporting researcher a re-test window: FORM provides a staging environment or confirms the production patch details so the researcher can verify the fix. We do not require re-testing, but we strongly encourage it for Critical findings.

### Stage 6 — Disclosure coordination

`security-engineer` contacts the researcher to agree on public disclosure content and timing. We will share a draft security advisory for review before publication.

### Stage 7 — Close

DEC-030 `vuln.disclosure_closed` event emitted. Researcher acknowledged in FORM's Hall of Fame (§12) with their consent. Evidence package filed to `compliance/pentest/` alongside the annual penetration test artefacts per `docs/SOC2_READINESS.md §16`.

---

## 12. Acknowledgement and Hall of Fame

FORM maintains a Security Hall of Fame to recognise researchers who responsibly disclose vulnerabilities.

### Eligibility

A researcher is eligible for Hall of Fame recognition if:

- The finding is validated as a genuine security vulnerability (any severity from Low to Critical).
- The finding was not known to FORM before the report.
- The researcher followed this policy in good faith.
- The researcher consents to being credited.

### What we publish

For each credited researcher (with their consent):

| Field | Published |
|---|---|
| Name or handle | As specified by the researcher |
| Vulnerability type | e.g., "Cross-tenant data access", "SAML assertion replay" |
| Severity | Critical / High / Medium / Low |
| Disclosure date | Date the finding was publicly disclosed |
| CVE ID | If assigned |

We do not publish: CVSS scores, proof-of-concept details, affected endpoint paths, or any information that could help attackers exploit the same class of vulnerability in unpatched systems elsewhere.

### No cash bounties (current programme)

The current programme does not offer cash rewards. See §13 for the Bug Bounty roadmap. Researchers who prefer not to be publicly credited may request private acknowledgement — we will maintain an internal record and notify the researcher by email when the fix ships.

---

## 13. Bug Bounty Programme

FORM plans to launch a formal bug bounty programme post-launch. This section documents the planned structure.

**Target launch:** Month 6 post-App Store launch, subject to: (a) first external penetration test closing all Critical and High findings (per `docs/SOC2_READINESS.md §16`); (b) SOC 2 Type I observation period begun.

**Rationale for delayed launch.** Launching a public bug bounty programme with known open Critical or High findings creates reputational and legal risk. FORM will not run a programme until the first pentest cycle closes known findings.

**Planned programme structure:**

| Severity | Planned reward |
|---|---|
| **Critical** | $500 – $2,500 USD |
| **High** | $200 – $500 USD |
| **Medium** | $50 – $200 USD |
| **Low / Informational** | Hall of Fame only |

**Platform:** HackerOne or Bugcrowd (decision pending — will be evaluated before M6 launch).

**Planned scope:** Same as §6 (production API, auth, admin dashboard, iOS app, Android app). Cloudflare Workers edge functions in scope.

**Payout currency:** USD via Stripe or equivalent. FORM will publish exchange-rate and payout policies before programme launch.

Researchers who submit valid Critical or High findings before the bounty programme launches will be retroactively considered for a bounty payment at the programme's floor rate, at FORM's discretion, when the programme goes live.

---

## 14. Enterprise Customers and Partners

### Enterprise customer security contacts

Enterprise customers with a signed MSA and an ACV ≥ $50k may request:

1. **Executive pentest summary** — Finding counts by severity, remediation timeline, and attestation letter. Available under NDA via `security@form.coach`.
2. **Custom disclosure notifications** — Enterprise customers are notified of Critical and High vulnerabilities that could affect their tenant data, even if no actual access occurred. Notification is sent to the `tenant_admin` email within **2 hours** of patch deployment for Critical findings (Template E-VULN-01) and within **24 hours** for High findings (Template E-VULN-02 — see `docs/SOC2_READINESS.md §41`).
3. **Penetration test coordination** — Enterprise customers may request FORM schedule its annual pentest to satisfy their own vendor security review timeline, subject to a 60-day advance notice and availability of the external testing firm.

### Partner security programme

FORM does not currently have an authorised security testing programme for partners. Partners who wish to conduct security testing of FORM API integrations must:

1. Contact `security@form.coach` to request an authorised testing window.
2. Describe the scope, methodology, and intended dates.
3. Receive written authorisation before commencing any testing.

Testing without prior authorisation is not covered by this policy's safe harbour, regardless of intent.

---

## 15. Things We Ask You Not to Do

The following actions are prohibited under this policy and **remove safe harbour protections**:

| Prohibited action | Reason |
|---|---|
| Access, download, or exfiltrate personal data or health data beyond what is minimally necessary to prove the vulnerability | Health data is Art. 9 special category; exfiltration creates independent legal violations |
| Share any personal data or health data with third parties | Privacy violation; endangers users |
| Perform denial-of-service attacks or disrupt production service | Breaches SLA with enterprise customers; violates Computer Fraud and Abuse Act and analogous EU laws |
| Test against accounts you do not own or have explicit written permission to test | Privacy violation |
| Conduct social engineering attacks against FORM employees | Out of scope |
| Submit automated scanner output without manual confirmation of exploitability | Noise; we cannot triage unvalidated scanner results |
| Demand payment or threaten public disclosure before the coordinated window expires | Extortion; removes safe harbour protections entirely |
| Post about the vulnerability on social media, forums, or public channels before coordinated disclosure | May enable active exploitation before patch ships |
| Test in production in a way that modifies or deletes data | Risk of permanent user data loss |
| Conduct cross-tenant attacks against real tenant data | Use the sandbox test tenant for tenant isolation research (contact us to provision one) |

---

## 16. DEC-030 Audit Events

All vulnerability disclosure lifecycle events are HMAC-chained in the DEC-030 audit log per `docs/AUDIT_LOG_SCHEMA.md`. The HMAC chain provides tamper-evident evidence of the disclosure lifecycle — confirming for SOC 2 auditors that every reported finding was received, triaged, and closed in sequence with no records altered.

| Event type | Severity | Retention | Description | Key metadata fields |
|---|---|---|---|---|
| `vuln.disclosure_received` | STANDARD | 7 years | External vulnerability report received at `security@form.coach` | `tracking_id`, `severity_reported_by_researcher` (string), `reporter_handle` (pseudonymous or "anonymous"), `affected_component`, `report_hash` (SHA-256 of encrypted email body) |
| `vuln.disclosure_triaged` | STANDARD | 7 years | Finding reproduced and CVSS score assigned | `tracking_id`, `cvss_base`, `cvss_with_uplift`, `uplift_rules_applied` (array: ["U-01","U-02","U-03"]), `severity_final`, `linear_ticket_id` |
| `vuln.disclosure_accepted` | STANDARD | 7 years | Finding confirmed as valid; remediation started | `tracking_id`, `severity_final`, `assigned_engineer`, `patch_target_date` |
| `vuln.disclosure_rejected` | STANDARD | 7 years | Finding closed as out-of-scope, not-reproducible, or known | `tracking_id`, `closure_reason` (enum: `out_of_scope` / `not_reproducible` / `known_finding` / `informational`) |
| `vuln.patch_deployed` | MEDIUM | 7 years | Fix deployed to production | `tracking_id`, `deploy_sha`, `deploy_timestamp`, `sla_met` (boolean), `days_from_receipt` (integer) |
| `vuln.enterprise_notified` | MEDIUM | 7 years | Enterprise customer notified per §14 | `tracking_id`, `template_id` (E-VULN-01 or E-VULN-02), `tenants_notified_count` (integer, no tenant IDs), `notification_channel` |
| `vuln.disclosure_closed` | STANDARD | 7 years | Finding publicly disclosed or closed as private | `tracking_id`, `disclosure_url` (null if private), `researcher_credited` (boolean), `hall_of_fame_name` (null if no credit) |
| `vuln.sla_breach` | HIGH | 7 years | Patch SLA missed (for any severity) | `tracking_id`, `severity`, `sla_days`, `actual_days`, `breach_reason`, `revised_target` |

**Privacy constraints for all `vuln.*` events:**
- No `user_id` in any event payload.
- No personal health data, coaching content, or PII in any event payload.
- `reporter_handle` is the researcher's self-reported pseudonym or "anonymous" — never an email address or real name.
- `report_hash` is a one-way hash of the submitted report content; the actual report text is stored only in the encrypted compliance vault, not in the audit log.
- `tenants_notified_count` is an integer count only; no `tenant_id` or company name in the log.

---

## 17. SOC 2 Evidence Mapping

This policy and the `vuln.*` DEC-030 events satisfy the following SOC 2 Type II criteria:

| Criterion | Description | How this document satisfies it |
|---|---|---|
| **CC7.1** | The entity uses detection and monitoring procedures to identify changes to configurations or the introduction of new vulnerabilities | This policy establishes the external vulnerability intake channel. The `vuln.disclosure_received` and `vuln.disclosure_triaged` DEC-030 events create an auditable record of vulnerability discovery. Combined with `docs/SOC2_READINESS.md §41` (Dependabot, `npm audit`, penetration test), this constitutes the full CC7.1 evidence set. |
| **CC7.2** | The entity monitors system components and the operation of those components for anomalies that are indicative of malicious acts, natural disasters, and errors affecting the entity's ability to meet its objectives | The `vuln.patch_deployed` and `vuln.sla_breach` events — combined with the SLA table in §9 — demonstrate that FORM both detects vulnerabilities and remediates them within defined timelines. |
| **CC7.3** | The entity evaluates security events to determine whether they could or have resulted in a failure of the entity to meet its objectives | Severity classification (§8) and uplift rules (U-01, U-02, U-03) constitute the documented criteria for evaluating whether a finding is a security event warranting incident response. U-02 (RLS bypass → P0) and U-01 (health data exposure path → tier uplift) are the explicit bridges between this policy and `docs/INCIDENT_RESPONSE.md`. |
| **CC7.4** | The entity responds to identified security incidents by executing a defined incident response programme | Cross-reference to `docs/INCIDENT_RESPONSE.md R-08` (supply chain / external vulnerability) ensures that Critical findings automatically activate the incident response programme. The `vuln.disclosure_accepted` event triggers the Linear-tracked remediation workflow per `docs/SOC2_READINESS.md §41`. |
| **CC2.3** | The entity communicates information about security commitments and responsibilities to external parties | This policy is published publicly at `security.form.coach/disclosure`, constituting FORM's external communication of its vulnerability intake process, safe harbour, and coordinated disclosure commitment. |
| **C1.2** | The entity disposes of confidential information to meet the entity's objectives related to confidentiality | The prohibition on health data retention by researchers (§2) and the `report_hash` privacy constraint (§16) ensure that confidential health data in vulnerability reports is not retained in audit infrastructure. |

**SOC 2 evidence artefacts for this policy:**

| Artefact ID | Description | Filing location |
|---|---|---|
| CVD-E-001 | This policy document (git-committed, version-dated) | `docs/RESPONSIBLE_DISCLOSURE.md` in `rbit27/form` |
| CVD-E-002 | `security.form.coach/disclosure` page screenshot with PGP key visible | `compliance/evidence/cvd/disclosure-page-screenshot-YYYY-MM.png` |
| CVD-E-003 | Sample `vuln.disclosure_received` DEC-030 event (redacted) from first researcher report | `compliance/evidence/cvd/sample-dec030-event-YYYY.json` |
| CVD-E-004 | DEC-030 HMAC chain verification output spanning observation period | `compliance/pentest/YYYY/hmac-chain-verification.txt` (shared with penetration test artefacts) |
| CVD-E-005 | Enterprise customer notification log excerpt (from `vuln.enterprise_notified` events) — integer counts only | `compliance/evidence/cvd/enterprise-notification-log-YYYY.csv` |

---

## 18. Policy History

| Version | Date | Author | Summary of changes |
|---|---|---|---|
| v1.0 | 2026-06-07 | `security-engineer` + `compliance-officer` | Initial policy. Establishes intake channel, safe harbour, 90-day coordinated disclosure window, CVSS v3.1 with FORM-specific uplift rules (U-01, U-02, U-03), response SLAs, DEC-030 event schema, Hall of Fame, Bug Bounty roadmap. Closes `/disclosure` P0 gap from `docs/SOC2_READINESS.md §39`. SOC 2 criteria: CC7.1, CC7.2, CC7.3, CC7.4, CC2.3, C1.2. |

*v1.0 additions: Full coordinated vulnerability disclosure policy for FORM. Scope covers production API, auth service, admin dashboard, Cloudflare Workers edge, iOS and Android mobile apps. Safe harbour with GDPR Art. 9 health data caveat — researchers must stop and delete if health data is encountered. PGP key block (placeholder for M4 key generation). 90-day coordinated disclosure window with 14-day extension and 104-day hard cap. CVSS 3.1 severity bands with three FORM-specific uplift rules: U-01 (Art. 9 health data path +1 tier), U-02 (RLS bypass → auto-Critical, no downgrade), U-03 (HMAC chain integrity +1 tier); uplifts cumulative. Response SLA table: Critical 24h, High 7d, Medium 30d, Low 90d from validation. Seven-stage response process (intake → triage → validation → remediation → re-test → disclosure coordination → close). Enterprise customer notifications: Template E-VULN-01 (Critical, within 2h of patch) and E-VULN-02 (High, within 24h). Hall of Fame eligibility criteria; no cash bounties in current programme; Bug Bounty roadmap at Month 6 post-launch (HackerOne or Bugcrowd, $50–$2,500 range, same scope). Eight DEC-030 HMAC-chained events: `vuln.disclosure_received` / `vuln.disclosure_triaged` / `vuln.disclosure_accepted` / `vuln.disclosure_rejected` / `vuln.patch_deployed` / `vuln.enterprise_notified` / `vuln.disclosure_closed` / `vuln.sla_breach`; all 7-year retention (MEDIUM/STANDARD/HIGH), no user_id, no health data, reporter_handle pseudonymous only. SOC 2 mapping: CC7.1, CC7.2, CC7.3, CC7.4, CC2.3, C1.2. Five evidence artefacts CVD-E-001 through CVD-E-005. Closes `/disclosure` page gap (🔴 Open → 🟡 Partial: policy authored; gap fully closes when `security.form.coach/disclosure` is deployed and screenshot filed as CVD-E-002). References: `docs/SOC2_READINESS.md §16, §39, §41`, `docs/INCIDENT_RESPONSE.md R-08`, `docs/AUDIT_LOG_SCHEMA.md`, `docs/ENTERPRISE.md`, `docs/DATA_MODEL.md §2.5`.*

---

**Owner:** `security-engineer` + `compliance-officer` + founder
**For vulnerability reports:** `security@form.coach` (PGP encrypted for sensitive findings — see §4)
**Policy questions:** `privacy@form.coach`
**Enterprise security reviews:** `security@form.coach` (NDA-gated artefacts)
