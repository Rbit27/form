# Privacy Complaint Intake Procedure

**File reference:** P1-CIP-001  
**SOC 2 criterion:** P8.1 — Monitors and Communicates  
**GDPR articles:** Art. 12–13 (transparency) · Art. 15–22 (data subject rights) · Art. 77 (right to lodge complaint with supervisory authority)  
**Remediates:** P-GAP-007 (P1) — no formal privacy complaint intake process; no `privacy@form.coach` mailbox published; no 30-day response SLA  
**References:** `docs/SOC2_READINESS.md §35.10` · `docs/AUDIT_LOG_SCHEMA.md` (DEC-030) · `docs/INCIDENT_RESPONSE.md §10`  
**Owner:** compliance-officer  
**Decision authority:** compliance-officer + founder  
**Status:** 🟡 Procedure defined — pending mailbox creation (§9.1) and privacy policy publication (P-GAP-001)  
**Effective date:** 2026-05-21  
**Next review:** Annually in Q1 — or upon material change to GDPR compliance posture or supervisory authority guidance  
**Artifact path:** `compliance/p1/complaint-intake-procedure.md`  
**Evidence artifact:** PRE-35-E-011 (new — to be registered in `docs/SOC2_READINESS.md §38` evidence table)

---

## Table of Contents

