# FORM · Audit Log Schema v1.4

> Що ми логуємо, як довго зберігаємо, хто може дивитись.
> Owner: `compliance-officer` + `security-engineer`. Reviewed quarterly.
> Required для SOC 2 Type II, ISO 27001, HIPAA tier (NA), GDPR Art. 30 records.

---

## TL;DR

Кожна privileged action у системі лишає immutable запис у `audit_log` table.
Логи `append-only`, `tamper-evident` (HMAC chain), `region-resident`.
Retention: **7 років** для SOC 2 / financial; **3 роки** для решти; **30 днів** для read-only access events (надмірно інакше).

---

## Schema

```sql
CREATE TABLE audit_log (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id       UUID NOT NULL,
  actor_id        UUID,                    -- user UUID або NULL для system
  actor_type      TEXT NOT NULL,           -- 'user', 'system', 'integration', 'support'
  actor_ip        INET,
  actor_ua        TEXT,
  action          TEXT NOT NULL,           -- canonical action name (див. taxonomy)
  resource_type   TEXT,                    -- 'user', 'workout', 'meal', 'org_setting', etc.
  resource_id     UUID,
  changes         JSONB,                   -- before/after для UPDATE; NULL для READ
  request_id      UUID,                    -- correlates з HTTP request trace
  session_id      UUID,
  outcome         TEXT NOT NULL,           -- 'success', 'denied', 'error'
  reason          TEXT,                    -- justification, error code, etc.
  metadata        JSONB,                   -- arbitrary additional context
  hmac_prev       BYTEA,                   -- chain integrity
  hmac_self       BYTEA NOT NULL,
  created_at      TIMESTAMPTZ NOT NULL DEFAULT now(),

  -- Partitioned by month для retention pruning
  PARTITION BY RANGE (created_at)
);

CREATE INDEX idx_audit_tenant_created ON audit_log (tenant_id, created_at DESC);
CREATE INDEX idx_audit_actor ON audit_log (actor_id, created_at DESC);
CREATE INDEX idx_audit_action ON audit_log (action, created_at DESC);
```

### HMAC chaining

Кожен новий рядок:

```
hmac_self = HMAC-SHA256(secret_key, hmac_prev || canonical_payload)
```

`secret_key` rotated quarterly, stored у KMS. Це дозволяє довести що **рядки не модифікувалися retroactively** — обчислюємо chain, порівнюємо з останнім hmac. Якщо break → tamper alert.

---

## Action taxonomy

### Auth events
- `auth.login` — успішний sign-in
- `auth.login_failed` — failed attempt (rate-limited, ip-tracked)
- `auth.logout`
- `auth.mfa_enabled` / `auth.mfa_disabled`
- `auth.password_reset_requested` / `auth.password_changed`
- `auth.sso_provisioned` — SCIM-created user
- `auth.session_expired`

### Tenant administration
- `tenant.created` (manual + self-serve)
- `tenant.settings_updated`
- `tenant.member_invited` / `tenant.member_removed`
- `tenant.role_changed` (ROLE_ADMIN ↔ ROLE_MEMBER)
- `tenant.sso_configured` (SAML / OIDC)
- `tenant.scim_provisioned` (bulk import)
- `tenant.billing_updated`
- `tenant.api_key_created` / `tenant.api_key_revoked`

### Data access (sensitive)
- `data.export_initiated` — GDPR/DSAR export started
- `data.export_completed`
- `data.bulk_deletion` (admin requests removing N users)
- `data.individual_deletion` (Right to be Forgotten)
- `data.read_aggregate` — admin reads aggregated wellness data
- `data.read_individual` — **forbidden** at HR tier; logged + alert if attempted

### Privacy-sensitive
- `privacy.consent_granted` / `privacy.consent_withdrawn`
- `privacy.cookies_changed`
- `privacy.analytics_opt_out`

### Privacy floor enforcement events (DEC-030 HMAC-chained · CRITICAL/HIGH/STANDARD severity · 7yr retention)

> Defined in `docs/INCIDENT_RESPONSE.md §R-22`. All five events are HMAC-chained. The two CRITICAL events (`privacy.floor_breach_detected`, `privacy.no_go_escalation_activated`) trigger immediate PagerDuty P0 and clinical-safety notification. **Privacy rule:** no individual user names or health values in any payload — `affected_employee_count` is aggregate; IR evidence references `ic_user_id` (UUID only). Failure to emit any CRITICAL or HIGH event aborts the corresponding response action (fail-closed). Cross-ref: ENTERPRISE.md §"Privacy floor for enterprise" (7 non-negotiable rules), DATA_MODEL.md §17 (aggregate-only admin reporting), SOC 2 P6.1/P6.7/P7.1/CC6.1/CC6.6/CC7.2/CC7.4.

- `privacy.floor_breach_detected` — privacy floor breach detected by FORM-PRIV-001/002/003/004 alerts; **must be emitted BEFORE any containment action**; payload: `{tenant_id, breach_source: 'api_serialiser'|'aggregate_query'|'cohort_label'|'code_review', scenario: 'A'|'B'|'C'|'D', clinical_safety_rules_violated: boolean, art9_confirmed: boolean, exposure_window_start: ISO8601, affected_employee_count: integer|'unknown_pending_assessment', ic_user_id: UUID, incident_slug: string}`; **CRITICAL severity, 7yr retention; HMAC-chained; PagerDuty P0 when `clinical_safety_rules_violated=true` or `art9_confirmed=true`**
- `privacy.floor_breach_contained` — breach contained (feature flag suppression, WAF rule, or session revoke applied); payload: `{tenant_id, containment_method: 'feature_flag'|'waf_rule'|'session_revoke', feature_flags_disabled: string[], contained_by_user_id: UUID, incident_slug: string}`; HIGH severity, 7yr retention; HMAC-chained; must follow `privacy.floor_breach_detected` in chain
- `privacy.floor_breach_tenant_notified` — enterprise tenant admin notified per Template PF-01 or employer controller notified per PF-02; **P0 SLA: ≤ 30 min after detection for PF-01**; payload: `{tenant_id, notification_template: 'PF-01'|'PF-02', notified_at: ISO8601, notified_by_user_id: UUID, employee_count_disclosed_to_tenant: integer, incident_slug: string}`; HIGH severity, 7yr retention; HMAC-chained
- `privacy.floor_breach_resolved` — breach fully resolved; root cause remediated; reporting re-enabled with 2-person approval including clinical-safety sign-off; payload: `{tenant_id, fix_commit_sha: string, fix_deployed_at: ISO8601, feature_flags_re_enabled: string[], clinical_safety_approved: boolean, two_person_approvers: UUID[], incident_slug: string}`; STANDARD severity, 7yr retention; HMAC-chained; must follow `privacy.floor_breach_contained` in chain
- `privacy.no_go_escalation_activated` — no-go escalation protocol activated: wellness-as-punishment or equivalent pattern detected (R-22.9); unconditional response regardless of root cause dispute; payload: `{tenant_id, no_go_reason: 'wellness_as_punishment'|'individual_data_demand'|'manager_report_demand'|'insurance_risk_scoring', dashboard_suspended_at: ISO8601, founder_notified_at: ISO8601, legal_counsel_engaged: boolean, incident_slug: string}`; **CRITICAL severity, 7yr retention; HMAC-chained; PagerDuty P0; clinical-safety VETO activated; triggers automatic global reporting suspension for affected tenant**

---

### Billing & GDPR erasure events (DEC-030 HMAC-chained · DATA_MODEL §30)

> Defined in `docs/DATA_MODEL.md §30` (OQ-BILL-05 resolution). One event covering the GDPR Art. 17 erasure record for `subscription_events` — the immutable audit trail that a billing row was pseudonymized during an erasure run. **Privacy invariant:** the event payload never contains the original `user_id` UUID — only the `erased_user_reference` keyed HMAC pseudonym (`[ERASED-{sha256(user_id + ERASURE_PSEUDONYM_SALT)}]`). The pseudonym is linkable only by an actor in possession of `ERASURE_PSEUDONYM_SALT` (write-once Cloudflare Workers Secret — see `docs/CRYPTOGRAPHY_POLICY.md §5/§6`). Cross-ref: SOC 2 P5.2 (pseudonymization evidence ERA-E-001), P8.0 (financial retention with `retention_basis` — ERA-E-002), CC8.1 (migration + erasure chain — ERA-E-003); GDPR Art. 17; DATA_MODEL §12 (Art. 17 erasure protocol step SUB-1); DATA_MODEL §30.7 items 3–5 (P0 M4 implementation checklist). Closes DATA_MODEL §30.7 item 4 (P0 M4 — patch `billing.user_erased` Zod schema in AUDIT_LOG_SCHEMA.md and deploy).

| Event type | Severity | Retention | Trigger | Key payload fields |
|---|---|---|---|---|
| `billing.user_erased` | HIGH | 7 yr | Erasure Worker step SUB-1 (`pseudonymizeSubscriptionEvents`) — fires once per user after all `subscription_events` rows for `user_id` are pseudonymized (DATA_MODEL §30.2.4); must be emitted within the same erasure transaction before committing | `erased_user_reference` (**required** — `[ERASED-{sha256(user_id + ERASURE_PSEUDONYM_SALT)}]` keyed HMAC pseudonym), `user_id` (**optional** — present only for pre-pseudonymization test events; **must be `null` / absent in all production erasure events**), `rows_pseudonymized` (integer — count of `subscription_events` rows updated in this run), `erasure_request_id` (UUID — FK to DSAR/Art. 17 request record), `pseudonymized_at` (ISO 8601) |

```typescript
// Zod schema v2 — canonical source (DATA_MODEL §30.5.2)
// Breaking change from v1: user_id is now optional; erased_user_reference is required.
// Existing pre-migration billing.user_erased events (v1, user_id present) remain valid
// against this schema — user_id: z.string().uuid().optional() accepts both forms.
const BillingUserErasedPayload = z.object({
  erased_user_reference: z.string().regex(/^\[ERASED-[0-9a-f]{64}\]$/),
  user_id: z.string().uuid().optional(),
  rows_pseudonymized: z.number().int().nonnegative(),
  erasure_request_id: z.string().uuid(),
  pseudonymized_at: z.string().datetime(),
}).refine(
  (d) => d.erased_user_reference !== undefined,
  { message: "erased_user_reference is required in all production erasure events" }
);
```

**HMAC chain requirement:** `billing.user_erased` must be emitted synchronously inside the erasure Worker transaction — failure aborts the erasure run (fail-closed). The event must carry `prev_hash`. No strict ordering relative to `data.individual_deletion` is enforced (the DSAR coordinator may emit `data.individual_deletion` before or after the erasure Worker run completes), but both events must reference the same `erasure_request_id` to allow chain cross-reference during SOC 2 fieldwork.

**Emitter assignment:** `erasure-worker` (`form_system`) — automated, fired by step SUB-1 of the erasure protocol. No manual emission path. The `emit-audit-event` Cloudflare Worker validates the Zod v2 schema before chain-appending; HTTP 422 on violation.

**Note on pre-migration events:** `billing.user_erased` events emitted before migration `0059` (`user_id NOT NULL` schema) carry `user_id` in the payload. The Zod v2 schema accommodates these via `user_id: z.string().uuid().optional()`. Post-migration events must omit `user_id` entirely. The weekly chain audit checks that any `billing.user_erased` event with a timestamp after migration `0059` promotion has `user_id = null` — presence of a non-null `user_id` in a post-migration event is flagged as a data-minimisation anomaly (PagerDuty P1 to compliance-officer).

---

### API Key credential events (DEC-030 HMAC-chained · HIGH severity · 7yr retention)

> Defined in `docs/SSO_SCIM_IMPLEMENTATION.md §26.6`. All 9 events are HIGH severity with 7-year retention. `*.ip_blocked` events carry `client_ip_hash` (HMAC-SHA256 of client IP with `IP_HASH_SALT`) — no plaintext IP ever enters the audit chain (DEC-030 privacy floor).

