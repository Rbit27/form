# FORM · Data Residency v1.0

> **Document type:** Technical and operational reference for enterprise data residency commitments.  
> **Owner:** `compliance-officer` + `enterprise-architect`. Co-reviewed: `security-engineer`, `devops-lead`.  
> **Status:** Active — governs all enterprise contracts from v1.0 onward.  
> **References:** `docs/ENTERPRISE.md` (hard commitment §What we promise customers), `docs/GDPR_DPIA.md` §3.3 (legal basis summary), `docs/AUDIT_LOG_SCHEMA.md` (DEC-030 HMAC chain, region-resident clause), `docs/SSO_SCIM_IMPLEMENTATION.md` (tenant provisioning), `docs/SUBPROCESSORS.md` (full sub-processor register with DPA status).  
> **Review cadence:** Quarterly, or on any change to infrastructure region topology.

---

## Table of Contents

1. [Purpose and Scope](#1-purpose-and-scope)
2. [Data Classification](#2-data-classification)
3. [Available Regions](#3-available-regions)
4. [Technical Implementation per Service Layer](#4-technical-implementation-per-service-layer)
   - 4.1 Primary Database — Supabase Postgres
   - 4.2 Object Storage — Cloudflare R2
   - 4.3 Edge Compute — Cloudflare Workers
   - 4.4 Identity — Auth0 / WorkOS (Enterprise SSO)
   - 4.5 Audit Logs — DEC-030 HMAC Chain
5. [Sub-Processor Residency and Transfer Mechanisms](#5-sub-processor-residency-and-transfer-mechanisms)
   - 5.1 Anthropic (Victor AI coach)
   - 5.2 ElevenLabs (Voice synthesis)
   - 5.3 Cloudflare (CDN, Workers, R2, Pages)
   - 5.4 PostHog (Product analytics)
   - 5.5 Sentry (Error monitoring)
   - 5.6 Better Stack (Log indexing / uptime)
6. [Tenant Region Configuration](#6-tenant-region-configuration)
7. [Data Portability and Export](#7-data-portability-and-export)
8. [Data Deletion](#8-data-deletion)
9. [EU–US Transfer Mechanisms](#9-euus-transfer-mechanisms)
10. [Residency Attestation](#10-residency-attestation)
11. [Privacy Floor Interaction](#11-privacy-floor-interaction)
12. [Roadmap](#12-roadmap)
13. [Open Questions](#13-open-questions)

---

## 1. Purpose and Scope

Data residency determines **where FORM stores and processes user data at rest and in transit**. For enterprise customers, residency is a contractual commitment, not a best-effort configuration.

FORM's public residency commitment (ENTERPRISE.md §What we promise customers):

> _"Data residency (EU customers stay in EU)"_

This document operationalises that commitment: what it means technically, which services are in scope, how it is configured per tenant, and what FORM can and cannot guarantee given its use of US-headquartered sub-processors.

**In scope:** All personal data processed under an enterprise contract — workout data, health profile, meal logs, aggregated wellness metrics, audit events, admin actions, SSO session tokens.

**Out of scope:** Stripe billing data (Stripe is data controller for payment processing). App Store / Google Play install metadata (outside FORM's control). On-device CV inference (processed and discarded locally — never transmitted; see §2).

---

## 2. Data Classification

Not all FORM data is equivalent from a residency perspective.

| Category | Residency scope | Sensitivity | Notes |
|---|---|---|---|
| **Account identity** | Tenant region | High | Email, display name, provisioning method (SCIM vs manual) |
| **Health profile** | Tenant region | Special category (Art. 9) | DoB, biological sex, height, resting HR, HRV baseline, goals, injury flags |
| **Workout records** | Tenant region | Special category (Art. 9) | Sets, reps, RPE, programme state, form score metadata |
| **Meal logs** | Tenant region | Special category (Art. 9) | Food items, macros, timestamps |
| **CV inference output** | On-device only | Special category (Art. 9) | Joint angles computed on device; only `form_score` + metadata written to DB in tenant region |
| **Victor AI context** | In-flight (US Anthropic API) | Pseudonymised | No PII in prompt payload; pseudonymised exercise context only. SCC Module 2 governs transfer. |
| **Admin dashboard events** | Tenant region | High | Seat provisioning, SCIM events, role changes, billing events |
| **Audit log** | Tenant region (DEC-030) | High | Append-only, HMAC-chained; region-resident is non-negotiable |
| **Aggregate wellness metrics** | Tenant region | Low (k-anon ≥ 5) | Employer-visible dashboard data; no individual PII |
| **SSO tokens** | In-flight | High | SAML assertions / OIDC tokens processed at edge, not stored; see §4.3 |

**CV inference clarification:** FORM's on-device pose estimation pipeline runs entirely on the user's device (Apple Vision Framework / Google ML Kit). Raw video frames never leave the device. Only the computed `form_score` integer and session metadata are written to the backend. This means the most sensitive body-image data is never transmitted to FORM servers regardless of tenant region.

---

## 3. Available Regions

| Region ID | Data centre | Cloud anchor | Availability | Target customers |
|---|---|---|---|---|
| `us-east-1` | Ashburn, VA / Dallas, TX | Supabase US East | GA | US customers (default) |
| `eu-central-1` | Frankfurt, Germany | Supabase EU (Frankfurt) | GA | German, Austrian, Swiss, DACH customers |
| `eu-west-1` | Dublin, Ireland | Supabase EU (Ireland) | GA | Polish, UK/IE, Dutch, Nordic customers |

**How region maps to customer geography:**

| Customer country | Recommended region | Notes |
|---|---|---|
| United States, Canada | `us-east-1` | Default; no EU/UK GDPR obligations |
| Germany, Austria, Switzerland | `eu-central-1` | Satisfies German data sovereignty expectations; Frankfurt jurisdiction |
| Poland, Netherlands, Belgium, France | `eu-west-1` | EU but outside German jurisdiction preference |
| United Kingdom | `eu-west-1` | UK GDPR requires EU adequacy or SCCs — Dublin satisfies via EU data location + UK adequacy decision |
| Ireland, Nordics | `eu-west-1` | Natural Dublin choice |

**Region is set at contract signing.** It cannot be changed post-deployment without a paid migration engagement and a re-executed Data Processing Addendum (DPA). This constraint exists because Supabase does not support live cross-region migrations without a downtime window, and audit log continuity (HMAC chain, DEC-030) cannot span regions without breaking chain integrity.

---

## 4. Technical Implementation per Service Layer

### 4.1 Primary Database — Supabase Postgres

Supabase is the primary store for all user personal data. Region is selected at project creation; FORM maintains three project clusters:

| Cluster | Region |
|---|---|
| `form-prod-us` | `us-east-1` (US East) |
| `form-prod-eu-central` | `eu-central-1` (Frankfurt) |
| `form-prod-eu-west` | `eu-west-1` (Dublin) |

Tenant data is isolated via Row-Level Security (RLS). See `docs/DATA_MODEL.md` for full RLS policy specification. Tenant records are written to, and only readable from, the tenant's assigned cluster.

Supabase EU clusters use **AWS Frankfurt** (eu-central-1) and **AWS Ireland** (eu-west-1) as the underlying infrastructure. All data at rest is encrypted with AES-256 managed by AWS KMS. FORM can request customer-managed keys (BYOK) via Supabase's BYOK programme for contracts ≥ $250k ARR (roadmap item, §12).

### 4.2 Object Storage — Cloudflare R2

Cloudflare R2 stores exported data packages, audit log cold archives, and generated report PDFs. R2 jurisdiction hints are applied per tenant:

| Tenant region | R2 jurisdiction hint | Storage class |
|---|---|---|
| `us-east-1` | `default` (US) | R2 Standard |
| `eu-central-1` | `eu` | R2 Standard (EU) |
| `eu-west-1` | `eu` | R2 Standard (EU) |

The `eu` jurisdiction hint instructs Cloudflare to store object data exclusively in EU data centres (Amsterdam, Frankfurt, Paris, Warsaw nodes).

**Audit log cold archive:** After the hot retention window (90 days queryable in Postgres), audit log records are exported to R2 with the same jurisdiction hint as the tenant's primary region, preserving the data residency guarantee across the full 7-year retention period (SOC 2 / financial) or 3-year period (all other). See `docs/AUDIT_LOG_SCHEMA.md` for retention tiers.

### 4.3 Edge Compute — Cloudflare Workers

Cloudflare Workers handles the API layer. Workers execute at the nearest Cloudflare PoP to the end user — this is a **transit** layer, not a **storage** layer. Personal data passes through Workers in memory but is never persisted at the edge. Workers KV and Durable Objects are not used for personal data storage.

**Important distinction for EU customers:** Request processing may technically route through a non-EU PoP if a user travels outside the EU. FORM cannot guarantee that in-flight request processing occurs exclusively in EU data centres. What FORM guarantees is that **data at rest** (Supabase + R2) remains in the contracted region.

### 4.4 Identity — Auth0 / WorkOS (Enterprise SSO)

Enterprise SSO (SAML 2.0 / OIDC) is handled via Auth0 or WorkOS, depending on customer IdP preference. Both providers offer EU data regions:

| Provider | EU region option | Used for |
|---|---|---|
| Auth0 | EU (Frankfurt) tenant | Customers on Auth0 Enterprise Plan |
| WorkOS | EU region | Customers on WorkOS hosted connections |

For EU enterprise tenants, FORM provisions the SSO tenant in the EU region of the respective provider, configured during the SSO implementation phase (Day 7–14 of the onboarding timeline per `docs/ENTERPRISE.md`).

SSO session tokens (SAML assertions, OIDC ID tokens) are short-lived (15 minutes for SAML, 1 hour for OIDC access tokens). They are validated in-memory by the Workers API layer and not stored in the primary database. Audit log events for SSO actions are written to the tenant's region-resident audit log.

### 4.5 Audit Logs — DEC-030 HMAC Chain

Per **DEC-030**, FORM's audit log is tamper-evident via an HMAC chain (each record's `hmac_self` is computed over its content + the previous record's `hmac_self`). The audit log is region-resident by design:

- Audit log events are written to the `audit_log` table in the **tenant's assigned Supabase cluster** (same region as all other tenant data).
- The HMAC chain is verified hourly by a pg_cron job (`slo_chain_integrity_check`, job 60) running inside the tenant's cluster.
- Chain integrity alerts fire to the `#security-alerts` Slack channel if a gap or tampering is detected.
- Cold archival to R2 preserves the EU jurisdiction hint (§4.2) across the full retention window.

See `docs/AUDIT_LOG_SCHEMA.md` for schema, HMAC construction, retention tiers, and the `form_audit` role spec.

---

## 5. Sub-Processor Residency and Transfer Mechanisms

Several sub-processors are US-headquartered. For EU tenants, data transferred to US sub-processors is governed by Standard Contractual Clauses (SCCs) Module 2 (controller→processor) per GDPR Art. 46(2)(c). Full DPA status is maintained in `docs/SUBPROCESSORS.md`.

### 5.1 Anthropic (Victor AI coach)

| Attribute | Detail |
|---|---|
| Headquarters | San Francisco, CA, USA |
| Processing location | Anthropic US API endpoints |
| Data transferred | Pseudonymised exercise context (no PII in prompt payload — user_id replaced with random session token; per PA-05 in `docs/GDPR_DPIA.md`) |
| Transfer mechanism | SCC Module 2 (FORM as controller → Anthropic as processor) |
| Anthropic data retention | API data retained for abuse monitoring (30 days); not used for model training per API ToS |
| DPA status | Signed |

**Key constraint:** Victor AI prompt calls always route to Anthropic's US API. Even for EU tenants, coaching inference transits to the US. Mitigation: strict pseudonymisation — no PII in the prompt payload means the transferred data is not directly identifiable.

For customers where even pseudonymised health-context data transiting the US is unacceptable (e.g., German public sector), FORM can configure Victor in **local-only mode**: coaching responses are pre-generated from a finite plan library without live API calls. This feature is gated behind a tenant flag and requires Product approval (roadmap §12).

### 5.2 ElevenLabs (Voice synthesis)

| Attribute | Detail |
|---|---|
| Headquarters | New York, NY, USA |
| Data transferred | Coaching text cues (no user PII in payload — templated phrases only) |
| Transfer mechanism | SCC Module 2 |
| DPA status | Signed |

### 5.3 Cloudflare (CDN, Workers, R2, Pages)

| Attribute | Detail |
|---|---|
| Headquarters | San Francisco, CA, USA |
| EU data centre coverage | Amsterdam, Frankfurt, Paris, Warsaw, Dublin, London, Stockholm |
| R2 EU storage | Guaranteed via `eu` jurisdiction hint (§4.2) |
| Transfer mechanism | Cloudflare EU DPA + SCCs Module 2 |
| DPA status | Signed |

### 5.4 PostHog (Product analytics)

| Attribute | Detail |
|---|---|
| EU region option | PostHog EU Cloud (Frankfurt) |
| Configuration | EU tenants pointed to PostHog EU instance |
| PII handling | User IDs pseudonymised before ingestion; no direct identifiers in event properties |
| Transfer mechanism | EU hosting (no SCC needed); US fallback covered by SCC Module 2 |
| DPA status | Signed |

### 5.5 Sentry (Error monitoring)

| Attribute | Detail |
|---|---|
| EU region option | Sentry EU (Frankfurt) |
| Configuration | EU tenant error events routed to Sentry EU org |
| PII handling | SDK `beforeSend` scrubber removes PII before transmission; user IDs pseudonymised |
| Transfer mechanism | EU hosting for EU tenants |
| DPA status | In progress (PII scrubbing is compensating control in the interim) |

### 5.6 Better Stack (Log indexing / uptime)

| Attribute | Detail |
|---|---|
| EU region option | Better Stack EU (Amsterdam) |
| Configuration | EU log ingestion endpoint used |
| Data in scope | Structured application logs (no user PII in log payload by policy) |
| Transfer mechanism | EU hosting |
| DPA status | Signed |

---

## 6. Tenant Region Configuration

### 6.1 When region is set

Region is selected at **contract signing**, captured in the Order Form under "Data Residency Election." Options: `us-east-1`, `eu-central-1`, `eu-west-1`. Default is `us-east-1` if no election is made.

EU enterprise customers must make an explicit election. FORM's sales team must not default EU customers to `us-east-1` — this is a hard requirement enforced in the Order Form template.

### 6.2 How it is provisioned

During the implementation phase (Day 7–14 per `docs/ENTERPRISE.md` onboarding timeline):

1. DevOps provisions the tenant record in the correct Supabase cluster.
2. R2 bucket is created with the appropriate jurisdiction hint.
3. SSO provider tenant is provisioned in EU region (§4.4).
4. Analytics and logging sub-processors are pointed to EU endpoints.
5. `tenant_region` is set in the tenant config table; all Workers routes enforce correct cluster routing.

### 6.3 What customers can verify

At any QBR, FORM provides:
- The Supabase project region URL.
- R2 bucket jurisdiction configuration.
- A compliance-officer statement confirming no data migration to a non-contracted region has occurred since the previous attestation (§10).

### 6.4 Region migration

If a customer needs to change their contracted region:

- **Requires:** Contract Amendment re-executing the DPA with the new region.
- **Downtime window:** Estimated 2–4 hours of scheduled maintenance.
- **Cost:** Paid professional services engagement (rates per `docs/ENTERPRISE_SLA.md`).
- **Audit log continuity:** A migration event is written as the final record in the old chain. The new chain starts fresh. Both chains are archived in R2 with the appropriate jurisdiction hint.

---

## 7. Data Portability and Export

### 7.1 Individual user export (GDPR Art. 20)

Users export their own data via the FORM app (Settings → Privacy → Export My Data). Export includes: account profile, full workout history with form scores, meal logs, Victor coaching session summaries. Delivery: machine-readable JSON archive, encrypted with the user's account key, via in-app download link (valid 7 days). No personal data transits outside the tenant's contracted region for the export operation.

### 7.2 Tenant bulk export (enterprise admin)

Enterprise admins request a full tenant export via the Admin Dashboard or the Enterprise Admin API (`POST /admin/data-export`, see `docs/ENTERPRISE_ADMIN_API.md`). Export includes: all user records (excluding direct health data per privacy floor §11), aggregate wellness metrics history, full audit log export, seat provisioning and SCIM event history.

**Delivery:** Encrypted archive to a tenant-specified S3 or Azure Blob Storage endpoint. FORM does not store the export after delivery. Export generation occurs within the tenant's contracted region.

---

## 8. Data Deletion

### 8.1 Individual user deletion (GDPR Art. 17)

1. User submits deletion request in app (Settings → Privacy → Delete My Account).
2. 30-day waiting period (fraud prevention).
3. Day 30: all personal data hard-deleted from Supabase. Workout aggregate events pseudonymised (`DELETED_[hash]`) and retained for tenant-level aggregate metrics.
4. Audit log records retained per DEC-030 retention schedule with `actor_id` replaced by `DELETED_[hash]`.
5. Deletion confirmation email sent.

### 8.2 Tenant offboarding

| Day | Action |
|---|---|
| Day 0 | Contract terminates. |
| Day 1–30 | Grace period: data accessible for export. |
| Day 31 | Tenant marked `OFFBOARDED`. Active sessions invalidated. |
| Day 60 | All user personal data hard-deleted from primary cluster. |
| Day 90 | Audit logs cold-archived to R2 (tenant region, EU jurisdiction preserved). Encryption key delivered to tenant. |
| 7-year anniversary of last record | R2 cold archive deleted. Tenant notified 90 days in advance. |

FORM issues a **deletion certificate** on request — signed by the compliance-officer, confirming no data remains in FORM systems after Day 60.

### 8.3 What cannot be deleted

- **Billing records:** Stripe-owned; subject to Stripe's 7-year financial retention obligation.
- **Audit log aggregate counts:** Non-identifiable row counts used for SOC 2 evidence.
- **Legal hold:** If FORM is subject to a valid legal hold order, affected records cannot be deleted until the hold is released.

---

## 9. EU–US Transfer Mechanisms

| Transfer | Module | SCC version | Status |
|---|---|---|---|
| FORM → Anthropic | Module 2 (controller → processor) | EU SCCs 2021/914 | Signed |
| FORM → ElevenLabs | Module 2 | EU SCCs 2021/914 | Signed |
| FORM → Cloudflare (non-EU PoPs) | Module 2 | EU SCCs 2021/914 | Signed |
| FORM → PostHog (US fallback) | Module 2 | EU SCCs 2021/914 | Signed |
| FORM → Sentry (US fallback) | Module 2 | EU SCCs 2021/914 | In progress |

**Transfer Impact Assessments (TIAs):** TIAs conducted for Anthropic, ElevenLabs, and Cloudflare. Available to enterprise customers under NDA via the security data room (`docs/DATA_ROOM.md`).

**UK GDPR:** UK customers covered by the UK IDTA or UK Addendum to EU SCCs (FORM uses the UK Addendum approach). UK customers on `eu-west-1` additionally benefit from the EU–UK adequacy decision for EU → UK transfers (Art. 45).

**Swiss nDSG:** Swiss customers using `eu-central-1` are covered by the Swiss nDSG (in force September 2023). An nDSG-specific annex is appended to Swiss customer DPAs (pending external counsel review — OQ-DR-04).

---

## 10. Residency Attestation

| Attestation type | Frequency | Format |
|---|---|---|
| Quarterly compliance attestation | Quarterly | PDF letter signed by compliance-officer |
| Annual SOC 2 residency control evidence | Annual | SOC 2 Type II report (CC6.x, CC7.x controls), under NDA |
| On-request point-in-time attestation | On request (≤ 5 business days) | Signed letter + Supabase project region URL |
| Post-migration attestation | After any migration engagement | Signed letter confirming new region + deletion from old region |

---

## 11. Privacy Floor Interaction

FORM's non-negotiable privacy floor (`docs/ENTERPRISE.md`):

> _"HR can never see individual user data. HR sees aggregates only."_

This floor applies independently of tenant region:

- **Aggregate wellness metrics** served to the Admin Dashboard are computed from individual records within the tenant's region and returned as aggregated rows. Individual records never appear in Admin Dashboard API responses.
- A **k-anonymity floor of k ≥ 5** is applied before any aggregate is surfaced (under review for k ≥ 10 per EU EDPB guidance — OQ-DR-01).
- **Data export requests** from HR / People admins return only aggregate data, not individual user records.

The privacy floor is enforced at the API layer (Workers RLS enforcement + response filtering) and at the database layer (RLS policies), independently of region.

---

## 12. Roadmap

| Milestone | Target | Notes |
|---|---|---|
| **APAC region** (`ap-southeast-1`, Singapore) | Post-Series A | Japan, Australia, Singapore demand dependent |
| **EU entity** (Ireland or Netherlands) | Series A (~M24) | Eliminates SCC requirement for EU → EU sub-processors; simplifies contracts |
| **Customer-managed encryption keys (BYOK)** | M18–M24 | Supabase BYOK; enterprise-only; requires contract ≥ $250k ARR |
| **Multi-region replication** | Post-Series A | Active-active DR across `eu-central-1` and `eu-west-1` for EU tenants |
| **EUR billing** | Series A | Requires EU entity (DEC-096); USD-only until then |
| **Local-only Victor mode** | M18 | Pre-generated plan library; no Anthropic API transfer; for German public sector / healthcare-adjacent customers |

---

## 13. Open Questions

| ID | Question | Owner | Status |
|---|---|---|---|
| OQ-DR-01 | **k-anonymity floor:** Current floor is k ≥ 5. EDPB guidance and some EU DPAs expect k ≥ 10 for health data. Raise before first EU enterprise pilot? | compliance-officer + platform-engineer | In review |
| OQ-DR-02 | **Sentry DPA completion:** Sentry EU DPA not yet signed. PII scrubbing is a compensating control. When does formal completion unblock EU enterprise sales? | compliance-officer | Blocking for EU contracts > $50k ARR |
| OQ-DR-03 | **UK adequacy decision renewal:** EU–UK adequacy decision (June 2021) valid 4 years. If it lapses, UK customers on `eu-west-1` require IDTA. Monitoring required. | compliance-officer | Monitoring |
| OQ-DR-04 | **Swiss nDSG annex counsel review:** Draft Swiss DPA annex pending external Swiss counsel review. | compliance-officer + outside counsel | Pending |
| OQ-DR-05 | **Anthropic SCC TIA — US surveillance law:** Schrems II requires TIAs to assess FISA 702 / EO 12333 risk. Has FORM's Anthropic TIA been independently reviewed? | security-engineer | Pending external review |

---

**v1.0 · 2026-07-07 · compliance-officer + enterprise-architect + security-engineer**
