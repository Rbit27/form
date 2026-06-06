# FORM Authorized Access Roster

**Version:** 1.0
**Authored:** 2026-06-05
**Phase:** Solo-founder (pre-hire)
**Owner:** compliance-officer
**Cross-reference:** `docs/SOC2_READINESS.md §23.5` (role specification), `§65.13 AR-P0-03` (P0 checklist item)
**SOC 2 criteria:** CC6.2, CC6.3

---

## Purpose

This file is the authoritative baseline for quarterly access reviews conducted per `docs/SOC2_READINESS.md §23`. During each review (Step 2), the reviewer compares the live account inventory against this roster. Any account not listed here is unauthorized by default and triggers the §23.4 Step 3 deprovisioning procedure.

This v1.0 constitutes the baseline established at the Q2 2026 access review (2026-06-05). It must be updated at every quarterly review when roles change.

---

## 1. Authorized Human Accounts

**Solo-founder phase:** The founder is the sole authorized human account holder across all systems. No additional human accounts are authorized. Any account belonging to a person other than the founder is unauthorized and must be deprovisioned within 24 hours (§23.4 Step 3).

| System | Account | Authorized role | Notes |
|---|---|---|---|
| GitHub | founder | Admin | Org owner and sole repository member |
| Cloudflare | founder | Super Admin | Sole account member; Zone admin for form.coach |
| Supabase | founder | Owner | Project owner; sole human with database access |
| 1Password | founder | Owner / Admin | Team admin; vault access to all FORM vaults |
| PostHog | founder | Admin | Project admin |
| Sentry | founder | Owner | Org owner |
| Stripe | founder | Admin | Account admin |
| ElevenLabs | founder | Admin | Account admin (TTS voice pipeline) |
| Anthropic | founder | Admin | Account admin (AI inference API keys) |
| Apple Developer | founder | Account Holder | App Store Connect account holder |
| Google Play | founder | Account owner | Google Play Console account owner |

**Independence caveat (§23.7):** During the solo-founder phase, the reviewer-independence requirement of CC6.3 cannot be met structurally. The compensating control (documentary evidence + HMAC-chained audit event) is documented in `docs/SOC2_READINESS.md §23.7`. This compensating control expires at the moment a second person is granted any production system access.

**Post-hire update trigger:** When any second person is granted production access, update this roster to v2.0 with the new role assignments before their first production action. The second person must independently countersign all future access review artifacts.

---

## 2. Authorized Service Accounts and API Tokens

Service accounts are scoped to least-privilege by design. Any service account with broader scope than listed here must be rotated and re-scoped within 24 hours.

| Credential / token | System (source → target) | Authorized scope | Rotation SLA | Notes |
|---|---|---|---|---|
| `SUPABASE_SERVICE_ROLE_JWT` | Supabase → Cloudflare Workers | Service role via API only — Workers only, never client-side | 90 days | §57 rotation runbook; HMAC key inventory in OBSERVABILITY §30 `form-crypto-health` KV |
| `ANTHROPIC_API_KEY` | Anthropic → Cloudflare Workers | AI coaching inference (claude-sonnet-4-6 endpoint) | 90 days | Scoped to FORM Workers only |
| `CLOUDFLARE_API_KEY` | Cloudflare → GitHub Actions | IaC drift detection (§50 drift detection workflow) | 180 days | CI-scoped, read-only for IaC audit |
| `WORKOS_API_KEY` | WorkOS → Cloudflare Workers | Enterprise SSO / SCIM provisioning endpoint only | 180 days | Pre-launch; zero active tenants; deactivated if SCIM migration occurs |
| `SENTRY_DSN` | Sentry → React Native app | Error reporting SDK (client-side, public) | 180 days | PII scrubbed at SDK layer per GDPR compensating control (§17.4) |
| `HMAC_AUDIT_CHAIN_KEY` | KMS → audit chain Worker | DEC-030 HMAC chain integrity signing (primary key) | Annual (dual-key rotation per §58) | Dual-key: primary + pending; chain continuity preserved during rotation |
| `KEYPOINTS_ENC_KEY` | KMS → CV pipeline Worker | Body keypoint encryption at rest (CRYPTOGRAPHY_POLICY §4) | 365 days | Health-category data; annual rotation aligned with GDPR Art. 9 control review |
| `SUPABASE_ANON_KEY` | Supabase → React Native app | Public RLS-gated access (anonymous read for public routes) | 365 days | Client-safe; RLS enforces tenant isolation |
| SCIM provisioning tokens (per enterprise tenant) | Supabase | SCIM v2 API endpoint only (`/v2/scim/`) — one token per active enterprise tenant | On tenant admin request or 12 months | 0 active tokens at v1.0 baseline (pre-launch) |
| GitHub Actions secrets | GitHub | CI pipeline execution (6 active secrets at v1.0 baseline) | 12 months | All 6 referenced by active workflows; orphan secret check performed at each quarterly review |
| Google Play service account key | Google Play → EAS submit | EAS Submit automation for Play Store releases | 12 months | |
| PostHog personal API key | PostHog → GitHub Actions CI | Analytics validation in CI pipeline | 12 months | |
| Cloudflare Tunnel credential | Cloudflare | Named tunnel for dev/staging environment | 12 months | Not used in production |
| Nightly backup Worker service key | Supabase | Read-only `pg_dump` export via restricted `backup_user` role | 90 days | `ALTER ROLE backup_user PASSWORD` rotation; confirmed via `pg_stat_activity` |

