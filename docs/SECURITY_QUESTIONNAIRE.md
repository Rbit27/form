# FORM · Security Questionnaire & CAIQ Lite Responses v1.0

| Field | Value |
|---|---|
| **Document title** | Security Questionnaire — Pre-filled CAIQ Lite & Common Enterprise DDQ Responses |
| **Version** | v1.1 |
| **Status** | IN FORCE |
| **Effective date** | 2026-06-06 |
| **Owner** | `security-engineer` + `compliance-officer` |
| **Evidence artifact ID** | SQ-E-001 |
| **Cross-references** | `docs/SOC2_READINESS.md`, `docs/AUDIT_LOG_SCHEMA.md` (DEC-030), `docs/CRYPTOGRAPHY_POLICY.md`, `docs/DATA_MODEL.md`, `docs/INCIDENT_RESPONSE.md`, `docs/SUBPROCESSORS.md`, `docs/GDPR_DPIA.md`, `docs/ENTERPRISE.md` |
| **Last reviewed** | 2026-06-20 |
| **Next review due** | 2027-06-20 |
| **Distribution** | NDA-gated. Available to qualified prospects and current enterprise customers via `security@form.coach`. |

---

## Purpose and Scope

This document provides FORM's pre-filled responses to the **Cloud Security Alliance (CSA) CAIQ Lite** questionnaire and to common enterprise vendor security questionnaire (VSQ/DDQ/SIG Lite) sections. It is the source document behind the security questionnaire hub referenced at `security.form.coach`.

**Audience:** IT security teams, procurement officers, CISOs, and DPOs conducting vendor security assessments.

**How to use this document:**
- All 16 CAIQ Lite domains are covered in §1.
- Common enterprise DDQ sections (data residency, incident SLAs, penetration testing, privacy) are covered in §2.
- Evidence artefacts (SOC 2 report, pen-test summary, DPA template) are available under NDA — contact `security@form.coach`.
- For custom questionnaire formats (SIG Full, HECVAT, bespoke), turnaround is 5 business days. Contact `security@form.coach`.

**Material statements:** All answers reflect FORM's current state as of the effective date above. Where a control is in progress (not yet fully implemented), this is stated explicitly. FORM does not misrepresent the maturity of controls.

---

## §1 — CAIQ Lite: Cloud Security Alliance Controls Assurance Initiative Questionnaire

### AIS — Application & Interface Security

**AIS-01: Do you use an industry-recognised secure software development lifecycle (SDLC) methodology?**

Yes. FORM's SDLC incorporates the following controls:
- Threat modelling for new features that introduce data flows or authentication changes.
- Mandatory PR review before merge to main (enforced via branch protection from the first engineering hire; compensating control during solo-founder phase: all self-merged PRs are tagged for retrospective security review at quarterly access review).
- Static analysis and secret-scanning in CI via GitHub Advanced Security (GHAS) push protection and TruffleHog; secrets are blocked from reaching the repository.
- OWASP Top 10 review integrated into code review checklist per `docs/SOC2_READINESS.md §21.5`.
- Dependency scanning via `npm audit` / `cargo audit` on every PR.

Evidence: CI pipeline configuration; `docs/SOC2_READINESS.md §21`.

---

**AIS-02: Do you use industry-recognised application security testing as part of your SDLC?**

Yes. FORM conducts:
- **Annual penetration tests** by a qualified third-party vendor (program documented in `docs/SOC2_READINESS.md §16`; first engagement scheduled for Month 7 of platform operation). Summary findings shared with Enterprise customers under NDA.
- **SAST / SCA in CI**: GHAS Code Scanning + TruffleHog on every PR.
- **OWASP Dependency-Check** integrated in CI pipeline.

Findings from penetration tests are tracked to remediation in Linear with `label:pentest-finding`; Critical / High findings require resolution before the next production deploy. Summary available upon request under NDA.

---

**AIS-03: Do you use a web application firewall (WAF)?**

Yes. All FORM API and web endpoints are protected by **Cloudflare WAF** at the edge layer. The WAF enforces:
- HTTPS-only (HTTP → HTTPS redirect; HSTS with 1-year max-age + `includeSubDomains`).
- OWASP Core Rule Set (managed ruleset, Cloudflare-maintained).
- IP-based rate limiting on authentication endpoints.
- Bot protection on public API paths.

Evidence: Cloudflare dashboard configuration; `docs/SOC2_READINESS.md §30` CC6.6 evidence.

---

### AAC — Audit Assurance & Compliance

**AAC-01: Do you provide customers with evidence of an independent security assessment (e.g., SOC 2, ISO 27001)?**

- **SOC 2 Type II**: Audit in progress under Drata continuous compliance monitoring. Target attestation: Q1 2027 (12 months post-commercial launch). Enterprise-tier customers receive a copy of the report under NDA upon completion. Starter / Growth customers may request a summary letter.
- During the interim period: FORM provides Drata trust centre access (continuous monitoring dashboard) under NDA upon request.
- **Annual penetration test summary** available under NDA from Month 7 onwards.

Contact `security@form.coach` to request available evidence packages.

---

**AAC-02: Do you have policies that define acceptable use, information security, and data classification?**

Yes.
- **Acceptable Use Policy**: `docs/ACCEPTABLE_USE_POLICY.md` — signed by all team members at onboarding.
- **Information Security Policy**: `docs/SECURITY.md` + `docs/CRYPTOGRAPHY_POLICY.md`.
- **Data Classification Policy**: `docs/SOC2_READINESS.md §13` — four levels (Public, Internal, Confidential, Restricted). Health data (GDPR Article 9) is classified as Restricted; applies the strictest controls including column-level AES-256-GCM encryption and the Privacy Floor (§13).
- **Privacy Policy**: `docs/PRIVACY_POLICY.md` (public).