- `api_key.created` — new tenant API key created; payload: `{tenant_id, key_preview, scopes[], label, ip_enforcement_enabled, created_by}`
- `api_key.rotated` — key rotation initiated; payload: `{tenant_id, old_key_preview, new_key_preview, overlap_expires_at, rotated_by}`; **chain requirement:** must be followed by `api_key.revoked` (old key) within 26h or AL-APIKEY-02 fires
- `api_key.revoked` — key revoked; payload: `{tenant_id, key_preview, reason: 'manual'|'expiry_overlap'|'compromise'|'tenant_offboarding', revoked_by}`; when `reason = 'tenant_offboarding'` one event is emitted per active key for the departing tenant (bulk-sequenced, HMAC-chained in order) — see `docs/ENTERPRISE_ONBOARDING.md §11.1`
- `api_key.ip_enforcement_enabled` — IP allowlist enforcement turned on for a key; payload: `{tenant_id, key_preview, enabled_by, allowlist_cidr_count}`
- `api_key.ip_enforcement_disabled` — IP allowlist enforcement turned off; payload: `{tenant_id, key_preview, disabled_by}`
- `api_key.ip_blocked` — API key request blocked by IP allowlist; payload: `{tenant_id, key_preview, client_ip_hash}`; high-volume events bulk-deduplicated via Cloudflare Queues before chain insertion
- `scim.ip_enforcement_enabled` — SCIM IP allowlist enforcement turned on (`tenant_sso_configs.scim_ip_enforcement_enabled = true`); payload: `{tenant_id, enabled_by, allowlist_cidr_count}`
- `scim.ip_enforcement_disabled` — SCIM IP allowlist enforcement turned off; payload: `{tenant_id, disabled_by}`
- `scim.ip_blocked` — SCIM provisioning request blocked by IP allowlist; payload: `{tenant_id, client_ip_hash}`; bulk-deduplicated via Cloudflare Queues

> **Note:** `tenant.api_key_created` and `tenant.api_key_revoked` listed in the Tenant administration section above are consumer-tier shorthand entries. Enterprise-tier API key lifecycle is fully covered by the `api_key.*` namespace above, which provides rotation, IP enforcement, and block events not available in the consumer shorthand.

---

### SSO authentication policy events (DEC-030 HMAC-chained · SSO §25.9)

> Defined in `docs/SSO_SCIM_IMPLEMENTATION.md §25.9`. Nine events covering IP allowlist enforcement, MFA policy changes, and admin lockout recovery. Privacy floor: `sso.ip_blocked` fires before credential validation — `user_id` is omitted entirely; only `client_ip_hash` (SHA-256[:32] keyed with `IP_HASH_SALT` Workers Secret) is stored. `change_summary` in policy-change events must contain only CIDR count changes, not IP values. CRITICAL events (`sso.mfa_bypass_granted`, `sso.auth_policy_lockout_recovery`) trigger compliance-officer notification within 4h. Burst writes for `sso.ip_blocked` are smoothed through Cloudflare Queues (FIFO consumer) to maintain HMAC chain ordering under misconfiguration lockout scenarios. Cross-ref: SOC 2 CC6.1/CC6.2/CC6.8/CC7.2; closes SSO_SCIM_IMPLEMENTATION.md §25.14 checklist item 6 (P0 M4).

| Event type | Severity | Retention | Payload fields | Trigger |
|---|---|---|---|---|
| `sso.auth_policy_updated` | HIGH | 7 yr | `tenant_id`, `changed_by_user_id`, `policy_dimension` (`ip_allowlist` / `mfa_enforcement` / `session_policy`), `change_summary` (human-readable diff — CIDR count changes only, no IP values), `policy_version_before`, `policy_version_after` | Any `PATCH /v1/admin/sso/policy` resulting in a DB write |
| `sso.ip_allowlist_entry_added` | HIGH | 7 yr | `tenant_id`, `changed_by_user_id`, `entry_id` (UUID), `cidr_prefix_length` (e.g. `24` for /24 — prefix length only, never the full CIDR), `label` | CIDR added to `ip_allowlist_config.entries` |
| `sso.ip_allowlist_entry_removed` | HIGH | 7 yr | `tenant_id`, `changed_by_user_id`, `entry_id`, `cidr_prefix_length`, `label` | CIDR removed from `ip_allowlist_config.entries` |
| `sso.ip_allowlist_toggled` | HIGH | 7 yr | `tenant_id`, `changed_by_user_id`, `enabled_before` (bool), `enabled_after` (bool), `entry_count_at_toggle` | `ip_allowlist_config.enabled` changed |
| `sso.ip_blocked` | HIGH | 7 yr | `tenant_id`, `client_ip_hash` (SHA-256[:32] — **no `user_id`; emitted before credential validation**), `enforcement_point` (`saml_callback` / `oidc_callback` / `token_refresh` / `admin_api` / `scim_api`), `policy_version` | Request blocked by IP allowlist enforcement |
| `sso.mfa_enforcement_changed` | HIGH | 7 yr | `tenant_id`, `changed_by_user_id`, `mode_before` (`none` / `recommended` / `required`), `mode_after`, `mfa_grace_hours`, `estimated_sessions_affected` | `mfa_enforcement_mode` changed via `PATCH /v1/admin/sso/policy` |
| `sso.mfa_enforcement_sweep_completed` | STANDARD | 3 yr | `tenant_id`, `sessions_revoked`, `sessions_already_satisfied`, `sweep_triggered_at` (ISO 8601), `grace_expired_at` (ISO 8601) | MFA grace period ends; hourly sweep job runs |
| `sso.mfa_bypass_granted` | CRITICAL | 7 yr | `tenant_id`, `granted_by_user_id` (tenant_admin — FORM support cannot grant), `subject_user_id`, `bypass_duration_hours` (max 48), `reason` (free text, max 500 chars) | Tenant admin grants per-user MFA bypass via `PATCH /v1/admin/users/{id}/mfa-bypass` |
| `sso.auth_policy_lockout_recovery` | CRITICAL | 7 yr | `tenant_id`, `recovery_type` (`ip_allowlist_disabled` / `policy_reset`), `recovery_performed_by` (PAM session ID — not user_id; FORM-internal action), `pam_session_id`, `affected_config_field`, `customer_notified` (bool) | FORM support performs lockout recovery via PAM `read_write` elevation (§25.8.1) |

**HMAC chain requirement:** All nine events must include `prev_hash`. `sso.ip_blocked` is the highest-volume event in this family; under misconfiguration lockout scenarios it may emit hundreds per minute — use Cloudflare Queues (FIFO consumer) for block events to smooth write pressure while maintaining chain ordering. `sso.auth_policy_lockout_recovery` must reference the `pam_session_id` of the PAM elevation used to perform the recovery, linking the policy change to the privileged access record.

---

### PAM (Privileged Access Management) events (DEC-030 HMAC-chained · SSO §24.6)

> Defined in `docs/SSO_SCIM_IMPLEMENTATION.md §24.6` and `docs/DATA_MODEL.md §29.6`. Seven events covering JIT privilege escalation lifecycle, break-glass protocol, and mandatory post-hoc break-glass review. Privacy floor: `justification_hash` stores SHA-256 of the justification text — the original text is retained in `PAM_KV` (7yr TTL) but never enters the audit chain; `pam.break_glass_activated` uses `cf_access_jti_hash` (SHA-256 of the Cloudflare Access JTI — not the raw JTI). `target_tenant_id` is nullable (null means FORM infrastructure, not a customer tenant). CRITICAL event `pam.break_glass_activated` triggers immediate PagerDuty P0 (AL-PAM-01) and compliance-officer email. Break-glass sessions require a mandatory post-hoc two-person review within 72h, linked via `pam_session_id`; the review closure is recorded by `pam.break_glass_review_completed` (HIGH). Cross-ref: SOC 2 CC6.1/CC6.2/CC6.3/CC6.6/CC6.7; closes SSO_SCIM_IMPLEMENTATION.md §24.10 checklist item 8 (P1 M4) and DATA_MODEL.md §29.10 checklist item 6 (P0 M6).

| Event type | Severity | Retention | Two-person rule | Payload fields |
|---|---|---|---|---|
| `pam.elevation_requested` | HIGH | 7 yr | No | `admin_user_id`, `access_level` (`read_only` / `read_write` / `destructive`), `pam_request_id`, `target_tenant_id` (nullable), `justification_hash` (SHA-256) |
| `pam.elevation_approved` | HIGH | 7 yr | Pre-auth for `read_write`/`destructive` | `admin_user_id`, `approver_id` (nullable for `read_only`), `access_level`, `pam_session_id`, `expires_at` (ISO 8601), `requester_webauthn_credential_id` (nullable), `approver_webauthn_credential_id` (nullable) |
| `pam.elevation_denied` | HIGH | 7 yr | No | `admin_user_id`, `access_level`, `pam_request_id`, `reason` (`self_approval_attempted` / `approval_timeout` / `cosign_timeout` / `totp_failure` / `webauthn_failure` / `policy_violation`) |
| `pam.session_expired` | STANDARD | 7 yr | No | `admin_user_id`, `pam_session_id`, `access_level`, `expired_at` (ISO 8601) — emitted by `pam-expiry-sweeper`; distinct from `pam.elevation_denied` (session completed its full TTL) |
| `pam.break_glass_activated` | CRITICAL | 7 yr | Post-hoc (72h review, §24.4.3) | `admin_user_id`, `pam_session_id`, `cf_access_jti_hash` (SHA-256), `activated_at` (ISO 8601); triggers PagerDuty P0 + compliance email immediately on emission |
| `pam.break_glass_expired` | HIGH | 7 yr | No | `admin_user_id`, `pam_session_id`, `expired_at` (ISO 8601), `review_issue_url` (GitHub issue auto-created at activation) |
| `pam.break_glass_review_completed` | HIGH | 7 yr | Post-hoc (two-person sign-off, 72h SLA) | `pam_session_id`, `outcome` (`no_anomaly` / `anomaly_detected` / `process_gap`), `reviewed_by_security` (UUID), `reviewed_by_compliance` (UUID), `review_completed_at` (ISO 8601), `github_issue_url` (optional URL), `pg_audit_lines_reviewed` (optional integer) |

**HMAC chain requirement:** `pam.elevation_approved` must follow `pam.elevation_requested` with the same `pam_request_id` in the chain. If `pam.break_glass_activated` is emitted without a preceding `pam.elevation_requested` for the same session (break-glass is a separate Cloudflare Access path), this is expected — do not treat the missing predecessor as a chain violation. A post-hoc `pam.elevation_approved` is not emitted for break-glass; the `pam.break_glass_activated` CRITICAL event is the authorisation record.

The break-glass DEC-030 three-event chain is: `pam.break_glass_activated` → `pam.break_glass_expired` → `pam.break_glass_review_completed`. An auditor can confirm the full break-glass lifecycle by querying these three event types with the same `pam_session_id`. The chain is complete only when `pam.break_glass_review_completed` is emitted; AL-PAM-BG-01 (OBSERVABILITY.md §29.4) fires a PagerDuty P1 if `pam.break_glass_review_completed` is not emitted within 72h of `pam.break_glass_activated`.

**`pam.break_glass_review_completed` Zod schema** (canonical source: DATA_MODEL.md §29.6):

```typescript
import { z } from 'zod';

const PamBreakGlassReviewCompletedPayload = z.object({
  pam_session_id:           z.string().uuid(),
  outcome:                  z.enum(['no_anomaly', 'anomaly_detected', 'process_gap']),
  reviewed_by_security:     z.string().uuid(),
  reviewed_by_compliance:   z.string().uuid(),
  review_completed_at:      z.string().datetime(),
  github_issue_url:         z.string().url().optional(),
  pg_audit_lines_reviewed:  z.number().int().nonneg().optional(),
});
```

Emitter: compliance-officer via admin API (`POST /internal/pam/break-glass-reviews/:pam_session_id/complete`) when both `reviewed_by_security` and `reviewed_by_compliance` have signed off in the `pam_break_glass_reviews` Postgres table. `emit-audit-event` Worker validates the Zod schema before chain-appending; HTTP 422 on schema violation.

---

### Google Directory Sync events (DEC-030 HMAC-chained · SSO §21)

> Defined in `docs/SSO_SCIM_IMPLEMENTATION.md §21`. Eight events covering Google Workspace Directory API group sync lifecycle. Privacy floor: `user_email_hash` (SHA-256) is used in all events that reference individual users — plaintext email never enters the audit chain. Credential events include only `project_id` from the service account JSON (never the key material or private key fingerprint). High-volume sync events (`sso.google_directory_sync_success`, `sso.google_directory_sync_cache_hit`) are STANDARD severity and may be deduplicated within a 5-minute window per `(tenant_id, user_email_hash)` to manage write pressure while preserving at least one event per sync cycle. Cross-ref: SOC 2 CC6.1/CC6.3/CC9.2; closes SSO_SCIM_IMPLEMENTATION.md §21 checklist item 5 (P0 M4).

