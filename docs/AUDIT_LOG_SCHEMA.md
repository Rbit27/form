# FORM · Audit Log Schema v0.3

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

### Admin (key management)

> HIGH severity · 7yr retention · HMAC-chained. Triggers PagerDuty P2 alert on unexpected rotation (outside scheduled window). Emitted synchronously inside KMS transaction — failure aborts rotation.

- `admin.encryption_key_rotated` — master encryption key rotated in KMS; payload: `{key_id, key_version_old, key_version_new, rotation_trigger: 'scheduled'|'incident'|'compromise', rotated_by, kms_provider}`; HIGH severity, 7yr retention; cross-ref: SOC2_READINESS §56, OBSERVABILITY §30.10 item 10, DEC-030
- `admin.signing_key_rotated` — HMAC chain signing key rotated; payload: `{key_id, rotation_trigger, rotated_by}`; HIGH severity, 7yr retention; DEC-030 chain continuity preserved via `key_transition` metadata

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
| `admin.*` (key management) | 7 years | Encryption governance + incident investigation |
| `support.*` | 10 years | Trust + future legal discovery |
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

**v0.3 · червень 2026 · owner: compliance-officer + security-engineer**
*v0.3 (2026-06-05): +privacy.floor_breach_detected, +privacy.floor_breach_contained, +privacy.floor_breach_tenant_notified, +privacy.floor_breach_resolved, +privacy.no_go_escalation_activated. +privacy.floor_breach_* retention row. Closes INCIDENT_RESPONSE.md R-22 §checklist item 6 (P0 — DEC-030 registration for all five privacy floor breach events). Events are HMAC-chained, CRITICAL/HIGH/STANDARD severity per R-22 §DEC-030 table; payloads canonical to INCIDENT_RESPONSE.md §R-22 lines 7848–7854.*
*v0.2 (2026-06-05): +system.access_review_completed, +system.credential_rotated, +admin.encryption_key_rotated, +admin.signing_key_rotated. Closes SOC2_READINESS §65.13 AR-P1-03/AR-P1-04/AR-P1-05.*
