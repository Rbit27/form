# FORM · CC6 Logical and Physical Access Controls — Evidence Index

> SOC 2 Type II · Trust Service Criteria CC6 (Logical and Physical Access Controls).
> Owner: security-engineer + platform-engineer + compliance-officer. Review: quarterly.
> Reference: `docs/SOC2_READINESS.md §26`, `docs/SSO_SCIM_IMPLEMENTATION.md`, `compliance/access-review/`.

---

## CC6 Criteria Coverage

| Criterion | Description | Status |
|---|---|---|
| CC6.1 | Logical access security software, infrastructure, and architectures have been implemented to support (1) identification and authentication of authorized internal and external users; (2) restriction of authorized user access to the registered information assets on a need-to-know basis; (3) prevention and detection of unauthorized access | 🟡 Partial — Cloudflare Access + Supabase Auth + WorkOS SSO fully specified (§26.1–26.3); MFA gaps remain (CC6-GAP-001/002) |
| CC6.2 | Prior to issuing system credentials and granting system access, the entity registers and authorizes new internal and external users whose access is administered by the entity | 🟡 Partial — provisioning workflow fully specified; SCIM auto-provisioning designed (§SSO §4–5); TOTP enforcement for tenant admins pending (CC6-GAP-003) |
| CC6.3 | The entity authorizes, modifies, or removes access to data, software, functions, and other protected information assets based on approved and documented access requests, and reviews logical access controls to identify any inappropriate access | 🟢 Authored — offboarding-procedure.md (CC6-E-001) covers modification/removal; Q2 2026 access review completed (CC6-GAP-001 closed by §65) |
| CC6.4 | The entity restricts physical access to facilities and protected information assets to authorized personnel | 🟡 Partial — solo-founder; no shared office; device inventory in device-register.csv; compensating control narrative in §26.4 |
| CC6.5 | The entity discontinues logical access to protected information assets when the access is no longer required | 🟡 Partial — offboarding SLA ≤24h defined; SCIM `DELETE /Users` handler pending (CC6-GAP-004) |
| CC6.6 | The entity implements logical access security measures to protect against threats from sources outside its system boundaries | 🟡 Partial — Cloudflare WAF + rate limiting + DDoS protection + mTLS documented (§26.6); external API key management documented; break-glass procedure defined |
| CC6.7 | The entity restricts the transmission, movement, and removal of information to authorized internal and external users and processes, and protects it during transmission | 🟢 Authored — TLS 1.3 enforced; HSTS configured; data-in-transit encryption fully documented (§26.7; CRYPTOGRAPHY_POLICY.md); tenant isolation via RLS in DATA_MODEL.md |
| CC6.8 | The entity implements controls to prevent or detect and act upon the introduction of unauthorized or malicious software | 🟡 Partial — dependency scanning architecture fully specified; CI gate pending (CC6-GAP-005); git-secrets pre-commit pending (CC6-GAP-006); CSP header pending (CC6-GAP-007) |

---

## Evidence Files

| File | Description | SOC 2 Criteria | Status |
|---|---|---|---|
| `compliance/cc6/offboarding-procedure.md` (CC6-E-001) | Personnel offboarding checklist — credential revocation, system access removal, SLA ≤24h from departure; covers 10 systems + device return | CC6.3, CC6.5 | 🟢 Authored · 2026-05-22 · ready for first hire execution |
| `compliance/access-review/2026-q2/access-review-2026-Q2.md` | Q2 2026 Quarterly Access Review — first execution; 11 systems reviewed; 0 unauthorized accounts; 3 credentials rotated; CC6-GAP-001 closed | CC6.2, CC6.3, CC6.5, CC4.2 | 🟢 Executed · 2026-06-05 · §65 evidence |
| `docs/SOC2_READINESS.md §26` | Full CC6 policy: MFA enforcement matrix, network isolation (Cloudflare Access), logical access table for 6 admin surfaces, external threat controls, dependency scanning architecture, break-glass procedure, device management | CC6.1–CC6.8 | 🟡 Partial (gaps in checklist) |
| `docs/SSO_SCIM_IMPLEMENTATION.md` | SAML 2.0 + OIDC federation via WorkOS; SCIM 2.0 provisioning/deprovisioning design; JIT provisioning; tenant isolation; audit log integration | CC6.1, CC6.2, CC6.4, CC6.5 | 🟡 Partial — authored; SCIM delete handler pending (CC6-GAP-004) |
| `docs/DATA_MODEL.md` | Multi-tenant schema with RLS policies; tenant isolation guarantees; Row Level Security rules for all health data tables | CC6.4, CC6.7 | 🟢 Authored |
| `docs/CRYPTOGRAPHY_POLICY.md` (POL-002) | TLS 1.3 enforcement, HSTS, cipher suite restrictions, key rotation SLAs | CC6.7, C1.1 | 🟢 In Force · 2026-05-21 |
| `compliance/assets/device-register.csv` | Physical device inventory — all company-owned and production-credential-holding devices (FORM-DEV-XXX, FORM-MOB-XXX) | CC6.4 | 🔴 To create (MDD-P0-01) |
| _(pending)_ `compliance/evidence/cc6/prp-26-e-001.md` | MFA enforcement evidence — screenshot of GitHub org 2FA requirement + Cloudflare account-level MFA config | CC6.1 | 🔴 Gap — CC6-GAP-001/002 execution required |
| _(pending)_ `compliance/cc6/trufflehog-results/` | TruffleHog CI scan results — per-PR log of verified credential scans | CC6.8 | 🔴 Gap — CC6-GAP-006; not yet implemented |

