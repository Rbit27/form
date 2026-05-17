# FORM · SOC 2 Type II Readiness v0.1

> Внутрішній roadmap до SOC 2 Type II certification.
> Власник: `compliance-officer` + `security-engineer`. Review: quarterly.
> Target: audit firm engaged Month 3 after enterprise launch; Type II report issued Month 12.
> Reference: DEC-030 (HMAC-chained audit log), docs/AUDIT_LOG_SCHEMA.md, docs/ENTERPRISE.md.

---

## TL;DR

SOC 2 Type II means an independent auditor tested our controls over a **minimum 6-month observation period** and found them operating effectively. It is a commercial prerequisite for enterprise deals >$50k ARR and mandatory for customers in regulated industries (FinServ, HealthTech, LegalTech).

Target timeline:
- **Month 0–3:** Gap remediation (this document)
- **Month 3:** Readiness assessment with audit firm
- **Month 3–9:** Observation period begins (controls operating)
- **Month 9–12:** Fieldwork, evidence collection, report issuance

---

## Trust Service Criteria (TSC) Coverage

AICPA defines five TSC. We pursue **all five** — Security is mandatory; the rest signal maturity to enterprise buyers.

| TSC | In Scope | Why |
|---|---|---|
| CC — Security | ✅ Required | Mandatory for any SOC 2 |
| A — Availability | ✅ Yes | 99.9% SLA в ENTERPRISE.md; enterprise buyers need uptime proof |
| PI — Processing Integrity | ✅ Yes | CV-coaching and nutrition outputs must be accurate and complete |
| C — Confidentiality | ✅ Yes | Health data, trade secrets, B2B customer data |
| P — Privacy | ✅ Yes | GDPR overlap; health data as GDPR Art. 9 special category |

---

## 1 · Security (CC Series)

### CC1 — Control Environment

**What SOC 2 requires:** Tone at the top, org structure, accountability, HR security practices.

| Control | Status | Evidence |
|---|---|---|
| Security responsibility assigned to named owner | ✅ Done | `docs/ENTERPRISE.md` — compliance-officer + security-engineer as primary owners |
| Code of conduct / acceptable use policy | 🟡 Gap | Draft needed; assign to compliance-officer |
| Background checks for employees with data access | 🟡 Gap | Required before first hire touching production |
| Documented org chart with data access roles | ✅ Done | `docs/ENTERPRISE.md` — role taxonomy; `docs/AUDIT_LOG_SCHEMA.md` — access control matrix |

### CC2 — Communication and Information

| Control | Status | Evidence |
|---|---|---|
| Security policies documented and accessible | 🟡 Partial | `docs/SECURITY.md` exists; formal Policy Pack (AUP, IRP, BCP) needed |
| Privacy policy published | 🔴 Gap | form.coach/privacy — must exist before first enterprise pilot |
| Vendor security review process | 🟡 Gap | DPAs signed with Anthropic, ElevenLabs, PostHog; formal vendor registry needed |
| Internal security training | 🔴 Gap | Annual training required once team > 1 person |

### CC3 — Risk Assessment

| Control | Status | Evidence |
|---|---|---|
| Formal risk assessment documented | ✅ Done | Section 14 — Formal Risk Register (18 risks, 6 categories, L×S scoring) |
| Risk register maintained, reviewed annually | 🟡 Partial | Section 14 exists; first annual formal review not yet performed (scheduled August 2026) |
| Vendor / third-party risk assessment | 🟡 Partial | Processor list in `docs/SECURITY.md` §5; needs formal scoring |

### CC4 — Monitoring of Controls

| Control | Status | Evidence |
|---|---|---|
| Continuous monitoring infrastructure | 🟡 Partial | PostHog for product; needs uptime monitoring (Better Uptime / Pagerduty) |
| HMAC-chained audit log with chain-integrity verification | ✅ Done | DEC-030; `docs/AUDIT_LOG_SCHEMA.md` — weekly cron, alert on chain break |
| Anomaly alerting (auth failures, spike detection) | 🟡 Gap | Rate limiting exists (Cloudflare WAF); needs alerting to security channel |
| Quarterly control effectiveness review | 🔴 Gap | Process needed post-hire |

### CC5 — Control Activities

| Control | Status | Evidence |
|---|---|---|
| Access control policies (least privilege) | ✅ Done | `docs/AUDIT_LOG_SCHEMA.md` — role matrix; break-glass procedure |
| MFA enforced for all admin access | 🟡 Gap | Enforce for Cloudflare dashboard, Supabase, 1Password |
| Separation of duties | 🟡 Partial | Founder is single-person; 2-person approval for break-glass is compensating control |
| Change management (code review, CI gates) | 🟡 Gap | Define PR review requirements and CI test gates |

### CC6 — Logical and Physical Access Controls

| Control | Status | Evidence |
|---|---|---|
| Unique credentials, no shared accounts | ✅ Done | `docs/SECURITY.md` §3 — no shared accounts policy |
| Session timeout, token expiry | ✅ Done | JWT 1-hour expiry, 30-day session with rotation |
| Certificate pinning (mobile) | ✅ Done | `docs/SECURITY.md` §2 Layer 1 |
| Offboarding process (credential revocation) | 🔴 Gap | Formal procedure needed before first hire |
| Physical security of infrastructure | ✅ N/A | Cloud-only (Cloudflare, Supabase) — inherit provider controls |

### CC7 — System Operations

| Control | Status | Evidence |
|---|---|---|
| Incident detection and response | ✅ Done | `docs/SECURITY.md` §7 severity levels (SEV-0 through SEV-3) + runbook |
| Vulnerability scanning (dependencies) | 🟡 Gap | Dependabot / `npm audit` planned; implement in CI |
| Patching SLA defined | 🟡 Gap | Define: critical patches <24h, high <7d, medium <30d |
| System health monitoring | 🟡 Gap | Uptime monitoring with customer-facing status page needed |

### CC8 — Change Management

| Control | Status | Evidence |
|---|---|---|
| All changes via version-controlled repo | ✅ Done | GitHub public repo (DEC-010) |
| Production deploy requires CI pass | 🟡 Gap | Enforce branch protection + required CI checks |
| Rollback capability documented | 🟡 Gap | Define rollback runbook in `docs/ENGINEERING_RUNBOOK.md` |
| Emergency change process | 🟡 Gap | Define; log as `system.config_changed` in audit log |

### CC9 — Risk Mitigation (Vendor Management)

| Control | Status | Evidence |
|---|---|---|
| DPAs signed with all sub-processors | ✅ Partial | Anthropic, ElevenLabs, PostHog — DPAs signed; Sentry pending |
| Sub-processor list published | 🔴 Gap | Required for GDPR + enterprise procurement |
| Annual vendor security review | 🔴 Gap | Process needed |

---

## 2 · Availability (A Series)

### A1 — Availability Commitments

| Control | Status | Evidence |
|---|---|---|
| SLA commitments documented | ✅ Done | `docs/ENTERPRISE.md` — 99.9% uptime, P0 <1h response |
| Uptime monitoring with historical data | 🔴 Gap | Must implement before observation period; Better Uptime or equivalent |
| Customer-facing status page | 🔴 Gap | status.form.coach — required for enterprise tier |
| SLA credit calculation and process | 🟡 Gap | Define formula and claim process |

### A1.2 — Capacity Planning

| Control | Status | Evidence |
|---|---|---|
| Load testing before launches | 🟡 Gap | Define pre-launch load test requirement |
| Auto-scaling configured | 🟡 Partial | Cloudflare Workers scale automatically; Supabase connection pooling needed |
| Disaster recovery RTO/RPO defined | ✅ Done | `docs/SECURITY.md` §10 — RTO 4h, RPO 1h |
| DR test performed annually | 🔴 Gap | Schedule annual DR drill |

---

## 3 · Processing Integrity (PI Series)

### PI1 — Processing Completeness and Accuracy

| Control | Status | Evidence |
|---|---|---|
| CV-coaching outputs validated before delivery | ✅ Done | `docs/SECURITY.md` §5 — confidence thresholds; uncertain outputs defer to templates |
| LLM output filtering (safety pass) | ✅ Done | Multi-step processing: classifier → response → output filter |
| Error handling — failed processing not silently dropped | 🟡 Gap | Define and test error propagation to user and logs |
| Data validation at all input boundaries | ✅ Done | `docs/SECURITY.md` §3 — Zod schema validation, parameterized queries |
| Audit trail for data transformations | ✅ Partial | `audit_log` covers privileged actions; pipeline-level lineage needed for analytics |

---

## 4 · Confidentiality (C Series)

### C1 — Confidentiality of Information

| Control | Status | Evidence |
|---|---|---|
| Confidential data classification policy | ✅ Done | Four-tier policy (Public / Internal / Confidential / Restricted) in Section 13; data inventory; handling requirements; audit log `data_class` field |
| Health data classified as Restricted | ✅ Done (implicitly) | GDPR Art. 9 treatment in SECURITY.md; needs explicit policy doc |
| Encryption at rest (AES-256) | ✅ Done | Supabase default; column-level encryption for sensitive fields |
| Encryption in transit (TLS 1.3) | ✅ Done | `docs/SECURITY.md` §2 — TLS 1.3 mandatory, HSTS |
| NDA / confidentiality agreements with employees | 🔴 Gap | Required before first hire |
| Data minimization in LLM prompts | ✅ Done | `docs/SECURITY.md` §5 — no PII in prompts, truncated history |

### C1.2 — Disposal of Confidential Information