1. [Purpose](#1-purpose)
2. [Scope](#2-scope)
3. [Intake Channel](#3-intake-channel)
4. [Response SLA](#4-response-sla)
5. [Triage Categories](#5-triage-categories)
6. [Response Procedure by Category](#6-response-procedure-by-category)
7. [DEC-030 Audit Events](#7-dec-030-audit-events)
8. [Complaint Log](#8-complaint-log)
9. [Open Items](#9-open-items)
10. [Escalation Path to Supervisory Authority](#10-escalation-path-to-supervisory-authority)
11. [Annual Review](#11-annual-review)
12. [Gap Closure Record](#12-gap-closure-record)

---

## 1. Purpose

FORM processes GDPR Article 9 special-category health data for both consumer users and enterprise employees. GDPR Art. 77 gives every data subject the right to lodge a complaint with a supervisory authority. Art. 12(2) requires that FORM *facilitates* the exercise of data subject rights (Art. 15–22), which means providing a working intake channel and meeting the response SLAs in Art. 12(3).

This document defines FORM's formal procedure for receiving, triaging, responding to, and logging privacy complaints and data subject rights requests. It closes **P-GAP-007** in the SOC 2 Type II gap register and:

1. Enables closure of **P-GAP-001** (privacy policy publication) — the `privacy@form.coach` contact address documented here must appear in the published privacy policy before any enterprise pilot data flows begin and before the SOC 2 observation period starts.
2. Satisfies **SOC 2 P8.1** — evidence that FORM has a documented, operated complaint handling and monitoring process.
3. Documents the Art. 77 escalation path to supervisory authorities.
4. Defines six new **DEC-030** HMAC-chained audit events for complaint lifecycle tracking.

**Privacy floor constraint.** No complaint response process may result in individual user health data being disclosed to HR personnel or tenant administrators. The k-anonymity floor (N < 5) and the `data.read_individual` forbidden-event rule remain in force regardless of the nature of the complaint. If a complaint relates to enterprise-tier data, the response is directed to the data subject, not to their employer.

---

## 2. Scope

This procedure covers requests received at `privacy@form.coach` from the following parties:

| Request type | GDPR basis | Channel | Art. 9 flag? |
|---|---|---|---|
| Privacy complaint (general) | Art. 77 | `privacy@form.coach` | Evaluate at triage |
| Data subject access request (DSAR fallback) | Art. 15 | `privacy@form.coach` (if in-app DSAR unavailable or incomplete) | Always `true` |
| Rectification | Art. 16 | `privacy@form.coach` | Evaluate at triage |
| Erasure | Art. 17 | `privacy@form.coach` (fallback to in-app where available) | Always `true` |
| Processing restriction | Art. 18 | `privacy@form.coach` | Evaluate at triage |
| Data portability | Art. 20 | `privacy@form.coach` | Always `true` |
| Objection to processing | Art. 21 | `privacy@form.coach` | Evaluate at triage |
| Automated decision-making | Art. 22 | `privacy@form.coach` | Evaluate at triage |

**Out of scope for this document:**

| Topic | Handled by |
|---|---|
| In-app DSAR requests (Settings → Privacy → Download My Data) | Automated DSAR flow — `docs/SOC2_READINESS.md §35.7.2` |
| Data breach notifications to supervisory authorities | `docs/INCIDENT_RESPONSE.md §10` (72h GDPR Art. 33 clock) |
| Sub-processor data requests | `compliance/p1/sub-processor-register.md §8` |
| Government / law enforcement requests | `compliance/p1/gov-request-policy.md` (P-GAP-006 — to be created) |
| Enterprise customer DPA disputes | `compliance-officer` + outside counsel |

---

## 3. Intake Channel

**Primary channel:** `privacy@form.coach`

This mailbox must be:

1. **Created before the privacy policy is published (P-GAP-001)** — the policy must reference a live, monitored contact address for Art. 12(2) compliance. See §9.1.
2. **Monitored every business day** (Monday–Friday, excluding public holidays in the jurisdiction of the compliance-officer).
3. **Accessible only to:** `compliance-officer` and the founder. No HR, no `tenant_admin`, no `tenant_manager`.
4. **Subject to backup access:** if the compliance-officer is unavailable for more than 2 consecutive business days, the founder assumes monitoring responsibility. Absence coverage must be confirmed at the start of any planned absence.

**Acknowledgement requirement.** GDPR Art. 12(3) requires that FORM responds to data subject requests *without undue delay* and within one month. To reduce supervisory risk, FORM's internal standard is to send a written acknowledgement within 3 business days of receipt. The acknowledgement must include:

- Confirmation that the request was received.
- FORM's reference number: `P1-CIP-<YYYY>-<NNN>` (sequential, e.g. `P1-CIP-2026-001`).
- Expected response date (30 calendar days from the date of receipt).
- A link to `form.coach/privacy` for the privacy policy.

**Identity verification.** Before responding to any request, FORM must verify the requester's identity with reasonable certainty. For registered users: match the sender's email address against the registered `user_id`. If the email does not match, request confirmation via the registered email address. For third-party representatives (e.g., legal counsel acting on behalf of a data subject): require written authorisation from the data subject. Do not request unnecessary identification — Art. 12(6) permits proportionate verification only.

---

## 4. Response SLA

| Stage | Deadline | Clock basis |
|---|---|---|
| Acknowledgement | 3 business days | Date of receipt in mailbox |
| Full response | 30 calendar days | Date of receipt in mailbox (Art. 12(3)) |
| Extension notice (complex or multiple requests) | Before day 30 | Date of original receipt |
| Maximum response period with extension | 90 calendar days | Date of original receipt |

**Extensions (Art. 12(3)).** GDPR Art. 12(3) permits extending the response period by a further 2 months (total 3 months from receipt) for complex or multiple requests. Extension requirements:

1. Communicate the extension **before the original 30-day deadline**.
2. Provide reasons for the delay.
3. Log the extension decision in `compliance/p1/complaint-log.csv` (field: `extended`, value: `true`; field: `extension_reason`).

**Supervisory escalation trigger.** If FORM has not sent a full response and has not communicated a valid extension before the 30-day deadline, the `compliance-officer` must notify the appropriate supervisory authority within 7 days of the missed deadline. Log `privacy.complaint_escalated` DEC-030 event (§7).

**SLA monitoring.** Until an automated 25-day alert is configured in PagerDuty (§9.8), the compliance-officer performs a weekly review of open items in `compliance/p1/complaint-log.csv` on each Monday, comparing `received_at` against the current date. Any item within 7 days of the 30-day deadline with `response_sent_at` blank receives immediate attention.

---

## 5. Triage Categories

Assign a category at first review. Multiple categories may apply to a single complaint; log the primary category in the CSV and note secondary categories in the `notes` field.

| Category code | GDPR basis | Description | Response owner | DEC-030 event |
|---|---|---|---|---|
| **CAT-01** | Art. 15 | DSAR fallback — access request by email because in-app DSAR was unavailable, incomplete, or the user is unable to use it | compliance-officer + engineering | `privacy.complaint_received` |
| **CAT-02** | Art. 16 | Rectification of inaccurate or incomplete personal data | compliance-officer + engineering | `privacy.complaint_received` |
| **CAT-03** | Art. 17 | Erasure — right to be forgotten; covers cases not satisfied by the in-app deletion flow | compliance-officer + engineering | `privacy.complaint_received` |
| **CAT-04** | Art. 18 | Processing restriction — requester disputes accuracy, objects to processing, or is pending Art. 21 objection outcome | compliance-officer | `privacy.complaint_received` + `privacy.processing_restricted` |
| **CAT-05** | Art. 20 | Data portability — export in structured, machine-readable format | engineering | `privacy.complaint_received` |
| **CAT-06** | Art. 21 | Objection to processing — including direct marketing opt-out and profiling objection | compliance-officer | `privacy.complaint_received` + `privacy.objection_received` |
| **CAT-07** | Art. 77 | General privacy complaint not mapping to CAT-01 through CAT-06 | compliance-officer | `privacy.complaint_received` |
| **CAT-08** | Art. 22 | Automated decision-making challenge — applies where Victor AI output is claimed to have produced a legally or similarly significant effect | compliance-officer + product + outside counsel | `privacy.complaint_received` + `privacy.automated_decision_review_requested` |

**Art. 9 flag.** If the request involves any of the following tables, mark `art9: true` in the complaint log and require compliance-officer sign-off on the response before sending:

- `health_profile` (explicit health data: age, sex, weight, injury flags, goal category)
- `meal_log` (dietary data)
- `wearable_readings` (HRV, resting HR, sleep, steps)
- `cv_sessions` (biometric-adjacent movement data)
- `coaching_turns` (health goals, injury context, coaching history)

All FORM users' data is Art. 9-adjacent given the nature of the product. Treat `art9: true` as the default unless the complaint concerns account/billing data only.

---

## 6. Response Procedure by Category

All responses must be:
- Written and sent from `privacy@form.coach`.
- Addressed to the verified data subject (not the enterprise employer, not the tenant admin).
- In plain language (GDPR Art. 12(1)).
- Signed by the compliance-officer or founder.
- Logged in `compliance/p1/complaint-log.csv` before the email is sent (write the log entry with `resolution_type: pending`, then update after send).

### 6.1 CAT-01 — DSAR Fallback (Art. 15)

1. Verify identity per §3.
2. Determine why the in-app DSAR was unavailable or incomplete — log the root cause.
3. Trigger the DSAR export manually via the admin tool: `data.export_initiated` DEC-030 event (engineering to perform, compliance-officer to authorise).
4. Deliver a signed URL to the requester's verified email address. URL expiry: 48 hours.
5. Log `data.export_completed` DEC-030 event.
6. If the root cause was a product bug, open a bug ticket and reference it in the complaint log notes.

### 6.2 CAT-02 — Rectification (Art. 16)

1. Verify identity per §3.
2. Identify the specific fields the data subject claims are inaccurate or incomplete. Request written specification if ambiguous.
3. Engineering applies the correction in the relevant Supabase table. For `user_profile` health fields: direct SQL update by engineering with compliance-officer authorisation.
4. Log `data.profile_updated` DEC-030 event with `fields_changed[]` and `old_values_hash` (the existing event schema).
5. Send written confirmation of the correction, identifying which fields were changed.
6. **Limitation note.** Coaching turn content (`coaching_turns`) is AI-generated output, not a factual claim stored *about* the user. Rectification of coaching turn content is not in scope under Art. 16. If this is raised, advise the data subject accordingly and note Art. 22 applicability (CAT-08).

### 6.3 CAT-03 — Erasure (Art. 17)

1. Verify identity per §3.
2. Confirm the legal basis for erasure. Document which Art. 17(1) ground applies (a–f).
3. Check for applicable exemptions under Art. 17(3) — particularly (b) legal obligation and (e) public interest. Document findings. If an exemption applies, the partial erasure procedure applies (erase everything except what the exemption covers).
4. Trigger erasure via the same procedure as the in-app deletion flow: `data.individual_deletion` DEC-030 event. Erasure scope:
   - Primary Postgres tables: `user_profile`, `workout_sessions`, `sets`, `coaching_turns`, `cv_sessions`, `meal_log`, `wearable_readings` — hard delete.
   - ClickHouse analytics events: manual deletion via `ALTER TABLE … DELETE WHERE user_id = ?` or wait for 2-year TTL (TTL is the default; manual deletion required only if explicitly requested within TTL window).
   - Sentry crash reports: assess whether any crash reports contain the user's `user_id`. If the `beforeSend` scrubber operated correctly, user_id should be absent; confirm before marking complete.
   - PostHog: user deletion via PostHog user deletion API.
   - Audit log: HMAC-chained audit log rows are **not deleted** — they are retained for 7 years (DEC-030 retention policy). The existence of audit log rows does not constitute "processing" of the user's data for the purpose of Art. 17 — it is an accountability obligation under Art. 5(2). Document this position in the response.
5. Send written confirmation of erasure, specifying which data categories were deleted and which (if any) were retained under an exemption.
6. Log `data.individual_deletion` DEC-030 event if not already triggered by the in-app flow.

### 6.4 CAT-04 — Processing Restriction (Art. 18)

1. Verify identity per §3.
2. Confirm which Art. 18(1) ground applies: (a) accuracy disputed, (b) processing unlawful but erasure refused, (c) FORM no longer needs data but data subject requires it for legal claims, (d) Art. 21 objection pending outcome.
3. Set `processing_restricted: true` on the user record. This column does not yet exist — see §9.2. Until it exists: apply an application-layer flag via the feature flags table and document the interim measure.
4. Cease all non-storage processing of the restricted user's data during the restriction period.
5. Log `privacy.processing_restricted` DEC-030 event.
6. Notify the requester in writing of the restriction activation and its scope.
7. When the basis for restriction is resolved (e.g., accuracy verified, objection resolved), notify the data subject **before** lifting the restriction, per Art. 18(3).

### 6.5 CAT-05 — Data Portability (Art. 20)

1. Verify identity per §3.
2. Art. 20 covers data "provided by" the data subject (health_profile fields entered by the user, meal_log entries, workout_sessions) and data "observed" by the controller where processed on the basis of consent (wearable_readings synced with the user's permission, cv_sessions).
3. Use the same signed-URL export as CAT-01. Confirm export format is JSON (machine-readable, structured).
4. AI-generated coaching turns (`coaching_turns`) — portability applies to the *input* data that the user provided (goal context, check-ins), not to the AI-generated outputs. Discuss with outside counsel if challenged.
5. Deliver to requester within the 30-day SLA. URL expiry: 48 hours.
6. Log `data.export_completed` DEC-030 event.

### 6.6 CAT-06 — Objection to Processing (Art. 21)

1. Verify identity per §3.
2. Determine the processing purpose the data subject objects to:
   - **Direct marketing or profiling for marketing:** Cease immediately. No override permitted under Art. 21(3). Confirm in writing within 30 days.
   - **Other processing:** Assess whether FORM has compelling legitimate grounds that override the data subject's interests, rights, and freedoms, or grounds for legal claims (Art. 21(1)). Document the balance of interests assessment in writing.
3. If FORM has compelling legitimate grounds: communicate these in the response within 30 days.
4. If FORM cannot demonstrate compelling legitimate grounds: cease the processing objected to.
5. Log `privacy.objection_received` DEC-030 event with `processing_purpose` and `objection_basis` fields.
6. Send written outcome within 30 days.

### 6.7 CAT-07 — General Privacy Complaint

1. Acknowledge within 3 business days.
2. Investigate the specific concern — review relevant data practices, logs, and system behaviour.
3. Where the complaint reveals a previously unidentified compliance gap: open a new gap entry in `docs/SOC2_READINESS.md` and reference the gap ID in the complaint log `notes` field.
4. Prepare a written response addressing each element of the complaint in plain language.
5. If the complaint cannot be resolved to the data subject's satisfaction, proactively inform them of their right to escalate to the supervisory authority (§10) before the 30-day deadline.

### 6.8 CAT-08 — Automated Decision-Making (Art. 22)

FORM's current position is that Victor AI coaching outputs are personalised recommendations and do not constitute legally or similarly significant decisions under Art. 22(1). This position requires outside counsel confirmation before the privacy policy (P-GAP-001) is finalised — see §9.6.

**In the interim, any CAT-08 request is handled as follows:**

1. Verify identity per §3.
2. Log `privacy.automated_decision_review_requested` DEC-030 event.
3. Escalate to compliance-officer + product team + outside counsel **same business day**.
4. Do not respond substantively until outside counsel confirms the Art. 22 position.
5. Send acknowledgement within 3 business days (§3) noting that the request is under review.
6. Respond within 30 days with the outcome of the legal and product review, including FORM's Art. 22 position and any remedial action taken.

---

## 7. DEC-030 Audit Events

All complaint lifecycle events must be written to the DEC-030 HMAC-chained audit log. The following six events are defined for this procedure and must be added to `docs/AUDIT_LOG_SCHEMA.md` in the privacy event taxonomy section (§ "Privacy-sensitive" events).

Until they are added to `docs/AUDIT_LOG_SCHEMA.md`, use the existing `privacy.consent_granted` event as a structural template — same HMAC chain mechanism, same seven-year retention class.

| Event name | Trigger point | Required fields | Retention |
|---|---|---|---|
| `privacy.complaint_received` | At triage assignment — before acknowledgement is sent | `reference_id` (P1-CIP-\<YYYY\>-\<NNN\>), `category` (CAT-01 through CAT-08), `art9` (boolean), `received_at` (ISO 8601), `user_id` (if known and verified; omit if identity unverified) | 7 years |
| `privacy.complaint_resolved` | When response is sent to data subject | `reference_id`, `category`, `resolution_type` (`fulfilled` / `partially_fulfilled` / `declined_exemption` / `escalated_to_dpa`), `resolved_at` (ISO 8601), `days_elapsed` (integer) | 7 years |
| `privacy.complaint_escalated` | When FORM refers a complaint to a supervisory authority, or when a data subject notifies FORM they are escalating | `reference_id`, `supervisory_authority` (e.g. `"ICO"`, `"DPC"`, `"APD"`), `reason` (`"sla_breach"` / `"unresolved_complaint"` / `"requester_initiated"`), `escalated_at` (ISO 8601) | 7 years |
| `privacy.processing_restricted` | On activation of processing restriction for a user (CAT-04) | `user_id`, `reference_id`, `restriction_basis` (Art. 18(1)(a)–(d) reference), `restricted_at` (ISO 8601) | 7 years |
| `privacy.objection_received` | On receipt of Art. 21 objection (CAT-06) | `user_id`, `reference_id`, `processing_purpose` (free text), `objection_basis` (`"direct_marketing"` / `"legitimate_interests"` / `"public_interest"`), `received_at` (ISO 8601) | 7 years |
| `privacy.automated_decision_review_requested` | On receipt of any Art. 22 challenge (CAT-08) | `user_id`, `reference_id`, `coaching_turn_id` (Supabase UUID if the specific coaching turn is identified; otherwise `null`), `received_at` (ISO 8601) | 7 years |

**Retention justification.** All six events are classified at the same retention tier as `privacy.consent_*` (7 years). Basis: GDPR Art. 5(2) accountability — FORM must be able to demonstrate it received and responded to complaints, including to supervisory authorities and in litigation. Seven years aligns with the general contractual limitation period in most EU member states.

**HMAC chain requirement.** All six events must be chained under DEC-030. An unchained complaint event is a control deficiency — escalate to `devops-lead` and log in `compliance/cc4/control-deficiency-log.csv`.

**No personal data in event payload.** Event payloads must not contain the data subject's name, email address, or the content of the complaint. These remain in the `privacy@form.coach` mailbox. The `reference_id` is the join key between the audit log and the mailbox record.

---

## 8. Complaint Log

The complaint log CSV provides the auditor-facing evidence artifact and the primary input for the annual transparency report. It does not replace the DEC-030 audit events — both are required.

**File path:** `compliance/p1/complaint-log.csv`

**Schema:**

```csv
reference_id,received_at,category,requester_type,user_id_hashed,art9,extended,extension_reason,response_sent_at,days_elapsed,resolution_type,supervisory_escalated,notes
P1-CIP-YYYY-NNN,ISO-8601,CAT-XX,data_subject|third_party_rep|enterprise_tenant_admin|supervisory_authority,sha256(user_id)|anonymous,true|false,true|false,reason or blank,ISO-8601|blank,integer|blank,fulfilled|partially_fulfilled|declined_exemption|escalated_to_dpa|pending,true|false,brief factual summary
```

**Field descriptions:**

| Field | Type | Description |
|---|---|---|
| `reference_id` | string | `P1-CIP-<YYYY>-<NNN>` — sequential within calendar year; reset to 001 each January |
| `received_at` | ISO 8601 date | Date the email was received in `privacy@form.coach` |
| `category` | enum | CAT-01 through CAT-08; comma-separate if multiple |
| `requester_type` | enum | `data_subject` / `third_party_rep` (legal representative) / `enterprise_tenant_admin` / `supervisory_authority` |
| `user_id_hashed` | string | SHA-256 hex of the Supabase `user_id`; `anonymous` if identity was not verified (e.g., requester declined to provide email) |
| `art9` | boolean | `true` if the request involves any Art. 9 data category |
| `extended` | boolean | `true` if the Art. 12(3) extension was invoked |
| `extension_reason` | string | Required if `extended: true`; blank otherwise |
| `response_sent_at` | ISO 8601 date | Date response was sent; blank if pending |
| `days_elapsed` | integer | `response_sent_at` − `received_at` in calendar days; blank if pending |
| `resolution_type` | enum | `fulfilled` / `partially_fulfilled` / `declined_exemption` / `escalated_to_dpa` / `pending` |
| `supervisory_escalated` | boolean | `true` if the data subject escalated to a DPA or FORM escalated on their own initiative |
| `notes` | string | Brief factual summary — **must not contain** the data subject's name, email, or the content of the complaint; may reference gap IDs or bug ticket numbers |

**Prohibited content.** The CSV must not contain: requester name, email address, the body of the complaint, or any personal data. These are held exclusively in the `privacy@form.coach` mailbox thread. The `user_id_hashed` field uses a one-way hash so that a CSV leak does not expose user identifiers.

**Access control.** The file is committed to the private compliance repository. Access: `compliance-officer` and founder only. Not accessible to `tenant_admin`, HR, or any external party without explicit legal process.

---

## 9. Open Items

These items must be resolved before P-GAP-007 is fully closed and before the privacy policy (P-GAP-001) is published.

| # | Item | Owner | Priority | Milestone |
|---|---|---|---|---|
| 9.1 | Create `privacy@form.coach` mailbox; confirm monitoring cadence and backup-access procedure with compliance-officer and founder | compliance-officer | **P0 — pre-publication gate** | Before P-GAP-001 |
| 9.2 | Add `processing_restricted BOOLEAN DEFAULT false NOT NULL` column to `users` table in Supabase; update RLS policies to block processing-restricted users from non-storage queries | platform-engineer | P1 | M3 |
| 9.3 | Add six `privacy.complaint_*` events to `docs/AUDIT_LOG_SCHEMA.md` privacy event taxonomy section; implement event emission in the admin complaint-handling tool | platform-engineer | P1 | M3 |
| 9.4 | Add `privacy@form.coach` contact and Art. 15–22 rights summary to Settings → Privacy → "Contact us about your data" deep-link in the mobile app | engineering | P1 | Pre-launch |
| 9.5 | Add `privacy@form.coach` contact address and data subject rights section to privacy policy draft (P-GAP-001); confirm with outside counsel | compliance-officer + outside counsel | **P0 — blocks P-GAP-001 finalisation** | Before P-GAP-001 |
| 9.6 | Outside counsel to confirm FORM's Art. 22 position re: Victor AI coaching outputs — legally or similarly significant decision? — before privacy policy publication | outside counsel | P1 — Art. 22 position affects policy §Data subject rights | Before P-GAP-001 |
| 9.7 | Create `compliance/p1/complaint-log.csv` with schema header row and the first example entry (blank / template row) | compliance-officer | P1 | Before first complaint is received |
| 9.8 | Configure PagerDuty (or Better Stack) alert: if a `privacy.complaint_received` DEC-030 event exists without a matching `privacy.complaint_resolved` event after 23 days, fire a P1 alert to `compliance-officer` | devops-lead | P1 | M3 |
| 9.9 | Register this document as **PRE-35-E-011** in `docs/SOC2_READINESS.md §38` evidence table under P8.1 | compliance-officer | P1 | With P-GAP-007 closure |
| 9.10 | Add `privacy.complaint_*` retention class to `docs/AUDIT_LOG_SCHEMA.md §Retention policy` table (7 years, same class as `privacy.consent_*`) | platform-engineer | P1 | M3 |

---

## 10. Escalation Path to Supervisory Authority

If FORM cannot resolve a complaint to the requester's satisfaction within 30 days, or the requester explicitly states their intention to escalate to a supervisory authority, FORM must:

1. **Inform the requester in writing** that they have the right to lodge a complaint with a supervisory authority. This must be included in the final response regardless of whether FORM was able to satisfy the request.
2. **Provide the relevant supervisory authority contact:**

| Jurisdiction | Supervisory Authority | Contact |
|---|---|---|
| United Kingdom | Information Commissioner's Office (ICO) | `ico.org.uk/concerns` |
| European Union (general) | Lead EU DPA — to be confirmed by outside counsel (Ireland DPC if FORM's EU establishment is Ireland) | `dataprotection.ie` (DPC) |
| EU member state (resident's DPA) | DPA of the member state of habitual residence of the data subject | Via EDPB DPA directory |
| Ukraine | Уповноважений Верховної Ради України з прав людини (Parliamentary Commissioner for Human Rights) | `ombudsman.gov.ua` |

3. **Log `privacy.complaint_escalated` DEC-030 event** with `supervisory_authority` and `reason` fields.
4. **Update `compliance/p1/complaint-log.csv`** with `supervisory_escalated: true` and `resolution_type: escalated_to_dpa`.
5. **If the supervisory authority initiates a formal investigation:** notify the founder and outside counsel immediately. Do not respond to the supervisory authority without outside counsel review.

**FORM's position.** Informing a data subject of their right to escalate is not an admission of fault or non-compliance. It is a statutory obligation under GDPR Art. 12(2) and Art. 77. FORM's goal is to resolve complaints before escalation — the escalation path is disclosed as a transparency obligation, not as a signal that FORM expects to fail.

---

## 11. Annual Review

This review is part of the P-GAP-008 closure commitment. Add the entry below to `docs/SOC2_READINESS.md §15` compliance calendar.

| Review item | Frequency | Owner | Calendar entry |
|---|---|---|---|
| `compliance/p1/complaint-log.csv` review — complaint volume, SLA adherence, category distribution, escalation rate | Annual (Q1) | compliance-officer | Q1 Privacy Review |
| Supervisory authority contact details — verify DPA addresses and jurisdiction mapping table (§10) remain current | Annual | compliance-officer | Q1 Privacy Review |
| Procedure reviewed against updated GDPR guidance or supervisory authority decisions | Annual | compliance-officer + outside counsel | Q1 Privacy Review |
| Annual transparency report published (aggregate statistics from complaint log: requests received by category, SLA compliance rate, escalation count) | Annual | compliance-officer + founder | Q1 Privacy Review — publish by end of March |
| Art. 22 position (§6.8) re-evaluated with counsel if Victor model architecture or output use changed materially | Annual or on major model change | compliance-officer + outside counsel | Q1 Privacy Review or ad hoc |
| `privacy@form.coach` mailbox access audit — confirm current access list matches authorised roles | Annual | compliance-officer | Q1 Privacy Review |

**First annual review date:** Launch date + 12 months. The first transparency report covers the 12-month period from launch.

---

## 12. Gap Closure Record

| Gap ID | SOC 2 Criterion | Description | Previous status | Status after this document |
|---|---|---|---|---|
| **P-GAP-007** | P8.1 | No formal privacy complaint intake process; no `privacy@form.coach` mailbox published; no 30-day response SLA; no complaint log | 🔴 Open | 🟡 **Partially closed** — procedure defined; five open items remain (§9.1–9.5) |

Full closure of P-GAP-007 requires **all five** of the following to be complete:

- [x] This document created (`compliance/p1/complaint-intake-procedure.md`)
- [ ] `privacy@form.coach` mailbox confirmed active and monitored (§9.1)
- [ ] Six `privacy.complaint_*` events added to `docs/AUDIT_LOG_SCHEMA.md` (§9.3)
- [ ] Settings → Privacy deep-link added in the mobile app (§9.4)
- [ ] Privacy policy published with `privacy@form.coach` contact address (§9.5 + P-GAP-001)

**Dependencies on other gaps:**

| Dependency | Direction |
|---|---|
| P-GAP-001 (privacy policy publication) | P-GAP-007 must partially close before P-GAP-001 can close — the mailbox address must be in the policy |
| P-GAP-006 (gov-request-policy.md) | Independent — no blocking dependency |
| P-GAP-008 (annual review calendar) | §11 of this document provides the calendar entry content; P-GAP-008 closure incorporates §11 |

**SOC 2 evidence registration:** This document files as **PRE-35-E-011**. To register: add a row to the evidence table in `docs/SOC2_READINESS.md §38` under criterion P8.1 with path `compliance/p1/complaint-intake-procedure.md`, cadence "Per procedure change; annual review", owner `compliance-officer`, status 🟡 Partially — mailbox and DEC-030 events pending.

---

*P1-CIP-001 · v0.1 · 2026-05-21 · Drafted by enterprise-builder (FORM) · Pending compliance-officer review and outside counsel sign-off on §6.8 (Art. 22 position)*

*Co-Authored-By: Claude (enterprise-builder) <cloud@form.coach>*
