# FORM · Audit Log Schema v2.13

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

### Privacy Incident Lifecycle events (DEC-030 HMAC-chained · SOC2_READINESS §77 · P8.2/P8.3)

> Defined in `docs/SOC2_READINESS.md §77`. Three events forming the immutable register of privacy incidents (PI-P0–PI-P3 severity taxonomy). **PRIV-INC-CHAIN-01 ordering invariant:** `privacy.incident_reviewed` and `privacy.incident_closed` must NOT be emitted for an `incident_id` with no preceding `privacy.incident_opened` — violation returns HTTP 422 (blocking). PI-P0/P1 incidents (`art9_suspected = true` or `severity_class IN ('PI-P0','PI-P1')`) trigger PagerDuty `form-privacy` CRITICAL page to compliance-officer + security-engineer + founder simultaneously. **Privacy floor (all three events):** no health values, no coaching turn content, no employee names; `incident_id` UUID is the sole cross-reference between chain events. `scope_assessment_notes` and `recurrence_prevention` are capped at 500 chars and must contain no user PII or tenant contact details. Cross-ref: `docs/SOC2_READINESS.md §77` (full classification taxonomy, runbook mapping, evidence artefacts PRIV-PI-E-001–004); `docs/INCIDENT_RESPONSE.md §15` (GDPR Art. 33/34 breach notification procedure — triggered by PI-P0/P1); `docs/INCIDENT_RESPONSE.md R-11` (biometric incident); `docs/INCIDENT_RESPONSE.md R-22` (enterprise privacy floor — PI-P0 escalation path); SOC 2 P8.2 (privacy incident identification and investigation), P8.3 (individual data accounting on request), CC7.4 (external party notification). Closes SOC2_READINESS.md §77.10 checklist item 1 (P0, M8).

- `privacy.incident_opened` — first detection of any privacy incident; **must be emitted BEFORE any investigation or containment action**; **CRITICAL severity for PI-P0/P1** (PagerDuty `form-privacy` immediate triple-page); **STANDARD severity for PI-P2/P3**; 7yr retention (PI-P0/P1), 3yr retention (PI-P2/P3); PRIV-INC-CHAIN-01 anchor
- `privacy.incident_reviewed` — scope assessment complete; Art. 33 determination made by compliance-officer; **HIGH severity; 7yr retention**; required for PI-P0/P1/P2; **optional for PI-P3** (process deviations often resolve in a single step — per OQ-PRIV-INC-02 resolution 2026-06-13); PRIV-INC-CHAIN-01 enforces predecessor `privacy.incident_opened` for same `incident_id`
- `privacy.incident_closed` — all remediation complete; PIR filed for PI-P0/P1 (`post_incident_review_filed: true` required); `art33_notification_id` present when GDPR Art. 33 notification was submitted to supervisory authority; **HIGH severity; 7yr retention**; PRIV-INC-CHAIN-01 enforces predecessor `privacy.incident_opened` for same `incident_id`

---

### Privacy no-go commercial ethics events (DEC-030 HMAC-chained · COST_MODEL §39.8.4 · CC1.4/CC9.2)

> Defined in `docs/COST_MODEL.md §39.8.4`. One DEC-030 HMAC-chained event recording each instance where FORM's commercial no-go criteria are operationally applied to reject a prospective enterprise customer. This is an ethical commitment event, not a privacy breach event — it provides auditors and investors with immutable evidence that FORM's stated no-go criteria (insurance risk scoring, government backdoors, wellness-as-punishment) are operationally enforced and not merely policy text. **Auto-companion invariant:** `privacy.no_go_criteria_applied` is always emitted as an atomic companion to `enterprise.deal_closed_lost` when `no_go_criteria_triggered = true` — it is never emitted standalone. The Worker appends it as the second event in the same array payload; missing companion on a `no_go = true` lost event is a chain anomaly detected by the weekly audit. **Privacy floor:** No prospect company name, contact email, or individual `user_id` in payload. `criteria_triggered` is a structured enum — no free-text rationale in the chain. Verbatim documentation of the rejection decision remains in CRM only. `review_confirmed_by` is the founder UUID (FORM-internal — not a prospect contact). Cross-ref: `docs/ENTERPRISE.md §"No-go customers"` (four criteria: insurance risk scoring, government backdoor requests, wellness-as-punishment, other no-go); `enterprise.deal_closed_lost` (§Enterprise Deal Close — predecessor event, same `deal_id`); `docs/COST_MODEL.md §39.9` (WIN-E-003 SOC 2 evidence — quarterly export, zero-count quarters filed as affirmative attestation); SOC 2 CC1.4 (commitment to integrity and ethical values — operationally enforced), CC9.2 (customer selection governance).

| Event type | Severity | Retention | Trigger | Key payload fields |
|---|---|---|---|---|
| `privacy.no_go_criteria_applied` | STANDARD | 3 yr | Automated companion — appended immediately after `enterprise.deal_closed_lost` in the same Worker request when `no_go_criteria_triggered = true`; emitter: `form_system` (auto) | `deal_id` (UUID — matches `enterprise.deal_closed_lost.deal_id`), `criteria_triggered` (enum: `insurance_risk_scoring`\|`government_backdoor_request`\|`wellness_as_punishment_use_case`\|`other_no_go`), `review_confirmed_by` (UUID — founder UUID; FORM-internal; not a prospect contact) |

**Chain ordering:** `privacy.no_go_criteria_applied` is always the second event in a two-event atomic array payload: `[enterprise.deal_closed_lost, privacy.no_go_criteria_applied]`. Both events share the same `prev_hash` anchor. The `emit-audit-event` Worker enforces this ordering — it cannot be emitted without a preceding `enterprise.deal_closed_lost` in the same request.

**Zero-count quarterly filing:** Even in quarters where no no-go criteria are triggered, compliance-officer files a signed attestation (`n = 0`) as WIN-E-003. This positive attestation is SOC 2 CC1.4 evidence that no applicable prospect was presented with a no-go use case during the quarter.

**3-year retention rationale:** No health data; minimal financial sensitivity. 3 years covers two SOC 2 observation windows. Competitive sensitivity (which criteria are triggered) degrades as market conditions evolve.

**Zod v2 schema (canonical source: `docs/COST_MODEL.md §39.8.4`):**

```typescript
const PrivacyNoGoCriteriaAppliedSchema = z.object({
  deal_id:             z.string().uuid(),
  criteria_triggered:  z.enum([
    'insurance_risk_scoring',
    'government_backdoor_request',
    'wellness_as_punishment_use_case',
    'other_no_go',
  ]),
  review_confirmed_by: z.string().uuid(),
});
```

**SOC 2 evidence artefact:**

| Artefact | SOC 2 criterion | Source | Retention |
|---|---|---|---|
| **WIN-E-003** | CC1.4 (commitment to integrity and ethical values — operationally enforced) / CC9.2 (customer selection governance) | Quarterly export of `privacy.no_go_criteria_applied` chain events; zero-count quarters filed as a signed zero-event attestation; `compliance/evidence/winloss/WIN-E-003_<YYYY-QN>.csv` | 3 yr |

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
| `sso.ip_blocked` | HIGH | 7 yr | `tenant_id`, `client_ip_hash` (SHA-256[:32] — **no `user_id`; emitted before credential validation**), `enforcement_point` (`saml_callback` / `oidc_callback` / `token_refresh` / `admin_api` / `scim_api`), `auth_path` (`'sso'` \| `'api_key'`, optional — default `'sso'`; added in DEC-062 §32.4 to distinguish API-key-path blocks from SSO-path blocks in evidence exports), `policy_version` | Request blocked by IP allowlist enforcement |
| `sso.mfa_enforcement_changed` | HIGH | 7 yr | `tenant_id`, `changed_by_user_id`, `mode_before` (`none` / `recommended` / `required`), `mode_after`, `mfa_grace_hours`, `estimated_sessions_affected` | `mfa_enforcement_mode` changed via `PATCH /v1/admin/sso/policy` |
| `sso.mfa_enforcement_sweep_completed` | STANDARD | 3 yr | `tenant_id`, `sessions_revoked`, `sessions_already_satisfied`, `sweep_triggered_at` (ISO 8601), `grace_expired_at` (ISO 8601) | MFA grace period ends; hourly sweep job runs |
| `sso.mfa_bypass_granted` | CRITICAL | 7 yr | `tenant_id`, `granted_by_user_id` (tenant_admin — FORM support cannot grant), `subject_user_id`, `bypass_duration_hours` (max 48), `reason` (free text, max 500 chars) | Tenant admin grants per-user MFA bypass via `PATCH /v1/admin/users/{id}/mfa-bypass` |
| `sso.auth_policy_lockout_recovery` | CRITICAL | 7 yr | `tenant_id`, `recovery_type` (`ip_allowlist_disabled` / `policy_reset`), `recovery_performed_by` (PAM session ID — not user_id; FORM-internal action), `pam_session_id`, `affected_config_field`, `customer_notified` (bool) | FORM support performs lockout recovery via PAM `read_write` elevation (§25.8.1) |

**HMAC chain requirement:** All nine events must include `prev_hash`. `sso.ip_blocked` is the highest-volume event in this family; under misconfiguration lockout scenarios it may emit hundreds per minute — use Cloudflare Queues (FIFO consumer) for block events to smooth write pressure while maintaining chain ordering. `sso.auth_policy_lockout_recovery` must reference the `pam_session_id` of the PAM elevation used to perform the recovery, linking the policy change to the privileged access record.

---

### Mobile SSO Browser Mode events (DEC-030 HMAC-chained · SSO §30 · SOC 2 CC6.1/CC6.2/CC8.1)

> Defined in `docs/SSO_SCIM_IMPLEMENTATION.md §30.9` (v2.2, 2026-06-14 — DEC-059). Two events covering the mobile SSO system-browser mandate and WebView-violation detection. Enterprise tier only. **Privacy floor (both events):** No employee `user_id`, name, email, or health data in any payload. `attempted_url_hash` in `sso.mobile_webview_blocked` is SHA-256 of the IdP auth URL truncated to the first 32 hex characters — the plaintext URL is never stored in the audit chain. `changed_by_user_id` in `sso.mobile_browser_mode_set` is a `tenant_admin` or `tenant_owner` UUID (configuration actor — not the end-user authenticating). **Zero-fire invariant for `sso.mobile_webview_blocked`:** this event must never appear in production; its presence constitutes an incident indicator that immediately triggers AL-SSO-WEB-01 (P1, PagerDuty `form-security`; `docs/OBSERVABILITY.md §26.8`). **MOBILE-CHAIN-01 ordering invariant:** `sso.mobile_webview_blocked` must be preceded in the HMAC chain by `sso.mobile_browser_mode_set` with `mobile_sso_enabled: true` for the same `tenant_id`; a `blocked` event appearing before `mode_set` is a MOBILE-CHAIN-01 violation → R-05 (`docs/INCIDENT_RESPONSE.md R-05`). Cross-ref: `docs/SSO_SCIM_IMPLEMENTATION.md §30.9` (canonical Zod schemas); `docs/SSO_SCIM_IMPLEMENTATION.md §30.10` (AL-SSO-WEB-01 alert spec + triage tree); `docs/OBSERVABILITY.md §26.8` (AL-SSO-WEB-01 registered); `docs/INCIDENT_RESPONSE.md R-05` (chain integrity violation response); `docs/INCIDENT_RESPONSE.md R-08` (supply-chain concern if AL-SSO-WEB-01 fires via rogue SDK injection); DEC-059; SOC 2 CC6.1 (process isolation — `ASWebAuthenticationSession` / Chrome Custom Tabs in separate OS process), CC6.2 (OAuth 2.0 BCP — RFC 9700 §4.1 mandate; embedded WebView prohibited), CC8.1 (SDLC security testing — `no-webview-in-sso.test.ts` CI gate + ESLint `no-restricted-imports` rule). **Closes SSO_SCIM_IMPLEMENTATION.md §30.13 checklist item 4 (P0, M8 — register both DEC-030 events in AUDIT_LOG_SCHEMA.md §6.SSO-Mobile before first enterprise mobile SSO tenant goes live).**

| Event type | Severity | Retention | Payload fields | Trigger |
|---|---|---|---|---|
| `sso.mobile_browser_mode_set` | STANDARD | 7 yr | `tenant_id`, `mobile_sso_enabled` (bool), `callback_scheme` (enum: `universal_link` \| `app_link` \| `custom_scheme`), `idp_type` (enum — same values as `tenant_sso_configs.idp_type`), `sso_protocol` (`saml` \| `oidc`), `changed_by_user_id` (UUID — `tenant_admin` or `tenant_owner`; **never the end-user**) | Tenant enables or disables mobile SSO via `PATCH /enterprise/sso/mobile-mode`; `tenant_sso_configs.mobile_sso_enabled` persisted |
| `sso.mobile_webview_blocked` | **HIGH** | 7 yr | `tenant_id`, `attempted_url_hash` (SHA-256 of IdP auth URL, **first 32 hex chars only** — plaintext URL never stored), `source_module` (string ≤ 100 chars, e.g. `"sso-browser.ts"`), `blocked_at` (ISO 8601) | Any path in `apps/mobile/src/auth/` attempts a WebView open matching an SSO endpoint pattern; runtime guard intercepts and emits this event before allowing the open; **zero-fire expected in production — any emission is an incident** |

**HMAC chain requirement:** Both events must include `prev_hash`. `sso.mobile_webview_blocked` is HIGH severity and emitted synchronously before the WebView is permitted — a chain-append failure aborts the open (fail-closed). `sso.mobile_browser_mode_set` is emitted synchronously in API middleware — failure blocks the configuration write. Any MOBILE-CHAIN-01 violation triggers R-05 with no grace period.

**Emitter assignments:** `sso.mobile_browser_mode_set` — API Workers middleware (`form_system`, on behalf of authenticated `tenant_admin` / `tenant_owner` request). `sso.mobile_webview_blocked` — React Native runtime guard in `apps/mobile/src/auth/sso-browser.ts` (`form_system`, automated; fires only if CI guard is bypassed by a future code change or dependency injection).

```typescript
// sso.mobile_browser_mode_set — configuration state event (7yr, STANDARD)
const SsoMobileBrowserModeSet = z.object({
  event:              z.literal('sso.mobile_browser_mode_set'),
  tenant_id:          z.string().uuid(),
  mobile_sso_enabled: z.boolean(),
  callback_scheme:    z.enum(['universal_link', 'app_link', 'custom_scheme']),
  idp_type:           z.enum([
    'okta_oidc', 'okta_saml', 'entra_oidc', 'entra_saml',
    'azure_ad_b2c_oidc', 'google_workspace_oidc',
    'pingfederate_saml', 'onelogin_saml', 'generic_saml',
  ]),
  sso_protocol:       z.enum(['saml', 'oidc']),
  changed_by_user_id: z.string().uuid(),
  // No employee PII, no health data, no coaching content
});

// sso.mobile_webview_blocked — zero-fire violation detection (7yr, HIGH)
// Expected production occurrence: zero. One event = one incident → AL-SSO-WEB-01.
const SsoMobileWebviewBlocked = z.object({
  event:              z.literal('sso.mobile_webview_blocked'),
  tenant_id:          z.string().uuid(),
  attempted_url_hash: z.string().regex(/^[a-f0-9]{32}$/), // SHA-256[:32] only — plaintext URL never stored
  source_module:      z.string().max(100),
  blocked_at:         z.string().datetime(),
  // No user_id, no plaintext URL, no health data
});
```

**SOC 2 evidence mapping:**

| Artefact ID | Criteria | Description | Cadence | Retention |
|---|---|---|---|---|
| **SSO-WEB-E-001** | CC6.1 — Process isolation; CC6.2 — OAuth 2.0 BCP compliance | Annual export of `sso.mobile_browser_mode_set` HMAC-chain events for all tenants with `mobile_sso_enabled: true` during the observation period; demonstrates the system-browser mandate was active for every mobile SSO tenant. SHA-256 checksum in MASTER-INDEX before upload. | Annual (observation-period close) | 7 yr |

**Auditor narrative for CC6.1:** FORM's mobile SSO authentication uses `ASWebAuthenticationSession` (iOS) and Chrome Custom Tabs (Android), each running in a separate OS-managed process. The host application has no access to authentication credentials, session cookies, or DOM content during the SSO flow. SSO-WEB-E-001 (`sso.mobile_browser_mode_set` chain export) demonstrates this control was active for every enterprise tenant with mobile SSO enabled. The zero-fire `sso.mobile_webview_blocked` attestation (SSO-WEB-E-002, defined at `docs/SSO_SCIM_IMPLEMENTATION.md §30.11`) confirms no in-process WebView authentication occurred during the observation period.

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

### SCIM Provisioning Lifecycle events (DEC-030 HMAC-chained · SSO §27 · SOC 2 CC6.1/CC6.3/CC6.4)

> Defined in `docs/SSO_SCIM_IMPLEMENTATION.md §27.10` (v1.9, 2026-06-13). Six events covering the complete SCIM provisioning lifecycle: user creation, reactivation, deprovisioning, attribute updates, group sync, and sensitive-attribute rejection. These are the canonical operational events emitted by the SCIM endpoint Worker (`apps/scim-worker/`). **Canonical name note:** `scim.user_deactivated` (used as an interim name in `docs/SSO_SCIM_IMPLEMENTATION.md §3.5`) is deprecated; the canonical name is `scim.user_deprovisioned` — consistent with INCIDENT_RESPONSE.md R-24 mass deprovisioning detection clause. All existing staging instrumentation using `scim.user_deactivated` must be renamed. **Privacy invariant (all six events):** no employee name, email address, or health/coaching data in any payload. `external_user_id` is the IdP-side opaque stable identifier. `user_id` is FORM-internal UUID. No health data linkage in any SCIM chain event. **HMAC chain requirement (SCIM-CHAIN-01):** for a given `user_id`, `scim.user_deprovisioned` must not precede `scim.user_provisioned` in the chain; violation → HIGH WARNING (non-blocking) + AL-SCIM-04 P1. Cross-ref: SOC 2 CC6.1 (logical access grant/revoke — SCIM-PROV-E-001), CC6.3 (timely access removal — SCIM-PROV-E-002; delta to session revocation ≤ 60 s per `ENTERPRISE_SLA.md §3.7`), CC6.4 (restricted access via Art. 9 attribute rejection — SCIM-PROV-E-003), CC7.2 (AL-SCIM-01/04 — SCIM-PROV-E-004); closes SSO_SCIM_IMPLEMENTATION.md §27.14 checklist item 1 (P0 M4). Requires G-013 DPA gate cleared before production use.

| Event type | Severity | Retention | Trigger | Key payload fields |
|---|---|---|---|---|
| `scim.user_provisioned` | HIGH | 7 yr | Successful `POST /scim/v2/Users` resulting in new `tenant_users` INSERT | `tenant_id`, `user_id` (FORM UUID), `external_user_id` (IdP opaque stable ID), `scim_request_id` (UUID, dedup key), `provisioning_source: 'scim'`, `assigned_role` (default `member`) |
| `scim.user_reactivated` | HIGH | 7 yr | `POST /scim/v2/Users` where user existed with `is_active = false`; or `PATCH /Groups` member-add targeting inactive user | `tenant_id`, `user_id`, `external_user_id`, `scim_request_id`, `prior_deactivated_at` (ISO 8601) |
| `scim.user_deprovisioned` | HIGH | 7 yr | `PATCH /scim/v2/Users/:id` with `active: false`, or `DELETE /scim/v2/Users/:id` | `tenant_id`, `user_id`, `external_user_id`, `scim_request_id`, `deprovision_method` (`scim_patch_inactive` \| `scim_delete`), `sessions_revoked_count` (integer) |
| `scim.user_updated` | STANDARD | 7 yr | `PUT` or `PATCH /scim/v2/Users/:id` updating non-activation attributes | `tenant_id`, `user_id`, `scim_request_id`, `update_method` (`put` \| `patch`), `fields_changed` (string[] — attribute names only; **no values**) |
| `scim.group_synced` | STANDARD | 5 yr | `PATCH /scim/v2/Groups/:id` processed successfully | `tenant_id`, `group_id` (FORM `scim_group_role_mappings.id` UUID), `idp_group_id` (IdP-side opaque group ID), `members_added` (integer), `members_removed` (integer), `role_changes` (integer), `scim_request_id` |
| `scim.rejected_sensitive_attribute` | HIGH | 7 yr | SCIM request body contains GDPR Art. 9 special category attribute; request rejected before any write | `tenant_id`, `scim_request_id`, `rejected_attribute_name`, `request_path`, `http_method` |
| `scim.session_revocation_kv_fallback` | HIGH | 7 yr | `env.SCIM_KV.put()` throws during SCIM PATCH deprovisioning; Supabase blocklist fallback path engaged | `tenant_id`, `scim_request_id`, `fallback_path: 'supabase_blocklist'`, `supabase_ok` (bool — `false` if Supabase fallback also failed), `kv_error_class` (string max 64 — e.g. `'KVError'`, `'NetworkError'`) |

**Privacy invariant (`scim.session_revocation_kv_fallback`):** No `user_id`, no email address, no role values. `tenant_id` and `scim_request_id` are the minimum identifiers required for incident correlation. `supabase_ok = false` escalates to P0 (both revocation paths failed) per §28.6 IC response protocol.

**Emitter assignment:** `scim.user_provisioned`, `scim.user_reactivated`, `scim.user_deprovisioned`, `scim.user_updated`, `scim.group_synced`, `scim.rejected_sensitive_attribute`, `scim.session_revocation_kv_fallback` — `form_system` (SCIM endpoint Worker `apps/scim-worker/` via `AUDIT_QUEUE` Cloudflare Queue binding). No human emitter for any lifecycle event; all are automated.

**HMAC chain requirement (REVOKE-CHAIN-01):** `scim.session_revocation_kv_fallback` for a given `scim_request_id` must follow `scim.user_deprovisioned` for the same `scim_request_id` in chain order. A chain break violates CC7.2 evidence integrity and triggers R-05 (HMAC Chain Break). Enforced by `emit-audit-event` Worker: HTTP 422 `REVOKE_CHAIN_01_VIOLATION` if `scim.session_revocation_kv_fallback` is submitted without a preceding `scim.user_deprovisioned` for the same `scim_request_id`.

**Zod schemas** (to register in `apps/audit-worker/src/schemas/scim-lifecycle.ts`):

```typescript
import { z } from 'zod';

export const ScimUserProvisionedSchema = z.object({
  tenant_id: z.string().uuid(),
  user_id: z.string().uuid(),
  external_user_id: z.string().max(256),
  scim_request_id: z.string().uuid(),
  provisioning_source: z.literal('scim'),
  assigned_role: z.enum(['tenant_admin', 'tenant_manager', 'member']),
});

export const ScimUserReactivatedSchema = z.object({
  tenant_id: z.string().uuid(),
  user_id: z.string().uuid(),
  external_user_id: z.string().max(256),
  scim_request_id: z.string().uuid(),
  prior_deactivated_at: z.string().datetime(),
});

export const ScimUserDeprovisionedSchema = z.object({
  tenant_id: z.string().uuid(),
  user_id: z.string().uuid(),
  external_user_id: z.string().max(256),
  scim_request_id: z.string().uuid(),
  deprovision_method: z.enum(['scim_patch_inactive', 'scim_delete']),
  sessions_revoked_count: z.number().int().min(0),
});

export const ScimUserUpdatedSchema = z.object({
  tenant_id: z.string().uuid(),
  user_id: z.string().uuid(),
  scim_request_id: z.string().uuid(),
  update_method: z.enum(['put', 'patch']),
  fields_changed: z.array(z.string()).min(1),
});

export const ScimGroupSyncedSchema = z.object({
  tenant_id: z.string().uuid(),
  group_id: z.string().uuid(),
  idp_group_id: z.string().max(256),
  members_added: z.number().int().min(0),
  members_removed: z.number().int().min(0),
  role_changes: z.number().int().min(0),
  scim_request_id: z.string().uuid(),
});

export const ScimRejectedSensitiveAttributeSchema = z.object({
  tenant_id: z.string().uuid(),
  scim_request_id: z.string().uuid(),
  rejected_attribute_name: z.string().max(256),
  request_path: z.string().max(256),
  http_method: z.enum(['POST', 'PUT', 'PATCH']),
});

export const ScimSessionRevocationKvFallbackSchema = z.object({
  tenant_id:       z.string().uuid(),
  scim_request_id: z.string().max(128),
  fallback_path:   z.literal('supabase_blocklist'),
  supabase_ok:     z.boolean(),
  kv_error_class:  z.string().max(64),
});
```

**Retention table additions:**

| Event family | Retention | SOC 2 / regulatory basis |
|---|---|---|
| `scim.user_provisioned` / `scim.user_reactivated` / `scim.user_deprovisioned` / `scim.rejected_sensitive_attribute` | 7 years | SOC 2 CC6.1 (logical access evidence), CC6.3 (timely revocation), CC6.4 (attribute rejection enforcement); GDPR Art. 28 processor obligation evidence |
| `scim.user_updated` | 7 years | SOC 2 CC6.1; role change trail linked to OQ-SSO-27.2 |
| `scim.group_synced` | 5 years | SOC 2 CC6.6 (authorised role assignments via group mapping); lower retention than lifecycle events — no direct erasure or access obligation |
| `scim.session_revocation_kv_fallback` | 7 years | SOC 2 CC6.3 (timely access removal — dual-path session revocation evidence; `supabase_ok = true` confirms 60-second P99 SLA met per `ENTERPRISE_SLA.md §3.7`), CC7.2 (Cloudflare KV degradation detection — triggers AL-REVOKE-01 P1) |

---

### SCIM Mass Deprovisioning events (DEC-030 HMAC-chained · INCIDENT_RESPONSE R-24)

> Defined in `docs/INCIDENT_RESPONSE.md §R-24.9`. Five events covering the detection, containment, admin lockout recovery, and resolution of accidental or erroneous mass SCIM deprovisioning events from enterprise IdPs (Okta, Azure AD, Google Workspace). Classified as a data integrity and availability incident distinct from R-04 (SSO/SAML Compromise) and R-19 (IdP Outage). **Privacy invariant (all five events):** `tenant_id` and aggregate counts only — no `user_id`, no individual employee names, no coaching or health data. `tenant_owner_email_hash` is SHA-256 of the tenant owner email — raw email never enters the audit chain. **CRITICAL event** (`scim.mass_deprovision_detected`) triggers immediate PagerDuty P0/P1 dual-page to `form-customer-success` + `form-platform` before any other response action (DEC-030 ordering: chain entry precedes state mutation). Cross-ref: SOC 2 A1.1 (capacity monitoring — SCIM-E-001), A1.2 (environmental threat detection — SCIM-E-002 DEC-030 chain), CC7.2 (proactive monitoring — SCIM-E-003 AL-SCIM-MASS-01 PagerDuty log), CC7.3 (anomaly response — SCIM-E-004 communication SLA), CC9.2 (IdP as third-party risk — SCIM-E-005 pause mechanism independence); `docs/OBSERVABILITY.md §26.7a` (AL-SCIM-MASS-01 alert rule); `docs/OBSERVABILITY.md §12.6` (pg_cron job 24 `scim_mass_deprovision_check`). Closes INCIDENT_RESPONSE.md R-24.15 checklist item 3 (P0 M5).

| Event type | Severity | Retention | Trigger | Key payload fields |
|---|---|---|---|---|
| `scim.mass_deprovision_detected` | **CRITICAL** | 7 yr | AL-SCIM-MASS-01 pg_cron fires: `scim.user_deprovisioned` events for a single `tenant_id` exceed `MAX(0.10 × seat_count, 10)` in a 10-minute rolling window | `tenant_id`, `deprovisioned_count` (integer), `pct_of_seats` (float — e.g. `0.62` for 62%), `window_minutes: 10`, `incident_id` (UUID — IC assigns at channel open), `detection_source` (`alert` \| `csm_report` \| `monitoring`) |
| `scim.sync_suspended` | HIGH | 7 yr | IC or platform-engineer writes `scim:sync:paused:{tenant_id}` Cloudflare KV key via `POST /internal/v1/admin/scim/pause` | `tenant_id`, `suspended_by_admin_id` (UUID), `reason` (free text, max 200 chars), `auto_resume_at` (ISO 8601 or null — IC-estimated resolution window), `incident_id` |
| `scim.admin_lockout_recovery` | HIGH | 7 yr | `POST /internal/v1/admin/magic-link` issued by platform-engineer when the `tenant_owner` account is locked out after mass deprovisioning | `tenant_id`, `issued_by_admin_id` (UUID), `tenant_owner_email_hash` (SHA-256 of tenant owner email — **never raw email**), `incident_id`, `magic_link_expires_at` (ISO 8601 — 15-minute TTL) |
| `scim.sync_resumed` | HIGH | 7 yr | IC removes `scim:sync:paused:{tenant_id}` KV key after root cause is resolved and IdP configuration confirmed safe | `tenant_id`, `resumed_by_admin_id` (UUID), `root_cause_hypothesis` (`H1` \| `H2` \| `H3` \| `H4`), `incident_id`, `users_restored_count` (integer — users confirmed re-provisioned before resume) |
| `scim.mass_reprovision_complete` | STANDARD | 3 yr | Full recovery confirmed: all deprovisioned users re-provisioned by IdP re-push or FORM-side restoration SQL (R-24-R1) | `tenant_id`, `restored_count` (integer), `unrestorable_count` (integer — 0 expected; > 0 triggers P1 follow-up), `incident_id`, `recovery_method` (`idp_repush` \| `form_manual` \| `both`) |

**HMAC chain requirement (R24-CHAIN-01):** `scim.mass_deprovision_detected` must be the first event emitted for any `incident_id` — chain guard in `emit-audit-event` Worker returns HTTP 422 if `scim.sync_suspended` or `scim.sync_resumed` is submitted for an `incident_id` with no preceding `scim.mass_deprovision_detected`. `scim.sync_suspended` must precede `scim.sync_resumed` for the same `incident_id`. `scim.admin_lockout_recovery` and `scim.mass_reprovision_complete` may appear at any point after `scim.mass_deprovision_detected`. A chain break between any of these events triggers R-05 (HMAC Chain Break) and immediate escalation to security-engineer.

**Emitter assignment:** `scim.mass_deprovision_detected` — `form_system` (automated pg_cron `scim_mass_deprovision_check` every 5 min) or `customer-success` (manual admin console report path); `scim.sync_suspended` — `platform-engineer` or `customer-success` (manual via `POST /internal/v1/admin/scim/pause`); `scim.admin_lockout_recovery` — `platform-engineer` (manual via `POST /internal/v1/admin/magic-link`); `scim.sync_resumed` — IC (manual KV key removal after root cause confirmed); `scim.mass_reprovision_complete` — `platform-engineer` or `form_system` (automated check after restoration SQL completes, or manual confirmation by IC).

---

### SCIM IP Enforcement Misconfiguration events (DEC-030 HMAC-chained · SSO §32.3 · DEC-062)

> Defined in `docs/SSO_SCIM_IMPLEMENTATION.md §32.3` (v2.4, 2026-06-16 — DEC-062). One **advisory-only** STANDARD event covering the misconfiguration case where a tenant enables the `scim_ip_enforcement_enabled` flag but leaves `scim_ip_allowlist_config` empty. **Advisory, not blocking:** the `enforceScimIpAllowlist()` function emits this event and continues (no 403 thrown) — the intent is to give the compliance-officer a signal that the control is active but misconfigured, without disrupting provisioning. **Privacy floor:** `tenant_id` only — no `user_id`, no employee data, no SCIM payload content. `form_api` REVOKED from this event's emitter path. **No HMAC chain ordering invariant** for this event — it is standalone (not part of a required sequence). Cross-ref: `docs/SSO_SCIM_IMPLEMENTATION.md §32.3` (OQ-SSO-25.1 resolution — `scim_ip_enforcement_enabled` flag design); migration `0076_scim_ip_enforcement_flag.sql` (flag default `false`); SOC 2 CC6.1 (advisory artefact CC6-E-SCIM-IP-001). **Closes SSO_SCIM_IMPLEMENTATION.md §32.8 checklist item 3 (P0, before SCIM go-live — register `scim.ip_enforcement_misconfigured` in AUDIT_LOG_SCHEMA.md §SCIM namespace).**

| Event type | Severity | Retention | Trigger | Key payload fields |
|---|---|---|---|---|
| `scim.ip_enforcement_misconfigured` | STANDARD | 3 yr | `enforceScimIpAllowlist()` is called when `scim_ip_enforcement_enabled = true` but `scim_ip_allowlist_config` is null or empty; emitted before returning (non-blocking — no 403) | `tenant_id` (UUID), `enforcement_flag: true` (confirms flag is set), `allowlist_entry_count: 0` (confirms allowlist is empty — no CIDRs), `scim_request_id` (UUID, from incoming SCIM request — for operator correlation only) |

**Advisory semantics:** This event is a compliance-officer health signal, not a security enforcement event. Emitting it means: "a tenant opted in to SCIM IP enforcement but has no CIDRs configured — all SCIM traffic is currently passing through unchecked." The correct response is for the CSM to contact the tenant admin and guide them through the Admin Dashboard "Directory Sync IP Restriction" panel (SSO §32.3.3). There is no `auth_path` field on this event (it is always SCIM context by definition). No PagerDuty alert — compliance-officer weekly SIEM review only.

**Emitter assignment:** `scim.ip_enforcement_misconfigured` — `form_system` (SCIM Worker `apps/scim-worker/` via `AUDIT_QUEUE` Cloudflare Queue binding, on each SCIM request that passes through the misconfigured enforcement path). Expected low-frequency in production: a tenant with this flag enabled and an empty allowlist is a transient state (admin enabled flag before entering CIDRs). The event should self-extinguish once the tenant's CSM guides them to add CIDRs. More than 50 emissions for the same `tenant_id` without an intervening `scim.ip_enforcement_disabled` event should trigger a CSM alert (not a PagerDuty page).

**Retention table addition:**

| Event family | Retention | SOC 2 / regulatory basis |
|---|---|---|
| `scim.ip_enforcement_misconfigured` | 3 years | SOC 2 CC6.1 (advisory — confirms SCIM IP control is configured correctly once CIDRs are added; low retention because this is a transient misconfiguration signal, not a compliance obligation in itself) |

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

#### Webhook delivery events (DEC-030 HMAC-chained · OBSERVABILITY §43 · CC9.2/CC6.8)

> Defined in `docs/OBSERVABILITY.md §43`. Six DEC-030 HMAC-chained events covering the enterprise webhook delivery lifecycle — the tenant-controlled HTTP push endpoint where enterprise tenants receive real-time operational notifications. This section extends the three previously stubbed events (`integration.webhook_created`, `integration.webhook_deleted`, `integration.webhook_fired`) with full payload specs and registers three new HIGH-severity events. **Scope:** enterprise tenants only (Growth and Enterprise tier), managing webhook subscriptions via `docs/ENTERPRISE_ADMIN_API.md §9`. **Privacy floor (all six events):** `endpoint_url` is tenant-sensitive commercial information — it does not appear in any DEC-030 chain payload; only `endpoint_url_hash` = SHA-256(`endpoint_url`) is stored in chain records, consistent with the DEC-054 `custom_domain_hash` pattern (`docs/OBSERVABILITY.md §42.13`). No GDPR Art. 9 health data in webhook payloads — subscription whitelist enforced in `audit-event-dispatcher`: only audit lifecycle, SCIM user lifecycle, GDPR/DSAR status, and SLA breach event types are eligible for delivery. `tenant_manager` (HR role) has no access to `tenant_webhooks` or `webhook_delivery_log`. `form_api` REVOKED from both management-plane tables. `integration.webhook_*` SIEM events routed to `compliance_reviewer` only — no cross-tenant SIEM delivery. Cross-ref: `docs/OBSERVABILITY.md §43.4` (WH-SLO-01 through WH-SLO-04); `docs/ENTERPRISE_ADMIN_API.md §9` (webhook API spec — create, list, delete, HMAC signature verification); `docs/ENTERPRISE_ADMIN_API.md §12` (DEC-030 event reference — payload extended per this section); `docs/OBSERVABILITY.md §23` (SLA credit engine — WH-SLO-01 breach triggers credit calculation for Growth/Enterprise tier). **Closes OBSERVABILITY §43.15 checklist item 1 (P0, M10 — register all six DEC-030 events in AUDIT_LOG_SCHEMA.md §Integration before first webhook-enabled enterprise tenant goes live).**

**WH-CHAIN-01 ordering invariant:** For each `webhook_id`, events must follow: `integration.webhook_created` → `integration.webhook_fired`^N → `integration.webhook_delivery_failed` → `integration.webhook_suspended` → `integration.webhook_reactivated` → `integration.webhook_fired`^N (cycle restarts after reactivation) → `integration.webhook_deleted`. A `webhook_fired` event appearing after `webhook_suspended` without a preceding `integration.webhook_reactivated` for the same `webhook_id` constitutes a WH-CHAIN-01 violation — the `emit-audit-event` Worker returns HTTP 422 (`WH_CHAIN_01_VIOLATION`) and triggers R-05 (Security Incident Response, `docs/INCIDENT_RESPONSE.md R-05`). `webhook_fired` before `webhook_created` for the same `webhook_id` is a blocking violation (R-05). `webhook_deleted` after `webhook_suspended` without `webhook_reactivated` is a permitted transition (tenant deletes a suspended webhook without re-activating). The weekly HMAC chain audit additionally flags any WH-CHAIN-01 violation discovered in batch as a HIGH anomaly requiring `compliance-officer` + `security-engineer` review within 48 hours.

| Event type | Severity | Retention | Trigger | Key payload fields |
|---|---|---|---|---|
| `integration.webhook_created` | STANDARD | 7 yr | Tenant creates a new webhook subscription via `POST /enterprise/webhooks`; `tenant_webhooks` row inserted with `status = 'active'` | `webhook_id` (UUID), `endpoint_url_hash` (SHA-256 of plaintext URL — URL never in chain), `events` (string[] of subscribed event-type prefixes ≥ 1 element), `created_by` (PAM session UUID for `form_system` operations; `tenant_owner` user_id for self-service) |
| `integration.webhook_deleted` | STANDARD | 7 yr | Tenant or `form_system` deletes a webhook via `DELETE /enterprise/webhooks/{id}`; `tenant_webhooks` row hard-deleted | `webhook_id` (UUID), `endpoint_url_hash` (SHA-256), `deleted_by` (enum: `tenant_owner` \| `tenant_admin` \| `form_system`) |
| `integration.webhook_fired` | STANDARD | 7 yr | `webhook-dispatcher` Cloudflare Worker records each delivery attempt — all five retry steps each produce one event | `webhook_id` (UUID), `tenant_id` (UUID), `event_type` (string — subscribed event delivered), `attempt_number` (int 1–5), `outcome` (enum: `success` \| `failure` \| `pending_retry`), `http_status` (nullable int — NULL on timeout / TLS / DNS failure), `error_class` (nullable enum: `timeout` \| `4xx` \| `5xx` \| `tls_error` \| `dns_error` \| `internal_error`), `latency_ms` (nullable int — NULL on hard timeout > 30 000 ms cut-off), `endpoint_url_hash` (SHA-256); **no outgoing request body ever persisted** |
| `integration.webhook_delivery_failed` | **HIGH** | 7 yr | 5th consecutive failed delivery attempt; `tenant_webhooks.status` transitions `active` → `degraded`; triggers AL-WH-02 (PagerDuty `form-platform` + `form-customer-success` dual-page, no auto-resolve); WH-SLO-04 tenant-notification clock starts | `webhook_id` (UUID), `tenant_id` (UUID), `endpoint_url_hash` (SHA-256), `consecutive_failures: 5` (literal int — always 5 at the `degraded` transition), `error_class` (enum — class of the 5th failed attempt), `degraded_since` (ISO 8601 timestamp of this emission) |
| `integration.webhook_suspended` | **HIGH** | 7 yr | 48 h elapsed in `degraded` state; `tenant_webhooks.status` auto-set to `suspended` by pg_cron job 34 (`webhook_degraded_escalation_check`, `*/30 * * * *`); triggers AL-WH-04 (PagerDuty CRITICAL: `form-platform` + `form-customer-success` dual-page + CSM email WH-NOTIF-02); WH-SLO-04 SLA breach confirmed | `webhook_id` (UUID), `tenant_id` (UUID), `endpoint_url_hash` (SHA-256), `degraded_since` (ISO 8601 — timestamp from the originating `integration.webhook_delivery_failed` event), `total_failed_attempts_in_window` (int ≥ 5 — total delivery attempts recorded since `degraded_since`) |
| `integration.webhook_reactivated` | **HIGH** | 7 yr | `tenant_owner` re-activates a suspended webhook via Admin Dashboard "Re-activate" button; `tenant_webhooks.status` transitions `suspended` → `active`; `consecutive_failures` reset to 0 | `webhook_id` (UUID), `tenant_id` (UUID), `endpoint_url_hash` (SHA-256), `reactivated_by: 'tenant_owner'` (literal — only `tenant_owner` RBAC role can invoke re-activation), `previous_status: 'suspended'` (literal) |

**HMAC chain requirement:** `integration.webhook_delivery_failed` and `integration.webhook_suspended` are HIGH severity and emitted synchronously — failure aborts the state transition in the `webhook-dispatcher` Worker and pg_cron job 34 respectively. `integration.webhook_reactivated` is emitted synchronously inside the Workers middleware re-activation endpoint — failure blocks the re-activation HTTP response (fail-closed). Any WH-CHAIN-01 violation is treated as a security incident per R-05; no grace period.

**Emitter assignments:** `integration.webhook_created` + `integration.webhook_deleted` — Workers API middleware (`form_system`, on behalf of authenticated `tenant_owner` or `tenant_admin` request). `integration.webhook_fired` — `webhook-dispatcher` Cloudflare Worker (`form_system`, automated, per-attempt). `integration.webhook_delivery_failed` — `webhook-dispatcher` Worker on 5th consecutive failure (`form_system`, automated). `integration.webhook_suspended` — pg_cron job 34 `webhook_degraded_escalation_check` at 48 h boundary (`form_system`, automated). `integration.webhook_reactivated` — Workers API middleware on `tenant_owner` re-activate request.

**Zod v2 schemas (canonical source for `emit-audit-event` Worker validation):**

```typescript
const IntegrationWebhookCreatedSchema = z.object({
  webhook_id:        z.string().uuid(),
  endpoint_url_hash: z.string().regex(/^[a-f0-9]{64}$/),  // SHA-256(endpoint_url) — URL never in chain
  events:            z.array(z.string().min(1)).min(1),
  created_by:        z.string(),  // PAM session UUID (form_system ops) or tenant_owner user_id (self-service)
});

const IntegrationWebhookDeletedSchema = z.object({
  webhook_id:        z.string().uuid(),
  endpoint_url_hash: z.string().regex(/^[a-f0-9]{64}$/),
  deleted_by:        z.enum(['tenant_owner', 'tenant_admin', 'form_system']),
});

const IntegrationWebhookFiredSchema = z.object({
  webhook_id:        z.string().uuid(),
  tenant_id:         z.string().uuid(),
  event_type:        z.string().min(1),
  attempt_number:    z.number().int().min(1).max(5),
  outcome:           z.enum(['success', 'failure', 'pending_retry']),
  http_status:       z.number().int().nullable(),
  error_class:       z.enum([
    'timeout', '4xx', '5xx', 'tls_error', 'dns_error', 'internal_error',
  ]).nullable(),
  latency_ms:        z.number().int().nonnegative().nullable(),
  endpoint_url_hash: z.string().regex(/^[a-f0-9]{64}$/),
});

const IntegrationWebhookDeliveryFailedSchema = z.object({
  webhook_id:           z.string().uuid(),
  tenant_id:            z.string().uuid(),
  endpoint_url_hash:    z.string().regex(/^[a-f0-9]{64}$/),
  consecutive_failures: z.literal(5),  // always 5 at degraded transition
  error_class:          z.enum([
    'timeout', '4xx', '5xx', 'tls_error', 'dns_error', 'internal_error',
  ]),
  degraded_since: z.string().datetime(),
});

const IntegrationWebhookSuspendedSchema = z.object({
  webhook_id:                      z.string().uuid(),
  tenant_id:                       z.string().uuid(),
  endpoint_url_hash:               z.string().regex(/^[a-f0-9]{64}$/),
  degraded_since:                  z.string().datetime(),
  total_failed_attempts_in_window: z.number().int().min(5),
});

const IntegrationWebhookReactivatedSchema = z.object({
  webhook_id:        z.string().uuid(),
  tenant_id:         z.string().uuid(),
  endpoint_url_hash: z.string().regex(/^[a-f0-9]{64}$/),
  reactivated_by:    z.literal('tenant_owner'),
  previous_status:   z.literal('suspended'),
});
```

**SOC 2 evidence artefacts** (store at `compliance/evidence/cc9/webhooks/` with `MANIFEST.sha256`):

| Artefact | Description | SOC 2 Criteria | Retention |
|---|---|---|---|
| **WH-E-001** | Quarterly export of `integration.webhook_created`, `integration.webhook_delivery_failed`, `integration.webhook_suspended`, `integration.webhook_reactivated`, `integration.webhook_deleted` DEC-030 chain events — demonstrates controlled third-party delivery lifecycle with no silent drops (WH-SLO-02 evidence) | CC9.2 — Third-party commitment monitoring; CC6.8 — Transmission integrity (HMAC chain proves no events were silently dropped) | 7 yr |
| **WH-E-002** | AL-WH-02 + AL-WH-04 PagerDuty incident history (90 days) + Resend delivery receipts for WH-NOTIF-01 / WH-NOTIF-02 — demonstrates automated detection and timely CSM + tenant notification for degraded and suspended webhooks | CC7.2 — Proactive anomaly detection; CC7.3 — Response to anomalies within defined escalation SLA | 3 yr |
| **WH-E-003** | Annual `webhook-dispatcher` Worker source code review confirmation — verifies (a) HMAC-SHA256 computed over raw request body for every delivery; (b) `endpoint_url` never appears in any log, analytics signal, or DEC-030 payload; (c) `WEBHOOK_SIGNING_SECRET_{webhook_id}` sourced from Cloudflare Worker Secrets only, never from Postgres | CC6.8 — Transmission integrity; CC6.1 — Access controls over tenant-sensitive endpoint URLs | 3 yr |

**Auditor narrative for CC9.2:** FORM's enterprise webhook delivery pipeline enables tenant SIEM systems to receive real-time DEC-030 event notifications. WH-E-001 provides quarterly chain-extracted proof that every webhook lifecycle event is HMAC-evidenced and that failed delivery cascades surface as `integration.webhook_delivery_failed` (HIGH, 7-year retention) rather than being silently discarded. The WH-CHAIN-01 ordering invariant ensures that a `webhook_fired` event cannot appear after `webhook_suspended` without a reactivation event — giving auditors tamper-evident evidence that the delivery pipeline operates under controlled state transitions. No health data flows through this pipeline — webhook payloads are restricted to operational metadata categories enumerated in `docs/OBSERVABILITY.md §43.12`.

---

#### Notification connector events (DEC-030 HMAC-chained · CC9.2)

> Covers the lifecycle of team-notification connectors (Slack incoming webhook, MS Teams connector, generic HTTP channel) that `tenant_admin` or `tenant_owner` activates to receive FORM wellness-signal digests in their communication platform. **Scope:** Growth and Enterprise tier only — connectors surface only opt-in aggregate signals; no individual employee data ever flows through them. **Privacy floor (both events):** `endpoint_url` is tenant-sensitive commercial information — only `endpoint_url_hash` = SHA-256(`endpoint_url`) appears in DEC-030 chain payloads, per the DEC-054 hash-only pattern (`docs/OBSERVABILITY.md §42.13`). No GDPR Art. 9 health data in payloads. `tenant_manager` (HR role) has no access to connector configuration; `form_api` REVOKED from the `tenant_connectors` table. Cross-ref: `docs/ENTERPRISE_ADMIN_API.md §10` (connector API spec); `docs/DATA_MODEL.md §27` (tenant_connectors table); DEC-030; DEC-054.

| Event type | Severity | Retention | Trigger | Key payload fields |
|---|---|---|---|---|
| `integration.connector_added` | STANDARD | 7 yr | `tenant_admin` or `tenant_owner` activates a notification connector via Admin Dashboard → Integrations → Add connector; `tenant_connectors` row inserted with `status = 'active'` | `connector_id` (UUID), `connector_type` (enum: `slack` \| `ms_teams` \| `generic_http`), `tenant_id` (UUID), `activated_by` (user UUID of `tenant_admin` or `tenant_owner` who submitted the form), `endpoint_url_hash` (SHA-256 of incoming webhook URL — URL never in chain) |
| `integration.connector_removed` | STANDARD | 7 yr | `tenant_admin` or `tenant_owner` removes a connector via Admin Dashboard; `tenant_connectors` row hard-deleted | `connector_id` (UUID), `connector_type` (enum: `slack` \| `ms_teams` \| `generic_http`), `tenant_id` (UUID), `removed_by` (UUID), `endpoint_url_hash` (SHA-256) |

**HMAC chain requirement:** Both events are STANDARD severity and emitted synchronously inside the Workers API middleware that handles connector create/delete. Failure aborts the HTTP response (fail-closed). `integration.connector_added` must precede `integration.connector_removed` for the same `connector_id` — the `emit-audit-event` Worker returns HTTP 422 on a `connector_removed` event with no prior `connector_added` for the same UUID.

**Emitter assignments:** Both events — Workers API middleware (`form_system`, on behalf of the authenticated `tenant_admin` or `tenant_owner` request). No pg_cron involvement; connectors do not have an auto-suspend lifecycle (unlike webhooks).

**Zod v2 schemas (canonical source for `emit-audit-event` Worker validation):**

```typescript
const ConnectorTypeSchema = z.enum(['slack', 'ms_teams', 'generic_http']);

const IntegrationConnectorAddedSchema = z.object({
  connector_id:       z.string().uuid(),
  connector_type:     ConnectorTypeSchema,
  tenant_id:          z.string().uuid(),
  activated_by:       z.string().uuid(),  // tenant_admin or tenant_owner user UUID
  endpoint_url_hash:  z.string().regex(/^[a-f0-9]{64}$/),  // SHA-256(endpoint_url) — URL never in chain
});

const IntegrationConnectorRemovedSchema = z.object({
  connector_id:       z.string().uuid(),
  connector_type:     ConnectorTypeSchema,
  tenant_id:          z.string().uuid(),
  removed_by:         z.string().uuid(),  // tenant_admin or tenant_owner user UUID
  endpoint_url_hash:  z.string().regex(/^[a-f0-9]{64}$/),
});
```

**SOC 2 mapping:** CC9.2 — monitoring of third-party notification channel lifecycle. `integration.connector_added` + `integration.connector_removed` provide a chain-evidenced record of every external notification endpoint activated in a tenant's account, satisfying auditor requests for proof that team-notification integrations are controlled and logged.

---

#### API call sampling events (volume management · not HMAC-chained)

> `integration.api_call` is a **sampled** operational telemetry record — 1-in-10 sampling rate applied at the `api-gateway` Cloudflare Worker before the record is written. Because the event set is intentionally incomplete by design (sampling), it is **not HMAC-chained** and is not an evidence-class event. It serves volume management and p95 latency baselining only. **Retention: 30 days** (partition-drop; not archived to R2). **Privacy floor:** no `user_id` in sampled records — sampling applied at the session level prevents reconstruction of individual activity patterns from the sampled set; `tenant_id` is included (quota monitoring, not individual attribution). No GDPR Art. 9 health data. Cross-ref: `docs/OBSERVABILITY.md §Retention table` (30-day partition drop); `docs/DATA_MODEL.md §28` (api_quota_usage — the authoritative quota ledger; this event is a monitoring supplement, not the ledger itself).

| Event type | Severity | Retention | Trigger | Key payload fields |
|---|---|---|---|---|
| `integration.api_call` | STANDARD | 30 days | `api-gateway` Worker samples 1 in 10 inbound API requests and writes a record; sampling decision is per-request random (not per-tenant or per-user) | `tenant_id` (UUID), `endpoint_category` (enum: `coaching` \| `cv` \| `analytics` \| `admin` \| `enterprise`), `method` (enum: `GET` \| `POST` \| `PATCH` \| `DELETE`), `http_status` (int 100–599), `latency_ms` (int ≥ 0), `sampled` (literal `true`), `sample_rate` (literal `0.1`) |

**Not HMAC-chained:** Sampled records are inherently incomplete; including them in the HMAC chain would produce unprovable gaps (a verifier cannot distinguish a legitimate 9-in-10 absence from a tampered deletion). The chain covers only events where completeness can be guaranteed.

**Zod v2 schema:**

```typescript
const IntegrationApiCallSchema = z.object({
  tenant_id:         z.string().uuid(),
  endpoint_category: z.enum(['coaching', 'cv', 'analytics', 'admin', 'enterprise']),
  method:            z.enum(['GET', 'POST', 'PATCH', 'DELETE']),
  http_status:       z.number().int().min(100).max(599),
  latency_ms:        z.number().int().nonnegative(),
  sampled:           z.literal(true),    // always true — unsampled calls are not written
  sample_rate:       z.literal(0.1),     // documents sampling policy at record level
});
```

**Operational note:** p95 latency derived from `integration.api_call` is used as a leading indicator in `docs/OBSERVABILITY.md §6.2` alert AL-API-02. The 30-day partition-drop retention means this event set **cannot** be used for SOC 2 evidence or GDPR Art. 30 records — use `security.quota_95pct_warning` (STANDARD, 3yr, HMAC-chained) and `admin.access_review_completed` (STANDARD, 7yr, HMAC-chained) for compliance purposes.

### System
- `system.backup_completed` / `system.backup_restored` — full Zod schema + retention in **§ Backup & DR Observability events** below
- `system.maintenance_started` / `system.maintenance_completed`
- `system.config_changed` (feature flag, environment var)
- `system.deployment_completed` (release SHA, environment) — full Zod schema + retention in **§ CI/CD Pipeline events** below
- `system.access_review_completed` — quarterly access review completed (SOC 2 CC6.2/CC6.3/CC6.5/CC4.2); payload: `{reviewer_id, quarter, artifact_sha256, systems_reviewed_count, accounts_reviewed_count, findings_count, review_latency_days}`; STANDARD severity, 7yr retention; HMAC-chained; cross-ref: SOC2_READINESS §23, §65
- `system.credential_rotated` — FORM-internal credential rotated (scheduled or triggered); payload: `{credential_name, rotation_trigger: 'scheduled'|'compromise'|'quarterly_review'|'key_ceremony', days_since_last_rotation, rotated_by}`; STANDARD severity, 7yr retention; HMAC-chained; cross-ref: SOC2_READINESS §56/§57, §65.9
- `system.cron_job_stale` — `pg-cron-health-monitor` detected that a monitored pg_cron job exceeded its freshness window; payload: `{job_name, schedule, last_successful_run, hours_since_last_run, freshness_window_hours}`; HIGH severity, 7yr retention; HMAC-chained; emits PagerDuty P1 for most jobs (P0 for `audit-chain-daily-check`, `row-count-monitor`, `audit-event-flush` — see OBSERVABILITY §12.6 job registry); cross-ref: SOC2_READINESS §71.2.2, OBSERVABILITY §12.6, DEC-030
- `system.cron_health_check_passed` — `pg-cron-health-monitor` completed its hourly check with all 9 monitored jobs within their freshness windows; payload: `{jobs_checked, all_healthy: true}`; LOW severity, 3yr retention; HMAC-chained; no alert (steady-state confirmation event); cross-ref: SOC2_READINESS §71.2.2, DEC-030
- `system.cron_health_check_failed` — `pg-cron-health-monitor` Edge Function itself errored (DB unreachable, `cron.job_run_details` query failure, or unhandled exception); payload: `{error_message, jobs_checked}`; HIGH severity, 7yr retention; HMAC-chained; PagerDuty P0 → devops-lead + platform-engineer; cross-ref: SOC2_READINESS §71.2.2, DEC-030
- `system.compliance_tool_connected` — Continuous compliance tooling (Vanta) connected to production integrations after DPA execution; emitted by `compliance-officer` (human, not `form_system`) once per activation; **STANDARD severity, 3yr retention**; HMAC-chained; payload: `{tool_name: 'vanta'|'drata', integrations_active: string[], dpa_executed_date: 'YYYY-MM-DD', dpa_file_ref: string, vanta_soc2_report_date?: 'YYYY-MM-DD', activated_by: uuid}`; **privacy invariant:** no user_id, no tenant_id, no health data — tool name and integration list only; DPA must be executed before this event is emitted (DPA-first discipline); Zod v2 schema in SOC2_READINESS §78.7; filing path: `compliance/evidence/ctool/activation-event.json` (CTOOL-E-006); cross-ref: SOC2_READINESS §78, docs/VENDOR_REGISTRY.md (Vanta High-tier vendor), DEC-030
- `system.siem_alert_activated` — AL-SIEM-06 dead-man's switch transitions from shadow mode (P3 Slack-only) to full production activation (P1 PagerDuty) per the graduated-activation policy (DEC-056); emitter: `devops-lead` (manual, via Admin Console → `POST /internal/siem/alerts/:rule_id/activate`) at the shadow→full transition decision point; **STANDARD severity, 3yr retention**; HMAC-chained; payload: `{rule_id: z.literal('AL-SIEM-06'), activation_mode: z.enum(['shadow', 'full']), shadow_period_start: z.string().datetime(), shadow_period_end: z.string().datetime(), auth_event_avg_per_30min: z.number().nonneg(), activated_by: z.string().uuid()}`; **privacy invariant:** no user_id, no tenant_id, no health data — operational calibration metadata only; evidence artefact CALIB-E-001 (SOC 2 CC4.1 — monitoring controls calibrated before full activation); filing path: `compliance/evidence/cc7/siem-calibration/siem-alert-activated-<iso-date>.json`; cross-ref: SOC2_READINESS §83.2, SOC2_READINESS §83.5, OBSERVABILITY §44.2, DEC-056, DEC-030
- `system.siem_calibration_verified` — CI post-step hook confirms per-tenant isolation of CR-01 brute-force aggregation rule; emitted automatically when `cr01-per-tenant-isolation.test.ts` passes in the PR pipeline; emitter: `form_system` (CI post-step hook, automated); **STANDARD severity, 3yr retention**; HMAC-chained; payload: `{correlation_rule: z.literal('CR-01'), test_id: z.string().min(1), test_result: z.literal('passed'), tenant_ids_used: z.array(z.string().regex(/^lt-/)).min(2) /* synthetic lt- tenants only */}`; **privacy invariant:** no real tenant_id — synthetic `lt-` tenant identifiers only; no user_id, no health data; evidence artefact CALIB-E-002 (SOC 2 CC7.2 — per-tenant threat monitoring isolation verified); filing path: `compliance/evidence/cc7/siem-calibration/cr01-calibration-<ci-run-id>.json`; cross-ref: SOC2_READINESS §83.3, SOC2_READINESS §83.5, OBSERVABILITY §44.3, DEC-057, DEC-030
- `system.quarterly_perf_reference_initiated` — k6 Cloud EU-West quarterly PERF-SLO-06 reference run started; emitter: `load-test-quarterly` GitHub Actions workflow (automated), immediately after k6 Cloud run is dispatched via the k6 Cloud API; **STANDARD severity, 3yr retention**; HMAC-chained; payload: `{run_type: z.literal('k6-cloud-quarterly'), cloud_run_id: z.string().uuid(), node_region: z.literal('eu-west'), test_suite: z.literal('perf-slo-06-reference'), staging_refresh_date: z.string().regex(/^\d{4}-\d{2}-\d{2}$/) /* date only — ANON-02 confirms no health data in staging */, clinical_safety_sign_off: z.literal(true), quarter: z.string().regex(/^\d{4}-Q[1-4]$/)}`; **privacy invariant:** no `user_id`, no real `tenant_id`, no health data; `cloud_run_id` is a k6-Cloud-assigned UUID with no FORM user or tenant mapping; `staging_refresh_date` is date only; `clinical_safety_sign_off: z.literal(true)` attests ANON-01–ANON-05 were verified before dispatch (HTTP 422 if false or absent); **PERF-CLOUD-CHAIN-01 ordering invariant:** this event must precede `system.quarterly_perf_reference_completed` for the same `cloud_run_id`; inversion (completed without prior initiated) → P1 PagerDuty `form-platform` (not R-05 — compliance-signal ordering, not audit-chain tamper); `emit-audit-event` Worker returns HTTP 422 on completed without prior initiated; SOC 2 A1.1 evidence chain anchor (LT-E-001 chain opened at dispatch, closed on completed); cross-ref: `docs/OBSERVABILITY.md §45.4` (canonical event spec and Zod schema), `docs/OBSERVABILITY.md §45.8` (P0 M6 checklist item 2 — closed by this registration), DEC-058, DEC-030
- `system.quarterly_perf_reference_completed` — k6 Cloud EU-West quarterly PERF-SLO-06 reference run completed; emitter: `load-test-quarterly` GitHub Actions workflow (automated), after k6 Cloud API polling confirms completion; **STANDARD severity, 3yr retention**; HMAC-chained; payload: `{cloud_run_id: z.string().uuid(), node_region: z.literal('eu-west'), slo_results: z.object({'PERF-SLO-01': z.object({threshold: z.number(), achieved: z.number(), passed: z.boolean()}), 'PERF-SLO-02': z.object({threshold: z.number(), achieved: z.number(), passed: z.boolean()}), 'PERF-SLO-03': z.object({threshold: z.number(), achieved: z.number(), passed: z.boolean()}), 'PERF-SLO-04': z.object({threshold: z.number(), achieved: z.number(), passed: z.boolean()}), 'PERF-SLO-05': z.object({threshold: z.number(), achieved: z.number(), passed: z.boolean()})}), p95_ms: z.number().nonneg(), p99_ms: z.number().nonneg(), error_rate_pct: z.number().min(0).max(100), commit_sha: z.string().length(40), quarter: z.string().regex(/^\d{4}-Q[1-4]$/), run_duration_s: z.number().positive()}`; **privacy invariant:** no `user_id`, no real `tenant_id`, no health data; `slo_results` contain aggregate performance metrics for synthetic `lt-synthetic-{N}` tenants only; `cloud_run_id` is k6-Cloud-assigned UUID with no FORM user or tenant mapping; SOC 2 A1.1 primary evidence artefact: `cloud_run_id` provides independent k6 Cloud portal corroboration for LT-E-001 (`compliance/evidence/a1/lt-e-001-YYYY-QN.csv`); **PERF-CLOUD-CHAIN-01:** must be preceded by `system.quarterly_perf_reference_initiated` for the same `cloud_run_id`; `emit-audit-event` Worker returns HTTP 422 on ordering violation; inversion → P1 PagerDuty `form-platform`; cross-ref: `docs/OBSERVABILITY.md §45.4` (canonical event spec and Zod schema), `docs/OBSERVABILITY.md §45.6` (LT-E-001 updated scope — k6 Cloud EU-West source), `docs/OBSERVABILITY.md §45.8` (P0 M6 checklist item 2 — closed by this registration), DEC-058, DEC-030

#### Evidence collection automation events (SOC2_READINESS.md §81)

> Emitted by the Cloudflare Cron Worker (§81.4–§81.6) and pg_cron job 33 (`evidence_cron_freshness_check`, OBSERVABILITY.md §12.6). **Privacy floor (all three events):** `actor_id = NULL`, `actor_type = 'system'`, `tenant_id = '00000000-0000-0000-0000-000000000000'` (platform-level sentinel — no individual tenant). HR never sees these events; `compliance_reviewer` role reads all; `form_api` REVOKED from these rows. DEC-030 HMAC-chained.

- `system.evidence_collection_automated` — Cloudflare Cron Worker completed a successful monthly evidence collection run (01:00 UTC, 1st of month); **STANDARD severity, 7yr retention**; SOC 2 evidence artefact AUTO-E-001 (CC4.2/CC7.1); **EVD-CHAIN-01 ordering invariant:** this event must be preceded in the audit chain by `system.evidence_vault_configured` for the same calendar year — `emit-audit-event` Worker returns HTTP 422 on violation; payload: `{period: /^\d{4}-\d{2}$/, artifacts_written: string[] (each must start with 'compliance/evidence/'), chain_integrity_ok: true (literal — Worker aborts if false), slo_all_met: boolean, master_index_hash: string (SHA-256 hex, 64 chars), run_duration_ms: number (positive int)}`; cross-ref: SOC2_READINESS §81.5, SOC2_READINESS §81.7, DEC-030

- `system.evidence_cron_stale` — pg_cron job 33 (`evidence_cron_freshness_check`, 03:05 UTC 2nd of month) detected no `system.evidence_collection_automated` event for the prior period within 48 h; **HIGH severity, 3yr retention**; dead-man's switch signal that triggers AL-EVD-01 (P2 Slack `#compliance-alerts`); payload: `{period: string (YYYY-MM — the period for which evidence was expected), checked_at: string (ISO-8601), window_hours: 48 (literal)}`; cross-ref: SOC2_READINESS §81.9 AL-EVD-01, OBSERVABILITY §12.6 job 33, DEC-030

- `system.evidence_cron_conflict` — Cloudflare Cron Worker detected R2 `If-Match` ETag conflict during MASTER-INDEX reconciliation; Worker retries once; if second attempt also fails, this event is emitted and the run is aborted (no `system.evidence_collection_automated` is emitted for that run); **MEDIUM severity, 1yr retention**; triggers AL-EVD-04 (P2 Slack); payload: `{period: string (YYYY-MM), r2_key: string (MASTER-INDEX path), attempt_number: 1 | 2}`; cross-ref: SOC2_READINESS §81.6, DEC-030

### Admin (key management)

> HIGH severity · 7yr retention · HMAC-chained. Triggers PagerDuty P2 alert on unexpected rotation (outside scheduled window). Emitted synchronously inside KMS transaction — failure aborts rotation.

- `admin.encryption_key_rotated` — master encryption key rotated in KMS; payload: `{key_id, key_version_old, key_version_new, rotation_trigger: 'scheduled'|'incident'|'compromise', rotated_by, kms_provider}`; HIGH severity, 7yr retention; cross-ref: SOC2_READINESS §56, OBSERVABILITY §30.10 item 10, DEC-030
- `admin.signing_key_rotated` — HMAC chain signing key rotated; payload: `{key_id, rotation_trigger, rotated_by}`; HIGH severity, 7yr retention; DEC-030 chain continuity preserved via `key_transition` metadata
- `admin.key_rotation_initiated` — PAM session opened and both engineers present; **CRITICAL severity, 7yr retention**; MUST be the first chain event in any rotation — `emit-audit-event` Worker rejects subsequent rotation events if no matching `key_rotation_initiated` for the same `key_name` within 24h; payload: `{key_name, rotation_type: 'scheduled'|'emergency'|'infrastructure_repair', pam_session_id, second_engineer_user_id, reason, pre_rotation_hmac_chain_status: 'intact', estimated_window_minutes}`; cross-ref: SOC2_READINESS §57.4.1
- `admin.key_rotation_announced` — Dev-env rotation notice sent to #engineering Slack; **HIGH severity, 7yr retention**; emitter: `security-engineer` (manual); payload: `{key_name, announcement_channel: 'slack:#engineering', local_env_rotation_deadline_iso}`; cross-ref: SOC2_READINESS §57.4.2
- `admin.emergency_key_rotation` — Unplanned P0 rotation triggered by compromise signal; **CRITICAL severity, 7yr retention**; triggers GDPR Art. 33 72h clock via `gdpr_art33_clock_started_iso` field; payload: `{key_name, trigger: 'trufflehog_detection'|'log_leak'|'anomalous_query'|'third_party_report'|'pitr_post_rotation', incident_id, pam_session_id, second_person_user_id, gdpr_art33_clock_started_iso, gdpr_art33_deadline_iso, scope_assessment_status: 'in_progress'}`; **privacy invariant: JWT value never stored — `iat` Unix timestamp only**; cross-ref: SOC2_READINESS §57.5.2, INCIDENT_RESPONSE R-16
- `admin.key_rotation_reminder` — Automated cadence alert (14d, 7d, 3d before scheduled rotation); **LOW severity, 3yr retention**; emitter: `workers/key-rotation-monitor` Cron (daily 09:00 UTC per KEY_ROTATION_KV); payload: `{key_name, days_until_expiry, next_rotation_due_iso, rotation_type: 'scheduled'}`; no PagerDuty — Slack `#security-ops` only; cross-ref: SOC2_READINESS §57.7, OBSERVABILITY §30 AL-KEY-01
- `admin.key_rotation_overdue` — Scheduled rotation window missed; **HIGH severity, 7yr retention**; fires when `days_until_expiry ≤ 0`; triggers AL-KEY-02 (P0 PagerDuty, no auto-resolve); payload: `{key_name, days_overdue, last_rotation_iso, rotation_period_days}`; cross-ref: SOC2_READINESS §57.7, OBSERVABILITY §30 AL-KEY-02

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

### Enterprise partner channel events (DEC-030 HMAC-chained · COST_MODEL §38.8)

> Defined in `docs/COST_MODEL.md §38.8`. Four events creating an immutable governance record of partner agreement execution, revenue share payments, deal attribution, and partner offboarding. Required for COST_MODEL §38.10 checklist item 1 (**P0, M10 — before first partner agreement is signed**). **Privacy invariant (all four events):** no individual employee `user_id`, no health values, no coaching content; no partner contact names or company names in chain payload; `deal_id` is a FORM-internal UUID never shared externally; `partner_id` (UUID FK to `enterprise_partners.id`) is the sole cross-reference identifier across all four events. Revenue share percentages and ACV values are aggregate financial metadata only. Cross-ref: SOC 2 CC9.2 (vendor and partner agreements in place — PART-E-001/002/003 evidence artefacts); CC4.1 (control activities over financial obligations — `privacy_floor_check_passed: true` payment gate in `enterprise.partner_revenue_share_paid`); CC7.4 (incident response activated on partner privacy breach — PART-CHAIN-01 prerequisite); `docs/ENTERPRISE.md` (partner no-go criteria and privacy floor); `docs/INCIDENT_RESPONSE.md R-22` (privacy floor breach — PART-CHAIN-01 predecessor requirement); `docs/COST_MODEL.md §38.6` (partner governance — Bronze/Silver/Gold tiers; quarterly revenue share gate); `docs/COST_MODEL.md §38.9` (SOC 2 evidence artefacts PART-E-001 through PART-E-003). Closes COST_MODEL.md §38.10 checklist item 1 (P0, M10).

| Event type | Severity | Retention | Trigger | Key payload fields |
|---|---|---|---|---|
| `enterprise.partner_agreement_signed` | STANDARD | 7 yr | Partner agreement executed; `enterprise_partners` row created with `status = 'active'`; compliance-officer or founder emits | `partner_id` (UUID), `partner_category` (`referral`\|`reseller`\|`integration`\|`white_label`), `revenue_share_pct` (float 0–45), `agreement_doc_ref` (internal doc ID — **no partner company name in payload**), `dpa_signed` (boolean — **must be `true` for reseller and integration; Worker returns HTTP 422 if false for those categories**), `created_by_pam_session` (UUID — PAM session of FORM `form_admin`) |
| `enterprise.partner_revenue_share_paid` | HIGH | 7 yr | Quarterly revenue share payment released; compliance-officer emits before payment processing — **never after** | `partner_id`, `period_start` (ISO 8601 date — quarter start), `period_end` (ISO 8601 date — quarter end), `attributed_arr_usd` (float ≥ 0 — total ARR attributed in period), `share_amount_usd` (float ≥ 0 — `attributed_arr_usd × revenue_share_pct / 4`), `payment_method` (`stripe_payout`\|`wire_transfer`), `paid_by_pam_session` (UUID), `privacy_floor_check_passed` (boolean — **`z.literal(true)`; Worker returns HTTP 422 if `false` or absent — payment blocked**) |
| `enterprise.partner_deal_attributed` | STANDARD | 7 yr | Enterprise deal closes at stage S5 with `attributed_to_partner_id` FK set in `enterprise_pipeline_stages`; founder or form_system emits at deal close | `partner_id`, `deal_id` (UUID FK to `enterprise_pipeline_stages.id` — **internal only, never shared externally**), `tier` (`starter`\|`growth`\|`enterprise`), `acv_usd` (float ≥ 0), `attribution_type` (`referral`\|`reseller`\|`integration`\|`white_label`), `sales_cycle_days` (positive integer — days from `lead_date` to S5 close; drives §38.5.1 CAC velocity comparison) |
| `enterprise.partner_offboarded` | HIGH | 7 yr | Partner agreement terminated; `enterprise_partners.status` set to `terminated`; compliance-officer emits (manual — no automated path) | `partner_id`, `termination_reason` (`privacy_floor_breach`\|`no_go_criteria_violation`\|`performance_inactive`\|`mutual_agreement`\|`agreement_expiry`), `outstanding_share_usd` (float ≥ 0 — $0 if fully settled, > $0 if disputed), `dispute_open` (boolean), `privacy_incident_ref` (string or null — **required and non-null when `termination_reason = 'privacy_floor_breach'`; Worker returns HTTP 422 if null for that reason**), `offboarded_by_pam_session` (UUID) |

**Zod schemas (canonical source for `emit-audit-event` Worker validation):**

```typescript
// enterprise.partner_agreement_signed
const PartnerAgreementSignedPayload = z.object({
  partner_id:             z.string().uuid(),
  partner_category:       z.enum(['referral', 'reseller', 'integration', 'white_label']),
  revenue_share_pct:      z.number().min(0).max(45),
  agreement_doc_ref:      z.string(),
  dpa_signed:             z.boolean(),
  created_by_pam_session: z.string().uuid(),
});

// enterprise.partner_revenue_share_paid
const PartnerRevenueSharePaidPayload = z.object({
  partner_id:                  z.string().uuid(),
  period_start:                z.string().date(),
  period_end:                  z.string().date(),
  attributed_arr_usd:          z.number().min(0),
  share_amount_usd:            z.number().min(0),
  payment_method:              z.enum(['stripe_payout', 'wire_transfer']),
  paid_by_pam_session:         z.string().uuid(),
  privacy_floor_check_passed:  z.literal(true), // must be true; 422 if false or absent
});

// enterprise.partner_deal_attributed
const PartnerDealAttributedPayload = z.object({
  partner_id:       z.string().uuid(),
  deal_id:          z.string().uuid(),
  tier:             z.enum(['starter', 'growth', 'enterprise']),
  acv_usd:          z.number().min(0),
  attribution_type: z.enum(['referral', 'reseller', 'integration', 'white_label']),
  sales_cycle_days: z.number().int().positive(),
});

// enterprise.partner_offboarded
const PartnerOffboardedPayload = z.object({
  partner_id:               z.string().uuid(),
  termination_reason:       z.enum([
    'privacy_floor_breach',
    'no_go_criteria_violation',
    'performance_inactive',
    'mutual_agreement',
    'agreement_expiry',
  ]),
  outstanding_share_usd:    z.number().min(0),
  dispute_open:             z.boolean(),
  privacy_incident_ref:     z.string().nullable(),
  offboarded_by_pam_session: z.string().uuid(),
});
```

**HMAC chain requirement (PART-CHAIN-01):** All four events must include `prev_hash`. No strict ordering is enforced across the four event types within a single partner's lifecycle — a deal may be attributed before or after a revenue share payment for the same partner.

**Exception — PART-CHAIN-01 ordering invariant:** `enterprise.partner_offboarded` with `termination_reason = 'privacy_floor_breach'` or `'no_go_criteria_violation'` **must** be preceded in the HMAC chain by a `privacy.floor_breach_detected` event for the affected `tenant_id` within the 12 months prior to offboarding. The `emit-audit-event` Worker verifies this predecessor exists before chain-appending; HTTP 422 `PART_CHAIN_01_VIOLATION` on absence → triggers R-05 (HMAC chain integrity failure, `docs/INCIDENT_RESPONSE.md`).

**Additional per-event hard constraints enforced by the Worker:**
- `enterprise.partner_agreement_signed`: `dpa_signed` must be `true` when `partner_category ∈ {'reseller', 'integration'}` — enforced per §38.6.1; referral-only partners with no data access may have `dpa_signed = false`.
- `enterprise.partner_revenue_share_paid`: `privacy_floor_check_passed` must be `z.literal(true)` — HTTP 422 `PART_PAY_FLOOR_CHECK` if `false` or absent; payment is blocked upstream until attestation is confirmed.
- `enterprise.partner_offboarded`: `privacy_incident_ref` must be non-null when `termination_reason = 'privacy_floor_breach'` — HTTP 422 if null; the incident slug links to the R-22 incident record.

Under no circumstance may `enterprise.partner_revenue_share_paid` be emitted with `privacy_floor_check_passed = false` or absent. A blocked payment attempt is recorded by a separate operational log (not in the DEC-030 chain).

**Emitter assignment:** `enterprise.partner_agreement_signed` — `compliance-officer` or `founder` (manual via admin console, PAM session required — emitted before any data or commissions flow to or from the partner); `enterprise.partner_revenue_share_paid` — `compliance-officer` (manual, quarterly, after `privacy_floor_check_passed` attestation; no automated path — human sign-off is the compliance control); `enterprise.partner_deal_attributed` — `customer-success` or `founder` (manual at S5 deal close) or `form_system` (automated trigger when `enterprise_pipeline_stages.stage = 'S5'` and `attributed_to_partner_id IS NOT NULL`); `enterprise.partner_offboarded` — `compliance-officer` (manual only — no automated path; offboarding always requires human review and PAM session).

**SOC 2 evidence artefacts (store at `compliance/evidence/partners/` with `MANIFEST.sha256`):**

| Artefact | Description | SOC 2 criteria | Retention |
|---|---|---|---|
| **PART-E-001** | Annual export of `enterprise.partner_agreement_signed` chain events for all active partners — demonstrates executed agreements and DPA status | CC9.2 — Vendor and partner agreements in place | 7 yr |
| **PART-E-002** | Annual export of `enterprise.partner_revenue_share_paid` chain events — demonstrates financial governance; every row has `privacy_floor_check_passed: true` | CC9.2 — Vendor financial controls; CC4.1 — Control activities over financial obligations | 7 yr |
| **PART-E-003** | `enterprise.partner_offboarded` export for observation period — demonstrates formal closure and privacy breach escalation per PART-CHAIN-01 | CC9.2 — Vendor offboarding; CC7.4 — Incident response activated on privacy floor breach | 7 yr |

**Auditor narrative (CC9.2):** FORM manages its partner channel through formally executed Partner Agreements gated on DPA execution for reseller and integration partners (`dpa_signed = true` enforced at chain-append time). Revenue share payments are blocked unless `privacy_floor_check_passed: true` is explicitly attested by the compliance-officer — invariant enforced at the `emit-audit-event` Worker layer (HTTP 422 `PART_PAY_FLOOR_CHECK` on violation). PART-E-001 through PART-E-003 provide chain-level evidence for auditors querying the SOC 2 observation window. No individual employee health data or user identifiers are shared with any partner at any tier — enforced at the API layer (`REVOKE ALL ON enterprise_partners FROM form_api`), at the DEC-030 payload schema (`user_id` is absent from all four event types), and at the contract layer (Partner Agreement §38.6.2 non-waivable privacy floor clauses).

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
| `system.compliance_tool_connected` | 3 years | SOC 2 CC4.1/CC9.2 — DPA-first compliance tooling activation record; CTOOL-E-006 |
| `system.evidence_collection_automated` | 7 years | SOC 2 CC4.2 / CC7.1 — monthly evidence collection chain record; AUTO-E-001; EVD-CHAIN-01 invariant |
| `system.evidence_cron_stale` | 3 years | SOC 2 CC4.1 — dead-man's switch miss record; operational monitoring evidence |
| `system.evidence_cron_conflict` | 1 year | R2 ETag conflict abort record; operational record only |
| `admin.*` (key management) | 7 years | Encryption governance + incident investigation |
| `api_key.*` | 7 years | SOC 2 CC6.4 credential lifecycle evidence |
| `scim.ip_enforcement_*` / `scim.ip_blocked` | 7 years | SOC 2 CC6.1 network-layer access evidence |
| `scim.mass_deprovision_detected` / `scim.sync_suspended` / `scim.admin_lockout_recovery` / `scim.sync_resumed` | 7 years | SOC 2 A1.1 (availability threat monitoring), A1.2 (environmental threat detection), CC7.2 (proactive monitoring), CC7.3 (anomaly response), CC9.2 (IdP vendor risk) — INCIDENT_RESPONSE.md R-24 incident audit trail; `scim.mass_deprovision_detected` CRITICAL 7yr is the anchor evidence artefact |
| `scim.mass_reprovision_complete` | 3 years | SCIM mass deprovisioning recovery completion record; 3yr sufficient as operational recovery confirmation (primary incident evidence covered by 7yr CRITICAL `scim.mass_deprovision_detected`) |
| `sso.auth_policy_updated` / `sso.ip_allowlist_*` / `sso.ip_blocked` / `sso.mfa_enforcement_changed` / `sso.mfa_bypass_granted` / `sso.auth_policy_lockout_recovery` | 7 years | SOC 2 CC6.1/CC6.2/CC6.8 auth policy audit trail |
| `sso.mfa_enforcement_sweep_completed` | 3 years | Operational sweep completion; not required for SOC 2 observation evidence |
| `sso.mobile_browser_mode_set` | 7 years | SOC 2 CC6.1 (process isolation — system-browser mandate); CC6.2 (OAuth 2.0 BCP — RFC 9700 §4.1 compliance); SSO-WEB-E-001 evidence artefact; `docs/SSO_SCIM_IMPLEMENTATION.md §30.9` (DEC-059) |
| `sso.mobile_webview_blocked` | 7 years | HIGH-severity zero-fire violation detection; SOC 2 CC6.1/CC6.2/CC8.1; any emission triggers AL-SSO-WEB-01 P1 + R-05 investigation; `docs/SSO_SCIM_IMPLEMENTATION.md §30.9` (DEC-059) |
| `pam.*` | 7 years | SOC 2 CC6.1/CC6.2/CC6.3 JIT privilege access evidence; break-glass record |
| `sso.google_directory_*` | 7 years | SOC 2 CC6.3/CC9.2 Google Workspace Directory sync evidence |
| `session.bulk_revocation_*` / `session.tenant_nuke_*` / `session.revocation_kv_sync_error` | 7 years | SOC 2 CC6.3 timely logical access removal evidence |
| `scim.session_revocation_kv_fallback` | 7 years | SOC 2 CC6.3 (dual-path revocation under KV failure), CC7.2 (KV health monitoring — AL-REVOKE-01 trigger) |
| `security.rls_bypass_attempt` / `security.definer_function_cross_tenant` | 7 years | Tenant isolation breach evidence; SOC 2 CC6.6/PI1.5 |
| `asset.*` (device disposal) | 7 years | SOC 2 C1.2 confidential media disposal + CC6.5 access termination evidence |
| `vuln.*` (vulnerability management) | 7 years | SOC 2 CC7.1 threat identification + CC7.2 remediation tracking evidence; `vuln.sla_exception_granted` also maps to CC7.4 (response to identified events) |
| `compliance.erasure_sla_alert_fired` | 7 years | SOC 2 C1.2/P5.2 GDPR Art. 17 erasure SLA monitoring evidence; belt-and-suspenders secondary signal to §70 DSAR automation |
| `compliance.data_asset_inventory_reviewed` / `compliance.encryption_verified` / `compliance.disposal_audit_completed` | 3 years | C1 operating evidence cadence records; routine compliance audit completion records per SOC2_READINESS.md §73 |
| `enterprise.pricing_exception_approved` / `enterprise.consumer_price_updated` / `enterprise.list_price_updated` / `enterprise.price_floor_override_requested` | 7 years | SOC 2 CC1.4 pricing integrity evidence; COST_MODEL.md §31.8 pricing governance trail; CRITICAL-severity `enterprise.price_floor_override_requested` also maps to CC5.2 |
| `enterprise.ai_cost_monitor_alert` / `enterprise.ai_cost_outlier_flagged` | 3 years | FORM-internal AI token budget monitoring; operational cost governance record; SOC 2 A1.1 (capacity monitoring); COST_MODEL.md §33.9 |
| `enterprise.renewal_risk_flagged` | 3 years | Renewal risk classification record; SOC 2 CC5.2 (business risk assessment); COST_MODEL.md §34.8 — 3yr sufficient as risk band changes are operational signals, not contract evidence |
| `enterprise.churn_confirmed` / `enterprise.retention_discount_granted` | 7 years | Contract lifecycle evidence (churn type, ACV, contract_id FK); pricing exception linkage via `pricing_exception_event_id`; SOC 2 CC1.4/CC5.2; 7yr matches financial audit retention requirement per COST_MODEL.md §34.8 |
| `enterprise.mid_contract_termination_risk_flagged` / `enterprise.contract_amended` / `enterprise.early_termination_fee_waived` | 7 years | Contract amendment and ETF lifecycle evidence; SOC 2 CC5.2 (business risk assessment), CC7.2 (monitoring of controls); 7yr matches financial audit retention requirement per COST_MODEL.md §35.9; HMAC ordering guard enforced (risk-flagged prerequisite for ETF waiver within 12-month window) |
| `enterprise.implementation_kickoff_completed` / `enterprise.sso_scim_setup_verified` / `enterprise.implementation_cost_model_calibrated` | 7 years | Implementation lifecycle evidence; SOC 2 CC5.2 (deal-level implementation governance), CC7.2 (SSO/SCIM smoke-test verification monitoring); COST_MODEL.md §36.10 — 7yr matches deal lifecycle evidence floor; `enterprise.implementation_cost_model_calibrated` cross-links to DECISION_LOG OQ-08 closure record |
| `enterprise.partner_agreement_signed` / `enterprise.partner_deal_attributed` | 7 years | Partner agreement execution and deal attribution records; SOC 2 CC9.2 (PART-E-001 evidence artefact — executed agreements and DPA status; PART-E-003 — offboarding audit trail); 7yr matches Ukrainian Tax Code Art. 44 financial record floor and SOC 2 multi-cycle evidence requirement; COST_MODEL.md §38.8 |
| `enterprise.partner_revenue_share_paid` / `enterprise.partner_offboarded` | 7 years | HIGH-severity financial payment record and partner termination record; SOC 2 CC9.2/CC4.1 (PART-E-002 revenue share payment evidence — `privacy_floor_check_passed: true` invariant); CC7.4 (PART-E-003 — PART-CHAIN-01 privacy breach escalation evidence); `partner_revenue_share_paid` is a financial transaction record; COST_MODEL.md §38.8 |
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
| `enterprise.offboarding_initiated` / `enterprise.sso_scim_revoked` / `enterprise.data_deletion_started` / `enterprise.data_deletion_completed` / `enterprise.financial_pseudonymized` / `enterprise.deletion_attestation_issued` / `enterprise.offboarding_on_hold` / `enterprise.offboarding_hold_released` / `enterprise.offboarding_step_failed` | 7 years | Enterprise tenant off-boarding lifecycle — irreversible or legally consequential operations; SOC 2 C1.2 (disposal — OFB-E-001), P8.0 (privacy in disposal — OFB-E-002), CC6.3 (access removal — OFB-E-003), CC9.1 (business continuity — OFB-E-004); 7yr matches Ukrainian Tax Code Art. 44 + EU VAT Directive Art. 245 financial retention floor and SOC 2 multi-cycle evidence requirement; `docs/DATA_MODEL.md §25` |
| `enterprise.data_export_started` / `enterprise.data_export_completed` / `enterprise.offboarding_completed` | 7 years | Data export and attestation lifecycle — reversible package generation is STANDARD severity but 7yr retention required: these events are part of the off-boarding HMAC chain and must outlive OFB-E-001–E-006 evidence collection; `enterprise.data_export_completed` carries OFB-REGION-01 EU routing invariant (DEC-061 — `is_eu_region`, `r2_bucket`); `docs/DATA_MODEL.md §25` + `docs/DATA_MODEL.md §36` |
| `tenant.invite_sent` / `tenant.invite_accepted` / `tenant.invite_revoked` | 7 years | SOC 2 CC6.2 (INV-E-002 — every seat grant has a named `invited_by` authoriser; `tenant.invite_sent`), CC6.3 (INV-E-003 — timely reserved-seat release; `tenant.invite_revoked` HIGH = irreversible compliance action); `tenant.invite_accepted.linked_via` distinguishes SCIM auto-link from manual registration for access-review fieldwork; `email_hash` only — no plaintext `invited_email` in any payload; `docs/DATA_MODEL.md §27.11` |
| `tenant.invite_expired` / `tenant.bulk_invite_started` / `tenant.bulk_invite_completed` | 3 years | Enterprise seat invitation operational lifecycle: invite expiry sweep (pg_cron `invite_expiry_sweep` daily 02:30 UTC), bulk CSV-import batch framing (INVITE-BULK-CHAIN-01 open/close events); 3yr sufficient — access-grant evidence (INV-E-002/003) is covered by the 7yr `tenant.invite_sent` + `tenant.invite_revoked` rows above; `docs/DATA_MODEL.md §27.11` |
| `system.quarterly_perf_reference_initiated` / `system.quarterly_perf_reference_completed` | 3 years | SOC 2 A1.1 — k6 Cloud EU-West quarterly PERF-SLO-06 reference run evidence; PERF-CLOUD-CHAIN-01 ordering evidence (initiated → completed per `cloud_run_id`); `cloud_run_id` in `completed` event provides independent third-party k6 Cloud portal corroboration for LT-E-001 availability testing artefact; 3yr sufficient (operational performance evidence; not a financial or contract record); `docs/OBSERVABILITY.md §45.4` (DEC-058) |
| `user.account_deletion_initiated` / `user.data_erasure_completed` | 7 years | GDPR Art. 17 erasure chain-of-custody record; SOC 2 P4/P5.2/P6; `user.data_erasure_completed.slo_met` is GDPR-SLO-01 evidence; 7yr matches FORM's general compliance-record floor — individual user data is absent from payloads so retention does not conflict with the erasure itself |
| `user.art9_data_hard_deleted` | 7 years | GDPR Art. 9 enterprise off-boarding deletion record; GDPR-E-005/E-006 evidence artefacts; SOC 2 CC6.5/P6; `latency_seconds` field is GDPR-SLO-03 evidence; 7yr is the DEC-030 HIGH event standard and matches FORM's compliance record floor |
| `data.workout_data_purged` | 1 year | GDPR Art. 5(1)(e) storage-limitation enforcement; GDPR-E-002 evidence artefact (SOC 2 PI1.2); 1yr sufficient — operational purge log, not a contract or financial record; daily cadence means 365 events/yr; older runs add no compliance value |
| `data.audit_log_purge_completed` | 7 years | DEC-030 / CC6.5 audit log retention enforcement record; GDPR-E-003 evidence artefact; self-referential: this event is itself part of the audit log chain it governs; 7yr matches the retention period it enforces (the event that records the purge must outlive what it purged by at least one audit cycle)

| `ci.migration_applied` | 7 years | SOC 2 CC8.1 migration chain evidence — `migration_sha256` + `rls_policy_changed` flag; primary evidence artefact CI-E-002 for change management audit fieldwork |
| `ci.migration_failed` | 7 years | SOC 2 CC8.1 production migration failure record; CRITICAL severity — no dedup, no auto-resolve; cross-ref INCIDENT_RESPONSE.md R-02; `error_message_hash` (no raw SQL); 7yr matches financial audit floor |
| `ci.deployment_rolled_back` | 7 years | SOC 2 CC8.1 deployment rollback evidence; cross-ref CI-E-004; `failing_commit_sha` links to prior `system.deployment_completed` in chain — orphaned rollback is a chain anomaly |
| `ci.pipeline_failed` | 3 years | CI pipeline failure record; `is_main = true` failures are SOC 2 CC8.1 operating evidence (AL-CI-01); PR-branch failures are developer noise — 3yr sufficient for trend analysis, not required as long-term compliance evidence |
| `system.deployment_completed` | 5 years | SOC 2 CC8.1 deployment evidence — CI-E-001; `smoke_probe_passed` boolean binds deploy record to §16 S-001/S-002 probes; 5yr covers multiple SOC 2 observation periods |
| `system.backup_completed` | 5 years | SOC 2 A1.2 backup freshness evidence; primary signal for BC-SLO-01/02/02b freshness gauges; `backup_id` + `region` + `triggered_by` are infrastructure-only — no user or tenant data |
| `system.backup_failed` | 7 years | SOC 2 A1.2 backup failure record; CRITICAL severity — no auto-resolve; requires compliance-officer joint sign-off to close; `error_message_hash` SHA-256 prevents credential leakage in chain |
| `system.restore_test_initiated` | 7 years | SOC 2 A1.3 DR test initiation record; `test_id` UUID links initiation → completion in BC-CHAIN-01; `rationale` enum documents quarterly vs. annual drill cadence |
| `system.restore_test_completed` | 7 years | SOC 2 A1.3 primary evidence artefact BC-E-002; `rto_achieved_minutes ≤ 240 AND privacy_floor_verified = true` is the dual-condition pass criterion; `rpo_gap_minutes` documents actual data loss window in test |
| `system.restore_test_failed` | 7 years | SOC 2 A1.3 DR test failure record; CRITICAL for `privacy_floor_violation` failure reason (triggers R-BC-01 and clinical-safety notification); `failure_reason` enum enables structured post-mortem analysis |
| `system.backup_staleness_detected` | 7 years | SOC 2 A1.2 backup staleness monitoring evidence; emitted by pg_cron job 29 (every 4h); `dedup_key` prevents chain spam; `alert_fired` enum links to AL-BC-01/02/03 PagerDuty routing |
| `privacy.pia_filed` / `privacy.pia_completed` / `privacy.pia_veto_issued` / `privacy.constraint_relaxation_rejected` | 7 years | SOC 2 P1.1/P3.2/P8.1 PIA governance evidence; GDPR Art. 5(2) accountability record; `privacy.pia_veto_issued` CRITICAL is the clinical-safety decision record — 7yr ensures multiple SOC 2 observation periods are covered; `privacy.constraint_relaxation_rejected` provides a durable record that FORM declined to relax a privacy constraint on request of an enterprise customer or government, satisfying CC1.4 / P6.5 |
| `privacy.annual_review_scope_drafted` | 3 years | Routine annual privacy review scope record; P8.1 operating-effectiveness evidence; superseded each year by a new scope draft — 3yr covers the current and prior year for any inspection window |
| `privacy.annual_review_completed` | 7 years | Annual privacy review completion record; SOC 2 P8.1 monitoring and enforcement evidence; 7yr required — auditors conducting a Type II engagement may request evidence covering multiple annual review cycles |
| `privacy.incident_opened` (PI-P0 / PI-P1) | 7 years | SOC 2 P8.2 privacy incident register — PI-P0/P1 incidents involve confirmed or suspected Art. 9 special category data and may trigger GDPR Art. 33 supervisory authority notification; 7yr retention matches the supervisory authority look-back window and ensures the full incident history is available across multiple SOC 2 Type II observation periods; `art9_suspected` and `art33_clock_started` fields provide the auditor narrative for notification decisions |
| `privacy.incident_opened` (PI-P2 / PI-P3) | 3 years | SOC 2 P8.2 operational incident register for control-failure and process-deviation incidents; 3yr sufficient for operating-effectiveness evidence within a single SOC 2 certification cycle; PI-P2/P3 incidents do not trigger Art. 33 notifications and are not financial-adjacent records |
| `privacy.incident_reviewed` / `privacy.incident_closed` | 7 years | SOC 2 P8.2/P8.3 investigation and resolution records; 7yr required across the board — `art33_notification_id` in `privacy.incident_closed` references a supervisory authority case number, which is a financial-adjacent regulatory record; `post_incident_review_filed` attests to the PIR document existence; auditors inspecting multi-year Type II reports need the full remediation chain for every PI-P0/P1 incident in scope |
| `sla.incident_opened` / `sla.incident_closed` | 7 years | SOC 2 A1.1 SLA incident evidence — primary artefact for auditing covered downtime against the 99.9% commitment; `incident_id` links open→close in SLA-CHAIN-01; 7yr matches financial audit floor (credits issued against these incidents are themselves 7yr records) |
| `sla.measurement_reconciled` | 7 years | SOC 2 A1.1 measurement methodology evidence — dual-source reconciliation record (Better Stack vs. Cloudflare Analytics); `betterstack_report_sha256` cross-reference makes this the chain anchor for the monthly uptime figure; 7yr required — auditors need multi-year availability data for Type II opinion |
| `sla.credit_calculated` / `sla.credit_approved` / `sla.credit_adjusted` | 7 years | SOC 2 A1.1 SLA credit issuance chain — three-event sequence covering automated calculation, compliance-officer approval, and any dispute-driven adjustment; financial records (credit applied to invoices) must meet 7yr accounting retention floor per FORM financial policy |
| `sla.dispute_opened` | 3 years | Tenant dispute initiation record; operational lifecycle log; 3yr sufficient — primary dispute resolution evidence is captured in the higher-severity `sla.dispute_resolved` (7yr) record that must follow per SLA-CHAIN-03 |
| `sla.dispute_resolved` | 7 years | SOC 2 A1.1 dispute resolution record — `outcome` (upheld/rejected) + `final_credit_usd` + `resolved_by` constitutes the compliance-officer decision record; 7yr matches financial audit floor and ensures availability across multiple SOC 2 Type II observation periods |
| `sla.maintenance_window_registered` | 3 years | Planned maintenance exclusion registration record; operational planning log; the `notification_hours` field provides evidence the §23.4 advance-notice requirement was met; 3yr sufficient — maintenance window records are operational, not financial |
| `sla.exclusion_reclassified` | 7 years | SOC 2 A1.1 probe false-positive reclassification evidence — `security-engineer` approval + second-approver requirement for windows > 30 min creates a tamper-evident record that downtime exclusions are subject to independent review; 7yr ensures auditors can inspect reclassification decisions across the full SOC 2 history |
| `tenant.white_label_provisioned` / `tenant.white_label_cert_renewed` | 7 years | SOC 2 A1.1 — WL-E-001 evidence artefact (annual `tenant.white_label_cert_renewed` chain export demonstrates no cert expired during observation period); `tenant.white_label_provisioned` is the WL-CHAIN-01 lifecycle anchor — must precede all `tenant.white_label_cert_*` events for the same `tenant_id`; 7yr matches enterprise contract and SOC 2 multi-cycle evidence floor |
| `tenant.white_label_cert_expiry_warning` / `tenant.white_label_revoked` | 7 years | SOC 2 CC7.2 — WL-E-002 evidence artefact (AL-WL-03/04/05 PagerDuty config + 90-day warning history); `tenant.white_label_revoked` satisfies CC6.3 timely access-removal record for the white-label surface (Cloudflare Custom Hostname deleted at revocation); `revoked_by_pam_session` links to the PAM chain (OBSERVABILITY §29); 7yr matches enterprise contract audit and access-removal evidence floor |
| `tenant.white_label_cert_expiry_breach` / `system.white_label_cert_check_stale` | 7 years | SOC 2 A1.1 CRITICAL — `tenant.white_label_cert_expiry_breach` constitutes WL-SLO-01 + WL-SLO-02 breach; directly triggers §23 SLA credit engine (`sla_incident_id` FK); financial record requiring 7yr accounting floor; `system.white_label_cert_check_stale` is the WL-CHAIN-01 monitoring gap record (HIGH — detective control failure); 7yr matches `system.cron_job_stale` precedent for operational monitoring failures requiring multi-cycle SOC 2 coverage |
| `dsar.certificate_issued` | 7 years | SOC 2 P8.0 — DSAR-E-011 evidence artefact (tamper-evident deletion certificate issuance record; quarterly `payload_hash` cross-check against R2 object SHA-256 confirms certificate integrity post-issuance); GDPR Art. 17 accountability proof that erasure was communicated with a verifiable certificate; `r2_object_key` + `payload_hash` create a dual-layer integrity trail (HMAC chain + R2 object); 7yr matches CERT-CHAIN-01 evidence floor and SOC 2 multi-cycle observation window |
| `dsar.certificate_delivered` | 7 years | SOC 2 P8.0 — delivery confirmation closes the CERT-CHAIN-01 four-event chain (`dsar.deletion_soft → deletion_confirmed → certificate_issued → certificate_delivered`); CC7.2 — `delivered_to_hash` + `delivery_channel` document that the disposal notification reached the intended party; STANDARD severity; 7yr required because the delivery record completes Art. 17 accountability — shorter retention would leave `dsar.certificate_issued` without a verifiable follow-through across multi-year SOC 2 Type II observation windows |

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

### Enterprise contract amendment & ETF lifecycle events (DEC-030 HMAC-chained · COST_MODEL §35)

> Defined in `docs/COST_MODEL.md §35.9`. Three events creating an immutable record of mid-contract termination risk, contract amendments (including price-lock renewals), and early termination fee waivers. Required for COST_MODEL §35.10 checklist item 1 (P0, M10 — before first enterprise multi-year contract close). **Privacy invariant (all three events):** `tenant_id` only — no `user_id`, no individual employee names, no coaching or health data. CHS (Customer Health Score) is a tenant-level composite metric. Verbatim exit interview notes and CSM call notes must remain in CRM only and must never appear in the DEC-030 chain. Cross-ref: SOC 2 CC5.2 (business risk assessment), CC7.2 (monitoring of controls); `docs/COST_MODEL.md §35.9.1–§35.9.3` (canonical event definitions + Zod schemas); `docs/OBSERVABILITY.md §36` (AL-ETF-01 alert rule + pg_cron job 25 `mid_contract_termination_risk_check`); `docs/MSA_TEMPLATE.md §11.4` (ETF clause — P0 M10 counsel review required). Closes COST_MODEL.md §35.10 checklist item 1 (P0, M10).

| Event name | Severity | Retention | Trigger | Key payload fields |
|---|---|---|---|---|
| `enterprise.mid_contract_termination_risk_flagged` | HIGH | 7 yr | pg_cron job 25 detects `chs_score < 20` sustained ≥ 4 consecutive weekly snapshots AND contract `days_remaining > 180`; 30-day per-tenant deduplication via `mid_contract_risk_alerted_at` in `tenants` table; emitted before CSM notification | `tenant_id`, `chs_score` (integer, most recent weekly), `weeks_below_threshold` (integer ≥ 4), `days_remaining` (integer), `contract_acv_usd` (integer), `alerted_to` (CSM + CS lead role enum) |
| `enterprise.contract_amended` | HIGH | 7 yr | Any signed contract amendment — price-lock renewals, seat reductions ≤ 10% deferred to annual date, custom SLA addenda; emitter: customer-success (manual, via `POST /internal/enterprise/contract-amendments`) | `tenant_id`, `amendment_type` (`price_lock_renewal` \| `seat_reduction` \| `sla_addendum` \| `other`), `contract_id`, `amendment_signed_at`, `acv_delta_usd` (integer, signed — negative for seat reductions), `approved_by_role`, `decision_log_ref` (optional — required for seat_reduction > 10% of contracted seats) |
| `enterprise.early_termination_fee_waived` | HIGH | 7 yr | ETF waiver approved per §35.5.5 approval authority matrix; `ev_analysis_completed` must be `true` for waivers > $5,000 or `emit-audit-event` Worker returns HTTP 422 with `ETF_WAIVER_REQUIRES_EV_ANALYSIS`; emitter: founder or approval-level actor (manual) | `tenant_id`, `etf_amount_waived_usd` (integer, positive), `exit_month` (integer ≥ 1), `remaining_months` (integer ≥ 1), `waiver_reason` (enum: `product_gap` \| `force_majeure` \| `champion_departed` \| `reference_value` \| `competitor_acquisition` \| `legal_dispute_resolution` \| `goodwill_concession` \| `small_etf_enforcement_uneconomic`), `approval_actor_id` (UUID, pseudonymous), `ev_analysis_completed` (boolean), `decision_log_ref` (string, optional) |

**HMAC chain ordering invariants:** No strict ordering is required between the three events (they represent independent lifecycle moments — risk detection, amendment, and waiver). However: `enterprise.mid_contract_termination_risk_flagged` must be emitted before any `enterprise.early_termination_fee_waived` for the same `tenant_id` within a 12-month window — the `emit-audit-event` Worker enforces this with HTTP 422 (`MID_CONTRACT_RISK_REQUIRED_BEFORE_ETF_WAIVER`) if no prior risk-flagged event exists for the tenant in the audit chain within 12 months (exception: `small_etf_enforcement_uneconomic` waiver reason bypasses this guard). **Privacy invariant:** `etf_amount_waived_usd` is an integer USD amount — no per-employee cost data, no individual user identifiers.

---

### Enterprise implementation lifecycle events (DEC-030 HMAC-chained · COST_MODEL §36)

> Defined in `docs/COST_MODEL.md §36.10`. Three events covering the per-deal implementation audit trail — from kickoff through SSO/SCIM verification to cost-model calibration. Required for COST_MODEL §36.11 item 1 (P0, M8 — register all three events before first enterprise deal closes). **Privacy invariant (all three events):** `tenant_id` + operational metadata only — no individual employee `user_id`, no health values, no coaching content. `csm_actor_id` and `engineer_actor_id` are internal FORM team UUIDs (not tenant employee IDs). `enterprise.implementation_cost_model_calibrated` contains aggregate cost data only — no per-tenant names, no individually identifiable deal amounts in payload. Cross-ref: SOC 2 CC5.2 (implementation governance as a business risk control); CC7.2 (SSO/SCIM smoke-test as a verification monitoring control); `docs/COST_MODEL.md §36.3–§36.6` (activity taxonomy and hour model — time basis for `enterprise_impl_time_log`); `docs/SSO_SCIM_IMPLEMENTATION.md §7.4` (test procedure — technical counterpart of `enterprise.sso_scim_setup_verified`); `docs/ENTERPRISE_ONBOARDING.md` (operational playbook — sequential trigger points for these events); `docs/DECISION_LOG.md` (DEC-XXX — OQ-08 closure record, referenced in `enterprise.implementation_cost_model_calibrated.decision_log_ref`). Closes COST_MODEL.md §36.11 checklist item 1 (P0, M8).

| Event name | Severity | Retention | Trigger | Key payload fields |
|---|---|---|---|---|
| `enterprise.implementation_kickoff_completed` | STANDARD | 7 yr | Kickoff call(s) complete; implementation project plan confirmed with customer; emitter: CSM (founder in solo phase) | `tenant_id`, `deal_sequence` (int — nth deal, for §36.9 OQ-08 cost calibration), `contracted_tier` (`starter`\|`growth`\|`enterprise`), `contracted_seats` (int), `idp_type` (`okta`\|`azure_ad`\|`google_workspace`\|`onelogin`\|`adfs`\|`pingfederate`\|`jumpcloud`\|`other_saml`\|`other_oidc`), `white_label_enabled` (bool), `eu_data_residency` (bool — EU premium applies per §36.6.2 if true), `kickoff_date` (ISO 8601 date), `target_go_live_date` (ISO 8601 date), `csm_actor_id` (UUID — FORM team member, not tenant employee) |
| `enterprise.sso_scim_setup_verified` | STANDARD | 7 yr | SSO IdP-initiated and SP-initiated flows pass smoke test per SSO_SCIM_IMPLEMENTATION.md §7.4; SCIM provisioning verified end-to-end; emitter: engineer (founder in solo phase) | `tenant_id`, `idp_type` (enum — same values as kickoff event), `sso_modes_verified` (object: `idp_initiated: bool`, `sp_initiated: bool`), `scim_features_enabled` (object: `user_crud: bool`, `group_sync: bool`, `role_mapping: bool`, `jit_provisioning: bool`), `eu_data_residency_confirmed` (bool), `engineer_actor_id` (UUID — FORM team member, not tenant employee), `verification_date` (ISO 8601 date) |
| `enterprise.implementation_cost_model_calibrated` | STANDARD | 7 yr | OQ-08 closure: first 3 deals fully time-tracked in `enterprise_impl_time_log`, §8.2 updated, DECISION_LOG entry written; emitter: founder (data-engineer post-Series A) | `deals_analysed` (int ≥ 3), `avg_impl_cost_starter` (number, optional — omit if no starter deals in sample), `avg_impl_cost_growth` (number, optional), `avg_impl_cost_enterprise` (number, optional), `variance_vs_model_pct` (signed float — positive = actuals exceed §36.6.1 model; negative = actuals below), `model_version_updated` (string — e.g. `"COST_MODEL.md §8.2 v2.3"`), `decision_log_ref` (string — DEC-XXX slug), `calibration_date` (ISO 8601 date) |

**HMAC chain requirement:** All three events must include `prev_hash`. No strict predecessor-successor ordering is enforced between the three event types — each fires at distinct calendar milestones (kickoff may precede SSO verification by weeks; cost calibration fires only after 3 deals close). Within a single deal, `enterprise.implementation_kickoff_completed` should logically precede `enterprise.sso_scim_setup_verified` for the same `tenant_id`; the `emit-audit-event` Worker logs a WARNING (non-blocking) if `sso_scim_setup_verified` is emitted with no prior `implementation_kickoff_completed` for the same `tenant_id`. `enterprise.implementation_cost_model_calibrated` carries no `tenant_id` field — it is a model-level aggregate event, not a per-tenant record — so no chain guard applies. No automatic PagerDuty routing for these events (STANDARD severity); they appear in the weekly chain audit and in the CSM implementation dashboard only.

**Emitter assignment:** `enterprise.implementation_kickoff_completed` — `customer-success` (manual via admin console at kickoff completion; solo phase: founder); `enterprise.sso_scim_setup_verified` — `platform-engineer` or `form_system` (automated smoke-test harness if available per §36.11 item 1 DoD; otherwise manual via admin console at SSO/SCIM verification step per SSO_SCIM_IMPLEMENTATION.md §7.4); `enterprise.implementation_cost_model_calibrated` — founder (manual via admin console; post-Series A: data-engineer). Neither `csm_actor_id` nor `engineer_actor_id` is a tenant employee UUID — both are internal FORM team identities and safe to include in the HMAC chain payload per the privacy invariant above.

---

### GDPR data lifecycle events (DEC-030 HMAC-chained · OBSERVABILITY §37)

> Defined in `docs/OBSERVABILITY.md §37`. Five events covering the complete GDPR data-deletion and retention enforcement pipeline — consumer account deletion, DSAR erasure completion, enterprise Art. 9 off-boarding, workout-data purge (job 26), and audit-log retention purge (job 27). **Privacy floor (all five events):** no plaintext `user_id` in any payload. Consumer-facing events reference `deletion_request_id` or `erasure_request_id` (UUID FKs to Postgres records protected by `form_system`-only RLS). Aggregate count fields only. The `user.art9_data_hard_deleted` payload contains `tenant_id` and counts — no individual employee UUIDs. **GDPR-SLO-03 / DEC-036:** `user.art9_data_hard_deleted` must be emitted within 4 hours of `tenants.offboarding_initiated_at`; `latency_seconds > 14400` indicates a GDPR Art. 9 breach requiring immediate P0 response per AL-GDPR-03. **DEC-030 ordering invariant for job 27:** `data.audit_log_purge_completed` must be emitted and HTTP 200 confirmed *before* `DELETE FROM audit_log_events` executes — the chain entry preserves the end-state before the oldest rows are removed. Cross-ref: SOC 2 PI1.2 (data minimisation), P4 (right to erasure), P5.2 (pseudonymisation evidence), P6 (privacy choices), CC6.5 (retention/deletion); `docs/OBSERVABILITY.md §37.4` (GDPR-SLO-01/02/03/04/05); `docs/OBSERVABILITY.md §37.5` (AL-GDPR-01 through AL-GDPR-06); `docs/OBSERVABILITY.md §37.7` (pg_cron jobs 26 and 27 DDL + ordering invariant); GDPR Art. 5(1)(e) (storage limitation), Art. 17 (right to erasure), Art. 9 (special categories). Closes OBSERVABILITY.md §37.10 checklist item 11 (P1, M6 — AUDIT_LOG_SCHEMA.md update required).

| Event type | Severity | Retention | Trigger | Key payload fields |
|---|---|---|---|---|
| `user.account_deletion_initiated` | STANDARD | 7 yr | Consumer account deletion Worker: user initiates deletion in-app → `users.deleted_at = NOW()` (soft-delete); 30-day hold period begins | `deletion_request_id` (UUID — internal deletion record; maps to `user_id` via `form_system`-only RLS — **no `user_id` in payload**), `initiated_at` (ISO 8601), `hard_delete_scheduled_at` (ISO 8601 — `initiated_at + 30 days`), `account_type` (`"consumer"` \| `"enterprise_member"` — enterprise members deprovision via SCIM; only `"consumer"` uses this Worker path) |
| `user.data_erasure_completed` | STANDARD | 7 yr | DSAR erasure Worker: deletion cascade confirmed complete; `data_subject_requests.status` updated to `'completed'` | `erasure_request_id` (UUID — FK to `data_subject_requests.id`; **no `user_id` in payload** — cross-ref via RLS-gated table), `request_age_days` (float — days from `requested_at` to completion; ≤ 25.0 = GDPR-SLO-01 met; ≤ 30.0 = Art. 17 met), `tables_purged` (string array — data categories erased, e.g. `["workout_sessions","coaching_turns","cv_session_keypoints"]`), `completed_at` (ISO 8601), `slo_met` (boolean — `request_age_days ≤ 25.0`) |
| `user.art9_data_hard_deleted` | **HIGH** | 7 yr | Enterprise Art. 9 off-boarding Worker: `tenants.status = 'offboarding'` triggers hard-delete of all Art. 9 health data for tenant users; target < 4h (GDPR-SLO-03 / DEC-036); emitted immediately after deletion cascade completes | `tenant_id` (UUID), `offboarding_initiated_at` (ISO 8601), `hard_delete_completed_at` (ISO 8601), `latency_seconds` (integer — elapsed time; **`latency_seconds > 14400` = GDPR-SLO-03 breach, triggers AL-GDPR-03 P0**), `users_hard_deleted_count` (integer — number of tenant users whose Art. 9 data was removed), `art9_tables_purged` (string array — e.g. `["workout_sessions","workout_sets","body_metrics","cv_session_keypoints","coaching_turns"]`) |
| `data.workout_data_purged` | STANDARD | 1 yr | pg_cron job 26 `workout_data_purge` (daily 02:00 UTC): permanent DELETE of `workout_sets`, `workout_sessions`, `body_metrics` for users with `deleted_at < NOW() - INTERVAL '30 days'`; emitted after successful DELETE | `purge_date` (ISO 8601 date), `users_purged_count` (integer — distinct deleted users whose data was removed), `workout_sessions_deleted` (integer), `workout_sets_deleted` (integer), `body_metrics_deleted` (integer), `pg_cron_run_id` (bigint — `cron.job_run_details.runid` for §12.6 freshness cross-reference) |
| `data.audit_log_purge_completed` | STANDARD | 7 yr | pg_cron job 27 `audit_log_retention_purge` (monthly, 1st, 03:00 UTC): 7-year (2,557-day) retention enforcement; **emitted and HTTP 200 confirmed BEFORE `DELETE FROM audit_log_events` executes** (DEC-030 ordering invariant — chain entry must precede row removal) | `purge_month` (YYYY-MM), `rows_eligible` (integer — COUNT before DELETE; 0 until ~2033), `rows_deleted` (integer), `oldest_retained_event_timestamp` (ISO 8601 — oldest event remaining after purge), `chain_verification_result` (`"pass"` \| `"skip_no_eligible_rows"` — Worker returns HTTP 422 and aborts purge if verification fails), `pg_cron_run_id` (bigint) |

```typescript
// Zod schemas — canonical source for emit-audit-event Worker validation

const UserAccountDeletionInitiatedPayload = z.object({
  deletion_request_id: z.string().uuid(),
  initiated_at:        z.string().datetime(),
  hard_delete_scheduled_at: z.string().datetime(),
  account_type:        z.enum(['consumer', 'enterprise_member']),
});

const UserDataErasureCompletedPayload = z.object({
  erasure_request_id:  z.string().uuid(),
  request_age_days:    z.number().nonnegative(),
  tables_purged:       z.array(z.string()).min(1),
  completed_at:        z.string().datetime(),
  slo_met:             z.boolean(),
});

const UserArt9DataHardDeletedPayload = z.object({
  tenant_id:                  z.string().uuid(),
  offboarding_initiated_at:   z.string().datetime(),
  hard_delete_completed_at:   z.string().datetime(),
  latency_seconds:            z.number().int().nonnegative(),
  users_hard_deleted_count:   z.number().int().nonnegative(),
  art9_tables_purged:         z.array(z.string()).min(1),
});

const DataWorkoutDataPurgedPayload = z.object({
  purge_date:               z.string().date(),
  users_purged_count:       z.number().int().nonnegative(),
  workout_sessions_deleted: z.number().int().nonnegative(),
  workout_sets_deleted:     z.number().int().nonnegative(),
  body_metrics_deleted:     z.number().int().nonnegative(),
  pg_cron_run_id:           z.number().int().positive(),
});

const DataAuditLogPurgeCompletedPayload = z.object({
  purge_month:                    z.string().regex(/^\d{4}-\d{2}$/),
  rows_eligible:                  z.number().int().nonnegative(),
  rows_deleted:                   z.number().int().nonnegative(),
  oldest_retained_event_timestamp: z.string().datetime(),
  chain_verification_result:       z.enum(['pass', 'skip_no_eligible_rows']),
  pg_cron_run_id:                  z.number().int().positive(),
});
```

**HMAC chain requirements:**

- `user.account_deletion_initiated` — emitted synchronously within the account deletion Worker transaction (fail-closed: failure aborts the soft-delete). If the user was an enterprise member recently deprovisioned via SCIM, `session.bulk_revocation_complete` will appear earlier in the chain for the associated `tenant_id`; no strict predecessor requirement for this event itself. Followed within 30 days by `billing.user_erased` (from §§Billing & GDPR erasure events) for the same account.
- `user.data_erasure_completed` — constitutes the GDPR Art. 17 chain-of-custody record. Chain auditors verify completeness by locating `data.individual_deletion` (existing taxonomy) followed by this event referencing the same `erasure_request_id`. No chain guard enforced (ordering may vary if erasure Worker processes in batches); compliance-officer must verify both events present for any DSAR inspection.
- `user.art9_data_hard_deleted` — emitted synchronously inside the Art. 9 off-boarding Worker transaction; `latency_seconds` field is the GDPR-SLO-03 evidence value. If `latency_seconds > 14400`, AL-GDPR-03 fires (P0, dual-page `form-platform` + `form-compliance`, no auto-resolve) — **do not emit the event and suppress the alert**; emit the event even on SLO breach so auditors can see the latency value.
- `data.workout_data_purge_completed` — low-volume (one emission per daily job run); no predecessor required. Emitter must confirm DELETE row count matches `users_purged_count` before emitting.
- `data.audit_log_purge_completed` — **DEC-030 ordering invariant**: this event must enter the chain (HTTP 200 from `emit-audit-event` Worker) before `DELETE FROM audit_log_events` executes. The self-referential constraint is: the purge event itself is a new audit log row; it becomes the new chain tail before the oldest rows are removed. A chain break on this event (e.g. `emit-audit-event` Worker unreachable) must abort the DELETE and trigger AL-GDPR-05 (P1).

**Emitter assignments:** `user.account_deletion_initiated` — `form_system` (consumer account deletion Worker, automated); `user.data_erasure_completed` — `form_system` (DSAR erasure Worker, automated); `user.art9_data_hard_deleted` — `form_system` (enterprise Art. 9 off-boarding Worker, automated); `data.workout_data_purged` — `form_system` (pg_cron job 26 via pg_net, automated daily); `data.audit_log_purge_completed` — `form_system` (pg_cron job 27 via pg_net, automated monthly, BEFORE DELETE). None of these events may be emitted manually via the admin console.

---

### Privacy Impact Assessment events (DEC-030 HMAC-chained · PRIVACY_IMPACT.md §7 · P1.1/P3.2/P8.1)

> Defined in `docs/PRIVACY_IMPACT.md §7`. Six events covering the full PIA governance lifecycle — from filing through completion, clinical-safety veto, constraint relaxation rejection, and annual review. Required for PRIVACY_IMPACT.md §11 P0/M4 item 1 (register all six events in AUDIT_LOG_SCHEMA.md before first enterprise pilot). **Privacy floor (all six events):** no `user_id`, `tenant_id`, or individual health values in any payload. `filed_by` and `completed_by` fields are internal FORM compliance-officer or clinical-safety UUIDs — not customer/employee IDs. `constraints_affected[]` and `constraints_protected[]` carry constraint IDs (e.g. `"OC-08"`, `"EF-05"`), not data values. **PIA-CHAIN-01 ordering invariant:** `privacy.pia_filed` must precede `privacy.pia_completed` for the same `pia_id`; `emit-audit-event` Worker logs a WARNING (non-blocking) if `privacy.pia_completed` is emitted with no prior `privacy.pia_filed` for the same `pia_id`. For veto events: `privacy.pia_veto_issued` may be emitted independently of `privacy.pia_completed` (veto terminates the PIA — no completion event is required after a veto). **PRIV-VETO-001 alert:** `privacy.pia_veto_issued` at CRITICAL severity triggers immediate PagerDuty dual-page to `compliance-officer` and `founder` — no dedup, no auto-resolve. Cross-ref: SOC 2 P1.1 (privacy notice management), P3.2 (explicit consent / change management record), P8.1 (monitoring and enforcement — annual review evidence); GDPR Art. 5(2) (accountability); `docs/PRIVACY_IMPACT.md §4` (five-step PIA process — these events are emitted at Steps 1 and 5); `docs/PRIVACY_IMPACT.md §5` (clinical-safety veto procedure — `privacy.pia_veto_issued` is the HMAC-chained record of the veto decision); `docs/PRIVACY_IMPACT.md §10` (PIA Register — `pia_id` FK links chain entry to register row). Closes PRIVACY_IMPACT.md §11 P0/M4 checklist item 1.

| Event type | Severity | Retention | Trigger | Key payload fields |
|---|---|---|---|---|
| `privacy.pia_filed` | STANDARD | 7 yr | Compliance-officer acknowledges PIA ticket at Step 1 of §4 process; Linear ticket created; emitter: compliance-officer (manual via admin console) | `pia_id` (string — `PIA-YYYY-NNN` format), `trigger_type` (`T-1`…`T-7` per §3), `proposed_by` (role enum — proposer of the change), `constraints_affected` (string array — constraint IDs, e.g. `["OC-08"]`), `linear_ticket_url` (string), `filed_by` (UUID — compliance-officer user_id) |
| `privacy.pia_completed` | HIGH | 7 yr | Compliance-officer records final decision at Step 5 of §4 process; emitter: compliance-officer (manual) | `pia_id` (string), `risk_rating` (`low`\|`medium`\|`high`\|`critical` per §4 risk-rated SLA table), `clinical_safety_required` (boolean), `clinical_safety_decision` (`approved`\|`approved_with_conditions`\|`vetoed`\|`not_required`), `decision` (`approved`\|`approved_with_conditions`\|`rejected`), `conditions` (string array — condition text list; empty if unconditional approval), `gdpr_dpia_delta_required` (boolean — whether GDPR_DPIA.md requires amendment), `completed_by` (UUID — compliance-officer user_id) |
| `privacy.pia_veto_issued` | **CRITICAL** | 7 yr | Clinical-safety issues veto at §5 step; **triggers PRIV-VETO-001 PagerDuty dual-page (compliance-officer + founder) immediately; no auto-resolve**; emitter: clinical-safety agent | `veto_id` (UUID — unique veto record), `pia_id` (string — the PIA being vetoed), `constraints_protected` (string array — constraint IDs the veto protects), `harm_prevented` (string, max 500 chars — free-text harm description; no health values, no user data), `vetoed_by` (literal `"clinical-safety"`), `acknowledged_by_founder` (boolean — set to `false` at emission; founder must update to `true` within 24h via admin console) |
| `privacy.constraint_relaxation_rejected` | HIGH | 7 yr | Any constraint relaxation proposal formally rejected by compliance-officer, regardless of whether a full PIA was filed; emitter: compliance-officer (manual) | `pia_id` (string — optional; null if rejection was made without a full PIA filing), `constraint_id` (string — e.g. `"EF-05"`, `"OC-08"`), `requester_type` (`internal`\|`enterprise_customer`\|`government`), `rejection_reason` (string, max 500 chars), `rejected_by` (UUID — compliance-officer user_id) |
| `privacy.annual_review_scope_drafted` | STANDARD | 3 yr | Compliance-officer finalises annual review scope checklist (Q1 §9 table) and drafts review agenda; emitter: compliance-officer (manual) | `review_year` (integer — calendar year, e.g. `2027`), `scope_items` (string array — §9 PR-1 through PR-9 item IDs included in scope), `drafted_by` (UUID — compliance-officer user_id) |
| `privacy.annual_review_completed` | HIGH | 7 yr | Compliance-officer signs off annual privacy review at P8.1; emitter: compliance-officer (manual) | `review_year` (integer), `dpia_version_reviewed` (string — e.g. `"v1.2"`), `pias_in_scope` (string array — PIA IDs reviewed this cycle), `gaps_opened` (non-negative integer — new gaps discovered), `gaps_closed` (non-negative integer — gaps remediated this cycle), `sign_off_by` (UUID — compliance-officer user_id) |

```typescript
// Zod schemas — canonical source for emit-audit-event Worker validation
import { z } from 'zod';

const PIA_ID_REGEX = /^PIA-\d{4}-\d{3}$/;

const PiaFiledSchema = z.object({
  pia_id:               z.string().regex(PIA_ID_REGEX),
  trigger_type:         z.enum(['T-1','T-2','T-3','T-4','T-5','T-6','T-7']),
  proposed_by:          z.enum([
                          'product-manager','enterprise-architect','compliance-officer',
                          'security-engineer','platform-engineer','founder','clinical-safety',
                        ]),
  constraints_affected: z.array(z.string()).min(1),
  linear_ticket_url:    z.string().url(),
  filed_by:             z.string().uuid(),
});

const PiaCompletedSchema = z.object({
  pia_id:                   z.string().regex(PIA_ID_REGEX),
  risk_rating:              z.enum(['low','medium','high','critical']),
  clinical_safety_required: z.boolean(),
  clinical_safety_decision: z.enum(['approved','approved_with_conditions','vetoed','not_required']),
  decision:                 z.enum(['approved','approved_with_conditions','rejected']),
  conditions:               z.array(z.string()),
  gdpr_dpia_delta_required: z.boolean(),
  completed_by:             z.string().uuid(),
});

const PiaVetoIssuedSchema = z.object({
  veto_id:                  z.string().uuid(),
  pia_id:                   z.string().regex(PIA_ID_REGEX),
  constraints_protected:    z.array(z.string()).min(1),
  harm_prevented:           z.string().max(500),
  vetoed_by:                z.literal('clinical-safety'),
  acknowledged_by_founder:  z.boolean(),
});

const ConstraintRelaxationRejectedSchema = z.object({
  pia_id:           z.string().regex(PIA_ID_REGEX).optional(),
  constraint_id:    z.string().regex(/^(EF|OC|DM)-\d{2}$/),
  requester_type:   z.enum(['internal','enterprise_customer','government']),
  rejection_reason: z.string().max(500),
  rejected_by:      z.string().uuid(),
});

const AnnualReviewScopeDraftedSchema = z.object({
  review_year:  z.number().int().min(2026),
  scope_items:  z.array(z.string()).min(1),
  drafted_by:   z.string().uuid(),
});

const AnnualReviewCompletedSchema = z.object({
  review_year:           z.number().int().min(2026),
  dpia_version_reviewed: z.string(),
  pias_in_scope:         z.array(z.string()),
  gaps_opened:           z.number().int().nonneg(),
  gaps_closed:           z.number().int().nonneg(),
  sign_off_by:           z.string().uuid(),
});
```

**HMAC chain requirements (PIA-CHAIN-01):**

- `privacy.pia_filed` — emitted synchronously when compliance-officer opens PIA ticket (Step 1). Fail-closed on `emit-audit-event` error — PIA process must not proceed until chain entry is confirmed (HTTP 200).
- `privacy.pia_completed` — emitted synchronously at Step 5 final decision. `emit-audit-event` Worker logs WARNING if no prior `privacy.pia_filed` with same `pia_id` in chain (PIA-CHAIN-01); WARNING is non-blocking to allow retroactive filing during compliance programme bootstrap.
- `privacy.pia_veto_issued` — emitted by clinical-safety agent immediately upon veto decision. May precede or replace `privacy.pia_completed`; no strict predecessor requirement. CRITICAL severity → PagerDuty PRIV-VETO-001 auto-fires.
- `privacy.constraint_relaxation_rejected` — emitted independently of PIA lifecycle (rejections without a formal PIA also require a chain entry). No predecessor requirement.
- `privacy.annual_review_scope_drafted` — no predecessor requirement; emitted once per calendar year at Q1 scope finalization.
- `privacy.annual_review_completed` — no strict predecessor requirement beyond a matching `privacy.annual_review_scope_drafted` for the same `review_year` (WARNING if absent, non-blocking).

**Emitter assignments:** All six events emitted manually by compliance-officer via admin console, except `privacy.pia_veto_issued` which is emitted by the clinical-safety agent. No automated system path for any PIA event — these are governance decisions requiring human judgement. `form_system` has INSERT privilege for break-glass scenarios only (e.g., retroactive baseline PIA-2026-000).

---

### Privacy Incident Lifecycle events — full Zod schemas (DEC-030 HMAC-chained · SOC2_READINESS §77.4 · P8.2/P8.3)

> Canonical Zod schemas for `emit-audit-event` Worker validation. All three events must be deployed together — PRIV-INC-CHAIN-01 requires `privacy.incident_opened` to be the chain anchor; partial deployment of only `incident_reviewed` or `incident_closed` without `incident_opened` enforcement creates an undetectable chain gap. Severity and retention split by incident class: `severity_class IN ('PI-P0','PI-P1')` → CRITICAL/HIGH severity, 7yr; `severity_class IN ('PI-P2','PI-P3')` → STANDARD/HIGH severity, 3yr/7yr (see table below). Privacy floor: `scope_assessment_notes` and `recurrence_prevention` are free-text fields capped at 500 chars — worker MUST reject payloads where these fields contain strings matching `/@|\buser_id\b|\bhealth\b/i` patterns (privacy floor lint). `art33_notification_id` is a DPA notification reference string (e.g. supervisory authority case number) — never the content of the notification. Closes SOC2_READINESS §77.10 P0 checklist item 1 and resolves OQ-PRIV-INC-02 (PI-P3 `incident_reviewed` optional).

| Event type | Severity (PI-P0/P1) | Severity (PI-P2/P3) | Retention | Trigger | PRIV-INC-CHAIN-01 |
|---|---|---|---|---|---|
| `privacy.incident_opened` | **CRITICAL** | STANDARD | 7yr (P0/P1) · 3yr (P2/P3) | First detection; before any investigation action | **Anchor** — must precede reviewed + closed for same `incident_id` |
| `privacy.incident_reviewed` | HIGH | HIGH | 7yr | Scope assessment complete; Art. 33 determination made | Required predecessor: `privacy.incident_opened` for same `incident_id` (HTTP 422 if absent); **optional for PI-P3** |
| `privacy.incident_closed` | HIGH | HIGH | 7yr | All remediation complete; PIR filed (PI-P0/P1) | Required predecessor: `privacy.incident_opened` for same `incident_id` (HTTP 422 if absent) |

```typescript
// Zod schemas — canonical source for emit-audit-event Worker validation
// Defined in docs/SOC2_READINESS.md §77.4
import { z } from 'zod';

const PrivacyIncidentOpenedSchema = z.object({
  incident_id:          z.string().uuid(),
  severity_class:       z.enum(['PI-P0', 'PI-P1', 'PI-P2', 'PI-P3']),
  incident_type:        z.enum([
    'art9_confirmed',
    'art9_suspected',
    'contact_data',
    'privacy_floor_breach',
    'dsar_delay',
    'consent_failure',
    'control_gap',
    'sub_processor',
    'process_deviation',
    'government_request',
  ]),
  detection_source:     z.enum([
    'pagerduty_alert',
    'dsar_review',
    'annual_review',
    'pentest_finding',
    'internal_report',
    'external_report',
    'sub_processor_notice',
    'regulatory_inquiry',
  ]),
  tenant_id:            z.string().uuid().optional(),   // null for consumer incidents
  art9_suspected:       z.boolean(),
  art33_clock_started:  z.boolean(),
  linear_ticket_url:    z.string().url().optional(),
  opened_by:            z.string().uuid(),
});

const PrivacyIncidentReviewedSchema = z.object({
  incident_id:                z.string().uuid(),
  final_severity_class:       z.enum(['PI-P0', 'PI-P1', 'PI-P2', 'PI-P3']),
  severity_changed:           z.boolean(),
  art33_required:             z.boolean(),
  art34_required:             z.boolean(),
  runbook_invoked:            z.array(z.enum([
    'R-11', 'R-12', 'R-13', 'R-14', 'R-22', 'section_15', 'R-07', 'none',
  ])),
  external_counsel_engaged:   z.boolean(),
  scope_assessment_notes:     z.string().max(500),      // no health values, no user PII
  reviewed_by:                z.string().uuid(),
});

const PrivacyIncidentClosedSchema = z.object({
  incident_id:                z.string().uuid(),
  resolution_type:            z.enum([
    'remediated_no_notification',
    'art33_notification_filed',
    'art34_notifications_sent',
    'false_positive',
    'process_improvement',
  ]),
  root_cause_category:        z.enum([
    'configuration_error',
    'code_defect',
    'human_error',
    'sub_processor_failure',
    'policy_gap',
    'unknown',
  ]),
  recurrence_prevention:      z.string().max(500),      // no health values, no user PII
  post_incident_review_filed: z.boolean(),
  closed_by:                  z.string().uuid(),
  art33_notification_id:      z.string().optional(),    // supervisory authority case ref
});
```

**PRIV-INC-CHAIN-01 ordering invariant enforcement:**

The `emit-audit-event` Worker enforces the following:
- `privacy.incident_reviewed` emitted with no prior `privacy.incident_opened` for same `incident_id` → HTTP 422 (`PRIV_INC_CHAIN_01_VIOLATION`); blocking.
- `privacy.incident_closed` emitted with no prior `privacy.incident_opened` for same `incident_id` → HTTP 422 (`PRIV_INC_CHAIN_01_VIOLATION`); blocking.
- `privacy.incident_reviewed` for PI-P3 incidents is **not enforced** — the Worker does NOT require a prior `incident_reviewed` before accepting `incident_closed` for `severity_class = 'PI-P3'`. PI-P3 process deviations may resolve in a single engineering fix without a formal scope assessment. Both `incident_reviewed` and `incident_closed` must still have a preceding `incident_opened`. This resolves OQ-PRIV-INC-02 from `docs/SOC2_READINESS.md §77.11` (P1, M8).

**PagerDuty routing:**

| Severity class | PagerDuty service | Recipients | Dedup |
|---|---|---|---|
| PI-P0 or `art9_suspected = true` | `form-privacy` CRITICAL | compliance-officer + security-engineer + founder (simultaneous) | No dedup — every PI-P0 `incident_opened` pages immediately |
| PI-P1 | `form-privacy` HIGH | compliance-officer + security-engineer | 15-min dedup window |
| PI-P2 | Slack `#privacy-incidents` | compliance-officer | No PagerDuty |
| PI-P3 | Linear ticket only | compliance-officer | No PagerDuty |

**Evidence artefacts (from SOC2_READINESS §77.7):**

| ID | Query | Filing path | SOC 2 | Retention |
|---|---|---|---|---|
| PRIV-PI-E-001 | `SELECT … FROM audit_log_events WHERE action = 'privacy.incident_opened' AND created_at BETWEEN $obs_start AND $obs_end` | `compliance/evidence/privacy/incident-register/priv-pi-e-001-YYYY-QN.jsonl` | P8.2, P8.3 | 7yr |
| PRIV-PI-E-002 | Monthly aggregate count by severity class (no individual incident details) | `compliance/evidence/privacy/incident-register/priv-pi-e-002-YYYY-QN.json` | P8.2 | 7yr |
| PRIV-PI-E-003 | PI-P0/P1 close confirmation — `incident_opened` JOIN `incident_closed` with `post_incident_review_filed = true` | `compliance/evidence/privacy/incident-register/priv-pi-e-003-YYYY-QN.json` | P8.2, P8.3 | 7yr |
| PRIV-PI-E-004 | `privacy.annual_review_completed` event for calendar year (from §60 evidence package) | `compliance/evidence/privacy/YYYY/p-e-008.json` | P8.1, P8.2 | 7yr |

**Emitter assignments:**

- `privacy.incident_opened` — compliance-officer (manual, within 30 min of P0/P1 awareness; 4h for P2/P3). Security-engineer may also emit for detection-triggered P0/P1 when compliance-officer is unavailable. No automated emission path — human judgement required to classify severity.
- `privacy.incident_reviewed` — compliance-officer only. `form_system` INSERT privilege reserved for break-glass retroactive filing with compliance-officer co-approval.
- `privacy.incident_closed` — compliance-officer only. `post_incident_review_filed: true` is self-attestation by the emitter — the PIR document itself must be filed separately at `compliance/evidence/privacy/incident-register/pir-{incident_id}.md`.

---

### CI/CD Pipeline events (DEC-030 HMAC-chained · OBSERVABILITY §38 · CC8.1)

> Defined in `docs/OBSERVABILITY.md §38`. Five events covering the full CI/CD deployment lifecycle — production deploys, database migrations, deployment rollbacks, and pipeline failures. **Privacy floor (all five events):** no `user_id`, no `tenant_id`, no health or coaching data in any payload. All actor fields reference internal FORM engineering identities (`rollback_initiated_by` is a `form_admin` UUID — never a customer user_id). `error_message_hash` in `ci.migration_failed` and `ci.deployment_rolled_back` uses SHA-256 to prevent raw SQL or stderr content from entering the chain. **CC8.1 note:** `ci.migration_applied` with `rls_policy_changed: true` is the primary SOC 2 evidence artefact that every RLS-touching migration was deployed through the CI pipeline (not manually); this is the observability companion to the adversarial RLS CI gate in `docs/DATA_MODEL.md §7`. **Ordering invariant (CI-CHAIN-01):** `ci.migration_applied` must follow the most recent `system.deployment_completed` for the same `commit_sha`; `ci.deployment_rolled_back.failing_commit_sha` must reference a prior `system.deployment_completed` in the chain — orphaned rollback events are flagged by the weekly chain audit batch. Cross-ref: `docs/SOC2_READINESS.md §21` (CC8.1 Change Management Controls — §38 is the observability layer); `docs/INCIDENT_RESPONSE.md R-02` (production outage rollback runbook — AL-CI-02 triggers R-02); `docs/ENTERPRISE_SLA.md §4.3` (RTO — CI-SLO-04 MTTR is the deployment sub-component). Closes OBSERVABILITY.md §38.10 checklist item 1 (P0, M7).

| Event type | Severity | Retention | Trigger | Key payload fields |
|---|---|---|---|---|
| `system.deployment_completed` | STANDARD | 5 yr | GitHub Actions Wrangler deploy succeeds + S-001 smoke probe passes | `environment` (enum), `surface` (enum), `commit_sha` (40 chars), `branch`, `github_run_id`, `duration_ms`, `smoke_probe_passed` (bool), `deployed_at` |
| `ci.migration_applied` | HIGH | 7 yr | `supabase db push` exits 0 in production-targeting workflow | `environment`, `migration_file` (regex), `migration_sha256` (64 chars), `commit_sha`, `tables_affected[]`, `rls_policy_changed` (bool — flags manual `security-engineer` review), `applied_at` |
| `ci.migration_failed` | **CRITICAL** | 7 yr | `supabase db push` exits non-zero in production workflow | `environment`, `migration_file`, `migration_sha256`, `commit_sha`, `error_code`, `error_message_hash` (SHA-256 — no raw SQL in chain), `partial_apply` (bool), `rollback_attempted` (bool), `failed_at` |
| `ci.deployment_rolled_back` | HIGH | 7 yr | On-call engineer executes rollback per R-02 runbook | `environment`, `surface`, `rolled_back_to_commit_sha`, `failing_commit_sha`, `trigger` (enum), `rollback_initiated_by` (UUID — `form_admin`), `minutes_since_deploy`, `incident_ticket_url` (optional), `rolled_back_at` |
| `ci.pipeline_failed` | STANDARD | 3 yr | Required CI job on main branch exits non-zero | `branch`, `is_main` (bool — drives AL-CI-01), `job_name` (enum), `conclusion` (enum), `commit_sha`, `github_run_id`, `duration_ms`, `failed_at` |

```typescript
import { z } from 'zod';

// Canonical Zod schemas — source of truth for emit-audit-event Worker validation
// Full schemas defined in docs/OBSERVABILITY.md §38.6

const DeploymentCompletedSchema = z.object({
  environment:         z.enum(['production', 'staging', 'preview']),
  surface:             z.enum(['worker', 'pages', 'eas_mobile', 'edge_function']),
  commit_sha:          z.string().length(40),
  branch:              z.string().max(100),
  github_run_id:       z.string(),
  wrangler_version:    z.string().optional(),
  eas_build_id:        z.string().optional(),
  duration_ms:         z.number().int().positive(),
  smoke_probe_passed:  z.boolean(),
  deployed_at:         z.string().datetime(),
});

const MigrationAppliedSchema = z.object({
  environment:          z.enum(['production', 'staging']),
  migration_file:       z.string().regex(/^\d{14}_[\w]+\.sql$/),
  migration_sha256:     z.string().length(64),
  commit_sha:           z.string().length(40),
  github_run_id:        z.string(),
  tables_affected:      z.array(z.string()).max(20),
  rls_policy_changed:   z.boolean(),
  applied_at:           z.string().datetime(),
  supabase_project_ref: z.string(),
});

const MigrationFailedSchema = z.object({
  environment:          z.enum(['production', 'staging']),
  migration_file:       z.string().regex(/^\d{14}_[\w]+\.sql$/),
  migration_sha256:     z.string().length(64),
  commit_sha:           z.string().length(40),
  github_run_id:        z.string(),
  error_code:           z.string().max(50),
  error_message_hash:   z.string().length(64),
  partial_apply:        z.boolean(),
  rollback_attempted:   z.boolean(),
  failed_at:            z.string().datetime(),
});

const DeploymentRolledBackSchema = z.object({
  environment:               z.enum(['production', 'staging']),
  surface:                   z.enum(['worker', 'pages', 'eas_mobile', 'edge_function']),
  rolled_back_to_commit_sha: z.string().length(40),
  failing_commit_sha:        z.string().length(40),
  trigger:                   z.enum(['smoke_test_failure', 'slo_breach', 'manual_oncall', 'post_incident']),
  rollback_initiated_by:     z.string().uuid(),
  minutes_since_deploy:      z.number().int().nonneg(),
  incident_ticket_url:       z.string().url().optional(),
  rolled_back_at:            z.string().datetime(),
});

const PipelineFailedSchema = z.object({
  branch:         z.string().max(100),
  is_main:        z.boolean(),
  job_name:       z.enum(['test', 'lint', 'build', 'migration', 'deploy-worker', 'eas-build']),
  conclusion:     z.enum(['failure', 'cancelled']),
  commit_sha:     z.string().length(40),
  github_run_id:  z.string(),
  duration_ms:    z.number().int().nonneg(),
  failed_at:      z.string().datetime(),
});
```

**HMAC chain ordering (CI-CHAIN-01):**
- `ci.migration_applied` must follow the most recent `system.deployment_completed` for the same `commit_sha`. Orphaned migration events (no preceding deploy) are flagged by the weekly chain audit batch.
- `ci.deployment_rolled_back.failing_commit_sha` must reference a prior `system.deployment_completed` in the chain. Orphaned rollback events trigger a chain anomaly alert.
- `ci.migration_failed` is CRITICAL severity: no deduplication, no auto-resolve. Every emission pages PagerDuty AL-CI-02 until manually closed.

**Emitter assignments:** `devops-lead` role via automated GitHub Actions steps, except `ci.deployment_rolled_back` which is emitted by the on-call engineer via admin console during R-02 runbook execution.

---

### Backup & DR Observability events (DEC-030 HMAC-chained · OBSERVABILITY §39 · A1.2/A1.3)

> Defined in `docs/OBSERVABILITY.md §39`. Six events covering backup health and disaster recovery test lifecycle — daily backup completions, backup failures, and quarterly restore tests. **Privacy floor (all six events):** `user_id = NULL`, `tenant_id = NULL` — infrastructure-only events. No user identifiers, tenant data, or health data appear in any payload. `error_message_hash` in `system.backup_failed` and `system.restore_test_failed` uses SHA-256 to prevent raw error output from entering the chain. **`privacy_floor_verified` invariant:** `system.restore_test_completed` MUST have `privacy_floor_verified: true` to count as a successful SOC 2 A1.3 evidence artefact — a restore with `privacy_floor_verified: false` is treated as a failed test regardless of `success` field value. **BC-CHAIN-01 ordering invariant:** `system.restore_test_completed` or `system.restore_test_failed` MUST have a corresponding `system.restore_test_initiated` with matching `test_id` earlier in the chain; the `emit-audit-event` Worker returns HTTP 422 on violation. Cross-ref: `docs/BUSINESS_CONTINUITY.md §3.1` (RTO/RPO commitments — §39 is the observability layer); `docs/ENTERPRISE_SLA.md §4.3` (RTO = 4h — `rto_target_minutes: 240` is the literal contract commitment in the chain); `docs/OBSERVABILITY.md §39.10` (SOC 2 evidence artefacts BC-E-001 through BC-E-005). Closes OBSERVABILITY.md §39.11 checklist item 1 (P0, M13 enterprise GA).

| Event type | Severity | Retention | Trigger | Key payload fields |
|---|---|---|---|---|
| `system.backup_completed` | STANDARD | 5 yr | FORM-managed backup Worker completes daily encrypted dump | `store` (enum), `backup_type` (enum), `initiated_at`, `completed_at`, `duration_minutes`, `size_bytes_estimate` (optional), `backup_id`, `region` (enum), `triggered_by` (enum) |
| `system.backup_failed` | **CRITICAL** | 7 yr | Backup Worker exits non-zero or R2/B2 write fails | `store`, `backup_type`, `initiated_at`, `failed_at`, `error_code`, `error_message_hash` (SHA-256), `triggered_by` |
| `system.restore_test_initiated` | STANDARD | 7 yr | devops-lead begins quarterly DR drill on staging | `test_id` (UUID), `store`, `restore_type` (enum), `initiated_by` (PAM session ID), `environment: 'staging'`, `backup_id`, `rationale` (enum) |
| `system.restore_test_completed` | STANDARD | 7 yr | devops-lead verifies restored DB passes checks | `test_id`, `store`, `initiated_at`, `completed_at`, `rto_achieved_minutes`, `rto_target_minutes: 240`, `success` (bool), `privacy_floor_verified` (bool — **must be true for SOC 2 A1.3 evidence**), `rpo_gap_minutes`, `verified_by` (PAM session ID) |
| `system.restore_test_failed` | **CRITICAL** | 7 yr | Restore exceeds RTO, privacy floor check fails, or backup corrupted | `test_id`, `store`, `initiated_at`, `failed_at`, `failure_reason` (enum — includes `privacy_floor_violation`), `error_hash` (SHA-256) |
| `system.backup_staleness_detected` | HIGH | 7 yr | pg_cron job 29 detects `backup_age_hours > 26` for any store | `store`, `last_completed_at`, `age_hours`, `threshold_hours: 26`, `detected_by: 'pg_cron_job_29'`, `alert_fired` (enum), `dedup_key` |

```typescript
import { z } from 'zod';

// Canonical Zod schemas — source of truth for emit-audit-event Worker validation
// Full schemas defined in docs/OBSERVABILITY.md §39.9

const BackupCompletedSchema = z.object({
  store:               z.enum(['postgres', 'r2_primary', 'r2_dr']),
  backup_type:         z.enum(['logical', 'wal_snapshot', 'pitr_checkpoint']),
  initiated_at:        z.string().datetime(),
  completed_at:        z.string().datetime(),
  duration_minutes:    z.number().nonneg(),
  size_bytes_estimate: z.number().nonneg().optional(),
  backup_id:           z.string(),
  region:              z.enum(['us-east-1', 'eu-central-1', 'eu-west-1']),
  pitr_point_in_time:  z.string().datetime().optional(),
  triggered_by:        z.enum(['scheduled', 'manual', 'pre_maintenance']),
});

const BackupFailedSchema = z.object({
  store:              z.enum(['postgres', 'r2_primary', 'r2_dr']),
  backup_type:        z.enum(['logical', 'wal_snapshot', 'pitr_checkpoint']),
  initiated_at:       z.string().datetime(),
  failed_at:          z.string().datetime(),
  error_code:         z.string(),
  error_message_hash: z.string(),
  triggered_by:       z.enum(['scheduled', 'manual', 'pre_maintenance']),
});

const RestoreTestInitiatedSchema = z.object({
  test_id:          z.string().uuid(),
  store:            z.enum(['postgres', 'r2_primary', 'r2_dr']),
  restore_type:     z.enum(['pitr', 'logical_dump', 'point_in_time']),
  initiated_by:     z.string().uuid(),
  environment:      z.literal('staging'),
  target_timestamp: z.string().datetime().optional(),
  backup_id:        z.string(),
  rationale:        z.enum([
    'quarterly_dr_drill',
    'annual_dr_drill',
    'ad_hoc_verification',
    'pre_production_migration',
  ]),
});

const RestoreTestCompletedSchema = z.object({
  test_id:                z.string().uuid(),
  store:                  z.enum(['postgres', 'r2_primary', 'r2_dr']),
  initiated_at:           z.string().datetime(),
  completed_at:           z.string().datetime(),
  rto_achieved_minutes:   z.number().nonneg(),
  rto_target_minutes:     z.literal(240),
  success:                z.boolean(),
  row_count_spot_check:   z.number().int().nonneg(),
  privacy_floor_verified: z.boolean(),
  rpo_gap_minutes:        z.number().nonneg(),
  environment:            z.literal('staging'),
  verified_by:            z.string().uuid(),
});

const RestoreTestFailedSchema = z.object({
  test_id:        z.string().uuid(),
  store:          z.enum(['postgres', 'r2_primary', 'r2_dr']),
  initiated_at:   z.string().datetime(),
  failed_at:      z.string().datetime(),
  failure_reason: z.enum([
    'backup_corrupted',
    'pitr_gap_detected',
    'rto_exceeded',
    'privacy_floor_violation',
    'row_count_mismatch',
    'connection_timeout',
    'manual_abort',
  ]),
  error_hash:     z.string(),
  environment:    z.literal('staging'),
});

const BackupStalenessDetectedSchema = z.object({
  store:             z.enum(['postgres', 'r2_primary', 'r2_dr']),
  last_completed_at: z.string().datetime(),
  age_hours:         z.number().nonneg(),
  threshold_hours:   z.literal(26),
  detected_by:       z.literal('pg_cron_job_29'),
  alert_fired:       z.enum(['AL-BC-01', 'AL-BC-02', 'AL-BC-03']),
  dedup_key:         z.string(),
});
```

**HMAC chain ordering (BC-CHAIN-01):** `system.restore_test_completed` or `system.restore_test_failed` MUST follow a `system.restore_test_initiated` with the same `test_id`. The `emit-audit-event` Worker returns HTTP 422 on violation — a completion event without a matching initiation event is rejected at write time, not flagged post-hoc.

**`failure_reason: 'privacy_floor_violation'`** in `system.restore_test_failed`: additionally notifies `clinical-safety` via PagerDuty and triggers the RLS-invariant investigation procedure in `docs/INCIDENT_RESPONSE.md`. A restore test that fails the privacy floor check is classified as a P0 infrastructure incident, not a routine test failure.

**Emitter assignments:** `system.backup_completed` and `system.backup_failed` are emitted by the FORM-managed backup Worker (automated, `form_system` role). `system.restore_test_*` are emitted manually by `devops-lead` via admin console during DR drill execution — no automated path (restore tests require human judgment and verification). `system.backup_staleness_detected` is emitted by pg_cron job 29 (`form_system` role, scheduled every 4h).

---

### Enterprise SLA events (DEC-030 HMAC-chained · OBSERVABILITY §23 · A1.1 / CC7.2)

> Defined in `docs/OBSERVABILITY.md §23`. Ten events covering the full SLA incident, measurement, credit, and dispute lifecycle for enterprise tenants. Closes OBSERVABILITY.md §23.11 checklist item 8 (P0, M4 — "Register all 10 `sla.*` DEC-030 event types in `AUDIT_LOG_SCHEMA.md` event registry").
>
> **Privacy floor (all ten events):** `actor_id` is a FORM internal identity (`devops-lead` UUID, `form_system`, or `compliance-officer` UUID). Tenant is identified via `tenant_id` only — no `user_id`, no employee PII, no health values. `sla.dispute_opened` captures `dispute_reason` (free text); this field must not include employee identifiers — the platform-layer `form_system` emitter truncates and hashes any value containing `@` characters before writing to chain. **Financial data:** `credit_amount_usd` and `final_credit_usd` fields are FORM–tenant contractual records; they appear in the chain under tenant scope and are protected by the same RLS tenant-isolation policy as `sla_monthly_reports` (§23.8 — tenant_admin sees only their own tenant's records).
>
> **SLA-CHAIN ordering invariants** (enforced at write time by `emit-audit-event` Worker — HTTP 422 on violation):
> - **SLA-CHAIN-01:** `sla.incident_closed` MUST follow a `sla.incident_opened` with the same `incident_id` in the chain. An orphaned close event (no matching open) is rejected.
> - **SLA-CHAIN-02:** `sla.credit_approved` MUST follow a `sla.credit_calculated` for the same `(tenant_id, report_month)`. Credit approval without a calculation record is rejected.
> - **SLA-CHAIN-03:** `sla.dispute_resolved` MUST follow a `sla.dispute_opened` for the same `(tenant_id, report_month)`. A resolved dispute without a corresponding open is rejected.
>
> A chain break on any `sla.*` event is a P0 incident per `docs/INCIDENT_RESPONSE.md §R-05`.
>
> Cross-ref: `docs/OBSERVABILITY.md §23.9` (canonical event table); `docs/ENTERPRISE_SLA.md` (committed SLA terms — these events are the contractual audit trail); `docs/SOC2_READINESS.md §2 A1.1` (gap closure — SLA credit calculation and process, OBSERVABILITY §23 is the primary evidence section).

| Event type | Severity | Retention | Trigger | Key payload fields |
|---|---|---|---|---|
| `sla.incident_opened` | HIGH | 7 yr | Probes detect covered downtime (month-close Worker) or devops-lead manually declares P0 outage | `tenant_id`, `incident_id` (UUID), `probe_ids` (string[]), `outage_type` (`full`\|`partial`), `downtime_weight` (1.0 or 0.5), `started_at` |
| `sla.incident_closed` | HIGH | 7 yr | Probes recover; `sla_incidents.ended_at` written | `tenant_id`, `incident_id`, `ended_at`, `duration_minutes`, `exclusion_applied` (bool), `exclusion_reason` (string\|null) |
| `sla.measurement_reconciled` | STANDARD | 7 yr | Month-end reconciliation Worker compares Better Stack vs. Cloudflare Analytics; conservative source chosen per §23.3 | `tenant_id`, `report_month` (ISO date), `betterstack_pct`, `cf_analytics_pct`, `delta_pct`, `chosen_source` (`betterstack`\|`cf_analytics`) |
| `sla.credit_calculated` | HIGH | 7 yr | `workers/sla/month-close.ts` determines credit tier for the reporting month | `tenant_id`, `report_month`, `availability_pct`, `covered_downtime_minutes`, `credit_tier_pct` (0\|5\|15\|25\|50), `credit_amount_usd`, `mrr_snapshot_usd` |
| `sla.credit_approved` | HIGH | 7 yr | `compliance-officer` approves credit after verifying reconciliation; applies credit to next invoice | `tenant_id`, `report_month`, `approved_by` (UUID — compliance-officer), `final_credit_usd`, `original_calculated_usd` |
| `sla.credit_adjusted` | HIGH | 7 yr | `compliance-officer` modifies credit from calculated amount (upward or downward, e.g. dispute upheld) | `tenant_id`, `report_month`, `original_usd`, `adjusted_usd`, `adjustment_delta_usd`, `reason` (string — plain-text rationale), `adjusted_by` (UUID) |
| `sla.dispute_opened` | STANDARD | 3 yr | Tenant submits dispute via admin dashboard "Dispute this report" flow; routes to `enterprise@form.coach` | `tenant_id`, `report_month`, `dispute_reason` (hashed if contains `@`), `submitted_via` (`admin_dashboard`\|`email`) |
| `sla.dispute_resolved` | HIGH | 7 yr | `compliance-officer` closes dispute — outcome `upheld` (credit increased) or `rejected` (original credit stands) | `tenant_id`, `report_month`, `outcome` (`upheld`\|`rejected`), `final_credit_usd`, `resolved_by` (UUID), `resolution_notes_hash` (SHA-256 — no raw text in chain) |
| `sla.maintenance_window_registered` | STANDARD | 3 yr | `devops-lead` inserts row into `maintenance_windows` table before window begins; required for §23.4 scheduled-maintenance exclusion | `tenant_id` (UUID\|null — null for platform-wide), `window_id` (UUID), `scheduled_start`, `scheduled_end`, `notification_hours`, `approved_by` (UUID) |
| `sla.exclusion_reclassified` | HIGH | 7 yr | `security-engineer` reclassifies a false-positive probe failure per §23.4; requires second approver for probe windows > 30 min | `tenant_id`, `incident_id`, `original_classification` (`covered`\|`partial`), `new_classification` (`excluded`), `exclusion_reason` (enum), `approved_by` (UUID — security-engineer), `reclassified_at` |

```typescript
import { z } from 'zod';

// sla.* Zod schemas — source of truth for emit-audit-event Worker validation
// Canonical definitions: docs/OBSERVABILITY.md §23.9
// All schemas share the standard DEC-030 envelope: { tenant_id, trace_id, actor_id, hmac_prev, hmac_self, created_at }

const SlaIncidentOpenedSchema = z.object({
  incident_id:     z.string().uuid(),
  probe_ids:       z.array(z.string()),
  outage_type:     z.enum(['full', 'partial']),
  downtime_weight: z.number().refine(w => w === 1.0 || w === 0.5),
  started_at:      z.string().datetime(),
});

const SlaIncidentClosedSchema = z.object({
  incident_id:        z.string().uuid(),
  ended_at:           z.string().datetime(),
  duration_minutes:   z.number().nonnegative(),
  exclusion_applied:  z.boolean(),
  exclusion_reason:   z.string().nullable(),
});

const SlaMeasurementReconciledSchema = z.object({
  report_month:    z.string().regex(/^\d{4}-\d{2}-01$/),  // e.g. "2026-05-01"
  betterstack_pct: z.number().min(0).max(100),
  cf_analytics_pct:z.number().min(0).max(100),
  delta_pct:       z.number().nonnegative(),
  chosen_source:   z.enum(['betterstack', 'cf_analytics']),
});

const SlaCreditCalculatedSchema = z.object({
  report_month:             z.string().regex(/^\d{4}-\d{2}-01$/),
  availability_pct:         z.number().min(0).max(100),
  covered_downtime_minutes: z.number().nonnegative(),
  credit_tier_pct:          z.union([z.literal(0), z.literal(5), z.literal(15), z.literal(25), z.literal(50)]),
  credit_amount_usd:        z.number().nonnegative(),
  mrr_snapshot_usd:         z.number().positive(),
});

const SlaCreditApprovedSchema = z.object({
  report_month:           z.string().regex(/^\d{4}-\d{2}-01$/),
  approved_by:            z.string().uuid(),
  final_credit_usd:       z.number().nonnegative(),
  original_calculated_usd:z.number().nonnegative(),
});

const SlaCreditAdjustedSchema = z.object({
  report_month:       z.string().regex(/^\d{4}-\d{2}-01$/),
  original_usd:       z.number().nonnegative(),
  adjusted_usd:       z.number().nonnegative(),
  adjustment_delta_usd:z.number(),
  reason:             z.string().min(10).max(500),
  adjusted_by:        z.string().uuid(),
});

const SlaDisputeOpenedSchema = z.object({
  report_month:   z.string().regex(/^\d{4}-\d{2}-01$/),
  dispute_reason: z.string().max(1000),  // hashed at write layer if contains '@'
  submitted_via:  z.enum(['admin_dashboard', 'email']),
});

const SlaDisputeResolvedSchema = z.object({
  report_month:          z.string().regex(/^\d{4}-\d{2}-01$/),
  outcome:               z.enum(['upheld', 'rejected']),
  final_credit_usd:      z.number().nonnegative(),
  resolved_by:           z.string().uuid(),
  resolution_notes_hash: z.string().length(64),  // SHA-256 hex — no raw text in chain
});

const SlaMaintenanceWindowRegisteredSchema = z.object({
  window_id:         z.string().uuid(),
  scheduled_start:   z.string().datetime(),
  scheduled_end:     z.string().datetime(),
  notification_hours:z.number().nonnegative(),
  approved_by:       z.string().uuid(),
  // tenant_id is null for platform-wide windows (present in DEC-030 envelope as null)
});

const SlaExclusionReclassifiedSchema = z.object({
  incident_id:           z.string().uuid(),
  original_classification:z.enum(['covered', 'partial']),
  new_classification:    z.literal('excluded'),
  exclusion_reason:      z.enum([
    'upstream_provider_outage',
    'scheduled_maintenance',
    'customer_idp_misconfiguration',
    'force_majeure',
    'synthetic_probe_false_positive',
    'beta_feature',
  ]),
  approved_by:        z.string().uuid(),
  reclassified_at:    z.string().datetime(),
});
```

**Emitter assignments:**
- `sla.incident_opened` / `sla.incident_closed`: `workers/sla/month-close.ts` (`form_system` role) on automated probe detection; `devops-lead` manually via admin console for P0 declarations.
- `sla.measurement_reconciled` / `sla.credit_calculated`: `workers/sla/month-close.ts` (`form_system` role), triggered by pg_cron at 00:30 UTC on the 1st of each month.
- `sla.credit_approved` / `sla.credit_adjusted` / `sla.dispute_resolved`: `compliance-officer` manually via admin console after verification — no automated path.
- `sla.dispute_opened`: `form_system` role on receipt of tenant dispute submission through admin dashboard.
- `sla.maintenance_window_registered`: `devops-lead` via admin console; `form_system` validates the `notification_hours ≥ 72` constraint (≥ 168 for Enterprise tier) before emitting.
- `sla.exclusion_reclassified`: `security-engineer` via admin console; a second approver (`compliance-officer`) is required for any reclassification that affects a window > 30 min (validated by the Worker before chain write).

---

### Incident Lifecycle events (DEC-030 HMAC-chained · INCIDENT_RESPONSE §16 · CC7.3/CC7.4/CC7.5)

> Defined in `docs/INCIDENT_RESPONSE.md §16.2`. Ten events forming the cross-cutting incident audit trail — distinct from domain-specific runbook events (breach.\*, legal.\*, dsar.\*) which record the *what*; these record the *when* and *who* of every incident management state transition. Required for CC7.4 (incident response activities documented and communicated) and CC7.5 (post-incident review). **Privacy floor:** no end-user PII (`user_id`, health values, coaching content, employee names) in any payload — `actor_user_id` is a FORM internal team UUID (IC, devops-lead, compliance-officer) never a tenant employee ID. `incident.update_posted.audience_scope` must be set by the IC — `privacy_floor_verified: true` is a required boolean that the IC attests before the event is accepted. **IRCHAIN-01 ordering invariant:** `incident.opened` must precede all other `incident.*` events for the same `incident_id`; the `emit-audit-event` Worker returns HTTP 422 on violation. **Sub-chain rule:** All `incident.*` events for a given `incident_id` form a sub-chain; a break in this sub-chain is P0 per R-05. Cross-ref: `docs/INCIDENT_RESPONSE.md §16.3` (TypeScript schemas), `§16.4` (SIEM-triggered automation), `§16.5` (IR-SLO meta-monitoring); SOC 2 CC7.3/CC7.4/CC7.5/CC4.1/CC2.2. Closes INCIDENT_RESPONSE.md §16.8 checklist items 1–2 (P0, M4).

| Event type | Severity | Retention | Trigger | Key payload fields |
|---|---|---|---|---|
| `incident.opened` | HIGH | 7 yr | IC declares incident or SIEM AL-\* rule auto-triggers (siem-incident-automator Worker) | `incident_id` (UUID), `title` (max 100 chars), `incident_severity` (P0–P3), `incident_runbook` (R-NN), `trigger_alert_id` (nullable), `trigger_correlation_rule` (nullable), `siem_correlation_score` (0–1, nullable), `source` (ic_declared\|siem_auto) |
| `incident.ic_assigned` | MEDIUM | 7 yr | First confirmed IC takes command; within P0 15 min / P1 30 min SLA (IR-SLO-01) | `incident_id`, `ic_user_id`, `ic_role`, `minutes_since_open` (float — IR-SLO-01 evidence) |
| `incident.severity_changed` | HIGH | 7 yr | Any severity upgrade OR IC-approved downgrade | `incident_id`, `previous_severity`, `new_severity`, `direction` (upgrade\|downgrade), `reason` (free text, max 500 chars), `approver_user_id` (required for downgrade) |
| `incident.escalated` | HIGH | 7 yr | Founder, board, external counsel, DPA, or enterprise tenant escalated | `incident_id`, `escalation_target` (founder\|board\|legal_counsel\|dpa\|enterprise_tenant), `reason`, `template_used` (nullable) |
| `incident.update_posted` | STANDARD | 3 yr | Status page update or enterprise tenant notification sent | `incident_id`, `channel` (status_page\|enterprise_tenant_email\|board_email\|internal_slack), `audience_scope` (public\|enterprise_tenants_affected\|all_enterprise\|internal), `privacy_floor_verified` (bool — **must be true**) |
| `incident.containment_verified` | HIGH | 7 yr | IC confirms active threat vector closed | `incident_id`, `containment_actions` (string[]), `data_exfiltration_confirmed` (bool), `hmac_chain_integrity` (bool) |
| `incident.eradicated` | HIGH | 7 yr | Root cause removed from production | `incident_id`, `root_cause_category` (config_error\|code_defect\|credential_compromise\|third_party_breach\|insider_threat\|hardware_failure\|unknown) |
| `incident.recovered` | HIGH | 7 yr | All services restored; SLAs re-met | `incident_id`, `recovery_actions` (string[]), `duration_minutes` (float) |
| `incident.pir_opened` | MEDIUM | 7 yr | PIR meeting scheduled; Linear PIR project created; within P0 24h / P1 72h of recovery (IR-SLO-02 clock starts) | `incident_id`, `pir_linear_url`, `pir_due_at` (ISO 8601) |
| `incident.pir_closed` | HIGH | 7 yr | PIR complete; all action items logged; document signed off (IR-SLO-03: P0 ≤7 days, P1 ≤14 days) | `incident_id`, `critical_action_items_count` (int), `pir_document_sha256`, `signed_off_by_user_id` |

```typescript
import { z } from 'zod';

const IncidentSeverity = z.enum(['P0', 'P1', 'P2', 'P3']);

// Common DEC-030 envelope fields omitted (tenant_id, trace_id, actor_id, hmac_prev, hmac_self, created_at)
// Privacy invariant: actor_user_id is always a FORM internal team UUID, never a tenant employee UUID.

const IncidentOpenedSchema = z.object({
  incident_id:             z.string().uuid(),
  title:                   z.string().max(100),
  incident_severity:       IncidentSeverity,
  incident_runbook:        z.string().regex(/^R-\d+$/),
  trigger_alert_id:        z.string().nullable(),
  trigger_correlation_rule:z.string().nullable(),
  siem_correlation_score:  z.number().min(0).max(1).nullable(),
  source:                  z.enum(['ic_declared', 'siem_auto']),
});

const IncidentIcAssignedSchema = z.object({
  incident_id:      z.string().uuid(),
  ic_user_id:       z.string().uuid(),
  ic_role:          z.string(),
  minutes_since_open: z.number().nonnegative(),
});

const IncidentSeverityChangedSchema = z.object({
  incident_id:      z.string().uuid(),
  previous_severity:IncidentSeverity,
  new_severity:     IncidentSeverity,
  direction:        z.enum(['upgrade', 'downgrade']),
  reason:           z.string().max(500),
  approver_user_id: z.string().uuid().optional(), // required for downgrade — Worker validates
});

const IncidentUpdatePostedSchema = z.object({
  incident_id:          z.string().uuid(),
  channel:              z.enum(['status_page', 'enterprise_tenant_email', 'board_email', 'internal_slack']),
  audience_scope:       z.enum(['public', 'enterprise_tenants_affected', 'all_enterprise', 'internal']),
  privacy_floor_verified: z.literal(true),  // IC attestation — false rejected at write time
});

const IncidentRecoveredSchema = z.object({
  incident_id:       z.string().uuid(),
  recovery_actions:  z.array(z.string()),
  duration_minutes:  z.number().nonnegative(),
});

const IncidentPirClosedSchema = z.object({
  incident_id:               z.string().uuid(),
  critical_action_items_count: z.number().int().nonnegative(),
  pir_document_sha256:       z.string().length(64),
  signed_off_by_user_id:     z.string().uuid(),
});
```

**HMAC chain ordering (IRCHAIN-01):** `incident.opened` is the sub-chain anchor for its `incident_id`. All subsequent `incident.*` events for that `incident_id` must reference the prior event in the sub-chain via `prev_hash`. Orphaned events (no matching `opened`) are rejected HTTP 422. **Dead-man's switches:** pg_cron checks at T+48h after `incident.recovered` for a missing `incident.pir_opened` (fires Slack + Linear auto-ticket); checks PIR `pir_due_at` daily for overdue PIRs (fires Slack + PagerDuty low-urgency). Both are IR-SLO-02/03 SLA enforcement mechanisms.

**Emitter assignments:** `incident.opened` — siem-incident-automator Worker (`form_system`) for SIEM auto-triggers, or IC (manual admin console) for declared incidents. `incident.ic_assigned` / `incident.severity_changed` / `incident.escalated` / `incident.containment_verified` / `incident.eradicated` / `incident.recovered` / `incident.pir_opened` / `incident.pir_closed` — IC (manual admin console). `incident.update_posted` — IC (manual; `privacy_floor_verified` attestation required before emitting).

---

### Wearable Integration & Sync Pipeline events (DEC-030 HMAC-chained · OBSERVABILITY §41 · A1.1/P3.2)

> Defined in `docs/OBSERVABILITY.md §41.6`. Six events covering the wearable data ingestion pipeline for all five supported sources (HealthKit, Health Connect, Whoop, Oura, Garmin). **Privacy floor (all six events):** no `user_id` in any payload — `tenant_id` is nullable (null for consumer tier). `error_message_hash` uses SHA-256 to prevent raw API error messages (which may contain access tokens) from entering the chain. `wearable.permission_revoked` is the GDPR Art. 7(3) withdrawal record — linked to the consent chain via `consent_event_id` foreign key; no direct `user_id`. `wearable.fleet_freshness_assessed` is emitted by pg_cron job 31 with a k-anonymity gate: `sources_breakdown` object is suppressed for any source where N < 5 across the active fleet. **WS-CHAIN-01:** `wearable.stale_data_coaching_context` MUST have `coaching_context_downgraded: true` — a `false` value is a clinical-safety policy violation rejected at write time (HTTP 422). Cross-ref: `docs/OBSERVABILITY.md §41.3` (WS-SLO-01 through WS-SLO-06); `docs/DATA_MODEL.md §14` (wearable_readings schema — §41 is the observability companion); `docs/CLINICAL_SAFETY.md` (stale-HRV veto authority). Closes OBSERVABILITY.md §41.11 checklist item 1 (P0, M5).

| Event type | Severity | Retention | Trigger | Key payload fields |
|---|---|---|---|---|
| `wearable.sync_completed` | STANDARD | 2 yr | `wearable-ingestion-worker` completes a batch for one source | `source` (healthkit\|health_connect\|whoop\|oura\|garmin), `reading_count` (int), `oldest_reading_age_hours` (float), `tenant_id` (UUID\|null) |
| `wearable.sync_failed` | HIGH | 3 yr | Worker exits with non-2xx from source API or Supabase insert error | `source`, `error_class` (oauth_expired\|rate_limited\|api_unavailable\|schema_mismatch\|insert_error), `error_message_hash` (SHA-256), `retry_count` (int), `tenant_id` (UUID\|null) |
| `wearable.oauth_token_expired` | HIGH | 3 yr | OAuth 2.0 token for Whoop, Oura, or Garmin is expired and refresh failed (HealthKit/Health Connect use OS permissions — no OAuth) | `source` (whoop\|oura\|garmin), `refresh_attempted` (bool), `tenant_id` (UUID\|null) |
| `wearable.permission_revoked` | HIGH | 5 yr | User revokes wearable permission (OS prompt, user-initiated in app, or enterprise MDM policy change); GDPR Art. 7(3) withdrawal record | `source`, `revocation_type` (user_initiated\|os_prompt\|enterprise_mdm), `consent_event_id` (UUID — FK to GDPR consent chain; **no `user_id`**) |
| `wearable.fleet_freshness_assessed` | STANDARD | 2 yr | pg_cron job 31 (`wearable_sync_freshness_check`) runs daily 07:05 UTC | `fresh_pct` (float 0–100), `total_connected` (int), `slo_met` (bool — WS-SLO-06), `sources_breakdown` (object by source — **suppressed if any source N < 5**), `detected_by: 'pg_cron_job_31'` |
| `wearable.stale_data_coaching_context` | HIGH | 3 yr | Victor ingestion layer detects HRV reading > 48h old used as coaching context | `source`, `hrv_data_age_hours` (float), `coaching_context_downgraded` (bool — **must be true; false = HTTP 422**) |

```typescript
import { z } from 'zod';

const WearableSource = z.enum(['healthkit', 'health_connect', 'whoop', 'oura', 'garmin']);
const OAuthSource    = z.enum(['whoop', 'oura', 'garmin']); // HealthKit/HC use OS permissions

const WearableSyncCompletedSchema = z.object({
  source:                  WearableSource,
  reading_count:           z.number().int().nonnegative(),
  oldest_reading_age_hours:z.number().nonnegative(),
  tenant_id:               z.string().uuid().nullable(),
});

const WearableSyncFailedSchema = z.object({
  source:              WearableSource,
  error_class:         z.enum(['oauth_expired', 'rate_limited', 'api_unavailable', 'schema_mismatch', 'insert_error']),
  error_message_hash:  z.string().length(64),   // SHA-256 hex
  retry_count:         z.number().int().nonnegative(),
  tenant_id:           z.string().uuid().nullable(),
});

const WearableOAuthTokenExpiredSchema = z.object({
  source:             OAuthSource,
  refresh_attempted:  z.boolean(),
  tenant_id:          z.string().uuid().nullable(),
});

const WearablePermissionRevokedSchema = z.object({
  source:           WearableSource,
  revocation_type:  z.enum(['user_initiated', 'os_prompt', 'enterprise_mdm']),
  consent_event_id: z.string().uuid(),   // FK to consent chain — no user_id
});

const WearableFleetFreshnessAssessedSchema = z.object({
  fresh_pct:         z.number().min(0).max(100),
  total_connected:   z.number().int().nonnegative(),
  slo_met:           z.boolean(),
  sources_breakdown: z.record(z.number()).optional(), // omitted when any source N < 5
  detected_by:       z.literal('pg_cron_job_31'),
});

const WearableStaleDataCoachingContextSchema = z.object({
  source:                     WearableSource,
  hrv_data_age_hours:         z.number().positive(),
  coaching_context_downgraded:z.literal(true),  // false rejected HTTP 422 — WS-CHAIN-01
});
```

**HMAC chain requirement:** All six events are DEC-030 HMAC-chained. `wearable.oauth_token_expired` for a `source` should be followed by `wearable.sync_failed` (same source, `error_class: 'oauth_expired'`) within the same batch window — the weekly chain audit flags an `oauth_token_expired` with no subsequent `sync_failed` as a chain gap. `wearable.permission_revoked` has 5-year retention (longest in this namespace) because it serves as the GDPR Art. 7(3) withdrawal evidence record — never suppress or redact.

**Emitter assignments:** `wearable.sync_completed` and `wearable.sync_failed` — `wearable-ingestion-worker` (`form_system`, automated). `wearable.oauth_token_expired` — `wearable-ingestion-worker` on refresh failure. `wearable.permission_revoked` — OS permission layer or enterprise MDM webhook handler (`form_system`). `wearable.fleet_freshness_assessed` — pg_cron job 31 (`form_system`). `wearable.stale_data_coaching_context` — Victor ingestion layer (`form_system`); clinical-safety has VETO on any implementation change that weakens the 48h gate.

---

### Performance & Load Testing events (DEC-030 HMAC-chained · OBSERVABILITY §40 · A1.1/CC5.2/CC8.1)

> Defined in `docs/OBSERVABILITY.md §40.5`. Five events forming the pre-launch load test gate audit trail — every deployment that requires a load test gate is evidence under SOC 2 A1.1 (capacity commitments met before go-live) and CC5.2 (technology controls verified before deployment). **Privacy floor (all five events):** `environment: 'staging'` is a literal constraint — no production load test events may be emitted (Worker enforces). Synthetic `lt-` tenant IDs only; no real user data in any test payload. `system.load_test_gate_bypassed.bypass_reason_hash` uses SHA-256; plaintext reason lives in the associated Linear ticket only. **PERF-CHAIN-01 ordering invariant:** `system.load_test_completed` or `system.load_test_failed` MUST follow a `system.load_test_initiated` with the same `commit_sha`; inversion is a P1 per R-05. Cross-ref: `docs/OBSERVABILITY.md §40.3` (PERF-SLO-01 through PERF-SLO-06); `docs/SOC2_READINESS.md §2` (A1.1 gap: load testing before launches). Closes OBSERVABILITY.md §40.10 checklist item 1 (P0, M5).

| Event type | Severity | Retention | Trigger | Key payload fields |
|---|---|---|---|---|
| `system.load_test_initiated` | STANDARD | 3 yr | GitHub Actions `load-test` job starts or devops-lead triggers manual run | `profile` (baseline\|sso_burst\|coaching\|db_saturation\|scim_burst\|all), `commit_sha` (40-char), `triggered_by` (github_actions\|manual), `environment: 'staging'` |
| `system.load_test_completed` | STANDARD | 3 yr | k6 run exits; all PERF-SLO-01…05 evaluated | `profile`, `commit_sha`, `pass` (bool), `p95_ms_max`, `error_rate`, `duration_seconds`, `vus_peak`, `slo_results` (object: each PERF-SLO → `{pass: bool, actual: number}`) |
| `system.load_test_failed` | HIGH | 7 yr | One or more PERF-SLO-01…05 gates fail; merge blocked | `profile`, `commit_sha`, `failing_slos` (string[]), `p95_ms_max`, `error_rate`, `gate_action: 'merge_blocked'` |
| `system.load_test_gate_bypassed` | **CRITICAL** | 7 yr | IC-approved bypass per §40.2 bypass protocol; triggers AL-PERF-04 (P1 PagerDuty, no auto-resolve) | `commit_sha`, `bypass_reason_hash` (SHA-256), `authorised_by_hash` (SHA-256), `post_deploy_test_linear_id`, `p0_incident_id` (nullable) |
| `system.perf_regression_detected` | HIGH | 7 yr | pg_cron job 30 quarterly check detects PERF-SLO-06 breach (P95 drift > +20% vs. reference quarter) | `quarter` (YYYY-QN), `endpoint`, `p95_current_ms`, `p95_reference_ms`, `drift_pct`, `slo: 'PERF-SLO-06'` |

**HMAC chain ordering (PERF-CHAIN-01):** `system.load_test_completed` or `system.load_test_failed` must follow `system.load_test_initiated` for the same `commit_sha`. Both a `completed` AND a `failed` event for the same `commit_sha` is a chain anomaly (detected by weekly audit). `system.load_test_gate_bypassed` must be emitted before any deployment event for the same `commit_sha` — the CI pipeline validates this ordering via the `emit-audit-event` Worker response before allowing the deployment step to proceed.

**Emitter assignments:** `system.load_test_initiated` / `system.load_test_completed` / `system.load_test_failed` — GitHub Actions (`form_system` via CI service token). `system.load_test_gate_bypassed` — IC (manual admin console; dual-person authorisation). `system.perf_regression_detected` — pg_cron job 30 (`form_system`, quarterly).

---

### Enterprise Pipeline & ARR Forecasting events (DEC-030 HMAC-chained · COST_MODEL §37 · CC4.2)

> Defined in `docs/COST_MODEL.md §37.7`. Four events forming the enterprise pipeline governance audit trail — weekly review cadence, monthly ARR bridge, deal aging triggers, and model recalibration. **Privacy floor (all four events):** no prospect names, contact emails, or individual `user_id` in any payload. `enterprise.deal_aged_out.deal_id` is a FORM-internal UUID from `enterprise_pipeline_stages` — never shared externally. `enterprise.arr_bridge_closed` is a financial record with 7-year retention per Ukrainian Tax Code Art. 44 and SOC 2 financial evidence floor. `enterprise.pipeline_reviewed` carries aggregate pipeline metrics only — `weighted_pipeline_usd` and `pipeline_coverage_ratio` are fleet-level numbers. **PIPE-CHAIN-01:** `enterprise.pipeline_conversion_model_recalibrated` requires a corresponding `docs/DECISION_LOG.md` entry (validated via `decision_log_ref` non-null check); missing `decision_log_ref` is rejected HTTP 422. Cross-ref: `docs/COST_MODEL.md §37.3` (stage conversion rate table), `§37.5` (ARR build table), `§37.8` (pg_cron job 31 + SQL queries). Closes COST_MODEL.md §37.10 checklist item 1 (P0, M7).

| Event type | Severity | Retention | Trigger | Key payload fields |
|---|---|---|---|---|
| `enterprise.pipeline_reviewed` | STANDARD | 3 yr | Weekly Monday pipeline review completed (cadence per §37.7.1) | `review_date` (YYYY-MM-DD), `active_deals` (int), `weighted_pipeline_usd` (number), `pipeline_coverage_ratio` (float — PCR), `deals_aged_out_count` (int), `closed_won_this_week_usd` (number), `actor_id` (founder UUID — FORM internal, not tenant employee) |
| `enterprise.arr_bridge_closed` | STANDARD | 7 yr | Monthly ARR bridge reconciled and signed off by founder (per §37.7.2); investor-grade financial record | `period_month` (YYYY-MM regex), `opening_arr_usd`, `new_arr_usd`, `expansion_arr_usd`, `contraction_arr_usd`, `churned_arr_usd`, `indexation_arr_usd`, `closing_arr_usd`, `monthly_nrr` (float), `active_contracts` (int), `actor_id` (UUID) |
| `enterprise.deal_aged_out` | STANDARD | 3 yr | pg_cron job 31 (`deal_aging_alert`, daily 07:00 UTC) detects deal age exceeding `max_stage_age_days` | `deal_id` (FORM-internal UUID — never shared externally), `stage` (S0_inbound\|S1_qualified\|S2_pilot\|S3_proposal\|S4_legal_review), `entered_at`, `age_days` (int), `max_age_days` (int), `action_required` (call_prospect\|review_qualification\|mid_pilot_intervention\|re_engage_buyer\|contact_legal) |
| `enterprise.pipeline_conversion_model_recalibrated` | STANDARD | 7 yr | Quarterly retrospective reveals > 10pp deviation in any stage conversion rate after ≥ 10 closed-won deals (OQ-PIPE-01 closure trigger) | `recalibration_date` (YYYY-MM-DD), `deals_analysed` (int ≥ 10), `stage_updates` (array: `{stage_transition, old_rate, new_rate, deviation_pp}`), `decision_log_ref` (DEC-XXX slug — **required; null rejected HTTP 422**), `actor_id` (UUID) |

**HMAC chain requirement:** `enterprise.arr_bridge_closed` events must be sequential by `period_month` with no gaps (monthly cadence enforced by the weekly chain audit — missing a month is flagged as a STANDARD compliance gap). `enterprise.pipeline_conversion_model_recalibrated` has a pre-condition guard: `emit-audit-event` Worker checks that `decision_log_ref` is non-null and a `DECISION_LOG.md` entry matching `DEC-` + numeric suffix exists in the git log before accepting the event.

**Emitter assignments:** `enterprise.pipeline_reviewed` — founder (manual, weekly via admin console). `enterprise.arr_bridge_closed` — founder (manual, monthly; data-engineer post-Series A). `enterprise.deal_aged_out` — pg_cron job 31 (`form_system`, daily). `enterprise.pipeline_conversion_model_recalibrated` — founder / data-engineer (manual; triggered on OQ-PIPE-01 closure after Deal 10).

---

### Enterprise Deal Close Governance events (DEC-030 HMAC-chained · COST_MODEL §39 · CC1.4/CC5.2/CC9.2)

> Defined in `docs/COST_MODEL.md §39.8`. Four DEC-030 HMAC-chained events covering the enterprise deal close lifecycle — individual deal-level close events (won and lost), competitive intelligence capture, and OQ-PIPE-01 recalibration. Three events in the `enterprise.*` namespace are registered here; the companion `privacy.no_go_criteria_applied` is registered in §Privacy. **Privacy floor (all four events):** No prospect company name, contact email, or individual employee `user_id` in any payload. `deal_id` is a FORM-internal UUID from `enterprise_pipeline_stages` — never shared externally or linked to individual health data. `competitor_category` and `criteria_triggered` are structured enums only; verbatim loss-call notes remain in CRM. `acv_usd` and `contracted_seats` are aggregate financial metadata with no link to individual health data. **WIN-CHAIN-01 (warning-level, non-blocking):** `enterprise.deal_closed_won` should be preceded by `enterprise.implementation_kickoff_completed` (§36.10) for the same `tenant_id` within 72 hours; missing predecessor logged to Better Stack. Not HTTP 422 — kickoff scheduling may legitimately lag the contract signature. **PIPE-CHAIN-02 (blocking):** `enterprise.win_loss_analysis_recalibrated` is rejected HTTP 422 (`PIPE_CHAIN_02_MISSING_DECISION_REF`) if `decision_log_ref` is null or empty — mirrors PIPE-CHAIN-01 on `enterprise.pipeline_conversion_model_recalibrated` (§37). Cross-ref: `docs/COST_MODEL.md §37` (pipeline stage model — `closed_lost` added to `enterprise_pipeline_stage_enum` via migration 0074 Step 1); `§37.3` (conversion rate table — §39.7 recalibration protocol); `§39.6` (analytics SQL — OQ-PIPE-01 resolution); `§31.5` (price floors — `floor_respected: true` invariant); `§31.8` (`enterprise.pricing_exception_approved` — `pricing_exception_event_id` soft-ref); `§36.10` (`enterprise.implementation_kickoff_completed` — WIN-CHAIN-01 predecessor); `docs/ENTERPRISE.md §"No-go customers"` (insurance risk scoring, gov backdoors, wellness-as-punishment); `docs/DECISION_LOG.md DEC-06X` (OQ-PIPE-01 closure record). **Closes COST_MODEL.md §39.10 checklist item 1 (P0, M9 — register all four §39.8 DEC-030 events before first S5 deal close).**

| Event type | Severity | Retention | Trigger | Key payload fields |
|---|---|---|---|---|
| `enterprise.deal_closed_won` | STANDARD | 7 yr | `enterprise_pipeline_stages.stage` set to `'S5'`; `enterprise_deal_outcomes` row inserted simultaneously; emitter: `customer-success` or `founder` (manual via Admin Console "Close Deal" modal; OQ-WIN-01 evaluates automation after Deal 5) | `deal_id` (FORM-internal UUID), `deal_sequence` (int positive), `tier` (`starter`\|`growth`\|`enterprise`), `contracted_seats` (int positive), `acv_usd` (int positive), `contract_years` (1\|2\|3), `floor_respected: true` (literal — enforced by Zod), `win_primary_reason` (enum: `cv_demo_differentiation`\|`privacy_floor_compliance`\|`soc2_roadmap_credibility`\|`competitive_price_per_seat`\|`product_velocity`\|`champion_pull`\|`pilot_conversion`\|`bundle_package`), `won_vs_competitor_category` (enum nullable: 6 values), `attributed_to_partner_id` (UUID nullable), `pricing_exception_event_id` (UUID nullable — soft-ref to §31.8 chain), `sales_cycle_days` (int positive), `first_payment_due_date` (YYYY-MM-DD) |
| `enterprise.deal_closed_lost` | STANDARD | 3 yr | Founder/AE marks deal lost; `enterprise_pipeline_stages.stage` set to `'closed_lost'`; `enterprise_deal_outcomes` row inserted; emitter: `customer-success` or `founder` (manual); when `no_go_criteria_triggered = true`, Worker auto-emits `privacy.no_go_criteria_applied` as atomic companion event immediately after in the same request | `deal_id` (UUID), `loss_primary_reason` (enum: `price_too_high`\|`feature_gap_ios_only`\|`soc2_not_yet_certified`\|`competitor_won`\|`champion_departed`\|`no_budget_owner`\|`no_go_criteria_triggered`\|`procurement_stall`\|`pilot_no_convert`\|`deferred_not_lost`), `competitor_category` (enum nullable: 6 values — required when `loss_primary_reason = 'competitor_won'`), `no_go_criteria_triggered` (boolean), `last_stage_reached` (S0\|S1\|S2\|S3\|S4), `sales_cycle_days` (int non-negative), `attributed_to_partner_id` (UUID nullable) |
| `enterprise.win_loss_analysis_recalibrated` | STANDARD | 7 yr | OQ-PIPE-01 calibration threshold met: `COUNT(*) FROM enterprise_deal_outcomes WHERE outcome IN ('closed_won','closed_lost') AND NOT no_go_criteria_triggered` ≥ 10; §37.3 updated; DECISION_LOG entry created; emitter: `founder` or `data-engineer` (manual via Admin Console — **PIPE-CHAIN-02: `decision_log_ref` required**) | `deals_analysed` (int ≥ 10), `no_go_excluded_count` (int ≥ 0), `s0_to_s1_actual_pct`\|`s1_to_s2_actual_pct`\|`s2_to_s3_actual_pct`\|`s3_to_s4_actual_pct`\|`s4_to_s5_actual_pct` (five floats 0–100), `overall_win_rate_pct` (float 0–100), `decision_log_ref` (non-empty string — HTTP 422 `PIPE_CHAIN_02_MISSING_DECISION_REF` if absent or empty), `cost_model_section_updated: '§37.3'` (literal), `calibration_date` (YYYY-MM-DD) |

**HMAC chain invariants:**
- **WIN-CHAIN-01 (warning-level):** `enterprise.deal_closed_won` should be preceded by `enterprise.implementation_kickoff_completed` (§36.10) for the same `tenant_id` within 72 hours. Worker logs to Better Stack on missing predecessor; does not return HTTP 422.
- **PIPE-CHAIN-02 (blocking):** `enterprise.win_loss_analysis_recalibrated` → HTTP 422 `PIPE_CHAIN_02_MISSING_DECISION_REF` if `decision_log_ref` is null or empty string. Enforced by `emit-audit-event` Worker before chain-append.
- **no_go companion invariant:** when `enterprise.deal_closed_lost.no_go_criteria_triggered = true`, the Worker appends `privacy.no_go_criteria_applied` in the same atomic array payload (both events share the same `prev_hash` anchor; companion appended second in chain order). Missing companion on a `no_go = true` lost event constitutes a chain anomaly detected by the weekly audit.

**no_go_criteria_triggered exclusion note:** `no_go_criteria_triggered` deals are excluded from OQ-PIPE-01 funnel conversion rate calculations (§39.6.1 WHERE clause) — ethical rejection is not a sales failure. They are counted separately via `privacy.no_go_criteria_applied` events (WIN-E-003 SOC 2 evidence).

**3-year retention rationale for `enterprise.deal_closed_lost`:** Closed-lost deals carry no health data and minimal financial sensitivity. 3 years covers two SOC 2 observation windows; competitive intelligence value degrades after 24 months.

**Zod v2 schemas (canonical source: `docs/COST_MODEL.md §39.8` — update COST_MODEL.md first on any schema change):**

```typescript
const EnterpriseDealClosedWonSchema = z.object({
  deal_id:                    z.string().uuid(),
  deal_sequence:              z.number().int().positive(),
  tier:                       z.enum(['starter', 'growth', 'enterprise']),
  contracted_seats:           z.number().int().positive(),
  acv_usd:                    z.number().int().positive(),
  contract_years:             z.union([z.literal(1), z.literal(2), z.literal(3)]),
  floor_respected:            z.literal(true),
  win_primary_reason:         z.enum([
    'cv_demo_differentiation', 'privacy_floor_compliance', 'soc2_roadmap_credibility',
    'competitive_price_per_seat', 'product_velocity', 'champion_pull',
    'pilot_conversion', 'bundle_package',
  ]),
  won_vs_competitor_category: z.enum([
    'enterprise_wellness_platform', 'point_solution_fitness', 'employee_assistance_program',
    'in_house_hr_initiative', 'no_decision', 'unknown',
  ]).nullable(),
  attributed_to_partner_id:   z.string().uuid().nullable(),
  pricing_exception_event_id: z.string().uuid().nullable(),
  sales_cycle_days:           z.number().int().positive(),
  first_payment_due_date:     z.string().regex(/^\d{4}-\d{2}-\d{2}$/),
});

const EnterpriseDealClosedLostSchema = z.object({
  deal_id:                  z.string().uuid(),
  loss_primary_reason:      z.enum([
    'price_too_high', 'feature_gap_ios_only', 'soc2_not_yet_certified',
    'competitor_won', 'champion_departed', 'no_budget_owner',
    'no_go_criteria_triggered', 'procurement_stall', 'pilot_no_convert', 'deferred_not_lost',
  ]),
  competitor_category:      z.enum([
    'enterprise_wellness_platform', 'point_solution_fitness', 'employee_assistance_program',
    'in_house_hr_initiative', 'no_decision', 'unknown',
  ]).nullable(),
  no_go_criteria_triggered: z.boolean(),
  last_stage_reached:       z.enum(['S0', 'S1', 'S2', 'S3', 'S4']),
  sales_cycle_days:         z.number().int().nonnegative(),
  attributed_to_partner_id: z.string().uuid().nullable(),
});

const EnterpriseWinLossAnalysisRecalibratedSchema = z.object({
  deals_analysed:             z.number().int().min(10),
  no_go_excluded_count:       z.number().int().nonnegative(),
  s0_to_s1_actual_pct:        z.number().min(0).max(100),
  s1_to_s2_actual_pct:        z.number().min(0).max(100),
  s2_to_s3_actual_pct:        z.number().min(0).max(100),
  s3_to_s4_actual_pct:        z.number().min(0).max(100),
  s4_to_s5_actual_pct:        z.number().min(0).max(100),
  overall_win_rate_pct:       z.number().min(0).max(100),
  decision_log_ref:           z.string().min(1),
  cost_model_section_updated: z.literal('§37.3'),
  calibration_date:           z.string().regex(/^\d{4}-\d{2}-\d{2}$/),
});
```

**SOC 2 evidence artefacts (stored at `compliance/evidence/winloss/` with `MANIFEST.sha256`):**

| Artefact | SOC 2 criterion | Source | Retention |
|---|---|---|---|
| **WIN-E-001** | CC5.2 (business risk assessed through deal-level close events) / CC1.4 (pricing floor integrity at close) | Annual export of `enterprise.deal_closed_won` chain events; verify `floor_respected: true` in all records; `compliance/evidence/winloss/WIN-E-001_<YYYY>.csv` | 7 yr |
| **WIN-E-002** | CC5.2 (monitoring of commercial risk) / CC4.1 (financial model governance with DECISION_LOG anchor) | Export of `enterprise.win_loss_analysis_recalibrated` event(s) at Deal 10/20/50 calibration thresholds; `compliance/evidence/winloss/WIN-E-002_<YYYY>.csv` | 7 yr |

---

### White-label custom domain & SSL certificate events (DEC-030 HMAC-chained · OBSERVABILITY §42 · A1.1/CC7.2)

> Defined in `docs/OBSERVABILITY.md §42.10`. Six DEC-030 HMAC-chained events covering the white-label custom domain and SSL certificate lifecycle for enterprise tenants (available ≥ $50k ARR per `docs/ENTERPRISE.md §Branding`). One additional high-volume operational event (`system.white_label_cert_checked`, STANDARD, 1yr — emitted per polled domain by pg_cron job 32 nightly) is excluded from this DEC-030 evidence set and from the weekly chain integrity audit. **Privacy floor (all six events):** Custom domain hostnames are commercially sensitive enterprise-customer identifiers — not GDPR Art. 4(1) personal data, but excluded from all cross-tenant exports, aggregated dashboards visible to other tenants, and SIEM streams beyond the affected tenant's own endpoint. `custom_domain_hash` = SHA-256(`custom_domain`) — plaintext custom domain does not appear in any DEC-030 payload pending OQ-WL-OBS-01 resolution (OBSERVABILITY §42.14, P1, before first white-label tenant goes live). `form_api` role has no access to `tenant_white_label_domains` (REVOKE ALL enforced in migration 0072). Cross-ref: `docs/OBSERVABILITY.md §42.4` (WL-SLO-01 through WL-SLO-04); `docs/ENTERPRISE.md §Branding`; `docs/OBSERVABILITY.md §23` (SLA credit engine — WL-SLO-01 breach triggers credit calculation via `sla_incident_id` FK); `docs/SSO_SCIM_IMPLEMENTATION.md §20` (SAML cert lifecycle — analogous expiry alert escalation ladder, different cert class). **Closes OBSERVABILITY.md §42.14 checklist item 1 (P0, M10 — register all six DEC-030 events in AUDIT_LOG_SCHEMA.md before first white-label tenant goes live).**

**WL-CHAIN-01 ordering invariant:** `tenant.white_label_provisioned` must precede all `tenant.white_label_cert_*` events for the same `tenant_id` in the HMAC chain. A `tenant.white_label_cert_expiry_breach` emitted without a preceding `tenant.white_label_cert_expiry_warning` within the same 30-calendar-day window constitutes a WL-CHAIN-01 violation — the `emit-audit-event` Worker returns HTTP 422 (`WL_CHAIN_01_VIOLATION`) and co-emits `system.white_label_cert_check_stale` to record the monitoring gap. `tenant.white_label_revoked` without a preceding `tenant.white_label_provisioned` for the same `tenant_id` emits a chain WARNING (non-blocking — accommodates rare provisioning-failure-then-cleanup paths). R-05 is triggered on any blocking WL-CHAIN-01 violation.

| Event type | Severity | Retention | Trigger | Key payload fields |
|---|---|---|---|---|
| `tenant.white_label_provisioned` | STANDARD | 7 yr | Cloudflare Custom Hostname API call succeeds; `tenant_white_label_domains` row created with `status = 'active'`; emitter: platform-engineer (PAM-gated `POST /internal/enterprise/white-label/provision`) | `tenant_id` (UUID), `custom_domain_hash` (SHA-256 — OQ-WL-OBS-01 pending), `cloudflare_hostname_id` (string max 128), `provisioned_by_pam_session` (UUID), `provisioned_at` (ISO 8601) |
| `tenant.white_label_cert_expiry_warning` | HIGH | 7 yr | pg_cron job 32 (`white_label_cert_check`, 02:00 UTC daily) detects `ssl_cert_expiry_at` < NOW() + 30 days; triggers AL-WL-03 (P2 Slack, 7-day cooldown per tenant) and AL-WL-04 (P1 PagerDuty dual-page at < 7 days) | `tenant_id` (UUID), `custom_domain_hash` (SHA-256), `days_to_expiry` (int ≥ 0, ≤ 29), `ssl_cert_expiry_at` (ISO 8601), `detected_by: 'pg_cron_job_32'` |
| `tenant.white_label_cert_renewed` | STANDARD | 7 yr | pg_cron job 32 nightly poll detects Cloudflare API returns `ssl_cert_expiry_at` > prior recorded value — cert auto-renewed by Cloudflare SSL for SaaS | `tenant_id` (UUID), `custom_domain_hash` (SHA-256), `old_expiry_at` (ISO 8601), `new_expiry_at` (ISO 8601), `renewed_by: 'cloudflare_auto_renewal'` |
| `tenant.white_label_cert_expiry_breach` | **CRITICAL** | 7 yr | pg_cron job 32 detects `ssl_cert_expiry_at` ≤ NOW() for any `status = 'active'` domain; constitutes WL-SLO-01 + WL-SLO-02 breach; triggers AL-WL-05 (P0 CRITICAL dual-page devops-lead + founder, no auto-resolve); opens §23 SLA incident | `tenant_id` (UUID), `custom_domain_hash` (SHA-256), `expired_at` (ISO 8601 — `ssl_cert_expiry_at` value at detection time), `sla_incident_id` (UUID — §23 SLA incident; optional until §23 SLA credit engine wired at M11), `detected_by: 'pg_cron_job_32'` |
| `tenant.white_label_revoked` | HIGH | 7 yr | Tenant ARR drops below $50k threshold (automated), customer requests removal, or non-payment; `tenant_white_label_domains.status` set to `'revoked'`; Cloudflare Custom Hostname deleted; emitter: `form_system` (automated ARR check) or platform-engineer (PAM-gated manual) | `tenant_id` (UUID), `custom_domain_hash` (SHA-256), `reason` (enum: `downgrade` \| `customer_request` \| `non_payment`), `revoked_by_pam_session` (UUID — no raw user_id), `effective_at` (ISO 8601) |
| `system.white_label_cert_check_stale` | HIGH | 7 yr | pg_cron job 32 detects one or more active domains with `last_checked_at` > 26 hours — nightly check stale or failed; also co-emitted by `emit-audit-event` Worker on WL-CHAIN-01 violation (breach without warning) | `stale_tenant_count` (int ≥ 1 — count of active domains with `last_checked_at` > 26h; no individual `tenant_id` in payload), `detected_by: 'pg_cron_job_32'`, `checked_at` (ISO 8601) |

**HMAC chain ordering (WL-CHAIN-01):** `tenant.white_label_cert_expiry_breach` without a prior `tenant.white_label_cert_expiry_warning` for the same `tenant_id` within 30 days → HTTP 422 + `system.white_label_cert_check_stale` co-emission. The weekly chain audit additionally flags any `tenant.white_label_cert_expiry_breach` without a preceding warning as a HIGH anomaly requiring compliance-officer review within 48 hours.

**Emitter assignments:** `tenant.white_label_provisioned` — platform-engineer (PAM-gated, manual). `tenant.white_label_cert_expiry_warning`, `tenant.white_label_cert_renewed`, `tenant.white_label_cert_expiry_breach`, `system.white_label_cert_check_stale` — pg_cron job 32 (`form_system`, automated nightly). `tenant.white_label_revoked` — `form_system` (automated ARR-threshold check) or platform-engineer (PAM-gated for manual revocations).

**Zod v2 schemas (canonical source for `emit-audit-event` Worker validation):**

```typescript
const TenantWhiteLabelProvisionedSchema = z.object({
  tenant_id:                  z.string().uuid(),
  custom_domain_hash:         z.string().regex(/^[a-f0-9]{64}$/),  // SHA-256(custom_domain) — OQ-WL-OBS-01
  cloudflare_hostname_id:     z.string().max(128),
  provisioned_by_pam_session: z.string().uuid(),
  provisioned_at:             z.string().datetime(),
});

const TenantWhiteLabelCertExpiryWarningSchema = z.object({
  tenant_id:          z.string().uuid(),
  custom_domain_hash: z.string().regex(/^[a-f0-9]{64}$/),
  days_to_expiry:     z.number().int().min(0).max(29),
  ssl_cert_expiry_at: z.string().datetime(),
  detected_by:        z.literal('pg_cron_job_32'),
});

const TenantWhiteLabelCertRenewedSchema = z.object({
  tenant_id:          z.string().uuid(),
  custom_domain_hash: z.string().regex(/^[a-f0-9]{64}$/),
  old_expiry_at:      z.string().datetime(),
  new_expiry_at:      z.string().datetime(),
  renewed_by:         z.literal('cloudflare_auto_renewal'),
});

const TenantWhiteLabelCertExpiryBreachSchema = z.object({
  tenant_id:          z.string().uuid(),
  custom_domain_hash: z.string().regex(/^[a-f0-9]{64}$/),
  expired_at:         z.string().datetime(),
  sla_incident_id:    z.string().uuid().optional(),
  detected_by:        z.literal('pg_cron_job_32'),
});

const TenantWhiteLabelRevokedSchema = z.object({
  tenant_id:              z.string().uuid(),
  custom_domain_hash:     z.string().regex(/^[a-f0-9]{64}$/),
  reason:                 z.enum(['downgrade', 'customer_request', 'non_payment']),
  revoked_by_pam_session: z.string().uuid(),
  effective_at:           z.string().datetime(),
});

const SystemWhiteLabelCertCheckStaleSchema = z.object({
  stale_tenant_count: z.number().int().min(1),
  detected_by:        z.literal('pg_cron_job_32'),
  checked_at:         z.string().datetime(),
});
```

**SOC 2 evidence artefacts:**

| Artefact | SOC 2 criterion | Source | Retention |
|---|---|---|---|
| **WL-E-001** | A1.1 — no active-tenant cert expired during observation period | Annual export of `tenant.white_label_cert_renewed` chain events for all white-label tenants; `compliance/evidence/a1/WL-E-001_<YYYY>.csv` | 7 yr |
| **WL-E-002** | CC7.2 — proactive detective control for cert expiry | AL-WL-03/04/05 PagerDuty config screenshots + 90-day incident history; `compliance/evidence/cc7/WL-E-002_<YYYY>.pdf` | 3 yr |
| **WL-E-003** | CC7.2 — nightly monitoring demonstrated | pg_cron job 32 run history (180-day `system.white_label_cert_checked` event export); `compliance/evidence/cc7/WL-E-003_<YYYY>.pdf` | 3 yr |

---

### Enterprise seat invitation events (DEC-030 HMAC-chained · DATA_MODEL §27 · CC6.1/CC6.2/CC6.3/P4.2/P5.2)

> Defined in `docs/DATA_MODEL.md §27.11` (v1.0, 2026-06-13). Six DEC-030 HMAC-chained events covering the full enterprise seat invitation lifecycle: individual invitations (sent, accepted, expired, revoked) and bulk CSV-import batch framing (started, completed). These events provide the access-authorisation audit trail that SOC 2 CC6.2 requires for every new enterprise seat granted, and the access-removal trail that CC6.3 requires when invitations lapse or are revoked before acceptance. **Privacy invariant (all six events):** `invited_email` never appears in any event payload. `email_hash` = HMAC-SHA256(`invited_email`, `INVITE_HASH_SALT`) — the same hash used in `tenant_invitations.invited_email_hash`; auditors can verify event-to-row linkage by recomputing the hash without accessing any plaintext email. No health data, no coaching data, no individual employee personal data beyond pseudonymous `invitation_id` and role UUIDs. HR (`tenant_manager`) has no access to invitation data — `form_api` read on `tenant_invitations` is gated to `tenant_admin` / `tenant_owner` only. **Emitter (all six events):** `form_system` (automated — invite Workers and pg_cron; no human emitter for any lifecycle event). **SOC 2 criteria:** CC6.1 (seat ceiling enforced via `assertSeatAvailable()` before INSERT — INV-E-001), CC6.2 (`tenant.invite_sent.invited_by` — every access grant has a named authoriser — INV-E-002), CC6.3 (`tenant.invite_revoked` — timely seat release — INV-E-003), P4.2 (privacy notice in invite email before account creation — INV-E-004), P5.2 (`invite_email_expiry_cleanup` pg_cron erasure evidence — INV-E-005). **Closes `docs/DATA_MODEL.md §27.15` checklist item 7 (P0, M5 — register all six DEC-030 events in AUDIT_LOG_SCHEMA.md §6 before invite feature ships to any tenant).**

**INVITE-BULK-CHAIN-01 ordering invariant:** For any CSV bulk import (`bulk_import_job_id`), `tenant.bulk_invite_started` MUST be the first event emitted; all `tenant.invite_sent` events referencing the same `bulk_import_job_id` MUST follow it in chain order; `tenant.bulk_invite_completed` MUST be the last event for that `bulk_import_job_id`. The `emit-audit-event` Worker validates: (a) HTTP 422 if `tenant.invite_sent` with a non-null `bulk_import_job_id` arrives without a preceding `tenant.bulk_invite_started` for that ID; (b) HTTP 422 if `tenant.bulk_invite_completed` arrives without a preceding `tenant.bulk_invite_started`. Inversion → P1 PagerDuty `form-platform` (not R-05 — compliance-signal ordering, not audit-chain tamper). Individual invitations (`source: 'manual'`, `bulk_import_job_id: null`) are exempt from this invariant.

| Event type | Severity | Retention | Trigger | Key payload fields |
|---|---|---|---|---|
| `tenant.invite_sent` | STANDARD | 7 yr | `workers/invitations/create-invite.ts` after `tenant_invitations` INSERT and Resend email dispatch confirmed; also emitted on resend (extends `expires_at`, rotates `token_hash` — OQ-INV-03 option a adopted) | `tenant_id`, `invitation_id` (UUID), `email_hash` (HMAC-SHA256, 64-char hex), `invited_by` (UUID — `tenant_owner` or `tenant_admin`), `source` (`manual`\|`csv_import`), `allocation_id` (UUID — FK `enterprise_seat_allocations`), `expires_at` (ISO 8601 — default 7 days), `bulk_import_job_id` (UUID or null) |
| `tenant.invite_accepted` | STANDARD | 7 yr | `workers/invitations/accept-invite.ts` on token verification + user registration; or SCIM auto-link path in `workers/scim/users/create.ts` when `email_hash` matches a pending invite row | `tenant_id`, `invitation_id` (UUID), `accepted_by` (UUID — newly created FORM user), `assignment_id` (UUID — `enterprise_seat_assignments.id`), `source` (`manual`\|`csv_import`), `linked_via` (`registration`\|`scim`) |
| `tenant.invite_expired` | STANDARD | 3 yr | pg_cron `invite_expiry_sweep` (daily 02:30 UTC — DATA_MODEL §27.15 item 8) sets `tenant_invitations.status = 'expired'` for rows past `expires_at`; emitted per expired invite (batch HMAC-chained); `enterprise_seat_assignments.revoked_at` set concurrently to release the reserved seat | `tenant_id`, `invitation_id` (UUID), `source` (`manual`\|`csv_import`), `bulk_import_job_id` (UUID or null) |
| `tenant.invite_revoked` | HIGH | 7 yr | Manual admin revocation via `DELETE /enterprise/invitations/{id}` with `revoked_by = admin UUID` and `reason = 'admin_action'` or `'seat_reduction'`; or §25 offboarding pipeline on `access_revoked` transition, which revokes all pending invitations for the departing tenant (`revoked_by = null`, `reason = 'offboarding'`) | `tenant_id`, `invitation_id` (UUID), `revoked_by` (UUID or null — null for automated offboarding revocation), `reason` (`admin_action`\|`offboarding`\|`seat_reduction`), `source` (`manual`\|`csv_import`) |
| `tenant.bulk_invite_started` | STANDARD | 3 yr | CSV import Worker begins row processing immediately after `bulk_seat_import_jobs` INSERT; opens the INVITE-BULK-CHAIN-01 chain window | `tenant_id`, `bulk_import_job_id` (UUID), `total_rows` (positive int), `initiated_by` (UUID — `tenant_owner` or `tenant_admin`), `allocation_id` (UUID) |
| `tenant.bulk_invite_completed` | STANDARD | 3 yr | CSV import Worker finishes processing all rows; `bulk_seat_import_jobs.status` updated; closes the INVITE-BULK-CHAIN-01 chain window | `tenant_id`, `bulk_import_job_id` (UUID), `invited_count` (int ≥ 0), `skipped_count` (int ≥ 0), `failed_count` (int ≥ 0), `status` (`completed`\|`partially_failed`\|`failed`) |

**Severity rationale.** `tenant.invite_sent` and `tenant.invite_accepted` are STANDARD — invitations are reversible access grants; their issuance is not a security-consequential event. `tenant.invite_revoked` is HIGH — removing a reserved seat is an irreversible compliance action requiring the full 7-year CC6.3 evidence window. `tenant.invite_expired`, `tenant.bulk_invite_started`, and `tenant.bulk_invite_completed` are STANDARD 3yr — operational lifecycle records sufficient for two SOC 2 observation windows; not required as long-term financial or contract evidence.

**Emitter assignments:** All six events — `form_system` (automated; no human emitter). `tenant.invite_sent` — `workers/invitations/create-invite.ts` (create and resend paths). `tenant.invite_accepted` — `workers/invitations/accept-invite.ts` (registration path) or `workers/scim/users/create.ts` (SCIM auto-link, `linked_via: 'scim'`). `tenant.invite_expired` — pg_cron `invite_expiry_sweep` (job 10, daily 02:30 UTC — DATA_MODEL §27.15 item 8; registered in `docs/OBSERVABILITY.md §12.6`). `tenant.invite_revoked` — `workers/invitations/revoke.ts` (manual, `reason: 'admin_action'`\|`'seat_reduction'`) or the §25 offboarding Worker (`reason: 'offboarding'`). `tenant.bulk_invite_started` + `tenant.bulk_invite_completed` — CSV import Worker.

**Zod v2 schemas (canonical source: `docs/DATA_MODEL.md §27.11` — update DATA_MODEL.md §27 first on any schema change; schemas below are authoritative copies for the `emit-audit-event` Worker registry):**

```typescript
const TenantInviteSentSchema = z.object({
  tenant_id:          z.string().uuid(),
  invitation_id:      z.string().uuid(),
  email_hash:         z.string().regex(/^[a-f0-9]{64}$/),
  invited_by:         z.string().uuid(),
  source:             z.enum(['manual', 'csv_import']),
  allocation_id:      z.string().uuid(),
  expires_at:         z.string().datetime(),
  bulk_import_job_id: z.string().uuid().nullable(),
});

const TenantInviteAcceptedSchema = z.object({
  tenant_id:     z.string().uuid(),
  invitation_id: z.string().uuid(),
  accepted_by:   z.string().uuid(),
  assignment_id: z.string().uuid(),
  source:        z.enum(['manual', 'csv_import']),
  linked_via:    z.enum(['registration', 'scim']),
});

const TenantInviteExpiredSchema = z.object({
  tenant_id:          z.string().uuid(),
  invitation_id:      z.string().uuid(),
  source:             z.enum(['manual', 'csv_import']),
  bulk_import_job_id: z.string().uuid().nullable(),
});

const TenantInviteRevokedSchema = z.object({
  tenant_id:     z.string().uuid(),
  invitation_id: z.string().uuid(),
  revoked_by:    z.string().uuid().nullable(),
  reason:        z.enum(['admin_action', 'offboarding', 'seat_reduction']),
  source:        z.enum(['manual', 'csv_import']),
});

const TenantBulkInviteStartedSchema = z.object({
  tenant_id:          z.string().uuid(),
  bulk_import_job_id: z.string().uuid(),
  total_rows:         z.number().int().positive(),
  initiated_by:       z.string().uuid(),
  allocation_id:      z.string().uuid(),
});

const TenantBulkInviteCompletedSchema = z.object({
  tenant_id:          z.string().uuid(),
  bulk_import_job_id: z.string().uuid(),
  invited_count:      z.number().int().min(0),
  skipped_count:      z.number().int().min(0),
  failed_count:       z.number().int().min(0),
  status:             z.enum(['completed', 'partially_failed', 'failed']),
});
```

**SOC 2 evidence artefacts:**

| Artefact | SOC 2 criterion | Source | Retention |
|---|---|---|---|
| **INV-E-001** | CC6.1 — Seat ceiling enforced; `assertSeatAvailable()` includes pending (`user_id IS NULL`) assignments | Semi-annual export of `enterprise_seat_allocations JOIN enterprise_seat_assignments WHERE user_id IS NULL OR user_id IS NOT NULL` for all active tenants; confirms no tenant exceeded `contracted_seats` at any point during the period; `compliance/evidence/invitations/INV-E-001_<YYYY-HN>.csv` | 7 yr |
| **INV-E-002** | CC6.2 — Every new enterprise seat access grant is authorised by a named actor | Annual export of `tenant.invite_sent` DEC-030 events for the observation period; every row carries `invited_by` UUID confirming no anonymous access grants; `compliance/evidence/invitations/INV-E-002_<YYYY>.csv` | 7 yr |
| **INV-E-003** | CC6.3 — Reserved seat access removed promptly when invitation is revoked | Annual export of `tenant.invite_revoked` DEC-030 events; cross-checked against `enterprise_seat_assignments.revoked_at` within 60 seconds of event `created_at`; `compliance/evidence/invitations/INV-E-003_<YYYY>.csv` | 7 yr |
| **INV-E-004** | P4.2 — Privacy notice provided to data subject before account creation | Version-controlled invite email HTML template at `compliance/evidence/invitations/invite-email-template-v{N}.html`; annotated to show the Privacy Policy hyperlink and one-sentence data-processing summary visible before the "Accept Invitation" CTA button | 7 yr |
| **INV-E-005** | P5.2 — Invited email addresses disposed per retention policy (30-day post-expiry) | pg_cron `invite_email_expiry_cleanup` (daily 04:00 UTC — DATA_MODEL §27.15 item 9) execution log over the observation period; confirms scheduled nullification of `tenant_invitations.invited_email` 30 days post-expiry or post-revocation is operational; `compliance/evidence/invitations/INV-E-005_<YYYY>.csv` | 3 yr |

---

### Enterprise DSAR deletion certificate events (DEC-030 HMAC-chained · DATA_MODEL §35 · P8.0/CC7.2)

> Defined in `docs/DATA_MODEL.md §35.6`. Two DEC-030 HMAC-chained events covering the GDPR Art. 17 deletion certificate issuance and delivery lifecycle for enterprise tenants. Deletion certificates prove to the data subject (employee) and employer HR contact that a two-phase hard-delete has been completed — the certificate is HMAC-SHA256-signed with `ERASURE_CERT_SECRET` (256-bit, annual rotation per `docs/CRYPTOGRAPHY_POLICY.md §5`) and stored durably at R2 `form-dsar-certs/<tenant_id>/<certificate_id>.json`. **Privacy floor (both events):** No plaintext user email, name, or health data in any payload. `user_id_hash` = SHA-256(user_id + PSEUDONYM_SALT); `delivered_to_hash` = SHA-256(email + PSEUDONYM_SALT). Certificate payload (signed JSON) is never stored in Postgres — only `payload_hash` (SHA-256 of the signed certificate) is recorded in the DEC-030 chain and in `dsar_deletion_certificates`. The Admin Dashboard "DSAR & Erasure" panel (`docs/DATA_MODEL.md §35.8`) is accessible to `tenant_admin` and `tenant_owner` only — `tenant_manager` (HR role) is explicitly blocked. Cross-ref: `docs/DATA_MODEL.md §35` (canonical event definitions, Zod schemas, CERT-CHAIN-01 invariant); `docs/DATA_MODEL.md §31` (consumer DSAR `dsar_requests` table — `dsar_id` soft-ref); `docs/DATA_MODEL.md §12` (Art. 17 two-phase erasure — §35 adds the certificate step after Phase 2 verification); `docs/INCIDENT_RESPONSE.md R-05` (CERT-CHAIN-01 chain break → R-05 activation); `docs/OBSERVABILITY.md §37.12` (AL-DSAR-05 — authoritative SLO breach alert, `enterprise_sla_counters.dsar_slo_misses_this_quarter`). **Closes DATA_MODEL.md §35.10 checklist item 5 (P0, M5 — register `dsar.certificate_issued` and `dsar.certificate_delivered` in AUDIT_LOG_SCHEMA.md §6.DSAR-enterprise with Zod schemas).**

**CERT-CHAIN-01 ordering invariant:** The full DSAR erasure chain sequence is `dsar.deletion_soft` → `dsar.deletion_confirmed` → `dsar.certificate_issued` → `dsar.certificate_delivered`. `dsar.certificate_issued` without a preceding `dsar.deletion_confirmed` for the same `dsar_id` is rejected HTTP 422 (`CERT_CHAIN_01_VIOLATION`). `dsar.certificate_delivered` without a preceding `dsar.certificate_issued` for the same `certificate_id` is rejected HTTP 422. Any CERT-CHAIN-01 violation triggers R-05 (HMAC chain break response). The weekly chain audit additionally validates that every `dsar.certificate_issued` has a subsequent `dsar.certificate_delivered` within 48 hours + 1-hour tolerance; absence at the 49-hour mark triggers a P1 compliance alert.

| Event type | Severity | Retention | Trigger | Key payload fields |
|---|---|---|---|---|
| `dsar.certificate_issued` | HIGH | 7 yr | `form-erasure-worker` completes Phase 2 deletion verification, calls `generateDeletionCertificate()` in `src/workers/erasure/certificate.ts`, R2 write succeeds, `dsar_deletion_certificates` metadata row inserted; emitter: `form_system` (automated, synchronous post-Phase-2) | `certificate_id` (UUID — PK of `dsar_deletion_certificates`), `dsar_id` (UUID — soft-ref to `dsar_requests`), `tenant_id` (UUID), `user_id_hash` (SHA-256), `deletion_type` (enum: `consumer_account` \| `enterprise_member_art17` \| `enterprise_employer_art15`), `tables_erased` (string[] min 1), `rows_deleted_total` (int ≥ 0), `erased_at` (ISO 8601), `payload_hash` (SHA-256 of signed certificate JSON), `r2_object_key` (regex: `^form-dsar-certs/[0-9a-f-]{36}/[0-9a-f-]{36}\.json$`) |
| `dsar.certificate_delivered` | STANDARD | 7 yr | Resend `dsar-deletion-cert` email successfully delivered (0h delay for employee, 48h delay for employer HR contact per GDPR Art. 34 notification ordering); emitter: `form-erasure-worker` (`form_system`) on successful Resend delivery confirmation | `certificate_id` (UUID — FK to `dsar_deletion_certificates`), `tenant_id` (UUID), `delivery_channel` (enum: `dashboard` \| `email_data_subject` \| `email_employer_hr`), `delivered_to_hash` (SHA-256(email + PSEUDONYM_SALT) — nullable for dashboard-only delivery), `resend_message_id` (string max 128 — nullable for dashboard delivery) |

**HMAC chain ordering (CERT-CHAIN-01):** Both events are validated by the `emit-audit-event` Worker against predecessor events via HTTP 422 on violation. The full four-event chain (`deletion_soft → deletion_confirmed → certificate_issued → certificate_delivered`) creates a tamper-evident, end-to-end Art. 17 accountability trail — from erasure initiation through certificate delivery — that satisfies SOC 2 P8.0 auditor requirements.

**Emitter assignments:** `dsar.certificate_issued` — `form-erasure-worker` (`form_system`; automated; synchronous post-Phase-2 deletion; fail-closed — erasure Worker aborts if certificate issuance fails). `dsar.certificate_delivered` — `form-erasure-worker` (`form_system`; triggered on Resend delivery webhook confirmation).

**Zod v2 schemas (canonical source: `docs/DATA_MODEL.md §35.7`; the schemas below are authoritative copies for the `emit-audit-event` Worker — update DATA_MODEL.md §35.7 first on any change):**

```typescript
const DsarCertificateIssuedSchema = z.object({
  certificate_id:     z.string().uuid(),
  dsar_id:            z.string().uuid(),
  tenant_id:          z.string().uuid(),
  user_id_hash:       z.string().regex(/^[a-f0-9]{64}$/),
  deletion_type:      z.enum(['consumer_account', 'enterprise_member_art17', 'enterprise_employer_art15']),
  tables_erased:      z.array(z.string()).min(1),
  rows_deleted_total: z.number().int().min(0),
  erased_at:          z.string().datetime(),
  payload_hash:       z.string().regex(/^[a-f0-9]{64}$/),
  r2_object_key:      z.string().regex(/^form-dsar-certs\/[0-9a-f-]{36}\/[0-9a-f-]{36}\.json$/),
});

const DsarCertificateDeliveredSchema = z.object({
  certificate_id:    z.string().uuid(),
  tenant_id:         z.string().uuid(),
  delivery_channel:  z.enum(['dashboard', 'email_data_subject', 'email_employer_hr']),
  delivered_to_hash: z.string().regex(/^[a-f0-9]{64}$/).nullable(),
  resend_message_id: z.string().max(128).nullable(),
});
```

**SOC 2 evidence artefacts:**

| Artefact | SOC 2 criterion | Source | Retention |
|---|---|---|---|
| **DSAR-E-011** | P8.0 — disposal confirmed to data subject with tamper-evident certificate | Quarterly export of `dsar.certificate_issued` chain events; `payload_hash` cross-checked against R2 object SHA-256; `compliance/evidence/dsar/DSAR-E-011_<YYYY-QN>.csv` | 7 yr |
| **DSAR-E-012** | CC7.2 — automated SLO monitoring detects SLA breach without human trigger | AL-DSAR-05 PagerDuty incident history + pg_cron job 33 quarterly reset run log; `compliance/evidence/dsar/DSAR-E-012_<YYYY-QN>.pdf` | 3 yr |

---

### Enterprise Tenant Offboarding & Data Egress events (DEC-030 HMAC-chained · DATA_MODEL §25 + DEC-061 · C1.2/P8.0/CC6.3/CC9.1)

> Defined in `docs/DATA_MODEL.md §25` (off-boarding state machine, package generation, GDPR Art. 17 deletion sequence) and `docs/DATA_MODEL.md §36` (DEC-061 — EU-region R2 bucket routing). Twelve events covering the full tenant off-boarding lifecycle — from contract end through access revocation, data export, deletion, financial pseudonymisation, and attestation issuance. **Privacy floor (all twelve events):** no individual employee `user_id`, name, email, or health values in any payload. `delivered_to_email` in `enterprise.data_export_completed` records the employer's administrative contact (organisation email, not an individual's personal data under GDPR Art. 4(1)); if a named individual's work address is used, the field must be nulled after 2 years per operational log retention. **State machine:** `pending → access_revoked → export_in_progress → export_delivered → deletion_in_progress → financial_pseudonymized → attestation_issued → completed`; plus `on_hold` (legal hold) and `failed` (unrecoverable Worker error) lateral states. A chain break on any off-boarding event is a P0 incident (→ R-05). **Two-person authorisation gate:** the `export_delivered → deletion_in_progress` transition requires two distinct FORM engineers (`authorised_by_1` and `authorised_by_2` on `tenant_offboarding_jobs`); no automated path bypasses this gate. `form_api` is REVOKED from all three off-boarding tables. Cross-ref: `docs/DATA_MODEL.md §25.9` (canonical event table), `docs/DATA_MODEL.md §36.6` (DEC-061 payload extension), `docs/DATA_MODEL.md §25.12` (OFB-E-001–004 evidence definitions), `docs/DATA_MODEL.md §36.7` (OFB-E-005–006 evidence definitions), `docs/SSO_SCIM_IMPLEMENTATION.md §22` (`nukeTenantSessions()` — called on `access_revoked` transition), `docs/INCIDENT_RESPONSE.md R-05` (chain break response), `docs/INCIDENT_RESPONSE.md R-25` (Art. 9 off-boarding overdue), `docs/SOC2_READINESS.md §C1.2` (disposal), `docs/SOC2_READINESS.md §P8.0` (privacy-in-disposal), `docs/SOC2_READINESS.md §CC6.3` (logical access removal), `docs/SOC2_READINESS.md §CC9.1` (business continuity at contract end). **Closes `docs/DATA_MODEL.md §25.13` checklist item 9 (P0, M4 — register all enterprise.offboarding_* DEC-030 events in AUDIT_LOG_SCHEMA.md) and `docs/DATA_MODEL.md §36.9` checklist item 5 (P0, M8 — extend enterprise.data_export_completed payload spec in AUDIT_LOG_SCHEMA.md §6.Enterprise-Offboarding).**

**Event table:**

| Event type | Severity | Retention | Trigger | Mandatory payload fields |
|---|---|---|---|---|
| `enterprise.offboarding_initiated` | **HIGH** | 7 yr | `tenant_offboarding_jobs` row created; `tenants.lifecycle_status → 'churned'` | `tenant_id`, `offboarding_job_id`, `trigger_type` (`contract_expired_no_renewal`\|`early_termination`\|`pilot_lapsed`), `contract_id` or `pilot_id` |
| `enterprise.offboarding_on_hold` | **HIGH** | 7 yr | Legal or compliance hold placed; job blocked from advancing | `tenant_id`, `offboarding_job_id`, `hold_reason` (string ≤ 256 chars), `placed_by_pam_session` (UUID) |
| `enterprise.offboarding_hold_released` | **HIGH** | 7 yr | Hold released by compliance officer; job re-queued for state advance | `tenant_id`, `offboarding_job_id`, `released_by_pam_session` (UUID), `hold_duration_hours` (integer ≥ 0) |
| `enterprise.sso_scim_revoked` | **HIGH** | 7 yr | `nukeTenantSessions()` called; `tenant_sso_configs` deactivated; SCIM tokens revoked; `status → 'access_revoked'` | `tenant_id`, `offboarding_job_id`, `sessions_nuked` (bool), `sso_config_count` (int), `scim_tokens_revoked_count` (int) |
| `enterprise.data_export_started` | STANDARD | 7 yr | Package generation begins; `status → 'export_in_progress'` | `tenant_id`, `offboarding_job_id`, `package_types` (array of `egress_package_type_enum` values) |
| `enterprise.data_export_completed` | STANDARD | 7 yr | All packages generated; signed URLs delivered to employer; `status → 'export_delivered'` | `tenant_id`, `offboarding_job_id`, `package_type`, `package_id`, `sha256_manifest`, `delivered_to_email`, **`data_region`** (DEC-061), **`r2_bucket`** (DEC-061), **`is_eu_region`** (DEC-061), **`package_size_bytes`** (DEC-061) |
| `enterprise.data_deletion_started` | **HIGH** | 7 yr | Two-person authorisation confirmed; deletion worker invoked; `status → 'deletion_in_progress'` | `tenant_id`, `offboarding_job_id`, `authorised_by` (two-element array of FORM engineer UUIDs), `art9_tables` (array of table names to be hard-deleted) |
| `enterprise.data_deletion_completed` | **HIGH** | 7 yr | All personal data hard-deletes and user anonymizations verified | `tenant_id`, `offboarding_job_id`, `users_anonymized_count` (int), `art9_rows_deleted_count` (int), `standard_rows_deleted_count` (int) |
| `enterprise.financial_pseudonymized` | **HIGH** | 7 yr | Billing records pseudonymized per §24.10; `status → 'financial_pseudonymized'` | `tenant_id`, `offboarding_job_id`, `financial_rows_pseudonymized` (int), `retention_basis` (literal `"Art17(3)(b)"`), `legal_reference` (string — cites Ukrainian Tax Code Art. 44 + EU VAT Directive Art. 245) |
| `enterprise.deletion_attestation_issued` | **HIGH** | 7 yr | `tenant_deletion_attestations` row created and HMAC-SHA256 signed; `status → 'attestation_issued'` | `tenant_id`, `offboarding_job_id`, `attestation_id` (UUID), `signing_key_version` (string — key rotation label), `issued_to_email` (employer administrative contact) |
| `enterprise.offboarding_completed` | STANDARD | 7 yr | Compliance officer marks job complete; terminal state | `tenant_id`, `offboarding_job_id`, `total_duration_hours` (int) |
| `enterprise.offboarding_step_failed` | **HIGH** | 7 yr | Worker step throws unrecoverable error; job → `failed` state | `tenant_id`, `offboarding_job_id`, `failed_step` (string — Worker function name), `error_class` (string ≤ 64), `recovery_action` (`retry`\|`manual_intervention_required`) |

**Severity rationale.** `enterprise.data_export_started`, `enterprise.data_export_completed`, and `enterprise.offboarding_completed` are STANDARD — package generation is reversible and attestation issuance is the irreversible milestone. All others are HIGH — access revocation, deletion, pseudonymization, and attestation events are irreversible or legally consequential operations.

---

**`enterprise.data_export_completed` — DEC-061 Payload Extension (EU-region routing)**

DEC-061 (2026-06-15) adds four fields to the existing `enterprise.data_export_completed` event. These fields provide chain-level evidence that EU packages were routed to the EU bucket without requiring a Postgres query:

```typescript
// Extends enterprise.data_export_completed (DATA_MODEL.md §36.6)
// Zod v2 schema — canonical source: DATA_MODEL.md §36.6
// (update DATA_MODEL.md first on any schema change)

const DataRegion = z.enum(['us-east-1', 'eu-central-1', 'eu-west-1']);
type DataRegion = z.infer<typeof DataRegion>;

const DataExportCompletedSchema = z.object({
  // Existing fields (unchanged):
  tenant_id:           z.string().min(1).max(128),
  offboarding_job_id:  z.string().uuid(),
  package_type:        z.enum([
    'aggregate_adoption_report',
    'audit_log_export',
    'contract_documents',
    'scim_provisioning_summary',
  ]),
  package_id:          z.string().uuid(),
  sha256_manifest:     z.string().regex(/^[0-9a-f]{64}$/),
  delivered_to_email:  z.string().email().nullable(),   // null if role-address unknown at delivery time

  // Added in DEC-061 — EU-region routing evidence:
  data_region:         DataRegion,
  r2_bucket:           z.enum([
                         'form-offboarding-exports',
                         'form-offboarding-exports-eu',
                       ]),
  is_eu_region:        z.boolean(),
  package_size_bytes:  z.number().int().nonnegative(),
});
```

**OFB-REGION-01 chain invariant:** For any `enterprise.data_export_completed` event where `is_eu_region = true`, `r2_bucket` MUST equal `'form-offboarding-exports-eu'`. The `emit-audit-event` Worker's payload validator enforces this at emission time:
- Violation → HTTP 422 (event rejected)
- Violation → P1 PagerDuty to `form-platform` + `form-compliance`
- Violation → chain-integrity incident (investigate per `docs/INCIDENT_RESPONSE.md R-05`)

---

**Zod schemas for key events:**

```typescript
// enterprise.offboarding_initiated
const OffboardingInitiatedSchema = z.object({
  tenant_id:           z.string().min(1).max(128),
  offboarding_job_id:  z.string().uuid(),
  trigger_type:        z.enum([
                         'contract_expired_no_renewal',
                         'early_termination',
                         'pilot_lapsed',
                       ]),
  contract_id:         z.string().uuid().optional(),
  pilot_id:            z.string().uuid().optional(),
}).refine(d => d.contract_id != null || d.pilot_id != null, {
  message: 'Either contract_id or pilot_id is required',
});

// enterprise.sso_scim_revoked
const SsoScimRevokedSchema = z.object({
  tenant_id:                   z.string().min(1).max(128),
  offboarding_job_id:          z.string().uuid(),
  sessions_nuked:              z.boolean(),
  sso_config_count:            z.number().int().nonnegative(),
  scim_tokens_revoked_count:   z.number().int().nonnegative(),
});

// enterprise.data_deletion_started
const DataDeletionStartedSchema = z.object({
  tenant_id:           z.string().min(1).max(128),
  offboarding_job_id:  z.string().uuid(),
  authorised_by:       z.tuple([z.string().uuid(), z.string().uuid()]),
  art9_tables:         z.array(z.string().min(1)).min(1),
});

// enterprise.deletion_attestation_issued
const DeletionAttestationIssuedSchema = z.object({
  tenant_id:            z.string().min(1).max(128),
  offboarding_job_id:   z.string().uuid(),
  attestation_id:       z.string().uuid(),
  signing_key_version:  z.string().min(1).max(64),
  issued_to_email:      z.string().email(),
});
```

**Emitter assignments:** All twelve events are emitted by `form_system` (automated off-boarding Workers) running within the same database transaction as the state-transition UPDATE on `tenant_offboarding_jobs`. The sole exception: `enterprise.offboarding_on_hold` and `enterprise.offboarding_hold_released` are emitted by a compliance officer or security engineer via the Admin Console (PAM session required). `enterprise.offboarding_completed` requires a compliance-officer action in the Admin Console to advance from `attestation_issued`; the event is emitted atomically with that status update.

**`form_api` access:** REVOKED from `tenant_offboarding_jobs`, `tenant_data_egress_packages`, and `tenant_deletion_attestations`. `tenant_admin`/`tenant_owner` may SELECT their own job status and packages (read-only, application layer excludes sensitive fields such as `authorised_by_1/2`, `r2_object_key`, and per-table deletion counts). `compliance_reviewer` SELECT all rows (audit and SOC 2 evidence collection). `form_system` full write.

---

**SOC 2 evidence artefacts:**

| Artefact | SOC 2 criterion | Source | Collection cadence | Retention |
|---|---|---|---|---|
| **OFB-E-001** | C1.2 — Confidential disposal | `SELECT * FROM tenant_deletion_attestations WHERE tenant_id = $1`; `signature` verified against `/.well-known/offboarding-signing-key.pub` | Per churned tenant at attestation issuance | 7 yr |
| **OFB-E-002** | P8.0 — Privacy-in-disposal | Export of `enterprise.data_deletion_completed` + `enterprise.financial_pseudonymized` chain events; `retention_basis` + `legal_reference` fields confirm Art. 17(3)(b) basis for pseudonymized financial records | Per churned tenant / observation period | 7 yr |
| **OFB-E-003** | CC6.3 — Timely logical access removal | Export of `enterprise.sso_scim_revoked` chain events + `tenant_offboarding_jobs.access_revoked_at`; demonstrates revocation within off-boarding SLA window | Per churned tenant / observation period | 7 yr |
| **OFB-E-004** | CC9.1 — Business continuity at contract end | `SELECT trigger_type, status, COUNT(*) FROM tenant_offboarding_jobs WHERE created_at BETWEEN $obs_start AND $obs_end GROUP BY 1, 2`; confirms no job stuck in non-terminal state without recorded `on_hold` reason | Observation-period end | 7 yr |
| **OFB-E-005** | C1.1, P4.0, CC6.1 — EU data residency | Quarterly export of `enterprise.data_export_completed` chain events where `is_eu_region = true`; confirms `r2_bucket = 'form-offboarding-exports-eu'`; zero-event quarters filed as affirmative OFB-REGION-01 attestation; `compliance/evidence/ofb/OFB-E-005_<YYYY-QN>.csv` | Quarterly | 7 yr |
| **OFB-E-006** | C1.1, CC6.1 — EU jurisdiction configuration | Annual Cloudflare R2 console screenshot of `form-offboarding-exports-eu` confirming EU jurisdiction; IAM policy export confirming `form-api` NO ACCESS; `compliance/evidence/ofb/OFB-E-006_<YYYY>.md` | Annual | 7 yr |

**SQL for OFB-E-005 collection (`compliance_reviewer` role):**

```sql
-- OFB-E-005: EU egress packages — DEC-030 chain evidence for OFB-REGION-01
SELECT
  event_id,
  created_at,
  payload->>'tenant_id'            AS tenant_id,
  payload->>'offboarding_job_id'   AS offboarding_job_id,
  payload->>'package_type'         AS package_type,
  payload->>'package_id'           AS package_id,
  payload->>'data_region'          AS data_region,
  payload->>'r2_bucket'            AS r2_bucket,
  (payload->>'is_eu_region')::bool AS is_eu_region,
  payload->>'package_size_bytes'   AS package_size_bytes
FROM audit_log_events
WHERE event_type = 'enterprise.data_export_completed'
  AND (payload->>'is_eu_region')::bool = true
  AND created_at >= date_trunc('quarter', now()) - INTERVAL '1 quarter'
  AND created_at <  date_trunc('quarter', now())
ORDER BY created_at;
-- PASS: every row has r2_bucket = 'form-offboarding-exports-eu'.
-- FAIL: any row with a different r2_bucket → escalate to R-05 + P1 PagerDuty.
```

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

**v2.13 · 2026-06-16 · owner: compliance-officer + platform-engineer**
*v2.14 (2026-06-16): +1 `scim.ip_enforcement_misconfigured` advisory event (SSO §32.3 · DEC-062); +`auth_path` optional field to `sso.ip_blocked` payload (SSO §32.4 · DEC-062). Closes SSO_SCIM_IMPLEMENTATION.md §32.8 checklist items 1 (P0/M4 — AUDIT_LOG_SCHEMA §SSO `sso.ip_blocked` payload extension) and 3 (P0/before-SCIM-go-live — register `scim.ip_enforcement_misconfigured` in §SCIM namespace). Changes: (1) `sso.ip_blocked` (§SSO authentication policy events) — payload extension: added `auth_path` field (`'sso' | 'api_key'`, optional, default `'sso'`; backwards-compatible addition; existing consumers ignore unknown fields); distinguishes SSO-path blocks from API-key-path blocks in CC6-E-APIKEY-IP-001 evidence exports. (2) New `### SCIM IP Enforcement Misconfiguration events` section — `scim.ip_enforcement_misconfigured` (STANDARD, 3yr, advisory): emitted by `enforceScimIpAllowlist()` when `scim_ip_enforcement_enabled = true` but `scim_ip_allowlist_config` is null/empty; non-blocking (no 403 thrown — provisioning continues); payload: `tenant_id`, `enforcement_flag: true`, `allowlist_entry_count: 0`, `scim_request_id`; no PagerDuty alert; compliance-officer weekly SIEM review; self-extinguishes once tenant adds CIDRs via Admin Dashboard. Retention table: +1 row (`scim.ip_enforcement_misconfigured` — 3yr, SOC 2 CC6.1 advisory). Privacy floor (both changes): no `user_id`, no employee PII, no health data; `auth_path` is a structural enum tag only; `tenant_id` is org slug. Cross-ref: `docs/SSO_SCIM_IMPLEMENTATION.md §32.3` (OQ-SSO-25.1 resolution — SCIM IP flag design); `docs/SSO_SCIM_IMPLEMENTATION.md §32.4` (OQ-SSO-25.3 resolution — API key IP enforcement); migration `0076_scim_ip_enforcement_flag.sql`; `docs/SOC2_READINESS.md §CC6.1` (CC6-E-SCIM-IP-001 — migration 0076 default-false confirmation); `docs/DECISION_LOG.md DEC-062`. Owner: compliance-officer + security-engineer + enterprise-architect.*
*v2.13 (2026-06-16): +6 `tenant.invite_*` / `tenant.bulk_invite_*` Enterprise seat invitation events — closes `docs/DATA_MODEL.md §27.15` checklist item 7 (P0, M5 — register all six DEC-030 events in AUDIT_LOG_SCHEMA.md §6 before invite feature ships to any tenant). New section `### Enterprise seat invitation events (DEC-030 HMAC-chained · DATA_MODEL §27 · CC6.1/CC6.2/CC6.3/P4.2/P5.2)` inserted before `### Enterprise DSAR deletion certificate events`. Six events registered: (1) `tenant.invite_sent` (STANDARD, 7yr — `email_hash` HMAC-SHA256 only; `invited_by` UUID per CC6.2; `bulk_import_job_id` nullable; emitted on create and resend); (2) `tenant.invite_accepted` (STANDARD, 7yr — `accepted_by` new user UUID; `linked_via` enum `registration`\|`scim` distinguishes auto-link path); (3) `tenant.invite_expired` (STANDARD, 3yr — pg_cron `invite_expiry_sweep` daily 02:30 UTC; `bulk_import_job_id` nullable); (4) `tenant.invite_revoked` (HIGH, 7yr — `revoked_by` nullable for automated offboarding revocation; `reason` 3-value enum: `admin_action`\|`offboarding`\|`seat_reduction`; CC6.3 irreversible compliance action); (5) `tenant.bulk_invite_started` (STANDARD, 3yr — INVITE-BULK-CHAIN-01 window open; `total_rows` positive int); (6) `tenant.bulk_invite_completed` (STANDARD, 3yr — INVITE-BULK-CHAIN-01 window close; 3-count summary + `status` enum). INVITE-BULK-CHAIN-01 ordering invariant: for any `bulk_import_job_id`, `tenant.bulk_invite_started` must precede all `tenant.invite_sent` events for that ID, and `tenant.bulk_invite_completed` must follow all of them; `emit-audit-event` Worker returns HTTP 422 on violation; P1 PagerDuty `form-platform` (not R-05 — compliance-signal ordering, not chain tamper). Six Zod v2 schemas provided (canonical source: `docs/DATA_MODEL.md §27.11`). Five SOC 2 evidence artefacts: INV-E-001 (CC6.1 — seat ceiling enforcement; semi-annual seat assignment export), INV-E-002 (CC6.2 — annual `tenant.invite_sent` chain export confirming `invited_by` for every access grant), INV-E-003 (CC6.3 — annual `tenant.invite_revoked` chain export + `revoked_at` cross-check ≤ 60 seconds), INV-E-004 (P4.2 — version-controlled invite email template with Privacy Policy link before CTA), INV-E-005 (P5.2 — pg_cron `invite_email_expiry_cleanup` execution log confirming 30-day post-expiry `invited_email` nullification). Retention table: +2 rows (7yr for `tenant.invite_sent` / `tenant.invite_accepted` / `tenant.invite_revoked`; 3yr for `tenant.invite_expired` / `tenant.bulk_invite_started` / `tenant.bulk_invite_completed`). Privacy floor (all six events): `invited_email` never in any payload; `email_hash` = HMAC-SHA256 with `INVITE_HASH_SALT`; HR (`tenant_manager`) blocked from invitation data; no health data, no coaching data. Cross-ref: `docs/DATA_MODEL.md §27.11` (canonical event spec and payload fields), `docs/DATA_MODEL.md §27.12` (INV-E-001–005 SOC 2 evidence artefact definitions), `docs/DATA_MODEL.md §27.15` item 7 (P0 checklist — 🟢 closed by this patch), `docs/DATA_MODEL.md §25` (§25 offboarding pipeline emits `tenant.invite_revoked` with `reason: 'offboarding'`), `docs/OBSERVABILITY.md §12.6` (pg_cron job 10 `invite_expiry_sweep` registry), `docs/SOC2_READINESS.md §CC6.2` (INV-E-002 evidence row), `docs/SOC2_READINESS.md §CC6.3` (INV-E-003 evidence row), `docs/INCIDENT_RESPONSE.md R-05` (INVITE-BULK-CHAIN-01 invariant violations — chain integrity escalation). Owner: compliance-officer + platform-engineer.*

**v2.12 · 2026-06-16 · owner: compliance-officer + devops-lead + platform-engineer**
*v2.12 (2026-06-16): +2 `system.quarterly_perf_reference_initiated` / `system.quarterly_perf_reference_completed` events (both STANDARD severity, 3yr retention, HMAC-chained) — closes `docs/OBSERVABILITY.md §45.8` checklist item 2 (P0, M6 — register both DEC-030 events in AUDIT_LOG_SCHEMA.md §6 System namespace before first quarterly PERF-SLO-06 reference run). Events added to the System namespace section after `system.siem_calibration_verified` (DEC-057). PERF-CLOUD-CHAIN-01 ordering invariant: `system.quarterly_perf_reference_initiated` must precede `system.quarterly_perf_reference_completed` for the same `cloud_run_id`; `emit-audit-event` Worker returns HTTP 422 on ordering violation; inversion → P1 PagerDuty `form-platform` (not R-05 — compliance-signal ordering, not audit-chain tamper). Privacy invariant (both events): no `user_id`, no real `tenant_id`, no health data; `staging_refresh_date` is date only; `cloud_run_id` is k6-Cloud-assigned UUID with no FORM user or tenant mapping; `clinical_safety_sign_off: z.literal(true)` in `initiated` event attests ANON-01–ANON-05 verified before dispatch. SOC 2 A1.1 evidence: `system.quarterly_perf_reference_completed.cloud_run_id` is the primary LT-E-001 anchor providing independent third-party k6 Cloud portal corroboration (`compliance/evidence/a1/lt-e-001-YYYY-QN.csv`). Retention table: +1 row (`system.quarterly_perf_reference_*` 3yr, A1.1). Also backfills missing `docs/DECISION_LOG.md DEC-058` entry (k6 hybrid load test platform — OQ-PERF-01 + OQ-PERF-02 resolution, 2026-06-14) which was referenced by `docs/OBSERVABILITY.md §45` but absent from the log. Cross-ref: `docs/OBSERVABILITY.md §45.4` (canonical event specs and Zod schemas), `docs/OBSERVABILITY.md §45.6` (LT-E-001 updated scope — k6 Cloud EU-West as evidence source), `docs/OBSERVABILITY.md §45.8` (implementation checklist item 2 — now 🟢 closed), `docs/DECISION_LOG.md DEC-058` (now filed — OQ-PERF-01/02 resolution), DEC-030. Owner: compliance-officer + devops-lead + platform-engineer.*

**v2.11 · 2026-06-15 · owner: compliance-officer + enterprise-architect + platform-engineer**
*v2.11 (2026-06-15): +12 Enterprise Tenant Offboarding & Data Egress events — closes `docs/DATA_MODEL.md §25.13` checklist item 9 (P0, M4 — register all 8+ `enterprise.offboarding_*` DEC-030 event types) and `docs/DATA_MODEL.md §36.9` checklist item 5 (P0, M8 — extend `enterprise.data_export_completed` payload spec with DEC-061 EU-routing fields). New section `### Enterprise Tenant Offboarding & Data Egress events (DEC-030 HMAC-chained · DATA_MODEL §25 + DEC-061 · C1.2/P8.0/CC6.3/CC9.1)` inserted before `## Export & delivery`. Twelve events registered: (1) `enterprise.offboarding_initiated` (HIGH, 7yr — `trigger_type` enum: contract_expired_no_renewal/early_termination/pilot_lapsed; `contract_id` or `pilot_id` required); (2) `enterprise.offboarding_on_hold` (HIGH, 7yr — legal hold; `hold_reason` ≤ 256 chars; PAM session); (3) `enterprise.offboarding_hold_released` (HIGH, 7yr — `hold_duration_hours`; PAM session); (4) `enterprise.sso_scim_revoked` (HIGH, 7yr — `sessions_nuked` bool; `sso_config_count`; `scim_tokens_revoked_count`); (5) `enterprise.data_export_started` (STANDARD, 7yr — `package_types` array); (6) `enterprise.data_export_completed` (STANDARD, 7yr — **DEC-061 extension**: `data_region` (`us-east-1`\|`eu-central-1`\|`eu-west-1`), `r2_bucket` (`form-offboarding-exports`\|`form-offboarding-exports-eu`), `is_eu_region` bool, `package_size_bytes` int; OFB-REGION-01 chain invariant: EU `is_eu_region=true` → `r2_bucket` must be EU bucket; HTTP 422 + P1 PagerDuty on violation → R-05); (7) `enterprise.data_deletion_started` (HIGH, 7yr — `authorised_by` two-element tuple; two-person gate); (8) `enterprise.data_deletion_completed` (HIGH, 7yr — `users_anonymized_count`, `art9_rows_deleted_count`, `standard_rows_deleted_count`); (9) `enterprise.financial_pseudonymized` (HIGH, 7yr — `retention_basis: literal("Art17(3)(b)")`; `legal_reference`); (10) `enterprise.deletion_attestation_issued` (HIGH, 7yr — `attestation_id`, `signing_key_version`, `issued_to_email`); (11) `enterprise.offboarding_completed` (STANDARD, 7yr — terminal state; compliance-officer Admin Console action); (12) `enterprise.offboarding_step_failed` (HIGH, 7yr — `failed_step`, `error_class`, `recovery_action` enum). Five Zod v2 schemas provided (canonical source: DATA_MODEL.md §25.9 and §36.6 — update DATA_MODEL.md first on any schema change). Six SOC 2 evidence artefacts: OFB-E-001 (C1.2 — `tenant_deletion_attestations` query + signature verification), OFB-E-002 (P8.0 — `data_deletion_completed` + `financial_pseudonymized` chain export), OFB-E-003 (CC6.3 — `sso_scim_revoked` + `access_revoked_at` cross-reference), OFB-E-004 (CC9.1 — distribution query on `tenant_offboarding_jobs`), OFB-E-005 (C1.1/P4.0/CC6.1 — quarterly EU `data_export_completed` chain export; OFB-REGION-01 verification; `compliance/evidence/ofb/OFB-E-005_<YYYY-QN>.csv`; zero-event quarters are affirmative attestation; SQL provided), OFB-E-006 (C1.1/CC6.1 — annual R2 EU jurisdiction screenshot + IAM export; `compliance/evidence/ofb/OFB-E-006_<YYYY>.md`). Retention table: +2 rows (all 12 events split by severity class — HIGH/STANDARD distinction noted, all 7yr because off-boarding events are part of the HMAC chain that must outlive OFB-E-001–006 evidence collection). `form_api` REVOKED from all three off-boarding tables. `compliance_reviewer` SELECT all. Two-person authorisation gate documented: `enterprise.data_deletion_started` `authorised_by` must contain two distinct FORM engineer UUIDs — no automated path bypasses this gate. Cross-ref: `docs/DATA_MODEL.md §25` (canonical event definitions), `docs/DATA_MODEL.md §36.6` (DEC-061 payload extension — canonical Zod schema), `docs/DATA_MODEL.md §36.7` (OFB-E-005/006 evidence definitions), `docs/DATA_MODEL.md §25.12` (OFB-E-001–004 evidence definitions), `docs/DATA_MODEL.md §25.13` item 9 (🟢 closed by this patch), `docs/DATA_MODEL.md §36.9` item 5 (🟢 closed by this patch), `docs/SSO_SCIM_IMPLEMENTATION.md §22` (`nukeTenantSessions()` — `enterprise.sso_scim_revoked` companion action), `docs/INCIDENT_RESPONSE.md R-05` (OFB-REGION-01 violation and any chain break), `docs/INCIDENT_RESPONSE.md R-25` (Art. 9 off-boarding GDPR-SLO-03 overdue — companion runbook to `enterprise.data_deletion_completed`), `docs/SOC2_READINESS.md §C1.2` (disposal criteria — OFB-E-001 primary evidence), `docs/SOC2_READINESS.md §P8.0` (privacy in disposal — OFB-E-002), `docs/SOC2_READINESS.md §CC6.3` (access removal — OFB-E-003), `docs/SOC2_READINESS.md §CC9.1` (third-party risk at contract end — OFB-E-004), `docs/DECISION_LOG.md DEC-061` (EU-region R2 routing decision — OFB-REGION-01 origin). Privacy floor (all 12 events): no individual employee `user_id`, name, email, or health values; `delivered_to_email` is employer administrative contact (organisation address, not GDPR Art. 4(1) personal data; must be nulled after 2yr if named individual's work address used); `authorised_by` contains FORM-internal engineer UUIDs, not tenant employee identifiers; aggregate counts only in deletion events; `form_api` REVOKED. Owner: compliance-officer + enterprise-architect + platform-engineer.*

**v2.10 · 2026-06-15 · owner: compliance-officer + security-engineer + platform-engineer**
*v2.10 (2026-06-15): +2 `sso.mobile_*` Mobile SSO Browser Mode events — closes SSO_SCIM_IMPLEMENTATION.md §30.13 checklist item 4 (P0, M8 — register both DEC-030 events before first enterprise mobile SSO tenant goes live). New section `### Mobile SSO Browser Mode events (DEC-030 HMAC-chained · SSO §30 · SOC 2 CC6.1/CC6.2/CC8.1)` inserted after `### SSO authentication policy events`. (1) `sso.mobile_browser_mode_set` (STANDARD, 7yr): emitted by API Workers middleware (`form_system`) when tenant enables or disables mobile SSO via `PATCH /enterprise/sso/mobile-mode`; payload: `tenant_id`, `mobile_sso_enabled` (bool), `callback_scheme` (3-value enum: universal_link/app_link/custom_scheme), `idp_type` (9-value enum matching `tenant_sso_configs.idp_type`), `sso_protocol` (saml/oidc), `changed_by_user_id` (tenant_admin or tenant_owner UUID — not the end-user); no employee PII, no health data. (2) `sso.mobile_webview_blocked` (HIGH, 7yr): emitted by React Native runtime guard in `apps/mobile/src/auth/sso-browser.ts` (`form_system`) if any code path attempts a WebView open matching an SSO endpoint pattern; payload: `tenant_id`, `attempted_url_hash` (SHA-256[:32] of IdP auth URL — plaintext never stored), `source_module` (string ≤ 100 chars), `blocked_at` (ISO 8601); **zero-fire expected in production — any emission triggers AL-SSO-WEB-01 (P1, PagerDuty `form-security`, `docs/OBSERVABILITY.md §26.8`) and constitutes a security incident**. MOBILE-CHAIN-01 ordering invariant: `sso.mobile_webview_blocked` must follow `sso.mobile_browser_mode_set` with `mobile_sso_enabled: true` for the same `tenant_id`; inversion → R-05 (HMAC chain integrity, `docs/INCIDENT_RESPONSE.md R-05`). Retention table: +2 rows (`sso.mobile_browser_mode_set` 7yr CC6.1/CC6.2; `sso.mobile_webview_blocked` 7yr CC6.1/CC6.2/CC8.1). SOC 2 evidence: SSO-WEB-E-001 (CC6.1/CC6.2 — annual `sso.mobile_browser_mode_set` chain export for all mobile SSO tenants; auditor proof that system-browser mandate was active throughout the observation period; 7yr; `compliance/evidence/sso/SSO-WEB-E-001_<YYYY>.csv`); SSO-WEB-E-002 remains defined in `docs/SSO_SCIM_IMPLEMENTATION.md §30.11` (CC6.2 + CC8.1 — AASA/assetlinks.json snapshots + CI test log + AL-SSO-WEB-01 zero-fire PagerDuty attestation; 3yr). Zod v2 schemas for both events provided in-line (canonical source: `docs/SSO_SCIM_IMPLEMENTATION.md §30.9` — update SSO_SCIM first on any schema change). Privacy floor (both events): no employee `user_id`, name, email, or health data; `attempted_url_hash` is SHA-256[:32] — plaintext URL never stored; HR (`tenant_manager`) has no access to `tenant_sso_configs` or mobile SSO configuration; `form_api` REVOKED. Cross-ref: `docs/SSO_SCIM_IMPLEMENTATION.md §30.9` (canonical Zod schemas + MOBILE-CHAIN-01); `docs/SSO_SCIM_IMPLEMENTATION.md §30.10` (AL-SSO-WEB-01 alert spec + triage tree); `docs/OBSERVABILITY.md §26.8` (AL-SSO-WEB-01 registered as `sso_browser_security` subsection — v4.2, 2026-06-15); `docs/INCIDENT_RESPONSE.md R-05` (chain violation response); `docs/INCIDENT_RESPONSE.md R-08` (supply-chain attack vector if AL-SSO-WEB-01 fires via SDK injection); `docs/DECISION_LOG.md DEC-059` (Mobile SSO system-browser mandate). Owner: compliance-officer + security-engineer + platform-engineer.*

**v2.9 · 2026-06-15 · owner: compliance-officer + enterprise-architect + customer-success**
*v2.9 (2026-06-15): +4 DEC-030 HMAC-chained events from COST_MODEL.md §39 Enterprise Deal Close Governance — closes COST_MODEL.md §39.10 checklist item 1 (P0, M9 — register all four §39.8 events before first S5 deal close). New section `### Enterprise Deal Close Governance events (DEC-030 HMAC-chained · COST_MODEL §39 · CC1.4/CC5.2/CC9.2)` inserted after `### Enterprise Pipeline & ARR Forecasting events`. New section `### Privacy no-go commercial ethics events (DEC-030 HMAC-chained · COST_MODEL §39.8.4 · CC1.4/CC9.2)` inserted in §Privacy after `### Privacy Incident Lifecycle events`. (1) `enterprise.deal_closed_won` (STANDARD, 7yr): emitted by `customer-success` or `founder` (manual via Admin Console "Close Deal" modal) when `enterprise_pipeline_stages.stage` transitions to `'S5'` and `enterprise_deal_outcomes` row is inserted; payload: `deal_id` (FORM-internal UUID — no prospect name or email), `deal_sequence` (positive int), `tier` (3-value enum: starter/growth/enterprise), `contracted_seats`, `acv_usd`, `contract_years` (1|2|3), `floor_respected: true` (literal — Zod z.literal(true)), `win_primary_reason` (8-value enum: cv_demo_differentiation/privacy_floor_compliance/soc2_roadmap_credibility/competitive_price_per_seat/product_velocity/champion_pull/pilot_conversion/bundle_package), `won_vs_competitor_category` (6-value nullable enum: enterprise_wellness_platform/point_solution_fitness/employee_assistance_program/in_house_hr_initiative/no_decision/unknown), `attributed_to_partner_id` (UUID nullable — FK to `enterprise_partners.id`), `pricing_exception_event_id` (UUID nullable — soft-ref to §31.8 chain), `sales_cycle_days` (positive int), `first_payment_due_date` (YYYY-MM-DD). WIN-CHAIN-01 (warning-level, non-blocking): should be preceded by `enterprise.implementation_kickoff_completed` (§36.10) for same `tenant_id` within 72h; missing predecessor logged to Better Stack; not HTTP 422 (kickoff scheduling may legitimately lag). (2) `enterprise.deal_closed_lost` (STANDARD, 3yr): emitted by `customer-success` or `founder` (manual) when deal marked lost; `enterprise_pipeline_stages.stage` set to `'closed_lost'` (added to `enterprise_pipeline_stage_enum` via migration 0074 Step 1); payload: `deal_id`, `loss_primary_reason` (10-value enum: price_too_high/feature_gap_ios_only/soc2_not_yet_certified/competitor_won/champion_departed/no_budget_owner/no_go_criteria_triggered/procurement_stall/pilot_no_convert/deferred_not_lost), `competitor_category` (6-value nullable enum — required when `loss_primary_reason = 'competitor_won'`), `no_go_criteria_triggered` (boolean), `last_stage_reached` (S0–S4 enum), `sales_cycle_days` (non-negative int), `attributed_to_partner_id` (UUID nullable); when `no_go_criteria_triggered = true`, Worker atomically appends `privacy.no_go_criteria_applied` as second event in same array payload (both events share same `prev_hash` anchor). 3yr retention: no health data, covers two SOC 2 observation windows. (3) `enterprise.win_loss_analysis_recalibrated` (STANDARD, 7yr): emitted by `founder` or `data-engineer` (manual via Admin Console) at OQ-PIPE-01 calibration threshold — `COUNT(*) FROM enterprise_deal_outcomes WHERE outcome IN ('closed_won','closed_lost') AND NOT no_go_criteria_triggered` ≥ 10; PIPE-CHAIN-02 (blocking): HTTP 422 `PIPE_CHAIN_02_MISSING_DECISION_REF` if `decision_log_ref` null or empty string; payload: `deals_analysed` (int ≥ 10), `no_go_excluded_count` (non-negative int), five stage conversion rate fields (s0→s1…s4→s5, each float 0–100), `overall_win_rate_pct` (float 0–100), `decision_log_ref` (required non-empty string), `cost_model_section_updated: '§37.3'` (literal), `calibration_date` (YYYY-MM-DD). (4) `privacy.no_go_criteria_applied` (STANDARD, 3yr): auto-companion to `enterprise.deal_closed_lost` when `no_go_criteria_triggered = true`; emitter: `form_system` (automated — never emitted standalone); payload: `deal_id`, `criteria_triggered` (4-value enum: insurance_risk_scoring/government_backdoor_request/wellness_as_punishment_use_case/other_no_go), `review_confirmed_by` (founder UUID — FORM-internal, not prospect contact). Zero-count quarterly filing (WIN-E-003) is affirmative CC1.4 attestation. `no_go_criteria_triggered` deals excluded from OQ-PIPE-01 funnel conversion calculations — ethical rejection is not a sales failure. Three SOC 2 evidence artefacts: WIN-E-001 (CC5.2/CC1.4 — annual `deal_closed_won` export, `floor_respected: true` verification, 7yr; `compliance/evidence/winloss/WIN-E-001_<YYYY>.csv`), WIN-E-002 (CC5.2/CC4.1 — `win_loss_analysis_recalibrated` event at Deal 10/20/50, 7yr; `compliance/evidence/winloss/WIN-E-002_<YYYY>.csv`), WIN-E-003 (CC1.4/CC9.2 — quarterly `no_go_criteria_applied` export + zero-count attestation, 3yr; `compliance/evidence/winloss/WIN-E-003_<YYYY-QN>.csv`). All four Zod v2 schemas provided in new §Enterprise section (canonical source: COST_MODEL.md §39.8 — update COST_MODEL.md first on any schema change). Privacy floor (all four events): no prospect company name, contact email, or individual employee `user_id`; `deal_id` is FORM-internal UUID; `competitor_category` and `criteria_triggered` are structured enums; verbatim loss-call notes remain in CRM only; `form_api` REVOKED from `enterprise_deal_outcomes` (migration 0074 RLS). Cross-ref: COST_MODEL.md §39.8 (canonical Zod schemas), §37.3 (conversion rate table — recalibrated by `win_loss_analysis_recalibrated`), §31.5 (price floors), §31.8 (`pricing_exception_approved` soft-ref), §36.10 (`implementation_kickoff_completed` — WIN-CHAIN-01 predecessor), §38.7 (`enterprise_partners` — `attributed_to_partner_id` FK), §39.9 (WIN-E-001/002/003 evidence paths and filing cadence); ENTERPRISE.md §"No-go customers" (four rejection criteria); DECISION_LOG.md DEC-06X (OQ-PIPE-01 closure — created after Deal 10 calibration); SOC2_READINESS.md CC1.4/CC5.2/CC9.2. Owner: compliance-officer + enterprise-architect + customer-success.*

---

**v2.8 · 2026-06-14 · owner: compliance-officer + security-engineer + enterprise-architect**
*v2.8 (2026-06-14): +8 events across two new namespaces — `tenant.white_label_*` + `system.white_label_cert_check_stale` (OBSERVABILITY §42 · P0, M10) and `dsar.certificate_*` (DATA_MODEL §35 · P0, M5). (1) White-label custom domain & SSL certificate events (6 events, OBSERVABILITY §42.10, A1.1/CC7.2): `tenant.white_label_provisioned` (STANDARD, 7yr — WL-CHAIN-01 anchor; `custom_domain_hash` SHA-256 pending OQ-WL-OBS-01; `cloudflare_hostname_id`, `provisioned_by_pam_session`); `tenant.white_label_cert_expiry_warning` (HIGH, 7yr — pg_cron job 32; `days_to_expiry` int ≤ 29; `ssl_cert_expiry_at`; triggers AL-WL-03/04); `tenant.white_label_cert_renewed` (STANDARD, 7yr — Cloudflare auto-renewal confirmed; `old_expiry_at` → `new_expiry_at`); `tenant.white_label_cert_expiry_breach` (CRITICAL, 7yr — WL-SLO-01/02 breach; triggers §23 SLA incident; `sla_incident_id` optional until M11 credit engine wired; AL-WL-05 dual-page no-auto-resolve); `tenant.white_label_revoked` (HIGH, 7yr — `reason` 3-value enum; `revoked_by_pam_session`; CC6.3 access-removal evidence); `system.white_label_cert_check_stale` (HIGH, 7yr — pg_cron job 32 > 26h stale OR WL-CHAIN-01 breach-without-warning co-emission; `stale_tenant_count` aggregate, no `tenant_id`). WL-CHAIN-01: `tenant.white_label_provisioned` must precede all `tenant.white_label_cert_*` for same `tenant_id`; breach without warning in 30-day window → HTTP 422 `WL_CHAIN_01_VIOLATION` + `system.white_label_cert_check_stale` co-emission; R-05 on blocking violation. Six Zod v2 schemas provided. Three SOC 2 evidence artefacts: WL-E-001 (A1.1 — annual cert_renewed export), WL-E-002 (CC7.2 — AL-WL alert config + 90-day history), WL-E-003 (CC7.2 — pg_cron job 32 180-day run history). Retention table: +3 rows. Privacy floor all six events: `custom_domain_hash` SHA-256 only; `form_api` REVOKED from `tenant_white_label_domains`; no cross-tenant exports. Closes OBSERVABILITY.md §42.14 checklist item 1 (P0, M10). (2) Enterprise DSAR deletion certificate events (2 events, DATA_MODEL §35.6, P8.0/CC7.2): `dsar.certificate_issued` (HIGH, 7yr — `certificate_id`, `dsar_id`, `tenant_id`, `user_id_hash` SHA-256, `deletion_type` 3-value enum, `tables_erased`[], `rows_deleted_total`, `erased_at`, `payload_hash` SHA-256, `r2_object_key` regex-validated path `form-dsar-certs/<tenant_id>/<cert_id>.json`); `dsar.certificate_delivered` (STANDARD, 7yr — `certificate_id`, `tenant_id`, `delivery_channel` 3-value enum, `delivered_to_hash` SHA-256 nullable, `resend_message_id` nullable). CERT-CHAIN-01 ordering invariant: `dsar.deletion_soft` → `dsar.deletion_confirmed` → `dsar.certificate_issued` → `dsar.certificate_delivered`; all predecessor checks enforced HTTP 422; chain break triggers R-05; weekly audit validates `certificate_issued` followed by `certificate_delivered` within 48h + 1h tolerance. Two Zod v2 schemas provided (canonical: DATA_MODEL.md §35.7). Two SOC 2 evidence artefacts: DSAR-E-011 (P8.0 — quarterly `dsar.certificate_issued` export with R2 payload_hash cross-check; 7yr), DSAR-E-012 (CC7.2 — AL-DSAR-05 PagerDuty history + pg_cron job 33 quarterly reset; 3yr). Retention table: +2 rows (both 7yr — `dsar.certificate_delivered` STANDARD 7yr justified: delivery record completes CERT-CHAIN-01 and must cover multi-year SOC 2 observation windows). Privacy floor both events: no plaintext user email or name; `user_id_hash`/`delivered_to_hash` SHA-256 + PSEUDONYM_SALT; certificate payload (signed JSON) never in Postgres; `tenant_manager` (HR) blocked from Admin Dashboard "DSAR & Erasure" panel. Closes DATA_MODEL.md §35.10 checklist item 5 (P0, M5). Cross-ref: OBSERVABILITY.md §42.10 (white-label canonical events), §42.14 (WL-E-001–003 evidence paths), §23 (SLA credit engine `sla_incident_id`); DATA_MODEL.md §35.6 (DSAR cert canonical definitions), §35.7 (canonical Zod schemas), §35.8 (Admin Dashboard panel), §35.9 (OQ-DSAR-04 — HMAC-SHA256 signing); INCIDENT_RESPONSE.md R-05 (WL-CHAIN-01 + CERT-CHAIN-01 violations); ENTERPRISE.md §Branding (≥ $50k ARR gate). Owner: compliance-officer + security-engineer + enterprise-architect.*

**v2.7 · 2026-06-14 · owner: compliance-officer + security-engineer + enterprise-architect**
*v2.7 (2026-06-14): +4 `enterprise.partner_*` partner channel events — closes COST_MODEL.md §38.10 checklist item 1 (P0, M10 — register all four DEC-030 events before first partner agreement is signed). New section `### Enterprise partner channel events (DEC-030 HMAC-chained · COST_MODEL §38.8)` inserted after `### Enterprise renewal & churn lifecycle events`. (1) `enterprise.partner_agreement_signed` (STANDARD, 7yr): compliance-officer or founder emits at agreement execution; Zod v2 schema — `partner_id` (UUID FK `enterprise_partners.id`), `partner_category` (4-value enum), `revenue_share_pct` (float 0–45), `agreement_doc_ref` (internal ID — no company name), `dpa_signed` (boolean — `z.literal(true)` required for reseller/integration; HTTP 422 on false); `created_by_pam_session` UUID. (2) `enterprise.partner_revenue_share_paid` (HIGH, 7yr): compliance-officer manual only — no automated path; hard invariant `privacy_floor_check_passed: z.literal(true)` — HTTP 422 `PART_PAY_FLOOR_CHECK` if false or absent; `partner_id`, `period_start`/`period_end` (ISO 8601 dates), `attributed_arr_usd`, `share_amount_usd`, `payment_method` (2-value enum), `paid_by_pam_session`. (3) `enterprise.partner_deal_attributed` (STANDARD, 7yr): founder or form_system at S5 close with `attributed_to_partner_id` set; `partner_id`, `deal_id` (internal UUID — never shared externally), `tier` (3-value enum), `acv_usd`, `attribution_type` (4-value enum), `sales_cycle_days` (positive integer — §38.5.1 CAC velocity). (4) `enterprise.partner_offboarded` (HIGH, 7yr): compliance-officer manual only; `partner_id`, `termination_reason` (5-value enum), `outstanding_share_usd`, `dispute_open`, `privacy_incident_ref` (string or null — required non-null when `termination_reason = 'privacy_floor_breach'`; HTTP 422 if null for that reason), `offboarded_by_pam_session`. PART-CHAIN-01 ordering invariant: `enterprise.partner_offboarded` with `termination_reason = 'privacy_floor_breach'` or `'no_go_criteria_violation'` must be preceded in chain by `privacy.floor_breach_detected` for affected `tenant_id` within 12 months prior; HTTP 422 `PART_CHAIN_01_VIOLATION` on violation → R-05. Three SOC 2 evidence artefacts defined: PART-E-001 (CC9.2 — annual `partner_agreement_signed` export), PART-E-002 (CC9.2/CC4.1 — annual `partner_revenue_share_paid` export with `privacy_floor_check_passed: true`), PART-E-003 (CC9.2/CC7.4 — `partner_offboarded` observation-period export). All four Zod v2 schemas provided (canonical source for `emit-audit-event` Worker validation; `form_api` REVOKED on `enterprise_partners` table). Retention table: +2 rows (`enterprise.partner_agreement_signed` + `enterprise.partner_deal_attributed` STANDARD 7yr; `enterprise.partner_revenue_share_paid` + `enterprise.partner_offboarded` HIGH 7yr). Cross-ref: `docs/COST_MODEL.md §38.8` (canonical event definitions and Zod schemas), §38.9 (SOC 2 evidence artefacts PART-E-001–003), §38.10 item 1 (P0 checklist — closed by this patch); `docs/INCIDENT_RESPONSE.md R-22` (privacy floor breach — PART-CHAIN-01 predecessor event); `docs/ENTERPRISE.md` (partner no-go criteria). Privacy floor (all four events): no `user_id`, no health values, no coaching content, no partner contact names; `deal_id` is FORM-internal UUID never surfaced to any partner. Owner: compliance-officer + enterprise-architect.*

**v2.6 · 2026-06-14 · owner: compliance-officer + security-engineer + enterprise-architect**
*v2.6 (2026-06-14): +1 `scim.session_revocation_kv_fallback` event (HIGH, 7yr) — closes `docs/SSO_SCIM_IMPLEMENTATION.md §28.8` checklist item 2 (P0, M5 — register `scim.session_revocation_kv_fallback` in AUDIT_LOG_SCHEMA.md before G-013 DPA cleared). Defined in `docs/SSO_SCIM_IMPLEMENTATION.md §28.5` (v2.0, 2026-06-14); resolves DEC-048 (Option c: HTTP 200 + Supabase fallback + AL-REVOKE-01 emit on SCIM KV write failure). Event: emitted by `apps/scim-worker` when `env.SCIM_KV.put()` throws during PATCH deprovisioning; Supabase blocklist fallback path engaged to maintain < 30 s revocation within the 60-second P99 SLA (`ENTERPRISE_SLA.md §3.7`). Payload (Zod v2 — `ScimSessionRevocationKvFallbackSchema`): `tenant_id` (UUID), `scim_request_id` (string max 128), `fallback_path: literal('supabase_blocklist')`, `supabase_ok` (boolean — `false` if both paths fail; triggers P0 co-activation R-02 + R-26), `kv_error_class` (string max 64 — e.g. `'KVError'`, `'NetworkError'`). Privacy invariant: no `user_id`, no email, no role values — `tenant_id` + `scim_request_id` only. REVOKE-CHAIN-01 ordering invariant added: `scim.session_revocation_kv_fallback` must follow `scim.user_deprovisioned` for same `scim_request_id`; `emit-audit-event` Worker returns HTTP 422 `REVOKE_CHAIN_01_VIOLATION` on violation → R-05 (HMAC Chain Break). AL-REVOKE-01 (P1, PagerDuty `form-platform`, dedup key `scim-kv-fallback-{tenant_id}` / 1-hour window, no auto-resolve) fires on `count(scim.session_revocation_kv_fallback) > 0` in any 5-minute window via `scim_kv_fallback_count` Analytics Engine metric. Retention table: +1 row (`scim.session_revocation_kv_fallback` 7yr, SOC 2 CC6.3 + CC7.2). Header corrected v2.4 → v2.6 (v2.5 entry existed in changelog but header was not incremented; this patch fixes the header gap). Cross-ref: `docs/SSO_SCIM_IMPLEMENTATION.md §28.5` (canonical event definition), §28.6 (AL-REVOKE-01 config), §28.7 (SCIM-REVOKE-E-001 evidence artefact), §28.8 checklist item 2 (🟢 closed); `docs/ENTERPRISE_SLA.md §3.7` (60 s P99 revocation SLA — `supabase_ok = true` confirms SLA met); `docs/INCIDENT_RESPONSE.md R-05` (REVOKE-CHAIN-01 violation → immediate escalation); `docs/INCIDENT_RESPONSE.md R-26` (SCIM deprovisioning latency — co-activated when `supabase_ok = false`); `docs/DECISION_LOG.md DEC-048`; `docs/OBSERVABILITY.md §26` (AL-REVOKE-01 alert rule update required — P0 M5). Owner: compliance-officer + security-engineer + platform-engineer.*

**v2.5 · 2026-06-13 · owner: compliance-officer + security-engineer**
*v2.5 (2026-06-13): +1 `system.*` event — `system.compliance_tool_connected` (STANDARD severity, 3yr retention, HMAC-chained). Registered per SOC2_READINESS §78.7. DPA-first activation record for continuous compliance tooling (Vanta or Drata). Emitter: `compliance-officer` (human, once per activation — not `form_system`). Payload: `{tool_name: 'vanta'|'drata', integrations_active: string[], dpa_executed_date: 'YYYY-MM-DD', dpa_file_ref: string, vanta_soc2_report_date?: 'YYYY-MM-DD', activated_by: uuid}`. Privacy invariant: no `user_id`, no `tenant_id`, no health data — tool name and integration list only. DPA must be executed before this event is emitted (DPA-first discipline enforced by §78.6 pre-activation procedure). Zod v2 schema canonical in SOC2_READINESS §78.7; filing path `compliance/evidence/ctool/activation-event.json` (CTOOL-E-006). Retention table: +1 row (`system.compliance_tool_connected` 3yr, SOC 2 CC4.1/CC9.2). Cross-ref: SOC2_READINESS §78, docs/VENDOR_REGISTRY.md (Vanta High-tier vendor), DEC-030. Advances PRE-25 🔴 Open → 🟡 Partial.*

**v2.4 · 2026-06-13 · owner: compliance-officer + security-engineer**
*v2.4 (2026-06-13): +3 `privacy.incident_*` Privacy Incident Lifecycle events — closes SOC2_READINESS.md §77.10 P0 checklist item 1 (register all three events in AUDIT_LOG_SCHEMA.md before first enterprise pilot, M8) and resolves OQ-PRIV-INC-02 (§77.11, P1) by documenting the PI-P3 optional `incident_reviewed` rule in the PRIV-INC-CHAIN-01 enforcement spec. (1) `privacy.incident_opened` (CRITICAL/STANDARD, 7yr/3yr by severity class): compliance-officer or security-engineer emits on first detection of any privacy incident before any investigation action; CRITICAL for PI-P0/P1 → PagerDuty `form-privacy` triple-page (compliance-officer + security-engineer + founder, no dedup); STANDARD for PI-P2/P3 (Slack `#privacy-incidents`); payload: `incident_id` UUID, `severity_class` (PI-P0–PI-P3 enum), `incident_type` (10-value enum: art9_confirmed/art9_suspected/contact_data/privacy_floor_breach/dsar_delay/consent_failure/control_gap/sub_processor/process_deviation/government_request), `detection_source` (8-value enum: pagerduty_alert/dsar_review/annual_review/pentest_finding/internal_report/external_report/sub_processor_notice/regulatory_inquiry), `tenant_id` (UUID optional — null for consumer incidents), `art9_suspected` (boolean), `art33_clock_started` (boolean — true starts 72h GDPR Art. 33 window), `linear_ticket_url` (optional), `opened_by` UUID. (2) `privacy.incident_reviewed` (HIGH, 7yr): compliance-officer emits after scope assessment and Art. 33 determination; required for PI-P0/P1/P2; **optional for PI-P3** (OQ-PRIV-INC-02 resolution: process deviations often resolve in a single engineering fix; Worker does NOT block `incident_closed` for PI-P3 if no prior `incident_reviewed` exists); payload: `incident_id`, `final_severity_class`, `severity_changed` (bool — true if escalated or downgraded from opening class), `art33_required`, `art34_required`, `runbook_invoked` (array of R-11/R-12/R-13/R-14/R-22/section_15/R-07/none), `external_counsel_engaged`, `scope_assessment_notes` (max 500 chars, no PII/health values), `reviewed_by` UUID. (3) `privacy.incident_closed` (HIGH, 7yr): compliance-officer emits after full remediation; `post_incident_review_filed: true` self-attests PIR document existence (PIR filed separately at `compliance/evidence/privacy/incident-register/pir-{incident_id}.md`); `art33_notification_id` carries supervisory authority case reference if Art. 33 notification was filed (never notification content); payload: `incident_id`, `resolution_type` (5-value enum: remediated_no_notification/art33_notification_filed/art34_notifications_sent/false_positive/process_improvement), `root_cause_category` (6-value enum), `recurrence_prevention` (max 500 chars), `post_incident_review_filed`, `closed_by` UUID, `art33_notification_id` (optional string). PRIV-INC-CHAIN-01: `incident_reviewed` and `incident_closed` require preceding `incident_opened` for same `incident_id` → HTTP 422 `PRIV_INC_CHAIN_01_VIOLATION` on violation (blocking); PI-P3 exception for `incident_reviewed` only. Evidence artefacts: PRIV-PI-E-001 (quarterly incident register — `incident_opened` export, jsonl, 7yr), PRIV-PI-E-002 (monthly aggregate count by class, no individual details, 7yr), PRIV-PI-E-003 (PI-P0/P1 close confirmation — `incident_opened` JOIN `incident_closed` with `post_incident_review_filed = true`, 7yr), PRIV-PI-E-004 (`privacy.annual_review_completed` annual event from §60). Retention table: +3 rows (`privacy.incident_opened` PI-P0/P1 7yr; `privacy.incident_opened` PI-P2/P3 3yr; `privacy.incident_reviewed`/`privacy.incident_closed` 7yr). Privacy floor: no health values, no coaching turn content, no employee names, no tenant contact details in any payload; `incident_id` UUID is the sole cross-reference. SOC 2 criteria: P8.2 (privacy incident identification, investigation, communication), P8.3 (individual data accounting — DSAR-triggered incidents), CC7.4 (external party notification — `incident_closed.art33_notification_id`). Cross-ref: `docs/SOC2_READINESS.md §77` (full taxonomy, runbook mapping, evidence paths); `docs/INCIDENT_RESPONSE.md §15` (Art. 33/34 procedure); R-11 (biometric), R-22 (enterprise privacy floor), R-13 (government request), R-14 (DSAR delay); `docs/PRIVACY_IMPACT.md §7` (PIA events — companion P-series event family); DEC-030 (HMAC chain requirement). Owner: compliance-officer + security-engineer.*

**v2.3 · 2026-06-13 · owner: compliance-officer + security-engineer**
*v2.3 (2026-06-13): +25 events across four new namespaces plus five Admin key rotation lifecycle events. (1) `incident.*` Incident Lifecycle (10 events, INCIDENT_RESPONSE §16, CC7.3/CC7.4/CC7.5/CC4.1): `incident.opened` (HIGH, 7yr — `incident_severity`, `incident_runbook`, `trigger_alert_id`, `siem_correlation_score`; IRCHAIN-01 anchor), `incident.ic_assigned` (MEDIUM, 7yr — `minutes_since_open` as IR-SLO-01 evidence), `incident.severity_changed` (HIGH, 7yr — `direction` upgrade/downgrade; `approver_user_id` required for downgrade), `incident.escalated` (HIGH, 7yr — `escalation_target` 5-value enum), `incident.update_posted` (STANDARD, 3yr — `privacy_floor_verified: literal(true)` required; false rejected HTTP 422), `incident.containment_verified` (HIGH, 7yr — `data_exfiltration_confirmed` + `hmac_chain_integrity` bools), `incident.eradicated` (HIGH, 7yr — `root_cause_category` 7-value enum), `incident.recovered` (HIGH, 7yr — `duration_minutes`), `incident.pir_opened` (MEDIUM, 7yr — `pir_due_at` ISO 8601), `incident.pir_closed` (HIGH, 7yr — `critical_action_items_count` + `pir_document_sha256`). IRCHAIN-01 sub-chain rule: `incident.opened` must precede all `incident.*` for same `incident_id`; break = P0 R-05. Dead-man's switches: pg_cron at T+48h for missing PIR + daily for overdue PIR. Closes INCIDENT_RESPONSE.md §16.8 checklist items 1–2 (P0, M4). (2) `wearable.*` Wearable Integration (6 events, OBSERVABILITY §41, A1.1/P3.2): `wearable.sync_completed` (STANDARD, 2yr), `wearable.sync_failed` (HIGH, 3yr — `error_class` enum + `error_message_hash` SHA-256), `wearable.oauth_token_expired` (HIGH, 3yr — `OAuthSource` enum: whoop/oura/garmin only), `wearable.permission_revoked` (HIGH, **5yr** — GDPR Art. 7(3) withdrawal record; `consent_event_id` FK to consent chain; no `user_id`), `wearable.fleet_freshness_assessed` (STANDARD, 2yr — k-anonymity gate: `sources_breakdown` suppressed if any source N < 5), `wearable.stale_data_coaching_context` (HIGH, 3yr — `coaching_context_downgraded: literal(true)`; false = HTTP 422 WS-CHAIN-01; clinical-safety VETO on gate weakening). Privacy floor all six: no `user_id`; `tenant_id` nullable. Closes OBSERVABILITY.md §41.11 checklist item 1 (P0, M5). (3) `system.load_test_*` Performance & Load Testing (5 events, OBSERVABILITY §40, A1.1/CC5.2/CC8.1): `system.load_test_initiated` (STANDARD, 3yr — `environment: literal('staging')` constraint), `system.load_test_completed` (STANDARD, 3yr — `slo_results` object per PERF-SLO-01…05), `system.load_test_failed` (HIGH, 7yr — `failing_slos` array + `gate_action: literal('merge_blocked')`), `system.load_test_gate_bypassed` (CRITICAL, 7yr — AL-PERF-04 no-auto-resolve; `bypass_reason_hash` SHA-256), `system.perf_regression_detected` (HIGH, 7yr — quarterly PERF-SLO-06 breach). PERF-CHAIN-01: completed/failed requires prior initiated for same `commit_sha`. Closes OBSERVABILITY.md §40.10 checklist item 1 (P0, M5). (4) `enterprise.pipeline_*` Enterprise Pipeline & ARR (4 events, COST_MODEL §37, CC4.2): `enterprise.pipeline_reviewed` (STANDARD, 3yr — weekly PCR + weighted pipeline), `enterprise.arr_bridge_closed` (STANDARD, **7yr** — 5-component ARR bridge; Ukrainian Tax Code Art. 44 + SOC 2 financial floor), `enterprise.deal_aged_out` (STANDARD, 3yr — pg_cron job 31 daily; `deal_id` internal UUID only, never shared), `enterprise.pipeline_conversion_model_recalibrated` (STANDARD, 7yr — `decision_log_ref` non-null required; null rejected HTTP 422 PIPE-CHAIN-01). Privacy floor all four: no prospect names or contact emails. Closes COST_MODEL.md §37.10 checklist item 1 (P0, M7). (5) Admin key rotation lifecycle (5 events, SOC2_READINESS §57, CC6.7/CC6.8): `admin.key_rotation_initiated` (CRITICAL, 7yr — PAM session anchor; must precede all rotation events for same `key_name` within 24h), `admin.key_rotation_announced` (HIGH, 7yr — Slack `#engineering` advance notice), `admin.emergency_key_rotation` (CRITICAL, 7yr — `gdpr_art33_clock_started_iso` field; triggers GDPR 72h notification clock), `admin.key_rotation_reminder` (LOW, 3yr — `workers/key-rotation-monitor` cron at 14/7/3 days before expiry), `admin.key_rotation_overdue` (HIGH, 7yr — fires when `days_until_expiry ≤ 0`; triggers AL-KEY-02 P0 no-auto-resolve). Privacy invariant: JWT value never stored — `iat` Unix timestamp only. Closes SOC2_READINESS.md §57.11 checklist items 1–2 (P0, M5). Retention table updated: +9 rows (incident HIGH/MEDIUM/STANDARD entries; wearable 2yr/3yr/5yr entries; load test STANDARD/HIGH/CRITICAL entries; pipeline STANDARD 3yr/7yr entries; admin key rotation CRITICAL/HIGH/LOW entries). Owner: compliance-officer + security-engineer.*

**v2.2 · 2026-06-13 · owner: compliance-officer + security-engineer**
*v2.2 (2026-06-13): +10 `sla.*` Enterprise SLA lifecycle events — closes OBSERVABILITY.md §23.11 checklist item 8 (P0, M4 — "Register all 10 `sla.*` DEC-030 event types in `AUDIT_LOG_SCHEMA.md` event registry"). New section: `### Enterprise SLA events (DEC-030 HMAC-chained · OBSERVABILITY §23 · A1.1 / CC7.2)`. Events: `sla.incident_opened` (HIGH, 7yr — `incident_id` + `probe_ids` + `outage_type` + `downtime_weight`); `sla.incident_closed` (HIGH, 7yr — `duration_minutes`, `exclusion_applied` bool + `exclusion_reason` nullable); `sla.measurement_reconciled` (STANDARD, 7yr — `betterstack_pct` vs. `cf_analytics_pct` dual-source reconciliation, `chosen_source` enum — conservative lower value; `report_month` ISO date regex validated); `sla.credit_calculated` (HIGH, 7yr — `credit_tier_pct` union literal 0|5|15|25|50, `credit_amount_usd`, `mrr_snapshot_usd`); `sla.credit_approved` (HIGH, 7yr — `approved_by` UUID, `final_credit_usd`, `original_calculated_usd`; compliance-officer manual emitter — no automated path); `sla.credit_adjusted` (HIGH, 7yr — `adjustment_delta_usd` signed, `reason` 10–500 chars, `adjusted_by` UUID); `sla.dispute_opened` (STANDARD, 3yr — `dispute_reason` hashed at write layer if contains `@`; `submitted_via` enum); `sla.dispute_resolved` (HIGH, 7yr — `outcome` upheld|rejected, `resolution_notes_hash` SHA-256 — no raw text in chain); `sla.maintenance_window_registered` (STANDARD, 3yr — `notification_hours` evidences §23.4 advance-notice requirement; `tenant_id` nullable for platform-wide windows); `sla.exclusion_reclassified` (HIGH, 7yr — `exclusion_reason` 6-value enum, `approved_by` security-engineer, second-approver required for windows > 30 min validated by Worker). Three SLA-CHAIN ordering invariants: SLA-CHAIN-01 (`sla.incident_closed` requires prior `sla.incident_opened` same `incident_id`); SLA-CHAIN-02 (`sla.credit_approved` requires prior `sla.credit_calculated` same `(tenant_id, report_month)`); SLA-CHAIN-03 (`sla.dispute_resolved` requires prior `sla.dispute_opened` same `(tenant_id, report_month)`); all enforced by `emit-audit-event` Worker (HTTP 422 on violation). Privacy floor: no user_id, no employee PII, no health values; `dispute_reason` hashed if contains `@`. Financial data (`credit_amount_usd`, `final_credit_usd`) protected by RLS tenant-isolation. Ten Zod schemas provided (canonical source for Worker validation). Retention table: +8 rows across five groupings (incident 7yr A1.1; measurement_reconciled 7yr A1.1; credit trio 7yr financial; dispute_opened 3yr operational; dispute_resolved 7yr A1.1 financial; maintenance_window_registered 3yr operational; exclusion_reclassified 7yr A1.1). SOC 2 criterion: A1.1 (availability commitments — incidents, measurement, credits are the contractual evidence chain); CC7.2 (SLA anomaly monitoring — incident events are the anomaly detection audit trail). Cross-ref: `docs/OBSERVABILITY.md §23.9` (canonical event definitions); `docs/ENTERPRISE_SLA.md` (99.9% commitment — these events are its audit trail); `docs/SOC2_READINESS.md §2 A1.1` (gap: SLA credit calculation process — closed by OBSERVABILITY §23 + these chain events). Owner: compliance-officer + security-engineer.*

**v2.1 · 2026-06-13 · owner: compliance-officer + enterprise-architect**
*v2.1 (2026-06-13): +11 events across two new families. CI/CD Pipeline (5 events, OBSERVABILITY §38, CC8.1): `system.deployment_completed` formalised Zod schema (STANDARD, 5yr — `smoke_probe_passed` bool binds deploy to §16 S-001/S-002 probes; `commit_sha` length-40 constraint); `ci.migration_applied` (HIGH, 7yr — `migration_sha256` SHA-256, `rls_policy_changed` bool triggers security-engineer review dashboard flag, `tables_affected[]`, `supabase_project_ref`); `ci.migration_failed` (CRITICAL, 7yr — `error_message_hash` SHA-256 prevents raw SQL leakage in chain; `partial_apply` + `rollback_attempted` bools for incident triage; no dedup, no auto-resolve); `ci.deployment_rolled_back` (HIGH, 7yr — `failing_commit_sha` links to prior `system.deployment_completed` via CI-CHAIN-01; `trigger` enum; `rollback_initiated_by` is `form_admin` UUID — never customer user_id; `minutes_since_deploy` for MTTR calculation); `ci.pipeline_failed` (STANDARD, 3yr — `is_main` bool separates compliance-relevant main failures from PR-branch noise; drives AL-CI-01 routing). Backup & DR (6 events, OBSERVABILITY §39, A1.2/A1.3): `system.backup_completed` (STANDARD, 5yr — `store` enum postgres/r2_primary/r2_dr; `pitr_point_in_time` optional; no user_id/tenant_id — infrastructure only); `system.backup_failed` (CRITICAL, 7yr — AL-BC-05 no-auto-resolve; `error_message_hash` SHA-256); `system.restore_test_initiated` (STANDARD, 7yr — `test_id` UUID is BC-CHAIN-01 anchor; `initiated_by` is PAM session ID not raw user_id; `environment: 'staging'` literal constraint — no production restore events); `system.restore_test_completed` (STANDARD, 7yr — `rto_target_minutes: 240` literal (ENTERPRISE_SLA §4.3); `privacy_floor_verified` bool — must be true for SOC 2 A1.3 evidence; `rpo_gap_minutes` documents actual data loss in test; primary BC-E-002 evidence artefact); `system.restore_test_failed` (CRITICAL, 7yr — `failure_reason: 'privacy_floor_violation'` additionally notifies clinical-safety); `system.backup_staleness_detected` (HIGH, 7yr — emitted by pg_cron job 29 every 4h; `dedup_key` prevents chain spam; `alert_fired` enum links to AL-BC-01/02/03). BC-CHAIN-01 ordering invariant: `restore_test_completed`/`restore_test_failed` requires prior `restore_test_initiated` with same `test_id`; `emit-audit-event` Worker returns HTTP 422 on violation. Retention table: +11 rows (ci.migration_applied 7yr CC8.1; ci.migration_failed 7yr CC8.1 CRITICAL; ci.deployment_rolled_back 7yr CC8.1; ci.pipeline_failed 3yr; system.deployment_completed 5yr CC8.1; system.backup_completed 5yr A1.2; system.backup_failed 7yr A1.2 CRITICAL; system.restore_test_initiated 7yr A1.3; system.restore_test_completed 7yr A1.3; system.restore_test_failed 7yr A1.3 CRITICAL; system.backup_staleness_detected 7yr A1.2). Updated generic `### System` section note: `system.deployment_completed` and `system.backup_completed`/`system.backup_restored` now have full Zod schemas in their respective dedicated sections. Closes OBSERVABILITY.md §38.10 checklist item 1 (P0, M7) and §39.11 checklist item 1 (P0, M13 enterprise GA). Cross-ref: `docs/OBSERVABILITY.md §38.6` (CI/CD canonical schemas); `docs/OBSERVABILITY.md §39.9` (BC canonical schemas); SOC 2 CC8.1 (change management deployment evidence); SOC 2 A1.2 (capacity/environmental monitoring); SOC 2 A1.3 (recovery plan testing). Owner: compliance-officer + devops-lead + platform-engineer.*
*v2.0 (2026-06-13): +6 `privacy.pia_*` Privacy Impact Assessment lifecycle events — closes PRIVACY_IMPACT.md §11 P0/M4 checklist item 1 (register all six event types in AUDIT_LOG_SCHEMA.md before first enterprise pilot). (1) `privacy.pia_filed` (STANDARD, 7yr): compliance-officer emits at PIA Step 1; payload: `pia_id` (PIA-YYYY-NNN regex), `trigger_type` (T-1…T-7), `proposed_by` (role enum), `constraints_affected[]`, `linear_ticket_url`, `filed_by` UUID. (2) `privacy.pia_completed` (HIGH, 7yr): compliance-officer emits at PIA Step 5 final decision; payload: `pia_id`, `risk_rating`, `clinical_safety_required`, `clinical_safety_decision` (4-value enum), `decision` (3-value enum), `conditions[]`, `gdpr_dpia_delta_required`, `completed_by` UUID. (3) `privacy.pia_veto_issued` (CRITICAL, 7yr): clinical-safety agent emits on veto; triggers PRIV-VETO-001 PagerDuty dual-page (compliance-officer + founder, no auto-resolve); payload: `veto_id`, `pia_id`, `constraints_protected[]`, `harm_prevented` (max 500 chars, no health values), `vetoed_by: "clinical-safety"`, `acknowledged_by_founder` (bool — founder must update to true within 24h). (4) `privacy.constraint_relaxation_rejected` (HIGH, 7yr): compliance-officer emits on any formal rejection of a constraint relaxation request; `pia_id` optional (rejections without a PIA also emit this event); payload: `constraint_id` (EF/OC/DM regex), `requester_type` (3-value enum), `rejection_reason` (max 500 chars), `rejected_by` UUID. (5) `privacy.annual_review_scope_drafted` (STANDARD, 3yr): annual scope finalization; payload: `review_year`, `scope_items[]`, `drafted_by`. (6) `privacy.annual_review_completed` (HIGH, 7yr): annual review sign-off; payload: `review_year`, `dpia_version_reviewed`, `pias_in_scope[]`, `gaps_opened/gaps_closed` counts, `sign_off_by`. PIA-CHAIN-01 ordering invariant: `pia_filed` must precede `pia_completed` for same `pia_id`; WARNING (non-blocking) logged if `pia_completed` emitted without prior `pia_filed`. Retention table: +3 rows (PIA governance 7yr; annual_review_scope_drafted 3yr; annual_review_completed 7yr). Privacy floor: no user_id, tenant_id, or individual health values in any payload. Zod schemas provided (canonical source for emit-audit-event Worker validation). Cross-ref: PRIVACY_IMPACT.md §7 (canonical event definitions, trigger conditions), §11 P0/M4 item 1 (checklist — closed by this patch), §10 (PIA Register — pia_id FK links chain entries to register rows); SOC 2 P1.1/P3.2/P8.1; GDPR Art. 5(2) accountability; DEC-046 (OQ-33 clinical-safety ruling — PIA-2026-001 filed in PRIVACY_IMPACT.md §10 as T-6 PIA). Owner: compliance-officer + enterprise-architect.*

**v1.9 · 2026-06-12 · owner: compliance-officer + enterprise-architect**
*v1.9 (2026-06-12): +5 GDPR data lifecycle events — closes OBSERVABILITY.md §37.10 checklist item 11 (P1, M6 — AUDIT_LOG_SCHEMA.md update required). (1) `user.account_deletion_initiated` (STANDARD, 7yr): consumer account deletion Worker fires synchronously after soft-delete (`users.deleted_at = NOW()`); payload: `deletion_request_id` (UUID, no `user_id`), `initiated_at`, `hard_delete_scheduled_at` (+30d), `account_type` (consumer|enterprise_member); fail-closed. (2) `user.data_erasure_completed` (STANDARD, 7yr): DSAR erasure Worker fires after cascade DELETE and `data_subject_requests.status = 'completed'`; payload: `erasure_request_id` (UUID, no `user_id`), `request_age_days` (float — GDPR-SLO-01 ≤25.0 evidence), `tables_purged` (array), `completed_at`, `slo_met` (bool). (3) `user.art9_data_hard_deleted` (HIGH, 7yr): enterprise Art. 9 off-boarding Worker; must complete ≤4h (GDPR-SLO-03 / DEC-036, P0 AL-GDPR-03 if breach); payload: `tenant_id`, `offboarding_initiated_at`, `hard_delete_completed_at`, `latency_seconds` (SLO evidence), `users_hard_deleted_count`, `art9_tables_purged` (array); no `user_id`. (4) `data.workout_data_purged` (STANDARD, 1yr): pg_cron job 26 (daily 02:00 UTC) fires after DELETE of workout_sets/sessions/body_metrics for post-30d deleted users; payload: `purge_date`, `users_purged_count`, `workout_sessions_deleted`, `workout_sets_deleted`, `body_metrics_deleted`, `pg_cron_run_id`; aggregate counts, no `user_id`; GDPR-E-002 (SOC 2 PI1.2). (5) `data.audit_log_purge_completed` (STANDARD, 7yr): pg_cron job 27 (monthly 1st 03:00 UTC); **DEC-030 ordering invariant — emitted and HTTP 200 confirmed BEFORE DELETE executes** (chain entry becomes new tail before oldest rows removed — failure aborts DELETE); payload: `purge_month`, `rows_eligible`, `rows_deleted`, `oldest_retained_event_timestamp`, `chain_verification_result` (pass|skip_no_eligible_rows), `pg_cron_run_id`; GDPR-E-003 (SOC 2 PI1.2/CC6.5). Five Zod schemas provided (canonical source for emit-audit-event Worker). Retention table: +4 rows (`user.account_deletion_initiated`/`user.data_erasure_completed` 7yr; `user.art9_data_hard_deleted` 7yr; `data.workout_data_purged` 1yr; `data.audit_log_purge_completed` 7yr). Privacy floor all five: no plaintext `user_id` in any payload; consumer-facing events use UUID FKs to RLS-gated Postgres tables; `user.art9_data_hard_deleted` is `tenant_id` + aggregate counts only. Emitter all five: `form_system` (automated Workers and pg_cron — no manual admin console path). Cross-ref: OBSERVABILITY.md §37 (canonical event definitions, SLOs, alert rules, pg_cron DDL); GDPR Art. 5(1)(e) storage limitation / Art. 17 erasure / Art. 9 special categories; SOC 2 PI1.2/P4/P5.2/P6/CC6.5; DEC-036 (Art. 9 hard-delete zero grace period). Owner: compliance-officer + platform-engineer + devops-lead.*

**v1.8 · 2026-06-12 · owner: compliance-officer + enterprise-architect**
*v1.8 (2026-06-12): +3 `enterprise.*` implementation lifecycle events — closes COST_MODEL.md §36.11 checklist item 1 (P0, M8 — register all three DEC-030 events before first enterprise deal closes). (1) `enterprise.implementation_kickoff_completed` (STANDARD, 7yr): CSM-emitted at kickoff completion; payload: `tenant_id`, `deal_sequence` (nth deal for OQ-08 cost calibration), `contracted_tier`, `contracted_seats`, `idp_type`, `white_label_enabled`, `eu_data_residency`, `kickoff_date`, `target_go_live_date`, `csm_actor_id` (FORM team UUID — not tenant employee); emitter: customer-success (founder in solo phase). (2) `enterprise.sso_scim_setup_verified` (STANDARD, 7yr): engineer-emitted after SSO/SCIM smoke test per SSO_SCIM_IMPLEMENTATION.md §7.4; payload: `tenant_id`, `idp_type`, `sso_modes_verified` (object: `idp_initiated`, `sp_initiated`), `scim_features_enabled` (object: `user_crud`, `group_sync`, `role_mapping`, `jit_provisioning`), `eu_data_residency_confirmed`, `engineer_actor_id` (FORM team UUID), `verification_date`; emitter: platform-engineer or form_system (automated smoke harness if available). (3) `enterprise.implementation_cost_model_calibrated` (STANDARD, 7yr): founder-emitted at OQ-08 closure after 3 deals time-tracked in `enterprise_impl_time_log`; payload: `deals_analysed` (int ≥ 3), `avg_impl_cost_*` (optional per tier), `variance_vs_model_pct` (signed float), `model_version_updated`, `decision_log_ref`, `calibration_date`; no `tenant_id` (aggregate model event, not per-tenant). Privacy invariant all three events: `tenant_id` + operational FORM-team metadata only — no individual employee user_id, no health values, no coaching content; `csm_actor_id` / `engineer_actor_id` are FORM-internal identities safe for chain payload. Chain ordering: no strict predecessor-successor enforcement between events; WARN (non-blocking) logged by emit-audit-event Worker if `sso_scim_setup_verified` has no prior `implementation_kickoff_completed` for same `tenant_id`; no chain guard for `implementation_cost_model_calibrated` (no tenant_id). No PagerDuty routing (STANDARD severity); events appear in weekly chain audit and CSM implementation dashboard only. Retention table: +1 row (all three events, 7yr, SOC 2 CC5.2 + CC7.2). Cross-ref: COST_MODEL.md §36.10 (canonical Zod schemas + trigger conditions), §36.11 checklist (this patch closes item 1), §36.9 (`enterprise_impl_time_log` table — source for calibration query); SSO_SCIM_IMPLEMENTATION.md §7.4 (test procedure — operational counterpart of sso_scim_setup_verified); ENTERPRISE_ONBOARDING.md (sequential trigger points); DECISION_LOG.md (OQ-08 closure record — decision_log_ref target). Owner: compliance-officer + enterprise-architect.*

**v1.7 · 2026-06-12 · owner: compliance-officer + enterprise-architect**
*v1.7 (2026-06-12): +3 `enterprise.*` contract amendment & ETF lifecycle events — closes COST_MODEL.md §35.10 checklist item 1 (P0, M10 — register all three DEC-030 events before first enterprise multi-year contract close). (1) `enterprise.mid_contract_termination_risk_flagged` (HIGH, 7yr): pg_cron job 25 `mid_contract_termination_risk_check` weekly detection of `chs_score < 20` sustained ≥ 4 consecutive snapshots AND `days_remaining > 180`; 30-day per-tenant deduplication; payload: `tenant_id`, `chs_score`, `weeks_below_threshold`, `days_remaining`, `contract_acv_usd`, `alerted_to`; emitter: `form_system`; no `user_id` (tenant-level signal). (2) `enterprise.contract_amended` (HIGH, 7yr): manual CSM-emitted on any signed contract amendment; four amendment_type values (price_lock_renewal, seat_reduction, sla_addendum, other); `decision_log_ref` required for seat_reduction > 10% contracted seats; `acv_delta_usd` signed integer (negative for reductions). (3) `enterprise.early_termination_fee_waived` (HIGH, 7yr): manual founder-emitted; `ev_analysis_completed: true` required for waivers > $5,000 or Worker returns HTTP 422 `ETF_WAIVER_REQUIRES_EV_ANALYSIS`; eight waiver_reason enum values matching §35.5.5 approval matrix; HMAC ordering guard: risk-flagged event required in chain within 12 months before ETF waiver (bypass for `small_etf_enforcement_uneconomic`). Retention table: +1 row (`enterprise.*` contract amendment events, 7yr — merged with existing HIGH `enterprise.*` row). New section inserted before `## Export & delivery`. Cross-ref: COST_MODEL.md §35.9.1–§35.9.3 (canonical Zod schemas), §35.5.5 (approval authority matrix), §35.10 (checklist); OBSERVABILITY.md §36 (AL-ETF-01 alert rule + pg_cron job 25); MSA_TEMPLATE.md §11.4 (ETF clause — P0 M10 update). Owner: compliance-officer + enterprise-architect.*

*v1.6 (2026-06-11): +5 `scim.*` SCIM mass deprovisioning events — closes INCIDENT_RESPONSE.md R-24.15 checklist item 3 (P0 M5 — register all five DEC-030 events before first enterprise pilot goes live). Document header corrected v1.4 → v1.6 (v1.5 content was applied 2026-06-11 but header was not incremented at that time). (1) `scim.mass_deprovision_detected` (CRITICAL, 7yr): AL-SCIM-MASS-01 pg_cron threshold breach; payload: `tenant_id`, `deprovisioned_count`, `pct_of_seats`, `window_minutes`, `incident_id`, `detection_source`; triggers PagerDuty P0/P1 dual-page before any containment action. (2) `scim.sync_suspended` (HIGH, 7yr): SCIM pause KV key written by IC/platform-engineer; payload: `tenant_id`, `suspended_by_admin_id`, `reason`, `auto_resume_at`, `incident_id`. (3) `scim.admin_lockout_recovery` (HIGH, 7yr): magic link issued when tenant_owner is locked out; payload: `tenant_id`, `issued_by_admin_id`, `tenant_owner_email_hash` (SHA-256 — never raw), `incident_id`, `magic_link_expires_at` (15-min TTL). (4) `scim.sync_resumed` (HIGH, 7yr): SCIM suspension lifted after root cause resolved; payload: `tenant_id`, `resumed_by_admin_id`, `root_cause_hypothesis` (H1/H2/H3/H4), `incident_id`, `users_restored_count`. (5) `scim.mass_reprovision_complete` (STANDARD, 3yr): full recovery confirmed; payload: `tenant_id`, `restored_count`, `unrestorable_count` (0 expected), `incident_id`, `recovery_method` (idp_repush/form_manual/both). Privacy invariant all five events: `tenant_id` and aggregate counts only — no `user_id`, no employee names, no health data. R24-CHAIN-01 ordering guard: `scim.mass_deprovision_detected` must precede `scim.sync_suspended`; `scim.sync_suspended` must precede `scim.sync_resumed` per `incident_id`; chain guard in `emit-audit-event` Worker returns HTTP 422 on violation. Retention table: +2 rows (`scim.mass_deprovision_detected / sync_suspended / admin_lockout_recovery / sync_resumed` 7yr; `scim.mass_reprovision_complete` 3yr). Cross-ref: INCIDENT_RESPONSE.md §R-24.9 (canonical event definitions), §R-24.15 item 3 (checklist — closed by this patch); `docs/OBSERVABILITY.md §26.7a` (AL-SCIM-MASS-01 alert rule and pg_cron job 24 `scim_mass_deprovision_check`); `docs/SOC2_READINESS.md §A1.1/A1.2/CC7.2/CC7.3/CC9.2` (evidence artefacts SCIM-E-001 through SCIM-E-005). Owner: compliance-officer + enterprise-architect.*

**v1.5 · 2026-06-11 · owner: compliance-officer + enterprise-architect**
*v1.5 (2026-06-11): +2 events from DATA_MODEL §30 OQ-BILL-05/RL-02 resolution. (1) `billing.user_erased` (HIGH, 7yr, HMAC-chained) — GDPR Art. 17 erasure record for `subscription_events`; Zod schema v2 patched: `erased_user_reference: z.string().regex(/^\[ERASED-[0-9a-f]{64}\]$/)` required; `user_id: z.string().uuid()` demoted to `.optional()` (backward-compatible with pre-migration events); `rows_pseudonymized`, `erasure_request_id`, `pseudonymized_at` fields; emitter: `erasure-worker` (`form_system`), step SUB-1, synchronous inside erasure transaction; fail-closed; weekly chain audit checks post-migration events have `user_id = null` (non-null post-migration = P1 compliance anomaly). (2) `security.quota_95pct_warning` (STANDARD, 3yr, HMAC-chained) — fires when tenant request count crosses 95% of monthly quota (`QUOTA_SECONDARY_WARN_PCT = 0.95`); deduplicated via `api_quota_usage.secondary_warn_sent_at` (DATA_MODEL §30.3.3); at most one emission per tenant per billing period; payload: `tenant_id`, `period_month`, `endpoint_category`, `current_requests`, `quota_limit`, `pct_used`, `warn_threshold: 0.95`; no `user_id` (tenant-level quota event); emitter: `quota-check` Cloudflare Worker. Security & tenant isolation section header updated: "Four" → "Five" events (quota_95pct_warning added to table). New "Billing & GDPR erasure events" section added before API Key section. Retention table: +2 rows (`billing.user_erased` 7yr, `security.quota_95pct_warning` 3yr). Closes DATA_MODEL §30.7 items 4 (P0 M4 — billing.user_erased Zod patch) and 5 (P0 M4 — security.quota_95pct_warning registration). Cross-ref: DATA_MODEL §30.2.4 (SUB-1 erasure step), §30.3.2 (quota-check.ts update), §30.3.3 (secondary_warn_sent_at dedup), §30.5.1 (quota_95pct_warning schema), §30.5.2 (user_erased Zod patch); SOC2_READINESS.md P5.2 ERA-E-001/002/003; CRYPTOGRAPHY_POLICY.md §5/§6 (ERASURE_PSEUDONYM_SALT write-once key). Owner: compliance-officer + enterprise-architect.*

**v1.5 · 2026-06-14 · owner: compliance-officer + security-engineer**
*v1.5 (2026-06-14): +3 `system.evidence_*` evidence collection automation events. `system.evidence_collection_automated` (STANDARD, 7yr, AUTO-E-001 SOC 2 CC4.2/CC7.1): emitted by Cloudflare Cron Worker (§81 of SOC2_READINESS.md) once per successful monthly run; EVD-CHAIN-01 ordering invariant enforced — `system.evidence_vault_configured` must precede this event in the chain for the same calendar year; HTTP 422 on violation; payload fields: `period` (`/^\d{4}-\d{2}$/`), `artifacts_written` (string[], each starting with `compliance/evidence/`), `chain_integrity_ok: true` (literal), `slo_all_met`, `master_index_hash` (SHA-256, 64 chars), `run_duration_ms` (positive int). `system.evidence_cron_stale` (HIGH, 3yr): emitted by pg_cron job 33 (`evidence_cron_freshness_check`, 03:05 UTC 2nd of month) when no `system.evidence_collection_automated` found for prior period within 48 h; dead-man's switch signal → AL-EVD-01 P2 Slack; payload: `period`, `checked_at` (ISO-8601), `window_hours: 48` (literal). `system.evidence_cron_conflict` (MEDIUM, 1yr): emitted by Cloudflare Cron Worker on R2 `If-Match` ETag conflict during MASTER-INDEX reconciliation — Worker retries once, emits this event and aborts if second attempt also fails; no `system.evidence_collection_automated` emitted for that run; triggers AL-EVD-04 P2 Slack; payload: `period`, `r2_key`, `attempt_number: 1 | 2`. Privacy floor (all three events): `actor_id = NULL`, `actor_type = 'system'`, `tenant_id = '00000000-0000-0000-0000-000000000000'` sentinel; HR never sees; `compliance_reviewer` reads; `form_api` REVOKED. Retention table: +3 rows. OBSERVABILITY.md §12.6 updated with job 33 (`evidence_cron_freshness_check`, 32-day freshness window) in the canonical pg_cron job registry. Closes SOC2_READINESS.md §81.11 P0 tasks 1 and 2. Cross-ref: SOC2_READINESS.md §81 (Worker spec), OBSERVABILITY.md §12.6 (job 33), DEC-030.*

**v1.5 · 2026-06-15 · owner: compliance-officer + devops-lead + security-engineer**
*v1.5 (2026-06-15): +2 `system.*` SIEM calibration governance events — `system.siem_alert_activated` (STANDARD, 3yr) and `system.siem_calibration_verified` (STANDARD, 3yr). Both HMAC-chained per DEC-030. (1) `system.siem_alert_activated`: emitted by `devops-lead` (manual, Admin Console) when AL-SIEM-06 dead-man's switch transitions from shadow mode (P3 Slack-only) to full activation (P1 PagerDuty) per the graduated-activation policy (DEC-056); payload: `rule_id` (literal `AL-SIEM-06`), `activation_mode` (shadow|full enum), `shadow_period_start`, `shadow_period_end`, `auth_event_avg_per_30min` scalar, `activated_by` UUID; evidence artefact CALIB-E-001 (SOC 2 CC4.1); filing path `compliance/evidence/cc7/siem-calibration/siem-alert-activated-<iso-date>.json`. (2) `system.siem_calibration_verified`: emitted by `form_system` CI post-step hook when `cr01-per-tenant-isolation.test.ts` passes in PR pipeline; payload: `correlation_rule` (literal `CR-01`), `test_id` (CI run ID), `test_result` (literal `passed`), `tenant_ids_used` (array of synthetic `lt-` tenant IDs — real tenant_id NEVER present); evidence artefact CALIB-E-002 (SOC 2 CC7.2); filing path `compliance/evidence/cc7/siem-calibration/cr01-calibration-<ci-run-id>.json`. Privacy invariant both events: no user_id, no real tenant_id, no health data — operational calibration metadata only. Closes SOC2_READINESS §83.4 AUDIT_LOG_SCHEMA registration items for CALIB-E-001 and CALIB-E-002. Cross-ref: SOC2_READINESS §83 (§83.2 DEC-056 resolution, §83.3 DEC-057 resolution, §83.5 evidence artefacts), OBSERVABILITY §44.2 (AL-SIEM-06 graduated activation), OBSERVABILITY §44.3 (CR-01 per-tenant isolation), DEC-056, DEC-057. Owner: compliance-officer + devops-lead + security-engineer.*

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
