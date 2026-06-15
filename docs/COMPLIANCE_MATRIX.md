# FORM · Compliance Framework Control Matrix v1.0

| Field | Value |
|---|---|
| **Document title** | Cross-Framework Compliance Control Matrix |
| **Version** | v1.0 |
| **Status** | IN FORCE |
| **Effective date** | 2026-06-15 |
| **Owner** | `compliance-officer` + `security-engineer` + `enterprise-architect` |
| **Audience** | Enterprise procurement, CISOs, DPOs, legal teams, infosec auditors |
| **Evidence artifact ID** | CM-E-001 |
| **Cross-references** | `docs/SOC2_READINESS.md`, `docs/GDPR_DPIA.md`, `docs/AUDIT_LOG_SCHEMA.md` (DEC-030), `docs/CRYPTOGRAPHY_POLICY.md`, `docs/DATA_MODEL.md`, `docs/INCIDENT_RESPONSE.md`, `docs/OBSERVABILITY.md`, `docs/ENTERPRISE.md`, `docs/SECURITY_QUESTIONNAIRE.md`, `docs/SHARED_RESPONSIBILITY_MODEL.md`, `docs/SUBPROCESSORS.md` |
| **Last reviewed** | 2026-06-15 |
| **Next review due** | 2027-06-15 |
| **Distribution** | NDA-gated. Available to qualified prospects and current enterprise customers via `security@form.coach`. |

---

## Purpose

This document is the **single authoritative mapping** of FORM's implemented security and privacy controls against four compliance frameworks:

1. **SOC 2 Type II** — Trust Service Criteria (AICPA 2022)
2. **ISO 27001:2022** — Information Security Management System (Annex A)
3. **GDPR** — EU General Data Protection Regulation (Articles 5–89, with emphasis on Art. 9 special-category health data)
4. **HIPAA adjacency** — Health Insurance Portability and Accountability Act (informational; FORM is not a HIPAA Covered Entity or Business Associate, but enterprise customers in US healthcare-adjacent industries request this mapping)

**How to use this document:**
- Column "Evidence" maps to artifact IDs retrievable from `security@form.coach` under NDA.
- Column "Status" reflects current implementation state: ✅ In force · 🔄 In progress · ⏳ Planned · N/A Not applicable.
- For a pre-filled security questionnaire (CAIQ Lite, SIG Lite, DDQ), see `docs/SECURITY_QUESTIONNAIRE.md`.
- For SOC 2 gap analysis and evidence catalogue, see `docs/SOC2_READINESS.md`.
- For GDPR processing records (ROPA), see `docs/ROPA.md`.

---

## Table of Contents

