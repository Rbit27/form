# FORM · SSO/SCIM Implementation v1.2

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
15. [SCIM 2.0 Groups Provisioning](#15-scim-20-groups-provisioning)
16. [Admin Dashboard SSO & SCIM Configuration UI](#16-admin-dashboard-sso--scim-configuration-ui)
17. [Enterprise Pilot Program: 90-Day Runbook](#17-enterprise-pilot-program-90-day-runbook)
18. [Tenant IdP Migration Runbook](#18-tenant-idp-migration-runbook)
19. [SCIM Groups Sync & Group-Based Role Mapping](#19-scim-groups-sync--group-based-role-mapping)
20. [SAML Certificate Lifecycle Management — Proactive Monitoring & Expiry Alerting](#20-saml-certificate-lifecycle-management--proactive-monitoring--expiry-alerting)

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
| G-006 | **`tenant_sso_configs.ip_allowlist` enforcement.** Schema column exists; the Cloudflare Worker that enforces IP allowlist on SSO callbacks is not written. | ~~Medium — promised in enterprise contracts~~ **UI design resolved — see §16.10; Worker implementation pending** | platform-engineer | Estimate: 1 week for Worker once §16.10 spec is approved |
| G-007 | **Admin dashboard SSO configuration UI.** Currently all SSO config changes require direct SQL. The admin dashboard UI for SSO setup, certificate management, and group mapping does not exist. | ~~High — required before first enterprise pilot~~ **Design resolved — see §16** | enterprise-architect (product spec) + platform-engineer (build) | Design complete. Implementation pending. Estimate: 3–4 weeks |
| G-008 | **SCIM group sync for Azure AD GUIDs.** The design handles GUID-based group IDs, but the admin dashboard UI for mapping GUIDs to roles (where the customer IT admin enters human-readable group names and FORM resolves GUIDs via Microsoft Graph) is not designed. | ~~Medium~~ **Design resolved — see §16.8** | enterprise-architect | Decision: call Microsoft Graph API to resolve display names; GUID shown as fallback if consent not granted. See §16.8. |
| G-009 | **Session blocklist persistence.** The design uses a Supabase table for session blocklisting. Under high-revocation scenarios (large tenant SCIM deactivation of 1000+ users simultaneously), this table lookup on every request becomes a hot-path bottleneck. | Medium — performance, not correctness | devops-lead + platform-engineer | Options: Redis cache layer, Bloom filter, JWT short-TTL with revocation threshold. Needs decision before >100-seat enterprise customers. |
| G-010 | **Magic link fallback security design.** The current fallback design in 8.3 and 8.4 uses verified email as the trust anchor. The exact verification flow (OTP via email, rate limiting, abuse detection) is not specified. | ~~High — security-critical~~  **Resolved — see §10** | security-engineer | Design specified in §10. Implementation pending. |
| G-011 | **Tenant SSO configuration test environment.** Customers currently have no sandbox/staging environment to test SSO config before production. This leads to live testing, which breaks existing users if misconfigured. | ~~Medium~~ **Design resolved — see §16.11** | enterprise-architect | Design: dedicated `{slug}-staging.form.coach` endpoint with linked staging `tenant_id`; one-button config copy to production. Implementation pending. |
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

## 16. Admin Dashboard SSO & SCIM Configuration UI

**Owner:** enterprise-architect (product spec), platform-engineer (build)
**Closes:** G-007 design phase, G-008 design phase, G-006 partial design phase, G-011 design phase
**Blocking condition lifted when:** §16.3–16.11 implementation checklist items P0 are complete and a QA pass is signed off by enterprise-architect.

This section is the canonical product specification for the SSO and SCIM configuration interface available to `tenant_owner` and `tenant_admin` roles in the FORM admin dashboard. All SSO/SCIM configuration that currently requires direct database access will be performed exclusively through this UI once it ships. It must be ready before the first enterprise pilot customer goes live.

**Privacy floor:** This interface must never expose individual user data to the configurator. SSO configuration (IdP certificates, SCIM tokens, group-role mappings, IP allowlists) does not constitute user personal data and is appropriately visible to `tenant_owner` and `tenant_admin` roles. Aggregate connection health statistics (login success rate, SCIM provisioning count) are permitted. No session-level or per-user activity data may appear on any configuration screen.

---

### 16.1 Role-Based Access to SSO Configuration

| Action | `tenant_owner` | `tenant_admin` | `tenant_manager` | `member` |
|---|---|---|---|---|
| View SSO connection status | ✅ | ✅ | ❌ | ❌ |
| Run OIDC or SAML setup wizard | ✅ | ❌ | ❌ | ❌ |
| Edit existing SSO configuration | ✅ | ❌ | ❌ | ❌ |
| Download SP metadata | ✅ | ✅ | ❌ | ❌ |
| Rotate SP signing certificate | ✅ | ❌ | ❌ | ❌ |
| Generate / revoke SCIM token | ✅ | ❌ | ❌ | ❌ |
| Edit group-to-role mappings | ✅ | ✅ | ❌ | ❌ |
| Edit session policy | ✅ | ❌ | ❌ | ❌ |
| Edit IP allowlist | ✅ | ❌ | ❌ | ❌ |
| Promote staging config to production | ✅ | ❌ | ❌ | ❌ |

**Access gate:** All SSO configuration screens are gated by `feature_flags.sso_enabled = true` for the tenant. Attempting to access any SSO screen when the flag is false returns HTTP 403 with error code `sso_not_enabled`. The admin dashboard renders the SSO nav item as disabled with tooltip: "SSO is available on the Growth and Enterprise plans. Contact your FORM account manager to upgrade."

**Audit requirement:** Every write action in this section is an audit event on the DEC-030 HMAC chain. The `performed_by` field is the `user_id` of the `tenant_owner` or `tenant_admin` executing the action, never a service account.

---

### 16.2 Dashboard Navigation Structure

The SSO & SCIM configuration is a top-level section within the admin dashboard, accessible from the left sidebar under **Settings → Identity & Access**.

```
Settings
├── General (org name, logo, timezone)
├── Identity & Access          ← §16
│   ├── Identity Provider      (SSO setup, status, metadata)
│   ├── SCIM Provisioning      (token, provisioning log)
│   ├── Group Mappings         (role assignment rules)
│   ├── Session Policy         (timeouts, concurrent sessions)
│   ├── IP Allowlist           (network access control)
│   └── Staging Environment    (pre-production SSO testing)
├── Billing & Plan
└── Security & Audit
```

The **Identity Provider** sub-page hosts the setup wizard when no IdP is configured. Once an IdP is configured, it shows the connection status panel with a link to edit.

---

### 16.3 Identity Provider Setup Wizard — OIDC

**Trigger:** `tenant_owner` clicks "Set up SSO" with no existing configuration, then selects "OpenID Connect (OIDC)".

**Step 1 — IdP Selection**

Radio buttons with logos:
- **Okta** — pre-populates discovery URL template: `https://{your-okta-domain}/.well-known/openid-configuration`
- **Microsoft Entra ID (Azure AD)** — input: Entra tenant ID or domain; auto-constructs: `https://login.microsoftonline.com/{tenant-id}/v2.0/.well-known/openid-configuration`
- **Google Workspace** — discovery URL: `https://accounts.google.com/.well-known/openid-configuration` (fixed)
- **Generic OIDC** — manual entry mode

**Step 2 — OIDC Credentials**

| Field | Requirement | Notes |
|---|---|---|
| Discovery URL | Required | Validated via fetch at submit time; must return a JSON document with `issuer`, `authorization_endpoint`, `token_endpoint`, `jwks_uri` |
| Client ID | Required | Stored as `tenant_sso_configs.oidc_client_id` (plaintext — not secret) |
| Client Secret | Required | AES-256-GCM encrypted; key stored in Cloudflare Workers Secrets as `OIDC_CLIENT_SECRET_ENCRYPTION_KEY`; ciphertext stored in `tenant_sso_configs.oidc_client_secret_enc` |
| Additional Scopes | Optional | Default: `openid email profile`; append space-separated additional scopes (e.g., `groups`) |

Discovery URL validation: FORM fetches the URL server-side (from the Cloudflare Worker, never client-side) and confirms the response is valid OIDC metadata. If the fetch fails or the response is malformed, show inline error: "Could not fetch IdP metadata — check the URL and try again."

**Step 3 — Attribute Mapping**

| OIDC Claim | FORM Field | Default | Editable |
|---|---|---|---|
| `email` | `users.email` | `email` | No — email is always the primary identifier |
| `name` or `given_name` + `family_name` | `users.display_name` | `name` | Yes |
| `sub` | `users.idp_subject_id` | `sub` | No |
| Custom claim (e.g., `groups`) | Role mapping source | `groups` | Yes |

Attribute mapping is stored in `tenant_sso_configs.attribute_mapping` JSONB. The UI shows a two-column table: "IdP claim" (text input) → "FORM field" (dropdown). A "Test Claim Extraction" button appears in Step 4 to validate the mapping with a real login.

**Step 4 — Test Login (Sandboxed)**

FORM opens a new browser tab to `https://app.form.coach/auth/sso/test?tenant_id=<uuid>&session=<nonce>`. The test flow:
1. Redirects to the IdP's authorization endpoint using the configured credentials.
2. After user authenticates, returns to `https://app.form.coach/auth/sso/test/callback`.
3. FORM validates the ID token, extracts configured attributes, and displays a success summary:
   - **Resolved email:** `alice@acme.com`
   - **Resolved display name:** `Alice Nakamura`
   - **Groups claim value:** `["form-admins", "all-employees"]`
   - **Role that would be assigned:** `tenant_admin` (via group mapping)
4. If validation fails, display the specific error: token signature invalid / issuer mismatch / missing required claim / email not present.

The test login does **not** create a session or provision a user. It is purely a validation handshake.

`sso_config.test_login_initiated` and `sso_config.test_login_passed` / `sso_config.test_login_failed` audit events are written regardless of outcome.

**Step 5 — Go-Live**

Toggle: "Enable SSO for my organization."

On enable:
- Sets `tenant_sso_configs.is_active = true`.
- Sets `tenant_sso_configs.enforced = false` by default (SSO available but not mandatory; users can still use magic link).
- Secondary toggle: "Require SSO for all users (disable magic link login)" — sets `tenant_sso_configs.enforced = true`. Warning banner: "This will prevent users without an IdP account from logging in. Ensure all users have IdP accounts before enabling."
- Writes `sso_config.activated` audit event with `protocol: 'oidc'`, `idp_type`, `enforced: bool`.

---

### 16.4 Identity Provider Setup Wizard — SAML 2.0

**Trigger:** `tenant_owner` clicks "Set up SSO" and selects "SAML 2.0".

**Step 1 — IdP Selection**

Same IdP choices as §16.3 Step 1, plus:
- **Microsoft ADFS** — flag shown: "ADFS requires manual certificate upload on each rotation. See §16.5 for the rotation procedure."
- **OneLogin**
- **Ping Identity**

**Step 2 — SP Metadata (FORM → IdP direction)**

Before entering IdP details, the admin must configure FORM as a Service Provider in their IdP. Display:

| FORM SP Field | Value |
|---|---|
| **SP Entity ID** | `https://app.form.coach/saml/sp/{tenant-slug}` |
| **ACS URL (Assertion Consumer Service)** | `https://app.form.coach/auth/saml/callback/{tenant-slug}` |
| **SLO URL** | `https://app.form.coach/auth/saml/slo/{tenant-slug}` |
| **SP Signing Certificate** | [Download PEM] [Download DER] |
| **NameID Format** | `urn:oasis:names:tc:SAML:1.1:nameid-format:emailAddress` |

"Download metadata.xml" button generates a complete SP metadata document and triggers a file download. The admin uploads this to their IdP.

**Step 3 — IdP Metadata (IdP → FORM direction)**

Two entry modes (tab selector):

*Upload XML:*
- File picker for IdP metadata XML.
- FORM parses: `entityID`, `SingleSignOnService Location`, `SingleLogoutService Location`, `X509Certificate`.
- Shows parsed values for confirmation before saving.

*Manual Entry:*
| Field | Description |
|---|---|
| IdP Entity ID | The IdP's `entityID` |
| SSO URL | `SingleSignOnService Location` (HTTP-Redirect or HTTP-POST) |
| SLO URL | `SingleLogoutService Location` (optional; if absent, SLO is disabled) |
| IdP X.509 Certificate | Paste PEM-encoded certificate |

Stored as `tenant_sso_configs.saml_idp_metadata_xml` (the raw XML) and parsed fields in individual columns for fast lookup.

**Step 4 — Test Login**

Same sandboxed flow as §16.3 Step 4, but triggers an SP-initiated SAML AuthnRequest to the IdP SSO URL. Success summary shows:
- **NameID value:** `alice@acme.com`
- **Attributes in assertion:** `email`, `displayName`, `http://schemas.xmlsoap.org/claims/Group`
- **Mapped attributes:** email → `alice@acme.com`, role → `tenant_admin`

**Step 5 — Attribute Mapping**

Same two-column table as §16.3 Step 3, but source keys are SAML assertion attribute names (e.g., `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress`) rather than OIDC claim names. Pre-populated defaults by IdP type:

| IdP | Default email attribute |
|---|---|
| Okta | `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress` |
| Entra ID | `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress` |
| Google Workspace | `email` (NameID, not attribute) |
| ADFS | `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress` |
| Generic | Manual entry |

**Step 6 — Go-Live**

Same toggle and enforce options as §16.3 Step 5. `sso_config.activated` event with `protocol: 'saml'`.

---

### 16.5 SP Metadata & Certificate Viewer

Available on the **Identity Provider** page once SSO is configured. Accessible to `tenant_owner` and `tenant_admin` (read-only for `tenant_admin`).

**Connection status banner:**
```
● Connected — Okta (OIDC)  |  Last successful login: 3 minutes ago  |  147 active sessions
```
Status is derived from `sso_login_stats` (an in-memory cache in the Cloudflare Worker; refreshed every 60 seconds from `audit_log` counts). It shows the IdP type, protocol, last `sso.login` event timestamp, and active session count from `enterprise_sessions`.

**SP Certificate panel:**

| Field | Value |
|---|---|
| Serial number | `4a:f2:...` |
| Subject | `CN=form-sp-{tenant-slug}` |
| Not Before | `2026-01-01` |
| Not After | `2027-01-01` |
| Days until expiry | `224` (colour-coded: green > 90 days, yellow 30–90, red < 30) |
| SHA-256 fingerprint | `ab:cd:ef:...` |

**Actions:**

- **Download PEM** — triggers file download of the public certificate.
- **Download metadata.xml** — regenerates and downloads the full SP metadata document with current certificate.
- **Rotate Certificate** — see §8.1 for the dual-cert overlap procedure. This button initiates that procedure and is restricted to `tenant_owner`. Confirmation dialog: "A new certificate will be added to SP metadata. You must update your IdP with the new metadata before the old certificate expires. Do not remove the old certificate until you have confirmed the IdP is using the new one." After confirmation writes `sso_cert.rotation_initiated` audit event.

**ADFS-specific warning (shown only when `idp_type = 'adfs'`):**
> ADFS requires manual certificate upload. When you rotate the FORM SP certificate, download the new SP metadata and upload the updated X.509 certificate in your ADFS Relying Party Trust configuration before the old certificate's expiry date. FORM will send an email reminder to the `tenant_owner` 60 and 30 days before expiry.

---

### 16.6 SCIM Token Management

Available on the **SCIM Provisioning** sub-page. Restricted to `tenant_owner`.

**Token display:**

```
SCIM Base URL:    https://app.form.coach/scim/v2
Bearer Token:     •••••••••••••••••••••••••••••••••••••••••••••••••• abcd1234
Token Age:        47 days  ✓
                  [Rotate Token]  [Copy Base URL]  [Copy Token]
```

Token is stored in `tenant_sso_configs.scim_token_hash` (bcrypt hash) and `scim_token_preview` (last 8 characters, plaintext). The full token is only displayed once at generation time and cannot be recovered — only rotated.

**Token age warning:** If `scim_token_generated_at < NOW() - INTERVAL '90 days'`, show amber banner: "Your SCIM token is 90+ days old. Rotate it to comply with your security policy." SOC 2 CC6.1 requires credential rotation; this banner is the nudge mechanism.

**Rotate Token flow:**
1. Confirmation dialog: "This will immediately invalidate the current token. Your IdP will fail to provision users until you update it with the new token. Do you want to continue?"
2. New token generated server-side (32 random bytes, base64url-encoded).
3. New token displayed in a one-time reveal modal: "Copy this token now. It will not be shown again."
4. `scim_token.generated` audit event written. Previous `scim_token.revoked` event written simultaneously.
5. 15-minute overlap window: both old and new tokens are accepted during transition (stored as `scim_token_hash_previous` with a `scim_token_previous_expires_at` timestamp). After 15 minutes, the previous token is rejected.

**SCIM provisioning summary (read-only, last 30 days):**

| Metric | Value |
|---|---|
| Users provisioned | 47 |
| Users deactivated | 3 |
| Last sync | 4 minutes ago |
| Provisioning errors | 0 |

These numbers are aggregate counts from `audit_log` events of type `scim.user_created`, `scim.user_deactivated`, `scim.user_error`. No individual user identifiers are shown on this screen (privacy floor).

---

### 16.7 Group-to-Role Mapping UI

Available on the **Group Mappings** sub-page. Accessible to `tenant_owner` (edit) and `tenant_admin` (edit, if permitted by session policy — see §16.9).

**Mapping table:**

| IdP Group | FORM Role | Actions |
|---|---|---|
| `form-admins` (Okta) | `tenant_admin` | Edit · Remove |
| `form-managers` (Okta) | `tenant_manager` | Edit · Remove |
| `all-employees` (Okta) | `member` | Edit · Remove |
| `00abc123-def4-...` (Entra ID) | `tenant_admin` | Edit · Remove · Resolve → |

For Entra ID rows where the group identifier is a GUID, show a **Resolve →** action that triggers the Graph API call (§16.8). Once resolved, the display name appears in parentheses: `00abc123-def4-... (Engineering Leadership)`.

**Add mapping:**

```
IdP Group Identifier: [ form-coaches           ]  (text input — enter group name or GUID)
FORM Role:            [ Coach                  ▼]
[ Add Mapping ]
```

Writes to `tenant_sso_configs.group_role_mappings` JSONB array:
```jsonc
[
  { "idp_group_id": "form-coaches", "form_role": "coach", "resolved_display_name": null },
  { "idp_group_id": "00abc123-...", "form_role": "tenant_admin", "resolved_display_name": "Engineering Leadership" }
]
```

Note: `tenant_owner` is excluded from group assignment per §5.3 — it cannot appear in the FORM Role dropdown and is validated server-side.

**Fallback role:**

Dropdown: "Users whose groups do not match any mapping will be assigned: `[ member ▼ ]`". Options: `member`, `viewer`, `deny` (deny login with error: group not mapped). Default: `member`. Stored in `tenant_sso_configs.unmapped_group_fallback_role`.

**Strict mode toggle:**

"Deny login if no group claim present (require all users to be in a mapped group)" — maps to `tenant_sso_configs.group_strict_mode: boolean`. Shown with warning: "Enabling strict mode will block login for any user whose IdP does not send a groups claim. Verify attribute mapping is correct before enabling." Writes `sso_config.group_mapping_updated` audit event with `strict_mode_changed: true`.

---

### 16.8 Azure AD Group GUID Resolution

**Problem (G-008):** Microsoft Entra ID sends group object GUIDs in SAML assertions and SCIM Group `id` fields — not human-readable display names. IT admins cannot identify a GUID like `00abc123-def4-5678-9012-abcdef012345` without consulting the Azure Portal.

**Solution:** FORM calls the Microsoft Graph API with a per-tenant app registration to resolve GUIDs to display names on-demand. This is an opt-in operation triggered by the admin — FORM does not perform background group sync.

**Schema additions to `tenant_sso_configs`:**

```sql
ALTER TABLE tenant_sso_configs
  ADD COLUMN graph_app_id          TEXT,
  ADD COLUMN graph_app_secret_key_ref TEXT,  -- Cloudflare Workers Secrets key name, not the secret itself
  ADD COLUMN graph_tenant_id       TEXT,     -- Entra ID tenant ID (GUID)
  ADD COLUMN graph_consent_granted_at TIMESTAMPTZ,
  ADD COLUMN graph_last_group_sync_at TIMESTAMPTZ;
```

`graph_app_secret_key_ref` stores the Cloudflare Workers Secrets key name (e.g., `GRAPH_SECRET_acme`). The actual client secret lives in Cloudflare Workers Secrets, never in the database.

**UI flow — "Connect Microsoft Graph":**

Shown only when `idp_type IN ('entra_id', 'azure_ad')`. Button: **Resolve Group Names (Microsoft Graph)**.

Step 1: Display instructions to create an Azure app registration:
```
In the Azure Portal:
1. Navigate to App Registrations → New registration.
2. Name: "FORM Group Sync"
3. Supported account types: Accounts in this organizational directory only
4. After creation, note the Application (client) ID.
5. Create a Client Secret under Certificates & Secrets.
6. Grant API permission: Microsoft Graph → Application → Group.Read.All
7. Grant admin consent.
```

Step 2: Input fields:
- Application (client) ID → stored as `graph_app_id`
- Client Secret → encrypted and stored in Cloudflare Workers Secrets, referenced by `graph_app_secret_key_ref`
- Tenant ID → stored as `graph_tenant_id` (auto-populated from existing Entra ID config if available)

Step 3: "Test Connection" — FORM calls `GET https://graph.microsoft.com/v1.0/groups?$top=1` to validate credentials. On success: "Connected to Microsoft Graph. You can now resolve group names."

**Resolving GUIDs:**

On any group mapping row where the IdP group identifier is a GUID, clicking **Resolve →** calls:
```
GET https://graph.microsoft.com/v1.0/groups/{guid}?$select=id,displayName
```

Response `displayName` is written to `group_role_mappings[].resolved_display_name`. The GUID remains the authoritative identifier used in SSO assertion matching; the display name is for human readability only.

**Batch resolve:** "Resolve All" button runs a batch request for all unresolved GUIDs in the mapping table using Microsoft Graph batch API (`POST /v1.0/$batch`). Maximum 20 GUIDs per batch request; pagination handled automatically.

**Graceful fallback:** If Graph credentials are not configured, or the Graph API call fails, the GUID is shown as-is. The admin may also enter a manual alias by clicking "Add alias" on any unresolved row, which sets `resolved_display_name` without calling Graph.

**Privacy:** The Graph API is only called when the `tenant_owner` explicitly triggers the resolve action. FORM stores only the `displayName` of mapped groups — not member lists, not user directory data. `graph_api.group_resolved` audit event is written for every successful resolution, recording `group_guid` and `resolved_display_name`.

---

### 16.9 Session Policy Configuration Panel

Available on the **Session Policy** sub-page. Restricted to `tenant_owner`.

| Setting | Type | Default | Constraint |
|---|---|---|---|
| Session timeout | Integer (hours) | `168` (7 days) | 1–720 |
| Maximum concurrent sessions per user | Integer | `∞` (unlimited) | 1–10, or 0 for unlimited |
| Force SSO (disable magic link) | Boolean | `false` | Must have ≥ 1 active SSO config before enabling |
| Require re-authentication for sensitive actions | Boolean | `false` | Sensitive actions: delete org, remove tenant_owner, export audit log |
| Allow member-initiated session termination | Boolean | `true` | If false, members cannot log out (kiosk mode for managed devices) |

All settings written to `tenant_sso_configs.session_policy` JSONB column (existing column, populated by §12). UI displays current values with inline edit capability (no separate "save" step per field — autosave with confirmation toast).

**Force SSO warning:** "Enabling this will block all login paths except SSO, including the emergency magic link. If your IdP is unavailable, users will be locked out. Ensure you have a documented IdP outage procedure (see §8.4) before enabling." A secondary confirmation checkbox is required: "I have an IdP outage recovery plan and understand the lockout risk."

Writes `sso_config.session_policy_updated` audit event with `changed_fields: ["session_timeout_hours", "force_sso"]` (field names only, no values logged).

---

### 16.10 IP Allowlist Configuration Panel

**Design specification for G-006 closure.**

Available on the **IP Allowlist** sub-page. Restricted to `tenant_owner`.

**Warning banner (always visible on this page):**
> Ensure all IP addresses used by your administrators are included in the allowlist before enabling. Once enabled, any IP not in the list will be blocked from SSO login — including your own. To recover from a lockout, contact FORM support. FORM support can disable the allowlist for your tenant using break-glass admin access (HMAC-chained audit event written).

**CIDR entry:**

```
IP / CIDR:   [ 10.0.0.0/8          ]   [Add]
             [ 203.0.113.45/32     ]   [Add]
```

Validations:
- Must be valid IPv4 or IPv6 CIDR notation.
- Rejects private RFC 1918 ranges when enforcement is enabled (warn-only: "This is a private IP range. Ensure it matches your network topology.").
- Maximum 50 CIDR entries per tenant (prevents misconfigured overly-broad allowlists).

**Current allowlist table:**

| CIDR | Label (optional) | Added by | Added at | Actions |
|---|---|---|---|---|
| `10.0.0.0/8` | Corporate VPN | alice@acme.com | 2026-05-20 | Remove |
| `203.0.113.0/24` | Office | alice@acme.com | 2026-05-20 | Remove |

Label is a free-text annotation for human readability. Stored in `tenant_sso_configs.ip_allowlist` JSONB array:
```jsonc
[
  { "cidr": "10.0.0.0/8", "label": "Corporate VPN", "added_by": "<user_id>", "added_at": "2026-05-20T..." },
  { "cidr": "203.0.113.0/24", "label": "Office", "added_by": "<user_id>", "added_at": "2026-05-20T..." }
]
```

**Enable/disable toggle:** "Block SSO login for IPs outside this list" — disabled by default. Two-step confirmation when enabling: type the org slug to confirm.

**Cloudflare Worker enforcement spec (for G-006 implementation):**

The Worker that handles the SSO callback route (`/auth/saml/callback/*` and `/auth/oidc/callback`) must read `tenant_sso_configs.ip_allowlist` and the request's `CF-Connecting-IP` header:

```typescript
async function enforceIpAllowlist(
  request: Request,
  tenantSsoConfig: TenantSsoConfig
): Promise<Response | null> {
  if (!tenantSsoConfig.ip_allowlist_enabled || tenantSsoConfig.ip_allowlist.length === 0) {
    return null;  // pass-through
  }
  const clientIp = request.headers.get('CF-Connecting-IP');
  if (!clientIp) return errorResponse(403, 'ip_allowlist_no_client_ip');
  const allowed = tenantSsoConfig.ip_allowlist.some(entry => cidrContains(entry.cidr, clientIp));
  if (!allowed) {
    await writeAuditEvent({ event_type: 'sso.ip_blocked', client_ip: clientIp, tenant_id: tenantSsoConfig.tenant_id });
    return errorResponse(403, 'ip_not_allowed', 'Your IP address is not permitted to access this organization.');
  }
  return null;  // pass-through
}
```

`cidrContains` must handle both IPv4 and IPv6. Use a well-tested library (e.g., `ip-cidr` npm package) — do not implement CIDR matching manually.

The allowlist is read from Supabase on every SSO callback. Cache with a 60-second TTL in the Worker's in-memory cache keyed by `tenant_id`. Cache invalidation is triggered by the `sso_config.ip_allowlist_updated` audit event via a Cloudflare D1 or KV write that the Worker polls.

Writes `sso_config.ip_allowlist_updated` audit event on any add/remove/enable/disable action.

---

### 16.11 SSO Configuration Staging Environment

**Design specification for G-011 closure.**

Availability: Growth and Enterprise plans only. Shown as disabled with upgrade prompt on Starter.

**Purpose:** Allow IT admins to test a complete SSO configuration against a production-identical FORM environment before enabling it for real users. A misconfigured SSO deployment in production causes immediate login failures for all users — the staging environment eliminates this risk.

**Architecture:**

Each enterprise tenant receives a linked staging tenant at provisioning time:

```sql
-- Schema additions to tenants table
ALTER TABLE tenants
  ADD COLUMN staging_tenant_id UUID REFERENCES tenants(id),
  ADD COLUMN is_staging         BOOLEAN NOT NULL DEFAULT false;
```

Constraints:
- A staging tenant always has `is_staging = true` and a non-null reference to its production parent via `staging_tenant_id`.
- A production tenant has `staging_tenant_id` pointing to its staging sibling.
- Staging tenants are not visible in FORM's tenant directory, billing, or analytics.
- No actual user workout or health data is ever copied to a staging tenant.

**Staging tenant properties:**

| Property | Staging Value |
|---|---|
| Subdomain | `{slug}-staging.form.coach` |
| `is_staging` | `true` |
| Features enabled | SSO, SCIM (both enabled regardless of plan — staging is always full-feature) |
| Billing | Not billed; linked to production tenant's contract |
| Users | Synthetic test users only (see below) |
| Data | Completely isolated from production; no production data |

**Synthetic test users:**

On staging tenant creation, FORM auto-creates 5 synthetic test user accounts:
```
test-owner@{slug}-staging.form.coach      (role: tenant_owner)
test-admin@{slug}-staging.form.coach      (role: tenant_admin)
test-manager@{slug}-staging.form.coach    (role: tenant_manager)
test-member-1@{slug}-staging.form.coach   (role: member)
test-member-2@{slug}-staging.form.coach   (role: member)
```

The `tenant_owner` may add additional test users via the admin dashboard (invite by email). These accounts have no real personal data. They are excluded from GDPR Art. 17 erasure requests unless the erasure is for the test email address specifically.

**SSO configuration on staging:**

Staging has its own independent `tenant_sso_configs` row. The admin configures SSO on staging using the same wizard as §16.3/§16.4, pointing to the staging ACS URL: `https://{slug}-staging.form.coach/auth/saml/callback/{slug}-staging`.

The IT admin must add the staging ACS URL to their IdP as a second, separate application (or a second ACS URL if their IdP supports multiple). The SP entity ID for staging is `https://{slug}-staging.form.coach/saml/sp/{slug}-staging`, distinct from the production entity ID.

**"Promote to Production" button:**

Once staging SSO is confirmed working via test logins, the admin clicks **Promote Config to Production**:

1. Confirmation dialog: "This will overwrite your production SSO configuration with the staging configuration. Existing user sessions will not be interrupted. New logins will use the promoted configuration immediately."
2. FORM copies `tenant_sso_configs` fields from staging to production: `protocol`, `oidc_*`, `saml_*`, `attribute_mapping`, `group_role_mappings`, `unmapped_group_fallback_role`.
3. Fields **not** copied: `scim_token_hash` (SCIM token remains independent), `ip_allowlist` (network policy is environment-specific), `session_policy` (policy is environment-specific), `is_active` (production activation is a separate go-live step).
4. After copy, `sso_config.activated` is **not** automatically triggered — the admin must explicitly enable SSO via the go-live toggle (§16.3 Step 5 / §16.4 Step 6).
5. `sso_staging.config_promoted` audit event written with `from_staging_tenant_id`, `to_production_tenant_id`, `fields_copied`.

**Privacy:** Staging tenants are isolated at the RLS level. Every RLS policy that checks `current_setting('app.current_tenant_id')` implicitly isolates staging from production because they have different `tenant_id` UUIDs. A production-tenant query can never accidentally read staging data.

---

### 16.12 Audit Events

All events below are appended to the DEC-030 HMAC chain. `performed_by` is the `user_id` of the acting `tenant_owner` or `tenant_admin`.

| Event Type | Trigger | Key Metadata Fields |
|---|---|---|
| `sso_config.wizard_started` | Admin opens OIDC or SAML setup wizard | `protocol`, `idp_type`, `tenant_id` |
| `sso_config.test_login_initiated` | Admin clicks "Test Login" in wizard (§16.3 Step 4 / §16.4 Step 4) | `protocol`, `idp_type`, `test_nonce` |
| `sso_config.test_login_passed` | Sandboxed test login succeeds | `protocol`, `idp_type`, `resolved_email`, `resolved_role` |
| `sso_config.test_login_failed` | Sandboxed test login fails | `protocol`, `idp_type`, `failure_reason` |
| `sso_config.activated` | Admin enables SSO via go-live toggle | `protocol`, `idp_type`, `enforced: bool` |
| `sso_config.deactivated` | Admin disables SSO | `protocol`, `idp_type` |
| `sso_config.deleted` | Admin deletes SSO configuration entirely | `protocol`, `idp_type` |
| `sso_cert.rotation_initiated` | Admin clicks "Rotate Certificate" (§16.5) | `cert_serial_old`, `cert_expiry_old`, `idp_type` |
| `sso_cert.rotation_completed` | Old certificate removed after dual-cert overlap window | `cert_serial_new`, `cert_expiry_new` |
| `scim_token.generated` | Admin generates a new SCIM token (§16.6) | `token_preview` (last 8 chars only) |
| `scim_token.revoked` | Previous SCIM token revoked on rotation | `token_preview`, `revoked_at` |
| `sso_config.ip_allowlist_updated` | Admin adds/removes/enables/disables an IP allowlist entry (§16.10) | `action` (add\|remove\|enable\|disable), `cidr` (if add/remove), `enabled_state` |
| `sso_config.session_policy_updated` | Admin changes any session policy field (§16.9) | `changed_fields` (names only, no values) |
| `sso_config.group_mapping_updated` | Admin adds/removes/edits a group mapping (§16.7) | `action` (add\|remove\|edit), `idp_group_id`, `form_role` (not logged on remove) |
| `graph_api.group_resolved` | Azure AD GUID resolved via Microsoft Graph (§16.8) | `group_guid`, `resolved_display_name` |
| `sso_staging.config_promoted` | Staging SSO config promoted to production (§16.11) | `from_staging_tenant_id`, `to_production_tenant_id`, `fields_copied` (list of field names) |

All events use HMAC-SHA256 chaining per DEC-030. The `previous_hash` is the hash of the immediately preceding event in the audit log for this tenant.

---

### 16.13 SOC 2 Evidence Mapping

| SOC 2 Criterion | How §16 Satisfies It |
|---|---|
| **CC6.1 — Logical and physical access controls** | SCIM token management panel (§16.6) enforces credential rotation; 90-day age alert provides the detective control. `scim_token.generated` and `scim_token.revoked` events create an auditable rotation trail. |
| **CC6.2 — Prior to issuing system credentials** | Group-to-role mapping UI (§16.7) is the documented mechanism through which enterprise admins define access scope for all SSO-provisioned users. Changes are HMAC-chained audit events. Auditor can query the event log to verify that role assignments were authorised by a `tenant_owner` before going live. |
| **CC6.3 — Network access controls** | IP allowlist panel (§16.10) is the documented control for restricting SSO login to known network ranges. `sso_config.ip_allowlist_updated` events provide the auditable change trail. The Cloudflare Worker enforcement spec ensures the control is implemented consistently — auditors may review the Worker code to verify enforcement. |
| **CC8.1 — Change management** | Every SSO/SCIM configuration change generates a HMAC-chained audit event (§16.12). Changes cannot be made through direct database access without a break-glass `form_admin` event being recorded in the audit log. The staging environment (§16.11) provides a structured change management process: test in staging, review test results, promote to production. |
| **CC6.7 — Restriction of authorised users** | Session policy panel (§16.9) documents the controls for session timeout, concurrent session limits, and forced re-authentication. Force SSO toggle provides an additional control that removes lower-assurance login paths for tenants that require it. |

**Auditor evidence artefacts:**
- `audit_log` events of type `sso_config.*`, `scim_token.*`, `sso_cert.*`, `graph_api.*`, `sso_staging.*` for the audit period are exported per §8.4 of `docs/AUDIT_LOG_SCHEMA.md`.
- SCIM token rotation history: `scim_token.generated` and `scim_token.revoked` event pairs with timestamps demonstrate rotation cadence.
- IP allowlist change log: all `sso_config.ip_allowlist_updated` events for the period demonstrate the allowlist was actively managed.

---

### 16.14 Implementation Checklist

| Task | Owner | Priority | Milestone |
|---|---|---|---|
| Build OIDC setup wizard (§16.3) — all 5 steps including server-side discovery URL validation and sandboxed test login | platform-engineer | **P0** | M4 |
| Build SAML setup wizard (§16.4) — all 6 steps including SP metadata display, IdP metadata XML parser, and sandboxed test login | platform-engineer | **P0** | M4 |
| Build SP metadata viewer + certificate panel (§16.5) — expiry countdown, PEM/metadata download, rotation button wiring to §8.1 procedure, ADFS warning | platform-engineer | **P0** | M4 |
| Build SCIM token management panel (§16.6) — masked display, rotate flow with 15-minute overlap window, 90-day age alert | platform-engineer | **P0** | M4 |
| Build group-to-role mapping table (§16.7) — add/remove/edit, fallback role, strict mode toggle, write to `group_role_mappings` JSONB | platform-engineer | **P0** | M4 |
| Implement Microsoft Graph GUID resolver (§16.8) — per-tenant app registration flow, client credentials token exchange, `GET /groups/{guid}` call, batch resolve, manual alias fallback | platform-engineer | **P0** | M4 |
| Add `graph_app_id`, `graph_app_secret_key_ref`, `graph_tenant_id`, `graph_consent_granted_at`, `graph_last_group_sync_at` columns to `tenant_sso_configs` | platform-engineer | **P0** | M4 |
| Build session policy panel (§16.9) — timeout, concurrent sessions, force-SSO toggle with lockout warning, re-auth gate | platform-engineer | **P0** | M4 |
| Build IP allowlist UI (§16.10) — CIDR add/remove, enable toggle with slug-confirmation modal, lockout warning | platform-engineer | **P1** | M4 |
| Implement Cloudflare Worker IP allowlist enforcement (§16.10 Worker spec) — `CF-Connecting-IP` check, 60-second TTL cache, `sso.ip_blocked` audit event | platform-engineer | **P1** | M4 |
| Build staging environment UI and provisioning (§16.11) — staging tenant auto-creation at contract sign, synthetic test users, independent SSO config wizard, "Promote to Production" button | platform-engineer | **P1** | M5 |
| Add `staging_tenant_id` and `is_staging` columns to `tenants` table; migration + RLS verification | platform-engineer | **P1** | M5 |
| Wire all 16 audit events (§16.12) to DEC-030 HMAC chain writer | platform-engineer | **P0** | M4 |
| Accessibility audit: all wizard steps, form controls, and table actions must meet WCAG 2.1 AA | qa-lead | **P1** | M4 |
| QA end-to-end test: OIDC wizard → test login → go-live → force-SSO → rotate cert → SCIM token rotate — against a real Okta dev tenant | qa-lead + enterprise-architect | **P0** | M4 |

**G-006 status update:** 🔴 → 🟡 Partial. IP allowlist UI spec complete. Worker implementation spec (§16.10) is the remaining engineering task. Unblocked from G-007.
**G-007 status update:** 🔴 → 🟡 Partial. Design complete. Implementation pending per checklist above.
**G-008 status update:** 🔴 → 🟡 Partial. Graph API resolver designed. Implementation pending per checklist above.
**G-011 status update:** 🔴 → 🟡 Partial. Staging environment designed. Infrastructure provisioning pending.

---

*v0.7 additions: Section 15 — SCIM 2.0 Groups Provisioning. New database tables: `scim_groups` (RFC 7643 §8 compliant, RLS-enforced via `current_setting('app.tenant_id')::UUID`) and `scim_group_members` (atomic with session revocation per §12.7). Six SCIM Group endpoints specified with pagination (cursor, `count=100&startIndex=1`), rate-limit interaction with §3.6, and `(tenant_id, external_id)` idempotency. Group-to-role mapping: `group_role_mappings` JSONB config extension to `tenant_sso_configs`, `highest_privilege` resolution algorithm (tenant_admin > coach > member > viewer), `tenant_owner` blocked from group assignment, unmapped group fallback with `scim.group_unmapped` audit event and opt-in strict-block mode. IdP-specific: Okta (Push Groups manual config required — common onboarding gap); Entra ID (nested group flattening §15.6.2a — skips `type:"Group"` members, logs `scim.group_nested_flattened`, flat security group or P1/P2 member flattening recommended; dynamic groups treated as static); Google Workspace (SCIM Groups not supported — JIT via §11 compensating control; G-014 new gap). Nine new audit events added to DEC-030 HMAC chain taxonomy. GDPR: group `displayName` as potential Art. 9 carrier; Art. 9 group name scanner with TypeScript `ART9_GROUP_NAME_PATTERNS` regex constants mirroring §14.5 `BLOCKED_ATTRIBUTE_PATTERNS`; non-blocking flag-and-review workflow (not reject); `scim.group_name_sensitive_detected` event; group membership excluded from analytics and tenant-manager RLS; Art. 17 hard-delete of `scim_group_members` on erasure. Customer onboarding checklist extended for all three IdPs. Implementation checklist: 15 tasks (10× P0, 4× P1, 1× P2), M4/M5. G-014: 🔴 Google Workspace by-design limitation; JIT compensating control documented; SOC 2 CC6.2 satisfied; re-evaluate M6 Q4 2026.* New database tables: `scim_groups` (RFC 7643 §8 compliant, RLS-enforced via `current_setting('app.tenant_id')::UUID`) and `scim_group_members` (atomic with session revocation per §12.7). Six SCIM Group endpoints specified with pagination (cursor, `count=100&startIndex=1`), rate-limit interaction with §3.6, and `(tenant_id, external_id)` idempotency. Group-to-role mapping: `group_role_mappings` JSONB config extension to `tenant_sso_configs`, `highest_privilege` resolution algorithm (tenant_admin > coach > member > viewer), `tenant_owner` blocked from group assignment, unmapped group fallback with `scim.group_unmapped` audit event and opt-in strict-block mode. IdP-specific: Okta (Push Groups manual config required — common onboarding gap); Entra ID (nested group flattening §15.6.2a — skips `type:"Group"` members, logs `scim.group_nested_flattened`, flat security group or P1/P2 member flattening recommended; dynamic groups treated as static); Google Workspace (SCIM Groups not supported — JIT via §11 compensating control; G-014 new gap). Nine new audit events added to DEC-030 HMAC chain taxonomy. GDPR: group `displayName` as potential Art. 9 carrier; Art. 9 group name scanner with TypeScript `ART9_GROUP_NAME_PATTERNS` regex constants mirroring §14.5 `BLOCKED_ATTRIBUTE_PATTERNS`; non-blocking flag-and-review workflow (not reject); `scim.group_name_sensitive_detected` event; group membership excluded from analytics and tenant-manager RLS; Art. 17 hard-delete of `scim_group_members` on erasure. Customer onboarding checklist extended for all three IdPs. Implementation checklist: 15 tasks (10× P0, 4× P1, 1× P2), M4/M5. G-014: 🔴 Google Workspace by-design limitation; JIT compensating control documented; SOC 2 CC6.2 satisfied; re-evaluate M6 Q4 2026.*

*v0.8 additions: Section 16 — Admin Dashboard SSO & SCIM Configuration UI. Product spec and UX design for the IT admin configuration surface in the FORM admin dashboard, required before first enterprise pilot. Closes G-007 (design phase). OIDC setup wizard (5-step: IdP selection → discovery URL / manual fields → attribute mapping → sandboxed test login → go-live toggle); SAML 2.0 setup wizard (6-step: IdP selection → metadata XML upload or manual entry → SP metadata download → test login → attribute mapping → go-live); SP metadata viewer with certificate expiry countdown, dual-cert rotation button, ADFS manual-upload flag. SCIM token management: masked display, one-click rotation with 15-minute overlap window, 90-day age alert. Group-to-role mapping table UI: add/remove/edit mappings, fallback role selector, strict-mode toggle — writes to `tenant_sso_configs.group_role_mappings` JSONB. Azure AD GUID resolver (closes G-008 design): opt-in Microsoft Graph API call (client credentials, per-tenant admin consent) resolves GUIDs to display names; stored in `group_role_mappings[].resolved_display_name`; graceful fallback to GUID display if consent not granted. Session policy panel: timeout, concurrent-session cap, force-SSO toggle, re-auth gate for sensitive actions. IP allowlist UI (closes G-006 design phase): CIDR add/remove, enable toggle, lockout-risk warning; Cloudflare Worker enforcement spec included. SSO sandbox environment (closes G-011 design phase): `{slug}-staging.form.coach` endpoint, linked staging `tenant_id`, one-button "promote config to production" flow, synthetic test users, full isolation from production data. New `tenants` schema columns: `staging_tenant_id UUID REFERENCES tenants(id)`, `is_staging BOOLEAN DEFAULT false`. Graph API schema: `tenant_sso_configs.graph_app_id`, `graph_app_secret_key_ref`, `graph_tenant_id`, `graph_consent_granted_at`, `graph_last_group_sync_at`. Sixteen new audit events added to DEC-030 HMAC chain: `sso_config.wizard_started`, `sso_config.test_login_initiated`, `sso_config.test_login_passed`, `sso_config.test_login_failed`, `sso_config.activated`, `sso_config.deactivated`, `sso_config.deleted`, `sso_cert.rotation_initiated`, `sso_cert.rotation_completed`, `scim_token.generated`, `scim_token.revoked`, `sso_config.ip_allowlist_updated`, `sso_config.session_policy_updated`, `sso_config.group_mapping_updated`, `graph_api.group_resolved`, `sso_staging.config_promoted`. SOC 2 mapping: CC6.1 (SCIM token rotation + age monitoring), CC6.2 (group-role mapping UI enforces principle of least privilege), CC6.3 (IP allowlist access restriction), CC8.1 (all config changes HMAC-chained). Implementation checklist: 15 tasks (8× P0, 5× P1, 2× P2), M4/M5. G-006: 🔴 → 🟡 Partial (UI + Worker spec done; Worker implementation pending). G-007: 🔴 → 🟡 Partial (design done; implementation pending). G-008: 🔴 → 🟡 Partial (Graph API resolver designed; implementation pending). G-011: 🔴 → 🟡 Partial (staging environment designed; infrastructure pending).*

---

## 17. Enterprise Pilot Program: 90-Day Runbook

This section specifies the end-to-end operational procedure for an enterprise pilot — from contract signature through 90-day evaluation to conversion or termination. It integrates the staging environment defined in §16.11, the SCIM provisioning flows of §3 and §15, and the audit event taxonomy of §§8.4 and 16.12.

The 90-day pilot is the standard commercial onboarding path for all enterprise customers. It is defined in `docs/ENTERPRISE.md` and reflected in `pricing-enterprise.html`. This section provides the technical specification that backs the commercial process.

---

### 17.1 Pilot Program Overview

**What a pilot is:**
A 90-day evaluation period conducted entirely on an isolated staging tenant (`{slug}-staging.form.coach`). Real employee data may be used. The staging tenant is fully isolated from production data per §16.11. If the pilot converts, a single "Promote to Production" operation migrates SSO/SCIM config; existing users are re-linked to the new production `tenant_id` via the SCIM migration path (§17.7). If the pilot terminates, data export is offered and the staging tenant is hard-deleted per §17.8.

**Pilot ownership:**

| Role | Responsibility |
|---|---|
| `customer-success` | Primary contact, milestone reviews, conversion proposal, termination handling |
| `enterprise-architect` | Staging tenant provisioning, SSO/SCIM technical setup, conversion protocol |
| `security-engineer` | Audit trail verification, incident handling per INCIDENT_RESPONSE.md §R-04 (SSO down) |
| `compliance-officer` | DPA status verification before pilot start, post-pilot data erasure sign-off |
| Founder | Approves all pilot terminations (CC8.1 evidence) |

**Pilot states (finite state machine):**

```
not_started → sso_setup → active → [converted | terminated]
                                  ↑
                              paused (if SLA breach pending resolution)
```

---

### 17.2 Pre-Pilot Technical Qualification

The following checklist must be fully complete before a staging tenant is provisioned. `customer-success` owns verification; `enterprise-architect` provides the final technical sign-off.

**Customer-side prerequisites:**

| # | Item | Owner | Verification |
|---|---|---|---|
| Q-01 | Compatible IdP confirmed (Okta, Azure AD / Entra ID, Google Workspace, or other SAML 2.0/OIDC) | Customer IT admin | IdP name + version on file |
| Q-02 | SCIM 2.0 support confirmed — customer IdP can send `POST /Users`, `PATCH /Users`, `DELETE /Users` | Customer IT admin | SCIM version documented |
| Q-03 | At minimum 10 pilot users with real work email addresses in the IdP identified (for SSO test) | Customer IT admin | User list (names + emails) on file via secure form |
| Q-04 | Customer IT admin with IdP admin rights identified; available for Day 7–14 technical setup window | Customer + CS | Named contact + calendar hold confirmed |
| Q-05 | Network: no firewall rule that would block outbound requests from `{slug}-staging.form.coach` to IdP metadata endpoints | Customer IT/security | If IP allowlist needed — provided to CS before provisioning |
| Q-06 | DPA (`docs/GDPR_DPIA.md §5`) signed and filed | compliance-officer | DPA receipt in `compliance/dpas/{slug}-pilot.pdf` |
| Q-07 | Enterprise pilot contract executed (SOW + Master Services Agreement or equivalent) | Founder | Contract signed copy on file |
| Q-08 | Billing contact and billing instrument on file (post-pilot invoicing) | Customer finance | |
| Q-09 | Data region preference confirmed: EU (default) or US | Customer + CS | Documented in `pilot_programs.data_region` |

**FORM-side prerequisites (platform-engineer signs off):**

| # | Item | Status |
|---|---|---|
| F-01 | §16.11 staging tenant infrastructure deployed to target data region | ✓ if deployed; block pilot if not |
| F-02 | SCIM provisioning endpoint (`/scim/v2`) live and passing integration tests | ✓ required |
| F-03 | DEC-030 HMAC audit chain writer deployed and running | ✓ required |
| F-04 | INCIDENT_RESPONSE.md on-call rotation active (security-engineer reachable within 15 min for P0) | ✓ required before go-live (Day 14+) |
| F-05 | `pilot_programs` and `pilot_milestone_events` tables migrated (§17.9) | ✓ required |

---

### 17.3 Pilot Tenant Provisioning

Performed by `enterprise-architect` after all Q-01 through Q-09 prerequisites are verified.

**Step 1: Provision production tenant record (placeholder)**

```sql
-- Production tenant: slug reserved but is_active = false until conversion
INSERT INTO tenants (id, slug, display_name, plan, seat_count, data_region, is_active, is_staging)
VALUES (
  gen_random_uuid(),         -- production_tenant_id (reserved)
  'acme-corp',               -- DNS-safe slug
  'Acme Corp',
  'enterprise',              -- plan: starter | growth | enterprise
  500,                       -- contracted seat count
  'eu',                      -- data_region: 'eu' | 'us'
  false,                     -- not yet active — activated at conversion (§17.7)
  false
);
```

**Step 2: Provision staging tenant record**

```sql
-- Staging tenant: is_staging = true; linked to production tenant via staging_tenant_id
INSERT INTO tenants (id, slug, display_name, plan, seat_count, data_region, is_active, is_staging)
VALUES (
  gen_random_uuid(),
  'acme-corp-staging',        -- slug for {slug}-staging.form.coach
  'Acme Corp (Pilot)',
  'enterprise',
  500,
  'eu',
  true,                       -- pilot is active from Day 0 on staging tenant
  true
);

-- Link staging to production
UPDATE tenants SET staging_tenant_id = <staging_tenant_id> WHERE id = <production_tenant_id>;
```

**Step 3: Create pilot_programs record**

```sql
INSERT INTO pilot_programs (
  staging_tenant_id, production_tenant_id, status, plan,
  seat_count_contracted, contract_value_usd,
  customer_success_owner, data_region
) VALUES (
  <staging_tenant_id>, <production_tenant_id>, 'sso_setup', 'enterprise',
  500, 36000.00,
  'cs@form.coach', 'eu'
);
```

**Step 4: Configure admin dashboard access**

Create a `tenant_owner` user record for the primary customer IT admin on the staging tenant. This user will complete the OIDC/SAML wizard in §16.3/§16.4.

Share with customer IT admin:
- Admin dashboard URL: `https://{slug}-staging.form.coach/admin`
- Temporary login token (magic-link, valid 24 hours)
- SSO Configuration Guide PDF (generated from §7.2 / §7.3 for their specific IdP)

Emit `pilot.tenant_provisioned` audit event (§17.10).

---

### 17.4 Pilot Timeline

| Day | Milestone | Owner | Pass Criteria | Action if Blocked |
|---|---|---|---|---|
| **0** | Contract signed; staging tenant provisioned | enterprise-architect | `pilot_programs.status = 'sso_setup'`; `pilot.tenant_provisioned` event in audit log | Escalate to founder if provisioning fails > 4 hours |
| **1–7** | FORM team completes staging environment setup: SCIM endpoint live, admin dashboard accessible, synthetic test users in place per §16.11 | platform-engineer + enterprise-architect | Test login with synthetic user passes | Extend window by 3 days max; notify customer |
| **7–14** | Customer IT admin configures SSO in their IdP using OIDC/SAML wizard (§16.3 / §16.4); runs sandboxed test login | Customer IT admin + CS | `sso_config.test_login_passed` event in audit log for ≥ 1 real employee test account | CS joins IT admin via screen share; max 2 calendar blocks before escalation to enterprise-architect |
| **14** | SSO go-live: `sso_config.activated` event; real users begin logging in | enterprise-architect | `sso_config.activated` in audit log; `pilot_programs.status = 'active'`; `pilot.started_at` set | Block go-live if test login not passed |
| **14–21** | SCIM provisioning configured: IT admin generates SCIM token in §16.6, configures in IdP; first real SCIM sync completes | Customer IT admin | `scim.user_provisioned` events appearing in audit log at IdP-sync cadence | CS support call; check §3 SCIM endpoint availability |
| **21** | SCIM first-sync validation: CS verifies user count matches IdP group; SCIM error rate < 1% | CS + enterprise-architect | `pilot_milestone_events` record with `event_type = 'scim_first_sync'`, `passed = true` | Investigate per §8.2 debugging runbook |
| **30** | **Day 30 Milestone Review** — CS meets with customer sponsor; reviews success metrics §17.5 | CS | See §17.5 Day-30 thresholds | If any P0 metric fails → pilot `paused` state; escalation to founder |
| **31–60** | Business-as-usual; FORM monitors per §17.6; customer trains employees; optional: coach accounts for HR/wellness team | CS monitors | Weekly health report sent; alert thresholds per §17.6 | Any P0 incident → INCIDENT_RESPONSE.md R-04 |
| **60** | **Day 60 Milestone Review** — conversion readiness assessment | CS | See §17.5 Day-60 thresholds | If score < threshold → plan remediation sprint in remaining 30 days |
| **85** | Conversion proposal sent by CS: pricing confirmation, MSA amendment, production tenant ID, billing start date | CS | Written proposal sent; response deadline Day 90 | If no response → follow-up at Day 88 |
| **90** | **Day 90 Decision** — customer converts or terminates | Founder (approves) | `pilot.converted` or `pilot.terminated` audit event | No extensions beyond Day 90 without founder approval |

---

### 17.5 Pilot Success Metrics

These metrics are computed from the staging tenant's audit log and operational data. They are reviewed at Day 30, Day 60, and Day 90.

**Metric definitions:**

| Metric | Definition | Day-30 Target | Day-60 Target | Day-90 (Conversion Gate) |
|---|---|---|---|---|
| **SSO Adoption Rate** | Distinct users with ≥1 `auth.sso_login` event in prior 7 days ÷ SCIM-provisioned active users | ≥ 50% | ≥ 70% | ≥ 80% |
| **SCIM Sync Error Rate** | Failed SCIM operations (4xx from FORM, not IdP-side 5xx) ÷ total SCIM operations in prior 7 days | < 2% | < 1% | < 0.5% |
| **P0 Incident Count** | Count of `P0` incidents in INCIDENT_RESPONSE.md taxonomy affecting this staging tenant during pilot | 0 (mandatory) | 0 (mandatory) | 0 (mandatory) |
| **Admin Dashboard Satisfaction** | Structured 5-question survey sent to IT admin at Day 30; score on 1–10 scale | ≥ 7/10 | ≥ 8/10 | (informational) |
| **Pilot NPS** | Net Promoter Score from 3-question survey to all active pilot users | (informational) | ≥ +20 | ≥ +30 |
| **SLA Compliance** | No breach of §12.2 SLA tier for the contracted plan | 100% | 100% | 100% |

**P0 metric failure protocol:**
If P0 Incident Count ≥ 1 during the pilot, the pilot automatically enters `paused` state. The incident must be fully remediated per INCIDENT_RESPONSE.md, the post-incident review filed, and the customer sponsor briefed. Resuming active state requires explicit founder approval and customer sign-off. The 90-day clock does **not** pause during P0 incidents — this is by design to ensure the pilot reflects realistic operating conditions.

**Conversion gate logic:**

```
CONVERT if:
  SSO Adoption Rate (Day-90 window) ≥ 80%
  AND SCIM Sync Error Rate (Day-90 window) < 0.5%
  AND P0 Incident Count (full pilot) = 0
  AND customer has returned signed contract amendment

ESCALATE TO FOUNDER if any metric misses — founder decides whether to convert
  with remediation commitments, extend (max 30 days, once), or terminate.
```

---

### 17.6 Mid-Pilot Health Monitoring

**Weekly health report (automated, sent every Monday 08:00 customer timezone):**

Cloudflare Worker (`pilot-health-worker`) queries the staging tenant's audit log and constructs a brief health digest sent to the CS owner and the customer's primary IT admin contact:

```
Subject: FORM Pilot Health — Acme Corp — Week 3 of 13

SSO Adoption Rate (7d):  62%  [target ≥50%]  ✓
SCIM Sync Error Rate (7d): 0.3% [target <2%]  ✓
Active Users (7d):       312  of 500 provisioned
P0 Incidents (pilot):    0    ✓
Days Remaining:          69
```

**Alert thresholds that page customer-success:**

| Condition | Alert Level | Action |
|---|---|---|
| SCIM error rate > 5% for > 2 consecutive hours | P1 | CS notified; check §8.2 |
| SSO adoption rate drops > 20% week-over-week | P2 | CS notified; schedule check-in call |
| `sso_config.deactivated` event on staging tenant | P0 | CS + enterprise-architect paged immediately |
| Certificate expiry < 30 days (§16.5 expiry monitor) | P1 | CS notified; schedule rotation |
| Pilot deadline < 14 days and no conversion proposal sent | P2 | CS notified; trigger conversion proposal workflow |

**OBSERVABILITY.md integration:**
Pilot-specific metrics are included in the enterprise tenant OBSERVABILITY.md §18 dashboard. A dedicated "Pilot Programs" panel shows all active pilots with current adoption rates, milestone status, and days remaining.

---

### 17.7 Pilot-to-Production Conversion Protocol

Triggered when the customer returns the signed contract amendment. Performed by `enterprise-architect`.

**Legal prerequisites (compliance-officer verifies):**
- Contract amendment signed and filed
- DPA updated if seat count or data categories changed from pilot scope
- Billing activation confirmed with finance

**Technical steps:**

**Step 1: Activate production tenant**
```sql
UPDATE tenants SET is_active = true, updated_at = now()
WHERE id = <production_tenant_id>;
```

**Step 2: "Promote to Production" — SSO/SCIM config migration**

Use the admin dashboard §16.11 "Promote to Production" button. This copies:
- `tenant_sso_configs` row from staging → production tenant_id
- `scim_group_role_mappings` JSONB from staging config
- `group_role_mappings` JSONB from staging config
- Certificate and key material (staging cert is valid until rotation)

Generates `sso_staging.config_promoted` audit event (§16.12) on both staging and production tenants.

**Step 3: SCIM user migration**

Option A (preferred for Okta and Azure): **Re-provision** — IT admin updates the SCIM base URL in IdP from `https://{slug}-staging.form.coach/scim/v2` to `https://{slug}.form.coach/scim/v2`; SCIM generates `DELETE /Users` for staging users and `POST /Users` for production users. Clean state on production. Staging user data is preserved during §17.8 30-day export window.

Option B (for Google Workspace JIT only): **User mapping migration** — run migration script that updates `user_sessions.tenant_id` from staging → production `tenant_id` for all users whose `email` domain matches the tenant's `oidc_hd_restriction`. Requires security-engineer sign-off on the migration SQL before execution.

```sql
-- Option B migration: reassign users from staging to production tenant
-- REQUIRES: two-person review before execution (enterprise-architect + security-engineer)
-- REQUIRES: pilot_programs.converted_at IS NULL before running (idempotency guard)
UPDATE users
SET tenant_id = <production_tenant_id>, updated_at = now()
WHERE tenant_id = <staging_tenant_id>
  AND email LIKE ANY(ARRAY['%@acme.com', '%@acme.co.uk']);

-- Verify count before committing:
-- SELECT count(*) FROM users WHERE tenant_id = <staging_tenant_id>;
-- Expected: 0 after migration
```

**Step 4: SCIM token regeneration**

IT admin generates a new SCIM token for the production tenant via §16.6 (the staging SCIM token is revoked). Generates `scim_token.generated` and `scim_token.revoked` audit events.

**Step 5: Validate production SSO login**

Run sandboxed test login on production tenant (§7.4 procedure). Confirm `sso_config.test_login_passed` event in production audit log before notifying customer that production is live.

**Step 6: Update pilot_programs record**

```sql
UPDATE pilot_programs
SET
  status = 'converted',
  converted_at = now(),
  updated_at = now()
WHERE staging_tenant_id = <staging_tenant_id>;

INSERT INTO pilot_milestone_events (pilot_id, event_type, passed, metric_snapshot, recorded_by)
VALUES (
  <pilot_id>, 'conversion', true,
  '{"sso_adoption_rate": 0.87, "scim_error_rate": 0.002, "p0_count": 0}'::jsonb,
  'enterprise-architect@form.coach'
);
```

Emit `pilot.converted` audit event (§17.10).

**Step 7: Staging tenant cleanup schedule**

Do NOT immediately delete the staging tenant. Set a 30-day export window per §17.8.

---

### 17.8 Pilot Termination and Data Handling

Applies when a pilot reaches Day 90 without converting, or when the customer chooses to terminate early.

**Authorization required:** Founder approval (email + `pilot.terminated` audit event) before any termination action.

**Data export window (all terminations):**

Within 24 hours of `pilot.terminated` event, `customer-success` sends the customer a DSAR-format export link:
- `form-pilot-export-{slug}-{date}.zip` containing all user data in the staging tenant, formatted per §12.5 DSAR export specification
- Export link valid for 30 days (hosted on R2, pre-signed URL)
- Notification email to all provisioned users with `data_export_available` notification (§18 notification system)

**Hard deletion (Day 30 after termination):**

```sql
-- Precondition: compliance-officer has confirmed 30-day export window has passed
-- Precondition: no active DSAR requests pending for this tenant

-- 1. Revoke all active sessions for staging tenant users
UPDATE user_sessions
SET revoked_at = now(), revoke_reason = 'pilot_termination'
WHERE tenant_id = <staging_tenant_id> AND revoked_at IS NULL;

-- 2. Run Art. 17 erasure worker for all users in staging tenant
-- (handled by existing erasure worker per DATA_MODEL.md §12.3 — invoked per user)

-- 3. Deactivate staging tenant
UPDATE tenants SET is_active = false, updated_at = now()
WHERE id = <staging_tenant_id>;

-- 4. Deactivate production tenant placeholder (was never activated)
UPDATE tenants SET is_active = false, updated_at = now()
WHERE id = <production_tenant_id>;
```

Emit `pilot.data_deleted` audit event. File in `compliance/evidence/pilot-terminations/{slug}/`.

**GDPR obligations on termination:**
- Art. 17 erasure: user personal data hard-deleted within 30 days of termination (or sooner on individual request)
- Art. 20 portability: export offered as described above
- DPA termination notice sent to customer (`compliance-officer` prepares; founder signs)
- Sub-processor Anthropic notified if Victor coaching turns were generated during pilot (Anthropic data processing notification per INCIDENT_RESPONSE.md §11)

---

### 17.9 Database Schema

#### Table: `pilot_programs`

```sql
CREATE TABLE pilot_programs (
  id                     UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  staging_tenant_id      UUID NOT NULL REFERENCES tenants(id),
  production_tenant_id   UUID NOT NULL REFERENCES tenants(id),
  status                 TEXT NOT NULL DEFAULT 'not_started'
    CHECK (status IN ('not_started', 'sso_setup', 'active', 'paused', 'converted', 'terminated')),
  plan                   TEXT NOT NULL CHECK (plan IN ('starter', 'growth', 'enterprise')),
  seat_count_contracted  INT NOT NULL CHECK (seat_count_contracted >= 10),
  contract_value_usd     NUMERIC(12,2) NOT NULL CHECK (contract_value_usd > 0),
  data_region            TEXT NOT NULL DEFAULT 'eu' CHECK (data_region IN ('eu', 'us')),
  pilot_started_at       TIMESTAMPTZ,
  pilot_deadline_at      TIMESTAMPTZ GENERATED ALWAYS AS (pilot_started_at + INTERVAL '90 days') STORED,
  converted_at           TIMESTAMPTZ,
  terminated_at          TIMESTAMPTZ,
  termination_reason     TEXT CHECK (termination_reason IN (
    'customer_choice', 'sla_breach', 'non_payment', 'security_finding', 'no_response'
  )),
  day30_review_at        TIMESTAMPTZ,
  day30_review_passed    BOOLEAN,
  day60_review_at        TIMESTAMPTZ,
  day60_review_passed    BOOLEAN,
  customer_success_owner TEXT NOT NULL,
  notes                  TEXT,
  created_at             TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at             TIMESTAMPTZ NOT NULL DEFAULT now(),

  CONSTRAINT pilot_dates_valid CHECK (
    (pilot_started_at IS NULL) OR
    (converted_at IS NULL OR converted_at >= pilot_started_at) AND
    (terminated_at IS NULL OR terminated_at >= pilot_started_at)
  ),
  CONSTRAINT pilot_terminal_exclusive CHECK (
    NOT (converted_at IS NOT NULL AND terminated_at IS NOT NULL)
  )
);

CREATE UNIQUE INDEX pilot_programs_staging_uniq ON pilot_programs (staging_tenant_id);
CREATE INDEX pilot_programs_status_idx ON pilot_programs (status) WHERE status IN ('sso_setup', 'active', 'paused');
CREATE INDEX pilot_programs_deadline_idx ON pilot_programs (pilot_deadline_at) WHERE status = 'active';
```

**RLS policies:**

```sql
ALTER TABLE pilot_programs ENABLE ROW LEVEL SECURITY;

-- form_admin: full access (internal ops)
CREATE POLICY pilot_programs_form_admin ON pilot_programs
  FOR ALL TO form_admin USING (true) WITH CHECK (true);

-- form_system: full access (worker reads)
CREATE POLICY pilot_programs_form_system ON pilot_programs
  FOR SELECT TO form_system USING (true);

-- No tenant_member access — pilot operational data is internal-only
```

#### Table: `pilot_milestone_events`

```sql
CREATE TABLE pilot_milestone_events (
  id               UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  pilot_id         UUID NOT NULL REFERENCES pilot_programs(id) ON DELETE RESTRICT,
  event_type       TEXT NOT NULL CHECK (event_type IN (
    'tenant_provisioned',
    'sso_go_live',
    'scim_first_sync',
    'day30_review',
    'day60_review',
    'day90_decision',
    'conversion',
    'termination',
    'sla_breach',
    'paused',
    'resumed'
  )),
  occurred_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
  passed           BOOLEAN,
  metric_snapshot  JSONB,  -- JSON snapshot of §17.5 metrics at milestone time
  notes            TEXT,
  recorded_by      TEXT NOT NULL  -- form_admin email; not nullable for audit accountability
);

CREATE INDEX pilot_milestones_pilot_id_idx ON pilot_milestone_events (pilot_id, occurred_at DESC);

ALTER TABLE pilot_milestone_events ENABLE ROW LEVEL SECURITY;

CREATE POLICY pilot_milestones_form_admin ON pilot_milestone_events
  FOR ALL TO form_admin USING (true) WITH CHECK (true);

CREATE POLICY pilot_milestones_form_system ON pilot_milestone_events
  FOR SELECT TO form_system USING (true);
```

**`metric_snapshot` schema (informational — validated at application layer by Zod):**

```typescript
interface PilotMetricSnapshot {
  sso_adoption_rate:   number;   // 0.0–1.0
  scim_error_rate:     number;   // 0.0–1.0
  p0_incident_count:   number;   // integer
  active_users_7d:     number;   // integer
  provisioned_users:   number;   // integer
  admin_sat_score:     number | null;   // 1–10 or null if not yet surveyed
  pilot_nps:           number | null;   // -100 to +100 or null
}
```

---

### 17.10 Audit Events

All events use HMAC-SHA256 chaining per DEC-030. Events are written on the `staging_tenant_id` tenant's audit chain unless noted otherwise.

| Event type | Trigger | Key metadata fields | Written on tenant |
|---|---|---|---|
| `pilot.tenant_provisioned` | Staging + production tenant records created (§17.3) | `staging_tenant_id`, `production_tenant_id`, `plan`, `seat_count_contracted`, `contract_value_usd`, `data_region` | Both tenants |
| `pilot.sso_setup_started` | IT admin first accesses admin dashboard on staging tenant | `pilot_id`, `idp_type` (okta\|entra\|google\|other), `protocol` (oidc\|saml) | Staging tenant |
| `pilot.started` | `sso_config.activated` on staging tenant + `pilot_programs.status` set to `'active'` | `pilot_id`, `pilot_started_at`, `pilot_deadline_at` | Staging tenant |
| `pilot.milestone_review` | Day 30 or Day 60 review completed | `pilot_id`, `milestone` (day30\|day60), `passed`, `metric_snapshot` (JSON) | Staging tenant |
| `pilot.paused` | P0 incident triggered pause | `pilot_id`, `reason`, `incident_id` (from INCIDENT_RESPONSE.md) | Staging tenant |
| `pilot.resumed` | Founder approves resumption after pause | `pilot_id`, `approved_by` (founder email), `notes` | Staging tenant |
| `pilot.converted` | Contract amendment signed; production tenant activated | `pilot_id`, `production_tenant_id`, `converted_at`, `final_metrics` (JSON snapshot) | Both tenants |
| `pilot.terminated` | Pilot not converted at Day 90 or early termination | `pilot_id`, `termination_reason`, `authorized_by` (founder email) | Both tenants |
| `pilot.data_deleted` | 30-day export window elapsed; staging tenant hard-deleted per §17.8 | `pilot_id`, `staging_tenant_id`, `users_deleted_count`, `audit_events_retained` | Production tenant (staging deleted) |

All nine events are added to the DEC-030 HMAC chain taxonomy (append-only; requires DECISION_LOG entry per DATA_MODEL.md §18.2 note on adding enum values).

---

### 17.11 SOC 2 Evidence Mapping

| SOC 2 Criterion | How §17 Satisfies It |
|---|---|
| **CC3.2 — Risk assessment** | The Pre-Pilot Technical Qualification checklist (§17.2) is a documented risk assessment performed before any customer data enters the FORM platform. Q-06 (DPA) and Q-07 (contract) ensure legal risk is addressed. Q-04 (F-04 on-call) ensures incident response capacity is in place before real users are admitted. The qualification sign-off is `pilot.tenant_provisioned` — an HMAC-chained event that creates an auditable record of when qualification was completed. |
| **CC6.2 — Access controls prior to issuing credentials** | No user credentials (SCIM-provisioned accounts or SSO sessions) are issued to real employees until after `sso_config.test_login_passed` event (Day 14 gate). The `pilot.started` event marks the first moment real-employee SSO sessions are permitted. Auditors can verify that access issuance followed the documented procedure by checking the event ordering in the audit chain. |
| **CC7.2 — System monitoring** | The weekly health report (§17.6) and the `pilot-health-worker` monitoring constitute documented, automated system monitoring for the pilot tenant. Alert thresholds and escalation paths are specified. Panel integration with OBSERVABILITY.md §18 dashboard provides auditor-accessible monitoring evidence. |
| **CC8.1 — Change management** | The conversion protocol (§17.7) is a documented change management procedure for the production activation event — arguably the highest-impact configuration change in the enterprise lifecycle. The "Promote to Production" button generates `sso_staging.config_promoted` (§16.12); the migration SQL requires two-person review; `pilot.converted` event records founder awareness. Termination requires founder approval (`pilot.terminated` metadata: `authorized_by`). Both change paths are auditable via the HMAC chain. |
| **CC9.2 — Vendor/customer management** | The termination protocol (§17.8) documents FORM's obligations to customers at relationship end: data export within 24h, DSAR-format portability, Art. 17 erasure within 30 days, DPA termination notice. `pilot.data_deleted` event provides evidence that obligations were completed. This satisfies the "vendor/customer lifecycle management" component of CC9.2. |
| **A1.1 — Availability commitments** | SLA Compliance is a named success metric (§17.5) with a 100% target. Any SLA breach triggers `pilot.paused` and INCIDENT_RESPONSE.md escalation. The audit trail of `pilot.paused` / `pilot.resumed` events, combined with the INCIDENT_RESPONSE.md post-incident review, constitutes the evidence record for availability commitment management during the pilot. |

**Auditor evidence artefacts:**
- `audit_log` events of type `pilot.*` for the pilot period, queryable by `staging_tenant_id` or `production_tenant_id`
- `pilot_milestone_events` table export: rows for all milestones including `metric_snapshot` JSON
- Pre-Pilot Qualification checklist (Q-01 through Q-09): filed in `compliance/evidence/pilots/{slug}/qualification.md`
- DPA receipt: `compliance/dpas/{slug}-pilot.pdf`
- Conversion SQL review artifact (Option B only): two-person sign-off logged in `compliance/evidence/pilots/{slug}/conversion-sql-review.md`

---

### 17.12 Implementation Checklist

| Task | Owner | Priority | Milestone |
|---|---|---|---|
| Write `migrations/YYYYMMDD_pilot_programs.sql` — `pilot_programs` table DDL, constraints, indexes, RLS policies | platform-engineer | **P0** | M4 |
| Write `migrations/YYYYMMDD_pilot_milestone_events.sql` — `pilot_milestone_events` table DDL, indexes, RLS policies | platform-engineer | **P0** | M4 |
| Implement `pilot-health-worker` Cloudflare Worker: cron Monday 06:00 UTC; compute §17.5 metrics from audit log for all `active` pilot tenants; insert `metric_snapshot` JSONB into `pilot_milestone_events`; send weekly health digest via SES to `customer_success_owner` + IT admin contact | platform-engineer + devops-lead | **P0** | M4 |
| Implement alert thresholds in `pilot-health-worker`: SCIM error rate > 5% for 2h → page CS (PagerDuty); SSO adoption drop > 20% WoW → Slack `#cs-alerts`; `sso_config.deactivated` on pilot tenant → P0 page | devops-lead | **P0** | M4 |
| Build "Pilot Programs" panel in Metabase admin dashboard: active pilots list, adoption rates, milestone status, days remaining, P0 count | data-engineer | **P1** | M4 |
| Wire all 9 pilot audit events (§17.10) to DEC-030 HMAC chain writer; validate chain integrity in staging | platform-engineer + security-engineer | **P0** | M4 |
| Write `compliance/evidence/pilots/TEMPLATE-qualification.md` — pre-pilot qualification checklist template (Q-01 through Q-09) for CS to fill out per customer | compliance-officer | **P1** | M4 |
| Add `pilot_programs.pilot_deadline_at` alert: 14 days before deadline + no conversion proposal → Slack `#cs-team` (Cloudflare Worker cron, daily 08:00 UTC) | devops-lead | **P1** | M4 |
| Implement DSAR-format export for staging tenant termination (§17.8): extend existing DSAR export job to accept `tenant_id` scope; generate `form-pilot-export-{slug}-{date}.zip`; upload to R2; generate 30-day pre-signed URL | platform-engineer | **P1** | M4 |
| Implement two-person SQL review protocol for Option B user migration (§17.7): GitHub PR template with required reviewers `enterprise-architect` + `security-engineer`; merge gate requires both approvals | enterprise-architect + security-engineer | **P1** | M5 |
| Add `pilot_programs.status` lifecycle enforcement at API layer: state machine validation (not_started → sso_setup → active; no skipping states); unauthorized state transitions return `422 Unprocessable Entity` | platform-engineer | **P0** | M4 |
| QA: end-to-end pilot lifecycle test against Okta dev tenant — provisioning → SSO go-live → Day 30 metric computation → conversion promotion → staging cleanup | qa-lead + enterprise-architect | **P0** | M5 |
| Add `pilot_programs` and `pilot_milestone_events` tables to Art. 17 erasure scope: confirm `pilot.data_deleted` event fires after staging tenant deletion | platform-engineer + compliance-officer | **P1** | M4 |

**Gap status updates:**
**G-011 status update:** 🟡 Partial → 🟡 Partial. §17 adds the operational runbook layer on top of §16.11's infrastructure design. Infrastructure provisioning (platform-engineer M4) remains the blocking item for G-011 to close to 🟢.

---

*v0.9 additions: Section 17 — Enterprise Pilot Program: 90-Day Runbook. Operational protocol for the full enterprise pilot lifecycle from contract signature through 90-day evaluation to conversion or termination, complementing the Admin Dashboard UI spec (§16) and Staging Environment design (§16.11). Pre-Pilot Technical Qualification: 9-item customer checklist (Q-01 compatible IdP, Q-02 SCIM 2.0, Q-03 pilot users, Q-04 IT admin availability, Q-05 network access, Q-06 DPA, Q-07 contract, Q-08 billing, Q-09 data region) and 5-item FORM platform prerequisite check. Pilot tenant provisioning: SQL walkthrough for staging + production tenant records, `pilot_programs` insert, and IT admin dashboard access setup. 90-day milestone timeline: 12 milestones from Day 0 provisioning through Day 90 conversion/termination decision with pass criteria and escalation paths for each. Pilot success metrics: 6 named metrics (SSO adoption rate, SCIM sync error rate, P0 incident count, admin dashboard satisfaction, Pilot NPS, SLA compliance) with Day-30/Day-60/Day-60/Day-90 thresholds and binary conversion gate logic. Mid-pilot health monitoring: weekly health digest Worker (Monday 06:00 UTC), 5 alert thresholds, OBSERVABILITY.md §18 integration. Pilot-to-production conversion: 7-step protocol including production tenant activation, "Promote to Production" button (§16.11), SCIM migration options (Option A re-provision vs Option B user-ID mapping with two-person SQL review gate), SCIM token regeneration, and staging cleanup schedule. Termination and data handling: 24h DSAR-format export link, 30-day export window, Art. 17 erasure sequence, DPA termination, sub-processor notification. Database: `pilot_programs` table (state machine with generated `pilot_deadline_at` column, terminal state exclusivity constraint, RLS: form_admin full / form_system read / no tenant_member access) and `pilot_milestone_events` table (typed event_type enum, `metric_snapshot` JSONB with TypeScript interface definition). Nine new DEC-030 HMAC-chained audit events: `pilot.tenant_provisioned`, `pilot.sso_setup_started`, `pilot.started`, `pilot.milestone_review`, `pilot.paused`, `pilot.resumed`, `pilot.converted`, `pilot.terminated`, `pilot.data_deleted`. SOC 2 mapping: CC3.2 (risk assessment at qualification), CC6.2 (access issuance gate at Day 14), CC7.2 (weekly health monitoring), CC8.1 (conversion + termination change management with founder approval), CC9.2 (customer lifecycle data obligations), A1.1 (SLA compliance as named metric). Implementation checklist: 13 tasks (7× P0, 5× P1, 1× P0 QA), M4/M5. G-011: remains 🟡 Partial — §17 adds the operational runbook layer; infrastructure provisioning is the remaining blocker.*

---

## 18. Tenant IdP Migration Runbook

### 18.1 Purpose and Scope

This section specifies the end-to-end operational procedure for migrating an active enterprise tenant from one identity provider to another — whether that means switching IdP vendors (Okta → Microsoft Entra ID), upgrading protocols (SAML → OIDC), migrating a SCIM endpoint to a new directory, or any combination of the above.

IdP migrations are **high-risk configuration changes**. Every active employee's ability to log in to FORM depends on `tenant_sso_configs` being correct. A misconfigured migration leaves users locked out; a botched `external_id` remap can fragment user history or grant the wrong role to a re-provisioned user. This runbook enforces a staging-first, founder-authorised, HMAC-audited procedure with a defined rollback path at every step.

**Scope:** Tenants in `'active'` or `'pilot'` status with `sso_enabled = true`. Protocol upgrades (SAML → OIDC) and SCIM-only migrations are handled as sub-types. Consumer tier is out of scope.

**Prerequisites:** §16.11 staging environment must be provisioned for the tenant before this runbook can begin. If staging does not exist, provision it first using the §16.11 procedure.

---

### 18.2 Migration Trigger Scenarios

| Scenario | Protocol change? | SCIM impact | Estimated effort | Owner |
|---|---|---|---|---|
| **M&A — new parent company IdP** (e.g., Okta → Entra ID) | Sometimes | High — new externalIds for all users | 3–5 days engineering + 1–2 weeks customer IT coordination | enterprise-architect + platform-engineer |
| **IdP vendor replacement** (e.g., Okta → Ping Identity) | Rarely | High if SCIM endpoint changes | 2–4 days engineering | platform-engineer |
| **Protocol upgrade** (SAML → OIDC on same IdP) | Yes | Low if IdP sub = prior NameID | 1–2 days engineering | platform-engineer |
| **SCIM endpoint migration** (same IdP, new SCIM app) | No | Medium — new token, possible new externalIds | 1 day engineering | platform-engineer |
| **Custom domain migration** (IT restructure, sub-domain change) | No | None | Hours — redirect rules only | devops-lead |
| **On-premises ADFS → cloud IdP** | Usually (SAML → OIDC) | High — ADFS objectGUID differs from cloud UPN | 5–7 days | enterprise-architect + platform-engineer |

**Customer IT coordination is always the critical path.** Engineering can complete its side in hours; customer IT configuring the new IdP application, testing SCIM with new credentials, and running user acceptance testing typically takes 1–4 weeks.

---

### 18.3 Pre-Migration Assessment Checklist

Before opening a migration record, the CS owner and enterprise-architect must complete this assessment jointly with the customer IT admin.

**FORM side (CS + enterprise-architect):**

| # | Check | Pass criteria |
|---|---|---|
| F-01 | Staging environment exists for tenant | `tenant_sso_configs` row with `is_staging = true` present; staging URL accessible |
| F-02 | Current SSO config is stable | No `sso.login_failed` spikes in last 7 days; `is_active = true`; no open P0/P1 incidents |
| F-03 | SCIM state is clean | `scim_provisioning_log` shows no `error` rows in last 24h; `users.provisioned_via = 'scim'` count matches IdP active users ± 2% |
| F-04 | User count documented | Record `active_user_count` at migration start; used as rollback success threshold (§18.9) |
| F-05 | DPA covers new IdP's data region | If new IdP routes SCIM through a different region (e.g., Entra GCC High, US-only), verify existing DPA data-region clause covers it; re-amend if not |
| F-06 | Founder authorisation obtained | Migration record in `sso_idp_migrations` with `authorized_by` = founder email required before any production change |

**Customer IT side (CS to confirm with customer):**

| # | Check | Pass criteria |
|---|---|---|
| C-01 | New IdP application configured in staging | SP metadata imported; test user assigned; test login successful in staging within 48h of migration start |
| C-02 | SCIM provisioning app created | New SCIM bearer token generated; test provisioning event sent and received in staging |
| C-03 | Group mappings documented | Customer IT provides `{new_group_id: form_role}` mapping for all groups; compared against current `scim_group_role_mappings` for role-equivalence |
| C-04 | Email domain unchanged | Users' email addresses remain the same in new IdP (required for email-match strategy §18.6.2). If emails change, use manual ID mapping strategy. |
| C-05 | Rollback window agreed | Customer IT agrees to maintain old IdP application in active state for 48 hours post-cutover; cannot be shortened |
| C-06 | Maintenance window scheduled | Migration cutover window agreed: 60 minutes, outside peak usage hours, with IT admin on call |

---

### 18.4 Migration Database Tables

Two new tables track the migration lifecycle and resolve identity continuity.

```sql
CREATE TABLE sso_idp_migrations (
  id                    UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id             UUID NOT NULL REFERENCES tenants(id),
  status                TEXT NOT NULL DEFAULT 'planning'
                          CHECK (status IN (
                            'planning',        -- assessment in progress
                            'staging',         -- new IdP tested in staging
                            'cutover_pending', -- staging validated; awaiting maintenance window
                            'committed',       -- production cutover complete
                            'rolled_back'      -- cutover reverted; old config restored
                          )),
  from_idp_type         TEXT NOT NULL,
  to_idp_type           TEXT NOT NULL,
  from_protocol         TEXT NOT NULL CHECK (from_protocol IN ('saml', 'oidc')),
  to_protocol           TEXT NOT NULL CHECK (to_protocol IN ('saml', 'oidc')),
  protocol_change       BOOLEAN GENERATED ALWAYS AS (from_protocol <> to_protocol) STORED,
  scim_migration        BOOLEAN NOT NULL DEFAULT FALSE,   -- true if SCIM endpoint also changes
  active_user_count     INTEGER NOT NULL,                 -- snapshot at assessment (F-04)
  staging_validated_at  TIMESTAMPTZ,
  cutover_at            TIMESTAMPTZ,
  rolled_back_at        TIMESTAMPTZ,
  authorized_by         TEXT NOT NULL,  -- founder email; required; enforced in API layer
  cs_owner              TEXT NOT NULL,
  notes                 TEXT,
  created_at            TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at            TIMESTAMPTZ NOT NULL DEFAULT now(),
  -- Only one migration can be in a non-terminal state per tenant at a time
  CONSTRAINT one_active_migration_per_tenant EXCLUDE USING btree (
    tenant_id WITH =
  ) WHERE (status NOT IN ('committed', 'rolled_back'))
);

-- RLS: form_admin full access (BYPASSRLS); form_system read-only; no tenant-facing access
ALTER TABLE sso_idp_migrations ENABLE ROW LEVEL SECURITY;
CREATE POLICY sso_idp_migrations_form_system ON sso_idp_migrations
  FOR SELECT TO form_system USING (true);
-- form_api: no policy = zero rows (fail-closed)

CREATE INDEX idx_sso_migrations_tenant ON sso_idp_migrations (tenant_id, created_at DESC);
CREATE INDEX idx_sso_migrations_status ON sso_idp_migrations (status)
  WHERE status NOT IN ('committed', 'rolled_back');
```

```sql
CREATE TABLE sso_external_id_mappings (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id       UUID NOT NULL REFERENCES tenants(id),
  migration_id    UUID NOT NULL REFERENCES sso_idp_migrations(id) ON DELETE CASCADE,
  user_id         UUID NOT NULL REFERENCES users(id),
  old_external_id TEXT NOT NULL,     -- users.external_id before migration
  new_external_id TEXT,              -- populated during staging validation; NULL = unmatched
  match_method    TEXT NOT NULL
                    CHECK (match_method IN (
                      'email',    -- matched via users.email = SCIM userName
                      'scim',     -- matched via SCIM externalId sent by new IdP
                      'manual'    -- manually mapped by enterprise-architect + security-engineer (two-person)
                    )),
  matched_at      TIMESTAMPTZ,
  confirmed_at    TIMESTAMPTZ,       -- set when test login with new external_id succeeds in staging
  UNIQUE (tenant_id, migration_id, user_id)
);

-- RLS: form_admin only (BYPASSRLS). This table maps old-to-new identity — never exposed to API or tenants.
ALTER TABLE sso_external_id_mappings ENABLE ROW LEVEL SECURITY;
-- No form_api or form_system policy = zero rows for both roles (fail-closed)

CREATE INDEX idx_ext_id_mappings_migration ON sso_external_id_mappings (migration_id, matched_at);
CREATE INDEX idx_ext_id_mappings_user ON sso_external_id_mappings (user_id, tenant_id);
```

---

### 18.5 Migration Runbook by Type

#### 18.5.1 Type A — Same Protocol, Different IdP Vendor

*Example: Okta SAML → Microsoft Entra ID SAML (no protocol change)*

**Phase 1 — Staging validation (Days 1–14)**

1. Enterprise-architect opens `sso_idp_migrations` row: `status = 'staging'`, `from_idp_type = 'okta'`, `to_idp_type = 'azure_ad'`, `authorized_by` = founder email. Emits `sso.migration_initiated` DEC-030 event (HIGH severity).
2. In the tenant's **staging** `tenant_sso_configs`, update:
   ```sql
   UPDATE tenant_sso_configs
   SET
     idp_type              = 'azure_ad',
     saml_idp_entity_id    = '<new_entra_entity_id>',
     saml_idp_metadata_url = '<new_entra_federation_metadata_url>',
     saml_idp_sso_url      = '<new_entra_sso_url>',
     saml_idp_certificate  = '<new_entra_signing_cert_pem>',
     -- saml_sp_certificate and saml_sp_private_key unchanged (FORM's SP cert stays the same)
     -- group_role_mappings updated with new Azure Object IDs per C-03 mapping
     group_role_mappings   = '<new_jsonb_group_mapping>',
     updated_at            = now()
   WHERE tenant_id = '<tenant_id>' AND is_staging = true;
   ```
3. Customer IT imports FORM's SP metadata at new Entra enterprise app. Test user completes SSO login against staging URL. Confirm `sso.login` DEC-030 event fires with `idp_type = 'azure_ad'`.
4. If SCIM is also migrating (`scim_migration = true`): customer IT configures new SCIM provisioning app against staging SCIM endpoint with new token. Confirm `scim.user_synced` events in staging.
5. Run `sso_external_id_mappings` population for all active users (§18.6).
6. Confirm 100% match rate before proceeding to Phase 2. Any unmatched user requires manual resolution (§18.6.3).
7. Enterprise-architect marks staging validated: `UPDATE sso_idp_migrations SET status = 'cutover_pending', staging_validated_at = now()`. Emits `sso.migration_staging_validated` DEC-030 event (MEDIUM severity).

**Phase 2 — Production cutover (maintenance window, ~30 minutes)**

1. Announce maintenance window in `#inc-<slug>-idp-migration` Slack channel. CS notifies customer IT admin and FORM on-call.
2. Set `tenant_sso_configs.is_active = false` on production config: users immediately fall back to email magic link (§10). No users are hard-locked out.
3. Copy staging `tenant_sso_configs` fields (idp_type, saml_idp_entity_id, saml_idp_metadata_url, saml_idp_certificate, group_role_mappings) to production row.
4. Apply `external_id` remaps from `sso_external_id_mappings` to production `users` table:
   ```sql
   -- Two-person SQL review required: enterprise-architect + security-engineer must both approve this PR
   UPDATE users u
   SET
     external_id       = m.new_external_id,
     idp_name_id       = m.new_external_id,   -- SLO NameID anchor updated to match
     updated_at        = now()
   FROM sso_external_id_mappings m
   WHERE
     u.id          = m.user_id
     AND u.tenant_id = m.tenant_id
     AND m.migration_id = '<migration_id>'
     AND m.new_external_id IS NOT NULL;
   ```
   This SQL requires a GitHub PR with `enterprise-architect` + `security-engineer` as required reviewers. Merge gate must require both approvals. (Same two-person rule as §17.7 Option B.)
5. Re-enable: `UPDATE tenant_sso_configs SET is_active = true WHERE tenant_id = '<tenant_id>' AND is_staging = false`.
6. Customer IT admin performs live test login on production with new IdP. Confirm `sso.login` event fires; confirm user's `tenant_member_roles` row is unchanged; confirm work sessions resume normally.
7. Update migration record: `UPDATE sso_idp_migrations SET status = 'committed', cutover_at = now()`. Emits `sso.migration_production_committed` DEC-030 event (HIGH severity).

#### 18.5.2 Type B — Protocol Upgrade (SAML → OIDC)

Protocol upgrades on the same IdP (e.g., customer IT team enables OIDC on Okta and wants to switch from legacy SAML app) follow the same staging-first procedure with one critical difference: **the SP configuration changes entirely** (no SP certificate reuse — OIDC uses client credentials, not SAML assertions).

Additional steps unique to Type B:

1. New `oidc_client_id` and `oidc_client_secret_enc` must be generated on the IdP side and stored in staging `tenant_sso_configs`. The SAML columns (`saml_idp_*`, `saml_sp_*`) are nulled out on the staging row.
2. `external_id` source changes: SAML uses NameID (typically `user.email` or `user.employeeId`); OIDC uses `sub` claim (typically an opaque identifier). **The `sub` will not match the old NameID.** `sso_external_id_mappings` must resolve every user via email-match strategy (§18.6.2) — manual override required for any user whose email differs.
3. SAML SLO (§13) is no longer applicable after migration. OIDC back-channel logout (§13) must be configured and tested in staging before cutover.
4. After committed cutover, verify `users.idp_name_id` is cleared (SAML-only field) to prevent stale SAML SLO attempts. OIDC sessions do not use `idp_name_id`.

#### 18.5.3 Type C — SCIM-Only Migration

When the SSO protocol and IdP vendor remain the same but the SCIM provisioning application changes (e.g., customer IT decommissions old Okta SCIM app and creates a new one):

1. No `tenant_sso_configs` update required for SSO fields.
2. Issue new `tenant_scim_tokens` entry in staging. Provide new endpoint URL and bearer token to customer IT.
3. Customer IT runs a full SCIM sync against staging. Verify all `scim.user_synced` events land with same `user_id` values (email-match reconciliation happens automatically via the existing SCIM PUT handler).
4. Old SCIM token: do NOT revoke until 24-hour overlap window after new SCIM app is confirmed stable in production. Use the existing 24-hour dual-token window from `tenant_scim_tokens` rotation.
5. SCIM group IDs may change (new Okta group Push IDs vs. old ones). Update `scim_groups.external_id` values and `scim_group_role_mappings.idp_group_id` values in production after confirming group sync in staging.

---

### 18.6 SCIM User Identity Preservation

#### 18.6.1 The external_id Problem

Every FORM user has `users.external_id` — the subject claim from the IdP (`sub` for OIDC, `NameID` for SAML). When a tenant switches IdPs, the new IdP assigns a different subject identifier for the same employee. Without remapping, the first SSO login from the new IdP would create a **duplicate user record** rather than matching the existing one, fragmenting the user's workout history, coaching history, wearable integrations, and consent records.

SCIM provisioning faces the same problem: `users.scim_external_id` (the SCIM `externalId` attribute) is set by the sending IdP. A new SCIM provisioning app typically sends different `externalId` values.

#### 18.6.2 Strategy 1: Email-Match (Default)

When the employee's work email address is identical in old and new IdP — the common case for same-org IdP replacements — FORM matches on `users.email`:

```sql
-- Populate sso_external_id_mappings for email-match strategy
-- Run against staging; verify before any production change
INSERT INTO sso_external_id_mappings (
  tenant_id, migration_id, user_id,
  old_external_id, new_external_id,
  match_method, matched_at
)
SELECT
  u.tenant_id,
  '<migration_id>',
  u.id,
  u.external_id,           -- current (old IdP) external_id
  scim.external_id_new,    -- new IdP's externalId from staging SCIM sync
  'email',
  now()
FROM users u
JOIN (
  -- Staging SCIM provisioning log: new externalId keyed by email
  SELECT DISTINCT ON (email)
    email,
    new_external_id AS external_id_new
  FROM scim_provisioning_log
  WHERE
    tenant_id   = '<staging_tenant_id>'
    AND action  = 'upsert'
    AND created_at > '<migration_start_at>'
  ORDER BY email, created_at DESC
) scim ON scim.email = u.email
WHERE
  u.tenant_id = '<tenant_id>'
  AND u.is_active = true;

-- Verify: any unmatched users (no entry in mapping table)?
SELECT u.id, u.email
FROM users u
LEFT JOIN sso_external_id_mappings m
  ON m.user_id = u.id AND m.migration_id = '<migration_id>'
WHERE
  u.tenant_id = '<tenant_id>'
  AND u.is_active = true
  AND m.id IS NULL;
-- Must return 0 rows before proceeding to cutover_pending
```

#### 18.6.3 Strategy 2: Manual ID Mapping

For M&A scenarios where employee emails change (e.g., `alice@acquiredco.com` → `alice@acquirer.com`), email-match fails. The enterprise-architect must produce a CSV mapping and apply it with two-person SQL review:

```
old_email,new_email,old_external_id,new_external_id
alice@acquiredco.com,alice@acquirer.com,okta|00ud8...,9a3f72...
```

The `match_method` for these rows is `'manual'`. Every manual mapping row must be approved by both `enterprise-architect` and `security-engineer` before it is applied to the `sso_external_id_mappings` table. The approval is recorded via the GitHub PR review trail (same gate as §17.7 Option B SQL review).

**Hard limit:** If more than 20% of active users require manual mapping, escalate to founder before proceeding. Mass manual remapping at this scale indicates a data quality problem in the customer's IdP that must be resolved on their side first.

---

### 18.7 Rollback Procedure

Rollback is available at any phase up to and including the first 48 hours post-cutover. After 48 hours, the old IdP application is decommissioned by customer IT (C-05) and rollback is no longer possible without customer IT re-activating it.

**Trigger conditions for rollback:**
- SSO failure rate > 5% for 10 consecutive minutes post-cutover (OBSERVABILITY.md §13 alert)
- `sso.login_failed` DEC-030 events with `failure_reason = 'external_id_not_found'` for > 10 unique users
- SCIM sync error rate > 10% in first hour post-cutover
- Customer IT admin requests rollback during 48-hour window

**Rollback steps:**

1. Immediately re-disable production SSO: `UPDATE tenant_sso_configs SET is_active = false` — users fall back to magic link (§10). Emits `sso.migration_rolled_back` DEC-030 event (HIGH severity).
2. Reverse `external_id` remaps on production `users` table using `sso_external_id_mappings.old_external_id`:
   ```sql
   -- Two-person review required (same gate as cutover)
   UPDATE users u
   SET
     external_id  = m.old_external_id,
     idp_name_id  = m.old_external_id,
     updated_at   = now()
   FROM sso_external_id_mappings m
   WHERE
     u.id          = m.user_id
     AND u.tenant_id = m.tenant_id
     AND m.migration_id = '<migration_id>';
   ```
3. Restore production `tenant_sso_configs` fields to pre-migration values (backed up in `sso_idp_migrations.notes` JSONB — enterprise-architect records snapshot at migration open).
4. Re-enable: `UPDATE tenant_sso_configs SET is_active = true`.
5. Customer IT admin confirms old IdP SSO login succeeds. Update `sso_idp_migrations.status = 'rolled_back'`, `rolled_back_at = now()`.
6. Notify customer: provide written summary of what failed and expected timeline for re-attempt.
7. Retain `sso_idp_migrations` and `sso_external_id_mappings` records — do not delete. Post-incident review required before scheduling a new migration attempt.

---

### 18.8 Post-Migration Validation Checklist

Run within 2 hours of `status = 'committed'`:

| # | Check | Method | Pass criteria |
|---|---|---|---|
| V-01 | SSO error rate | OBSERVABILITY.md §13 per-tenant dashboard | `sso.login_failed` rate < 0.5% in first hour |
| V-02 | All pre-migration users can log in | `sso.login` count in 24h ≥ 80% of `active_user_count` snapshot | — |
| V-03 | No duplicate users created | `SELECT COUNT(*) FROM users WHERE tenant_id = ... AND created_at > cutover_at` | Should be ≈ 0 (only genuinely new employees) |
| V-04 | Role assignments preserved | Sample 10 random users: confirm `tenant_member_roles.role` matches pre-migration state | Manual spot-check |
| V-05 | SCIM sync functional | Deprovision a test user in new IdP → confirm `users.is_active = false` in FORM within 10 minutes | Automated smoke test if available |
| V-06 | Group role mappings active | New IdP group triggers correct FORM role for a test user | Manual with customer IT |
| V-07 | HMAC chain intact | Run §R-05 chain verification query against `audit_log_events` for tenant since cutover | Zero chain breaks |
| V-08 | No cross-tenant bleed | Standard `rls_isolation.test.ts` CI suite passes against production (read-only mode) | All assertions pass |

---

### 18.9 DEC-030 Audit Events

Seven new events added to the DEC-030 HMAC-chained taxonomy. All require `migration_id` in metadata. Events are written to `audit_log_events` with `tenant_id` of the production tenant.

| Event type | Trigger | Severity | Retention | Key metadata |
|---|---|---|---|---|
| `sso.migration_initiated` | `sso_idp_migrations` row created | HIGH | 7 years | `migration_id`, `from_idp_type`, `to_idp_type`, `from_protocol`, `to_protocol`, `authorized_by`, `active_user_count` |
| `sso.migration_staging_validated` | `status` → `'cutover_pending'` | MEDIUM | 7 years | `migration_id`, `matched_user_count`, `unmatched_user_count`, `staging_validated_at` |
| `sso.migration_production_committed` | `status` → `'committed'` after cutover SQL | HIGH | 7 years | `migration_id`, `cutover_at`, `users_remapped_count`, `scim_migration` |
| `sso.migration_rolled_back` | `status` → `'rolled_back'` | HIGH | 7 years | `migration_id`, `rolled_back_at`, `rollback_reason`, `users_restored_count` |
| `sso.external_id_remapped` | Each row written to `sso_external_id_mappings` with `confirmed_at` set | HIGH | 7 years | `migration_id`, `user_id` (UUID), `match_method`; **no** `old_external_id` or `new_external_id` in event — values are in the mapping table, not the audit log |
| `sso.migration_old_config_decommissioned` | Form engineer marks old IdP app decommissioned (48h post-cutover) | MEDIUM | 7 years | `migration_id`, `decommissioned_at` |
| `sso.scim_token_rotated_for_migration` | New SCIM token issued, old token revocation scheduled | HIGH | 7 years | `migration_id`, `new_token_id`, `old_token_revocation_at` |

**Note on `sso.external_id_remapped` event design:** The `old_external_id` and `new_external_id` values are deliberately omitted from the DEC-030 event payload. These values are IdP subject identifiers that, combined, create a linkage between the old and new identity — which is sensitive data. The `sso_external_id_mappings` table (form_admin access only, BYPASSRLS) is the authoritative record. The DEC-030 event proves the action occurred and names the `user_id`; auditors query the mapping table directly for the actual values under break-glass procedure.

---

### 18.10 SOC 2 Evidence Mapping

| SOC 2 Criterion | How §18 Satisfies It |
|---|---|
| **CC8.1 — Change management** | The migration runbook is a documented change management procedure for identity provider changes. The `sso_idp_migrations` state machine enforces a planning → staging → cutover_pending → committed flow with no skip transitions. Founder authorisation (`authorized_by` column) is required before any production change. Two-person SQL review (§18.5.1 step 4, §18.6.3) mirrors the pattern established in §17.7. The `sso.migration_production_committed` HMAC-chained event creates an immutable record of when the change was made and who authorised it. |
| **CC6.1 — Logical access controls** | The staging-first validation requirement (§18.3 F-01) ensures that new IdP access controls are verified against FORM's RLS before production activation. The `external_id` remap procedure (§18.6) guarantees that user identity continuity is preserved — preventing duplicate accounts from receiving access that should belong to an existing user. The V-04 spot-check and V-08 RLS isolation test confirm access integrity post-migration. |
| **CC6.2 — Access controls prior to issuing credentials** | `tenant_sso_configs.is_active = false` during cutover (§18.5.1 step 2) ensures no SSO credentials are accepted from the new IdP until the `external_id` remap is complete. The rollback trigger conditions (§18.7) fire before access control failures can propagate to more than 10 users. |
| **CC7.2 — System monitoring** | OBSERVABILITY.md §13 per-tenant SSO monitoring provides the post-cutover failure rate signal (V-01). The `sso.login_failed` event with `failure_reason` label allows automatic detection of `external_id_not_found` rollback triggers. |
| **CC9.2 — Vendor/customer management** | The migration procedure protects user data integrity across the identity provider change — a vendor/customer obligation. The 48-hour rollback window (C-05) and post-migration validation checklist (§18.8) document FORM's commitment to continuity. The `sso.migration_old_config_decommissioned` event records when the old vendor relationship was formally closed. |
| **C1.1 — Confidentiality commitments** | The `sso.external_id_remapped` event design (§18.9) deliberately excludes `old_external_id` and `new_external_id` values from the audit log to prevent unnecessary identity-linkage exposure. The `sso_external_id_mappings` table is BYPASSRLS form_admin access only — the minimum necessary access for a security-critical operation. |

**Auditor evidence artefacts:**
- `sso_idp_migrations` table export for any migration during observation period: status, `authorized_by`, timestamps
- `audit_log_events` of type `sso.migration_*` for the migration, HMAC chain verified
- Two-person SQL review approval history (GitHub PR): `enterprise-architect` + `security-engineer` required reviewers
- Pre-migration assessment checklist: `compliance/evidence/idp-migrations/{slug}-{date}/assessment.md`
- Post-migration validation results: `compliance/evidence/idp-migrations/{slug}-{date}/validation.md`

---

### 18.11 Gap Status Updates

| Gap | Before §18 | After §18 |
|---|---|---|
| **G-004** (Certificate rotation automation) | 🟡 Partial — §8.1 rotation procedure exists but is manual; no expiry cron or dual-cert metadata endpoint | 🟡 Partial — §18 provides the full migration runbook which subsumes certificate-replacement scenarios. G-004 closes to 🟢 when the 60-day expiry alert cron (OBSERVABILITY.md §6 alert rule) and admin dashboard certificate management UI (§16.3) are implemented. |

---

### 18.12 Implementation Checklist

| Task | Owner | Priority | Milestone |
|---|---|---|---|
| Write `migrations/YYYYMMDD_sso_idp_migrations.sql` — `sso_idp_migrations` table DDL, exclusion constraint, indexes, RLS policies | platform-engineer | **P0** | M5 |
| Write `migrations/YYYYMMDD_sso_external_id_mappings.sql` — `sso_external_id_mappings` table DDL, indexes, BYPASSRLS-only policy | platform-engineer + security-engineer | **P0** | M5 |
| Implement `POST /internal/admin/sso/migrations` — create migration record with founder-auth check; emit `sso.migration_initiated` DEC-030 event | platform-engineer | **P0** | M5 |
| Implement email-match population query (§18.6.2) as a managed script in `scripts/sso-migration-email-match.ts`; dry-run mode that reports unmatched count without writing | platform-engineer | **P0** | M5 |
| Implement production `external_id` remap SQL as a reviewed migration script template in `scripts/sso-migration-apply-remap.sql`; wire GitHub PR template requiring enterprise-architect + security-engineer approval | enterprise-architect + security-engineer | **P0** | M5 |
| Wire all 7 DEC-030 events (§18.9) to their respective triggers; validate HMAC chain in staging | platform-engineer + security-engineer | **P0** | M5 |
| Implement rollback script `scripts/sso-migration-rollback.sql` reversing `external_id` from `sso_external_id_mappings.old_external_id`; same two-person PR review gate | enterprise-architect + security-engineer | **P0** | M5 |
| Add post-migration validation checks V-01 through V-08 (§18.8) as a runnable script in `scripts/sso-migration-validate.ts` outputting pass/fail per check | platform-engineer | **P1** | M5 |
| Add `sso_idp_migrations` read panel to admin dashboard (internal FORM view only — not customer-facing): active migrations, status, days since cutover | platform-engineer | **P1** | M5 |
| Create `compliance/evidence/idp-migrations/TEMPLATE-assessment.md` — pre-migration assessment checklist template for CS to fill out | compliance-officer | **P1** | M5 |
| Create `compliance/evidence/idp-migrations/TEMPLATE-validation.md` — post-migration validation template | compliance-officer | **P1** | M5 |
| Add `sso_idp_migrations` to Art. 17 erasure scope: on tenant hard-delete, confirm mapping table rows are deleted (referential cascade handles `sso_external_id_mappings`) | platform-engineer | **P2** | M6 |

---

*v1.0 additions: Section 18 — Tenant IdP Migration Runbook. Closes the identity lifecycle gap: §1–§17 cover provisioning, session management, logout, GDPR compliance, groups, admin UI, and pilot programs; §18 covers the previously undocumented scenario of migrating between identity providers without fragmenting user history. Migration trigger table: 6 scenarios from M&A-driven vendor replacement to on-premises ADFS → cloud IdP, with estimated engineering effort and critical-path note (customer IT coordination). Pre-migration assessment: 6 FORM-side checks (F-01 through F-06, including staging environment prerequisite and founder authorisation requirement) and 6 customer IT checks (C-01 through C-06, including mandatory 48-hour rollback window). New database tables: `sso_idp_migrations` (state machine with exclusion constraint preventing concurrent active migrations per tenant; `authorized_by` column enforced at API layer; BYPASSRLS for form_admin; form_system read-only; form_api zero-rows fail-closed) and `sso_external_id_mappings` (old↔new external_id linkage; form_admin BYPASSRLS only — never exposed to API or tenants; retains match_method and confirmed_at for audit trail). Three migration type runbooks: Type A (same protocol, different vendor — SAML field swap with two-person external_id remap SQL, staging-first, 30-minute maintenance window), Type B (SAML → OIDC protocol upgrade — SP reconfiguration, NameID→sub resolution via email-match, SAML SLO cleanup), Type C (SCIM-only — dual-token 24-hour overlap, group external_id update). SCIM identity preservation: the external_id problem (new IdP assigns new subject IDs; naïve handling creates duplicate users); Strategy 1 email-match (default — staging SCIM sync keyed by email; SQL to populate mappings and verify 0 unmatched); Strategy 2 manual ID mapping (M&A email-change scenario; CSV import with two-person review; hard limit at 20% manual rate requiring escalation). Rollback: 4 trigger conditions (SSO failure rate, login_failed count, SCIM error rate, customer IT request); reverse-remap SQL; `status = 'rolled_back'`; post-incident review gate before re-attempt. Post-migration validation checklist: 8 checks (V-01 SSO error rate via OBSERVABILITY §13, V-02 login coverage, V-03 no duplicate users, V-04 role preservation spot-check, V-05 SCIM deprovision smoke test, V-06 group mapping activation, V-07 HMAC chain integrity, V-08 RLS isolation CI suite). Seven new DEC-030 HMAC-chained events: sso.migration_initiated (HIGH), sso.migration_staging_validated (MEDIUM), sso.migration_production_committed (HIGH), sso.migration_rolled_back (HIGH), sso.external_id_remapped (HIGH — no id values in event; design rationale documented), sso.migration_old_config_decommissioned (MEDIUM), sso.scim_token_rotated_for_migration (HIGH). SOC 2 mapping: CC8.1 (change management — state machine + founder auth + two-person SQL gate), CC6.1 (no duplicate access via remap procedure), CC6.2 (is_active gate during cutover), CC7.2 (OBSERVABILITY §13 rollback signal), CC9.2 (48h rollback window + decommission event), C1.1 (external_id values excluded from audit log). G-004 status update: 🟡 Partial → 🟡 Partial (§18 covers certificate-replacement scenarios; G-004 closes on expiry cron + admin UI implementation). Auditor evidence artefacts: sso_idp_migrations export, sso.migration_* chain, two-person SQL PR reviews, compliance/evidence/idp-migrations/ templates. Implementation checklist: 12 tasks (7× P0, 4× P1, 1× P2), M5/M6.*

---

## 19. SCIM Groups Sync & Group-Based Role Mapping

> Precursors: §6 (SCIM User API — user lifecycle endpoints), §8 (Role Mapping — direct user role assignment), §18 (IdP Migration — group external_id update on SCIM-only migration). This section closes the group lifecycle gap: §5–§8 handle user provisioning and direct role assignment; §19 handles group objects as first-class SCIM resources and the mapping from group membership to FORM roles.

---

### 19.1 Purpose and Scope

Enterprise IT workflows provision access in two stages: (1) create/update the user record, (2) assign the user to one or more groups that carry role semantics. When FORM only supported individual user role assignment (§8), IT administrators were required to set each user's FORM role explicitly after provisioning — an error-prone manual step that scaled poorly beyond ~50 users and blocked fully-automated onboarding pipelines.

SCIM Groups Sync solves this by treating groups as a first-class resource:

- IdP groups (Okta groups, Azure AD security groups, Google groups via bridge) are pushed to FORM's `/scim/v2/Groups` endpoint.
- A tenant admin maps each group to a FORM role via the admin dashboard "Group Role Mapping" panel.
- When a user is added to a group in the IdP, the SCIM provisioner pushes a `PATCH /scim/v2/Groups/{id}` with `op=add` for that user. FORM resolves the user's effective role automatically.
- When a user is removed from all groups or deprovisioned entirely, §6 SCIM user lifecycle handles suspension; SCIM group removal ensures role is withdrawn even if the user account persists.

**What is not in scope for §19:**
- Nested group hierarchies (not defined by SCIM 2.0; see OQ-SSO-19.2).
- IdP-side group provisioning (FORM is always the Service Provider receiving pushes, never initiating group writes to the IdP).
- Consumer (non-enterprise) tenants — all `/scim/v2/Groups` endpoints require a valid `tenant_scim_tokens` bearer token scoped to an enterprise tenant.

---

### 19.2 SCIM Groups Endpoint Specification

All endpoints are hosted on the Cloudflare Workers edge runtime at the same base URL as the SCIM User API (§6): `https://api.form.coach/scim/v2/`. Authentication: `Authorization: Bearer <tenant_scim_token>` — same token used for user provisioning. The token resolves `tenant_id` server-side; no tenant identifier appears in the URL path. All responses use `Content-Type: application/scim+json`.

#### 19.2.1 GET /scim/v2/Groups — List Groups

Returns an RFC 7644 `ListResponse` of all groups for the authenticated tenant.

**Query parameters:**

| Parameter | Type | Description |
|---|---|---|
| `startIndex` | integer (1-based) | Pagination offset. Default: `1`. |
| `count` | integer | Page size. Default: `100`. Max: `1000`. |
| `filter` | string | RFC 7644 filter expression. Supported attribute: `displayName`. Operator: `eq`, `sw` (starts-with). Example: `filter=displayName eq "FORM-Admins"`. |

**TypeScript pseudo-code (Cloudflare Worker):**

```typescript
app.get('/scim/v2/Groups', async (c) => {
  const tenantId = c.var.scimToken.tenant_id; // resolved by auth middleware
  const startIndex = Math.max(1, parseInt(c.req.query('startIndex') ?? '1'));
  const count = Math.min(1000, parseInt(c.req.query('count') ?? '100'));
  const filter = c.req.query('filter');

  // form_admin BYPASSRLS connection — SCIM bearer resolves tenant_id; RLS-enforced
  // query still scopes explicitly to tenant_id for defence-in-depth
  const { groups, total } = await db.scimGroups.list({
    tenantId,
    offset: startIndex - 1,
    limit: count,
    displayNameFilter: parseScimFilter(filter), // returns { op, value } | null
  });

  return c.json({
    schemas: ['urn:ietf:params:scim:api:messages:2.0:ListResponse'],
    totalResults: total,
    startIndex,
    itemsPerPage: groups.length,
    Resources: groups.map(toScimGroupResource),
  }, 200);
});
```

**Response shape (abbreviated):**

```json
{
  "schemas": ["urn:ietf:params:scim:api:messages:2.0:ListResponse"],
  "totalResults": 3,
  "startIndex": 1,
  "itemsPerPage": 3,
  "Resources": [
    {
      "schemas": ["urn:ietf:params:scim:core:2.0:Group"],
      "id": "grp_01HZ...",
      "displayName": "FORM-Admins",
      "members": [
        { "value": "usr_01HZ...", "display": "alice@acme.com" }
      ],
      "meta": {
        "resourceType": "Group",
        "created": "2025-03-10T09:00:00Z",
        "lastModified": "2025-05-01T14:22:00Z",
        "location": "https://api.form.coach/scim/v2/Groups/grp_01HZ..."
      }
    }
  ]
}
```

#### 19.2.2 GET /scim/v2/Groups/{id} — Single Group

Returns a single SCIM Group resource including the full `members` array. Returns `404` if the group does not exist within the authenticated tenant (no cross-tenant information leakage — a group belonging to another tenant returns the same `404` as a non-existent ID).

```typescript
app.get('/scim/v2/Groups/:id', async (c) => {
  const tenantId = c.var.scimToken.tenant_id;
  const group = await db.scimGroups.findOne({ tenantId, id: c.req.param('id') });
  if (!group) return scimError(c, 404, 'Group not found');
  return c.json(toScimGroupResource(group), 200);
});
```

#### 19.2.3 POST /scim/v2/Groups — Create Group

Creates a new group under the authenticated tenant. Inserts a row into `tenant_groups` and returns `201 Created` with a `Location` header pointing to the new resource. Idempotency: if a group with the same `externalId` already exists for the tenant, returns `409 Conflict` rather than creating a duplicate.

```typescript
app.post('/scim/v2/Groups', async (c) => {
  const tenantId = c.var.scimToken.tenant_id;
  const body = await c.req.json<ScimGroupCreateRequest>();

  if (!body.displayName) return scimError(c, 400, 'displayName is required');

  // Idempotency check on externalId if provided by IdP
  if (body.externalId) {
    const existing = await db.scimGroups.findByExternalId({ tenantId, externalId: body.externalId });
    if (existing) return scimError(c, 409, 'Group with this externalId already exists');
  }

  const group = await db.scimGroups.create({
    tenantId,
    displayName: body.displayName,
    externalId: body.externalId ?? null,
    members: body.members ?? [],
  });

  await emitAuditEvent('scim.group_created', { tenantId, groupId: group.id, triggeredBy: 'scim' });

  const location = `https://api.form.coach/scim/v2/Groups/${group.id}`;
  c.header('Location', location);
  return c.json(toScimGroupResource(group), 201);
});
```

#### 19.2.4 PUT /scim/v2/Groups/{id} — Full Replace

Replaces the entire group resource including its `members` array. Idempotent: calling PUT twice with identical payloads produces the same final state and emits a single `scim.group_updated` event only if a delta exists. Members not present in the PUT body are removed from the group.

```typescript
app.put('/scim/v2/Groups/:id', async (c) => {
  const tenantId = c.var.scimToken.tenant_id;
  const id = c.req.param('id');
  const body = await c.req.json<ScimGroupReplaceRequest>();

  const existing = await db.scimGroups.findOne({ tenantId, id });
  if (!existing) return scimError(c, 404, 'Group not found');

  const updated = await db.scimGroups.replace({ tenantId, id, displayName: body.displayName, members: body.members ?? [] });

  if (hasChanges(existing, updated)) {
    await emitAuditEvent('scim.group_updated', { tenantId, groupId: id, triggeredBy: 'scim' });
  }

  return c.json(toScimGroupResource(updated), 200);
});
```

#### 19.2.5 PATCH /scim/v2/Groups/{id} — Incremental Member Add / Remove

Implements RFC 7644 §3.5.2 PatchOp. Supports `op=add` and `op=remove` on `path=members`. This is the primary operation used by Okta and Azure AD push-group provisioning — they do not send full PUT payloads on member change, only PATCHes.

**PatchOp payload examples:**

Add a member:
```json
{
  "schemas": ["urn:ietf:params:scim:api:messages:2.0:PatchOp"],
  "Operations": [
    {
      "op": "add",
      "path": "members",
      "value": [{ "value": "usr_01HZ..." }]
    }
  ]
}
```

Remove a member:
```json
{
  "schemas": ["urn:ietf:params:scim:api:messages:2.0:PatchOp"],
  "Operations": [
    {
      "op": "remove",
      "path": "members",
      "value": [{ "value": "usr_01HZ..." }]
    }
  ]
}
```

**Worker pseudo-code:**

```typescript
app.patch('/scim/v2/Groups/:id', async (c) => {
  const tenantId = c.var.scimToken.tenant_id;
  const id = c.req.param('id');
  const body = await c.req.json<ScimPatchRequest>();

  const group = await db.scimGroups.findOne({ tenantId, id });
  if (!group) return scimError(c, 404, 'Group not found');

  for (const op of body.Operations) {
    if (op.path !== 'members') return scimError(c, 400, `Unsupported path: ${op.path}`);

    const userIds = (op.value as Array<{ value: string }>).map(v => v.value);

    if (op.op === 'add') {
      // Verify all userIds belong to this tenant before inserting (cross-tenant guard)
      await assertUsersInTenant(tenantId, userIds);
      await db.scimGroupMembers.addBatch({ tenantId, groupId: id, userIds });
      for (const userId of userIds) {
        await emitAuditEvent('scim.group_member_added', { tenantId, groupId: id, userId, triggeredBy: 'scim' });
      }
    } else if (op.op === 'remove') {
      await db.scimGroupMembers.removeBatch({ tenantId, groupId: id, userIds });
      for (const userId of userIds) {
        await emitAuditEvent('scim.group_member_removed', { tenantId, groupId: id, userId, triggeredBy: 'scim' });
      }
    } else {
      return scimError(c, 400, `Unsupported op: ${op.op}`);
    }
  }

  const updated = await db.scimGroups.findOne({ tenantId, id });
  return c.json(toScimGroupResource(updated!), 200);
});
```

**Constraint:** `op=replace` on `path=members` is treated equivalently to a full member list replacement (same semantics as PUT). Any `op` value other than `add`, `remove`, or `replace` returns `400 Bad Request`.

#### 19.2.6 DELETE /scim/v2/Groups/{id} — Delete Group

Removes the group from `tenant_groups`. Cascades to `scim_group_members` (membership rows deleted) and `scim_group_role_mappings` (role mapping deleted). Does **not** delete or suspend member users — user lifecycle is governed by §6 SCIM user endpoints. Returns `204 No Content` on success.

```typescript
app.delete('/scim/v2/Groups/:id', async (c) => {
  const tenantId = c.var.scimToken.tenant_id;
  const id = c.req.param('id');

  const group = await db.scimGroups.findOne({ tenantId, id });
  if (!group) return scimError(c, 404, 'Group not found');

  // Cascade handled by FK ON DELETE CASCADE in DDL (§19.4)
  await db.scimGroups.delete({ tenantId, id });

  await emitAuditEvent('scim.group_deleted', {
    tenantId,
    groupId: id,
    displayName: group.displayName, // display name only — no user PII in audit log
    triggeredBy: 'scim',
  });

  return new Response(null, { status: 204 });
});
```

**Important:** Deletion of a group that carries a role mapping (via `scim_group_role_mappings`) is an access-reduction event. The role the group conferred is immediately withdrawn from all members who held that role exclusively through group membership. Users who also hold a direct role assignment (§8) retain that direct assignment unaffected.

---

### 19.3 Group → FORM Role Mapping

#### 19.3.1 Mapping Model

Group-to-role mapping is a separate, explicitly-configured resource — it is not automatic. A tenant admin must visit the admin dashboard "Group Role Mapping" panel and assign a FORM role to each group. This is a deliberate fail-closed design: a newly pushed group has no role until an admin maps it. This satisfies CC6.2 (no auto-escalation of access).

**FORM roles available for group mapping:**

| FORM Role | Description | Equivalent direct-assign role (§8) |
|---|---|---|
| `owner` | Full tenant control | `tenant_owner` |
| `admin` | Manage users, configure, view analytics | `tenant_admin` |
| `manager` | View team analytics (no PII) | `tenant_manager` |
| `member` | Regular FORM user inside org | `member` |
| `viewer` | Read-only access to shared content | `support_readonly` |

#### 19.3.2 Role Resolution Priority

When a user is a member of multiple groups with different role mappings, or holds both a direct role assignment and a group-derived role, the following priority order applies:

1. **Explicit direct user role assignment** (set by a tenant admin or via SCIM `roles` attribute on the user resource — §6/§8) — always wins over any group-derived role.
2. **Group membership — highest privilege wins.** If a user is in Group A mapped to `member` and Group B mapped to `admin`, the effective role is `admin`.

Rationale for highest-privilege-wins on group conflict (see OQ-SSO-19.1): enterprise IT departments assign users to groups based on job function, not minimum access. A user assigned to both a broad "All Staff" group (member) and a specialist "FORM Coaches" group (admin) should receive the more permissive role reflecting their specialist function. Lowest-privilege-wins would require IT to exclude specialist users from general groups — a fragile configuration that breaks with group nesting patterns common in Active Directory. This choice is documented here and requires founder sign-off before the feature ships (OQ-SSO-19.1).

**Role resolution is computed at session creation time,** not stored. `group_member_effective_role` view (§19.4.4) provides the real-time computed role for any user. The session token carries the resolved role; role changes propagate on next login or token refresh (whichever comes first — configured by tenant session policy §12).

#### 19.3.3 Admin Dashboard — Group Role Mapping Panel

Location in admin dashboard: **Settings > Identity > Group Role Mapping**.

Panel behaviour:
- Lists all groups currently synced for the tenant (read from `tenant_groups`).
- Each row displays: group display name, member count, current mapped role (or "— unmapped —"), last synced timestamp.
- Role selector: dropdown with options `owner`, `admin`, `manager`, `member`, `viewer`, "Remove mapping".
- Save button is per-row (not a bulk save) to prevent accidental bulk changes.
- Changing a mapping emits `scim.group_role_mapping_changed` DEC-030 event immediately (before HTTP response, same transaction).
- Warning banner displayed when saving a mapping to `owner` or `admin`: "This grants elevated access to all N members of this group. Confirm?"
- Groups with no role mapping are displayed in grey with a "Not mapped — no access granted via this group" tooltip.

**Default for new tenants:** no group-to-role mappings exist. Tenant admin must configure explicitly. SCIM provisioning can push groups and members freely without mapping; those users receive no group-derived role until mapping is configured.

---

### 19.4 Database Schema

#### 19.4.1 Existing tenant_groups Table

The `tenant_groups` table stores SCIM group objects. It is the authoritative source for group identity within a tenant. Columns referenced by §19:

```sql
-- Reference — table created in earlier migration (§6/groups schema)
-- Shown here for FK reference clarity; do not re-run this DDL
CREATE TABLE tenant_groups (
  id            uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id     uuid NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
  display_name  text NOT NULL,
  external_id   text,           -- IdP-assigned group ID (Okta groupId, Azure Object ID, etc.)
  created_at    timestamptz NOT NULL DEFAULT now(),
  updated_at    timestamptz NOT NULL DEFAULT now(),
  UNIQUE (tenant_id, external_id)
);
```

#### 19.4.2 scim_group_role_mappings DDL

```sql
CREATE TABLE scim_group_role_mappings (
  id            uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id     uuid NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
  scim_group_id uuid NOT NULL REFERENCES tenant_groups(id) ON DELETE CASCADE,
  form_role     text NOT NULL CHECK (form_role IN ('owner', 'admin', 'manager', 'member', 'viewer')),
  mapped_by     uuid NOT NULL REFERENCES users(id),   -- tenant admin who set the mapping
  mapped_at     timestamptz NOT NULL DEFAULT now(),
  UNIQUE (tenant_id, scim_group_id)                   -- one role per group per tenant
);

-- Indexes
CREATE INDEX idx_scim_group_role_mappings_tenant
  ON scim_group_role_mappings (tenant_id);

CREATE INDEX idx_scim_group_role_mappings_group
  ON scim_group_role_mappings (scim_group_id);

-- RLS
ALTER TABLE scim_group_role_mappings ENABLE ROW LEVEL SECURITY;

-- tenant_admin: read + write own tenant
CREATE POLICY scim_group_role_mappings_tenant_admin
  ON scim_group_role_mappings
  FOR ALL
  TO form_api
  USING (
    tenant_id = current_setting('app.current_tenant_id')::uuid
    AND EXISTS (
      SELECT 1 FROM tenant_member_roles tmr
      WHERE tmr.user_id = current_setting('app.current_user_id')::uuid
        AND tmr.tenant_id = scim_group_role_mappings.tenant_id
        AND tmr.role IN ('owner', 'admin')
    )
  )
  WITH CHECK (
    tenant_id = current_setting('app.current_tenant_id')::uuid
    AND EXISTS (
      SELECT 1 FROM tenant_member_roles tmr
      WHERE tmr.user_id = current_setting('app.current_user_id')::uuid
        AND tmr.tenant_id = scim_group_role_mappings.tenant_id
        AND tmr.role IN ('owner', 'admin')
    )
  );

-- form_system: read-only (for role resolution at session creation)
CREATE POLICY scim_group_role_mappings_form_system_read
  ON scim_group_role_mappings
  FOR SELECT
  TO form_system
  USING (true);

-- form_admin BYPASSRLS: implicit — no explicit policy needed
-- form_api: zero rows if not tenant_admin (covered by tenant_admin policy above; default-deny)
```

#### 19.4.3 scim_group_members Table

Stores the many-to-many relationship between groups and users as managed by SCIM. This is separate from any direct role assignment rows.

```sql
CREATE TABLE scim_group_members (
  tenant_id  uuid NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
  group_id   uuid NOT NULL REFERENCES tenant_groups(id) ON DELETE CASCADE,
  user_id    uuid NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  added_at   timestamptz NOT NULL DEFAULT now(),
  PRIMARY KEY (group_id, user_id)
);

CREATE INDEX idx_scim_group_members_user
  ON scim_group_members (tenant_id, user_id);

ALTER TABLE scim_group_members ENABLE ROW LEVEL SECURITY;

-- form_system: read (role resolution)
CREATE POLICY scim_group_members_form_system_read
  ON scim_group_members FOR SELECT TO form_system USING (true);

-- form_api: tenant-scoped read (SCIM worker reads membership for PATCH responses)
CREATE POLICY scim_group_members_form_api_tenant
  ON scim_group_members FOR ALL TO form_api
  USING (tenant_id = current_setting('app.current_tenant_id')::uuid);
```

#### 19.4.4 group_member_effective_role View

Resolves the final FORM role for every user in a tenant, combining direct role assignment and group-derived roles using the priority rule from §19.3.2 (explicit direct > highest group role).

```sql
CREATE OR REPLACE VIEW group_member_effective_role AS
WITH direct_roles AS (
  -- Explicit per-user role assignments (§8)
  SELECT
    tmr.tenant_id,
    tmr.user_id,
    tmr.role                        AS direct_role,
    -- Role ordinal for comparison; higher ordinal = higher privilege
    CASE tmr.role
      WHEN 'owner'   THEN 5
      WHEN 'admin'   THEN 4
      WHEN 'manager' THEN 3
      WHEN 'member'  THEN 2
      WHEN 'viewer'  THEN 1
      ELSE 0
    END                             AS direct_ordinal
  FROM tenant_member_roles tmr
),
group_derived_roles AS (
  -- Highest-privilege role from any group the user belongs to
  SELECT
    sgm.tenant_id,
    sgm.user_id,
    MAX(
      CASE sgrm.form_role
        WHEN 'owner'   THEN 5
        WHEN 'admin'   THEN 4
        WHEN 'manager' THEN 3
        WHEN 'member'  THEN 2
        WHEN 'viewer'  THEN 1
        ELSE 0
      END
    )                               AS group_max_ordinal,
    -- Retrieve the actual role label for the max ordinal
    (array_agg(sgrm.form_role ORDER BY
      CASE sgrm.form_role
        WHEN 'owner'   THEN 5
        WHEN 'admin'   THEN 4
        WHEN 'manager' THEN 3
        WHEN 'member'  THEN 2
        WHEN 'viewer'  THEN 1
        ELSE 0
      END DESC
    ))[1]                           AS group_derived_role
  FROM scim_group_members sgm
  JOIN scim_group_role_mappings sgrm
    ON sgrm.scim_group_id = sgm.group_id
   AND sgrm.tenant_id     = sgm.tenant_id
  GROUP BY sgm.tenant_id, sgm.user_id
),
combined AS (
  SELECT
    COALESCE(dr.tenant_id, gdr.tenant_id) AS tenant_id,
    COALESCE(dr.user_id,   gdr.user_id)   AS user_id,
    dr.direct_role,
    dr.direct_ordinal,
    gdr.group_derived_role,
    gdr.group_max_ordinal,
    -- Priority: direct > group
    CASE
      WHEN dr.direct_role IS NOT NULL THEN dr.direct_role
      WHEN gdr.group_derived_role IS NOT NULL THEN gdr.group_derived_role
      ELSE NULL
    END                                   AS effective_role,
    CASE
      WHEN dr.direct_role IS NOT NULL THEN 'direct'
      WHEN gdr.group_derived_role IS NOT NULL THEN 'group'
      ELSE NULL
    END                                   AS role_source
  FROM direct_roles dr
  FULL OUTER JOIN group_derived_roles gdr
    ON dr.tenant_id = gdr.tenant_id AND dr.user_id = gdr.user_id
)
SELECT
  tenant_id,
  user_id,
  effective_role,
  role_source,
  direct_role,
  group_derived_role
FROM combined
WHERE effective_role IS NOT NULL;

-- RLS on the view inherits from underlying tables (form_system, form_api scoping)
-- form_api callers see only their current_tenant_id rows via underlying table RLS
```

---

### 19.5 Okta / Azure AD / Google Workspace Push Group Configuration

#### 19.5.1 Okta — Push Groups

Okta supports native SCIM Group Push. Configuration path in Okta Admin Console:

1. Navigate to **Applications > [FORM SCIM App] > Provisioning > Push Groups**.
2. Click **Push Groups** and select **Find groups by name** or **Find groups by rule**.
3. Recommended rule expression to scope only FORM-relevant groups: `name sw "FORM-"` (groups whose display name starts with "FORM-"). This prevents sending all Okta groups to FORM and keeps the tenant's `tenant_groups` table clean.
4. For each pushed group, select **Push group memberships immediately**.
5. `memberOf` attribute mapping: in the SCIM attribute mappings (**Provisioning > To App > Attribute Mappings**), ensure `groups` is mapped if using SAML attribute-based group extraction (§19.6). For pure SCIM push, `memberOf` mapping is not needed — Okta pushes group membership directly via PATCH.
6. Confirm SCIM Group Push is working: navigate to the Push Groups tab, click the group, and check **Push Status = Active**. FORM must respond `200 OK` or `201 Created` within Okta's 10-second timeout. The Cloudflare Worker target latency for group endpoints is < 200 ms p99.

**Known Okta behaviour:** Okta sends `PUT` (full replace) when a group is first pushed, then `PATCH` for subsequent membership changes. The FORM Worker handles both (§19.2.4, §19.2.5).

#### 19.5.2 Azure AD / Microsoft Entra ID — Group Sync

Azure AD Provisioning (Entra ID) configuration path:

1. **Enterprise Application > [FORM App] > Provisioning > Mappings**.
2. Click **Synchronize Azure Active Directory Groups to [FORM SCIM App]**.
3. Ensure mapping is **Enabled**.
4. Attribute mapping review: verify `displayName → displayName`, `objectId → externalId`, `members → members`. The `objectId` is Azure's stable group identifier — mapping it to `externalId` ensures that if a group is renamed in Azure AD, FORM can reconcile by `externalId` rather than creating a duplicate.
5. Scope filter: under **Provisioning > Settings > Scope**, select **Sync only assigned users and groups** (preferred) or configure an attribute filter. Recommended attribute filter: add a group attribute filter `appRoleAssignments` or, for simpler setups, assign only the relevant security groups to the enterprise application.
6. If the tenant uses Azure AD dynamic groups (membership rule-based), confirm the dynamic group membership is resolved before provisioning runs. Azure AD provisioning runs every ~40 minutes by default; FORM group state may lag by up to 40 minutes after a membership rule change.

**Azure AD SCIM behaviour:** Entra ID sends full membership lists via `PUT` on initial sync and incremental `PATCH` on delta sync. Entra does not support `filter=displayName` queries against FORM's `GET /Groups` endpoint — it uses `GET /Groups?filter=externalId eq "<objectId>"` to check existence before creating. Ensure `externalId` is populated in FORM's `POST /Groups` response and returned in `GET /Groups` responses.

#### 19.5.3 Google Workspace — No Native SCIM Groups Push

Google Workspace does not support outbound SCIM Group Push natively. Google's SCIM support covers user provisioning only (§6); group objects are not pushed to external service providers.

**Workarounds (in order of recommendation):**

| Option | Description | Complexity |
|---|---|---|
| Okta as SCIM bridge | Sync Google Workspace → Okta (Okta's Google Workspace integration); enable Okta Push Groups to FORM | Medium — requires Okta licence |
| Microsoft Entra External ID as bridge | Provision Google identities into Entra; use Entra SCIM provisioning to push groups to FORM | High — cross-IdP complexity |
| SAML `groups` attribute (JIT path) | Configure Google SAML app to include Google Groups in assertion; FORM extracts groups from SAML attribute (§19.6) | Low — no SCIM push; JIT only |
| Manual group management in FORM | Tenant admin manages group membership directly in FORM admin dashboard; no IdP sync | Lowest — loses automation benefit |

The SAML `groups` attribute path (§19.6) is the recommended Google Workspace workaround for tenants that cannot introduce a SCIM bridge.

---

### 19.6 SAML Groups Attribute Mapping (JIT Path)

When a tenant uses SAML SSO (§1) and the IdP does not support SCIM Group Push (or the tenant has not configured push groups), FORM can extract group membership from the SAML assertion at login time. This is the JIT (Just-in-Time) group path — complementary to, not a replacement for, SCIM group sync.

#### 19.6.1 Assertion Attribute Configuration

The IdP must be configured to include group membership in the SAML assertion. Common attribute names:

| IdP | Attribute name in assertion | Format |
|---|---|---|
| Okta | `groups` | Multi-value string list |
| Azure AD / Entra | `http://schemas.microsoft.com/ws/2008/06/identity/claims/groups` | Multi-value (Object IDs) |
| Google (SAML app) | `groups` (custom attribute — must be configured explicitly) | Multi-value string list |
| OneLogin | `memberOf` | Multi-value string list |
| Ping Identity | `memberOf` | Multi-value string list |

FORM reads the attribute name from `tenant_sso_configs.attribute_mapping` JSONB field. To enable group extraction, the tenant admin (or CS engineer via internal tooling) adds a `groups_attribute` key:

```json
{
  "email": "email",
  "first_name": "firstName",
  "last_name": "lastName",
  "groups_attribute": "groups"
}
```

The value of `groups_attribute` is the exact XML attribute name in the assertion. If the key is absent, group extraction is skipped (fail-closed).

#### 19.6.2 JIT Group Resolution

On successful SAML login, after the user record is created or matched (§11 JIT Provisioning):

```typescript
async function resolveJitGroups(
  tenantId: string,
  userId: string,
  samlAttributes: Record<string, string[]>,
  tenantSsoConfig: TenantSsoConfig,
): Promise<void> {
  const groupsAttrKey = tenantSsoConfig.attribute_mapping?.groups_attribute;
  if (!groupsAttrKey) return; // no group extraction configured

  const groupNames: string[] = samlAttributes[groupsAttrKey] ?? [];
  if (groupNames.length === 0) return;

  const jitEnabled = await featureFlag('scim.jit_group_creation_enabled', tenantId);

  for (const groupName of groupNames) {
    let group = await db.tenantGroups.findByDisplayName({ tenantId, displayName: groupName });

    if (!group) {
      if (!jitEnabled) {
        // Group does not exist and JIT creation is disabled — skip silently, log warning
        logger.warn('jit_group_skip', { tenantId, groupName, reason: 'jit_group_creation_disabled' });
        continue;
      }
      // JIT create the group
      group = await db.tenantGroups.create({ tenantId, displayName: groupName, externalId: null });
      await emitAuditEvent('scim.group_created', { tenantId, groupId: group.id, triggeredBy: 'saml_jit' });
    }

    // Upsert membership
    const alreadyMember = await db.scimGroupMembers.exists({ tenantId, groupId: group.id, userId });
    if (!alreadyMember) {
      await db.scimGroupMembers.add({ tenantId, groupId: group.id, userId });
      await emitAuditEvent('scim.group_member_added', {
        tenantId, groupId: group.id, userId, triggeredBy: 'saml_jit',
      });
    }
  }
}
```

#### 19.6.3 JIT Group Creation Feature Flag

`scim.jit_group_creation_enabled` is a **tenant-scoped feature flag**, disabled by default. Enabling it allows FORM to automatically create `tenant_groups` rows based on group names appearing in SAML assertions. This can result in unbounded group creation if the IdP sends large or variable group lists.

**Safety constraints when enabling:**
- Maximum groups per tenant (§OQ-SSO-19.3 default: 500). Attempts to create a 501st group via JIT return a logged warning and are silently skipped — the user still logs in successfully.
- Group names are sanitised: max 255 characters, stripped of leading/trailing whitespace. Names that fail sanitisation are skipped with a warning log.
- The flag may only be enabled by a `tenant_admin` or higher via the admin dashboard (Settings > Identity > Advanced).
- Enabling emits `scim.group_role_mapping_changed` with a note that JIT group creation was activated — auditor visibility.

**When to keep disabled (default):** Tenants using SCIM Group Push (§19.5) should keep JIT group creation disabled to avoid creating duplicate group entries from SAML assertions that may use different group name formats than SCIM `displayName` values.

---

### 19.7 DEC-030 HMAC-Chained Audit Events

All group lifecycle and role mapping events follow the DEC-030 HMAC chain specification (docs/AUDIT_LOG_SCHEMA.md). PII constraint: audit events must contain group IDs and user IDs only — never email addresses, display names (except for `scim.group_created`/`scim.group_deleted` where `displayName` is the group's own name, not a user attribute), or role values in plaintext where not required for audit purpose.

| Event name | Severity | Retention | Trigger | Key payload fields |
|---|---|---|---|---|
| `scim.group_created` | STANDARD | 7 yr | POST /scim/v2/Groups or JIT creation | `tenant_id`, `group_id`, `display_name`, `triggered_by` (scim \| saml_jit \| admin) |
| `scim.group_updated` | STANDARD | 7 yr | PUT /scim/v2/Groups/{id} (when delta exists) | `tenant_id`, `group_id`, `triggered_by` |
| `scim.group_deleted` | HIGH | 7 yr | DELETE /scim/v2/Groups/{id} | `tenant_id`, `group_id`, `display_name`, `member_count_at_deletion`, `triggered_by` |
| `scim.group_member_added` | STANDARD | 7 yr | PATCH op=add or JIT membership upsert | `tenant_id`, `group_id`, `user_id` (scim_user_id), `triggered_by` (scim \| saml_jit \| admin) |
| `scim.group_member_removed` | HIGH | 7 yr | PATCH op=remove or group DELETE cascade | `tenant_id`, `group_id`, `user_id`, `triggered_by` |
| `scim.group_role_mapping_changed` | HIGH | 7 yr | Admin dashboard save, or JIT flag toggle | `tenant_id`, `group_id`, `old_role` (null if new mapping), `new_role` (null if mapping removed), `mapped_by_user_id` |

**Rationale for HIGH severity on removal/deletion events:** Access-reduction events (`scim.group_member_removed`, `scim.group_deleted`) require HIGH severity because they may be used as evidence in offboarding audits, access reviews, or incident investigations where the exact time of access revocation is material. HIGH severity events are surfaced prominently in the admin dashboard audit log viewer and in SOC 2 evidence exports.

**HMAC chain integrity note:** Each event carries the HMAC of the previous event in the chain. Events emitted within the same database transaction (e.g., a PATCH removing 5 members) must be serialised — emit each `scim.group_member_removed` sequentially, not concurrently, to maintain chain integrity. The Worker must await each `emitAuditEvent` call before emitting the next.

---

### 19.8 SOC 2 Evidence Mapping

| SOC 2 Criterion | Requirement | How §19 Satisfies It | Evidence Artefact |
|---|---|---|---|
| CC6.1 | Logical access is restricted to authorised individuals | Group-based access control documented (§19.3), auditable via DEC-030 `scim.group_*` events, `group_member_effective_role` view provides point-in-time access state | `scim_group_role_mappings` table export; `scim.group_role_mapping_changed` audit chain |
| CC6.2 | Access is removed on a timely basis; no auto-escalation | New groups have no role mapping by default (§19.3.3 fail-closed); tenant admin explicit action required before any group confers access | Admin dashboard audit log showing mapping creation date vs. group push date |
| CC6.3 | Policies exist for provisioning and deprovisioning access | SCIM GROUP DELETE cascades role revocation immediately (§19.2.6); `scim.group_member_removed` HIGH event confirms offboarding path | `scim.group_deleted` and `scim.group_member_removed` events in audit chain for terminated user |
| CC6.6 | Logical access controls use identified authorised users | JIT group creation disabled by default; groups only created via authenticated SCIM push or explicit tenant admin action (§19.6.3) | Feature flag audit log; `scim.group_created` triggered_by field |
| CC7.2 | System monitoring detects and responds to unauthorised access | HIGH severity removal events surfaced in admin dashboard and available for SIEM export; role resolution view enables access snapshots for incident review | `group_member_effective_role` view snapshot; HIGH event SIEM integration |
| C1.1 | PII protection in audit events | Audit events carry IDs only; no email addresses or display names beyond group's own `displayName` (§19.7 PII constraint) | Audit log schema review; DEC-030 compliance attestation |

---

### 19.9 Open Questions

| ID | Question | Current Answer / Disposition |
|---|---|---|
| OQ-SSO-19.1 | **Role conflict resolution:** when a user is a member of two groups with conflicting role mappings, should higher or lower privilege win? | Preferred: higher privilege wins. Rationale: §19.3.2. Requires **founder sign-off** before feature ships. If security team prefers lowest-privilege-wins, the `group_member_effective_role` view's `MAX()` logic must be replaced with `MIN()`, and §19.3.2 rationale updated accordingly. |
| OQ-SSO-19.2 | **Nested group support:** should FORM support groups-within-groups (hierarchical group membership)? | Current answer: **no.** SCIM 2.0 (RFC 7643) does not define nested group semantics. IdPs handle nesting inconsistently — Okta flattens membership before pushing; Azure AD may or may not flatten depending on group type. Supporting nesting would require recursive SQL, add query complexity, and create role-resolution ambiguity. If a customer requires nested groups, the recommended workaround is to configure the IdP to push flattened membership to FORM. |
| OQ-SSO-19.3 | **Maximum groups per tenant:** what is the default cap? | Proposed default: **500 groups per tenant.** Configurable per enterprise agreement (field in `tenants.feature_limits` JSONB). Rationale: 500 covers the largest expected enterprise deployment (a 10,000-person org with granular team groups); above 500 suggests the customer may be pushing all IdP groups rather than FORM-specific groups, which indicates a misconfigured push filter. Requires **enterprise-architect decision** and confirmation that the `tenant_groups` table index performance is acceptable at 500 rows (trivially yes for B-tree). |

---

### 19.10 Implementation Checklist

| Task | Owner | Priority | Milestone |
|---|---|---|---|
| Write `migrations/YYYYMMDD_scim_group_role_mappings.sql` — `scim_group_role_mappings` DDL, indexes, RLS policies (§19.4.2) | platform-engineer | **P0** | M4 |
| Write `migrations/YYYYMMDD_scim_group_members.sql` — `scim_group_members` DDL, indexes, RLS policies (§19.4.3) | platform-engineer | **P0** | M4 |
| Write `migrations/YYYYMMDD_group_member_effective_role_view.sql` — `group_member_effective_role` view (§19.4.4); include test query validating direct-beats-group and highest-group-wins cases | platform-engineer + enterprise-architect | **P0** | M4 |
| Implement `GET`, `POST`, `PUT`, `PATCH`, `DELETE` on `/scim/v2/Groups` in Cloudflare Worker (§19.2); enforce tenant isolation guard in PATCH `op=add` (`assertUsersInTenant`) | platform-engineer | **P0** | M4 |
| Wire all 6 DEC-030 audit events (§19.7) with correct severity levels; validate HMAC chain ordering for batch PATCH operations (sequential emission) | platform-engineer + security-engineer | **P0** | M4 |
| Implement `resolveJitGroups` function (§19.6.2) in SAML JIT login path; gate on `scim.jit_group_creation_enabled` feature flag; enforce 500-group cap | platform-engineer | **P0** | M4 |
| Get founder sign-off on OQ-SSO-19.1 (higher-privilege-wins) and record decision in `docs/DECISION_LOG.md` | enterprise-architect | **P0** | M4 |
| Build admin dashboard "Group Role Mapping" panel (§19.3.3): group list, role selector, save-per-row, warning banner for owner/admin mapping | platform-engineer | **P1** | M5 |
| Write Okta Push Groups configuration guide for CS team (§19.5.1); include Okta filter expression recommendation `name sw "FORM-"` and expected SCIM call sequence | enterprise-architect | **P1** | M5 |
| Add `scim_group_role_mappings` and `scim_group_members` to Art. 17 erasure scope: confirm FK CASCADE handles tenant hard-delete; add erasure smoke test to CI suite | platform-engineer | **P1** | M5 |

---

---

## 20. SAML Certificate Lifecycle Management — Proactive Monitoring & Expiry Alerting

> Owner: `enterprise-architect` + `security-engineer` + `devops-lead`. Review: quarterly, and on any certificate event.
> References: §6.1 (certificate concepts), §8.1 (emergency SP cert rotation runbook), DEC-030, docs/AUDIT_LOG_SCHEMA.md, docs/OBSERVABILITY.md §6 (alerting rules), docs/INCIDENT_RESPONSE.md R-04 (SSO compromise runbook).

### 20.1 Purpose and SOC 2 Mapping

Section §8.1 references an automated `cert_expiry_check` cron that "triggers at 60 days before expiry." This section designs that cron and the full lifecycle management system around it.

**Two certificate classes require monitoring:**

| Class | Owner | Location in `tenant_sso_configs` | Typical Validity | Risk on Expiry |
|---|---|---|---|---|
| **SP signing certificate** | FORM | `saml_sp_certificate` (BYTEA, encrypted) | 2 years (RSA-2048, generated by FORM) | FORM's assertions become unverifiable by IdP → SSO hard-broken for all users of that tenant |
| **IdP signing certificate** | Customer's IT | `saml_idp_certificate` (BYTEA, encrypted) | 1–3 years (IdP-generated; varies by vendor) | FORM cannot verify IdP assertions → SSO hard-broken for all users of that tenant |

Both classes can expire silently if not monitored. IdP certificate expiry is the more common failure mode in production — customers rotate IdP certificates as part of their own credential hygiene and occasionally forget to update the certificate in FORM's configuration.

**SOC 2 criteria this section addresses:**

| Criterion | Requirement | Coverage |
|---|---|---|
| CC6.1 | Credentials (including X.509 certificates) have a defined lifecycle including rotation and revocation | Schema columns track expiry; cron monitors and alerts before expiry; DEC-030 events provide tamper-evident lifecycle record |
| CC7.2 | Entity monitors for anomalous conditions including credential approaching expiry | `cert-expiry-check` Worker cron runs daily; PagerDuty routing defined per tier; alert history in `audit_log` as SOC 2 evidence |
| CC8.1 | Change management controls apply to certificate rotation | Rotation procedure (§8.1) is logged via `sso.cert_rotated` event; this section adds pre-rotation alert events to complete the lifecycle record |

---

### 20.2 Schema Extension — Certificate Metadata Columns

Add expiry metadata as plaintext columns on `tenant_sso_configs`. The actual certificate PEM remains encrypted in `saml_sp_certificate` / `saml_idp_certificate` BYTEA columns (§4.2 schema). These metadata columns are computed at upload time and read by the monitoring cron without decryption.

```sql
-- Migration: YYYYMMDD_sso_cert_lifecycle_metadata.sql

ALTER TABLE tenant_sso_configs
  -- SP certificate metadata (FORM-generated, populated at cert generation and each rotation)
  ADD COLUMN IF NOT EXISTS saml_sp_cert_expires_at       TIMESTAMPTZ,
  ADD COLUMN IF NOT EXISTS saml_sp_cert_fingerprint      TEXT,           -- SHA-256 hex, last 8 chars for display
  ADD COLUMN IF NOT EXISTS saml_sp_cert_issued_at        TIMESTAMPTZ,

  -- IdP certificate metadata (customer-provided, parsed from PEM at upload)
  ADD COLUMN IF NOT EXISTS saml_idp_cert_expires_at      TIMESTAMPTZ,
  ADD COLUMN IF NOT EXISTS saml_idp_cert_fingerprint     TEXT,
  ADD COLUMN IF NOT EXISTS saml_idp_cert_uploaded_at     TIMESTAMPTZ,

  -- Alert state machine (per-tenant, covers whichever class expires sooner)
  ADD COLUMN IF NOT EXISTS cert_alert_tier               TEXT DEFAULT 'ok'
    CHECK (cert_alert_tier IN ('ok', 't90', 't60', 't30', 't14', 't7', 't2', 'expired')),
  ADD COLUMN IF NOT EXISTS cert_alert_last_sent_at       TIMESTAMPTZ,
  ADD COLUMN IF NOT EXISTS cert_rotation_state           TEXT DEFAULT 'idle'
    CHECK (cert_rotation_state IN ('idle', 'notified', 'rotation_in_progress', 'overdue', 'complete'));

-- Index for the monitoring cron — filters active tenants with SAML by expiry date
CREATE INDEX IF NOT EXISTS idx_sso_configs_sp_cert_expiry
  ON tenant_sso_configs (saml_sp_cert_expires_at)
  WHERE is_active = true AND sso_type = 'saml';

CREATE INDEX IF NOT EXISTS idx_sso_configs_idp_cert_expiry
  ON tenant_sso_configs (saml_idp_cert_expires_at)
  WHERE is_active = true AND sso_type = 'saml';
```

**Backfill procedure for existing tenants:** On migration deploy, a one-shot Worker script parses the `saml_sp_certificate` and `saml_idp_certificate` BYTEA fields using the Web Crypto API (`crypto.subtle.importKey` → X.509 `validity.notAfter`), writes the expiry timestamps and SHA-256 fingerprints back to the new columns. Run as a Supabase Edge Function with service-role key, log result counts.

---

### 20.3 Alert Escalation Matrix

The monitoring cron evaluates both SP and IdP certificate expiry independently. Alerts fire on the first occurrence each day a cert is within a threshold window.

| Tier | Days Remaining | Severity | Notification Target | DEC-030 Event | Action Required |
|---|---|---|---|---|---|
| `t90` | ≤ 90 days | INFO | FORM `#security-alerts` Slack (ops only) | `sso.cert_expiry_alert` MEDIUM | Note: initiate rotation planning. Schedule rotation window with customer CSM for SP cert; notify customer IT for IdP cert. |
| `t60` | ≤ 60 days | P3 | FORM `#security-alerts` + PagerDuty LOW + CSM email | `sso.cert_expiry_alert` MEDIUM | **SP cert:** begin §8.1 rotation procedure. **IdP cert:** CSM opens Slack thread with customer IT contact. |
| `t30` | ≤ 30 days | P2 | PagerDuty MEDIUM + CSM email + tenant admin in-app banner | `sso.cert_expiry_alert` HIGH | CSM confirms rotation is scheduled. If customer unresponsive: escalate to founder. |
| `t14` | ≤ 14 days | P1 | PagerDuty HIGH + tenant admin email (CRT-02) + CSM phone | `sso.cert_expiry_alert` HIGH | Rotation must be complete this week. CSM calls customer. If SP cert: §8.1 Day 0 must have already started. |
| `t7` | ≤ 7 days | P1 | PagerDuty HIGH + tenant admin email (CRT-03) + founder | `sso.cert_expiry_alert` CRITICAL | Emergency rotation. For IdP cert: customer sends new PEM or updated metadata URL immediately. |
| `t2` | ≤ 2 days | P0 | PagerDuty CRITICAL + founder + tenant admin phone | `sso.cert_expiry_alert` CRITICAL | Follow INCIDENT_RESPONSE.md R-04 immediately. Enable email-magic-link fallback (§8.3). |
| `expired` | 0 or past | P0 | PagerDuty CRITICAL + R-04 incident opened | `sso.cert_expired` CRITICAL | P0 incident. All users of this tenant cannot log in via SSO. Email fallback must be active. |

**De-duplication rule:** once an alert tier fires for a given `(tenant_id, cert_class)` pair, the cron does not re-fire that same tier until 7 days have elapsed — prevents alert fatigue while ensuring weekly repetition during the critical window. `cert_alert_last_sent_at` enforces this.

---

### 20.4 `cert-expiry-check` Worker Cron

**File:** `apps/api-gateway/src/crons/cert-expiry-check.ts`
**Schedule:** daily at `02:00 UTC` (`0 2 * * *` in `wrangler.toml`)

```typescript
// Cron handler — registered in wrangler.toml under [triggers] crons
import { createClient, SupabaseClient } from '@supabase/supabase-js'
import { emitAuditEvent, AuditSeverity } from '../audit/dec030'
import { sendPagerDutyAlert } from '../integrations/pagerduty'
import { sendAlertEmail } from '../integrations/resend'

// Days thresholds in ascending order of urgency
const THRESHOLDS = [
  { tier: 't90',     daysMax: 90,  daysMin: 61,  severity: 'MEDIUM'   as AuditSeverity, pdSeverity: null },
  { tier: 't60',     daysMax: 60,  daysMin: 31,  severity: 'MEDIUM'   as AuditSeverity, pdSeverity: 'low' },
  { tier: 't30',     daysMax: 30,  daysMin: 15,  severity: 'HIGH'     as AuditSeverity, pdSeverity: 'warning' },
  { tier: 't14',     daysMax: 14,  daysMin: 8,   severity: 'HIGH'     as AuditSeverity, pdSeverity: 'error' },
  { tier: 't7',      daysMax: 7,   daysMin: 3,   severity: 'CRITICAL' as AuditSeverity, pdSeverity: 'critical' },
  { tier: 't2',      daysMax: 2,   daysMin: 0,   severity: 'CRITICAL' as AuditSeverity, pdSeverity: 'critical' },
] as const

export async function runCertExpiryCheck(env: Env): Promise<void> {
  const db = createClient(env.SUPABASE_URL, env.SUPABASE_SERVICE_ROLE_KEY)
  const now = new Date()
  const window90 = new Date(now.getTime() + 90 * 24 * 60 * 60 * 1000)

  // Fetch all active SAML tenants with a cert expiring within 90 days
  const { data: configs, error } = await db
    .from('tenant_sso_configs')
    .select(`
      id, tenant_id, is_active,
      saml_sp_cert_expires_at, saml_sp_cert_fingerprint,
      saml_idp_cert_expires_at, saml_idp_cert_fingerprint,
      cert_alert_tier, cert_alert_last_sent_at,
      tenants ( slug, status, owner_email )
    `)
    .eq('is_active', true)
    .eq('sso_type', 'saml')
    .or(
      `saml_sp_cert_expires_at.lte.${window90.toISOString()},` +
      `saml_idp_cert_expires_at.lte.${window90.toISOString()}`
    )

  if (error) {
    await emitAuditEvent(env, {
      event_type: 'sso.cert_monitor_error',
      severity: 'HIGH',
      actor_type: 'system',
      metadata: { error: error.message }
    })
    throw error
  }

  for (const cfg of (configs ?? [])) {
    for (const certClass of ['sp', 'idp'] as const) {
      const expiresAt = certClass === 'sp'
        ? cfg.saml_sp_cert_expires_at
        : cfg.saml_idp_cert_expires_at
      const fingerprint = certClass === 'sp'
        ? cfg.saml_sp_cert_fingerprint
        : cfg.saml_idp_cert_fingerprint

      if (!expiresAt) continue

      const msRemaining = new Date(expiresAt).getTime() - now.getTime()
      const daysRemaining = Math.floor(msRemaining / (24 * 60 * 60 * 1000))

      if (daysRemaining < 0) {
        // Certificate already expired — P0 incident
        await handleExpired(env, db, cfg, certClass, fingerprint, expiresAt)
        continue
      }

      const tier = THRESHOLDS.find(t => daysRemaining <= t.daysMax && daysRemaining >= t.daysMin)
      if (!tier) continue // > 90 days remaining — no action

      // De-duplicate: skip if same tier fired within last 7 days
      const lastSent = cfg.cert_alert_last_sent_at
        ? new Date(cfg.cert_alert_last_sent_at)
        : null
      const daysSinceLast = lastSent
        ? (now.getTime() - lastSent.getTime()) / (24 * 60 * 60 * 1000)
        : Infinity
      if (cfg.cert_alert_tier === tier.tier && daysSinceLast < 7) continue

      await fireAlert(env, db, cfg, certClass, fingerprint, expiresAt, daysRemaining, tier)
    }
  }
}

async function fireAlert(
  env: Env,
  db: SupabaseClient,
  cfg: any,
  certClass: 'sp' | 'idp',
  fingerprint: string | null,
  expiresAt: string,
  daysRemaining: number,
  tier: typeof THRESHOLDS[number]
): Promise<void> {
  const meta = {
    tenant_id:       cfg.tenant_id,
    tenant_slug:     cfg.tenants?.slug,
    cert_class:      certClass,         // 'sp' or 'idp'
    fingerprint_sha256: fingerprint,
    expires_at:      expiresAt,
    days_remaining:  daysRemaining,
    alert_tier:      tier.tier,
  }

  await emitAuditEvent(env, {
    event_type: 'sso.cert_expiry_alert',
    severity:   tier.severity,
    actor_type: 'system',
    resource_type: 'sso_certificate',
    resource_id:   cfg.id,
    metadata:      meta,
  })

  if (tier.pdSeverity) {
    await sendPagerDutyAlert(env, {
      summary: `FORM SSO cert expiry — ${cfg.tenants?.slug} ${certClass.toUpperCase()} cert expires in ${daysRemaining}d`,
      severity: tier.pdSeverity,
      routing_key: env.PAGERDUTY_ROUTING_KEY_SECURITY,
      custom_details: meta,
    })
  }

  // Email to tenant admin for t30 and tighter
  if (['t30', 't14', 't7', 't2'].includes(tier.tier)) {
    await sendAlertEmail(env, {
      to:       cfg.tenants?.owner_email,
      template: `sso-cert-expiry-${tier.tier}` as const,
      data:     { ...meta, tenant_name: cfg.tenants?.slug },
    })
  }

  await db
    .from('tenant_sso_configs')
    .update({ cert_alert_tier: tier.tier, cert_alert_last_sent_at: new Date().toISOString() })
    .eq('id', cfg.id)
}

async function handleExpired(
  env: Env,
  db: SupabaseClient,
  cfg: any,
  certClass: 'sp' | 'idp',
  fingerprint: string | null,
  expiresAt: string
): Promise<void> {
  const meta = {
    tenant_id:      cfg.tenant_id,
    tenant_slug:    cfg.tenants?.slug,
    cert_class:     certClass,
    fingerprint_sha256: fingerprint,
    expires_at:     expiresAt,
    days_overdue:   Math.abs(Math.floor(
      (new Date(expiresAt).getTime() - Date.now()) / (24 * 60 * 60 * 1000)
    )),
  }

  await emitAuditEvent(env, {
    event_type: 'sso.cert_expired',
    severity:   'CRITICAL',
    actor_type: 'system',
    resource_type: 'sso_certificate',
    resource_id:   cfg.id,
    metadata:      meta,
  })

  await sendPagerDutyAlert(env, {
    summary: `P0 — FORM SSO cert EXPIRED — ${cfg.tenants?.slug} ${certClass.toUpperCase()} cert expired`,
    severity: 'critical',
    routing_key: env.PAGERDUTY_ROUTING_KEY_SECURITY,
    custom_details: meta,
  })

  await db
    .from('tenant_sso_configs')
    .update({ cert_alert_tier: 'expired', cert_rotation_state: 'overdue' })
    .eq('id', cfg.id)
}
```

**`wrangler.toml` trigger registration:**

```toml
[triggers]
crons = [
  "0 2 * * *",   # cert-expiry-check — daily 02:00 UTC
  # ... other crons
]
```

The `scheduled` handler dispatches by cron expression:

```typescript
export default {
  async scheduled(event: ScheduledEvent, env: Env): Promise<void> {
    if (event.cron === '0 2 * * *') {
      await runCertExpiryCheck(env)
    }
  }
}
```

---

### 20.5 Customer Notification Templates

These templates are sent via Resend. Template IDs match the `sendAlertEmail` call in §20.4. Tone follows brand-voice guidelines — direct, non-alarming at T-30, urgent at T-7.

**Template `sso-cert-expiry-t30` — CRT-01 (T-30 alert, first customer-visible notification)**

```
Subject: Action required — FORM SSO certificate expires in 30 days · {{tenant_name}}

Hi {{tenant_admin_name}},

Your FORM SSO configuration includes a {{cert_class_label}} certificate that
expires on {{expires_at_formatted}} ({{days_remaining}} days from today).

Certificate:  {{cert_class_label}}
Fingerprint:  ...{{fingerprint_last8}}
Expires:      {{expires_at_formatted}} UTC

{{#if cert_class_is_idp}}
To prevent SSO disruption for your team:

1. Log in to your identity provider admin console (Okta / Azure AD / Google
   Workspace).
2. Locate the FORM application and generate or export a new X.509 signing
   certificate.
3. Send the new certificate PEM to your FORM CSM, or upload it via your
   FORM admin dashboard → SSO Settings → IdP Certificate.

Your CSM, {{csm_name}}, will follow up this week to coordinate the update.
{{/if}}
{{#if cert_class_is_sp}}
FORM will rotate our signing certificate on your behalf. Your CSM,
{{csm_name}}, will contact your IT team to schedule re-import of FORM's
updated SP metadata. Zero downtime is guaranteed during the rotation
(both old and new certificates remain valid during the transition window).
{{/if}}

Rotation guide: https://form.coach/docs/sso-certificate-rotation

Questions? Reply to this email or contact {{csm_email}}.

FORM Enterprise · enterprise@form.coach
```

**Template `sso-cert-expiry-t7` — CRT-02 (T-7 alert, urgent)**

```
Subject: Urgent — FORM SSO certificate expires in {{days_remaining}} days · {{tenant_name}}

Hi {{tenant_admin_name}},

This is an urgent reminder that your FORM SSO {{cert_class_label}} certificate
expires in {{days_remaining}} days ({{expires_at_formatted}} UTC).

If this certificate expires without rotation, all employees at {{tenant_name}}
will be unable to log in via SSO. Email-based access will remain available
as a fallback.

{{cert_class_action_block}}

Please respond to this email or contact {{csm_name}} directly at
{{csm_phone}} today.

FORM Enterprise · enterprise@form.coach
```

**Template `sso-cert-expiry-expired` — CRT-03 (Post-expiry, email fallback active)**

```
Subject: FORM SSO is temporarily unavailable — action required · {{tenant_name}}

Hi {{tenant_admin_name}},

The SSO {{cert_class_label}} certificate for your FORM tenant expired on
{{expires_at_formatted}} UTC. SSO login is currently unavailable.

Immediate action taken by FORM:
- Email magic-link login has been activated for all users in your tenant.
  Employees can continue to access FORM using their work email address.

To restore SSO, please {{cert_class_restore_instructions}}.

{{csm_name}} is attempting to reach you by phone. Priority response
required — our on-call engineer is standing by.

FORM Enterprise · enterprise@form.coach
```

---

### 20.6 Rotation State Machine

The `cert_rotation_state` column on `tenant_sso_configs` tracks coordination state for the current rotation cycle. Transitions:

```
idle
  │  (alert tier reaches t60)
  ▼
notified
  │  (customer acknowledges / CSM opens rotation ticket)
  ▼
rotation_in_progress
  │  (new cert uploaded; §8.1 Day 0 started)
  ▼
complete  ──── (sso.cert_rotated DEC-030 event emitted; reset to idle on next cert cycle)
  
notified ──── (no customer response by t7)
  │
  ▼
overdue  ──── (escalate to founder; R-04 pre-positioning)
  │  (cert expires)
  ▼
expired  (P0 incident; email fallback activated per §8.3)
```

State transitions are written by:
- The monitoring cron (`cert-expiry-check`) advancing from `idle` to `notified` on first CSM notification email send
- The CSM or admin dashboard advancing from `notified` to `rotation_in_progress` when confirming the rotation window
- The cert rotation completion code in §8.1 advancing to `complete`
- The cron advancing from `notified` to `overdue` if `cert_alert_tier = 't7'` and `cert_rotation_state = 'notified'`

The state machine is intentionally simple — it is a coordination aid, not an enforcement gate. SSO does not break because of state machine state; it breaks because of actual certificate expiry.

---

### 20.7 DEC-030 HMAC-Chained Audit Events

The following events are added to the event registry defined in §6.5 and `docs/AUDIT_LOG_SCHEMA.md`. All are emitted with HMAC chain continuity.

| Event | Trigger | Severity | Retention | Key `metadata` fields |
|---|---|---|---|---|
| `sso.cert_expiry_alert` | Daily cron detects cert within a threshold window | MEDIUM (t90/t60) · HIGH (t30/t14) · CRITICAL (t7/t2) | 7 years | `tenant_id`, `cert_class` ('sp'\|'idp'), `fingerprint_sha256`, `expires_at`, `days_remaining`, `alert_tier` |
| `sso.cert_expired` | Cron detects cert past expiry date without rotation | CRITICAL | 7 years | `tenant_id`, `cert_class`, `fingerprint_sha256`, `expires_at`, `days_overdue` |
| `sso.cert_uploaded` | New IdP certificate PEM provided by customer (distinct from SP rotation) | STANDARD | 7 years | `tenant_id`, `cert_class: 'idp'`, `new_fingerprint_sha256`, `new_expires_at`, `uploaded_by` |
| `sso.cert_monitor_error` | Cron fails to query or parse certificate data | HIGH | 7 years | `error` (message only — no PEM or key material) |

Note: `sso.cert_rotated` (existing event, §6.5) covers SP certificate rotation completion and is unchanged. The new events above cover the monitoring and upload phases that precede rotation.

**Privacy constraint:** No certificate PEM, private key material, or customer IdP metadata is included in any audit event. `fingerprint_sha256` is the public SHA-256 fingerprint of the certificate's DER encoding (standard X.509 identifier, not secret).

---

### 20.8 Admin Dashboard Certificate Panel

Visible to FORM internal admins only (not tenant admins). Located in the per-tenant detail view under **SSO & SCIM → Certificates**.

| Field | Source | Display |
|---|---|---|
| SP certificate | `saml_sp_cert_expires_at`, `saml_sp_cert_fingerprint` | Expiry date + days remaining; fingerprint last 8 hex chars; status badge (green / amber / red / expired) |
| IdP certificate | `saml_idp_cert_expires_at`, `saml_idp_cert_fingerprint` | Same as above |
| Alert tier | `cert_alert_tier` | Current tier label (`t90`, `t30`, etc.) |
| Rotation state | `cert_rotation_state` | `idle` / `notified` / `rotation_in_progress` / `overdue` / `complete` |
| Last alert sent | `cert_alert_last_sent_at` | Relative timestamp ("3 days ago") |

**Tenant admin view:** A non-dismissible banner appears in the tenant admin dashboard when `saml_idp_cert_expires_at` is ≤ 30 days out. Banner text: *"Your SSO identity provider certificate expires on [date]. Please update it in your IdP admin console and provide the new certificate to your FORM CSM."* Banner links to the self-service IdP certificate upload UI (§20.9 open question OQ-SSO-20.3).

FORM ops do not surface SP certificate expiry warnings to the tenant admin — SP cert rotation is handled by FORM on the customer's behalf.

---

### 20.9 SOC 2 Evidence Mapping

| Control | Criterion text (abbreviated) | Coverage by §20 |
|---|---|---|
| CC6.1 | Logical access credentials are managed through their full lifecycle | Certificate inventory (schema columns), expiry tracking, and rotation state provide a complete lifecycle record. Evidence: DEC-030 `sso.cert_expiry_alert` and `sso.cert_rotated` events; `cert_rotation_state` audit trail |
| CC7.2 | Entity monitors for anomalies including credential expiry | `cert-expiry-check` cron runs daily at 02:00 UTC; PagerDuty routing table provides verifiable escalation; alert history in `audit_log` |
| CC8.1 | Changes to systems are authorized and controlled | Certificate rotation follows §8.1 change procedure; the `cert_rotation_state` state machine ensures coordination before activation |

**Evidence artefacts for SOC 2 auditor:**

| Artefact ID | Description | Collection method |
|---|---|---|
| CC6-E-CERT-001 | `audit_log` export: all `sso.cert_expiry_alert` events for the observation period | SQL: `SELECT * FROM audit_log WHERE event_type = 'sso.cert_expiry_alert' AND created_at BETWEEN $obs_start AND $obs_end ORDER BY created_at` |
| CC6-E-CERT-002 | `audit_log` export: all `sso.cert_rotated` events for the observation period | Same query with `event_type = 'sso.cert_rotated'` |
| CC6-E-CERT-003 | `tenant_sso_configs` certificate column snapshot (expiry timestamps + fingerprints only — no PEM) | `SELECT tenant_id, saml_sp_cert_expires_at, saml_sp_cert_fingerprint, saml_idp_cert_expires_at, saml_idp_cert_fingerprint, cert_rotation_state FROM tenant_sso_configs WHERE is_active = true` — run at start and end of observation period |
| CC6-E-CERT-004 | Cron execution log: Cloudflare Workers Tail log or R2-archived daily summary confirming `cert-expiry-check` ran on schedule | Cloudflare dashboard → Workers & Pages → cert-expiry-check → Logs |

---

### 20.10 Open Questions

| ID | Question | Disposition |
|---|---|---|
| OQ-SSO-20.1 | Should FORM support automatic SP certificate push to customer IdP via SAML metadata URL auto-refresh? | Partially supported — FORM's SP metadata URL at `/auth/saml/{tenant_id}/metadata` dynamically reflects the current SP cert. Customers who configure their IdP to consume this URL (recommended in onboarding §7.1) get automatic SP cert updates. However, some IdPs (older Okta configurations, PingFederate) do not support metadata URL auto-refresh. For those: manual re-import required. Target: document IdP-by-IdP metadata refresh behaviour in §2.x subsections. |
| OQ-SSO-20.2 | Should FORM accept self-signed IdP certificates or require CA-signed? | Self-signed accepted (common practice for enterprise SAML). CA-signed preferred for OCSP validation. No enforcement gate — the certificate is validated for structure and expiry only, not CA chain. Risk documented in §6.1. |
| OQ-SSO-20.3 | Should the tenant admin dashboard include a self-service IdP certificate upload UI? | Yes, but gated on M5. Today: CSM-mediated upload only. Self-service upload reduces support burden and enables faster customer response to expiry alerts. Requires admin dashboard build (§16 implementation checklist). |
| OQ-SSO-20.4 | At what SP certificate validity period should FORM generate? Currently 2 years (§6.1). | 2 years is standard. Google Workspace and Azure AD both support up to 3 years. Okta recommends ≤ 2 years. Keep at 2 years unless a major IdP requires longer. Review at M6. |

---

### 20.11 Implementation Checklist

| # | Action | Owner | Priority | Milestone |
|---|---|---|---|---|
| 1 | Write `migrations/YYYYMMDD_sso_cert_lifecycle_metadata.sql` — ALTER TABLE `tenant_sso_configs` with eight new columns per §20.2; add two partial indexes | platform-engineer | **P0** | M4 |
| 2 | Write backfill Edge Function: parse `saml_sp_certificate` and `saml_idp_certificate` BYTEA fields, write expiry timestamps and fingerprints to new columns for all active SAML tenants | platform-engineer | **P0** | M4 |
| 3 | Implement `cert-expiry-check` Worker cron (`apps/api-gateway/src/crons/cert-expiry-check.ts`) per §20.4; register in `wrangler.toml` at `0 2 * * *` | platform-engineer | **P0** | M4 |
| 4 | Wire `sso.cert_expiry_alert`, `sso.cert_expired`, `sso.cert_uploaded`, `sso.cert_monitor_error` DEC-030 events (§20.7) into the `emitAuditEvent` registry; validate HMAC chain ordering | platform-engineer + security-engineer | **P0** | M4 |
| 5 | Configure PagerDuty routing rules per §20.3 alert tier table: LOW/WARNING/ERROR/CRITICAL routing to `#security-alerts` escalation path | devops-lead | **P0** | M4 |
| 6 | Add `sso-cert-expiry-t30`, `sso-cert-expiry-t7`, `sso-cert-expiry-expired` email templates (CRT-01 through CRT-03) to Resend template library per §20.5 | brand-voice | **P1** | M4 |
| 7 | Update SP certificate generation code to write `saml_sp_cert_expires_at` and `saml_sp_cert_fingerprint` at generation time and on each rotation | platform-engineer | **P0** | M4 |
| 8 | Update IdP certificate upload path (CSM-mediated and future self-service) to parse PEM and write `saml_idp_cert_expires_at` and `saml_idp_cert_fingerprint` | platform-engineer | **P0** | M4 |
| 9 | Build FORM internal admin dashboard Certificate Panel (§20.8): SP cert status, IdP cert status, alert tier, rotation state badges | platform-engineer | **P1** | M5 |
| 10 | Add tenant admin in-app banner (§20.8 end): show when IdP cert ≤ 30 days from expiry; link to CSM contact | platform-engineer | **P1** | M5 |
| 11 | Verify first cron execution: confirm `sso.cert_expiry_alert` events appear in `audit_log` with correct severity; confirm HMAC chain is intact; archive cron log to R2 as CC6-E-CERT-004 | devops-lead + compliance-officer | **P0** | M4 |

---

*v1.1 additions (2026-05-30): Section 19 — SCIM Groups Sync & Group-Based Role Mapping. Closes the group lifecycle gap left by §5–§8 (which cover user provisioning and direct role assignment but not group objects as first-class SCIM resources). Six new SCIM endpoints on the Cloudflare Workers edge runtime: GET /Groups (paginated, filterable, RFC 7644 ListResponse), GET /Groups/{id} (single resource with members array), POST /Groups (creates `tenant_groups` row, 201 + Location header, idempotency on externalId), PUT /Groups/{id} (full replace, delta-aware audit emission), PATCH /Groups/{id} (RFC 7644 PatchOp, op=add/remove on path=members, with cross-tenant guard on member userIds), DELETE /Groups/{id} (cascade role revocation, no user deletion). Group-to-role mapping model: new `scim_group_role_mappings` table (tenant_id, scim_group_id FK, form_role ENUM, mapped_by, mapped_at; UNIQUE constraint one role per group per tenant; RLS: tenant_admin read/write own tenant, form_system read-only, form_api zero-rows fail-closed). Role resolution priority: explicit direct assignment beats group membership; among group memberships highest privilege wins (highest-privilege-wins rationale documented in §19.3.2; requires founder sign-off per OQ-SSO-19.1). New `scim_group_members` table for SCIM-managed many-to-many membership. New `group_member_effective_role` view computing final FORM role per user via FULL OUTER JOIN of direct roles and group-derived roles using CASE ordinals and MAX aggregation. IdP push group configuration: Okta Push Groups (filter expression, memberOf, PUT-then-PATCH behaviour), Azure AD / Entra SCIM mappings (objectId → externalId, scope filter, 40-minute sync lag caveat, GET by externalId query pattern), Google Workspace (no native SCIM Groups Push; three workarounds: Okta bridge, Entra bridge, SAML groups attribute). SAML JIT groups path: `groups_attribute` key in `tenant_sso_configs.attribute_mapping` JSONB; `resolveJitGroups` function; `scim.jit_group_creation_enabled` feature flag (disabled by default; safety: 500-group cap, 255-char name limit, admin-only toggle). Six DEC-030 HMAC-chained audit events: scim.group_created (STANDARD, 7yr), scim.group_updated (STANDARD, 7yr), scim.group_deleted (HIGH, 7yr), scim.group_member_added (STANDARD, 7yr), scim.group_member_removed (HIGH, 7yr), scim.group_role_mapping_changed (HIGH, 7yr); sequential emission required for batch PATCH to maintain HMAC chain integrity. SOC 2 mapping: CC6.1 (group access auditable), CC6.2 (no-mapping-by-default fail-closed), CC6.3 (DELETE cascade offboarding), CC6.6 (JIT flag gated), CC7.2 (HIGH events for SIEM), C1.1 (IDs only in audit events). Open questions: OQ-SSO-19.1 (higher vs. lower privilege conflict resolution — founder sign-off required), OQ-SSO-19.2 (nested groups — no, not supported), OQ-SSO-19.3 (500 group cap — enterprise-architect decision). Implementation checklist: 10 tasks (7× P0, 3× P1), M4/M5.*

*v1.2 additions (2026-05-30): Section 20 — SAML Certificate Lifecycle Management — Proactive Monitoring & Expiry Alerting. Closes the design gap for the `cert_expiry_check` cron referenced (but not specified) in §8.1. Covers both SP certificate (FORM-generated, 2-year RSA-2048) and IdP certificate (customer-generated, validity varies) expiry monitoring. Eight ALTER TABLE columns on `tenant_sso_configs`: four expiry-metadata columns (`saml_sp_cert_expires_at`, `saml_sp_cert_fingerprint`, `saml_idp_cert_expires_at`, `saml_idp_cert_fingerprint`), two partial indexes on those columns, and two state-machine columns (`cert_alert_tier` ENUM 7-value, `cert_rotation_state` ENUM 5-value) with backfill Edge Function procedure. Seven-tier alert escalation matrix (t90 through expired) with PagerDuty severity mapping (null → low → warning → error → critical) and de-duplication rule (7-day cooldown per tier per tenant). Full TypeScript Worker cron implementation (`cert-expiry-check.ts`): daily 02:00 UTC via Cloudflare Workers Cron; queries active SAML tenants with cert expiring ≤90 days; handles both cert classes in one pass; emits DEC-030 before PagerDuty; updates `cert_alert_tier` and `cert_alert_last_sent_at` after each alert; separate `handleExpired` path for P0 post-expiry case. Three Resend email templates: CRT-01 (`sso-cert-expiry-t30`, first customer-visible notification with cert-class-conditional body), CRT-02 (`sso-cert-expiry-t7`, urgent 7-day), CRT-03 (`sso-cert-expiry-expired`, post-expiry email-fallback active notification). Five-state rotation state machine (`idle → notified → rotation_in_progress → complete`; `overdue` branch from `notified` at t7 if unresponsive). Four new DEC-030 HMAC-chained events: `sso.cert_expiry_alert` (MEDIUM/HIGH/CRITICAL by tier, 7yr), `sso.cert_expired` (CRITICAL, 7yr), `sso.cert_uploaded` (STANDARD, 7yr — IdP cert upload distinct from rotation), `sso.cert_monitor_error` (HIGH, 7yr — cron self-monitoring). FORM internal admin dashboard Certificate Panel spec: SP + IdP status badges, alert tier, rotation state. Tenant admin in-app banner at ≤30 days IdP cert expiry. SOC 2 evidence: CC6-E-CERT-001 through CC6-E-CERT-004 (audit_log exports, config snapshot, cron execution log). SOC 2 mapping: CC6.1 (credential lifecycle), CC7.2 (anomaly monitoring), CC8.1 (change management). 4 open questions (OQ-SSO-20.1 through OQ-SSO-20.4: metadata auto-refresh, self-signed cert policy, self-service upload UI, 2yr validity review). 11-item implementation checklist (8× P0 M4, 3× P1 M5).*