---

## 3. Unauthorized by Default

All of the following are unauthorized unless explicitly added to this roster:

- Any human account in any system not listed in §1
- Any service account with scope broader than listed in §2
- Any SCIM provisioning token for a tenant not listed in the current quarterly review artifact's enterprise tenant section
- Any personal API key for a system not listed in §2
- Any CI/CD token persisting beyond 30 days after the departure of the engineer who created it

---

## 4. Unauthorized Customers (No-Go List)

Per `docs/ENTERPRISE.md` and `docs/SOC2_READINESS.md §8` no-go customer policy, the following categories of enterprise customers are not authorized regardless of contract value:

- Insurance companies seeking to use FORM data for risk-scoring or underwriting
- Government entities requesting backdoor access or data disclosure outside legal process
- Wellness programmes structured to punish non-participants or tie metrics to insurance pricing
- Sports leagues seeking performance data for athlete evaluation without athlete consent
- Any entity whose use case requires compromising the privacy floor (`docs/SOC2_READINESS.md §21`)

These restrictions are enforced at the contract review stage by the compliance-officer and cannot be waived by a commercial approver alone.

---

## 5. Version History

| Version | Date | Author | Change summary |
|---|---|---|---|
| 1.0 | 2026-06-05 | compliance-officer (founder) | Initial baseline — solo-founder phase; 11 human accounts (founder only); 14 service account / API token rows; established at Q2 2026 access review first execution (SOC2_READINESS.md §65). Closes AR-P0-03. |

**Next update trigger:** First engineering or compliance hire (activate post-hire protocol above) **or** Q3 2026 quarterly access review (2026-07-31), whichever is first.

---

## 6. Cross-References

| Document | Relevant section | Purpose |
|---|---|---|
| `docs/SOC2_READINESS.md` | §23 | Full quarterly access review procedure |
| `docs/SOC2_READINESS.md` | §23.5 | Role specification (source of truth for authorized roles) |
| `docs/SOC2_READINESS.md` | §23.7 | Solo-founder compensating control for CC6.3 |
| `docs/SOC2_READINESS.md` | §23.8 | PRE-23 implementation checklist (item 2 closed by this file) |
| `docs/SOC2_READINESS.md` | §57 | `SUPABASE_SERVICE_ROLE_JWT` rotation runbook |
| `docs/SOC2_READINESS.md` | §58 | `HMAC_AUDIT_CHAIN_KEY` dual-key rotation procedure |
| `docs/SOC2_READINESS.md` | §65 | Q2 2026 access review first execution evidence |
| `docs/SOC2_READINESS.md` | §65.13 AR-P0-03 | P0 checklist item this file closes |
| `docs/AUDIT_LOG_SCHEMA.md` | DEC-030 `system.access_review_completed` | HMAC-chained event emitted on each review completion |
| `docs/OBSERVABILITY.md` | §30 `form-crypto-health` KV | Service account rotation state tracking |
| `docs/CRYPTOGRAPHY_POLICY.md` | §4 | Encryption key policy for `KEYPOINTS_ENC_KEY` |
| `docs/ENTERPRISE.md` | §Privacy floor | No-go customer categories (§4 above) |

---

*v1.0 (2026-06-05): Initial authorized access roster — solo-founder phase baseline. Establishes the authoritative role assignments for §23.5 quarterly access review comparisons. Authored as part of Q2 2026 access review first execution (SOC2_READINESS.md §65), closing AR-P0-03 (P0 checklist item, deadline 2026-06-07). §1 human accounts: founder is sole authorized human across all 11 systems (GitHub Admin, Cloudflare Super Admin, Supabase Owner, 1Password Owner/Admin, PostHog Admin, Sentry Owner, Stripe Admin, ElevenLabs Admin, Anthropic Admin, Apple Developer Account Holder, Google Play Account owner); §23.7 solo-founder independence compensating control noted. §2 service accounts: 14 entries covering `SUPABASE_SERVICE_ROLE_JWT` (90-day, Workers-only), `ANTHROPIC_API_KEY` (90-day), `CLOUDFLARE_API_KEY` (180-day, IaC drift CI), `WORKOS_API_KEY` (180-day, SSO pre-launch), `SENTRY_DSN` (180-day, client-safe with PII scrub), `HMAC_AUDIT_CHAIN_KEY` (annual dual-key), `KEYPOINTS_ENC_KEY` (365-day, health-category), `SUPABASE_ANON_KEY` (365-day, public RLS-gated), SCIM tokens (0 active pre-launch, 12-month SLA), GitHub Actions secrets (6 active, 12-month), Google Play service key (12-month EAS), PostHog personal key (12-month CI), Cloudflare Tunnel (12-month dev/staging), nightly backup Worker (90-day restricted role). §3 unauthorized-by-default categories enumerated. §4 no-go customer categories per ENTERPRISE.md. Next update trigger: first hire or Q3 2026 review. Closes: SOC2_READINESS.md §23.8 item 2, §65.13 AR-P0-03.*
