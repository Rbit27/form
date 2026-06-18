# FORM · Master Service Agreement · Template v0.3

> **Internal use only — PRE-LEGAL-REVIEW DRAFT.**
> This template must be reviewed and approved by outside counsel before execution with any customer.
> Owner: `compliance-officer` + `enterprise-architect` + founder.
> Exhibits: Exhibit B = `docs/ENTERPRISE_SLA.md`; Exhibit C = `docs/SUBPROCESSORS.md §5` (DPA); Exhibit D = TOMs.
> References: `docs/ENTERPRISE.md`, `docs/AUDIT_LOG_SCHEMA.md` (DEC-030), `docs/ENTERPRISE_SLA.md`, `docs/SUBPROCESSORS.md`.
> Versions: standard terms (< 2,000 seats — self-service redline up to ±10%); negotiated (≥ 2,000 seats — outside counsel required).
> Filing: executed copies to `compliance/contracts/{CUSTOMER_SLUG}/msa-v{N}-{DATE}.pdf`; 7-year retention.

---

## TL;DR

This is the master commercial agreement that governs every enterprise customer relationship. It binds: services, pricing, SLA (Exhibit B), data processing (Exhibit C), and Privacy Floor. The Privacy Floor in §8 is **non-negotiable at any contract value** — confirm this with the customer before any redline begins. DPA (Exhibit C) must be executed before any pilot data flows.

---

## MASTER SERVICE AGREEMENT

**This Master Service Agreement** ("Agreement") is entered into as of `[EFFECTIVE DATE]` ("Effective Date") by and between:

**FORM, Inc.**, a Delaware corporation ("FORM", "we", "us"), and

**`[CUSTOMER LEGAL NAME]`**, a `[JURISDICTION/ENTITY TYPE]` ("Customer", "you").

---

## ARTICLE 1 — DEFINITIONS

| Term | Meaning |
|---|---|
| **Agreement** | This MSA, together with all Exhibits, Order Forms, and Addenda, as amended from time to time. |
| **Admin Dashboard** | FORM's web application enabling Authorized Administrators to manage tenant configuration, user provisioning, and aggregate wellness metrics. |
| **Authorized Users** | Customer's employees, contractors, and affiliates who are granted access to the Services under an Order Form. |
| **Confidential Information** | Any information designated as confidential or that a reasonable party would consider confidential given the circumstances, excluding information that is publicly available, independently developed, or received from a third party without restriction. |
| **Customer Data** | All data, content, and information submitted to the Services by Authorized Users or provisioned via SCIM/SSO, including Personal Data. |
| **Documentation** | FORM's technical and user documentation, including `docs/ENTERPRISE_ONBOARDING.md`, `docs/SSO_CLIENT_CONFIG.md`, and the Admin Dashboard help centre. |
| **Order Form** | A mutually executed document specifying Services, seat count, pricing, contract period, and applicable Addenda. Each Order Form is governed by this Agreement. |
| **Personal Data** | Any information that identifies or could identify an individual Authorized User, as defined under applicable data-protection law. |
| **Privacy Floor** | The set of non-negotiable data-access restrictions in §8 of this Agreement. |
| **Services** | FORM's AI fitness coaching platform delivered as a multi-tenant SaaS, including the mobile application, Admin Dashboard, SSO/SCIM provisioning, and Victor AI coaching, as described in Exhibit A. |
| **SLA** | The service level commitments in Exhibit B (`docs/ENTERPRISE_SLA.md`). |
| **SOC 2 Type II** | Independent auditor attestation that FORM's security, availability, and confidentiality controls operated effectively over a minimum 6-month observation period. |
| **Sub-processor** | A third party engaged by FORM to process Customer Data in connection with the Services. Current register: `docs/SUBPROCESSORS.md`. |
| **Tenant** | Customer's isolated environment in the FORM platform, identified by a unique `tenant_id`. |

---

## ARTICLE 2 — SERVICES

### 2.1 Scope of Services

FORM will provide the Services described in Exhibit A and each applicable Order Form. Services are delivered via Cloudflare's global edge network with data residency in the region specified in the Order Form (EU: Frankfurt `eu-central-1`; US: `us-east-1`; other regions subject to availability).

### 2.2 Service Changes

FORM may modify the Services at any time, provided that:

- **Breaking changes** to the Admin Dashboard API, SSO metadata URLs, or SCIM endpoints require minimum 60 days written notice.
- **New sub-processors** require minimum 30 days advance notice and are subject to Customer's right to object per Exhibit C §6.
- **Planned maintenance** requiring Service interruption requires minimum 48 hours advance notice (Standard tier) or 72 hours (Premium tier), sent to `tenant_ops_email`. Emergency maintenance for P0 security incidents may proceed with notification within 1 hour of start. Full maintenance policy: `docs/ENTERPRISE_SLA.md §7`.

### 2.3 Professional Services

Implementation support (SSO configuration, white-label setup, training) is governed by a separately executed Statement of Work (SOW). This Agreement does not obligate FORM to provide professional services absent an executed SOW.

---

## ARTICLE 3 — LICENSE GRANT AND RESTRICTIONS

### 3.1 License Grant

Subject to Customer's payment obligations and compliance with this Agreement, FORM grants Customer a non-exclusive, non-transferable, non-sublicensable right to allow Authorized Users to access and use the Services during the Order Form term, solely for Customer's internal business purposes.

### 3.2 Restrictions

Customer must not, and must not allow any third party to:

1. Sublicense, resell, or provide the Services on a service-bureau or time-sharing basis to any third party.
2. Reverse-engineer, decompile, or disassemble any component of the Services.
3. Access the Services to build a competing product or service.
4. Use the Services to process data of individuals who have not been provisioned as Authorized Users.
5. Remove or obscure any proprietary notices or branding, except as permitted under a White-Label Addendum executed per §17.

### 3.3 Seat Compliance

Seats are licensed per individual Authorized User. Shared accounts are not permitted. Customer may substitute seats (remove a user, add a replacement) at no charge; **adding** seats above the Order Form quantity requires a written amendment or self-serve upgrade via the Admin Dashboard.

---

## ARTICLE 4 — ORDER FORMS, FEES, AND PAYMENT

### 4.1 Fees

Fees are specified in each Order Form. Pricing tiers follow `docs/ENTERPRISE.md §5`:

| Tier | Seats | Price | Notes |
|---|---|---|---|
| **Starter** | 50–200 | $12/seat/month (annual) | Self-serve redline |
| **Growth** | 200–1,000 | $9/seat/month | Dedicated CSM; quarterly reviews |
| **Enterprise** | 1,000+ | $6–8/seat/month (negotiated) | Multi-year; white-label option |

Multi-year discounts: 15% (2-year contract), 25% (3-year contract). Annual upfront discount: 10% (non-multi-year).

### 4.2 Invoicing and Payment

Unless otherwise specified in the Order Form: annual contracts are invoiced annually in advance; monthly contracts are invoiced monthly in advance. Payment terms: net 30 from invoice date.

### 4.3 Late Payment

Undisputed invoices unpaid after 30 days accrue interest at 1.5% per month (or the maximum permitted by law, if lower). FORM may suspend access to the Services on 10 business days' written notice if an undisputed invoice remains unpaid after 45 days. Suspension does not release Customer from payment obligations.

### 4.4 Taxes

Fees are exclusive of all taxes, levies, and duties. Customer is responsible for all applicable taxes, excluding taxes on FORM's net income.

### 4.5 Fee Changes

FORM may adjust fees for renewal terms with minimum 90 days written notice prior to the renewal date. Increases exceeding 10% per year require compliance-officer review before issuance of the notice.

---

## ARTICLE 5 — INTELLECTUAL PROPERTY

### 5.1 FORM IP

FORM owns all right, title, and interest in the Services, underlying technology, documentation, and all improvements thereto. Customer's feedback does not constitute a transfer of IP.

### 5.2 Customer Data

Customer retains all right, title, and interest in Customer Data. FORM acquires no intellectual property rights in Customer Data by virtue of this Agreement.

### 5.3 No AI Training Use

FORM will not use Customer Data — including Authorized User health data, workout logs, meal logs, biometric data, or any other Personal Data — to train, fine-tune, or otherwise improve any AI or machine learning model. This restriction applies to FORM directly and to all Sub-processors handling Customer Data. Anthropic's enterprise DPA (Sub-processor SP-07) contractually prohibits training use; see `docs/SUBPROCESSORS.md §2`. This is a condition of the enterprise relationship, not a configuration option.

