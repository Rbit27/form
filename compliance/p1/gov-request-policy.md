# Government Request Handling Policy

**File reference:** PRE-35-E-006  
**SOC 2 criteria:** P6.5 — Disclosure to Third Parties (Government Requests) · CC6.1 · CC6.3 · C1.1 · CC9.1  
**GDPR articles:** Art. 6(1)(c) legal obligation · Art. 9(2)(f) legal claims · Art. 48 international transfers · Art. 58(1) supervisory authority powers  
**Remediates:** P-GAP-006 (P1) — government request handling policy not formalized as standalone auditor exhibit  
**Input documents:** `docs/SOC2_READINESS.md §35.8.2` · `docs/INCIDENT_RESPONSE.md §R-13` · `docs/ENTERPRISE.md §When we say no`  
**Owner:** compliance-officer  
**Decision authority:** compliance-officer + outside counsel (EU-qualified and US-qualified) + founder sign-off  
**Status:** ✅ Drafted — pending outside counsel review and founder signature before publication as PRE-35-E-006  
**Effective date:** 2026-06-10 (draft)  
**Next review:** Annually — or upon receipt of the first legal request, whichever is sooner  
**Artifact path:** `compliance/p1/gov-request-policy.md`

---

## Purpose

This policy governs how FORM responds to requests from government bodies, law enforcement agencies, national security authorities, courts, and regulatory or supervisory authorities seeking disclosure of FORM user data, metadata, or system records.

Its purpose is threefold:

1. **Close P-GAP-006** — formalize FORM's no-backdoor principle (ENTERPRISE.md §When we say no) as a standalone policy that satisfies AICPA SOC 2 Privacy TSC P6.5
2. **Provide auditor evidence** — serve as PRE-35-E-006 in the SOC 2 Type II evidence collection, demonstrating that FORM has a documented, reviewed, and founder-approved policy for government data requests
3. **Protect users** — establish enforceable constraints that apply regardless of the nature or urgency of the request, preserving user trust and GDPR Art. 9 special-category data protections

This policy document contains the four core principles and process obligations. The full operational runbook — including request type taxonomy, severity matrix, scope minimization tables, user notification templates, enterprise tenant notification protocol, and DEC-030 HMAC-chained audit event schema — is maintained in `docs/INCIDENT_RESPONSE.md §R-13`.

---

## Scope

This policy applies to:

- All current and future FORM team members, agents, contractors, and outside counsel acting on FORM's behalf
- All requests from any government or law enforcement body — domestic (US, EU Member States) or foreign
- All FORM user data irrespective of tier (consumer or enterprise), jurisdiction of the user, or the data category involved
- Both compulsory legal process (court orders, subpoenas, search warrants, GDPR Art. 58 supervisory authority demands) and informal voluntary requests

It does **not** apply to:

- Internal access to user data by authorized FORM personnel under documented data access procedures (governed by `compliance/cc6/offboarding-procedure.md` and the RBAC policy in `docs/SSO_SCIM_IMPLEMENTATION.md`)
- GDPR Data Subject Access Requests from individuals (governed by `docs/INCIDENT_RESPONSE.md §R-14` and the DSAR process in `docs/GDPR_DPIA.md`)

---

## Four Governing Principles

### Principle 1 — Legal Review Before Any Action

No FORM user data is released in response to any government or law enforcement request — of any type, from any jurisdiction — without prior review by outside legal counsel.

- Informal requests (phone calls, unserved letters, courtesy notices, oral demands) are declined in writing. No data is produced. No informal cooperation.
- Formal compulsory process (court orders, subpoenas, search warrants, GDPR Art. 58 binding demands) is reviewed by counsel to confirm: (a) the instrument is legally valid and correctly served, (b) it falls within the serving authority's jurisdiction, (c) the scope of data demanded is proportionate and legally required, (d) GDPR Art. 48 compliance obligations are met for non-EU orders covering EU-resident user data.
- **No team member** may acknowledge, respond, or produce data without founder authorization and counsel sign-off. This constraint survives any claimed urgency.
- For NSL and FISA orders: outside US-qualified NSL-experienced counsel must be retained **before** FORM acknowledges receipt to the requesting agency. Gag order implications take precedence over all internal communication norms.

### Principle 2 — Narrowest Possible Response

FORM produces only the minimum data explicitly required by the legal instrument. No data beyond what is compelled.

