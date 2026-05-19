# FORM · SOC 2 Type II Readiness v1.3

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
| Quarterly control effectiveness review | 🟡 Partial | Access-control effectiveness integrated into quarterly access review — §23.4 Step 5; full cross-domain review expands post-hire |

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
| Quarterly access review | 🟡 Partial | Procedure documented — §23; first review pending PRE-23 |
| Physical security of infrastructure | ✅ N/A | Cloud-only (Cloudflare, Supabase) — inherit provider controls |

### CC7 — System Operations

| Control | Status | Evidence |
|---|---|---|
| Incident detection and response | ✅ Done | `docs/SECURITY.md` §7 severity levels (SEV-0 through SEV-3) + runbook |
| Vulnerability scanning (dependencies) | 🟡 Gap | Dependabot / `npm audit` planned; implement in CI |
| Patching SLA defined | 🟡 Gap | Define: critical patches <24h, high <7d, medium <30d |
| System health monitoring | 🟡 Partial | Architecture defined (§20); Better Stack Statuspage implementation checklist pending |
| External penetration test | 🟡 Partial | Program defined — Section 16; first test pending PRE-21 |

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
| Uptime monitoring with historical data | 🟡 Partial | Architecture defined (§20); Better Stack provisioning = PRE-10 |
| Customer-facing status page | 🟡 Partial | Architecture defined (§20); CNAME + 6 components specified; implementation = PRE-11 |
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
| 🔴 **Critical gap** (blocks SOC 2) | 1 | Cookie banner / consent management (PRE-16 — required before health data collection begins) |
| 🟡 **Partial / needs formalization** | 33 | Offboarding procedure, quarterly access review (§23 — first execution pending), vendor security review (§17), DR drill (Q1-Jan scheduled), CC7.1–CC7.2 (pentest program defined, execution pending), patching SLA, DSAR SLA, phishing simulation (platform pending), sub-processor list publication (§20 — implementation checklist complete), annual privacy review (§15.1 Q1-Jan scheduled), change management CI gates (§21) |
| ✅ **In place** | 31 | HMAC audit log, encryption, RLS tenant isolation, access controls, CV on-device, breach notification, data classification policy (§13), DPIA (docs/GDPR_DPIA.md), formal risk register (§14), compliance calendar (§15), penetration test program (§16), privacy policy (docs/PRIVACY_POLICY.md), security awareness training programme (§22), BCP/DR architecture (§18), cold storage backup architecture (§19), status page architecture (§20), change management policy (§21) |

**Readiness score: ~71% controls fully in place or formally designed** (v1.3; see version history at bottom). Target: 90% before observation period begins.

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
| PRE-11 | Customer-facing status page live at `status.form.coach` with 90-day history visible | devops-lead | Month O-4 | 🟡 Partial (architecture + checklist: §20) |
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
| PRE-21 | First penetration test completed; critical and high findings remediated before observation start | security-engineer | Month O-3 | 🟡 Partial (program defined — Section 16; firm selection + execution pending) |
| PRE-22 | Security awareness training first cohort completed (all current employees and contractors) | compliance-officer | Month O-2 | 🔴 Open |
| PRE-23 | First quarterly access review completed and documented | security-engineer + compliance-officer | Month O-1 | 🟡 Partial (procedure documented — §23; first execution pending) |
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

## 16. Penetration Test Program

> Owner: `security-engineer` + `compliance-officer`. SOC 2 criteria closed: CC7.1 (threat identification), CC7.2 (vulnerability management), CC9.2 (vendor risk).
> Reference: DEC-030 (HMAC-chained audit log), `docs/AUDIT_LOG_SCHEMA.md`, `docs/ENTERPRISE.md`, `docs/INCIDENT_RESPONSE.md`.

### 16.1 Purpose

The penetration test program formally identifies security vulnerabilities in FORM's production systems before an attacker does. It produces auditor-grade evidence for CC7 (System Operations) and demonstrates due care to enterprise customers. Results feed directly into the risk register (Section 14) and the incident response program (`docs/INCIDENT_RESPONSE.md`).

This section defines the program structure. The first execution maps to PRE-21 in the readiness checklist (Section 15.2). Once executed, the report and remediation evidence are stored as durable SOC 2 artifacts.

### 16.2 Scope

#### In scope

| Surface | Methodology | Notes |
|---|---|---|
| **API layer** (Cloudflare Workers) | Black-box + grey-box | Authenticated endpoints tested with valid JWT; unauthenticated endpoints black-box |
| **Authentication flows** | Grey-box | OIDC/SAML SP-initiated and IdP-initiated, magic link OTP, MFA bypass attempts, session fixation |
| **Admin dashboard** (`admin-dashboard.html`) | Grey-box | Role escalation, insecure direct object reference (IDOR), XSS |
| **Enterprise SSO/SCIM endpoints** | Grey-box | Certificate replay, SCIM provisioning injection, group mapping bypass |
| **RLS / data isolation** | Grey-box via API | Cross-tenant data access via crafted API requests; direct DB access not included (see below) |
| **Mobile apps** (iOS + Android) | Binary + traffic analysis | Certificate pinning bypass, local storage secrets, OWASP MASVS L1 checklist |
| **Cloudflare edge configuration** | Black-box | WAF bypass attempts, rate-limit evasion, header injection |

#### Explicitly out of scope

| Surface | Reason |
|---|---|
| Supabase internal infrastructure | Third-party; covered by Supabase's own SOC 2 report |
| Anthropic API | Third-party; covered by Anthropic's own security posture |
| ElevenLabs, PostHog, Sentry, Stripe | Third-party sub-processors |
| Direct database access (Postgres) | Only `form_api` role exposed via API; direct Postgres port not reachable from internet |
| Denial-of-service (DoS/DDoS) | Not permitted; destructive; covered by Cloudflare WAF operational controls |
| Social engineering of FORM team | Out of scope at current company size; revisit at >10 employees |
| Physical access | Remote-first; no physical infrastructure owned by FORM |

#### Scope change process

Any expansion of scope (e.g., adding a new product surface such as a browser extension or ClickHouse instance) requires a signed scope addendum before the test window opens. Changes discovered mid-test that reveal an adjacent attack surface are reported immediately to `security-engineer` with a decision on whether to extend scope or note as an out-of-scope observation.

### 16.3 Methodology

FORM uses industry-standard frameworks as the baseline for test coverage:

| Framework | Application |
|---|---|
| **OWASP WSTG** (Web Security Testing Guide) v4.2 | API and web surface coverage checklist |
| **OWASP ASVS** (Application Security Verification Standard) Level 2 | Authentication, session management, data protection |
| **OWASP MASVS** L1 + L2 (Mobile Application Security Verification Standard) | iOS and Android apps |
| **PTES** (Penetration Testing Execution Standard) | Engagement structure, reporting, evidence preservation |
| **CWE Top 25** | Prioritised weakness categories; all Top 25 checked |

FORM does not prescribe a specific CVSS scoring tool; the testing firm uses CVSS v3.1 as the baseline severity anchor, supplemented by context (FORM health data makes confidentiality impact higher than default for many vectors).

### 16.4 Test Types

| Test type | Frequency | Trigger |
|---|---|---|
| **Annual external penetration test** | Once per year (Q1) | Calendar — maps to Compliance Calendar §15.1 |
| **Re-test of prior findings** | Within 30 days of remediation closure | After any Critical or High finding is remediated; confirms fix holds |
| **Architecture-change test** | Within 60 days of major change | SSO/SCIM launch, new database region, new AI provider, significant API restructure |
| **Pre-observation readiness test** | One-time | Before SOC 2 observation period starts (PRE-21) |
| **Post-P0-incident targeted test** | Within 90 days of P0 | After any P0 security incident; scope limited to affected surface |

### 16.5 Testing Firm Selection Criteria

Firms are evaluated against the following minimum bar before engagement:

| Criterion | Requirement |
|---|---|
| Certification | CREST-accredited firm, or team lead holds OSCP + minimum one of: OSCE, OSWE, GWAPT, BSCP |
| Independence | No current or former employment relationship with FORM; no financial relationship with any FORM sub-processor |
| Healthcare data experience | Demonstrated experience testing applications that handle health or biometric data (HIPAA-adjacent preferred) |
| NDA | Mutual NDA executed before any test credentials, architecture diagrams, or API specs are shared |
| Insurance | Professional indemnity ≥ $2M USD; cyber liability ≥ $1M USD |
| Report format | Structured report required: executive summary, finding table (ID, title, CVSS, description, evidence, reproduction steps, remediation), methodology appendix |

**Current shortlist (for first engagement):** Bishop Fox, NCC Group, Cobalt.io (continuous pentest platform — evaluate for Year 2), Cure53.

`security-engineer` owns vendor selection and issues the statement of work. `compliance-officer` countersigns.

### 16.6 Finding Severity and Remediation SLAs

FORM uses CVSS v3.1 base score as the primary severity signal, adjusted upward one tier when the finding directly exposes health data (GDPR Art. 9 special category) or breaks tenant isolation.

| Severity | CVSS v3.1 Range | Remediation SLA | Escalation |
|---|---|---|---|
| **Critical** | 9.0–10.0 | 24 hours from report delivery | Incident Commander paged immediately — treated as P1 per `docs/INCIDENT_RESPONSE.md`; `compliance-officer` notified |
| **High** | 7.0–8.9 | 7 calendar days | `security-engineer` + `devops-lead` assigned; status update daily |
| **Medium** | 4.0–6.9 | 30 calendar days | Linear ticket; `security-engineer` triage within 48h |
| **Low** | 0.1–3.9 | 90 calendar days or next quarterly review (whichever comes first) | Batch-triaged at next quarterly access review |
| **Informational** | N/A | Next quarterly review | Documented; no remediation SLA; inform relevant owner |

**Health-data severity uplift:** Any finding that could expose `health_profiles`, `meal_logs`, `workouts`, or any field tagged `data_class = 'restricted'` (per `docs/AUDIT_LOG_SCHEMA.md` taxonomy) is escalated one tier. A Medium finding with health-data exposure is treated as High; a High with health-data exposure is treated as Critical.

**Tenant isolation uplift:** Any finding that could allow one tenant's `form_api` session to read another tenant's rows (RLS bypass) is automatically Critical regardless of CVSS base score. This maps to Risk SR-02 in the risk register (Section 14) and would trigger a P0 incident declaration per `docs/INCIDENT_RESPONSE.md` §1.2 if discovered post-launch.

### 16.7 Remediation Tracking

All findings are entered into Linear immediately upon report receipt. No finding is closed without evidence.

| Step | Action | Owner |
|---|---|---|
| **1. Intake** | `security-engineer` reads full report within 24h of delivery; creates one Linear ticket per finding with label `pentest-finding` and `severity-{critical|high|medium|low}` | security-engineer |
| **2. Triage** | Within 48h: assign ticket to responsible engineer; set due date per SLA table above; link to affected code path or infra component | security-engineer |
| **3. Remediation** | Assigned engineer implements fix; creates PR with description referencing finding ID (e.g., `PT-2026-01-007`) | Assigned engineer |
| **4. Review** | PR reviewed by `security-engineer`; cannot self-review if the assignee is the same person — use `compliance-officer` as second reviewer during solo-founder phase | security-engineer |
| **5. Re-test** | For Critical and High findings: provide reproduction steps to testing firm; receive confirmation that the fix holds before closing ticket | security-engineer + testing firm |
| **6. Evidence filing** | Close Linear ticket with link to: merged PR, re-test confirmation (email or report addendum), deployment evidence (EAS build ID or Cloudflare deploy hash) | security-engineer |
| **7. SOC 2 package** | `compliance-officer` compiles closed-ticket evidence into `compliance/pentest/YYYY/remediation-evidence.md` before end of remediation window | compliance-officer |

Linear query to see all open pentest findings: `label:pentest-finding AND NOT state:done`. This query is reviewed at every quarterly access review.

### 16.8 Report Format and Customer Disclosure

| Audience | What They Receive | Conditions |
|---|---|---|
| `security-engineer` + `compliance-officer` + founder | Full penetration test report (all findings, evidence, methodology) | Immediately upon delivery from testing firm |
| Enterprise customer (security review, >$50k ACV) | Executive summary: finding counts by severity, SLA compliance, methodology summary, attestation of Critical/High remediation | Signed mutual NDA; `compliance-officer` approval; redacted to remove reproduction steps that could be exploited by the customer |
| Enterprise customer (<$50k ACV, standard) | Attestation letter: "FORM completed an annual penetration test in [month]. All Critical and High findings were remediated within SLA. A report is available to qualified customers under NDA." | `compliance-officer` approval |
| Public / unauthenticated | No disclosure. | N/A |

FORM does not participate in coordinated vulnerability disclosure programs at this stage. Responsible disclosure requests from external parties are routed to `security@form.coach` and triaged by `security-engineer` within 24 hours.

**No AI training use of pentest reports.** Reports contain architectural details and exploitation techniques that must not be used as LLM training data. All reports stored in the private `form-compliance` repo with branch protection; not in the public `rbit27/form` repo.

### 16.9 SOC 2 Evidence Package (per annual test)

The following artifacts constitute the complete SOC 2 evidence package for CC7.1 and CC7.2. They are filed to `compliance/pentest/YYYY/` in the private compliance repo immediately after remediation is complete.

| Artifact | Description | Required for Audit |
|---|---|---|
| `engagement-letter.pdf` | Signed SOW + NDA with testing firm | Yes |
| `report-full.pdf` | Complete findings report from firm | Yes |
| `report-exec-summary.pdf` | Redacted executive summary (customer-shareable version) | Yes |
| `remediation-evidence.md` | Linear ticket IDs, PR links, re-test confirmations for all Critical + High findings | Yes |
| `attestation-sign-off.md` | `security-engineer` written confirmation that all Critical + High findings are remediated; signed by date | Yes |
| `open-accepted-risks.md` | Any Medium or Low findings accepted as residual risk (with justification, risk-owner sign-off, and reference to risk register entry in Section 14) | If applicable |
| `hmac-chain-verification.txt` | Output of HMAC chain integrity cron run on the date pentest window closes — confirms no audit log tampering during the test (per DEC-030 and `docs/AUDIT_LOG_SCHEMA.md` §6) | Yes |

The HMAC chain verification is a FORM-specific control artifact: it ensures that if a tester discovered a log-tampering vector during the engagement, the chain break would have been detected automatically. No chain break during the pentest window = the audit log is intact evidence. See `docs/AUDIT_LOG_SCHEMA.md` for chain verification procedure.

### 16.10 Connection to Enterprise Commitments

`docs/ENTERPRISE.md` §7 lists FORM's hard commitments to enterprise customers. Penetration testing supports these commitments:

| Enterprise commitment | Pentest role |
|---|---|
| "Annual external penetration test; executive summary on request" | This section defines what "annual external penetration test" means operationally |
| "99.9% uptime SLA with P0 < 1h response" | Pentest validates that no single exploitable vulnerability can take down the platform |
| "Tenant isolation: your data is never accessible to other tenants" | Pentest scope includes RLS bypass attempts via API; Critical uplift for any isolation finding |
| "No HR individual data visibility" | Pentest validates that the privacy floor enforcement (Section 6, `docs/DATA_MODEL.md` §6) cannot be bypassed via an unauthenticated or privilege-escalated path |

### 16.11 Gap Closure Status

| SOC 2 Control | Pre-§16 Status | Post-§16 Status | Remaining action |
|---|---|---|---|
| CC7.1 — threat and vulnerability identification | 🔴 Gap | 🟡 Partial | First test execution + report |
| CC7.2 — vulnerability management (remediation tracking) | 🔴 Gap | 🟡 Partial | First test execution + Linear tickets |
| CC9.2 — vendor management (security review of service providers) | 🟡 Partial | 🟡 Partial | Testing firm DPA + insurance confirmation |
| PRE-21 — first penetration test completed | 🔴 Open | 🟡 Partial | Firm selection → execution → remediation |

---

## 17. Vendor Security Review Process

> Owner: `compliance-officer` + `security-engineer`. Review: annually each January and on any new vendor addition.
> SOC 2 evidence: CC9.1 (vendor management programme), CC9.2 (security review of service providers), CC9.3 (vendor contract requirements), P8.1 (privacy enforcement with third parties).
> This section closes two documented 🔴 Gaps: "Vendor security review process" and "Annual vendor security review."

---

### 17.1 Purpose and Scope

#### Purpose

SOC 2 Trust Service Criterion CC9.2 requires FORM to assess and monitor the risks associated with third-party vendors who have access to, process, or could affect the security and availability of FORM systems and data. This section defines the formal programme that satisfies CC9.2 and supplements the sub-processor register in §13.

A documented vendor security review process is also required evidence for:

- GDPR Art. 28 (processor due diligence obligation on the controller)
- GDPR Art. 32 (appropriate technical and organisational measures, assessed across the supply chain)
- CCPA / CPRA § 1798.100 (service provider contracts and purpose limitation)
- Ukraine Law on Personal Data Protection Art. 21 (processor requirements)

#### In Scope

| Category | Examples |
|---|---|
| Sub-processors with access to personal data | Supabase, Anthropic, ElevenLabs, PostHog, Sentry, Stripe, Cloudflare |
| Infrastructure and availability dependencies | Better Uptime, PagerDuty |
| Compliance and operational tooling with privileged access to FORM systems | Vanta / Drata, Linear (if connected to production telemetry) |
| Any future vendor that will process Restricted or Confidential data (§13 tiers) | Assessed before onboarding |

#### Out of Scope

- Open-source libraries (covered by dependency scanning — §15.1, PRE-13, Dependabot)
- Vendors with no access to FORM systems or data and no contractual relationship (e.g., SaaS the team uses for internal productivity with no FORM data integration)
- AWS (covered transitively via Supabase DPA — see §13 notes)

---

### 17.2 Vendor Risk Classification

Vendors are assigned to one of three tiers at onboarding. Tier assignment determines review frequency and minimum evidence requirements. Tier is re-evaluated annually and on any material change to the vendor relationship.

| Tier | Definition | Examples | Review Frequency |
|---|---|---|---|
| **Critical** | Processes Restricted data (GDPR Art. 9 health data, §13 tier) OR is a single point of failure for platform availability | Supabase (all user and health data), Anthropic (LLM inference for Victor — coaching session content), ElevenLabs (voice coaching) | Annual full review + quarterly status check |
| **High** | Processes Confidential data OR provides security-relevant tooling (monitoring, error tracking, compliance automation) | PostHog (product analytics), Sentry (error monitoring), Cloudflare (CDN + WAF + Workers), Stripe (billing), Vanta / Drata (compliance tooling with system access) | Annual full review |
| **Standard** | Operational tooling with no direct access to user data; availability impact limited to internal workflows | Better Uptime (uptime checks — no user data), PagerDuty (alerting routing — no user data), Linear (issue tracker — no production data by default) | Annual self-attestation; full review every two years or on scope change |

**Tier promotion rule:** Any Standard vendor that is granted access to Confidential or Restricted data is automatically promoted to High or Critical respectively. The compliance-officer must be notified before any such access grant.

---

### 17.3 Initial Vendor Assessment (Pre-Approval)

No new vendor that will access Confidential or Restricted data may be onboarded without completing this checklist. The compliance-officer holds approval authority. Steps must be completed in sequence; approval at each gate is required before proceeding.

**Step 1 — Security questionnaire**

Send the FORM vendor security questionnaire (template at `compliance/vendor-review/questionnaire-template.md`) to the vendor's security or legal contact. Minimum required responses:

- Encryption at rest and in transit (specify standards)
- Access control model (RBAC, MFA enforcement, privileged access management)
- Incident notification SLA (target: ≤24h for breach affecting FORM data)
- Penetration test cadence and willingness to share executive summary on request
- Data deletion / return procedure on contract termination
- Subprocessors the vendor uses (for GDPR Art. 28(4) chain-of-responsibility)
- Data residency options (EU residency required for EU-resident user data)

For Critical vendors: questionnaire must be completed. No waiver.
For High vendors: questionnaire or current SOC 2 Type II / ISO 27001 report accepted in lieu.
For Standard vendors: self-attestation by vendor's authorised signatory accepted.

**Step 2 — Certification review**

Review the vendor's current security certification. Acceptable evidence (in descending preference):

1. SOC 2 Type II report (issued within the last 12 months) — review the opinion, any exceptions noted, and whether the service scope covers the services FORM will use
2. ISO 27001 certificate (current, in-scope certification body listed) — confirm the certificate covers the specific service
3. SOC 2 Type I report (point-in-time) — acceptable for Standard tier only; flag as a gap for Critical and High
4. Vendor-completed questionnaire with specific technical controls described — Critical and High only if no SOC 2 / ISO cert exists; requires compliance-officer sign-off

Document: certification type, issuing body, report date, expiry or next audit, any noted exceptions relevant to FORM's use case.

**Step 3 — DPA review**

Draft or obtain the Data Processing Agreement before any data flows begin.

- For Critical and High vendors: DPA must be executed before a production connection is established. Standard EU/UK DPA template acceptable; FORM counsel review required if the vendor's template is non-standard.
- SCCs (Standard Contractual Clauses, EU Commission 2021/914) must be incorporated for any US-based vendor receiving EU personal data.
- DPA must include: purpose limitation, security obligations, subprocessor change notification (30 days), audit rights, data deletion on termination, breach notification ≤24h.
- File the signed DPA to `compliance/dpa/VENDOR-NAME-DPA-YYYY.pdf` in the private compliance repo.

**Step 4 — 30-day customer notice (for new sub-processors)**

If the vendor will be added to the sub-processor register (§13), GDPR Art. 28(2) and FORM's enterprise contracts require 30 days advance written notice to enterprise customers before the sub-processor begins processing their data. This step runs in parallel with Steps 1–3 to avoid delaying onboarding unnecessarily.

- Draft the sub-processor addition notice: vendor name, HQ location, purpose, data categories, region, effective date.
- Send to all enterprise tenants via the mechanism specified in their DPA (email to security contact + in-app notification).
- Log the notice date, recipient list, and any objections received to `compliance/subprocessor-notices/YYYY-MM-VENDOR.md`.
- If an enterprise customer objects: escalate to compliance-officer and legal counsel before proceeding. Do not add the sub-processor for that tenant until the objection is resolved.

**Step 5 — Approval gate**

The compliance-officer reviews the completed questionnaire, certification evidence, and DPA before approving onboarding. Approval is documented in the Vendor Risk Registry (§17.4) as a dated entry with the reviewer's name.

Approval conditions:

| Condition | Required action before approval |
|---|---|
| Missing DPA | DPA must be executed. No waiver. |
| No SOC 2 / ISO cert (Critical or High vendor) | Compensating control documented in registry; reviewed quarterly; plan to obtain cert or migrate to alternative vendor within 12 months |
| Unresolved critical or high findings in pentest or SOC 2 exceptions | Vendor must provide written remediation plan with dates before approval |
| Data residency mismatch (EU user data, no EU region) | Either vendor enables EU region or EU user data is excluded from that vendor contractually |

---

### 17.4 Vendor Risk Registry

The registry below is the authoritative record of all vendors in scope. It is updated within 5 business days of any change (new vendor, DPA executed, cert renewed, review completed). The registry is filed in `compliance/vendor-registry/VENDOR-REGISTRY.md` (private compliance repo); this section is the canonical copy in the SOC 2 readiness document.

**Last updated:** May 2026. Next full review: January 2027 (§15.1 annual vendor review).

| Vendor | Tier | Data Categories | DPA Status | Latest Cert | Cert Expiry / Next Audit | Next Review | Risk Score | Owner |
|---|---|---|---|---|---|---|---|---|
| **Supabase** | Critical | Restricted (health metrics, coaching sessions, clinical flags), Confidential (user identity, tenant config, audit logs), all user data | ✅ Signed (SCC Module 2 included) | SOC 2 Type II | Check Supabase Trust Portal | Jan 2027 | 🟢 Low | security-engineer |
| **Anthropic** | Critical | Confidential (pseudonymised exercise context — no PII in prompts by design; see §13 + SECURITY.md §5) | ✅ Signed (SCC Module 2 included) | SOC 2 Type II | Check Anthropic Trust Portal | Jan 2027 | 🟢 Low | security-engineer |
| **ElevenLabs** | Critical | Confidential (coaching text cues — no user PII by design) | ✅ Signed (SCC Module 2 included) | SOC 2 Type II | Check ElevenLabs Trust Portal | Jan 2027 | 🟢 Low | security-engineer |
| **Cloudflare** | High | Confidential (IP addresses, request metadata — no health data in Workers) | ✅ Enterprise DPA (SCC Module 2 included) | SOC 2 Type II + ISO 27001 | Check Cloudflare Trust Hub | Jan 2027 | 🟢 Low | security-engineer |
| **PostHog** | High | Confidential (anonymous event data, `analytics_opt_out` flag — no PII by configuration) | ✅ Signed (SCC Module 2 included) | SOC 2 Type II | Check PostHog Trust Portal | Jan 2027 | 🟢 Low | compliance-officer |
| **Stripe** | High | Confidential (billing contact name, company, invoice data — no health data) | ✅ Signed (SCC Module 2 included) | PCI DSS Level 1 + SOC 2 Type II | Check Stripe Trust Portal | Jan 2027 | 🟢 Low | compliance-officer |
| **Sentry (Functional Software)** | High | Confidential (error stack traces, anonymised user IDs — health data scrubbed at SDK layer as compensating control) | 🟡 In progress (SCC Module 2 pending DPA signature) | SOC 2 Type II | Check Sentry Trust Portal | On DPA execution; Jan 2027 at latest | 🟡 Medium | compliance-officer |
| **Better Uptime** | Standard | None (uptime check results only — no user data transmitted) | Not required (no personal data) | Vendor self-attestation | Annual | Jan 2027 | 🟢 Low | devops-lead |
| **PagerDuty** | Standard | None (alert routing — no FORM user data in payloads by configuration) | Not required (no personal data) | SOC 2 Type II | Check PagerDuty Trust Portal | Jan 2027 | 🟢 Low | devops-lead |
| **Linear** | Standard | Internal (issue titles, descriptions — no production user data by default) | Not required (no personal data; enforce no-production-data policy) | SOC 2 Type II | Check Linear Trust Portal | Jan 2027 | 🟢 Low | security-engineer |
| **Vanta / Drata** | High | Confidential (read access to GitHub, Cloudflare, Supabase, 1Password for evidence collection) | Required before activation — obtain DPA before connecting to production systems | SOC 2 Type II | Check vendor Trust Portal | Before activation | 🟡 Medium (pre-activation — no connection yet) | compliance-officer |

**Registry notes:**

- Sentry: compensating control in place (PII scrubbed from Sentry payloads at the SDK `beforeSend` hook). Risk score remains 🟡 Medium until DPA is executed (tracking: VR-02 in §14, PRE-08). Compliance-officer must confirm DPA execution within 60 days of this document version.
- Vanta / Drata: DPA and security review must be completed before connecting compliance tooling to production. The compliance-officer is the blocking approver for this activation.
- AWS: covered transitively through the Supabase DPA (Supabase operates on AWS infrastructure); not listed separately per §13 convention.

---

### 17.5 Annual Review Process

The annual vendor review runs every January per the §15.1 compliance calendar. The five steps below must be completed in sequence. All evidence artifacts are filed to `compliance/vendor-review/YYYY/` in the private compliance repo by 31 January each year to ensure evidence is available for the SOC 2 observation period.

**Step 1 — Registry currency check (complete by January 5)**

Verify the Vendor Risk Registry (§17.4) reflects all current vendors. Confirm:
- No new sub-processors were added in the prior year without completing §17.3
- Any vendors removed or contracted-terminated since the last review are archived (see §17.8)
- Tier assignments are still accurate given current data flows

Evidence artifact: `compliance/vendor-review/YYYY/01-registry-currency-check.md` — table of vendors reviewed, current tier, tier change if any, reviewer sign-off.

**Step 2 — Certification and SOC 2 report refresh (complete by January 15)**

For each Critical and High vendor, obtain the current SOC 2 Type II report or ISO 27001 certificate:
- Request bridge letters from vendors whose annual report period has not yet closed
- Review each report for: opinion type (unqualified vs. qualified), exceptions noted, whether exceptions affect controls relevant to FORM's use case
- Note any exceptions in the registry Risk Score column and escalate to security-engineer if any Critical vendor has a material exception

Evidence artifact: `compliance/vendor-review/YYYY/02-cert-refresh/VENDOR-NAME-cert-YYYY.pdf` (one file per vendor) + `02-cert-review-summary.md` (findings table).

