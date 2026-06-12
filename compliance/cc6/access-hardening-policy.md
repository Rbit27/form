# FORM · Credential & Access Hardening Policy

| Field | Value |
|---|---|
| **Policy ID** | POL-015 |
| **Version** | v1.0 |
| **Status** | IN_FORCE |
| **Effective date** | 2026-06-07 |
| **Owner** | `security-engineer` + `devops-lead` + `compliance-officer` |
| **Next review** | 2027-01-01 (§15 compliance calendar Q1-Jan) |
| **Gap closures** | CC6-GAP-001 · CC6-GAP-002 · CC6-GAP-003 · CC6-GAP-004 · CC6-GAP-005 · CC6-GAP-006 · CC6-GAP-007 (P1) |
| **SOC 2 criteria** | CC6.1 · CC6.2 · CC6.3 · CC6.6 · CC6.8 |
| **Evidence artifacts** | CC6-E-001a · CC6-E-001b · CC6-E-002 · CC6-E-003 · CC6-E-004 · CC6-E-005 · CC6-E-006 |
| **Source section** | `docs/SOC2_READINESS.md §43` |

> **Standalone auditor exhibit.** This document is designed to be delivered directly to SOC 2 auditors without requiring cross-reference to `docs/SOC2_READINESS.md §26`, which provides the access architecture narrative. §26 is the architecture document; this policy is the binding implementation specification.
>
> **Relationship to audit log:** All enforcement events listed below are DEC-030 HMAC-chained events per `docs/AUDIT_LOG_SCHEMA.md`. Chain break on any event in this policy's scope is a P0 incident per `docs/INCIDENT_RESPONSE.md §R-05`.

---

## 1. Purpose and Scope

This policy documents FORM's mandatory credential and access hardening controls at four layers:

| Layer | Control | SOC 2 Criterion |
|---|---|---|
| Internal tooling MFA | GitHub org + Cloudflare account-level 2FA enforcement | CC6.1, CC6.2 |
| Application-layer TOTP | Tenant admin / tenant owner role-gated TOTP enforcement | CC6.1, CC6.2 |
| SCIM deprovisioning | `DELETE /scim/v2/Users/{userId}` lifecycle handler | CC6.3 |
| CI dependency audit | `npm audit --audit-level=critical` + Dependabot | CC6.8 |
| Secret scanning | `git-secrets` pre-commit hook + CI scan step | CC6.6 |
| CSP + static analysis | `Content-Security-Policy` header + ESLint `no-eval` (P1) | CC6.8 |

**Out of scope:** Consumer user authentication flows (see `docs/SSO_SCIM_IMPLEMENTATION.md §6`); physical access controls (fully remote, cloud-native architecture); break-glass procedure (see `docs/AUDIT_LOG_SCHEMA.md`); penetration testing (see `docs/SOC2_READINESS.md §36`); vulnerability management SLA (see `docs/SOC2_READINESS.md §41`).

---

## 2. Internal Tooling MFA Enforcement

### 2.1 GitHub Organization — CC6-GAP-001

**Policy:** Every GitHub organization member must enrol TOTP or a hardware security key. SMS OTP alone is not accepted (NIST SP 800-63B deprecates SMS for Level 2 assurance).

**Required configuration:**
```
GitHub org settings → Security → Authentication security:
  ☑  Require two-factor authentication for everyone in the organisation
```

**Permitted 2FA methods:**

| Method | Status |
|---|---|
| TOTP authenticator app (Google Authenticator, Authy, 1Password TOTP) | Required minimum |
| Hardware security key (YubiKey, passkey) | Recommended; required for engineers with production-deploy access |
| SMS OTP | Not accepted as sole method |

**Compensating control (solo-founder phase):** Founder already uses 1Password TOTP on GitHub account (§26.3.1). The org-level policy must be enabled before any second member is added — this is a **pre-hire gate**, not a pre-launch gate.

**Evidence artefact CC6-E-001a:**
```
File: compliance/evidence/cc6/github-mfa-org-policy-YYYY-MM.png
Content: Screenshot of GitHub org Security settings with "Require two-factor
         authentication" checked and org name visible
Cadence: At time of enabling; re-captured at each quarterly access review (§23)
Retention: 7 years
```

