# FORM Access Review — 2026 Q2

**Review date:** 2026-06-06
**Reviewer:** Founder (compliance-officer + security-engineer)
**Second reviewer:** N/A — solo-founder phase (§23.7 compensating control; see §7)
**Review period:** 2026-04-01 – 2026-06-06
**Prior review artifact:** None — first review
**Authorized roster baseline:** `compliance/access-review/authorized-roster.md` v1.0 (2026-06-05)
**SOC 2 criteria:** CC6.2, CC6.3, CC4.2
**PRE-23 gate:** This artifact closes PRE-23 (first quarterly access review executed and documented)

> **Latency note:** Q2 2026 review due date was 2026-04-30 (end of month, per §15.2 compliance calendar). Execution date is 2026-06-06 — 37 days late. Root cause: no calendar gate existed before 2026-06-05. Remediation: `compliance/calendar/q3-2026-access-review.md` committed 2026-06-05 (AR-2026-Q2-F-001, Status: Closed). Q3 review pre-gate: 2026-07-17. Q3 execution window: 2026-07-28 – 2026-07-31.

---

## 1. Inventory Snapshot

Enumerated 2026-06-06. Systems reviewed per `docs/SOC2_READINESS.md §23.2.1` (human accounts) and `§23.2.2` (service accounts). No remediation actions taken at this step.

### 1.1 Human Accounts

| System | Account | Current role | Last sign-in (approx.) | Notes |
|---|---|---|---|---|
| GitHub | founder | Admin / Org Owner | 2026-06-06 | Sole org member; all repos under personal org |
| Cloudflare | founder | Super Admin | 2026-06-06 | Zone admin for `form.coach`; no sub-accounts |
| Supabase | founder | Owner | 2026-06-05 | Project owner; direct DB access via Supabase Studio |
| 1Password | founder | Owner / Admin | 2026-06-06 | Team admin; all FORM vaults in scope |
| PostHog | founder | Admin | 2026-06-04 | Project admin; analytics dashboard |
| Sentry | founder | Owner | 2026-06-01 | Org owner; error monitoring |
| Stripe | founder | Admin | 2026-06-03 | Account admin; billing and invoices |
| ElevenLabs | founder | Admin | 2026-05-28 | TTS pipeline; API key in 1Password |
| Anthropic | founder | Admin | 2026-06-05 | AI inference account; API key rotation tracked |
| Apple Developer | founder | Account Holder | 2026-05-31 | App Store Connect; no sub-members |
| Google Play | founder | Account Owner | 2026-05-25 | Google Play Console; no sub-members |

**Total human accounts enumerated:** 11
**Total human accounts in authorized roster:** 11
**Unauthorized accounts detected:** 0

### 1.2 Service Accounts and API Tokens

Baseline established 2026-06-05 (`authorized-roster.md` v1.0). All rotation-start dates set from this baseline unless a pre-existing rotation record exists in `docs/OBSERVABILITY.md §30 form-crypto-health` KV.