### 5.4 Aggregate Analytics

FORM may use **anonymised, aggregate, non-attributable** data derived from all customers to improve product performance metrics, provided no Customer Data can be reasonably re-identified. Minimum cohort size k ≥ 5 applies to any published aggregate; FORM's internal analytics apply k ≥ 5 as a floor and targets k ≥ 10 for health categories.

---

## ARTICLE 6 — CONFIDENTIALITY

### 6.1 Obligations

Each party will: (a) protect the other party's Confidential Information with at least the same degree of care it uses for its own, and in no event less than reasonable care; (b) use Confidential Information only to exercise rights or fulfill obligations under this Agreement; (c) not disclose Confidential Information to any third party without prior written consent, except to employees and contractors who have a need to know and are bound by equivalent confidentiality obligations.

### 6.2 Compelled Disclosure

If required by law or valid legal process to disclose Confidential Information, the receiving party will: (a) provide prompt written notice to the disclosing party (where permitted by law); (b) cooperate with any reasonable request by the disclosing party to contest or limit the disclosure; (c) disclose only the minimum required.

### 6.3 Injunctive Relief

Each party acknowledges that breach of this Article would cause irreparable harm, and that injunctive relief may be sought without bond or proof of damages.

---

## ARTICLE 7 — SECURITY AND COMPLIANCE

### 7.1 FORM Security Obligations

FORM will maintain an information security programme that includes at minimum:

- Encryption of Customer Data at rest (AES-256) and in transit (TLS 1.3).
- Multi-factor authentication enforcement for all FORM personnel with access to production systems.
- Annual penetration test by an independent firm. A summary of findings and remediation status is shared with Customer upon written request.
- Tamper-evident, append-only audit logs for all privileged actions per DEC-030 (`docs/AUDIT_LOG_SCHEMA.md`). HMAC-chained entries; 7-year retention for SOC 2 / financial events; 3-year retention for authentication events.
- SOC 2 Type II attestation (target: Month 12 after enterprise launch). Current SOC 2 posture: `docs/SOC2_READINESS.md`. Annual attestation report shared with Customer on written request.
- Quarterly compliance calendar reviews; continuous compliance tooling (Vanta or equivalent).

### 7.2 GDPR and Data Protection

Where Customer Data includes Personal Data of EU/EEA data subjects, the Data Processing Agreement in Exhibit C governs. Exhibit C is incorporated by reference and has the same legal force as this Agreement. In the event of conflict between this Agreement and Exhibit C on data-protection matters, Exhibit C prevails.

### 7.3 Customer Security Obligations

Customer is responsible for: (a) securing all credentials and access tokens for the Admin Dashboard and API; (b) configuring SSO and SCIM correctly per `docs/SSO_CLIENT_CONFIG.md`; (c) timely de-provisioning of Authorized Users who depart or change roles; (d) notifying FORM promptly (within 24 hours) of any suspected credential compromise.

### 7.4 Security Questionnaires

FORM will complete Customer's standard vendor security questionnaires within 20 business days of receipt, referencing `docs/SECURITY_QUESTIONNAIRE.md` for standard responses. Custom questionnaires exceeding 100 items may be subject to a professional services fee.

---

## ARTICLE 8 — PRIVACY FLOOR (NON-NEGOTIABLE)

The following restrictions apply to **all** enterprise deployments regardless of contract value, Customer request, or any other provision of this Agreement. These are product principles that cannot be removed by contract. Compliance-officer and clinical-safety have veto authority; customer-success must confirm Customer acknowledgement before contract execution.

1. **No individual user data to HR or employers.** The Customer (including HR, People Operations, and management) has no access to any individual Authorized User's health data, workout data, meal logs, body composition data, or mental health data.
2. **Aggregates only.** Admin Dashboard metrics show aggregate cohort data only (e.g., percentage of users active, opt-in participation rate, anonymised wellness trend indicators). Minimum cohort size k ≥ 5 applies to all Admin Dashboard displays.
3. **No manager-report correlation.** Customer cannot access data broken down by reporting line, team, or any organisational structure that would enable individual identification.
4. **Revocation right.** Any Authorized User may revoke their company association at any time from their account settings. Revocation removes the user from Admin Dashboard aggregates immediately and prevents any future inclusion in employer-facing reporting. Revocation does not delete the user's personal FORM account.
5. **ED screening data excluded.** Eating disorder (ED) screening responses are never included in any aggregate, export, or dashboard view. Result is a UX flag only within the individual user's experience.
6. **Body composition excluded from aggregates.** Weight, body fat percentage, and body composition metrics are never aggregated to employer-facing reports.
7. **Mental health data excluded.** Any in-app data categorised as mental health, mood, or stress is never aggregated to employer-facing reports.
8. **No wellness-as-punishment.** FORM will not configure or enable features that penalise Authorized Users for non-participation, below-average metrics, or failure to meet wellness targets set by Customer.

Customer represents that its use of the Services complies with these restrictions and with applicable employment law. Customer will not attempt to identify individual users from aggregate reports, export data for individual performance evaluations, or request features that contravene this §8 without exception.

**Acceptance acknowledgement.** By signing this Agreement, Customer's authorised signatory confirms they have read and understood the Privacy Floor and agree it is non-negotiable.

---

## ARTICLE 9 — ACCEPTABLE USE

Customer and all Authorized Users must comply with the Acceptable Use Policy in Exhibit E (`docs/ACCEPTABLE_USE_POLICY.md`). Prohibited use cases include:

1. Insurance risk-scoring: using any FORM data to inform individual health risk ratings for insurance pricing or eligibility determinations.
2. Government backdoors: providing or enabling government access to individual user data outside of valid legal process.
3. Wellness-as-insurance-pricing: linking FORM participation or metrics to insurance premium calculation or underwriting.
4. Athlete evaluation: using individual performance data to evaluate athletes for team selection, contract decisions, or salary negotiations (sports-league use case).
5. Participation penalties: structuring any corporate wellness programme so that Authorized Users who do not participate or who fall below wellness targets face adverse employment, insurance, or benefits consequences.

FORM may terminate this Agreement immediately for cause upon Customer's material breach of this §9.

---

## ARTICLE 10 — CUSTOMER OBLIGATIONS (CUEC)

Customer Utilisation Enablement Controls (CUECs) are Customer's responsibilities that are assumed in FORM's SOC 2 Type II attestation. Failure to fulfil these obligations may affect the efficacy of FORM's security controls:

1. **Identity hygiene.** Customer ensures that SSO/SCIM user data is accurate and that terminated employees are de-provisioned within 24 hours of departure.
2. **Voluntary participation.** Customer confirms that Authorized User participation in FORM's wellness programme is voluntary and freely chosen, as required for valid GDPR Art. 9 consent by Authorized Users.
3. **Data accuracy.** Customer provides accurate organisational data (employee count, region) for correct data residency configuration.
4. **Security incident reporting.** Customer notifies FORM within 24 hours of discovery of any security incident that may involve the Services.
5. **Authorised access only.** Customer does not share Admin Dashboard access credentials or API keys between individual administrators.
6. **Compliance with Privacy Floor.** Customer does not attempt to access, derive, or reconstruct individual user data from aggregate reports or exports.

---

## ARTICLE 11 — TERM AND TERMINATION

### 11.1 Term

This Agreement begins on the Effective Date and continues until all Order Forms expire or are terminated. Each Order Form specifies its initial contract period (minimum 12 months for enterprise). Order Forms auto-renew for successive 12-month periods unless either party gives written notice of non-renewal at least 90 days before the renewal date.

### 11.2 Termination for Cause

Either party may terminate this Agreement or an individual Order Form for material breach if:
- The breach is not cured within 30 days of written notice (or 10 days for payment defaults).
- The breaching party is the subject of insolvency or bankruptcy proceedings.

FORM may terminate immediately upon Customer's breach of §8 (Privacy Floor) or §9 (Acceptable Use).

### 11.3 Termination for Convenience

Customer may terminate an Order Form with 90 days written notice. Fees for the remaining contract period are not waived on termination for convenience; prepaid annual fees for the remaining period are forfeited. The Early Termination Fee framework in §11.4 governs the calculation of amounts owed on early termination.

### 11.4 Early Termination Fee