Evidence: Signed AUP acknowledgement artefacts per `docs/ONBOARDING_SECURITY.md §3 Step 1`.

---

**AAC-03: Do you have a documented internal audit programme?**

Yes. FORM operates a **compliance calendar** (`docs/SOC2_READINESS.md §15`) with scheduled quarterly and annual reviews:
- Quarterly: access review (CC6.2, CC6.3), patch review, incident register audit.
- Semi-annual: DR table-top exercise; security awareness training.
- Annual: penetration test; privacy review (DPIA refresh); SOC 2 evidence package review; vendor security reviews.

The compliance calendar is owned jointly by `compliance-officer` and `security-engineer` with founder oversight. Control effectiveness reviews feed into the quarterly PIR cycle per `docs/INCIDENT_RESPONSE.md §9.3`.

---

### BCR — Business Continuity Management & Operational Resilience

**BCR-01: Do you have a documented Business Continuity Plan (BCP) and Disaster Recovery (DR) plan?**

Yes.
- **Business Continuity Plan**: `docs/BUSINESS_CONTINUITY.md` — covers people, process, and technology continuity.
- **DR Runbooks**: Five operational runbooks in `docs/runbooks/dr/`:
  - `FORM-DR-001`: Supabase (primary database) outage — RTO 2h / RPO 1h.
  - `FORM-DR-002`: Cloudflare Workers / Edge API failure.
  - `FORM-DR-003`: R2 Object Storage data loss or corruption.
  - `FORM-DR-004`: Anthropic Claude API sustained outage — graceful degradation via `VICTOR_FALLBACK=true` KV flag within 15 minutes.
  - `FORM-DR-005`: Compound multi-component failure.

**SLA targets** (Enterprise tier): RTO ≤ 2 hours / RPO ≤ 1 hour for P0 database incidents. See `docs/ENTERPRISE_SLA.md §7`.

---

**BCR-02: Do you conduct regular DR testing / table-top exercises?**

Yes. DR table-top exercises are scheduled in Q1 and Q3 of each calendar year per `docs/SOC2_READINESS.md §15.3`. The first scheduled DR drill is Q1-January. Drill completion is recorded as a `system.dr_drill_completed` DEC-030 audit event.

---

**BCR-03: Do you have redundant infrastructure across multiple availability zones or regions?**

Yes.
- **Cloudflare Workers**: Globally distributed edge compute; no single-point-of-failure for API routing layer.
- **Supabase Postgres**: Multi-AZ deployment within the selected region (EU: `de-fra1` Frankfurt for EU-tenant data residency). Point-in-time recovery (PITR) enabled with ≤ 1-hour RPO.
- **R2 Object Storage** (Cloudflare): Geo-replicated within Cloudflare's global network. EU jurisdictional restriction enforced for EU tenant data.
- **Backblaze B2 WORM** (cold backup): Cold backup tier for long-term compliance retention; separate provider from primary storage to avoid correlated failure.

---

### CCC — Change Control & Configuration Management

**CCC-01: Do you have a documented change management policy?**

Yes. `docs/SOC2_READINESS.md §21` documents FORM's change management policy including:
- All changes deployed via CI/CD pipeline (Cloudflare Workers CI; Supabase migrations via `supabase db push --linked`).
- Branch protection enforces PR review; no direct push to `main` (enforced from first engineering hire).
- CI gates block deployment if: (a) RLS integration tests fail, (b) cross-tenant isolation tests fail, (c) `supabase test db --filter=rls` fails.
- Database migrations are forward-only; no destructive schema change without a documented rollback plan.
- Infrastructure changes (Cloudflare WAF rules, Supabase configuration) require a change ticket and secondary approval above the solo-founder phase.

Evidence: CI pipeline configuration; branch protection rules; `docs/SOC2_READINESS.md §21`.

---

**CCC-02: Do you maintain a configuration management database (CMDB) or equivalent inventory?**

Yes. FORM maintains an infrastructure inventory in `docs/SOC2_READINESS.md §17` (vendor register) and `docs/SUBPROCESSORS.md` (sub-processor register). The vendor register covers all production systems with assigned owners, classification tiers, and annual review dates.

---

### DSP — Data Security & Privacy Lifecycle Management

**DSP-01: Do you have a documented data retention and disposal policy?**

Yes.
- **Audit log retention**: 7 years for SOC 2 / financial events; 3 years for operational events; 30 days for read-only access events. Policy in `docs/AUDIT_LOG_SCHEMA.md` and `docs/SOC2_READINESS.md §12`.
- **User data retention**: Retained for the duration of active subscription + 30 days post-cancellation. Upon written deletion request, user data is purged from primary storage within 30 days and from backups within 90 days (encrypted backups). GDPR Art. 17 right-to-erasure procedure in `docs/INCIDENT_RESPONSE.md §14` (DSAR runbook).
- **Enterprise tenant data**: Upon contract termination, tenant data is exported to the customer on request (within 30 days) and purged from FORM systems within 90 days. HMAC-chained audit logs are retained for the full statutory period regardless of contract status.

---

**DSP-02: Do you have a documented data classification policy?**

Yes. Four tiers per `docs/SOC2_READINESS.md §13`:
| Class | Examples | Controls |
|---|---|---|
| **Restricted** | Health profile data, CV biometric data (on-device), workout history linked to user_id, HMAC keys | AES-256-GCM column encryption; RLS isolation; no logging of field values; Privacy Floor enforced |
| **Confidential** | Email addresses, employee names (enterprise tenants), OAuth tokens, API keys | Encrypted at rest; access restricted to owning service; audit log on all access |
| **Internal** | Aggregate analytics, error logs (PII scrubbed), product metrics | Internal access only; pseudonymous identifiers in PostHog |
| **Public** | Marketing content, blog posts, pricing information | No restrictions |

