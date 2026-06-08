# FORM · Audit Log Schema v0.6

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

### API Key credential events (DEC-030 HMAC-chained · HIGH severity · 7yr retention)

> Defined in `docs/SSO_SCIM_IMPLEMENTATION.md §26.6`. All 9 events are HIGH severity with 7-year retention. `*.ip_blocked` events carry `client_ip_hash` (HMAC-SHA256 of client IP with `IP_HASH_SALT`) — no plaintext IP ever enters the audit chain (DEC-030 privacy floor).

- `api_key.created` — new tenant API key created; payload: `{tenant_id, key_preview, scopes[], label, ip_enforcement_enabled, created_by}`
- `api_key.rotated` — key rotation initiated; payload: `{tenant_id, old_key_preview, new_key_preview, overlap_expires_at, rotated_by}`; **chain requirement:** must be followed by `api_key.revoked` (old key) within 26h or AL-APIKEY-02 fires
- `api_key.revoked` — key revoked; payload: `{tenant_id, key_preview, reason: 'manual'|'expiry_overlap'|'compromise', revoked_by}`
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

> Defined in `docs/SSO_SCIM_IMPLEMENTATION.md §24.6`. Six events covering JIT privilege escalation lifecycle and break-glass protocol. Privacy floor: `justification_hash` stores SHA-256 of the justification text — the original text is retained in `PAM_KV` (7yr TTL) but never enters the audit chain; `pam.break_glass_activated` uses `cf_access_jti_hash` (SHA-256 of the Cloudflare Access JTI — not the raw JTI). `target_tenant_id` is nullable (null means FORM infrastructure, not a customer tenant). CRITICAL event `pam.break_glass_activated` triggers immediate PagerDuty P0 (AL-PAM-01) and compliance-officer email. Break-glass sessions require a mandatory post-hoc two-person review within 72h, linked via `pam_session_id`. Cross-ref: SOC 2 CC6.1/CC6.2/CC6.3/CC6.7; closes SSO_SCIM_IMPLEMENTATION.md §24.10 checklist item 8 (P1 M4).

| Event type | Severity | Retention | Two-person rule | Payload fields |
|---|---|---|---|---|
| `pam.elevation_requested` | HIGH | 7 yr | No | `admin_user_id`, `access_level` (`read_only` / `read_write` / `destructive`), `pam_request_id`, `target_tenant_id` (nullable), `justification_hash` (SHA-256) |
| `pam.elevation_approved` | HIGH | 7 yr | Pre-auth for `read_write`/`destructive` | `admin_user_id`, `approver_id` (nullable for `read_only`), `access_level`, `pam_session_id`, `expires_at` (ISO 8601), `requester_webauthn_credential_id` (nullable), `approver_webauthn_credential_id` (nullable) |
| `pam.elevation_denied` | HIGH | 7 yr | No | `admin_user_id`, `access_level`, `pam_request_id`, `reason` (`self_approval_attempted` / `approval_timeout` / `cosign_timeout` / `totp_failure` / `webauthn_failure` / `policy_violation`) |
| `pam.session_expired` | STANDARD | 7 yr | No | `admin_user_id`, `pam_session_id`, `access_level`, `expired_at` (ISO 8601) — emitted by `pam-expiry-sweeper`; distinct from `pam.elevation_denied` (session completed its full TTL) |
| `pam.break_glass_activated` | CRITICAL | 7 yr | Post-hoc (72h review, §24.4.3) | `admin_user_id`, `pam_session_id`, `cf_access_jti_hash` (SHA-256), `activated_at` (ISO 8601); triggers PagerDuty P0 + compliance email immediately on emission |
| `pam.break_glass_expired` | HIGH | 7 yr | No | `admin_user_id`, `pam_session_id`, `expired_at` (ISO 8601), `review_issue_url` (GitHub issue auto-created at activation) |

**HMAC chain requirement:** `pam.elevation_approved` must follow `pam.elevation_requested` with the same `pam_request_id` in the chain. If `pam.break_glass_activated` is emitted without a preceding `pam.elevation_requested` for the same session (break-glass is a separate Cloudflare Access path), this is expected — do not treat the missing predecessor as a chain violation. A post-hoc `pam.elevation_approved` is not emitted for break-glass; the `pam.break_glass_activated` CRITICAL event is the authorisation record.

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

> Defined in `docs/SOC2_READINESS.md §64.7`. Three events covering RLS tenant isolation monitoring and CI test evidence. Privacy floor: `request_ip_hash` in `security.rls_bypass_attempt` is SHA-256 of the client IP with a daily rotating salt (per DPIA §4 constraint) — never a raw IP. `security.definer_function_cross_tenant` must never be aggregated or batched — emit synchronously per invocation (HIGH severity treated equivalent to a confirmed RLS bypass; triggers immediate P0 incident via R-09). Cross-ref: SOC 2 CC6.1/CC6.3/CC6.6/PI1.5-C3; closes SOC2_READINESS.md §64.9 checklist items 2 (P0) and 3 (P1).

| Event type | Severity | Retention | Emitter | Payload fields |
|---|---|---|---|---|
| `security.rls_bypass_attempt` | MEDIUM | 7 yr | Cloudflare Worker `rls-guard` middleware | `tenant_id_in_jwt` (UUID), `tenant_id_in_filter` (UUID), `table_name`, `query_path` (e.g. `/rest/v1/workout_sessions`), `request_ip_hash` (SHA-256 of IP + daily salt — **not raw IP**) |
| `security.definer_function_cross_tenant` | HIGH | 7 yr | Cloudflare Worker or Supabase Edge Function `definer-audit` wrapper | `function_name`, `caller_tenant_id` (UUID), `returned_tenant_ids` (UUID array — distinct tenant_ids that do not match caller), `row_count` (total cross-tenant rows) — triggers PagerDuty P0 immediately |
| `system.rls_test_suite_run` | STANDARD | 3 yr | CI workflow step in `rls-isolation-tests.yml` | `run_id` (GitHub Actions run_id), `ci_build_id` (run_id + run_attempt), `git_sha` (40-char), `tc_rls_001_result` / `tc_rls_002_result` / `tc_rls_003_result` / `tc_rls_004_result` / `tc_rls_005_result` (each: `PASS` / `FAIL` / `SKIP`), `overall_verdict` (`PASS` / `FAIL`), `evidence_path` (R2 object key for PRE-36-E-007 artefact) |

