---
version: 0.1.0-draft
status: pre-legal-review
effective_date: TBD (pre-launch — do not publish until counsel review complete)
last_reviewed: 2026-05-17
owner: compliance-officer
counsel_review_required: true
jurisdictions: EU (GDPR), UA (Закон про захист персональних даних), US (CCPA/CPRA)
soc2_evidence: CC2.2, CC2.3, P1.1, P1.2, P2.1, P3.1, P4.1, P4.2, P5.1, P6.1, P6.2, P8.1
---

# FORM Privacy Policy

**Version:** 0.1.0-draft  
**Status:** Pre-legal-review. Not yet effective. Do not publish.  
**Draft date:** 2026-05-17  
**Effective date:** To be confirmed on publication.  
**Controller:** [LEGAL ENTITY NAME — to be confirmed] ("FORM", "we", "us", "our")  
**Privacy contact:** privacy@form.coach

<!-- SOC2: CC2.3, P1.1 — Privacy policy document existence and accessibility at point of collection -->

---

## Table of Contents

1. [Who We Are](#1-who-we-are)
2. [Scope of This Policy](#2-scope-of-this-policy)
3. [Data We Collect](#3-data-we-collect)
4. [Legal Basis for Processing](#4-legal-basis-for-processing)
5. [How We Use Your Data](#5-how-we-use-your-data)
6. [Data Sharing and Sub-Processors](#6-data-sharing-and-sub-processors)
7. [Data Retention](#7-data-retention)
8. [Your Rights](#8-your-rights)
9. [Data Subject Access Requests (DSAR)](#9-data-subject-access-requests-dsar)
10. [Children and Age Restrictions](#10-children-and-age-restrictions)
11. [International Data Transfers](#11-international-data-transfers)
12. [Analytics and Tracking](#12-analytics-and-tracking)
13. [Changes to This Policy](#13-changes-to-this-policy)
14. [Contact and Supervisory Authority](#14-contact-and-supervisory-authority)

---

## 1. Who We Are

<!-- SOC2: CC2.3, P1.1 — Controller identity and contact information disclosed -->

FORM is an AI-powered strength coaching application for iOS and Android. Our service is operated by [LEGAL ENTITY NAME — to be confirmed], a company incorporated under [JURISDICTION — to be confirmed] ("the Controller").

**Registered address:** [REGISTERED ADDRESS — to be confirmed]  
**Privacy contact:** privacy@form.coach  
**Security contact:** security@form.coach  
**Data Protection Officer (EU):** [DPO NAME or CONSULTANCY — to be confirmed; required if scale triggers GDPR Art. 37]

FORM acts as a **data controller** for the personal data you provide to us when using the FORM application and related services. Where we engage third-party service providers to process data on our behalf, those providers act as **data processors** (also called sub-processors). A complete list of our sub-processors is set out in Section 6.

**Enterprise deployments:** If you access FORM through an employer-sponsored enterprise programme, your employer acts as a separate controller for the purposes of account provisioning. FORM acts as a controller for all health and coaching data you generate within the application. Your employer has contractually agreed that they will never access your individual health data. See Section 5.3 for the enterprise privacy floor.

---

## 2. Scope of This Policy

<!-- SOC2: P1.1, P1.2 — Policy covers all data categories at point of collection -->

This Privacy Policy applies to:

- The FORM iOS and Android applications
- The FORM web application at form.coach (if applicable)
- Any related services, APIs, or integrations operated by FORM

This Policy does not apply to:

- Third-party applications or services that you choose to connect to FORM (such as Apple Health or Google Health Connect), which are governed by their own privacy policies
- FORM's enterprise customer organisations, which act as separate controllers for their internal HR and identity management data
- Payment processing, which is handled entirely by Apple App Store and Google Play Store on our behalf; FORM never receives, stores, or processes your payment card data

If you have questions about the scope of this Policy, contact privacy@form.coach.

---

## 3. Data We Collect

<!-- SOC2: P1.2, P3.1 — Complete disclosure of data categories collected -->
<!-- GDPR Art. 13(1)(d), 14(1)(d) — Categories of personal data described -->
<!-- Apple App Store requirement: explicit health data disclosure -->

We collect the following categories of personal data. We indicate where collection is mandatory (required to use the service), optional (you choose to provide it), or automatic (collected by operation of the software).

### 3.1 Account and Identity Data

| Data element | How collected | Mandatory / Optional |
|---|---|---|
| Email address | Apple Sign In or direct registration | Mandatory |
| Display name | You provide at onboarding | Mandatory |
| Authentication token (Apple User ID) | Apple Sign In (opaque identifier, not your Apple ID email) | Mandatory |
| Account creation date and time | Automatic | Mandatory |
| Device type, operating system version | Automatic | Mandatory |

**Purpose:** To create and maintain your account and to enable you to authenticate securely to the service.

**Data classification:** Confidential (Tier 2).

### 3.2 Health Profile Data

**IMPORTANT NOTICE FOR APPLE APP STORE REVIEW:** The following data elements constitute health and fitness data under Apple's App Store Review Guidelines. This data is collected for the sole purpose of personalising your AI coaching experience. It is never used for advertising, sold to third parties, or shared with employers in identifiable form.

| Data element | How collected | Mandatory / Optional |
|---|---|---|
| Date of birth (for age calculation) | You provide at onboarding | Mandatory |
| Biological sex (for exercise programming) | You provide at onboarding | Optional |
| Height | You provide at onboarding | Optional |
| Weight (current and historical, if you log it) | You provide; user-controlled | Optional |
| Body composition goal (e.g., strength, fat loss) | You provide at onboarding | Optional |
| Resting heart rate | You provide or synced from wearable | Optional |
| Heart rate variability (HRV) baseline | Synced from wearable via HealthKit / Health Connect | Optional |
| Injuries or physical restrictions | You provide at onboarding | Optional |

**Legal basis:** Explicit consent (GDPR Art. 9(2)(a)). You will be asked to provide separate, granular consent for each health data category during onboarding. You may withdraw consent at any time in Settings → Privacy.

**Data classification:** Restricted (Tier 3 — GDPR Art. 9 special category).

### 3.3 Workout Data

| Data element | How collected | Mandatory / Optional |
|---|---|---|
| Workout type, date, duration | You initiate a workout session | Mandatory (when using workout features) |
| Exercises performed (name, sets, reps, weight) | You log or the app detects | Generated by use |
| Perceived exertion (RPE) | You provide | Optional |
| Movement quality score ("form score", 0–100) | Computed on-device from camera analysis | Generated by use |
| Rep count | Computed on-device | Generated by use |
| Workout programme phase and training block | Assigned by Victor AI coach | Generated by use |

**Critical notice about camera data:** Your camera is used exclusively for on-device movement analysis. **Raw camera frames and video are never uploaded to FORM's servers or any third party.** Movement analysis is performed entirely on your device using Apple Neural Engine or Android Neural Processing Unit. Only the derived form_score (a scalar value 0–100) and rep_count are transmitted to our servers. This is an architectural commitment, not merely a policy statement — no upload API exists for raw frames.

**Legal basis:** Contract performance (GDPR Art. 6(1)(b)) for core workout logging; explicit consent (GDPR Art. 9(2)(a)) for health inferences derived from movement analysis.

**Data classification:** Restricted (Tier 3 — GDPR Art. 9 special category).

### 3.4 Food and Nutrition Logs

| Data element | How collected | Mandatory / Optional |
|---|---|---|
| Food items logged | You enter via text or photo | Optional — requires explicit opt-in |
| Macronutrient breakdown (calories, protein, carbohydrates, fat) | Computed from food log | Optional — only if food log enabled |
| Meal type (breakfast, lunch, dinner, snack) | You provide | Optional |
| Log timestamp | Automatic when you create a log entry | Only if food log enabled |

**Legal basis:** Explicit consent (GDPR Art. 9(2)(a)). Food and nutrition data is treated as health-adjacent special category data. You must separately opt in to the food logging feature. Nutrition data is **never** included in any enterprise dashboard or aggregate report.

**Data classification:** Restricted (Tier 3 — GDPR Art. 9 special category).

### 3.5 Wearable and Biometric Data

| Data element | How collected | Mandatory / Optional |
|---|---|---|
| Heart rate variability (HRV) readings | Synced from Apple HealthKit or Google Health Connect with your explicit permission | Optional — requires explicit per-device, per-data-type opt-in |
| Resting heart rate (daily) | Synced from HealthKit / Health Connect | Optional |
| Step count | Synced from HealthKit / Health Connect | Optional |
| Sleep stages (light, deep, REM, awake) | Synced from HealthKit / Health Connect | Optional |

**Legal basis:** Explicit consent (GDPR Art. 9(2)(a)). Wearable data integration requires your explicit permission on both FORM and the wearable platform (HealthKit or Health Connect). You may revoke wearable data access at any time in your device's Health settings, independently of FORM.

**Wearable data remains on-device for initial processing.** FORM reads summary values through the HealthKit and Health Connect APIs. Raw sensor data from your wearable device is processed by those platforms per their own privacy policies, which are independent of this Policy.

**Data classification:** Restricted (Tier 3 — GDPR Art. 9 special category).

### 3.6 AI Coaching Conversation History

| Data element | How collected | Mandatory / Optional |
|---|---|---|
| Messages you send to Victor (your AI coach) | You type or speak | Optional — coaching feature |
| Victor's responses to you | Generated by AI system | Generated by use |
| Coaching session metadata (timestamp, session duration) | Automatic | Generated by use when you use coaching |

**How AI coaching works:** When you send a message to Victor, FORM assembles a coaching context and sends it to our AI inference provider (Anthropic). **The context sent to Anthropic contains only pseudonymised exercise data — your sets, reps, RPE, programme phase, and equipment. Your name, email address, age, weight, body metrics, and all other personal data are excluded from prompts.** The Anthropic Data Processing Agreement prohibits use of FORM's data to train Anthropic's models.

**Legal basis:** Contract performance (GDPR Art. 6(1)(b)). The coaching context sent to Anthropic does not contain personal data as defined under GDPR (it contains only pseudonymised exercise context). Your stored conversation history on FORM's servers is processed under contract performance and is subject to your right of erasure under GDPR Art. 17.

**Data classification:** Restricted (Tier 3) for stored coaching content; pseudonymised context transmitted to Anthropic is not personal data by design.

### 3.7 Eating Disorder Screening

At onboarding, FORM conducts a brief, non-diagnostic eating disorder awareness screen (based on a simplified SCOFF framework). This screen is conducted for your safety and cannot be skipped.

| Data element | How collected | Stored |
|---|---|---|
| Your responses to three wellbeing questions | In-app onboarding screen | Not stored as diagnostic data |
| Safety flag (binary UX control value only) | Computed from responses | Stored as a UX flag only |

**This is not a clinical diagnosis.** Your responses are used only to calibrate FORM's coaching language to avoid content that may be harmful if you are experiencing disordered eating. The result is stored as a binary UX flag — it is never stored as a clinical record, never included in any report accessible to your employer, never shared with any third party including health providers, and never exported in identifiable form.

**Legal basis:** Explicit consent (GDPR Art. 9(2)(a)) plus FORM's legitimate interest in user safety (GDPR Art. 6(1)(f)).

**Data classification:** Restricted (Tier 3 — highest protection; clinical-safety veto applies to any feature accessing this data).

### 3.8 Device and Usage Data

| Data element | How collected | Mandatory / Optional |
|---|---|---|
| Device type and operating system version | Automatic | Automatic |
| App version | Automatic | Automatic |
| Anonymous event data (screen views, feature interactions) | Automatic — anonymised | Automatic; opt-out available |
| Crash reports and error logs (anonymised) | Automatic — PII scrubbed | Automatic |
| IP address (held in edge logs, not stored in user record) | Network-level | Transient |

**Legal basis:** Legitimate interest (GDPR Art. 6(1)(f)). Anonymous analytics and error monitoring are necessary to provide a reliable service. You may opt out of analytics at any time in Settings → Privacy. See Section 12 for full analytics disclosure.

**Data classification:** Internal (Tier 1) for anonymous analytics; Confidential (Tier 2) for audit log entries.

### 3.9 Payment Metadata

FORM does not process payment card data. All payments are handled by Apple App Store and Google Play Store under their respective terms and privacy policies.

FORM receives only:

| Data element | Source | Purpose |
|---|---|---|
| Subscription tier (free, pro, enterprise) | Apple / Google IAP receipt validation | To unlock features |
| Purchase date and expiry date | Apple / Google IAP receipt | To manage subscription state |
| Refund or cancellation status | Apple / Google IAP | To manage access |

**For enterprise customers billed directly (not through Apple/Google):** FORM uses Stripe Inc as a payment processor for enterprise invoicing. Stripe receives billing contact name, company name, and invoice information. FORM does not store full card numbers. Stripe is subject to a separate Data Processing Agreement and Standard Contractual Clauses. See Section 6 for the sub-processor table.

**Legal basis:** Contract performance (GDPR Art. 6(1)(b)).

**Data classification:** Confidential (Tier 2).

---

## 4. Legal Basis for Processing

<!-- SOC2: P2.1 — Consent and lawful basis documented -->
<!-- GDPR Art. 6, Art. 9 — Lawful basis for each processing activity -->
<!-- GDPR Art. 13(1)(c) — Purposes and legal basis disclosed -->

Under the General Data Protection Regulation (GDPR) and equivalent legislation, we are required to have a legal basis for each processing activity. The following table maps our processing activities to their legal basis.

### 4.1 Consumer tier

| Processing activity | Legal basis | Notes |
|---|---|---|
| Account creation and management | Art. 6(1)(b) — performance of contract | Necessary to provide the service you signed up for |
| Health profile (body metrics, goals, restrictions) | Art. 9(2)(a) — explicit consent | Granular consent per category at onboarding; withdrawable at any time |
| CV-based workout analysis (form score, rep count) | Art. 9(2)(a) + Art. 6(1)(b) | Health inference from movement requires Art. 9 consent; service delivery requires Art. 6(1)(b) |
| Food and nutrition logging | Art. 9(2)(a) — explicit consent | Health-adjacent data; feature requires separate explicit opt-in |
| Wearable data integration (HRV, heart rate, sleep) | Art. 9(2)(a) — explicit consent | Per-device, per-data-type consent; revocable independently |
| AI coaching conversations (Victor) | Art. 6(1)(b) — contract performance | Pseudonymised context only; no health data in AI prompts by design |
| Voice synthesis via ElevenLabs | Art. 6(1)(b) — contract performance | Coaching text cues only; no user personal data transmitted |
| Eating disorder awareness screen | Art. 9(2)(a) — explicit consent + Art. 6(1)(f) legitimate interest (user safety) | Result is UX flag only; not a clinical record |
| Product analytics (PostHog) | Art. 6(1)(f) — legitimate interest | Anonymous events; opt-out available; LIA completed (see GDPR DPIA §5.3) |
| Error monitoring and crash reporting (Sentry) | Art. 6(1)(f) — legitimate interest | PII scrubbed at SDK; necessary for service reliability |
| Audit logging | Art. 6(1)(c) — legal obligation | Required for SOC 2 / GDPR Art. 30 record-keeping obligations |
| Breach notification | Art. 6(1)(c) — legal obligation | GDPR Art. 33/34 requires us to hold sufficient data to notify you if your data is compromised |

### 4.2 Enterprise tier (additional bases)

| Processing activity | Legal basis | Notes |
|---|---|---|
| SSO/SCIM provisioning | Art. 6(1)(b) — contract (joint controller with employer) | Enterprise DPA governs employer-FORM relationship |
| Aggregate wellness reporting to HR dashboard | Art. 6(1)(f) — legitimate interest of enterprise customer | Strictly k-anonymised (groups < 5 users suppressed); individual data is never exposed |
| Audit log for tenant admin | Art. 6(1)(c) — legal obligation + Art. 6(1)(f) | Administrative access log only; no health data in admin-visible audit entries |

### 4.3 Consent management

When you provide consent for health data processing, FORM records the following in our tamper-evident audit log:

- The version of the Privacy Policy and consent form you agreed to
- The specific data categories you consented to
- The date and time of consent
- The mechanism through which consent was given (e.g., in-app onboarding screen)

You may **withdraw consent** at any time by:

1. Going to Settings → Privacy → Data Permissions in the FORM app
2. Toggling off any individual health data category
3. Emailing privacy@form.coach with a specific withdrawal request

Withdrawal of consent does not affect the lawfulness of processing carried out before withdrawal. If you withdraw consent for a category of health data, FORM will cease processing that category and will offer you the option to delete previously collected data in that category.

**Withdrawal of consent for core health data categories may impair the functionality of the FORM service**, since personalised coaching requires some health data to operate. We will communicate clearly which features are affected by each withdrawal.

---

## 5. How We Use Your Data

<!-- SOC2: P3.1 — Purpose limitation; data used only for identified purposes -->
<!-- GDPR Art. 5(1)(b) — Purpose limitation principle -->

### 5.1 Core coaching and personalisation

We use your workout data, health profile, wearable data, food logs (if enabled), and coaching conversation history to:

- Generate personalised workout programmes through your AI coach, Victor
- Analyse your movement quality and provide form feedback during workouts
- Track your training progress over time
- Adapt coaching intensity and volume based on your recovery signals (HRV, sleep, readiness)
- Provide nutritional guidance in alignment with your fitness goals (if food logging is enabled)
- Synthesise voice coaching cues through our text-to-speech integration

Your health data is used exclusively to provide you with a better coaching experience. We do not use your health data for advertising, profiling unrelated to fitness, or any purpose beyond those stated in this Policy.

### 5.2 Service reliability and safety

We use anonymous usage events and crash reports to:

- Identify and fix bugs and crashes
- Monitor service uptime and performance
- Detect unusual activity that may indicate a security issue
- Ensure the accuracy and reliability of coaching outputs

Health event content (specific meal items, form score values, coaching conversation content) is never included in analytics events. Analytics contain only anonymous, aggregated behavioural signals (screen views, feature interactions, session duration).

### 5.3 Enterprise deployments — privacy floor

If you access FORM through an employer-sponsored programme, the following privacy protections apply unconditionally. These are technical controls enforced at the database level, not merely policy commitments:

1. **Your employer never sees your individual health data.** No HR administrator, people manager, or company executive can access your individual workout records, body metrics, food logs, HRV data, coaching conversations, or eating disorder screening results.

2. **Aggregate wellness data shown to employers is k-anonymised.** Your employer's HR dashboard shows only aggregated summaries (e.g., number of active users, average workout frequency). Individual data is mathematically suppressed if a group contains fewer than five members. Body composition data, nutrition data, and eating disorder screening results are excluded from all employer-visible aggregates.

3. **You can disconnect from your employer at any time.** Settings → Privacy → "Disconnect from employer" immediately removes the association between your FORM account and your employer's account. Your personal health data remains yours.

4. **We do not share your data with your employer outside the k-anonymised dashboard.** No exceptions, regardless of customer requests. This constraint is enforced technically (database access controls and row-level security policies) and contractually (enterprise terms of service).

### 5.4 What we do not do

FORM does not:

- Sell your personal data or health data to any third party
- Share your data with insurance companies, healthcare providers, or government agencies except as required by law
- Use your data to train external AI models (Anthropic's enterprise Data Processing Agreement prohibits this)
- Use your health data for advertising targeting, on or off the FORM platform
- Share aggregate or de-identified wellness data with your employer that could enable individual re-identification
- Use your eating disorder screening results for any purpose other than calibrating coaching language

---

## 6. Data Sharing and Sub-Processors

<!-- SOC2: CC9.2, P6.1, P6.2 — Sub-processor disclosure; vendor management -->
<!-- GDPR Art. 13(1)(e), Art. 28 — Processor disclosure obligation -->
<!-- SOC2 critical gap closed: "Sub-processor disclosure" -->

FORM engages the following sub-processors to operate our service. Each sub-processor has signed a Data Processing Agreement (DPA) with FORM. For EU users, Standard Contractual Clauses (SCCs) Module 2 (controller-to-processor) are in place for any transfer of personal data to processors located outside the European Economic Area.

<!-- SOC2: CC9.2 — Sub-processor register with DPA status evidence -->

### 6.1 Sub-processor register

| Sub-Processor | Headquarters | Purpose | Personal / Health Data Involved | Region(s) | DPA Status | GDPR Transfer Mechanism |
|---|---|---|---|---|---|---|
| **Supabase Inc** | San Francisco, US | Managed PostgreSQL database, user authentication, file storage — primary data store for all user and tenant data | All personal data categories described in Section 3 | US (`us-east-1` default); EU (`eu-central-1`) available for enterprise tenants on request | Signed | SCC Module 2; EU data residency available eliminates transfer for EU tenants who select EU region |
| **Amazon Web Services (AWS)** | Seattle, US | Cloud infrastructure (compute, storage, backups) — accessed via Supabase; FORM does not have a direct AWS account separate from Supabase | All data (through Supabase) | Inherits Supabase region selection | Covered by Supabase DPA | Supabase SCCs cover downstream AWS processing |
| **Cloudflare Inc** | San Francisco, US | Content delivery network (CDN), DDoS protection, edge compute (Cloudflare Workers) — API gateway layer | IP addresses, request metadata; no user health data passes through Workers unencrypted | US + global edge network | Enterprise DPA signed | SCC Module 2 |
| **Anthropic PBC** | San Francisco, US | Large language model inference — powers Victor, your AI coaching assistant | Pseudonymised exercise context only: sets, reps, RPE, programme phase, equipment. Your name, email, age, health metrics, and all personal data are excluded from prompts by architectural design. | US | Signed | SCC Module 2 |
| **ElevenLabs Inc** | New York, US | Text-to-speech voice synthesis — converts Victor's text coaching cues into spoken audio | Coaching text cues only — no user personal data is included in text-to-speech payloads | US | Signed | SCC Module 2 |
| **PostHog Inc** | San Francisco, US | Product analytics — anonymous event tracking to understand how users interact with the app | Anonymous event data, opt-out flag, device type; no personally identifiable information and no health event content by configuration | US / EU (EU self-hosted available) | Signed | SCC Module 2; EU hosting available |
| **Functional Software Inc (Sentry)** | San Francisco, US | Error monitoring and crash reporting — captures anonymised crash reports for debugging | Error stack traces, anonymised session identifiers; health data is scrubbed from Sentry payloads at the SDK level before transmission | US | In progress — DPA review underway; PII scrubbing at SDK is compensating control pending formal DPA | SCC Module 2 pending DPA signature |
| **Apple Inc** / **Google LLC** | Cupertino, US / Mountain View, US | In-app purchase processing (IAP) — all consumer subscription billing | Subscription tier, purchase date, expiry status only; FORM never receives card data | US | Governed by Apple / Google developer agreements | Apple and Google developer programme terms; no health data transferred |
| **Stripe Inc** | San Francisco, US | Payment processing for enterprise direct billing (enterprise customers not billed through Apple / Google IAP) | Billing contact name, company name, invoice data; no health data | US / EU | Signed | SCC Module 2 |

### 6.2 Key data minimisation commitments for sub-processors

- **Anthropic receives no personally identifiable information.** Prompts to the Anthropic API contain only pseudonymised exercise context. Health values, user identifiers, names, and demographics are excluded at the prompt-assembly layer and validated in automated testing.

- **ElevenLabs receives no user data.** Text-to-speech requests contain only the coaching text cue — a coaching instruction in plain text with no reference to any user.

- **PostHog events contain no health data.** Events track anonymous behavioural signals (which screen was viewed, whether a feature was used). Specific health values (form score, meal content, HRV readings) are never included in analytics events.

- **Sentry payloads are scrubbed before transmission.** The Sentry SDK is configured to strip workout data, body metrics, and coaching context fields from crash reports before they leave your device. This is a compensating control while the formal Sentry DPA is being finalised.

### 6.3 New sub-processors

FORM will notify enterprise customers at least **30 days in advance** before engaging a new sub-processor that will process their tenant data. Enterprise customers have the right to object to a new sub-processor under GDPR Art. 28(2). If you object and FORM cannot accommodate the objection, we will work with you to find an alternative or allow contract termination without penalty.

Consumer users will be notified of material new sub-processors through an update to this Privacy Policy (see Section 13).

### 6.4 Government and law enforcement requests

FORM does not provide backdoor access to any government or law enforcement agency. We will only disclose user data in response to a lawful court order, subpoena, or equivalent compulsory legal process, and only to the extent required by that process. Where permitted by law, we will notify affected users before complying. FORM will not comply with any request that we believe is unlawful.

We publish government request statistics as a transparency measure [publication cadence to be confirmed].

---

## 7. Data Retention

<!-- SOC2: P4.1, P4.2 — Retention schedule defined; disposal procedures documented -->
<!-- GDPR Art. 5(1)(e) — Storage limitation principle -->

FORM retains your data for the minimum period necessary to fulfil the purpose for which it was collected, subject to any legal retention obligations.

### 7.1 Retention schedule

| Data category | Retention period | Basis |
|---|---|---|
| Account and identity data | Duration of account, then deleted within 30 days of account closure | Contract performance |
| Health profile (body metrics, goals, restrictions) | Duration of account; user can delete individual entries at any time; deleted within 30 days of account closure | Consent; user control |
| Workout logs and form scores | Duration of account; user can delete individual sessions at any time; deleted within 30 days of account closure | Contract performance; user control |
| Food and nutrition logs | Duration of account; user can delete individual logs at any time; deleted within 30 days of account closure | Consent; user control |
| Wearable data (HRV, heart rate, sleep) | Duration of account; user can revoke wearable access and delete synced data at any time; deleted within 30 days of account closure | Consent; user control |
| AI coaching conversation history | Duration of account; user can delete individual sessions; deleted within 30 days of account closure | Contract performance |
| Eating disorder screening flag | Duration of account; deleted within 30 days of account closure | Consent |
| Anonymous analytics events | Up to 24 months in PostHog; anonymised and non-recoverable to individual identity | Legitimate interest |
| Error monitoring data (Sentry) | 90 days in Sentry, then automatically purged | Legitimate interest |
| Audit log (HMAC-chained integrity log) | 7 years — retained for SOC 2 Type II audit evidence and legal compliance; entries are anonymised (user ID removed, action and timestamp retained) when a user account is deleted | Legal obligation (Art. 6(1)(c)) |
| Backup copies of primary data | Maximum 90 days in encrypted backup storage; deleted as part of the scheduled backup rotation cycle | Operational necessity |

### 7.2 Account deletion and right to erasure

When you delete your FORM account (Settings → Profile → Delete Account):

1. Your account is immediately marked for deletion and becomes inaccessible.
2. A 30-day grace period begins, during which you may restore your account by contacting privacy@form.coach.
3. After 30 days, all personal data in your user record is hard-deleted from our primary database.
4. Backup copies are purged within 60 days of the deletion date (the next backup rotation cycle after the 30-day grace period).
5. Audit log entries are anonymised: your user ID is removed and replaced with a pseudonymous token; the action type, timestamp, and system metadata are retained for audit integrity purposes.
6. You will receive a confirmation email when deletion is complete.

Total maximum retention after deletion request: **90 days** (30-day grace + 60-day backup purge). After this period, no data identifiable to you exists in FORM's systems or backups, except for anonymised audit log entries.

**Enterprise tenant data:** Upon contract termination, an equivalent process applies to the tenant's user data. A deletion certificate is issued on request.

### 7.3 Data you can delete yourself

You do not need to contact us to delete most of your data. Within the FORM app:

- **Individual workout sessions:** Delete from your training history in the app
- **Food log entries:** Delete individual entries within the food log
- **Wearable data sync:** Revoke HealthKit / Health Connect access and delete previously synced data
- **All your data:** Request full account deletion at Settings → Profile → Delete Account

---

## 8. Your Rights

<!-- SOC2: P5.1, P7.1 — Data subject rights access mechanism documented -->
<!-- GDPR Art. 15–22 — Rights of data subjects -->

Depending on your location, you have the following rights regarding your personal data. FORM respects these rights and has implemented mechanisms to fulfil them promptly.

### 8.1 Right of access (GDPR Art. 15)

You have the right to obtain confirmation of whether FORM processes personal data about you, and to receive a copy of that data. You can access and export all your personal data at any time through:

- **In-app export:** Settings → Privacy → Export My Data (JSON format, delivered as a secure download link within 48 hours)
- **Written request:** Email privacy@form.coach with subject "Data Access Request"

See Section 9 for the full DSAR procedure.

### 8.2 Right to rectification (GDPR Art. 16)

You have the right to correct inaccurate personal data. You can update most of your profile data directly in the app. For data you cannot update yourself, contact privacy@form.coach and we will correct it within 30 days.

### 8.3 Right to erasure / right to be forgotten (GDPR Art. 17)

You have the right to request deletion of your personal data. You can exercise this right by:

- **Deleting your account** in the app (Settings → Profile → Delete Account) — this triggers complete data erasure as described in Section 7.2
- **Deleting specific data categories** within the app (individual sessions, food logs, wearable data)
- **Emailing** privacy@form.coach to request erasure of specific data elements

**Exceptions:** The right to erasure does not apply to data we are required to retain by law (for example, anonymised audit log records required for SOC 2 compliance) or data necessary to establish, exercise, or defend legal claims.

### 8.4 Right to data portability (GDPR Art. 20)

You have the right to receive your personal data in a structured, commonly used, machine-readable format and to transmit it to another service. FORM provides:

- **JSON export** of all your personal data, available through Settings → Privacy → Export My Data
- The export includes: account information, workout history, health profile, food logs, wearable data, and coaching conversation history
- Export is delivered via a time-limited secure download link within 48 hours

### 8.5 Right to object (GDPR Art. 21)

You have the right to object to processing based on legitimate interest (GDPR Art. 6(1)(f)). Specifically:

- **Object to analytics:** Opt out in Settings → Privacy → Analytics. All PostHog event transmission stops immediately and permanently until you opt back in.
- **Object to error monitoring:** Contact privacy@form.coach. Note that disabling error monitoring may affect our ability to identify and fix bugs affecting your experience.

### 8.6 Right to withdraw consent (GDPR Art. 7(3))

Where we process your data on the basis of consent, you have the right to withdraw consent at any time. Withdrawal does not affect the lawfulness of prior processing. You can withdraw consent for any health data category in Settings → Privacy → Data Permissions.

### 8.7 Right not to be subject to automated decision-making (GDPR Art. 22)

FORM does not make decisions that have legal or similarly significant effects on you using solely automated means without human review. Your AI coaching programme is generated by the Victor AI, but it does not determine your employment, insurance, credit, or any other consequential matter. You may request human review of any AI coaching recommendation by contacting support@form.coach.

### 8.8 Right to restriction of processing (GDPR Art. 18)

You have the right to request that FORM restrict processing of your data (i.e., continue to store it but not use it) in specific circumstances, for example while a rectification request is being assessed. Contact privacy@form.coach to exercise this right.

### 8.9 Right to lodge a complaint (GDPR Art. 77)

You have the right to lodge a complaint with a data protection supervisory authority. FORM's lead supervisory authority is anticipated to be:

- **Ireland:** Data Protection Commission (dataprotection.ie), if Supabase EU (`eu-west-1`) hosting establishes an Irish nexus
- **Your local supervisory authority** — you may always lodge a complaint with the supervisory authority in the EU member state where you reside or work, or where an alleged infringement occurred

**Ukrainian users:** The Commissioner for the Protection of Personal Data (Уповноважений Верховної Ради України з прав людини) is the relevant supervisory authority under Ukraine's Law on Personal Data Protection.

**California users:** You have rights under the California Privacy Rights Act (CPRA) that are substantially equivalent to those described above. FORM does not sell or share your personal information as defined under CPRA. To exercise any CPRA right, contact privacy@form.coach.

We encourage you to contact us first at privacy@form.coach before lodging a complaint, so we can attempt to resolve the issue directly.

---

## 9. Data Subject Access Requests (DSAR)

<!-- SOC2: P5.1 — DSAR procedure documented and tested -->
<!-- GDPR Art. 12 — Response time and procedure for exercising rights -->
<!-- SOC2 critical gap closed: "DSAR workflow tested end-to-end < 30 days" -->

### 9.1 How to submit a DSAR

You may submit a DSAR by any of the following methods:

1. **In-app (preferred):** Settings → Privacy → Export My Data or Delete My Data — automated processing begins immediately
2. **Email:** Send a request to privacy@form.coach with subject line "DSAR Request — [your name]"
3. **Postal mail:** [REGISTERED ADDRESS — to be confirmed] — marked "DSAR"

For email or postal requests, include:

- Your full name and the email address associated with your FORM account
- The nature of your request (access, deletion, rectification, portability, etc.)
- If applicable, the specific data categories or time periods your request covers

### 9.2 Identity verification

To protect your data from unauthorised access, we may ask you to verify your identity before processing your request. For in-app requests, your active authenticated session constitutes verification. For email requests, we may ask you to confirm from your registered email address or answer a security question.

### 9.3 Response timeline

| Request type | Response time | Notes |
|---|---|---|
| Acknowledgement of DSAR receipt | Within 72 hours | Automated for in-app requests; manual for email/postal |
| Data export (access / portability) | Within 30 calendar days | Secure download link delivered to your registered email; available for 7 days |
| Account deletion confirmation | Within 30 calendar days | Includes confirmation that deletion is complete; a deletion certificate is available on request |
| Rectification | Within 30 calendar days | Confirmation of correction |
| Complex requests | Up to 90 calendar days | GDPR Art. 12(3) permits extension by 60 additional days for complex requests; you will be notified within the initial 30 days if an extension is needed |

**We commit to a 30-day SLA for all standard DSAR responses.** This SLA is tracked through our internal ticketing system. DSAR handling events are logged to our audit trail (`data.export_initiated`, `data.deletion_requested`) for SOC 2 evidence.

### 9.4 No charge

FORM does not charge a fee for DSARs. We may charge a reasonable administrative fee only if requests are manifestly unfounded or excessive, for example if the same person submits a large number of identical requests in a short period. We will inform you before any fee is applied.

---

## 10. Children and Age Restrictions

<!-- SOC2: P3.1 — Collection limitation — age requirement documented -->
<!-- GDPR Art. 8 — Conditions applicable to child's consent -->

**FORM is not designed for or directed at children under the age of 16.**

We do not knowingly collect personal data from anyone under the age of 16. By creating a FORM account, you confirm that you are at least 16 years old.

For users between 16 and 18 years of age, FORM provides age-appropriate coaching parameters (e.g., intensity limits, exercise selection constraints) and excludes certain features that may be inappropriate for developing physiology. These constraints are applied automatically based on the date of birth you provide at onboarding.

If you believe that a person under the age of 16 has provided personal data to FORM, please contact privacy@form.coach immediately. We will take steps to delete any such data as quickly as practicable.

**Note for enterprise deployments:** If your organisation intends to provide FORM access to employees or members under the age of 16, contact your FORM account manager before provisioning to discuss appropriate controls. FORM's standard enterprise configuration does not support under-16 access.

---

## 11. International Data Transfers

<!-- SOC2: P6.1 — Transfer mechanisms documented -->
<!-- GDPR Art. 44–49 — Lawful basis for international transfers -->

FORM is a global service with infrastructure primarily located in the United States, with EU data residency options available for enterprise customers.

### 11.1 Transfers to the United States

The majority of our sub-processors are headquartered in the United States (Anthropic, ElevenLabs, Cloudflare, PostHog, Sentry, Supabase, Stripe — see Section 6). When personal data is transferred from the European Economic Area (EEA), the United Kingdom, or Ukraine to these US-based processors, we rely on:

**Standard Contractual Clauses (SCCs)** — specifically SCC Module 2 (controller-to-processor), as adopted by the European Commission. SCCs are in place with Anthropic, ElevenLabs, Cloudflare, Supabase, PostHog, and Stripe. Sentry SCCs are being finalised in parallel with the DPA.

The SCCs are available on request by emailing privacy@form.coach.

### 11.2 EU data residency option

Enterprise customers may elect to store their data in the European Union (AWS `eu-central-1` — Frankfurt, or `eu-west-1` — Ireland). With EU data residency selected:

- All database data is stored and processed within the EU
- Database backups remain within the EU
- Supabase EU and PostHog EU instances are used, eliminating the need for SCC-covered transfers for those processors
- Anthropic and Cloudflare remain US-based (SCCs in place); their processing of pseudonymised exercise context and request metadata, respectively, continues to be covered by SCCs

EU data residency is contractually bound at tenant creation and cannot be changed post-deployment without a full data migration engagement.

### 11.3 UK transfers

Following the UK's departure from the EU, transfers from the UK to the EU and EEA are permitted under the UK GDPR framework. Transfers from the UK to US processors are covered by the UK International Data Transfer Agreement (IDTA), which FORM uses in conjunction with SCC-equivalent addenda with our US sub-processors.

### 11.4 Ukrainian users

Transfers of personal data from Ukraine to processors outside Ukraine are subject to the Law of Ukraine on Personal Data Protection (Закон України «Про захист персональних даних»). FORM relies on contractual safeguards (DPAs and SCCs) with all processors receiving data from Ukrainian users. The Ukrainian data protection framework is substantively equivalent to the GDPR framework applied elsewhere in this Policy.

### 11.5 No onward transfers without equivalent protection

FORM's sub-processors are not permitted to transfer your data to any fourth party without FORM's prior written approval and the implementation of equivalent transfer safeguards. This is a contractual obligation in each sub-processor DPA.

---

## 12. Analytics and Tracking

<!-- SOC2: P2.1 — Choice and consent for analytics; opt-out mechanism documented -->
<!-- GDPR Art. 6(1)(f) — Legitimate interest basis for analytics; balancing test completed -->

### 12.1 PostHog product analytics

FORM uses PostHog Inc to collect anonymous product analytics. Analytics help us understand which features are used, how users navigate the app, and where we can improve.

**What PostHog collects:**
- Screen views and navigation events
- Feature interaction events (e.g., "started a workout", "opened coaching")
- Session duration
- Device type and operating system version (not linked to your identity)
- Whether analytics opt-out is active

**What PostHog does not collect:**
- Your name, email address, or any personally identifiable information
- Health data values (no form scores, body metrics, HRV readings, food log contents in events)
- Coaching conversation content
- Raw IP address (PostHog is configured to truncate IP addresses)

**Legal basis:** Legitimate interest (GDPR Art. 6(1)(f)). A Legitimate Interest Assessment (LIA) has been completed and is documented in our GDPR Data Protection Impact Assessment. The LIA concluded that: (a) product analytics is a legitimate business interest; (b) anonymous event data is the minimum necessary; and (c) the user's right to opt out is preserved and easy to exercise.

**Opt out:** You can opt out of analytics at any time. Go to Settings → Privacy → Analytics and toggle the switch to "Off". When you opt out:
- All PostHog event transmission stops immediately
- The opt-out preference is stored in your account and persists across devices and reinstallations
- Your opt-out preference is logged to our audit trail (`privacy.analytics_opt_out`)
- No events collected while opted out will be retroactively included in analytics

### 12.2 Sentry error monitoring

FORM uses Sentry (Functional Software Inc) for crash reporting and error monitoring. Sentry receives anonymised crash reports to help us identify and fix bugs.

**What Sentry collects:**
- Error type and stack trace
- App version
- Device type and OS version
- Anonymised session identifier (not linked to your identity)

**What Sentry does not collect (scrubbed at SDK level):**
- Health data fields (workout_data, body_metrics, coaching_context)
- User email, display name, or other PII
- Raw camera or audio data

**Legal basis:** Legitimate interest (GDPR Art. 6(1)(f)). Error monitoring is necessary to provide a reliable service.

**Note on Sentry DPA:** A formal Data Processing Agreement with Sentry is currently being finalised. PII scrubbing at the SDK level is a compensating control in place in the interim. Once the DPA is signed and SCCs are executed, this notice will be updated.

### 12.3 No cross-site tracking or advertising trackers

FORM does not use advertising tracking pixels, Google Analytics, Facebook Pixel, or any similar cross-site tracking technology. We do not participate in any advertising network data sharing. We do not sell, share, or rent your data to any advertising or data broker platform.

### 12.4 Apple and Google platform data

Apple and Google collect their own analytics from your device when you use any iOS or Android application. These analytics are governed by Apple's and Google's own privacy policies. FORM has no control over and no visibility into this platform-level data.

---

## 13. Changes to This Policy

<!-- SOC2: CC2.3 — Communication of policy changes to users -->
<!-- GDPR Art. 13(2)(b) — Notification of changes -->

### 13.1 How we notify you of changes

We will notify you of material changes to this Privacy Policy at least **30 days before** the changes take effect, through one or more of the following methods:

- An in-app notification displayed when you open FORM
- An email to your registered email address
- A prominent notice on form.coach/privacy

For changes that expand our use of your health data or that introduce new sub-processors who receive health data, we will ask for your renewed consent before the change takes effect.

For non-material changes (such as clarifications, corrections, or changes to contact details), we may update this Policy without advance notice, but the change date will be reflected in the "last_reviewed" field at the top of this document.

### 13.2 Version history

| Version | Date | Summary |
|---|---|---|
| 0.1.0-draft | 2026-05-17 | Initial draft — pre-legal-review, pre-launch. Covers all data categories. Not yet effective. |

### 13.3 Effective version for your agreement

The version of this Privacy Policy that was in effect at the time you created your FORM account, or at the time you provided consent to specific data processing, governs FORM's obligations to you for data collected under that version. If material changes require new consent, you will be asked to review and accept the new version before continued use.

---

## 14. Contact and Supervisory Authority

<!-- SOC2: P8.1 — Contact mechanism for privacy inquiries documented -->
<!-- GDPR Art. 13(1)(a), 77 — Controller contact and supervisory authority details -->

### 14.1 Privacy contact

For all privacy-related inquiries, rights requests, DSAR submissions, or concerns:

**Email:** privacy@form.coach  
**Subject line for DSARs:** "DSAR Request — [your name]"  
**Subject line for complaints:** "Privacy Complaint — [brief description]"  
**Response time:** We aim to acknowledge all privacy inquiries within 72 hours and to resolve them within 30 calendar days.

### 14.2 Security contact

For security vulnerabilities or to report a potential breach:

**Email:** security@form.coach  
**Responsible disclosure:** We support responsible disclosure. We ask that you allow us reasonable time to investigate and remediate before public disclosure.

### 14.3 Postal contact

[LEGAL ENTITY NAME — to be confirmed]  
[REGISTERED ADDRESS — to be confirmed]  
Attn: Privacy Officer

### 14.4 Data Protection Officer

[If required by GDPR Art. 37 based on scale of health data processing:]  
**DPO contact:** [DPO NAME or CONSULTANCY — to be confirmed]  
**DPO email:** dpo@form.coach [to be confirmed]

### 14.5 Supervisory authorities

You have the right to lodge a complaint with a data protection supervisory authority at any time.

**EU lead supervisory authority (anticipated):**  
Data Protection Commission (Ireland)  
21 Fitzwilliam Square South  
Dublin 2, D02 RD28  
Ireland  
www.dataprotection.ie

**Your local EU supervisory authority:** You may also lodge a complaint in your country of residence. A list of EU supervisory authorities is available at edpb.europa.eu.

**Ukraine:**  
Commissioner for the Protection of Personal Data  
Уповноважений Верховної Ради України з прав людини  
vru.gov.ua

**United Kingdom:**  
Information Commissioner's Office (ICO)  
ico.org.uk

**California (US):**  
California Privacy Protection Agency  
cppa.ca.gov

---

## Appendix A — SOC 2 Control Mapping

*This appendix is intended for use by FORM's SOC 2 auditor and enterprise customers conducting security due diligence. It maps Privacy Policy sections to AICPA Trust Service Criteria.*

<!-- SOC2: This appendix closes the audit evidence requirement for P1.1 "privacy notice exists and covers all categories" -->

| SOC 2 Criterion | Description | Section in this Policy |
|---|---|---|
| **P1.1** | Privacy notice exists and is accessible at point of collection | Sections 1, 2, 3 — all data categories disclosed |
| **P1.2** | Privacy notice covers all categories of personal information collected | Section 3 — complete data category inventory (3.1–3.9) |
| **P2.1** | Consent collected before processing; withdrawal mechanism exists | Section 4.3, Section 8.6, Section 12.1 |
| **P3.1** | Personal information collected only for identified purposes | Section 5 — purpose limitation; Section 3 — stated purpose per category |
| **P4.1** | Retention schedule defined | Section 7.1 — retention schedule table |
| **P4.2** | Disposal procedures documented | Section 7.2 — account deletion and erasure procedure |
| **P5.1** | Mechanism for data subject access | Section 8.1, Section 9 — DSAR procedure with 30-day SLA |
| **P6.1** | Sub-processor list published; transfer mechanisms disclosed | Section 6.1 — sub-processor register with DPA status |
| **P6.2** | Notice of new sub-processors | Section 6.3 — 30-day advance notice procedure |
| **P7.1** | Mechanism for users to correct data | Section 8.2 — rectification right |
| **P8.1** | Contact for privacy complaints | Section 14.1 — privacy@form.coach |
| **CC2.2** | Internal communication of privacy commitments | This document; referenced in docs/SOC2_READINESS.md |
| **CC2.3** | External communication of privacy policy | This document — intended for publication at form.coach/privacy |
| **CC9.2** | Vendor management — sub-processor disclosure | Section 6.1 — sub-processor register |

---

## Appendix B — Apple App Store Health Data Disclosure Summary

*This appendix summarises health data handling for Apple App Store review purposes, in addition to the full disclosures above.*

FORM collects and processes the following health and fitness data categories:

| Apple data type | FORM usage | Shared with third parties? |
|---|---|---|
| Body measurements (height, weight) | Coaching personalisation | No — not shared in identifiable form |
| Fitness (workout logs, form scores) | Core product feature | No — pseudonymised context only to Anthropic (no health values) |
| Heart rate | Recovery-based coaching | No |
| Heart rate variability | Readiness calibration | No |
| Sleep analysis | Recovery recommendations | No |
| Nutrition (food logs) | Nutritional coaching (if enabled) | No |

- Health data is never used for advertising.
- Health data is never sold to third parties.
- Camera data (used for movement analysis) never leaves the device.
- Individual health data is never accessible to employers in enterprise deployments.
- Users can delete all health data at any time within the app.

---

*End of Privacy Policy*

*FORM Privacy Policy v0.1.0-draft — 2026-05-17 — pre-legal-review*  
*Owner: compliance-officer*  
*Next action: legal counsel review (EU, US, UA jurisdictions) before publication*