| Event type | Severity | Retention | Payload fields |
|---|---|---|---|
| `sso.google_directory_sync_success` | STANDARD | 7 yr | `tenant_id`, `user_email_hash` (SHA-256), `group_count`, `fetch_duration_ms`, `cache_hit: false` |
| `sso.google_directory_sync_cache_hit` | STANDARD | 7 yr | `tenant_id`, `user_email_hash` (SHA-256), `group_count`, `cache_age_seconds` |
| `sso.google_directory_sync_error` | HIGH | 7 yr | `tenant_id`, `user_email_hash` (SHA-256), `error_code`, `http_status`, `grace_window_used` (bool) — feeds AL-SSO-GDIR-01 error rate alert |
| `sso.google_directory_credential_uploaded` | HIGH | 7 yr | `tenant_id`, `actor_id`, `uploaded_by_role`, `project_id` (from service account JSON — **never the key or fingerprint**) |
| `sso.google_directory_credential_rotated` | HIGH | 7 yr | `tenant_id`, `actor_id`, `old_project_id`, `new_project_id` |
| `sso.google_directory_sync_enabled` | HIGH | 7 yr | `tenant_id`, `actor_id`, `admin_email_hash` (SHA-256 of impersonation email) |
| `sso.google_directory_sync_disabled` | HIGH | 7 yr | `tenant_id`, `actor_id`, `reason` (free text, max 200 chars) |
| `sso.google_directory_sync_validated` | STANDARD | 7 yr | `tenant_id`, `actor_id`, `test_user_role` |

**HMAC chain requirement:** `sso.google_directory_credential_rotated` must follow `sso.google_directory_credential_uploaded` for the new key in a traceable sequence — the `actor_id` on both events must match (or be a `form_system` service account for programmatic rotation). `sso.google_directory_sync_disabled` by a `form_system` actor without a preceding `sso.google_directory_credential_rotated` (compromise path) must trigger AL-SSO-GDIR-04 (unauthorized actor alert).

---

### Session revocation events (DEC-030 HMAC-chained · SSO §22)

> Defined in `docs/SSO_SCIM_IMPLEMENTATION.md §22`. Five events covering KV-backed bulk session revocation and tenant nuke operations. Privacy floor: `authorised_by` contains two user UUIDs only — no names, emails, or roles are stored in the audit event. CRITICAL events (`session.tenant_nuke_started`, `session.tenant_nuke_complete`) require two distinct user IDs in `authorised_by`; the API layer enforces this before the event can be emitted — a nuke attempted with a single authoriser is rejected 403 and logged as `support.unauthorized_nuke_attempt` (HIGH, 7yr). Cross-ref: SOC 2 CC6.3/CC7.2/CC7.3; closes SSO_SCIM_IMPLEMENTATION.md §22 checklist item 6 (P0 M4).

| Event type | Severity | Retention | Payload fields |
|---|---|---|---|
| `session.bulk_revocation_started` | HIGH | 7 yr | `tenant_id`, `user_count`, `scim_request_id`; **emitted before first KV write** (DEC-030 ordering: chain entry precedes state mutation) |
| `session.bulk_revocation_complete` | HIGH | 7 yr | `tenant_id`, `user_count`, `sessions_revoked`, `duration_ms`, `scim_request_id` |
| `session.tenant_nuke_started` | CRITICAL | 7 yr | `tenant_id`, `reason`, `authorised_by` (array of exactly two user UUIDs — API layer rejects nuke if array length ≠ 2 or both IDs are identical); **emitted before KV write** |
| `session.tenant_nuke_complete` | CRITICAL | 7 yr | `tenant_id`, `reason`, `authorised_by`, `kv_ttl_seconds` |
| `session.revocation_kv_sync_error` | HIGH | 7 yr | `tenant_id`, `user_id`, `session_id`, `error_code`, `fallback_supabase` (bool) — triggers PagerDuty P1 (AL-REVOKE-01); fallback Supabase path engaged when `fallback_supabase: true` |

**HMAC chain requirement:** `session.bulk_revocation_started` must be committed to the chain before `handleBulkScimRevocation()` writes the first KV key. `session.tenant_nuke_started` must be committed before `nukeTenantSessions()` writes the `revoke:tenant:{tenant_id}:all` KV key. These ordering constraints are mandatory for DEC-030 tamper evidence — a chain entry inserted retroactively for an action that already occurred is a chain violation.

**`support.unauthorized_nuke_attempt` (HIGH, 7yr):** Emitted when a tenant nuke API call is received with a single authoriser or with two identical user IDs. Payload: `{tenant_id, attempted_by_user_id, reason_provided, rejection_code: 'single_authoriser' | 'duplicate_authoriser'}`. This event is in the `support.*` namespace because it records a violation of an internal safety control, not a customer-visible action.

---

### Security & tenant isolation events (DEC-030 HMAC-chained · SOC2 §64.7)

> Defined in `docs/SOC2_READINESS.md §64.7` and `docs/DECISION_LOG.md DEC-042`. Five events covering RLS tenant isolation monitoring, CI test evidence, PAM audit record access transparency, and quota soft-warn telemetry. Privacy floor: `request_ip_hash` in `security.rls_bypass_attempt` is SHA-256 of the client IP with a daily rotating salt (per DPIA §4 constraint) — never a raw IP. `security.definer_function_cross_tenant` must never be aggregated or batched — emit synchronously per invocation (HIGH severity treated equivalent to a confirmed RLS bypass; triggers immediate P0 incident via R-09). `security.break_glass_audit_record_accessed` implements the alert-on-access governance for break-glass audit records per DEC-042 — no query-time approval gate; every read of break-glass rows is itself chained in DEC-030. Cross-ref: SOC 2 CC6.1/CC6.3/CC6.6/PI1.5-C3; closes SOC2_READINESS.md §64.9 checklist items 2 (P0) and 3 (P1).

| Event type | Severity | Retention | Emitter | Payload fields |
|---|---|---|---|---|
| `security.rls_bypass_attempt` | MEDIUM | 7 yr | Cloudflare Worker `rls-guard` middleware | `tenant_id_in_jwt` (UUID), `tenant_id_in_filter` (UUID), `table_name`, `query_path` (e.g. `/rest/v1/workout_sessions`), `request_ip_hash` (SHA-256 of IP + daily salt — **not raw IP**) |
| `security.definer_function_cross_tenant` | HIGH | 7 yr | Cloudflare Worker or Supabase Edge Function `definer-audit` wrapper | `function_name`, `caller_tenant_id` (UUID), `returned_tenant_ids` (UUID array — distinct tenant_ids that do not match caller), `row_count` (total cross-tenant rows) — triggers PagerDuty P0 immediately |
| `system.rls_test_suite_run` | STANDARD | 3 yr | CI workflow step in `rls-isolation-tests.yml` | `run_id` (GitHub Actions run_id), `ci_build_id` (run_id + run_attempt), `git_sha` (40-char), `tc_rls_001_result` / `tc_rls_002_result` / `tc_rls_003_result` / `tc_rls_004_result` / `tc_rls_005_result` (each: `PASS` / `FAIL` / `SKIP`), `overall_verdict` (`PASS` / `FAIL`), `evidence_path` (R2 object key for PRE-36-E-007 artefact) |
| `security.break_glass_audit_record_accessed` | HIGH | 7 yr | `pam-db-proxy` Cloudflare Worker — emits on every `SELECT` query that filters `admin_jit_escalations WHERE is_break_glass = true` | `queried_by_user_id` (UUID — admin_user_id of the accessor, resolved from `form_admin` JWT), `queried_by_role` (`security_reviewer` / `form_compliance`), `query_timestamp` (ISO 8601), `row_count_returned` (integer — number of break-glass rows in result set), `pam_session_ids_accessed` (UUID array — `pam_session_id` values in the result, for cross-reference with `pam.break_glass_review_completed` chain) |
| `security.quota_95pct_warning` | STANDARD | 3 yr | `quota-check` Cloudflare Worker — fires when a tenant's request count in the current billing period crosses `QUOTA_SECONDARY_WARN_PCT = 0.95` of their monthly quota; deduplicated via `api_quota_usage.secondary_warn_sent_at` column (per DATA_MODEL §30.3); **at most one emission per tenant per billing period** | `tenant_id` (UUID), `period_month` (YYYY-MM), `endpoint_category` (`coaching` \| `cv` \| `analytics` \| `all`), `current_requests` (integer), `quota_limit` (integer — tier monthly quota from DATA_MODEL §28), `pct_used` (float — must be ≥ 0.95 at emission), `warn_threshold: 0.95` — **no `user_id`** (quota is a tenant-level construct; individual user request counts are never exposed in this event) |