| Control | Status | Evidence |
|---|---|---|
| Secure deletion process (Right to Erasure) | ✅ Done | `docs/SECURITY.md` §6 — 30-day grace, hard delete, backup purge within 60d |
| Audit log anonymization on delete | ✅ Done | Timestamp + action retained, user-id removed |
| Media/device disposal policy | 🔴 Gap | Formal policy needed (remote work context) |

---

## 5 · Privacy (P Series)

### P1 — Privacy Notice

| Control | Status | Evidence |
|---|---|---|
| Privacy policy published at collection point | 🔴 Gap | form.coach/privacy — must pre-date observation period |
| Privacy policy covers all data categories collected | 🔴 Gap | Health data, workout data, food data, payment data (via Apple/Google) |
| Cookie banner / consent management | 🔴 Gap | Required for GDPR; privacy.consent_granted logged per DEC-030 |

### P2 — Choice and Consent

| Control | Status | Evidence |
|---|---|---|
| Consent collected before processing special category data | 🟡 Partial | ED-screening flow per DEC-018; formal consent record in audit_log |
| Consent withdrawal mechanism | ✅ Done | `docs/SECURITY.md` §9 — opt-out analytics toggle, account deletion |
| analytics_opt_out logged | ✅ Done | `privacy.analytics_opt_out` in audit taxonomy |

### P3 — Collection Limitation

| Control | Status | Evidence |
|---|---|---|
| Data minimization — only collect what's needed | ✅ Done | No SSN, no gov ID, no payment info; pseudonymized IDs |
| Camera frames never leave device | ✅ Done | `docs/SECURITY.md` §4 — on-device inference, no frame upload |
| Purpose limitation documented | ✅ Done | `docs/GDPR_DPIA.md` v0.1 (May 2026) — full DPIA for health data processing; 12 processing activities (PA-01…PA-12), necessity & proportionality analysis, 12 risks scored |

### P4 — Use, Retention, Disposal

| Control | Status | Evidence |
|---|---|---|
| Retention schedule defined | ✅ Done | `docs/AUDIT_LOG_SCHEMA.md` — tiered retention 30d to 10y |
| Automated partition drop after retention period | ✅ Done | Cron-based partition drop; chain continuity via `audit_log_retention_summary` |
| Health data retention limit defined | 🟡 Gap | Define in privacy policy: how long workout/meal data is kept |

### P5 — Access

| Control | Status | Evidence |
|---|---|---|
| Data Subject Access Request (DSAR) process | 🟡 Gap | Define handling procedure; data.export_initiated in audit log |
| Export format defined (JSON) | ✅ Done | `docs/SECURITY.md` §9 — JSON export in Settings |
| Response time SLA for DSAR (GDPR: 30 days) | 🔴 Gap | Document and test |

### P6 — Disclosure and Notification

| Control | Status | Evidence |
|---|---|---|
| Breach notification procedure | ✅ Done | `docs/SECURITY.md` §6 — 72h supervisory authority, user notification |
| Sub-processor disclosure | 🔴 Gap | Published list required |
| Government request handling policy | 🟡 Gap | Define; FORM declines government backdoor requests (docs/ENTERPRISE.md) |

### P7 — Quality

| Control | Status | Evidence |
|---|---|---|
| Mechanism for users to correct their data | 🟡 Gap | In-app edit profile; audit log of changes needed |
| Accuracy checks on imported data | ✅ Partial | Zod validation at input; no automated accuracy audit |

### P8 — Monitoring and Enforcement

| Control | Status | Evidence |
|---|---|---|
| Privacy floor enforced — HR cannot see individual data | ✅ Done | `docs/ENTERPRISE.md` — 7 non-negotiable privacy rules; data.read_individual = forbidden + alert |
| Privacy incident tracking | 🟡 Gap | Formalize as part of incident response (severity classification) |
| Annual privacy review | 🔴 Gap | Schedule as part of compliance calendar |

---

## Gap Analysis Summary

| Severity | Count | Examples |
|---|---|---|
| 🔴 **Critical gap** (blocks SOC 2) | 9 | Privacy policy, status page, security training, offboarding procedure |
| 🟡 **Partial / needs formalization** | 21 | Vendor registry, patching SLA, annual risk review, DSAR SLA |
| ✅ **In place** | 25 | HMAC audit log, encryption, access controls, CV on-device, breach notification, data classification policy, DPIA, formal risk register |

**Readiness score: ~45% controls fully in place.** Target: 90% before observation period begins.

---

## Remediation Roadmap

### Phase 1 — Policies (Month 1–2) · compliance-officer owns

| Item | Deliverable |
|---|---|
| Publish privacy policy | form.coach/privacy — covers all data categories |
| Publish sub-processor list | form.coach/legal/sub-processors |
| Code of conduct / AUP | Internal document, signed by all employees |
| Data classification policy | Public / Internal / Confidential / Restricted |
| Vendor risk registry | List all processors with DPA status |
| DPIA for health data | Data Protection Impact Assessment |
| Employee NDA template | Before first hire |
| Offboarding checklist | Credential revocation within 24h |

### Phase 2 — Technical Controls (Month 2–3) · security-engineer + devops-lead own

| Item | Deliverable |
|---|---|
| Uptime monitoring | Better Uptime or equivalent; alert < 1min detection |
| Customer status page | status.form.coach with 90-day history |
| MFA enforcement | Cloudflare dashboard, Supabase, 1Password — all admin surfaces |
| Dependency scanning in CI | Dependabot + `npm audit`; fail CI on critical CVEs |
| Branch protection | Require PR review + CI pass before merge to main |
| Patching SLA enforcement | Automate CVE triage; critical <24h tracked in Linear |
| Cookie consent banner | GDPR-compliant; logs `privacy.consent_granted` to audit log |
| Anomaly alerting | Auth failure spikes → PagerDuty alert |

### Phase 3 — Process (Month 2–4) · compliance-officer + security-engineer own

| Item | Deliverable |
|---|---|
| Risk register | Quarterly review; owns CC3 evidence |
| Security training curriculum | Annual; completion tracked in HR system |
| DSAR handling procedure | 30-day SLA; data.export_initiated in audit log |
| DR test schedule | Annual drill; RTO/RPO validation |
| Penetration test | External firm; findings tracked to closure |
| Incident response tabletop | Quarterly exercise |
| Compliance calendar | Annual privacy review, vendor reviews, training |

---

## Evidence Collection Process

SOC 2 auditors require **evidence for each control** — not just documentation, but proof controls operated during the observation period. Below is what we collect and where it lives.

### Evidence registry

| Control Area | Evidence Type | Source | Frequency |
|---|---|---|---|
| Audit log integrity | HMAC chain validation report | `audit_log` HMAC cron | Weekly (automated) |
| Access reviews | List of active credentials and roles | IAM export (Cloudflare, Supabase) | Quarterly |
| Uptime | Historical availability data | status.form.coach + monitoring tool | Continuous; pulled at audit |
| Vulnerability management | Dependabot alerts and closure dates | GitHub / Linear | Continuous |
| Penetration test | Pentest report + finding remediation evidence | External firm | Annual |
| Security training | Completion records | HR system / LMS | Annual |
| Incident response | Incident tickets with timeline | Linear + Slack #incident | Per event |
| Change management | PR history, CI logs, deploy logs | GitHub + Cloudflare | Continuous |
| Vendor reviews | DPA list, security questionnaire responses | Compliance officer folder | Annual |
| DSAR handling | Export request tickets, delivery timestamps | Linear + audit log | Per event |
| Consent records | `privacy.consent_granted` events | audit_log | Continuous |
| Breach notifications | Notification emails sent + timestamps | Email records + audit log | Per event |

### Evidence collection cadence

**Continuous (automated):**
- Audit log entries — every privileged action
- HMAC chain verification — weekly cron
- Uptime monitoring — 1-min polling
- Dependency vulnerability alerts — on PR + daily

**Monthly (manual pull):**
- Access list review — who has what access
- Open vulnerability review — unpatched CVEs
- Incident review — any events in period

**Quarterly:**
- Formal access review + deprovisioning
- Vendor security review
- Control effectiveness assessment
- Risk register update

**Annual:**
- External penetration test
- DR drill
- Security training completion audit
- Full control gap assessment (re-run this document)

---

## Audit Firm Engagement

### Selection criteria

- Experience with SaaS / health-adjacent products
- Familiar with Cloudflare + Supabase stack
- Can issue report within 3-month fieldwork window
- Pricing range: $15k–$40k for Type II (B2B SaaS scale)

### Engagement timeline

```
Month 3  Readiness assessment (pre-audit)
         → Identify remaining gaps
         → Agree observation period start date

Month 3  Observation period begins
         → All controls must be operating
         → Evidence collection automated

Month 9  Fieldwork begins
         → Auditor tests controls, interviews team
         → Evidence package submitted

Month 11 Draft report review
         → Management response to exceptions
         → Remediation of minor findings

Month 12 Final report issued
         → SOC 2 Type II report available to customers under NDA
```

### What customers see

Enterprise customers receive the SOC 2 Type II report under NDA on request. The summary letter (bridge letter) is available quarterly between annual reports. FORM does not publish the full report publicly.

---

## Continuous Compliance Tooling

Post-Month 3, we implement continuous compliance tooling to reduce manual evidence collection burden.

| Tool | Purpose | Timing |
|---|---|---|
| **Vanta** or **Drata** | Control monitoring, evidence automation, auditor portal | Month 3 (observation period start) |
| **Dependabot** | Vulnerability scanning, CVE triage | Month 2 (in CI) |
| **Better Uptime** | Availability monitoring, status page | Month 2 |
| **PagerDuty** | Incident alerting, on-call rotation | Month 3 |
| **Linear** | Vulnerability tracking, incident tickets | Existing |