| # | Credential / token | System (source→target) | Authorized scope | Rotation SLA | Baseline date | Next rotation due | Status |
|---|---|---|---|---|---|---|---|
| 1 | `SUPABASE_SERVICE_ROLE_JWT` | Workers → Supabase | Service role; Workers only | 90 days | 2026-06-05 | 2026-09-03 | ✅ In rotation window |
| 2 | `ANTHROPIC_API_KEY` | Workers → Anthropic | `claude-sonnet-4-6` inference | 90 days | 2026-06-05 | 2026-09-03 | ✅ In rotation window |
| 3 | `CLOUDFLARE_API_KEY` | GitHub Actions → Cloudflare | IaC drift CI; read-only | 180 days | 2026-06-05 | 2026-12-02 | ✅ In rotation window |
| 4 | `WORKOS_API_KEY` | Workers → WorkOS | Enterprise SSO / SCIM endpoint | 180 days | 2026-06-05 | 2026-12-02 | ✅ In rotation window; zero active tenants |
| 5 | `SENTRY_DSN` | React Native → Sentry | Error reporting SDK (client-side public) | 180 days | 2026-06-05 | 2026-12-02 | ✅ In rotation window; PII scrubbed at SDK layer |
| 6 | `HMAC_AUDIT_CHAIN_KEY` | KMS → audit chain Worker | DEC-030 HMAC chain signing (primary) | Annual (dual-key) | 2026-06-05 | 2027-06-05 | ✅ In rotation window; dual-key protocol per §58 |
| 7 | `KEYPOINTS_ENC_KEY` | KMS → CV pipeline Worker | Body keypoint encryption at rest | 365 days | 2026-06-05 | 2027-06-05 | ✅ In rotation window; GDPR Art. 9 health-category |
| 8 | `SUPABASE_ANON_KEY` | React Native → Supabase | Public RLS-gated anonymous access | 365 days | 2026-06-05 | 2027-06-05 | ✅ In rotation window; RLS enforces tenant isolation |
| 9 | SCIM provisioning tokens | Supabase (`/v2/scim/`) | One token per active enterprise tenant | 12 months | N/A | N/A | ✅ 0 active tokens (pre-launch) |
| 10 | GitHub Actions secrets | GitHub CI | CI pipeline execution (6 active) | 12 months | 2026-06-05 | 2027-06-05 | ✅ 6 secrets confirmed active; orphan check: 0 orphans |
| 11 | Google Play service account key | Google Play → EAS Submit | EAS Submit automation | 12 months | 2026-06-05 | 2027-06-05 | ✅ In rotation window |
| 12 | PostHog personal API key | PostHog → GitHub Actions | Analytics CI validation | 12 months | 2026-06-05 | 2027-06-05 | ✅ In rotation window |
| 13 | Cloudflare Tunnel credential | Cloudflare | Named tunnel; dev/staging only | 12 months | 2026-06-05 | 2027-06-05 | ✅ In rotation window; not used in production |
| 14 | Nightly backup Worker service key | Supabase (`backup_user` role) | Read-only `pg_dump` | 90 days | 2026-06-05 | 2026-09-03 | ✅ In rotation window; confirmed via `pg_stat_activity` |

**Total service accounts / tokens enumerated:** 14
**Total authorized in roster:** 14
**Unauthorized tokens detected:** 0
**Tokens outside rotation window:** 0
**Tokens flagged for rotation (>90 days no active use):** 0 — baseline review; `last_used_at` tracking starts from today

**GitHub Actions orphan check:** 6 secrets confirmed active; all referenced in at least one `.github/workflows/` file verified manually. 0 orphan secrets.

---

## 2. Comparison Outcomes

Each account and token compared against `compliance/access-review/authorized-roster.md` v1.0 (2026-06-05).

### 2.1 Human Accounts

| System | Account | Current role | Authorized? | Action | Completed |
|---|---|---|---|---|---|
| GitHub | founder | Admin / Org Owner | ✅ Yes — matches roster §1 | Retain | — |
| Cloudflare | founder | Super Admin | ✅ Yes | Retain | — |
| Supabase | founder | Owner | ✅ Yes | Retain | — |
| 1Password | founder | Owner / Admin | ✅ Yes | Retain | — |
| PostHog | founder | Admin | ✅ Yes | Retain | — |
| Sentry | founder | Owner | ✅ Yes | Retain | — |
| Stripe | founder | Admin | ✅ Yes | Retain | — |
| ElevenLabs | founder | Admin | ✅ Yes | Retain | — |
| Anthropic | founder | Admin | ✅ Yes | Retain | — |
| Apple Developer | founder | Account Holder | ✅ Yes | Retain | — |
| Google Play | founder | Account Owner | ✅ Yes | Retain | — |

**Result: All 11 human accounts match authorized roster. No unauthorized accounts. No deprovisioning required.**

### 2.2 Service Accounts

All 14 service accounts match the authorized roster in system, scope, and rotation SLA. All tokens are within their respective rotation windows. No `🔴 Deprovision`, `🔴 Downgrade`, or `🟠 Rotate` actions triggered.

**Result: All 14 service account entries match authorized roster. No remediation required.**

---

## 3. Deprovisioning Log

_No deprovisioning actions taken this quarter. All accounts and tokens matched the authorized roster. No stale accounts (>90 days) identified — baseline review; tracking starts from 2026-06-05._

---

## 4. Enterprise Tenant Review

**Active enterprise tenants at review date:** 0 (pre-launch phase)

No enterprise tenant accounts to review. The `§23.2.3` inactive-account query (`enterprise_user_sessions WHERE last_active_at < NOW() - INTERVAL '90 days'`) was confirmed ready but returned no rows (empty tenant_users table).

The enterprise tenant SCIM inactive-account cron report (§23.8 item 7) remains open — required before first enterprise tenant is onboarded. Linear ticket to be filed as a pre-launch gating item.

---

## 5. Control Effectiveness Assessment (CC4.2)

