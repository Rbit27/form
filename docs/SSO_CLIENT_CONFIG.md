# FORM · SSO Client Configuration Reference v1.0

> Owner: enterprise-architect + platform-engineer. Review: on any IdP addition, client upgrade, or quarterly.
> Scope: FORM's own OAuth2/OIDC/SAML client applications — how FORM registers with enterprise IdPs.
> Complementary to: `docs/SSO_SCIM_IMPLEMENTATION.md §2` (customer IT admin setup) and `§23` (CAE/SSF).
> References: DEC-030 (audit log), `docs/AUDIT_LOG_SCHEMA.md`, `docs/ENTERPRISE.md`, `docs/SSO_SCIM_IMPLEMENTATION.md`.

---

## Table of Contents

1. [Purpose and Scope](#1-purpose-and-scope)
2. [FORM Client Application Inventory](#2-form-client-application-inventory)
3. [OAuth2/OIDC Registration Parameters per IdP](#3-oauth2oidc-registration-parameters-per-idp)
4. [PKCE Enforcement](#4-pkce-enforcement)
5. [Scopes and Claims Requirements](#5-scopes-and-claims-requirements)
6. [Token Validation Rules](#6-token-validation-rules)
7. [JWKS Endpoint Caching Policy](#7-jwks-endpoint-caching-policy)
8. [Entra CAE — `client_capabilities=CP1` and `xms_cc`](#8-entra-cae--client_capabilitiescp1-and-xms_cc)
9. [Session Configuration Defaults and Overrides](#9-session-configuration-defaults-and-overrides)
10. [DEC-030 Audit Events](#10-dec-030-audit-events)
11. [Implementation Checklist](#11-implementation-checklist)
12. [Open Questions](#12-open-questions)

---

## 1. Purpose and Scope

`docs/SSO_SCIM_IMPLEMENTATION.md §2` documents what a **customer's IT admin** configures in their IdP portal (Okta console, Azure portal, Google Admin Console). This document covers the **other side of the same handshake**: what FORM registers and configures in FORM's own codebases and infrastructure when connecting to enterprise IdPs.

This reference is authoritative for:

- Which OAuth2/OIDC client applications FORM operates and their registration parameters
- Per-IdP parameter variations (scopes, `response_mode`, `client_authentication`, audience)
- PKCE implementation requirements across FORM clients
- The Entra Continuous Access Evaluation (CAE) `client_capabilities=CP1` flag (closes `SSO_SCIM_IMPLEMENTATION.md §23` checklist item 12)
- Token validation rules enforced by FORM's auth layer
- JWKS caching behaviour and staleness handling

This document does **not** cover:
- Customer IdP portal configuration steps → see `SSO_SCIM_IMPLEMENTATION.md §2`
- SCIM provisioning → see `SSO_SCIM_IMPLEMENTATION.md §3`
- Session revocation architecture → see `SSO_SCIM_IMPLEMENTATION.md §22`
- CAE event processing pipeline → see `SSO_SCIM_IMPLEMENTATION.md §23`

---

## 2. FORM Client Application Inventory

FORM registers **three distinct OAuth2/OIDC client applications** with each enterprise IdP. Each has different security requirements.

### 2.1 Client: `form-admin-dashboard` (Web SPA)

| Property | Value |
|---|---|
| Type | OAuth2 public client (SPA — no client secret) |
| Auth method | PKCE (`code_challenge_method=S256`) — no `client_secret` |
| Redirect URI | `https://admin.form.coach/auth/callback` |
| Post-logout URI | `https://admin.form.coach/auth/logout-complete` |
| Token storage | Memory only — tokens never written to `localStorage` or `sessionStorage` |
| FORM session cookie | `HttpOnly`, `Secure`, `SameSite=Strict`, path `/` |
| Entra CAE | `client_capabilities=CP1` — **required** (see §8) |
| Refresh token | Not requested from IdP — FORM's own server-side refresh token issued post-exchange |

Because `form-admin-dashboard` is a public client, **no client secret is stored in browser code**. The backend Cloudflare Worker (`form-sso-exchange`) performs the token exchange using the authorization code + PKCE code verifier. The SPA never holds the IdP access token or refresh token.

### 2.2 Client: `form-mobile-ios` (Mobile)

| Property | Value |
|---|---|
| Type | OAuth2 public client (native mobile) |
| Auth method | PKCE (`code_challenge_method=S256`) — no `client_secret` |
| Redirect URI | `com.form.coach://auth/callback` (custom scheme) + `https://form.coach/auth/ios-callback` (universal link) |
| Token storage | iOS Keychain (encrypted at rest via Secure Enclave where available) |
| Entra CAE | `client_capabilities=CP1` — **required** (see §8) |
| Auth flow | ASWebAuthenticationSession (in-app browser, no SFSafariViewController) |
| Refresh token | Not stored in iOS Keychain directly — FORM session JWT stored; refresh via FORM backend |

Mobile SSO is only available for enterprise tenants. Consumer iOS login uses Apple Sign In (`SSO_SCIM_IMPLEMENTATION.md §1` scope note).

### 2.3 Client: `form-sso-exchange` (Cloudflare Worker — Confidential)

| Property | Value |
|---|---|
| Type | OAuth2 confidential client (server-side Cloudflare Worker) |
| Auth method | `client_secret_post` for most IdPs; `private_key_jwt` for Okta with JWT auth policy |
| `client_id` | Stored in Cloudflare Workers Secret (`SSO_CLIENT_ID_{TENANT_ID}`) — never in code |
| `client_secret` | Stored in Cloudflare Workers Secret (`SSO_CLIENT_SECRET_{TENANT_ID}`) — never in code |
| Purpose | Authorization code exchange, token introspection, back-channel logout |
| PKCE | Receives `code_verifier` from SPA/mobile client; forwards in token exchange |
| Entra CAE | Not applicable — Worker is not an interactive client; does not acquire user tokens |

The Worker is the only component that holds the `client_secret`. The SPA and mobile clients are public clients with no secret.

---

## 3. OAuth2/OIDC Registration Parameters per IdP

### 3.1 Okta

**form-admin-dashboard (SPA)**

| Parameter | Value |
|---|---|
| Application type | SPA (Single-Page Application) |
| Grant type | `authorization_code` |
| PKCE | Required — Okta enforces PKCE for SPA type |
| Allowed redirect URI | `https://admin.form.coach/auth/callback` |
| Allowed origins | `https://admin.form.coach` |
| `response_mode` | `query` (code flow) |
| Token endpoint auth method | None (PKCE-only; no `client_secret` for SPA) |
| `id_token` signed algorithm | RS256 |
| Refresh token | Not requested (FORM manages its own session lifetime) |

**form-sso-exchange (Worker — confidential)**

| Parameter | Value |
|---|---|
| Application type | Web (confidential) |
| Grant type | `authorization_code` |
| Client auth method | `client_secret_post` (default) or `private_key_jwt` (if Okta org enforces JWT auth) |
| Token endpoint | `https://{okta_domain}/oauth2/v1/token` |
| Introspection endpoint | `https://{okta_domain}/oauth2/v1/introspect` (used for §22 token introspection during revocation) |
| `client_id` | Stored as `SSO_CLIENT_ID_{TENANT_ID}` Workers Secret |
| `client_secret` | Stored as `SSO_CLIENT_SECRET_{TENANT_ID}` Workers Secret |

**Okta CAE note:** Okta does not use the `client_capabilities=CP1` mechanism. Okta CAEP is configured via the SSF transmitter endpoint (see `SSO_SCIM_IMPLEMENTATION.md §23.6.2`).

---

### 3.2 Microsoft Azure AD / Entra ID

**form-admin-dashboard and form-mobile-ios (Public Clients)**

| Parameter | Value |
|---|---|
| Application type | Single-page application (SPA) / Mobile and desktop application |
| Grant type | `authorization_code` + PKCE |
| Redirect URI (SPA) | `https://admin.form.coach/auth/callback` |
| Redirect URI (mobile) | `com.form.coach://auth/callback` |
| Supported account types | Accounts in this organizational directory only (single-tenant) |
| API permissions | `openid`, `profile`, `email`, `User.Read` |
| Admin consent | Required for `User.Read` on behalf of org |
| Groups claim | App Registration manifest → `"groupMembershipClaims": "SecurityGroup"` |
| `client_capabilities` | `["CP1"]` — **required for CAE; see §8** |
| `response_mode` | `fragment` for SPA; `query` for mobile |

**form-sso-exchange (Worker — confidential)**

| Parameter | Value |
|---|---|
| Application type | Web (confidential client) |
| Grant type | `authorization_code` |
| Client auth method | `client_secret_post` |
| Token endpoint | `https://login.microsoftonline.com/{entra_tenant_id}/oauth2/v2.0/token` |
| `client_id` | `SSO_CLIENT_ID_{TENANT_ID}` Workers Secret |
| `client_secret` | `SSO_CLIENT_SECRET_{TENANT_ID}` Workers Secret |
| CAEP receiver | `https://api.form.coach/enterprise/v1/sso/caep-receiver/{tenant_id}` (registered as notification endpoint in Microsoft Graph) |

---

### 3.3 Google Workspace

**form-admin-dashboard and form-mobile-ios (Public Clients)**

| Parameter | Value |
|---|---|
| Application type | Web application / Android / iOS |
| Grant type | `authorization_code` + PKCE |
| Redirect URI | `https://admin.form.coach/auth/callback` / `com.form.coach://auth/callback` |
| OAuth 2.0 scopes | `openid email profile` |
| `hd` parameter | Sent in auth request: `hd={customer_domain}` — restricts login to org domain |
| `access_type` | `online` (no offline refresh tokens requested from Google) |
| `prompt` | `select_account` on first login; `none` on silent renewal attempts |

**form-sso-exchange (Worker — confidential)**

| Parameter | Value |
|---|---|
| Application type | Web (server-side) |
| Token endpoint | `https://oauth2.googleapis.com/token` |
| `client_id` | `SSO_CLIENT_ID_{TENANT_ID}` Workers Secret |
| `client_secret` | `SSO_CLIENT_SECRET_{TENANT_ID}` Workers Secret |
| Google RISC receiver | `https://api.form.coach/enterprise/v1/sso/caep-receiver/{tenant_id}` (see `SSO_SCIM_IMPLEMENTATION.md §23.6.3`) |

**Google service account (for group sync only):**

When a Google Workspace tenant requires group-based role mapping (`SSO_SCIM_IMPLEMENTATION.md §21`), a separate service account is created per tenant with domain-wide delegation. This is **not** an OAuth2 client registration — it is a separate GCP service account with impersonation scope `https://www.googleapis.com/auth/admin.directory.group.readonly`. The service account JSON is stored encrypted in `tenant_sso_configs.google_service_account_json` (column-level AES-256-GCM, key from `KEYPOINTS_ENC_KEY`). This must not be confused with the OIDC client credentials above.

---

### 3.4 Generic SAML 2.0 SP

FORM acts as a SAML 2.0 Service Provider. The SP-side configuration is tenant-scoped:

| Parameter | Value |
|---|---|
| SP Entity ID | `https://form.coach/saml/{tenant_id}` |
| ACS URL | `https://form.coach/auth/saml/{tenant_id}/acs` |
| SP metadata URL | `https://form.coach/auth/saml/{tenant_id}/metadata` (served dynamically, auto-refreshable by IdP) |
| SLO URL | `https://form.coach/auth/saml/{tenant_id}/slo` |
| NameID policy | `urn:oasis:names:tc:SAML:1.1:nameid-format:emailAddress` |
| Signature algorithm | `RSA-SHA256` (FORM's SP signing key; rotated annually per `SSO_SCIM_IMPLEMENTATION.md §20`) |
| Digest algorithm | `SHA-256` |
| SP signing key | Stored in `SAML_SP_PRIVATE_KEY_{TENANT_ID}` Workers Secret; public cert exposed at metadata URL |
| Response binding | HTTP-POST |
| IdP-initiated | Disabled by default; configurable per tenant via `tenant_sso_configs.allow_idp_initiated` |

---

### 3.5 Generic OIDC

For any OIDC-compliant IdP not in §3.1–3.3:

| Parameter | Value |
|---|---|
| Discovery URL | Must provide `/.well-known/openid-configuration` |
| Grant type | `authorization_code` + PKCE |
| Client auth method | `client_secret_post` (default); `private_key_jwt` on request |
| Minimum scopes | `openid email profile` |
| `response_type` | `code` |
| `response_mode` | `query` |
| `id_token` algorithm | RS256 preferred; PS256 accepted; HS256 **rejected** (symmetric, not suitable for distributed verification) |

---

## 4. PKCE Enforcement

PKCE is mandatory for all FORM OAuth2/OIDC flows. No exceptions.

| Component | PKCE role |
|---|---|
| `form-admin-dashboard` (SPA) | Generates `code_verifier` (32 bytes, crypto.getRandomValues); computes `code_challenge = base64url(SHA-256(code_verifier))`; sends `code_challenge` + `code_challenge_method=S256` in auth request; forwards `code_verifier` to Worker on callback |
| `form-mobile-ios` (mobile) | Same as SPA; uses iOS `SecRandomCopyBytes` for `code_verifier` generation; ASWebAuthenticationSession handles redirect |
| `form-sso-exchange` (Worker) | Receives `code_verifier` from SPA/mobile in the callback POST; forwards it with `code` to IdP token endpoint; verifies IdP returns tokens only when `code_verifier` matches |

**Rejection policy:** If the IdP token response returns an access token without the PKCE exchange having been completed (e.g., `code_verifier` was absent or mismatched), the Worker MUST:
1. Discard the token response
2. Emit `sso.login_failed` DEC-030 event with `failure_reason: "pkce_mismatch"`
3. Return `401` to the browser/mobile

HS256 `client_secret_basic` flows (no PKCE) are not permitted. This is a hard platform constraint enforced at the Worker level.

---

## 5. Scopes and Claims Requirements

### 5.1 Minimum Required Scopes

| IdP | Scopes |
|---|---|
| Okta | `openid profile email groups` |
| Entra | `openid profile email User.Read` |
| Google Workspace | `openid email profile` |
| Generic OIDC | `openid email profile` (+ IdP-specific groups scope if available) |

### 5.2 Required Claims in ID Token

| Claim | Required | Notes |
|---|---|---|
| `sub` | Yes | Stable user identifier; stored as `external_user_id` in `tenant_users` |
| `email` | Yes | Primary identity claim |
| `email_verified` | Conditional | If present, must be `true`; absence is tolerated but logged as `WARN` |
| `given_name` | Yes | Used for display; falls back to `name` if absent |
| `family_name` | Yes | Same fallback |
| `groups` (Okta/Entra) | Conditional | Required when `tenant_sso_configs.role_mapping_enabled = true` |
| `hd` (Google) | Required for Google | Must match `tenant_sso_configs.allowed_domain`; missing or mismatched `hd` → reject |
| `oid` (Entra) | Preferred over `sub` | Entra `sub` is not stable across tenants; use `oid` as `external_user_id` |
| `xms_cc` (Entra) | CAE | Must contain `{"cp1": true}` when `client_capabilities=CP1` sent; see §8 |

### 5.3 Claims FORM Never Stores in Audit Log

Privacy floor — hardcoded in the auth handler, not configurable:
- Raw `access_token` or `refresh_token` values
- `password` or credential hints
- MFA method details
- Personal health identifiers from the IdP

---

## 6. Token Validation Rules

These rules are enforced by `form-sso-exchange` on every ID token received from any IdP. Validation failure → reject the response, emit `sso.login_failed`, return `401`.

| Rule | Detail |
|---|---|
| Signature | Verify against IdP's JWKS endpoint; `kid` header must match a key in the JWKS document |
| Algorithm | RS256 or PS256 only; HS256 → reject |
| `iss` | Must exactly match the configured issuer for this `tenant_id` in `tenant_sso_configs.oidc_issuer` |
| `aud` | Must contain FORM's `client_id` for this tenant |
| `exp` | Must be in the future; clock skew tolerance: ±60 seconds |
| `iat` | Must be within the past 10 minutes (prevents replay of old tokens) |
| `nonce` | Must match the nonce stored in the PKCE state cookie from step 1 of the auth flow; absent nonce → reject |
| `email_verified` | See §5.2; configurable to enforce via `tenant_sso_configs.require_email_verified` |
| `hd` (Google only) | Must equal `tenant_sso_configs.allowed_domain`; absent or mismatched → reject |
| `oid` vs `sub` (Entra) | Prefer `oid`; if absent fall back to `sub`; log `WARN` if `sub` used (less stable) |

**JWKS kid miss handling:** If the `kid` in the ID token header is not found in the cached JWKS document, the Worker fetches a fresh copy of the JWKS endpoint (bypassing cache). If the key is still not found after a fresh fetch, emit `sso.jwks_key_miss` (HIGH severity) and reject the token. This prevents JWKS cache poisoning (a stale cache could allow verification with a retired key).

---

## 7. JWKS Endpoint Caching Policy

FORM caches JWKS documents in `SSO_KV` (Cloudflare KV) to avoid fetching on every token validation (which would be high-latency and risk rate limiting by the IdP).

| Parameter | Value | Rationale |
|---|---|---|
| Cache TTL (normal) | 1 hour | Key rotation events are rare; 1-hour staleness is acceptable |
| Cache TTL (forced refresh) | Immediate | On `kid` miss (see §6) |
| Cache key | `jwks:{tenant_id}:{iss_hash}` | Tenant-scoped; prevents cross-tenant cache collisions |
| Stale-while-revalidate | Not used | Prefer synchronous refresh on miss; JWKS fetch adds ~50ms P95 latency acceptable |
| Lock on refresh | 5-second Durable Object lock | Prevents thundering herd on cache miss for same tenant (parallel requests wait for first fetch) |
| Error handling | If JWKS fetch fails: serve stale (if < 6h old) + emit `sso.jwks_fetch_failed` (HIGH) |

**SOC 2 evidence:** The `sso.jwks_key_miss` and `sso.jwks_fetch_failed` events are CC7.2 monitoring evidence.

---

## 8. Entra CAE — `client_capabilities=CP1` and `xms_cc`

This section closes `SSO_SCIM_IMPLEMENTATION.md §23` implementation checklist item 12.

### 8.1 What CP1 Does

Microsoft Entra Continuous Access Evaluation (CAE) allows the IdP to push real-time risk signals to FORM (account disabled, credential change, high-risk sign-in) and have FORM honour them within seconds rather than waiting for the access token to expire (typically 60–75 minutes without CAE).

When FORM's clients include `client_capabilities=["CP1"]` in the OAuth2 token request, Entra knows that FORM can process CAE revocation signals. Entra then:
1. Issues **CAE-capable tokens** — these tokens have longer lifetimes (up to 28 hours) because Entra trusts FORM to revoke them on signal
2. Includes the `xms_cc: {"cp1": true}` claim in the issued access token as confirmation

Without CP1, Entra may issue standard shorter-lived tokens without CAE coverage. Omitting CP1 reduces the security value of the §23 CAE integration.

### 8.2 How to Add CP1 to FORM Clients

**`form-admin-dashboard` SPA (MSAL.js configuration)**

```typescript
import { PublicClientApplication, Configuration } from "@azure/msal-browser";

const msalConfig: Configuration = {
  auth: {
    clientId: process.env.ENTRA_CLIENT_ID,
    authority: `https://login.microsoftonline.com/${tenantId}`,
    redirectUri: "https://admin.form.coach/auth/callback",
    clientCapabilities: ["CP1"],   // ← CAE capability declaration
  },
  cache: {
    cacheLocation: "memory",       // NEVER sessionStorage or localStorage
    storeAuthStateInCookie: false,
  },
};

const msalInstance = new PublicClientApplication(msalConfig);
```

**`form-mobile-ios` (MSAL.objc / Swift)**

```swift
let config = MSALPublicClientApplicationConfig(
    clientId: entraTenantClientId,
    redirectUri: "com.form.coach://auth/callback",
    authority: try MSALAADAuthority(url: authorityURL)
)
config.clientApplicationCapabilities = ["CP1"]   // ← CAE capability declaration

let application = try MSALPublicClientApplication(configuration: config)
```

**`form-sso-exchange` Cloudflare Worker (token request parameters)**

The Worker itself does not declare CP1 — it is not an interactive client. However, when forwarding the authorization code exchange to Entra's token endpoint, the Worker must NOT strip the CP1-related parameters. The CP1 declaration is part of the **auth request** sent by the SPA or mobile client in step 1 of the auth flow; it is not re-sent in the token exchange.

**Verification on staging:**

After deploying CP1 configuration:
1. Complete an Entra OIDC login flow in staging
2. Decode the access token returned by Entra (base64url decode the payload)
3. Confirm `xms_cc: {"cp1": true}` claim is present
4. If `xms_cc` is absent: re-check that `clientCapabilities: ["CP1"]` is in the MSAL config and that the Entra app registration has CAE enabled under **Authentication → Advanced settings**
5. Record the decoded token (with PII fields redacted) as evidence artifact `CC6-E-CAE-CP1-001`

### 8.3 CAE Claim Interaction with FORM Session Layer

When FORM's auth middleware validates an Entra access token and detects `xms_cc: {"cp1": true}`, it sets a flag in the session record:
```
session_store: caep_enabled = true, idp = "entra"
```

This flag gates the CAE revocation path in `SSO_SCIM_IMPLEMENTATION.md §22`'s `isRevoked()` function — only sessions with `caep_enabled = true` receive CAE-sourced revocation status from the CAEP processor.

### 8.4 Entra App Registration — CAE Prerequisite

In the Entra App Registration for `form-admin-dashboard` and `form-mobile-ios`:
1. Navigate to **Authentication → Advanced settings**
2. Confirm **"Allow public client flows"** is set appropriately (enabled for mobile, disabled for SPA)
3. Confirm **"Continuous access evaluation"** is set to **"Strict enforcement"** (not "Disabled")

If CAE strict enforcement is disabled in the app registration, Entra will not issue CAE tokens even if the client sends CP1. This is a customer IT admin action documented in `SSO_SCIM_IMPLEMENTATION.md §2.2`.

---

## 9. Session Configuration Defaults and Overrides

These are FORM's session parameters as configured in `tenant_sso_configs`. They interact with but are separate from the IdP token lifetimes.

| Parameter | Default | Min | Max | Notes |
|---|---|---|---|---|
| `session_max_duration_hours` | 8 | 1 | 24 | FORM session JWT TTL; independent of IdP token lifetime |
| `refresh_token_max_days` | 30 | 7 | 90 | FORM refresh token TTL; rotated on use |
| `require_email_verified` | `false` | — | — | Enforce `email_verified: true` claim |
| `allow_idp_initiated` | `false` | — | — | SAML IdP-initiated flows; disabled by default |
| `ip_allowlist` | `null` (unrestricted) | — | — | CIDR ranges; see `SSO_SCIM_IMPLEMENTATION.md §25` |
| `mfa_enforcement` | `false` | — | — | Require MFA signal in id_token or SAML assertion |
| `caep_enabled` | `false` | — | — | Set to `true` after CAEP stream registration (§23) |
| `require_device_compliance` | `false` | — | — | Entra Conditional Access device compliance signal |
| `google_service_account_json` | `null` | — | — | Encrypted; required for Google group sync (§21) |

Tenant admins can adjust `session_max_duration_hours` and `refresh_token_max_days` within bounds via the admin dashboard. All other parameters are set during onboarding by `enterprise-architect`.

---

## 10. DEC-030 Audit Events

All events are HMAC-SHA256 chained per DEC-030. Cross-reference: `docs/AUDIT_LOG_SCHEMA.md`.

### 10.1 `sso.client_config_updated` — HIGH — 7yr retention

Emitted when any field in `tenant_sso_configs` is written by a FORM operator or via the admin dashboard SSO configuration UI (`SSO_SCIM_IMPLEMENTATION.md §16`).

```json
{
  "event_type": "sso.client_config_updated",
  "severity": "HIGH",
  "tenant_id": "{uuid}",
  "actor_id": "{operator_user_id}",
  "actor_type": "form_admin | tenant_owner",
  "changed_fields": ["session_max_duration_hours", "caep_enabled"],
  "previous_values_hash": "{sha256 of previous config snapshot}",
  "new_values_hash": "{sha256 of new config snapshot}",
  "ts": "{iso8601}"
}
```

Privacy note: `changed_fields` contains field names only, not values. Previous and new values are stored as hashes — not plaintext — to avoid logging secrets (`client_secret`, `google_service_account_json`).

### 10.2 `sso.jwks_key_miss` — HIGH — 7yr retention

Emitted when a token arrives with a `kid` not found in the cached JWKS, triggering a fresh fetch.

```json
{
  "event_type": "sso.jwks_key_miss",
  "severity": "HIGH",
  "tenant_id": "{uuid}",
  "idp": "entra | okta | google | generic_oidc",
  "kid_hash": "{sha256 of kid value}",
  "cache_age_seconds": 3420,
  "fresh_fetch_success": true,
  "ts": "{iso8601}"
}
```

Multiple `sso.jwks_key_miss` events within 30 minutes for the same tenant indicate a possible JWKS rotation at the IdP or a potential poisoning attempt — see AL-JWKS-01 alert in §23.

### 10.3 `sso.jwks_fetch_failed` — HIGH — 7yr retention

Emitted when a JWKS endpoint is unreachable and stale cache must be served.

```json
{
  "event_type": "sso.jwks_fetch_failed",
  "severity": "HIGH",
  "tenant_id": "{uuid}",
  "idp": "entra | okta | google | generic_oidc",
  "jwks_uri_hash": "{sha256 of JWKS URI}",
  "stale_cache_age_seconds": 18300,
  "error": "connection_timeout | dns_failure | tls_error | rate_limited",
  "ts": "{iso8601}"
}
```

### 10.4 `sso.pkce_validation_failed` — HIGH — 7yr retention

Emitted on `code_verifier` mismatch in the token exchange step.

```json
{
  "event_type": "sso.pkce_validation_failed",
  "severity": "HIGH",
  "tenant_id": "{uuid}",
  "client_type": "admin_dashboard | mobile_ios",
  "failure_reason": "verifier_mismatch | verifier_absent | challenge_downgrade_attempt",
  "source_ip_hash": "{sha256:16 of client IP with IP_HASH_SALT}",
  "ts": "{iso8601}"
}
```

`challenge_downgrade_attempt` fires when a client attempts to use `code_challenge_method=plain` instead of S256.

### 10.5 `sso.entra_cp1_verified` — STANDARD — 7yr retention

Emitted on the first successful Entra login by a tenant after CP1 configuration is deployed, confirming `xms_cc: {"cp1": true}` was received.

```json
{
  "event_type": "sso.entra_cp1_verified",
  "severity": "STANDARD",
  "tenant_id": "{uuid}",
  "actor_id": "{user_uuid}",
  "xms_cc_present": true,
  "token_lifetime_hours": 24,
  "ts": "{iso8601}"
}
```

Evidence artifact for `CC6-E-CAE-CP1-001` in SOC 2.

---

## 11. Implementation Checklist

Owner: platform-engineer + enterprise-architect

| # | Task | Owner | Priority | Milestone |
|---|---|---|---|---|
| 1 | Deploy `clientCapabilities: ["CP1"]` in `form-admin-dashboard` MSAL.js config; confirm `xms_cc` claim appears in Entra access tokens on staging | platform-engineer | **P0** | M4 |
| 2 | Deploy `clientApplicationCapabilities = ["CP1"]` in `form-mobile-ios` MSAL Swift config | platform-engineer | **P0** | M4 |
| 3 | Register `sso.entra_cp1_verified` event type in DEC-030 Zod schema registry (`compliance/dec030/event-types.ts`) | platform-engineer | **P0** | M4 |
| 4 | Register `sso.client_config_updated`, `sso.jwks_key_miss`, `sso.jwks_fetch_failed`, `sso.pkce_validation_failed` event types in DEC-030 schema registry | platform-engineer | **P0** | M4 |
| 5 | Implement `sso.jwks_key_miss` + forced fresh fetch path in `form-sso-exchange` Worker | platform-engineer | **P0** | M4 |
| 6 | Add `cacheLocation: "memory"` enforcement to `form-admin-dashboard` MSAL config; add CI lint rule that fails if `sessionStorage` or `localStorage` appears in auth token paths | platform-engineer | **P0** | M4 |
| 7 | Verify Entra app registration has CAE strict enforcement enabled (customer IT admin action; document in onboarding runbook update to `SSO_SCIM_IMPLEMENTATION.md §7.2`) | enterprise-architect | **P1** | M4 |
| 8 | Record `sso.entra_cp1_verified` event from first Entra staging login as evidence artifact `CC6-E-CAE-CP1-001` in `compliance/cc6/` | compliance-officer | **P1** | M4 |
| 9 | Add JWKS cache monitoring: AL-JWKS-01 alert on > 3 `sso.jwks_key_miss` events in 30 min for same tenant → PagerDuty P2 | platform-engineer | **P1** | M5 |
| 10 | Review `token_lifetime_hours` in `sso.entra_cp1_verified` events to confirm Entra is issuing CAE-extended tokens (target: > 8h; < 8h indicates CP1 not being honoured) | enterprise-architect | **P2** | M5 |

---

## 12. Open Questions

**OQ-SSC-01: Should `form-sso-exchange` support `private_key_jwt` client authentication for Okta by default?**

Some Okta organizations enforce `private_key_jwt` client authentication instead of `client_secret_post` as a security policy. `private_key_jwt` is more secure (asymmetric, no shared secret) but requires key rotation management. Currently, `form-sso-exchange` supports `client_secret_post` only; `private_key_jwt` is mentioned as an option but not implemented.

Recommended resolution: implement `private_key_jwt` as an opt-in per-tenant setting in `tenant_sso_configs.client_auth_method` (enum: `client_secret_post | private_key_jwt`). Key management should follow `docs/KEY_ROTATION.md` patterns — the private key stored as a Workers Secret, rotation procedure documented in KEY_ROTATION §4. Owner: platform-engineer + security-engineer. Priority: **P2 — before first Okta enterprise customer that requires JWT auth policy.** Resolution: `tenant_sso_configs` schema migration + Worker implementation + KEY_ROTATION.md addendum.

**OQ-SSC-02: How should FORM handle Entra guest/B2B accounts in enterprise tenants?**

Some Entra tenants include guest accounts (B2B) — users from other Azure directories invited into the customer's tenant. Guest accounts have a different `iss` claim structure (`https://login.microsoftonline.com/{external_tenant_id}/v2.0` rather than the enterprise tenant's issuer). The current `iss` validation rule (§6) would reject guest account tokens.

Recommended resolution: add a `allow_guest_accounts: boolean` flag to `tenant_sso_configs`. When enabled, `iss` validation is relaxed to accept tokens from guest-invited tenants, but the `aud` and `tid` claim validation is retained (must still target FORM's client and the registered enterprise tenant). Owner: enterprise-architect. Priority: **P2 — when first customer requests B2B guest account support.** Resolution: `tenant_sso_configs` schema migration + token validation rule update + DEC-030 `sso.login` payload extension to capture `is_guest_account: boolean`.

**OQ-SSC-03: Should `sso.client_config_updated` capture diff of non-secret fields in plaintext?**

Currently, `sso.client_config_updated` stores only field names and SHA-256 hashes of previous/new config snapshots. This makes the event useful for detecting that a change occurred, but auditors may request the plaintext diff of non-secret fields (e.g., `session_max_duration_hours: 8 → 24`) for SOC 2 evidence review.

Recommended resolution: store plaintext diff for a defined set of non-sensitive fields (explicit allow-list: `session_max_duration_hours`, `refresh_token_max_days`, `allow_idp_initiated`, `mfa_enforcement`, `caep_enabled`, `require_device_compliance`, `require_email_verified`). Fields not on the allow-list (including `client_secret`, `google_service_account_json`, `caep_webhook_secret`) remain hash-only. Owner: compliance-officer + platform-engineer. Priority: **P1 — before SOC 2 audit readiness review (PRE-21 gate).** Resolution: DEC-030 `sso.client_config_updated` schema update + migration + `docs/AUDIT_LOG_SCHEMA.md` entry update.

---

*v1.0 (2026-06-06): Initial release — FORM OAuth2/OIDC client application configuration reference. Closes `SSO_SCIM_IMPLEMENTATION.md §23` implementation checklist item 12 (CP1 client capability documentation). Three client applications: `form-admin-dashboard` (SPA, PKCE-only, memory token storage), `form-mobile-ios` (native, PKCE, Keychain), `form-sso-exchange` (Cloudflare Worker, confidential, `client_secret_post`). Per-IdP registration parameters for Okta, Entra, Google Workspace, Generic SAML SP, Generic OIDC. §8 Entra CAE `client_capabilities=CP1` / `xms_cc` implementation with MSAL.js and MSAL Swift code samples, verification procedure, `CC6-E-CAE-CP1-001` evidence artifact, and CAE strict enforcement prerequisite note. §7 JWKS caching policy (1h TTL, Durable Object lock on miss, stale fallback up to 6h). §6 token validation rules (RS256/PS256 only, HS256 rejected, nonce, `hd` for Google, `oid` preference for Entra). §4 PKCE enforcement — mandatory for all flows, `plain` downgrade attempt → HIGH audit event. §10 five DEC-030 events: `sso.client_config_updated` (HIGH), `sso.jwks_key_miss` (HIGH), `sso.jwks_fetch_failed` (HIGH), `sso.pkce_validation_failed` (HIGH), `sso.entra_cp1_verified` (STANDARD). §11 10-item implementation checklist (4× P0 M4, 4× P1 M4/M5, 2× P2 M5). §12 three open questions: OQ-SSC-01 (`private_key_jwt` for Okta — P2, pre-first Okta customer with JWT policy), OQ-SSC-02 (Entra B2B guest accounts — P2, on-demand), OQ-SSC-03 (plaintext diff in `sso.client_config_updated` — P1, pre-SOC 2 audit). SOC 2 controls: CC6.1 (PKCE enforcement), CC6.2 (JWKS key miss monitoring), CC6.3 (client config audit events), CC7.2 (JWKS + PKCE alerting), CC7.3 (PagerDuty AL-JWKS-01). Cross-references: `docs/SSO_SCIM_IMPLEMENTATION.md §2` (customer IdP setup), `§23` (CAE/SSF pipeline), `§22` (session revocation), `§20` (SAML cert rotation), `§16` (admin dashboard SSO UI), `§7.2` (onboarding checklist), `docs/AUDIT_LOG_SCHEMA.md` (DEC-030 registry), `docs/KEY_ROTATION.md §4` (Workers Secret rotation), `docs/SOC2_READINESS.md CC6-E-CAE-CP1-001`, `docs/ENTERPRISE.md` (enterprise tier scope). enterprise-builder cloud worker · 2026-06-06.*