**Privacy constraint for `security.break_glass_audit_record_accessed`:** `pam_session_ids_accessed` array must contain only UUIDs — no `business_justification` text, no `target_tenant_id` (to avoid revealing which tenants were subject to break-glass access to the accessor's entire audit history). `row_count_returned = 0` is still emitted (confirms the query fired, even if no break-glass rows existed), which is the behaviour needed for SOC 2 CC6.6 evidence (access attempt, not just access success).

**HMAC chain requirement:** `security.rls_bypass_attempt` is expected to be low-volume under normal conditions (rare client bugs or adversarial probes) — no Queues batching needed. A spike > 10 events / 10 min for a single `tenant_id_in_filter` value should be escalated to R-09 Security Incident. `system.rls_test_suite_run` fires on every push to `main` — this is a high-frequency routine event; STANDARD severity is intentional. `security.break_glass_audit_record_accessed` is expected at very low frequency (< 1/month outside of SOC 2 fieldwork); any unexpected spike (> 3 within 24h) should trigger a PagerDuty P1 to `security-engineer`.

---

### Integration
- `integration.webhook_created` / `integration.webhook_fired`
- `integration.connector_added` (Slack, MS Teams, etc.)
- `integration.api_call` (sampled, not every call)

### System
- `system.backup_completed` / `system.backup_restored`
- `system.maintenance_started` / `system.maintenance_completed`
- `system.config_changed` (feature flag, environment var)
- `system.deployment_completed` (release SHA, environment)
- `system.access_review_completed` — quarterly access review completed (SOC 2 CC6.2/CC6.3/CC6.5/CC4.2); payload: `{reviewer_id, quarter, artifact_sha256, systems_reviewed_count, accounts_reviewed_count, findings_count, review_latency_days}`; STANDARD severity, 7yr retention; HMAC-chained; cross-ref: SOC2_READINESS §23, §65
- `system.credential_rotated` — FORM-internal credential rotated (scheduled or triggered); payload: `{credential_name, rotation_trigger: 'scheduled'|'compromise'|'quarterly_review'|'key_ceremony', days_since_last_rotation, rotated_by}`; STANDARD severity, 7yr retention; HMAC-chained; cross-ref: SOC2_READINESS §56/§57, §65.9
- `system.cron_job_stale` — `pg-cron-health-monitor` detected that a monitored pg_cron job exceeded its freshness window; payload: `{job_name, schedule, last_successful_run, hours_since_last_run, freshness_window_hours}`; HIGH severity, 7yr retention; HMAC-chained; emits PagerDuty P1 for most jobs (P0 for `audit-chain-daily-check`, `row-count-monitor`, `audit-event-flush` — see OBSERVABILITY §12.6 job registry); cross-ref: SOC2_READINESS §71.2.2, OBSERVABILITY §12.6, DEC-030
- `system.cron_health_check_passed` — `pg-cron-health-monitor` completed its hourly check with all 9 monitored jobs within their freshness windows; payload: `{jobs_checked, all_healthy: true}`; LOW severity, 3yr retention; HMAC-chained; no alert (steady-state confirmation event); cross-ref: SOC2_READINESS §71.2.2, DEC-030
- `system.cron_health_check_failed` — `pg-cron-health-monitor` Edge Function itself errored (DB unreachable, `cron.job_run_details` query failure, or unhandled exception); payload: `{error_message, jobs_checked}`; HIGH severity, 7yr retention; HMAC-chained; PagerDuty P0 → devops-lead + platform-engineer; cross-ref: SOC2_READINESS §71.2.2, DEC-030

### Admin (key management)

> HIGH severity · 7yr retention · HMAC-chained. Triggers PagerDuty P2 alert on unexpected rotation (outside scheduled window). Emitted synchronously inside KMS transaction — failure aborts rotation.

- `admin.encryption_key_rotated` — master encryption key rotated in KMS; payload: `{key_id, key_version_old, key_version_new, rotation_trigger: 'scheduled'|'incident'|'compromise', rotated_by, kms_provider}`; HIGH severity, 7yr retention; cross-ref: SOC2_READINESS §56, OBSERVABILITY §30.10 item 10, DEC-030
- `admin.signing_key_rotated` — HMAC chain signing key rotated; payload: `{key_id, rotation_trigger, rotated_by}`; HIGH severity, 7yr retention; DEC-030 chain continuity preserved via `key_transition` metadata

### Asset & Device Management

> Defined in `compliance/c1/device-disposal-policy.md §8` (POL-013) and `docs/SOC2_READINESS.md §66.10`. All five events are HMAC-chained with 7-year retention. Privacy invariant: no serial number in plain text — use `device_asset_id` (FORM-DEV-XXX). The 30-day `disposal_initiated` → `disposal_completed` gap triggers alert MDD-AL-01. Closes MDD-P0-03; cross-ref: SOC 2 C1.2 / CC6.5 / CC6.7.

| Event type | Severity | Retention | Trigger | Key payload fields |
|---|---|---|---|---|
| `asset.disposal_initiated` | STANDARD | 7 yr | Before any wipe or handoff step begins | `device_asset_id`, `disposal_method`, `authorized_by_user_id`, `authorized_by_self` (bool) |
| `asset.disposal_completed` | STANDARD | 7 yr | After wipe verification or vendor certificate received | `device_asset_id`, `disposal_method`, `wipe_verification_screenshot_sha256`, `vendor_cert_id` (if third-party) |
| `asset.device_sanitized` | HIGH | 7 yr | After EACS or MDM wipe confirmation | `device_asset_id`, `sanitization_standard: "NIST-SP-800-88-Purge"`, `sanitized_by`, `mdm_wipe_event_id` (if MDM) |
| `asset.chain_of_custody_transferred` | HIGH | 7 yr | When device leaves FORM possession for third-party destruction | `device_asset_id`, `recipient_vendor`, `chain_of_custody_form_sha256`, `transport_method` |
| `asset.disposal_log_filed` | STANDARD | 7 yr | When `device-register.csv` is updated with disposal record | `device_asset_id`, `register_commit_sha`, `mdd_evidence_id` |

**HMAC chain requirement:** `asset.disposal_initiated` must precede `asset.disposal_completed` within 30 days; gap > 30 days triggers MDD-AL-01 (PagerDuty P2). `asset.device_sanitized` and `asset.chain_of_custody_transferred` are HIGH severity and must be emitted synchronously — failure aborts the disposal workflow.

---

### Vulnerability Management events (DEC-030 HMAC-chained · CC7.1 / CC7.2)

> Defined in `compliance/cc7/vuln-management-policy.md §10` (POL-010). Five events covering the full CVE lifecycle: discovery, patch deployment, SLA exception, Won't Fix closure, and enterprise tenant notification. Privacy floor: no `vuln.*` payload may contain `user_id`, `session_id`, health data, or coaching content — `cve_id`, `component`, and `affected_version` are infrastructure identifiers with no user data linkage. A chain break on any `vuln.*` event is a P0 incident per `docs/INCIDENT_RESPONSE.md §R-05`. Cross-ref: SOC 2 CC7.1 (threat identification) / CC7.2 (vulnerability monitoring and remediation tracking) / CC5.3 (standalone Vulnerability Management Policy — POL-010). Closes `compliance/cc7/vuln-management-policy.md` checklist item 6 (P1 M3).

| Event type | Severity | Retention | Trigger | Key payload fields |
|---|---|---|---|---|
| `vuln.cve_discovered` | MEDIUM | 7 yr | Linear `[SEC]` ticket created for CVSS ≥ 4.0 finding | `cve_id`, `cvss_base`, `final_severity`, `uplift_applied` (bool), `uplift_rules[]` (`U-01`\|`U-02`\|`U-03`), `component`, `affected_version`, `sla_deadline` (ISO 8601) |
| `vuln.patch_deployed` | MEDIUM | 7 yr | Production deploy of the patch PR confirmed (CI post-deploy hook) | `cve_id`, `patch_pr_url`, `merged_at` (ISO 8601), `deployed_at` (ISO 8601), `sla_met` (bool), `elapsed_hours` |
| `vuln.sla_exception_granted` | HIGH | 7 yr | Risk acceptance approved per vuln-management-policy §7.2 | `cve_id`, `original_severity`, `approved_by[]` (UUID array — joint approval per §7.2 matrix), `new_deadline` (ISO 8601), `compensating_control` (free text, max 500 chars), `rationale_ref` (Linear ticket URL) |
| `vuln.wont_fix_closed` | MEDIUM | 7 yr | Linear ticket closed as Won't Fix per vuln-management-policy §7.3 | `cve_id`, `cvss_base`, `rationale` (free text, max 500 chars), `approved_by` (UUID — security-engineer) |
| `vuln.enterprise_notified` | MEDIUM | 7 yr | Template E-VULN-01 or E-VULN-02 sent to enterprise tenant | `cve_id`, `tenant_id`, `template_used` (`E-VULN-01`\|`E-VULN-02`), `sent_by` (UUID), `notified_at` (ISO 8601) |

**HMAC chain requirement:** `vuln.patch_deployed` must reference the same `cve_id` as a preceding `vuln.cve_discovered` in the chain. `vuln.sla_exception_granted` must reference a `cve_id` with an open `vuln.cve_discovered` and no `vuln.patch_deployed`. `vuln.wont_fix_closed` and `vuln.patch_deployed` are mutually exclusive for the same `cve_id` — both present in the chain for the same `cve_id` is a chain anomaly detected by the weekly audit batch. **Uplift rule U-02 (RLS bypass potential) makes a finding permanently ineligible for Won't Fix** — emitting `vuln.wont_fix_closed` with `uplift_rules: ['U-02']` is a policy violation; the weekly chain audit flags this and triggers PagerDuty P1 (no compliant path exists for an RLS-bypass Won't Fix).