### 2.2 Cloudflare Account — CC6-GAP-002

**Blast radius:** Unauthorized Cloudflare access exposes all production secrets in Workers Secrets (Anthropic key, Supabase service role key, DEC-030 HMAC signing key, ElevenLabs API key), WAF rules, and edge routing. MFA enforcement here is non-negotiable even before the first team hire.

**Required configuration:**
```
Cloudflare dashboard → Account → Security:
  ☑  Require 2FA for all members
```

**Evidence artefact CC6-E-001b:**
```
File: compliance/evidence/cc6/cloudflare-mfa-policy-YYYY-MM.png
Content: Screenshot of Cloudflare Account Security settings showing
         "Require 2FA for all members" enabled and account name visible
Cadence: At time of enabling; re-captured at each quarterly access review
Retention: 7 years
```

---

## 3. Application-Layer Tenant Admin TOTP Enforcement — CC6-GAP-003

### 3.1 Policy Decision

All `tenant_owner` and `tenant_admin` role holders must complete TOTP enrolment via Supabase Auth `mfa_factors` before invoking any role-gated admin API endpoint. This applies to both SSO and non-SSO admin accounts, subject to the IdP delegation exemption in §3.3.

**Gap closed:** Enterprise tenant admins could previously perform SSO configuration, bulk provisioning, and audit log export without MFA at the FORM application layer.

### 3.2 TOTP Enforcement Flow

**Grace period:**

| Day since provisioning | Admin experience |
|---|---|
| Day 0–7 | Access allowed; persistent `⚠ Enroll MFA within N days` banner; `access.tenant_admin_totp_waiver_used` DEC-030 event emitted per invocation |
| Day 8+ | Admin-gated endpoints return `HTTP 403 MFA_ENROLLMENT_REQUIRED` |

**Enforcement middleware (Cloudflare Worker):**
```typescript
async function requireAdminMfa(request, env, claims) {
  if (!['tenant_owner', 'tenant_admin'].includes(claims.role)) return null;
  const enrolled = await isMfaEnrolled(claims.sub, env);
  if (enrolled) return null;
  const gracePeriodExpired = await isGracePeriodExpired(claims.sub, env);
  if (!gracePeriodExpired) {
    await emitDec030(env, { event_type: 'access.tenant_admin_totp_waiver_used', ... });
    return null;
  }
  return new Response(JSON.stringify({
    error: 'MFA_ENROLLMENT_REQUIRED',
    enroll_url: `${env.APP_URL}/settings/security/mfa`
  }), { status: 403 });
}
```

### 3.3 SSO Tenant Exemption

Enterprise tenants using SAML 2.0 or OIDC SSO may claim a TOTP exemption if **all three** conditions are met:
1. `tenant_sso_configs.sso_enabled = true`
2. SAML assertion or OIDC ID token includes qualifying auth context: `TimeSyncToken`, `X509Certificate`, or `acr_values = mfa` (note: `PasswordProtectedTransport` does **not** qualify)
3. Tenant IT admin provides written attestation at `compliance/enterprise-tenants/{slug}/mfa-idp-attestation.md`

**Evidence artefact CC6-E-002:**
```
File: compliance/evidence/cc6/tenant-admin-totp-enforcement-YYYY-MM.md
Content: Deployment confirmation (Worker commit SHA); count of tenant_owner/admin
         users with verified mfa_factors; count of totp_waiver events in quarter;
         list of tenants with mfa_delegated_to_idp = true + attestation reference
Cadence: Generated at each quarterly access review (§23)
Retention: 7 years
```

---

## 4. SCIM User Deprovisioning — CC6-GAP-004

**Policy:** The `DELETE /scim/v2/Users/{userId}` endpoint must be implemented as a full lifecycle handler. Without it, terminated employees remain in an active session state until manually removed, violating CC6.3 and breaking the offboarding SLA evidence trail.

**Endpoint behaviour:**