---

## Gap Status

| Gap ID | Item | Priority | Status | Closed by |
|---|---|---|---|---|
| **CC6-GAP-001** | GitHub org "require 2FA for all members" enforcement | P0 — blocks CC6.1 MFA from 🟡 → 🟢 | 🟡 Partial → Q2 access review executed (§65); GitHub 2FA enforcement still pending | Enable in GitHub org settings; file PRE-26-E-001 screenshot |
| **CC6-GAP-002** | Cloudflare account-level MFA requirement | P0 — completes MFA enforcement matrix | 🔴 Open | Account Settings → Authentication → Require 2FA; file PRE-26-E-001 |
| **CC6-GAP-003** | Tenant admin TOTP enforcement at application layer | P0 — closes CC6.2 for admin roles | 🔴 Open | Supabase Auth TOTP verification in middleware before any `tenant.*` write |
| **CC6-GAP-004** | SCIM `DELETE /Users` deprovisioning handler in Edge Function | P0 — closes CC6.5 at technical layer | 🔴 Open | Spec in `docs/SSO_SCIM_IMPLEMENTATION.md §5.4`; calls `auth.admin.deleteUser` |
| **CC6-GAP-005** | `npm audit --audit-level=critical` in CI + Dependabot config | P0 — closes CC6.8 dependency scanning | 🔴 Open | `.github/dependabot.yml` + CI workflow update |
| **CC6-GAP-006** | `git-secrets` pre-commit hook for credential pattern scanning | P0 — prevents credential leakage | 🔴 Open | `pre-commit` or husky; patterns: Supabase, Stripe, Anthropic, ElevenLabs |
| **CC6-GAP-007** | CSP header via Cloudflare Transform Rule + ESLint `no-eval` | P1 — closes CC6.8 malicious-software angle | 🔴 Open | Cloudflare Transform Rule; ESLint config in CI |
| **CC6-GAP-008** | Tailwind CDN SRI evaluation — self-host or pin version | P1 — SRI gap (CDN does not publish SRI hashes) | 🔴 Open | Build pipeline: `npm install tailwindcss`, serve from `form.coach/assets/` |
| **CC6-GAP-009** | Dependabot alerts for `high` CVEs (in addition to critical) | P1 | 🔴 Open | GitHub Dependabot webhook → Linear ticket automation |
| **CC6-GAP-010** | MDM deployment (Jamf Now ≤3 devices, Kandji at scale) | P1 — closes device management + disposal gaps | 🔴 Open | Jamf Now free tier; enrol FORM-DEV-* and FORM-MOB-* devices |
| **CC6-GAP-011** | File PRE-26-E-001 through PRE-26-E-004 first-time evidence artefacts | P1 — evidence filing | 🔴 Open | No technical dependency; requires CC6-GAP-001/002 execution first |
| **CC6-GAP-012** | Cloudflare Business plan for Page Shield monitoring | P2 — CC6.8 external script monitoring | 🔴 Open | ~$200/mo; evaluate at enterprise GA |
| **CC6-GAP-013** | Break-glass `access.break_glass_initiated` / `access.break_glass_expired` Linear automation | P2 — audit trail completeness | 🔴 Open | Audit log webhook → Linear CC6/Break-Glass/ ticket |
| **CC6-GAP-014** | API key rotation calendar reminders at T-30d, T-7d in §15 | P2 | 🔴 Open | `docs/SOC2_READINESS.md §15` calendar update |

