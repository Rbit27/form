# FORM · SSO/SCIM Implementation v0.3

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

**v0.3 · May 2026 · enterprise-architect + security-engineer**
**Review trigger: any IdP integration change, any SCIM schema change, or quarterly — whichever comes first.**
*v0.2 additions: Section 10 — Magic Link Fallback Security Design. Threat model (8 attack vectors), OTP spec, rate limits, trigger conditions, verification flow, abuse detection, audit events, admin controls, implementation sequencing, GDPR/DPA note. Closes G-010 design phase. Critical implementation dependencies identified.*

*v0.3 additions: Section 11 — Just-in-Time (JIT) Provisioning Design. JIT vs. SCIM decision tree, provisioning flow with seat-limit and domain-verification gates, SAML and OIDC claim extraction mapping, per-tenant `claim_mapping` JSONB config, seat limit enforcement with FOR UPDATE transaction lock to prevent race condition, JIT-to-SCIM reconciliation (409 Conflict → PATCH adopt path), 6 JIT audit events, admin controls (jit_provisioning_enabled, default_role, notify_on_jit_provision), implementation sequencing. Security note: domain verification is not optional.*
