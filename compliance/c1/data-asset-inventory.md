# FORM Confidential Data Asset Inventory

**Document ID:** C1-INV-001  
**Artifact ID (SOC 2):** PRE-34-E-001  
**Remediates:** C1-GAP-002 — no standalone auditor-presentable data asset inventory previously existed as a discrete signed exhibit  
**SOC 2 criteria:** C1.1 (identify and maintain confidential information consistent with the entity's objectives related to confidentiality)  
**GDPR articles:** Art. 30 (records of processing activities) · Art. 5(1)(e) (storage limitation) · Art. 9(1) (special category health data)  
**Owner:** compliance-officer + data-engineer  
**Status:** 🟡 Authored — pending founder signature for full C1-GAP-002 closure  
**Version:** 0.1  
**Effective date:** [Founder signature date — TBD]  
**Next review:** Annual — Q1 January, or upon any material change to FORM's data architecture, new data category, new sub-processor, or material change in any retention period  
**Artifact path:** `compliance/c1/data-asset-inventory.md`

---

## 1. Purpose and Scope

### 1.1 Purpose

AICPA SOC 2 C1.1 requires that the entity identifies all confidential information it creates, collects, and processes, and places it under controls commensurate with its classification level. This document is the formal, version-controlled, founder-signed inventory of all systems that hold or process data classified as Confidential, Sensitive, or Restricted under FORM's Data Classification Policy (hereafter §DCP, located at `docs/SOC2_READINESS.md § Data Classification Policy`).

This inventory serves three audiences:

1. **Auditors** — the primary evidence exhibit for C1.1 (data inventory sub-criterion), filed as PRE-34-E-001 in the SOC 2 evidence library; provides auditors with a single document from which to form a preliminary view of FORM's confidential information landscape
2. **Enterprise customers** — provided on request during security questionnaire responses; establishes that FORM knows where all sensitive data lives and under what controls
3. **Internal team** — authoritative reference for data handling decisions; all engineers and future hires must understand the classification of data they access before receiving production access

### 1.2 Scope

This inventory covers all systems in which FORM personal data at classification level Confidential, Sensitive, or Restricted resides, transits, or is processed:

- Supabase (primary PostgreSQL database + Auth + Edge Functions)
- Cloudflare Workers (compute), R2 (Tier 2 backup), KV (session cache)
- Backblaze B2 (Tier 3 cold storage / WORM archive)
- All active sub-processors that receive FORM personal data (cross-reference: `compliance/p1/sub-processor-register.md`)
- GitHub repository and CI/CD pipeline (source code + secrets surface)
- 1Password Teams (credential vault)
- Cloudflare Workers Secrets (runtime secret store)
- Analytics systems (PostHog EU, ClickHouse — M9+)

**Excluded from scope (independent controllers):**

- Apple Inc — iOS App Store IAP payment processing. Apple is an independent controller for payment PII; FORM receives only receipt tokens. No instruction relationship exists within GDPR Art. 4(8) definition.
- Google LLC — Android Play Store. Same as Apple.

These entities are noted in `compliance/p1/sub-processor-register.md §5.8` for disclosure completeness.

**In-transit data** (data that passes through Cloudflare CDN / WAF without being persisted) is governed by the TLS-in-transit controls in `docs/SECURITY.md §2` and is not independently inventoried here.

### 1.3 Classification Framework

FORM's Data Classification Policy defines four tiers with a health-data overlay:

| Tier | Label | Scope | Primary handling requirement |
|---|---|---|---|
| 0 | **Public** | Intentionally public: marketing website, open-source code, press releases | No restriction |
| 1 | **Internal** | Internal use; not for external distribution without approval: engineering runbooks, internal analytics aggregates | Need-to-know within team; no external transmission without approval |
| 2 | **Confidential** | Business-sensitive: enterprise tenant configurations, subscription metadata, B2B contract terms, source code | Access controls + DPA required for any sub-processor; no transmission without encryption in transit |
| 3 | **Sensitive** | GDPR Art. 9 special-category health data; biometric-adjacent inference data | Explicit consent gate (per DEC-030 `consent.granted` event) + column-level encryption on highest-sensitivity fields + Art. 9(2)(a) legal basis required; sub-processor DPA must explicitly cover Art. 9 data |
| 4 | **Restricted** | Production credentials, signing keys, HMAC secrets, API keys, private keys, break-glass access materials | Vault management (1Password / Cloudflare Workers Secrets) + DEC-030 HMAC-chained audit event on every access + break-glass dual-authorization (SECURITY.md §8) |

**Health-data overlay:** Any data category from DC-02 through DC-07 (defined in `compliance/p1/sub-processor-register.md §2`) is classified **Sensitive** regardless of the table it appears in, and is subject to full GDPR Art. 9 protection obligations. This classification is not overridden by table-level defaults.

---

## 2. Primary Database — Supabase (PostgreSQL)

**System:** Supabase managed PostgreSQL  
**Deployment region:** `eu-central-1` (Frankfurt) — EU production; `eu-west-1` (Dublin) — EU enterprise tenant routing  
**Encryption at rest:** AES-256-GCM, Supabase default, applied to all tables and tablespaces  
**Encryption in transit:** TLS 1.3 enforced; `sslmode=require` on all connection strings; no plaintext database connections permitted  
**Access controls:** Row-Level Security (RLS) enforced on all user-data tables — see §2.2; four-role RBAC (owner / admin / member / viewer) per `docs/SSO_SCIM_IMPLEMENTATION.md §4`; break-glass access requires `incident_id` + dual authorization per `docs/SECURITY.md §8`  
**Backup coverage:** Tier 1 — Supabase PITR (point-in-time recovery, 30-day window); Tier 2 — Cloudflare R2 nightly encrypted exports (90-day TTL); Tier 3 — Backblaze B2 WORM object-locked archive (7-year retention)  
**Audit:** Every access to Sensitive or Restricted data generates a DEC-030 HMAC-chained audit event with `data_class` field per `docs/AUDIT_LOG_SCHEMA.md`

### 2.1 Table Inventory

| Table | Data category | Classification | Decided retention | Column-level encryption | Backup tiers |
|---|---|---|---|---|---|
| `user_profile` — identity fields (`email`, `display_name`, `auth_provider`) | DC-01 | **Confidential** | Until account deletion (account-active data) | None — AES-256 at rest (Supabase default) | T1, T2, T3 |
| `user_profile` — health fields (`height_cm`, `weight_kg_goal`, `biological_sex`, `date_of_birth`, `fitness_goal`, `injury_flags`, `restrictions`) | DC-02 | **Sensitive — Art. 9(1)** | Until account deletion or consent withdrawal; GDPR Art. 17 immediate on request | `pgp_sym_encrypt` on `injury_flags`; remaining columns pending PRE-34-E-006 (C1-GAP-002 note) | T1, T2, T3 |
| `workout_sessions` | DC-03 | **Sensitive — Art. 9 inference** | **3 years** from session date (P1-RET-001 §4.1) | None — AES-256 at rest | T1, T2, T3 |
| `sets` | DC-03 | **Sensitive — Art. 9 inference** | **3 years** (cascade delete from `workout_sessions`) | None | T1, T2, T3 |
| `coaching_turns` | DC-06 | **Sensitive — Art. 9** | **2 years** from creation (P1-RET-001 §4.1) | `pgp_sym_encrypt` on `content` column (DATA_MODEL.md §6) | T1, T2, T3 |
| `cv_sessions` | DC-04 | **Sensitive — Art. 9 (biometric-adjacent)** | **1 year** from creation; `keypoints_enc` nullified immediately (no grace period) on erasure request (DATA_MODEL.md §15) | AES-256-GCM on `keypoints_enc` via Cloudflare Workers Secret (DATA_MODEL.md §15) | T1, T2, T3 |
| `wearable_readings` | DC-05 | **Sensitive — Art. 9(1)** | **2 years** from reading date (P1-RET-001 §4.1) | None — AES-256 at rest | T1, T2, T3 |
| `meal_log` | DC-07 | **Sensitive — Art. 9 (dietary)** | **2 years** from entry creation (P1-RET-001 §4.1) | None — AES-256 at rest | T1, T2, T3 |
| `audit_log` (DEC-030 HMAC chain) | Internal audit records; anonymized user references post-erasure | **Restricted** | **7 years** (SOC 2 evidence requirement; GDPR Art. 30 record-keeping) | HMAC-SHA-256 chain integrity per DEC-030; content not encrypted (integrity-protected) | T1 (PITR), T3 (WORM — immutable) |
| `tenants` | DC-01 (tenant identity); enterprise contract metadata; SSO configuration | **Confidential** | Until tenant offboarding + 3 years (legal hold) | None — AES-256 at rest | T1, T2, T3 |
| `tenant_users` | DC-01 (SCIM-provisioned user directory: name, email, role, department) | **Confidential** | Until SCIM deprovisioning + 30-day grace period | None | T1, T2, T3 |
| `programs` | Training programme definitions (no personal data) | **Internal** | Indefinite (product content asset) | None | T1, T2 |
| `subscriptions` | DC-08 (subscription tier, IAP receipt tokens, renewal status) | **Confidential** | Until account deletion + 3 years (financial records) | None — AES-256 at rest | T1, T2, T3 |
| `analytics_events` (staging queue, pre-PostHog sync) | DC-10 (pseudonymized events) | **Internal** | 30-day TTL (PostHog sync window) | None | T1 only |

**Retention authority:** All retention periods in this table derive from `compliance/p1/retention-decisions.md` (P1-RET-001). Any change to a retention period requires a new version of P1-RET-001 with updated founder sign-off, followed by a new version of this document.

### 2.2 RLS Access Control Matrix

| Postgres role | User's own Sensitive data | Other users' data | Tenant aggregate | Admin/break-glass |
|---|---|---|---|---|
| `form_user` | Read + write own rows only (all Sensitive tables) | **No access** — RLS predicate blocks cross-user queries at DB layer | **No access** | No |
| `form_coach` (Victor AI context) | Read coaching context only (session-scoped `coaching_turns`) | **No access** | **No access** | No |
| `form_tenant_admin` | **No individual data** — privacy floor: HR never sees individual user data | **No access** | Aggregated only (k-anonymity floor N ≥ 5; suppressed below threshold) | Tenant configuration only |
| `form_system` | Full read/write within operation scope (service operations) | Full (bounded by operation context and scope) | Full | No break-glass capability |
| `form_admin` (break-glass) | Full read; write gated by dual-authorization | Full read; write gated | Full | Requires `incident_id` + IC approval + DEC-030 HIGH-severity audit event; separate decryption endpoint for `keypoints_enc` |

*Source: `docs/DATA_MODEL.md §8` (RLS policies + CI assertions); `docs/SSO_SCIM_IMPLEMENTATION.md §4` (RBAC)*

---

## 3. Cloudflare Infrastructure

### 3.1 Cloudflare Workers (API Compute Layer)

**System:** Cloudflare Workers serverless runtime (all API routes, auth middleware, SCIM handlers, webhook delivery)  
**Classification:** **Restricted** — Workers have runtime access to production secrets via Workers Secrets binding; all credentials injected as environment variables, not logged  
**Persistent data storage:** None. Workers process data in-memory per request; no data persisted between invocations in the Worker process itself  
**Secrets held:** Supabase service-role key, Anthropic API key, ElevenLabs API key, HMAC signing secret (DEC-030), CV decryption key, Stripe webhook secret — all via Cloudflare Workers Secrets (see §6.3)  
**Encryption in transit:** TLS 1.3 at Cloudflare edge; HSTS preload enforced; origin pull over HTTPS  
**Audit:** Every access to Sensitive data via a Worker generates a DEC-030 HMAC event with `actor_type = "system"` and the relevant `data_class` tag

### 3.2 Cloudflare R2 (Tier 2 Backup Store)

**System:** Cloudflare R2 object storage  
**Classification:** **Restricted** — encrypted backup files contain all Supabase table data  
**Data categories:** DC-01 through DC-07, DC-08 (in encrypted backup exports)  
**Encryption at rest:** SSE-C (server-side encryption, FORM-managed AES-256 keys); keys held in 1Password (offline escrow)  
**Encryption in transit:** TLS 1.3  
**Retention:** 90-day TTL enforced by R2 Object Lifecycle rules; erasure-triggered purge on next rotation after `privacy.erasure_completed` DEC-030 event  
**Access controls:** R2 API key scoped to write + read (no delete) for devops-lead role; no public access; Worker-level access via bound R2 bucket binding only; no public endpoint  
**Sub-processor DPA:** Cloudflare Enterprise DPA, SCC Module 2

### 3.3 Cloudflare Workers KV (Session Cache)

**System:** Cloudflare KV (key-value store)  
**Classification:** **Confidential** — session tokens are access credentials  
**Data categories:** Session tokens, rate-limit counters, temporary authentication state, ElevenLabs character-count delta (cost monitor — no personal data)  
**Retention:** Session TTL ≤ 24 hours; no persistent personal data stored in KV  
**Encryption:** Cloudflare KV values encrypted at rest (Cloudflare-managed); TLS 1.3 in transit  
**Access:** Only the Worker bound to the KV namespace can read/write; no external KV API access

---

## 4. Backblaze B2 — Cold Storage (Tier 3 / WORM Archive)

**System:** Backblaze B2 WORM (Write-Once-Read-Many) object store  
**Classification:** **Restricted** — encrypted; includes 7-year immutable audit log chain  
**Data categories:** Long-term encrypted database archives (DC-01 through DC-07); DEC-030 audit log WORM copies; SOC 2 compliance evidence exports; `cost_daily` CSV exports (`compliance/evidence/`)  
**Encryption at rest:** SSE-B2 (Backblaze native AES-256) + FORM AEAD encryption layer on audit log exports before upload  
**Encryption in transit:** TLS 1.3  
**Retention:** 7-year immutable WORM Object Lock (no early deletion possible at the object storage layer; even FORM cannot delete before the lock expires)  
**Key escrow:** B2 Application Key and FORM AEAD layer key held in offline escrow (1Password emergency kit); devops-lead hold; founder secondary  
**Access controls:** B2 Application Key scoped to write + read; no delete permission on WORM bucket; every access generates DEC-030 `backup.accessed` event (HIGH severity)  
**Sub-processor DPA:** Backblaze GDPR DPA; SCC Module 2 (US-EU transfer); reviewed annually

**Anonymization-on-restore policy:** Because WORM backups cannot be retroactively modified after user erasure, FORM's restore procedure (`docs/DATA_MODEL.md §10`) includes a mandatory anonymization step: before any WORM-tier backup data is surfaced in a restored environment, the anonymization script (`src/analytics/backfills/erasure_template.sql`) is applied to suppress rows for users whose `privacy.erasure_completed` event was filed before the restore date.

---

## 5. Sub-Processor Data Holdings Summary

Full DPA status, tier classification, and risk assessments are in `compliance/p1/sub-processor-register.md`. This section provides the data-asset view — what data each sub-processor holds or processes on FORM's behalf.

| Sub-processor | Data held / processed | Classification | Retention at processor | Notes |
|---|---|---|---|---|
| **Anthropic** (Claude API) | DC-06 coaching context in-transit only; `stripPersonalProperties()` applied before every API call — no name, email, or PII injected into prompts | Sensitive (in-transit only; zero retention at Anthropic per DPA) | Zero — DPA prohibits use of FORM data for model training | Pseudonymized coaching context only; coaching transcript content stays in Supabase (`coaching_turns`) |
| **ElevenLabs** (TTS API) | Coaching text synthesized to audio in-transit only; audio streamed to client, not stored by FORM or ElevenLabs | Sensitive (in-transit only) | Zero — audio not persisted per DPA | Voice audio never stored server-side; client playback only |
| **PostHog EU** | DC-10 pseudonymized event stream (`distinct_id` = hashed `user_id`); `person_profiles: "identified_only"`; health event content excluded by `stripForbiddenProperties()` middleware | Internal | 2 years (ClickHouse TTL at PostHog EU) | EU servers only; GDPR DPA signed; SCC not required (EU–EU transfer) |
| **Sentry** | DC-09 scrubbed crash reports: stack traces, anonymized user IDs; health-adjacent fields removed by `beforeSend` hook before transmission | Internal | 90 days (Sentry default) | 🟡 DPA in progress; compensating control: PII scrubbed at SDK layer before leaving FORM's boundary |
| **Stripe** | DC-08 subscription tier, renewal status, IAP receipt validation tokens; no card data (Apple/Google hold PAN) | Confidential | Per PCI DSS + Stripe DPA | Stripe DPA signed; SCC Module 2 (US-EU) |
| **Supabase** | DC-01 through DC-07 (all user health data), DC-08, DC-10 (analytics queue) | Sensitive — Art. 9 | Per FORM retention policy (P1-RET-001) | DPA signed; SCC Module 2; `eu-central-1` for EU production |
| **Cloudflare** | DC-01 in-transit (non-R2 path); DC-all in R2 (encrypted backup exports only) | Confidential / Restricted | R2: 90-day TTL; KV: session TTL; Workers: no persistence | Enterprise DPA signed; SCC Module 2 |
| **Better Stack** | Synthetic probe results; sanitized structured log data (no user PII in log pipelines per OBSERVABILITY.md §4) | Internal | 90 days | 🔴 DPA pending; compensating control: only non-PII operational metrics in log pipelines |

---

## 6. Source Code and Secrets Management

### 6.1 GitHub Repository (`Rbit27/form`)

**System:** GitHub private repository  
**Classification:** **Restricted** — proprietary source code; CI/CD pipeline with deployment access  
**Personal data stored:** None — production personal data must never appear in source code or git history; `git-secrets` pre-commit hook (CC6-GAP-006, pending deployment) enforces this  
**Secrets policy:** All credentials are stored in 1Password / Cloudflare Workers Secrets (§6.2, §6.3); zero secrets in `.env` files committed to git; Dependabot security alerts enabled; secret scanning enabled  
**Access controls:** Team-based GitHub access control; 2FA enforcement pending (CC6-GAP-001); principle of least privilege per role  
**Audit:** GitHub audit log (90-day retention in GitHub); DEC-030 not applicable (source code system, no personal data boundary)

### 6.2 1Password Teams (Credential Vault)

**System:** 1Password Teams  
**Classification:** **Restricted** — holds all production API keys, database passwords, signing keys, and backup encryption keys  
**Data categories:** All Restricted credentials (Supabase service-role key, Anthropic API key, ElevenLabs API key, Stripe secret key, Backblaze B2 key, Cloudflare API token, HMAC signing secret, CV decryption key, R2 SSE-C key)  
**Encryption:** 1Password AES-256-GCM with PBKDF2 key derivation; end-to-end encrypted — 1Password cannot access vault contents; FORM does not have access to 1Password master encryption infrastructure  
**Access controls:** Founder-only production vault; developer vault restricted to non-production credentials; MFA enforced on all 1Password accounts; emergency access kit in offline escrow  
**Audit:** 1Password activity log (per-access); DEC-030 `iam.break_glass_accessed` event fired on every production credential access outside normal deployment automation

### 6.3 Cloudflare Workers Secrets (Runtime Secret Store)

**System:** Cloudflare Workers Secrets (encrypted environment variable store for Workers runtime)  
**Classification:** **Restricted** — these are the live production credentials that Workers use at runtime  
**Secrets held:** Supabase service-role key, Anthropic API key, ElevenLabs API key, Stripe webhook secret, HMAC signing secret (DEC-030 chain), CV decryption key (`CV_KEYPOINTS_KEY`), `COST_THRESHOLDS` JSON blob  
**Encryption:** Secrets encrypted at rest by Cloudflare; not accessible via Cloudflare dashboard after initial `wrangler secret put` (write-only after creation); not written to Worker logs  
**Access controls:** Cloudflare account MFA enforcement pending (CC6-GAP-002); access limited to devops-lead and security-engineer roles; no developer read access to production Workers Secrets values  
**Rotation schedule:** Per `docs/CRYPTOGRAPHY_POLICY.md`: API keys every 180 days; JWT signing key every 90 days; HMAC chain key annually (with chain-continuity procedure)

---

## 7. Analytics and Business Intelligence

| System | Data categories | Classification | Retention | Access controls | Notes |
|---|---|---|---|---|---|
| **PostHog EU** (active) | DC-10 (pseudonymized events; `distinct_id` = hashed user ID) | Internal | 2 years (PostHog EU ClickHouse TTL) | data-engineer + devops-lead access; health-adjacent events excluded via `stripForbiddenProperties()` | `person_profiles: "identified_only"` — no PII profiles created; health event content categorically excluded |
| **ClickHouse** (M9+, not yet deployed) | DC-10 (pseudonymized events); cohort aggregates (k-anonymity N ≥ 5) | Internal | 2 years (TTL enforced in DDL) | data-engineer + devops-lead access; analytics role gated | ClickHouse will receive PostHog export on M9 deployment; no health data by design |
| **Metabase** (query layer) | Query interface over Supabase + ClickHouse | Confidential (can query enterprise tenant aggregates) | No storage — query tool only | data-engineer + devops-lead; enterprise tenant aggregate view restricted to compliance-officer role | No individual user data visible in Metabase; tenant aggregate view enforces k-anonymity N ≥ 5 |

---

## 8. Data Erasure and Disposal Controls

### 8.1 User-Initiated Erasure (GDPR Art. 17)

On erasure request or account deletion, the following sequence executes within the defined SLAs:

| Step | Action | SLA | Mechanism |
|---|---|---|---|
| 0 | Consent withdrawal recorded | Immediate | DEC-030 `consent.withdrawn` event |
| 1 | `cv_sessions.keypoints_enc` nullified | Immediate (no grace period) | Art. 17 biometric elevated right; direct DB update |
| 2 | 30-day grace period begins | — | User-restorable window |
| 3 | Hard delete — Supabase cascade | Day 30 | `cv_sessions` → `coaching_turns` → `meal_log` → `wearable_readings` → `workout_sessions` → `sets` → `user_profile` |
| 4 | `audit_log` anonymization | Day 30 | `user_id` → `[DELETED-{sha256(user_id)}]`; HMAC chain preserved |
| 5 | PostHog `distinct_id` deletion | Within 30 days | PostHog deletion API |
| 6 | ClickHouse `user_id` row deletion | Within 30 days (M9+) | `erasure_template.sql` |
| 7 | DEC-030 `privacy.erasure_completed` event | Day 30 | Row counts + timestamp filed to audit chain |
| 8 | R2 backup flagged for purge | Next TTL rotation (≤90 days) | Erasure queue worker flag |
| 9 | B2 WORM — anonymize-on-restore | On any restore from WORM after Day 30 | Mandatory pre-surface anonymization step |

*Source: `docs/INCIDENT_RESPONSE.md §12.5`, `docs/DATA_MODEL.md §13.6.2`*

### 8.2 Scheduled Retention Expiry (Automated)

Time-based TTL sweeps execute nightly at 03:00 UTC via Supabase `pg_cron` jobs (configuration in P1-RET-001 §4.1). Each sweep emits a DEC-030 `privacy.scheduled_purge` event with row count and table name. The sweep is idempotent — safe to re-run on failure.

### 8.3 Media and Device Disposal (Interim Rule)

The formal media/device disposal policy (`compliance/c1/device-disposal-policy.md`) is under authorship (C1-GAP-004; target M3). Until that policy is published and signed, the following interim rule applies:

**Interim rule:** Any device that has ever held a Cloudflare Workers production secret, a Supabase connection string with elevated privileges, or local copies of production-tier backup files must be wiped to **NIST SP 800-88 Purge level** (cryptographic erase for SSDs; verified overwrite for HDDs) before disposal, resale, or return. Every device disposal must be recorded in a Linear ticket with: device serial number, disposal date, wipe method, and sign-off by compliance-officer or security-engineer.

Mobile test devices that connected to production APIs during incident response or debugging are subject to the same rule even if they did not hold local copies of credentials.

---

## 9. Gap Status at Inventory Date (2026-05-22)

| Gap ID | Description | Priority | Status |
|---|---|---|---|
| **C1-GAP-001** | NDA / employment confidentiality agreement template — required before any hire with production access | P0 — pre-hire gate | 🔴 Open — target M3 |
| **C1-GAP-002** | This document (Confidential Data Asset Inventory) | P1 | 🟡 Authored — pending founder signature; PRE-34-E-001 |
| **C1-GAP-003** | DPA receipts for all sub-processors filed in `compliance/c1/dpas/` — Sentry and Better Stack outstanding | P0 — pre-enterprise gate | 🟡 Partial — target M3 |
| **C1-GAP-004** | Media/device disposal policy (`compliance/c1/device-disposal-policy.md`) | P1 | 🔴 Open — interim rule in §8.3; target M3 |
| **PRE-34-E-006** | Column-level encryption DDL implementation commit — `coaching_turns.content`, remaining `user_profile` health fields | P1 | 🟡 Partial — DDL authored in DATA_MODEL.md §6; implementation commit pending |

---

## 10. Signatory Block

This document is authoritative as of the founder signature date. Any material change to FORM's data architecture — a new table holding personal data, a new sub-processor, a new data category, or a change to any retention period — requires a new version of this document with updated founder sign-off within 30 days of the change taking effect.

| Role | Name | Signature confirmation | Date |
|---|---|---|---|
| Founder / Data Controller Representative | [Founder full name] | _______________________________________________ | ________________ |
| Compliance Officer | compliance-officer (enterprise-builder) | _______________________________________________ | ________________ |

**Version:** 0.1-draft  
**Next review date:** January 2027 (annual) or upon material architecture change, whichever is first  
**Filing path:** `compliance/c1/data-asset-inventory.md`  
**SOC 2 artifact ID:** PRE-34-E-001  
**Evidence directory:** `compliance/c1/`

---

*v0.1 (2026-05-22): Initial authorship. Remediates C1-GAP-002. Covers 14 Supabase table entries across DC-01 through DC-10 data categories; Cloudflare Workers (compute, R2 Tier 2 backup, KV session cache); Backblaze B2 WORM Tier 3 (7-year immutable); 8 sub-processor data holdings cross-referenced to `compliance/p1/sub-processor-register.md`; GitHub repository and CI/CD pipeline; 1Password Teams credential vault; Cloudflare Workers Secrets runtime store; PostHog EU, ClickHouse (M9+), and Metabase analytics layer. Four-tier classification scheme (Public/Internal/Confidential/Sensitive/Restricted) plus Art. 9 health-data overlay applied throughout. Retention periods drawn from P1-RET-001 (formally decided May 2026). RLS access control matrix cross-referenced from DATA_MODEL.md §8 and SSO_SCIM_IMPLEMENTATION.md §4. Erasure sequence in §8.1 cross-referenced from INCIDENT_RESPONSE.md §12.5 and DATA_MODEL.md §13.6.2. Device disposal interim rule in §8.3 pending formal policy (C1-GAP-004). Gap status table covers C1-GAP-001 through C1-GAP-004 and PRE-34-E-006. Pending founder signature for full C1-GAP-002 closure.*

**Co-Authored-By:** Claude (enterprise-builder) <cloud@form.coach>