Integration point: Vanta/Drata connects to GitHub, Cloudflare, Supabase, and pulls evidence automatically. Reduces auditor evidence package from weeks of manual work to days.

---

## Sub-Processor Register

SOC 2 CC9 and GDPR Art. 28 require FORM to maintain and disclose a sub-processor list. This register is provided to enterprise customers on request and published at `form.coach/legal/sub-processors` (target: before observation period start). Customers are notified **30 days in advance** of any new sub-processor addition and have the right to object per GDPR Art. 28(2).

| Sub-Processor | HQ | Purpose | Data Categories | Region | DPA Status | GDPR Transfer Mechanism |
|---|---|---|---|---|---|---|
| Anthropic PBC | San Francisco, US | LLM inference (Victor coaching AI) | Pseudonymised exercise context; no PII in prompts by design | US | ✅ Signed | SCC Module 2 (controller→processor) |
| ElevenLabs Inc | New York, US | Text-to-speech (Victor voice) | Coaching text cues; no user PII | US | ✅ Signed | SCC Module 2 |
| Supabase Inc | San Francisco, US | Managed PostgreSQL database, authentication, storage | All user and tenant data | US `us-east-1` (default) / EU `eu-central-1` opt-in | ✅ Signed | SCC Module 2; EU region available |
| Amazon Web Services | Seattle, US | Cloud infrastructure (via Supabase) | All data (via Supabase) | Inherits Supabase region | Covered by Supabase DPA | Supabase SCC covers downstream |
| Cloudflare Inc | San Francisco, US | CDN, DDoS, edge compute (Workers) | IP addresses, request metadata; no user health data in Workers | US + global edge | ✅ Enterprise DPA | SCC Module 2 |
| PostHog Inc | San Francisco, US | Product analytics | Anonymous event data, `analytics_opt_out` flag; no PII by configuration | US / EU self-hosted option | ✅ Signed | SCC Module 2; EU hosting available |
| Functional Software (Sentry) | San Francisco, US | Error monitoring, crash reporting | Error stack traces, anonymised user IDs | US | 🟡 In progress | SCC Module 2 pending DPA signature |
| Stripe Inc | San Francisco, US | Payment processing (enterprise direct billing) | Billing contact name, company, invoice data; no health data | US / EU | ✅ Signed | SCC Module 2 |

**Key notes:**
- No PII or health data is included in prompts sent to Anthropic. Prompts contain only pseudonymised exercise context (sets, reps, RPE, programme phase). See `docs/SECURITY.md §5`.
- Sentry DPA is in-progress; as a compensating control, PII is scrubbed from Sentry payloads at the SDK level pending formal DPA signature.
- EU data residency (Supabase `eu-central-1`, PostHog EU) is available for enterprise tenants and is selected at tenant creation; it cannot be changed post-deployment without a full data migration.

---

## Complementary User Entity Controls (CUECs)

In a SOC 2 Type II report, the auditor documents controls that FORM's enterprise customers (user entities) must implement on their own side for FORM's controls to be fully effective. These CUECs will appear in the issued SOC 2 report. Enterprise customers should review and confirm implementation before relying on the FORM SOC 2 report for their own compliance obligations.

### Identity and Access Management

| # | Customer Obligation |
|---|---|
| CUEC-01 | Designate at least one named FORM `tenant_owner` responsible for access management — provisioning, deprovisioning, and role assignments within the tenant. |
| CUEC-02 | Configure and maintain a secure Identity Provider (IdP) for SAML/OIDC SSO. FORM enforces SSO for configured tenants but cannot control the security of the customer's IdP (Okta, Azure AD, Google Workspace, etc.). |
| CUEC-03 | Enforce MFA on all users in the IdP for accounts that authenticate to FORM. For SSO-configured tenants, FORM delegates MFA enforcement to the IdP; customers must ensure MFA is active at the IdP layer. |
| CUEC-04 | Notify FORM — or automate via SCIM — within 24 hours of employee termination or role change requiring deprovisioning. SCIM-managed accounts are deactivated within minutes of a `PATCH active: false` event; non-SCIM tenants depend on manual action by the customer admin. |
| CUEC-05 | Do not share `tenant_admin` or `tenant_owner` credentials. Shared credentials invalidate the per-user audit trail and will be treated as a control failure during audit. |
| CUEC-06 | Review the FORM admin audit log at least quarterly for unexpected access patterns. FORM provides audit log export (JSON, CSV). Investigation of anomalies specific to customer behaviour is the customer's responsibility. |

### Data Responsibility

| # | Customer Obligation |
|---|---|
| CUEC-07 | Inform employees about the FORM wellness programme and data collection before provisioning accounts. FORM obtains explicit in-app consent at first login; the employer is responsible for workforce communication and pre-enrollment disclosure. |
| CUEC-08 | Do not use FORM admin dashboard aggregate metrics as the basis for individual employment decisions. FORM enforces k-anonymity (groups < 5 suppressed) and does not expose individual metrics; customers must maintain internal policies governing permissible uses of aggregate wellness data. |
| CUEC-09 | Ensure employee devices meet the customer's endpoint security standards. FORM enforces certificate pinning, TLS 1.3, and on-device CV inference; device compromise at the client side is outside FORM's control boundary. |
| CUEC-10 | Make employees aware of their right to disconnect from the employer tenant at any time (Settings → Privacy → "Disconnect from employer"). FORM enforces this right technically; the customer must ensure employees know it exists. |

### Incident Cooperation

| # | Customer Obligation |
|---|---|
| CUEC-11 | Maintain a current security contact in the FORM admin dashboard. FORM commits to P0 tenant notification within 1 hour; an unreachable contact delays customer-side incident response. |
| CUEC-12 | Cooperate with FORM during incident response and post-mortems. Customers may hold IdP authentication logs, endpoint logs, or HR access records that FORM requires for forensic investigation. |

---

## Common Security Questionnaire Responses

Enterprise procurement teams routinely send security questionnaires (CAIQ v4, SIG Lite, or custom). Below are pre-approved answers to the most common questions. Reviewed quarterly by compliance-officer + security-engineer. For full questionnaire submissions, contact `security@form.coach`.

### Data residency
> *Where is customer data stored? Can we request EU-only storage?*

By default, data is stored in AWS `us-east-1` (N. Virginia). Enterprise customers may elect EU residency at tenant creation (`eu-central-1` or `eu-west-1`), routing all database, object storage, and backups to EU infrastructure. Region selection is contractually bound and cannot be changed post-deployment without a migration engagement.

### Encryption
> *How is data encrypted at rest and in transit?*

**At rest:** AES-256 via AWS RDS encrypted volumes for all data. Health data and CV pose keypoints additionally encrypted with per-tenant AWS KMS Customer Managed Keys (CMKs); annual automatic rotation.

**In transit:** TLS 1.3 minimum for all connections. HSTS enforced on all web endpoints. Certificate pinning on iOS and Android clients. Internal service-to-service communication uses mutual TLS.

### Penetration testing
> *Do you conduct penetration tests? Can we see the report?*

Annual external penetration test by an independent security firm. The executive summary (finding counts by severity, remediation timeline) is available to enterprise customers under NDA. Full reports are shared at FORM's discretion with customers who have a material security review requirement on enterprise contracts >$50k ACV.

### Incident notification
> *How quickly will you notify us of a security breach?*

FORM commits to notifying affected enterprise tenants within **1 hour** of a confirmed P0 security incident affecting their tenant data. GDPR 72-hour supervisory authority notification is handled by FORM's compliance-officer. See `docs/INCIDENT_RESPONSE.md` for the full escalation tree and communication templates.

### Data deletion and offboarding
> *What happens to our data when we terminate the contract?*

Upon contract termination, FORM holds data for a 30-day export window. After 30 days, all tenant user data is hard-deleted from primary storage; backup copies purged within 60 days. Audit logs are retained for 7 years (SOC 2 / financial records requirement) but anonymised after user deletion (user_id replaced; action retained per `docs/AUDIT_LOG_SCHEMA.md`). A deletion certificate is issued on request.

### Business continuity
> *What are your RTO and RPO targets?*

**RTO:** 4 hours (time to restore service after complete infrastructure failure). **RPO:** 1 hour (maximum data loss). Both are contractually committed at the Enterprise tier, backed by AWS Multi-AZ RDS with Point-in-Time Recovery (PITR) at 1-minute granularity. Annual DR drills validate these targets; drill reports are available under NDA.

### AI model training
> *Is our data used to train your AI models?*

No. Enterprise customer data is never used to train or fine-tune any AI model. Prompts to Anthropic Claude (via API) are subject to Anthropic's enterprise DPA, which prohibits using customer data for model training. FORM's DPA with enterprise customers explicitly excludes customer data from any training or evaluation use.

### Subprocessors
> *Who are your subprocessors? How do you notify us of changes?*

See the Sub-Processor Register in this document and `form.coach/legal/sub-processors`. FORM provides 30 days' advance notice before adding a new sub-processor, giving customers the right to object. Objection procedure: written notice to `privacy@form.coach` within the notice period; FORM will work to find an alternative or allow contract termination without penalty if no alternative is acceptable.

---

## Data Classification Policy

> Owner: compliance-officer + security-engineer. Effective: May 2026. Review: annual or on material data model change. Closes SOC 2 gap C1.1.

FORM classifies all data into four tiers. Every engineer, vendor, and team member handling FORM data must apply the controls corresponding to the data's tier. The tier follows the data — if a document, log entry, or API response contains data from multiple tiers, the **highest tier applies to the whole**.

---

### Tier Definitions

#### Tier 0 — Public

Data explicitly approved for unrestricted external distribution.