| Status | Condition |
|---|---|
| `204 No Content` | Successful deprovision, or user already deprovisioned (idempotent) |
| `404 Not Found` | User ID does not exist in this tenant |
| `409 Conflict` | Request would remove the last `tenant_owner` of a tenant |

**Required actions (atomic SQL transaction):**
1. Soft-delete `tenant_users` row (`active = FALSE`, `deprovisioned_at = NOW()`)
2. Release seat allocation (`tenant_contracts.seats_active - 1`)
3. Invalidate all active sessions: `supabase.auth.admin.signOut(userId, 'others')`
4. Emit `auth.scim_user_deprovisioned` DEC-030 event (HIGH, 7yr) with `scim_user_id`, `tenant_id`, `seat_release_count`

`DELETE /scim/v2/Groups/{groupId}` must be implemented simultaneously, cascading per-user deprovision.

**Evidence artefact CC6-E-003:**
```
File: compliance/evidence/cc6/scim-delete-integration-test-results-YYYY-MM.md
Content: Okta dev-tenant integration test log; idempotency test result;
         last-owner conflict test (expected 409); DEC-030 event chain excerpt
Cadence: On SCIM handler implementation; re-tested on any IdP config change
Retention: 7 years
```

---

## 5. CI Dependency Audit — CC6-GAP-005

**Policy:** Every pull request targeting `main` must pass `npm audit --audit-level=critical`. Critical CVEs block merge. A Dependabot configuration monitors all npm dependencies on a weekly cadence.

### 5.1 GitHub Actions — `dependency-audit` job

```yaml
dependency-audit:
  runs-on: ubuntu-latest
  steps:
    - uses: actions/checkout@v4
    - uses: actions/setup-node@v4
      with:
        node-version-file: .nvmrc
        cache: npm
    - run: npm ci --ignore-scripts
    - run: npm audit --audit-level=critical
```

**Exception bypass:** If a critical CVE has no fix available, `compliance-officer` files a waiver in `compliance/evidence/cc6/npm-audit-waiver-{CVE-ID}.md` with: CVE ID, package, affected version, CVSS score, no-fix rationale, mitigating control, and target remediation date. The `npm audit` command is then run with `--ignore-scan-packages={package}` only after the waiver is filed and approved in `compliance/policy-approval-log.csv`.

### 5.2 Dependabot Configuration

```yaml
# .github/dependabot.yml
version: 2
updates:
  - package-ecosystem: npm
    directory: /
    schedule:
      interval: weekly
      day: monday
      time: "09:00"
      timezone: UTC
    reviewers:
      - rbit27
    open-pull-requests-limit: 10
  - package-ecosystem: github-actions
    directory: /
    schedule:
      interval: weekly
      day: monday
      time: "09:00"
      timezone: UTC
    reviewers:
      - rbit27
```

**Evidence artefact CC6-E-004:**
```
File: compliance/evidence/cc6/dependabot-weekly-report-YYYY-WN.md
Content: Count of PRs opened; critical CVEs (0 = pass); high CVEs (1 week
         remediation SLA per docs/SOC2_READINESS.md §41); merged or waived
Cadence: Exported at each quarterly access review; any critical-CVE event triggers
         immediate incident triage per docs/INCIDENT_RESPONSE.md R-03
Retention: 7 years
```

---

## 6. Secret Scanning — CC6-GAP-006

**Policy:** No credential may be committed to the git repository. `git-secrets` patterns must block commits containing FORM-specific credential strings.

### 6.1 Pattern File

```bash
# .github/git-secrets-patterns
sk-ant-[a-zA-Z0-9\-_]{95}
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9\.[a-zA-Z0-9_\-]+\.[a-zA-Z0-9_\-]+
sk_live_[a-zA-Z0-9]{24,}
[a-zA-Z0-9_\-]{32}\.elevenlab
Bearer\s[a-zA-Z0-9\-._~+/]{40,}=*
[0-9a-f]{40}
```

### 6.2 CI Scan Step