**8.1 Liquidated Damages.** If Customer terminates for convenience before the end of the contracted subscription term, Customer agrees to pay FORM an Early Termination Fee ("ETF") representing a genuine pre-estimate of FORM's losses from the early termination, including unamortised customer acquisition cost and lost recurring revenue. The ETF is not a penalty.

**8.2 ETF Calculation.** The ETF is calculated on Customer's Contracted Annual Contract Value ("ACV") — the actual contracted rate post any multi-year discount — not the list price.

**8.3 Declining Balance Rate Schedule.**

| Exit at month | ETF rate (% of remaining contracted ACV) |
|---|---|
| M1–M3 | 60% |
| M4–M6 | 50% |
| M7–M12 | 40% |
| M13–M18 | 30% |
| M19–M24 | 20% |
| M25–M30 | 10% |
| M31–M36 | 0% + minimum floor |
| M37+ (multi-year renewal term) | Resets to M1 schedule for the new term |

**8.4 Minimum Floor.** The ETF shall not be less than one (1) month of the contracted ACV, regardless of the rate schedule above.

**8.5 Seat Reduction Policy.** Contracted seat counts are fixed for the term. Seat reductions of ten percent (10%) or fewer of contracted seats may be deferred to the next annual renewal date at no ETF. Seat reductions greater than ten percent (10%), or reductions that would bring the account below the tier minimum seat count, trigger an ETF calculated on the ACV reduction amount using the rate schedule in §8.3 applied to the remaining term.

**8.6 Waiver Authority.** FORM may waive the ETF in whole or in part at its sole discretion, subject to an expected-value analysis confirming the waiver exceeds enforcement net present value. Any waiver of an ETF exceeding USD 5,000 requires approval by FORM's founder. Any waiver exceeding USD 100,000 requires approval by FORM's founder and a lead investor. ETF waivers are logged as immutable audit events per FORM's DEC-030 policy.

**8.7 Counsel Review Checkpoint.** *[⚠ OUTSIDE COUNSEL REVIEW REQUIRED BEFORE FIRST MULTI-YEAR CONTRACT — M10. Confirm enforceability of liquidated damages framing in DE (§§339–340 BGB) and US (NY/DE preferred; CA requires special consideration). Confirm ACV basis vs list ACV (current position: contracted ACV). See docs/COST_MODEL.md OQ-ETF-01 and OQ-ETF-02.]*

**8.8 Audit Record.** Any ETF waiver and any contract amendment (including price-lock renewals and seat reductions) is logged as an immutable HMAC-chained DEC-030 audit event (`enterprise.early_termination_fee_waived`, `enterprise.contract_amended`) and retained for 7 years per FORM's audit log policy (`docs/AUDIT_LOG_SCHEMA.md`).

### 11.5 Effect of Termination

Upon termination or expiry:
- FORM will continue to provide the Services during any applicable notice period.
- Authorized User access is suspended on the termination effective date.
- Data export and deletion obligations apply per §12.

---

## ARTICLE 12 — DATA EXPORT AND DELETION

### 12.1 Export Window

FORM provides Customer Data export capability per the following schedule. Export is initiated by the Customer via the Admin Dashboard (CSV/JSON) or via a written request to `enterprise@form.coach`.

| Termination type | Export window | Data format |
|---|---|---|
| Non-renewal / planned termination | 30 days from contract end date | CSV, JSON |
| Customer-initiated early termination | 14 days from notice receipt | CSV, JSON |
| FORM-initiated termination for cause | 14 days from notice | CSV, JSON |
| Mutual termination / other | 30 days from agreement | CSV, JSON |

### 12.2 Art. 9 Health Data — Zero Grace Period

Special category health data (workout records, biometric data, meal logs, ED-screening flags) is subject to **no grace period** for deletion upon Authorized User account closure or tenant offboarding. Within 1 hour of the data deletion trigger event: health data is hard-deleted from the primary Supabase database. Backups purge on their standard rotation schedule (maximum 7 days for daily snapshots). This zero-grace-period rule is an absolute invariant per DEC-036; it cannot be overridden by contract, Customer request, or legal hold (except for active litigation hold on non-health data). See `docs/DATA_MODEL.md §8.2`.

### 12.3 Post-Export Deletion

Within 30 days after the export window closes, FORM will:
1. Delete all Customer Data (including Personal Data) from production systems.
2. Purge all backup snapshots containing Customer Data on their natural rotation cycle (maximum 30 days).
3. Provide written certification of deletion to Customer within 10 business days of completion.

FORM retains: (a) anonymised, aggregate data that cannot be attributed to Customer; (b) audit log entries required for legal or regulatory compliance (per `docs/AUDIT_LOG_SCHEMA.md` retention schedules); (c) invoice records required by financial law.

### 12.4 Regulatory Retention Hold

Where FORM is subject to a valid regulatory or legal obligation requiring retention of Customer Data beyond the deletion timeline, FORM will: (a) notify Customer in writing within 5 business days; (b) apply the minimum retention required; (c) apply access controls equivalent to those in `docs/CRYPTOGRAPHY_POLICY.md` for the retained data; (d) delete as soon as the obligation expires.

---

## ARTICLE 13 — WARRANTIES

### 13.1 FORM Warranties

FORM warrants that:
1. The Services will perform materially in accordance with the Documentation.
2. FORM will not knowingly introduce malware or unauthorised code into the Services.
3. FORM has the right to grant the licence in §3.1.
4. FORM's security programme meets the standards described in §7.1 and `docs/SECURITY.md`.

### 13.2 Customer Warranties

Customer warrants that:
1. Customer has all rights necessary to provide Customer Data to FORM.
2. Customer's use of the Services will comply with applicable law, including employment law and GDPR where applicable.
3. Authorised User participation is voluntary and consented, as required for valid Art. 9 health data processing.

### 13.3 Disclaimer

EXCEPT AS EXPRESSLY SET OUT IN §13.1, THE SERVICES ARE PROVIDED "AS IS". FORM DISCLAIMS ALL WARRANTIES, EXPRESS OR IMPLIED, INCLUDING FITNESS FOR A PARTICULAR PURPOSE AND NON-INFRINGEMENT. FORM DOES NOT WARRANT SPECIFIC HEALTH OUTCOMES, WEIGHT LOSS, FITNESS IMPROVEMENTS, OR ANY OTHER INDIVIDUAL HEALTH RESULT.

---

## ARTICLE 14 — INDEMNIFICATION

### 14.1 FORM Indemnification

FORM will indemnify, defend, and hold harmless Customer and its officers, employees, and agents from claims by third parties that the Services as provided by FORM infringe any patent, copyright, trademark, or trade secret, provided that Customer: (a) promptly notifies FORM of the claim; (b) gives FORM sole control of defence and settlement; (c) provides reasonable assistance.

If the Services are subject to an infringement claim, FORM may at its option: (i) obtain a licence to continue; (ii) modify the Services to be non-infringing without material functional loss; or (iii) terminate the relevant Order Form and refund prepaid fees for the unused period.

### 14.2 Customer Indemnification

Customer will indemnify, defend, and hold harmless FORM from claims arising from: (a) Customer Data; (b) Customer's use of the Services in violation of this Agreement, applicable law, or Privacy Floor; (c) Customer's breach of the CUEC obligations in §10; (d) any claims by Authorised Users that Customer misrepresented the voluntary nature of participation.

---

## ARTICLE 15 — LIMITATION OF LIABILITY

### 15.1 Exclusion of Consequential Damages

NEITHER PARTY WILL BE LIABLE FOR INDIRECT, INCIDENTAL, SPECIAL, CONSEQUENTIAL, EXEMPLARY, OR PUNITIVE DAMAGES, INCLUDING LOST PROFITS, LOSS OF DATA, OR BUSINESS INTERRUPTION, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGES.

### 15.2 Aggregate Cap

FORM's total cumulative liability to Customer under or in connection with this Agreement (whether in contract, tort, or otherwise) will not exceed the greater of: (a) the total fees paid or payable by Customer in the **twelve months** immediately preceding the event giving rise to the claim; or (b) USD $1,000,000 per incident.

### 15.3 Exceptions to Cap

The liability cap in §15.2 does not apply to:
1. Breaches of §8 (Privacy Floor).
2. Breaches of §6 (Confidentiality) involving Customer Data.
3. Indemnification obligations under §14.
4. Gross negligence or wilful misconduct.
5. Deaths or personal injury caused by negligence.

