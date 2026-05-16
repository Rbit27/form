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

**v0.1 · травень 2026 · owner: compliance-officer + security-engineer + enterprise-architect**
**Review cadence: quarterly. Next review: серпень 2026.**
