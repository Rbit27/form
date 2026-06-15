# FORM · White-Label Implementation Guide

> Single-source reference for implementing, provisioning, monitoring, and revoking white-label custom domains and branding for enterprise tenants. This document ties together the schema (DATA_MODEL §20), observability (OBSERVABILITY §42), audit chain (AUDIT_LOG_SCHEMA §WhiteLabel), and business rules (ENTERPRISE.md §Branding) into one navigable guide.
>
> **Primary owners:** enterprise-architect · platform-engineer · devops-lead · compliance-officer
> **Status:** P0 implementation required before first white-label tenant activates (M10)
> **DEC:** DEC-054 (SHA-256 `custom_domain_hash` in DEC-030 payloads)

---

## Table of Contents

1. [TL;DR](#1-tldr)
2. [Eligibility and Entitlement Gate](#2-eligibility-and-entitlement-gate)
3. [Architecture Overview](#3-architecture-overview)
4. [Schema Reference](#4-schema-reference)
5. [Provisioning Workflow](#5-provisioning-workflow)
6. [DNS Setup Guide (IT Admin)](#6-dns-setup-guide-it-admin)
7. [SSL Certificate Lifecycle](#7-ssl-certificate-lifecycle)
8. [Branding Assets](#8-branding-assets)
9. [Admin Dashboard Panel (Tenant Admin)](#9-admin-dashboard-panel-tenant-admin)
10. [Revocation](#10-revocation)
11. [DEC-030 HMAC Chain Events](#11-dec-030-hmac-chain-events)
12. [SOC 2 Evidence Artefacts](#12-soc-2-evidence-artefacts)
13. [Privacy Constraints](#13-privacy-constraints)
14. [Open Questions](#14-open-questions)
15. [Implementation Checklist](#15-implementation-checklist)

---

## 1. TL;DR

White-label gives enterprise tenants at **≥ $50k ARR** a custom domain and tenant branding (logo, primary colour) in place of the FORM wordmark. The feature is:

- **Gated** by the `branding.white_label_enabled` feature flag on `tenant_feature_flags` + `tenants.white_label = TRUE`.
- **Provisioned manually** by a FORM platform-engineer under PAM escalation — tenant admins initiate via the Admin Dashboard panel; FORM completes the Cloudflare Custom Hostnames API call.
- **Monitored** via pg_cron job 32 (`white_label_cert_check`, 02:00 UTC daily) and six PagerDuty alert rules (AL-WL-01 through AL-WL-06).
- **Tamper-evident** via six DEC-030 HMAC-chained events (AUDIT_LOG_SCHEMA §WhiteLabel), with `custom_domain_hash` = SHA-256(`custom_domain`) in all payloads (DEC-054).
- **Auto-revoked** when tenant ARR falls below $50k; also manually revocable by platform-engineer (PAM-gated).

**"Powered by FORM" footer** is non-removable below $50k ARR; removal requires a signed contract amendment and a `form_admin` write to `tenant_branding.powered_by_removal`.

---

## 2. Eligibility and Entitlement Gate

### 2.1 ARR Threshold

White-label is available to tenants with a signed contract carrying `contract_arr ≥ $50,000`. The `contracts` table `contract_arr` column is the source of truth. ARR is checked:

1. **At provisioning time** — platform-engineer verifies `contract_arr ≥ 50000` before the Cloudflare API call (§5.2).
2. **Daily** — pg_cron job 35 (`white_label_eligibility_sweep`, 03:30 UTC) cross-checks `contract_arr` for all white-label tenants; triggers revocation workflow (§10.1) if ARR drops below threshold.

### 2.2 Feature Flag

| Flag key | Minimum tier | Table |
|---|---|---|
| `branding.white_label_enabled` | Enterprise (custom) | `tenant_feature_flags` |

The flag is registered in the `feature_flag_registry` seed data (DATA_MODEL §19.3). All branding write paths call `assertFeatureEnabled('branding.white_label_enabled')` before accepting a write.

The `tenants.white_label BOOLEAN` column is the **fast-path gate** in Worker entitlement checks — no JOIN required. A tenant_branding row without `tenants.white_label = TRUE` is an invariant violation detected by pg_cron consistency check (DATA_MODEL §20.10).

### 2.3 Powered-by-FORM Rule

| ARR | `powered_by_removal` writable to `TRUE`? |
|---|---|
| `< $50,000` | ❌ Never — `powered_by_removal` must be `FALSE` |
| `≥ $50,000` | ✅ Only by `form_admin` role (BYPASSRLS) with signed contract amendment |
| `≥ $50,000` + amendment | `powered_by_removal_ref` (DECISION_LOG entry ID) must be non-null |

---

## 3. Architecture Overview

```
Tenant IT admin sets up:
  coaching.acme.com  CNAME → form.coach
                            │
                     Cloudflare edge
                     Custom Hostnames
                            │
                     FORM Worker reads X-Tenant-Domain header
                     → resolves tenant_id via tenant_white_label_domains
                            │
              ┌─────────────┴──────────────┐
              │                            │
       Supabase RLS                 R2 (form-tenant-assets)
       tenant_branding row          tenant-assets/{tenant_id}/
       logo_url / colors            logo.png / logo_dark.png
       custom_domain_status         favicon.png
```

**Three surfaces monitored by §42 observability:**
1. Custom domain DNS (CNAME → Cloudflare) — Better Stack synthetic probes S-WL-{n}
2. SSL certificate lifecycle — Cloudflare Custom Hostnames API polled by pg_cron job 32
3. Branding asset CDN delivery — R2 P95 latency tracked as WL-SLO-04

---

## 4. Schema Reference

Two tables underpin white-label. Full DDL and RLS policies are defined in the canonical locations below; this section provides a navigation map.

### 4.1 `tenant_branding` (DATA_MODEL §20)

Stores the branding assets and configuration for white-label tenants.

| Column | Purpose |
|---|---|
| `logo_url` / `logo_dark_url` / `favicon_url` | R2 public URLs; written by asset upload API only |
| `primary_color` / `primary_color_contrast` | WCAG AA validated at application layer |
| `custom_domain` / `custom_domain_status` | FQDN + provisioning status (`not_configured` → `pending_dns` → `active` → `error`) |
| `powered_by_removal` | `form_admin` write only; requires `powered_by_removal_ref` |
| `email_sender_name` / `email_reply_to` | Transactional email branding |

**Full DDL:** DATA_MODEL §20.3  
**RLS policies:** DATA_MODEL §20.6  
**Migration:** `0071_tenant_branding.sql`

### 4.2 `tenant_white_label_domains` (OBSERVABILITY §42.6)

Stores the Cloudflare Custom Hostnames record and SSL certificate state for monitoring.

| Column | Purpose |
|---|---|
| `custom_domain` | FQDN (UNIQUE); cross-reference to `tenant_branding.custom_domain` |
| `cloudflare_hostname_id` | Cloudflare Custom Hostnames API ID for polling |
| `status` | `pending` → `active` → `error` → `revoked` |
| `ssl_cert_expiry_at` / `ssl_cert_status` | Updated nightly by pg_cron job 32 |
| `last_checked_at` | Stale detection threshold: > 26 hours triggers HIGH DEC-030 event |

**Full DDL:** OBSERVABILITY §42.6  
**RLS policies:** `tenant_admin` SELECT own; `form_system` ALL; `compliance_reviewer` SELECT all; `form_api` REVOKED  
**Migration:** `0072_tenant_white_label_domains.sql`

---

## 5. Provisioning Workflow

### 5.1 Tenant Admin Initiates

1. Tenant admin navigates to **Admin Dashboard → Domain & Branding** panel (§9).
2. Enters the desired CNAME target FQDN (e.g. `coaching.acme.com`).
3. Dashboard displays DNS instructions (§6) and sets `custom_domain_status = 'pending_dns'`.
4. Tenant admin configures DNS at their registrar (§6).
5. Dashboard shows validation status (polling `custom_domain_verified_at`).

### 5.2 Platform-Engineer PAM-Gated Activation

Once the CNAME is verified:

1. Platform-engineer escalates via PAM (`POST /internal/pam/escalate` → session UUID).
2. Verifies eligibility:
   - `contracts.contract_arr ≥ 50000` for the tenant.
   - `tenant_feature_flags.branding.white_label_enabled = TRUE`.
3. Calls Cloudflare Custom Hostnames API:
   ```
   POST /zones/{zone_id}/custom_hostnames
   {
     "hostname": "coaching.acme.com",
     "ssl": { "method": "http", "type": "dv", "settings": { "min_tls_version": "1.2" } }
   }
   ```
4. Records `cloudflare_hostname_id` and creates `tenant_white_label_domains` row with `status = 'active'`.
5. Updates `tenant_branding.custom_domain_status = 'active'` and sets `custom_domain_verified_at`.
6. Emits `tenant.white_label_provisioned` DEC-030 event (§11).
7. Configures Better Stack synthetic probe S-WL-{n} for the new domain (60-second interval, 2-consecutive-failure trigger → AL-WL-01).

**PAM session ID** is recorded in the `tenant.white_label_provisioned` payload as `provisioned_by_pam_session`.

### 5.3 Internal API Endpoint

```
POST /internal/enterprise/white-label/provision
Authorization: PAM-session {session_uuid}
Content-Type: application/json

{
  "tenant_id": "uuid",
  "custom_domain": "coaching.acme.com"
}
```

Response on success:
```json
{
  "status": "provisioned",
  "cloudflare_hostname_id": "...",
  "cert_status": "pending_validation",
  "chain_event_id": "uuid"
}
```

Worker returns HTTP 403 if PAM session is missing, 412 if eligibility check fails.

---

## 6. DNS Setup Guide (IT Admin)

Send this to the customer's IT admin during onboarding (Days 14–21 per ENTERPRISE.md implementation timeline).

---

**FORM White-Label DNS Setup**

To activate your custom domain, add the following DNS record at your domain registrar or DNS provider:

| Type | Host | Value | TTL |
|---|---|---|---|
| `CNAME` | `coaching` (or your chosen subdomain) | `form.coach` | 3600 |

**Notes:**
- The subdomain must be one level deep — `coaching.acme.com` is valid; `app.coaching.acme.com` is not.
- Propagation typically completes within 15 minutes; up to 24 hours in rare cases.
- FORM validates the CNAME via Cloudflare DNS resolution before SSL provisioning begins.
- **Do not proxy** the CNAME through your own CDN or Cloudflare Orange-Cloud — Cloudflare Custom Hostnames requires grey-cloud CNAME passthrough.

**SSL:** FORM provisions a DV certificate automatically via Cloudflare SSL for SaaS. No certificate upload is required on your end.

---

### 6.1 CNAME Validation

The admin dashboard polls CNAME status via the Cloudflare Custom Hostnames API. Possible states displayed to tenant admin:

| Status | Displayed as | Meaning |
|---|---|---|
| `pending_dns` | ⏳ Awaiting DNS | CNAME not yet resolved |
| `active` | ✅ Domain active | CNAME verified, SSL provisioned |
| `error` | ⚠️ DNS error | CNAME misconfigured; see error detail |

---

## 7. SSL Certificate Lifecycle

### 7.1 Auto-Renewal

SSL certificates for white-label domains are issued and automatically renewed by **Cloudflare SSL for SaaS**. FORM's responsibility is failure detection, not renewal initiation.

Cloudflare renews DV certificates ≥ 30 days before expiry. pg_cron job 32 (`white_label_cert_check`) detects renewal by comparing the polled `ssl_cert_expiry_at` from the Cloudflare API against the previously stored value in `tenant_white_label_domains`.

### 7.2 pg_cron Job 32 — `white_label_cert_check`

| Property | Value |
|---|---|
| Schedule | `0 2 * * *` (02:00 UTC daily — after Cloudflare renewal window) |
| Mechanism | Worker proxy (holds `CF_API_TOKEN` as Worker Secret) polls Cloudflare Custom Hostnames API per active domain |
| On each poll | Updates `ssl_cert_expiry_at`, `ssl_cert_status`, `last_checked_at`; emits `system.white_label_cert_checked` (STANDARD, 1yr — excluded from chain audit) |
| Stale threshold | `last_checked_at > 26h` → emits `system.white_label_cert_check_stale` (HIGH, 7yr) |

Full specification: OBSERVABILITY §42.7

### 7.3 Expiry Alert Escalation Ladder

| Days to expiry | Alert | Severity | Routing | Cooldown |
|---|---|---|---|---|
| < 30 days | AL-WL-03 | P2 | Slack `#infra-alerts` | 7 days per tenant |
| < 7 days | AL-WL-04 | P1 | PagerDuty `form-platform` dual-page | 24 hours |
| Expired (≤ 0) | AL-WL-05 | P0 | PagerDuty dual-page founder + devops-lead; no auto-resolve; CSM notification ≤ 15 min | — |

AL-WL-05 also opens a §23 SLA incident (WL-SLO-01 + WL-SLO-02 breach) via `sla_incident_id` FK.

### 7.4 SLOs

| SLO ID | Metric | Target | Window |
|---|---|---|---|
| WL-SLO-01 | Custom domain HTTPS availability (per tenant) | ≥ 99.9% | Calendar month |
| WL-SLO-02 | Zero expired certs on active tenants | 100% zero-tolerance | Continuous |
| WL-SLO-03 | Cert expiry warning ≥ 30 days before expiry | 100% zero-tolerance | Continuous |
| WL-SLO-04 | Branding asset CDN P95 latency | < 500 ms | 7-day rolling |

WL-SLO-01 is the **SLA-covered surface** — breach triggers credit calculation per ENTERPRISE_SLA §3.4.

---

## 8. Branding Assets

### 8.1 Asset Constraints

| Asset | Format | Max size | Constraint |
|---|---|---|---|
| `logo_url` (light) | PNG or WebP | 200 KB | No SVG — XSS risk on shared CDN origin |
| `logo_dark_url` (dark) | PNG or WebP | 200 KB | Fallback: `logo_url` if not provided |
| `favicon_url` | `.ico` or 32×32 PNG | 50 KB | — |
| `primary_color` | `#RRGGBB` hex | — | WCAG AA ≥ 4.5:1 contrast vs white enforced at API layer |

SVG is explicitly **not allowed** — see DATA_MODEL §20.2 for rationale.

### 8.2 R2 Storage Path

```
form-tenant-assets/
└── tenant-assets/
    └── {tenant_id}/
        ├── logo.png
        ├── logo_dark.png
        └── favicon.png
```

Bucket: `form-tenant-assets`. `form_api` has no access. `logo_url` in `tenant_branding` stores the R2 public URL; cache headers are set at the Cloudflare R2 bucket level.

Asset upload API validates:
1. MIME type matches extension.
2. File size within limit.
3. Path resolves to `tenant-assets/{request_tenant_id}/` — cross-tenant write blocked.

### 8.3 Color Palette

The admin dashboard API enforces WCAG AA contrast (≥ 4.5:1) for `primary_color` against `#ffffff` before writing. `primary_color_contrast` is auto-calculated as `#ffffff` or `#000000` (whichever achieves contrast) and stored in `tenant_branding`.

Color is applied to: buttons, active nav states, progress indicators. It does **not** override the global ink (`#0B0A07`) or paper (`#F3EEE0`) token — preventing full white-label from destabilising accessibility.

### 8.4 Powered-by-FORM Policy

| Condition | `powered_by_removal` | Footer behaviour |
|---|---|---|
| `contract_arr < $50,000` | `FALSE` (locked) | "Powered by FORM" footer visible |
| `contract_arr ≥ $50,000` + signed amendment | `TRUE` (form_admin write) | Footer hidden |

Write path: `form_admin` role (BYPASSRLS) only. `form_system` and `form_api` writes to `powered_by_removal = TRUE` return HTTP 403. The API requires `powered_by_removal_ref` (DECISION_LOG entry ID) to be non-null when setting `TRUE`.

### 8.5 Email Sender Branding

`tenant_branding.email_sender_name` overrides the sender display name in transactional emails:
> `Acme Fitness via FORM <noreply@form.coach>`

`email_reply_to` accepts an arbitrary email address (HR-team inbox). See OQ-BRAND-01 for outstanding question on SPF alignment for reply-to addresses.

---

## 9. Admin Dashboard Panel (Tenant Admin)

Panel visible when `tenants.white_label_enabled = TRUE` on the tenant feature flag.

**Displayed information:**

| Field | Detail |
|---|---|
| Domain status badge | `pending_dns` / `active` / `error` |
| SSL cert expiry | Countdown in days (< 30 days highlighted amber; < 7 days red) |
| CNAME setup guide | Inline DNS instructions from §6 |
| Logo upload | Drag-and-drop; format/size validated client-side and server-side |
| Primary colour | Hex picker with live WCAG contrast score |
| Powered-by-FORM toggle | Locked unless `form_admin` has set `powered_by_removal = TRUE` |
| Email sender name | Free-text field |

**Internal fleet view** (FORM DevOps only — Metabase "White-Label Fleet Health" dashboard, OBSERVABILITY §42.9):
- Active white-label tenant count
- Cert expiry timeline (60-day lookahead)
- Provisioning status breakdown
- pg_cron job 32 freshness indicator
- Recent AL-WL-01/04/05 incidents

Full admin panel spec: OBSERVABILITY §42.8

---

## 10. Revocation

### 10.1 Automated (ARR Threshold)

pg_cron job 35 (`white_label_eligibility_sweep`, 03:30 UTC daily):

1. Queries `tenants JOIN contracts` for `white_label = TRUE AND contract_arr < 50000`.
2. For each matched tenant:
   - Sets `tenant_white_label_domains.status = 'revoked'`.
   - Calls Cloudflare API to delete the Custom Hostname.
   - Sets `tenant_branding.custom_domain_status = 'not_configured'`.
   - Emits `tenant.white_label_revoked` DEC-030 event with `reason = 'downgrade'`.
   - Triggers CSM notification to inform the tenant.

### 10.2 Manual (PAM-Gated)

```
DELETE /internal/enterprise/white-label/{tenant_id}
Authorization: PAM-session {session_uuid}
Content-Type: application/json

{
  "reason": "customer_request" | "non_payment"
}
```

Platform-engineer:
1. Calls Cloudflare Custom Hostnames API `DELETE /zones/{zone_id}/custom_hostnames/{hostname_id}`.
2. Sets `tenant_white_label_domains.status = 'revoked'` and `revoked_at = now()`.
3. Emits `tenant.white_label_revoked` DEC-030 event with `reason` and `revoked_by_pam_session`.
4. Removes the Better Stack synthetic probe S-WL-{n} for the domain.

### 10.3 Post-Revocation

- `tenant_branding.custom_domain` is **not nulled** — retained as an audit record. `custom_domain_status` set to `'not_configured'`.
- DNS CNAME at customer's registrar can be removed by their IT admin — FORM has no control over this.
- If the tenant's ARR recovers above $50k, re-provisioning follows §5 from the start.

---

## 11. DEC-030 HMAC Chain Events

All six events are registered in **AUDIT_LOG_SCHEMA §WhiteLabel** (v2.8, 2026-06-14). Canonical Zod schemas are defined there; this section summarises the lifecycle events and their ordering invariant.

### 11.1 Event Summary

| Event | Severity | Retention | Emitter |
|---|---|---|---|
| `tenant.white_label_provisioned` | STANDARD | 7 yr | platform-engineer (PAM-gated) |
| `tenant.white_label_cert_expiry_warning` | HIGH | 7 yr | pg_cron job 32 (`form_system`) |
| `tenant.white_label_cert_renewed` | STANDARD | 7 yr | pg_cron job 32 (`form_system`) |
| `tenant.white_label_cert_expiry_breach` | **CRITICAL** | 7 yr | pg_cron job 32 (`form_system`) |
| `tenant.white_label_revoked` | HIGH | 7 yr | `form_system` (automated) or platform-engineer (PAM) |
| `system.white_label_cert_check_stale` | HIGH | 7 yr | pg_cron job 32 or `emit-audit-event` Worker (WL-CHAIN-01 violation) |

`system.white_label_cert_checked` (STANDARD, 1yr — per polled domain nightly) is **excluded** from the DEC-030 evidence set and from the weekly chain integrity audit.

### 11.2 WL-CHAIN-01 Ordering Invariant

```
tenant.white_label_provisioned
  └─→ tenant.white_label_cert_expiry_warning  (if expiry < 30d)
        └─→ tenant.white_label_cert_renewed    (on Cloudflare auto-renewal)
             OR
             tenant.white_label_cert_expiry_breach  (if cert expires)
  └─→ tenant.white_label_revoked              (on revocation)
```

**Invariant violations:**
- `tenant.white_label_cert_expiry_breach` without a preceding `tenant.white_label_cert_expiry_warning` within 30 calendar days → `emit-audit-event` Worker returns HTTP 422 (`WL_CHAIN_01_VIOLATION`) and co-emits `system.white_label_cert_check_stale`.
- `tenant.white_label_revoked` without a preceding `tenant.white_label_provisioned` for the same `tenant_id` → WARN (non-blocking; accommodates provisioning-failure-then-cleanup).

Any blocking WL-CHAIN-01 violation triggers INCIDENT_RESPONSE R-05.

### 11.3 Privacy in Payloads

Per DEC-054: `custom_domain` **never appears in plaintext** in any DEC-030 payload. All events use `custom_domain_hash = SHA-256(custom_domain)`. Plaintext `custom_domain` is retained in `tenant_white_label_domains` under RLS.

---

## 12. SOC 2 Evidence Artefacts

| Artefact | Trust Criterion | Content | Path | Retention |
|---|---|---|---|---|
| WL-E-001 | A1.1 — no active-tenant cert expired during observation | Annual export of `tenant.white_label_cert_renewed` chain events for all white-label tenants | `compliance/evidence/a1/WL-E-001_<YYYY>.csv` | 7 yr |
| WL-E-002 | CC7.2 — proactive monitoring demonstrated | AL-WL-03/04/05 PagerDuty config + 90-day warning history | `compliance/evidence/cc7/WL-E-002_<YYYY>.pdf` | 3 yr |
| WL-E-003 | CC7.2 — nightly monitoring demonstrated | pg_cron job 32 run history (180-day `system.white_label_cert_checked` event export) | `compliance/evidence/cc7/WL-E-003_<YYYY>.pdf` | 3 yr |

Collection trigger: after first full observation quarter with at least one active white-label tenant (P2, M13).

Cross-reference: SOC2_READINESS §A1.1, §CC7.2, §CC9.2 for control mapping.

---

## 13. Privacy Constraints

| Constraint | Enforcement |
|---|---|
| `custom_domain` is a commercially sensitive tenant identifier — not GDPR Art. 4(1) personal data but excluded from cross-tenant exports | RLS on `tenant_white_label_domains` (`form_api` REVOKED); Metabase schema filter excludes table from raw explorer |
| `custom_domain_hash` (SHA-256) only in DEC-030 payloads — no plaintext domain in HMAC chain | `emit-audit-event` Worker schema validation (DEC-054) |
| SIEM stream scoped per tenant — no white-label domain visible to other tenants' SIEM endpoints | OBSERVABILITY §27 tenant-scoped SIEM routing |
| `compliance_reviewer` access to `tenant_white_label_domains` disclosed in DPA Annex B | DPA template §14.3.2; update required before first EU enterprise DPA execution |
| `form_api` role has REVOKE ALL on `tenant_white_label_domains` | migration 0072 DDL |
| HR (`tenant_manager` role) cannot access white-label domain or branding config — HR sees aggregate wellness metrics only | RLS policy; Admin Dashboard "Domain & Branding" panel gated to `tenant_owner` + `tenant_admin` only |

**Privacy floor:** HR never sees individual user data. White-label configuration is owned by tenant IT/admin roles, not HR roles.

---

## 14. Open Questions

| ID | Question | Priority | Owner | Resolution path |
|---|---|---|---|---|
| **OQ-BRAND-01** | Should `email_reply_to` require a FORM-operated domain (e.g. `@acme.form.coach`) to prevent SPF misalignment and phishing risk? Currently accepts arbitrary email addresses. | Medium | compliance-officer + security-engineer | Resolve before first white-label transactional email send; DATA_MODEL §20.9 |
| **OQ-WL-OBS-02** | What is the Cloudflare Custom Hostnames API rate limit for nightly polling at scale? At ≤ 100 active white-label tenants, nightly polling is within 1,200 req/5-min limit. At 1,000+ tenants, parallel polling risks rate-limiting. | P2 | platform-engineer | Evaluate at 100 active white-label tenants; add exponential backoff with jitter to polling Worker; OBSERVABILITY §42.13 |
| **OQ-WL-MOB-01** | Single mobile app binary with dynamic branding (tenant logo + primary colour loaded from `tenant_branding` at login) vs custom mobile build per tenant? | P1 | enterprise-architect + platform-engineer | Likely single app for ≤ $250k ARR; custom build for top-tier; resolve before first mobile white-label activation |
| **OQ-WL-EMAIL-01** | Should transactional email sender domain be `@acme.form.coach` (FORM-provisioned subdomain) or `@form.coach` with display-name override? SPF/DKIM implications differ. | P2 | platform-engineer + devops-lead | Resolve before first white-label transactional email; requires Resend/Postmark custom domain setup if FORM-provisioned subdomain adopted |

---

## 15. Implementation Checklist

### P0 — Before first white-label tenant activates (M10)

| # | Task | Owner | Status |
|---|---|---|---|
| 1 | Register all six DEC-030 events in `docs/AUDIT_LOG_SCHEMA.md §WhiteLabel`; deploy to `emit-audit-event` Worker event registry | platform-engineer + compliance-officer | [ ] |
| 2 | Create `tenant_white_label_domains` table (migration `0072_tenant_white_label_domains.sql`) with DDL + RLS per OBSERVABILITY §42.6 | platform-engineer | [ ] |
| 3 | Create `tenant_branding` table (migration `0071_tenant_branding.sql`) with DDL + RLS per DATA_MODEL §20.3 | platform-engineer | [ ] |
| 4 | Implement pg_cron job 32 `white_label_cert_check` (OBSERVABILITY §42.7) + polling Worker proxy holding `CF_API_TOKEN` as Worker Secret; register in §12.6 pg_cron registry | devops-lead + platform-engineer | [ ] |
| 5 | Implement `POST /internal/enterprise/white-label/provision` PAM-gated endpoint (§5.3) | platform-engineer | [ ] |
| 6 | Configure AL-WL-04 (P1) and AL-WL-05 (P0) in PagerDuty `form-platform`; verify with synthetic test event before first provisioning | devops-lead | [ ] |
| 7 | Register `branding.white_label_enabled` in `feature_flag_registry` seed data (DATA_MODEL §19.3) | platform-engineer | [ ] |
| 8 | Implement pg_cron job 35 `white_label_eligibility_sweep` (§10.1) | platform-engineer + devops-lead | [ ] |
| 9 | Add nightly pg_cron invariant check at 04:00 UTC (DATA_MODEL §20.10) — CRITICAL DEC-030 `branding.invariant_violation` + P1 PagerDuty if any non-white-label tenant has `custom_domain IS NOT NULL` or `powered_by_removal = TRUE` | platform-engineer + devops-lead | [ ] |

### P1 — Before SOC 2 observation period (M11)

| # | Task | Owner | Status |
|---|---|---|---|
| 10 | Configure AL-WL-01 (P1), AL-WL-02 (P1), AL-WL-03 (P2) in PagerDuty / Slack `#infra-alerts`; verify with test events | devops-lead | [ ] |
| 11 | Build Admin Dashboard "Domain & Branding" panel (§9); gate behind `branding.white_label_enabled` feature flag | platform-engineer | [ ] |
| 12 | Register WL-SLO-01 through WL-SLO-04 in OBSERVABILITY §2.1 master SLO table; wire WL-SLO-01 into §23 SLA credit engine as covered service | devops-lead + compliance-officer | [ ] |
| 13 | Configure Better Stack synthetic probe S-WL-{n} for first white-label domain; document scripting procedure for future activations | devops-lead | [ ] |
| 14 | Implement `DELETE /internal/enterprise/white-label/{tenant_id}` PAM-gated revocation endpoint (§10.2) | platform-engineer | [ ] |
| 15 | Resolve OQ-BRAND-01 (email_reply_to SPF policy); update `tenant_branding` DDL if domain restriction adopted | compliance-officer + security-engineer | [ ] |
| 16 | Disclose `compliance_reviewer` access to `tenant_white_label_domains` in DPA Annex B template before first EU enterprise DPA execution | compliance-officer | [ ] |

### P2 — Before first SOC 2 evidence collection (M13)

| # | Task | Owner | Status |
|---|---|---|---|
| 17 | Build Metabase "White-Label Fleet Health" internal dashboard (OBSERVABILITY §42.9); configure schema filter to exclude `tenant_white_label_domains` from raw explorer | data-engineer | [ ] |
| 18 | Collect WL-E-001, WL-E-002, WL-E-003 artefacts after first full observation quarter with ≥ 1 active white-label tenant; file in `compliance/evidence/` paths from §12 | compliance-officer | [ ] |
| 19 | Evaluate OQ-WL-OBS-02 (polling rate limit) at 100 active white-label tenants; implement exponential backoff Worker if needed | platform-engineer | [ ] |
| 20 | Resolve OQ-WL-MOB-01 (single app vs custom build); document decision in DECISION_LOG | enterprise-architect + platform-engineer | [ ] |

---

## Cross-References

| Document | Sections |
|---|---|
| `docs/ENTERPRISE.md` | §Branding (feature spec: ≥ $50k ARR, CNAME, logo, colour override, powered-by-FORM) |
| `docs/DATA_MODEL.md` | §20 (tenant_branding DDL + RLS), §19 (feature flags), §12 (GDPR Art. 17 erasure — cascade) |
| `docs/OBSERVABILITY.md` | §42 (full observability spec: RED metrics, SLOs, alerts, pg_cron, dashboards, DEC-030 events) |
| `docs/AUDIT_LOG_SCHEMA.md` | §WhiteLabel (six DEC-030 HMAC-chain events, Zod schemas, WL-CHAIN-01 invariant) |
| `docs/ENTERPRISE_SLA.md` | §3 (WL-SLO-01 as SLA-covered service), §19.4 (offboarding bulk export) |
| `docs/SSO_SCIM_IMPLEMENTATION.md` | §4.5 (SSO custom domain flow — separate from white-label domain), §20 (SAML cert lifecycle, analogous alert ladder) |
| `docs/INCIDENT_RESPONSE.md` | R-05 (WL-CHAIN-01 + CERT-CHAIN-01 violation response procedure) |
| `docs/COST_MODEL.md` | §15.5 (white-label COGS), §8 (enterprise economics) |
| `docs/DECISION_LOG.md` | DEC-054 (SHA-256 `custom_domain_hash` in DEC-030 payloads — OQ-WL-OBS-01 resolved) |

---

*v1.0 (2026-06-15) — Initial specification. Ties together DATA_MODEL §20 (tenant_branding schema), OBSERVABILITY §42 (cert monitoring + SLOs + alerts), AUDIT_LOG_SCHEMA §WhiteLabel (six DEC-030 events), ENTERPRISE.md §Branding (business rules), and DEC-054 (SHA-256 domain hash) into a single implementation reference. Four open questions registered (OQ-BRAND-01, OQ-WL-OBS-02, OQ-WL-MOB-01, OQ-WL-EMAIL-01). Twenty-item implementation checklist: 9× P0/M10, 7× P1/M11, 4× P2/M13. Ownership: enterprise-architect · platform-engineer · devops-lead · compliance-officer.*