### 15.4 SLA Sole Remedy

Service Credits under Exhibit B are the Customer's sole and exclusive remedy for SLA availability breaches. They do not limit Customer's rights under §14 (indemnification) or §8 (Privacy Floor breach).

---

## ARTICLE 16 — DATA BREACH NOTIFICATION

### 16.1 FORM Notification Obligations

FORM will notify Customer's designated security contact (`tenant_security_email`) without undue delay and within **72 hours** of FORM becoming aware of a Personal Data breach affecting Customer Data, providing:
1. Nature of the breach and approximate number of affected individuals.
2. Categories of Personal Data involved.
3. Likely consequences.
4. Measures taken or proposed to address the breach.

Where not all information is available within 72 hours, FORM will provide an initial notice and follow up with complete information as soon as available.

### 16.2 FORM Regulatory Notification

For EU/EEA Customer Data, FORM (as processor) will assist Customer (as controller) in meeting any notification obligation to supervisory authorities or data subjects under GDPR Art. 33–34, per Exhibit C.

### 16.3 Customer Notification Obligations

Customer must notify FORM within 24 hours of becoming aware of any security incident that may affect the Services or FORM's systems.

### 16.4 Status Channel

Real-time incident status is available at `https://status.form.coach` (Better Stack). This channel is referenced in this Agreement as the primary breach communication channel. See `docs/ENTERPRISE_SLA.md §7` for maintenance and incident communication procedures.

---

## ARTICLE 17 — WHITE-LABEL ADDENDUM (CONDITIONAL)

White-label features are available to customers at or above $50,000 ARR only. A White-Label Addendum, incorporated into this Agreement on execution, governs:

1. **Custom domain:** CNAME to `[tenant].form.coach` or `[tenant-custom-domain]`. FORM manages TLS.
2. **Logo replacement:** Tenant logo replaces FORM logo in the mobile app and Admin Dashboard.
3. **Colour override:** Up to 3 brand colours, subject to WCAG 2.1 AA accessibility floor. FORM's `brand-system` agent verifies compliance.
4. **"Powered by FORM" attribution:** Footer attribution remains non-removable for white-label contracts below $50,000 ARR. Attribution may be removed above $50,000 ARR per mutual written agreement.
5. **Branding restrictions:** Customer may not use FORM's visual identity, Fraunces typeface, or FORM's design system elements outside the white-label product surface without separate written licence.

---

## ARTICLE 18 — DISPUTE RESOLUTION

### 18.1 Informal Resolution

The parties will attempt to resolve any dispute through good-faith negotiation for 30 days before commencing formal proceedings.

### 18.2 Governing Law and Jurisdiction

For US customers: this Agreement is governed by the laws of the State of Delaware, without regard to conflict-of-law principles. Disputes are subject to binding arbitration administered by JAMS under its Commercial Arbitration Rules, with proceedings in New York, NY.

For EU/EEA customers: this Agreement is governed by the laws of Ireland. Disputes are subject to the exclusive jurisdiction of the courts of Dublin, Ireland, without prejudice to mandatory local consumer-protection law.

### 18.3 SLA Disputes

SLA disputes not resolved within 20 business days escalate to the dispute resolution process in §18.2. The evidentiary record is FORM's DEC-030 audit log and Better Stack uptime report for the relevant period.

---

## ARTICLE 19 — GENERAL PROVISIONS

### 19.1 Entire Agreement

This Agreement, including all Exhibits and Order Forms, constitutes the entire agreement between the parties and supersedes all prior agreements, proposals, and representations regarding the subject matter.

### 19.2 Order of Precedence

In the event of conflict: (1) this MSA body text; (2) Exhibit C (DPA) on data-protection matters; (3) Order Form on commercial terms; (4) remaining Exhibits. The Privacy Floor (§8) prevails over all conflicting terms regardless of position in this hierarchy.

### 19.3 Amendments

This Agreement may be amended only by written instrument signed by authorised representatives of both parties. Online click-through terms for add-on features do not amend this Agreement.

### 19.4 Assignment

Neither party may assign this Agreement without prior written consent, except in connection with a merger, acquisition, or sale of all or substantially all of its assets, provided the assignee assumes all obligations. FORM may assign to an affiliate without consent.

### 19.5 Force Majeure

Neither party is liable for delay or failure to perform due to events outside its reasonable control, including natural disasters, governmental actions, internet infrastructure failures, or pandemic. The affected party must notify the other promptly and resume performance as soon as reasonably practicable.

### 19.6 Severability

If any provision is held invalid or unenforceable, the remaining provisions continue in full force. The parties will negotiate in good faith to replace the invalid provision with a lawful provision that most closely achieves the intent.

### 19.7 Waiver

Failure to enforce any provision is not a waiver of future enforcement rights.

### 19.8 Notices

Legal notices under this Agreement must be in writing and delivered by: (a) courier with confirmed delivery; (b) email with read receipt to the addresses specified in the Order Form. Routine operational notices (maintenance, SLA reports) may be delivered by email to `tenant_ops_email`.

### 19.9 Counterparts

This Agreement may be executed in counterparts (including electronic signature via DocuSign), each of which is an original and all of which together constitute one instrument.

---

## EXHIBIT A — ORDER FORM

```
Order Form No.: OF-[CUSTOMER_SLUG]-[YYYYMMDD]
Effective Date: [DATE]
Contract Period: [START_DATE] to [END_DATE] ([N]-year)
Auto-renewal: Yes / No (circle)

Customer:            [CUSTOMER LEGAL NAME]
Customer Address:    [ADDRESS]
Billing Contact:     [NAME / EMAIL]
Technical Contact:   [NAME / EMAIL]
Security Contact:    [NAME / EMAIL]
GDPR Rep (EU):       [NAME / EMAIL]   ← required for EU customers

Services:            FORM Enterprise SaaS Platform
                     Mobile application (iOS + Android)
                     Admin Dashboard
                     SSO / SCIM provisioning (WorkOS)
                     AI coaching (Victor — Anthropic Claude API)
                     Voice coaching (ElevenLabs)

Seat Count:          [N] Authorized Users
Pricing Tier:        □ Starter ($12/seat/mo)
                     □ Growth ($9/seat/mo)
                     □ Enterprise ($[X]/seat/mo — negotiated)
Annual Contract Value: $[ACV]
Multi-year discount: □ 15% (2-year) □ 25% (3-year) □ None
Upfront discount:    □ 10% (annual upfront) □ None

Data Residency:      □ EU (Frankfurt)  □ US (us-east-1)  □ Other: ______
White-label:         □ Yes (Addendum 1 required)  □ No
CCPA Addendum:       □ Yes (for CA/US customers)  □ No
SIEM Addendum:       □ Yes (Addendum 4 required if SIEM export enabled)  □ No
Support Tier:        □ Standard (email 24h)  □ Growth (Slack 4h)  □ Enterprise (phone + Slack 24×7)

Exhibits incorporated:
  □ Exhibit B — Service Level Agreement
  □ Exhibit C — Data Processing Agreement
  □ Exhibit D — Technical and Organizational Measures
  □ Exhibit E — Acceptable Use Policy
  □ Addendum 1 — White-Label (if applicable)
  □ Addendum 2 — US Privacy / CCPA SPA (if applicable)
  □ Addendum 4 — SIEM Data Processing (if SIEM export enabled)

FORM signatory:      _________________________ Date: _________
                     [NAME], [TITLE]

Customer signatory:  _________________________ Date: _________
                     [NAME], [TITLE]
```

---

## EXHIBIT B — SERVICE LEVEL AGREEMENT

Incorporated by reference: `docs/ENTERPRISE_SLA.md` v1.0 (or the version current at Effective Date, filed to `compliance/contracts/{CUSTOMER_SLUG}/sla-exhibit-b-v{N}.pdf`).

Key commitments:

| Metric | Standard | Premium |
|---|---|---|
| Monthly Uptime | 99.9% | 99.95% |
| P0 Resolution | ≤ 4 hours | ≤ 2 hours |
| P0 Acknowledgement | ≤ 15 minutes | ≤ 10 minutes |
| GDPR Breach Notification | ≤ 72 hours | ≤ 72 hours |
| Max Monthly Credit | 50% of MRR | 50% of MRR |
| Maintenance Notice | 48 hours | 72 hours |

