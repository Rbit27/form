# FORM · Enterprise Admin API v0.1

| Field | Value |
|---|---|
| **Base URL** | `https://api.form.coach/v1/admin` |
| **SCIM base URL** | `https://api.form.coach/scim/v2` |
| **Version** | v0.1 — pre-production spec; subject to change before GA |
| **Auth** | Bearer API key (`Authorization: Bearer <key>`) or `X-Form-Api-Key: <key>` |
| **Format** | JSON (UTF-8); `Content-Type: application/json` |
| **TZ convention** | All timestamps ISO 8601 UTC (`2026-07-01T14:00:00Z`) |
| **Owner** | `enterprise-architect` + `platform-engineer` |
| **Cross-references** | `docs/DATA_MODEL.md §17`, `docs/AUDIT_LOG_SCHEMA.md`, `docs/SSO_SCIM_IMPLEMENTATION.md`, `docs/ENTERPRISE_SLA.md`, `docs/ENTERPRISE_ONBOARDING.md` |
| **DEC-030** | Every mutating admin API call emits an HMAC-chained audit event (see §12) |

> **NOTICE.** This is a pre-production API specification authored for Founding Engineer hand-off and enterprise prospect security reviews. Endpoints marked `[DESIGNED]` are fully specified and will be implemented in Sprint 2–4 per `docs/PRODUCT_SPEC.md`. Breaking changes before v1.0 GA will be communicated with 30 days notice per `docs/ENTERPRISE_SLA.md §7`.

---

## Table of Contents

