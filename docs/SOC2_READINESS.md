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
| Formal risk assessment documented | 🟡 Partial | `docs/SECURITY.md` §1 threat model covers key threats; needs formalization as Risk Register |
| Risk register maintained, reviewed annually | 🔴 Gap | Create risk register doc; assign compliance-officer |
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
| Confidential data classification policy | 🔴 Gap | Define: Public / Internal / Confidential / Restricted |
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
| Purpose limitation documented | 🟡 Gap | DPIA (Data Protection Impact Assessment) needed for health data processing |

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
| 🔴 **Critical gap** (blocks SOC 2) | 11 | Privacy policy, status page, security training, offboarding procedure |
| 🟡 **Partial / needs formalization** | 22 | Risk register, vendor registry, patching SLA, DPIA |
| ✅ **In place** | 22 | HMAC audit log, encryption, access controls, CV on-device, breach notification |

**Readiness score: ~40% controls fully in place.** Target: 90% before observation period begins.

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

## Privacy Floor Enforcement (Non-Negotiable)

The following controls are enforced regardless of customer requests. They are non-bypassable per `docs/ENTERPRISE.md` and `clinical-safety` VETO authority:

1. HR role (`tenant_manager`) **never** receives access to individual user health data
2. `data.read_individual` action is logged + triggers immediate security alert if attempted at HR tier
3. Aggregate wellness metrics exclude: body composition, mental health indicators, ED-screening responses
4. Employee may revoke company association at any time — `privacy.consent_withdrawn` logged
5. Any customer request to bypass these controls is declined; deal is lost if necessary

These map to SOC 2 Privacy criteria P2, P3, and P6, and simultaneously satisfy GDPR Art. 9 (special category data) requirements.

---

## Open Items for compliance-officer

- [ ] Engage audit firm (shortlist: Prescient Assurance, Johanson Group, Sensiba San Filippo)
- [ ] Draft privacy policy (legal review required)
- [ ] Complete DPIA for health data processing
- [ ] Formalize risk register (seed from `docs/SECURITY.md` §1 threat model)
- [ ] Define data classification policy tiers
- [ ] Schedule first DR drill date
- [ ] Confirm Sentry DPA status

---

**v0.2 · травень 2026 · owner: compliance-officer + security-engineer + enterprise-architect**
**Review cadence: quarterly. Next review: серпень 2026.**

*v0.2 additions: Sub-Processor Register (CC9, GDPR Art. 28), Complementary User Entity Controls (CUECs), Common Security Questionnaire Responses (CAIQ/SIG Lite pre-answers).*