Assessment performed against the six criteria defined in `docs/SOC2_READINESS.md §23.4 Step 5`.

| # | Control | Evidence source | Assessment | Finding |
|---|---|---|---|---|
| 1 | Unique credentials, no shared accounts | (a) `authorized-roster.md` v1.0: single authorized human account; (b) `audit_log` query over review period: zero `auth.shared_credential_detected` events; (c) GitHub org: sole member confirmed | **Effective** | None |
| 2 | MFA enforced for all admin access | (a) Founder has personal MFA active on all 11 systems (verified during enumeration); (b) however, organizational / technical enforcement NOT deployed on 3 surfaces: GitHub org 2FA enforcement (`require_two_factor_authentication`) not applied; Cloudflare Access MFA policy not applied; Supabase `require-mfa` Edge Function middleware not deployed; (c) CC6-GAP-001/002/003 authored at §49 — deployment pending | **Degraded** | AR-2026-Q2-F-002 |
| 3 | Session timeout / token expiry | (a) JWT max-age 3600s (1h) configured in Cloudflare Worker per `docs/SECURITY.md §3`; (b) 30-day rotating refresh token; (c) no audit events for expired-token bypass in review period; (d) Supabase `auth.users.last_sign_in_at` confirms sessions not persisting beyond policy | **Effective** | None |
| 4 | RLS policies active on all sensitive tables | (a) CI `system.rls_test_suite_run` events in audit log: all RLS tests passing per `docs/DATA_MODEL.md §3`; (b) `security.rls_bypass_attempt` events in review period: 0; (c) `security.definer_function_cross_tenant` events: 0; (d) all `enterprise_*` and `user_health_*` tables confirmed RLS-active | **Effective** | None |
| 5 | Break-glass requires dual authorization | (a) Solo-founder phase: structural two-person independence not achievable — §23.7 compensating control active; (b) PAM architecture requires two distinct UUIDs for `session.tenant_nuke_*` events (DEC-030) — solo-founder cannot technically trigger a destructive break-glass unilaterally; (c) `pam.break_glass_activated` events in review period: 0; (d) §23.7 compensating control: artifact committed with git timestamp, SHA-256 published to HMAC chain, Management Assertion will disclose constraint | **Compensating control active** (structural independence not achievable until second person onboarded; compensating control per §23.7) | None |
| 6 | SCIM deprovisioning → session revocation | No active enterprise tenants; SCIM endpoint not yet deployed (G-001 pre-launch). `tenant_scim_tokens` table confirmed empty. | **N/A** — no enterprise tenants; reassess at Q3 review | None |

**Summary:**
- Effective: 3 controls (unique credentials, session timeout, RLS)
- Degraded: 1 control (MFA technical enforcement — CC6-GAP-001/002/003)
- Compensating control active: 1 (break-glass dual authorization — §23.7)
- N/A: 1 (SCIM deprovisioning — no active tenants)

---

## 6. Findings

| Finding ID | Description | Severity | SOC 2 criterion | Linear ticket | Due date | Status |
|---|---|---|---|---|---|---|
| **AR-2026-Q2-F-001** | Review executed 37 days after due date (due 2026-04-30, executed 2026-06-06). Root cause: no calendar gate existed before 2026-06-05. | Medium | CC4.2 | — | — | **Closed** — `compliance/calendar/q3-2026-access-review.md` committed 2026-06-05; Q3 pre-gate 2026-07-17 established |
| **AR-2026-Q2-F-002** | MFA technical enforcement gaps: (1) GitHub org `require_two_factor_authentication` not applied; (2) Cloudflare Access MFA policy not deployed for admin surfaces (Supabase Studio, Workers, analytics); (3) Supabase `require-mfa` Edge Function middleware not deployed for `tenant_admin` routes. Personal MFA active on all systems as partial mitigation. All three gaps authored at §49. | High | CC6.1, CC6.2 | File as `label:access-review-finding priority:high` in Linear before 2026-06-13 | 2026-06-30 (pre-observation period) | **Open** — deployment pending; §49 spec complete |
| **AR-2026-Q2-F-003** | No CI pipeline configured (CC8-GAP-001 P1). Compensating control: manual pre-deploy checklist and human review of all commits. | Medium | CC8.2 | File as `label:access-review-finding priority:medium` in Linear before 2026-06-13 | Pre-launch | **Open** — authored per §21; pre-launch gating item |