**Examples at FORM:** form.coach marketing pages, blog posts, press releases, app store screenshots, public pricing pages, open-source code, public API documentation, status page uptime history, security disclosure policy (`security@form.coach`).

| Requirement | Rule |
|---|---|
| Encryption in transit | TLS (standard) |
| Encryption at rest | Not required |
| Access control | None — intentionally public |
| Logging | Not required |
| Breach notification | Not applicable |
| HR visibility | Allowed |
| LLM prompt inclusion | Allowed |
| Code label | `class:public` |

---

#### Tier 1 — Internal

Data shared within the FORM team that is not approved for external disclosure.

**Examples at FORM:** Architecture docs (TECHNICAL.md, ENGINEERING_RUNBOOK.md), OKRs, roadmap, non-public financial projections, hiring docs, internal Slack channels, Linear tickets, vendor contracts (non-health-data-related), team meeting notes, non-public investor materials.

| Requirement | Rule |
|---|---|
| Encryption in transit | TLS 1.3 |
| Encryption at rest | Not required (GitHub private repo, Notion workspace controls suffice) |
| Access control | Team members only; no external sharing without compliance-officer approval |
| Logging | Not required for reads; data.export logged for bulk extractions |
| Breach notification | Notify compliance-officer within 24h; external notification only if personal data is confirmed exposed |
| HR visibility | Allowed |
| LLM prompt inclusion | Allowed (no PII in prompts per `docs/SECURITY.md §5`) |
| Code label | `class:internal` |

---

#### Tier 2 — Confidential

Business-sensitive data whose exposure would materially harm FORM or its enterprise customers.

**Examples at FORM:** User email, display name, and account metadata; enterprise tenant configuration (company name, contract value, seat count, IdP settings); employee PII; Stripe billing records; API keys and secrets (non-health); DPAs with vendors; audit log entries (without health payload); Linear issue content referencing customer details.

| Requirement | Rule |
|---|---|
| Encryption in transit | TLS 1.3 mandatory |
| Encryption at rest | AES-256 (Supabase default; Cloudflare KV encryption for edge-cached values) |
| Access control | Role-based; least-privilege; documented in `docs/AUDIT_LOG_SCHEMA.md` role matrix |
| Logging | All privileged reads and all writes logged to `audit_log` with `data_class: confidential` |
| Breach notification | Affected enterprise tenants within 1h (P0 protocol); GDPR Art. 33 supervisory authority within 72h |
| HR visibility | No individual customer or employee data without explicit role assignment |
| LLM prompt inclusion | Pseudonymised only — user IDs replaced with session tokens; no direct email or name in prompt |
| Code label | `class:confidential` |
| Secret rotation | API keys: 90-day rotation maximum; break-glass keys: immediate rotation after use |

---

#### Tier 3 — Restricted (Special Category)

GDPR Art. 9 special category data and any data whose exposure directly harms users' physical or psychological wellbeing. This tier carries the highest protection requirements and is **never negotiable** regardless of customer or investor requests.

**Examples at FORM:** Workout biometrics (heart rate, HRV, wearable sensor data); CV pose keypoints and inferred movement patterns; ED-screening responses; explicit nutrition disorder indicators; mental health self-reports (surfaced in Victor coaching sessions); body composition data; clinical_flags entries; coach_sessions content containing health context.

| Requirement | Rule |
|---|---|
| Encryption in transit | TLS 1.3 mandatory; mTLS for service-to-service paths carrying Restricted data |
| Encryption at rest | AES-256 + per-tenant AWS KMS Customer Managed Key (CMK); annual automatic rotation |
| Access control | Break-glass only for operational access; requires `incident_id` justification logged before access; RLS policy `rls_restricted_no_hr` blocks HR-tier roles at the database layer |
| Logging | Every access (read, write, export, delete) logged to `audit_log` with `data_class: restricted`; system actor reads are `actor_type: system` to suppress per-access alert while still logging |
| Alert | Any `data_class: restricted` access by a non-system actor triggers automated alert to `#security-alerts` within 30 seconds |
| Breach notification | P0 protocol (tenant notification within 1h) + GDPR Art. 33 supervisory authority 72h + Art. 34 individual notification if high-risk exposure confirmed |
| HR visibility | **Never.** `tenant_manager` role has zero query access to Restricted rows. No exception, no contract clause can override. |
| LLM prompt inclusion | **Never.** Restricted data is not included in any prompt sent to Anthropic or any other external model. Prompts contain only pseudonymised exercise context (sets/reps/RPE/phase). See `docs/SECURITY.md §5`. |
| Cross-border transfer | SCC Module 2 (controller→processor) required; DPIA required before new processing purpose |
| Code label | `class:restricted` |
| Retention | User-controlled right to erasure; hard delete ≤30 days from request; backup purge ≤60 days; audit log anonymised (user_id replaced, action and timestamp retained per DEC-030) |

---

### FORM Data Inventory by Classification Tier

| Data Type | Tier | Primary Location | At-Rest Encryption | Notes |
|---|---|---|---|---|
| form.coach marketing pages | Public | Cloudflare Pages | TLS only | No PII |
| Blog posts, press releases | Public | GitHub (public) | TLS only | No PII |
| Architecture docs, roadmap | Internal | GitHub (private) | TLS only | Team access |
| OKRs, financials (non-public) | Internal | Notion / private repo | TLS only | NDA covers |
| User email, display name | Confidential | Supabase `auth.users` | AES-256 | Identity data |
| Enterprise company name, ACV | Confidential | Supabase `tenants` | AES-256 | B2B metadata |
| Admin email, IdP / SCIM config | Confidential | Supabase `tenant_sso_config` | AES-256 | Enterprise SSO |
| Stripe billing data | Confidential | Stripe (processor) + Supabase `billing_events` | AES-256 | SCC covers Stripe |
| API keys (Anthropic, ElevenLabs) | Confidential | 1Password + Cloudflare Secrets | Vault encryption | 90-day rotation |
| Audit log entries (no health) | Confidential | Supabase `audit_log` | AES-256 + HMAC | 7-year retention |
| Workout biometrics (HR, HRV) | **Restricted** | Supabase `wearable_sync` | CMK + AES-256 | GDPR Art. 9 |
| CV pose keypoints | **Restricted** | On-device only | Device encryption | Never uploaded |
| ED-screening responses | **Restricted** | Supabase `user_flags` | CMK + AES-256 | clinical-safety VETO |
| Mental health self-reports | **Restricted** | Supabase `coach_sessions` | CMK + AES-256 | Aggregated-only in admin |
| Victor coaching session content | **Restricted** | Supabase `coach_sessions` | CMK + AES-256 | No PII in Anthropic prompts |
| Body composition data | **Restricted** | Supabase `body_metrics` | CMK + AES-256 | k-anon in admin dashboard |
| `clinical_flags` entries | **Restricted** | Supabase `clinical_flags` | CMK + AES-256 | clinical-safety VETO |

*CV pose keypoints never leave the device (on-device CoreML inference). They are listed for completeness; the "never uploaded" note is a privacy-by-design guarantee, not a policy control.*

---

### Handling Requirements Comparison

| Requirement | Public | Internal | Confidential | Restricted |
|---|---|---|---|---|
| TLS in transit | ✅ | ✅ 1.3 | ✅ 1.3 | ✅ 1.3 + mTLS |
| Encryption at rest | ❌ | ❌ | ✅ AES-256 | ✅ AES-256 + CMK |
| Access logging | ❌ | Bulk only | ✅ All privileged | ✅ All access |
| Security alert on access | ❌ | ❌ | ❌ | ✅ Non-system actor |
| HR visibility | ✅ | ✅ | No individual | 🚫 Never |
| LLM prompt allowed | ✅ | ✅ | Pseudonymised | 🚫 Never |
| Cross-border SCC required | ❌ | ❌ | ✅ | ✅ + DPIA |
| Breach notification SLA | None | 24h internal | 1h tenants + 72h SA | P0 + Art. 33 + Art. 34 |
| Retention control | Indefinite | While relevant | Contract / law | User-controlled + erasure |
| Key rotation | n/a | n/a | 90 days (secrets) | Annual (CMK) |

---

### Audit Log `data_class` Field

All audit log entries for data-access actions **must** include a `data_class` field. This enables auditors to verify that access patterns match the policy defined above.

```jsonc
{
  "event_id": "evt_01J…",
  "timestamp": "2026-05-17T10:00:00.000Z",
  "action": "data.read",
  "resource": "wearable_sync",
  "data_class": "restricted",         // ← required on all data-access events
  "actor_id": "usr_abc123",
  "actor_type": "user",               // "user" | "system" | "break-glass"
  "tenant_id": "acme-corp",
  "incident_id": null,                // required if actor_type = "break-glass"
  "trace_id": "4bf92f3577b34da6a"
}
```

**Alerting rule:** Any event where `data_class = restricted` AND `actor_type = user` triggers an automated Slack alert to `#security-alerts` within 30 seconds. System cron jobs use `actor_type: system` and are logged without alert; break-glass access requires a non-null `incident_id` before the query is permitted.

This field also enables the compliance officer to run a quarterly access review query:

```sql
SELECT actor_id, data_class, COUNT(*) AS access_count, MAX(timestamp) AS last_access
FROM audit_log
WHERE timestamp >= NOW() - INTERVAL '90 days'
  AND data_class IN ('confidential', 'restricted')
GROUP BY actor_id, data_class
ORDER BY data_class DESC, access_count DESC;
```

---

### SOC 2 Criteria Closed by This Policy