```yaml
secret-scan:
  runs-on: ubuntu-latest
  steps:
    - uses: actions/checkout@v4
      with:
        fetch-depth: 0
    - name: Install git-secrets
      run: |
        brew install git-secrets 2>/dev/null || \
        (git clone https://github.com/awslabs/git-secrets.git && \
         cd git-secrets && sudo make install)
    - name: Register patterns
      run: git secrets --add-pattern-file .github/git-secrets-patterns
    - name: Scan history
      run: git secrets --scan-history
```

**Evidence artefact CC6-E-005:**
```
File: compliance/evidence/cc6/secret-scan-ci-log-YYYY-QN.md
Content: CI run URL; git-secrets version; pattern count; scan result (PASS/FAIL);
         any findings (if FAIL, remediation ticket reference)
Cadence: Filed at each quarterly access review; any FAIL triggers immediate rotation
         of the exposed credential (docs/INCIDENT_RESPONSE.md R-05)
Retention: 7 years
```

---

## 7. CSP Header and Static Analysis — CC6-GAP-007 (P1)

### 7.1 Content-Security-Policy Header

**CSP policy string:**
```
Content-Security-Policy:
  default-src 'self';
  script-src 'self' 'nonce-{NONCE}';
  style-src 'self' 'unsafe-inline' https://fonts.googleapis.com;
  font-src 'self' https://fonts.gstatic.com;
  img-src 'self' data: https:;
  connect-src 'self' https://api.form.coach https://*.supabase.co wss://*.supabase.co
              https://posthog.form.coach;
  frame-ancestors 'none';
  base-uri 'self';
  form-action 'self';
  upgrade-insecure-requests;
  block-all-mixed-content;
  report-uri /api/csp-report
```

Deployed via Cloudflare Transform Rule on `form.coach/*` and `admin.form.coach/*`.

### 7.2 ESLint Static Analysis

The following rules are required in all FORM TypeScript/JavaScript files:

```json
{
  "rules": {
    "no-eval": "error",
    "no-new-func": "error",
    "no-implied-eval": "error"
  }
}
```

CI step: `eslint . --max-warnings 0` as a required status check on `main`.

**Evidence artefact CC6-E-006:**
```
File: compliance/evidence/cc6/csp-deployment-YYYY-MM.md
Content: curl -I output showing CSP header live on form.coach; ESLint CI run URL
         confirming 0 eval-pattern violations
Cadence: On initial deployment; re-captured after any CSP policy change
Retention: 7 years
```

---

## 8. DEC-030 Audit Events

| Event type | Severity | Retention | Trigger |
|---|---|---|---|
| `access.tenant_admin_totp_enrolled` | HIGH | 7 yr | Tenant admin completes TOTP enrolment |
| `access.tenant_admin_totp_waiver_used` | STANDARD | 7 yr | Admin-gated endpoint accessed during 7-day grace period |
| `auth.scim_user_deprovisioned` | HIGH | 7 yr | `DELETE /scim/v2/Users/{id}` handler completes |
| `auth.scim_group_deprovisioned` | HIGH | 7 yr | `DELETE /scim/v2/Groups/{id}` handler completes |
| `security.dependency_audit_failed` | HIGH | 7 yr | `npm audit --audit-level=critical` exits non-zero on main branch |
| `security.secret_scan_finding` | CRITICAL | 7 yr | git-secrets pattern match detected in CI scan or pre-commit hook |

---

## 9. SOC 2 Criteria Mapping

| Criterion | How this policy satisfies it |
|---|---|
| **CC6.1 — Logical access identification and authentication** | GitHub org + Cloudflare MFA enforcement (§2); tenant admin TOTP (§3) |
| **CC6.2 — Prior registration and authorisation** | Tenant admin TOTP enrolment required before admin-gated endpoint access (§3.2) |
| **CC6.3 — Access modification and removal** | SCIM DELETE handler (§4) enables automated deprovisioning on IdP-triggered termination within the SSO-provisioned user lifecycle |
| **CC6.6 — Protection against external threats** | Secret scanning (§6) prevents credential leakage to VCS; git-secrets pre-commit hook as first-line defence |
| **CC6.8 — Prevention of unauthorised software** | CI dependency audit (§5) blocks critical-CVE dependency merges; CSP + ESLint (§7) prevents eval-based injection at application and browser layers |

---

## 10. Gap Closure Record