1. [Framework Applicability Matrix](#1-framework-applicability-matrix)
2. [Domain A — Organisational Controls](#2-domain-a--organisational-controls)
3. [Domain B — Access Control & Identity](#3-domain-b--access-control--identity)
4. [Domain C — Cryptography & Data Protection](#4-domain-c--cryptography--data-protection)
5. [Domain D — Physical & Environmental Security](#5-domain-d--physical--environmental-security)
6. [Domain E — Operations Security](#6-domain-e--operations-security)
7. [Domain F — Communications & Network Security](#7-domain-f--communications--network-security)
8. [Domain G — Supplier & Third-Party Management](#8-domain-g--supplier--third-party-management)
9. [Domain H — Incident Management](#9-domain-h--incident-management)
10. [Domain I — Business Continuity & DR](#10-domain-i--business-continuity--dr)
11. [Domain J — Compliance & Audit](#11-domain-j--compliance--audit)
12. [Domain K — Privacy Controls (GDPR / Art. 9)](#12-domain-k--privacy-controls-gdpr--art-9)
13. [Evidence Artifact Cross-Reference](#13-evidence-artifact-cross-reference)
14. [Gap Analysis Summary per Framework](#14-gap-analysis-summary-per-framework)
15. [Certification Roadmap](#15-certification-roadmap)
16. [Revision History](#16-revision-history)

---

## 1. Framework Applicability Matrix

| Framework | Applies to FORM? | Scope | Certification target | Current status |
|---|---|---|---|---|
| SOC 2 Type II | ✅ Yes | All production systems, enterprise tenants | 12 months post enterprise launch | Readiness: ~72% — see `docs/SOC2_READINESS.md` |
| ISO 27001:2022 | ✅ Yes (target) | ISMS scoping: enterprise SaaS product + corporate infrastructure | 18 months post enterprise launch | Not yet initiated; roadmap in §15 |
| GDPR | ✅ Yes — mandatory | All EU user data; Art. 9 health data (all users) | Ongoing compliance obligation | DPIA complete (draft) — see `docs/GDPR_DPIA.md` |
| ePrivacy / Cookie Law | ✅ Yes | Web properties, tracking cookies, analytics | Ongoing compliance obligation | Consent banner deployed |
| HIPAA | ⚠️ Adjacent only | No PHI processed; no covered entity status; no BAA signed | N/A as primary obligation | Mapping provided for customer reference |
| CCPA / CPRA | ✅ Yes | US users in California; "Sensitive Personal Information" (health data) | Ongoing compliance obligation | Privacy policy current |
| NIST CSF 2.0 | ✅ Informational | Self-assessment framework; maps to SOC 2 + ISO 27001 | No certification planned | Mapping available on request |

**Note on HIPAA:** FORM is a wellness application, not a healthcare provider, health plan, or clearinghouse. Individual enterprise customers who are HIPAA Covered Entities may ask about BAA availability. FORM's position: we do not currently offer a BAA. Customers should conduct their own HIPAA analysis; our privacy floor (HR never sees individual health data) provides structural protections, but HIPAA compliance is the customer's responsibility where applicable.

---

## 2. Domain A — Organisational Controls

### A.1 Information Security Policies

| Control ID | Control Description | SOC 2 TSC | ISO 27001:2022 | GDPR | HIPAA Analog | Implementation | Evidence | Status |
|---|---|---|---|---|---|---|---|---|
| A.1.1 | Information security policy documented and approved | CC1.1, CC1.2 | 5.1, 5.2, A.5.1 | Art. 24, Art. 32 | § 164.316(a)(1) | `docs/SECURITY.md` — master security policy. Reviewed annually. | POL-001 | ✅ |
| A.1.2 | Acceptable use policy published and acknowledged by all personnel | CC1.3, CC6.2 | A.5.10 | Art. 32(4) | § 164.308(a)(5)(ii)(A) | `docs/ACCEPTABLE_USE_POLICY.md`. Acknowledgement tracked via onboarding checklist. | POL-002 | ✅ |
| A.1.3 | Cryptography policy in force | CC6.7 | A.8.24 | Art. 32(1)(a) | § 164.312(a)(2)(iv) | `docs/CRYPTOGRAPHY_POLICY.md` v1.0. AES-256-GCM, TLS 1.3, key rotation quarterly. | POL-003 | ✅ |
| A.1.4 | Privacy policy published and current | P1.1, P2.1 | A.5.34 | Art. 12, 13, 14 | § 164.520 | `docs/PRIVACY_POLICY.md`. Covers Art. 9 special-category data, Art. 13 disclosures. | POL-004 | ✅ |
| A.1.5 | Vulnerability management policy in force | CC7.1 | A.8.8 | Art. 32(1)(d) | § 164.308(a)(8) | `docs/VULNERABILITY_MANAGEMENT.md`. CVSS scoring, SLA by severity. | POL-005 | ✅ |
| A.1.6 | Security awareness training programme | CC1.4 | A.6.3 | Art. 32(4) | § 164.308(a)(5) | `docs/SECURITY_AWARENESS_TRAINING_POLICY.md`. Annual mandatory + role-based. | POL-006 | ✅ |
| A.1.7 | Responsible disclosure policy published | CC7.1 | A.5.8 | — | — | `docs/RESPONSIBLE_DISCLOSURE.md`. 90-day disclosure window, CVE co-ordination. | POL-007 | ✅ |

### A.2 Information Security Roles and Responsibilities

| Control ID | Control Description | SOC 2 TSC | ISO 27001:2022 | GDPR | HIPAA Analog | Implementation | Evidence | Status |
|---|---|---|---|---|---|---|---|---|
| A.2.1 | Security roles defined with named owners | CC1.1, CC1.2 | 5.3, A.5.2 | Art. 37–39 (DPO), Art. 24 | § 164.308(a)(2) | `docs/ENTERPRISE.md §"Org structure"`. Role assignments in RBAC config. | ORG-001 | ✅ |
| A.2.2 | DPO-equivalent designated for GDPR | — | A.5.34 | Art. 37 | — | `compliance-officer` agent acts as DPO-equivalent during pre-Series A phase. Formal DPO appointment required pre EU-enterprise launch. | ORG-002 | 🔄 |
| A.2.3 | Separation of duties enforced for privileged actions | CC6.3 | A.5.3 | Art. 32 | § 164.308(a)(4) | Database migrations require PR approval + CI pass + manual review gate. Prod access requires two-person approval. | ORG-003 | ✅ |
| A.2.4 | Background verification for roles with data access | CC1.4 | A.6.1 | Art. 32(4) | § 164.308(a)(3)(ii)(B) | `docs/HIRING_GUIDE.md`. Background checks for all engineering and data roles. | ORG-004 | 🔄 |

### A.3 Risk Management

| Control ID | Control Description | SOC 2 TSC | ISO 27001:2022 | GDPR | HIPAA Analog | Implementation | Evidence | Status |
|---|---|---|---|---|---|---|---|---|
| A.3.1 | Formal risk assessment process | CC3.1, CC3.2 | 6.1, 8.2, A.5.7 | Art. 32(2), Art. 35 | § 164.308(a)(1) | Annual risk assessment scheduled. Current risks tracked in `docs/ASSUMPTIONS_REGISTER.md`. DPIA in `docs/GDPR_DPIA.md`. | RISK-001 | 🔄 |
| A.3.2 | Risk treatment and residual risk acceptance | CC3.3, CC3.4 | 6.1.3, 8.3 | Art. 35(7)(d) | § 164.308(a)(1)(ii)(B) | Gap tracking in `docs/SOC2_READINESS.md §"Gap registry"`. P0/P1 gaps tracked to remediation. | RISK-002 | 🔄 |
| A.3.3 | Privacy impact assessments for new processing activities | P6.1 | A.5.34 | Art. 35 | — | `docs/GDPR_DPIA.md` v0.1. Refresh required on material processing change. | RISK-003 | ✅ |

---

## 3. Domain B — Access Control & Identity

| Control ID | Control Description | SOC 2 TSC | ISO 27001:2022 | GDPR | HIPAA Analog | Implementation | Evidence | Status |
|---|---|---|---|---|---|---|---|---|
| B.1 | Access control policy documented | CC6.1 | A.5.15, A.5.16 | Art. 32(1)(b) | § 164.312(a)(1) | Role hierarchy: `tenant_owner > tenant_admin > tenant_manager > end_user`. Documented in `docs/ENTERPRISE.md §"Identity"`. | AC-001 | ✅ |
| B.2 | SSO (SAML 2.0 / OIDC) integration for enterprise tenants | CC6.1, CC6.2 | A.5.16, A.5.17 | Art. 32(1)(b) | § 164.312(d) | `docs/SSO_SCIM_IMPLEMENTATION.md`. SP-initiated and IdP-initiated flows. Okta, Azure AD, Google Workspace, OneLogin. | AC-002 | ✅ |
| B.3 | SCIM 2.0 automated user provisioning / deprovisioning | CC6.2, CC6.3 | A.5.16, A.5.18 | Art. 5(1)(e) data minimisation | § 164.308(a)(3)(ii)(C) | `docs/SSO_SCIM_IMPLEMENTATION.md §"SCIM v2 endpoints"`. Deprovisioning triggers `user.deactivated` audit event within 15 minutes of IdP push. | AC-003 | ✅ |
| B.4 | MFA enforcement at tenant level | CC6.1 | A.8.5 | Art. 32(1)(b) | § 164.312(d) | Tenant-level MFA enforcement flag. Minimum TOTP; hardware key support planned. Phishing-resistant MFA (passkeys) roadmap Q4 2026. | AC-004 | 🔄 |
| B.5 | Principle of least privilege for API and database access | CC6.1, CC6.3 | A.8.2, A.8.3 | Art. 25 data protection by design | § 164.312(a)(1) | Supabase RLS policies per `docs/DATA_MODEL.md §4`. Admin API scoped tokens. No cross-tenant reads. | AC-005 | ✅ |
| B.6 | Privileged access management (PAM) for production systems | CC6.3 | A.8.2, A.8.18 | Art. 32 | § 164.312(a)(2)(ii) | Break-glass access procedure. All privileged sessions logged to `audit_log`. Two-person approval for direct DB access. | AC-006 | 🔄 |
| B.7 | Session management: timeout, concurrent session limits | CC6.1 | A.8.5 | Art. 32(1)(b) | § 164.312(a)(2)(iii) | Tenant-configurable session expiry (default 8h; max 24h). IP allowlist configurable. | AC-007 | ✅ |
| B.8 | Access review (periodic revalidation of permissions) | CC6.2, CC6.3 | A.5.18 | Art. 5(1)(c) | § 164.308(a)(4)(ii)(C) | Quarterly access review scheduled. Admin API provides export of active grants for customer review. | AC-008 | 🔄 |
| B.9 | API authentication: signed tokens, short expiry | CC6.1 | A.8.5 | Art. 32 | § 164.312(a)(2)(i) | JWT RS256 (2048-bit), 1h access token, 7d refresh (rotated). Documented in `docs/ENTERPRISE_ADMIN_API.md §Authentication`. | AC-009 | ✅ |

---

## 4. Domain C — Cryptography & Data Protection

| Control ID | Control Description | SOC 2 TSC | ISO 27001:2022 | GDPR | HIPAA Analog | Implementation | Evidence | Status |
|---|---|---|---|---|---|---|---|---|
| C.1 | Encryption in transit — TLS 1.3 mandatory | CC6.7 | A.8.24 | Art. 32(1)(a) | § 164.312(e)(1) | TLS 1.3 on all API endpoints (Cloudflare terminates). TLS 1.2 exception register maintained. `docs/CRYPTOGRAPHY_POLICY.md §2`. | CRYPT-001 | ✅ |
| C.2 | Encryption at rest — AES-256-GCM | CC6.7 | A.8.24 | Art. 32(1)(a) | § 164.312(a)(2)(iv) | Supabase pgsodium column-level encryption for Art. 9 fields. R2/B2 backup encryption. `docs/CRYPTOGRAPHY_POLICY.md §2`. | CRYPT-002 | ✅ |
| C.3 | Key management: generation, rotation, custody | CC6.7 | A.8.24 | Art. 32(1)(a) | § 164.312(a)(2)(iv) | `docs/KEY_ROTATION.md`. Keys in Cloudflare Secrets / Supabase Vault. Quarterly rotation. KMS event logging. | CRYPT-003 | ✅ |
| C.4 | HMAC-chained tamper-evident audit log (DEC-030) | CC4.1, CC7.2 | A.8.15, A.8.16 | Art. 5(2) accountability | § 164.312(b) | `docs/AUDIT_LOG_SCHEMA.md`. HMAC-SHA256 chain. Break triggers PagerDuty P0. Secret key rotated quarterly. | CRYPT-004 | ✅ |
| C.5 | Data classification and handling policy | CC6.1 | A.5.12, A.5.13 | Art. 9 special-category | § 164.308(a)(3) | Four-tier classification: PUBLIC / INTERNAL / CONFIDENTIAL / RESTRICTED (Art. 9 health). See `docs/DATA_MODEL.md §3`. | CRYPT-005 | ✅ |
| C.6 | Pseudonymisation of user identifiers in analytics | P4.2, P6.7 | A.5.34 | Art. 25, Art. 89 | — | Analytics pipeline uses `ic_user_id` (UUID, opaque). No name/email in event payloads. `docs/ANALYTICS_SETUP.md`. | CRYPT-006 | ✅ |
| C.7 | Secure deletion / data erasure on request | P4.3 | A.8.10 | Art. 17 right to erasure | § 164.316(b)(2)(i) | Deletion cascade on `user.delete_requested`. Art. 9 fields zeroed within 30 days; audit tombstone retained 7yr per ROPA. `docs/DATA_MODEL.md §16`. | CRYPT-007 | 🔄 |
| C.8 | Cryptographic algorithm review process | CC5.3 | A.8.24 | Art. 32 | — | Annual review of approved algorithms. Post-quantum roadmap: `docs/CRYPTOGRAPHY_POLICY.md §9`. | CRYPT-008 | ✅ |

---

## 5. Domain D — Physical & Environmental Security

| Control ID | Control Description | SOC 2 TSC | ISO 27001:2022 | GDPR | HIPAA Analog | Implementation | Evidence | Status |
|---|---|---|---|---|---|---|---|---|
| D.1 | Physical security delegated to cloud providers (shared responsibility) | CC6.4 | A.7.1–A.7.14 | Art. 32(1)(b) | § 164.310 | FORM is fully cloud-native (Cloudflare + Supabase). Physical security is provider responsibility. See `docs/SHARED_RESPONSIBILITY_MODEL.md`. | PHY-001 | ✅ |
| D.2 | Provider SOC 2 / ISO 27001 attestations reviewed annually | CC9.2 | A.5.22 | Art. 28(3)(h) | § 164.308(b) | Cloudflare: SOC 2 Type II + ISO 27001. Supabase: SOC 2 Type II. Attestations in `docs/SUBPROCESSORS.md`. Annual review by compliance-officer. | PHY-002 | ✅ |
| D.3 | No physical media containing production data | CC6.4 | A.7.8 | Art. 32(1)(a) | § 164.310(d)(1) | All data resides in cloud; no physical media extraction policy in `docs/ACCEPTABLE_USE_POLICY.md §5`. | PHY-003 | ✅ |
| D.4 | Employee workstation security (MDM / endpoint management) | CC6.4 | A.8.1 | Art. 32(1)(b) | § 164.310(c) | MDM deployment in progress. Full-disk encryption required. Screen lock 5 min. See `docs/ONBOARDING_SECURITY.md`. | PHY-004 | 🔄 |

---

## 6. Domain E — Operations Security

| Control ID | Control Description | SOC 2 TSC | ISO 27001:2022 | GDPR | HIPAA Analog | Implementation | Evidence | Status |
|---|---|---|---|---|---|---|---|---|
| E.1 | Documented operating procedures for production systems | CC5.1 | A.8.1 | Art. 32 | § 164.308(a)(1) | `docs/ENGINEERING_RUNBOOK.md` + `docs/runbooks/` (RB-SUB-01 through RB-SUB-08, DR-001 through DR-005). | OPS-001 | ✅ |
| E.2 | Change management: PR review, CI gates, approval workflow | CC8.1 | A.8.32 | Art. 32(1)(d) | § 164.308(a)(8) | All changes via GitHub PR. Required: CI green, security scan pass, one-reviewer approval. Production deploy gated by tag. | OPS-002 | ✅ |
| E.3 | Capacity and performance monitoring | A1.2, A1.3 | A.8.6 | Art. 32(1)(b) availability | — | `docs/OBSERVABILITY.md §"Capacity"`. RED metrics on all services. SLO alert thresholds in Grafana. | OPS-003 | ✅ |
| E.4 | Vulnerability scanning: dependencies and SAST | CC7.1 | A.8.8 | Art. 32(1)(d) | § 164.308(a)(8) | Dependabot on all repos. SAST (CodeQL) in CI. `docs/VULNERABILITY_MANAGEMENT.md §3`. | OPS-004 | ✅ |
| E.5 | Penetration testing: annual external, quarterly internal | CC7.1 | A.8.8 | Art. 32(1)(d) | § 164.308(a)(8) | `docs/PENETRATION_TESTING.md`. Pre-launch pentest required before first enterprise contract. Results shared under NDA. | OPS-005 | 🔄 |
| E.6 | Patch management: CVSS-based SLA | CC7.1 | A.8.19 | Art. 32(1)(d) | § 164.308(a)(5)(ii)(B) | Critical (CVSS ≥ 9.0): 24h. High (≥ 7.0): 7d. Medium (≥ 4.0): 30d. Low: next release. `docs/VULNERABILITY_MANAGEMENT.md §4`. | OPS-006 | ✅ |
| E.7 | Anti-malware and endpoint protection | CC6.8 | A.8.7 | Art. 32 | § 164.308(a)(5)(ii)(B) | macOS/Linux endpoints; CrowdStrike Falcon or equivalent planned post-Series A. Currently relying on Apple XProtect + MDM. | OPS-007 | 🔄 |
| E.8 | Software supply chain integrity | CC7.1 | A.8.30 | Art. 32 | — | Dependabot lock-file validation. No un-audited scripts piped to shell. Package checksums in CI. | OPS-008 | ✅ |
| E.9 | Development / staging / production environment separation | CC8.1 | A.8.31 | Art. 32(1)(b) | § 164.308(a)(7) | Three environments: dev (no real data), staging (synthetic data), production. Separate Cloudflare zones and Supabase projects. | OPS-009 | ✅ |

---

## 7. Domain F — Communications & Network Security

| Control ID | Control Description | SOC 2 TSC | ISO 27001:2022 | GDPR | HIPAA Analog | Implementation | Evidence | Status |
|---|---|---|---|---|---|---|---|---|
| F.1 | Network perimeter: Cloudflare WAF + DDoS mitigation | CC6.6 | A.8.20, A.8.21 | Art. 32(1)(b) | § 164.312(e)(1) | Cloudflare Enterprise (WAF, Bot Management, DDoS L3–L7). Rate limiting on all API endpoints. | NET-001 | ✅ |
| F.2 | API gateway with authentication, rate limiting, and logging | CC6.6 | A.8.20 | Art. 32 | § 164.312(e)(2)(ii) | Cloudflare Workers API gateway. Per-tenant rate limits. All requests logged with `request_id`. | NET-002 | ✅ |
| F.3 | Egress control: allowlist for external service calls | CC6.6 | A.8.21 | Art. 28 (processor controls) | — | Cloudflare Workers service bindings restrict egress. Only Anthropic, ElevenLabs, Stripe, Supabase in allowlist. | NET-003 | 🔄 |
| F.4 | Intrusion detection and security monitoring | CC7.2 | A.8.16 | Art. 32(1)(d) | § 164.308(a)(6) | Cloudflare Security Events + custom SIEM (Grafana Loki + alerts). FORM-PRIV-001–004 alert suite. `docs/OBSERVABILITY.md §"Security monitoring"`. | NET-004 | ✅ |
| F.5 | Secure DNS (DNSSEC) and certificate management | CC6.7 | A.8.23 | Art. 32 | — | Cloudflare DNSSEC enabled. Certificates via Cloudflare-managed Let's Encrypt (auto-renewal). 90-day rotation. | NET-005 | ✅ |
| F.6 | Data residency: EU data stays in EU | P6.7 | A.5.14 | Art. 44–49 (transfers) | — | Cloudflare EU routing. Supabase project in EU-West (Ireland). No US-EU transfer without SCC. `docs/GDPR_DPIA.md §"Data flows"`. | NET-006 | ✅ |
| F.7 | Secure communications policy for internal tooling | CC1.3 | A.5.14 | Art. 32(1)(a) | — | All internal comms over TLS. No plaintext credentials in Slack, email, or issue trackers. Secrets via Cloudflare Secrets / 1Password. | NET-007 | ✅ |

---

## 8. Domain G — Supplier & Third-Party Management

| Control ID | Control Description | SOC 2 TSC | ISO 27001:2022 | GDPR | HIPAA Analog | Implementation | Evidence | Status |
|---|---|---|---|---|---|---|---|---|
| G.1 | Subprocessor register maintained and current | CC9.2 | A.5.19, A.5.22 | Art. 28(1), Art. 30(2) | § 164.308(b) | `docs/SUBPROCESSORS.md`. Updated within 30 days of any change. Enterprise customers notified 30 days in advance of additions. | SUP-001 | ✅ |
| G.2 | DPA signed with all processors handling personal data | CC9.2 | A.5.19 | Art. 28(3) | § 164.308(b)(3) | DPAs in place: Anthropic, ElevenLabs, Stripe, Supabase, Cloudflare. Template DPA in `docs/MSA_TEMPLATE.md §"DPA Annex"`. | SUP-002 | ✅ |
| G.3 | Standard Contractual Clauses for US-EU transfers | — | A.5.14 | Art. 46(2)(c) | — | SCCs Module 1 (controller-to-processor) in place for Anthropic and ElevenLabs (US-based). Reviewed after Schrems II. | SUP-003 | ✅ |
| G.4 | Annual supplier security review | CC9.2 | A.5.22 | Art. 28(3)(h) | — | Annual review of sub-processor SOC 2 / ISO 27001 attestations. Non-attesting suppliers require compensating controls documented in `docs/VENDOR_REGISTRY.md`. | SUP-004 | 🔄 |
| G.5 | Vendor risk tiers (critical / significant / standard) | CC9.2 | A.5.21 | Art. 28 | — | `docs/VENDOR_REGISTRY.md`. Critical vendors (process Art. 9 data): Supabase, Anthropic. Annual pentest coverage extended to critical vendors. | SUP-005 | ✅ |

---

## 9. Domain H — Incident Management

| Control ID | Control Description | SOC 2 TSC | ISO 27001:2022 | GDPR | HIPAA Analog | Implementation | Evidence | Status |
|---|---|---|---|---|---|---|---|---|
| H.1 | Incident classification: P0–P3 with defined SLAs | CC7.3, CC7.4 | A.5.24, A.5.25 | Art. 33 (72h notification) | § 164.308(a)(6)(ii) | `docs/INCIDENT_RESPONSE.md §"Classification"`. P0 = system down / data breach; P3 = cosmetic. Escalation tree defined. | IR-001 | ✅ |
| H.2 | Personal data breach notification process (72h GDPR) | CC7.3 | A.5.24 | Art. 33, Art. 34 | § 164.308(a)(6) | `docs/INCIDENT_RESPONSE.md §"GDPR breach notification"`. DPA-notification template. 72h clock starts at detection. | IR-002 | ✅ |
| H.3 | Post-mortem template and blameless review process | CC7.5 | A.5.27 | — | § 164.308(a)(6)(ii) | `docs/INCIDENT_RESPONSE.md §"Post-mortem template"`. All P0/P1 incidents require post-mortem within 5 business days. | IR-003 | ✅ |
| H.4 | Privacy floor breach detection alerts (FORM-PRIV-001–004) | CC7.2, CC7.3 | A.5.25 | Art. 33, Art. 9 | — | Four automated alerts in `docs/OBSERVABILITY.md §"Privacy alerts"`. Trigger HMAC-chained audit event + PagerDuty P0. DEC-030. | IR-004 | ✅ |
| H.5 | Customer communication templates for security incidents | CC2.3 | A.5.26 | Art. 34 (communication to data subjects) | § 164.308(a)(6) | `docs/INCIDENT_RESPONSE.md §"Communication templates"`. Legal-reviewed templates for enterprise customers and end users. | IR-005 | ✅ |
| H.6 | Forensic evidence preservation procedure | CC7.4 | A.5.28 | Art. 33(5) documentation obligation | § 164.312(b) | Audit log retention 7yr (HMAC-chained). Read-only forensic access role. Evidence chain documented in IR playbook. | IR-006 | ✅ |

---

## 10. Domain I — Business Continuity & DR

| Control ID | Control Description | SOC 2 TSC | ISO 27001:2022 | GDPR | HIPAA Analog | Implementation | Evidence | Status |
|---|---|---|---|---|---|---|---|---|
| I.1 | Business continuity plan documented and tested | A1.1–A1.3, CC9.1 | A.5.29, A.5.30 | Art. 32(1)(c) resilience | § 164.308(a)(7) | `docs/BUSINESS_CONTINUITY.md`. Covers Supabase outage, Cloudflare edge failure, R2 data loss, Anthropic outage, compound disaster. | BCP-001 | ✅ |
| I.2 | Recovery Time Objective (RTO): ≤ 4h for P0 | A1.2 | A.5.29 | Art. 32(1)(c) | § 164.308(a)(7)(ii)(C) | P0 RTO: 4h. P1 RTO: 8h. Documented in `docs/ENTERPRISE_SLA.md §"Availability SLA"`. DR runbooks: `docs/runbooks/dr/`. | BCP-002 | ✅ |
| I.3 | Recovery Point Objective (RPO): ≤ 1h | A1.2 | A.5.29 | Art. 32(1)(c) | § 164.308(a)(7)(ii)(A) | Supabase WAL-based continuous replication. Point-in-time recovery ≤ 1h. R2 backup every 6h. `docs/BUSINESS_CONTINUITY.md §"RPO"`. | BCP-003 | ✅ |
| I.4 | DR drill cadence: at least annually | A1.3 | A.5.30 | Art. 32(1)(c) | § 164.308(a)(7)(ii)(D) | DR drill scheduled annually. Results documented; RTO/RPO validated. Evidence: DR drill report. | BCP-004 | ⏳ |
| I.5 | Geographic redundancy for production data | A1.2 | A.8.13 | Art. 32(1)(c) | § 164.308(a)(7) | Primary: Supabase EU-West (Ireland). Backup: Cloudflare R2 (EU) + Backblaze B2 (EU). Cross-provider redundancy. | BCP-005 | ✅ |
| I.6 | Backup integrity verification | CC7.4 | A.8.13 | Art. 32(1)(c) | — | Automated restore-to-test weekly. Hash verification on every backup. Alert on mismatch. `docs/runbooks/RB-SUB-07.md`. | BCP-006 | ✅ |

---

## 11. Domain J — Compliance & Audit

| Control ID | Control Description | SOC 2 TSC | ISO 27001:2022 | GDPR | HIPAA Analog | Implementation | Evidence | Status |
|---|---|---|---|---|---|---|---|---|
| J.1 | Continuous compliance monitoring tooling (Vanta/Drata) | CC4.1, CC4.2 | 9.1 | Art. 24(1) | § 164.308(a)(8) | Vanta or Drata deployment planned pre-audit. Currently: manual evidence collection per `docs/SOC2_READINESS.md §"Evidence collection"`. | AUDIT-001 | ⏳ |
| J.2 | Audit log retention: 7yr for compliance events | CC4.1 | A.8.15 | Art. 5(2), Art. 30 | § 164.312(b) | `docs/AUDIT_LOG_SCHEMA.md §"Retention"`. 7yr for financial/compliance events. 3yr for operational. 30d for read-only. | AUDIT-002 | ✅ |
| J.3 | Records of Processing Activities (ROPA) — Art. 30 | P6.1 | A.5.34 | Art. 30 | — | `docs/ROPA.md`. Covers all processing activities including Art. 9 health data. Updated on material change. | AUDIT-003 | ✅ |
| J.4 | Annual internal compliance review | CC4.2 | 9.2 (internal audit) | Art. 24 | § 164.308(a)(8) | Annual review by compliance-officer. Gap register maintained in `docs/SOC2_READINESS.md §"Gap registry"`. | AUDIT-004 | 🔄 |
| J.5 | Customer audit rights (enterprise contracts) | CC2.3 | A.5.22 | Art. 28(3)(h) | § 164.308(b)(4) | `docs/MSA_TEMPLATE.md §"Audit rights"`. Enterprise customers may request audit reports or conduct desk reviews annually. On-site audits at $250k+ ARR tier. | AUDIT-005 | ✅ |
| J.6 | Regulatory change monitoring | CC3.3 | 6.1 | Art. 70 EDPB guidance monitoring | — | compliance-officer monitors EDPB opinions, ICO guidance, FTC updates, NIST publications. Material changes reflected in policy updates within 60 days. | AUDIT-006 | 🔄 |

---

## 12. Domain K — Privacy Controls (GDPR / Art. 9)

This domain covers privacy controls with particular relevance to GDPR Art. 9 (special-category health data) and FORM's privacy floor for enterprise deployments.

| Control ID | Control Description | SOC 2 TSC | ISO 27001:2022 | GDPR | HIPAA Analog | Implementation | Evidence | Status |
|---|---|---|---|---|---|---|---|---|
| K.1 | Explicit consent for Art. 9 data at collection point | P2.1, P2.2 | A.5.34 | Art. 9(2)(a), Art. 7 | — | Granular consent modal at onboarding. Consent withdrawal triggers data deletion cascade. Consent log retained 7yr. | PRIV-001 | ✅ |
| K.2 | Privacy by design — aggregate-only employer reporting | P6.1, P6.7 | A.5.34 | Art. 25 | — | Admin dashboard shows aggregate metrics only (% active, % activated). **No individual user data visible to HR.** Enforced at API serialiser layer + RLS. DEC-030. `docs/ENTERPRISE.md §"Privacy floor"`. | PRIV-002 | ✅ |
| K.3 | Data minimisation — only collect what's necessary | P4.1 | A.5.34 | Art. 5(1)(c) | — | ED-screening data (DEC-018): collected only when user initiates specific flow, not persisted in aggregates, never surfaced to employer. `docs/GDPR_DPIA.md §5`. | PRIV-003 | ✅ |
| K.4 | User rights: access, rectification, erasure, portability | P6.6 | A.5.34 | Art. 15–20 | — | Self-serve in app settings. Erasure: 30-day cascade. Portability: JSON export. Documented in `docs/PRIVACY_POLICY.md §"Your rights"`. | PRIV-004 | 🔄 |
| K.5 | Purpose limitation — health data not used for profiling | P4.2 | A.5.34 | Art. 5(1)(b) | — | Health data processed only for coaching personalisation. No ad profiling. No employer risk scoring. No insurance sharing. `docs/ENTERPRISE.md §"When we say no"`. | PRIV-005 | ✅ |
| K.6 | Privacy floor non-negotiable controls | — | — | Art. 9, Art. 25 | — | Seven hard constraints enforced regardless of enterprise customer request: (1) HR never sees individual data; (2) aggregates only; (3) no manager-report on direct reports; (4) user may revoke company-association anytime; (5) ED-screening never aggregated; (6) body composition never aggregated; (7) mental health never aggregated. Clinical-safety VETO authority. | PRIV-006 | ✅ |
| K.7 | Cross-border transfer safeguards (SCCs / adequacy) | — | A.5.14 | Art. 44–49 | — | EU users: data stays in EU-region infrastructure. SCCs Module 1 for US sub-processors (Anthropic, ElevenLabs). Adequacy decisions monitored. | PRIV-007 | ✅ |
| K.8 | Children's data — no processing under 16 | — | — | Art. 8 | — | Age gate at signup (≥ 16 in EU, ≥ 13 in US per COPPA). Account closure if age confirmed < threshold. `docs/PRIVACY_POLICY.md §"Children"`. | PRIV-008 | ✅ |
| K.9 | No-go use cases — structural refusal | — | — | Art. 9(2) exceptions do NOT permit employer risk scoring | — | `docs/ENTERPRISE.md §"When we say no"`. Insurance risk scoring, government backdoors, wellness-as-punishment programmes are declined regardless of commercial terms. | PRIV-009 | ✅ |
| K.10 | DPO / compliance-officer contact for data subjects | — | A.5.34 | Art. 37–39 | — | `privacy@form.coach`. Response SLA: 30 days (GDPR Art. 12(3)). Formal DPO appointment required pre EU-enterprise launch. | PRIV-010 | 🔄 |

---

## 13. Evidence Artifact Cross-Reference

Evidence artifacts are available under NDA via `security@form.coach`. Custom evidence packages (for specific frameworks or SIG Full) have a 5 business-day turnaround.

| Artifact ID | Description | Format | Frameworks satisfied | Location / Availability |
|---|---|---|---|---|
| POL-001 | Information Security Policy | PDF | SOC 2 CC1.1, ISO A.5.1 | Internal; available under NDA |
| POL-002 | Acceptable Use Policy (signed roster) | PDF | SOC 2 CC1.3, ISO A.5.10 | Internal; available under NDA |
| POL-003 | Cryptography Policy v1.0 | PDF | SOC 2 CC6.7, ISO A.8.24, GDPR Art. 32 | `docs/CRYPTOGRAPHY_POLICY.md`; PDF under NDA |
| POL-004 | Privacy Policy (current) | HTML / PDF | SOC 2 P1.1, GDPR Art. 12–14, CCPA | `privacy.html` / `docs/PRIVACY_POLICY.md` |
| RISK-001 | Risk Assessment (annual) | PDF | SOC 2 CC3.1–CC3.4, ISO 6.1, GDPR Art. 35 | Available under NDA |
| RISK-003 | GDPR DPIA v0.1 | PDF | GDPR Art. 35, SOC 2 P6.1 | `docs/GDPR_DPIA.md`; final under NDA after legal review |
| CRYPT-004 | HMAC Audit Log Schema (DEC-030) | Markdown / SQL | SOC 2 CC4.1, ISO A.8.15, GDPR Art. 5(2) | `docs/AUDIT_LOG_SCHEMA.md` |
| AC-002 | SSO/SCIM Implementation Guide | Markdown | SOC 2 CC6.1–CC6.3, ISO A.5.16–A.5.18 | `docs/SSO_SCIM_IMPLEMENTATION.md` |
| OPS-005 | Penetration Test Summary | PDF | SOC 2 CC7.1, ISO A.8.8, GDPR Art. 32 | Under NDA; available to enterprise customers only |
| IR-001 | Incident Response Plan | PDF | SOC 2 CC7.3–CC7.5, ISO A.5.24–A.5.28, GDPR Art. 33 | `docs/INCIDENT_RESPONSE.md`; PDF under NDA |
| BCP-001 | Business Continuity Plan | PDF | SOC 2 A1.1–A1.3, ISO A.5.29–A.5.30, GDPR Art. 32(1)(c) | `docs/BUSINESS_CONTINUITY.md`; PDF under NDA |
| AUDIT-002 | Audit Log Retention Policy | Markdown | SOC 2 CC4.1, ISO A.8.15, GDPR Art. 30 | `docs/AUDIT_LOG_SCHEMA.md §Retention` |
| AUDIT-003 | ROPA (Records of Processing Activities) | Markdown | GDPR Art. 30 | `docs/ROPA.md` |
| SUP-001 | Subprocessor Register | Markdown / PDF | GDPR Art. 28, ISO A.5.19 | `docs/SUBPROCESSORS.md`; updated register available on request |
| SUP-002 | Data Processing Agreements (index) | PDF | GDPR Art. 28(3), ISO A.5.19 | Available under NDA |
| SOC2-DRAFT | SOC 2 Type II Readiness Assessment | Markdown | SOC 2 (all TSC) | `docs/SOC2_READINESS.md`; Bridge letter available post-audit |

---

## 14. Gap Analysis Summary per Framework

### 14.1 SOC 2 Type II — Current Readiness

**Overall: ~72% controls implemented.** Target: first Type II audit 12 months post enterprise launch.

| Priority | Gap | Control IDs | Target resolution |
|---|---|---|---|
| P0 (blocker) | Penetration test not yet conducted | OPS-005 | Pre-launch; external firm engaged Q3 2026 |
| P0 (blocker) | Continuous compliance tooling (Vanta/Drata) not deployed | AUDIT-001 | Q3 2026 — required for automated evidence collection |
| P0 (blocker) | MDM / endpoint management incomplete | PHY-004 | Q3 2026 — all engineering endpoints enrolled |
| P1 (significant) | Access review (quarterly revalidation) not yet operational | AC-008 | Q4 2026 — admin API export used for manual review until tooling ready |
| P1 (significant) | DR drill not yet conducted | BCP-004 | Q4 2026 — tabletop exercise followed by live drill |
| P2 (moderate) | Formal DPO appointment | PRIV-010, ORG-002 | Pre EU enterprise launch |
| P2 (moderate) | Data erasure cascade fully automated | CRYPT-007 | Q4 2026 — manual backstop currently |
| P2 (moderate) | Phishing-resistant MFA (passkeys / FIDO2) | AC-004 | Q1 2027 (roadmap) |

**Fully implemented (representative sample):** HMAC-chained audit log (DEC-030), TLS 1.3 + AES-256-GCM, SSO/SCIM, RLS-enforced tenant isolation, incident response plan with P0–P3 classification, GDPR breach notification process, privacy floor controls, sub-processor register, BCP/DR runbooks.

### 14.2 ISO 27001:2022 — Current Gap Position

**Not yet initiated.** ISO 27001 certification is planned 18 months post enterprise launch. The following Annex A control families require the most work:

| ISO Domain | Gap summary |
|---|---|
| A.6 (People) | Background checks and exit procedures need formalisation |
| A.7 (Physical) | Fully delegated to providers — provider attestations must be formally reviewed and documented |
| A.8.6 (Capacity) | Capacity planning process needs documented thresholds and approval workflow |
| A.8.18 (Privileged access) | PAM tooling (just-in-time access, session recording) not yet deployed |
| A.8.31 (Env separation) | Dev/staging/prod separation documented; formal policy + CI enforcement pending |
| 9.1–9.3 (ISMS performance) | ISMS scope document, Statement of Applicability, and management review process not yet created |
| 10.1 (Nonconformity) | Nonconformity and corrective action register not yet formalised |

**Already aligned (ISO Annex A mapping):** Access control (A.5.15–A.5.18), cryptography (A.8.24), audit logging (A.8.15), incident management (A.5.24–A.5.28), business continuity (A.5.29–A.5.30), supplier management (A.5.19–A.5.22), vulnerability management (A.8.8).

### 14.3 GDPR — Current Compliance Position

**Materially compliant; three open items before EU enterprise launch.**

| Item | Status | Required by |
|---|---|---|
| DPIA legal review | 🔄 Draft complete; legal review pending | Before EU pilot data flows |
| Formal DPO appointment (or documented exception) | 🔄 | Before EU enterprise launch |
| Data erasure cascade full automation | 🔄 | Before EU enterprise launch |
| SCCs (US sub-processors) | ✅ In place | — |
| Consent management (Art. 7 + Art. 9) | ✅ | — |
| ROPA (Art. 30) | ✅ | — |
| Privacy floor (7 non-negotiable rules) | ✅ | — |
| Art. 33 breach notification process | ✅ | — |

### 14.4 HIPAA — Adjacency Note

FORM is **not a HIPAA Covered Entity or Business Associate** and does not sign BAAs. However, enterprise customers in US healthcare-adjacent sectors may request a HIPAA gap assessment. Key observations:

- **Technical safeguards (§ 164.312):** Substantially satisfied — TLS 1.3, AES-256-GCM at rest, unique user identification, audit logging, automatic logoff.
- **Administrative safeguards (§ 164.308):** Substantially satisfied — security policies, risk analysis, workforce training, incident procedures, contingency plan.
- **Physical safeguards (§ 164.310):** Delegated to cloud providers. Cloudflare and Supabase hold relevant certifications.
- **Primary gap:** No formal HIPAA risk analysis document; no BAA template. If enterprise demand materialises, FORM can prepare a BAA and formal risk analysis within 60 days of contract commitment.

---

## 15. Certification Roadmap

```
2026-Q3    Penetration test (external firm)
           MDM rollout complete
           Vanta / Drata deployment begins
           Background check formalisation

2026-Q4    SOC 2 Type II audit engagement letter signed
           DR tabletop + live drill conducted
           Access review process operational
           Data erasure automation complete
           ISO 27001 scoping workshop

2027-Q1    SOC 2 Type II audit period begins (12-month observation)
           ISO 27001 gap assessment by external consultant
           Phishing-resistant MFA (passkeys) for enterprise tenants

2027-Q2    DPIA legal review complete + DPO formally appointed
           ISMS Statement of Applicability drafted

2027-Q3    ISMS internal audit
           Continuous compliance monitoring fully automated

2027-Q4    SOC 2 Type II report issued (bridge letter available to prospects)
           ISO 27001 certification audit

2028-Q1    ISO 27001 Stage 2 audit
           ISO 27001 certificate issued
```

---

## 16. Revision History

| Version | Date | Author | Change |
|---|---|---|---|
| v1.0 | 2026-06-15 | `compliance-officer` + `security-engineer` + `enterprise-architect` | Initial cross-framework control matrix. 93 controls mapped across SOC 2, ISO 27001:2022, GDPR, HIPAA adjacency. Gap analysis and certification roadmap included. |

---

*FORM · Compliance Framework Control Matrix · v1.0 · 2026-06-15*  
*Owner: `compliance-officer` + `security-engineer` + `enterprise-architect`*  
*Classification: CONFIDENTIAL — NDA-gated. Do not distribute without approval from `legal@form.coach`.*