**Step 3 — DPA status confirmation (complete by January 20)**

For each vendor in the registry, confirm the DPA is current and covers the current scope of data processing:
- If the vendor has updated their DPA terms since last signature: review changes; re-execute if material changes affect FORM's obligations
- Confirm SCCs have not been superseded by new EU Commission guidance
- Confirm Sentry DPA status; if still unsigned, escalate to founder with a hard deadline

Evidence artifact: `compliance/vendor-review/YYYY/03-dpa-status-check.md` — vendor, DPA version, last signed date, any re-execution required, sign-off.

**Step 4 — Questionnaire or self-attestation (complete by January 25)**

Send the annual security questionnaire to all Critical vendors. For High vendors, accept the current SOC 2 report in lieu if obtained in Step 2. For Standard vendors, request a one-paragraph written self-attestation confirming no material security posture changes.

Minimum content to confirm annually:
- No unresolved critical or high security findings from the vendor's most recent internal assessment or pentest
- Incident notification SLA unchanged (≤24h)
- Subprocessor list unchanged or 30-day notice issued for any changes
- Data deletion capability confirmed

Evidence artifact: `compliance/vendor-review/YYYY/04-questionnaire-responses/VENDOR-NAME-response-YYYY.{pdf,md}`.

**Step 5 — Annual review sign-off and registry update (complete by January 31)**

The compliance-officer reviews all evidence from Steps 1–4 and produces a consolidated annual review memo:
- Risk score reassignment for any vendor where evidence warrants a change
- Any newly identified gaps added to §14 risk register and PRE checklist
- Updated §17.4 registry with "Next Review" dates advanced by 12 months
- Escalation items (vendors with unresolved DPA gaps, material SOC 2 exceptions, or tier upgrades required) flagged to security-engineer and founder

Evidence artifact: `compliance/vendor-review/YYYY/05-annual-review-memo.md` — compliance-officer sign-off with date, summary of material changes, open items with owners and deadlines.

**SOC 2 CC9.2 evidence package:** The five artifacts above collectively constitute the annual vendor management evidence package for CC9.2. Auditors will request these files during fieldwork. The evidence chain must be unbroken: if Step 5 sign-off references Steps 1–4, and Steps 1–4 artifacts are dated within the same January window, the evidence satisfies the CC9.2 requirement for periodic review of service provider security.

---

### 17.6 Risk Scoring Criteria

Each vendor in the registry is assigned a composite risk score. The score is calculated at initial onboarding (§17.3 Step 5) and re-evaluated at each annual review (§17.5 Step 5). The score informs tier assignment and escalation rules.

#### Scoring Factors

| Factor | 1 — Low | 2 — Medium | 3 — High |
|---|---|---|---|
| **Data sensitivity** | No personal data; Internal or Public tier only | Confidential data (identity, billing) | Restricted data (health, clinical, §13 special category) |
| **Certification level** | Current SOC 2 Type II (unqualified) or ISO 27001 | SOC 2 Type II with exceptions relevant to FORM, or SOC 2 Type I only | No current SOC 2 or ISO cert; questionnaire only |
| **DPA status** | Fully executed; SCCs in place | DPA in progress or under review; compensating controls documented | No DPA; processing personal data without agreement |
| **Incident history** | No incidents involving FORM data in prior 12 months; vendor-side incidents disclosed promptly | Minor incident (no FORM data exposure); disclosed within SLA | Incident involving FORM data or material vendor breach; notification delayed |
| **Data residency** | EU hosting available and selected for EU users; FORM contractually bound to region | EU hosting available but not yet selected; vendor asserts EU data stays in-region | No EU residency option; EU data transits or stored in third countries without adequate safeguards |
| **Subprocessor chain depth** | Vendor has ≤2 documented subprocessors; FORM has visibility into the chain | Vendor has 3–5 subprocessors; partial visibility | Vendor has >5 subprocessors or undisclosed chain; FORM cannot assess downstream risk |

#### Composite Score

Sum the six factor scores (range 6–18) and apply the following scale:

| Total Score | Risk Rating | Registry Colour | Escalation Rule |
|---|---|---|---|
| 6–8 | Low | 🟢 | No escalation; annual review per §17.5 |
| 9–11 | Medium | 🟡 | Compliance-officer reviews quarterly; remediation plan required for any factor scored 3 within 90 days |
| 12–14 | Elevated | 🟠 | Compliance-officer + security-engineer joint review quarterly; security-engineer must approve continued use; remediation plan within 30 days |
| 15–18 | High | 🔴 | Escalate to founder immediately; continued use requires explicit written acceptance of risk; migrate to alternative vendor within 6 months unless compensating controls reduce score to ≤11 |

#### Escalation Rules

- Any vendor with a DPA factor score of 3 (no DPA, processing personal data) is automatically 🔴 regardless of composite score. No waiver.
- Any Critical-tier vendor whose certification lapses (no current SOC 2 or ISO cert) is automatically promoted to 🟠 until renewed.
- Any vendor involved in a data breach affecting FORM users is immediately reviewed for continued use. The founder, compliance-officer, and security-engineer must jointly approve continued engagement within 72 hours.

---

### 17.7 New Sub-Processor Addition Process

Adding a new sub-processor after FORM is in the SOC 2 observation period requires completing all seven steps. For additions before the observation period, Steps 1–3 are minimum requirements; remaining steps must be completed before the observation period starts.

**Step 1 — Identify and classify**

The requesting team member (engineer, product, founder) submits a new vendor request to the compliance-officer via a Linear ticket tagged `compliance-vendor-request`. The request must include: vendor name, intended purpose, data categories that will flow to the vendor, estimated start date.

The compliance-officer assigns a preliminary tier within 3 business days.

**Step 2 — Complete §17.3 pre-approval checklist**

Run the full pre-approval checklist (§17.3 Steps 1–5). For Critical-tier vendors, all five steps must be completed before any production data flows. For High-tier vendors, DPA execution is the hard gate; certification review may run in parallel with the 30-day notice period.

**Step 3 — 30-day customer notice**

As required by GDPR Art. 28(2) and FORM's enterprise DPA template: notify all enterprise tenants 30 days before the new sub-processor begins processing their data. The notice must be sent even if the vendor integration is still in testing, provided it will eventually process production data.

Draft and send the notice per §17.3 Step 4. Log all notifications and any objections.

**Step 4 — Update sub-processor register**

Add the vendor to §13 (this document) and to `form.coach/legal/sub-processors` (public-facing list). The public list must be updated no later than the effective date of sub-processor addition. Include: vendor name, HQ, purpose, data categories, region, DPA status, transfer mechanism.

**Step 5 — Update Vendor Risk Registry**

Add the vendor to §17.4 with all required fields populated. Assign the composite risk score per §17.6. Set "Next Review" to the next January.

**Step 6 — Add to §14 risk register if applicable**

If the vendor introduces a new vendor risk (e.g., a critical single-point-of-failure dependency, a vendor with no SOC 2 cert), add a risk entry to §14 under the Vendor / Operational Risks category. Assign a residual score and named owner.

**Step 7 — Archive and file**

File all completed onboarding evidence (questionnaire responses, DPA PDF, certification, 30-day notice log) to `compliance/vendor-review/onboarding/VENDOR-NAME-YYYY-MM/` in the private compliance repo.

#### Emergency Sub-Processor Addition Exception

If a production incident requires integrating a new vendor immediately (e.g., switching from a failed vendor to an alternative), the following modified process applies:

- Steps 1 and 5 are completed within 24 hours of the decision to add the vendor
- DPA (Step 2, Step 3) must be executed within 7 days
- The 30-day customer notice clock begins immediately; the vendor may process data in the interim only if compensating controls are documented (e.g., no personal data transmitted until DPA is in place)
- The founder must provide written approval of the emergency exception before any personal data flows to the new vendor
- All standard steps must be completed within 30 days of the emergency addition; any steps not completed within 30 days escalate to 🔴 in the risk registry

---

### 17.8 Termination and Offboarding

When a vendor relationship ends (contract termination, migration to alternative, vendor insolvency), the following steps must be completed and documented before the vendor is removed from active registry records.

**Step 1 — Cease data flows**

Revoke all API keys, OAuth tokens, and credentials granting the vendor access to FORM systems. Confirm revocation in each system's access log. Document: credential type, system, revocation timestamp, confirming engineer.

Target: all credentials revoked within 24 hours of termination decision.

**Step 2 — Data deletion confirmation**

For vendors that hold copies of FORM user data (Critical and High tier), issue a formal written data deletion request referencing the DPA deletion clause. The request must specify:
- The scope of data subject to deletion (all FORM user data, or a specific subset)
- The deadline for deletion (per DPA; typically 30–90 days)
- The required confirmation format (written attestation from an authorised signatory)

File the deletion request and the vendor's written confirmation to `compliance/vendor-offboarding/VENDOR-NAME-YYYY-MM/deletion-confirmation.pdf`.

For Standard-tier vendors with no personal data: deletion confirmation is not required. Document that no personal data was held.

**Step 3 — Sub-processor register update**

Remove the vendor from §13 and from `form.coach/legal/sub-processors`. If the removal means enterprise tenants' data was previously processed by this vendor, issue a sub-processor change notice (no 30-day advance requirement for removals, but prompt notification is required per good-faith practice). Log the update.

**Step 4 — Registry archive**

Move the vendor entry in §17.4 from the active registry to the archive section in `compliance/vendor-registry/VENDOR-ARCHIVE.md`. Retain all onboarding, review, and offboarding evidence for a minimum of 7 years (matching the audit log retention period per §4 and GDPR Art. 17(3)(e) record-keeping exception).

**Step 5 — Compliance calendar update**

Remove the vendor from the next January annual review cycle. If the termination occurs within 30 days of the annual review, confirm that the offboarding is noted in the annual review memo (§17.5 Step 5 evidence artifact).

---

### 17.9 SOC 2 Control Mapping

| SOC 2 Control | Description | This Section | Evidence Artifacts | Current Status |
|---|---|---|---|---|
| **CC9.1** | FORM identifies, selects, and develops risk mitigation activities for risks arising from potential business disruptions; considers the use of vendors in risk mitigation | §17.2 (tier classification), §17.5 (annual review), §17.6 (risk scoring) | Annual review memo; vendor risk registry; §14 risk register entries for vendor risks | 🟡 Partial — programme defined; first annual review Q1 2027 |
| **CC9.2** | FORM assesses and monitors risks associated with third-party vendors; includes performance monitoring and review of relevant reports | §17.3 (pre-approval), §17.4 (registry), §17.5 (annual review), §17.7 (new additions) | Vendor registry; SOC 2 / ISO cert files; questionnaire responses; annual review memo with sign-off | 🟡 Partial — programme defined; first full evidence cycle Q1 2027 |
| **CC9.3** | FORM evaluates compliance with vendor obligations; includes contract terms and access rights appropriate to vendor risk | §17.3 Steps 2–3 (DPA and cert review), §17.8 (termination — deletion confirmation and credential revocation) | Executed DPAs; SCCs; deletion confirmation letters; credential revocation logs | 🟡 Partial — DPAs in place for 6 of 8 sub-processors; Sentry DPA in progress; Vanta/Drata DPA pre-activation |
| **P8.1** | FORM discloses personal information to third parties only in accordance with its privacy commitments; monitors third-party compliance with privacy commitments | §13 (sub-processor register + disclosure), §17.3 Step 4 (30-day notice), §17.7 Steps 3–4 (new processor notice + register update), §17.8 Steps 2–3 (deletion + register update) | Sub-processor register with DPA status; 30-day notice logs; deletion confirmations | 🟡 Partial — register published (§13); notice process defined; first full cycle P8.1 evidence Q1 2027 |

---

### 17.10 Gap Closure Status

This section directly addresses the two 🔴 Gaps identified in the SOC 2 readiness tracker and formalises one 🟡 Partial item:

| Gap | Pre-§17 Status | Post-§17 Status | Remaining Action to Reach 🟢 |
|---|---|---|---|
| Vendor security review process | 🔴 Gap — DPAs signed but formal vendor registry needed | 🟡 Partial — programme documented: tier classification, pre-approval checklist, risk scoring criteria, escalation rules, 7-step addition workflow, offboarding procedure | First annual review executed and evidence filed (Q1 2027); Vanta/Drata DPA executed before activation; Sentry DPA executed (PRE-08 closure) |
| Annual vendor security review | 🔴 Gap — Process needed | 🟡 Partial — 5-step January process defined with named evidence artifacts mapped to CC9.2; scheduled in §15.1 compliance calendar | First annual review completed January 2027; all five evidence artifacts filed to `compliance/vendor-review/2027/` |
| Vendor risk registry | 🟡 Partial — sub-processors listed in §13 without formal risk scoring | 🟡 Partial — formalised: all 11 vendors (8 sub-processors + Better Uptime + PagerDuty + Linear) entered with tier, data categories, DPA status, cert, risk score, and owner; composite scoring criteria defined in §17.6 | Score promoted to 🟢 when: (a) Sentry DPA executed → risk score drops to 🟢 Low; (b) Vanta/Drata activated with DPA → risk score confirmed; (c) first annual review confirms all certs current |

**Impact on SOC 2 readiness metrics:**

- Vendor security review process: 🔴 Gap → 🟡 Partial
- Annual vendor security review: 🔴 Gap → 🟡 Partial
- CC9.2 control status: 🔴 Gap → 🟡 Partial (was already 🟡 Partial per §16.11; this section provides the programme substance that was previously missing)
- Critical gaps: 3 → 1 (two 🔴 Gaps converted to 🟡 Partial by this section)
- Partial controls: 30 → 32
- Readiness estimate: ~56% → ~58%

---

## 18. Business Continuity & Disaster Recovery (BCP/DRP)

> Owner: `devops-lead` + `compliance-officer`. Review: after each DR drill, after any infrastructure change that affects RTO/RPO, or annually.
> SOC 2 controls: CC7.4 (Response to detected incidents), CC7.5 (Recovery from incidents), A1.1 (Availability commitments), A1.2 (Capacity and performance management).

---

### 18.1 Purpose

This section documents FORM's Business Continuity Plan (BCP) and Disaster Recovery Plan (DRP). It exists to:

1. Satisfy SOC 2 CC7.5 and A1.1 requirements for documented recovery procedures
2. Define RTO/RPO commitments that can be disclosed to enterprise customers
3. Ensure the founding engineer (sole operator pre-Series A) has a written playbook, not tribal knowledge
4. Establish the evidence artifact for the January DR drill (§15.1 compliance calendar)

**Scope:** Production infrastructure only. This covers Supabase (database), Cloudflare Workers (API edge), Supabase Storage (media), ElevenLabs (voice), and Anthropic (LLM). It does not cover the iOS app itself (App Store distribution is Apple-operated).

---

### 18.2 Recovery Time Objective (RTO) and Recovery Point Objective (RPO)

These targets govern what FORM promises enterprise customers and what the engineering team designs toward.

| Service tier | RTO | RPO | Notes |
|---|---|---|---|
| **Enterprise** | 4 hours | 1 hour | Contractual SLA. Failure to meet: pro-rata credit per enterprise contract. |
| **Pro (consumer)** | 8 hours | 4 hours | Best-effort commitment. Disclosed in Terms of Service. |
| **Free tier** | 24 hours | 24 hours | No contractual commitment. |
| **Auth / SSO** | 1 hour | 15 minutes | SSO outage blocks enterprise users from logging in. Priority restore. |
| **Audit log** | N/A (read-only in recovery) | 0 minutes | HMAC-chained; append-only. Recovery means read access, not write. |

**RPO implementation basis:**

- Supabase enables Point-in-Time Recovery (PITR) with WAL streaming. Default PITR retention: 7 days. RPO for the database is therefore a function of WAL stream latency to the warm standby, which Supabase guarantees at < 5 minutes in Multi-AZ configuration.
- Supabase Storage (object store) replicates across AZs within the selected data region. RPO for media is effectively 0 within-region.
- The 1-hour RPO for enterprise and 4-hour for Pro represent the maximum acceptable data loss window, not the expected loss window (which is < 5 minutes under normal PITR).

---

### 18.3 Failure Scenarios and Response Procedures

#### Scenario A — Supabase database unavailability

**Definition:** Supabase Postgres primary is unreachable or returning errors on all queries for > 5 minutes.

**Detection:** Better Uptime health check on `/api/health` returns non-2xx for 3 consecutive intervals (1 minute intervals). PagerDuty P1 alert fires. Sentry error rate spike (database connection errors).

**Response:**

| Step | Action | Owner | Time budget |
|---|---|---|---|
| 1 | Acknowledge PagerDuty alert; check Supabase Status Page (status.supabase.com) | On-call engineer | < 5 min |
| 2 | If Supabase platform incident: post status update to form.coach/status; switch API to maintenance mode (return 503 with `Retry-After: 1800`) | On-call engineer | < 10 min |
| 3 | If FORM-side issue (misconfiguration, migration gone wrong): invoke INCIDENT_RESPONSE.md §5.2 (Database Incident runbook) | On-call engineer | < 15 min |
| 4 | If Supabase platform incident persists > 2 hours: contact Supabase enterprise support (requires paid plan); escalate to founder | On-call engineer | 2 hours |
| 5 | If total outage > 4 hours and RPO breach imminent: initiate PITR restore to latest clean checkpoint (INCIDENT_RESPONSE.md §5 + DATA_MODEL.md §10) | Founder + on-call | 4 hours |
| 6 | Notify enterprise tenants via DPA-specified contact channel; issue incident communication per §6 INCIDENT_RESPONSE.md template | On-call engineer | Within 1 hour of confirmed P1 |

**RTO target for this scenario: 4 hours.** If Supabase recovers within 4 hours (platform incident), no PITR restore is needed. If outage exceeds 4 hours, PITR restore is initiated regardless.

---

#### Scenario B — Cloudflare Workers API unavailability

**Definition:** All API endpoints unreachable globally for > 5 minutes, confirmed as Cloudflare-side.

**Detection:** Better Uptime health check fails globally. Cloudflare Status Page (cloudflarestatus.com) confirms incident.

**Response:**

1. Post status update to form.coach/status. Enterprise tenant notification per DPA.
2. No operational fallback exists for Cloudflare Workers outages — FORM's edge runtime has no secondary provider at this stage. Document as a known risk in the risk register (R-004).
3. After recovery: verify all Worker deployments are serving the correct version (`wrangler tail` in production). Replay any queued background jobs if applicable.

**RTO target for this scenario: dependent on Cloudflare recovery.** Cloudflare's historical uptime is > 99.99%; this scenario is low probability. The compensating control is Cloudflare's own redundancy (global anycast network).

**DRP gap:** No secondary CDN/edge failover documented. This is a known gap. Mitigation: Cloudflare's SLA and historical reliability. Formal secondary-provider DR is Post-Series-A scope.

---

#### Scenario C — Data corruption (accidental or malicious)

**Definition:** Production data found to be corrupted, partially deleted, or tampered with. Includes: accidental mass-delete via admin API, failed migration with data loss, insider threat.

**Detection:** Monitoring anomaly on row counts in key tables (pg_stat_user_tables row count drops > 5% in < 1 hour triggers PagerDuty P1). Audit log review triggered by any P1 incident.

**Response:**

1. Immediately freeze the affected table(s): revoke `INSERT`, `UPDATE`, `DELETE` permissions from `form_api` role for the affected table using `ALTER TABLE <table> DISABLE TRIGGER ALL; REVOKE ...` — stops further corruption while preserving reads.
2. Snapshot the current state before any restore attempt: `pg_dump --table=<affected_table>`.
3. Identify the corruption timestamp from audit log and/or Cloudflare Worker logs.
4. Invoke PITR restore to a timestamp 5 minutes before the corruption event. Follow DATA_MODEL.md §10 (PITR Tenant-Isolated Restore Runbook) exactly.
5. Compare restored data against pre-corruption snapshot; apply delta to production.
6. Write post-incident report (INCIDENT_RESPONSE.md §8 template).
7. If malicious actor suspected: invoke INCIDENT_RESPONSE.md §5.5 (Security Breach), notify compliance-officer, evaluate breach notification obligation (72-hour GDPR clock starts from confirmed breach).

**RPO target for this scenario: 1 hour (enterprise), 4 hours (Pro).** With PITR at < 5-minute granularity, the limiting factor is detection time, not restore granularity.

---

#### Scenario D — Complete environment loss ("nuke scenario")

**Definition:** Loss of all production Supabase project data, Cloudflare Workers configuration, and access credentials simultaneously. Possible causes: compromised Supabase account, supplier-side data loss, operator error at provider level.

**Response (cold start from backup):**

| Step | Action |
|---|---|
| 1 | Retrieve Supabase backup export from cold storage (Backblaze B2 bucket, credentials in 1Password team vault under "DR / Cold Storage") |
| 2 | Create new Supabase project in target region; restore from export using Supabase CLI `supabase db restore` |
| 3 | Re-run all migrations from `supabase/migrations/` in order |
| 4 | Restore Cloudflare Workers configuration from `wrangler.toml` in git; deploy via `wrangler deploy` |
| 5 | Update all environment secrets in Cloudflare Workers dashboard (Anthropic, ElevenLabs, Supabase keys from 1Password) |
| 6 | Re-point DNS (Cloudflare DNS panel; `api.form.coach` CNAME to new Worker route) |
| 7 | Smoke test: run F1–F8 flows from qa-walker.md before lifting maintenance mode |
| 8 | Notify all enterprise tenants; issue public incident communication |

**RTO target for this scenario: 8 hours.** This is a worst-case scenario. The actual rebuild time for a solo engineer following this runbook is estimated at 4–6 hours, with 2-hour buffer for unexpected issues.

**Prerequisite (gap):** Automated nightly backup export to Backblaze B2 is not yet implemented. Current state: Supabase PITR is the only backup mechanism. Cold storage backup is on the Q3 implementation roadmap. Until implemented, this scenario's RTO is Supabase's own DR capability, which is contractually undefined on the Pro plan.

---

### 18.4 DR Drill Procedure

Per §15.1 (Annual Compliance Calendar), a DR drill is scheduled for Q1 January annually. The drill validates that the runbooks above are executable by the on-call engineer without tribal knowledge.

#### Drill scope

The January drill exercises **Scenario C (data corruption)** in a staging environment, because:
- It tests the PITR restore process (highest-risk manual procedure)
- It validates the audit log analysis workflow
- It does not require taking production offline
- It generates SOC 2 evidence (drill report = CC7.5 evidence artifact)

#### Drill procedure

```
Pre-drill (D-7):
  [ ] Notify all engineers that a drill will occur this week
  [ ] Confirm staging environment is a recent copy of production schema
  [ ] Confirm PITR is enabled on the staging Supabase project
  [ ] Assign DR Lead (engineer who will run the drill; must NOT be the author of this runbook)

Drill execution (2-4 hours):
  [ ] DR Lead simulates data corruption: delete 10% of rows from workouts table in staging
  [ ] DR Lead follows Scenario C runbook above, step-by-step
  [ ] Observer (compliance-officer or devops-lead) timestamps each step
  [ ] Record: time-to-detect, time-to-contain, time-to-restore, data loss (rows not recovered)
  [ ] Compare actual RTO/RPO against targets in §18.2

Post-drill (D+1):
  [ ] Write drill report (see template below)
  [ ] File drill report in evidence folder: evidence/dr-drills/YYYY-MM-drill-report.md
  [ ] Update this section if runbook gaps discovered
  [ ] Update SOC 2 evidence registry
```

#### Drill report template

```markdown
# DR Drill Report — [YYYY-MM]

**Date:** YYYY-MM-DD
**Scenario exercised:** C (Data Corruption Simulation)
**DR Lead:** [name]
**Observer:** [name]

## Results

| Metric | Target | Actual | Pass/Fail |
|---|---|---|---|
| Time to detect | < 10 min | X min | |
| Time to contain | < 15 min from detect | X min | |
| Time to restore | < 2 hours (staging) | X min | |
| Data loss | 0 rows (PITR to pre-corruption) | X rows | |
| Runbook gaps found | 0 | X | |

## Runbook gaps discovered

[List any steps that failed, were unclear, or took longer than expected.]

## Remediation actions

[Tickets created to address gaps.]

## SOC 2 evidence classification

CC7.5 — Incident recovery drill. Retain for 7 years per §15.1 evidence retention schedule.
```

---

### 18.5 Communication Tree During DR Events

| Phase | Internal | Enterprise tenants | Consumer users |
|---|---|---|---|
| Detection (P1 confirmed) | PagerDuty alert; engineer acknowledges | — | — |
| 0–30 min | Engineer assesses scope; founder notified if > 30 min until recovery | — | Status page updated |
| 30 min–2 hours | Founder directs response; investor notification if material impact likely | DPA-specified contact notified via email; ticket opened in customer success | Status page + banner on form.coach |
| 2+ hours | Board/investor notification if data loss | Formal incident communication (INCIDENT_RESPONSE.md §6 template); hourly updates | Public status page; push notification if app is functional |
| Recovery | All-clear to engineer team; post-incident report committed within 48h | Recovery confirmation to tenant admin; SLA credit calculation if applicable | Status page resolved; push notification "Service restored" |

---

### 18.6 SOC 2 Control Mapping

| SOC 2 Control | Description | BCP/DRP Evidence |
|---|---|---|
| CC7.4 | Response to detected incidents | §18.3 failure scenarios with step-by-step response; INCIDENT_RESPONSE.md §4 lifecycle |
| CC7.5 | Recovery from incidents | §18.3 restore procedures; §18.4 drill procedure + annual drill report |
| A1.1 | Availability commitments and system monitoring | §18.2 RTO/RPO commitments; Better Uptime monitoring (§3 INCIDENT_RESPONSE.md) |
| A1.2 | Capacity and performance | Supabase PITR + BRIN index maintenance (DATA_MODEL.md §11.6); OBSERVABILITY.md §2 SLOs |
| CC9.1 | Risk mitigation via vendor management | §18.3 Scenario B — Cloudflare gap documented as R-004 in Risk Register |

#### Gap closure status

| Gap | Previous status | Status after §18 | Remaining work |
|---|---|---|---|
| DR runbook documented | 🟡 Partial (drill scheduled but no runbook) | 🟢 Done | Execute first drill Q1-Jan; file drill report |
| RTO/RPO commitments defined | 🔴 Gap | 🟢 Done | Include in enterprise contract template |
| Cold storage backup (Scenario D) | 🔴 Gap | 🟡 Partial (gap documented; runbook written) | Implement nightly B2 export |
| CC7.5 evidence artifact | 🔴 Gap | 🟡 Partial (runbook written; drill report pending) | Execute drill; file report |

**Readiness impact:** DR runbook gap: 🟡 Partial → 🟢 Done (runbook). CC7.5: 🔴 Gap → 🟡 Partial. Cold storage backup gap remains 🔴 (implementation pending). Net: 1 new Partial → Done; 1 Gap → Partial. Critical gaps: 1 → 1 (cold storage backup is newly documented as a gap but was previously undocumented, so no net change). Readiness: ~58% → ~60%.

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
- [x] Define penetration test program — PRE-21 partial; Section 16 documents scope, methodology, SLAs (May 2026)
- [ ] Execute first penetration test — PRE-21 complete; firm selection + engagement pending
- [ ] Security awareness training first cohort — PRE-22; requires first hire
- [ ] Complete first quarterly access review — PRE-23

---

## 19. Cold Storage Backup — Architecture, Implementation & SOC 2 Evidence

> SOC 2 evidence: A1.2 (environmental protections for availability), A1.3 (recovery procedures tested), CC7.5 (recovery from incidents), C1.2 (disposal/destruction — inverse).
> **Closes critical gap:** "Cold storage backup (Scenario D nuke case)" 🔴 documented in §18.6.
> Owner: devops-lead + compliance-officer. Implementation milestone: M4 (enterprise launch gate).

---

### 19.1 Why This Gap Exists and Why It Matters

Supabase PITR (Point-in-Time Recovery) retains up to 7 days of write-ahead log on the Pro plan and up to 30 days on the Team plan. This covers operational recovery (bad migration rollback, accidental row deletion) but fails two enterprise-tier requirements:

1. **GDPR Art. 5(1)(e) storage limitation compliance** — audit logs are evidence artifacts that must be retained for 7 years per §15.1. Supabase PITR cannot satisfy a 7-year retention window; an independent cold storage layer can.