Service Credits are Customer's sole and exclusive remedy for availability SLA breaches per §15.4.

---

## EXHIBIT C — DATA PROCESSING AGREEMENT

Incorporated by reference: `docs/SUBPROCESSORS.md §5` ("Enterprise DPA Template"), version current at Effective Date, filed to `compliance/contracts/{CUSTOMER_SLUG}/dpa-exhibit-c-v{N}.pdf`.

This DPA governs FORM's processing of Personal Data as a data processor on behalf of Customer as data controller, in accordance with GDPR Art. 28. Execution of this DPA is a condition precedent to any pilot or production data flow.

Key elements (per `docs/SOC2_READINESS.md §13` DPA requirements):
- Purpose limitation: processing only for service delivery, never for AI training.
- Security obligations: SOC 2 Type II controls, `docs/CRYPTOGRAPHY_POLICY.md`.
- Sub-processor change notification: 30 days advance notice; right to object.
- Audit rights: FORM responds to written audit requests within 10 business days.
- Data deletion: on termination per §12 of this Agreement.
- Breach notification: ≤ 72 hours to Customer (FORM notifies supervisory authority independently).
- Standard Contractual Clauses (SCCs): Module 2 (controller→processor) incorporated for EU→US transfers for active service processing (real-time API, AI coaching, Auth, SSO/SCIM). For EU-region Customers (`data_region IN ('eu-central-1', 'eu-west-1')`): off-boarding export packages stored exclusively in Cloudflare R2 EU jurisdiction — no Chapter V GDPR transfer mechanism required for that processing activity. See Addendum 3 (EU Data Residency Terms).

---

## EXHIBIT D — TECHNICAL AND ORGANIZATIONAL MEASURES (TOMs)

Summary of FORM's security controls (full detail: `docs/SECURITY.md`, `docs/SOC2_READINESS.md §3`):

| Control area | Measure |
|---|---|
| Encryption at rest | AES-256 via Supabase (AWS KMS-managed keys) |
| Encryption in transit | TLS 1.3 minimum; HSTS; certificate managed by Cloudflare |
| Key rotation | Quarterly rotation per `docs/KEY_ROTATION.md` |
| Access control | Role-based (tenant_owner / tenant_admin / tenant_manager); MFA enforced for all admin roles |
| Audit logging | Append-only HMAC-chained audit log per DEC-030; `docs/AUDIT_LOG_SCHEMA.md` |
| Tenant isolation | Row-Level Security (RLS) policies on all Supabase tables; `tenant_id` partitioning; `docs/DATA_MODEL.md §3` |
| Vulnerability management | Annual penetration test (third-party firm); quarterly dependency review; findings tracked in Linear |
| Incident response | `docs/INCIDENT_RESPONSE.md`; P0–P3 classification; post-mortem within 48h |
| Business continuity | `docs/BUSINESS_CONTINUITY.md`; RPO 4 hours; RTO 12 hours |
| Personnel security | `docs/ONBOARDING_SECURITY.md`; annual security awareness training; NDA at hire |
| Sub-processors | DPAs executed with all T1 sub-processors; register at `docs/SUBPROCESSORS.md` |
| CV / on-device ML | Pose estimation runs on-device; no raw video transmitted to FORM servers |

---

## EXHIBIT E — ACCEPTABLE USE POLICY

Incorporated by reference: `docs/ACCEPTABLE_USE_POLICY.md` (version current at Effective Date).

Prohibited use cases explicitly prohibited under §9 of this Agreement:
1. Insurance risk-scoring on employee health data.
2. Government or law enforcement backdoors.
3. Wellness-as-insurance-pricing programmes.
4. Athlete selection or salary/contract determination.
5. Adverse consequences for non-participation.

---

## ADDENDUM 1 — WHITE-LABEL TERMS

*(Applicable only if Exhibit A Order Form indicates White-label: Yes.)*

Executed separately. Requirements: ARR ≥ $50,000. Customer provides brand assets (logo, hex codes, font) to FORM's `brand-system` agent for accessibility review (WCAG 2.1 AA). Timeline: 10 business days from asset delivery to staging deployment; 5 business days for production. "Powered by FORM" footer: removable above $50,000 ARR only with explicit written agreement.

---

## ADDENDUM 2 — US PRIVACY / CCPA SPA ADDENDUM

*(Applicable for US enterprise customers where Customer is a business under CCPA/CPRA.)*

Executed separately. Governs: CCPA service-provider obligations; prohibition on selling or sharing Authorised User Personal Information; DSAR support; opt-out signal handling. This addendum is executed in the same DocuSign envelope as Exhibit C (DPA) for US customers. Template: `compliance/templates/us-privacy-spa-addendum-v{N}.pdf` (USP-E-005). Reference: `docs/SOC2_READINESS.md §22669`.

---

## ADDENDUM 3 — EU DATA RESIDENCY TERMS

*(Applicable for EU enterprise customers where Order Form specifies `data_region` as `eu-central-1` (Frankfurt) or `eu-west-1` (Dublin).)*

> **PRE-LEGAL-REVIEW DRAFT.** Outside counsel review required before execution with any EU enterprise Customer — see Addendum 3.6.

Execute in the same DocuSign envelope as Exhibit C (DPA).

---

### Addendum 3.1 Applicability

This Addendum applies where Customer's Order Form specifies a European Union data residency region (`eu-central-1` or `eu-west-1`). It supplements Exhibit C and, for the specific processing activities listed in §3.3, supersedes the Standard Contractual Clauses (SCCs) statement in Exhibit C.

For US-region Customers (`data_region: us-east-1`): this Addendum does not apply; SCC Module 2 governs all processing including off-boarding export packages.

---

### Addendum 3.2 EU Data Residency Commitment

FORM commits to the following data-residency controls for EU-region Customers:

1. **Off-boarding export packages.** Customer Data export packages generated during tenant off-boarding are stored exclusively in Cloudflare R2 EU jurisdiction (`form-offboarding-exports-eu`). Technical enforcement: routing function `resolveEgressBucket()` maps `eu-central-1` and `eu-west-1` to the EU bucket without exception. Chain invariant OFB-REGION-01 (DEC-061, 2026-06-15; see `docs/DATA_MODEL.md §36.6`) is enforced at emission time in the `emit-audit-event` Worker: any event where `is_eu_region: true` and `r2_bucket ≠ 'form-offboarding-exports-eu'` is rejected with HTTP 422, triggers P1 PagerDuty `form-platform`, and is investigated as a potential data-residency breach per `docs/INCIDENT_RESPONSE.md R-05`.

2. **Health data (GDPR Art. 9).** Authorized User health data — workout history, biometric data, AI coaching transcripts, CV pose estimation keypoints — is stored in Supabase Postgres (EU region per Order Form). No Art. 9 data is included in any off-boarding export package; this is enforced structurally by `egress_package_type_enum` — no enum value produces an individual health-data export path. Privacy Floor (`docs/ENTERPRISE.md §8`) applies without exception.

3. **Audit log.** The HMAC-chained audit log (DEC-030; `docs/AUDIT_LOG_SCHEMA.md`) is stored in Supabase Postgres (EU region). Quarterly chain exports confirming OFB-REGION-01 compliance are filed as evidence artefact OFB-E-005 and available under Exhibit C audit rights.

---

### Addendum 3.3 DPA Annex B — Data Transfer Mechanism Declaration

For EU-region Customers, the following table constitutes DPA Annex B and replaces the single SCC statement in Exhibit C on a per-activity basis:

| Processing Activity | Data Location | Transfer Mechanism (GDPR Chapter V) | Notes |
|---|---|---|---|
| **Off-boarding export packages** (`enterprise.data_export_completed`) | Cloudflare R2 EU jurisdiction (`form-offboarding-exports-eu`) | **None required.** Data stored and processed within EU/EEA; no transfer to third countries for this activity. | Enforcement: OFB-REGION-01 chain invariant. Annual confirmation: OFB-E-006. |
| **Active service processing** (real-time API, AI coaching via Victor, SSO/SCIM, Admin Dashboard) | Cloudflare Workers (global edge) + Supabase Postgres (EU region) | **SCC Module 2** (controller → processor). FORM Inc. (Delaware, US) as processor; Cloudflare Ireland Ltd (EU) as sub-processor with Cloudflare DPA EU addendum. | SCCs signed and on file per `docs/SUBPROCESSORS.md §5`. |
| **Analytics** (PostHog — aggregate, pseudonymised) | PostHog EU Cloud (`eu.posthog.com`) | **None required** for PostHog EU Cloud deployment. | Aggregate data only; individual health-category data prohibited per `docs/OBSERVABILITY.md §25.9`; k ≥ 5 pseudonymisation floor. |
| **Error monitoring** (Sentry) | Sentry EU region | **SCC Module 2** if applicable. | Sentry processes technical error metadata only; no Customer Data content. |