- If the order names specific users: only those users' data is produced.
- If the order specifies a date range: only data within that range is produced.
- If the order specifies data categories: only those categories are produced.
- If the order is overbroad — temporally unlimited, user-class based ("all users who did X"), or does not explicitly authorize Art. 9 special-category health data production — counsel files a motion to quash or narrowing letter before any partial disclosure.
- Art. 9 health data (workout logs, coaching session content, CV form scores, biometric wearable readings, health profiles, meal logs) requires an explicit, higher legal standard in the instrument beyond a general "all records" subpoena. Counsel must confirm Art. 9 production is specifically and lawfully required.
- CV session `keypoints_enc` raw biometric data is architecturally never producible: on-device encryption means FORM does not hold the decryption key. This is not a policy decision — it is a technical impossibility by design.

### Principle 3 — User Notification Where Legally Permitted

FORM notifies the named user(s) before complying with legal process, unless a court order or gag order legally prohibits prior notification.

- Notification is sent as soon as the disclosure decision is made and before data is produced.
- Where notification is legally prohibited (NSL gag order, FISA order, explicit court prohibition): the existence of the order is never publicly disclosed, but FORM records the prohibition in the incident log and reflects the prohibition (without identifying the order) in the annual Transparency Report.
- For enterprise tenant users: the tenant IT admin is notified separately per `docs/INCIDENT_RESPONSE.md §R-13.9` using Template T-01, subject to the same gag order prohibition constraint.
- Where GDPR Art. 34 (high-risk notification to data subjects) is independently triggered by the nature of the production — particularly for Art. 9 biometric data — compliance-officer assesses and acts on that obligation in parallel.

### Principle 4 — Annual Transparency Report

FORM publishes an annual Government Transparency Report covering:

| Item | Disclosure format |
|---|---|
| Number of government requests received | Exact count, or "0 to N" range if gag-order constraints require |
| Breakdown by request type (criminal, civil, NSL, FISA, EU DPA Art. 58, foreign intelligence) | Per-category count |
| Number of requests where FORM disclosed data | Count |
| Number of requests FORM challenged or narrowed | Count |
| Number of requests FORM declined (informal or legally defective) | Count |
| Number of users affected by disclosed requests | Aggregate only — not individual identities |
| Number of requests where user notification was legally prohibited | Count (without identifying the orders) |
| Art. 48 cross-border requests: count and outcome | Count + comply/challenge/decline |

**Publication cadence:** Annual.  
**First report due:** 12 months after first enterprise customer launch, or upon receipt of the first legal request — whichever is sooner.  
**Owner:** compliance-officer + founder.  
**Format:** Plain-text table; no attorney-client privileged content; no specific incident dates or agency names unless the underlying instrument is publicly available.  
**Location:** Published as a standalone public document at form.coach/transparency or in the press section; also archived at `compliance/evidence/transparency-report/YYYY.md`.

---

## Mandatory Process Gates

These process gates apply to every government request without exception.

| Gate | Description | Authority |
|---|---|---|
| **G-01: Receive and log** | Instrument received, scanned, timestamped, and filed at `compliance/evidence/legal-requests/<YYYY-MM-DD-slug>/received/`. SHA-256 hashes recorded. | First person to receive |
| **G-02: Founder notified** | Founder and compliance-officer notified immediately via Signal DM — not Slack. No public acknowledgment until both are informed. | First person to receive |
| **G-03: DEC-030 event emitted** | `legal.request_received` DEC-030 HMAC-chained event emitted at T+0. This event cannot be backdated. See DEC-030 event schema below. | compliance-officer |
| **G-04: Counsel retained** | Outside legal counsel retained within 2 hours (P0/P1) or 24 hours (P2). EU-qualified counsel for GDPR matters; US-qualified for NSL/FISA. | founder |
| **G-05: Legal hold activated** | `legal_hold_active = true` set on affected records in Postgres. No data deletion runs against held records until hold is released. `legal.legal_hold_activated` DEC-030 event emitted. | compliance-officer + platform-engineer |
| **G-06: GDPR Art. 48 assessment** | For any non-EU order covering EU-resident user data: EU-qualified counsel confirms treaty basis (MLAT or adequacy decision) before production proceeds. | outside counsel (EU) |
| **G-07: Founder disclosure decision** | No data is produced without written founder sign-off (Signal or email with compliance-officer and counsel on thread). | founder |
| **G-08: Disclosure executed and logged** | Production package delivered per production letter. `legal.disclosure_executed` DEC-030 event emitted immediately on delivery. Evidence package filed. | compliance-officer |
| **G-09: User notified (if permitted)** | User notification sent before or at production (unless legally prohibited). Gag-order prohibition documented if applicable. `legal.user_notified` DEC-030 event emitted. | compliance-officer |
| **G-10: Transparency tally updated** | Annual report tally incremented for current reporting year. | compliance-officer |