**HMAC chain requirement:** `security.rls_bypass_attempt` is expected to be low-volume under normal conditions (rare client bugs or adversarial probes) — no Queues batching needed. A spike > 10 events / 10 min for a single `tenant_id_in_filter` value should be escalated to R-09 Security Incident. `system.rls_test_suite_run` fires on every push to `main` — this is a high-frequency routine event; STANDARD severity is intentional.

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

**v0.7 · 2026-06-08 · owner: compliance-officer + devops-lead**
*v0.7 (2026-06-08): +3 `system.cron_*` pg_cron health monitoring events — `system.cron_job_stale` (HIGH, 7yr), `system.cron_health_check_passed` (LOW, 3yr), `system.cron_health_check_failed` (HIGH, 7yr). All HMAC-chained; emitted by `pg-cron-health-monitor` Supabase Edge Function (schedule: `0 * * * *`). Payload fields per §71.2.2 DEC-030 event table. Retention table updated with 3 new rows. Closes SOC2_READINESS.md §71.8 item 2 (P0 M5 — DEC-030 registration for all three cron health event types). Cross-ref: SOC2_READINESS.md §71.2.2 (9-job registry + freshness windows), OBSERVABILITY.md §12.6 (canonical pg_cron job registry). Privacy note: no `user_id` or `tenant_id` in any cron health event — operational metadata only.*

**v0.6 · 2026-06-07 · owner: compliance-officer + security-engineer**
*v0.6 (2026-06-07): +5 `vuln.*` vulnerability management events (vuln.cve_discovered, vuln.patch_deployed, vuln.sla_exception_granted, vuln.wont_fix_closed, vuln.enterprise_notified). All MEDIUM/HIGH severity, 7-year retention, HMAC-chained. Uplift rule U-02 (RLS bypass) enforced as Won't Fix blocker in chain audit. Retention table updated with `vuln.*` row. Closes `compliance/cc7/vuln-management-policy.md` (POL-010) checklist item 6 (P1 M3); cross-ref: SOC 2 CC7.1/CC7.2/CC5.3.*
*v0.4 (2026-06-05): +31 events across five new families: SSO authentication policy (9 events: sso.auth_policy_updated, sso.ip_allowlist_entry_added, sso.ip_allowlist_entry_removed, sso.ip_allowlist_toggled, sso.ip_blocked, sso.mfa_enforcement_changed, sso.mfa_enforcement_sweep_completed, sso.mfa_bypass_granted, sso.auth_policy_lockout_recovery); PAM/privileged access (6 events: pam.elevation_requested, pam.elevation_approved, pam.elevation_denied, pam.session_expired, pam.break_glass_activated, pam.break_glass_expired); Google Directory Sync (8 events: sso.google_directory_sync_success/cache_hit/error/credential_uploaded/credential_rotated/sync_enabled/sync_disabled/sync_validated); session revocation (5 events: session.bulk_revocation_started/complete, session.tenant_nuke_started/complete, session.revocation_kv_sync_error + support.unauthorized_nuke_attempt in support section); security/RLS isolation (3 events: security.rls_bypass_attempt, security.definer_function_cross_tenant, system.rls_test_suite_run). Retention table updated with 8 new rows for all new event classes. Closes: SSO_SCIM_IMPLEMENTATION.md §25.14 item 6 (P0 M4 — sso.auth_policy_* registration), §24.10 item 8 (P1 M4 — pam.* registration), §21 checklist item 5 (P0 M4 — sso.google_directory_* registration), §22 checklist item 6 (P0 M4 — session revocation registration); SOC2_READINESS.md §64.9 items 2 (P0 — security.rls_bypass_attempt, security.definer_function_cross_tenant) and 3 (P1 — system.rls_test_suite_run).*
*v0.3 (2026-06-05): +privacy.floor_breach_detected, +privacy.floor_breach_contained, +privacy.floor_breach_tenant_notified, +privacy.floor_breach_resolved, +privacy.no_go_escalation_activated. +privacy.floor_breach_* retention row. Closes INCIDENT_RESPONSE.md R-22 §checklist item 6 (P0 — DEC-030 registration for all five privacy floor breach events). Events are HMAC-chained, CRITICAL/HIGH/STANDARD severity per R-22 §DEC-030 table; payloads canonical to INCIDENT_RESPONSE.md §R-22 lines 7848–7854.*
*v0.5 (2026-06-07): +5 asset.* device disposal events (asset.disposal_initiated, asset.disposal_completed, asset.device_sanitized, asset.chain_of_custody_transferred, asset.disposal_log_filed). Retention table updated with `asset.*` 7-year row. Closes MDD-P0-03 (SOC2_READINESS §66.12); cross-ref: compliance/c1/device-disposal-policy.md POL-013 §8, SOC 2 C1.2/CC6.5/CC6.7.*
*v0.2 (2026-06-05): +system.access_review_completed, +system.credential_rotated, +admin.encryption_key_rotated, +admin.signing_key_rotated. Closes SOC2_READINESS §65.13 AR-P1-03/AR-P1-04/AR-P1-05.*