**Outside counsel review note:** The "None required" declaration for off-boarding packages depends on Cloudflare R2 EU jurisdiction constituting intra-EEA storage with no onward transfer to Cloudflare US entities under current Cloudflare DPA arrangements. FORM must retain annual OFB-E-006 evidence (Cloudflare R2 EU jurisdiction screenshot) confirming this. If Cloudflare modifies its R2 EU data-processing terms, FORM must re-evaluate and attach SCCs until a replacement mechanism is confirmed.

---

### Addendum 3.4 Evidence Commitments

FORM will maintain and make available on written request under Exhibit C audit rights:

| Artefact | Description | Cadence | Path |
|---|---|---|---|
| **OFB-E-005** | Export of `enterprise.data_export_completed` HMAC-chained events confirming `is_eu_region: true` and `r2_bucket = 'form-offboarding-exports-eu'` for all Customer off-boarding operations. Zero-event periods filed as affirmative attestation. SOC 2 criteria: C1.1/P4.0/CC6.1. | Quarterly | `compliance/evidence/offboarding/OFB-E-005_<YYYY-QN>.csv` |
| **OFB-E-006** | Cloudflare R2 EU jurisdiction configuration screenshot + `wrangler r2 bucket get form-offboarding-exports-eu --json` output confirming `location: "eu"`. IAM policy export confirming `form-api` has no read access. SOC 2 criteria: C1.1/CC6.1. | Annual | `compliance/evidence/offboarding/OFB-E-006_<YYYY>.md` |

Source definitions: `docs/DATA_MODEL.md §36.7` (DEC-061, 2026-06-15).

---

### Addendum 3.5 Governing Law and Conflict

Governing law for this Addendum: Ireland (EU-region Customers), same as Exhibit C (DPA). In the event of conflict between this Addendum and Exhibit C, this Addendum prevails on the specific processing activities listed in §3.3.

Cross-references: `docs/DATA_MODEL.md §36` (DEC-061 — EU routing architecture); `docs/GDPR_DPIA.md §10.3` (k-anonymity floor open question); `docs/SOC2_READINESS.md §67.9` (CC6.1/C1.1 criteria mapping).

---

### Addendum 3.6 Outside Counsel Review Requirement

This Addendum **must** be reviewed by outside counsel before execution with any EU enterprise Customer. Review points:

1. Confirm "None required" Chapter V declaration for Cloudflare R2 EU is correct under current Cloudflare DPA and GDPR Chapter V.
2. Confirm Schrems II Transfer Impact Assessment (TIA) is not required for Cloudflare Ireland Ltd processing.
3. Confirm SCC Module 2 coverage is adequate for active-service processing (Cloudflare Workers global edge).
4. Confirm k ≥ 5 pseudonymisation floor satisfies EDPB guidance on Art. 9 health-category analytics.
5. Confirm EU governing law (Ireland) is enforceable against FORM Inc. (Delaware).

Filing: outside counsel sign-off memo to `compliance/contracts/{CUSTOMER_SLUG}/eu-dpa-annex-b-counsel-signoff-v{N}.pdf` before DocuSign execution.

---

## ADDENDUM 4 — SIEM DATA PROCESSING ADDENDUM

*(Applicable only if Customer enables SIEM export via the FORM Admin Dashboard. Must be signed before FORM may transmit DEC-030 HMAC-chained audit events to a Customer-designated endpoint.)*

> **PRE-LEGAL-REVIEW DRAFT.** Outside counsel review required before first production execution. Signing mechanism: DocuSign envelope initiated from Admin Dashboard consent capture UI gate (`docs/OBSERVABILITY.md §47.6`). Template version: v1.0 (2026-06-18).

---

**Addendum 4 — SIEM Data Processing Addendum v1.0**

This Addendum is incorporated into and forms part of the Master Subscription Agreement ("MSA") between FORM Health Ltd ("FORM") and the enterprise tenant identified in the Order Form ("Customer"). Capitalised terms not defined here have the meaning given in the MSA.

---

### Addendum 4.1 Clause 1 — Scope and Activation

This Addendum supplements the Master Subscription Agreement ("MSA") between FORM Health Ltd ("FORM") and the enterprise tenant ("Customer"). This Addendum governs FORM's transmission of DEC-030 HMAC-chained audit log events ("Audit Events") from FORM's `emit-audit-event` infrastructure to Customer's designated SIEM endpoint ("Designated Endpoint"). This Addendum becomes effective on the date Customer's authorised representative (as defined in the MSA) accepts it through the FORM Admin Dashboard in-product consent flow ("Effective Date"). FORM will not activate the Audit Event transmission pipeline until the Effective Date.

**SIEM-CONSENT-01 chain invariant:** FORM's `emit-audit-event` Worker will not emit `siem.tenant_export_enabled` for Customer's tenant unless a valid `siem.consent_addendum_signed` DEC-030 HMAC-chained event has been recorded for Customer's `tenant_id`. Any attempt to activate export without a prior signed Addendum 4 will be rejected with HTTP 422 `SIEM_CONSENT_01_NO_ADDENDUM`.

---

### Addendum 4.2 Clause 2 — Documented Instructions (GDPR Art. 28(3)(a))

Customer, acting as data controller for the Audit Event stream it receives through the Designated Endpoint, hereby provides FORM with written documented instructions to transmit Audit Events to the Designated Endpoint. These instructions are effective from the Effective Date and remain in force until revoked by Customer through the FORM Admin Dashboard "Revoke SIEM Addendum" action or until termination of the MSA. FORM shall not transmit Audit Events to any endpoint other than the Designated Endpoint specified at acceptance time without Customer providing updated documented instructions through the same mechanism.

**Legal basis:** These documented written instructions satisfy GDPR Art. 28(3)(a), which requires that FORM as processor act only on documented instructions from Customer as controller. A UI click alone does not constitute documented instructions; this signed Addendum is the required written record.

---

### Addendum 4.3 Clause 3 — Customer Obligations

Customer warrants that:

(a) it has an adequate legal basis under applicable data protection law (including GDPR Art. 6(1) or an equivalent provision) for processing the Audit Events received at the Designated Endpoint;

(b) the Designated Endpoint is operated under an appropriate data processing agreement or sub-processing arrangement where required by applicable law;

(c) Customer will not attempt to re-identify individual FORM users from pseudonymised fields in Audit Events (including `actor_hash`, `user_sha256`, `endpoint_url_hash`, or IP subnet fields);

(d) Customer will retain Audit Events received at the Designated Endpoint for no longer than the period permitted by applicable law and Customer's internal data retention policy;

(e) Customer will notify FORM immediately if Customer becomes aware that Audit Events have been accessed by an unauthorised party.

---

### Addendum 4.4 Clause 4 — FORM Obligations

FORM warrants that:

(a) Audit Events transmitted to the Designated Endpoint are processed in accordance with FORM's privacy floor — no plaintext `user_id`, no Art. 9 special category health data, no body composition values, no coaching content, and no plaintext SIEM endpoint URL are included in any transmitted event;

(b) each transmitted event is HMAC-chained and includes `chain_hash` and `prev_hash` fields enabling Customer to verify chain integrity;

(c) FORM will notify Customer within 72 hours of becoming aware of a breach affecting Audit Events in transit to the Designated Endpoint;

(d) FORM maintains a DEC-030 chain record of this Addendum acceptance (`siem.consent_addendum_signed` HIGH 7yr) and any subsequent revocation (`siem.consent_addendum_revoked` HIGH 7yr) to support Customer's audit and compliance obligations.

---

### Addendum 4.5 Clause 5 — Revocation

Customer may revoke this Addendum at any time by clicking "Revoke SIEM Addendum" in the FORM Admin Dashboard. Revocation takes effect within 5 minutes of the Customer action (one DEC-030 event cycle). Upon revocation:

(a) FORM will cease transmission to the Designated Endpoint within 5 minutes;