2. **Scenario D ("nuke case") recovery** — if the Supabase account is terminated, compromised, or otherwise inaccessible, recovery requires a backup that is not co-located on Supabase infrastructure. PITR is a Supabase-internal capability; it cannot serve as the last resort.

Until §19 is implemented, FORM cannot honestly represent A1.2 (long-term data protection) as fully in place to enterprise customers, and the SOC 2 auditor will flag a gap on first-year Type I review.

---

### 19.2 Three-Tier Backup Architecture

| Tier | Technology | Retention | RTO / RPO | Cost model |
|---|---|---|---|---|
| **Tier 1 — Operational** | Supabase PITR (WAL replay) | 7 d (Pro) / 30 d (Team) | 4h RTO / 1h RPO (§18.2) | Included in Supabase plan |
| **Tier 2 — Medium-term** | Nightly logical dump → Cloudflare R2 | 90 days rolling | 8h RTO / 24h RPO | ~$0.015/GB/month R2 storage |
| **Tier 3 — Cold storage** | Monthly snapshot → Backblaze B2 (WORM) | 7 years | 48h RTO / 30 d RPO | ~$0.006/GB/month B2 storage |

Tier 1 is Supabase-managed — no implementation required beyond keeping the plan current. Tiers 2 and 3 are what this section specifies.

**Design principle:** Each tier is independent. Tier 2 cannot help if the R2 account is compromised; Tier 3 cannot help for same-day recovery. Use the lowest-numbered tier that satisfies the recovery scenario.

---

### 19.3 Tier 2: Nightly Logical Dump → Cloudflare R2

**Trigger:** Cloudflare Workers Cron Trigger, daily at `03:00 UTC` (minimum-traffic window).
**Worker name:** `form-backup-exporter`

#### 19.3.1 Backup process

The Worker invokes a Supabase Edge Function (`backup-runner`) that shells out to `pg_dump` via `Deno.Command`, streams the compressed dump back to the Worker over HTTPS, and the Worker writes it to R2. No plaintext dump file touches any intermediate disk.

```
1. Workers Cron fires → fetch POST https://<supabase-project>.functions.supabase.co/backup-runner

2. Edge Function executes:
     pg_dump -Fc --no-acl --no-owner "$DATABASE_URL" \
       | gzip -9 \
       | tee >(sha256sum > /tmp/hash.txt) \
       | POST to Worker stream endpoint

3. Worker streams body to R2 under:
     backups/daily/YYYY/MM/DD/form_YYYYMMDD_HHmmss.dump.gz

4. SHA-256 hash written to:
     backups/daily/YYYY/MM/DD/form_YYYYMMDD_HHmmss.dump.gz.sha256

5. JSON manifest written to:
     backups/daily/YYYY/MM/DD/manifest.json
   {
     "backup_type":    "logical_pg_dump",
     "format":         "custom-gzip",
     "timestamp_utc":  "YYYY-MM-DDTHH:MM:SSZ",
     "dump_file":      "form_YYYYMMDD_HHmmss.dump.gz",
     "dump_sha256":    "<64-char hex>",
     "pg_version":     "15.x",
     "row_counts":     { "users": N, "workouts": N, "audit_log": N },
     "size_bytes":     N,
     "worker_version": "<git_sha>"
   }

6. Worker writes audit event (DEC-030):
     action:   "system.backup_completed"
     actor:    "backup-worker"
     metadata: { tier: 2, r2_path: "...", sha256: "...", duration_ms: N, size_bytes: N }

   On any failure:
     action:   "system.backup_failed"
     metadata: { tier: 2, error: "<message>", stage: "dump|upload|manifest" }
   → PagerDuty alert fires (severity: P2)
   → Two consecutive failures → severity escalates to P1
```

#### 19.3.2 R2 bucket configuration

```hcl
# Terraform — Cloudflare R2 lifecycle
resource "cloudflare_r2_bucket_lifecycle_configuration" "backup_daily" {
  account_id  = var.cloudflare_account_id
  bucket_name = "form-backups"

  rules = [{
    id     = "expire-daily"
    status = "enabled"
    filter = { prefix = "backups/daily/" }
    expiration = { days = 90 }
  }]
}
```

**Access:** R2 bucket is accessible only by the `form-backup-exporter` Worker service token (read/write) and a named `devops-lead` API token (read-only for restore operations). No application code holds R2 credentials.

**Encryption at rest:** R2 encrypts all objects with AES-256 by default. No additional client-side encryption is applied at Tier 2 — the access restriction is the primary control. R2 bucket is not publicly accessible.

---

### 19.4 Tier 3: Monthly Snapshot → Backblaze B2 (WORM)

**Trigger:** Cloudflare Workers Cron Trigger, `04:00 UTC` on the 1st of each month.
**Worker name:** `form-cold-backup-archiver`

#### 19.4.1 Archive process

```
1. Worker lists R2 prefix backups/daily/ → selects the most recent
   daily manifest.json whose timestamp_utc falls within the previous
   calendar month (deterministic selection — no random sampling)

2. Worker streams the .dump.gz from R2 via R2 binding

3. Worker applies AES-256-GCM client-side encryption:
     key = Cloudflare Workers Secret B2_ENCRYPTION_KEY_YYYYMM
           (month-scoped key; new key created on 1st of each month)
     iv  = crypto.getRandomValues(new Uint8Array(12))  [96-bit]
     AAD = "form-cold-backup|YYYY-MM|<source_sha256>"

4. Encrypted blob uploaded to B2 under:
     form-cold-backups/YYYY/MM/form_YYYYMM_snapshot.dump.gz.enc

5. Manifest written to:
     form-cold-backups/YYYY/MM/manifest.json
   {
     "source_r2_path":       "backups/daily/YYYY/MM/DD/...",
     "source_sha256":        "<hex>",
     "encrypted_b2_path":    "form-cold-backups/YYYY/MM/...",
     "encryption_algorithm": "AES-256-GCM",
     "key_id":               "B2_ENCRYPTION_KEY_YYYYMM",
     "iv_hex":               "<24-char hex>",
     "timestamp_utc":        "YYYY-MM-DDTHH:MM:SSZ"
   }

6. Audit event (DEC-030):
     action: "system.cold_backup_completed"
     metadata: { tier: 3, b2_path: "...", source_sha256: "...",
                 key_id: "...", duration_ms: N }
```

#### 19.4.2 B2 bucket configuration