GDPR Article 9 health data is always classified as Restricted, regardless of context.

---

**DSP-03: Do you conduct Data Protection Impact Assessments (DPIAs)?**

Yes. A DPIA for FORM's processing of health-adjacent data (GDPR Article 9) is documented in `docs/GDPR_DPIA.md`. The DPIA covers:
- Processing purposes and legal bases.
- Necessity and proportionality assessment.
- Risk identification and mitigation measures (including on-device CV processing to avoid biometric data leaving the device).
- Constraints applied to AI inference (no PII or health values in Anthropic prompts — pseudonymised exercise context only).
- Privacy Floor as a hard technical constraint at the data layer.

DPIAs are reviewed annually and triggered by any new feature that introduces new health-data processing pathways.

---

**DSP-04: Do you process or store any special-category personal data under GDPR Article 9?**

Yes. FORM processes health-adjacent data as follows:
- **Workout activity data** (exercise type, sets, reps, duration) — health data category under GDPR Art. 9.
- **CV pose estimation data** — processed **entirely on-device** via Apple Vision Framework / MediaPipe Pose. Raw landmark data never leaves the user's device. Only derived metrics (rep count, form score) are transmitted.
- **Recovery and HRV signals** — imported via HealthKit/Health Connect on opt-in basis.

**Privacy Floor (non-negotiable technical constraint):**
HR administrators at enterprise customers never see individual employee health data. Aggregated wellness metrics (activation rate, weekly engagement %, opt-in NPS) are available only when the anonymity threshold (n=15 minimum) is satisfied. This constraint is enforced at the data layer — not a policy toggle.

Evidence: `docs/GDPR_DPIA.md §4`; `docs/DATA_MODEL.md §8` (RLS privacy floor enforcement); `docs/SOC2_READINESS.md §13`.

---

### DCS — Datacenter Security

**DCS-01: Do you operate your own datacentres?**

No. FORM is a cloud-native platform with no on-premises infrastructure. Physical security is inherited from sub-processors' certified facilities:
- **Cloudflare** (edge, CDN, R2): SOC 2 Type II (2025), ISO 27001. Cloudflare's physical security programme covers all facilities globally.
- **Supabase** (Postgres database, authentication): SOC 2 Type I (2024); hosted on DigitalOcean. EU data in Frankfurt `de-fra1`.

FORM does not have independent physical access to any datacentre facility.

---

**DCS-02: Do you have EU data residency options?**

Yes.
- **German / EU tenants**: Cloudflare R2 with `EU-Central` jurisdictional restriction (Frankfurt). Supabase Postgres in `de-fra1` (Frankfurt, EU adequacy applies — no SCCs required for intra-EU processing).
- **UK / IE tenants**: Supabase `eu-west` (Dublin).
- **US tenants**: US region with Standard Contractual Clauses (SCCs Module 2) for EU-originating personal data.

Data residency election is recorded in the enterprise tenant configuration and in the DPA as of signing date. See `docs/SUBPROCESSORS.md §3` for detailed transfer mechanism table.

---

### EKM — Encryption & Key Management

**EKM-01: Do you encrypt data at rest?**

Yes.
- **Database layer**: AES-256-GCM via Supabase `pgsodium` (column-level encryption for health-adjacent and OAuth token columns). Database-level encryption enabled at the Supabase platform level.
- **Backups (R2, Backblaze B2)**: AES-256-GCM, per-object random IVs.
- **Mobile device**: Health data stored in iOS HealthKit / Android Health Connect (OS-native encrypted stores). FORM does not write raw biometric data to its own storage.
- **Prohibited**: AES-128 is prohibited for health data (insufficient margin for GDPR Art. 9 special-category data). AES-ECB is prohibited for all uses.

Reference: `docs/CRYPTOGRAPHY_POLICY.md §2`.

---

**EKM-02: Do you encrypt data in transit?**

Yes.
- **TLS 1.3** on all primary API and web endpoints (ECDHE + AES-256-GCM; forward secrecy mandatory).
- **TLS 1.2** as fallback only for legacy client compatibility (documented in TLS Exception Register per `docs/CRYPTOGRAPHY_POLICY.md §5.1`).
- **TLS 1.0 / TLS 1.1 prohibited** in all environments (Cloudflare edge enforces minimum TLS 1.2; Cloudflare recommends TLS 1.3 for all new connections).
- **HSTS** enforced with 1-year max-age + `includeSubDomains`.

---

**EKM-03: Do you have a documented key management policy including key rotation?**

Yes. `docs/KEY_ROTATION.md` and `docs/CRYPTOGRAPHY_POLICY.md §3` document:

| Key type | Rotation schedule | Custody |
|---|---|---|
| DB encryption master key (Supabase Vault / pgsodium) | Annual | `compliance-officer` escrow + Supabase KMS |
| HMAC audit chain key (DEC-030) | Annual; immediate on compromise | Dual 1Password vaults: `security-engineer` + `compliance-officer` |
| Backup encryption key (R2 / Backblaze B2 WORM) | Annual | Offline encrypted USB (founder physical custody) + 1Password `compliance-officer` vault |
| TLS certificates | Cloudflare auto-renew | Cloudflare-managed; Better Stack alert at T-30 days |
| Supabase Auth JWT secret | Annual; immediate on compromise | 1Password `security-engineer` vault |

Emergency (out-of-cycle) rotation protocol is documented in `docs/KEY_ROTATION.md §6` and triggered by a `key.compromise_suspected` DEC-030 audit event.