(b) FORM will emit a `siem.consent_addendum_revoked` DEC-030 event (HIGH, 7yr) recording the revocation;

(c) Customer acknowledges that events already transmitted to the Designated Endpoint before revocation remain subject to Customer's data retention obligations under Clause 3(d);

(d) Customer may re-activate transmission by accepting a new Addendum 4 through the Admin Dashboard — a new `siem.consent_addendum_signed` chain record will be emitted before re-activation.

---

### Addendum 4.6 Evidence and Audit Record

FORM maintains quarterly SOC 2 evidence artefact **SIEM-CONSENT-E-001** (CC9.2/CC1.1): a DEC-030 export of all `siem.consent_addendum_signed` and `siem.consent_addendum_revoked` events per tenant, confirming that no tenant SIEM export was activated without a prior signed Addendum 4. Cross-check query: `SELECT COUNT(*) FROM tenant_siem_configs WHERE export_enabled = true AND addendum_signed_at IS NULL` must return zero. Filed quarterly to `compliance/evidence/siem-consent/SIEM-CONSENT-E-001_<YYYY-QN>.csv`; available to Customer under Exhibit C audit rights.

**Privacy floor on artefact:** No individual employee `user_id`, health data, or Art. 9 special category data in any artefact. `endpoint_url_hash` is SHA-256[:32] truncated. `signed_by_email_hash` is SHA-256 of the authorised representative email — no plaintext email stored in the chain record. `tenant_id` is org slug only.

Full specification: `docs/OBSERVABILITY.md §47` (DEC-065, 2026-06-18).

---

### Addendum 4.7 Governing Law and Conflict

Governing law for this Addendum follows the MSA: Delaware (US Customers); Ireland (EU Customers per Addendum 3). In the event of conflict between this Addendum and the MSA or Exhibit C (DPA), this Addendum prevails on the specific processing activities it governs (Audit Event transmission to the Designated Endpoint).

---

### Addendum 4.8 Outside Counsel Review Requirement

This Addendum must be reviewed by outside counsel before first production execution. Review points:

1. Confirm Clause 2 satisfies GDPR Art. 28(3)(a) documented-instruction requirement for this data flow pattern (FORM-as-processor transmitting to tenant-designated third-party endpoint operated by tenant-as-controller).
2. Confirm Clause 3(a) customer warranty is adequate — or whether FORM should require evidence of Customer's legal basis rather than a warranty alone.
3. Confirm 72-hour breach notification in Clause 4(c) is consistent with MSA §14 and GDPR Art. 33/34 timelines.
4. Confirm Clause 5 revocation mechanism satisfies GDPR Art. 28(3)(a) instruction-withdrawal requirements.
5. Confirm DocuSign electronic acceptance constitutes valid "written" instruction under applicable GDPR supervisory authority guidance.

Filing: outside counsel sign-off memo to `compliance/contracts/msa-addendum-4-v1.0-counsel-signoff.pdf` before first production execution.

---

## INTERNAL NOTES (REMOVE BEFORE SENDING TO CUSTOMER)

| Topic | Guidance |
|---|---|
| Privacy Floor acknowledgement | Must get Customer signatory to explicitly confirm §8 in writing. Do not waive or negotiate. |
| DPA must precede data flows | Exhibit C must be signed before any SSO/SCIM configuration or pilot data is ingested. |
| Insurance / gov use cases | If in discovery Customer mentions insurance integration, gov data access, or wellness penalties — stop and escalate to founder before proceeding. See `docs/ENTERPRISE.md §10`. |
| k-anonymity floor | EU enterprise customers: legal review required before sales if k ≥ 5 threshold may be too low for EDPB guidance on health-category aggregates. Open item `docs/GDPR_DPIA.md §10.3`. |
| Voluntary participation CUEC | Confirm with Customer HR lead that participation is not de facto mandatory. GDPR Art. 9 consent requires freely-given consent; employment pressure may invalidate. See `docs/SOC2_READINESS.md OQ-P2-02`. |
| Governing law | Delaware for US; Ireland for EU. If Customer insists on a different EU jurisdiction, escalate to outside counsel before agreeing. |
| SLA credit | Credits are sole remedy for uptime breaches. Do not negotiate away the exclusivity clause without compliance-officer review. |
| AI training prohibition | §5.3 is non-negotiable. Anthropic DPA already prohibits training use; this reinforces it contractually at the customer layer. |
| SOC 2 attestation timing | SOC 2 Type II target is Month 12 after enterprise launch. For early customers signed before attestation: offer SOC 2 bridge letter + pentest summary + DPIA as interim evidence package. |
| Art. 9 zero-grace-period | §12.2 health data deletion zero grace period is per DEC-036. Cannot be modified by contract. Confirm customer-success has briefed IT contact on offboarding flow. |
| Redline limits | Standard-term self-service redline ≤ ±10% on commercial terms only. Any redline touching §8 (Privacy Floor), §9 (AUP), §5.3 (AI training), or §12.2 (health data deletion) requires compliance-officer approval regardless of deal size. |
| EU DPA Annex B | **Addendum 3 is required for all EU-region Customers (`eu-central-1` / `eu-west-1`).** Outside counsel review MANDATORY before execution (see Addendum 3.6). Confirm Cloudflare R2 EU jurisdiction annually (OFB-E-006). Do NOT sign EU enterprise DPA without Addendum 3 + counsel sign-off filed at `compliance/contracts/{CUSTOMER_SLUG}/eu-dpa-annex-b-counsel-signoff-v{N}.pdf`. |
| SIEM Addendum 4 | **Addendum 4 is required before any SIEM export is activated.** SIEM-CONSENT-01 chain invariant enforces this at the Worker layer (HTTP 422 rejection if no prior signed addendum). Outside counsel review required before first production execution (see §4.8). Do NOT activate SIEM export without Addendum 4 signed and `siem.consent_addendum_signed` DEC-030 event emitted. Evidence: SIEM-CONSENT-E-001 quarterly. Full spec: `docs/OBSERVABILITY.md §47` (DEC-065). |

---

*v0.4 · 2026-06-18 · owners: compliance-officer, enterprise-architect, founder · next review: before first EU enterprise DPA execution (outside counsel required per Addendum 3.6), before first SIEM export goes live (outside counsel required per §4.8), and before first enterprise multi-year contract (M10 per §11.4) · references: `docs/ENTERPRISE.md`, `docs/AUDIT_LOG_SCHEMA.md` (DEC-030), `docs/ENTERPRISE_SLA.md`, `docs/SUBPROCESSORS.md §5`, `docs/SOC2_READINESS.md §13`, `docs/COST_MODEL.md §35`, `docs/DATA_MODEL.md §36` (DEC-061), `docs/OBSERVABILITY.md §47` (DEC-065)*