---

## DEC-030 HMAC-Chained Audit Events

All government request handling events are emitted as DEC-030 HMAC-chained audit log entries per `docs/AUDIT_LOG_SCHEMA.md`. The HMAC chain ensures tamper evidence and provides SOC 2 P6.5 auditor-accessible records.

| Event type | Severity | Retention | When emitted |
|---|---|---|---|
| `legal.request_received` | HIGH | 7 years | T+0: instrument received and logged (Gate G-03) |
| `legal.counsel_retained` | MEDIUM | 7 years | Outside counsel engagement confirmed (Gate G-04) |
| `legal.legal_hold_activated` | HIGH | 7 years | `legal_hold_active` flag set on affected records (Gate G-05) |
| `legal.challenge_filed` | MEDIUM | 7 years | Motion to quash or narrowing letter filed with court |
| `legal.user_notified` | MEDIUM | 7 years | User notification sent (Gate G-09) |
| `legal.tenant_notified` | MEDIUM | 7 years | Enterprise tenant IT admin notified per R-13.9 |
| `legal.dpa_notified` | HIGH | 7 years | EU supervisory authority notified (Art. 48 cases) |
| `legal.disclosure_executed` | CRITICAL | 7 years | Data production delivered to requesting party (Gate G-08) |
| `legal.emergency_voluntary_disclosure` | CRITICAL | 7 years | Emergency life-safety voluntary disclosure per R-13.3 Constraint 5 |
| `legal.legal_hold_released` | MEDIUM | 7 years | Litigation hold lifted post-resolution |

**Privacy invariant for all legal.* events:** No health data values, coaching content, or form scores appear in any event payload. Payload contains only: `request_id` (UUID), `request_type` (taxonomy from R-13.1), `instrument_slug` (opaque reference to evidence file), `user_count` (integer — aggregate), `data_categories` (string array of category names, not values), `timestamp_utc`, `founder_authorized` (boolean), `counsel_name` (string — outside counsel firm name only).

No payload field may contain Art. 9 data values. If an audit event for a legal request were itself to contain health data, FORM would be creating a second copy of the data outside normal RLS controls — this is prohibited.

---

## GDPR Art. 48 Constraint Summary

FORM stores EU-resident user data in Supabase `eu-west-1` (Frankfurt). Non-EU court orders or government demands for EU-resident user data must satisfy GDPR Art. 48 before FORM can produce any data:

- The order must be based on an international agreement (MLAT or adequacy decision) in force between the EU and the requesting country.
- Where no treaty basis exists, FORM cannot comply except under narrow Art. 49 derogations (vital interests of the data subject — requires counsel confirmation).
- The competent EU supervisory authority (BfDI for German-resident users; the user's home-country DPA for others; Ireland DPC as lead DPA for cross-border cases) must be notified before production, unless a gag order prohibits notification.
- EU-resident user Art. 9 health data produced under a non-EU order without an Art. 48 treaty basis would constitute an unauthorized international transfer — FORM will not execute such a production without counsel confirming legal authority.

The full Art. 48 analysis procedure, MLAT status table, and DPA contact details are maintained in `docs/INCIDENT_RESPONSE.md §R-13.10`.

---

## No-Go Reaffirmation

ENTERPRISE.md §When we say no establishes the following unconditional constraints. This policy reaffirms them as legally enforceable obligations:

| Request type | FORM's position |
|---|---|
| Government backdoor — ongoing credential access, surveillance hook, or programmatic monitoring | Refused unconditionally. Report to legal counsel immediately. |
| Insurance company risk-scoring of employee health data | Refused unconditionally. |
| Wellness-tied insurance pricing data feed | Refused unconditionally. |
| Any request to bypass the FORM privacy floor (HR individual user visibility) | Refused unconditionally. No contract term or legal instrument overrides this. |
| Informal requests without compulsory legal instrument | Declined in writing. Direct to obtain compulsory process. |

Any team member who receives a request of this nature must notify the founder and compliance-officer immediately via Signal, preserve any written record of the request, and take no further action until counsel is retained.

---

## Operational Runbook Reference

The complete operational runbook for government requests — including response timelines, user and enterprise tenant notification templates, scope minimization tables, Art. 48 procedure, evidence package file structure, tabletop drill scenario H, and SOC 2 evidence mapping — is maintained in:

> `docs/INCIDENT_RESPONSE.md §R-13: Law Enforcement / Government Data Request`

This policy document provides the **governing principles and mandatory process gates**. The runbook provides the **step-by-step operational procedure**. Both must be consistent; any conflict is resolved in favour of the policy document (this file), with the runbook updated to match within 5 business days of any policy amendment.

---

## SOC 2 Mapping

| SOC 2 Criterion | How this policy satisfies it |
|---|---|
| **P6.5 — Disclosure to Third Parties: Government Requests** | Four principles operationalize FORM's no-backdoor commitment. Transparency report satisfies disclosure and notification obligations. DEC-030 `legal.disclosure_executed` provides the auditor-accessible per-disclosure record. |
| **CC6.1 — Logical access security over protected information** | FORM's refusal to comply with informal requests and the never-produce list (R-13.7 in the runbook) demonstrate that health data assets are protected from unauthorized government access as rigorously as from technical attackers. |
| **CC6.3 — Authorization of access to data** | This policy defines the authorization policy for government access to user data: compulsory legal process only, minimum necessary scope, founder authorization required, DEC-030 event as the auditable authorization record. |
| **C1.1 — Confidential information** | Art. 9 production constraints and the higher-legal-standard table (R-13.7) operationalize FORM's confidentiality commitments for health data specifically. |
| **CC9.1 — Business relationships** | Enterprise tenant notification protocol (Gate G-09 + Template T-01 in the runbook) ensures contractual DPA obligations to enterprise customers regarding legal demands are consistently executed. |

**Evidence artefacts for P6.5 (PRE-35-E-006):**

| Artefact | Location | Cadence |
|---|---|---|
| This policy document | `compliance/p1/gov-request-policy.md` | Per policy revision; outside counsel review required before each revision goes effective |
| Per-request case files | `compliance/evidence/legal-requests/<YYYY-MM-DD-slug>/` | Per incident; 7-year retention |
| Annual transparency report | `compliance/evidence/transparency-report/YYYY.md` | Annual |
| DEC-030 `legal.disclosure_executed` events | `audit_log_events` table (ClickHouse; 7-year WORM) | Per disclosure |
| DEC-030 `legal.legal_hold_activated` events | `audit_log_events` table | Per request |

---

## Sign-Off and Version History

| Version | Date | Change | Approved by |
|---|---|---|---|
| v0.1 (draft) | 2026-06-10 | Initial draft — closes P-GAP-006. Four governing principles, nine mandatory process gates, DEC-030 event schema, Art. 48 summary, SOC 2 mapping. | compliance-officer (authored) |
| v1.0 | Pending | Outside counsel (EU + US qualified) review and founder signature — required before publication as PRE-35-E-006 | compliance-officer + outside counsel + founder |

**Outside counsel review is a precondition for this document becoming effective.** Until v1.0 is signed, this draft satisfies the "authored" milestone of P-GAP-006 but does not fully close it. P-GAP-006 status moves from 🔴 Open → 🟡 Authored. It closes to 🟢 Closed upon v1.0 founder signature.

---

*v0.1 (2026-06-10): Initial government request handling policy. Closes P-GAP-006 to 🟡 Authored. Files as draft PRE-35-E-006. Four governing principles: (1) legal review before any action, (2) narrowest possible response, (3) user notification where legally permitted, (4) annual transparency report. Ten mandatory process gates (G-01 through G-10). Ten DEC-030 HMAC-chained audit events (`legal.*` series) with 7-year retention and privacy invariant on payload content. GDPR Art. 48 constraint summary for non-EU orders on EU-resident data with Supabase eu-west-1 anchor. No-go reaffirmation (five unconditional refusal categories from ENTERPRISE.md). SOC 2 P6.5, CC6.1, CC6.3, C1.1, CC9.1 mapping. Operational runbook pointer to INCIDENT_RESPONSE.md §R-13. Requires outside counsel (EU + US qualified) review and founder signature before PRE-35-E-006 is effective. Owner: compliance-officer + outside counsel.*