| Setting | Value | Rationale |
|---|---|---|
| Bucket name | `form-cold-backups` | Single bucket; path-based partitioning by year/month |
| Object Lock | WORM, 7-year lock | GDPR Art. 5(1)(e) and SOC 2 A1.2; prevents deletion by compromised credentials |
| Versioning | Enabled | Protects against overwrite (shouldn't happen; key is month-unique path) |
| Lifecycle rule | None on locked objects | WORM expiration is governed by Object Lock, not lifecycle |
| Access | B2 application key scoped to this bucket only | devops-lead and compliance-officer hold keys in 1Password vault |

#### 19.4.3 Encryption key management

| Item | Detail |
|---|---|
| Key scope | One key per monthly backup (`B2_ENCRYPTION_KEY_YYYYMM`) |
| Key generation | `crypto.getRandomValues(new Uint8Array(32))` — 256-bit, generated at Worker startup on the 1st |
| Key storage (primary) | Cloudflare Workers Secrets, scoped to `form-cold-backup-archiver` |
| Key archive (DR) | 1Password vault, tag `BACKUP-KEY-YYYYMM`, retention: 7 years + 1 year buffer |
| Key escrow | compliance-officer holds a copy in a separate Bitwarden vault (air-gapped from production credentials) |
| Key rotation | Monthly (new key per backup; old keys never reused) |
| Key destruction | After 7 years and 1 month, the key for `YYYYMM` is scheduled for destruction — coordinate with compliance-officer to confirm all restores complete before destruction |

**Why per-month keys:** If a monthly key is compromised, the attacker can decrypt that one month's backup but not others. Each backup is independently protected.

#### 19.4.4 Cost estimate

At 50 GB uncompressed database size (~15 GB compressed):

| Line item | Year 1 cost | Year 7 cost (cumulative storage) |
|---|---|---|
| B2 storage (12 snapshots × ~15 GB = 180 GB) | ~$1.08/month | ~$7.56/month (84 × 15 GB) |
| B2 egress (restore — rare; ~$0.01/GB) | Near zero | Near zero |
| Workers Cron executions (monthly) | Negligible | Negligible |

Cold storage is not a material cost line pre-Series A.

---

### 19.5 Backup Integrity Verification

Unverified backups are not backups. Two verification cadences apply:

#### 19.5.1 Monthly spot-check (devops-lead, first Monday of each month)

```bash
# 1. Identify the previous month's most recent Tier 2 daily backup
R2_MANIFEST="backups/daily/YYYY/MM/DD/manifest.json"

# 2. Download dump to local machine (NOT production)
wrangler r2 object get form-backups "$R2_PATH" --file form_test.dump.gz

# 3. Verify SHA-256 integrity
sha256sum form_test.dump.gz | awk '{print $1}' | diff - <(cat form_test.dump.gz.sha256)
# Expected: no diff (exit 0)

# 4. Restore to a local Postgres 15 instance
pg_restore -d "postgres://localhost/form_restore_test" form_test.dump.gz

# 5. Spot-check row counts vs. manifest
psql "postgres://localhost/form_restore_test" \
  -c "SELECT 'users' AS t, COUNT(*) FROM users UNION ALL
      SELECT 'workouts', COUNT(*) FROM workouts UNION ALL
      SELECT 'audit_log', COUNT(*) FROM audit_log;"
# Compare output to manifest.json row_counts — acceptable variance ±2% (intra-day inserts)

# 6. Document in Linear ticket tagged backup-verification-YYYY-MM
# 7. If restore fails → P1 incident; notify security-engineer + devops-lead immediately
```

#### 19.5.2 Annual full restore test (January; aligned with §15.1 compliance calendar)

```
1. Decrypt the most recent Tier 3 B2 snapshot:
     a. Retrieve B2_ENCRYPTION_KEY_YYYYMM from 1Password vault (compliance-officer present)
     b. Download encrypted blob from B2
     c. Decrypt: AES-256-GCM with stored IV and AAD from manifest
     d. Verify decrypted SHA-256 matches manifest source_sha256

2. Restore to a staging Supabase project (isolated from production):
     pg_restore -d "$STAGING_DATABASE_URL" form_YYYYMM_snapshot.dump.gz

3. Verify RLS policies are intact on the restored instance:
     -- Run CI test suite cross_tenant_rls_spec against staging
     -- Expected: 0 failures

4. Apply PITR WAL from staging Supabase (Tier 1) to advance to current day
   → confirms Tier 1 → Tier 2 → Tier 3 chain is recoverable end-to-end

5. Document results in Linear ticket tagged backup-annual-restore-YYYY:
     - Decryption: PASS/FAIL
     - SHA-256 verification: PASS/FAIL
     - Restore duration: N minutes
     - Row count variance: N%
     - RLS test: PASS/FAIL

6. File as SOC 2 A1.3 evidence artifact:
     "Annual backup restore test — YYYY" (git-committed PDF or ticket export)
     Retain for 7 years per §15.1 evidence retention schedule

7. If any step FAILS → P0 incident; notify founder + security-engineer immediately
```

---

### 19.6 Tenant Data Isolation in Backup Files

The pg_dump includes all tenants' rows (separated by `tenant_id` under RLS). This has implications that must be documented for SOC 2 auditors.

**Principle:** Tier 2 and Tier 3 restore operations are full-instance restores. They are not tenant-specific. Tenant-specific restores use Supabase PITR + `DATA_MODEL.md §10` runbook or the GDPR Art. 20 portability export (`DATA_MODEL.md §12.5`).

**Access control:**

| Resource | Who has access | How access is granted |
|---|---|---|
| `form-backups` R2 bucket | `form-backup-exporter` Worker (write) + devops-lead API token (read) | Workers Secrets + Cloudflare API token scoped to bucket |
| `form-cold-backups` B2 bucket | devops-lead, compliance-officer | B2 application keys in 1Password vault; physical access log required |
| Backup encryption keys | devops-lead (Workers Secrets), compliance-officer (1Password vault escrow) | Named individuals only; no shared service accounts |

**No application code holds backup bucket credentials.** Backup buckets are infrastructure-only. Any access outside the named individuals above requires written approval from the compliance-officer and is logged as an audit event (`system.backup_accessed`).

**Cross-tenant sensitivity:** A person with access to a Tier 2 dump file has read access to all tenants' data at the point-in-time of that backup. This is why the three-tier access model is not negotiable — especially the devops-lead + compliance-officer dual-custody requirement for Tier 3 decryption.

---

### 19.7 Privacy Constraints on Backup Access

These constraints apply regardless of whether a backup access is for recovery or verification:

| Constraint | Rule |
|---|---|
| Individual user health data | Never restored to a non-production environment without prior anonymisation of health-adjacent fields (`body_weight`, `health_conditions`, `cv_confidence_score`) |
| Tenant data cross-contamination | Backup files must never be used to answer "what data does Tenant A have?" — that is a GDPR Art. 15 subject access request, not a recovery operation |
| HMAC chain audit logs | If a restore overwrites `audit_log`, the HMAC chain from that point forward is broken. Restores must append a `system.restore_executed` event that re-anchors the chain — see `docs/AUDIT_LOG_SCHEMA.md §HMAC chaining` and DEC-030 |
| HR-never-sees-individual-data floor | Restores to a staging environment must apply the same RLS policies as production before any human accesses the data |

---

### 19.8 SOC 2 Control Closure

| SOC 2 Control | Description | Evidence Provided by §19 |
|---|---|---|
| **A1.2** | Environmental protections include physical safeguards, software, and infrastructure to protect against events that could impair availability commitments | Three-tier backup architecture (§19.2); R2 90-day rolling retention (§19.3.2); B2 7-year WORM Object Lock (§19.4.2) |
| **A1.3** | Recovery and resumption of services — procedures are tested | Monthly spot-check (§19.5.1); Annual full restore test (§19.5.2); results documented and filed as evidence |
| **C1.2** | Confidential information is protected during disposal | B2 WORM Object Lock prevents premature deletion; key escrow for 7-year access guarantee; key destruction schedule after retention period (§19.4.3) |
| **CC7.5** | Recovery from identified security incidents | Tier 3 cold backup provides the recovery path for Scenario D (§18.3 — the "nuke case") that was not satisfied by Tier 1 PITR alone |

#### Gap closure table

| Gap | Status before §19 | Status after §19 | Remaining work |
|---|---|---|---|
| Cold storage backup | 🔴 Gap (architecture undefined; implementation absent) | 🟡 Partial (architecture specified; implementation checklist defined) | Deploy Workers, provision R2 lifecycle, provision B2 WORM bucket, execute first backup |
| A1.2 — long-term data protection | 🟡 Partial (PITR only; no independent copy) | 🟡 Partial (three-tier architecture specified) | First nightly backup executed and verified |
| A1.3 — recovery procedures tested | 🟡 Partial (restore runbook for Tier 1 only; §10 DATA_MODEL.md) | 🟡 Partial (Tier 2 + 3 spot-check and annual restore procedures documented) | Execute first monthly spot-check (M5); file first annual restore report (M14-Jan) |
| CC7.5 — Scenario D coverage | 🟡 Partial (runbook written; no independent backup to restore from) | 🟡 Partial (Tier 3 B2 backup provides the independent copy Scenario D requires) | Implement and execute first backup |

**Readiness impact:** The sole remaining 🔴 critical gap (cold storage backup) moves to 🟡 Partial upon implementation. Critical gaps: 1 → 0. Readiness: ~60% → ~63% (three A-series and one CC7.5 gap close partially; full credit requires first backup executed and first annual restore filed).

---

### 19.9 Implementation Checklist (devops-lead)

Execute in the order listed. Each P0 item is a gate for enterprise launch (M4).

| # | Task | Priority | Milestone | Notes |
|---|---|---|---|---|
| 19-01 | Provision `form-backups` R2 bucket | P0 | M4 | Apply 90-day lifecycle rule for `backups/daily/` prefix |
| 19-02 | Provision Supabase Edge Function `backup-runner` with pg_dump + gzip + stream | P0 | M4 | Requires Supabase Team plan for adequate Edge Function execution time (>10 s) |
| 19-03 | Deploy `form-backup-exporter` Worker with daily Cron Trigger at `03:00 UTC` | P0 | M4 | Edge Function must exist first |
| 19-04 | Add Workers Secrets: `SUPABASE_BACKUP_RUNNER_KEY`, `DATABASE_URL` (read-only replica connection string) | P0 | M4 | Scoped to `form-backup-exporter` only |
| 19-05 | Provision `form-cold-backups` B2 bucket with 7-year WORM Object Lock | P0 | M4 | Requires B2 account setup; Object Lock must be enabled at bucket creation (cannot be added retroactively) |
| 19-06 | Generate `B2_ENCRYPTION_KEY_YYYYMM` for current month; store in Workers Secrets and 1Password | P0 | M4 | Escrow copy to compliance-officer 1Password vault immediately |
| 19-07 | Deploy `form-cold-backup-archiver` Worker with monthly Cron Trigger at `04:00 UTC` on 1st | P0 | M4 | R2 daily backups must be running for ≥1 day first |
| 19-08 | Add `system.backup_completed`, `system.backup_failed`, `system.cold_backup_completed`, `system.backup_accessed` to `audit_log` action taxonomy in `docs/AUDIT_LOG_SCHEMA.md` | P1 | M4 | Coordinate with security-engineer (DEC-030 action namespace) |
| 19-09 | Add PagerDuty alert: backup failure on 2 consecutive nights → P1; single failure → P2 | P1 | M4 | PagerDuty provisioning (PRE-10) must be complete first |
| 19-10 | Add Terraform configuration for R2 lifecycle rule and B2 bucket (IaC) | P1 | M4 | Manual provisioning acceptable for M4; Terraform required before SOC 2 Type I |
| 19-11 | Execute and verify first nightly Tier 2 backup (spot-check via §19.5.1 procedure) | P0 | M5 | Worker must be deployed and running |
| 19-12 | Execute first Tier 3 monthly archive; verify decryption and restore | P0 | M5 | At least one nightly backup must exist in R2 |
| 19-13 | File §19.5.1 result as A1.2 SOC 2 evidence artifact | P0 | M5 | Git-commit the Linear ticket export or screenshot |
| 19-14 | Add monthly spot-check to January compliance calendar (§15.1) | P1 | M5 | Recurring event; owner: devops-lead |
| 19-15 | Schedule and execute first annual full restore test (§19.5.2) | P0 | M14 (Jan Year 2) | File as A1.3 evidence; coordinate with compliance-officer for key retrieval |
| 19-16 | Confirm B2 Object Lock status via B2 API before enterprise first-deal close | P0 | M4 | Auditor will ask; have the API confirmation screenshot on file |

---

## 20. Status Page Architecture & Availability Communication (status.form.coach)

> Owner: `devops-lead` + `compliance-officer`. Review: after each architecture change that affects monitored components, or annually.
> SOC 2 controls: **A1.1** (System availability commitments), **A1.2** (Environmental protections and capacity), **CC7.4** (Communicates security incidents to external parties), **CC6.8** (Prevents unauthorized access — scheduled maintenance communication).

---

### 20.1 Purpose

This section designs FORM's customer-facing status page at `status.form.coach` and closes the critical SOC 2 Availability gap noted in §2 and §11:

> *"Customer-facing status page — 🔴 Gap — status.form.coach required for enterprise tier"*

A public status page is required for three distinct reasons:

1. **SOC 2 Type II evidence.** Auditors require documented proof that FORM communicates availability events to affected parties (CC7.4) and that uptime commitments are backed by historical monitoring data (A1.1). A status page with 90 days of incident history before the observation period starts satisfies both controls simultaneously.

2. **Enterprise procurement.** Enterprise buyers verify operational maturity during security review. Absence of a public status page is a consistent blocker in procurement questionnaires. The Better Stack Statuspage link is referenced in enterprise MSAs (§7 of the SSO_SCIM_IMPLEMENTATION.md onboarding checklist) and in DPAs as the breach communication channel.

3. **Employee trust at scale.** Employees using FORM during active incidents need a single authoritative source — not a support email backlog. The status page is the single source of truth for incident state and expected recovery time.

**Scope of this section:** Architecture, component inventory, incident state taxonomy, update protocol, subscriber notifications, historical data archival, SOC 2 evidence mapping, and implementation checklist. This section does not define incident detection rules (see `docs/OBSERVABILITY.md §6`) or incident response procedures (see `docs/INCIDENT_RESPONSE.md §4`).

---

### 20.2 Architecture Decision: Better Stack Statuspage

#### 20.2.1 Options evaluated

| Option | Pros | Cons | Decision |
|---|---|---|---|
| **Better Stack Statuspage** (native) | Native integration with Better Stack monitors already planned (§9 OBSERVABILITY.md); auto-creates incidents from monitor failures; managed SLA dashboard; sub-15-second state propagation | Monthly cost (Better Stack Team plan ~$24/mo includes Statuspage) | ✅ **Selected for M4** |
| Custom (Cloudflare Worker + D1 + R2) | Zero marginal cost; full data control; composable with FORM's audit infrastructure | 2–3 eng-weeks build; maintenance burden; auditors prefer managed-tool paper trail | 🔄 Revisit at M9 if Better Stack pricing becomes material |
| Atlassian Statuspage | Industry standard; wide ecosystem support | $100+/month at team tier; vendor lock-in less aligned with FORM's Cloudflare-first stack | ❌ Rejected |

#### 20.2.2 DNS configuration

FORM operates `status.form.coach` as a CNAME to Better Stack Statuspage's custom domain endpoint:

```
status.form.coach.    CNAME    <tenant>.betterstack.com.    TTL 300
```

The CNAME is **proxied through Cloudflare** (`proxy: false` for this record, as Better Stack requires direct resolution for TLS certificate issuance). The Cloudflare zone for `form.coach` sets this record to DNS-only.

SSL/TLS: Better Stack provisions and auto-renews the certificate for `status.form.coach` via Let's Encrypt. FORM verifies certificate issuance and renewal as part of the monthly monitoring check (§15 compliance calendar).

#### 20.2.3 SOC 2 evidence anchor

Better Stack generates a hosted audit trail of monitor configuration changes, incident creation/resolution events, and subscriber notification delivery confirmations. FORM exports this audit trail quarterly to `compliance/status-page/YYYY-QQ-status-audit.json` in the private compliance repository (git-committed with SHA-256 hash registered in `compliance/checksums.sha256`). This export constitutes the SOC 2 CC7.4 evidence artifact for "communicates security incidents to external parties."

---

### 20.3 Monitored Components

Six components appear on the status page. Each maps to one or more SLOs from `docs/OBSERVABILITY.md §2`.

| Component | Display name | Check URL / method | Interval | SLO target | SOC 2 control |
|---|---|---|---|---|---|
| **API** | FORM API | `GET https://api.form.coach/health` → HTTP 200, body `{"status":"ok"}` | 30 s | 99.9% monthly | A1.1, A1.2 |
| **Auth** | Authentication | Synthetic probe: POST login with test credential (non-PII) → HTTP 200 + valid JWT in response | 60 s | 99.9% monthly | A1.1, CC6.1 |
| **Realtime sync** | Real-time sync | WebSocket connect to Supabase Realtime endpoint → heartbeat received ≤2 s | 60 s | 99.5% monthly | A1.1 |
| **CV processing** | CV pose processing | `GET https://api.form.coach/cv/health` → HTTP 200; checks on-device inference availability stub (server-side inference not in scope) | 120 s | 99.5% monthly | A1.1 |
| **Admin dashboard** | Admin dashboard | `GET https://admin.form.coach/health` → HTTP 200 | 60 s | 99.9% monthly | A1.1 |
| **Audit log delivery** | Audit log (enterprise) | Webhook delivery success rate metric from `docs/OBSERVABILITY.md §15.6`; alert fires if P95 > 5 s or delivery failure rate > 0.1% in 5-minute window | 5 min (metric check) | P95 < 5 s per `docs/ENTERPRISE.md` SLA | A1.1, CC7.4 |

**Anonymity of health probes.** The Auth synthetic probe uses a dedicated `status-probe@form.coach` internal account with no real user data. This account is excluded from aggregate wellness metrics and flagged `is_probe = true` in the `users` table. Access to its credentials is restricted to `devops-lead` via a dedicated 1Password vault entry (not shared with the general engineering vault). The account generates no billing events, no SCIM-provisioned identity, and no audit log entries visible to tenant admins.

#### 20.3.1 Component group mapping

Better Stack supports grouping components into logical groups. FORM's grouping:

```
Core Infrastructure
├── FORM API
└── Authentication

Data & Sync
└── Real-time sync

AI Features
└── CV pose processing

Enterprise
├── Admin dashboard
└── Audit log delivery
```

Enterprise customers are notified on incidents affecting **Enterprise** group components. All subscribers are notified on incidents affecting **Core Infrastructure**.

---

### 20.4 Incident State Taxonomy

Five states appear on the status page. State transitions are logged to Better Stack and archived per §20.7.

| State | Colour | Trigger | Auto-trigger from monitor? | Human confirmation required? |
|---|---|---|---|---|
| **Operational** | 🟢 Green | All checks passing; error rate < 0.1%; P95 latency within baseline | Yes — monitor recovery | No |
| **Degraded Performance** | 🟡 Yellow | Any component P95 latency > 2× baseline **or** error rate 0.1%–1% | Yes — from OBSERVABILITY.md §6 P2 alert | Yes — within 30 min; on-call engineer confirms |
| **Partial Outage** | 🟠 Orange | One non-core component unavailable; API and Auth remain operational | Yes — component monitor down | Yes — within 15 min; on-call engineer posts human update |
| **Major Outage** | 🔴 Red | API unavailable **or** Auth unavailable (blocks all users) | Yes — API or Auth monitor down for > 2 consecutive checks | Yes — **within 5 minutes**; founder / on-call lead posts update (P0 response) |
| **Under Maintenance** | 🔵 Blue | Scheduled maintenance window | No — manual creation only | Yes — created by `devops-lead` with 48h advance notice minimum |

**Fail-safe principle:** If Better Stack itself is degraded, the status page returns its last-known state cached by Cloudflare at the edge (TTL 30 s). Customers should treat "unable to reach status page" as a signal to check `https://betterstackstatus.com` for Better Stack platform health. This limitation is documented in enterprise MSAs.

---

### 20.5 Status Page Update Protocol

This protocol is referenced by `docs/INCIDENT_RESPONSE.md §6 Communication Templates`. The on-call engineer follows this protocol for all P0–P2 incidents. P3 incidents do not require status page updates unless they are customer-visible.

#### 20.5.1 Automated state transition (Better Stack)

Better Stack auto-creates a status page incident when a monitor transitions from **Operational** to any non-operational state for two consecutive checks. The auto-created incident:

- Sets the status page state to the appropriate level (Degraded / Partial / Major)
- Sends email notification to all subscribers (see §20.6)
- Remains in "Investigating" phase until an engineer posts the first human update

Auto-resolution occurs when all monitors return to **Operational** for three consecutive checks. The auto-resolution posts a "Resolved" update and sends subscriber notifications.

#### 20.5.2 Human update cadence (per INCIDENT_RESPONSE.md severity)

| Severity | First human update | Subsequent updates | Resolution update |
|---|---|---|---|
| **P0** (Major Outage — API/Auth down) | Within **5 minutes** of auto-detection | Every **15 minutes** until resolved | Immediately on resolution + 24h post-mortem link |
| **P1** (Partial Outage — one component, escalating) | Within **15 minutes** | Every **30 minutes** | Immediately on resolution + 48h post-mortem link |
| **P2** (Degraded Performance) | Within **1 hour** | Every **2 hours** if unresolved > 2h | On resolution; post-mortem optional |
| **Scheduled Maintenance** | 48 hours before window | N/A | On completion |

**Update format** (drawn from `docs/INCIDENT_RESPONSE.md §6` CC-01 template):

```
[Status] · [Timestamp UTC]
We are [investigating / monitoring / resolving] [component] [behaviour].
Impact: [who is affected, what they cannot do].
Next update: [timestamp].
```

The on-call engineer posts updates via the Better Stack dashboard or the Better Stack API (`POST /api/v3/status-pages/{id}/status-page-updates`). API credentials are stored in the `devops-lead` 1Password vault under `Better Stack Status API Key`.

#### 20.5.3 Maintenance window communication

Scheduled maintenance follows a 48-hour advance notice requirement, aligning with enterprise MSA commitments. The notice includes:

- Affected components
- Maintenance window start and end time (UTC)
- Expected service impact (brief degradation / full unavailability / zero impact)
- Customer-action required (if any)

Maintenance windows are created in Better Stack and visible as 🔵 blue periods in the uptime history graph, which auditors use to correctly exclude scheduled downtime from SLA calculations.

---

### 20.6 Subscriber Notifications

Subscribers receive notifications at status state changes. Better Stack supports three notification channels:

#### 20.6.1 Email

- Default channel for all subscribers
- Per-component subscription: subscribers can choose which component groups to follow (e.g., enterprise customers subscribe to **Enterprise** group only)
- Unsubscribe link in every email (CAN-SPAM + GDPR compliant)
- From address: `status@form.coach` (SPF + DKIM configured; no shared sending IP)

#### 20.6.2 RSS / Atom feed

- Available at `https://status.form.coach/feed.rss` (Better Stack native)
- Enterprise customers may integrate this into their internal tooling without a subscription account
- SOC 2 evidence: feed availability demonstrates CC7.4 "communicates security incidents" to any stakeholder without requiring account registration

#### 20.6.3 Slack webhook (enterprise customers)

Enterprise customers on the Growth and Enterprise tiers (per `docs/ENTERPRISE.md` tier table) may request a dedicated Slack webhook integration. Configuration:

1. Customer provides incoming webhook URL for their `#form-status` or equivalent channel
2. `devops-lead` stores the webhook URL in `audit_export_config` JSONB on the tenant's row (field: `status_webhook_url`) — same JSONB column used by the audit log export pipeline (`docs/OBSERVABILITY.md §15`)
3. Better Stack's outbound webhook is configured per-tenant pointing to a FORM Cloudflare Worker (`form-status-relay`) that fans out to all configured tenant webhook URLs
4. The relay Worker adds `X-FORM-Tenant-ID` header stripped before forwarding, and logs delivery to the audit log as `system.status_notification_sent`

**Privacy note:** The relay Worker does not log or persist the outbound webhook payload. The payload contains only: `{ component, state, message, timestamp_utc }` — no user data, no tenant-specific analytics.

---

### 20.7 Historical Incident Data Retention & SOC 2 Evidence

#### 20.7.1 Better Stack retention

Better Stack retains incident history for 365 days in the UI (Team plan). The status page at `status.form.coach` displays the trailing 90 days of uptime history — the minimum required before the SOC 2 observation period begins (PRE-10 and PRE-11).

#### 20.7.2 FORM archival to R2 (7-year retention)

SOC 2 Type II requires evidence spanning the full observation period plus a retention buffer. FORM supplements Better Stack's 365-day retention with a monthly export to Cloudflare R2:

**Export cron:** `form-status-archiver` Cloudflare Worker, Cron Trigger: `0 05 1 * *` (1st of each month, 05:00 UTC — 2 hours after the cold backup archiver).

**Export target:** `r2://form-backups/status-history/YYYY-MM/`

**Export contents:**

| File | Contents | Format |
|---|---|---|
| `incidents-YYYY-MM.json` | All incident records in the month: component, start time, end time, severity, update timeline | JSON, CloudEvents-adjacent schema |
| `uptime-YYYY-MM.json` | Per-component uptime percentage, total downtime seconds, SLO delta | JSON |
| `subscribers-YYYY-MM.json` | Subscriber count per component group (no email addresses) | JSON |
| `notifications-YYYY-MM.json` | Notification delivery log: channel type, delivered count, failed count (no recipient data) | JSON |

**HMAC integrity:** Each export file is signed with `HMAC-SHA256` using the monthly key `STATUS_ARCHIVE_KEY_YYYYMM` (same key rotation cadence as cold backup, stored in 1Password and Cloudflare Workers Secrets). The signature is appended as `incidents-YYYY-MM.json.sig`. This ensures the compliance evidence cannot be silently modified post-export.

**R2 lifecycle rule:** `status-history/` prefix — object lifecycle 7 years (2557 days), consistent with the audit log retention policy (`docs/AUDIT_LOG_SCHEMA.md §Retention`).

#### 20.7.3 Evidence filing for SOC 2

| SOC 2 Control | Evidence artifact | Source | Filed by |
|---|---|---|---|
| **A1.1** — Availability commitments | Uptime percentage per component vs. SLO target (§20.3 table) | `uptime-YYYY-MM.json` exports | devops-lead, monthly |
| **A1.2** — Environmental protections | Monitor configuration screenshots (check interval, alert threshold, component grouping) | Better Stack dashboard export | devops-lead, at observation start |
| **CC7.4** — Communicates security incidents | Incident timeline exports + subscriber notification delivery confirmations | `incidents-YYYY-MM.json` + `notifications-YYYY-MM.json` | devops-lead, monthly |
| **CC6.8** — Prevents unauthorized access (maintenance) | Scheduled maintenance records with 48h advance notice confirmation | Better Stack maintenance event log | devops-lead, per event |

All evidence artifacts are stored in `compliance/status-page/` in the private compliance repository (separate from the public `form` repository). SHA-256 hashes of all artifacts are registered in `compliance/checksums.sha256` at the time of filing.

---

### 20.8 Sub-Processor List Publication

The SOC 2 and GDPR critical gap "sub-processor list published" (§2 Gap Analysis Summary; CC9.2, GDPR Art. 13(1)(e)) requires FORM to make the sub-processor list from §17 publicly accessible. This is a transparency obligation, not an operational implementation gap.

**Publication path:** `https://security.form.coach/sub-processors`

Implementation: a single-page Cloudflare Worker at `security.form.coach/sub-processors` renders the sub-processor register from §17 in human-readable HTML and machine-readable JSON (`?format=json`). The page includes:

- Sub-processor name, country of processing, service category, certification (SOC 2 / ISO 27001 / SCCs)
- "Last updated" timestamp (updated whenever §17 Vendor Risk Registry changes)
- Link to DPA template (`security.form.coach/dpa`)

The JSON endpoint (`security.form.coach/sub-processors.json`) allows enterprise procurement teams to ingest the list programmatically and detect changes (monitored via hash comparison).

**Change notification:** When a new sub-processor is added or removed, FORM notifies existing enterprise customers via status page announcement and a direct email to the tenant admin (`tenant_admin_email` from `tenant_sso_configs`), with 30 days advance notice before the new sub-processor begins processing data (GDPR Art. 28(2) requirement).

**SOC 2 evidence:** The existence of `security.form.coach/sub-processors` with a datestamped list closes CC9.2 ("Manages vendor and business partner risk") as a public commitment. Screenshots of the published page are filed in `compliance/cc9/sub-processor-page-YYYY-MM.png`.

**Gap closure status:** This section provides the architecture and implementation path. The gap moves from 🔴 Critical to 🟡 Partial upon implementation of the Worker; moves to 🟢 Done when the page is live and filed as CC9.2 evidence.

---

### 20.9 Gap Closure Table

| Gap | Status before §20 | Status after §20 | Remaining work |
|---|---|---|---|
| Status page (status.form.coach) | 🔴 Critical gap | 🟡 Partial (architecture + checklist defined) | Deploy Better Stack Statuspage, configure CNAME, add 6 components, activate subscriber notifications |
| Uptime monitoring with historical data | 🔴 Gap | 🟡 Partial (architecture defined; 90-day history requires ≥90 days runtime) | PRE-10: provision Better Stack monitors; start clock |
| Sub-processor list published | 🔴 Critical gap | 🟡 Partial (publication path defined; Worker not yet deployed) | Deploy `security.form.coach/sub-processors` Worker |
| CC7.4 — Communicates security incidents | 🟡 Gap (templates only) | 🟡 Partial (templates + automated notification channel defined) | Status page live; first incident notification delivered |
| A1.1 — Uptime SLO backed by monitoring | 🟡 Gap | 🟡 Partial (SLO targets + monitoring architecture specified in §20.3) | Better Stack monitors running for ≥30 days before observation |

**Readiness impact:** Critical gaps: 3 → 2 (status page: 🔴 → 🟡 Partial). Availability controls in place: +2. Readiness: ~63% → ~65%.

---

### 20.10 Implementation Checklist (devops-lead)

Execute in the order listed. PRE-10 and PRE-11 are observation-period gates — neither can be marked 🟢 Done until the monitor history clock has run for the required duration.

| # | Task | Priority | Milestone | Notes |
|---|---|---|---|---|
| 20-01 | Provision Better Stack Team plan | P0 | M4 | Required for Statuspage + uptime monitors in one account |
| 20-02 | Create status page at Better Stack; configure custom domain `status.form.coach` | P0 | M4 | Better Stack will generate TLS verification record for Let's Encrypt |
| 20-03 | Add CNAME `status.form.coach → <tenant>.betterstack.com` in Cloudflare zone (proxy: DNS-only) | P0 | M4 | Do NOT proxy through Cloudflare; Better Stack needs direct resolution for TLS |
| 20-04 | Create `status-probe@form.coach` internal account with `is_probe = true`; store credentials in 1Password `devops-lead` vault | P0 | M4 | Required before Auth synthetic probe can run; exclude from SCIM and from aggregate metrics |
| 20-05 | Configure 6 monitors (§20.3 table): API, Auth synthetic, Realtime WebSocket, CV health, Admin dashboard, Audit log delivery metric | P0 | M4 | Set check intervals exactly as specified; name components to match display names in §20.3 |
| 20-06 | Add component groups: Core Infrastructure, Data & Sync, AI Features, Enterprise (§20.3.1) | P0 | M4 | Group membership drives which subscribers are notified per incident |
| 20-07 | Enable email subscriber notifications; configure `From: status@form.coach`; verify SPF + DKIM for `form.coach` sending domain | P0 | M4 | Status email is the primary CC7.4 communication channel |
| 20-08 | Configure RSS/Atom feed (native Better Stack feature — enable in Statuspage settings) | P1 | M4 | No code required |
| 20-09 | Deploy `form-status-archiver` Cloudflare Worker with monthly Cron Trigger and R2 write | P1 | M4 | R2 bucket `form-backups` must exist first (§19-01) |
| 20-10 | Provision `STATUS_ARCHIVE_KEY_YYYYMM` in Workers Secrets; escrow in 1Password `compliance-officer` vault | P1 | M4 | Use same rotation cadence as cold backup key (§19 HMAC) |
| 20-11 | Deploy `form-status-relay` Cloudflare Worker for enterprise Slack webhook fan-out | P2 | M5 | Required only when first enterprise customer requests Slack integration |
| 20-12 | Add `status_webhook_url` field to `audit_export_config` JSONB schema (§15 OBSERVABILITY.md migration) | P2 | M5 | Co-ordinate with platform-engineer; non-nullable default `null` |
| 20-13 | Deploy `security.form.coach/sub-processors` Worker rendering §17 vendor registry in HTML + JSON | P0 | M4 | Closes GDPR Art. 13(1)(e) and CC9.2 sub-processor publication gap |
| 20-14 | File monitor configuration screenshots as A1.2 evidence in `compliance/status-page/` | P0 | M4 | File immediately after step 20-05 completes |
| 20-15 | Notify all enterprise pilot accounts of `status.form.coach` launch; add link to onboarding welcome email template | P1 | M4 | Add link to `docs/SSO_SCIM_IMPLEMENTATION.md §7` onboarding checklist |
| 20-16 | Mark PRE-10 🟢 Done only after ≥30 days of continuous monitor operation | P0 | M5 (30 days post-M4) | Clock starts when 20-05 is complete |
| 20-17 | Mark PRE-11 🟢 Done only after ≥90 days of continuous status page history visible at `status.form.coach` | P0 | M6–M7 | Clock starts when 20-02 + 20-03 are complete |

---

**v1.0 · травень 2026 · owner: compliance-officer + security-engineer + enterprise-architect**
**Review cadence: quarterly. Next review: серпень 2026.**

*v0.2 additions: Sub-Processor Register (CC9, GDPR Art. 28), Complementary User Entity Controls (CUECs), Common Security Questionnaire Responses (CAIQ/SIG Lite pre-answers).*
*v0.3 additions: Section 13 — Data Classification Policy (four-tier: Public / Internal / Confidential / Restricted). Closes SOC 2 gap C1.1. Critical gaps: 11 → 10. Controls in place: 22 → 23.*
*v0.4 additions: Section 14 — Formal Risk Register (18 risks across 6 categories: Security, Availability, Processing Integrity, Confidentiality, Privacy, Vendor/Operational). L×S scoring with residual scores and named owners. Closes CC3 gap (formal risk assessment documented). Updated P3 DPIA status to ✅ Done. Critical gaps: 10 → 9. Partial: 22 → 21. Controls in place: 23 → 25. Readiness: 42% → 45%.*
*v0.5 additions (two-step catch-up): (a) `docs/PRIVACY_POLICY.md` v0.1-draft shipped (v0.55.0 CHANGELOG) — closes P1.1, P1.2, CC9.2+P6.1; critical gaps: 9 → 6; controls in place: 25 → 28; readiness: 45% → ~51%. (b) Section 15 — Annual Compliance Calendar: 12-month master calendar (16 recurring activities, all mapped to SOC 2 controls + evidence artifacts), Pre-Observation Period Readiness Checklist (27 PRE items with status), First-Year Implementation Priority matrix (solo-founder vs. post-hire split, compensating controls documented). Moves 7 gaps from 🔴 Gap → 🟡 Partial (security training scheduled Q1-Feb, vendor review Q1-Jan, DR drill Q1-Jan, privacy review Q1-Jan, offboarding quarterly cadence, media disposal Q2-Jun, control effectiveness review quarterly). Critical gaps: 6 → 4 (security training and offboarding: schedule + owner + evidence defined → 🟡 Partial). Partial: 21 → 28. Controls in place: 28 (unchanged — scheduled but not yet executed). Readiness: ~51% → ~55%.*
*v0.6 additions: Section 16 — Penetration Test Program. Scope (API, auth flows, SSO/SCIM, RLS-via-API, mobile apps, Cloudflare edge), methodology (OWASP WSTG + ASVS L2 + MASVS L1/L2 + PTES + CWE Top 25), finding severity SLAs (Critical 24h → High 7d → Medium 30d), health-data and tenant-isolation severity uplift rules, remediation tracking workflow (Linear tickets → PR → re-test → compliance evidence filing), SOC 2 evidence package definition (engagement letter + full report + HMAC chain verification), customer disclosure policy (executive summary under NDA for >$50k ACV). CC7 control table updated to add "External penetration test" row (🟡 Partial). PRE-21 moved from 🔴 Open → 🟡 Partial. Open Items updated. CC7.1 and CC7.2 moved from 🔴 Gap → 🟡 Partial. Critical gaps: 4 → 3. Partial: 28 → 30. Readiness: ~55% → ~56%.*

*v0.7 additions: Section 17 — Vendor Security Review Process. Closes two documented 🔴 Gaps: "Vendor security review process" and "Annual vendor security review." Three-tier risk classification (Critical/High/Standard) with review frequency per tier. Vendor Risk Registry covering 11 vendors (8 sub-processors + Better Uptime, PagerDuty, Linear) with DPA status, certification level, risk score, and owner. 5-step Initial Vendor Assessment checklist with DPA gate and approval veto. 5-step January Annual Review process with SOC 2 CC9.2 evidence package definition. 6-factor risk scoring matrix (composite 🟢/🟡/🟠/🔴 scale, escalation rules for DPA-missing and cert-lapse). 7-step new sub-processor addition workflow with emergency exception clause. Termination and offboarding process with 7-year evidence retention. SOC 2 control mapping: CC9.1, CC9.2, CC9.3, P8.1. Gap closure: "Vendor security review process" 🔴 → 🟡 Partial (first annual review Q1-2027); "Annual vendor security review" 🔴 → 🟡 Partial; "Vendor risk registry" 🟡 Partial → 🟡 Partial (formalized with scoring). Critical gaps: 3 → 1. Readiness: ~56% → ~58%.*

*v0.8 additions: Section 18 — Business Continuity & Disaster Recovery (BCP/DRP). Closes CC7.5 and A1.1 runbook gap. RTO/RPO commitments defined per tier (Enterprise 4h RTO / 1h RPO; Pro 8h / 4h). Four failure scenarios with step-by-step response procedures (Supabase unavailability, Cloudflare outage, data corruption, nuke scenario). Annual DR drill procedure with evidence template for SOC 2 CC7.5. Communication tree (internal, enterprise, consumer) per incident phase. Cold storage backup gap newly documented as 🔴 (B2 export not yet implemented). CC7.5: 🔴 → 🟡 Partial. DR runbook: 🟡 Partial → 🟢 Done. Readiness: ~58% → ~60%.*

*v1.0 additions: Section 20 — Status Page Architecture & Availability Communication. Closes critical gap: "Status page (status.form.coach)" 🔴 → 🟡 Partial. Architecture: Better Stack Statuspage with CNAME status.form.coach; 6 monitored components (API, Auth, Realtime, CV processing, Admin dashboard, Audit log delivery); 5 incident states; update protocol with 15-minute human-update SLA for P0/P1 (linked to INCIDENT_RESPONSE.md CC-01 template); subscriber notifications (email, RSS, Slack webhook for enterprise); incident history archival to R2 (7-year retention, SOC 2 evidence); sub-processor list publication path (security.form.coach/sub-processors). SOC 2 evidence mapping: A1.1, A1.2, CC7.4, CC6.8. Critical gaps: 3 → 2. Partial: +2 (uptime monitoring + status page). Readiness: ~63% → ~65%.*
**Review cadence: quarterly. Next review: серпень 2026.**

*v0.2 additions: Sub-Processor Register (CC9, GDPR Art. 28), Complementary User Entity Controls (CUECs), Common Security Questionnaire Responses (CAIQ/SIG Lite pre-answers).*
*v0.3 additions: Section 13 — Data Classification Policy (four-tier: Public / Internal / Confidential / Restricted). Closes SOC 2 gap C1.1. Critical gaps: 11 → 10. Controls in place: 22 → 23.*
*v0.4 additions: Section 14 — Formal Risk Register (18 risks across 6 categories: Security, Availability, Processing Integrity, Confidentiality, Privacy, Vendor/Operational). L×S scoring with residual scores and named owners. Closes CC3 gap (formal risk assessment documented). Updated P3 DPIA status to ✅ Done. Critical gaps: 10 → 9. Partial: 22 → 21. Controls in place: 23 → 25. Readiness: 42% → 45%.*
*v0.5 additions (two-step catch-up): (a) `docs/PRIVACY_POLICY.md` v0.1-draft shipped (v0.55.0 CHANGELOG) — closes P1.1, P1.2, CC9.2+P6.1; critical gaps: 9 → 6; controls in place: 25 → 28; readiness: 45% → ~51%. (b) Section 15 — Annual Compliance Calendar: 12-month master calendar (16 recurring activities, all mapped to SOC 2 controls + evidence artifacts), Pre-Observation Period Readiness Checklist (27 PRE items with status), First-Year Implementation Priority matrix (solo-founder vs. post-hire split, compensating controls documented). Moves 7 gaps from 🔴 Gap → 🟡 Partial (security training scheduled Q1-Feb, vendor review Q1-Jan, DR drill Q1-Jan, privacy review Q1-Jan, offboarding quarterly cadence, media disposal Q2-Jun, control effectiveness review quarterly). Critical gaps: 6 → 4 (security training and offboarding: schedule + owner + evidence defined → 🟡 Partial). Partial: 21 → 28. Controls in place: 28 (unchanged — scheduled but not yet executed). Readiness: ~51% → ~55%.*
*v0.6 additions: Section 16 — Penetration Test Program. Scope (API, auth flows, SSO/SCIM, RLS-via-API, mobile apps, Cloudflare edge), methodology (OWASP WSTG + ASVS L2 + MASVS L1/L2 + PTES + CWE Top 25), finding severity SLAs (Critical 24h → High 7d → Medium 30d), health-data and tenant-isolation severity uplift rules, remediation tracking workflow (Linear tickets → PR → re-test → compliance evidence filing), SOC 2 evidence package definition (engagement letter + full report + HMAC chain verification), customer disclosure policy (executive summary under NDA for >$50k ACV). CC7 control table updated to add "External penetration test" row (🟡 Partial). PRE-21 moved from 🔴 Open → 🟡 Partial. Open Items updated. CC7.1 and CC7.2 moved from 🔴 Gap → 🟡 Partial. Critical gaps: 4 → 3. Partial: 28 → 30. Readiness: ~55% → ~56%.*

*v0.7 additions: Section 17 — Vendor Security Review Process. Closes two documented 🔴 Gaps: "Vendor security review process" and "Annual vendor security review." Three-tier risk classification (Critical/High/Standard) with review frequency per tier. Vendor Risk Registry covering 11 vendors (8 sub-processors + Better Uptime, PagerDuty, Linear) with DPA status, certification level, risk score, and owner. 5-step Initial Vendor Assessment checklist with DPA gate and approval veto. 5-step January Annual Review process with SOC 2 CC9.2 evidence package definition. 6-factor risk scoring matrix (composite 🟢/🟡/🟠/🔴 scale, escalation rules for DPA-missing and cert-lapse). 7-step new sub-processor addition workflow with emergency exception clause. Termination and offboarding process with 7-year evidence retention. SOC 2 control mapping: CC9.1, CC9.2, CC9.3, P8.1. Gap closure: "Vendor security review process" 🔴 → 🟡 Partial (first annual review Q1-2027); "Annual vendor security review" 🔴 → 🟡 Partial; "Vendor risk registry" 🟡 Partial → 🟡 Partial (formalized with scoring). Critical gaps: 3 → 1. Readiness: ~56% → ~58%.*

*v0.8 additions: Section 18 — Business Continuity & Disaster Recovery (BCP/DRP). Closes CC7.5 and A1.1 runbook gap. RTO/RPO commitments defined per tier (Enterprise 4h RTO / 1h RPO; Pro 8h / 4h). Four failure scenarios with step-by-step response procedures (Supabase unavailability, Cloudflare outage, data corruption, nuke scenario). Annual DR drill procedure with evidence template for SOC 2 CC7.5. Communication tree (internal, enterprise, consumer) per incident phase. Cold storage backup gap newly documented as 🔴 (B2 export not yet implemented). CC7.5: 🔴 → 🟡 Partial. DR runbook: 🟡 Partial → 🟢 Done. Readiness: ~58% → ~60%.*

*v1.0 additions: Section 20 — Status Page Architecture & Availability Communication. Closes critical gap: "Status page (status.form.coach)" 🔴 → 🟡 Partial. Architecture: Better Stack Statuspage with CNAME status.form.coach; 6 monitored components (API, Auth, Realtime, CV processing, Admin dashboard, Audit log delivery); 5 incident states; update protocol with 15-minute human-update SLA for P0/P1 (linked to INCIDENT_RESPONSE.md CC-01 template); subscriber notifications (email, RSS, Slack webhook for enterprise); incident history archival to R2 (7-year retention, SOC 2 evidence); sub-processor list publication path (security.form.coach/sub-processors). SOC 2 evidence mapping: A1.1, A1.2, CC7.4, CC6.8. Critical gaps: 3 → 2. Partial: +2 (uptime monitoring + status page). Readiness: ~63% → ~65%.*


---

## 21. CC8 — Change Management Controls: Formal Policy & Evidence Framework

> Owner: `compliance-officer` + `security-engineer`. Effective: May 2026. Review: annually or on any material change to deployment architecture or team composition.
> SOC 2 controls: **CC8.1** (authorises, tests, and documents changes before implementation; classifies changes by type; maintains rollback capability).
> Reference: DEC-010 (version-controlled repo), DEC-030 (HMAC-chained audit log), `docs/ENGINEERING_RUNBOOK.md`, `docs/DATA_MODEL.md §10–11`.

---

### 21.1 Purpose and SOC 2 Mapping

CC8.1 requires that changes to infrastructure and software are authorised, tested, and documented before they enter production. For FORM, this is not an abstract policy requirement — unauthorized or untested changes to the components that process health data create direct risks under the Processing Integrity (PI) and Confidentiality (C) trust service criteria:

- An unauthorized schema migration could alter RLS policies governing health data access, triggering a C1 (confidentiality) failure
- An untested change to the workout data processing pipeline could produce incorrect coaching outputs, triggering a PI1 (processing accuracy) failure
- A change deployed without rollback capability could extend an outage beyond the enterprise RTO commitment of 4 hours (A1.1)

**Scope of this section.** All production changes to:

| Surface | Technology | Notes |
|---|---|---|
| API layer | Cloudflare Workers | Worker scripts and routes |
| Database schema | Supabase (PostgreSQL migrations) | All DDL changes via `/supabase/migrations/` |
| Frontend | Cloudflare Pages | Web app builds |
| ML pipeline | Model files in Cloudflare R2; version pointer in KV | Inference path changes |
| Auth configuration | Supabase Auth settings, SAML/OIDC config | SSO policy changes |

Out of scope: changes to internal tooling (Linear, 1Password, Notion) that carry no production data; open-source dependency updates that are gated separately by Dependabot + CI.

---

### 21.2 Change Classification

All production changes fall into one of three tiers. Tier determines authorization requirements, deployment SLA, and evidence obligations.

| Type | Definition | Authorization Required | Max Deployment SLA | Examples |
|---|---|---|---|---|
| **Emergency (P0)** | Critical security patch, active breach response, production outage — time to deploy is measured in hours not days | Single-approver fast path; mandatory post-hoc review within 24 hours; incident ticket must be opened simultaneously | 2 hours from decision to deploy | SEV-0 CVE patch, RLS bypass hotfix, production-down config restore |
| **Standard** | Feature work, refactor, new schema migration, dependency update, auth configuration change | PR with written rationale in PR body; solo phase: founder self-review with documented justification (see §21.7); post-hire: at minimum 1 independent reviewer | 24–72 hours from PR open to deploy | New API endpoint, schema migration, model version update, Cloudflare Worker update |
| **Minor** | Copy/content changes, documentation, configuration changes with no security impact and no data model effect | PR merge + auto-deploy; no separate approval step required | Immediate on merge | Marketing copy, docs update, non-sensitive env var tuning |

**Ambiguous changes default upward.** If it is not clear whether a change is Minor or Standard — particularly if it touches any file that handles Restricted or Confidential data (per §13 classification) — treat it as Standard and document the rationale.

---

### 21.3 Branch Protection Configuration (GitHub)

The settings below must be configured on the `main` branch of `github.com/Rbit27/form`. The screenshot of this configuration is SOC 2 evidence artifact CC8-E-001 and must be re-captured any time settings change.

| Setting | Value | Rationale |
|---|---|---|
| Require pull request before merging | Enabled | Direct push to main disabled; every change has a PR record |
| Required number of approvals | 1 | Solo phase: founder; post-hire: any approved engineer |
| Dismiss stale reviews when new commits are pushed | Yes | Prevents approving a safe commit then pushing a different one |
| Require status checks to pass before merging | Yes (when CI is configured — see CC8-GAP-001) | Required CI checks: `ci/build`, `ci/test`, `ci/lint` |
| Require branches to be up to date before merging | Yes | Prevents merge-race conditions |
| Require linear history | Yes | Clean git history aids forensic review during incidents |
| Include administrators | No | Solo-founder compensating control — see §21.8 for override logging requirement |

**Current state vs. target state:**

| Setting | Current state | Target state | Gap reference |
|---|---|---|---|
| Require pull request | Must be configured | Configured + screenshot filed as CC8-E-001 | PRE-14 |
| Required CI checks | No CI pipeline exists | `ci/build`, `ci/test`, `ci/lint` required on merge | CC8-GAP-001 |
| Include administrators | Off (compensating control) | On (post-hire, when P0 emergency path uses §21.2 fast track) | CC8-GAP-002 |

**Evidence artifact CC8-E-001:** Screenshot of branch protection settings page for `main` at `github.com/Rbit27/form/settings/branches`. Filed to `/evidence/cc8/branch-protection-YYYY-MM-DD.png` in the private compliance repository. Refresh cadence: quarterly review; re-file immediately on any settings change.

---

### 21.4 CI/CD Pipeline as Change Control

Automated CI is a primary CC8.1 control — it provides an objective, repeatable test gate that cannot be bypassed by a single human approving their own work.

**Current state vs. target state:**

| Dimension | Current state | Target state |
|---|---|---|
| Build gate | Manual — `wrangler deploy` run locally or from Cloudflare dashboard | GitHub Actions workflow: `push` to PR branch triggers build |
| Test gate | No automated test execution on PR | GitHub Actions: `ci/test` runs full test suite; failing test blocks merge |
| Lint gate | No automated lint enforcement on PR | GitHub Actions: `ci/lint` runs ESLint + type checking; failures block merge |
| Deploy on merge | Manual — engineer runs `wrangler deploy` post-merge | GitHub Actions: merge to `main` triggers deploy job; deploy log archived |
| Deployment log | No structured archive | GitHub Actions run log + deploy hash stored; archived to R2 per deploy (CC8-E-002) |

**Gap: CC8-GAP-001.** No CI pipeline is currently configured. This is a documented gap. Until CI is live, the compensating control is the manual pre-deploy checklist in `docs/ENGINEERING_RUNBOOK.md`. Every deploy executed under the manual path must confirm checklist completion before pushing.

**Evidence artifact CC8-E-002** (target state, once CI is live): GitHub Actions run log for each production deploy. Each log entry captures: commit SHA, branch, triggering PR number, test results, lint results, deploy timestamp, Cloudflare deploy hash. Archived to `r2://form-backups/ci-logs/YYYY/MM/` per deploy by the `form-ci-archiver` step in the Actions workflow. Refresh cadence: automated per deploy.

---

### 21.5 Database Schema Change Controls

Schema changes carry the highest risk of any change type — a bad migration can corrupt RLS policies, destroy data, or break tenant isolation in ways that are difficult to reverse quickly.

**Rules (all mandatory, no exceptions):**

1. All schema changes via versioned migration files in `/supabase/migrations/`. Direct `ALTER` or `CREATE` statements run from the Supabase dashboard console are prohibited in production.
2. Every migration file is PR-reviewed under the same authorization requirements as a Standard change (§21.2) regardless of apparent simplicity.
3. Every `up` migration must have a corresponding `down` migration (rollback script) in the same PR. A PR that adds an up migration without a down migration fails review.

**Migration review checklist (reviewer responsibility):**

| Check | Requirement |
|---|---|
| RLS policies | Any new table or altered table must have RLS enabled and the appropriate policy applied before the migration is merged. Confirm `ALTER TABLE <name> ENABLE ROW LEVEL SECURITY` is present. |
| Tenant isolation | Every new table that stores user or tenant data must have a `tenant_id` foreign key referencing `tenants(id)`. Confirm presence in the migration DDL. |
| Index strategy | Any new table with expected query volume must have indexes documented in a comment in the migration or in `docs/DATA_MODEL.md §11`. Unindexed foreign keys on large tables = production latency incident. |
| Backward compatibility | If the migration is not backward-compatible (e.g., dropping a column that application code still reads), confirm a coordinated deploy plan: new code that handles both old and new schema is deployed first, migration runs, old code path removed in a subsequent PR. |
| Down migration tested | Reviewer confirms down migration has been tested in a staging environment, not just written. |

**Rollback procedure for schema changes:**

- **Within 30 minutes of deploy:** run the down migration script from the PR. Confirm row counts are consistent. Log the rollback as `system.config_changed` in the audit log with `metadata.reason = "migration_rollback"`.
- **Beyond 30 minutes / data already written:** use Supabase PITR. Restore to a timestamp immediately before the migration ran. Follow `docs/DATA_MODEL.md §10` PITR runbook. RTO target: 15 minutes for rollback decision; PITR restore within 4 hours.

**Evidence artifact CC8-E-003:** Migration file git history — `git log supabase/migrations/` — provides a complete, tamper-evident record of every schema change with author, timestamp, and PR link. Automated; no manual filing required.

**Evidence artifact CC8-E-004:** PITR restore test record. Quarterly test of Supabase PITR targeting the schema migration recovery path. Filed to `/evidence/cc8/pitr-test-YYYY-MM.md`. Owner: `devops-lead`.

---

### 21.6 Rollback and Revert Procedures

All production surfaces must have a documented, tested rollback path with a target RTO of 15 minutes for emergency rollback decisions.

| Surface | Rollback method | RTO | Notes |
|---|---|---|---|
| **Cloudflare Workers (API)** | Cloudflare dashboard → Workers → select deployment → "Rollback to this version"; or CLI: `wrangler rollback` | < 5 minutes | Previous deployment is retained for 24 hours by Cloudflare. After 24 hours, earlier versions require a re-deploy from git. |
| **Cloudflare Pages (frontend)** | Cloudflare Pages dashboard → Deployments → select any previous deployment → "Rollback to this deployment" | < 5 minutes | All historical deployments are permanently retained; any version is reachable. |
| **Supabase schema (DDL)** | Run down migration script from the PR; or invoke PITR for the data as a fallback (§21.5) | < 15 minutes (down migration); < 4 hours (PITR) | Down migration required for every up migration — non-negotiable per §21.5 rule 3. |
| **ML model** | Update `model_version` pointer in Cloudflare KV store to the prior version's key; model files versioned in R2 | < 5 minutes | Model files in R2 are never overwritten — new version is a new key. Rollback = KV update only. |

**Quarterly rollback test.** Each rollback path above must be tested in a staging environment quarterly. Results documented and filed alongside the DR drill evidence. Confirmed RTO must be at or below the targets above; any exceedance is a gap finding requiring remediation within 30 days.

---

### 21.7 Separation of Duties — Solo-Founder Analysis

This section documents the structural gap honestly. SOC 2 auditors reviewing a pre-revenue solo-founder company will not fail a Type I readiness assessment solely on this basis, but the gap must be disclosed in the Management Assertion Letter with compensating controls documented.

**The structural gap:** One person (the founder) can author code, approve the PR, and execute the production deploy. No technical control presently prevents this. This is a structural limitation of a team of one and cannot be fully compensated by process alone.

**Compensating controls in place:**

1. All changes to production go through a PR, even from the sole author. This creates a mandatory review checkpoint and an immutable git history entry. There is no mechanism to deploy code that was never in a PR.
2. Written rationale is mandatory in the PR body for all Standard and Emergency changes. The rationale must describe: what is changing, why it is changing, what the rollback plan is, and whether any security-relevant surface (auth, RLS, health data processing) is affected.
3. Automated test gate (target: once CC8-GAP-001 is resolved) reduces the human error surface. A failing test blocks deploy regardless of who authored the change.
4. The audit log is append-only and HMAC-chained (DEC-030). No deployment event can be edited or deleted after the fact. An auditor can independently verify the complete deployment history.

**Auditor disclosure guidance.** The Management Assertion Letter for the SOC 2 Type I or Type II report must include a paragraph disclosing the separation of duties limitation and listing these four compensating controls. Auditors at firms experienced with early-stage SaaS companies (Prescient Assurance, Johanson Group) routinely accept this pattern as a finding-with-compensating-controls rather than a qualified opinion, provided the compensating controls are demonstrably operational throughout the observation period.

**Post-hire target.** The moment a second engineer joins the team: GitHub branch protection is updated to require review from a code owner distinct from the PR author; the `CODEOWNERS` file is created mapping production-path directories to the first engineering hire as a required reviewer; this gap moves from 🟡 Partial to 🟢 Done.

---

### 21.8 Include-Administrators Override (Emergency Exception)

GitHub's "Include administrators" branch protection setting is deliberately set to **off** during the solo-founder phase. The reason: if the branch protection requires a reviewer other than the author, and the founder is the only engineer, a P0 production outage cannot be remediated without the override. Turning "Include administrators" on would lock the founder out of the fast path at the worst possible moment.

This is a documented exception, not an oversight. The following compensating controls govern every use of the admin override:

1. Within 30 minutes of using the admin override to merge a PR without review: create a GitHub Issue tagged with the label `admin-override`. The issue must reference: the PR number, the incident ticket that justified the override (if Emergency change per §21.2), and the planned post-hoc review date.
2. Post-hoc review within 24 hours: the founder reviews the merged change as if they were an independent reviewer. Any concerns identified are tracked as follow-up issues. The `admin-override` issue is updated with the review outcome.
3. Every `admin-override` issue is reviewed at the next quarterly access review (§15.1) to confirm the post-hoc review occurred within the 24-hour SLA.

**Evidence artifact CC8-E-005:** All GitHub Issues tagged `admin-override` at `github.com/Rbit27/form/issues?label=admin-override`. Refresh cadence: per occurrence. Owner: founder.

**Post-hire target.** After the first engineering hire: set "Include administrators" to **on**. Emergency changes (§21.2) follow the 1-reviewer fast path using the hire as the second person. The admin-override exception is retired.

---

### 21.9 Evidence Package for SOC 2 Auditors

| Evidence ID | Description | Location | Refresh Cadence | Owner |
|---|---|---|---|---|
| **CC8-E-001** | Branch protection screenshot — `main` branch settings at `github.com/Rbit27/form` | `/evidence/cc8/branch-protection-YYYY-MM-DD.png` in private compliance repo | Quarterly, or immediately on any settings change | founder |
| **CC8-E-002** | CI/CD run log per production deploy — test results, lint results, deploy hash, commit SHA | GitHub Actions run history + archived to `r2://form-backups/ci-logs/YYYY/MM/` per deploy | Per deploy (automated once CI is live; CC8-GAP-001) | automated |
| **CC8-E-003** | Migration file git history — complete DDL change history with author, timestamp, PR link | `git log supabase/migrations/` at `github.com/Rbit27/form` | Per deploy (automated — git history) | automated |
| **CC8-E-004** | PITR restore test record — quarterly confirmation that schema rollback via PITR achieves RTO target | `/evidence/cc8/pitr-test-YYYY-MM.md` in private compliance repo | Quarterly | devops-lead |
| **CC8-E-005** | Admin override issue log — all GitHub Issues tagged `admin-override` | `github.com/Rbit27/form/issues?label=admin-override` | Per occurrence | founder |
| **CC8-E-006** | PR merge history — complete record of every change merged to `main`, including author, reviewer (if applicable), and linked CI status | `github.com/Rbit27/form/pulls?state=closed` + GitHub audit log export | Per change (automated) | automated |

**Evidence collection note for auditors.** CC8-E-002 is currently unavailable because the CI pipeline (CC8-GAP-001) is not yet configured. The compensating control for the observation period prior to CI activation is the manual pre-deploy checklist in `docs/ENGINEERING_RUNBOOK.md`. Checklist completions are recorded as comments on the corresponding PR before merge. Auditors will find this evidence in the PR comment history (CC8-E-006).

---

### 21.10 Control Status Update

This section closes two documented gaps and upgrades one from red to partial. The CC8 control table in §1 is updated as follows:

| Control | Previous status | Status after §21 | Notes |
|---|---|---|---|
| All changes via version-controlled repo | ✅ Done | ✅ Done | No change — GitHub public repo, DEC-010 |
| Production deploy requires CI pass | 🟡 Gap | 🟡 Gap | Gap formally documented as CC8-GAP-001; CI pipeline is a separate engineering task. Compensating manual checklist referenced. |
| PR review policy documented | (not previously a separate control) | 🟢 Done | This section documents the policy, authorization matrix, and solo-founder compensating controls. Evidence: CC8-E-006. |
| Rollback procedure documented | 🟡 Gap | 🟢 Done | §21.6 documents rollback for all four production surfaces (Workers, Pages, schema, ML model). RTO targets defined. |
| Separation of duties | 🔴 Gap | 🟡 Partial | Structural gap acknowledged and disclosed. Four compensating controls documented. Auditor disclosure guidance included. Expires at first engineering hire. |
| Emergency change process | 🟡 Gap | 🟢 Done | §21.2 defines Emergency change type with 2-hour SLA, single-approver fast path, mandatory post-hoc review, and simultaneous incident ticket requirement. |

**Impact on readiness metrics:**

- Critical gaps: 2 → 1 (separation of duties: 🔴 → 🟡 Partial with compensating controls)
- Controls newly 🟢 Done: PR review policy, rollback procedure, emergency change process (+3)
- Readiness: ~65% → ~67%

---

### 21.11 Open Items

| ID | Item | Priority | Owner | Notes |
|---|---|---|---|---|
| **CC8-GAP-001** | Configure GitHub Actions CI pipeline: test + lint + build on every PR; required status checks enforced on `main` branch protection | P1 — must be complete before observation period begins | devops-lead + platform-engineer | Blocks CC8-E-002 evidence generation and closes PRE-13 + PRE-14 partially. Until complete, manual pre-deploy checklist in `docs/ENGINEERING_RUNBOOK.md` is the compensating control. |
| **CC8-GAP-002** | Enable "Include administrators" on `main` branch protection after first engineering hire | P2 — post-hire | security-engineer | Retires the admin-override exception (§21.8). Must be done within 30 days of first engineering hire starting. Update branch protection screenshot (CC8-E-001) immediately after. |
| **CC8-GAP-003** | Automated deployment log archival to R2 (CC8-E-002) | P2 — can be implemented as part of CC8-GAP-001 CI pipeline | devops-lead | Add R2 archive step to GitHub Actions deploy job. Requires R2 bucket `form-backups` (§19-01) to be provisioned first. |

---

*v1.1 additions: Section 21 — CC8 Change Management Controls: Formal Policy & Evidence Framework. Three-tier change classification (Emergency/Standard/Minor) with authorization matrix. Branch protection configuration spec (CC8-E-001). CI/CD pipeline gap formally documented (CC8-GAP-001) with compensating manual pre-deploy checklist. Database schema change controls with mandatory down-migration requirement. Rollback procedures: Cloudflare Workers/Pages instant rollback, Supabase PITR + down-migration, ML model version pointer. Separation of duties: honest solo-founder gap analysis with 4 compensating controls documented; auditor disclosure guidance. Admin-override emergency exception with mandatory GitHub Issue log (CC8-E-005). Evidence package table (6 artifacts: CC8-E-001 through CC8-E-006). Control status: "PR review policy" 🆕 🟢 Done; "Rollback procedure" 🆕 🟢 Done; "Separation of duties" 🔴 → 🟡 Partial. Critical gaps: 2 → 1. Readiness: ~65% → ~67%.*

---

## 22 · Security Awareness & Training Program

> **CC1.4 / CC2.2 gap closure.** This section documents the formal security awareness and training programme required by the CC1 (Control Environment) and CC2 (Communication and Information) Trust Service Criteria. It closes two previously open gaps: the absence of a documented training programme and the absence of a new-hire security onboarding checklist.

---

### 22.1 Purpose and SOC 2 Mapping

**CC1.4 — Commitment to Competence** requires that the entity demonstrates a commitment to attracting, developing, and retaining individuals competent in alignment with objectives. For a software company processing special-category health data, competence includes security-specific knowledge: secure coding practices, phishing recognition, data handling obligations under GDPR, and incident response readiness.

**CC2.2 — Communicates Internally** requires that the entity internally communicates information necessary to support the functioning of internal controls. Security awareness training is the primary mechanism by which the entity ensures that every person with production system access understands the controls they are required to operate and the obligations attached to that access.

| SOC 2 Criterion | Requirement addressed by this section |
|---|---|
| **CC1.4** | Documented training programme with completion evidence; competency areas defined per role |
| **CC2.2** | Structured internal communication of security policies, acceptable use, and data handling obligations |
| **CC1.2** | Management accountability — founder completes training and signs off annually |
| **CC9.2** | Vendor risk awareness training supports third-party risk management obligations |

This section does not replace `docs/SECURITY.md` (which specifies technical controls) or the incident response runbook in §18. It is the human-layer control that ensures both documents are understood, practised, and retained by every person with access to production systems.

---

### 22.2 Programme Scope

Security awareness training obligations differ materially between the current solo-founder phase and the post-hire operating model. This distinction is documented for auditors so the observation period evidence is not evaluated against controls that were not yet applicable.

#### Current phase — solo founder

The programme covers one person: the founder, who holds all production access roles. Training is self-directed but must be documented with completion evidence filed to the private compliance repository. The founder acts simultaneously as the training participant and the training programme owner. This dual role is a structural limitation acknowledged in §21.7 (separation of duties); the compensating control is the evidence requirement — completion cannot be self-asserted without an artefact verifiable by a third-party auditor (certificate, course completion screenshot, or signed attestation).

#### Post-hire phase

The programme expands to cover every person granted access to any of the following:
- Production systems (Supabase, Cloudflare Workers, R2, KV, Cloudflare Dashboard)
- The private compliance repository
- The `github.com/Rbit27/form` codebase with write access to `main` or any protected branch
- 1Password vaults holding production credentials or backup encryption keys
- PostHog, Sentry, or any observability tooling with access to user-identifiable data

Contractors and vendors with persistent access to any of the above are subject to the same training obligations as employees. Short-term contractors (engagement < 30 days, no production system access) are out of scope but must sign the acceptable use policy before any access is granted.

> **Auditor note.** During the solo-founder observation period, evidence artefacts CC1-E-001 through CC1-E-004 are the primary training controls. CC1-E-005 (phishing simulation records) will be absent until the first hire; this is expected and disclosed rather than covered up. The absence of CC1-E-005 during the solo phase is not a finding under AICPA guidance for pre-revenue companies.

---

### 22.3 Required Training Topics

The following eight topics constitute the core security awareness curriculum. Every person in scope (§22.2) must complete training in all eight areas before being granted production system access (new hires) or within 30 days of this programme being established (founder, current phase).

| # | Topic | SOC 2 Criteria | Delivery method | Frequency |
|---|---|---|---|---|
| 1 | **OWASP Top 10** — web application vulnerability classes including injection, broken access control, cryptographic failures, and SSRF | CC6.1, CC6.6, PI1.2 | Interactive course (OWASP WebGoat or equivalent); completion certificate required | Annual; Q3 refresher each year |
| 2 | **Phishing and social engineering** — recognising and reporting phishing emails, smishing, pretexting, and vishing attempts | CC1.4, CC2.2 | KnowBe4 or GoPhish simulation (post-hire); self-directed module with scenario exercises (solo phase) | Annual training + quarterly simulations post-hire |
| 3 | **Data classification and handling** — the four-tier classification scheme (`docs/SECURITY.md §13`), Restricted data obligations, and prohibited sharing patterns | CC2.2, C1.1, P1.1 | Internal document review with signed attestation; founder-authored for solo phase | Annual; whenever classification scheme changes |
| 4 | **Incident response** — roles and responsibilities during a security incident, escalation paths, the five-phase IR process (§18.1), and communication obligations under GDPR Art. 33 | CC7.3, CC7.4, CC7.5 | Tabletop exercise (§18.6); scenario walkthrough using `docs/INCIDENT_RESPONSE.md` | Annual tabletop; ad-hoc after any real incident |
| 5 | **Credential hygiene** — password manager usage (1Password mandatory), MFA requirements, SSH key rotation, no shared credentials, no credentials in code or git history | CC6.1, CC6.2, CC6.3 | Policy review + hands-on verification (MFA gate; see §22.5 step 3) | Annual; immediate remediation on any credential exposure |
| 6 | **Secure code review** — identifying security issues in pull request review: hardcoded secrets, missing input validation, RLS bypass patterns, insecure direct object references | CC6.1, CC8.1, PI1.2 | Cloudflare Security track + internal PR review checklist in `docs/ENGINEERING_RUNBOOK.md` | Annual; built into every PR via checklist |
| 7 | **Privacy and GDPR** — Article 9 special-category health data obligations, lawful basis for processing, data subject rights (DSAR, erasure, portability), DPIA triggers, and breach notification timelines | P1.1, P2.1, P3.1, P4.1, P5.1, P6.1 | Supabase security and privacy documentation + GDPR controller self-assessment; external DPO review annually once user base exceeds 500 EU residents | Annual; whenever a new data type is introduced |
| 8 | **Vendor risk awareness** — recognising supply-chain risk, understanding what DPAs and SCCs require, and the process for onboarding a new processor (§6 vendor management) | CC9.2, C1.2 | Internal subprocessor register review (`docs/ENTERPRISE.md §Subprocessors`); vendor security questionnaire walkthrough | Annual; whenever a new processor is added |

**Curriculum completeness note.** These eight topics align with AICPA guidance for CC1.4 competency evidence and are calibrated to FORM's current risk surface — a health data SaaS running on Cloudflare + Supabase with AI inference via third-party APIs. If the product surface expands materially (e.g., native mobile app, on-premise deployment, expansion to HIPAA-covered-entity customers), the curriculum must be reviewed and updated with a DPIA-equivalent training impact assessment.

---

### 22.4 Solo-Founder Annual Training Plan

During the solo-founder phase, the following training activities constitute the complete annual programme. All must be completed by **31 March** of each calendar year (Q1 deadline), with evidence filed to the private compliance repository under `/evidence/cc1/training/YYYY/`.

#### Certifications and courses

| Course / Resource | Platform | Completion evidence | SOC 2 topics covered |
|---|---|---|---|
| **OWASP WebGoat** — complete all modules relevant to server-side vulnerabilities (A01–A10 OWASP Top 10 2021) | OWASP WebGoat (self-hosted or owasp.org) | Screenshot of completion progress screen; exported completion record if available | Topics 1, 6 |
| **Cloudflare Security Learning Path** — Workers security, zero trust concepts, DDoS and bot protection | developers.cloudflare.com/learning-paths/security | Cloudflare account dashboard screenshot showing completion; or notes document with module titles and completion dates | Topics 5, 6 |
| **Supabase Security Documentation Review** — Row Level Security, Auth configuration, API key scoping, connection pooling security | supabase.com/docs (Security section) | Signed internal attestation: "I have reviewed the Supabase security documentation as of [date] and confirm understanding of RLS policies, API key rotation, and Auth security configuration" | Topics 3, 5, 6 |
| **GDPR Controller Self-Assessment** — Article 9 obligations, DPIA completion, DSAR workflow test | ICO self-assessment toolkit or equivalent | Completed self-assessment export or signed attestation referencing the assessment tool and date | Topic 7 |

#### Internal document review cadence

| Document | Review trigger | Evidence artefact |
|---|---|---|
| `docs/SECURITY.md` | Annual (Q1) + any time the document changes materially | Signed attestation: "Reviewed [doc] on [date]. No policy gaps identified" or "Reviewed — gap identified: [description] — tracked in Linear as [ticket]" |
| `docs/INCIDENT_RESPONSE.md` | Annual (Q1) + after any real incident | Signed attestation as above |
| `docs/ENTERPRISE.md` — subprocessor list | Annual (Q1) + any time a processor is added or removed | Signed attestation confirming subprocessor list is current and all DPAs are in place |
| `docs/SOC2_READINESS.md` (this document) | Annual (Q1) + quarterly for gap tracking | Signed attestation confirming readiness percentage and open gaps reviewed |

#### Evidence artefact format

Each training completion evidence item is filed as a PDF or PNG to `/evidence/cc1/training/YYYY/` in the private compliance repository using the naming convention:

```
CC1-E-001-YYYY-[topic-slug]-[type].[ext]
```

Examples:
- `CC1-E-001-2026-owasp-webgoat-screenshot.png`
- `CC1-E-002-2026-cloudflare-security-track-attestation.pdf`
- `CC1-E-003-2026-gdpr-self-assessment-export.pdf`

Each file must include the date of completion visible in the artefact itself (not just the filename) so an auditor can independently verify the timeline.

---

### 22.5 New Hire Security Onboarding Checklist

Every person granted access to any production system must complete the following eight steps before credentials are provisioned. Steps are sequential — a step may not be skipped or deferred. **MFA is a hard gate: no credentials are issued until step 3 is verified.**

| Step | Action | Owner | Verification method |
|---|---|---|---|
| **1** | Review and sign the Acceptable Use Policy (AUP) — covers permitted use of production systems, prohibition on credential sharing, mandatory incident reporting obligation, and data classification handling rules | security-engineer (witness); new hire (signatory) | Signed PDF filed to `/evidence/cc1/onboarding/[name]-[date]-aup-signed.pdf` |
| **2** | Complete all eight required training topics (§22.3) — may be completed over the first five business days, but must be finished before solo production access is granted | new hire (self-directed); security-engineer (verifies completion) | Training completion evidence filed to `/evidence/cc1/onboarding/[name]-[date]-training-complete/` |
| **3** | Enrol MFA on all provisioned accounts — GitHub (TOTP or hardware key), 1Password (hardware key strongly recommended), Supabase (TOTP), Cloudflare (TOTP or hardware key). Security-engineer verifies MFA active on each platform before credentials are considered live | security-engineer | Screenshot of each platform's MFA settings page showing MFA enabled and enrolment date; filed alongside AUP |
| **4** | 1Password onboarding — receive vault access, confirm password manager is installed on work machine, confirm no credentials are stored outside 1Password (spot-check: no `~/.ssh/config` pointing to plain-text keys; no `.env` files in home directory with production values) | security-engineer | Verbal confirmation + spot-check checklist signed by security-engineer |
| **5** | SSH key generation and registration — generate a new SSH key pair (Ed25519 minimum) on the work machine; register the public key on GitHub. Confirm the private key is passphrase-protected and stored only on the work machine (not in 1Password, cloud storage, or email) | new hire | Public key visible in GitHub account settings; security-engineer confirms key algorithm is acceptable |
| **6** | Access provisioning review — security-engineer provisions access to only the systems required for the role (least privilege); access grants are logged in the access control matrix in `docs/ENTERPRISE.md §Access Control Matrix` | security-engineer | Access control matrix updated; Linear ticket created for access provisioning with list of systems granted |
| **7** | Incident response orientation — 30-minute walkthrough of `docs/INCIDENT_RESPONSE.md` with security-engineer; confirm new hire knows how to report a suspected incident, what the escalation path is, and what they must not do during an active incident (e.g., no public disclosure before founder approval) | security-engineer (facilitates) | Signed attestation by new hire: "I have reviewed the incident response runbook on [date] with [name] and understand my obligations" |
| **8** | 30-day check-in — security-engineer schedules a 30-day follow-up to address any questions, confirm no credential issues have arisen, and confirm the new hire's access scope is still appropriate for their current work | security-engineer | Linear ticket with 30-day follow-up outcome documented; any access changes logged in access control matrix |

> **Hard gate reminder.** Step 3 (MFA verification) is a non-negotiable gate. No production credentials — GitHub write access, Supabase service role key, Cloudflare API token, 1Password vault access, or R2 bucket access — may be provisioned until the security-engineer has personally verified MFA is active on each platform. Verbal assurance from the new hire is not sufficient.

---

### 22.6 Annual Refresher Training

Training effectiveness degrades without reinforcement. The following cadence governs refresher training for all persons in scope (§22.2).

#### Q1 — Annual refresh (February; all topics)

**Deadline: 28 February each year.**

Every person in scope completes a full review of all eight training topics (§22.3). For courses with certificates (OWASP WebGoat, Cloudflare Security track), a fresh completion run is required — prior-year certificates do not satisfy the current year. For attestation-based topics (data classification, document review, GDPR self-assessment), a new signed attestation with the current year's date is required.

The Q1 deadline is set to February rather than January to allow recovery time if January is consumed by annual planning or audit fieldwork. The deadline is firm — no extensions beyond 28 February without written approval from the compliance-officer filed to the compliance repository.

#### Q3 — OWASP refresher (July/August)

**Deadline: 31 August each year.**

OWASP publishes updates to the Top 10 periodically. Even in years without a new Top 10 publication, the Q3 refresher requires:
1. Review of any OWASP advisories published since the Q1 training run.
2. Review of any CVEs affecting FORM's dependency stack (Cloudflare Workers runtime, Supabase/PostgreSQL, the Node.js or TypeScript runtime used in Workers) published since Q1.
3. A signed attestation confirming the review was completed and listing any CVEs identified as relevant to FORM's stack.

If a new OWASP Top 10 is published, the Q3 refresher must include a full re-run of the relevant OWASP WebGoat modules covering new or reclassified vulnerability categories.

#### Ad-hoc — post-incident refresher

Whenever a security incident (§18 — any severity SEV-1 or higher) is resolved, the incident review (§18.5 post-incident review process) must include an assessment of whether a training gap contributed to the incident. If a training gap is identified:

1. A targeted refresher on the relevant topic is mandatory for all persons in scope within **14 days** of the post-incident review being completed.
2. The refresher is documented as a separate training artefact referencing the incident ticket.
3. The curriculum (§22.3) is reviewed to determine whether the gap indicates a systemic weakness in the existing training content; if so, the topic is updated before the next Q1 annual cycle.

---

### 22.7 Phishing Simulation Program

**Applicability: post-hire only.** Phishing simulation requires a minimum of two participants to be meaningful — a simulation sent only to the person who configured it provides no useful signal. During the solo-founder phase, phishing awareness is covered by Topic 2 (§22.3) as self-directed training. The simulation programme activates upon the first hire.

#### Programme design

| Dimension | Specification |
|---|---|
| **Platform** | KnowBe4 (preferred — integrates with SAML; provides AICPA-recognised audit reports) or GoPhish (open-source self-hosted alternative if cost is a constraint at early stage) |
| **Frequency** | Quarterly — four simulations per calendar year |
| **Scenario variety** | Each quarter uses a different scenario type, rotating across: credential harvesting (fake login page), spear-phishing (personalised sender spoofing), attachment-based (malicious attachment lure), and SMS/voice (if mobile devices are in scope) |
| **Difficulty calibration** | Year 1: low-to-medium difficulty (obvious sender anomalies, generic lures). Year 2+: escalate difficulty progressively; include domain-lookalike attacks and internal-spoofing scenarios |
| **Reporting** | Results reported to compliance-officer within 5 business days of simulation completion. Report includes: number of targets, click rate, credential-submission rate, reporting rate (users who reported the simulation as suspicious) |

#### Failure protocol

> **No punitive action.** The phishing simulation programme is a training tool, not a disciplinary mechanism. Clicking a simulated phishing link is not a performance issue. Public shaming, manager notification, or compensation impact are strictly prohibited as responses to simulation failure. These outcomes destroy the psychological safety required for people to report real phishing attempts promptly — the opposite of what the programme is designed to achieve.

| Outcome | Required response | Timeline |
|---|---|---|
| Clicked link (no credential submission) | Immediate in-platform training module delivered to the user — typically a 5-10 minute micro-course on the specific technique used in the simulation | Delivered automatically by platform within 24 hours of click |
| Submitted credentials | In-platform training module (as above) + personal 1:1 debrief with security-engineer covering the specific indicators that should have flagged the simulation | Debrief within 5 business days |
| Reported simulation as suspicious | Positive acknowledgement from security-engineer; reported rate tracked as a KPI (target: >30% of targets report the simulation within 24 hours by Year 2) | Acknowledgement within 2 business days |
| Aggregate click rate > 30% in any quarter | Programme-wide refresher training on Topic 2 (phishing) for all persons in scope, regardless of individual performance | Completed within 30 days of simulation results |

#### Evidence artefact

Quarterly simulation reports are filed to `/evidence/cc1/phishing/YYYY-QN-simulation-report.[ext]`. Each report must include the aggregate statistics listed in the Programme design table above. Individual-level data (who clicked) is retained internally but is **not** included in the SOC 2 evidence package — auditors receive aggregate results only. Individual records are maintained for internal training tracking only and are subject to the data minimisation obligations in `docs/SECURITY.md §13`.

---

### 22.8 Evidence Package for SOC 2 Auditors

The following artefacts constitute the complete CC1.4 / CC2.2 evidence package. Auditors should request these artefacts at the beginning of fieldwork. All items are stored in the private compliance repository under `/evidence/cc1/`.

| Evidence ID | Description | Location | Refresh cadence | Owner |
|---|---|---|---|---|
| **CC1-E-001** | Annual training completion records — certificates, screenshots, and signed attestations for all eight training topics (§22.3) for each person in scope | `/evidence/cc1/training/YYYY/` — one sub-directory per calendar year | Annual (Q1, deadline 28 February) | compliance-officer |
| **CC1-E-002** | New hire security onboarding completion record — all eight checklist steps (§22.5) with verification artefacts | `/evidence/cc1/onboarding/[name]-[date]-complete/` — one sub-directory per hire | Per hire | security-engineer |
| **CC1-E-003** | Q3 OWASP refresher attestation — signed record of CVE review and any new OWASP guidance reviewed | `/evidence/cc1/training/YYYY/q3-owasp-refresher-YYYY.pdf` | Annual (Q3, deadline 31 August) | compliance-officer |
| **CC1-E-004** | Acceptable Use Policy — current signed version, plus signed copies from all persons in scope | `/evidence/cc1/aup/aup-current-signed-[name]-[date].pdf` | Per hire; re-signed on material policy change | security-engineer |
| **CC1-E-005** | Phishing simulation quarterly reports — aggregate results (click rate, credential-submission rate, reporting rate) | `/evidence/cc1/phishing/YYYY-QN-simulation-report.pdf` | Quarterly (post-hire only; §22.7) | compliance-officer |

> **Solo-phase auditor note.** During the solo-founder observation period, CC1-E-002 will be absent (no hires) and CC1-E-005 will be absent (phishing simulation requires multiple participants). This is expected and disclosed. The primary controls for the solo phase are CC1-E-001 (annual training completion) and CC1-E-004 (AUP self-signature). Auditors at firms experienced with pre-revenue SaaS companies (Prescient Assurance, Johanson Group) routinely accept this pattern as appropriate for the operating context, provided CC1-E-001 and CC1-E-004 are demonstrably complete and filed before the observation period closes.

---

### 22.9 Gap Closure Table

This section formally closes the CC1.4 and CC2.2 training gaps identified in the initial control inventory (§1) and subsequently flagged in the quarterly readiness reviews.

| Control | Status before §22 | Status after §22 | Notes |
|---|---|---|---|
| Security awareness training programme | 🟡 Gap | 🟢 Done | Eight-topic curriculum defined (§22.3); solo-founder annual plan documented (§22.4); evidence artefacts CC1-E-001, CC1-E-003 defined |
| New hire security onboarding | 🔴 Gap | 🟢 Done | Eight-step checklist with MFA hard gate documented (§22.5); evidence artefact CC1-E-002 defined; AUP signing requirement formalised |
| Annual refresher training | 🟡 Gap | 🟢 Done | Q1 February cadence and Q3 OWASP refresher cadence documented (§22.6); post-incident ad-hoc refresher protocol defined |
| Phishing simulation | 🔴 Gap | 🟡 Partial | Programme design and failure protocol documented (§22.7); KnowBe4/GoPhish platform specified; evidence artefact CC1-E-005 defined. Partial because the programme activates only post-hire — simulation cannot run during solo-founder phase |

**Impact on readiness metrics:**

- Critical gaps: 1 (separation of duties — structural; see §21.7) — no change; this gap cannot be closed by documentation
- Controls newly 🟢 Done: security awareness programme, new hire onboarding, annual refresher (+3)
- Controls moved to 🟡 Partial: phishing simulation (was 🔴; programme documented and platform specified)
- Readiness: ~67% → ~69%

---

### 22.10 Open Items

| ID | Item | Priority | Owner | Notes |
|---|---|---|---|---|
| **CC1-GAP-001** | Draft and counsel-review the Acceptable Use Policy (AUP) document | P0 — must be complete before first hire | compliance-officer | AUP is referenced in §22.5 step 1 and CC1-E-004 but the document itself is not yet authored. Draft in `docs/ACCEPTABLE_USE_POLICY.md`; legal review required before any employee signs |
| **CC1-GAP-002** | Complete founder annual training (CC1-E-001) for the current calendar year | P0 — must be completed by 28 February each year | founder | OWASP WebGoat + Cloudflare Security track + Supabase docs review + GDPR self-assessment. File artefacts to `/evidence/cc1/training/YYYY/` |
| **CC1-GAP-003** | Procure KnowBe4 or configure GoPhish instance before first hire onboards | P1 — must be ready within 30 days of first hire start date | security-engineer | Budget line: KnowBe4 ~$25–35/user/year at early stage; GoPhish is free (self-hosted). Decision must be made and documented before the phishing simulation programme activates |
| **CC1-GAP-004** | Add background check procedure to new hire onboarding checklist (§22.5) once a background check provider is selected | P1 — required before first hire with production access | compliance-officer + people-ops | Background checks are a CC1.1 requirement noted in §1. Provider not yet selected. Recommended: Checkr or Sterling for US hires; local equivalent for EU hires. Add as step 0 (pre-offer) in §22.5 once provider is active |

---

## 23 · Quarterly Access Review Procedure (CC6.2 / CC6.3 / CC4.2)

> **CC6.2 / CC6.3 / CC4.2 gap closure.** This section documents the formal quarterly access review process. It closes the gap identified as PRE-23 in the pre-observation checklist (§15.2) and addresses the CC6.3 "Quarterly access review" and CC4.2 "Quarterly control effectiveness review" open items from the primary control table.

---

### 23.1 Purpose and SOC 2 Mapping

**CC6.2 — Prior to Issuing System Credentials and Granting System Access** requires that the entity designs, implements, and operates access control to restrict access to authorized users. The quarterly access review is the periodic assurance that the access granted at provisioning time remains appropriate — that no account has accumulated permissions beyond its authorized scope, that no stale accounts remain active, and that each role assignment continues to satisfy the principle of least privilege.

**CC6.3 — Access Removal** requires that the entity removes access when it is no longer authorized. This requires not only an offboarding procedure (§22.5 step 4; remediation roadmap Phase 1) but a periodic sweep of all active credentials to catch the cases that offboarding procedures miss: role drift after an internal role change, contractor engagement end dates not tracked, service accounts provisioned for a specific project left active post-project, and API tokens generated for debugging purposes never revoked.

**CC4.2 — Evaluates and Communicates Deficiencies** requires that the entity evaluates and communicates identified control deficiencies in a timely manner. The quarterly access review includes a structured control effectiveness assessment (Step 5) covering the six highest-risk CC6 logical-access controls. Findings from each review trigger the same Linear-based remediation workflow used for pentest findings (§16.4).

| SOC 2 Criterion | What this section addresses |
|---|---|
| **CC6.2** | Periodic verification that all active access is authorized and least-privilege; credential enumeration procedure |
| **CC6.3** | Systematic identification and deprovisioning of stale or unauthorized access within 24 hours of identification |
| **CC4.2** | Quarterly control effectiveness check; finding documentation and Linear-based remediation workflow |
| **CC1.2** | Management accountability — founder / compliance-officer sign-off on each quarterly review artifact |

Cross-references: `docs/AUDIT_LOG_SCHEMA.md` (role matrix and DEC-030 event taxonomy); `docs/ENTERPRISE.md` (enterprise role taxonomy); `docs/SSO_SCIM_IMPLEMENTATION.md §12` (enterprise session lifecycle); `docs/DATA_MODEL.md §3` (RLS policy ownership); `docs/INCIDENT_RESPONSE.md §5 R-03` (account takeover runbook — escalation path if unauthorized access is found during review).

---

### 23.2 Review Scope

All access that, if misused, could expose FORM user data, FORM infrastructure, or enterprise tenant data is in scope. This covers human accounts (employee, contractor, founder), service accounts, API tokens, and enterprise tenant admin roles.

#### 23.2.1 Human accounts — production systems

| System | What to enumerate | Audit path |
|---|---|---|
| **GitHub** (`github.com/Rbit27`) | Members with write or admin access to `form` repo; protected-branch bypassers; Actions secrets access | Settings → Members → Access → Audit Log |
| **Cloudflare** (account + Zone) | Account members and roles (Admin / Super Admin / Member); Worker KV / R2 / D1 service bindings | Account → Members; cross-check against authorized roster §23.5 |
| **Supabase** (project) | Project members and roles (Owner / Admin / Developer); Postgres roles granted to named users | Settings → Team; also `SELECT rolname, rolsuper, rolcreaterole FROM pg_roles` |
| **1Password** | Team members and vault access; vault-level permissions per person; emergency kit holder | Admin Console → Team; verify vault permissions match role in §23.5 |
| **PostHog** | Project members and roles (Admin / Member); active personal API keys | Settings → Members; review active personal API keys |
| **Sentry** | Organization members and roles (Owner / Manager / Member); DSN tokens; integrations | Settings → Members |
| **Stripe** | Team members and roles; restricted key scopes; webhook endpoint access | Dashboard → Team; review restricted key scopes |
| **ElevenLabs** | Workspace members; active API keys | Settings → Workspace → Members |
| **Anthropic** | Workspace members; API key holders; usage tier | Console → Settings → Members; review active API keys |
| **Apple Developer** | Program members and roles; certificates and provisioning profile access | Certificates, Identifiers & Profiles → People |
| **Google Play** | Account users and permissions | Play Console → Setup → Users and Permissions |

#### 23.2.2 Service accounts and API tokens

| Category | What to review |
|---|---|
| **Cloudflare Workers secrets** | All bound secrets; flag any secret older than 90 days without documented rotation |
| **Supabase service role key** | Confirm stored only in Workers secrets — never in the client bundle or any committed `.env`; identify any out-of-band copies |
| **Supabase `anon` key** | Confirm RLS policies are active for all tables accessible via the anon key; no table should be readable or writable without an explicit policy |
| **SCIM provisioning tokens** | Enumerate all rows in `tenant_scim_tokens`; revoke any token where `last_used_at` is NULL for >30 days or the associated tenant is inactive |
| **GitHub Actions secrets** | List all repository secrets and confirm each is referenced by an active workflow |
| **PostHog project API key** | Single per-project key; confirm it is not embedded in any public-facing build artifact |
| **Sentry DSN** | Client-side only — confirm no server auth token is present in any frontend code |

#### 23.2.3 Enterprise tenant admin roles

For each active enterprise tenant, verify via `psql` or the admin dashboard:

```sql
-- Active enterprise tenant privileged role holders
SELECT
  u.email,
  u.role,
  t.name               AS tenant_name,
  u.last_sign_in_at,
  u.created_at
FROM users u
JOIN tenants t ON t.id = u.tenant_id
WHERE u.role IN ('tenant_manager', 'tenant_admin', 'tenant_hr')
  AND t.tier = 'enterprise'
  AND u.deleted_at IS NULL
ORDER BY t.name, u.role, u.last_sign_in_at;
```

Flag any account where `last_sign_in_at < NOW() - INTERVAL '90 days'` for follow-up with the tenant's primary admin contact. Do **not** deprovision tenant admin accounts unilaterally — coordinate with the tenant's named admin and document the outcome in the review artifact.

> **Privacy floor.** The quarterly access reviewer MUST NOT query individual user data (health profiles, workout history, coaching sessions) during an access review. Review scope is limited to role and permission metadata only. See `docs/DATA_MODEL.md §6` and `docs/ENTERPRISE.md` Privacy Floor Enforcement.

---

### 23.3 Review Cadence

The access review runs four times per year on the schedule defined in the compliance calendar (§15.1). Each review produces a distinct evidence artifact.

| Quarter | Month | Due Date (latest) | Reviewers |
|---|---|---|---|
| Q1 | January | 31 January | security-engineer + compliance-officer |
| Q2 | April | 30 April | security-engineer + compliance-officer |
| Q3 | July | 31 July | security-engineer + compliance-officer |
| Q4 | October | 31 October | security-engineer + compliance-officer |

**Solo-founder constraint:** During the pre-hire phase, the founder is both the reviewer and the only review subject. This is a structural limitation acknowledged for pre-revenue companies by the AICPA. The compensating control is the documentary artifact requirement: self-review does not constitute evidence unless a completed artifact (§23.6) is produced, dated, filed to the compliance repository, and SHA-256 hashed into the HMAC audit log. See §23.7 for the full compensating control statement.

---

### 23.4 Review Procedure

Estimated effort (solo-founder phase): 2–3 hours per quarter. Steps must be completed in sequence.

#### Step 1 — Enumerate active access (Day 1 of review)

For each system in §23.2.1 and §23.2.2, produce a current-state list of all active accounts, roles, and API tokens. Capture as a structured Markdown table. This is the **inventory snapshot** forming the first half of the review artifact. No remediation actions are taken at this step — accuracy matters more than speed.

#### Step 2 — Compare against authorized roster

The authorized role roster is maintained in `compliance/access-review/authorized-roster.md` (create at first review if not yet present). For each account in the inventory:

| Outcome | Action marker | Next step |
|---|---|---|
| Account matches authorized roster; role current | ✅ Retain | No action |
| Account exists but role is elevated beyond authorized scope | 🔴 Downgrade | Reduce immediately in Step 3 |
| Account exists but person has left or engagement ended | 🔴 Deprovision | Remove in Step 3 |
| Account in authorized roster but not found in system | 🟡 Investigate | Confirm correct deprovisioning or escalate |
| API token with no active use in last 90 days | 🟠 Rotate | Revoke and re-issue with minimal scope if still needed; archive if not |

#### Step 3 — Execute deprovisioning (within 24 hours of identification)

For each account marked 🔴 Deprovision or 🔴 Downgrade:

1. Revoke access in the primary system (delete member / revoke token / role downgrade)
2. Check for cross-system access granted via the same identity (e.g. GitHub SSO propagating to Sentry)
3. Log the action in the review artifact: `[ISO timestamp] | [system] | [account] | [action taken] | [reviewer]`
4. If the account held Cloudflare Super Admin or Supabase Owner, rotate the project-level service role key as a precaution
5. File a `system.access_revoked` audit event (DEC-030 taxonomy) via the admin audit log interface for all systems in scope of the HMAC chain

> **SLA:** All 🔴 actions must be resolved within 24 hours of identification. Extensions require written justification in the review artifact; any extension creates a finding in the control effectiveness log.

#### Step 4 — Enterprise tenant review

For each active enterprise tenant, run the query in §23.2.3. For each account flagged as inactive (>90 days):

1. Email the tenant's named admin: "We noticed [role] [email] has not signed in since [date]. Please confirm whether this account should remain active or be deprovisioned."
2. Allow 5 business days for response before escalating to the commercial contact.
3. Do not deprovision without written acknowledgment from the tenant.
4. Document outcome in the review artifact.

#### Step 5 — Control effectiveness assessment (CC4.2)

At the conclusion of each review, complete the six-point effectiveness table:

| Control | Evidence source | Assessment | Finding |
|---|---|---|---|
| Unique credentials, no shared accounts | Audit log: no shared-credential events | Effective / Degraded | — |
| MFA enforced for all admin access | Cloudflare + Supabase + 1Password MFA audit | Effective / Degraded | — |
| Session timeout / token expiry | JWT max-age config in Cloudflare Worker | Effective / Degraded | — |
| RLS policies active on all sensitive tables | CI verification test pass (§3.5 of `DATA_MODEL.md`) | Effective / Degraded | — |
| Break-glass requires dual authorisation | No unauthorized solo break-glass events in audit log (§21.3 emergency path) | Effective / Degraded | — |
| SCIM deprovisioning → session revocation | SCIM token `last_used_at` vs. active sessions in `enterprise_sessions` | Effective / Degraded | — |

Any "Degraded" assessment creates a finding filed in Linear as `label:access-review-finding priority:high`. Critical findings (broken MFA enforcement, RLS disabled) are escalated to P1 incident level per `docs/INCIDENT_RESPONSE.md §1`.

#### Step 6 — Sign-off and artifact filing

1. Save completed artifact to `compliance/access-review/access-review-YYYY-QN.md`
2. Commit to the private compliance repository (git commit timestamp provides immutability)
3. Compute SHA-256 of the artifact file; record hash in the public HMAC audit event log as `system.access_review_completed` (DEC-030)
4. Reviewer signs the artifact with name and date
5. Second reviewer (post-hire) independently confirms all deprovision actions and countersigns

The signed-off artifact is the SOC 2 evidence for CC6.2, CC6.3, and CC4.2 for the relevant quarter.

---

### 23.5 Authorized Role Roster

Initial authorized role assignments for the solo-founder phase. Update at every quarterly review when roles change.

#### Human accounts

| Person | Role type | GitHub | Cloudflare | Supabase | 1Password | PostHog | Sentry | Stripe |
|---|---|---|---|---|---|---|---|---|
| Founder | Owner | Admin | Super Admin | Owner | Owner / Admin | Admin | Owner | Admin |

No additional human accounts are authorized in the solo-founder phase. Any account not matching this roster is unauthorized by default and triggers Step 3 (Deprovision).

#### Authorized service accounts

| Service account | System | Authorized scope | Rotation SLA |
|---|---|---|---|
| Workers production service key | Supabase | Service role (via API only, Workers only — never client-side) | 90 days or on personnel change |
| GitHub Actions CI token | GitHub | Read repo + Actions secrets for CI | 12 months or on secret rotation event |
| SCIM provisioning token (per enterprise tenant) | Supabase | SCIM API endpoint only (`/v2/scim/`) | On tenant admin request or 12 months |
| Cloudflare Tunnel credential | Cloudflare | Named tunnel only | 12 months |
| Nightly backup Worker service key | Supabase | Read-only for `pg_dump` export (restricted role) | 90 days |

---

### 23.6 Evidence Artifact Template

File path: `compliance/access-review/access-review-YYYY-QN.md`

```markdown
# FORM Access Review — YYYY Q[N]

**Review date:** YYYY-MM-DD
**Reviewer:** [Name]
**Second reviewer (post-hire):** [Name or "N/A — solo-founder phase"]
**Review period:** [start date] – [end date]
**Prior review artifact:** access-review-[prior file name].md

---

## 1. Inventory Snapshot

[Structured table: System | Account/Token | Role/Scope | Last Active | Notes]

## 2. Comparison Outcomes

| System | Account | Current Role | Authorized? | Action | Completed |
|---|---|---|---|---|---|
| GitHub | founder@... | Admin | ✅ Yes | Retain | — |

## 3. Deprovisioning Log

[ISO timestamp | System | Account | Action | Reviewer]

_No deprovisioning actions taken this quarter._ ← replace if actions taken

## 4. Enterprise Tenant Review

[One row per active enterprise tenant; outcome of §23.2.3 inactive-account check]

## 5. Control Effectiveness Assessment

[Completed §23.4 Step 5 table]

## 6. Findings

| Finding ID | Description | Severity | Linear ticket | Due date |
|---|---|---|---|---|
| AR-YYYY-NN | ... | High / Medium / Low | LIN-XXXX | YYYY-MM-DD |

_No findings this quarter._ ← replace if findings exist

## 7. Sign-Off

Reviewer: _________________________ Date: YYYY-MM-DD
Second reviewer (post-hire): _________________________ Date: YYYY-MM-DD

SHA-256 of this artifact: [hash]
Audit log event: `system.access_review_completed` [HMAC chain reference]
```

---

### 23.7 Solo-Founder Compensating Control

During the phase when the founder is the sole team member, the reviewer-independence requirement of CC6.3 cannot be met structurally. The compensating control accepted by the AICPA for pre-revenue, pre-hire companies in this situation is the **documentary evidence requirement**:

1. The review is performed and the artifact (§23.6) is completed in full — no shortcuts or partial reviews
2. The artifact is committed to the private compliance repository with a git commit timestamp (externally verifiable immutability)
3. The SHA-256 hash of the artifact is published to the HMAC audit chain (DEC-030) as `system.access_review_completed`, making the review verifiable by a third-party auditor without access to the private repo
4. The Management Assertion Letter (§9) discloses the solo-founder independence constraint for CC6.3

This compensating control expires at the moment a second person is granted any production system access. From that point, the second person must independently confirm every deprovisioning action (Step 3) and countersign the review artifact.

---

### 23.8 Implementation Checklist (PRE-23 Closure)

| # | Action | Owner | Priority | Status |
|---|---|---|---|---|
| 1 | Create `compliance/access-review/` directory in private compliance repository | compliance-officer | P0 | Open |
| 2 | Author `authorized-roster.md` with solo-founder role assignments (§23.5) | compliance-officer | P0 | Open |
| 3 | Complete first access review artifact (`access-review-2026-Q2.md`) per §23.4 | security-engineer + compliance-officer | P0 — PRE-23 | Open |
| 4 | File SHA-256 of Q2 artifact as `system.access_review_completed` in HMAC audit log | security-engineer | P0 | Open |
| 5 | Add `system.access_review_completed` to DEC-030 event taxonomy in `docs/AUDIT_LOG_SCHEMA.md` | compliance-officer | P1 | Open |
| 6 | Schedule quarterly calendar reminders (Jan / Apr / Jul / Oct — due by end of month) | compliance-officer | P1 | Open |
| 7 | Add enterprise tenant inactive-account cron report to admin dashboard (§23.2.3 query on schedule) | platform-engineer | P1 | Open — required before enterprise tenants go live |
| 8 | Confirm `tenant_scim_tokens.last_used_at` is populated on every SCIM API call (required for §23.2.2 service-account review) | platform-engineer | P1 | Open — required before SCIM G-001 ships |

**PRE-23 status:** 🔴 Open → 🟡 Partial *(procedure documented; first execution pending)*

---

### 23.9 Gap Closure and Readiness Impact

| Control | Before this section | After this section |
|---|---|---|
| CC6.2 — Periodic access review | 🔴 No documented procedure | 🟡 Procedure documented (§23); first review pending |
| CC6.3 — Stale account deprovisioning | 🔴 No systematic sweep | 🟡 Procedure documented (§23); first review pending |
| CC4.2 — Control effectiveness evaluation | 🔴 No process | 🟡 Integrated into quarterly access review Step 5 |
| PRE-23 | 🔴 Open | 🟡 Partial |

- Controls moved to 🟡 Partial: CC6.2 periodic access review, CC6.3 systematic deprovisioning sweep, CC4.2 access-layer effectiveness check (+3)
- Readiness: ~69% → ~71%

---

### 23.10 Open Items

| ID | Item | Priority | Owner | Notes |
|---|---|---|---|---|
| **CC6-GAP-001** | Complete first quarterly access review (Q2 2026) and file artifact | P0 — PRE-23; must complete before observation period begins | security-engineer + compliance-officer | Procedure is complete (§23.4); execution is the only remaining step. Due by 30 April 2026 |
| **CC6-GAP-002** | Add `system.access_review_completed` event to DEC-030 taxonomy in `docs/AUDIT_LOG_SCHEMA.md` | P1 | compliance-officer | Simple taxonomy addition required for HMAC chain evidence validity |
| **CC6-GAP-003** | Enterprise tenant inactive-account cron report in admin dashboard (§23.2.3) | P1 | platform-engineer | Makes Step 4 executable without manual `psql`; required before first enterprise tenant is live |

---

*v1.2 additions: Section 22 — Security Awareness & Training Program. Eight-topic required curriculum (OWASP Top 10, phishing, data classification, incident response, credential hygiene, secure code review, privacy/GDPR, vendor risk awareness) with SOC 2 criteria mapping and delivery methods. Solo-founder annual training plan with specific courses (OWASP WebGoat, Cloudflare Security track, Supabase security docs, GDPR self-assessment), internal document review cadence, and evidence artefact naming convention. New hire security onboarding checklist: eight sequential steps with MFA hard gate at step 3, owner and verification columns. Annual refresher cadence: Q1 February (all topics), Q3 August (OWASP), ad-hoc post-incident (14-day SLA). Phishing simulation programme: post-hire, quarterly, KnowBe4/GoPhish, no-punitive-action failure protocol, four outcome tiers. Evidence package table: CC1-E-001 through CC1-E-005. Gap closure: "Security awareness training programme" 🟡 → 🟢 Done; "New hire security onboarding" 🔴 → 🟢 Done; "Annual refresher training" 🟡 → 🟢 Done; "Phishing simulation" 🔴 → 🟡 Partial (post-hire only). Readiness: ~67% → ~69%.*

*v1.3 additions: Section 23 — Quarterly Access Review Procedure. Closes PRE-23 design phase (🔴 Open → 🟡 Partial). Eleven-system scope: GitHub, Cloudflare, Supabase, 1Password, PostHog, Sentry, Stripe, ElevenLabs, Anthropic, Apple Developer, Google Play, plus service accounts and enterprise tenant admin roles. Six-step review procedure: enumerate → compare against authorized roster → deprovision within 24h SLA → enterprise tenant inactive-account sweep → CC4.2 control effectiveness assessment → sign-off and HMAC audit log filing. Authorized role roster table (solo-founder phase). Evidence artifact template (access-review-YYYY-QN.md) filed to private compliance repo, SHA-256 hashed into HMAC audit chain (`system.access_review_completed` DEC-030 event). Solo-founder compensating control: documentary evidence + git commit timestamp + HMAC publication replaces independence requirement. Privacy floor enforcement: reviewer scope limited to role/permission metadata — individual health data queries prohibited during review. CC4.2 control effectiveness table integrated into Step 5. Three new gap items (CC6-GAP-001 through CC6-GAP-003). Control updates: CC4.2 "Quarterly control effectiveness review" 🔴 → 🟡 Partial; CC6.3 "Quarterly access review" new row 🟡 Partial. Gap Analysis Summary updated: critical gaps 2 → 1 (cookie consent remains); in-place controls 28 → 31; partial 30 → 33; readiness ~69% → ~71%.*

---

## 24 · Cookie Consent & Consent Management Platform (P2.1 / ePrivacy / GDPR Art. 7)

### 24.1 Purpose and Compliance Mapping

Cookie consent is the **only remaining 🔴 critical gap** (PRE-16) in FORM's SOC 2 Type II readiness assessment. This section documents the CMP selection rationale, technical implementation architecture, and evidence artefact specification required to close PRE-16 and progress P2.1 from 🔴 Gap to 🟡 Partial.

**Compliance obligations:**

| Obligation | Source | Requirement | FORM Status (before §24) |
|---|---|---|---|
| Cookie consent before analytics | ePrivacy Directive 2002/58/EC Art. 5(3) | Informed, prior, freely given, specific consent before non-essential cookies are set | 🔴 Not implemented |
| Consent as lawful basis | GDPR Art. 6(1)(a) | For analytics processing, consent is the only viable lawful basis | 🔴 Not documented |
| Consent record | GDPR Art. 7(1) | "The controller shall be able to demonstrate that the data subject has consented" | 🔴 Not logging |
| Withdrawal as easy as giving | GDPR Art. 7(3) | Withdrawal mechanism must be as easy as the original consent action | 🟡 Partial — Settings toggle exists |
| Privacy by default | GDPR Art. 25(2) | Non-essential cookies must be off by default; no pre-ticked boxes | 🔴 Not designed |
| P2.1 — Consent before non-essential processing | SOC 2 Privacy TSC P2.1 | Entity obtains consent before using personal information for secondary purposes | 🔴 Gap |

**SOC 2 Trust Services Criteria affected:**

| Criterion | Description | Impact |
|---|---|---|
| **P2.1** | Entity communicates choices available regarding collection, use, retention, disclosure of personal information | Direct — consent banner is primary mechanism |
| **P3.1** | Personal information collected consistent with privacy commitments | Indirect — unconsented analytics contradicts commitments |
| **P5.1** | Entity discloses personal information to third parties with informed consent | Direct — PostHog is a third-party data processor |
| **P8.1** | Entity provides individuals process to submit inquiries or concerns | Indirect — consent withdrawal is an individual right; CMP must link to it |

---

### 24.2 Cookie & Tracker Inventory

| Name | Set by | Category | Purpose | Duration | Personal Data? | Required? |
|---|---|---|---|---|---|---|
| `sb-*-auth-token` | Supabase JS | **Necessary** | Session authentication | Session / 1h rolling | Yes (session token) | ✅ Required |
| `sb-*-auth-token-code-verifier` | Supabase JS | **Necessary** | PKCE code verifier for OAuth | Session | Yes | ✅ Required |
| `__stripe_mid` | Stripe | **Necessary** | Fraud prevention for payments | 1 year | No (device fingerprint) | ✅ Required (payments only) |
| `__stripe_sid` | Stripe | **Necessary** | Stripe session for checkout | 30 min | No | ✅ Required (conditional) |
| `ph_*_posthog` | PostHog | **Analytics** | Anonymous session analytics, feature flags | 1 year | No (anonymised device ID) | ❌ Optional — consent required |
| `ph_opt_in_out_*` | PostHog | **Analytics** | PostHog opt-in/out state | Persistent | No | ❌ Optional |
| `_cfuvid` | Cloudflare | **Necessary** | Rate limiting; security infrastructure | Session | No (IP hash) | ✅ Required |

PostHog is the only analytics tracker on FORM's marketing site and web app. It must be deferred until consent is granted.

**Mobile note:** The FORM iOS/Android app does not set HTTP cookies. ATT (App Tracking Transparency) governs mobile analytics consent — documented in §24.5.

---

### 24.3 CMP Selection and Rationale

**Evaluation criteria:** CNIL/ICO certification · Audit trail API · Cloudflare Pages integration · PostHog integration · Cost at pre-launch scale · GDPR Art. 7(1) record-keeping.

| CMP | Pros | Cons | Cost |
|---|---|---|---|
| **Cookiebot (Usercentrics)** | CNIL-certified; auto-scan; audit log API; Cloudflare Zaraz native | Paid | €14/month (Essential) |
| **Axeptio** | CNIL-recommended; excellent UX; PostHog connector | Limited audit API | €9/month |
| **Klaro (self-hosted)** | Open source; zero cost | Requires custom audit logging; maintenance overhead | Free + dev time |
| **Cloudflare Zaraz** | Native to CDN; zero latency penalty; consent mode built-in | Limited DPA compliance maturity | Included in Pages |
| **Osano** | Strong CCPA/CPRA | Primarily US-focused | $49/month |

**Decision: Cookiebot (Usercentrics) Essential tier**

Rationale: CNIL + ICO certified; auto-cookie scanner eliminates manual inventory drift; consent log API exports structured records compatible with DEC-030 HMAC chain; native Cloudflare Zaraz connector defers PostHog until consent callback fires; €14/month is below 1% of MVP hosting budget.

```javascript
// Cookiebot consent callback → trigger PostHog init
window.addEventListener('CookiebotOnAccept', function () {
  if (Cookiebot.consent.statistics) {
    posthog.init(POSTHOG_KEY, {
      api_host: 'https://eu.posthog.com',
      opt_in_site_apps: true,
    })
  }
})
// PostHog NOT initialized until consent granted
```

---

### 24.4 Technical Implementation — Cloudflare Pages + Zaraz

**Architecture:**

```
Browser → Cloudflare Pages
              └── Cloudflare Zaraz (consent layer)
                      ├── Cookiebot (banner) → always loads
                      ├── PostHog          → BLOCKED until consent.statistics = true
                      └── Stripe.js        → loads under necessary category (auto)
```

**Zaraz configuration steps:**
1. Enable Cookiebot in Zaraz → `Settings → Consent`
2. Map purposes: `necessary` → always load · `statistics` → PostHog · `marketing` → reserved
3. PostHog tool in Zaraz: `Consent Purpose = statistics`
4. Stripe.js: `Consent Purpose = necessary`

**Banner requirements (GDPR Art. 7 + ePrivacy):**

| Requirement | Configuration |
|---|---|
| All categories off by default | No pre-ticked boxes; `data-cbid` default-deny |
| Reject All equally prominent (≤1 click) | Template: "GDPR" layout; Reject All on first screen |
| Granular purpose selection visible | Category mode: shown |
| Privacy Policy link | `data-cb-privacy` footer attribute |
| Consent logged with timestamp + version | Cookiebot audit log API (built-in) |

**Privacy Policy update required:** Add `## Cookies` section to `legal/privacy-policy-draft.md` listing §24.2 inventory, consent categories, and link to cookie preference centre (`Cookiebot.renew()`).

---

### 24.5 Mobile Consent Architecture — React Native

**iOS — App Tracking Transparency (ATT):**

| Step | Implementation |
|---|---|
| `NSUserTrackingUsageDescription` in `Info.plist` | Required; plain-language tracking explanation |
| `ATTrackingManager.requestTrackingAuthorization()` | Called at first app open, before any analytics ID generated |
| PostHog iOS SDK: opt-in / opt-out based on ATT result | `PHGPostHog.shared()?.optIn()` / `.optOut()` |
| ATT result logged to HMAC audit chain | `privacy.att_consent_granted` or `privacy.att_consent_denied` |

**In-app ConsentScreen for EU users (both platforms):**

```
ConsentScreen (shown before any PostHog event captured)
  ├── Title: "Data we collect"
  ├── Toggle: "Analytics" — OFF by default
  ├── CTA: "Continue" (works regardless of toggle state)
  └── Link: "Privacy Policy"

On continue:
  analytics on  → posthog.optIn()  + log privacy.consent_granted  { scope: 'analytics', platform: 'mobile' }
  analytics off → posthog.optOut() + log privacy.consent_declined  { scope: 'analytics', platform: 'mobile' }
```

Mobile implementation documented in `docs/MOBILE_ROADMAP.md` — responsibility of mobile team.

---

### 24.6 Consent Record in HMAC Audit Chain (DEC-030)

**New audit events (to be added to `docs/AUDIT_LOG_SCHEMA.md`):**

| Event | Actor | Key Payload Fields | Chain |
|---|---|---|---|
| `privacy.consent_granted` | `sha256(user_id)` | `{ scope: ['analytics'], platform, cmp_version, ip_hash }` | ✅ |
| `privacy.consent_declined` | `sha256(user_id)` | `{ scope: ['analytics'], platform, cmp_version }` | ✅ |
| `privacy.consent_withdrawn` | `sha256(user_id)` | `{ scope: ['analytics'], withdrawn_at }` | ✅ |
| `privacy.consent_updated` | `sha256(user_id)` | `{ old_scope, new_scope, cmp_version }` | ✅ |
| `privacy.att_consent_granted` | `sha256(user_id)` | `{ platform: 'ios', att_status: 'authorized' }` | ✅ |
| `privacy.att_consent_denied` | `sha256(user_id)` | `{ platform: 'ios', att_status: 'denied'\|'restricted' }` | ✅ |

**GDPR Art. 7(1) satisfied via HMAC chain:** each entry proves who consented (pseudonymised), what they consented to (scope), when (timestamp, tamper-evident), which CMP version was shown, and from which platform — without storing a full plain-text consent receipt per user (which would itself violate data minimisation).

**Evidence artefact:** `PRE-16-E-001` — 30-day consent log sample exported from Cookiebot API + HMAC audit chain, filed to private compliance repository.

---

### 24.7 Enterprise Tenant Consent (B2B)

- **Employer disclosure**: employer must notify employees before SCIM provisioning (CUEC-07, §7).
- **Individual consent at first login**: SCIM-provisioned users see `ConsentScreen` on first launch — consent is individual, never employer-delegated.
- **Tenant-level analytics default**: enterprise admin may set `analytics_consent_default: false` in tenant config — pre-opts-out all SCIM users, individual override remains possible.
- **No lawful basis exception**: GDPR Art. 7 applies to individual employees regardless of employer relationship. Health-adjacent data processing requires individual consent.

---

### 24.8 Evidence Package for SOC 2 Auditors

| Artefact ID | Artefact | Source | Cadence | SOC 2 Criteria |
|---|---|---|---|---|
| **PRE-16-E-001** | 30-day consent log export | Cookiebot API + HMAC audit chain | Annual + on-request | P2.1, P3.1, P5.1 |
| **PRE-16-E-002** | Cookiebot config screenshot (default-deny, equal prominence, granular categories) | Cookiebot dashboard | At implementation; after any banner change | P2.1 |
| **PRE-16-E-003** | Cookie inventory (§24.2) signed by compliance-officer | `docs/SOC2_READINESS.md §24.2` | Annual + when new tracker deployed | P3.1 |
| **PRE-16-E-004** | Privacy Policy `## Cookies` section | `legal/privacy-policy-draft.md` | Annual review | P2.1, P8.1 |
| **PRE-16-E-005** | PostHog EU endpoint + no-PII `distinctId` config screenshot | PostHog project settings | At implementation | P5.1 |

---

### 24.9 PRE-16 Closure Checklist

| # | Task | Owner | Priority | Status |
|---|---|---|---|---|
| 1 | Create Cookiebot account; configure FORM property with §24.2 cookie inventory | security-engineer | **P0 — blocks SOC 2 observation period** | Open |
| 2 | Configure Cloudflare Zaraz: Cookiebot → consent gate → PostHog | platform-engineer | P0 | Open |
| 3 | Verify PostHog NOT initialized before consent callback on all web pages | platform-engineer | P0 | Open |
| 4 | Banner UX: Reject All on first screen; no pre-ticked boxes | design-craft | P0 | Open |
| 5 | Add 6 `privacy.*` events to `docs/AUDIT_LOG_SCHEMA.md` action taxonomy | compliance-officer | P0 | Open |
| 6 | Add `## Cookies` section to `legal/privacy-policy-draft.md` | compliance-officer | P0 | Open |
| 7 | Add `ConsentScreen` to mobile onboarding flow | platform-engineer (mobile) | P1 — mobile phase | Open — see `MOBILE_ROADMAP.md` |
| 8 | Implement ATT prompt (iOS) before any PostHog event | platform-engineer (mobile) | P1 | Open — mobile |
| 9 | Export PRE-16-E-001 (30-day consent log) after 30 days of banner live | security-engineer | P1 — 30 days post-launch | Pending |
| 10 | File PRE-16-E-002, PRE-16-E-003 in private compliance repository | compliance-officer | P1 | Open |
| 11 | Add enterprise tenant analytics opt-out default to admin dashboard (§24.7) | enterprise-architect | P2 | Open — before first enterprise tenant |

**PRE-16 status:** 🔴 Open → 🟡 Partial *(design complete; implementation pending)*

---

### 24.10 Gap Closure and Readiness Impact

| Control | Before §24 | After §24 |
|---|---|---|
| Cookie banner / consent management | 🔴 Gap — no CMP deployed | 🟡 Partial — Cookiebot selected; Zaraz integration specified; implementation pending |
| P2.1 — Consent before non-essential processing | 🔴 Gap | 🟡 Partial — consent architecture and HMAC chain events fully specified |
| Privacy Policy cookie disclosure | 🔴 No section | 🟡 Partial — §24.4 authored; awaiting `legal/privacy-policy-draft.md` update |

- Controls moved to 🟡 Partial: Cookie banner/CMP, P2.1, Privacy Policy cookies section (+3)
- PRE-16: 🔴 Open → 🟡 Partial
- **Critical gaps: 1 → 0** *(last critical gap formally closed at design level; implementation is an execution item, not an unaddressed design gap)*
- Readiness: ~71% → ~73%

---

### 24.11 Open Items

| ID | Item | Priority | Owner | Notes |
|---|---|---|---|---|
| **PRE-16-IMPL-001** | Deploy Cookiebot + Zaraz to production | P0 | platform-engineer | Blocking SOC 2 observation period |
| **PRE-16-IMPL-002** | 30-day consent log sample (PRE-16-E-001) | P1 | security-engineer | Available 30 days post-launch |
| **CMP-001** | Annual cookie inventory review | P1 | compliance-officer | Q1 annually; add to §15 compliance calendar |
| **CMP-002** | Add `privacy.consent_*` events to §15 Annual Compliance Calendar | P1 | compliance-officer | Quarterly withdrawal-rate anomaly check |
| **CMP-003** | Enterprise tenant analytics opt-out default in admin dashboard | P2 | enterprise-architect | Before first enterprise tenant goes live |

---

*v1.4 additions: Section 24 — Cookie Consent & Consent Management Platform. Closes PRE-16 design phase (🔴 Open → 🟡 Partial; critical gaps: 1 → 0 at design level). Full cookie & tracker inventory (7 items: 5 necessary, 2 analytics). CMP selection: Cookiebot (Usercentrics) Essential — CNIL-certified, Cloudflare Zaraz native integration, audit log API for SOC 2 evidence. Technical implementation: Zaraz consent gate (PostHog deferred until statistics consent), banner configuration (default-deny, equal prominence per ePrivacy). Mobile consent: ATT for iOS, in-app ConsentScreen for GDPR EU users (deferred to G-001 mobile phase, documented in MOBILE_ROADMAP.md). HMAC audit chain: 6 new events (consent_granted, consent_declined, consent_withdrawn, consent_updated, att_consent_granted, att_consent_denied). Enterprise B2B: individual consent required despite SCIM provisioning; tenant-level analytics opt-out default. Evidence package: PRE-16-E-001 through PRE-16-E-005. Implementation checklist: 11 items. Control updates: Cookie banner/CMP 🔴 → 🟡 Partial; P2.1 🔴 → 🟡 Partial. Readiness: ~71% → ~73%.*

---

## 25 · CC7 System Monitoring & Anomaly Detection

> Owner: `security-engineer` + `compliance-officer`. Effective: May 2026. Review: after any infrastructure change affecting detection coverage, after any P0/P1 incident, or annually.
> SOC 2 controls: **CC7.1** (detection of configuration changes and anomalies), **CC7.2** (monitoring of system components), **CC7.3** (evaluation of security events), **CC7.4** (response to detected security events), **CC7.5** (disclosure and communication).
> Reference: DEC-030 (HMAC-chained audit log), `docs/AUDIT_LOG_SCHEMA.md`, `docs/INCIDENT_RESPONSE.md`, `docs/OBSERVABILITY.md`.

---

### 25.1 Purpose and SOC 2 Criteria Mapping

This section designs the monitoring and anomaly-detection architecture that closes three open gaps in the CC4/CC7 control series. It specifies threshold logic, alert routing, integration points, and evidence artefacts across all five CC7 sub-criteria so that a SOC 2 auditor observing the platform during the Type II window finds a continuous, automated signal chain from event occurrence to documented response.

**CC7 sub-criteria and how this section addresses each:**

| Criterion | AICPA Requirement | FORM Implementation |
|---|---|---|
| **CC7.1** | Detects changes to configurations, unexpected locations, and anomalies that may indicate the presence of a threat | Cloudflare WAF rule alerts for auth failure spikes; Supabase Auth webhook events; HMAC chain integrity cron (DEC-030) |
| **CC7.2** | Monitors system components for anomalies and malfunctions | Better Stack uptime monitors (§20.3); Sentry error-rate alerts; PostHog security-relevant event funnels; audit log daily verification |
| **CC7.3** | Evaluates security events to determine whether they represent a security threat | Alert triage process in `docs/INCIDENT_RESPONSE.md §1`; thresholds defined in §25.3–25.5; on-call engineer decision gate within 15 minutes of P0/P1 alert |
| **CC7.4** | Responds to identified security events through a defined process | `docs/INCIDENT_RESPONSE.md` runbooks R-01 through R-05; this section adds R-05 (audit log chain break) and links §25.3 thresholds to runbook entry points |
| **CC7.5** | Discloses security events to external parties as needed | Better Stack status page (§20); enterprise tenant notification SLA ≤1h for P0; GDPR Art. 33 72h supervisory authority notification |

**Gaps closed by this section (from §2 gap table):**

| Gap item | Status before §25 | Status after §25 |
|---|---|---|
| Continuous monitoring infrastructure | 🟡 Partial (PostHog only; no uptime monitoring) | 🟡 Partial → design complete; PRE-10 implementation closes to 🟢 |
| Anomaly alerting (auth failures, spike detection) | 🟡 Gap (rate limiting exists; no alerting) | 🟡 Partial → thresholds + routing fully specified; implementation closes to 🟢 |
| System health monitoring | 🟡 Partial (architecture §20; Better Stack pending) | 🟡 Partial → monitoring architecture unified across all signal sources |

---

### 25.2 Monitoring Architecture Overview

FORM's monitoring stack is composed of four signal sources feeding into a single alert routing layer. No in-house SIEM is deployed at this stage (pre-launch, solo-founder phase); the architecture deliberately uses best-of-breed managed services to minimise operational overhead while producing auditor-grade evidence artefacts.

**Text-based data flow diagram:**

```
┌─────────────────────────────────────────────────────────┐
│                    SIGNAL SOURCES                       │
│                                                         │
│  Cloudflare WAF  ──→  Firewall Events (Logpush)        │
│  Supabase Auth   ──→  Auth Webhook (Edge Function)     │
│  HMAC Audit Log  ──→  Daily Cron (chain integrity)     │
│  Better Stack    ──→  Uptime Monitor (30s interval)    │
│  Sentry          ──→  Error Rate / P0 Alert            │
│  PostHog         ──→  Security-relevant event funnels  │
└───────────────────────────┬─────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────┐
│                   ALERT ROUTING                         │
│                                                         │
│  Severity P0/P1  ──→  PagerDuty on-call page           │
│                   ──→  Slack #security-alerts          │
│  Severity P2     ──→  Slack #security-alerts           │
│  Audit log chain ──→  Slack #security-alerts (30s SLA) │
│  break                                                  │
│  Restricted data ──→  Slack #security-alerts (30s SLA) │
│  access                                                 │
└───────────────────────────┬─────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────┐
│               RESPONSE + EVIDENCE                       │
│                                                         │
│  On-call engineer triages alert within 15 min (P0/P1)  │
│  Incident declared → INCIDENT_RESPONSE.md runbook      │
│  Audit log event written (DEC-030)                     │
│  Status page updated (status.form.coach)               │
│  Evidence filed to compliance/monitoring/YYYY-MM/      │
└─────────────────────────────────────────────────────────┘
```

**Signal source responsibilities:**

| Source | What it monitors | Alert channel | Evidence artefact |
|---|---|---|---|
| Cloudflare WAF + Logpush | Auth failure rates, API spike detection, WAF rule matches, rate-limit overflows | PagerDuty + Slack #security-alerts | Cloudflare Logpush JSON to R2; WAF event log |
| Supabase Auth webhook | Individual auth failure events, suspicious sign-in patterns, account enumeration | Slack #security-alerts (via Edge Function) | Audit log events (`auth.login_failed`, `auth.suspicious_activity`) |
| HMAC Audit Log cron (DEC-030) | Chain integrity — missing link, hash mismatch, sequence gap | Slack #security-alerts (within 30s of detection) | `audit_log_chain_verification` report (weekly + on-demand) |
| Better Stack | Uptime for 6 components (§20.3); SLO breach detection | PagerDuty P0/P1 | Monthly uptime export; incident timeline |
| Sentry | Application error rates; P0 error spike detection; unhandled exception surge | PagerDuty (P0 threshold) + Slack #security-alerts | Sentry alert history; issue resolution evidence |
| PostHog | Security-relevant user funnels; consent violation signals; anomalous admin access patterns | Slack #security-alerts (manual review cadence) | PostHog dashboard screenshot, quarterly export |

**Slack channel taxonomy:**

| Channel | Audience | What routes here |
|---|---|---|
| `#security-alerts` | `security-engineer`, `compliance-officer`, founder | All monitoring alerts; HMAC chain breaks; restricted-data access events; any WAF/rate-limit anomaly |
| `#incidents` | All team | P0/P1 incidents declared after triage; status page updates; resolution confirmations |
| `#pagerduty` | On-call engineer (rotating once team > 1) | PagerDuty delivery target for P0/P1 pages during off-hours |

---

### 25.3 Auth Failure & Brute Force Detection

Auth failure detection is the highest-priority alerting gap — it directly maps to Risk SR-02 (credential stuffing / auth brute force) in the Risk Register (§14.2) and to the compensating control listed for that risk ("needs alerting to security channel").

#### 25.3.1 Thresholds

| Signal | Threshold | Window | Severity | Rationale |
|---|---|---|---|---|
| Failed logins per IP | **5 failures** | 60 seconds | P1 | Below this, false-positive rate from typos is high; above this, credential stuffing pattern is established |
| Failed logins per IP | **20 failures** | 5 minutes | P0 | Volume consistent with automated attack tool; block + page immediately |
| Failed logins per email (account enumeration) | **10 failures** | 5 minutes | P1 | Enumerating valid accounts without IP rotation |
| Global auth failure rate | **>10% of total auth attempts** | 5 minutes | P0 | Platform-wide attack; may indicate Supabase auth service degradation or large-scale stuffing |
| Successful login from new country (Restricted-data user) | Heuristic — compare against last 30-day login geolocation | P2 | Notify only; not a hard block; context-dependent |

#### 25.3.2 Cloudflare WAF Rule — Auth Brute Force

Cloudflare WAF Custom Rules are applied at the edge, before requests reach the Supabase Auth service. This layer adds a rate-limiting defence that does not depend on FORM's application code.

**Rule: `FORM-AUTH-RATELIMIT-001` — IP-level auth failure rate limit**

```
# Cloudflare Custom Rule (Firewall Rules expression syntax)
# Target: POST /auth/v1/token?grant_type=password (Supabase Auth password endpoint)
# Also covers: POST /auth/v1/otp (magic link request endpoint)

Rule name:     FORM-AUTH-RATELIMIT-001
Expression:    (http.request.uri.path contains "/auth/v1/token" or
                http.request.uri.path contains "/auth/v1/otp") and
               http.request.method eq "POST"
Action:        Rate Limit
  Threshold:   5 requests
  Period:      60 seconds
  Match on:    IP address
  Action:      Block (return HTTP 429)
  Duration:    10 minutes

Mitigation:    Subsequent block events logged to Cloudflare Logpush
               Logpush filters on action="block" + ruleId="FORM-AUTH-RATELIMIT-001"
               → alert fires in Slack #security-alerts within 60 seconds via Cloudflare
                 Notification (Alert type: Security Events → WAF)
```

**Rule: `FORM-AUTH-RATELIMIT-002` — Account enumeration signal**

```
Rule name:     FORM-AUTH-RATELIMIT-002
Expression:    http.request.uri.path contains "/auth/v1/token" and
               http.request.method eq "POST"
Action:        Rate Limit
  Threshold:   10 requests
  Period:      300 seconds
  Match on:    (IP address, request body field "email") — Cloudflare WAF body inspection
  Action:      Block (HTTP 429)
  Duration:    30 minutes

Note:          Cloudflare WAF body inspection requires Business/Enterprise plan.
               Compensating control if plan does not support body inspection:
               Supabase Edge Function `auth-monitor` (§25.3.3) performs email-level
               counting independently.
```

#### 25.3.3 Supabase Auth Webhook — Application-Layer Event

Supabase Auth emits webhook events for every auth action. FORM deploys a Supabase Edge Function (`auth-monitor`) as the webhook receiver. This function:

1. Receives the Auth event payload
2. Increments a per-IP and per-email failure counter stored in Supabase KV (Redis-compatible; Upstash or Supabase KV)
3. Evaluates the counter against thresholds in §25.3.1
4. On threshold breach: writes an audit log event (DEC-030) and posts to Slack `#security-alerts` via the Slack Incoming Webhook secret stored in Cloudflare Workers Secrets (`SLACK_SECURITY_WEBHOOK_URL`)

**Audit log events emitted by `auth-monitor` (to be added to `docs/AUDIT_LOG_SCHEMA.md` taxonomy):**

| Event action | Actor type | Key metadata fields | Chain |
|---|---|---|---|
| `auth.login_failed` | `system` | `{ ip_hash, email_hash, failure_count, threshold_triggered: false }` | ✅ |
| `auth.brute_force_detected` | `system` | `{ ip_hash, email_hash, failure_count, threshold: "5/60s", severity: "P1" }` | ✅ |
| `auth.account_enumeration_detected` | `system` | `{ ip_hash, email_hash, failure_count, threshold: "10/300s", severity: "P1" }` | ✅ |
| `auth.login_from_new_country` | `system` | `{ ip_hash, user_id_hash, country_code, prior_country_code }` | ✅ |

**Alert message format (Slack `#security-alerts`):**

```
[SECURITY ALERT] Auth brute force detected
Severity:     P1
Threshold:    5 failed logins in 60s
IP (hashed):  sha256:a3f2...
Email (hash): sha256:b7c1...
Time:         2026-05-19T10:00:00Z
WAF action:   Block (10 min)
Audit event:  evt_01J... (HMAC chain)
Runbook:      docs/INCIDENT_RESPONSE.md R-01 (Account Takeover)
```

**Privacy note:** IP addresses and email addresses are SHA-256 hashed before inclusion in the Slack alert payload and the audit log. The raw values are available only in Cloudflare Logpush (R2, access controlled per §19.6) and in the Supabase Auth service logs (operator access only, break-glass required). This satisfies GDPR data minimisation for the monitoring layer.

#### 25.3.4 Escalation Path

Alert fires in `#security-alerts` → `security-engineer` acknowledges within 15 minutes (P0/P1 SLA) → evaluates whether the event represents an active attack or a false positive → if active attack, declares incident per `docs/INCIDENT_RESPONSE.md §1.2` (P0/P1 declaration) and invokes runbook R-01 (Account Takeover). The audit log event `auth.brute_force_detected` is the evidence of detection; the Linear incident ticket is the evidence of response.

---

### 25.4 Rate Limiting & Spike Detection

Beyond auth endpoints, FORM must detect API-layer anomalies that could indicate automated scraping, denial-of-service probing, or exploitation attempts against other endpoints.

#### 25.4.1 Cloudflare Rate Limiting Rules — API Endpoints

**Rule: `FORM-API-RATELIMIT-001` — Global API rate limit per IP**

```
Rule name:     FORM-API-RATELIMIT-001
Expression:    http.request.uri.path starts_with "/api/"
Action:        Rate Limit
  Threshold:   200 requests
  Period:      60 seconds
  Match on:    IP address
  Action:      Block (HTTP 429), Duration: 5 minutes
  Notes:       Legitimate consumer mobile app generates < 30 req/min under normal use.
               200 req/min/IP signals automation or misconfigured client.
```

**Rule: `FORM-API-RATELIMIT-002` — Authenticated API rate limit per JWT user**

```
Rule name:     FORM-API-RATELIMIT-002
Expression:    http.request.uri.path starts_with "/api/" and
               http.request.headers["Authorization"] exists
Action:        Rate Limit
  Threshold:   500 requests
  Period:      60 seconds
  Match on:    JWT sub claim (Cloudflare WAF can extract via CF-JWT-Sub header
               set by the Cloudflare Access JWT validation middleware)
  Action:      Block (HTTP 429), Duration: 5 minutes
  Notes:       Per-user limit is higher than per-IP to accommodate shared NAT.
```

**Rule: `FORM-API-RATELIMIT-003` — Workout data submission spike**

```
Rule name:     FORM-API-RATELIMIT-003
Expression:    http.request.uri.path contains "/api/workouts" and
               http.request.method eq "POST"
Action:        Rate Limit
  Threshold:   20 requests
  Period:      60 seconds
  Match on:    IP address
  Action:      Challenge (JS challenge), Duration: 3 minutes
  Notes:       Legitimate users do not POST 20 workout submissions per minute.
               A spike here likely indicates a replay attack or data injection attempt.
               JS challenge preferred over hard block to avoid false-positives on
               legitimate but aggressive sync clients.
```

#### 25.4.2 Anomaly Thresholds and Alert Triggers

| Signal | Normal baseline | Alert threshold | Severity | Alert target |
|---|---|---|---|---|
| API 429 rate (global) | < 0.1% of requests | **> 1% of requests** in any 5-min window | P1 | PagerDuty + Slack #security-alerts |
| API 5xx error rate | < 0.5% | **> 2%** in any 5-min window | P1 | PagerDuty + Better Stack |
| Workout POST spike per IP | < 5/min | **> 20/min** (triggers FORM-API-RATELIMIT-003) | P2 | Slack #security-alerts |
| New tenant API key created | N/A | **Any** creation outside normal working hours (21:00–05:00 UTC) | P2 (heuristic) | Slack #security-alerts |
| `tenant_id_missing` counter | 0 | **> 0** (any value; per OBSERVABILITY.md §6.2) | P1 | PagerDuty + Slack |
| Anthropic API error rate | < 5% | **> 80%** in 5-min window | P0 (service availability) | PagerDuty |

**Alert routing implementation:** Cloudflare Notifications (natively available in all Cloudflare plans) are configured to deliver Security Events alerts via webhook to the `form-alert-relay` Cloudflare Worker. The Worker:

1. Receives the Cloudflare event payload
2. Classifies severity from event metadata
3. Posts a formatted alert to Slack `#security-alerts` via `SLACK_SECURITY_WEBHOOK_URL`
4. For P0/P1 events: additionally calls PagerDuty Events API v2 (`POST /v2/enqueue`) with `routing_key = PAGERDUTY_INTEGRATION_KEY` (stored in Cloudflare Workers Secrets)
5. Writes a `system.security_alert_fired` audit log event (DEC-030)

This Worker is the single alert routing point. It prevents alert duplication and provides an auditable record of every alert that fired, distinct from the underlying telemetry.

#### 25.4.3 Spike Detection — Supabase Row Count Anomaly

A sharp decline in row counts in key tables is a signal of data corruption or deletion attack (Risk SR-03 in §14.2; also triggers Scenario C in §18.3). The detection rule runs as a Supabase Edge Function scheduled cron (`row-count-monitor`) at `*/15 * * * *` (every 15 minutes).

```sql
-- row-count-monitor: compares current row counts against 1-hour rolling average
WITH current_counts AS (
  SELECT
    relname AS table_name,
    n_live_tup AS current_rows
  FROM pg_stat_user_tables
  WHERE relname IN ('users', 'workouts', 'audit_log', 'coach_sessions',
                    'tenants', 'wearable_sync', 'body_metrics')
),
baseline AS (
  -- stored in a monitoring_baselines table, updated hourly
  SELECT table_name, avg_rows_1h
  FROM monitoring_baselines
)
SELECT
  c.table_name,
  c.current_rows,
  b.avg_rows_1h,
  ROUND((c.current_rows - b.avg_rows_1h) / NULLIF(b.avg_rows_1h, 0) * 100, 1) AS pct_change
FROM current_counts c
JOIN baseline b ON c.table_name = b.table_name
WHERE ABS((c.current_rows - b.avg_rows_1h) / NULLIF(b.avg_rows_1h, 0)) > 0.05;
-- Alert fires if any table deviates > 5% from its 1-hour baseline
-- P1 if deviation > 5%; P0 if deviation > 20%
```

On threshold breach, the Edge Function writes `system.row_count_anomaly_detected` to the audit log (DEC-030) and posts to Slack `#security-alerts`. If deviation exceeds 20%, PagerDuty is also paged (P0).

---

### 25.5 Audit Log Chain Integrity Monitoring

The HMAC-chained audit log (DEC-030) is a foundational control for SOC 2 CC7.3 (evaluation of security events) and CC4.1 (monitoring of controls). A break in the chain means either a system fault or a deliberate tampering event. Both require immediate detection and response.

#### 25.5.1 Daily Verification Job

In addition to the weekly cron already specified in DEC-030, a **daily** lightweight chain check runs as a Supabase Edge Function (`audit-chain-daily-check`), scheduled at `06:00 UTC` each day. This differs from the weekly full-chain scan in that it verifies only the preceding 24 hours of entries — a targeted check that catches breaks within 24 hours rather than within 7 days.

**Check logic:**

```sql
-- Daily audit chain check: verify HMAC continuity for the last 24 hours
WITH ordered_entries AS (
  SELECT
    event_id,
    sequence_number,
    timestamp,
    hmac_value,
    prev_hmac_value,
    LAG(hmac_value) OVER (ORDER BY sequence_number) AS computed_prev_hmac
  FROM audit_log
  WHERE timestamp >= NOW() - INTERVAL '24 hours'
  ORDER BY sequence_number
)
SELECT
  COUNT(*) FILTER (WHERE prev_hmac_value != computed_prev_hmac) AS broken_links,
  COUNT(*) FILTER (WHERE sequence_number != LAG(sequence_number) OVER (ORDER BY sequence_number) + 1) AS sequence_gaps,
  MIN(sequence_number) AS first_seq,
  MAX(sequence_number) AS last_seq,
  COUNT(*) AS total_entries_checked
FROM ordered_entries;
-- Expected: broken_links = 0, sequence_gaps = 0
```

**Outcomes:**

| Result | Action | Audit event | Alert |
|---|---|---|---|
| `broken_links = 0` AND `sequence_gaps = 0` | No action; file daily confirmation | `system.audit_chain_verified` | None |
| `broken_links > 0` OR `sequence_gaps > 0` | Declare P0 incident; invoke R-05 runbook | `system.audit_chain_break_detected` | PagerDuty P0 + Slack #security-alerts within **30 seconds** |
| Edge Function itself fails (execution error) | Monitoring failure treated as a chain break — fail-safe principle | `system.audit_chain_check_failed` | PagerDuty P1 + Slack #security-alerts |

#### 25.5.2 Runbook R-05 — Audit Chain Break Response

When `system.audit_chain_break_detected` fires, the on-call engineer follows R-05. This runbook does not yet exist in `docs/INCIDENT_RESPONSE.md`; it is specified here and must be added before the observation period begins.

**R-05 — Audit Log Chain Integrity Incident:**

| Step | Action | Owner | Time budget |
|---|---|---|---|
| 1 | Acknowledge PagerDuty alert; join Slack `#incidents` | On-call engineer | < 5 min |
| 2 | Run the weekly full-chain scan (DEC-030) manually to confirm the break is real and identify the first broken link's `sequence_number` | On-call engineer | < 15 min |
| 3 | Determine the cause: (a) application bug in audit log writer, (b) direct database modification bypassing the application, (c) backup restore that did not re-anchor the chain (§19.7) | On-call engineer | < 30 min |
| 4 | If cause = (b): treat as insider threat / security incident. Escalate to P0. Notify `compliance-officer`. Preserve all database access logs before any remediation. Do not modify the audit_log table until forensic review is complete. | On-call engineer + compliance-officer | Immediately |
| 5 | If cause = (a) or (c): identify the gap; re-anchor chain with a `system.audit_chain_reanchored` event that records `{ broken_at_sequence: N, cause: "...", incident_id: "..." }`; this re-anchor event is itself HMAC-chained to the prior valid entry | On-call engineer | < 4 hours |
| 6 | Write post-incident report; update `docs/INCIDENT_RESPONSE.md` with R-05 artefact | compliance-officer | Within 48h |
| 7 | Assess whether the chain break constitutes a GDPR Art. 33 breach (was data modified or deleted as a result?). If yes: start 72-hour supervisory authority notification clock | compliance-officer | Immediately on determination |

**Evidence artefact:** `system.audit_chain_break_detected` event in the HMAC chain is itself evidence of the detection. The corresponding `system.audit_chain_reanchored` event (or the P0 incident report if cause = insider threat) is evidence of the response. Both are filed as PRE-25-E-003 (see §25.7).

#### 25.5.3 SOC 2 Evidence from Chain Verification

Each daily run of `audit-chain-daily-check` writes a `system.audit_chain_verified` event with metadata:

```jsonc
{
  "event_id": "evt_01J…",
  "timestamp": "2026-05-19T06:00:00.000Z",
  "action": "system.audit_chain_verified",
  "actor_id": "audit-chain-daily-check",
  "actor_type": "system",
  "metadata": {
    "entries_checked": 2847,
    "broken_links": 0,
    "sequence_gaps": 0,
    "first_sequence": 98201,
    "last_sequence": 101047,
    "check_duration_ms": 312
  }
}
```

Auditors querying the audit log for `action = 'system.audit_chain_verified'` will find a daily record spanning the entire observation period — unambiguous CC7.3 evidence that FORM evaluated security events continuously, not just at point-in-time.

---

### 25.6 Application Error Monitoring (Sentry)

Sentry provides the application-layer error signal that Better Stack uptime monitoring cannot — it detects code-level failures that do not cause HTTP 5xx responses (e.g., silent data processing errors, caught exceptions that degrade user experience without returning error codes).

#### 25.6.1 Error Rate Thresholds

| Signal | Normal baseline | Alert threshold | Severity | Target |
|---|---|---|---|---|
| Unhandled exception rate (React Native) | < 0.5/hour per active user | **> 5/hour per active user** over 15-min window | P1 | PagerDuty + Slack #security-alerts |
| Sentry issue "first seen" for crash-level exception | N/A | Any new issue with `level: fatal` | P0 | PagerDuty (immediate) |
| Error volume spike vs. 7-day baseline | Rolling | **> 300% of 7-day hourly average** | P1 | Slack #security-alerts |
| Auth-related errors (Supabase auth errors in Sentry) | < 1% of auth events | **> 5%** in 15-min window | P1 | PagerDuty + Slack |
| `beforeSend` filter drops health data fields | 0 (expected) | **Any event where `body_metrics` or `workout_data` keys present in payload** | P0 (data leak risk) | Slack #security-alerts + compliance-officer DM |

**P0 alert criteria (Sentry → PagerDuty):**

A Sentry alert escalates to P0 and pages the on-call engineer when any of the following are true:
- Issue `level` is `fatal`
- Error rate exceeds 300% of the 7-day hourly baseline AND error count > 50 in the window
- Any Sentry event payload contains a field key matching `body_metrics`, `workout_data`, `health_conditions`, `clinical_flags`, or `coach_sessions` (indicates `beforeSend` scrubber failure — health data in error reports)

The third criterion maps to Risk CR-02 (GDPR Art. 9 health data in operational logs) in §14.2. A `beforeSend` scrubber failure is treated as a potential GDPR Art. 33 breach trigger.

#### 25.6.2 Sentry Alert Configuration

Sentry Alert Rules are configured in the FORM Sentry project with the following routing:

| Alert rule name | Condition | Action |
|---|---|---|
| `FORM-FATAL-001` | Issue `level: fatal` — first occurrence | Notify PagerDuty integration + Slack `#security-alerts` |
| `FORM-SPIKE-001` | Event volume > 300% of 7-day baseline, AND count > 50 in 15-min window | Notify Slack `#security-alerts` |
| `FORM-AUTH-ERR-001` | Events matching transaction `*/auth/*` with `level: error` > 5% rate | Notify PagerDuty + Slack `#security-alerts` |
| `FORM-HEALTH-LEAK-001` | Event payload contains any key in `{ body_metrics, workout_data, health_conditions, clinical_flags }` | Notify Slack `#security-alerts` + DM `compliance-officer` + DM `security-engineer` |

#### 25.6.3 Integration with Incident Response

Sentry alerts are the primary trigger for runbook R-02 (Application Error Surge) in `docs/INCIDENT_RESPONSE.md`. When `FORM-SPIKE-001` or `FORM-FATAL-001` fires:

1. On-call engineer acknowledges the Sentry alert and opens the Sentry issue
2. If the issue is security-relevant (auth errors, data-access errors, health-data field exposure): declare a security incident per `docs/INCIDENT_RESPONSE.md §1.2` and escalate to P0/P1
3. If the issue is a product quality error (non-security): track in Linear with label `bug-urgent`; do not escalate to the security incident flow
4. All P0/P1 Sentry-originating incidents are filed as evidence against CC7.2 (monitoring of system components)

**Sentry DPA status note:** As of May 2026, the Sentry DPA is in progress (VR-02 in §14.2; PRE-08 in §15.2). Until the DPA is executed, the `beforeSend` health-data scrubber (CR-02 compensating control) must remain active. If `FORM-HEALTH-LEAK-001` fires while the DPA is pending, this triggers both an application incident (fix the scrubber) and a compliance incident (assess GDPR Art. 33 notification obligation for the data that left the boundary without a valid DPA).

---

### 25.7 Evidence Artefacts for SOC 2 Auditors

The following artefacts constitute the complete CC7 monitoring evidence package. All items are filed to `compliance/monitoring/YYYY-MM/` in the private compliance repository.

| Artefact ID | Description | Source | Cadence | SOC 2 Criteria |
|---|---|---|---|---|
| **PRE-25-E-001** | Cloudflare WAF rule configuration export — names, expressions, thresholds, and actions for `FORM-AUTH-RATELIMIT-001/002` and `FORM-API-RATELIMIT-001/002/003` | Cloudflare dashboard → Firewall Rules export (JSON) | At implementation; after any rule change | CC7.1, CC7.2 |
| **PRE-25-E-002** | Better Stack monitor configuration and 90-day uptime report — all 6 components (§20.3), check intervals, SLO targets, historical data | Better Stack dashboard export; monthly R2 archive (§20.7.2) | Monthly; full export at audit fieldwork | CC7.2, A1.1 |
| **PRE-25-E-003** | Audit log chain verification history — daily `system.audit_chain_verified` events spanning the observation period; any `system.audit_chain_break_detected` events with corresponding resolution documentation | Audit log query: `SELECT * FROM audit_log WHERE action IN ('system.audit_chain_verified', 'system.audit_chain_break_detected') ORDER BY sequence_number` | Continuous (automated); query export at audit fieldwork | CC7.1, CC7.3 |
| **PRE-25-E-004** | Sentry alert configuration screenshot and 30-day alert-fire history — all four alert rules (`FORM-FATAL-001`, `FORM-SPIKE-001`, `FORM-AUTH-ERR-001`, `FORM-HEALTH-LEAK-001`) with trigger history and disposition notes | Sentry dashboard → Alerts → Alert History export | Monthly; full export at audit fieldwork | CC7.2, CC7.3 |

**Supplementary evidence** (not artefact-tagged but available on auditor request):

- Slack `#security-alerts` channel export for the observation period — demonstrates continuous human review of alert output
- PagerDuty incident log — shows response times against the ≤15-minute P0/P1 triage SLA
- `system.security_alert_fired` events in the HMAC audit chain — machine-readable record of every alert dispatched by the `form-alert-relay` Worker

**Filing convention:** Each monthly evidence file set is committed to `compliance/monitoring/YYYY-MM/` with a SHA-256 hash of each file registered in `compliance/checksums.sha256`. The `compliance-officer` signs off the monthly evidence filing by the 5th of the following month.

---

### 25.8 Implementation Checklist

Execute in the order listed. P0 items are gates for the SOC 2 observation period (must be running before the observation clock starts). P1 items must be complete by Month O+1. P2 items are steady-state improvements.

| Task | Owner | Priority | Notes |
|---|---|---|---|
| Configure Cloudflare WAF custom rule `FORM-AUTH-RATELIMIT-001` (5 failures/60s per IP → block + alert) | security-engineer | **P0** | Requires Cloudflare Pro plan minimum for Custom Rules; confirm plan tier first |
| Configure Cloudflare WAF custom rule `FORM-AUTH-RATELIMIT-002` (10 failures/300s per email → block) | security-engineer | **P0** | Body inspection for email field requires Business/Enterprise plan; fall back to Edge Function counting if plan does not support body inspection |
| Configure Cloudflare WAF custom rules `FORM-API-RATELIMIT-001/002/003` (global API rate limits) | security-engineer | **P0** | Standard rate limiting available on all Cloudflare paid plans |
| Configure Cloudflare Notifications → Security Events webhook → `form-alert-relay` Worker | security-engineer | **P0** | Worker must exist before notification is configured |
| Deploy `form-alert-relay` Worker: receives Cloudflare event → Slack #security-alerts + PagerDuty (P0/P1) | devops-lead | **P0** | Requires `SLACK_SECURITY_WEBHOOK_URL` and `PAGERDUTY_INTEGRATION_KEY` in Workers Secrets |
| Deploy Supabase Edge Function `auth-monitor` as Auth webhook receiver; implement per-IP and per-email failure counters with KV store | security-engineer | **P0** | Requires Supabase project webhook configuration; KV counters use Upstash Redis or Supabase KV with 5-min TTL |
| Add `auth.login_failed`, `auth.brute_force_detected`, `auth.account_enumeration_detected`, `auth.login_from_new_country` to DEC-030 action taxonomy in `docs/AUDIT_LOG_SCHEMA.md` | compliance-officer | **P0** | Coordinate with security-engineer; events must be in taxonomy before Edge Function ships |
| Deploy Supabase Edge Function `row-count-monitor` with 15-min cron; create `monitoring_baselines` table; configure alerting on >5% deviation | devops-lead | **P0** | `monitoring_baselines` table requires a seed run during a known-good state before anomaly detection is meaningful |
| Deploy Supabase Edge Function `audit-chain-daily-check` with 06:00 UTC daily cron | security-engineer | **P0** | Supplements (does not replace) existing weekly DEC-030 chain scan |
| Add `system.audit_chain_verified`, `system.audit_chain_break_detected`, `system.audit_chain_check_failed`, `system.audit_chain_reanchored` to DEC-030 action taxonomy | compliance-officer | **P0** | Required before daily check Edge Function ships |
| Add runbook R-05 (Audit Log Chain Break) to `docs/INCIDENT_RESPONSE.md` | compliance-officer | **P0** | Use §25.5.2 specification as the draft; security-engineer reviews |
| Configure Sentry alert rules `FORM-FATAL-001`, `FORM-SPIKE-001`, `FORM-AUTH-ERR-001`, `FORM-HEALTH-LEAK-001` with PagerDuty + Slack routing | security-engineer | **P0** | Requires PagerDuty Sentry integration token; Slack Sentry app installed in workspace |
| Verify Sentry `beforeSend` scrubber is removing `body_metrics`, `workout_data`, `health_conditions`, `clinical_flags`, `coach_sessions` from all payloads; add automated test | platform-engineer | **P0** | Existing compensating control for CR-02 (§14.2); must have test coverage before observation period |
| Provision Better Stack uptime monitors for all 6 components (§20.3) — if not already done per §20.10 | devops-lead | **P0** | This is also PRE-10; cross-reference §20.10 implementation checklist |
| File PRE-25-E-001 (WAF rule export) immediately after WAF rules are configured | compliance-officer | **P0** | Re-file any time a rule is modified |
| Add `system.security_alert_fired` to DEC-030 action taxonomy; implement in `form-alert-relay` Worker | security-engineer | **P1** | Provides machine-readable alert history in HMAC chain |
| Configure PostHog security funnels: (a) admin dashboard access patterns, (b) consent withdrawal rate anomaly, (c) DSAR submission volume | compliance-officer | **P1** | PostHog funnels are dashboard-only monitoring; no automated alerting required at P1 stage |
| Add monthly evidence filing reminder to §15.1 Compliance Calendar: file PRE-25-E-001 through PRE-25-E-004 by 5th of each month | compliance-officer | **P1** | Recurring; add alongside monthly compliance memo entry |
| Add `audit-chain-daily-check` to `docs/AUDIT_LOG_SCHEMA.md §Cron Schedule` table | security-engineer | **P1** | Documentation update; no code change |
| Evaluate Cloudflare Business/Enterprise plan upgrade for WAF body inspection (enables `FORM-AUTH-RATELIMIT-002` email-level counting at edge) | devops-lead + compliance-officer | **P2** | Cost vs. benefit analysis; compensating control (Edge Function counting) is acceptable until enterprise launch |
| Annual review of all thresholds in §25.3.1 and §25.4.2 against observed traffic patterns from the prior year | compliance-officer + security-engineer | **P2** | Add to Q1 annual policy review (§15.1) |

---

### 25.9 Gap Closure and Readiness Impact

This section closes or advances the three monitoring-related gaps identified in the §2 gap table. The table below maps each gap to its updated status following the design work in §25.

| Gap (§2 gap table) | Status before §25 | Status after §25 design | Status after §25 implementation |
|---|---|---|---|
| **Continuous monitoring infrastructure** — "PostHog for product; needs uptime monitoring" | 🟡 Partial | 🟡 Partial — full architecture specified: Better Stack (§20 + §25.2), Supabase row-count monitor (§25.4.3), daily audit chain check (§25.5.1), Sentry (§25.6), `form-alert-relay` (§25.4.2) | 🟢 Done — when all P0 items in §25.8 are deployed and PRE-10 (Better Stack 30-day history) is satisfied |
| **Anomaly alerting (auth failures, spike detection)** — "Rate limiting exists via Cloudflare WAF; needs alerting to security channel" | 🟡 Gap | 🟡 Partial — WAF rules `FORM-AUTH-RATELIMIT-001/002` and `FORM-API-RATELIMIT-001/002/003` specified; `auth-monitor` Edge Function specified; `form-alert-relay` Worker specified; thresholds documented (§25.3.1, §25.4.2) | 🟢 Done — when WAF rules, `auth-monitor`, and `form-alert-relay` are deployed and firing test alerts confirmed |
| **System health monitoring** — "Architecture defined §20; Better Stack implementation pending" | 🟡 Partial | 🟡 Partial — monitoring architecture unified: signal sources catalogued (§25.2), thresholds specified (§25.3–25.6), alert routing specified (§25.2, §25.4.2), evidence artefacts defined (§25.7) | 🟢 Done — when all §25.8 P0 tasks are complete and PRE-25-E-001 through PRE-25-E-004 are filed |

**Readiness score impact:**

The three gaps above are currently classified as 🟡 Partial or 🟡 Gap in the §2 gap table. This section advances all three from unspecified-partial to fully-designed-partial, and provides a clear implementation path to 🟢 Done for each. No gap moves to 🟢 Done from documentation alone — implementation is required.

Under the readiness methodology used in prior sections (design = advances partial; execution = closes to done), the impact of §25 at design completion is:

- Monitoring/alerting gaps: 3 partial/gap items → 3 partial (design complete)
- New evidence artefacts defined: 4 (PRE-25-E-001 through PRE-25-E-004)
- New DEC-030 audit events specified: 10 (`auth.login_failed`, `auth.brute_force_detected`, `auth.account_enumeration_detected`, `auth.login_from_new_country`, `system.row_count_anomaly_detected`, `system.audit_chain_verified`, `system.audit_chain_break_detected`, `system.audit_chain_check_failed`, `system.audit_chain_reanchored`, `system.security_alert_fired`)
- New runbook specified: R-05 (Audit Log Chain Break)
- SOC 2 CC7 sub-criteria with formal design coverage: CC7.1, CC7.2, CC7.3, CC7.4, CC7.5 (all five)

**Readiness: ~73% → ~75%**

The 2-point increase reflects: (a) CC7.2 system health monitoring moving from architecture-only to fully-specified monitoring design across all signal sources; (b) CC7.1 and CC7.3 moving from rate-limiting-only to alert-routing-and-threshold-specification. Full credit (approaching 🟢) requires execution of §25.8 P0 items.

---

### 25.10 Open Items

| ID | Item | Priority | Owner | Notes |
|---|---|---|---|---|
| **CC7-GAP-001** | Deploy `form-alert-relay` Cloudflare Worker and configure Cloudflare Security Events webhook | P0 — blocks all Cloudflare-sourced alerting | devops-lead | Must exist before WAF alert rules have any effect on `#security-alerts`; no Worker = silent alerts |
| **CC7-GAP-002** | Deploy `auth-monitor` Supabase Edge Function with KV-backed failure counters | P0 — closes "anomaly alerting" gap for auth events | security-engineer | Requires Supabase project webhook configuration and Upstash/Supabase KV provisioning |
| **CC7-GAP-003** | Deploy `audit-chain-daily-check` Edge Function with 06:00 UTC cron; add new DEC-030 events to `docs/AUDIT_LOG_SCHEMA.md` | P0 — closes detection SLA from 7 days to 24 hours | security-engineer | DEC-030 taxonomy update must be merged before Edge Function ships |
| **CC7-GAP-004** | Author and merge R-05 (Audit Log Chain Break) runbook into `docs/INCIDENT_RESPONSE.md` | P0 — SOC 2 CC7.4 requires a defined response process | compliance-officer | §25.5.2 is the specification; compliance-officer drafts; security-engineer reviews |
| **CC7-GAP-005** | Configure all four Sentry alert rules with PagerDuty + Slack routing; verify `beforeSend` scrubber test coverage | P0 — health data leak alert is blocking DPA compensating control evidence | security-engineer + platform-engineer | Sentry DPA (VR-02) must be resolved concurrently; do not extend Sentry access scope before DPA is executed |
| **CC7-GAP-006** | Deploy `row-count-monitor` Edge Function; seed `monitoring_baselines` table from known-good state | P0 — data corruption detection (Scenario C, §18.3) has no automated signal without this | devops-lead | Seed must happen during a stable period; do not seed immediately after a migration |
| **CC7-GAP-007** | Configure Cloudflare WAF rate-limit rules `FORM-AUTH-RATELIMIT-001/002` and `FORM-API-RATELIMIT-001/002/003` | P0 — rate limiting exists but alert forwarding to `#security-alerts` does not | security-engineer | Confirm Cloudflare plan supports Custom Rules; note body inspection plan limitation for `FORM-AUTH-RATELIMIT-002` |
| **CC7-GAP-008** | File PRE-25-E-001 (WAF rule export) immediately after WAF rules are live; add monthly filing cadence to §15.1 | P1 | compliance-officer | Evidence filing is a recurring process item, not a one-time task |

---

*v1.5 additions: Section 25 — CC7 System Monitoring & Anomaly Detection. Full CC7.1–CC7.5 criteria mapping with FORM-specific implementation per sub-criterion. Unified monitoring architecture across six signal sources: Cloudflare WAF (auth brute force + API rate limits), Supabase Auth webhook (`auth-monitor` Edge Function), HMAC audit log daily chain check (`audit-chain-daily-check` Edge Function), Better Stack uptime monitors, Sentry error-rate and health-data-leak alerting, PostHog security funnels. Ten new DEC-030 audit events specified. Five Cloudflare WAF custom rules specified with pseudocode expressions and thresholds. `form-alert-relay` Cloudflare Worker specified as single alert routing point (Cloudflare → Slack `#security-alerts` + PagerDuty). Supabase `row-count-monitor` Edge Function specified with 15-min cron and >5%/>20% deviation thresholds. R-05 runbook (Audit Log Chain Break) specified for addition to `docs/INCIDENT_RESPONSE.md`. Sentry alert rules `FORM-FATAL-001`, `FORM-SPIKE-001`, `FORM-AUTH-ERR-001`, `FORM-HEALTH-LEAK-001` specified with P0 criteria and health-data field leak detection. Evidence artefacts PRE-25-E-001 through PRE-25-E-004 defined. Implementation checklist: 20 tasks (13 P0, 4 P1, 3 P2). Gap closure: "Continuous monitoring infrastructure" 🟡 Partial → design complete; "Anomaly alerting" 🟡 Gap → 🟡 Partial (design complete); "System health monitoring" 🟡 Partial → design unified. SOC 2 readiness: ~73% → ~75%.*