---

### GRC — Governance, Risk & Compliance

**GRC-01: Do you have a formal information security risk management programme?**

Yes. FORM maintains a formal risk register (`docs/SOC2_READINESS.md §14`) with:
- Risk identification, likelihood (1–5), impact (1–5), and inherent risk score.
- Control mapping per risk (each risk linked to one or more CC criteria).
- Residual risk after controls applied.
- Owner assigned per risk.
- Quarterly review cadence integrated into the compliance calendar.

Top risks: insider threat (SR-03, medium after HMAC audit log controls applied), supply chain compromise (SR-09, medium), external credential compromise (SR-04, low after MFA enforcement).

---

**GRC-02: Do you have a formal security awareness training programme?**

Yes. All team members with production access complete security awareness training within 5 business days of joining, covering:
- Social engineering and phishing recognition.
- Password hygiene and MFA (hardware key or TOTP; SMS 2FA prohibited).
- Data classification and handling (special focus on Privacy Floor commitment).
- Incident reporting obligations (1-hour reporting window for suspected breach).
- Clinical safety — why health data requires extra care.
- Acceptable use and remote-work security.
- Secure development practices (secure coding, OWASP Top 10).
- Regulatory basics (GDPR Article 9, SOC 2 context).

Annual refresher training required. Completion is tracked as a `cc.training_completed` DEC-030 audit event.

Evidence: Completion artefacts per `docs/ONBOARDING_SECURITY.md §3 Step 2`; programme in `docs/SOC2_READINESS.md §22`.

---

**GRC-03: Do you have a responsible disclosure / bug bounty programme?**

Yes. FORM operates a responsible disclosure programme:
- Reporting channel: `security@form.coach` (encrypted PGP key available on request).
- Scope: All FORM-operated systems at `form.coach` and `*.form.coach`. Out of scope: third-party sub-processor infrastructure (Supabase, Cloudflare, etc.).
- 90-calendar-day disclosure window.
- Severity / remediation SLAs per CVSS 3.1:

| Severity | Patch SLA | Confirmation SLA |
|---|---|---|
| Critical (CVSS ≥ 9.0) | 1 business day (24h target) | Same business day |
| High (CVSS 7.0–8.9) | 3 business days (7-day target) | 1 business day |
| Medium (CVSS 4.0–6.9) | 10 business days (30-day target) | 3 business days |
| Low (CVSS 0.1–3.9) | 90 days or next quarterly review | 5 business days |

FORM does not pursue legal action against researchers who follow the programme's rules. Safe-harbour language applies.

---

### HRS — Human Resources

**HRS-01: Do you conduct background checks on employees with access to customer data?**

Reference checks are conducted for all engineering and operations hires with production access. Background check scope is determined by local law (Ukraine jurisdiction for founding engineering team). Production access is provisioned only after the security onboarding checklist (`docs/ONBOARDING_SECURITY.md`) is complete.

---

**HRS-02: Do you have an employee termination / offboarding procedure?**

Yes. `docs/SOC2_READINESS.md §23.3` documents the offboarding procedure:
- All production credentials revoked within 2 hours of termination notice (or immediately for involuntary termination).
- SCIM deprovision for enterprise-tier operators via the admin dashboard.
- 1Password vault access removed.
- GitHub, Cloudflare, Supabase, Sentry, and PostHog access reviewed and revoked.
- Offboarding is logged as a `user.access_revoked` DEC-030 audit event.

---

### IAM — Identity & Access Management

**IAM-01: Do you support enterprise SSO (SAML 2.0 / OIDC)?**

Yes. FORM supports:
- **SAML 2.0** (IdP-initiated and SP-initiated flows).
- **OIDC** (Authorization Code flow with PKCE).
- Supported IdPs (verified configurations): Okta, Azure AD / Entra ID, Google Workspace, OneLogin, JumpCloud.
- Per-IdP setup guides: `docs/SSO_CLIENT_CONFIG.md`.
- Implementation details (attribute mapping, role sync): `docs/SSO_SCIM_IMPLEMENTATION.md`.
- SSO is implemented via **WorkOS** (SP-11 in `docs/SUBPROCESSORS.md`) — SOC 2 Type II certified (2024).

---

**IAM-02: Do you support SCIM v2 for automated user provisioning?**

Yes. FORM supports **SCIM v2** for:
- User create / update / deactivate (soft-delete; data retained per §12 retention policy).
- Group-to-RBAC-role mapping (directory groups → FORM roles: `owner`, `admin`, `manager`).
- Provisioning triggers: SCIM `PUT /Users/{id}` with `active: false` triggers session invalidation and access revocation within 60 seconds.

Full endpoint specification: `docs/SSO_SCIM_IMPLEMENTATION.md §8`.

---

**IAM-03: Do you enforce multi-factor authentication?**

Yes.
- **Consumer users**: MFA available (TOTP); not enforced by default.
- **Enterprise tenants**: MFA enforcement configurable at tenant level by the `owner` role. When `mfa_enforcement: required`, users who do not complete MFA setup cannot access FORM. Session policy (max duration, IP allowlist) is configurable per tenant.
- **FORM production access** (internal team): MFA is mandatory for all production systems. SMS 2FA is prohibited. Hardware security keys (FIDO2/WebAuthn) or TOTP are the only permitted MFA methods. Enforced from day one per `docs/ONBOARDING_SECURITY.md §3 Step 3`.

---

**IAM-04: Do you implement least-privilege access controls?**