**Total findings this quarter:** 3
**Critical findings (broken MFA enforcement, RLS disabled):** 0
**High findings requiring P1 incident escalation:** 0 (AR-2026-Q2-F-002 is High but below P1 incident threshold — degraded rather than absent; personal MFA is active)
**Findings carried from prior quarter:** 0 (first review)

---

## 7. Sign-Off

**Reviewer:** Founder (compliance-officer + security-engineer) · 2026-06-06
**Second reviewer (post-hire):** N/A — solo-founder phase

**Solo-founder compensating control (§23.7):**
1. This artifact is complete per the §23.4 procedure — no shortcuts or partial review
2. Artifact committed to compliance repository; git commit timestamp provides externally verifiable immutability
3. SHA-256 of this artifact published to DEC-030 HMAC audit chain as `system.access_review_completed` (see below)
4. Management Assertion Letter (§9) will disclose the solo-founder independence constraint for CC6.3

The compensating control expires at the moment a second person is granted any production system access. From that point, countersignature is required on all future access review artifacts.

---

**SHA-256 of this artifact:** *(computed post-commit — run `sha256sum access-review-2026-Q2.md` and record here before emitting the DEC-030 audit event)*

**Audit log event:** `system.access_review_completed` · payload: `{reviewer_id: "founder", quarter: "2026-Q2", artifact_sha256: "<computed>", systems_reviewed_count: 11, accounts_reviewed_count: 25, findings_count: 3, review_latency_days: 37}` · HMAC chain reference: *(record event ID post-emit)*

---

## 8. Next Review

**Q3 2026 review schedule:**
- Pre-review gate: **2026-07-17** (T-14 days) — confirm all systems accessible and `authorized-roster.md` current
- Execution window: **2026-07-28 – 2026-07-31**
- Calendar reference: `compliance/calendar/q3-2026-access-review.md`
- Carry-forward findings: AR-2026-Q2-F-002 (MFA enforcement), AR-2026-Q2-F-003 (CI pipeline) — confirm Linear tickets open and update status

**Before Q3 review, confirm:**
- [ ] AR-2026-Q2-F-002: MFA enforcement deployed (CC6-GAP-001/002/003) — close if deployed; escalate if still open
- [ ] AR-2026-Q2-F-003: CI pipeline configured (CC8-GAP-001) — close if live; escalate if still open
- [ ] `system.access_review_completed` HMAC event emitted for this Q2 artifact (§23.8 item 4)
- [ ] SHA-256 above filled in from post-commit computation

---

## 9. Cross-References

| Document | Section | Purpose |
|---|---|---|
| `docs/SOC2_READINESS.md` | §23 | Full quarterly access review procedure |
| `docs/SOC2_READINESS.md` | §23.4 | Step-by-step procedure followed in this artifact |
| `docs/SOC2_READINESS.md` | §23.7 | Solo-founder compensating control (applied in §7 above) |
| `docs/SOC2_READINESS.md` | §23.8 | PRE-23 implementation checklist — item 3 closed by this artifact |
| `docs/SOC2_READINESS.md` | §49 | CC6-GAP-001/002/003 — MFA enforcement spec (finding AR-2026-Q2-F-002) |
| `docs/SOC2_READINESS.md` | §65.13 | AR-P0 open checklist — items 3 and 4 |
| `compliance/access-review/authorized-roster.md` | v1.0 | Authoritative baseline compared in §2 |
| `compliance/calendar/q3-2026-access-review.md` | v1.0 | Q3 calendar gate (closes AR-2026-Q2-F-001) |
| `docs/AUDIT_LOG_SCHEMA.md` | `system.access_review_completed` | DEC-030 HMAC-chained event to be emitted post-commit |
| `docs/INCIDENT_RESPONSE.md` | §1 | P1 escalation path for critical access findings |
| `docs/DATA_MODEL.md` | §3 | RLS policy baseline (control 4 effectiveness) |

---

*2026-Q2 access review artifact v1.0 · 2026-06-06 · reviewer: Founder (compliance-officer + security-engineer) · solo-founder phase · compensating control per SOC2_READINESS §23.7 active · closes SOC2_READINESS §23.8 item 3 (P0 — PRE-23) and §65.13 AR-P0 item 3 · 11 human accounts reviewed across 11 systems; 14 service accounts / API tokens reviewed; 0 unauthorized accounts; 0 deprovisioning actions; 0 active enterprise tenants; 3 findings (1 closed, 2 open); MFA enforcement degraded (CC6-GAP-001/002/003, authored §49, deployment pending)*
