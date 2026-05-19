# FORM · SSO/SCIM Implementation v0.6

> Owner: enterprise-architect + security-engineer. Review: on any IdP change or quarterly.
> Scope: enterprise tier only. Consumer mobile (iOS) uses Apple Sign In — outside this document.
> References: DEC-030 (audit log), docs/AUDIT_LOG_SCHEMA.md, docs/ENTERPRISE.md, docs/SOC2_READINESS.md.

---

## Table of Contents

1. [Architecture Overview](#1-architecture-overview)
2. [Supported Identity Providers](#2-supported-identity-providers)
3. [SCIM 2.0 Provisioning](#3-scim-20-provisioning)
4. [Multi-Tenant Isolation](#4-multi-tenant-isolation)
5. [Role Mapping](#5-role-mapping)
6. [Security Controls](#6-security-controls)
7. [Customer Onboarding Checklist](#7-customer-onboarding-checklist)
8. [Operational Runbook](#8-operational-runbook)
9. [Open Questions / Gaps](#9-open-questions--gaps)
10. [Magic Link Fallback Security Design](#10-magic-link-fallback-security-design)
11. [Just-in-Time (JIT) Provisioning Design](#11-just-in-time-jit-provisioning-design)
12. [Session Token Lifecycle & Refresh Management](#12-session-token-lifecycle--refresh-management)
13. [Federated Logout — SAML SLO & OIDC Back-Channel Logout](#13-federated-logout--saml-slo--oidc-back-channel-logout)
14. [SCIM Data Processing: GDPR Article 28 Compliance Framework](#14-scim-data-processing-gdpr-article-28-compliance-framework)

---

## 1. Architecture Overview

### 1.1 Protocol Choice Rationale

FORM supports both SAML 2.0 and OIDC. OIDC is preferred for new deployments. SAML 2.0 is supported because a significant portion of enterprise deals involve legacy IdP configurations (Okta SAML apps pre-dating OIDC policies, on-premises ADFS, financial-sector AD FS hardened installations) where switching protocols is a multi-month internal IT process the customer cannot accelerate.

| Dimension | OIDC | SAML 2.0 |
|---|---|---|
| Token format | JWT (compact, inspectable) | XML assertion (verbose, requires XML sig validation) |
| Flow complexity | Lower — standard OAuth 2.0 + ID token | Higher — HTTP-POST/Redirect bindings, assertion consumer service |
| Modern IdP support | Native | Supported but deprecated path in some IdPs |
| Legacy enterprise support | Partial | Full — required for ADFS, Ping Federate, older Okta orgs |
| Logout complexity | Back-channel or front-channel | SLO adds significant complexity |
| PKCE support | Yes — enforced for all FORM OIDC flows | N/A |
| Preferred for | Google Workspace, Azure AD (modern), Okta (modern), generic | Okta (legacy policy), OneLogin, Ping Identity, ADFS |

**Decision:** Default to OIDC for all new tenants. SAML 2.0 activated per tenant by `enterprise-architect` request. Both supported in production from day one — no "we'll add SAML later" deferral.

---

### 1.2 Auth Flow Diagrams

#### OIDC SP-Initiated Flow (preferred)

```
Browser / Admin Dashboard                FORM Backend              Customer IdP
        |                                     |                          |
        |  1. GET /dashboard (unauthenticated)|                          |
        |------------------------------------>|                          |
        |                                     |                          |
        |  2. Redirect to IdP auth endpoint   |                          |
        |     ?client_id=...                  |                          |
        |     &redirect_uri=https://form.coach/auth/callback/{tenant_id} |
        |     &scope=openid profile email groups                         |
        |     &response_type=code             |                          |
        |     &state={csrf_token}             |                          |
        |     &code_challenge={pkce_challenge}|                          |
        |     &code_challenge_method=S256     |                          |
        |<------------------------------------|                          |
        |                                                               |
        |  3. GET {idp_auth_endpoint}?...    |                          |
        |-------------------------------------------------------------->|
        |                                                               |
        |  4. User authenticates at IdP       |                          |
        |<--------------------------------------------------------------|
        |                                                               |
        |  5. IdP redirects with auth code to form.coach/auth/callback  |
        |-------------------------------------------------------------->|
        |                                     |                          |
        |  6. FORM backend exchanges code     |                          |
        |     POST {idp_token_endpoint}       |                          |
        |     code + code_verifier (PKCE)     |                          |
        |------------------------------------>|------------------------->|
        |                                     |                          |
        |                                     |  7. IdP returns:         |
        |                                     |  access_token (IdP)      |
        |                                     |  id_token (JWT)          |
        |                                     |  refresh_token           |
        |                                     |<-------------------------|
        |                                     |                          |
        |                                     |  8. FORM validates id_token:
        |                                     |  - sig via JWKS endpoint |
        |                                     |  - iss, aud, exp claims  |
        |                                     |  - nonce (replay prevention)
        |                                     |  - hd claim (domain lock)|
        |                                     |                          |
        |                                     |  9. Extract: sub, email, |
        |                                     |  given_name, family_name,|
        |                                     |  groups                  |
        |                                     |                          |
        |                                     |  10. Upsert user in tenant
        |                                     |  (JIT provisioning if    |
        |                                     |  SCIM not active)        |
        |                                     |                          |
        |                                     |  11. Issue FORM session  |
        |                                     |  token (JWT, signed HS256|
        |                                     |  with tenant-scoped key) |
        |                                     |                          |
        |  12. Set session cookie (HttpOnly,  |                          |
        |      Secure, SameSite=Strict)       |                          |
        |<------------------------------------|                          |
        |                                     |                          |
        |  13. Audit log: sso.login           |                          |
        |      {tenant_id, actor_id, idp,     |                          |
        |       session_id, outcome: success} |                          |
```

#### SAML 2.0 SP-Initiated Flow

```
Browser / Admin Dashboard              FORM SP                   Customer IdP
        |                                  |                          |
        |  1. GET /dashboard               |                          |
        |--------------------------------->|                          |
        |                                  |                          |
        |  2. FORM generates SAMLRequest   |                          |
        |     AuthnRequest XML:            |                          |
        |     - AssertionConsumerServiceURL|                          |
        |       = https://form.coach/auth/saml/{tenant_id}/acs        |
        |     - ID (unique), IssueInstant  |                          |
        |     - Issuer = FORM entity ID    |                          |
        |     - NameIDPolicy: email        |                          |
        |  3. Deflate + base64 + sign      |                          |
        |  4. Redirect to IdP SSO URL      |                          |
        |<---------------------------------|                          |
        |                                                             |
        |  5. POST {idp_sso_url}?SAMLRequest=...&RelayState=...       |
        |------------------------------------------------------------>|
        |                                                             |
        |  6. User authenticates, IdP validates                       |
        |<------------------------------------------------------------|
        |                                                             |
        |  7. IdP POST to ACS with SAMLResponse (base64 XML)          |
        |------------------------------------------------------------>|
        |                                  |                          |
        |                                  |  8. FORM validates:      |
        |                                  |  - XML signature (IdP cert)
        |                                  |  - Destination URL match |
        |                                  |  - NotBefore / NotOnOrAfter
        |                                  |  - InResponseTo = step 2 ID
        |                                  |  - Issuer == tenant IdP entityID
        |                                  |                          |
        |                                  |  9. Extract NameID (email)
        |                                  |  + attribute assertions: |
        |                                  |  email, givenName,       |
        |                                  |  sn, department, groups  |
        |                                  |                          |
        |                                  |  10. Upsert user, map roles
        |                                  |                          |
        |                                  |  11. Issue FORM session  |
        |                                  |                          |
        |  12. Session cookie set          |                          |
        |<---------------------------------|                          |
        |                                  |                          |
        |  13. Audit log: sso.login        |                          |
```

#### IdP-Initiated SAML Flow (supported, not recommended)

IdP-initiated flows skip steps 1-4 above — the IdP posts an unsolicited SAMLResponse to the ACS. FORM accepts these only if:
- The tenant has explicitly enabled `allow_idp_initiated: true` in `tenant_sso_configs`
- The InResponseTo attribute is absent (expected for IdP-initiated)
- RelayState is validated to be a registered redirect URL within `form.coach`

IdP-initiated OIDC is not supported. If an IdP attempts an OIDC flow without a prior auth request, FORM returns 400.

---

### 1.3 Token Lifecycle

| Token | Issued by | TTL | Storage | Rotation |
|---|---|---|---|---|
| FORM session JWT | FORM backend | 8 hours (configurable per tenant, min 1h, max 24h) | HttpOnly cookie, Secure, SameSite=Strict | Rotated on each request that crosses 50% of TTL |
| FORM refresh token | FORM backend | 30 days (configurable, min 7d, max 90d per tenant policy) | HttpOnly cookie, separate path | Rotated on use (refresh rotation) |
| OIDC access token (IdP) | Customer IdP | IdP-controlled, typically 1h | Server-side only, never sent to browser | Not stored beyond initial token exchange |
| OIDC refresh token (IdP) | Customer IdP | IdP-controlled | Encrypted in Supabase `tenant_sso_sessions`, column-level AES-256 | Used only for back-channel logout and token introspection |
| SAML assertion | Customer IdP | `NotOnOrAfter` in assertion, typically 5–60 min | Validated once, not stored (stateless) | N/A |

**Session policy overrides:** Tenant admins can set the following in the admin dashboard, within FORM platform limits:
- `session_max_duration_hours` (1–24)
- `refresh_token_max_days` (7–90)
- `require_mfa: true/false` (cannot disable if FORM platform MFA policy enforced)
- `ip_allowlist: []` (CIDR ranges; empty = no restriction)
- `idle_timeout_minutes` (30–480; 0 = disabled)

---

## 2. Supported Identity Providers

### 2.1 Okta

Okta is the most common IdP in FORM's target segment (tech-forward orgs, $50k–$500k ARR). Treat this as the primary integration.

**OIDC (preferred)**

| Field | Value |
|---|---|
| Discovery URL | `https://{okta_domain}/.well-known/openid-configuration` |
| Authorization endpoint | `https://{okta_domain}/oauth2/v1/authorize` |
| Token endpoint | `https://{okta_domain}/oauth2/v1/token` |
| JWKS URI | `https://{okta_domain}/oauth2/v1/keys` |
| Required scopes | `openid profile email groups` |
| `groups` claim | Requires Okta admin to add "groups" claim to ID token via Authorization Server policy |
| Client authentication | `client_secret_post` (not `client_secret_basic`) |

Required attributes in ID token:
- `sub` — Okta user ID (stable identifier, use as external_user_id in FORM)
- `email` — must be present and verified
- `given_name`, `family_name`
- `groups` — array of group names

**SAML 2.0 (fallback)**

| Field | Value |
|---|---|
| Metadata URL | `https://{okta_domain}/app/{app_id}/sso/saml/metadata` |
| FORM ACS URL | `https://form.coach/auth/saml/{tenant_id}/acs` |
| FORM Entity ID | `https://form.coach/saml/{tenant_id}` |
| NameID format | `urn:oasis:names:tc:SAML:1.1:nameid-format:emailAddress` |
| Signature algorithm | `RSA-SHA256` |

Required attribute statements:
- `email` → User email (or map from NameID)
- `firstName` → given name
- `lastName` → family name
- `department` → optional, used for display
- `groups` → group memberships for role mapping

**Okta-specific setup steps for customer IT admin:**
1. Create a new OIDC app integration in Okta Admin Console (Applications → Create App Integration → OIDC → Web Application)
2. Set Sign-in redirect URI to `https://form.coach/auth/callback/{tenant_id}` (FORM provides `tenant_id` at contract signature)
3. Set Sign-out redirect URI to `https://form.coach/auth/logout/{tenant_id}`
4. Under Assignments, assign the app to the relevant groups
5. Under Sign On → OpenID Connect ID Token, add a "groups" claim: Name=`groups`, Include in=ID Token, Value type=Groups, Filter=Matches regex `.*`
6. Share with FORM: Client ID, Client Secret, Okta domain (`https://{yourorg}.okta.com`)

---

### 2.2 Microsoft Azure AD / Entra ID

**OIDC (preferred)**

| Field | Value |
|---|---|
| Discovery URL | `https://login.microsoftonline.com/{tenant_id_or_common}/v2.0/.well-known/openid-configuration` |
| Authorization endpoint | `https://login.microsoftonline.com/{tenant_id}/oauth2/v2.0/authorize` |
| Token endpoint | `https://login.microsoftonline.com/{tenant_id}/oauth2/v2.0/token` |
| JWKS URI | `https://login.microsoftonline.com/{tenant_id}/discovery/v2.0/keys` |
| Required scopes | `openid profile email User.Read` |
| Groups claim | Requires app manifest edit: `"groupMembershipClaims": "SecurityGroup"` or use App Roles |

Required attributes in ID token:
- `sub` or `oid` — prefer `oid` (Azure object ID, tenant-scoped stable identifier)
- `email` or `preferred_username`
- `given_name`, `family_name`
- `groups` — array of Azure group object IDs (GUIDs), or app roles if using role-based assignment

Note: Azure AD `groups` claim contains GUIDs, not display names. The tenant_sso_configs `group_mappings` table stores `{azure_group_guid: form_role}` mappings, not names.

**SAML 2.0 (fallback)**

| Field | Value |
|---|---|
| Metadata URL | `https://login.microsoftonline.com/{tenant_id}/federationmetadata/2007-06/federationmetadata.xml` |
| FORM ACS URL | `https://form.coach/auth/saml/{tenant_id}/acs` |
| FORM Entity ID | `https://form.coach/saml/{tenant_id}` |
| NameID format | `urn:oasis:names:tc:SAML:1.1:nameid-format:emailAddress` |
| Attribute claims | Configure in Azure → Enterprise Apps → Single sign-on → Attributes & Claims |

Required SAML attribute claims:
- `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress` → email
- `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/givenname` → given_name
- `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/surname` → family_name
- `http://schemas.microsoft.com/ws/2008/06/identity/claims/groups` → groups (GUIDs)

**Azure-specific setup steps for customer IT admin:**
1. Azure Portal → Azure Active Directory → Enterprise Applications → New Application → Create your own application
2. Set up Single sign-on (SAML or OIDC as preferred)
3. For OIDC: Register as App Registration → Add redirect URI `https://form.coach/auth/callback/{tenant_id}` → Generate client secret → Grant admin consent for `openid profile email User.Read`
4. For groups to appear in token: App Registration → Manifest → set `"groupMembershipClaims": "SecurityGroup"`
5. Share with FORM: Application (client) ID, Directory (tenant) ID, client secret
6. Assign users/groups to the application under Enterprise Application → Users and Groups

---

### 2.3 Google Workspace

Google Workspace supports OIDC natively. SAML is available but OIDC is strongly preferred.

**OIDC (preferred)**

| Field | Value |
|---|---|
| Discovery URL | `https://accounts.google.com/.well-known/openid-configuration` |
| Authorization endpoint | `https://accounts.google.com/o/oauth2/v2/auth` |
| Token endpoint | `https://oauth2.googleapis.com/token` |
| JWKS URI | `https://www.googleapis.com/oauth2/v3/certs` |
| Required scopes | `openid email profile` |
| Domain restriction | Use `hd` param in auth request to restrict to `{customer_domain}.com`; validate `hd` claim in id_token |

Required attributes in ID token:
- `sub` — Google user ID (stable)
- `email`
- `given_name`, `family_name`
- `hd` — hosted domain (validate this — without it, personal Gmail accounts can authenticate)

**Groups limitation:** Google Workspace OIDC does not include group memberships in the ID token. Two options:
1. Use Google Admin SDK Directory API to fetch group memberships post-authentication (requires service account + domain-wide delegation)
2. Use SAML with group attribute mapping via Google's Admin Console SAML app

For tenants requiring group-based role mapping, option 1 is the implementation path. This requires a service account per tenant — tracked in `tenant_sso_configs.google_service_account_json` (encrypted at column level).

**SAML 2.0 (for group-based role mapping)**

| Field | Value |
|---|---|
| Metadata URL | Not available as URL; Google provides XML download from Admin Console |
| FORM ACS URL | `https://form.coach/auth/saml/{tenant_id}/acs` |
| FORM Entity ID | `https://form.coach/saml/{tenant_id}` |
| NameID format | `EMAIL` |

Configure in Google Admin Console → Apps → Web and mobile apps → Add app → Add custom SAML app. Map attributes:
- Primary email → `email`
- First name → `firstName`
- Last name → `lastName`
- Department → `department` (optional)

**Google-specific setup steps for customer IT admin:**
1. Google Admin Console → Apps → Web and mobile apps → Add app → Add custom SAML app (or search for OIDC OAuth client in GCP Console for OIDC path)
2. For OIDC: GCP Console → APIs & Services → Credentials → OAuth 2.0 Client ID → Web Application → Authorized redirect URI: `https://form.coach/auth/callback/{tenant_id}`
3. Share with FORM: Client ID, Client Secret
4. If using Google Admin SDK for group sync: create service account, enable domain-wide delegation with scope `https://www.googleapis.com/auth/admin.directory.group.readonly`, share service account JSON securely

---

### 2.4 Generic SAML 2.0

For customers using OneLogin, Ping Identity, ADFS, or any SAML 2.0-compliant IdP.

**What FORM needs from the customer:**

| Item | Description |
|---|---|
| IdP metadata URL or XML | Provides entityID, SSO URL, x509 signing certificate |
| NameID format | Must be email or persistent + email attribute |
| Attribute mappings | Which SAML attribute contains email, given_name, family_name, groups |

**What FORM provides to the customer:**

| Item | Value |
|---|---|
| SP Entity ID | `https://form.coach/saml/{tenant_id}` |
| ACS URL | `https://form.coach/auth/saml/{tenant_id}/acs` |
| SP Metadata URL | `https://form.coach/auth/saml/{tenant_id}/metadata` |
| Signing certificate | Provided per tenant, downloadable from admin dashboard |
| SLO URL (if supported) | `https://form.coach/auth/saml/{tenant_id}/slo` |

FORM's SP metadata URL serves a dynamically generated XML document with the current signing certificate, entity ID, and ACS URL. Customers should configure their IdP to consume FORM's metadata URL directly (auto-refresh), not paste the XML statically — this enables zero-downtime certificate rotation.

---

### 2.5 Generic OIDC

For any OIDC-compliant IdP not listed above.

**What FORM needs from the customer:**

| Item | Description |
|---|---|
| Issuer / Discovery URL | `/.well-known/openid-configuration` endpoint |
| Client ID | Registered in the IdP |
| Client Secret | Registered in the IdP (or JWKS for private key JWT auth) |
| Scopes | At minimum: `openid email profile` |
| Claims mapping | Which claim contains email, given_name, family_name, groups |

**Validation requirements FORM enforces on all OIDC ID tokens:**
- `iss` matches the configured issuer for this tenant
- `aud` contains FORM's client_id
- `exp` is in the future (clock skew tolerance: 60 seconds)
- `iat` is within the past 10 minutes
- `nonce` matches the value sent in the auth request
- `email_verified: true` (if claim is present; absence treated as unverified — log warning, allow login unless tenant has `require_email_verified: true`)

---

## 3. SCIM 2.0 Provisioning

### 3.1 Endpoint Base URL

```
https://form.coach/scim/v2/
```

All requests require Bearer token authentication:
```
Authorization: Bearer {scim_bearer_token}
```

SCIM bearer tokens are generated per tenant in the admin dashboard (Settings → Directory Sync). Tokens are 256-bit random, stored as HMAC-SHA256 hashes in `tenant_scim_tokens`. Tokens never expire automatically but can be rotated at any time from the dashboard. Rotation is a two-token overlap window of 24 hours (old token remains valid for 24h after new token is generated).

### 3.2 Supported SCIM Operations

#### Users

| Operation | HTTP Method | Path | Notes |
|---|---|---|---|
| Create user | `POST` | `/scim/v2/Users` | Creates or reactivates |
| Get user | `GET` | `/scim/v2/Users/{id}` | `id` = FORM user UUID |
| List users | `GET` | `/scim/v2/Users?filter=...` | Supports `eq` on `userName`, `email`, `active` |
| Update user (full) | `PUT` | `/scim/v2/Users/{id}` | Full replacement |
| Update user (partial) | `PATCH` | `/scim/v2/Users/{id}` | Preferred; supports `add`, `replace`, `remove` operations |
| Deactivate user | `PATCH` | `/scim/v2/Users/{id}` | Set `active: false` |
| Delete user | `DELETE` | `/scim/v2/Users/{id}` | Soft-delete only; data retained per customer DPA |

#### Groups

| Operation | HTTP Method | Path | Notes |
|---|---|---|---|
| Create group | `POST` | `/scim/v2/Groups` | Creates FORM role mapping group |
| Get group | `GET` | `/scim/v2/Groups/{id}` | |
| List groups | `GET` | `/scim/v2/Groups` | |
| Update group members | `PATCH` | `/scim/v2/Groups/{id}` | Add/remove members; triggers role re-evaluation |
| Delete group | `DELETE` | `/scim/v2/Groups/{id}` | Members retain their last-assigned FORM role |

#### Schema Discovery (required by SCIM spec)

| Endpoint | Description |
|---|---|
| `GET /scim/v2/ServiceProviderConfig` | FORM's SCIM capabilities |
| `GET /scim/v2/Schemas` | Full schema definitions |
| `GET /scim/v2/ResourceTypes` | Supported resource types |

### 3.3 Attribute Mapping

| SCIM Attribute | FORM Field | Notes |
|---|---|---|
| `userName` | `email` | Must be unique within tenant; used as login identifier |
| `name.givenName` | `first_name` | |
| `name.familyName` | `last_name` | |
| `name.formatted` | Ignored | Derived from first + last |
| `displayName` | `display_name` | Fallback: `givenName + " " + familyName` |
| `emails[primary=true].value` | `email` | Must match `userName` |
| `active` | `is_active` | `false` = deactivated, seat released immediately |
| `externalId` | `external_user_id` | IdP-side stable identifier; used for dedup |
| `id` | `id` (FORM UUID) | Returned in SCIM responses; FORM-generated |
| `groups[].value` | Role mapping trigger | Group `id` looked up in `scim_group_role_mappings` |
| `urn:ietf:params:scim:schemas:extension:enterprise:2.0:User:department` | `department` | Optional; displayed in admin dashboard |
| `urn:ietf:params:scim:schemas:extension:enterprise:2.0:User:manager.value` | Ignored (v0.1) | Will be used for team hierarchy in v1.x |
| `urn:ietf:params:scim:schemas:extension:enterprise:2.0:User:organization` | Ignored | Tenant is already known from SCIM token scope |

**Example SCIM User create payload:**

```json
{
  "schemas": [
    "urn:ietf:params:scim:schemas:core:2.0:User",
    "urn:ietf:params:scim:schemas:extension:enterprise:2.0:User"
  ],
  "userName": "dmytro.kovalenko@acme.com",
  "name": {
    "givenName": "Dmytro",
    "familyName": "Kovalenko",
    "formatted": "Dmytro Kovalenko"
  },
  "emails": [
    { "value": "dmytro.kovalenko@acme.com", "primary": true }
  ],
  "active": true,
  "externalId": "okta_00u1abc2def3GHI4jk5",
  "urn:ietf:params:scim:schemas:extension:enterprise:2.0:User": {
    "department": "Engineering"
  }
}
```

**Example SCIM deactivation patch payload:**

```json
{
  "schemas": ["urn:ietf:params:scim:schemas:api:messages:2.0:PatchOp"],
  "Operations": [
    { "op": "replace", "path": "active", "value": false }
  ]
}
```

### 3.4 Just-in-Time (JIT) Provisioning Fallback

When SCIM is not active for a tenant, FORM falls back to JIT provisioning on first SSO login. Behavior:

1. User authenticates via SSO (OIDC or SAML)
2. FORM checks if a user record exists with `(tenant_id, email)` = match
3. If no record: create user with data from the ID token / SAML assertion
4. If record exists but was deactivated: reactivate and log `scim.user_created` with `source: jit`
5. On subsequent logins, JIT updates `first_name`, `last_name`, `department` if they have changed in the token (profile sync)
6. Groups/roles are re-evaluated on every JIT login, not just first login

JIT limitations (document for customers choosing this path):
- Users cannot be pre-provisioned (they must log in at least once)
- Deprovisioning is not automatic — FORM admin must manually deactivate, or the customer must activate SCIM
- No bulk import without SCIM or CSV upload in admin dashboard

### 3.5 De-provisioning

When `active: false` is received (SCIM PATCH) or a SCIM DELETE is received:

1. `is_active` set to `false` immediately — user cannot log in
2. All active sessions invalidated within 60 seconds (session revocation via Redis-style blocklist in Supabase)
3. Seat count decremented for billing
4. User data retained per customer DPA (default: 90 days after deactivation, configurable to 0 or up to 7 years)
5. Audit event: `scim.user_deactivated` logged with `{tenant_id, actor_id: "scim_system", resource_id: user_uuid}`

SCIM DELETE vs PATCH `active: false`: FORM treats both as deactivation, not hard deletion. Hard deletion requires a separate GDPR Right to Erasure request through the data deletion endpoint or admin dashboard. This is intentional — SCIM DELETE from an IdP should not destroy audit history.

### 3.6 Rate Limits

SCIM endpoints are rate-limited per tenant. Limits are calibrated for bulk directory sync, not real-time transactional use.

| Endpoint | Rate Limit | Burst |
|---|---|---|
| `POST /scim/v2/Users` | 100 req/min per tenant | 200 for first 5 min (initial sync) |
| `PATCH /scim/v2/Users/{id}` | 200 req/min per tenant | 400 |
| `DELETE /scim/v2/Users/{id}` | 100 req/min per tenant | 200 |
| `GET /scim/v2/Users` (list) | 30 req/min per tenant | 60 |
| `GET /scim/v2/Users/{id}` | 300 req/min per tenant | 600 |
| `POST /scim/v2/Groups` | 50 req/min per tenant | 100 |
| `PATCH /scim/v2/Groups/{id}` | 100 req/min per tenant | 200 |

Rate limit headers returned on every response:
```
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 87
X-RateLimit-Reset: 1716000060
```

When limit exceeded: `HTTP 429` with `Retry-After` header. SCIM clients are expected to implement exponential backoff — Okta, Azure AD, and OneLogin all do this natively.

---

## 4. Multi-Tenant Isolation

### 4.1 Tenant Identification

Tenant resolution order for inbound requests to admin dashboard and API:

1. **Subdomain** (primary for admin dashboard): `{tenant_slug}.form.coach/dashboard` resolves tenant via `tenants.slug` lookup
2. **Custom domain** (white-label): `coaching.acme.com` → Cloudflare CNAME → form.coach, resolved via `tenants.custom_domain` lookup in Cloudflare Worker before request hits origin
3. **JWT claim** (primary for API calls): `tenant_id` claim in FORM session JWT, validated server-side on every request
4. **SCIM token scope**: SCIM bearer token is tenant-scoped; `tenant_id` is embedded in token metadata, not sent in request

**What is never used for tenant identification:**
- `X-Tenant-ID` request header from client (untrusted)
- URL path prefix (e.g., `/api/tenant/{id}/...`) — this was rejected because it creates risk of path traversal leaking tenant IDs; JWT claim is authoritative

### 4.2 Tenant SSO Configuration Schema

```sql
CREATE TABLE tenant_sso_configs (
  id                        UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id                 UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,

  -- Protocol selection
  protocol                  TEXT NOT NULL CHECK (protocol IN ('oidc', 'saml')),
  is_active                 BOOLEAN NOT NULL DEFAULT false,
  allow_jit_provisioning    BOOLEAN NOT NULL DEFAULT true,
  allow_idp_initiated       BOOLEAN NOT NULL DEFAULT false,  -- SAML only, default off

  -- OIDC fields
  oidc_discovery_url        TEXT,
  oidc_client_id            TEXT,
  oidc_client_secret        BYTEA,                          -- AES-256 encrypted at column level
  oidc_scopes               TEXT[] DEFAULT ARRAY['openid', 'email', 'profile'],
  oidc_claims_mapping       JSONB,                          -- {"email": "email", "given_name": "given_name", "groups": "groups"}
  oidc_require_email_verified BOOLEAN DEFAULT true,
  oidc_hd_restriction       TEXT,                          -- Google hd claim lock (e.g. "acme.com")

  -- SAML fields
  saml_idp_entity_id        TEXT,
  saml_idp_sso_url          TEXT,
  saml_idp_metadata_url     TEXT,                          -- preferred; if set, cert auto-refreshed
  saml_idp_certificate      BYTEA,                         -- PEM, encrypted at column level
  saml_attribute_mapping    JSONB,                          -- {"email": "email", "firstName": "given_name", ...}
  saml_sp_certificate       BYTEA,                         -- FORM SP signing cert, per-tenant
  saml_sp_private_key       BYTEA,                         -- Encrypted private key for SP cert

  -- Session policy (tenant override; NULL = platform default)
  session_max_duration_hours  INT,
  refresh_token_max_days      INT,
  idle_timeout_minutes        INT,
  require_mfa                 BOOLEAN,
  ip_allowlist                INET[],                       -- empty = no restriction

  -- Google-specific
  google_service_account_json BYTEA,                       -- Encrypted JSON for Directory API

  -- Metadata
  configured_by             UUID REFERENCES users(id),
  configured_at             TIMESTAMPTZ,
  last_validated_at         TIMESTAMPTZ,
  created_at                TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at                TIMESTAMPTZ NOT NULL DEFAULT now(),

  UNIQUE (tenant_id)
);

-- RLS: tenant users cannot read or write another tenant's SSO config
ALTER TABLE tenant_sso_configs ENABLE ROW LEVEL SECURITY;

CREATE POLICY tenant_sso_configs_isolation ON tenant_sso_configs
  USING (tenant_id = current_setting('app.current_tenant_id')::UUID);

-- SCIM token table
CREATE TABLE tenant_scim_tokens (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id       UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
  token_hash      TEXT NOT NULL,                            -- HMAC-SHA256 of token
  label           TEXT,                                     -- human-readable ("Okta SCIM Token")
  is_active       BOOLEAN NOT NULL DEFAULT true,
  created_by      UUID REFERENCES users(id),
  created_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
  expires_at      TIMESTAMPTZ,                              -- NULL = no expiry
  last_used_at    TIMESTAMPTZ
);

-- SCIM group-to-role mapping
CREATE TABLE scim_group_role_mappings (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id       UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
  idp_group_id    TEXT NOT NULL,                            -- group ID from IdP (GUID for Azure, name for Okta)
  idp_group_name  TEXT,                                     -- display name, for readability
  form_role       TEXT NOT NULL CHECK (form_role IN ('tenant_owner', 'tenant_admin', 'tenant_manager', 'member')),
  created_at      TIMESTAMPTZ NOT NULL DEFAULT now(),

  UNIQUE (tenant_id, idp_group_id)
);
```

### 4.3 Row-Level Security

Every table containing tenant data has RLS enabled. The application sets `app.current_tenant_id` at the start of every database transaction, sourced from the validated JWT `tenant_id` claim. No query runs without this being set.

```sql
-- Example for tenant_users table
CREATE POLICY tenant_users_isolation ON tenant_users
  USING (tenant_id = current_setting('app.current_tenant_id')::UUID);
```

Lint rule enforced in CI: any new migration file that creates a table with a `tenant_id` column must include `ENABLE ROW LEVEL SECURITY` and a corresponding policy. Migration CI fails without this.

Cross-tenant queries (e.g., for FORM internal analytics) run under a dedicated service role with RLS bypassed, logged as `support.data_accessed` in the audit log, and available only via break-glass procedure per docs/AUDIT_LOG_SCHEMA.md.

### 4.4 White-Label SSO and Custom Domain

Tenants above $50k ARR may configure a custom domain. The custom domain resolves SSO flows as follows:

1. Customer configures DNS: `CNAME coaching.acme.com → {tenant_slug}.form.coach`
2. FORM provisions SSL certificate via Cloudflare for SaaS (Let's Encrypt or Cloudflare CA)
3. Cloudflare Worker intercepts requests to `coaching.acme.com`, resolves `tenant_id` from `tenants.custom_domain`, sets `X-FORM-Tenant-Id` internal header (trusted between Cloudflare Worker and origin — not from the internet)
4. SSO redirect URIs registered in IdP must use the custom domain: `https://coaching.acme.com/auth/callback`
5. `tenant_sso_configs` stores `allowed_redirect_uris` — must include both `https://{tenant_slug}.form.coach/auth/callback` and `https://{custom_domain}/auth/callback` for rollback safety

The "Powered by FORM" footer remains visible at all custom-domain tiers below the contractual threshold. Removal requires contract amendment.

---

## 5. Role Mapping

### 5.1 FORM Role Definitions

| FORM Role | Description | SCIM/SSO assignable |
|---|---|---|
| `tenant_owner` | Full org control including billing, SSO config, delete org | No — must be set manually by FORM enterprise-architect |
| `tenant_admin` | Manage users, view full analytics, configure SSO, manage branding | Yes |
| `tenant_manager` | View aggregate team analytics (no individual PII), read-only | Yes |
| `member` | Regular FORM user within org, access personal data only | Yes (default) |
| `support_readonly` | FORM internal — customer success access, read-only | No — internal only |

`tenant_owner` is excluded from SCIM/SSO role mapping deliberately. Ownership changes are a high-stakes action that must go through FORM enterprise-architect + require explicit confirmation from the outgoing owner or founder authorization. If an IdP group could reassign `tenant_owner`, a compromised IdP becomes a full org takeover vector.

### 5.2 Group-to-Role Mapping Table

Configured per tenant in `scim_group_role_mappings`. The admin dashboard provides a UI for this configuration.

Example configuration:

| IdP Group ID / Name | FORM Role |
|---|---|
| `form-admins` (Okta group name) | `tenant_admin` |
| `form-managers` | `tenant_manager` |
| `form-members` | `member` |
| `all-employees` | `member` |
| `00abc123-...` (Azure AD group GUID) | `tenant_admin` |

**Precedence rules:**
1. A user in multiple groups receives the highest-privilege matching role
2. Precedence order: `tenant_admin` > `tenant_manager` > `member`
3. If a user is in no mapped groups, the tenant's `default_role` applies (configured in `tenant_sso_configs.default_role`, default: `member`)
4. `tenant_owner` is never assigned via group mapping

### 5.3 FORM Admin Role Override

A `tenant_admin` or FORM enterprise-architect can manually override a user's role in the admin dashboard, independent of IdP group membership. The override is stored in `tenant_users.role_override` with a timestamp and actor.

**Role override behavior:**
- If `role_override` is set, it takes precedence over IdP group mapping on every login
- Override is preserved across SCIM updates (SCIM updates do not clear role_override)
- Override is cleared if: the tenant admin explicitly removes it, or the user is deprovisioned and re-provisioned
- Every role override is logged as `tenant.role_changed` in the audit log with `{actor_id, resource_id: user_uuid, before_role, after_role, reason}`

### 5.4 Role Evaluation on Every Login

Roles are not cached between sessions. On every SSO login (whether OIDC or SAML), FORM re-evaluates:
1. Current IdP group memberships (from token claims)
2. `scim_group_role_mappings` for this tenant
3. `tenant_users.role_override` (wins if set)
4. Result: effective role stored in FORM session JWT

This means role changes in the IdP propagate on next login, not immediately. For immediate revocation, SCIM `PATCH active: false` is the correct path.

---

## 6. Security Controls

### 6.1 SAML Certificate Management

FORM issues a per-tenant SP signing certificate at SSO configuration time. Certificates are 2048-bit RSA, valid for 2 years.

**Certificate storage:**
- Private key: `tenant_sso_configs.saml_sp_private_key` — AES-256-GCM encrypted at column level, key in Cloudflare KMS
- Certificate (public): `tenant_sso_configs.saml_sp_certificate` — plaintext, included in SP metadata

**Certificate rotation procedure (zero-downtime):**

See Section 8.1 for the full operational runbook. Summary:
1. Generate new cert/key pair
2. Add new cert to SP metadata (dual-cert mode — both old and new present)
3. Notify customer IT admin to re-import SP metadata from `https://form.coach/auth/saml/{tenant_id}/metadata`
4. Wait for customer confirmation that new cert is imported in their IdP
5. Begin signing with new cert; old cert still in metadata (IdP can validate either)
6. After 7-day soak: remove old cert from SP metadata
7. Log `sso.cert_rotated` in audit log

**IdP certificate rotation (customer's cert):**

If the customer rotates their IdP signing certificate:
1. Customer provides new certificate (PEM) or updated metadata URL
2. FORM imports via admin dashboard (Certificate Management panel)
3. FORM stores new cert in dual-cert mode (accepts signatures from either old or new)
4. After 48h confirmation window: remove old cert
5. Log `sso.cert_rotated` in audit log

FORM's SP metadata URL (`/auth/saml/{tenant_id}/metadata`) auto-refreshes its content when the signing certificate changes. Customers who configure their IdP to consume FORM's metadata URL (rather than static XML paste) will automatically pick up new certificates — this is the preferred setup recommended in customer onboarding.

### 6.2 OIDC PKCE Enforcement

FORM enforces PKCE (RFC 7636) for all OIDC flows. Specifically:

- `code_challenge_method=S256` only — plain is rejected
- `code_verifier` is validated server-side at token exchange
- Implicit flow (`response_type=token` or `response_type=id_token`) is rejected at the `/auth/callback` endpoint with `400 Bad Request`
- Hybrid flow (`response_type=code id_token`) is not supported

If a customer's IdP does not support PKCE (rare for modern IdPs), the SAML 2.0 path must be used instead.

### 6.3 Session Fixation Prevention

On successful SSO authentication (both OIDC and SAML):
1. Any pre-existing session cookie is invalidated immediately
2. A new session UUID is generated (`gen_random_uuid()`)
3. The new session JWT is issued with the new session UUID
4. The old session UUID is added to a session blocklist (TTL = old session's remaining valid duration)

The session blocklist is stored in Supabase with a GIN index on session UUID. The blocklist check runs synchronously on every authenticated request before the request handler executes.

### 6.4 Logout: SLO and Back-Channel Logout

**SAML Single Logout (SLO):**

FORM supports SAML SLO in both SP-initiated and IdP-initiated modes.

SP-initiated SLO (user clicks "Sign Out" in FORM):
1. FORM invalidates local session immediately
2. FORM constructs a `<LogoutRequest>` XML, signs it with the SP private key
3. Redirect to IdP SLO URL (`saml_idp_sso_url` with SLO path from metadata)
4. IdP terminates its session, sends `<LogoutResponse>` back to FORM's SLO endpoint: `https://form.coach/auth/saml/{tenant_id}/slo`
5. Log `sso.logout` with `{method: "saml_slo", initiated_by: "sp"}`

IdP-initiated SLO (user logs out at IdP portal):
1. IdP sends `<LogoutRequest>` to FORM's SLO endpoint
2. FORM validates signature
3. FORM invalidates all sessions for the NameID in the assertion (all active sessions for that user in that tenant)
4. Log `sso.logout` with `{method: "saml_slo", initiated_by: "idp"}`

If SLO fails (IdP does not respond, or SLO URL not configured in tenant metadata), FORM still invalidates the local session. The user is logged out of FORM but may remain logged in at the IdP — this is documented to customers as expected behavior when SLO is not fully configured.

**OIDC Back-Channel Logout (RFC 7009, RFC 9101):**

FORM implements the OIDC back-channel logout specification. Tenant SSO config stores `oidc_backchannel_logout_uri` where the IdP should send logout tokens.

IdP-initiated logout via back-channel:
1. IdP POST to FORM: `https://form.coach/auth/oidc/{tenant_id}/backchannel-logout` with `logout_token` in body
2. FORM validates the logout token (JWT signed by IdP, contains `sub` and/or `sid`)
3. FORM invalidates all sessions matching `sub` (user) or `sid` (specific session)
4. Response: `200 OK` (success) or `400`/`501` (error)
5. Log `sso.logout` with `{method: "oidc_backchannel", sub, sid}`

FORM's back-channel logout URI registered at the IdP: `https://form.coach/auth/oidc/{tenant_id}/backchannel-logout`

### 6.5 Audit Events for SSO/SCIM

All SSO and SCIM events are logged to the HMAC-chained `audit_log` table (DEC-030, docs/AUDIT_LOG_SCHEMA.md). The following events are defined specifically for SSO/SCIM:

| Event | Trigger | Key `metadata` fields |
|---|---|---|
| `sso.login` | Successful SSO authentication | `idp_type`, `protocol`, `session_id`, `jit_provisioned: bool`, `role_assigned` |
| `sso.login_failed` | Failed SSO attempt (invalid assertion, expired, sig invalid) | `idp_type`, `protocol`, `failure_reason`, `email_attempted` |
| `sso.logout` | User or IdP-initiated logout | `method` (sp_initiated, idp_initiated, saml_slo, oidc_backchannel), `session_id` |
| `sso.cert_rotated` | SP or IdP certificate rotation | `cert_subject`, `old_cert_fingerprint`, `new_cert_fingerprint`, `rotated_by` |
| `sso.config_updated` | SSO configuration changed | `changed_fields` (list of field names, no values), `configured_by` |
| `scim.user_created` | SCIM POST /Users or JIT provisioning | `email`, `source` (scim or jit), `role_assigned`, `external_user_id` |
| `scim.user_updated` | SCIM PUT/PATCH /Users (not deactivation) | `changed_fields` (list only, no values — privacy) |
| `scim.user_deactivated` | SCIM PATCH active=false or DELETE | `email`, `sessions_revoked_count`, `deprovisioned_by` (scim_system or actor_id) |
| `scim.user_reactivated` | SCIM PATCH active=true on deactivated user | `email`, `role_assigned` |
| `scim.group_synced` | SCIM group membership update | `group_id`, `members_added_count`, `members_removed_count` |
| `scim.token_rotated` | SCIM bearer token replaced | `label`, `rotated_by` |

Privacy constraint: `metadata.email` is included in `sso.login` and `scim.*` events because it is operationally necessary for debugging and incident response. It is not a behavioral data point. All other PII fields (name, department, health data) are never included in audit log entries per docs/AUDIT_LOG_SCHEMA.md.

---

## 7. Customer Onboarding Checklist

### 7.1 What FORM Collects Before Configuration

Before any SSO/SCIM configuration begins, FORM needs the following from the customer IT admin. Collected via a secure form in the enterprise onboarding portal (not email).

**For OIDC:**

| Item | Example |
|---|---|
| IdP type | Okta / Azure AD / Google Workspace / Other |
| Discovery / Issuer URL | `https://acme.okta.com` or `https://login.microsoftonline.com/{tenant_id}/v2.0` |
| Client ID | `0oa1abc2def3GHI4jk` |
| Client Secret | (transmitted via secure form, stored encrypted) |
| Scopes supported | `openid email profile groups` |
| Claims mapping | Which claim carries email, given name, family name, group memberships |
| Email domain(s) | `acme.com`, `acme.co.uk` (used for domain-locking) |

**For SAML:**

| Item | Example |
|---|---|
| IdP type | Okta / Azure AD / OneLogin / Ping / ADFS / Other |
| IdP metadata URL | `https://acme.okta.com/app/{app_id}/sso/saml/metadata` (preferred) OR uploaded XML |
| Attribute mapping | `email` attribute name, `givenName` attribute name, `sn` attribute name, `groups` attribute name |
| NameID format | Email preferred |

**For SCIM (additional):**

| Item | Notes |
|---|---|
| SCIM version | 2.0 only supported |
| Which operations customer IdP will use | Create, Update, Deactivate, Group Sync |
| Sync frequency | Typically real-time for Okta, near-real-time for Azure |

### 7.2 Step-by-Step: OIDC Onboarding

**FORM engineer steps (enterprise-architect or devops-lead):**

1. Create tenant record in `tenants` table; note the `tenant_id` UUID
2. Generate tenant slug (e.g., `acme-corp`) — must be DNS-safe, lowercase, unique
3. Share with customer IT admin:
   - Redirect URI: `https://form.coach/auth/callback/{tenant_id}`
   - Logout URI: `https://form.coach/auth/logout/{tenant_id}`
   - Back-channel logout URI: `https://form.coach/auth/oidc/{tenant_id}/backchannel-logout`

**Customer IT admin steps:**

1. Register FORM as OIDC app in IdP (see per-IdP steps in Section 2)
2. Set redirect URIs as provided
3. Obtain Client ID and Client Secret
4. Configure group membership in ID token (IdP-specific)
5. Share Client ID, Client Secret, and Discovery URL with FORM via secure form

**FORM engineer steps (continued):**

6. Insert into `tenant_sso_configs`:
   - `protocol: 'oidc'`
   - `oidc_discovery_url`: customer's discovery URL
   - `oidc_client_id`: from customer
   - `oidc_client_secret`: encrypted, from customer
   - `oidc_hd_restriction`: customer's email domain(s)
   - `oidc_claims_mapping`: agreed attribute mapping
   - `is_active: false` (not yet live)
7. Test using a FORM test account in the customer's IdP (see Section 7.4)
8. Set `is_active: true` after successful test
9. Configure group-to-role mappings in `scim_group_role_mappings`
10. Log `sso.config_updated` in audit log (this happens automatically via admin dashboard action)

### 7.3 Step-by-Step: SAML Onboarding

**FORM engineer steps:**

1. Create tenant record; generate SP certificate and private key for this tenant
2. Store in `tenant_sso_configs.saml_sp_certificate` and `.saml_sp_private_key` (encrypted)
3. Share with customer IT admin:
   - SP Entity ID: `https://form.coach/saml/{tenant_id}`
   - ACS URL: `https://form.coach/auth/saml/{tenant_id}/acs`
   - SP Metadata URL: `https://form.coach/auth/saml/{tenant_id}/metadata` (preferred)
   - SLO URL: `https://form.coach/auth/saml/{tenant_id}/slo`

**Customer IT admin steps:**

1. Create SAML app in IdP
2. Import FORM SP metadata from the metadata URL (or manually enter ACS URL, Entity ID)
3. Configure attribute statements/claims (see per-IdP attribute names in Section 2)
4. Assign users/groups to the application
5. Export IdP metadata XML or provide IdP metadata URL
6. Share IdP metadata URL (or XML) with FORM

**FORM engineer steps (continued):**

6. Import IdP metadata: if URL provided, store in `saml_idp_metadata_url` and fetch to populate `saml_idp_entity_id`, `saml_idp_sso_url`, `saml_idp_certificate`
7. Configure `saml_attribute_mapping` based on agreed attribute names
8. `is_active: false` — test before go-live
9. Test (Section 7.4)
10. Set `is_active: true`

### 7.4 Test Procedure Before Go-Live

The following tests must pass before `is_active` is set to `true` for any tenant.

**Test matrix:**

| Test | Expected Result |
|---|---|
| SP-initiated login with valid user | Successful redirect to admin dashboard, correct name displayed |
| SP-initiated login with user in mapped admin group | `tenant_admin` role assigned |
| SP-initiated login with user in no mapped group | `member` role assigned (default) |
| SP-initiated login with inactive user (SCIM deactivated) | 403, "Account deactivated" error shown |
| SP-initiated login with user from wrong domain (OIDC `hd` mismatch) | 403, "Login not allowed from this domain" |
| Logout from FORM | Session cleared; re-visit to dashboard requires re-auth |
| SLO / back-channel logout | Session revoked server-side within 60s |
| SAML only: replay attack (reuse same SAMLResponse) | Rejected with 400 "Assertion already consumed" |
| OIDC only: expired id_token (manipulate exp locally) | Rejected with 401 |

Testing uses a FORM-provided test user account created in the customer's IdP. Test users are removed after go-live. Test events are marked `metadata.environment: "pre_production_test"` in audit log and excluded from SOC 2 evidence collection.

### 7.5 Rollback Procedure

If SSO breaks after go-live:

1. FORM engineer sets `tenant_sso_configs.is_active: false` for the tenant — users immediately fall back to email magic link login (tenant admin only; standard members cannot log in until SSO is restored or FORM grants temporary password auth)
2. Log `sso.config_updated` with `{changed_fields: ["is_active"], previous_value: true, new_value: false, reason: "rollback"}`
3. Notify tenant admin via email (automated, triggered by `is_active` flip)
4. Debug using Section 8.2 checklist
5. Re-enable once root cause resolved: set `is_active: true`

The fallback to email magic link for tenant admin is a deliberate design choice — it preserves administrative access without exposing all members to a broken SSO flow. Standard members receive a user-facing error page with a support contact during SSO outage.

---

## 8. Operational Runbook

### 8.1 Rotating the SAML SP Signing Certificate (Zero-Downtime)

Execute this procedure when the current SP certificate approaches expiry (trigger at 60 days before expiry — automated alert from `cert_expiry_check` cron).

**Timeline: ~10 business days end-to-end (depends on customer IT responsiveness)**

```
Day 0   Alert fires: cert expires in 60 days
        enterprise-architect opens ticket in Linear

Day 0   Step 1: Generate new RSA-2048 cert/key pair for tenant
        Private key encrypted with KMS before storage
        Do NOT set as the active signing key yet

Day 0   Step 2: Add new cert to tenant_sso_configs.saml_sp_certificate_new
        The SP metadata endpoint now advertises BOTH certificates:
        <md:KeyDescriptor use="signing"> (old cert)
        <md:KeyDescriptor use="signing"> (new cert)
        FORM can validate signatures from either cert (dual-verify mode)
        FORM continues signing with the OLD cert at this point

Day 1   Step 3: Notify customer IT admin
        "FORM is rotating its SAML signing certificate. Please re-import
        FORM's SP metadata from https://form.coach/auth/saml/{tenant_id}/metadata
        at your earliest convenience. Both old and new certificates will be
        accepted for 14 days. No action is required immediately."

Day 1–7 Customer IT admin re-imports FORM metadata (their IdP now knows both certs)
        FORM waits for customer confirmation or polls with a test assertion

Day 7   Step 4: Switch active signing key to new cert
        Update tenant_sso_configs: saml_sp_certificate = new cert, saml_sp_private_key = new key
        Remove saml_sp_certificate_new
        FORM now signs assertions with new cert
        Old cert still in metadata (IdP can verify signatures from either)

Day 7–14  Soak period: monitor for SAML signature validation failures in logs
          If failures: revert signing key to old cert, recontact customer IT

Day 14  Step 5: Remove old cert from SP metadata
        SP metadata now advertises only the new cert
        Rotation complete

Day 14  Step 6: Log sso.cert_rotated in audit log
        {tenant_id, old_cert_fingerprint, new_cert_fingerprint,
         rotated_by: actor_id, valid_from, valid_until}
```

**Rollback at any point before Day 7 (Step 4):** remove `saml_sp_certificate_new`, no impact since signing key was unchanged.

**Rollback after Step 4:** revert `saml_sp_certificate` to old cert and `saml_sp_private_key` to old key. This works as long as the old cert is still in metadata and the customer IdP still accepts it.

### 8.2 Debugging SSO Login Failures

When a tenant reports "users cannot log in," work through this checklist in order. Most failures are in steps 1–5.

**Checklist:**

- [ ] **1. Check `audit_log` for `sso.login_failed` events**
  - Query: `SELECT metadata, created_at FROM audit_log WHERE tenant_id = $1 AND action = 'sso.login_failed' ORDER BY created_at DESC LIMIT 20`
  - `metadata.failure_reason` is the first diagnostic signal

- [ ] **2. Is `tenant_sso_configs.is_active = true`?**
  - If false: SSO is disabled for this tenant — intentional or rollback?

- [ ] **3. Clock skew (SAML)**
  - `failure_reason = "assertion_expired"` or `"not_yet_valid"` → FORM server time vs IdP time delta > 60s
  - Check FORM server NTP sync; check with customer if their IdP has known clock issues

- [ ] **4. Certificate mismatch (SAML)**
  - `failure_reason = "signature_invalid"` → IdP certificate stored in `saml_idp_certificate` does not match what the IdP is currently signing with
  - Check if customer recently rotated their IdP cert without notifying FORM
  - Fetch current cert from `saml_idp_metadata_url` and compare fingerprint

- [ ] **5. Audience restriction mismatch (SAML)**
  - `failure_reason = "audience_mismatch"` → IdP is sending assertions with Audience = something other than `https://form.coach/saml/{tenant_id}`
  - Common cause: customer IT admin set the wrong Entity ID in their IdP

- [ ] **6. Attribute missing (SAML or OIDC)**
  - `failure_reason = "missing_required_attribute: email"` → IdP is not sending email in the assertion
  - For SAML: check attribute statement names in IdP vs `saml_attribute_mapping`
  - For OIDC: check that the `email` scope is requested and granted

- [ ] **7. Domain restriction (OIDC)**
  - `failure_reason = "hd_mismatch"` → User's `hd` claim does not match `oidc_hd_restriction`
  - Customer user is logging in with a personal Google account, or the `hd_restriction` is misconfigured

- [ ] **8. Groups claim missing**
  - Login succeeds but user gets `member` role unexpectedly
  - Check that IdP is including groups in the token (Okta: ID token claims; Azure: manifest groupMembershipClaims)
  - Check `scim_group_role_mappings` contains the correct group IDs/names

- [ ] **9. SCIM deactivation causing SSO failure**
  - `failure_reason = "user_deactivated"` → SCIM deprovisioned this user but they're trying to log in
  - Expected behavior. If unintended: check SCIM group sync — user may have been removed from a group that triggered deactivation

- [ ] **10. IdP-initiated flow rejected**
  - `failure_reason = "idp_initiated_not_allowed"` → `allow_idp_initiated = false`
  - If customer wants IdP-initiated: set `allow_idp_initiated = true` in `tenant_sso_configs`

**Escalation:** If none of the above, capture the full SAMLResponse or OIDC token exchange (redact sensitive claims) and escalate to enterprise-architect + security-engineer.

### 8.3 Force-Disable SSO for a Tenant (Emergency Access)

Use this when a tenant's IdP is compromised, the SSO configuration is catastrophically broken, or at the explicit request of the `tenant_owner` through a verified channel.

**Requires:** enterprise-architect + security-engineer approval (2-person rule). Logged as `support.config_override`.

```sql
-- Step 1: Disable SSO
UPDATE tenant_sso_configs
SET is_active = false
WHERE tenant_id = $TENANT_ID;

-- Step 2: Revoke all active SSO sessions for this tenant
-- All sessions from SSO-authenticated users will require re-auth
-- Implemented via session blocklist — invalidate all session_ids for tenant
INSERT INTO session_blocklist (session_id, reason, blocked_at)
SELECT session_id, 'sso_emergency_disable', now()
FROM active_sessions
WHERE tenant_id = $TENANT_ID AND auth_method = 'sso';

-- Step 3: Issue temporary email magic link capability for tenant_admin/tenant_owner
-- Set in tenants table:
UPDATE tenants
SET allow_magic_link_fallback = true,
    magic_link_fallback_expires_at = now() + interval '24 hours'
WHERE id = $TENANT_ID;
```

After the above SQL:
- Notify `tenant_owner` and `tenant_admin` users via email that SSO has been disabled and they can log in via magic link for 24 hours
- Create Linear ticket with the emergency action documented
- Log `support.config_override` in audit log with `{reason, requested_by, approved_by_1, approved_by_2}`
- Customer receives automated notification within 24h per docs/AUDIT_LOG_SCHEMA.md `support.*` policy

To re-enable: set `is_active = true`, `allow_magic_link_fallback = false` after root cause is resolved.

### 8.4 IdP Outage — Fallback Procedure

If the customer's IdP is down (Okta status page red, Azure AD incident, etc.):

**Standard members:** Cannot log in during IdP outage. This is expected behavior — FORM does not maintain parallel credential stores for enterprise users. The session JWT extends the last valid session for its remaining TTL (up to 8h by default), so users who were already logged in continue to work.

**Tenant admin / tenant owner only — emergency access:**

1. Tenant admin contacts FORM support with a verified request (via contractually specified security contact email + OTP verification)
2. FORM enterprise-architect grants temporary magic link login for the admin role only:
   ```sql
   UPDATE tenants
   SET allow_magic_link_fallback = true,
       magic_link_fallback_roles = ARRAY['tenant_admin', 'tenant_owner'],
       magic_link_fallback_expires_at = now() + interval '4 hours'
   WHERE id = $TENANT_ID;
   ```
3. Admin receives magic link at their verified email address (the email stored in their FORM user record, not asserted by the failing IdP)
4. Admin can access the admin dashboard to view status, manage users
5. Magic link fallback expires after 4 hours; can be extended in 4-hour increments with re-verification
6. Log `support.config_override` with `{reason: "idp_outage_fallback", duration_hours: 4}`

**Standard members do not get magic link access during IdP outage.** This is intentional — allowing password bypass for all users during an IdP outage is a social engineering attack vector. The correct communication to standard members is: "We are experiencing an issue with single sign-on. Your administrator has been notified. Please check your company's status page."

---

## 9. Open Questions / Gaps

The following items are not yet built or require a decision before implementation. Each has an owner assigned.

| # | Gap | Severity | Owner | Notes |
|---|---|---|---|---|
| G-001 | **SCIM endpoint not yet implemented.** Schema is defined; endpoint code, token authentication, and attribute mapping are not written. | Critical — blocks any SCIM provisioning | enterprise-architect + platform-engineer | Estimate: 6–8 weeks engineering |
| G-002 | **SAML SLO not yet implemented.** SP-initiated and IdP-initiated SLO are designed but not coded. Logout currently only invalidates FORM session, does not propagate to IdP. | High — required for SOC 2 CC6 (logical access revocation) | security-engineer | Estimate: 2–3 weeks |
| G-003 | **OIDC back-channel logout not implemented.** The endpoint exists in design but not in code. | High | platform-engineer | Estimate: 1 week (simpler than SLO) |
| G-004 | **Certificate rotation automation.** The rotation runbook in 8.1 is manual. The 60-day expiry alert cron and the dual-cert metadata endpoint do not exist yet. | Medium — operational risk | devops-lead | Estimate: 1–2 weeks |
| G-005 | **Google Directory API integration for group sync.** Currently Google Workspace OIDC lacks group membership in ID token. The service account approach is designed but not implemented. | Medium — Google Workspace tenants cannot use group-based role mapping without this | platform-engineer | Estimate: 2 weeks; requires per-tenant service account credential management |
| G-006 | **`tenant_sso_configs.ip_allowlist` enforcement.** Schema column exists; the Cloudflare Worker that enforces IP allowlist on SSO callbacks is not written. | Medium — promised in enterprise contracts | platform-engineer | Estimate: 1 week |
| G-007 | **Admin dashboard SSO configuration UI.** Currently all SSO config changes require direct SQL. The admin dashboard UI for SSO setup, certificate management, and group mapping does not exist. | High — required before first enterprise pilot | enterprise-architect (product spec) + platform-engineer (build) | Estimate: 3–4 weeks |
| G-008 | **SCIM group sync for Azure AD GUIDs.** The design handles GUID-based group IDs, but the admin dashboard UI for mapping GUIDs to roles (where the customer IT admin enters human-readable group names and FORM resolves GUIDs via Microsoft Graph) is not designed. | Medium | enterprise-architect | Needs product design decision: do we show GUIDs and require customer to map, or do we call Graph API to resolve names? |
| G-009 | **Session blocklist persistence.** The design uses a Supabase table for session blocklisting. Under high-revocation scenarios (large tenant SCIM deactivation of 1000+ users simultaneously), this table lookup on every request becomes a hot-path bottleneck. | Medium — performance, not correctness | devops-lead + platform-engineer | Options: Redis cache layer, Bloom filter, JWT short-TTL with revocation threshold. Needs decision before >100-seat enterprise customers. |
| G-010 | **Magic link fallback security design.** The current fallback design in 8.3 and 8.4 uses verified email as the trust anchor. The exact verification flow (OTP via email, rate limiting, abuse detection) is not specified. | ~~High — security-critical~~  **Resolved — see §10** | security-engineer | Design specified in §10. Implementation pending. |
| G-011 | **Tenant SSO configuration test environment.** Customers currently have no sandbox/staging environment to test SSO config before production. This leads to live testing, which breaks existing users if misconfigured. | Medium | enterprise-architect | Proposal: dedicated `{tenant_slug}.staging.form.coach` environment with separate `tenant_id` for pre-production testing |
| G-012 | **OIDC private key JWT client authentication.** Some enterprise IdPs require `private_key_jwt` client auth instead of `client_secret_post`. Not supported yet. | Low — affects <5% of prospective customers (primarily financial sector) | platform-engineer | Estimate: 1 week once RFC 7523 is understood |
| G-013 | **Legal: DPA clause for SCIM data processing.** SCIM provisioning involves FORM processing employee directory data (names, emails, departments) on behalf of the customer. The standard DPA template needs a clause explicitly covering SCIM as a processing activity. | High — GDPR Art. 28 compliance | compliance-officer + outside counsel | Block: do not enable SCIM for any customer until DPA is updated |

---

## 10. Magic Link Fallback Security Design

### 10.1 Purpose and Scope

This section specifies the security design for the email OTP fallback authentication mechanism referenced in §8.3 and §8.4. The fallback allows a limited set of users to authenticate without an active IdP connection, using a time-limited one-time passcode delivered to a verified email address.

This path is high-risk because it bypasses the primary SSO control entirely. A weak fallback implementation creates an SSO bypass vector that renders the entire SSO security posture moot. The design constraints in this section are non-negotiable: no implementation that relaxes these controls may ship without a formal security review.

Scope:
- Applies to enterprise tenants with SSO configured (`tenant_sso_configs.is_active = true`)
- Governs both admin-initiated fallback (§8.3) and IdP-outage fallback (§8.4)
- Does not apply to consumer mobile users (Apple Sign In, out of scope)
- Does not cover SCIM provisioning — fallback OTPs are authentication-only; no directory write operations are permitted during a fallback session

### 10.2 Threat Model

| Attack Vector | Severity | Mitigation |
|---|---|---|
| Email account takeover — attacker controls the target's email inbox and receives the OTP | Critical | Fallback requires prior SSO session history (user must have ≥1 prior SSO login). New accounts with no SSO history cannot use fallback. Tenant admin notification on every fallback use. |
| OTP brute force — attacker submits guesses against a valid OTP session | High | Max 3 attempts per OTP; lockout for 30 minutes after 3 failures. OTP is single-use and expires after 10 minutes regardless of attempts remaining. |
| OTP phishing — attacker sends a fake FORM login page and relays the OTP in real time | High | OTP is bound to the session cookie established at the start of the fallback flow. An OTP submitted from a different session (different cookie) is rejected even if the OTP value is correct. |
| OTP enumeration — attacker submits OTP requests for many email addresses to determine which are valid FORM users | Medium | The OTP request endpoint returns an identical response (`202 Accepted`) regardless of whether the email is a known user in the tenant. No timing differential. Rate limiting applies per IP before user lookup. |
| Fallback abuse during non-outage — attacker DoSes the IdP (e.g., floods the IdP OIDC discovery endpoint) to trigger fallback conditions and then uses fallback to authenticate | High | Fallback is gated on 3 consecutive health check failures at 20-second intervals (60 seconds minimum confirmation window). A synthetic IdP outage lasting under 60 seconds does not trigger fallback. Health check monitors the IdP JWKS or metadata endpoint, not the authorization endpoint, which is harder to selectively block. Admin-explicit activation requires a verified out-of-band request. |
| Replay attack — attacker captures an OTP (e.g., via email compromise after the fact) and attempts to reuse it | High | OTPs are single-use. On successful consumption, the Redis key is deleted immediately. An OTP that has already been consumed returns a distinct error (`otp_already_consumed`) distinguishable from expiry for audit purposes, but both are rejected. |
| Session fixation — attacker pre-establishes a session and tricks the victim into completing the OTP flow on the attacker's session | High | Session cookie is validated at OTP submission; the cookie must match the one issued when the OTP request was initiated. On successful OTP validation, the existing session is invalidated and a new session UUID is issued (see §6.3 session fixation prevention). |
| Fresh account abuse — attacker creates a new account in the tenant (via SCIM or JIT) and immediately attempts fallback to bypass IdP authentication | Medium | Fallback requires ≥1 prior SSO session recorded in `tenant_sso_sessions`. An account with no SSO history is ineligible regardless of tenant fallback settings. |

### 10.3 OTP Specification

| Parameter | Value | Rationale |
|---|---|---|
| Format | 6 numeric digits | Sufficient entropy for the attempt limit; easier to transcribe than alphanumeric; no ambiguous characters |
| TTL | 10 minutes | Short enough to limit the exposure window; long enough for transactional email delivery in degraded conditions |
| Max attempts | 3 | Standard industry floor; beyond 3 attempts the probability of legitimate entry error is low relative to brute-force risk |
| Lockout duration | 30 minutes | Applies per user per email address; lockout is stored in Redis with TTL; not a hard account lock |
| Single-use | Yes | OTP Redis key is deleted on first successful validation, before the session is issued |
| Session binding | Yes | OTP is stored with the `session_id` from the initiating request. Validation rejects the OTP if the session cookie does not match `session_id` stored alongside the OTP in Redis |
| Generation | Cryptographically random via `crypto.getRandomValues()` (Web Crypto API, Cloudflare Workers) | No Math.random(), no sequential counters |

### 10.4 Rate Limits

| Limit Type | Threshold | Enforcement Layer | Behavior on Breach |
|---|---|---|---|
| Per-email OTP requests | 3 requests per hour per email address | Cloudflare Worker + Redis counter | `429 Too Many Requests`; no OTP generated; response body does not reveal whether email is valid |
| Per-IP OTP requests | 10 requests per hour per IP | Cloudflare WAF rate rule | `429`; IP temporarily blocked at WAF layer before request reaches origin |
| Per-tenant OTP requests | 50 requests per hour per tenant | Cloudflare Worker + Redis counter | `429`; PagerDuty P2 alert triggered simultaneously |
| Concurrent active OTPs per user | 1 | Redis: issuing a new OTP invalidates any existing OTP for the same user | Only one active OTP per user at a time; eliminates parallel brute-force across multiple sessions |
| Global spike alert | 500 OTP requests per minute across all tenants | Cloudflare Analytics + alert rule | PagerDuty P1; does not auto-block (too high a risk of false-positive during legitimate multi-tenant outage) |

### 10.5 Trigger Conditions

All of the following conditions must be true simultaneously for an OTP to be issued:

1. `tenant.sso_fallback_enabled = TRUE` (tenant admin can set this to FALSE to permanently disable fallback)
2. IdP unavailability confirmed — one of:
   - 3 consecutive health check failures against the tenant's IdP JWKS or metadata endpoint, at 20-second polling intervals (minimum 60 seconds of confirmed unavailability); OR
   - Explicit activation by a FORM enterprise-architect following a verified request from the `tenant_owner` or `tenant_admin`, recorded as `fallback.admin.enabled` in the audit log
3. The requesting user has at least one prior `sso.login` event recorded in `audit_log` for this `(tenant_id, user_id)` pair
4. The email domain of the requesting user matches a domain in `tenant_sso_configs.verified_domains`

**Important:** When the IdP is reachable (health check passing), no OTP is issued for that tenant even if the fallback API endpoint is called. The endpoint checks IdP health synchronously on every request before evaluating any other condition. This check is not bypassable by the calling client.

### 10.6 Verification Flow

```
User                        FORM API (Cloudflare Worker)         Redis           Email Delivery
 |                                   |                              |                  |
 |  GET /auth/fallback/initiate      |                              |                  |
 |  {tenant_id, session_cookie}      |                              |                  |
 |---------------------------------->|                              |                  |
 |                                   |  Check IdP health (3x        |                  |
 |                                   |  timeout at 20s intervals)   |                  |
 |                                   |  [fails 3 consecutive]       |                  |
 |                                   |                              |                  |
 |  200 OK: fallback form rendered   |                              |                  |
 |<----------------------------------|                              |                  |
 |                                   |                              |                  |
 |  POST /auth/fallback/request-otp  |                              |                  |
 |  {email, tenant_id, session_id}   |                              |                  |
 |---------------------------------->|                              |                  |
 |                                   |  Rate limit check            |                  |
 |                                   |  (per-email, per-IP,         |                  |
 |                                   |   per-tenant)                |                  |
 |                                   |                              |                  |
 |                                   |  Verify: prior SSO session   |                  |
 |                                   |  exists for (tenant, email)  |                  |
 |                                   |                              |                  |
 |                                   |  Generate OTP (6 digits,     |                  |
 |                                   |  crypto.getRandomValues)     |                  |
 |                                   |                              |                  |
 |                                   |  SET otp:{tenant}:{user_id}  |                  |
 |                                   |  {otp_hash, session_id,      |                  |
 |                                   |   attempt_count: 0}          |                  |
 |                                   |  TTL = 600s                  |                  |
 |                                   |----------------------------->|                  |
 |                                   |                              |                  |
 |                                   |  Queue transactional email   |                  |
 |                                   |  (OTP, tenant name, TTL,     |                  |
 |                                   |   IP of requestor)           |                  |
 |                                   |------------------------------------------------>|
 |                                   |                              |                  |
 |  202 Accepted                     |                              |                  |
 |<----------------------------------|                              |                  |
 |                                   |                              |                  |
 |  [User reads email, enters OTP]   |                              |                  |
 |                                   |                              |                  |
 |  POST /auth/fallback/verify       |                              |                  |
 |  {otp, session_cookie}            |                              |                  |
 |---------------------------------->|                              |                  |
 |                                   |  GET otp:{tenant}:{user_id}  |                  |
 |                                   |<-----------------------------|                  |
 |                                   |                              |                  |
 |                                   |  Validate:                   |                  |
 |                                   |  - session_id matches cookie |                  |
 |                                   |  - HMAC(otp) matches stored  |                  |
 |                                   |  - attempt_count < 3         |                  |
 |                                   |  - key not expired           |                  |
 |                                   |                              |                  |
 |                                   |  DEL otp:{tenant}:{user_id}  |                  |
 |                                   |  (single-use: delete first,  |                  |
 |                                   |   then issue session)        |                  |
 |                                   |----------------------------->|                  |
 |                                   |                              |                  |
 |                                   |  Invalidate old session_id   |                  |
 |                                   |  Issue new FORM session JWT  |                  |
 |                                   |  auth_method: fallback       |                  |
 |                                   |                              |                  |
 |                                   |  Write audit log:            |                  |
 |                                   |  fallback.otp.consumed       |                  |
 |                                   |  fallback.session.issued     |                  |
 |                                   |                              |                  |
 |  Set-Cookie: new session JWT      |                              |                  |
 |  Redirect to admin dashboard      |                              |                  |
 |<----------------------------------|                              |                  |
```

On OTP validation failure: increment `attempt_count` in Redis. If `attempt_count >= 3`, delete the key and set a lockout key `otp_lockout:{tenant}:{user_id}` with TTL = 1800 seconds. Write `fallback.otp.failed` to audit log. Return `401` with a generic error message (do not distinguish between wrong OTP, expired OTP, or session mismatch in the response body — differentiate only in the audit log).

### 10.7 Abuse Detection and Alerting

| Event | Threshold | Alert |
|---|---|---|
| OTP validation failures | 3 failures within 30 minutes for a single user | PagerDuty P2 + email to tenant admin |
| OTP request spike for a tenant | More than 20 OTP requests per minute for a single tenant | PagerDuty P2 |
| Cross-tenant OTP attempt | Any request where the email domain does not match the target tenant's verified domains | PagerDuty P1 (critical); request rejected; event logged as `fallback.abuse.flagged` |
| Geo anomaly on first fallback use | OTP request IP country (via `CF-IPCountry` header) differs from the country of the user's last SSO login | Email to user + email to tenant admin; does not block the request |
| Successful fallback after IdP restored | Any `fallback.session.issued` event where the IdP health check is passing at the time of session issuance | Email to tenant admin; this indicates the fallback was used after the trigger condition cleared |

All abuse detection events are written to the audit log as `fallback.abuse.flagged` with `metadata` fields identifying the specific condition triggered.

### 10.8 Audit Events

All fallback events are logged to the HMAC-chained `audit_log` table per DEC-030. All entries include `metadata.auth_method: "fallback"`.

| Event | Trigger | Key `metadata` Fields |
|---|---|---|
| `fallback.otp.requested` | User submits email on the fallback form; OTP generated and sent | `tenant_id`, `user_id`, `ip`, `country` (CF-IPCountry), `idp_health_status` |
| `fallback.otp.consumed` | OTP successfully validated | `tenant_id`, `user_id`, `session_id_old`, `session_id_new`, `attempt_number` |
| `fallback.otp.failed` | OTP validation fails (wrong value, expired, session mismatch) | `tenant_id`, `user_id`, `failure_reason` (`wrong_otp`, `expired`, `session_mismatch`, `lockout`), `attempt_number` |
| `fallback.otp.expired` | TTL on Redis OTP key elapses without consumption (detected at cleanup or next request attempt) | `tenant_id`, `user_id` |
| `fallback.session.issued` | New FORM session JWT issued following successful OTP | `tenant_id`, `user_id`, `session_id`, `idp_health_status_at_issue` |
| `fallback.abuse.flagged` | Abuse detection condition triggered (see §10.7) | `tenant_id`, `user_id`, `abuse_type`, `ip`, `country` |
| `fallback.admin.disabled` | Tenant admin or FORM engineer sets `sso_fallback_enabled = FALSE` | `tenant_id`, `actor_id`, `previous_value: true` |
| `fallback.admin.enabled` | FORM engineer re-enables fallback following a verified request | `tenant_id`, `actor_id`, `requested_by`, `reason` |

### 10.9 Admin Controls

Tenant admins control fallback behavior via the admin dashboard (Settings → Security → SSO Fallback). All changes are audit-logged.

| Setting | Default | Description |
|---|---|---|
| `sso_fallback_enabled` | `TRUE` | Master switch. When `FALSE`, no OTP is ever issued for this tenant regardless of IdP health. Turning this off disables all fallback; communicate the implications to the tenant admin before they toggle it. |
| `sso_fallback_notify_admin_on_use` | `TRUE` | When `TRUE`, tenant admins receive an email notification every time a fallback session is issued for any user in their tenant. Recommended to leave enabled. |
| `sso_fallback_allowed_hours` | `24` (hours) | Maximum duration that the fallback mechanism remains active after it is triggered. After this window, fallback is suspended even if the IdP remains down, until a FORM engineer re-activates it following re-verification. Configurable range: 1–72 hours. |
| Emergency one-click disable | N/A | A button in the admin dashboard immediately sets `sso_fallback_enabled = FALSE` and writes `fallback.admin.disabled` to the audit log. No confirmation dialog — designed for rapid response. |

### 10.10 Implementation Dependencies

| Dependency | Status | Notes |
|---|---|---|
| IdP health check polling | Not implemented | Required to be implemented first. Must poll the tenant IdP JWKS or OIDC discovery endpoint at 20-second intervals. Health state must be stored per-tenant in Redis with a short TTL. Without this, the trigger condition in §10.5 cannot be evaluated. |
| Upstash Redis for OTP TTL storage | Not implemented | OTP keys, attempt counters, lockout keys, and per-tenant/per-IP rate limit counters all require a Redis store with per-key TTL. Supabase is not suitable for this use case due to row-level TTL requirements and latency requirements at the hot path. |
| Geo IP via Cloudflare `CF-IPCountry` header | Available | `CF-IPCountry` is available on all Cloudflare Worker requests in production. No additional integration required. Only valid in production; not present in local development — mock with `XX` in non-production environments. |
| Transactional email delivery | Not implemented | Evaluate: Resend, Postmark, or AWS SES. Requirements: delivery latency under 30 seconds at p99, DKIM/SPF configured on `form.coach` sending domain, per-tenant "from name" templating, delivery receipt webhook for audit purposes. Decision must be made before OTP implementation. |
| Admin dashboard — SSO Fallback settings panel | Not implemented | Blocked by G-007 (admin dashboard). The settings in §10.9 require a UI. Implement the underlying API endpoints first; dashboard UI follows G-007. |

**Implementation sequencing recommendation:**

1. IdP health check polling (prerequisite for automated trigger)
2. OTP generation + Redis storage + transactional email delivery (core OTP flow)
3. Rate limiting (per-email, per-IP, per-tenant counters in Redis)
4. Audit log integration (all events in §10.8)
5. Geo anomaly detection (CF-IPCountry comparison against last SSO login country)
6. Admin dashboard settings panel (after G-007 unblocks)

Do not ship steps 2–6 without step 1 complete. An OTP flow without IdP health gating is an unconditional SSO bypass.

### 10.11 DPA / GDPR Implications

Fallback OTP processing involves the following personal data categories: email address, IP address, country (derived from IP), authentication event timestamps. The lawful basis for this processing is Art. 6(1)(b) GDPR (performance of a contract — the authentication service the customer has contracted FORM to provide).

OTP values are stored as HMAC hashes in Redis, not plaintext. Redis keys expire at the OTP TTL (10 minutes for OTPs, 30 minutes for lockout keys) and are not persisted to any durable store beyond the TTL. No OTP value is ever written to the audit log — only outcomes and metadata are logged.

The audit log entries generated by fallback events (see §10.8) are retained per the schedule defined in `docs/PRIVACY_POLICY.md`. These entries constitute authentication records and are subject to the same retention and access controls as `sso.login` events.

G-013 (DPA template update for SCIM) must be resolved before any SCIM-provisioned user can use the fallback mechanism in a production tenant. The DPA clause covering SCIM data processing (employee directory data) should be extended to include the following sentence: "The processor may also process the data subject's email address and network identifiers for the purpose of delivering one-time authentication codes during identity provider unavailability events, as part of the authentication services described in this agreement."

The fallback OTP processing activity must be listed in FORM's Article 30 Record of Processing Activities (RoPA) under the same entry as the primary SSO authentication activity, with a note that email and IP are processed transiently during IdP unavailability events.

---

## 11. Just-in-Time (JIT) Provisioning Design

> Owner: `enterprise-architect` + `security-engineer`. Required reading before implementing SSO callback handler.
> Dependency: §1 (Architecture), §4 (Multi-Tenant Isolation), §5 (Role Mapping), §6 (Security Controls).

JIT provisioning creates a FORM user account on the user's first successful SSO login, without requiring prior SCIM provisioning. It is the default provisioning mode for enterprise tenants who have SSO enabled but have not set up SCIM.

---

### 11.1 When JIT Applies

| Scenario | Provisioning method |
|---|---|
| SCIM enabled + user pre-provisioned | SCIM wins. JIT is skipped. User record already exists. |
| SCIM enabled + user NOT pre-provisioned | JIT creates the account. SCIM will "adopt" it on next sync (SCIM PATCH identifies by `userName`). |
| SCIM disabled + SSO enabled | JIT is the only provisioning path. All users arrive via first login. |
| SCIM disabled + SSO disabled | Invite-only provisioning. JIT does not apply. |

**Default posture for new enterprise tenants:** SCIM disabled, SSO enabled → JIT is the provisioning path until the admin configures SCIM.

---

### 11.2 JIT Provisioning Flow

```
User authenticates at their IdP (Okta / Azure AD / Google Workspace)
  ↓
IdP issues SAML assertion or OIDC ID token to FORM callback
  ↓
Cloudflare Worker: validate token signature (JWKS from tenant IdP config)
  ↓
Extract claims: { sub, email, given_name, family_name, groups, ... }
  ↓
Lookup: SELECT id FROM users WHERE tenant_id = :tid AND email = :email

  ── User exists ──────────────────────────────────────────────────────┐
  |  Update last_login_at; refresh role from IdP groups claim          |
  |  (if groups-to-roles mapping is configured in tenant_sso_configs)  |
  └────────────────────────────────────────────────────────────────────┘
  
  ── User does NOT exist ──────────────────────────────────────────────┐
  |  JIT check: is jit_provisioning_enabled for this tenant? (default: TRUE)
  |    → FALSE: reject with 403; log audit event 'sso.jit.rejected'   |
  |    → TRUE: proceed                                                 |
  |                                                                    |
  |  Seat check: SELECT COUNT(*) FROM users WHERE tenant_id = :tid     |
  |              AND deactivated_at IS NULL                            |
  |    → count >= tenant.max_seats: reject with 403; log              |
  |      audit event 'sso.jit.seat_limit_reached'; alert tenant admin  |
  |    → count < max_seats: proceed                                    |
  |                                                                    |
  |  Domain check: is email domain in tenant.verified_domains?        |
  |    → NO: reject with 403; log 'sso.jit.domain_mismatch'           |
  |    → YES: proceed                                                  |
  |                                                                    |
  |  INSERT INTO users: {                                              |
  |    id: gen_random_uuid(),                                          |
  |    tenant_id: :tid,                                                |
  |    email: :email,                                                  |
  |    display_name: given_name + ' ' + family_name,                  |
  |    role: mapped_role (from groups claim) or tenant.default_role,   |
  |    provisioned_via: 'sso_jit',                                     |
  |    external_id: :sub (IdP subject),                                |
  |    email_verified: TRUE (IdP has verified),                        |
  |    created_at: NOW()                                               |
  |  }                                                                 |
  |  Write audit_log: 'sso.jit.provisioned'                           |
  |  Send tenant admin notification (if notify_on_jit_provision = TRUE)|
  └────────────────────────────────────────────────────────────────────┘

  ↓
Issue FORM session JWT
  ↓
Redirect to app (deep link or web dashboard)
```

---

### 11.3 Claims Extraction and Mapping

#### SAML assertion claims

| SAML attribute | FORM field | Required? | Fallback if absent |
|---|---|---|---|
| `NameID` (or `uid`) | `external_id` | Required | Reject login |
| `email` (or `mail`) | `email` | Required | Reject login |
| `givenName` (or `firstName`) | `display_name` (first part) | Optional | Use email prefix |
| `sn` (or `lastName`) | `display_name` (last part) | Optional | Empty string |
| `groups` (or `memberOf`) | Role mapping (see §5) | Optional | Assign `default_role` from tenant config |

#### OIDC ID token claims

| OIDC claim | FORM field | Required? | Fallback if absent |
|---|---|---|---|
| `sub` | `external_id` | Required | Reject login |
| `email` | `email` | Required | Reject login |
| `email_verified` | Validation gate | Required (must be `true`) | Reject login; unverified emails not accepted |
| `given_name` | `display_name` (first part) | Optional | Use email prefix |
| `family_name` | `display_name` (last part) | Optional | Empty string |
| `groups` (custom claim) | Role mapping (see §5) | Optional | Assign `default_role` |

**Claim extraction configuration:** Per-tenant claim mapping is stored in `tenant_sso_configs.claim_mapping JSONB`. This allows tenants whose IdP uses non-standard attribute names (e.g., Okta custom attributes like `customEmailField`) to configure the correct source attribute name without a code change.

Example `claim_mapping`:
```json
{
  "email": "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress",
  "given_name": "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/givenname",
  "groups": "http://schemas.microsoft.com/ws/2008/06/identity/claims/groups"
}
```

---

### 11.4 Domain Verification Gate

JIT provisioning is restricted to email addresses whose domain matches the tenant's verified domains. This prevents an attacker with a valid session at a different tenant's IdP from provisioning themselves into another tenant.

**Verified domain list** is stored in `tenant_sso_configs.allowed_domains TEXT[]`. Each domain is verified by the FORM admin team before SSO is activated for the tenant (DNS TXT record verification, same mechanism as Google Workspace domain verification).

```sql
-- Tenant SSO config schema addition (supplement to §2.10):
ALTER TABLE tenant_sso_configs
  ADD COLUMN IF NOT EXISTS allowed_domains       TEXT[] NOT NULL DEFAULT '{}',
  ADD COLUMN IF NOT EXISTS jit_provisioning_enabled BOOLEAN NOT NULL DEFAULT TRUE,
  ADD COLUMN IF NOT EXISTS default_role          TEXT NOT NULL DEFAULT 'tenant_member'
    CHECK (default_role IN ('tenant_member', 'tenant_manager')),
  ADD COLUMN IF NOT EXISTS notify_on_jit_provision BOOLEAN NOT NULL DEFAULT TRUE,
  ADD COLUMN IF NOT EXISTS claim_mapping         JSONB NOT NULL DEFAULT '{}';
```

Domain check implementation (Cloudflare Worker, TypeScript):

```typescript
function isEmailDomainAllowed(email: string, allowedDomains: string[]): boolean {
  const domain = email.split('@')[1]?.toLowerCase();
  if (!domain) return false;
  // Exact match only — no subdomain matching; prevents bypass via sub.attacker.corp.example.com
  return allowedDomains.map(d => d.toLowerCase()).includes(domain);
}
```

Subdomain matching is intentionally excluded. If a tenant needs `us.corp.example.com` and `eu.corp.example.com`, both must be listed explicitly in `allowed_domains`.

---

### 11.5 Seat Limit Enforcement

JIT provisioning is blocked if the tenant has reached `max_seats`. The check is performed **inside a transaction** with a row lock to prevent race conditions when multiple users attempt JIT provisioning simultaneously.

```sql
BEGIN;

-- Row-lock the tenant row to prevent concurrent JIT provisioning from overshooting max_seats
SELECT max_seats FROM tenants WHERE id = :tenant_id FOR UPDATE;

-- Count active seats
SELECT COUNT(*) INTO active_seat_count
  FROM users
  WHERE tenant_id = :tenant_id
    AND deactivated_at IS NULL
    AND deleted_at IS NULL;

-- Conditional insert
IF active_seat_count < max_seats THEN
  INSERT INTO users (...) VALUES (...);
  -- audit log write
ELSE
  -- Return error; write audit log 'sso.jit.seat_limit_reached'
END IF;

COMMIT;
```

When the seat limit is reached, the response to the user is a generic "Account provisioning error — contact your IT administrator" message. The detailed reason (seat limit) is visible only to tenant admins in the admin dashboard, not to end users (prevents seat-count enumeration).

---

### 11.6 JIT vs. SCIM Reconciliation

When SCIM is activated on a tenant that already has JIT-provisioned users, the SCIM sync must reconcile without creating duplicate accounts or overwriting roles that were manually assigned post-JIT.

**Reconciliation logic (SCIM POST /Users on a JIT-provisioned user):**

1. SCIM provider sends a `POST /Users` for a user who already exists in FORM (matched by `userName` = email).
2. FORM SCIM handler: check if a user with `email = :email AND tenant_id = :tid` already exists.
3. If exists: return `409 Conflict` with `detail: "User already exists"` per RFC 7644 §3.3. The IdP SCIM client will then switch to a `PATCH` or `PUT` to update the existing record.
4. FORM updates: `external_id` (overwrites JIT's IdP `sub` with SCIM `externalId`), `provisioned_via: 'scim'` (promotes the account to SCIM-managed).
5. Role is NOT overwritten unless the SCIM payload explicitly includes the `roles` attribute. Manual role overrides survive SCIM reconciliation.

This ensures JIT-provisioned users are seamlessly adopted by SCIM without data loss or duplicate records.

---

### 11.7 JIT Audit Events

All JIT events are written to the HMAC-chained `audit_log` table per DEC-030.

| Event | Trigger | Key metadata |
|---|---|---|
| `sso.jit.provisioned` | New user created via JIT | `tenant_id`, `user_id` (new), `email`, `external_id`, `role_assigned`, `domain` |
| `sso.jit.login` | Existing JIT-provisioned user logs in via SSO | `tenant_id`, `user_id`, `session_id`, `idp_name` |
| `sso.jit.seat_limit_reached` | JIT blocked due to seat limit | `tenant_id`, `email`, `current_seat_count`, `max_seats` |
| `sso.jit.domain_mismatch` | JIT blocked; email domain not in `allowed_domains` | `tenant_id`, `email`, `email_domain`, `allowed_domains` |
| `sso.jit.rejected` | JIT blocked because `jit_provisioning_enabled = FALSE` | `tenant_id`, `email` |
| `sso.jit.scim_adopted` | JIT-provisioned user's record taken over by SCIM | `tenant_id`, `user_id`, `previous_provisioned_via: 'sso_jit'`, `new_external_id` |

---

### 11.8 Admin Controls

Tenant admins control JIT behavior via Settings → Security → Provisioning.

| Setting | Default | Description |
|---|---|---|
| `jit_provisioning_enabled` | `TRUE` | When `FALSE`, SSO logins by unknown users are rejected with a 403. SCIM must be used to pre-provision all users. |
| `default_role` | `tenant_member` | Role assigned to JIT-provisioned users when the IdP groups claim is absent or unmapped. Options: `tenant_member`, `tenant_manager`. Cannot be set to `tenant_admin` or `tenant_owner` via JIT. |
| `notify_on_jit_provision` | `TRUE` | When `TRUE`, tenant admin receives an email notification for each new JIT-provisioned user. Recommended for tenants without SCIM (provides visibility into who is accessing). |
| `allowed_domains` | Set at onboarding | Editable by FORM admin team only (not self-serve); domain verification required before adding. |

**Security note:** Setting `jit_provisioning_enabled = FALSE` is the right control for tenants who want strict control over who can access FORM — e.g., regulated industries where every user must be explicitly pre-approved in the IdP before accessing SaaS tools. For these tenants, the required flow is: HR/IT creates user in IdP → SCIM syncs to FORM → user can log in. A user who attempts SSO login without a SCIM-provisioned record receives a 403.

---

### 11.9 Implementation Sequencing

JIT provisioning implementation depends on:

| Dependency | Status | Notes |
|---|---|---|
| SAML assertion validation (§1, §2) | Not implemented | Core SSO flow must be implemented first |
| OIDC token validation (§1, §2) | Not implemented | Core OIDC flow must be implemented first |
| `tenant_sso_configs` schema additions (§11.4) | Not implemented | Migration required to add `jit_provisioning_enabled`, `allowed_domains`, `default_role`, `claim_mapping`, `notify_on_jit_provision` columns |
| Seat count transactional check (§11.5) | Not implemented | Requires careful transaction design to avoid race condition |
| Transactional email for admin notifications | Not implemented | Same dependency as §10.10 OTP email; Resend/Postmark/SES |
| Admin dashboard — Provisioning settings panel | Not implemented | Blocked by G-007 |

**Recommended implementation order:**
1. Core SSO callback handler (SAML assertion parse + OIDC token validate)
2. Schema migration for `tenant_sso_configs` additions
3. JIT provisioning logic in callback handler (the upsert + seat check + domain check)
4. Audit log integration
5. Admin notification email
6. Admin dashboard settings panel (after G-007)

Do not ship JIT without domain verification. An SSO implementation without domain verification is a tenant isolation bypass.

---

## 12. Session Token Lifecycle & Refresh Management

### 12.1 Overview

FORM enterprise sessions use two token types with distinct lifetimes and storage strategies. This split is deliberate: a short-lived access token limits the blast radius of token theft; a long-lived refresh token enables seamless re-authentication without forcing users through SSO on every request.

| Token type | Lifetime | Storage | Transport |
|---|---|---|---|
| JWT access token | 15 minutes | JavaScript memory only — never localStorage, never sessionStorage | `Authorization: Bearer` header on every API request |
| Refresh token | Configurable per tenant (default: 7 days) | `enterprise_sessions` table (hashed); opaque UUID in httpOnly cookie | httpOnly Secure SameSite=Lax cookie — never accessible to JavaScript |

Both token types exist at the enterprise tier only. Consumer mobile sessions use Supabase Auth's built-in session model (outside this document's scope). The enterprise session layer is implemented in Cloudflare Workers middleware sitting in front of Supabase.

**Supabase Auth relationship:** Supabase Auth manages its own JWT issuance for direct Supabase client calls. The enterprise session layer described here is an additional, parallel control plane sitting at the Cloudflare Worker edge. The Workers middleware validates and issues FORM enterprise JWTs independently of Supabase's internal session mechanism. SCIM deprovisioning, tenant-scoped timeout policies, token family tracking, and re-authentication enforcement are not achievable using Supabase Auth primitives alone — they require this custom layer.

---

### 12.2 Access Token Design

#### 12.2.1 Payload Structure

```json
{
  "sub": "usr_01J5S4hxwP2mBy6yzmCdVJnt",
  "tenant_id": "ten_01HXYZ1234ABCDEFGHIJKLMN",
  "role": "tenant_admin",
  "session_id": "ses_01J5ABCDE12345678FGHIJKL",
  "iat": 1747526400,
  "exp": 1747527300,
  "iss": "https://form.coach",
  "aud": "form-enterprise-api",
  "jti": "550e8400-e29b-41d4-a716-446655440000"
}
```

Field constraints:

| Field | Type | Notes |
|---|---|---|
| `sub` | FORM-internal user ID | Never the IdP's `sub`. Stable after JIT provisioning and SCIM adoption. |
| `tenant_id` | FORM-internal tenant ID | Present on every token. Every downstream query must validate this matches the request context. |
| `role` | Enum | One of: `tenant_owner`, `tenant_admin`, `tenant_manager`, `member`, `support_readonly`. Must match current DB value at token issuance. Role changes do not propagate to in-flight tokens — tokens must be refreshed. |
| `session_id` | Reference to `enterprise_sessions` row | Used to check revocation on the token's session without decoding the refresh token. |
| `exp` | `iat + 900` (15 minutes) | Not configurable per tenant. Short enough to limit exposure; long enough to complete any API interaction. |
| `jti` | UUID v4 | For blocklist lookup when individual token revocation is needed (elevated operations). Stored in Redis with TTL = remaining `exp` time. |

No PII in the access token payload. Email, name, and device information are not included. If a service needs the user's email, it fetches it from the database using `sub` + `tenant_id` — it does not read it from the token.

#### 12.2.2 Signing

- Algorithm: **RS256** (RSA-2048 with SHA-256). Symmetric HS256 is prohibited — it requires sharing the secret with every verifying service.
- Key rotation: monthly. Two keys are active simultaneously during a rolling 48-hour overlap window to allow existing tokens to expire without a hard cutover.
- Key storage: Cloudflare Workers Secrets (encrypted at rest). Private key is never logged, never returned in API responses, never written to Supabase.
- Public key published at: `https://form.coach/.well-known/jwks.json`. Internal services validate signatures against this endpoint with a 5-minute local cache. Cache-bust on rotation via a `kid` (key ID) field in the JWT header.

#### 12.2.3 Storage and Transport

The access token lives exclusively in the JavaScript heap of the browser process that requested it. It is never written to `localStorage`, `sessionStorage`, `IndexedDB`, or any other persistent browser store. On page reload, the access token is gone — the browser sends the httpOnly refresh token cookie, and the Workers middleware issues a new access token before the page finishes loading.

This makes XSS harder to exploit: a script injected into the page cannot exfiltrate the access token via `document.cookie` (httpOnly cookie is not accessible) and cannot read `localStorage` (nothing is there). The in-memory token is accessible only to the page's own JavaScript execution context, which is already inside the trust boundary of a page that XSS has compromised — this residual risk is accepted and mitigated by the 15-minute TTL.

---

### 12.3 Refresh Token Design

#### 12.3.1 Token Format

The refresh token is an opaque UUID v4 string (e.g., `f47ac10b-58cc-4372-a567-0e02b2c3d479`). It carries no claims. All state associated with the refresh token is stored server-side in the `enterprise_sessions` table, keyed by the SHA-256 hash of the token value.

Why opaque rather than a signed JWT: a signed refresh token would allow offline validation, which means a compromised private key enables offline token forgery. An opaque token requires a database lookup on every use, which is a controlled hot-path but eliminates the offline forgery risk. For enterprise session management — where a single SCIM deactivation must immediately invalidate all sessions — the database-backed model is required.

#### 12.3.2 Cookie Attributes

```
Set-Cookie: form_rt=f47ac10b-58cc-4372-a567-0e02b2c3d479;
  HttpOnly;
  Secure;
  SameSite=Lax;
  Path=/auth/token;
  Domain=form.coach;
  Max-Age=604800
```

| Attribute | Value | Rationale |
|---|---|---|
| `HttpOnly` | Always set | Prevents JavaScript from reading the refresh token. XSS cannot exfiltrate it. |
| `Secure` | Always set | Cookie transmitted only over HTTPS. Enforced by Cloudflare; HTTP requests are redirected before the Worker sees them. |
| `SameSite=Lax` | Lax, not Strict | Strict would prevent the cookie from being sent after a top-level navigation redirect from the IdP (e.g., the final leg of an OIDC flow). Lax allows top-level GET navigations while still blocking cross-site POST CSRF. |
| `Path=/auth/token` | Scoped path | Cookie is sent only to the refresh endpoint, not to every API call. Reduces the surface for cookie interception on other routes. |
| `Domain` | `form.coach` | Not `.form.coach` (no leading dot in modern spec). White-label tenants (`tenant.example.com`) use a separate cookie on their custom domain — see §12.3.3. |
| `Max-Age` | `tenant_sso_configs.session_timeout_hours * 3600` | Cookie lifespan matches session expiry. On browser close with session timeout, the cookie persists — the database `expires_at` is the authoritative expiry, not the cookie TTL. |

#### 12.3.3 White-Label Custom Domain Refresh Tokens

For tenants using a custom domain (e.g., `fit.acmecorp.com` CNAME → `form.coach`), the refresh token cookie is issued under the tenant's domain. The Cloudflare Worker handling the custom domain is the same Worker, but it sets `Domain=fit.acmecorp.com`. This means:

- The cookie is not sent to `form.coach` — it stays on the tenant's domain.
- FORM's Worker must receive and validate it on requests arriving via the custom domain.
- This is achieved via Cloudflare Workers routing: the Worker is attached to both `form.coach/*` and `*.form.coach/*`, and custom domain traffic is proxied through Cloudflare with a matching route.

#### 12.3.4 Single-Use with Rotation

Every use of a refresh token — whether to obtain a new access token or to obtain a new refresh token — immediately invalidates the used token and issues a new one. The `enterprise_sessions` table is updated atomically: the `refresh_token_hash` column is overwritten with the hash of the new token, and `generation` is incremented.

This property means a refresh token is a one-time credential. If an attacker steals the cookie value at rest (e.g., from a backup of the browser's cookie store) and uses it after the legitimate client has already rotated it, the stolen token is dead. If the attacker uses it first, the legitimate client's next request will trigger the token family attack detection (§12.4).

---

### 12.4 Token Family Attack Protection

#### 12.4.1 The Attack

A stolen refresh token is used by an attacker before the legitimate user's next refresh. From this point:

1. Attacker holds a valid refresh token (generation N+1) from the stolen token.
2. Legitimate user still holds the old token (generation N, now invalidated by the attacker's rotation).
3. Legitimate user's next API call triggers a refresh — their generation-N token is submitted.
4. The server sees a generation-N token submitted against a session that is now at generation N+1.

This "replay of a previously rotated token" is the detection signal. It means either a clock skew edge case (benign, rare) or active token theft and prior use (malicious, must be treated as compromise).

#### 12.4.2 Detection and Response Flow

```
Client A (legitimate)                   Client B (attacker)               Worker / DB
       |                                        |                              |
       |  [Token stolen at generation N]        |                              |
       |                                        |                              |
       |                                        | POST /auth/token             |
       |                                        | Cookie: form_rt=<gen-N>      |
       |                                        |----------------------------->|
       |                                        |                              | Lookup hash(gen-N) → found
       |                                        |                              | generation = N, family_id = F
       |                                        |                              | Issue gen-N+1, write to DB
       |                                        |<-----------------------------|
       |                                        | 200 + new cookie (gen-N+1)   |
       |                                        |                              |
       |  POST /auth/token                      |                              |
       |  Cookie: form_rt=<gen-N> (still held)  |                              |
       |---------------------------------------------------------------------->|
       |                                        |                              | Lookup hash(gen-N) → NOT FOUND
       |                                        |                              | (row now has hash(gen-N+1))
       |                                        |                              |
       |                                        |                              | DETECTION: stale token submitted
       |                                        |                              | → Revoke ALL sessions WHERE family_id = F
       |                                        |                              | → Set revoked_reason = 'token_reuse_detected'
       |                                        |                              | → Write audit event: session.token_reuse_detected
       |                                        |                              |
       |<----------------------------------------------------------------------|
       | 401 Unauthorized                       |                              |
       | WWW-Authenticate: Bearer               |                              |
       | X-FORM-Error: session_revoked          |                              |
       |                                        |                              |
       |                                        | POST /auth/token (any future)|
       |                                        |----------------------------->|
       |                                        |                              | Lookup hash(gen-N+1) → NOT FOUND
       |                                        |                              | (row revoked_at IS NOT NULL)
       |                                        |<-----------------------------|
       |                                        | 401 Unauthorized             |
```

#### 12.4.3 Family Definition

A token family is a lineage of refresh tokens issued within a single authentication event (SSO login, magic link login). Every session row carries a `family_id UUID`. When a new session is created (on SSO callback), a new `family_id` is generated. Every rotation within that session preserves the same `family_id`. When a token family is revoked, the UPDATE sets `revoked_at` and `revoked_reason = 'token_reuse_detected'` on every row sharing that `family_id`.

A single user can have multiple active families simultaneously (e.g., logged in on a laptop and a mobile device — two separate SSO logins = two separate families). Revoking one family does not affect the other.

#### 12.4.4 Clock Skew Tolerance

There is a 30-second grace window for token submissions within a single rotation cycle. If two requests from the same client arrive within 30 seconds of each other and both carry the generation-N token (due to a race between parallel requests), the second submission is treated as a duplicate rather than a reuse attack. This is tracked by `last_used_at` on the session row: if `now() - last_used_at < 30 seconds` and the submitted token matches the previous generation (N-1), issue the current generation's token again without revoking the family.

This grace window is implemented as a short-circuit before the family revocation path. Any submission of a token older than the previous generation (N-2 or older) bypasses the grace window and immediately triggers family revocation.

---

### 12.5 Sessions Table Schema

```sql
CREATE TABLE enterprise_sessions (
    session_id          UUID        PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id             UUID        NOT NULL REFERENCES auth.users(id) ON DELETE CASCADE,
    tenant_id           UUID        NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
    refresh_token_hash  CHAR(64)    NOT NULL,          -- SHA-256 hex of current refresh token
    family_id           UUID        NOT NULL,           -- Lineage group for token reuse detection
    generation          INTEGER     NOT NULL DEFAULT 0, -- Increments on every rotation
    device_fingerprint  TEXT,                           -- Hashed: UA + screen dims + timezone; for display only
    ip_last             INET,                           -- IP of last token use; for display in session list
    created_at          TIMESTAMPTZ NOT NULL DEFAULT now(),
    last_used_at        TIMESTAMPTZ NOT NULL DEFAULT now(),
    expires_at          TIMESTAMPTZ NOT NULL,           -- Authoritative expiry (tenant policy)
    revoked_at          TIMESTAMPTZ,                    -- NULL = active; NOT NULL = revoked
    revoked_reason      TEXT        CHECK (revoked_reason IN (
                            'user_logout',
                            'admin_revoked',
                            'scim_deprovisioned',
                            'token_reuse_detected',
                            'session_timeout',
                            'idle_timeout',
                            'force_reauth',
                            'key_rotation_invalidation'
                        )),

    CONSTRAINT tenant_isolation CHECK (tenant_id IS NOT NULL),
    CONSTRAINT expiry_after_creation CHECK (expires_at > created_at),
    CONSTRAINT revocation_consistency CHECK (
        (revoked_at IS NULL AND revoked_reason IS NULL) OR
        (revoked_at IS NOT NULL AND revoked_reason IS NOT NULL)
    )
);

-- Lookup path: refresh token submission (hot path)
CREATE UNIQUE INDEX idx_sessions_token_hash
    ON enterprise_sessions (refresh_token_hash)
    WHERE revoked_at IS NULL;

-- Family revocation (reuse detection)
CREATE INDEX idx_sessions_family_id
    ON enterprise_sessions (family_id)
    WHERE revoked_at IS NULL;

-- SCIM deprovisioning: revoke all active sessions for a user
CREATE INDEX idx_sessions_user_tenant
    ON enterprise_sessions (user_id, tenant_id)
    WHERE revoked_at IS NULL;

-- Expiry sweep (background job)
CREATE INDEX idx_sessions_expires_at
    ON enterprise_sessions (expires_at)
    WHERE revoked_at IS NULL;
```

Row-Level Security policy (tenant isolation):

```sql
ALTER TABLE enterprise_sessions ENABLE ROW LEVEL SECURITY;

CREATE POLICY sessions_tenant_isolation ON enterprise_sessions
    USING (tenant_id = current_setting('app.current_tenant_id')::UUID);
```

The `refresh_token_hash` column stores `encode(sha256(token_bytes), 'hex')`. The raw token value is never written to the database. SHA-256 is not reversible; a database dump does not expose usable refresh tokens.

`device_fingerprint` stores a hash (SHA-256, truncated to 16 hex chars for display) of the user agent string combined with timezone and screen dimensions — collected client-side via the admin dashboard's session management page. It is display-only metadata, not a security control. It allows users to recognize which session corresponds to which device. It is not used in token validation.

---

### 12.6 Enterprise Session Timeout Configuration

#### 12.6.1 Policy Fields

Session timeout policy is stored per tenant in `tenant_sso_configs.session_policy JSONB`:

```json
{
  "session_timeout_hours": 168,
  "idle_timeout_minutes": null,
  "max_concurrent_sessions": null,
  "reauth_on_sensitive_ops": true,
  "reauth_method": "sso"
}
```

| Field | Type | Default | Range | Notes |
|---|---|---|---|---|
| `session_timeout_hours` | Integer | `168` (7 days) | `8` – `720` | Absolute session lifetime from creation. When `expires_at` is reached, the session is dead regardless of activity. Enterprise standard is 7 days. Regulated industries (finance, healthcare) commonly require 8 hours. Legal/compliance team at the customer must specify this during onboarding. |
| `idle_timeout_minutes` | Integer or null | `null` (disabled) | `30` – `480` | If set, the session is revoked if `last_used_at` has not been updated within this window. Evaluated on every token refresh attempt. A session that has been idle longer than this threshold receives a 401 with `revoked_reason = 'idle_timeout'` on next access attempt. |
| `max_concurrent_sessions` | Integer or null | `null` (unlimited) | `1` – `10` | If set, creating a new session (SSO login) beyond this count revokes the oldest active session for the user within the tenant. Useful for tenants with strict device-control policies. |
| `reauth_on_sensitive_ops` | Boolean | `true` | — | When true, sensitive operations (listed in §12.8) require re-authentication through the IdP regardless of session age. |
| `reauth_method` | Enum | `"sso"` | `"sso"`, `"mfa_totp"` | Whether re-authentication goes back to the IdP or can be satisfied by an in-app MFA step. `"sso"` is required for tenants with SAML ForceAuthn or OIDC `prompt=login` in their security policy. |

#### 12.6.2 Schema Addition

```sql
ALTER TABLE tenant_sso_configs
    ADD COLUMN IF NOT EXISTS session_policy JSONB NOT NULL DEFAULT '{
        "session_timeout_hours": 168,
        "idle_timeout_minutes": null,
        "max_concurrent_sessions": null,
        "reauth_on_sensitive_ops": true,
        "reauth_method": "sso"
    }';

-- Validated constraint: session_timeout_hours must be between 8 and 720
ALTER TABLE tenant_sso_configs
    ADD CONSTRAINT session_policy_timeout_range CHECK (
        (session_policy->>'session_timeout_hours')::INTEGER BETWEEN 8 AND 720
    );
```

#### 12.6.3 Idle Timeout Enforcement

Idle timeout is not enforced via a background job — it is enforced lazily at the point of the next refresh token use. When the Worker processes a `POST /auth/token` request:

1. Look up the session row by `refresh_token_hash`.
2. Compute `now() - last_used_at`.
3. If the result exceeds `session_policy.idle_timeout_minutes`, immediately set `revoked_at = now()`, `revoked_reason = 'idle_timeout'`, and return `401`.
4. If the session is valid, update `last_used_at = now()` atomically with the token rotation.

This lazy approach means the idle timeout has a precision floor equal to the access token TTL (15 minutes). A session that has been idle for exactly `idle_timeout_minutes` will not be revoked until the next refresh attempt — which can happen up to 15 minutes after the idle window closes. This is acceptable: the worst-case overage is the access token TTL, which is short.

For tenants requiring hard idle enforcement (no grace window), a background sweeper job runs every 5 minutes and revokes sessions where `last_used_at < now() - (idle_timeout_minutes * interval '1 minute')` and `revoked_at IS NULL`. This is opt-in (`strict_idle_enforcement: true` in `session_policy`) and is only offered to tenants contractually requiring it, as it adds database write pressure.

#### 12.6.4 Propagating Policy Changes

When a tenant admin changes `session_policy` (e.g., reduces `session_timeout_hours` from 168 to 8), the change takes effect:

- **New sessions:** immediately. The new `session_timeout_hours` sets `expires_at` on all sessions created after the policy change.
- **Existing sessions:** the next refresh token use re-evaluates policy. If the session's `expires_at` now exceeds `now() + new_session_timeout_hours`, it is capped. This is implemented by checking policy on each refresh: if `expires_at > now() + interval '1 hour' * session_timeout_hours`, truncate it.

An audit event `tenant.session_policy_updated` is written with the previous and new policy values (redacted to field names and values, no PII).

---

### 12.7 SCIM Deprovisioning → Immediate Session Revocation

When SCIM sends a deactivation signal for a user, all active sessions for that user within the tenant must be revoked synchronously before the SCIM operation returns a 200.

#### 12.7.1 Trigger Paths

| SCIM operation | Meaning | Action |
|---|---|---|
| `PATCH /Users/{id}` with `{ "active": false }` | User deactivated in IdP | Revoke all active sessions for `user_id` + `tenant_id` |
| `DELETE /Users/{id}` | User deleted from IdP | Revoke all active sessions, then soft-delete the FORM user record |
| `PATCH /Users/{id}` with `{ "active": true }` | User reactivated | No session action. The user must re-authenticate via SSO to obtain a new session. |

#### 12.7.2 Revocation Query

```sql
UPDATE enterprise_sessions
SET
    revoked_at     = now(),
    revoked_reason = 'scim_deprovisioned'
WHERE
    user_id    = $user_id
    AND tenant_id  = $tenant_id
    AND revoked_at IS NULL
RETURNING session_id;
```

This runs inside the same database transaction as the SCIM user record update. If the transaction rolls back (e.g., due to a constraint violation), the sessions are not revoked. Atomicity is required: a deprovisioned user whose FORM record failed to update but whose sessions were revoked would be in an inconsistent state.

The returned `session_id` list is used to:

1. Purge any cached access token JTI entries from Redis (best-effort, non-blocking).
2. Write one `session.revoked_by_scim` audit event per revoked session (see §12.9).

#### 12.7.3 Latency Requirement

The session revocation SQL must complete within the SCIM response window. The SCIM endpoint has a 10-second response timeout enforced by the Cloudflare Worker. If the bulk revocation query takes longer than 8 seconds (leaving 2 seconds for network overhead), the Worker returns a `503 Service Unavailable` to the SCIM client, and the IdP SCIM client will retry. The retry is idempotent — revoking already-revoked sessions is a no-op.

For tenants with a large number of concurrent sessions per user (e.g., a highly active user with 50+ device sessions), the index `idx_sessions_user_tenant` ensures this query completes in milliseconds.

#### 12.7.4 Access Token Residual Window

After SCIM revocation, an already-issued access token (JWT) with up to 15 minutes of remaining TTL is still cryptographically valid. The Worker validates the access token's signature and expiry independently of the `enterprise_sessions` table — the access token carries a `session_id` claim but the hot path does not re-check the session row on every request (that would defeat the purpose of a short-lived JWT).

Accepted residual window: up to 15 minutes post-deprovisioning. This is the documented and accepted trade-off for the JWT model.

For tenants contractually requiring zero-latency deprovisioning (i.e., the deprovisioned user cannot make any API call after the SCIM signal arrives), a JTI blocklist check is added to the Worker's access token validation path. The JTI of every active access token issued under a revoked session is written to Redis with TTL = remaining JWT `exp` seconds. This adds one Redis lookup to every request for all enterprise tenants — a performance cost that must be weighed against the contractual requirement. This is an opt-in feature (`jti_revocation_check: true` in `session_policy`) at additional infrastructure cost.

---

### 12.8 SSO Re-authentication for Sensitive Operations

#### 12.8.1 Sensitive Operation List

The following operations require a fresh IdP authentication regardless of session age:

| Operation | Rationale |
|---|---|
| Export tenant user data (bulk PII export) | GDPR Art. 32 — data exports carry high breach risk |
| Change billing information or subscription tier | Financial fraud vector |
| Promote a user to `tenant_admin` or `tenant_owner` | Privilege escalation must be explicitly authorized |
| Add or modify SSO configuration (IdP credentials, ACS URL) | A compromised session could redirect all future SSO logins |
| Generate or rotate SCIM bearer tokens | A SCIM token is a provisioning credential — equivalent impact to admin access |
| Disable SSO entirely for the tenant | Enables magic link fallback — high-impact configuration change |
| Add a new verified domain | Domain additions affect JIT provisioning scope |
| Download audit log export | Audit logs contain behavioral data with privacy implications |

#### 12.8.2 OIDC Re-authentication

For tenants using OIDC, the Worker redirects the browser to the OIDC authorization endpoint with `prompt=login` added to the authorization request:

```
GET {idp_authorization_endpoint}
  ?client_id={form_client_id}
  &redirect_uri=https://form.coach/auth/callback/{tenant_id}
  &scope=openid profile email groups
  &response_type=code
  &state={csrf_token}:{reauth_return_path}
  &nonce={nonce}
  &prompt=login
  &max_age=0
```

`prompt=login` instructs the IdP to force credential entry even if the user has an active IdP session (SSO web session). `max_age=0` is redundant reinforcement — it tells the IdP the authentication must have happened within 0 seconds, which forces re-authentication.

After the IdP callback, the Worker validates the `auth_time` claim in the ID token. If `auth_time < now() - 60 seconds`, the re-authentication is rejected as stale (guards against IdP implementations that ignore `prompt=login`). The sensitive operation proceeds only if this check passes. The existing session is preserved — re-authentication does not create a new session; it issues a short-lived re-auth assertion (a signed, single-use JWT with `scope: reauth`, `exp: now() + 300s`) that authorizes one specific sensitive operation.

#### 12.8.3 SAML Re-authentication

For tenants using SAML 2.0, the Worker issues an `<AuthnRequest>` with `ForceAuthn="true"`:

```xml
<samlp:AuthnRequest
  xmlns:samlp="urn:oasis:names:tc:SAML:2.0:protocol"
  ID="_form_{uuid}"
  Version="2.0"
  IssueInstant="{now}"
  AssertionConsumerServiceURL="https://form.coach/auth/saml/callback/{tenant_id}"
  ForceAuthn="true"
  IsPassive="false">
  <saml:Issuer>https://form.coach/saml/{tenant_id}</saml:Issuer>
</samlp:AuthnRequest>
```

`ForceAuthn="true"` is the SAML equivalent of OIDC `prompt=login`. The IdP must prompt for credentials. After the assertion arrives, the Worker checks that the assertion's `AuthnInstant` is within 60 seconds of the current time.

#### 12.8.4 Re-auth Assertion Format

Regardless of protocol, the outcome of a successful re-authentication for a sensitive operation is a short-lived re-auth assertion attached to the current session:

```json
{
  "type": "reauth_assertion",
  "session_id": "ses_01J5ABCDE12345678FGHIJKL",
  "user_id": "usr_01J5S4hxwP2mBy6yzmCdVJnt",
  "tenant_id": "ten_01HXYZ1234ABCDEFGHIJKLMN",
  "authorized_operation": "export_user_data",
  "auth_time": 1747527300,
  "exp": 1747527600,
  "jti": "unique-per-assertion"
}
```

The assertion is signed with the same RS256 key as access tokens. The Worker validates it on the sensitive operation's API endpoint before processing the request. The assertion is single-use — consuming it immediately marks its JTI as used in Redis.

---

### 12.9 Token Revocation Audit Events

All events are written to the HMAC-chained `audit_log` table per DEC-030. No PII in metadata columns. Timestamps and identifiers only.

| Event name | Trigger | Key metadata |
|---|---|---|
| `session.created` | New session issued after SSO login | `session_id`, `tenant_id`, `user_id`, `family_id`, `generation: 0`, `ip`, `device_fingerprint_hint` (first 8 chars of hash) |
| `session.token_rotated` | Refresh token successfully rotated | `session_id`, `tenant_id`, `user_id`, `generation` (new value), `ip` |
| `session.token_reuse_detected` | Stale refresh token submitted; family revoked | `session_id`, `tenant_id`, `user_id`, `family_id`, `submitted_generation`, `current_generation`, `ip`, `sessions_revoked_count` |
| `session.revoked_user_logout` | User explicitly logs out | `session_id`, `tenant_id`, `user_id` |
| `session.revoked_admin` | Tenant admin revokes a specific session via admin dashboard | `session_id`, `tenant_id`, `user_id`, `revoked_by_user_id` |
| `session.revoked_by_scim` | SCIM deactivation or deletion revokes session | `session_id`, `tenant_id`, `user_id`, `scim_operation` (`patch_deactivate` or `delete`), `scim_request_id` |
| `session.expired` | Session swept by expiry job (or lazy expiry at refresh) | `session_id`, `tenant_id`, `user_id`, `expired_reason` (`session_timeout` or `idle_timeout`) |
| `session.family_revoked` | All sessions in a family revoked (reuse detection) | `family_id`, `tenant_id`, `user_id`, `sessions_count`, `trigger_session_id` |
| `session.reauth_initiated` | Sensitive operation triggered IdP re-authentication | `session_id`, `tenant_id`, `user_id`, `operation`, `protocol` (`oidc` or `saml`) |
| `session.reauth_completed` | Re-auth assertion issued | `session_id`, `tenant_id`, `user_id`, `operation`, `auth_time` |
| `session.reauth_stale_rejected` | IdP returned assertion with `auth_time` outside 60s window | `session_id`, `tenant_id`, `user_id`, `operation`, `auth_time`, `threshold_seconds: 60` |
| `session.jti_blocklisted` | Access token JTI added to revocation blocklist | `jti`, `session_id`, `tenant_id`, `user_id`, `expires_at` |
| `tenant.session_policy_updated` | Tenant admin changed `session_policy` | `tenant_id`, `changed_by_user_id`, `previous_policy` (field names + values, no PII), `new_policy` |

The `session.token_reuse_detected` event is a security-critical signal. FORM's monitoring infrastructure must alert on any occurrence of this event within 5 minutes. It indicates either a compromised refresh token or a client bug causing replay — both require investigation.

---

### 12.10 Implementation Notes

#### 12.10.1 Supabase Auth Boundary

Supabase Auth has its own session and JWT management. The enterprise session layer described in this document is not a replacement for Supabase Auth — it is an additional control plane for enterprise-specific requirements that Supabase Auth does not support:

| Requirement | Supabase Auth | Enterprise session layer (this doc) |
|---|---|---|
| Tenant-scoped session timeout | Not supported | Implemented in Cloudflare Worker |
| Token family tracking / reuse detection | Not supported | `enterprise_sessions` table |
| SCIM deprovisioning → instant revocation | Not supported | Synchronous revocation in SCIM handler |
| Per-operation re-authentication | Not supported | Re-auth assertion flow (§12.8) |
| Idle timeout | Not supported | Lazy enforcement in Worker |
| Session list in admin dashboard | Not supported | Query on `enterprise_sessions` |
| `session_id` in audit log | Supabase has its own session ID | FORM's `session_id` from `enterprise_sessions` |

The architecture is:

```
Browser
  |
  | httpOnly cookie (form_rt = refresh token)
  v
Cloudflare Worker (enterprise-sessions middleware)
  |
  ├─ Validates form_rt against enterprise_sessions table (Supabase DB direct connection)
  ├─ Issues FORM JWT access token (RS256, custom claims)
  ├─ Applies tenant session_policy
  ├─ Runs token family detection
  |
  ├─ On valid session: passes request to Supabase with FORM JWT + Supabase service key
  |    (Worker impersonates the user for Supabase RLS by setting `app.current_tenant_id`
  |     and `app.current_user_id` in the connection context)
  |
  └─ Supabase Auth sessions: still issued for direct Supabase client calls (Realtime, Storage)
       These are short-lived Supabase JWTs issued by Supabase Auth on the Worker's behalf.
       They are scoped to a single request context and are not the enterprise refresh token.
```

Supabase Auth's own `auth.sessions` table remains in use for Supabase-internal session tracking. The `enterprise_sessions` table is additive — it does not replace Supabase Auth but wraps it.

#### 12.10.2 Cloudflare Worker Middleware Structure

The session middleware is a Cloudflare Worker that intercepts all requests to `form.coach/api/*` and `form.coach/auth/token`. Structure:

```typescript
// workers/enterprise-sessions/index.ts

export default {
  async fetch(request: Request, env: Env): Promise<Response> {
    const path = new URL(request.url).pathname;

    // Token refresh endpoint — processes the refresh token cookie
    if (path === '/auth/token' && request.method === 'POST') {
      return handleTokenRefresh(request, env);
    }

    // API routes — validate the Bearer JWT access token
    if (path.startsWith('/api/')) {
      return handleApiRequest(request, env);
    }

    // SSO callbacks — no session required; handled by auth-callback Worker
    if (path.startsWith('/auth/callback/') || path.startsWith('/auth/saml/callback/')) {
      return env.AUTH_CALLBACK_WORKER.fetch(request);
    }

    return new Response('Not Found', { status: 404 });
  }
};
```

Key functions:

- `handleTokenRefresh`: reads `form_rt` cookie → SHA-256 hash → lookup in `enterprise_sessions` → check expiry, idle timeout, revocation → rotate token → issue new JWT access token → set new `form_rt` cookie.
- `handleApiRequest`: extract `Authorization: Bearer` → verify RS256 signature → validate `exp`, `iss`, `aud`, `tenant_id` → optionally check JTI blocklist (if `jti_revocation_check: true`) → forward to Supabase.
- `revokeFamily`: called on token reuse detection; bulk UPDATE on `enterprise_sessions` WHERE `family_id = ?`.
- `revokeByScim`: called by SCIM handler; bulk UPDATE WHERE `user_id = ? AND tenant_id = ?`.

#### 12.10.3 Migration from Supabase Default Sessions

The migration path for any tenant who authenticated before the enterprise session layer was deployed:

1. The enterprise session layer is deployed behind a feature flag: `ENTERPRISE_SESSION_LAYER_ENABLED` (Cloudflare Worker environment variable).
2. When the flag is off: requests pass through to Supabase Auth as before.
3. When the flag is on for a tenant: the Worker intercepts requests. If no `form_rt` cookie is present (the user has a Supabase Auth session but not a FORM enterprise session), the Worker responds with `401` and a `X-FORM-Auth: sso_relogin_required` header.
4. The frontend detects this header and redirects the user through the SSO flow, which issues a new enterprise session via the SSO callback handler.
5. After the SSO callback, `form_rt` is set, and subsequent requests pass through the enterprise session layer normally.

This means the migration requires each user to complete one fresh SSO login after the enterprise session layer is enabled for their tenant. There is no automatic session migration — Supabase Auth sessions are not promoted to enterprise sessions. Users are notified via an in-app banner 48 hours before the cutover: "Your organization's security settings are being updated. You'll be asked to sign in again on [date]."

The cutover is done tenant by tenant. No big-bang migration.

#### 12.10.4 RS256 Key Rotation Procedure

Key rotation happens monthly on the first Monday of each month. Procedure:

1. Generate a new RSA-2048 key pair in the secure build environment (not on any developer's machine).
2. Upload the new private key to Cloudflare Workers Secrets as `JWT_PRIVATE_KEY_NEW`.
3. Publish the new public key to `/.well-known/jwks.json` alongside the existing key (dual-key response). Both keys are active.
4. Deploy Worker update that uses `JWT_PRIVATE_KEY_NEW` for signing new tokens. Validation accepts both the old and new key (matching by `kid` in JWT header).
5. Wait 15 minutes (one full access token TTL). All tokens signed with the old key have expired.
6. Remove the old public key from `/.well-known/jwks.json`. Deploy.
7. Delete `JWT_PRIVATE_KEY_OLD` from Cloudflare Workers Secrets. Rename `JWT_PRIVATE_KEY_NEW` to `JWT_PRIVATE_KEY`.

Write `key.rotation.completed` audit event with `{kid_old, kid_new, rotation_timestamp}`.

If a key is compromised before the monthly rotation: trigger emergency rotation immediately using the same procedure, compressing step 5 from "wait 15 minutes" to "immediately revoke all active enterprise sessions" (setting `revoked_at` + `revoked_reason = 'key_rotation_invalidation'` on all non-revoked rows). This forces all users to re-authenticate, accepting the user impact in exchange for eliminating any in-flight tokens signed with the compromised key.

#### 12.10.5 Implementation Dependencies

| Dependency | Status | Notes |
|---|---|---|
| `enterprise_sessions` table migration | Not implemented | Schema in §12.5; run after SSO callback handler exists |
| `tenant_sso_configs.session_policy` column | Not implemented | Schema in §12.6.2; can be added independently |
| Cloudflare Worker session middleware | Not implemented | Blocked by: `enterprise_sessions` table, RS256 key pair in Workers Secrets |
| RS256 key pair generation | Not implemented | Required before Worker deployment |
| Redis (Cloudflare KV or Upstash) for JTI blocklist | Not implemented | Required only if `jti_revocation_check: true` is offered; can defer until first regulated-industry customer |
| SCIM deprovisioning integration | Not implemented | Blocked by G-001 (SCIM endpoint) |
| Admin dashboard — session list UI | Not implemented | Blocked by G-007 (admin dashboard) |
| Re-authentication flow (§12.8) | Not implemented | Blocked by core SSO flow (§1, §2) |
| Background expiry sweeper job | Not implemented | Cloudflare Workers Cron Trigger; low-priority (lazy enforcement covers most cases) |

**Recommended implementation order:**
1. `enterprise_sessions` table migration + RLS policy
2. `session_policy` column on `tenant_sso_configs`
3. RS256 key pair generation; publish JWKS endpoint
4. Cloudflare Worker session middleware (token refresh + JWT validation)
5. SSO callback handler updates to write to `enterprise_sessions` on login
6. SCIM deprovisioning revocation (after G-001)
7. Audit log integration for all §12.9 events
8. Re-authentication flow for sensitive operations (after core SSO)
9. Admin dashboard session management UI (after G-007)
10. JTI blocklist (Redis) — when first regulated-industry customer requires zero-latency deprovisioning

---

**v0.4 · May 2026 · enterprise-architect + security-engineer**
**Review trigger: any IdP integration change, any SCIM schema change, any session policy change, or quarterly — whichever comes first.**
*v0.2 additions: Section 10 — Magic Link Fallback Security Design. Threat model (8 attack vectors), OTP spec, rate limits, trigger conditions, verification flow, abuse detection, audit events, admin controls, implementation sequencing, GDPR/DPA note. Closes G-010 design phase. Critical implementation dependencies identified.*

*v0.3 additions: Section 11 — Just-in-Time (JIT) Provisioning Design. JIT vs. SCIM decision tree, provisioning flow with seat-limit and domain-verification gates, SAML and OIDC claim extraction mapping, per-tenant `claim_mapping` JSONB config, seat limit enforcement with FOR UPDATE transaction lock to prevent race condition, JIT-to-SCIM reconciliation (409 Conflict → PATCH adopt path), 6 JIT audit events, admin controls (jit_provisioning_enabled, default_role, notify_on_jit_provision), implementation sequencing. Security note: domain verification is not optional.*

*v0.4 additions: Section 12 — Session Token Lifecycle & Refresh Management. Dual-token model (RS256 JWT access token 15min memory-only; opaque UUID refresh token httpOnly cookie database-backed). `enterprise_sessions` table schema with `family_id` + `generation` for token reuse detection. Token family attack protection with full flow diagram and 30-second clock-skew grace window. Enterprise session timeout configuration via `tenant_sso_configs.session_policy` JSONB (`session_timeout_hours`, `idle_timeout_minutes`, `max_concurrent_sessions`). SCIM deprovisioning synchronous session revocation with accepted 15-minute access token residual window and opt-in JTI blocklist for zero-latency requirement. SSO re-authentication for sensitive operations (OIDC `prompt=login`, SAML `ForceAuthn=true`, re-auth assertion pattern). 13 audit events including `session.token_reuse_detected` as security-critical alert signal. Cloudflare Worker middleware architecture, Supabase Auth boundary definition, RS256 monthly key rotation procedure with emergency path, migration from Supabase default sessions (per-tenant flag + re-login trigger). 9 implementation dependencies with recommended sequencing.*

---

## 13. Federated Logout — SAML SLO & OIDC Back-Channel Logout

**Status: Design complete. Implementation pending. Closes G-002 and G-003 design phase.**

---

### 13.1 Why Federated Logout Is Hard

Clicking "Sign out" in FORM clears the FORM session — it invalidates the refresh token in `enterprise_sessions`, expires the access token JWT, and removes the `form_rt` cookie. This is local logout. It is necessary but not sufficient for enterprise security posture.

In an SSO environment there are three distinct session layers:

| Layer | Owner | Cleared by local logout? |
|---|---|---|
| FORM application session | FORM (`enterprise_sessions` table) | Yes |
| IdP SSO session (browser cookie at `login.okta.com`, etc.) | The Identity Provider | No |
| Other SP sessions (other apps the user is also logged into via the same IdP SSO session) | Each SP independently | No |

When an employee is terminated, the correct sequence is: HR deactivates the account in the IdP directory → SCIM signals FORM → FORM revokes sessions (§12.7). But if an employee with an active browser session chooses to log out of FORM themselves — without an upstream deactivation — the IdP session remains alive. Any other application that trusts that IdP session can still authenticate the user silently. FORM's local session is gone, but the employee could re-authenticate to FORM within seconds using the still-active IdP SSO session.

Federated logout addresses this by propagating the logout signal from FORM outward to the IdP, and from the IdP outward to all other SPs registered in the same SSO federation.

Two complementary protocols handle this:

- **SAML 2.0 Single Logout (SLO)** — synchronous, browser-mediated (HTTP-Redirect or HTTP-POST binding) or asynchronous (SOAP binding, not implemented here). FORM sends a signed `LogoutRequest` to the IdP's SLO endpoint; the IdP propagates and returns a `LogoutResponse`.
- **OIDC Back-Channel Logout** (RFC 8414 / OpenID Connect Back-Channel Logout 1.0) — the IdP POSTs a signed `logout_token` directly to FORM's registered back-channel logout URI, without browser involvement. FORM validates and revokes.

These are complementary, not alternative. A tenant using SAML 2.0 uses SLO. A tenant using OIDC uses back-channel logout. Some IdPs support both protocols; in that case FORM applies the protocol matching the tenant's configured SSO protocol.

Neither protocol guarantees atomicity across all SPs. The design in this section never blocks the local logout on the federated outcome. The FORM session is always revoked first, unconditionally. Federation is a best-effort propagation layer on top of a guaranteed local revocation.

---

### 13.2 SAML 2.0 Single Logout (SLO)

#### 13.2.1 SP-Initiated SLO

SP-initiated SLO begins when the user explicitly logs out from within FORM. The flow is:

```
User clicks "Sign out" in FORM
          |
          v
FORM immediately revokes local session (enterprise_sessions UPDATE, §12.7.2 pattern)
          |
          v
FORM constructs signed LogoutRequest
          |
          v
FORM redirects browser to IdP SLO endpoint (HTTP-Redirect binding)
          |
          v
IdP validates LogoutRequest signature
          |
          v
IdP propagates LogoutRequest to all other SPs that share the SSO session
          |
          v
IdP terminates its own SSO session
          |
          v
IdP redirects browser back to FORM's SLO endpoint with signed LogoutResponse
          |
          v
FORM validates LogoutResponse signature
          |
          v
FORM redirects user to post-logout landing page (/login?logged_out=1)
```

The local session revocation (step 2) happens before the federated round-trip. The user cannot re-authenticate to FORM during the SLO round-trip because their `form_rt` cookie is already invalid.

##### LogoutRequest XML Structure

```xml
<samlp:LogoutRequest
  xmlns:samlp="urn:oasis:names:tc:SAML:2.0:protocol"
  xmlns:saml="urn:oasis:names:tc:SAML:2.0:assertion"
  ID="_form_logout_{uuid_v4}"
  Version="2.0"
  IssueInstant="{ISO-8601-UTC}"
  Destination="{idp_slo_url}"
  NotOnOrAfter="{IssueInstant + 2 minutes}">
  <saml:Issuer>https://form.coach/saml/{tenant_id}</saml:Issuer>
  <saml:NameID
    Format="urn:oasis:names:tc:SAML:2.0:nameid-format:persistent"
    SPNameQualifier="https://form.coach/saml/{tenant_id}"
    NameQualifier="{idp_entity_id}">
    {name_id_value}
  </saml:NameID>
  <samlp:SessionIndex>{session_index_from_authn_assertion}</samlp:SessionIndex>
</samlp:LogoutRequest>
```

Key fields:

- `ID`: must be unique per request; stored in a short-lived cache (Redis TTL 5 minutes) to match against the `InResponseTo` field on the `LogoutResponse`. Prevents response replay.
- `Destination`: the IdP's SLO endpoint URL, read from `tenant_sso_configs.slo_url` (new column, see §13.8).
- `NameID`: the persistent identifier stored in `enterprise_sessions.idp_name_id` at session creation time. Must use the same `Format` that the IdP issued in the `AuthnResponse`.
- `SessionIndex`: the IdP-side session identifier from the original `AuthnResponse` assertion, stored in `enterprise_sessions.idp_session_id` (new column, see §13.8). Required so the IdP can target the correct SSO session when the user may have multiple active sessions.
- `NotOnOrAfter`: 2-minute validity window. The IdP must process the request before this timestamp.

##### HTTP-Redirect Binding

The SAML HTTP-Redirect binding encodes the `LogoutRequest` as follows:

1. Deflate-compress the XML (RFC 1951 raw deflate, no zlib wrapper).
2. Base64-encode the compressed bytes (standard alphabet, no line breaks).
3. URL-encode the result.
4. Construct the query string: `SAMLRequest={encoded}&SigAlg=http%3A%2F%2Fwww.w3.org%2F2001%2F04%2Fxmldsig-more%23rsa-sha256&RelayState={opaque_state}`.
5. Sign the entire query string (not just `SAMLRequest`) using the FORM SP private key (same RS256 key pair as `AuthnRequest`, see §12.10.4). The signature covers the literal query string bytes.
6. Append `&Signature={base64url_signature}`.

The `RelayState` carries an opaque token (max 80 bytes per SAML spec) that FORM uses to recover the post-logout redirect path when the IdP returns.

**HTTP-POST binding:** HTTP-POST binding avoids the URL-length limitation and allows the full XML to be transmitted in a form body. It is preferred when the IdP supports it, as query string signatures in HTTP-Redirect are vulnerable to query parameter reordering by intermediate proxies. FORM prefers HTTP-POST for `LogoutRequest` where both sides support it. The binding to use is negotiated at SSO configuration time and stored in `tenant_sso_configs` alongside the existing `binding` field.

Both `LogoutRequest` (outbound) and `LogoutResponse` (inbound) must be signed. The signature algorithm is RS256 (`http://www.w3.org/2001/04/xmldsig-more#rsa-sha256`). FORM rejects any unsigned or HMAC-signed SLO messages.

#### 13.2.2 IdP-Initiated SLO

IdP-initiated SLO occurs without user action in FORM. The IdP sends an unsolicited `LogoutRequest` — typically triggered by a user logging out of another SP in the same federation, or by an admin terminating an IdP session from the IdP admin console.

FORM's SLO endpoint: `POST /auth/saml/slo` (HTTP-POST binding preferred) or `GET /auth/saml/slo` (HTTP-Redirect binding).

Processing steps:

1. Parse and validate the `LogoutRequest` XML:
   - Verify XML signature against the IdP's signing certificate (from `tenant_sso_configs.idp_cert`).
   - Check `IssueInstant` is within ±5 minutes of `now()` (clock skew tolerance, consistent with §12.5 assertion window).
   - Confirm `Destination` matches `https://api.form.coach/auth/saml/slo`.
   - Confirm `Issuer` matches `tenant_sso_configs.idp_entity_id`.
2. Extract `NameID` value and `SessionIndex` from the request.
3. Look up matching sessions in `enterprise_sessions`:
   ```sql
   UPDATE enterprise_sessions
   SET
       revoked_at        = now(),
       revocation_reason = 'federated_logout'
   WHERE
       tenant_id         = $tenant_id
       AND idp_name_id   = $name_id
       AND idp_session_id = $session_index
       AND revoked_at    IS NULL
   RETURNING session_id;
   ```
   If `SessionIndex` is absent from the request (some IdPs omit it), fall back to revoking all active sessions for the `NameID` within the tenant.
4. Write audit events (§13.7).
5. Construct and return a signed `LogoutResponse`:

```xml
<samlp:LogoutResponse
  xmlns:samlp="urn:oasis:names:tc:SAML:2.0:protocol"
  xmlns:saml="urn:oasis:names:tc:SAML:2.0:assertion"
  ID="_form_logoutresp_{uuid_v4}"
  Version="2.0"
  IssueInstant="{ISO-8601-UTC}"
  Destination="{idp_slo_response_url}"
  InResponseTo="{LogoutRequest_ID}">
  <saml:Issuer>https://form.coach/saml/{tenant_id}</saml:Issuer>
  <samlp:Status>
    <samlp:StatusCode Value="urn:oasis:names:tc:SAML:2.0:status:Success"/>
  </samlp:Status>
</samlp:LogoutResponse>
```

If validation fails (invalid signature, expired request, unknown `Issuer`), return a `LogoutResponse` with `StatusCode` `urn:oasis:names:tc:SAML:2.0:status:Responder` and write a `slo.failed` audit event. Do not return HTTP 4xx for SAML-level errors — the SAML protocol requires a `LogoutResponse` even for failures; HTTP errors are reserved for transport-level problems (malformed POST body, etc.).

#### 13.2.3 SLO State Machine

The SLO flow for SP-initiated logout passes through six states:

| State | Description | Transitions |
|---|---|---|
| `Idle` | No logout in progress | → `LogoutInitiated` (user clicks sign out) |
| `LogoutInitiated` | Local session revoked; `LogoutRequest` being constructed | → `WaitingIdPResponse` (request sent); → `Complete` (IdP SLO not configured — local-only path) |
| `WaitingIdPResponse` | `LogoutRequest` dispatched to IdP; awaiting redirect-back | → `PropagatingToOtherSPs` (IdP confirms propagating); → `SessionsRevoked` (IdP returns `LogoutResponse`); → `Failed` (10s timeout) |
| `PropagatingToOtherSPs` | IdP is propagating logout to other SPs in the federation | → `SessionsRevoked` (propagation complete, IdP sends final `LogoutResponse`) |
| `SessionsRevoked` | `LogoutResponse` received and validated; all FORM sessions revoked | → `Complete` |
| `Complete` | User redirected to post-logout page; SLO flow closed | Terminal |
| `Failed` | IdP timeout, invalid signature, or unsupported SLO | → triggers fallback (§13.2.4); user still redirected |

State is tracked in a short-lived Redis key `slo:{tenant_id}:{slo_request_id}` with TTL of 15 seconds. State is not persisted to the database — it is ephemeral coordination state for a single browser round-trip. If Redis is unavailable, the SLO flow degrades to the local-only fallback (§13.2.4).

#### 13.2.4 Failure Modes and Graceful Degradation

Three failure modes are defined. In every case, the FORM local session has already been revoked before any IdP communication begins. Failure never leaves the user with a live FORM session.

| Failure | Condition | Response |
|---|---|---|
| IdP SLO not configured | `tenant_sso_configs.slo_url IS NULL` | Skip federated SLO entirely. Write `slo.fallback_local_only` audit event. Redirect user normally. |
| IdP SLO timeout | No `LogoutResponse` received within 10 seconds of sending `LogoutRequest` | Abort SLO round-trip. Write `slo.failed` + `slo.fallback_local_only` audit events. Redirect user normally. Log warning to operational monitoring. |
| Partial SP propagation failure | IdP returns `LogoutResponse` with `StatusCode` indicating partial success (non-`Success` status) | Accept the response, write `slo.completed` with `partial: true` flag in metadata. FORM has revoked its own session; other SPs' failure is outside FORM's control. |

The 10-second timeout on IdP SLO response is implemented as a Worker `AbortController` signal wrapping the redirect-back wait. If the IdP round-trip has not completed within the window, the Worker redirects the user to the post-logout page and writes the failure events asynchronously. User experience is not blocked by a non-responsive IdP.

FORM never retries a failed SLO round-trip. Retry introduces the risk of the IdP processing the same `LogoutRequest` twice. Session revocation on the FORM side is already complete; the SLO retry would only affect the IdP-side session, which is outside FORM's security boundary.

---

### 13.3 OIDC Back-Channel Logout

Back-channel logout (OpenID Connect Back-Channel Logout 1.0) allows the IdP to notify FORM of a session termination without browser involvement. This is the preferred mechanism for OIDC tenants because it is not subject to browser cookie restrictions and is not visible in browser history.

#### 13.3.1 Registration

FORM registers a back-channel logout URI with each OIDC IdP at tenant onboarding time:

```
https://api.form.coach/auth/oidc/backchannel-logout
```

This is a single shared endpoint. Tenant routing is determined by the `iss` (issuer) claim in the `logout_token` — each IdP has a unique issuer URL that maps to one or more `tenant_sso_configs` rows. If a single IdP serves multiple FORM tenants (multi-tenant IdP like Azure AD with multiple Entra tenants), the `aud` claim disambiguates: FORM's `client_id` is unique per FORM tenant.

The back-channel logout URI is registered in the IdP's application/client configuration alongside `redirect_uri` and `post_logout_redirect_uri`. It is not a protected endpoint — it does not require a bearer token. Authentication is provided by the signed `logout_token` JWT itself.

#### 13.3.2 logout_token Validation

When the IdP POSTs to the back-channel logout endpoint, the request body is `application/x-www-form-urlencoded` with a single parameter: `logout_token`.

The `logout_token` is a signed JWT. FORM performs the following validation checks in order:

| Check | Requirement | Failure action |
|---|---|---|
| JWT structure | Must be a three-part base64url-encoded JWT | Return HTTP 400 |
| Signature | Must be signed by the IdP's public key (fetched from IdP JWKS endpoint, cached with 5-minute TTL) | Return HTTP 400 |
| `iss` | Must match a known `tenant_sso_configs.oidc_issuer` value | Return HTTP 400 |
| `aud` | Must include FORM's `client_id` for the relevant tenant | Return HTTP 400 |
| `iat` | Must be within ±5 minutes of `now()` | Return HTTP 400 |
| `nonce` | MUST NOT be present (spec requirement — distinguishes logout_token from ID token) | Return HTTP 400 |
| `events` claim | Must contain `{"http://schemas.openid.net/event/backchannel-logout": {}}` | Return HTTP 400 |
| `sub` or `sid` | At least one of `sub` (subject) or `sid` (session ID) must be present | Return HTTP 400 |
| Feature enabled | `tenant_sso_configs.backchannel_logout_enabled` must be true for the tenant | Return HTTP 501 |

On successful validation, FORM writes a `backchannel_logout.validated` audit event and proceeds to revocation.

#### 13.3.3 Revocation

Session lookup uses whichever identifiers are present in the `logout_token`:

```sql
UPDATE enterprise_sessions
SET
    revoked_at        = now(),
    revocation_reason = 'federated_logout'
WHERE
    tenant_id  = $tenant_id
    AND revoked_at IS NULL
    AND (
        -- Prefer sid match (precise — targets one SSO session)
        (idp_session_id = $sid AND $sid IS NOT NULL)
        OR
        -- Fall back to sub match (broader — revokes all sessions for this user)
        (user_id = (
            SELECT id FROM users
            WHERE tenant_id = $tenant_id AND idp_subject = $sub
        ) AND $sid IS NULL)
    )
RETURNING session_id;
```

The `idp_session_id` column stores the OIDC `sid` claim from the ID token at login time, if the IdP provided it. The `idp_subject` column on the `users` table stores the OIDC `sub` claim and is indexed.

If `sid` is present in the `logout_token`, FORM targets the specific SSO session — a user who is logged into FORM from two devices with two separate OIDC sessions will have only the correct device session revoked. If only `sub` is present, all active FORM sessions for that user within the tenant are revoked.

#### 13.3.4 Response and Timing

FORM must return an HTTP response to the IdP within a bounded time window. The specification requires the response before the IdP connection times out (typically 5–30 seconds depending on IdP).

FORM's SLA for this endpoint: **return HTTP 200 within 2 seconds**. The response indicates acknowledgement of the request, not completion of revocation.

For tenants with a small number of active sessions per user (typical case), revocation is synchronous — the SQL runs before the 200 is returned. For tenants with a large number of sessions (e.g., shared service accounts with many concurrent sessions), revocation is dispatched to a background queue and the 200 is returned immediately. The queue processes the revocation within 5 seconds in normal operation.

Response codes:

| HTTP status | Meaning |
|---|---|
| `200 OK` | `logout_token` accepted; revocation initiated or complete |
| `400 Bad Request` | `logout_token` malformed, invalid signature, failed a validation check |
| `501 Not Implemented` | Back-channel logout not enabled for this tenant |

FORM does not return `503` for transient database errors — if the revocation fails for a transient reason, FORM still returns `200` and the background queue retries the SQL up to 3 times with exponential backoff. The rationale: returning `503` to the IdP may cause the IdP to retry the `logout_token`, and retry logic on the IdP side varies wildly by vendor. It is safer to accept the token, guarantee at-least-once processing internally, and avoid confusing IdPs with unexpected error responses.

If all retries fail, a `slo.failed` audit event is written with `reason: db_revocation_failed` and an operational alert is raised.

---

### 13.4 OIDC Front-Channel Logout (Not Implemented)

OIDC also defines a front-channel logout mechanism (OpenID Connect Front-Channel Logout 1.0) where the IdP embeds an `<iframe>` or `<img>` pointing to each SP's logout URI in the browser, causing the SP to receive a request with the session cookie in context.

FORM does not implement front-channel logout. The reasons:

1. **SameSite cookie restrictions.** FORM's `form_rt` cookie is set with `SameSite=Strict`. A cross-origin iframe or image request from the IdP's domain will not carry the `form_rt` cookie. Front-channel logout therefore cannot reliably identify the FORM session to revoke.
2. **Session state in URL.** Front-channel logout URI parameters may contain `sid` values that appear in browser history and server access logs, creating a minor but avoidable information disclosure.
3. **Browser blocking.** Third-party cookie deprecation in Chrome and Safari means cross-origin requests from iframes increasingly fail to carry session state, making front-channel logout unreliable regardless of `SameSite` settings.
4. **Back-channel is strictly better.** For OIDC tenants, back-channel logout (§13.3) achieves the same goal reliably without browser involvement. There is no scenario where front-channel provides a capability that back-channel does not.

This decision closes G-003 for OIDC tenants: the implementation path is back-channel only. Any IdP requirement for front-channel logout is a blocker to onboarding that IdP configuration — it must be documented in the enterprise contract and the customer directed to configure back-channel logout on their IdP instead.

Reference: G-002 (SAML SLO) and G-003 (OIDC back-channel) are the two approved federated logout mechanisms. Front-channel is explicitly rejected and will not be revisited without a formal security review demonstrating that SameSite and third-party cookie restrictions no longer apply.

---

### 13.5 Session Revocation Mechanics

#### 13.5.1 Integration with §12 enterprise_sessions Table

Both SLO and back-channel logout converge on the same revocation pattern used by SCIM deprovisioning (§12.7.2), with a distinct `revocation_reason` value to distinguish the trigger in the audit trail:

```sql
-- Federated logout revocation (SLO or back-channel)
UPDATE enterprise_sessions
SET
    revoked_at        = now(),
    revocation_reason = 'federated_logout'
WHERE
    tenant_id  = $tenant_id
    AND (
        idp_name_id    = $name_id      -- SAML path
        OR idp_session_id = $session_id -- OIDC back-channel path (sid)
        OR user_id     = $user_id      -- fallback: sub-only back-channel
    )
    AND revoked_at IS NULL
RETURNING session_id, user_id;
```

The `revocation_reason` values used across the codebase are:

| Value | Trigger |
|---|---|
| `'scim_deprovisioned'` | SCIM `PATCH active=false` or `DELETE` (§12.7) |
| `'federated_logout'` | SAML SLO or OIDC back-channel logout (§13) |
| `'idle_timeout'` | Lazy or sweeper idle timeout enforcement (§12.6.3) |
| `'session_timeout'` | Session exceeded absolute TTL (§12.6) |
| `'token_reuse_detected'` | Token family attack detected (§12.4) |
| `'admin_revoked'` | Tenant admin explicitly revoked the session |
| `'key_rotation_invalidation'` | Emergency RS256 key rotation (§12.10.4) |
| `'user_logout'` | Normal user-initiated local logout (no SLO) |

#### 13.5.2 JTI Blocklist Interaction

When `jti_revocation_check: true` is set in `session_policy` (opt-in per §12.7.4), revocation of a session via federated logout must also blocklist any in-flight access tokens whose parent session was just revoked.

The `RETURNING session_id, user_id` clause from the revocation query provides the session IDs. For each revoked session, the Worker must look up any access tokens issued for that session that have not yet expired. Because FORM does not store issued access tokens in the database (they are stateless JWTs), the blocklist approach is:

1. The session record in `enterprise_sessions` contains `last_access_token_jti` and `last_access_token_exp` — two columns that are updated on every token refresh (see §12.5 for schema context). These are the only in-flight access token candidates.
2. For each revoked session, INSERT `last_access_token_jti` into the JTI blocklist (Redis) with TTL = `last_access_token_exp - now()`. If `last_access_token_exp < now()`, the token has already expired and no blocklist entry is needed.

This requires two additional columns on `enterprise_sessions`:

```sql
ALTER TABLE enterprise_sessions
  ADD COLUMN last_access_token_jti  TEXT,
  ADD COLUMN last_access_token_exp  TIMESTAMPTZ;
```

These columns are updated atomically with each token rotation. The existing index `idx_sessions_user_tenant` covers the revocation query; no additional index is needed for JTI blocklist insertion (it is a primary key lookup by `session_id`).

#### 13.5.3 SCIM Deprovisioning vs. Federated Logout

These are different triggers using the same revocation mechanism:

| Attribute | SCIM deprovisioning (§12.7) | Federated logout (§13) |
|---|---|---|
| Trigger | HR/IdP directory event (account deletion/deactivation) | User or IdP initiated logout action |
| Revocation scope | All sessions for `user_id` + `tenant_id` | Specific `idp_session_id` or `idp_name_id` (may be partial) |
| `revocation_reason` | `'scim_deprovisioned'` | `'federated_logout'` |
| User re-authentication allowed? | No — account is deactivated | Yes — user can log in again immediately |
| Synchronous requirement | Yes — must complete before SCIM 200 response | Best-effort within 2s for back-channel; synchronous for SLO |
| User record soft-deleted? | On `DELETE`: yes. On `PATCH active=false`: no | Never — logout does not delete the user record |

The revocation SQL path is shared; only the `WHERE` clause target and `revocation_reason` value differ.

---

### 13.6 SAML NameID Format Handling

#### 13.6.1 Persistent NameID (Required)

FORM requires IdPs to use the persistent NameID format:

```
urn:oasis:names:tc:SAML:2.0:nameid-format:persistent
```

A persistent NameID is an opaque, durable identifier that the IdP assigns to the user for a specific SP. It does not change across sessions or after the user changes their email address. This is the correct format for user identity continuity in a long-lived SaaS product.

FORM stores the persistent NameID in `enterprise_sessions.idp_name_id` at session creation and in `users.idp_name_id` at JIT provisioning time. Logout requests use this stored value — they do not rely on any mutable user attribute.

The `AuthnRequest` should request this format explicitly:

```xml
<samlp:NameIDPolicy
  Format="urn:oasis:names:tc:SAML:2.0:nameid-format:persistent"
  AllowCreate="false"/>
```

`AllowCreate="false"` tells the IdP not to create a new user — FORM handles provisioning through JIT (§11) or SCIM (§6). The IdP should only assert identity for users that already exist in its directory.

#### 13.6.2 Transient NameID Fallback

Some legacy IdPs (and misconfigured modern ones) issue transient NameIDs:

```
urn:oasis:names:tc:SAML:2.0:nameid-format:transient
```

Transient NameIDs change on every authentication — they cannot be used as a stable user identifier, and they cannot be used for SLO (the SLO `LogoutRequest` must carry the NameID value the IdP issued, but that value has no meaning after the session ends).

FORM's fallback chain when a transient NameID is detected:

1. **Attempt email attribute lookup.** Check the assertion's attribute statement for an email address claim (common attribute names: `email`, `mail`, `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress`). If found, use the email to look up the FORM user: `SELECT id FROM users WHERE email = $email AND tenant_id = $tenant_id`.
2. **Attempt persistent identifier attribute.** Some IdPs that issue transient NameIDs also provide a persistent identifier as a separate attribute (e.g., `employeeID`, `objectGUID`). Check `tenant_sso_configs.claim_mapping` for a configured persistent identifier attribute.
3. **Fail authentication** if neither of the above resolves to a unique, unambiguous FORM user. Write an audit event `sso.authn_failed` with `reason: transient_nameid_unresolvable`. Do not create a new user record based on a transient NameID.

When a transient NameID is used, SLO is functionally impaired:

- SP-initiated SLO: the `LogoutRequest` must carry the transient NameID value from the active session. FORM stores it in `enterprise_sessions.idp_name_id` per-session, so the SLO request can be constructed for the current session. However, the IdP may not be able to look up the session by transient NameID if it has already rotated the value internally.
- IdP-initiated SLO: if the IdP sends a `LogoutRequest` with a transient NameID, FORM can look up sessions only by exact match on the stored transient value. This is reliable within a single session but will not match sessions from prior logins.

The recommendation is to require persistent NameID as a condition of enterprise SSO onboarding. Tenants whose IdPs cannot issue persistent NameIDs should be offered a fallback configuration path, documented as a limitation in the customer's enterprise agreement, with an acknowledgement that SLO reliability is reduced.

#### 13.6.3 Email-Format NameID

A third format is occasionally seen:

```
urn:oasis:names:tc:SAML:1.1:nameid-format:emailAddress
```

This uses the user's email address as the NameID value. It is stable as long as the user's email address does not change, but it is mutable — an email address change at the IdP will break the NameID→user lookup. FORM accepts this format but logs a warning at JIT provisioning time and stores the email-format NameID in `idp_name_id`. SLO operates normally for email-format NameIDs as long as the email address has not changed.

---

### 13.7 Audit Events

All events are written to the HMAC-chained `audit_log` table per DEC-030. No PII in metadata. Timestamps, identifiers, and boolean outcomes only.

| Event name | Trigger | Key metadata |
|---|---|---|
| `slo.sp_initiated` | User triggered logout from FORM; local session revoked; SLO round-trip beginning | `session_id`, `tenant_id`, `user_id`, `slo_request_id`, `idp_slo_url` |
| `slo.idp_initiated` | IdP pushed a `LogoutRequest` to FORM's SLO endpoint | `tenant_id`, `idp_name_id` (stored — not raw PII), `session_index`, `logout_request_id`, `sessions_revoked_count` |
| `slo.completed` | SP-initiated SLO round-trip completed; valid `LogoutResponse` received from IdP | `session_id`, `tenant_id`, `slo_request_id`, `duration_ms`, `partial` (boolean) |
| `slo.failed` | SLO flow error — IdP timeout, invalid `LogoutResponse` signature, unexpected `StatusCode`, or DB revocation failure | `session_id`, `tenant_id`, `slo_request_id`, `reason` (`idp_timeout`, `invalid_signature`, `status_code_{value}`, `db_revocation_failed`) |
| `slo.fallback_local_only` | Federated SLO unavailable (not configured or timed out); local session cleared, no IdP propagation | `session_id`, `tenant_id`, `user_id`, `reason` (`slo_not_configured`, `idp_timeout`) |
| `backchannel_logout.received` | `logout_token` received at `/auth/oidc/backchannel-logout`; not yet validated | `tenant_id` (derived from `iss`), `iss`, `request_id` |
| `backchannel_logout.validated` | `logout_token` passed all validation checks (§13.3.2) | `tenant_id`, `iss`, `sub_hash` (SHA-256 of `sub`, not the raw value), `sid`, `request_id` |
| `backchannel_logout.revoked` | Sessions successfully revoked following back-channel logout | `tenant_id`, `sessions_revoked_count`, `revocation_method` (`sid` or `sub`), `request_id`, `async` (boolean — whether revocation was deferred to queue) |

The `sub_hash` in `backchannel_logout.validated` is the SHA-256 of the OIDC `sub` claim. This allows correlation across audit events (e.g., matching a `backchannel_logout.validated` to a prior `session.created` event that also stores a `sub_hash`) without storing the raw OIDC subject identifier, which may be considered PII in some jurisdictions.

Where `slo.failed` co-occurs with `slo.fallback_local_only`, both events are written. The `fallback_local_only` event confirms that the user's FORM session was cleared regardless of the SLO failure.

---

### 13.8 Implementation Dependencies and Sequencing

| Dependency | Status | Blocking concern | Notes |
|---|---|---|---|
| SAML library SLO support | Not verified | Must confirm before committing to SAML SLO implementation | `node-saml` supports SLO as of v4.x — verify `LogoutRequest` construction and `LogoutResponse` parsing. `samlify` is an alternative but has had historical CVEs in XML parsing. Do not implement custom XML signing. |
| `tenant_sso_configs.slo_url` column | Not implemented | Required for SP-initiated SLO | `ALTER TABLE tenant_sso_configs ADD COLUMN slo_url TEXT;` — nullable; NULL means SLO not configured for tenant |
| `tenant_sso_configs.slo_binding` column | Not implemented | Required | `ALTER TABLE tenant_sso_configs ADD COLUMN slo_binding TEXT CHECK (slo_binding IN ('HTTP-POST', 'HTTP-Redirect'));` |
| `tenant_sso_configs.backchannel_logout_enabled` column | Not implemented | Required for back-channel routing | `ALTER TABLE tenant_sso_configs ADD COLUMN backchannel_logout_enabled BOOLEAN NOT NULL DEFAULT false;` |
| `enterprise_sessions.idp_name_id` column | Not implemented | Required for SAML SLO lookup | `ALTER TABLE enterprise_sessions ADD COLUMN idp_name_id TEXT;` — populated at session creation from SAML assertion `NameID` |
| `enterprise_sessions.idp_session_id` column | Not implemented | Required for SAML `SessionIndex` and OIDC `sid` | `ALTER TABLE enterprise_sessions ADD COLUMN idp_session_id TEXT;` — populated from SAML `SessionIndex` or OIDC `sid` claim at session creation |
| `enterprise_sessions.last_access_token_jti` and `last_access_token_exp` | Not implemented | Required for JTI blocklist path (opt-in) | Only needed if `jti_revocation_check: true`; can defer until first regulated-industry customer |
| SLO state ephemeral store (Redis / Cloudflare KV) | Not implemented | Required for SP-initiated SLO request/response correlation | Redis key `slo:{tenant_id}:{slo_request_id}`, TTL 15 seconds. Cloudflare KV acceptable if Redis is not available (higher latency but sufficient for 15s TTL) |
| `logout_token` validation library | Not implemented | Required for back-channel logout | Any RFC 7519-compliant JWT library with RS256 + ES256 support. `jose` (npm) is already used for access token validation per §12 — reuse it. |
| Async queue for high-volume back-channel requests | Not implemented | Required for tenants with large concurrent session counts | Cloudflare Queue or Upstash Queue. Not required for initial launch if max sessions per user is bounded (e.g., `max_concurrent_sessions` ≤ 10 from `session_policy`). |

**Recommended implementation order:**

1. Schema migrations: `slo_url`, `slo_binding`, `backchannel_logout_enabled` on `tenant_sso_configs`; `idp_name_id`, `idp_session_id` on `enterprise_sessions`.
2. Update SSO callback handlers (SAML and OIDC) to populate `idp_name_id` and `idp_session_id` on session creation.
3. Verify `node-saml` SLO support. Write integration test against Okta developer sandbox.
4. Implement IdP-initiated SLO endpoint (`POST /auth/saml/slo`) — simpler than SP-initiated; no redirect loop.
5. Implement SP-initiated SLO — adds the redirect round-trip and state correlation.
6. Implement back-channel logout endpoint (`POST /auth/oidc/backchannel-logout`) — self-contained; no browser state.
7. Audit event integration for all §13.7 events.
8. Admin dashboard: SLO configuration UI (show `slo_url` field, toggle `backchannel_logout_enabled`). Depends on G-007.
9. JTI blocklist integration for federated logout path (after or alongside §12 JTI blocklist work).
10. Async queue for back-channel revocation — when first customer has `max_concurrent_sessions > 10`.

---

### 13.9 Gap Closure

| Gap | Previous status | Updated status | Notes |
|---|---|---|---|
| G-002 — SAML SLO | 🔴 Not designed | 🟡 Design complete, implementation pending | SP-initiated and IdP-initiated flows fully specified. SLO state machine defined. Failure modes and graceful degradation specified. Blocked on: `node-saml` SLO verification, schema migrations, SSO callback handler updates. |
| G-003 — OIDC back-channel logout | 🔴 Not designed | 🟡 Design complete, implementation pending | Back-channel logout endpoint, `logout_token` validation, `enterprise_sessions` revocation path fully specified. Front-channel explicitly rejected. Blocked on: schema migrations, SSO callback handler updates. |

**SOC 2 CC6.1 impact:** CC6.1 requires that access to systems is restricted based on logical access controls, and that those controls include procedures for timely revocation. The current state (local-only logout without IdP session propagation) is a partial control. Implementing §13 closes the gap between FORM session revocation and IdP session revocation, satisfying the "timely revocation across all access paths" requirement of CC6.1. This section, once implemented, enables the auditor finding for CC6.1 to move from "compensating control documented" to "control implemented and evidenced."

Full closure of G-002 and G-003 (i.e., moving from 🟡 to 🟢) requires:
- Implementation complete and deployed to production.
- Integration tests passing against at least one IdP per protocol (Okta for SAML SLO, Azure AD for OIDC back-channel).
- Audit events firing and queryable in the admin dashboard audit log viewer.

---

### 13.10 Open Questions

**Q1: What happens if a customer's IdP does not support SLO or back-channel logout?**

Not all IdPs support these protocols, particularly older on-premises deployments (ADFS pre-2016, some legacy Shibboleth configurations). When an enterprise customer's IdP does not support SLO:

- FORM falls back to the local-only logout path (§13.2.4 fallback).
- The limitation is documented in the enterprise contract: "Federated logout propagation requires IdP SLO support (SAML) or back-channel logout support (OIDC). Where the customer's IdP does not support these protocols, FORM will revoke its own session on user logout; the customer's IdP SSO session will remain active until it expires per the IdP's own session timeout policy."
- FORM's session timeout policy (§12.6) acts as the backstop: if the IdP session expires before the user re-authenticates to FORM, access is naturally terminated. For tenants without SLO, it is recommended to set a shorter `session_timeout_hours` in `session_policy` (e.g., 8 hours instead of 168) to reduce the window of exposure.
- This is a contractual and security posture decision, not a FORM implementation decision. The customer acknowledges the limitation at onboarding.

**Q2: ADFS SLO certificate requirements.**

Microsoft ADFS requires the SP's SLO `LogoutRequest` to be signed with the same certificate that is registered in the ADFS relying party trust configuration. ADFS does not support JWKS-based certificate discovery — the SP certificate must be uploaded to the ADFS console as a static X.509 certificate (PEM or DER format).

This has implications for FORM's key rotation procedure (§12.10.4):

- The RS256 key pair used for SAML signing must be accompanied by a self-signed X.509 certificate wrapper (the key pair is the same; the certificate wraps the public key in X.509 format).
- When FORM rotates its signing key pair, ADFS customers must manually update the SP certificate in their ADFS relying party trust configuration before the old certificate expires.
- FORM must notify ADFS customers at least 30 days before the signing certificate's `notAfter` date. This requires a separate certificate expiry monitoring job that is aware of which tenants are using ADFS (identifiable by the IdP metadata format or a `tenant_sso_configs.idp_type = 'adfs'` flag).

**Decision required (security-engineer + enterprise-architect):** Should FORM maintain a separate, longer-lived (2-year) signing certificate for ADFS tenants, distinct from the monthly-rotated RS256 key pair used for JWTs? The tradeoff is operational simplicity for ADFS customers (fewer certificate update events) vs. a longer exposure window if the ADFS signing certificate is compromised. Recommendation: use a 1-year certificate for ADFS tenants, with a 60-day advance notice to the customer for each rotation. This aligns with the G-004 certificate rotation automation work and should be addressed in the same implementation sprint.

**Q3: Behaviour when a logout_token `sid` does not match any known enterprise_sessions.idp_session_id.**

This can occur legitimately if: (a) the user authenticated via a path that did not populate `idp_session_id` (e.g., before the §13.8 schema migrations were deployed), or (b) the `sid` in the logout_token refers to an IdP session that was authenticated before FORM's back-channel logout feature was enabled for the tenant.

In this case, FORM has no session to revoke. FORM should return HTTP 200 (not 400) — the IdP's instruction is valid; FORM simply has no matching session. A `backchannel_logout.received` event is written, but no `backchannel_logout.revoked` event follows. An operational metric should track the count of no-match back-channel requests per tenant to detect misconfiguration.

**Q4: Multi-device logout scope for OIDC back-channel when only `sub` is present.**

If the IdP sends a logout_token with only `sub` (no `sid`), FORM revokes all active sessions for that user within the tenant. This is intentional per the spec: the absence of `sid` means the IdP is revoking all sessions for the subject, not just one specific SSO session. However, this means a user logging out on one device will also lose their session on all other devices, which may surprise users.

This is spec-compliant behaviour and is the correct interpretation. If per-device logout is required, the IdP must include `sid` in its logout tokens. FORM should document this in the IdP configuration guide: "To enable per-device logout (rather than full account logout), configure your IdP to include the `sid` claim in logout tokens."

---

*v0.5 additions: Section 13 — Federated Logout: SAML SLO & OIDC Back-Channel Logout. Closes G-002 design phase (SAML SLO: SP-initiated + IdP-initiated flows, SLO state machine, 6-state lifecycle, graceful degradation on IdP timeout). Closes G-003 design phase (OIDC Back-Channel logout_token validation, RFC-compliant revocation, enterprise_sessions integration). Front-channel logout explicitly rejected (SameSite/browser restrictions). 8 new audit events. idp_session_id column requirement on enterprise_sessions documented. G-002: 🔴 → 🟡 Partial (design done, implementation pending). G-003: 🔴 → 🟡 Partial. SOC 2 CC6.1 partial closure. Implementation sequencing: core SSO → §13 → G-007 admin UI.*

---

## 14. SCIM Data Processing: GDPR Article 28 Compliance Framework

**Owner:** compliance-officer (primary), enterprise-architect (technical review), security-engineer (audit controls review)
**Closes:** G-013 design phase
**Blocking condition lifted when:** compliance-officer forwards §14.3 clause language to outside counsel, counsel approves, and the revised standard DPA template is signed.

### 14.1 Why SCIM Requires a Dedicated DPA Annex

FORM's standard Data Processing Agreement (DPA) covers user-generated data: workouts, meals, coaching sessions, and health profile data. SCIM provisioning introduces a qualitatively different processing activity: FORM now processes **employer-held employee directory data** on behalf of the enterprise customer.

This distinction matters under GDPR Art. 28 for three reasons:

1. **Source of data.** SCIM data originates in the customer's HR/IT systems (Okta, Azure AD, Google Workspace), not from the individual user. The user typically has no direct relationship with FORM at provisioning time — they may not yet have logged in or accepted any FORM privacy notice.

2. **Legal basis at the controller.** The enterprise customer's lawful basis for processing employee directory data through SCIM is Art. 6(1)(b) (performance of an employment contract) or Art. 6(1)(c) (compliance with a legal obligation), not Art. 6(1)(a) (consent). FORM's DPA must reflect this, because the lawful basis chain affects how FORM may process, retain, and delete the data.

3. **Data subjects have separate rights channels.** An employee's right of access (Art. 15), rectification (Art. 16), and erasure (Art. 17) against SCIM-sourced data runs primarily through the **employer** (controller), not through FORM. The DPA must clarify how FORM assists the controller in responding to data subject requests affecting SCIM-provisioned records.

Without an explicit SCIM annex in the DPA, FORM is processing employee directory data without a documented legal basis chain from processor to controller — a finding that would appear in any GDPR audit or enterprise procurement review.

### 14.2 SCIM Personal Data Inventory

The following table defines exactly what personal data FORM processes through SCIM, the retention period for each attribute class, and the legal basis the enterprise customer relies on. This table is incorporated by reference into the DPA Annex (§14.3).

| Attribute class | Specific fields | Retention in active state | Retention after deprovisioning | Legal basis at controller |
|---|---|---|---|---|
| **Identity** | `email` (= `userName`), `displayName`, `givenName`, `familyName` | While `active = true` | 30 days in users table (soft-delete), then audit log hash only · 7 years per DEC-030 | Art. 6(1)(b) employment contract |
| **Account state** | `active` (boolean), `externalId` (IdP-assigned UID) | While `active = true` | Audit log only | Art. 6(1)(b) |
| **Organisational** | `department`, `title`, `costCenter` | While `active = true` | Deleted at deprovisioning; not retained in audit log | Art. 6(1)(b) |
| **Reporting structure** | `manager` (resolved to FORM `user_id`) | While `active = true` | Deleted at deprovisioning | Art. 6(1)(b) |
| **Groups** | Group `displayName`, group `id` (IdP GUID or slug) | While group membership active | Deleted when user removed from group or deprovisioned | Art. 6(1)(b) |
| **SCIM sync log** | Full operation payload (create/update/delete events) | 90 days rolling | Not retained post-deprovisioning | Art. 6(1)(f) legitimate interest (operational debugging) |
| **Audit trail** | Actor (`system`), action (`auth.sso_provisioned`, `tenant.scim_provisioned`), `resource_id` (user UUID), timestamp | Indefinite (HMAC chain) | 7 years from creation · partition-drop pruned per `AUDIT_LOG_SCHEMA.md` retention schedule | Art. 6(1)(c) legal obligation (SOC 2, Art. 30 records) |

**Special category data (GDPR Art. 9) — strict prohibition.** SCIM attribute mapping (§3.3) must reject, at the API layer, any attribute that could constitute special category data: health status, disability, religion, ethnicity, union membership, political opinion. If an enterprise IdP schema extension includes such fields, FORM's SCIM endpoint returns `HTTP 400 Bad Request` with `scimType: "invalidValue"` and logs a `scim.rejected_sensitive_attribute` audit event. No exception to this rule exists. See §14.5 for the enforcement mechanism.

### 14.3 DPA Annex B — SCIM Provisioning Processing Activity (Template Clause)

The following clause language is ready for outside counsel review. compliance-officer forwards this section to counsel for incorporation into FORM's standard DPA template. Until that template is updated and signed, **SCIM provisioning must not be enabled for any enterprise customer** (G-013 block).

---

**ANNEX B TO THE DATA PROCESSING AGREEMENT**
**Processing Activity: SCIM 2.0 User and Group Provisioning**

**1. Subject-matter of processing.**
Automated synchronisation of enterprise directory data (users and groups) to provision, update, and deprovision user accounts within the FORM platform, using the SCIM 2.0 protocol (RFC 7643, RFC 7644).

**2. Duration of processing.**
For the term of the Enterprise Agreement, plus thirty (30) days following expiry or termination to allow deprovisioning completion. Audit log records are retained for seven (7) years from creation, consistent with Processor's SOC 2 Type II obligations and GDPR Article 30 records of processing activities. All other personal data is deleted within thirty (30) days of the end of the processing period.

**3. Nature and purpose of processing.**
Automated execution of SCIM operations (create, read, update, replace, delete) triggered by changes in the Controller's Identity Provider directory, for the purpose of:
(a) Granting FORM platform access to eligible employees without manual onboarding;
(b) Updating employee records (name, role, organisational attributes) to maintain accurate access control;
(c) Revoking FORM platform access promptly when an employee is offboarded or transferred, in support of the Controller's information security obligations.

**4. Types of personal data processed.**
Email address; display name (first and last name); username (IdP-assigned); employee identifier (externalId, if provided); department; job title; cost centre; manager reference (resolved to an internal FORM user identifier); group memberships (group name and IdP-assigned group identifier). Processor will not request, accept, or store any attribute constituting special category data within the meaning of GDPR Article 9, including but not limited to health status, disability, ethnicity, religion, union membership, political opinion, or biometric data. Any such attribute received from the Controller's IdP will be rejected at the API layer and logged as a security event.

**5. Categories of data subjects.**
Employees (and, where applicable, contractors or agency workers) of the Controller who are designated by the Controller for access to the FORM platform.

**6. Controller's obligations specific to SCIM provisioning.**
(a) The Controller is responsible for ensuring that employees designated for SCIM provisioning have been informed, in accordance with GDPR Articles 13–14, that their directory data will be processed by FORM for the purpose described in clause 3 above. FORM's standard user privacy notice (available at form.coach/privacy) covers data processed after first login; GDPR Art. 14 notice for pre-login SCIM provisioning is the Controller's responsibility.
(b) The Controller shall ensure that the SCIM bearer token (§14.6) is treated as a credential of equivalent sensitivity to an administrative password and is rotated at least annually or immediately upon suspected compromise.
(c) The Controller shall configure its IdP to send only the attribute classes listed in clause 4. Custom IdP extension schemas that include special category data must be disabled before SCIM provisioning is activated.

**7. Processor's sub-processors for SCIM data.**
No sub-processors beyond those listed in the main body of this DPA are introduced by SCIM provisioning. SCIM data transits through Cloudflare Workers (zero-retention API layer) and is stored in Supabase (primary database), both of which are listed sub-processors. The Processor will provide thirty (30) days' advance notice of any new sub-processor involved in SCIM data processing.

**8. Assistance with data subject rights.**
The Processor will assist the Controller in responding to data subject rights requests affecting SCIM-provisioned data, as follows:
- **Access (Art. 15):** Processor provides a tenant-admin API endpoint to export all stored attributes for a given `user_id`. The Controller is responsible for fulfilling the subject access request from that export.
- **Rectification (Art. 16):** SCIM PATCH or PUT from the Controller's IdP is the authoritative correction mechanism. Direct corrections via FORM admin dashboard are also supported.
- **Erasure (Art. 17):** See §14.6 of `docs/SSO_SCIM_IMPLEMENTATION.md` for the full erasure procedure. Identity attributes are anonymised; audit log hash-chain integrity is preserved per DEC-030.
- **Restriction (Art. 18):** Processor will suspend all SCIM sync operations for a specific user upon written request from the Controller's data protection contact, pending resolution of the processing dispute.
- **Portability (Art. 20):** Not applicable to SCIM-sourced data, as such data originates with the Controller and is already available in the Controller's IdP system of record.

**[END OF ANNEX B TEMPLATE]**

---

**Outside counsel action required:** Review clauses 4, 6(a), and 8 for jurisdiction-specific language requirements (UK GDPR, Swiss DSG, CCPA B2B exemption). Confirm whether Art. 14 transparency notice obligation in clause 6(a) requires a specific notice template or whether a reference to FORM's privacy policy is sufficient under the applicable supervisory authority's guidance.

### 14.4 Privacy Floor Enforcement for SCIM Attributes

FORM's privacy floor (see `docs/ENTERPRISE.md §Privacy Floor`, `docs/DATA_MODEL.md §6`) prohibits HR-tier tenant admins from accessing individual user data. SCIM-sourced organisational attributes (`department`, `title`, `manager`) are potentially the most sensitive data in the system for this purpose: a manager could use them to identify which individual users have (or have not) engaged with the platform.

**Storage.** SCIM organisational attributes are stored in a dedicated `users.scim_attributes` JSONB column, separate from the core `users` record. This column is:
- Excluded from all tenant-admin query results by default
- Not returned by any HR-tier API endpoint
- Used only by the role-mapping engine (§5.2) at authentication time
- Readable only by `tenant_owner` role and the Processor's support team (with audit log)

**Role mapping opacity.** When SCIM group membership triggers a role assignment (e.g., group `formAdmins` → `tenant_admin` role), the admin dashboard shows the resulting role, not the source group attribute. A tenant admin can see "User X is a tenant_admin" but cannot see "User X is a tenant_admin *because they are in the Azure AD group formAdmins*" unless they are themselves the `tenant_owner` who configured that mapping.

**Aggregate-only wellness reporting.** The admin dashboard wellness dashboard (described in `docs/ENTERPRISE.md §Admin Dashboard`) uses a k-anonymity floor of 5: any aggregate metric cell covering fewer than 5 users is suppressed and displayed as "< 5 users." SCIM organisational attributes (department, title) may be used as dimensions in aggregate reports only if the resulting cell count clears the k-anonymity floor. Dimensions that would identify individuals are never exposed.

**HR admin access prohibition — hard rule.** The `tenant_manager` role may never receive a query response that includes `users.scim_attributes` content, regardless of API path. This constraint is enforced at three layers: (1) RLS policy on the `users` table (`docs/DATA_MODEL.md §3.3`), (2) application-layer field filter in the tenant API middleware, (3) audit alert if `data.read_individual` event is logged for a `tenant_manager` actor (see `AUDIT_LOG_SCHEMA.md`).

### 14.5 Data Minimisation and Attribute Rejection

The SCIM endpoint must reject at the API layer any attribute not in the approved mapping table (§3.3). Rejection is enforced as follows:

**Unknown core attributes.** SCIM core schema attributes not in the mapping table (e.g., `phoneNumbers`, `addresses`, `x509Certificates`, `ims`, `entitlements`) are silently dropped from the request — the operation succeeds, but the unknown attribute is not stored. This matches RFC 7644 §3.3 guidance on partial attribute support.

**Enterprise extension attributes beyond the approved list.** The approved SCIM enterprise extension attributes are: `department`, `costCenter`, `organization`, `division`, `manager`. Any other enterprise extension attribute (e.g., a custom `urn:okta:*` schema extension) is silently dropped.

**Special category data detection.** A pre-storage attribute scanner inspects incoming SCIM payloads for attribute names that suggest special category data, using a blocklist:

```
BLOCKED_ATTRIBUTE_PATTERNS = [
  /health/i, /disability/i, /medical/i, /condition/i,
  /religion/i, /ethnic/i, /race/i, /national.*origin/i,
  /politic/i, /union/i, /biometric/i, /genetic/i,
  /sex.*orient/i, /gender.*ident/i,
  /salary/i, /compensation/i, /pay.*grade/i  # financial — not Art. 9 but excluded
]
```

If a match is found anywhere in the SCIM payload (attribute name or value key), the entire SCIM operation is rejected with `HTTP 400`, `scimType: "invalidValue"`, and a `scim.rejected_sensitive_attribute` audit event is written. The event includes `tenant_id`, `matched_pattern`, and the offending attribute name (not value), so the tenant admin can diagnose and fix their IdP configuration without exposing the sensitive value in logs.

**Minimum required attributes.** If a SCIM `POST /Users` request lacks any of `userName`, `name.givenName + name.familyName` (or `displayName` as fallback), and `active`, the request is rejected with `HTTP 400`, `scimType: "invalidValue"`. These three fields are non-negotiable minimums.

### 14.6 Right to Erasure for SCIM-Provisioned Users (Art. 17 Procedure)

SCIM-provisioned users have a dual-path erasure scenario: their identity is managed by the employer's IdP, but they have independent GDPR rights against FORM.

**Path A — Employer-initiated deprovisioning (SCIM DELETE or PATCH active=false).**

This is the standard offboarding path. It is not a GDPR erasure; it is an access revocation. FORM's response:
1. Sets `users.active = false`, adds `users.deleted_at` timestamp.
2. Revokes all active sessions for the user (§12 session revocation).
3. Retains all user data (workouts, meals, coaching sessions, health profile) for 30 days in the soft-delete state.
4. After 30 days: hard-deletes `user_health_profile`, workout sets, meal logs, coaching turns per `DATA_MODEL.md §8.3`.
5. Anonymises `users` record: `email` replaced with SHA-256 hash (stored only in audit log, not in users table), `display_name` replaced with `Deleted User {short_hash}`, `scim_attributes` column nulled.
6. Audit log entries for the user remain intact with the anonymised `actor_id` hash for 7 years (DEC-030 HMAC chain continuity).

**Path B — User-initiated GDPR erasure request (Art. 17).**

A SCIM-provisioned user who has logged in and created data may request erasure directly from FORM (e.g., via the in-app settings or a written request to privacy@form.coach). This creates a tension with the employer's active provisioning of the account.

FORM's procedure:
1. Verify that the request comes from the verified account owner (SSO-authenticated request or email-verified OTP).
2. Assess whether Art. 17(3) exceptions apply: no legal obligation, no public task, no archiving/research ground applies for FORM's use case. The erasure proceeds.
3. Execute the health data erasure (same as Path A steps 3–5 above).
4. **Notify the tenant admin** via a `system.gdpr_erasure_request` audit event and, if the tenant has a webhook configured (§15), a webhook notification: "A user with an active SCIM-provisioned account has requested GDPR erasure. FORM has completed erasure of user-generated health data. The user's directory account remains active in your IdP. You should review whether continued SCIM provisioning of this user's record is appropriate given the erasure request." FORM does not delete the IdP-provisioned record unilaterally — the Controller retains ownership of their directory.
5. If the user is still `active` in the IdP and FORM continues to receive SCIM updates for that user_id after erasure, FORM accepts the SCIM updates to the identity fields (email, name) but does not re-populate health data. The account remains a shell for access control purposes only.
6. A `data.individual_deletion` audit event is written per `AUDIT_LOG_SCHEMA.md`, with `actor_type = 'user'` and `changes = { "triggered_by": "gdpr_art17_request" }`.

**Audit log preservation.** The HMAC chain (DEC-030) must remain intact through an erasure event. The user's past audit entries (login events, data access events, provisioning events) are not deleted; the `actor_id` in those entries is replaced with the anonymised hash. The hash is stored in a `gdpr_erasure_log` table (separate from `audit_log`) that maps the original `user_id` to the anonymised hash, accessible only to compliance-officer and security-engineer for chain verification purposes. This table is itself audited.

### 14.7 SCIM Audit Events (HMAC Chain per DEC-030)

All SCIM operations write to the `audit_log` table using the HMAC chain defined in `docs/AUDIT_LOG_SCHEMA.md`. The following events are added to the action taxonomy:

| Event | Trigger | Notable fields in `changes` |
|---|---|---|
| `scim.user.provisioned` | SCIM POST /Users succeeds | `user_id`, `email_hash`, `scim_attributes_received` (attribute names only, not values), `source_idp` |
| `scim.user.updated` | SCIM PUT or PATCH /Users/:id succeeds | `user_id`, `changed_attributes` (names only), `active_before`, `active_after` |
| `scim.user.deprovisioned` | SCIM DELETE /Users/:id or PATCH active=false | `user_id`, `email_hash`, `final_session_count_revoked` |
| `scim.group.created` | SCIM POST /Groups succeeds | `group_id`, `group_display_name`, `initial_member_count` |
| `scim.group.updated` | SCIM PATCH /Groups/:id | `group_id`, `members_added_count`, `members_removed_count`, `role_mapping_triggered` |
| `scim.group.deleted` | SCIM DELETE /Groups/:id | `group_id`, `affected_user_count`, `role_assignments_revoked_count` |
| `scim.rejected_sensitive_attribute` | Attribute scanner detects blocked pattern | `tenant_id`, `matched_pattern`, `offending_attribute_name` (not value) |
| `scim.token.rotated` | Customer rotates SCIM bearer token via admin dashboard | `tenant_id`, `initiated_by`, `old_token_last_4` |
| `scim.token.revoked` | Emergency SCIM token revocation | `tenant_id`, `initiated_by`, `reason` |
| `system.gdpr_erasure_request` | Art. 17 erasure request received for a SCIM-provisioned user | `user_id`, `request_method`, `tenant_notified`, `health_data_deleted_at` |

**Retention.** SCIM provisioning events (`scim.user.*`, `scim.group.*`) are retained for 7 years per `AUDIT_LOG_SCHEMA.md §Retention schedule` (tenant administrative events class). `scim.rejected_sensitive_attribute` events are retained for 7 years (security events class). `system.gdpr_erasure_request` retained for 7 years (data export/deletion class).

**HMAC chain continuity for SCIM events.** SCIM events are written to the same HMAC chain as all other audit events within the tenant. There is no separate SCIM audit chain. This ensures that a sequence such as `scim.user.provisioned` → `auth.login` → `data.read_aggregate` → `scim.user.deprovisioned` forms a single verifiable chain, which customers can verify using the chain verification procedure in `AUDIT_LOG_SCHEMA.md §HMAC chaining`.

### 14.8 Sub-Processor Disclosure

SCIM provisioning introduces no sub-processors beyond those already listed in FORM's main DPA. The complete data flow for a SCIM operation is:

```
Customer IdP (controller infrastructure)
  → HTTPS POST to api.form.coach/scim/v2/
  → Cloudflare Workers (transit only — zero persistent storage, <1ms retention)
  → Supabase PostgreSQL (primary data store — existing DPA sub-processor)
  → Cloudflare R2 (audit log archive — existing DPA sub-processor, if audit export configured)
```

No third-party enrichment, analytics, or AI inference is performed on SCIM-provisioned directory data. SCIM attributes are never sent to Anthropic, ElevenLabs, PostHog, or Sentry.

**PostHog clarification.** FORM's PostHog integration tracks product usage events (feature interactions, session metrics). SCIM `email` and `displayName` values are **not** sent as PostHog identify traits for SCIM-provisioned users. PostHog receives only the FORM internal `user_id` (a UUID) and anonymised usage events. This constraint is enforced in the PostHog initialisation code: `distinctId` is always the FORM `user_id` UUID, never an email address.

### 14.9 EU Data Residency for SCIM Attributes

SCIM-provisioned user records — including all attributes in the personal data inventory (§14.2) — are stored in the same Supabase region as all other data for that tenant. There is no SCIM-specific data store.

For EU enterprise customers (Supabase EU region, Frankfurt `eu-central-1`):
- All SCIM attributes are stored exclusively in `eu-central-1`.
- The Cloudflare Workers API layer processes SCIM requests at the edge closest to the inbound request, but holds no persistent state. Cloudflare's edge nodes for European requests are governed by Cloudflare's EU DPA and Standard Contractual Clauses.
- Audit log entries for SCIM operations are partitioned within the same Supabase EU region.
- No SCIM data for EU-region tenants transits through or is stored in US-region infrastructure.

For US enterprise customers (Supabase US East, `us-east-1`):
- Data residency is US by default.
- EU-resident employees who are SCIM-provisioned under a US-region tenant: this is a controller-level decision. FORM's DPA discloses that US-region tenants store data in `us-east-1`. The enterprise customer (controller) is responsible for ensuring they have an appropriate legal basis for transferring EU employee data to a US-based processor — Standard Contractual Clauses (Module 2, controller-to-processor) are included in FORM's DPA as the transfer mechanism.

### 14.10 SCIM DPA Compliance Implementation Checklist

| Task | Owner | Priority | Blocking condition |
|---|---|---|---|
| Forward §14.3 Annex B clause language to outside counsel for review | compliance-officer | **P0 — blocks enterprise SCIM enable** | G-013 hard block |
| Incorporate approved clause language into FORM standard DPA template | compliance-officer + outside counsel | P0 | Requires counsel approval |
| Update FORM DPA template version number and effective date | compliance-officer | P0 | After counsel approval |
| Confirm Art. 14 transparency notice approach with counsel (is privacy policy sufficient, or is a separate employee notice required?) | compliance-officer | P0 | Jurisdiction-dependent; resolve before first EU enterprise SCIM customer |
| Implement `scim.rejected_sensitive_attribute` audit event in SCIM endpoint code | platform-engineer | P0 | Part of G-001 SCIM implementation |
| Implement `BLOCKED_ATTRIBUTE_PATTERNS` scanner (§14.5) in SCIM endpoint | platform-engineer | P0 | Part of G-001 SCIM implementation |
| Confirm `users.scim_attributes` column is excluded from all tenant-manager RLS policies | security-engineer | P1 | Verify at schema migration time when G-001 is built |
| Add `system.gdpr_erasure_request` event to audit log action taxonomy in `AUDIT_LOG_SCHEMA.md` | compliance-officer | P1 | Before first GDPR erasure request is processed |
| Add Art. 17 procedure for SCIM users (§14.6 Path B) to support runbook | customer-success | P1 | Before first enterprise customer goes live |
| Verify PostHog SCIM data isolation (no email in distinctId for SCIM users) | platform-engineer | P1 | Part of G-001 implementation review |
| Document SCIM in FORM's GDPR Art. 30 records of processing activities | compliance-officer | P1 | Art. 30 requires documentation of all processing activities |
| Confirm SCC Module 2 coverage for US-region tenants with EU employees | compliance-officer + outside counsel | P2 — affects EU-employee data under US tenants | Not blocking for US-only enterprise customers |

### 14.11 G-013 Status Update

| Field | Value |
|---|---|
| **Gap ID** | G-013 |
| **Previous status** | 🔴 Not designed — block on SCIM enablement |
| **Updated status** | 🟡 Design complete, outside counsel review and DPA template update pending |
| **Design deliverables** | §14.3 Annex B clause language; §14.2 personal data inventory; §14.5 attribute rejection spec; §14.6 Art. 17 procedure; §14.7 audit event taxonomy; §14.8 sub-processor disclosure; §14.9 residency commitments |
| **Blocking condition** | G-013 hard block lifts when: (1) outside counsel approves revised DPA template, and (2) at least one signed DPA with revised Annex B exists for the first SCIM customer |
| **Next action** | compliance-officer forwards §14.3 to outside counsel. ETA: 1–2 weeks for review. |
| **SOC 2 impact** | SCIM provisioning, once implemented (G-001), contributes to CC6.1 (logical access) and CC6.2 (user access management). G-013 closure is required for the auditor to accept SCIM as a SOC 2 in-scope control rather than a compliance exception. |
| **GDPR Art. 30 impact** | SCIM must be added to FORM's records of processing activities (Art. 30(2) processor records) — see §14.10 implementation checklist. |

---

*v0.6 additions: Section 14 — SCIM Data Processing: GDPR Article 28 Compliance Framework. Closes G-013 design phase (previously 🔴 "block: do not enable SCIM for any customer until DPA is updated" → 🟡 design complete, outside counsel review pending). Personal data inventory for all SCIM attribute classes with retention periods and legal basis chain. DPA Annex B template clause language (subject-matter, duration, nature/purpose, data types, data subject categories, controller obligations, sub-processor disclosure, Art. 15–20 assistance matrix) — ready for outside counsel review. Privacy floor enforcement: `users.scim_attributes` excluded from tenant-manager RLS; role mapping opacity; k-anonymity floor applies to SCIM-dimension aggregates. Data minimisation: unknown attribute silent-drop, enterprise extension allowlist, Art. 9 special category attribute scanner with `BLOCKED_ATTRIBUTE_PATTERNS` blocklist and `scim.rejected_sensitive_attribute` audit event. Art. 17 dual-path erasure: employer-initiated deprovisioning (Path A) vs. user-initiated GDPR erasure (Path B) with tenant notification and HMAC chain continuity via anonymised `actor_id` hash stored in separate `gdpr_erasure_log` table. 10 new audit events added to taxonomy (DEC-030 HMAC chain). Sub-processor disclosure: confirmed no new processors; PostHog `distinctId` isolation constraint documented. EU residency: Supabase `eu-central-1` for EU tenants; SCC Module 2 documented for US-region tenants with EU employees. Implementation checklist: 12 items (P0/P1/P2). G-013: 🔴 → 🟡 Partial.*

---

## 15. SCIM 2.0 Groups Provisioning

> Owner: enterprise-architect. Review: on any IdP group schema change or quarterly.
> Dependencies: §3 (SCIM 2.0 Provisioning), §5 (Role Mapping), §11 (JIT Provisioning), §12 (Session Lifecycle), §14 (GDPR Art. 28 Framework).

---

### 15.1 Overview

SCIM Groups provisioning allows an enterprise customer's identity provider to push group memberships to FORM, which then drives role assignment without manual per-user configuration. Three provisioning paths are available; the right choice depends on org size, IdP capability, and role-mapping complexity.

| Dimension | SCIM Groups | Direct role assignment | JIT role claims |
|---|---|---|---|
| **How roles are assigned** | IdP group membership → FORM role mapping table | Admin manually sets each user's role in FORM admin dashboard | Role claim in OIDC/SAML token evaluated at login |
| **Auto-deprovisioning** | Yes — group removal triggers role revocation and session termination | No — manual only | No — revocation only on next login |
| **Best for** | Orgs with >20 seats, structured IdP groups, HR-driven joiners/movers/leavers | Small pilots (<20 seats), orgs with no IdP group discipline | Google Workspace (see §15.6.3), orgs with mature claim-enrichment pipeline |
| **IdP support** | Okta, Entra ID (full); Google Workspace (not supported — see §15.6.3 and G-014) | Any | Any (requires token claim config) |
| **Bulk role change** | Yes — rename/reassign IdP group → all members updated | No | Yes — update claim value in IdP |
| **Audit granularity** | Per-member add/remove events | Per-user manual event | Per-login event only |
| **Recommended threshold** | **>20 seats** | ≤20 seats | Any size where SCIM Groups unavailable |

**Recommendation:** Enable SCIM Groups for any new enterprise customer with more than 20 licensed seats and a functioning Okta or Entra ID group structure. For Google Workspace customers, fall back to JIT role claims per §11 and document the gap per §15.11 (G-014).

---

### 15.2 SCIM Group Resource Schema

#### 15.2.1 RFC 7643 §8 Compliant JSON Example

The following is the canonical SCIM Group resource representation FORM accepts and returns. FORM is the SP (SCIM server); the IdP is the client.

```json
{
  "schemas": ["urn:ietf:params:scim:schemas:core:2.0:Group"],
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "externalId": "00g1emaKYZTWRISHRRZT",
  "displayName": "FORM Coaches — EMEA",
  "members": [
    {
      "value": "2819c223-7f76-453a-919d-413861904646",
      "display": "Jane Smith",
      "$ref": "https://api.form.coach/scim/v2/Users/2819c223-7f76-453a-919d-413861904646",
      "type": "User"
    },
    {
      "value": "c75ad752-64ae-4823-840d-ffa80929976c",
      "display": "Marcus Reid",
      "$ref": "https://api.form.coach/scim/v2/Users/c75ad752-64ae-4823-840d-ffa80929976c",
      "type": "User"
    }
  ],
  "meta": {
    "resourceType": "Group",
    "created": "2026-03-15T09:00:00Z",
    "lastModified": "2026-05-10T14:32:00Z",
    "location": "https://api.form.coach/scim/v2/Groups/550e8400-e29b-41d4-a716-446655440000",
    "version": "W/\"a330bc54f0671c9\""
  }
}
```

Notes:
- `id` is FORM's internal UUID, stable across all operations.
- `externalId` is the IdP-assigned group identifier (Okta group ID or Azure Object ID). FORM stores this and uses it for idempotent reconciliation.
- `members.$ref` is the canonical SCIM URL for the referenced User resource. For groups with >500 members, FORM returns `$ref` values only (omitting `display`) to reduce payload size (see §15.5).
- Nested Group `type: "Group"` members are not accepted; FORM returns `HTTP 400 scimType: "invalidValue"` with a descriptive detail message. Entra ID nested groups are flattened before storage — see §15.6.2a.

#### 15.2.2 PostgreSQL Table Schema

```sql
-- SCIM Groups: one row per group pushed by IdP
CREATE TABLE scim_groups (
  id               UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id        UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
  external_id      TEXT NOT NULL,           -- IdP-assigned group ID (Okta group ID, Azure Object ID)
  display_name     TEXT NOT NULL,
  created_at       TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at       TIMESTAMPTZ NOT NULL DEFAULT now(),

  UNIQUE (tenant_id, external_id)
);

-- RLS: strict tenant isolation — no cross-tenant group reads
ALTER TABLE scim_groups ENABLE ROW LEVEL SECURITY;

CREATE POLICY scim_groups_tenant_isolation ON scim_groups
  USING (tenant_id = current_setting('app.tenant_id')::UUID);

-- SCIM Group Members: junction table between groups and provisioned users
CREATE TABLE scim_group_members (
  id               UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id        UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
  group_id         UUID NOT NULL REFERENCES scim_groups(id) ON DELETE CASCADE,
  user_id          UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  added_at         TIMESTAMPTZ NOT NULL DEFAULT now(),
  added_by_op      TEXT NOT NULL DEFAULT 'scim_patch', -- 'scim_patch' | 'scim_put' | 'scim_post'

  UNIQUE (tenant_id, group_id, user_id)
);

-- RLS: same tenant_id constraint enforced at row level
ALTER TABLE scim_group_members ENABLE ROW LEVEL SECURITY;

CREATE POLICY scim_group_members_tenant_isolation ON scim_group_members
  USING (tenant_id = current_setting('app.tenant_id')::UUID);

-- Index: fast member lookup for role-resolution on login
CREATE INDEX idx_scim_group_members_user
  ON scim_group_members (tenant_id, user_id);

-- Index: fast group membership list for SCIM GET /Groups/{id}
CREATE INDEX idx_scim_group_members_group
  ON scim_group_members (tenant_id, group_id);

-- Trigger: keep scim_groups.updated_at current on any member change
CREATE OR REPLACE FUNCTION update_scim_group_updated_at()
RETURNS TRIGGER LANGUAGE plpgsql AS $$
BEGIN
  UPDATE scim_groups
    SET updated_at = now()
    WHERE id = COALESCE(NEW.group_id, OLD.group_id)
      AND tenant_id = current_setting('app.tenant_id')::UUID;
  RETURN NULL;
END;
$$;

CREATE TRIGGER trg_scim_group_members_updated
  AFTER INSERT OR DELETE ON scim_group_members
  FOR EACH ROW EXECUTE FUNCTION update_scim_group_updated_at();
```

**Migration note.** The `scim_group_role_mappings` table introduced in §4.2 remains the authoritative mapping table. `scim_groups` and `scim_group_members` are the new SCIM resource tables that back the `/Groups` endpoint. The `scim_group_role_mappings.idp_group_id` foreign-key-by-value relationship links to `scim_groups.external_id` within the same `tenant_id`.

---

### 15.3 Supported SCIM Group Operations

FORM implements the following SCIM 2.0 Group endpoint operations at `https://api.form.coach/scim/v2/Groups`. All requests must include the tenant-scoped SCIM bearer token (§3.2).

| Operation | HTTP method + path | Description | Notes |
|---|---|---|---|
| List groups | `GET /Groups` | Returns paginated list of all groups for tenant | Supports `filter=displayName eq "..."`, `count`, `startIndex` |
| Get single group | `GET /Groups/{id}` | Returns full group resource including members array | `{id}` is FORM UUID, not `externalId` |
| Create group | `POST /Groups` | Creates a new group; `externalId` required | Idempotent on `(tenant_id, externalId)` — returns existing group if already present |
| Replace group | `PUT /Groups/{id}` | Full replace of group resource including member list | Existing members not in PUT body are removed; triggers per-member audit events |
| Update group (members) | `PATCH /Groups/{id}` | Add or remove individual members; update `displayName` | Preferred over PUT for membership changes; supports `op: add`, `op: remove`, `op: replace` |
| Delete group | `DELETE /Groups/{id}` | Removes group and all memberships; triggers role revocation | Member session revocation is atomic with group deletion (see §15.7) |

**Deferred to M5:** Bulk group operations (`POST /Bulk` with Group resources). See §15.10 checklist item.

**Pagination requirement:** Any group with more than 1000 members must be retrieved or written using cursor-based pagination (see §15.5). A `POST /Groups` or `PUT /Groups/{id}` with a `members` array exceeding 1000 entries returns `HTTP 400` with `detail: "members array exceeds 1000; use PATCH with paginated add operations"`.

---

### 15.4 Group-to-Role Mapping

#### 15.4.1 Tenant SSO Config Extension

The `tenant_sso_configs` table (§4.2) is extended with a `group_role_mappings` JSONB column. The JSON structure for that column:

```json
{
  "group_role_mappings": [
    {
      "idp_group_id": "00g1emaKYZTWRISHRRZT",
      "idp_group_display_name": "FORM Admins — Global",
      "form_role": "tenant_admin"
    },
    {
      "idp_group_id": "00g2xzKLMNOPQRSTUVWX",
      "idp_group_display_name": "FORM Coaches — EMEA",
      "form_role": "coach"
    },
    {
      "idp_group_id": "00g3abcDEFGHIJKLMNOP",
      "idp_group_display_name": "FORM Members — All Staff",
      "form_role": "member"
    }
  ],
  "default_role_for_unmatched_groups": "member",
  "multiple_group_resolution": "highest_privilege"
}
```

Fields:

| Field | Type | Description |
|---|---|---|
| `group_role_mappings` | array | Each entry maps one IdP group to one FORM role. `idp_group_id` is the stable IdP identifier (not the display name). |
| `default_role_for_unmatched_groups` | string | FORM role assigned to a user whose groups are all unmapped. Must be one of the valid FORM roles. Cannot be `tenant_owner`. |
| `multiple_group_resolution` | enum | `"highest_privilege"` (default) or `"lowest_privilege"`. Governs how FORM resolves a user who belongs to multiple mapped groups with different roles. |

#### 15.4.2 `highest_privilege` Algorithm

When a user belongs to multiple groups and `multiple_group_resolution = "highest_privilege"`, FORM evaluates all mapped groups the user is a member of and assigns the highest role in the following priority order:

```
tenant_admin  >  coach  >  member  >  viewer
```

`tenant_owner` cannot be assigned via group mapping — it must be set explicitly by a FORM `enterprise-architect`. This is a hard constraint: the mapping config schema rejects `"form_role": "tenant_owner"` with an error at save time.

Example: a user is in groups mapped to `coach` and `member`. The resolved role is `coach`.

When `multiple_group_resolution = "lowest_privilege"`, the same priority order is used in reverse (safest role wins). This is appropriate for orgs using broad catch-all groups.

Role resolution runs on every authentication event (JIT login) and on every SCIM PATCH that modifies group membership, not just on first provisioning.

#### 15.4.3 Unmapped Group Handling

If a user's group membership set contains no entries in `group_role_mappings`:

1. The user is assigned `default_role_for_unmatched_groups` (configured per tenant; defaults to `member`).
2. A `scim.group_unmapped` audit event is written with fields: `tenant_id`, `user_id`, `idp_group_ids` (list of group IDs that triggered the condition), `assigned_default_role`.
3. The user is allowed to authenticate and access FORM at the default role level. They are not blocked.

If `default_role_for_unmatched_groups` is deliberately set to `null` by the tenant admin (opt-in strict mode), unmapped users are blocked from accessing FORM and receive a tenant-branded error page: "Your account is not assigned to an authorised group. Contact your IT administrator." The `scim.group_unmapped` event is still written.

---

### 15.5 Member Synchronization & Pagination

#### 15.5.1 PATCH Operation: Add Members

The following PATCH body adds two members to an existing group. This is the standard operation Okta and Entra ID issue when a user is added to a pushed group.

```json
{
  "schemas": ["urn:ietf:params:scim:api:messages:2.0:PatchOp"],
  "Operations": [
    {
      "op": "add",
      "path": "members",
      "value": [
        {
          "value": "2819c223-7f76-453a-919d-413861904646",
          "display": "Jane Smith",
          "$ref": "https://api.form.coach/scim/v2/Users/2819c223-7f76-453a-919d-413861904646"
        },
        {
          "value": "c75ad752-64ae-4823-840d-ffa80929976c",
          "display": "Marcus Reid",
          "$ref": "https://api.form.coach/scim/v2/Users/c75ad752-64ae-4823-840d-ffa80929976c"
        }
      ]
    }
  ]
}
```

FORM response on success: `HTTP 200` with the updated Group resource. If any `value` (user SCIM ID) does not exist within the tenant, FORM returns `HTTP 400 scimType: "noTarget"` for that specific member and does not apply the partial add — the entire operation is rejected atomically.

#### 15.5.2 PATCH Operation: Remove Members

The following PATCH body removes one member from a group. This is the standard operation issued when a user is removed from an IdP group.

```json
{
  "schemas": ["urn:ietf:params:scim:api:messages:2.0:PatchOp"],
  "Operations": [
    {
      "op": "remove",
      "path": "members[value eq \"2819c223-7f76-453a-919d-413861904646\"]"
    }
  ]
}
```

FORM response on success: `HTTP 200` with the updated Group resource. The member removal, role revocation, and session termination for the affected user are executed atomically in a single database transaction (see §15.7 and §12.7).

#### 15.5.3 Large Group Pagination (>100 Members)

For `GET /Groups/{id}` requests on groups with more than 100 members, FORM uses cursor-based pagination via standard SCIM list pagination parameters.

Paginated list request:
```
GET /scim/v2/Groups/{id}/Members?count=100&startIndex=1
```

FORM response envelope:

```json
{
  "schemas": ["urn:ietf:params:scim:api:messages:2.0:ListResponse"],
  "totalResults": 847,
  "itemsPerPage": 100,
  "startIndex": 1,
  "Resources": [
    { "value": "...", "display": "...", "$ref": "..." }
  ]
}
```

To retrieve the next page: `startIndex=101`, then `startIndex=201`, and so on until `startIndex > totalResults`.

For `PATCH add` operations adding more than 100 members in a single batch, FORM accepts the full array up to the 1000-member hard cap (§15.3). Above 1000, the client must issue multiple PATCH operations.

#### 15.5.4 `members.$ref` Optimisation for Large Groups

For groups with more than 500 members, the `GET /Groups/{id}` response omits the `display` field from member entries and returns only `value` and `$ref`. This reduces the payload size by approximately 40% for typical display name lengths and keeps responses within Cloudflare Worker response body limits.

Clients that require `display` for a specific member should call `GET /Users/{id}` directly.

#### 15.5.5 Rate Limit Interaction

Group membership synchronisation from a large IdP push (e.g., initial Okta group sync, Entra ID provisioning cycle) may generate a high volume of `PATCH /Groups/{id}` requests in a short window. The per-tenant rate limits from §3.6 apply:

| Endpoint | Rate Limit | Burst |
|---|---|---|
| `POST /scim/v2/Groups` | 50 req/min | 100 |
| `PATCH /scim/v2/Groups/{id}` | 100 req/min | 200 |
| `DELETE /scim/v2/Groups/{id}` | 50 req/min | 100 |
| `GET /scim/v2/Groups` (list) | 30 req/min | 60 |
| `GET /scim/v2/Groups/{id}` | 300 req/min | 600 |

When the limit is exceeded, FORM returns `HTTP 429` with a `Retry-After` header specifying the number of seconds until the window resets. Okta, Entra ID, and OneLogin all implement native exponential backoff when they receive `429` responses. No additional configuration is required on the customer side.

**Initial sync guidance:** Customers performing an initial group push of >50 groups should contact their FORM customer success manager to request a temporary burst increase for the sync window. This is a manual process executed by the enterprise-architect; it is not self-serve.

---

### 15.6 IdP-Specific Group Behavior

#### 15.6.1 Okta

| Attribute | Detail |
|---|---|
| `externalId` source | Okta Group ID (e.g. `00g1emaKYZTWRISHRRZT`) — stable, use as the canonical group identifier |
| Group push mechanism | Okta admin must configure "Push Groups" in the FORM app integration under the Provisioning → Push Groups tab |
| Membership change operations | Okta issues `PATCH /Groups/{id}` with `op: add` or `op: remove` on individual member changes — never a full PUT replace |
| Group deletion | Okta issues `DELETE /Groups/{id}` when "Push Groups" is removed or the group is unlocked from push |
| Profile attributes | Okta may include group profile attributes in the Group resource body. FORM silently ignores all attributes not in the SCIM core Group schema (RFC 7643 §8). No `HTTP 400` is returned for unknown attributes — they are dropped and a `scim.group_updated` event notes `extra_attributes_dropped: true` |
| Setup requirement | Customer IT admin must explicitly navigate to Provisioning → Push Groups in Okta and add each group to be pushed. Groups are not automatically pushed by default — this is a common onboarding gap |

**Okta customer setup steps:**
1. In Okta Admin Console, navigate to Applications → FORM app → Provisioning → Push Groups.
2. Click "Push Groups" → "Find groups by name" → select each group to sync.
3. Confirm "Create Group" is selected (not "Link Group") for new groups.
4. Save. Okta will issue a `POST /Groups` for each selected group followed by `PATCH /Groups/{id}` for initial member population.

#### 15.6.2 Microsoft Entra ID (Azure AD)

| Attribute | Detail |
|---|---|
| `externalId` source | Azure AD Object ID (GUID, e.g. `7b2c4e1a-3f8d-4b9e-a123-456789abcdef`) — stable, tenant-scoped |
| Group push mechanism | Entra ID Provisioning app uses automatic attribute mapping; groups are pushed via the SCIM `/Groups` endpoint when group assignment is configured in the Enterprise Application |
| Membership change operations | Entra ID may issue either `PATCH /Groups/{id}` (incremental) or `PUT /Groups/{id}` (full replace) depending on the provisioning cycle and group size |
| Dynamic groups | Entra ID dynamic membership groups are treated as static by FORM — FORM receives the resolved member list and does not interpret the dynamic membership rule. Changes to dynamic membership propagate to FORM on the next Entra provisioning cycle (typically 20–40 minutes) |

##### 15.6.2a Nested Group Flattening

Entra ID supports nested group membership (a group containing other groups). FORM does not accept nested Group members in the SCIM `members` array — entries with `type: "Group"` are rejected (see §15.2.1).

**How FORM handles this:** When an Entra ID provisioning push includes a group with nested group members, FORM's SCIM endpoint:

1. Accepts all `type: "User"` members normally.
2. For each `type: "Group"` member entry, logs a `scim.group_nested_flattened` audit event with `tenant_id`, `parent_group_id`, `nested_group_ref`, and `action: "skipped"`.
3. Returns `HTTP 200` with the created/updated Group resource — the nested group entries are omitted from the stored membership, not reflected back in the response.
4. Does **not** resolve the nested group's membership transitively. Transitive member resolution is not performed server-side.

**Customer recommendation — two options:**

Option A (preferred): Restructure IdP groups to use flat security groups. Move all individual user accounts into a single-level group with no nesting. This is the simplest path and aligns with Microsoft's own provisioning best-practice guidance for third-party SCIM endpoints.

Option B: Enable the Entra ID P1/P2 "Member flattening" setting if available in the customer's tenant configuration. With this setting, Entra ID resolves nested group members before pushing to third-party SCIM endpoints, sending only direct `User` member entries. This requires an Entra P1 or P2 licence.

If neither option is implemented, FORM will only provision direct members of the pushed group. Members who are members via nesting will not be provisioned and will not receive roles. The `scim.group_nested_flattened` audit events in the admin dashboard give visibility into how many members were skipped.

#### 15.6.3 Google Workspace

Google Workspace does not support native SCIM Group provisioning to third-party SCIM endpoints. The Google Workspace Directory API can push user provisioning (SCIM Users), but does not expose a third-party SCIM Groups interface.

**Consequence:** Customers using Google Workspace as their sole IdP cannot use SCIM Groups for role assignment in FORM.

**Recommended path:** Use JIT provisioning with role claims per §11. The customer configures a custom attribute or group claim in their Google Workspace SAML/OIDC app configuration, and FORM maps the claim value to a FORM role at login time. Role changes take effect on the user's next login.

This is a by-design limitation of Google Workspace's SCIM implementation, not a FORM defect. The gap is formally documented as G-014 (§15.11).

---

### 15.7 Audit Events for Group Operations

All group operations write to the tenant's HMAC-chained `audit_log` per DEC-030. There is no separate group audit chain — group events are interleaved with all other tenant audit events, maintaining chain continuity.

The following 9 events are added to the audit event taxonomy:

| Event | Trigger | Key fields in `changes` |
|---|---|---|
| `scim.group_created` | `POST /Groups` succeeds | `group_id`, `external_id`, `display_name`, `initial_member_count`, `source_idp` |
| `scim.group_updated` | `PUT /Groups/{id}` or `PATCH /Groups/{id}` (displayName only) | `group_id`, `old_display_name`, `new_display_name`, `extra_attributes_dropped` |
| `scim.group_member_added` | `PATCH /Groups/{id} op:add` or `PUT` net-add | `group_id`, `user_id`, `user_email_hash`, `role_mapping_triggered`, `resolved_role` |
| `scim.group_member_removed` | `PATCH /Groups/{id} op:remove` or `PUT` net-remove | `group_id`, `user_id`, `user_email_hash`, `sessions_revoked_count` |
| `scim.group_deleted` | `DELETE /Groups/{id}` | `group_id`, `external_id`, `display_name`, `affected_user_count`, `sessions_revoked_count` |
| `scim.group_role_mapping_applied` | Role resolved from group membership during login or PATCH | `user_id`, `group_ids_evaluated`, `resolved_role`, `resolution_strategy` |
| `scim.group_unmapped` | User's groups have no mapping entries | `user_id`, `idp_group_ids`, `assigned_default_role`, `strict_mode_blocked` |
| `scim.group_nested_flattened` | Entra ID pushes a group with `type: "Group"` members | `parent_group_id`, `nested_group_ref`, `action: "skipped"`, `members_skipped_count` |
| `scim.group_pagination_started` | `GET /Groups/{id}/Members` paginated request begins | `group_id`, `total_members`, `page_size`, `start_index`, `requested_by` |

**Atomicity requirement (member removal + session revocation).** The `scim.group_member_removed` event and the corresponding session revocation (§12.7) must be written in the same PostgreSQL transaction. If the session revocation step fails, the entire transaction is rolled back and the group membership is not removed. This prevents a state where a user has lost group membership (and thus their role entitlement) but retains an active session with the old role. The SCIM endpoint returns `HTTP 500` in this case and the IdP will retry per its backoff schedule.

This constraint mirrors the session revocation atomicity requirement established in §12.7 for SCIM user deprovisioning and must be implemented with the same transaction pattern.

---

### 15.8 Customer Onboarding Checklist — Groups

This checklist supplements §7 (Customer Onboarding Checklist). It covers the additional steps required to enable and validate SCIM Groups provisioning for a new enterprise customer.

#### FORM side

| Step | Owner | Notes |
|---|---|---|
| Enable `feature_flag: scim_groups` for tenant | enterprise-architect | Set in admin dashboard feature flags UI (§15.10 item 11). Not enabled by default — requires explicit activation |
| Configure `group_role_mappings` in tenant SSO config | enterprise-architect (with customer) | Populate `idp_group_id`, `idp_group_display_name`, `form_role` for each group. Review with customer IT admin before activation |
| Set `default_role_for_unmatched_groups` | enterprise-architect | Confirm with customer — default is `member`. Consider `null` (strict block) for high-security deployments |
| Confirm `multiple_group_resolution` strategy | enterprise-architect | `highest_privilege` is default. Discuss with customer if they have overlapping group structures |
| Verify rate limits are sufficient for initial sync | enterprise-architect | Assess group count and member count. Request temporary burst increase if >50 groups or >5000 total member assignments |
| Confirm Art. 9 group name scanner is active | security-engineer | Verify `scim.group_name_sensitive_detected` events are enabled for the tenant (§15.9.5) |

#### Customer side — Okta

| Step | Customer role | Notes |
|---|---|---|
| Navigate to FORM app → Provisioning → Push Groups | IT admin | Required — groups are not pushed automatically |
| Add each group to be pushed | IT admin | Use "Find groups by name". Confirm "Create Group" mode is selected |
| Verify initial sync completes without errors | IT admin | Check Okta provisioning logs for `400` or `429` errors. Share FORM audit log export if troubleshooting |
| Confirm no sensitive group names are in scope | IT admin + compliance | Review §15.9.1 — group names matching Art. 9 patterns trigger a compliance review |

#### Customer side — Microsoft Entra ID

| Step | Customer role | Notes |
|---|---|---|
| Assign groups to the FORM Enterprise Application | IT admin | Azure Portal → Enterprise Applications → FORM → Users and Groups → Add assignment → select groups |
| Verify no nested groups are in scope | IT admin | Flatten nested groups or enable Entra P1/P2 member flattening before first sync (§15.6.2a) |
| Check provisioning logs for nested group skips | IT admin | `scim.group_nested_flattened` events in FORM audit log indicate members missed due to nesting |
| Confirm dynamic group membership is current | IT admin | Dynamic groups reflect membership at sync time. Confirm Entra evaluation cycle is recent before first FORM sync |

#### Customer side — Google Workspace

| Step | Customer role | Notes |
|---|---|---|
| Acknowledge SCIM Groups not supported | IT admin + FORM CSM | G-014 disclosure (§15.11) — customer must acknowledge in writing |
| Configure JIT role claims per §11 | IT admin | Set up custom attribute or group claim in Google Workspace SAML/OIDC app |
| Verify role claim values match FORM role names | IT admin + enterprise-architect | Test with a pilot user before full rollout |

---

### 15.9 GDPR Implications of Group Membership Data

#### 15.9.1 Group `displayName` as Sensitive Data (Art. 9)

Group `displayName` values can reveal Art. 9 special category data about the individuals in that group. Examples encountered in enterprise deployments:

- `"PTSD Support Programme Members"` — reveals mental health condition
- `"Oncology Patient Fitness Track"` — reveals health data
- `"Ramadan Wellness Programme"` — reveals religious belief
- `"Disability Access Accommodations"` — reveals disability status
- `"Union Representatives"` — reveals trade union membership

FORM does not control what names enterprise customers assign to their IdP groups. The `displayName` is a required field in the SCIM Group schema and must be accepted to function. However, storing and processing a group name that reveals Art. 9 data about its members creates a processing obligation FORM must handle under its DPA with the customer.

#### 15.9.2 Group Membership as Organisational Hierarchy Data

Beyond Art. 9 cases, group membership data reveals an employee's position in the organisational hierarchy, team, department, and access tier. This is personal data under Art. 4(1) GDPR.

- Group membership data (which user is in which group) is excluded from all FORM analytics pipelines. It is not sent to PostHog, not used for aggregate reporting, and not included in any feature-usage event.
- The privacy floor from §14 applies: `scim_group_members` data is excluded from `tenant_manager` RLS policies. `tenant_manager` role can see aggregate adoption metrics but cannot query which users are in which groups.
- Group `displayName` values are stored in `scim_groups.display_name` but are not surfaced in any UI visible to roles below `tenant_admin`.

#### 15.9.3 Art. 17 Erasure Procedure for Group Membership

When a GDPR Art. 17 erasure request is received for a SCIM-provisioned user (§14.6 Path B), group membership is handled as follows:

1. All rows in `scim_group_members` where `user_id = {erased_user_id}` are hard-deleted immediately (no soft-delete period — group membership is access control data, not user-generated content).
2. The `scim_groups` records themselves are not deleted — they exist independently of individual users and belong to the tenant, not the individual.
3. Audit log entries referencing the user's group memberships (`scim.group_member_added`, `scim.group_member_removed`) are retained with the anonymised `actor_id` hash per §14.6 and DEC-030 HMAC chain continuity requirements.
4. The erasure completion event (`data.individual_deletion`) includes `group_memberships_deleted: true` and `group_count` in its `changes` field.

#### 15.9.4 Art. 30 Records of Processing

SCIM Group membership data must be added to FORM's Art. 30 records of processing activities. The processing activity is:

- **Purpose:** Access control and role assignment within FORM enterprise tier
- **Legal basis (controller):** Legitimate interests of the employer (Art. 6(1)(f)) — maintaining role-based access to a workplace tool
- **Data types:** Group membership relationships (user–group pairs); group display names
- **Retention:** Active while SCIM provisioning active; hard-deleted on user erasure (Art. 17); group records retained until tenant deletes the group via SCIM or tenant offboarding
- **Recipients:** FORM backend only. Not shared with sub-processors beyond Supabase PostgreSQL and Cloudflare R2 (audit archive)

#### 15.9.5 Art. 9 Group Name Scanner

FORM implements a pre-storage scanner on incoming SCIM Group `displayName` values. The scanner uses the following TypeScript regex constants, which mirror the `BLOCKED_ATTRIBUTE_PATTERNS` from §14.5 but are applied to group names rather than SCIM attribute names:

```typescript
// src/scim/art9-group-name-scanner.ts

/**
 * Patterns that may indicate Art. 9 special category data encoded in
 * a SCIM Group displayName. Mirrors BLOCKED_ATTRIBUTE_PATTERNS (§14.5)
 * but applied to group names rather than attribute names.
 *
 * Positive match triggers scim.group_name_sensitive_detected audit event
 * and routes the group for compliance-officer review. The group is NOT
 * rejected — it is stored with a flag and the compliance officer decides
 * whether to retain, rename (via admin dashboard), or request the customer
 * rename the group in their IdP.
 */
export const ART9_GROUP_NAME_PATTERNS: RegExp[] = [
  // Health / medical / disability
  /health/i,
  /medical/i,
  /disability/i,
  /disabilit/i,
  /condition/i,
  /diagnosis/i,
  /patient/i,
  /oncolog/i,
  /mental.?health/i,
  /ptsd/i,
  /recovery/i,
  /rehab/i,
  /wheelchair/i,
  /chronic/i,

  // Religion / belief
  /religion/i,
  /faith/i,
  /christian/i,
  /muslim/i,
  /islam/i,
  /jewish/i,
  /hindu/i,
  /buddhis/i,
  /ramadan/i,
  /prayer/i,

  // Trade union / political
  /union/i,
  /politic/i,
  /party.member/i,
  /activist/i,
  /labour.rep/i,
  /shop.steward/i,

  // Ethnic / racial origin
  /ethnic/i,
  /race/i,
  /national.*origin/i,
  /indigenous/i,
  /minority/i,

  // Sexual orientation / gender identity
  /sex.*orient/i,
  /gender.*ident/i,
  /lgbtq/i,
  /lgbt/i,
  /trans(?:gender)?/i,
  /nonbinary/i,
  /queer/i,
];

/**
 * Scans a SCIM Group displayName for Art. 9 indicators.
 * Returns the first matched pattern or null if no match.
 */
export function scanGroupDisplayName(displayName: string): RegExp | null {
  for (const pattern of ART9_GROUP_NAME_PATTERNS) {
    if (pattern.test(displayName)) {
      return pattern;
    }
  }
  return null;
}
```

**Behaviour on positive match:**

1. FORM **does not reject** the group. Rejection would break the customer's provisioning flow and could leave users without roles. Instead, the group is stored with a `sensitive_name_flagged: true` column in `scim_groups`.
2. A `scim.group_name_sensitive_detected` audit event is written with: `tenant_id`, `group_id`, `display_name` (stored in audit log only — see note below), `matched_pattern_index` (not the pattern text itself, to avoid redundant storage), `action_required: "compliance_review"`.
3. The tenant's compliance officer (or `tenant_owner` if no compliance officer role is assigned) receives an in-dashboard notification: "A newly provisioned group name may contain sensitive data requiring review under Art. 9 GDPR. Please review in the Group Settings page."
4. The compliance officer can either: (a) mark as reviewed and acceptable, (b) request the customer rename the group in their IdP, or (c) manually override the `display_name` in FORM's admin dashboard (the IdP `externalId` and `displayName` are unchanged — this is a FORM-side display override only).

**Note on audit log storage of sensitive group names.** The `displayName` value is stored in the `scim.group_name_sensitive_detected` event to enable the compliance review workflow. This is an intentional exception to the general principle of not storing values in audit logs. The audit log entry is classified as `sensitivity: HIGH` and is accessible only to `compliance_officer` and `security_engineer` roles, not to `tenant_admin` or `tenant_manager`.

---

### 15.10 Implementation Checklist

| # | Task | Owner | Priority | Milestone | Notes |
|---|---|---|---|---|---|
| 1 | Create `scim_groups` and `scim_group_members` tables with RLS policies | platform-engineer | **P0** | M4 | Migration must include lint-check for RLS per §4.3 policy |
| 2 | Implement `GET /Groups` and `GET /Groups/{id}` endpoints | platform-engineer | **P0** | M4 | Include pagination per §15.5.3 |
| 3 | Implement `POST /Groups` endpoint with `(tenant_id, external_id)` idempotency | platform-engineer | **P0** | M4 | |
| 4 | Implement `PUT /Groups/{id}` full-replace endpoint | platform-engineer | **P0** | M4 | Net-add/remove member diff must trigger per-member audit events |
| 5 | Implement `PATCH /Groups/{id}` with `op: add`, `op: remove`, `op: replace` | platform-engineer | **P0** | M4 | |
| 6 | Implement `DELETE /Groups/{id}` endpoint | platform-engineer | **P0** | M4 | |
| 7 | Integrate group-to-role resolution in JIT login flow (§15.4.2 algorithm) | platform-engineer | **P0** | M4 | Must run on every login, not just first provisioning |
| 8 | Atomic member removal + session revocation in single DB transaction (§15.7, §12.7) | platform-engineer | **P0** | M4 | Transaction rollback on session revocation failure — no partial state |
| 9 | Art. 9 group name scanner (§15.9.5) — required before first enterprise customer | security-engineer | **P0** | M4 | No enterprise pilot without this active |
| 10 | All 9 `scim.group_*` audit events wired to DEC-030 HMAC chain | platform-engineer | **P0** | M4 | Includes `scim.group_name_sensitive_detected` |
| 11 | Feature flag `scim_groups` in admin dashboard UI — enable/disable per tenant | platform-engineer | **P1** | M4 | Default off; enterprise-architect enables per customer |
| 12 | SCIM conformance test pass via Okta SCIM 2.0 test tool for Groups endpoints | platform-engineer + enterprise-architect | **P1** | M4 | Use Okta's published SCIM 2.0 compliance test suite |
| 13 | Entra ID nested group integration test — verify `scim.group_nested_flattened` fires correctly | platform-engineer | **P1** | M4 | Test with a real Entra ID dev tenant |
| 14 | Google Workspace JIT documentation update — reference §15.6.3 and G-014 in §7 onboarding checklist | enterprise-architect | **P1** | M4 | Customer-facing disclosure required before any Google Workspace pilot |
| 15 | Bulk group operations (`POST /Bulk` with Group resources) | platform-engineer | **P2** | M5 | Not blocking M4 enterprise pilots |

---

### 15.11 G-014 New Gap: SCIM Groups — Google Workspace

| Field | Value |
|---|---|
| **Gap ID** | G-014 |
| **Summary** | Google Workspace does not support SCIM Group provisioning to third-party SCIM endpoints. Google's SCIM implementation covers user lifecycle (create, update, deactivate) but does not expose a `/Groups` push interface for third-party apps. |
| **Impact** | Enterprise customers using Google Workspace as their sole IdP cannot use SCIM Groups for automated role assignment in FORM. Role assignment must be handled via JIT provisioning with role claims (§11) or manual assignment in the admin dashboard. Joiners/movers/leavers automation is limited compared to Okta and Entra ID deployments. |
| **Status** | 🔴 By-design limitation of Google Workspace SCIM. Not a FORM defect. No remediation available within FORM's control. |
| **Mitigation** | Use JIT role claims per §11. Customer configures a custom attribute or group membership claim in their Google Workspace SAML/OIDC app. FORM evaluates the claim at login and assigns the corresponding FORM role. Role changes require the affected user to re-authenticate to take effect. |
| **Customer disclosure requirement** | Must be disclosed in writing during onboarding for any Google Workspace customer. The disclosure must state: "FORM's SCIM Groups provisioning is not available for Google Workspace. Automated role management uses JIT provisioning with role claims. User deprovisioning requires manual action in the FORM admin dashboard or SCIM User deactivation; group-based automatic deprovisioning is not available." Customer acknowledgement (email or signed onboarding document) must be retained in the customer success record. |
| **SOC 2 impact** | None — CC6.2 (user access management) is satisfied via JIT provisioning. The JIT path is a documented and audited access control mechanism. SOC 2 auditor must be briefed that SCIM Groups is unavailable for Google Workspace customers and that JIT provisioning is the compensating control. No CC6.2 exception is required. |
| **Resolution path** | Monitor Google Workspace SCIM roadmap. Google has not publicly committed to third-party SCIM Groups support as of the current document date. Re-evaluate at M6 (est. Q4 2026). If Google introduces SCIM Groups support, implement using the same endpoint and schema as the Okta/Entra ID paths — no FORM schema changes expected. |

---

*v0.7 additions: Section 15 — SCIM 2.0 Groups Provisioning. New database tables: `scim_groups` (RFC 7643 §8 compliant, RLS-enforced via `current_setting('app.tenant_id')::UUID`) and `scim_group_members` (atomic with session revocation per §12.7). Six SCIM Group endpoints specified with pagination (cursor, `count=100&startIndex=1`), rate-limit interaction with §3.6, and `(tenant_id, external_id)` idempotency. Group-to-role mapping: `group_role_mappings` JSONB config extension to `tenant_sso_configs`, `highest_privilege` resolution algorithm (tenant_admin > coach > member > viewer), `tenant_owner` blocked from group assignment, unmapped group fallback with `scim.group_unmapped` audit event and opt-in strict-block mode. IdP-specific: Okta (Push Groups manual config required — common onboarding gap); Entra ID (nested group flattening §15.6.2a — skips `type:"Group"` members, logs `scim.group_nested_flattened`, flat security group or P1/P2 member flattening recommended; dynamic groups treated as static); Google Workspace (SCIM Groups not supported — JIT via §11 compensating control; G-014 new gap). Nine new audit events added to DEC-030 HMAC chain taxonomy. GDPR: group `displayName` as potential Art. 9 carrier; Art. 9 group name scanner with TypeScript `ART9_GROUP_NAME_PATTERNS` regex constants mirroring §14.5 `BLOCKED_ATTRIBUTE_PATTERNS`; non-blocking flag-and-review workflow (not reject); `scim.group_name_sensitive_detected` event; group membership excluded from analytics and tenant-manager RLS; Art. 17 hard-delete of `scim_group_members` on erasure. Customer onboarding checklist extended for all three IdPs. Implementation checklist: 15 tasks (10× P0, 4× P1, 1× P2), M4/M5. G-014: 🔴 Google Workspace by-design limitation; JIT compensating control documented; SOC 2 CC6.2 satisfied; re-evaluate M6 Q4 2026.*