Yes.
- **Supabase RLS**: Every database query is executed in a transaction with `SET LOCAL role = 'authenticated'; SET LOCAL jwt.claims.tenant_id = '<tenant_id>';`. RLS policies enforce that a query cannot return rows belonging to a different tenant.
- **RBAC (enterprise tier)**: Three roles — `owner` (full admin including SSO config and billing), `admin` (user management, aggregate reporting), `manager` (read-only aggregated wellness metrics for their team). Managers never see individual user data.
- **FORM internal**: Least-privilege access per system. Quarterly access review confirms all credentials are still required and appropriately scoped. Break-glass access (emergency production access) requires 2-person approval + time-limited role (4-hour maximum) + every action logged + customer notified if data was touched.

Evidence: `docs/DATA_MODEL.md §8`; `docs/SOC2_READINESS.md §23`.

---

### IVS — Infrastructure & Virtualization Security

**IVS-01: Do you use a cloud-native, managed infrastructure model?**

Yes. FORM is entirely cloud-native:
- **Compute**: Cloudflare Workers (serverless edge compute; no VMs to patch; runtime isolation via V8 isolates — no shared memory between tenant requests).
- **Database**: Supabase (managed Postgres; Supabase manages OS patching, database version upgrades, and security updates).
- **CDN / WAF**: Cloudflare (automatic security patches for edge layer).

No server infrastructure is operated or maintained directly by FORM. Patch management for infrastructure components is delegated to cloud providers with SOC 2 / ISO 27001 certifications.

---

**IVS-02: Do you isolate customer data at the infrastructure level?**

Yes. Multi-tenant isolation is implemented at two levels:
1. **Row-Level Security (RLS)** in Postgres: Every row in every user-facing table includes `tenant_id`. RLS policies enforce that authenticated sessions can only access rows where `tenant_id` matches the JWT claim. Cross-tenant queries are blocked by the database engine, not by application-layer filtering.
2. **Request isolation**: Cloudflare Workers V8 isolates prevent memory sharing between tenant requests. Worker execution contexts are ephemeral; no cross-request state persistence.

Cross-tenant isolation tests run in CI on every PR. A PR that breaks RLS isolation cannot be merged. Evidence: `docs/DATA_MODEL.md §5`; `docs/SOC2_READINESS.md §57`.

---

### IPY — Interoperability & Portability

**IPY-01: Do you support data portability (export of customer data)?**

Yes.
- **Enterprise tenant data export**: Enterprise admins can export aggregated wellness metrics from the admin dashboard in CSV format. Individual user data is not available to enterprise admins (Privacy Floor constraint).
- **User-level data portability (GDPR Art. 20)**: Individual users can export their full personal data profile (workout history, meal log, health profile) in JSON format via the app settings. This includes all data associated with their account.
- **Audit log export**: Available via REST API (`GET /v1/audit-log`), S3 sync (Growth+), and SIEM webhooks (Growth+) in JSON Lines format.

---

**IPY-02: Can customers terminate service and retrieve their data?**

Yes. Upon contract termination:
1. Enterprise admin can trigger a full tenant data export (30-day export window).
2. FORM confirms export completion and deletes tenant data from production within 90 days of termination.
3. HMAC-chained audit logs are retained for statutory periods regardless of contract status.
4. Individual user accounts are migrated to a consumer tier if the user opts in (data ownership stays with the user, not the employer).

---

### LOG — Logging & Monitoring

**LOG-01: Do you maintain audit logs of administrative and user actions?**