---

## Compliance Calendar

| Quarter | Task | Artefact |
|---|---|---|
| Q1 (Jan) | Quarterly access review — enumerate all 11 systems + service accounts; deprovision any stale access; file to `compliance/access-review/` | `access-review-YYYY-Q1.md` |
| Q2 (Apr 30) | Quarterly access review — **CC6-GAP-001 note:** due date was 30 April 2026; first execution was 36 days late (AR-2026-Q2-01 finding); Q3 gate set for 2026-07-31 | `access-review-2026-Q3.md` |
| Q3 (Jul 31) | Quarterly access review — target on-time execution; Sentry DPA closure target by 2026-06-12 | `access-review-2026-Q3.md` |
| Q4 (Oct 31) | Quarterly access review | `access-review-2026-Q4.md` |
| Annual (Jan) | Policy review — re-review SSO_SCIM_IMPLEMENTATION.md, offboarding-procedure.md, §26 | Updated `policy-approval-log.csv` rows |
| On every hire | Execute offboarding-procedure.md template on their eventual departure; execute provisioning checklist on onboarding | `compliance/evidence/cc6/onboarding/` and `offboarding/` |
| On any SCIM deprovisioning event | File `auth.sso_deprovisioned` audit event to HMAC chain | DEC-030 `auth.sso_deprovisioned` event |
| On any break-glass event | File break-glass report within 24h | `compliance/evidence/cc6/break-glass/YYYY-MM-DD.md` |

---

## Open Action Items

| ID | Action | Owner | Priority |
|---|---|---|---|
| CC6-P0-001 | Execute CC6-GAP-001: enable GitHub org "require 2FA for all members"; screenshot and file as PRE-26-E-001 | security-engineer | P0 — before observation period |
| CC6-P0-002 | Execute CC6-GAP-002: enable Cloudflare account-level MFA; document in PRE-26-E-001 alongside GitHub | devops-lead | P0 — before observation period |
| CC6-P0-003 | Implement CC6-GAP-004: SCIM `DELETE /Users` handler in Edge Function per `docs/SSO_SCIM_IMPLEMENTATION.md §5.4` | platform-engineer | P0 — before first enterprise pilot |
| CC6-P0-004 | Implement CC6-GAP-005: add `npm audit --audit-level=critical` CI step + `.github/dependabot.yml` | devops-lead | P0 — before observation period |
| CC6-P0-005 | Implement CC6-GAP-006: install `git-secrets` pre-commit hook; add to `README.md` dev setup | security-engineer | P0 — before observation period |
| CC6-P1-006 | Implement CC6-GAP-007: CSP header in Cloudflare Transform Rule + ESLint `no-eval` in CI | platform-engineer | P1 — before observation period |
| CC6-P1-007 | Deploy MDM (CC6-GAP-010): Jamf Now free tier; enrol all active FORM-DEV-* devices; screenshot enrollment | compliance-officer | P1 — before observation period |
| CC6-P1-008 | File PRE-26-E-001 through PRE-26-E-004 after CC6-GAP-001/002 are executed | compliance-officer | P1 — within 5 days of executing P0 items |
| CC6-P2-009 | Evaluate Tailwind CDN self-hosting (CC6-GAP-008): pin version in `package.json`, serve from `form.coach/assets/` | platform-engineer | P2 — before GA |
| CC6-P2-010 | Add Q3 quarterly access review calendar gate (due 2026-07-31) to prevent recurrence of 36-day latency finding AR-2026-Q2-01 | compliance-officer | P2 — set by 2026-06-14 |

---

*v1.0 · 2026-06-07 · Owner: security-engineer + platform-engineer + compliance-officer*
*Gaps open: CC6-GAP-001 through CC6-GAP-014 (13 items; P0: 6, P1: 4, P2: 4)*
*CC6-GAP-001 partially progressed: Q2 2026 access review completed (§65); GitHub 2FA enforcement still pending execution*
*Cross-ref: `docs/SOC2_READINESS.md §26` (full CC6 policy narrative); `compliance/access-review/2026-q2/` (Q2 evidence); `docs/SSO_SCIM_IMPLEMENTATION.md` (provisioning design)*