*v0.4 (2026-06-18): Addendum 4 — SIEM Data Processing Addendum v1.0. Closes `docs/OBSERVABILITY.md §47.11` checklist item 5 (P1/M5 — "Author `docs/MSA_TEMPLATE.md §Addendum 4`"). Decision: DEC-065 (2026-06-18). Five-clause structure per §47.4: (1) Clause 1 — Scope and Activation (SIEM-CONSENT-01 chain invariant reference; Designated Endpoint designation; no-activation-before-Effective-Date commitment); (2) Clause 2 — Documented Instructions (GDPR Art. 28(3)(a) written-instruction compliance; instruction scope and duration; endpoint constraint); (3) Clause 3 — Customer Obligations (five sub-clauses: adequate legal basis; Designated Endpoint DPA coverage; no-reidentification warranty on pseudonymised fields; retention-policy compliance; breach notification to FORM); (4) Clause 4 — FORM Obligations (privacy floor warranty — no user_id/Art. 9/body composition/coaching content/plaintext endpoint URL in transmitted events; HMAC-chain integrity fields; 72h breach notification; DEC-030 chain record of acceptance and revocation); (5) Clause 5 — Revocation (Admin Dashboard "Revoke SIEM Addendum" action; 5-minute effect; DEC-030 `siem.consent_addendum_revoked` HIGH 7yr record; re-acceptance protocol). §4.6 SOC 2 evidence: SIEM-CONSENT-E-001 (CC9.2/CC1.1 — quarterly DEC-030 export; cross-check query returns zero); privacy floor on artefact (SHA-256[:32] endpoint_url_hash; SHA-256 signed_by_email_hash; no individual health data). §4.7 Governing law (Delaware/Ireland per MSA; Addendum 4 prevails on SIEM data flow activities). §4.8 Outside counsel review requirement (five review points: Art. 28(3)(a) documented-instruction sufficiency; customer warranty vs. evidence-of-basis; 72h breach notification alignment; revocation-as-instruction-withdrawal; DocuSign electronic acceptance validity; filing path `compliance/contracts/msa-addendum-4-v1.0-counsel-signoff.pdf`). Order Form updated: SIEM Addendum checkbox added; Exhibits incorporated list updated with Addendum 4. Internal Notes: SIEM Addendum 4 guidance row added. Privacy floor: no individual employee user_id, health data, Art. 9 special category, body composition, or coaching content in any Audit Event transmitted under this Addendum; endpoint_url_hash SHA-256[:32]; signed_by_email_hash SHA-256(lowercase(email)); tenant_id org slug only; form_api REVOKED from tenant_siem_configs. Cross-references: `docs/OBSERVABILITY.md §47` (DEC-065 — SIEM consent architecture, SIEM-CONSENT-01 invariant, Admin Dashboard gate, DEC-030 events, SIEM-CONSENT-E-001 artefact); `docs/OBSERVABILITY.md §47.11 item 5` (checklist obligation — now [x] Done); `docs/AUDIT_LOG_SCHEMA.md §SIEM` (siem.consent_addendum_signed HIGH 7yr + siem.consent_addendum_revoked HIGH 7yr — to be registered P0/M5); `docs/DATA_MODEL.md §SIEM` (tenant_siem_configs addendum_signed_at + addendum_version columns — migration 0076, P0/M5); `docs/SOC2_READINESS.md §88` (SIEM-CONSENT-E-001 SOC 2 evidence registration); `docs/ENTERPRISE.md §8` (Privacy Floor — non-negotiable; applies to all Audit Events transmitted under this Addendum); `docs/DECISION_LOG.md DEC-065` (decision rationale). Owner: compliance-officer + enterprise-architect + legal.*

*v0.3 (2026-06-15): Addendum 3 — EU Data Residency Terms (DPA Annex B). Closes `docs/DATA_MODEL.md §36.9` checklist item 6 (P0, before first EU enterprise DPA — "Update DPA Annex B template transfer mechanism row"). (1) Header corrected v0.1 → v0.3 (v0.2 note was present in footer but header line not updated at time of v0.2 commit). (2) Exhibit C SCC bullet made conditional: SCC Module 2 applies to active-service processing; for EU-region Customers (`data_region IN ('eu-central-1', 'eu-west-1')`), off-boarding export packages stored in Cloudflare R2 EU — no Chapter V GDPR transfer mechanism required for that activity; reference to Addendum 3 added. (3) Addendum 3 — EU Data Residency Terms: §3.1 Applicability (eu-central-1 / eu-west-1 Order Form regions); §3.2 EU Data Residency Commitment — off-boarding packages to `form-offboarding-exports-eu` (OFB-REGION-01 chain invariant enforcement; HTTP 422 on violation; R-05 escalation), Art. 9 health data structural prohibition (egress_package_type_enum), HMAC audit log EU storage; §3.3 DPA Annex B four-row transfer mechanism table — (a) off-boarding packages: None required (EU R2, intra-EEA), (b) active service processing: SCC Module 2 (Cloudflare Ireland Ltd sub-processor with EU addendum), (c) PostHog EU Cloud analytics: None required (k ≥ 5 floor), (d) Sentry EU: SCC Module 2 (technical metadata only); outside counsel note on R2 EU jurisdiction dependency; (4) §3.4 Evidence commitments: OFB-E-005 (quarterly chain export confirming OFB-REGION-01; C1.1/P4.0/CC6.1) + OFB-E-006 (annual R2 EU jurisdiction screenshot; C1.1/CC6.1); both from `docs/DATA_MODEL.md §36.7` (DEC-061). (5) §3.5 Governing law: Ireland for EU Customers; Addendum 3 prevails on §3.3 activities. (6) §3.6 Outside counsel requirement: five review points (R2 EU Chapter V declaration, TIA, SCC Module 2 coverage, k ≥ 5 EDPB, governing law enforceability); filing path `compliance/contracts/{CUSTOMER_SLUG}/eu-dpa-annex-b-counsel-signoff-v{N}.pdf`. (7) Internal Notes: EU DPA Annex B row added (Addendum 3 mandatory for EU Customers, outside counsel required, OFB-E-006 annual confirmation). Privacy floor: no individual employee `user_id`, health data, or Art. 9 category in any artefact specified by this Addendum; `tenant_id` slug only in OFB-E-005; OFB-E-006 is infrastructure screenshot with no personal data. Cross-references: `docs/DATA_MODEL.md §36.6` (OFB-REGION-01 spec); `docs/DATA_MODEL.md §36.7` (OFB-E-005/006 source); `docs/DATA_MODEL.md §36.9` (checklist item 6 — now [x] Done); `docs/SOC2_READINESS.md §67.8` (OFB-E-005/006 evidence table); `docs/SOC2_READINESS.md §67.9` (CC6.1/C1.1 EU residency criteria); `docs/INCIDENT_RESPONSE.md R-05` (OFB-REGION-01 violation response); `docs/GDPR_DPIA.md §10.3` (k-anonymity open question); `docs/SUBPROCESSORS.md §5` (DPA template reference); `docs/DECISION_LOG.md DEC-061`. Owner: compliance-officer + enterprise-architect + legal.*

*v0.2 · 2026-06-12 · owners: compliance-officer, enterprise-architect, founder · next review: before first enterprise multi-year contract execution (M10 — outside counsel review required per §11.4 §8.7) · references: `docs/ENTERPRISE.md`, `docs/AUDIT_LOG_SCHEMA.md` (DEC-030), `docs/ENTERPRISE_SLA.md`, `docs/SUBPROCESSORS.md §5`, `docs/SOC2_READINESS.md §13`, `docs/COST_MODEL.md §35`*

*v0.2 (2026-06-12): +§11.4 Early Termination Fee clause — closes COST_MODEL.md §35.10 checklist item 2 (P0, M10 — add ETF clause to MSA before first enterprise multi-year contract close). Added: (1) §11.4.8.1 liquidated damages framing (ETF is a genuine pre-estimate of FORM losses, not a penalty); (2) §11.4.8.2 ETF calculated on contracted ACV post multi-year discount, not list price; (3) §11.4.8.3 declining balance rate schedule (M1–M3: 60%, M4–M6: 50%, M7–M12: 40%, M13–M18: 30%, M19–M24: 20%, M25–M30: 10%, M31–M36: 0% + floor, M37+: resets); (4) §11.4.8.4 minimum floor of one month contracted ACV; (5) §11.4.8.5 seat reduction policy — reductions ≤ 10% deferred to annual date at no ETF; reductions > 10% or below tier minimum trigger ETF on ACV reduction amount; (6) §11.4.8.6 waiver authority — founder approval for waivers > USD 5,000; founder + lead investor for waivers > USD 100,000; (7) §11.4.8.7 outside counsel review checkpoint — M10 required before first multi-year contract (DE §§339–340 BGB enforceability + US NY/DE/CA considerations; OQ-ETF-01/OQ-ETF-02 from COST_MODEL.md); (8) §11.4.8.8 audit record — ETF waivers and contract amendments logged as DEC-030 HMAC-chained events per AUDIT_LOG_SCHEMA.md. §11.3 updated to reference §11.4. Former §11.4 (Effect of Termination) renumbered to §11.5. Cross-ref: COST_MODEL.md §35.3 (rate schedule), §35.5.5 (approval authority matrix), §35.9 (DEC-030 event definitions), §35.10 (P0 M10 checklist); AUDIT_LOG_SCHEMA.md §enterprise-contract-amendment-etf (enterprise.mid_contract_termination_risk_flagged, enterprise.contract_amended, enterprise.early_termination_fee_waived). Owner: compliance-officer, enterprise-architect, founder.*

*v0.1 · 2026-06-09 · owners: compliance-officer, enterprise-architect, founder · initial template draft · next review: before first enterprise contract execution · references: `docs/ENTERPRISE.md`, `docs/AUDIT_LOG_SCHEMA.md` (DEC-030), `docs/ENTERPRISE_SLA.md`, `docs/SUBPROCESSORS.md §5`, `docs/SOC2_READINESS.md §13`*