**Emitter assignment:** `vuln.cve_discovered` — `security-engineer` (manual via admin API); `vuln.patch_deployed` — `devops-lead` (automated via CI post-deploy hook, implementation: vuln-management-policy.md checklist item #7 P1 M4); `vuln.sla_exception_granted` — `compliance-officer` (manual); `vuln.wont_fix_closed` — `security-engineer` (manual); `vuln.enterprise_notified` — `customer-success` (manual when Template E-VULN-01/E-VULN-02 is sent).

---

### Compliance operating evidence events (DEC-030 HMAC-chained · C1.1/C1.2/P5.2 · SOC2_READINESS.md §73)

> Defined in `docs/SOC2_READINESS.md §73`. Four events providing the C1 Confidentiality TSC operating evidence cadence — the operational counterparts to the A1 (`system.cron_*`) and PI1 (`billing.*` / `coaching.*`) evidence streams. Privacy invariant: no `user_id`, `dsar_request_id`, tenant email, health values, or disposal serial numbers in any payload — operational counts, compliance metadata, and `device_asset_id` references only. The `erasure_sla_alert_fired` payload contains only `breach_count` (integer) and alert metadata; a chain audit that exposes individual DSAR identifiers is a privacy floor violation. Cross-ref: SOC 2 C1.1/C1.2/P5.2; `docs/OBSERVABILITY.md §6.2 c1_erasure_sla` subsection.

| Event type | Severity | Retention | Trigger | Key payload fields |
|---|---|---|---|---|
| `compliance.data_asset_inventory_reviewed` | STANDARD | 3 yr | Quarterly data asset inventory review completed per §73.2.1 cadence | `review_date`, `reviewer_user_id`, `asset_register_commit_sha`, `items_reviewed_count`, `changes_detected` (bool) |
| `compliance.encryption_verified` | STANDARD | 3 yr | Annual DDL encryption-at-rest verification completed per §73.2.2 | `verification_date`, `verifier_user_id`, `tables_verified_count`, `all_encrypted` (bool), `evidence_path` (`compliance/evidence/c1/encryption-verification-YYYY.md`) |
| `compliance.erasure_sla_alert_fired` | HIGH | 7 yr | pg_cron `c1-erasure-sla-monitor` (daily 08:00 UTC) detects ≥ 1 open Art. 17 erasure request with `submitted_at < now() - INTERVAL '33 days'`; implements AL-C1-01 in §73.3.1 | `breach_count` (integer — count only, no IDs), `alert_id: "AL-C1-01"`, `threshold_days: 33` |
| `compliance.disposal_audit_completed` | STANDARD | 3 yr | Quarterly/annual device disposal audit completed per §73.3.2 cadence | `audit_date`, `auditor_user_id`, `audit_scope` (`quarterly`\|`annual`), `devices_reviewed_count`, `disposals_completed_count`, `evidence_path` |

**HMAC chain requirement:** `compliance.erasure_sla_alert_fired` is HIGH severity and must be emitted synchronously within the pg_cron job — if the `net.http_post` call to `emit-audit-event` fails, the pg_cron job failure surfaces as `system.cron_job_stale` (AL-CRON-01 via §12.6 health monitor). C1-CHAIN-01/02/03 monitors (defined in SOC2_READINESS.md §73.2–§73.3) enforce that each cadenced event type recurs within 100 days; absence triggers PagerDuty P1 on the `form-compliance` service.

**Emitter assignment:** `compliance.data_asset_inventory_reviewed` and `compliance.encryption_verified` — `compliance-officer` (manual via admin API at cadence); `compliance.erasure_sla_alert_fired` — `form_system` (automated, pg_cron `c1-erasure-sla-monitor`); `compliance.disposal_audit_completed` — `compliance-officer` (manual at audit completion).

---

### Enterprise pricing governance events (DEC-030 HMAC-chained · COST_MODEL §31.8)

> Defined in `docs/COST_MODEL.md §31.8`. Four events covering the full pricing-decision audit trail — non-standard discounts, consumer price changes, enterprise list-price changes, and price-floor breach attempts. Required for COST_MODEL §31.9 item 1 (P0, M4 — before first enterprise quote is sent). Privacy invariant: no prospect company names, customer PII, health values, or employee names in any payload. `deal_id` is an internal CRM reference only. `rationale_text` and `rationale_ref` must not contain PII. **Ordering rule:** `enterprise.pricing_exception_approved` **must be emitted before the quote is sent** — a chain entry inserted after the quote was sent is a compliance violation detectable by the weekly chain audit. `enterprise.price_floor_override_requested` must emit a record regardless of `decision` value (`approved`/`denied`/`restructured`); the absence of a denied attempt is not evidence that none occurred. Cross-ref: SOC 2 CC1.4 (commitment to values of integrity), CC5.2 (management establishes pricing controls), A1.1 (capacity management); `docs/ENTERPRISE.md` §Pricing; `docs/MSA_TEMPLATE.md`; DECISION_LOG.md (pricing change decisions).

| Event type | Severity | Retention | Trigger | Key payload fields |
|---|---|---|---|---|
| `enterprise.pricing_exception_approved` | HIGH | 7 yr | Non-standard discount (beyond pre-approved schedule in §31.6) approved by founder or investor; **must be emitted before the quote is sent to the prospect** | `deal_id` (CRM ref), `tier` (`starter`\|`growth`\|`enterprise`), `list_price_usd`, `approved_price_usd`, `effective_discount_pct`, `seat_count`, `contract_years`, `approver_user_id`, `approval_level` (`founder`\|`founder_plus_investor`), `rationale_text` (free text, max 200 chars — **no PII or company name**), `quote_ref` |
| `enterprise.consumer_price_updated` | HIGH | 7 yr | Consumer Pro list price changed; emitted by founder via admin console | `previous_price_usd`, `new_price_usd`, `effective_date` (ISO 8601), `grandfathering_policy` (free text — describes treatment of existing subscribers), `updated_by` (founder `user_id`), `rationale_ref` (DECISION_LOG.md entry slug) |
| `enterprise.list_price_updated` | HIGH | 7 yr | Enterprise tier list prices updated; emitted by founder | `tier` (`starter`\|`growth`\|`enterprise`), `previous_price_usd`, `new_price_usd`, `effective_date` (ISO 8601), `updated_by` (founder `user_id`), `active_contracts_affected` (integer — count of contracts at prior list rate that will face change at renewal), `rationale_ref` |
| `enterprise.price_floor_override_requested` | CRITICAL | 7 yr | Quote below contractual floor (`$6.00`/`$4.50`/`$4.00` per §31.5) requested — **record emitted even when `decision = 'denied'`** | `deal_id`, `tier`, `requested_price_usd`, `floor_price_usd`, `requester_user_id`, `decision` (`approved`\|`denied`\|`restructured`), `approver_chain` (array of user UUIDs — empty array if denied at requester stage) |

**HMAC chain requirement:** All four events must include `prev_hash`. `enterprise.pricing_exception_approved` links forward to `enterprise.contract_signed` (§23.7) via `pricing_exception_event_id` when the deal closes — auditors can traverse the chain from exception approval to signed contract. `enterprise.price_floor_override_requested` is CRITICAL severity: emitting it fires PagerDuty P2 and a compliance-officer email regardless of `decision`. A denied floor-breach attempt is not a no-op — the record exists so auditors can verify that no unrecorded floor breaches occurred during the observation window. Under no circumstance may `enterprise.price_floor_override_requested` be suppressed because the outcome was `denied`.

**Emitter assignment:** `enterprise.pricing_exception_approved` — founder (manual via admin console, required before quote is sent); `enterprise.consumer_price_updated` — founder (manual at price change); `enterprise.list_price_updated` — founder (manual at list price change); `enterprise.price_floor_override_requested` — pricing calculator (client-side prevention emits no audit record; server-side approval request emits this event automatically when a below-floor request reaches the API).

---

### Enterprise AI token budget governance events (DEC-030 HMAC-chained · COST_MODEL §33.9)

> Defined in `docs/COST_MODEL.md §33.9`. Two events covering AI token overage monitoring and cross-tenant statistical outlier detection. Required for COST_MODEL §33.10 checklist item 2 (P0, M5 — before enterprise GA). **Privacy invariant (both events):** payload contains `tenant_id` (UUID) only. No `user_id`, no per-user token breakdown, no coaching content, no prompt or response text. Enforced by schema constraint — `user_id` column is NULL for both event types. `tenant_id` is the granularity floor; k-anonymity applies to outlier detection (tier-mean computation requires ≥ 5 tenants in cohort). Cross-ref: SOC 2 CC5.2 (business risk monitoring), A1.1 (capacity management); `docs/COST_MODEL.md §33.3` (token budget tiers); `docs/COST_MODEL.md §33.7` (overage bands); `docs/DATA_MODEL.md §21.8` (`tenant_monthly_coaching_cost` view — source of `actual_tokens`). Closes COST_MODEL.md §33.10 checklist item 2 (P0, M5).

| Event type | Severity | Retention | Trigger | Key payload fields |
|---|---|---|---|---|
| `enterprise.ai_cost_monitor_alert` | STANDARD | 3 yr | Nightly pg_cron job: tenant `actual_tokens` crosses Yellow (100–119%), Orange (120–149%), or Red (≥ 150%) threshold vs. tier budget from §33.3 | `budget_period` (YYYY-MM), `budget_tokens` (integer — tier token allowance from §33.3), `actual_tokens` (integer — from `tenant_monthly_coaching_cost` view), `overage_pct` (float), `overage_band` (`yellow`\|`orange`\|`red`), `alert_sent_to` (string array) |
| `enterprise.ai_cost_outlier_flagged` | STANDARD | 3 yr | Monthly pg_cron job: tenant per-seat spend ≥ 2σ above tier mean across all tenants in the same tier (minimum 5-tenant cohort for statistical validity) | `budget_period` (YYYY-MM), `tier` (`starter`\|`growth`\|`enterprise`), `tenant_spend_usd` (integer), `tier_mean_spend_usd` (integer), `tier_stddev_usd` (integer), `sigma_above_mean` (float — must be ≥ 2.0 at emission) |

**HMAC chain requirement:** Both events must include `prev_hash`. Neither event has a required predecessor or successor — they are standalone monitoring records. Both events are FORM-internal operational records; they are not surfaced to the enterprise tenant admin or customer. `enterprise.ai_cost_monitor_alert` may fire multiple times in a month (each crossing of Yellow, Orange, or Red threshold is a separate chain entry). The `budget_period` field allows auditors to group events by month.

**Emitter assignment:** Both events — `form_system` (automated nightly/monthly pg_cron jobs as specified in COST_MODEL §33.10 items 3 and 4). The `emit-audit-event` Cloudflare Worker validates the Zod schema before chain-appending; HTTP 422 on violation. Neither event may be emitted manually via the admin console.

---

### Enterprise renewal & churn lifecycle events (DEC-030 HMAC-chained · COST_MODEL §34.8)

> Defined in `docs/COST_MODEL.md §34.8`. Three events creating an immutable record of renewal risk classification, confirmed churn, and retention discounts granted. Required for COST_MODEL §34.9 checklist item 1 (P0, M10 — before first enterprise renewal date). **Privacy invariant (all three events):** tenant-aggregate data only — no `user_id`, no individual engagement data, no health metrics, no employee names. CHS (Customer Health Score) is a tenant-level composite; k-anonymity enforced in `tenant_engagement_snapshots` per OBSERVABILITY.md §33. Verbatim exit interview notes must remain in CRM only and must never appear in the DEC-030 chain. Cross-ref: SOC 2 CC5.2 (business risk assessment), CC7.2 (monitoring of controls), CC1.4 (pricing integrity — retention discount); `docs/OBSERVABILITY.md §33` (CHS model, AL-ENGAGE-02/05/06 alert rules); `docs/COST_MODEL.md §31.5` (price floors — `floor_respected` invariant in `enterprise.retention_discount_granted`); `docs/COST_MODEL.md §32` (pricing exception approval chain — `pricing_exception_event_id` linkage for discounts > 15%). Closes COST_MODEL.md §34.9 checklist item 1 (P0, M10).

| Event type | Severity | Retention | Trigger | Key payload fields |
|---|---|---|---|---|
| `enterprise.renewal_risk_flagged` | STANDARD | 3 yr | OBSERVABILITY §33 alert routing: AL-ENGAGE-02, AL-ENGAGE-05, or AL-ENGAGE-06 fires; or `renewal_proximity` trigger (renewal < 90 days + Amber CHS band) | `risk_band` (`amber`\|`red`), `chs_score` (integer 0–100 — tenant-aggregate composite), `prior_risk_band` (`green`\|`amber`), `trigger_rule` (alert rule slug or `renewal_proximity`), `days_to_renewal` (integer or null if > 180 days / no active contract), `alerted_to` (string array — e.g. `['csm-assigned', 'cs-lead']`) |
| `enterprise.churn_confirmed` | HIGH | 7 yr | Tenant formally notifies non-renewal, or contract is terminated early; CSM must classify churn reason before emitting | `churn_type` (`non_renewal`\|`mid_contract_termination`), `churn_effective_date` (ISO 8601), `contract_id` (FK to `tenant_contracts.id`), `acv_usd` (integer — no cent precision), `final_risk_band` (`green`\|`amber`\|`red`), `intervention_count` (integer — CSM intervention units logged), `churn_reason_category` (`low_utilization`\|`budget_cut`\|`champion_left`\|`product_gap`\|`competitor`\|`unknown`) |
| `enterprise.retention_discount_granted` | HIGH | 7 yr | Retention discount approved and included in renewal quote | `prior_acv_usd` (integer), `new_acv_usd` (integer), `discount_pct` (float — e.g. 15.0), `effective_rate_per_seat` (float — must be ≥ price floor for tier per §31.5), `floor_respected` (boolean — **always `true`**; floor enforced upstream; event never emitted on floor violation), `risk_band_at_grant` (`amber`\|`red`), `approver_role` (`cs_lead`\|`founder`\|`founder_investor`), `pricing_exception_event_id` (UUID or null — **required** if `discount_pct > 15`; FK to `enterprise.pricing_exception_approved` from §31.8) |

**HMAC chain requirement:** All three events must include `prev_hash`. No strict ordering is enforced across the three event types (they may or may not appear together for a given account). However: `enterprise.retention_discount_granted` with `discount_pct > 15` **must** be preceded by `enterprise.pricing_exception_approved` (from §31.8) for the same account, linked via `pricing_exception_event_id` — the `emit-audit-event` Worker validates that this predecessor event exists in the chain before appending; HTTP 422 if absent. `floor_respected` is always `true` in `enterprise.retention_discount_granted` — a retention discount that would breach the price floor must not proceed; the attempted breach is recorded as `enterprise.price_floor_override_requested` (CRITICAL, §31.8), which fires even for denied attempts.

**Emitter assignment:** `enterprise.renewal_risk_flagged` — `form_system` (automated: fired by OBSERVABILITY §33 alert routing on AL-ENGAGE-02/05/06 or `renewal_proximity` trigger); `enterprise.churn_confirmed` — `customer-success` (manual via admin console — CSM must classify `churn_reason_category` in CRM first; this event is the compliance record of that classification); `enterprise.retention_discount_granted` — `customer-success` (manual via admin console at renewal quote generation — triggers pricing exception flow in §32 if `discount_pct > 15`; `emit-audit-event` Worker validates `pricing_exception_event_id` presence before chain-appending).

---

### Victor AI safety events (DEC-030 HMAC-chained · OBSERVABILITY §32 · INCIDENT_RESPONSE R-23)

> Defined in `docs/INCIDENT_RESPONSE.md §R-23` and `docs/OBSERVABILITY.md §32`. Five events covering the full Victor AI clinical-safety incident lifecycle — from P0 trigger through containment, global disable, staged re-enable, and resolution. **Privacy floor:** no per-user coaching content, no user health values, no `user_id` in any payload — `incident_id` (UUID), `session_id` (UUID), and aggregate counts only. `clinical_safety_rules_violated` is a boolean; the specific rule text is in the incident record, not the audit chain. VSAFETY-CHAIN-01/02/03 DEC-030 chain monitors (defined in OBSERVABILITY.md §32.7) enforce ordering: `ai.safety_incident_opened` must precede `ai.victor_disabled`; `ai.victor_reenabled` requires a preceding `ai.safety_incident_resolved` for the same `incident_id` — chain guard in `emit-audit-event` Worker returns HTTP 422 on violation. **CRITICAL events (`ai.safety_incident_opened` for VT-03/04/05/06) trigger PagerDuty `FORM Clinical Safety` P0 and clinical-safety VETO unconditionally.** Cross-ref: SOC 2 CC7.2/CC7.4; GDPR Art. 9 (health data integrity record); `docs/OBSERVABILITY.md §32.5` (FORM-VICTOR-001 through FORM-VICTOR-004 alert rules); `docs/OBSERVABILITY.md §2.1` (VICTOR-SLO-01 through VICTOR-SLO-04); closes OBSERVABILITY.md §32.10 checklist item 7 (P0 M4) and INCIDENT_RESPONSE.md R-23 checklist item 2 (P0 M4).

| Event type | Severity | Retention | Trigger | Key payload fields |
|---|---|---|---|---|
| `ai.safety_incident_opened` | **CRITICAL** (trigger_category ∈ {VT-03, VT-04, VT-05, VT-06}); HIGH (all other categories) | 7 yr | First `VICTOR_SAFETY_TELEMETRY` P0 row detected; fires before containment (DEC-030 ordering: chain entry precedes state mutation) | `incident_id` (UUID), `severity_class` (`P0`\|`P1`\|`P2`), `trigger_category` (`VT-01`…`VT-10`), `affected_session_count` (integer — aggregate only), `model_version`, `prompt_version`, `clinical_safety_rules_violated` (bool) |
| `ai.safety_incident_contained` | HIGH | 7 yr | Containment step applied (Victor feature flag suppressed or rate-limit engaged); emitted by `incident-response` Worker synchronously inside containment transaction | `incident_id` (UUID), `containment_method` (`feature_flag_disable`\|`rate_limit`\|`prompt_version_rollback`), `contained_by_user_id` (UUID — `form_system` for automated path) |
| `ai.victor_disabled` | HIGH | 7 yr | `victor_coach_enabled` feature flag set to `false` globally; triggered by P0 clinical-safety VETO path in INCIDENT_RESPONSE.md R-23 §R-23.3 T+3min | `incident_id` (UUID), `disabled_by` (UUID — clinical-safety owner or `form_system` on auto-disable), `feature_flag_key`: `"victor_coach_enabled"` |
| `ai.victor_reenabled` | HIGH | 7 yr | Staged re-enable ramp step applied; requires `ai.safety_incident_resolved` for same `incident_id` to already exist in chain — VSAFETY-CHAIN-03 write-guard blocks this event (HTTP 422) if resolved event is absent | `incident_id` (UUID), `ramp_percentage` (integer 0–100), `approved_by_user_id` (UUID), `clinical_safety_approved` (bool — must be `true`; chain guard rejects if `false`) |
| `ai.safety_incident_resolved` | STANDARD | 3 yr | Incident closed by clinical-safety + security-engineer sign-off; root cause documented; post-incident controls agreed | `incident_id` (UUID), `resolution_method` (`prompt_patch`\|`model_rollback`\|`guardrail_injection`\|`feature_disabled_permanently`), `root_cause_hypothesis` (free text, max 500 chars — **no user PII**), `post_incident_controls` (string array) |

**HMAC chain requirement:** `ai.safety_incident_opened` must precede all other `ai.*` events for the same `incident_id` (VSAFETY-CHAIN-01 enforced). `ai.victor_disabled` must follow `ai.safety_incident_opened` for the same `incident_id` (VSAFETY-CHAIN-02 enforced — an `ai.victor_disabled` without a preceding `ai.safety_incident_opened` is a chain violation, P0 via PagerDuty). `ai.victor_reenabled` requires a preceding `ai.safety_incident_resolved` for the same `incident_id` in the chain — VSAFETY-CHAIN-03 write-guard in `emit-audit-event` Worker returns HTTP 422 and fires PagerDuty P0 if this constraint is violated. All five events must carry `prev_hash`. High-frequency non-P0 scenarios (VT-01/02 P1 flag spikes producing multiple `ai.safety_incident_opened` HIGH events during model evaluation) are deduplicated by `incident_id` in the PagerDuty integration layer but must each enter the HMAC chain individually.

**Emitter assignment:** `ai.safety_incident_opened` — `form_system` (automated, fired by `FORM-VICTOR-001` Cloudflare Workers pipeline on first `VICTOR_SAFETY_TELEMETRY` P0 row, or manually by clinical-safety owner via admin console); `ai.safety_incident_contained` — `form_system` (automated containment Worker) or `clinical-safety` (manual); `ai.victor_disabled` — `form_system` (automated P0 path, T+3min per R-23 §R-23.3) or `clinical-safety` (manual); `ai.victor_reenabled` — `clinical-safety` + `security-engineer` (manual two-party re-enable; `form_system` path not permitted); `ai.safety_incident_resolved` — `clinical-safety` + `security-engineer` (manual joint sign-off required).

---

### Vendor & Sub-Processor Management events (DEC-030 HMAC-chained · SOC2_READINESS §75.2.4)

> Defined in `docs/SOC2_READINESS.md §75.2.4`. Two events covering the sub-processor roster lifecycle — addition and removal. Required for P6.1 (sub-processor disclosure) and CC9.2 (vendor monitoring) SOC 2 controls. **Privacy invariant:** no user_id, session data, or health values in any payload — infrastructure vendor identifiers only (`SP-XX` codes, service metadata). `vendor.sub_processor_added` is emitted at the effective date (30 days after notice sent to enterprise tenants per Art. 28(2)), not at the notice dispatch; the notice dispatch is a separate operational record. Cross-ref: SOC2_READINESS §75.2.4 (procedure), `docs/SUBPROCESSORS.md §8.2` (notification procedure), PRE-75-E-002 (first-use evidence artifact), P6.1/CC9.2. Closes SOC2_READINESS.md §75.8 checklist item 5 (P1 Pre-GA).

| Event type | Severity | Retention | Trigger | Key payload fields |
|---|---|---|---|---|
| `vendor.sub_processor_added` | HIGH | 7 yr | At effective date of addition (30 days after notice sent); emitted by `compliance-officer` (manual via admin console) | `resource_type: "sub_processor"`, `resource_id` (SP-XX), `metadata.vendor_name`, `metadata.service_role`, `metadata.data_categories[]`, `metadata.processing_location`, `metadata.transfer_mechanism`, `metadata.effective_date` (ISO 8601), `metadata.notice_sent_date` (ISO 8601), `metadata.notice_period_days: 30`, `metadata.enterprise_tenants_notified` (integer) |
| `vendor.sub_processor_removed` | STANDARD | 3 yr | At effective date of removal; emitted by `compliance-officer` (manual via admin console) | `resource_type: "sub_processor"`, `resource_id` (SP-XX), `metadata.vendor_name`, `metadata.service_role`, `metadata.data_categories[]`, `metadata.processing_location`, `metadata.transfer_mechanism`, `metadata.effective_date` (ISO 8601), `metadata.notice_sent_date` (ISO 8601), `metadata.enterprise_tenants_notified` (integer), `metadata.deletion_certificate_ref` (string) |

**HMAC chain requirement:** Both events must include `prev_hash`. `vendor.sub_processor_added` records the addition becoming effective; a chain entry for the notice dispatch does not exist (the 30-day notice is tracked in the operational record at `compliance/dpa/sub-processor-notices/`). `vendor.sub_processor_removed` must be emitted synchronously at the point of removal so the audit chain accurately reflects the sub-processor roster at any point in the SOC 2 observation window.

---

### Government & Legal Disclosure events (DEC-030 HMAC-chained · SOC2_READINESS §75.5.3)

> Defined in `docs/SOC2_READINESS.md §75.5.3`. Ten events covering the complete government and law enforcement request lifecycle — from receipt through legal hold, legal assessment, and resolution (disclosure or decline). **CRITICAL privacy rule:** No user names, email addresses, or authority names in plain text in any payload. `users_named_count` and `users_affected_count` are aggregate integers only. `recipient_authority_hash` is SHA-256 of the authority name — never plain text. `request_id` is an internal UUID; it must not encode jurisdiction or authority identity. **CRITICAL events** (`legal.disclosure_executed`, `legal.disclosure_declined`) trigger PagerDuty P0 and compliance-officer notification immediately on emission. `legal.disclosure_declined` must be emitted even when the request is declined at the initial review stage — a declined attempt is not a no-op. **HMAC chain ordering (LEGAL-CHAIN-01):** `legal.request_received` must precede `legal.disclosure_executed` or `legal.disclosure_declined` for the same `request_id`; the chain guard in `emit-audit-event` Worker returns HTTP 422 on violation. Cross-ref: SOC 2 P6.5 (government disclosure) / CC1.4 (integrity) / A1.1 (legal hold affects data availability); SOC2_READINESS §75.5.3; `compliance/p1/gov-request-policy.md` (P1-CIP-002). Closes SOC2_READINESS.md §75.8 checklist item 8 (P1 Pre-GA).

| Event type | Severity | Retention | Key payload fields |
|---|---|---|---|
| `legal.request_received` | HIGH | 7 yr | `request_id` (UUID), `request_jurisdiction` (`"UA"` \| `"EU"` \| `"US"`), `legal_process_type` (`"court_order"` \| `"subpoena"` \| `"national_security_letter"` \| `"voluntary_request"` \| `"regulatory_inquiry"`), `request_received_at` (ISO 8601), `data_categories_requested` (string array), `users_named_count` (integer — **aggregate count only, no IDs**) |
| `legal.founder_notified` | HIGH | 7 yr | `request_id`, `request_jurisdiction`, `legal_process_type`, `notified_at` (ISO 8601), `channel` (`"signal"` \| `"encrypted_email"`) |
| `legal.counsel_retained` | HIGH | 7 yr | `request_id`, `request_jurisdiction`, `legal_process_type`, `retained_at` (ISO 8601), `counsel_ref` (string — internal code, **no personal name**) |
| `legal.legal_hold_activated` | HIGH | 7 yr | `request_id`, `request_jurisdiction`, `legal_process_type`, `hold_scope` (string, max 200 chars — data categories only, **no user IDs**), `hold_activated_at` (ISO 8601) |
| `legal.art48_assessed` | HIGH | 7 yr | `request_id`, `request_jurisdiction`, `legal_process_type`, `treaty_basis_found` (bool), `assessment_summary` (string, max 300 chars — **no PII**) |
| `legal.founder_signoff_obtained` | HIGH | 7 yr | `request_id`, `request_jurisdiction`, `legal_process_type`, `signoff_at` (ISO 8601) |
| `legal.disclosure_executed` | **CRITICAL** | 7 yr | `request_id`, `request_jurisdiction`, `legal_process_type`, `disclosed_at` (ISO 8601), `data_categories_disclosed` (string array), `users_affected_count` (integer — **aggregate only**), `recipient_authority_hash` (SHA-256 of authority name — **never plain text**) |
| `legal.subject_notified` | HIGH | 7 yr | `request_id`, `request_jurisdiction`, `legal_process_type`, `notified_count` (integer), `gag_order_exception` (bool), `notification_channel` (`"in_app"` \| `"email"` \| `"gag_order_blocked"`) |
| `legal.transparency_tally_updated` | STANDARD | 3 yr | `request_id`, `request_jurisdiction`, `legal_process_type`, `report_year` (integer), `category` (string), `range_low` (integer), `range_high` (integer) |
| `legal.disclosure_declined` | **CRITICAL** | 7 yr | `request_id`, `request_jurisdiction`, `legal_process_type`, `declined_at` (ISO 8601), `decline_reason` (`"no_go_category"` \| `"insufficient_legal_basis"` \| `"art48_no_treaty"` \| `"founder_veto"`) |

**HMAC chain requirement (LEGAL-CHAIN-01):** `legal.request_received` must be the first event emitted for any given `request_id`. Procedural events (`legal.founder_notified`, `legal.counsel_retained`, `legal.legal_hold_activated`, `legal.art48_assessed`, `legal.founder_signoff_obtained`) may be emitted in any order between `legal.request_received` and the terminal event. Terminal events `legal.disclosure_executed` and `legal.disclosure_declined` are mutually exclusive for the same `request_id` — both present is a chain anomaly detected by the weekly audit batch. Both CRITICAL terminal events trigger PagerDuty P0 + compliance-officer notification immediately. `legal.disclosure_declined` must be emitted even when declined at the initial review stage (immediately after `legal.request_received` with no intervening procedural events).

**Emitter assignment:** `compliance-officer` (manual via admin console) for all `legal.*` events.

---

### Support actions (highest privilege)
- `support.impersonation_started` — FORM employee acting as customer
- `support.impersonation_ended`
- `support.data_accessed` — internal viewing tenant data
- `support.config_override` — emergency change w/o customer ticket

**`support.*` events trigger email-notification до tenant admin within 24h.** Це non-negotiable trust requirement.

---

## Retention policy

| Class | Retention | Why |
|---|---|---|
| `auth.*` | 3 years | Common compliance ask |
| `tenant.*` administrative | 7 years | SOC 2 / financial audit trail |
| `data.export/deletion` | 7 years | GDPR proof of compliance |
| `privacy.consent_*` | 7 years | Regulatory disputes |
| `privacy.floor_breach_*` | 7 years | GDPR Art. 9 enterprise incident record + SOC 2 P6.1/P7.1/CC7.4 evidence |
| `system.deployment` | 5 years | Incident investigation |
| `system.access_review_completed` | 7 years | SOC 2 CC6 quarterly audit evidence |
| `system.credential_rotated` | 7 years | SOC 2 CC6 key management trail |
| `system.rls_test_suite_run` | 3 years | CI tenant isolation test run log |
| `system.cron_job_stale` | 7 years | SOC 2 A1.1 — pg_cron freshness breach; operational capacity monitoring evidence |
| `system.cron_health_check_passed` | 3 years | Routine pg_cron health confirmation; operational audit trail |
| `system.cron_health_check_failed` | 7 years | SOC 2 A1.1 — `pg-cron-health-monitor` Edge Function failure record |
| `admin.*` (key management) | 7 years | Encryption governance + incident investigation |
| `api_key.*` | 7 years | SOC 2 CC6.4 credential lifecycle evidence |
| `scim.ip_enforcement_*` / `scim.ip_blocked` | 7 years | SOC 2 CC6.1 network-layer access evidence |
| `sso.auth_policy_updated` / `sso.ip_allowlist_*` / `sso.ip_blocked` / `sso.mfa_enforcement_changed` / `sso.mfa_bypass_granted` / `sso.auth_policy_lockout_recovery` | 7 years | SOC 2 CC6.1/CC6.2/CC6.8 auth policy audit trail |
| `sso.mfa_enforcement_sweep_completed` | 3 years | Operational sweep completion; not required for SOC 2 observation evidence |
| `pam.*` | 7 years | SOC 2 CC6.1/CC6.2/CC6.3 JIT privilege access evidence; break-glass record |
| `sso.google_directory_*` | 7 years | SOC 2 CC6.3/CC9.2 Google Workspace Directory sync evidence |
| `session.bulk_revocation_*` / `session.tenant_nuke_*` / `session.revocation_kv_sync_error` | 7 years | SOC 2 CC6.3 timely logical access removal evidence |
| `security.rls_bypass_attempt` / `security.definer_function_cross_tenant` | 7 years | Tenant isolation breach evidence; SOC 2 CC6.6/PI1.5 |
| `asset.*` (device disposal) | 7 years | SOC 2 C1.2 confidential media disposal + CC6.5 access termination evidence |
| `vuln.*` (vulnerability management) | 7 years | SOC 2 CC7.1 threat identification + CC7.2 remediation tracking evidence; `vuln.sla_exception_granted` also maps to CC7.4 (response to identified events) |
| `compliance.erasure_sla_alert_fired` | 7 years | SOC 2 C1.2/P5.2 GDPR Art. 17 erasure SLA monitoring evidence; belt-and-suspenders secondary signal to §70 DSAR automation |
| `compliance.data_asset_inventory_reviewed` / `compliance.encryption_verified` / `compliance.disposal_audit_completed` | 3 years | C1 operating evidence cadence records; routine compliance audit completion records per SOC2_READINESS.md §73 |
| `enterprise.pricing_exception_approved` / `enterprise.consumer_price_updated` / `enterprise.list_price_updated` / `enterprise.price_floor_override_requested` | 7 years | SOC 2 CC1.4 pricing integrity evidence; COST_MODEL.md §31.8 pricing governance trail; CRITICAL-severity `enterprise.price_floor_override_requested` also maps to CC5.2 |
| `enterprise.ai_cost_monitor_alert` / `enterprise.ai_cost_outlier_flagged` | 3 years | FORM-internal AI token budget monitoring; operational cost governance record; SOC 2 A1.1 (capacity monitoring); COST_MODEL.md §33.9 |
| `enterprise.renewal_risk_flagged` | 3 years | Renewal risk classification record; SOC 2 CC5.2 (business risk assessment); COST_MODEL.md §34.8 — 3yr sufficient as risk band changes are operational signals, not contract evidence |
| `enterprise.churn_confirmed` / `enterprise.retention_discount_granted` | 7 years | Contract lifecycle evidence (churn type, ACV, contract_id FK); pricing exception linkage via `pricing_exception_event_id`; SOC 2 CC1.4/CC5.2; 7yr matches financial audit retention requirement per COST_MODEL.md §34.8 |
| `billing.user_erased` | 7 years | GDPR Art. 17 erasure record for `subscription_events`; SOC 2 P5.2 (ERA-E-001 pseudonymization evidence), P8.0 (ERA-E-002 financial retention with `retention_basis`), CC8.1 (ERA-E-003 migration + erasure chain); 7yr matches Ukrainian Tax Code Art. 44 and EU VAT Directive Art. 245 financial retention floor |
| `security.quota_95pct_warning` | 3 years | Tenant API quota soft-warn telemetry (95% consumption); SOC 2 A1.1 (capacity monitoring); operational deduplication evidence (secondary_warn_sent_at); 3yr sufficient — not a financial or contract record |
| `ai.safety_incident_opened` / `ai.safety_incident_contained` / `ai.victor_disabled` / `ai.victor_reenabled` | 7 years | Victor AI P0/P1 clinical-safety incident evidence; SOC 2 CC7.2 (threat monitoring) / CC7.4 (incident response); GDPR Art. 9 health data integrity record; VSAFETY-CHAIN-01/02/03 chain monitor evidence |
| `ai.safety_incident_resolved` | 3 years | Incident closure record; resolution method and post-incident controls; not required as long-term SOC 2 evidence (covered by the 7yr `ai.safety_incident_opened` record) |
| `vendor.sub_processor_added` | 7 years | SOC 2 CC9.2 / P6.1 sub-processor addition evidence; 30-day advance notice compliance record |
| `vendor.sub_processor_removed` | 3 years | SOC 2 CC9.2 sub-processor removal record; deletion certificate reference |
| `legal.*` (except `legal.transparency_tally_updated`) | 7 years | SOC 2 P6.5 government disclosure evidence; CC1.4 integrity record; A1.1 legal hold record; GDPR Art. 6 lawful processing documentation |
| `legal.transparency_tally_updated` | 3 years | Annual transparency report tally; operational record not required as long-term SOC 2 evidence |
| `support.*` | 10 years | Trust + future legal discovery |
| `support.unauthorized_nuke_attempt` | 7 years | Internal safety control violation record |
| `integration.api_call` (sampled) | 30 days | Volume management |
| `data.read_aggregate` | 90 days | Investigation but not unlimited |

After retention period, rows **hard-deleted via partition drop** (not soft-delete). Hash of deleted-partition's last row preserved у `audit_log_retention_summary` for chain continuity proof.

---

## Access controls

### Who can read audit logs

| Role | Scope |
|---|---|
| Tenant admin | Their tenant only, all actions |
| FORM Security Team | All tenants, read-only, 2-factor required |
| FORM Compliance Officer | All tenants, read-only, 2-factor required |
| FORM Engineering | **No default access.** Break-glass via approval workflow → logged as `support.*` |
| Tenant member | **No access.** Only own actions via `/me/activity` endpoint |

### Break-glass procedure

Engineer needs production debug access:
1. Open Linear ticket з justification
2. Get 2-person approval (manager + security-engineer)
3. Time-bounded role (4h max)
4. Every action logged as `support.data_accessed` with ticket ID
5. Customer notified within 24h if their data touched

---

## Export & delivery

Enterprise tenants can:
- Stream live audit log via **webhook** (`POST` to their endpoint)
- Daily **S3 bucket sync** (their bucket, our role assumes)
- Pull historical via **REST API** (`GET /v1/audit/logs?since=...&until=...`)
- SIEM integrations: **Datadog, Splunk, Sumo Logic** (native exporters)

Default format: JSON Lines (NDJSON). Optional CEF for SIEM.

---

## Privacy guarantees

- `actor_ip` — masked to /24 (IPv4) or /48 (IPv6) before storage. Full IP only retained 30 days for security investigation, then truncated.
- `actor_ua` — full string acceptable (not PII).
- `changes` JSONB redacted for sensitive fields. Specifically: `password`, `mfa_secret`, `api_key`, `health_data_blob`. Replaced with `"[REDACTED]"`.
- Individual user health/workout data **never** logged in `changes` — only metadata about access (who, when, scope).

---

## Implementation notes for `platform-engineer`

- Write path: every privileged action goes through `auditLog()` middleware. Failure to log = transaction abort (fail-closed).
- Async path NOT acceptable for `auth.*`, `tenant.*`, `support.*` (must be synchronous w/ transaction).
- Async path acceptable for `data.read_aggregate`, `integration.api_call` (high volume).
- Partition rotation cron daily at 02:00 UTC.
- HMAC validation cron weekly — alerts if chain broken.

---

## SLO

- Log durability: **99.999%** (5 nines — backed up 3 regions)
- Log latency: **P95 < 50ms** added to request
- Chain integrity verification: **weekly batch**, no in-band cost
- Export latency: webhook delivery **P95 < 5s** after event

---

**v1.5 · 2026-06-11 · owner: compliance-officer + enterprise-architect**
*v1.5 (2026-06-11): +2 events from DATA_MODEL §30 OQ-BILL-05/RL-02 resolution. (1) `billing.user_erased` (HIGH, 7yr, HMAC-chained) — GDPR Art. 17 erasure record for `subscription_events`; Zod schema v2 patched: `erased_user_reference: z.string().regex(/^\[ERASED-[0-9a-f]{64}\]$/)` required; `user_id: z.string().uuid()` demoted to `.optional()` (backward-compatible with pre-migration events); `rows_pseudonymized`, `erasure_request_id`, `pseudonymized_at` fields; emitter: `erasure-worker` (`form_system`), step SUB-1, synchronous inside erasure transaction; fail-closed; weekly chain audit checks post-migration events have `user_id = null` (non-null post-migration = P1 compliance anomaly). (2) `security.quota_95pct_warning` (STANDARD, 3yr, HMAC-chained) — fires when tenant request count crosses 95% of monthly quota (`QUOTA_SECONDARY_WARN_PCT = 0.95`); deduplicated via `api_quota_usage.secondary_warn_sent_at` (DATA_MODEL §30.3.3); at most one emission per tenant per billing period; payload: `tenant_id`, `period_month`, `endpoint_category`, `current_requests`, `quota_limit`, `pct_used`, `warn_threshold: 0.95`; no `user_id` (tenant-level quota event); emitter: `quota-check` Cloudflare Worker. Security & tenant isolation section header updated: "Four" → "Five" events (quota_95pct_warning added to table). New "Billing & GDPR erasure events" section added before API Key section. Retention table: +2 rows (`billing.user_erased` 7yr, `security.quota_95pct_warning` 3yr). Closes DATA_MODEL §30.7 items 4 (P0 M4 — billing.user_erased Zod patch) and 5 (P0 M4 — security.quota_95pct_warning registration). Cross-ref: DATA_MODEL §30.2.4 (SUB-1 erasure step), §30.3.2 (quota-check.ts update), §30.3.3 (secondary_warn_sent_at dedup), §30.5.1 (quota_95pct_warning schema), §30.5.2 (user_erased Zod patch); SOC2_READINESS.md P5.2 ERA-E-001/002/003; CRYPTOGRAPHY_POLICY.md §5/§6 (ERASURE_PSEUDONYM_SALT write-once key). Owner: compliance-officer + enterprise-architect.*

**v1.4 · 2026-06-11 · owner: compliance-officer + enterprise-architect**
*v1.4 (2026-06-11): +5 `enterprise.*` events — two AI token budget governance events from COST_MODEL.md §33.9 and three renewal & churn lifecycle events from COST_MODEL.md §34.8. (1) `enterprise.ai_cost_monitor_alert` (STANDARD, 3yr): nightly pg_cron fires when tenant token consumption crosses Yellow/Orange/Red overage band vs. §33.3 tier budget; payload: `budget_period`, `budget_tokens`, `actual_tokens`, `overage_pct`, `overage_band`, `alert_sent_to`; emitter: form_system. (2) `enterprise.ai_cost_outlier_flagged` (STANDARD, 3yr): monthly pg_cron fires when tenant per-seat spend ≥ 2σ above tier mean (minimum 5-tenant cohort); payload: `budget_period`, `tier`, `tenant_spend_usd`, `tier_mean_spend_usd`, `tier_stddev_usd`, `sigma_above_mean`; emitter: form_system. Privacy invariant both §33.9 events: tenant_id only, no user_id, no coaching content. (3) `enterprise.renewal_risk_flagged` (STANDARD, 3yr): OBSERVABILITY §33 alert routing on AL-ENGAGE-02/05/06 or renewal_proximity trigger; payload: `risk_band`, `chs_score`, `prior_risk_band`, `trigger_rule`, `days_to_renewal`, `alerted_to`; emitter: form_system. (4) `enterprise.churn_confirmed` (HIGH, 7yr): CSM confirms non-renewal or mid-contract termination after classifying churn_reason_category in CRM; payload: `churn_type`, `churn_effective_date`, `contract_id`, `acv_usd`, `final_risk_band`, `intervention_count`, `churn_reason_category`; emitter: customer-success (manual). (5) `enterprise.retention_discount_granted` (HIGH, 7yr): retention discount approved at renewal; `floor_respected` invariant always true; `pricing_exception_event_id` required for discount_pct > 15 (emit-audit-event Worker validates predecessor exists — HTTP 422 on absent); payload: `prior_acv_usd`, `new_acv_usd`, `discount_pct`, `effective_rate_per_seat`, `floor_respected`, `risk_band_at_grant`, `approver_role`, `pricing_exception_event_id`; emitter: customer-success (manual). Privacy invariant all §34.8 events: tenant-aggregate only, no user_id, no health metrics, no individual engagement data; verbatim exit interview notes → CRM only. Retention table: +3 rows (§33.9 3yr, `enterprise.renewal_risk_flagged` 3yr, §34.8 HIGH events 7yr). Header: v1.3 → v1.4 (v1.3 content was applied 2026-06-11 but header not incremented at that time). Closes COST_MODEL.md §33.10 checklist item 2 (P0/M5) and COST_MODEL.md §34.9 checklist item 1 (P0/M10). Cross-ref: COST_MODEL.md §33.3 (token budgets), §33.7 (overage bands), §33.9 (event definitions), §34.8 (event definitions), §34.9 (checklist); OBSERVABILITY.md §33 (CHS model, AL-ENGAGE alert rules); docs/DATA_MODEL.md §21.8 (tenant_monthly_coaching_cost view); docs/COST_MODEL.md §31.5 (price floors), §31.8 (pricing_exception_approved predecessor). Owner: compliance-officer + enterprise-architect.*

**v1.3 · 2026-06-11 · owner: compliance-officer + platform-engineer**
*v1.3 (2026-06-11): +1 `pam.break_glass_review_completed` event (HIGH, 7yr, two-person sign-off) — closes the break-glass DEC-030 HMAC chain (`pam.break_glass_activated` → `pam.break_glass_expired` → `pam.break_glass_review_completed`). Emitter: compliance-officer via `POST /internal/pam/break-glass-reviews/:pam_session_id/complete` when both reviewers have signed off in `pam_break_glass_reviews`. Payload: `pam_session_id`, `outcome` (3-value enum: no_anomaly / anomaly_detected / process_gap), `reviewed_by_security` UUID, `reviewed_by_compliance` UUID, `review_completed_at`, optional `github_issue_url`, optional `pg_audit_lines_reviewed`. Zod schema provided in-line (canonical source: DATA_MODEL.md §29.6). PAM section header updated: "Six" → "Seven" events; CC6.6 added to SOC 2 cross-ref. HMAC chain note extended: three-event break-glass lifecycle documented with AL-PAM-BG-01 enforcement reference. `emit-audit-event` Worker validates Zod schema before chain-append (HTTP 422 on violation). Closes DATA_MODEL.md §29.10 checklist item 6 (P0 M6 — register `pam.break_glass_review_completed` in AUDIT_LOG_SCHEMA.md and deploy to event registry). Cross-ref: DATA_MODEL.md §29.6 (canonical schema), SOC2_READINESS.md §24.8 (CC6.6 evidence PAM-E-004 — `pam_break_glass_reviews` all reviewed within 72h + this event chain), OBSERVABILITY.md §29.4 (AL-PAM-BG-01 enforces the 72h review SLA by querying `pam_break_glass_reviews.review_due_at`). Owner: compliance-officer + platform-engineer.*

**v1.2 · 2026-06-10 · owner: compliance-officer + security-engineer**
*v1.2 (2026-06-10): +2 `vendor.*` sub-processor management events (vendor.sub_processor_added HIGH/7yr, vendor.sub_processor_removed STANDARD/3yr) per SOC2_READINESS §75.2.4; +10 `legal.*` government request lifecycle events (legal.disclosure_executed and legal.disclosure_declined CRITICAL/7yr; 8× procedural HIGH/7yr; legal.transparency_tally_updated STANDARD/3yr) per SOC2_READINESS §75.5.3. LEGAL-CHAIN-01 chain guard documented (HTTP 422 on ordering violation). Privacy invariant: no plain-text authority names or user identifiers in any `legal.*` payload — `recipient_authority_hash` SHA-256 only. Retention table updated with 4 new rows. Closes SOC2_READINESS.md §75.8 checklist items 5 and 8 (both P1 Pre-GA). Cross-ref: docs/SUBPROCESSORS.md §8.2 (notification procedure), SOC2_READINESS §75.2.4 and §75.5.3, P6.1/P6.5/CC9.2. Owner: compliance-officer + platform-engineer.*

**v1.0 · 2026-06-10 · owner: compliance-officer + security-engineer**
*v1.0 (2026-06-10): +5 `ai.*` Victor AI safety events — `ai.safety_incident_opened` (CRITICAL/HIGH, 7yr), `ai.safety_incident_contained` (HIGH, 7yr), `ai.victor_disabled` (HIGH, 7yr), `ai.victor_reenabled` (HIGH, 7yr), `ai.safety_incident_resolved` (STANDARD, 3yr). All HMAC-chained per DEC-030. VSAFETY-CHAIN-01/02/03 ordering invariants enforced by `emit-audit-event` Worker write-guard (HTTP 422 on violation → PagerDuty P0). Privacy floor: no user_id, coaching content, or health values in any payload — incident_id (UUID) and affected_session_count (aggregate integer) only. Retention table updated with 2 new rows. Closes OBSERVABILITY.md §32.10 checklist item 7 (P0 M4) and INCIDENT_RESPONSE.md R-23 checklist item 2 (P0 M4). Cross-ref: OBSERVABILITY.md §32.5 (FORM-VICTOR-001 through -004 alert rules), §2.1 (VICTOR-SLO-01 through -04), §32.7 (VSAFETY-CHAIN monitors); INCIDENT_RESPONSE.md R-23 §R-23.3 (T+3min auto-disable path); SOC 2 CC7.2/CC7.4; GDPR Art. 9.*

**v0.9 · 2026-06-10 · owner: compliance-officer + security-engineer**
*v0.9 (2026-06-10): +4 `enterprise.*` pricing governance events — `enterprise.pricing_exception_approved` (HIGH, 7yr), `enterprise.consumer_price_updated` (HIGH, 7yr), `enterprise.list_price_updated` (HIGH, 7yr), `enterprise.price_floor_override_requested` (CRITICAL, 7yr). All HMAC-chained per DEC-030. Ordering rule: `enterprise.pricing_exception_approved` must precede quote dispatch; `enterprise.price_floor_override_requested` records even denied attempts. `enterprise.pricing_exception_approved` links to `enterprise.contract_signed` (§23.7) via `pricing_exception_event_id`. Privacy invariant: no PII, company names, or health values in any payload. Retention table updated with 1 new row for `enterprise.*` events (SOC 2 CC1.4/CC5.2). Emitter: founder (manual) for exception-approved / price-updated / list-price-updated; pricing calculator server-side API (automated) for floor-override-requested. Closes COST_MODEL.md §31.9 checklist item 1 (P0, M4 — register four pricing event types in AUDIT_LOG_SCHEMA.md; Zod schema validation and `emit-audit-event` Worker deployment are implementation items for platform-engineer + data-engineer per §31.9 item 1 DoD). Cross-ref: COST_MODEL.md §31.8 (event definitions), §31.6 (discount authority matrix), §31.5 (price floors); ENTERPRISE.md §Pricing; MSA_TEMPLATE.md (price escalation clause — §31.7.2); DECISION_LOG.md.*

**v0.8 · 2026-06-09 · owner: compliance-officer + devops-lead**
*v0.8 (2026-06-09): +4 `compliance.*` C1 operating evidence events — `compliance.data_asset_inventory_reviewed` (STANDARD, 3yr), `compliance.encryption_verified` (STANDARD, 3yr), `compliance.erasure_sla_alert_fired` (HIGH, 7yr), `compliance.disposal_audit_completed` (STANDARD, 3yr). All HMAC-chained; defined in `docs/SOC2_READINESS.md §73`. Privacy invariant: no user_id, dsar_request_id, or health values in any payload. Retention table updated with 2 new rows. Closes SOC2_READINESS.md §73 cross-reference to AUDIT_LOG_SCHEMA.md ("four new DEC-030 events"). Emitters: form_system for erasure_sla_alert_fired (automated pg_cron `c1-erasure-sla-monitor`); compliance-officer (manual) for the other three cadenced events. Cross-ref: OBSERVABILITY.md §6.2 `c1_erasure_sla` subsection (AL-C1-01); SOC2_READINESS.md §73.4 (C1 DEC-030 event table).*

**v0.7 · 2026-06-08 · owner: compliance-officer + devops-lead**
*v0.7 (2026-06-08): +3 `system.cron_*` pg_cron health monitoring events — `system.cron_job_stale` (HIGH, 7yr), `system.cron_health_check_passed` (LOW, 3yr), `system.cron_health_check_failed` (HIGH, 7yr). All HMAC-chained; emitted by `pg-cron-health-monitor` Supabase Edge Function (schedule: `0 * * * *`). Payload fields per §71.2.2 DEC-030 event table. Retention table updated with 3 new rows. Closes SOC2_READINESS.md §71.8 item 2 (P0 M5 — DEC-030 registration for all three cron health event types). Cross-ref: SOC2_READINESS.md §71.2.2 (9-job registry + freshness windows), OBSERVABILITY.md §12.6 (canonical pg_cron job registry). Privacy note: no `user_id` or `tenant_id` in any cron health event — operational metadata only.*

**v0.6 · 2026-06-07 · owner: compliance-officer + security-engineer**
*v0.6 (2026-06-07): +5 `vuln.*` vulnerability management events (vuln.cve_discovered, vuln.patch_deployed, vuln.sla_exception_granted, vuln.wont_fix_closed, vuln.enterprise_notified). All MEDIUM/HIGH severity, 7-year retention, HMAC-chained. Uplift rule U-02 (RLS bypass) enforced as Won't Fix blocker in chain audit. Retention table updated with `vuln.*` row. Closes `compliance/cc7/vuln-management-policy.md` (POL-010) checklist item 6 (P1 M3); cross-ref: SOC 2 CC7.1/CC7.2/CC5.3.*
*v0.4 (2026-06-05): +31 events across five new families: SSO authentication policy (9 events: sso.auth_policy_updated, sso.ip_allowlist_entry_added, sso.ip_allowlist_entry_removed, sso.ip_allowlist_toggled, sso.ip_blocked, sso.mfa_enforcement_changed, sso.mfa_enforcement_sweep_completed, sso.mfa_bypass_granted, sso.auth_policy_lockout_recovery); PAM/privileged access (6 events: pam.elevation_requested, pam.elevation_approved, pam.elevation_denied, pam.session_expired, pam.break_glass_activated, pam.break_glass_expired); Google Directory Sync (8 events: sso.google_directory_sync_success/cache_hit/error/credential_uploaded/credential_rotated/sync_enabled/sync_disabled/sync_validated); session revocation (5 events: session.bulk_revocation_started/complete, session.tenant_nuke_started/complete, session.revocation_kv_sync_error + support.unauthorized_nuke_attempt in support section); security/RLS isolation (3 events: security.rls_bypass_attempt, security.definer_function_cross_tenant, system.rls_test_suite_run). Retention table updated with 8 new rows for all new event classes. Closes: SSO_SCIM_IMPLEMENTATION.md §25.14 item 6 (P0 M4 — sso.auth_policy_* registration), §24.10 item 8 (P1 M4 — pam.* registration), §21 checklist item 5 (P0 M4 — sso.google_directory_* registration), §22 checklist item 6 (P0 M4 — session revocation registration); SOC2_READINESS.md §64.9 items 2 (P0 — security.rls_bypass_attempt, security.definer_function_cross_tenant) and 3 (P1 — system.rls_test_suite_run).*
*v0.3 (2026-06-05): +privacy.floor_breach_detected, +privacy.floor_breach_contained, +privacy.floor_breach_tenant_notified, +privacy.floor_breach_resolved, +privacy.no_go_escalation_activated. +privacy.floor_breach_* retention row. Closes INCIDENT_RESPONSE.md R-22 §checklist item 6 (P0 — DEC-030 registration for all five privacy floor breach events). Events are HMAC-chained, CRITICAL/HIGH/STANDARD severity per R-22 §DEC-030 table; payloads canonical to INCIDENT_RESPONSE.md §R-22 lines 7848–7854.*
*v0.5 (2026-06-07): +5 asset.* device disposal events (asset.disposal_initiated, asset.disposal_completed, asset.device_sanitized, asset.chain_of_custody_transferred, asset.disposal_log_filed). Retention table updated with `asset.*` 7-year row. Closes MDD-P0-03 (SOC2_READINESS §66.12); cross-ref: compliance/c1/device-disposal-policy.md POL-013 §8, SOC 2 C1.2/CC6.5/CC6.7.*
*v0.2 (2026-06-05): +system.access_review_completed, +system.credential_rotated, +admin.encryption_key_rotated, +admin.signing_key_rotated. Closes SOC2_READINESS §65.13 AR-P1-03/AR-P1-04/AR-P1-05.*