| Criterion | Description | Status |
|---|---|---|
| **C1.1** | FORM identifies and maintains confidential information per commitments and requirements | ✅ Closed — this policy |
| **CC6.1** | Implements access controls limiting logical access to confidential information | ✅ Supported — RLS policy `rls_restricted_no_hr` + role matrix |
| **P3.1** | Collects personal information only for identified purposes | ✅ Supported — data inventory + purpose limitation per tier |
| **P6.1** | Creates and retains records of authorized disclosures of personal information | ✅ Supported — audit log `data_class` field |

---

## Privacy Floor Enforcement (Non-Negotiable)

The following controls are enforced regardless of customer requests. They are non-bypassable per `docs/ENTERPRISE.md` and `clinical-safety` VETO authority:

1. HR role (`tenant_manager`) **never** receives access to individual user health data
2. `data.read_individual` action is logged + triggers immediate security alert if attempted at HR tier
3. Aggregate wellness metrics exclude: body composition, mental health indicators, ED-screening responses
4. Employee may revoke company association at any time — `privacy.consent_withdrawn` logged
5. Any customer request to bypass these controls is declined; deal is lost if necessary

These map to SOC 2 Privacy criteria P2, P3, and P6, and simultaneously satisfy GDPR Art. 9 (special category data) requirements.

---

## 14. Formal Risk Register (CC3)

> Owner: `compliance-officer` + `security-engineer`. Review: quarterly. First review: August 2026.
> SOC 2 evidence: CC3.1 (risk identification), CC3.2 (risk analysis), CC3.3 (risk mitigation).

### 14.1 Scoring Methodology

| Dimension | 1 | 2 | 3 | 4 | 5 |
|---|---|---|---|---|---|
| **Likelihood** | Rare (<once/5yr) | Unlikely (<once/yr) | Possible (1×/yr) | Likely (several/yr) | Almost Certain (monthly+) |
| **Impact** | Negligible | Minor (service hiccup) | Moderate (user-visible, recoverable) | Major (SLA breach, data exposure) | Catastrophic (breach, regulatory, existential) |

**Score = Likelihood × Impact**

| Score | Rating | Response |
|---|---|---|
| 1–5 | LOW | Accept; monitor quarterly |
| 6–11 | MEDIUM | Monitor; document mitigations |
| 12–19 | HIGH | Mitigate; assign owner; track to closure |
| 20–25 | CRITICAL | Immediate remediation; halt feature work if active |

**Risk appetite:** MEDIUM — inherent scores ≥ 12 require documented mitigation plan and named owner. FORM has zero tolerance for risks affecting health data confidentiality or HMAC audit log integrity.

### 14.2 Risk Register

#### Security Risks (CC6, CC7)

| ID | Risk | Inherent L | Inherent I | Inherent Score | Controls | Residual Score | Owner | Status |
|---|---|---|---|---|---|---|---|---|
| SR-01 | **JWT / session token compromise** — attacker obtains a valid token via MITM or device compromise, gains access to user data | 3 | 5 | **15 HIGH** | TLS 1.3 mandatory; JWT 1h expiry; 30-day session rotation; RLS fail-closed (token cannot escalate beyond its `tenant_id`); certificate pinning (mobile) | **6 MEDIUM** | security-engineer | Mitigated |
| SR-02 | **Credential stuffing / auth brute force** — automated bot replays leaked credentials against FORM login | 3 | 4 | **12 HIGH** | Cloudflare WAF rate limits (50 req/min/IP cap); `supabase_auth_failures_total` alert (>50/min → P1 page); future: CAPTCHA after 5 failures | **4 LOW** | security-engineer | Mitigated |
| SR-03 | **Insider threat** — future employee with production access exfiltrates user health data or tampers with audit logs | 2 | 5 | **10 MEDIUM** | RBAC (least privilege); break-glass dual-authorization; HMAC-chained audit log (DEC-030); `data.read_individual` audit event + `#security-alerts`; access reviewed quarterly | **4 LOW** | compliance-officer | Mitigated |
| SR-04 | **Supply chain attack** — malicious npm package or Cloudflare Worker dependency introduces backdoor | 2 | 5 | **10 MEDIUM** | Dependabot + `npm audit` in CI; `package-lock.json` pinned; planned: CI fails on critical CVEs (Phase 2 roadmap) | **5 MEDIUM** | devops-lead | Partially mitigated |
| SR-05 | **API key / secret exposure** — credentials committed to codebase or leaked via CI logs | 2 | 4 | **8 MEDIUM** | All secrets in GitHub Secrets + 1Password; `git-secrets` pre-commit hook; GitHub secret scanning enabled; no `.env` committed policy | **3 LOW** | security-engineer | Mitigated |

#### Availability Risks (A1)

| ID | Risk | Inherent L | Inherent I | Inherent Score | Controls | Residual Score | Owner | Status |
|---|---|---|---|---|---|---|---|---|
| AR-01 | **Anthropic API extended outage** (>4h) — Victor cannot respond; core product feature unavailable | 2 | 4 | **8 MEDIUM** | Graceful degradation: pre-built Victor coaching-cue templates served for common scenarios; user-visible status message; `anthropic_api_requests_total{outcome="error"}` alert (P0 if >80% in 5min) | **4 LOW** | devops-lead | Mitigated |
| AR-02 | **Supabase Postgres outage** — database unreachable; all authenticated operations fail | 2 | 5 | **10 MEDIUM** | Supabase High Availability (primary + replica, automatic failover); PITR enabled (RPO 1h, RTO 4h per SECURITY.md §10); PITR restore runbook (DATA_MODEL.md §10); uptime check pages on-call in <60s | **4 LOW** | devops-lead | Mitigated |
| AR-03 | **Cloudflare Workers platform outage** — edge API completely unreachable | 1 | 5 | **5 LOW** | Cloudflare 99.99% global network SLA; Workers run across 300+ PoPs; no single-region dependency; Cloudflare status page auto-opens P1 incident | **2 LOW** | devops-lead | Accept |
| AR-04 | **ElevenLabs voice service outage** — voice coaching unavailable | 3 | 2 | **6 MEDIUM** | Silent fallback: text coaching continues uninterrupted; R2 TTS audio cache serves common cues (target >60% hit rate); user messaging: "Voice unavailable — coaching continues in text" | **2 LOW** | platform-engineer | Mitigated |

#### Processing Integrity Risks (PI1)

| ID | Risk | Inherent L | Inherent I | Inherent Score | Controls | Residual Score | Owner | Status |
|---|---|---|---|---|---|---|---|---|
| IR-01 | **Victor AI produces harmful coaching output** — injury-inducing advice, disordered-eating framing, or medically inappropriate guidance delivered to user | 2 | 4 | **8 MEDIUM** | `clinical-safety` VETO protocol (mandatory review before any body-image / injury / mental-health content ships); output filters; confidence thresholds (uncertain output → template fallback); ED-screening isolation (DEC-018); no-go list enforced | **2 LOW** | clinical-safety | Mitigated |
| IR-02 | **Cross-tenant data corruption via migration** — schema migration fan-out corrupts or intermixes tenant data | 2 | 5 | **10 MEDIUM** | RLS fail-closed on all reads; canary migration on test tenant before production; automated rollback on CI failure; migration tested against PITR restore (DATA_MODEL.md §10) | **4 LOW** | enterprise-architect | Mitigated |

#### Confidentiality Risks (C1)

| ID | Risk | Inherent L | Inherent I | Inherent Score | Controls | Residual Score | Owner | Status |
|---|---|---|---|---|---|---|---|---|
| CR-01 | **Cross-tenant RLS bypass** — one enterprise tenant reads another tenant's workout or health data | 2 | 5 | **10 MEDIUM** | Fail-closed RLS (missing `tenant_id` → query returns 0 rows, never all rows); `tenant_id` sourced from verified JWT claims (not client-supplied); `tenant_id_missing` counter alert fires P1 if counter >0 (OBSERVABILITY.md §6.2); quarterly RLS policy audit | **3 LOW** | security-engineer | Mitigated |
| CR-02 | **GDPR Art. 9 health data in operational logs** — workout data, body metrics, or AI prompt content logged to Cloudflare Logpush or Sentry | 2 | 5 | **10 MEDIUM** | AI inference log schema stores only token counts and latency — no prompt or response content (OBSERVABILITY.md §4.5); Sentry `beforeSend` hook strips `workout_data`, `body_metrics`, `coaching_context` before transmission (gap: not yet implemented — see OBSERVABILITY.md Open Questions); 30-day retention for AI inference logs | **4 LOW** | platform-engineer | Partially mitigated |

#### Privacy Risks (P1–P8)

| ID | Risk | Inherent L | Inherent I | Inherent Score | Controls | Residual Score | Owner | Status |
|---|---|---|---|---|---|---|---|---|
| PR-01 | **Consent management failure** — GDPR-required consent not recorded before processing special-category health data | 3 | 4 | **12 HIGH** | Consent-at-collection flow (in-app); `privacy.consent_granted` logged to audit trail; ED-screening consent separate from general T&C (DEC-018); withdrawal mechanism in Settings → `privacy.consent_withdrawn` | **6 MEDIUM** | compliance-officer | Partially mitigated (gap: formal consent management platform not yet deployed) |
| PR-02 | **DSAR response missed** — FORM fails to deliver data export within GDPR 30-day deadline | 3 | 3 | **9 MEDIUM** | `data.export_initiated` logged to audit log; JSON export available in Settings (SECURITY.md §9); formal DSAR SLA procedure (Phase 3 roadmap — not yet documented); Linear ticket with 30-day deadline on each DSAR | **6 MEDIUM** | compliance-officer | Partially mitigated |

#### Vendor / Operational Risks (CC9, CC1)