Yes. FORM maintains a **tamper-evident, HMAC-chained audit log** (DEC-030) with the following properties:
- **Append-only**: No DELETE or UPDATE permitted on `audit_log` table.
- **HMAC-SHA256 chaining**: Each row includes `hmac_prev` (HMAC of the previous row) and `hmac_self` (HMAC of the current row's canonical payload). Any retroactive modification breaks the chain and triggers a tamper alert.
- **Retention**: 7 years for SOC 2 / financial events; 3 years for operational events; 30 days for read-only access events.
- **Scope**: All privileged actions (SSO configuration changes, user provisioning/deprovisioning, RBAC role changes, admin data access, support access to tenant data, key rotation events, billing changes).
- **P95 delivery latency**: < 5 seconds (SLA commitment per `docs/ENTERPRISE_SLA.md §3`).

Schema specification: `docs/AUDIT_LOG_SCHEMA.md`. DEC-030 event taxonomy: `docs/AUDIT_LOG_SCHEMA.md §2`.

---

**LOG-02: Do you provide customers access to their audit logs?**

Yes. Enterprise customers access audit logs via:
- **Admin dashboard UI**: Searchable audit log viewer (Starter+).
- **REST API** (`GET /v1/audit-log`): Paginated JSON export (Starter+).
- **S3 sync**: Continuous delivery of audit log events to customer-owned S3 bucket (Growth+).
- **SIEM webhooks**: Native exporters for Datadog, Splunk, and Sumo Logic (Growth+).

Note on SIEM export and sub-processor chain: if the customer routes audit log exports to a third-party MSSP not under FORM's sub-processor list, a DPA addendum is required (as that MSSP becomes a downstream sub-processor). See `docs/SUBPROCESSORS.md §6`.

---

**LOG-03: Do you have a security monitoring programme with alerting on anomalous events?**

Yes. FORM's observability stack includes:
- **Better Stack** (uptime monitoring + alerting): Production endpoint health probes, PagerDuty-integrated alert routing.
- **Sentry** (error monitoring): Application errors, with `beforeSend` PII scrubbing hook enforced — no health field values in error payloads.
- **PostHog** (product analytics): Anonymised event stream (pseudonymous `distinct_id`; no health metrics in analytics).
- **Security alerting**: Supabase Realtime subscription on `audit_log` for critical events (`data.read_individual`, `auth.break_glass_activated`, `key.compromise_suspected`); alerts routed to `#security-alerts` Slack channel.
- **HMAC chain integrity verification**: Scheduled job verifies audit chain integrity; break triggers P0 alert.

Observability architecture: `docs/OBSERVABILITY.md`. SLO taxonomy: `docs/OBSERVABILITY.md §4`.

---

**LOG-04: Can customers independently verify the integrity of their exported audit log events?**

Yes. FORM publishes the full chain-verification algorithm specification and test vector as Data Room artefact **HMAC-VERIFY-ALGO-001** (`compliance/docs/hmac-chain-verification-algorithm.md`).

- **Algorithm specification:** DEC-030 chain structure (event_id, event_type, payload, created_at, hmac_signature, previous_event_id), HMAC input string format, sentinel value for chain-start, canonical payload serialisation (JSON.stringify with sorted keys, no whitespace).
- **Verification pseudocode:** Python 3.x reference implementation with `verify_chain()` function; reports `SIGNATURE_MISMATCH` and `CHAIN_POINTER_MISMATCH` for any detected tamper. Implementation time: < 2 hours.
- **Test vector:** Three-event synthetic chain with a known HMAC secret; allows customers to confirm their implementation produces expected signatures before running against production exports.
- **Per-tenant HMAC verification key:** Each enterprise tenant receives a dedicated `tenant_hmac_verify_key` (HKDF-SHA256 derived from the FORM master secret; 32 bytes hex) available at Admin Dashboard → Security → Audit Export. Tenants verify their own chain without FORM exposing the master secret.
- **FORM-side monitoring:** Chain integrity is independently monitored by FORM via an automated daily job (AL-SIEM-05, P0 severity). Any detected chain break auto-opens incident R-01. Customer-side verification is a due-diligence option, not a prerequisite for chain integrity.
- **Library availability:** A FORM-provided Python verification library is available to enterprise pilot customers who request it via their CSM. Requests are tracked and a library is authored when ≥ 2 distinct pilot customers request it (DEC-071).

**Standard security questionnaire response (audit log integrity and verification):**

*"FORM publishes the full chain-verification algorithm specification and test vector as Data Room artefact HMAC-VERIFY-ALGO-001. Enterprise customers can implement verification in < 2 hours using the provided pseudocode. FORM also monitors chain integrity internally via DEC-030 event AL-SIEM-05 (P0 alert, auto-opens incident R-01 on any detected chain break). A FORM-provided library is available to enterprise pilot customers who request it via their CSM."*

Algorithm specification: `docs/OBSERVABILITY.md §50` (DEC-071). SOC 2 mapping: CC1.1 (entity communicates control environment integrity), C1.1 (confidentiality — algorithm spec demonstrates what data is in the chain and what is excluded per privacy floor).

---

### SEF — Security Incident Management, E-Discovery & Cloud Forensics

**SEF-01: Do you have a documented security incident response plan?**

Yes. `docs/INCIDENT_RESPONSE.md` is FORM's authoritative incident response document (13 runbooks covering P0–P3 incidents across security, availability, data privacy, and legal categories). Key elements:

- **Incident classification**: P0 (platform down / confirmed breach / GDPR 72h trigger), P1 (significant degradation / suspicious access), P2 (single-component failure), P3 (minor issues).
- **Response SLAs** (Enterprise-tier customers):
  - P0: Acknowledge ≤ 15 minutes; first update ≤ 1 hour; resolve ≤ 4 hours.
  - P1: Acknowledge ≤ 1 hour; first update ≤ 4 hours.
  - P2: Acknowledge ≤ 4 hours; resolve ≤ 24 hours.
- **GDPR breach notification**: To supervisory authority within 72 hours of becoming aware of a qualifying breach. To affected enterprise customers within 24 hours of FORM becoming aware that their tenant data may be affected.
- **Post-mortem**: All P0 and P1 incidents trigger a blameless post-mortem within 48 hours. Enterprise customers receive a summary within 5 business days.

Evidence: `docs/INCIDENT_RESPONSE.md`; `docs/ENTERPRISE_SLA.md §6`.

---

**SEF-02: Do you notify customers in the event of a security incident or data breach?**

Yes. Notification obligations:
- **Enterprise customers**: Notified within 24 hours of FORM confirming that the incident affects their tenant data. Notification includes: incident description, data categories affected, approximate number of users affected, containment steps taken, and contact for ongoing communication.
- **GDPR supervisory authority**: Within 72 hours of FORM becoming aware of a qualifying personal data breach per GDPR Art. 33.
- **Individual users**: If required under GDPR Art. 34 (high risk to individual rights) — notification within 30 days of breach confirmation.

Notification is delivered via the enterprise admin's registered email and via the admin dashboard notification centre.

---

**SEF-03: Do you retain forensic evidence following a security incident?**

Yes. Evidence preservation procedures are documented in `docs/INCIDENT_RESPONSE.md §5` (Evidence Preservation Protocol):
- HMAC-chained audit log is immutable and preserved automatically.
- Database PITR snapshot taken at incident declaration time (before any remediation).
- Worker trace logs exported and archived.
- Private Slack channel created (`#inc-YYYYMMDD-name`) with access restricted to incident responders; channel export preserved as evidence.
- All evidence stored in `compliance/evidence/incidents/` for 7 years.

---

### STA — Supply Chain Management, Transparency & Accountability

**STA-01: Do you maintain a list of sub-processors?**

Yes. FORM's sub-processor register is maintained at `docs/SUBPROCESSORS.md` and published publicly at `form.coach/legal/sub-processors`. The register includes:
- 11 active sub-processors as of document effective date.
- For each: company name, processing purpose, data categories processed, region/jurisdiction, legal transfer mechanism, DPA status, and certifications.
- Tier classification: T1 Critical (full SOC 2 review + DPA + right-to-audit), T2 Important (SOC 2 bridge letter or security questionnaire + DPA).

**30-day advance notice** of any sub-processor additions or replacements is provided to enterprise customers via admin dashboard notification and the published register update.

---

**STA-02: Do you have a vendor security review programme?**

Yes. `docs/SOC2_READINESS.md §17` and `docs/SUBPROCESSORS.md §4` document the vendor risk management (VRM) programme:
- T1 Critical vendors: SOC 2 report review + security questionnaire + right-to-audit clause in DPA. Annual review.
- T2 Important vendors: SOC 2 bridge letter or 8-item security questionnaire. Annual review.
- New vendors: VRM assessment required before onboarding (documented in `docs/VENDOR_REGISTRY.md`).

Outstanding DPA executions (VRM-GAP-001): see `docs/SUBPROCESSORS.md §3` — Supabase, Resend, RevenueCat, Sentry DPAs pending execution (target: M6).

---

**STA-03: Do you have a software supply chain security programme?**

Yes.
- **Dependency scanning**: `npm audit` / `cargo audit` on every PR and scheduled nightly.
- **Secrets scanning**: TruffleHog + GitHub Advanced Security (GHAS) push protection on all branches.
- **SBOM**: Software Bill of Materials generated at each production release.
- **Pinned dependencies**: Production dependency versions are pinned to specific hashes; automated PRs for updates are reviewed before merge.
- **Third-party code review**: Any dependency with > 100k weekly downloads or with access to user data undergoes a manual security review before first introduction.

---

### TVM — Threat & Vulnerability Management

**TVM-01: Do you have a vulnerability management programme?**

Yes. `docs/SOC2_READINESS.md §16` documents FORM's vulnerability management programme:

| Severity | Patch SLA | Process |
|---|---|---|
| Critical (CVSS ≥ 9.0) | 24 hours | Emergency deploy; security-engineer on call |
| High (CVSS 7.0–8.9) | 7 calendar days | Out-of-cycle release |
| Medium (CVSS 4.0–6.9) | 30 calendar days | Scheduled release |
| Low (CVSS 0.1–3.9) | 90 days or next quarterly review | Batch |

**Patch cadence**: Cloudflare Worker dependencies reviewed weekly; Supabase platform patching managed by Supabase (no operator action required for OS patches); mobile app dependencies reviewed with each release cycle.

---

**TVM-02: Do you conduct penetration tests?**

Yes. FORM's penetration testing programme (`docs/SOC2_READINESS.md §16`):
- **Frequency**: Annual, by a qualified third-party vendor.
- **Scope**: FORM API (`api.form.coach`), web application (`form.coach`), admin dashboard, SSO/SCIM endpoints, Cloudflare Workers edge layer.
- **First engagement**: Scheduled for Month 7 of platform operation (pre-commercial launch).
- **Findings tracking**: All findings tracked in Linear with `label:pentest-finding`. Critical / High findings resolved before next commercial deployment.
- **Customer sharing**: Enterprise customers receive the penetration test summary findings report under NDA upon request.

---

## §2 — Extended Enterprise DDQ Sections

### 2.1 Data Residency and Sovereignty

| Question | FORM Response |
|---|---|
| Where is customer data stored? | EU tenants: Frankfurt, Germany (Supabase `de-fra1`, Cloudflare R2 EU jurisdictional restriction). US tenants: US East. UK/IE tenants: Dublin, Ireland. |
| Can data residency be customised? | Yes, at the enterprise tier. Data residency election is recorded in the DPA and in the tenant configuration at onboarding. |
| Does data leave the selected region for processing? | CV pose estimation: never (on-device only). AI coaching context: routed to Anthropic (US) as pseudonymised exercise context only — no PII, no health values. PostHog: EU Cloud (Frankfurt) for EU tenants. |
| Do you comply with data localisation laws for regulated industries? | FORM processes GDPR Art. 9 health data. FORM does not process data subject to HIPAA (US healthcare), financial services regulations, or defence/government data. Insurance risk-scoring and government backdoor contracts are explicitly declined. |
| Is your organisation subject to laws that may require disclosure to authorities without notification to the data subject? | FORM's legal entity is Ukrainian. Compulsory legal process is handled per `docs/INCIDENT_RESPONSE.md §15` (Law Enforcement Demand runbook). FORM does not voluntarily disclose user data to any authority; only compulsory process (subpoena, court order, or equivalent) triggers the procedure, and customers are notified to the extent permitted by law. FORM declines contracts that require government backdoor access — this is a hard no-go (see `docs/ENTERPRISE.md`). |

---

### 2.2 Privacy Floor — What FORM Will Never Do

The following restrictions are enforced at the data layer. They cannot be overridden by contract terms, customer request, or any internal approval level. `clinical-safety` has veto power.

| Never | Rationale |
|---|---|
| HR sees individual user activity, workout, or health records | Privacy Floor; enforced via RLS policies and RBAC (manager role excludes individual data access) |
| Manager-level reports on direct reports' health data | Privacy Floor; anonymity threshold n=15 minimum for all aggregate metrics |
| Eating disorder screening data aggregated to employer dashboards | Clinical safety floor; enforced at data model level |
| Body composition data surfaced to the employer | Privacy Floor |
| Mental health signals aggregated for employer reporting | Privacy Floor |
| Employee unable to revoke company-association at any time | GDPR Art. 7(3) right to withdraw consent; always available in user settings |

---

### 2.3 Incident Response SLAs

| Tier | P0 response | P1 response | P2 response | GDPR breach notification |
|---|---|---|---|---|
| Starter (50–200 seats) | Acknowledge ≤ 1h; first update ≤ 4h | Acknowledge ≤ 4h | Acknowledge ≤ 24h | Within 24h of FORM confirming tenant data affected |
| Growth (201–1,000 seats) | Acknowledge ≤ 1h; first update ≤ 4h | Acknowledge ≤ 4h | Acknowledge ≤ 24h | Within 24h of FORM confirming tenant data affected |
| Enterprise (1,001+ seats) | Acknowledge ≤ 15min; first update ≤ 1h; resolve ≤ 4h (custom SLA addendum) | Acknowledge ≤ 1h; first update ≤ 4h | Acknowledge ≤ 4h; resolve ≤ 24h | Within 24h of FORM confirming tenant data affected |

GDPR supervisory authority notification: within 72 hours of FORM becoming aware of a qualifying breach (GDPR Art. 33).

---

### 2.4 Access Controls — What HR / People Leads See

Enterprise admins and managers see only the following aggregated metrics (minimum anonymity threshold n=15 for all):

| Metric | What it shows | What it does NOT show |
|---|---|---|
| Activation rate | % of invited employees who completed onboarding | Names, departments, individual users |
| Weekly engagement | % of activated employees with ≥1 session in the last 7 days | Per-user sessions, workout types |
| Opt-in NPS | Aggregated satisfaction score from survey participants | Individual responses, names |
| Seat utilisation | Contracted vs. active seats (billing reconciliation) | Per-user breakdown |

HR does not receive — and cannot request — individual-level wellness, workout, or health data under any FORM contract. A request that requires compromising this floor is declined.

---

### 2.5 Contracts We Decline

FORM explicitly declines the following customer types, regardless of contract value:
- **Insurance companies** seeking employee health or fitness data for risk-scoring or underwriting.
- **Government contracts** requiring backdoor access to user data or the ability to conduct unannounced access to employee health records.
- **Wellness-as-punishment programmes** (e.g., benefits that penalise non-participating employees or link fitness data to insurance pricing).
- **Sports leagues or teams** seeking individual athlete performance data for contract valuation or player evaluation (conflict with Privacy Floor and player union agreements).

---

### 2.6 Compliance Status Summary

| Standard / Regulation | Status | Evidence available |
|---|---|---|
| SOC 2 Type II | **In progress** — audit underway via Drata continuous monitoring. Target: Q1 2027. | Drata dashboard access (NDA); summary letter on request |
| GDPR (EU) | **Compliant** — DPIA complete; DPA available; SCCs executed for US sub-processors; data residency options available. | `docs/GDPR_DPIA.md`; DPA template (`security@form.coach`) |
| GDPR Art. 9 (special category data) | **Compliant** — on-device CV processing; DPIA covers health data; Privacy Floor enforced at data layer | `docs/GDPR_DPIA.md §4`; `docs/DATA_MODEL.md §8` |
| ISO 27001 | **Not currently certified** — control framework alignment in progress; target post-Series A | — |
| HIPAA | **Not in scope** — FORM does not serve US covered entities or business associates; no PHI processed | — |
| PCI-DSS | **Cloudflare + Stripe scope** — FORM does not handle card data; PCI-DSS scope is limited to Stripe (Level 1 certified) | Stripe PCI certificate |
| UK GDPR | **Compliant** — UK Addendum to SCCs applied for UK data subjects; data residency in Dublin (EU-West) | DPA template with UK addendum |
| CCPA | **Assessment in progress** — CCPA service provider addendum available on request | — |

---

### 2.7 Requesting Evidence Packages

The following artefacts are available to qualified enterprise prospects and current customers under NDA via `security@form.coach`:

| Artefact | Availability | NDA required |
|---|---|---|
| SOC 2 Type II report (when issued) | Enterprise tier | Yes |
| Drata continuous monitoring dashboard access | All tiers (interim) | Yes |
| Penetration test summary findings | Enterprise tier | Yes |
| GDPR DPA (standard template) | All tiers — signed before any pilot data flows | No |
| GDPR DPA (custom terms) | Enterprise tier | Yes |
| SUBPROCESSORS register | Public | No |
| Security questionnaire (this document) | Public (NDA-gated version for SIG Lite / custom formats) | No (public) / Yes (custom) |
| Architecture overview (security-relevant) | Enterprise tier | Yes |

Turnaround for custom questionnaire formats (SIG Full, HECVAT, CAIQ Full): 5 business days.

---

## §3 — Changelog

| Version | Date | Changes |
|---|---|---|
| v1.1 | 2026-06-20 | LOG-04 added: customer-side HMAC audit log integrity verification. References HMAC-VERIFY-ALGO-001 (DEC-071, `docs/OBSERVABILITY.md §50`). Standard questionnaire response language per §50.8. Closes `docs/OBSERVABILITY.md §50.10` item 5 (P1/M10). Owner: security-engineer + compliance-officer. |
| v1.0 | 2026-06-06 | Initial release. 16 CAIQ Lite domains + §2 extended DDQ sections. Reviewed against `docs/SOC2_READINESS.md`, `docs/CRYPTOGRAPHY_POLICY.md`, `docs/AUDIT_LOG_SCHEMA.md` (DEC-030), `docs/SUBPROCESSORS.md`, `docs/INCIDENT_RESPONSE.md`, `docs/DATA_MODEL.md`, `docs/ENTERPRISE.md`. |

*Next review due: 2027-06-20 or immediately upon material change to security posture, sub-processor list, or SOC 2 audit status.*

---

*Owner: `security-engineer` + `compliance-officer`. Veto: `clinical-safety` on any privacy-floor-related disclosure. Distribution: NDA-gated via `security@form.coach`. Public CAIQ Lite summary PDF generated from this source document.*
