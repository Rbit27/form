# FORM · Enterprise Shared Responsibility Model v1.0

> **Owner:** enterprise-architect + compliance-officer + security-engineer  
> **Audience:** Enterprise customers (security teams, IT, legal, procurement), FORM customer-success  
> **Review:** Quarterly and on any material architecture or contract change  
> **References:** docs/ENTERPRISE.md, docs/DATA_MODEL.md, docs/SSO_SCIM_IMPLEMENTATION.md, docs/OBSERVABILITY.md, docs/INCIDENT_RESPONSE.md, docs/SOC2_READINESS.md, DEC-030, docs/ENTERPRISE_SLA.md

---

## Table of Contents

1. [Purpose and How to Read This Document](#1-purpose-and-how-to-read-this-document)
2. [Summary Matrix](#2-summary-matrix)
3. [Infrastructure Security](#3-infrastructure-security)
4. [Application and Platform Security](#4-application-and-platform-security)
5. [Identity and Access Management](#5-identity-and-access-management)
6. [Data Security and Privacy](#6-data-security-and-privacy)
7. [Audit Logging and Evidence](#7-audit-logging-and-evidence)
8. [Operations, Monitoring, and Incident Response](#8-operations-monitoring-and-incident-response)
9. [Compliance and Regulatory Obligations](#9-compliance-and-regulatory-obligations)
10. [Enterprise Onboarding and Offboarding](#10-enterprise-onboarding-and-offboarding)
11. [Liability Boundaries and Escalation Path](#11-liability-boundaries-and-escalation-path)
12. [Non-Negotiable Floors](#12-non-negotiable-floors)
13. [Glossary](#13-glossary)
14. [Version History](#14-version-history)

---

## 1. Purpose and How to Read This Document

### 1.1 Why a Shared Responsibility Model Exists

FORM is a multi-tenant SaaS product. When your organisation deploys FORM for employees, security controls span two domains: the infrastructure and application FORM operates, and the identity, configuration, and policy layer your IT and security teams own. A clear division of responsibility is required for:

- **SOC 2 Type II audits** — auditors must attribute each control to its owner. Ambiguity creates control gaps.
- **GDPR compliance** — the controller–processor boundary determines who is legally responsible for each processing activity (GDPR Art. 4(7)–(8), Art. 28).
- **Incident response** — when something goes wrong, both parties must know who acts first and who supports.
- **Due diligence** — your security team can sign off faster when responsibilities are explicit.

### 1.2 How to Read the Tables

Each table uses three columns:

| Column | Meaning |
|---|---|
| **FORM** | FORM engineering, security-engineer, devops-lead, or compliance-officer is the primary owner. Failure here is FORM's liability. |
| **Shared** | Both parties have a role. Clarified by sub-bullet. |
| **Customer** | Your IT, security, or HR team is the primary owner. FORM provides tooling; you operate it. |

Where a row is marked **Shared**, the sub-description specifies what each party does. Silence in a column means that party has no obligation in that row.

### 1.3 Scope of This Document

This document covers the **enterprise tier** only (Starter, Growth, and Enterprise plans). Consumer-tier subscribers (individual Pro) operate under a simplified model where FORM holds nearly all responsibility because there is no customer-administered layer.

---

## 2. Summary Matrix

Quick-reference view across all domains. Detailed tables follow in Sections 3–10.

| Domain | FORM | Shared | Customer |
|---|:---:|:---:|:---:|
| Physical infrastructure (Cloudflare, Supabase) | ✅ | | |
| Network security and DDoS protection | ✅ | | |
| Platform patching (OS, Postgres, Workers runtime) | ✅ | | |
| Application security (auth, API, CV pipeline) | ✅ | | |
| Encryption at rest and in transit | ✅ | | |
| Backup and disaster recovery | ✅ | | |
| Uptime and SLA adherence | ✅ | | |
| Vulnerability management (FORM-owned code/infra) | ✅ | | |
| Penetration testing (annual) | ✅ | | |
| IdP configuration (Okta, Azure AD, Google Workspace) | | ✅ | |
| MFA policy enforcement | | ✅ | |
| SCIM provisioning setup | | ✅ | |
| Role-to-group mapping | | ✅ | |
| Session timeout policy | | ✅ | |
| GDPR controller obligations (employee data) | | | ✅ |
| Art. 13–14 employee notice (pre-login SCIM data) | | | ✅ |
| Data Subject Access Request fulfillment | | ✅ | |
| Employee offboarding from FORM | | ✅ | |
| Acceptable use policy enforcement | | | ✅ |
| Incident response (P0/P1 — FORM infrastructure) | ✅ | | |
| Incident response (P2/P3 — user-impacting on customer side) | | ✅ | |
| Compliance attestation (your industry) | | | ✅ |
| SOC 2 Type II report sharing | ✅ | | |
| DPA execution | | ✅ | |
| Audit log retention (FORM infrastructure events) | ✅ | | |
| Audit log review (admin actions, your tenant) | | | ✅ |
| White-label branding configuration | | ✅ | |
| Custom domain (CNAME setup) | | ✅ | |

---

## 3. Infrastructure Security

FORM runs on Cloudflare Workers (edge compute), Supabase (managed Postgres), and Cloudflare R2 (object storage). The underlying infrastructure is operated by these sub-processors, each holding SOC 2 Type II and ISO 27001 certifications. FORM is responsible for its use of these services; the sub-processors are responsible for the physical and network layers beneath them.

### 3.1 Physical Infrastructure

| Control | FORM | Shared | Customer |
|---|---|---|---|
| Data centre physical security | ✅ Cloudflare / Supabase data centres — SOC 2 certified. FORM does not operate physical infrastructure. | | |
| Hardware lifecycle and disposal | ✅ Sub-processor responsibility (Cloudflare, Supabase). | | |
| Environmental controls (power, cooling) | ✅ Sub-processor responsibility. | | |
| Geographic region selection | ✅ FORM assigns tenants to EU (Frankfurt) or US (Virginia) regions based on contract. | ✅ Customer specifies region requirement during onboarding. | |

### 3.2 Network Security

| Control | FORM | Shared | Customer |
|---|---|---|---|
| DDoS mitigation | ✅ Cloudflare enterprise DDoS protection on all FORM endpoints. | | |
| TLS termination | ✅ TLS 1.2 minimum, TLS 1.3 preferred. Enforced at Cloudflare edge. HSTS with `max-age=31536000; includeSubDomains; preload`. | | |
| API rate limiting | ✅ Per-tenant and per-user rate limits enforced at Worker layer. Overage grace of 500 requests before hard block (DEC-045). | | |
| FORM internal network segmentation | ✅ Supabase VPC; Workers communicate via private service bindings, not public internet, where available. | | |
| Customer network (egress from your IdP to FORM) | | | ✅ Customer ensures outbound HTTPS is permitted from their IdP to FORM's SSO endpoints. FORM publishes required IP ranges in `docs/SSO_CLIENT_CONFIG.md`. |
| IP allowlisting for admin dashboard | | ✅ FORM provides the technical control (`session_policy.ip_allowlist`). Customer configures the list during onboarding. | |

### 3.3 Platform Patching and Updates

| Control | FORM | Shared | Customer |
|---|---|---|---|
| Cloudflare Workers runtime updates | ✅ Cloudflare manages runtime. FORM monitors for deprecations. | | |
| Supabase Postgres version upgrades | ✅ FORM owns the upgrade schedule and tests migrations in staging before production. | | |
| FORM application dependency updates | ✅ Dependabot + weekly security patch sprint. Critical CVEs patched within 72 hours per `docs/VULNERABILITY_MANAGEMENT.md`. | | |
| Mobile app updates (iOS / Android) | ✅ FORM publishes updates. | | ✅ Customer cannot force app updates on employee devices. Employees update via App Store / Google Play on their own device management cadence. |
| MDM / EMM management of FORM app | | | ✅ If your organisation requires managed app distribution, your MDM team configures the deployment. FORM does not integrate with MDM management planes. |

---

## 4. Application and Platform Security

### 4.1 Application Authentication

| Control | FORM | Shared | Customer |
|---|---|---|---|
| SSO integration (SAML 2.0 / OIDC) | ✅ FORM maintains the SP-side implementation (assertion validation, PKCE, token issuance). | | |
| IdP configuration (Okta, Entra ID, Google Workspace, OneLogin, ADFS, Ping) | | ✅ FORM provides configuration guides in `docs/SSO_CLIENT_CONFIG.md`. Customer IT configures the IdP application. | |
| SP metadata / JWKS publication | ✅ FORM publishes at `https://api.form.coach/auth/sso/{tenant_slug}/saml/metadata` and `https://api.form.coach/auth/sso/.well-known/openid-configuration`. | | |
| Certificate rotation | ✅ FORM rotates signing certificates 90 days before expiry and publishes updated metadata. | ✅ Customer IT must update the SP certificate in IdP configurations that do not support automatic metadata refresh (e.g., ADFS). FORM notifies 60 days in advance via CSM and technical contact email. | |
| Credential stuffing protection | ✅ Consumer-tier: `fail2ban`-equivalent rate limiting + CAPTCHA threshold on password auth. Enterprise-tier: SSO is the only auth path once enabled (password auth disabled per tenant config). | | |
| API key management (Admin API) | ✅ FORM issues API keys; keys are hashed at rest (HMAC-SHA256). | ✅ Customer is responsible for storing API keys securely. Leaked keys must be reported to FORM immediately for revocation. | |

### 4.2 Application Vulnerability Management

| Control | FORM | Shared | Customer |
|---|---|---|---|
| Secure SDLC | ✅ Code review required before merge. SAST (CodeQL) in CI. No secrets in source code (gitleaks pre-commit hook). | | |
| Annual penetration test | ✅ FORM commissions an independent external penetration test annually. Findings are remediated per severity SLA in `docs/PENETRATION_TESTING.md`. Summary shared with enterprise customers under NDA. | | |
| Bug bounty / responsible disclosure | ✅ FORM operates a responsible disclosure programme at `docs/RESPONSIBLE_DISCLOSURE.md`. | | |
| Third-party application integrations (your tools) | | | ✅ Customer is responsible for securing any third-party tools that access FORM via the Admin API. OAuth tokens issued to customer-operated integrations are the customer's responsibility to rotate and revoke. |
| Custom code (customer builds on top of Admin API) | | | ✅ Customer-authored code interacting with FORM APIs is outside FORM's security scope. |

### 4.3 Computer Vision (CV) Pipeline

| Control | FORM | Shared | Customer |
|---|---|---|---|
| On-device inference (pose estimation) | ✅ CV models run on-device. Raw camera frames are never uploaded. Only derived pose keypoints are transmitted. | | |
| Model integrity | ✅ Models are signed. The app validates the signature before loading. | | |
| Camera permission | ✅ FORM requests camera permission only at the moment CV coaching is initiated, using the OS-native permission prompt. | | ✅ Employees control camera permission. Enterprise IT may restrict camera access via MDM policy — this will disable CV coaching for those devices. |
| Workout session data retention | ✅ Session data retained per the schedule in `docs/DATA_MODEL.md §12`. | | |

---

## 5. Identity and Access Management

### 5.1 User Lifecycle

| Control | FORM | Shared | Customer |
|---|---|---|---|
| User provisioning (manual) | | ✅ FORM admin dashboard provides bulk CSV import and manual invite. Customer HR/IT uses these tools. | |
| User provisioning (SCIM 2.0) | ✅ FORM maintains the SCIM v2 endpoint. | ✅ Customer IT configures the SCIM connector in their IdP. FORM provides per-IdP guides in `docs/SSO_CLIENT_CONFIG.md`. | |
| JIT (Just-In-Time) provisioning | ✅ FORM creates user records on first successful SSO assertion when JIT is enabled. | ✅ Customer controls whether JIT is enabled (`jit_provisioning_enabled` in tenant config, set during onboarding). Regulated environments typically disable JIT and require SCIM-first provisioning. | |
| User deprovisioning | | ✅ FORM revokes all active sessions synchronously when a SCIM deactivation signal arrives. Customer IT is responsible for removing the user from the IdP group or disabling the IdP account to trigger SCIM deactivation. Deprovisioning does **not** happen automatically on HR system termination unless the HR system is connected to the IdP. | |
| `tenant_owner` role changes | ✅ `tenant_owner` cannot be assigned via SCIM or IdP group mapping — this is a hard constraint enforced at the schema level. Changes require explicit request to FORM enterprise-architect. This prevents IdP compromise from becoming a full org takeover. | | |
| Employee's ability to revoke company association | ✅ Any employee can disassociate their FORM account from the tenant at any time from within the app (Settings → Privacy → Leave organisation). FORM enforces this unconditionally. | | |

### 5.2 Role-Based Access Control (RBAC)

FORM maintains four enterprise roles. The mapping from customer IdP groups to FORM roles is configured by the customer.

| Role | Permission scope | Who assigns |
|---|---|---|
| `tenant_owner` | Full org control: billing, SSO config, delete org | FORM enterprise-architect only (cannot be mapped via SCIM or SSO group) |
| `tenant_admin` | All admin capabilities except delete org and billing transfer | Customer IT via SCIM group mapping or manual assignment |
| `tenant_manager` | Read-only aggregate reports; manage assigned user groups | Customer IT via SCIM group mapping |
| `member` | Personal coaching access only | All provisioned users by default |

| Control | FORM | Shared | Customer |
|---|---|---|---|
| Role schema and enforcement | ✅ Roles enforced in JWT claims + database RLS. | | |
| Role-to-group mapping configuration | | ✅ FORM provides the mapping configuration interface in the admin dashboard. Customer IT defines which IdP groups map to which FORM roles. | |
| Principle of least privilege enforcement in IdP | | | ✅ Customer is responsible for granting only the minimum necessary FORM role to each employee group. |
| Access review (who has `tenant_admin` and `tenant_owner` access) | | ✅ FORM provides the audit log viewer and SCIM roster API. Customer's security or IT team is responsible for running periodic access reviews. FORM recommends quarterly. | |

### 5.3 MFA and Session Policy

| Control | FORM | Shared | Customer |
|---|---|---|---|
| MFA for enterprise SSO users | ✅ FORM enforces that SSO is the **only** authentication path once it is configured for a tenant. MFA on the SSO path is the IdP's responsibility. | ✅ Customer IdP administrator is responsible for enforcing MFA on the IdP application. FORM cannot enforce MFA independently of the IdP. FORM **strongly recommends** MFA enforcement and will flag its absence in onboarding. | |
| MFA for `tenant_admin` dashboard access | ✅ Dashboard access is gated on a valid SSO session. MFA enforcement falls back to the IdP. | | |
| Session duration | ✅ FORM enforces the maximum session duration configured for the tenant (`session_timeout_hours`, default 168h / 7 days). Sessions expire unconditionally at the configured maximum. | ✅ Customer specifies session timeout during onboarding. Regulated environments (healthcare, finance) commonly require 8 hours. | |
| Session revocation on deprovisioning | ✅ FORM revokes sessions synchronously on SCIM deactivation. | | |
| IP allowlisting for admin access | ✅ FORM provides the technical control. | ✅ Customer configures the IP allowlist. FORM does not maintain or validate the IP list beyond storing and enforcing it. | |

---

## 6. Data Security and Privacy

### 6.1 Encryption

| Control | FORM | Shared | Customer |
|---|---|---|---|
| Encryption in transit | ✅ TLS 1.2+ on all endpoints. Certificate managed by Cloudflare. HSTS enforced. | | |
| Encryption at rest (Postgres) | ✅ AES-256 encryption at rest on Supabase Postgres (managed by Supabase; SOC 2 evidence available). | | |
| Encryption at rest (object storage — R2) | ✅ Cloudflare R2 AES-256 at rest. | | |
| Health data fields (special-category GDPR Art. 9) | ✅ Health data columns (`body_metrics`, `workout_sessions`, `ed_screening`) use column-level encryption with AES-256-GCM. Keys managed via `docs/KEY_ROTATION.md`. | | |
| Key management and rotation | ✅ Documented in `docs/KEY_ROTATION.md`. Annual rotation schedule. Emergency rotation procedure defined. | | |
| Customer-managed encryption keys (BYOK) | Not currently available. Roadmap: post-Series A. Enterprise customers requiring BYOK must note this as a contractual gap. | | |

### 6.2 Multi-Tenant Data Isolation

FORM uses Row-Level Security (RLS) on a shared Postgres database. This is the primary tenant isolation mechanism. The architecture decision and guarantees are documented in `docs/DATA_MODEL.md §1–4`.

| Control | FORM | Shared | Customer |
|---|---|---|---|
| RLS policy enforcement | ✅ Every application query runs under a Postgres role that has RLS enforced. The `form_admin` (BYPASSRLS) role is restricted to migration runner and compliance toolchain — never the API request path. | | |
| `tenant_id` sourced from JWT claims (not client-supplied) | ✅ `tenant_id` is extracted from the verified JWT, never from client request parameters. | | |
| Cross-tenant RLS penetration test | ✅ Annual penetration test includes explicit cross-tenant isolation testing. A failed RLS bypass is a P0 finding with immediate remediation. | | |
| Fail-closed behaviour | ✅ A missing or invalid `tenant_id` returns 0 rows, never all rows. This is enforced at the policy level, not the application level. | | |
| Tenant data exfiltration via the Admin API | ✅ Admin API enforces tenant scoping at all endpoints. | ✅ Customer is responsible for the security of Admin API keys. A compromised key can read all data within that tenant's scope. | |

### 6.3 GDPR Roles and Responsibilities

Under GDPR, FORM is a **data processor** for employee data, and the enterprise customer is the **data controller**.

| Obligation | FORM (Processor) | Customer (Controller) |
|---|---|---|
| DPA execution | ✅ FORM provides a GDPR-compliant Data Processing Addendum (DPA) based on `docs/MSA_TEMPLATE.md §DPA`. Executed before pilot data flows. | ✅ Customer legal must review and sign the DPA before any employee data enters FORM. |
| Lawful basis for processing employee data | | ✅ Customer must establish and document a lawful basis under Art. 6 (typically Art. 6(1)(b) — performance of employment contract, or Art. 6(1)(a) — consent, depending on the programme design). |
| Art. 13–14 employee notice | ✅ FORM's consumer privacy notice (form.coach/privacy) covers data processed after first login. | ✅ Customer must issue a separate Art. 14 notice to employees about SCIM-provisioned data (directory attributes synced before first login). FORM's privacy notice does not cover this. |
| Standard Contractual Clauses (SCCs) for US-EU transfers | ✅ FORM includes SCCs Module 2 (controller-to-processor) in the DPA for US-region tenants processing EU employee data. | ✅ Customer's legal team must confirm that SCCs are an appropriate transfer mechanism for their legal context (some sector-specific rules impose stricter requirements). |
| Data residency | ✅ FORM assigns EU tenants to Frankfurt region. US tenants to Virginia region. Data does not cross regions in normal operation. | ✅ Customer must specify their region requirement at contract time. Post-migration between regions requires a separate engineering engagement. |
| Sub-processor disclosure | ✅ FORM maintains a public sub-processor list at form.coach/legal/sub-processors (`docs/SUBPROCESSORS.md`). 30-day advance notice for material changes. | ✅ Customer has the right to object to new sub-processors within the 30-day notice window. |

### 6.4 Privacy Floor — Non-Negotiable Data Access Boundaries

These boundaries are enforced by FORM at the application and database layer. They cannot be waived by contract.

| Privacy floor | Enforcement mechanism |
|---|---|
| HR / admin roles never see individual user health data | Aggregate-only API enforced by `tenant_manager` and `tenant_admin` role scope. Health data is accessible only in cohort aggregates (minimum 10 users, suppressed if fewer). |
| No manager-on-direct-reports health reports | Org hierarchy reporting is not a product feature. Cannot be enabled by any configuration. |
| Body composition data never in employer aggregate | `body_metrics` is excluded from all admin reporting schemas. Column-level access control. |
| ED-screening data never in employer aggregate | `ed_screening` is never included in any aggregate export or admin view. |
| Mental health data never in employer aggregate | Coaching session content and mood data are excluded from admin APIs. |
| Employee can revoke company association | Settings → Privacy → Leave organisation is always available to the employee, regardless of tenant SCIM configuration. |
| FORM employees accessing tenant data require 2-person approval + time-bounded break-glass role | Implemented via break-glass PAM procedure. All access logged to `audit_log` with DEC-030 HMAC chain. Tenant admin is notified within 24 hours. |

See `docs/ENTERPRISE.md §Privacy floor` and `docs/DATA_MODEL.md §6` for full enforcement specification.

---

## 7. Audit Logging and Evidence

### 7.1 What FORM Logs

FORM maintains an append-only, HMAC-chained audit log for all privileged actions (DEC-030). The HMAC chain provides tamper-evidence: any modification to a historical log record will break the chain and be detected.

| Log category | Retention | Who can read | Notifies tenant |
|---|---|---|---|
| `auth.*` — login, logout, SSO assertions, MFA events | 3 years | Tenant `tenant_admin` + FORM security-engineer | No |
| `admin.*` — user provisioning, role changes, config changes | 7 years | Tenant `tenant_admin` + FORM security-engineer | No (visible in audit log viewer) |
| `support.*` — FORM employee views tenant data | 7 years | FORM security-engineer + compliance-officer | ✅ Tenant admin email within 24h |
| `data.*` — sensitive field access, bulk exports, DSAR responses | 7 years | FORM compliance-officer | No (available on DSAR request) |
| `security.*` — rate limit hits, quota warnings, break-glass access | 3 years | FORM security-engineer + tenant `tenant_admin` | Alert for quota 80% / 95% |
| `billing.*` — contract events, seat changes, pilot state | 7 years | Tenant `tenant_owner` + FORM billing | No |

Full schema: `docs/AUDIT_LOG_SCHEMA.md`.

### 7.2 Audit Log Access Responsibilities

| Control | FORM | Shared | Customer |
|---|---|---|---|
| Generating and storing audit logs | ✅ All logs generated automatically by the FORM platform. Cannot be disabled by tenant config. | | |
| HMAC chain integrity validation | ✅ Validated on read and by scheduled nightly job. Integrity failure triggers P0 incident. | | |
| Exporting audit logs (Admin API) | ✅ FORM provides `/v1/audit-logs` export endpoint for tenant admins. | | |
| Reviewing audit logs for anomalies | | | ✅ Customer's security team is responsible for reviewing their tenant's audit logs. FORM provides the data; FORM does not perform threat hunting on behalf of customers. |
| Retaining logs beyond FORM's retention period | | | ✅ If customer compliance requirements exceed FORM's retention periods (7 years for admin events), customer is responsible for exporting and storing logs in their own SIEM/archive before the FORM retention period elapses. |
| SIEM integration | | ✅ FORM provides audit log export via API. Customer is responsible for ingesting these logs into their SIEM. FORM does not provide a native SIEM integration at this time. | |

---

## 8. Operations, Monitoring, and Incident Response

### 8.1 Uptime and SLA

| Control | FORM | Shared | Customer |
|---|---|---|---|
| Uptime monitoring | ✅ FORM monitors availability on 60-second intervals from three geographic probes. PagerDuty alert at first failed probe. | | |
| SLA commitment | ✅ 99.9% monthly uptime for API and admin dashboard (Growth and Enterprise plans). 99.5% for Starter. Defined in `docs/ENTERPRISE_SLA.md`. | | |
| Planned maintenance windows | ✅ FORM communicates planned maintenance with a minimum of 7 days' notice via email to the tenant technical contact. Emergency maintenance: best-effort notice, compensating SLA credit if > 30 minutes. | | |
| Status page | ✅ FORM maintains a public status page (status.form.coach). Enterprise customers receive email notification for any incident affecting their region. | | |
| SLA credit claiming | ✅ FORM calculates and issues credits automatically. | ✅ Customer may dispute credit calculations within 30 days of the incident month close. | |

### 8.2 Incident Response

FORM's full incident response process is in `docs/INCIDENT_RESPONSE.md`. The shared responsibilities by incident severity:

**P0 — Critical (data breach or full service outage)**

| Step | Owner |
|---|---|
| Incident detection | FORM (automated alerting) or Customer (report via `security@form.coach`) |
| Incident declaration and command | FORM security-engineer / devops-lead |
| Internal triage | FORM |
| Customer notification (initial) | FORM → tenant technical contact within 1 hour of declaration |
| GDPR 72-hour breach notification (if applicable) | FORM provides the technical facts. Customer (as controller) decides whether Art. 33 notification to DPA is required and drafts the notification. FORM provides a written incident summary within 24 hours of resolution. |
| Employee / HR communication | Customer |
| Post-mortem | FORM (shared with customer under NDA within 5 business days) |

**P1 — High (significant degradation or security vulnerability)**

| Step | Owner |
|---|---|
| Incident response | FORM |
| Customer notification | FORM → tenant technical contact within 4 hours |
| GDPR assessment | FORM (documented in incident record). Shared with customer if personal data at risk. |
| Workaround communication | FORM CSM |

**P2 / P3 — Configuration or customer-controlled issues**

If a P2/P3 incident is caused by customer configuration (e.g., misconfigured SCIM, IdP misconfiguration causing lockout), the customer's IT team leads remediation with FORM's support team available within the standard SLA response window. FORM provides diagnostic tooling (audit logs, SCIM sync status API); customer IT applies configuration changes on the IdP side.

### 8.3 Disaster Recovery

| Control | FORM | Shared | Customer |
|---|---|---|---|
| Database backup | ✅ Supabase Point-in-Time Recovery (PITR) with 7-day restore window. RPO ≤ 1 hour. | | |
| Backup restore procedure | ✅ Documented in `docs/SOC2_READINESS.md §DR`. Tenant-isolated restore capability. | | |
| RTO for full service restoration | ✅ RTO target ≤ 4 hours for Supabase primary failure. Documented in DR runbook. | | |
| Annual DR drill | ✅ FORM conducts an annual disaster recovery drill and documents results. | | |
| Customer data export before tenant offboarding | | ✅ FORM provides a data egress API and bulk export prior to tenant offboarding. Customer is responsible for initiating export before the offboarding deadline and retaining the data in their own storage. | |

---

## 9. Compliance and Regulatory Obligations

### 9.1 FORM's Compliance Obligations

| Compliance area | FORM's obligation | Evidence |
|---|---|---|
| SOC 2 Type II | Target completion 12 months post-commercial launch. Continuous monitoring via Drata. | SOC 2 report shared under NDA with all active enterprise customers on completion. In the interim: pen-test summary + Drata dashboard access available. |
| GDPR (as processor) | DPA executed before data flows. Sub-processor list maintained. DSAR tooling provided. SCC included in DPA for EU-US transfers. | `docs/GDPR_DPIA.md`, `docs/SUBPROCESSORS.md`, DPA in `docs/MSA_TEMPLATE.md §DPA`. |
| Data retention enforcement | Automated retention jobs per `docs/DATA_MODEL.md §12`. | Retention schedule in DPA. |
| Penetration testing | Annual external pen test. Critical findings remediated within 30 days. | Summary shared under NDA. Full report available under NDA + MSA. |
| Cryptographic controls | FIPS-aligned cipher suites. Key rotation policy. | `docs/CRYPTOGRAPHY_POLICY.md`, `docs/KEY_ROTATION.md`. |
| Security awareness training | All FORM personnel complete annual training. | Internal compliance record (`compliance/cc1/security-training-log.md`). |

### 9.2 Customer's Compliance Obligations

FORM does not assess, verify, or warrant that using FORM satisfies any industry-specific compliance requirement. The following are the customer's obligations:

| Obligation | Guidance |
|---|---|
| **HIPAA** (if applicable) | FORM is not a Covered Entity and does not accept HIPAA BAAs. Enterprise customers in US healthcare must evaluate whether employee wellness data constitutes PHI under their use case. FORM's privacy floor (§6.4) minimises HIPAA exposure but does not eliminate it. Customers with a potential HIPAA obligation must consult their legal counsel. |
| **ISO 27001 supplier assessment** | Customers conducting ISO 27001 supplier assessments may request FORM's ISMS documentation summary, pen-test executive summary, and SOC 2 bridge letter under NDA. |
| **FedRAMP / StateRAMP** | FORM does not hold and does not plan to pursue FedRAMP authorisation. Government sector customers requiring FedRAMP-authorised products cannot use FORM at this time. |
| **Employee consent programmes** | Where the customer's legal jurisdiction or employment law requires employee consent before collecting health or wellness data, the customer is responsible for obtaining, recording, and honouring that consent. FORM provides the technical consent-at-onboarding flow in the app; whether this flow is legally sufficient depends on jurisdiction and programme design. |
| **Works council / union consultation** (EU) | In EU jurisdictions where employee health monitoring requires works council approval, the customer is responsible for obtaining that approval before deploying FORM. |
| **Industry-specific data residency requirements** | FORM offers EU and US regions. Customers in jurisdictions requiring data residency to a specific country not served by FORM's current regions must disclose this requirement before signing. Country-level residency (e.g., Germany-only) is not currently available. |

### 9.3 Data Subject Access Requests (DSARs)

| Step | FORM (Processor) | Customer (Controller) |
|---|---|---|
| Receiving the DSAR | | ✅ Customer receives the DSAR from the employee (data subject). |
| Providing the data export | ✅ FORM provides a tenant-admin API endpoint to export all data FORM holds for a given user: `GET /v1/users/{user_id}/export`. JSON format, encrypted at rest, available to `tenant_owner`. | |
| Fulfilling the DSAR to the data subject | | ✅ Customer combines the FORM export with data from other systems and delivers it to the data subject within GDPR's 30-day window. |
| Deletion (Right to Erasure, Art. 17) | ✅ FORM provides `DELETE /v1/users/{user_id}` which triggers the erasure cascade defined in `docs/DATA_MODEL.md §12`. Soft delete immediately; hard delete within 30 days. Audit log entries are anonymised (not deleted) to preserve HMAC chain integrity. | ✅ Customer initiates the deletion API call or uses the admin dashboard delete button. |
| Rectification (Art. 16) | ✅ FORM provides update endpoints for correctable user attributes. | ✅ Customer initiates rectification via admin API or contacts FORM support. |
| Response time SLA | ✅ FORM responds to processor-side DSAR requests (data export, deletion confirmation) within 5 business days. | ✅ Customer is responsible for the 30-day response SLA to the data subject. |

---

## 10. Enterprise Onboarding and Offboarding

### 10.1 Onboarding Responsibilities

| Step | FORM | Customer |
|---|---|---|
| DPA and MSA execution | ✅ FORM prepares DPA and MSA (`docs/MSA_TEMPLATE.md`). | ✅ Customer legal executes. DPA must be signed before any pilot data enters FORM. |
| Tenant provisioning (slug, region, plan) | ✅ FORM enterprise-architect provisions the tenant in production. | |
| SSO / SCIM configuration | ✅ FORM provides step-by-step guides per IdP (`docs/SSO_CLIENT_CONFIG.md`). FORM CSM is available for live support. | ✅ Customer IT implements the IdP configuration and tests SSO in the staging environment before pilot launch. |
| Role mapping | ✅ FORM provides the group-to-role mapping interface. | ✅ Customer IT defines and implements the mapping. |
| White-label branding | ✅ FORM provides the branding configuration panel (logo, primary colour, custom domain). | ✅ Customer provides brand assets (SVG logo, hex colour) and configures CNAME for custom domain. DNS ownership stays with customer. |
| Pilot user list | | ✅ Customer HR/IT provides the pilot user list (up to 250 users for a standard free pilot). |
| Internal employee communication | ✅ FORM provides communication templates and guidance. | ✅ Customer HR/People team sends internal launch communications. FORM cannot communicate with employees directly without customer approval. |
| Success criteria definition | ✅ FORM CSM facilitates. Template: `docs/ENTERPRISE_ONBOARDING.md §success-criteria`. | ✅ Customer HR/People lead signs off on success criteria before pilot launch. |

### 10.2 Offboarding Responsibilities

When an enterprise customer terminates their contract, or when an individual employee leaves the organisation:

**Tenant offboarding (full contract termination):**

| Step | FORM | Customer |
|---|---|---|
| Data export | ✅ FORM holds data available for export for 30 days after contract end date. | ✅ Customer must initiate bulk data export before the 30-day window closes. Schema: `docs/DATA_MODEL.md §25`. |
| Data deletion | ✅ FORM executes hard delete of all tenant data within 30 days of the export window close. Written confirmation provided. | |
| Audit log retention | ✅ Audit log entries for the tenant are retained for the regulatory retention period (7 years for financial and admin events) even after tenant deletion, in anonymised form. | |
| SSO / SCIM decommissioning | | ✅ Customer IT decommissions the FORM application from their IdP after offboarding to prevent residual access. |

**Employee offboarding (individual):**

| Step | FORM | Customer |
|---|---|---|
| Session revocation | ✅ Immediate on SCIM deactivation signal. | ✅ Customer IT must deprovision the user in the IdP and trigger SCIM sync. |
| Data anonymisation | ✅ Employee's data is anonymised on request (Settings → Privacy → Leave organisation) or on tenant admin deletion. Raw PII is removed; aggregate data derived from that employee's historical activity is retained for tenant reporting. | |
| Data retention (post-offboarding) | ✅ Per retention schedule in DPA. Default: 90 days post-deactivation for personal workout data; 7 years for billing and audit records. | ✅ Customer may request accelerated deletion under Art. 17 if there is a legitimate basis. |

---

## 11. Liability Boundaries and Escalation Path

### 11.1 When Something Falls in a Gap

If an issue arises that is not clearly attributed to FORM or the customer by this document, the default escalation path is:

1. **Customer security team or IT admin** raises the issue with their **FORM CSM** (Growth/Enterprise plan) or **support@form.coach** (Starter plan).
2. FORM's **enterprise-architect** and **security-engineer** jointly assess whether the control gap is on the FORM side, the customer side, or genuinely shared.
3. A written gap assessment is provided within 5 business days.
4. If the gap is on FORM's side: FORM files an internal issue and adds it to the remediation backlog with a target date.
5. If the gap is on the customer's side: FORM provides a configuration guide and CSM support.
6. If genuinely shared: both parties agree on a remediation plan.

### 11.2 Controls FORM Cannot Provide

The following are explicitly outside FORM's scope, regardless of contract tier:

| Out-of-scope control | Reason |
|---|---|
| MFA enforcement on the SSO path | Controlled by the customer's IdP. FORM cannot force-enable MFA on a third-party identity provider. |
| Physical device security for employee devices | FORM is a mobile app on employee-owned or employer-managed devices. Physical security is outside FORM's perimeter. |
| Preventing insider threat from within the customer's IdP admin team | A compromised IdP admin can provision rogue users or modify group memberships. FORM mitigates this by requiring FORM enterprise-architect approval for `tenant_owner` changes, but cannot fully prevent an IdP-level compromise. |
| Monitoring of employee use of the FORM app (content of coaching sessions) | FORM's privacy floor prohibits employer access to individual user data. This is a product constraint, not a technical gap. |
| HIPAA BAA | Not available at this time. |
| Country-level data residency below EU / US region | Not available at this time. |

---

## 12. Non-Negotiable Floors

The following customer requests are declined regardless of contract value or legal pressure. These protections exist for employees (FORM's end users) and cannot be waived.

| Request | Response |
|---|---|
| Grant HR access to individual employee health, body composition, or coaching data | Declined. Privacy floor enforced at application + database layer. Not configurable. |
| Share workout performance or health data for insurance premium calculation | Declined per ENTERPRISE.md §When we say no. |
| Enable real-time monitoring of individual employee fitness activity | Declined. Admin dashboard shows aggregates only, with a minimum cohort size of 10. |
| Provide a backdoor or law enforcement access without valid legal process | Declined. FORM follows the government request policy (`compliance/p1/gov-request-policy.md`): legal-review-first, narrowest possible response, user notification where legally permitted. |
| Tie wellness participation to performance reviews, insurance, or employment consequences | Declined. Employee participation data is never surfaced to managers in any form. |
| Custom feature that weakens the HMAC audit chain (DEC-030) | Declined. Audit chain integrity is a SOC 2 Type II control. |

---

## 13. Glossary

| Term | Definition |
|---|---|
| **Processor** | FORM's legal role under GDPR. Processes personal data on behalf of the controller. |
| **Controller** | The enterprise customer. Determines the purpose and means of processing. |
| **DPA** | Data Processing Addendum. Contract between controller and processor per GDPR Art. 28. |
| **SCIM** | System for Cross-domain Identity Management. Protocol for automated user provisioning. |
| **RLS** | Row-Level Security. Postgres feature that enforces tenant data isolation at the database layer. |
| **JIT** | Just-In-Time provisioning. User record created in FORM on first successful SSO login. |
| **HMAC chain** | Hash-based Message Authentication Code chain. Provides tamper-evident audit log integrity (DEC-030). |
| **Break-glass** | Emergency access procedure for FORM personnel to access a tenant's data with 2-person approval and time-bounded scope. |
| **CSM** | Customer Success Manager. Named FORM contact for Growth and Enterprise plan customers. |
| **DSAR** | Data Subject Access Request. Formal request by an individual to receive or delete their personal data. |
| **SLA** | Service Level Agreement. Uptime and response time commitments in `docs/ENTERPRISE_SLA.md`. |
| **Pilot** | Evaluation period (90 or 180 days) before a commercial contract. Governed by `enterprise_contracts.pilot_start_at / pilot_end_at`. DPA is signed before pilot data flows. |
| **Privacy floor** | Non-negotiable data access boundaries protecting employees from employer surveillance. Enforced at application and database layer. |
| **SOC 2 Type II** | Audit of security controls over a 12-month observation period by an independent CPA firm. |
| **Standard Contractual Clauses (SCCs)** | EU-approved contractual mechanism for transferring personal data from the EU to third countries. Included in FORM's DPA. |

---

## 14. Version History

| Version | Date | Author | Changes |
|---|---|---|---|
| v1.0 | 2026-06-12 | enterprise-architect + compliance-officer + security-engineer | Initial publication. Covers infrastructure, application, identity, data, audit, operations, compliance, onboarding, offboarding, and liability boundaries. References DATA_MODEL.md v1.9, SSO_SCIM_IMPLEMENTATION.md, INCIDENT_RESPONSE.md, SOC2_READINESS.md, ENTERPRISE.md, ENTERPRISE_SLA.md. |

---

*FORM · enterprise-architect + compliance-officer + security-engineer · v1.0 · 2026-06-12*