| ID | Risk | Inherent L | Inherent I | Inherent Score | Controls | Residual Score | Owner | Status |
|---|---|---|---|---|---|---|---|---|
| VR-01 | **Anthropic pricing increase or service termination** — LLM infrastructure costs double or primary vendor exits market | 2 | 4 | **8 MEDIUM** | Model tiering flexibility (Haiku for classification, Sonnet for coaching); prompt caching reduces token consumption ~35–40%; cost sensitivity analysis (COST_MODEL.md §10.1 — 2× API cost moves GM only 1.4 pp); architecture does not hard-code Anthropic SDK in mobile client | **5 MEDIUM** | founder | Accept and monitor |
| VR-02 | **Sentry DPA not confirmed** — processing EU crash data without valid DPA creates GDPR Art. 28 gap | 3 | 3 | **9 MEDIUM** | Sentry DPA review in progress; EU Sentry server (sentry.io EU region) available as alternative; `beforeSend` health data filter (partially implemented) limits sensitive data in crash reports | **6 MEDIUM** | compliance-officer | In progress |
| OR-01 | **Key person dependency** — founder incapacitation; no other person can operate production systems | 2 | 5 | **10 MEDIUM** | All infrastructure documented in `docs/` repo; all code in GitHub; break-glass credentials in 1Password shared vault (emergency-access contact defined); `docs/ENGINEERING_RUNBOOK.md` covers all operational procedures | **7 MEDIUM** | founder | Accept; revisit at first engineering hire |

### 14.3 Risk Heatmap Summary

```
Impact →      1-Negligible  2-Minor  3-Moderate  4-Major  5-Catastrophic
Likelihood ↓
5-Almost Cert                                              
4-Likely                                                   
3-Possible                          PR-02,VR-02  SR-02,PR-01  
2-Unlikely            AR-03         VR-01        AR-01,SR-05  SR-01,SR-03,SR-04
                                                 IR-01        AR-02,IR-02
                                                              CR-01,CR-02
1-Rare                AR-03
```

**Inherent risks requiring active mitigation (score ≥ 12):** SR-01 (15), SR-02 (12), PR-01 (12). All three have residual scores ≤ 6 with current controls.

**Highest residual risks (score ≥ 6):** PR-01 (6), PR-02 (6), VR-01 (5), VR-02 (6), SR-04 (5), OR-01 (7). These are tracked as open items in the remediation roadmap.

### 14.4 Review and Update Schedule

| Trigger | Action |
|---|---|
| Quarterly | Re-score all risks; update status column; add new risks identified since last review |
| After any P0 or P1 incident | Create new risk entry or update existing risk's inherent score |
| Before audit observation period begins | Full risk register review with audit firm; confirm all HIGH inherent risks have documented mitigations |
| On any new vendor added to sub-processor register | Add vendor risk entry; assign to compliance-officer |
| On any new product feature involving health data | Add risk entry before feature ships; clinical-safety review required |

---

## 15. Annual Compliance Calendar

> Owner: `compliance-officer` + `security-engineer`. Review: annually each January, or on material change to team size, product scope, or regulatory environment.
> Purpose: converts the recurring controls in Sections 1–14 from documented policies into operated controls with datable evidence — the thing SOC 2 Type II actually audits.
> Reference: CC4.1 (monitoring), CC4.2 (control effectiveness), CC9.1 (vendor management), CC1.4 (HR practices), A1.2 (capacity/DR), P8.1 (privacy enforcement), CC2.2 (training).

---

### 15.1 Master Compliance Calendar

The table below is the single authoritative schedule for recurring compliance activities. Each row maps to a specific control, a named owner, a concrete evidence artifact, and the SOC 2 criterion it closes. "Month" references are relative to the observation period start date (Month O+0). For Year 1 (pre-observation), use the absolute month mapping in Section 15.2.

| Month | Activity | Cadence | Owner | Evidence Artifact | SOC 2 Control Closed |
|---|---|---|---|---|---|
| **Every month** | Evidence collection checkpoint — pull access list, open CVEs, audit log anomaly review, incident summary | Monthly | compliance-officer | Monthly compliance memo filed to `compliance/YYYY-MM/evidence-memo.md` in private repo | CC4.1 (monitoring of controls) |
| **Every month** | Sub-processor list currency check — confirm no new processors added without 30-day customer notice | Monthly | compliance-officer | Updated `form.coach/legal/sub-processors` publish date + git commit timestamp | CC9.1, GDPR Art. 28(3) |
| **Every month** | HMAC chain integrity verification — confirm weekly cron ran and no chain breaks | Monthly | security-engineer | `audit_log_chain_verification` report; alert-free confirmation in `#security-alerts` | CC4.1, CC7.3 |
| **Q1 (Jan)** | Annual Disaster Recovery drill — simulate complete Supabase primary failure; validate RTO ≤4h, RPO ≤1h using PITR restore runbook | Annual | devops-lead + security-engineer | DR drill report: test date, RTO achieved, RPO achieved, gaps identified, sign-off by compliance-officer | A1.2 (DR test performed annually) — closes 🔴 gap |
| **Q1 (Jan)** | Annual penetration test — commission external firm; scope: API endpoints, auth flows, RLS bypass attempts, Cloudflare Workers | Annual | security-engineer (procurement); external firm (execution) | Pentest report with finding counts by severity + remediation plan; executive summary for enterprise customers | CC7.1 (vulnerability identification), CC9.2 |
| **Q1 (Jan)** | Annual policy review — AUP, IRP, BCP, data classification, vendor management, offboarding checklist, media disposal policy | Annual | compliance-officer | Policy review log: each policy, reviewer, date reviewed, version bumped (or "no change" noted), signed by owner | CC1.2, CC2.2, C1.1 |
| **Q1 (Jan)** | NDA / confidentiality agreement audit — confirm all current employees and contractors have signed current-version NDA; re-sign if policy version changed | Annual | compliance-officer | Signed NDA register (name, date, document version) stored in `compliance/ndas/` | CC1.4 — closes 🔴 gap |
| **Q1 (Jan)** | Annual privacy review — re-evaluate DPIA; confirm lawful basis for each processing activity; review data retention enforcement; check for new data types requiring DPIA update | Annual | compliance-officer | Privacy review memo: DPIA version, processing activities reviewed, residual risks re-scored, sign-off date | P8.1 — closes 🔴 gap |
| **Q1 (Jan)** | Annual vendor / sub-processor security review — send security questionnaire to all sub-processors; review SOC 2 / ISO 27001 reports; re-confirm DPA status | Annual | compliance-officer | Completed questionnaire responses or current SOC 2 bridge letter for each sub-processor; gap notes for any vendor failing review | CC9.1 — closes 🔴 gap |
| **Q1 (Feb)** | Annual security awareness training — all employees and active contractors complete training; track completion | Annual | compliance-officer (content); security-engineer (delivery) | LMS or equivalent completion records: name, completion date, score if applicable; 100% completion required before proceeding to Q2 access review | CC1.4, CC2.2 — closes 🔴 gap |
| **Q1 (Mar)** | Penetration test findings review — all critical and high findings from January pentest must have documented remediation plans; medium findings triaged | Annual | security-engineer | Remediation tracking in Linear: finding ID, severity, assigned engineer, target date, closure evidence | CC7.1 |
| **Q2 (Apr)** | Quarterly access review — enumerate all active credentials (Cloudflare, Supabase, 1Password, GitHub, PostHog, Sentry, Stripe dashboard); confirm least privilege; deprovision stale accounts within 24h | Quarterly | security-engineer + compliance-officer | Access review checklist: system, account, role, last-active date, action taken (retain / deprovision), reviewer sign-off | CC6.2, CC6.3 |
| **Q2 (Apr)** | Quarterly offboarding audit — confirm any departures since last review completed full offboarding checklist (credential revocation, NDA reminder, data return) within 24h of departure | Quarterly | compliance-officer | Offboarding log: name, departure date, checklist completion date, any exceptions with justification | CC6.3 — addresses 🔴 gap |
| **Q2 (Apr)** | Q2 control effectiveness review — assess all controls in Sections 1–5 for the prior quarter; re-score any controls showing degradation | Quarterly | compliance-officer + security-engineer | Control effectiveness report: control ID, operating status (effective / degraded / failed), evidence citation, remediation assigned | CC4.2 — closes 🔴 gap |
| **Q2 (May)** | DSAR response test — submit a test DSAR as an internal user; confirm export is delivered complete and within 30 days; document end-to-end elapsed time | Annual | compliance-officer | Test DSAR ticket in Linear: submission timestamp, delivery timestamp, elapsed days, completeness checklist | P5.2 (DSAR 30-day SLA) — closes 🔴 gap |
| **Q2 (Jun)** | Media and device disposal audit — confirm all disposed devices since last review used certified data destruction; log each device by serial/asset number | Annual | compliance-officer | Device disposal log: asset ID, disposal date, destruction method, certificate reference | C1.2 — closes 🔴 gap |
| **Q3 (Jul)** | Quarterly access review | Quarterly | security-engineer + compliance-officer | Same as Q2 artifact | CC6.2, CC6.3 |
| **Q3 (Jul)** | Quarterly offboarding audit | Quarterly | compliance-officer | Same as Q2 artifact | CC6.3 |
| **Q3 (Jul)** | Q3 control effectiveness review | Quarterly | compliance-officer + security-engineer | Same as Q2 artifact | CC4.2 |
| **Q3 (Aug)** | Risk register formal review — re-score all 18 risks in Section 14; add new risks identified since last review; confirm HIGH-inherent risks have current mitigations | Annual (first instance); Quarterly (steady state) | compliance-officer + security-engineer | Updated Section 14 with new review date; Linear tickets for any newly-scored HIGH or CRITICAL risks | CC3.1, CC3.2, CC3.3 |
| **Q3 (Sep)** | Audit firm engagement check — if observation period is active, confirm auditor evidence package is on track; if pre-observation, confirm engagement timeline per Section 15.2 | Annual milestone check | compliance-officer | Email thread with audit firm confirming status; updated engagement milestone tracker | SOC 2 Type II readiness |
| **Q4 (Oct)** | Quarterly access review | Quarterly | security-engineer + compliance-officer | Same as Q2 artifact | CC6.2, CC6.3 |
| **Q4 (Oct)** | Quarterly offboarding audit | Quarterly | compliance-officer | Same as Q2 artifact | CC6.3 |
| **Q4 (Oct)** | Q4 control effectiveness review | Quarterly | compliance-officer + security-engineer | Same as Q2 artifact | CC4.2 |
| **Q4 (Nov)** | Annual training completion audit — verify 100% of employees and contractors completed security awareness training; identify and remediate any gaps before year-end | Annual | compliance-officer | Completion register cross-referenced against current headcount; gap note if anyone incomplete + remediation date | CC1.4, CC2.2 |
| **Q4 (Dec)** | Annual sub-processor list publication refresh — update `form.coach/legal/sub-processors` to reflect any additions, removals, or changes; send 30-day advance notice for any new processors taking effect in Q1 | Annual | compliance-officer | Published page with "last updated" date; 30-day notice emails sent to enterprise tenants (or confirmation that no changes occurred) | CC9.1, GDPR Art. 28(3) |
| **Q4 (Dec)** | Year-end compliance programme review — assess overall SOC 2 readiness score; update this document; plan gap remediation for following year; confirm audit firm engagement for upcoming year | Annual | compliance-officer + security-engineer | Updated SOC2_READINESS.md version; gap delta from prior year; budget line items confirmed for next year compliance activities | CC1.1, overall programme |
| **Q1 (following year)** | Annual cycle repeats — DR drill, pentest, policy review, NDA audit, privacy review, vendor review, security training | Annual | compliance-officer | Continuous evidence chain spanning the observation period | All CC, A, PI, C, P criteria |

