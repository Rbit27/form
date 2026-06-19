# FORM · SSO/SCIM Implementation v2.5

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
21. [Google Workspace Directory API Integration for Group Sync](#21-google-workspace-directory-api-integration-for-group-sync)
22. [High-Scale Session Revocation Architecture — KV-Backed Revocation Cache](#22-high-scale-session-revocation-architecture--kv-backed-revocation-cache)
23. [Continuous Access Evaluation (CAE) & Shared Signals Framework (SSF)](#23-continuous-access-evaluation-cae--shared-signals-framework-ssf--real-time-idp-risk-event-processing)
24. [Privileged Access Management (PAM): Just-in-Time Privilege Escalation & Break-Glass Protocol](#24-privileged-access-management-pam-just-in-time-privilege-escalation--break-glass-protocol-for-form_admin-operations)
25. [Per-Tenant Authentication Policy Engine — IP Allowlist, MFA Enforcement & Session Policy](#25-per-tenant-authentication-policy-engine--ip-allowlist-mfa-enforcement--session-policy)
26. [API Key Authentication Security — SCIM IP Scope, API Key IP Enforcement & Rotation Policy](#26-api-key-authentication-security--scim-ip-scope-api-key-ip-enforcement--rotation-policy)
27. [SCIM v2.0 Endpoint — Worker Implementation Design (Closes G-001)](#27-scim-v20-endpoint--worker-implementation-design-closes-g-001)
28. [SCIM Role Change Audit Trail & Session Revocation Fallback Design](#28-scim-role-change-audit-trail--session-revocation-fallback-design)
29. [OQ-SSO-28.2 Resolution — `tenant_users_role_history` Retention Period (DEC-051)](#29-oq-sso-282-resolution--tenant_users_role_history-retention-period-dec-051)
30. [OQ-MOBILE-03 Resolution — Mobile SSO Browser Mode & Deep-Link Security (DEC-059)](#30-oq-mobile-03-resolution--mobile-sso-browser-mode--deep-link-security-dec-059)
31. [OQ-SSO-32.1 & OQ-SSO-32.2 Resolution — SCIM IP Enforcement Onboarding Visibility & KV Cache Invalidation](#33-oq-sso-321--oq-sso-322-resolution--scim-ip-enforcement-onboarding-visibility--kv-cache-invalidation)

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
| G-001 | **SCIM endpoint not yet implemented.** ~~Schema is defined; endpoint code, token authentication, and attribute mapping are not written.~~ **🟡 Design complete — see §27.** Full Cloudflare Worker architecture, token authentication pipeline, User/Group CRUD handlers, idempotency contract, RFC 7644 error format, and six DEC-030 lifecycle events fully specified. Implementation pending per §27.14 checklist (P0 M4/M5). | ~~Critical — blocks any SCIM provisioning~~ **🟡 Design complete; implementation pending §27.14 P0 checklist** | enterprise-architect + platform-engineer | ~~Estimate: 6–8 weeks engineering~~ **Design complete in §27 (v1.9); implementation per §27.14 checklist. SOC 2 CC6.3: advance to 🟢 Closed once §27.14 P0 tasks deploy to production and G-013 DPA block is cleared.** |
| G-002 | **SAML SLO not yet implemented.** SP-initiated and IdP-initiated SLO are designed but not coded. Logout currently only invalidates FORM session, does not propagate to IdP. | High — required for SOC 2 CC6 (logical access revocation) | security-engineer | Estimate: 2–3 weeks |
| G-003 | **OIDC back-channel logout not implemented.** The endpoint exists in design but not in code. | High | platform-engineer | Estimate: 1 week (simpler than SLO) |
| G-004 | **Certificate rotation automation.** The rotation runbook in 8.1 is manual. The 60-day expiry alert cron and the dual-cert metadata endpoint do not exist yet. | Medium — operational risk | devops-lead | Estimate: 1–2 weeks |
| G-005 | **Google Directory API integration for group sync.** ~~Currently Google Workspace OIDC lacks group membership in ID token. The service account approach is designed but not implemented.~~ **🟡 Design complete — see §21.** Service account credential management, per-tenant KV token storage (`google_svc_token_{client_email}` TTL 55 min), `getGroupsForUser()` Admin SDK call, group-to-role resolution, and `scim.group_synced` DEC-030 audit event chain fully specified in §21. Implementation pending per §21.14 checklist (P0 M4/M5). | ~~Medium — Google Workspace tenants cannot use group-based role mapping without this~~ **🟡 Design complete; implementation blocked on §21.14 P0 checklist tasks** | platform-engineer | ~~Estimate: 2 weeks~~ **Design complete in §21 (v1.7); implementation per §21.14 checklist. SOC 2 CC6: advance to 🟢 Closed once §21.14 P0 tasks deploy to production.** |
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
| Q-05 | Network: no firewall rule that would block outbound requests from `{slug}-staging.form.coach` to IdP metadata endpoints. **If the customer operates a self-hosted SCIM proxy (e.g., Okta SCIM Connector Agent, Azure AD App Proxy) and wants to restrict inbound SCIM connections to that proxy's IP range: capture the stable CIDR block(s) during this qualification call. `scim_ip_enforcement_enabled` is available at Admin Dashboard → SSO Settings → SCIM → IP Restriction and defaults to `false`. Enable it only after SCIM is confirmed working — at least one successful full-directory sync with > 0 users provisioned.** | Customer IT/security + CS | If IP enforcement desired: proxy CIDR block(s) on file before pilot go-live; enablement deferred until post-sync confirmation (§17.3 Step 3 note) |
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
| OQ-SSO-19.1 | **Role conflict resolution:** when a user is a member of two groups with conflicting role mappings, should higher or lower privilege win? | 🟢 **Resolved — DEC-039 (2026-06-11).** **Higher-privilege-wins** confirmed by founder + enterprise-architect. `group_member_effective_role` view retains `MAX()` aggregation with ordinal role scoring. `tenant_owner` excluded from group assignment. Rationale: IT assigns groups by function; specialist users in broad groups should receive their specialist role. See `docs/DECISION_LOG.md` DEC-039 for full rationale and reverse-cost assessment. |
| OQ-SSO-19.2 | **Nested group support:** should FORM support groups-within-groups (hierarchical group membership)? | Current answer: **no.** SCIM 2.0 (RFC 7643) does not define nested group semantics. IdPs handle nesting inconsistently — Okta flattens membership before pushing; Azure AD may or may not flatten depending on group type. Supporting nesting would require recursive SQL, add query complexity, and create role-resolution ambiguity. If a customer requires nested groups, the recommended workaround is to configure the IdP to push flattened membership to FORM. |
| OQ-SSO-19.3 | **Maximum groups per tenant:** what is the default cap? | 🟢 **Resolved — DEC-039 (2026-06-11).** Default cap: **500 groups per tenant.** Configurable per enterprise agreement via `tenants.feature_limits JSONB`. Rationale: 500 covers a 10,000-seat org with granular team groups; above 500 indicates a misconfigured IdP push filter. B-tree index on `(tenant_id, scim_group_id)` is performant well beyond 500 rows. Cap enforced in `resolveJitGroups()` (§19.6.2) and SCIM Group POST validation. See `docs/DECISION_LOG.md` DEC-039. |

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
| ~~Get founder sign-off on OQ-SSO-19.1 (higher-privilege-wins) and record decision in `docs/DECISION_LOG.md`~~ **[x] Done — DEC-039 (2026-06-11):** higher-privilege-wins confirmed; `docs/DECISION_LOG.md` DEC-039 is the authoritative record; OQ-SSO-19.1 and OQ-SSO-19.3 updated to 🟢 Resolved in §19.9. | enterprise-architect | ~~**P0**~~ **🟢 Closed** | Done |
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

## 21. Google Workspace Directory API Integration for Group Sync

> Owner: `enterprise-architect` + `platform-engineer`. Review: on any Google Workspace IdP configuration change or quarterly.

### 21.1 Purpose and Scope

**Problem statement.** Google Workspace OIDC does not include group memberships in the ID token (see §2.3). The `groups` claim is absent from Google's standard OIDC response regardless of scopes requested. As a result, Google Workspace tenants cannot use FORM's group-based role mapping (§5.2, §19) via the OIDC assertion path alone.

**Impact.** Without group sync, every Google Workspace user logs into FORM with the default `member` role (per §11.3 JIT provisioning defaults). Tenant admins must manually assign elevated roles (`tenant_admin`, `tenant_manager`) per user. At 100+ seats this is operationally unsustainable and is a common objection in enterprise deals where Google Workspace is the primary IdP.

**This section.** Specifies the Google Workspace Directory API group synchronisation path: a service-account-based integration that resolves group memberships for each authenticating user at login time, prior to role resolution. Group memberships are cached per-user (TTL: 5 minutes) to stay within Google API rate limits. The resolved groups feed directly into the role-mapping engine (§5.2) and audit log (§6.5), identical to SCIM-provisioned groups.

**Relationship to §19.** Section 19 specifies SCIM Group Sync — the IdP-push model where group changes are delivered to FORM via SCIM PATCH/PUT. The Directory API path in this section is the pull model: FORM calls Google on each login. The two paths are complementary:
- SCIM Groups (§19) is preferred when available: lower latency, smaller API surface, IdP-managed push.
- Directory API (this section) is the correct path for Google Workspace tenants who cannot use the three §19 workarounds (Okta bridge, Entra bridge, SAML groups attribute).

Both paths write to `scim_group_role_mappings` and resolve via the `group_member_effective_role` view (§19.4). The role resolution engine (§5.4) is unchanged.

**Scope.**
- Applies to tenants with `protocol = 'oidc'` AND `oidc_discovery_url` matching `accounts.google.com`.
- Not applicable to SAML 2.0 Google Workspace tenants (use §19.6 SAML groups attribute instead).
- Not applicable to non-Google OIDC tenants.

**Closes:** G-005 from §9 (Google Directory API integration for group sync — designed but not implemented).

---

### 21.2 Why Google OIDC Cannot Include Groups

Google's OIDC implementation deliberately does not expose group memberships as an ID token claim. This is a documented, intentional limitation:
- Google's OAuth 2.0 scopes do not include a `groups` scope.
- The `openid profile email` scopes return user identity attributes only.
- Even granting `https://www.googleapis.com/auth/cloud-identity.groups.readonly` (Cloud Identity API) only permits querying groups server-side — not embedding them in the OIDC assertion.

**The consequence:** Any application needing Google Workspace group memberships for authorisation must call the Google Admin SDK Directory API or Cloud Identity Groups API separately. The standard industry pattern is a service account with domain-wide delegation (DWD). FORM adopts this pattern.

**Google's own documentation** (Admin SDK Directory API, Groups.list) confirms: service account + DWD + `https://www.googleapis.com/auth/admin.directory.group.readonly` scope is the recommended path for server-side group resolution.

---

### 21.3 Architecture Overview

```
Google Workspace OIDC Login (SP-initiated, §1.2)
         │
         ▼
FORM OIDC Callback Worker (Cloudflare edge)
  ├─ Validate ID token (§6.2 PKCE + signature + nonce)
  ├─ Extract user sub + email from ID token
  ├─ Check google_directory_group_cache (TTL 5 min)
  │     ├─ CACHE HIT  → use cached group list
  │     └─ CACHE MISS → call Google Directory API
  │           │
  │           ▼
  │     Google Admin SDK Directory API
  │     POST https://oauth2.googleapis.com/token  (service account JWT)
  │     GET  https://admin.googleapis.com/admin/directory/v1/groups
  │          ?userKey={user_email}&maxResults=200  (paginated)
  │           │
  │           ▼
  │     Write group list to google_directory_group_cache (TTL 5 min)
  │
  ├─ Map Google groups → form_role via scim_group_role_mappings (§19.3)
  │     (same lookup path as SCIM groups — no schema changes to §19)
  ├─ Apply role resolution (§5.4 — explicit assignment beats group)
  ├─ Emit sso.login (DEC-030) with group_count in payload
  └─ Issue FORM session token (§12)
```

**Service account per tenant.** Each Google Workspace tenant configures one GCP service account with domain-wide delegation. The service account JSON credential (RSA private key + client email) is stored in `tenant_sso_configs.google_service_account_json` as AES-256-GCM ciphertext (same encryption envelope as `oidc_client_secret`). FORM's Cloudflare Worker decrypts the credential at runtime using the Cloudflare KMS key.

**No user OAuth consent required.** Domain-wide delegation means the Google Workspace super admin grants server-side access on behalf of all users in the domain. Individual users are not prompted for consent during login. This is the correct enterprise posture — user-facing OAuth consent for group membership lookup would create friction.

---

### 21.4 Schema Extensions

Two additions to `tenant_sso_configs` and one new cache table.

#### 21.4.1 `tenant_sso_configs` additions

```sql
-- Note: google_service_account_json BYTEA already exists in the §4.2 schema.
-- This section populates it. No new BYTEA column needed.

ALTER TABLE tenant_sso_configs
  ADD COLUMN google_service_account_email  TEXT,       -- service-account@project.iam.gserviceaccount.com
  ADD COLUMN google_directory_admin_email  TEXT,       -- admin@acme.com — impersonated super-admin
  ADD COLUMN google_directory_sync_enabled BOOLEAN NOT NULL DEFAULT false,
  ADD COLUMN google_directory_last_sync_at TIMESTAMPTZ,
  ADD COLUMN google_directory_sync_error   TEXT;       -- NULL = last sync OK

-- Partial index: only Google tenants with Directory sync enabled
CREATE INDEX idx_tsc_google_directory_sync
  ON tenant_sso_configs (tenant_id)
  WHERE google_directory_sync_enabled = true
    AND protocol = 'oidc'
    AND google_service_account_json IS NOT NULL;
```

#### 21.4.2 `google_directory_group_cache` table

```sql
CREATE TABLE google_directory_group_cache (
  id               UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id        UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
  user_email       TEXT NOT NULL,                        -- normalized lowercase
  groups           JSONB NOT NULL,                       -- [{id, email, name}] from Directory API
  fetched_at       TIMESTAMPTZ NOT NULL DEFAULT now(),
  expires_at       TIMESTAMPTZ NOT NULL
    GENERATED ALWAYS AS (fetched_at + INTERVAL '5 minutes') STORED,
  fetch_duration_ms INT,                                 -- API latency for cost monitoring
  error_message    TEXT,                                 -- populated if last fetch failed

  UNIQUE (tenant_id, user_email)
);

ALTER TABLE google_directory_group_cache ENABLE ROW LEVEL SECURITY;

-- Tenant isolation: tenant users cannot read another tenant's cache
CREATE POLICY google_dir_cache_isolation
  ON google_directory_group_cache
  USING (tenant_id = current_setting('app.current_tenant_id')::UUID);

-- form_system (backend Worker): unrestricted write for upsert path
CREATE POLICY google_dir_cache_system_write
  ON google_directory_group_cache
  FOR ALL TO form_system
  USING (true) WITH CHECK (true);

-- form_api: read own-tenant only (used for role resolution in callback)
CREATE POLICY google_dir_cache_api_read
  ON google_directory_group_cache
  FOR SELECT TO form_api
  USING (tenant_id = current_setting('app.current_tenant_id')::UUID);

-- TTL cleanup: pg_cron purges expired entries every 10 minutes
-- SELECT cron.schedule('google-dir-cache-cleanup', '*/10 * * * *',
--   $$DELETE FROM google_directory_group_cache WHERE expires_at < now() - INTERVAL '10 minutes'$$);
-- The 10-minute buffer beyond TTL supports the §21.5 grace-window fallback.
```

**Privacy constraint.** `google_directory_group_cache` is not exposed to `tenant_manager` or `tenant_admin` query paths. Admins can see the *resolved FORM role* for a user (via the admin dashboard) but not the raw Google group list. This is the same opacity guarantee as §14.4 for SCIM attributes — group source data is never surfaced to HR-tier roles.

---

### 21.5 Directory API Client Implementation

```typescript
// apps/api-gateway/src/sso/google-directory-groups.ts

const GOOGLE_TOKEN_URL = 'https://oauth2.googleapis.com/token';
const DIRECTORY_API_BASE = 'https://admin.googleapis.com/admin/directory/v1';
const GROUPS_PER_PAGE = 200;           // Google API maximum per page
const CACHE_GRACE_WINDOW_MS = 10 * 60 * 1000;  // 10 min: use stale on API failure

export interface GoogleGroup {
  id: string;      // Google group unique key (persistent, not email)
  email: string;   // group@domain.com
  name: string;    // human-readable display name
}

interface ServiceAccountCredential {
  client_email: string;
  private_key: string;
  project_id: string;
}

export async function resolveGoogleGroups(
  tenantId: string,
  userEmail: string,
  env: Env,
  db: SupabaseClient,
): Promise<GoogleGroup[]> {

  // 1. Cache lookup
  const { data: cached } = await db
    .from('google_directory_group_cache')
    .select('groups, expires_at, fetched_at, error_message')
    .eq('tenant_id', tenantId)
    .eq('user_email', userEmail.toLowerCase())
    .maybeSingle();

  if (cached) {
    const expiresAt = new Date(cached.expires_at).getTime();
    if (Date.now() < expiresAt) {
      return cached.groups as GoogleGroup[];  // fresh cache hit
    }
    // Expired — attempt refresh; fall back to stale within grace window
    try {
      return await fetchAndCacheGroups(tenantId, userEmail, env, db);
    } catch (err) {
      const graceDeadline = expiresAt + CACHE_GRACE_WINDOW_MS;
      if (Date.now() < graceDeadline && !cached.error_message) {
        return cached.groups as GoogleGroup[];  // stale but within grace
      }
      throw err;  // beyond grace window: propagate, login blocked
    }
  }

  // 2. No cache entry — fetch fresh
  return await fetchAndCacheGroups(tenantId, userEmail, env, db);
}

async function fetchAndCacheGroups(
  tenantId: string,
  userEmail: string,
  env: Env,
  db: SupabaseClient,
): Promise<GoogleGroup[]> {

  const startMs = Date.now();

  // 3. Load and decrypt service account credential
  const { data: ssoConfig } = await db
    .from('tenant_sso_configs')
    .select('google_service_account_json, google_directory_admin_email')
    .eq('tenant_id', tenantId)
    .eq('google_directory_sync_enabled', true)
    .single();

  if (!ssoConfig?.google_service_account_json) {
    throw new Error('google_directory_sync_enabled=true but credential missing');
  }

  const credential: ServiceAccountCredential = JSON.parse(
    await env.kms.decryptColumn(ssoConfig.google_service_account_json)
  );

  // 4. Mint service account access token (JWT bearer grant, RFC 7523)
  //    Access token cached in KV (TTL 55 min) — see OQ-SSO-21.3
  const accessToken = await getOrMintAccessToken(credential,
    ssoConfig.google_directory_admin_email, env);

  // 5. Paginate groups for this user (returns only groups user belongs to)
  const groups: GoogleGroup[] = [];
  let pageToken: string | undefined;

  do {
    const url = new URL(`${DIRECTORY_API_BASE}/groups`);
    url.searchParams.set('userKey', userEmail.toLowerCase());
    url.searchParams.set('maxResults', String(GROUPS_PER_PAGE));
    if (pageToken) url.searchParams.set('pageToken', pageToken);

    const resp = await fetch(url.toString(), {
      headers: { Authorization: `Bearer ${accessToken}` },
    });

    if (!resp.ok) {
      const body = await resp.text();
      await upsertCache(db, tenantId, userEmail, groups,
        Date.now() - startMs, `HTTP ${resp.status}: ${body.slice(0, 200)}`);
      throw new Error(`Directory API ${resp.status}: ${body.slice(0, 200)}`);
    }

    const page = await resp.json() as {
      groups?: Array<{ id: string; email: string; name: string }>;
      nextPageToken?: string;
    };
    for (const g of page.groups ?? []) {
      groups.push({ id: g.id, email: g.email, name: g.name });
    }
    pageToken = page.nextPageToken;
  } while (pageToken);

  // 6. Write cache and update last_sync_at
  await upsertCache(db, tenantId, userEmail, groups, Date.now() - startMs, null);
  await db.from('tenant_sso_configs').update({
    google_directory_last_sync_at: new Date().toISOString(),
    google_directory_sync_error: null,
  }).eq('tenant_id', tenantId);

  return groups;
}

async function getOrMintAccessToken(
  credential: ServiceAccountCredential,
  impersonateEmail: string,
  env: Env,
): Promise<string> {
  const kvKey = `google_svc_token_${credential.client_email}`;
  const cached = await env.SSO_KV.get(kvKey);
  if (cached) return cached;

  const token = await mintServiceAccountToken(credential, impersonateEmail);
  await env.SSO_KV.put(kvKey, token, { expirationTtl: 55 * 60 });  // 55 min
  return token;
}

async function mintServiceAccountToken(
  credential: ServiceAccountCredential,
  impersonateEmail: string,
): Promise<string> {
  const now = Math.floor(Date.now() / 1000);
  const header = { alg: 'RS256', typ: 'JWT' };
  const payload = {
    iss: credential.client_email,
    sub: impersonateEmail,   // DWD impersonation
    scope: 'https://www.googleapis.com/auth/admin.directory.group.readonly',
    aud: GOOGLE_TOKEN_URL,
    iat: now,
    exp: now + 3600,
  };

  const b64url = (obj: object) =>
    btoa(JSON.stringify(obj))
      .replace(/=/g, '').replace(/\+/g, '-').replace(/\//g, '_');

  const sigInput = `${b64url(header)}.${b64url(payload)}`;
  const privateKey = await importRsaPrivateKey(credential.private_key);
  const sig = await crypto.subtle.sign(
    { name: 'RSASSA-PKCS1-v1_5' },
    privateKey,
    new TextEncoder().encode(sigInput),
  );
  const sigB64 = btoa(String.fromCharCode(...new Uint8Array(sig)))
    .replace(/=/g, '').replace(/\+/g, '-').replace(/\//g, '_');

  const tokenResp = await fetch(GOOGLE_TOKEN_URL, {
    method: 'POST',
    headers: { 'Content-Type': 'application/x-www-form-urlencoded' },
    body: new URLSearchParams({
      grant_type: 'urn:ietf:params:oauth:grant-type:jwt-bearer',
      assertion: `${sigInput}.${sigB64}`,
    }),
  });
  if (!tokenResp.ok) {
    throw new Error(`Token mint failed: ${await tokenResp.text()}`);
  }
  const { access_token } = await tokenResp.json() as { access_token: string };
  return access_token;
}

async function importRsaPrivateKey(pem: string): Promise<CryptoKey> {
  const pemBody = pem
    .replace(/-----BEGIN PRIVATE KEY-----/, '')
    .replace(/-----END PRIVATE KEY-----/, '')
    .replace(/\s/g, '');
  const der = Uint8Array.from(atob(pemBody), c => c.charCodeAt(0));
  return crypto.subtle.importKey(
    'pkcs8', der.buffer,
    { name: 'RSASSA-PKCS1-v1_5', hash: 'SHA-256' },
    false, ['sign'],
  );
}

async function upsertCache(
  db: SupabaseClient,
  tenantId: string,
  userEmail: string,
  groups: GoogleGroup[],
  durationMs: number,
  errorMessage: string | null,
): Promise<void> {
  await db.from('google_directory_group_cache').upsert({
    tenant_id: tenantId,
    user_email: userEmail.toLowerCase(),
    groups,
    fetched_at: new Date().toISOString(),
    fetch_duration_ms: durationMs,
    error_message: errorMessage,
  }, { onConflict: 'tenant_id,user_email' });
}
```

**Error handling policy.** If `resolveGoogleGroups` throws after the grace window expires, the OIDC callback Worker returns HTTP 502 with an SSO error page. The user sees: *"We couldn't verify your organisation groups. Please try again or contact your IT administrator."* Login is blocked — FORM does not allow login with unresolvable group membership when Directory sync is enabled, because the resolved role might be incorrect.

**Backwards compatibility.** If `google_directory_sync_enabled = false` (the default), Google Workspace users proceed via JIT provisioning at the default `member` role (§11.3). No behaviour change for tenants who do not configure Directory sync.

---

### 21.6 Integration with Role Resolution (§5.4)

After `resolveGoogleGroups` returns the `GoogleGroup[]` array, the OIDC callback Worker maps Google group IDs to FORM roles using the existing `scim_group_role_mappings` table (§19.4), keyed on `scim_group_id = google_group.id`.

```typescript
// Inside oidc-callback.ts, after resolveGoogleGroups()

const googleGroups = await resolveGoogleGroups(tenantId, userEmail, env, db);

const { data: mappings } = await db
  .from('scim_group_role_mappings')
  .select('scim_group_id, form_role')
  .eq('tenant_id', tenantId)
  .in('scim_group_id', googleGroups.map(g => g.id));

const groupDerivedRoles = (mappings ?? []).map(m => m.form_role);
const effectiveRole = resolveEffectiveRole(directRole, groupDerivedRoles);
// §5.4: explicit direct assignment beats group; among groups, highest privilege wins
```

**Why reuse `scim_group_role_mappings`?** Google group IDs are persistent opaque identifiers (e.g., `03znysh73zz1yjz`), structurally identical to SCIM external IDs. Reusing the same mapping table avoids a parallel schema and ensures that Google Workspace tenants who later add Okta SCIM can transfer their group-role mappings without schema changes.

**Admin dashboard.** The Group Mapping UI (§16.7) shows groups from two sources for Google Workspace tenants: (a) Directory-API-resolved groups (with a "Google Directory" source badge), and (b) SCIM-pushed groups if any (with a "SCIM" badge). Tenant admins map either type using the same role-assignment interface.

---

### 21.7 Customer Configuration Steps

Performed by the customer's Google Workspace super admin. FORM's CSM provides the guide and assists.

#### Step 1 — Create GCP Project and Service Account

1. [console.cloud.google.com](https://console.cloud.google.com) → New Project (recommended name: `form-sso-<slug>`).
2. **APIs & Services → Library** → search "Admin SDK API" → Enable.
3. **IAM & Admin → Service Accounts** → Create Service Account:
   - Name: `form-directory-sync`
   - Grant no GCP-level IAM roles (Google Workspace DWD is the access mechanism, not GCP IAM).
4. Create and download a **JSON key** for the service account.

#### Step 2 — Enable Domain-Wide Delegation

1. [admin.google.com](https://admin.google.com) → **Security → API Controls → Domain-wide Delegation**.
2. **Add new** → enter service account **Client ID** (from the JSON key `client_id` field).
3. OAuth scope: `https://www.googleapis.com/auth/admin.directory.group.readonly`
4. Click Authorize.

> **Principle of least privilege.** The `admin.directory.group.readonly` scope is read-only and returns only group metadata for a specific user. Do not grant `admin.directory.user.readonly` — FORM does not require it.

#### Step 3 — Designate Impersonation Account

Choose a service or role-based administrative account (`sso-service@acme.com`) rather than a named admin's personal account, to avoid disruption if the personal account is suspended or the admin leaves.

#### Step 4 — Provide Credentials to FORM CSM

The customer transmits the JSON key file and the impersonation email to FORM via the secure CSM portal (not email). FORM's CSM uploads both via **Admin Dashboard → SSO → Google Directory Sync**. The JSON key is encrypted with AES-256-GCM before storage — the plaintext is never persisted.

#### Step 5 — Map Groups to FORM Roles

Admin Dashboard → Groups → Group Mapping → **Fetch Groups**: FORM calls the Directory API once to list all groups in the domain for the preview. The admin maps selected groups to FORM roles and saves — entries are written to `scim_group_role_mappings`.

#### Step 6 — Validate

A test user (non-admin) logs in via Google SSO. The admin verifies the user's FORM role in Admin Dashboard → Users matches the expected mapping. FORM emits `sso.google_directory_sync_validated` DEC-030 event.

---

### 21.8 Credential Rotation Protocol

Service account JSON keys should be rotated annually or immediately on suspected compromise.

| # | Action | Owner |
|---|---|---|
| 1 | GCP Console → Service Accounts → `form-directory-sync` → Keys → **Add Key (JSON)** | Customer IT admin |
| 2 | Download new JSON key | Customer IT admin |
| 3 | Admin Dashboard → SSO → Google Directory Sync → **Rotate Credential** → upload new JSON key | Tenant owner or FORM CSM |
| 4 | FORM encrypts new key, writes to `tenant_sso_configs.google_service_account_json`, emits `sso.google_directory_credential_rotated` DEC-030 event | FORM (automated) |
| 5 | Verify test login succeeds with new key | Customer IT admin |
| 6 | GCP Console → Service Accounts → Delete **old key** | Customer IT admin |
| 7 | Confirm key deletion in GCP audit log | Customer IT admin |

**Compromise path:** Customer IT admin deletes the GCP key immediately (Step 6 before Step 1). Google Workspace tenants with Directory sync enabled will see login failures until new credential is uploaded. FORM CSM must be notified immediately to expedite re-upload.

---

### 21.9 DEC-030 HMAC-Chained Audit Events

All events follow the standard DEC-030 envelope (`docs/AUDIT_LOG_SCHEMA.md`). HMAC chain must be maintained across all events.

| Event type | Trigger | Severity | Retention | Key payload fields |
|---|---|---|---|---|
| `sso.google_directory_sync_success` | API call succeeds, cache written | STANDARD | 7yr | `tenant_id`, `user_email_hash` (SHA-256), `group_count`, `fetch_duration_ms`, `cache_hit: false` |
| `sso.google_directory_sync_cache_hit` | Fresh cache entry used, no API call | STANDARD | 7yr | `tenant_id`, `user_email_hash`, `group_count`, `cache_age_seconds` |
| `sso.google_directory_sync_error` | API call fails; login blocked or grace-window fallback used | HIGH | 7yr | `tenant_id`, `user_email_hash`, `error_code`, `http_status`, `grace_window_used: bool` |
| `sso.google_directory_credential_uploaded` | Service account JSON key stored | HIGH | 7yr | `tenant_id`, `actor_id`, `uploaded_by_role`, `project_id` (from JSON, never the key itself) |
| `sso.google_directory_credential_rotated` | Existing key replaced with new key | HIGH | 7yr | `tenant_id`, `actor_id`, `old_project_id`, `new_project_id` |
| `sso.google_directory_sync_enabled` | `google_directory_sync_enabled` set true | HIGH | 7yr | `tenant_id`, `actor_id`, `admin_email_hash` (SHA-256 of impersonation email) |
| `sso.google_directory_sync_disabled` | `google_directory_sync_enabled` set false | HIGH | 7yr | `tenant_id`, `actor_id`, `reason` |
| `sso.google_directory_sync_validated` | Post-configuration validation test passed | STANDARD | 7yr | `tenant_id`, `actor_id`, `test_user_role` |

**Privacy note.** Raw email addresses are not logged. `user_email_hash` = SHA-256 of `userEmail.toLowerCase()`. A known email can be hashed and matched against audit records without the log exposing the address to all readers. Consistent with `data.breach_notification_sent` design in `docs/INCIDENT_RESPONSE.md §15.9`.

---

### 21.10 Alerting Rules

| Rule ID | Condition | Severity | Routing | Notes |
|---|---|---|---|---|
| AL-SSO-GDIR-01 | `sso.google_directory_sync_error` rate > 20% of Google Workspace logins in a 5-min window | P2 | `#sso-alerts` | Likely invalid credential or DWD permission revoked |
| AL-SSO-GDIR-02 | 3 consecutive `sso.google_directory_sync_error` events for the same tenant without intervening success | P2 | `#sso-alerts` + `#csm-alerts` | Tenant-specific failure; CSM notification required |
| AL-SSO-GDIR-03 | `fetch_duration_ms` P95 > 3,000 ms over 1-hour window | P3 | `#sso-alerts` | Google API latency degradation |
| AL-SSO-GDIR-04 | `sso.google_directory_sync_disabled` for any active tenant | P2 | `#sso-alerts` + `#csm-alerts` | Unexpected disable requires CSM review |
| AL-SSO-GDIR-05 | `sso.google_directory_credential_uploaded` without `sso.google_directory_sync_validated` within 30 minutes | P3 | `#sso-alerts` | Unvalidated credential; CSM follow-up |

---

### 21.11 GDPR and Privacy Constraints

#### 21.11.1 Google as Sub-Processor

When FORM calls the Google Admin SDK Directory API, Google processes employee directory data on FORM's behalf. This constitutes a sub-processing activity under GDPR Art. 28.

**Position.** Google Workspace's Data Processing Amendment (DPA), accepted when the GCP project is created, covers Admin SDK usage. The enterprise tenant is the controller; FORM is the processor; Google (for API execution) is a sub-processor.

**Disclosure obligation.** The sub-processor list (`docs/SOC2_READINESS.md §44`) must include a conditional entry for the Google Workspace Directory API — applicable only to tenants with `google_directory_sync_enabled = true`. The entry is not universal.

**DPA extension.** The SCIM DPA Annex B template (§14.3) must be extended to cover Directory API processing: (a) scope limited to group membership only; (b) no persistent storage of group names beyond 5-minute cache TTL; (c) no GDPR Art. 9 special category data in Directory API responses; (d) standard sub-processor change-notification obligation.

#### 21.11.2 Data Minimisation

`google_directory_group_cache.groups` stores only `id`, `email`, and `name`. No other Directory API response fields (description, aliases, directMembersCount, adminCreated, nonEditableAliases) are persisted. This satisfies GDPR Art. 5(1)(c) data minimisation.

The 5-minute TTL ensures group membership data is not retained beyond operational necessity. When a user is removed from a Google Workspace group, the stale cache expires within 5 minutes; the next login reflects the updated group membership.

#### 21.11.3 Art. 14 Transparency Obligation

FORM's processing of group membership data via the Directory API is not directly visible to affected employees. The customer controller is responsible for informing employees that FORM reads their group memberships for access control purposes (Art. 14 notice obligation). FORM's enterprise onboarding checklist (§7) includes a reminder to customer IT contacts about this obligation.

---

### 21.12 SOC 2 Evidence Mapping

| Control | Criterion | Coverage by §21 |
|---|---|---|
| CC6.1 | Logical access credentials are managed through their full lifecycle | Service account credential upload, rotation, and deletion audit events (`sso.google_directory_credential_*`) constitute the lifecycle record. Encryption at rest enforced via AES-256-GCM (same envelope as `oidc_client_secret`). |
| CC6.2 | Access to add, modify, or remove credentials requires authorisation | `sso.google_directory_credential_uploaded` and `sso.google_directory_sync_enabled` events record `actor_id` + `uploaded_by_role`; only `tenant_owner` and FORM CSM may perform these operations (RLS-enforced). |
| CC6.3 | Logical access is removed on a timely basis | When a user is removed from a Google group, the 5-minute cache TTL ensures the group-derived role is revoked within 5 minutes on the next login. `sso.google_directory_sync_disabled` HIGH event evidences intentional disablement. |
| CC7.2 | Entity monitors for anomalies | AL-SSO-GDIR-01 through AL-SSO-GDIR-05 provide continuous anomaly detection on Directory API sync health. `sso.google_directory_sync_error` HIGH events feed the SIEM alert surface. |
| CC9.2 | Entity manages vendor risk | Google Workspace Directory API usage is covered by Google's DPA (accepted at GCP project creation). Conditional sub-processor entry in `docs/SOC2_READINESS.md §44`. |
| C1.1 | Confidential information is protected | `user_email_hash` not plaintext in all audit events. Group membership not exposed to `tenant_manager` role. Raw service account JSON encrypted at column level; never stored in plaintext; never logged. |

**Evidence artefacts for SOC 2 auditor:**

| Artefact ID | Description | Collection method |
|---|---|---|
| CC6-E-GDIR-001 | `audit_log` export: all `sso.google_directory_credential_*` events for the observation period | `SELECT * FROM audit_log WHERE event_type LIKE 'sso.google_directory_credential_%' AND created_at BETWEEN $obs_start AND $obs_end ORDER BY created_at` |
| CC6-E-GDIR-002 | `audit_log` export: all `sso.google_directory_sync_error` HIGH events | `SELECT * FROM audit_log WHERE event_type = 'sso.google_directory_sync_error' AND created_at BETWEEN $obs_start AND $obs_end` |
| CC6-E-GDIR-003 | `google_directory_group_cache` performance snapshot (no groups content — minimised) | `SELECT tenant_id, fetched_at, expires_at, fetch_duration_ms, error_message IS NOT NULL AS had_error FROM google_directory_group_cache WHERE fetched_at BETWEEN $obs_start AND $obs_end` |
| CC6-E-GDIR-004 | `tenant_sso_configs` Directory sync status snapshot for all active Google Workspace tenants | `SELECT tenant_id, google_directory_sync_enabled, google_directory_last_sync_at, google_directory_sync_error FROM tenant_sso_configs WHERE google_directory_sync_enabled = true` |

---

### 21.13 Open Questions

| ID | Question | Disposition |
|---|---|---|
| OQ-SSO-21.1 | Should FORM support proactive background group sync independent of login events? | Not in M4 scope. Login-time pull is sufficient for enterprise launch. Background sync would tighten the role-revocation window below 5 minutes but adds scheduling complexity and higher API quota consumption. Revisit at M6 if tenants request tighter revocation SLA. |
| OQ-SSO-21.2 | What happens if a tenant's Google Workspace org has > 10,000 groups? | `?userKey={email}` returns only groups the specific user belongs to (max ~2,000 direct memberships per Google's limits). Paginated at 200/page. Load test required: measure P95 login latency for a user in 500 groups, with and without access-token KV caching. Target: < 1,500 ms additional latency beyond baseline (stays within §2.1 API Gateway SLO). Owner: qa-lead + devops-lead, M4. |
| OQ-SSO-21.3 | Cache the service account access token (1-hour lifetime) or mint fresh per login? | Cache per-tenant in Cloudflare KV (`google_svc_token_{client_email}`, TTL 55 min — 5 min before Google's 1-hour expiry). KV lookup ~1 ms vs. ~200 ms for a fresh JWT grant. Strongly recommended for tenants with high login frequency. Implemented in `getOrMintAccessToken()` above. |
| OQ-SSO-21.4 | Support Google Cloud Identity Groups (separate product) or only Google Workspace Groups (Admin SDK)? | Admin SDK Groups only in M4. Cloud Identity Groups requires a different API endpoint and is used primarily by GCP-native orgs, not standard Google Workspace customers. Revisit at M6 if a prospect requests it. |

---

### 21.14 Gap Status Update

| Gap ID | Description | Before §21 | After §21 |
|---|---|---|---|
| G-005 (§9) | Google Directory API integration for group sync — designed but not implemented | 🔴 Open | 🟡 Authored (full design + TypeScript implementation; pending M4 build, test, and load validation) |

---

### 21.15 Implementation Checklist

| # | Action | Owner | Priority | Milestone |
|---|---|---|---|---|
| 1 | Write migration `migrations/YYYYMMDD_tenant_sso_google_directory.sql` — ALTER TABLE additions per §21.4.1; create `google_directory_group_cache` + RLS per §21.4.2; add pg_cron cleanup job | platform-engineer | **P0** | M4 |
| 2 | Implement `apps/api-gateway/src/sso/google-directory-groups.ts` per §21.5; unit test with mock Directory API responses: success, paginated (3 pages), error + grace window, credential missing | platform-engineer | **P0** | M4 |
| 3 | Integrate `resolveGoogleGroups()` into the OIDC callback Worker (`apps/api-gateway/src/sso/oidc-callback.ts`); gate on `google_directory_sync_enabled = true`; test with a real Google Workspace staging tenant | platform-engineer | **P0** | M4 |
| 4 | Implement service account access token KV caching per OQ-SSO-21.3 (`getOrMintAccessToken()`, TTL 55 min in `env.SSO_KV`) | platform-engineer | **P1** | M4 |
| 5 | Register all 8 `sso.google_directory_*` DEC-030 event types in `docs/AUDIT_LOG_SCHEMA.md` event registry; validate HMAC chain with staging test sequence covering all 8 event types in order | platform-engineer + security-engineer | **P0** | M4 |
| 6 | Build Admin Dashboard → SSO → Google Directory Sync panel: credential upload UI, impersonation email field, enable/disable toggle, last sync timestamp, sync error message, "Fetch Groups" preview button | platform-engineer | **P1** | M4 |
| 7 | Add Google Directory group source to Group Mapping UI (§16.7): "Google Directory" source badge vs. "SCIM" badge; same role-assignment interaction | platform-engineer | **P1** | M4 |
| 8 | Configure alerting rules AL-SSO-GDIR-01 through AL-SSO-GDIR-05 in PagerDuty and add to `docs/OBSERVABILITY.md §6.2` alert rules table | devops-lead | **P0** | M4 |
| 9 | Add conditional Google Workspace Directory API sub-processor entry to `docs/SOC2_READINESS.md §44` KV payload (field `google_directory_api_conditional: true`; note: applies only to Google Workspace tenants with sync enabled) | compliance-officer | **P0** | M4 |
| 10 | Extend SCIM DPA Annex B template (§14.3) with Directory API processing clause (see §21.11.1); legal review; update DPA template in `compliance/contracts/dpa-enterprise-template.md` | compliance-officer + outside counsel | **P1** | M4 |
| 11 | Update G-005 gap entry in §9 to reference §21 (design complete); advance from 🔴 to 🟡 in `docs/SOC2_READINESS.md` CC6 section | compliance-officer | **P1** | M4 |
| 12 | Load test per OQ-SSO-21.2: test user in 500 Google groups; measure login latency P95 with and without KV token caching; confirm < 1,500 ms additional latency; document result in `compliance/evidence/perf/google-dir-load-test-M4.md` | qa-lead + devops-lead | **P1** | M4 |

---

---

## 22. High-Scale Session Revocation Architecture — KV-Backed Revocation Cache

> Precursors: §6.3 (session fixation — origin of the `session_blocklist` Supabase table), §3.5 (SCIM de-provisioning — user-level session invalidation on deactivation), §12.7 (JIT enterprise session revocation on SCIM; §12.7.4 describes the "Redis" JTI blocklist this section supersedes), G-009 (§9 — performance gap flagged as blocking before >100-seat enterprise customers).

### 22.1 Purpose and Scope

This section closes G-009. The existing `session_blocklist` Supabase table (§6.3) is correct by design — it produces a durable, auditable record of every revoked session. The problem is read performance: the Worker queries this table synchronously on every authenticated request to check whether the presented session has been revoked. At low concurrency this is unnoticeable. At enterprise scale — 100+ concurrent users each making multiple API calls — the table lookup becomes a hot-path bottleneck.

Concrete failure mode: a 300-seat enterprise tenant triggers a SCIM bulk deactivation of 200 users. The SCIM endpoint must write 200 × (average 3 sessions/user) = 600 rows to `session_blocklist` synchronously before responding to the SCIM client. This exceeds the 10-second Worker response timeout, causing SCIM client retries, duplicate audit events, and unpredictable session state.

This section specifies:
1. A **Cloudflare KV cache layer** (`SESSION_REVOCATION_KV`) that serves revocation checks at ~1 ms latency.
2. A **four-key-type schema** handling single session, per-user, bulk, and tenant-nuke revocations.
3. A **bulk revocation handler** for large SCIM batches completing 1,000-user deactivation in < 200 ms.
4. A **tenant nuke function** (O(1) single KV write) used in §8.3 emergency SSO disable and INCIDENT_RESPONSE.md R-01.
5. Integration of the "Redis" JTI blocklist described in §12.7.4 into the existing Cloudflare KV infrastructure — no new external service required.

Supabase `session_blocklist` remains the authoritative audit trail. KV is a **write-through cache**, not a replacement. Every revocation writes to both KV (fast, TTL-based) and Supabase (durable, queryable, auditor-visible).

### 22.2 Current Architecture Bottleneck Analysis

| Operation | Current Implementation | Latency | Problem at Scale |
|---|---|---|---|
| Single revocation write | `INSERT INTO session_blocklist` | ~5 ms | Acceptable |
| Per-request blocklist check | `SELECT 1 FROM session_blocklist WHERE session_id = $1` | ~15–25 ms | At 100 RPS: 20 concurrent DB queries just for revocation checks; saturates Supabase `nano` connection pool |
| Bulk user revocation (200 users, 3 sessions/user) | 600 INSERT rows, sequential | >10 s | Exceeds SCIM response timeout; causes SCIM client retry storms |
| Tenant nuke (all sessions for a tenant) | `INSERT INTO session_blocklist SELECT session_id ... WHERE tenant_id = $1` | Unbounded — depends on active session count | A 500-seat tenant with high active sessions could take 30–60 s; Supabase connection pool exhausted during incident response |
| JTI blocklist check (§12.7.4 opt-in) | Described as "Redis" in §12.7.4 — no Redis deployment in current architecture | N/A — not yet built | Gap: the JTI revocation feature cannot ship without a fast key-value store |

**KV comparison:**

| Metric | Supabase SELECT | Cloudflare KV GET |
|---|---|---|
| P50 latency | ~10 ms | ~1 ms |
| P99 latency | ~50 ms | ~5 ms |
| Connection pool impact | Yes — consumes Supabase connection per query | None |
| Quota limit | Supabase `nano`: 60 connections | Workers KV: 100,000 reads/day free; $0.50/10M paid |
| Bulk write speed | ~50 rows/s sequential | ≥50 parallel KV writes per request |

### 22.3 Two-Tier Revocation Architecture

```
  Authenticated request arrives at Cloudflare Worker
                      │
             ┌────────▼─────────┐
             │  Validate JWT     │  RS256 signature + exp + iss + aud + tenant_id
             └────────┬─────────┘
                      │ Valid JWT
             ┌────────▼──────────────────────────┐
             │     KV Revocation Check            │  ~1–3 ms total
             │                                    │
             │  Promise.all([                     │
             │    KV.get(tenant:{id}:all),         │  ← O(1) tenant nuke check
             │    KV.get(user:{tid}:{uid}),        │  ← user-level revocation
             │    KV.get(session:{sid}),           │  ← individual session
             │  ])                                │
             │                                    │
             │  if jti_check_enabled:             │
             │    KV.get(jti:{jti})               │  ← zero-deprovisioning-latency
             └────────┬──────────────────┬────────┘
                      │                  │
             KV HIT (revoked)       KV MISS
                      │                  │
              Return 401          ┌──────▼─────────────────┐
              (HMAC event         │  Migration fallback     │  ← first 30 days only
               session.revoked    │  Supabase SELECT        │
               _at_check)         │  (catches pre-KV rows)  │
                                  └──────┬──────────────────┘
                                         │
                                  Continue to handler
```

**Tier 1 — Cloudflare KV** (`SESSION_REVOCATION_KV`): Hot cache, ~1 ms read. Primary check path for all requests. TTL-based expiry — no sweep job needed, KV reclaims expired keys automatically.

**Tier 2 — Supabase `session_blocklist`**: Authoritative audit trail, ~20 ms read. Written on every revocation. During a 30-day migration window, also read on KV MISS to catch sessions revoked before the KV layer was deployed. After 30 days, all such sessions have expired naturally; the fallback is removed.

**Fail-open vs. fail-closed on KV unavailability:** For tenants with `jti_revocation_check: false` (default), if KV is unavailable the request falls back to the Supabase check path — fail-open relative to KV but still checked. For tenants with `jti_revocation_check: true` (zero-deprovisioning-latency SLA), a KV unavailability on the JTI check fails closed (`503 Service Unavailable`) rather than allowing a potentially-revoked token through. This asymmetry is intentional and contractual.

### 22.4 KV Key Schema

**Namespace:** `SESSION_REVOCATION_KV` — distinct from `SSO_KV` (§21) to isolate quota and prevent accidental purge during SSO KV maintenance.

| Key | Value | TTL | Emitted by |
|---|---|---|---|
| `revoke:tenant:{tenant_id}:all` | `"1"` | `max_jwt_ttl_seconds` (default 28800 = 8 h) | `nukeTenantSessions()` — §8.3 emergency disable, INCIDENT_RESPONSE R-01 |
| `revoke:user:{tenant_id}:{user_id}` | `"1"` | 86400 (24 h; covers max JWT TTL) | `revokeUserSessions()` — SCIM deactivation, admin kick |
| `revoke:session:{session_id}` | `"1"` | `remaining_exp_seconds` (≥ 1) | `revokeSession()` — logout, session fixation prevention |
| `revoke:jti:{jti}` | `"1"` | `remaining_exp_seconds` (≥ 1) | `revokeJti()` — opt-in zero-deprovisioning-latency tenants only |

**Privacy:** No PII in any key or value. `session_id` is a UUID. `user_id` is an internal UUID (not email, not name). `tenant_id` is an internal UUID. Value is always the string `"1"`. No health data, no biometric data, no names, no emails stored in KV at any point.

**TTL rationale:** Keys expire precisely when the revoked credential would have expired anyway. A revoked session JWT that would have expired in 10 minutes produces a KV key with TTL = 600 seconds. After 600 seconds, both the JWT and the KV key are gone — no stale blocklist entries and no maintenance overhead.

### 22.5 Worker Implementation

**File:** `apps/api-gateway/src/sso/session-revocation-kv.ts`

```typescript
// apps/api-gateway/src/sso/session-revocation-kv.ts
// KV-backed session revocation cache — closes G-009.
// Supabase session_blocklist is the audit trail; this file is the fast-path cache.

export type RevocationReason =
  | 'logout'
  | 'session_fixation'
  | 'scim_deactivation'
  | 'scim_deletion'
  | 'admin_kick'
  | 'tenant_nuke'
  | 'jti_explicit'
  | 'reauth_timeout';

export type KvSyncStatus = 'pending' | 'synced' | 'failed';

// ── Key constructors ────────────────────────────────────────────────────────

const K = {
  tenantAll: (tenantId: string) => `revoke:tenant:${tenantId}:all`,
  user:      (tenantId: string, userId: string) => `revoke:user:${tenantId}:${userId}`,
  session:   (sessionId: string) => `revoke:session:${sessionId}`,
  jti:       (jti: string) => `revoke:jti:${jti}`,
} as const;

// ── Check (hot path — called on every authenticated request) ────────────────

export async function isRevoked(
  kv: KVNamespace,
  params: {
    sessionId: string;
    tenantId: string;
    userId: string;
    jti?: string;
    jtiCheckEnabled: boolean;
  }
): Promise<{ revoked: boolean; reason: RevocationReason | null }> {
  // Three parallel KV reads — tenant nuke, user-level, session-level
  const [tenantHit, userHit, sessionHit] = await Promise.all([
    kv.get(K.tenantAll(params.tenantId)),
    kv.get(K.user(params.tenantId, params.userId)),
    kv.get(K.session(params.sessionId)),
  ]);

  if (tenantHit)   return { revoked: true, reason: 'tenant_nuke' };
  if (userHit)     return { revoked: true, reason: 'scim_deactivation' };
  if (sessionHit)  return { revoked: true, reason: 'logout' };

  // JTI check is opt-in (zero-deprovisioning-latency tenants); separate read
  if (params.jtiCheckEnabled && params.jti) {
    const jtiHit = await kv.get(K.jti(params.jti));
    if (jtiHit) return { revoked: true, reason: 'jti_explicit' };
  }

  return { revoked: false, reason: null };
}

// ── Single session revocation ───────────────────────────────────────────────

export async function revokeSession(
  kv: KVNamespace,
  supabase: SupabaseClient,
  params: {
    sessionId: string;
    tenantId: string;
    userId: string;
    reason: RevocationReason;
    remainingTtlSeconds: number;
    jti?: string;
    jtiCheckEnabled?: boolean;
  }
): Promise<void> {
  const kvWrites: Promise<void>[] = [
    kv.put(K.session(params.sessionId), '1', {
      expirationTtl: Math.max(params.remainingTtlSeconds, 1),
    }),
  ];

  if (params.jtiCheckEnabled && params.jti) {
    kvWrites.push(
      kv.put(K.jti(params.jti), '1', {
        expirationTtl: Math.max(params.remainingTtlSeconds, 1),
      })
    );
  }

  // KV write and Supabase audit write in parallel
  const [kvResult] = await Promise.allSettled([
    Promise.all(kvWrites),
    supabase.from('session_blocklist').insert({
      session_id:     params.sessionId,
      tenant_id:      params.tenantId,
      user_id:        params.userId,
      reason:         params.reason,
      blocked_at:     new Date().toISOString(),
      kv_sync_status: 'pending',
    }),
  ]);

  if (kvResult.status === 'rejected') {
    // KV write failed — Supabase fallback active; emit HIGH alert event
    await emitDec030(supabase, {
      event_type: 'session.revocation_kv_sync_error',
      severity:   'HIGH',
      tenant_id:  params.tenantId,
      metadata:   { session_id: params.sessionId, error: String(kvResult.reason) },
    });
    await supabase
      .from('session_blocklist')
      .update({ kv_sync_status: 'failed' })
      .eq('session_id', params.sessionId);
  } else {
    await supabase
      .from('session_blocklist')
      .update({ kv_sync_status: 'synced' })
      .eq('session_id', params.sessionId);
  }
}

// ── User-level revocation (SCIM deactivation — single user) ────────────────

export async function revokeUserSessions(
  kv: KVNamespace,
  supabase: SupabaseClient,
  params: {
    tenantId: string;
    userId: string;
    reason: RevocationReason;
    activeSessions: Array<{ session_id: string; jti?: string; exp_unix: number }>;
    jtiCheckEnabled?: boolean;
  }
): Promise<void> {
  const now = Math.floor(Date.now() / 1000);
  const maxTtl = Math.max(...params.activeSessions.map(s => s.exp_unix - now), 1);

  // User-level key: single write covers all current sessions
  await kv.put(K.user(params.tenantId, params.userId), '1', {
    expirationTtl: Math.min(maxTtl, 86400),
  });

  // Per-session keys (for JTI check path and precise fallback during migration window)
  const perSessionWrites = params.activeSessions.flatMap(s => {
    const ttl = Math.max(s.exp_unix - now, 1);
    const writes: Promise<void>[] = [
      kv.put(K.session(s.session_id), '1', { expirationTtl: ttl }),
    ];
    if (params.jtiCheckEnabled && s.jti) {
      writes.push(kv.put(K.jti(s.jti), '1', { expirationTtl: ttl }));
    }
    return writes;
  });

  await Promise.all(perSessionWrites);
  // Supabase audit trail is written by the SCIM handler via session.revoked_by_scim events (§12.9)
}

// ── Bulk SCIM revocation (>50 users — batched to avoid Worker CPU limit) ───

const BATCH_SIZE = 50;

function chunk<T>(arr: T[], size: number): T[][] {
  const out: T[][] = [];
  for (let i = 0; i < arr.length; i += size) out.push(arr.slice(i, i + size));
  return out;
}

export async function handleBulkScimRevocation(
  kv: KVNamespace,
  supabase: SupabaseClient,
  params: {
    tenantId: string;
    deactivatedUserIds: string[];
    scimRequestId: string;
    jtiCheckEnabled?: boolean;
  }
): Promise<void> {
  const startMs = Date.now();

  // Emit started event BEFORE any state mutation (HMAC chain ordering)
  await emitDec030(supabase, {
    event_type: 'session.bulk_revocation_started',
    severity:   'HIGH',
    tenant_id:  params.tenantId,
    metadata:   {
      user_count:      params.deactivatedUserIds.length,
      scim_request_id: params.scimRequestId,
    },
  });

  // Fetch all active sessions for the deactivated users
  const { data: activeSessions } = await supabase
    .from('enterprise_sessions')
    .select('session_id, user_id, expires_at')
    .in('user_id', params.deactivatedUserIds)
    .eq('tenant_id', params.tenantId)
    .is('revoked_at', null);

  const sessions = activeSessions ?? [];
  const now = Math.floor(Date.now() / 1000);

  // Write user-level KV keys in batches of BATCH_SIZE
  for (const batch of chunk(params.deactivatedUserIds, BATCH_SIZE)) {
    await Promise.all(
      batch.map(uid =>
        kv.put(K.user(params.tenantId, uid), '1', { expirationTtl: 86400 })
      )
    );
  }

  // Write per-session KV keys in batches of BATCH_SIZE
  for (const batch of chunk(sessions, BATCH_SIZE)) {
    await Promise.all(
      batch.map(s => {
        const ttl = Math.max(
          Math.floor(new Date(s.expires_at).getTime() / 1000) - now,
          1
        );
        return kv.put(K.session(s.session_id), '1', { expirationTtl: ttl });
      })
    );
  }

  const durationMs = Date.now() - startMs;

  await emitDec030(supabase, {
    event_type: 'session.bulk_revocation_complete',
    severity:   'HIGH',
    tenant_id:  params.tenantId,
    metadata:   {
      user_count:      params.deactivatedUserIds.length,
      sessions_revoked: sessions.length,
      duration_ms:     durationMs,
      scim_request_id: params.scimRequestId,
    },
  });
}

// ── Tenant nuke — emergency SSO disable (§8.3) and P0 incident response ────
// Requires two-person authorisation. Replaces the unbounded
// INSERT SELECT pattern in §8.3 with a single O(1) KV write.

export async function nukeTenantSessions(
  kv: KVNamespace,
  supabase: SupabaseClient,
  params: {
    tenantId: string;
    reason: 'sso_emergency_disable' | 'incident_response' | 'admin_request';
    authorisedBy: [string, string]; // two-person rule: [approver_1_user_id, approver_2_user_id]
    maxJwtTtlSeconds?: number;      // default 28800 (8 h)
  }
): Promise<void> {
  const ttl = params.maxJwtTtlSeconds ?? 28800;

  await emitDec030(supabase, {
    event_type: 'session.tenant_nuke_started',
    severity:   'CRITICAL',
    tenant_id:  params.tenantId,
    metadata:   { reason: params.reason, authorised_by: params.authorisedBy },
  });

  // Single KV write — all in-flight JWTs for this tenant are now revoked
  await kv.put(K.tenantAll(params.tenantId), '1', { expirationTtl: ttl });

  await emitDec030(supabase, {
    event_type: 'session.tenant_nuke_complete',
    severity:   'CRITICAL',
    tenant_id:  params.tenantId,
    metadata:   {
      reason:        params.reason,
      authorised_by: params.authorisedBy,
      kv_ttl_seconds: ttl,
    },
  });
}
```

### 22.6 Integration Points

#### 22.6.1 Updating §8.3 Emergency SSO Disable

The SQL `INSERT INTO session_blocklist SELECT ... WHERE tenant_id = $TENANT_ID` in §8.3 is replaced by a call to `nukeTenantSessions()`:

```typescript
// In: apps/api-gateway/src/sso/emergency-disable.ts
await nukeTenantSessions(env.SESSION_REVOCATION_KV, supabase, {
  tenantId:     tenantId,
  reason:       'sso_emergency_disable',
  authorisedBy: [approver1UserId, approver2UserId],
});
// Then disable SSO config and enable magic link fallback (SQL unchanged from §8.3 steps 1 and 3)
```

The `INSERT INTO session_blocklist` in §8.3 step 2 is retained as the audit trail write — the KV nuke is the fast-path; Supabase remains the evidence source.

#### 22.6.2 Updating §6.3 Session Fixation Prevention

Step 4 in §6.3 ("The old session UUID is added to a session blocklist") becomes a call to `revokeSession()` instead of a raw INSERT:

```typescript
await revokeSession(env.SESSION_REVOCATION_KV, supabase, {
  sessionId:           oldSessionId,
  tenantId:            tenantId,
  userId:              userId,
  reason:              'session_fixation',
  remainingTtlSeconds: remainingTtl,
  jti:                 oldJti,
  jtiCheckEnabled:     sessionPolicy.jti_revocation_check,
});
```

#### 22.6.3 Updating §12.7.4 JTI Blocklist

The "Redis" JTI blocklist described in §12.7.4 is now implemented via `SESSION_REVOCATION_KV` with the `revoke:jti:{jti}` key pattern. No external Redis or Upstash deployment is required. The `jti_revocation_check: true` tenant feature flag in `session_policy` JSONB (§12.7.4) continues to gate this behaviour.

When `jti_revocation_check` is enabled for a tenant, `revokeUserSessions()` also writes `revoke:jti:{jti}` keys for every active access token under the revoked sessions. The `isRevoked()` function checks this key as the fourth parallel KV read.

### 22.7 Schema Change

One migration, one column addition to `session_blocklist`:

```sql
-- migrations/YYYYMMDD_session_blocklist_kv_status.sql

-- New ENUM for KV sync tracking
CREATE TYPE revocation_kv_status AS ENUM ('pending', 'synced', 'failed');

ALTER TABLE session_blocklist
  ADD COLUMN kv_sync_status revocation_kv_status NOT NULL DEFAULT 'pending';

-- Backfill: rows inserted before this migration have no corresponding KV entry.
-- Mark them 'synced' conceptually (they are caught by the Supabase fallback path
-- during the 30-day migration window, then expire naturally).
UPDATE session_blocklist
SET kv_sync_status = 'synced'
WHERE blocked_at < NOW();

-- Index for the compliance evidence query (§22.10 CC6-E-REV-002)
CREATE INDEX CONCURRENTLY idx_session_blocklist_kv_status
  ON session_blocklist (kv_sync_status)
  WHERE kv_sync_status != 'synced';

COMMENT ON COLUMN session_blocklist.kv_sync_status IS
  'Whether this revocation has been mirrored to SESSION_REVOCATION_KV. '
  '''failed'' means KV write failed; Supabase fallback is active for this session.';
```

**RLS:** `session_blocklist` RLS policy is unchanged. The new column is write-accessible to `form_system` (SCIM handler + Worker) and `form_admin` (compliance queries); `form_api` and `tenant_manager` have no access to this table (same as before).

### 22.8 Performance Model

| Scenario | Before (Supabase only) | After (KV + Supabase audit) | Improvement |
|---|---|---|---|
| Per-request revocation check | ~20 ms Supabase SELECT | ~3 ms 3× parallel KV GET | 6.7× faster |
| Single session revocation write | ~5 ms INSERT | ~1 ms KV PUT + ~5 ms INSERT (parallel) | No regression; total wall time ~5 ms |
| User-level revocation (5 sessions/user) | ~25 ms (5× INSERT) | ~3 ms (user KV PUT + 5× session KV PUT in parallel) | 8× faster |
| Bulk 1,000-user SCIM deactivation | >10 s (INSERT SELECT timeout) | ~150 ms (30 batches × 5 ms) | >66× faster; fits in SCIM response window |
| Tenant nuke | Unbounded (INSERT SELECT full table) | ~2 ms (single KV PUT) | O(N) → O(1) |

**KV quota at 300-seat enterprise tenant (10 RPS per user):**
- Reads: 300 × 10 × 3 = 9,000 reads/second = 777M reads/day — requires Workers Paid plan ($0.50/10M = ~$38.85/day per tenant). **This cost must be factored into enterprise COGS (COST_MODEL.md §15.6 infrastructure overhead).**
- Writes: < 1,000 revocations/day per typical enterprise tenant = negligible.

**KV cost ceiling:** Cloudflare Workers Paid KV pricing applies per namespace. At $0.50 per 10M reads and 777M reads/day (300-seat tenant at 10 RPS), the per-tenant daily KV read cost is ~$0.039/day = ~$14.17/year. At the $6–12/seat/month enterprise price point, this is < 0.4% of revenue for the heaviest users. See COST_MODEL.md §15.4.

### 22.9 Migration Window Protocol

**Day 0 (deploy):** `SESSION_REVOCATION_KV` namespace created and bound to the API Gateway Worker. All new revocations write to both KV and Supabase. The `isRevoked()` function checks KV first; on KV MISS during the migration window it falls back to the Supabase SELECT.

**Days 1–30 (migration window):** Supabase fallback is active. Sessions revoked before Day 0 are not in KV; they are caught by the Supabase fallback. `kv_sync_status = 'synced'` (backfilled) on pre-existing rows; Workers skip the Supabase fallback for these (the backfill is a no-op correctness signal, not a data migration).

**Day 30 (cutover):** Maximum JWT TTL is 8 hours. All sessions revoked before Day 0 have expired within 8 hours of Day 0 — well within the 30-day window. Remove the Supabase fallback path from `isRevoked()`. Set a feature flag `session_revocation_kv_only: true` in `tenants.feature_flags` to gate the cutover per-tenant if needed.

### 22.10 DEC-030 HMAC-Chained Audit Events

Five new events. All registered in `docs/AUDIT_LOG_SCHEMA.md`. All HMAC-chained per DEC-030.

| Event Type | Severity | Retention | Key Metadata | Trigger |
|---|---|---|---|---|
| `session.bulk_revocation_started` | HIGH | 7 yr | `tenant_id`, `user_count`, `scim_request_id` | Emitted before first KV write in `handleBulkScimRevocation()` |
| `session.bulk_revocation_complete` | HIGH | 7 yr | `tenant_id`, `user_count`, `sessions_revoked`, `duration_ms`, `scim_request_id` | Emitted after all KV writes complete |
| `session.tenant_nuke_started` | CRITICAL | 7 yr | `tenant_id`, `reason`, `authorised_by` (array of two user IDs, no names) | Emitted before KV write; two-person authorisation enforced at API layer |
| `session.tenant_nuke_complete` | CRITICAL | 7 yr | `tenant_id`, `reason`, `authorised_by`, `kv_ttl_seconds` | Emitted after KV write confirmed |
| `session.revocation_kv_sync_error` | HIGH | 7 yr | `tenant_id`, `session_id`, `error` (error class only, no stack trace) | Emitted when KV PUT fails; Supabase-only fallback activates; triggers PagerDuty P1 |

**HMAC chain ordering constraint:** `session.bulk_revocation_started` must be emitted and its HMAC chain entry committed before the first KV write for that batch. This preserves the invariant that audit chain entries precede state mutations — required for DEC-030 tamper evidence (a chain entry cannot be retroactively inserted for an action that already occurred).

**`session.tenant_nuke_started` two-person rule:** The `nukeTenantSessions()` function accepts `authorisedBy: [string, string]` — two distinct user IDs. The API layer (the endpoint calling this function) must verify that the two IDs are different users and that both hold the `form_admin` role. This check must occur before `emitDec030` is called. A nuke attempted with a single authoriser must be rejected with `403 Forbidden` and logged as `support.unauthorized_nuke_attempt` (HIGH severity).

### 22.11 Alerting Rules

| ID | Condition | Severity | Response SLA | Runbook |
|---|---|---|---|---|
| AL-REVOKE-01 | `session.revocation_kv_sync_error` rate > 1% of revocation events over any 5-minute window | P1 | Acknowledge < 30 min | Check `SESSION_REVOCATION_KV` namespace binding in Worker `wrangler.toml`; confirm KV quota not exceeded (`wrangler kv:key list --namespace-id=...`); if quota exceeded, contact Cloudflare support and activate Supabase-only fallback mode (`session_revocation_kv_only: false`) |
| AL-REVOKE-02 | `session.bulk_revocation_complete.duration_ms` > 5,000 ms over any 1-hour window | P2 | Acknowledge < 2 h | Reduce `BATCH_SIZE` from 50 to 25 in `session-revocation-kv.ts`; confirm Cloudflare KV region latency; check if deactivated user count per batch is unusually high (large SCIM sync); consider increasing SCIM client retry timeout |

Add both rules to `docs/OBSERVABILITY.md §6.2` alert rules table at the `session_revocation` subsection.

### 22.12 SOC 2 Evidence Mapping

| Criterion | Control | Evidence | Collection Method |
|---|---|---|---|
| CC6.3 | Logical access is removed on a timely basis | KV revocation completes in < 100 ms for single-user; < 200 ms for 1,000-user bulk; tenant nuke in < 5 ms | `audit_log` export: `session.bulk_revocation_complete.duration_ms` distribution; P99 < 500 ms |
| CC6.3 | JTI revocation window (opt-in tenants) | Contractual zero-deprovisioning-latency achieved: JTI revoked synchronously within the SCIM handler response | `session.jti_blocklisted` events in audit chain; confirm `jti_revocation_check: true` in `session_policy` for qualifying tenants |
| CC7.2 | Environmental threat monitoring | AL-REVOKE-01 monitors KV sync health; error rate > 1% triggers P1 alert | PagerDuty incident log for AL-REVOKE-01; `session.revocation_kv_sync_error` event count in 90-day observation window |
| CC7.3 | Response to anomalies | `session.revocation_kv_sync_error` triggers immediate Supabase fallback — no revocation is silently lost | `kv_sync_status = 'failed'` count in `session_blocklist` for observation period (must be < 0.1%) |

**Evidence artefacts for SOC 2 auditor:**

| Artefact ID | Description | Collection SQL / Method |
|---|---|---|
| CC6-E-REV-001 | `audit_log` export: all `session.bulk_revocation_*` and `session.tenant_nuke_*` events for the observation period | `SELECT event_type, created_at, tenant_id, metadata->>'user_count' AS user_count, metadata->>'duration_ms' AS duration_ms FROM audit_log WHERE event_type LIKE 'session.bulk_revocation_%' OR event_type LIKE 'session.tenant_nuke_%' AND created_at BETWEEN $obs_start AND $obs_end ORDER BY created_at` |
| CC6-E-REV-002 | `session_blocklist` KV sync status distribution (confirms > 99% of revocations mirrored to KV) | `SELECT kv_sync_status, count(*), round(100.0 * count(*) / sum(count(*)) OVER (), 2) AS pct FROM session_blocklist WHERE blocked_at BETWEEN $obs_start AND $obs_end GROUP BY kv_sync_status ORDER BY kv_sync_status` |

### 22.13 Open Questions

| ID | Question | Disposition |
|---|---|---|
| OQ-SSO-22.1 | At what seat count should the Cloudflare Workers Paid KV plan be activated? | Activate at first enterprise tenant go-live regardless of seat count — the free tier (100k reads/day) is exceeded by a single active 10-seat tenant at 10 RPS. The KV paid plan ($5/month base + $0.50/10M reads) is a fixed cost that should be included in COGS from day one of enterprise operations. Owner: devops-lead. Decision: before first pilot go-live. |
| OQ-SSO-22.2 | Should the `session_revocation_kv_only` feature flag be per-tenant or global? | Global flag initially (simpler operational story). Per-tenant only if a specific enterprise customer requires a contractual guarantee that their sessions are never checked via the Supabase fallback path. Default: global cutover at Day 30 per §22.9. |
| OQ-SSO-22.3 | The `duration_ms` field in `session.bulk_revocation_complete` includes KV write time only. Should it include the Supabase `enterprise_sessions` fetch time? | Yes — the full elapsed time from `handleBulkScimRevocation()` entry to exit is more useful for SCIM timeout debugging. Update `startMs` to capture before the Supabase SELECT, not after. Owner: platform-engineer. Low priority — does not affect correctness. |

### 22.14 Gap Status Update

| Gap ID | Description | Before §22 | After §22 |
|---|---|---|---|
| G-009 (§9) | Session blocklist persistence — Supabase table lookup hot-path bottleneck at > 100-seat enterprise deployments | 🔴 Open (performance, not correctness; but blocking before large-tenant go-live) | 🟡 Authored (full design + TypeScript implementation; KV key schema; bulk revocation handler; tenant nuke; DEC-030 events; SOC 2 evidence; pending M4 build and load validation) |

### 22.15 Implementation Checklist

| # | Action | Owner | Priority | Milestone |
|---|---|---|---|---|
| 1 | Create `SESSION_REVOCATION_KV` KV namespace in Cloudflare dashboard; add binding to `wrangler.toml` for the API Gateway Worker; confirm binding name matches `env.SESSION_REVOCATION_KV` in TypeScript | devops-lead | **P0** | M4 |
| 2 | Write migration `migrations/YYYYMMDD_session_blocklist_kv_status.sql` per §22.7; apply to staging; confirm `kv_sync_status` column exists; confirm index created CONCURRENTLY without locking | platform-engineer | **P0** | M4 |
| 3 | Implement `apps/api-gateway/src/sso/session-revocation-kv.ts` per §22.5; unit test `isRevoked()` for all four key types; unit test `revokeSession()` with KV failure injection; unit test `handleBulkScimRevocation()` with 1,000-user mock | platform-engineer | **P0** | M4 |
| 4 | Integrate `isRevoked()` into the Worker's authenticated request validation path (after JWT verification, before handler dispatch); confirm it replaces the direct Supabase `session_blocklist` SELECT | platform-engineer | **P0** | M4 |
| 5 | Update §8.3 emergency SSO disable procedure: replace `INSERT INTO session_blocklist SELECT` with `nukeTenantSessions()` call; add two-person authorisation check to the endpoint; test with a staging tenant | platform-engineer + security-engineer | **P0** | M4 |
| 6 | Update §6.3 session fixation prevention: replace raw INSERT with `revokeSession()` call in the SSO callback Worker | platform-engineer | **P0** | M4 |
| 7 | Register all 5 new DEC-030 event types in `docs/AUDIT_LOG_SCHEMA.md` event registry; validate HMAC chain with staging test covering: single revocation → bulk revocation → tenant nuke sequence | platform-engineer + security-engineer | **P0** | M4 |
| 8 | Configure AL-REVOKE-01 and AL-REVOKE-02 alerting rules in PagerDuty; add to `docs/OBSERVABILITY.md §6.2` | devops-lead | **P0** | M4 |
| 9 | Update G-009 entry in §9 to reference §22 (design complete); advance from 🔴 to 🟡 in internal gap tracker | compliance-officer | **P1** | M4 |
| 10 | Load test: simulate 1,000-user bulk SCIM deactivation against staging; confirm `session.bulk_revocation_complete.duration_ms` < 500 ms; confirm zero SCIM client timeouts; document result in `compliance/evidence/perf/session-revocation-bulk-load-test-M4.md` | qa-lead + devops-lead | **P1** | M4 |
| 11 | Day 30 after deploy: confirm all pre-deploy sessions have expired; remove Supabase fallback from `isRevoked()`; set `session_revocation_kv_only: true`; collect CC6-E-REV-002 snapshot as cutover evidence | devops-lead + compliance-officer | **P1** | M5 |

---

*v1.1 additions (2026-05-30): Section 19 — SCIM Groups Sync & Group-Based Role Mapping. Closes the group lifecycle gap left by §5–§8 (which cover user provisioning and direct role assignment but not group objects as first-class SCIM resources). Six new SCIM endpoints on the Cloudflare Workers edge runtime: GET /Groups (paginated, filterable, RFC 7644 ListResponse), GET /Groups/{id} (single resource with members array), POST /Groups (creates `tenant_groups` row, 201 + Location header, idempotency on externalId), PUT /Groups/{id} (full replace, delta-aware audit emission), PATCH /Groups/{id} (RFC 7644 PatchOp, op=add/remove on path=members, with cross-tenant guard on member userIds), DELETE /Groups/{id} (cascade role revocation, no user deletion). Group-to-role mapping model: new `scim_group_role_mappings` table (tenant_id, scim_group_id FK, form_role ENUM, mapped_by, mapped_at; UNIQUE constraint one role per group per tenant; RLS: tenant_admin read/write own tenant, form_system read-only, form_api zero-rows fail-closed). Role resolution priority: explicit direct assignment beats group membership; among group memberships highest privilege wins (highest-privilege-wins rationale documented in §19.3.2; requires founder sign-off per OQ-SSO-19.1). New `scim_group_members` table for SCIM-managed many-to-many membership. New `group_member_effective_role` view computing final FORM role per user via FULL OUTER JOIN of direct roles and group-derived roles using CASE ordinals and MAX aggregation. IdP push group configuration: Okta Push Groups (filter expression, memberOf, PUT-then-PATCH behaviour), Azure AD / Entra SCIM mappings (objectId → externalId, scope filter, 40-minute sync lag caveat, GET by externalId query pattern), Google Workspace (no native SCIM Groups Push; three workarounds: Okta bridge, Entra bridge, SAML groups attribute). SAML JIT groups path: `groups_attribute` key in `tenant_sso_configs.attribute_mapping` JSONB; `resolveJitGroups` function; `scim.jit_group_creation_enabled` feature flag (disabled by default; safety: 500-group cap, 255-char name limit, admin-only toggle). Six DEC-030 HMAC-chained audit events: scim.group_created (STANDARD, 7yr), scim.group_updated (STANDARD, 7yr), scim.group_deleted (HIGH, 7yr), scim.group_member_added (STANDARD, 7yr), scim.group_member_removed (HIGH, 7yr), scim.group_role_mapping_changed (HIGH, 7yr); sequential emission required for batch PATCH to maintain HMAC chain integrity. SOC 2 mapping: CC6.1 (group access auditable), CC6.2 (no-mapping-by-default fail-closed), CC6.3 (DELETE cascade offboarding), CC6.6 (JIT flag gated), CC7.2 (HIGH events for SIEM), C1.1 (IDs only in audit events). Open questions: OQ-SSO-19.1 (higher vs. lower privilege conflict resolution — founder sign-off required), OQ-SSO-19.2 (nested groups — no, not supported), OQ-SSO-19.3 (500 group cap — enterprise-architect decision). Implementation checklist: 10 tasks (7× P0, 3× P1), M4/M5.*

*v1.2 additions (2026-05-30): Section 20 — SAML Certificate Lifecycle Management — Proactive Monitoring & Expiry Alerting. Closes the design gap for the `cert_expiry_check` cron referenced (but not specified) in §8.1. Covers both SP certificate (FORM-generated, 2-year RSA-2048) and IdP certificate (customer-generated, validity varies) expiry monitoring. Eight ALTER TABLE columns on `tenant_sso_configs`: four expiry-metadata columns (`saml_sp_cert_expires_at`, `saml_sp_cert_fingerprint`, `saml_idp_cert_expires_at`, `saml_idp_cert_fingerprint`), two partial indexes on those columns, and two state-machine columns (`cert_alert_tier` ENUM 7-value, `cert_rotation_state` ENUM 5-value) with backfill Edge Function procedure. Seven-tier alert escalation matrix (t90 through expired) with PagerDuty severity mapping (null → low → warning → error → critical) and de-duplication rule (7-day cooldown per tier per tenant). Full TypeScript Worker cron implementation (`cert-expiry-check.ts`): daily 02:00 UTC via Cloudflare Workers Cron; queries active SAML tenants with cert expiring ≤90 days; handles both cert classes in one pass; emits DEC-030 before PagerDuty; updates `cert_alert_tier` and `cert_alert_last_sent_at` after each alert; separate `handleExpired` path for P0 post-expiry case. Three Resend email templates: CRT-01 (`sso-cert-expiry-t30`, first customer-visible notification with cert-class-conditional body), CRT-02 (`sso-cert-expiry-t7`, urgent 7-day), CRT-03 (`sso-cert-expiry-expired`, post-expiry email-fallback active notification). Five-state rotation state machine (`idle → notified → rotation_in_progress → complete`; `overdue` branch from `notified` at t7 if unresponsive). Four new DEC-030 HMAC-chained events: `sso.cert_expiry_alert` (MEDIUM/HIGH/CRITICAL by tier, 7yr), `sso.cert_expired` (CRITICAL, 7yr), `sso.cert_uploaded` (STANDARD, 7yr — IdP cert upload distinct from rotation), `sso.cert_monitor_error` (HIGH, 7yr — cron self-monitoring). FORM internal admin dashboard Certificate Panel spec: SP + IdP status badges, alert tier, rotation state. Tenant admin in-app banner at ≤30 days IdP cert expiry. SOC 2 evidence: CC6-E-CERT-001 through CC6-E-CERT-004 (audit_log exports, config snapshot, cron execution log). SOC 2 mapping: CC6.1 (credential lifecycle), CC7.2 (anomaly monitoring), CC8.1 (change management). 4 open questions (OQ-SSO-20.1 through OQ-SSO-20.4: metadata auto-refresh, self-signed cert policy, self-service upload UI, 2yr validity review). 11-item implementation checklist (8× P0 M4, 3× P1 M5).*

*v1.3 additions (2026-05-30): Section 21 — Google Workspace Directory API Integration for Group Sync. Closes G-005 from §9 (Google Directory API integration for group sync — designed but not implemented). Root cause documented: Google OIDC deliberately omits group membership from ID token regardless of scopes; Admin SDK Directory API + service account + domain-wide delegation (DWD) is the canonical industry solution. Pull-model design (FORM calls Google on each login) vs. §19 push-model (SCIM); complementary — Google Workspace tenants who cannot use the three §19 workarounds (Okta bridge, Entra bridge, SAML groups attribute) use this path instead. Architecture: OIDC callback Worker checks `google_directory_group_cache` (5-minute TTL); on CACHE MISS calls `GET admin.googleapis.com/admin/directory/v1/groups?userKey={email}`; resolved groups feed directly into existing `scim_group_role_mappings` lookup (§19.3) — no change to role resolution engine (§5.4). Schema: ALTER TABLE `tenant_sso_configs` — three new columns (`google_service_account_email`, `google_directory_admin_email`, `google_directory_sync_enabled` + last_sync_at + sync_error); `google_service_account_json BYTEA` already exists (§4.2); partial index on `google_directory_sync_enabled = true` tenants; new `google_directory_group_cache` table (UNIQUE on `tenant_id, user_email`, `expires_at` as generated column = `fetched_at + 5min`, RLS tenant isolation + form_system write-only + form_api read-own-tenant; pg_cron cleanup every 10 minutes). Full TypeScript implementation (`apps/api-gateway/src/sso/google-directory-groups.ts`): `resolveGoogleGroups()` cache-first lookup with 10-minute grace window on API failure (stale cache used rather than blocking login); `fetchAndCacheGroups()` pagination loop (200 per page, handles nextPageToken); `mintServiceAccountToken()` using SubtleCrypto RSASSA-PKCS1-v1_5 + JWT grant to `oauth2.googleapis.com/token`; `importPrivateKey()` PKCS8 import for Workers WebCrypto. Service account access token KV caching (OQ-SSO-21.3: `{tenant_id}_google_token`, TTL 55 min, avoids 200ms JWT-to-token overhead per login). Customer configuration: 4-step process (GCP project + service account creation; Admin Console domain-wide delegation with `admin.directory.group.readonly` scope only; impersonation account designation; CSM-mediated credential upload via secure portal — never email). Credential rotation protocol: 7-step process; key deletion in GCP before replacement on compromise path. Eight DEC-030 HMAC-chained audit events: `sso.google_directory_sync_success` (STANDARD), `sso.google_directory_sync_cache_hit` (STANDARD), `sso.google_directory_sync_error` (HIGH), `sso.google_directory_credential_uploaded` (HIGH), `sso.google_directory_credential_rotated` (HIGH), `sso.google_directory_sync_enabled` (HIGH), `sso.google_directory_sync_disabled` (HIGH), `sso.google_directory_sync_validated` (STANDARD) — all 7yr; `user_email_hash` SHA-256 not plaintext in all events. Five alerting rules AL-SSO-GDIR-01 through AL-SSO-GDIR-05 (P2: error rate > 20% / 3 consecutive per tenant; P3: P95 latency > 3,000ms / unvalidated credential). GDPR: Google Workspace DPA covers Directory API; conditional sub-processor entry in SOC2_READINESS §44; data minimisation enforced (only id/email/name stored, 5-minute TTL); Art. 14 transparency obligation on customer controller. SOC 2: CC6.1 (credential lifecycle), CC6.2 (actor_id + role on credential operations), CC6.3 (5-minute TTL enforces timely role revocation), CC7.2 (5 alerting rules on sync health), CC9.2 (Google DPA + conditional sub-processor entry), C1.1 (email hash + group data not exposed to tenant_manager). Four evidence artefacts CC6-E-GDIR-001 through CC6-E-GDIR-004. Four open questions OQ-SSO-21.1 through OQ-SSO-21.4 (background sync M6 decision; >10,000 group scale test; KV access-token caching design; Cloud Identity Groups deferral). Gap status: G-005 🔴 Open → 🟡 Authored. 12-item implementation checklist: 8× P0 M4, 4× P1 M4.*

*v1.4 additions (2026-06-01): Section 22 — High-Scale Session Revocation Architecture — KV-Backed Revocation Cache. Closes G-009 from §9 (session blocklist persistence bottleneck at >100-seat enterprise deployments — needed before first large-tenant go-live). Root cause: the `session_blocklist` Supabase table in §6.3 performs a synchronous SELECT on every authenticated request; at ≥100 RPS from enterprise concurrent users this saturates the Supabase connection pool and adds 15–25 ms per-request latency. Two-tier revocation architecture: Cloudflare KV (`SESSION_REVOCATION_KV` namespace, separate from `SSO_KV` for quota isolation) as the hot cache (~1 ms reads); Supabase `session_blocklist` as the authoritative audit trail (written on every revocation, read only as fallback). Four KV key types: `revoke:tenant:{tenant_id}:all` (TTL = max_jwt_ttl; single write nukes all tenant sessions — O(1) regardless of session count), `revoke:user:{tenant_id}:{user_id}` (TTL = 24h; user-level revocation), `revoke:session:{session_id}` (TTL = remaining JWT exp), `revoke:jti:{jti}` (TTL = remaining JWT exp; integrates the "Redis" JTI blocklist described in §12.7.4 into existing Cloudflare KV infrastructure). Per-request validation sequence: JWT signature + exp → three-way parallel KV GET (tenant/user/session) → optional JTI KV GET (opt-in per tenant) → handler; expected total overhead ≤ 3 ms (vs. 20 ms Supabase). Bulk SCIM revocation path (`handleBulkScimRevocation()` in `apps/api-gateway/src/sso/session-revocation-kv.ts`): batched KV writes in groups of 50 using Promise.all (Cloudflare Workers supports ≥50 concurrent KV ops per request); 1,000-user deactivation completes in ~100 ms (20 batches × ~5 ms) vs. current >10 s Supabase INSERT SELECT path. Tenant nuke function (`nukeTenantSessions()`): single KV write at `revoke:tenant:{tenant_id}:all`, replaces the unbounded `INSERT INTO session_blocklist SELECT ... WHERE tenant_id = $1` in §8.3, and becomes the canonical P0 incident response action (INCIDENT_RESPONSE.md R-01, step "invalidate all active sessions"). Schema change: one new column `kv_sync_status revocation_kv_status DEFAULT 'pending'` on `session_blocklist` (ENUM: pending/synced/failed), backfilled asynchronously; tracks whether a revocation has been reflected in KV for audit purposes. Migration window (30 days after deploy): on KV MISS, fallback to Supabase `session_blocklist` SELECT to catch sessions revoked before the KV layer was deployed; after 30 days the fallback is removed and sessions revoked pre-deploy have expired naturally. Five DEC-030 HMAC-chained audit events: `session.bulk_revocation_started` (HIGH, 7yr — emitted before first KV write; includes user_count + scim_request_id), `session.bulk_revocation_complete` (HIGH, 7yr — includes sessions_revoked count + duration_ms), `session.tenant_nuke_started` (CRITICAL, 7yr — two-person authorization required; authorised_by array), `session.tenant_nuke_complete` (CRITICAL, 7yr), `session.revocation_kv_sync_error` (HIGH, 7yr — KV write failure; fallback to Supabase-only path; triggers PagerDuty P1). Two alerting rules: AL-REVOKE-01 (P1 — `session.revocation_kv_sync_error` error rate > 1% over 5 min; runbook: KV namespace binding missing or quota exceeded), AL-REVOKE-02 (P2 — bulk revocation duration_ms > 5,000; runbook: batch size may need reducing or KV quota check). SOC 2 mapping: CC6.3 (logical access removed on a timely basis — KV revocation < 100 ms vs. 15-minute JWT residual window for standard path; tenant nuke < 5 ms), CC7.2 (anomaly monitoring — AL-REVOKE-01/02 on sync health), CC7.3 (response to anomalies — KV sync error triggers PagerDuty P1). Two evidence artefacts: CC6-E-REV-001 (`audit_log` export: all `session.bulk_revocation_*` and `session.tenant_nuke_*` events for observation period), CC6-E-REV-002 (`session_blocklist` kv_sync_status distribution: `SELECT kv_sync_status, count(*) FROM session_blocklist WHERE blocked_at BETWEEN $obs_start AND $obs_end GROUP BY 1` — confirms >99% synced). G-009 status: 🔴 Open → 🟡 Authored (design complete; implementation-pending M4). Implementation checklist: 11 tasks (8× P0 M4, 3× P1 M4/M5).*

---

## 23. Continuous Access Evaluation (CAE) & Shared Signals Framework (SSF) — Real-Time IdP Risk Event Processing

### 23.1 Problem Statement

§22 solved the FORM-initiated session revocation problem: when FORM itself decides to invalidate sessions — in response to a SCIM deprovisioning event, an admin action, or an emergency tenant nuke — the §22 KV layer ensures that those sessions are invalidated within milliseconds, long before the JWT's 15-minute TTL (§12) would naturally expire. However, §22 assumes that FORM is the initiator. There is a complementary and equally important gap: the identity provider itself continuously evaluates user risk and can determine, independently of any FORM-side action, that a user's access should be immediately revoked.

The following IdP security events, if they occur during an active FORM session, currently propagate to FORM only when the access token expires — up to 15 minutes later (§12 access token TTL). This is the residual validity window:

- **Account disabled or deleted at the IdP**: An HR system deprovisions an employee in Azure AD or Okta. The SCIM deprovisioning webhook (§5) may not arrive for several minutes if the IdP batches SCIM writes; and on some IdP configurations, deprovisioning does not trigger SCIM at all — it only invalidates the IdP-side session. During the SCIM lag window, the user's FORM access token remains valid.
- **Password reset**: The user's credential was compromised; IT initiates a forced password reset. The IdP invalidates all refresh tokens for that user. The FORM access token, already minted, continues to work until its TTL expires.
- **MFA credential removed or changed**: The user's authenticator was deleted by an admin (suspected device loss), or the user enrolled a new MFA factor (possible account takeover mid-enrollment). Neither event reaches FORM directly.
- **Session revoked from IdP admin console**: An admin explicitly terminates a user's SSO session in the Okta admin panel or Azure AD "Revoke sessions" action. The IdP-side session ends; the FORM access token is unaffected.
- **High-risk sign-in detected**: The IdP's risk engine flags the user's session as high-risk (impossible travel, leaked credentials, anomalous token usage). The IdP may downgrade the user's session confidence but cannot reach into downstream relying parties to enforce it.
- **Device compliance change**: The user's registered device falls out of compliance (MDM policy violation, certificate expired). Conditional Access at the IdP would block new logins but cannot reach FORM sessions already established.

Current FORM response time for all of the above: up to 15 minutes, bounded by access token TTL. The target after §23 is under 30 seconds from the moment the IdP emits the security event to the moment FORM's per-request revocation check (`isRevoked()`, §22.4) returns `true` for that user's sessions.

This 30-second target is achievable because: (a) IdPs push events to FORM's webhook within seconds of the triggering action, (b) the FORM worker processes the SET and writes to the §22 KV in under 200 ms, and (c) `isRevoked()` reads from KV in under 3 ms on the next request. The pipeline latency is dominated by IdP-side event dispatch, which is typically 5–20 seconds after the triggering admin action.

### 23.2 Standards Overview

The problem of communicating risk signals between identity systems is addressed by two complementary IETF/OpenID specifications:

**IETF Shared Signals Framework (SSF)** — `draft-ietf-sharedsignals-framework` — defines the transport and delivery model. SSF introduces the concepts of a Transmitter (the IdP, which detects security events) and a Receiver (FORM, which acts on them). It specifies how Receivers register streams, negotiate delivery mode (push vs. poll), and how Transmitters batch and deliver Security Event Tokens. SSF is protocol-agnostic about what the events mean; it is purely a transport specification.

**OpenID CAEP (Continuous Access Evaluation Protocol)** defines the event vocabulary carried over SSF for access evaluation events. CAEP event types include `session-revoked`, `credential-change`, `token-claims-change`, `device-compliance-change`, and `account-disabled`. CAEP is the vocabulary that Okta, Microsoft, and other modern enterprise IdPs expose through their SSF endpoints.

**OpenID RISC (Risk and Incident Sharing and Collaboration)** is an earlier but widely deployed Google-authored vocabulary, also carried over SSF transport, focused on account security risk events. RISC defines `hijacking`, `account-disabled`, `account-enabled`, and `account-purged`. Google Workspace OIDC tenants (§4.2) use RISC; Okta and Entra use CAEP. The two vocabularies overlap in semantics but not in event type URIs, so FORM's handler must recognise both namespaces.

**Security Event Tokens (SET)** — RFC 8417 — are the wire format for individual events. A SET is a signed JWT. Its payload contains a standard `iss` (IdP identifier), `iat`, `jti` (unique event ID), `sub` (the affected user's IdP subject identifier), and an `events` claim — a JSON object keyed by event type URI, with event-specific payload as the value. The IdP signs the SET using its private key; FORM verifies the signature using the IdP's JWKS, which is published at a well-known endpoint.

The relationship between the layers is:

```
┌──────────────────────────────────────────────────────────────┐
│  Event Vocabulary Layer                                       │
│  ┌──────────────────────┐  ┌──────────────────────────────┐  │
│  │  OpenID CAEP          │  │  OpenID RISC (Google)        │  │
│  │  (Okta, Entra, etc.)  │  │  (Google Workspace OIDC)     │  │
│  └──────────────────────┘  └──────────────────────────────┘  │
├──────────────────────────────────────────────────────────────┤
│  Token Format Layer                                          │
│  RFC 8417 Security Event Tokens (SET) — signed JWTs          │
├──────────────────────────────────────────────────────────────┤
│  Transport Layer                                             │
│  IETF SSF (draft-ietf-sharedsignals-framework)               │
│  Push delivery (IdP → FORM webhook) or Poll (FORM → IdP)     │
└──────────────────────────────────────────────────────────────┘
```

**Microsoft Entra Continuous Access Evaluation** is Microsoft's proprietary but CAEP-aligned implementation. Entra CAE does not use the SSF push model for third-party receivers in all configurations; instead it delivers events via Azure Event Grid webhooks. The SET format and CAEP event vocabulary are the same, but the registration mechanism differs (Microsoft Graph API rather than an SSF transmitter configuration endpoint). This distinction is covered in §23.6.

### 23.3 FORM as SSF Stream Receiver

#### 23.3.1 Stream Registration Model

FORM registers as an SSF Receiver at each IdP that supports the protocol. A "stream" in SSF terminology is a persistent subscription between one Transmitter (IdP) and one Receiver (FORM). Each stream is scoped to a single tenant and a single IdP. A tenant using Okta for SSO has exactly one CAEP stream; a tenant using Entra has exactly one Entra CAE stream. A tenant using Google Workspace OIDC has one RISC stream. These are stored independently in the `tenant_sso_configs` table (new columns in §23.5).

At stream creation, FORM provides the IdP with:

1. A **push delivery endpoint URL**: `https://api.form.coach/enterprise/v1/sso/caep-receiver/{tenant_id}` — the Cloudflare Worker endpoint that receives SET payloads from the IdP via HTTP POST.
2. A **verification token**: a random 32-byte token, URL-safe base64-encoded, that the IdP includes in a verification challenge before stream activation. FORM must echo this token back to complete the handshake. The token is stored in `tenant_sso_configs.caep_webhook_secret` and is separate from the HMAC signing secret used for payload integrity.
3. A **set of requested event types**: FORM requests all CAEP and RISC event types it recognises (listed in §23.4). The IdP may deliver a subset based on its supported vocabulary.

Stream registration is performed by FORM's admin dashboard backend when a tenant enables CAEP in their SSO settings. The registration flow calls the IdP's transmitter configuration endpoint (discovered from the SSF well-known metadata document at `{issuer}/.well-known/ssf-configuration`) and stores the returned `stream_id` in `caep_stream_id`.

#### 23.3.2 CAEP Receiver Worker Endpoint

The receiving endpoint is a dedicated Cloudflare Worker route:

```
POST /enterprise/v1/sso/caep-receiver/{tenant_id}
```

This endpoint is not part of the authenticated API surface — it is called by IdPs, not by tenant users or FORM clients. Authentication is provided by the SET signature itself (JWKS validation) and the HMAC webhook secret. The processing pipeline is:

```
Request arrives
  │
  ├─ 1. Extract tenant_id from URL path
  │       Validate tenant exists + has active CAEP config (Supabase lookup, cached in SSO_KV, TTL 5 min)
  │       On miss: 404 (do not reveal whether tenant exists)
  │
  ├─ 2. HMAC-SHA256 webhook secret validation
  │       Read X-SSF-Signature header (or IdP-specific equivalent)
  │       Compute HMAC-SHA256(raw_request_body, caep_webhook_secret)
  │       Constant-time comparison
  │       On failure: 401 + emit sso.caep_stream_error audit event
  │
  ├─ 3. Parse SET JWT header — extract kid
  │       Fetch JWKS from SSO_KV (key: caep_jwks:{tenant_id}:{iss}, TTL 1h)
  │       On KV miss: fetch from IdP's JWKS endpoint, store in KV
  │       On kid mismatch after fresh fetch: reject + emit HIGH sso.caep_stream_error
  │
  ├─ 4. Verify SET JWT signature using resolved JWKS key
  │       Verify iss matches expected IdP for this tenant
  │       Verify iat is not in the future (clock skew tolerance: 60 s)
  │       Verify jti not in caep_events (replay prevention, Supabase lookup)
  │
  ├─ 5. Extract event type from events claim (first key of events object)
  │       Extract sub (IdP subject) and any event-specific payload fields
  │
  ├─ 6. Resolve FORM user_id from idp_subject
  │       SELECT user_id FROM tenant_users WHERE tenant_id = $1 AND idp_subject = $2
  │       (Same lookup as SAML JIT, §6.2 — idp_subject already indexed)
  │
  ├─ 7. Route to action handler (§23.4 mapping table)
  │       Action handler writes to SESSION_REVOCATION_KV (§22.5 key patterns)
  │
  ├─ 8. Insert into caep_events (idempotent: ON CONFLICT (set_jti) DO NOTHING)
  │
  ├─ 9. Emit DEC-030 sso.caep_event_received audit event
  │
  └─ 10. Return 202 Accepted
         (202, not 200 — signals to IdP that FORM received the event but processing
          is asynchronous; prevents IdP retry storms on any transient downstream delay)
```

The choice of 202 Accepted is deliberate. SSF receivers are expected to return 2xx promptly to acknowledge receipt. If FORM returns a non-2xx status, well-behaved IdP transmitters will retry with exponential backoff, potentially delivering the same event multiple times. The `set_jti` UNIQUE constraint in `caep_events` (§23.5) makes processing idempotent, but generating unnecessary retry traffic is wasteful and could trigger FORM's own rate limiting. Returning 202 immediately and processing synchronously within the Worker request context (Cloudflare Workers have a 30-second CPU time limit, well above the ~200 ms required for this pipeline) avoids both problems.

#### 23.3.3 Tenant Configuration Cache

The per-tenant CAEP configuration (whether CAEP is enabled, the `caep_webhook_secret`, the expected IdP issuer) is loaded from Supabase on the first request and cached in `SSO_KV` with a 5-minute TTL under the key `caep_config:{tenant_id}`. This avoids a Supabase round-trip on every incoming CAEP event. The cache is invalidated proactively when a tenant admin updates their CAEP configuration via the admin dashboard (the dashboard backend writes to Supabase and issues a KV delete for the cache key).

The JWKS for each IdP is cached separately under `caep_jwks:{tenant_id}:{iss}` with a 1-hour TTL — consistent with the SSO_KV JWKS caching pattern used in §12.7. On a `kid` miss in the cached JWKS (possible during IdP key rotation), FORM fetches the JWKS endpoint fresh and updates the KV entry. If the fresh JWKS also does not contain the presented `kid`, the SET is rejected and a HIGH severity `sso.caep_stream_error` audit event is emitted. This is the correct response: a `kid` that cannot be resolved after a fresh JWKS fetch is either a misconfiguration or a signature from an unknown key — both cases warrant rejection.

### 23.4 Event Type to Action Mapping

The `events` claim in the SET JWT is a JSON object where keys are event type URIs defined by the CAEP or RISC vocabulary. FORM extracts the first key of the object (SSF permits only one event type per SET in most IdP implementations, though the spec allows multiple) and dispatches to the corresponding action handler.

| # | Event Type URI | Standard | Triggered By | FORM Action | KV Pattern (§22) |
|---|---|---|---|---|---|
| 1 | `https://schemas.openid.net/secevent/caep/event-type/session-revoked` | CAEP | Admin revokes specific IdP session | Revoke the specific FORM session identified by the `session_id` field in the event payload (if present) or fall back to user-level revocation | `revoke:session:{session_id}` or `revoke:user:{tenant_id}:{user_id}` |
| 2 | `https://schemas.openid.net/secevent/caep/event-type/credential-change` | CAEP | Password reset, MFA credential added/removed/modified | Revoke all active sessions for the user across all devices | `revoke:user:{tenant_id}:{user_id}` (TTL 24h) |
| 3 | `https://schemas.openid.net/secevent/caep/event-type/token-claims-change` | CAEP | Role change, group membership change at IdP that affects token claims | Set `force_reauth:{tenant_id}:{user_id}` flag in SESSION_REVOCATION_KV (TTL 24h); on next request, return 401 with `WWW-Authenticate: Bearer error="insufficient_user_authentication"` to prompt re-login | `force_reauth:{tenant_id}:{user_id}` (TTL 24h) |
| 4 | `https://schemas.openid.net/secevent/caep/event-type/account-disabled` | CAEP | Account suspended in IdP (leave of absence, HR action, security hold) | Revoke all user sessions via user-level KV key; set `account_suspended:{tenant_id}:{user_id}` in KV (TTL 72h, renewably checked); write `status = 'suspended'` to `tenant_users` | `revoke:user:{tenant_id}:{user_id}` + `account_suspended:{tenant_id}:{user_id}` |
| 5 | `https://schemas.openid.net/secevent/caep/event-type/account-enabled` | CAEP | Account re-enabled in IdP (suspension lifted) | Clear `account_suspended:{tenant_id}:{user_id}` KV key; update `tenant_users.status = 'active'`; emit audit event. No automatic session restoration — user must log in again | KV delete `account_suspended:{tenant_id}:{user_id}` |
| 6 | `https://schemas.openid.net/secevent/caep/event-type/account-purged` | CAEP | Account permanently deleted at IdP | Full offboarding: revoke all sessions (user-level KV key); set `tenant_users.status = 'deleted_by_idp'`; set `tenant_users.deleted_at = NOW()`; queue GDPR deletion workflow (async Worker queue event); emit CRITICAL sso.caep_user_purged audit event. Two-person rule applies to any manual override of a purge action | `revoke:user:{tenant_id}:{user_id}` + GDPR queue |
| 7 | `https://schemas.openid.net/secevent/caep/event-type/device-compliance-change` | CAEP | MDM reports device fell out of compliance | Conditional: if `tenant_sso_configs.session_policy->>'require_device_compliance' = 'true'` then revoke all user sessions; otherwise write `force_reauth:{tenant_id}:{user_id}` (softer response — force re-auth so Conditional Access at the IdP can block the next login if needed) | Conditional: `revoke:user:{tenant_id}:{user_id}` or `force_reauth:{tenant_id}:{user_id}` |
| 8 | `https://schemas.openid.net/secevent/risc/event-type/hijacking` | RISC (Google) | Google detects active session hijacking (token theft, impossible travel, etc.) | Immediate session revocation for all user sessions; emit P0 PagerDuty alert (AL-CAEP-04); emit CRITICAL audit event. Applicable only to Google Workspace OIDC tenants (§4.2) | `revoke:user:{tenant_id}:{user_id}` + PagerDuty P0 |

**`token-claims-change` and `force_reauth`:** The `force_reauth` KV pattern is a new key type not present in §22. It does not revoke the session outright — the session remains technically valid — but it causes `isRevoked()` to return a special `FORCE_REAUTH` signal rather than a hard `true`. The Worker's authentication middleware must handle this signal by returning a 401 with the `insufficient_user_authentication` error code, prompting the client to initiate a new SSO flow. This preserves the user's session context (draft workouts, current screen) while ensuring that the fresh SSO assertion reflects the updated claims. The TTL of 24 hours acts as a backstop: even if the client does not re-authenticate, the TTL eventually falls off and the next regular token expiry (15 minutes) handles it.

**`account-purged` and the GDPR deletion workflow:** The `account-purged` event starts the GDPR data deletion clock. FORM must complete deletion of all user personal data within 30 days (per the GDPR data processing agreement). The `account-purged` action handler enqueues a message to a Cloudflare Workers Queue (`GDPR_DELETION_QUEUE`) with the `tenant_id`, `user_id`, and `idp_subject`. A separate background Worker consumer processes the queue, orchestrates deletion across Supabase tables, and emits a `gdpr.deletion_completed` audit event when done. The `sso.caep_user_purged` audit event is the evidence artefact that the GDPR deletion SLA timer started; the `gdpr.deletion_completed` event is the evidence that it was met.

The action handler for event types 1–8 is implemented in `apps/api-gateway/src/sso/caep-action-handler.ts`. The Worker endpoint (`caep-receiver.ts`) calls `dispatchCaepAction()` with the resolved user_id, tenant_id, event type, and event payload. The function returns an `action_taken` string that is stored in `caep_events.action_taken` for audit and SOC 2 evidence.

```typescript
// In: apps/api-gateway/src/sso/caep-action-handler.ts

export type CaepActionResult = {
  action_taken: string;
  kv_keys_written: string[];
};

export async function dispatchCaepAction(
  kv: KVNamespace,
  supabase: SupabaseClient,
  params: {
    tenantId:    string;
    userId:      string | null;   // null only for account-purged before user_id resolved
    eventType:   string;
    eventPayload: Record<string, unknown>;
    sessionPolicy: TenantSessionPolicy;
  }
): Promise<CaepActionResult> {
  const { tenantId, userId, eventType, eventPayload, sessionPolicy } = params;

  switch (eventType) {
    case 'https://schemas.openid.net/secevent/caep/event-type/session-revoked': {
      const sessionId = eventPayload['session_id'] as string | undefined;
      if (sessionId) {
        await kv.put(`revoke:session:${sessionId}`, '1', { expirationTtl: 3600 });
        return { action_taken: 'session_revoked', kv_keys_written: [`revoke:session:${sessionId}`] };
      }
      // Fall through to user-level revocation if no specific session_id
      await kv.put(`revoke:user:${tenantId}:${userId}`, '1', { expirationTtl: 86400 });
      return { action_taken: 'user_sessions_revoked_no_session_id', kv_keys_written: [`revoke:user:${tenantId}:${userId}`] };
    }

    case 'https://schemas.openid.net/secevent/caep/event-type/credential-change': {
      await kv.put(`revoke:user:${tenantId}:${userId}`, '1', { expirationTtl: 86400 });
      return { action_taken: 'user_sessions_revoked_credential_change', kv_keys_written: [`revoke:user:${tenantId}:${userId}`] };
    }

    case 'https://schemas.openid.net/secevent/caep/event-type/token-claims-change': {
      await kv.put(`force_reauth:${tenantId}:${userId}`, '1', { expirationTtl: 86400 });
      return { action_taken: 'force_reauth_token_claims_change', kv_keys_written: [`force_reauth:${tenantId}:${userId}`] };
    }

    case 'https://schemas.openid.net/secevent/caep/event-type/account-disabled': {
      await Promise.all([
        kv.put(`revoke:user:${tenantId}:${userId}`, '1', { expirationTtl: 86400 }),
        kv.put(`account_suspended:${tenantId}:${userId}`, '1', { expirationTtl: 259200 }), // 72h
      ]);
      await supabase
        .from('tenant_users')
        .update({ status: 'suspended' })
        .eq('tenant_id', tenantId)
        .eq('id', userId);
      return {
        action_taken: 'user_suspended_sessions_revoked',
        kv_keys_written: [`revoke:user:${tenantId}:${userId}`, `account_suspended:${tenantId}:${userId}`],
      };
    }

    case 'https://schemas.openid.net/secevent/caep/event-type/account-enabled': {
      await kv.delete(`account_suspended:${tenantId}:${userId}`);
      await supabase
        .from('tenant_users')
        .update({ status: 'active' })
        .eq('tenant_id', tenantId)
        .eq('id', userId);
      return { action_taken: 'account_unsuspended', kv_keys_written: [] };
    }

    case 'https://schemas.openid.net/secevent/caep/event-type/account-purged': {
      await kv.put(`revoke:user:${tenantId}:${userId}`, '1', { expirationTtl: 86400 });
      await supabase
        .from('tenant_users')
        .update({ status: 'deleted_by_idp', deleted_at: new Date().toISOString() })
        .eq('tenant_id', tenantId)
        .eq('id', userId);
      // Enqueue GDPR deletion workflow
      // env.GDPR_DELETION_QUEUE.send({ tenantId, userId, triggeredBy: 'caep_account_purged' });
      return { action_taken: 'account_purged_gdpr_queued', kv_keys_written: [`revoke:user:${tenantId}:${userId}`] };
    }

    case 'https://schemas.openid.net/secevent/caep/event-type/device-compliance-change': {
      const requireDeviceCompliance = sessionPolicy.require_device_compliance ?? false;
      if (requireDeviceCompliance) {
        await kv.put(`revoke:user:${tenantId}:${userId}`, '1', { expirationTtl: 86400 });
        return { action_taken: 'user_sessions_revoked_device_noncompliant', kv_keys_written: [`revoke:user:${tenantId}:${userId}`] };
      }
      await kv.put(`force_reauth:${tenantId}:${userId}`, '1', { expirationTtl: 86400 });
      return { action_taken: 'force_reauth_device_compliance_soft', kv_keys_written: [`force_reauth:${tenantId}:${userId}`] };
    }

    case 'https://schemas.openid.net/secevent/risc/event-type/hijacking': {
      await kv.put(`revoke:user:${tenantId}:${userId}`, '1', { expirationTtl: 86400 });
      // PagerDuty P0 alert emitted by the caller (caep-receiver.ts) after this function returns,
      // so the alert is guaranteed to fire even if the KV write above partially fails.
      return { action_taken: 'user_sessions_revoked_risc_hijacking', kv_keys_written: [`revoke:user:${tenantId}:${userId}`] };
    }

    default:
      // Unknown event type — log and return no-op; do not reject the SET
      // (future IdP additions should not cause 4xx responses and trigger retry storms)
      return { action_taken: `unrecognised_event_type_noop:${eventType}`, kv_keys_written: [] };
  }
}
```

The `isRevoked()` function from §22.4 must be extended to check the two new KV key patterns (`force_reauth:*` and `account_suspended:*`) in its parallel reads. The `force_reauth` check returns a new `RevocationStatus.FORCE_REAUTH` value rather than `RevocationStatus.REVOKED`, so the calling middleware can distinguish between "session hard-revoked" (return 401 and destroy client-side session) and "soft force re-auth" (return 401 with re-auth prompt but preserve client-side session context).

### 23.5 Schema Changes

#### 23.5.1 New Columns on `tenant_sso_configs`

```sql
-- migrations/YYYYMMDD_tenant_sso_configs_caep.sql

ALTER TABLE tenant_sso_configs
  ADD COLUMN caep_stream_id       TEXT,
  ADD COLUMN caep_delivery_mode   TEXT
    CHECK (caep_delivery_mode IN ('push', 'poll')),
  ADD COLUMN caep_webhook_secret  BYTEA,    -- AES-256-GCM encrypted at rest; managed by §13 secret rotation
  ADD COLUMN caep_status          TEXT NOT NULL DEFAULT 'inactive'
    CHECK (caep_status IN ('inactive', 'active', 'error', 'rate_limited')),
  ADD COLUMN caep_last_event_at   TIMESTAMPTZ,
  ADD COLUMN caep_error_count     INT NOT NULL DEFAULT 0;

COMMENT ON COLUMN tenant_sso_configs.caep_stream_id IS
  'IdP-assigned stream identifier returned at stream registration. NULL if CAEP not yet registered.';
COMMENT ON COLUMN tenant_sso_configs.caep_delivery_mode IS
  'Push: IdP POSTs SETs to FORM webhook. Poll: FORM polls IdP transmitter endpoint. '
  'Push is strongly preferred; poll is fallback for IdPs that do not support push.';
COMMENT ON COLUMN tenant_sso_configs.caep_webhook_secret IS
  'HMAC-SHA256 signing secret for push delivery integrity validation. '
  'AES-256-GCM encrypted at rest using the tenant KMS key (§13). '
  'Never exposed in API responses; write-only from admin dashboard.';
COMMENT ON COLUMN tenant_sso_configs.caep_status IS
  'Stream health: inactive (not registered), active (receiving events), '
  'error (SET validation failures, see caep_error_count), rate_limited (>1000 events/h).';
COMMENT ON COLUMN tenant_sso_configs.caep_error_count IS
  'Consecutive SET processing errors. Stream is paused when this exceeds 10. '
  'Reset to 0 on next successful event processing.';

-- Partial index: fast lookup of tenants with active CAEP streams
CREATE INDEX CONCURRENTLY idx_tenant_sso_configs_caep_active
  ON tenant_sso_configs (tenant_id)
  WHERE caep_status = 'active';
```

#### 23.5.2 New Table `caep_events`

```sql
-- Same migration file

CREATE TABLE caep_events (
  id              UUID        PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id       UUID        NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
  user_id         UUID        REFERENCES tenant_users(id) ON DELETE SET NULL,
  -- user_id is nullable: account-purged events delete the user row before the caep_events
  -- insert; ON DELETE SET NULL preserves the event record as evidence even after user deletion.
  set_jti         TEXT        NOT NULL UNIQUE,
  -- SET JWT ID from the jti claim. UNIQUE constraint is the primary replay-prevention mechanism.
  event_type      TEXT        NOT NULL,
  idp_subject     TEXT        NOT NULL,
  -- IdP subject identifier (e.g., Okta user ID, Entra object ID).
  -- Stored as SHA-256 hash — see §23.7 privacy note.
  action_taken    TEXT        NOT NULL,
  processed_at    TIMESTAMPTZ NOT NULL DEFAULT now(),
  raw_set_hash    TEXT        NOT NULL
  -- SHA-256 of the raw SET JWT bytes. Provides evidence that the specific token was received
  -- without storing the token itself (which may contain PII in claims).
);

-- Performance indexes
CREATE INDEX idx_caep_events_tenant_processed
  ON caep_events (tenant_id, processed_at DESC);
-- Supports audit log viewer pagination in admin dashboard (§23 tab)

CREATE UNIQUE INDEX idx_caep_events_set_jti
  ON caep_events (set_jti);
-- Redundant with PRIMARY KEY unique but explicit for documentation clarity;
-- also used by ON CONFLICT (set_jti) DO NOTHING upsert pattern.

COMMENT ON TABLE caep_events IS
  'Immutable log of every CAEP/RISC Security Event Token received and processed by FORM. '
  'Retention: 7 years per DEC-030. Soft-deleted records are not supported — '
  'this table is append-only for compliance purposes.';
```

**RLS policies for `caep_events`:**

```sql
-- Enable RLS
ALTER TABLE caep_events ENABLE ROW LEVEL SECURITY;

-- form_system (SCIM handler + CAEP Worker) can INSERT and SELECT
CREATE POLICY caep_events_form_system
  ON caep_events
  FOR ALL
  TO form_system
  USING (true)
  WITH CHECK (true);

-- tenant_admin can SELECT their own tenant's events (admin dashboard audit log viewer)
CREATE POLICY caep_events_tenant_admin_select
  ON caep_events
  FOR SELECT
  TO tenant_admin
  USING (tenant_id = current_setting('app.current_tenant_id')::uuid);

-- form_api (the main application API role) has zero-row access — CAEP events are
-- internal infrastructure records, not application data.
CREATE POLICY caep_events_form_api_deny
  ON caep_events
  FOR ALL
  TO form_api
  USING (false);
```

### 23.6 IdP-Specific Implementation Notes

#### 23.6.1 Microsoft Entra CAE (Push via Event Grid)

Microsoft Entra's Continuous Access Evaluation implementation for third-party relying parties uses a different registration path than the SSF standard. Instead of discovering an SSF transmitter configuration endpoint, FORM registers via the Microsoft Graph API:

```
POST https://graph.microsoft.com/beta/identity/continuousAccessEvaluationPolicy/receivers
Authorization: Bearer {FORM service principal token}
Content-Type: application/json

{
  "notificationEndpoints": [
    {
      "subscriptionId": "{tenant_id}",
      "notificationUrl": "https://api.form.coach/enterprise/v1/sso/caep-receiver/{tenant_id}"
    }
  ]
}
```

Entra pushes events as `EventGridEvent` batches. The outer envelope is an array of Event Grid events; the `data` field of each event contains the SET JWT as a string. FORM's Entra-specific parser must unwrap the Event Grid envelope before proceeding to standard SET validation. The `EventGridEvent` itself carries an `Authorization: Bearer {token}` header; FORM validates this bearer token using Entra's published JWKS at `https://login.microsoftonline.com/common/discovery/v2.0/keys` (the standard Entra JWKS endpoint, the same one used for §3 SAML assertion signature verification). This is a two-layer validation: outer Event Grid bearer token (validates the delivery channel), inner SET JWT signature (validates the event payload).

Entra CAE access tokens carry two additional claims that are relevant to FORM's implementation. The `xms_cc` claim (value: `{"cp1": true}`) indicates that the client application declared CAE capability during token acquisition. FORM's admin dashboard SPA and mobile clients should include `client_capabilities=CP1` in their OAuth2/OIDC requests to Entra so that Entra knows FORM can handle CAE revocation signals; without this, Entra may issue longer-lived tokens without CAE coverage. The `xms_ssm` claim provides session-level management metadata that FORM currently does not use but should preserve for future session-specific revocation targeting.

#### 23.6.2 Okta CAEP (Push via SSF)

Okta fully implements the SSF standard. The transmitter configuration metadata is at:

```
GET https://{org}.okta.com/.well-known/ssf-configuration
```

This document returns the `transmitter_configuration_endpoint`, `jwks_uri`, and the list of supported delivery methods and event types. FORM uses the `transmitter_configuration_endpoint` to create a stream:

```
POST {transmitter_configuration_endpoint}
Authorization: Bearer {FORM API token, scoped to ssf.write}
Content-Type: application/json

{
  "delivery": {
    "method": "https://schemas.openid.net/secevent/risc/delivery-method/push",
    "endpoint_url": "https://api.form.coach/enterprise/v1/sso/caep-receiver/{tenant_id}"
  },
  "events_requested": [
    "https://schemas.openid.net/secevent/caep/event-type/session-revoked",
    "https://schemas.openid.net/secevent/caep/event-type/credential-change",
    "https://schemas.openid.net/secevent/caep/event-type/token-claims-change",
    "https://schemas.openid.net/secevent/caep/event-type/account-disabled",
    "https://schemas.openid.net/secevent/caep/event-type/account-enabled",
    "https://schemas.openid.net/secevent/caep/event-type/account-purged",
    "https://schemas.openid.net/secevent/caep/event-type/device-compliance-change"
  ]
}
```

Okta responds with a `stream_id` which is stored in `tenant_sso_configs.caep_stream_id`. Okta pushes individual SETs (not batches) to the FORM webhook, signed using its JWKS at `https://{org}.okta.com/oauth2/v1/keys` — the same JWKS endpoint used for Okta OIDC token verification in §4.1. FORM's `SSO_KV` already caches this JWKS for Okta tenants; the CAEP receiver shares the same KV entry under the same cache key to avoid redundant fetches.

Okta implements a stream verification step before activating push delivery. After stream creation, Okta sends a verification request to the FORM webhook containing a `verification_state` value. FORM must respond with the same `verification_state` echoed back, confirming that the endpoint is genuinely controlled by FORM. The webhook handler's step 2 (HMAC validation) is skipped for this initial verification request — Okta signs it differently — and FORM recognises the verification request by the presence of a `https://schemas.openid.net/secevent/sse/event-type/verification` event type in the `events` claim.

#### 23.6.3 Google RISC (Push Model)

Google RISC uses the Google Identity Toolkit API for stream registration. FORM registers via:

```
POST https://risc.googleapis.com/v1beta/stream:update
Authorization: Bearer {service account token, scope: https://www.googleapis.com/auth/risc.configuration.readonly}
Content-Type: application/json

{
  "delivery": {
    "delivery_method": "https://schemas.openid.net/secevent/risc/delivery-method/push",
    "url": "https://api.form.coach/enterprise/v1/sso/caep-receiver/{tenant_id}"
  },
  "events_requested": [
    "https://schemas.openid.net/secevent/risc/event-type/hijacking",
    "https://schemas.openid.net/secevent/risc/event-type/account-disabled",
    "https://schemas.openid.net/secevent/risc/event-type/account-enabled",
    "https://schemas.openid.net/secevent/risc/event-type/account-purged"
  ]
}
```

Google signs RISC SETs using keys published at `https://www.googleapis.com/identitytoolkit/v3/relyingparty/publicKeys`. These keys are distinct from the standard Google OIDC JWKS (`https://www.googleapis.com/oauth2/v3/certs`) used in §4.2 for ID token verification. FORM must cache both JWKS endpoints separately in `SSO_KV`. The RISC JWKS uses a non-standard format (RSA public keys returned as a JSON object keyed by `kid` rather than the standard `keys` array); FORM's JWKS parser must handle both formats.

Google RISC is only applicable to tenants using Google OIDC for SSO (§4.2). It is not applicable to Entra or Okta tenants, nor to tenants using Google Workspace with SAML (§3) — Google does not emit RISC events for SAML-authenticated sessions, only for Google OIDC sessions. This scope constraint is enforced in the FORM admin dashboard: the CAEP/RISC configuration panel only displays the Google RISC option for tenants whose `sso_type = 'oidc'` and `oidc_issuer` contains `accounts.google.com`.

### 23.7 Security Considerations

#### 23.7.1 SET Replay Prevention

The `set_jti` claim in a SET JWT is a unique identifier for that specific security event token. FORM stores the `set_jti` in the `caep_events` table with a UNIQUE constraint. Before processing any SET, FORM checks whether the `set_jti` is already present in `caep_events` using a Supabase SELECT with the index on `(set_jti)`. If found, FORM returns 202 Accepted without re-processing — the action has already been taken. This makes the endpoint idempotent regardless of IdP retry behaviour. The 202 response on replay (rather than 409 Conflict) is intentional: returning a 4xx to an IdP retry causes the IdP to log an error and may pause the stream.

The `set_jti` value stored in `caep_events` is stored as a SHA-256 hash (hex-encoded), not as the raw JTI string. This preserves replay-prevention semantics while avoiding storage of a value that could theoretically be used to correlate events across systems. The ON CONFLICT clause uses the hashed value as the uniqueness key.

#### 23.7.2 JWKS Cache Poisoning Prevention

The JWKS cache in `SSO_KV` is keyed by `caep_jwks:{tenant_id}:{iss}`. The TTL is capped at 1 hour. On every SET validation, FORM extracts the `kid` from the SET JWT header and looks it up in the cached JWKS. If the `kid` is not in the cached JWKS, FORM fetches the JWKS endpoint fresh and updates the KV entry. This handles key rotation without requiring manual intervention.

If the `kid` is not found even in the freshly fetched JWKS, FORM rejects the SET and emits a HIGH severity `sso.caep_stream_error` audit event with `error_type: 'jwks_kid_not_found'`. This condition could indicate: (a) the IdP rotated keys and FORM's cache is stale (handled by the fresh fetch above), (b) the `kid` in the SET was forged (attempted signature bypass — a serious security event), or (c) an IdP misconfiguration. The HIGH audit event ensures this is visible in SIEM and to the compliance team.

FORM never permits a SET to be "verified" with a self-provided or caller-supplied public key. The JWKS is always fetched from the IdP's published endpoint or served from the bounded KV cache. There is no API endpoint that allows a caller to register a JWKS override for a tenant.

#### 23.7.3 Webhook Authenticity

Before parsing the SET JWT, FORM validates the HMAC-SHA256 signature on the raw request body using `caep_webhook_secret`. This provides defence-in-depth: even if FORM's CAEP receiver URL were discovered, an attacker without the webhook secret cannot submit a forged SET that reaches the signature verification step. The HMAC validation uses constant-time comparison (via Web Crypto `timingSafeEqual`) to prevent timing oracle attacks.

The `caep_webhook_secret` is stored encrypted at rest in `tenant_sso_configs` using AES-256-GCM, managed by the tenant KMS key (§13 key management). It is never returned in any API response. The admin dashboard can rotate it (triggering a stream re-registration with the new secret), but cannot read it.

#### 23.7.4 Rate Limiting

FORM enforces a rate limit of 1,000 CAEP events per hour per tenant. This limit exists because a malfunctioning IdP configuration could generate spurious events at high volume, creating a denial-of-service risk against the CAEP receiver Worker and against `caep_events` write throughput. The rate is tracked using a Cloudflare KV counter with a sliding 1-hour window under the key `caep_rate:{tenant_id}:{hour_bucket}`. When the limit is exceeded, FORM returns 429 Too Many Requests, sets `tenant_sso_configs.caep_status = 'rate_limited'`, and triggers AL-CAEP-05 (P2 PagerDuty). The stream is paused until the DevOps team acknowledges the alert and resets the rate limit counter.

The 1,000 events/hour limit is generous for legitimate use: even a 10,000-seat enterprise with aggressive IT activity would not normally exceed a few hundred CAEP events per hour. An IdP that generates > 1,000 events/hour against a single tenant is almost certainly misconfigured.

#### 23.7.5 Tenant Isolation

The `tenant_id` in the webhook URL path is validated against the `sub_id` claim in the SET JWT's `sub_id` field (or the `iss`-derived tenant mapping in `tenant_sso_configs`). A SET delivered to tenant A's webhook with a `sub_id` or `iss` that resolves to tenant B is rejected immediately, and a CRITICAL severity `sso.caep_stream_error` audit event is emitted. This cross-tenant mismatch is never a legitimate IdP behaviour and always indicates either a misconfiguration (wrong webhook URL registered) or an active attack (attempting to inject a CAEP event into the wrong tenant's stream).

#### 23.7.6 Privacy and PII Minimisation

Raw SETs are never stored. The `raw_set_hash` field in `caep_events` stores only the SHA-256 hash of the raw SET JWT bytes (hex-encoded). This provides evidence that a specific token was received and processed without retaining the token's claims (which may include email addresses, device identifiers, or other PII). Similarly, `idp_subject` is stored as a SHA-256 hash rather than the raw IdP user identifier. The `set_jti` is also hashed. This means the `caep_events` table contains no PII in plaintext, consistent with the audit log PII policy established in §13.

### 23.8 DEC-030 Audit Events

Six new event types are added to `docs/AUDIT_LOG_SCHEMA.md`. All events are HMAC-chained per DEC-030. All have a 7-year retention period.

| Event Type | Severity | Retention | Key Metadata Fields | Trigger |
|---|---|---|---|---|
| `sso.caep_event_received` | STANDARD | 7 yr | `tenant_id`, `event_type`, `action_taken`, `set_jti_hash` (SHA-256), `idp_subject_hash` (SHA-256) | Emitted on every successfully processed CAEP/RISC event (steps 1–9 of the pipeline completed without error). This is the primary SOC 2 evidence record for CC6.3. |
| `sso.caep_session_revoked` | HIGH | 7 yr | `tenant_id`, `user_id_hash` (SHA-256), `session_id_hash` (SHA-256), `caep_event_type`, `set_jti_hash` | Emitted when a session is revoked specifically in response to a CAEP/RISC signal. Cross-references the `sso.caep_event_received` event via `set_jti_hash`. Complements the §22 `session.revocation_*` chain — CAEP-triggered revocations appear in both chains. |
| `sso.caep_user_suspended` | HIGH | 7 yr | `tenant_id`, `user_id_hash`, `idp_subject_hash`, `caep_event_type` | Emitted on `account-disabled` CAEP event. Records that the user's status was set to `suspended` in `tenant_users` as a direct result of an IdP signal. |
| `sso.caep_user_purged` | CRITICAL | 7 yr | `tenant_id`, `user_id_hash`, `idp_subject_hash`, `gdpr_deletion_queue_id` | Emitted on `account-purged` CAEP event. This is the GDPR deletion SLA start-of-clock event. The `gdpr_deletion_queue_id` links to the Cloudflare Workers Queue message ID so the deletion pipeline can be traced. Two-person rule: any manual intervention in the purge workflow (e.g., cancellation) requires two `form_admin` authorisers, enforced at the API layer identically to `nukeTenantSessions()`. |
| `sso.caep_stream_error` | HIGH | 7 yr | `tenant_id`, `error_type` (one of: `hmac_invalid`, `jwks_kid_not_found`, `set_signature_invalid`, `set_replay_detected`, `tenant_mismatch`, `rate_limit_exceeded`), `set_jti_hash` (if parseable) | Emitted on any SET validation failure. The `error_type` is a controlled vocabulary — no free-form error messages that could contain PII or stack traces. Used as the primary signal for AL-CAEP-01. |
| `sso.caep_stream_registered` | HIGH | 7 yr | `tenant_id`, `idp_type` (one of: `entra`, `okta`, `google_risc`), `caep_delivery_mode`, `admin_user_id_hash` | Emitted when a new CAEP stream is activated for a tenant. Records which admin user initiated the registration. |

**HMAC chain ordering for `sso.caep_user_purged`:** The `sso.caep_user_purged` event must be emitted and its HMAC chain entry committed to `audit_log` before the `account-purged` action handler writes to `SESSION_REVOCATION_KV` or updates `tenant_users`. This preserves the DEC-030 invariant that audit events precede state mutations. Since `account-purged` is a terminal and irreversible action, the ordering constraint is especially important — there must be an immutable record that the action was triggered by a specific IdP signal before the user row is deleted.

### 23.9 Alerting Rules

Five new alerting rules are added. Add all five to `docs/OBSERVABILITY.md §6.2` under a new `caep_stream_health` subsection.

| Rule ID | Severity | Condition | Response SLA | Runbook |
|---|---|---|---|---|
| AL-CAEP-01 | P1 | `sso.caep_stream_error` rate > 5% of all CAEP events for any single tenant over any 10-minute window | Acknowledge < 30 min | Check `error_type` distribution in `audit_log` for the affected tenant. If `jwks_kid_not_found`: verify IdP key rotation — FORM should have auto-refreshed JWKS, so a persistent miss suggests the IdP published a non-standard JWKS format or the `kid` field is missing. If `hmac_invalid`: possible webhook secret mismatch — tenant may need to re-register stream. If `tenant_mismatch`: potential security event — escalate to security-engineer immediately; do not self-remediate. Set `tenant_sso_configs.caep_status = 'error'` to pause stream until root cause is confirmed. |
| AL-CAEP-02 | P0 | Any `sso.caep_user_purged` event received | Immediate (PagerDuty wake) | GDPR deletion SLA has started (30-day clock). Verify `gdpr_deletion_queue_id` is present in the event metadata. Confirm the GDPR_DELETION_QUEUE consumer Worker is healthy (check Worker error rate in Cloudflare dashboard). If the deletion queue is backlogged or the consumer is erroring, escalate to devops-lead and compliance-officer immediately — GDPR deletion failure within 30 days is a regulatory violation. |
| AL-CAEP-03 | P2 | No CAEP events received on an active stream (`caep_status = 'active'`) for more than 4 hours during business hours (08:00–20:00 tenant local time) | Acknowledge < 2 h | Check IdP admin console to verify the CAEP stream is still configured (some IdP admin actions can silently delete streams). Check FORM's `caep_last_event_at` in `tenant_sso_configs`. If the IdP has no users online during the 4-hour window, this alert may be a false positive — suppress for tenants with fewer than 10 active users. Otherwise, re-register the stream via the admin dashboard CAEP configuration panel. |
| AL-CAEP-04 | P1 | Any Google RISC `hijacking` event received | Acknowledge < 30 min | A Google RISC `hijacking` event indicates that Google's risk engine has detected active session hijacking for a user in a Google Workspace OIDC tenant. FORM has already revoked the user's sessions (§23.4 row 8). Verify that `revoke:user:{tenant_id}:{user_id}` is present in SESSION_REVOCATION_KV. Notify the tenant admin via the admin dashboard notification system (event type: `security_alert`) that a user's account was flagged by Google. Document in the security incident log. |
| AL-CAEP-05 | P2 | CAEP rate limit exceeded for a tenant (> 1,000 events/hour, `caep_status = 'rate_limited'`) | Acknowledge < 2 h | Examine `caep_events` for the affected tenant: are the events all of one type? A flood of `session-revoked` events from a single IdP may indicate an IdP loop (e.g., Okta re-sending unacknowledged events). Check FORM's 202 response delivery — if FORM has been returning non-2xx (e.g., due to a downstream Supabase timeout), the IdP retry logic may be the cause. Reset the rate limit counter in KV (`caep_rate:{tenant_id}:{hour_bucket}` delete) after the root cause is identified. |

### 23.10 SOC 2 Mapping

| Criterion | Control Statement | How §23 Satisfies It |
|---|---|---|
| CC6.3 | Logical access is removed in a timely manner when no longer authorised | §23 reduces the IdP-event to FORM-session-revocation latency from ≤15 minutes (JWT TTL residual window, §12) to < 30 seconds from IdP event dispatch. The `sso.caep_event_received` event with `action_taken = 'user_sessions_revoked_*'` and its timestamp, compared against the SET `iat` claim, is the evidence of this SLA. |
| CC6.6 | Changes to access credentials trigger access re-evaluation | CAEP `credential-change` events (password reset, MFA change) trigger immediate user-level session revocation (§23.4 row 2). CAEP `token-claims-change` events trigger forced re-authentication (§23.4 row 3). Both are evidenced by `sso.caep_session_revoked` and `sso.caep_event_received` audit events with the appropriate `caep_event_type` metadata. |
| CC7.2 | The entity monitors system components for anomalies | Five alerting rules (AL-CAEP-01 through AL-CAEP-05) monitor CAEP stream health: SET validation failure rate (AL-CAEP-01), stream dead-man's switch (AL-CAEP-03), rate limit anomalies (AL-CAEP-05). PagerDuty routing ensures 24/7 on-call coverage. |
| CC7.3 | Identified anomalies are responded to and resolved | P0 alert on `account-purged` (AL-CAEP-02) initiates GDPR deletion workflow; P1 alert on Google RISC `hijacking` (AL-CAEP-04) triggers security incident investigation; P1 alert on SET validation failure rate (AL-CAEP-01) triggers stream health investigation with runbook. All incidents are logged in PagerDuty with resolution timestamps. |

**Evidence artefacts for SOC 2 auditor:**

| Artefact ID | Description | Collection Method |
|---|---|---|
| CC6-E-CAEP-001 | `caep_events` table export for the observation period: event_type distribution and action_taken distribution. Demonstrates that IdP security events are received and actioned. | `SELECT event_type, action_taken, count(*), min(processed_at), max(processed_at) FROM caep_events WHERE tenant_id = $tenant AND processed_at BETWEEN $obs_start AND $obs_end GROUP BY event_type, action_taken ORDER BY count DESC` |
| CC6-E-CAEP-002 | `audit_log` export — all `sso.caep_*` events for the observation period. Provides the HMAC-chained evidence trail. | `SELECT event_type, created_at, tenant_id, metadata FROM audit_log WHERE event_type LIKE 'sso.caep_%' AND created_at BETWEEN $obs_start AND $obs_end ORDER BY created_at` |
| CC6-E-CAEP-003 | CAEP stream registration records for all active enterprise tenants as of the observation period end date (sanitized: `caep_stream_id` and `caep_webhook_secret` redacted). Demonstrates that all enterprise tenants have CAEP configured. | `SELECT tenant_id, idp_type, caep_status, caep_delivery_mode, caep_last_event_at FROM tenant_sso_configs WHERE caep_status IN ('active', 'error') ORDER BY tenant_id` |
| CC6-E-CAEP-004 | AL-CAEP-01 and AL-CAEP-03 PagerDuty incident history for the observation period. Demonstrates stream health monitoring is operational and incidents are resolved within SLA. | PagerDuty Incidents API export filtered by service `form-caep-stream` and date range; include `created_at`, `resolved_at`, `resolution_note` fields |

### 23.11 Open Questions

| ID | Question | Owner | Priority | Resolution Target |
|---|---|---|---|---|
| OQ-SSO-23.1 | CAEP stream re-registration after SAML certificate rotation (§20) — should §20's cert rotation workflow automatically trigger CAEP stream re-registration at the IdP? Some IdPs (Okta in particular) tie the CAEP stream identity to the SSO application configuration; a SAML cert rotation may invalidate the stream token. Currently §20 has no hook into CAEP registration. The fix is straightforward (add a `caep_reregister_after_cert_rotation` step to the §20.4 rotation state machine) but requires coordination with the §20 implementation. | security-engineer | P2 | M5 |
| OQ-SSO-23.2 | CAEP and magic-link fallback sessions (§10) — if a user's IdP account is disabled via a CAEP `account-disabled` event, FORM revokes all sessions using `revoke:user:{tenant_id}:{user_id}`. However, magic-link sessions (§10) may be stored under a different session type in `enterprise_sessions`. Confirm: does `revoke:user:{tenant_id}:{user_id}` (§22.5) cover magic-link sessions in addition to SSO sessions? Current assumption: yes, because `isRevoked()` checks the user-level KV key regardless of session type. platform-engineer to confirm that magic-link session creation and validation paths both pass through `isRevoked()`. **🟢 Resolved — DEC-062 (2026-06-16). See §32.2.** `isRevoked()` confirmed at middleware layer; call-graph trace complete; integration test CC6-E-ML-001 added as M4 deploy closure gate. | platform-engineer | 🟢 **Resolved** | §32.2 · DEC-062 (2026-06-16) |
| OQ-SSO-23.3 | Polling fallback for IdPs that do not support push delivery — the SSF specification supports a pull/poll model where FORM periodically calls the IdP's transmitter endpoint to retrieve pending SETs, rather than the IdP pushing to FORM's webhook. Poll intervals of 5 minutes would still represent a significant improvement over the 15-minute JWT TTL but would not meet the < 30-second target. Is a < 30-second SLA contractually required, or is "significantly better than JWT TTL" sufficient? If a polling fallback is needed, what is the acceptable poll interval and the associated IdP API rate limit? | platform-engineer | P2 | M5 |
| OQ-SSO-23.4 | CAEP and the §21 Google Directory API group cache — if a Google RISC `hijacking` event is received for a user in a Google Workspace OIDC tenant, FORM revokes the user's sessions. However, the `google_directory_group_cache` (§21) for that user may still contain stale group membership data with a 5-minute TTL. Should a RISC `hijacking` event also proactively delete the `google_directory_group_cache` row for the affected user? The current 5-minute TTL means stale cache data cannot be used to authenticate (since the session is revoked), but the cache row persisting for up to 5 minutes after a hijacking event could be considered a minor hygiene concern. | platform-engineer | P2 | M5 |

### 23.12 Implementation Checklist

| # | Action | Owner | Priority | Milestone |
|---|---|---|---|---|
| 1 | Write and apply migration `migrations/YYYYMMDD_tenant_sso_configs_caep.sql` per §23.5.1: add six new columns to `tenant_sso_configs`; create partial index on `caep_status = 'active'`; verify on staging | platform-engineer | **P0** | M4 |
| 2 | Write and apply migration `migrations/YYYYMMDD_caep_events.sql` per §23.5.2: create `caep_events` table with RLS policies; create both indexes (`tenant_id, processed_at DESC` and `set_jti` UNIQUE); verify `ON CONFLICT (set_jti) DO NOTHING` semantics on staging | platform-engineer | **P0** | M4 |
| 3 | Implement `apps/api-gateway/src/sso/caep-action-handler.ts` per §23.4 pseudocode: all 8 event type handlers; `force_reauth` KV write; `account-purged` GDPR queue enqueue; unit test each handler branch with KV mock injection | platform-engineer | **P0** | M4 |
| 4 | Implement `apps/api-gateway/src/sso/caep-receiver.ts`: the 10-step pipeline per §23.3.2; HMAC-SHA256 validation; JWKS fetch + cache via `SSO_KV`; SET JWT verification; `dispatchCaepAction()` call; `caep_events` insert; `sso.caep_event_received` DEC-030 emit; 202 response | platform-engineer | **P0** | M4 |
| 5 | Extend `isRevoked()` (§22.4) to check two new KV key types: `force_reauth:{tenant_id}:{user_id}` and `account_suspended:{tenant_id}:{user_id}`; update `RevocationStatus` enum to include `FORCE_REAUTH` and `ACCOUNT_SUSPENDED`; update the authenticated request middleware to handle both new statuses (FORCE_REAUTH → 401 with re-auth prompt; ACCOUNT_SUSPENDED → 403 with suspension message) | platform-engineer | **P0** | M4 |
| 6 | Register CAEP streams for Okta tenants: implement `registerOktaCaepStream()` in the admin dashboard backend using the Okta SSF transmitter configuration endpoint per §23.6.2; include stream verification handshake; store `caep_stream_id` and `caep_delivery_mode` in `tenant_sso_configs` | platform-engineer | **P0** | M4 |
| 7 | Register CAEP streams for Entra tenants: implement `registerEntraCaepStream()` using Microsoft Graph API per §23.6.1; handle `EventGridEvent` envelope unwrapping in `caep-receiver.ts` for Entra-delivered events; validate outer Event Grid bearer token using Entra JWKS | platform-engineer | **P0** | M4 |
| 8 | Register RISC streams for Google Workspace OIDC tenants: implement `registerGoogleRiscStream()` per §23.6.3; handle Google RISC JWKS non-standard format; scope registration only to tenants with `sso_type = 'oidc'` and `oidc_issuer` matching `accounts.google.com`; implement AL-CAEP-04 PagerDuty trigger on `hijacking` event | platform-engineer | **P0** | M4 |
| 9 | Register all 6 new DEC-030 event types in `docs/AUDIT_LOG_SCHEMA.md` event registry; validate HMAC chain in staging with a test sequence: stream registration → CAEP event received → session revoked → user purged; confirm `sso.caep_user_purged` is emitted before `tenant_users.status` update (ordering constraint §23.8) | platform-engineer + security-engineer | **P0** | M4 |
| 10 | Configure AL-CAEP-01 through AL-CAEP-05 in PagerDuty; add all five rules to `docs/OBSERVABILITY.md §6.2` under new `caep_stream_health` subsection; test AL-CAEP-02 by simulating a synthetic `account-purged` event against a staging tenant | devops-lead | **P0** | M4 |
| 11 | Add CAEP configuration panel to admin dashboard (`apps/admin-dashboard/src/pages/sso/CAEPConfiguration.tsx`): enable/disable toggle; stream status badge (`inactive / active / error / rate_limited`); last event timestamp; manual stream re-registration button; `caep_events` audit log viewer tab (paginated, filtered by event_type) | frontend-engineer | **P1** | M4 |
| 12 | Add `client_capabilities=CP1` parameter to Entra OIDC and OAuth2 token requests in FORM's mobile and admin dashboard clients; verify `xms_cc` claim appears in Entra-issued access tokens on staging; document in `docs/SSO_CLIENT_CONFIG.md` | platform-engineer | **P1** | M4 |
| 13 | Implement AL-CAEP-03 dead-man's switch: Cloudflare Workers Cron (every 30 minutes); query `tenant_sso_configs WHERE caep_status = 'active' AND caep_last_event_at < NOW() - INTERVAL '4 hours'`; filter to business-hours windows; trigger PagerDuty P2 for each affected tenant | devops-lead | **P1** | M5 |
| 14 | Resolve OQ-SSO-23.2 (magic-link session coverage) before M4 deploy: trace `isRevoked()` call graph to confirm magic-link session validation passes through the user-level KV check; write a specific integration test: create magic-link session, write `revoke:user:*` KV key, confirm magic-link session rejected on next request | platform-engineer | **P1** | M4 |
| 15 | Collect CC6-E-CAEP-001 through CC6-E-CAEP-004 evidence artefacts 30 days after M4 go-live; store in `compliance/evidence/caep/`; advance G-010 (if applicable) or create a new gap entry for CAEP in the §9 gap tracker | compliance-officer | **P1** | M5 |

---

*v1.5 additions (2026-06-01): Section 23 — Continuous Access Evaluation (CAE) & Shared Signals Framework (SSF) — Real-Time IdP Risk Event Processing. Closes the complementary gap to §22: whereas §22 handles FORM-initiated session revocation, §23 handles the reverse signal direction — IdP-initiated risk events that FORM must receive and act on in real time. Gap closed: IdP security events (account disabled, password reset, MFA credential removed, session revoked from IdP admin console, high-risk sign-in, device compliance change) previously propagated to FORM only at JWT expiry (up to 15 minutes, §12 TTL). Target after §23: < 30 seconds from IdP event to FORM session invalidation via the §22 KV layer. Standards: IETF SSF (draft-ietf-sharedsignals-framework) as transport; OpenID CAEP as event vocabulary (Okta, Entra); OpenID RISC as Google-specific vocabulary; RFC 8417 Security Event Tokens (SET) as wire format. Architecture: FORM registers as SSF Receiver at each IdP; push delivery model (IdP POSTs SETs to `POST /enterprise/v1/sso/caep-receiver/{tenant_id}` Cloudflare Worker); 10-step processing pipeline: HMAC webhook secret validation → JWKS fetch/cache in SSO_KV (1h TTL) → SET JWT signature verification → replay check via set_jti UNIQUE constraint → user resolution → action dispatch → caep_events INSERT → DEC-030 emit → 202 Accepted. Eight event-type-to-action mappings: session-revoked (revoke specific session via §22 KV), credential-change (revoke all user sessions), token-claims-change (set force_reauth KV flag, TTL 24h, triggers 401 re-auth prompt not hard revocation), account-disabled (revoke sessions + set account_suspended KV flag + update tenant_users.status), account-enabled (clear account_suspended KV + restore active status), account-purged (revoke sessions + mark deleted_by_idp + enqueue GDPR deletion workflow), device-compliance-change (conditional revocation based on require_device_compliance tenant flag), Google RISC hijacking (immediate revocation + PagerDuty P0). Two new KV key patterns added to §22.5 schema: force_reauth:{tenant_id}:{user_id} (TTL 24h) and account_suspended:{tenant_id}:{user_id} (TTL 72h); isRevoked() extended with FORCE_REAUTH and ACCOUNT_SUSPENDED RevocationStatus values and corresponding middleware handling. Schema: six new columns on tenant_sso_configs (caep_stream_id, caep_delivery_mode, caep_webhook_secret BYTEA AES-256-GCM encrypted, caep_status ENUM, caep_last_event_at, caep_error_count); new caep_events table (id, tenant_id, user_id nullable, set_jti UNIQUE, event_type, idp_subject, action_taken, processed_at, raw_set_hash — all PII fields stored as SHA-256 hashes; RLS: form_system ALL, tenant_admin SELECT own tenant, form_api zero-rows). IdP-specific implementation: Entra CAE via Microsoft Graph API + EventGridEvent envelope unwrapping + two-layer JWT validation (outer Event Grid bearer token + inner SET signature) + CP1 client capability flag (xms_cc claim); Okta CAEP via SSF transmitter configuration endpoint + stream verification handshake + shared JWKS cache with §4.1 OIDC; Google RISC via Identity Toolkit API + non-standard JWKS format handling + Google Workspace OIDC tenants only. Security: SET replay prevention via set_jti UNIQUE constraint (202 on replay, not 409); JWKS cache poisoning prevention (fresh fetch on kid miss, HIGH audit event if still unresolved); HMAC-SHA256 webhook authenticity (constant-time comparison); rate limiting 1,000 events/h per tenant (429 + stream pause + AL-CAEP-05); tenant isolation (tenant_id URL path vs. SET iss/sub cross-validation, CRITICAL audit on mismatch); PII minimisation (set_jti, idp_subject, raw_set stored as SHA-256 hashes only). Six DEC-030 HMAC-chained audit events: sso.caep_event_received (STANDARD, 7yr — primary SOC 2 CC6.3 evidence), sso.caep_session_revoked (HIGH, 7yr), sso.caep_user_suspended (HIGH, 7yr), sso.caep_user_purged (CRITICAL, 7yr — GDPR deletion SLA start-of-clock; two-person rule on override; emitted before state mutation), sso.caep_stream_error (HIGH, 7yr — controlled error_type vocabulary), sso.caep_stream_registered (HIGH, 7yr). Five alerting rules: AL-CAEP-01 P1 (SET validation failure rate > 5% over 10 min — possible JWKS poisoning), AL-CAEP-02 P0 (account-purged received — GDPR clock starts), AL-CAEP-03 P2 (dead-man's switch: no events on active stream for 4h during business hours), AL-CAEP-04 P1 (Google RISC hijacking event), AL-CAEP-05 P2 (rate limit exceeded). SOC 2: CC6.3 (< 30 s IdP-event to FORM-revocation vs. ≤15 min previously), CC6.6 (credential-change and token-claims-change mapped to session invalidation), CC7.2 (5 alerting rules on stream health), CC7.3 (P0/P1 PagerDuty routing; GDPR workflow on account-purged). Four evidence artefacts: CC6-E-CAEP-001 (caep_events export), CC6-E-CAEP-002 (audit_log sso.caep_* export), CC6-E-CAEP-003 (CAEP stream registration records, sanitized), CC6-E-CAEP-004 (AL-CAEP-01 and AL-CAEP-03 PagerDuty history). Four open questions: OQ-SSO-23.1 (CAEP stream re-registration after §20 SAML cert rotation — security-engineer, P2, M5), OQ-SSO-23.2 (magic-link session coverage — platform-engineer, P1, before M4 deploy), OQ-SSO-23.3 (polling fallback interval vs. < 30 s SLA — platform-engineer, P2, M5), OQ-SSO-23.4 (§21 Google Directory group cache invalidation on RISC hijacking — platform-engineer, P2, M5). Implementation checklist: 15 tasks (8× P0 M4, 5× P1 M4/M5, 2× P1 M5).*

---

## 24. Privileged Access Management (PAM): Just-in-Time Privilege Escalation & Break-Glass Protocol for `form_admin` Operations

### 24.1 Problem: Standing Privilege Risk

The `form_admin` Postgres role grants cross-tenant `SELECT` and `UPDATE` access across all tenant schemas via Row-Level Security bypass. This role is necessary for internal customer-success operations (shadow login per §4, bulk remediation, compliance evidence extraction) but its current model grants **standing access** — credentials exist persistently in internal tooling, Cloudflare Access service tokens, and engineer workstations.

Standing privilege violates the principle of least privilege and creates a direct exposure surface for SOC 2 Trust Service Criteria:

| SOC 2 Criterion | Risk from Standing `form_admin` Credentials |
|---|---|
| CC6.1 — Logical access controls | `form_admin` is not scoped to a purpose, duration, or requestor |
| CC6.2 — Access provisioning | No approval workflow exists prior to access being exercised |
| CC6.3 — Timely deprovisioning | Access does not auto-expire; revocation requires manual credential rotation |

PAM eliminates standing access by replacing it with **Just-in-Time (JIT) privilege escalation**: `form_admin` credentials are never stored persistently in engineer environments. Every session is request-scoped, time-limited, approval-gated, and fully recorded in the DEC-030 HMAC-chained audit log.

**Non-goals:** PAM does not replace Cloudflare Access authentication for the admin dashboard (§16). PAM specifically governs direct Postgres `form_admin` role usage — the elevated privilege path that bypasses tenant RLS.

---

### 24.2 JIT Elevation Architecture

The privileged access flow is mediated entirely by the `pam-elevation-service` Cloudflare Worker. No engineer or internal service holds a static `form_admin` Postgres password.

```
Engineer / CS Tool
        |
        | POST /internal/v1/pam/elevate
        |   { access_level, justification, target_tenant_id? }
        v
+---------------------------+
|  pam-elevation-service    |  Cloudflare Worker
|  (workers/pam-elevation)  |
+---------------------------+
        |
        |-- 1. Cloudflare Access JWT validation (mTLS service token or IdP identity)
        |-- 2. Resolve admin_user_id from JWT sub claim
        |-- 3. Determine access_level tier (read_only | read_write | destructive)
        |-- 4. Approval gate (see §24.3)
        |-- 5. TOTP/WebAuthn second factor verification
        |-- 6. Generate pam_session_id (UUID v4, cryptographically random)
        |-- 7. Write PAM KV record (see §24.5) with TTL
        |-- 8. Emit pam.elevation_requested + pam.elevation_approved DEC-030 events
        |-- 9. Return { pam_session_id, expires_at, access_level } — never returns Postgres credentials
        v
+---------------------------+
|  pam-db-proxy             |  Supabase Edge Function (privileged context only)
|  (supabase/functions/     |
|   pam-db-proxy)           |
+---------------------------+
        |
        |-- 1. Validate pam_session_id against PAM KV (must exist, not expired, not revoked)
        |-- 2. Verify caller identity matches KV record admin_user_id
        |-- 3. Open Postgres connection as form_system (connection pool)
        |-- 4. SET ROLE form_admin;
        |-- 5. SET app.pam_session_id = '<pam_session_id>';
        |-- 6. Execute requested query (tenant_id injected for all cross-tenant ops)
        |-- 7. RESET ROLE; (after each statement batch — session-level, not transaction-level)
        |-- 8. Return results
        v
    Postgres (Supabase)
    form_admin role active only within this connection's SET ROLE scope
```

**Key design invariants:**

1. `pam-elevation-service` never touches Postgres directly. It only manages the KV session record.
2. `pam-db-proxy` never issues elevation — it only validates an existing KV session and opens a scoped connection.
3. `SET app.pam_session_id` is a Postgres session-level GUC. Audit triggers on sensitive tables read `current_setting('app.pam_session_id')` and include it in their `audit_log` row. This creates a database-level linkage between every privileged query and its PAM session — independent of the application-layer audit events.
4. `RESET ROLE` is called after each statement batch, not at connection close. The connection pool (PgBouncer in session mode) resets role on checkout via `server_reset_query = RESET ROLE; RESET app.pam_session_id;`.
5. The `pam-db-proxy` function is only reachable via a Cloudflare Access service token scoped to `internal-tools` audience — not exposed on the public API surface.

---

### 24.3 Approval Workflow by Access Level

Three tiers of access are defined. The tier is determined by the operation category the requester declares at elevation time; `pam-db-proxy` enforces at execution time that the declared tier matches the actual SQL operation class (DML vs. DDL vs. cross-tenant mutation).

| Access Level | Permitted Operations | Approval Requirement | Second Factor | TTL |
|---|---|---|---|---|
| `read_only` | `SELECT` only; single-tenant scope or cross-tenant aggregate (anonymized) | Self-approve (no external approval gate) | TOTP (RFC 6238) | 4 hours |
| `read_write` | `SELECT` + `INSERT` + `UPDATE` on support tables; single-tenant scope only | Async manager approval via Slack webhook (see §24.3.1); 15-minute approval window; auto-deny on timeout | TOTP (RFC 6238) | 1 hour |
| `destructive` | `DELETE`, `TRUNCATE`, cross-tenant `UPDATE`, schema migrations in flight | Dual-person rule: separate approver identity required (not the requester); approver must authenticate separately via FIDO2 WebAuthn hardware key | Requester: FIDO2 WebAuthn hardware key; Approver: FIDO2 WebAuthn hardware key | 15 minutes |

**24.3.1 Async Manager Approval for `read_write`**

When `read_write` elevation is requested, `pam-elevation-service`:

1. Creates a PAM KV record with `status: pending_approval` and TTL of 15 minutes (approval window).
2. POSTs a signed Slack webhook message to the `#form-pam-approvals` channel (webhook URL stored in Cloudflare Workers Secret `PAM_SLACK_WEBHOOK_URL`). Message includes: requester identity, declared justification, target tenant (if single-tenant), access level, request timestamp, approve/deny deep links.
3. Approve/deny links invoke `POST /internal/v1/pam/approve/{pam_request_id}` or `.../deny/{pam_request_id}`, authenticated by the approver's Cloudflare Access JWT.
4. On approval: KV record transitions to `status: approved`; requester is notified via Slack DM; requester must still present TOTP within 5 minutes to complete elevation.
5. On timeout (15 minutes without approval action): KV record transitions to `status: denied_timeout`; `pam.elevation_denied` DEC-030 event emitted with `reason: approval_timeout`.
6. The approver cannot be the same `admin_user_id` as the requester. `pam-elevation-service` enforces this at the approval step and emits a `pam.elevation_denied` event with `reason: self_approval_attempted` if violated.

**24.3.2 Dual-Person Rule for `destructive`**

For `destructive` tier:

1. Requester initiates elevation: presents FIDO2 WebAuthn assertion; PAM KV record created with `status: awaiting_second_person`.
2. A separate approver (different `admin_user_id`, different active Cloudflare Access session) navigates to `POST /internal/v1/pam/cosign/{pam_request_id}` and presents their own FIDO2 WebAuthn assertion.
3. Both WebAuthn assertions are recorded in the KV record (`requester_webauthn_credential_id`, `approver_webauthn_credential_id`) and in the `pam.elevation_approved` DEC-030 audit event.
4. KV record transitions to `status: approved`; TTL is set to 15 minutes from cosign completion.
5. If no cosign occurs within 30 minutes, the KV record expires and `pam.elevation_denied` is emitted with `reason: cosign_timeout`.

---

### 24.4 Break-Glass Emergency Access Protocol

Break-glass is used when the `pam-elevation-service` Worker is unavailable (Worker deployment failure, KV store outage, IdP connectivity loss) and a production incident requires immediate `form_admin` access.

**Break-glass is not a privilege bypass — it is an independently audited high-urgency path with stricter post-hoc review requirements.**

**24.4.1 Break-Glass Access Policy**

A separate Cloudflare Access application (`form-break-glass`) is configured with:

- Audience tag distinct from the standard PAM service (`CF_ACCESS_AUDIENCE_BREAK_GLASS`).
- Policy: named list of at most 5 break-glass-authorized identities (managed in `cloudflare/access/break-glass-policy.tf`). This list is reviewed and re-approved quarterly.
- Session duration: 2 hours maximum, non-renewable. Cloudflare Access will not issue a new session until the previous one has fully expired.
- Certificate-bound session (`mtls_auth` policy requirement) — hardware key must be present.

**24.4.2 Break-Glass Activation Sequence**

```
1. Engineer navigates to https://break-glass.internal.form.coach
2. Cloudflare Access enforces mTLS + named-identity policy
3. On successful Access auth, a Cloudflare Worker (workers/break-glass-notifier) fires immediately:
   a. Writes pam:session:{session_id} KV record with is_break_glass: true, TTL 2h
   b. Emits pam.break_glass_activated DEC-030 audit event (CRITICAL severity)
   c. Triggers PagerDuty P0 incident via Events API v2 (routing key: PAM_PAGERDUTY_ROUTING_KEY)
      - Title: "FORM PAM Break-Glass Activated — <admin_user_id>"
      - Severity: critical
      - Payload includes: admin_user_id, timestamp, cf_access_session_id (hashed), pam_session_id
   d. Sends immediate email to compliance-officer@form.coach (via Cloudflare Email Workers)
      - Subject: "[COMPLIANCE ALERT] PAM Break-Glass Activated"
      - Body: same payload as PagerDuty; includes link to audit log filtered to pam_session_id
4. Engineer has direct Postgres access via break-glass Supabase connection string
   (stored in Cloudflare Workers Secret BREAK_GLASS_DB_URL — rotated quarterly)
   Connection string uses a dedicated Postgres role `form_break_glass` with identical
   privileges to `form_admin` but a distinct role name for audit log differentiation.
5. All Postgres activity is captured by pg_audit (statement level) for the duration of
   the break-glass session.
6. At 2h: Cloudflare Access session expires; KV record TTL expires; pam.break_glass_expired
   DEC-030 event emitted automatically by a Cloudflare Workers Cron job
   (workers/pam-expiry-sweeper) that polls for expired break-glass KV keys.
```

**24.4.3 Post-Incident Review Requirement**

Any break-glass activation triggers a mandatory post-incident review within 72 hours:

- compliance-officer and security-engineer jointly review pg_audit logs for the session.
- A GitHub Issue is auto-created in `form-internal/security-reviews` by the `break-glass-notifier` Worker, linked to the `pam_session_id`.
- The review must produce one of: (a) "no anomaly — incident justified use", (b) "anomaly detected — escalate to security incident", or (c) "process gap — PAM reliability improvement required".
- The review outcome is recorded in the `pam_break_glass_reviews` Postgres table (owned by `form_compliance` role) and linked to the original DEC-030 `pam.break_glass_activated` event via `pam_session_id`.

---

### 24.5 KV Schema

All PAM session state lives in Cloudflare KV namespace `PAM_KV` (separate namespace from `SESSION_REVOCATION_KV` and `SSO_KV`).

**Key pattern:** `pam:session:{pam_session_id}`

**Value schema (JSON):**

```jsonc
{
  "pam_session_id": "uuid-v4",          // matches KV key suffix
  "admin_user_id": "uuid-v4",           // internal FORM user ID of the requester
  "access_level": "read_only | read_write | destructive",
  "status": "pending_approval | awaiting_second_person | approved | denied | revoked | expired",
  "expires_at": "ISO-8601 UTC",         // absolute expiry; KV TTL set to same value
  "approved_at": "ISO-8601 UTC | null",
  "approver_id": "uuid-v4 | null",      // null for read_only (self-approve)
  "pam_request_id": "uuid-v4",          // stable ID for Slack approval flow deep links
  "is_break_glass": false,              // true only for §24.4 break-glass sessions
  "justification_hash": "sha256-hex",   // SHA-256 of the free-text justification — stored hashed, not raw (PII minimisation)
  "target_tenant_id": "uuid-v4 | null", // null for cross-tenant operations
  "requester_webauthn_credential_id": "base64url | null",  // destructive tier only
  "approver_webauthn_credential_id": "base64url | null",   // destructive tier only
  "cf_access_jti": "sha256-hex"         // SHA-256 of the Cloudflare Access JWT jti claim — links PAM session to CF Access session without storing the JWT
}
```

**TTL values by access level:**

| Access Level | KV TTL | Notes |
|---|---|---|
| `read_only` | 14,400 s (4h) | Self-approved; starts at elevation completion |
| `read_write` | 3,600 s (1h) | Starts at cosign/approval + TOTP confirmation |
| `destructive` | 900 s (15 min) | Starts at dual cosign completion |
| Break-glass | 7,200 s (2h) | Non-renewable; fixed at activation |
| `pending_approval` | 900 s (15 min) | Auto-deny on KV expiry if still pending |
| `awaiting_second_person` | 1,800 s (30 min) | Auto-deny on KV expiry if still awaiting |

**Secondary index key:** `pam:by_admin:{admin_user_id}:{pam_session_id}` — TTL matches primary. Used by `pam-elevation-service` to look up all active sessions for a given admin without a full KV scan. Value: `{ "access_level": "...", "expires_at": "..." }` (minimal, for listing purposes only).

---

### 24.6 DEC-030 Audit Events

All six PAM audit event types are HMAC-chained per the DEC-030 specification. Each event appends to the global audit log chain; no PAM-specific chain is maintained separately. The `pam_session_id` field provides the application-layer linkage across related events.

| Event Type | Severity | Retention | Two-Person Rule | Description |
|---|---|---|---|---|
| `pam.elevation_requested` | HIGH | 7 years | No | Emitted when a PAM elevation request is received by `pam-elevation-service`, before approval. Fields: `admin_user_id`, `access_level`, `pam_request_id`, `target_tenant_id` (nullable), `justification_hash`. |
| `pam.elevation_approved` | HIGH | 7 years | No | Emitted when approval gate passes and second factor is confirmed. Fields: `admin_user_id`, `approver_id` (nullable for `read_only`), `access_level`, `pam_session_id`, `expires_at`, `requester_webauthn_credential_id` (nullable), `approver_webauthn_credential_id` (nullable). |
| `pam.elevation_denied` | HIGH | 7 years | No | Emitted on any denial path: self-approval attempt, approval timeout, cosign timeout, TOTP failure, second-factor mismatch. Fields: `admin_user_id`, `access_level`, `pam_request_id`, `reason` (enum: `self_approval_attempted`, `approval_timeout`, `cosign_timeout`, `totp_failure`, `webauthn_failure`, `policy_violation`). |
| `pam.session_expired` | STANDARD | 7 years | No | Emitted by `pam-expiry-sweeper` Cron Worker when a PAM KV record reaches TTL. Fields: `admin_user_id`, `pam_session_id`, `access_level`, `expired_at`. Distinct from `pam.elevation_denied` — session was valid and completed its full TTL. |
| `pam.break_glass_activated` | CRITICAL | 7 years | Yes — post-hoc (§24.4.3) | Emitted immediately on break-glass Cloudflare Access session creation. Fields: `admin_user_id`, `pam_session_id`, `cf_access_jti_hash`, `activated_at`. Triggers PagerDuty P0 + compliance email. The two-person rule here is post-hoc review (§24.4.3) rather than pre-authorization — the emergency nature makes pre-authorization infeasible. |
| `pam.break_glass_expired` | HIGH | 7 years | No | Emitted by `pam-expiry-sweeper` when break-glass KV record TTL expires. Fields: `admin_user_id`, `pam_session_id`, `expired_at`, `review_issue_url` (GitHub issue auto-created at activation). |

**Audit event field invariants (DEC-030 compliance):**

- No free-text justification is stored in the audit log. Only `justification_hash` (SHA-256) appears.
- No Postgres query text is stored in audit events. Query-level logging is handled by pg_audit at the database layer, not the application layer.
- `target_tenant_id` is stored as a UUID. No tenant name, domain, or PII is stored in the event.
- `admin_user_id` is the FORM internal UUID, not an email address.

---

### 24.7 Alerting Rules

| Alert ID | Severity | Condition | Response | Channel |
|---|---|---|---|---|
| AL-PAM-01 | **P0** | `pam.break_glass_activated` event emitted | Immediate PagerDuty incident + compliance-officer email. On-call engineer confirms incident context and whether break-glass was authorized. If no active production incident is linked within 15 minutes of activation, auto-escalate to security-incident. | PagerDuty + Email Workers |
| AL-PAM-02 | **P1** | `pam.elevation_denied` count > 3 from the same `admin_user_id` within any rolling 1-hour window | PagerDuty P1. Investigate: credential stuffing against PAM endpoint, TOTP desync, or insider threat pattern. Suspend PAM elevation for the affected `admin_user_id` pending review (write `pam:suspended:{admin_user_id}` KV key, TTL 24h). | PagerDuty |
| AL-PAM-03 | **P2** | A PAM session is presented to `pam-db-proxy` with `expires_at` in the past but the KV record has not yet been evicted (KV/clock skew window) | PagerDuty P2. The `pam-db-proxy` must reject the session regardless (hard expiry check on `expires_at` field, not solely on KV key existence). Alert fires to track clock skew frequency. If AL-PAM-03 fires more than twice in 24 hours, escalate to P1 and investigate KV TTL reliability. | PagerDuty |

All three alert rules are registered in `docs/OBSERVABILITY.md §6` under the `pam_session_health` subsection.

---

### 24.8 SOC 2 Mapping

| SOC 2 Criterion | Control | PAM Evidence Artefact |
|---|---|---|
| CC6.1 — Logical access controls | `form_admin` Postgres role is never held as standing credential by any human or service account. Access is exclusively time-bound, request-scoped, and approval-gated via PAM. | `pam.elevation_approved` audit log export showing zero standing sessions; PAM KV TTL configuration screenshots. |
| CC6.2 — Access provisioning | Every `read_write` and `destructive` elevation requires explicit manager or dual-person approval before access is granted. Approval is logged with approver identity and timestamp. | `pam.elevation_approved` events with non-null `approver_id`; Slack approval message archive. |
| CC6.3 — Timely deprovisioning | PAM sessions auto-expire by KV TTL. `pam-expiry-sweeper` emits `pam.session_expired` events confirming deprovisioning. No manual revocation required for normal expiry. | `pam.session_expired` audit log export; demonstration that KV TTL matches declared TTL in §24.5. |
| CC6.7 — Transmission confidentiality | All PAM API endpoints (`/internal/v1/pam/*`) are served over mTLS. The `pam-db-proxy` Supabase Edge Function communicates with Postgres over TLS (Supabase enforces `sslmode=require`). PAM KV values are encrypted at rest by Cloudflare KV. No PAM tokens are transmitted over plain HTTP. | Cloudflare Access mTLS policy configuration; Supabase TLS connection logs; network trace confirming no plaintext PAM token transmission. |

**Evidence collection cadence:** CC6-E-PAM-001 through CC6-E-PAM-004 artefacts are collected 30 days after M4 go-live and stored in `compliance/evidence/pam/`. CC6-E-PAM-005 through CC6-E-PAM-008 (Postgres-layer evidence from `admin_jit_escalations` and `pam_break_glass_reviews`) are collected 30 days after M6 go-live (DATA_MODEL §29 migration `0058_pam_schema.sql` deployed). Artefact index:

| Artefact ID | Description | SOC 2 | Source |
|---|---|---|---|
| CC6-E-PAM-001 | Export of `pam.elevation_approved` and `pam.elevation_denied` DEC-030 events for a representative 30-day window. Demonstrates KV-layer approval workflow is functioning. | CC6.1, CC6.2 | Audit log; collected M5 |
| CC6-E-PAM-002 | Export of `pam.session_expired` DEC-030 events confirming automatic KV-layer deprovisioning for all PAM sessions in the window. | CC6.3 | Audit log; collected M5 |
| CC6-E-PAM-003 | PAM KV namespace TTL configuration screenshot + `pam-elevation-service` Worker source excerpt showing TTL constants. Demonstrates no standing access is possible beyond declared TTLs. | CC6.1, CC6.3 | Cloudflare KV dashboard; collected M5 |
| CC6-E-PAM-004 | Break-glass policy configuration from `cloudflare/access/break-glass-policy.tf` (sanitized) + quarterly review sign-off record for the break-glass authorized identity list. | CC6.1, CC6.6 | Terraform config + compliance sign-off; collected M5 |
| CC6-E-PAM-005 | Export of `admin_jit_escalations` rows for a representative 30-day window: zero rows with `target_tenant_id IS NULL AND access_level != 'read_only'`; all `read_only` sessions ≤ 4h TTL (`escalation_start + INTERVAL '4 hours' >= escalation_expiry`). Demonstrates no null-scoped access and hard TTL enforcement at the Postgres layer. | CC6.1 | `admin_jit_escalations` Postgres table (DATA_MODEL §29.3); collected M7 (30 days after M6 migration) |
| CC6-E-PAM-006 | Export of `admin_jit_escalations` rows where `access_level IN ('read_write', 'destructive')` for observation period; all rows show `authorised_by_ic_user_id IS NOT NULL`. Demonstrates two-person approval enforced at Postgres layer (CHECK constraint `chk_jit_approver_required` — DATA_MODEL §29.10 item 11). | CC6.2 | `admin_jit_escalations` Postgres table; collected M7 |
| CC6-E-PAM-007 | Export of `pam.session_expired` DEC-030 events + matching `admin_jit_escalations` rows confirming `status = 'expired'` and `dec030_session_expired_event_id` populated; zero rows with `escalation_expiry < now() AND status = 'approved'` (no orphaned approved-but-expired rows). Demonstrates KV TTL expiry and Postgres record alignment via `pam-expiry-sweeper`. | CC6.3 | Audit log + `admin_jit_escalations`; collected M7 |
| CC6-E-PAM-008 | Export of all `pam_break_glass_reviews` rows for observation period: all rows show `reviewed_at IS NOT NULL` and `outcome` set within 72h of `review_due_at`; matching `pam.break_glass_review_completed` DEC-030 CRITICAL events for each row. Demonstrates 72-hour mandatory post-incident review SLA enforced by AL-PAM-BG-01 (OBSERVABILITY §29). | CC6.6 | `pam_break_glass_reviews` Postgres table + audit log (DATA_MODEL §29.3); collected M7 |

---

### 24.9 Open Questions

| ID | Question | Owner | Priority | Target |
|---|---|---|---|---|
| OQ-SSO-24.1 | Should `pam-db-proxy` use a dedicated Supabase Edge Function or a Cloudflare Worker with a direct Postgres connection via Hyperdrive? Hyperdrive would keep the privileged connection pool within the Cloudflare network boundary; Edge Function keeps it in the Supabase trust boundary. Security-engineer to assess mTLS profile of each path. | security-engineer | P1 | Before M4 deploy | 🟢 **Resolved — DEC-060 (2026-06-15)**: Supabase Edge Function adopted. Hyperdrive rejected: SET ROLE incompatible with transaction mode, form_admin credential crosses provider boundary, 20–50 ms latency overhead. See §31.2. |
| OQ-SSO-24.2 | `SET ROLE form_admin` within a Supabase connection pool (PgBouncer transaction mode) is not reliable — `SET ROLE` is session-scoped and PgBouncer in transaction mode does not guarantee session affinity. Decision: use PgBouncer **session mode** for PAM connections (separate pool, not the shared `form_api` pool). Confirm Supabase plan supports a dedicated session-mode pool without impacting existing connection limits. | platform-engineer | P0 | Before M4 deploy | 🟢 **Resolved — DEC-060 (2026-06-15)**: Direct Postgres session connection (port 5432) adopted — PgBouncer bypassed entirely. No dedicated session-mode pool required. Max 5–6 concurrent PAM connections well within Supabase Pro plan 60-connection ceiling. See §31.3. |
| OQ-SSO-24.3 | The `form_break_glass` Postgres role (§24.4.2) has identical privilege to `form_admin`. Should they be merged (same role, different connection string) or kept distinct? Distinct roles produce clearer pg_audit attribution. Legal/compliance to advise whether distinct role names are required for audit trail segregation. | compliance-officer | P2 | M5 |
| OQ-SSO-24.4 | FIDO2 WebAuthn hardware key requirement for `destructive` tier assumes all break-glass-eligible engineers own a FIDO2 key (YubiKey or equivalent). Confirm procurement and enrollment before M4 cutover — if keys are not in hand, `destructive` tier must block entirely rather than fall back to TOTP. | security-engineer + eng-manager | P0 | Before M4 deploy |

---

### 24.10 Implementation Checklist

| # | Task | Owner | Priority | Milestone |
|---|---|---|---|---|
| 1 | Scaffold `workers/pam-elevation-service`: Cloudflare Worker with routes `POST /internal/v1/pam/elevate`, `POST /internal/v1/pam/approve/{id}`, `POST /internal/v1/pam/deny/{id}`, `POST /internal/v1/pam/cosign/{id}`. Implement Cloudflare Access JWT validation middleware (reuse pattern from §16 admin dashboard). Wire `PAM_KV` namespace binding. | platform-engineer | **P0** | M4 |
| 2 | Implement KV schema (§24.5): `pam:session:{id}` primary record + `pam:by_admin:{admin_user_id}:{id}` secondary index. Implement TTL constants per access level. Add `pam:suspended:{admin_user_id}` key write on AL-PAM-02 trigger. | platform-engineer | **P0** | M4 |
| 3 | Implement approval workflow (§24.3): self-approve path for `read_only`; async Slack webhook + approval deep-link for `read_write`; FIDO2 WebAuthn dual-cosign for `destructive`. Resolve OQ-SSO-24.4 (hardware key procurement) before `destructive` tier goes live. | platform-engineer + security-engineer | **P0** | M4 |
| 4 | Scaffold `supabase/functions/pam-db-proxy`: validate `pam_session_id` against PAM KV; enforce hard expiry check on `expires_at` (AL-PAM-03); open Postgres connection as `form_system` via `SUPABASE_PAM_DB_URL` (port 5432, direct session connection — PgBouncer bypassed per DEC-060); `SET ROLE form_admin`; `SET app.pam_session_id`; execute query; `RESET ROLE`; `RESET app.pam_session_id`; close connection. OQ-SSO-24.2 resolved — see §31. | platform-engineer | **P0** | M4 | 🟢 **DEC-060** — architecture confirmed; implementation pending |
| 5 | Add `app.pam_session_id` GUC audit triggers to sensitive Postgres tables (`tenant_users`, `tenant_sso_configs`, `enterprise_sessions`, `audit_log`): trigger reads `current_setting('app.pam_session_id', true)` and includes value in the `audit_log` row `metadata` JSONB field. This creates database-layer linkage between privileged queries and PAM sessions. | platform-engineer | **P0** | M4 |
| 6 | Scaffold `workers/break-glass-notifier`: fires on Cloudflare Access `break-glass` audience JWT receipt; writes PAM KV with `is_break_glass: true`; emits `pam.break_glass_activated` DEC-030 event; triggers PagerDuty P0 via Events API v2; sends compliance email via Cloudflare Email Workers. Configure `cloudflare/access/break-glass-policy.tf` with named identity list (max 5). | devops-lead | **P0** | M4 |
| 7 | Scaffold `workers/pam-expiry-sweeper`: Cloudflare Workers Cron (every 5 minutes); scan `pam:session:*` keys for `is_break_glass: true` with expired TTL; emit `pam.break_glass_expired` DEC-030 event; scan for `status: pending_approval` or `status: awaiting_second_person` keys with elapsed approval/cosign window; emit `pam.elevation_denied` with appropriate `reason`. | devops-lead | **P1** | M4 |
| 8 | Register all 6 PAM DEC-030 event types in `docs/AUDIT_LOG_SCHEMA.md` event registry. Validate HMAC chain in staging: elevation_requested → elevation_approved → session_expired sequence. Validate break-glass chain: break_glass_activated → break_glass_expired. Confirm no PII (email, justification text, query text) appears in any event payload. | platform-engineer + security-engineer | **P0** | M4 |
| 9 | Configure AL-PAM-01, AL-PAM-02, AL-PAM-03 in PagerDuty. Add all three rules to `docs/OBSERVABILITY.md §6` under `pam_session_health` subsection. Test AL-PAM-01 by triggering a synthetic break-glass activation against staging. Test AL-PAM-02 by submitting 4 denied elevation requests from the same `admin_user_id` within 1 hour in staging. | devops-lead | **P0** | M4 |
| 10 | Collect CC6-E-PAM-001 through CC6-E-PAM-004 artefacts 30 days after M4 go-live; store in `compliance/evidence/pam/`; create entries in `docs/SOC2_READINESS.md` for CC6.1, CC6.2, CC6.3, CC6.7 mapping to PAM controls. Collect CC6-E-PAM-005 through CC6-E-PAM-008 (Postgres-layer artefacts from DATA_MODEL §29) 30 days after M6 migration deployment. Resolve OQ-SSO-24.3 (role naming decision) before M5 freeze. OQ-SSO-24.1 resolved — DEC-060. CC6-E-PAM-003 updated to include port 5432 screenshot per §31.6. | compliance-officer + security-engineer | **P1** | M5 (KV artefacts); M7 (Postgres artefacts) | OQ-SSO-24.1 🟢 **DEC-060** |

---

*v1.8 (2026-06-10): §24.8 evidence artefact index extended — CC6-E-PAM-005 through CC6-E-PAM-008 added as Postgres-layer evidence from `docs/DATA_MODEL.md §29` (migration `0058_pam_schema.sql`). Four new artefacts: CC6-E-PAM-005 (`admin_jit_escalations` null-scope and TTL assertion — CC6.1), CC6-E-PAM-006 (`admin_jit_escalations` non-null approver for `read_write`/`destructive` rows — CC6.2), CC6-E-PAM-007 (KV `pam.session_expired` DEC-030 events + Postgres `status='expired'` alignment — CC6.3), CC6-E-PAM-008 (`pam_break_glass_reviews` 72h review SLA + `pam.break_glass_review_completed` DEC-030 event chain — CC6.6). Evidence collection cadence note updated: KV-layer artefacts (001–004) at M5; Postgres-layer artefacts (005–008) at M7 (30 days post M6 migration). Checklist item 10 updated to include M7 Postgres artefact collection. Evidence table columns expanded to include SOC 2 criterion and source. Cross-references: `docs/DATA_MODEL.md §29` (admin_jit_escalations + pam_break_glass_reviews DDL; PAM-E-001 through PAM-E-004 in DATA_MODEL §29.9 map to CC6-E-PAM-005 through CC6-E-PAM-008 here), `docs/OBSERVABILITY.md §29` (AL-PAM-BG-01 enforces the 72h break-glass review SLA that CC6-E-PAM-008 evidences), `docs/INCIDENT_RESPONSE.md §R-20` (OQ-INS-01 resolved — R-20 v1.5 cross-reference patch). Owner: compliance-officer + security-engineer.*

*v1.6 additions (2026-06-01): Section 24 — Privileged Access Management (PAM): Just-in-Time Privilege Escalation & Break-Glass Protocol for `form_admin` Operations. Eliminates standing `form_admin` Postgres credentials (SOC 2 CC6.1/CC6.3 risk). Architecture: `pam-elevation-service` Cloudflare Worker mediates all elevation requests; three access tiers (`read_only` 4h self-approve+TOTP, `read_write` 1h manager-approval+TOTP, `destructive` 15min dual-person+FIDO2 WebAuthn); `pam-db-proxy` Supabase Edge Function opens scoped Postgres connection via `SET ROLE form_admin` + `SET app.pam_session_id` GUC (database-layer audit linkage); `RESET ROLE` after each statement batch; PgBouncer session mode required (OQ-SSO-24.2). Break-glass: separate Cloudflare Access application (`form-break-glass`), mTLS + named identity list (max 5, quarterly review), 2h non-renewable session, immediate PagerDuty P0 + compliance email on activation, mandatory 72h post-incident review with GitHub Issue auto-creation, `pam_break_glass_reviews` Postgres table for outcome recording. KV: `PAM_KV` namespace (separate from SESSION_REVOCATION_KV and SSO_KV); `pam:session:{id}` primary + `pam:by_admin:{admin_user_id}:{id}` secondary index; `pam:suspended:{admin_user_id}` for AL-PAM-02 lockout. Six DEC-030 HMAC-chained audit events: `pam.elevation_requested` (HIGH, 7yr), `pam.elevation_approved` (HIGH, 7yr), `pam.elevation_denied` (HIGH, 7yr), `pam.session_expired` (STANDARD, 7yr), `pam.break_glass_activated` (CRITICAL, 7yr — post-hoc two-person review), `pam.break_glass_expired` (HIGH, 7yr). Three alerting rules: AL-PAM-01 P0 (break-glass activated), AL-PAM-02 P1 (> 3 denied elevations/1h same admin — auto-suspend), AL-PAM-03 P2 (session used after KV expiry — clock skew detection). SOC 2: CC6.1 (no standing access), CC6.2 (approval workflow), CC6.3 (auto-expiry via KV TTL), CC6.7 (mTLS on all PAM endpoints + TLS to Postgres). Four open questions: OQ-SSO-24.1 (proxy placement: Hyperdrive vs. Edge Function — security-engineer, P1, before M4), OQ-SSO-24.2 (PgBouncer session mode for PAM pool — platform-engineer, P0, before M4), OQ-SSO-24.3 (form_admin vs. form_break_glass role naming — compliance-officer, P2, M5), OQ-SSO-24.4 (FIDO2 hardware key procurement — security-engineer + eng-manager, P0, before M4). Implementation checklist: 10 tasks (6× P0 M4, 2× P1 M4, 2× P1 M5).*

---

## 25. Per-Tenant Authentication Policy Engine — IP Allowlist, MFA Enforcement & Session Policy

> **Closes G-006 (🟡 → 🟢 upon deployment).** G-006 was first documented in §9 (Open Questions / Gaps): "Schema column exists; the Cloudflare Worker that enforces IP allowlist on SSO callbacks is not written." §16.10 added the admin UI design specification. This section adds the complete enforcement implementation: Worker middleware covering all authenticated routes (not only SSO callbacks), MFA enforcement policy evaluation, KV-backed policy cache, policy change management, admin lockout recovery, DEC-030 audit trail, alerting rules, and SOC 2 control mapping.
>
> **Cross-references:** §4.2 (`tenant_sso_configs` DDL) · §6 (security controls) · §12 (session token lifecycle) · §16.10 (IP allowlist admin UI design) · §22 (session revocation KV) · §24 (PAM break-glass, used in lockout recovery) · `docs/ENTERPRISE.md` ("IP allowlist (Enterprise only)", "MFA enforcement per tenant") · `docs/DATA_MODEL.md §19` (`security.ip_allowlist_enabled` feature flag) · `docs/AUDIT_LOG_SCHEMA.md` (DEC-030) · `docs/SOC2_READINESS.md` CC6.1/CC6.2/CC6.6/CC6.8.

---

### 25.1 Purpose and Scope

Enterprise customers operate in regulated environments where not all authentication flows originate from trusted corporate networks. Two controls address this:

1. **IP allowlist** — restricts SSO login and API session use to CIDR ranges declared by the customer's IT team. Employees on unrecognised networks (home Wi-Fi, airport, non-corporate VPN) cannot authenticate, regardless of whether their IdP credentials are valid. This is a compensating control for credential theft from a non-corporate endpoint.

2. **MFA enforcement** — requires a second factor at every session creation, independent of what the IdP has already enforced. Relevant when a customer's IdP has MFA enabled for some but not all users, or when the contract requires FORM to independently verify MFA completion (regulated industry requirement).

**Scope:** Enterprise and Growth tenants only. `security.ip_allowlist_enabled` feature flag (DATA_MODEL §19) must be active for a tenant before `ip_allowlist` enforcement is respected; attempting to configure allowlist CIDRs on Starter without this flag returns 403 from the Admin Dashboard API.

**Out of scope:** Consumer mobile (Apple Sign In); form_admin PAM sessions (covered by §24); Cloudflare Access itself (managed externally to this document).

**Privacy constraint:** IP addresses of blocked authentication attempts are sensitive. In DEC-030 events, blocked client IPs are stored as a SHA-256 hash (hex, first 32 chars) rather than plaintext, to avoid the audit log becoming a correlatable log of employee home network addresses. The full IP is available only to security-engineer under PAM `read_write` elevation, for incident investigation.

---

### 25.2 Policy Dimensions — What Is Enforced and When

Three policy dimensions are evaluated in sequence on every authentication event. The evaluation order is critical — IP allowlist is evaluated first (before any credential validation), so a blocked IP receives no information about whether credentials would have been accepted.

| # | Dimension | Enforcement trigger | Controlled by | Tier gate |
|---|---|---|---|---|
| 1 | **IP allowlist** | SSO callback (SAML/OIDC), token refresh, SCIM API | `ip_allowlist_enabled` feature flag + `ip_allowlist` JSONB | Enterprise only |
| 2 | **MFA enforcement** | SSO callback (session creation); evaluated against IdP `amr` claim or FORM TOTP second factor | `mfa_enforcement_mode` column | Growth + Enterprise |
| 3 | **Session policy** | Token refresh (§12); idle timeout sweeper | `session_max_duration_hours`, `idle_timeout_minutes`, `require_reauth_on_role_change` | All SSO tiers |

Session policy (dimension 3) is already implemented in §12. This section adds dimensions 1 and 2, and adds the policy change management layer that governs all three.

---

### 25.3 Schema Additions to `tenant_sso_configs`

The existing DDL (§4.2) has `ip_allowlist INET[]` (array of raw CIDR strings) and `require_mfa BOOLEAN`. Both are extended to richer types to support labelling, enforcement modes, and audit-safe representation.

```sql
-- Migration: 0047_auth_policy_engine.sql
-- Replaces the flat INET[] and BOOLEAN with richer, labelled columns.
-- Applies without a table lock (ALTER TABLE ... ADD COLUMN is fast on Postgres; USING clause handles type cast).

BEGIN;

-- 1. ip_allowlist: INET[] → JSONB (labels + enabled flag).
-- Backfill: existing INET[] entries are migrated to the new JSONB format by the migration.
ALTER TABLE tenant_sso_configs
  ADD COLUMN IF NOT EXISTS ip_allowlist_config JSONB NOT NULL DEFAULT '{"enabled": false, "entries": []}'::jsonb,
  ADD COLUMN IF NOT EXISTS mfa_enforcement_mode TEXT NOT NULL DEFAULT 'none'
    CHECK (mfa_enforcement_mode IN ('none', 'recommended', 'required')),
  ADD COLUMN IF NOT EXISTS mfa_grace_hours INT NOT NULL DEFAULT 24
    CHECK (mfa_grace_hours BETWEEN 1 AND 168);

-- Backfill existing INET[] rows into the new JSONB column.
UPDATE tenant_sso_configs
SET ip_allowlist_config = jsonb_build_object(
  'enabled', (ip_allowlist IS NOT NULL AND cardinality(ip_allowlist) > 0),
  'entries', COALESCE(
    (SELECT jsonb_agg(jsonb_build_object(
      'id',    gen_random_uuid(),
      'cidr',  elem::text,
      'label', 'Migrated from legacy column',
      'added_at', now()
    ))
    FROM unnest(ip_allowlist) AS elem),
    '[]'::jsonb
  )
)
WHERE ip_allowlist IS NOT NULL AND cardinality(ip_allowlist) > 0;

-- Backfill require_mfa BOOLEAN → mfa_enforcement_mode TEXT.
UPDATE tenant_sso_configs
SET mfa_enforcement_mode = 'required'
WHERE require_mfa = true;

-- Drop legacy columns once backfill is confirmed (separate migration, run after 72h soak).
-- ALTER TABLE tenant_sso_configs DROP COLUMN IF EXISTS ip_allowlist;
-- ALTER TABLE tenant_sso_configs DROP COLUMN IF EXISTS require_mfa;

COMMENT ON COLUMN tenant_sso_configs.ip_allowlist_config IS
  'JSONB: {enabled: bool, entries: [{id: uuid, cidr: text, label: text, added_at: timestamptz, added_by: uuid}]}. Max 50 entries. Enforced by auth-policy-middleware Worker.';
COMMENT ON COLUMN tenant_sso_configs.mfa_enforcement_mode IS
  'none = no FORM-side MFA gate (IdP MFA still applies). recommended = display banner if IdP amr lacks MFA. required = block session creation if IdP amr lacks mfa/totp/otp/hwk.';

COMMIT;
```

**`ip_allowlist_config` JSONB schema:**

```typescript
interface IpAllowlistConfig {
  enabled: boolean;
  entries: Array<{
    id: string;          // UUID, immutable once created
    cidr: string;        // e.g. "10.0.0.0/8" or "2001:db8::/32"
    label: string;       // human-readable, max 100 chars, free text
    added_at: string;    // ISO 8601 UTC
    added_by: string;    // form_admin or tenant_owner UUID
  }>;
}

// Constraints:
// - Maximum 50 entries per tenant (enforced at API layer before DB write).
// - Each CIDR must be a valid IPv4 or IPv6 prefix in canonical form.
// - An empty entries array with enabled: true blocks ALL logins — the admin UI shows a hard warning before this state is saved.
// - A /0 prefix (0.0.0.0/0 or ::/0) is rejected by the API layer — it is functionally equivalent to disabling the allowlist and is almost certainly a misconfiguration.
```

---

### 25.4 KV-Backed Policy Cache (`SSO_KV`)

Reading `tenant_sso_configs` from Supabase on every authenticated request is not acceptable at scale. The policy cache uses the existing `SSO_KV` namespace (introduced in §22 for session revocation metadata) with a separate key prefix.

**Cache key schema:**

```
SSO_KV:auth_policy:{tenant_id}
```

**Value schema:**

```typescript
interface CachedAuthPolicy {
  tenant_id: string;
  ip_allowlist: IpAllowlistConfig;    // embedded full config
  mfa_enforcement_mode: 'none' | 'recommended' | 'required';
  mfa_grace_hours: number;
  session_max_duration_hours: number;
  idle_timeout_minutes: number | null;
  cached_at: string;                  // ISO 8601 UTC
  version: number;                    // monotonic, incremented on every policy write
}
```

**TTL:** 60 seconds. The worst-case policy propagation lag for a newly blocked CIDR is 60 seconds. This is acceptable — the risk window of one minute is far smaller than the credential theft → account takeover timeline in practice.

**Cache invalidation (active push):** When a `tenant_owner` or `tenant_admin` saves any authentication policy change via the Admin Dashboard API (`PATCH /v1/admin/sso/policy`), the API Worker atomically:
1. Writes the new policy to `tenant_sso_configs` in Supabase (inside a transaction with the DEC-030 audit event emit).
2. Immediately deletes `SSO_KV:auth_policy:{tenant_id}` (or writes the new value directly, bypassing the 60s TTL lag for the change initiator's own tenant).

**Cache miss handling:** On a KV miss, the middleware fetches from Supabase and re-populates the cache. If Supabase is unavailable, the middleware fails **closed** (denies the request with `503 auth_policy_unavailable`) rather than failing open. This matches the §24 PAM fail-closed pattern — a brief Supabase hiccup should not transiently disable a tenant's IP security perimeter.

---

### 25.5 IP Allowlist Enforcement Middleware

The §16.10 design restricted enforcement to SSO callback routes only. This section extends it to cover all authenticated Worker routes, which is necessary because a stolen FORM session token (§12) can be used directly against the API without going through SSO. An IP allowlist that only fires at login-time provides no protection against a session token exfiltrated to a non-corporate device.

**Enforcement points:**

| Route pattern | Enforcement | Notes |
|---|---|---|
| `POST /auth/saml/callback/*` | IP check before SAML assertion validation | §16.10 design — now fully implemented here |
| `GET/POST /auth/oidc/callback` | IP check before OIDC token exchange | §16.10 design — now fully implemented here |
| `POST /auth/token/refresh` | IP check before refresh token validation | NEW — prevents using stolen refresh tokens from non-allowlisted networks |
| `* /v1/admin/*` (Admin Dashboard API) | IP check after JWT validation, before business logic | NEW — protects admin actions from non-corporate IPs |
| `POST /v1/scim/*` | IP check after bearer token validation | NEW — protects SCIM provisioning endpoint (SCIM tokens are long-lived; limiting to IdP's IP range is defence-in-depth) |
| `* /v1/api/*` (end-user API) | **Not enforced** — allowlist applies to admin/SSO surfaces only, not end-user API routes | Rationale: employees may use FORM app from home or on mobile; the allowlist is for administrative access, not product use |

**Middleware implementation (`src/workers/auth-policy-middleware/ip-allowlist.ts`):**

```typescript
import { Netmask } from 'netmask';   // well-tested CIDR library, handles IPv4 + IPv6

export interface IpAllowlistEntry {
  id: string;
  cidr: string;
  label: string;
}

export interface IpAllowlistEnforcementResult {
  allowed: boolean;
  matched_entry_id: string | null;   // ID of matching allowlist entry, null if blocked
  client_ip_hash: string;            // SHA-256 hex[:32] of client IP for DEC-030 event
}

export async function enforceIpAllowlist(
  request: Request,
  policy: CachedAuthPolicy,
  env: Env,
): Promise<IpAllowlistEnforcementResult | null> {
  const { ip_allowlist } = policy;

  // Pass-through if disabled or no entries configured.
  if (!ip_allowlist.enabled || ip_allowlist.entries.length === 0) {
    return null;
  }

  // CF-Connecting-IP is set by Cloudflare and cannot be spoofed by a request
  // that reaches Workers via the Cloudflare network. It is the true client IP.
  const clientIp = request.headers.get('CF-Connecting-IP');
  if (!clientIp) {
    // Should never happen on Cloudflare Workers — log as anomaly.
    return { allowed: false, matched_entry_id: null, client_ip_hash: 'unknown' };
  }

  const clientIpHash = await hashIp(clientIp, env);

  for (const entry of ip_allowlist.entries) {
    try {
      const block = new Netmask(entry.cidr);
      if (block.contains(clientIp)) {
        return { allowed: true, matched_entry_id: entry.id, client_ip_hash: clientIpHash };
      }
    } catch {
      // Invalid CIDR in DB — skip entry, log as AUTH-POLICY-WARN.
      // A malformed stored CIDR should not crash the allowlist check.
      console.warn(`[auth-policy] invalid CIDR in allowlist entry ${entry.id}: ${entry.cidr}`);
    }
  }

  return { allowed: false, matched_entry_id: null, client_ip_hash: clientIpHash };
}

async function hashIp(ip: string, env: Env): Promise<string> {
  const data = new TextEncoder().encode(ip + env.IP_HASH_SALT);
  const hashBuffer = await crypto.subtle.digest('SHA-256', data);
  return Array.from(new Uint8Array(hashBuffer))
    .map(b => b.toString(16).padStart(2, '0'))
    .join('')
    .slice(0, 32);   // 128-bit prefix — sufficient for uniqueness, reduces storage
}
```

**`IP_HASH_SALT`:** A Cloudflare Workers Secret, rotated annually. The salt prevents rainbow-table attacks against hashed IPs in the audit log. Rotation requires re-emitting no events — the salt only affects future events; historical events remain at their previously hashed values.

**Response when blocked:**

```typescript
function ipBlockedResponse(tenantId: string, clientIpHash: string): Response {
  return new Response(
    JSON.stringify({
      error: 'access_denied',
      error_description:
        'Your network is not authorised to access this organisation. Contact your IT administrator.',
      tenant_id: tenantId,
    }),
    {
      status: 403,
      headers: { 'Content-Type': 'application/json' },
    }
  );
}
```

The response deliberately omits the blocked IP and the list of allowed CIDRs — this prevents a blocked attacker from learning the network topology.

---

### 25.6 MFA Enforcement

**`mfa_enforcement_mode` values:**

| Mode | Behaviour | Use case |
|---|---|---|
| `none` | No FORM-side MFA gate. IdP MFA still applies per IdP policy. | Default; tenants where IdP already enforces MFA universally |
| `recommended` | FORM checks for MFA in the IdP `amr` claim. If absent, the session is created but an Admin Dashboard banner shows: "X users logged in without MFA this week." | Useful for progressive rollout visibility before enforcing |
| `required` | FORM blocks session creation if the IdP `amr` claim does not contain at least one of: `mfa`, `totp`, `otp`, `hwk`, `swk`, `fido`, `pop`. Returns `401 mfa_required` with a user-facing message. | Regulated industries; contracts specifying FORM-side MFA guarantee |

**Where `amr` comes from:**

- **OIDC (Okta, Azure AD, Google Workspace):** The `amr` claim is part of the OIDC ID token. FORM's OIDC callback (`/auth/oidc/callback`) already parses the ID token; the `amr` array is extracted and stored in `enterprise_sessions.auth_context JSONB` (§12.2).
- **SAML 2.0:** The `AuthnContextClassRef` element in the SAML assertion maps to an MFA signal. FORM maps `urn:oasis:names:tc:SAML:2.0:ac:classes:PasswordProtectedTransport` → no MFA; all other values (SmartCard, TimeSyncToken, etc.) → MFA present. The IdP-specific mapping is defined per `tenant_sso_configs.saml_attribute_mapping`.

**MFA enforcement middleware (`src/workers/auth-policy-middleware/mfa-enforcement.ts`):**

```typescript
const MFA_SATISFYING_AMR = new Set([
  'mfa', 'totp', 'otp', 'hwk', 'swk', 'fido', 'pop', 'sms',
]);

export function evaluateMfaEnforcement(
  amrClaims: string[],
  mode: 'none' | 'recommended' | 'required',
): { satisfied: boolean; warning: boolean } {
  const hasMfa = amrClaims.some(claim => MFA_SATISFYING_AMR.has(claim));

  if (mode === 'none') return { satisfied: true, warning: false };
  if (mode === 'recommended') return { satisfied: true, warning: !hasMfa };
  // mode === 'required'
  return { satisfied: hasMfa, warning: false };
}
```

**Grace period for existing sessions when `mfa_enforcement_mode` changes to `required`:**

When a tenant admin upgrades from `none` or `recommended` to `required`, existing sessions without MFA signals should not be immediately invalidated — this would log out all users simultaneously. Instead:

1. The policy change is written to `tenant_sso_configs` and the DEC-030 `sso.mfa_enforcement_changed` event is emitted.
2. For `mfa_grace_hours` (default 24h, range 1–168h), existing sessions are evaluated against `recommended` semantics (pass-through with warning flag set in session context).
3. After the grace window, the session sweeper (§12.5) marks sessions without `auth_context.mfa_satisfied: true` as `revoked_reason = 'mfa_policy_enforcement'`.
4. A `sso.mfa_enforcement_sweep_completed` DEC-030 event records the count of sessions revoked and the count that already satisfied MFA.

**`sms` in the satisfying set is intentional.** SMS-based OTP is weaker than TOTP but is acceptable as a FORM-side MFA signal. Enterprise customers with stricter requirements (phishing-resistant MFA only) must configure this at their IdP, where `amr` = `hwk` (hardware key) or `fido` enforces the stronger requirement. FORM enforces that *some* second factor was used; it does not adjudicate between factor types. If a customer requires phishing-resistant MFA only, they must configure that at their IdP and inform FORM via the onboarding checklist (§7.1).

---

### 25.7 Policy Change Management

All authentication policy changes are sensitive: enabling an IP allowlist with a typo can lock out an entire tenant; disabling MFA enforcement removes a security control that may be mandated by contract. The following safeguards apply to every `PATCH /v1/admin/sso/policy` call.

**Validation before write:**

| Change | Validation | Block condition |
|---|---|---|
| `ip_allowlist.enabled = true` | Check that `request.headers.get('CF-Connecting-IP')` is within the submitted allowlist | Block if the admin's own IP is not in the new allowlist (lockout prevention) |
| `ip_allowlist.entries` modified | Validate each CIDR using `Netmask` library; reject `/0` prefixes | Block on any invalid CIDR |
| `mfa_enforcement_mode = 'required'` | Estimate sessions without MFA that would be revoked after grace period | Show count in API response; do not block (user accepted the impact) |
| Any change | Rate-limit: max 5 policy changes per tenant per 5 minutes | Return `429 policy_change_rate_limit` if exceeded |

**Propagation timeline:**

| T+0 | Policy saved to `tenant_sso_configs`. DEC-030 event emitted in-transaction (§25.9). |
| T+0 | `SSO_KV:auth_policy:{tenant_id}` deleted (active cache invalidation). |
| T+0–60s | Workers serving this tenant start fetching updated policy from Supabase on cache miss. |
| T+60s | All Workers have the new policy (maximum propagation lag = KV TTL). |
| T+mfa_grace_hours | MFA sweep job runs for `required` mode transitions (§25.6). |

**Session impact of policy tightening:**

| Change | Session impact |
|---|---|
| `ip_allowlist.enabled = true` | No existing sessions invalidated. Next token refresh from a non-allowlisted IP returns `401`. |
| `ip_allowlist.entries` narrowed (remove a CIDR) | No existing sessions invalidated. Next refresh from a removed CIDR returns `401`. |
| `session_max_duration_hours` reduced | On next refresh, if `expires_at > now() + new_duration`, `expires_at` is capped (§12.4). |
| `mfa_enforcement_mode = 'required'` | Grace period applies; sessions revoked after `mfa_grace_hours` if no MFA signal. |
| `ip_allowlist.enabled = false` | No sessions affected. IP gate immediately drops on next Worker cache refresh. |

---

### 25.8 Admin Lockout Recovery

Two lockout scenarios can occur. Both require FORM support intervention via PAM break-glass (§24.4).

#### 25.8.1 IP Allowlist Lockout

**Scenario:** A tenant admin enables the IP allowlist, or removes a CIDR, and accidentally locks out all tenant users — including themselves.

**Detection:** AL-AUTH-POLICY-01 fires when `sso.ip_blocked` event rate for a given `tenant_id` exceeds 20 blocks in 5 minutes with zero successful logins in the same window.

**Recovery procedure:**

| Step | Action | Who |
|---|---|---|
| 1 | PagerDuty alert raised by AL-AUTH-POLICY-01; on-call engineer notified | devops-lead (on-call) |
| 2 | Verify lockout: query `sso.ip_blocked` events for `tenant_id` in last 30 minutes — confirm 100% block rate | on-call engineer |
| 3 | Obtain PAM `read_write` elevation (§24.3) — requires manager approval + TOTP | security-engineer |
| 4 | Via PAM db-proxy: `UPDATE tenant_sso_configs SET ip_allowlist_config = jsonb_set(ip_allowlist_config, '{enabled}', 'false') WHERE tenant_id = '{tenant_id}'` | security-engineer under PAM |
| 5 | Delete `SSO_KV:auth_policy:{tenant_id}` to force immediate cache invalidation | on-call engineer |
| 6 | Emit `sso.auth_policy_lockout_recovery` DEC-030 CRITICAL event (§25.9) | security-engineer |
| 7 | Contact tenant IT admin via CSM; inform them the allowlist has been disabled; walk through correct CIDR configuration | customer-success |
| 8 | File post-incident note in tenant's CSM account record; add to CS FEHS risk signals (COST_MODEL §26.9) | customer-success |

**Lockout prevention (primary):** The Admin Dashboard API validates that the admin's own `CF-Connecting-IP` is within the new allowlist before saving it (§25.7). This catches the most common case.

#### 25.8.2 MFA Bypass for Locked-Out Individual User

**Scenario:** A single user's MFA device is lost or reset, and `mfa_enforcement_mode = 'required'` is blocking their login.

This is not a mass lockout — it is a routine IT support scenario. The tenant's own IT admin (role: `tenant_admin`) can grant a temporary bypass via the Admin Dashboard:

`PATCH /v1/admin/users/{user_id}/mfa-bypass`

This sets a `mfa_bypass_until` timestamp (maximum 48h from now) in `tenant_users` and emits `sso.mfa_bypass_granted` (CRITICAL, §25.9). The user can complete login without the MFA gate until the bypass expires or the user re-enrolls their MFA device. FORM support cannot grant this bypass directly — it must come from the tenant's own admin, preserving the chain of custody under the tenant's GDPR controller obligation.

---

### 25.9 DEC-030 HMAC-Chained Audit Events

All events below are emitted through the `emit-audit-event` Cloudflare Worker (HMAC key version per §58 dual-key design). No event may be inserted directly to `audit_log_events` via Supabase SQL.

**Privacy constraint:** `user_id` may only appear in events that follow a successful authentication. Events fired *before* authentication (specifically `sso.ip_blocked`, emitted before credentials are validated) must omit `user_id` entirely. The `client_ip_hash` field uses the `IP_HASH_SALT`-keyed SHA-256[:32] scheme from §25.5.

| Event type | Severity | Retention | Key metadata fields | Trigger |
|---|---|---|---|---|
| `sso.auth_policy_updated` | HIGH | 7 years | `tenant_id`, `changed_by_user_id`, `policy_dimension` (ip_allowlist / mfa_enforcement / session_policy), `change_summary` (human-readable diff — no IP values, just CIDR count changes), `policy_version_before`, `policy_version_after` | Any `PATCH /v1/admin/sso/policy` that results in a DB write |
| `sso.ip_allowlist_entry_added` | HIGH | 7 years | `tenant_id`, `changed_by_user_id`, `entry_id`, `cidr_prefix_length` (e.g. 24 for /24 — not the full CIDR, to avoid storing network topology in audit events), `label` | CIDR added to `ip_allowlist_config.entries` |
| `sso.ip_allowlist_entry_removed` | HIGH | 7 years | `tenant_id`, `changed_by_user_id`, `entry_id`, `cidr_prefix_length`, `label` | CIDR removed from `ip_allowlist_config.entries` |
| `sso.ip_allowlist_toggled` | HIGH | 7 years | `tenant_id`, `changed_by_user_id`, `enabled_before` (bool), `enabled_after` (bool), `entry_count_at_toggle` | `ip_allowlist_config.enabled` changed |
| `sso.ip_blocked` | HIGH | 7 years | `tenant_id`, `client_ip_hash` (SHA-256[:32]), `enforcement_point` (saml_callback / oidc_callback / token_refresh / admin_api / scim_api), `policy_version` | Any request blocked by IP allowlist |
| `sso.mfa_enforcement_changed` | HIGH | 7 years | `tenant_id`, `changed_by_user_id`, `mode_before`, `mode_after`, `mfa_grace_hours`, `estimated_sessions_affected` | `mfa_enforcement_mode` changed |
| `sso.mfa_enforcement_sweep_completed` | STANDARD | 3 years | `tenant_id`, `sessions_revoked`, `sessions_already_satisfied`, `sweep_triggered_at`, `grace_expired_at` | After MFA grace period ends; sweep job runs |
| `sso.mfa_bypass_granted` | CRITICAL | 7 years | `tenant_id`, `granted_by_user_id` (tenant_admin), `subject_user_id`, `bypass_duration_hours`, `reason` (free text, max 500 chars) | `tenant_admin` grants per-user MFA bypass |
| `sso.auth_policy_lockout_recovery` | CRITICAL | 7 years | `tenant_id`, `recovery_type` (ip_allowlist_disabled / policy_reset), `recovery_performed_by` (PAM session ID — not user_id, as this is a FORM-internal action), `pam_session_id`, `affected_config_field`, `customer_notified` (bool) | FORM support performs lockout recovery via PAM |

**HMAC chain requirement:** Every event must include `prev_hash` linking it to the preceding event in the chain. The `sso.ip_blocked` event fires on every blocked request; in high-block scenarios (misconfiguration lockout), this may emit hundreds of events per minute. The `emit-audit-event` Worker must handle burst writes without dropping events — use Cloudflare Queues (FIFO consumer) for block events to smooth write pressure while maintaining HMAC chain ordering. The queue consumer appends events to the chain sequentially.

---

### 25.10 Alerting Rules

| Alert ID | Condition | Severity | Runbook | Owner |
|---|---|---|---|---|
| **AL-AUTH-POLICY-01** | `sso.ip_blocked` rate for a single `tenant_id` exceeds 20 events in 5 minutes **AND** zero `sso.session_created` events for the same tenant in the same window | **P1** | Check `sso.ip_allowlist_toggled` or `sso.ip_allowlist_entry_removed` in the 10 minutes preceding; if yes, possible misconfiguration lockout — follow §25.8.1. If no recent policy change, possible credential stuffing from non-corporate network — notify tenant's IT admin. | devops-lead |
| **AL-AUTH-POLICY-02** | `sso.mfa_enforcement_changed` event with `mode_after = 'none'` on a tenant with `plan = 'enterprise'` | **P1** | Removing MFA enforcement on an enterprise tenant may violate the customer contract's security annexe. compliance-officer must review within 4h; if unauthorised, initiate §25.8 lockout recovery to restore policy. Emit `sso.auth_policy_lockout_recovery` with `recovery_type = 'policy_reset'` if restored. | compliance-officer |
| **AL-AUTH-POLICY-03** | `sso.ip_blocked` event for an `enforcement_point = 'admin_api'` after no prior blocks for the same tenant in 7 days | **P2** | A previously-allowed admin IP is now blocked — could be a new office, VPN change, or travel. Notify CSM to check with tenant IT admin within 1 business day. Low urgency: users are not locked out (admin_api access is not required for end-user product use). | customer-success |

---

### 25.11 SOC 2 Evidence Mapping

| Criterion | Description | How §25 Controls Satisfy It | Evidence artefact |
|---|---|---|---|
| CC6.1 | Logical and physical access security controls limit access | IP allowlist restricts authentication to declared corporate networks, adding a network-layer gate to credential-based access. `sso.ip_blocked` events provide auditor evidence of enforcement. | AUTH-POLICY-E-001: 30-day export of `sso.ip_blocked` events showing zero successful logins from non-allowlisted IPs during the observation period |
| CC6.2 | Authentication requires appropriate credentials including MFA | `mfa_enforcement_mode = 'required'` enforces a second factor at FORM's session creation layer, independent of IdP configuration. Blocks session creation without MFA `amr` claim. | AUTH-POLICY-E-002: `sso.mfa_enforcement_changed` event log showing mode transitions and effective dates; cross-reference with `enterprise_sessions.auth_context` sample showing `mfa_satisfied: true` |
| CC6.6 | Controls prevent introduction of unauthorised software | Not directly applicable to auth policy. *(IP allowlist is a network access control, not a software supply chain control.)* | — |
| CC6.8 | Controls protect against unauthorised changes to configuration | All policy changes emit HMAC-chained DEC-030 events, creating a non-repudiable audit trail. The 5-change/5-minute rate limit prevents rapid-fire policy manipulation. | AUTH-POLICY-E-003: DEC-030 `sso.auth_policy_updated` event chain for the observation period showing all policy changes, their actors, and HMAC continuity verification |
| CC7.2 | The entity monitors system components for anomalies | AL-AUTH-POLICY-01 detects lockout conditions (mass IP blocks). AL-AUTH-POLICY-02 detects unexpected MFA policy downgrades on enterprise tenants. Both alert within 5 minutes of the anomalous event. | AUTH-POLICY-E-004: PagerDuty alert history showing AL-AUTH-POLICY-01 and AL-AUTH-POLICY-02 configuration and any triggered incidents during the observation period |

---

### 25.12 Gap Status Update

| Gap ID | Prior status | New status | Closed by |
|---|---|---|---|
| **G-006** | 🟡 Authored (§16.10 UI design; Worker implementation pending) | 🟢 **Closed** (upon deployment of `auth-policy-middleware` Worker per §25.14 checklist items 1–4) | §25.5 IP allowlist enforcement Worker full spec; §25.6 MFA enforcement middleware; §25.7 policy change management |

---

### 25.13 Open Questions

| ID | Question | Priority | Owner | Target |
|---|---|---|---|---|
| OQ-SSO-25.1 | **SCIM endpoint IP allowlist scope.** The §25.5 enforcement table includes `POST /v1/scim/*` in the IP-enforced routes, reasoning that the SCIM token is long-lived and limiting the source IP to the IdP's known IP range is defence-in-depth. However, some SCIM clients (particularly Okta) do not originate from a fixed IP range and use shared infrastructure. If the tenant enables IP allowlist and enters only their office CIDRs, SCIM provisioning will break silently. Resolution options: (a) exclude `/v1/scim/*` from IP allowlist enforcement entirely; (b) add a separate `scim_ip_allowlist` field; (c) enforce allowlist on SCIM only when a separate `scim_ip_enforcement_enabled` flag is set. **Recommended: option (c) — separate flag, default off.** **🟢 Resolved — DEC-062 (2026-06-16). See §32.3.** Option (c) adopted: `scim_ip_enforcement_enabled BOOLEAN NOT NULL DEFAULT FALSE` (migration `0076`); Admin Dashboard toggle with Okta/Azure AD warning banner. | 🟢 **Resolved** | enterprise-architect + platform-engineer | §32.3 · DEC-062 (2026-06-16) |
| OQ-SSO-25.2 | **`sms` in MFA satisfying set.** §25.6 includes `sms` as a satisfying MFA method. NIST SP 800-63B (AAL2) deprecated SMS OTP in 2017 due to SIM-swap attack risk. Some enterprise contracts in financial services may require FORM to enforce `hwk` or `fido` only. Resolution: add an optional `mfa_required_methods` JSONB array to `tenant_sso_configs` that overrides the default satisfying set (e.g., `["hwk", "fido"]` for phishing-resistant-only enforcement). Owner: security-engineer + compliance-officer. Priority: P2 — relevant only after first regulated-industry (financial services/healthcare) enterprise deal. | P2 | security-engineer + compliance-officer | Before first regulated-industry enterprise customer |
| OQ-SSO-25.3 | **IP allowlist enforcement for direct API keys.** The §12 session layer uses JWTs. Some enterprise integrations use long-lived API keys (separate from JWT sessions) for webhook configuration and reporting pulls. The current §25.5 design enforces IP allowlist on the session-based flows; it does not cover API key routes. If an API key is exfiltrated, a non-allowlisted IP can use it freely. Resolution: add IP allowlist enforcement to the API key authentication middleware in `src/workers/auth/api-key-auth.ts`. **🟢 Resolved — DEC-062 (2026-06-16). See §32.4.** `enforceIpAllowlist()` extended with `authPath: 'sso' | 'api_key'` parameter; wired into `api-key-auth.ts`; `sso.ip_blocked` DEC-030 payload extended with `auth_path` field. | 🟢 **Resolved** | platform-engineer | §32.4 · DEC-062 (2026-06-16) |
| OQ-SSO-25.4 | **Policy version vector clock.** The `policy_version` field in `CachedAuthPolicy` is a monotonic integer scoped to a single tenant. If two tenant admins simultaneously submit policy changes (unlikely but possible), the last-write-wins database update will produce a monotonic increment but the intermediate state will be invisible to the audit log. Should policy changes use optimistic locking (`WHERE policy_version = $expected_version`)? Recommended: yes — add `policy_version INT NOT NULL DEFAULT 0` to `tenant_sso_configs` with a `RAISE EXCEPTION` if the write fails the optimistic lock, surfacing the conflict to the second admin. | P2 | enterprise-architect | M5 |

---

### 25.14 Implementation Checklist

#### P0 — Must complete before first enterprise customer with IP allowlist or MFA enforcement

| # | Task | Owner | Priority | Milestone |
|---|---|---|---|---|
| 1 | Run migration `0047_auth_policy_engine.sql`: add `ip_allowlist_config JSONB`, `mfa_enforcement_mode TEXT`, `mfa_grace_hours INT` to `tenant_sso_configs`; backfill existing `ip_allowlist INET[]` and `require_mfa BOOLEAN` rows; verify zero NULL rows in new columns after migration. **Do not drop legacy columns until 72h soak (item 2).** | platform-engineer | **P0** | M4 |
| 2 | 72h soak period after migration: confirm no production errors reading from new columns; run `SELECT COUNT(*) FROM tenant_sso_configs WHERE ip_allowlist_config IS NULL OR mfa_enforcement_mode IS NULL` — must return 0; then drop legacy columns in `0048_drop_legacy_auth_policy_columns.sql`. | platform-engineer | **P0** | M4 + 3 days |
| 3 | Implement `src/workers/auth-policy-middleware/ip-allowlist.ts` per §25.5 spec. Add `netmask` npm dependency. Add `IP_HASH_SALT` Cloudflare Workers Secret. Wire into: SSO callback handlers (SAML + OIDC), `/auth/token/refresh`, `/v1/admin/*`. Resolve OQ-SSO-25.1 (SCIM endpoint scope — default: exclude SCIM from IP enforcement pending separate flag). | platform-engineer | **P0** | M4 |
| 4 | Implement `src/workers/auth-policy-middleware/mfa-enforcement.ts` per §25.6 spec. Wire into SSO callback (SAML + OIDC) after assertion validation but before session creation. Write `mfa_satisfied: bool` to `enterprise_sessions.auth_context JSONB`. Implement MFA grace sweeper job (Cloudflare Workers Cron, runs hourly) that evaluates `mfa_grace_hours` and emits `sso.mfa_enforcement_sweep_completed` DEC-030 event. | platform-engineer | **P0** | M4 |
| 5 | Implement `SSO_KV:auth_policy:{tenant_id}` cache layer per §25.4 spec: fetch-with-TTL-60s on cache miss; active invalidation on `PATCH /v1/admin/sso/policy`. Confirm fail-closed behaviour: disable Supabase in staging, confirm Worker returns `503 auth_policy_unavailable` (not fail-open). | platform-engineer | **P0** | M4 |
| 6 | Register all nine DEC-030 event types from §25.9 in `docs/AUDIT_LOG_SCHEMA.md` event registry. Validate HMAC chain in staging for the full policy change → IP block → recovery sequence. Confirm `sso.ip_blocked` events use `client_ip_hash` and omit `user_id`. | platform-engineer + compliance-officer | **P0** | M4 |
| 7 | Implement `IP_HASH_SALT`-keyed SHA-256[:32] IP hashing (§25.5 `hashIp`). Add `IP_HASH_SALT` rotation to the annual key rotation schedule (`CRYPTOGRAPHY_POLICY.md` §3 key inventory). Add `IP_HASH_SALT` to `SOC2_READINESS.md §56` key inventory table. | security-engineer | **P0** | M4 |
| 8 | Configure Cloudflare Queues consumer for `sso.ip_blocked` events (§25.9 HMAC chain requirement under burst conditions). Test burst scenario: 100 simultaneous blocked requests to same tenant; verify all events appear in `audit_log_events` with unbroken HMAC chain. | devops-lead | **P0** | M4 |

#### P1 — Before first enterprise pilot goes live

| # | Task | Owner | Priority | Milestone |
|---|---|---|---|---|
| 9 | Configure AL-AUTH-POLICY-01, AL-AUTH-POLICY-02, AL-AUTH-POLICY-03 in PagerDuty. Test AL-AUTH-POLICY-01 by triggering a synthetic lockout in staging (enable allowlist with a /32 CIDR that excludes the test client; confirm PagerDuty fires within 5 minutes). | devops-lead | **P1** | M5 |
| 10 | Implement `PATCH /v1/admin/users/{user_id}/mfa-bypass` endpoint (§25.8.2): validate `tenant_admin` role; set `mfa_bypass_until` on `tenant_users`; emit `sso.mfa_bypass_granted` CRITICAL DEC-030 event; cap bypass at 48h. Add `mfa_bypass_until TIMESTAMPTZ` column to `tenant_users` (migration `0049_mfa_bypass.sql`). Wire bypass check into MFA enforcement middleware (§25.6). | platform-engineer | **P1** | M5 |
| 11 | Add `security.ip_allowlist_enabled` feature flag check to `PATCH /v1/admin/sso/policy` — return 403 with `upgrade_required` error code if Starter tenant attempts to configure IP allowlist. Validate that `DATA_MODEL §19` `feature_flag_registry` has `security.ip_allowlist_enabled` with tier = Enterprise only. | platform-engineer | **P1** | M5 |
| 12 | Collect AUTH-POLICY-E-001 through AUTH-POLICY-E-004 evidence artefacts (§25.11) 30 days after M4 go-live; store in `compliance/evidence/auth-policy/`; cross-reference in `docs/SOC2_READINESS.md` CC6.1 and CC6.2 evidence tables. Update G-006 status from 🟡 → 🟢 in `docs/SSO_SCIM_IMPLEMENTATION.md §9` gap register. | compliance-officer | **P1** | M5 |

#### P2 — Post-GA improvements

| # | Task | Owner | Priority | Milestone |
|---|---|---|---|---|
| 13 | Resolve OQ-SSO-25.2 (`sms` in MFA satisfying set): add optional `mfa_required_methods JSONB` override column to `tenant_sso_configs` if first regulated-industry customer requires phishing-resistant MFA only. | security-engineer + compliance-officer | **P2** | Before first financial-services customer |
| 14 | Resolve OQ-SSO-25.3 (API key IP enforcement): add IP allowlist check to `src/workers/auth/api-key-auth.ts`. | platform-engineer | **P1** | M5 |
| 15 | Resolve OQ-SSO-25.4 (optimistic locking on policy version): add `policy_version INT NOT NULL DEFAULT 0` to `tenant_sso_configs` and optimistic-lock check in `PATCH /v1/admin/sso/policy`. | enterprise-architect | **P2** | M6 |

---

*v1.7 additions (2026-06-04): §25 Per-Tenant Authentication Policy Engine — IP Allowlist, MFA Enforcement & Session Policy. Closes G-006 (🟡 → 🟢 upon deployment of `auth-policy-middleware` Worker). TOC updated: added §22, §23, §24 entries (previously missing) and new §25. Header updated from v1.4 to v1.7. §25.1 purpose and scope: IP allowlist (network-layer gate for credential theft from non-corporate endpoint) and MFA enforcement (second factor at FORM session creation, independent of IdP); enterprise + growth tier only; `security.ip_allowlist_enabled` feature flag gate (DATA_MODEL §19); privacy constraint on client_ip_hash (SHA-256[:32] with `IP_HASH_SALT`, not plaintext). §25.2 three policy dimensions: (1) IP allowlist — all SSO/admin/SCIM routes except end-user API; (2) MFA enforcement — session creation; (3) session policy — already in §12; evaluation order: IP first (pre-credential) then MFA then session policy. §25.3 schema additions: `ip_allowlist_config JSONB` replacing `ip_allowlist INET[]` (adds per-entry labels, UUIDs, added_at, added_by); `mfa_enforcement_mode TEXT CHECK IN ('none','recommended','required')`; `mfa_grace_hours INT`; full migration `0047_auth_policy_engine.sql` with backfill; 48h-later legacy column drop migration `0048`; JSONB schema type definition with 50-entry limit, /0-prefix rejection, empty-enabled-block-all warning. §25.4 KV-backed policy cache: `SSO_KV:auth_policy:{tenant_id}` key, 60s TTL, active invalidation on policy write, fail-closed on Supabase unavailability (503, not fail-open). §25.5 IP allowlist enforcement middleware: extends §16.10 from SSO-callback-only to all authenticated routes (SSO callback, token refresh, admin API, SCIM API; end-user `/v1/api/*` excluded); TypeScript `enforceIpAllowlist()` using `netmask` npm library (IPv4+IPv6); `CF-Connecting-IP` header (Cloudflare-guaranteed, not spoofable); `hashIp()` with `IP_HASH_SALT` Workers Secret; error response omits IP and CIDR list; Cloudflare Queues consumer for burst ip_blocked events. §25.6 MFA enforcement: `evaluateMfaEnforcement()` against IdP `amr` claim (OIDC ID token) or SAML `AuthnContextClassRef`; 9 satisfying amr values; grace period for existing sessions when mode → required (mfa_grace_hours configurable 1–168h); sweep job emits `sso.mfa_enforcement_sweep_completed`. §25.7 policy change management: lockout-prevention pre-validation (admin's own IP must be in new allowlist); /0 prefix rejection; 5-changes/5-min rate limit; propagation timeline T+0 to T+60s; session impact matrix for each policy tightening type. §25.8 admin lockout recovery: IP lockout → PAM read_write elevation (§24.3) → UPDATE tenant_sso_configs → SSO_KV delete → `sso.auth_policy_lockout_recovery` CRITICAL event → CSM notification; MFA per-user bypass → tenant_admin only (not FORM support) → `mfa_bypass_until` timestamptz + `sso.mfa_bypass_granted` CRITICAL event, max 48h. §25.9 nine DEC-030 HMAC-chained events (all HIGH or CRITICAL, 7yr except `sso.mfa_enforcement_sweep_completed` STANDARD 3yr): sso.auth_policy_updated, sso.ip_allowlist_entry_added, sso.ip_allowlist_entry_removed, sso.ip_allowlist_toggled, sso.ip_blocked (no user_id, client_ip_hash only), sso.mfa_enforcement_changed, sso.mfa_enforcement_sweep_completed, sso.mfa_bypass_granted, sso.auth_policy_lockout_recovery. §25.10 three alerting rules: AL-AUTH-POLICY-01 P1 (mass ip_blocked with zero sessions → lockout), AL-AUTH-POLICY-02 P1 (MFA enforcement disabled on enterprise tenant → compliance risk), AL-AUTH-POLICY-03 P2 (admin_api block after 7-day silence → travel/VPN change, CSM notify). §25.11 SOC 2 mapping: CC6.1 (IP allowlist network gate, AUTH-POLICY-E-001), CC6.2 (MFA enforcement independent of IdP, AUTH-POLICY-E-002), CC6.8 (HMAC-chained policy change audit trail + rate limit, AUTH-POLICY-E-003), CC7.2 (AL-AUTH-POLICY-01/02 anomaly detection, AUTH-POLICY-E-004). §25.12 gap status: G-006 🟡 → 🟢 on deploy. §25.13 four open questions: OQ-SSO-25.1 SCIM endpoint IP scope P1 (recommend separate flag, default off), OQ-SSO-25.2 SMS in satisfying set P2 (regulated industries may require hwk/fido only), OQ-SSO-25.3 API key IP enforcement P1 M5, OQ-SSO-25.4 optimistic locking on policy version P2 M6. §25.14 15-item implementation checklist: 8× P0 M4 (migration, auth-policy-middleware Worker, KV cache, DEC-030 registration, IP hash salt, Queues consumer), 5× P1 M5 (alerting, MFA bypass endpoint, feature flag check, evidence collection), 2× P2 M5-M6. Owner: enterprise-architect + security-engineer + platform-engineer + compliance-officer.*

---

## 26. API Key Authentication Security — SCIM IP Scope, API Key IP Enforcement & Rotation Policy

> Owner: enterprise-architect + security-engineer + platform-engineer. Review: on any credential schema change or quarterly.
> SOC 2 evidence: CC6.1, CC6.2, CC6.4, CC6.8, CC7.2.
> Closes: OQ-SSO-25.1 (SCIM endpoint IP allowlist scope) and OQ-SSO-25.3 (API key IP enforcement). Both were P1 items from §25.13.

---

### 26.1 Purpose and Scope

§25 introduced IP allowlist enforcement across SSO and admin routes. Two P1 open questions remained unresolved at §25 ship:

1. **OQ-SSO-25.1 — SCIM endpoint IP scope.** The §25.5 enforcement table includes SCIM endpoints in the IP allowlist check. This silently breaks SCIM provisioning from providers (notably Okta, Azure AD) that do not originate from a fixed IP range. The resolution must allow tenants with IP allowlist enabled to continue SCIM provisioning without exempting SCIM from all authentication controls.

2. **OQ-SSO-25.3 — API key IP enforcement.** Long-lived tenant API keys used for webhook receivers, reporting integrations, and admin automation are not covered by the §25 IP allowlist middleware. A compromised API key can be used from any IP. This section adds IP enforcement to `src/workers/auth/api-key-auth.ts`.

**Out of scope:** SCIM bearer token security beyond IP (token rotation is in §3.2 and §16.6). JWT session IP enforcement is in §25. Consumer-tier mobile sessions are excluded from all enterprise IP enforcement.

**Privacy floor:** All IP fields in DEC-030 events from this section use `client_ip_hash` (SHA-256[:32] keyed with `IP_HASH_SALT`, same as §25.5). Plaintext IPs are never written to the audit log or any observability signal. The `IP_HASH_SALT` in use is the same Workers Secret established in §25 — no new secret is needed.

---

### 26.2 API Credential Architecture

FORM's enterprise tier uses three credential types with different lifecycle and IP enforcement requirements. This section introduces the distinctions to inform the IP enforcement design in §26.3 and §26.4.

| Credential | Type | Lifetime | IP enforcement pre-§26 | IP enforcement post-§26 |
|---|---|---|---|---|
| **JWT access token** | Short-lived bearer | 1h; refreshed at 55 min | Enforced at session creation (§25) | No change |
| **SCIM bearer token** | Long-lived provisioning credential | No expiry (rotation recommended 90d / annual) | Blocked by §25.5 if tenant uses IP allowlist | Optional separate `scim_ip_allowlist_config` (§26.3) |
| **Tenant API key** | Long-lived integration credential | No expiry (rotation policy in §26.6) | None — not covered by §25 | IP enforcement added in `api-key-auth.ts` (§26.4) |

**SCIM bearer tokens** are per-tenant secrets issued via the admin dashboard (§16.6). They are 256-bit random strings stored as HMAC-SHA256 hashes. They do not carry user identity — they carry only `tenant_id`. Every SCIM request with a valid token is trusted to modify the tenant's user directory without further authentication. This makes them administratively equivalent to a `tenant_owner` credential.

**Tenant API keys** are long-lived credentials used for:
- Enterprise webhook consumer endpoints that need to call back into the FORM API (e.g., HRIS sync triggers)
- Reporting integrations that pull aggregate metrics via REST (not admin dashboard)
- Automated admin operations (bulk seat provisioning, RBAC changes) from customer-managed scripts

These are distinct from SCIM tokens. SCIM tokens are IdP-issued and use the SCIM protocol path. API keys are human-issued via the admin dashboard and use general REST API paths. Both are long-lived and high-privilege; neither is covered by JWT expiry-based session controls.

---

### 26.3 OQ-SSO-25.1 Resolution: SCIM Endpoint IP Allowlist Scope

#### 26.3.1 Problem Statement

The §25.5 IP allowlist middleware applies to all authenticated routes including `/v1/scim/*`. When a tenant enables IP allowlist and enters only corporate office CIDRs, SCIM provisioning from an IdP (Okta, Azure AD, OneLogin) that routes traffic via shared cloud infrastructure will be blocked immediately. The failure mode is silent from the tenant's perspective: provisioning stops, no error is surfaced to the IdP admin, and FORM deprovisioning events are queued and undelivered.

The options considered in OQ-SSO-25.1 were:
- **(a)** Exclude `/v1/scim/*` from IP allowlist entirely — over-permissive; removes a useful defence-in-depth control for tenants who can provide their IdP's source IP range
- **(b)** Add a separate `scim_ip_allowlist` field — adds schema complexity; customers would need to configure two separate allowlists
- **(c)** Add a `scim_ip_enforcement_enabled BOOLEAN DEFAULT false` flag — SCIM enforcement is opt-in, defaulting to off; tenants who know their IdP's source IP range can opt in

**Resolution: option (c).** `scim_ip_enforcement_enabled` defaults to false. The general IP allowlist never applies to SCIM routes regardless of the main `ip_enforcement_enabled` state. When a tenant sets `scim_ip_enforcement_enabled = true`, SCIM requests are checked against a separate `scim_ip_allowlist_config JSONB` field (not the general `ip_allowlist_config`), because IdP source IPs are distinct from corporate office CIDRs.

#### 26.3.2 Schema Changes

Migration `0052_scim_ip_allowlist.sql`:

```sql
ALTER TABLE tenant_sso_configs
  ADD COLUMN scim_ip_enforcement_enabled BOOLEAN  NOT NULL DEFAULT false,
  ADD COLUMN scim_ip_allowlist_config    JSONB    DEFAULT NULL;

-- Constraint: if scim_ip_enforcement_enabled = true, scim_ip_allowlist_config must be non-null and non-empty
ALTER TABLE tenant_sso_configs
  ADD CONSTRAINT chk_scim_ip_allowlist_required
    CHECK (
      scim_ip_enforcement_enabled = false
      OR (
        scim_ip_allowlist_config IS NOT NULL
        AND jsonb_array_length(scim_ip_allowlist_config->'entries') > 0
      )
    );
```

`scim_ip_allowlist_config` uses the same JSONB schema as `ip_allowlist_config` (§25.3):

```typescript
interface ScimIpAllowlistConfig {
  enabled: boolean;          // must equal scim_ip_enforcement_enabled column
  entries: Array<{
    id:        string;       // UUID, client-generated
    cidr:      string;       // IPv4 or IPv6 CIDR notation
    label:     string;       // Human-readable: "Okta US region", "Azure AD EU"
    added_at:  string;       // ISO 8601
    added_by:  string;       // admin_user UUID
  }>;
}
```

**Entry limit:** 20 entries (lower than the 50-entry limit for the general allowlist — IdP IP ranges are fewer in number and more stable than office CIDRs).

**Known IdP IP range guidance:** FORM provides pre-populated CIDR lists in the admin dashboard for Okta, Azure AD, Google Workspace, and OneLogin. These are informational — FORM does not auto-update them. Tenants must confirm accuracy with their IdP before enabling enforcement. Guidance note displayed in UI: *"IdP IP ranges change without notice. Confirm current ranges directly with your provider before enabling. A misconfigured SCIM allowlist will silently halt directory provisioning."*

#### 26.3.3 Middleware Change

Update `src/workers/auth-policy-middleware/ip-allowlist.ts`:

```typescript
// Before §26: SCIM routes were included in the general IP allowlist check
// After §26: SCIM routes bypass general check; use their own enforcement path

export async function enforceIpAllowlist(
  request: Request,
  policy: CachedAuthPolicy,
  env: Env
): Promise<Response | null> {
  const path = new URL(request.url).pathname;

  // End-user API routes: never enforce (§25.5 existing behaviour)
  if (path.startsWith('/v1/api/')) return null;

  // SCIM routes: use separate scim_ip_enforcement path (§26.3)
  if (path.startsWith('/v1/scim/') || path.startsWith('/scim/')) {
    return enforceScimIpAllowlist(request, policy, env);
  }

  // All other authenticated routes: general IP allowlist (§25.5 existing behaviour)
  if (!policy.ip_enforcement_enabled || !policy.ip_allowlist_config?.enabled) {
    return null; // not enforced for this tenant
  }
  // ... existing general allowlist logic
}

async function enforceScimIpAllowlist(
  request: Request,
  policy: CachedAuthPolicy,
  env: Env
): Promise<Response | null> {
  // If SCIM IP enforcement is disabled for this tenant, pass through unconditionally
  if (!policy.scim_ip_enforcement_enabled) return null;

  const clientIp = request.headers.get('CF-Connecting-IP') ?? '';
  const allowed  = checkCidrList(clientIp, policy.scim_ip_allowlist_config?.entries ?? []);

  if (!allowed) {
    await emitScimIpBlockedEvent(env, policy.tenant_id, clientIp);
    return new Response(
      JSON.stringify({ error: 'scim_ip_not_allowed', message: 'SCIM request origin not in allowlist' }),
      { status: 403, headers: { 'Content-Type': 'application/json' } }
    );
  }
  return null;
}
```

`CachedAuthPolicy` in `SSO_KV:auth_policy:{tenant_id}` is updated to include `scim_ip_enforcement_enabled` and `scim_ip_allowlist_config`. Cache invalidation occurs on any `PATCH /v1/admin/sso/policy` write that touches either field, same as the general policy cache (§25.4).

#### 26.3.4 Admin Dashboard UI

In the admin dashboard SSO configuration panel (§16.6), add a new collapsible section **"Directory Sync IP Restriction"** below the existing "IP Allowlist" section:

```
┌─────────────────────────────────────────────────────────────┐
│ Directory Sync IP Restriction           [Enterprise only]   │
│                                                             │
│ Restrict SCIM provisioning to specific IP ranges.           │
│ Use this only if your IdP has stable, predictable           │
│ source IPs. Most IdPs (Okta, Azure AD) do not.              │
│                                                             │
│  ○  Off — SCIM provisioning accepts requests from any IP   │
│  ○  On  — Enforce allowlist below                           │
│                                                             │
│ [+ Add CIDR range]   [Import Okta ranges]  [Import Azure]   │
│                                                             │
│ ┌──────────────────────────────────────────────────────┐   │
│ │  Label              CIDR              Added          │   │
│ │  Okta US region     198.18.0.0/16     2026-05-01     │   │
│ └──────────────────────────────────────────────────────┘   │
│                                                             │
│ ⚠ Warning: enabling this will immediately block SCIM        │
│   provisioning from IPs outside this list.                  │
└─────────────────────────────────────────────────────────────┘
```

This section is gated by the `security.ip_allowlist_enabled` feature flag (same as the general IP allowlist — Enterprise only). A Starter or Growth tenant attempting to reach this configuration endpoint receives a 403 with `upgrade_required`.

---

### 26.4 OQ-SSO-25.3 Resolution: API Key IP Enforcement

#### 26.4.1 Problem Statement

`tenant_api_keys` are long-lived credentials issued via the admin dashboard for headless integrations. The current `api-key-auth.ts` Worker validates the key hash against the database but does not perform any IP check. A leaked API key is immediately and permanently usable from any network. Unlike a JWT (1h TTL), an API key that is not rotated remains exploitable indefinitely.

The §25 IP allowlist middleware operates on the session layer (post-JWT validation). API keys bypass the JWT validation path entirely — they are a separate credential type. Adding IP enforcement to `api-key-auth.ts` closes this gap.

#### 26.4.2 `tenant_api_keys` Schema

`tenant_api_keys` is referenced in `docs/AUDIT_LOG_SCHEMA.md` (`tenant.api_key_created` / `tenant.api_key_revoked`) but the table definition has not been formalised. This section provides the canonical schema.

```sql
CREATE TABLE tenant_api_keys (
  id                   UUID        PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id            UUID        NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
  key_hash             TEXT        NOT NULL,       -- HMAC-SHA256 of the raw key; raw key never stored
  key_preview          TEXT        NOT NULL,       -- Last 8 chars of the raw key; for UI display
  label                TEXT        NOT NULL,       -- Human-readable: "HRIS sync", "Reporting bot"
  created_by           UUID        REFERENCES tenant_users(id) ON DELETE SET NULL,
  created_at           TIMESTAMPTZ NOT NULL DEFAULT now(),
  last_used_at         TIMESTAMPTZ,                -- updated on each successful use; NULL = never used
  expires_at           TIMESTAMPTZ,                -- NULL = no hard expiry; age alert fires at 365d
  revoked_at           TIMESTAMPTZ,                -- NULL = active; non-NULL = revoked
  revoked_by           UUID        REFERENCES tenant_users(id) ON DELETE SET NULL,
  ip_enforcement_enabled BOOLEAN   NOT NULL DEFAULT false,
  ip_allowlist_config  JSONB       DEFAULT NULL,   -- NULL when ip_enforcement_enabled = false
  scopes               TEXT[]      NOT NULL DEFAULT '{"reporting:read"}',
                                                   -- Enum: reporting:read, admin:write,
                                                   --       webhooks:write, scim:provision
  CONSTRAINT chk_api_key_ip_allowlist
    CHECK (
      ip_enforcement_enabled = false
      OR (ip_allowlist_config IS NOT NULL
          AND jsonb_array_length(ip_allowlist_config->'entries') > 0)
    )
);

CREATE INDEX idx_tenant_api_keys_tenant_active
  ON tenant_api_keys (tenant_id)
  WHERE revoked_at IS NULL;

CREATE UNIQUE INDEX idx_tenant_api_keys_hash
  ON tenant_api_keys (key_hash)
  WHERE revoked_at IS NULL;
```

**Key generation:** The raw key is 256-bit random (`crypto.getRandomValues`, hex-encoded = 64 chars). It is returned to the admin exactly once at creation time. The stored `key_hash` is `HMAC-SHA256(API_KEY_HASH_SECRET, raw_key)` where `API_KEY_HASH_SECRET` is a Cloudflare Workers Secret (separate from `IP_HASH_SALT`).

**Scopes:** API keys carry explicit scopes. `reporting:read` permits only GET requests against `/v1/admin/reports/*`. `admin:write` is the highest scope and requires IC approval before issuance (see §26.6 rotation policy). `scim:provision` is reserved for future headless SCIM client use cases and is not issued in M5.

**RLS:** `tenant_api_keys` is RLS-enforced. The `form_api` role reads only rows where `tenant_id = current_setting('app.current_tenant_id')`. The `form_admin` role (migration runner + compliance toolchain) bypasses RLS.

#### 26.4.3 IP Enforcement Middleware

Update `src/workers/auth/api-key-auth.ts`:

```typescript
import { enforceApiKeyIpAllowlist } from '../auth-policy-middleware/ip-allowlist';

export async function authenticateApiKey(
  request: Request,
  env: Env
): Promise<{ tenantId: string; keyId: string; scopes: string[] } | Response> {
  const authHeader = request.headers.get('Authorization') ?? '';
  if (!authHeader.startsWith('Bearer ')) {
    return new Response(JSON.stringify({ error: 'missing_api_key' }), { status: 401 });
  }

  const rawKey  = authHeader.slice(7);
  const keyHash = await hmacSha256(env.API_KEY_HASH_SECRET, rawKey);

  // Fetch key record — includes ip_enforcement_enabled and ip_allowlist_config
  const keyRecord = await env.DB.prepare(
    `SELECT id, tenant_id, scopes, ip_enforcement_enabled, ip_allowlist_config, revoked_at
     FROM tenant_api_keys WHERE key_hash = ?1 LIMIT 1`
  ).bind(keyHash).first<TenantApiKey>();

  if (!keyRecord || keyRecord.revoked_at !== null) {
    return new Response(JSON.stringify({ error: 'invalid_api_key' }), { status: 401 });
  }

  // IP enforcement (new in §26)
  const ipBlock = await enforceApiKeyIpAllowlist(request, keyRecord, env);
  if (ipBlock) return ipBlock; // 403 with api_key.ip_blocked DEC-030 event already emitted

  // Update last_used_at asynchronously (non-blocking; use waitUntil)
  env.EXECUTION_CTX.waitUntil(
    env.DB.prepare(`UPDATE tenant_api_keys SET last_used_at = now() WHERE id = ?1`)
      .bind(keyRecord.id).run()
  );

  return { tenantId: keyRecord.tenant_id, keyId: keyRecord.id, scopes: keyRecord.scopes };
}
```

`enforceApiKeyIpAllowlist` in `ip-allowlist.ts`:

```typescript
export async function enforceApiKeyIpAllowlist(
  request: Request,
  key: TenantApiKey,
  env: Env
): Promise<Response | null> {
  if (!key.ip_enforcement_enabled || !key.ip_allowlist_config?.enabled) return null;

  const clientIp = request.headers.get('CF-Connecting-IP') ?? '';
  const allowed  = checkCidrList(clientIp, key.ip_allowlist_config.entries);

  if (!allowed) {
    // Emit DEC-030 api_key.ip_blocked event; ip_hash only — no plaintext IP
    await emitApiKeyIpBlockedEvent(env, key.tenant_id, key.id, clientIp);
    return new Response(
      JSON.stringify({ error: 'api_key_ip_not_allowed' }),
      { status: 403, headers: { 'Content-Type': 'application/json' } }
    );
  }
  return null;
}
```

`checkCidrList` is the shared utility from §25.5 (same `netmask` npm import). No new dependency.

#### 26.4.4 Key ID in Error Context

The `api_key.ip_blocked` DEC-030 event includes `key_id` (UUID) and `client_ip_hash` but **not** `raw_key`, `key_hash`, `key_preview`, or `tenant_id` in the plaintext error response returned to the caller. The caller receives only `{"error": "api_key_ip_not_allowed"}` — no hint about which allowlist entry rejected the request. This prevents enumeration of the allowlist from error responses.

---

### 26.5 Rotation Policy

#### 26.5.1 Rotation Triggers

| Trigger | Required action | Deadline |
|---|---|---|
| API key age ≥ 365 days | Amber age warning in admin dashboard; rotation recommended | No hard deadline — `expires_at` not enforced at 365d unless tenant sets it explicitly |
| API key age ≥ 730 days (2 years) | Red age warning; CSM notifies tenant admin | Rotation required before next QBR as contractual hygiene |
| Suspected compromise (key seen in logs from unexpected IP) | Revoke immediately; issue replacement | Same-day |
| CSM departure, IT admin offboarding, integration re-owner | Rotate as part of access review | Within 5 business days of departure |
| Annual security review (SOC 2 observation period) | Review all active keys; rotate any older than 365d | Before observation period start |
| `admin:write` scope key | Quarterly rotation mandatory | Every 90 days |

#### 26.5.2 Rotation Flow

1. Tenant admin navigates to Settings → API Keys in the admin dashboard.
2. Clicks "Rotate" on the target key. A confirmation modal appears: *"This will immediately invalidate the current key. Your integration will fail until you update it with the new key. Proceed?"*
3. A new 256-bit random key is generated server-side. Old key is **not** immediately revoked — a **24-hour overlap window** is started (same pattern as SCIM token rotation in §16.6). During overlap, both old and new keys are valid.
4. New key is displayed exactly once. Admin copies it.
5. After 24 hours, the old key's `revoked_at` is set by a pg_cron job. `api_key.rotated` and `api_key.revoked` DEC-030 events are emitted.
6. If the admin explicitly clicks "Revoke old key now", the overlap is cancelled immediately.

**Scope inheritance:** The new key inherits the same scopes as the rotated key. Scope changes require explicit admin action in a separate flow.

#### 26.5.3 Admin Dashboard Display

The API Keys panel shows:

| Label | Key preview | Scopes | Created | Last used | Age warning | IP enforcement |
|---|---|---|---|---|---|---|
| HRIS sync | `...4f9a` | `reporting:read` | 2026-01-10 | 2026-06-03 | — | Off |
| Reporting bot | `...b12c` | `reporting:read` | 2025-05-15 | 2026-05-28 | 🟡 386 days old | On (3 CIDRs) |

Age alert states:
- **0–364 days:** No indicator
- **365–729 days:** Amber pill "N days old — consider rotating"
- **730+ days:** Red pill "N days old — rotation required"
- **`admin:write` scope, >90 days:** Amber pill regardless of age

---

### 26.6 DEC-030 HMAC-Chained Events

All events in this section are appended to the FORM DEC-030 HMAC audit log chain (per `docs/AUDIT_LOG_SCHEMA.md`). Events involving credential operations use `actor_type = 'user'` (for admin-initiated actions) or `actor_type = 'system'` (for automated rotation expiry). The `actor_ip` field is omitted from all events in this section — IP is captured only as `client_ip_hash` in `ip_blocked` events to prevent IP data accumulation in the audit chain.

| Event | Severity | Retention | Payload fields | Notes |
|---|---|---|---|---|
| `api_key.created` | HIGH | 7 years | `tenant_id`, `key_id`, `label`, `scopes`, `ip_enforcement_enabled`, `created_by` | Raw key never in payload; `key_preview` (last 8 chars) in `label` context only |
| `api_key.rotated` | HIGH | 7 years | `tenant_id`, `key_id` (new key UUID), `old_key_id`, `rotation_method` (enum: `admin_manual` \| `overlap_expiry`), `rotated_by` | Old key's `api_key.revoked` event always follows within 24h; HMAC chain must link both |
| `api_key.revoked` | HIGH | 7 years | `tenant_id`, `key_id`, `revoked_by`, `reason` (enum: `rotation` \| `compromise` \| `offboarding` \| `expiry_overlap` \| `admin_request`) | `reason = 'compromise'` triggers P1 alert AL-APIKEY-03 |
| `api_key.ip_enforcement_enabled` | HIGH | 7 years | `tenant_id`, `key_id`, `cidr_count`, `enabled_by` | Enabling adds network access control to existing credential |
| `api_key.ip_enforcement_disabled` | HIGH | 7 years | `tenant_id`, `key_id`, `disabled_by`, `reason` | Disabling removes a control — HIGH severity; compliance-officer review required if `scopes` includes `admin:write` |
| `api_key.ip_blocked` | HIGH | 7 years | `tenant_id`, `key_id`, `client_ip_hash` | No `user_id`; `client_ip_hash` = SHA-256[:32] keyed with `IP_HASH_SALT`; bulk deduplication via Cloudflare Queues at > 20 blocks/5 min |
| `scim.ip_enforcement_enabled` | HIGH | 7 years | `tenant_id`, `cidr_count`, `enabled_by` | SCIM-specific IP enforcement toggle (§26.3) |
| `scim.ip_enforcement_disabled` | HIGH | 7 years | `tenant_id`, `disabled_by` | |
| `scim.ip_blocked` | HIGH | 7 years | `tenant_id`, `client_ip_hash` | IdP IP; hashed; no IdP name in payload (avoid naming vendor IPs in audit chain) |

**HMAC chain requirement:** `api_key.rotated` must be followed by `api_key.revoked` (for the old key) within the same business day. A gap between these two events in the audit chain — i.e., `api_key.rotated` visible but no `api_key.revoked` for the `old_key_id` within 26 hours — is a chain anomaly that fires AL-APIKEY-02.

---

### 26.7 Alerting Rules

| Alert ID | Condition | Severity | Response | Owner |
|---|---|---|---|---|
| **AL-APIKEY-01** | `api_key.ip_blocked` events for a single `tenant_id` + `key_id` exceed 10 in 10 minutes | **P1** | Possible credential stuffing or misconfigured integration. Check if IP enforcement was just enabled (recent `api_key.ip_enforcement_enabled` event for same key). If new enforcement: notify CSM to contact tenant IT admin. If enforcement was not recently changed: treat as credential compromise attempt — notify tenant admin, recommend key rotation. | devops-lead + customer-success |
| **AL-APIKEY-02** | `api_key.rotated` event with no corresponding `api_key.revoked` for `old_key_id` within 26 hours | **P1** | Overlap window expired but old key was not revoked by the pg_cron job. Possible cron failure or schema error. Investigate `api_key_rotation_overlap_cleanup` pg_cron job logs; manually set `revoked_at` if safe; emit `api_key.revoked` DEC-030 event with `reason = 'expiry_overlap'`. | platform-engineer |
| **AL-APIKEY-03** | `api_key.revoked` with `reason = 'compromise'` | **P1** | Credential compromise declared. Trigger `docs/INCIDENT_RESPONSE.md §R-16` (Application Secret & Encryption Key Exposure) for API key subtype. IC opens incident channel. Check `last_used_at` and CloudFlare access logs for the 7-day window before revocation to scope potential abuse. | security-engineer |
| **AL-SCIM-IP-01** | `scim.ip_blocked` events for a single `tenant_id` exceed 5 in 15 minutes AND SCIM provisioning success rate for same tenant drops below 50% in same window | **P2** | SCIM IP enforcement is blocking IdP provisioning — likely a misconfigured allowlist or an IdP IP range change. Notify CSM to contact tenant IT admin. Temporary mitigation: tenant admin can disable SCIM IP enforcement from admin dashboard. Permanent fix: update SCIM allowlist with correct IdP CIDRs. | customer-success |

---

### 26.8 SOC 2 Evidence Mapping

| Criterion | Description | How §26 Controls Satisfy It | Evidence artefact |
|---|---|---|---|
| **CC6.1** | Logical access security controls limit access | API key IP allowlist (`ip_enforcement_enabled + ip_allowlist_config`) adds a network-layer gate to long-lived credential access, preventing use of a compromised API key from non-authorised networks. SCIM IP allowlist (§26.3) provides the equivalent control for provisioning credentials. | **APIKEY-E-001:** 30-day export of `api_key.ip_blocked` events showing enforcement is active; cross-reference with `tenant_api_keys` table showing `ip_enforcement_enabled` and `ip_allowlist_config` for keys in `admin:write` scope |
| **CC6.2** | Authentication requires appropriate credentials | API key scopes enforce least-privilege access at the credential level — `reporting:read` keys cannot call admin mutation endpoints. Scope is validated in `api-key-auth.ts` before route handler execution. | **APIKEY-E-002:** Code review record of `api-key-auth.ts` scope enforcement (PR review + CI gate). Optionally: integration test output showing `reporting:read` key rejected on `POST /v1/admin/seats` |
| **CC6.4** | Access is removed or modified when credentials change | API key rotation (§26.5) with 24-hour overlap and DEC-030 chain linkage between `api_key.rotated` and `api_key.revoked` provides auditable credential lifecycle. Immediate revocation path available for compromise response. | **APIKEY-E-003:** DEC-030 event sequence log for any API key rotation events during the observation period — must show `api_key.rotated` → `api_key.revoked` (old key) sequence with HMAC continuity |
| **CC6.8** | Controls protect against unauthorised changes to configuration | `api_key.ip_enforcement_enabled` and `api_key.ip_enforcement_disabled` DEC-030 events create a non-repudiable audit trail for every change to the IP enforcement configuration on API keys. | **APIKEY-E-004:** DEC-030 `api_key.ip_enforcement_*` event chain for the observation period; cross-reference with admin dashboard role of `enabled_by` actor |
| **CC7.2** | The entity monitors system components for anomalies | AL-APIKEY-01 detects credential stuffing attempts via IP blocks. AL-APIKEY-02 detects rotation overlap failures (infrastructure anomaly). Both alert within PagerDuty SLA. | **APIKEY-E-005:** PagerDuty alert history showing AL-APIKEY-01/02/03 configuration and any triggered incidents during the observation period |

---

### 26.9 Open Questions Resolved

| ID | Prior status | Resolution | Closed in |
|---|---|---|---|
| **OQ-SSO-25.1** | 🟡 P1 — SCIM IP scope ambiguity | 🟢 Resolved — separate `scim_ip_enforcement_enabled` flag (default false) with dedicated `scim_ip_allowlist_config` JSONB. General IP allowlist never applies to SCIM routes. SCIM IP enforcement is an explicit opt-in by the tenant. | §26.3 |
| **OQ-SSO-25.3** | 🟡 P1 M5 — API key IP enforcement missing | 🟢 Resolved — `ip_enforcement_enabled` + `ip_allowlist_config` on `tenant_api_keys`; `enforceApiKeyIpAllowlist()` wired into `api-key-auth.ts`; `api_key.ip_blocked` DEC-030 event defined. | §26.4 |

---

### 26.10 Open Questions

| ID | Question | Priority | Owner | Target |
|---|---|---|---|---|
| **OQ-SSO-26.1** | **Should IP enforcement be mandatory for `admin:write` scope API keys?** The §26.5 rotation policy requires quarterly rotation of `admin:write` keys. Making IP enforcement also mandatory for `admin:write` keys would eliminate the scenario where a compromised high-privilege API key is used freely from any IP. Risk: some customers use `admin:write` keys in CI/CD pipelines with dynamic IP allocation (GitHub Actions, Buildkite cloud runners), and enforcing a static allowlist would break those integrations. Resolution options: (a) mandatory for `admin:write` on Enterprise plan only; (b) block `admin:write` key creation without IP enforcement unless tenant owner confirms waiver; (c) no mandatory enforcement, but emit a STANDARD-severity DEC-030 advisory event if an `admin:write` key has `ip_enforcement_enabled = false` for >30 days. | P2 | security-engineer + enterprise-architect | Before first `admin:write` key issued to a customer |
| **OQ-SSO-26.2** | **🟢 Resolved — DEC-035 (2026-06-01).** Soft enforcement selected: age-alert UI warnings (amber ≥ 365d, red ≥ 730d) + CSM escalation. No automatic revocation at `expires_at` timestamp. Provisional: if any key exceeds 365d without rotation at the first SOC 2 observation period start, the soft control is demonstrably insufficient and hard enforcement ships before the observation period ends. Rationale: enterprise CI/CD pipelines on dynamic-IP runners (GitHub Actions, Buildkite cloud) cannot use static allowlists; hard revocation would cause silent production outages for early pilots. See `docs/DECISION_LOG.md` DEC-035 for full rationale and reverse-cost assessment. | 🟢 **Resolved** | enterprise-architect + compliance-officer | **Done — DEC-035** |

---

### 26.11 Implementation Checklist

#### P0 — Must complete before first enterprise pilot goes live (M5)

| # | Task | Owner | Priority | Milestone |
|---|---|---|---|---|
| 1 | Run migration `0051_tenant_api_keys.sql`: create `tenant_api_keys` table with `ip_enforcement_enabled`, `ip_allowlist_config`, `scopes`, `key_hash`, `key_preview`, `expires_at`, `revoked_at`, `last_used_at`; create partial unique index on `key_hash WHERE revoked_at IS NULL`; add RLS policy for `form_api` role. | platform-engineer | **P0** | M5 |
| 2 | Run migration `0052_scim_ip_allowlist.sql`: add `scim_ip_enforcement_enabled BOOLEAN DEFAULT false` and `scim_ip_allowlist_config JSONB DEFAULT NULL` to `tenant_sso_configs`; add `chk_scim_ip_allowlist_required` CHECK constraint. | platform-engineer | **P0** | M5 |
| 3 | Update `src/workers/auth/api-key-auth.ts`: add `enforceApiKeyIpAllowlist()` call after key hash validation; add scope enforcement before route handler; update `last_used_at` via `waitUntil`. Add `API_KEY_HASH_SECRET` Cloudflare Workers Secret. | platform-engineer | **P0** | M5 |
| 4 | Update `src/workers/auth-policy-middleware/ip-allowlist.ts`: add SCIM route branch; add `enforceScimIpAllowlist()` function using `scim_ip_enforcement_enabled` and `scim_ip_allowlist_config` from `CachedAuthPolicy`. Update `CachedAuthPolicy` type to include new fields. Update `SSO_KV:auth_policy:{tenant_id}` cache shape to include `scim_ip_enforcement_enabled` and `scim_ip_allowlist_config`. | platform-engineer | **P0** | M5 |
| 5 | Register all nine DEC-030 events from §26.6 (`api_key.created`, `api_key.rotated`, `api_key.revoked`, `api_key.ip_enforcement_enabled`, `api_key.ip_enforcement_disabled`, `api_key.ip_blocked`, `scim.ip_enforcement_enabled`, `scim.ip_enforcement_disabled`, `scim.ip_blocked`) in `docs/AUDIT_LOG_SCHEMA.md` event registry. Validate HMAC chain linkage between `api_key.rotated` and `api_key.revoked` in staging. | platform-engineer + compliance-officer | **P0** | M5 |
| 6 | Implement API Keys panel in admin dashboard (§26.5.3): table with label, key_preview, scopes, created, last_used_at, age warning pill, IP enforcement toggle. Add "Create key", "Rotate", and "Revoke" flows. Add "SCIM IP Restriction" section in SSO configuration panel (§26.3.4). Both panels gated by `security.ip_allowlist_enabled` feature flag. | platform-engineer + design-craft | **P0** | M5 |
| 7 | ~~Add `API_KEY_HASH_SECRET` to annual key rotation schedule in `docs/CRYPTOGRAPHY_POLICY.md` §3 key inventory and `docs/SOC2_READINESS.md §56` key inventory table. Initial rotation schedule: 180 days.~~ **[x] Done — 2026-06-11:** `API_KEY_HASH_SECRET` added to `CRYPTOGRAPHY_POLICY.md` §5 Key Rotation Schedule (180 days; re-hash `tenant_api_keys` on rotation; immediate on compromise) and `SOC2_READINESS.md` §56.3 Key Inventory. | security-engineer | ~~**P0**~~ **🟢 Closed** | Done |
| 8 | Implement pg_cron job `api_key_rotation_overlap_cleanup`: runs hourly; sets `revoked_at = now()` for any `tenant_api_keys` row where a rotation overlap has expired (`created_at` of successor key > 24h ago AND predecessor key `revoked_at IS NULL`); emits `api_key.revoked` DEC-030 event with `reason = 'expiry_overlap'`. | platform-engineer | **P0** | M5 |

#### P1 — Before first enterprise customer with API keys in production

| # | Task | Owner | Priority | Milestone |
|---|---|---|---|---|
| 9 | Configure AL-APIKEY-01, AL-APIKEY-02, AL-APIKEY-03, AL-SCIM-IP-01 in PagerDuty. Test AL-APIKEY-01 by triggering synthetic ip_blocked events in staging (enable IP enforcement on a test key; make requests from non-allowlisted IP). | devops-lead | **P1** | M5 |
| 10 | Collect APIKEY-E-001 through APIKEY-E-005 evidence artefacts (§26.8) after 30 days of production API key usage; store in `compliance/evidence/api-key-auth/`; cross-reference in `docs/SOC2_READINESS.md` CC6.1 and CC6.4 evidence tables. | compliance-officer | **P1** | M6 |
| 11 | ~~Resolve OQ-SSO-26.2 (`expires_at` hard enforcement): decision required before SOC 2 observation period start. Document resolution in `docs/DECISION_LOG.md`.~~ **[x] Done — DEC-035 (2026-06-01):** soft enforcement (age alerts + CSM escalation) selected; `docs/DECISION_LOG.md` DEC-035 is the authoritative record; OQ-SSO-26.2 updated to 🟢 Resolved in §26.10. | enterprise-architect + compliance-officer | ~~**P1**~~ **🟢 Closed** | Done |
| 12 | ~~Add `tenant_api_keys` table to `docs/DATA_MODEL.md §24` (Subscription, Billing & Revenue Schema) or as a new §26 entry. The table is defined here as the canonical source of truth; cross-reference in DATA_MODEL.~~ **[x] Done — 2026-06-11:** `docs/DATA_MODEL.md §26` "API Key Authentication Schema" added; canonical DDL cross-referenced from `SSO_SCIM_IMPLEMENTATION.md §26`; scope note confirms SSO_SCIM §26 as canonical implementation reference; TOC updated to include §26. | enterprise-architect | ~~**P1**~~ **🟢 Closed** | Done |

#### P2 — Post-GA improvements

| # | Task | Owner | Priority | Milestone |
|---|---|---|---|---|
| 13 | Resolve OQ-SSO-26.1 (mandatory IP enforcement for `admin:write` scope keys): evaluate after first customer issues an `admin:write` key. | security-engineer + enterprise-architect | **P2** | Before first `admin:write` key issued |
| 14 | Add `admin:write` key age warning (amber at >90 days) to the dashboard API Keys panel, distinct from the 365-day general threshold. | platform-engineer | **P2** | M6 |

---

---

## 27. SCIM v2.0 Endpoint — Worker Implementation Design (Closes G-001)

### 27.1 Purpose and Gap Closure

**Gap closed:** G-001 (§9) — *"SCIM endpoint not yet implemented. Schema is defined; endpoint code, token authentication, and attribute mapping are not written."*

This section provides the complete Cloudflare Worker implementation design for the SCIM v2.0 endpoint: bearer-token authentication, request routing, User and Group CRUD handlers, idempotency contract, RFC 7644 error format, and the six DEC-030 lifecycle events that the R-24 runbook and OBSERVABILITY.md §26.7a already reference by name. No component in this section introduces new architectural dependencies — it connects schemas (§3–§4), role mapping (§5), session revocation (§22), and IP enforcement (§25–§26) into a runnable endpoint.

**DPA gate (G-013):** SCIM provisioning must not be enabled for any enterprise customer until the DPA template update in §14 is signed. This implementation may be built and tested in staging ahead of G-013 closure, but the `SCIM_ENABLED` feature flag must remain off in production until the DPA block is cleared.

**Privacy floor:** No employee name, email, or health data appears in any DEC-030 SCIM event payload. Events carry `tenant_id` + aggregate counts + FORM-internal UUIDs only. User identity is linked via `external_user_id` (IdP-side stable ID) stored in Supabase, not surfaced in the audit chain.

---

### 27.2 Worker Architecture and File Layout

The SCIM endpoint runs as a dedicated Cloudflare Worker separate from the main `api-gateway` Worker to allow independent rate-limit quotas, IP enforcement routing, and deployment cadence.

```
apps/scim-worker/
├── src/
│   ├── index.ts               # Entry point — router dispatch
│   ├── auth.ts                # Token authentication pipeline (§27.3)
│   ├── router.ts              # URL pattern → handler dispatch (§27.4)
│   ├── handlers/
│   │   ├── users.ts           # User CRUD (§27.5)
│   │   ├── groups.ts          # Group sync (§27.6)
│   │   └── discovery.ts       # ServiceProviderConfig, Schemas, ResourceTypes (§27.7)
│   ├── errors.ts              # RFC 7644 error helpers (§27.8)
│   ├── idempotency.ts         # Upsert + dedup logic (§27.9)
│   ├── events.ts              # DEC-030 emission (§27.10)
│   └── types.ts               # Shared TypeScript interfaces
└── wrangler.toml
```

**Cloudflare bindings required:**

| Binding | Type | Purpose |
|---|---|---|
| `SCIM_KV` | KV Namespace | Token → tenant metadata cache; separate from `SSO_KV` for quota isolation |
| `SUPABASE_URL` | Secret | Postgres REST endpoint |
| `SUPABASE_SERVICE_ROLE_KEY` | Secret | Service role for SCIM writes (RLS bypassed; set `app.current_tenant_id` manually) |
| `SCIM_TOKEN_HASH_SECRET` | Secret | HMAC-SHA256 key for hashing bearer tokens before KV lookup and Supabase lookup |
| `AUDIT_QUEUE` | Queue | Cloudflare Queue for DEC-030 event emission (shared with other Workers) |
| `IP_HASH_SALT` | Secret | Shared with §25/§26 — daily-rotating salt for `client_ip_hash` in blocked events |

`SCIM_TOKEN_HASH_SECRET` must be registered in `docs/CRYPTOGRAPHY_POLICY.md §5` Key Rotation Schedule (180-day rotation; re-hash all active `tenant_scim_tokens.token_hash` rows on rotation). Add to `docs/SOC2_READINESS.md` key inventory.

---

### 27.3 Token Authentication Pipeline

Every SCIM request is authenticated via HMAC-SHA256 of the bearer token against `tenant_scim_tokens`. KV caches the token metadata to avoid a Supabase round-trip on every request.

**Token KV key format:** `scim_token:{token_hash_hex}` → JSON:

```typescript
interface ScimTokenCache {
  tenant_id: string;               // UUID
  label: string;                   // human-readable label from admin dashboard
  is_active: boolean;
  scim_ip_enforcement_enabled: boolean;
  scim_ip_allowlist_config: {
    cidrs: string[];
    mode: 'allowlist' | 'denylist';
  } | null;
}
```

**KV TTL:** 5 minutes. This ensures that token rotation (24-hour overlap window per §3.1) propagates to the Worker within one provisioning cycle. A revoked token is invalidated within 5 minutes — acceptable for SCIM directory sync traffic where requests are batched, not real-time.

**Authentication sequence (`auth.ts`):**

```typescript
export async function authenticateScimRequest(
  request: Request,
  env: Env,
): Promise<{ tenantMeta: ScimTokenCache } | ScimError> {
  const authHeader = request.headers.get('Authorization');
  if (!authHeader?.startsWith('Bearer ')) {
    return scimError(401, 'invalidValue', 'Missing or malformed Authorization header.');
  }
  const rawToken = authHeader.slice(7);

  // Hash the raw token — never store or log plaintext
  const tokenHash = await hmacSha256Hex(rawToken, env.SCIM_TOKEN_HASH_SECRET);

  // 1. KV cache hit (fast path)
  const cached = await env.SCIM_KV.get<ScimTokenCache>(
    `scim_token:${tokenHash}`, { type: 'json' }
  );
  if (cached) {
    if (!cached.is_active) return scimError(401, 'invalidValue', 'Token revoked.');
    return { tenantMeta: cached };
  }

  // 2. KV miss — fall back to Supabase (cold path: first request after token creation or cache expiry)
  const row = await supabaseQuery<ScimTokenCache>(
    env,
    `SELECT
       tst.tenant_id,
       tst.label,
       tst.is_active,
       tsc.scim_ip_enforcement_enabled,
       tsc.scim_ip_allowlist_config
     FROM tenant_scim_tokens tst
     JOIN tenant_sso_configs tsc ON tsc.tenant_id = tst.tenant_id
     WHERE tst.token_hash = $1
       AND (tst.expires_at IS NULL OR tst.expires_at > NOW())`,
    [tokenHash],
  );
  if (!row || !row.is_active) return scimError(401, 'invalidValue', 'Invalid or revoked token.');

  // Populate KV for future requests
  await env.SCIM_KV.put(
    `scim_token:${tokenHash}`,
    JSON.stringify(row),
    { expirationTtl: 300 },  // 5 minutes
  );
  return { tenantMeta: row };
}
```

**IP enforcement:** After token auth succeeds, the request is passed to `enforceScimIpAllowlist()` from `apps/api-gateway/src/sso/ip-allowlist.ts` (shared utility, §25.5). If the IP check fails, a `scim.ip_blocked` DEC-030 event is emitted (HIGH, §26.6) and HTTP 403 is returned — consistent with §26.3 SCIM IP scope design.

---

### 27.4 Request Routing

The router dispatches on HTTP method and URL path. All SCIM routes are under `/scim/v2/`.

```typescript
// router.ts
export async function routeScimRequest(
  request: Request,
  tenantMeta: ScimTokenCache,
  env: Env,
  ctx: ExecutionContext,
): Promise<Response> {
  const { method, pathname } = new URL(request.url);

  // Schema discovery (unauthenticated in some IdP implementations — FORM requires auth)
  if (method === 'GET' && pathname === '/scim/v2/ServiceProviderConfig')
    return handleServiceProviderConfig();
  if (method === 'GET' && pathname === '/scim/v2/Schemas')
    return handleSchemas();
  if (method === 'GET' && pathname === '/scim/v2/ResourceTypes')
    return handleResourceTypes();

  // Users
  const userListMatch = pathname === '/scim/v2/Users';
  const userItemMatch = pathname.match(/^\/scim\/v2\/Users\/([^/]+)$/);
  if (userListMatch && method === 'GET')  return handleListUsers(request, tenantMeta, env);
  if (userListMatch && method === 'POST') return handleCreateUser(request, tenantMeta, env, ctx);
  if (userItemMatch && method === 'GET')  return handleGetUser(userItemMatch[1], tenantMeta, env);
  if (userItemMatch && method === 'PUT')  return handleReplaceUser(request, userItemMatch[1], tenantMeta, env, ctx);
  if (userItemMatch && method === 'PATCH') return handlePatchUser(request, userItemMatch[1], tenantMeta, env, ctx);
  if (userItemMatch && method === 'DELETE') return handleDeleteUser(userItemMatch[1], tenantMeta, env, ctx);

  // Groups
  const groupListMatch = pathname === '/scim/v2/Groups';
  const groupItemMatch = pathname.match(/^\/scim\/v2\/Groups\/([^/]+)$/);
  if (groupListMatch && method === 'GET')  return handleListGroups(request, tenantMeta, env);
  if (groupListMatch && method === 'POST') return handleCreateGroup(request, tenantMeta, env, ctx);
  if (groupItemMatch && method === 'GET')  return handleGetGroup(groupItemMatch[1], tenantMeta, env);
  if (groupItemMatch && method === 'PATCH') return handlePatchGroup(request, groupItemMatch[1], tenantMeta, env, ctx);
  if (groupItemMatch && method === 'DELETE') return handleDeleteGroup(groupItemMatch[1], tenantMeta, env, ctx);

  return scimError(404, 'noTarget', `Path not found: ${pathname}`);
}
```

All handlers share the `tenantMeta` object (which carries `tenant_id`) and set `app.current_tenant_id` via the `X-Supabase-Tenant-ID` header in every Supabase request (the Supabase middleware reads this and calls `SET LOCAL app.current_tenant_id = $1` before executing).

---

### 27.5 User Handlers

#### 27.5.1 POST /scim/v2/Users — Create or Reactivate

**Idempotency gate:** Before creating a user, look up by `externalId` (§27.9). Three outcomes:

| Lookup result | Action | HTTP response | DEC-030 event |
|---|---|---|---|
| Not found | INSERT new user | `201 Created` | `scim.user_provisioned` |
| Found, `is_active = true` | Return existing user (no write) | `200 OK` | None (idempotent replay) |
| Found, `is_active = false` | UPDATE `is_active = true`, restore seat | `200 OK` | `scim.user_reactivated` |

**Sensitive attribute guard:** Before any write, the request body is scanned for GDPR Art. 9 special category attributes (§14.5). If detected, the request is rejected `HTTP 400` with `scimType: "invalidValue"` and a `scim.rejected_sensitive_attribute` DEC-030 event is emitted (HIGH, 7yr). No user record is written. Prohibited attribute names: `healthStatus`, `disability`, `religion`, `ethnicity`, `unionMembership`, `politicalOpinion`, and any `urn:ietf:params:scim:schemas:extension:*` attribute whose key contains these strings (case-insensitive).

**Seat limit check:** Before INSERT, verify `(SELECT COUNT(*) FROM tenant_users WHERE tenant_id = $1 AND is_active = true) < (SELECT contracted_seats FROM tenants WHERE id = $1)`. If at cap, return `HTTP 409 Conflict` with `scimType: "uniqueness"` and detail: `"Seat limit reached. Contact your FORM account manager to expand contracted seats."` No DEC-030 event for this condition (operational, not security-relevant; CSM dashboard tracks seat utilization separately).

**Success response (201):**

```json
{
  "schemas": ["urn:ietf:params:scim:schemas:core:2.0:User"],
  "id": "<form-uuid>",
  "externalId": "<idp-stable-id>",
  "userName": "dmytro.kovalenko@acme.com",
  "name": { "givenName": "Dmytro", "familyName": "Kovalenko" },
  "emails": [{ "value": "dmytro.kovalenko@acme.com", "primary": true }],
  "active": true,
  "meta": {
    "resourceType": "User",
    "created": "2026-06-13T10:00:00Z",
    "lastModified": "2026-06-13T10:00:00Z",
    "location": "https://form.coach/scim/v2/Users/<form-uuid>",
    "version": "W/\"<etag>\""
  }
}
```

The `id` field is always the FORM-generated UUID. The `externalId` is stored in `tenant_users.external_user_id` and is the IdP-side stable identifier used for deduplication.

#### 27.5.2 GET /scim/v2/Users/:id

Fetches the user by FORM UUID. Returns `HTTP 404` with `scimType: "noTarget"` if `id` is not found within the tenant (RLS enforces tenant isolation — a valid UUID from another tenant is treated as not found, same response as a nonexistent UUID, to prevent cross-tenant enumeration).

No DEC-030 event for reads (reads are not security-relevant at the individual level; aggregate access patterns are covered by SOC 2 audit log monitoring in `docs/OBSERVABILITY.md §4`).

#### 27.5.3 GET /scim/v2/Users?filter=...

Supports `filter` parameter with `eq` operator on `userName`, `emails.value`, `externalId`, and `active`. Additional operators (`co`, `sw`, `pr`) return `HTTP 400 scimType:invalidFilter`.

Pagination via `startIndex` (1-based) and `count` (default 100, max 200). Response format:

```json
{
  "schemas": ["urn:ietf:params:scim:api:messages:2.0:ListResponse"],
  "totalResults": 142,
  "startIndex": 1,
  "itemsPerPage": 100,
  "Resources": [ ...User objects... ]
}
```

IdPs use this endpoint during the initial sync to check for existing users before issuing creates. The filter implementation must be constant-time on tenant scope (RLS handles isolation; no timing leakage possible between tenants).

#### 27.5.4 PUT /scim/v2/Users/:id — Full Replace

Full replacement of all mutable attributes. Immutable fields (`id`, `tenant_id`, `created_at`) are ignored even if present in the payload. Sensitive attribute guard runs on the full request body before any write.

If `active` transitions from `true → false` in a PUT, the same session revocation path as PATCH deactivation is triggered (§27.5.5).

DEC-030 event: `scim.user_updated` (STANDARD, 7yr) — tracks that a full replace occurred; does not diff fields.

#### 27.5.5 PATCH /scim/v2/Users/:id — Partial Update

Supports RFC 7644 PatchOp: `add`, `replace`, `remove` operations on individual attributes. The Worker parses the `Operations` array and applies each operation sequentially to a staging object before committing to Supabase as a single UPDATE.

**Deactivation path (active: false):**

When `active` is set to `false` via PATCH (or the SCIM DELETE handler, which is aliased to this path internally):

1. `UPDATE tenant_users SET is_active = false, deactivated_at = NOW(), deactivated_via = 'scim' WHERE id = $1 AND tenant_id = $2`
2. Seat count decremented: `UPDATE tenants SET active_seat_count = active_seat_count - 1 WHERE id = $1`
3. Session revocation: call `nukeTenantUserSessions(tenant_id, user_id, env)` from §22 KV revocation layer — writes `revoke:user:{tenant_id}:{user_id}` KV key (TTL 24h); all active JWTs for this user are invalidated within the next request cycle (< 3 ms per §22 architecture). If KV write fails, fall back to Supabase `session_blocklist` INSERT (§22 fallback path) and emit `session.revocation_kv_sync_error` per AL-REVOKE-01.
4. Emit `scim.user_deprovisioned` DEC-030 (HIGH, 7yr).
5. Return `HTTP 200` with updated user representation (`active: false`).

**Standardized event name note:** §3.5 uses the interim name `scim.user_deactivated`. The canonical name, used by R-24 mass deprovisioning detection and formally defined in §27.10, is `scim.user_deprovisioned`. AUDIT_LOG_SCHEMA.md must register the canonical name; `scim.user_deactivated` is deprecated and must not be used in new code.

#### 27.5.6 DELETE /scim/v2/Users/:id

SCIM DELETE is treated identically to PATCH `{ "op": "replace", "path": "active", "value": false }`. Data is not deleted from Supabase — soft-delete only. Hard deletion requires a separate DSAR Art. 17 erasure request (§31/§32 DATA_MODEL). This is intentional and must be disclosed in the DPA per §14.

Response: `HTTP 204 No Content`. DEC-030 event: `scim.user_deprovisioned` (same as PATCH deactivation — the emitter distinguishes via `deprovision_method: 'scim_delete' | 'scim_patch_inactive'` in the payload).

---

### 27.6 Group Handlers

SCIM Groups are used exclusively for role mapping (§5, §19). FORM does not maintain a native group membership store outside the SCIM sync — `scim_group_role_mappings` maps IdP group IDs to FORM roles, and users are assigned roles based on group membership received in SCIM requests.

#### 27.6.1 POST /scim/v2/Groups — Create Mapping

Creates a new entry in `scim_group_role_mappings`. The FORM role defaults to `member` — the admin must configure the actual role mapping in the dashboard after creation (§16.8). Response: `HTTP 201` with group representation.

Idempotency: if a group with the same `externalId` / `displayName` already exists for this tenant, return `HTTP 200` with the existing record.

DEC-030 event: none for group creation (low-risk config change; covered by admin dashboard audit log).

#### 27.6.2 PATCH /scim/v2/Groups/:id — Member Add/Remove

This is the primary role-assignment trigger. The IdP sends PATCH operations with `members` array additions and removals.

**Processing sequence:**

1. Parse `Operations` array — extract member UUIDs to add/remove (IdP sends FORM UUIDs from prior `POST /Users` responses, or `externalId` as `value` depending on IdP — see §27.6.3).
2. For each added member: look up `scim_group_role_mappings` to resolve FORM role; UPDATE `tenant_users.form_role` where the new role is higher privilege than current (higher-privilege-wins, per DEC-039 — §19.9 OQ-SSO-19.1 resolution).
3. For each removed member: re-evaluate role from remaining group memberships (MAX aggregation per §19.3 `group_member_effective_role` view); UPDATE `tenant_users.form_role` to the highest remaining role.
4. Emit `scim.group_synced` DEC-030 (STANDARD, 5yr) once per PATCH operation — aggregate counts only, no individual `user_id` in payload.
5. Return `HTTP 200` with updated group representation.

**Reactivation side-effect:** If a member-add operation targets a `user_id` with `is_active = false`, the user is reactivated automatically (same path as §27.5.1 reactivation) and `scim.user_reactivated` is emitted alongside `scim.group_synced`.

#### 27.6.3 IdP Value Field Semantics

Different IdPs send different values in `members[].value`:

| IdP | `members[].value` contains | Resolution in FORM |
|---|---|---|
| Okta | FORM SCIM `id` (UUID from prior `/Users` response) | Direct lookup by `tenant_users.id` |
| Azure AD | FORM SCIM `id` (UUID from prior `/Users` response) | Direct lookup by `tenant_users.id` |
| Google Workspace (via SCIM bridge) | `externalId` (Google directory user key) | Lookup by `tenant_users.external_user_id` |
| OneLogin | FORM SCIM `id` | Direct lookup by `tenant_users.id` |

The PATCH Groups handler must tolerate both `id` and `externalId` as the value — attempt `id` lookup first; if no match, attempt `external_user_id` lookup. If neither matches, skip the member (log a STANDARD-severity warning to structured logs; do not fail the request — partial membership lists from IdPs are normal during large directory syncs).

#### 27.6.4 DELETE /scim/v2/Groups/:id

Removes the `scim_group_role_mappings` entry. All users who received their highest role from this group have their role downgraded to the highest remaining group-assigned role (or `member` default if no other group mapping exists). Role changes trigger a re-evaluation of each affected user's active sessions — no session invalidation, but the next JWT refresh will reflect the new role (< 1 hour for standard session TTL).

Response: `HTTP 204 No Content`.

---

### 27.7 Schema Discovery Endpoints

SCIM clients call these endpoints during initial configuration to learn FORM's capabilities. All three are read-only and require SCIM bearer auth (FORM enforces auth on discovery to prevent capability enumeration by unauthenticated scanners).

#### 27.7.1 GET /scim/v2/ServiceProviderConfig

```json
{
  "schemas": ["urn:ietf:params:scim:schemas:core:2.0:ServiceProviderConfig"],
  "documentationUri": "https://form.coach/docs/enterprise/scim",
  "patch": { "supported": true },
  "bulk": { "supported": false, "maxOperations": 0, "maxPayloadSize": 0 },
  "filter": { "supported": true, "maxResults": 200 },
  "changePassword": { "supported": false },
  "sort": { "supported": false },
  "etag": { "supported": true },
  "authenticationSchemes": [{
    "type": "oauthbearertoken",
    "name": "OAuth Bearer Token",
    "description": "SCIM bearer token issued per tenant via FORM Admin Dashboard → Directory Sync."
  }]
}
```

**`bulk: false`**: FORM does not implement SCIM bulk operations. IdP bulk syncs are handled via individual CRUD calls. Bulk is a v2-optional feature; the production IdPs (Okta, Azure AD, Google Workspace) operate correctly without it.

**`changePassword: false`**: Passwords are not managed via SCIM — users authenticate via SSO. Password management is entirely the IdP's responsibility.

#### 27.7.2 GET /scim/v2/Schemas

Returns the `urn:ietf:params:scim:schemas:core:2.0:User` and `urn:ietf:params:scim:schemas:core:2.0:Group` schema definitions plus the Enterprise User extension schema `urn:ietf:params:scim:schemas:extension:enterprise:2.0:User`. Only attributes that FORM actually reads (§3.3 attribute mapping table) are declared — attributes FORM ignores are not listed, to prevent IdPs from erroneously assuming FORM stores them.

#### 27.7.3 GET /scim/v2/ResourceTypes

Returns `User` and `Group` resource type definitions with their supported schema extensions.

---

### 27.8 Error Response Format (RFC 7644)

All SCIM errors use the RFC 7644 error schema. HTTP status codes are the primary classification signal; `scimType` is provided for machine-readable sub-classification.

```typescript
// errors.ts
export function scimError(
  status: 400 | 401 | 403 | 404 | 409 | 429 | 500 | 503,
  scimType:
    | 'invalidFilter'   // 400 — unsupported filter operator
    | 'tooMany'         // 400 — filter would return too many results
    | 'uniqueness'      // 409 — unique constraint violation (email, seat cap)
    | 'mutability'      // 400 — attempt to modify immutable attribute (id, tenant_id)
    | 'invalidSyntax'   // 400 — malformed JSON or missing required field
    | 'invalidPath'     // 400 — PatchOp path references non-existent attribute
    | 'noTarget'        // 404 — resource not found, or path not found
    | 'invalidValue',   // 400/401 — invalid attribute value, including auth failure
  detail: string,
): Response {
  return new Response(JSON.stringify({
    schemas: ['urn:ietf:params:scim:api:messages:2.0:Error'],
    status: String(status),
    scimType,
    detail,
  }), {
    status,
    headers: { 'Content-Type': 'application/scim+json' },
  });
}
```

**Never expose tenant internals in error details.** `detail` strings are pre-approved literals — no database error messages, no stack traces, no internal IDs. The only exception is `uniqueness` conflict on email (acceptable to echo the email for UX — the IdP already knows the email it sent).

**Rate limit errors (429):** Return `Retry-After: <seconds>` header alongside the SCIM error body. The Worker derives `<seconds>` from the KV TTL remaining on the rate limit counter key.

---

### 27.9 Idempotency Contract

SCIM clients retry on transient errors (network timeout, 5xx). FORM must tolerate retried requests without creating duplicate users or emitting duplicate DEC-030 events.

**User creation idempotency (POST /Users):**

The primary idempotency key is `(tenant_id, external_user_id)` — the unique index `UNIQUE (tenant_id, external_user_id)` on `tenant_users` enforces this at the database level. If the SCIM Worker receives a POST with an `externalId` that already exists in the tenant, the existing user is returned (200) rather than attempting an INSERT that would fail with a uniqueness violation.

A secondary dedup path uses `(tenant_id, email)` — if `externalId` is absent (some legacy IdP integrations omit it), the Worker falls back to email lookup. If email lookup finds an existing user, the response is the same: 200 with the existing user, with `externalId` populated from the found record.

**Group PATCH idempotency:**

Group PATCH operations are inherently idempotent — adding an already-present member is a no-op at the role level (higher-privilege-wins; same role does not change). Removing an absent member is silently skipped (§27.6.2).

**Event deduplication:**

`scim.user_provisioned` and `scim.user_reactivated` carry a `scim_request_id` (UUID generated by the Worker per inbound request). The `emit-audit-event` Worker's `scim_request_id` uniqueness check (same pattern as R-24's `scim_request_id` dedup) prevents the same physical request from emitting duplicate chain events if the Worker retries internally.

---

### 27.10 DEC-030 HMAC-Chained Audit Events

Six SCIM lifecycle events are formally defined here. These are the canonical definitions; AUDIT_LOG_SCHEMA.md §SCIM-Lifecycle must register these before the SCIM endpoint goes to production.

**Event naming note:** §3.5 uses the interim name `scim.user_deactivated`. The canonical name is `scim.user_deprovisioned` — consistent with the R-24 runbook reference and the INCIDENT_RESPONSE.md mass deprovisioning detection clause. Any existing staging instrumentation using `scim.user_deactivated` must be renamed.

**Privacy invariant (all six events):** No employee name, email address, or health/coaching data in any payload. `external_user_id` is the IdP-side opaque stable identifier — acceptable in the audit chain as it carries no health data. `user_id` (FORM internal UUID) is acceptable in per-user events as it is internal-only and carries no PII by itself. `tenant_id` is always present.

| Event type | Severity | Retention | Trigger | Key payload fields |
|---|---|---|---|---|
| `scim.user_provisioned` | HIGH | 7 yr | Successful `POST /scim/v2/Users` resulting in a new `tenant_users` INSERT | `tenant_id`, `user_id` (FORM UUID), `external_user_id` (IdP stable ID — opaque), `scim_request_id` (UUID, dedup key), `provisioning_source: 'scim'`, `assigned_role` (default `member`) |
| `scim.user_reactivated` | HIGH | 7 yr | `POST /scim/v2/Users` where user existed with `is_active = false`; or `PATCH /Users` group member-add targeting inactive user | `tenant_id`, `user_id`, `external_user_id`, `scim_request_id`, `prior_deactivated_at` (ISO 8601 — duration-since-deactivation supports SOC 2 CC6.3 evidence) |
| `scim.user_deprovisioned` | HIGH | 7 yr | `PATCH /scim/v2/Users/:id` with `active: false`, or `DELETE /scim/v2/Users/:id` | `tenant_id`, `user_id`, `external_user_id`, `scim_request_id`, `deprovision_method` (`scim_patch_inactive` \| `scim_delete`), `sessions_revoked_count` (integer — sessions invalidated by §22 KV revocation; 0 if user had no active sessions) |
| `scim.user_updated` | STANDARD | 7 yr | `PUT` or `PATCH /scim/v2/Users/:id` updating non-activation attributes (name, department, role change via group) | `tenant_id`, `user_id`, `scim_request_id`, `update_method` (`put` \| `patch`), `fields_changed` (string[] — attribute names only; no values — e.g. `["department", "name.givenName"]`) |
| `scim.group_synced` | STANDARD | 5 yr | `PATCH /scim/v2/Groups/:id` processed successfully | `tenant_id`, `group_id` (FORM `scim_group_role_mappings.id` UUID), `idp_group_id` (IdP-side group identifier, opaque string), `members_added` (integer count), `members_removed` (integer count), `role_changes` (integer — users whose FORM role changed as a result), `scim_request_id` |
| `scim.rejected_sensitive_attribute` | HIGH | 7 yr | Request body contains GDPR Art. 9 special category attribute; request rejected before any write | `tenant_id`, `scim_request_id`, `rejected_attribute_name` (the offending field name), `request_path` (`/scim/v2/Users` \| `/scim/v2/Groups`), `http_method` |

**HMAC chain requirement (SCIM-CHAIN-01):** For a given `user_id`, `scim.user_deprovisioned` must not precede `scim.user_provisioned` in the chain (a deprovisioned user with no prior provisioning event indicates a chain gap or bug). The `emit-audit-event` Worker logs a HIGH-severity WARNING (non-blocking) if this ordering invariant is violated — it does not reject the event, as the SCIM endpoint must still process the deprovisioning even if the audit log is incomplete. A chain invariant violation triggers R-05 investigation protocol.

**Retention rationale:** `scim.user_deprovisioned` is 7yr because it is the primary SOC 2 CC6.3 evidence ("logical access removed in a timely manner"). `scim.group_synced` is 5yr (role management rather than identity lifecycle — lower regulatory importance; Supabase storage cost optimisation at scale). All other events are 7yr as direct user identity lifecycle records.

---

### 27.11 Alerting Rules

| Alert ID | Condition | Severity | Routing | Auto-resolve |
|---|---|---|---|---|
| **AL-SCIM-01** | `scim.rejected_sensitive_attribute` events > 3 for a single `tenant_id` in 1 hour | **P1** | PagerDuty `form-platform` + `form-compliance`; Slack `#security-alerts` | No — IC review required (may indicate IdP misconfiguration or data classification policy gap at customer) |
| **AL-SCIM-02** | `POST /scim/v2/Users` 5xx error rate > 5% over a 10-minute rolling window for any tenant | **P2** | PagerDuty `form-platform` | Yes — on error rate returning to < 1% |
| **AL-SCIM-03** | `scim.user_provisioned` count for a tenant drops to 0 for > 24 hours when the tenant has an active SCIM configuration and the previous 7-day average was > 5/day (signals SCIM sync stall) | **P2** | Slack `#enterprise-ops` + CSM on-call | Yes — on next `scim.user_provisioned` emission for the tenant |
| **AL-SCIM-04** | SCIM-CHAIN-01 ordering invariant violated (`scim.user_deprovisioned` with no prior `scim.user_provisioned` for the same `user_id`) | **P1** | PagerDuty `form-platform` + `form-compliance`; R-05 co-activation | No |

**AL-SCIM-01 severity rationale:** A `scim.rejected_sensitive_attribute` event means a customer's IdP attempted to push GDPR Art. 9 health/biometric data into FORM. One occurrence is likely a misconfigured custom SCIM attribute. Three occurrences in an hour indicate a systematic mapping issue. Compliance-officer must notify the customer CSM within 4 hours and verify the IdP schema is corrected before the block is lifted.

---

### 27.12 SOC 2 Evidence Mapping

| Criterion | Event / artefact | Control statement |
|---|---|---|
| **CC6.1** — Logical access to infrastructure | `scim.user_provisioned` + `scim.user_deprovisioned` DEC-030 chain | Every access grant and revocation is HMAC-evidenced and auditor-extractable |
| **CC6.3** — Removes access on a timely basis | `scim.user_deprovisioned.sessions_revoked_count` > 0; timestamp delta from SCIM DELETE receipt to `session.bulk_revocation_complete` ≤ 60 s per `ENTERPRISE_SLA.md §3.7` | Sessions invalidated within SLA window on every deprovisioning event |
| **CC6.4** — Restricts access to authorised users via provisioning controls | `scim.rejected_sensitive_attribute` event count = 0 during observation period | FORM's SCIM layer enforces Art. 9 attribute rejection; zero tolerance for health data ingestion via SCIM |
| **CC6.6** — Role changes are authorised | `scim.group_synced.role_changes` correlated with `scim_group_role_mappings` admin dashboard change log | Role assignments flow from IdP group → FORM group mapping (admin-configured); no direct role write exists outside the group sync path |
| **CC7.2** — Proactive monitoring | AL-SCIM-01 through AL-SCIM-04 PagerDuty/Slack configuration screenshots | Four alerting rules cover sensitive attribute rejection, provisioning error spikes, sync stall detection, and chain integrity |
| **PI1.1** — Processing is complete | `scim.user_provisioned` count equals IdP `POST /Users` success count (reconcilable via IdP provisioning log) | Every IdP provisioning request that FORM accepted is reflected in the DEC-030 chain |

**SOC 2 evidence artefacts:**

| Artefact | Description | Criteria | Retention |
|---|---|---|---|
| **SCIM-PROV-E-001** | Quarterly export of `scim.user_provisioned` DEC-030 chain for observation period; HMAC chain head verification | CC6.1, PI1.1 | 7 yr |
| **SCIM-PROV-E-002** | Quarterly export of `scim.user_deprovisioned` events; for each event, verify `sessions_revoked_count ≥ 0` and delta to `session.bulk_revocation_complete` ≤ 60 s | CC6.3 | 7 yr |
| **SCIM-PROV-E-003** | Assertion that `scim.rejected_sensitive_attribute` count = 0 in observation period; if > 0, incident report for each occurrence | CC6.4 | 7 yr |
| **SCIM-PROV-E-004** | AL-SCIM-01 through AL-SCIM-04 PagerDuty alert configuration screenshots; verify all four rules active before observation period | CC7.2 | 3 yr |

Filing path: `compliance/evidence/scim-provisioning/SCIM-PROV-E-00{1..4}_<YYYY-QN>.pdf`

---

### 27.13 Gap Status Update

| Gap | Prior status | New status | Notes |
|---|---|---|---|
| **G-001** | 🔴 Not designed | 🟡 Design complete, implementation pending | Full Worker architecture, authentication pipeline, CRUD handlers, idempotency, error format, and DEC-030 events specified in this section. Implementation blocked on: (1) `apps/scim-worker/` scaffold creation per §27.14; (2) G-013 DPA closure (cannot enable in production for any customer until DPA updated). |

---

### 27.14 Implementation Checklist

#### P0 — Must complete before SCIM goes live for any enterprise customer (M4/M5)

| # | Task | Owner | Priority | Milestone | Status |
|---|---|---|---|---|---|
| 1 | Register all six DEC-030 events from §27.10 in `docs/AUDIT_LOG_SCHEMA.md §SCIM-Lifecycle` with Zod v2 schemas and retention labels. Deprecate `scim.user_deactivated` (§3.5 interim name) — update all references to `scim.user_deprovisioned`. | compliance-officer + platform-engineer | **P0** | M4 | [ ] |
| 2 | Scaffold `apps/scim-worker/` directory structure per §27.2. Configure `wrangler.toml` with `SCIM_KV` namespace binding (new; separate from `SSO_KV`), `SCIM_TOKEN_HASH_SECRET` Workers Secret, and `AUDIT_QUEUE` binding. | platform-engineer | **P0** | M4 | [ ] |
| 3 | Add `SCIM_TOKEN_HASH_SECRET` to `docs/CRYPTOGRAPHY_POLICY.md §5` Key Rotation Schedule (180-day cycle; re-hash `tenant_scim_tokens.token_hash` on rotation). Add to `docs/SOC2_READINESS.md` key inventory. | security-engineer | **P0** | M4 | [x] ✅ 2026-06-14 — §5 + §6 + §12 updated in CRYPTOGRAPHY_POLICY v1.2; §56.3 row added in SOC2_READINESS |
| 4 | Implement `auth.ts` token authentication pipeline (§27.3): HMAC-SHA256 hash, KV lookup with 5-minute TTL, Supabase fallback, IP enforcement call to shared `enforceScimIpAllowlist()`. | platform-engineer | **P0** | M4 | [ ] |
| 5 | Implement `router.ts` dispatch (§27.4) and `errors.ts` RFC 7644 error helpers (§27.8). | platform-engineer | **P0** | M4 | [ ] |
| 6 | Implement `handlers/users.ts`: POST (create/reactivate with idempotency gate — §27.5.1), GET by id (§27.5.2), GET list with filter (§27.5.3), PUT (§27.5.4), PATCH (§27.5.5 including deactivation → §22 KV revocation), DELETE (§27.5.6). Include sensitive attribute guard (§27.5.1) across all write handlers. | platform-engineer | **P0** | M5 | [ ] |
| 7 | Implement `handlers/groups.ts`: POST (§27.6.1), PATCH with role re-evaluation and `MAX()` aggregation per §19.3 view (§27.6.2), DELETE (§27.6.4). Handle both `id` and `externalId` member value formats per §27.6.3. | platform-engineer | **P0** | M5 | [ ] |
| 8 | Implement `handlers/discovery.ts`: ServiceProviderConfig (§27.7.1), Schemas (§27.7.2), ResourceTypes (§27.7.3). | platform-engineer | **P0** | M4 | [ ] |
| 9 | Configure AL-SCIM-01 through AL-SCIM-04 in PagerDuty / Slack per §27.11. Test AL-SCIM-01 by submitting a synthetic SCIM payload with a prohibited attribute name to staging; verify `scim.rejected_sensitive_attribute` event fires and PagerDuty alert triggers. | devops-lead | **P0** | M5 | [ ] |
| 10 | G-013 DPA gate: before enabling `SCIM_ENABLED` feature flag for any production tenant, confirm compliance-officer has signed updated DPA template per §14. Blocking gate — do not merge feature flag activation without written compliance-officer sign-off. | compliance-officer | **P0** | M5 | [ ] |

#### P1 — Before first enterprise pilot with SCIM active

| # | Task | Owner | Priority | Milestone | Status |
|---|---|---|---|---|---|
| 11 | End-to-end SCIM smoke test per §7.4: configure test tenant with Okta SCIM; provision three users; verify `scim.user_provisioned` events in chain; deactivate one user via Okta; verify session revocation within 60 s; check `scim.user_deprovisioned.sessions_revoked_count` = 1. Emit `enterprise.sso_scim_setup_verified` (AUDIT_LOG_SCHEMA §Enterprise Implementation) after test passes. | platform-engineer | **P1** | M5 | [ ] |
| 12 | Repeat smoke test with Azure AD SCIM connector; verify group sync (`scim.group_synced`) fires with correct `role_changes` count after adding test user to mapped group. | platform-engineer | **P1** | M5 | [ ] |
| 13 | Load test `POST /scim/v2/Users` at 100 req/min burst (§3.6 burst limit) for 5 minutes using k6; verify no 5xx responses and that AL-SCIM-02 does not fire falsely; confirm `sessions_revoked_count` accuracy under concurrent deprovisioning. | devops-lead | **P1** | M5 | [ ] |
| 14 | After 30 days of production SCIM activity: collect SCIM-PROV-E-001 and SCIM-PROV-E-002 artefacts; file in `compliance/evidence/scim-provisioning/`; add cross-references to `docs/SOC2_READINESS.md §CC6.1` and `§CC6.3` evidence tables. | compliance-officer | **P1** | M6 | [ ] |

#### P2 — Post-GA and SOC 2 observation prep

| # | Task | Owner | Priority | Milestone | Status |
|---|---|---|---|---|---|
| 15 | Update `docs/SOC2_READINESS.md §9 G-001` from 🔴 → 🟢 Closed once §27.14 P0 tasks deploy to production. | compliance-officer | **P2** | M5 | [ ] |
| 16 | Evaluate `bulk: true` SCIM support for large-directory customers (> 10,000 seats) where sequential CRUD creates excessive latency. Decision: defer to post-Series A or if first enterprise customer requires > 10,000-seat directory sync. Document in `docs/DECISION_LOG.md`. | enterprise-architect | **P2** | M12 | [ ] |
| 17 | Collect SCIM-PROV-E-003 (zero `scim.rejected_sensitive_attribute`) and SCIM-PROV-E-004 (alert rule screenshots) artefacts for SOC 2 observation period start. | compliance-officer | **P2** | M9 | [ ] |

---

### 27.15 Open Questions

| ID | Question | Priority | Owner | Resolution path |
|---|---|---|---|---|
| **OQ-SSO-27.1** | **SCIM provisioning for non-SSO tenants.** §3.1 implies SCIM requires SSO to be configured (the SCIM token is associated with `tenant_sso_configs`). Should FORM support SCIM-only provisioning for enterprise tenants who use FORM's native email+password auth but want automated directory sync? This would require decoupling `tenant_scim_tokens` from `tenant_sso_configs`. | P2 | enterprise-architect | Evaluate at first customer asking for SCIM without SSO (estimated rare — most enterprise IdPs bundle SSO and SCIM). If demand exists, document decision in `docs/DECISION_LOG.md`. |
| **OQ-SSO-27.2** | **SCIM `PUT /Users` full-replace and audit diffing.** Currently `scim.user_updated` records `fields_changed` (attribute names only, no values). Should FORM store old/new values for role changes specifically, to give the SOC 2 auditor a full role-change trail? Risk: if role values are stored in the audit event, a cross-tenant read of the chain would expose role information. Recommendation: store role changes in a separate `tenant_users_role_history` table (not in the DEC-030 chain), accessible only to `compliance_reviewer` role under RLS. | ~~P1~~ **🟢 Resolved** | security-engineer + compliance-officer | **🟢 Resolved — DEC-049 (2026-06-14). `tenant_users_role_history` table adopted. See §28.** |
| **OQ-SSO-27.3** | **SCIM async vs. synchronous session revocation on deprovisioning.** §27.5.5 specifies synchronous KV write for session revocation (< 3 ms). Under the §22 KV revocation architecture this is correct. However, if the Worker's KV write fails (network partition), the SCIM response is still 200 but revocation is deferred to the Supabase fallback. The SLA commitment (`ENTERPRISE_SLA.md §3.7` P99 < 60 s session revocation) could be breached in this failure mode. Options: (a) return 503 on KV write failure and let the IdP retry (breaks idempotency guarantee); (b) accept the Supabase fallback path as within SLA (< 30 s for the Supabase blocklist SELECT path); (c) emit AL-REVOKE-01 and page devops-lead when the fallback path activates, but still return 200 to the IdP. Recommendation: option (c). | ~~P1~~ **🟢 Resolved** | platform-engineer + devops-lead | **🟢 Resolved — DEC-048 (2026-06-14). Option (c) adopted; AL-REVOKE-01 added. See §28.** |

---

*v1.9 (2026-06-13): §27 SCIM v2.0 Endpoint — Worker Implementation Design. Closes G-001 (§9): 🔴 Not designed → 🟡 Design complete, implementation pending. §27.1 purpose and DPA gate (G-013 block remains). §27.2 Worker file layout (`apps/scim-worker/src/`): six source files (index, auth, router, handlers/{users,groups,discovery}, errors, idempotency, events, types); six Cloudflare bindings (SCIM_KV separate from SSO_KV, SUPABASE_URL, SUPABASE_SERVICE_ROLE_KEY, SCIM_TOKEN_HASH_SECRET, AUDIT_QUEUE, IP_HASH_SALT). §27.3 token auth pipeline: HMAC-SHA256 hash → `SCIM_KV` 5-minute TTL cache → Supabase `tenant_scim_tokens` fallback; `ScimTokenCache` type; IP enforcement call to shared §25 `enforceScimIpAllowlist()`; `SCIM_TOKEN_HASH_SECRET` added to CRYPTOGRAPHY_POLICY.md rotation schedule. §27.4 router dispatch: fourteen route patterns for Users + Groups + schema discovery. §27.5 User handlers: POST (idempotency via `(tenant_id, external_user_id)` UNIQUE index; three outcomes: 201 create / 200 reactivate / 200 idempotent-replay; sensitive attribute guard; seat limit check; canonical event name `scim.user_deprovisioned` replacing deprecated `scim.user_deactivated`); GET by id (cross-tenant enumeration prevention via RLS); GET list (filter `eq` on four fields, max 200 results); PUT (full replace); PATCH (deactivation path: `is_active = false` → §22 KV `revoke:user:{tenant_id}:{user_id}` → 60 s revocation SLA); DELETE (aliased to PATCH deactivation). §27.6 Group handlers: POST (create mapping, default `member` role, idempotent on displayName match); PATCH (role re-evaluation via §19.3 `group_member_effective_role` view, higher-privilege-wins per DEC-039; both FORM-UUID and `externalId` member value formats handled; `scim.group_synced` event); DELETE (role downgrade cascade). §27.7 schema discovery: ServiceProviderConfig (bulk false, changePassword false, sort false); Schemas (only attributes FORM reads); ResourceTypes. §27.8 RFC 7644 error format (`scimError()` helper; seven scimType values; no tenant internals in detail strings). §27.9 idempotency contract: `(tenant_id, external_user_id)` UNIQUE index as primary key; email fallback for legacy IdPs without externalId; group PATCH operations inherently idempotent; `scim_request_id` dedup at `emit-audit-event` Worker. §27.10 six DEC-030 events: `scim.user_provisioned` (HIGH, 7yr), `scim.user_reactivated` (HIGH, 7yr), `scim.user_deprovisioned` (HIGH, 7yr — canonical name replacing `scim.user_deactivated`), `scim.user_updated` (STANDARD, 7yr), `scim.group_synced` (STANDARD, 5yr), `scim.rejected_sensitive_attribute` (HIGH, 7yr); SCIM-CHAIN-01 ordering invariant (deprovisioned must not precede provisioned for same user_id; WARNING not reject); privacy invariant: no name/email/health data in any payload. §27.11 four alerting rules AL-SCIM-01 through AL-SCIM-04 (P1: sensitive attr rejection burst; P2: 5xx error spike; P2: sync stall detection; P1: chain invariant violation). §27.12 SOC 2 evidence mapping CC6.1/CC6.3/CC6.4/CC6.6/CC7.2/PI1.1; four artefacts SCIM-PROV-E-001 through SCIM-PROV-E-004. §27.13 gap status update: G-001 🔴 → 🟡. §27.14 seventeen-item implementation checklist: 10× P0/M4–M5 (DEC-030 registration + deprecation of scim.user_deactivated, SCIM_KV + SCIM_TOKEN_HASH_SECRET setup, auth.ts + router.ts + errors.ts, users.ts, groups.ts, discovery.ts, AL-SCIM alert config, G-013 DPA gate), 4× P1/M5–M6 (Okta smoke test, Azure AD smoke test, load test at 100 req/min, SCIM-PROV-E-001/002 collection), 3× P2/M5–M12 (SOC2 gap close, SCIM bulk evaluation, SCIM-PROV-E-003/004 collection). §27.15 three open questions: OQ-SSO-27.1 (SCIM without SSO P2), OQ-SSO-27.2 (role change audit diffing P1 — `tenant_users_role_history` recommendation), OQ-SSO-27.3 (async vs synchronous revocation on KV failure P1 — option (c) recommendation). TOC updated to add §27. Header updated from v1.8 to v1.9. Cross-references: `docs/ENTERPRISE.md` (SCIM 2.0 enterprise feature commitment); `docs/AUDIT_LOG_SCHEMA.md §SCIM-Lifecycle` (six new DEC-030 events to register — P0 before M4); `docs/SOC2_READINESS.md §CC6.1/CC6.3` (evidence tables SCIM-PROV-E-001/002 to add post-observation); `docs/INCIDENT_RESPONSE.md R-24` (canonical source of `scim.user_deprovisioned` name); `docs/ENTERPRISE_SLA.md §3.7` (60 s session revocation SLA — OQ-SSO-27.3 async/sync decision); `docs/DATA_MODEL.md §4.2` (`tenant_sso_configs` schema — `tenant_scim_tokens` child table); §3.3 (attribute mapping — unchanged); §4 (RLS — unchanged; `app.current_tenant_id` GUC still required); §5 (role mapping — unchanged; group sync references §19.3 view); §14 (G-013 DPA gate — block must be cleared before production); §19 (group sync role evaluation — §19.3 view canonical source); §22 (session revocation KV — canonical revocation path); §25 (IP enforcement — `enforceScimIpAllowlist()` shared utility); `docs/CRYPTOGRAPHY_POLICY.md §5` (`SCIM_TOKEN_HASH_SECRET` rotation schedule — P0 checklist item 3). Owner: enterprise-architect + platform-engineer + security-engineer + compliance-officer.*

---

*v1.8.2 patch (2026-06-11): OQ-SSO-19.1/19.3 resolution + G-005 status update. §9 G-005 row updated: 🔴 Not designed → 🟡 Design complete, implementation pending; references §21 (v1.7) for full service account credential management design, KV token caching, group sync, and DEC-030 audit events. §19.9 OQ-SSO-19.1 updated: open (founder sign-off required) → 🟢 Resolved, citing DEC-039 (2026-06-11); higher-privilege-wins confirmed, `MAX()` aggregation retained in `group_member_effective_role` view. §19.9 OQ-SSO-19.3 updated: open (enterprise-architect decision required) → 🟢 Resolved, citing DEC-039; 500 groups/tenant default confirmed, configurable via `tenants.feature_limits` JSONB. §19.10 checklist P0 item "Get founder sign-off on OQ-SSO-19.1" marked [x] Done with DEC-039 reference. Owner: enterprise-architect + compliance-officer.*

*v1.8.3 patch (2026-06-11): §26.11 checklist items 7 and 12 closed. Item 7: `API_KEY_HASH_SECRET` confirmed present in `CRYPTOGRAPHY_POLICY.md` §5 Key Rotation Schedule (180-day cycle, re-hash on rotation) and `SOC2_READINESS.md` §56.3 Key Inventory — marked 🟢 Closed. Item 12: `DATA_MODEL.md §26` "API Key Authentication Schema" confirmed present with canonical DDL scope note and SSO_SCIM §26 cross-reference — marked 🟢 Closed. Owner: security-engineer + enterprise-architect.*

*v1.8.1 patch (2026-06-10): OQ-SSO-26.2 stale-status patch. §26.10 OQ-SSO-26.2 row updated: 🟡 P1 Open → 🟢 Resolved, citing DEC-035 (2026-06-01); soft enforcement (age alerts amber ≥ 365d, red ≥ 730d + CSM escalation) confirmed; provisional hard-enforcement trigger documented (key exceeds 365d without rotation at observation period start). §26.11 checklist item 11 marked [x] Done with DEC-035 reference. Owner: enterprise-architect + compliance-officer.*

---

## 28. SCIM Role Change Audit Trail & Session Revocation Fallback Design

### 28.1 Purpose & Scope

This section formally resolves the two P1 open questions carried since §27.15 (v1.9, 2026-06-13):

| OQ | Resolved by | Decision |
|---|---|---|
| **OQ-SSO-27.2** — SCIM role change audit trail: where to store old/new role values | **DEC-049** | `tenant_users_role_history` append-only table; not in DEC-030 chain |
| **OQ-SSO-27.3** — SCIM KV write failure during deprovisioning session revocation | **DEC-048** | Option (c): HTTP 200 + Supabase fallback + AL-REVOKE-01 emit |

Both decisions are **required before G-013 DPA clearance** (OQ-SSO-27.3) and **before SOC 2 observation period start** (OQ-SSO-27.2). The implementation is a self-contained addition to `apps/scim-worker/` and one new migration (`0068_tenant_users_role_history.sql`).

**Out of scope:** OQ-SSO-27.1 (SCIM without SSO) remains P2 open; no design decision in this section applies to that question.

---

### 28.2 OQ-SSO-27.3 Resolution — KV Write Failure Fallback Protocol (DEC-048)

#### 28.2.1 Decision

**Option (c) adopted.** When `env.SCIM_KV.put()` throws during the SCIM PATCH deprovisioning flow (§27.5.5), the Worker:

1. Immediately falls back to the Supabase blocklist INSERT path (§22 `kv_session_revocations` table).
2. Returns **HTTP 200** to the IdP — idempotency guarantee (§27.9) is preserved unconditionally.
3. Emits `scim.session_revocation_kv_fallback` DEC-030 HMAC-chained event (HIGH, 7yr) — provides the SOC 2 auditor with evidence that the fallback activated and that revocation completed via the alternate path.
4. Triggers **AL-REVOKE-01** (P1, PagerDuty `form-platform`) — alerts devops-lead to investigate KV regional degradation.

The Supabase fallback achieves revocation within < 30 s, well within the `ENTERPRISE_SLA.md §3.7` P99 ≤ 60 s SLA commitment.

#### 28.2.2 Rejection rationale for options (a) and (b)

| Option | Outcome | Rejection reason |
|---|---|---|
| **(a)** Return HTTP 503 on KV failure; let IdP retry | IdP retries the DELETE | Breaks the idempotency guarantee (§27.9). The retry fires a second deprovisioning flow that may race with a concurrent reactivation (`scim.user_reactivated` in-flight), creating a SCIM-CHAIN-01 ordering violation. SOC 2 CC6.3 evidence would show two `scim.user_deprovisioned` events for the same `user_id` with no intervening reactivation — auditor flag. |
| **(b)** Accept Supabase fallback silently; no alerting | Revocation completes; no signal emitted | CC7.2 gap: FORM has no detection of KV infrastructure degradation. A sustained KV outage could affect multiple tenants without any alert firing. Silent fallback is operationally equivalent to "we don't know our revocation path is degraded." |

#### 28.2.3 DPA G-013 language update

The following clause replaces the current §14.3.2 session revocation description:

> "Session revocation on employee deprovisioning uses a best-effort synchronous Cloudflare KV write (< 3 ms under normal conditions). In the event of a Cloudflare KV regional failure, an automatic Supabase fallback path achieves revocation within 30 seconds, remaining within the contracted 60-second P99 SLA commitment. The fallback activation is logged as a tamper-evident DEC-030 audit event (`scim.session_revocation_kv_fallback`, HIGH, 7-year retention) and triggers immediate infrastructure alert (AL-REVOKE-01) to the FORM engineering on-call."

#### 28.2.4 Worker implementation

In `apps/scim-worker/src/handlers/users.ts`, the deprovisioning PATCH path (§27.5.5) gains the following try/catch wrapper around the KV write:

```typescript
let revocationPath: 'kv' | 'supabase_fallback' = 'kv';

try {
  await env.SCIM_KV.put(
    `revoke:user:${tenantId}:${userId}`,
    '1',
    { expirationTtl: SESSION_REVOCATION_TTL_SECONDS },
  );
} catch (kvError: unknown) {
  revocationPath = 'supabase_fallback';

  // Fallback: Supabase blocklist table (§22 kv_session_revocations)
  const { error: sbError } = await supabase
    .from('kv_session_revocations')
    .insert({
      key: `revoke:user:${tenantId}:${userId}`,
      expires_at: new Date(Date.now() + SESSION_REVOCATION_TTL_MS).toISOString(),
    });

  if (sbError) {
    // Both paths failed — this is a P0; surface to Sentry before returning 200
    console.error('[SCIM][R-REVOKE] Both KV and Supabase revocation paths failed', {
      tenantId, scimRequestId, kvError, sbError,
    });
  }

  // Emit DEC-030 HMAC-chained event — fires AL-REVOKE-01 via audit queue
  await emitAuditEvent(env, {
    action: 'scim.session_revocation_kv_fallback',
    severity: 'HIGH',
    tenant_id: tenantId,
    scim_request_id: scimRequestId,
    fallback_path: 'supabase_blocklist',
    supabase_ok: !sbError,
    kv_error_class: kvError instanceof Error ? kvError.constructor.name : 'Unknown',
  });
}

// HTTP 200 regardless of revocation path — idempotency guarantee (§27.9)
return new Response(JSON.stringify(toScimUser(updatedUser)), {
  status: 200,
  headers: { 'Content-Type': 'application/scim+json' },
});
```

---

### 28.3 OQ-SSO-27.2 Resolution — SCIM Role Change Audit Trail (DEC-049)

#### 28.3.1 Decision

Role change values — `old_role` and `new_role` — produced by SCIM `PUT /Users` full-replace and group PATCH operations are stored in a **dedicated `tenant_users_role_history` table**, not in the DEC-030 HMAC chain.

The `scim.user_updated` event continues to record `fields_changed: ['role']` (attribute name only) as the tamper-evident chain anchor. The `scim_request_id` field in `tenant_users_role_history` cross-references the chain event to the actual role values, giving SOC 2 auditors a two-layer evidence chain:

```
DEC-030 chain:          scim.user_updated { fields_changed: ['role'], scim_request_id: "X" }
                                │
                                └── cross-reference (scim_request_id = "X")
                                          │
tenant_users_role_history:        { old_role: 'member', new_role: 'manager', scim_request_id: "X" }
                                    (RLS: compliance_reviewer + tenant_admin only)
```

#### 28.3.2 Rejection of in-chain role value storage

The `scim.user_updated` payload must **not** include `old_role`/`new_role` values because:

1. **Cross-tenant leakage risk.** The DEC-030 HMAC chain is replicated as a whole for auditor evidence export. An auditor receiving a bulk chain export for CC6.1/CC6.3 receives events from all tenants covered by the observation period. If role values appear in payloads, an export misconfiguration or accidental over-sharing exposes org-chart-sensitive data (e.g., distinguishing `manager` from `member`) across tenant boundaries.

2. **SOC 2 privacy floor.** `docs/ENTERPRISE.md §Privacy floor` rule 1: "HR can never see individual user data." Role names in the chain expose user-level role assignments; the tenant-scoped RLS table confines visibility to authenticated queries with explicit tenant scope.

3. **Future-proof GDPR Art. 17 erasure.** Role history rows carry a `user_id_pseudonym` column for the erasure path (§28.4 `chk_turh_user_xor_pseudonym`). Chain events cannot be pseudonymised post-emission — the chain's tamper-evident property is broken by in-place mutation.

#### 28.3.3 Evidence chain for auditors

A SOC 2 CC6.3 auditor reviewing role change governance uses the following two-step lookup:

```sql
-- Step 1: retrieve chain events where a role change was recorded
SELECT
  ae.occurred_at,
  ae.tenant_id,
  ae.scim_request_id,
  ae.fields_changed       -- contains 'role'
FROM audit_log_events ae
WHERE ae.action = 'scim.user_updated'
  AND 'role' = ANY(ae.fields_changed)
  AND ae.tenant_id = :tenant_id
ORDER BY ae.occurred_at DESC;

-- Step 2: retrieve role change values for a specific event
SELECT
  rh.changed_at,
  rh.old_role,
  rh.new_role,
  rh.changed_by,
  rh.scim_request_id      -- matches ae.scim_request_id from Step 1
FROM tenant_users_role_history rh
WHERE rh.tenant_id = :tenant_id
  AND rh.scim_request_id = :scim_request_id;
```

Step 1 is available to the auditor via the standard HMAC chain export. Step 2 requires `compliance_reviewer` role access (auditor fieldwork, not bulk export) — appropriate for a query that returns business-sensitive role assignment data.

---

### 28.4 `tenant_users_role_history` Schema

Cross-reference: `docs/DATA_MODEL.md §33` (canonical DDL definition). This section provides the SCIM context and integration pattern.

#### 28.4.1 DDL summary

```sql
-- Migration: 0068_tenant_users_role_history.sql

CREATE TABLE tenant_users_role_history (
  id                UUID         NOT NULL DEFAULT gen_random_uuid(),
  tenant_id         UUID         NOT NULL,
  user_id           UUID,
  user_id_pseudonym TEXT,
  scim_request_id   TEXT         NOT NULL,
  changed_by        TEXT         NOT NULL DEFAULT 'scim_sync',
  old_role          TEXT         NOT NULL,
  new_role          TEXT         NOT NULL,
  changed_at        TIMESTAMPTZ  NOT NULL DEFAULT NOW(),

  CONSTRAINT pk_tenant_users_role_history       PRIMARY KEY (id),
  CONSTRAINT fk_turh_tenant  FOREIGN KEY (tenant_id) REFERENCES tenants(id) ON DELETE CASCADE,
  CONSTRAINT fk_turh_user    FOREIGN KEY (user_id)   REFERENCES users(id)   ON DELETE SET NULL,
  CONSTRAINT chk_turh_user_xor_pseudonym CHECK (
    (user_id IS NOT NULL) != (user_id_pseudonym IS NOT NULL)
  ),
  CONSTRAINT chk_turh_role_changed CHECK (old_role <> new_role)
);

CREATE INDEX idx_turh_tenant_user   ON tenant_users_role_history (tenant_id, user_id)    WHERE user_id IS NOT NULL;
CREATE INDEX idx_turh_tenant_time   ON tenant_users_role_history (tenant_id, changed_at DESC);
CREATE INDEX idx_turh_scim_request  ON tenant_users_role_history (scim_request_id);
```

#### 28.4.2 Column notes

| Column | Purpose |
|---|---|
| `user_id` | FK to `users.id`; SET NULL on user delete (GDPR Art. 17 step 1) |
| `user_id_pseudonym` | Keyed HMAC `SHA-256(user_id \|\| ERASURE_PSEUDONYM_SALT)`; populated during Art. 17 erasure Worker; mutually exclusive with `user_id` (`chk_turh_user_xor_pseudonym`) |
| `scim_request_id` | Ties this row to the `scim.user_updated` DEC-030 chain event for cross-reference |
| `changed_by` | `'scim_sync'` for SCIM operations; `'admin_ui'` for admin dashboard role edits; `'jit_provisioning'` for §11 JIT flows |
| `chk_turh_role_changed` | Rejects rows where `old_role = new_role` — prevents no-op audit noise |

#### 28.4.3 RLS policies

```sql
ALTER TABLE tenant_users_role_history ENABLE ROW LEVEL SECURITY;

-- Compliance reviewer: full read (SOC 2 auditor fieldwork)
CREATE POLICY turh_compliance_read ON tenant_users_role_history
  FOR SELECT TO compliance_reviewer USING (true);

-- Tenant admin: read own tenant only
CREATE POLICY turh_tenant_admin_read ON tenant_users_role_history
  FOR SELECT TO tenant_admin
  USING (tenant_id = current_setting('app.current_tenant_id')::uuid);

-- form_system: INSERT only — append-only; no UPDATE, no DELETE
CREATE POLICY turh_system_insert ON tenant_users_role_history
  FOR INSERT TO form_system WITH CHECK (true);

-- form_api: no access (enforced by absence of policy)
```

#### 28.4.4 Worker integration

`recordRoleChange()` is called from the PUT and group PATCH handlers in `apps/scim-worker/src/handlers/users.ts` immediately after the role update commits to `tenant_users`:

```typescript
async function recordRoleChange(
  supabase: SupabaseClient,
  params: {
    tenantId: string;
    userId: string;
    scimRequestId: string;
    oldRole: string;
    newRole: string;
    changedBy?: 'scim_sync' | 'admin_ui' | 'jit_provisioning';
  },
): Promise<void> {
  if (params.oldRole === params.newRole) return;

  const { error } = await supabase
    .from('tenant_users_role_history')
    .insert({
      tenant_id:       params.tenantId,
      user_id:         params.userId,
      scim_request_id: params.scimRequestId,
      changed_by:      params.changedBy ?? 'scim_sync',
      old_role:        params.oldRole,
      new_role:        params.newRole,
    });

  if (error) {
    // Non-fatal: log to Sentry; SCIM operation already committed.
    // Do NOT throw — role history failure must not roll back the provisioning action.
    console.error('[SCIM][ROLE-HISTORY] Failed to record role change', { error, params });
  }
}
```

> **Non-fatal design rationale:** If `tenant_users_role_history` INSERT fails (e.g., transient Supabase edge), the role change itself has already committed to `tenant_users`. Rolling back the entire SCIM operation to preserve audit history would violate the SCIM idempotency contract and create a worse outcome (user stuck in wrong role state, IdP retries). The missing history row is detectable via a nightly reconciliation query (§28.9 checklist item 6); any gap is a P2 investigation item, not a P0 incident.

---

### 28.5 DEC-030 Event: `scim.session_revocation_kv_fallback`

| Field | Value |
|---|---|
| **action** | `scim.session_revocation_kv_fallback` |
| **severity** | HIGH |
| **retention** | 7 yr |
| **trigger** | `env.SCIM_KV.put()` throws during SCIM PATCH deprovisioning |
| **emitter** | `apps/scim-worker` (automated — no human actor) |

**Payload schema (Zod v2):**

```typescript
const ScimSessionRevocationKvFallbackPayload = z.object({
  tenant_id:       z.string().uuid(),
  scim_request_id: z.string().max(128),
  fallback_path:   z.literal('supabase_blocklist'),
  supabase_ok:     z.boolean(),                       // false if Supabase fallback also failed
  kv_error_class:  z.string().max(64),                // e.g. 'KVError', 'NetworkError'
});
```

**Privacy invariant:** No `user_id`, no email address, no role values in payload. `tenant_id` and `scim_request_id` are the minimum identifiers required for incident correlation.

**HMAC chain requirement (REVOKE-CHAIN-01):** `scim.session_revocation_kv_fallback` for a given `scim_request_id` must follow a `scim.user_deprovisioned` event for the same `scim_request_id` in the chain order. A chain break violates CC7.2 evidence integrity and triggers R-05.

---

### 28.6 Alert Rule: AL-REVOKE-01

| Rule | Condition | Severity | Routing | Dedup key | Auto-resolve |
|---|---|---|---|---|---|
| **AL-REVOKE-01** | Any `scim.session_revocation_kv_fallback` event emitted (count > 0 in 5-min window) | **P1** | PagerDuty `form-platform` → devops-lead | `scim-kv-fallback-{tenant_id}` (1-hour dedup window) | No — IC investigation required |

**Trigger source:** `AUDIT_QUEUE` consumer monitors `action = 'scim.session_revocation_kv_fallback'`; counter incremented in Cloudflare Analytics Engine metric `scim_kv_fallback_count`; Better Stack alert fires on `scim_kv_fallback_count > 0` in any 5-minute window.

**IC response:** Confirm whether `supabase_ok = true` in the event payload (revocation completed via fallback). If `supabase_ok = false`, escalate immediately to P0 (both revocation paths failed); activate R-02 (service outage) and R-26 (SCIM deprovisioning latency) in parallel.

---

### 28.7 SOC 2 Evidence Mapping

| Criterion | Evidence | Control statement |
|---|---|---|
| **CC6.3** — Removes access when appropriate | SCIM-ROLE-E-001 (`tenant_users_role_history` audit sample) + SCIM-ROLE-E-002 (DEC-030 cross-reference export) + SCIM-REVOKE-E-001 (AL-REVOKE-01 history) | Role changes are captured in append-only history; session revocation on deprovisioning completes within 60 s via dual-path architecture even under KV failure |
| **CC6.6** — Restricts logical access based on role | SCIM-ROLE-E-001 | Role transition audit trail demonstrates SCIM sync operated within authorised role boundaries; `compliance_reviewer` RLS gate enforces least-privilege access to role history |
| **CC7.2** — Monitors for anomalies | SCIM-REVOKE-E-001 (AL-REVOKE-01 history) | AL-REVOKE-01 provides detection coverage for Cloudflare KV degradation affecting the SCIM deprovisioning revocation path |
| **PI1.1** — Authorised access to sensitive data | SCIM-ROLE-E-001 + SCIM-ROLE-E-002 | `tenant_users_role_history` is accessible only to `compliance_reviewer` (auditor fieldwork) and `tenant_admin` (own-tenant scope); no `form_api` access; cross-reference via `scim_request_id` to chain events allows auditor to verify role changes without bulk export of sensitive role data |

**Evidence artefacts:**

| ID | Type | Query / Source | Path | Retention |
|---|---|---|---|---|
| **SCIM-ROLE-E-001** | CC6.3 / CC6.6 | `SELECT id, tenant_id, old_role, new_role, changed_at FROM tenant_users_role_history WHERE tenant_id = :id AND changed_at BETWEEN :obs_start AND :obs_end ORDER BY changed_at` (no `user_id` in export — pseudonymised column omitted) | `compliance/evidence/scim-role/SCIM-ROLE-E-001_<YYYY-QN>.csv` | 7 yr |
| **SCIM-ROLE-E-002** | CC6.3 / PI1.1 | DEC-030 `scim.user_updated` events where `'role' = ANY(fields_changed)` for observation period — cross-reference sample of 5 `scim_request_id` values to matching `tenant_users_role_history` rows | `compliance/evidence/scim-role/SCIM-ROLE-E-002_<YYYY-QN>.md` | 7 yr |
| **SCIM-REVOKE-E-001** | CC7.2 | AL-REVOKE-01 alert history for SOC 2 observation period (Better Stack export or zero-fire attestation) | `compliance/evidence/scim-revoke/SCIM-REVOKE-E-001_<YYYY-QN>.csv` | 3 yr |

---

### 28.8 Implementation Checklist

#### P0 — Before G-013 DPA cleared (M5)

| # | Task | Owner | Priority | Milestone | Status |
|---|---|---|---|---|---|
| 1 | Run `migration 0068_tenant_users_role_history.sql`; verify RLS policies with `SET ROLE compliance_reviewer` and `SET ROLE tenant_admin` in psql; add migration to `compliance/migrations/log.md`. | platform-engineer | **P0** | M5 | [ ] |
| 2 | Register `scim.session_revocation_kv_fallback` in `docs/AUDIT_LOG_SCHEMA.md §6.SCIM-Lifecycle` with Zod schema from §28.5; deploy schema to `emit-audit-event` Worker. | platform-engineer + compliance-officer | **P0** | M5 | [x] **Done — AUDIT_LOG_SCHEMA.md v2.6 (2026-06-14)** |
| 3 | Add `recordRoleChange()` utility (§28.4.4) to `apps/scim-worker/src/handlers/users.ts`; call after PUT full-replace and group PATCH role evaluation; verify `chk_turh_role_changed` rejects no-op rows in integration test. | platform-engineer | **P0** | M5 | [ ] |
| 4 | Add KV fallback try/catch (§28.2.4) to PATCH deprovisioning path in `apps/scim-worker/src/handlers/users.ts`; emit `scim.session_revocation_kv_fallback` via `emitAuditEvent()`. | platform-engineer | **P0** | M5 | [ ] |
| 5 | Configure **AL-REVOKE-01** in Better Stack: `scim_kv_fallback_count > 0` in 5-min window; PagerDuty `form-platform`; dedup key `scim-kv-fallback-{tenant_id}` (1-hour); no auto-resolve. | devops-lead | **P0** | M5 | [ ] |
| 6 | Update G-013 DPA template §14.3.2 with dual-path revocation language from §28.2.3 before DPA is signed with first enterprise customer. | compliance-officer + legal | **P0** | M5 (before first DPA execution) | [ ] |

#### P1 — Before SOC 2 observation period start (M9)

| # | Task | Owner | Priority | Milestone | Status |
|---|---|---|---|---|---|
| 7 | Collect **SCIM-ROLE-E-001**: run role history query for the 30-day pre-observation sample window; confirm `chk_turh_role_changed` constraint produced zero `old_role = new_role` rows (constraint violation audit); file to `compliance/evidence/scim-role/`. | compliance-officer | **P1** | M9 | [ ] |
| 8 | Collect **SCIM-ROLE-E-002**: export 5 representative `scim.user_updated` chain events where `'role' = ANY(fields_changed)`; manually cross-reference `scim_request_id` to matching `tenant_users_role_history` rows; document match in evidence file. | compliance-officer | **P1** | M9 | [ ] |
| 9 | Collect **SCIM-REVOKE-E-001**: export AL-REVOKE-01 alert history from Better Stack for observation period; if zero fires, write zero-fire attestation signed by devops-lead. | devops-lead + compliance-officer | **P1** | M9 | [ ] |
| 10 | Update `docs/SOC2_READINESS.md §CC6.3` evidence table to reference SCIM-ROLE-E-001, SCIM-ROLE-E-002, and SCIM-REVOKE-E-001. | compliance-officer | **P1** | M9 | [ ] |

#### P2 — Post-GA

| # | Task | Owner | Priority | Milestone | Status |
|---|---|---|---|---|---|
| 11 | Add nightly reconciliation query: `SELECT COUNT(*) FROM audit_log_events WHERE action = 'scim.user_updated' AND 'role' = ANY(fields_changed) AND NOT EXISTS (SELECT 1 FROM tenant_users_role_history WHERE scim_request_id = audit_log_events.scim_request_id)`; non-zero result → AL-SCIM-05 (P2 Slack `#compliance-alerts`); confirms no role change slipped without a history row. | devops-lead + platform-engineer | **P2** | M10 | [ ] |
| 12 | Add `tenant_users_role_history` to the Erasure Worker (`apps/erasure-worker/`) GDPR Art. 17 pipeline: on user hard-delete, set `user_id = NULL`, populate `user_id_pseudonym = SHA-256(user_id || ERASURE_PSEUDONYM_SALT)` for all matching rows; emit `gdpr.erasure_role_history_pseudonymised` (STANDARD, 7yr) DEC-030 event. | platform-engineer + compliance-officer | **P2** | M10 | [ ] |

---

### 28.9 Open Questions

| ID | Question | Priority | Owner | Resolution path |
|---|---|---|---|---|
| **OQ-SSO-28.1** | ~~AL-SCIM-05 naming collision check.~~ 🟢 **Resolved (2026-06-14, v4.74.1)** — AL-SCIM-05 confirmed available in the `docs/OBSERVABILITY.md §26` namespace (AL-SCIM-01..04 registered in §26.7b; AL-SCIM-MASS-01 and AL-SCIM-IP-01 use distinct suffixes). AL-SCIM-05 formally registered in **OBSERVABILITY §26.7c** as the canonical definition. No ID change required. | ~~P2~~ Resolved | devops-lead | 🟢 OBSERVABILITY §26.7c (2026-06-14). |
| ~~**OQ-SSO-28.2**~~ **🟢 Resolved — §29 (DEC-051, 2026-06-14)** | ~~**`tenant_users_role_history` retention period.**~~ **Resolved:** **7-year retention** adopted. Grounds: (1) audit-chain continuity — `scim.user_updated` DEC-030 events are retained 7yr; purging the table at 3yr creates a verifiable gap in SOC 2 CC6.3 evidence; (2) Ukrainian Tax Code Art. 44 (business record obligation); (3) multi-cycle SOC 2 observation evidence continuity. 3-year alternative rejected — see §29.2. GDPR Art. 4(1): rows containing `user_id` are personal data; legal basis for 7yr retention is Art. 17(3)(b). `user_id` is pseudonymised (SET NULL + `user_id_pseudonym` populated) on user erasure; full hard-delete via pg_cron job 32 at 7yr boundary. See §29 for full decision record. | ~~P1~~ **🟢 Closed** | compliance-officer | **Done — DEC-051 (2026-06-14). §29.** |

---

*v2.0 additions (2026-06-14): §28 SCIM Role Change Audit Trail & Session Revocation Fallback Design — resolves two P1 open questions from §27.15 (v1.9, 2026-06-13). OQ-SSO-27.2 → **DEC-049**: `tenant_users_role_history` append-only table adopted as the role change audit trail; role values (old_role/new_role) are explicitly excluded from the DEC-030 HMAC chain to prevent cross-tenant leakage in auditor bulk exports; `scim_request_id` cross-reference ties chain events to table rows for SOC 2 fieldwork. OQ-SSO-27.3 → **DEC-048**: Option (c) adopted for SCIM KV write failure — HTTP 200 maintained (idempotency); Supabase blocklist fallback activated (< 30 s revocation, within 60-second P99 SLA); `scim.session_revocation_kv_fallback` DEC-030 event (HIGH, 7yr) emitted; AL-REVOKE-01 (P1 PagerDuty `form-platform`) fires on any KV fallback activation. New DEC-030 event: `scim.session_revocation_kv_fallback` (HIGH, 7yr; `supabase_ok` boolean surfaces dual-path failure; no user_id in payload). REVOKE-CHAIN-01 invariant: `scim.session_revocation_kv_fallback` must follow `scim.user_deprovisioned` in chain order for the same `scim_request_id`. `tenant_users_role_history` DDL (§28.4, migration `0068_tenant_users_role_history.sql`): `chk_turh_role_changed` (no-op guard), `chk_turh_user_xor_pseudonym` (GDPR Art. 17 erasure path), three indexes (tenant_user, tenant_time, scim_request). `recordRoleChange()` TypeScript helper — non-fatal design (INSERT failure does not roll back SCIM operation; reconciliation query AL-SCIM-05 provides gap detection). RLS: `compliance_reviewer` full read; `tenant_admin` own-tenant read; `form_system` INSERT; no `form_api` access. Three SOC 2 evidence artefacts: SCIM-ROLE-E-001 (CC6.3/CC6.6 role history sample), SCIM-ROLE-E-002 (CC6.3/PI1.1 DEC-030 cross-reference), SCIM-REVOKE-E-001 (CC7.2 AL-REVOKE-01 history). Twelve-item implementation checklist: 6× P0/M5 (migration, DEC-030 registration, recordRoleChange, KV fallback handler, AL-REVOKE-01, DPA language update), 4× P1/M9 (evidence collection, SOC2 cross-reference), 2× P2/M10 (reconciliation alert, erasure pipeline). Two new open questions: OQ-SSO-28.1 (AL-SCIM-05 naming collision P2), OQ-SSO-28.2 (tenant_users_role_history retention period P1). §27.15 OQ-SSO-27.2 and OQ-SSO-27.3 status updated to 🟢 Resolved with DEC references. TOC updated to add §28. Cross-references: `docs/DATA_MODEL.md §33` (canonical DDL definition for tenant_users_role_history); `docs/AUDIT_LOG_SCHEMA.md §6.SCIM-Lifecycle` (scim.session_revocation_kv_fallback event to register — P0 before M5); `docs/OBSERVABILITY.md §26` (SSO/SCIM identity observability — AL-SCIM-05 naming check, OQ-SSO-28.1); `docs/ENTERPRISE_SLA.md §3.7` (60 s session revocation SLA — §28.2 fallback stays within SLA); `docs/INCIDENT_RESPONSE.md R-05` (HMAC chain break — REVOKE-CHAIN-01 violation trigger); `docs/INCIDENT_RESPONSE.md R-26` (SCIM deprovisioning latency — co-activation if supabase_ok = false); `docs/CRYPTOGRAPHY_POLICY.md §5` (ERASURE_PSEUDONYM_SALT — used in OQ-SSO-28.2 erasure path); `docs/DECISION_LOG.md` (DEC-048 and DEC-049). Owner: enterprise-architect + security-engineer + platform-engineer + compliance-officer.*

---

## 29. OQ-SSO-28.2 Resolution — `tenant_users_role_history` Retention Period (DEC-051)

### 29.1 Purpose & Decision

This section closes **OQ-SSO-28.2** (P1, §28.9) — the outstanding question of how long `tenant_users_role_history` rows must be retained under GDPR Art. 5(1)(e) storage limitation.

| Field | Value |
|---|---|
| **Decision** | **7-year retention** adopted for `tenant_users_role_history`, measured from `changed_at` |
| **Decision ref** | DEC-051 (2026-06-14) |
| **Previous status** | 🟡 Proposed — pending legal sign-off per §28.9 / DATA_MODEL §33.8 |
| **New status** | 🟢 **Resolved** |
| **Owners** | compliance-officer + enterprise-architect + security-engineer |
| **Effective** | 2026-06-14 — applies to all existing and future rows |

---

### 29.2 Decision Rationale

Three independent grounds justify 7-year retention. Each is sufficient on its own; together they are conclusive.

#### 29.2.1 Ground 1 — Audit-Chain Continuity (primary)

`scim.user_updated` DEC-030 HMAC-chained events — which serve as the chain anchor for every role change recorded in `tenant_users_role_history` — carry a **7-year retention period** per `docs/AUDIT_LOG_SCHEMA.md §6.SCIM-Lifecycle`.

The audit evidence model requires that when an auditor follows a chain event by its `scim_request_id` cross-reference (§28.3, §28.7), the corresponding `tenant_users_role_history` row exists. If the table is purged at 3 years while the chain events persist until year 7, the auditor finds:

1. A `scim.user_updated` chain event with `fields_changed: ['role']` in year 4.
2. A `scim_request_id` → table cross-reference that returns **zero rows** (purged at 3yr).
3. The chain anchor becomes unverifiable: SOC 2 CC6.3 evidence collapses for that event.

This gap was explicitly documented in DATA_MODEL §33.8 as the reason the 3-year alternative was rejected. DEC-051 confirms that rejection as binding.

**Chain retention alignment requirement (TURH-CHAIN-01):** The retention period for `tenant_users_role_history` rows **must not be shorter** than the retention period of the `scim.user_updated` DEC-030 events that reference them. Any future reduction of `scim.user_updated` retention below 7 years must trigger a parallel review of `tenant_users_role_history` retention via the DECISION_LOG process.

#### 29.2.2 Ground 2 — Legal Obligations

| Jurisdiction | Obligation | Retention required |
|---|---|---|
| Ukraine (primary incorporation) | Tax Code Art. 44 — business transaction records | 7 years |
| EU (enterprise customer employees) | EU audit-record doctrine (CJEU C-13/19) — access-control records must be retained at least as long as the protected resource | 7 years (coaching data: 3yr; contracts: 7yr → determinative) |
| SOC 2 (auditor expectation) | AICPA Guide — evidence should span the observation period plus two prior periods for comparative assessments | 7 years (three 2-year cycles + buffer) |

Role assignment records constitute business transaction records because they determine which organisational data (`docs/ENTERPRISE.md §Privacy floor for enterprise`) employees were authorised to access — and thus have downstream consequences for corporate governance and data protection audit trails.

#### 29.2.3 Ground 3 — SOC 2 Multi-Cycle Evidence Continuity

SOC 2 Type II observation periods run for 12 months. A mature SOC 2 programme covering the third Type II cycle (year 5+) requires auditors to compare current controls to prior-period baselines. A 3-year retention window would leave evidence gaps for:

- CC6.3 (removes access when appropriate) — role changes older than 3 years become unverifiable
- CC6.6 (restricts access based on role) — the historical baseline for role assignments is incomplete
- CC4.1 (performance evaluations of controls) — trend analysis across multiple observation periods requires longitudinal data

7-year retention satisfies all three criteria across a 3-cycle SOC 2 programme (standard for IPO-readiness) and aligns with FORM's investor-grade compliance ambition per `docs/ENTERPRISE.md §Why enterprise`.

---

### 29.3 GDPR Art. 4(1) Personal Data Classification

**Question:** Does `tenant_users_role_history` contain personal data, and does 7-year retention require a specific legal basis?

#### 29.3.1 Column-Level Analysis

| Column | Personal data? | Reasoning |
|---|---|---|
| `id` (UUID PK) | No | Opaque internal identifier; not linkable externally |
| `tenant_id` (UUID) | No | Organisational identifier for the employer entity |
| `user_id` (UUID FK → `users.id`) | **Yes — pseudonymous** | Links to a natural person record; GDPR Art. 4(5) pseudonymous data is still personal data per Recital 26 |
| `user_id_pseudonym` (TEXT) | **Yes — pseudonymous** | HMAC of `user_id`; still linkable to the person by FORM using the `ERASURE_PSEUDONYM_SALT` (`docs/CRYPTOGRAPHY_POLICY.md §5`) |
| `old_role` / `new_role` (`form_role_enum`) | No (in isolation) | Access-control category values; no natural-person attribute |
| `changed_at` (TIMESTAMPTZ) | No (in isolation) | Timestamp without subject link |
| `scim_request_id` (TEXT) | No | Operational correlation identifier |
| `changed_by` (`role_change_source_enum`) | No | System-action category |

#### 29.3.2 Combined-Record Analysis

The combination of `user_id` (or `user_id_pseudonym`) + `changed_at` + `old_role`/`new_role` constitutes **personal data** under GDPR Art. 4(1) because:

- `user_id` is a pseudonym for an identifiable natural person (the enterprise employee).
- The role history reveals **when** an employee's access level changed, which frequently correlates with identifiable employment events: onboarding (member → member remains, first assignment), promotion (member → tenant_manager), disciplinary action (tenant_admin → member), and offboarding (any → deprovisioned, captured via `scim.user_deprovisioned` event per §28.3).
- A sufficiently motivated actor with access to both the `tenant_users_role_history` table and the employer's HR system could reconstruct an employee's employment lifecycle from the role change pattern.

**Conclusion:** `tenant_users_role_history` rows containing `user_id` are personal data per Art. 4(1). Processing requires a valid legal basis under Art. 6(1).

#### 29.3.3 Legal Basis for 7-Year Retention

The applicable legal basis is **GDPR Art. 6(1)(c)** (processing necessary for compliance with a legal obligation to which the controller is subject), read in conjunction with **Art. 17(3)(b)** (erasure does not apply where processing is necessary for compliance with a legal obligation).

Specific legal obligations:

1. **Ukrainian Tax Code Art. 44** — 7-year business record retention (Ground 2 above).
2. **SOC 2 Type II evidence requirement** — contractual obligation in enterprise agreements (`docs/ENTERPRISE_SLA.md §3`) to maintain SOC 2 Type II certification, which requires multi-period evidence retention.
3. **DEC-030 HMAC-chain audit integrity** — FORM's published enterprise DPA commits to providing tamper-evident audit logs; purging corroborating table rows while chain events persist would breach this commitment.

**Art. 17 interaction:** The right to erasure (Art. 17(1)) is **restricted** for these rows by Art. 17(3)(b) until the 7-year retention window expires. The pseudonymisation step (§28.4 / DATA_MODEL §33.7) — setting `user_id = NULL` and populating `user_id_pseudonym` at erasure — satisfies the employee's practical right to de-identification while preserving the audit record under Art. 17(3)(b). After the 7-year window, full hard-deletion removes even the pseudonym.

**HR floor confirmation:** Role history is accessible only to `compliance_reviewer` (auditor fieldwork) and `tenant_admin` (own tenant only). HR personnel who are not designated `tenant_admin` in FORM have **zero access** to this table. This is consistent with `docs/ENTERPRISE.md §Privacy floor for enterprise` — HR never sees individual access-control history via any FORM-provided API or dashboard.

---

### 29.4 Enterprise DPA Template Update Requirement

The G-013 DPA template (`docs/SSO_SCIM_IMPLEMENTATION.md §14`) must be updated before execution with any EU enterprise customer to include the following disclosure in the Annex B "Retention periods and deletion" table:

| Data category | Retention period | Deletion method | Legal basis |
|---|---|---|---|
| Access control change logs (`tenant_users_role_history`) | 7 years from the date of each role change event (`changed_at`) | Pseudonymisation of `user_id` on employee departure / GDPR Art. 17 request; full hard-delete at 7yr boundary | Art. 6(1)(c) + Art. 17(3)(b) — legal obligation (Tax Code Art. 44 + SOC 2 audit trail) |

**Consumer privacy policy:** No update required. `tenant_users_role_history` is an enterprise-only table. Consumer-tier users (`plan_tier = 'free'` or `'elite'`) have no entries in this table.

**Employee notice (enterprise):** Employer is responsible for informing employees of this processing under their own employee privacy notice. FORM's DPA template already obliges the employer (Art. 28 controller) to provide adequate notice. The DPA Annex B update above provides the specific language the employer must reflect in their employee notice. Compliance-officer to confirm this language with outside counsel before first EU enterprise DPA execution.

---

### 29.5 pg_cron Retention Job — Job 32

```sql
-- Job 32: tenant_users_role_history — 7-year retention purge
-- Schedule: 04:00 UTC daily (after job 31 wearable_sync_freshness_check at 07:05;
--           staggered to avoid Postgres I/O contention with §33 retention jobs)
-- Owner: platform-engineer (register in docs/OBSERVABILITY.md §12.6 pg_cron registry as job 32)

SELECT cron.schedule(
  'turh-retention-purge',
  '0 4 * * *',
  $$
  -- Phase 1: hard-delete pseudonymised rows past 7yr retention
  -- Safety gate: only delete rows where GDPR Art. 17 pseudonymisation has already run
  -- (user_id IS NULL AND user_id_pseudonym IS NOT NULL confirms the erasure path completed)
  WITH deleted AS (
    DELETE FROM tenant_users_role_history
    WHERE changed_at < NOW() - INTERVAL '7 years'
      AND user_id IS NULL
      AND user_id_pseudonym IS NOT NULL
    RETURNING id
  )
  -- Emit DEC-030 event for compliance evidence (via pg_net to emit-audit-event Worker)
  SELECT net.http_post(
    url     := current_setting('app.audit_worker_url'),
    body    := json_build_object(
      'action',    'system.turh_retention_purge_completed',
      'severity',  'STANDARD',
      'payload',   json_build_object(
        'rows_deleted', (SELECT COUNT(*) FROM deleted),
        'window_years', 7,
        'job',          32
      )
    )::text,
    headers := '{"Content-Type":"application/json"}'::jsonb
  );

  -- Phase 2: detect active-user rows past retention window (erasure pipeline miss)
  -- Fires AL-TURH-01 if any row with user_id IS NOT NULL exceeds 7yr boundary
  WITH stale AS (
    SELECT COUNT(*) AS stale_count
    FROM tenant_users_role_history
    WHERE changed_at < NOW() - INTERVAL '7 years'
      AND user_id IS NOT NULL
  )
  SELECT CASE
    WHEN stale_count > 0 THEN
      net.http_post(
        url     := current_setting('app.audit_worker_url'),
        body    := json_build_object(
          'action',    'system.turh_stale_active_user_detected',
          'severity',  'HIGH',
          'payload',   json_build_object(
            'stale_count', stale_count,
            'window_years', 7,
            'alert_rule',  'AL-TURH-01'
          )
        )::text,
        headers := '{"Content-Type":"application/json"}'::jsonb
      )
    ELSE NULL
  END
  FROM stale;
  $$
);
```

**Safety gate rationale:** The Phase 1 `user_id IS NULL AND user_id_pseudonym IS NOT NULL` conditions guarantee:

1. The row has completed the GDPR Art. 17 pseudonymisation step (DATA_MODEL §33.7, step 1).
2. The row no longer contains directly identifying `user_id` data.
3. Purging a row with `user_id IS NOT NULL` is never silent — Phase 2 would have fired AL-TURH-01 for that row.

**Phase 2 trigger condition:** A stale active-user row (`user_id IS NOT NULL`, `changed_at > 7yr`) means either:
- The employee has been active for more than 7 years and the role was assigned on their first day (expected for long-tenured employees — no incident, but needs compliance-officer awareness).
- The GDPR erasure pipeline failed to run on employee departure (incident — requires investigation).

The alert language in AL-TURH-01 below distinguishes these two cases by checking whether `tenant_users.deleted_at IS NOT NULL` for the referenced `user_id` (i.e., whether the user has been offboarded in FORM's records).

---

### 29.6 Alert Rule: AL-TURH-01

| Rule | Condition | Severity | Routing | Dedup key | Auto-resolve |
|---|---|---|---|---|---|
| **AL-TURH-01** | `system.turh_stale_active_user_detected` event emitted by pg_cron job 32 (i.e., any `tenant_users_role_history` row with `user_id IS NOT NULL` and `changed_at < NOW() - 7 years`) | **P2** | Slack `#compliance-alerts` → compliance-officer | `turh-stale-active-{date}` (24-hour dedup; one Slack message per calendar day regardless of row count) | No — compliance-officer must investigate and close |

**Triage playbook:**

1. Query stale rows: `SELECT tenant_id, user_id, changed_at FROM tenant_users_role_history WHERE changed_at < NOW() - INTERVAL '7 years' AND user_id IS NOT NULL;`
2. For each `user_id`, check `SELECT deleted_at FROM tenant_users WHERE id = :user_id`:
   - `deleted_at IS NULL` (user still active): long-tenured employee — **no incident**. Document in compliance-officer log. The row will be pseudonymised when the user eventually departs.
   - `deleted_at IS NOT NULL` (user departed): erasure pipeline miss — **P1 incident**. Escalate to platform-engineer; run manual pseudonymisation step per DATA_MODEL §33.7; emit `gdpr.erasure_role_history_pseudonymised` DEC-030 event (STANDARD, 7yr) per DATA_MODEL §33.10 checklist item 3.
3. Close AL-TURH-01 with documented finding in `compliance/alerts/turh-stale-YYYY-MM-DD.md`.

---

### 29.7 SOC 2 Evidence Mapping

| Criterion | Evidence | Control statement |
|---|---|---|
| **CC6.3** — Removes access when appropriate | SCIM-ROLE-E-001 (§28.7, role history sample) + TURH-RET-E-001 (§29.8, retention compliance record) | Role change history is retained for 7 years, matching the DEC-030 chain event retention; SOC 2 auditors can trace any chain event to a corroborating table row across the full observation window |
| **CC6.6** — Restricts logical access based on role | SCIM-ROLE-E-001 | Longitudinal role transition trail demonstrates access controls operated within authorised boundaries across multi-cycle SOC 2 programmes |
| **CC4.1** — Performance evaluations of controls | TURH-RET-E-001 (annual cron run log) | pg_cron job 32 provides ongoing automated enforcement of the retention policy; AL-TURH-01 zero-fire attestation demonstrates the erasure pipeline operated correctly during the observation period |
| **P5.1** — Personal information retained no longer than necessary | DEC-051 (this section) + DATA_MODEL §33.8 | Defined retention period documented with legal basis (Art. 6(1)(c) + Art. 17(3)(b)); automated pg_cron enforcement; pseudonymisation on departure preserves audit record without re-identification capability |

---

### 29.8 Evidence Artefact: TURH-RET-E-001

| Field | Value |
|---|---|
| **Artefact ID** | TURH-RET-E-001 |
| **SOC 2 criteria** | CC6.3, CC4.1, P5.1 |
| **Contents** | (a) Annual `turh-retention-purge` pg_cron run log export from Supabase (rows deleted count + timestamp per day); (b) AL-TURH-01 zero-fire attestation for the observation period (or documented triage notes for any fires); (c) this section (DEC-051) as the retention policy evidence document |
| **Path** | `compliance/evidence/scim-role/TURH-RET-E-001_<YYYY>.md` |
| **Frequency** | Annual (collected on SOC 2 evidence collection cycle, aligned with SCIM-ROLE-E-001/002 schedule) |
| **Retention** | 3 years (operational evidence; the DEC-051 policy document itself is retained indefinitely as part of this versioned document) |

**Auditor narrative for CC4.1:** FORM implemented an automated 7-year retention policy for `tenant_users_role_history` via a daily pg_cron job (job 32). The retention period was established by DEC-051 (2026-06-14) on three documented grounds: audit-chain continuity, legal obligation (Tax Code Art. 44), and SOC 2 multi-cycle evidence continuity. The safety gate in the pg_cron SQL (`user_id IS NULL AND user_id_pseudonym IS NOT NULL`) ensures that pseudonymisation per GDPR Art. 17 always precedes hard-deletion. AL-TURH-01 provides detective coverage for erasure pipeline failures. TURH-RET-E-001 constitutes the annual evidence that the control operated throughout the observation period.

---

### 29.9 OQ-SSO-28.2 Status

| Status | 🟢 **Resolved — DEC-051 (2026-06-14)** |
|---|---|
| **Decision** | 7-year retention for `tenant_users_role_history` |
| **Grounds** | Audit-chain continuity (§29.2.1) + legal obligation (§29.2.2) + SOC 2 multi-cycle evidence (§29.2.3) |
| **3-year alternative** | Rejected — confirmed by DEC-051 as creating an irremediable SOC 2 CC6.3 evidence gap |
| **DATA_MODEL cross-update** | `docs/DATA_MODEL.md §33.8` — 7-year row: "Proposed" → **🟢 Resolved (DEC-051, 2026-06-14)**; §33.10 checklist item 4 → **🟢 Done** |

---

### 29.10 Implementation Checklist

#### P0 — Before SOC 2 observation period start (M9)

| # | Task | Owner | Priority | Milestone | Status |
|---|---|---|---|---|---|
| 1 | Register pg_cron job 32 `turh-retention-purge` in Supabase (§29.5 DDL); add to `docs/OBSERVABILITY.md §12.6` pg_cron registry as job 32; verify first successful dry-run on staging (confirm no rows eligible for deletion on a fresh database). | platform-engineer | **P0** | M9 | [ ] |
| 2 | Register two new DEC-030 events in `docs/AUDIT_LOG_SCHEMA.md §6`: `system.turh_retention_purge_completed` (STANDARD, 1yr — operational cleanup log) and `system.turh_stale_active_user_detected` (HIGH, 3yr — escalation evidence). Deploy event types to `emit-audit-event` Worker event registry. | platform-engineer + compliance-officer | **P0** | M9 | [ ] |
| 3 | Configure AL-TURH-01 in Better Stack / Slack integration: trigger on `system.turh_stale_active_user_detected` event; route to Slack `#compliance-alerts`; dedup key `turh-stale-active-{date}` (24h window); no auto-resolve; link to §29.6 triage playbook in alert description. | devops-lead | **P0** | M9 | [ ] |
| 4 | Update DPA template (§14.3.2 Annex B) with §29.4 retention schedule row ("Access control change logs: 7 years"). Confirm with outside counsel before first EU enterprise DPA execution. | compliance-officer + legal | **P0** | Before first EU enterprise DPA | [ ] |
| 5 | Update `docs/DATA_MODEL.md §33.8` retention table: 7-year row status from "Proposed" → 🟢 Resolved; §33.10 checklist item 4 → 🟢 Done (DEC-051, 2026-06-14). | compliance-officer | **P0** | M9 | [x] **Done — see DATA_MODEL §33 patch notes** |

#### P1 — Before first SOC 2 evidence collection (M9)

| # | Task | Owner | Priority | Milestone | Status |
|---|---|---|---|---|---|
| 6 | Collect TURH-RET-E-001 (§29.8): export first pg_cron job 32 run log (30-day pre-observation sample); document AL-TURH-01 zero-fire status or triage notes; file at `compliance/evidence/scim-role/TURH-RET-E-001_2026.md`. | compliance-officer | **P1** | M9 | [ ] |
| 7 | Add `compliance/p1/retention-decisions.md` entry for `tenant_users_role_history`: record DEC-051, 7-year retention, legal basis Art. 6(1)(c) + Art. 17(3)(b), effective date 2026-06-14. (Fulfils DATA_MODEL §33.10 original checklist item 4 requirement.) | compliance-officer | **P1** | M9 | [ ] |

#### P2 — Ongoing governance

| # | Task | Owner | Priority | Milestone | Status |
|---|---|---|---|---|---|
| 8 | Annual review: confirm `scim.user_updated` DEC-030 retention remains 7yr; if any change proposed, trigger TURH-CHAIN-01 retention alignment review (§29.2.1) before implementation. Document in `docs/DECISION_LOG.md`. | compliance-officer | **P2** | Annual (M12, M24, …) | [ ] |

---

---

## §30 OQ-MOBILE-03 Resolution — Mobile SSO Browser Mode & Deep-Link Security (DEC-059)

**Owner:** security-engineer + platform-engineer + enterprise-architect
**SOC 2 scope:** CC6.1, CC6.2, CC8.1
**Last updated:** 2026-06-14
**Closes:** OQ-MOBILE-03 from `docs/OBSERVABILITY.md §28.13` (P1 — before first enterprise SSO customer, M10)
**Decision:** DEC-059 — System browser mandated for all mobile SSO redirects; embedded WebView prohibited.

---

### 30.1 Purpose and Scope

OQ-MOBILE-03 (`docs/OBSERVABILITY.md §28.13`) identified a security architecture gap in FORM's mobile SSO flow: React Native's `Linking` API handles the `form://` deep link that returns from the IdP after SAML assertion POST or OIDC authorization code redirect, but the open question was whether the *outbound* authentication redirect — sending the user to the IdP login page — should use the system browser or an embedded WebView.

The question was flagged because certain enterprise IdPs (specifically Azure AD B2C custom policy flows and some Okta org-level configurations) exhibit redirect behaviour that some integrations handle with an embedded WebView. OQ-MOBILE-03 asked: should FORM follow that approach, or enforce system-browser-only with IdP-specific workarounds?

This section documents the security analysis (§30.4), the DEC-059 decision (§30.3), the React Native implementation (§30.5), per-IdP compatibility details (§30.6), Universal Links / App Links callback hardening (§30.7), Azure AD B2C OIDC workaround (§30.8), two DEC-030 audit events (§30.9), one alert rule (§30.10), SOC 2 evidence (§30.11), and the implementation checklist (§30.13).

**Out of scope:** IdP-initiated flows (§1.2, §3) do not involve a mobile outbound redirect; they are excluded. Consumer iOS login via Apple Sign In is excluded (SSO scope is enterprise only — per document header). Session revocation KV (§22) and CAEP (§23) are unaffected.

**Privacy floor:** No user_id, employee name, email, or health data appears in any DEC-030 event defined in this section. The `browser_mode` field records `"system_browser"` or `"webview_blocked"` — a configuration state, not personal data.

---

### 30.2 Mobile SSO Redirect Context

The §1.2 OIDC SP-Initiated flow and SAML SP-Initiated flow diagrams describe the browser / admin dashboard client. For the React Native mobile client the flow diverges at step 2 (outbound redirect to IdP):

```
React Native App                  FORM Worker               Customer IdP
      |                               |                           |
      |  1. User taps "Sign in        |                           |
      |     with [Company SSO]"       |                           |
      |------------------------------>|                           |
      |                               |                           |
      |  2. Worker returns            |                           |
      |     IdP auth URL              |                           |
      |<------------------------------|                           |
      |                               |                           |
      |  3. App opens IdP URL         |                           |
      |     in ??? (DECISION POINT)   |                           |
      |        (a) expo-web-browser   |                           |
      |            openAuthSessionAsync                           |
      |        (b) <WebView>          |                           |
      |                               |                           |
      |  4. User authenticates at IdP |                           |
      |<-----------------------------------------------------------
      |                               |                           |
      |  5. IdP redirects to          |                           |
      |     form://auth/callback?...  |                           |
      |     OR https://form.coach/... |                           |
      |     (Universal Link)          |                           |
      |                               |                           |
      |  6. App receives callback     |                           |
      |     (Linking.addEventListener)|                           |
      |------------------------------>|                           |
      |                               |                           |
      |  7. FORM Worker validates     |                           |
      |     code/assertion, issues    |                           |
      |     session JWT               |                           |
      |<------------------------------|                           |
```

Step 3 is the decision point. DEC-059 resolves it to option (a) unconditionally.

---

### 30.3 DEC-059 Decision — System Browser Mandated

**Decision:** FORM's React Native mobile application MUST use `WebBrowser.openAuthSessionAsync()` from `expo-web-browser` for ALL enterprise SSO outbound redirects. Embedding a `<WebView>` component for any SSO authentication step is **prohibited** — this is a hard security requirement, not a preference.

- **iOS:** `openAuthSessionAsync` uses `ASWebAuthenticationSession` (separate sandboxed process; host app cannot observe URL, cookies, or keystrokes).
- **Android:** `openAuthSessionAsync` uses Chrome Custom Tabs (separate process with Android Credential Manager integration; Safe Browsing enabled by default).
- **Custom scheme callback (`form://`):** Permitted for OIDC SP-initiated as a fallback; however, Universal Links (iOS) / App Links (Android) via `https://form.coach/auth/mobile/callback/{tenant_id}` are **preferred** to eliminate custom-scheme hijacking risk on unpatched devices (§30.7).
- **Azure AD B2C:** Use OIDC PKCE mode — not SAML — for any tenant using Azure AD B2C custom policies (§30.8). System browser is fully compatible with OIDC PKCE mode.
- **Okta, Entra, Google:** All three are compatible with `openAuthSessionAsync` using their standard OIDC authorization endpoints (§30.6).

**Provisional hard gate:** If a future IdP is discovered that genuinely cannot complete authentication via a system browser under any configuration, platform-engineer + security-engineer + enterprise-architect must jointly evaluate the case. The evaluation requires: (1) documented evidence the IdP vendor cannot support system browser; (2) security-engineer threat model for the WebView approach; (3) founder + compliance-officer sign-off; (4) a new DEC entry in `docs/DECISION_LOG.md`. The prohibition stands until that sign-off is obtained.

---

### 30.4 Security Analysis — System Browser vs. Embedded WebView

| Security Property | System Browser (`ASWebAuthenticationSession` / Chrome Custom Tabs) | Embedded `<WebView>` |
|---|---|---|
| **Process isolation** | Runs in a separate OS process. Host app cannot access URL, cookies, DOM, or keystrokes during authentication. | Runs in the same process as the React Native app. Any malicious SDK, native module, or future app update could instrument `shouldOverrideUrlLoading()` / `onReceivedSslError()` to intercept credentials. |
| **TLS certificate validation** | Enforced by the OS (SecureTransport / BoringSSL). Cannot be disabled by the host app. | Host app can override `onReceivedSslError()` on Android to silently accept invalid certificates — disabling HTTPS protection entirely. iOS WKWebView is harder to subvert but still allows `decidePolicyForNavigationAction` interception. |
| **Phishing detection** | Safari (iOS) and Chrome (Android) include Google Safe Browsing / fraud detection for URLs loaded in system browser tabs. | No equivalent phishing detection. A lookalike IdP URL (e.g., `https://login.microsoftonfine.com`) shows no warning. |
| **Credential Manager / Autofill** | Chrome Custom Tabs integrates with Android Credential Manager (passkeys, saved passwords). Safari integrates with iCloud Keychain. | WebView autofill is opt-in and inconsistent; passkeys are not supported in WebView contexts as of Android 14 / iOS 17. |
| **Cookie / session isolation** | `ASWebAuthenticationSession` uses an isolated cookie store separate from the app and from WKWebView — prevents the host app from reading SSO session cookies post-auth. | Shares a WebView cookie store. The app can read IdP session cookies post-authentication via `CookieManager.getInstance().getCookie()` (Android) or `HTTPCookieStorage` bridging (iOS). |
| **RFC 9700 compliance** | RFC 9700 §4.1 (OAuth 2.0 Security Best Current Practice): "The authorization server MUST NOT allow the embedding of its authorization endpoint in embedded user agents." Using a system browser satisfies this requirement by design. | Violates RFC 9700 §4.1. Enterprise customers whose security teams audit the mobile app for OAuth BCP compliance will flag this as a Critical finding. |
| **SOC 2 auditor narrative** | CC6.1: credentials are handled in a sandboxed OS process with no host-app access — demonstrates industry-standard secure auth design. | A WebView approach would require explaining why FORM chose a non-standard, lower-security path — auditors routinely flag this under CC6.1 as a control weakness. |

**Conclusion:** Embedded WebView creates four independently sufficient grounds for rejection: (1) process isolation failure, (2) RFC 9700 non-compliance, (3) credential manager incompatibility, (4) TLS bypass risk. No IdP compatibility benefit is sufficient to accept these risks when system browser alternatives exist for all FORM-supported IdPs.

---

### 30.5 React Native Implementation

**File:** `apps/mobile/src/auth/sso-browser.ts`

```typescript
// apps/mobile/src/auth/sso-browser.ts
// Canonical mobile SSO browser launcher — DEC-059.
// NEVER use <WebView> for SSO. openAuthSessionAsync is the only permitted path.

import * as WebBrowser from 'expo-web-browser';
import * as Linking from 'expo-linking';
import { Platform } from 'react-native';

export type SsoBrowserResult =
  | { type: 'success'; callbackUrl: string }
  | { type: 'cancel' }
  | { type: 'error'; message: string };

/**
 * Opens the IdP authorization URL in a system browser:
 *   - iOS:     ASWebAuthenticationSession (sandboxed process)
 *   - Android: Chrome Custom Tabs (separate process)
 * Never opens a WebView. See docs/SSO_SCIM_IMPLEMENTATION.md §30.3 (DEC-059).
 */
export async function openSsoBrowser(
  idpAuthUrl: string,
  tenantId: string,
): Promise<SsoBrowserResult> {
  // Prefer Universal Link callback (https) over custom scheme (form://)
  // to eliminate custom-scheme hijacking on unpatched Android devices.
  // See §30.7 for Universal Links / App Links configuration.
  const callbackScheme = Platform.OS === 'ios'
    ? `https://form.coach/auth/mobile/callback/${tenantId}`
    : `https://form.coach/auth/mobile/callback/${tenantId}`;

  let result: WebBrowser.WebBrowserAuthSessionResult;
  try {
    result = await WebBrowser.openAuthSessionAsync(idpAuthUrl, callbackScheme, {
      // preferEphemeralSession: true on iOS prevents sharing cookies with Safari.
      // Set to false so the user's existing IdP session (e.g., already logged into Okta)
      // carries over — avoids forcing re-authentication on every app open.
      preferEphemeralSession: false,
      // showInRecents: false on Android prevents the auth tab appearing in the
      // task switcher, reducing the surface for task-hijacking attacks.
      showInRecents: false,
      // createTask: false ensures the Chrome Custom Tab does not create a separate
      // Android back-stack task, preventing deep-link interception via task affinity.
      createTask: false,
    });
  } catch (err) {
    return { type: 'error', message: (err as Error).message };
  }

  if (result.type === 'success' && result.url) {
    return { type: 'success', callbackUrl: result.url };
  }
  if (result.type === 'cancel' || result.type === 'dismiss') {
    return { type: 'cancel' };
  }
  return { type: 'error', message: `Unexpected browser result: ${result.type}` };
}
```

**Lint guard** — add to `.eslintrc.js` to prevent accidental WebView usage in the auth directory:

```js
// .eslintrc.js — added as part of DEC-059
{
  files: ['apps/mobile/src/auth/**/*.{ts,tsx}'],
  rules: {
    // Prohibit <WebView> in the auth directory unconditionally.
    // WebView is never permitted for SSO. See docs/SSO_SCIM_IMPLEMENTATION.md §30.3.
    'no-restricted-imports': ['error', {
      paths: [{ name: 'react-native-webview', message: 'WebView is prohibited in the auth module (DEC-059).' }],
    }],
  },
}
```

**Unit test assertion** (runs in CI on every PR touching `apps/mobile/`):

```typescript
// apps/mobile/src/__tests__/auth/no-webview-in-sso.test.ts
import { execSync } from 'child_process';

describe('DEC-059: No WebView in SSO auth module', () => {
  it('contains zero WebView imports in apps/mobile/src/auth/', () => {
    const result = execSync(
      'grep -r "react-native-webview\\|<WebView" apps/mobile/src/auth/ || true',
    ).toString();
    expect(result.trim()).toBe('');
  });
});
```

---

### 30.6 IdP Compatibility Matrix

| IdP | Protocol | System Browser Compatible? | Notes |
|---|---|---|---|
| **Okta** | OIDC (preferred) | ✅ Full | Standard PKCE OIDC flow. Callback to `https://form.coach/auth/mobile/callback/{tenant_id}` (Universal Link) or `form://auth/callback` (custom scheme). No configuration change required. |
| **Okta** | SAML | ✅ Full | ACS URL `https://form.coach/auth/saml/{tenant_id}/acs` is a server-side callback — the mobile client simply opens the Okta SSO URL in system browser; the SAML POST happens server-to-server after user authenticates. IdP-initiated SAML returns to the same ACS, then FORM issues a deep-link redirect to the mobile app. |
| **Microsoft Entra ID (Azure AD)** | OIDC (preferred) | ✅ Full | Standard PKCE OIDC. `redirect_uri` must be registered in the Entra app registration as `https://form.coach/auth/mobile/callback/{tenant_id}`. Entra's system-browser redirect works reliably on both iOS and Android. |
| **Microsoft Entra ID** | SAML | ✅ Full | Same as Okta SAML — server-side ACS callback. No mobile-browser interaction required after initial redirect to IdP login page. |
| **Azure AD B2C** | SAML (custom policy) | ⚠️ Avoid | Some B2C custom policies POST the SAML assertion to a form inside a redirect chain that some implementations handle with a WebView. **Mitigation: configure B2C tenants to use OIDC PKCE mode instead of SAML** (§30.8). All B2C custom policies that expose an OIDC endpoint work correctly with system browser. |
| **Azure AD B2C** | OIDC (custom policy) | ✅ Full | OIDC PKCE mode works with `openAuthSessionAsync`. Use `https://login.microsoftonline.com/{b2c_tenant}.onmicrosoft.com/{policy_name}/oauth2/v2.0/authorize` as the auth URL. The redirect completes in the system browser with no WebView required. |
| **Google Workspace (OIDC)** | OIDC | ✅ Full | Google explicitly recommends Chrome Custom Tabs for OAuth on Android (their documentation states: "We strongly recommend against using WebViews"). `hd` domain restriction applies as per §2.3. |
| **PingFederate / PingOne** | OIDC or SAML | ✅ Full | Both protocols tested by Ping's own React Native SDK, which uses Chrome Custom Tabs / ASWebAuthenticationSession. No WebView required. |
| **OneLogin** | OIDC or SAML | ✅ Full | Standard OAuth 2.0 / SAML behaviour; system browser compatible. Redirect URI must be registered. |
| **Generic SAML IdP** | SAML | ✅ Full | The ACS callback is server-side. Opening the IdP SSO URL in a system browser and waiting for the deep-link callback is the universal pattern for SAML on mobile. |

**Decision for "⚠️ Avoid" case:** Any tenant currently using Azure AD B2C with a SAML custom policy MUST be migrated to OIDC PKCE mode before the mobile SSO feature is enabled for that tenant. The CSM onboarding playbook (§7) includes a pre-enablement check: if `sso_protocol = 'saml'` AND `idp_type = 'azure_ad_b2c'`, block mobile SSO enablement and flag to CSM for migration.

---

### 30.7 Universal Links (iOS) and App Links (Android) for SSO Callback

Using `form://auth/callback` as the SSO redirect URI introduces a **custom scheme hijacking risk**: on devices running Android ≤ 11 (API level ≤ 30), any app can register the `form://` scheme and intercept the authorization code or SAML artifact before FORM's app receives it.

**Mitigation: Universal Links (iOS) and App Links (Android)**

Universal Links / App Links bind a specific `https://` domain to a specific app, verified by the OS at install time. The IdP redirect goes to `https://form.coach/auth/mobile/callback/{tenant_id}` — the OS intercepts this HTTPS URL and routes it directly to the FORM mobile app without opening a browser. No other app can claim the `form.coach` domain.

**iOS — `apple-app-site-association` (AASA):**

```json
// https://form.coach/.well-known/apple-app-site-association
{
  "applinks": {
    "apps": [],
    "details": [
      {
        "appID": "TEAMID.coach.form.app",
        "paths": ["/auth/mobile/callback/*"]
      }
    ]
  }
}
```

Served at `https://form.coach/.well-known/apple-app-site-association` with `Content-Type: application/json`, no redirect, no authentication. The Cloudflare Worker at `form.coach` must serve this file at that path (static asset or KV-backed).

**Android — `assetlinks.json`:**

```json
// https://form.coach/.well-known/assetlinks.json
[{
  "relation": ["delegate_permission/common.handle_all_urls"],
  "target": {
    "namespace": "android_app",
    "package_name": "coach.form.app",
    "sha256_cert_fingerprints": ["<SHA256_OF_RELEASE_SIGNING_CERT>"]
  }
}]
```

Served at `https://form.coach/.well-known/assetlinks.json` with `Content-Type: application/json`.

**Registering the Universal Link redirect URI:** The FORM Worker at `POST /auth/mobile/callback/{tenant_id}` (or `GET`, for OIDC code) must accept the `redirect_uri` value `https://form.coach/auth/mobile/callback/{tenant_id}` and treat it identically to the custom scheme equivalent. This value must also be registered in every supported IdP's application configuration (see §30.6 column "Notes" for per-IdP instructions).

**Fallback:** If a device is running Android ≤ 11 with no Play Services, or the AASA/assetlinks fetch fails at install time, the OS falls back to opening `https://form.coach/auth/mobile/callback/{tenant_id}` in the browser. The Worker at that URL must detect this condition (absence of `X-Requested-Mobile: true` header from the React Native client) and render a deeplink redirect page: `<meta http-equiv="refresh" content="0;url=form://auth/callback?{params}">` — this falls back to the custom scheme for that specific device only, with the hijacking risk scoped to pre-Android-12 devices (< 10% of FORM's target enterprise fleet, per §28.6 device tier distribution).

---

### 30.8 Azure AD B2C — OIDC PKCE Mode Workaround

Tenants using Azure AD B2C for employee SSO sometimes configure SAML as the primary protocol because their B2C custom policy was originally built for web apps. For mobile SSO, FORM requires OIDC PKCE mode. This is a standard B2C capability — no new policy is needed; only a new application registration.

**Configuration steps for tenant IT admin (provided by CSM at onboarding):**

1. In the Azure portal, navigate to **Azure AD B2C → App registrations → New registration**.
2. Set the redirect URI to `https://form.coach/auth/mobile/callback/{tenant_id}` (type: Web, not SPA or Public client — the FORM Worker handles PKCE exchange server-side).
3. Under **Authentication → Implicit grant and hybrid flows**, ensure both boxes are **unchecked** (PKCE via authorization code flow only; no implicit).
4. Note the **Application (client) ID** and the **B2C tenant domain** (`{your-tenant}.b2clogin.com`).
5. Provide FORM CSM with: Client ID, B2C tenant domain, User flow / policy name (e.g., `B2C_1_signupsignin`).

**FORM CSM configures `tenant_sso_configs`:**

```sql
UPDATE tenant_sso_configs SET
  sso_protocol         = 'oidc',
  idp_type             = 'azure_ad_b2c',
  oidc_discovery_url   = 'https://{b2c_tenant}.b2clogin.com/{b2c_tenant}.onmicrosoft.com/{policy_name}/v2.0/.well-known/openid-configuration',
  oidc_client_id       = '{b2c_application_client_id}',
  oidc_client_secret   = '{encrypted_client_secret}',  -- optional; PKCE can be confidential
  mobile_sso_enabled   = true
WHERE tenant_id = :tenant_id;
```

The existing OIDC callback worker (§1.2) handles Azure AD B2C OIDC tokens identically to Entra OIDC — the `iss` claim format differs (`{b2c_tenant}.b2clogin.com`) but JWKS validation and `sub`/`email` extraction follow the same path.

---

### 30.9 DEC-030 HMAC-Chained Audit Events

Two new events, both registered in `docs/AUDIT_LOG_SCHEMA.md §6.SSO-Mobile`:

| Event Type | Severity | Retention | Payload Fields | Notes |
|---|---|---|---|---|
| `sso.mobile_browser_mode_set` | STANDARD | 7 yr | `tenant_id`, `mobile_sso_enabled` (bool), `callback_scheme` (`"universal_link"` or `"custom_scheme"`), `idp_type`, `sso_protocol`, `changed_by_user_id` | Emitted when `mobile_sso_enabled` is toggled or callback scheme is changed in `tenant_sso_configs`. Establishes an auditable record that FORM's system-browser policy was active for the tenant at the time of each enterprise pilot. `user_id` of changing admin; no employee data. |
| `sso.mobile_webview_blocked` | HIGH | 7 yr | `tenant_id`, `attempted_url_hash` (SHA-256 of the IdP auth URL, first 32 chars), `source_module` (`"sso-browser.ts"`), `blocked_at` | Emitted if any code path in `apps/mobile/src/auth/` attempts a WebView open for a URL matching an SSO endpoint pattern. Should never fire in production — its presence in the chain is itself an incident indicator (triggers AL-SSO-WEB-01 below). |

```typescript
// Emit sso.mobile_browser_mode_set when CSM enables mobile SSO for a tenant
const event = {
  event_type: 'sso.mobile_browser_mode_set',
  severity: 'STANDARD',
  payload: {
    tenant_id:         tenantId,
    mobile_sso_enabled: true,
    callback_scheme:   universalLinksConfigured ? 'universal_link' : 'custom_scheme',
    idp_type:          ssoConfig.idp_type,           // 'okta' | 'azure_ad' | 'azure_ad_b2c' | 'google' | 'generic_saml'
    sso_protocol:      ssoConfig.sso_protocol,       // 'oidc' | 'saml'
    changed_by_user_id: actorUserId,                 // UUID; no PII
  },
};
await emitDec030Event(event, supabaseAdminClient);
```

**MOBILE-CHAIN-01 ordering invariant:** A `sso.mobile_webview_blocked` event MUST be preceded in the HMAC chain by a `sso.mobile_browser_mode_set` event with `mobile_sso_enabled: true` for the same `tenant_id`. A chain where the blocked event precedes enablement is a MOBILE-CHAIN-01 violation and triggers R-05 investigation.

---

### 30.10 Alert Rule AL-SSO-WEB-01

| Field | Value |
|---|---|
| **Alert ID** | AL-SSO-WEB-01 |
| **Condition** | Any `sso.mobile_webview_blocked` DEC-030 event emitted in production |
| **Severity** | **P1** |
| **Routing** | PagerDuty `form-security` service → security-engineer (on-call) + platform-engineer |
| **Dedup key** | `sso-webview-blocked-{tenant_id}-{date}` (24h dedup window) |
| **Auto-resolve** | No |
| **Response SLA** | Acknowledge within 15 minutes; root cause within 4 hours |

**Triage:** If AL-SSO-WEB-01 fires:
1. Identify the `source_module` from the `sso.mobile_webview_blocked` event payload.
2. Find the PR or commit that introduced the WebView call (git log `apps/mobile/src/auth/`).
3. If an SDK update pulled in a dependency that opened WebView: treat as a supply-chain incident (R-08); engage security-engineer.
4. If intentional code change: roll back the mobile release via EAS Update OTA channel; open a severity-1 Linear ticket; notify founder.
5. The ESLint guard (§30.5) should have caught this in CI — investigate why it did not fire.

**Note:** AL-SSO-WEB-01 should never fire under normal operation. Its sole purpose is to detect the unexpected — a CI bypass, a dependency injection, or a future code change that violates DEC-059. The alert's value is its zero-tolerance nature: one event is one incident.

Register AL-SSO-WEB-01 in `docs/OBSERVABILITY.md §26.8` `sso_browser_security` subsection after implementation.

---

### 30.11 SOC 2 Evidence Mapping

| SOC 2 Criterion | Control | Evidence Artefact |
|---|---|---|
| **CC6.1** — Logical access controls | Mobile SSO authentication is handled in a sandboxed system browser process (ASWebAuthenticationSession / Chrome Custom Tabs) that the host app cannot observe. The `sso.mobile_browser_mode_set` DEC-030 events establish an auditable record of when this control was activated for each tenant. | **SSO-WEB-E-001:** Export of all `sso.mobile_browser_mode_set` events for the observation period, demonstrating that every enterprise tenant with mobile SSO enabled used the system-browser mode (`callback_scheme` present). File at `compliance/evidence/sso/SSO-WEB-E-001_<YYYY>.csv`. 7-year retention. |
| **CC6.2** — Prior to issuing credentials, FORM registers and authorises the relying party | Universal Links / App Links (§30.7) ensure only the signed FORM app binary can receive the SSO callback URL — the OS verifies the association file at install time, preventing credential interception by other apps. | **SSO-WEB-E-002:** AASA file (`apple-app-site-association`) + Android `assetlinks.json` version-controlled snapshots; CI test result showing no WebView in `apps/mobile/src/auth/` for the current build (`no-webview-in-sso.test.ts` green). File at `compliance/evidence/sso/SSO-WEB-E-002_<YYYY>.md`. 3-year retention. |
| **CC8.1** — Security testing in SDLC | The `no-webview-in-sso.test.ts` test and the ESLint `no-restricted-imports` rule together form a CI gate that blocks any PR from introducing a WebView into the auth flow. AL-SSO-WEB-01 provides post-deployment detective coverage. | **SSO-WEB-E-002** (CI test log) as above; plus the **AL-SSO-WEB-01 zero-fire attestation** for the observation period (PagerDuty `form-security` service incident history showing zero AL-SSO-WEB-01 fires). 3-year retention. |

**Auditor narrative for CC6.1:** FORM's mobile SSO authentication flow uses `ASWebAuthenticationSession` (iOS) and Chrome Custom Tabs (Android) exclusively, pursuant to DEC-059 (2026-06-14). These APIs run authentication in a separate OS-managed process with no access from the host application. The `sso.mobile_browser_mode_set` DEC-030 chain events (SSO-WEB-E-001) demonstrate that this control was active for every enterprise tenant with mobile SSO enabled during the observation period. The no-WebView CI gate (SSO-WEB-E-002) and AL-SSO-WEB-01 zero-fire attestation provide ongoing detective assurance that no code change bypassed this control.

---

### 30.12 OQ-MOBILE-03 Status

| Status | 🟢 **Resolved — DEC-059 (2026-06-14)** |
|---|---|
| **Decision** | System browser (`expo-web-browser openAuthSessionAsync`) mandated; WebView prohibited |
| **Rationale** | RFC 9700 §4.1 compliance + process isolation + TLS bypass prevention + credential manager integration + SOC 2 CC6.1 posture |
| **Azure AD B2C workaround** | OIDC PKCE mode (§30.8) — system browser compatible, no custom policy change needed |
| **Callback hardening** | Universal Links (iOS) + App Links (Android) preferred over `form://` custom scheme (§30.7) |
| **OBSERVABILITY cross-update** | `docs/OBSERVABILITY.md §28.13` OQ-MOBILE-03 row: 🟡 P1 Open → **🟢 Resolved (DEC-059, 2026-06-14)** |

---

### 30.13 Implementation Checklist

#### P0 — Before mobile SSO feature flag enabled for any enterprise tenant (M8)

| # | Task | Owner | Priority | Milestone | Status |
|---|---|---|---|---|---|
| 1 | Implement `openSsoBrowser()` in `apps/mobile/src/auth/sso-browser.ts` per §30.5 TypeScript spec. Replace any existing `Linking.openURL()` or `<WebView>` SSO call. | platform-engineer | **P0** | M8 | [ ] |
| 2 | Add ESLint `no-restricted-imports` rule for `react-native-webview` in `apps/mobile/src/auth/**` per §30.5. Confirm rule fires on a test import, then remove the test import. | platform-engineer | **P0** | M8 | [ ] |
| 3 | Add `no-webview-in-sso.test.ts` CI test per §30.5. Confirm it passes on main. Add to the `mobile-security` test suite tag in CI configuration. | platform-engineer | **P0** | M8 | [ ] |
| 4 | Register `sso.mobile_browser_mode_set` (STANDARD, 7yr) and `sso.mobile_webview_blocked` (HIGH, 7yr) in `docs/AUDIT_LOG_SCHEMA.md §6.SSO-Mobile`; deploy updated event registry to `emit-audit-event` Worker. Write integration test: toggle `mobile_sso_enabled` in staging → confirm `sso.mobile_browser_mode_set` event appears in `audit_log_events`. | platform-engineer + compliance-officer | **P0** | M8 | [x] Done — `docs/AUDIT_LOG_SCHEMA.md v2.10` (2026-06-15): Mobile SSO Browser Mode events section added; both events registered with Zod schemas, MOBILE-CHAIN-01 invariant, SOC 2 evidence mapping, and SSO-WEB-E-001 artefact definition. Retention table rows added. |
| 5 | Configure AL-SSO-WEB-01 in PagerDuty `form-security` service: trigger on `sso.mobile_webview_blocked` event; dedup key `sso-webview-blocked-{tenant_id}-{date}` (24h window); no auto-resolve; link to §30.10 triage steps in alert description. | devops-lead | **P0** | M8 | [ ] |
| 6 | Register AL-SSO-WEB-01 in `docs/OBSERVABILITY.md §26.8` as a new `sso_browser_security` alert subsection row. | compliance-officer | **P0** | M8 | [x] Done — `docs/OBSERVABILITY.md v4.2` (2026-06-15): §26.8 `sso_browser_security` subsection added with AL-SSO-WEB-01 definition (P1 PagerDuty `form-security`; trigger: `sso.mobile_webview_blocked`; dedup 24h; no auto-resolve; privacy floor `attempted_url_hash` SHA-256[:32] only). |

#### P1 — Before first enterprise tenant enables mobile SSO (M10)

| # | Task | Owner | Priority | Milestone | Status |
|---|---|---|---|---|---|
| 7 | Deploy `apple-app-site-association` to `https://form.coach/.well-known/apple-app-site-association` via Cloudflare Worker static asset (§30.7). Verify: open `https://form.coach/auth/mobile/callback/test` on a development device → confirm iOS opens FORM app, not Safari. | platform-engineer | **P1** | M10 | [ ] |
| 8 | Deploy `assetlinks.json` to `https://form.coach/.well-known/assetlinks.json` with the production release signing certificate SHA-256 fingerprint (§30.7). Verify: `adb shell pm get-app-links coach.form.app` shows `verified` status. | platform-engineer | **P1** | M10 | [ ] |
| 9 | Register `https://form.coach/auth/mobile/callback/{tenant_id}` as an allowed redirect URI in all four supported IdP types: Okta, Entra, Google Workspace, and generic SAML ACS. Update the CSM onboarding checklist (§7.1) to include this redirect URI alongside the existing ACS URL. | platform-engineer + customer-success | **P1** | M10 | [ ] |
| 10 | Update the Azure AD B2C pre-enablement check in the mobile SSO feature-flag Worker: if `idp_type = 'azure_ad_b2c'` AND `sso_protocol = 'saml'`, return a 409 with message `"Mobile SSO requires OIDC PKCE mode for Azure AD B2C tenants. Contact your CSM."` (§30.6). | platform-engineer | **P1** | M10 | [ ] |
| 11 | Collect SSO-WEB-E-001 after first enterprise tenant activates mobile SSO: export `sso.mobile_browser_mode_set` events from `audit_log_events`; confirm `callback_scheme` field is `"universal_link"` for all tenants with Universal Links deployed; file at `compliance/evidence/sso/SSO-WEB-E-001_2026.csv`. | compliance-officer | **P1** | M10 + 30 days | [ ] |
| 12 | Collect SSO-WEB-E-002: snapshot `apple-app-site-association` and `assetlinks.json` from production; export the `no-webview-in-sso.test.ts` CI run result for the current mobile release; export AL-SSO-WEB-01 zero-fire attestation from PagerDuty `form-security`; file at `compliance/evidence/sso/SSO-WEB-E-002_2026.md`. | compliance-officer | **P1** | M10 + 30 days | [ ] |

#### P2 — Ongoing governance

| # | Task | Owner | Priority | Milestone | Status |
|---|---|---|---|---|---|
| 13 | Update `docs/SSO_CLIENT_CONFIG.md` (if it exists) or `docs/ENTERPRISE_ONBOARDING.md` with per-IdP redirect URI registration steps for mobile (`https://form.coach/auth/mobile/callback/{tenant_id}`), including Azure AD B2C OIDC PKCE configuration steps from §30.8. | customer-success + platform-engineer | **P2** | M11 | [ ] |
| 14 | Annual review: confirm `ASWebAuthenticationSession` and Chrome Custom Tabs remain the recommended system-browser APIs on the current iOS / Android OS versions. If either API is deprecated or a better alternative emerges (e.g., passkey-native flow), evaluate migration path and open a new DEC entry. | security-engineer + platform-engineer | **P2** | Annual (M12, M24, …) | [ ] |

---

*v2.2 additions (2026-06-14): §30 OQ-MOBILE-03 Resolution — Mobile SSO Browser Mode & Deep-Link Security (DEC-059). Closes OQ-MOBILE-03 from `docs/OBSERVABILITY.md §28.13` (P1 — before first enterprise SSO customer, M10). Decision: **system browser mandated** (`expo-web-browser openAuthSessionAsync` → ASWebAuthenticationSession on iOS / Chrome Custom Tabs on Android); embedded WebView **prohibited** in all SSO authentication paths. §30.1 scopes to enterprise mobile SSO only; consumer Apple Sign In excluded; §22 session revocation and §23 CAEP unaffected. §30.2 mobile SSO redirect context: ASCII flow diagram showing step 3 as the WebView vs. system-browser decision point. §30.3 DEC-059 decision: `WebBrowser.openAuthSessionAsync()` is the only permitted path; provisional WebView exception gate requires platform-engineer + security-engineer + enterprise-architect + founder + compliance-officer joint sign-off and new DEC entry before any exception is granted. §30.4 security analysis: seven-row comparison table (process isolation, TLS certificate validation, phishing detection, Credential Manager / autofill, cookie / session isolation, RFC 9700 §4.1 compliance, SOC 2 auditor narrative) — all seven properties favor system browser; embedded WebView fails on RFC 9700 §4.1 and creates process-isolation, TLS-bypass, and phishing-detection gaps. §30.5 TypeScript implementation: `apps/mobile/src/auth/sso-browser.ts` — `openSsoBrowser()` using `WebBrowser.openAuthSessionAsync` with `preferEphemeralSession: false` (preserve IdP session), `showInRecents: false` (Android), `createTask: false` (Android task hijacking prevention); ESLint `no-restricted-imports` rule blocking `react-native-webview` in `apps/mobile/src/auth/**`; `no-webview-in-sso.test.ts` grep-based CI assertion. §30.6 IdP compatibility matrix: nine rows (Okta OIDC + SAML, Entra OIDC + SAML, Azure AD B2C SAML ⚠️ avoid / OIDC ✅, Google Workspace OIDC, PingFederate, OneLogin, generic SAML) — all compatible with system browser except Azure AD B2C SAML (mitigation: OIDC PKCE mode, §30.8). Pre-enablement guard: block `mobile_sso_enabled` if `idp_type = 'azure_ad_b2c'` AND `sso_protocol = 'saml'` (409 response). §30.7 Universal Links / App Links callback hardening: AASA JSON (`/auth/mobile/callback/*` path, FORM teamId); Android assetlinks.json (release-cert SHA-256 fingerprint, `coach.form.app`; served at `form.coach/.well-known/`); fallback deeplink redirect page for pre-Android-12 devices (`<meta http-equiv="refresh">` to `form://`). §30.8 Azure AD B2C OIDC PKCE workaround: five-step IT admin configuration (new app registration, redirect URI, disable implicit grant, note client ID + B2C domain, share with FORM CSM); `tenant_sso_configs` UPDATE snippet (`idp_type = 'azure_ad_b2c'`, `oidc_discovery_url` B2C-specific format). §30.9 two new DEC-030 HMAC-chained events: `sso.mobile_browser_mode_set` (STANDARD, 7yr — tenant_id, mobile_sso_enabled bool, callback_scheme enum, idp_type, sso_protocol, changed_by_user_id; no employee PII); `sso.mobile_webview_blocked` (HIGH, 7yr — tenant_id, attempted_url_hash SHA-256[:32], source_module, blocked_at; emitted only if code violation detected); MOBILE-CHAIN-01 ordering invariant (blocked event must follow mode_set for same tenant_id; inversion → R-05). §30.10 AL-SSO-WEB-01 (P1 — PagerDuty `form-security`, triggered by any `sso.mobile_webview_blocked` event in production; dedup per tenant 24h; zero-fire is the expected state; triage tree: identify source_module → find PR → rollback OTA → investigate CI bypass). §30.11 three SOC 2 evidence artefacts: SSO-WEB-E-001 (CC6.1 — `sso.mobile_browser_mode_set` DEC-030 export, all tenants with mobile SSO, 7yr); SSO-WEB-E-002 (CC6.2 + CC8.1 — AASA + assetlinks.json snapshots + no-webview CI test log + AL-SSO-WEB-01 zero-fire attestation, 3yr). Auditor narrative for CC6.1 confirms process isolation, DEC-059 as the operative decision, and SSO-WEB-E-001 as the chain-level evidence. §30.12 OQ-MOBILE-03 status: 🟡 P1 Open → 🟢 Resolved (DEC-059, 2026-06-14); OBSERVABILITY §28.13 cross-update required (see checklist). §30.13 fourteen-item implementation checklist: 6× P0/M8 (sso-browser.ts implementation, ESLint rule, CI test, DEC-030 event registration + integration test, AL-SSO-WEB-01 PagerDuty config, OBSERVABILITY §26.8 row addition), 6× P1/M10–M10+30 (AASA deploy + iOS verification, assetlinks.json deploy + Android adb verification, redirect URI registration in all IdPs + CSM onboarding update, B2C pre-enablement 409 guard, SSO-WEB-E-001 first collection, SSO-WEB-E-002 first collection), 2× P2/M11-Annual (SSO_CLIENT_CONFIG / ENTERPRISE_ONBOARDING update, annual API review). Cross-references: `docs/OBSERVABILITY.md §28.13` (OQ-MOBILE-03 source — update to 🟢 Resolved); `docs/OBSERVABILITY.md §26.8` (AL-SSO-WEB-01 registration in sso_browser_security subsection — P0 checklist item 6); `docs/ENTERPRISE.md §Enterprise SSO` (enterprise SSO feature scope — mobile SSO is enterprise tier only); `docs/AUDIT_LOG_SCHEMA.md §6.SSO-Mobile` (two new events to register — P0 checklist item 4); `docs/INCIDENT_RESPONSE.md R-05` (MOBILE-CHAIN-01 violation → R-05); `docs/INCIDENT_RESPONSE.md R-08` (supply-chain concern if AL-SSO-WEB-01 fires due to SDK injection); `docs/SSO_SCIM_IMPLEMENTATION.md §1.2` (SP-initiated OIDC + SAML flows — §30 adds the mobile-client perspective to the same protocol flows); `docs/ENTERPRISE_ONBOARDING.md` (CSM onboarding checklist — mobile redirect URI and B2C OIDC guidance to be added); `docs/DECISION_LOG.md DEC-059`. Privacy floor: `sso.mobile_browser_mode_set` payload contains tenant_id + config state only — no employee user_id, name, email, or health data; `sso.mobile_webview_blocked` contains attempted_url_hash (SHA-256 of IdP URL, first 32 chars only) — the full URL is never stored; no personal data in either event. Owner: security-engineer + platform-engineer + enterprise-architect.*

*v2.1 additions (2026-06-14): §29 OQ-SSO-28.2 Resolution — `tenant_users_role_history` Retention Period (DEC-051). Closes the sole remaining P1 open question from §28.9 (v2.0, 2026-06-14). Decision: **7-year retention** adopted for `tenant_users_role_history`, effective from `changed_at`. Three grounds documented: (1) audit-chain continuity — `scim.user_updated` DEC-030 events are retained 7yr; purging the table at 3yr creates a verifiable SOC 2 CC6.3 evidence gap where the chain references a role change but the table row has been deleted (DATA_MODEL §33.8 rejection rationale confirmed); (2) legal obligation — Ukrainian Tax Code Art. 44 (7yr business records) and EU audit-record doctrine (access-control records should be retained at least as long as the data they protect access to); (3) SOC 2 CC6.3/CC6.6 evidence continuity across multi-cycle observation windows. GDPR Art. 4(1) analysis: the `user_id` + `changed_at` + role history combination qualifies as personal data because it enables reconstruction of an employee's access history; legal basis for 7yr retention is GDPR Art. 17(3)(b) (legal obligation overrides erasure right); `user_id` is pseudonymised on user erasure per §28.4 / DATA_MODEL §33.7; full hard-delete occurs at 7yr `changed_at` boundary via pg_cron job 32. DPA Annex B update required: enterprise DPA template (§14.3.2) must add a "role change logs: 7yr" row to the retention schedule before DPA execution with any EU enterprise customer. Consumer privacy policy: no change — `tenant_users_role_history` is enterprise-only. New pg_cron job 32 `turh_retention_purge` (04:00 UTC daily): hard-deletes pseudonymised rows past 7yr; safety gate (`user_id IS NULL AND user_id_pseudonym IS NOT NULL`) prevents accidental deletion of active-user records; companion AL-TURH-01 (P2, Slack `#compliance-alerts`) fires when any row with `user_id IS NOT NULL` exceeds the 7yr window (indicates erasure pipeline miss). New SOC 2 evidence artefact TURH-RET-E-001: annual `#turh-retention-purge` cron run log + AL-TURH-01 zero-fire attestation filed to `compliance/evidence/scim-role/`; covers CC6.3 retention-policy-as-evidence and CC4.1 ongoing monitoring. `docs/DATA_MODEL.md §33.8` retention table updated: 7-year row from "Proposed" → 🟢 Resolved. `docs/DATA_MODEL.md §33.10` checklist item 4 updated to 🟢 Done. Header updated v1.9 → v2.1 (v2.0 was §28). Cross-references: `docs/DATA_MODEL.md §33` (canonical DDL and erasure path — §33.7 pseudonymisation path; §33.8 retention table now resolved; §33.10 checklist item 4 closed); `docs/DATA_MODEL.md §34` (DEC-050 `form_role_enum` — column types for `old_role`/`new_role` in `tenant_users_role_history`); `docs/AUDIT_LOG_SCHEMA.md §6.SCIM-Lifecycle` (`scim.user_updated` 7yr retention anchor); `docs/ENTERPRISE.md §Privacy floor for enterprise` (HR never sees individual user data — §29.4 clarifies role history is admin-only, not HR-accessible); `docs/CRYPTOGRAPHY_POLICY.md §5` (ERASURE_PSEUDONYM_SALT — key used in pg_cron job 32 safety gate); `docs/OBSERVABILITY.md §26` (AL-SCIM-05 taxonomy — AL-TURH-01 is a new parallel alert using its own prefix); `docs/INCIDENT_RESPONSE.md R-05` (HMAC chain break — pg_cron stale-active-user detection escalates to compliance-officer, not P0 IR, because it does not breach the chain itself); `docs/SOC2_READINESS.md §CC6.3` (SCIM-ROLE-E-001 evidence table — TURH-RET-E-001 extends this evidence set); `docs/DECISION_LOG.md DEC-051`. Owner: compliance-officer + enterprise-architect + security-engineer.*

*v1.8 additions (2026-06-04): §26 API Key Authentication Security — SCIM IP Scope, API Key IP Enforcement & Rotation Policy. Closes OQ-SSO-25.1 (🟡 P1 → 🟢 Resolved) and OQ-SSO-25.3 (🟡 P1 M5 → 🟢 Resolved) from §25.13. TOC updated to add §26. Header updated from v1.7 to v1.8. §26.1 purpose and scope: two P1 open questions from §25 resolved; privacy floor (client_ip_hash via IP_HASH_SALT in all DEC-030 events; no plaintext IP in audit chain). §26.2 credential architecture disambiguation: three credential types (JWT 1h, SCIM bearer long-lived provisioning, tenant API key long-lived integration); §26 covers the two non-JWT types that were not covered by §25 IP enforcement. §26.3 OQ-SSO-25.1 resolution — SCIM IP scope: option (c) selected (separate flag, default off); migration 0052_scim_ip_allowlist.sql adding `scim_ip_enforcement_enabled BOOLEAN DEFAULT false` and `scim_ip_allowlist_config JSONB DEFAULT NULL` to tenant_sso_configs; chk_scim_ip_allowlist_required CHECK constraint; `enforceScimIpAllowlist()` branch in ip-allowlist.ts (SCIM routes bypass general allowlist; use scim_ip_allowlist_config only when scim_ip_enforcement_enabled = true); admin dashboard "Directory Sync IP Restriction" UI (collapsible, Enterprise-gated, import buttons for Okta/Azure pre-populated CIDRs, warning banner about IdP IP range volatility). §26.4 OQ-SSO-25.3 resolution — API key IP enforcement: formalised tenant_api_keys table schema (UUID PK, key_hash HMAC-SHA256, key_preview last-8-chars, label, created_by, last_used_at, expires_at, revoked_at, ip_enforcement_enabled BOOLEAN DEFAULT false, ip_allowlist_config JSONB, scopes TEXT[] DEFAULT '{reporting:read}'; RLS for form_api; partial unique index on key_hash WHERE revoked_at IS NULL); api-key-auth.ts updated to call enforceApiKeyIpAllowlist() using shared checkCidrList() utility from §25 (no new dependency); scope enforcement before route handler; last_used_at updated via waitUntil; error response omits all IP/allowlist detail to prevent enumeration. §26.5 rotation policy: mandatory triggers (age ≥ 365d amber, ≥ 730d red, suspected compromise same-day, offboarding 5 business days, `admin:write` scope quarterly 90d); 24-hour overlap window (same as SCIM token rotation §16.6); admin dashboard age-alert pill states (0–364d none, 365–729d amber, 730d+ red, admin:write >90d amber). §26.6 nine DEC-030 HMAC-chained events (all HIGH, 7yr): api_key.created, api_key.rotated, api_key.revoked, api_key.ip_enforcement_enabled, api_key.ip_enforcement_disabled, api_key.ip_blocked (client_ip_hash only, Queues bulk dedup), scim.ip_enforcement_enabled, scim.ip_enforcement_disabled, scim.ip_blocked (client_ip_hash only); HMAC chain requirement: api_key.rotated must be followed by api_key.revoked (old key) within 26h or AL-APIKEY-02 fires. §26.7 four alerting rules: AL-APIKEY-01 P1 (>10 ip_blocked per key in 10 min → compromise attempt or misconfiguration), AL-APIKEY-02 P1 (rotation overlap not cleaned up within 26h → cron failure), AL-APIKEY-03 P1 (revoked with reason=compromise → trigger R-16), AL-SCIM-IP-01 P2 (SCIM ip_blocked >5 in 15 min AND provisioning success <50% → misconfigured allowlist). §26.8 SOC 2 evidence mapping CC6.1/CC6.2/CC6.4/CC6.8/CC7.2 with five artefacts APIKEY-E-001 through APIKEY-E-005. §26.9 OQ-SSO-25.1 and OQ-SSO-25.3 formally resolved. §26.10 two new open questions: OQ-SSO-26.1 (mandatory IP enforcement for admin:write keys P2, evaluated before first admin:write key issued), OQ-SSO-26.2 (expires_at hard enforcement vs soft alerts P1, decision before SOC 2 observation period start). §26.11 fourteen-item implementation checklist: 8× P0 M5 (tenant_api_keys migration, scim_ip_allowlist migration, api-key-auth.ts update, ip-allowlist.ts SCIM branch, DEC-030 event registry, admin dashboard panels, API_KEY_HASH_SECRET rotation schedule, pg_cron overlap cleanup), 4× P1 M5-M6 (alerting, evidence collection, OQ-SSO-26.2 decision, DATA_MODEL cross-reference), 2× P2 M6. Owner: enterprise-architect + security-engineer + platform-engineer + compliance-officer.*

---

## §31 OQ-SSO-24.1 + OQ-SSO-24.2 Resolution — `pam-db-proxy` Architecture: Supabase Edge Function & Direct Postgres Session Connection (DEC-060)

**Owner:** security-engineer + platform-engineer + enterprise-architect
**SOC 2 scope:** CC6.1, CC6.7
**Last updated:** 2026-06-15
**Closes:** OQ-SSO-24.1 (P1, Before M4 deploy — proxy placement: Hyperdrive vs. Edge Function) and OQ-SSO-24.2 (P0, Before M4 deploy — PgBouncer session mode for PAM pool)
**Decision:** DEC-060 — Supabase Edge Function adopted for `pam-db-proxy`; direct Postgres session connection (port 5432) adopted for PAM operations — PgBouncer bypassed entirely.

---

### 31.1 Purpose and Scope

§24 introduced the Privileged Access Management (PAM) architecture and specified `pam-db-proxy` as the sole path through which `form_admin` Postgres privilege is exercised. §24 left two implementation questions open:

- **OQ-SSO-24.1** (P1): Should `pam-db-proxy` be implemented as a Supabase Edge Function or as a Cloudflare Worker using Hyperdrive for its Postgres connection? The original design diagram in §24.2 shows "Supabase Edge Function" but the choice was not formally analysed or decided.
- **OQ-SSO-24.2** (P0): The `SET ROLE form_admin` + `RESET ROLE` pattern in §24.2 is session-scoped and incompatible with PgBouncer transaction mode (port 6543), which is the default Supabase connection pooler. The OQ asked whether a dedicated PgBouncer session-mode pool is required, and whether the Supabase plan supports it without impacting shared pool limits.

This section formally resolves both questions, documents the security analysis, and updates the relevant §24 checklist items.

**Out of scope:** The `pam-elevation-service` Cloudflare Worker (§24.2 — manages approval workflow, KV sessions, DEC-030 events) is unaffected by either decision. Break-glass connection handling (§24.4.2 `BREAK_GLASS_DB_URL`) is a separate secret; the decision here applies to normal PAM sessions only. No new DEC-030 event types are introduced by this section — the §24.6 event taxonomy covers the PAM session lifecycle completely.

**Privacy floor:** No employee user_id, name, email, or health data appears in any PAM database connection variable, Postgres GUC, or audit log row beyond what §24.6 already defines. The `pam_session_id` UUID field is the only FORM-internal identifier in the connection context.

---

### 31.2 OQ-SSO-24.1 Resolution — `pam-db-proxy` Host: Supabase Edge Function Adopted

#### 31.2.1 Options Analysis

| Dimension | Supabase Edge Function | Cloudflare Worker + Hyperdrive |
|---|---|---|
| **Network path for privileged connection** | Intra-VPC: Edge Function → Postgres within Supabase's managed infrastructure. No network traversal outside Supabase's boundary for the privileged connection. | Cross-provider: Cloudflare Worker → Hyperdrive (Cloudflare) → Supabase Postgres public endpoint. Two provider boundaries crossed for every privileged query. |
| **Trust boundary for `form_admin`** | `form_admin` credentials exist only within Supabase's trust boundary (Postgres + Edge Function runtime). The credential never crosses a provider boundary. | `form_admin` credentials must be stored as a Cloudflare Workers Secret and transmitted from Cloudflare infrastructure to Supabase's public Postgres endpoint on every PAM query. |
| **`SET ROLE form_admin` compatibility** | Edge Function opens a direct Postgres connection (port 5432, session mode). `SET ROLE` is session-scoped and fully reliable. | Hyperdrive's default pooling mode is transaction mode (equivalent to PgBouncer transaction mode). `SET ROLE` in transaction mode is NOT session-sticky — the role may not be the same across connection checkout boundaries. Hyperdrive does not expose a session-mode configuration option as of June 2026. |
| **Connection latency** | Edge Function → Postgres: intra-VPC, typically < 5 ms round-trip within Supabase's managed compute boundary. | Worker → Hyperdrive → Postgres: Cloudflare PoP → Hyperdrive cluster → Supabase Postgres public endpoint. Two external hops add 20–50 ms per query under typical EU-West conditions. For a PAM session that may execute 5–20 queries, cumulative overhead is 100–1,000 ms vs. < 100 ms for Edge Function. |
| **Third-party intermediary in privileged path** | None — two components (Edge Function + Postgres) both operated by Supabase. | Hyperdrive is a third Cloudflare-managed component in the privileged data path, increasing attack surface for compromise or misconfiguration. |
| **mTLS profile** | See §31.2.2 | See §31.2.2 |
| **Operational complexity** | `pam-db-proxy` is a single Supabase Edge Function with one environment variable: the direct Postgres session connection string. | `pam-db-proxy` would require a Cloudflare Worker, Hyperdrive binding configuration, Hyperdrive-specific connection pool settings, and cross-provider secret management for `form_admin` credentials. |
| **Session-mode availability** | Direct Postgres connection (port 5432) is available on all Supabase paid plans and bypasses PgBouncer entirely. | Session mode is not a native Hyperdrive feature; workarounds (e.g., Hyperdrive `disableCache: true` + server-reset-query) do not guarantee `SET ROLE` session affinity under concurrent load. |

#### 31.2.2 mTLS Profile Comparison

**Supabase Edge Function path:**

1. `pam-elevation-service` (Cloudflare Worker) → `pam-db-proxy` endpoint: Cloudflare Access mTLS service-to-service (Cloudflare mTLS with a client certificate scoped to the `internal-tools` audience). This hop is already specified in §24.2 and §24.8 CC6.7 evidence. The Edge Function URL is not on the public API surface — it is only accessible via the Cloudflare Access audience.
2. `pam-db-proxy` (Edge Function) → Postgres: TLS enforced by `sslmode=require` in the Supabase connection string. This is an intra-VPC connection within Supabase's managed infrastructure; the TLS protects against any potential lateral movement within the VPC. Supabase issues the server certificate; FORM trusts Supabase's PKI for this connection. There is no external CA cross-validation to manage.

**Cloudflare Worker + Hyperdrive path (rejected):**

1. `pam-elevation-service` → `pam-db-proxy` (Worker): standard Cloudflare Worker-to-Worker communication; no mTLS between Workers in Cloudflare's runtime.
2. Worker → Hyperdrive: Cloudflare internal connection (Cloudflare-managed TLS); FORM has no visibility into the Hyperdrive ↔ Postgres TLS negotiation.
3. Hyperdrive → Postgres: Hyperdrive connects to Supabase's public Postgres endpoint using the connection string stored in Hyperdrive's configuration. Supabase enforces `sslmode=require`; however, the TLS termination occurs at Supabase's public boundary rather than at an intra-VPC endpoint. The `form_admin` credential is transmitted from Cloudflare's infrastructure to Supabase's network on every new Hyperdrive pool connection — a higher exposure surface than the intra-VPC Supabase Edge Function path.

**Assessment:** The Supabase Edge Function path provides an equivalent or superior mTLS posture for the cross-provider segment (Cloudflare Access mTLS on the `pam-elevation-service` → Edge Function call) while eliminating the Hyperdrive-as-intermediary exposure for the privileged Postgres connection.

#### 31.2.3 Decision: Supabase Edge Function Adopted

**DEC-060 resolution for OQ-SSO-24.1:** `pam-db-proxy` is implemented as a Supabase Edge Function. The Cloudflare Worker + Hyperdrive option is rejected on four independently sufficient grounds:

1. **`SET ROLE` incompatibility.** Hyperdrive uses transaction-mode pooling. `SET ROLE form_admin` is session-scoped; its effect is not guaranteed to persist across connection checkouts in a transaction-mode pool. This is not a configuration gap — it is a fundamental PgBouncer transaction-mode constraint. No documented Hyperdrive workaround eliminates this risk for FORM's PAM pattern.
2. **`form_admin` credential exposure.** Hyperdrive requires the `form_admin` Postgres credential to be stored in Cloudflare Workers Secrets and transmitted from Cloudflare's infrastructure to Supabase's network on every new pool connection. The Supabase Edge Function keeps this credential within Supabase's trust boundary — the provider that already holds the database and manages its PKI.
3. **Cross-provider network hop for privileged queries.** Every query in a PAM session traverses the Cloudflare → Supabase public network boundary with Hyperdrive. The Supabase Edge Function executes queries intra-VPC, minimising exposure time and reducing latency for multi-query PAM sessions.
4. **Operational surface.** The Edge Function requires one environment variable (the direct Postgres session connection string). The Hyperdrive option requires: Cloudflare Worker scaffolding, Hyperdrive binding, Hyperdrive configuration, Workers Secret for `form_admin`, and per-PAM-session connection lifecycle management — all without `SET ROLE` session-affinity guarantees.

The §24.2 architecture diagram already shows "Supabase Edge Function" — this section formally confirms that design and documents the analysis behind it.

---

### 31.3 OQ-SSO-24.2 Resolution — PAM Postgres Connection Mode: Direct Session Connection (Port 5432) Adopted

#### 31.3.1 The PgBouncer Transaction Mode Problem

Supabase exposes two connection endpoints:

| Endpoint | Port | Mode | `SET ROLE` support |
|---|---|---|---|
| PgBouncer pooler (shared) | 6543 | **Transaction mode** | ❌ Not safe — `SET ROLE` is session-scoped; different transactions in the same logical "session" may execute on different Postgres backend connections |
| Direct Postgres (session) | 5432 | **Session mode** (native Postgres connection) | ✅ Reliable — the connection maintains full Postgres session state, including `SET ROLE form_admin` and `SET app.pam_session_id` GUC |

The `form_api` Supabase service-role connection (used by Cloudflare Workers for all normal application queries) uses the port 6543 PgBouncer pooler in transaction mode — this is correct for stateless application queries. The PAM design in §24.2 requires `SET ROLE form_admin` followed by one or more privileged queries followed by `RESET ROLE`, which is inherently session-scoped. Using the port 6543 pooler for this pattern creates a silent data integrity risk: `RESET ROLE` may execute on a different Postgres backend than `SET ROLE`, leaving an elevated-privilege connection in the pool that another application request could inadvertently acquire.

#### 31.3.2 Direct Connection (Port 5432) Analysis

The direct Postgres connection string uses Supabase's session-mode endpoint:

```
SUPABASE_PAM_DB_URL=postgresql://postgres.[project-ref]:[password]@aws-0-eu-west-2.pooler.supabase.com:5432/postgres?sslmode=require
```

Properties of the direct connection:

- **Native session semantics.** Every `pam-db-proxy` invocation opens one dedicated Postgres connection. `SET ROLE form_admin` is guaranteed to persist for the life of that connection. `RESET ROLE` is guaranteed to execute on the same backend.
- **No PgBouncer.** The connection bypasses port 6543 entirely. There is no `server_reset_query` dependency — `RESET ROLE` is called explicitly by the Edge Function before closing the connection (belt-and-suspenders; the `server_reset_query = RESET ROLE; RESET app.pam_session_id;` note in §24.2 invariant #4 is retained as a defensive check for any future connection pooler layer, but is not relied upon for correctness in the Edge Function path).
- **`sslmode=require`.** Supabase enforces TLS on the session endpoint; no plaintext connection is possible.
- **`form_system` base role.** The Edge Function authenticates to Postgres as `form_system` (the application service role with least-privilege access) and executes `SET ROLE form_admin` only after validating the PAM session. On `RESET ROLE`, access returns to `form_system` level automatically. The `form_admin` role is never the connection's login role.

#### 31.3.3 Connection Limit Analysis

A common concern with direct Postgres connections (vs. PgBouncer) is connection count exhaustion. This analysis confirms the concern does not apply to the PAM use case.

**Supabase connection capacity:** The Supabase Pro plan (FORM's current plan) supports up to 60 direct Postgres connections. The Supabase Pro Plus / Team plan supports up to 200. The shared `form_api` application uses the port 6543 PgBouncer pooler and does not consume direct connections.

**PAM connection ceiling:**

| Factor | Limit | Basis |
|---|---|---|
| PAM-eligible engineers | ≤ 5 (founder + max 4 senior engineers per `cloudflare/access/break-glass-policy.tf` named list cap) | §24.4.1 named identity list max 5 |
| Concurrent `read_only` sessions per admin | 1 (enforced by `pam:by_admin:{id}` secondary KV index — `pam-elevation-service` blocks a second elevation if an active session exists for the same admin) | §24.5 KV schema |
| Concurrent `read_write` / `destructive` sessions | 1 system-wide (dual-person approval for `destructive` ensures only one `destructive` session exists at a time; `read_write` is per-admin) | §24.3.2 |
| Maximum concurrent direct PAM connections | **≤ 5** (one per PAM-eligible engineer, all simultaneously elevated — an extreme scenario not expected during normal operations) | Combined constraint |
| Break-glass direct connections | 1 (non-renewable 2h session; separate `BREAK_GLASS_DB_URL`) | §24.4.2 |

**Conclusion:** At FORM's current team scale, PAM consumes at most 5–6 direct Postgres connections simultaneously, out of a 60-connection pool. The remaining 54+ connections are available for Supabase Edge Functions and any future direct-connection consumers. No Supabase plan upgrade is required. If FORM scales to a team where > 10 PAM-eligible engineers exist, the PAM direct-connection ceiling should be re-evaluated against the Supabase plan in effect at that time.

#### 31.3.4 Decision: Direct Postgres Session Connection (Port 5432) Adopted

**DEC-060 resolution for OQ-SSO-24.2:** `pam-db-proxy` connects to Postgres using the direct session-mode endpoint (port 5432), not the PgBouncer transaction-mode pooler (port 6543). This eliminates the `SET ROLE` session-affinity risk and requires no dedicated PgBouncer session-mode pool — the question of whether Supabase supports a separate session-mode PgBouncer pool is moot because the direct connection bypasses PgBouncer entirely.

The `SUPABASE_PAM_DB_URL` environment variable (Supabase Edge Function secret) is distinct from `SUPABASE_SERVICE_ROLE_KEY` (used by Cloudflare Workers for application queries via port 6543). The two secrets serve different purposes and must not be conflated:

| Secret | Used By | Port | Mode | Role |
|---|---|---|---|---|
| `SUPABASE_SERVICE_ROLE_KEY` | Cloudflare Workers (application tier) | 6543 | Transaction (PgBouncer) | `form_api` / `form_system` |
| `SUPABASE_PAM_DB_URL` | `pam-db-proxy` Edge Function | 5432 | Session (direct Postgres) | `form_system` → `SET ROLE form_admin` |
| `BREAK_GLASS_DB_URL` | Break-glass Worker (§24.4.2) | 5432 | Session (direct Postgres) | `form_break_glass` (direct login role) |

---

### 31.4 Updated `pam-db-proxy` Connection Specification

The §24.2 step-3 annotation "Open Postgres connection as form_system (connection pool)" is amended. The parenthetical "(connection pool)" referred to a future PgBouncer session-mode pool that this section eliminates. The authoritative specification is:

```
Step 3:  Open direct Postgres session connection using SUPABASE_PAM_DB_URL
         (port 5432, sslmode=require, login role: form_system)
Step 4:  SET ROLE form_admin;
Step 5:  SET app.pam_session_id = '<pam_session_id>';
Step 6:  Execute requested query (tenant_id injected for all cross-tenant ops)
Step 7:  RESET ROLE;         -- explicit, in finally block
Step 8:  RESET app.pam_session_id;  -- explicit, in finally block
Step 9:  Close connection    -- do not return to pool; each PAM session gets a fresh connection
```

**Why Step 9 closes rather than pools:** The `pam-db-proxy` Edge Function opens one connection per invocation and closes it when the PAM query completes. Connection reuse across PAM sessions is not implemented. This is intentional:

1. **Audit isolation.** Each PAM session starts with a clean Postgres connection that has no residual session state from a previous PAM session. `app.pam_session_id` GUC cannot leak between sessions.
2. **PAM session TTL alignment.** PAM sessions range from 15 minutes to 4 hours. A "warm" connection pool would hold open Postgres connections for this duration with no queries — wasteful and unnecessary given the ≤ 5 concurrent connection ceiling.
3. **Simplicity.** Edge Functions are serverless and designed for short-lived execution; managing a persistent connection pool within an Edge Function is an anti-pattern.

The §24.2 design invariant #4 now reads: `RESET ROLE` and `RESET app.pam_session_id` are called explicitly in a `finally` block before the Edge Function returns. The `server_reset_query` annotation is retained as defensive documentation only — it describes the pattern that *would* be required if a connection pool were introduced in the future.

---

### 31.5 Environment Variable Specification

No new environment variables are introduced beyond what §24 already references. The canonical `SUPABASE_PAM_DB_URL` secret is specified here for implementation clarity:

**Supabase Edge Function secret: `SUPABASE_PAM_DB_URL`**

```
Format:  postgresql://postgres.[PROJECT_REF]:[FORM_SYSTEM_PASSWORD]@aws-0-[REGION].pooler.supabase.com:5432/postgres?sslmode=require
Port:    5432  (direct session mode — NOT 6543 which is PgBouncer)
Role:    form_system (least-privilege application role; SET ROLE form_admin applied after PAM session validation)
SSL:     sslmode=require (Supabase enforces this; connection rejected if server presents no certificate)
```

**Rotation:** `SUPABASE_PAM_DB_URL` follows the same rotation policy as `SUPABASE_SERVICE_ROLE_KEY` per `docs/CRYPTOGRAPHY_POLICY.md §5`: annual rotation scheduled, immediate rotation on suspected compromise. The password component is the `form_system` Postgres role password; rotation requires a `\password form_system` operation in a `form_admin` session (via break-glass or a new `read_write` PAM session by the compliance-officer) followed by updating the Supabase Edge Function secret.

**CRYPTOGRAPHY_POLICY.md §5 update required (P0, M4):** Add `SUPABASE_PAM_DB_URL` to the key inventory table with:
- Type: Postgres connection string (session credential)
- Rotation policy: Annual; immediate on suspected compromise
- Owner: devops-lead
- Storage: Supabase Edge Function Secrets (encrypted at rest by Supabase)

---

### 31.6 SOC 2 Evidence Update

§24.8 CC6.7 evidence states: "The `pam-db-proxy` Supabase Edge Function communicates with Postgres over TLS (Supabase enforces `sslmode=require`)." This section confirms and extends that statement:

| SOC 2 Criterion | Updated Control Statement | Evidence |
|---|---|---|
| **CC6.7 — Transmission confidentiality** | `pam-db-proxy` connects to Postgres via the direct session endpoint (port 5432, `sslmode=require`). The connection uses a native Postgres TLS channel within Supabase's managed infrastructure. No privileged connection traverses the public internet without TLS. The `SUPABASE_PAM_DB_URL` secret is stored in Supabase Edge Function Secrets (encrypted at rest; no plaintext in version control or Cloudflare environment). | Existing CC6-E-PAM-003 (Supabase TLS connection logs) updated: include screenshot of `SUPABASE_PAM_DB_URL` in Supabase Edge Function Secrets console (confirming port 5432 and `sslmode=require`); no new artefact required. |
| **CC6.1 — Logical access controls** | `form_admin` Postgres credentials exist only within Supabase's trust boundary (`SUPABASE_PAM_DB_URL` stored in Supabase Edge Function Secrets; never in Cloudflare Workers Secrets or any cross-provider store). The privileged connection does not traverse any third-party intermediary (Hyperdrive rejected). | Existing CC6-E-PAM-001 + CC6-E-PAM-003 unchanged in scope; this section provides the architectural basis for the CC6.7 narrative that was implicit in §24. |

**No new evidence artefacts are required.** The existing CC6-E-PAM-001 through CC6-E-PAM-008 taxonomy from §24.8 remains complete. §31 provides the architectural rationale that auditors may request when reviewing the `pam-db-proxy` trust boundary and connection mode.

---

### 31.7 §24.9 / §24.10 Cross-Updates

**§24.9 open question table:**

| ID | Previous Status | New Status |
|---|---|---|
| OQ-SSO-24.1 | 🟡 P1 Open — pam-db-proxy: Hyperdrive vs. Edge Function | 🟢 **Resolved — DEC-060 (2026-06-15)**: Supabase Edge Function adopted. §31.2. |
| OQ-SSO-24.2 | 🟡 P0 Open — PgBouncer session mode for PAM pool | 🟢 **Resolved — DEC-060 (2026-06-15)**: Direct Postgres session connection (port 5432) adopted; PgBouncer bypassed entirely. No dedicated session-mode pool required. §31.3. |

**§24.10 checklist cross-updates:**

- **Item 4** (P0, M4): "Resolve OQ-SSO-24.2 (PgBouncer session mode) before deploy" — **🟢 Done (DEC-060, 2026-06-15).** Scaffold `pam-db-proxy` using `SUPABASE_PAM_DB_URL` (port 5432, direct session connection). No PgBouncer session pool configuration needed.
- **Item 10** (P1, M5): "Resolve OQ-SSO-24.1 (proxy placement decision) before M5 freeze" — **🟢 Done (DEC-060, 2026-06-15).** Supabase Edge Function confirmed as the host; M5 freeze action is to verify that `pam-db-proxy` is deployed and CC6-E-PAM-003 evidence updated to show port 5432 connection.

---

### 31.8 OQ-SSO-24.1 + OQ-SSO-24.2 Status

| Status | 🟢 **Resolved — DEC-060 (2026-06-15)** |
|---|---|
| **OQ-SSO-24.1 Decision** | Supabase Edge Function adopted for `pam-db-proxy`; Cloudflare Worker + Hyperdrive rejected |
| **OQ-SSO-24.1 Key grounds** | `SET ROLE` incompatibility in Hyperdrive transaction mode; `form_admin` credential stays within Supabase trust boundary; intra-VPC network path eliminates cross-provider hop for privileged connection; mTLS profile equivalent or superior |
| **OQ-SSO-24.2 Decision** | Direct Postgres session connection (port 5432) via `SUPABASE_PAM_DB_URL`; PgBouncer bypassed entirely; no dedicated session-mode pool required |
| **OQ-SSO-24.2 Key grounds** | Session-mode semantics guaranteed natively; max 5–6 concurrent direct PAM connections well within Supabase Pro plan capacity (60 direct connections); `form_system` login role + `SET ROLE form_admin` pattern fully reliable |
| **§24.2 annotation update** | "(connection pool)" parenthetical removed; "direct Postgres session connection" is the authoritative term |
| **§24.10 checklist updates** | Items 4 and 10 marked 🟢 Done |

---

### 31.9 Implementation Checklist

#### P0 — Before `pam-db-proxy` M4 deploy

| # | Task | Owner | Priority | Milestone | Status |
|---|---|---|---|---|---|
| 1 | Provision `SUPABASE_PAM_DB_URL` as a Supabase Edge Function secret: construct the direct session connection string (port 5432, `sslmode=require`, `form_system` role). Confirm the string uses port 5432 (not 6543). Test connectivity from the Edge Function runtime with a `SELECT 1` query using `form_system` privileges (no `SET ROLE` at this step). | devops-lead | **P0** | M4 | [ ] |
| 2 | Add `SUPABASE_PAM_DB_URL` to `docs/CRYPTOGRAPHY_POLICY.md §5` key inventory table per §31.5 specification: type, rotation policy (annual + immediate on compromise), owner (devops-lead), storage location (Supabase Edge Function Secrets). | devops-lead + compliance-officer | **P0** | M4 | [ ] |
| 3 | Implement `supabase/functions/pam-db-proxy/index.ts`: open connection using `SUPABASE_PAM_DB_URL`; validate `pam_session_id` against `PAM_KV` (Cloudflare Access JWT passed as caller credential); enforce hard expiry check on `expires_at`; execute `SET ROLE form_admin` + `SET app.pam_session_id`; run query; call `RESET ROLE` and `RESET app.pam_session_id` in `finally` block; close connection. Do NOT return the connection to any pool. | platform-engineer | **P0** | M4 | [ ] |
| 4 | Integration test in staging: (a) elevate via `pam-elevation-service` to `read_only`; (b) execute a `SELECT` via `pam-db-proxy`; (c) verify `pg_audit` logs show `role: form_admin` and `application_name: pam-db-proxy`; (d) verify `app.pam_session_id` appears in the `audit_log_events` row emitted by the audit trigger (§24.2 invariant #3); (e) verify connection is closed after the query (no lingering `form_admin` sessions in `pg_stat_activity`). | platform-engineer + security-engineer | **P0** | M4 | [ ] |
| 5 | Update CC6-E-PAM-003 evidence artefact spec (§24.8): include screenshot of `SUPABASE_PAM_DB_URL` in Supabase Edge Function Secrets console showing port 5432 in the connection string. File at `compliance/evidence/pam/CC6-E-PAM-003_v2.md` (versioned update to existing artefact; both v1 and v2 retained). | compliance-officer | **P0** | M5 (30 days post M4 go-live) | [ ] |

#### P1 — Before M5 architecture freeze

| # | Task | Owner | Priority | Milestone | Status |
|---|---|---|---|---|---|
| 6 | Confirm Supabase plan direct connection capacity: run `SELECT count(*) FROM pg_stat_activity WHERE backend_type = 'client backend'` in staging under simulated PAM load (3 concurrent PAM sessions). Confirm total connection count remains below 20 (well under Pro plan 60-connection ceiling). Document result in `compliance/evidence/pam/PAM-CONN-E-001_M5.md`. | devops-lead | **P1** | M5 | [ ] |
| 7 | Implement `RESET ROLE` + `RESET app.pam_session_id` in a TypeScript `finally` block within `pam-db-proxy`. Write a chaos test that intentionally throws an exception mid-query and confirms that (a) `RESET ROLE` fires before connection closure; (b) the abandoned connection shows `role: form_system` (not `form_admin`) in `pg_stat_activity`. | platform-engineer | **P1** | M5 | [ ] |

#### P2 — Ongoing governance

| # | Task | Owner | Priority | Milestone | Status |
|---|---|---|---|---|---|
| 8 | If FORM's engineering team grows beyond 10 PAM-eligible engineers, re-evaluate the 5–6 concurrent PAM connection ceiling per §31.3.3. If a dedicated connection pool becomes necessary, evaluate Supabase's dedicated pooler option (available on Team plan) or a lightweight PgBouncer instance in session mode. Document decision in `docs/DECISION_LOG.md`. | devops-lead + enterprise-architect | **P2** | When engineering team > 10 | [ ] |
| 9 | Annual review: confirm that Hyperdrive has not introduced a native session-mode option that would change the OQ-SSO-24.1 analysis. If it has, re-evaluate against the four rejection grounds in §31.2.3 and open a new DECISION_LOG entry if the conclusion changes. | security-engineer + platform-engineer | **P2** | Annual (M12, M24, …) | [ ] |

---

*v2.3 additions (2026-06-15): §31 OQ-SSO-24.1 + OQ-SSO-24.2 Resolution — `pam-db-proxy` Architecture: Supabase Edge Function & Direct Postgres Session Connection (DEC-060). Closes OQ-SSO-24.1 (P1, Before M4 deploy — pam-db-proxy host: Hyperdrive vs. Edge Function) and OQ-SSO-24.2 (P0, Before M4 deploy — PgBouncer session mode for PAM pool). Both from §24.9. §31.1 scopes to pam-db-proxy connection architecture only; pam-elevation-service Worker, break-glass protocol (§24.4), and §24.6 DEC-030 event taxonomy are all unchanged. §31.2 OQ-SSO-24.1 resolution: Supabase Edge Function adopted over Cloudflare Worker + Hyperdrive. Six-row options analysis: network path (intra-VPC vs. cross-provider), trust boundary for form_admin credential, SET ROLE compatibility (transaction mode ❌ vs. session mode ✅), connection latency (< 5 ms intra-VPC vs. 20–50 ms cross-provider), third-party intermediary, operational complexity. §31.2.2 mTLS profile comparison: Supabase path (Cloudflare Access mTLS for Worker → Edge Function; Supabase-managed TLS for Edge Function → Postgres intra-VPC); Hyperdrive path (no Worker-to-Worker mTLS; Hyperdrive-managed TLS to Supabase public endpoint — form_admin credential crosses provider boundary on every new pool connection). §31.2.3 four rejection grounds for Hyperdrive: (1) SET ROLE incompatibility in transaction mode — not a configuration gap but a PgBouncer architectural constraint; (2) form_admin credential must leave Supabase trust boundary if stored in Cloudflare Workers Secrets; (3) cross-provider network hop adds 20–50 ms latency per query; (4) operational complexity disproportionate to PAM access frequency. §31.3 OQ-SSO-24.2 resolution: direct Postgres session connection (port 5432) adopted; PgBouncer transaction mode (port 6543) bypassed entirely. §31.3.1 problem: PgBouncer transaction mode does not guarantee session affinity for SET ROLE — a silent data integrity risk if RESET ROLE executes on a different backend than SET ROLE. §31.3.2 direct connection properties: native session semantics, sslmode=require, form_system login role + SET ROLE form_admin after PAM validation. §31.3.3 connection limit analysis: ≤ 5 concurrent PAM-eligible engineers (§24.4.1 named list cap); 1 active session per admin (KV secondary index enforcement); max 5–6 direct PAM connections simultaneously; Supabase Pro plan supports 60 direct connections — no plan upgrade required; break-glass uses a separate BREAK_GLASS_DB_URL. §31.4 updated pam-db-proxy connection specification: nine-step sequence (open session connection → SET ROLE form_admin → SET app.pam_session_id → execute query → RESET ROLE → RESET app.pam_session_id → close connection; do not pool); rationale for "close rather than pool" (audit isolation, TTL alignment, Edge Function anti-pattern for persistent pools); §24.2 invariant #4 server_reset_query annotation retained as defensive documentation only. §31.5 SUPABASE_PAM_DB_URL secret specification: three-row credential disambiguation table (SUPABASE_SERVICE_ROLE_KEY port 6543 transaction mode form_api/form_system; SUPABASE_PAM_DB_URL port 5432 session mode form_system → SET ROLE form_admin; BREAK_GLASS_DB_URL port 5432 session mode form_break_glass); CRYPTOGRAPHY_POLICY.md §5 update required (annual rotation, devops-lead owner, Supabase Edge Function Secrets storage). §31.6 SOC 2 evidence update: CC6.7 narrative extended (no new artefacts — CC6-E-PAM-003 updated to include port 5432 screenshot); CC6.1 note on form_admin credential staying within Supabase trust boundary. §31.7 §24.9/§24.10 cross-updates: OQ-SSO-24.1 🟡 P1 → 🟢 Resolved; OQ-SSO-24.2 🟡 P0 → 🟢 Resolved; §24.10 checklist items 4 and 10 marked 🟢 Done. §31.8 status table. §31.9 nine-item implementation checklist: 5× P0/M4-M5 (SUPABASE_PAM_DB_URL provisioning, CRYPTOGRAPHY_POLICY update, Edge Function implementation, integration test, CC6-E-PAM-003 artefact update), 2× P1/M5 (connection capacity confirmation + PAM-CONN-E-001 doc, RESET ROLE chaos test), 2× P2/Annual-governance (team growth re-evaluation gate, annual Hyperdrive session-mode review). No new DEC-030 event types — §24.6 event taxonomy unchanged. Cross-references: §24.2 (pam-db-proxy step-3 annotation updated — "(connection pool)" → "direct Postgres session connection"); §24.4.2 (BREAK_GLASS_DB_URL — separate credential, unchanged); §24.5 (PAM KV schema — unchanged); §24.6 (DEC-030 events — unchanged); §24.8 (CC6-E-PAM-003 artefact — updated scope per §31.6); §24.9 (OQ-SSO-24.1/24.2 — both now 🟢 Resolved); §24.10 (items 4 and 10 — both 🟢 Done); `docs/CRYPTOGRAPHY_POLICY.md §5` (SUPABASE_PAM_DB_URL key inventory entry — P0 checklist item 2); `docs/OBSERVABILITY.md §29` (AL-PAM-BG-01 — unchanged, break-glass audit); `docs/INCIDENT_RESPONSE.md R-05` (HMAC chain — no chain events change); `docs/DECISION_LOG.md DEC-060`. Privacy floor: no employee user_id, name, email, or health data in any PAM connection variable or audit GUC; pam_session_id is a FORM-internal UUID with no personal data mapping. Owner: security-engineer + platform-engineer + enterprise-architect.*

---

## §32 OQ-SSO-23.2 · OQ-SSO-25.1 · OQ-SSO-25.3 Resolution — Magic-Link CAEP Session Coverage, SCIM IP Allowlist Scope & API Key IP Enforcement (DEC-062)

> **Closes:** OQ-SSO-23.2 (P1, Before M4 deploy — magic-link session coverage in CAEP `account-disabled` revocation path), OQ-SSO-25.1 (P1, Before first IP-allowlist enterprise customer — SCIM endpoint scope in per-tenant IP policy), OQ-SSO-25.3 (P1, M5 — API key IP allowlist enforcement). All three from §23.9 and §25.13.

---

### §32.1 Purpose and Scope

This section closes three P1 open questions that share a common theme: **coverage completeness** — ensuring that FORM's authentication controls (session revocation, IP allowlist, credential enforcement) apply uniformly to every authentication path, not just the primary SSO/JWT path.

**Scope:** Magic-link session revocation behaviour under CAEP; SCIM Worker IP allowlist flag design; API key authentication middleware IP check. **Out of scope:** The `isRevoked()` implementation for SSO JWT sessions (§22 — unchanged); the IP allowlist enforcement table for SSO flows (§25.5 — unchanged); the SCIM Worker request-handling logic (§27 — unchanged).

All three questions were marked as pre-M4 or M5 blockers. They are resolved together in DEC-062 (2026-06-16) because all three affect the same `ip-allowlist.ts` and `api-key-auth.ts` Worker modules, and the schema change for OQ-SSO-25.1 touches the same `tenant_sso_configs` ALTER TABLE migration window as the OQ-SSO-25.3 middleware update.

---

### §32.2 OQ-SSO-23.2 Resolution — Magic-Link Session Coverage in CAEP Revocation (DEC-062)

#### §32.2.1 Problem

§23.3 specifies that when a CAEP `account-disabled` event is received, FORM revokes all sessions for the affected user via `revoke:user:{tenant_id}:{user_id}` KV key (§22.5). The question in OQ-SSO-23.2 was: does `isRevoked()` check this key when validating a **magic-link session** (§10), or only when validating an SSO JWT session?

Magic-link sessions are issued via `POST /auth/magic-link` as an enterprise fallback path when SSO is unavailable (§10.2). They produce an `enterprise_sessions` row with `session_type = 'magic_link'` and generate a signed JWT identical in structure to an SSO JWT — same `tenant_id`, `user_id`, and `jti` claims.

#### §32.2.2 Decision: `isRevoked()` Covers Magic-Link Sessions — Confirmed (DEC-062)

**Decision: OQ-SSO-23.2 confirmed resolved.** Magic-link sessions are covered by `revoke:user:{tenant_id}:{user_id}` because the magic-link JWT passes through the same authentication middleware stack as SSO JWTs, and `isRevoked()` is called at the middleware level — before any session-type branching.

The key insight is that session revocation in §22 operates on the *credential* (the JWT claim pair `{tenant_id, user_id}`), not on the *session type* stored in `enterprise_sessions`. The `isRevoked()` function (§22.6) reads two KV keys:

```typescript
// src/workers/auth/middleware/session-revocation.ts (unchanged — confirmation only)
export async function isRevoked(
  env: Env,
  tenantId: string,
  userId: string,
  sessionJti: string,
): Promise<RevocationStatus> {
  // Check 1: user-level revocation (covers ALL session types for this user)
  const userKey = `revoke:user:${tenantId}:${userId}`;
  const userRevoked = await env.SESSION_REVOCATION_KV.get(userKey);
  if (userRevoked !== null) return RevocationStatus.USER_REVOKED;

  // Check 2: session-level revocation (covers specific jti)
  const sessionKey = `revoke:session:${tenantId}:${sessionJti}`;
  const sessionRevoked = await env.SESSION_REVOCATION_KV.get(sessionKey);
  if (sessionRevoked !== null) return RevocationStatus.SESSION_REVOKED;

  // Check 3: force re-auth flag (set by CAEP token-claims-change event)
  const forceReauthKey = `force_reauth:${tenantId}:${userId}`;
  const forceReauth = await env.SESSION_REVOCATION_KV.get(forceReauthKey);
  if (forceReauth !== null) return RevocationStatus.FORCE_REAUTH;

  // Check 4: account suspended flag (set by CAEP account-disabled event)
  const suspendedKey = `account_suspended:${tenantId}:${userId}`;
  const suspended = await env.SESSION_REVOCATION_KV.get(suspendedKey);
  if (suspended !== null) return RevocationStatus.ACCOUNT_SUSPENDED;

  return RevocationStatus.ACTIVE;
}
```

`isRevoked()` is called in `src/workers/auth/middleware/authenticate.ts` on every request, before the request handler reads `session_type`. Both magic-link and SSO JWTs carry `tenant_id` and `user_id` claims — both are caught by `RevocationStatus.USER_REVOKED` on the first KV lookup.

#### §32.2.3 Call-Graph Trace: Magic-Link → `isRevoked()` Path

```
POST /auth/magic-link → magicLinkHandler() → signs JWT { tenant_id, user_id, session_type: 'magic_link', jti }
  ↓
GET /api/* (magic-link JWT in Authorization header)
  ↓
authenticate() middleware [src/workers/auth/middleware/authenticate.ts]
  → verifyJwt()   — validates signature, expiry, tenant_id
  → isRevoked()   — checks KV keys:
      revoke:user:{tenant_id}:{user_id}     ← CAEP account-disabled writes this
      revoke:session:{tenant_id}:{jti}
      force_reauth:{tenant_id}:{user_id}
      account_suspended:{tenant_id}:{user_id}
  → if ACCOUNT_SUSPENDED → 401 { error: 'account_suspended' }
  ↓
request handler — session_type is read here, AFTER revocation check
```

**The call graph confirms: `isRevoked()` is invoked unconditionally at the middleware layer, before session type is examined.** A CAEP `account-disabled` event that writes `account_suspended:{tenant_id}:{user_id}` to SESSION_REVOCATION_KV will block any magic-link JWT for that user on the next request, within the TTL established in §23.3.2 (72-hour `account_suspended` TTL; immediate revocation via `revoke:user` KV key with no TTL).

#### §32.2.4 Required Integration Test (OQ-SSO-23.2 Closure Gate)

The following integration test must be added to `src/tests/auth/magic-link-revocation.test.ts` before M4 deploy as confirmation evidence:

```typescript
describe('Magic-link session — CAEP account-disabled coverage', () => {
  it('rejects magic-link JWT when revoke:user KV key is set', async () => {
    // Step 1: create a magic-link session
    const { jwt } = await createMagicLinkSession(env, testTenantId, testUserId);

    // Step 2: simulate CAEP account-disabled event writing user-level revocation key
    await env.SESSION_REVOCATION_KV.put(
      `revoke:user:${testTenantId}:${testUserId}`,
      JSON.stringify({ reason: 'caep_account_disabled', ts: Date.now() }),
    );

    // Step 3: attempt to use the magic-link JWT
    const res = await app.fetch(
      new Request('https://api.form.coach/api/me', {
        headers: { Authorization: `Bearer ${jwt}` },
      }),
      env,
    );

    expect(res.status).toBe(401);
    const body = await res.json();
    expect(body.error).toBe('account_suspended');
  });

  it('rejects magic-link JWT when account_suspended KV key is set', async () => {
    const { jwt } = await createMagicLinkSession(env, testTenantId, testUserId);
    await env.SESSION_REVOCATION_KV.put(
      `account_suspended:${testTenantId}:${testUserId}`,
      '1',
      { expirationTtl: 72 * 3600 },
    );
    const res = await app.fetch(
      new Request('https://api.form.coach/api/me', {
        headers: { Authorization: `Bearer ${jwt}` },
      }),
      env,
    );
    expect(res.status).toBe(401);
  });
});
```

This test serves as **CC6.3 evidence** (session revocation applies to all credential types) and is filed as **CC6-E-ML-001** at `compliance/evidence/sso/CC6-E-ML-001_M4.md`.

---

### §32.3 OQ-SSO-25.1 Resolution — SCIM Endpoint IP Allowlist Scope (DEC-062)

#### §32.3.1 Problem

§25.5 includes `POST /v1/scim/*` in the per-tenant IP allowlist enforcement table. The concern: major SCIM clients — Okta in particular — do not originate from stable, customer-configurable IP ranges. Okta's SCIM gateway uses shared Okta cloud infrastructure with CIDR ranges that change without customer notice. If a tenant enables IP allowlist and enters only their office CIDRs, SCIM provisioning breaks silently: the IdP's SCIM PATCH requests are blocked at the IP layer, returning 403s that the IdP may retry indefinitely or silently drop, causing provisioning stall.

Three options were documented:
- **(a)** Exclude `/v1/scim/*` from IP allowlist enforcement entirely
- **(b)** Add a separate `scim_ip_allowlist` field alongside the main IP allowlist
- **(c)** Enforce IP allowlist on SCIM only when a new `scim_ip_enforcement_enabled` flag is explicitly set (default `false`)

#### §32.3.2 Decision: Option (c) — `scim_ip_enforcement_enabled` Flag, Default Off (DEC-062)

**Rationale:**
1. **Silent breakage is the failure mode to prevent.** Most enterprise customers (Okta-primary, Azure AD-primary) run SCIM from IdP-managed cloud infrastructure with no fixed CIDRs. If IP allowlist were applied to SCIM by default, the enablement of a customer's IP allowlist would silently break provisioning — a P1 incident waiting to happen on every enterprise onboarding.
2. **Opt-in is correct for the minority case.** Some enterprise customers run a self-hosted identity proxy (e.g., on-premises Okta Agent, LDAP bridge) with a known static IP. For these customers, `scim_ip_enforcement_enabled = true` provides meaningful defence-in-depth against SCIM token theft from an off-network attacker.
3. **Option (a) is too permissive.** Removing SCIM from IP scope entirely eliminates defence-in-depth for customers who *can* provide stable CIDRs. Option (b) is operationally complex (two allowlist fields to explain to every customer).
4. **Default off is the safe default.** SCIM tokens are rotated per §26.3 and HMAC-hashed in KV per §27.3; IP enforcement is additive security, not the primary control.

#### §32.3.3 Schema Change — `tenant_sso_configs` ALTER TABLE

```sql
-- Migration: 0076_scim_ip_enforcement_flag.sql

ALTER TABLE tenant_sso_configs
  ADD COLUMN scim_ip_enforcement_enabled BOOLEAN NOT NULL DEFAULT FALSE;

COMMENT ON COLUMN tenant_sso_configs.scim_ip_enforcement_enabled IS
  'When true, SCIM Worker enforces the tenant IP allowlist on /v1/scim/* requests. '
  'Default false: most IdPs (Okta, Azure AD) use shared cloud CIDRs incompatible with '
  'static allowlists. Enable only for customers with a self-hosted IdP proxy.';

-- Verify: existing rows default to false (safe — no change to current SCIM behaviour)
SELECT COUNT(*) FROM tenant_sso_configs WHERE scim_ip_enforcement_enabled IS NULL;
-- Must return 0

-- Index: not needed (boolean column queried alongside tenant_id which is already indexed)
```

RLS: `scim_ip_enforcement_enabled` is writable by `tenant_owner` and `form_admin` roles only. `tenant_manager` may read but not write. `form_api` has SELECT (needed for SCIM Worker auth pipeline). This follows the existing RLS policy on `tenant_sso_configs` — no new policies required.

#### §32.3.4 Updated `enforceScimIpAllowlist()` Function

```typescript
// src/workers/auth/middleware/ip-allowlist.ts — SCIM-specific guard
// (Main SSO IP enforcement `enforceIpAllowlist()` in §25.5 is UNCHANGED.)

export async function enforceScimIpAllowlist(
  env: Env,
  tenantId: string,
  clientIp: string,
): Promise<void> {
  // Step 1: look up SCIM IP enforcement flag
  const config = await getScimIpEnforcementConfig(env, tenantId);

  if (!config.scim_ip_enforcement_enabled) {
    // Default path: no SCIM IP enforcement — return immediately
    return;
  }

  // Step 2: check IP against tenant's ip_allowlist (same logic as SSO path in §25.5)
  if (!config.ip_allowlist || config.ip_allowlist.length === 0) {
    // scim_ip_enforcement_enabled = true but no allowlist configured — misconfig
    // Emit advisory event and allow (do not break SCIM provisioning on misconfig)
    await emitAuditEvent(env, {
      event_type: 'scim.ip_enforcement_misconfigured',
      severity: 'STANDARD',
      tenant_id: tenantId,
      client_ip_hash: sha256(clientIp + env.IP_HASH_SALT),
    });
    return;
  }

  const allowed = config.ip_allowlist.some((cidr: string) =>
    ipInCidr(clientIp, cidr),
  );

  if (!allowed) {
    await emitAuditEvent(env, {
      event_type: 'scim.ip_allowlist_blocked',
      severity: 'HIGH',
      tenant_id: tenantId,
      client_ip_hash: sha256(clientIp + env.IP_HASH_SALT),
    });
    throw new ScimError(403, 'forbidden', 'Request IP not in tenant allowlist');
  }
}

// KV-cached config lookup (separate cache key from SSO config to avoid cross-invalidation)
async function getScimIpEnforcementConfig(
  env: Env,
  tenantId: string,
): Promise<{ scim_ip_enforcement_enabled: boolean; ip_allowlist: string[] }> {
  const cacheKey = `scim_ip_cfg:${tenantId}`;
  const cached = await env.SSO_KV.get(cacheKey, 'json');
  if (cached) return cached as { scim_ip_enforcement_enabled: boolean; ip_allowlist: string[] };

  const { data } = await env.SUPABASE.from('tenant_sso_configs')
    .select('scim_ip_enforcement_enabled, ip_allowlist')
    .eq('tenant_id', tenantId)
    .single();

  const result = {
    scim_ip_enforcement_enabled: data?.scim_ip_enforcement_enabled ?? false,
    ip_allowlist: data?.ip_allowlist ?? [],
  };

  await env.SSO_KV.put(cacheKey, JSON.stringify(result), { expirationTtl: 300 }); // 5 min
  return result;
}
```

**SCIM Worker call site** (`apps/scim-worker/src/auth.ts`) — replace the unconditional `enforceScimIpAllowlist()` call stub with the flag-gated version:

```typescript
// Before (§27.3 stub):
await enforceScimIpAllowlist(env, tenantId, clientIp);  // was: always enforced

// After (§32.3.4):
await enforceScimIpAllowlist(env, tenantId, clientIp);  // now: flag-gated internally
```

The SCIM Worker call site is **unchanged** — the flag-gating is encapsulated inside `enforceScimIpAllowlist()`. No import changes needed.

**Admin Dashboard** — the SCIM configuration panel (§16) must expose `scim_ip_enforcement_enabled` as a toggle labelled:

> **"Restrict SCIM provisioning to allowlisted IPs"** *(default: off — only enable if your IdP uses a static IP or self-hosted SCIM proxy)*

The toggle must display a warning banner when enabled: *"Your IdP's SCIM client must originate from the IPs listed above. Okta and most cloud IdPs use dynamic IPs — enabling this will break provisioning unless you have a self-hosted SCIM proxy."*

---

### §32.4 OQ-SSO-25.3 Resolution — API Key IP Allowlist Enforcement (DEC-062)

#### §32.4.1 Problem

§25.5 (IP allowlist enforcement table) covers SSO session flows and refresh endpoints. It does not cover long-lived API keys (§26, `tenant_api_keys` table in DATA_MODEL §26). An API key exfiltrated from a tenant's integration — via a misconfigured webhook endpoint, a CI/CD secret leak, or a compromised third-party integration — can be used from any IP unless IP enforcement is also applied to the API key authentication middleware.

#### §32.4.2 Decision: Extend IP Allowlist to API Key Auth Middleware (DEC-062)

**Decision:** Add an IP allowlist check to `src/workers/auth/api-key-auth.ts` using the same `enforceIpAllowlist()` function already used for SSO flows (§25.5). The check is applied only when the tenant has `ip_allowlist` configured (non-empty array in `tenant_sso_configs`) — no new per-API-key or per-key-type flag is introduced. This is deliberately conservative: if a tenant has configured an IP allowlist, it should apply to all authentication paths.

**Rejection of "API key–specific allowlist" alternative:** A separate `api_key_ip_allowlist` field on `tenant_api_keys` adds operational complexity (two allowlists to manage per tenant) and creates a security gap where a tenant administrator correctly configures the SSO allowlist but leaves the API key allowlist open. The consistent application of `tenant_sso_configs.ip_allowlist` across all auth paths is simpler to explain, audit, and support.

**Privacy note on IP hashing:** The `enforceIpAllowlist()` function (§25.5) hashes the client IP with `IP_HASH_SALT` before storing in DEC-030 events. This salt is already provisioned as a Cloudflare Workers Secret. No new secret is required.

#### §32.4.3 TypeScript Implementation

```typescript
// src/workers/auth/api-key-auth.ts — add IP allowlist check after token validation

import { enforceIpAllowlist } from './middleware/ip-allowlist';
import { emitAuditEvent } from '../audit/emit';

export async function authenticateApiKey(
  request: Request,
  env: Env,
): Promise<ApiKeyAuthResult> {
  // 1. Extract API key from Authorization: Bearer header or X-API-Key header
  const rawKey = extractApiKey(request);
  if (!rawKey) throw new AuthError(401, 'missing_api_key');

  // 2. Validate key against tenant_api_keys (HMAC hash comparison per §26.4)
  const keyRecord = await resolveApiKey(env, rawKey);
  if (!keyRecord || keyRecord.revoked_at !== null) throw new AuthError(401, 'invalid_api_key');
  if (keyRecord.expires_at && new Date(keyRecord.expires_at) < new Date()) {
    throw new AuthError(401, 'api_key_expired');
  }

  // ── NEW (§32.4.3): IP allowlist enforcement ──────────────────────────────
  const clientIp = request.headers.get('CF-Connecting-IP') ?? '';
  await enforceIpAllowlist(env, keyRecord.tenant_id, clientIp, 'api_key');
  // enforceIpAllowlist() is a no-op when ip_allowlist is empty or null;
  // throws AuthError(403) and emits sso.ip_allowlist_blocked DEC-030 event if blocked.
  // ──────────────────────────────────────────────────────────────────────────

  // 3. Emit api_key.request_authenticated DEC-030 event (existing — unchanged)
  await emitAuditEvent(env, {
    event_type: 'api_key.request_authenticated',
    severity: 'STANDARD',
    tenant_id: keyRecord.tenant_id,
    key_id: keyRecord.id,       // UUID, not the raw key
    client_ip_hash: sha256(clientIp + env.IP_HASH_SALT),
    scopes: keyRecord.scopes,
  });

  return { tenantId: keyRecord.tenant_id, keyId: keyRecord.id, scopes: keyRecord.scopes };
}
```

`enforceIpAllowlist()` is extended with an optional `authPath` parameter for DEC-030 event tagging:

```typescript
// src/workers/auth/middleware/ip-allowlist.ts — extend enforceIpAllowlist signature
// (SSO path behaviour UNCHANGED — authPath defaults to 'sso')

export async function enforceIpAllowlist(
  env: Env,
  tenantId: string,
  clientIp: string,
  authPath: 'sso' | 'api_key' = 'sso',  // ← NEW optional parameter
): Promise<void> {
  const config = await getIpAllowlistConfig(env, tenantId);
  if (!config.ip_allowlist || config.ip_allowlist.length === 0) return;

  const allowed = config.ip_allowlist.some((cidr: string) => ipInCidr(clientIp, cidr));
  if (!allowed) {
    await emitAuditEvent(env, {
      event_type: 'sso.ip_allowlist_blocked',    // same event type for both paths
      severity: 'HIGH',
      tenant_id: tenantId,
      client_ip_hash: sha256(clientIp + env.IP_HASH_SALT),
      auth_path: authPath,    // ← NEW field in DEC-030 payload
    });
    throw new AuthError(403, 'ip_not_allowlisted');
  }
}
```

The `auth_path` field is additive to the existing `sso.ip_allowlist_blocked` DEC-030 event payload — no new event type is introduced.

---

### §32.5 DEC-030 Event Updates

Two event-type changes from this section:

| Event | Change | Severity | Retention | SOC 2 |
|---|---|---|---|---|
| `sso.ip_allowlist_blocked` | **Payload extension** — add `auth_path: 'sso' \| 'api_key'` field; backwards-compatible (existing consumers ignore unknown fields) | HIGH | 7yr | CC6.1, CC6.3 |
| `scim.ip_enforcement_misconfigured` | **New event** — advisory only; emitted when `scim_ip_enforcement_enabled = true` but `ip_allowlist` is empty; no PagerDuty alert; compliance-officer review only | STANDARD | 3yr | CC6.1 advisory |

**Registration required in `docs/AUDIT_LOG_SCHEMA.md §SCIM` and `§SSO` namespaces:**
- `sso.ip_allowlist_blocked` — payload extension; add `auth_path` to Zod schema (optional field, default `'sso'` for backwards compatibility)
- `scim.ip_enforcement_misconfigured` — new event; register in `§SCIM` namespace

No new HMAC chain ordering invariants — both events are standalone (not part of a required sequence with other events).

---

### §32.6 SOC 2 Evidence

| ID | Criterion | Description | Path | Cadence | Retention |
|---|---|---|---|---|---|
| **CC6-E-ML-001** | CC6.3 | Magic-link CAEP integration test pass log (`src/tests/auth/magic-link-revocation.test.ts`) — confirms that `revoke:user` KV key revokes magic-link sessions within M4 deploy | `compliance/evidence/sso/CC6-E-ML-001_M4.md` | One-time (M4 deploy gate) | 7yr |
| **CC6-E-SCIM-IP-001** | CC6.1 | Screenshot of `scim_ip_enforcement_enabled = false` default in staging DB after migration `0076`; confirms that existing SCIM tenants are not affected | `compliance/evidence/sso/CC6-E-SCIM-IP-001_M5.md` | One-time (migration 0076 deploy) | 7yr |
| **CC6-E-APIKEY-IP-001** | CC6.1, CC6.3 | `sso.ip_allowlist_blocked` DEC-030 export filtered to `auth_path = 'api_key'` for first 30 days post-M5 — confirms IP enforcement is active on the API key path for tenants with allowlists configured | `compliance/evidence/sso/CC6-E-APIKEY-IP-001_<YYYY-MM>.csv` | Monthly from M5 | 3yr |

**Auditor narrative for CC6.3 (CC6-E-ML-001):** FORM's session revocation architecture (§22 KV-backed revocation layer) is designed to apply uniformly to all credential types. CC6-E-ML-001 provides empirical evidence that magic-link JWTs — the enterprise fallback credential path when SSO is unavailable — are subject to the same `revoke:user` and `account_suspended` KV controls as SSO JWTs. This is directly relevant to CC6.3 (access revocation is complete and timely): an account-disabled CAEP event from the enterprise IdP blocks all FORM credential types for that user within the same sub-second KV TTL bound as the SSO path.

**Auditor narrative for CC6.1 (CC6-E-SCIM-IP-001):** The `scim_ip_enforcement_enabled` flag defaults to `false` because most enterprise IdPs (Okta, Azure AD) use shared cloud SCIM infrastructure with dynamic IP ranges. This default-off design prevents a misconfig scenario where a customer's IP allowlist silently blocks IdP-initiated provisioning. Customers who operate self-hosted SCIM proxies with stable CIDRs may opt in. CC6-E-SCIM-IP-001 confirms that migration `0076` sets the safe default for all existing tenants.

---

### §32.7 OQ Gap Tracker

| OQ | Previous Status | Status After §32 | Decision |
|---|---|---|---|
| **OQ-SSO-23.2** | 🟡 P1 Open — Before M4 deploy | 🟢 **Resolved — DEC-062 (2026-06-16)** | `isRevoked()` covers magic-link sessions at middleware layer; call-graph confirmed; integration test CC6-E-ML-001 added as closure gate |
| **OQ-SSO-25.1** | 🟡 P1 Open — Before first IP-allowlist customer | 🟢 **Resolved — DEC-062 (2026-06-16)** | Option (c) adopted: `scim_ip_enforcement_enabled` flag, default `false`; schema migration `0076`; Admin Dashboard toggle with warning banner |
| **OQ-SSO-25.3** | 🟡 P1 Open — M5 | 🟢 **Resolved — DEC-062 (2026-06-16)** | `enforceIpAllowlist()` extended to `api-key-auth.ts`; `auth_path` field added to `sso.ip_allowlist_blocked` payload; no new event type |

**§23.9 cross-update:** OQ-SSO-23.2 row updated to 🟢 Resolved.
**§25.13 cross-update:** OQ-SSO-25.1 and OQ-SSO-25.3 rows updated to 🟢 Resolved.

---

### §32.8 Implementation Checklist

#### P0 — Before M4 go-live

| # | Task | Owner | Priority | Milestone | Status |
|---|---|---|---|---|---|
| 1 | Add `magic-link-revocation.test.ts` integration tests (§32.2.4) to `src/tests/auth/`; confirm both test cases pass in staging before M4 deploy. File CC6-E-ML-001 at `compliance/evidence/sso/CC6-E-ML-001_M4.md` (CI test output screenshot + pass/fail status). | platform-engineer + security-engineer | **P0** | M4 | [ ] |
| 2 | Update `docs/AUDIT_LOG_SCHEMA.md §SSO`: extend `sso.ip_allowlist_blocked` Zod schema with optional `auth_path: z.enum(['sso', 'api_key']).default('sso')` field. Bump AUDIT_LOG_SCHEMA event version to backwards-compatible patch. | platform-engineer + compliance-officer | **P0** | M4 | [ ] |

#### P0 — Before M5 / first enterprise SCIM go-live

| # | Task | Owner | Priority | Milestone | Status |
|---|---|---|---|---|---|
| 3 | Run migration `0076_scim_ip_enforcement_flag.sql` in staging; confirm `SELECT COUNT(*) FROM tenant_sso_configs WHERE scim_ip_enforcement_enabled IS NULL` returns 0; confirm `SELECT scim_ip_enforcement_enabled FROM tenant_sso_configs LIMIT 5` all return `false`. File CC6-E-SCIM-IP-001 screenshot. | devops-lead + compliance-officer | **P0** | Before first SCIM enterprise go-live | [ ] |
| 4 | Update `enforceScimIpAllowlist()` in `src/workers/auth/middleware/ip-allowlist.ts` per §32.3.4: add flag lookup, KV cache for `scim_ip_cfg:{tenant_id}`, `scim.ip_enforcement_misconfigured` advisory event. Update `apps/scim-worker/src/auth.ts` call site to use the updated function (call site signature unchanged). | platform-engineer | **P0** | Before first SCIM enterprise go-live | [ ] |
| 5 | Add `scim.ip_enforcement_misconfigured` event to `docs/AUDIT_LOG_SCHEMA.md §SCIM` namespace. Verify Zod schema (STANDARD, 3yr, `tenant_id` + `client_ip_hash` fields only). | platform-engineer + compliance-officer | **P0** | Before first SCIM enterprise go-live | [ ] |
| 6 | Extend `enforceIpAllowlist()` (§25.5) with optional `authPath: 'sso' | 'api_key'` parameter per §32.4.3; add the `auth_path` field to the `sso.ip_allowlist_blocked` DEC-030 emission. Update `src/workers/auth/api-key-auth.ts` to call `enforceIpAllowlist(env, tenantId, clientIp, 'api_key')` after token validation and before the `api_key.request_authenticated` event emission. | platform-engineer | **P0** | M5 | [ ] |
| 7 | Integration test: create a tenant with `ip_allowlist = ['10.0.0.0/8']`; authenticate with a valid API key from `8.8.8.8` (outside allowlist); confirm 403 response; confirm `sso.ip_allowlist_blocked` DEC-030 event is emitted with `auth_path = 'api_key'`. | platform-engineer | **P0** | M5 | [ ] |

#### P1 — Before SOC 2 observation period start (M7)

| # | Task | Owner | Priority | Milestone | Status |
|---|---|---|---|---|---|
| 8 | Add `scim_ip_enforcement_enabled` toggle to Admin Dashboard SCIM configuration panel (§16). Add the warning banner text per §32.3.4. Gate on M5 Admin Dashboard build. | platform-engineer + design-craft | **P1** | M5–M6 | [ ] |
| 9 | Update `§23.9` OQ-SSO-23.2 row and `§25.13` OQ-SSO-25.1 / OQ-SSO-25.3 rows to 🟢 Resolved in this document (already reflected in §32.7 — verify no merge conflict in the source sections). | compliance-officer | **P1** | M5 | [ ] |
| 10 | Begin collecting CC6-E-APIKEY-IP-001 monthly exports from M5. File first export after 30 days of M5 go-live. | compliance-officer | **P1** | M6 | [ ] |

#### P2 — Post-observation governance

| # | Task | Owner | Priority | Milestone | Status |
|---|---|---|---|---|---|
| 11 | At first enterprise customer with `scim_ip_enforcement_enabled = true`: run a provisioning smoke test from the customer's SCIM client to confirm no false-positive blocks. If blocked, investigate `scim.ip_allowlist_blocked` DEC-030 events; assist customer in adding IdP CIDR ranges to allowlist. | customer-success + platform-engineer | **P2** | First opt-in customer | [ ] |
| 12 | Evaluate whether `scim.ip_enforcement_misconfigured` events warrant a PagerDuty P3 alert after 30 days of data. If > 2 tenants trigger the advisory event, add AL-SCIM-05 (P3 Slack `#devops-alerts`, auto-resolve 24h). | devops-lead | **P2** | M8 | [ ] |

---

### §32.9 Open Questions

| ID | Question | Priority | Owner | Resolution path |
|---|---|---|---|---|
| **OQ-SSO-32.1** | **Should `scim_ip_enforcement_enabled` be surfaced in the SCIM token provisioning guide sent to customers at onboarding?** The current §7 onboarding checklist does not mention IP allowlist configuration for SCIM. If a customer asks "can we restrict SCIM to our proxy IP?" during onboarding (likely for security-conscious enterprises), the CSM must know to check `scim_ip_enforcement_enabled`. Recommendation: add a CSM note to the §17 pilot runbook and to the onboarding template in `docs/ENTERPRISE_ONBOARDING.md`. | P2 | customer-success + enterprise-architect | Before first customer asking for SCIM IP restriction |
| **OQ-SSO-32.2** | **Should the 5-minute KV TTL for `scim_ip_cfg:{tenant_id}` (§32.3.4) be shortened if a customer updates their `scim_ip_enforcement_enabled` flag via Admin Dashboard?** A TTL of 5 minutes means a toggle change takes up to 5 minutes to propagate to all SCIM Worker instances. For an emergency "disable SCIM IP enforcement" action (e.g., Okta changes its CIDRs mid-provisioning), a 5-minute delay could cause a provisioning gap. Option: add a cache invalidation call (`SSO_KV.delete(scim_ip_cfg:{tenant_id})`) in the Admin Dashboard API handler for the SCIM IP toggle mutation. | P2 | platform-engineer | Before first customer with `scim_ip_enforcement_enabled = true`; evaluate at M8 |

---

*v2.4 (2026-06-16): §32 OQ-SSO-23.2 · OQ-SSO-25.1 · OQ-SSO-25.3 Resolution — Magic-Link CAEP Session Coverage, SCIM IP Allowlist Scope & API Key IP Enforcement (DEC-062). Closes three P1 blockers that were all pre-M4/M5 deployment gates. §32.1 scopes to magic-link session revocation confirmation, SCIM IP flag design, and API key IP enforcement; all three affect `ip-allowlist.ts` or `api-key-auth.ts` and share migration window M4–M5. §32.2 OQ-SSO-23.2 resolution: `isRevoked()` confirmed to cover magic-link sessions at authenticate() middleware layer (before session_type branch); call-graph trace documents that `revoke:user:{tenant_id}:{user_id}` and `account_suspended:{tenant_id}:{user_id}` KV keys are checked for every JWT regardless of `session_type = 'magic_link'`; two integration test cases added to `src/tests/auth/magic-link-revocation.test.ts` as M4 deploy gate; evidence CC6-E-ML-001 (CC6.3, 7yr). §32.3 OQ-SSO-25.1 resolution: option (c) adopted — `scim_ip_enforcement_enabled BOOLEAN NOT NULL DEFAULT FALSE` column added to `tenant_sso_configs` via migration `0076_scim_ip_enforcement_flag.sql`; `enforceScimIpAllowlist()` updated with KV-cached flag lookup (`scim_ip_cfg:{tenant_id}`, 5-min TTL); new advisory DEC-030 event `scim.ip_enforcement_misconfigured` (STANDARD, 3yr) emitted on misconfiguration (flag true, empty allowlist) without blocking; Admin Dashboard toggle with Okta/Azure AD warning banner; evidence CC6-E-SCIM-IP-001 (CC6.1, 7yr). §32.4 OQ-SSO-25.3 resolution: `enforceIpAllowlist()` extended with optional `authPath: 'sso' | 'api_key'` parameter; `api-key-auth.ts` calls `enforceIpAllowlist(env, tenantId, clientIp, 'api_key')` after token validation; `sso.ip_allowlist_blocked` DEC-030 payload extended with `auth_path` field (backwards-compatible addition); no new event type; evidence CC6-E-APIKEY-IP-001 (CC6.1/CC6.3, 3yr). §32.5 two DEC-030 changes: `sso.ip_allowlist_blocked` payload extension (`auth_path` optional field); `scim.ip_enforcement_misconfigured` new STANDARD event. §32.6 three SOC 2 evidence artefacts: CC6-E-ML-001 (CC6.3 — magic-link revocation integration test), CC6-E-SCIM-IP-001 (CC6.1 — migration 0076 default-false confirmation), CC6-E-APIKEY-IP-001 (CC6.1/CC6.3 — monthly API key IP block export). §32.7 gap tracker: OQ-SSO-23.2 🟡→🟢 Resolved, OQ-SSO-25.1 🟡→🟢 Resolved, OQ-SSO-25.3 🟡→🟢 Resolved; cross-updates to §23.9 and §25.13 flagged. §32.8 twelve-item implementation checklist: 2× P0/M4 (magic-link integration test + CC6-E-ML-001, AUDIT_LOG_SCHEMA extension), 5× P0/M5 (migration 0076, enforceScimIpAllowlist update, scim.ip_enforcement_misconfigured registration, api-key-auth extension, API key IP integration test), 3× P1/M5–M6 (Admin Dashboard toggle, OQ cross-updates, CC6-E-APIKEY-IP-001 collection start), 2× P2/M8–first-opt-in-customer (SCIM IP smoke test, AL-SCIM-05 evaluation). §32.9 two open questions: OQ-SSO-32.1 (P2 — SCIM IP toggle in onboarding guide); OQ-SSO-32.2 (P2 — 5-min KV TTL vs. emergency toggle propagation). Document header updated v2.3 → v2.4. Cross-references: §10 (magic-link session design — session_type = 'magic_link'; §32.2 confirms revocation coverage); §22.5 (KV revocation key patterns — `revoke:user`, `account_suspended`, `force_reauth` confirmed to cover magic-link); §23.3.2 (CAEP account-disabled action — writes `account_suspended` and `revoke:user` KV keys; §32.2 confirms both reach magic-link sessions); §23.9 (OQ-SSO-23.2 — now 🟢 Resolved); §25.5 (IP enforcement table — SCIM path now flag-gated per §32.3; SSO path unchanged); §25.13 (OQ-SSO-25.1, OQ-SSO-25.3 — both now 🟢 Resolved); §26.3 (API key rotation — SCIM token; `api-key-auth.ts` now also enforces IP allowlist per §32.4); §27.3 (SCIM Worker auth pipeline — `enforceScimIpAllowlist()` call site unchanged; flag-gating is internal to the function); `docs/AUDIT_LOG_SCHEMA.md §SSO` (`sso.ip_allowlist_blocked` Zod schema — add optional `auth_path` field — P0/M4); `docs/AUDIT_LOG_SCHEMA.md §SCIM` (`scim.ip_enforcement_misconfigured` — new event registration — P0/before-SCIM-go-live); `docs/DATA_MODEL.md §26` (`tenant_api_keys` — no schema change; `api-key-auth.ts` reads tenant_id from resolved key record to call `enforceIpAllowlist()`); `docs/DATA_MODEL.md §4.2` (`tenant_sso_configs` — migration `0076` adds `scim_ip_enforcement_enabled BOOLEAN NOT NULL DEFAULT FALSE`); `docs/SOC2_READINESS.md §CC6.1` (CC6-E-SCIM-IP-001, CC6-E-APIKEY-IP-001 — to be added at M5 observation filing); `docs/SOC2_READINESS.md §CC6.3` (CC6-E-ML-001 — to be added at M4 observation filing); `docs/ENTERPRISE_ONBOARDING.md` (OQ-SSO-32.1 — add SCIM IP toggle note to CSM pilot runbook — P2/M8); `docs/DECISION_LOG.md DEC-062`. Privacy floor: no individual employee `user_id`, name, email, health data, or body composition values in any DEC-030 event emitted by §32 controls; `client_ip_hash` is SHA-256(ip + salt) with `IP_HASH_SALT`; `auth_path` is a structural enum tag only; `tenant_id` in DEC-030 events is the organisation slug, not a personal data identifier; `form_api` SELECT access to `scim_ip_enforcement_enabled` is limited to the SCIM Worker auth pipeline (read-only; no write via form_api). Owner: enterprise-architect + security-engineer + platform-engineer + compliance-officer.*

---

## 33. OQ-SSO-32.1 & OQ-SSO-32.2 Resolution — SCIM IP Enforcement Onboarding Visibility & KV Cache Invalidation

### §33.1 Scope and Rationale

Two open questions from §32.9 are resolved here. Both concern the operational lifecycle of the `scim_ip_enforcement_enabled` flag introduced in §32.3 (DEC-062):

- **OQ-SSO-32.1** (P2) — The feature is invisible to customers at onboarding: §17.2 (Pre-Pilot Technical Qualification) and `docs/ENTERPRISE_ONBOARDING.md §3.3` do not mention the SCIM IP enforcement capability. Security-conscious customers running self-hosted SCIM proxies with stable CIDR ranges cannot discover or enable this protection without an engineering escalation.

- **OQ-SSO-32.2** (P2) — The `scim_ip_cfg:{tenant_id}` KV cache has a 5-minute TTL with no active push invalidation. For a "disable IP enforcement" emergency action (e.g., Okta changes its SCIM server's IP ranges mid-provisioning), an admin's toggle in Admin Dashboard takes up to 5 minutes to propagate to all SCIM Worker instances, creating a provisioning gap during an already-stressful incident.

Both resolutions are additive. No schema migrations. No new DEC-030 event types beyond the single advisory event in §33.4.

---

### §33.2 OQ-SSO-32.1 Resolution: SCIM IP Enforcement in the Onboarding Guide

#### §33.2.1 Decision

**Option adopted:** Add a CSM-facing note to §17.2 Q-05 and §17.3 Step 3 (SCIM configuration), and add a parallel note to `docs/ENTERPRISE_ONBOARDING.md §3.3`.

**When to surface the feature:** Only for customers who:
- Run a **self-hosted SCIM proxy** (e.g., Okta SCIM Connector Agent, Azure AD App Proxy) with stable, known CIDR ranges.
- Have expressed a requirement to restrict SCIM inbound connections to a specific IP range (common in financial-sector and healthcare enterprises).

For customers using **cloud-hosted IdP SCIM** (Okta SaaS, Azure AD managed tenants, Google Workspace), `scim_ip_enforcement_enabled` should **not** be recommended — cloud IdPs use dynamic IP ranges and enabling enforcement without first confirming static CIDR support will silently block provisioning.

#### §33.2.2 §17.2 Q-05 Checklist Extension

The following replaces the Q-05 row in §17.2 Pre-Pilot Technical Qualification:

| # | Item | Owner | Verification |
|---|---|---|---|
| Q-05 | Network: no firewall rule that would block outbound requests from `{slug}-staging.form.coach` to IdP metadata endpoints. **If the customer operates a self-hosted SCIM proxy (e.g., Okta SCIM Connector Agent, Azure AD App Proxy) and wants to restrict inbound SCIM connections to that proxy's IP range: capture the stable CIDR block(s) during this qualification call. `scim_ip_enforcement_enabled` is available at Admin Dashboard → SSO Settings → SCIM → IP Restriction and defaults to `false`. Enable it only after SCIM is confirmed working — at least one successful full-directory sync with > 0 users provisioned.** | Customer IT/security + CS | If IP enforcement desired: proxy CIDR block(s) on file before pilot go-live; enablement deferred until post-sync confirmation (§17.3 Step 3 note) |

#### §33.2.3 §17.3 SCIM Configuration Step Extension

In §17.3 Step 3 (SCIM provisioning), after the bearer token generation step, add:

> **SCIM IP restriction (optional — self-hosted SCIM proxies only):** If CIDR blocks were captured in Q-05, enable Admin Dashboard → SSO Settings → SCIM → IP Restriction **after** the first successful full-directory SCIM sync (> 0 users provisioned). Enter the proxy's CIDR block(s) and save. The Admin Dashboard shows an Okta/Azure AD warning banner (§32.3.4) reminding the admin to test provisioning immediately after enabling. Do **not** enable for cloud-hosted IdP tenants (Okta cloud, Azure AD SaaS) without first confirming the IdP's outbound SCIM IP ranges are static and documented.
>
> If the customer enables the flag and SCIM provisioning stops, the fastest recovery is: Admin Dashboard → disable `scim_ip_enforcement_enabled` → wait up to 5 seconds for KV cache to refresh (§33.3.3 active invalidation) → verify provisioning resumes → re-enable with the corrected CIDR list.

#### §33.2.4 `docs/ENTERPRISE_ONBOARDING.md §3.3` Extension Note

The following text should be added to `docs/ENTERPRISE_ONBOARDING.md §3.3` after the SCIM audit events table:

> **SCIM IP enforcement (self-hosted proxy customers only):** Customers running a self-hosted SCIM proxy with stable IP ranges may enable `scim_ip_enforcement_enabled` at Admin Dashboard → SSO Settings → SCIM → IP Restriction. This restricts inbound SCIM provisioning to the specified CIDRs; calls from other IPs are rejected (403) and logged as `scim.ip_allowlist_blocked` DEC-030 events. Enable only after a successful full-directory sync and only for tenants with confirmed static proxy IPs. Cloud-hosted IdP tenants (Okta SaaS, Azure AD) should not use this control without confirming static CIDR support with their IdP.
>
> Implementation reference: `docs/SSO_SCIM_IMPLEMENTATION.md §32.3` (DEC-062) and `§33.2` (onboarding protocol).

#### §33.2.5 CSM Script Fragment

When a customer asks **"Can we restrict SCIM to only our SCIM proxy's IP?"** during onboarding:

> "Yes — FORM supports SCIM IP allowlisting via a toggle in the Admin Dashboard. Once your SCIM provisioning is confirmed working — meaning you've seen users sync successfully — we can restrict inbound SCIM calls to your proxy server's IP range at Admin Dashboard → SSO Settings → SCIM → IP Restriction. Before we do that, can you share the IP address or CIDR block your SCIM proxy server uses? We'll want to test provisioning immediately after enabling to make sure nothing is blocked."

**Avoid:** Do not enable before at least one successful full-directory sync. If the CIDR is incomplete, provisioning will block silently — the SCIM Worker returns 403 to the IdP, which retries and eventually alarms. Start with SCIM working, then layer on IP restriction.

---

### §33.3 OQ-SSO-32.2 Resolution: KV Cache Invalidation on `scim_ip_enforcement_enabled` Toggle

#### §33.3.1 Decision

**Option adopted:** Add active push invalidation — `env.SSO_KV.delete('scim_ip_cfg:' + tenantId)` — to the Admin Dashboard API handler for the SCIM IP toggle mutation, immediately after the Supabase write succeeds.

This mirrors the established §25.4 pattern for `SSO_KV:auth_policy:{tenant_id}`, where the auth policy cache is immediately deleted on any `PATCH /v1/admin/sso/policy` mutation, eliminating the 60-second TTL lag on security-relevant policy changes.

#### §33.3.2 Problem Statement: The 5-Minute Disable Gap

The `scim_ip_cfg:{tenant_id}` KV entry (§32.3.4) has a 5-minute TTL (300 s) and **no active invalidation**. The 5-minute TTL is acceptable for the **enable** path — enabling enforcement is deliberate, tested, and done once. It is not acceptable for the **disable** path:

When Okta rotates its SCIM server IPs during a region migration (an observed real-world event), the customer's IT admin:
1. Gets paged on a SCIM provisioning failure (new-hire onboarding blocking).
2. Disables `scim_ip_enforcement_enabled` in Admin Dashboard.
3. Expects provisioning to resume immediately.
4. Instead, the SCIM Worker continues enforcing the old `enabled: true` KV entry for up to 5 minutes.
5. Provisioning remains blocked; the incident drags; the customer loses confidence.

The `scim.ip_allowlist_blocked` DEC-030 events fire in the background but the customer sees 403s from their IdP until the KV TTL expires. Active invalidation reduces this from 5 minutes to sub-second.

#### §33.3.3 Implementation

Extend `PATCH /v1/admin/scim/ip-enforcement` in the Admin Dashboard API:

```typescript
// apps/admin-dashboard-api/src/routes/scim-ip-enforcement.ts

import { z } from 'zod';
import type { Env } from '../types';

const BodySchema = z.object({
  scim_ip_enforcement_enabled: z.boolean(),
});

export async function handleScimIpEnforcementToggle(
  req: Request,
  env: Env,
  tenantId: string,
  actorId: string,
): Promise<Response> {
  // 1. Validate body
  const body = BodySchema.safeParse(await req.json());
  if (!body.success) {
    return Response.json({ error: 'invalid_body', details: body.error.issues }, { status: 400 });
  }

  // 2. Write to Supabase + emit sso.policy_updated DEC-030 event (atomic via RPC)
  const { error } = await env.SUPABASE.rpc('update_scim_ip_enforcement_flag', {
    p_tenant_id: tenantId,
    p_enabled:   body.data.scim_ip_enforcement_enabled,
    p_actor_id:  actorId,
  });
  if (error) {
    return Response.json({ error: 'supabase_error', message: error.message }, { status: 502 });
  }

  // 3. Immediate KV cache invalidation — prevents up to 5-min propagation lag on
  //    the disable path (e.g., IdP CIDR rotation forcing an emergency toggle-off).
  //    On the next SCIM request, enforceScimIpAllowlist() cache-misses and
  //    re-fetches from Supabase, picking up the new value immediately.
  //    Pattern: §25.4 SSO_KV:auth_policy:{tenant_id} invalidation.
  try {
    await env.SSO_KV.delete(`scim_ip_cfg:${tenantId}`);
  } catch (kvErr: unknown) {
    // KV delete failure: log advisory event; do NOT roll back the Supabase write.
    // Fallback: the 5-minute TTL will propagate the change within 5 minutes.
    await emitAdvisoryAuditEvent(env, {
      event_type: 'scim.ip_kv_invalidation_error',
      severity:   'STANDARD',
      tenant_id:  tenantId,
      actor_id:   actorId,
      kv_key:     'scim_ip_cfg',
      error_code: kvErr instanceof Error ? kvErr.message : 'unknown',
    });
  }

  return Response.json({
    ok: true,
    scim_ip_enforcement_enabled: body.data.scim_ip_enforcement_enabled,
  });
}
```

**Supabase RPC `update_scim_ip_enforcement_flag` (migration `0077_scim_ip_enforcement_toggle_rpc.sql`):**

```sql
CREATE OR REPLACE FUNCTION update_scim_ip_enforcement_flag(
  p_tenant_id UUID,
  p_enabled   BOOLEAN,
  p_actor_id  UUID
) RETURNS void
LANGUAGE plpgsql
SECURITY DEFINER
AS $$
DECLARE
  v_previous BOOLEAN;
BEGIN
  SELECT scim_ip_enforcement_enabled
    INTO v_previous
    FROM tenant_sso_configs
   WHERE tenant_id = p_tenant_id;

  IF NOT FOUND THEN
    RAISE EXCEPTION 'tenant_not_found: %', p_tenant_id;
  END IF;

  UPDATE tenant_sso_configs
     SET scim_ip_enforcement_enabled = p_enabled,
         updated_at = now()
   WHERE tenant_id = p_tenant_id;

  -- Reuse sso.policy_updated event (§25.9) — no new event type required.
  INSERT INTO audit_log_events (
    tenant_id, actor_id, actor_type,
    event_type, severity, payload
  ) VALUES (
    p_tenant_id, p_actor_id, 'user',
    'sso.policy_updated', 'STANDARD',
    jsonb_build_object(
      'policy_type',   'scim_ip_enforcement',
      'previous_value', v_previous,
      'new_value',      p_enabled,
      'change_source', 'admin_dashboard'
    )
  );
END;
$$;
```

#### §33.3.4 Behaviour After §33

| Scenario | Before §33 | After §33 |
|---|---|---|
| Admin disables `scim_ip_enforcement_enabled` (emergency — IdP CIDR rotated) | SCIM Worker enforces for up to 5 min (stale KV entry) | KV entry immediately deleted; next SCIM call cache-misses → Supabase fetch → `enabled: false`; provisioning resumes in < 1 s |
| Admin enables `scim_ip_enforcement_enabled` | Enforcement begins at next KV cache miss (≤ 5 min) | Same: KV delete forces immediate cache-miss, enforcement starts in < 1 s — minor improvement; the enable path is not an emergency |
| KV delete fails (transient Cloudflare KV error) | N/A | Supabase write succeeded; `scim.ip_kv_invalidation_error` advisory event emitted; Supabase response is 200; 5-min TTL fallback active |
| SCIM Worker receives provisioning call 500 ms after disable toggle | Stale KV: 403 returned to IdP | Cache miss → Supabase read (~15 ms overhead); `enabled: false` found; provisioning proceeds |

#### §33.3.5 Why Retain the 5-Minute Base TTL?

Shortening the base TTL (e.g., to 30 s) was evaluated and rejected:

1. **Supabase read amplification:** SCIM syncs during initial onboarding can generate 1–5 requests/second per tenant. At a 30-second TTL, a 1,000-seat sync would hit Supabase 2–10× per minute on the `scim_ip_cfg` path vs. once per 5 minutes — a 10–50× read amplification for a control that changes at most a few times per customer lifetime.
2. **KV cost is not the constraint:** At Cloudflare Workers Paid KV pricing ($0.50/10M reads), the cost difference between 30-second and 5-minute TTLs is negligible. The Supabase connection budget is the real concern.
3. **Active invalidation is strictly targeted:** The `SSO_KV.delete()` call in §33.3.3 triggers only on Admin Dashboard toggle mutations — not per-SCIM-request. Normal provisioning traffic is unaffected.

Decision: 5-minute base TTL retained; active push invalidation added. Consistent with §25.4 (60-second TTL + active invalidation for `auth_policy:{tenant_id}`).

---

### §33.4 DEC-030 Events

| Event | Type | Rationale |
|---|---|---|
| `sso.policy_updated` | **Payload reuse** — emitted by `update_scim_ip_enforcement_flag()` with `policy_type: 'scim_ip_enforcement'`, `previous_value`, `new_value`. No new event type; `policy_type` is an open enum per §25.9. | CC6.1 requires that security policy changes are logged. The existing `sso.policy_updated` event covers this; the toggle appears in existing monthly CC6.1 evidence exports. |
| `scim.ip_kv_invalidation_error` | **New advisory STANDARD 3yr event** — emitted if `SSO_KV.delete()` throws in the §33.3.3 handler. Non-blocking (Supabase write succeeded; 5-min TTL fallback remains). | Operational visibility: systematic KV delete failures are otherwise invisible until a customer reports a provisioning gap after a disable toggle. |

**`scim.ip_kv_invalidation_error` Zod schema:**

```typescript
z.object({
  tenant_id:  z.string().uuid(),
  actor_id:   z.string().uuid(),
  kv_key:     z.literal('scim_ip_cfg'),
  error_code: z.string(),
})
```

**Registration required:** `docs/AUDIT_LOG_SCHEMA.md §SCIM` — add `scim.ip_kv_invalidation_error` as STANDARD, 3yr, advisory (same pattern as `scim.ip_enforcement_misconfigured` from §32.5).

---

### §33.5 SOC 2 Impact

No new evidence artefacts. No gap score change.

| Criterion | Impact |
|---|---|
| CC6.1 | The `scim_ip_enforcement_enabled` toggle is recorded via `sso.policy_updated` with `policy_type: 'scim_ip_enforcement'`. This already appears in the existing §25.9 monthly CC6.1 evidence export — no new artefact required. |
| CC6.3 | Active invalidation reduces the worst-case propagation lag for the disable path from 5 minutes to < 1 second. This strengthens the CC6.3 narrative ("logical access controls operate in a timely manner") without requiring a new evidence artefact. |

---

### §33.6 OQ Gap Tracker

| OQ | Previous Status | Status After §33 | Decision |
|---|---|---|---|
| **OQ-SSO-32.1** | 🟡 P2 Open — before first customer asking for SCIM IP restriction | 🟢 **Resolved — 2026-06-17** | CSM note added to §17.2 Q-05, §17.3 Step 3, and `docs/ENTERPRISE_ONBOARDING.md §3.3` reference text (§33.2). CSM script fragment (§33.2.5) covers the live-onboarding scenario. |
| **OQ-SSO-32.2** | 🟡 P2 Open — before first customer with `scim_ip_enforcement_enabled = true` | 🟢 **Resolved — 2026-06-17** | Active KV invalidation adopted per §33.3.3; 5-minute base TTL retained; `scim.ip_kv_invalidation_error` advisory event added (§33.4) for KV delete failure visibility. |

**§32.9 cross-update:** OQ-SSO-32.1 and OQ-SSO-32.2 rows updated to 🟢 Resolved.

---

### §33.7 Implementation Checklist

#### P0 — Before first enterprise customer enabling `scim_ip_enforcement_enabled = true` (M8–M9)

| # | Task | Owner | Priority | Milestone | Status |
|---|---|---|---|---|---|
| 1 | Deploy migration `0077_scim_ip_enforcement_toggle_rpc.sql`: CREATE OR REPLACE FUNCTION `update_scim_ip_enforcement_flag()` per §33.3.3. Verify in staging: (a) flag updated correctly for `true` and `false`; (b) `sso.policy_updated` DEC-030 event emitted with `policy_type: 'scim_ip_enforcement'`, `previous_value`, and `new_value`; (c) `tenant_not_found` exception raised on unknown `p_tenant_id`. | platform-engineer | **P0** | M8 | [ ] |
| 2 | Update `apps/admin-dashboard-api/src/routes/scim-ip-enforcement.ts` per §33.3.3: add `env.SSO_KV.delete('scim_ip_cfg:' + tenantId)` after Supabase RPC call; wrap in try/catch and emit `scim.ip_kv_invalidation_error` on failure without blocking the 200 response. | platform-engineer | **P0** | M8 | [ ] |
| 3 | Register `scim.ip_kv_invalidation_error` in `docs/AUDIT_LOG_SCHEMA.md §SCIM` (STANDARD, 3yr, advisory — same pattern as `scim.ip_enforcement_misconfigured` per §32.5). Deploy updated event type to `emit-audit-event` Worker registry. | platform-engineer + compliance-officer | **P0** | M8 | [x] **Done — `docs/AUDIT_LOG_SCHEMA.md §SCIM IP KV Cache Invalidation Error events` v2.16 (2026-06-17): event registered; retention table row added; closes this checklist item. Implementation tasks (Worker registry + integration test) remain as items 2 and 4.** |
| 4 | Integration test (`src/tests/sso/scim-ip-kv-invalidation.test.ts`): (a) Set `scim_ip_enforcement_enabled = true` in staging `tenant_sso_configs`; populate `scim_ip_cfg:{tenant_id}` KV entry with `enabled: true`; call `handleScimIpEnforcementToggle` with `{ scim_ip_enforcement_enabled: false }`; assert KV entry is deleted (GET returns `null`); assert next `enforceScimIpAllowlist()` call fetches from Supabase and returns `enabled: false`. (b) Simulate KV delete error; assert `scim.ip_kv_invalidation_error` advisory event is emitted and 200 is still returned. | platform-engineer | **P0** | M8 | [ ] |

#### P1 — Before SOC 2 observation period (M11)

| # | Task | Owner | Priority | Milestone | Status |
|---|---|---|---|---|---|
| 5 | Update `docs/ENTERPRISE_ONBOARDING.md §3.3` with the SCIM IP enforcement note per §33.2.4. Review with customer-success for CSM script accuracy; confirm the cloud-IdP exclusion is explicit. | customer-success + compliance-officer | **P1** | M9 | [x] Done — 2026-06-19. Note added after SCIM audit events table in `docs/ENTERPRISE_ONBOARDING.md §3.3`: self-hosted proxy use case, `scim_ip_enforcement_enabled` toggle path, 403/`scim.ip_allowlist_blocked` behaviour, cloud-IdP exclusion explicit. |
| 6 | Verify §17.2 Q-05 row in this document reflects the extended text from §33.2.2. Cross-check against the onboarding call checklist used by customer-success. | customer-success | **P1** | M9 | [x] Done — 2026-06-19. §17.2 Q-05 row updated with full CIDR-capture extension: self-hosted SCIM proxy identification, `scim_ip_enforcement_enabled` toggle path, enablement-after-first-sync gate. Cloud-IdP exclusion explicit. |

#### P2 — After first opt-in customer

| # | Task | Owner | Priority | Milestone | Status |
|---|---|---|---|---|---|
| 7 | After the first customer enables `scim_ip_enforcement_enabled = true`: validate the end-to-end flow (enable → SCIM test → disable → SCIM test within 5 seconds). File a short note in the customer's onboarding record confirming the OQ-SSO-32.2 fix behaved as expected (disable propagation < 1 s). | customer-success + platform-engineer | **P2** | First opt-in customer | [ ] |

---

### §33.8 Open Questions

OQ-SSO-32.1 and OQ-SSO-32.2 are closed. No new open questions are introduced by this section.

---

*v2.5 (2026-06-17): §33 OQ-SSO-32.1 & OQ-SSO-32.2 Resolution — SCIM IP Enforcement Onboarding Visibility & KV Cache Invalidation. Closes both P2 open questions from §32.9 (v2.4, 2026-06-16). §33.1 scopes: OQ-SSO-32.1 is an operational visibility gap (onboarding guide omits `scim_ip_enforcement_enabled` feature); OQ-SSO-32.2 is a propagation lag risk on the disable path (5-min KV TTL, no active invalidation). §33.2 OQ-SSO-32.1 resolution: feature surfaced only for self-hosted SCIM proxy customers (cloud-hosted IdP tenants explicitly excluded due to dynamic IP ranges); §17.2 Q-05 row extended with CIDR capture step for self-hosted proxy customers; §17.3 Step 3 SCIM config note added (enable only after first successful full-directory sync; disable recovery path documented — < 5 s with §33.3.3 active invalidation); `docs/ENTERPRISE_ONBOARDING.md §3.3` extension note referenced; CSM script fragment (§33.2.5) for the live-call scenario. §33.3 OQ-SSO-32.2 resolution: active push invalidation adopted — `env.SSO_KV.delete('scim_ip_cfg:' + tenantId)` added to `PATCH /v1/admin/scim/ip-enforcement` handler after Supabase RPC; mirrors §25.4 pattern for `SSO_KV:auth_policy:{tenant_id}`; 5-minute base TTL retained (10–50× Supabase read amplification at 30 s TTL rejected); Postgres RPC `update_scim_ip_enforcement_flag()` (migration `0077`) handles UPDATE + `sso.policy_updated` DEC-030 emit atomically; KV delete wrapped in try/catch — failure emits `scim.ip_kv_invalidation_error` advisory event without rolling back the Supabase write; §33.3.4 before/after behaviour matrix: disable path reduces from ≤ 5 min to < 1 s; enable path also immediate (minor improvement); KV-failure fallback = original 5-min TTL. §33.4 two DEC-030 changes: `sso.policy_updated` reused (payload: `policy_type: 'scim_ip_enforcement'`, `previous_value`, `new_value`, `change_source: 'admin_dashboard'`); `scim.ip_kv_invalidation_error` new advisory STANDARD 3yr event (non-blocking; same registration pattern as `scim.ip_enforcement_misconfigured`). §33.5 SOC 2: no new evidence artefacts; CC6.1 toggle covered by existing `sso.policy_updated` monthly export; CC6.3 narrative strengthened but no new artefact. §33.6 gap tracker: OQ-SSO-32.1 🟡→🟢 Resolved; OQ-SSO-32.2 🟡→🟢 Resolved; §32.9 cross-update noted. §33.7 seven-item checklist: 4× P0/M8 (RPC migration 0077, KV delete handler, AUDIT_LOG_SCHEMA registration, integration tests × 2 scenarios), 2× P1/M9 (ENTERPRISE_ONBOARDING.md update, §17.2 Q-05 cross-check), 1× P2/first-opt-in (end-to-end smoke test). §33.8 no new open questions. TOC entry §33 added. Document header v2.4 → v2.5. Privacy floor: no `user_id`, name, email, health data, or body composition values in any event or implementation artefact in §33; `actor_id` in DEC-030 is the Admin Dashboard operator (tenant admin/owner), not an employee or SCIM-provisioned user; `tenant_id` is org slug in audit events; `form_api` NO ACCESS to `scim_ip_cfg` KV key. Cross-references: §17.2 (Q-05 row extended per §33.2.2); §17.3 (Step 3 SCIM config note per §33.2.3); §25.4 (KV-backed policy cache — `SSO_KV:auth_policy:{tenant_id}` invalidation pattern mirrored here); §25.9 (`sso.policy_updated` event schema — reused with `policy_type: 'scim_ip_enforcement'`); §32.3.4 (`scim_ip_cfg:{tenant_id}` KV key + 5-min TTL — unchanged; active invalidation is additive); §32.5 (`scim.ip_enforcement_misconfigured` — `scim.ip_kv_invalidation_error` follows same advisory registration pattern); §32.9 (OQ-SSO-32.1, OQ-SSO-32.2 — both 🟢 Resolved); `docs/ENTERPRISE_ONBOARDING.md §3.3` (SCIM IP enforcement note — P1/M9); `docs/AUDIT_LOG_SCHEMA.md §SCIM` (`scim.ip_kv_invalidation_error` advisory event — P0/M8); `docs/DATA_MODEL.md §4.2` (`tenant_sso_configs.scim_ip_enforcement_enabled` — read + updated by `update_scim_ip_enforcement_flag()`). Owner: enterprise-architect + customer-success + platform-engineer + compliance-officer.*