| Gap ID | Status | Closed by |
|---|---|---|
| **CC6-GAP-001** — GitHub org MFA not enforced | 🟡 AUTHORED (closes to 🟢 on §2.1 config + CC6-E-001a filed) | This policy §2.1 |
| **CC6-GAP-002** — Cloudflare MFA not enforced | 🟡 AUTHORED (closes to 🟢 on §2.2 config + CC6-E-001b filed) | This policy §2.2 |
| **CC6-GAP-003** — Tenant admin TOTP not enforced | 🟡 AUTHORED (closes to 🟢 on Worker middleware deploy + CC6-E-002 filed) | This policy §3 |
| **CC6-GAP-004** — SCIM DELETE handler missing | 🟡 AUTHORED (closes to 🟢 on handler implementation + integration test + CC6-E-003 filed) | This policy §4 |
| **CC6-GAP-005** — npm audit CI gate missing | 🟡 AUTHORED (closes to 🟢 on CI step merge + first main-branch run passes + CC6-E-004 filed) | This policy §5 |
| **CC6-GAP-006** — git-secrets pre-commit missing | 🟡 AUTHORED (closes to 🟢 on CI scan step runs on first PR + CC6-E-005 filed) | This policy §6 |
| **CC6-GAP-007** — CSP header and ESLint missing | 🟡 AUTHORED, P1 (closes to 🟢 on deployment + CC6-E-006 filed) | This policy §7 |

---

## 11. Implementation Checklist

| # | Task | Owner | Priority | Milestone |
|---|---|---|---|---|
| 1 | Enable "Require two-factor authentication" in GitHub org Security settings; file CC6-E-001a screenshot | security-engineer | **P0** | Pre-hire |
| 2 | Enable "Require 2FA for all members" in Cloudflare Account → Security; file CC6-E-001b screenshot | devops-lead | **P0** | Pre-launch |
| 3 | Add `dependency-audit` job to `.github/workflows/ci.yml` (§5.1 YAML); mark as required status check on `main` | devops-lead | **P0** | Pre-launch |
| 4 | Commit `.github/dependabot.yml` (§5.2 config); merge initial Dependabot PRs to clear baseline | devops-lead | **P0** | Pre-launch |
| 5 | Create `.github/git-secrets-patterns` with six FORM-specific patterns (§6.1); add to `docs/ENGINEERING_RUNBOOK.md §Security` | security-engineer | **P0** | Pre-launch |
| 6 | Add `secret-scan` CI job to `.github/workflows/ci.yml` (§6.2 YAML); mark as required status check | security-engineer | **P0** | Pre-launch |
| 7 | Implement `DELETE /scim/v2/Users/{userId}` handler (§4 spec); integration test against Okta dev tenant; file CC6-E-003 | platform-engineer | **P0** | M4 |
| 8 | Implement `DELETE /scim/v2/Groups/{groupId}` handler with cascading per-user deprovision (§4) | platform-engineer | **P0** | M4 |
| 9 | Implement `requireAdminMfa()` Worker middleware (§3.2 TypeScript spec); emit DEC-030 events; file CC6-E-002 | platform-engineer | **P1** | M3 |
| 10 | Register six DEC-030 event types (§8) in `docs/AUDIT_LOG_SCHEMA.md` | security-engineer | **P1** | M3 |
| 11 | Deploy CSP header via Cloudflare Transform Rule (§7.1 policy string); wire `/api/csp-report` Worker; file CC6-E-006 | platform-engineer | **P1** | M3 |
| 12 | Add ESLint `no-eval`, `no-new-func`, `no-implied-eval` to ESLint config; add CI lint step with `--max-warnings 0` (§7.2) | platform-engineer | **P1** | M3 |
| 13 | Record policy in `compliance/policy-approval-log.csv` with POL-015 row | compliance-officer | **P1** | M3 |

---

*Owner: security-engineer + devops-lead + compliance-officer · v1.0 · 2026-06-12 · Extracted from `docs/SOC2_READINESS.md §43` · Next review: 2027-01-01*

*Co-Authored-By: Claude (enterprise-builder) <cloud@form.coach>*