---

### 15.2 Pre-Observation Period Readiness Checklist

The SOC 2 Type II observation clock **cannot start** until the items below are fully in place and operating. Auditors will confirm the start date; any control not yet operating at that date is out-of-scope for the observation period and must be treated as a gap finding. Complete this checklist before engaging the audit firm for the readiness assessment.

#### Legal and Policy Foundations

| # | Item | Owner | Target Date | Status |
|---|---|---|---|---|
| PRE-01 | Privacy policy published at `form.coach/privacy`; covers all FORM data categories; counsel-reviewed for EU, US, UA jurisdictions | compliance-officer | Month O-6 | 🔴 Open |
| PRE-02 | Sub-processor list published at `form.coach/legal/sub-processors`; all processors listed with DPA status | compliance-officer | Month O-6 | 🟡 Partial (Sentry DPA pending) |
| PRE-03 | Acceptable Use Policy (AUP) finalized and signed by all current team members | compliance-officer | Month O-5 | 🔴 Open |
| PRE-04 | NDA / confidentiality agreements signed by all employees and active contractors (current-version template) | compliance-officer | Month O-5 | 🔴 Open (pre-hire) |
| PRE-05 | Offboarding procedure documented and tested end-to-end (credential revocation within 24h, data return, NDA reminder) | compliance-officer + security-engineer | Month O-5 | 🔴 Open |
| PRE-06 | Media and device disposal policy published; covers remote-work scenario (personal device with FORM dev access) | compliance-officer | Month O-5 | 🔴 Open |
| PRE-07 | DPIA completed and filed for all health data processing activities (`docs/GDPR_DPIA.md` — current version confirmed) | compliance-officer | Month O-4 | 🟢 Done (v0.1, May 2026) |
| PRE-08 | DPAs confirmed and signed with all sub-processors (Anthropic, ElevenLabs, Supabase, Cloudflare, PostHog, Sentry, Stripe) | compliance-officer | Month O-4 | 🟡 Partial (Sentry pending) |
| PRE-09 | SCCs (Standard Contractual Clauses) in place for all US-EU data transfers with each sub-processor | compliance-officer | Month O-4 | 🟡 Partial |

#### Technical Controls

| # | Item | Owner | Target Date | Status |
|---|---|---|---|---|
| PRE-10 | Uptime monitoring live with ≥30 days of historical data before observation start; alert fires in <60 seconds | devops-lead | Month O-4 | 🔴 Open |
| PRE-11 | Customer-facing status page live at `status.form.coach` with 90-day history visible | devops-lead | Month O-4 | 🔴 Open |
| PRE-12 | MFA enforced on all admin surfaces: Cloudflare dashboard, Supabase, 1Password, GitHub, PostHog, Sentry | security-engineer | Month O-4 | 🟡 Partial |
| PRE-13 | Dependency scanning (Dependabot + `npm audit`) running in CI; critical CVEs fail the build | devops-lead | Month O-3 | 🟡 Partial |
| PRE-14 | Branch protection enforced: PR review required + CI pass before merge to `main` | security-engineer | Month O-3 | 🟡 Partial |
| PRE-15 | Patching SLA enforced and tracked: critical <24h, high <7d, medium <30d (Linear ticket per CVE) | security-engineer | Month O-3 | 🔴 Open |
| PRE-16 | Cookie consent banner deployed; `privacy.consent_granted` logged to audit log before any health data collection | security-engineer | Month O-3 | 🔴 Open |
| PRE-17 | Anomaly alerting live: auth failure spikes → PagerDuty P1 within 60 seconds | security-engineer | Month O-3 | 🟡 Partial |
| PRE-18 | Audit log HMAC chain running continuously with weekly cron; no chain breaks since implementation | security-engineer | Month O-1 | 🟢 Done (DEC-030) |
| PRE-19 | Backup PITR confirmed active; restore runbook tested (can achieve RTO ≤4h, RPO ≤1h) | devops-lead | Month O-1 | 🟢 Done (per SECURITY.md §10) |

#### Process Controls (must be operating — not just documented)

| # | Item | Owner | Target Date | Status |
|---|---|---|---|---|
| PRE-20 | First DR drill completed and report filed | devops-lead + security-engineer | Month O-3 | 🔴 Open |
| PRE-21 | First penetration test completed; critical and high findings remediated before observation start | security-engineer | Month O-3 | 🔴 Open |
| PRE-22 | Security awareness training first cohort completed (all current employees and contractors) | compliance-officer | Month O-2 | 🔴 Open |
| PRE-23 | First quarterly access review completed and documented | security-engineer + compliance-officer | Month O-1 | 🔴 Open |
| PRE-24 | DSAR handling procedure tested end-to-end; elapsed time ≤30 days confirmed | compliance-officer | Month O-1 | 🟡 Partial |
| PRE-25 | Continuous compliance tooling (Vanta or Drata) connected to GitHub, Cloudflare, Supabase, 1Password; controls mapped | compliance-officer | Month O-1 | 🔴 Open |
| PRE-26 | Incident response tabletop exercise completed; IRP updated with lessons | compliance-officer + security-engineer | Month O-1 | 🔴 Open |
| PRE-27 | Risk register reviewed and current (Section 14); all HIGH inherent risks have documented mitigations | compliance-officer | Month O-1 | 🟢 Done (Section 14, May 2026) |

#### Audit Firm Engagement Milestones

| Milestone | Activity | Target |
|---|---|---|
| Month O-6 | Shortlist audit firms (Prescient Assurance, Johanson Group, Sensiba San Filippo); issue RFP | compliance-officer |
| Month O-5 | Select audit firm; execute engagement letter; agree on observation period start date | compliance-officer |
| Month O-4 | Readiness assessment call with audit firm; receive gap list; prioritize remaining PRE items | compliance-officer + security-engineer |
| Month O-3 | All critical PRE items (PRE-01 through PRE-22) complete; confirm with auditor | compliance-officer |
| Month O-1 | Final pre-observation walkthrough with audit firm; observation period start date confirmed in writing | compliance-officer |
| Month O+0 | **Observation period begins.** All controls operating. Evidence collection automated. No new gaps permitted without management response. | all |
| Month O+6 | Observation period ends. Evidence package compiled. Auditor fieldwork begins. | compliance-officer |
| Month O+8 | Draft report delivered. Management response to exceptions within 10 business days. | compliance-officer |
| Month O+9 | Final SOC 2 Type II report issued. Available to enterprise customers under NDA on request. | compliance-officer |

---

### 15.3 First-Year Implementation Priority

FORM is currently a solo-founder operation. Not all compliance activities can be executed simultaneously without compromising product velocity. The table below distinguishes what must be done before the first hire from what can be deferred until the team exists to operate it.

The foundational principle: **documented policies with no operator to run them are compliance debt, not compliance**. Solo-founder controls are necessarily compensating controls — auditors will accept them during a pre-revenue readiness phase, but the observation period requires people to perform recurring activities.

#### Before First Hire (Solo Founder — Can Do Now)

These items require only the founder's time and produce durable artifacts that survive hiring:

| Priority | Item | Effort | Why It Cannot Wait |
|---|---|---|---|
| 🔴 P1 | Publish privacy policy (`form.coach/privacy`) — engage outside counsel; budget $2-5k | 2 weeks | Blocks enterprise pilots; blocks SOC 2 observation period; GDPR requirement |
| 🔴 P1 | Complete and sign DPA with Sentry; confirm EU region routing | 1 week | 🔴 Gap; active processing without valid DPA is GDPR Art. 28 violation |
| 🔴 P1 | Publish sub-processor list (`form.coach/legal/sub-processors`) | 2 days | 🔴 Gap; GDPR Art. 13 disclosure requirement; enterprise procurement asks for this on day one |
| 🔴 P1 | Document offboarding procedure (even for a team of one — template ready for first hire) | 2 days | 🔴 Gap; must exist and be tested before first hire, not after departure |
| 🔴 P1 | Document and test media/device disposal policy | 1 day | 🔴 Gap; founder's personal device has FORM production access; policy applies now |
| 🟡 P2 | Draft NDA template with employment counsel; store in `compliance/ndas/`; sign with any current contractors | 1 week | 🔴 Gap becomes blocking on first hire; zero effort to pre-draft now |
| 🟡 P2 | Implement uptime monitoring (Better Uptime or equivalent) and status page | 3 days | 🔴 Gap; without 30+ days of historical data before observation, Availability criteria cannot be evidenced |
| 🟡 P2 | Enforce MFA on all admin surfaces (Cloudflare, Supabase, GitHub, 1Password, PostHog, Sentry) | 1 day | 🟡 Gap; low effort; high audit risk if missed |
| 🟡 P2 | Implement Dependabot + `npm audit` in CI with critical CVE gate | 2 days | 🟡 Gap; founder can configure CI; evidence starts accumulating immediately |
| 🟡 P2 | Engage audit firm — issue RFP, receive quotes, select firm; no commitment to start date yet | 2 weeks | Audit firms book 6-8 weeks out; delaying engagement delays report date, not readiness |
| 🟡 P2 | Schedule DR drill date and add to calendar (even if 3 months away) | 1 day | 🔴 Gap in schedule; removes the "not scheduled" finding immediately |
| 🟢 P3 | Continuous compliance tooling evaluation (Vanta vs. Drata) — free trial; confirm GitHub + Supabase integration works | 1 week | Month O-1 dependency; evaluate now, purchase when observation nears |

#### After First Engineering or Compliance Hire (Team of 2+)

These activities require a second person because they involve independent review, training delivery, or dual-authorization controls:

| Priority | Item | Who Does It | Why It Needs a Second Person |
|---|---|---|---|
| 🔴 P1 | Security awareness training — first cohort; all employees complete before observation period | compliance-officer delivers; all employees receive | Founder cannot be the sole trainer and sole trainee; training records require independent attestation |
| 🔴 P1 | First quarterly access review — enumerate all credentials; independent second reviewer confirms deprovisions | security-engineer + compliance-officer | Access review where the reviewer is the only user is a control fiction; SOC 2 auditors require independent confirmation |
| 🔴 P1 | First DR drill — simulate failure; security-engineer runs restore; devops-lead or compliance-officer observes and documents RTO/RPO | devops-lead + security-engineer | Drill results require an independent observer to sign off; solo restores with no witness are inadmissible as drill evidence |
| 🔴 P1 | First penetration test — external firm; internal triage and remediation lead | security-engineer (point of contact); external firm | Requires an external party by definition; intern response tracking needs an assigned engineer |
| 🔴 P1 | First quarterly control effectiveness review — review Sections 1–14 operating status | compliance-officer + security-engineer | A single person assessing their own controls is a red flag for auditors; two sign-offs required |
| 🔴 P1 | Background checks for any engineering hire with production access | compliance-officer (process); third-party provider | Requires a hire to exist; policy must be ready at offer stage, not post-start |
| 🟡 P2 | Break-glass dual-authorization — currently compensating control (founder sole approver); first hire enables real two-person rule | security-engineer + compliance-officer | SOC 2 CC5 prefers dual authorization; hire makes this possible for real |
| 🟡 P2 | Incident response tabletop exercise — simulate SEV-0 breach; test communication templates; run postmortem | compliance-officer (facilitates); security-engineer + any other team members | Tabletop with one person is a monologue; requires at least two participants for meaningful findings |
| 🟡 P2 | Change management policy enforcement — PR review mandatory (currently: self-merge possible with founder alone) | security-engineer (reviewer) + founder (author) | Cannot review your own PR as a compensating control for change management; requires at least one other engineer |
| 🟢 P3 | Employee security training programme — ongoing, LMS-delivered, tracked annually | compliance-officer (content); LMS (delivery) | Programme design can start now; rollout waits for first employee to receive it |
| 🟢 P3 | Separation of duties — production deploy approval separate from code author | devops-lead + security-engineer | Structural control; fully implementable only when team exceeds one person |

#### Solo-Founder Compensating Controls (Documented Acceptance)

The following controls are formally impossible with one person. They are documented here as **compensating control acceptances** — auditors reviewing a pre-revenue startup in a Type I readiness context will accept these with management attestation, provided the compensating controls are clearly documented and the team explicitly acknowledges the gap. These acceptances expire the moment the relevant second person is hired.

| Gap | Compensating Control | Acceptance Condition |
|---|---|---|
| Independent access review | Founder reviews own access; all systems documented in `compliance/access-review/`; monthly review log shows consistent execution | Expires at first engineering hire |
| Two-person rule for break-glass | Founder is sole break-glass holder; all break-glass access logged with mandatory `incident_id` before query executes; HMAC log tamper-evident | Expires at first engineering hire; real dual-auth required within 30 days of hire |
| Security awareness training (self) | Founder documents completion of equivalent external training (e.g., SANS Securing the Human, Google Security Training); certificate stored | Expires when first employee needs a training cohort |
| Independent DR drill observation | Founder runs and documents drill solo; drill report includes screenshot evidence of PITR timestamps and RTO measurement; accepted as compensating during pre-observation phase | Expires at observation period start — must have independent observer by Month O-1 |
| PR self-review | All self-merged PRs flagged in Linear with `solo-founder-compensating-control` tag; security-engineer reviews retrospectively at quarterly access review | Expires at first engineering hire; branch protection `required_reviewers: 1` enforced from day one of second engineer |

---

**v0.5 additions (Section 15):** Annual Compliance Calendar (master table, 28 recurring activities), Pre-Observation Period Readiness Checklist (27 items, PRE-01 through PRE-27), First-Year Implementation Priority (solo-founder vs. post-hire split, compensating control acceptances). Addresses 🔴 gaps: security training scheduled, quarterly control effectiveness review scheduled, annual vendor security review scheduled, DR test scheduled, annual privacy review scheduled, DSAR response test scheduled, offboarding audit scheduled, media/device disposal audit scheduled. Critical gaps closed by schedule: 8. Remaining critical gaps: 1 (NDA template — pre-drafted, executes at first hire). Readiness score: ~45% → 48% (calendar evidences recurring process intent; score moves fully on first execution).**

---

## Open Items for compliance-officer

- [ ] Engage audit firm (shortlist: Prescient Assurance, Johanson Group, Sensiba San Filippo) — PRE-milestone Month O-6
- [x] Draft privacy policy — `docs/PRIVACY_POLICY.md` v0.1-draft (May 2026, pre-legal-review)
- [x] Complete DPIA for health data processing — `docs/GDPR_DPIA.md` v0.1 (May 2026)
- [x] Formalize risk register — Section 14 (May 2026)
- [x] Define data classification policy tiers — Section 13 (May 2026)
- [x] Schedule first DR drill date — Q1 January (Section 15.1 master calendar)
- [ ] Confirm Sentry DPA status — PRE-08; blocking full sub-processor register
- [ ] Implement uptime monitoring + status page — PRE-10, PRE-11
- [ ] Complete first penetration test — PRE-21; required before observation period
- [ ] Security awareness training first cohort — PRE-22; requires first hire
- [ ] Complete first quarterly access review — PRE-23

---

**v0.5 · травень 2026 · owner: compliance-officer + security-engineer + enterprise-architect**
**Review cadence: quarterly. Next review: серпень 2026.**

*v0.2 additions: Sub-Processor Register (CC9, GDPR Art. 28), Complementary User Entity Controls (CUECs), Common Security Questionnaire Responses (CAIQ/SIG Lite pre-answers).*
*v0.3 additions: Section 13 — Data Classification Policy (four-tier: Public / Internal / Confidential / Restricted). Closes SOC 2 gap C1.1. Critical gaps: 11 → 10. Controls in place: 22 → 23.*
*v0.4 additions: Section 14 — Formal Risk Register (18 risks across 6 categories: Security, Availability, Processing Integrity, Confidentiality, Privacy, Vendor/Operational). L×S scoring with residual scores and named owners. Closes CC3 gap (formal risk assessment documented). Updated P3 DPIA status to ✅ Done. Critical gaps: 10 → 9. Partial: 22 → 21. Controls in place: 23 → 25. Readiness: 42% → 45%.*
*v0.5 additions (two-step catch-up): (a) `docs/PRIVACY_POLICY.md` v0.1-draft shipped (v0.55.0 CHANGELOG) — closes P1.1, P1.2, CC9.2+P6.1; critical gaps: 9 → 6; controls in place: 25 → 28; readiness: 45% → ~51%. (b) Section 15 — Annual Compliance Calendar: 12-month master calendar (16 recurring activities, all mapped to SOC 2 controls + evidence artifacts), Pre-Observation Period Readiness Checklist (27 PRE items with status), First-Year Implementation Priority matrix (solo-founder vs. post-hire split, compensating controls documented). Moves 7 gaps from 🔴 Gap → 🟡 Partial (security training scheduled Q1-Feb, vendor review Q1-Jan, DR drill Q1-Jan, privacy review Q1-Jan, offboarding quarterly cadence, media disposal Q2-Jun, control effectiveness review quarterly). Critical gaps: 6 → 4 (security training and offboarding: schedule + owner + evidence defined → 🟡 Partial). Partial: 21 → 28. Controls in place: 28 (unchanged — scheduled but not yet executed). Readiness: ~51% → ~55%.*