1. [Authentication & API Keys](#1-authentication--api-keys)
2. [RBAC: Roles and Permissions](#2-rbac-roles-and-permissions)
3. [Tenant Configuration](#3-tenant-configuration)
4. [SSO / IdP Management](#4-sso--idp-management)
5. [User Provisioning](#5-user-provisioning)
6. [SCIM v2](#6-scim-v2)
7. [Aggregate Reporting](#7-aggregate-reporting)
8. [Audit Log Export](#8-audit-log-export)
9. [Webhook Management](#9-webhook-management)
10. [Data Subject Requests (GDPR)](#10-data-subject-requests-gdpr)
11. [Rate Limits](#11-rate-limits)
12. [DEC-030 Event Reference](#12-dec-030-event-reference)
13. [Error Codes](#13-error-codes)
14. [Privacy Floor & Prohibited Operations](#14-privacy-floor--prohibited-operations)
15. [SOC 2 Mapping](#15-soc-2-mapping)

---

## 1. Authentication & API Keys

### 1.1 API Key format

FORM enterprise API keys are opaque 40-character strings with a `fk_live_` prefix for production and `fk_test_` for sandbox.

```
fk_live_a1b2c3d4e5f6a1b2c3d4e5f6a1b2c3d4e5f6a1b2
```

The last 6 characters (the "preview") are stored in the audit log and visible in the Admin Dashboard. The full key is shown **once at creation time only** and never returned by any GET endpoint. Store it in your secrets manager immediately.

### 1.2 Sending the key

```http
GET /v1/admin/tenant HTTP/1.1
Host: api.form.coach
Authorization: Bearer fk_live_<key>
```

Both `Authorization: Bearer <key>` and `X-Form-Api-Key: <key>` are accepted. The `Authorization` header takes precedence.

### 1.3 Key scopes

| Scope | Grants | Minimum role required |
|---|---|---|
| `read:reporting` | GET all `/v1/admin/reporting/*` endpoints | `tenant_admin` |
| `read:audit_log` | GET `/v1/admin/audit-log` | `tenant_admin` |
| `read:users` | List provisioned users, view SCIM groups | `tenant_admin` |
| `write:users` | Deprovision user, initiate GDPR erasure | `tenant_admin` |
| `read:sso` | GET SSO / OIDC config (secrets masked) | `tenant_owner` |
| `write:sso` | PUT SSO / OIDC config | `tenant_owner` |
| `write:webhooks` | Create, list, delete webhooks | `tenant_admin` |
| `write:api_keys` | Create, rotate, revoke API keys | `tenant_owner` |
| `export:data` | POST `/v1/admin/reporting/export`, GDPR export | `tenant_owner` |

When creating a key, pass the desired scope array. Omitting scopes creates a read-only key (`read:reporting`, `read:audit_log`, `read:users`).

### 1.4 Create API key [DESIGNED]

```http
POST /v1/admin/api-keys
Authorization: Bearer <owner-key>
Content-Type: application/json

{
  "label": "Okta SCIM Integration",
  "scopes": ["read:reporting", "read:users"],
  "ip_allowlist": ["203.0.113.0/24"],
  "expires_at": null
}
```

Response `201 Created`:
```json
{
  "id": "key_01J9ABCXYZ",
  "key": "fk_live_a1b2c3d4e5f6...",
  "label": "Okta SCIM Integration",
  "scopes": ["read:reporting", "read:users"],
  "preview": "...a1b2",
  "ip_allowlist": ["203.0.113.0/24"],
  "created_at": "2026-07-01T14:00:00Z",
  "expires_at": null
}
```

> The `key` field is shown **once only**. Subsequent GET requests return `"key": "***"`.

Emits DEC-030 event `api_key.created` (HIGH, 7-year retention).

### 1.5 List API keys [DESIGNED]

```http
GET /v1/admin/api-keys
```

Response: array of key objects with `"key": "***"`, ordered by `created_at DESC`.

### 1.6 Rotate API key [DESIGNED]

Rotation creates a new key and starts a **26-hour overlap window** during which both the old and new keys are valid. After 26 hours the old key is automatically revoked.

```http
POST /v1/admin/api-keys/{id}/rotate
Content-Type: application/json

{ "overlap_hours": 24 }
```

Response `200 OK`: new key object (same shape as create, `key` shown once).

Emits DEC-030 `api_key.rotated` (HIGH) and `api_key.revoked` on overlap expiry (HIGH).

If the old key is not revoked within 26 hours, alert `AL-APIKEY-02` fires to the tenant owner and FORM security-engineer. See `docs/AUDIT_LOG_SCHEMA.md §api_key`.

### 1.7 Revoke API key [DESIGNED]

```http
DELETE /v1/admin/api-keys/{id}
Content-Type: application/json

{ "reason": "compromised" }
```

`reason` enum: `manual` | `compromised` | `expiry_overlap`. Response `204 No Content`.

Emits DEC-030 `api_key.revoked` (HIGH). A `compromised` reason triggers immediate P1 incident alert to FORM security-engineer per `docs/INCIDENT_RESPONSE.md R-03`.

### 1.8 IP allowlisting

If `ip_allowlist` is set on a key, requests from IPs outside the CIDR list return `403 Forbidden`. The blocked attempt emits DEC-030 `api_key.ip_blocked` (bulk-deduplicated via Cloudflare Queues to prevent chain flooding). To enable/disable enforcement:

```http
PATCH /v1/admin/api-keys/{id}
{ "ip_enforcement_enabled": true }
```

Emits `api_key.ip_enforcement_enabled` or `api_key.ip_enforcement_disabled` (STANDARD, 7-year retention).

---

## 2. RBAC: Roles and Permissions

FORM enterprise uses four roles per tenant. Roles are assigned at provisioning time and can be updated via SCIM `roles` attribute or the Admin Dashboard.

| Role | Who | What they can do | What they cannot do |
|---|---|---|---|
| `tenant_owner` | IT lead, FORM CSM contact (max 2 per contract) | All admin operations: SSO config, API key management, billing, offboarding, GDPR export | No individual user health data under any circumstance |
| `tenant_admin` | HR / People Ops lead, IT admin | Aggregate reports, audit log read, user list, deprovision user, webhook management | SSO config, API key creation, individual health data |
| `tenant_manager` | Team leads (optional) | Same DB-level access as `tenant_member` — no admin API access | Everything admin |
| `tenant_member` | Regular employee | Own data only | Others' data, all admin operations |

**Role assignment via SCIM:** Map IdP groups to FORM roles using the `roles` attribute in the SCIM `/Groups` endpoint or the `role_map` JSONB column in `tenant_sso_configs`. Example: `{"cn=it-admins,dc=corp": "tenant_owner", "cn=hr-leads,dc=corp": "tenant_admin"}`. See `docs/SSO_SCIM_IMPLEMENTATION.md §4.3`.

**Role escalation:** Changing a user from `tenant_member` to `tenant_admin` or `tenant_owner` emits DEC-030 `admin.role_escalated` (HIGH, 7-year retention). This event triggers a quarterly access review check item. See `docs/SOC2_READINESS.md §23` (CC6 access review).

---

## 3. Tenant Configuration

### 3.1 Get tenant settings [DESIGNED]

```http
GET /v1/admin/tenant
```

Response `200 OK`:
```json
{
  "id": "ten_01J9ABC",
  "slug": "acme-corp",
  "display_name": "Acme Corporation",
  "tier": "enterprise",
  "seat_count": 500,
  "seats_provisioned": 423,
  "data_residency": "eu-central-1",
  "contract": {
    "starts_at": "2026-07-01",
    "ends_at": "2027-06-30",
    "annual_value_usd": 36000,
    "auto_renew": true
  },
  "features": {
    "sso_enabled": true,
    "scim_enabled": true,
    "white_label_enabled": false,
    "audit_log_webhook_enabled": true,
    "siem_export_enabled": false,
    "custom_sla_tier": "enterprise"
  },
  "privacy_floor_k": 5,
  "mfa_enforcement": "required",
  "session_max_hours": 24,
  "created_at": "2026-07-01T00:00:00Z"
}
```

### 3.2 Update tenant settings [DESIGNED]

Only `tenant_owner` role. Mutable fields: `display_name`, `mfa_enforcement`, `session_max_hours`. Contract and billing fields are read-only (managed by FORM).

```http
PATCH /v1/admin/tenant
Content-Type: application/json

{
  "mfa_enforcement": "required",
  "session_max_hours": 12
}
```

Response `200 OK`: updated tenant object.

Emits DEC-030 `admin.tenant_settings_updated` (STANDARD, 7-year retention) with before/after diff.

### 3.3 White-label configuration [DESIGNED] (Enterprise tier · ≥ $50k ARR)

White-label requires `tier = enterprise` and a signed Order Form with the white-label add-on. Setup is performed by FORM's enterprise-architect during onboarding.

```http
GET /v1/admin/tenant/white-label
```

```json
{
  "enabled": true,
  "custom_domain": "fit.acme.com",
  "cname_verified": true,
  "ssl_provisioned": true,
  "logo_url": "https://assets.form.coach/wl/acme-corp/logo-v2.png",
  "primary_color": "#005CA9",
  "wcag_aa_verified": true
}
```

Color changes are validated against WCAG 2.1 AA contrast ratios before applying. Failing validation returns `422` with specific contrast ratio data.

---

## 4. SSO / IdP Management

For full protocol details (SAML 2.0 metadata, OIDC discovery, SCIM token management), see `docs/SSO_SCIM_IMPLEMENTATION.md`. This section documents the REST API layer.

### 4.1 Get SSO configuration [DESIGNED]

```http
GET /v1/admin/sso
```

Response `200 OK` (SAML example):
```json
{
  "protocol": "saml2",
  "sp_entity_id": "https://api.form.coach/saml/acme-corp",
  "sp_acs_url": "https://api.form.coach/saml/acme-corp/acs",
  "sp_metadata_url": "https://api.form.coach/saml/acme-corp/metadata",
  "idp_entity_id": "https://dev-12345678.okta.com",
  "idp_sso_url": "https://dev-12345678.okta.com/app/form/sso/saml",
  "idp_certificate_preview": "MIIC...vQ==",
  "idp_certificate_expires_at": "2028-06-30T00:00:00Z",
  "attribute_mapping": {
    "email": "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress",
    "display_name": "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/displayname",
    "groups": "http://schemas.xmlsoap.org/claims/Group"
  },
  "role_map": {
    "form-it-admins": "tenant_owner",
    "form-hr-leads": "tenant_admin",
    "form-employees": "tenant_member"
  },
  "status": "active",
  "last_successful_login_at": "2026-06-10T09:15:00Z"
}
```

`idp_certificate_preview` returns only the first 12 and last 4 characters. The full certificate is never returned via this endpoint.

### 4.2 Configure SAML IdP [DESIGNED]

`tenant_owner` only. Provide either the IdP metadata XML URL or the individual fields.

```http
PUT /v1/admin/sso/saml
Content-Type: application/json

{
  "idp_metadata_url": "https://dev-12345678.okta.com/app/abc123/sso/saml/metadata",
  "role_map": {
    "form-it-admins": "tenant_owner",
    "form-hr-leads": "tenant_admin",
    "form-employees": "tenant_member"
  },
  "allow_idp_initiated": false
}
```

FORM fetches and validates the metadata URL, stores the certificate, and sets status to `pending_test`. The first successful SSO login advances status to `active`. While `pending_test`, existing password/Apple Sign In sessions remain valid.

Response `200 OK`: updated SSO config object.

Emits DEC-030 `admin.sso_config_updated` (HIGH, 7-year retention).

### 4.3 Configure OIDC IdP [DESIGNED]

```http
PUT /v1/admin/sso/oidc
Content-Type: application/json

{
  "discovery_url": "https://accounts.google.com/.well-known/openid-configuration",
  "client_id": "123456789-abc.apps.googleusercontent.com",
  "client_secret": "GOCSPX-...",
  "role_map": {
    "form-it@acme.com": "tenant_owner",
    "hr@acme.com": "tenant_admin"
  },
  "pkce_enabled": true
}
```

`client_secret` is AES-256-GCM encrypted on write using the tenant's KMS-derived key. The GET endpoint returns `"client_secret": "***"`. Never logged.

### 4.4 Test SSO configuration [DESIGNED]

Triggers a synthetic SP-initiated authentication flow from FORM's infrastructure and returns the result without creating a real session. Useful for validating IdP certificate rotation.

```http
POST /v1/admin/sso/test
```

Response `200 OK`:
```json
{
  "result": "success",
  "assertions_received": ["email", "display_name", "groups"],
  "role_resolved": "tenant_member",
  "latency_ms": 312,
  "tested_at": "2026-07-01T14:00:00Z"
}
```

### 4.5 IdP certificate rotation [DESIGNED]

When an IdP certificate is about to expire, FORM alerts the tenant owner at 60, 30, and 7 days via email + Slack Connect. To upload the new certificate (while keeping the old one valid during the transition):

```http
POST /v1/admin/sso/certificate/rotate
Content-Type: application/json

{
  "new_idp_certificate_pem": "-----BEGIN CERTIFICATE-----\n...",
  "overlap_days": 7
}
```

Both the old and new certificates are accepted during the overlap window. After `overlap_days`, the old certificate is rejected. See `docs/INCIDENT_RESPONSE.md R-04` for the full SSO/SAML certificate compromise runbook.

---

## 5. User Provisioning

Preferred provisioning path is SCIM v2 (§6). The REST endpoints below are for manual operations and administrative tooling.

### 5.1 List users [DESIGNED]

```http
GET /v1/admin/users?page=1&per_page=50&status=active
```

Query params: `page` (default 1), `per_page` (max 200), `status` (`active` | `suspended` | `all`), `scim_group` (filter by SCIM group slug).

Response `200 OK`:
```json
{
  "total": 423,
  "page": 1,
  "per_page": 50,
  "users": [
    {
      "id": "usr_01J9ABC",
      "email": "jane.smith@acme.com",
      "display_name": "Jane S.",
      "role": "tenant_member",
      "status": "active",
      "scim_groups": ["engineering", "london-office"],
      "provisioned_at": "2026-07-01T10:00:00Z",
      "last_active_at": "2026-07-10T08:30:00Z",
      "onboarding_completed": true
    }
  ]
}
```

**Privacy note:** `display_name` is truncated to first name + last initial to reduce identifier precision. Full name and email are returned because the admin must be able to identify which employee to deprovision — this is the only context where identity appears in the admin API.

### 5.2 Get user [DESIGNED]

```http
GET /v1/admin/users/{user_id}
```

Returns the same user object. No health data, workout data, or coaching content is included in this response under any circumstance. See §14.

### 5.3 Update user role [DESIGNED]

`tenant_owner` only.

```http
PATCH /v1/admin/users/{user_id}
Content-Type: application/json

{ "role": "tenant_admin" }
```

Emits DEC-030 `admin.role_escalated` (HIGH) if role is being elevated, or `admin.role_deescalated` (STANDARD) if being reduced.

### 5.4 Suspend user [DESIGNED]

Suspending a user invalidates all active sessions and prevents new logins without deleting data.

```http
POST /v1/admin/users/{user_id}/suspend
Content-Type: application/json

{ "reason": "security_investigation" }
```

`reason` enum: `offboarding` | `security_investigation` | `leave_of_absence` | `policy_violation`.

Emits DEC-030 `admin.user_suspended` (HIGH, 7-year retention).

### 5.5 Reactivate user [DESIGNED]

```http
POST /v1/admin/users/{user_id}/reactivate
```

Emits DEC-030 `admin.user_reactivated` (STANDARD).

### 5.6 Deprovision user [DESIGNED]

Deprovision removes the user from the tenant's seat pool. It does NOT delete health data immediately — that follows the GDPR erasure path (§10). Use SCIM DELETE for fully automated offboarding (see §6).

```http
DELETE /v1/admin/users/{user_id}
Content-Type: application/json

{ "reason": "employment_ended" }
```

`reason` enum: `employment_ended` | `transfer_to_other_tenant` | `gdpr_request`.

Response `204 No Content`.

Emits DEC-030 `admin.user_deprovisioned` (HIGH). If `reason = gdpr_request`, the erasure queue is also triggered (see §10.2).

---

## 6. SCIM v2

Full SCIM v2 protocol documentation, including provisioning flows, SCIM attribute mapping, and IdP-specific setup guides, is at `docs/SSO_SCIM_IMPLEMENTATION.md §3–§8` and `docs/SSO_CLIENT_CONFIG.md`.

**SCIM base URL:** `https://api.form.coach/scim/v2`

**Authentication:** Bearer token separate from the admin API key. Create via Admin Dashboard → Settings → SCIM. Tokens are stored as `SHA-256(token)` in `tenant_scim_tokens.token_hash` — the raw token is shown once at creation and never retrievable.

### 6.1 Supported SCIM endpoints

| Method | Path | Purpose |
|---|---|---|
| GET | `/Users` | List provisioned users with filter (`?filter=userName eq "jane@acme.com"`) |
| GET | `/Users/{id}` | Get SCIM user by FORM internal ID |
| POST | `/Users` | Provision new user (JIT — creates FORM account on first SSO login) |
| PUT | `/Users/{id}` | Full user replacement (all attributes) |
| PATCH | `/Users/{id}` | Partial update (e.g., update `active`, `roles`) |
| DELETE | `/Users/{id}` | Deprovision user; triggers GDPR erasure queue (30-day window per DPA) |
| GET | `/Groups` | List SCIM groups |
| POST | `/Groups` | Create group (maps to `scim_groups` for k-anonymity cohort reports) |
| PUT | `/Groups/{id}` | Full group replacement |
| PATCH | `/Groups/{id}` | Add/remove group members |
| DELETE | `/Groups/{id}` | Delete group |
| GET | `/ServiceProviderConfig` | Discover SCIM capabilities |
| GET | `/ResourceTypes` | Discover resource schemas |
| GET | `/Schemas` | Return User and Group attribute schemas |

### 6.2 FORM SCIM User schema extensions

```json
{
  "schemas": [
    "urn:ietf:params:scim:schemas:core:2.0:User",
    "urn:form:scim:schemas:extension:enterprise:2.0:User"
  ],
  "urn:form:scim:schemas:extension:enterprise:2.0:User": {
    "formRole": "tenant_admin",
    "department": "Engineering",
    "officeLocation": "London"
  }
}
```

`formRole` maps to FORM's RBAC roles: `tenant_owner` | `tenant_admin` | `tenant_member`. If absent, defaults to `tenant_member`.

### 6.3 Rate limits for SCIM

| Operation | Limit |
|---|---|
| Bulk user provision (POST `/Users`) | 100 requests / minute per tenant |
| User updates (PATCH, PUT) | 300 requests / minute per tenant |
| Reads (GET) | 600 requests / minute per tenant |

Exceeding limits returns `429 Too Many Requests` with `Retry-After` header.

---

## 7. Aggregate Reporting

All reporting endpoints read exclusively from pre-computed materialized views. No live queries against user data are permitted on this path. Privacy floor is enforced at three layers (database MV definition, Worker middleware, response validator). See `docs/DATA_MODEL.md §17` for full specification.

**k-anonymity floor:** Cohort groups with `user_count < 5` are suppressed. The response for a suppressed metric is:
```json
{ "suppressed": true, "reason": "k_anonymity_floor", "minimum_cohort_size": 5 }
```

**DPA gate:** `/v1/admin/reporting/*` returns `403 Forbidden` for tenants whose DPA is not executed (`dpa_ref IS NULL`). This is a GDPR Art. 28 enforcement control.

### 7.1 Wellness summary [DESIGNED]

```http
GET /v1/admin/reporting/wellness?weeks=13
```

Query params: `weeks` (1–52, default 13), `group` (SCIM group slug — filtered cohort, subject to k-floor).

Response `200 OK`:
```json
{
  "tenant_id": "ten_01J9ABC",
  "report_generated_at": "2026-07-10T04:00:00Z",
  "data_through": "2026-07-07",
  "cohort_size": 423,
  "weeks": [
    {
      "week_start": "2026-07-01",
      "active_users": 312,
      "total_sessions": 891,
      "avg_duration_min": 38.5,
      "median_duration_min": 35.0,
      "cv_adoption_pct": 64.2,
      "voice_coaching_adoption_pct": 41.8,
      "d30_retention_pct": 71.4,
      "avg_form_score": 7.2,
      "opted_in_users": 308
    }
  ]
}
```

`avg_form_score` is omitted if the CV cohort for that week has `user_count < 5`. `d30_retention_pct` omitted if the week's cohort is too small to calculate without individual identification.

### 7.2 Engagement metrics [DESIGNED]

```http
GET /v1/admin/reporting/engagement?weeks=4
```

Response includes WAU, MAU, activation rate (% of provisioned seats with completed onboarding), and WAU/MAU ratio (stickiness proxy). No individual user data.

### 7.3 Feature adoption [DESIGNED]

```http
GET /v1/admin/reporting/features?weeks=4
```

```json
{
  "weeks": [
    {
      "week_start": "2026-07-01",
      "cv_enabled_pct": 64.2,
      "voice_coaching_enabled_pct": 41.8,
      "wearable_connected_pct": 55.1,
      "program_assigned_pct": 88.3,
      "total_active_users": 312
    }
  ]
}
```

### 7.4 Cohort breakdown [DESIGNED]

Breaks down WAU/MAU by SCIM group. Useful for regional engagement analysis or department comparisons. Groups with fewer than 5 active members in the period are suppressed.

```http
GET /v1/admin/reporting/cohorts?week=2026-07-01
```

```json
{
  "week_start": "2026-07-01",
  "cohorts": [
    {
      "scim_group": "engineering",
      "group_size": 180,
      "active_users": 142,
      "wau_pct": 78.9
    },
    {
      "scim_group": "london-office",
      "group_size": 3,
      "suppressed": true,
      "reason": "k_anonymity_floor"
    }
  ]
}
```

### 7.5 CSV export [DESIGNED]

Exports all four aggregate views as a multi-sheet CSV archive. For GDPR data minimisation, CSV exports contain no user-identifiable columns.

```http
POST /v1/admin/reporting/export
Content-Type: application/json

{
  "date_from": "2026-01-01",
  "date_to": "2026-06-30",
  "include": ["wellness", "engagement", "features", "cohorts"]
}
```

Response `202 Accepted` with `Location: /v1/admin/reporting/export/{job_id}`. Poll for completion or receive webhook event `admin.export_completed`. Download URL expires in 24 hours.

Emits DEC-030 `admin.report_exported` (STANDARD, 3-year retention) with `row_count` and `suppressed_cohorts` count.

---

## 8. Audit Log Export

The audit log contains every privileged action performed in the tenant — SSO logins, admin API calls, user provisioning changes, data exports. All events are HMAC-chained per DEC-030. Full schema at `docs/AUDIT_LOG_SCHEMA.md`.

### 8.1 Pull audit log [DESIGNED]

```http
GET /v1/admin/audit-log?since=2026-07-01T00:00:00Z&until=2026-07-08T00:00:00Z&event_type=auth.&limit=500
```

Query params:
- `since` — ISO 8601 UTC (required)
- `until` — ISO 8601 UTC (default: now; max range 90 days per request)
- `event_type` — prefix filter (e.g., `auth.` for all auth events, `admin.` for admin actions, `api_key.` for key lifecycle)
- `limit` — max 1,000 per page (default 500)
- `cursor` — pagination cursor from previous response

Response `200 OK`:
```json
{
  "events": [
    {
      "id": "evt_01J9ABCXYZ",
      "event_type": "auth.sso_login_success",
      "severity": "STANDARD",
      "tenant_id": "ten_01J9ABC",
      "actor": "jane.smith@acme.com",
      "ip_hash": "sha256:7f3a...",
      "payload": {
        "idp": "okta",
        "role_resolved": "tenant_member",
        "session_id": "ses_01J9"
      },
      "chain_hash": "hmac_sha256:...",
      "key_version": "v1",
      "created_at": "2026-07-05T09:15:00Z"
    }
  ],
  "cursor": "eyJpZCI6...",
  "has_more": true,
  "total_in_range": 12842
}
```

**Chain integrity:** the `chain_hash` field is the HMAC-SHA256 of the event payload concatenated with the previous event's `chain_hash`. To verify chain integrity independently, use the HMAC chain verification endpoint (§8.3) or request FORM's daily verification artefact under NDA.

**Privacy note:** Auth events contain `ip_hash` (SHA-256 of client IP) not the raw IP. Health data fields are never present in audit events. See `docs/AUDIT_LOG_SCHEMA.md §privacy-guarantees`.

### 8.2 Export full audit log for period [DESIGNED]

For bulk export to your SIEM (Datadog, Splunk, Sumo Logic):

```http
POST /v1/admin/audit-log/export
Content-Type: application/json

{
  "since": "2026-01-01T00:00:00Z",
  "until": "2026-07-01T00:00:00Z",
  "format": "ndjson",
  "delivery": "s3",
  "s3_bucket": "acme-security-logs",
  "s3_prefix": "form-audit/2026-H1/",
  "s3_role_arn": "arn:aws:iam::123456789012:role/FormAuditExport"
}
```

`delivery` options: `s3` | `webhook` | `download`. For `download`, response is `202 Accepted` with polling URL.

Emits DEC-030 `admin.audit_log_exported` (HIGH, 7-year retention). Only `tenant_owner` with `export:data` scope can trigger.

### 8.3 Verify HMAC chain [DESIGNED]

```http
GET /v1/admin/audit-log/chain-status
```

```json
{
  "last_verified_at": "2026-07-10T03:00:00Z",
  "chain_status": "intact",
  "total_events": 84291,
  "oldest_event_at": "2026-07-01T00:00:00Z",
  "latest_event_at": "2026-07-10T02:59:47Z",
  "key_versions_in_chain": ["v1"]
}
```

`chain_status` values: `intact` | `break_detected` | `verification_in_progress`. A `break_detected` status triggers automatic P0 incident via `docs/INCIDENT_RESPONSE.md R-05`.

---

## 9. Webhook Management

Webhooks deliver audit log events in real time to your SIEM or security tooling. All webhook deliveries are signed; verify the signature before processing.

### 9.1 Create webhook [DESIGNED]

```http
POST /v1/admin/webhooks
Content-Type: application/json

{
  "url": "https://siem.acme.com/ingest/form",
  "label": "Splunk SIEM",
  "event_types": ["auth.*", "admin.*", "api_key.*"],
  "signing_secret": null
}
```

If `signing_secret` is null, FORM generates a 32-byte random secret. It is shown **once** in the response.

Response `201 Created`:
```json
{
  "id": "wh_01J9ABCXYZ",
  "url": "https://siem.acme.com/ingest/form",
  "label": "Splunk SIEM",
  "event_types": ["auth.*", "admin.*", "api_key.*"],
  "signing_secret": "whs_a1b2c3d4...",
  "status": "active",
  "created_at": "2026-07-01T14:00:00Z"
}
```

Emits DEC-030 `integration.webhook_created` (STANDARD, 7-year retention).

### 9.2 List webhooks [DESIGNED]

```http
GET /v1/admin/webhooks
```

Returns array. `signing_secret` is never included in list responses.

### 9.3 Delete webhook [DESIGNED]

```http
DELETE /v1/admin/webhooks/{id}
```

Response `204 No Content`. Emits DEC-030 `integration.webhook_deleted` (STANDARD).

### 9.4 Webhook payload format

```json
{
  "form_webhook_id": "wh_01J9ABCXYZ",
  "delivery_id": "del_01J9DEF",
  "event": {
    "id": "evt_01J9ABCXYZ",
    "event_type": "auth.sso_login_success",
    "tenant_id": "ten_01J9ABC",
    "created_at": "2026-07-05T09:15:00Z",
    "payload": { ... }
  },
  "delivered_at": "2026-07-05T09:15:02Z"
}
```

### 9.5 Signature verification

FORM signs every webhook payload with `HMAC-SHA256(signing_secret, raw_body)`. The signature is in the `X-Form-Signature-256` header as `sha256=<hex>`.

```typescript
import { createHmac } from 'node:crypto';

function verifyWebhook(rawBody: string, signature: string, secret: string): boolean {
  const expected = 'sha256=' + createHmac('sha256', secret)
    .update(rawBody, 'utf8')
    .digest('hex');
  return expected === signature;
}
```

Reject any request where the signature does not match. Do not process replay attacks: reject events with `delivered_at` more than 5 minutes in the past.

### 9.6 Retry policy

Failed deliveries (non-2xx response or timeout > 30s) are retried with exponential backoff: 5s → 30s → 5min → 30min → 2h → 8h. After 5 failed attempts the webhook is marked `degraded` and the tenant owner receives an email alert. After 48 hours in `degraded` state, the webhook is suspended.

---

## 10. Data Subject Requests (GDPR)

### 10.1 Get GDPR export for user [DESIGNED]

`tenant_owner` with `export:data` scope only. Initiates a full export of all personal data held for a specific user. The user receives an email notification when the export is ready.

```http
POST /v1/admin/users/{user_id}/gdpr/export
Content-Type: application/json

{
  "requester": "jane.smith@acme.com",
  "dsar_reference": "DSAR-2026-0042",
  "legal_basis": "art15_access_request"
}
```

Response `202 Accepted`:
```json
{
  "job_id": "dsar_01J9ABCXYZ",
  "user_id": "usr_01J9ABC",
  "status": "queued",
  "estimated_completion_at": "2026-07-11T14:00:00Z",
  "sla_deadline": "2026-08-10T14:00:00Z"
}
```

25-day PagerDuty alert fires if the export is not complete by `sla_deadline - 5d`. 30-day GDPR Art. 15 SLA is anchored to this `job_id`'s `created_at`. See `docs/SOC2_READINESS.md §70` (DSAR lifecycle automation).

Emits DEC-030 `data.export_initiated` (HIGH, 7-year retention).

### 10.2 Initiate erasure for user [DESIGNED]

`tenant_owner` with `write:users` scope only.

```http
POST /v1/admin/users/{user_id}/gdpr/erasure
Content-Type: application/json

{
  "requester": "jane.smith@acme.com",
  "dsar_reference": "DSAR-2026-0043",
  "legal_basis": "art17_right_to_erasure",
  "confirm": true
}
```

`"confirm": true` is required as a safeguard. Omitting it returns `400 Bad Request`.

The erasure schedule per `docs/DATA_MODEL.md §12`:
- Art. 9 data (CV keypoints, health profiles): **immediate nullification**, no grace period
- Standard personal data: 30-day soft-delete window, hard delete at day 30
- Audit log entries: **retained per retention schedule** — erasure does not remove audit log events (GDPR recital 65 allows retention for legal obligation and security evidence)
- Anonymised aggregate data in `tenant_wellness_summary_v2`: not erased (non-personal)

Response `202 Accepted` with erasure job ID. DEC-030 `data.erasure_initiated` (HIGH, 7-year retention).

### 10.3 Erasure status [DESIGNED]

```http
GET /v1/admin/gdpr/erasures/{job_id}
```

```json
{
  "job_id": "era_01J9ABCXYZ",
  "status": "completed",
  "phases": {
    "art9_nullified_at": "2026-07-10T14:01:00Z",
    "sessions_cleared_at": "2026-07-10T14:01:05Z",
    "soft_delete_at": "2026-07-10T14:01:05Z",
    "hard_delete_scheduled_at": "2026-08-09T14:01:05Z"
  },
  "confirmation_sent_to": "hr-privacy@acme.com"
}
```

---

## 11. Rate Limits

Rate limits apply per tenant API key. Limits are reset on a rolling 60-second window.

| Endpoint group | Limit | Burst |
|---|---|---|
| `GET /v1/admin/reporting/*` | 120 req / min | 20 |
| `POST /v1/admin/reporting/export` | 2 req / min | 2 |
| `GET /v1/admin/audit-log` | 60 req / min | 10 |
| `POST /v1/admin/audit-log/export` | 1 req / 10 min | 1 |
| `GET /v1/admin/users` | 60 req / min | 10 |
| `POST /v1/admin/users/*/suspend` | 30 req / min | 5 |
| `PUT /v1/admin/sso/*` | 10 req / min | 2 |
| `POST /v1/admin/api-keys` | 20 req / min | 5 |
| `SCIM /Users` reads | 600 req / min | 100 |
| `SCIM /Users` writes | 300 req / min | 50 |

Rate limit response headers on every request:
```
X-RateLimit-Limit: 120
X-RateLimit-Remaining: 87
X-RateLimit-Reset: 1688220060
```

On limit exceeded, `429 Too Many Requests` with `Retry-After: <seconds>`.

Enterprise tier customers may negotiate higher limits via `enterprise@form.coach` for specific integration needs (e.g., initial SCIM bulk sync). The negotiated limit increase is logged as DEC-030 `admin.rate_limit_exception_granted` (STANDARD, 7-year retention).

---

## 12. DEC-030 Event Reference

Every mutating admin API call and significant read operation emits a HMAC-chained audit event per `docs/AUDIT_LOG_SCHEMA.md`. The table below maps API operations to DEC-030 event types.

| API operation | DEC-030 event type | Severity | Retention |
|---|---|---|---|
| Create API key | `api_key.created` | HIGH | 7 years |
| Rotate API key | `api_key.rotated` | HIGH | 7 years |
| Revoke API key | `api_key.revoked` | HIGH | 7 years |
| Enable IP enforcement | `api_key.ip_enforcement_enabled` | STANDARD | 7 years |
| Disable IP enforcement | `api_key.ip_enforcement_disabled` | STANDARD | 7 years |
| IP blocked by allowlist | `api_key.ip_blocked` | STANDARD | 7 years |
| Update tenant settings | `admin.tenant_settings_updated` | STANDARD | 7 years |
| Update SSO config | `admin.sso_config_updated` | HIGH | 7 years |
| Escalate user role | `admin.role_escalated` | HIGH | 7 years |
| De-escalate user role | `admin.role_deescalated` | STANDARD | 7 years |
| Suspend user | `admin.user_suspended` | HIGH | 7 years |
| Reactivate user | `admin.user_reactivated` | STANDARD | 7 years |
| Deprovision user | `admin.user_deprovisioned` | HIGH | 7 years |
| Export aggregate report | `admin.report_exported` | STANDARD | 3 years |
| Export audit log | `admin.audit_log_exported` | HIGH | 7 years |
| Create webhook | `integration.webhook_created` | STANDARD | 7 years |
| Delete webhook | `integration.webhook_deleted` | STANDARD | 7 years |
| Initiate GDPR export | `data.export_initiated` | HIGH | 7 years |
| Initiate erasure | `data.erasure_initiated` | HIGH | 7 years |
| DPA accepted | `admin.dpa_accepted` | HIGH | 7 years |
| Webhook fired | `integration.webhook_fired` | STANDARD | 7 years |

Events are immutable after write. No FORM employee — including support staff — can delete or modify audit log entries. This is a DEC-030 invariant and a SOC 2 CC7.4 control. HMAC chain verification runs daily; a chain break triggers `docs/INCIDENT_RESPONSE.md R-05`.

---

## 13. Error Codes

All errors follow a consistent shape:

```json
{
  "error": {
    "code": "INSUFFICIENT_ROLE",
    "message": "This operation requires tenant_owner role. Current role: tenant_admin.",
    "request_id": "req_01J9ABCXYZ",
    "docs": "https://form.coach/docs/enterprise-admin-api#2-rbac"
  }
}
```

| HTTP status | Error code | Meaning |
|---|---|---|
| 400 | `INVALID_REQUEST` | Malformed JSON, missing required field, or invalid enum value |
| 400 | `CONFIRMATION_REQUIRED` | Destructive operation requires `"confirm": true` |
| 401 | `INVALID_API_KEY` | Key not found, revoked, or expired |
| 401 | `KEY_SCOPE_MISSING` | Key lacks the required scope for this endpoint |
| 403 | `INSUFFICIENT_ROLE` | Authenticated user's RBAC role cannot perform this action |
| 403 | `IP_NOT_ALLOWED` | Request IP not in key's allowlist |
| 403 | `DPA_REQUIRED` | Tenant DPA not executed; data processing endpoint blocked |
| 404 | `NOT_FOUND` | Resource ID does not exist or is not visible to this tenant |
| 409 | `CONFLICT` | Duplicate resource (e.g., webhook URL already registered) |
| 422 | `VALIDATION_FAILED` | Value passes type check but fails business rule (e.g., WCAG contrast < 4.5:1) |
| 429 | `RATE_LIMIT_EXCEEDED` | See §11. `Retry-After` header present. |
| 500 | `INTERNAL_ERROR` | FORM infrastructure error. Tracked in Sentry. `request_id` helps FORM support triage. |
| 503 | `MAINTENANCE` | Scheduled maintenance window. Check `form.statuspage.io`. |

---

## 14. Privacy Floor & Prohibited Operations

The privacy floor from `docs/DATA_MODEL.md §6` and `docs/ENTERPRISE_ONBOARDING.md §0` is enforced at the API layer as well as at the database layer. The following operations are **not available through the admin API under any circumstance**, regardless of tier, price, or contractual request:

| Prohibited operation | Enforcement |
|---|---|
| Read individual user's workout history | No route exists. `tenant_admin` role returns 0 rows from the `workouts` table by RLS policy. |
| Read individual user's form score | No route exists. Form scores appear only as aggregates (N ≥ 5). |
| Read coaching turn content | No route exists. `coaching_turns.content` is blocked at application layer for all admin roles. |
| Read individual user's health profile (goals, restrictions, BMI) | `health_profile_owner_only` RLS policy returns 0 rows for admin roles. |
| Read meal log data | `meal_logs` are excluded from all admin reporting queries. Clinical-safety veto, no exceptions. |
| Identify which users opted out of aggregates | Opted-out users are excluded silently from all MVs. The opt-out status is not exposed to admin. |
| Access body composition metrics (BMI, body fat) | Clinical-safety veto. Not in any reporting schema. |
| Access mental health signals from coaching | Clinical-safety veto. Not in any reporting schema. |

Attempting any of these operations programmatically (e.g., by directly constructing SCIM queries that imply individual health data) results in zero-row responses with no error, consistent with the "privacy by silence" principle. Admin role access to any prohibited category triggers DEC-030 `admin.consent_override_attempted` (CRITICAL, 7-year retention) and an immediate alert to FORM compliance-officer.

**Change process:** Adding any prohibited operation to the admin API requires:
1. Compliance-officer sign-off in writing
2. Clinical-safety review (for body composition, meal, and mental health categories)
3. DECISION_LOG.md entry
4. 30-day notice to all current enterprise customers

This is non-negotiable at any contract value.

---

## 15. SOC 2 Mapping

| SOC 2 criterion | Admin API control | Evidence artefact |
|---|---|---|
| **CC6.1 — Logical access restricted** | RBAC roles enforced at API layer; no individual health data route for admin roles; `/v1/admin/reporting/users` does not exist | DEC-030 `admin.consent_override_attempted` event; quarterly access review (CC6-QAR) |
| **CC6.2 — New access authorised** | API key creation requires `tenant_owner` role; DEC-030 `api_key.created` with creator identity | Audit log export for access-review cycle |
| **CC6.3 — Access removed** | Deprovision endpoint (`DELETE /v1/admin/users/{id}`); SCIM DELETE → erasure queue; DEC-030 `admin.user_deprovisioned` | Access review offboarding log (§65 in SOC2_READINESS) |
| **CC6.4 — Credential lifecycle** | API key rotation with 26-hour overlap; revocation on offboarding; DEC-030 `api_key.*` lifecycle events | `api_key.revoked` events with `reason: tenant_offboarding` filed as CC6-E-004 |
| **CC7.2 — Anomaly detection** | `api_key.ip_blocked` events bulk-ingested to SIEM; chain-break alert (R-05) | Alert rule AL-APIKEY-01 / AL-APIKEY-02 |
| **CC7.4 — Incident response** | Webhook delivery tracks `integration.webhook_fired`; failed deliveries alert tenant owner | Webhook retry log + DEC-030 chain evidence |
| **A1.1 — Availability capacity** | SCIM rate limits protect against provisioning storms; per-tenant quota enforced at Workers layer | Rate limit Cloudflare Analytics Engine metrics |
| **C1.1 — Confidentiality commitments** | OIDC/SAML secrets never returned in GET responses; API keys shown once only; `signing_secret` never in list responses | SSO config audit trail; DEC-030 `admin.sso_config_updated` |
| **P1.1 — Privacy notice** | DPA gate on `/v1/admin/reporting/*` (403 before DPA signed) | `admin.dpa_accepted` DEC-030 event |
| **P5.1 — DSAR intake** | `/v1/admin/users/{id}/gdpr/export` with 30-day SLA and PagerDuty alert | `data.export_initiated` DEC-030 event; DSAR job ID |
| **P4.3 — Disposal** | `/v1/admin/users/{id}/gdpr/erasure` with Art. 9 immediate nullification | `data.erasure_initiated` DEC-030 event |

---

## Implementation Notes

This document is the authoritative pre-production API specification for Founding Engineer hand-off. Key implementation guidance:

1. **All admin endpoints require a Cloudflare Workers JWT middleware** that resolves `app.current_role` and `app.current_tenant_id` from the API key, then passes these as Postgres `SET` calls before any query. See `docs/DATA_MODEL.md §1.2`.

2. **k-anonymity middleware** (`k-anonymity-guard.ts`) must be applied to every `/v1/admin/reporting/*` handler. See `docs/DATA_MODEL.md §17.4.2` for the `applyKAnonymityGuard` implementation.

3. **DEC-030 emission** is synchronous for HIGH-severity events (emission failure = request failure) and async for STANDARD events. This is the same invariant as `docs/AUDIT_LOG_SCHEMA.md §implementation-notes`.

4. **SCIM bearer tokens** are `SHA-256`-hashed before storage. The raw token is never persisted. The `form_scim_auth` Postgres function (SECURITY INVOKER) handles comparison to avoid handling the plaintext in application code. See `docs/DATA_MODEL.md §6.3`.

5. **SSO secrets** (`oidc_client_secret_enc`, `saml_idp_certificate`) are encrypted via `pgcrypto.encrypt` using the tenant's KMS-derived key. Never log, never return, never appear in `pg_dump`. See `docs/DATA_MODEL.md §4.3.1`.

6. **Privacy floor test suite** runs against every admin API endpoint on every CI deploy. Any endpoint returning user-identifiable data fails the CI gate. See `docs/DATA_MODEL.md §17.10`.

---

*v0.1 (2026-06-10): Initial specification — Enterprise Admin REST API reference. Covers: API key management (create/rotate/revoke/IP enforcement, DEC-030 lifecycle); RBAC (tenant_owner/admin/manager/member, role escalation events); tenant configuration (settings, white-label); SSO management (SAML 2.0, OIDC, certificate rotation); user provisioning (list/get/update-role/suspend/deprovision); SCIM v2 endpoint index (cross-references SSO_SCIM_IMPLEMENTATION.md); aggregate reporting endpoints (/wellness, /engagement, /features, /cohorts, /export — all privacy-floor enforced, k-anonymity N≥5, DPA gate); audit log pull API and HMAC chain verification; webhook management (create/list/delete, signature verification, retry policy); GDPR endpoints (export, erasure with Art. 9 immediate nullification, erasure status); rate limit table (12 endpoint groups); DEC-030 event reference (22 event types); error code table (11 codes); privacy floor enforcement table (8 prohibited operations, clinical-safety vetoes); SOC 2 mapping (CC6.1–CC6.4, CC7.2, CC7.4, A1.1, C1.1, P1.1, P5.1, P4.3); implementation notes for Founding Engineer (5 key items: Workers JWT middleware, k-anonymity guard, DEC-030 sync/async emission, SCIM token hashing, SSO secret encryption). All endpoints marked [DESIGNED] — pre-production spec. Cross-references: DATA_MODEL.md §17 (aggregate reporting schema), AUDIT_LOG_SCHEMA.md (DEC-030 event schema), SSO_SCIM_IMPLEMENTATION.md (protocol detail), ENTERPRISE_SLA.md (SLA tiers), ENTERPRISE_ONBOARDING.md (privacy floor, no-go use cases). Owner: enterprise-architect + platform-engineer.*
