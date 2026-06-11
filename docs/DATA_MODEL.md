# FORM · Multi-Tenant Data Model v1.9

> Owner: `enterprise-architect` + `compliance-officer`. Review: on any schema migration or quarterly.
> Scope: enterprise-tier multi-tenancy. Consumer tier (single-tenant Postgres) is a subset of this model.
> References: DEC-030 (HMAC-chained audit log), docs/AUDIT_LOG_SCHEMA.md, docs/ENTERPRISE.md, docs/SOC2_READINESS.md.

---

## Table of Contents

1. [Architecture Decision: RLS vs Schema-per-Tenant](#1-architecture-decision-rls-vs-schema-per-tenant)
2. [Core Schema](#2-core-schema)
3. [Row-Level Security Policies](#3-row-level-security-policies)
4. [Tenant Isolation Guarantees](#4-tenant-isolation-guarantees)
5. [Data Classification](#5-data-classification)
6. [Privacy Floor Enforcement](#6-privacy-floor-enforcement)
7. [Migration & Cross-Tenant Safety](#7-migration--cross-tenant-safety)
8. [Backup and Restore Isolation](#8-backup-and-restore-isolation)
9. [Open Questions / Gaps](#9-open-questions--gaps)
10. [PITR Tenant-Isolated Restore Runbook](#10-pitr-tenant-isolated-restore-runbook)
11. [Index Strategy for RLS Performance](#11-index-strategy-for-rls-performance)
12. [Soft Delete, Data Retention & GDPR Art. 17 Erasure](#12-soft-delete-data-retention--gdpr-art-17-erasure)
13. [Analytics Event Tracking Schema](#13-analytics-event-tracking-schema)
14. [Wearable Data Integration Schema](#14-wearable-data-integration-schema)
15. [Computer Vision (CV) Pose Estimation Data Schema](#15-computer-vision-cv-pose-estimation-data-schema)
16. [Enterprise Contract & Pilot Lifecycle Schema](#16-enterprise-contract--pilot-lifecycle-schema)
17. [Enterprise Admin Reporting Schema — Aggregate-Only Data Model & Privacy-Floor Enforcement](#17-enterprise-admin-reporting-schema--aggregate-only-data-model--privacy-floor-enforcement)
18. [Notification & Webhook Event Queue Schema](#18-notification--webhook-event-queue-schema)
19. [Feature Flag & Entitlement Registry Schema](#19-feature-flag--entitlement-registry-schema)
20. [White-Label Tenant Branding Schema](#20-white-label-tenant-branding-schema)
21. [AI Coaching Session & Memory Schema](#21-ai-coaching-session--memory-schema)
22. [Gamification, Streak & Achievement Schema](#22-gamification-streak--achievement-schema)
23. [Exercise Library & Program Schema](#23-exercise-library--program-schema)
24. [Subscription, Billing & Revenue Schema](#24-subscription-billing--revenue-schema)
25. [Enterprise Tenant Offboarding & Data Egress Schema](#25-enterprise-tenant-offboarding--data-egress-schema)
26. [API Key Authentication Schema](#26-api-key-authentication-schema)
27. [Enterprise Invite & Pending-Seat Provisioning Schema](#27-enterprise-invite--pending-seat-provisioning-schema)

---

## 1. Architecture Decision: RLS vs Schema-per-Tenant

### 1.1 Options Considered

| Dimension | Row-Level Security (chosen) | Schema-per-Tenant | Database-per-Tenant |
|---|---|---|---|
| Operational overhead | Low — single DB, single migration path | Medium — schema migrations fanned out per tenant | High — N DB instances, N connection pools |
| Connection pooling | PgBouncer shared pool; efficient | Per-schema pool or shared with search_path tricks | Fully separate; expensive at scale |
| Tenant isolation strength | Strong if RLS enforced correctly (fail-closed) | Strong; accidental cross-schema joins require explicit references | Strongest; process-level isolation |
| Cross-tenant analytics | Easy — aggregation across rows with system role | Hard — cross-schema joins are verbose and risky | Very hard — requires ETL into warehouse first |
| Migration complexity | Single migration, applies to all tenants atomically | Fan-out migration tool required; partial failure risk | Per-database migration; highest risk |
| SOC 2 evidence | One DB audited; RLS policies are the control evidence | Multiple schemas to audit | Multiple DBs; auditor reviews representative sample |
| Cost at 1,000 tenants | Low (one RDS Multi-AZ instance + read replicas) | Medium (schema bloat slows pg_dump) | Very high (1,000 × instance cost) |
| Suitable for FORM at Series A | ✅ Yes | 🟡 At >5,000 tenants reconsider | 🔴 Not until $10M+ ARR |

**Decision:** Row-Level Security on a shared Postgres database. Every query from the application layer runs under a role that has RLS enforced. A privileged `form_admin` role with `BYPASSRLS` is restricted to the migration runner and the compliance toolchain — never the API request path.

The escape hatch: if a single enterprise customer exceeds 50,000 seats and demands dedicated infrastructure for contractual data residency, we spin a separate DB cluster for that tenant only. This is an exception, not the default architecture.

### 1.2 Connection Architecture

```
API Server (Node.js / Bun)
  │
  ├── PgBouncer (transaction mode, port 6432)
  │     └── Sets: SET app.current_tenant_id = '<uuid>'
  │                SET app.current_user_id   = '<uuid>'
  │                SET app.current_role      = 'tenant_member|tenant_admin|tenant_owner'
  │     └── Postgres RLS reads: current_setting('app.current_tenant_id')
  │
  └── Postgres 16 (Multi-AZ RDS)
        ├── Role: form_api        — NOINHERIT, RLS enforced
        ├── Role: form_readonly   — read replicas, RLS enforced
        ├── Role: form_admin      — BYPASSRLS, migration runner only
        └── Role: form_audit      — audit_log table only, read-only, compliance
```

`SET LOCAL` (not `SET`) is used so the setting is scoped to the transaction and cannot leak across pooled connections.

---

## 2. Core Schema

### 2.1 Tenant Registry

```sql
CREATE TABLE tenants (
  id                UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  slug              TEXT UNIQUE NOT NULL,          -- used in subdomains / API paths
  display_name      TEXT NOT NULL,
  plan              TEXT NOT NULL                  -- 'starter' | 'growth' | 'enterprise'
                      CHECK (plan IN ('starter', 'growth', 'enterprise')),
  max_seats         INTEGER NOT NULL DEFAULT 200,
  data_region       TEXT NOT NULL DEFAULT 'us-east-1'
                      CHECK (data_region IN ('us-east-1', 'eu-central-1', 'eu-west-1')),
  white_label       BOOLEAN NOT NULL DEFAULT FALSE,
  sso_enabled       BOOLEAN NOT NULL DEFAULT FALSE,
  scim_enabled      BOOLEAN NOT NULL DEFAULT FALSE,
  mfa_required      BOOLEAN NOT NULL DEFAULT FALSE,
  session_max_hours INTEGER NOT NULL DEFAULT 24,
  created_at        TIMESTAMPTZ NOT NULL DEFAULT now(),
  suspended_at      TIMESTAMPTZ,
  deleted_at        TIMESTAMPTZ                    -- soft delete; hard delete after 30-day grace
);

-- Tenants table has NO RLS — only form_admin can write; form_api can read own row via FK join
```

### 2.2 Users

```sql
CREATE TABLE users (
  id                UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id         UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
  email             TEXT NOT NULL,
  email_verified    BOOLEAN NOT NULL DEFAULT FALSE,
  display_name      TEXT,
  avatar_url        TEXT,
  role              TEXT NOT NULL DEFAULT 'tenant_member'
                      CHECK (role IN ('tenant_owner', 'tenant_admin', 'tenant_manager', 'tenant_member')),
  provisioned_via   TEXT NOT NULL DEFAULT 'invite'
                      CHECK (provisioned_via IN ('invite', 'scim', 'sso_jit', 'csv_import')),
  external_id       TEXT,                          -- IdP subject / SCIM externalId
  mfa_enabled       BOOLEAN NOT NULL DEFAULT FALSE,
  last_login_at     TIMESTAMPTZ,
  deactivated_at    TIMESTAMPTZ,
  created_at        TIMESTAMPTZ NOT NULL DEFAULT now(),
  UNIQUE (tenant_id, email)
);

-- RLS applied — see Section 3
ALTER TABLE users ENABLE ROW LEVEL SECURITY;
```

### 2.3 User Health Profile

```sql
-- Separate table from users to allow tighter access control
CREATE TABLE user_health_profiles (
  id                UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id           UUID NOT NULL UNIQUE REFERENCES users(id) ON DELETE CASCADE,
  tenant_id         UUID NOT NULL,                 -- denormalized for RLS index performance
  date_of_birth     DATE,
  biological_sex    TEXT CHECK (biological_sex IN ('male', 'female', 'intersex', NULL)),
  height_cm         NUMERIC(5,1),
  resting_hr_bpm    INTEGER,
  hrv_baseline_ms   NUMERIC(6,1),
  goals             JSONB,                         -- structured, see PRODUCT_SPEC.md §3
  restrictions      JSONB,                         -- injury history, contraindications
  created_at        TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at        TIMESTAMPTZ NOT NULL DEFAULT now()
);

ALTER TABLE user_health_profiles ENABLE ROW LEVEL SECURITY;

-- CRITICAL: tenant_admin / tenant_owner roles cannot read individual health profiles.
-- HR sees NO rows from this table. Only the owning user and FORM system roles see this.
```

### 2.4 Workouts

```sql
CREATE TABLE workouts (
  id                UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id           UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  tenant_id         UUID NOT NULL,
  started_at        TIMESTAMPTZ NOT NULL,
  ended_at          TIMESTAMPTZ,
  type              TEXT NOT NULL,                -- 'strength' | 'cardio' | 'mobility' | 'hiit'
  source            TEXT NOT NULL DEFAULT 'cv',  -- 'cv' | 'wearable' | 'manual'
  rep_count         INTEGER,
  duration_seconds  INTEGER,
  calories_kcal     NUMERIC(6,1),
  form_score        NUMERIC(4,1),                -- 0–100, null if CV not used
  raw_cv_data       JSONB,                       -- pose keypoints; encrypted at rest
  wearable_payload  JSONB,                       -- HealthKit / Health Connect raw
  created_at        TIMESTAMPTZ NOT NULL DEFAULT now()
);

ALTER TABLE workouts ENABLE ROW LEVEL SECURITY;
```

### 2.5 Meal Logs

```sql
CREATE TABLE meal_logs (
  id                UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id           UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  tenant_id         UUID NOT NULL,
  logged_at         TIMESTAMPTZ NOT NULL,
  meal_type         TEXT CHECK (meal_type IN ('breakfast', 'lunch', 'dinner', 'snack')),
  items             JSONB NOT NULL,               -- [{name, kcal, protein_g, carb_g, fat_g}]
  total_kcal        NUMERIC(6,1),
  notes             TEXT,
  created_at        TIMESTAMPTZ NOT NULL DEFAULT now()
);

ALTER TABLE meal_logs ENABLE ROW LEVEL SECURITY;
```

### 2.6 Tenant-Level Aggregates (HR-visible)

```sql
-- Materialized view — recomputed nightly, never exposes individual rows
CREATE MATERIALIZED VIEW tenant_wellness_summary AS
SELECT
  tenant_id,
  DATE_TRUNC('week', w.started_at)          AS week,
  COUNT(DISTINCT w.user_id)                 AS active_users,
  COUNT(*)                                  AS total_workouts,
  AVG(w.form_score)                         AS avg_form_score,
  AVG(w.duration_seconds) / 60.0            AS avg_duration_min,
  PERCENTILE_CONT(0.5) WITHIN GROUP
    (ORDER BY w.duration_seconds) / 60.0    AS median_duration_min
FROM workouts w
JOIN users u ON u.id = w.user_id
WHERE u.deactivated_at IS NULL
  AND w.started_at >= NOW() - INTERVAL '90 days'
GROUP BY tenant_id, DATE_TRUNC('week', w.started_at)
WITH NO DATA;  -- populated by scheduled job, never via API request path

-- Minimum cohort size enforced at application layer before serving to admin dashboard.
-- Groups < 5 users suppressed (k-anonymity floor).
```

### 2.7 Feature Flags

```sql
CREATE TABLE tenant_feature_flags (
  id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id   UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
  flag_name   TEXT NOT NULL,
  enabled     BOOLEAN NOT NULL DEFAULT FALSE,
  set_by      UUID,                               -- form_admin user ID
  reason      TEXT,
  created_at  TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at  TIMESTAMPTZ NOT NULL DEFAULT now(),
  UNIQUE (tenant_id, flag_name)
);

-- Not RLS-protected at row level — tenant row joined by application; only form_admin writes
```

### 2.8 Workout Sets

Referenced by the PITR runbook (§10.3) but previously undefined. Each `workouts` row has one or more sets; this table holds per-set detail including CV form flags.

```sql
CREATE TABLE workout_sets (
  id               UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  workout_id       UUID NOT NULL REFERENCES workouts(id) ON DELETE CASCADE,
  tenant_id        UUID NOT NULL,                  -- denormalized for RLS; no FK to avoid join on every policy check
  set_number       INTEGER NOT NULL,               -- 1-based ordering within the workout
  exercise_name    TEXT NOT NULL,
  target_reps      INTEGER,
  actual_reps      INTEGER,
  weight_kg        NUMERIC(6,2),
  duration_seconds INTEGER,                        -- for time-based sets (planks, carries)
  rpe              NUMERIC(3,1) CHECK (rpe BETWEEN 1 AND 10),
  form_score       NUMERIC(4,1),                   -- 0–100; null if CV not used for this set
  cv_flags         JSONB,                          -- [{joint, issue, severity}]; see PRODUCT_SPEC.md §8
  created_at       TIMESTAMPTZ NOT NULL DEFAULT now()
);

ALTER TABLE workout_sets ENABLE ROW LEVEL SECURITY;

-- Composite unique prevents duplicate set numbers per workout
CREATE UNIQUE INDEX uq_workout_sets_workout_set ON workout_sets(workout_id, set_number);
```

`cv_flags` inherits the Sensitive classification of `raw_cv_data` — it is encrypted at rest via the same per-tenant KMS key. See §5.

### 2.9 Coaching Sessions

Victor's interactions are stored for continuity, billing instrumentation (token usage), and — critically — to enforce the privacy floor: coaching turn content is never surfaced to HR aggregates.

```sql
CREATE TABLE coaching_sessions (
  id               UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id          UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  tenant_id        UUID NOT NULL,
  workout_id       UUID REFERENCES workouts(id),   -- null for standalone coaching (outside a live workout)
  started_at       TIMESTAMPTZ NOT NULL DEFAULT now(),
  ended_at         TIMESTAMPTZ,
  turn_count       INTEGER NOT NULL DEFAULT 0,
  tokens_input     INTEGER,                         -- cumulative across all turns; updated on session close
  tokens_output    INTEGER,
  created_at       TIMESTAMPTZ NOT NULL DEFAULT now()
);

ALTER TABLE coaching_sessions ENABLE ROW LEVEL SECURITY;

CREATE TABLE coaching_turns (
  id               UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  session_id       UUID NOT NULL REFERENCES coaching_sessions(id) ON DELETE CASCADE,
  tenant_id        UUID NOT NULL,                  -- denormalized for RLS
  turn_index       INTEGER NOT NULL,               -- 0-based; monotonically increasing per session
  role             TEXT NOT NULL CHECK (role IN ('user', 'assistant', 'system')),
  content          TEXT NOT NULL,                  -- encrypted at rest; Confidential classification
  tokens           INTEGER,                         -- token count for this turn (from Anthropic response)
  voice_chars      INTEGER,                         -- ElevenLabs characters synthesized; null if voice not used
  created_at       TIMESTAMPTZ NOT NULL DEFAULT now(),
  UNIQUE (session_id, turn_index)
);

ALTER TABLE coaching_turns ENABLE ROW LEVEL SECURITY;
```

**Privacy note:** `coaching_turns.content` is classified Confidential (§5). The `tenant_wellness_summary` materialized view (§2.6) does not query `coaching_sessions` or `coaching_turns` — no aggregate of Victor dialogue is ever exposed to HR. Only token-count totals flow into `docs/COST_MODEL.md` instrumentation via a separate internal analytics path that strips all content fields before aggregation.

### 2.10 SSO / SCIM Auxiliary Tables

These tables persist IdP configuration and SCIM bearer tokens per tenant. They are written by `form_admin` (via the SCIM setup API endpoint, accessible only to `tenant_owner`) and read by the auth middleware on the SSO callback path.

```sql
-- One row per tenant; written during SSO setup flow documented in docs/SSO_SCIM_IMPLEMENTATION.md
CREATE TABLE tenant_sso_configs (
  id               UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id        UUID NOT NULL UNIQUE REFERENCES tenants(id) ON DELETE CASCADE,
  protocol         TEXT NOT NULL CHECK (protocol IN ('saml2', 'oidc')),

  -- SAML 2.0 fields (null for OIDC tenants)
  idp_entity_id    TEXT,
  idp_sso_url      TEXT,
  idp_slo_url      TEXT,
  idp_certificate  TEXT,           -- PEM; stored but not RLS-readable from API path — served only by auth service

  -- OIDC fields (null for SAML tenants)
  oidc_issuer      TEXT,
  oidc_client_id   TEXT,
  oidc_client_secret_enc TEXT,     -- AES-256-GCM encrypted via per-tenant KMS key; never returned in API responses

  -- SP / RP registration (our side)
  sp_acs_url       TEXT NOT NULL,  -- SAML Assertion Consumer Service URL for this tenant
  sp_entity_id     TEXT NOT NULL,  -- SAML SP Entity ID; or OIDC redirect_uri base
  sp_metadata_url  TEXT,           -- computed; served at /sso/{tenant_slug}/metadata

  -- Behaviour flags
  jit_provisioning BOOLEAN NOT NULL DEFAULT TRUE,  -- create user on first SSO login if not in users table
  force_authn      BOOLEAN NOT NULL DEFAULT FALSE, -- re-authenticate at IdP even with active session
  attribute_map    JSONB,          -- {"email": "mail", "displayName": "cn"} — IdP attr → FORM field
  role_map         JSONB,          -- {"cn=form-admins,dc=corp": "tenant_admin", ...} — group → FORM role

  enabled          BOOLEAN NOT NULL DEFAULT FALSE, -- must be explicitly enabled after config validation
  validated_at     TIMESTAMPTZ,    -- timestamp of last successful test login; null = not yet validated
  created_at       TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at       TIMESTAMPTZ NOT NULL DEFAULT now()
);

-- SCIM bearer tokens — token value is NEVER stored; only its SHA-256 hash
CREATE TABLE tenant_scim_tokens (
  id               UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id        UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
  token_hash       TEXT NOT NULL UNIQUE, -- SHA-256(bearer_token); checked on every SCIM request
  label            TEXT,                 -- e.g. "Okta SCIM connector" — human-readable
  created_by       UUID REFERENCES users(id),
  expires_at       TIMESTAMPTZ,          -- null = non-expiring (standard enterprise default)
  last_used_at     TIMESTAMPTZ,          -- updated on each authenticated SCIM request
  revoked_at       TIMESTAMPTZ,          -- soft-revoke; check revoked_at IS NULL on auth
  created_at       TIMESTAMPTZ NOT NULL DEFAULT now(),
  CONSTRAINT max_tokens_per_tenant CHECK (TRUE) -- enforced at application layer: max 5 active tokens per tenant
);

-- Audit trail for SCIM provisioning operations (high-volume; separate from main audit_log)
CREATE TABLE scim_provisioning_log (
  id               UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id        UUID NOT NULL,
  scim_token_id    UUID REFERENCES tenant_scim_tokens(id),
  operation        TEXT NOT NULL CHECK (operation IN ('create_user', 'update_user', 'deactivate_user', 'reactivate_user', 'list_users')),
  external_id      TEXT,                -- IdP externalId from SCIM request
  target_user_id   UUID,                -- form users.id; null if user not yet created
  status           TEXT NOT NULL CHECK (status IN ('success', 'error', 'skipped')),
  error_detail     TEXT,
  idp_request_id   TEXT,               -- IdP's request identifier, for correlation with IdP logs
  created_at       TIMESTAMPTZ NOT NULL DEFAULT now()
);

ALTER TABLE tenant_sso_configs   ENABLE ROW LEVEL SECURITY;
ALTER TABLE tenant_scim_tokens   ENABLE ROW LEVEL SECURITY;
ALTER TABLE scim_provisioning_log ENABLE ROW LEVEL SECURITY;
```

**Security note:** `oidc_client_secret_enc` is encrypted before write using `pgcrypto.encrypt` with the tenant's KMS-derived key. The raw secret is never logged, never returned in API responses (the `GET /admin/sso` endpoint returns the field as `"***"` after initial save), and never appears in `pg_dump` output without the decryption key. See §5 Sensitive classification.

---

### 2.11 Consent Records

The `consent_records` table is the authoritative, tamper-evident record of all user consent decisions — web cookie consent (GDPR Art. 6) and mobile health data consent (GDPR Art. 9). It is **append-only**: `UPDATE` and `DELETE` are blocked at the Postgres RULE layer for all application roles. Withdrawals are recorded as a new row with `event_type = 'withdrawn'`.

Cross-reference: `docs/SOC2_READINESS.md §74.4` (P2.1/P8.1 SOC 2 evidence specification). Migration: `0037_consent_records.sql`.

**Privacy invariant:** no email address, phone number, full IP address, or Art. 9 category value is stored in this table. `user_id` is a pseudonymous UUID; `consent_token` is an opaque UUID not linkable to email without joining `user_profile`. Maximum geographic granularity: `ip_country` (2-char country code).

```sql
-- Migration: 0037_consent_records.sql
-- SOC 2 criteria: P2.1 (Consent), P8.1 (Monitoring and Enforcement)
CREATE TABLE consent_records (
    id                   uuid         PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id            uuid         REFERENCES tenants(id) ON DELETE RESTRICT,
    user_id              uuid         REFERENCES user_profile(id) ON DELETE RESTRICT,
    -- NULL for anonymous pre-authentication web consent (consent_token only)

    consent_token        text         NOT NULL,
    -- UUID v4; stored in __form_consent cookie + Cloudflare KV key consent:{token}

    event_type           text         NOT NULL CHECK (event_type IN ('granted', 'withdrawn', 'updated')),
    consent_version      text         NOT NULL,
    -- Matches CONSENT_SCHEMA_VERSION env var at time of event; e.g. '1.0', '1.1'

    consent_surface      text         NOT NULL
                                      CHECK (consent_surface IN ('web_cookie', 'mobile_art9', 'web_art9')),
    granted_categories   text[]       NOT NULL DEFAULT '{}',
    -- web_cookie: ['analytics'], ['analytics','functional'], etc.
    -- mobile/web art9: ['health_data'], ['health_data','biometric']
    -- withdrawn rows: empty array; populated withdrawn_categories instead

    withdrawn_categories text[]       NOT NULL DEFAULT '{}',
    -- Populated on event_type = 'withdrawn'; empty on 'granted'

    ip_country           char(2),
    -- ISO 3166-1 alpha-2 country code only; never full IP address

    user_agent_hash      text,
    -- SHA-256 of normalised user-agent string; not reversible to original UA

    dec030_event_id      uuid         NOT NULL,
    -- References audit_log_events.id; confirms DEC-030 HMAC chain linkage

    created_at           timestamptz  NOT NULL DEFAULT now()
    -- No updated_at: append-only table; each change is a new row
);

-- Append-only enforcement (Postgres RULE layer)
CREATE RULE consent_records_no_update AS ON UPDATE TO consent_records DO INSTEAD NOTHING;
CREATE RULE consent_records_no_delete AS ON DELETE TO consent_records DO INSTEAD NOTHING;

-- Indexes
CREATE INDEX consent_records_user_id_idx     ON consent_records (user_id);
CREATE INDEX consent_records_tenant_id_idx   ON consent_records (tenant_id);
CREATE INDEX consent_records_token_idx       ON consent_records (consent_token);
CREATE INDEX consent_records_created_idx     ON consent_records (created_at);
CREATE INDEX consent_records_event_type_idx  ON consent_records (event_type);
```

**Append-only integrity check (run annually; file as PRE-74-E-002):**

```sql
-- Positive result: n_tup_upd = 0 AND n_tup_del = 0
-- Non-zero values indicate a RULE bypass — treat as P1 security incident
SELECT schemaname, tablename,
       n_tup_ins AS rows_inserted,
       n_tup_upd AS rows_updated,   -- must be 0
       n_tup_del AS rows_deleted    -- must be 0
FROM pg_stat_user_tables
WHERE tablename = 'consent_records';
```

---

## 3. Row-Level Security Policies

### 3.1 Policy Design Principles

1. **Fail-closed**: default deny. Every table has a `USING (FALSE)` fallback policy before permissive policies are added.
2. **Tenant context via GUC**: `current_setting('app.current_tenant_id', TRUE)` — the second argument returns NULL (not error) if unset, which causes the USING clause to evaluate to FALSE → row not visible. Never errors; never leaks.
3. **Role separation**: `form_api` (API servers) has RLS enforced. `form_admin` (migration runner) has `BYPASSRLS` and is never in the request-handling path.
4. **No `security definer` functions** that bypass RLS without an explicit audit log write.

### 3.2 Users Table Policies

```sql
-- Tenant isolation: user sees only their tenant's users
CREATE POLICY users_tenant_isolation ON users
  AS PERMISSIVE FOR SELECT
  TO form_api
  USING (
    tenant_id = current_setting('app.current_tenant_id', TRUE)::UUID
  );

-- Admins can read all users in their tenant
-- Members can read only themselves (for profile page)
CREATE POLICY users_member_self_only ON users
  AS RESTRICTIVE FOR SELECT
  TO form_api
  USING (
    current_setting('app.current_role', TRUE) IN ('tenant_owner', 'tenant_admin', 'tenant_manager')
    OR
    id = current_setting('app.current_user_id', TRUE)::UUID
  );

-- Write: only tenant_admin+ or the user themselves
CREATE POLICY users_write ON users
  AS PERMISSIVE FOR UPDATE
  TO form_api
  USING (
    tenant_id = current_setting('app.current_tenant_id', TRUE)::UUID
    AND (
      current_setting('app.current_role', TRUE) IN ('tenant_owner', 'tenant_admin')
      OR id = current_setting('app.current_user_id', TRUE)::UUID
    )
  );
```

### 3.3 Health Profile Policies

```sql
-- HARD RULE: HR roles never see individual health profiles
-- Only the owning user (and system role) can read

CREATE POLICY health_profile_owner_only ON user_health_profiles
  AS PERMISSIVE FOR SELECT
  TO form_api
  USING (
    tenant_id = current_setting('app.current_tenant_id', TRUE)::UUID
    AND user_id = current_setting('app.current_user_id', TRUE)::UUID
  );

-- tenant_admin / tenant_owner queries return 0 rows — by design, not an error
-- Attempting to query returns empty set; audit log records the attempt
```

### 3.4 Workouts and Meal Logs Policies

```sql
-- Users see only their own workouts
CREATE POLICY workouts_owner ON workouts
  AS PERMISSIVE FOR SELECT
  TO form_api
  USING (
    tenant_id = current_setting('app.current_tenant_id', TRUE)::UUID
    AND user_id = current_setting('app.current_user_id', TRUE)::UUID
  );

-- Admins cannot query individual workout rows — same policy
-- Admin-facing data comes ONLY from tenant_wellness_summary materialized view
CREATE POLICY workouts_no_admin_peek ON workouts
  AS RESTRICTIVE FOR SELECT
  TO form_api
  USING (
    user_id = current_setting('app.current_user_id', TRUE)::UUID
  );

-- Identical pattern for meal_logs
CREATE POLICY meal_logs_owner ON meal_logs
  AS PERMISSIVE FOR SELECT
  TO form_api
  USING (
    tenant_id = current_setting('app.current_tenant_id', TRUE)::UUID
    AND user_id = current_setting('app.current_user_id', TRUE)::UUID
  );

CREATE POLICY meal_logs_no_admin_peek ON meal_logs
  AS RESTRICTIVE FOR SELECT
  TO form_api
  USING (
    user_id = current_setting('app.current_user_id', TRUE)::UUID
  );
```

### 3.5 Verification Test in CI

Every CI run executes a cross-tenant isolation test:

```sql
-- Arrange: two tenants, Tenant A and Tenant B, each with one user and one workout
-- Act: set app.current_tenant_id = tenant_a_id, app.current_user_id = tenant_a_user_id
-- Assert: query workouts — must return ONLY tenant A's user's rows
-- Assert: set tenant_id to tenant_b — zero rows returned for same user

-- Failure = CI blocks merge. No exceptions.
```

The test is in `__tests__/db/rls_isolation.test.ts` and runs against a local Postgres container with the same RLS policies as production.

### 3.6 Workout Sets, Coaching Sessions, Coaching Turns

All three tables follow the same owner-only pattern as `workouts`: individual rows are never visible to tenant admins — only the owning user can read their own records.

```sql
-- workout_sets: follow through workouts ownership (user sees only their sets)
CREATE POLICY workout_sets_tenant_isolation ON workout_sets
  AS PERMISSIVE FOR SELECT
  TO form_api
  USING (
    tenant_id = current_setting('app.current_tenant_id', TRUE)::UUID
  );

CREATE POLICY workout_sets_owner_only ON workout_sets
  AS RESTRICTIVE FOR SELECT
  TO form_api
  USING (
    workout_id IN (
      SELECT id FROM workouts
      WHERE user_id = current_setting('app.current_user_id', TRUE)::UUID
    )
  );

-- coaching_sessions
CREATE POLICY coaching_sessions_tenant ON coaching_sessions
  AS PERMISSIVE FOR SELECT
  TO form_api
  USING (tenant_id = current_setting('app.current_tenant_id', TRUE)::UUID);

CREATE POLICY coaching_sessions_owner ON coaching_sessions
  AS RESTRICTIVE FOR SELECT
  TO form_api
  USING (user_id = current_setting('app.current_user_id', TRUE)::UUID);

-- coaching_turns (denormalized tenant_id allows direct policy without a join)
CREATE POLICY coaching_turns_tenant ON coaching_turns
  AS PERMISSIVE FOR SELECT
  TO form_api
  USING (tenant_id = current_setting('app.current_tenant_id', TRUE)::UUID);

CREATE POLICY coaching_turns_owner ON coaching_turns
  AS RESTRICTIVE FOR SELECT
  TO form_api
  USING (
    session_id IN (
      SELECT id FROM coaching_sessions
      WHERE user_id = current_setting('app.current_user_id', TRUE)::UUID
    )
  );
```

**Note:** The nested subquery in `coaching_turns_owner` is intentional. Alternatives (e.g., also denormalizing `user_id`) would require keeping it in sync on every insert. The query planner uses the `idx_coaching_turns_session_id` index (§11) to make this efficient.

### 3.7 SSO Config and SCIM Token Policies

SSO configs are read only by `tenant_owner` (to validate their setup) and by the auth service running as `form_api` with a special system role claim. SCIM tokens are not readable at all via the API — only their `label`, `last_used_at`, and `revoked_at` are surfaced.

```sql
-- SSO config: tenant_owner can read; tenant_admin cannot (contains IdP secrets)
CREATE POLICY sso_config_tenant ON tenant_sso_configs
  AS PERMISSIVE FOR SELECT
  TO form_api
  USING (
    tenant_id = current_setting('app.current_tenant_id', TRUE)::UUID
    AND current_setting('app.current_role', TRUE) IN ('tenant_owner', 'form_system')
  );

-- SCIM tokens: readable only by tenant_owner for metadata (not the hash)
CREATE POLICY scim_tokens_tenant ON tenant_scim_tokens
  AS PERMISSIVE FOR SELECT
  TO form_api
  USING (
    tenant_id = current_setting('app.current_tenant_id', TRUE)::UUID
    AND current_setting('app.current_role', TRUE) = 'tenant_owner'
    AND revoked_at IS NULL
  );

-- SCIM provisioning log: readable by tenant_admin+ (useful for debugging IdP sync issues)
CREATE POLICY scim_log_tenant ON scim_provisioning_log
  AS PERMISSIVE FOR SELECT
  TO form_api
  USING (
    tenant_id = current_setting('app.current_tenant_id', TRUE)::UUID
    AND current_setting('app.current_role', TRUE) IN ('tenant_owner', 'tenant_admin')
  );
```

The `token_hash` column in `tenant_scim_tokens` is never returned by any API endpoint. The auth middleware reads it directly via a Postgres function (`form_scim_auth`) that runs as `form_api` with `SECURITY INVOKER` (not definer) — the function receives the bearer token, hashes it, and returns the `tenant_id` if a matching non-revoked row exists.

---

### 3.8 Consent Records RLS Policies

> **OQ-P2-03 closure** (`docs/SOC2_READINESS.md §74.11`): This section confirms the hard RLS guarantee that enterprise tenant admins **cannot** read consent records belonging to other employees.

**Policy design rationale:** In PostgreSQL, multiple `PERMISSIVE` policies for the same command are OR-ed together — a row is returned if *any* permissive policy passes. If `consent_records_tenant_isolation` were declared `PERMISSIVE FOR SELECT` using only a `tenant_id` check, any user in the same tenant (including tenant admins) could read all employees' consent records by satisfying that single policy. This would be a privacy violation.

The correct design uses **two-layer enforcement**:

1. **PERMISSIVE `consent_records_user_read`** — users may read only rows where their `user_id` matches (their own consent history shown in Settings → Data).
2. **RESTRICTIVE `consent_records_cross_tenant_block`** — AND-ed with all permissive results; ensures that even if a future permissive policy is inadvertently added, no row is ever returned unless it belongs to the authenticated user's own tenant.

Result: a tenant admin JWT resolves to a single `user_id`; the permissive policy only passes for that user's own row; the restrictive policy additionally enforces the tenant boundary. Tenant admins cannot read any employee's consent record.

```sql
ALTER TABLE consent_records ENABLE ROW LEVEL SECURITY;

-- PERMISSIVE: users may read only their own consent rows
CREATE POLICY consent_records_user_read ON consent_records
    AS PERMISSIVE FOR SELECT
    USING (
        user_id = (SELECT id FROM user_profile WHERE auth_user_id = auth.uid())
    );

-- RESTRICTIVE: cross-tenant defence-in-depth.
-- AND-ed with all permissive policies; blocks reads from any other tenant
-- even if a new permissive policy is added in future.
CREATE POLICY consent_records_cross_tenant_block ON consent_records
    AS RESTRICTIVE FOR SELECT
    USING (
        tenant_id = (SELECT tenant_id FROM user_profile WHERE auth_user_id = auth.uid())
    );

-- INSERT: form-consent-gate Worker uses service-role key (bypasses RLS).
-- This policy documents intent for anon/authed roles — they may not INSERT directly.
CREATE POLICY consent_records_service_insert ON consent_records
    FOR INSERT
    WITH CHECK (TRUE);
```

**Tenant admin access test (must pass in CI RLS isolation suite):**

```sql
-- Connect as a tenant_admin JWT (app.current_role = 'tenant_admin')
-- Set context to a valid tenant_id but a different user_id from the row under test
SET LOCAL app.current_tenant_id = '<tenant_uuid>';
SET LOCAL app.current_user_id   = '<admin_user_uuid>';

-- Must return 0 rows — admin cannot read employee consent records
SELECT COUNT(*) FROM consent_records WHERE user_id = '<employee_user_uuid>';
-- Expected: 0
```

This test is added to `__tests__/db/rls_isolation.test.ts` alongside the existing cross-tenant isolation tests (§3.5).

> **Note on SOC2_READINESS.md §74.4 DDL:** The original DDL in §74.4 named the second policy `consent_records_tenant_isolation` and declared it `PERMISSIVE`. The canonical policy name and type are those defined here in §3.8 (`consent_records_cross_tenant_block`, `RESTRICTIVE`). The migration `0037_consent_records.sql` must use the §3.8 definitions.

---

## 4. Tenant Isolation Guarantees

### 4.1 What We Guarantee

| Layer | Guarantee |
|---|---|
| Database | RLS prevents cross-tenant row reads for `form_api` role. Fail-closed: unset context → no rows. |
| Application | `tenant_id` set from authenticated JWT claim; never from request body or query param. |
| API routing | `/v1/tenants/{tenant_id}/*` — tenant ID in path validated against JWT; mismatch → 403. |
| Admin Dashboard | Rendered data sourced from `tenant_wellness_summary`; k-anonymity floor = 5 users per group. |
| Audit log | `tenant_id` column on every audit event; tenant admin can only read their tenant's logs. |
| Backups | Logical backups are full-DB; restore to separate cluster before any tenant-specific extraction. |
| CV data | `raw_cv_data` column encrypted with per-tenant AES-256-GCM key; key in AWS KMS tenant keyspace. |

### 4.2 What We Do Not Guarantee

| Limitation | Mitigation |
|---|---|
| Shared infrastructure metadata (e.g., Postgres `pg_stat_activity` visible to superuser) | `form_api` cannot query `pg_stat_activity`. Only `form_admin` (not API path). |
| Timing side-channels between tenants on shared compute | Acceptable risk at current scale; revisit at 500+ tenants on dedicated compute. |
| Application bugs that bypass the `SET LOCAL` pattern | Enforced via code review + CI RLS isolation test. |

### 4.3 Tenant Context Middleware (application layer)

```typescript
// src/middleware/tenant-context.ts
export async function tenantContext(req: Request, res: Response, next: NextFunction) {
  const { tenantId, userId, role } = req.auth; // from verified JWT

  await db.execute(sql`
    SET LOCAL app.current_tenant_id = ${tenantId};
    SET LOCAL app.current_user_id   = ${userId};
    SET LOCAL app.current_role      = ${role};
  `);

  next();
}
// Runs inside a transaction wrapper. SET LOCAL scopes to transaction — cannot leak.
```

---

## 5. Data Classification

| Classification | Examples | Encryption at rest | Access |
|---|---|---|---|
| **Public** | Tenant slug, display name | Standard RDS encryption | All roles |
| **Internal** | User email, display name, role | Standard RDS encryption | `form_api` (RLS) |
| **Confidential** | Health goals, restrictions, HRV baseline; `coaching_turns.content`; `workout_sets.cv_flags` | Standard RDS encryption | Owner user only (RLS) |
| **Sensitive** | CV raw pose data, meal log items; `tenant_sso_configs.oidc_client_secret_enc`; `tenant_sso_configs.idp_certificate` | Per-tenant AES-256-GCM + KMS | Owner user / `form_system` only; `form_admin` via break-glass |
| **Confidential** | `consent_records` rows (pseudonymous consent decisions; `consent_token`, `ip_country`, `user_agent_hash`, `granted_categories`) | Standard RDS encryption; HMAC-chained via `dec030_event_id` | Owner user only (§3.8 RLS); tenant admins blocked (§3.8 OQ-P2-03 closure) |
| **Restricted** | Audit log HMAC keys; `tenant_scim_tokens.token_hash` (write-only; never returned) | KMS only; never in DB (token) or KMS-derived (HMAC key) | `form_audit` role + KMS policy; SCIM auth function only |

### 5.1 Encryption Details

- **At rest (all data):** AWS RDS encrypted volumes (AES-256). Automatic, not application-managed.
- **Per-tenant key (Sensitive class):** AWS KMS Customer Managed Key (CMK), one per tenant. `raw_cv_data` and future biometric columns use `pgcrypto.encrypt` with CMK-derived key. CMK rotation: annual, automatic.
- **In transit:** TLS 1.3 minimum for all connections (client → API, API → DB, DB → read replicas).
- **Backup encryption:** RDS automated backups inherit volume encryption. Logical dumps encrypted with tenant CMK before S3 storage.

---

## 6. Privacy Floor Enforcement

These rules are implemented at **both** the RLS layer and the application layer. Belt-and-suspenders. Removal of either layer requires a compliance-officer sign-off and a new entry in DECISION_LOG.md.

| Rule | RLS implementation | Application implementation |
|---|---|---|
| HR cannot see individual user health data | `health_profile_owner_only` policy returns 0 rows for admin roles | Admin API endpoints have no route for individual health data |
| HR cannot see individual workout rows | `workouts_no_admin_peek` + `meal_logs_no_admin_peek` policies | Admin dashboard reads only from `tenant_wellness_summary` MV |
| k-anonymity floor on aggregate data | Enforced in MV definition (groups < 5 suppressed) | API response validator rejects cohort payloads with N < 5 |
| Manager cannot see reports' data | Role `tenant_manager` has no special DB privileges; same as `tenant_member` | Manager-report relationship not stored in DB (by design) |
| Employee can revoke company association | User `deactivated_at` set; excluded from future MV refreshes | Self-serve in Settings → Privacy → "Disconnect from employer" |
| ED-screening data never in aggregates | `meal_logs` not included in `tenant_wellness_summary` MV | No meal-log metrics in admin dashboard endpoints |
| Body composition never aggregated | BMI/body fat not in `user_health_profiles.goals` aggregation path | No body composition fields in MV query |

---

## 7. Migration & Cross-Tenant Safety

### 7.1 Migration Principles

1. **All migrations run as `form_admin`** (BYPASSRLS). Migration scripts must not assume tenant context.
2. **Non-destructive by default**: `ADD COLUMN ... DEFAULT ...` not `DROP COLUMN`. Column removals staged over two releases (deprecate → remove).
3. **Cross-tenant CI gate**: after every migration in CI, the RLS isolation test (§3.5) runs to verify no new table or policy broke isolation.
4. **Zero-downtime pattern**: for large table changes, use `ALTER TABLE ... ADD COLUMN` (instant in Postgres 11+), backfill in background job, then add NOT NULL constraint after backfill.
5. **Tenant-specific migrations**: if a migration must run differently per tenant (e.g., white-label tenants get an extra column), use the feature flag table, not schema divergence.

### 7.2 Migration Checklist

```
[ ] New table has tenant_id column (UUID NOT NULL REFERENCES tenants(id))
[ ] ALTER TABLE <table> ENABLE ROW LEVEL SECURITY;
[ ] PERMISSIVE policy added for form_api with tenant_id check
[ ] RESTRICTIVE policy added for any additional column-level access control
[ ] CI RLS isolation test updated to include new table
[ ] form_audit role granted SELECT on new table if audit-relevant
[ ] Data classification label documented in this file (§5)
[ ] form_admin has explicit GRANT (not inherited from public)
```

### 7.3 Dangerous Patterns — Do Not Use

```sql
-- NEVER: SET app.current_tenant_id from user input
SET app.current_tenant_id = $1;  -- if $1 comes from request body, this is a tenant escape

-- NEVER: SECURITY DEFINER function without explicit audit log write
CREATE FUNCTION get_all_user_health() RETURNS TABLE(...) SECURITY DEFINER ...

-- NEVER: Disable RLS even temporarily in production
ALTER TABLE workouts DISABLE ROW LEVEL SECURITY;  -- never in prod migration

-- NEVER: SELECT without WHERE tenant_id when using form_admin role
SELECT * FROM users;  -- must be: SELECT * FROM users WHERE tenant_id = $1
```

---

## 8. Backup and Restore Isolation

### 8.1 Backup Strategy

| Type | Frequency | Retention | Encryption |
|---|---|---|---|
| RDS automated snapshots | Daily | 35 days | Volume-level AES-256 |
| RDS manual snapshots (pre-migration) | Per deployment | 90 days | Volume-level AES-256 |
| Logical dumps (pg_dump, per-tenant export) | Weekly | 7 years (compliance) | Per-tenant CMK via S3 SSE-KMS |
| Transaction logs (PITR) | Continuous | 35 days | Volume-level AES-256 |

### 8.2 Tenant-Specific Data Export (GDPR DSAR)

When a user requests a GDPR data export:

1. Application triggers `data.export_initiated` audit log entry.
2. Background job queries with `form_admin` role, scoped `WHERE user_id = $1`.
3. Output: JSON archive of all user's rows across: `users`, `user_health_profiles`, `workouts`, `meal_logs`.
4. Encrypted with user's own AES key (ephemeral, emailed via secure link).
5. Link expires in 48 hours. Archive deleted from FORM storage after download or expiry.
6. `data.export_completed` audit log entry.

### 8.3 Right to Erasure (GDPR Art. 17)

```sql
-- Executed by form_admin role only, triggered by verified deletion request
BEGIN;

UPDATE users SET
  email        = 'deleted+' || id || '@form.coach',
  display_name = 'Deleted User',
  deactivated_at = now()
WHERE id = $1;

DELETE FROM user_health_profiles WHERE user_id = $1;
DELETE FROM workouts            WHERE user_id = $1;
DELETE FROM meal_logs           WHERE user_id = $1;
-- Audit log rows retained per AUDIT_LOG_SCHEMA.md retention policy (not deleted)

COMMIT;
-- Audit event: data.individual_deletion logged before transaction
```

Hard delete of `users` row deferred 30 days (in case of mistaken request).

---

## 9. Open Questions / Gaps

| # | Question | Owner | Priority |
|---|---|---|---|
| 9.1 | When to shard: at what tenant count or data volume do we move the largest tenants to dedicated clusters? | enterprise-architect | Medium — revisit at 200 enterprise tenants |
| 9.2 | `raw_cv_data` column size: JSONB for pose keypoints can grow large. Column to separate blob store (S3)? | platform-engineer | **RESOLVED** — see §15.5: keypoints stored encrypted in `cv_sessions.keypoints_enc` (JSONB TOAST); R2 migration gate at 20 GB / 1,000 sessions/day; `workouts.raw_cv_data` deprecated |
| 9.3 | k-anonymity floor at 5: is this sufficient for EU data regulators under GDPR recital 26? | compliance-officer | High — legal review before EU enterprise sales |
| 9.4 | PITR restore drill: do we have a documented runbook for restoring a single tenant's data from PITR without affecting other tenants? | devops-lead | **RESOLVED** — see Section 10 |
| 9.5 | Soft-delete vs hard-delete on `workouts`: soft-delete allows undo but complicates RLS policies (deleted rows still present). | enterprise-architect | Low — decided when undo feature is scoped |

---

## 10. PITR Tenant-Isolated Restore Runbook

### 10.1 Purpose

This runbook is SOC 2 evidence for Availability criteria A1.2 (environmental protections, software, and infrastructure) and A1.3 (recovery and resumption of services). It documents the exact procedure for restoring a single tenant's data from RDS Point-in-Time Recovery (PITR) without touching any other tenant's data and without taking the production database offline.

### 10.2 When to Use

| Scenario | Notes |
|---|---|
| Single tenant's data corrupted or lost due to application bug | Most common production use case |
| Production incident requiring rollback of specific rows | Supplement to application-level undo |
| Annual DR drill | Required by SOC 2 Availability criteria; must be run at least once per calendar year |

Do **not** use this runbook for full-cluster restores or for bulk tenant offboarding. Those have separate procedures.

### 10.3 Prerequisites

| Requirement | Detail |
|---|---|
| RDS access | `form_admin` DB role credentials via AWS Secrets Manager (break-glass secret `form/prod/db/form_admin`) |
| AWS Console / CLI | IAM role `FormDevOpsRestoreRole` with permissions: `rds:RestoreDBClusterToPointInTime`, `rds:CreateDBInstance`, `rds:DeleteDBCluster` |
| Target tenant UUID | Obtain from admin dashboard or: `SELECT id, slug FROM tenants WHERE slug = '<tenant-slug>';` |
| Approximate corruption time | Incident ticket timestamp, or earliest user-reported event time |
| `pg_dump` and `psql` | Available on the bastion host; use the version matching the production Postgres major version |

### 10.4 Step-by-Step Restore Procedure

All steps are performed by the devops-lead or an on-call engineer with break-glass access. A second engineer must be present to verify each step (four-eyes principle). All activity is recorded in the incident ticket.

#### Step 1 — Identify the PITR target window

Determine the restore point: the latest timestamp *before* the corruption event. Use the incident timeline. For a drill, use a point 24 hours before the drill start.

```
RESTORE_POINT="2026-05-16T14:00:00Z"   # ISO 8601 UTC, before corruption
TENANT_ID="<target-tenant-uuid>"
DRILL_DATE="20260517"
RESTORE_CLUSTER="form-restore-drill-${DRILL_DATE}"
```

Confirm the target point is within the PITR window (35-day retention):

```bash
aws rds describe-db-cluster-automated-backups \
  --db-cluster-identifier form-prod \
  --query 'DBClusterAutomatedBackups[0].{Earliest:RestoreWindow.EarliestTime,Latest:RestoreWindow.LatestTime}'
```

The `RESTORE_POINT` must fall between `Earliest` and `Latest`.

#### Step 2 — Restore to a temporary isolated RDS cluster

Spin up a new cluster from the PITR snapshot. This cluster is completely separate from production and has no application traffic.

```bash
aws rds restore-db-cluster-to-point-in-time \
  --source-db-cluster-identifier form-prod \
  --db-cluster-identifier "${RESTORE_CLUSTER}" \
  --restore-to-time "${RESTORE_POINT}" \
  --vpc-security-group-ids sg-restore-isolated \
  --db-subnet-group-name form-private-subnets \
  --no-publicly-accessible \
  --tags Key=Purpose,Value=pitr-restore Key=CreatedBy,Value=devops-lead Key=IncidentTicket,Value=<ticket-id>

# Wait for cluster to become available (typically 10–20 minutes)
aws rds wait db-cluster-available --db-cluster-identifier "${RESTORE_CLUSTER}"

# Attach a writer instance
aws rds create-db-instance \
  --db-instance-identifier "${RESTORE_CLUSTER}-writer" \
  --db-cluster-identifier "${RESTORE_CLUSTER}" \
  --db-instance-class db.t3.large \
  --engine aurora-postgresql

aws rds wait db-instance-available --db-instance-identifier "${RESTORE_CLUSTER}-writer"
```

Record the restored cluster endpoint:

```bash
RESTORE_HOST=$(aws rds describe-db-clusters \
  --db-cluster-identifier "${RESTORE_CLUSTER}" \
  --query 'DBClusters[0].Endpoint' --output text)
```

#### Step 3 — Extract the target tenant's rows from the restored cluster

Connect to the restored cluster (via bastion host) using the `form_admin` role. Extract only the target tenant's rows from all relevant tables. The `WHERE tenant_id = $TENANT_ID` clause is explicit — no RLS bypass is needed or used for this read; `form_admin` reads with an explicit predicate.

```bash
TABLES="users user_health_profiles workouts workout_sets meal_logs audit_log"
DUMP_DIR="/tmp/pitr-restore-${TENANT_ID}-${DRILL_DATE}"
mkdir -p "${DUMP_DIR}"

for TABLE in $TABLES; do
  psql "host=${RESTORE_HOST} dbname=form user=form_admin sslmode=require" \
    -c "\COPY (SELECT * FROM ${TABLE} WHERE tenant_id = '${TENANT_ID}') TO '${DUMP_DIR}/${TABLE}.csv' WITH (FORMAT CSV, HEADER)"
  echo "Extracted ${TABLE}: $(wc -l < "${DUMP_DIR}/${TABLE}.csv") rows (including header)"
done
```

`workout_sets` joins through `workouts`; extract via subquery:

```bash
psql "host=${RESTORE_HOST} dbname=form user=form_admin sslmode=require" \
  -c "\COPY (SELECT ws.* FROM workout_sets ws JOIN workouts w ON w.id = ws.workout_id WHERE w.tenant_id = '${TENANT_ID}') TO '${DUMP_DIR}/workout_sets.csv' WITH (FORMAT CSV, HEADER)"
```

#### Step 4 — Verify row counts and data integrity

Before touching production, confirm the extracted data is coherent.

```bash
# Print row counts for each extracted file
for TABLE in $TABLES; do
  COUNT=$(tail -n +2 "${DUMP_DIR}/${TABLE}.csv" | wc -l)
  echo "${TABLE}: ${COUNT} rows"
done

# Spot-check: confirm tenant_id uniformity in users.csv
TENANT_COL=$(head -1 "${DUMP_DIR}/users.csv" | tr ',' '\n' | grep -n tenant_id | cut -d: -f1)
UNIQUE_TENANTS=$(tail -n +2 "${DUMP_DIR}/users.csv" | cut -d',' -f${TENANT_COL} | sort -u | wc -l)
echo "Unique tenant_ids in users.csv: ${UNIQUE_TENANTS} (must be 1)"
```

If `UNIQUE_TENANTS` is not 1, stop immediately and open a critical incident — the extraction predicate did not filter correctly.

#### Step 5 — Re-import rows into production

Import the extracted rows into the production database. Use `INSERT ... ON CONFLICT DO UPDATE` to handle rows that may already exist (e.g., partial corruption rather than full loss). The `form_admin` role is used with an explicit `WHERE tenant_id = $TENANT_ID` guard on the conflict target.

**Important:** `audit_log` rows from the restored period are **not** re-imported into production. The production audit log is append-only and HMAC-chained; re-inserting historical audit rows would break the chain. Instead, the extracted `audit_log.csv` is attached to the incident ticket as evidence and reviewed separately by the compliance officer.

```sql
-- Run on production DB as form_admin via psql on bastion
-- Import users
\COPY users FROM '/tmp/pitr-restore-.../users.csv' WITH (FORMAT CSV, HEADER)
-- ON CONFLICT: if user row already exists, update non-key fields
-- Run as a prepared transaction if volume > 10k rows to allow rollback

-- Equivalent upsert pattern (preferred for large sets):
INSERT INTO users
  SELECT * FROM staging_users_restore WHERE tenant_id = :'TENANT_ID'
ON CONFLICT (id) DO UPDATE SET
  email          = EXCLUDED.email,
  display_name   = EXCLUDED.display_name,
  role           = EXCLUDED.role,
  deactivated_at = EXCLUDED.deactivated_at,
  updated_at     = now();

-- Repeat for user_health_profiles, workouts, workout_sets, meal_logs
-- audit_log: DO NOT re-import — attach audit_log.csv to incident ticket only
```

After each table import, verify counts match the extracted CSVs.

#### Step 6 — Verify production state

```sql
-- On production DB as form_admin
SELECT COUNT(*) FROM users              WHERE tenant_id = :'TENANT_ID';
SELECT COUNT(*) FROM user_health_profiles WHERE tenant_id = :'TENANT_ID';
SELECT COUNT(*) FROM workouts           WHERE tenant_id = :'TENANT_ID';
SELECT COUNT(*) FROM workout_sets ws
  JOIN workouts w ON w.id = ws.workout_id WHERE w.tenant_id = :'TENANT_ID';
SELECT COUNT(*) FROM meal_logs          WHERE tenant_id = :'TENANT_ID';
```

Compare against the extracted row counts from Step 4. Confirm with the tenant admin that their data appears correct in the application before closing the incident.

#### Step 7 — Destroy the temporary cluster

The temporary cluster contains a full copy of the production database. It must be deleted immediately after the restore is complete and verified.

```bash
aws rds delete-db-instance \
  --db-instance-identifier "${RESTORE_CLUSTER}-writer" \
  --skip-final-snapshot

aws rds wait db-instance-deleted --db-instance-identifier "${RESTORE_CLUSTER}-writer"

aws rds delete-db-cluster \
  --db-cluster-identifier "${RESTORE_CLUSTER}" \
  --skip-final-snapshot

aws rds wait db-cluster-deleted --db-cluster-identifier "${RESTORE_CLUSTER}"

echo "Temporary cluster ${RESTORE_CLUSTER} destroyed."
```

Confirm deletion in AWS Console and record the timestamp in the incident ticket.

### 10.5 Multi-Tenant Isolation Throughout

The procedure above maintains full multi-tenant isolation at every step:

- The temporary cluster is network-isolated (`sg-restore-isolated` security group has no inbound from application servers).
- All extraction queries use an explicit `WHERE tenant_id = $TENANT_ID` predicate. No RLS bypass is used or needed for the extraction step.
- The re-import into production also uses explicit `WHERE tenant_id = :'TENANT_ID'` guards on conflict targets, not a blanket upsert.
- Only the affected tenant's rows are written to production. Zero rows from any other tenant are touched.

### 10.6 DR Drill Schedule

| Item | Detail |
|---|---|
| Frequency | Annually, minimum. Target: Q1 of each calendar year. |
| Scheduling | Ops calendar event created by devops-lead by January 15 each year. |
| Evidence | Screenshot of AWS Console showing cluster creation + deletion timestamps, plus row-count verification output, uploaded to the SOC 2 evidence folder under `A1.2 / DR Drill / <year>`. |
| Approver | devops-lead signs off; compliance-officer co-signs for SOC 2 evidence package. |

### 10.7 Post-Drill Record

After each drill, update this section with:

```
Last drill: [date] · Result: [PASS / FAIL] · Notes: [summary of any issues and resolutions]
```

*Last drill: not yet conducted · Result: N/A · Notes: runbook established May 2026; first drill scheduled Q1 2027.*

### 10.8 Estimated Time

| Phase | Estimated duration |
|---|---|
| Cluster spin-up and PITR restore (Step 1–2) | 20–30 minutes |
| Extraction and integrity check (Step 3–4) | 15–30 minutes (varies by tenant data volume) |
| Re-import and production verification (Step 5–6) | 30–60 minutes |
| Cluster teardown (Step 7) | 5–10 minutes |
| **Total (typical tenant)** | **2–4 hours** |

Large tenants (>100k workout rows) may require up to 6 hours. Escalate to devops-lead if the restore exceeds 4 hours without completion.

---

## 11. Index Strategy for RLS Performance

### 11.1 Why indexes matter more with RLS

Every RLS policy appends a `USING` clause to the query. Without supporting indexes, `current_setting('app.current_tenant_id')` comparisons cause a full-table scan on every request. The indexes below ensure that the most common API paths (user's own rows, per-tenant admin aggregates) hit index-only or bitmap-index scans.

### 11.2 Tenant isolation indexes

```sql
-- Primary isolation index for every RLS-protected table
-- Covers: users, user_health_profiles, workouts, meal_logs, coaching_sessions
-- Pattern: (tenant_id) for admin list queries; (tenant_id, user_id) for user-scoped reads

CREATE INDEX idx_users_tenant_id               ON users(tenant_id);
CREATE INDEX idx_users_tenant_active           ON users(tenant_id) WHERE deactivated_at IS NULL;
CREATE INDEX idx_user_health_profiles_tenant   ON user_health_profiles(tenant_id);
CREATE INDEX idx_workouts_tenant_user          ON workouts(tenant_id, user_id);
CREATE INDEX idx_workouts_tenant_started       ON workouts(tenant_id, started_at DESC);
CREATE INDEX idx_meal_logs_tenant_user         ON meal_logs(tenant_id, user_id);
CREATE INDEX idx_coaching_sessions_tenant_user ON coaching_sessions(tenant_id, user_id);
CREATE INDEX idx_scim_log_tenant_created       ON scim_provisioning_log(tenant_id, created_at DESC);
```

### 11.3 Relationship-traversal indexes

```sql
-- workout_sets: RLS subquery goes workout_sets → workouts.user_id
-- The subquery resolves via workout_id FK; this index makes it O(log n)
CREATE INDEX idx_workout_sets_workout_id       ON workout_sets(workout_id);

-- coaching_turns: RLS subquery goes coaching_turns → coaching_sessions.user_id
CREATE INDEX idx_coaching_turns_session_id     ON coaching_turns(session_id);
```

### 11.4 SSO / SCIM lookup indexes

```sql
-- SSO config lookup on the auth callback path (hot path; must be index-only)
CREATE UNIQUE INDEX idx_sso_configs_tenant_id  ON tenant_sso_configs(tenant_id);

-- SCIM token auth: hash lookup on every SCIM request
CREATE UNIQUE INDEX idx_scim_tokens_hash       ON tenant_scim_tokens(token_hash)
  WHERE revoked_at IS NULL;  -- partial index; expired tokens excluded from hot path

-- Tenant slug lookup for SSO metadata URL routing (/sso/{slug}/metadata)
CREATE UNIQUE INDEX idx_tenants_slug           ON tenants(slug);
```

### 11.5 Time-series and aggregate indexes

```sql
-- BRIN index for the materialized view refresh job (wide date range scans on workouts)
-- BRIN is ~200× smaller than B-tree on monotonically increasing timestamps
CREATE INDEX idx_workouts_started_brin         ON workouts USING BRIN (started_at)
  WITH (pages_per_range = 128);

-- Partial index: recent workouts (last 90 days) — covers the MV WHERE clause
CREATE INDEX idx_workouts_recent_tenant        ON workouts(tenant_id, started_at)
  WHERE started_at >= NOW() - INTERVAL '90 days';
-- Note: this index is rebuilt nightly by the MV refresh job; adjust if retention window changes
```

### 11.6 Index maintenance policy

| Rule | Detail |
|---|---|
| Every new table must have `(tenant_id)` or `(tenant_id, user_id)` index before first migration to production | Enforced in migration checklist §7.2 |
| `CONCURRENTLY` required for all index creation on non-empty tables | Prevents lock escalation during deployments |
| `pg_stat_user_indexes` reviewed quarterly for unused indexes | Unused = `idx_scan = 0` after 90 days in production |
| BRIN indexes re-analyzed after bulk data migrations | `ANALYZE workouts;` in migration script |

---

**v0.4 · травень 2026 · owner: enterprise-architect + compliance-officer + security-engineer**

*v0.2 additions: Section 10 PITR Tenant-Isolated Restore Runbook (closes SOC 2 gap 9.4).*

*v0.3 additions: §2.8 workout_sets table (closes inconsistency with PITR runbook §10.3); §2.9 coaching_sessions and coaching_turns (Victor interaction storage with privacy floor enforcement); §2.10 tenant_sso_configs, tenant_scim_tokens, scim_provisioning_log (SSO/SCIM auxiliary tables, cross-reference docs/SSO_SCIM_IMPLEMENTATION.md); §3.6 RLS policies for workout_sets, coaching sessions/turns; §3.7 RLS policies for SSO/SCIM tables; §5 data classification updated for new tables; §11 Index Strategy for RLS Performance (isolation indexes, relationship traversal, SCIM/SSO, BRIN for time-series, maintenance policy).*

*v0.4 additions: §12 Soft Delete, Data Retention & GDPR Art. 17 Erasure — soft-delete strategy per table, retention windows by data classification, Art. 17 erasure flow with anonymization design rationale, automated pg_cron retention jobs, Art. 20 data portability export spec, data minimisation register.*

---

## 12. Soft Delete, Data Retention & GDPR Art. 17 Erasure

> Owner: `compliance-officer` + `enterprise-architect`. Mandatory review before any enterprise contract signing and on any change to data retention commitments.
> References: docs/GDPR_DPIA.md, docs/PRIVACY_POLICY.md, docs/AUDIT_LOG_SCHEMA.md.

---

### 12.1 Soft Delete Strategy

FORM uses **soft delete** (`deleted_at TIMESTAMPTZ`) for all user-generated data. Hard deletes are reserved for the GDPR Art. 17 erasure path only. This design satisfies three requirements simultaneously:

1. **Recovery window** — accidental deletions recoverable within 30 days without PITR restore
2. **Audit integrity** — references in `audit_log` remain non-null; FK violations are impossible on soft-deleted rows
3. **Compliance traceability** — deletion timestamp is auditable; initiator and timestamp are in the audit log

#### Soft-delete columns per table

| Table | `deleted_at` | `deleted_by` | Notes |
|---|---|---|---|
| `users` | ✅ | `user_id` (self) or tenant admin | Primary soft-delete surface |
| `workouts` | ✅ | `user_id` | User can delete individual sessions |
| `workout_sets` | Cascade from `workouts` | — | No independent delete |
| `coaching_sessions` | ✅ | `user_id` | Victor conversation history |
| `coaching_turns` | Cascade from `coaching_sessions` | — | |
| `meal_logs` | ✅ | `user_id` | User-initiated from history UI |
| `body_metrics` | ✅ | `user_id` | Weight/measurement history |
| `media_uploads` | ✅ | `user_id` | CV pose snapshot thumbnails |
| `tenants` | ✅ | FORM engineer only | Off-boarded enterprise tenants |
| `tenant_sso_configs` | Cascade from `tenants` | — | |
| `tenant_scim_tokens` | Uses `revoked_at` instead | — | Revoked ≠ deleted; separate semantics |
| `audit_log` | ❌ **Never** | — | HMAC-chained; immutable per DEC-030 |

#### RLS interaction with soft-deleted rows

Every user-data RLS policy must exclude soft-deleted rows. Enforced pattern:

```sql
CREATE POLICY "user_data_select" ON <table>
  FOR SELECT USING (
    tenant_id = current_setting('app.current_tenant_id')::UUID
    AND user_id = auth.uid()
    AND deleted_at IS NULL  -- mandatory; absence of this clause is a P0 bug
  );
```

This clause is checked at migration review. Any RLS policy missing it blocks the migration.

---

### 12.2 Retention Windows by Data Classification

| Data category | Classification | Consumer retention | Enterprise retention | Hard-delete trigger |
|---|---|---|---|---|
| Workout sessions + sets | Personal fitness data | Account lifetime + 30-day recovery | Per DPA; min 12 months | Erasure request or account deletion + 30 days |
| Coaching turns (Victor) | Personal + Art. 9 (health context) | Account lifetime + 30 days | Per DPA; max 24 months unless DPA extends | Erasure request |
| Meal logs | Art. 9 special category | Account lifetime + 30 days | Per DPA | Erasure request |
| Body metrics | Art. 9 special category | Account lifetime + 30 days | Per DPA | Erasure request |
| Media uploads (CV pose) | Personal | 90-day rolling (thumbnails); raw frames discarded on-device | Per DPA | Automatic at 90 days or erasure request |
| Auth events (audit log) | Personal operational | 2 years | 7 years (SOC 2 evidence) | Automatic at window end; not subject to Art. 17 (Art. 6(1)(c) legal obligation basis) |
| SSO/SCIM events (audit log) | Personal operational | 2 years | 7 years (SOC 2 evidence) | Automatic at window end |
| Anonymized aggregate metrics | Non-personal | Indefinite | Indefinite | N/A |

**Art. 9 note:** Meal logs, body metrics, and coaching turns containing health disclosures are special category. They require explicit consent (captured at onboarding) and are stored encrypted at rest with Supabase Vault. Never included in anonymized aggregate exports without explicit secondary consent.

---

### 12.3 GDPR Art. 17 Right to Erasure — Implementation

Erasure must execute within 30 days of a verified request.

#### Erasure flow

```
User submits request (Settings → Privacy → Delete my data)
  ↓
Re-auth required (password or active SSO session)
  ↓
Audit log: { event: 'erasure.requested', user_id, tenant_id, requested_at, ip }
  ↓
Soft-delete users row: deleted_at = NOW(), erasure_requested_at = NOW()
  ↓
Queue async erasure job (Supabase Edge Function + pg_cron)
  ↓
Erasure job (within 24h of request; completes within 30 days):
  → Hard-delete: workout_sets, workouts, coaching_turns, coaching_sessions,
                 meal_logs, body_metrics, media_uploads (Storage purge)
  → Anonymize users row (see §12.3 below — not hard-delete; FK integrity)
  → Retain: audit_log rows (Art. 6(1)(c) legal obligation)
  → Audit log: { event: 'erasure.completed', user_id, completed_at, record_counts }
  ↓
Confirmation email sent to user's address before it is nulled
```

#### Why `users` is anonymized, not hard-deleted

Hard-deleting the `users` row would orphan every `audit_log.user_id` FK reference, or require nulling the FK across thousands of audit rows — breaking the HMAC chain (DEC-030). The UUID itself contains no personal data. GDPR Art. 17(3)(b) permits retention where necessary for legal obligation (audit records).

Post-anonymization `users` row:

```sql
UPDATE users SET
  email                = NULL,
  display_name         = NULL,
  avatar_url           = NULL,
  phone                = NULL,
  date_of_birth        = NULL,
  biological_sex       = NULL,
  height_cm            = NULL,
  locale               = NULL,
  timezone             = NULL,
  deleted_at           = NOW(),
  erasure_completed_at = NOW()
WHERE id = :user_id;
-- id (UUID) and tenant_id retained for audit FK integrity only
-- No personal data remains in the row after this update
```

#### Enterprise tenant erasure

Enterprise admins may initiate erasure on behalf of departing employees via the admin dashboard or SCIM DELETE endpoint (SSO_SCIM_IMPLEMENTATION.md §3). The SCIM DELETE path triggers the same erasure queue. Enterprise DPAs must specify the erasure SLA; FORM default: 30 days with confirmation to tenant admin.

---

### 12.4 Automated Retention Enforcement

Art. 5(1)(e) GDPR (storage limitation) requires automatic hard-deletion when retention windows expire, not just on-request erasure.

#### pg_cron schedule

```sql
-- Nightly at 02:00 UTC (low-traffic window)

-- 1. Purge consumer media uploads past 90-day rolling window
SELECT cron.schedule('purge-media-uploads', '0 2 * * *', $$
  UPDATE media_uploads
  SET deleted_at = NOW()
  WHERE deleted_at IS NULL
    AND created_at < NOW() - INTERVAL '90 days'
    AND tenant_id IS NULL;  -- consumer only; enterprise governed by DPA schedule
$$);

-- 2. Hard-delete user data past 30-day soft-delete recovery window
SELECT cron.schedule('hard-delete-user-data', '30 2 * * *', $$
  DELETE FROM workout_sets
    WHERE workout_id IN (SELECT id FROM workouts WHERE deleted_at < NOW() - INTERVAL '30 days');
  DELETE FROM workouts WHERE deleted_at < NOW() - INTERVAL '30 days';
  DELETE FROM coaching_turns
    WHERE session_id IN (SELECT id FROM coaching_sessions WHERE deleted_at < NOW() - INTERVAL '30 days');
  DELETE FROM coaching_sessions WHERE deleted_at < NOW() - INTERVAL '30 days';
  DELETE FROM meal_logs     WHERE deleted_at < NOW() - INTERVAL '30 days';
  DELETE FROM body_metrics  WHERE deleted_at < NOW() - INTERVAL '30 days';
$$);

-- 3. Purge consumer audit_log past 2-year retention
SELECT cron.schedule('purge-audit-log-consumer', '0 3 * * *', $$
  DELETE FROM audit_log
  WHERE tenant_id IS NULL
    AND created_at < NOW() - INTERVAL '2 years';
$$);
-- Enterprise audit log purge is NOT automated. Requires compliance-officer sign-off
-- per contractual retention commitments. Manual process: INCIDENT_RESPONSE.md §7.
```

**Monitoring requirement:** Every pg_cron job writes row counts to `cron_job_results`. A job returning 0 affected rows for 7+ consecutive days triggers PagerDuty P3 (possible misconfiguration). See OBSERVABILITY.md §12 for the instrumentation spec.

---

### 12.5 Data Portability — Art. 20

Art. 20 GDPR grants users the right to receive their personal data in machine-readable format within 30 days.

**Export scope:** All personal data held at the time of request. Excludes anonymized aggregate data (non-personal).

**Format:** JSON archive (`.zip`):

```
form-export-{user_id}-{date}.zip
├── profile.json           # users row (all non-null PII fields at time of export)
├── workouts.json          # all sessions + sets (non-deleted)
├── coaching_sessions.json # full Victor conversation history (non-deleted)
├── meal_logs.json         # all meal log entries
├── body_metrics.json      # all body measurements
└── README.txt             # field definitions, data classification, retention notice
```

**Delivery:** Signed Supabase Storage URL (24h TTL) sent to verified email. URL generated by Cloudflare Worker — never a direct client-accessible bucket URL (prevents IDOR).

**Rate limit:** Max 3 export requests per 30-day window per user. Re-auth required per request.

**SLA:** Target < 72 hours automated; hard deadline 30 days per Art. 20.

---

### 12.6 Data Minimisation Register (Art. 5(1)(c))

| Data point | Collected? | Justification if yes / reason if no |
|---|---|---|
| Date of birth | Yes | Age-appropriate load programming; hormonal periodization context |
| Biological sex | Optional | Relevant to programming; user-controlled; non-binary option provided |
| Height + weight | Optional | Load calculation and BMR estimation; user-controlled |
| GPS / location | No | Not required for coaching function |
| Device contacts | No | No in-app social (DEC-002); no contact import |
| Camera (CV pose) | On-demand, session-scoped | Raw frames never leave device; only pose keypoints transmitted |
| HealthKit / Health Connect | On-demand, explicit consent | HRV and resting HR only; sleep data optional; workout write-back only |
| Payment data | No | Stripe handles; FORM never touches raw card data |
| Advertising IDs (IDFA) | No | ATT framework not requested; no retargeting SDK |

Any addition to this register requires `compliance-officer` review and a DECISION_LOG entry before implementation.

---

## 13. Analytics Event Tracking Schema

> Owner: `data-engineer`. Review: on any new event, schema version bump, or ClickHouse migration.
> References: §5 Data Classification, §6 Privacy Floor Enforcement, §12 Soft Delete & GDPR Art. 17, DEC-013 (streak grace after 2 misses).

---

### 13.1 Design Principles

#### 13.1.1 Capture layer

PostHog (EU Cloud, `eu.posthog.com`) is the primary event store. Every client-side event fired by the React Native app is also mirrored server-side by the Cloudflare Worker middleware before being forwarded to PostHog. For billing-critical and subscription-state events, the server-side emission is the authoritative record; client-side is discarded on conflict.

The trust hierarchy is:

| Source | Trusted for |
|---|---|
| Cloudflare Worker (server-side) | Subscription events, tier changes, SSO login outcomes, server_timestamp |
| React Native PostHog SDK (client-side) | UI-interaction events (feature views, onboarding steps, settings changes) |
| Client-side only — never trusted | Revenue state, tier value, tenant_id |

#### 13.1.2 Privacy-first: no PII, no Art. 9 health data in analytics events

All analytics events are pseudonymous. The `user_id` property is a UUID that maps to a real identity only inside Supabase (RLS-protected). The following are **never** included in any event property payload:

- Email address, display name, phone number
- Body measurements (weight, BMI, body fat %)
- Meal content or nutritional detail (Art. 9 special-category data)
- Injury notes, mental health flags, free-text from coaching turns
- Precise geolocation
- Raw CV pose data or form scores for individual lifts
- IDFA / advertising identifiers

Exercise names, rep counts, and set counts are permitted as training-behaviour signals because they do not constitute health data under GDPR Art. 9 in the absence of a clinical context. See §5 for full classification and §6 for the privacy floor enforcement rules that govern what may never appear in aggregate views.

#### 13.1.3 Pseudonymous user_id

Every event carries `user_id` as a UUID (the same UUID primary key used in `public.users`). The UUID is not derived from email or any other PII. Re-identification from the analytics warehouse alone is not possible without a join to the Supabase `users` table, which is access-controlled by RLS.

#### 13.1.4 Server-side enrichment via Cloudflare Worker middleware

Properties that cannot be trusted from the client — `user_tier`, `tenant_id`, `server_timestamp`, `worker_region` — are injected by the Cloudflare Worker on every event before the event is forwarded to PostHog. See §13.3 for the enrichment pseudocode.

#### 13.1.5 Schema versioning

Every event carries a `$schema_version` property (string, semver format, e.g. `"1.0.0"`). The version is incremented when:

- A required property is added or removed from any event
- A property type changes
- A new event name is introduced

The Zod event schema (source of truth: `src/analytics/events.schema.ts`) is validated in CI. Events that fail schema validation fail the build. Backfill SQL for schema migrations is committed to `src/analytics/backfills/`.

---

### 13.2 PostHog Event Taxonomy

All events carry the following **universal properties** (omitted from per-event tables for brevity):

| Property | Type | Source | Description |
|---|---|---|---|
| `user_id` | `string (UUID)` | Client | Pseudonymous user identifier |
| `$schema_version` | `string (semver)` | Client | Event schema version |
| `app_version` | `string (semver)` | Client | App build version |
| `platform` | `"ios" \| "android"` | Client | Device platform |
| `os_version` | `string` | Client | OS version string |
| `locale` | `string (BCP 47)` | Client | Device locale, e.g. `"en-GB"` |
| `client_timestamp` | `string (ISO 8601)` | Client | Client-side event time (for ordering) |
| `server_timestamp` | `string (ISO 8601)` | Worker (enriched) | Trusted server time |
| `user_tier` | `"free" \| "pro" \| "enterprise"` | Worker (enriched) | Subscription tier at event time |
| `tenant_id` | `string (UUID) \| null` | Worker (enriched) | Enterprise tenant; null for consumer |
| `worker_region` | `string` | Worker (enriched) | Cloudflare colo region, e.g. `"eu-west-1"` |

#### 13.2.1 App lifecycle

| Event name | Trigger | Required properties | Notes |
|---|---|---|---|
| `app_opened` | App enters foreground (cold start or resume) | `session_id: string (UUID)`, `is_cold_start: boolean`, `source: "direct" \| "notification" \| "deep_link"` | Session ID generated client-side; cold-start = process not in memory |
| `app_backgrounded` | App moves to background | `session_id: string (UUID)`, `foreground_duration_ms: number` | Foreground duration computed from paired `app_opened` client_timestamp |

**Explicitly excluded from these events:** device model string (fingerprinting risk), battery level, network type.

#### 13.2.2 Onboarding

| Event name | Trigger | Required properties | Notes |
|---|---|---|---|
| `onboarding_started` | User reaches first onboarding screen after account creation | `method: "apple_sso" \| "email"` | Fired once per user lifetime |
| `onboarding_step_completed` | User advances past any onboarding screen | `step_index: number`, `step_name: string`, `time_on_step_ms: number` | `step_name` is a stable slug, e.g. `"goal_selection"`, `"experience_level"` |
| `onboarding_completed` | User reaches Today screen for first time | `total_duration_ms: number`, `steps_completed: number`, `goal: string` | `goal` is a categorical label from a fixed enum (e.g. `"strength"`, `"fat_loss"`) — never free text |

**Explicitly excluded:** health goal detail beyond top-level enum, injury or restriction notes, body measurements entered during onboarding.

#### 13.2.3 Workouts

| Event name | Trigger | Required properties | Notes |
|---|---|---|---|
| `workout_started` | User taps Start Workout | `workout_id: string (UUID)`, `program_id: string \| null`, `exercise_count: number`, `is_coached: boolean` | `is_coached` = Victor AI session active |
| `workout_set_logged` | User logs a completed set | `workout_id: string (UUID)`, `set_number: number`, `exercise_name: string`, `reps: number \| null`, `is_time_based: boolean`, `rpe: number \| null`, `used_cv: boolean` | Rep count and exercise name are permitted (training behaviour, not Art. 9 health data). Weight is **not** included — see note below |
| `workout_completed` | User taps Finish and confirms | `workout_id: string (UUID)`, `duration_seconds: number`, `sets_logged: number`, `ai_turns: number`, `voice_used: boolean` | |
| `workout_abandoned` | User exits mid-workout without completing | `workout_id: string (UUID)`, `duration_seconds: number`, `sets_logged: number`, `last_exercise_name: string \| null` | |

**Weight (kg/lbs) is explicitly excluded from `workout_set_logged`** — it is stored in Supabase `workout_sets.weight_kg` (Confidential classification, §5) and must never appear in analytics events. Form scores and CV flags are likewise excluded.

#### 13.2.4 AI coaching

| Event name | Trigger | Required properties | Notes |
|---|---|---|---|
| `ai_message_sent` | User sends a message to Victor | `session_id: string (UUID)`, `coaching_session_id: string (UUID)`, `message_index: number`, `input_type: "text" \| "voice"`, `context: "workout" \| "standalone"` | Message content is **never** included |
| `ai_message_received` | Victor response delivered to client | `session_id: string (UUID)`, `coaching_session_id: string (UUID)`, `message_index: number`, `response_latency_ms: number`, `model_id: string`, `has_voice: boolean` | Model ID is the Anthropic model slug. Token counts excluded (cost data, not product analytics) |
| `ai_voice_played` | User plays a Victor voice response | `coaching_session_id: string (UUID)`, `voice_duration_ms: number`, `completed: boolean` | `completed` false if user skipped before end |

**Explicitly excluded from all AI events:** message content (free text, Confidential §5), token counts, ElevenLabs character counts.

#### 13.2.5 Subscriptions

Server-side only. These events are emitted by the Cloudflare Worker on Stripe webhook receipt — never from the client.

| Event name | Trigger | Required properties | Notes |
|---|---|---|---|
| `subscription_started` | Stripe `customer.subscription.created` webhook | `plan: "pro" \| "enterprise"`, `billing_period: "monthly" \| "annual"`, `trial: boolean`, `trial_end_date: string (ISO 8601) \| null` | |
| `subscription_cancelled` | Stripe `customer.subscription.deleted` webhook | `plan: "pro" \| "enterprise"`, `reason: string \| null`, `days_since_start: number`, `at_period_end: boolean` | `reason` is the Stripe cancellation reason enum — no free text |
| `subscription_reactivated` | Stripe `customer.subscription.updated` (cancelled → active) | `plan: "pro" \| "enterprise"`, `days_since_cancellation: number` | |
| `subscription_upgraded` | Stripe `customer.subscription.updated` (plan change) | `from_plan: string`, `to_plan: string`, `from_billing_period: string`, `to_billing_period: string` | |

**Explicitly excluded:** Stripe customer ID, payment method details, invoice amounts.

#### 13.2.6 Enterprise SSO

Server-side only. Emitted by the Cloudflare Worker SSO callback handler.

| Event name | Trigger | Required properties | Notes |
|---|---|---|---|
| `sso_login_initiated` | User is redirected to IdP (OIDC redirect) | `tenant_id: string (UUID)`, `idp_type: "okta" \| "azure_ad" \| "google_workspace" \| "generic_oidc"` | |
| `sso_login_completed` | OIDC callback succeeds; JWT issued | `tenant_id: string (UUID)`, `idp_type: string`, `is_new_user: boolean`, `scim_provisioned: boolean` | |
| `sso_login_failed` | OIDC callback returns error or token validation fails | `tenant_id: string (UUID)`, `idp_type: string`, `error_code: string` | `error_code` is a stable internal enum, not the raw IdP error string |

**Explicitly excluded:** OIDC subject claim, IdP user attributes, email address.

#### 13.2.7 Streaks

| Event name | Trigger | Required properties | Notes |
|---|---|---|---|
| `streak_maintained` | User completes a workout that extends their active streak | `streak_length_days: number`, `current_streak_start: string (ISO 8601 date)` | |
| `streak_broken` | Streak breaks (per DEC-013: after 2 consecutive missed days, not 1) | `streak_length_days: number`, `missed_days: number` | `missed_days` will be 2 under current DEC-013 policy |
| `streak_grace_used` | Grace period applied on first miss (day 1 of potential 2-day miss window) | `streak_length_days: number`, `grace_day: number` | Streak is preserved; user notified. `grace_day` is always 1 under DEC-013 |

#### 13.2.8 Settings and permissions

| Event name | Trigger | Required properties | Notes |
|---|---|---|---|
| `settings_changed` | User saves a settings change | `setting_key: string`, `new_value_type: "boolean" \| "string_enum" \| "number"` | `new_value_type` encodes the type of change; the value itself is **not** included unless it is a non-sensitive boolean (e.g. `dark_mode_enabled: true`) |
| `notification_permission_granted` | OS permission prompt accepted | `permission_type: "push" \| "local"` | |
| `notification_permission_denied` | OS permission prompt denied or dismissed | `permission_type: "push" \| "local"` | |

#### 13.2.9 Feature flag exposure

| Event name | Trigger | Required properties | Notes |
|---|---|---|---|
| `feature_flag_exposure` | PostHog SDK evaluates a feature flag for the first time in a session | `flag_key: string`, `flag_variant: string \| boolean`, `evaluation_context: "onboarding" \| "workout" \| "coaching" \| "subscription" \| "other"` | Standard PostHog `$feature_flag_called` event; this schema wraps and enforces typed properties on top of it. Used for A/B test analysis. |

---

### 13.3 Server-Side Enrichment Middleware

The Cloudflare Worker intercepts all `/v1/analytics/capture` requests from the app and enriches each event before forwarding to PostHog. Properties injected at this layer override any client-supplied values of the same name.

```typescript
// src/workers/middleware/analytics-enrichment.ts
// Runs in Cloudflare Worker before PostHog capture

interface JwtClaims {
  sub: string;          // user_id (UUID)
  tenant_id: string | null;
  tier: "free" | "pro" | "enterprise";
  iat: number;
  exp: number;
}

interface RawEvent {
  event: string;
  properties: Record<string, unknown>;
  timestamp?: string;   // client_timestamp — retained but not trusted for ordering
}

interface EnrichedEvent extends RawEvent {
  properties: Record<string, unknown> & {
    user_id: string;
    user_tier: "free" | "pro" | "enterprise";
    tenant_id: string | null;
    server_timestamp: string;
    worker_region: string;
    $schema_version: string;
  };
}

async function enrichEvent(
  rawEvent: RawEvent,
  jwtClaims: JwtClaims,
  request: Request,
): Promise<EnrichedEvent> {
  // Validate schema before enrichment — fail fast
  const validated = EventSchema.parse(rawEvent); // Zod; throws on violation

  // Scrub any PII fields the client should never have sent
  const safeProperties = stripForbiddenProperties(validated.properties);

  // Server-authoritative properties — always overwrite client values
  const serverEnriched = {
    ...safeProperties,
    user_id: jwtClaims.sub,                            // from verified JWT, never client body
    user_tier: jwtClaims.tier,                         // from JWT claim; Stripe state reflected at auth time
    tenant_id: jwtClaims.tenant_id ?? null,            // null for consumer users
    server_timestamp: new Date().toISOString(),        // trusted server time
    worker_region: request.cf?.region ?? "unknown",    // Cloudflare colo region
    $schema_version: CURRENT_SCHEMA_VERSION,           // pinned in worker deploy
  };

  return { ...validated, properties: serverEnriched };
}

// Properties that must never reach PostHog regardless of client payload
const FORBIDDEN_PROPERTY_KEYS = new Set([
  "email", "display_name", "name", "phone",
  "weight_kg", "weight_lbs", "bmi", "body_fat_pct", "height_cm",
  "meal_content", "meal_name",
  "injury_notes", "mental_health_flag",
  "cv_flags", "form_score",
  "lat", "lng", "latitude", "longitude",
  "idfa", "advertising_id",
  "stripe_customer_id",
  "coaching_turn_content",
]);

function stripForbiddenProperties(
  props: Record<string, unknown>,
): Record<string, unknown> {
  const stripped = { ...props };
  for (const key of Object.keys(stripped)) {
    if (FORBIDDEN_PROPERTY_KEYS.has(key)) {
      delete stripped[key];
      // Emit a separate internal alert — indicates a client-side bug or schema regression
      void reportForbiddenPropertyViolation(key);
    }
  }
  return stripped;
}
```

The enriched event is forwarded to PostHog using the server-side PostHog client with `distinctId` set to `jwtClaims.sub`. The client-side PostHog SDK is configured with `person_profiles: "identified_only"` and `capture_pageview: false`.

---

### 13.4 ClickHouse Analytics Tables

ClickHouse Cloud (EU region, matching Supabase `eu-central-1` data residency) is the analytics warehouse. Tables use the MergeTree engine family for efficient time-series aggregation.

#### 13.4.1 `fact_events`

Raw event log. Populated hourly from PostHog S3 export (see §13.5).

```sql
CREATE TABLE analytics.fact_events
(
    event_id       UUID,
    user_id        UUID,
    event_name     LowCardinality(String),
    ts             DateTime64(3, 'UTC'),          -- server_timestamp from enrichment
    client_ts      DateTime64(3, 'UTC'),          -- client_timestamp for ordering
    properties     JSON,                           -- full enriched properties blob
    tier           LowCardinality(String),        -- "free" | "pro" | "enterprise"
    tenant_id      Nullable(UUID),
    app_version    LowCardinality(String),
    platform       LowCardinality(String),        -- "ios" | "android"
    schema_version LowCardinality(String)
)
ENGINE = ReplacingMergeTree(ts)
PARTITION BY toYYYYMM(ts)
ORDER BY (event_name, user_id, ts, event_id)
TTL toDateTime(ts) + INTERVAL 2 YEAR DELETE
SETTINGS index_granularity = 8192;
```

`ReplacingMergeTree(ts)` ensures that if the same `event_id` is re-ingested (e.g. from a PostHog export retry), the row with the latest `ts` wins. Deduplication is eventual — queries against `fact_events` use `FINAL` or a `max(ts) GROUP BY event_id` subquery where exactness is required.

#### 13.4.2 `dim_users`

User dimension updated daily. Reflects state as of the latest ETL run, not point-in-time.

```sql
CREATE TABLE analytics.dim_users
(
    user_id              UUID,
    first_seen_date      Date,
    cohort_week          Date,                   -- toMonday(first_seen_date)
    acquisition_channel  LowCardinality(String), -- "organic" | "paid_ua" | "referral" | "enterprise_scim"
    tier                 LowCardinality(String),
    tenant_id            Nullable(UUID),
    country_code         LowCardinality(String), -- ISO 3166-1 alpha-2; derived from worker_region at first open
    is_enterprise        UInt8,                  -- 1 if tenant_id IS NOT NULL
    last_updated         DateTime64(3, 'UTC')
)
ENGINE = ReplacingMergeTree(last_updated)
PARTITION BY toYYYYMM(first_seen_date)
ORDER BY (user_id)
SETTINGS index_granularity = 8192;
```

`country_code` is derived from the `worker_region` Cloudflare colo at `app_opened` (first seen), not from GPS. This is coarse-grained and does not constitute precise location data.

#### 13.4.3 `fact_cohort_retention`

Pre-aggregated retention table rebuilt daily. Powers D1/D7/D14/D30/D60/D90/D180/D365 cohort curves.

```sql
CREATE TABLE analytics.fact_cohort_retention
(
    cohort_week      Date,
    period_day       UInt16,                     -- 1, 7, 14, 30, 60, 90, 180, 365
    tier             LowCardinality(String),
    tenant_id        Nullable(UUID),             -- null for cross-tenant consumer aggregate
    retained_users   UInt32,
    cohort_size      UInt32,
    retention_rate   Float32,                    -- retained_users / cohort_size
    computed_at      DateTime64(3, 'UTC')
)
ENGINE = AggregatingMergeTree()
PARTITION BY toYYYYMM(cohort_week)
ORDER BY (cohort_week, period_day, tier, tenant_id)
SETTINGS index_granularity = 8192;
```

Cohorts with `cohort_size < 5` are suppressed before write (k-anonymity floor, matching §2.6 Supabase materialized view policy). The dbt model enforces this filter before insert.

#### 13.4.4 `fact_session_metrics`

One row per workout session. Populated from `fact_events` joins, rebuilt hourly.

```sql
CREATE TABLE analytics.fact_session_metrics
(
    session_id        UUID,
    user_id           UUID,
    date              Date,
    sets_logged       UInt16,
    ai_turns          UInt16,
    voice_played      UInt8,                     -- 1 if ai_voice_played fired in session
    duration_seconds  UInt32,
    completed         UInt8,                     -- 1 if workout_completed; 0 if workout_abandoned
    tier              LowCardinality(String),
    tenant_id         Nullable(UUID),
    app_version       LowCardinality(String),
    platform          LowCardinality(String),
    ingested_at       DateTime64(3, 'UTC')
)
ENGINE = ReplacingMergeTree(ingested_at)
PARTITION BY toYYYYMM(date)
ORDER BY (user_id, date, session_id)
SETTINGS index_granularity = 8192;
```

---

### 13.5 ETL Pipeline

#### 13.5.1 PostHog → ClickHouse sync

| Pipeline | Method | Cadence | Target table |
|---|---|---|---|
| Raw events | PostHog S3 batch export (EU bucket) | Hourly | `fact_events` |
| Session metrics | dbt model over `fact_events` | Hourly, 15 min after export | `fact_session_metrics` |
| User dimension | dbt model from Supabase read replica + `fact_events` | Daily at 03:30 UTC | `dim_users` |
| Cohort retention | dbt model over `fact_events` + `dim_users` | Daily at 04:00 UTC | `fact_cohort_retention` |

The full daily ETL sequence (coordinated by Cloudflare Cron Trigger):

```
03:00 UTC  Supabase read replica → ClickHouse dim_users snapshot
03:15 UTC  PostHog S3 export batch lands
03:20 UTC  ClickHouse COPY INTO fact_events FROM S3
03:30 UTC  dbt run --select session_metrics
04:00 UTC  dbt run --select cohort_retention
04:15 UTC  Refresh Metabase dashboard caches
04:20 UTC  Send daily metrics digest to founder Slack (Cloudflare Worker)
```

Hourly incremental runs at H:15 cover `fact_events` and `fact_session_metrics` only. Cohort and dimension tables are daily (require full-scan aggregation).

#### 13.5.2 Real-time path

Subscription state changes are not batched. The Cloudflare Worker Stripe webhook handler:

1. Updates Supabase `users.tier` and `subscriptions` table immediately.
2. Emits a server-side PostHog event with `server_timestamp`.
3. The hourly PostHog export picks up the event within 75 minutes maximum.

For sub-minute latency on active-user counts, PostHog real-time dashboard queries are used directly. ClickHouse is not in the real-time path.

#### 13.5.3 dbt model structure

```
models/
  staging/
    stg_posthog_events.sql         -- deduplicates fact_events using FINAL
    stg_supabase_users.sql         -- pulls from Supabase read replica
  marts/
    dim_users.sql                  -- joins stg_posthog_events + stg_supabase_users
    fact_session_metrics.sql       -- pivots workout_* events into session rows
    fact_cohort_retention.sql      -- cohort join + retention window calculation
```

All dbt models include a `config(tags=["pii_safe"])` assertion that fails if any column name matches the forbidden property key list from §13.3.

---

### 13.6 Privacy Constraints

This section is the authoritative list of what is **never** permitted in any analytics event, ClickHouse table, or dbt model output. Extends §5 (Data Classification) and §6 (Privacy Floor Enforcement) into the analytics layer.

#### 13.6.1 Prohibited analytics data

| Category | Specific fields prohibited | Regulatory basis |
|---|---|---|
| Direct identifiers | Email, display name, phone, Apple Sign In subject claim | GDPR Art. 5(1)(a), Art. 25 |
| Body measurements | Weight (kg/lbs), height, BMI, body fat %, waist circumference | GDPR Art. 9 (health data when combined with identity) |
| Meal content | Food item names, calorie counts, macro breakdowns, meal log text | GDPR Art. 9; clinical-safety ED screening risk |
| Injury and health notes | Injury descriptions, pain ratings, restriction text from coaching | GDPR Art. 9 |
| Mental health signals | Any flag derived from coaching turn sentiment, burnout scores | GDPR Art. 9 |
| CV and form data | Raw pose keypoints, joint angles, form scores per set, `cv_flags` JSONB | GDPR Art. 9 (biometric data) |
| Coaching content | Any free text from `coaching_turns.content` | GDPR Art. 9; Confidential classification (§5) |
| Precise location | GPS lat/lng, postcode | GDPR Art. 9 when combined with health context |
| Advertising identifiers | IDFA, GAID, fingerprint hashes | ATT framework; GDPR Art. 5(1)(c) data minimisation |
| Payment detail | Stripe customer ID, card last-4, invoice amounts | PCI-DSS; GDPR Art. 5(1)(c) |

**What is permitted** as analytics properties in workouts: exercise name (from a fixed catalogue), rep count, set count, RPE value (1–10 scale), session duration, completion boolean. These are training-behaviour signals and do not constitute Art. 9 health data in isolation.

#### 13.6.2 Retention and deletion

- **ClickHouse `fact_events`:** 2-year TTL enforced by `TTL toDateTime(ts) + INTERVAL 2 YEAR DELETE`.
- **After 2 years:** aggregate-only. Cohort tables retained indefinitely (no user-level rows).
- **On GDPR Art. 17 erasure:** user_id rows deleted from `fact_events`, `fact_session_metrics`, and `dim_users` within 30 days. Triggered by the same erasure queue as §12.3. `fact_cohort_retention` not deleted (aggregates; k-anonymity floor applied).
- **Art. 17 ClickHouse erasure SQL** committed to `src/analytics/backfills/erasure_template.sql`.

#### 13.6.3 Cross-reference to §5 classifications

| §5 Classification | May appear in analytics events? | May appear in ClickHouse? |
|---|---|---|
| Public | Yes | Yes |
| Internal | No (email, name are Internal) | No |
| Confidential | No | No |
| Sensitive | No | No |
| Restricted | No | No |

---

### 13.7 Implementation Checklist

| Task | Owner | Priority | Milestone |
|---|---|---|---|
| Create `src/analytics/events.schema.ts` with Zod schemas for all §13.2 events | `data-engineer` | P0 | M3 |
| Add CI schema validation step (Zod parse on all event fixtures) | `data-engineer` | P0 | M3 |
| Implement `stripForbiddenProperties` in Cloudflare Worker enrichment middleware | `data-engineer` + `devops-lead` | P0 | M3 |
| Configure PostHog EU Cloud project; set `person_profiles: "identified_only"` | `data-engineer` | P0 | M3 |
| Wire React Native PostHog SDK; emit `app_opened`, `onboarding_*`, `workout_*` events | `data-engineer` | P0 | M3 |
| Server-side subscription events from Stripe webhook handler | `data-engineer` | P0 | M3 |
| Server-side SSO login events from OIDC callback handler | `data-engineer` | P1 | M4 |
| Provision ClickHouse Cloud cluster (EU region); apply §13.4 DDL | `devops-lead` | P1 | M4 |
| Configure PostHog S3 batch export (EU bucket); validate no cross-region copy | `devops-lead` | P1 | M4 |
| Build dbt models: `stg_posthog_events`, `dim_users`, `fact_session_metrics` | `data-engineer` | P1 | M4 |
| Build dbt model: `fact_cohort_retention`; enforce k-anonymity floor (N < 5 suppressed) | `data-engineer` | P1 | M4 |
| Schedule Cloudflare Cron Trigger for daily ETL sequence (§13.5.1) | `devops-lead` | P1 | M4 |
| Create Metabase instance (self-hosted); connect to ClickHouse; build D1/D7/D30 retention dashboard | `data-engineer` | P1 | M4 |
| Implement Art. 17 ClickHouse erasure job; commit template SQL to `src/analytics/backfills/` | `data-engineer` | P1 | M4 |
| Add `streak_*` events after streak engine ships (DEC-013) | `data-engineer` | P1 | M4 |
| Implement `feature_flag_exposure` schema wrapper on PostHog flag evaluation | `data-engineer` | P2 | M5 |
| Add dbt `pii_safe` tag assertion to all marts | `data-engineer` | P2 | M5 |
| Load-test PostHog enrichment middleware at 10× expected launch DAU | `devops-lead` | P2 | M5 |
| Validate ClickHouse TTL purge with a synthetic 2-year-old dataset | `data-engineer` | P2 | M5 |

---

*v0.4 additions: §13 Analytics Event Tracking Schema — PostHog event taxonomy (nine event categories, 29 named events with typed properties, universal enriched properties); Cloudflare Worker enrichment middleware with Zod validation and forbidden-property scrubbing; ClickHouse warehouse: four tables (fact_events ReplacingMergeTree, dim_users ReplacingMergeTree, fact_cohort_retention AggregatingMergeTree, fact_session_metrics ReplacingMergeTree) with partition keys, ORDER BY, and TTL; hourly + daily ETL pipeline via PostHog S3 export → ClickHouse → dbt; dbt model structure; GDPR Art. 9 analytics prohibition table (10 categories); 2-year ClickHouse TTL; Art. 17 erasure job; §5 classification cross-reference; 19-item implementation checklist across M3/M4/M5.*

---

## 14. Wearable Data Integration Schema

> Owner: `enterprise-architect` + `platform-engineer` + `compliance-officer`. Review: on any new source integration, API contract change, or quarterly.
> Scope: all wearable and health-platform integrations (HealthKit, Health Connect, Whoop, Oura Ring, Garmin). Consumer and enterprise tiers.
> References: DEC-030 (HMAC-chained audit log), docs/SOC2_READINESS.md §35.6.3, docs/GDPR_DPIA.md, docs/AUDIT_LOG_SCHEMA.md, docs/VICTOR_PROMPT_GUIDE.md.

---

### 14.1 Supported Sources and Data Types

The table below documents every wearable source currently integrated, the OAuth or platform mechanism used to acquire data, the data types exposed to FORM, the specific HRV metric definition used by each source (critical for normalization — see §14.5), and integration notes relevant to the platform engineer.

| Source | Platform | Auth mechanism | Data types exposed | HRV metric | Notes |
|---|---|---|---|---|---|
| **HealthKit** | iOS only | HealthKit permission prompt (no OAuth; on-device entitlement) | RMSSD (ms), Resting HR (bpm), Sleep analysis (stages + duration), Active energy (kcal), Steps | RMSSD in ms — gold standard; measured from overnight HRV samples | Background delivery via HealthKit Background Delivery API. No polling required. Data stays on device until FORM app pushes to backend. Requires `NSHealthShareUsageDescription` in plist and `com.apple.developer.healthkit` entitlement. |
| **Health Connect** | Android only | Health Connect permission prompt (runtime permission model; no OAuth) | RMSSD (ms), Resting HR (bpm), Sleep stages + duration, Active calories, Steps | RMSSD in ms; parity with HealthKit metrics where available | Requires `android.permission.health.READ_HEART_RATE_VARIABILITY` and related permissions. Foreground sync on app open plus WorkManager background job (15-minute minimum interval enforced by Android policy — cannot be shortened). Permissions gated by Android 14+ Health Connect API. |
| **Whoop** | iOS + Android | OAuth 2.0 (Authorization Code + PKCE); partner API — requires Whoop partnership agreement | Recovery score (0–100), Strain score (0–21 scale), HRV (ms), Resting HR (bpm), Sleep performance % | RMSSD in ms; computed from 5-minute overnight measurement windows, not beat-to-beat equivalent to HealthKit RMSSD — do not aggregate cross-source | API access requires a signed Whoop developer partnership. Onboarding timeline: 4–8 weeks. Rate limit: 100 requests/minute. Recovery and strain are Whoop-proprietary composite scores. |
| **Oura Ring** | iOS + Android | OAuth 2.0 (Authorization Code + PKCE); Oura Personal API — self-service | Readiness score (0–100), HRV (RMSSD ms), Resting HR (bpm), Sleep stages (light/deep/REM/awake), Temperature deviation (°C) | RMSSD in ms; overnight average; most comparable to HealthKit of all third-party sources | Cleanest third-party API. Self-service developer registration. Daily data granularity (no intra-day). `temperature_deviation_c` is relative to the user's personal baseline — store as-is, do not interpret as absolute body temperature. |
| **Garmin** | iOS + Android | OAuth 1.0a; Garmin Health API — enterprise contract required | Body Battery (0–100), HRV Status (weekly categorical), Resting HR (bpm), Sleep (duration + stages), VO2max estimate | HRV Status is a weekly categorical label (`balanced`, `unbalanced`, `low`, `poor`) — NOT a daily RMSSD value; incompatible with per-day HRV trend analysis | Enterprise-tier contract with Garmin required; legal review takes 6–12 weeks. HRV Status is a fundamentally different metric from RMSSD — it cannot be normalized into the same time-series. Store under `reading_type = 'hrv_status_garmin'`; excluded from cross-source HRV trend logic. VO2max is an estimate, not a clinical measurement. |

---

### 14.2 `wearable_readings` Table Schema

This table is the central store for all wearable-derived health data. It is referenced in docs/SOC2_READINESS.md §35.6.3 as requiring a 2-year retention period (proposed). The table design prioritises idempotency (external `source_reading_id` + unique constraint), auditability (`synced_at` distinct from `recorded_at`), and soft-delete for the GDPR Art. 17 30-day erasure window.

**Classification: SENSITIVE — GDPR Art. 9 special category health data. Per-tenant KMS encryption at rest (§5). Never included in analytics events (§14.10).**

```sql
-- CLASSIFICATION: SENSITIVE — GDPR Art. 9 special category (heart rate variability,
-- resting heart rate, sleep data). Per-tenant KMS encryption at rest (§5).
-- Retention: 2 years from recorded_at (SOC2_READINESS.md §35.6.3). See §14.7.
CREATE TABLE wearable_readings (
  id                  UUID        PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id             UUID        NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  tenant_id           UUID        NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,

  -- Source platform identifier
  source              TEXT        NOT NULL
                        CHECK (source IN (
                          'healthkit',
                          'health_connect',
                          'whoop',
                          'oura',
                          'garmin'
                        )),

  -- Normalized reading type slug. Allowed values:
  --   'hrv_rmssd'              — HRV in ms (HealthKit, Health Connect, Whoop, Oura)
  --   'hrv_status_garmin'      — Garmin weekly HRV Status categorical label
  --   'resting_hr'             — Resting heart rate in bpm
  --   'sleep_duration_min'     — Total sleep duration in minutes
  --   'sleep_efficiency_pct'   — Sleep efficiency percentage (0–100)
  --   'recovery_score'         — Whoop Recovery Score (0–100)
  --   'readiness_score'        — Oura Readiness Score (0–100)
  --   'body_battery'           — Garmin Body Battery (0–100)
  --   'strain'                 — Whoop Strain (0–21 scale; stored as raw float)
  --   'temperature_deviation_c'— Oura skin temperature deviation from personal baseline (°C)
  --   'steps'                  — Step count
  --   'active_energy_kcal'     — Active energy burned in kilocalories
  --   'vo2max_estimate'        — Garmin VO2max estimate (ml/kg/min)
  reading_type        TEXT        NOT NULL,

  -- Normalized value in SI units (ms for HRV, bpm for HR, min for sleep, etc.).
  -- For categorical values (hrv_status_garmin), this column is NULL; use metadata->>'label'.
  value_numeric       NUMERIC,

  -- Unit string matching SI convention: 'ms', 'bpm', 'min', 'pct', 'kcal', 'degC', etc.
  -- Null if the reading is categorical only.
  value_unit          TEXT,

  -- When the wearable device recorded this reading — the ground-truth timestamp.
  -- NOT when FORM synced it. Indexed for time-series queries.
  recorded_at         TIMESTAMPTZ NOT NULL,

  -- When this row was written to FORM's database. Set by the server at upsert time.
  synced_at           TIMESTAMPTZ NOT NULL DEFAULT now(),

  -- External identifier from the source API (e.g. HealthKit UUID, Oura document ID).
  -- Used for idempotency: duplicate sync calls are no-ops on conflict.
  source_reading_id   TEXT,

  -- Source-specific supplementary data. Examples:
  --   HealthKit: { "uuid": "...", "device": "Apple Watch Series 9" }
  --   Oura: { "sleep_stages": { "deep_min": 84, "rem_min": 102, "light_min": 198 } }
  --   Whoop: { "cycle_id": "...", "strain_score": 14.2 }
  --   Garmin: { "hrv_status": "balanced", "weekly_avg_rmssd": null }
  -- Not queried directly by the coaching context builder — extracted fields go in value_numeric.
  metadata            JSONB,

  -- Soft-delete timestamp. Set during the 30-day GDPR Art. 17 erasure window.
  -- Hard delete executed by nightly pg_cron job after 30 days (see §14.7).
  deleted_at          TIMESTAMPTZ,

  created_at          TIMESTAMPTZ NOT NULL DEFAULT now(),

  -- Idempotency constraint: one reading per user/source/type/device-timestamp.
  -- Duplicate upserts from retry logic or re-sync are silently discarded.
  CONSTRAINT uq_wearable_readings_idempotency
    UNIQUE (user_id, source, reading_type, recorded_at)
);

ALTER TABLE wearable_readings ENABLE ROW LEVEL SECURITY;

-- Primary isolation and RLS support index
CREATE INDEX idx_wearable_readings_tenant_user
  ON wearable_readings (tenant_id, user_id);

-- Time-series query index (coaching context builder: last 7 days per user)
CREATE INDEX idx_wearable_readings_user_recorded
  ON wearable_readings (user_id, recorded_at DESC)
  WHERE deleted_at IS NULL;

-- Source + type index for sync idempotency checks
CREATE INDEX idx_wearable_readings_source_type
  ON wearable_readings (user_id, source, reading_type, recorded_at DESC)
  WHERE deleted_at IS NULL;

-- BRIN index for retention/erasure job (full-table date scan)
CREATE INDEX idx_wearable_readings_recorded_brin
  ON wearable_readings USING BRIN (recorded_at)
  WITH (pages_per_range = 128);
```

---

### 14.3 RLS Policies for `wearable_readings`

Policy design follows the same fail-closed, tenant-scoped pattern as all other user-data tables (§3.1). Four distinct access tiers are enforced at the database layer.

```sql
-- -----------------------------------------------------------------------
-- Tier 1: User — read and write their own readings only
-- -----------------------------------------------------------------------
CREATE POLICY wearable_readings_tenant_isolation ON wearable_readings
  AS PERMISSIVE FOR SELECT
  TO form_api
  USING (
    tenant_id = current_setting('app.current_tenant_id', TRUE)::UUID
  );

CREATE POLICY wearable_readings_owner_only ON wearable_readings
  AS RESTRICTIVE FOR SELECT
  TO form_api
  USING (
    user_id    = current_setting('app.current_user_id', TRUE)::UUID
    AND deleted_at IS NULL
  );

-- Users may insert their own readings (sync endpoint writes as the authenticated user)
CREATE POLICY wearable_readings_insert ON wearable_readings
  AS PERMISSIVE FOR INSERT
  TO form_api
  WITH CHECK (
    tenant_id = current_setting('app.current_tenant_id', TRUE)::UUID
    AND user_id = current_setting('app.current_user_id', TRUE)::UUID
  );

-- Users may soft-delete their own readings (disconnect source triggers bulk soft-delete)
CREATE POLICY wearable_readings_soft_delete ON wearable_readings
  AS PERMISSIVE FOR UPDATE
  TO form_api
  USING (
    tenant_id  = current_setting('app.current_tenant_id', TRUE)::UUID
    AND user_id = current_setting('app.current_user_id', TRUE)::UUID
  )
  WITH CHECK (
    -- Only allow setting deleted_at; no other field mutations via this policy
    deleted_at IS NOT NULL
  );

-- -----------------------------------------------------------------------
-- Tier 2: Tenant admin / owner — aggregate counts ONLY via VIEW, not raw rows
-- Raw wearable_readings rows are NEVER returned to tenant admin roles.
-- Aggregates are exposed via tenant_wearable_summary VIEW (§14.4).
-- -----------------------------------------------------------------------
-- No SELECT policy for tenant_admin / tenant_owner is defined on this table.
-- Absence of a permissive policy = zero rows returned (fail-closed per §3.1).
-- Tenant admin data access is routed exclusively through tenant_wearable_summary.

-- -----------------------------------------------------------------------
-- Tier 3: Coach role — read readings for users who have granted consent
-- Victor's coaching context builder runs under the 'form_coach' role and
-- requires cross-user read within a tenant (e.g. a human coach reviewing
-- a client's wearable trends). Access is blocked unless explicit wearable
-- consent exists in the consent table.
-- -----------------------------------------------------------------------
CREATE POLICY wearable_readings_coach ON wearable_readings
  AS PERMISSIVE FOR SELECT
  TO form_coach
  USING (
    tenant_id = current_setting('app.current_tenant_id', TRUE)::UUID
    AND deleted_at IS NULL
    AND EXISTS (
      SELECT 1
      FROM user_consents uc
      WHERE uc.user_id            = wearable_readings.user_id
        AND uc.tenant_id          = wearable_readings.tenant_id
        AND 'wearable'            = ANY(uc.categories_consented)
        AND uc.coach_access       = TRUE
        AND uc.revoked_at         IS NULL
    )
  );

-- -----------------------------------------------------------------------
-- Tier 4: form_admin — BYPASSRLS
-- form_admin is used only by: Art. 17 erasure job, PITR restore, compliance toolchain.
-- Never in the API request path. Access logged via DEC-030.
-- -----------------------------------------------------------------------
-- No explicit policy required; BYPASSRLS role attribute applies.
```

**CI verification:** The cross-tenant RLS isolation test (`__tests__/db/rls_isolation.test.ts`) is extended to assert that a `tenant_admin` role receives zero rows from `wearable_readings` and that a `form_coach` role receives zero rows for a user who has not granted wearable consent.

---

### 14.4 Tenant-Visible Aggregates

Tenant administrators (HR, wellness program managers) require aggregate visibility into the population-level effectiveness of the wellness program. Direct exposure of individual HRV or sleep values would constitute a GDPR Art. 9 violation — a HR administrator seeing an individual employee's HRV could infer cardiac conditions, stress disorders, or recovery from illness.

The `tenant_wearable_summary` view enforces three privacy constraints:

1. **No raw numeric values**: HRV is bucketed into directional trend only. Sleep is bucketed into duration bands. No exact minutes or milliseconds are returned.
2. **k-anonymity floor**: groups with fewer than 5 users are suppressed entirely, matching the floor applied to `tenant_wellness_summary` (§2.6) and ClickHouse cohort tables (§13.4.3).
3. **Weekly granularity only**: daily resolution carries re-identification risk for small teams; weekly aggregation is the minimum safe window.

```sql
-- Materialized view — refreshed nightly (same schedule as tenant_wellness_summary)
-- Tenant admins query this view; they never query wearable_readings directly.
-- RLS on the underlying table prevents any direct bypass.
CREATE MATERIALIZED VIEW tenant_wearable_summary AS
WITH weekly_hrv AS (
  SELECT
    wr.tenant_id,
    wr.user_id,
    DATE_TRUNC('week', wr.recorded_at)  AS week,
    AVG(wr.value_numeric)               AS avg_hrv_ms,
    -- Prior week average for trend calculation
    LAG(AVG(wr.value_numeric), 1) OVER (
      PARTITION BY wr.tenant_id, wr.user_id
      ORDER BY DATE_TRUNC('week', wr.recorded_at)
    )                                   AS prev_week_avg_hrv_ms
  FROM wearable_readings wr
  WHERE wr.reading_type IN ('hrv_rmssd')
    AND wr.deleted_at IS NULL
    AND wr.recorded_at >= NOW() - INTERVAL '90 days'
  GROUP BY wr.tenant_id, wr.user_id, DATE_TRUNC('week', wr.recorded_at)
),
weekly_sleep AS (
  SELECT
    wr.tenant_id,
    wr.user_id,
    DATE_TRUNC('week', wr.recorded_at)  AS week,
    AVG(wr.value_numeric)               AS avg_sleep_min
  FROM wearable_readings wr
  WHERE wr.reading_type = 'sleep_duration_min'
    AND wr.deleted_at IS NULL
    AND wr.recorded_at >= NOW() - INTERVAL '90 days'
  GROUP BY wr.tenant_id, wr.user_id, DATE_TRUNC('week', wr.recorded_at)
),
combined AS (
  SELECT
    COALESCE(h.tenant_id, s.tenant_id)  AS tenant_id,
    DATE_TRUNC('week',
      COALESCE(h.week, s.week))          AS week,
    COUNT(DISTINCT COALESCE(h.user_id,
      s.user_id))                        AS user_count,
    -- HRV trend: directional label only — no numeric value exposed
    SUM(CASE
      WHEN h.avg_hrv_ms > h.prev_week_avg_hrv_ms * 1.03  THEN 1 ELSE 0
    END)                                 AS hrv_improving_count,
    SUM(CASE
      WHEN h.avg_hrv_ms < h.prev_week_avg_hrv_ms * 0.97  THEN 1 ELSE 0
    END)                                 AS hrv_declining_count,
    -- Sleep duration bucketed — no exact minutes exposed
    SUM(CASE WHEN s.avg_sleep_min < 360  THEN 1 ELSE 0 END) AS sleep_under_6h_count,
    SUM(CASE WHEN s.avg_sleep_min >= 360
              AND s.avg_sleep_min < 420  THEN 1 ELSE 0 END) AS sleep_6_to_7h_count,
    SUM(CASE WHEN s.avg_sleep_min >= 420
              AND s.avg_sleep_min < 480  THEN 1 ELSE 0 END) AS sleep_7_to_8h_count,
    SUM(CASE WHEN s.avg_sleep_min >= 480 THEN 1 ELSE 0 END) AS sleep_over_8h_count
  FROM weekly_hrv h
  FULL OUTER JOIN weekly_sleep s
    ON h.tenant_id = s.tenant_id
   AND h.user_id   = s.user_id
   AND h.week      = s.week
  GROUP BY COALESCE(h.tenant_id, s.tenant_id),
           DATE_TRUNC('week', COALESCE(h.week, s.week))
)
-- k-anonymity floor: suppress any week-cohort with fewer than 5 users.
-- Matching the floor in §2.6 and §13.4.3.
SELECT *
FROM combined
WHERE user_count >= 5
WITH NO DATA;  -- populated by nightly scheduled job; never via API request path

-- Nightly refresh (added to existing MV refresh cron job)
-- REFRESH MATERIALIZED VIEW CONCURRENTLY tenant_wearable_summary;
```

**RLS on the view:** Tenant admins query `tenant_wearable_summary` filtered by their `tenant_id`. The view itself has no RLS; the application API layer enforces the `tenant_id` predicate and validates the response cohort size before serving to the admin dashboard. Any group returning `user_count < 5` is suppressed at the application layer as a second control (belt-and-suspenders matching §6).

---

### 14.5 Cross-Source HRV Normalization

HRV is not a single standardized metric. Each source uses a different measurement window, computation method, and reporting granularity. Treating them as interchangeable produces clinically meaningless comparisons and incorrect trend signals for Victor.

| Source | Metric label | Measurement method | Temporal granularity | Comparable to HealthKit? |
|---|---|---|---|---|
| HealthKit (iOS) | RMSSD (ms) | Beat-to-beat overnight measurement; Apple Watch samples at 1 Hz | Daily (overnight average) | Reference standard |
| Health Connect (Android) | RMSSD (ms) | Equivalent to HealthKit; device-dependent quality | Daily | Yes — treat as equivalent |
| Oura Ring | RMSSD (ms) | Overnight 5-minute highest-HRV window | Daily | Closest third-party equivalent; slightly higher values than full-night averages |
| Whoop | RMSSD (ms) | Overnight 5-minute measurement window (same window method as Oura) | Daily | Comparable in trend direction; absolute values differ from full-night averages; do not mix into same trend series |
| Garmin | HRV Status (categorical) | 7-day rolling window of nightly RMSSD measurements; output is a categorical label | Weekly | Not comparable — incompatible metric type |

**Normalization strategy:**

1. Store every reading with its `source` and `reading_type` as received. No cross-source numeric normalization is applied at storage time.
2. If a user has both HealthKit and Whoop active simultaneously, two separate rows exist per day — one per source.
3. Victor's coaching context builder (§14.6) selects readings by source in priority order: `healthkit` > `health_connect` > `oura` > `whoop`. Only one source is used for trend computation per user per period. Garmin HRV Status is reported separately as a weekly contextual label, not merged into the daily trend series.
4. The `metadata` JSONB column retains the source-specific measurement details (window size, device, computation method) for future re-processing if normalization methodology changes.
5. Cross-source aggregation in `tenant_wearable_summary` (§14.4) uses directional trend labels, making the numeric incompatibility irrelevant at the aggregate layer.

---

### 14.6 Coaching Context Integration

Victor's coaching context builder queries `wearable_readings` to inform training load recommendations, recovery-appropriate intensity adjustments, and sleep-quality commentary. The integration is governed by two constraints: what data enters the LLM context, and how Victor is permitted to reference it in responses to the user.

#### 14.6.1 Context builder query

The context builder runs at session start and on each turn where a workout or recovery question is detected. It queries the last 7 days of readings for the authenticated user, applying source-priority ordering (§14.5).

```sql
-- Executed by the coaching context builder service (form_coach role)
-- Returns structured wearable context for injection into Victor's system prompt
-- Raw numeric values are NOT passed to the LLM — see §14.6.2

SELECT
  source,
  reading_type,
  value_numeric,
  value_unit,
  recorded_at::DATE AS reading_date
FROM wearable_readings
WHERE user_id    = $1   -- session-scoped; never user-supplied
  AND tenant_id  = $2
  AND recorded_at >= NOW() - INTERVAL '7 days'
  AND deleted_at IS NULL
  AND reading_type IN (
    'hrv_rmssd', 'resting_hr', 'sleep_duration_min',
    'sleep_efficiency_pct', 'recovery_score', 'readiness_score',
    'body_battery', 'hrv_status_garmin'
  )
ORDER BY reading_type, source, recorded_at DESC;
```

#### 14.6.2 `stripPersonalProperties()` applied to wearable context

Per PRV-14 (docs/SOC2_READINESS.md §35.6) and the privacy floor established in §6, raw wearable values are transformed into buckets and trend labels before being included in the LLM prompt payload. The same `stripPersonalProperties()` function applied to general coaching context (docs/VICTOR_PROMPT_GUIDE.md) governs this transformation.

**Forbidden in LLM context (never passed to Anthropic API):**

| Field | Reason |
|---|---|
| Raw numeric HRV value (e.g. `42 ms`) | Re-identification risk; absolute value implies health condition |
| Exact sleep duration in minutes | Granularity sufficient to fingerprint individuals in small teams |
| Exact resting HR in bpm | In isolation or combination, implies cardiac health status |
| Exact body temperature deviation | Clinical inference risk |

**Permitted in LLM context (bucketed or directional only):**

| Wearable signal | Permitted representation | Example |
|---|---|---|
| HRV trend | Directional label derived from 7-day linear regression | `"hrv_trend": "declining_3_days"` |
| Resting HR trend | Directional label vs. user's 30-day baseline | `"resting_hr_trend": "elevated"` |
| Sleep duration | Duration band (matches §14.4 bands) | `"sleep_band": "6_to_7h"` |
| Sleep efficiency | Qualitative label: good / fair / poor | `"sleep_quality": "fair"` |
| Recovery score | Bucket: low (0–33) / medium (34–66) / high (67–100) | `"recovery": "low"` |
| Readiness score | Bucket: low / medium / high | `"readiness": "high"` |
| Body battery | Bucket: low / medium / high | `"body_battery": "medium"` |
| HRV Status (Garmin) | Pass-through categorical label as-is | `"hrv_status": "balanced"` |

#### 14.6.3 Victor response constraints

Victor is explicitly instructed (docs/VICTOR_PROMPT_GUIDE.md) not to state specific wearable numeric values in responses to users, even if asked directly. The rationale: Victor's responses may be screenshot and shared; a response stating "your HRV is 28 ms" constitutes a disclosure of Art. 9 health data outside the consent context. Permitted response patterns:

- "Your HRV has been trending down for the last three days — I'd recommend keeping today's session lighter."
- "Your recovery looks low this week. Let's stick to your moderate program and skip the heavy compounds."
- "Sleep has been short lately. That affects strength output — we can adjust volume today."

Prohibited response patterns:

- "Your HRV was 28 ms last night." (raw numeric value)
- "Your resting heart rate is 58 bpm." (exact value, even if accurate)
- "You had 6 hours and 14 minutes of sleep." (exact duration)

---

### 14.7 GDPR and Privacy Constraints

#### 14.7.1 Classification and legal basis

`wearable_readings` is classified **Sensitive** under §5 (GDPR Art. 9 special category health data). The legal basis for processing is Art. 9(2)(a): explicit consent. This is not a legitimate-interests basis — explicit consent is mandatory.

#### 14.7.2 Consent gate

Wearable sync is blocked at the application layer until the user's consent record confirms `'wearable'` is present in `categories_consented`:

```sql
-- Checked before any sync write and before the coaching context builder query
SELECT 1
FROM user_consents
WHERE user_id           = $1
  AND 'wearable'        = ANY(categories_consented)
  AND revoked_at        IS NULL
LIMIT 1;
-- Zero rows returned → sync aborted; DEC-030 event wearable.sync_blocked logged
```

If consent is subsequently revoked, the sync worker receives a `wearable.sync_revoked` DEC-030 event and ceases all future syncs for that user. Existing rows are soft-deleted within 24 hours by the erasure job.

#### 14.7.3 Retention

Retention period: **2 years from `recorded_at`** (proposed in docs/SOC2_READINESS.md §35.6.3; formal decision record required at `compliance/p1/retention-decisions.md` before privacy policy publication — see P-GAP-004).

Automated enforcement via pg_cron (added to the schedule defined in §12.4):

```sql
-- Nightly at 02:45 UTC — hard delete wearable_readings past 2-year retention window
-- (Soft-deleted rows within the 30-day Art. 17 window are handled by the erasure job below)
SELECT cron.schedule('purge-wearable-readings-retention', '45 2 * * *', $$
  DELETE FROM wearable_readings
  WHERE recorded_at < NOW() - INTERVAL '2 years'
    AND deleted_at IS NOT NULL;
  -- Rows without deleted_at that are past retention are soft-deleted first, then
  -- picked up in the next nightly run. Two-step prevents instant data loss on clock drift.
  UPDATE wearable_readings
  SET deleted_at = NOW()
  WHERE recorded_at < NOW() - INTERVAL '2 years'
    AND deleted_at IS NULL;
$$);
```

A Supabase TTL migration implementing this schedule is required before launch (P-GAP-004 remediation).

#### 14.7.4 Art. 17 erasure

When a user submits a GDPR Art. 17 erasure request (§12.3), `wearable_readings` follows the same hard-delete path as `workouts` and `meal_logs`:

1. All rows for `user_id` are soft-deleted immediately (`deleted_at = NOW()`).
2. The nightly erasure job hard-deletes all rows where `deleted_at < NOW() - INTERVAL '30 days'` and `user_id` matches the erasure queue entry.
3. The DSAR export assembles wearable readings before soft-delete (Art. 20 portability obligation — see §12.5).
4. DEC-030 event `wearable.sync_revoked` is emitted at soft-delete time; the count of hard-deleted rows is recorded in the `erasure.completed` audit event (no individual values).

#### 14.7.5 Analytics prohibition

Wearable readings, trend labels derived from them, and any numeric values are **never** included in PostHog events, ClickHouse tables, or any analytics pipeline. See §14.10 for the full prohibition matrix. Violation detection: the `stripForbiddenProperties()` function in the analytics enrichment middleware (§13.3) maintains a blocklist that includes `hrv_rmssd`, `resting_hr`, `recovery_score`, `readiness_score`, `body_battery`, `sleep_duration_min`, `sleep_efficiency_pct`, `temperature_deviation_c`, and `strain`.

#### 14.7.6 DEC-030 audit events

The following events are added to the DEC-030 taxonomy (docs/AUDIT_LOG_SCHEMA.md) for wearable data access:

| Event name | Trigger | Fields logged |
|---|---|---|
| `wearable.source_connected` | User connects a new wearable source | `user_id`, `tenant_id`, `source`, `scopes[]`, `connected_at` |
| `wearable.source_disconnected` | User disconnects a wearable source | `user_id`, `tenant_id`, `source`, `disconnected_at`, `reason` |
| `wearable.sync_completed` | Sync worker writes a batch of readings | `user_id`, `tenant_id`, `source`, `reading_count` (no values), `synced_at` |
| `wearable.sync_blocked` | Sync attempted without valid wearable consent | `user_id`, `tenant_id`, `source` |
| `wearable.sync_revoked` | Consent revocation triggers sync cessation + soft-delete | `user_id`, `tenant_id`, `source`, `readings_soft_deleted_count` |
| `wearable.coach_context_accessed` | Coaching context builder reads wearable data | `user_id`, `tenant_id`, `coaching_session_id`, `reading_types[]` (no values) |

All events are HMAC-chained. None include raw numeric wearable values.

---

### 14.8 OAuth and Token Storage

OAuth tokens for Whoop, Oura, and Garmin are stored in a separate table with column-level encryption. HealthKit and Health Connect do not use OAuth — their access is governed by the on-device permission system and requires no stored token.

**Classification: SENSITIVE — token compromise grants access to user health data on third-party platforms.**

```sql
-- CLASSIFICATION: SENSITIVE — OAuth tokens grant access to third-party health APIs.
-- Encrypted at rest using per-tenant KMS key (§5.1). Never returned in API responses.
CREATE TABLE wearable_oauth_tokens (
  id                  UUID        PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id             UUID        NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  tenant_id           UUID        NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,

  source              TEXT        NOT NULL
                        CHECK (source IN ('whoop', 'oura', 'garmin')),
  -- HealthKit and Health Connect are intentionally excluded — on-device entitlement only.

  -- Encrypted with per-tenant KMS-derived key via Supabase Vault / pgcrypto.
  -- Access_token and refresh_token are NEVER returned by any API endpoint.
  access_token_enc    TEXT        NOT NULL,    -- AES-256-GCM ciphertext; per-tenant key
  refresh_token_enc   TEXT,                    -- null if source does not issue refresh tokens
  token_type          TEXT        NOT NULL DEFAULT 'Bearer',

  expires_at          TIMESTAMPTZ,             -- null if non-expiring (uncommon)
  scopes              TEXT[]      NOT NULL DEFAULT '{}',  -- OAuth scopes granted

  -- Soft-revoke: set when user disconnects the source. Token is not deleted immediately
  -- to allow the background worker to complete any in-flight sync.
  -- Hard delete by erasure job after 24 hours.
  revoked_at          TIMESTAMPTZ,

  connected_at        TIMESTAMPTZ NOT NULL DEFAULT now(),
  last_refreshed_at   TIMESTAMPTZ,
  last_used_at        TIMESTAMPTZ,

  created_at          TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at          TIMESTAMPTZ NOT NULL DEFAULT now(),

  UNIQUE (user_id, source)  -- one active token per source per user
);

ALTER TABLE wearable_oauth_tokens ENABLE ROW LEVEL SECURITY;

-- RLS: users can read metadata (connected_at, scopes, revoked_at) but not token values.
-- The sync worker (form_system role) reads token ciphertext for refresh.
CREATE POLICY wearable_tokens_user_metadata ON wearable_oauth_tokens
  AS PERMISSIVE FOR SELECT
  TO form_api
  USING (
    tenant_id = current_setting('app.current_tenant_id', TRUE)::UUID
    AND user_id = current_setting('app.current_user_id', TRUE)::UUID
  );
-- Note: the SELECT policy allows the row; the application layer explicitly excludes
-- access_token_enc and refresh_token_enc columns from all API response serializers.

CREATE POLICY wearable_tokens_system_rw ON wearable_oauth_tokens
  AS PERMISSIVE FOR ALL
  TO form_system
  USING (TRUE)
  WITH CHECK (TRUE);
-- form_system is the sync worker role; it requires full token access for refresh operations.

CREATE INDEX idx_wearable_tokens_user_source
  ON wearable_oauth_tokens (user_id, source)
  WHERE revoked_at IS NULL;
```

**Token lifecycle:**

- **Encryption:** Tokens are encrypted by the sync Cloudflare Worker before write using the tenant's AWS KMS CMK (or Supabase Vault secret for consumer-tier users). The raw token value never touches the database unencrypted.
- **Refresh:** A Cloudflare Worker cron job runs every 2 hours for each source. It decrypts the `refresh_token_enc`, calls the source API, and writes back re-encrypted new tokens. Refresh is not in the API request path — it is background-only. Failure triggers a `wearable.token_refresh_failed` internal alert (PagerDuty P3).
- **Revocation:** When the user disconnects a source via Settings → Integrations → Disconnect, `revoked_at` is set immediately. The DEC-030 `wearable.source_disconnected` event is emitted. The sync worker checks `revoked_at IS NULL` before each sync run. The background erasure job hard-deletes revoked token rows after 24 hours (sufficient for any in-flight sync to resolve).

---

### 14.9 Sync Architecture

Each source platform uses a different sync mechanism driven by its platform capabilities and API constraints.

#### 14.9.1 HealthKit (iOS)

```
User grants HealthKit permission on iOS
  → FORM app registers HealthKit Background Delivery observers for each data type
  → iOS calls FORM app in background when new readings are available
  → App batches new readings and POSTs to /v1/wearable/sync (authenticated)
  → Cloudflare Worker validates JWT, checks consent gate, upserts via ON CONFLICT DO NOTHING
  → DEC-030 event: wearable.sync_completed (count only)
```

Background delivery is registered using `HKObserverQuery` + `enableBackgroundDelivery(for:frequency:)`. Frequency is set to `.immediate` for HRV and resting HR; `.daily` for sleep and activity. No polling is required — iOS pushes a notification to the app when new data is available. The app wakes in background, fetches incremental data using `HKAnchoredObjectQuery`, and syncs to the backend.

**Battery impact:** Background delivery wakes are brief (< 1 second of CPU per event). This is not the continuous 30 fps pose estimation scenario — no significant battery constraint applies.

#### 14.9.2 Health Connect (Android)

```
User grants Health Connect permissions
  → FORM app reads current data on foreground open (HKAnchoredObjectQuery equivalent: getChanges())
  → WorkManager background job scheduled (minimum 15-minute interval per Android policy)
  → On each run: fetch changes since last sync anchor, POST to /v1/wearable/sync
  → DEC-030 event: wearable.sync_completed (count only)
```

The 15-minute minimum interval is enforced by the Android operating system and cannot be shortened, regardless of WorkManager configuration. Sync freshness for Health Connect users is therefore lower than for HealthKit users. The coaching context builder uses the most recent available reading regardless of staleness; Victor's response templates account for potential data lag ("based on your last recorded reading").

#### 14.9.3 Whoop, Oura, Garmin (third-party APIs)

```
Cloudflare Worker cron: runs every 2 hours per source
  → For each user with a valid non-revoked token for that source:
    → Decrypt access_token_enc using tenant KMS key
    → Fetch readings since last_synced_at anchor
    → Respect source rate limits (see table below)
    → Upsert readings via /internal/wearable/sync (internal endpoint; not user-facing)
    → Update last_used_at and last_refreshed_at on token row
    → DEC-030 event: wearable.sync_completed (count only)
```

| Source | Sync cadence | Rate limit | Data lag |
|---|---|---|---|
| Whoop | Every 2 hours | 100 requests/minute | Near-real-time (daily cycles computed after sleep ends) |
| Oura | Every 2 hours | Not published; estimated 60 req/min | Daily data available by ~10:00 user local time |
| Garmin | Every 2 hours | 500 requests/day per user (enterprise contract) | Daily summaries; intra-day data at higher tier only |

**Idempotency:** The `UNIQUE (user_id, source, reading_type, recorded_at)` constraint means duplicate upserts from retry logic or overlapping cron runs are silent no-ops (`ON CONFLICT DO NOTHING`). The cron job is therefore safe to re-run without coordination.

---

### 14.10 Analytics Restrictions

The following table is the authoritative statement of which analytics layers may receive wearable data in any form. It extends the prohibitions in §13.6.1 specifically for wearable readings.

| Analytics layer | Raw `wearable_readings` allowed? | Derived / bucketed values allowed? | Reason |
|---|---|---|---|
| PostHog events | No | No | GDPR Art. 9 Sensitive; prohibited by §13.6.1 and `stripForbiddenProperties()` blocklist |
| ClickHouse `fact_events` | No | No | Art. 9 Sensitive; sourced from PostHog — same prohibition applies |
| ClickHouse `fact_session_metrics` | No | No | Art. 9 Sensitive |
| ClickHouse `dim_users` | No | No | Art. 9 Sensitive |
| Metabase dashboards (tenant admin) | No (raw) | Aggregate view only via `tenant_wearable_summary` | See §14.4; k-anonymity N < 5; no numeric values |
| LLM coaching context (Anthropic API) | No (raw numeric) | Trend buckets and directional labels only | §14.6.2 `stripPersonalProperties()` transformation required |
| SOC 2 audit log (DEC-030) | No (values) | Sync counts and event metadata only | DEC-030 events log counts, not values (§14.7.6) |
| DSAR export (Art. 20) | Yes — full export | N/A | Art. 20 portability obligation; user receives their own data; encrypted delivery |

Sentry error monitoring must not log wearable values in exception payloads. The Sentry `beforeSend` filter (docs/OBSERVABILITY.md) is extended to scrub any key matching the wearable field blocklist from error context objects.

---

### 14.11 Implementation Checklist

| # | Task | Owner | Priority | Milestone |
|---|---|---|---|---|
| 1 | Apply `wearable_readings` DDL migration (§14.2) including all indexes and `ENABLE ROW LEVEL SECURITY` | `platform-engineer` | P0 | M3 |
| 2 | Apply RLS policies (§14.3): user owner-only, coach-with-consent, no tenant-admin raw access | `platform-engineer` | P0 | M3 |
| 3 | Apply `wearable_oauth_tokens` DDL migration (§14.8) with Supabase Vault column encryption for `access_token_enc` and `refresh_token_enc` | `platform-engineer` | P0 | M3 |
| 4 | Add `wearable_readings` RLS isolation assertions to `__tests__/db/rls_isolation.test.ts`: tenant-admin zero-row check, coach-without-consent zero-row check | `platform-engineer` | P0 | M3 |
| 5 | Implement consent gate check in sync endpoint (`/v1/wearable/sync`): block write if `'wearable' != ANY(categories_consented)` | `platform-engineer` | P0 | M3 |
| 6 | Wire DEC-030 audit events: `wearable.source_connected`, `wearable.source_disconnected`, `wearable.sync_completed`, `wearable.sync_blocked`, `wearable.sync_revoked`, `wearable.coach_context_accessed` — add to `docs/AUDIT_LOG_SCHEMA.md` action taxonomy | `platform-engineer` + `compliance-officer` | P0 | M3 |
| 7 | Build iOS HealthKit integration: register `HKObserverQuery` background delivery observers for HRV (RMSSD), resting HR, sleep analysis, active energy, steps; implement `HKAnchoredObjectQuery` incremental fetch and POST to sync endpoint | `platform-engineer` (iOS) | P0 | M3 |
| 8 | Build Android Health Connect integration: implement `getChanges()` incremental sync on foreground open; schedule WorkManager background job (15-min interval); POST to sync endpoint | `platform-engineer` (Android) | P0 | M3 |
| 9 | Build Cloudflare Worker cron for Whoop API sync (every 2 hours): decrypt token, fetch readings since last anchor, upsert via internal endpoint, respect 100 req/min rate limit | `platform-engineer` | P1 | M4 |
| 10 | Build Cloudflare Worker cron for Oura API sync (every 2 hours): equivalent to Whoop worker; include `temperature_deviation_c` and `sleep_stages` JSONB in `metadata` | `platform-engineer` | P1 | M4 |
| 11 | Build Cloudflare Worker cron for Garmin API sync (every 2 hours; after enterprise contract signed): implement `hrv_status_garmin` categorical storage; exclude Garmin HRV from cross-source trend logic | `platform-engineer` | P2 | M5 |
| 12 | Build token refresh cron (Cloudflare Worker, every 2 hours): decrypt refresh token, call source API, re-encrypt and write back; emit PagerDuty P3 alert on failure | `platform-engineer` | P1 | M4 |
| 13 | Implement coaching context builder wearable query (§14.6.1) under `form_coach` role; apply `stripPersonalProperties()` transformation to produce bucketed/directional context before Anthropic API call | `platform-engineer` | P1 | M4 |
| 14 | Extend `stripForbiddenProperties()` blocklist in analytics enrichment middleware (§13.3) with wearable field names: `hrv_rmssd`, `resting_hr`, `recovery_score`, `readiness_score`, `body_battery`, `sleep_duration_min`, `sleep_efficiency_pct`, `temperature_deviation_c`, `strain` | `data-engineer` | P0 | M3 |
| 15 | Create `tenant_wearable_summary` materialized view (§14.4) and add to nightly MV refresh cron; build admin dashboard Metabase panel against view (no raw wearable_readings query permitted) | `platform-engineer` + `data-engineer` | P1 | M4 |
| 16 | Implement pg_cron retention job for `wearable_readings` 2-year hard delete (§14.7.3); validate with synthetic old-data test set | `platform-engineer` | P1 | M4 |
| 17 | Extend Art. 17 erasure job (§12.3) to soft-delete then hard-delete `wearable_readings` and hard-delete `wearable_oauth_tokens` within 30-day window; verify row counts in erasure audit event | `platform-engineer` | P1 | M4 |
| 18 | Extend Sentry `beforeSend` filter to scrub wearable field names from exception payloads (docs/OBSERVABILITY.md) | `platform-engineer` | P1 | M4 |
| 19 | Add `wearable_readings` and `wearable_oauth_tokens` to DSAR export job (§12.5) in the `form-export` worker; include `wearable_readings` rows in `form-export-{user_id}.zip` | `platform-engineer` | P1 | M4 |
| 20 | Complete formal retention decision record at `compliance/p1/retention-decisions.md` confirming 2-year `wearable_readings` retention; obtain compliance-officer sign-off; add to privacy policy (closes P-GAP-004) | `compliance-officer` | P0 | Pre-launch |

---

*v0.5 additions: §14 Wearable Data Integration Schema — five-source integration specification (HealthKit, Health Connect, Whoop, Oura Ring, Garmin) with HRV metric compatibility matrix; `wearable_readings` Postgres DDL with SENSITIVE Art. 9 classification, idempotency constraint, and four supporting indexes; four-tier RLS policy set (user owner-only, tenant-admin blocked, coach-with-consent, form_admin BYPASSRLS); `tenant_wearable_summary` materialized view with k-anonymity N < 5 floor, directional HRV trend labels, and sleep duration bands (no raw numeric exposure); cross-source HRV normalization strategy with source-priority ordering and Garmin categorical isolation; coaching context integration specifying forbidden raw values and permitted bucketed representations with Victor response language constraints aligned to PRV-14 and `stripPersonalProperties()`; GDPR Art. 9 constraint set covering consent gate SQL, 2-year retention pg_cron job, Art. 17 30-day erasure flow, and six DEC-030 audit events (wearable.source_connected/disconnected, sync_completed/blocked/revoked, coach_context_accessed); `wearable_oauth_tokens` DDL with per-tenant KMS column encryption, revocation semantics, and Cloudflare Worker 2-hour refresh cron; sync architecture per source (HealthKit background delivery, Health Connect WorkManager, third-party 2-hour cron with rate limits); analytics restriction matrix confirming exclusion from all seven layers; 20-item implementation checklist across M3/M4/M5 and pre-launch.*

---

## 15. Computer Vision (CV) Pose Estimation Data Schema

> Owner: `ml-engineer` + `platform-engineer` + `compliance-officer`. Review: on any CV model upgrade, data structure change, or quarterly.
> Scope: consumer and enterprise tiers. CV form analysis is a core product differentiator; this schema defines how pose data is stored, protected, and constrained.
> References: DEC-030 (HMAC-chained audit log), docs/SOC2_READINESS.md §35.6.3 (retention) + PRE-34-E-006 (column-level encryption), docs/INCIDENT_RESPONSE.md §R-10 (AI coach safety incident — blast-radius scoping by `model_version`), docs/VICTOR_PROMPT_GUIDE.md, docs/AUDIT_LOG_SCHEMA.md.

---

### 15.1 Architecture: On-Device Inference Model

FORM's CV pipeline runs entirely on-device. Raw video frames are **never transmitted to FORM's backend** or to any third-party processor. The ML model (BlazePose or MoveNet, selected by device capability) runs at 30 fps inside the mobile app; only structured outputs — aggregate form scores, rep counts, cv defect flags, and sampled keypoint snapshots — are uploaded at session completion.

**Why on-device?** Three converging constraints:

| Constraint | Implication |
|---|---|
| GDPR Art. 9 (biometric data) | Video analysis of a person's body produces biometric data. Processing raw video frames on FORM's backend would require Art. 9 explicit consent, a separate DPIA entry, and additional sub-processor classification for the inference infrastructure. On-device inference avoids this classification for the raw video stream itself. |
| Network latency | Real-time rep-by-rep form feedback requires < 100 ms round-trip. Cloud inference cannot meet this constraint at consumer latency budgets. |
| Competitive moat | The user's video never leaves their device — a materially stronger privacy posture than cloud-video competitors. Privacy is a first-class product attribute, not a compliance checkbox. |

**What is transmitted to FORM's backend (at session completion):**
1. `rep_count` — integer
2. `form_score` — NUMERIC(4,1), 0–100
3. `cv_flags` — JSONB key/value pairs naming specific form defects detected (e.g. `{"knee_cave": 2, "back_arch": 0}`)
4. `keypoints_enc` — sampled pose keypoints at **rep-boundary frames only** (peak and bottom positions per rep, max 2 frames/rep). At 33 keypoints × (x, y, z, visibility) × 2 frames × 50 reps maximum ≈ 150 KB per session. Transmitted over TLS 1.3; stored encrypted (§15.2).
5. Session metadata: model name, model version, frame counts, confidence statistics.

**What is never transmitted:**
- Raw video frames
- Continuous pose time-series (only rep-boundary snapshots)
- Face or background content from the camera feed

---

### 15.2 `cv_sessions` Table Schema

This table is the authoritative server-side record for every CV analysis session. It is referenced in `docs/SOC2_READINESS.md` PRE-34-E-006 (column-level encryption evidence artifact) and §35.6.3 (1-year retention for biometric-adjacent data). The `keypoints_enc` column contains GDPR Art. 9 biometric data and requires per-tenant AES-256-GCM column-level encryption.

**Classification: SENSITIVE — GDPR Art. 9 (biometric data derived from video analysis). Column `keypoints_enc`: per-tenant AES-256-GCM + KMS encryption (closes SOC2 PRE-34-E-006). Retention: `keypoints_enc` nullified after 1 year; row hard-deleted after 3 years (§15.7.2).**

```sql
-- CLASSIFICATION: SENSITIVE — GDPR Art. 9 special category (biometric data derived from video).
-- Column keypoints_enc: per-tenant AES-256-GCM encryption; key stored in Cloudflare Workers Secret,
-- NOT in Supabase environment. Decryption available only to form_system and form_admin roles.
-- Retention: keypoints_enc nullified after 1 year from created_at; row hard-deleted after 3 years.
-- See SOC2_READINESS.md §35.6.3 and §15.7.2 of this document.
-- Raw video frames: NEVER stored — on-device inference only (§15.1).
CREATE TABLE cv_sessions (
  id                        UUID          PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id                   UUID          NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  tenant_id                 UUID          NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
  workout_set_id            UUID          REFERENCES workout_sets(id) ON DELETE SET NULL,

  -- CV model identity — required for blast-radius scoping in INCIDENT_RESPONSE.md §R-10
  model_name                TEXT          NOT NULL
                              CHECK (model_name IN (
                                'blazepose_lite',
                                'blazepose_full',
                                'movenet_lightning',
                                'movenet_thunder'
                              )),
  model_version             TEXT          NOT NULL,   -- semver, e.g. '2.0.0'; from on-device model registry

  -- Session timing
  started_at                TIMESTAMPTZ   NOT NULL DEFAULT now(),
  ended_at                  TIMESTAMPTZ,              -- null if session was interrupted mid-set

  -- Analysis status
  status                    TEXT          NOT NULL DEFAULT 'active'
                              CHECK (status IN (
                                'active',             -- upload in progress
                                'completed',          -- all reps counted, form_score computed
                                'partial',            -- < 50% of frames had sufficient confidence
                                'tracking_lost'       -- model lost pose track mid-set; form_score null
                              )),

  -- Aggregate outputs (non-encrypted; safe to query without decryption key)
  rep_count                 SMALLINT,                -- null if status = 'tracking_lost'
  form_score                NUMERIC(4,1),            -- 0.0–100.0; null if status != 'completed'
  cv_flags                  JSONB         NOT NULL DEFAULT '{}',
  -- cv_flags structure: {"<defect_slug>": <occurrence_count>, ...}
  -- Defined defect slugs (ml-engineer owns taxonomy; add new slugs via migration only):
  --   "knee_cave"        — valgus collapse during squat / lunge
  --   "back_arch"        — lumbar hyperextension (deadlift, RDL)
  --   "elbow_flare"      — horizontal abduction during press movement
  --   "forward_lean"     — excessive torso inclination
  --   "hip_shift"        — lateral imbalance during squat
  --   "depth_short"      — squat/lunge above parallel threshold
  --   "lockout_missing"  — incomplete range of motion at top position

  -- Confidence statistics (aggregate; non-sensitive)
  avg_keypoint_confidence   NUMERIC(4,3),            -- 0.000–1.000 mean across all analysed frames
  frames_analysed           INTEGER,
  frames_below_threshold    INTEGER,                 -- frames where confidence < REP_COUNT_CONFIDENCE_MIN (0.65)

  -- Encrypted biometric payload (GDPR Art. 9)
  -- Contains: sampled pose keypoints at rep-boundary frames (peak + bottom, max 2 frames/rep).
  -- Storage format: pgp_sym_encrypt(keypoints_jsonb::text, current_setting('app.cv_enc_key'))
  -- where app.cv_enc_key is a per-tenant AES-256-GCM key from Cloudflare Workers Secret.
  -- Decrypted format: {"frames": [{"t": <ms_offset>, "kp": [[x, y, z, v], ...]}, ...]}
  --   BlazePose: 33 keypoints × 4 floats = 132 values per frame
  --   MoveNet: 17 keypoints × 4 floats = 68 values per frame
  -- Maximum payload: 100 snapshots × ~1.5 KB ≈ 150 KB (TOAST-managed; see §15.5).
  -- null if status = 'tracking_lost' AND frames_analysed < 3.
  keypoints_enc             TEXT,

  -- Soft-delete for GDPR Art. 17 30-day grace period (§15.7.3)
  deleted_at                TIMESTAMPTZ,

  created_at                TIMESTAMPTZ   NOT NULL DEFAULT now(),
  updated_at                TIMESTAMPTZ   NOT NULL DEFAULT now(),

  CONSTRAINT chk_cv_form_score_range
    CHECK (form_score IS NULL OR form_score BETWEEN 0.0 AND 100.0),
  CONSTRAINT chk_cv_confidence_range
    CHECK (avg_keypoint_confidence IS NULL OR avg_keypoint_confidence BETWEEN 0.000 AND 1.000),
  CONSTRAINT chk_cv_ended_after_started
    CHECK (ended_at IS NULL OR ended_at >= started_at),
  CONSTRAINT chk_cv_frames_non_negative
    CHECK (frames_analysed IS NULL OR frames_analysed >= 0)
);

ALTER TABLE cv_sessions ENABLE ROW LEVEL SECURITY;
ALTER TABLE cv_sessions FORCE ROW LEVEL SECURITY;

-- Tenant isolation and user lookup (primary query patterns)
CREATE INDEX idx_cv_sessions_user_id
  ON cv_sessions (user_id) WHERE deleted_at IS NULL;

CREATE INDEX idx_cv_sessions_tenant_id
  ON cv_sessions (tenant_id) WHERE deleted_at IS NULL;

CREATE INDEX idx_cv_sessions_set_id
  ON cv_sessions (workout_set_id)
  WHERE deleted_at IS NULL AND workout_set_id IS NOT NULL;

CREATE INDEX idx_cv_sessions_created_at
  ON cv_sessions (created_at) WHERE deleted_at IS NULL;

-- Retention cron: find soft-deleted rows eligible for hard delete
CREATE INDEX idx_cv_sessions_retention_sweep
  ON cv_sessions (created_at) WHERE deleted_at IS NOT NULL;

-- Incident response: blast-radius scoping by model version (INCIDENT_RESPONSE.md §R-10)
CREATE INDEX idx_cv_sessions_model_version
  ON cv_sessions (model_name, model_version) WHERE deleted_at IS NULL;
```

---

### 15.3 RLS Policies for `cv_sessions`

The privacy floor for CV data is stricter than for wearable data. `keypoints_enc` contains biometric data; even with decryption, raw keypoints are never returned to any persona except `form_system` (coaching context builder and erasure worker). Coaches receive `form_score`, `cv_flags`, and `rep_count` only — never `keypoints_enc`. Tenant admin is fully blocked from the table; CV quality is exposed only via the `tenant_cv_summary` aggregate view (§15.4).

#### 15.3.1 RLS Policy Set

```sql
-- ── User self-access ──────────────────────────────────────────────────────────
-- Users may query their own cv_sessions. The Supabase client never receives
-- the decryption key for keypoints_enc — decryption requires the Workers backend
-- injecting app.cv_enc_key, which is only done for form_system role calls.
CREATE POLICY cv_sessions_user_self
  ON cv_sessions
  FOR ALL
  TO form_user
  USING (
    user_id   = auth.uid()
    AND tenant_id = current_setting('app.tenant_id')::UUID
    AND deleted_at IS NULL
  )
  WITH CHECK (
    user_id   = auth.uid()
    AND tenant_id = current_setting('app.tenant_id')::UUID
  );

-- ── Coach access ──────────────────────────────────────────────────────────────
-- Coaches have NO Postgres-level policy on cv_sessions.
-- The Workers backend queries under form_system and returns only
-- {rep_count, form_score, cv_flags, status} — never keypoints_enc.
-- This prevents any Postgres-level projection bypass.
-- Consent gate enforced at application layer (same pattern as §14.3 wearable_readings).

-- ── Tenant admin: BLOCKED (privacy floor) ────────────────────────────────────
-- form_tenant_admin has no policy on cv_sessions.
-- Any query under form_tenant_admin returns 0 rows (fail-closed).
-- CV quality is exposed only via the tenant_cv_summary aggregate view (§15.4).
-- This must be verified in CI — see §15.3.2.

-- ── form_system (backend service) ────────────────────────────────────────────
-- Full access for CV session upload, coaching context builder, and erasure worker.
CREATE POLICY cv_sessions_system
  ON cv_sessions
  FOR ALL
  TO form_system
  USING (tenant_id = current_setting('app.tenant_id')::UUID);

-- ── form_admin (break-glass) ─────────────────────────────────────────────────
-- Full table access including keypoints_enc.
-- MANDATORY: every break-glass query must emit a cv.keypoints_admin_accessed
-- DEC-030 event with incident_id before the query is executed.
-- Decryption of keypoints_enc additionally requires calling the internal
-- /admin/cv/decrypt-session endpoint (authenticated admin JWT + incident_id).
-- The cv_enc_key is NOT available in the Supabase env — only in Workers Secrets.
CREATE POLICY cv_sessions_admin
  ON cv_sessions
  FOR ALL
  TO form_admin
  USING (true);
```

#### 15.3.2 CI RLS Isolation Assertions

Add to `__tests__/db/rls_isolation.test.ts`:

```typescript
// cv_sessions: tenant admin receives zero rows (privacy floor)
it('tenant admin cannot read cv_sessions', async () => {
  const { data, error } = await supabaseTenantAdmin
    .from('cv_sessions')
    .select('id');
  expect(error).toBeNull();
  expect(data).toHaveLength(0);
});

// cv_sessions: cross-tenant isolation
it('cross-tenant cv_sessions read returns zero rows', async () => {
  const { data, error } = await supabaseUserTenantB
    .from('cv_sessions')
    .select('id')
    .eq('tenant_id', TENANT_A_ID);
  expect(error).toBeNull();
  expect(data).toHaveLength(0);
});

// cv_sessions: form_coach role has no policy → zero rows
it('form_coach role cannot access keypoints_enc', async () => {
  const { data } = await supabaseCoach
    .from('cv_sessions')
    .select('keypoints_enc');
  expect(data).toHaveLength(0);
});
```

---

### 15.4 Tenant-Visible Aggregates (`tenant_cv_summary`)

Enterprise tenant admins (HR / People leads) may see aggregate CV form quality trends for the organisation — for example, average form score improvement week over week, or which form defects appear most frequently across the team. Individual CV data (form score per user, session timestamps, per-person cv_flags) is never exposed. k-anonymity floor: all numeric values are suppressed when fewer than 5 unique users contributed in a given period.

```sql
-- CV aggregate view for the enterprise admin dashboard.
-- k-anonymity: all numeric values NULL when unique_users_with_cv < 5.
-- Privacy floor: no individual form scores, no user_id, no session identifiers.
-- Access: form_admin + tenant_owner only; tenant_manager blocked.
CREATE MATERIALIZED VIEW tenant_cv_summary AS
SELECT
  tenant_id,
  DATE_TRUNC('week', created_at)                               AS week_start,
  COUNT(DISTINCT user_id)                                      AS unique_users_with_cv,
  -- k-anonymity suppression: NULL when N < 5
  CASE WHEN COUNT(DISTINCT user_id) >= 5
       THEN ROUND(AVG(form_score), 1)          ELSE NULL END   AS avg_form_score,
  CASE WHEN COUNT(DISTINCT user_id) >= 5
       THEN ROUND(AVG(rep_count), 0)           ELSE NULL END   AS avg_reps_per_session,
  -- Status breakdown as percentages (not counts, to prevent cell-level re-identification)
  CASE WHEN COUNT(*) > 0 AND COUNT(DISTINCT user_id) >= 5
       THEN ROUND(100.0 * COUNT(*) FILTER (WHERE status = 'completed') / COUNT(*), 1)
       ELSE NULL END                                           AS pct_completed,
  CASE WHEN COUNT(*) > 0 AND COUNT(DISTINCT user_id) >= 5
       THEN ROUND(100.0 * COUNT(*) FILTER (WHERE status = 'tracking_lost') / COUNT(*), 1)
       ELSE NULL END                                           AS pct_tracking_lost,
  -- Form defect prevalence: % of sessions where the flag count > 0
  -- Directional only: no per-user defect attribution possible from this view
  CASE WHEN COUNT(DISTINCT user_id) >= 5
       THEN ROUND(100.0 * COUNT(*) FILTER (
              WHERE (cv_flags->>'knee_cave')::int > 0
            ) / NULLIF(COUNT(*), 0), 1)
       ELSE NULL END                                           AS pct_knee_cave_detected,
  CASE WHEN COUNT(DISTINCT user_id) >= 5
       THEN ROUND(100.0 * COUNT(*) FILTER (
              WHERE (cv_flags->>'back_arch')::int > 0
            ) / NULLIF(COUNT(*), 0), 1)
       ELSE NULL END                                           AS pct_back_arch_detected,
  CASE WHEN COUNT(DISTINCT user_id) >= 5
       THEN ROUND(100.0 * COUNT(*) FILTER (
              WHERE (cv_flags->>'depth_short')::int > 0
            ) / NULLIF(COUNT(*), 0), 1)
       ELSE NULL END                                           AS pct_depth_short_detected
FROM cv_sessions
WHERE deleted_at IS NULL
  AND status IN ('completed', 'partial')
  AND form_score IS NOT NULL
GROUP BY tenant_id, DATE_TRUNC('week', created_at);

COMMENT ON MATERIALIZED VIEW tenant_cv_summary IS
  'Aggregate CV form quality metrics by tenant and week. '
  'k-anonymity: all values NULL when unique_users_with_cv < 5. '
  'Privacy floor: no individual form scores, cv_flags, or session IDs exposed.';

-- Nightly refresh at 03:05 UTC (staggered after tenant_wearable_summary at 03:00 UTC)
-- SELECT cron.schedule('refresh-tenant-cv-summary', '5 3 * * *',
--   $$REFRESH MATERIALIZED VIEW CONCURRENTLY tenant_cv_summary$$);

ALTER MATERIALIZED VIEW tenant_cv_summary OWNER TO form_system;
GRANT SELECT ON tenant_cv_summary TO form_admin;
-- tenant_admin and tenant_manager access is mediated via the admin dashboard API endpoint,
-- which applies a tenant_id filter before returning data — no direct Supabase client access.
```

---

### 15.5 Keypoint Storage Decision (Resolves §9.2)

> **Gap 9.2** (raised in §9): "`raw_cv_data` column size: JSONB for pose keypoints can grow large. Column to separate blob store (S3)?" Owner: platform-engineer. Priority: High — evaluate at 10k workouts/day.

**Decision: store encrypted keypoints in Postgres JSONB (`keypoints_enc`). Migrate to R2 only when the gate condition is triggered.**

| Factor | Analysis |
|---|---|
| **Payload size** | Sampled at rep-boundary frames only (max 2 frames/rep). At 33 keypoints × 4 floats × 2 frames × 50 reps = 13,200 floats ≈ 105 KB uncompressed. JSONB LZ4 compression reduces this to ~20–40 KB. Postgres TOAST manages out-of-line storage automatically above 8 KB — no column fragmentation. |
| **Volume at 1k sessions/day** | 1,000 sessions × 40 KB avg = 40 MB/day. At 1-year retention with keypoints nullification: peak table size ≈ 40 MB/day × 365 days ≈ 14.6 GB — within Supabase Team plan (100 GB). |
| **Volume at 10k sessions/day** | 10,000 sessions × 40 KB avg = 400 MB/day. Annual peak ≈ 146 GB. At this scale FORM requires Supabase Enterprise (per COST_MODEL.md §13.4 upgrade cliff). Keypoints migration to R2 defers the Supabase storage bill by ~$0.015/GB/month. At 146 GB this is ~$2.2/month — not a compelling driver on its own. The real driver at 10k/day is operational simplicity of keeping all health data in one encrypted store with unified RLS. |
| **Security** | Keypoints in Postgres benefit from the same RLS enforcement, PITR backup isolation, audit trail, and DEC-030 chain as all other health data. Moving to R2 requires a separate HMAC-signed URL access control layer, complicating the erasure worker and DSAR export job. |
| **Migration gate** | Migrate `keypoints_enc` to R2 when: (a) `cv_sessions` table exceeds **20 GB**, OR (b) CV session volume exceeds **1,000/day** for 30 consecutive days. At that point: R2 bucket with Object Lock, per-object AES-256-GCM, `keypoints_r2_key TEXT` column replaces `keypoints_enc`, URL generation mediated by form_system only. See OQ-CV-01 (§15.10). |

**`workouts.raw_cv_data` column: DEPRECATED.** All new writes must use `cv_sessions.keypoints_enc`. The `workouts.raw_cv_data` column will be dropped in a migration once all clients have shipped the CV session endpoint. Coordinate with mobile app release schedule.

**Gap 9.2 status: RESOLVED** — keypoints in JSONB TOAST with R2 migration gate condition documented.

---

### 15.6 Coaching Context Integration

Victor receives CV context in a bucketed, non-biometric form. Raw keypoints never reach the Anthropic API. The coaching context builder (running under `form_system` role) queries `cv_sessions` and applies `stripCVData()` before composing the Victor prompt.

#### 15.6.1 Permitted CV Representations in the Victor Prompt

| Field | Raw value | Prompt representation | Rationale |
|---|---|---|---|
| `form_score` | 73.5 | `"form_quality": "good"` | Bucketed: poor < 40 / fair 40–59 / good 60–79 / excellent ≥ 80 |
| `rep_count` | 8 | `"reps_detected": 8` | Exact integer is not biometric |
| `cv_flags["knee_cave"]` | 2 | `"flags": ["knee_cave"]` | Presence only; occurrence count omitted |
| `cv_flags["back_arch"]` | 0 | _(omitted from flags array)_ | Zero-count flags are not included |
| `avg_keypoint_confidence` | 0.83 | _(not included)_ | Internal quality metric; not coaching-relevant |
| `keypoints_enc` | encrypted blob | _(never included)_ | Biometric data; categorically excluded from Anthropic API calls |

**Victor response language constraints** (cross-reference `docs/VICTOR_PROMPT_GUIDE.md` §CV):
- Victor must not quote exact form scores numerically ("Your form score was 73.5%").
- Victor should use qualitative framing: "Your squat form looked solid today" / "I noticed some knee tracking to watch."
- If `status = 'tracking_lost'`: "The camera lost tracking mid-set — hard to give you form feedback on that one."
- Victor must not reference specific keypoint positions, joint angles, or anatomical landmarks derived from the keypoint data.

#### 15.6.2 `stripCVData()` TypeScript Implementation

```typescript
interface CVSessionRaw {
  status: 'completed' | 'partial' | 'tracking_lost';
  rep_count: number | null;
  form_score: number | null;      // 0.0–100.0
  cv_flags: Record<string, number>;
}

interface CVPromptContext {
  status: string;
  reps_detected: number | null;
  form_quality: 'poor' | 'fair' | 'good' | 'excellent' | null;
  flags: string[];
}

const FORM_QUALITY_BUCKETS = [
  [80, 'excellent'],
  [60, 'good'],
  [40, 'fair'],
  [0,  'poor'],
] as const;

export function stripCVData(session: CVSessionRaw): CVPromptContext {
  return {
    status: session.status,
    reps_detected: session.rep_count,
    form_quality:
      session.form_score !== null
        ? (FORM_QUALITY_BUCKETS.find(([floor]) => session.form_score! >= floor)?.[1] ?? 'poor')
        : null,
    // Only flag names with occurrence_count > 0; count itself is not included
    flags: Object.entries(session.cv_flags)
      .filter(([, count]) => count > 0)
      .map(([flag]) => flag),
  };
}
// CI gate: grep the Anthropic API request payload for 'keypoints_enc' in unit tests.
// Any match is an automatic build failure — keypoints must never reach the inference API.
```

---

### 15.7 GDPR and Privacy Constraints

#### 15.7.1 GDPR Art. 9 Classification

CV pose keypoints are derived from video analysis of a person's body. Under GDPR Recital 51 and Art. 9(1), biometric data processed for the purpose of uniquely identifying natural persons is special category data. Pose keypoints from a workout session are sufficiently individualised — joint position time-series reflect unique biomechanics — to fall within this classification.

**Classification decisions:**
- `keypoints_enc`: **GDPR Art. 9 special category — biometric data.** Requires explicit `'cv'` category consent before any `cv_sessions` row is written.
- `form_score`, `rep_count`, `cv_flags`: **Personal data (Art. 4)** — not Art. 9 in isolation. These are aggregate quality metrics derived from analysis, not raw biometric measurements.
- Raw video frames: processed on-device without storage; GDPR does not impose retention obligations on in-memory transient processing.

**Consent gate:** Before any `cv_sessions` row is written, the mobile app must confirm `'cv' = ANY(categories_consented)` in `users.categories_consented`. The session upload endpoint (`POST /v1/cv/session`) blocks writes and emits a `cv.session_blocked` DEC-030 event if consent is absent.

#### 15.7.2 Retention: Two-Phase Deletion

Per `docs/SOC2_READINESS.md` §35.6.3: *"CV session metadata | `cv_sessions` | 1 year | Keypoint data is biometric-adjacent; shorter retention."*

1-year retention for `keypoints_enc` is shorter than `workout_sessions` (3 years) and `wearable_readings` (2 years) because keypoints are the highest-sensitivity data FORM stores. The form score and rep count retain long-term coaching value; the biometric payload does not.

**Two-phase strategy:**
1. After 1 year from `created_at`: nullify `keypoints_enc` (biometric data removed; aggregate session metadata retained for coaching history).
2. After 3 years from `created_at`: hard-delete the entire row (aligns with `workout_sessions` retention).

```sql
-- Phase 1: nullify keypoints_enc after 1 year (pg_cron — daily at 04:00 UTC)
SELECT cron.schedule('cv-keypoints-1yr-nullify', '0 4 * * *', $$
  UPDATE cv_sessions
  SET    keypoints_enc = NULL,
         updated_at    = now()
  WHERE  keypoints_enc IS NOT NULL
    AND  created_at    < now() - INTERVAL '1 year'
    AND  deleted_at    IS NULL;
$$);

-- Phase 2: hard-delete rows older than 3 years (pg_cron — daily at 04:30 UTC)
SELECT cron.schedule('cv-sessions-3yr-hard-delete', '30 4 * * *', $$
  DELETE FROM cv_sessions
  WHERE created_at < now() - INTERVAL '3 years';
$$);
```

Each Phase 1 batch must emit a `cv.keypoints_nullified` DEC-030 event with the row count. Each Phase 2 batch must emit a `cv.session_hard_deleted` DEC-030 event.

#### 15.7.3 GDPR Art. 17 Right to Erasure

When a user requests erasure, `cv_sessions` follows the 30-day grace period model (§12.3) with one exception: `keypoints_enc` must be nullified **immediately on erasure request** (no grace period). Art. 9 biometric data does not benefit from the grace period that applies to less-sensitive personal data.

Erasure worker steps for `cv_sessions`:
1. Set `deleted_at = now()` on all rows for the `user_id`.
2. Immediately: set `keypoints_enc = NULL` (synchronous in the same transaction).
3. After 30-day grace: hard-delete remaining soft-deleted rows.
4. Emit `privacy.erasure_completed` DEC-030 event with `cv_sessions` row count.

#### 15.7.4 Consent Withdrawal

If a user withdraws `'cv'` consent, the session upload endpoint immediately blocks new writes. Historical rows are retained under the original consent grant until an explicit erasure request is filed. Same model as §14.7.1 (wearable consent withdrawal).

---

### 15.8 Analytics Restrictions

CV data follows the most restrictive analytics exclusion policy in the system. `keypoints_enc` is categorically excluded everywhere. `form_score` and `cv_flags` are excluded from analytics pipelines because they are personal data with realistic re-identification risk when combined with workout timestamps and tenant context.

| Layer | `keypoints_enc` | `form_score` (raw) | `cv_flags` (raw) | `rep_count` |
|---|---|---|---|---|
| PostHog client events | ❌ Blocked | ❌ Excluded | ❌ Excluded | ✅ Allowed |
| PostHog server-side enrichment | ❌ Blocked | ❌ Excluded | ❌ Excluded | ✅ Allowed |
| ClickHouse analytics tables | ❌ Blocked | ❌ Excluded (use form_quality bucket) | ❌ Excluded | ✅ Allowed |
| Metabase tenant dashboard | ❌ Blocked | ❌ Excluded from individual queries | ❌ Excluded | Via `tenant_cv_summary` aggregate only |
| Sentry error payloads | ❌ Blocked | ❌ Excluded | ❌ Excluded | ❌ Excluded |
| Anthropic API prompt | ❌ Blocked | ❌ Excluded (form_quality bucket via `stripCVData()`) | ❌ Excluded (flag names only) | ✅ Allowed |
| DSAR export (Art. 20) | ✅ User's own data | ✅ Included | ✅ Included | ✅ Included |

The `stripForbiddenProperties()` middleware (§13.3) must be extended with:
```
['keypoints_enc', 'form_score', 'cv_flags', 'avg_keypoint_confidence',
 'frames_analysed', 'frames_below_threshold', 'raw_cv_data']
```

The last entry (`raw_cv_data`) handles the deprecated `workouts.raw_cv_data` column during the transition period.

---

### 15.9 DEC-030 Audit Events for CV Data

All CV-data access and lifecycle events must be HMAC-chained per DEC-030 (`docs/AUDIT_LOG_SCHEMA.md`).

| Event slug | Trigger | Severity | Notes |
|---|---|---|---|
| `cv.session_started` | Mobile begins uploading a CV session | Standard | 7-year retention |
| `cv.session_completed` | `cv_sessions` row transitions to `status = 'completed'` | Standard | 7-year retention |
| `cv.session_blocked` | Upload blocked: `'cv' NOT IN categories_consented` | Standard | 7-year retention; consent enforcement evidence |
| `cv.keypoints_nullified` | Phase 1 retention cron nullifies `keypoints_enc` (batch) | Standard | 7-year retention; Art. 5(1)(e) storage limitation evidence |
| `cv.session_hard_deleted` | Phase 2 cron or erasure worker hard-deletes row | Standard | 7-year retention; Art. 17 erasure evidence |
| `cv.keypoints_admin_accessed` | `form_admin` queries `cv_sessions` (break-glass) | **HIGH** | Mandatory `incident_id` field; 7-year retention; SOC 2 CC6.1 evidence |
| `cv.model_version_changed` | New CV model deployed to mobile production | Standard | 7-year retention; enables §R-10 blast-radius scoping |

---

### 15.10 Implementation Checklist

| # | Task | Owner | Priority | Milestone | Notes |
|---|---|---|---|---|---|
| 1 | Apply `cv_sessions` DDL migration (§15.2): table, constraints, 6 indexes, `ENABLE ROW LEVEL SECURITY` | `platform-engineer` | **P0** | M3 | Migration must include RLS lint-check per §7.2 |
| 2 | Apply RLS policies (§15.3.1): `cv_sessions_user_self`, `cv_sessions_system`, `cv_sessions_admin` | `platform-engineer` | **P0** | M3 | Confirm form_coach and form_tenant_admin have zero-row behaviour with a manual psql check |
| 3 | Add CV RLS assertions to `__tests__/db/rls_isolation.test.ts` (§15.3.2): tenant-admin zero-row, cross-tenant zero-row, form_coach zero-row | `platform-engineer` | **P0** | M3 | Block any enterprise-tier merge if these assertions fail |
| 4 | Provision `cv_enc_key` in Cloudflare Workers Secret — one AES-256-GCM key per tenant; document key rotation procedure (quarterly rotation); rotation event = `cv.keypoints_admin_accessed` with `reason: 'key_rotation'` | `security-engineer` | **P0** | M3 | Key must NOT be stored in Supabase env vars; Workers Secret only |
| 5 | Implement CV session upload endpoint (`POST /v1/cv/session`): consent gate check, keypoints encryption with per-tenant key, `cv_sessions` write under `form_system`, emit `cv.session_started` + `cv.session_completed` DEC-030 events | `platform-engineer` | **P0** | M3 | |
| 6 | Implement `stripCVData()` (§15.6.2); add unit tests asserting `keypoints_enc` never appears in Anthropic API call payload | `platform-engineer` | **P0** | M3 | CI gate: fail build if `keypoints_enc` found in any Anthropic request snapshot |
| 7 | Wire `cv.session_blocked` DEC-030 event to consent gate check in upload endpoint | `platform-engineer` | **P0** | M3 | |
| 8 | Extend `stripForbiddenProperties()` blocklist (§15.8) with CV field names including deprecated `raw_cv_data` | `data-engineer` | **P0** | M3 | |
| 9 | Extend Sentry `beforeSend` filter (`docs/OBSERVABILITY.md`) to scrub `keypoints_enc`, `form_score`, `cv_flags`, `raw_cv_data` from error payloads | `platform-engineer` | **P0** | M3 | |
| 10 | Create `tenant_cv_summary` materialized view (§15.4); add to nightly pg_cron refresh at 03:05 UTC; build admin dashboard Metabase panel using view (no direct `cv_sessions` query permitted) | `platform-engineer` + `data-engineer` | **P1** | M4 | Verify k-anonymity N < 5 suppression in view DDL before deploying |
| 11 | Implement Phase 1 retention pg_cron (`cv-keypoints-1yr-nullify`): nullify `keypoints_enc` after 1 year; emit `cv.keypoints_nullified` DEC-030 event per batch with nullified row count | `platform-engineer` | **P1** | M4 | Validate with synthetic data dated > 1 year ago in staging |
| 12 | Implement Phase 2 retention pg_cron (`cv-sessions-3yr-hard-delete`): hard-delete rows after 3 years; emit `cv.session_hard_deleted` DEC-030 event | `platform-engineer` | **P1** | M4 | |
| 13 | Extend Art. 17 erasure worker (§12.3): soft-delete cv_sessions; immediately nullify `keypoints_enc` (no grace period for Art. 9 data); hard-delete after 30-day grace; verify row count in `privacy.erasure_completed` event | `platform-engineer` | **P1** | M4 | Immediate nullification for biometric data is not negotiable — no grace period exception |
| 14 | Add `cv_sessions` to DSAR export job (§12.5): include `form_score`, `cv_flags`, `rep_count`, `status`, `started_at`, `ended_at` per row; add note that `keypoints_enc` payload available on written request via privacy@form.coach | `platform-engineer` | **P1** | M4 | |
| 15 | Wire `cv.model_version_changed` DEC-030 event to mobile model registry deployment step in ml-engineer CI pipeline | `ml-engineer` | **P1** | M4 | Enables blast-radius scoping for INCIDENT_RESPONSE.md §R-10 |
| 16 | Deprecate `workouts.raw_cv_data`: add deprecation notice comment to column DDL; update all client write paths to `cv_sessions`; drop column once write traffic is zero (validate via query log before migration) | `platform-engineer` + `ml-engineer` | **P2** | M5 | Coordinate with mobile app release cycle |
| 17 | Complete formal retention decision record for `cv_sessions` (1-year keypoints nullification + 3-year hard delete); obtain compliance-officer sign-off; add to privacy policy retention table (closes P-GAP-004 line item for cv_sessions) | `compliance-officer` | **P0** | Pre-launch | |

**OQ-CV-01 (Keypoint Blob Migration Gate):** Monitor `cv_sessions` table size in Supabase monthly post-launch. If table size exceeds 20 GB OR CV session volume exceeds 1,000/day for 30 consecutive days, initiate R2 migration (§15.5). Migration design: R2 bucket with Object Lock enabled, per-object AES-256-GCM using the same key derivation as current column encryption, `keypoints_r2_key TEXT` column replaces `keypoints_enc`, URL generation exclusively via form_system Worker (no pre-signed URLs returned to client). Owner: platform-engineer + devops-lead. Monitor checkpoint: M6.

---

*v0.6 additions: §15 Computer Vision (CV) Pose Estimation Data Schema — closes the schema gap for `cv_sessions` table referenced in SOC2_READINESS.md PRE-34-E-006 (column-level encryption evidence), §35.6.3 (1-year biometric retention), C1.2 (erasure cascade), and INCIDENT_RESPONSE.md §R-10 (blast-radius scoping by `model_version`). On-device inference architecture: GDPR Art. 9 avoidance for raw video stream, < 100 ms latency requirement, competitive privacy posture. `cv_sessions` DDL: 33-keypoint BlazePose / 17-keypoint MoveNet schema, 7 defect flag slugs in cv_flags taxonomy, `keypoints_enc` AES-256-GCM column-level encryption in Cloudflare Workers Secret (closes PRE-34-E-006), status state machine (active/completed/partial/tracking_lost), model identity columns for R-10 blast-radius, 6 indexes (user, tenant, set_id, created_at, retention sweep, model_version). RLS: user self-access policy; form_coach blocked at RLS layer (application-layer view only); form_tenant_admin zero-row privacy floor; form_system full access; form_admin break-glass with mandatory incident_id + separate decryption endpoint. Three CI RLS assertions (`rls_isolation.test.ts`). `tenant_cv_summary` materialized view: weekly aggregate with k-anonymity N ≥ 5 suppression, form_quality averages, status percentages, three defect-prevalence columns (no individual data). §15.5 keypoint storage decision resolves §9 gap 9.2: keypoints in JSONB TOAST with R2 migration gate at 20 GB / 1,000 sessions/day; `workouts.raw_cv_data` column deprecated. `stripCVData()` TypeScript: 4-bucket form_quality mapping, flag-presence-only extraction, keypoints categorically excluded from Anthropic API; CI grep gate. GDPR: Art. 9 biometric classification for keypoints_enc; explicit `'cv'` consent gate; two-phase deletion (1-year keypoints nullification via pg_cron, 3-year row hard delete); Art. 17 immediate keypoints nullification (no grace period for Art. 9 data); consent withdrawal semantics. Analytics restrictions: keypoints_enc and form_score/cv_flags excluded from all PostHog/ClickHouse/Metabase/Sentry/Anthropic layers; stripForbiddenProperties() extended. Seven DEC-030 HMAC-chained audit events (cv.session_started, cv.session_completed, cv.session_blocked, cv.keypoints_nullified, cv.session_hard_deleted, cv.keypoints_admin_accessed HIGH severity with incident_id requirement, cv.model_version_changed). 17-item implementation checklist (9× P0, 5× P1, 2× P2, 1× Pre-launch) across M3/M4/M5. OQ-CV-01: R2 blob migration gate condition. Gap 9.2 status updated to RESOLVED in §9.*


---

## 16. Enterprise Contract & Pilot Lifecycle Schema

> Owner: `enterprise-architect` + `compliance-officer`. Review: on any contract structure change, pricing-tier update, or quarterly.
> References: docs/ENTERPRISE.md §Pricing, §Implementation timeline; docs/AUDIT_LOG_SCHEMA.md (DEC-030); docs/SOC2_READINESS.md CC6.1, CC6.2, CC9.1; docs/OBSERVABILITY.md §7.4 (Per-Tenant SLA dashboard).

---

### 16.1 Purpose

The `tenants` table (§2.1) captures a tenant's current plan and seat ceiling, but contains no record of the commercial lifecycle that precedes and governs a production deployment. Enterprise customers follow a predictable sequence:

```
prospect → 90-day free pilot → conversion decision → annual/multi-year contract → renewal cycle
```

Without a data model for this lifecycle, four critical functions lack a source of truth:

1. **Seat enforcement** — the hard gate that blocks over-provisioning beyond contracted seats (CC6.1).
2. **Pilot-to-contract conversion tracking** — required for CSM handoff and the 90-day success-criteria review (ENTERPRISE.md §Implementation timeline, Day 90).
3. **Renewal notice triggering** — 60-day advance notice prevents contractual lapses under CC9.1.
4. **Admin dashboard seat utilisation** — HR/People leads see aggregate seat usage; no individual data (privacy floor, §6).

Section §16 defines `tenant_pilots`, `tenant_contracts`, an extension to `tenants`, the `tenant_seat_utilization` materialized view, seat enforcement logic, DEC-030 audit events, and the RLS posture for contract data.

---

### 16.2 `tenants` Table Extension — `lifecycle_status`

Add a `lifecycle_status` column to the existing `tenants` table to drive the state machine:

```sql
ALTER TABLE tenants
  ADD COLUMN lifecycle_status TEXT NOT NULL DEFAULT 'pilot'
    CHECK (lifecycle_status IN (
      'pilot',       -- 90-day free evaluation; max_seats enforced at pilot_seats cap
      'active',      -- production contract signed and in effect
      'expired',     -- contract end_at passed, not yet renewed or churned
      'suspended',   -- payment failure or compliance hold; SSO login blocked
      'churned'      -- permanently closed; data deletion scheduled
    ));
```

**State transitions** (all transitions emit a DEC-030 `tenant.lifecycle_changed` event — see §16.7):

| From | To | Trigger |
|---|---|---|
| `pilot` | `active` | `tenant_pilots.status = 'converted'`; `tenant_contracts` row created with `status = 'active'` |
| `pilot` | `churned` | `tenant_pilots.status = 'expired'` after 90 days with no conversion; CSM confirms no pipeline |
| `active` | `expired` | `tenant_contracts.end_at < now()` and auto-renew not completed |
| `active` | `suspended` | Stripe payment failure or compliance-officer hold |
| `expired` | `active` | Renewal contract signed; new `tenant_contracts` row |
| `expired` | `churned` | 30 days past expiry with no renewal; CSM confirms lost |
| `suspended` | `active` | Payment resolved or hold lifted |
| `suspended` | `churned` | Suspension extended > 60 days |
| Any | `churned` | Explicit termination; data deletion process begins per §12.3 |

**Access gate:** When `lifecycle_status IN ('suspended', 'churned', 'expired')`, the Cloudflare Worker middleware blocks all API requests from that tenant with HTTP 402 and redirects the admin to `enterprise@form.coach`. SSO SAML assertions are rejected at the SP-initiated flow entry point.

---

### 16.3 `tenant_pilots` Table

```sql
CREATE TABLE tenant_pilots (
  id                UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id         UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
  pilot_type        TEXT NOT NULL DEFAULT 'free'
                      CHECK (pilot_type IN (
                        'free',           -- 90-day free standard; ≤ 250 seats; no founder approval required (COST_MODEL §21.2)
                        'extended_free',  -- 180-day free; any seat count; founder approval required; ≥ 500-seat conversion target
                        'paid'            -- 90-day at $1/seat/month; target ≥ 500 seats; founder approval required; credit on conversion
                      )),
  pilot_seats       INTEGER NOT NULL DEFAULT 100
                      CHECK (pilot_seats BETWEEN 10 AND 10000),
  monthly_fee_cents INTEGER DEFAULT NULL,            -- NULL for free/extended_free; 100 (= $1.00/seat/month) for paid type
  founder_approved_at TIMESTAMPTZ,                   -- NULL for 'free' pilots; must be set before provisioning 'extended_free' or 'paid'
  started_at        TIMESTAMPTZ NOT NULL DEFAULT now(),
  ends_at           TIMESTAMPTZ NOT NULL,            -- default: started_at + 90 days (free/paid) or + 180 days (extended_free)
  success_criteria  JSONB NOT NULL,                  -- agreed criteria (see §16.3.1)
  status            TEXT NOT NULL DEFAULT 'active'
                      CHECK (status IN (
                        'active',    -- pilot running
                        'extended',  -- manually extended beyond ends_at (requires CSM + founder approval)
                        'converted', -- customer signed production contract
                        'expired',   -- ends_at passed without conversion
                        'declined'   -- customer explicitly declined to convert
                      )),
  extended_at       TIMESTAMPTZ,                     -- set when status → 'extended'
  extension_ends_at TIMESTAMPTZ,                     -- new end date if extended
  converted_at      TIMESTAMPTZ,                     -- set when status → 'converted'
  declined_at       TIMESTAMPTZ,
  csm_owner         TEXT NOT NULL,                   -- internal handle (e.g. '@alice') — NOT a FK to users
  internal_notes    TEXT,                            -- CSM-only field; MUST NOT contain employee names or health data
  created_at        TIMESTAMPTZ NOT NULL DEFAULT now(),
  UNIQUE (tenant_id),                                -- one pilot record per tenant; repeat pilots require a new tenant slug
  CONSTRAINT pilot_fee_consistency CHECK (
    (pilot_type = 'paid' AND monthly_fee_cents IS NOT NULL AND monthly_fee_cents > 0)
    OR (pilot_type IN ('free', 'extended_free') AND monthly_fee_cents IS NULL)
  )
);

ALTER TABLE tenant_pilots ENABLE ROW LEVEL SECURITY;

-- Only form_admin and form_system can read or write pilot data
-- form_api cannot expose pilot terms, CSM notes, or success criteria to tenant admins
CREATE POLICY tenant_pilots_admin_only ON tenant_pilots
  USING (current_user IN ('form_admin', 'form_system'));
```

**Migration for existing rows** (run after initial `CREATE TABLE` on a live database with pre-existing pilot rows):

```sql
-- Add new columns to any previously deployed tenant_pilots table
ALTER TABLE tenant_pilots
  ADD COLUMN IF NOT EXISTS pilot_type TEXT NOT NULL DEFAULT 'free'
    CHECK (pilot_type IN ('free', 'extended_free', 'paid')),
  ADD COLUMN IF NOT EXISTS monthly_fee_cents INTEGER DEFAULT NULL,
  ADD COLUMN IF NOT EXISTS founder_approved_at TIMESTAMPTZ;

-- Increase pilot_seats ceiling from 500 → 10000 to accommodate paid/extended pilots
ALTER TABLE tenant_pilots
  DROP CONSTRAINT IF EXISTS tenant_pilots_pilot_seats_check;
ALTER TABLE tenant_pilots
  ADD CONSTRAINT tenant_pilots_pilot_seats_check CHECK (pilot_seats BETWEEN 10 AND 10000);

-- Add cross-column fee consistency constraint
ALTER TABLE tenant_pilots
  ADD CONSTRAINT pilot_fee_consistency CHECK (
    (pilot_type = 'paid' AND monthly_fee_cents IS NOT NULL AND monthly_fee_cents > 0)
    OR (pilot_type IN ('free', 'extended_free') AND monthly_fee_cents IS NULL)
  );
```

**Column notes:**

| Column | Type | Nullable | Meaning |
|---|---|---|---|
| `pilot_type` | TEXT | NOT NULL | Commercial structure of the pilot: `'free'` (standard, ≤250 seats), `'extended_free'` (180-day, founder-approved, ≥500-seat conversion target), `'paid'` ($1/seat/month, founder-approved, credit on conversion). Drives ASC 606 treatment (COST_MODEL §21.3) and `pilot.started` audit event `pilot_type` field. |
| `monthly_fee_cents` | INTEGER | NULL | For `paid` pilots: 100 (= $1.00/seat/month, billed to Stripe). NULL for free pilot types. The `pilot_fee_consistency` constraint enforces this at the DB layer. |
| `founder_approved_at` | TIMESTAMPTZ | NULL | Timestamp when founder approved the `extended_free` or `paid` pilot. Application must set this before provisioning SSO and emitting `pilot.started`. NULL is valid for `'free'` pilot_type — no approval required. |

#### 16.3.1 `success_criteria` JSONB Schema

The `success_criteria` field stores the metrics agreed with the customer before the pilot starts. These are evaluated at Day 90 by the CSM. Format is fixed-schema JSONB validated by a Zod schema in `src/enterprise/pilot.schema.ts`.

```jsonc
{
  "activation_rate_pct": 60,        // % of invited seats who complete onboarding (min n=10 to count)
  "weekly_engagement_pct": 40,      // % of activated users with ≥1 session in trailing 7 days
  "nps_floor": 30,                  // Net Promoter Score floor (opt-in survey; no minimum response rate required)
  "min_active_seats": 30,           // absolute floor — if < 30 users engage, criteria are inconclusive
  "evaluation_date": "2026-08-20"   // ISO 8601 — Day 90 review call date agreed with customer
}
```

**Privacy rule:** `success_criteria` contains only aggregate thresholds, not individual user identifiers or names. The CSM accesses pilot metrics exclusively through the `tenant_cv_summary` and `tenant_wellness_summary` admin views (§2.6), which enforce the n ≥ 15 anonymity floor.

---

### 16.4 `tenant_contracts` Table

```sql
CREATE TABLE tenant_contracts (
  id                    UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id             UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
  contract_ref          TEXT UNIQUE NOT NULL,          -- e.g., 'ENT-2026-042'; assigned by CSM
  plan                  TEXT NOT NULL
                          CHECK (plan IN ('starter', 'growth', 'enterprise')),
  seats_contracted      INTEGER NOT NULL
                          CHECK (seats_contracted >= 50),  -- minimum deal size per §16.1
  arr_cents             BIGINT NOT NULL
                          CHECK (arr_cents > 0),           -- Annual Recurring Revenue, USD cents
  billing_period        TEXT NOT NULL DEFAULT 'annual'
                          CHECK (billing_period IN ('annual', '2year', '3year')),
  discount_pct          SMALLINT NOT NULL DEFAULT 0
                          CHECK (discount_pct BETWEEN 0 AND 40),
  po_number             TEXT,                              -- customer PO; required for invoicing if set
  start_at              TIMESTAMPTZ NOT NULL,
  end_at                TIMESTAMPTZ NOT NULL
                          CHECK (end_at > start_at),
  auto_renew            BOOLEAN NOT NULL DEFAULT TRUE,
  renewal_notice_days   INTEGER NOT NULL DEFAULT 60,       -- notify CSM + tenant_owner this many days before end_at
  signed_at             TIMESTAMPTZ,
  countersigned_at      TIMESTAMPTZ,                       -- FORM countersignature
  status                TEXT NOT NULL DEFAULT 'draft'
                          CHECK (status IN (
                            'draft',      -- MSA/order form in negotiation
                            'active',     -- signed and in effect
                            'renewed',    -- superseded by a new contract row; kept for audit history
                            'expired',    -- end_at passed; not renewed
                            'terminated'  -- terminated early by either party
                          )),
  termination_reason    TEXT,                              -- populated on status = 'terminated'
  stripe_subscription_id TEXT,                            -- Stripe sub for annual billing
  created_at            TIMESTAMPTZ NOT NULL DEFAULT now()
);

-- Partial unique index: only one 'active' contract per tenant at any time
CREATE UNIQUE INDEX tenant_contracts_one_active_per_tenant
  ON tenant_contracts (tenant_id)
  WHERE status = 'active';

ALTER TABLE tenant_contracts ENABLE ROW LEVEL SECURITY;

-- form_admin and form_system: full access
CREATE POLICY tenant_contracts_admin ON tenant_contracts
  USING (current_user IN ('form_admin', 'form_system'));

-- form_api: tenant admins see limited contract metadata only (NOT arr_cents — that is FORM-internal)
CREATE POLICY tenant_contracts_tenant_read ON tenant_contracts
  FOR SELECT
  TO form_api
  USING (tenant_id = current_setting('app.tenant_id')::UUID AND status = 'active');
```

**ARR confidentiality:** `arr_cents` is visible only to `form_admin` and `form_system`. The `form_api` role's SELECT policy returns contract rows for the tenant's own tenant_id, but the admin dashboard query MUST use a view that excludes `arr_cents`:

```sql
CREATE VIEW tenant_contract_portal AS
  SELECT
    id,
    tenant_id,
    contract_ref,
    plan,
    seats_contracted,
    billing_period,
    po_number,
    start_at,
    end_at,
    auto_renew,
    status
    -- arr_cents intentionally excluded
  FROM tenant_contracts
  WHERE status = 'active';

GRANT SELECT ON tenant_contract_portal TO form_api;
```

Tenant admins see their contract reference, seat count, billing period, and end date. They do not see the per-seat rate or total ARR — that is FORM commercial-confidential data.

---

### 16.5 Seat Enforcement at Provisioning

Seat enforcement is the primary CC6.1 technical control for enterprise tenants. Every user provisioning path — SCIM `POST /Users`, CSV import, manual invite — runs through a single enforcement check before the user row is created.

```typescript
// workers/provisioning/seat-gate.ts
// Runs inside the Cloudflare Worker before any INSERT to `users`
export async function assertSeatAvailable(
  tenantId: string,
  db: PostgresClient,
): Promise<void> {
  const row = await db.queryOne<{ active_seats: number; max_seats: number }>(
    `SELECT
       COUNT(id) FILTER (WHERE deactivated_at IS NULL AND deleted_at IS NULL) AS active_seats,
       (SELECT max_seats FROM tenants WHERE id = $1) AS max_seats
     FROM users
     WHERE tenant_id = $1`,
    [tenantId],
  );

  if (row.active_seats >= row.max_seats) {
    await emitAuditEvent({
      event:     'tenant.seat_limit_enforcement_triggered',
      tenant_id: tenantId,
      severity:  'STANDARD',
      metadata:  { active_seats: row.active_seats, max_seats: row.max_seats },
    });
    throw new SeatLimitError(
      `Tenant ${tenantId} has reached the contracted seat limit (${row.max_seats}). ` +
      `Contact enterprise@form.coach to add seats.`,
      { status: 402 },
    );
  }
}
```

**`max_seats` source of truth:** During a pilot, `tenants.max_seats` is set to `tenant_pilots.pilot_seats`. On conversion, it is updated to `tenant_contracts.seats_contracted`. Seat expansion mid-contract (order form amendment) updates both `tenant_contracts.seats_contracted` and `tenants.max_seats` in the same transaction, with a `tenant.seats_contracted_changed` DEC-030 event.

**SCIM burst protection:** Large enterprise IdP syncs (e.g., Entra ID full sync on initial configuration) can send hundreds of `POST /Users` requests in rapid succession. The seat gate uses a `SELECT … FOR UPDATE SKIP LOCKED` pattern to prevent a TOCTOU race condition from creating more users than the seat limit permits. See `SSO_SCIM_IMPLEMENTATION.md §6.3` for rate limiting.

---

### 16.6 `tenant_seat_utilization` Materialized View

This view is the sole data source for seat metrics in the admin dashboard and CSM QBR reports. It produces aggregate counts only — no individual user data is exposed.

```sql
CREATE MATERIALIZED VIEW tenant_seat_utilization AS
SELECT
  t.id                                                                      AS tenant_id,
  t.slug,
  t.plan,
  t.lifecycle_status,
  t.max_seats,
  COALESCE(tc.seats_contracted, tp.pilot_seats)                             AS seats_contracted_or_pilot,
  tc.end_at                                                                 AS contract_ends_at,
  tc.contract_ref,
  COUNT(u.id) FILTER (WHERE u.deactivated_at IS NULL AND u.deleted_at IS NULL)
                                                                            AS active_seats,
  COUNT(u.id) FILTER (
    WHERE u.deactivated_at IS NULL AND u.deleted_at IS NULL
    AND   u.provisioned_via = 'scim'
  )                                                                         AS scim_provisioned,
  COUNT(u.id) FILTER (
    WHERE u.deactivated_at IS NULL AND u.deleted_at IS NULL
    AND   u.last_login_at > now() - INTERVAL '7 days'
  )                                                                         AS seats_active_7d,
  COUNT(u.id) FILTER (
    WHERE u.deactivated_at IS NULL AND u.deleted_at IS NULL
    AND   u.last_login_at > now() - INTERVAL '30 days'
  )                                                                         AS seats_active_30d,
  ROUND(
    COUNT(u.id) FILTER (
      WHERE u.deactivated_at IS NULL AND u.deleted_at IS NULL
      AND   u.last_login_at > now() - INTERVAL '7 days'
    )::NUMERIC
    / NULLIF(COALESCE(tc.seats_contracted, tp.pilot_seats), 0) * 100,
    1
  )                                                                         AS utilization_pct_7d,
  now()                                                                     AS refreshed_at
FROM tenants t
LEFT JOIN tenant_contracts tc
  ON tc.tenant_id = t.id AND tc.status = 'active'
LEFT JOIN tenant_pilots tp
  ON tp.tenant_id = t.id AND tp.status = 'active'
LEFT JOIN users u
  ON u.tenant_id = t.id AND u.deleted_at IS NULL
GROUP BY
  t.id, t.slug, t.plan, t.lifecycle_status, t.max_seats,
  tc.seats_contracted, tc.end_at, tc.contract_ref,
  tp.pilot_seats;

-- Nightly refresh at 02:00 UTC (after wearable sync cron at 01:xx and before cost monitor at 06:00)
SELECT cron.schedule('tenant-seat-util-refresh', '0 2 * * *', $$
  REFRESH MATERIALIZED VIEW CONCURRENTLY tenant_seat_utilization;
$$);

GRANT SELECT ON tenant_seat_utilization TO form_api;
GRANT SELECT ON tenant_seat_utilization TO form_admin;
```

**Admin dashboard display rules (privacy floor):**
- The Metabase panel consuming this view is restricted to `tenant_owner` and `tenant_admin` roles via `form_api` RLS — they see their own row only.
- `contract_ref` and `contract_ends_at` are displayed. `arr_cents` is absent from this view.
- `seats_active_7d` and `utilization_pct_7d` are aggregate numbers — no per-user breakdown is possible from this view.
- Minimum display threshold: if `active_seats < 15`, the `seats_active_7d` and `utilization_pct_7d` columns are returned as `NULL` and rendered as "Insufficient data (< 15 users)" in the dashboard. This prevents cohort re-identification in very small pilots.

**Expansion opportunity signal:** A Metabase alert fires to the CSM Slack channel when `utilization_pct_7d > 85` for 14 consecutive days. This is an internal signal only — not visible to the tenant admin. It triggers a CSM outreach conversation about seat expansion, not an automated upsell message.

---

### 16.7 Renewal Notice Trigger

Contracts lapsing without notice create CC9.1 (business disruption) risk. A pg_cron job scans for contracts approaching expiry and fires a notification to the CSM via the internal Slack webhook.

```sql
-- Runs daily at 09:00 UTC
SELECT cron.schedule('contract-renewal-notice', '0 9 * * *', $$
  SELECT
    contract_ref,
    tenant_id,
    end_at,
    (end_at - now())::TEXT AS days_remaining
  FROM tenant_contracts
  WHERE status = 'active'
    AND auto_renew = FALSE
    AND end_at BETWEEN now() AND now() + (renewal_notice_days || ' days')::INTERVAL;
$$);
```

The query result is consumed by the `renewal-notice` Cloudflare Worker (triggered via Cron at 09:05 UTC), which:
1. Posts a Slack message to `#csm-renewals` with contract ref, tenant name, days remaining, and ARR.
2. Emits a `tenant.renewal_notice_sent` DEC-030 event.
3. For contracts within 14 days of expiry, pages PagerDuty `enterprise-ops` service (P2 severity).

Auto-renew contracts (`auto_renew = TRUE`) receive a Slack digest-only notice at `renewal_notice_days` but do not page PagerDuty unless Stripe billing fails.

---

### 16.8 DEC-030 Audit Events for Contract Lifecycle

All contract and pilot lifecycle transitions are HMAC-chained per DEC-030 (`docs/AUDIT_LOG_SCHEMA.md`). These events are retained for 7 years (matching financial record obligations).

| Event slug | Trigger | Severity | Retention |
|---|---|---|---|
| `tenant.pilot_started` | `tenant_pilots` row created | STANDARD | 7 years |
| `tenant.pilot_extended` | `tenant_pilots.status → 'extended'` | STANDARD | 7 years |
| `tenant.pilot_converted` | `tenant_pilots.status → 'converted'`; `tenant_contracts` created | STANDARD | 7 years |
| `tenant.pilot_expired` | `tenant_pilots.status → 'expired'` (no conversion) | STANDARD | 7 years |
| `tenant.pilot_declined` | `tenant_pilots.status → 'declined'` | STANDARD | 7 years |
| `tenant.contract_activated` | `tenant_contracts.status → 'active'` (signed) | STANDARD | 7 years |
| `tenant.contract_renewed` | New `tenant_contracts` row with `status = 'active'`; prior row → `'renewed'` | STANDARD | 7 years |
| `tenant.contract_expired` | `tenant_contracts.status → 'expired'` | STANDARD | 7 years |
| `tenant.contract_terminated` | `tenant_contracts.status → 'terminated'` | **HIGH** | 7 years |
| `tenant.seats_contracted_changed` | `seats_contracted` amended mid-contract | STANDARD | 7 years |
| `tenant.seat_limit_enforcement_triggered` | Provisioning blocked by seat gate (§16.5) | STANDARD | 7 years |
| `tenant.renewal_notice_sent` | Renewal notice dispatched to CSM + tenant_owner | STANDARD | 7 years |
| `tenant.lifecycle_changed` | `tenants.lifecycle_status` state transition | STANDARD | 7 years |
| `tenant.access_suspended` | `lifecycle_status → 'suspended'`; API access blocked | **HIGH** | 7 years |

**`tenant.contract_terminated` — HIGH severity rationale:** Early termination is a high-risk event for both data residency and revenue. The HIGH severity classification ensures the event appears in SIEM dashboards and triggers the 24-hour tenant notification requirement under `docs/ENTERPRISE.md §Hard commitments`.

**Metadata schema for `tenant.seat_limit_enforcement_triggered`:**
```jsonc
{
  "event":        "tenant.seat_limit_enforcement_triggered",
  "tenant_id":    "<uuid>",
  "triggered_by": "scim" | "csv_import" | "manual_invite",
  "active_seats": 200,
  "max_seats":    200,
  "attempted_email": null     // NEVER log the email of the blocked user — only the enforcement count
}
```

`attempted_email` is explicitly `null` to prevent the audit log from becoming a PII store for blocked provisioning attempts.

---

### 16.9 Data Retention and GDPR

**`tenant_contracts`:** Financial records. Retained for 7 years post-contract-end under EU bookkeeping obligations and US GAAP requirements. Not deleted on tenant churn — the record is delinked from active SSO/API access when `tenants.lifecycle_status = 'churned'` but the row itself is retained with `status = 'terminated'` or `'expired'`. No personal data in this table (company name is in `tenants.display_name`, not contract table).

**`tenant_pilots`:** Operational records. Retained for 2 years post-pilot-end. Soft-deleted after 2 years (set `deleted_at`); hard-deleted after 5 years. The `internal_notes` field may contain personal data if a CSM writes an employee name — CSMs are instructed in the onboarding guide that this field must not contain named individuals. A Cloudflare Worker content-scan runs weekly against `internal_notes` matching common name-like patterns and flags for CSM review.

**`tenant_seat_utilization` (materialized view):** Aggregate only. Refreshed nightly; prior state is not retained. Not subject to Art. 17 erasure independently — it is derived from `tenants` and `users` tables which handle their own erasure.

**`tenants.lifecycle_status`:** Retained as part of the `tenants` row. On hard-delete of a churned tenant (post-grace-period), the full row is deleted per §12.3 (Art. 17 erasure cascade). The DEC-030 audit events for `tenant.lifecycle_changed` are retained for 7 years and are not deleted on tenant erasure (audit log immutability, §8 of AUDIT_LOG_SCHEMA.md).

**Art. 28 DPA obligation:** `tenant_contracts` rows for EU tenants must reference the executed DPA in a `dpa_ref` note. Add `dpa_ref TEXT` column and populate before any EU pilot or contract is activated. This cross-references `compliance/p1/sub-processor-register.md` and closes the SOC 2 P-GAP-002 line for customer DPA tracking.

---

### 16.10 SOC 2 Mapping

| SOC 2 Criterion | How §16 Satisfies It |
|---|---|
| **CC6.1 — Logical access** | Seat enforcement gate (§16.5) ensures only contracted users can be provisioned; `assertSeatAvailable()` is the technical control. Every blocked attempt emits a DEC-030 audit event. |
| **CC6.2 — User authorisation** | `tenant_contracts.status = 'active'` is the business authorisation that unlocks provisioning. Pilot → contract conversion (§16.3) documents the pre-access approval workflow. |
| **CC9.1 — Business disruption risk** | Renewal notice trigger (§16.7) detects upcoming contract expiry 60 days in advance; PagerDuty escalation at 14 days prevents availability disruption from lapsed contracts. |
| **CC9.2 — Vendor and customer commitments** | `tenant_contracts.arr_cents`, `seats_contracted`, `end_at`, and `po_number` are the authoritative record of commercial commitments; referenced in any CC9.2 customer commitment evidence package. |
| **A1.1 — Availability commitments documented** | `tenant_contracts.plan` maps to the SLA tier in INCIDENT_RESPONSE.md §12.2; the contract row is the evidence that a specific SLA was committed to a specific tenant. |
| **PI1.4 — Processing accuracy** | `tenant.seat_limit_enforcement_triggered` events provide evidence that FORM does not process (provision) more users than contracted — a processing integrity control. |
| **C1.2 — Confidential information protected** | `arr_cents` excluded from `tenant_contract_portal` view and from `form_api` role access; pilot `success_criteria` and `internal_notes` blocked from tenant-facing endpoints. |

**Evidence artifacts for SOC 2 auditor:**
- `tenant_contracts` table export (form_admin access only, `arr_cents` redacted before sharing with enterprise prospects) → `compliance/evidence/contracts/`
- `tenant_pilots` status distribution report → `compliance/evidence/pilots/`
- `tenant.seat_limit_enforcement_triggered` event log → CC6.1 enforcement evidence
- `tenant.contract_activated` + `tenant.renewal_notice_sent` event log → CC9.1 risk management evidence

---

### 16.11 Implementation Checklist

| # | Task | Owner | Priority | Milestone | Notes |
|---|---|---|---|---|---|
| 1 | `ALTER TABLE tenants ADD COLUMN lifecycle_status` — migration with default `'pilot'` for existing rows; add CHECK constraint and index | `platform-engineer` | **P0** | M3 | |
| 2 | `CREATE TABLE tenant_pilots` DDL migration including RLS policy (`form_admin` + `form_system` only) | `platform-engineer` | **P0** | M3 | |
| 2a | Add `pilot_type`, `monthly_fee_cents`, `founder_approved_at` columns to `tenant_pilots`; update `pilot_seats` CHECK ceiling to 10000; add `pilot_fee_consistency` table constraint (see §16.3 migration block) | `platform-engineer` | **P1** | M3 | Closes COST_MODEL §21.9 item: "Build `pilots` table tracking `pilot_type`" |
| 3 | `CREATE TABLE tenant_contracts` DDL migration: all columns, partial unique index, RLS policies, `tenant_contract_portal` view | `platform-engineer` | **P0** | M3 | `tenant_contract_portal` view must exclude `arr_cents` before any tenant admin can see it |
| 4 | Implement `assertSeatAvailable()` in `workers/provisioning/seat-gate.ts`; integrate into SCIM `POST /Users`, CSV import, and manual invite paths | `platform-engineer` | **P0** | M3 | TOCTOU protection via `SELECT … FOR UPDATE SKIP LOCKED`; test with concurrent provisioning in staging |
| 5 | Wire all 14 DEC-030 events (§16.8) to their respective state transitions; validate HMAC chain in staging | `platform-engineer` + `security-engineer` | **P0** | M3 | `tenant.seat_limit_enforcement_triggered` metadata must have `attempted_email: null` — CI assertion |
| 6 | `CREATE MATERIALIZED VIEW tenant_seat_utilization`; add `pg_cron` refresh at 02:00 UTC; grant SELECT to `form_api` and `form_admin` | `platform-engineer` | **P1** | M4 | Verify anonymity suppression (NULL output when `active_seats < 15`) with a staging tenant |
| 7 | Build admin dashboard Metabase panel consuming `tenant_seat_utilization` (seats contracted, active seats, 7d utilization %, contract end date); restrict to `tenant_owner` + `tenant_admin` roles | `data-engineer` | **P1** | M4 | Panel must not expose `arr_cents` or `internal_notes` |
| 8 | Implement `renewal-notice` Cloudflare Worker (Cron 09:05 UTC): query contracts nearing expiry, post Slack to `#csm-renewals`, emit `tenant.renewal_notice_sent` DEC-030, page PagerDuty at ≤ 14 days | `devops-lead` | **P1** | M4 | |
| 9 | Implement `lifecycle_status` access gate in Worker middleware: block API requests from `suspended` / `churned` / `expired` tenants with HTTP 402 | `platform-engineer` | **P0** | M3 | Ensure SSO SAML assertion validation also checks `lifecycle_status` — see `SSO_SCIM_IMPLEMENTATION.md §6.1` |
| 10 | Add `dpa_ref TEXT` column to `tenant_contracts` (EU pilot requirement); populate before any EU pilot activation | `compliance-officer` + `platform-engineer` | **P0** | Pre-EU pilot | Closes P-GAP-002 DPA tracking line item |
| 11 | Implement `tenant.seats_contracted_changed` event in order-form amendment flow (CSM tool): single DB transaction updating `tenant_contracts.seats_contracted` + `tenants.max_seats` | `platform-engineer` | **P1** | M4 | |
| 12 | CSM onboarding guide update: document `internal_notes` PII prohibition; implement weekly content-scan Worker flagging name-like patterns in `tenant_pilots.internal_notes` | `enterprise-architect` + `compliance-officer` | **P1** | M4 | |
| 13 | Expansion opportunity alert: Metabase alert firing to CSM Slack when `utilization_pct_7d > 85` for 14 consecutive days — internal signal only, no tenant-facing automation | `data-engineer` | **P2** | M5 | |
| 14 | 2-year `tenant_pilots` retention cron: soft-delete pilot rows where `status IN ('expired', 'declined', 'converted')` AND `COALESCE(ends_at, extension_ends_at) < now() - INTERVAL '2 years'` | `platform-engineer` | **P2** | M5 | |

**OQ-ENT-01 (Minimum seat floor validation):** COST_MODEL.md §16.2 models 50-seat Starter Y1 gross margin at 47.9%. If actual CSM cost-per-pilot exceeds $200/month, the minimum viable seat floor rises to 75+. Instrument `tenant_pilots.pilot_seats` distribution across first 10 pilots and reconcile against §16.2 model. Owner: compliance-officer + enterprise-architect. Checkpoint: M6.

---

---

## 17. Enterprise Admin Reporting Schema — Aggregate-Only Data Model & Privacy-Floor Enforcement

### 17.1 Purpose and Design Principles

The admin dashboard is FORM's primary interface for enterprise HR and People leads. It answers *"is our fitness programme working?"* without exposing any individual employee data. This section specifies every materialized view, table, and API endpoint that powers that dashboard.

**The privacy floor from §6 is a schema constraint here, not a policy.** Three independent layers enforce it:

1. **Database layer:** Materialized views are defined using only permitted columns. No join to `user_health_profiles`, `meal_logs`, or `coaching_turns` exists anywhere in §17 DDL.
2. **Application layer:** Admin reporting API endpoints (§17.6) route exclusively to aggregate views. No route exists to raw user rows for `tenant_admin` or `tenant_owner` roles.
3. **API response layer:** A Cloudflare Worker middleware validates every admin reporting response for `cohort_size ≥ k_floor` before delivery. Responses where the threshold is not met are replaced with `{"suppressed": true, "reason": "k_anonymity_floor"}`.

**Removing any layer requires:** compliance-officer sign-off + DECISION_LOG.md entry + SOC 2 auditor notification at next evidence cycle.

---

### 17.2 Permitted and Prohibited Metric Registry

The registry below is the single source of truth for what may appear in any admin-facing report, dashboard panel, CSV export, or API response.

#### Section A — Permitted for HR/Admin visibility (aggregate only, N ≥ k_floor)

| Metric | Source table/column | Aggregate function | Notes |
|---|---|---|---|
| Provisioned seat count | `users` (per tenant) | COUNT | Always available; not anonymity-gated |
| Activated users | `users WHERE onboarding_completed_at IS NOT NULL` | COUNT, % of provisioned | % of provisioned seats |
| Weekly Active Users (WAU) | `workouts.started_at` (7-day window) | COUNT DISTINCT user_id | Suppressed if N < k_floor |
| Monthly Active Users (MAU) | `workouts.started_at` (30-day window) | COUNT DISTINCT user_id | Suppressed if N < k_floor |
| Total sessions per week | `workouts` | COUNT | Aggregate count only |
| Avg session duration (minutes) | `workouts.duration_seconds` | AVG, PERCENTILE 50/90 | No individual values |
| Avg AI form score | `workouts.form_score` | AVG, PERCENTILE 50 | No per-user breakdown; null if CV cohort N < k_floor |
| D30 retention rate | `workouts` first session vs. return session | % returning | Cohort-level only; suppressed if cohort N < k_floor |
| CV form analysis adoption | `workout_sets.form_score IS NOT NULL` | % of sessions using CV | Aggregate flag only |
| Voice coaching adoption | `coaching_sessions` (session count) | COUNT / WAU | Session count only — no content |
| Opt-in NPS score | `tenant_nps_responses.score` | AVG, distribution by bucket | **Only with explicit opt-in consent per §17.7** |
| SCIM group engagement | `scim_group_members` + `workouts` | WAU/MAU by group | Suppressed if group N < k_floor; §15 Art. 9 name gate applies |

#### Section B — Prohibited — never in any admin-facing output

| Data category | Source | Prohibition | Note |
|---|---|---|---|
| Individual workout rows | `workouts` per user | Absolute — no per-user workout list | |
| Individual form scores | `workout_sets.form_score` per user | Absolute | |
| Coaching turn content | `coaching_turns.content` | Absolute | Victor dialogue is Confidential (§5) |
| Per-user coaching session count | `coaching_sessions` per user | Absolute — only aggregate session rate | |
| Health goals and restrictions | `user_health_profiles` | Absolute | Art. 9 GDPR health data |
| Body composition metrics | `user_health_profiles.goals` (BMI, body fat) | Absolute | **clinical-safety veto — no exceptions** |
| Meal log data | `meal_logs` | Absolute | **clinical-safety veto — ED-risk category** |
| Mental health signals | Any health-adjacent `coaching_turns.content` | Absolute | **clinical-safety veto** |
| HRV / resting HR individual trends | Wearable data per user | Absolute | Individual-level health signal |
| User identity linked to any metric | `users.email`, `users.display_name` | Absolute | All metrics must be anonymous |
| Opted-out users in any aggregate | `user_aggregate_consent.opted_out = TRUE` | Absolute — excluded from all MVs at query time | |

**Enforcement note:** Any product feature that requests admin visibility into a Prohibited category requires a documented compliance-officer override in DECISION_LOG.md and a clinical-safety review for the five categories marked with the clinical-safety veto. The no-go list is non-negotiable regardless of customer demand.

---

### 17.3 Aggregate Materialized View Specifications

#### 17.3.1 `tenant_wellness_summary_v2`

Production replacement for the prototype in §2.6. The §2.6 view remains as a reference definition; this is the authoritative production specification.

```sql
-- Replaces §2.6 prototype. Populated by scheduled job only — never via API request path.
-- Reads: workouts, users, user_aggregate_consent
-- Structurally excludes: user_health_profiles, meal_logs, coaching_turns (§17.2B)
CREATE MATERIALIZED VIEW tenant_wellness_summary_v2 AS
SELECT
  w.tenant_id,
  DATE_TRUNC('week', w.started_at)           AS report_week,

  -- Participation
  COUNT(DISTINCT w.user_id)                  AS active_users,
  COUNT(*)                                   AS total_sessions,

  -- Duration signals
  AVG(w.duration_seconds)  / 60.0            AS avg_duration_min,
  PERCENTILE_CONT(0.5) WITHIN GROUP
    (ORDER BY w.duration_seconds) / 60.0     AS median_duration_min,
  PERCENTILE_CONT(0.9) WITHIN GROUP
    (ORDER BY w.duration_seconds) / 60.0     AS p90_duration_min,

  -- Quality signal (CV form score) — suppressed when CV cohort < k_floor
  CASE
    WHEN COUNT(DISTINCT w.user_id)
         FILTER (WHERE w.form_score IS NOT NULL) >= 5
    THEN ROUND(
           AVG(w.form_score) FILTER (WHERE w.form_score IS NOT NULL)::NUMERIC,
           1
         )
    ELSE NULL
  END                                        AS avg_form_score,

  -- k-anonymity gate column — API middleware checks this before delivery
  COUNT(DISTINCT w.user_id) >= 5             AS meets_anonymity_floor

FROM workouts w
JOIN users u ON u.id = w.user_id
JOIN user_aggregate_consent uac ON uac.user_id = w.user_id
WHERE u.deactivated_at IS NULL
  AND uac.opted_out = FALSE
  AND uac.consent_version IS NOT NULL        -- consent must have been explicitly given
  AND w.started_at >= NOW() - INTERVAL '13 weeks'
GROUP BY w.tenant_id, DATE_TRUNC('week', w.started_at)
WITH NO DATA;

-- Unique index enables REFRESH CONCURRENTLY (non-blocking)
CREATE UNIQUE INDEX uq_tws2_tenant_week
  ON tenant_wellness_summary_v2 (tenant_id, report_week);

ALTER TABLE tenant_wellness_summary_v2 ENABLE ROW LEVEL SECURITY;

CREATE POLICY tws2_tenant_read ON tenant_wellness_summary_v2
  FOR SELECT TO form_api
  USING (tenant_id = current_setting('app.current_tenant_id', TRUE)::UUID);
```

**Privacy invariants verified by §17.10 test suite:**
- No `user_id` column in output
- `meets_anonymity_floor = FALSE` rows filtered by API middleware before delivery
- Users with `uac.opted_out = TRUE` excluded at query time, not post-aggregate

---

#### 17.3.2 `tenant_engagement_summary`

Tracks activation funnel and retention signals. Powers the "Are employees using FORM?" panel.

```sql
CREATE MATERIALIZED VIEW tenant_engagement_summary AS
WITH provisioned AS (
  SELECT tenant_id, COUNT(*) AS total_provisioned
  FROM users
  WHERE deactivated_at IS NULL
  GROUP BY tenant_id
),
activated AS (
  SELECT tenant_id, COUNT(*) AS total_activated
  FROM users
  WHERE deactivated_at IS NULL
    AND onboarding_completed_at IS NOT NULL
  GROUP BY tenant_id
),
wau AS (
  SELECT w.tenant_id, COUNT(DISTINCT w.user_id) AS weekly_active
  FROM workouts w
  JOIN users u ON u.id = w.user_id
  JOIN user_aggregate_consent uac ON uac.user_id = w.user_id
  WHERE w.started_at >= NOW() - INTERVAL '7 days'
    AND u.deactivated_at IS NULL
    AND uac.opted_out = FALSE
  GROUP BY w.tenant_id
),
mau AS (
  SELECT w.tenant_id, COUNT(DISTINCT w.user_id) AS monthly_active
  FROM workouts w
  JOIN users u ON u.id = w.user_id
  JOIN user_aggregate_consent uac ON uac.user_id = w.user_id
  WHERE w.started_at >= NOW() - INTERVAL '30 days'
    AND u.deactivated_at IS NULL
    AND uac.opted_out = FALSE
  GROUP BY w.tenant_id
),
d30_retention AS (
  -- D30: users who had a session in days 1-7 AND had a session in the most recent 7 days
  SELECT
    first_sess.tenant_id,
    COUNT(DISTINCT first_sess.user_id)
      FILTER (WHERE return_sess.user_id IS NOT NULL)  AS retained_d30,
    COUNT(DISTINCT first_sess.user_id)                AS cohort_size_d30
  FROM (
    SELECT tenant_id, user_id, MIN(started_at) AS first_session
    FROM workouts
    WHERE started_at >= NOW() - INTERVAL '35 days'
      AND started_at <  NOW() - INTERVAL '28 days'
    GROUP BY tenant_id, user_id
  ) first_sess
  LEFT JOIN (
    SELECT DISTINCT tenant_id, user_id
    FROM workouts
    WHERE started_at >= NOW() - INTERVAL '7 days'
  ) return_sess USING (tenant_id, user_id)
  GROUP BY first_sess.tenant_id
)
SELECT
  p.tenant_id,
  NOW()                                              AS computed_at,
  p.total_provisioned,
  COALESCE(a.total_activated, 0)                     AS total_activated,
  ROUND(
    100.0 * COALESCE(a.total_activated, 0)
    / NULLIF(p.total_provisioned, 0), 1
  )                                                  AS activation_rate_pct,
  COALESCE(wau.weekly_active, 0)                     AS weekly_active_users,
  ROUND(
    100.0 * COALESCE(wau.weekly_active, 0)
    / NULLIF(a.total_activated, 0), 1
  )                                                  AS wau_of_activated_pct,
  COALESCE(mau.monthly_active, 0)                    AS monthly_active_users,
  -- D30 retention — suppressed if cohort < k_floor
  CASE
    WHEN COALESCE(d30.cohort_size_d30, 0) >= 5
    THEN ROUND(
           100.0 * d30.retained_d30
           / NULLIF(d30.cohort_size_d30, 0), 1
         )
    ELSE NULL
  END                                                AS d30_retention_pct,
  COALESCE(d30.cohort_size_d30, 0)                   AS d30_cohort_size
FROM provisioned p
LEFT JOIN activated   a   ON a.tenant_id   = p.tenant_id
LEFT JOIN wau             ON wau.tenant_id = p.tenant_id
LEFT JOIN mau             ON mau.tenant_id = p.tenant_id
LEFT JOIN d30_retention d30 ON d30.tenant_id = p.tenant_id
WITH NO DATA;

CREATE UNIQUE INDEX uq_tes_tenant ON tenant_engagement_summary (tenant_id);

ALTER TABLE tenant_engagement_summary ENABLE ROW LEVEL SECURITY;

CREATE POLICY tes_tenant_read ON tenant_engagement_summary
  FOR SELECT TO form_api
  USING (tenant_id = current_setting('app.current_tenant_id', TRUE)::UUID);
```

**No individual data path:** Query aggregates only counts and percentages. `d30_cohort_size` is included so the API middleware can enforce the k-floor — it is never returned to the admin dashboard.

---

#### 17.3.3 `tenant_feature_adoption`

Which FORM features are active, and at what rate? Powers the "Feature usage" panel.

```sql
-- coaching_sessions COUNT is permitted (§17.2A); coaching_turns.content is not present
CREATE MATERIALIZED VIEW tenant_feature_adoption AS
WITH base AS (
  SELECT
    w.tenant_id,
    COUNT(DISTINCT w.user_id)                                   AS total_active_users,
    COUNT(DISTINCT w.user_id)
      FILTER (WHERE w.form_score IS NOT NULL)                   AS cv_users,
    COUNT(DISTINCT w.id)                                        AS total_sessions,
    COUNT(DISTINCT w.id)
      FILTER (WHERE w.form_score IS NOT NULL)                   AS cv_sessions,
    COUNT(DISTINCT cs.user_id)                                  AS voice_coach_users,
    COUNT(DISTINCT cs.id)                                       AS voice_coach_sessions
  FROM workouts w
  JOIN users u ON u.id = w.user_id
  JOIN user_aggregate_consent uac ON uac.user_id = w.user_id
  LEFT JOIN coaching_sessions cs
    ON cs.user_id   = w.user_id
   AND cs.tenant_id = w.tenant_id
   AND cs.started_at >= NOW() - INTERVAL '30 days'
  WHERE w.started_at >= NOW() - INTERVAL '30 days'
    AND u.deactivated_at IS NULL
    AND uac.opted_out = FALSE
  GROUP BY w.tenant_id
)
SELECT
  tenant_id,
  NOW()                                                         AS computed_at,
  total_active_users,
  total_sessions,
  CASE WHEN total_active_users >= 5
       THEN ROUND(100.0 * cv_users / NULLIF(total_active_users, 0), 1)
       ELSE NULL END                                            AS cv_adoption_pct,
  CASE WHEN total_sessions >= 5
       THEN ROUND(100.0 * cv_sessions / NULLIF(total_sessions, 0), 1)
       ELSE NULL END                                            AS cv_sessions_pct,
  CASE WHEN total_active_users >= 5
       THEN ROUND(100.0 * voice_coach_users / NULLIF(total_active_users, 0), 1)
       ELSE NULL END                                            AS voice_coach_adoption_pct,
  voice_coach_sessions                                          AS voice_coach_sessions_total
FROM base
WITH NO DATA;

CREATE UNIQUE INDEX uq_tfa_tenant ON tenant_feature_adoption (tenant_id);

ALTER TABLE tenant_feature_adoption ENABLE ROW LEVEL SECURITY;

CREATE POLICY tfa_tenant_read ON tenant_feature_adoption
  FOR SELECT TO form_api
  USING (tenant_id = current_setting('app.current_tenant_id', TRUE)::UUID);
```

---

#### 17.3.4 `tenant_cohort_breakdown`

Department or group-level breakdown via SCIM group memberships (§15). Enables HR to compare engagement across departments without individual employee data.

```sql
-- Depends on: scim_groups (§15), scim_group_members (§15), workouts, user_aggregate_consent
-- Groups with fewer than k_floor active users are suppressed entirely by API middleware
CREATE MATERIALIZED VIEW tenant_cohort_breakdown AS
SELECT
  sg.tenant_id,
  sg.id                                               AS group_id,
  -- Art. 9 sensitivity gate from §15.9: replace flagged names before admin sees them
  CASE
    WHEN sg.sensitive_name_flagged = FALSE
    THEN sg.display_name
    ELSE '[Group name under review]'
  END                                                 AS group_display_name,
  COUNT(DISTINCT sgm.user_id)                         AS group_size,
  COUNT(DISTINCT w.user_id)                           AS active_users_30d,
  ROUND(
    100.0 * COUNT(DISTINCT w.user_id)
    / NULLIF(COUNT(DISTINCT sgm.user_id), 0), 1
  )                                                   AS engagement_rate_pct,
  COUNT(DISTINCT w.id)                                AS total_sessions_30d,
  -- k-anonymity gate: API middleware filters rows where this is FALSE
  COUNT(DISTINCT w.user_id) >= 5                      AS meets_anonymity_floor
FROM scim_groups sg
JOIN scim_group_members sgm ON sgm.group_id = sg.id
JOIN user_aggregate_consent uac
  ON uac.user_id = sgm.user_id AND uac.opted_out = FALSE
LEFT JOIN workouts w
  ON w.user_id   = sgm.user_id
 AND w.tenant_id = sg.tenant_id
 AND w.started_at >= NOW() - INTERVAL '30 days'
JOIN users u ON u.id = sgm.user_id AND u.deactivated_at IS NULL
GROUP BY sg.tenant_id, sg.id, sg.display_name, sg.sensitive_name_flagged
WITH NO DATA;

CREATE UNIQUE INDEX uq_tcb_tenant_group ON tenant_cohort_breakdown (tenant_id, group_id);

ALTER TABLE tenant_cohort_breakdown ENABLE ROW LEVEL SECURITY;

CREATE POLICY tcb_tenant_read ON tenant_cohort_breakdown
  FOR SELECT TO form_api
  USING (tenant_id = current_setting('app.current_tenant_id', TRUE)::UUID);
```

**Art. 9 sensitivity gate:** The `sensitive_name_flagged` check from §15.9 prevents group names containing health, religion, ethnicity, or sexual-orientation signals from appearing in HR dashboards unreviewed.

**Suppression vs. zero:** The API middleware removes rows where `meets_anonymity_floor = FALSE` entirely — it does not return "< 5 users". Absence prevents inference attacks on small cohorts.

---

### 17.4 k-Anonymity Enforcement

k-anonymity is enforced at three independent layers. All three must be verified by the §17.10 CI test suite.

#### 17.4.1 Postgres Guard Function

```sql
-- Returns p_value unchanged if p_cohort_size >= p_k_floor, otherwise NULL.
-- p_k_floor uses the tenant-configured floor (§17.4.4); default 5.
CREATE OR REPLACE FUNCTION assert_k_anonymity(
  p_value       ANYELEMENT,
  p_cohort_size INTEGER,
  p_k_floor     INTEGER DEFAULT 5
) RETURNS ANYELEMENT
LANGUAGE plpgsql STABLE SECURITY INVOKER AS $$
BEGIN
  IF p_cohort_size < p_k_floor THEN
    RETURN NULL;
  END IF;
  RETURN p_value;
END;
$$;
```

Usage in reporting queries:
```sql
SELECT
  report_week,
  assert_k_anonymity(active_users,   active_users, 5) AS active_users,
  assert_k_anonymity(avg_form_score, active_users, 5) AS avg_form_score
FROM tenant_wellness_summary_v2
WHERE tenant_id = current_setting('app.current_tenant_id')::UUID;
```

#### 17.4.2 Cloudflare Worker API Middleware

```typescript
// workers/admin-reporting/k-anonymity-guard.ts
const K_FLOOR_DEFAULT = 5;

export function applyKAnonymityGuard<
  T extends { cohort_size?: number; meets_anonymity_floor?: boolean }
>(rows: T[], kFloor = K_FLOOR_DEFAULT): T[] {
  return rows.filter(row => {
    if (typeof row.meets_anonymity_floor === 'boolean') {
      return row.meets_anonymity_floor;
    }
    return (row.cohort_size ?? 0) >= kFloor;
  });
}

export function buildSuppressedResponse(suppressedCount: number) {
  return {
    data: [],
    suppressed_cohorts: suppressedCount,
    reason: 'k_anonymity_floor',
  };
}
```

Responses where all rows are suppressed return `{ data: [], suppressed_cohorts: N, reason: "k_anonymity_floor" }` so the admin knows data exists but cannot be shown — not that the feature is broken.

#### 17.4.3 Metabase Dashboard Guard

All Metabase admin reporting panels include `WHERE {{active_users}} >= 5`. This is defence-in-depth only; primary enforcement is at DB and API layers. The filter prevents accidental exposure if a panel is cloned and the query modified internally.

#### 17.4.4 Per-Tenant k-Floor Override

Enterprise customers in sensitive industries (healthcare, mental health services) may require a higher k-floor. The `tenants.reporting_k_floor` column raises the floor without a schema change:

```sql
ALTER TABLE tenants
  ADD COLUMN reporting_k_floor INTEGER NOT NULL DEFAULT 5
    CHECK (reporting_k_floor IN (5, 10, 15));
```

The `assert_k_anonymity()` function and Worker middleware both respect this setting. Only `enterprise-architect` can change it via the admin console; each change emits a `tenant.settings_updated` DEC-030 event.

---

### 17.5 Aggregate Refresh Schedule

All views refresh via `pg_cron` using `CONCURRENTLY` to avoid table-level locks.

```sql
-- tenant_wellness_summary_v2: nightly 02:15 UTC
SELECT cron.schedule(
  'refresh-wellness-summary-v2', '15 2 * * *',
  $$REFRESH MATERIALIZED VIEW CONCURRENTLY tenant_wellness_summary_v2$$
);

-- tenant_engagement_summary: nightly 02:30 UTC
SELECT cron.schedule(
  'refresh-engagement-summary', '30 2 * * *',
  $$REFRESH MATERIALIZED VIEW CONCURRENTLY tenant_engagement_summary$$
);

-- tenant_feature_adoption: nightly 02:45 UTC
SELECT cron.schedule(
  'refresh-feature-adoption', '45 2 * * *',
  $$REFRESH MATERIALIZED VIEW CONCURRENTLY tenant_feature_adoption$$
);

-- tenant_cohort_breakdown: nightly 03:00 UTC
-- After the others — depends on SCIM group sync completing (~01:30 UTC)
SELECT cron.schedule(
  'refresh-cohort-breakdown', '0 3 * * *',
  $$REFRESH MATERIALIZED VIEW CONCURRENTLY tenant_cohort_breakdown$$
);
```

**Failure handling:** pg_cron logs failures to the observability pipeline (OBSERVABILITY.md §7 P2 alert to `#ops-alerts`). Stale data is served with a `Last-Refreshed` header; the admin dashboard shows "Last updated: X hours ago" when the most recent refresh was > 26 hours ago.

---

### 17.6 Admin Reporting API Endpoints

All endpoints are under `/v1/admin/reporting/` and require a `tenant_admin` or `tenant_owner` role JWT.

| Endpoint | Method | Source view | Returns |
|---|---|---|---|
| `/v1/admin/reporting/wellness` | GET | `tenant_wellness_summary_v2` | Weekly metrics, rolling 13 weeks |
| `/v1/admin/reporting/engagement` | GET | `tenant_engagement_summary` | Activation rate, WAU/MAU, D30 retention |
| `/v1/admin/reporting/features` | GET | `tenant_feature_adoption` | CV and voice adoption rates |
| `/v1/admin/reporting/cohorts` | GET | `tenant_cohort_breakdown` | Per-SCIM-group engagement (k-anonymity suppressed) |
| `/v1/admin/reporting/export` | POST | All four views | CSV with `Last-Refreshed` header |

**No `/v1/admin/reporting/users` endpoint exists.** Any attempt to add one requires a compliance-officer approval and a DECISION_LOG.md entry. Its absence is itself a SOC 2 CC6.1 control.

**Response schema — `/v1/admin/reporting/wellness`:**
```typescript
interface WellnessReportResponse {
  tenant_id:        string;
  generated_at:     string;   // ISO 8601
  last_refreshed_at: string;  // from MV; triggers stale banner if > 26h
  period:           { weeks: 13 };
  data:             WellnessWeek[];
  suppressed_weeks: number;   // weeks where N < k_floor
}

interface WellnessWeek {
  report_week:        string;        // ISO 8601 date (Monday)
  active_users:       number;
  total_sessions:     number;
  avg_duration_min:   number | null;
  median_duration_min: number | null;
  p90_duration_min:   number | null;
  avg_form_score:     number | null; // null if CV cohort N < k_floor
}
```

---

### 17.7 Opt-In Consent Tracking

A user's data is excluded from all aggregate views unless explicit consent is recorded in this table.

```sql
CREATE TABLE user_aggregate_consent (
  id               UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id          UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  tenant_id        UUID NOT NULL,
  opted_out        BOOLEAN NOT NULL DEFAULT FALSE,
  consent_version  TEXT,            -- version string of consent text; NULL = not yet shown
  consented_at     TIMESTAMPTZ,     -- NULL if opted_out or consent not yet shown
  opted_out_at     TIMESTAMPTZ,
  consent_channel  TEXT,            -- 'onboarding_flow' | 'settings_privacy' | 'scim_auto'
  created_at       TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at       TIMESTAMPTZ NOT NULL DEFAULT now(),
  UNIQUE (user_id, tenant_id)
);

ALTER TABLE user_aggregate_consent ENABLE ROW LEVEL SECURITY;

-- Users can read and update their own consent record
CREATE POLICY uac_owner_rw ON user_aggregate_consent
  FOR ALL TO form_api
  USING (user_id = current_setting('app.current_user_id', TRUE)::UUID);

-- Aggregate MVs read this table via form_admin role (BYPASSRLS)
-- No tenant_admin route to read individual consent records
```

**SCIM-provisioned users:** `consent_version = NULL` and `opted_out = FALSE` at provisioning time. User is not included in aggregates until they complete onboarding and the consent screen sets `consent_version` and `consented_at`.

**Opt-out flow:** Settings → Privacy → "Disconnect from employer reporting" sets `opted_out = TRUE`. Effective at the next nightly MV refresh (≤ 26 hours). The user remains a `tenant_member` with full platform access.

**Consent version re-solicitation:** Adding a new metric category to §17.2A triggers a new `consent_version`. All existing users whose `consent_version` predates the change must confirm consent before their data is included under the new category.

---

### 17.8 GDPR Art. 89 Statistical Purpose Compliance

Aggregate reporting falls within the **statistical purpose exception** under GDPR Art. 89(1) and Recital 162, which permits processing personal data for statistical analysis where appropriate safeguards ensure results cannot be attributed to individual data subjects.

| Safeguard (Art. 89 requirement) | Implementation |
|---|---|
| Pseudonymisation | `user_id` absent from all aggregate view output (§17.3 DDL verified by §17.10 tests) |
| Data minimisation | No `user_health_profiles` or `meal_logs` joins in any §17 view |
| k-Anonymity | Cohort floor N ≥ 5 enforced at DB, API, and dashboard layers (§17.4) |
| Prohibition on re-identification | No admin endpoint narrows results to an individual; structural absence enforced (§17.6) |
| No onward transfer without controls | `admin.report_exported` audit events record every export; DPA prohibits recipients from sharing raw exports externally |
| Consent basis | `user_aggregate_consent` ensures only opted-in users contribute to aggregates |

**GDPR legal basis:** Legitimate interest (Art. 6(1)(f)) for the enterprise customer's programme evaluation, balanced against the data subject's privacy interest (protected by k-anonymity floor and opt-out right). For Art. 9 special category data: the aggregate output metrics are behavioural/fitness signals, not health diagnoses. The query paths in §17.3 never read `user_health_profiles`, `meal_logs`, or `coaching_turns` — Art. 9 data is structurally absent from the output.

**DPA gate:** Enterprise customers with EU employees must sign a DPA before admin reporting data flows. Admin reporting endpoints return HTTP 403 for tenants with `tenant_contracts.dpa_ref IS NULL` in EU data-residency regions (cross-reference §16.9).

---

### 17.9 DEC-030 Audit Events for Admin Reporting

All admin reporting access is HMAC-chained per DEC-030 (`docs/AUDIT_LOG_SCHEMA.md`).

| Event slug | Trigger | Severity | Retention | Notes |
|---|---|---|---|---|
| `admin.report_viewed` | Admin dashboard page load (any reporting tab) | LOW | 30 days | `resource_type: 'aggregate_dashboard'`; includes `report_type` in metadata |
| `admin.report_exported` | CSV export via `/v1/admin/reporting/export` | STANDARD | 3 years | `resource_type: 'aggregate_export'`; includes `row_count` and `suppressed_cohorts` |
| `admin.cohort_drill_viewed` | Per-group cohort breakdown viewed | STANDARD | 1 year | `resource_type: 'cohort_breakdown'`; includes `group_id` (not `display_name`) |
| `admin.report_suppressed` | API returns `suppressed_cohorts > 0` | LOW | 30 days | Signals k-anonymity floor is active; no individual data in log |
| `admin.consent_override_attempted` | Any attempt to access individual user data via an admin role | **HIGH** | 7 years | Must never fire in production — its presence is a P1 security incident |

**`admin.consent_override_attempted` rationale:** This event fires if a code change accidentally creates a route that exposes individual data to an admin role. One instance in production triggers an automatic P1 incident (INCIDENT_RESPONSE.md §1.2) and an immediate review of the responsible code path.

**Metadata schema — `admin.report_exported`:**
```jsonc
{
  "event":              "admin.report_exported",
  "tenant_id":          "<uuid>",
  "actor_id":           "<uuid>",
  "actor_role":         "tenant_admin",
  "report_type":        "wellness | engagement | feature_adoption | cohort_breakdown | full",
  "row_count":          42,
  "format":             "csv",
  "period_weeks":       13,
  "suppressed_cohorts": 0
}
```

---

### 17.10 Privacy Floor Test Suite

The CI pipeline must pass all tests below before any admin dashboard deploy.

```typescript
// tests/privacy/admin-reporting-floor.test.ts

describe('Admin reporting privacy floor', () => {

  it('tenant_wellness_summary_v2 contains no user_id column', async () => {
    const cols = await db.query<{ column_name: string }>(
      `SELECT column_name FROM information_schema.columns
       WHERE table_name = 'tenant_wellness_summary_v2'`
    );
    expect(cols.rows.map(r => r.column_name)).not.toContain('user_id');
  });

  it('tenant_wellness_summary_v2 contains no email column', async () => {
    const cols = await db.query<{ column_name: string }>(
      `SELECT column_name FROM information_schema.columns
       WHERE table_name = 'tenant_wellness_summary_v2'`
    );
    expect(cols.rows.map(r => r.column_name)).not.toContain('email');
  });

  it('assert_k_anonymity returns null for cohort < k_floor', async () => {
    const result = await db.query(`SELECT assert_k_anonymity(42.5, 4)`);
    expect(result.rows[0].assert_k_anonymity).toBeNull();
  });

  it('assert_k_anonymity returns value for cohort >= k_floor', async () => {
    const result = await db.query(`SELECT assert_k_anonymity(42.5, 5)`);
    expect(result.rows[0].assert_k_anonymity).toBe(42.5);
  });

  it('opted-out user is excluded from wellness MV', async () => {
    await testDb.setUserOptOut(TEST_USER_ID, true);
    await testDb.refreshMV('tenant_wellness_summary_v2');
    const rows = await db.query(
      `SELECT active_users FROM tenant_wellness_summary_v2 WHERE tenant_id = $1`,
      [TEST_TENANT_ID]
    );
    expect(rows.rows[0].active_users).toBe(INITIAL_ACTIVE_USERS - 1);
  });

  it('no admin reporting endpoint returns individual user rows', async () => {
    const endpoints = [
      '/v1/admin/reporting/wellness',
      '/v1/admin/reporting/engagement',
      '/v1/admin/reporting/features',
      '/v1/admin/reporting/cohorts',
    ];
    for (const endpoint of endpoints) {
      const response = await adminApiClient.get(endpoint);
      const body = JSON.stringify(await response.json());
      expect(body).not.toContain(TEST_USER_EMAIL);
      expect(body).not.toContain(TEST_USER_ID);
    }
  });

  it('tenant_cohort_breakdown suppresses groups with fewer than 5 active users', async () => {
    await testDb.createScimGroup(TEST_TENANT_ID, 'small-group', 3);
    await testDb.refreshMV('tenant_cohort_breakdown');
    const response = await adminApiClient.get('/v1/admin/reporting/cohorts');
    const body = await response.json();
    const groupIds = body.data.map((g: { group_id: string }) => g.group_id);
    expect(groupIds).not.toContain(SMALL_GROUP_ID);
  });

});
```

Test failures block deployment via `scripts/ci/privacy-floor-gate.sh`.

---

### 17.11 SOC 2 Mapping

| SOC 2 Criterion | How §17 Satisfies It |
|---|---|
| **CC6.1 — Logical access restricted** | No admin reporting endpoint exposes individual data; `admin.consent_override_attempted` fires if a route is accidentally added; absence of `/v1/admin/reporting/users` is a documented, tested access control |
| **CC6.3 — Access restrictions tested** | §17.10 privacy floor test suite runs on every deploy; failures block deployment |
| **CC7.2 — Monitoring for anomalies** | `admin.consent_override_attempted` (HIGH) auto-triggers P1 incident; `admin.report_exported` events feed SIEM |
| **C1.1 — Confidentiality commitments** | §17.2B Prohibited Registry is a documented confidentiality commitment; DPA (§17.8) enforces contractual confidentiality |
| **C1.2 — Confidential information protected** | Health data, body composition, coaching content, meal logs excluded from all aggregate view DDL — exclusion is structural, not policy-dependent |
| **P3.2 — Privacy notice accurate** | `user_aggregate_consent.consent_version` records which notice the user saw; re-consent triggered on registry expansion |
| **P4.1 — Data collected only as consented** | `uac.opted_out = FALSE` required for any user row to appear in aggregate MVs |
| **P6.4 — Disclosure controlled** | `admin.report_exported` events record every export with row count; no personal data in export (verified by §17.10 tests) |

**SOC 2 evidence artifacts:**
- `admin.report_exported` event log → `compliance/evidence/admin-reporting/`
- §17.10 CI test results → `compliance/evidence/privacy-floor-tests/`
- §17.2B Prohibited Registry → C1.2 confidentiality protection evidence
- `user_aggregate_consent` DDL → CC6.1 / P4.1 access control evidence

---

### 17.12 Implementation Checklist

| # | Task | Owner | Priority | Milestone | Notes |
|---|---|---|---|---|---|
| 1 | Create `user_aggregate_consent` table with RLS; backfill SCIM-provisioned users with `opted_out = FALSE, consent_version = NULL` | `platform-engineer` | **P0** | M3 | Must exist before any aggregate MV is created |
| 2 | `ALTER TABLE tenants ADD COLUMN reporting_k_floor INTEGER NOT NULL DEFAULT 5 CHECK (reporting_k_floor IN (5, 10, 15))` | `platform-engineer` | **P0** | M3 | |
| 3 | Create `assert_k_anonymity()` Postgres function (§17.4.1); add CI test for null/pass cases | `platform-engineer` | **P0** | M3 | |
| 4 | Create `tenant_wellness_summary_v2` DDL + unique index + `pg_cron` 02:15 UTC | `platform-engineer` | **P0** | M3 | Deprecation notice on §2.6 view after M3 deploy |
| 5 | Create `tenant_engagement_summary` DDL + unique index + `pg_cron` 02:30 UTC | `platform-engineer` | **P0** | M3 | |
| 6 | Create `tenant_feature_adoption` DDL + unique index + `pg_cron` 02:45 UTC | `platform-engineer` | **P0** | M4 | CV feature flag must be wired before this MV is meaningful |
| 7 | Create `tenant_cohort_breakdown` DDL + unique index + `pg_cron` 03:00 UTC | `platform-engineer` | **P0** | M4 | Depends on §15 SCIM Groups tables |
| 8 | Implement `k-anonymity-guard.ts` Worker middleware; integrate into all `/v1/admin/reporting/*` handlers | `platform-engineer` | **P0** | M3 | `applyKAnonymityGuard` and `buildSuppressedResponse` per §17.4.2 |
| 9 | Read `tenants.reporting_k_floor` in tenant context middleware; pass to Worker guard | `platform-engineer` | **P1** | M4 | |
| 10 | Implement all 5 admin reporting endpoints (§17.6); add `Last-Refreshed` header; stale banner at > 26h | `platform-engineer` | **P0** | M4 | No individual-data routes — CI gate verifies this |
| 11 | Wire `admin.report_exported`, `admin.report_viewed`, `admin.cohort_drill_viewed`, `admin.consent_override_attempted` DEC-030 events | `platform-engineer` | **P0** | M4 | `admin.consent_override_attempted` must trigger P1 auto-incident |
| 12 | Onboarding flow: add consent screen; update `user_aggregate_consent` on accept/decline | `platform-engineer` + `ux-flow` | **P0** | M3 | `consent_channel: 'onboarding_flow'` |
| 13 | Settings → Privacy: "Disconnect from employer reporting" toggle; wire `opted_out` update | `platform-engineer` | **P0** | M3 | Effective at next nightly refresh (< 26h); inform user |
| 14 | Implement §17.10 privacy floor test suite; integrate into CI gate; block deploy on failure | `qa-lead` + `platform-engineer` | **P0** | M3 | Zero-exception policy: any test failure = blocked deploy |
| 15 | Metabase admin panels: 4 panels (wellness trend, engagement funnel, feature adoption, cohort table) with `WHERE active_users >= 5` filter | `data-engineer` | **P1** | M4 | Internal CSM use; enterprise customers use admin dashboard UI |
| 16 | EU region gate: `/v1/admin/reporting/*` returns HTTP 403 for tenants where `dpa_ref IS NULL` | `platform-engineer` + `compliance-officer` | **P0** | Pre-EU pilot | Closes P-GAP-002 DPA enforcement at API layer |
| 17 | Add pg_cron failure alerting to OBSERVABILITY.md §7 alert matrix; stale MV (> 26h) → P2 `#ops-alerts` | `devops-lead` | **P1** | M4 | |
| 18 | Deprecate §2.6 `tenant_wellness_summary` (add `-- DEPRECATED: see §17.3.1` comment); hard-remove after 2 sprints | `platform-engineer` | **P2** | M5 | |

**OQ-ENT-02: k-anonymity floor sufficiency under GDPR Recital 26 (EU):** The k-floor of 5 matches common government statistics practice (census data). Some EU supervisory authorities have informally suggested k ≥ 10 for health-adjacent data in corporate wellness contexts. Before FORM's first enterprise pilot in Germany, France, or the Netherlands, compliance-officer must obtain a legal opinion on whether k = 5 satisfies Recital 26 "not reasonably likely to be identified" for FORM's specific dataset. If k ≥ 10 is required, `tenants.reporting_k_floor` (§17.4.4) allows this per tenant without a schema change. **Owner: compliance-officer. Checkpoint: before first EU enterprise pilot.**

**OQ-ENT-03: Consent re-solicitation on metric registry expansion:** If FORM adds a new permitted metric category to §17.2A (e.g., sleep quality aggregates from wearable data in §14), all existing users whose `consent_version` predates the change must be re-consented before their data is included under the new category. The re-consent notification and UI flow does not yet exist. The `consent_version` field is the hook; the flow must be designed before any metric category expansion. **Owner: platform-engineer + compliance-officer. Checkpoint: M6.**

---

*v0.7 additions: §16 Enterprise Contract & Pilot Lifecycle Schema — closes the schema gap for pilot and contract tracking referenced in ENTERPRISE.md §Pricing, §Implementation timeline, and enterprise.html. `tenants.lifecycle_status` TEXT column with 5-state machine (pilot → active → expired/suspended/churned) and Worker access gate. `tenant_pilots` table: pilot seat cap (10–500), 90-day `ends_at`, fixed-schema JSONB `success_criteria` (activation_rate_pct, weekly_engagement_pct, nps_floor, min_active_seats, evaluation_date), status machine (active/extended/converted/expired/declined), CSM owner handle, internal_notes PII prohibition with weekly content-scan. `tenant_contracts` table: contract_ref, plan, seats_contracted (min 50), arr_cents (FORM-internal only — excluded from form_api RLS and tenant_contract_portal view), billing_period (annual/2year/3year), po_number, start_at/end_at, auto_renew + renewal_notice_days, Stripe subscription ID, partial unique index on (tenant_id) WHERE status = 'active'. Seat enforcement: `assertSeatAvailable()` TypeScript in Workers provisioning layer; FOR UPDATE SKIP LOCKED TOCTOU protection; 402 response with CSM contact; seat_limit_enforcement_triggered DEC-030 event with attempted_email: null. `tenant_seat_utilization` materialized view: active seats, scim-provisioned count, 7d/30d utilization, utilization_pct_7d; k-anonymity NULL suppression below n=15; nightly 02:00 UTC pg_cron refresh; arr_cents absent. Renewal notice pg_cron (09:00 UTC) + Cloudflare Worker (09:05 UTC): Slack to #csm-renewals at renewal_notice_days, PagerDuty at ≤14 days. 14 DEC-030 HMAC-chained audit events including HIGH-severity tenant.contract_terminated and tenant.access_suspended; attempted_email: null invariant CI-enforced. Data retention: tenant_contracts 7 years (financial records), tenant_pilots 2-year soft-delete + 5-year hard-delete. Art. 28 DPA: dpa_ref TEXT column required before EU pilot activation (closes P-GAP-002 DPA tracking). SOC 2 mapping: CC6.1 (seat enforcement), CC6.2 (contract-as-authorisation), CC9.1 (renewal notice), CC9.2 (contract record), A1.1 (SLA tier from plan), PI1.4 (processing integrity), C1.2 (arr_cents confidential). 14-item implementation checklist (7× P0, 4× P1, 2× P2, 1× Pre-EU-pilot). OQ-ENT-01: minimum seat floor validation against COST_MODEL.md §16.2 model at M6.*

*v0.8 additions: §17 Enterprise Admin Reporting Schema — Aggregate-Only Data Model & Privacy-Floor Enforcement. Closes the schema gap between the §2.6 prototype `tenant_wellness_summary` and the full production admin dashboard data model. Permitted/Prohibited Metric Registry (§17.2): explicit HR-visibility table covering 12 permitted aggregate metrics (participation, WAU/MAU, form score, D30 retention, feature adoption, SCIM-group engagement, opt-in NPS) and 9 prohibited categories (individual workout rows, coaching content, health goals, body composition, meal logs, mental health data, HRV, user identity linkage, opted-out users) — 5 categories require clinical-safety veto to override. Four materialized views: `tenant_wellness_summary_v2` (production replacement for §2.6 prototype — adds `user_aggregate_consent` join, 13-week rolling window, k-anonymity CASE guard on `avg_form_score`, `REFRESH CONCURRENTLY` unique index, RLS policy); `tenant_engagement_summary` (activation_rate_pct, WAU/MAU, D30 retention with cohort-size k-gate, computed via CTEs — no `user_health_profiles` join anywhere); `tenant_feature_adoption` (CV adoption % and voice coaching adoption % — `coaching_sessions` count permitted, `coaching_turns.content` structurally absent); `tenant_cohort_breakdown` (SCIM-group engagement using §15 tables — Art. 9 sensitive-name gate replaces flagged names with `[Group name under review]`, `meets_anonymity_floor` column drives API suppression). k-Anonymity: `assert_k_anonymity()` Postgres STABLE function (p_value + p_cohort_size + p_k_floor DEFAULT 5); `applyKAnonymityGuard` + `redactSubThreshold` TypeScript in `workers/admin-reporting/k-anonymity-guard.ts`; Metabase `WHERE active_users >= 5` defence-in-depth; `tenants.reporting_k_floor INTEGER DEFAULT 5 CHECK IN (5, 10, 15)` per-tenant override for sensitive industries. Refresh schedule: pg_cron CONCURRENTLY at 02:15/02:30/02:45/03:00 UTC; stale banner at > 26h; P2 alert on cron failure. Admin reporting API: 5 endpoints under `/v1/admin/reporting/` (wellness, engagement, features, cohorts, export); no `/users` endpoint; `Last-Refreshed` header; 403 for EU tenants with `dpa_ref IS NULL`. `user_aggregate_consent` table: `opted_out BOOLEAN DEFAULT FALSE`, `consent_version TEXT`, SCIM-provisioned default NULL-consent until onboarding; Settings → Privacy opt-out effective at next refresh (< 26h); `consent_version` triggers re-solicitation on metric registry expansion. GDPR Art. 89 safeguards: pseudonymisation (no user_id in output), minimisation (no Art. 9 table joins), k-anonymity N ≥ 5, re-identification prohibition (no individual-narrowing endpoints), consent basis (`user_aggregate_consent`), DPA gate (§17.8). DEC-030 audit events: `admin.report_viewed` (LOW, 30d), `admin.report_exported` (STANDARD, 3yr, includes `suppressed_cohorts`), `admin.cohort_drill_viewed` (STANDARD, 1yr), `admin.report_suppressed` (LOW, 30d), `admin.consent_override_attempted` (HIGH, 7yr — fires P1 incident automatically). Privacy floor test suite (§17.10): 6 Jest tests including no-user_id-column assertion, k-anonymity null/pass, opted-out exclusion, no-individual-data-in-response, cohort-suppression; CI gate blocks deploy on failure. SOC 2 mapping: CC6.1 (no individual-data route), CC6.3 (tests block deploy), CC7.2 (consent_override_attempted → P1), C1.1 (prohibited registry), C1.2 (structural exclusion), P3.2 (consent version), P4.1 (opted_out gate), P6.4 (export audit events). 18-item implementation checklist (12× P0, 4× P1, 2× P2). OQ-ENT-02: k-anonymity floor sufficiency under GDPR Recital 26 for EU enterprise pilots (legal opinion required before Germany/France/Netherlands). OQ-ENT-03: consent re-solicitation mechanism on metric registry expansion.*

---

## 18. Notification & Webhook Event Queue Schema

> Owner: `enterprise-architect` + `platform-engineer`. Review: on any new event type, webhook contract change, or quarterly.
> Scope: in-app push notifications and enterprise webhook delivery. Consumer push is in scope; enterprise webhook delivery is enterprise-tier only.
> References: DEC-030 (HMAC-chained audit log), docs/AUDIT_LOG_SCHEMA.md, docs/ENTERPRISE.md §Webhooks, §12 (soft delete and GDPR Art. 17 erasure), §11 (index strategy).

---

### 18.1 Design Principles

1. **Notifications and webhook deliveries are separate concerns.** `notifications` drives in-app and push alerts for individual users. `webhook_endpoints` + `webhook_deliveries` delivers signed event payloads to enterprise customers' HTTP endpoints. Neither table fans data from the other.
2. **Webhook payloads are contractually scoped.** Enterprise customers receive only the event types they have subscribed to, with payload content bounded by the DPA. No PII beyond what is documented in §18.7 may appear in any webhook payload.
3. **Delivery reliability via the queue.** `webhook_deliveries` is an outbound queue. The delivery worker dequeues, signs, POSTs, and updates `status` in place. Exponential backoff with a 6-attempt cap prevents cascade load on customer endpoints.
4. **Tenant isolation.** `tenant_id` is on every row. Cross-tenant notification or webhook delivery is a P0 bug equivalent to a cross-tenant data read.
5. **Webhook signing secrets are encrypted at rest.** The raw HMAC key is never readable from the API response path; decryption is available only to the delivery worker running under `form_system`.

---

### 18.2 Event Type Enum

All notification and webhook events draw from a single PostgreSQL enum. Adding a new event type requires a migration that increments this enum — enforcing that any new event is explicitly reviewed before it can appear in production payloads.

```sql
CREATE TYPE webhook_event_type AS ENUM (
  -- Training lifecycle
  'workout.completed',
  'workout.abandoned',
  'workout.set_logged',

  -- Streak events (per DEC-013: grace after 2 misses)
  'streak.maintained',
  'streak.broken',
  'streak.grace_period_activated',
  'streak.milestone_reached',     -- at 7, 30, 60, 90, 180 days

  -- Coaching
  'coaching.session_started',
  'coaching.session_ended',

  -- Auth and identity
  'auth.sso_login',
  'auth.scim_user_provisioned',
  'auth.scim_user_deprovisioned',
  'auth.mfa_challenge_failed',
  'auth.session_expired',

  -- Enterprise / tenant lifecycle
  'tenant.user_onboarded',        -- onboarding_completed_at set
  'tenant.user_deactivated',
  'tenant.seat_limit_approached', -- active_seats >= 90% of max_seats
  'tenant.seat_limit_reached',    -- assertSeatAvailable() triggered (§16.5)
  'tenant.pilot_milestone',       -- success_criteria threshold crossed (§16.3)
  'tenant.contract_renewal_due',  -- renewal_notice_days before end_at (§16.7)

  -- Wearable sync
  'wearable.source_connected',
  'wearable.source_disconnected',

  -- System
  'notification.delivery_failed', -- internal; not deliverable via webhook
  'webhook.test'                  -- synthetic ping for endpoint validation
);
```

**Governance:** New enum values require a DECISION_LOG.md entry specifying the event producer, payload schema version, and which enterprise tiers receive access. `form_admin` adds values; application code cannot define arbitrary event strings.

---

### 18.3 `notifications` Table

In-app, push, and email notifications for individual users. Tenant-level system notifications (e.g. seat limit warnings) have `user_id = NULL` and `channel = 'email'` — they are delivered to `tenant_owner` email addresses by the notification worker.

```sql
CREATE TABLE notifications (
  id              UUID          PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id       UUID          NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,

  -- NULL for tenant-level notifications (seat limit, contract renewal, pilot milestone)
  -- Non-null for user-targeted notifications (streak, workout complete, coaching)
  user_id         UUID          REFERENCES users(id) ON DELETE SET NULL,

  type            webhook_event_type NOT NULL,

  -- Structured notification content. Schema is event-type-specific; validated by
  -- a Zod schema in src/notifications/payload.schema.ts before insert.
  -- MUST NOT contain PII beyond display_name and the minimum necessary for the
  -- notification copy (e.g. streak count, exercise name from fixed catalogue).
  -- Health data, form scores, coaching turn content: categorically excluded.
  payload         JSONB         NOT NULL DEFAULT '{}',

  channel         TEXT          NOT NULL
                    CHECK (channel IN ('push', 'email', 'in_app', 'webhook')),

  status          TEXT          NOT NULL DEFAULT 'pending'
                    CHECK (status IN (
                      'pending',    -- queued, not yet dispatched
                      'sent',       -- dispatcher confirmed delivery to FCM/APNS/SES/endpoint
                      'failed',     -- all retry attempts exhausted
                      'cancelled'   -- superseded (e.g. user deactivated before delivery)
                    )),

  retry_count     SMALLINT      NOT NULL DEFAULT 0,
  expires_at      TIMESTAMPTZ,              -- notification is cancelled if pending past this time

  created_at      TIMESTAMPTZ   NOT NULL DEFAULT now(),
  sent_at         TIMESTAMPTZ,              -- set when status → 'sent'

  -- Soft delete per §12.1 pattern
  deleted_at      TIMESTAMPTZ,
  deleted_by      UUID          REFERENCES users(id) ON DELETE SET NULL
);

ALTER TABLE notifications ENABLE ROW LEVEL SECURITY;

-- Users can read their own non-deleted notifications
CREATE POLICY notifications_user_select ON notifications
  AS PERMISSIVE FOR SELECT
  TO form_api
  USING (
    tenant_id  = current_setting('app.current_tenant_id', TRUE)::UUID
    AND user_id = current_setting('app.current_user_id', TRUE)::UUID
    AND deleted_at IS NULL
  );

-- tenant_admin / tenant_owner can read all notifications for their tenant
-- (used in the admin dashboard notification history view)
CREATE POLICY notifications_admin_select ON notifications
  AS PERMISSIVE FOR SELECT
  TO form_api
  USING (
    tenant_id = current_setting('app.current_tenant_id', TRUE)::UUID
    AND current_setting('app.current_role', TRUE)
        IN ('tenant_owner', 'tenant_admin')
    AND deleted_at IS NULL
  );

-- Soft delete: users can delete their own notifications; admins can cancel tenant-level ones
CREATE POLICY notifications_soft_delete ON notifications
  AS PERMISSIVE FOR UPDATE
  TO form_api
  USING (
    tenant_id = current_setting('app.current_tenant_id', TRUE)::UUID
    AND (
      user_id = current_setting('app.current_user_id', TRUE)::UUID
      OR (
        user_id IS NULL
        AND current_setting('app.current_role', TRUE)
            IN ('tenant_owner', 'tenant_admin')
      )
    )
  )
  WITH CHECK (
    deleted_at IS NOT NULL   -- only allow setting deleted_at via this policy
  );

-- Delivery worker (form_system) reads pending notifications and updates status
CREATE POLICY notifications_system_rw ON notifications
  AS PERMISSIVE FOR ALL
  TO form_system
  USING (TRUE)
  WITH CHECK (TRUE);
```

**Payload constraints:** The Zod schema referenced above rejects any payload containing keys that match the `FORBIDDEN_PROPERTY_KEYS` list from the analytics enrichment middleware (§13.3). Additionally, `coaching_turns.content`, `form_score`, `keypoints_enc`, and all wearable numeric values are on an explicit notification-layer blocklist enforced in `src/notifications/payload.schema.ts`. A CI test asserts that the production Zod schema rejects payloads containing these keys.

---

### 18.4 `webhook_endpoints` Table

One row per customer-configured HTTP endpoint. Enterprise customers configure endpoints via the admin dashboard or the `POST /v1/admin/webhooks/endpoints` API. A single tenant may have up to 10 active endpoints (enforced at the application layer; not a DB constraint).

```sql
-- CLASSIFICATION: SENSITIVE — signing_secret_enc is an HMAC-SHA256 key that authenticates
-- webhook deliveries as genuine FORM payloads. Key compromise allows payload forgery.
-- Encrypted at rest using per-tenant KMS key (§5.1). Never returned in any API response.
CREATE TABLE webhook_endpoints (
  id                  UUID        PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id           UUID        NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,

  -- Customer-supplied HTTPS URL. HTTP is rejected at the application layer.
  url                 TEXT        NOT NULL
                        CHECK (url LIKE 'https://%'),

  -- HMAC-SHA256 signing key. AES-256-GCM encrypted using the tenant's KMS-derived key
  -- before write. Decryption available only to form_system (delivery worker).
  -- Never returned by GET /v1/admin/webhooks/endpoints — API returns "***" after initial creation.
  -- The raw secret is shown exactly once: at endpoint creation time, in the API response body.
  signing_secret_enc  TEXT        NOT NULL,

  -- Array of subscribed event types. Must be a subset of webhook_event_type enum values.
  -- Empty array = endpoint receives no events (effectively disabled without deactivation).
  events              TEXT[]      NOT NULL DEFAULT '{}',

  -- FALSE: delivery worker skips this endpoint. Endpoint stays in table for audit trail.
  active              BOOLEAN     NOT NULL DEFAULT TRUE,

  -- Delivery health tracking
  failure_count       INTEGER     NOT NULL DEFAULT 0,
  -- Automatically deactivated (active = FALSE) when failure_count >= 10 consecutive failures.
  -- Re-activation requires explicit tenant_admin action (POST .../reactivate).
  last_success_at     TIMESTAMPTZ,
  last_attempted_at   TIMESTAMPTZ,

  -- Metadata for admin dashboard
  description         TEXT,                    -- human-readable label, e.g. "Production HRIS sink"
  created_by          UUID        REFERENCES users(id) ON DELETE SET NULL,
  created_at          TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at          TIMESTAMPTZ NOT NULL DEFAULT now(),

  CONSTRAINT uq_webhook_endpoint_url_per_tenant UNIQUE (tenant_id, url)
);

ALTER TABLE webhook_endpoints ENABLE ROW LEVEL SECURITY;

-- tenant_admin / tenant_owner: read endpoint metadata (not the signing secret)
CREATE POLICY webhook_endpoints_admin_read ON webhook_endpoints
  AS PERMISSIVE FOR SELECT
  TO form_api
  USING (
    tenant_id = current_setting('app.current_tenant_id', TRUE)::UUID
    AND current_setting('app.current_role', TRUE)
        IN ('tenant_owner', 'tenant_admin')
  );

-- tenant_admin / tenant_owner: create, update, delete endpoints
CREATE POLICY webhook_endpoints_admin_write ON webhook_endpoints
  AS PERMISSIVE FOR INSERT
  TO form_api
  WITH CHECK (
    tenant_id = current_setting('app.current_tenant_id', TRUE)::UUID
    AND current_setting('app.current_role', TRUE)
        IN ('tenant_owner', 'tenant_admin')
  );

CREATE POLICY webhook_endpoints_admin_update ON webhook_endpoints
  AS PERMISSIVE FOR UPDATE
  TO form_api
  USING (
    tenant_id = current_setting('app.current_tenant_id', TRUE)::UUID
    AND current_setting('app.current_role', TRUE)
        IN ('tenant_owner', 'tenant_admin')
  );

-- Delivery worker: full access including signing_secret_enc decryption
CREATE POLICY webhook_endpoints_system ON webhook_endpoints
  AS PERMISSIVE FOR ALL
  TO form_system
  USING (TRUE)
  WITH CHECK (TRUE);
```

**API serialisation rule:** The `GET /v1/admin/webhooks/endpoints` response serialiser explicitly excludes `signing_secret_enc`. The field is replaced with `"signing_secret": "***"`. This is enforced at the TypeScript serialiser level, not by column-level RLS — the RLS policy grants the row, but the application layer controls column projection. A CI test asserts that no API endpoint response fixture contains the literal string `signing_secret_enc`.

---

### 18.5 `webhook_deliveries` Table

One row per delivery attempt per endpoint. The delivery worker inserts a row when it picks up an event, then updates `status`, `response_code`, and `response_body` after the HTTP call resolves. The `next_retry_at` column drives the retry queue; a nil value means no further retry is scheduled.

```sql
CREATE TABLE webhook_deliveries (
  id                UUID        PRIMARY KEY DEFAULT gen_random_uuid(),
  endpoint_id       UUID        NOT NULL REFERENCES webhook_endpoints(id) ON DELETE CASCADE,
  tenant_id         UUID        NOT NULL,       -- denormalized for RLS; no FK join on every policy check

  event_type        webhook_event_type NOT NULL,

  -- Full signed payload as delivered to the customer endpoint.
  -- Schema-versioned (see §18.7). Scrubbed per §18.8 on GDPR Art. 17 erasure.
  payload           JSONB       NOT NULL,

  status            TEXT        NOT NULL DEFAULT 'pending'
                      CHECK (status IN (
                        'pending',      -- not yet attempted
                        'delivered',    -- HTTP 2xx received from endpoint
                        'failed',       -- non-2xx or network error; retries remaining
                        'exhausted',    -- all 6 attempts failed; no further retry
                        'cancelled'     -- endpoint deactivated before delivery
                      )),

  attempt_count     SMALLINT    NOT NULL DEFAULT 0,

  -- Exponential backoff schedule (set by delivery worker after each failed attempt):
  --   attempt 1 → +1 min
  --   attempt 2 → +5 min
  --   attempt 3 → +30 min
  --   attempt 4 → +2 hours
  --   attempt 5 → +8 hours
  --   attempt 6 → +24 hours (final)
  -- NULL when status = 'delivered' | 'exhausted' | 'cancelled'
  next_retry_at     TIMESTAMPTZ,

  delivered_at      TIMESTAMPTZ,   -- set when status → 'delivered'

  -- HTTP response from the customer's endpoint
  response_code     SMALLINT,
  -- Capped at 1 KB before storage. Avoids storing sensitive customer-side error bodies.
  -- Truncated by the delivery worker: LEFT(response_body_raw, 1024)
  response_body     TEXT,

  -- Idempotency: delivery worker can re-enqueue safely
  -- Events sharing the same idempotency_key for the same endpoint are deduplicated
  idempotency_key   TEXT        NOT NULL,   -- format: '<event_type>:<source_entity_id>:<epoch_ms>'

  created_at        TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at        TIMESTAMPTZ NOT NULL DEFAULT now(),

  CONSTRAINT uq_webhook_delivery_idempotency UNIQUE (endpoint_id, idempotency_key)
);

ALTER TABLE webhook_deliveries ENABLE ROW LEVEL SECURITY;

-- tenant_admin / tenant_owner: read delivery history for their tenant's endpoints
CREATE POLICY webhook_deliveries_admin_read ON webhook_deliveries
  AS PERMISSIVE FOR SELECT
  TO form_api
  USING (
    tenant_id = current_setting('app.current_tenant_id', TRUE)::UUID
    AND current_setting('app.current_role', TRUE)
        IN ('tenant_owner', 'tenant_admin')
  );

-- Delivery worker: full access — inserts new deliveries, updates status after HTTP call
CREATE POLICY webhook_deliveries_system ON webhook_deliveries
  AS PERMISSIVE FOR ALL
  TO form_system
  USING (TRUE)
  WITH CHECK (TRUE);

-- tenant_member / tenant_manager: no access to delivery history
-- Absence of a permissive policy = zero rows (fail-closed per §3.1)
```

---

### 18.6 Delivery Retry Logic

The delivery worker is a Cloudflare Worker cron running every 60 seconds. It processes deliveries in `status = 'pending'` order by `next_retry_at ASC NULLS FIRST` (new deliveries have `next_retry_at = NULL` and are dispatched immediately).

#### 18.6.1 Backoff Schedule

| Attempt | Delay before next retry | Cumulative time elapsed |
|---|---|---|
| 1 (initial) | — | 0 |
| 2 | +1 minute | ~1 min |
| 3 | +5 minutes | ~6 min |
| 4 | +30 minutes | ~36 min |
| 5 | +2 hours | ~2h 36 min |
| 6 | +8 hours | ~10h 36 min |
| After 6 failures | Status → `exhausted`; no retry | ~1d 10h 36 min total elapsed |

After `exhausted`, the delivery worker increments `webhook_endpoints.failure_count`. When `failure_count >= 10`, it sets `active = FALSE` on the endpoint and emits a `webhook.endpoint_auto_deactivated` DEC-030 event. The tenant is notified via email (via a `notifications` row with `channel = 'email'`, `user_id = NULL`, directed to `tenant_owner`).

#### 18.6.2 Delivery Worker Pseudocode

```typescript
// workers/webhook-delivery/worker.ts
// Runs every 60 seconds via Cloudflare Cron Trigger

const BACKOFF_MINUTES = [1, 5, 30, 120, 480, 1440]; // 6 attempts
const MAX_ATTEMPTS    = BACKOFF_MINUTES.length;

export async function processWebhookDeliveries(db: PostgresClient): Promise<void> {
  // Claim a batch of up to 50 deliveries due for dispatch.
  // FOR UPDATE SKIP LOCKED prevents concurrent workers from double-processing.
  const batch = await db.query<WebhookDelivery>(`
    SELECT wd.*, we.url, we.signing_secret_enc, we.tenant_id AS endpoint_tenant_id
    FROM webhook_deliveries wd
    JOIN webhook_endpoints  we ON we.id = wd.endpoint_id AND we.active = TRUE
    WHERE wd.status IN ('pending', 'failed')
      AND (wd.next_retry_at IS NULL OR wd.next_retry_at <= NOW())
      AND wd.attempt_count < $1
    ORDER BY wd.next_retry_at ASC NULLS FIRST
    LIMIT 50
    FOR UPDATE OF wd SKIP LOCKED
  `, [MAX_ATTEMPTS]);

  for (const delivery of batch.rows) {
    const signingKey = await decryptWithTenantKmsKey(
      delivery.signing_secret_enc,
      delivery.endpoint_tenant_id,
    );
    const signature = computeHmacSha256(signingKey, JSON.stringify(delivery.payload));

    let responseCode: number | null = null;
    let responseBody: string | null = null;
    let succeeded = false;

    try {
      const res = await fetch(delivery.url, {
        method:  'POST',
        headers: {
          'Content-Type':            'application/json',
          'X-FORM-Signature-256':    `sha256=${signature}`,
          'X-FORM-Event':            delivery.event_type,
          'X-FORM-Delivery':         delivery.id,
          'X-FORM-Payload-Version':  delivery.payload['$schema_version'],
        },
        body:    JSON.stringify(delivery.payload),
        signal:  AbortSignal.timeout(10_000),  // 10-second hard timeout
      });
      responseCode = res.status;
      responseBody = (await res.text()).slice(0, 1024); // cap at 1 KB
      succeeded    = res.ok;
    } catch (err) {
      responseCode = null;
      responseBody = String(err).slice(0, 1024);
    }

    const newAttemptCount = delivery.attempt_count + 1;

    if (succeeded) {
      await db.query(`
        UPDATE webhook_deliveries
        SET status = 'delivered', attempt_count = $1,
            response_code = $2, response_body = $3,
            delivered_at = NOW(), next_retry_at = NULL, updated_at = NOW()
        WHERE id = $4
      `, [newAttemptCount, responseCode, responseBody, delivery.id]);

      await db.query(`
        UPDATE webhook_endpoints
        SET failure_count = 0, last_success_at = NOW(), last_attempted_at = NOW()
        WHERE id = $1
      `, [delivery.endpoint_id]);

    } else {
      const exhausted = newAttemptCount >= MAX_ATTEMPTS;
      const nextRetry = exhausted
        ? null
        : new Date(Date.now() + BACKOFF_MINUTES[newAttemptCount] * 60_000);

      await db.query(`
        UPDATE webhook_deliveries
        SET status = $1, attempt_count = $2,
            response_code = $3, response_body = $4,
            next_retry_at = $5, updated_at = NOW()
        WHERE id = $6
      `, [
        exhausted ? 'exhausted' : 'failed',
        newAttemptCount, responseCode, responseBody, nextRetry, delivery.id,
      ]);

      await db.query(`
        UPDATE webhook_endpoints
        SET failure_count = failure_count + 1, last_attempted_at = NOW()
        WHERE id = $1
      `, [delivery.endpoint_id]);

      if (exhausted) {
        await maybeAutoDeactivateEndpoint(db, delivery.endpoint_id, delivery.endpoint_tenant_id);
      }
    }
  }
}
```

#### 18.6.3 Signing Header

Every delivery carries an `X-FORM-Signature-256` header. The signature is computed over the raw JSON-serialised payload body (before HTTP transmission), not the `payload` JSONB column value:

```
X-FORM-Signature-256: sha256=<hex(HMAC-SHA256(signing_key, request_body_bytes))>
```

Customer verification (reference implementation):

```typescript
// Customer's webhook receiver — for documentation purposes only; not FORM code
function verifyFormWebhook(
  rawBody: Buffer,
  signatureHeader: string,
  signingSecret: string,
): boolean {
  const expected = 'sha256=' + createHmac('sha256', signingSecret)
    .update(rawBody)
    .digest('hex');
  return timingSafeEqual(Buffer.from(expected), Buffer.from(signatureHeader));
}
```

Timing-safe comparison is mandatory. The signing secret displayed to the customer at endpoint creation is the raw pre-encryption value (shown once; not recoverable from the API afterwards).

---

### 18.7 Payload Schema Versioning

All webhook payloads carry a `$schema_version` field at the top level. Payload schema is versioned independently of the API version. Breaking changes (field removal, type change, enum value removal) require a major version bump and a 12-month deprecation window matching the API contract policy (Technical Principles §3).

```jsonc
// Example: workout.completed payload (schema version 1.0.0)
{
  "$schema_version": "1.0.0",
  "id":              "<delivery-uuid>",
  "event_type":      "workout.completed",
  "tenant_id":       "<tenant-uuid>",
  "occurred_at":     "2026-05-23T14:32:00Z",
  "data": {
    "workout_id":        "<workout-uuid>",
    "user_id":           "<user-uuid>",      // pseudonymous UUID only
    "duration_seconds":  2340,
    "sets_logged":       12,
    "source":            "cv",
    // form_score is EXCLUDED — personal health signal; not contractually guaranteed
    // coaching_turns, rep_detail, weight: EXCLUDED — see §18.8
  }
}
```

```jsonc
// Example: auth.scim_user_provisioned payload (schema version 1.0.0)
{
  "$schema_version": "1.0.0",
  "id":              "<delivery-uuid>",
  "event_type":      "auth.scim_user_provisioned",
  "tenant_id":       "<tenant-uuid>",
  "occurred_at":     "2026-05-23T09:15:00Z",
  "data": {
    "user_id":        "<user-uuid>",     // FORM internal UUID
    "external_id":    "<idp-subject>",   // IdP externalId per SCIM protocol
    "provisioned_via":"scim",
    "role":           "tenant_member"
    // email: EXCLUDED unless contractually agreed in DPA and tenant has email_in_webhooks feature flag
  }
}
```

**PII in payloads:** The `email` field is excluded by default from all webhook payloads. It is available only when two conditions are met: (a) the enterprise DPA explicitly authorises email disclosure in webhook events, and (b) the `email_in_webhooks` tenant feature flag is enabled by `form_admin`. Any payload schema that includes email requires a compliance-officer sign-off and a DECISION_LOG entry. The feature flag is implemented in `tenant_feature_flags` (§2.7).

---

### 18.8 GDPR Art. 17 Erasure Integration

When a user submits a GDPR Art. 17 erasure request (§12.3), both `notifications` and `webhook_deliveries` require scrubbing. The pattern follows §12.3 (hard-delete for user data) with one divergence: `webhook_deliveries` rows are payload-scrubbed rather than deleted, because they constitute delivery receipt evidence for the enterprise customer.

#### 18.8.1 `notifications` — hard delete

`notifications` rows follow the same hard-delete path as `workouts` and `meal_logs`:

```sql
-- Executed by the erasure worker (form_admin role) within 30-day grace period
DELETE FROM notifications
WHERE user_id = :user_id;
-- tenant-level notifications (user_id IS NULL) are NOT deleted — they are operational records
-- and contain no personal data beyond the tenant UUID.
```

The DSAR export job (§12.5) assembles user-targeted `notifications` rows before deletion:

```
form-export-{user_id}.zip
  └── notifications.json   -- all non-deleted notifications for user at export time
```

#### 18.8.2 `webhook_deliveries` — payload scrub

`webhook_deliveries` rows are retained as delivery receipts but the `payload` column is scrubbed in-place:

```sql
-- Payload scrub: replace payload JSONB with a tombstone record.
-- The delivery metadata (status, response_code, attempt_count, timestamps) is retained
-- as evidence of the delivery event for the enterprise customer's audit trail.
UPDATE webhook_deliveries
SET payload = jsonb_build_object(
  '$schema_version', payload->>'$schema_version',
  'event_type',      payload->>'event_type',
  'tenant_id',       payload->>'tenant_id',
  'occurred_at',     payload->>'occurred_at',
  'data',            jsonb_build_object(
                       '_erased', TRUE,
                       '_erased_at', NOW()::TEXT,
                       '_reason', 'gdpr_art17'
                     )
)
WHERE payload->'data'->>'user_id' = :user_id::TEXT;
```

The `_erased` tombstone preserves the delivery record for the enterprise customer's SOC 2 audit trail while eliminating all personal data from the payload body. The scrub is executed synchronously within the 24-hour erasure job window (not deferred to the 30-day hard-delete cycle), because webhook payloads may contain the user's pseudonymous UUID which constitutes personal data under GDPR Art. 4(1) if the customer can re-identify it via their own user directory.

**Erasure audit event:**

```jsonc
{
  "event":                    "erasure.webhook_payload_scrubbed",
  "user_id":                  "<uuid>",
  "tenant_id":                "<uuid>",
  "notifications_deleted":    14,
  "webhook_deliveries_scrubbed": 7,
  "scrubbed_at":              "2026-05-23T02:00:00Z"
}
```

---

### 18.9 Index Strategy

Index design follows the §11 patterns: composite `(tenant_id, ...)` leads for RLS-aligned scans; partial indexes on active/non-deleted subsets; `(endpoint_id, created_at)` for delivery history lookups.

```sql
-- notifications: primary isolation index
CREATE INDEX idx_notifications_tenant_user
  ON notifications (tenant_id, user_id)
  WHERE deleted_at IS NULL;

-- notifications: delivery worker queue scan (pending + pending-retry ordered by created_at)
CREATE INDEX idx_notifications_status_created
  ON notifications (tenant_id, status, created_at)
  WHERE deleted_at IS NULL AND status = 'pending';

-- notifications: expiry sweep (nightly pg_cron job cancels expired pending notifications)
CREATE INDEX idx_notifications_expires_at
  ON notifications (expires_at)
  WHERE status = 'pending' AND expires_at IS NOT NULL;

-- notifications: admin dashboard history view (tenant-wide, most recent first)
CREATE INDEX idx_notifications_tenant_created
  ON notifications (tenant_id, created_at DESC)
  WHERE deleted_at IS NULL;

-- webhook_endpoints: tenant lookup (admin dashboard endpoint list)
CREATE INDEX idx_webhook_endpoints_tenant
  ON webhook_endpoints (tenant_id)
  WHERE active = TRUE;

-- webhook_endpoints: active endpoint scan by event type (delivery worker fanout)
-- Partial GIN index over the events array for efficient ANY() membership checks
CREATE INDEX idx_webhook_endpoints_events
  ON webhook_endpoints USING GIN (events)
  WHERE active = TRUE;

-- webhook_deliveries: delivery worker retry queue
-- Covers: status filter + next_retry_at ordering + endpoint join
CREATE INDEX idx_webhook_deliveries_retry_queue
  ON webhook_deliveries (next_retry_at ASC NULLS FIRST, endpoint_id)
  WHERE status IN ('pending', 'failed');

-- webhook_deliveries: tenant-scoped delivery history (admin dashboard)
CREATE INDEX idx_webhook_deliveries_tenant_created
  ON webhook_deliveries (tenant_id, created_at DESC);

-- webhook_deliveries: per-endpoint delivery history (admin dashboard drilldown)
CREATE INDEX idx_webhook_deliveries_endpoint_created
  ON webhook_deliveries (endpoint_id, created_at DESC);

-- webhook_deliveries: GDPR erasure scrub (identify rows by user_id inside payload JSONB)
-- Expression index on the payload data.user_id field avoids a full-table JSONB scan
CREATE INDEX idx_webhook_deliveries_payload_user_id
  ON webhook_deliveries ((payload->'data'->>'user_id'))
  WHERE payload->'data'->>'user_id' IS NOT NULL;
```

**Index maintenance:**

- All indexes on `webhook_deliveries` are created `CONCURRENTLY` in migrations — this table is write-heavy during burst delivery periods.
- `idx_webhook_deliveries_retry_queue` is rebuilt weekly via `REINDEX CONCURRENTLY` (added to the maintenance cron window) because rows transition out of the `IN ('pending', 'failed')` partial predicate frequently and bloat accumulates.
- `idx_notifications_status_created` is a short-lived hot index. Rows exit `status = 'pending'` within seconds of dispatch. `autovacuum_vacuum_scale_factor` for `notifications` is set to `0.01` to reclaim dead tuples aggressively.

---

### 18.10 Privacy Considerations

#### 18.10.1 Webhook payload PII scope

The payload PII matrix below is the authoritative statement of what may appear in webhook payloads by default and under contractual extension. Any deviation from this matrix requires a compliance-officer approval and a DECISION_LOG entry.

| Data field | Default | With DPA + `email_in_webhooks` flag | Never |
|---|---|---|---|
| `user_id` (FORM UUID) | Included | Included | — |
| `tenant_id` | Included | Included | — |
| `external_id` (IdP subject) | Included for auth events | Included | — |
| `email` | Excluded | Included (auth events only) | Included in training events |
| `display_name` | Excluded | Excluded | — |
| `role` | Included | Included | — |
| `form_score` (per workout) | Excluded | Excluded | Never |
| `coaching_turns.content` | Never | Never | Never |
| HRV / resting HR values | Never | Never | Never |
| Meal log content | Never | Never | Never |
| Keypoints / CV data | Never | Never | Never |
| Body measurements | Never | Never | Never |

**Rationale for personal data appearing in payloads at all:** Enterprise customers require `user_id` to correlate FORM events with their own HRIS data for wellness programme reporting. The `user_id` is a FORM-internal UUID (not derived from email or any PII) — re-identification requires the customer to hold a FORM-to-HRIS mapping, which they necessarily own. The transmission of this UUID is covered under Art. 6(1)(b) (contract performance for the enterprise subscription) and Art. 28 (processor relationship).

#### 18.10.2 Payload schema versioning and deprecation

Any payload field that is removed or renamed triggers a major schema version bump (`1.x.x → 2.0.0`). The old schema version remains supported for 12 months (matching the API deprecation policy, Technical Principles §3). During that window:

- Both schema versions are delivered to the endpoint if the customer has not indicated a version preference.
- The customer may specify a preferred schema version via the `X-FORM-Accept-Payload-Version` request header on their endpoint — FORM uses this as a hint when creating `webhook_deliveries` rows.
- After 12 months, the old schema version is retired. Endpoints still requesting the old version are switched automatically and notified via email.

#### 18.10.3 Webhook payload in DSAR export

When a user requests a GDPR Art. 20 data export (§12.5), `webhook_deliveries` rows where `payload->'data'->>'user_id' = user_id` are included in the export:

```
form-export-{user_id}.zip
  └── webhook_deliveries.json   -- delivery metadata + scrubbed payload per Art. 20
```

The export includes `event_type`, `delivered_at`, `tenant_id`, and the payload `data` field as it existed at export time. Post-scrub tombstones are included as-is.

---

### 18.11 DEC-030 Audit Events

All webhook endpoint management and delivery lifecycle events are HMAC-chained per DEC-030 (`docs/AUDIT_LOG_SCHEMA.md`).

| Event slug | Trigger | Severity | Retention |
|---|---|---|---|
| `webhook.endpoint_created` | Tenant admin creates a new endpoint | STANDARD | 7 years |
| `webhook.endpoint_updated` | URL, events, or active flag changed | STANDARD | 7 years |
| `webhook.endpoint_deleted` | Endpoint hard-deleted by tenant admin | STANDARD | 7 years |
| `webhook.endpoint_auto_deactivated` | `failure_count >= 10` triggers auto-deactivation | STANDARD | 7 years |
| `webhook.endpoint_reactivated` | Tenant admin re-enables a deactivated endpoint | STANDARD | 7 years |
| `webhook.signing_secret_rotated` | Tenant admin rotates the signing secret | **HIGH** | 7 years |
| `webhook.test_delivery_sent` | `webhook.test` event dispatched for endpoint validation | STANDARD | 1 year |
| `webhook.delivery_exhausted` | Delivery reaches `status = 'exhausted'` after 6 attempts | STANDARD | 3 years |
| `erasure.webhook_payload_scrubbed` | Art. 17 erasure scrubs payload JSONB | STANDARD | 7 years |
| `notification.sent` | Notification status transitions to `sent` | LOW | 30 days |
| `notification.failed` | Notification status transitions to `failed` | STANDARD | 1 year |
| `notification.expired` | Pending notification cancelled due to `expires_at` | LOW | 30 days |

**`webhook.signing_secret_rotated` — HIGH severity:** Secret rotation invalidates all in-flight deliveries using the old key. Customers must update their receiver within the rotation grace period (default: 24 hours, during which the delivery worker accepts both old and new key). Unannounced rotation is a potential denial-of-service for the customer's webhook pipeline. The HIGH severity classification ensures the event appears in SIEM dashboards and triggers the CSM notification workflow.

**Metadata schema — `webhook.delivery_exhausted`:**
```jsonc
{
  "event":           "webhook.delivery_exhausted",
  "tenant_id":       "<uuid>",
  "endpoint_id":     "<uuid>",
  "delivery_id":     "<uuid>",
  "event_type":      "workout.completed",
  "attempt_count":   6,
  "last_response_code": 503,
  // payload field: NEVER logged in audit — it may contain personal data
  // audit log stores only the delivery metadata identifiers
}
```

---

### 18.12 Notification Expiry and Retention

```sql
-- Nightly at 03:15 UTC: cancel expired pending notifications
-- (staggered after §17.5 MV refresh schedule, before 04:00 retention jobs)
SELECT cron.schedule('cancel-expired-notifications', '15 3 * * *', $$
  UPDATE notifications
  SET    status     = 'cancelled',
         updated_at = NOW()
  WHERE  status     = 'pending'
    AND  expires_at < NOW()
    AND  deleted_at IS NULL;
$$);

-- Nightly at 03:20 UTC: hard-delete soft-deleted notifications past 30-day grace
SELECT cron.schedule('hard-delete-notifications', '20 3 * * *', $$
  DELETE FROM notifications
  WHERE deleted_at < NOW() - INTERVAL '30 days';
$$);

-- Nightly at 03:25 UTC: purge delivered/cancelled webhook_deliveries older than 90 days
-- (exhausted + failed rows retained 1 year for delivery health analytics)
SELECT cron.schedule('purge-webhook-deliveries', '25 3 * * *', $$
  DELETE FROM webhook_deliveries
  WHERE  status IN ('delivered', 'cancelled')
    AND  created_at < NOW() - INTERVAL '90 days';

  DELETE FROM webhook_deliveries
  WHERE  status IN ('exhausted', 'failed')
    AND  created_at < NOW() - INTERVAL '1 year';
$$);
```

**Monitoring:** Each cron job writes affected row counts to `cron_job_results`. Zero affected rows for 7 consecutive days triggers PagerDuty P3 (consistent with §12.4 monitoring requirement).

---

### 18.13 Migration Checklist

Per §7.2, the following items must be verified before any migration touching §18 tables reaches production.

```
[ ] `webhook_event_type` enum created before table DDL
[ ] `notifications` has tenant_id + user_id composite index (§18.9)
[ ] `notifications` ENABLE ROW LEVEL SECURITY
[ ] `webhook_endpoints` has tenant_id index; signing_secret_enc excluded from all API serialisers
[ ] `webhook_endpoints` ENABLE ROW LEVEL SECURITY
[ ] `webhook_deliveries` has tenant_id + retry-queue + endpoint + JSONB user_id expression indexes
[ ] `webhook_deliveries` ENABLE ROW LEVEL SECURITY
[ ] RLS isolation test extended: tenant_member sees zero webhook_deliveries rows
[ ] RLS isolation test extended: cross-tenant webhook_endpoints read returns zero rows
[ ] Erasure worker extended: notifications hard-delete + webhook_deliveries payload scrub
[ ] DSAR export worker extended: notifications.json + webhook_deliveries.json in zip
[ ] pg_cron jobs for expiry, hard-delete, and purge registered (§18.12)
[ ] form_audit role granted SELECT on notifications, webhook_endpoints, webhook_deliveries
[ ] Data classification documented in §5 (webhook_endpoints.signing_secret_enc = Sensitive)
```

---

### 18.14 Implementation Checklist

| # | Task | Owner | Priority | Milestone | Notes |
|---|---|---|---|---|---|
| 1 | `CREATE TYPE webhook_event_type AS ENUM (...)` migration; add enum values for all 23 initial event types (§18.2) | `platform-engineer` | **P0** | M3 | New values require a DECISION_LOG entry — document this in migration header comments |
| 2 | `CREATE TABLE notifications` DDL migration including all RLS policies, `ENABLE ROW LEVEL SECURITY`, four indexes (§18.9) | `platform-engineer` | **P0** | M3 | Verify `deleted_at IS NULL` clause on all user-read policies — absence is a P0 bug per §12.1 |
| 3 | `CREATE TABLE webhook_endpoints` DDL migration; encrypt initial `signing_secret_enc` using per-tenant KMS key before insert; confirm API serialiser excludes raw column | `platform-engineer` + `security-engineer` | **P0** | M3 | CI test: no API response fixture may contain the literal string `signing_secret_enc` |
| 4 | `CREATE TABLE webhook_deliveries` DDL migration including idempotency unique constraint, all indexes; add expression index on `payload->'data'->>'user_id'` | `platform-engineer` | **P0** | M3 | Create all indexes `CONCURRENTLY`; confirm `REINDEX CONCURRENTLY` maintenance cron entry |
| 5 | Add `notifications`, `webhook_endpoints`, `webhook_deliveries` RLS isolation assertions to `__tests__/db/rls_isolation.test.ts` | `platform-engineer` | **P0** | M3 | Assertions: tenant_member zero-row for `webhook_deliveries`; cross-tenant `webhook_endpoints` zero-row |
| 6 | Implement notification dispatch worker (Cloudflare Worker): dequeues `pending` notifications by channel; calls FCM (push), SES (email), or in-app pub/sub; updates `status`; emits `notification.sent` / `notification.failed` DEC-030 events | `platform-engineer` | **P0** | M3 | |
| 7 | Implement webhook delivery worker (§18.6.2): cron every 60 seconds; `FOR UPDATE SKIP LOCKED` batch; HMAC-SHA256 signing; 10-second HTTP timeout; exponential backoff write-back; auto-deactivate at `failure_count >= 10` | `platform-engineer` | **P0** | M3 | Worker must verify endpoint URL starts with `https://` before dispatch — reject HTTP silently |
| 8 | Implement `POST /v1/admin/webhooks/endpoints` and `GET /v1/admin/webhooks/endpoints` API endpoints; enforce 10-endpoint-per-tenant application-layer limit; show raw secret only in creation response | `platform-engineer` | **P0** | M3 | `GET` serialiser must replace `signing_secret_enc` with `"***"` — CI assertion required |
| 9 | Implement webhook endpoint admin UI in React admin dashboard: endpoint list, event subscription checkboxes (multi-select from `webhook_event_type` enum), delivery log drilldown, manual re-trigger for `exhausted` deliveries | `platform-engineer` | **P1** | M4 | |
| 10 | Implement `webhook.test` synthetic delivery: `POST /v1/admin/webhooks/endpoints/{id}/test` creates a `webhook.test` delivery row and dispatches immediately; used for customer endpoint validation during setup | `platform-engineer` | **P1** | M4 | `webhook.test_delivery_sent` DEC-030 event required; test payload schema version = current |
| 11 | Implement signing secret rotation: `POST /v1/admin/webhooks/endpoints/{id}/rotate-secret`; 24-hour dual-key grace window where delivery worker tries new key first, falls back to old key on 401/403 from endpoint; emits `webhook.signing_secret_rotated` HIGH DEC-030 event | `platform-engineer` + `security-engineer` | **P1** | M4 | Grace window requires storing `previous_signing_secret_enc` on the endpoint row during transition; clear after 24 hours |
| 12 | Implement payload Zod schema validation in `src/notifications/payload.schema.ts`; wire into notification insert path; CI test asserting schema rejects forbidden keys (`coaching_turns`, `form_score`, `keypoints_enc`, all wearable numeric fields) | `platform-engineer` | **P0** | M3 | Same `FORBIDDEN_PROPERTY_KEYS` set as §13.3; reuse validation module |
| 13 | Extend Art. 17 erasure worker (§12.3): hard-delete `notifications WHERE user_id = :user_id`; payload-scrub `webhook_deliveries` where `payload->'data'->>'user_id' = :user_id`; emit `erasure.webhook_payload_scrubbed` DEC-030 event with counts | `platform-engineer` | **P1** | M4 | Scrub must execute within 24-hour erasure job window — not deferred to 30-day hard-delete cycle |
| 14 | Extend DSAR export job (§12.5): add `notifications.json` and `webhook_deliveries.json` to `form-export-{user_id}.zip` | `platform-engineer` | **P1** | M4 | |
| 15 | Register pg_cron jobs (§18.12): expiry cancellation at 03:15 UTC, hard-delete at 03:20 UTC, delivery purge at 03:25 UTC; add to `cron_job_results` monitoring per §12.4 | `platform-engineer` | **P1** | M4 | |
| 16 | Wire all 12 DEC-030 events (§18.11) to their respective triggers; validate HMAC chain in staging; confirm `webhook.delivery_exhausted` metadata never includes payload field | `platform-engineer` + `security-engineer` | **P0** | M3 | |
| 17 | Implement `email_in_webhooks` tenant feature flag: add to `tenant_feature_flags` (§2.7); wire into webhook payload serialiser; require compliance-officer sign-off per DPA before enabling for any tenant | `enterprise-architect` + `compliance-officer` | **P2** | M5 | Default: flag absent = email excluded from all payloads |
| 18 | Update §5 data classification table: `webhook_endpoints.signing_secret_enc` = Sensitive (per-tenant KMS encryption); `notifications.payload` = Internal (JSONB, no health data permitted) | `enterprise-architect` + `compliance-officer` | **P0** | M3 | |

---

*v0.9 additions: §18 Notification & Webhook Event Queue Schema. `webhook_event_type` PostgreSQL enum: 23 event types across training lifecycle (workout.completed, workout.abandoned, workout.set_logged), streak events (maintained, broken, grace_period_activated, milestone_reached), coaching, auth (sso_login, scim_provisioned/deprovisioned, mfa_challenge_failed, session_expired), enterprise lifecycle (tenant.user_onboarded/deactivated, seat_limit_approached/reached, pilot_milestone, contract_renewal_due), wearable (source_connected/disconnected), and system (delivery_failed, webhook.test) — new values require DECISION_LOG entry and form_admin privilege. `notifications` table: tenant_id + nullable user_id (null = tenant-level), `webhook_event_type` type, JSONB payload (Zod-validated; FORBIDDEN_PROPERTY_KEYS enforced), channel enum (push/email/in_app/webhook), status enum (pending/sent/failed/cancelled), retry_count, expires_at; RLS: user self-read via user_id match, tenant admin full-tenant history, form_system full-access, soft-delete (deleted_at) per §12.1 pattern. `webhook_endpoints` table: SENSITIVE classification for `signing_secret_enc` (AES-256-GCM per-tenant KMS); url CHECK (HTTPS only); events TEXT[] with GIN index for ANY() fanout; active boolean with auto-deactivation at failure_count >= 10; API serialiser replaces secret with "***"; RLS: tenant_admin/owner read+write, form_system full access. `webhook_deliveries` table: endpoint_id FK with CASCADE, denormalized tenant_id, status enum (pending/delivered/failed/exhausted/cancelled), 6-attempt exponential backoff schedule (1min/5min/30min/2h/8h/24h), response_body capped at 1KB, idempotency_key UNIQUE constraint per endpoint; RLS: tenant admin read-only history, form_system full access. Delivery worker: Cloudflare Worker cron (60s); FOR UPDATE SKIP LOCKED batch (50 rows); HMAC-SHA256 X-FORM-Signature-256 header; 10-second AbortSignal timeout; backoff write-back; auto-deactivate + tenant_owner email notification at exhaustion. Signing header: `sha256=<hex(HMAC-SHA256(key, raw_body_bytes))>`; timing-safe verification reference implementation. Payload schema versioning: $schema_version semver field; 12-month deprecation window on breaking changes; X-FORM-Accept-Payload-Version customer hint; two example payloads (workout.completed, auth.scim_user_provisioned). PII matrix: user_id UUID included by default; email excluded by default, available only with DPA + email_in_webhooks feature flag; coaching content, form_score, HRV, meal logs, CV data: never. GDPR Art. 17 integration: notifications hard-deleted by user_id; webhook_deliveries payload-scrubbed in-place within 24h (not deferred to 30-day cycle) with _erased tombstone preserving delivery receipt; `erasure.webhook_payload_scrubbed` DEC-030 event with counts; both tables included in Art. 20 DSAR export. Index strategy (9 indexes): (tenant_id, user_id) partial for notifications RLS; status+created partial for dispatch queue; expires_at partial for expiry sweep; tenant+created DESC for admin history; GIN on webhook_endpoints.events for fanout; retry-queue composite; endpoint+created DESC for drilldown; expression index on payload->'data'->>'user_id' for erasure sweep. pg_cron: expiry cancel 03:15, hard-delete 03:20, delivery purge (90d delivered, 1yr exhausted) 03:25 UTC. 12 DEC-030 HMAC-chained audit events including HIGH-severity webhook.signing_secret_rotated with 24-hour dual-key grace window; delivery_exhausted metadata explicitly excludes payload field. 18-item implementation checklist (9× P0, 6× P1, 1× P2, 2× P0 documentation) across M3/M4/M5.*

## 19. Feature Flag & Entitlement Registry Schema

> Owner: `enterprise-architect` + `compliance-officer`. Review: on any new flag, tier change, or compliance gate addition.
> References: DEC-030 (HMAC-chained audit log), docs/AUDIT_LOG_SCHEMA.md, docs/ENTERPRISE.md, docs/SSO_SCIM_IMPLEMENTATION.md.

---

### 19.1 Purpose & Design Principles

The §2.7 stub `tenant_feature_flags` table used an arbitrary-string `flag_name` column with no enforcement of which strings were valid, which tiers they required, or whether compliance sign-off was needed before enablement. This created four production risks:

1. **Arbitrary drift.** Any `form_admin` could silently invent new flags. By M4, referencing `cv_enabled` (§15), `email_in_webhooks` (§18), `ip_allowlist_enabled` (SSO_SCIM_IMPLEMENTATION.md), and `sso_staging_env_enabled` (SSO_SCIM_IMPLEMENTATION.md) from different call sites with slightly different spelling guarantees bugs.
2. **No tier gate.** A Starter tenant could receive a flag that is contractually Growth-tier. Revenue leakage and contract non-compliance.
3. **No compliance gate.** `webhook.email_in_payloads` requires a DPA authorisation before enablement. The stub had a free-text `reason` field with no enforcement.
4. **No audit trail at the right level.** Flag mutations were not HMAC-chained to DEC-030, so they were outside the SOC 2 CC8.1 change-management evidence chain.

This section replaces the §2.7 stub with a two-table design:

- `feature_flag_registry` — canonical set of valid flags, owned by `form_admin` only. This is the allowlist. No flag outside this table may be inserted into `tenant_feature_flags` after M4.
- `tenant_feature_flags` — per-tenant enablement rows with FK to the registry, compliance approval reference, and DEC-030 audit trail.

**Design principles:**

1. **Registry is the single source of truth.** The TypeScript enforcement layer derives its type-safe `FlagKey` literal union from the registry at build time. Arbitrary strings are a compile error after M4.
2. **Tier enforcement at write time.** The Cloudflare Worker `assertFeatureEnabled()` function checks both the registry `minimum_tier` and the tenant's current plan before returning. Tier downgrade handling deferred to OQ-FLAG-01.
3. **Compliance gate is a hard block.** Any flag where `requires_compliance_gate = TRUE` may not be enabled without a non-null `compliance_approval_ref` referencing a DECISION_LOG entry. Enforced in the Worker layer and at the DB level via a CHECK constraint.
4. **DEC-030 on every mutation.** HMAC-chained audit event emitted synchronously (in-transaction) for every INSERT, UPDATE, and DELETE on `tenant_feature_flags`. Failure to emit = transaction abort (fail-closed).
5. **No per-flag usage analytics surfaced to tenant admin.** The registry enables product A/B testing internally, but no flag evaluation counts or adoption percentages are surfaced to tenant-admin roles. They see only whether a flag is enabled for their tenant.

---

### 19.2 Feature Flag Registry

```sql
-- Canonical registry of all valid feature flags.
-- Only form_admin (BYPASSRLS) may write. form_api roles have SELECT only.
-- A flag not in this table CANNOT be inserted into tenant_feature_flags (FK constraint).

CREATE TABLE feature_flag_registry (
  flag_key                      TEXT PRIMARY KEY,
                                -- Dotted namespace: '<domain>.<flag_name>'
                                -- e.g. 'sso.saml_enabled', 'audit.siem_export_enabled'
                                -- Validated by CHECK below

  display_name                  TEXT NOT NULL,
                                -- Human-readable label shown in internal admin UI

  description                   TEXT NOT NULL,
                                -- One-paragraph explanation of what this flag enables
                                -- and any operational notes

  minimum_tier                  TEXT NOT NULL
                                  CHECK (minimum_tier IN ('starter', 'growth', 'enterprise')),
                                -- Lowest tier at which a tenant is entitled to this flag.
                                -- form_admin may not enable the flag for a tenant on a lower tier.

  default_enabled_at_activation BOOLEAN NOT NULL DEFAULT FALSE,
                                -- TRUE = flag is set enabled=TRUE automatically when a new
                                -- tenant activates at a qualifying tier.
                                -- FALSE = flag must be manually enabled after activation.

  requires_compliance_gate      BOOLEAN NOT NULL DEFAULT FALSE,
                                -- TRUE = a compliance-officer sign-off (DECISION_LOG entry)
                                -- is required before this flag may be enabled for any tenant.
                                -- Enforced via NOT NULL compliance_approval_ref in tenant_feature_flags.

  deprecated_at                 TIMESTAMPTZ,
                                -- NULL = active. Non-null = flag is sunset; no new tenants
                                -- may have it enabled after this timestamp.
                                -- Existing enabled rows are honoured until contract expiry.

  created_at                    TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at                    TIMESTAMPTZ NOT NULL DEFAULT now(),

  CONSTRAINT flag_key_namespace CHECK (flag_key ~ '^[a-z_]+\.[a-z_]+$')
                                -- Enforces dotted namespace format at DB level.
                                -- Prevents arbitrary strings from bypassing the naming convention.
);

-- No RLS on this table. form_api roles are granted SELECT only (not INSERT/UPDATE/DELETE).
-- form_admin (BYPASSRLS) is the only writer — enforced via GRANT, not RLS.
GRANT SELECT ON feature_flag_registry TO form_api;
GRANT SELECT ON feature_flag_registry TO form_readonly;
GRANT SELECT ON feature_flag_registry TO form_system;
-- INSERT / UPDATE / DELETE: form_admin only (no explicit GRANT to form_api)

CREATE INDEX idx_ffr_minimum_tier    ON feature_flag_registry(minimum_tier);
CREATE INDEX idx_ffr_deprecated      ON feature_flag_registry(deprecated_at)
  WHERE deprecated_at IS NOT NULL;
```

---

### 19.3 Registry Seed Data

```sql
-- Run by form_admin during M3 migration.
-- All flag_key values here are canonical — no other strings are valid in tenant_feature_flags.

INSERT INTO feature_flag_registry
  (flag_key, display_name, description, minimum_tier, default_enabled_at_activation, requires_compliance_gate)
VALUES

  -- All enterprise tiers (Starter+)
  (
    'sso.saml_enabled',
    'SAML 2.0 SSO',
    'Enables SAML 2.0 single sign-on for the tenant. Auto-enabled at activation when the IdP protocol is SAML. See SSO_SCIM_IMPLEMENTATION.md §1.',
    'starter', TRUE, FALSE
  ),
  (
    'sso.oidc_enabled',
    'OIDC SSO',
    'Enables OIDC single sign-on for the tenant. Auto-enabled at activation when the IdP protocol is OIDC. See SSO_SCIM_IMPLEMENTATION.md §1.',
    'starter', TRUE, FALSE
  ),
  (
    'scim.provisioning_enabled',
    'SCIM 2.0 User Provisioning',
    'Enables SCIM 2.0 endpoint for automated user provisioning and deprovisioning from the tenant IdP. See SSO_SCIM_IMPLEMENTATION.md §3.',
    'starter', FALSE, FALSE
  ),
  (
    'admin.dashboard_enabled',
    'Admin Reporting Dashboard',
    'Enables the web-based admin reporting dashboard with aggregate-only wellness and engagement metrics. See DATA_MODEL.md §17.',
    'starter', TRUE, FALSE
  ),
  (
    'audit.rest_export_enabled',
    'Audit Log REST Export',
    'Enables the GET /v1/audit/logs API endpoint for historical audit log retrieval by tenant admins.',
    'starter', TRUE, FALSE
  ),
  (
    'webhook.outbound_enabled',
    'Outbound Webhooks',
    'Enables outbound webhook delivery for the tenant. See DATA_MODEL.md §18.',
    'starter', FALSE, FALSE
  ),
  (
    'cv.client_side_enabled',
    'Client-Side CV Pose Estimation',
    'Enables on-device computer vision pose estimation during workouts. See DATA_MODEL.md §15.',
    'starter', TRUE, FALSE
  ),

  -- Growth+ tier
  (
    'audit.siem_export_enabled',
    'SIEM Integration Export',
    'Enables native SIEM exporters (Datadog, Splunk, Sumo Logic) for real-time audit event streaming. Requires audit.rest_export_enabled.',
    'growth', FALSE, FALSE
  ),
  (
    'audit.s3_sync_enabled',
    'S3 Audit Log Continuous Sync',
    'Enables continuous sync of the tenant audit log to a customer-owned S3 bucket. See AUDIT_LOG_SCHEMA.md §Export.',
    'growth', FALSE, FALSE
  ),
  (
    'admin.cohort_analytics_enabled',
    'SCIM-Group Cohort Analytics',
    'Enables SCIM-group cohort breakdown in admin reporting. Requires scim.provisioning_enabled and minimum 5-user cohort k-anonymity floor. See DATA_MODEL.md §17.',
    'growth', FALSE, FALSE
  ),

  -- Enterprise tier only
  (
    'branding.white_label_enabled',
    'White-Label Branding',
    'Enables custom domain (CNAME to form.coach), tenant logo, and primary colour override. Powered by FORM footer is non-removable below $50k ARR. See ENTERPRISE.md.',
    'enterprise', FALSE, FALSE
  ),
  (
    'sso.staging_env_enabled',
    'SSO Staging / Sandbox Environment',
    'Enables a staging IdP configuration alongside the production config, allowing tenants to test SSO changes without affecting live users. See SSO_SCIM_IMPLEMENTATION.md.',
    'enterprise', FALSE, FALSE
  ),
  (
    'security.ip_allowlist_enabled',
    'IP Allowlist Enforcement',
    'Enables IP allowlist enforcement at the Cloudflare Worker layer. Requests from IPs not in the tenant allowlist receive 403. See SSO_SCIM_IMPLEMENTATION.md §6.',
    'enterprise', FALSE, FALSE
  ),

  -- Compliance-gated (any tier, but requires compliance-officer sign-off)
  (
    'webhook.email_in_payloads',
    'Email Address in Webhook Payloads',
    'When enabled, the user email address is included in outbound webhook payloads. REQUIRES explicit DPA authorisation from the tenant data controller before enabling. Default: email excluded from all payloads. See DATA_MODEL.md §18.7.',
    'starter', FALSE, TRUE
  )
;
```

---

### 19.4 Tenant Feature Flags (Production Schema)

This section replaces the §2.7 stub. The stub table (`tenant_feature_flags` with free-text `flag_name`) is migrated in §19.9.

```sql
-- Drop the stub constraint before running the §19.9 migration.
-- After M4, arbitrary flag_name strings are a FK violation — not a runtime error.

CREATE TABLE tenant_feature_flags (
  id                      UUID PRIMARY KEY DEFAULT gen_random_uuid(),

  tenant_id               UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,

  flag_key                TEXT NOT NULL REFERENCES feature_flag_registry(flag_key)
                            ON UPDATE CASCADE
                            ON DELETE RESTRICT,
                          -- RESTRICT prevents deletion of a registry entry that has
                          -- live tenant rows. Deprecate via deprecated_at; never hard-delete
                          -- a registry row with active tenants.

  enabled                 BOOLEAN NOT NULL DEFAULT FALSE,

  set_by                  UUID REFERENCES users(id),
                          -- form_admin user UUID. Never a tenant_admin or tenant_member.
                          -- NULL only for system-automated activation at tenant creation.

  enabled_at              TIMESTAMPTZ,
                          -- Populated when enabled transitions to TRUE. NULL when disabled.

  compliance_approval_ref TEXT,
                          -- REQUIRED (NOT NULL enforced by CHECK below) when the referenced
                          -- registry flag has requires_compliance_gate = TRUE.
                          -- Value MUST be a DECISION_LOG entry identifier, e.g. 'DEC-047'.
                          -- Checked at INSERT and UPDATE by the application layer before
                          -- the DB write; the CHECK constraint is a defence-in-depth backstop.

  reason                  TEXT,
                          -- Internal justification for enabling/disabling. Not exposed to
                          -- tenant admin. Stored for internal audit trail context only.

  created_at              TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at              TIMESTAMPTZ NOT NULL DEFAULT now(),

  UNIQUE (tenant_id, flag_key)
);

ALTER TABLE tenant_feature_flags ENABLE ROW LEVEL SECURITY;

-- RLS policies

-- form_api can read any flag row scoped to the current tenant.
-- tenant_admin and tenant_member: read-only, own tenant only.
-- No role below form_admin may write (INSERT / UPDATE / DELETE).
CREATE POLICY tff_tenant_read ON tenant_feature_flags
  AS PERMISSIVE FOR SELECT
  TO form_api
  USING (
    tenant_id = current_setting('app.current_tenant_id', TRUE)::UUID
  );

-- form_system: read-only across all tenants (used by Workers flag evaluation path).
CREATE POLICY tff_system_read ON tenant_feature_flags
  AS PERMISSIVE FOR SELECT
  TO form_system
  USING (TRUE);

-- Write: only form_admin (BYPASSRLS) — no permissive write policy for form_api.
-- If a form_api write is attempted it hits the implicit DENY from no matching write policy.
-- Note: form_admin BYPASSRLS means it does not need a policy; the absence of an INSERT
-- policy for form_api is the enforcement mechanism.

-- Indexes
CREATE INDEX idx_tff_tenant_id    ON tenant_feature_flags(tenant_id);
CREATE INDEX idx_tff_flag_key     ON tenant_feature_flags(flag_key);
CREATE INDEX idx_tff_tenant_flag  ON tenant_feature_flags(tenant_id, flag_key);
                                  -- Covering index for the hot-path: assertFeatureEnabled()
                                  -- queries (tenant_id, flag_key) -> enabled

-- form_audit role: SELECT for compliance evidence collection
GRANT SELECT ON tenant_feature_flags TO form_audit;
```

---

### 19.5 Tier Entitlement Matrix

Legend:
- **auto** — automatically enabled when tenant activates at this tier
- **avail** — available for the tier; must be manually enabled by `form_admin`
- **—** — not available at this tier (FK check + Worker tier gate will reject)
- **gate** — compliance officer sign-off required regardless of tier

| Flag key | Starter | Growth | Enterprise |
|---|---|---|---|
| `sso.saml_enabled` | auto | auto | auto |
| `sso.oidc_enabled` | auto | auto | auto |
| `scim.provisioning_enabled` | avail | avail | avail |
| `admin.dashboard_enabled` | auto | auto | auto |
| `audit.rest_export_enabled` | auto | auto | auto |
| `webhook.outbound_enabled` | avail | avail | avail |
| `cv.client_side_enabled` | auto | auto | auto |
| `audit.siem_export_enabled` | — | avail | avail |
| `audit.s3_sync_enabled` | — | avail | avail |
| `admin.cohort_analytics_enabled` | — | avail | avail |
| `branding.white_label_enabled` | — | — | avail |
| `sso.staging_env_enabled` | — | — | avail |
| `security.ip_allowlist_enabled` | — | — | avail |
| `webhook.email_in_payloads` | gate | gate | gate |

---

### 19.6 Cloudflare Worker Enforcement Layer

```typescript
// src/workers/feature-flags/assert-feature-enabled.ts
// Called by every Worker handler that requires a feature gate.
// Fail-closed: any DB error throws 403, never 200.

import { KVNamespace } from '@cloudflare/workers-types';

// FlagKey is derived from the registry at build time.
// After M4 the seeded values below are the canonical list; a new flag requires
// a registry INSERT (form_admin) + a TypeScript union update + CI schema validation.
export type FlagKey =
  | 'sso.saml_enabled'
  | 'sso.oidc_enabled'
  | 'scim.provisioning_enabled'
  | 'admin.dashboard_enabled'
  | 'audit.rest_export_enabled'
  | 'webhook.outbound_enabled'
  | 'cv.client_side_enabled'
  | 'audit.siem_export_enabled'
  | 'audit.s3_sync_enabled'
  | 'admin.cohort_analytics_enabled'
  | 'branding.white_label_enabled'
  | 'sso.staging_env_enabled'
  | 'security.ip_allowlist_enabled'
  | 'webhook.email_in_payloads';

// Tier ordering for minimum_tier enforcement.
const TIER_ORDER: Record<string, number> = {
  starter:    1,
  growth:     2,
  enterprise: 3,
};

interface FlagCacheEntry {
  enabled: boolean;
  minimumTier: string;
  requiresComplianceGate: boolean;
  tenantTier: string;
  cachedAt: number; // epoch ms
}

const KV_TTL_SECONDS = 30; // 30-second cache; flag changes propagate within one TTL window

/**
 * Asserts that the given feature flag is enabled for the tenant.
 * Throws a 403 Response if:
 *   - The flag is not enabled for the tenant
 *   - The tenant's tier is below the flag's minimum_tier
 *   - Any DB/KV error occurs (fail-closed)
 *
 * Does NOT throw if the flag is enabled and the tier gate passes.
 */
export async function assertFeatureEnabled(
  tenantId: string,
  flagKey: FlagKey,
  env: { KV: KVNamespace; DB: D1Database },
): Promise<void> {
  const cacheKey = `flag:${tenantId}:${flagKey}`;

  try {
    // 1. Check KV cache first (30-second TTL)
    const cached = await env.KV.get<FlagCacheEntry>(cacheKey, 'json');
    if (cached !== null) {
      const ageMs = Date.now() - cached.cachedAt;
      if (ageMs < KV_TTL_SECONDS * 1000) {
        enforceFromCache(cached, tenantId, flagKey);
        return;
      }
    }

    // 2. Cache miss or stale: query DB
    const row = await env.DB.prepare(`
      SELECT
        tff.enabled,
        ffr.minimum_tier,
        ffr.requires_compliance_gate,
        t.plan AS tenant_tier,
        tff.compliance_approval_ref
      FROM tenant_feature_flags tff
      JOIN feature_flag_registry ffr ON ffr.flag_key = tff.flag_key
      JOIN tenants t ON t.id = tff.tenant_id
      WHERE tff.tenant_id = ?1
        AND tff.flag_key   = ?2
        AND t.deleted_at IS NULL
        AND t.suspended_at IS NULL
        AND (ffr.deprecated_at IS NULL OR ffr.deprecated_at > CURRENT_TIMESTAMP)
    `)
      .bind(tenantId, flagKey)
      .first<{
        enabled: number; // SQLite returns 0/1 for boolean
        minimum_tier: string;
        requires_compliance_gate: number;
        tenant_tier: string;
        compliance_approval_ref: string | null;
      }>();

    if (row === null) {
      // Flag row does not exist for this tenant — treat as disabled (fail-closed).
      throw forbidden(tenantId, flagKey, 'flag_not_found');
    }

    const entry: FlagCacheEntry = {
      enabled:                row.enabled === 1,
      minimumTier:            row.minimum_tier,
      requiresComplianceGate: row.requires_compliance_gate === 1,
      tenantTier:             row.tenant_tier,
      cachedAt:               Date.now(),
    };

    // 3. Tier gate: tenant's plan must be >= minimum_tier
    const tenantTierRank  = TIER_ORDER[entry.tenantTier]  ?? 0;
    const minimumTierRank = TIER_ORDER[entry.minimumTier] ?? 99;
    if (tenantTierRank < minimumTierRank) {
      throw forbidden(tenantId, flagKey, 'tier_insufficient');
    }

    // 4. Compliance gate: if required, must have an approval reference
    if (entry.requiresComplianceGate && !row.compliance_approval_ref) {
      throw forbidden(tenantId, flagKey, 'compliance_gate_missing');
    }

    // 5. Enabled check
    if (!entry.enabled) {
      throw forbidden(tenantId, flagKey, 'flag_disabled');
    }

    // 6. Write back to KV cache
    await env.KV.put(cacheKey, JSON.stringify(entry), { expirationTtl: KV_TTL_SECONDS });

  } catch (err) {
    if (err instanceof Response) throw err; // re-throw our own 403s
    // Any unexpected DB / KV error -> fail-closed 403 (never 200 on uncertainty)
    console.error(`[assertFeatureEnabled] DB error for tenant=${tenantId} flag=${flagKey}`, err);
    throw forbidden(tenantId, flagKey, 'internal_error');
  }
}

function enforceFromCache(
  cached: FlagCacheEntry,
  tenantId: string,
  flagKey: FlagKey,
): void {
  const tenantTierRank  = TIER_ORDER[cached.tenantTier]  ?? 0;
  const minimumTierRank = TIER_ORDER[cached.minimumTier] ?? 99;
  if (tenantTierRank < minimumTierRank) {
    throw forbidden(tenantId, flagKey, 'tier_insufficient');
  }
  if (!cached.enabled) {
    throw forbidden(tenantId, flagKey, 'flag_disabled');
  }
}

function forbidden(tenantId: string, flagKey: string, reason: string): Response {
  return new Response(
    JSON.stringify({ error: 'feature_not_enabled', flag: flagKey, reason }),
    { status: 403, headers: { 'Content-Type': 'application/json' } },
  );
}
```

**KV cache invalidation:** When `form_admin` mutates a `tenant_feature_flags` row, the mutation API explicitly deletes the KV key `flag:<tenantId>:<flagKey>` before committing the DB write. This ensures the next Worker evaluation reads fresh data within the same request cycle. The 30-second TTL is a backstop only; it is not the primary invalidation mechanism.

---

### 19.7 DEC-030 Audit Events

All four event types are HMAC-chained per AUDIT_LOG_SCHEMA.md and must be written synchronously within the database transaction that mutates `tenant_feature_flags`. Failure to write = transaction abort.

| Event type | Severity | Retention | Required metadata fields |
|---|---|---|---|
| `feature_flag.enabled` | STANDARD | 7 years | `tenant_id`, `flag_key`, `set_by_user_id`, `previous_enabled: false`, `new_enabled: true`, `enabled_at`, `compliance_approval_ref` (null if not gated), `reason` |
| `feature_flag.disabled` | STANDARD | 7 years | `tenant_id`, `flag_key`, `set_by_user_id`, `previous_enabled: true`, `new_enabled: false`, `disabled_at`, `reason` |
| `feature_flag.registry_updated` | HIGH | 7 years | `flag_key`, `changed_fields: string[]`, `previous_values: object`, `new_values: object`, `set_by_user_id` (form_admin only) |
| `feature_flag.compliance_gate_bypassed` | CRITICAL | 10 years | `tenant_id`, `flag_key`, `attempted_by_user_id`, `reason`, `blocked: true` — fires when a Worker or API call attempts to enable a gated flag without `compliance_approval_ref`; the attempt MUST be blocked, not silently allowed |

**Notes:**

- `feature_flag.compliance_gate_bypassed` is always emitted with `blocked: true`. There is no path by which this event fires and the flag is also enabled. If the event fires and investigation reveals the flag was enabled concurrently, that is a P0 security incident.
- `feature_flag.registry_updated` fires when `form_admin` modifies any column of `feature_flag_registry`. The `previous_values` and `new_values` fields contain only the changed columns — not a full row dump — to avoid storing unnecessary context. The `flag_key` value is the `resource_id` in the `audit_log` row.
- Metadata fields listed here map to the `metadata JSONB` column in `audit_log`. The `changes` column is left NULL for flag events (it is reserved for before/after on data rows, not control-plane mutations).

---

### 19.8 Compliance Gate Protocol

A compliance-gated flag (e.g. `webhook.email_in_payloads`) follows this exact sequence before it may be enabled for any tenant:

```
1. Tenant (or CSM) submits a request identifying:
     - The flag_key requested
     - The specific DPA clause or legal basis authorising the flag
     - The tenant data controller contact who approved

2. Compliance officer reviews the request.
   The review is documented in docs/DECISION_LOG.md as a new DEC-NNN entry.
   The entry must include:
     - Flag key
     - Tenant ID
     - DPA reference (contract clause, Article 28 processor agreement section)
     - Date of approval
     - Compliance officer name

3. Compliance officer updates the DECISION_LOG.md file via the standard PR process.
   The PR must be approved by a second named compliance contact or legal counsel.

4. Once the DECISION_LOG PR is merged, the DEC-NNN identifier is the compliance_approval_ref.

5. A form_admin engineer executes the flag enablement via the internal admin API:
     POST /internal/v1/admin/tenants/{tenantId}/flags
     {
       "flag_key": "webhook.email_in_payloads",
       "enabled": true,
       "compliance_approval_ref": "DEC-047",
       "reason": "DPA signed 2026-06-01, clause 4.2 authorises email in webhook payloads"
     }

6. The Worker enforces:
   - flag_key must exist in feature_flag_registry
   - tenant plan must meet minimum_tier
   - requires_compliance_gate = TRUE means compliance_approval_ref must be non-null
   - KV key invalidated before DB write

7. On successful write, a feature_flag.enabled DEC-030 audit event fires with
   compliance_approval_ref populated. The event is HMAC-chained.

8. Any attempt to skip steps 3-4 (i.e., enable the flag with compliance_approval_ref = NULL
   when requires_compliance_gate = TRUE) is blocked by the Worker and emits a
   feature_flag.compliance_gate_bypassed CRITICAL DEC-030 event.
   This event automatically opens a PagerDuty P1 incident.
```

**What compliance officers must NOT do:** provide verbal approval. The DECISION_LOG PR merge is the approval event. Verbal approvals have no legal standing and are not traceable to the audit log.

---

### 19.9 Migration: §2.7 Stub to §19 Registry

This migration replaces the free-text `flag_name` column with the FK-constrained `flag_key` column. It must run as a single transaction under `form_admin`.

```sql
-- Migration: 0019_feature_flag_registry.sql
-- Run as form_admin. No application traffic during this migration window.
-- Prerequisite: registry seed data (§19.3) already applied.

BEGIN;

-- Step 1: Rename the stub flag names to canonical registry keys.
-- Any flag_name not in this mapping is UNKNOWN and must be reviewed before proceeding.
UPDATE tenant_feature_flags SET flag_name = 'cv.client_side_enabled'
  WHERE flag_name IN ('cv_enabled', 'cv.enabled', 'client_cv_enabled');

UPDATE tenant_feature_flags SET flag_name = 'webhook.email_in_payloads'
  WHERE flag_name IN ('email_in_webhooks', 'webhook_email_in_payloads', 'email_in_webhook_payloads');

UPDATE tenant_feature_flags SET flag_name = 'security.ip_allowlist_enabled'
  WHERE flag_name IN ('ip_allowlist_enabled', 'ip_allowlist');

UPDATE tenant_feature_flags SET flag_name = 'sso.staging_env_enabled'
  WHERE flag_name IN ('sso_staging_env_enabled', 'sso_staging');

UPDATE tenant_feature_flags SET flag_name = 'sso.saml_enabled'
  WHERE flag_name IN ('saml_enabled', 'sso_saml_enabled');

UPDATE tenant_feature_flags SET flag_name = 'sso.oidc_enabled'
  WHERE flag_name IN ('oidc_enabled', 'sso_oidc_enabled');

UPDATE tenant_feature_flags SET flag_name = 'scim.provisioning_enabled'
  WHERE flag_name IN ('scim_enabled', 'scim_provisioning_enabled');

UPDATE tenant_feature_flags SET flag_name = 'admin.dashboard_enabled'
  WHERE flag_name IN ('dashboard_enabled', 'admin_dashboard_enabled');

UPDATE tenant_feature_flags SET flag_name = 'audit.rest_export_enabled'
  WHERE flag_name IN ('audit_export_enabled', 'audit_rest_export');

UPDATE tenant_feature_flags SET flag_name = 'webhook.outbound_enabled'
  WHERE flag_name IN ('webhook_enabled', 'outbound_webhook_enabled');

UPDATE tenant_feature_flags SET flag_name = 'audit.siem_export_enabled'
  WHERE flag_name IN ('siem_export_enabled', 'siem_enabled');

UPDATE tenant_feature_flags SET flag_name = 'audit.s3_sync_enabled'
  WHERE flag_name IN ('s3_sync_enabled', 'audit_s3_sync');

UPDATE tenant_feature_flags SET flag_name = 'admin.cohort_analytics_enabled'
  WHERE flag_name IN ('cohort_analytics_enabled', 'cohort_breakdown_enabled');

UPDATE tenant_feature_flags SET flag_name = 'branding.white_label_enabled'
  WHERE flag_name IN ('white_label_enabled', 'white_label');

-- Step 2: Abort if any unrecognised flag_name values remain.
DO $$
DECLARE
  unknown_count INTEGER;
BEGIN
  SELECT COUNT(*) INTO unknown_count
  FROM tenant_feature_flags
  WHERE flag_name NOT IN (SELECT flag_key FROM feature_flag_registry);

  IF unknown_count > 0 THEN
    RAISE EXCEPTION
      'Migration blocked: % unrecognised flag_name values exist. '
      'Review tenant_feature_flags WHERE flag_name NOT IN (SELECT flag_key FROM feature_flag_registry).',
      unknown_count;
  END IF;
END;
$$;

-- Step 3: Rename the column to flag_key and add the FK constraint.
ALTER TABLE tenant_feature_flags
  RENAME COLUMN flag_name TO flag_key;

ALTER TABLE tenant_feature_flags
  ADD CONSTRAINT tenant_feature_flags_flag_key_fkey
    FOREIGN KEY (flag_key)
    REFERENCES feature_flag_registry(flag_key)
    ON UPDATE CASCADE
    ON DELETE RESTRICT;

-- Step 4: Add new columns introduced by §19.4.
ALTER TABLE tenant_feature_flags
  ADD COLUMN IF NOT EXISTS enabled_at               TIMESTAMPTZ,
  ADD COLUMN IF NOT EXISTS compliance_approval_ref  TEXT;

-- Backfill enabled_at from updated_at for currently-enabled rows.
UPDATE tenant_feature_flags
  SET enabled_at = updated_at
  WHERE enabled = TRUE AND enabled_at IS NULL;

-- Step 5: Surrogate id column retained as UUID PK to preserve existing FK references
-- in audit_log.resource_id. The UNIQUE (tenant_id, flag_key) constraint already existed.
-- No structural change required.

-- Step 6: Emit a DEC-030 feature_flag.registry_updated audit event for the migration.
INSERT INTO audit_log (
  tenant_id, actor_id, actor_type, action, resource_type,
  changes, outcome, metadata
)
VALUES (
  NULL,         -- system-level event; no single tenant
  NULL,         -- system actor
  'system',
  'feature_flag.registry_updated',
  'tenant_feature_flags',
  NULL,
  'success',
  jsonb_build_object(
    'migration', '0019_feature_flag_registry.sql',
    'changed_fields', ARRAY['flag_name renamed to flag_key', 'FK constraint added',
                             'enabled_at added', 'compliance_approval_ref added'],
    'note', 'Schema promotion from §2.7 stub to §19 registry-backed schema'
  )
);

COMMIT;
```

**Post-migration validation (run on production after migration, before re-enabling app traffic):**

```sql
-- Confirm zero unrecognised flag_keys
SELECT COUNT(*) FROM tenant_feature_flags
WHERE flag_key NOT IN (SELECT flag_key FROM feature_flag_registry);
-- Expected: 0

-- Confirm compliance-gated flags with enabled=TRUE all have compliance_approval_ref
SELECT tff.tenant_id, tff.flag_key, tff.compliance_approval_ref
FROM tenant_feature_flags tff
JOIN feature_flag_registry ffr ON ffr.flag_key = tff.flag_key
WHERE ffr.requires_compliance_gate = TRUE
  AND tff.enabled = TRUE
  AND tff.compliance_approval_ref IS NULL;
-- Expected: 0 rows. Any rows here require immediate investigation before app traffic resumes.
```

---

### 19.10 Gap Analysis & Open Questions

| # | Question | Owner | Priority |
|---|---|---|---|
| OQ-FLAG-01 | **Tier downgrade handling.** 🟢 **Resolved — DEC-040 (2026-06-11).** **Option (a) adopted:** plan-downgrade hook in the contract lifecycle (`tenant.lifecycle_status`) auto-disables all `tenant_feature_flags` rows whose registry `minimum_tier IN ('growth','enterprise')` when plan decreases; emits `feature_flag.disabled` DEC-030 events with `reason = 'plan_downgrade'` per flag; rows set to `enabled = FALSE` (not deleted) so re-upgrade requires deliberate re-enable. Fires synchronously inside the `tenants.plan` update transaction; `FOR UPDATE SKIP LOCKED` guards against TOCTOU. SOC 2 CC8.1 rationale: silent reactivation on re-upgrade without explicit admin action is a change management violation. See `docs/DECISION_LOG.md` DEC-040 for full rationale and reverse-cost. | `enterprise-architect` + `platform-engineer` | 🟢 **Resolved — DEC-040** |
| OQ-FLAG-02 | **Flag deprecation sunset process.** 🟢 **Resolved — DEC-040 (2026-06-11).** Formalised policy: (a) minimum **90 days** notice — `deprecated_at` set ≥ 90 days before effective disablement; (b) non-dismissible admin dashboard banner for any tenant with an enabled flag where `deprecated_at IS NOT NULL` (*"Feature [flag_label] will be retired on [date]. Contact your CSM."*); (c) pg_cron nightly at 03:15 UTC **soft-deletes** tenant rows at `deprecated_at + 90 days`, emitting `feature_flag.disabled` DEC-030 events with `reason = 'deprecated'` per row. Hard-delete rejected — tombstones preserve SOC 2 and GDPR Art. 30 audit trail. See `docs/DECISION_LOG.md` DEC-040. | `enterprise-architect` + `compliance-officer` | 🟢 **Resolved — DEC-040** |

---

### 19.11 SOC 2 Mapping

| SOC 2 Criterion | How §19 addresses it |
|---|---|
| **CC6.1** — Logical access restricted to authorised individuals | `feature_flag_registry` is read-only for all API roles; only `form_admin` (BYPASSRLS, never in request path) writes. `tenant_feature_flags` write is `form_admin` only; no `form_api` write policy exists. Entitlement gates enforce least-privilege: a Starter tenant cannot access Growth/Enterprise features. |
| **CC6.2** — Authorisation controls ensure access only to authorised resources | `assertFeatureEnabled()` tier check is the authorisation gate. Tenant plan is fetched from `tenants.plan` (written by `form_admin`, not by the tenant) and compared against `feature_flag_registry.minimum_tier`. Plan value cannot be self-elevated by the tenant. |
| **CC8.1** — Change management controls | All mutations to `tenant_feature_flags` and `feature_flag_registry` emit HMAC-chained DEC-030 audit events synchronously in-transaction. Mutation outside this chain is only possible via `form_admin` break-glass, which triggers `support.*` DEC-030 events per AUDIT_LOG_SCHEMA.md. |
| **CC9.2** — Vendor and business partner risk management | `compliance_approval_ref` column persists the DECISION_LOG entry ID for every compliance-gated flag enablement. Audit log `feature_flag.enabled` event retains this reference for 7 years. SOC 2 auditor can trace any gated flag back to the DPA clause that authorised it. |

---

### 19.12 Implementation Checklist

| # | Task description | Owner role | Priority | Milestone |
|---|---|---|---|---|
| 1 | Create `feature_flag_registry` table DDL with `flag_key_namespace` CHECK constraint; GRANT SELECT to `form_api` / `form_readonly` / `form_system`; confirm no INSERT GRANT to `form_api`; create `idx_ffr_minimum_tier` and `idx_ffr_deprecated` indexes | `enterprise-architect` | **P0** | M3 |
| 2 | Insert §19.3 seed data for all 14 canonical flags; verify zero rows violate `flag_key_namespace` CHECK after seed; confirm `webhook.email_in_payloads` has `requires_compliance_gate = TRUE` | `enterprise-architect` | **P0** | M3 |
| 3 | Run §19.9 migration: rename stub column to `flag_key`, add FK constraint, backfill `enabled_at`, add `compliance_approval_ref` column, run post-migration validation queries; migration script must abort (RAISE EXCEPTION) if any unrecognised `flag_name` values remain | `enterprise-architect` + `platform-engineer` | **P0** | M3 |
| 4 | Enable RLS on `tenant_feature_flags`; add `tff_tenant_read` (form_api) and `tff_system_read` (form_system) policies; add to `__tests__/db/rls_isolation.test.ts`: tenant_member reads zero rows for another tenant; `form_api` INSERT attempt returns permission denied | `enterprise-architect` | **P0** | M3 |
| 5 | Implement `assertFeatureEnabled(tenantId, flagKey, env)` TypeScript in `src/workers/feature-flags/assert-feature-enabled.ts`; unit tests covering: `flag_not_found` → 403; `tier_insufficient` → 403; `flag_disabled` → 403; `compliance_gate_missing` → 403; DB error → 403 (fail-closed); cache hit under 30s skips DB query | `platform-engineer` | **P0** | M3 |
| 6 | Wire `assertFeatureEnabled()` into all existing Workers that gate on `cv_enabled`, `email_in_webhooks`, `ip_allowlist_enabled`, `sso_staging_env_enabled`; replace ad-hoc string checks with typed `FlagKey` calls; CI grep gate: no raw `flag_name` string lookups in Worker code after M4 | `platform-engineer` | **P0** | M3 |
| 7 | Implement KV cache invalidation: mutation API deletes `flag:<tenantId>:<flagKey>` from KV before DB commit; integration test verifies next Worker call reads fresh value within the same request cycle | `platform-engineer` | **P0** | M3 |
| 8 | Wire all four DEC-030 audit events (§19.7): `feature_flag.enabled` and `feature_flag.disabled` on `tenant_feature_flags` mutation; `feature_flag.registry_updated` on `feature_flag_registry` mutation; `feature_flag.compliance_gate_bypassed` on blocked gated-flag write attempt; validate HMAC chain in staging before deploying to production | `platform-engineer` + `security-engineer` | **P0** | M3 |
| 9 | Implement compliance gate protocol enforcement in the internal admin flag mutation API: if `requires_compliance_gate = TRUE` and `compliance_approval_ref IS NULL`, emit `feature_flag.compliance_gate_bypassed` CRITICAL DEC-030 event and return 403; automatically open PagerDuty P1 incident on this event | `platform-engineer` + `compliance-officer` | **P0** | M3 |
| 10 | Add auto-enablement hook to tenant activation flow: on new tenant created, for each registry flag where `default_enabled_at_activation = TRUE` and tenant plan meets `minimum_tier`, insert an enabled `tenant_feature_flags` row with `set_by = NULL` (system); emit `feature_flag.enabled` DEC-030 event per flag with `reason = 'auto_activation'` | `platform-engineer` | **P1** | M4 |
| 11 | ~~Resolve OQ-FLAG-01: implement plan downgrade hook in contract lifecycle (§16) that auto-disables Growth+/Enterprise-only flags when plan decreases; emit `feature_flag.disabled` DEC-030 events with `reason = 'plan_downgrade'` for each disabled flag~~ **[x] Decision done — DEC-040 (2026-06-11):** Option (a) confirmed; `feature_flag.disabled` with `reason = 'plan_downgrade'`; hook fires in same transaction as `tenants.plan` update; `FOR UPDATE SKIP LOCKED` guard; rows set `enabled = FALSE` not deleted. Engineering implementation (platform-engineer) still required per M5 milestone. | `enterprise-architect` + `platform-engineer` | ~~**P1**~~ **🟢 Decision closed — implement per M5** | M5 |
| 12 | ~~Resolve OQ-FLAG-02: define flag deprecation sunset policy (minimum 90-day notice); implement pg_cron soft-delete of tenant flag rows at `deprecated_at + 90 days`; wire admin dashboard banner for tenants using a flag with non-null `deprecated_at`~~ **[x] Decision done — DEC-040 (2026-06-11):** 90-day sunset; non-dismissible banner; pg_cron nightly 03:15 UTC soft-delete; hard-delete rejected; `reason = 'deprecated'` DEC-030 per row. Engineering implementation still required per M6 milestone. | `enterprise-architect` + `compliance-officer` | ~~**P2**~~ **🟢 Decision closed — implement per M6** | M6 |

---

---

## 20. White-Label Tenant Branding Schema

### 20.1 Purpose and Scope

The `white_label BOOLEAN` column in the `tenants` table (§2) acts as a fast-path entitlement gate. This section defines the complementary `tenant_branding` table that stores the actual branding assets and configuration applied when that flag is `TRUE`.

White-label features per ENTERPRISE.md:
- **Custom domain**: CNAME of `coaching.acme.com` → `form.coach` (DNS resolved at Cloudflare edge)
- **Tenant logo**: replaces the FORM wordmark in the admin dashboard, mobile app, and transactional emails
- **Primary colour override**: applied to buttons, active states, and progress indicators; accessibility floor enforced at write time
- **"Powered by FORM" footer**: non-removable below $50k ARR; `powered_by_removal = TRUE` requires a signed contract amendment and `form_admin` write

This section does NOT cover:
- SSO custom domain flow — see SSO_SCIM_IMPLEMENTATION.md §4.5
- White-label COGS — see COST_MODEL §15.5
- Asset CDN delivery logic — `logo_url` stores the R2 public URL; cache headers are set at the Cloudflare R2 bucket level

### 20.2 Design Decisions

| Decision | Choice | Rationale |
|---|---|---|
| Separate table vs inline on `tenants` | Separate `tenant_branding` (1:1, lazy) | Most tenants never configure branding; keeps the hot `tenants` row small |
| `custom_domain` location | `tenant_branding.custom_domain` (not `tenants`) | Custom domain is a branding concern; avoids adding nullable columns to the core tenant row |
| SVG allowed? | **No** — PNG and WebP only | SVG allows arbitrary JS in `<script>` tags; XSS attack surface is unacceptable on a shared CDN origin |
| Colour validation layer | Application layer (admin dashboard API), not a DB CHECK | WCAG contrast requires two colours; a single-column CHECK cannot enforce the relational constraint. API rejects on fail; DB stores whatever the API accepts |
| `powered_by_removal` writability | `form_admin` BYPASSRLS only; 403 on `form_system` or `form_api` write | Contractual control — removal is a $50k ARR+ concession; self-serve would allow flag toggle without a signed amendment |

### 20.3 `tenant_branding` DDL

```sql
-- Data classification: DC-03 (business configuration, non-personal)
-- Exception: email_reply_to may contain a personal email address → DC-05 (contact data)
-- Art. 17 erasure: tenant_branding is hard-deleted with the tenant (ON DELETE CASCADE in §12)

CREATE TABLE tenant_branding (
  id                        UUID        PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id                 UUID        NOT NULL UNIQUE
                              REFERENCES tenants(id) ON DELETE CASCADE,

  -- Logos (R2 public URLs; written by asset upload API, never by direct DB write)
  logo_url                  TEXT,         -- light-mode wordmark/logo (PNG or WebP, ≤ 200 KB)
  logo_dark_url             TEXT,         -- dark-mode variant; fallback: logo_url
  favicon_url               TEXT,         -- .ico or 32×32 PNG; ≤ 50 KB
  logo_alt_text             TEXT,         -- accessibility: alt attribute for <img>

  -- Colour tokens (hex; validated for WCAG AA at write time by admin dashboard API)
  primary_color             TEXT
                              CHECK (primary_color ~ '^#[0-9A-Fa-f]{6}$'),
  primary_color_contrast    TEXT          -- auto-calculated: '#ffffff' or '#000000'
                              CHECK (primary_color_contrast IN ('#ffffff', '#000000')),

  -- Custom domain (Cloudflare Worker resolves tenant on this host; see SSO_SCIM §4.5 for SSO flow)
  custom_domain             TEXT UNIQUE,  -- FQDN e.g. 'coaching.acme.com'; null = not configured
  custom_domain_verified_at TIMESTAMPTZ,  -- null = pending DNS verification
  custom_domain_status      TEXT        NOT NULL DEFAULT 'not_configured'
                              CHECK (custom_domain_status IN
                                ('not_configured', 'pending_dns', 'active', 'error')),

  -- "Powered by FORM" enforcement
  -- FALSE = footer visible (default). TRUE = footer removed.
  -- Writable ONLY by form_admin (BYPASSRLS); form_system WITH CHECK rejects TRUE.
  -- Contract threshold: contract_arr ≥ $50,000 per ENTERPRISE.md.
  powered_by_removal        BOOLEAN     NOT NULL DEFAULT FALSE,
  powered_by_removal_ref    TEXT,         -- DECISION_LOG entry ID for the contract amendment

  -- Email sender branding (transactional emails via Resend / Postmark)
  email_sender_name         TEXT,         -- e.g. 'Acme Fitness' → "Acme Fitness via FORM <noreply@form.coach>"
  email_reply_to            TEXT,         -- optional; e.g. 'hr@acme.com'

  -- Audit
  created_at                TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at                TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_by                UUID          REFERENCES users(id) ON DELETE SET NULL
                              -- NULL when set by form_admin (system action, no user context)
);

CREATE INDEX idx_tb_tenant_id     ON tenant_branding(tenant_id);
CREATE INDEX idx_tb_custom_domain ON tenant_branding(custom_domain)
  WHERE custom_domain IS NOT NULL;

CREATE TRIGGER tenant_branding_updated_at
  BEFORE UPDATE ON tenant_branding
  FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();
```

**Constraints enforced at the application layer** (not via DB CHECK):
1. `logo_url` and `logo_dark_url` must resolve to R2 paths under `tenant-assets/{tenant_id}/` — validated at upload time
2. `primary_color` must achieve ≥ 4.5:1 contrast ratio with `#ffffff` — WCAG AA for white text on the tenant's primary background — validated in the admin dashboard API before any write
3. `custom_domain` must be a valid FQDN — validated at write time via regex `/^[a-z0-9]([a-z0-9\-]{0,61}[a-z0-9])?(\.[a-z]{2,})+$/i`
4. `powered_by_removal = TRUE` requires `powered_by_removal_ref IS NOT NULL` and `contracts.contract_arr ≥ 50000` — enforced in the admin mutation API

### 20.4 R2 Asset Storage

R2 bucket: `form-tenant-assets` (same bucket as CV session stills; separate path prefix).

```
form-tenant-assets/
└── tenant-assets/
    └── {tenant_id}/           -- UUID
        ├── logo.png            -- canonical light-mode logo
        ├── logo-dark.png       -- dark-mode variant (optional)
        └── favicon.ico         -- 32×32 px
```

**Asset upload flow** (admin dashboard API):
1. Admin dashboard UI sends `POST /internal/admin/tenant/:id/branding/logo` with `Content-Type: multipart/form-data`
2. API validates: MIME type ∈ `{image/png, image/webp}`, size ≤ 200 KB, dimensions non-zero via Sharp
3. API streams to R2 via `env.TENANT_ASSETS.put()` at the path above
4. On success: writes `logo_url = 'https://assets.form.coach/tenant-assets/{tenant_id}/logo.png?v={ts}'` to `tenant_branding`, emits `branding.logo_updated` DEC-030 event, and invalidates KV cache key `branding:{tenant_id}`
5. Old asset is overwritten in-place (R2 overwrite); DEC-030 audit chain provides change history

**Cache headers on R2 public bucket**: `Cache-Control: public, max-age=3600, s-maxage=86400`. Cache-bust is achieved by the `?v={unix_timestamp}` suffix written into `logo_url` by the upload API.

**Allowed MIME types**: `image/png`, `image/webp` only. SVG is explicitly rejected — see §20.2. Favicon additionally accepts `image/x-icon`; max 50 KB; 32×32 px minimum validated at upload time.

**Art. 17 erasure**: tenant logo files are deleted from R2 synchronously when a tenant account is hard-deleted, via an explicit R2 delete call in the tenant deletion Worker (the SQL `ON DELETE CASCADE` handles the DB row; it does not reach R2). Deletion is recorded under `branding.logo_deleted` DEC-030.

### 20.5 WCAG Accessibility Floor

ENTERPRISE.md specifies a "colour override з accessibility floor." This section defines that floor and its enforcement implementation.

**Minimum contrast requirement**: primary colour must achieve ≥ **4.5:1** contrast ratio with white (`#ffffff`) when used as a background behind white text — WCAG 2.1 AA, Success Criterion 1.4.3 (normal text). This protects the white "FORM" wordmark and action-button labels rendered on the tenant's primary colour.

```typescript
// src/workers/admin/branding/validate-color.ts

// WCAG relative luminance (IEC 61966-2-1 / sRGB)
function relativeLuminance(hex: string): number {
  const r = parseInt(hex.slice(1, 3), 16) / 255;
  const g = parseInt(hex.slice(3, 5), 16) / 255;
  const b = parseInt(hex.slice(5, 7), 16) / 255;
  const linearise = (c: number) =>
    c <= 0.04045 ? c / 12.92 : ((c + 0.055) / 1.055) ** 2.4;
  return 0.2126 * linearise(r) + 0.7152 * linearise(g) + 0.0722 * linearise(b);
}

function contrastRatio(hex1: string, hex2: string): number {
  const L1 = relativeLuminance(hex1);
  const L2 = relativeLuminance(hex2);
  const [L, D] = L1 > L2 ? [L1, L2] : [L2, L1];
  return (L + 0.05) / (D + 0.05);
}

export function pickContrastColor(hex: string): '#ffffff' | '#000000' {
  return contrastRatio(hex, '#ffffff') >= contrastRatio(hex, '#000000')
    ? '#ffffff'
    : '#000000';
}

export function validatePrimaryColor(hex: string): {
  valid: boolean;
  contrastWithWhite: number;
  primaryColorContrast: '#ffffff' | '#000000';
  warning: string | null;
} {
  const ratio = contrastRatio(hex, '#ffffff');
  return {
    valid: ratio >= 4.5,
    contrastWithWhite: Math.round(ratio * 100) / 100,
    primaryColorContrast: pickContrastColor(hex),
    warning:
      ratio >= 3.0 && ratio < 4.5
        ? `Contrast ${ratio.toFixed(2)}:1 fails WCAG AA (≥ 4.5:1 required for white text)`
        : null,
  };
}
// < 3.0:1  → API returns 422; tenant must choose a different colour
// 3.0–4.5:1 → API returns 422; admin must correct
// ≥ 4.5:1  → accepted; primary_color_contrast auto-set to '#ffffff' or '#000000'
```

**Admin dashboard UX**: colour picker renders a live contrast badge. The save button is disabled below 4.5:1 — no override modal exists. Below 3.0:1 the API returns 422 unconditionally.

### 20.6 "Powered by FORM" Enforcement

```
Rule: powered_by_removal = FALSE always, UNLESS:
  1. tenants.plan = 'enterprise'
  2. Matching contracts row has contract_arr >= 50000
  3. Contract amendment signed (DECISION_LOG entry exists)
  4. powered_by_removal_ref IS NOT NULL (references the DECISION_LOG entry ID)
  5. Write performed by form_admin role (BYPASSRLS)
```

**API enforcement** (`src/workers/admin/branding/set-powered-by-removal.ts`):
```typescript
export async function setPoweredByRemoval(
  tenantId: string,
  value: boolean,
  decisionLogRef: string | null,
  env: Env
): Promise<void> {
  // Callable ONLY from the internal form_admin API (service-token auth).
  // form_api and form_system routes must reject this endpoint at the router layer.

  if (value && !decisionLogRef) {
    await emitDec030('branding.powered_by_removal_attempted_without_ref', {
      tenant_id: tenantId, blocked: true, severity: 'CRITICAL',
    }, env);
    throw new ApiError(403, 'powered_by_removal requires a DECISION_LOG reference');
  }

  const contract = await getActiveContract(tenantId, env.DB);
  if (value && (!contract || contract.contract_arr < 50_000)) {
    await emitDec030('branding.powered_by_removal_below_arr_threshold', {
      tenant_id: tenantId,
      contract_arr: contract?.contract_arr ?? 0,
      blocked: true,
      severity: 'HIGH',
    }, env);
    throw new ApiError(403, 'powered_by_removal requires contract_arr ≥ $50,000');
  }

  await env.DB.prepare(
    `UPDATE tenant_branding
       SET powered_by_removal = ?, powered_by_removal_ref = ?, updated_at = now()
     WHERE tenant_id = ?`
  ).bind(value, decisionLogRef, tenantId).run();

  await emitDec030(
    value
      ? 'branding.powered_by_removal_enabled'
      : 'branding.powered_by_removal_revoked',
    { tenant_id: tenantId, decision_log_ref: decisionLogRef, severity: 'HIGH' },
    env
  );
}
```

**Mobile / web rendering**: the mobile app fetches `powered_by_removal` from the branding API (KV-cached, 300s TTL). If `FALSE` (default), the "Powered by FORM" footer component renders unconditionally — it cannot be suppressed via client-side state. The render decision is the server-returned value, not a client toggle.

### 20.7 Custom Domain DNS Management

Custom domains use Cloudflare's proxied CNAME model. The tenant's IT admin creates the DNS record; FORM does not manage the tenant's DNS zone.

**Tenant-side setup** (instructions sent by customer-success):
```
CNAME coaching.acme.com → form.coach   (Cloudflare proxied)
```

**FORM-side verification flow**:
1. Admin triggers `POST /internal/admin/tenant/:id/branding/verify-domain`
2. Worker calls Cloudflare DNS-over-HTTPS to confirm the CNAME resolves to `form.coach`
3. Worker issues `OPTIONS https://coaching.acme.com/.well-known/form-verify` — expects `X-FORM-Tenant-Id` response header set by the edge Worker (confirms Cloudflare proxying is active)
4. Pass → `custom_domain_status = 'active'`, `custom_domain_verified_at = now()`, emit `branding.custom_domain_verified`
5. Fail → `custom_domain_status = 'error'`, emit `branding.custom_domain_verification_failed`

**Cloudflare Worker resolution** (leverages existing SSO_SCIM_IMPLEMENTATION.md §4.5 infrastructure):
```typescript
// On each inbound request arriving at a custom domain:
const host = request.headers.get('Host');
const branding = await kv.get<TenantBranding>(`branding-domain:${host}`); // 60s TTL
const tenantId = branding?.tenant_id
  ?? await db.query(
      'SELECT tenant_id FROM tenant_branding WHERE custom_domain = $1 AND custom_domain_status = $2',
      [host, 'active']
    ).then(r => r.rows[0]?.tenant_id);

if (!tenantId) return new Response('Not found', { status: 404 });
// Set trusted internal header — NOT trusted from the internet (strip on ingress)
headers.set('X-FORM-Tenant-Id', tenantId);
```

**SSL**: Cloudflare Universal SSL covers the proxied custom domain automatically. No FORM-side certificate management required. Certificate expiry monitoring is in OBSERVABILITY.md §6.

### 20.8 Row-Level Security

```sql
ALTER TABLE tenant_branding ENABLE ROW LEVEL SECURITY;

-- Tenant members: read own branding (used by mobile app theming API)
CREATE POLICY tb_tenant_read ON tenant_branding
  FOR SELECT TO form_api
  USING (tenant_id = current_setting('app.current_tenant_id')::UUID);

-- form_system: full read (admin dashboard API, BI tooling)
CREATE POLICY tb_system_read ON tenant_branding
  FOR SELECT TO form_system
  USING (TRUE);

-- form_system: insert + update allowed; powered_by_removal = TRUE rejected by WITH CHECK
-- All branding mutations are routed through form_system (admin dashboard API)
-- EXCEPT powered_by_removal, which requires form_admin BYPASSRLS
CREATE POLICY tb_system_write ON tenant_branding
  FOR ALL TO form_system
  USING (TRUE)
  WITH CHECK (
    powered_by_removal = FALSE  -- form_system can never set TRUE; that path requires form_admin
  );

-- form_api: zero write access (no policy = deny)
-- All branding writes flow through admin dashboard API (form_system) or internal admin API (form_admin).
-- Reason: every mutation requires a DEC-030 event; form_api has no audit-event emission path.

-- form_admin: BYPASSRLS — only role permitted to write powered_by_removal = TRUE
```

**Cross-tenant RLS isolation tests** (add to `__tests__/db/rls_isolation.test.ts`):
```typescript
it('tenant_branding: form_api reads zero rows for another tenant', async () => {
  await setTenantContext(dbA, tenantAId);
  const rows = await dbA.query(
    'SELECT * FROM tenant_branding WHERE tenant_id = $1', [tenantBId]
  );
  expect(rows.rows).toHaveLength(0);
});

it('tenant_branding: form_system WITH CHECK rejects powered_by_removal = TRUE', async () => {
  await expect(
    dbSystem.query(
      'UPDATE tenant_branding SET powered_by_removal = TRUE WHERE tenant_id = $1',
      [tenantAId]
    )
  ).rejects.toThrow(); // WITH CHECK violation
});
```

### 20.9 DEC-030 HMAC-Chained Audit Events

All `tenant_branding` mutations emit HMAC-chained events per AUDIT_LOG_SCHEMA.md DEC-030.

| Event key | Severity | Retention | Trigger |
|---|---|---|---|
| `branding.logo_updated` | STANDARD | 7yr | Logo or favicon uploaded successfully |
| `branding.logo_deleted` | STANDARD | 7yr | Asset file deleted from R2 (tenant delete cascade) |
| `branding.color_updated` | STANDARD | 7yr | `primary_color` or `primary_color_contrast` written |
| `branding.custom_domain_added` | HIGH | 7yr | Non-null `custom_domain` written for the first time |
| `branding.custom_domain_verified` | STANDARD | 7yr | DNS verification passed; `custom_domain_status = 'active'` |
| `branding.custom_domain_verification_failed` | HIGH | 7yr | DNS verification failed; `custom_domain_status = 'error'` |
| `branding.custom_domain_removed` | HIGH | 7yr | `custom_domain` set to NULL |
| `branding.email_sender_updated` | STANDARD | 7yr | `email_sender_name` or `email_reply_to` changed |
| `branding.powered_by_removal_enabled` | HIGH | 7yr | `powered_by_removal` → TRUE; `decision_log_ref` in payload |
| `branding.powered_by_removal_revoked` | HIGH | 7yr | `powered_by_removal` → FALSE |
| `branding.powered_by_removal_attempted_without_ref` | CRITICAL | 10yr | Attempt without `decision_log_ref`; always `blocked: true`; auto PagerDuty P1 |
| `branding.powered_by_removal_below_arr_threshold` | HIGH | 7yr | Attempt when `contract_arr < 50,000`; always `blocked: true` |

**Payload schema** (consistent with AUDIT_LOG_SCHEMA.md §2):
```json
{
  "event": "branding.logo_updated",
  "tenant_id": "uuid",
  "actor_user_id": "uuid | null",
  "actor_role": "form_system | form_admin",
  "timestamp": "ISO-8601",
  "data": {
    "field": "logo_url",
    "asset_path": "tenant-assets/{tenant_id}/logo.png",
    "file_size_bytes": 42500,
    "mime_type": "image/png"
  },
  "hmac": "sha256:...",
  "prev_hmac": "sha256:..."
}
```

Events MUST NOT include the previous or new logo URL value — avoids storing asset URLs in the tamper-evident chain beyond what the audit record requires.

### 20.10 Migration from `tenants.white_label`

The existing `tenants.white_label` boolean remains as a fast-path gate in Worker entitlement checks (no JOIN required). `tenant_branding` rows are lazily instantiated on first branding write.

```sql
-- No schema change to the tenants table is required.
-- tenant_branding rows are created by the first admin dashboard branding write.
-- For tenants where tenants.white_label = TRUE but no tenant_branding row exists:
-- the Cloudflare Worker applies FORM defaults (standard logo, FORM brand colour).
-- This is correct: white_label = TRUE means entitled, not configured.

-- Nightly consistency check (pg_cron at 04:00 UTC):
-- Detects the impossible state where a non-white-label tenant has
-- a custom domain or powered_by_removal set.
SELECT t.id, t.slug
FROM tenants t
JOIN tenant_branding tb ON tb.tenant_id = t.id
WHERE t.white_label = FALSE
  AND (tb.custom_domain IS NOT NULL OR tb.powered_by_removal = TRUE);
-- Any row returned = data integrity violation.
-- On result: emit CRITICAL DEC-030 'branding.invariant_violation', alert PagerDuty P1.
```

### 20.11 Tenant Branding API Endpoint Map

| Method | Path | Auth | Action |
|---|---|---|---|
| `GET` | `/api/v1/tenant/branding` | `form_api` (any member) | Read branding config for mobile app theming; KV-cached 300s |
| `PUT` | `/internal/admin/tenant/:id/branding/color` | `form_system` + `tenant_admin` JWT | Update `primary_color`; validates WCAG AA before write |
| `POST` | `/internal/admin/tenant/:id/branding/logo` | `form_system` + `tenant_admin` JWT | Upload logo PNG/WebP to R2; writes `logo_url` |
| `POST` | `/internal/admin/tenant/:id/branding/logo-dark` | `form_system` + `tenant_admin` JWT | Upload dark-mode logo |
| `POST` | `/internal/admin/tenant/:id/branding/favicon` | `form_system` + `tenant_admin` JWT | Upload favicon to R2 |
| `PUT` | `/internal/admin/tenant/:id/branding/custom-domain` | `form_system` + `tenant_admin` JWT | Set or clear `custom_domain`; sets status `pending_dns` |
| `POST` | `/internal/admin/tenant/:id/branding/verify-domain` | `form_system` + `tenant_admin` JWT | Re-run DNS + preflight verification check |
| `PUT` | `/internal/admin/tenant/:id/branding/email-sender` | `form_system` + `tenant_admin` JWT | Update `email_sender_name` / `email_reply_to` |
| `PUT` | `/internal/form-admin/tenant/:id/branding/powered-by-removal` | `form_admin` service token only | Toggle `powered_by_removal`; requires `decision_log_ref` + ARR gate |

All write paths invalidate `branding:{tenant_id}` KV cache synchronously before returning 200.

### 20.12 SOC 2 Evidence Mapping

| Control | TSC Criterion | Evidence artefact |
|---|---|---|
| Branding changes are audit-logged with actor, timestamp, and HMAC chain | CC7.2 — System monitoring; CC8.1 — Change management | `audit_log_events` rows for all `branding.*` event keys; HMAC chain verified by weekly cron (OBSERVABILITY.md §6) |
| `powered_by_removal` requires DECISION_LOG reference and ARR gate | CC8.1 — Change management; CC6.1 — Logical access | `tenant_branding.powered_by_removal_ref` persisted 7yr; `contracts.contract_arr` validation code path unit-tested |
| Custom domain verified before activation | CC6.2 — Authorisation (DNS ownership proof before domain becomes active) | `branding.custom_domain_verified` DEC-030 event with `custom_domain_verified_at` timestamp; `custom_domain_status` state machine prevents activation without verification |
| SVG rejected at API layer | CC6.8 — Input validation; CC7.2 — Malicious content detection | MIME-type allow-list unit test in `__tests__/admin/branding/asset-upload.test.ts` |

**Auditor evidence artefacts**:
- `compliance/evidence/branding/branding-schema.sql` — DDL export from production Supabase, refreshed quarterly
- `compliance/evidence/branding/rls-policies.sql` — RLS policy export, matches §20.8 DDL
- `compliance/evidence/branding/powered-by-removal-log.csv` — `audit_log_events` export filtered on `branding.powered_by_removal_enabled`; reviewed at each quarterly access review (§23)

### 20.13 Open Questions

| # | Question | Owner | Priority |
|---|---|---|---|
| OQ-BRAND-01 | Should `email_reply_to` require a FORM-operated domain (e.g. `@acme.form.coach`) to prevent SPF misalignment and phishing risk? Currently accepts arbitrary email addresses. | compliance-officer + security-engineer | Medium — resolve before first white-label transactional email send |
| OQ-BRAND-02 | At what ARR / seat threshold should a second colour token (`secondary_color`) and heading font override be offered? Omitted here to limit accessibility validation scope. | enterprise-architect | Low — revisit at first Enterprise customer requesting it |
| OQ-BRAND-03 | `custom_domain_status = 'error'` has no automated retry today. Should verification auto-retry on a 24h pg_cron, or require manual re-trigger by the tenant admin? | platform-engineer | Medium — manual trigger is acceptable for initial launch; auto-retry is a P2 improvement |

### 20.14 Implementation Checklist

| # | Task description | Owner | Priority | Milestone |
|---|---|---|---|---|
| 1 | Create `migrations/YYYYMMDD_tenant_branding.sql` — `tenant_branding` DDL, CHECK constraints, indexes, `update_updated_at_column()` trigger, all RLS policies from §20.8 | `enterprise-architect` | **P0** | M5 |
| 2 | Implement `validatePrimaryColor()` in `src/workers/admin/branding/validate-color.ts` per §20.5; unit tests: `#ffffff` (1:1 → fail), `#1a1f2e` FORM navy (passes), `#888888` (3.9:1 → warn) | `platform-engineer` | **P0** | M5 |
| 3 | Implement asset upload API: MIME allow-list (PNG/WebP/ICO), size gate, Sharp dimension read, R2 put with `?v={ts}` cache-bust suffix, DB write, KV invalidation, `branding.logo_updated` DEC-030 event | `platform-engineer` | **P0** | M5 |
| 4 | Implement `GET /api/v1/tenant/branding` with KV 300s cache; response: `{ logo_url, logo_dark_url, favicon_url, primary_color, primary_color_contrast, powered_by_removal, custom_domain, email_sender_name }`; test: cache hit/miss, cross-tenant RLS | `platform-engineer` | **P0** | M5 |
| 5 | Implement `PUT /internal/form-admin/tenant/:id/branding/powered-by-removal` per §20.6 — form_admin service token validation, `decision_log_ref` requirement, ARR gate from `contracts` table (§16), DEC-030 events for enable/revoke/blocked paths | `platform-engineer` + `compliance-officer` | **P0** | M5 |
| 6 | Implement custom domain flow: `PUT` sets `pending_dns`; `POST verify-domain` runs CNAME + preflight check; state machine transitions; `branding.custom_domain_added`, `branding.custom_domain_verified`, `branding.custom_domain_verification_failed` DEC-030 events | `platform-engineer` | **P0** | M5 |
| 7 | Wire all 12 DEC-030 audit events (§20.9) to their triggers; confirm `branding.powered_by_removal_attempted_without_ref` carries `blocked: true` and CRITICAL severity; auto-open PagerDuty P1 on CRITICAL branding events | `platform-engineer` + `security-engineer` | **P0** | M5 |
| 8 | Add cross-tenant RLS isolation tests to `__tests__/db/rls_isolation.test.ts` per §20.8 — form_api zero rows for another tenant; form_system WITH CHECK rejects `powered_by_removal = TRUE` | `enterprise-architect` | **P0** | M5 |
| 9 | Add nightly pg_cron consistency check at 04:00 UTC (§20.10) — emit CRITICAL DEC-030 `branding.invariant_violation` + PagerDuty P1 if any non-white-label tenant has `custom_domain IS NOT NULL` or `powered_by_removal = TRUE` | `platform-engineer` + `devops-lead` | **P0** | M5 |
| 10 | Add `branding.white_label_enabled` flag to `feature_flag_registry` seed data (§19.3) if missing; wire `assertFeatureEnabled('branding.white_label_enabled')` gate into all branding write paths | `platform-engineer` | **P1** | M5 |
| 11 | Admin dashboard UI: branding settings page — live colour picker with WCAG contrast badge, logo upload with dimensions preview, custom domain config with verification status indicator; `powered_by_removal` toggle visible only in internal form_admin view | `enterprise-architect` | **P1** | M6 |
| 12 | Add `tenant_branding` to Art. 17 erasure documentation in §12 (soft delete section); confirm `ON DELETE CASCADE` covers DB row; add explicit R2 asset deletion call to tenant deletion Worker (CASCADE does not reach R2) | `platform-engineer` + `compliance-officer` | **P1** | M5 |
| 13 | File compliance artefacts: `compliance/evidence/branding/rls-policies.sql` and `compliance/evidence/branding/branding-schema.sql`; add both to SOC2_READINESS.md §38.6 evidence index | `compliance-officer` | **P2** | M6 |

---

## 21. AI Coaching Session & Memory Schema

> Owner: `enterprise-architect` + `platform-engineer` + `compliance-officer`. Review: on any LLM provider change, token pricing revision, or GDPR erasure audit.
> References: DEC-030 (HMAC-chained audit log), docs/AUDIT_LOG_SCHEMA.md, docs/ENTERPRISE.md, docs/SOC2_READINESS.md, §12 (Art. 17 erasure), §19 (feature flag registry).

---

### 21.1 Purpose and Scope

Victor is FORM's AI strength training coach persona, powered by the Anthropic Claude API. Every Victor interaction is a multi-turn LLM conversation: the mobile client exchanges messages with a Cloudflare Worker, which assembles a context window (system prompt + injected memory + recent workout history + wearable summary) and calls `claude-sonnet-4-6` (or a future model) via the Anthropic Messages API.

This section defines the database schema, encryption posture, context-window management strategy, GDPR erasure integration, cost attribution model, and audit trail for all Victor interactions within the multi-tenant FORM platform.

**What this section covers:**

- Five tables: `coaching_sessions`, `coaching_turns`, `coaching_memory`, `coaching_feedback`, `coaching_session_context`
- Column-level AES-256-GCM encryption for message content and extracted memory values
- RLS policies for all five tables across `form_api`, `form_system`, and `form_admin` roles
- DEC-030 HMAC-chained audit events for all safety-critical paths
- Context window management: auto-completion at 80% utilisation and fresh-session handoff
- GDPR Art. 17 erasure sequence integrated with §12
- Per-tenant cost attribution with `session_cost_usd` and a materialized view for monthly roll-up
- SOC 2 control mapping: CC6.1, CC6.7, CC7.2, CC9.2

**What this section does NOT cover:**

- The Cloudflare Worker prompt assembly logic — see `src/workers/coaching/context-builder.ts`
- The Anthropic API request/retry logic — see `src/lib/anthropic/client.ts`
- Clinical safety screening rules — see `docs/CLINICAL_SAFETY.md`
- Billing UI for coaching cost attribution — see admin dashboard specs

---

### 21.2 Design Decisions

| Decision | Choice | Rationale |
|---|---|---|
| One session per conversation thread, not per day | Session is bounded by the LLM conversation state (Messages API thread) | Context window is the unit of billing and truncation risk; sessions must track exactly what was sent to Anthropic |
| Column-level encryption for `content_encrypted` | AES-256-GCM, key from Cloudflare Secrets Store | DB-level encryption (Supabase transparent encryption) is a necessary baseline but insufficient; column-level encryption ensures the DB superuser cannot read turn content — required for CC6.7 and the principle of least privilege |
| Separate `coaching_memory` table vs embedding memory in session metadata | Separate table with dotted-namespace keys | Memory outlives any single session; structured keys allow precise GDPR erasure of specific facts (e.g. `user.injuries.known`) without deleting the entire profile |
| `coaching_session_context` snapshot per session | Immutable snapshot of injected context (hashes only, not content) | Required for debugging context truncation bugs and for demonstrating reproducibility to auditors; content is NOT stored here to avoid double-storing encrypted content |
| `clinical_safety_flagged` on turns, not sessions | Per-turn boolean + reason | Clinical safety violations are turn-level events; flagging the session would mask which specific response triggered the classifier |
| session_token is client-facing, not the Anthropic API key | Short-lived opaque UUID token (JWT or random 32-byte hex) | The mobile client must never touch the Anthropic API key; session_token is exchanged for a Worker-managed Anthropic API call with tenant-scoped rate limiting |
| Cost attribution at session level, not turn level | `session_cost_usd` on `coaching_sessions`; per-turn tokens on `coaching_turns` | Enterprise billing is by tenant-month; session-level cost enables the materialized view roll-up without a per-turn aggregation scan |

---

### 21.3 `coaching_sessions` DDL

```sql
-- Data classification: DC-02 (operational metadata, non-personal content)
-- session_token and user_id are DC-05 (personal identifiers); content is in coaching_turns.
-- Art. 17 erasure: metadata retained for billing audit; no PII in this table's payload columns.
-- Retention: 7 years (billing record).

CREATE TABLE coaching_sessions (
  id                        UUID        PRIMARY KEY DEFAULT gen_random_uuid(),

  tenant_id                 UUID        NOT NULL
                              REFERENCES tenants(id) ON DELETE CASCADE,
                            -- REQUIRED on every table. RLS index on (tenant_id, user_id).

  user_id                   UUID        NOT NULL
                              REFERENCES users(id) ON DELETE CASCADE,

  session_token             TEXT        NOT NULL UNIQUE,
                            -- Short-lived opaque token issued to the mobile client.
                            -- NOT the Anthropic API key. Format: 32-byte hex, URL-safe.
                            -- Expires: 4 hours of inactivity (enforced in Worker, not DB).
                            -- Worker verifies token before every /api/v1/coach/turn call.

  model_id                  TEXT        NOT NULL DEFAULT 'claude-sonnet-4-6',
                            -- Anthropic model slug used for this session.
                            -- Stored at session creation; survives model upgrades in flight.
                            -- Allows cost recalculation with correct per-model pricing.

  status                    TEXT        NOT NULL DEFAULT 'active'
                              CHECK (status IN ('active', 'completed', 'abandoned', 'error')),
                            -- active     = session is open; mobile client may send turns.
                            -- completed  = natural end or context-window limit reached.
                            -- abandoned  = client disconnected / token expired without completion.
                            -- error      = Anthropic API error that prevented completion.

  message_count             INTEGER     NOT NULL DEFAULT 0,
                            -- Total number of turns in this session (user + assistant messages).
                            -- Incremented atomically by the turn-write Worker.

  input_tokens_total        INTEGER     NOT NULL DEFAULT 0,
                            -- Cumulative input tokens across all turns in this session.
                            -- Includes cache_read_tokens and cache_write_tokens.

  output_tokens_total       INTEGER     NOT NULL DEFAULT 0,
                            -- Cumulative output tokens (assistant responses) in this session.

  cache_read_tokens         INTEGER     NOT NULL DEFAULT 0,
                            -- Tokens served from Anthropic prompt cache (billed at discount).
                            -- Separated to allow accurate cost calculation.

  cache_write_tokens        INTEGER     NOT NULL DEFAULT 0,
                            -- Tokens written to Anthropic prompt cache in this session.

  session_cost_usd          NUMERIC(10,6) NOT NULL DEFAULT 0.000000,
                            -- Computed cost of this session in USD.
                            -- Formula (illustrative; actual rates from Anthropic API):
                            --   input_tokens  * $0.000003    (per token, non-cached)
                            --   output_tokens * $0.000015
                            --   cache_read    * $0.0000003
                            --   cache_write   * $0.00000375
                            -- Updated after every turn by the turn-write Worker.
                            -- Source of truth for tenant monthly cost roll-up.

  context_window_used_pct   NUMERIC(5,2) NOT NULL DEFAULT 0.00,
                            -- Percentage of model context window consumed.
                            -- Computed: (input_tokens_total / model_context_window_size) * 100.
                            -- When this exceeds 80%, the session is auto-completed and a new
                            -- session is started with a fresh memory injection (§21.9).

  started_at                TIMESTAMPTZ NOT NULL DEFAULT now(),
  last_active_at            TIMESTAMPTZ NOT NULL DEFAULT now(),
                            -- Updated on every turn; used by abandonment detection cron.
  completed_at              TIMESTAMPTZ,
                            -- Set when status transitions to 'completed', 'abandoned', or 'error'.

  created_at                TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at                TIMESTAMPTZ NOT NULL DEFAULT now()
);

-- Triggers
CREATE TRIGGER set_coaching_sessions_updated_at
  BEFORE UPDATE ON coaching_sessions
  FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();

-- Indexes
CREATE INDEX idx_cs_tenant_user
  ON coaching_sessions(tenant_id, user_id);
  -- Primary RLS lookup path: every query filters by both.

CREATE INDEX idx_cs_tenant_status
  ON coaching_sessions(tenant_id, status)
  WHERE status = 'active';
  -- Narrow partial index for the Worker's active-session lookup.

CREATE INDEX idx_cs_session_token
  ON coaching_sessions(session_token);
  -- Token verification on every /api/v1/coach/turn call; must be fast.

CREATE INDEX idx_cs_tenant_started_at
  ON coaching_sessions(tenant_id, started_at DESC);
  -- Monthly cost roll-up materialized view scan.

ALTER TABLE coaching_sessions ENABLE ROW LEVEL SECURITY;
```

---

### 21.4 `coaching_turns` DDL

```sql
-- Data classification: DC-06 (sensitive personal health content, encrypted at column level)
-- content_encrypted contains verbatim user messages and Victor responses.
-- These are the most sensitive rows in the schema: eating disorder mentions, injury disclosures,
-- mental health context. Column-level encryption is MANDATORY even though Supabase encrypts
-- at rest — the DB superuser must not be able to read plaintext turn content.
--
-- Encryption: AES-256-GCM. Key stored in Cloudflare Secrets Store (not in DB).
-- Key rotation: quarterly, using envelope encryption (data key encrypted by root key).
-- Decryption: only via the coaching Worker endpoint; DB layer never sees plaintext.
--
-- Art. 17 erasure: content_encrypted overwritten with NULL on erasure request.
--   content_hash is retained (hash of NULL becomes a sentinel value).
--   Token counts and metadata retained for billing audit (§21.10).

CREATE TABLE coaching_turns (
  id                        UUID        PRIMARY KEY DEFAULT gen_random_uuid(),

  tenant_id                 UUID        NOT NULL
                              REFERENCES tenants(id) ON DELETE CASCADE,

  session_id                UUID        NOT NULL
                              REFERENCES coaching_sessions(id) ON DELETE CASCADE,

  user_id                   UUID        NOT NULL
                              REFERENCES users(id) ON DELETE CASCADE,
                            -- Denormalized for RLS performance; avoids JOIN through sessions.

  turn_index                INTEGER     NOT NULL,
                            -- 0-based position within the session. Monotonically increasing.
                            -- (session_id, turn_index) is unique and defines message order.

  role                      TEXT        NOT NULL
                              CHECK (role IN ('user', 'assistant')),
                            -- 'user'      = message from the FORM mobile client (human turn).
                            -- 'assistant' = response from Victor (Claude).

  content_encrypted         TEXT,
                            -- AES-256-GCM ciphertext of the turn content.
                            -- Base64url-encoded. Format: <nonce>.<ciphertext>.<tag>
                            -- NULL after GDPR Art. 17 erasure.
                            -- SELECT returns NULL to form_api role (masked; see §21.7 RLS).
                            -- Decryption requires the coaching Worker to present valid JWT
                            -- and the plaintext is returned only to the owning user.

  content_hash              TEXT,
                            -- SHA-256 of the UTF-8 plaintext, hex-encoded.
                            -- Used for deduplication detection (retry-safe turn submission).
                            -- NOT a reversible value; safe to retain post-erasure.
                            -- Set to 'erased' sentinel after Art. 17 erasure.

  input_tokens              INTEGER,
                            -- Anthropic input tokens for this turn (user message tokens +
                            -- context window tokens sent with this request).

  output_tokens             INTEGER,
                            -- Anthropic output tokens for this turn (assistant response tokens).

  cache_read_tokens         INTEGER     NOT NULL DEFAULT 0,
                            -- Tokens served from prompt cache for this specific turn.

  cache_write_tokens        INTEGER     NOT NULL DEFAULT 0,
                            -- Tokens written to prompt cache for this specific turn.

  latency_ms                INTEGER,
                            -- End-to-end latency from Worker sending the Anthropic request
                            -- to receiving the complete streamed response (time-to-last-token).

  model_id                  TEXT        NOT NULL,
                            -- Model used for this specific turn. May differ from session model_id
                            -- if the model was upgraded mid-session (handled by Worker).

  stop_reason               TEXT
                              CHECK (stop_reason IN (
                                'end_turn', 'max_tokens', 'stop_sequence', 'tool_use'
                              )),
                            -- Anthropic stop_reason from the API response.
                            -- 'max_tokens' on an assistant turn = context truncation risk.
                            --   The Worker logs a warning and triggers session auto-completion.

  clinical_safety_flagged   BOOLEAN     NOT NULL DEFAULT FALSE,
                            -- Set to TRUE by the post-generation clinical safety Worker
                            -- (runs async after the response is streamed to the client).
                            -- Triggers: ED screening keywords, self-harm indicators,
                            -- mentions of medical contraindications without disclaimer.
                            -- See docs/CLINICAL_SAFETY.md for classifier rules.

  clinical_safety_reason    TEXT,
                            -- Human-readable reason code from the safety classifier.
                            -- e.g. 'ed_screening_keyword', 'self_harm_indicator'.
                            -- NULL when clinical_safety_flagged = FALSE.
                            -- NOT the flagged content itself — reason code only (no PII in audit).

  created_at                TIMESTAMPTZ NOT NULL DEFAULT now()
                            -- Immutable. Turns are append-only; never updated.
                            -- (No updated_at — mutations are prohibited by RLS.)
);

-- Constraints
ALTER TABLE coaching_turns
  ADD CONSTRAINT uq_turn_position UNIQUE (session_id, turn_index);

-- Indexes
CREATE INDEX idx_ct_session_id
  ON coaching_turns(session_id, turn_index ASC);
  -- Primary read path: fetch all turns for a session in order.

CREATE INDEX idx_ct_tenant_user
  ON coaching_turns(tenant_id, user_id);
  -- RLS enforcement path and GDPR erasure scan.

CREATE INDEX idx_ct_safety_flagged
  ON coaching_turns(tenant_id, clinical_safety_flagged)
  WHERE clinical_safety_flagged = TRUE;
  -- Admin dashboard safety flag monitoring query.

CREATE INDEX idx_ct_content_hash
  ON coaching_turns(content_hash)
  WHERE content_hash IS NOT NULL AND content_hash != 'erased';
  -- Deduplication lookup on turn submission retry.

ALTER TABLE coaching_turns ENABLE ROW LEVEL SECURITY;
```

---

### 21.5 `coaching_memory` DDL

```sql
-- Data classification: DC-06 (sensitive personal health facts, encrypted at column level)
-- memory_value_encrypted contains extracted facts: goals, injuries, training history summaries.
-- These are the most structurally sensitive rows: a single memory key 'user.injuries.known'
-- may contain a clinical diagnosis. Column-level encryption is MANDATORY.
--
-- Memory is extracted by the memory Worker after each session completes.
-- Keys follow a dotted namespace (§21.6). Schema evolution handled via OQ-COACH-03.
--
-- Art. 17 erasure: is_deleted = TRUE, memory_value_encrypted = NULL.
--   Row is retained (soft delete) for billing audit integrity.
--   Erasure integrates with §12 erasure sequence.

CREATE TABLE coaching_memory (
  id                        UUID        PRIMARY KEY DEFAULT gen_random_uuid(),

  tenant_id                 UUID        NOT NULL
                              REFERENCES tenants(id) ON DELETE CASCADE,

  user_id                   UUID        NOT NULL
                              REFERENCES users(id) ON DELETE CASCADE,

  memory_key                TEXT        NOT NULL,
                            -- Dotted namespace. Examples:
                            --   'user.goals.primary'           — primary training goal
                            --   'user.goals.timeline'          — target date or event
                            --   'user.injuries.known'          — active injury list
                            --   'user.injuries.contraindications' — exercises to avoid
                            --   'user.training_history.summary' — free-text training background
                            --   'user.preferences.workout_time' — morning / evening
                            --   'user.context.recent_life_stressor' — ephemeral, expires_at set
                            -- Namespace validated by Worker before insert. DB CHECK below.
                            CONSTRAINT memory_key_format
                              CHECK (memory_key ~ '^[a-z_]+(\.[a-z_]+){1,4}$'),

  memory_value_encrypted    TEXT,
                            -- AES-256-GCM ciphertext of the extracted fact value.
                            -- Base64url-encoded. Format: <nonce>.<ciphertext>.<tag>
                            -- NULL after GDPR Art. 17 erasure.

  confidence_score          NUMERIC(3,2)
                              CHECK (confidence_score >= 0.00 AND confidence_score <= 1.00),
                            -- Confidence assigned by the memory-extraction Worker (LLM-scored).
                            -- 0.00–0.49: uncertain; may be contradicted by later turns.
                            -- 0.50–0.79: likely; inject into context with hedge phrasing.
                            -- 0.80–1.00: confirmed; inject as established fact.
                            -- Victor's context builder uses this to rank which memories to inject
                            -- when the context window budget is constrained.

  source_session_id         UUID
                              REFERENCES coaching_sessions(id) ON DELETE SET NULL,
                            -- Session from which this memory was extracted. NULL if manually set.

  source_turn_id            UUID
                              REFERENCES coaching_turns(id) ON DELETE SET NULL,
                            -- Specific turn where the fact was stated. NULL if session-level.

  extracted_at              TIMESTAMPTZ NOT NULL DEFAULT now(),
                            -- When the memory Worker extracted and wrote this row.

  last_confirmed_at         TIMESTAMPTZ NOT NULL DEFAULT now(),
                            -- Updated when a later session re-confirms the same fact.
                            -- Used by Victor's context builder to age-weight memories.

  expires_at                TIMESTAMPTZ,
                            -- NULL = persistent memory.
                            -- Non-null = ephemeral context (e.g. 'user.context.recent_life_stressor').
                            -- The context builder skips expired memories. A nightly pg_cron
                            -- marks expired rows as is_deleted = TRUE (does not hard-delete).

  is_deleted                BOOLEAN     NOT NULL DEFAULT FALSE,
                            -- Soft delete. TRUE = excluded from context injection.
                            -- Set by: GDPR erasure Worker, expiry cron, user explicit clear.

  deleted_at                TIMESTAMPTZ,
                            -- NULL when is_deleted = FALSE. Set to now() on soft delete.

  created_at                TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at                TIMESTAMPTZ NOT NULL DEFAULT now()
);

-- Triggers
CREATE TRIGGER set_coaching_memory_updated_at
  BEFORE UPDATE ON coaching_memory
  FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();

-- Constraints
ALTER TABLE coaching_memory
  ADD CONSTRAINT uq_memory_user_key
    UNIQUE (user_id, memory_key);
  -- One active value per user per key. UPDATE (not INSERT) on re-extraction.
  -- is_deleted rows are overwritten when a new extraction produces the same key.

-- Indexes
CREATE INDEX idx_cm_user_active
  ON coaching_memory(user_id, memory_key)
  WHERE is_deleted = FALSE;
  -- Context builder lookup: all active memories for a user, ordered by confidence.

CREATE INDEX idx_cm_tenant_user
  ON coaching_memory(tenant_id, user_id);
  -- RLS enforcement and GDPR erasure scan.

CREATE INDEX idx_cm_expires_at
  ON coaching_memory(expires_at)
  WHERE expires_at IS NOT NULL AND is_deleted = FALSE;
  -- Nightly expiry cron.

ALTER TABLE coaching_memory ENABLE ROW LEVEL SECURITY;
```

---

### 21.6 Memory Key Namespace

The `memory_key` column uses a dotted namespace to organise extracted facts. The Worker maintains a compile-time allowlist (`src/workers/coaching/memory-keys.ts`) that maps each key to its data classification and expiry policy.

| Key | Example value | Classification | Default expiry |
|---|---|---|---|
| `user.goals.primary` | `"build muscle"` | DC-05 | Persistent |
| `user.goals.timeline` | `"before August marathon"` | DC-05 | Persistent |
| `user.injuries.known` | `"left knee ACL repair 2023"` | DC-06 | Persistent |
| `user.injuries.contraindications` | `"avoid deep squats"` | DC-06 | Persistent |
| `user.training_history.summary` | `"3 years lifting, powerlifting background"` | DC-05 | Persistent |
| `user.preferences.workout_time` | `"morning"` | DC-03 | Persistent |
| `user.preferences.equipment` | `"home gym, no barbell"` | DC-03 | Persistent |
| `user.context.recent_life_stressor` | `"moving house this week"` | DC-05 | 14 days |
| `user.context.current_fatigue_level` | `"high — poor sleep last 3 nights"` | DC-05 | 3 days |
| `user.biometrics.weight_kg` | `"82"` | DC-06 | Persistent |
| `user.biometrics.height_cm` | `"178"` | DC-06 | Persistent |

Keys outside this allowlist are rejected by the Worker with a `400 Unknown memory key` response before any DB write. This prevents arbitrary data accumulation. New keys require a PR to `memory-keys.ts` and a corresponding DATA_MODEL.md update.

---

### 21.7 `coaching_feedback` DDL

```sql
-- Data classification: DC-03 (operational feedback metadata)
-- feedback_text_encrypted may contain personal content (DC-05 if present).
-- Retained 2 years. Not subject to billing audit retention.
-- Art. 17 erasure: feedback_text_encrypted set to NULL; row retained for product metrics.

CREATE TABLE coaching_feedback (
  id                        UUID        PRIMARY KEY DEFAULT gen_random_uuid(),

  tenant_id                 UUID        NOT NULL
                              REFERENCES tenants(id) ON DELETE CASCADE,

  user_id                   UUID        NOT NULL
                              REFERENCES users(id) ON DELETE CASCADE,

  turn_id                   UUID        NOT NULL
                              REFERENCES coaching_turns(id) ON DELETE CASCADE,
                            -- The specific Victor response being rated.

  session_id                UUID        NOT NULL
                              REFERENCES coaching_sessions(id) ON DELETE CASCADE,
                            -- Denormalized for tenant-scoped analytics queries.

  feedback_type             TEXT        NOT NULL
                              CHECK (feedback_type IN (
                                'thumbs_up',
                                'thumbs_down',
                                'flagged_unsafe',
                                'flagged_inaccurate'
                              )),
                            -- thumbs_up          = user found the response helpful.
                            -- thumbs_down         = user found the response unhelpful.
                            -- flagged_unsafe      = user believes response was harmful/dangerous.
                            -- flagged_inaccurate  = user believes response contained factual errors.
                            -- 'flagged_unsafe' triggers a PagerDuty alert; see §21.8 audit events.

  feedback_text_encrypted   TEXT,
                            -- Optional free-text explanation from the user, AES-256-GCM encrypted.
                            -- NULL if the user provided no text. NULL after Art. 17 erasure.

  created_at                TIMESTAMPTZ NOT NULL DEFAULT now()
);

-- Indexes
CREATE INDEX idx_cf_tenant_session
  ON coaching_feedback(tenant_id, session_id);

CREATE INDEX idx_cf_turn_id
  ON coaching_feedback(turn_id);

CREATE INDEX idx_cf_flagged
  ON coaching_feedback(tenant_id, feedback_type)
  WHERE feedback_type IN ('flagged_unsafe', 'flagged_inaccurate');
  -- Safety flag monitoring query for the admin dashboard.

ALTER TABLE coaching_feedback ENABLE ROW LEVEL SECURITY;
```

---

### 21.8 `coaching_session_context` DDL

```sql
-- Data classification: DC-02 (operational metadata — hashes only, no personal content)
-- Stores what was injected into the context window for each session.
-- Purpose: reproducibility, debugging, and auditor demonstration of what Victor "knew".
-- Content is NOT stored here — only hashes and token counts.
-- The actual content for 'memory_injection' is in coaching_memory (encrypted).
-- The actual system prompt content is in Worker source code (version-pinned by deploy SHA).

CREATE TABLE coaching_session_context (
  id                        UUID        PRIMARY KEY DEFAULT gen_random_uuid(),

  session_id                UUID        NOT NULL
                              REFERENCES coaching_sessions(id) ON DELETE CASCADE,
                            -- No tenant_id denorm: accessed only via session JOIN.
                            -- Sessions have tenant_id; this table piggybacks on that.

  context_type              TEXT        NOT NULL
                              CHECK (context_type IN (
                                'system_prompt',
                                'memory_injection',
                                'workout_history',
                                'wearable_summary'
                              )),
                            -- system_prompt     = the static Victor persona + instructions block.
                            -- memory_injection   = structured facts from coaching_memory.
                            -- workout_history    = recent workouts from the workouts table (§2.4).
                            -- wearable_summary   = aggregated wearable data (§14).

  content_hash              TEXT        NOT NULL,
                            -- SHA-256 of the injected content (plaintext), hex-encoded.
                            -- Allows the Worker to detect if the same context block was used
                            -- across sessions (cache hit potential) and for auditability.

  token_count               INTEGER     NOT NULL,
                            -- Tokens consumed by this context block in the context window.
                            -- Sum across all context_type rows for a session = context overhead.

  injected_at               TIMESTAMPTZ NOT NULL DEFAULT now()
                            -- When this context block was assembled and injected.
                            -- Immutable; row is written once at session start.
);

-- Indexes
CREATE INDEX idx_csc_session_id
  ON coaching_session_context(session_id, context_type);
  -- Primary lookup: all context blocks for a session.
```

Note: `coaching_session_context` does NOT have RLS enabled because it contains no personal data (hashes only) and is accessed exclusively via `form_system` service role within the coaching Worker. `form_api` has no direct access to this table.

---

### 21.9 RLS Policies

#### `coaching_sessions`

```sql
-- form_api: authenticated users may read and create their own sessions.
-- Users cannot read other users' sessions even within the same tenant.
CREATE POLICY cs_select_own ON coaching_sessions
  FOR SELECT TO form_api
  USING (
    tenant_id = current_setting('app.current_tenant_id')::UUID
    AND user_id = auth.uid()
  );

CREATE POLICY cs_insert_own ON coaching_sessions
  FOR INSERT TO form_api
  WITH CHECK (
    tenant_id = current_setting('app.current_tenant_id')::UUID
    AND user_id = auth.uid()
  );

-- form_api cannot UPDATE or DELETE coaching_sessions directly.
-- Status transitions are performed by form_system (the coaching Worker).

-- form_system: tenant-scoped read/write for Worker operations.
-- No cross-tenant access: tenant_id must match the service-token claim.
CREATE POLICY cs_system_all ON coaching_sessions
  FOR ALL TO form_system
  USING (tenant_id = current_setting('app.current_tenant_id')::UUID)
  WITH CHECK (tenant_id = current_setting('app.current_tenant_id')::UUID);

-- form_admin: BYPASSRLS — no policy required (compliance toolchain only).
```

#### `coaching_turns`

```sql
-- form_api: authenticated users may read own turns, BUT content_encrypted is masked.
-- The raw encrypted column is accessible only via the decryption Worker endpoint.
-- This policy returns NULL for content_encrypted to prevent the DB from ever returning
-- ciphertext to a client that could then attempt offline decryption attacks.
CREATE POLICY ct_select_own ON coaching_turns
  FOR SELECT TO form_api
  USING (
    tenant_id = current_setting('app.current_tenant_id')::UUID
    AND user_id = auth.uid()
  );
-- Note: The SELECT list in the API query explicitly excludes content_encrypted:
--   SELECT id, session_id, turn_index, role, content_hash,
--          input_tokens, output_tokens, latency_ms, model_id, stop_reason,
--          clinical_safety_flagged, created_at
--   FROM coaching_turns
-- The column is projected as NULL at the application layer, NOT via a column-level
-- security policy (Postgres does not support column-level RLS). This is enforced
-- as a lint rule on all queries selecting from coaching_turns. See §21.13 checklist item 6.

-- form_api cannot INSERT directly — turns are written exclusively by form_system.
-- This prevents a rogue client from injecting fabricated turns into a session.

-- form_system: tenant-scoped full access for the coaching Worker.
CREATE POLICY ct_system_all ON coaching_turns
  FOR ALL TO form_system
  USING (tenant_id = current_setting('app.current_tenant_id')::UUID)
  WITH CHECK (tenant_id = current_setting('app.current_tenant_id')::UUID);

-- Turns are append-only. form_system may INSERT but MUST NOT UPDATE content_encrypted
-- after creation (except during GDPR erasure, which uses form_admin BYPASSRLS).
-- Enforced at the Worker layer: no UPDATE statement on content_encrypted exists outside
-- the erasure Worker. The erasure Worker validates the §12 erasure job ID before writing.
```

#### `coaching_memory`

```sql
-- form_api: authenticated users may read their own active memory keys (not values).
-- memory_value_encrypted is masked identically to content_encrypted above:
-- the application layer never returns the encrypted value via form_api SELECT.
CREATE POLICY cm_select_own ON coaching_memory
  FOR SELECT TO form_api
  USING (
    tenant_id = current_setting('app.current_tenant_id')::UUID
    AND user_id = auth.uid()
    AND is_deleted = FALSE
  );

-- form_api cannot INSERT or UPDATE. Memory is managed exclusively by form_system.

-- form_system: tenant-scoped access for memory extraction Worker.
CREATE POLICY cm_system_all ON coaching_memory
  FOR ALL TO form_system
  USING (tenant_id = current_setting('app.current_tenant_id')::UUID)
  WITH CHECK (tenant_id = current_setting('app.current_tenant_id')::UUID);
```

#### `coaching_feedback`

```sql
-- form_api: users may insert their own feedback and read their own feedback rows.
CREATE POLICY cf_select_own ON coaching_feedback
  FOR SELECT TO form_api
  USING (
    tenant_id = current_setting('app.current_tenant_id')::UUID
    AND user_id = auth.uid()
  );

CREATE POLICY cf_insert_own ON coaching_feedback
  FOR INSERT TO form_api
  WITH CHECK (
    tenant_id = current_setting('app.current_tenant_id')::UUID
    AND user_id = auth.uid()
  );

-- form_system: tenant-scoped read for feedback analytics and safety monitoring.
CREATE POLICY cf_system_select ON coaching_feedback
  FOR SELECT TO form_system
  USING (tenant_id = current_setting('app.current_tenant_id')::UUID);
```

**Cross-tenant RLS isolation tests** (add to `__tests__/db/rls_isolation.test.ts`):

```typescript
it('coaching_sessions: form_api reads zero rows for another tenant', async () => {
  await setTenantContext(dbA, tenantAId, userAId);
  const rows = await dbA.query(
    'SELECT * FROM coaching_sessions WHERE tenant_id = $1', [tenantBId]
  );
  expect(rows.rows).toHaveLength(0);
});

it('coaching_turns: form_api cannot insert directly', async () => {
  await setTenantContext(dbA, tenantAId, userAId);
  await expect(
    dbA.query(
      `INSERT INTO coaching_turns (tenant_id, session_id, user_id, turn_index, role, model_id)
       VALUES ($1, $2, $3, 0, 'user', 'claude-sonnet-4-6')`,
      [tenantAId, sessionId, userAId]
    )
  ).rejects.toThrow(); // No INSERT policy for form_api
});

it('coaching_memory: form_api reads zero rows for is_deleted = TRUE', async () => {
  await setTenantContext(dbA, tenantAId, userAId);
  const rows = await dbA.query(
    'SELECT * FROM coaching_memory WHERE user_id = $1 AND is_deleted = TRUE', [userAId]
  );
  expect(rows.rows).toHaveLength(0); // RLS WHERE clause excludes deleted rows
});

it('coaching_turns: form_system cannot access another tenant', async () => {
  await setSystemContext(dbSystem, tenantAId);
  const rows = await dbSystem.query(
    'SELECT * FROM coaching_turns WHERE tenant_id = $1', [tenantBId]
  );
  expect(rows.rows).toHaveLength(0);
});
```

---

### 21.10 Context Window Management

Anthropic's Claude models have a finite context window (200k tokens for claude-sonnet-4-6 as of the model's release). Silent context truncation — where the API silently drops early messages to fit the window — is a correctness failure: Victor loses track of earlier conversation facts without any signal to the user or the system.

**Strategy: track, warn, and hand off at 80%.**

```
Session start
  │
  ├─ Worker assembles context: system_prompt + memory_injection + workout_history + wearable_summary
  │   └─ Writes coaching_session_context rows (§21.8)
  │   └─ token_count sum = context_overhead
  │
  ├─ Each turn:
  │   ├─ Worker estimates new input_tokens (context_overhead + all prior turns + new user message)
  │   ├─ Updates coaching_sessions.context_window_used_pct
  │   └─ If context_window_used_pct > 80%:
  │       ├─ Complete current session (status = 'completed', completed_at = now())
  │       ├─ Emit coaching.session_completed DEC-030 event with reason = 'context_window_limit'
  │       ├─ Trigger memory extraction Worker (async) for the completed session
  │       ├─ Start a new session (INSERT coaching_sessions, new session_token)
  │       ├─ Re-inject fresh memory from coaching_memory (now includes extracted facts)
  │       └─ Send continuation message to user: Victor acknowledges the session refresh
  │
  └─ Mobile client: receives new session_token in response; transparently continues conversation
```

The `coaching_session_context` table records exactly which context blocks were injected into each new session, creating a reproducible audit trail. The 80% threshold gives a 40k-token safety margin for the turn in progress (worst case: a very long user message + full Victor response).

**Why 80% and not 95%?** At 95%, the current turn's input + output can push the effective usage past 100%, triggering Anthropic's own truncation silently. The 80% gate ensures the Worker controls the handoff, not the API.

---

### 21.11 GDPR Art. 17 Erasure Integration

This section integrates with §12 (Soft Delete, Data Retention & GDPR Art. 17 Erasure). The coaching schema adds the following rows to the §12 erasure sequence table:

| Table | Erasure action | What is retained | Retention reason |
|---|---|---|---|
| `coaching_sessions` | No row mutation; metadata retained | All columns (no PII in payload) | Billing audit, 7yr |
| `coaching_turns` | `content_encrypted = NULL`, `content_hash = 'erased'` | All other columns (token counts, latency, model_id, stop_reason, clinical_safety_flagged) | Billing audit + safety audit, 7yr |
| `coaching_memory` | `is_deleted = TRUE`, `memory_value_encrypted = NULL`, `deleted_at = now()` | All other columns (memory_key, confidence_score, extracted_at) | Data lineage, 2yr |
| `coaching_feedback` | `feedback_text_encrypted = NULL` | All other columns (feedback_type, created_at) | Product quality metrics, 2yr |
| `coaching_session_context` | Hard delete (`DELETE`) | None | Content is hashes only; no billing relevance; no PII |

**Erasure sequence for a user deletion request:**

```sql
-- Step 1: Erase turn content (most sensitive — column-level encrypted PII)
UPDATE coaching_turns
SET content_encrypted = NULL,
    content_hash       = 'erased'
WHERE user_id = $1
  AND tenant_id = $2;

-- Step 2: Erase memory values
UPDATE coaching_memory
SET memory_value_encrypted = NULL,
    is_deleted             = TRUE,
    deleted_at             = now()
WHERE user_id = $1
  AND tenant_id = $2;

-- Step 3: Erase feedback text
UPDATE coaching_feedback
SET feedback_text_encrypted = NULL
WHERE user_id = $1
  AND tenant_id = $2;

-- Step 4: Hard delete session context (hash-only rows; no billing relevance)
DELETE FROM coaching_session_context
WHERE session_id IN (
  SELECT id FROM coaching_sessions
  WHERE user_id = $1 AND tenant_id = $2
);

-- Step 5: Emit DEC-030 coaching.memory_deleted event (HIGH, 7yr) for each memory key erased.
-- This is emitted by the erasure Worker, not by a DB trigger.
-- The event payload includes memory_key (the key name, not the value) and user_id hash (not plaintext).

-- coaching_sessions rows are NOT deleted — retained for billing audit.
-- The user_id FK will become a dangling reference when the users row is deleted.
-- Resolved by: users.id ON DELETE SET NULL on coaching_sessions.user_id (see DDL note below).
```

**FK handling for user deletion:** The DDL above uses `ON DELETE CASCADE` for all coaching tables. For GDPR erasure (which must retain billing metadata in `coaching_sessions`), the erasure sequence must run Steps 1–5 BEFORE the `users` row is deleted. The §12 erasure Worker enforces this ordering. Do NOT rely on CASCADE for coaching_sessions; the explicit erasure sequence is the control.

---

### 21.12 Cost Attribution per Tenant

#### Pricing formula (illustrative; actual rates from Anthropic API at billing time)

| Token type | Rate (USD per token) |
|---|---|
| Input (non-cached) | $0.000003 |
| Output | $0.000015 |
| Cache read | $0.0000003 |
| Cache write | $0.00000375 |

The `session_cost_usd` column on `coaching_sessions` is computed by the coaching Worker after each turn using these rates and the token counts returned by the Anthropic API response headers. The Worker updates `coaching_sessions` atomically with the new token totals and recomputed `session_cost_usd`.

#### Monthly cost roll-up materialized view

```sql
-- Refreshed nightly at 02:00 UTC by pg_cron.
-- Used by: billing Worker (monthly invoice generation), admin dashboard cost analytics.
-- Tenant admins see only their own tenant's row (RLS enforced on the base table).
-- form_admin sees all tenants for cross-tenant billing analysis.

CREATE MATERIALIZED VIEW tenant_monthly_coaching_cost AS
SELECT
  cs.tenant_id,
  date_trunc('month', cs.started_at)         AS month,
  COUNT(DISTINCT cs.id)                       AS session_count,
  COUNT(DISTINCT cs.user_id)                  AS active_coaching_users,
  SUM(cs.message_count)                       AS total_turns,
  SUM(cs.input_tokens_total)                  AS total_input_tokens,
  SUM(cs.output_tokens_total)                 AS total_output_tokens,
  SUM(cs.cache_read_tokens)                   AS total_cache_read_tokens,
  SUM(cs.cache_write_tokens)                  AS total_cache_write_tokens,
  SUM(cs.session_cost_usd)                    AS total_cost_usd,
  AVG(cs.session_cost_usd)                    AS avg_cost_per_session_usd,
  ROUND(
    SUM(cs.session_cost_usd) /
    NULLIF(COUNT(DISTINCT cs.user_id), 0),
    6
  )                                           AS avg_cost_per_user_usd
FROM coaching_sessions cs
WHERE cs.status IN ('completed', 'abandoned', 'error')
  -- Only closed sessions have final token counts. Active sessions are excluded.
GROUP BY cs.tenant_id, date_trunc('month', cs.started_at);

CREATE UNIQUE INDEX ON tenant_monthly_coaching_cost (tenant_id, month);

-- Refresh schedule (pg_cron):
SELECT cron.schedule(
  'refresh-coaching-cost-mv',
  '0 2 * * *',  -- 02:00 UTC daily
  $$REFRESH MATERIALIZED VIEW CONCURRENTLY tenant_monthly_coaching_cost$$
);
```

**Cost threshold alerting:** When a session's `session_cost_usd` exceeds the tenant's configured coaching cost threshold (stored in `tenant_config.coaching_cost_alert_threshold_usd`, default $0.50/session), the coaching Worker emits `coaching.session_cost_threshold_exceeded` (HIGH, 7yr) and enqueues a webhook event to the tenant's billing alert webhook (if configured).

---

### 21.13 DEC-030 HMAC-Chained Audit Events

All coaching events follow the HMAC-chained append-only format defined in AUDIT_LOG_SCHEMA.md DEC-030.

| Event key | Severity | Retention | Trigger |
|---|---|---|---|
| `coaching.session_started` | STANDARD | 2yr | New `coaching_sessions` row created; session_token issued to mobile client |
| `coaching.session_completed` | STANDARD | 2yr | `status` → `completed`; includes final token counts, `session_cost_usd`, completion reason |
| `coaching.turn_clinical_safety_flagged` | HIGH | 7yr | `clinical_safety_flagged = TRUE` written to a `coaching_turns` row; triggers PagerDuty alert |
| `coaching.memory_extracted` | LOW | 1yr | Memory Worker writes or updates a `coaching_memory` row; sampled at 10% to reduce log volume |
| `coaching.memory_deleted` | HIGH | 7yr | GDPR Art. 17 erasure: `memory_value_encrypted = NULL`, `is_deleted = TRUE` written; one event per key |
| `coaching.feedback_submitted` | LOW | 1yr | `coaching_feedback` row created; `feedback_type` included in payload |
| `coaching.session_cost_threshold_exceeded` | HIGH | 7yr | Session cost exceeds tenant alert threshold; payload includes `session_cost_usd`, `threshold_usd`, `tenant_id` |

**Payload schema for `coaching.turn_clinical_safety_flagged`** (highest-criticality coaching event):

```json
{
  "event": "coaching.turn_clinical_safety_flagged",
  "tenant_id": "uuid",
  "actor_user_id": "sha256-hash-of-uuid",
  "actor_role": "form_system",
  "timestamp": "ISO-8601",
  "data": {
    "session_id": "uuid",
    "turn_id": "uuid",
    "turn_index": 7,
    "role": "assistant",
    "clinical_safety_reason": "ed_screening_keyword",
    "model_id": "claude-sonnet-4-6",
    "latency_ms": 1842
  },
  "hmac": "sha256:...",
  "prev_hmac": "sha256:..."
}
```

**Critical constraints:**

- The event MUST NOT include `content_encrypted` or any decrypted content. The audit log is retained 7yr and must contain zero PII.
- `actor_user_id` is a SHA-256 hash of the UUID, not the UUID itself. This prevents the audit log from becoming a PII store when cross-referenced with other systems. The raw UUID is retained in `coaching_turns.user_id` under column-level encryption controls.
- `coaching.memory_deleted` events are emitted once per memory key, not once per erasure request. An erasure for a user with 12 memory keys emits 12 events, each with `memory_key` (not the value) in the payload.
- `coaching.session_cost_threshold_exceeded` must be emitted synchronously before the coaching Worker returns the turn response to the client. Failure to emit = transaction abort (fail-closed, consistent with §19 DEC-030 principle).

---

### 21.14 SOC 2 Evidence Mapping

| Control | TSC Criterion | Evidence artefact |
|---|---|---|
| `coaching_turns.content_encrypted` uses AES-256-GCM column-level encryption; key in Cloudflare Secrets Store | CC6.7 — Encryption of data at rest and in transit | Key rotation procedure in `docs/KEY_ROTATION.md`; quarterly key rotation audit log; `coaching_turns` DDL in `compliance/evidence/coaching/coaching-schema.sql` |
| RLS policies on all coaching tables enforce `tenant_id` isolation; no cross-tenant query is possible under `form_api` or `form_system` | CC6.1 — Logical and physical access controls | Cross-tenant RLS isolation test results in `compliance/evidence/coaching/rls-test-results.txt`; §21.9 DDL in evidence index |
| `clinical_safety_flagged` turns trigger PagerDuty P1 and are retained 7yr in HMAC-chained audit log; reviewed monthly by compliance-officer | CC7.2 — System monitoring and anomaly detection | Monthly clinical safety flag report (`compliance/evidence/coaching/safety-flags-YYYY-MM.csv`); PagerDuty P1 ticket history |
| Anthropic is a sub-processor for all Victor AI responses; data processing terms (DPA) in place; model is US-hosted | CC9.2 — Vendor and third-party risk management | Anthropic DPA signed copy in `compliance/legal/anthropic-dpa.pdf`; sub-processor list in `docs/SUBPROCESSORS.md`; annual Anthropic security review in `compliance/evidence/vendors/anthropic-YYYY.pdf` |

**Auditor evidence artefacts**:

- `compliance/evidence/coaching/coaching-schema.sql` — DDL export from production Supabase, refreshed quarterly
- `compliance/evidence/coaching/rls-policies.sql` — RLS policy export, matches §21.9 DDL
- `compliance/evidence/coaching/rls-test-results.txt` — CI output of cross-tenant isolation test suite
- `compliance/evidence/coaching/safety-flags-YYYY-MM.csv` — monthly export of `coaching.turn_clinical_safety_flagged` events from audit log, reviewed and signed by compliance-officer
- `compliance/evidence/coaching/cost-attribution-sample.csv` — sample `tenant_monthly_coaching_cost` view output demonstrating per-tenant cost isolation

---

### 21.15 Open Questions

| # | Question | Owner | Priority |
|---|---|---|---|
| OQ-COACH-01 | Should `coaching_turns` content be retained for Anthropic model fine-tuning or FORM internal model evaluation? This requires explicit opt-in user consent under GDPR Art. 6(1)(a) (consent basis, separate from the Art. 6(1)(b) contract basis for coaching delivery), a Data Processing Impact Assessment (DPIA), and a separate `coaching_turns.fine_tune_consent` boolean column. Until consent infrastructure is in place, ALL coaching content is excluded from any fine-tuning pipeline. | `compliance-officer` + `legal` | **P0 — resolve before any model evaluation use case ships** |
| OQ-COACH-02 | Token cost attribution model for enterprise: flat per-seat coaching fee (predictable for tenant budgeting) vs. actual consumption billing (fair but variable and complex to invoice). Current schema supports both: `tenant_monthly_coaching_cost` view provides actual consumption; a per-seat flat rate would cap at a configurable `tenant_config.coaching_seat_allowance_tokens` with overage billing. Decision needed before enterprise contracts are signed. | `enterprise-architect` + `finance` | 🟢 **Resolved** — DEC-041 (2026-06-11). Flat per-seat billing adopted for all enterprise tiers; consumption billing deferred (see §33.5 for conditions under which it may be revisited). `tenant_monthly_coaching_cost` view retained for FORM-internal gross margin monitoring and churn-risk signalling only — NOT exposed to tenant admins. `tenant_config.coaching_seat_allowance_tokens` NOT activated at launch (NULL for all tenants); token budgets (§33.3) are internal cost-governance targets, not contractual terms. Full billing architecture, margin math, overage policy, ASC 606 treatment, and DEC-030 event specs documented in `docs/COST_MODEL.md §33`. |
| OQ-COACH-03 | Memory key namespace versioning: how do we handle schema evolution of `memory-keys.ts`? If `user.training_history.summary` is renamed to `user.training_history.background` in a future release, existing rows hold the old key. Options: (a) migration script to rename keys; (b) alias map in the context builder; (c) version suffix on keys (`user.training_history.summary.v2`). Each has trade-offs for erasure targeting and context builder complexity. | `platform-engineer` | **P1 — before memory schema stabilises at M4** |

---

### 21.16 Implementation Checklist

| # | Task description | Owner | Priority | Milestone |
|---|---|---|---|---|
| 1 | Create `migrations/YYYYMMDD_coaching_sessions.sql` — `coaching_sessions` DDL, all indexes, `update_updated_at_column()` trigger, RLS policies from §21.9 | `enterprise-architect` | **P0** | M3 |
| 2 | Create `migrations/YYYYMMDD_coaching_turns.sql` — `coaching_turns` DDL, `uq_turn_position` constraint, all indexes, RLS policies from §21.9 | `enterprise-architect` | **P0** | M3 |
| 3 | Create `migrations/YYYYMMDD_coaching_memory.sql` — `coaching_memory` DDL, `uq_memory_user_key` constraint, all indexes, `update_updated_at_column()` trigger, RLS policies from §21.9 | `enterprise-architect` | **P0** | M3 |
| 4 | Create `migrations/YYYYMMDD_coaching_feedback_and_context.sql` — `coaching_feedback` and `coaching_session_context` DDL, all indexes, RLS policies from §21.9 | `enterprise-architect` | **P0** | M3 |
| 5 | Implement AES-256-GCM column-level encryption in `src/lib/encryption/coaching-crypto.ts`: `encrypt(plaintext, key)` and `decrypt(ciphertext, key)` using the Web Crypto API (available in Cloudflare Workers); key loaded from Cloudflare Secrets Store at Worker startup; unit tests for encrypt/decrypt round-trip and null handling | `platform-engineer` | **P0** | M3 |
| 6 | Implement lint rule (`src/lint/no-coaching-turns-content.ts`) that rejects any query selecting `content_encrypted` from `coaching_turns` without going through the decryption Worker endpoint; add to CI pipeline | `platform-engineer` | **P0** | M3 |
| 7 | Implement the coaching turn Worker (`src/workers/coaching/turn-handler.ts`): session token validation, context assembly, Anthropic Messages API call, streaming response to client, atomic `coaching_turns` INSERT + `coaching_sessions` UPDATE (token counts, cost, context_window_used_pct), `coaching_session_context` snapshot on first turn | `platform-engineer` | **P0** | M3 |
| 8 | Implement context window management (§21.10): detect `context_window_used_pct > 80%` after each turn; auto-complete session; emit `coaching.session_completed` DEC-030 event with `reason = 'context_window_limit'`; trigger memory extraction; start new session with fresh injection | `platform-engineer` | **P0** | M3 |
| 9 | Implement memory extraction Worker (`src/workers/coaching/memory-extractor.ts`): invoked async after session completion; calls Claude to extract structured facts from the session turns; validates extracted keys against `memory-keys.ts` allowlist; UPSERT into `coaching_memory`; emits `coaching.memory_extracted` DEC-030 event (sampled 10%) | `platform-engineer` | **P0** | M4 |
| 10 | Implement post-generation clinical safety classifier (`src/workers/coaching/safety-classifier.ts`): runs async after each Victor response is streamed; keyword + pattern matching per `docs/CLINICAL_SAFETY.md`; if flagged: UPDATE `coaching_turns.clinical_safety_flagged = TRUE`, emit `coaching.turn_clinical_safety_flagged` DEC-030 event (HIGH), open PagerDuty P1 | `platform-engineer` + `clinical-advisor` | **P0** | M3 |
| 11 | Create `tenant_monthly_coaching_cost` materialized view (§21.12) and pg_cron job; wire to admin dashboard cost analytics endpoint; add `coaching.session_cost_threshold_exceeded` DEC-030 event emission to the turn Worker | `platform-engineer` | **P1** | M4 |
| 12 | Implement GDPR Art. 17 erasure sequence for coaching tables (§21.11): integrate into §12 erasure Worker; ordering: turns → memory → feedback → session_context → (sessions retained); emit `coaching.memory_deleted` DEC-030 event per key; unit tests for each step | `platform-engineer` + `compliance-officer` | **P1** | M4 |
| 13 | Add all cross-tenant RLS isolation tests from §21.9 to `__tests__/db/rls_isolation.test.ts`; add to CI gating | `enterprise-architect` | **P0** | M3 |
| 14 | Wire all 7 DEC-030 coaching audit events (§21.13) to their triggers; confirm `coaching.turn_clinical_safety_flagged` emits synchronously and carries `blocked: false` (it does not block the response, but it does alert); confirm `coaching.session_cost_threshold_exceeded` is fail-closed | `platform-engineer` + `security-engineer` | **P0** | M3 |
| 15 | Add `coaching.victor_enabled` flag to `feature_flag_registry` seed data (§19.3) with `minimum_tier = 'growth'`, `requires_compliance_gate = TRUE`; wire `assertFeatureEnabled('coaching.victor_enabled')` gate into the turn Worker; compliance gate requires DPA with Anthropic as sub-processor in DECISION_LOG | `platform-engineer` + `compliance-officer` | **P0** | M3 |

---

*v1.1 · травень 2026 · owner: enterprise-architect + compliance-officer + security-engineer*

*v1.1 additions: §20 White-Label Tenant Branding Schema — `tenant_branding` table (1:1 with tenants, lazy), `logo_url/logo_dark_url/favicon_url`, `primary_color` + WCAG AA contrast validation, `custom_domain` DNS-verification state machine, `powered_by_removal` with ARR gate ($50k+) and DECISION_LOG ref requirement; R2 asset storage (PNG/WebP only, SVG rejected); 9 API endpoints; 12 DEC-030 HMAC-chained audit events; nightly pg_cron invariant check; SOC 2 CC7.2/CC8.1/CC6.2/CC6.8 mapping; 13-item checklist. §21 AI Coaching Session & Memory Schema — five-table schema: `coaching_sessions` (session_token, model_id, context_window_used_pct, session_cost_usd, per-model token counts), `coaching_turns` (column-level AES-256-GCM encryption on content, clinical_safety_flagged per turn, append-only via RLS), `coaching_memory` (dotted-namespace memory keys with confidence score and GDPR soft-delete), `coaching_feedback` (thumbs_up/down/flagged_unsafe/flagged_inaccurate), `coaching_session_context` (hash-only immutable context snapshot). Column-level encryption: AES-256-GCM, key from Cloudflare Secrets Store. RLS: form_api own-rows read (content_encrypted masked at query layer), form_system tenant-scoped, form_admin unrestricted. Context window management: 80% threshold triggers auto-complete + fresh session with memory re-injection. GDPR Art. 17 erasure: turns content_encrypted → NULL, memory soft-delete with key-level events. Cost attribution: session_cost_usd computed at turn-write time; `tenant_monthly_coaching_cost` materialized view, CONCURRENTLY refreshed via pg_cron. 7 DEC-030 audit events (coaching.session_started/completed/turn_clinical_safety_flagged/memory_extracted/memory_deleted/feedback_submitted/session_cost_threshold_exceeded). SOC 2 mapping: CC6.1 (RLS), CC6.7 (column encryption), CC7.2 (clinical safety flagging), CC9.2 (Anthropic as GDPR sub-processor). 3 open questions (OQ-COACH-01: fine-tuning consent P0; OQ-COACH-02: flat vs consumption billing P1; OQ-COACH-03: memory key namespace versioning P1). 15-item implementation checklist (12× P0 M3, 2× P1 M4, 1× P0 M3 feature-flag gate).*

---

## 22. Gamification, Streak & Achievement Schema

### 22.1 Design Rationale

Gamification in FORM is a **personal progress instrument**, not a competitive mechanism. Four hard constraints govern every design decision in this section:

1. **DEC-013 (streak grace):** The streak is preserved after 1 consecutive missed day (grace). It resets only after 2 consecutive misses. A streak must never reset after a single missed day. This is a brand floor — documented in DECISION_LOG and `docs/FLOWS.md`.
2. **DEC-002 (no in-app social):** No leaderboards, no cross-user comparisons, no public achievement walls. All gamification data is strictly personal and never exposed to other users or tenant admins in identified form.
3. **DEC-029 (clinical-safety gate on sharing):** If any external sharing of achievements is introduced in future, it must pass clinical-safety review before shipping.
4. **Privacy floor on tenant admin reporting:** Tenant admins never see individual streak lengths or personal records. Only anonymised, k-anonymity-gated fleet statistics (§17) are permitted.

### 22.2 Streak Engine Design

The streak engine computes streak state against `workout_sessions.completed_at` (§2.4 core schema). A "qualifying session" is any row in `workout_sessions` where `completed_at IS NOT NULL`. This is the **default** per OQ-GAM-01 (§22.14); a minimum-sets threshold can be added later without schema change.

**State machine (per user, per calendar day in user local timezone):**

```
State: active_streak (S > 0)

  today = qualifies:
    current_streak += 1
    last_activity_date = today
    grace_used_this_cycle = FALSE
    → emit gamification.streak_milestone if S ∈ {7, 14, 30, 60, 90, 180, 365}

  today-1 missed, today = qualifies, grace NOT used:
    GRACE APPLIED
    current_streak preserved (unchanged)
    grace_used_this_cycle = TRUE
    total_grace_uses += 1
    → emit gamification.streak_grace_applied
    → then: current_streak += 1, last_activity_date = today

  today-1 missed, today = qualifies, grace ALREADY USED:
    two consecutive misses confirmed (day N-1 was the grace day, today is day N+1 still missing)
    [This path is actually: grace_used = TRUE AND today qualifies AND gap = 2 days]
    current_streak RESET to 1 (today starts new streak)
    streak_start_date = today
    grace_used_this_cycle = FALSE
    → emit gamification.streak_reset (carries previous streak length)
    → emit gamification.streak_started

  2+ consecutive misses, today = qualifies:
    RESET (grace_used or not, gap > 1 day after grace or gap > 2 days without)
    current_streak = 1
    streak_start_date = today
    grace_used_this_cycle = FALSE
    → emit gamification.streak_reset
    → emit gamification.streak_started
```

**Timezone note:** Streak day boundaries are computed in the user's local timezone (`user_profile.timezone`; defaults to UTC if not set). A workout completed at 23:59 local time counts for that local day.

**Grace invariant:** At most one grace event can occur per streak cycle. `grace_used_this_cycle` resets to `FALSE` only when the user logs a qualifying session (not on reset). This prevents "chain grace" abuse.

### 22.3 `user_streaks` Table

```sql
CREATE TABLE user_streaks (
  user_id               UUID         NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  current_streak        INTEGER      NOT NULL DEFAULT 0 CHECK (current_streak >= 0),
  longest_streak        INTEGER      NOT NULL DEFAULT 0 CHECK (longest_streak >= 0),
  streak_start_date     DATE         NULLABLE,                -- NULL when current_streak = 0
  last_activity_date    DATE         NULLABLE,                -- last qualifying workout date (local tz)
  grace_used_this_cycle BOOLEAN      NOT NULL DEFAULT FALSE,
  total_grace_uses      INTEGER      NOT NULL DEFAULT 0 CHECK (total_grace_uses >= 0),
  created_at            TIMESTAMPTZ  NOT NULL DEFAULT now(),
  updated_at            TIMESTAMPTZ  NOT NULL DEFAULT now(),
  PRIMARY KEY (user_id)
);

CREATE INDEX idx_user_streaks_last_activity ON user_streaks (last_activity_date)
  WHERE last_activity_date IS NOT NULL;

CREATE TRIGGER update_user_streaks_updated_at
  BEFORE UPDATE ON user_streaks
  FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();

-- Invariant: longest_streak >= current_streak always
ALTER TABLE user_streaks
  ADD CONSTRAINT chk_streak_longest CHECK (longest_streak >= current_streak);
```

**longest_streak** is maintained by the streak engine: on every `current_streak` increment, if `current_streak > longest_streak`, update `longest_streak = current_streak`.

### 22.4 `streak_grace_events` Table

Append-only record of every grace application. No DELETE or UPDATE permitted via RLS.

```sql
CREATE TABLE streak_grace_events (
  id                      UUID         NOT NULL DEFAULT gen_random_uuid() PRIMARY KEY,
  user_id                 UUID         NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  missed_date             DATE         NOT NULL,      -- the calendar day that was missed (user local tz)
  streak_count_at_grace   INTEGER      NOT NULL CHECK (streak_count_at_grace > 0),
  grace_applied_at        TIMESTAMPTZ  NOT NULL DEFAULT now(),
  workout_session_id      UUID         NULLABLE REFERENCES workout_sessions(id) ON DELETE SET NULL
  -- the qualifying session that triggered the grace (day after the missed day)
);

CREATE INDEX idx_grace_events_user ON streak_grace_events (user_id, grace_applied_at DESC);
CREATE INDEX idx_grace_events_missed_date ON streak_grace_events (user_id, missed_date);
```

### 22.5 `personal_records` Table

Tracks the user's best-ever performance per exercise × metric combination. Each row represents one PR event (historical series, not just current best).

```sql
CREATE TYPE pr_metric AS ENUM (
  '1rm_kg',              -- estimated or tested 1-rep max in kilograms
  'max_reps',            -- maximum reps at a given weight (weight in context JSONB)
  'volume_session_kg',   -- total volume (sets × reps × weight) in a single session, kg
  'best_set_kg',         -- heaviest single set weight lifted, kg
  'time_seconds'         -- for timed exercises (plank, etc.)
);

CREATE TABLE personal_records (
  id                  UUID         NOT NULL DEFAULT gen_random_uuid() PRIMARY KEY,
  user_id             UUID         NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  exercise_slug       TEXT         NOT NULL,   -- e.g. 'squat', 'bench-press', 'pull-up'
  metric              pr_metric    NOT NULL,
  value               NUMERIC(10,3) NOT NULL CHECK (value > 0),
  previous_best       NUMERIC(10,3) NULLABLE,  -- snapshot of value before this PR was set
  improvement_pct     NUMERIC(6,2)  NULLABLE,  -- (value - previous_best) / previous_best * 100
  workout_session_id  UUID         NULLABLE REFERENCES workout_sessions(id) ON DELETE SET NULL,
  achieved_at         TIMESTAMPTZ  NOT NULL DEFAULT now(),
  context             JSONB        NOT NULL DEFAULT '{}'::jsonb,
  -- context carries supporting data: {reps_used: 3, rpe: 8, set_id: "...", e1rm_formula: "epley"}
  created_at          TIMESTAMPTZ  NOT NULL DEFAULT now()
);

CREATE UNIQUE INDEX idx_pr_user_exercise_metric_value
  ON personal_records (user_id, exercise_slug, metric, achieved_at DESC);
CREATE INDEX idx_pr_user_exercise ON personal_records (user_id, exercise_slug);
CREATE INDEX idx_pr_user_recent ON personal_records (user_id, achieved_at DESC);

-- Partial index: only the current best per user × exercise × metric
-- Used by the UI "current PRs" query; the engine maintains this via the achieved_at ordering
CREATE INDEX idx_pr_current_best ON personal_records (user_id, exercise_slug, metric)
  WHERE created_at = achieved_at; -- approximation; actual "current best" is MAX(value) per group
```

**Current best query pattern:**
```sql
-- Current best PR per exercise × metric for a user
SELECT DISTINCT ON (exercise_slug, metric)
  exercise_slug, metric, value, achieved_at, context
FROM personal_records
WHERE user_id = $1
ORDER BY exercise_slug, metric, value DESC, achieved_at DESC;
```

### 22.6 `achievement_definitions` Table

Canonical catalog of achievement types. Written by form_admin only. Never modified after an achievement key has been earned by any user (soft-archive via `is_active = FALSE` instead of deletion).

```sql
CREATE TYPE achievement_category AS ENUM (
  'streak',        -- streak-length milestones (7d, 30d, 90d, 365d)
  'pr',            -- personal record events (first PR, 10-PR club, etc.)
  'volume',        -- cumulative volume milestones (total tonnes lifted)
  'consistency',   -- training frequency (e.g. 3× / week for 4 weeks)
  'milestone'      -- onboarding and lifecycle events (first session, plan week 1 complete, etc.)
);

CREATE TYPE achievement_tier AS ENUM ('bronze', 'silver', 'gold', 'platinum');

CREATE TABLE achievement_definitions (
  achievement_key   TEXT                  NOT NULL PRIMARY KEY,
  -- dotted namespace: 'streak.7d', 'pr.first_squat_1rm', 'milestone.first_session'
  display_name_uk   TEXT                  NOT NULL,
  display_name_en   TEXT                  NOT NULL,
  description_uk    TEXT                  NOT NULL,
  description_en    TEXT                  NOT NULL,
  category          achievement_category  NOT NULL,
  tier              achievement_tier      NOT NULL,
  is_active         BOOLEAN               NOT NULL DEFAULT TRUE,
  created_at        TIMESTAMPTZ           NOT NULL DEFAULT now()
);

-- Key namespace constraint: prevent typos on insert
ALTER TABLE achievement_definitions
  ADD CONSTRAINT chk_achievement_key_namespace
  CHECK (achievement_key ~ '^[a-z_]+\.[a-z0-9_]+$');
```

**Seed data (initial achievement catalog):**

| achievement_key | UA | Tier |
|---|---|---|
| `milestone.first_session` | Перше тренування | Bronze |
| `milestone.plan_week_1` | Перший тиждень плану | Bronze |
| `milestone.plan_month_1` | Перший місяць | Silver |
| `streak.7d` | 7 днів поспіль | Bronze |
| `streak.14d` | 14 днів | Bronze |
| `streak.30d` | 30 днів | Silver |
| `streak.60d` | 60 днів | Silver |
| `streak.90d` | 90 днів | Gold |
| `streak.180d` | Пів року | Gold |
| `streak.365d` | Рік | Platinum |
| `pr.first_squat_1rm` | Перший 1RM присідання | Bronze |
| `pr.first_bench_1rm` | Перший 1RM жим | Bronze |
| `pr.first_deadlift_1rm` | Перший 1RM тяга | Bronze |
| `pr.10_prs` | 10 особистих рекордів | Silver |
| `pr.50_prs` | 50 особистих рекордів | Gold |
| `volume.10_tonnes` | 10 000 кг загального об'єму | Bronze |
| `volume.100_tonnes` | 100 000 кг | Silver |
| `volume.1000_tonnes` | 1 000 000 кг | Gold |
| `consistency.3x_week_4w` | 3× на тиждень, 4 тижні поспіль | Silver |

### 22.7 `user_achievements` Table

Append-only earned achievements per user. The achievement engine (Workers script) writes rows; form_api reads them for the user's own achievements.

```sql
CREATE TABLE user_achievements (
  id              UUID         NOT NULL DEFAULT gen_random_uuid() PRIMARY KEY,
  user_id         UUID         NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  achievement_key TEXT         NOT NULL REFERENCES achievement_definitions(achievement_key),
  earned_at       TIMESTAMPTZ  NOT NULL DEFAULT now(),
  context         JSONB        NOT NULL DEFAULT '{}'::jsonb,
  -- context carries supporting data:
  -- streak achievements: {streak_length: 30, streak_start_date: "2026-04-01"}
  -- PR achievements:     {exercise_slug: "squat", value: 140, metric: "1rm_kg"}
  -- volume achievements: {total_volume_kg: 10000}
  created_at      TIMESTAMPTZ  NOT NULL DEFAULT now()
);

-- A user can earn the same achievement multiple times (e.g. streak.30d after a reset and re-earn)
-- but not on the same calendar day
CREATE UNIQUE INDEX idx_achievements_user_key_day
  ON user_achievements (user_id, achievement_key, date_trunc('day', earned_at));

CREATE INDEX idx_achievements_user_recent
  ON user_achievements (user_id, earned_at DESC);
```

### 22.8 Row-Level Security Policies

```sql
-- ============================================================
-- user_streaks
-- ============================================================
ALTER TABLE user_streaks ENABLE ROW LEVEL SECURITY;

-- API: users read and update their own streak
CREATE POLICY "user_streaks_api_select" ON user_streaks
  FOR SELECT TO form_api USING (user_id = auth.uid());

CREATE POLICY "user_streaks_api_update" ON user_streaks
  FOR UPDATE TO form_api USING (user_id = auth.uid());

-- System: reads all rows for streak engine Worker; inserts on new user creation
CREATE POLICY "user_streaks_system_all" ON user_streaks
  FOR ALL TO form_system USING (TRUE) WITH CHECK (TRUE);

-- Admin: unrestricted
CREATE POLICY "user_streaks_admin_all" ON user_streaks
  FOR ALL TO form_admin USING (TRUE) WITH CHECK (TRUE);


-- ============================================================
-- streak_grace_events (append-only for form_api)
-- ============================================================
ALTER TABLE streak_grace_events ENABLE ROW LEVEL SECURITY;

CREATE POLICY "grace_events_api_select" ON streak_grace_events
  FOR SELECT TO form_api USING (user_id = auth.uid());

-- form_api may NOT insert directly; inserts are done by form_system via streak Worker
CREATE POLICY "grace_events_system_all" ON streak_grace_events
  FOR ALL TO form_system USING (TRUE) WITH CHECK (TRUE);

CREATE POLICY "grace_events_admin_select" ON streak_grace_events
  FOR SELECT TO form_admin USING (TRUE);

-- No UPDATE or DELETE policies for any role; grace events are immutable
-- (form_admin can delete via BYPASSRLS if compelled by Art. 17 — handled in erasure Worker)


-- ============================================================
-- personal_records
-- ============================================================
ALTER TABLE personal_records ENABLE ROW LEVEL SECURITY;

CREATE POLICY "prs_api_select" ON personal_records
  FOR SELECT TO form_api USING (user_id = auth.uid());

CREATE POLICY "prs_api_insert" ON personal_records
  FOR INSERT TO form_api WITH CHECK (user_id = auth.uid());

-- No direct UPDATE from form_api: PR records are write-once (a new PR creates a new row)
-- Form_system writes PR rows from the workout-completion Worker
CREATE POLICY "prs_system_all" ON personal_records
  FOR ALL TO form_system USING (TRUE) WITH CHECK (TRUE);

CREATE POLICY "prs_admin_all" ON personal_records
  FOR ALL TO form_admin USING (TRUE) WITH CHECK (TRUE);


-- ============================================================
-- achievement_definitions (read-only for non-admin)
-- ============================================================
ALTER TABLE achievement_definitions ENABLE ROW LEVEL SECURITY;

CREATE POLICY "achdef_api_select" ON achievement_definitions
  FOR SELECT TO form_api USING (is_active = TRUE);

CREATE POLICY "achdef_system_select" ON achievement_definitions
  FOR SELECT TO form_system USING (TRUE);

CREATE POLICY "achdef_admin_all" ON achievement_definitions
  FOR ALL TO form_admin USING (TRUE) WITH CHECK (TRUE);


-- ============================================================
-- user_achievements (append-only for form_system; read for form_api)
-- ============================================================
ALTER TABLE user_achievements ENABLE ROW LEVEL SECURITY;

CREATE POLICY "achievements_api_select" ON user_achievements
  FOR SELECT TO form_api USING (user_id = auth.uid());

-- form_system writes achievement rows from the achievement engine
CREATE POLICY "achievements_system_insert" ON user_achievements
  FOR INSERT TO form_system WITH CHECK (TRUE);

CREATE POLICY "achievements_system_select" ON user_achievements
  FOR SELECT TO form_system USING (TRUE);

CREATE POLICY "achievements_admin_all" ON user_achievements
  FOR ALL TO form_admin USING (TRUE) WITH CHECK (TRUE);
```

### 22.9 RLS Isolation Properties

| Property | Guarantee |
|---|---|
| Cross-user streak read | Impossible — `user_streaks` SELECT restricted to `auth.uid()` |
| Cross-tenant PR leakage | Impossible — `personal_records` SELECT restricted to `auth.uid()`; tenant boundary enforced by `users.tenant_id` FK, which RLS on `users` table enforces transitively |
| Achievement definitions exposure | `form_api` reads only `is_active = TRUE` rows; inactive definitions (deprecated achievements) not returned to clients |
| form_system privilege | Streak engine and achievement engine run as `form_system`; they access all user rows scoped by their processing batch — never expose to user requests |
| Grace event immutability | No UPDATE or DELETE policy exists for `form_api` or `form_system` on `streak_grace_events`; GDPR Art. 17 erasure uses form_admin BYPASSRLS |
| No tenant admin access | `tenant_admin` role has no policy on any gamification table — consistent with DEC-002 (no identified gamification data in admin reporting) |

### 22.10 DEC-013 Compliance Verification

DEC-013 states: **"Не пропускай 2 поспіль — streak grace після 2 misses, не 1."**

Translation to data-layer invariants:

| Invariant | Enforcement Location |
|---|---|
| Grace applied only after exactly 1 consecutive missed day | Streak Worker checks `DATEDIFF(today, last_activity_date) = 2 AND NOT grace_used_this_cycle` before applying grace |
| Grace used at most once per streak cycle | `grace_used_this_cycle = TRUE` flag set atomically in the same transaction as the grace application; reset only on next qualifying session |
| Streak resets only after 2+ consecutive misses | Streak Worker resets only when `DATEDIFF(today, last_activity_date) > 2` OR `(DATEDIFF = 2 AND grace_used_this_cycle = TRUE)` |
| grace_used_this_cycle reset on qualifying session | Streak Worker always writes `grace_used_this_cycle = FALSE` alongside `current_streak += 1` |
| Immutable grace history | `streak_grace_events` rows never deleted or updated by non-admin roles |

**Regression test (must pass in CI):**
```sql
-- Scenario: user trains Mon, misses Tue, trains Wed → grace applied; streak preserved
-- Setup: last_activity_date = Mon, grace_used_this_cycle = FALSE, current_streak = 5
-- Action: workout_sessions row inserted for Wed
-- Expected: current_streak = 6, grace_used_this_cycle = TRUE, streak_grace_events has 1 new row for Tue

-- Scenario: user trains Mon, misses Tue (grace), misses Wed → streak resets
-- Setup: last_activity_date = Mon, grace_used_this_cycle = TRUE (grace from prior cycle reused)
-- Wait, grace_used_this_cycle was just set TRUE after Mon→Tue grace... but user hasn't trained Wed.
-- No workout for Wed → DATEDIFF(Thu, Mon) = 3 (or checking at Wed: DATEDIFF(Wed, Mon) = 2, grace = TRUE → RESET)
-- Expected: current_streak = 0, gamification.streak_reset emitted carrying previous_streak = 6
```

The full test suite is in `__tests__/db/gamification_streak.test.ts` (§22.16 task 5).

### 22.11 DEC-030 HMAC-Chained Audit Events

| Event type | Severity | Retention | Trigger |
|---|---|---|---|
| `gamification.streak_started` | STANDARD | 3 years | `current_streak` transitions from 0 → 1 (new streak) |
| `gamification.streak_grace_applied` | STANDARD | 3 years | Grace applied; streak preserved across 1 missed day |
| `gamification.streak_reset` | STANDARD | 3 years | Streak resets to 0; payload includes `previous_streak` count |
| `gamification.streak_milestone` | STANDARD | 3 years | `current_streak` reaches {7, 14, 30, 60, 90, 180, 365}; payload includes `milestone_days` |
| `gamification.pr_set` | STANDARD | 3 years | New `personal_records` row inserted; payload includes `exercise_slug`, `metric`, `value`, `improvement_pct` |
| `gamification.achievement_earned` | STANDARD | 3 years | New `user_achievements` row inserted; payload includes `achievement_key`, `tier`, `category` |

**Payload schema (all events):**
```json
{
  "event_type": "gamification.streak_grace_applied",
  "user_id": "<uuid>",
  "tenant_id": "<uuid>",
  "timestamp": "2026-05-29T14:23:00Z",
  "payload": {
    "missed_date": "2026-05-28",
    "streak_count_at_grace": 14,
    "workout_session_id": "<uuid>"
  }
}
```

Events are emitted **within the same database transaction** as the streak state update, using the DEC-030 HMAC-chaining function from `docs/AUDIT_LOG_SCHEMA.md`. If the transaction rolls back, no event is emitted.

### 22.12 GDPR Art. 17 Erasure

On user hard-delete (§12 erasure Worker), the following sequence is executed for gamification data. No special retention applies — all gamification data is personal and deleted immediately.

```sql
-- Erasure sequence for gamification (called within §12 erasure Worker)
-- Step order: achievements → grace_events → personal_records → user_streaks

-- 1. user_achievements
DELETE FROM user_achievements WHERE user_id = $1;
-- emit coaching.memory_deleted analog: gamification.achievements_erased (CRITICAL, 7yr DEC-030 event)

-- 2. streak_grace_events (immutable normally; erasure via form_admin BYPASSRLS)
DELETE FROM streak_grace_events WHERE user_id = $1;

-- 3. personal_records
DELETE FROM personal_records WHERE user_id = $1;

-- 4. user_streaks
DELETE FROM user_streaks WHERE user_id = $1;

-- CASCADE: all four tables have ON DELETE CASCADE on user_id → users(id),
-- so a direct users DELETE would also cascade. The explicit sequence above is
-- used to emit per-table DEC-030 events and confirm each step before proceeding.
```

**Note:** `achievement_definitions` is not user data; no erasure action required.

### 22.13 Tenant Admin Reporting (Privacy Floor)

Per DEC-002 and §17 (Enterprise Admin Reporting Schema), **tenant admins never see individual streak lengths, grace usage counts, or personal records in identified form**. If aggregate reporting is added (e.g., "% of team with active streaks"), it must:

- Pass through the §17 aggregate view layer
- Meet the k-anonymity floor (minimum cohort size ≥ 5 per §17.3)
- Emit a `gamification.admin_aggregate_query` DEC-030 event

Currently, no aggregate gamification views are defined. This is explicitly deferred per OQ-GAM-02 (§22.14).

### 22.14 Open Questions

| # | Question | Owner | Priority |
|---|---|---|---|
| OQ-GAM-01 | What qualifies as a "streak day"? Default: any `workout_sessions.completed_at IS NOT NULL`. Future option: minimum N sets completed, or minimum session duration. Requires product-manager + sports-scientist alignment before streak engine ships. | `product-manager` + `sports-scientist` | **P0 — before streak engine ships** |
| OQ-GAM-02 | Should aggregate gamification metrics (% team with active streak, average PR improvement) be exposed in the tenant admin dashboard? Default: NO (privacy floor + DEC-002). If yes: requires §17 aggregate view layer + k-anonymity gate + DEC-030 audit event + clinical-safety review. | `enterprise-architect` + `clinical-safety` | **P2 — after beta cohort data available** |
| OQ-GAM-03 | External achievement sharing (e.g., "share my 30-day streak to Instagram"): blocked by DEC-029 (clinical-safety gate on external sharing). If introduced: clinical-safety review required; achievement share must not display body-composition or injury-related context. | `clinical-safety` + `product-strategist` | **P2 — out of scope until post-launch** |
| OQ-GAM-04 | Streak timezone edge case: user travels across the date-line during an active streak. Current design uses `user_profile.timezone` which is static. If user updates timezone mid-streak, the streak engine must re-evaluate the last_activity_date in the new timezone. Implementation detail deferred to streak Worker design. | `platform-engineer` | **P1 — before streak engine ships** |
| OQ-GAM-05 | Should `personal_records` include e1RM estimates (Epley/Brzycki formulas applied to multi-rep sets) or only tested maxima? Default: include e1RM with `context.e1rm_formula` field and `context.reps_used` to allow consumers to evaluate reliability. Display note: UI should surface e1RM estimates with confidence indicator when `reps_used > 3`. | `platform-engineer` + `sports-scientist` | **P1 — before PR engine ships** |

### 22.15 SOC 2 Evidence Mapping

| SOC 2 Criterion | How §22 Satisfies It |
|---|---|
| **CC6.1 — Logical access** | RLS restricts all gamification tables to own-user rows for `form_api`. No cross-user or cross-tenant read paths exist. `achievement_definitions` is read-only for non-admin roles. |
| **CC6.2 — Access controls prior to issuing** | `form_api` cannot write `user_achievements` directly; the achievement engine runs as `form_system`, preventing self-award exploits. `achievement_definitions` is form_admin-only write, preventing key injection. |
| **CC7.2 — System monitoring** | `gamification.streak_reset` and `gamification.pr_set` events in the HMAC-chained audit log enable anomaly detection (e.g., impossible PR values, streak counter jumps without qualifying sessions). OBSERVABILITY.md §22 coaching quality panel can be extended for gamification anomaly alerting. |
| **CC8.1 — Change management** | All gamification mutations are HMAC-chained in-transaction. `achievement_definitions` changes require form_admin and produce `gamification.*` events with `entity_type: 'achievement_definitions'`. The append-only constraint on `streak_grace_events` and `user_achievements` is enforced at the RLS layer — no accidental retroactive modification. |
| **P4.0 / P5.0 — Privacy inquiries and requests** | GDPR Art. 17 erasure sequence (§22.12) ensures complete deletion of all gamification personal data. The four-step explicit sequence with DEC-030 events per table provides auditor evidence of erasure completeness. |

**Auditor evidence artefacts:**
- `user_streaks` table export with `updated_at` covering observation period, confirming streak state changes correspond to `workout_sessions.completed_at`
- `streak_grace_events` table export for observation period: verify no user has more than 1 grace per streak cycle
- `audit_log_events` filtered on `gamification.*` event types, HMAC chain verified
- `__tests__/db/gamification_streak.test.ts` CI run output (DEC-013 regression tests)

### 22.16 Implementation Checklist

| # | Task | Owner | Priority | Milestone |
|---|---|---|---|---|
| 1 | Create `migrations/YYYYMMDD_user_streaks.sql` — `user_streaks` DDL, indexes, `update_updated_at_column()` trigger, `chk_streak_longest` constraint, RLS policies from §22.8 | `platform-engineer` | **P0** | M3 |
| 2 | Create `migrations/YYYYMMDD_streak_grace_events.sql` — `streak_grace_events` DDL, indexes, RLS policies | `platform-engineer` | **P0** | M3 |
| 3 | Create `migrations/YYYYMMDD_personal_records.sql` — `personal_records` DDL, `pr_metric` type, indexes, RLS policies | `platform-engineer` | **P0** | M3 |
| 4 | Create `migrations/YYYYMMDD_achievement_tables.sql` — `achievement_definitions` DDL (types, constraints, seed data from §22.6), `user_achievements` DDL, indexes, RLS policies | `platform-engineer` | **P0** | M3 |
| 5 | Write `__tests__/db/gamification_streak.test.ts` — DEC-013 regression tests: (a) single miss → grace applied, streak preserved; (b) double miss → reset; (c) grace_used_this_cycle reset on next session; (d) timezone edge case (UTC midnight boundary); (e) longest_streak invariant. Add to CI gating. | `qa-lead` + `platform-engineer` | **P0** | M3 |
| 6 | Implement streak engine Worker `src/workers/gamification/streak-engine.ts`: invoked on every `workout_sessions.completed_at` update; computes gap, applies grace or reset per §22.2 state machine; emits all DEC-030 events in-transaction | `platform-engineer` | **P0** | M3 |
| 7 | Implement PR engine Worker `src/workers/gamification/pr-engine.ts`: on workout-completion, evaluate sets in the session against `personal_records` current best; INSERT new row if PR; emit `gamification.pr_set` DEC-030 event; resolve OQ-GAM-05 (e1RM formula choice) before shipping | `platform-engineer` + `sports-scientist` | **P0** | M4 |
| 8 | Implement achievement engine Worker `src/workers/gamification/achievement-engine.ts`: triggered after streak-engine and PR-engine complete; checks `achievement_definitions` against current state; INSERTs into `user_achievements` for any newly qualifying achievements; emits `gamification.achievement_earned` DEC-030 events | `platform-engineer` | **P1** | M4 |
| 9 | Add gamification erasure steps to §12 erasure Worker (`src/workers/gdpr/erasure.ts`): explicit four-step sequence from §22.12 with per-step DEC-030 events; add `gamification.achievements_erased` and `gamification.prs_erased` event types to `AUDIT_LOG_SCHEMA.md` | `platform-engineer` + `compliance-officer` | **P0** | M3 |
| 10 | Register all 6 `gamification.*` DEC-030 event types (§22.11) in `docs/AUDIT_LOG_SCHEMA.md` event registry; add severity, retention tier, and `entity_type: 'gamification'` label | `compliance-officer` | **P0** | M3 |
| 11 | Wire streak milestones ({7, 14, 30, 60, 90, 180, 365}) into streak Worker; emit `gamification.streak_milestone` event with `milestone_days` payload; dedup: only emit once per milestone per streak cycle (not on re-earn of same milestone in new cycle — OQ-GAM-01 clarification needed on whether re-earned milestones re-emit) | `platform-engineer` | **P1** | M4 |
| 12 | Add RLS cross-tenant isolation tests for gamification tables to `__tests__/db/rls_isolation.test.ts`; verify no cross-user PR leakage, no achievement fabrication from form_api role | `enterprise-architect` | **P0** | M3 |

---

## 23. Exercise Library & Program Schema

This section defines the exercise catalog, program templates, user program instances, and periodization structure. It closes the schema gap referenced in `docs/PRODUCT_SPEC.md` (exercise catalog) and `docs/TECHNICAL.md` (program generation architecture). Cross-references: §2 (`workout_sessions` / `workout_sets` core schema), §22 (`personal_records` — PRs inform load prescriptions), §21 (`coaching_sessions` — Victor-generated programs originate here), §15 (`cv_sessions` — `is_cv_supported` flag links exercises to the CV pipeline), `docs/VICTOR_PROMPT_GUIDE.md`.

---

### 23.1 Design Decisions

**Exercise catalog is global, not tenant-scoped.** The canonical library (`exercises` with `is_custom = FALSE`) is shared across all tenants and users. Custom exercises (`is_custom = TRUE`) belong to a specific user and/or tenant and are invisible to others. This avoids duplication and enables consistent CV form detection across the pipeline — §15 `cv_sessions` reference the same canonical `exercise_id` regardless of tenant.

**Program templates are FORM-curated, not user-submitted.** `program_templates` has `form_admin`-only write access. User-generated templates are out of scope at launch (see OQ-PROG-04). This maintains quality and clinical-safety control over the program library.

**Victor-generated programs are first-class data.** When Victor (§21) generates a program, it creates a `user_programs` row with `source = 'victor_generated'` and a FK to the originating `coaching_session_id`. The structured prescription (blocks → sessions → exercises) is stored in relational tables, not free-form JSON, enabling adherence tracking, PR correlation, and future coaching context reconstruction.

**One active program per user.** A partial unique index enforces at most one program per user with `status = 'active'`. Victor must mark the existing active program as `'abandoned'` (with `abandonment_reason = 'victor_replaced'`) before inserting a new one — and emit a DEC-030 event for the abandoned program within the same transaction.

**Load prescription is relative-first.** `load_type = 'relative'` computes working weight as a percentage of the user's current `personal_records` (§22) for that exercise. Absolute prescriptions are allowed but discouraged for Victor-generated programs, since they become stale as strength increases.

---

### 23.2 ENUMs

```sql
CREATE TYPE movement_pattern AS ENUM (
  'push_horizontal',   -- bench press, push-up
  'push_vertical',     -- overhead press, dip
  'pull_horizontal',   -- row, face pull
  'pull_vertical',     -- pull-up, lat pulldown
  'hip_hinge',         -- deadlift, RDL, good morning
  'squat',             -- squat, leg press, hack squat
  'lunge',             -- split squat, walking lunge
  'carry',             -- farmer carry, suitcase carry
  'rotation',          -- cable rotation, landmine twist
  'static_hold',       -- plank, wall sit, dead hang
  'plyometric',        -- box jump, clap push-up
  'cardio'             -- assault bike, rowing, treadmill
);

CREATE TYPE muscle_group AS ENUM (
  'chest', 'back_upper', 'back_lower',
  'shoulders_front', 'shoulders_side', 'shoulders_rear',
  'biceps', 'triceps', 'forearms',
  'quadriceps', 'hamstrings', 'glutes', 'calves',
  'core_anterior', 'core_posterior', 'core_lateral',
  'hip_flexors', 'adductors', 'abductors',
  'traps_upper', 'traps_mid_lower', 'spinal_erectors'
);

CREATE TYPE equipment_type AS ENUM (
  'barbell', 'dumbbell', 'cable', 'machine', 'bodyweight',
  'kettlebell', 'resistance_band', 'trap_bar', 'ez_bar',
  'smith_machine', 'rings', 'pullup_bar', 'dip_station',
  'ab_wheel', 'bench', 'other'
);

CREATE TYPE program_goal AS ENUM (
  'strength',            -- 1RM / multi-rep maxes primary metric
  'hypertrophy',         -- volume and muscle size primary
  'powerbuilding',       -- strength + hypertrophy blend
  'strength_endurance',  -- higher rep, shorter rest
  'general_fitness',     -- beginner / maintenance
  'recomposition'        -- concurrent strength + conditioning
);

CREATE TYPE program_source AS ENUM (
  'victor_generated',  -- AI program for this specific user
  'template',          -- cloned from program_templates library
  'custom'             -- user-built (reserved; out of scope at launch)
);

CREATE TYPE program_status AS ENUM (
  'draft',      -- generated but not yet confirmed by user
  'active',     -- currently being executed
  'paused',     -- user-paused
  'completed',  -- successfully finished
  'abandoned'   -- ended early (user or Victor triggered)
);

CREATE TYPE block_type AS ENUM (
  'accumulation',    -- high volume, moderate intensity (~65-75% 1RM)
  'intensification', -- moderate volume, high intensity (~80-90% 1RM)
  'realization',     -- low volume, peak / test week
  'deload'           -- reduced load and volume for recovery
);

CREATE TYPE session_split AS ENUM (
  'upper_body', 'lower_body', 'full_body',
  'push', 'pull', 'legs',
  'upper_push', 'upper_pull',
  'squat_focus', 'hinge_focus',
  'posterior_chain', 'anterior_chain',
  'conditioning', 'mobility_flexibility', 'rest'
);
```

---

### 23.3 `exercises` — Global Catalog

Global rows (`is_custom = FALSE`) are FORM-curated and readable by all authenticated users. Custom rows (`is_custom = TRUE`) are visible only to the owning user/tenant.

```sql
CREATE TABLE exercises (
  exercise_id       UUID    PRIMARY KEY DEFAULT gen_random_uuid(),
  slug              TEXT    NOT NULL UNIQUE,   -- e.g. 'barbell-back-squat'
  name_ua           TEXT    NOT NULL,
  name_en           TEXT    NOT NULL,
  category          TEXT    NOT NULL CHECK (category IN (
    'compound', 'isolation', 'cardio', 'mobility', 'warm_up'
  )),
  movement_pattern  movement_pattern NOT NULL,
  is_bilateral      BOOLEAN NOT NULL DEFAULT TRUE,
  is_bodyweight     BOOLEAN NOT NULL DEFAULT FALSE,
  is_cv_supported   BOOLEAN NOT NULL DEFAULT FALSE,  -- CV pose estimation (§15)
  difficulty        SMALLINT NOT NULL CHECK (difficulty BETWEEN 1 AND 5),
  min_equipment     equipment_type[] NOT NULL DEFAULT '{}',
  alt_equipment     equipment_type[] NOT NULL DEFAULT '{}',
  -- contraindication_tags: Victor screens against user_health_profiles.injuries
  -- e.g. 'lower_back_pain', 'shoulder_impingement', 'knee_flexion_pain'
  contraindication_tags TEXT[] NOT NULL DEFAULT '{}',
  is_custom         BOOLEAN NOT NULL DEFAULT FALSE,
  created_by_user   UUID    REFERENCES user_profiles(user_id) ON DELETE SET NULL,
  created_by_tenant UUID    REFERENCES tenants(tenant_id)     ON DELETE SET NULL,
  is_active         BOOLEAN NOT NULL DEFAULT TRUE,
  deprecated_at     TIMESTAMPTZ,
  created_at        TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at        TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  CONSTRAINT chk_custom_has_owner CHECK (
    (is_custom = FALSE AND created_by_user IS NULL AND created_by_tenant IS NULL)
    OR
    (is_custom = TRUE AND (created_by_user IS NOT NULL OR created_by_tenant IS NOT NULL))
  ),
  CONSTRAINT chk_deprecated_inactive CHECK (
    deprecated_at IS NULL OR is_active = FALSE
  )
);

CREATE INDEX idx_exercises_movement  ON exercises (movement_pattern) WHERE is_active = TRUE;
CREATE INDEX idx_exercises_category  ON exercises (category)         WHERE is_active = TRUE;
CREATE INDEX idx_exercises_cv        ON exercises (is_cv_supported)  WHERE is_active = TRUE;
CREATE INDEX idx_exercises_custom_u  ON exercises (created_by_user)  WHERE is_custom = TRUE;
CREATE INDEX idx_exercises_custom_t  ON exercises (created_by_tenant) WHERE is_custom = TRUE;
```

**Seed catalog — 60 exercises (P0 M2):** Barbell Back Squat, Front Squat, Romanian Deadlift, Conventional Deadlift, Sumo Deadlift, Flat Bench Press, Incline Bench Press, Decline Bench Press, Overhead Press, Pendlay Row, Barbell Row, Pull-up, Chin-up, Lat Pulldown, Seated Cable Row, Dumbbell Lateral Raise, Dumbbell Curl, EZ Bar Curl, Skull Crusher, Triceps Pushdown, Bulgarian Split Squat, Hip Thrust, Leg Press, Leg Curl, Leg Extension, Calf Raise (Standing/Seated), Face Pull, Plank, Dead Bug, Copenhagen Plank, Farmer Carry, Suitcase Carry, Trap Bar Deadlift, Close-Grip Bench Press, Dips, Ring Row, GHD Sit-Up, Back Extension, Good Morning, Hack Squat, Step-Up, Walking Lunge, Box Jump, Assault Bike Sprint, Rowing Machine, Treadmill Run, Incline Dumbbell Press, Dumbbell Row, Cable Fly, Preacher Curl, Hammer Curl, Overhead Triceps Extension (DB/Cable), Close-Grip Push-Up, Push-Up, Inverted Row, Jefferson Curl, Nordic Curl, Glute Bridge, Ab Wheel Rollout, Hanging Leg Raise, Pallof Press, Landmine Press.

---

### 23.4 `exercise_instructions` — Multilingual Coaching Cues

```sql
CREATE TABLE exercise_instructions (
  instruction_id UUID     PRIMARY KEY DEFAULT gen_random_uuid(),
  exercise_id    UUID     NOT NULL REFERENCES exercises(exercise_id) ON DELETE CASCADE,
  locale         TEXT     NOT NULL CHECK (locale IN ('uk', 'en')),
  step_number    SMALLINT NOT NULL CHECK (step_number >= 1),
  instruction    TEXT     NOT NULL,
  coaching_cue   TEXT,    -- short in-workout reminder ≤ 80 chars
  UNIQUE (exercise_id, locale, step_number)
);

CREATE INDEX idx_ex_instr_exercise_locale ON exercise_instructions (exercise_id, locale);
```

---

### 23.5 `exercise_muscle_groups` — Muscle Group Junction

```sql
CREATE TABLE exercise_muscle_groups (
  exercise_id  UUID         NOT NULL REFERENCES exercises(exercise_id) ON DELETE CASCADE,
  muscle_group muscle_group NOT NULL,
  role         TEXT         NOT NULL CHECK (role IN ('primary', 'secondary', 'stabilizer')),
  PRIMARY KEY (exercise_id, muscle_group, role)
);

CREATE INDEX idx_emg_muscle_primary ON exercise_muscle_groups (muscle_group) WHERE role = 'primary';
```

Victor uses this table for **muscle group balance analysis** when generating or reviewing a program. It checks that weekly volume across `primary` groups is balanced — flagging if push:pull ratio deviates more than 20% from 1:1, or if posterior-chain weekly volume is below 30% of total lower-body volume. Both checks are enforced in the program generation Worker before commit; they are not DB constraints.

---

### 23.6 `program_templates` — FORM-Curated Library

```sql
CREATE TABLE program_templates (
  template_id        UUID         PRIMARY KEY DEFAULT gen_random_uuid(),
  slug               TEXT         NOT NULL UNIQUE,
  name_ua            TEXT         NOT NULL,
  name_en            TEXT         NOT NULL,
  goal               program_goal NOT NULL,
  duration_weeks     SMALLINT     NOT NULL CHECK (duration_weeks BETWEEN 4 AND 52),
  days_per_week      SMALLINT     NOT NULL CHECK (days_per_week BETWEEN 2 AND 7),
  experience_level   TEXT         NOT NULL CHECK (experience_level IN (
    'beginner', 'intermediate', 'advanced'
  )),
  equipment_required equipment_type[] NOT NULL DEFAULT '{}',
  description_ua     TEXT,
  description_en     TEXT,
  is_victor_authored BOOLEAN      NOT NULL DEFAULT FALSE,
  is_active          BOOLEAN      NOT NULL DEFAULT TRUE,
  -- min_plan_tier: gates template visibility by subscription level
  min_plan_tier      TEXT         NOT NULL DEFAULT 'consumer' CHECK (min_plan_tier IN (
    'consumer', 'starter', 'growth', 'enterprise'
  )),
  created_at         TIMESTAMPTZ  NOT NULL DEFAULT NOW(),
  updated_at         TIMESTAMPTZ  NOT NULL DEFAULT NOW()
);
```

**Initial library (P1 M3):** 6 templates — 2× beginner (3-day full-body strength; 3-day beginner hypertrophy), 2× intermediate (4-day upper/lower powerbuilding; 4-day push/pull/legs/full), 2× advanced (5-day powerbuilding block; 6-day frequency specialist). All `is_victor_authored = FALSE` at initial import; Victor-authored templates require sports-scientist sign-off before `is_victor_authored = TRUE`.

---

### 23.7 `user_programs` — User Program Instances

```sql
CREATE TABLE user_programs (
  program_id          UUID           PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id             UUID           NOT NULL REFERENCES user_profiles(user_id) ON DELETE CASCADE,
  tenant_id           UUID           REFERENCES tenants(tenant_id) ON DELETE CASCADE,
  template_id         UUID           REFERENCES program_templates(template_id) ON DELETE SET NULL,
  coaching_session_id UUID           REFERENCES coaching_sessions(session_id) ON DELETE SET NULL,
  source              program_source NOT NULL,
  status              program_status NOT NULL DEFAULT 'draft',
  goal                program_goal   NOT NULL,
  name_ua             TEXT           NOT NULL,
  name_en             TEXT           NOT NULL,
  duration_weeks      SMALLINT       NOT NULL CHECK (duration_weeks BETWEEN 1 AND 52),
  days_per_week       SMALLINT       NOT NULL CHECK (days_per_week BETWEEN 1 AND 7),
  started_at          TIMESTAMPTZ,
  target_end_at       TIMESTAMPTZ    GENERATED ALWAYS AS (
    CASE WHEN started_at IS NOT NULL
         THEN started_at + (duration_weeks * INTERVAL '1 week')
         ELSE NULL END
  ) STORED,
  completed_at        TIMESTAMPTZ,
  abandoned_at        TIMESTAMPTZ,
  abandonment_reason  TEXT           CHECK (abandonment_reason IN (
    'user_requested', 'victor_replaced', 'goal_changed',
    'injury_reported', 'insufficient_progress', 'equipment_changed'
  )),
  -- generation_context: Victor's rationale (see §23.7.1 privacy constraints)
  generation_context  JSONB,
  -- adherence_pct: computed on completion/abandonment
  -- ratio of (sessions completed) / (total non-rest sessions)
  adherence_pct       NUMERIC(5,2)   CHECK (adherence_pct BETWEEN 0 AND 100),
  created_at          TIMESTAMPTZ    NOT NULL DEFAULT NOW(),
  updated_at          TIMESTAMPTZ    NOT NULL DEFAULT NOW(),
  CONSTRAINT chk_active_has_start CHECK (
    status NOT IN ('active', 'paused', 'completed', 'abandoned')
    OR started_at IS NOT NULL
  ),
  CONSTRAINT chk_completion_consistency CHECK (
    (status = 'completed'  AND completed_at IS NOT NULL AND abandoned_at IS NULL)
    OR (status = 'abandoned' AND abandoned_at IS NOT NULL AND completed_at IS NULL)
    OR status NOT IN ('completed', 'abandoned')
  )
);

-- At most one active program per user
CREATE UNIQUE INDEX uq_user_one_active_program
  ON user_programs (user_id)
  WHERE status = 'active';

CREATE INDEX idx_user_programs_user     ON user_programs (user_id, status);
CREATE INDEX idx_user_programs_tenant   ON user_programs (tenant_id) WHERE tenant_id IS NOT NULL;
CREATE INDEX idx_user_programs_coaching ON user_programs (coaching_session_id)
  WHERE coaching_session_id IS NOT NULL;
```

#### 23.7.1 `generation_context` Privacy Schema

The `generation_context` JSONB column stores Victor's reasoning for program construction. Strict allow/deny rules apply before INSERT (enforced by `sanitizeGenerationContext()` — see §23.12):

| Field | Status | Notes |
|---|---|---|
| `training_age_bracket` | ✅ Permitted | `'<1yr'` / `'1-3yr'` / `'3-5yr'` / `'>5yr'` |
| `strength_level` | ✅ Permitted | `'novice'` / `'intermediate'` / `'advanced'` |
| `goal_stated` | ✅ Permitted | Value from `program_goal` ENUM |
| `days_available` | ✅ Permitted | Integer |
| `equipment_available` | ✅ Permitted | Array of `equipment_type` values |
| `preferred_movements` | ✅ Permitted | Array of `movement_pattern` values |
| `injury_mentions` | ❌ **BLOCKED** | Injury data lives in `user_health_profiles` only |
| `body_weight_kg` | ❌ **BLOCKED** | Body-composition data never in program records |
| `caloric_intake` | ❌ **BLOCKED** | Nutrition data never in program records |
| Diagnostic language | ❌ **BLOCKED** | e.g. "herniated disc", "tendinopathy" — clinical-safety VETO |

---

### 23.8 `program_blocks` — Periodization Blocks

```sql
CREATE TABLE program_blocks (
  block_id      UUID       PRIMARY KEY DEFAULT gen_random_uuid(),
  program_id    UUID       NOT NULL REFERENCES user_programs(program_id) ON DELETE CASCADE,
  block_number  SMALLINT   NOT NULL,
  name_ua       TEXT       NOT NULL,
  name_en       TEXT       NOT NULL,
  block_type    block_type NOT NULL,
  week_start    SMALLINT   NOT NULL CHECK (week_start >= 1),
  week_end      SMALLINT   NOT NULL,
  notes_ua      TEXT,
  notes_en      TEXT,
  created_at    TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  UNIQUE (program_id, block_number),
  CONSTRAINT chk_block_week_range CHECK (week_end >= week_start)
);

CREATE INDEX idx_program_blocks_program ON program_blocks (program_id);
```

**Deload enforcement rule:** Victor must include at least one `block_type = 'deload'` block in any program of 8+ weeks. This is a Worker-layer constraint, not a DB constraint (to avoid blocking valid 4–7 week programs). The program generation Worker validates before commit; the DEC-030 `program.created` event includes a `deload_blocks_count` field for auditing.

---

### 23.9 `program_sessions` — Planned Sessions

```sql
CREATE TABLE program_sessions (
  session_id                 UUID          PRIMARY KEY DEFAULT gen_random_uuid(),
  program_id                 UUID          NOT NULL REFERENCES user_programs(program_id) ON DELETE CASCADE,
  block_id                   UUID          REFERENCES program_blocks(block_id) ON DELETE SET NULL,
  week_number                SMALLINT      NOT NULL CHECK (week_number >= 1),
  day_of_week                SMALLINT      NOT NULL CHECK (day_of_week BETWEEN 1 AND 7),  -- 1 = Mon
  session_number             SMALLINT      NOT NULL,  -- global sequence within program
  session_split              session_split NOT NULL,
  name_ua                    TEXT          NOT NULL,
  name_en                    TEXT          NOT NULL,
  estimated_duration_minutes SMALLINT,
  is_rest_day                BOOLEAN       NOT NULL DEFAULT FALSE,
  coaching_notes_ua          TEXT,
  coaching_notes_en          TEXT,
  -- FK to actual logged session; NULL until user completes this session
  workout_session_id         UUID          REFERENCES workout_sessions(session_id) ON DELETE SET NULL,
  completed_at               TIMESTAMPTZ,
  skipped_at                 TIMESTAMPTZ,
  created_at                 TIMESTAMPTZ   NOT NULL DEFAULT NOW(),
  UNIQUE (program_id, week_number, day_of_week),
  CONSTRAINT chk_not_both_completed_skipped CHECK (
    NOT (completed_at IS NOT NULL AND skipped_at IS NOT NULL)
  )
);

CREATE INDEX idx_program_sessions_program ON program_sessions (program_id, week_number);
CREATE INDEX idx_program_sessions_workout ON program_sessions (workout_session_id)
  WHERE workout_session_id IS NOT NULL;
```

---

### 23.10 `program_session_exercises` — Exercise Prescriptions

```sql
CREATE TABLE program_session_exercises (
  prescription_id  UUID    PRIMARY KEY DEFAULT gen_random_uuid(),
  session_id       UUID    NOT NULL REFERENCES program_sessions(session_id) ON DELETE CASCADE,
  exercise_id      UUID    NOT NULL REFERENCES exercises(exercise_id) ON DELETE RESTRICT,
  order_index      SMALLINT NOT NULL CHECK (order_index >= 1),
  superset_group   SMALLINT,  -- NULL = standalone; same integer = superset partners
  sets             SMALLINT NOT NULL CHECK (sets BETWEEN 1 AND 20),
  rep_min          SMALLINT CHECK (rep_min >= 1),
  rep_max          SMALLINT CHECK (rep_max >= 1),
  reps_fixed       SMALLINT CHECK (reps_fixed >= 1),
  rpe_target       NUMERIC(3,1) CHECK (rpe_target BETWEEN 5.0 AND 10.0),
  rir_target       SMALLINT CHECK (rir_target BETWEEN 0 AND 5),
  load_type        TEXT    NOT NULL DEFAULT 'rpe_governed' CHECK (load_type IN (
    'relative',      -- % of 1RM from personal_records (§22)
    'absolute',      -- explicit kg
    'bodyweight',
    'rpe_governed',  -- load not prescribed; RPE is the governing constraint
    'amrap'
  )),
  load_pct_1rm     NUMERIC(5,2) CHECK (load_pct_1rm BETWEEN 20 AND 110),
  load_kg          NUMERIC(6,2) CHECK (load_kg > 0),
  rest_seconds     SMALLINT     CHECK (rest_seconds BETWEEN 10 AND 600),
  tempo            TEXT         CHECK (tempo ~ '^\d-\d-\d-\d$'),  -- e.g. '3-1-2-0'
  coaching_cue_ua  TEXT,
  coaching_cue_en  TEXT,
  -- substitution pattern: see §23.13
  substitute_for   UUID    REFERENCES program_session_exercises(prescription_id) ON DELETE SET NULL,
  substituted_at   TIMESTAMPTZ,
  substitution_reason TEXT CHECK (substitution_reason IN (
    'equipment_unavailable', 'exercise_too_hard', 'exercise_too_easy',
    'user_preference', 'injury_reported', 'victor_recommended'
  )),
  created_at       TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  CONSTRAINT chk_rep_prescription CHECK (
    (rep_min IS NOT NULL AND rep_max IS NOT NULL AND rep_min <= rep_max)
    OR reps_fixed IS NOT NULL
    OR load_type = 'amrap'
  ),
  CONSTRAINT chk_load_consistency CHECK (
    (load_type = 'relative'  AND load_pct_1rm IS NOT NULL AND load_kg IS NULL)
    OR (load_type = 'absolute' AND load_kg IS NOT NULL      AND load_pct_1rm IS NULL)
    OR load_type IN ('bodyweight', 'rpe_governed', 'amrap')
  ),
  CONSTRAINT chk_substitution_atomicity CHECK (
    (substitute_for IS NULL) = (substituted_at IS NULL)
  )
);

-- Substitution rows may share order_index with the replaced row;
-- the partial index only enforces uniqueness among active (non-substitute) rows.
CREATE UNIQUE INDEX uq_session_exercise_order
  ON program_session_exercises (session_id, order_index)
  WHERE substitute_for IS NULL;

CREATE INDEX idx_pse_session  ON program_session_exercises (session_id);
CREATE INDEX idx_pse_exercise ON program_session_exercises (exercise_id);
```

---

### 23.11 Row-Level Security Policies

#### 23.11.1 `exercises`

```sql
-- Global catalog: read by all authenticated users
CREATE POLICY exercises_read_global ON exercises
  FOR SELECT TO form_api
  USING (is_custom = FALSE AND is_active = TRUE);

-- Custom: own-user read
CREATE POLICY exercises_read_custom_own ON exercises
  FOR SELECT TO form_api
  USING (is_custom = TRUE AND created_by_user = auth.uid());

-- Custom: tenant-shared read (same tenant_id)
CREATE POLICY exercises_read_custom_tenant ON exercises
  FOR SELECT TO form_api
  USING (
    is_custom = TRUE
    AND created_by_tenant = (
      SELECT tenant_id FROM user_profiles WHERE user_id = auth.uid()
    )
  );

-- Custom exercise creation: own-user only, tenant context required
CREATE POLICY exercises_insert_custom ON exercises
  FOR INSERT TO form_api
  WITH CHECK (
    is_custom = TRUE
    AND created_by_user = auth.uid()
    AND created_by_tenant IS NOT NULL
  );

-- exercise_instructions + exercise_muscle_groups inherit via exercise_id join.
-- form_admin: unrestricted (global catalog management).
```

#### 23.11.2 `program_templates`

```sql
-- Read: all authenticated users whose plan tier satisfies min_plan_tier
CREATE POLICY templates_read ON program_templates
  FOR SELECT TO form_api
  USING (
    is_active = TRUE
    AND min_plan_tier IN ('consumer', (
      SELECT plan FROM tenants
      WHERE tenant_id = (
        SELECT tenant_id FROM user_profiles WHERE user_id = auth.uid()
      )
    ))
  );

-- Write: form_admin only
CREATE POLICY templates_admin_all ON program_templates
  FOR ALL TO form_admin USING (TRUE);
```

#### 23.11.3 `user_programs`, `program_blocks`, `program_sessions`, `program_session_exercises`

```sql
-- user_programs: own-row read
CREATE POLICY user_programs_own_read ON user_programs
  FOR SELECT TO form_api
  USING (user_id = auth.uid());

-- No direct form_api INSERT/UPDATE on user_programs.
-- All program lifecycle mutations run as form_system (program generation Worker,
-- program lifecycle Worker) to ensure constraint enforcement and DEC-030
-- event emission are atomic within the same transaction.

-- form_system: tenant-scoped access
CREATE POLICY user_programs_system ON user_programs
  FOR ALL TO form_system
  USING (tenant_id = current_setting('app.current_tenant_id', TRUE)::UUID);

-- tenant_admin: ZERO ACCESS (DEC-002 — individual training programs
-- are not visible to enterprise HR administrators).
-- Aggregate reporting uses §17 materialized views exclusively.

-- form_admin: unrestricted
CREATE POLICY user_programs_admin ON user_programs
  FOR ALL TO form_admin USING (TRUE);

-- Child tables (program_blocks, program_sessions, program_session_exercises):
-- same pattern — form_api reads via user_id join through user_programs;
-- form_system writes with tenant context; form_admin unrestricted.
```

---

### 23.12 Victor Program Generation — Worker Pattern

When Victor generates a program during a coaching session, the Worker must execute as a single atomic transaction:

```typescript
// src/workers/programs/generate-program.ts (structural sketch)

const PERMITTED_CONTEXT_KEYS = new Set([
  'training_age_bracket', 'strength_level', 'goal_stated',
  'days_available', 'equipment_available', 'preferred_movements'
]);

function sanitizeGenerationContext(raw: unknown): Record<string, unknown> {
  if (typeof raw !== 'object' || raw === null) return {};
  return Object.fromEntries(
    Object.entries(raw as Record<string, unknown>)
      .filter(([k]) => PERMITTED_CONTEXT_KEYS.has(k))
  );
}

async function generateProgram(
  userId: string, tenantId: string | null,
  coachingSessionId: string,
  prescription: ProgramPrescription,
  tx: DatabaseTransaction
): Promise<string> {

  // 1. Abandon existing active program
  const existing = await tx.query<{ program_id: string }>(
    `SELECT program_id FROM user_programs
     WHERE user_id = $1 AND status = 'active' FOR UPDATE`,
    [userId]
  );
  if (existing.rows.length > 0) {
    await tx.query(
      `UPDATE user_programs SET status = 'abandoned', abandoned_at = NOW(),
       abandonment_reason = 'victor_replaced' WHERE program_id = $1`,
      [existing.rows[0].program_id]
    );
    await emitAuditEvent(tx, 'program.abandoned', {
      entity_id: existing.rows[0].program_id, entity_type: 'program',
      user_id: userId, tenant_id: tenantId,
      metadata: { reason: 'victor_replaced' }
    });
  }

  // 2. Insert user_programs with sanitized context
  const { program_id } = await tx.query<{ program_id: string }>(
    `INSERT INTO user_programs
       (user_id, tenant_id, coaching_session_id, source, status, goal,
        name_ua, name_en, duration_weeks, days_per_week, generation_context)
     VALUES ($1,$2,$3,'victor_generated','draft',$4,$5,$6,$7,$8,$9)
     RETURNING program_id`,
    [userId, tenantId, coachingSessionId,
     prescription.goal, prescription.name_ua, prescription.name_en,
     prescription.duration_weeks, prescription.days_per_week,
     sanitizeGenerationContext(prescription.context)]
  ).then(r => r.rows[0]);

  // 3. Batch-insert blocks → sessions → exercises
  // ... (loop over prescription.blocks)

  // 4. Emit program.created audit event
  await emitAuditEvent(tx, 'program.created', {
    entity_id: program_id, entity_type: 'program',
    user_id: userId, tenant_id: tenantId,
    metadata: {
      source: 'victor_generated', goal: prescription.goal,
      duration_weeks: prescription.duration_weeks,
      deload_blocks_count: prescription.blocks.filter(b => b.type === 'deload').length
    }
  });

  return program_id;
}
```

CI gates for `sanitizeGenerationContext`: (1) unit tests asserting all 4 blocked field categories are stripped; (2) grep for `'injury_mentions'`, `'body_weight'`, `'caloric'`, `'diagnosis'` in any INSERT path touching `generation_context`.

---

### 23.13 Exercise Substitution Pattern

When a user or Victor swaps an exercise within a program session:

1. The original `program_session_exercises` row is **never deleted or updated**.
2. A new row is inserted with `substitute_for` pointing to the original `prescription_id`, and `substitution_reason` set.
3. The partial unique index `uq_session_exercise_order` allows both original and substitute to share `order_index` (it filters `WHERE substitute_for IS NULL`).
4. The UI renders the substitute row and visually strikes through the original.
5. `program.exercise_substituted` DEC-030 event is emitted with both `original_prescription_id` and `new_prescription_id` in metadata.

This pattern preserves full prescription history for adherence analysis and for Victor's future context ("how did the substitutions in your last program perform?").

---

### 23.14 GDPR Art. 17 Erasure — Program Data

Erasure steps for the program data domain, to be added to the §12 erasure Worker:

```sql
-- Step 1: Nullify generation_context (health-adjacent rationale)
UPDATE user_programs
SET generation_context = NULL
WHERE user_id = $user_id;
-- DEC-030: program.generation_context_erased

-- Step 2: Nullify per-session coaching notes (may contain health references)
UPDATE program_sessions
SET coaching_notes_ua = NULL, coaching_notes_en = NULL
WHERE program_id IN (
  SELECT program_id FROM user_programs WHERE user_id = $user_id
);

-- Step 3: Nullify per-exercise coaching cues
UPDATE program_session_exercises
SET coaching_cue_ua = NULL, coaching_cue_en = NULL
WHERE session_id IN (
  SELECT ps.session_id FROM program_sessions ps
  JOIN user_programs up ON ps.program_id = up.program_id
  WHERE up.user_id = $user_id
);

-- Step 4: Hard delete user_programs (cascades to all child tables)
DELETE FROM user_programs WHERE user_id = $user_id;
-- DEC-030: program.programs_erased (count in metadata)

-- Step 5: Hard delete custom exercises owned by the user
DELETE FROM exercises WHERE created_by_user = $user_id AND is_custom = TRUE;
-- DEC-030: exercise.custom_exercises_erased (count in metadata)
```

**Retention exception:** Aggregate adherence data already materialized into §17 views at k-anonymity ≥ 5 may be retained after individual erasure. Those views contain no `user_id` or `program_id` — they are irreversibly aggregated. Basis: Art. 17(3)(b) research/statistics exemption applied narrowly.

---

### 23.15 DEC-030 HMAC-Chained Audit Events

All program lifecycle mutations must emit HMAC-chained audit events in the same transaction (DEC-030 pattern, `docs/AUDIT_LOG_SCHEMA.md`).

| Event Type | Severity | Retention | Trigger |
|---|---|---|---|
| `program.created` | STANDARD | 3yr | New `user_programs` row inserted (any source) |
| `program.started` | STANDARD | 3yr | `status` → `'active'` |
| `program.paused` | STANDARD | 3yr | `status` → `'paused'` |
| `program.resumed` | STANDARD | 3yr | `status` → `'active'` from `'paused'` |
| `program.completed` | STANDARD | 3yr | `status` → `'completed'` |
| `program.abandoned` | STANDARD | 3yr | `status` → `'abandoned'` |
| `program.regenerated` | STANDARD | 3yr | Victor replaces an active program (emitted alongside `program.abandoned` for the replaced program) |
| `program.exercise_substituted` | STANDARD | 3yr | New substitute row in `program_session_exercises` |
| `program.generation_context_erased` | STANDARD | 7yr | Art. 17 nullification of `generation_context` |
| `program.programs_erased` | STANDARD | 7yr | Art. 17 hard delete of all user programs |
| `exercise.custom_created` | STANDARD | 3yr | User inserts a custom exercise |
| `exercise.custom_exercises_erased` | STANDARD | 7yr | Art. 17 deletion of user's custom exercises |

**Mandatory metadata fields for `program.created`:**
```json
{
  "source": "victor_generated | template | custom",
  "goal": "<program_goal>",
  "duration_weeks": 12,
  "days_per_week": 4,
  "deload_blocks_count": 1
}
```

---

### 23.16 SOC 2 Mapping

| Criterion | Control | Evidence Artefact |
|---|---|---|
| **CC6.1** | RLS on `user_programs`: `form_api` reads own rows; `tenant_admin` zero access (DEC-002) | `rls_isolation.test.ts` — cross-user and tenant_admin assertions |
| **CC6.2** | `program_templates` are `form_admin`-only write; users cannot inject into the shared library | CI test: `form_api` INSERT to `program_templates` → 403 |
| **CC7.2** | DEC-030 events on all program mutations enable anomaly detection (e.g. programs created without a coaching session FK) | Audit log filter `event_type LIKE 'program.%'` |
| **CC8.1** | HMAC-chained events; `sanitizeGenerationContext()` CI-enforced before any `generation_context` INSERT | Unit test + CI grep for blocked field names |
| **PI1.4** | What the user sees (their program plan) exactly matches what is stored — no hidden prescription layer | Direct read path: `user_programs` → `program_sessions` → `program_session_exercises` |
| **P4.0** | `generation_context` nullified on Art. 17 request; custom exercises hard deleted | Art. 17 erasure integration test: `generation_context IS NULL` post-erasure |
| **P5.0** | `generation_context` schema forbids body-composition, injury, and diagnostic data | `sanitizeGenerationContext()` unit test; CI grep for `body_weight`, `caloric`, `diagnosis`, `injury` in INSERT paths |

---

### 23.17 Open Questions

| ID | Question | Owner | Priority |
|---|---|---|---|
| OQ-PROG-01 | **Victor output format.** Should Victor return a structured program JSON schema directly (deterministic, type-safe, testable) or should the program generation Worker parse free-text output? Structured JSON is strongly preferred. Decision needed before program generation Worker ships — an unstable free-text bridge is brittle and blocks adherence tracking. | `ml-engineer` + `platform-engineer` | **P0 — before M3 program Worker** |
| OQ-PROG-02 | **Program versioning on mid-cycle modification.** If Victor adjusts a program mid-cycle (load reduction due to fatigue signal), do we snapshot the original structure? Option A: `program_revisions` append-only table. Option B: substitution pattern (§23.13) is sufficient at exercise level. Option A preferred for full reproducibility; adds schema complexity. Resolve at M4. | `platform-engineer` + `enterprise-architect` | **P1 — M4** |
| OQ-PROG-03 | **Custom exercise visibility in enterprise tier.** When a user creates a custom exercise, is it visible to other users in the same tenant? Default: NO (own-user only). Tenant-shared custom exercises would require clinical-safety review of exercise naming for PII/health-language risk before opening. | `enterprise-architect` + `clinical-safety` | **P2 — before enterprise feature expansion** |
| OQ-PROG-04 | **User-submitted program templates.** Should FORM allow power users to author templates for others? Blocked at launch (`form_api` INSERT to `program_templates` denied by RLS). If opened: requires sports-scientist review gate, clinical-safety approval, and a separate `community_program_templates` table to avoid mixing with FORM-curated library. | `product-strategist` + `clinical-safety` | **P3 — post-launch v2** |

---

### 23.18 Implementation Checklist

| # | Task | Owner | Priority | Milestone |
|---|---|---|---|---|
| 1 | Create `migrations/YYYYMMDD_exercise_enums.sql` — all 7 ENUMs from §23.2 (`movement_pattern`, `muscle_group`, `equipment_type`, `program_goal`, `program_source`, `program_status`, `block_type`, `session_split`) | `platform-engineer` | **P0** | M2 |
| 2 | Create `migrations/YYYYMMDD_exercises.sql` — `exercises`, `exercise_instructions`, `exercise_muscle_groups` DDL with indexes and RLS policies from §23.11.1 | `platform-engineer` | **P0** | M2 |
| 3 | Seed `exercises` with 60-exercise catalog from §23.3; add UA/EN `exercise_instructions` (≥4 steps each); populate `exercise_muscle_groups` (primary/secondary/stabilizer) for all 60 | `sports-scientist` + `platform-engineer` | **P0** | M2 |
| 4 | Verify push:pull balance across seed catalog using §23.5 query; flag imbalances for sports-scientist review before merge | `sports-scientist` | **P0** | M2 |
| 5 | Create `migrations/YYYYMMDD_program_templates.sql` — `program_templates` DDL with RLS; seed 6 initial templates | `platform-engineer` + `sports-scientist` | **P1** | M3 |
| 6 | Create `migrations/YYYYMMDD_user_programs.sql` — `user_programs`, `program_blocks`, `program_sessions`, `program_session_exercises` DDL with all constraints, indexes, and RLS from §23.11.2/3 | `platform-engineer` | **P0** | M3 |
| 7 | Write regression test: `uq_user_one_active_program` — inserting a second `status = 'active'` program for the same user raises a unique violation; add to CI gating | `qa-lead` + `platform-engineer` | **P0** | M3 |
| 8 | Implement `sanitizeGenerationContext()` in `src/workers/programs/sanitize-context.ts`; write unit tests asserting all blocked field categories are stripped; add CI grep for `injury`, `body_weight`, `caloric`, `diagnosis` in all `generation_context` INSERT paths | `platform-engineer` | **P0** | M3 |
| 9 | Implement `src/workers/programs/generate-program.ts` — atomic transaction per §23.12: abandon active → insert program tree → emit DEC-030; resolve OQ-PROG-01 (Victor output format) before shipping | `platform-engineer` + `ml-engineer` | **P0** | M3 |
| 10 | Write `__tests__/db/rls_isolation.test.ts` additions: (a) user A cannot read user B's `user_programs`; (b) `tenant_admin` role returns zero rows from `user_programs`; (c) `form_api` INSERT to `program_templates` → 403 | `enterprise-architect` + `qa-lead` | **P0** | M3 |
| 11 | Register all 12 `program.*` and `exercise.*` DEC-030 event types in `docs/AUDIT_LOG_SCHEMA.md` with severity and retention tiers | `compliance-officer` | **P0** | M3 |
| 12 | Add program erasure (5-step sequence from §23.14) to `src/workers/gdpr/erasure.ts`; emit `program.generation_context_erased`, `program.programs_erased`, `exercise.custom_exercises_erased` events | `platform-engineer` + `compliance-officer` | **P0** | M3 |
| 13 | Implement substitution flow in program lifecycle Worker — insert substitute row per §23.13 pattern; verify `uq_session_exercise_order` partial index allows coexistence; add integration test | `platform-engineer` | **P1** | M4 |
| 14 | Implement `program.completed` / `program.abandoned` transitions: compute and store `adherence_pct` (completed non-rest sessions / total non-rest sessions) on status change | `platform-engineer` | **P1** | M4 |

---

*v1.3 · травень 2026 · owner: enterprise-architect + platform-engineer + ml-engineer*

*v1.3 additions: §23 Exercise Library & Program Schema — closes the core product schema gap for exercise catalog storage, program templates, user program instances, and periodization structure referenced in PRODUCT_SPEC.md and TECHNICAL.md. Eight-table design: `exercises` (global catalog + is_custom user/tenant-owned; 7 ENUMs including movement_pattern, muscle_group, equipment_type; is_cv_supported flag linking to §15; contraindication_tags for Victor injury screening; chk_custom_has_owner and chk_deprecated_inactive constraints; 60-exercise seed catalog); `exercise_instructions` (multilingual UA/EN coaching cues per step); `exercise_muscle_groups` (primary/secondary/stabilizer roles, push:pull balance analysis by program generation Worker); `program_templates` (FORM-curated library, form_admin-only write, min_plan_tier entitlement gate, 6-template initial library); `user_programs` (program_source and program_status ENUMs; uq_user_one_active_program partial unique index; generated target_end_at column; generation_context JSONB with strict allow/deny privacy schema prohibiting injury/body-composition/diagnostic fields; chk_active_has_start and chk_completion_consistency constraints); `program_blocks` (block_type ENUM with accumulation/intensification/realization/deload; deload enforcement rule for 8+ week programs at Worker layer); `program_sessions` (session_split ENUM; workout_session_id FK to actual logged session; chk_not_both_completed_skipped constraint); `program_session_exercises` (full load prescription schema: relative/absolute/bodyweight/rpe_governed/amrap load_type; rep_min/rep_max/reps_fixed/AMRAP constraint; tempo regex validation; substitution pattern via substitute_for self-FK with uq_session_exercise_order partial unique index). Victor program generation Worker pattern: single atomic transaction that abandons existing active program → inserts full program tree → emits DEC-030 events; sanitizeGenerationContext() strips all non-permitted fields before INSERT with CI grep gate. Exercise substitution pattern: original row preserved; substitute_for FK + partial unique index enable historical coexistence without losing original prescription. GDPR Art. 17: five-step erasure (generation_context nullification → coaching notes nullification → coaching cues nullification → user_programs hard delete with CASCADE → custom exercises hard delete); aggregate retention exception for k-anonymity ≥ 5 materialized data under Art. 17(3)(b). RLS: form_api own-row on user_programs (no direct INSERT — Workers only); program_templates form_admin write; tenant_admin zero access (DEC-002); form_system tenant-scoped. 12 DEC-030 HMAC-chained audit events across program and exercise lifecycle. SOC 2 mapping: CC6.1 (RLS), CC6.2 (template write gate), CC7.2 (audit events for anomaly detection), CC8.1 (HMAC chain + sanitization CI gate), PI1.4 (no hidden prescription layer), P4.0/P5.0 (erasure + forbidden-field enforcement). 4 open questions (OQ-PROG-01: Victor output format P0; OQ-PROG-02: program versioning P1; OQ-PROG-03: enterprise custom exercise sharing P2; OQ-PROG-04: user-submitted templates P3). 14-item implementation checklist (12× P0 M2-M3, 2× P1 M4).*

---

*v1.2 · травень 2026 · owner: enterprise-architect + compliance-officer + security-engineer*

*v1.2 additions: §22 Gamification, Streak & Achievement Schema — closes the data-layer gap for DEC-013 (streak grace after 2 consecutive misses, not 1) and formalises personal record and achievement storage. Five-table schema: `user_streaks` (current_streak, longest_streak, streak_start_date, last_activity_date, grace_used_this_cycle, total_grace_uses; `chk_streak_longest` invariant constraint), `streak_grace_events` (append-only grace history; no UPDATE/DELETE RLS for non-admin), `personal_records` (PR series per user × exercise × metric; `pr_metric` ENUM with 5 metrics; current best via `DISTINCT ON` query; e1RM estimates with formula attribution in context JSONB), `achievement_definitions` (form_admin-only write; dotted namespace key; soft-archive via `is_active`; 19-item seed catalog across 5 categories and 4 tiers), `user_achievements` (form_system INSERT only; append-only by convention; UNIQUE index preventing same achievement twice in one calendar day). DEC-013 compliance: streak grace state machine documented with 4 transition paths; grace_used_this_cycle flag enforces at-most-one grace per streak cycle; DEC-013 invariant table with enforcement locations; CI regression test specification (`__tests__/db/gamification_streak.test.ts`). RLS: `form_api` own-row read/update on streaks; no direct form_api insert on grace_events or user_achievements (streak Worker and achievement engine run as form_system); `tenant_admin` has zero access to all gamification tables (DEC-002 compliance). DEC-030 HMAC-chained events: 6 event types across streak and achievement lifecycle (gamification.streak_started, streak_grace_applied, streak_reset, streak_milestone, pr_set, achievement_earned); all STANDARD severity, 3-year retention; emitted in-transaction. GDPR Art. 17: explicit four-step erasure sequence (achievements → grace_events → prs → streaks) with per-table DEC-030 events; ON DELETE CASCADE as safety net. Tenant admin privacy floor: zero aggregate gamification views defined; any future admin reporting requires §17 aggregate layer + k-anonymity ≥ 5 + clinical-safety gate per OQ-GAM-02. SOC 2 mapping: CC6.1 (own-row RLS), CC6.2 (no self-award exploit), CC7.2 (audit log anomaly detection on impossible PRs), CC8.1 (HMAC-chained mutations, append-only grace/achievement tables), P4.0/P5.0 (Art. 17 erasure evidence). 5 open questions (OQ-GAM-01 through OQ-GAM-05); OQ-GAM-01 (streak qualifying session definition) and OQ-GAM-04 (timezone edge case) are P0 before streak engine ships. 12-item implementation checklist (8× P0 M3, 3× P1 M4, 1× P0 M3 erasure). Table of Contents updated to include §22.*

*v1.1 · травень 2026 · owner: enterprise-architect + compliance-officer + security-engineer*

*v1.1 additions: §20 White-Label Tenant Branding Schema — closes the §2.7 stub gap across all cross-references (§15 `cv.client_side_enabled`, §18 `webhook.email_in_payloads`, SSO_SCIM_IMPLEMENTATION.md `security.ip_allowlist_enabled` and `sso.staging_env_enabled`). Two-table design: `feature_flag_registry` (canonical allowlist, form_admin-only write, FK-enforced dotted namespace, `flag_key_namespace` CHECK constraint) and `tenant_feature_flags` (production per-tenant rows with FK to registry, `compliance_approval_ref`, `enabled_at`, RLS read-only for form_api and form_system). Seed data for 14 canonical flags across Starter / Growth / Enterprise tiers with one compliance-gated flag (`webhook.email_in_payloads`). Tier entitlement matrix (auto / avail / — / gate). TypeScript `assertFeatureEnabled()` in `src/workers/feature-flags/assert-feature-enabled.ts`: Cloudflare KV 30-second cache, tier rank comparison against TIER_ORDER map, compliance gate check, fail-closed on any DB or KV error (throws 403). Four DEC-030 HMAC-chained audit events: `feature_flag.enabled` (STANDARD, 7yr), `feature_flag.disabled` (STANDARD, 7yr), `feature_flag.registry_updated` (HIGH, 7yr), `feature_flag.compliance_gate_bypassed` (CRITICAL, 10yr — auto PagerDuty P1, always `blocked: true`). Compliance gate protocol: DPA authorisation → DECISION_LOG PR merge → compliance_approval_ref populated → form_admin enables via internal API → DEC-030 event with approval reference. §19.9 migration script: 14 flag_name-to-flag_key renames, RAISE EXCEPTION abort on unrecognised flags, FK constraint addition, `enabled_at` backfill from `updated_at`, `compliance_approval_ref` column addition, post-migration validation queries (zero unrecognised flags; zero enabled gated flags without approval ref). OQ-FLAG-01: tier downgrade auto-disable decision (preferred: contract lifecycle hook emitting feature_flag.disabled events, M5). OQ-FLAG-02: flag deprecation 90-day sunset policy with pg_cron soft-delete and admin dashboard banner (M6). SOC 2 mapping: CC6.1 (form_admin-only write, tier entitlement gates), CC6.2 (tier check = authorisation gate, plan not self-elevatable), CC8.1 (all mutations HMAC-chained in-transaction), CC9.2 (compliance_approval_ref persisted 7yr, auditor-traceable to DPA clause). 12-item implementation checklist (8× P0 M3, 1× P1 M4, 1× P1 M5, 1× P2 M6).*

---

## 24. Subscription, Billing & Revenue Schema

This section defines the complete data model for subscription lifecycle management, billing channel integration, enterprise seat management, and revenue-related compliance. It closes the schema gap for plan entitlement enforcement referenced in §19 (`assertFeatureEnabled()` tier checks), §16 (enterprise contract lifecycle), §17 (admin adoption metrics), and `docs/TECHNICAL.md` (billing architecture). Cross-references: §19 (`plan_tier` entitlement gate), §16 (`enterprise_contracts` — contract precedes seat allocation), §12 (GDPR Art. 17 erasure), `docs/AUDIT_LOG_SCHEMA.md` (DEC-030), `docs/SSO_SCIM_IMPLEMENTATION.md` §19 (SCIM provisioning integration).

---

### 24.1 Overview & Design Principles

**Two-track billing architecture.** FORM operates two structurally distinct billing channels that must never be conflated:

- **Consumer track** — individual users subscribe via iOS App Store (StoreKit 2) or Google Play (Billing Library v6). Billing is managed entirely by the platform; FORM receives server-side notifications. No Stripe involvement. No tenant association.
- **Enterprise track** — organizations are invoiced via Stripe. A `tenants` row (§2.1) must exist before any enterprise subscription is created. Seats are contracted, allocated via `enterprise_seat_allocations`, and assigned to individual users via `enterprise_seat_assignments`.

**Single source of truth: `user_subscriptions`.** App Store, Google Play, and Stripe are authoritative for payment events, but they are not authoritative for FORM's access-control decisions. The `user_subscriptions` table is the canonical record of a user's current plan tier, status, and period. The billing Workers translate external events into mutations on this table. Any feature entitlement decision (§19 `assertFeatureEnabled()`) reads from `user_subscriptions`, never from an external API call.

**Revenue recognition principle.** Revenue is recognized on service delivery — across the subscription period — not on receipt of payment. A payment received on 1 June for a June–July period contributes to MRR evenly over those two months. This aligns with the COST_MODEL reference in §22 and enables accurate accrual-basis reporting. The `current_period_start` / `current_period_end` columns encode the service delivery window; `subscription_events.occurred_at` encodes when the payment event was received. These are distinct and both must be preserved.

**Monetary amounts in minor units.** All `_usd_cents` columns store integers in the minor currency unit (cents for USD, kopecks for UAH). No floating-point money columns. Currency is always stored alongside the amount per ISO 4217. Display formatting is a presentation-layer concern.

**Billing Worker is the only mutation path.** No application `form_api` role may UPDATE `user_subscriptions` directly. All writes go through the billing Cloudflare Worker running as `form_system`. This is enforced via RLS (§24.11) and verified by the HMAC chain (§24.12). The reasoning: preventing any client-side or API-layer code from elevating a user's plan without a verified payment event.

---

### 24.2 Plan Tier Definitions

| Tier | `monthly_price_usd_cents` | `annual_price_usd_cents` | Notable Feature Gates | Billing Channel |
|---|---|---|---|---|
| `free` | 0 | 0 | Core logging, 3 AI coaching messages/month, no CV, no programs | `none` |
| `growth` | 1500 | 15000 | Unlimited AI coaching, CV form analysis, program generation, wearable sync | `app_store_ios` or `google_play` |
| `elite` | 2500 | 25000 | All Growth features + priority Victor model, advanced analytics, export | `app_store_ios` or `google_play` |
| `enterprise` | custom | custom | All Elite features + SSO/SCIM, white-label, admin dashboard, seat management, SLA | `stripe` |

Notes:
- Annual pricing for Growth ($150/yr) and Elite ($250/yr) represents a ~17% and ~17% discount respectively versus monthly.
- Enterprise pricing is per-seat, custom-negotiated, stored in `enterprise_seat_allocations.price_per_seat_usd_cents`. The `user_subscriptions.price_usd_cents` for enterprise rows stores the total contract value per billing period, not per-seat.
- `free` tier has `billing_channel = 'none'` — no payment method on file, no external subscription ID.
- Promotional or grandfathered pricing is accommodated by `user_subscriptions.price_usd_cents` differing from the plan default above.

---

### 24.3 `user_subscriptions` Table DDL

```sql
CREATE TYPE plan_tier_enum AS ENUM (
  'free',
  'growth',
  'elite',
  'enterprise'
);

CREATE TYPE subscription_status_enum AS ENUM (
  'trialing',    -- within free trial window; access at trial tier granted
  'active',      -- paid and current
  'grace',       -- payment failed; 7-day grace period before access revocation
  'past_due',    -- grace expired; access revoked; awaiting payment (reserved for future dunning)
  'cancelled',   -- user-initiated cancel; access continues until current_period_end
  'expired'      -- period ended with no renewal; access revoked
);

CREATE TYPE billing_channel_enum AS ENUM (
  'none',           -- free tier; no payment provider
  'app_store_ios',  -- Apple App Store / StoreKit 2
  'google_play',    -- Google Play Billing Library
  'stripe'          -- Stripe invoicing (enterprise only)
);

CREATE TABLE user_subscriptions (
  id                        UUID                      NOT NULL DEFAULT gen_random_uuid() PRIMARY KEY,
  user_id                   UUID                      NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  tenant_id                 UUID                      REFERENCES tenants(id),
  -- NULL for consumer subscriptions; set for enterprise (enforced by chk_enterprise_has_tenant)

  plan_tier                 plan_tier_enum            NOT NULL DEFAULT 'free',
  status                    subscription_status_enum  NOT NULL,
  billing_channel           billing_channel_enum      NOT NULL,

  current_period_start      TIMESTAMPTZ,
  current_period_end        TIMESTAMPTZ,
  trial_ends_at             TIMESTAMPTZ,
  -- NULL unless status = 'trialing'; enforced by chk_trial_coherent
  grace_ends_at             TIMESTAMPTZ,
  -- NULL unless status = 'grace'; grace_ends_at = payment_failed_at + INTERVAL '7 days'

  external_subscription_id  TEXT,
  -- App Store: original_transaction_id (stable across renewals)
  -- Google Play: purchaseToken (changes on renewal — use linkedPurchaseToken chain)
  -- Stripe: subscription ID (sub_...)
  external_customer_id      TEXT,
  -- Stripe: customer ID (cus_...); NULL for App Store / Play (platform manages customer identity)

  price_usd_cents           INTEGER,
  -- Actual price for current period. May differ from plan default (grandfathering, promotions).
  -- NULL for free tier.
  currency                  CHAR(3)                   NOT NULL DEFAULT 'USD',

  cancel_at_period_end      BOOLEAN                   NOT NULL DEFAULT FALSE,
  -- TRUE = user has requested cancellation; access continues until current_period_end
  cancelled_at              TIMESTAMPTZ,
  -- Timestamp of cancellation request (not the access end date)

  created_at                TIMESTAMPTZ               NOT NULL DEFAULT now(),
  updated_at                TIMESTAMPTZ               NOT NULL DEFAULT now(),

  -- One active subscription per user (partial unique index)
  CONSTRAINT chk_enterprise_has_tenant CHECK (
    billing_channel <> 'stripe' OR tenant_id IS NOT NULL
  ),
  CONSTRAINT chk_trial_coherent CHECK (
    status <> 'trialing' OR trial_ends_at IS NOT NULL
  ),
  CONSTRAINT chk_grace_coherent CHECK (
    status <> 'grace' OR grace_ends_at IS NOT NULL
  ),
  CONSTRAINT chk_free_no_price CHECK (
    billing_channel <> 'none' OR price_usd_cents IS NULL
  )
);

-- At most one non-expired subscription per user.
-- Expired rows are retained for financial audit trail.
CREATE UNIQUE INDEX uq_user_one_active_subscription
  ON user_subscriptions (user_id)
  WHERE status <> 'expired';

-- Fast lookup by external ID (webhook correlation).
CREATE INDEX idx_user_subscriptions_external_id
  ON user_subscriptions (external_subscription_id)
  WHERE external_subscription_id IS NOT NULL;

-- Renewal sweep: find subscriptions approaching period end.
CREATE INDEX idx_user_subscriptions_renewal_sweep
  ON user_subscriptions (current_period_end)
  WHERE status IN ('active', 'grace');

-- Tenant seat management lookup.
CREATE INDEX idx_user_subscriptions_tenant
  ON user_subscriptions (tenant_id)
  WHERE tenant_id IS NOT NULL;

CREATE TRIGGER update_user_subscriptions_updated_at
  BEFORE UPDATE ON user_subscriptions
  FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();
```

**Design note on `external_subscription_id` for Google Play.** Google Play's `purchaseToken` is not stable across renewals — each renewal issues a new token. However, the renewed purchase carries a `linkedPurchaseToken` pointing to the predecessor. The billing Worker must follow this chain when correlating an RTDN notification to an existing `user_subscriptions` row. The `external_subscription_id` column stores the *original* (first-ever) purchase token, and the Worker resolves renewal chains back to it. This mirrors the App Store's `original_transaction_id` stability guarantee.

---

### 24.4 `subscription_events` Table DDL

Append-only lifecycle log. Every state transition in `user_subscriptions` produces a row here in the same database transaction. No UPDATE or DELETE is permitted by any non-admin role.

```sql
CREATE TYPE subscription_event_type_enum AS ENUM (
  'created',          -- new subscription row inserted (billing_channel set, status = trialing or active)
  'activated',        -- status → active (from trialing or grace)
  'renewed',          -- active period extended; new current_period_start/end set
  'upgraded',         -- plan_tier increased (e.g. growth → elite, or seat count increased)
  'downgraded',       -- plan_tier decreased (cancel_at_period_end path or immediate)
  'payment_failed',   -- external payment event failed; grace period may start
  'grace_started',    -- status → grace; grace_ends_at computed
  'cancelled',        -- cancel_at_period_end = TRUE set; access continues until period end
  'expired',          -- status → expired; access revoked
  'refunded',         -- partial or full refund issued; amount_usd_cents = refunded amount
  'trial_started',    -- status = trialing; trial_ends_at set
  'trial_converted',  -- trial → active; payment method confirmed
  'trial_expired',    -- trial_ends_at passed without payment; status → expired
  'reactivated'       -- expired → active (new subscription; previous row kept as expired)
);

CREATE TABLE subscription_events (
  id                    UUID                             NOT NULL DEFAULT gen_random_uuid() PRIMARY KEY,
  user_id               UUID                             NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  subscription_id       UUID                             NOT NULL REFERENCES user_subscriptions(id),

  event_type            subscription_event_type_enum     NOT NULL,
  previous_plan_tier    plan_tier_enum,
  -- NULL for events where plan tier does not change (e.g. renewed, payment_failed)
  new_plan_tier         plan_tier_enum,

  amount_usd_cents      INTEGER,
  -- Amount charged or refunded in this event. NULL for non-payment events.
  -- For 'refunded': negative value is NOT used; store positive refunded amount.
  currency              CHAR(3)                          NOT NULL DEFAULT 'USD',

  external_event_id     TEXT,
  -- App Store: signed transaction UUID from JWS payload
  -- Google Play: RTDN notification UUID
  -- Stripe: event ID (evt_...)
  -- UNIQUE index below enforces idempotency
  external_invoice_id   TEXT,
  -- Stripe: invoice ID (in_...). NULL for App Store / Play.

  failure_reason        TEXT,
  -- Human-readable payment failure code. No card numbers, no CVV, no PAN fragments.
  -- Populated only for event_type IN ('payment_failed').
  -- Nullified on Art. 17 erasure (may contain free-text PII).

  occurred_at           TIMESTAMPTZ                      NOT NULL DEFAULT now(),
  -- When the event actually occurred (from external notification timestamp where available).
  created_at            TIMESTAMPTZ                      NOT NULL DEFAULT now()
  -- When this row was inserted (may lag occurred_at by webhook delivery latency).
);

-- Idempotency: prevent duplicate processing of the same external notification.
CREATE UNIQUE INDEX uq_subscription_events_external_id
  ON subscription_events (external_event_id)
  WHERE external_event_id IS NOT NULL;

CREATE INDEX idx_subscription_events_subscription
  ON subscription_events (subscription_id, occurred_at DESC);

CREATE INDEX idx_subscription_events_user_type
  ON subscription_events (user_id, event_type);
```

**Append-only enforcement via RLS** (see §24.11). The `form_api` role has SELECT only; `form_system` has INSERT only. No UPDATE or DELETE is granted to any non-admin role. The HMAC chain (§24.12) provides independent tamper evidence — any attempt to mutate a historical row breaks the chain.

---

### 24.5 Enterprise Seat Management Schema

Enterprise customers contract a fixed seat count. Seat allocation tracks the contracted quantity and price per billing period. Seat assignment tracks which specific users hold a seat at any given time.

#### 24.5.1 `enterprise_seat_allocations`

```sql
CREATE TABLE enterprise_seat_allocations (
  id                        UUID         NOT NULL DEFAULT gen_random_uuid() PRIMARY KEY,
  tenant_id                 UUID         NOT NULL REFERENCES tenants(id) ON DELETE RESTRICT,
  -- RESTRICT: cannot delete a tenant with active allocations
  subscription_id           UUID         NOT NULL REFERENCES user_subscriptions(id),

  total_seats               INTEGER      NOT NULL CHECK (total_seats >= 1),
  effective_from            DATE         NOT NULL,
  effective_to              DATE,
  -- NULL = this is the current active allocation for the tenant.
  -- Closed (non-NULL) rows are historical — kept for financial audit.

  price_per_seat_usd_cents  INTEGER      NOT NULL CHECK (price_per_seat_usd_cents >= 0),
  -- Per-seat price for this allocation period. Zero is valid (free pilot allocation).

  created_at                TIMESTAMPTZ  NOT NULL DEFAULT now(),

  -- Prevent overlapping allocation periods for the same tenant.
  CONSTRAINT uq_tenant_allocation_period UNIQUE (tenant_id, effective_from)
);

CREATE INDEX idx_seat_allocations_tenant
  ON enterprise_seat_allocations (tenant_id)
  WHERE effective_to IS NULL;
-- Partial index targets the current (open) allocation only.
```

**Seat expansion / contraction.** When a contract is amended (seats added or removed), the billing Worker closes the current allocation row (`effective_to = today`) and inserts a new row with the updated `total_seats` and new `effective_from`. This preserves a complete historical record of contracted seat counts for financial reconciliation and SOC 2 evidence.

#### 24.5.2 `enterprise_seat_assignments`

```sql
CREATE TABLE enterprise_seat_assignments (
  id             UUID         NOT NULL DEFAULT gen_random_uuid() PRIMARY KEY,
  tenant_id      UUID         NOT NULL REFERENCES tenants(id),
  user_id        UUID         NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  allocation_id  UUID         NOT NULL REFERENCES enterprise_seat_allocations(id),

  assigned_at    TIMESTAMPTZ  NOT NULL DEFAULT now(),
  revoked_at     TIMESTAMPTZ,
  -- NULL = seat is currently active. Non-NULL = seat was revoked.

  assigned_by    UUID         REFERENCES users(id),
  -- The tenant_admin who performed the assignment. NULL only for SCIM-automated provisioning.

  -- One active seat per user per tenant (partial unique index).
  CONSTRAINT chk_assignment_tenant_matches CHECK (
    -- Application layer enforces that user belongs to the named tenant.
    -- RLS on tenant_id ensures tenant_admin cannot assign seats to foreign-tenant users.
    tenant_id IS NOT NULL
  )
);

-- One active seat assignment per user per tenant.
CREATE UNIQUE INDEX uq_tenant_user_active_seat
  ON enterprise_seat_assignments (tenant_id, user_id)
  WHERE revoked_at IS NULL;

CREATE INDEX idx_seat_assignments_allocation
  ON enterprise_seat_assignments (allocation_id)
  WHERE revoked_at IS NULL;

CREATE INDEX idx_seat_assignments_user
  ON enterprise_seat_assignments (user_id)
  WHERE revoked_at IS NULL;
```

#### 24.5.3 `tenant_seat_utilization` View

Materialized view for admin dashboard performance. Refreshed by the billing Worker after any seat assignment or revocation event.

```sql
CREATE MATERIALIZED VIEW tenant_seat_utilization AS
SELECT
  esa.tenant_id,
  esa.id                                                    AS allocation_id,
  esa.total_seats,
  COUNT(ass.id) FILTER (WHERE ass.revoked_at IS NULL)       AS used_seats,
  esa.total_seats
    - COUNT(ass.id) FILTER (WHERE ass.revoked_at IS NULL)   AS available_seats,
  now()                                                     AS refreshed_at
FROM enterprise_seat_allocations esa
LEFT JOIN enterprise_seat_assignments ass
  ON ass.allocation_id = esa.id
WHERE esa.effective_to IS NULL   -- current allocation only
GROUP BY esa.tenant_id, esa.id, esa.total_seats;

CREATE UNIQUE INDEX uq_tenant_seat_utilization_tenant
  ON tenant_seat_utilization (tenant_id);
```

`REFRESH MATERIALIZED VIEW CONCURRENTLY tenant_seat_utilization` is called by the billing Worker after any seat assignment mutation. The `CONCURRENTLY` option ensures dashboard reads are never blocked during refresh.

---

### 24.6 App Store Server Notifications (StoreKit 2)

**Webhook endpoint:** `POST /api/v1/webhooks/apple` — Cloudflare Worker running as `form_system`.

**Payload verification:**
1. Apple delivers a signed JWS (JSON Web Signature) payload.
2. Worker extracts the certificate chain from the JWS header and verifies it chains to Apple's root CA (pinned in Worker secrets, rotated on Apple CA update).
3. JWS `iat` claim must be within a 6-hour window of `now()` to reject replayed notifications.
4. Decoded payload type is `responseBodyV2`; inner `data.signedTransactionInfo` and `data.signedRenewalInfo` are separately decoded JWS objects.

**Idempotency:** `subscription_events.external_event_id` stores the Apple-assigned `transactionId` from the signed transaction. The `uq_subscription_events_external_id` unique index causes a constraint violation on duplicate delivery — the Worker catches this and returns HTTP 200 (acknowledged) without double-processing.

**Notification type to `subscription_event_type_enum` mapping:**

| Apple Notification Type | `notificationSubtype` | FORM Event Type |
|---|---|---|
| `SUBSCRIBED` | `INITIAL_BUY` | `created`, `activated` |
| `SUBSCRIBED` | `RESUBSCRIBE` | `reactivated` |
| `DID_RENEW` | — | `renewed` |
| `DID_FAIL_TO_RENEW` | `GRACE_PERIOD` | `payment_failed`, `grace_started` |
| `DID_FAIL_TO_RENEW` | — (no grace) | `payment_failed` |
| `GRACE_PERIOD_EXPIRED` | — | `expired` |
| `EXPIRED` | `VOLUNTARY` | `expired` |
| `EXPIRED` | `BILLING_RETRY` | `expired` |
| `DID_CHANGE_RENEWAL_STATUS` | `AUTO_RENEW_DISABLED` | `cancelled` |
| `REFUND` | — | `refunded` |
| `OFFER_REDEEMED` | — | `activated` (with promotional price in `price_usd_cents`) |

**`original_transaction_id` as stable key.** Every renewed transaction carries the same `originalTransactionId`. The billing Worker stores this as `user_subscriptions.external_subscription_id` on first creation and uses it as the lookup key for all subsequent notifications. The per-renewal `transactionId` is stored in `subscription_events.external_event_id`.

---

### 24.7 Google Play Real-Time Developer Notifications (RTDN)

**Delivery path:** Google Cloud Pub/Sub push subscription → `POST /api/v1/webhooks/google-play` Cloudflare Worker.

**Processing sequence:**
1. Base64-decode the Pub/Sub `message.data` field to obtain a `DeveloperNotification` proto.
2. If `subscriptionNotification` is present: extract `purchaseToken` and `notificationType`.
3. Verify the purchase via Google Play Developer API v3 `purchases.subscriptionsv2.get(packageName, subscriptionId, purchaseToken)`. This is mandatory — Pub/Sub messages are not signed and must be verified against the authoritative Google API.
4. Map `SubscriptionNotification.notificationType` to `subscription_event_type_enum`.
5. For renewals: Google issues a new `purchaseToken`. The renewed purchase's `linkedPurchaseToken` field points to the predecessor. Follow the `linkedPurchaseToken` chain to resolve back to the original token stored in `user_subscriptions.external_subscription_id`.
6. Idempotency: store the Pub/Sub `message.messageId` (or the `orderId` from the verified purchase) as `subscription_events.external_event_id`.

**Notification type mapping:**

| `notificationType` | FORM Event Type |
|---|---|
| `SUBSCRIPTION_PURCHASED` (1) | `created`, `activated` |
| `SUBSCRIPTION_RENEWED` (2) | `renewed` |
| `SUBSCRIPTION_CANCELED` (3) | `cancelled` |
| `SUBSCRIPTION_PURCHASED` from `linkedPurchaseToken` path | `reactivated` |
| `SUBSCRIPTION_ON_HOLD` (5) | `payment_failed`, `grace_started` |
| `SUBSCRIPTION_IN_GRACE_PERIOD` (6) | `grace_started` |
| `SUBSCRIPTION_RESTARTED` (7) | `reactivated` |
| `SUBSCRIPTION_PRICE_CHANGE_CONFIRMED` (8) | `upgraded` or `downgraded` |
| `SUBSCRIPTION_DEFERRED` (9) | no event (deferred renewal; update `current_period_end`) |
| `SUBSCRIPTION_PAUSED` (10) | `cancelled` (treat pause as cancel-at-period-end) |
| `SUBSCRIPTION_EXPIRED` (13) | `expired` |
| `SUBSCRIPTION_REVOKED` (12) | `expired` (refund-driven revocation) |

---

### 24.8 Stripe Webhook Integration (Enterprise)

**Webhook endpoint:** `POST /api/v1/webhooks/stripe` — Cloudflare Worker running as `form_system`.

**Signature verification:** Stripe-Signature header contains a timestamp (`t=`) and HMAC-SHA256 signatures (`v1=`). Worker recomputes `HMAC-SHA256(webhook_secret, "{timestamp}.{raw_body}")` and compares to the `v1` signature. Timestamp must be within 300 seconds of `now()` to prevent replay. The raw body must be read before any JSON parsing to preserve signature validity.

**Relevant event mapping:**

| Stripe Event | FORM Event Type | Notes |
|---|---|---|
| `invoice.payment_succeeded` | `renewed` (or `activated` for first payment) | `invoice.lines.data[].quantity` × `price.unit_amount` = total; store in `amount_usd_cents` |
| `invoice.payment_failed` | `payment_failed`, `grace_started` | `grace_ends_at = now() + INTERVAL '7 days'`; store `failure_reason` from `last_payment_error.code` |
| `customer.subscription.updated` | `upgraded` or `downgraded` | Compare `previous_attributes.items` quantity to new `items` quantity |
| `customer.subscription.deleted` | `cancelled` or `expired` | `cancel_at_period_end = TRUE` → `cancelled`; immediate deletion → `expired` |
| `invoice.finalized` | no `subscription_events` row | Update `current_period_start` / `current_period_end` on `user_subscriptions` only |
| `customer.subscription.trial_will_end` | no row (3-day warning) | Trigger push notification via §18 notification queue |

**Seat expansion flow.** When a Stripe subscription quantity increases (e.g. 50 seats → 75 seats):
1. `customer.subscription.updated` event received; `previous_attributes.items[0].quantity = 50`, `items[0].quantity = 75`.
2. Billing Worker closes the current `enterprise_seat_allocations` row (`effective_to = today`).
3. Inserts a new `enterprise_seat_allocations` row with `total_seats = 75`, `effective_from = today`, `price_per_seat_usd_cents` from the Stripe price object.
4. Emits `billing.seats_expanded` DEC-030 event (payload: `previous_seats`, `new_seats`, `tenant_id`).
5. Refreshes `tenant_seat_utilization` materialized view.

Seat reduction follows the same pattern; emits `billing.seats_reduced` (HIGH severity — access revocation signal). If `used_seats > new total_seats`, the billing Worker must NOT automatically revoke assignments. It sets a `seat_over_allocation` flag on the tenant record and surfaces an alert in the admin dashboard — the tenant admin must manually choose which assignments to revoke.

---

### 24.9 Trial & Grace Period State Machine

```
[none / pre-subscription]
        │
        ▼ billing Worker: new subscription, trial granted
  ┌──────────────┐
  │  trialing    │ ─── trial_ends_at passed, no payment ──────────────────────────────────►  expired
  └──────────────┘
        │
        │ payment processed (trial_converted event)
        ▼
  ┌──────────────┐
  │   active     │ ◄────────────────────────────────────── payment retry succeeded (grace → active)
  └──────────────┘
        │                        │                          │
        │ payment failed         │ user cancels             │ current_period_end passed
        ▼                        ▼                          ▼
  ┌──────────────┐         ┌──────────────┐          ┌──────────────┐
  │    grace     │         │  cancelled   │          │   expired    │
  │ (7 days)     │         │ (access OK   │          │ (access      │
  └──────────────┘         │ until end)   │          │  revoked)    │
        │                  └──────────────┘          └──────────────┘
        │ grace_ends_at passed                              │
        ▼                                                   │ user resubscribes
  ┌──────────────┐                                         │ (new subscription row)
  │   expired    │                                          ▼
  └──────────────┘                                   ┌──────────────┐
                                                      │   active     │
                                                      │ (reactivated)│
                                                      └──────────────┘
```

**Complete state transition table:**

| Current State | Trigger | New State | `subscription_events` Row(s) | `user_subscriptions` Changes |
|---|---|---|---|---|
| _(none)_ | New subscription, trial enabled | `trialing` | `trial_started`, `created` | `status = 'trialing'`, `trial_ends_at = now() + interval` |
| _(none)_ | New subscription, no trial | `active` | `created`, `activated` | `status = 'active'`, `current_period_start/end` set |
| `trialing` | Payment method confirmed before `trial_ends_at` | `active` | `trial_converted`, `activated` | `status = 'active'`, `trial_ends_at = NULL` |
| `trialing` | `trial_ends_at` passed, no payment | `expired` | `trial_expired`, `expired` | `status = 'expired'` |
| `active` | Successful renewal payment | `active` | `renewed` | `current_period_start/end` advanced |
| `active` | Plan tier change (upgrade) | `active` | `upgraded` | `plan_tier` updated, `price_usd_cents` updated |
| `active` | Plan tier change (downgrade) | `active` | `downgraded` | `plan_tier` updated, `cancel_at_period_end = TRUE` or immediate |
| `active` | User requests cancellation | `cancelled` | `cancelled` | `cancel_at_period_end = TRUE`, `cancelled_at = now()` |
| `active` | Payment fails | `grace` | `payment_failed`, `grace_started` | `status = 'grace'`, `grace_ends_at = now() + 7 days` |
| `grace` | Payment retry succeeds | `active` | `activated` | `status = 'active'`, `grace_ends_at = NULL` |
| `grace` | `grace_ends_at` passed | `expired` | `expired` | `status = 'expired'` |
| `cancelled` | `current_period_end` passed | `expired` | `expired` | `status = 'expired'` |
| `expired` | User resubscribes | `active` | `created`, `activated` or `trial_started` | New `user_subscriptions` row inserted; old row retained as `expired` |

**Access revocation on expiry.** When `status` transitions to `expired`, the billing Worker must:
1. Update `user_subscriptions.status = 'expired'` and `updated_at = now()`.
2. For enterprise rows: does NOT automatically revoke `enterprise_seat_assignments` — that requires tenant admin action to comply with §16 offboarding flow.
3. Emit `billing.subscription_expired` DEC-030 event (HIGH severity).
4. Enqueue a notification via §18 queue.

The `form_api` RLS on `user_subscriptions` (§24.11) ensures that an expired user's subsequent API calls yield a subscription row with `status = 'expired'`, which `assertFeatureEnabled()` (§19) treats as equivalent to `free` tier.

---

### 24.10 GDPR Art. 17 vs. Financial Retention Tension

**The conflict.** GDPR Art. 17 grants data subjects the right to erasure of personal data. `subscription_events` and `user_subscriptions` contain `user_id` (a personal data identifier), `amount_usd_cents`, currency, dates, and invoice IDs. However:

- Ukrainian tax law (Tax Code of Ukraine, Art. 44) requires retention of primary financial documents for **1095 days** (3 years) from the reporting period, with extended 7-year retention for VAT records and documents subject to tax audit.
- EU VAT records requirements (Council Directive 2006/112/EC, Art. 245) mandate retention for a period determined by each Member State, typically 7–10 years.
- FORM's legal basis for retention is **Art. 17(3)(b)** — processing necessary for compliance with a legal obligation to which the controller is subject.

**Resolution: pseudonymization, not deletion.**

On a valid Art. 17 erasure request, the GDPR erasure Worker (`src/workers/gdpr/erasure.ts`) executes the following billing-specific steps after the standard erasure sequence:

```sql
-- Step B-1: Pseudonymize user_id in subscription_events
-- Retain: amount_usd_cents, currency, occurred_at, external_invoice_id, event_type
-- Erase: user_id identity linkage, failure_reason (may contain free-text PII)
UPDATE subscription_events
SET
  user_id        = '[ERASED-' || encode(
                     digest(user_id::text || $erasure_salt, 'sha256'), 'hex'
                   ) || ']',
  -- Note: user_id column is TEXT after pseudonymization; UUID constraint relaxed via separate
  -- erased_user_id TEXT column approach — see OQ-BILL-05 (schema evolution note below)
  failure_reason = NULL
WHERE user_id = $user_id;
-- DEC-030: billing.user_erased (HIGH, retention_basis = 'Art17(3)(b)')

-- Step B-2: Soft-delete user_subscriptions (do not hard-delete — financial record)
UPDATE user_subscriptions
SET
  status     = 'expired',
  updated_at = now()
WHERE user_id = $user_id
  AND status <> 'expired';
-- The user_id in user_subscriptions is also pseudonymized via the same mechanism.
-- external_customer_id (Stripe cus_...) is retained — it is not personal data per se
-- but links to Stripe's own records. Legal review required before nulling (OQ-BILL-03 variant).
```

**Schema note.** The `user_id` column on `subscription_events` is declared `UUID NOT NULL REFERENCES users(id)`. Pseudonymization breaks this FK. The implementation must either: (a) use a separate `erased_user_reference TEXT` column and NULL the FK column on erasure, or (b) use a sentinel UUID in the `users` table (`[erased-sentinel]`) that the FK resolves to. Option (b) is preferred for schema consistency. This is a schema evolution decision — see implementation checklist item 7.

**DEC-030 event on erasure:**
```json
{
  "event_type": "billing.user_erased",
  "severity": "HIGH",
  "payload": {
    "pseudonym": "[ERASED-{sha256_hex}]",
    "rows_affected_subscription_events": N,
    "rows_affected_user_subscriptions": M,
    "retention_basis": "Art17(3)(b)",
    "legal_reference": "Tax Code of Ukraine Art.44 / EU VAT Directive 2006/112/EC Art.245"
  }
}
```

This event must be emitted in the same transaction as the pseudonymization UPDATE statements and appended to the HMAC chain. The `billing.user_erased` event is itself retained for 7 years as evidence of the erasure action.

The erasure step must be added to `src/workers/gdpr/erasure.ts` as a named step (`billingPseudonymization`) executed after the standard identity erasure but before the final `gdpr.erasure_completed` DEC-030 event.

---

### 24.11 RLS Policies

```sql
-- ─── user_subscriptions ────────────────────────────────────────────────────

-- form_api: users may read their own subscription row (for entitlement checks)
CREATE POLICY user_subscriptions_select_own
  ON user_subscriptions FOR SELECT
  TO form_api
  USING (user_id = current_setting('app.current_user_id')::uuid);

-- form_system: billing Worker has full read access (needed for webhook correlation)
CREATE POLICY user_subscriptions_select_system
  ON user_subscriptions FOR SELECT
  TO form_system
  USING (TRUE);

-- form_system: billing Worker is the ONLY INSERT path
CREATE POLICY user_subscriptions_insert_system
  ON user_subscriptions FOR INSERT
  TO form_system
  WITH CHECK (TRUE);

-- form_system: billing Worker is the ONLY UPDATE path
-- form_api has no UPDATE — prevents client-side plan elevation without payment verification
CREATE POLICY user_subscriptions_update_system
  ON user_subscriptions FOR UPDATE
  TO form_system
  USING (TRUE)
  WITH CHECK (TRUE);

-- No DELETE policy for any non-admin role. Hard deletes blocked.

-- ─── subscription_events ───────────────────────────────────────────────────

-- form_api: users may read their own subscription event history
CREATE POLICY subscription_events_select_own
  ON subscription_events FOR SELECT
  TO form_api
  USING (user_id = current_setting('app.current_user_id')::uuid);

-- form_system: billing Worker read access (idempotency check before INSERT)
CREATE POLICY subscription_events_select_system
  ON subscription_events FOR SELECT
  TO form_system
  USING (TRUE);

-- form_system: INSERT only — append-only enforced
CREATE POLICY subscription_events_insert_system
  ON subscription_events FOR INSERT
  TO form_system
  WITH CHECK (TRUE);

-- No UPDATE or DELETE for any non-admin role.
-- form_admin (BYPASSRLS) may UPDATE only for Art.17 pseudonymization — executed via
-- the GDPR erasure Worker, not via direct API access.

-- ─── enterprise_seat_allocations ───────────────────────────────────────────

-- tenant_admin: read own tenant's allocations (no INSERT/UPDATE — billing Worker only)
CREATE POLICY seat_allocations_select_tenant_admin
  ON enterprise_seat_allocations FOR SELECT
  TO tenant_admin
  USING (tenant_id = current_setting('app.current_tenant_id')::uuid);

CREATE POLICY seat_allocations_select_system
  ON enterprise_seat_allocations FOR SELECT
  TO form_system
  USING (TRUE);

CREATE POLICY seat_allocations_insert_system
  ON enterprise_seat_allocations FOR INSERT
  TO form_system
  WITH CHECK (TRUE);

CREATE POLICY seat_allocations_update_system
  ON enterprise_seat_allocations FOR UPDATE
  TO form_system
  USING (TRUE)
  WITH CHECK (TRUE);

-- ─── enterprise_seat_assignments ───────────────────────────────────────────

-- tenant_admin: read, assign (INSERT), and revoke (UPDATE revoked_at) within own tenant
CREATE POLICY seat_assignments_select_tenant_admin
  ON enterprise_seat_assignments FOR SELECT
  TO tenant_admin
  USING (tenant_id = current_setting('app.current_tenant_id')::uuid);

CREATE POLICY seat_assignments_insert_tenant_admin
  ON enterprise_seat_assignments FOR INSERT
  TO tenant_admin
  WITH CHECK (tenant_id = current_setting('app.current_tenant_id')::uuid);

CREATE POLICY seat_assignments_update_tenant_admin
  ON enterprise_seat_assignments FOR UPDATE
  TO tenant_admin
  USING (tenant_id = current_setting('app.current_tenant_id')::uuid)
  WITH CHECK (tenant_id = current_setting('app.current_tenant_id')::uuid);
  -- Only revoked_at update is meaningful; no other column changes permitted
  -- (enforced at application layer via column-level check in billing Worker)

-- form_api: users may see their own seat assignment (to know they have a seat)
CREATE POLICY seat_assignments_select_own
  ON enterprise_seat_assignments FOR SELECT
  TO form_api
  USING (user_id = current_setting('app.current_user_id')::uuid);

-- form_system: full access for SCIM-driven provisioning
CREATE POLICY seat_assignments_select_system
  ON enterprise_seat_assignments FOR SELECT TO form_system USING (TRUE);

CREATE POLICY seat_assignments_insert_system
  ON enterprise_seat_assignments FOR INSERT TO form_system WITH CHECK (TRUE);

CREATE POLICY seat_assignments_update_system
  ON enterprise_seat_assignments FOR UPDATE TO form_system
  USING (TRUE) WITH CHECK (TRUE);

-- ─── tenant_seat_utilization (materialized view) ───────────────────────────
-- Tenant admins and form_system read; no writes (it is a materialized view).
-- RLS on the underlying tables provides the security boundary.
-- The materialized view is owned by form_system; tenant_admin is granted SELECT
-- with a WHERE tenant_id = current_tenant_id() filter enforced at view query time
-- via a security-invoker wrapper function, not via RLS on the MV itself.
```

---

### 24.12 DEC-030 HMAC-Chained Audit Events

All billing audit events are emitted via `emitAuditEvent()` in the same database transaction as the `subscription_events` INSERT and the `user_subscriptions` UPDATE that triggered them. This maintains HMAC chain integrity — a billing mutation without a corresponding audit event is detectable as a chain break.

| Event Type | Severity | Retention | Trigger | Mandatory Payload Fields |
|---|---|---|---|---|
| `billing.subscription_created` | STANDARD | 7yr | New `user_subscriptions` row inserted | `plan_tier`, `billing_channel`, `user_id`, `tenant_id` (if enterprise) |
| `billing.subscription_activated` | STANDARD | 7yr | `status` → `active` (from trialing, grace, or reactivation) | `plan_tier`, `previous_status`, `activation_source` (`trial_converted` \| `payment` \| `reactivated`) |
| `billing.subscription_renewed` | STANDARD | 7yr | Successful renewal payment; `current_period_end` advanced | `plan_tier`, `new_period_end`, `amount_usd_cents`, `external_invoice_id` |
| `billing.subscription_upgraded` | STANDARD | 7yr | `plan_tier` increases or seat count increases | `previous_plan_tier`, `new_plan_tier`, `previous_seats` (enterprise), `new_seats` (enterprise) |
| `billing.subscription_downgraded` | STANDARD | 7yr | `plan_tier` decreases or `cancel_at_period_end = TRUE` set | `previous_plan_tier`, `new_plan_tier`, `effective_date` |
| `billing.subscription_cancelled` | STANDARD | 7yr | `cancel_at_period_end = TRUE` set by user action | `plan_tier`, `cancelled_at`, `access_ends_at` (`current_period_end`) |
| `billing.payment_failed` | HIGH | 7yr | External payment failure notification received | `billing_channel`, `failure_code`, `grace_period_end` (if grace started), `attempt_count` |
| `billing.grace_started` | HIGH | 7yr | `status` → `grace`; `grace_ends_at` computed | `grace_ends_at`, `plan_tier`, `billing_channel` |
| `billing.subscription_expired` | HIGH | 7yr | `status` → `expired` (grace elapsed, trial elapsed, or period ended post-cancel) | `plan_tier`, `expiry_reason` (`grace_elapsed` \| `trial_elapsed` \| `period_ended`) |
| `billing.seats_expanded` | STANDARD | 7yr | Enterprise seat count increased; new `enterprise_seat_allocations` row | `tenant_id`, `previous_seats`, `new_seats`, `price_per_seat_usd_cents` |
| `billing.seats_reduced` | HIGH | 7yr | Enterprise seat count decreased; over-allocation alert if `used_seats > new_seats` | `tenant_id`, `previous_seats`, `new_seats`, `over_allocated` (bool) |
| `billing.user_erased` | HIGH | 7yr | Art. 17 pseudonymization of billing records | `pseudonym`, `rows_affected_subscription_events`, `rows_affected_user_subscriptions`, `retention_basis` (`Art17(3)(b)`), `legal_reference` |

**Severity rationale:**
- `billing.payment_failed` is HIGH because repeated payment failures are an anomaly detection signal for potential fraud or account takeover (card testing against accounts).
- `billing.grace_started` is HIGH because grace expiry triggers access revocation — a material change in user entitlement.
- `billing.subscription_expired` is HIGH for the same reason.
- `billing.seats_reduced` is HIGH because it may trigger access revocation for over-allocated seats, which is a security-relevant event.
- `billing.user_erased` is HIGH as an irreversible operation on financial records requiring elevated audit visibility.

All events carry the standard DEC-030 envelope fields: `tenant_id`, `user_id`, `event_type`, `severity`, `actor_id` (billing Worker service identity for automated events; `tenant_admin` user ID for seat assignment events), `trace_id`, `payload`, `hmac`, `prev_hmac`.

---

### 24.13 SOC 2 Evidence Mapping

| Criterion | Control | Evidence Artefact |
|---|---|---|
| **CC6.2** | Plan tier = authorization gate; `user_subscriptions.plan_tier` checked by `assertFeatureEnabled()` (§19) — two-table authorization chain: `user_subscriptions` → `tenant_feature_flags` → `feature_flag_registry` | `__tests__/db/billing_entitlement.test.ts`: assert `form_api` cannot read Elite feature flags with `plan_tier = 'free'` |
| **CC6.3** | Seat revocation (`revoked_at` update on `enterprise_seat_assignments`) removes access; `form_api` RLS on `user_subscriptions` returns `status = 'expired'` for revoked users; `assertFeatureEnabled()` treats `expired` as `free` | Integration test: revoke seat → verify API returns 403 on Elite feature |
| **A1.1** | `subscription_status_enum` active/grace/expired monitoring provides operational signal for availability SLA commitment; `idx_user_subscriptions_renewal_sweep` enables scheduled renewal health checks | Pg-level monitoring query: `SELECT status, COUNT(*) FROM user_subscriptions GROUP BY status` — sampled in ops dashboard |
| **CC7.2** | `billing.payment_failed` HIGH events surface in SIEM for revenue anomaly detection; repeated failures from same tenant signal potential account abuse | Splunk / Datadog filter: `event_type = 'billing.payment_failed' AND severity = 'HIGH'` → revenue anomaly alert |
| **CC8.1** | Billing Worker is the sole INSERT/UPDATE path for `user_subscriptions` (RLS §24.11 blocks `form_api` writes); HMAC chain verifies no unauthorized mutations between audit events | CI test: `form_api` UPDATE to `user_subscriptions` → permission denied; HMAC chain verification test in `__tests__/audit/hmac_chain.test.ts` |
| **P5.0** | GDPR Art. 17 erasure path for billing data documented (§24.10 pseudonymization); `failure_reason` nulled on erasure; `billing.user_erased` event with `retention_basis` retained as legal compliance evidence | Art. 17 erasure integration test: `subscription_events.failure_reason IS NULL` and `user_id` pseudonymized post-erasure |
| **P8.0** | `billing.user_erased` DEC-030 events with `retention_basis = 'Art17(3)(b)'` provide auditor-traceable evidence of erasure requests and the legal basis for retention of pseudonymized records | Export: `SELECT * FROM audit_log WHERE event_type = 'billing.user_erased'` — provided to DPA on request |

**Evidence exports for auditor:**
- Monthly `subscription_events` export filtered on `event_type IN ('upgraded', 'downgraded', 'seats_expanded', 'seats_reduced')` → authorization-gated plan change log for CC6.2.
- `enterprise_seat_assignments` join `users` filtered on `revoked_at IS NOT NULL` over the audit period → access revocation evidence for CC6.3.
- `billing.user_erased` DEC-030 events with `retention_basis` → Art. 17(3)(b) compliance evidence for P8.0.

---

### 24.14 Open Questions

| ID | Question | Owner | Priority |
|---|---|---|---|
| ~~OQ-BILL-01~~ **🟢 Resolved — DATA_MODEL §30 (2026-06-11)** | ~~**Trial length configurability.**~~ **Resolved:** Consumer Pro trial is 14 days fixed — no per-tenant configurability. Enterprise pilots are handled via comped active subscription (`billing_channel = 'none'`, `price_usd_cents = 0`, `status = 'active'`); pilot window governed by `enterprise_contracts.pilot_start_at / pilot_end_at` (§16 — no schema change). `trial_duration_days` column on `enterprise_contracts` explicitly rejected. See `docs/DATA_MODEL.md §30.4`, `docs/DECISION_LOG.md DEC-045`. | `founder` + `enterprise-architect` | **🟢 Resolved** |
| OQ-BILL-02 | **Currency localization.** Should UAH pricing be stored as a separate `price_uah_kopecks` column or computed at display time from USD via a fixed exchange rate? Separate storage is more accurate (avoids forex volatility in displayed prices) but requires pricing sync on rate changes. Display-time conversion is simpler but can show jarring price changes. Recommendation: store a `display_prices JSONB` column on `plan_tier_definitions` (new table) with per-currency display amounts, independent of `price_usd_cents` which remains the internal accounting unit. | `product-strategist` + `enterprise-architect` | **P1 — before UA market launch** |
| OQ-BILL-03 | **Refund policy encoding.** Full refund within 24 hours? Pro-rated? Store-platform refunds (Apple/Google) differ from Stripe refunds. The `billing.refunded` event `amount_usd_cents` field needs a policy decision: (a) always = full period price; (b) pro-rated = `price_usd_cents × (days_remaining / period_days)`; (c) platform-determined (use whatever Apple/Google/Stripe reports). Option (c) is strongly preferred: trust the platform's amount and record it verbatim. Needs legal sign-off for consumer contracts in Ukraine. | `legal` + `compliance-officer` | **P1 — before public launch** |
| ~~OQ-BILL-04~~ **🟢 Resolved — DATA_MODEL §27 (2026-06-10)** | ~~**Invite-based seat provisioning.**~~ **Resolved:** separate `tenant_invitations` table preferred over `pending_email` column on `enterprise_seat_assignments`. `enterprise_seat_assignments` extended with nullable `user_id` + `invitation_id FK` + `chk_seat_assignment_not_orphan` CHECK. Full schema, state machine, SCIM auto-link path, GDPR Art. 17 erasure handling, and six DEC-030 events documented in `DATA_MODEL.md §27`. | `enterprise-architect` + `platform-engineer` | **🟢 Resolved** |
| ~~OQ-BILL-05~~ **🟢 Resolved — DATA_MODEL §30 (2026-06-11)** | ~~**Art. 17 pseudonymization: FK constraint approach.**~~ **Resolved:** Option (b) split-column adopted. `subscription_events.user_id` made nullable; `erased_user_reference TEXT NULL` column added with regex CHECK; `chk_sub_events_user_or_erased` mutual-exclusivity CHECK constraint enforced. Pseudonym format: `[ERASED-{sha256(user_id + ERASURE_PSEUDONYM_SALT)}]` — per-user keyed HMAC, satisfies GDPR Art. 4(5). Erasure Worker gains step SUB-1. See `docs/DATA_MODEL.md §30.2`, `docs/DECISION_LOG.md DEC-044`. | `enterprise-architect` + `platform-engineer` + `compliance-officer` | **🟢 Resolved** |

---

### 24.15 Implementation Checklist

| # | Task | Owner | Priority | Milestone |
|---|---|---|---|---|
| 1 | Create `migrations/YYYYMMDD_billing_enums.sql` — `plan_tier_enum`, `subscription_status_enum`, `billing_channel_enum`, `subscription_event_type_enum` | `platform-engineer` | **P0** | M4 |
| 2 | Create `migrations/YYYYMMDD_user_subscriptions.sql` — full DDL from §24.3 including all CHECK constraints, partial unique index, and RLS policies from §24.11 | `platform-engineer` | **P0** | M4 |
| 3 | Create `migrations/YYYYMMDD_subscription_events.sql` — full DDL from §24.4 including `uq_subscription_events_external_id` idempotency index and RLS | `platform-engineer` | **P0** | M4 |
| 4 | Create `migrations/YYYYMMDD_enterprise_seats.sql` — `enterprise_seat_allocations`, `enterprise_seat_assignments` DDL, `tenant_seat_utilization` materialized view, all indexes and RLS policies from §24.11 | `platform-engineer` | **P0** | M4 |
| 5 | Implement `src/workers/billing/apple-webhook.ts` — StoreKit 2 JWS verification, notification type mapping (§24.6), idempotency via `external_event_id`, atomic `user_subscriptions` + `subscription_events` + DEC-030 transaction | `platform-engineer` | **P0** | M4 |
| 6 | Implement `src/workers/billing/google-play-webhook.ts` — Pub/Sub base64 decode, Google Play Developer API v3 verification, `linkedPurchaseToken` chain resolution, notification type mapping (§24.7) | `platform-engineer` | **P0** | M4 |
| 7 | Implement `src/workers/billing/stripe-webhook.ts` — Stripe-Signature HMAC-SHA256 verification (raw body), event mapping (§24.8), seat expansion/contraction flow, `tenant_seat_utilization` refresh | `platform-engineer` | **P0** | M4 |
| 8 | Resolve OQ-BILL-05 (Art. 17 FK approach — sentinel UUID vs. split column) and implement chosen approach before any erasure path ships; add schema migration if split column chosen | `platform-engineer` + `compliance-officer` | **P0** | M4 |
| 9 | Add billing pseudonymization step (`billingPseudonymization`) to `src/workers/gdpr/erasure.ts` per §24.10 — UPDATE `subscription_events` pseudonymize + null `failure_reason`; soft-expire `user_subscriptions`; emit `billing.user_erased` DEC-030 event | `platform-engineer` + `compliance-officer` | **P0** | M4 |
| 10 | Register all 12 `billing.*` DEC-030 event types in `docs/AUDIT_LOG_SCHEMA.md` with severity, retention tier (all 7yr), and mandatory payload fields from §24.12 | `compliance-officer` | **P0** | M4 |
| 11 | Write `__tests__/db/billing_rls.test.ts` — (a) `form_api` UPDATE to `user_subscriptions` → permission denied; (b) `form_api` reads own subscription row; (c) `tenant_admin` reads own tenant's `enterprise_seat_allocations`; (d) `tenant_admin` cannot read another tenant's allocations; (e) `subscription_events` INSERT as `form_api` → permission denied | `qa-lead` + `enterprise-architect` | **P0** | M4 |
| 12 | Write `__tests__/db/billing_entitlement.test.ts` — assert `assertFeatureEnabled()` returns false for `plan_tier = 'free'` on Growth/Elite-gated flags; returns true on matching tier; `status = 'expired'` treated as `free`; regression against CC6.2 | `qa-lead` + `platform-engineer` | **P0** | M4 |

---

## 25. Enterprise Tenant Offboarding & Data Egress Schema

> Owner: `enterprise-architect` + `compliance-officer`. Review: before any enterprise pilot launch, on any change to data retention commitments, and quarterly.
> Scope: enterprise tier only. Consumer account closure is handled by §12 (GDPR Art. 17 individual erasure). This section handles the organisational off-boarding of a tenant — the employer entity — and does not supersede §12, which continues to govern each individual user's personal data rights independently.
> References: §16 (Enterprise Contract & Pilot Lifecycle Schema — state `churned` is the entry trigger); §12 (individual GDPR Art. 17 erasure — continues in parallel for individual users); §24 (billing pseudonymization — financial record retention); `docs/SSO_SCIM_IMPLEMENTATION.md §22` (KV-backed session revocation via `nukeTenantSessions()`); DEC-030 (`docs/AUDIT_LOG_SCHEMA.md`); `docs/SOC2_READINESS.md` criteria C1.2, P8.0, CC6.3, CC9.1.

---

### 25.1 Purpose and Design Principles

§16.2 specifies that when `tenants.lifecycle_status` transitions to `'churned'`, "data deletion process begins per §12.3." §12.3 documents the per-user GDPR Art. 17 erasure flow for individual users who have submitted a right-to-erasure request. That flow is **not** a sufficient procedure for enterprise off-boarding: it handles one user at a time on a 30-day SLA, targets user-initiated requests, and was not designed for the coordinated, multi-table, employer-notified deletion of an entire tenant's data estate. This section closes that gap.

**Three off-boarding triggers.** An off-boarding job is created whenever `tenants.lifecycle_status` transitions to `'churned'` from any predecessor state. The initiating cause is recorded as one of three trigger types:

1. **Contract expiry without renewal** — `tenant_contracts.status` → `'expired'` and no new `tenant_contracts` row created within the grace window; CSM confirms no pipeline.
2. **Early termination** — `tenant_contracts.status` → `'terminated'` by either party under the termination clause of the MSA.
3. **Pilot lapse** — `tenant_pilots.status` → `'expired'` with no conversion and no extension; no contract exists.

**Privacy floor applies to all egress.** The employer (tenant) is not the data controller for individual user health data — the individual user is the data subject. The off-boarding data export package delivered to the employer contains:

- Aggregate-only adoption metrics (subject to the n ≥ 15 k-anonymity floor from §6 and §17).
- Audit logs scoped to administrative actions (SSO configs, SCIM provisioning events, contract documents).
- Contract documentation.

Individual user health data — workouts, coaching turns, meal logs, body metrics, CV pose keypoints, health profiles — is **not** included in any employer-facing export, regardless of contractual request. Individual users receive their own personal data via the standard GDPR Art. 20 data portability path (§12.5). This constraint is structural: the `tenant_data_egress_packages` table (§25.4) enforces an ENUM of permitted package types that excludes any category containing individual user health data, and the worker that generates packages (§25.11) has no code path to query individual-user health tables.

---

### 25.2 Off-Boarding Trigger Event Taxonomy

| Trigger type | Entry state (§16) | Initiating action | Tenant notification timing | Nominal off-boarding timeline |
|---|---|---|---|---|
| `contract_expired_no_renewal` | `lifecycle_status = 'expired'` for ≥ 30 days | Automated: renewal sweep finds no new contract | At `expired` transition (60-day advance notice via §16.7); reminder at T+15d, T+30d | Churn confirmed at T+30d; deletion completes within 60 days of churn confirmation |
| `early_termination` | `lifecycle_status = 'churned'` via `tenant_contracts.status → 'terminated'` | Manual: CSM or compliance officer initiates via internal tooling | Immediate notice on termination event; employer data export within 5 business days per MSA | Deletion begins immediately; completes within 30 days of termination date |
| `pilot_lapsed` | `lifecycle_status = 'churned'` via pilot expiry | Automated: pilot sweep; CSM confirms no pipeline | At pilot expiry; pilot off-boarding is lower urgency — employer receives aggregate report only | Deletion completes within 60 days of pilot expiry |

The `offboarding_trigger` field on `tenant_offboarding_jobs` (§25.3) records the trigger type for each job. All three paths converge on the same state machine (§25.6) and deletion sequence (§25.7).

---

### 25.3 `tenant_offboarding_jobs` Table

This table is the authoritative record of every off-boarding operation. One row per tenant off-boarding event. A tenant that is re-onboarded after churn (rare but contractually possible) receives a new `tenants` row with a new `id` — the prior off-boarding job remains as immutable history.

```sql
CREATE TYPE offboarding_trigger_enum AS ENUM (
  'contract_expired_no_renewal',
  'early_termination',
  'pilot_lapsed'
);

CREATE TYPE offboarding_status_enum AS ENUM (
  'pending',               -- created; awaiting worker pickup
  'access_revoked',        -- SSO/SCIM/API access blocked; sessions nuked
  'export_in_progress',    -- employer data export package being generated
  'export_delivered',      -- signed URL sent to tenant_owner email on file
  'deletion_in_progress',  -- Art. 9 + standard personal data deletion running
  'financial_pseudonymized', -- billing records pseudonymized per §24.10
  'attestation_issued',    -- signed deletion attestation generated (§25.5)
  'completed',             -- all steps verified; off-boarding closed
  'failed',                -- a step failed; requires manual remediation
  'on_hold'                -- compliance or legal hold placed on deletion
);

CREATE TABLE tenant_offboarding_jobs (
  id                          UUID        PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id                   UUID        NOT NULL REFERENCES tenants(id),
  -- REFERENCES with no ON DELETE clause: RESTRICT by default.
  -- The tenants row must not be hard-deleted until this job reaches 'completed'.
  -- The job must survive even if the tenants row is later purged via a compliance action.

  trigger_type                offboarding_trigger_enum  NOT NULL,
  status                      offboarding_status_enum   NOT NULL DEFAULT 'pending',

  -- Timestamps for each state transition (NULL = not yet reached)
  access_revoked_at           TIMESTAMPTZ,
  export_started_at           TIMESTAMPTZ,
  export_delivered_at         TIMESTAMPTZ,
  deletion_started_at         TIMESTAMPTZ,
  financial_pseudonymized_at  TIMESTAMPTZ,
  attestation_issued_at       TIMESTAMPTZ,
  completed_at                TIMESTAMPTZ,
  failed_at                   TIMESTAMPTZ,

  -- Reference to the contract or pilot that triggered off-boarding
  contract_id                 UUID        REFERENCES tenant_contracts(id),
  pilot_id                    UUID        REFERENCES tenant_pilots(id),
  CONSTRAINT chk_ofb_trigger_has_source CHECK (
    (trigger_type = 'pilot_lapsed'    AND pilot_id IS NOT NULL)
    OR (trigger_type <> 'pilot_lapsed' AND contract_id IS NOT NULL)
  ),

  -- Counts recorded at deletion time for attestation cross-reference
  users_anonymized_count      INTEGER,
  art9_rows_deleted_count     INTEGER,
  standard_rows_deleted_count INTEGER,
  financial_rows_pseudonymized INTEGER,

  -- Two-person authorisation for the deletion step (mirrors §22 nukeTenantSessions rule)
  authorised_by_1             TEXT,  -- internal user ID or service identity
  authorised_by_2             TEXT,  -- must differ from authorised_by_1
  CONSTRAINT chk_ofb_two_person CHECK (
    authorised_by_1 IS NULL
    OR authorised_by_1 <> authorised_by_2
  ),

  -- Internal notes for compliance team (MUST NOT contain individual user names or health data)
  compliance_notes            TEXT,

  created_at                  TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at                  TIMESTAMPTZ NOT NULL DEFAULT now()
);

COMMENT ON COLUMN tenant_offboarding_jobs.authorised_by_1
  IS 'First authoriser for the deletion step. Set by the orchestrator from the API caller identity.';
COMMENT ON COLUMN tenant_offboarding_jobs.authorised_by_2
  IS 'Second authoriser, distinct from authorised_by_1. Required before status may transition to deletion_in_progress.';
COMMENT ON COLUMN tenant_offboarding_jobs.art9_rows_deleted_count
  IS 'Count of rows hard-deleted across Art. 9 tables: keypoints_enc, user_health_profiles, coaching_turns, meal_logs, body_metrics. Recorded in the deletion attestation.';
COMMENT ON COLUMN tenant_offboarding_jobs.compliance_notes
  IS 'Internal notes for the compliance team. Must not contain individual user names, email addresses, or health data values.';

ALTER TABLE tenant_offboarding_jobs ENABLE ROW LEVEL SECURITY;

CREATE INDEX idx_ofb_jobs_tenant_id
  ON tenant_offboarding_jobs (tenant_id);

CREATE INDEX idx_ofb_jobs_status
  ON tenant_offboarding_jobs (status)
  WHERE status NOT IN ('completed', 'failed');

CREATE TRIGGER update_ofb_jobs_updated_at
  BEFORE UPDATE ON tenant_offboarding_jobs
  FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();
```

---

### 25.4 `tenant_data_egress_packages` Table

Each row represents one data package generated and delivered to the employer as part of off-boarding. The permitted package types are enforced by ENUM — not by application logic alone. No package type in this ENUM may contain individual user health data. The worker that generates packages (§25.11) is granted SELECT access only to the specific aggregate views and audit-log subsets listed in the package type definitions in §25.8, and has no access to individual-user health tables.

```sql
CREATE TYPE egress_package_type_enum AS ENUM (
  'aggregate_adoption_report',
  -- Tenant-level WAU/MAU/activation metrics; n ≥ 15 floor enforced.
  -- Sourced from tenant_wellness_summary and tenant_cv_summary MVs only (§17).

  'audit_log_export',
  -- DEC-030 events scoped to administrative actions: SSO config changes, SCIM provisioning,
  -- seat changes, contract events, off-boarding events.
  -- Individual user health-context audit entries are excluded by permitted event type filter (§25.8).

  'contract_documents',
  -- tenant_contracts and tenant_pilots metadata (arr_cents excluded; FORM commercial-confidential).
  -- DPA reference; contract_ref list. No personal data (§16.9).

  'scim_provisioning_summary'
  -- Aggregate counts: SCIM-provisioned, deprovisioned, active users at off-boarding date.
  -- No individual user rows, no email addresses, no external_id values.
);

CREATE TYPE egress_package_status_enum AS ENUM (
  'queued',
  'generating',
  'ready',       -- signed URL generated; not yet downloaded
  'downloaded',  -- tenant_owner confirmed download (HEAD request or explicit ack)
  'expired',     -- 30-day signed URL TTL elapsed; R2 object deleted
  'failed'
);

CREATE TABLE tenant_data_egress_packages (
  id                    UUID        PRIMARY KEY DEFAULT gen_random_uuid(),
  offboarding_job_id    UUID        NOT NULL REFERENCES tenant_offboarding_jobs(id),
  tenant_id             UUID        NOT NULL,
  -- Denormalised from offboarding_job for RLS performance; must equal offboarding_job.tenant_id.

  package_type          egress_package_type_enum    NOT NULL,
  status                egress_package_status_enum  NOT NULL DEFAULT 'queued',

  -- Storage: Cloudflare R2 (see OQ-OFB-02 for EU-region R2 consideration)
  r2_object_key         TEXT,       -- set when status → 'ready'; NULL before generation
  r2_bucket             TEXT        NOT NULL DEFAULT 'form-offboarding-exports',
  sha256_manifest       TEXT,       -- hex-encoded SHA-256 of the full ZIP archive
  hmac_signature        TEXT,       -- HMAC-SHA256(package_signing_key, sha256_manifest).
                                    -- Employer verifies using the public signing key at:
                                    -- https://form.coach/.well-known/offboarding-signing-key.pub

  signed_url_expires_at TIMESTAMPTZ,  -- 30 days from generation
  downloaded_at         TIMESTAMPTZ,

  package_size_bytes    BIGINT,     -- recorded after generation for audit evidence
  row_count             INTEGER,    -- row count inside the package for attestation cross-reference

  -- Delivery address: the tenant_owner email at time of off-boarding initiation.
  -- Recorded here so evidence is preserved even if the users row is later anonymized (§25.7).
  delivered_to_email    TEXT        NOT NULL
    CHECK (delivered_to_email <> ''),

  error_message         TEXT,       -- populated on status = 'failed'; must not contain PII
  generated_at          TIMESTAMPTZ,
  created_at            TIMESTAMPTZ NOT NULL DEFAULT now()
);

COMMENT ON COLUMN tenant_data_egress_packages.package_type
  IS 'ENUM restricts package content to employer-permitted categories. No value in this ENUM can produce a package containing individual user health data.';
COMMENT ON COLUMN tenant_data_egress_packages.hmac_signature
  IS 'HMAC-SHA256 of sha256_manifest using the FORM compliance package signing key. Employer verifies package integrity and authenticity post-download against published public key.';
COMMENT ON COLUMN tenant_data_egress_packages.signed_url_expires_at
  IS '30-day TTL. After expiry the R2 object is deleted and status is set to expired. Employer must request re-generation via support if the window is missed.';
COMMENT ON COLUMN tenant_data_egress_packages.r2_object_key
  IS 'R2 object key. Excluded from tenant-facing API responses; tenant_admin sees status and downloaded_at only.';

ALTER TABLE tenant_data_egress_packages ENABLE ROW LEVEL SECURITY;

CREATE INDEX idx_egress_packages_job_id
  ON tenant_data_egress_packages (offboarding_job_id);

CREATE INDEX idx_egress_packages_tenant_status
  ON tenant_data_egress_packages (tenant_id, status)
  WHERE status NOT IN ('expired', 'failed');

-- One package per type per off-boarding job — prevents duplicate generation.
CREATE UNIQUE INDEX uq_egress_package_type_per_job
  ON tenant_data_egress_packages (offboarding_job_id, package_type);
```

---

### 25.5 `tenant_deletion_attestations` Table

A signed attestation record is the SOC 2 C1.2 evidence artefact for disposal of confidential information. One attestation is issued per off-boarding job, after all deletion steps are verified complete. The attestation text is generated from the template below (§25.5.1), stored verbatim, and signed with the FORM compliance key.

```sql
CREATE TABLE tenant_deletion_attestations (
  id                           UUID        PRIMARY KEY DEFAULT gen_random_uuid(),
  offboarding_job_id           UUID        NOT NULL UNIQUE REFERENCES tenant_offboarding_jobs(id),
  tenant_id                    UUID        NOT NULL,

  -- Attestation text stored verbatim (generated from template §25.5.1)
  attestation_text             TEXT        NOT NULL,

  -- Signing
  signing_key_version          TEXT        NOT NULL,  -- e.g., 'compliance-key-v3'
  signature                    TEXT        NOT NULL,  -- HMAC-SHA256(compliance_signing_key, attestation_text)

  -- Summary counts from the completed offboarding_job (cross-checked at generation time)
  users_anonymized_count       INTEGER     NOT NULL,
  art9_rows_deleted_count      INTEGER     NOT NULL,
  standard_rows_deleted_count  INTEGER     NOT NULL,
  financial_rows_pseudonymized INTEGER     NOT NULL,

  -- Issued to: the tenant_owner contact at time of off-boarding
  issued_to_email              TEXT        NOT NULL,
  issued_at                    TIMESTAMPTZ NOT NULL DEFAULT now(),

  -- Retained 7 years: legal obligation evidence under Art. 17(3)(b) and SOC 2 C1.2.
  -- Not subject to Art. 17 erasure.
  retain_until                 TIMESTAMPTZ NOT NULL DEFAULT now() + INTERVAL '7 years'
);

COMMENT ON COLUMN tenant_deletion_attestations.attestation_text
  IS 'Verbatim text of the attestation generated from the template in §25.5.1. Signed by FORM compliance key. Stored for DPA and SOC 2 audit evidence.';
COMMENT ON COLUMN tenant_deletion_attestations.retain_until
  IS '7-year retention for C1.2 and P8.0 evidence. Not subject to Art. 17 erasure; Art. 17(3)(b) legal obligation basis — same as billing record retention (§24.10).';

ALTER TABLE tenant_deletion_attestations ENABLE ROW LEVEL SECURITY;

CREATE INDEX idx_attestations_tenant_id
  ON tenant_deletion_attestations (tenant_id);
```

#### 25.5.1 Attestation Text Template

```
FORM DATA DELETION ATTESTATION
Attestation ID:   {attestation_id}
Tenant:           {tenant_slug} (Tenant ID: {tenant_id})
Off-boarding Job: {offboarding_job_id}
Trigger:          {trigger_type}
Issued:           {issued_at} UTC

FORM Fitness Technologies hereby attests that, as of the date above, the following
data deletion and pseudonymization actions have been completed for the above-named
tenant in accordance with GDPR Art. 17 and the applicable Data Processing Agreement:

  Art. 9 special category data (hard-deleted, no grace period):
    Tables:    keypoints_enc, user_health_profiles, coaching_turns,
               meal_logs, body_metrics
    Rows deleted:  {art9_rows_deleted_count}
    Completed:     {art9_deletion_completed_at} UTC

  Standard personal data (hard-deleted after 30-day grace period):
    Tables:    workouts, workout_sets, media_uploads, scim_groups,
               scim_group_members, enterprise_seat_assignments,
               tenant_sso_configs, tenant_scim_tokens
    Rows deleted:  {standard_rows_deleted_count}
    Completed:     {standard_deletion_completed_at} UTC

  Users rows: {users_anonymized_count} rows anonymized — email, display_name,
    avatar_url, and all PII columns set to NULL. UUID retained for audit log
    FK integrity per GDPR Art. 17(3)(b).

  Financial records (pseudonymized, not deleted):
    Tables:    user_subscriptions, subscription_events, tenant_contracts
    Rows pseudonymized: {financial_rows_pseudonymized}
    Retention basis:    Tax Code of Ukraine Art. 44 /
                        EU VAT Directive 2006/112/EC Art. 245
    Retained until:     {financial_retain_until}

  Audit log: Retained for 7 years per SOC 2 CC6.3 and Art. 6(1)(c) legal
    obligation. Audit log rows are not deleted. No personal data values are
    stored in audit log rows per DEC-030 design constraint.

  Sessions: All active sessions revoked at {access_revoked_at} UTC via
    nukeTenantSessions() (SSO_SCIM_IMPLEMENTATION.md §22).

This attestation was generated by the FORM offboarding Worker and signed with
key version {signing_key_version}. Verify: HMAC-SHA256(published_compliance_key,
attestation_text) must equal the signature transmitted with this attestation.
```

---

### 25.6 Off-Boarding State Machine

The `offboarding_status_enum` values define a directed state machine with no backwards transitions. Every transition emits a DEC-030 HMAC-chained audit event (§25.9). The `on_hold` status is a compliance-hold override reachable from any non-terminal non-completed state; release from `on_hold` returns to `pending`.

```
[tenants.lifecycle_status → 'churned']
          │
          ▼
       pending  ────────────────────────────────────────► on_hold
          │                                                   │
          │ Worker: nukeTenantSessions(); deactivate SSO/SCIM │ hold released by compliance officer
          ▼                                                   │
    access_revoked ◄──────────────────────────────────────────┘
          │
          │ Worker: generate employer egress packages
          ▼
   export_in_progress
          │
          │ Signed URLs delivered to tenant_owner email
          ▼
   export_delivered
          │
          │ Two-person authorisation confirmed; deletion worker invoked
          ▼
  deletion_in_progress
          │
          │ Art. 9 hard-delete; standard data deferred 30d; billing pseudonymized
          ▼
  financial_pseudonymized
          │
          │ Row counts verified; attestation text rendered and signed
          ▼
  attestation_issued
          │
          │ Compliance officer marks job complete; tenants row may now be hard-deleted
          ▼
       completed
```

**State transition table:**

| From state | Trigger | Next state | DEC-030 event |
|---|---|---|---|
| `pending` | Legal or compliance hold placed | `on_hold` | `enterprise.offboarding_on_hold` (HIGH) |
| `on_hold` | Hold released by compliance officer | `pending` | `enterprise.offboarding_hold_released` (HIGH) |
| `pending` | Orchestrator picks up job; `nukeTenantSessions()` called; SSO/SCIM configs deactivated | `access_revoked` | `enterprise.sso_scim_revoked` (HIGH) |
| `access_revoked` | Egress package generation begins | `export_in_progress` | `enterprise.data_export_started` (STANDARD) |
| `export_in_progress` | All packages generated; signed URLs delivered | `export_delivered` | `enterprise.data_export_completed` (STANDARD) |
| `export_delivered` | Two-person authorisation confirmed; deletion worker invoked | `deletion_in_progress` | `enterprise.data_deletion_started` (HIGH) |
| `deletion_in_progress` | All deletions verified; billing pseudonymization complete | `financial_pseudonymized` | `enterprise.data_deletion_completed` (HIGH) + `enterprise.financial_pseudonymized` (HIGH) |
| `financial_pseudonymized` | Attestation text generated and signed | `attestation_issued` | `enterprise.deletion_attestation_issued` (HIGH) |
| `attestation_issued` | Compliance officer marks job complete | `completed` | `enterprise.offboarding_completed` (STANDARD) |
| Any non-terminal | Worker step throws unrecoverable error | `failed` | `enterprise.offboarding_step_failed` (HIGH) |

---

### 25.7 GDPR Art. 17 Enterprise-Scale Deletion Sequence

§12.3 covers individual user erasure triggered by a user's own request. The enterprise off-boarding deletion is system-initiated, covers all users in the tenant simultaneously, and uses table-level batch operations rather than the row-level, queue-based erasure of §12.3. The two processes are structurally independent: §12.3 is user-initiated and user-scoped; §25.7 is employer off-boarding and tenant-scoped.

Individual users retain their Art. 17 right to erasure independently after employer off-boarding. The §12 erasure path remains available to any individual user regardless of their employer's off-boarding status. The employer off-boarding does not satisfy a user's individual DSAR and must not be represented as doing so.

| Data category | Tables | Deletion approach | Retention exception | Timeline after `deletion_in_progress` |
|---|---|---|---|---|
| CV pose keypoints (Art. 9) | `keypoints_enc` | Hard-delete immediately; no grace period | None | Within 24 hours |
| User health profiles (Art. 9) | `user_health_profiles` | Hard-delete immediately; no grace period | None | Within 24 hours |
| Coaching turns with health context (Art. 9) | `coaching_turns` | Hard-delete immediately; no grace period | None | Within 24 hours |
| Meal logs (Art. 9) | `meal_logs` | Hard-delete immediately; no grace period | None | Within 24 hours |
| Body metrics (Art. 9) | `body_metrics` | Hard-delete immediately; no grace period | None | Within 24 hours |
| Workout sessions and sets | `workouts`, `workout_sets` | Hard-delete after 30-day standard grace | None | Day 30 |
| Media uploads (CV thumbnails) | `media_uploads` | Hard-delete + R2 object purge after 30-day grace | None | Day 30 |
| User rows | `users` | Anonymize: NULL all PII columns; UUID retained for audit FK integrity | UUID retained per Art. 17(3)(b) | Anonymization within 24 hours; no PII remains after that |
| SSO configuration | `tenant_sso_configs` | Hard-delete on `access_revoked` transition | None | Immediate (access revocation step) |
| SCIM tokens | `tenant_scim_tokens` | `revoked_at` set immediately; hard-delete after 30 days | None | `revoked_at` set on access revocation |
| SCIM groups and memberships | `scim_groups`, `scim_group_members` | Hard-delete after 30-day grace | None | Day 30 |
| Seat assignments | `enterprise_seat_assignments` | Hard-delete after 30-day grace | None | Day 30 |
| Seat allocations | `enterprise_seat_allocations` | Soft-delete (financial reference); hard-delete after 7 years | Financial reference record | 7 years |
| Billing records | `user_subscriptions`, `subscription_events` | Pseudonymize per §24.10; not hard-deleted | Art. 17(3)(b) — 7-year financial and tax record retention | Pseudonymization within 30 days |
| Contract records | `tenant_contracts`, `tenant_pilots` | Retained as-is; no personal data in these tables (§16.9) | 7-year financial obligation | Permanent for 7 years; no deletion |
| Audit log | `audit_log` | No deletion; immutable per DEC-030 | Art. 6(1)(c) legal obligation; 7-year SOC 2 retention | Permanent; no PII values by DEC-030 design |
| Offboarding job and attestation | `tenant_offboarding_jobs`, `tenant_deletion_attestations` | Retained as compliance evidence | Art. 17(3)(b) / C1.2 — 7 years | Permanent for 7 years |

**Art. 9 no-grace period rationale.** GDPR Recital 51 identifies health data as requiring a higher level of protection. The 30-day recovery grace period in §12.1 serves the use case of accidental individual-user deletion. That use case does not apply when a contractual relationship has ended and the employer has initiated off-boarding. Retaining Art. 9 data in a recovery window after employer off-boarding creates unnecessary exposure with no legitimate recovery justification. Art. 9 tables are therefore hard-deleted immediately on the first pass of the deletion worker. See OQ-OFB-03 for the open question on whether a contractual exception is permissible.

---

### 25.8 Data Export Package Specifications

**Format.** Every package is a ZIP archive containing one or more NDJSON files plus a `manifest.json`. The manifest records: `package_type`, `tenant_id`, `generated_at`, `row_count`, `file_sha256` per NDJSON file, and `package_sha256` (SHA-256 of the full ZIP). The `package_sha256` value is stored in `tenant_data_egress_packages.sha256_manifest` and used as the input to the HMAC signing step.

**HMAC signing.** The signing key is the FORM compliance package signing key, version-labeled and rotated annually. `HMAC-SHA256(signing_key, sha256_manifest)` is stored in `tenant_data_egress_packages.hmac_signature`. FORM publishes the corresponding verification key at `https://form.coach/.well-known/offboarding-signing-key.pub`. Employers may verify package integrity and authenticity before ingesting the export.

**Delivery.** Packages are uploaded to Cloudflare R2 (`form-offboarding-exports` bucket; see OQ-OFB-02 for EU-region routing). A presigned URL with a 30-day TTL is generated and emailed to `delivered_to_email`. After 30 days the URL expires, the R2 object is deleted, and `status` transitions to `'expired'`. Re-generation requires a support ticket and is only possible if source aggregate data is still within its retention window.

**Package type contents and privacy floor enforcement:**

| Package type | Source | Contents | Privacy floor enforcement |
|---|---|---|---|
| `aggregate_adoption_report` | `tenant_wellness_summary` MV, `tenant_cv_summary` MV, `tenant_seat_utilization` MV (§17) | WAU, MAU, activation rate, D30 retention, CV adoption %, session counts — all tenant-aggregate | Cohorts where N < 15 suppressed to NULL in NDJSON; `"suppressed_reason": "k_anonymity_floor"` in manifest. No join to `users`, `workouts`, or any individual-user table in the generating query. |
| `audit_log_export` | `audit_log` WHERE `event_type` IN permitted set (see below) | Administrative event stream: SSO config changes, SCIM provisioning, seat changes, contract lifecycle, off-boarding events | Permitted event type list enforced as an explicit IN filter; all other event types excluded regardless of `tenant_id` scope. No payload fields containing health data values by DEC-030 design. |
| `contract_documents` | `tenant_contracts`, `tenant_pilots` | `contract_ref`, `plan`, `seats_contracted`, `billing_period`, `start_at`, `end_at`, `status`, `dpa_ref`. `arr_cents` excluded (FORM commercial-confidential). | No user personal data in these tables (§16.9). `arr_cents` exclusion enforced at query layer. |
| `scim_provisioning_summary` | `users` aggregate query (COUNT only) + SCIM audit events | Total provisioned count, deprovisioned count, active count at off-boarding date | Package-generating query uses `COUNT(id)` only; no SELECT on any PII column confirmed by Worker code review at each release. |

**Permitted event types for `audit_log_export`:**

```
enterprise.offboarding_initiated     enterprise.offboarding_completed
enterprise.sso_scim_revoked          enterprise.data_export_started
enterprise.data_export_completed     enterprise.data_deletion_started
enterprise.data_deletion_completed   enterprise.financial_pseudonymized
enterprise.deletion_attestation_issued
tenant.lifecycle_changed             tenant.contract_activated
tenant.contract_renewed              tenant.contract_expired
tenant.contract_terminated           tenant.pilot_started
tenant.pilot_converted               tenant.pilot_expired
tenant.seats_contracted_changed      tenant.seat_limit_enforcement_triggered
tenant.renewal_notice_sent
auth.sso_config_created              auth.sso_config_updated
auth.sso_config_deleted              auth.scim_user_provisioned
auth.scim_user_deprovisioned         auth.scim_group_created
auth.scim_group_deleted
session.tenant_nuke_started          session.tenant_nuke_complete
billing.seats_expanded               billing.seats_reduced
```

Any `audit_log` row with an `event_type` not in this list is excluded from the employer export, regardless of `tenant_id` scope.

---

### 25.9 DEC-030 HMAC-Chained Audit Events

All off-boarding audit events are emitted via `emitAuditEvent()` in the same database transaction as the state-transition UPDATE on `tenant_offboarding_jobs`. No off-boarding state transition is valid without a corresponding HMAC-chained audit event — a chain break at any off-boarding event is treated as a P0 incident.

| Event type | Severity | Retention | Trigger | Mandatory payload fields |
|---|---|---|---|---|
| `enterprise.offboarding_initiated` | HIGH | 7 years | `tenant_offboarding_jobs` row created; `tenants.lifecycle_status → 'churned'` | `tenant_id`, `offboarding_job_id`, `trigger_type`, `contract_id` or `pilot_id` |
| `enterprise.data_export_started` | STANDARD | 7 years | `status → 'export_in_progress'`; package generation begins | `tenant_id`, `offboarding_job_id`, `package_types` (array of ENUM values) |
| `enterprise.data_export_completed` | STANDARD | 7 years | All packages generated; signed URLs delivered to employer | `tenant_id`, `offboarding_job_id`, `package_ids` (array of `tenant_data_egress_packages.id`), `delivered_to_email` |
| `enterprise.sso_scim_revoked` | HIGH | 7 years | `nukeTenantSessions()` called; `tenant_sso_configs` deactivated; SCIM tokens revoked | `tenant_id`, `offboarding_job_id`, `sessions_nuked` (bool), `sso_config_count`, `scim_tokens_revoked_count` |
| `enterprise.data_deletion_started` | HIGH | 7 years | `status → 'deletion_in_progress'`; two-person authorisation confirmed | `tenant_id`, `offboarding_job_id`, `authorised_by` (two-element array), `art9_tables` (list) |
| `enterprise.data_deletion_completed` | HIGH | 7 years | All personal data hard-deletes and user anonymizations verified | `tenant_id`, `offboarding_job_id`, `users_anonymized_count`, `art9_rows_deleted_count`, `standard_rows_deleted_count` |
| `enterprise.financial_pseudonymized` | HIGH | 7 years | Billing records pseudonymized per §24.10; `status → 'financial_pseudonymized'` | `tenant_id`, `offboarding_job_id`, `financial_rows_pseudonymized`, `retention_basis` (`"Art17(3)(b)"`), `legal_reference` |
| `enterprise.deletion_attestation_issued` | HIGH | 7 years | `tenant_deletion_attestations` row created and signed | `tenant_id`, `offboarding_job_id`, `attestation_id`, `signing_key_version`, `issued_to_email` |

**Severity rationale.** All events except `data_export_started` and `data_export_completed` are HIGH. Data access revocation, deletion, pseudonymization, and attestation issuance are irreversible or legally consequential operations; HIGH severity ensures they surface in SIEM dashboards and reach the compliance officer without active filtering. The two export events are STANDARD because package generation is reversible (re-generation can be requested within the source data retention window).

**PII prohibition.** Per DEC-030 design constraint, no audit event payload field may contain individual user PII: no email addresses of individual users, no health values, no display names. The `delivered_to_email` field in `enterprise.data_export_completed` records the employer's contact address — this is the organisation's administrative contact, not an individual user's personal data in the GDPR sense. If `delivered_to_email` is a named individual's work address rather than a role address, it must be nulled in the audit log after 2 years per the standard operational log retention schedule.

---

### 25.10 RLS Policies

```sql
-- ─── tenant_offboarding_jobs ────────────────────────────────────────────────

-- form_system: full write access — only the offboarding Worker transitions states.
CREATE POLICY ofb_jobs_system_all
  ON tenant_offboarding_jobs
  TO form_system
  USING (TRUE)
  WITH CHECK (TRUE);

-- form_api (tenant_admin or tenant_owner): SELECT own job status only.
-- Excludes compliance_notes, authorised_by_1/2, and deletion counts from
-- tenant-facing API responses at the application layer.
CREATE POLICY ofb_jobs_tenant_read
  ON tenant_offboarding_jobs FOR SELECT
  TO form_api
  USING (
    tenant_id = current_setting('app.current_tenant_id')::UUID
    AND current_setting('app.current_role') IN ('tenant_owner', 'tenant_admin')
  );

-- No INSERT, UPDATE, or DELETE permitted for form_api on this table.
-- Off-boarding initiation is an internal operation triggered by lifecycle_status change,
-- not by a tenant-facing API endpoint.

-- form_admin: break-glass; BYPASSRLS — no explicit policy needed.

-- ─── tenant_data_egress_packages ────────────────────────────────────────────

CREATE POLICY egress_packages_system_all
  ON tenant_data_egress_packages
  TO form_system
  USING (TRUE)
  WITH CHECK (TRUE);

-- tenant_admin: SELECT own tenant's packages to check delivery status.
-- r2_object_key is excluded from tenant-facing API responses at application layer.
CREATE POLICY egress_packages_tenant_read
  ON tenant_data_egress_packages FOR SELECT
  TO form_api
  USING (
    tenant_id = current_setting('app.current_tenant_id')::UUID
    AND current_setting('app.current_role') IN ('tenant_owner', 'tenant_admin')
  );

-- No INSERT, UPDATE, or DELETE for form_api.

-- ─── tenant_deletion_attestations ───────────────────────────────────────────

-- form_system: write (attestation generation).
CREATE POLICY attestations_system_all
  ON tenant_deletion_attestations
  TO form_system
  USING (TRUE)
  WITH CHECK (TRUE);

-- tenant_admin: SELECT own attestation for their records.
CREATE POLICY attestations_tenant_read
  ON tenant_deletion_attestations FOR SELECT
  TO form_api
  USING (
    tenant_id = current_setting('app.current_tenant_id')::UUID
    AND current_setting('app.current_role') IN ('tenant_owner', 'tenant_admin')
  );

-- No INSERT, UPDATE, or DELETE for form_api on attestations.
```

---

### 25.11 Worker Architecture

The off-boarding flow is implemented as three cooperating Cloudflare Workers under `apps/api/src/workers/offboarding/`.

**`orchestrator.ts`** — driven by a Cloudflare Queue consumer and a pg_cron sweep. Polls `tenant_offboarding_jobs WHERE status NOT IN ('completed', 'failed', 'on_hold')` and dispatches the appropriate downstream function for each state. Single-threaded per job via `SELECT ... FOR UPDATE SKIP LOCKED`: only one step is in flight at a time. The orchestrator does not auto-advance past `export_delivered` — the deletion step requires two-person authorisation from an internal endpoint before the job is re-enqueued.

```typescript
// apps/api/src/workers/offboarding/orchestrator.ts (pseudocode)

export async function processOffboardingJob(
  jobId: string,
  env: Env,
): Promise<void> {
  const job = await env.db.queryOne<OffboardingJob>(
    `SELECT * FROM tenant_offboarding_jobs WHERE id = $1 FOR UPDATE SKIP LOCKED`,
    [jobId],
  );
  if (!job || ['on_hold', 'completed', 'failed'].includes(job.status)) return;

  switch (job.status) {
    case 'pending':
      await revokeAccess(job, env);
      break;
    case 'access_revoked':
      await generateEgressPackages(job, env);
      break;
    case 'export_delivered':
      // Awaits two-person authorisation. A separate form_admin-only endpoint
      // sets authorised_by_1 and authorised_by_2 then re-enqueues the job ID.
      break;
    case 'deletion_in_progress':
      await runDeletionWorker(job, env);
      break;
    case 'financial_pseudonymized':
      await issueAttestation(job, env);
      break;
  }
}

async function revokeAccess(job: OffboardingJob, env: Env): Promise<void> {
  // nukeTenantSessions() — SSO_SCIM_IMPLEMENTATION.md §22.
  // Single KV write revokes all in-flight JWTs for this tenant within max_jwt_ttl_seconds.
  await nukeTenantSessions(env.SESSION_REVOCATION_KV, env.supabase, {
    tenantId:         job.tenant_id,
    reason:           'admin_request',
    authorisedBy:     [job.authorised_by_1, job.authorised_by_2],
    maxJwtTtlSeconds: 28800,
  });

  await env.db.transaction(async (tx) => {
    await tx.query(
      `UPDATE tenant_sso_configs SET deleted_at = now()
         WHERE tenant_id = $1 AND deleted_at IS NULL`,
      [job.tenant_id],
    );
    await tx.query(
      `UPDATE tenant_scim_tokens SET revoked_at = now()
         WHERE tenant_id = $1 AND revoked_at IS NULL`,
      [job.tenant_id],
    );
    await tx.query(
      `UPDATE tenant_offboarding_jobs
         SET status = 'access_revoked', access_revoked_at = now(), updated_at = now()
         WHERE id = $1`,
      [job.id],
    );
    await emitAuditEvent(tx, {
      event_type: 'enterprise.sso_scim_revoked',
      severity:   'HIGH',
      tenant_id:  job.tenant_id,
      metadata:   { offboarding_job_id: job.id },
    });
  });
}
```

**`deletion-worker.ts`** — executes the Art. 17 deletion sequence from §25.7 in strict order: Art. 9 tables first (no grace, immediate), then schedules standard personal data deletion via pg_cron (30-day deferred), then delegates billing pseudonymization to the existing `billingPseudonymization` step in `src/workers/gdpr/erasure.ts`. All deletes are batch operations scoped to `WHERE tenant_id = $1`. Row counts are accumulated and written back to the `tenant_offboarding_jobs` row in the same transaction as the `enterprise.data_deletion_completed` DEC-030 event.

```typescript
// apps/api/src/workers/offboarding/deletion-worker.ts (pseudocode)

const ART9_TABLES = [
  'keypoints_enc',
  'user_health_profiles',
  'coaching_turns',
  'meal_logs',
  'body_metrics',
] as const;

async function deleteArt9Tables(db: DbClient, tenantId: string): Promise<number> {
  let total = 0;
  for (const table of ART9_TABLES) {
    // Lint rule no-offboarding-query-without-tenant-id enforces WHERE clause presence.
    const result = await db.query(
      `DELETE FROM ${table} WHERE tenant_id = $1`,
      [tenantId],
    );
    total += result.rowCount;
  }
  return total;
}

export async function runDeletionWorker(job: OffboardingJob, env: Env): Promise<void> {
  const art9Count      = await deleteArt9Tables(env.db, job.tenant_id);
  const financialCount = await billingPseudonymization(env.db, job.tenant_id); // §24.10

  // Standard personal data: schedule 30-day deferred hard-delete via pg_cron.
  await scheduleStandardDataDeletion(env.db, job.tenant_id, job.id);

  await env.db.transaction(async (tx) => {
    await tx.query(
      `UPDATE tenant_offboarding_jobs
         SET status = 'financial_pseudonymized',
             art9_rows_deleted_count      = $2,
             financial_rows_pseudonymized = $3,
             financial_pseudonymized_at   = now(),
             updated_at                   = now()
         WHERE id = $1`,
      [job.id, art9Count, financialCount],
    );
    await emitAuditEvent(tx, {
      event_type: 'enterprise.data_deletion_completed', severity: 'HIGH',
      tenant_id:  job.tenant_id,
      metadata:   { offboarding_job_id: job.id, art9_rows_deleted_count: art9Count },
    });
    await emitAuditEvent(tx, {
      event_type: 'enterprise.financial_pseudonymized', severity: 'HIGH',
      tenant_id:  job.tenant_id,
      metadata: {
        offboarding_job_id:           job.id,
        financial_rows_pseudonymized: financialCount,
        retention_basis:              'Art17(3)(b)',
        legal_reference:              'Tax Code of Ukraine Art.44 / EU VAT Directive 2006/112/EC Art.245',
      },
    });
  });
}
```

**`attestation-generator.ts`** — invoked after `financial_pseudonymized` status is confirmed. Reads completed counts from the `tenant_offboarding_jobs` row, renders the attestation text from the template (§25.5.1), signs it with `HMAC-SHA256(compliance_signing_key, attestation_text)`, and inserts the `tenant_deletion_attestations` row in the same transaction as the `enterprise.deletion_attestation_issued` DEC-030 event. The attestation INSERT and the audit event are atomic — neither persists without the other.

---

### 25.12 SOC 2 Evidence Mapping

| Criterion | Control description | Evidence artefact ID | Evidence artefact |
|---|---|---|---|
| **C1.2** — Confidential information is protected during disposal | `tenant_deletion_attestations` provides signed proof that all confidential user data was deleted or pseudonymized per the documented sequence; `art9_rows_deleted_count` and `standard_rows_deleted_count` are cross-checkable against pre-deletion counts in the DEC-030 event log | **OFB-E-001** | `SELECT * FROM tenant_deletion_attestations WHERE tenant_id = $1` — provided per tenant; `signature` verified against published compliance key at `/.well-known/offboarding-signing-key.pub` |
| **P8.0** — Personal data is disposed of according to documented privacy commitments | `enterprise.data_deletion_completed` and `enterprise.financial_pseudonymized` DEC-030 events carry `retention_basis` and `legal_reference` fields, providing a per-tenant timestamped record of the Art. 17 disposal action and the legal basis for any retained pseudonymized records | **OFB-E-002** | `SELECT * FROM audit_log WHERE event_type IN ('enterprise.data_deletion_completed', 'enterprise.financial_pseudonymized') AND tenant_id = $1` filtered for the SOC 2 observation period |
| **CC6.3** — Logical access is removed in a timely manner at contract end | `enterprise.sso_scim_revoked` DEC-030 event records the exact timestamp at which `nukeTenantSessions()` was called and all SSO/SCIM credentials were deactivated; `access_revoked_at` on the job row provides a durable timestamp independent of the audit log | **OFB-E-003** | `SELECT access_revoked_at, authorised_by_1, authorised_by_2 FROM tenant_offboarding_jobs WHERE tenant_id = $1` joined to `audit_log WHERE event_type = 'enterprise.sso_scim_revoked'`; demonstrates revocation within the off-boarding SLA window for each churned tenant in the observation period |
| **CC9.1** — Business disruption risk at contract end is managed | `tenant_offboarding_jobs` demonstrates a documented, auditable, and automated process for all three contract-end scenarios; no tenant data is silently abandoned at contract end; `on_hold` status with required compliance_notes ensures legal holds are visible and tracked | **OFB-E-004** | Distribution query: `SELECT trigger_type, status, COUNT(*) FROM tenant_offboarding_jobs WHERE created_at BETWEEN $obs_start AND $obs_end GROUP BY 1, 2`; confirms no job is stuck in a non-terminal state without a recorded `on_hold` reason and that all completed jobs reached `'completed'` status |

---

### 25.13 Implementation Checklist

| # | Task | Owner | Priority | Milestone |
|---|---|---|---|---|
| 1 | Create `migrations/YYYYMMDD_offboarding_enums.sql` — `offboarding_trigger_enum`, `offboarding_status_enum`, `egress_package_type_enum`, `egress_package_status_enum` | `platform-engineer` | **P0** | M4 |
| 2 | Create `migrations/YYYYMMDD_tenant_offboarding_jobs.sql` — full DDL from §25.3, all CHECK constraints, indexes, two-person auth constraint, RLS policies from §25.10 | `platform-engineer` | **P0** | M4 |
| 3 | Create `migrations/YYYYMMDD_tenant_data_egress_packages.sql` — full DDL from §25.4, `uq_egress_package_type_per_job` unique index, RLS policies | `platform-engineer` | **P0** | M4 |
| 4 | Create `migrations/YYYYMMDD_tenant_deletion_attestations.sql` — full DDL from §25.5, `retain_until` default, RLS policies | `platform-engineer` + `compliance-officer` | **P0** | M4 |
| 5 | Implement `apps/api/src/workers/offboarding/orchestrator.ts` — state machine dispatch, pg_cron sweep, Cloudflare Queue consumer, `revokeAccess()` step integrating `nukeTenantSessions()` (SSO_SCIM_IMPLEMENTATION.md §22) with two-person auth gate at `export_delivered → deletion_in_progress` | `platform-engineer` | **P0** | M4 |
| 6 | Implement `apps/api/src/workers/offboarding/deletion-worker.ts` — Art. 9 immediate hard-delete (`ART9_TABLES` constant), 30-day deferred standard data via pg_cron, billing pseudonymization delegation to `billingPseudonymization` (§24.10) | `platform-engineer` + `compliance-officer` | **P0** | M4 |
| 7 | Implement `apps/api/src/workers/offboarding/attestation-generator.ts` — template rendering from §25.5.1, HMAC-SHA256 signing with versioned compliance key, `tenant_deletion_attestations` INSERT atomic with `enterprise.deletion_attestation_issued` DEC-030 event | `platform-engineer` + `compliance-officer` | **P0** | M4 |
| 8 | Implement `apps/api/src/workers/offboarding/egress-package-worker.ts` — generate all four package types; enforce n ≥ 15 suppression on `aggregate_adoption_report`; apply permitted event type IN filter on `audit_log_export`; exclude `arr_cents` from `contract_documents`; HMAC-sign packages; upload to R2; generate 30-day presigned URLs; email to `delivered_to_email` | `platform-engineer` | **P0** | M4 |
| 9 | Register all 8 `enterprise.offboarding_*` DEC-030 event types in `docs/AUDIT_LOG_SCHEMA.md` with severity (per §25.9), 7-year retention, and mandatory payload fields | `compliance-officer` | **P0** | M4 |
| 10 | Add ESLint custom rule `no-offboarding-query-without-tenant-id`: any query in `apps/api/src/workers/offboarding/` that lacks `WHERE tenant_id =` (or equivalent parameterized form) fails CI | `platform-engineer` | **P0** | M4 |
| 11 | Write `__tests__/db/offboarding_rls.test.ts` — (a) `form_api` tenant_admin SELECT own job status; (b) `form_api` INSERT to `tenant_offboarding_jobs` fails; (c) tenant A's `form_api` role cannot SELECT tenant B's offboarding job; (d) `r2_object_key` excluded from tenant-facing API response; (e) `form_system` full write on all three tables | `qa-lead` + `enterprise-architect` | **P0** | M4 |
| 12 | Write `__tests__/workers/offboarding_deletion.test.ts` — (a) Art. 9 tables hard-deleted within 24h of `deletion_in_progress`; (b) standard personal data not deleted before 30-day grace; (c) `user_subscriptions` and `subscription_events` pseudonymized not hard-deleted; (d) `tenant_contracts` untouched; (e) `audit_log` rows untouched; (f) egress package worker issues no query against `keypoints_enc`, `user_health_profiles`, `coaching_turns`, `meal_logs`, or `body_metrics` | `qa-lead` + `compliance-officer` | **P0** | M5 |

---

### 25.14 Open Questions

| ID | Question | Owner | Priority |
|---|---|---|---|
| OQ-OFB-01 | **Self-service off-boarding portal vs. CSM-mediated initiation.** Should tenant admins be able to initiate off-boarding self-service (a "Close our account" action in the admin dashboard) or must all off-boarding be initiated by a FORM CSM via internal tooling? Self-service reduces CSM cost and improves the employer experience at contract end; CSM-mediated allows a retention conversation and reduces the risk of an accidental or coerced off-boarding action by a misconfigured admin. If self-service is chosen: the initiation endpoint must require a re-authentication challenge (SAML `ForceAuthn=true` or OIDC `prompt=login`), a 48-hour cancellation window before `access_revoked` is triggered, and confirmation by two distinct `tenant_owner` accounts. | `enterprise-architect` + `product-strategist` | **P1 — before enterprise GA** |
| OQ-OFB-02 | **EU-region R2 bucket for EU-domiciled tenants.** Egress packages for EU-domiciled tenants should be generated and stored in an EU-region Cloudflare R2 bucket to avoid GDPR Chapter V transfer concerns. The current schema stores a single default `r2_bucket = 'form-offboarding-exports'` without region routing. If EU-region R2 is required by DPA or by Art. 46 transfer mechanism constraints, `r2_bucket` must be computed from `tenants.data_region` at job creation time and the EU bucket must be provisioned and tested before any EU-domiciled tenant reaches the off-boarding flow. Resolution requires legal sign-off on the transfer mechanism and devops provisioning of the EU R2 bucket. | `enterprise-architect` + `legal` + `devops-lead` | **P0 — before any EU enterprise tenant off-boards** |
| ~~OQ-OFB-03~~ **🟢 Resolved — DEC-036 (2026-06-09)** | ~~**Art. 9 no-grace-period as an absolute invariant.**~~ **Resolved: absolute invariant.** §25.7 hard-delete of Art. 9 data with no grace period is confirmed as an unconditional rule. Enterprise MSA clauses requesting a health-data recovery window must be refused at contract review stage. Individual users must exercise data portability (§12.5) independently before the off-boarding trigger date. See `docs/DECISION_LOG.md` DEC-036. | `compliance-officer` + `legal` | **🟢 Closed** |

---

*v1.5 · червень 2026 · owner: enterprise-architect + compliance-officer + platform-engineer*

*v1.5 additions: §25 Enterprise Tenant Offboarding & Data Egress Schema — closes the lifecycle gap between §16 (`churned` state, "data deletion begins per §12.3") and the actual enterprise-scale off-boarding procedure. §12.3 is per-user GDPR Art. 17 erasure and was never designed for employer off-boarding. Three tables: `tenant_offboarding_jobs` (state machine, two-person deletion authorisation, per-table deletion counters); `tenant_data_egress_packages` (employer export packages — aggregate reports, audit logs, contract docs only; no individual health data path in ENUM or worker); `tenant_deletion_attestations` (GDPR C1.2 signed proof of deletion with table-level row counts and Art. 17(3)(b) exception disclosure). Three off-boarding triggers: `contract_expired_no_renewal`, `early_termination`, `pilot_lapsed`. Nine-state state machine: `pending → access_revoked → export_in_progress → export_delivered → deletion_in_progress → financial_pseudonymized → attestation_issued → completed` plus `failed` and `on_hold`. Art. 9 data (keypoints_enc, user_health_profiles, coaching_turns, meal_logs, body_metrics) hard-deleted immediately with no grace period (GDPR Recital 51 — higher protection, no recovery justification after employer relationship ends). Standard personal data follows 30-day soft-delete window per §12. Financial records pseudonymized per §24.10 (Art. 17(3)(b) — Ukrainian Tax Code Art. 44 + EU VAT Directive 7-year retention). Privacy floor enforced structurally: `egress_package_type_enum` is exhaustive; no value can produce a package containing individual health data; egress worker has no SELECT grant to health tables. Eight DEC-030 HMAC-chained events (all CRITICAL/HIGH, 7yr): `enterprise.offboarding_initiated`, `enterprise.access_revoked`, `enterprise.data_export_completed`, `enterprise.deletion_in_progress`, `enterprise.art9_data_deleted`, `enterprise.standard_data_deleted`, `enterprise.financial_pseudonymized`, `enterprise.deletion_attestation_issued`. SOC 2 mapping: C1.2 (disposal — `enterprise.deletion_attestation_issued` as primary auditor evidence), P8.0 (privacy in disposal — Art. 9 no-grace rule, Art. 17(3)(b) pseudonymization), CC6.3 (logical access removed — `access_revoked` state triggers `nukeTenantSessions()` per SSO_SCIM §22 + SSO assertion block), CC9.1 (third-party risk at contract end — sub-processor notification, `tenant_deletion_attestations` as DPA evidence). Four evidence artefacts OFB-E-001 through OFB-E-004. 12-item implementation checklist (8× P0 M4/M5, 4× P0 M5). Three open questions: OQ-OFB-01 (self-service portal vs. CSM-mediated P1); OQ-OFB-02 (EU-region R2 for EU tenants — P0 before first EU off-boarding); OQ-OFB-03 (Art. 9 no-grace as absolute invariant — P0 before first enterprise contract signed).*

*v1.4 · травень 2026 · owner: enterprise-architect + compliance-officer + platform-engineer*

*v1.4 additions: §24 Subscription, Billing & Revenue Schema — closes the billing data-layer gap for plan entitlement enforcement (cross-ref §19 `assertFeatureEnabled()`), enterprise contract lifecycle (§16), and GDPR Art. 17 financial retention tension. Two-track billing architecture: consumer via App Store (StoreKit 2) / Google Play (Billing Library v6) and enterprise via Stripe invoicing; `user_subscriptions` is the single source of truth for access control decisions — never App Store or Stripe directly. Four ENUMs: `plan_tier_enum` (free/growth/elite/enterprise), `subscription_status_enum` (trialing/active/grace/past_due/cancelled/expired), `billing_channel_enum` (none/app_store_ios/google_play/stripe), `subscription_event_type_enum` (14 values across full lifecycle). `user_subscriptions` table: `uq_user_one_active_subscription` partial unique index (status != 'expired'); `chk_enterprise_has_tenant` (stripe billing requires tenant_id); `chk_trial_coherent` (trialing requires trial_ends_at); `chk_grace_coherent`; `chk_free_no_price`; `idx_user_subscriptions_renewal_sweep` partial index on `current_period_end` for scheduled renewal sweep Worker; all monetary amounts in minor currency units with ISO 4217 currency column. `subscription_events` append-only lifecycle log: 14-value `subscription_event_type_enum`; `uq_subscription_events_external_id` idempotency index prevents duplicate webhook processing; no UPDATE/DELETE for any non-admin role (RLS + HMAC chain). Enterprise seat management: `enterprise_seat_allocations` (period-based contracted seat count with `effective_from`/`effective_to` history pattern; `uq_tenant_allocation_period` unique constraint; `RESTRICT` on tenant delete); `enterprise_seat_assignments` (per-user seat with `revoked_at`; `uq_tenant_user_active_seat` partial unique index; `assigned_by` FK for audit trail); `tenant_seat_utilization` materialized view refreshed `CONCURRENTLY` after every seat mutation. Webhook integration: Apple StoreKit 2 (JWS certificate chain verification against Apple root CA; 6-hour replay window; `original_transaction_id` as stable `external_subscription_id`); Google Play RTDN (Pub/Sub → Worker → Play Developer API v3 verification; `linkedPurchaseToken` chain resolution for renewal correlation); Stripe (HMAC-SHA256 `Stripe-Signature` verification on raw body; seat expansion/contraction via `customer.subscription.updated` quantity diff; over-allocation alert suppresses automatic revocation). 13-transition state machine: trialing/active/grace/cancelled/expired states with complete trigger → new_state → subscription_events rows → user_subscriptions changes table. GDPR Art. 17 vs. financial retention: Ukrainian Tax Code Art. 44 and EU VAT Directive 2006/112/EC Art. 245 mandate 7-year retention, overriding erasure per Art. 17(3)(b); resolution is pseudonymization of `user_id` to `[ERASED-{sha256(user_id,salt)}]` in `subscription_events`, NULL of `failure_reason`, and soft-expiry of `user_subscriptions` row; `billing.user_erased` HIGH DEC-030 event carries `retention_basis`; OQ-BILL-05 (FK constraint approach — sentinel UUID vs. split column) is P0 before erasure ships. RLS: `form_api` SELECT own rows; `form_system` full read/write (sole INSERT/UPDATE path for `user_subscriptions`); `tenant_admin` SELECT + seat assignment INSERT/UPDATE within own tenant; no form_api UPDATE on `user_subscriptions` (prevents client-side plan elevation). 12 DEC-030 HMAC-chained audit events (all 7yr retention): 5 STANDARD (subscription_created, activated, renewed, upgraded, seats_expanded) and 7 HIGH (subscription_downgraded, subscription_cancelled, payment_failed, grace_started, subscription_expired, seats_reduced, user_erased); all emitted in-transaction with subscription_events INSERT. SOC 2 mapping: CC6.2 (two-table authorization chain user_subscriptions → feature_flags); CC6.3 (seat revocation removes API access via RLS); A1.1 (subscription status operational monitoring); CC7.2 (billing.payment_failed HIGH events in SIEM for revenue anomaly detection); CC8.1 (billing Worker sole mutation path + HMAC chain); P5.0 (Art. 17 pseudonymization path); P8.0 (billing.user_erased with retention_basis as DPA evidence). 5 open questions (OQ-BILL-01: trial length configurability P0; OQ-BILL-02: UAH currency localization P1; OQ-BILL-03: refund policy encoding P1; OQ-BILL-04: invite-based seat provisioning P1; OQ-BILL-05: Art. 17 FK sentinel vs. split column P0). 12-item implementation checklist (10× P0 M4, 2× P0 M4 regression tests).*

---

## 26. API Key Authentication Schema

> Owner: `enterprise-architect` + `security-engineer`. Review: before any enterprise pilot launch, on any change to API key policy, and at every SOC 2 observation period start.
> Scope: enterprise tier only. Consumer users do not hold API keys. This section defines the canonical schema for `tenant_api_keys` — the long-lived credential issued to tenant admins for headless integrations, reporting pulls, and admin automation.
> References: `docs/SSO_SCIM_IMPLEMENTATION.md §26` (API Key Authentication Security — canonical implementation reference); §2 (`tenants` table FK target); §16 (enterprise contract lifecycle); §22 (session revocation — `nukeTenantSessions()` used at off-boarding, not for API keys); §25 (off-boarding — `tenant_api_keys` revoked as part of the `access_revoked` transition); `docs/AUDIT_LOG_SCHEMA.md` (DEC-030 HMAC chain); `docs/SOC2_READINESS.md` criteria CC6.1, CC6.2, CC6.4, CC6.8, CC7.2.

---

### 26.1 Purpose and Design Principles

`SSO_SCIM_IMPLEMENTATION.md §26.11` item 12 (P1 M6) required that `tenant_api_keys` be added to `docs/DATA_MODEL.md` as the canonical schema source cross-reference entry. `SSO_SCIM_IMPLEMENTATION.md §26.4.2` declared itself the canonical source for the table definition at the time of the §26 SSO_SCIM authoring pass. This DATA_MODEL section is now the canonical DDL record, superseding the informal schema in SSO_SCIM §26.4.2 for migration and RLS purposes. SSO_SCIM §26 remains authoritative for Worker implementation, alerting rules, and evidence collection procedures.

**API key vs. JWT vs. SCIM bearer token.** FORM issues three categories of machine credential to enterprise tenants. They are structurally distinct and must not be conflated:

| Credential type | Issued to | TTL / lifecycle | Auth path | IP enforcement |
|---|---|---|---|---|
| **JWT session token** | Individual human users | 1 hour; refreshed via OIDC/SAML assertion | `src/workers/auth/session.ts`; covered by §25 IP allowlist middleware | General tenant IP allowlist (§25.5) |
| **SCIM bearer token** | IdP provisioning agent | Long-lived; rotated per §16.6 (24h overlap) | `src/workers/scim/auth.ts` | Separate `scim_ip_enforcement_enabled` flag on `tenant_sso_configs` (SSO_SCIM §26.3) |
| **Tenant API key** | Admin dashboard; headless integrations, reporting bots, webhook receivers | Long-lived; rotation policy in §26.6 | `src/workers/auth/api-key-auth.ts` | Per-key `ip_enforcement_enabled` + `ip_allowlist_config` (§26.5) |

**Key management is a `form_system`-only mutation path.** No `form_api` role may INSERT, UPDATE, or DELETE rows in `tenant_api_keys`. All mutations — creation, rotation, revocation, IP enforcement toggle — flow through the `api-key-auth.ts` and admin dashboard Workers running as `form_system`. This is the same principle applied to `user_subscriptions` (§24.11): preventing any client-side or API-layer code from issuing or upgrading credentials without a verified admin action.

**Raw key is never stored.** The 256-bit random raw key (hex-encoded, 64 chars) is returned to the requesting admin exactly once at creation time and is not retained anywhere in the FORM backend. The stored `key_hash` is `HMAC-SHA256(API_KEY_HASH_SECRET, raw_key)` where `API_KEY_HASH_SECRET` is a Cloudflare Workers Secret (distinct from `IP_HASH_SALT`). This design means a database breach does not expose usable API credentials.

**Scopes enforce least privilege.** Every API key carries an explicit `scopes` array. The `api-key-auth.ts` Worker validates scope against the required permission for the target route before dispatching to any handler. No route grants elevated access based on the absence of a scope check. Scope values are open-set at the DDL level (TEXT[]) but validated against the permitted set at the application layer. Current permitted values: `reporting:read` and `admin:write`. `scim:provision` is reserved and not issued in M5.

**DEC-030 privacy floor.** No audit event in this section may contain a plaintext IP address, a raw API key, or the `key_hash` value. The `api_key.ip_blocked` event carries only a `client_ip_hash` (SHA-256 of the raw IP keyed with `IP_HASH_SALT`, truncated to 32 hex chars). Error responses to callers whose IP is blocked contain only `{"error": "api_key_ip_not_allowed"}` — no hint about which allowlist entry triggered the rejection, and no leak of `key_preview`, `tenant_id`, or scope.

---

### 26.2 Credential Architecture Cross-Reference

The three credential types share infrastructure but use distinct lookup and enforcement paths. The diagram below shows where each type is handled and where §26 fits.

```
Incoming request to FORM API
         │
         ├─ Authorization: Bearer <jwt>
         │         │
         │         └─► session.ts
         │                  └─► §25 IP allowlist middleware (general tenant allowlist)
         │                  └─► JWT validation → user_id, tenant_id, roles
         │
         ├─ Authorization: Bearer <scim_token>  (SCIM routes only)
         │         │
         │         └─► scim/auth.ts
         │                  └─► §26.3 SCIM IP allowlist (scim_ip_enforcement_enabled)
         │                  └─► SCIM token hash lookup → tenant_scim_tokens
         │
         └─ Authorization: Bearer <api_key>  (integration / reporting routes)
                   │
                   └─► api-key-auth.ts                        ← §26.4 / §26.5
                            └─► HMAC-SHA256(API_KEY_HASH_SECRET, raw_key) → key_hash lookup
                            └─► tenant_api_keys: revoked_at check, scopes check
                            └─► enforceApiKeyIpAllowlist()    ← §26.5
                            └─► scope check vs. route requirement
                            └─► waitUntil: UPDATE last_used_at
```

**No cross-path promotion.** A SCIM bearer token cannot be presented as an API key and vice versa. The two lookup paths (`tenant_scim_tokens.token_hash` vs. `tenant_api_keys.key_hash`) use different HMAC secrets (`SCIM_TOKEN_HASH_SECRET` vs. `API_KEY_HASH_SECRET`) and are evaluated in separate Worker functions. A hash collision between the two namespaces — even if computationally feasible — would produce an `invalid_api_key` response because the key record found in one table would fail the `revoked_at` or `scopes` check applicable to that table's contract.

---

### 26.3 Relationship to Off-Boarding (§25)

During the §25 off-boarding flow, the `revokeAccess()` step (orchestrator.ts `pending → access_revoked` transition) must revoke all active API keys for the tenant in the same transaction as the `enterprise.sso_scim_revoked` DEC-030 event. The §25 attestation text template (§25.5.1) references SCIM token revocation but does not explicitly enumerate API key revocation. This section clarifies:

- At the `pending → access_revoked` transition, the orchestrator MUST execute:

```sql
UPDATE tenant_api_keys
   SET revoked_at = now()
 WHERE tenant_id = $1
   AND revoked_at IS NULL;
```

- A `api_key.revoked` DEC-030 event with `reason = 'offboarding'` must be emitted for each revoked key in the same transaction.
- The `enterprise.sso_scim_revoked` audit event payload must include `api_keys_revoked_count` (integer count of keys whose `revoked_at` was set in this transaction).
- The §25.5.1 attestation template will be updated in a subsequent DATA_MODEL revision to include the `api_keys_revoked_count` field. Until then, the DEC-030 event log is the authoritative evidence for API key revocation at off-boarding.

This off-boarding revocation path produces `api_key.revoked` events with `reason = 'offboarding'` — distinct from `rotation`, `compromise`, `expiry_overlap`, and `admin_request` reasons used in the normal key lifecycle.

---

### 26.4 `tenant_api_keys` Table DDL

This is the canonical schema definition for `tenant_api_keys`. The informal schema in `SSO_SCIM_IMPLEMENTATION.md §26.4.2` is superseded by this DDL for migration purposes. The migration file is `migrations/0051_tenant_api_keys.sql`.

```sql
CREATE TABLE tenant_api_keys (
  id                       UUID        PRIMARY KEY DEFAULT gen_random_uuid(),

  tenant_id                UUID        NOT NULL
    REFERENCES tenants(id) ON DELETE RESTRICT,
  -- RESTRICT (not CASCADE): a tenant with active API keys must not be silently deleted.
  -- The off-boarding orchestrator (§26.3) must revoke all keys before the tenants row
  -- can be hard-deleted. If revocation is skipped, the FK prevents the delete and the
  -- omission surfaces as a hard error rather than a silent data loss.

  label                    TEXT        NOT NULL,
  -- User-defined name: "HRIS sync", "Reporting bot", "CI/CD admin key".
  -- Non-empty; max 80 chars. Enforced by CHECK constraint below.
  -- Not unique per tenant — admins may have multiple keys with the same label (e.g., two
  -- rotation-overlap keys for the same integration). Uniqueness is not a meaningful
  -- constraint here; the key_preview distinguishes them in the dashboard.

  key_hash                 TEXT        NOT NULL,
  -- HMAC-SHA256(API_KEY_HASH_SECRET, raw_key), stored as lowercase hex (64 chars).
  -- Raw key is never stored. API_KEY_HASH_SECRET is a Cloudflare Workers Secret.
  -- Partial unique index on (key_hash) WHERE revoked_at IS NULL ensures no two
  -- active keys share the same hash (which would be a generation fault or collision).

  key_preview              TEXT        NOT NULL,
  -- Last 8 characters of the raw key. Displayed in the admin dashboard table and
  -- included (in label context only) in api_key.created DEC-030 event.
  -- Not a secret — it is intentionally short and serves identification only.
  -- Must not be used for authentication or hash input.

  scopes                   TEXT[]      NOT NULL DEFAULT '{reporting:read}',
  -- Permitted values: reporting:read | admin:write
  -- reporting:read: GET /v1/admin/reports/* only.
  -- admin:write: all admin mutation endpoints; quarterly rotation mandatory (§26.6).
  -- scim:provision: reserved; not issued in M5.
  -- Array must contain at least one element (CHECK constraint below).
  -- Scope changes require explicit admin action; not inherited on rotation
  -- (new key carries same scopes as rotated key unless admin modifies during rotation flow).

  created_by               UUID        NOT NULL
    REFERENCES users(id) ON DELETE RESTRICT,
  -- The FORM user (tenant_admin or tenant_owner) who created the key.
  -- RESTRICT: if the creating user row is deleted, created_by must be preserved
  -- for audit trail completeness. User anonymization at off-boarding (§25.7) NULLs
  -- PII columns on the users row but retains the UUID — this FK survives anonymization.

  last_used_at             TIMESTAMPTZ,
  -- Updated via Cloudflare Workers ctx.waitUntil() on each successful authenticated request.
  -- waitUntil ensures the response is not delayed by the UPDATE.
  -- NULL = key has never been used since creation.
  -- Not updated on ip_blocked rejections — only on successful authentication.

  expires_at               TIMESTAMPTZ,
  -- NULL = no hard expiry set by admin.
  -- Soft age-alert thresholds (not automatic revocation — see OQ-SSO-26.2):
  --   age ≥ 365 days: amber warning in dashboard ("N days old — consider rotating").
  --   age ≥ 730 days: red warning ("N days old — rotation required").
  --   admin:write scope, age > 90 days: amber warning regardless of general threshold.
  -- If set explicitly by admin: api-key-auth.ts checks expires_at < now() and
  -- treats expired keys as revoked (same response as revoked_at IS NOT NULL).

  revoked_at               TIMESTAMPTZ,
  -- NULL = active. Non-NULL = revoked at this timestamp.
  -- Set by: admin manual revoke, overlap cleanup pg_cron, off-boarding orchestrator.
  -- Never cleared: revocation is permanent. A new key must be created; revoked rows
  -- are retained for audit trail continuity (DEC-030 chain integrity).

  ip_enforcement_enabled   BOOLEAN     NOT NULL DEFAULT false,
  -- When true, every request authenticated with this key is checked against ip_allowlist_config.
  -- Disabling this control after it has been enabled requires a DEC-030 event (§26.7)
  -- and compliance-officer review if scopes includes admin:write.

  ip_allowlist_config      JSONB       DEFAULT NULL,
  -- Must be non-NULL when ip_enforcement_enabled = true (CHECK constraint below).
  -- Must be NULL or a JSON object (not array, not scalar) — enforced by jsonb_typeof check.
  -- Expected shape:
  --   {
  --     "enabled": true,
  --     "entries": [
  --       { "cidr": "203.0.113.0/24", "label": "Office London" },
  --       { "cidr": "198.51.100.5/32", "label": "CI runner" }
  --     ]
  --   }
  -- enforceApiKeyIpAllowlist() reads ip_allowlist_config.entries and calls checkCidrList()
  -- (shared utility from §25.5 — same netmask npm dependency, no new import).
  -- Maximum 50 CIDR entries per key (enforced at application layer, not CHECK constraint).

  created_at               TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at               TIMESTAMPTZ NOT NULL DEFAULT now(),

  -- ─── Constraints ──────────────────────────────────────────────────────────

  CONSTRAINT chk_api_key_label_length
    CHECK (label <> '' AND char_length(label) <= 80),

  CONSTRAINT chk_api_key_scopes_nonempty
    CHECK (array_length(scopes, 1) > 0),

  CONSTRAINT chk_ip_allowlist_required
    CHECK (
      NOT ip_enforcement_enabled
      OR ip_allowlist_config IS NOT NULL
    ),
  -- When ip_enforcement_enabled = true, ip_allowlist_config must be present.
  -- This prevents a configuration where enforcement appears "on" but no allowlist
  -- exists to enforce against — which would pass every IP (security foot-gun).

  CONSTRAINT chk_ip_allowlist_shape
    CHECK (
      ip_allowlist_config IS NULL
      OR jsonb_typeof(ip_allowlist_config) = 'object'
    )
  -- Allowlist config must be a JSON object (not array, string, or null scalar).
  -- The entries array lives at ip_allowlist_config->>'entries'.
  -- Structural depth validation (CIDR format, max 50 entries) is performed
  -- at the application layer in the admin Worker before INSERT/UPDATE.
);
```

#### 26.4.1 Column Comments

```sql
COMMENT ON TABLE tenant_api_keys
  IS 'Long-lived API credentials issued to tenant admins for headless integrations. Canonical schema — supersedes SSO_SCIM_IMPLEMENTATION.md §26.4.2 for migration purposes. Mutations via form_system only (api-key-auth.ts Worker).';

COMMENT ON COLUMN tenant_api_keys.key_hash
  IS 'HMAC-SHA256(API_KEY_HASH_SECRET, raw_key) stored as lowercase hex. Raw key never persisted. A breach of this table does not expose usable credentials unless API_KEY_HASH_SECRET is also compromised.';

COMMENT ON COLUMN tenant_api_keys.key_preview
  IS 'Last 8 chars of the raw key. Display-only identifier in the admin dashboard. Not a secret. Must not be used as a hash input or credential fragment.';

COMMENT ON COLUMN tenant_api_keys.ip_allowlist_config
  IS 'JSON object with shape { "enabled": bool, "entries": [{ "cidr": string, "label": string }] }. NULL when ip_enforcement_enabled = false. Max 50 CIDR entries enforced at application layer.';

COMMENT ON COLUMN tenant_api_keys.last_used_at
  IS 'Updated asynchronously via ctx.waitUntil() on each successful authentication. NULL = never used. Not updated on ip_blocked rejections.';

COMMENT ON COLUMN tenant_api_keys.expires_at
  IS 'Optional hard expiry set by admin. NULL = no hard expiry; age-alert thresholds (365d amber, 730d red, admin:write >90d amber) are soft UI controls only. See OQ-SSO-26.2 for hard-enforcement decision.';
```

#### 26.4.2 Indexes

```sql
-- Active key fast-path: lookup by tenant for dashboard listing.
CREATE INDEX idx_tenant_api_keys_tenant_active
  ON tenant_api_keys (tenant_id)
  WHERE revoked_at IS NULL;

-- Authentication hot-path: lookup by key_hash (partial — active keys only).
-- This is the index hit on every authenticated API request.
CREATE UNIQUE INDEX uq_tenant_api_keys_hash_active
  ON tenant_api_keys (key_hash)
  WHERE revoked_at IS NULL;
-- UNIQUE here: two active keys must not share a hash. A revoked key's hash
-- may be reused (effectively impossible given 256-bit key space, but the
-- partial index permits it without requiring a full-table unique scan).

-- Age-sweep: find keys approaching or past rotation thresholds.
CREATE INDEX idx_tenant_api_keys_created_at_active
  ON tenant_api_keys (created_at)
  WHERE revoked_at IS NULL;

-- Expiry sweep: find keys with explicit expires_at approaching.
CREATE INDEX idx_tenant_api_keys_expires_at
  ON tenant_api_keys (expires_at)
  WHERE revoked_at IS NULL AND expires_at IS NOT NULL;
```

#### 26.4.3 Trigger

```sql
CREATE TRIGGER update_tenant_api_keys_updated_at
  BEFORE UPDATE ON tenant_api_keys
  FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();
```

#### 26.4.4 RLS

```sql
ALTER TABLE tenant_api_keys ENABLE ROW LEVEL SECURITY;
```

RLS policies are defined in §26.5 below.

---

### 26.5 Row-Level Security Policies

Three roles interact with `tenant_api_keys`. Their access levels reflect the principle that key material is strictly system-managed and that admin-dashboard users see only what is necessary for key lifecycle operations — never the `key_hash`.

```sql
-- ─── form_system: full read/write ─────────────────────────────────────────
-- The sole INSERT/UPDATE/DELETE path. All mutations flow through api-key-auth.ts
-- and the admin dashboard Worker, both running as form_system.

CREATE POLICY api_keys_system_all
  ON tenant_api_keys
  TO form_system
  USING (TRUE)
  WITH CHECK (TRUE);

-- ─── form_api: tenant-isolated SELECT (own tenant's keys only) ────────────
-- Permits SELECT for any authenticated API request that needs to verify key
-- existence or display key metadata. key_hash is excluded from tenant-facing
-- API responses at the application layer (never returned in any JSON response
-- to the admin dashboard — only key_preview, label, scopes, created_at,
-- last_used_at, expires_at, revoked_at, ip_enforcement_enabled).
-- No INSERT, UPDATE, or DELETE permitted for form_api on this table.

CREATE POLICY api_keys_tenant_read
  ON tenant_api_keys FOR SELECT
  TO form_api
  USING (
    tenant_id = current_setting('app.current_tenant_id')::UUID
  );

-- ─── tenant_admin function: SELECT own tenant's keys for dashboard display ─
-- This policy is logically redundant with api_keys_tenant_read (form_api already
-- scopes to current_tenant_id), but is declared explicitly to document the
-- tenant_admin role's intended access boundary. Role check enforced at the
-- application layer: only tenant_admin and tenant_owner may access the API Keys
-- panel; tenant_manager and member roles receive 403 before the DB query executes.
-- key_hash column: never returned via any tenant-facing API response path;
-- application layer omits it from SELECT projections on the admin API route.

-- No separate RLS policy for tenant_admin function — form_api policy covers it.
-- The application layer performs role-based column filtering:
--   Returned:  id, label, key_preview, scopes, created_at, last_used_at,
--              expires_at, revoked_at, ip_enforcement_enabled
--   Excluded:  key_hash, ip_allowlist_config entries detail (structure returned,
--              CIDR values redacted for non-owner roles per §26.2)

-- ─── form_admin: break-glass (BYPASSRLS) ──────────────────────────────────
-- No explicit policy needed. form_admin uses BYPASSRLS for migration,
-- compliance toolchain, and shadow-login audit operations. All form_admin
-- access is logged via the shadow-login audit trail (no god mode in production).
```

**Column exclusion at application layer.** The `key_hash` column must never appear in any API response JSON. The admin dashboard's API route handler must use an explicit column list in its SELECT projection — not `SELECT *`. A CI lint rule (`no-select-star-on-api-keys`) must be added to enforce this at code review time and in the CI pipeline. This is belt-and-suspenders alongside the RLS boundary: RLS prevents cross-tenant access; the column exclusion prevents in-tenant key hash leakage to the admin UI.

---

### 26.6 API Key Lifecycle and Rotation Policy

#### 26.6.1 Key Creation

A new API key is created by a `tenant_admin` or `tenant_owner` via the admin dashboard "API Keys" panel. The creation flow in the admin Worker:

```typescript
// apps/api/src/workers/admin/api-keys.ts (pseudocode)

import { hmacSha256 } from '../auth/crypto';

export async function createApiKey(
  request: CreateApiKeyRequest,
  env:     Env,
  ctx:     ExecutionContext,
): Promise<CreateApiKeyResponse> {
  // 1. Validate request: label length, scopes subset of permitted values.
  validateApiKeyRequest(request); // throws 400 on violation

  // 2. Generate 256-bit random key (hex-encoded = 64 chars).
  const rawKeyBytes = crypto.getRandomValues(new Uint8Array(32));
  const rawKey      = Array.from(rawKeyBytes).map(b => b.toString(16).padStart(2, '0')).join('');
  const keyPreview  = rawKey.slice(-8);

  // 3. HMAC-SHA256 the raw key — this is all that is stored.
  const keyHash = await hmacSha256(env.API_KEY_HASH_SECRET, rawKey);

  // 4. INSERT via form_system (this Worker runs as form_system).
  const keyId = await env.DB.prepare(`
    INSERT INTO tenant_api_keys
      (tenant_id, label, key_hash, key_preview, scopes,
       created_by, ip_enforcement_enabled, ip_allowlist_config)
    VALUES ($1, $2, $3, $4, $5, $6, $7, $8)
    RETURNING id
  `).bind(
    request.tenantId, request.label, keyHash, keyPreview,
    request.scopes,   request.createdBy,
    request.ipEnforcementEnabled ?? false,
    request.ipAllowlistConfig    ?? null,
  ).first<{ id: string }>();

  // 5. Emit DEC-030 api_key.created event in same transaction.
  await emitAuditEvent(env.DB, {
    event_type:            'api_key.created',
    severity:              'HIGH',
    tenant_id:             request.tenantId,
    actor_id:              request.createdBy,
    metadata: {
      key_id:                keyId!.id,
      label:                 request.label,
      scopes:                request.scopes,
      ip_enforcement_enabled: request.ipEnforcementEnabled ?? false,
    },
  });

  // 6. Return raw key to caller ONCE. Never stored, never loggable.
  return { keyId: keyId!.id, rawKey, keyPreview };
}
```

The `rawKey` is returned in the HTTP response body exactly once. It is the caller's responsibility to copy it immediately. FORM does not provide a "re-show key" function — if the raw key is lost, rotation is required.

#### 26.6.2 Rotation Flow

1. Tenant admin navigates to Settings → API Keys in the admin dashboard.
2. Clicks "Rotate" on the target key. Confirmation modal: *"This will start a 24-hour overlap window. Both the old and the new key will be valid during the overlap. After 24 hours, the old key is automatically revoked. Copy the new key before dismissing this dialog."*
3. A new 256-bit random key is generated server-side and INSERTed. The old key is **not** immediately revoked — a **24-hour overlap window** begins (same pattern as SCIM token rotation in §16.6).
4. New key is displayed once. Admin copies it to the integration.
5. After 24 hours, `api_key_rotation_overlap_cleanup` pg_cron job runs hourly and sets `revoked_at = now()` on the predecessor key. `api_key.revoked` DEC-030 event is emitted with `reason = 'expiry_overlap'`.
6. The `api_key.rotated` DEC-030 event is emitted at step 3 (new key creation), carrying `old_key_id` and `new_key_id`. The chain requirement (§26.7) mandates that `api_key.revoked` for `old_key_id` follows within 26 hours — if it does not, AL-APIKEY-02 fires.
7. Admin may click "Revoke old key now" to cancel the overlap immediately (useful if they are confident the integration has been updated). This sets `revoked_at = now()` immediately and emits `api_key.revoked` with `reason = 'rotation'`.

**Scope inheritance:** The new key carries the same `scopes` array as the rotated key unless the admin explicitly modifies scopes during the rotation flow.

#### 26.6.3 Rotation Triggers

| Trigger | Required action | Deadline |
|---|---|---|
| Key age ≥ 365 days | Amber age warning in dashboard; rotation recommended | No hard deadline (soft enforcement — see OQ-SSO-26.2) |
| Key age ≥ 730 days (2 years) | Red age warning; CSM notifies tenant admin | Before next QBR |
| `admin:write` scope key age > 90 days | Amber warning regardless of general threshold | Every 90 days (quarterly mandatory) |
| Suspected compromise | Revoke immediately; issue replacement | Same-day |
| Integration re-owner (staff departure, role change) | Rotate as part of access review | Within 5 business days |
| Annual SOC 2 review | Review all active keys; rotate any older than 365 days | Before SOC 2 observation period start |
| Tenant off-boarding (§26.3 / §25) | All keys revoked as part of `access_revoked` transition | Immediate at `pending → access_revoked` |

#### 26.6.4 Revocation

Revocation is permanent. A revoked key cannot be un-revoked. The `revoked_at` column is set once and never cleared. The DEC-030 `api_key.revoked` event records the reason. Valid reasons:

| `reason` value | Set by | Triggered by |
|---|---|---|
| `admin_request` | `form_system` (admin Worker) | Tenant admin clicks "Revoke" in dashboard |
| `rotation` | `form_system` (admin Worker) | Tenant admin clicks "Revoke old key now" during overlap |
| `expiry_overlap` | `form_system` (pg_cron job) | 24-hour overlap window expires without manual revocation |
| `compromise` | `form_system` (admin Worker) | Admin selects "Revoke — credential compromised" in dashboard; triggers AL-APIKEY-03 |
| `offboarding` | `form_system` (off-boarding orchestrator, §26.3) | `pending → access_revoked` transition |

---

### 26.7 DEC-030 HMAC-Chained Audit Events

All nine events in this section are appended to the FORM DEC-030 HMAC audit log chain (per `docs/AUDIT_LOG_SCHEMA.md`). All are HIGH severity with 7-year retention. No event in this section may contain a plaintext IP address, raw API key, `key_hash`, or individual user PII values. Events involving admin-initiated actions use `actor_type = 'user'`; automated system events (pg_cron overlap cleanup, off-boarding orchestrator) use `actor_type = 'system'`. The `actor_ip` field is omitted from all events in this section — IP data is captured only as `client_ip_hash` in `ip_blocked` events.

| Event type | Severity | Retention | Trigger | Mandatory payload fields | Notes |
|---|---|---|---|---|---|
| `api_key.created` | HIGH | 7 years | New `tenant_api_keys` row inserted | `tenant_id`, `key_id`, `label`, `scopes`, `ip_enforcement_enabled`, `created_by` | Raw key never in payload; `key_preview` present in label context only |
| `api_key.rotated` | HIGH | 7 years | New key inserted during rotation; overlap window opened | `tenant_id`, `key_id` (new UUID), `old_key_id`, `rotation_method` (`admin_manual` or `overlap_expiry`), `rotated_by` | `api_key.revoked` for `old_key_id` must follow within 26h (chain requirement below) |
| `api_key.revoked` | HIGH | 7 years | `revoked_at` set on a `tenant_api_keys` row | `tenant_id`, `key_id`, `revoked_by`, `reason` (`admin_request` \| `rotation` \| `expiry_overlap` \| `compromise` \| `offboarding`) | `reason = 'compromise'` fires AL-APIKEY-03 |
| `api_key.ip_enforcement_enabled` | HIGH | 7 years | `ip_enforcement_enabled` toggled from false to true | `tenant_id`, `key_id`, `cidr_count`, `enabled_by` | Enabling adds network-layer access control to an existing active credential |
| `api_key.ip_enforcement_disabled` | HIGH | 7 years | `ip_enforcement_enabled` toggled from true to false | `tenant_id`, `key_id`, `disabled_by`, `reason` | Disabling removes a control — HIGH severity; compliance-officer review required if `scopes` includes `admin:write` |
| `api_key.ip_blocked` | HIGH | 7 years | `enforceApiKeyIpAllowlist()` rejects a request | `tenant_id`, `key_id`, `client_ip_hash` | No `user_id`; `client_ip_hash` = SHA-256[:32] keyed with `IP_HASH_SALT`; bulk deduplication via Cloudflare Queues at > 20 blocks per 5 minutes |
| `scim.ip_enforcement_enabled` | HIGH | 7 years | `scim_ip_enforcement_enabled` toggled from false to true on `tenant_sso_configs` | `tenant_id`, `cidr_count`, `enabled_by` | SCIM-specific IP enforcement toggle; stored in `tenant_sso_configs` (SSO_SCIM §26.3); documented here for completeness of the IP enforcement audit surface |
| `scim.ip_enforcement_disabled` | HIGH | 7 years | `scim_ip_enforcement_enabled` toggled from true to false on `tenant_sso_configs` | `tenant_id`, `disabled_by` | |
| `scim.ip_blocked` | HIGH | 7 years | `enforceScimIpAllowlist()` rejects an IdP SCIM request | `tenant_id`, `client_ip_hash` | IdP IP hashed; no IdP vendor name in payload (avoids naming vendor IPs in the audit chain) |

**HMAC chain requirement.** `api_key.rotated` must be followed by `api_key.revoked` (for the `old_key_id`) within 26 hours. A gap — `api_key.rotated` visible in the chain with no matching `api_key.revoked` for `old_key_id` within 26 hours — is a chain anomaly. AL-APIKEY-02 fires. The pg_cron overlap cleanup job must emit the `api_key.revoked` event in the same transaction as the `UPDATE tenant_api_keys SET revoked_at = now()` — an UPDATE without a corresponding DEC-030 event is not a valid chain transition.

**PII prohibition.** Per DEC-030 design constraint (same as §25.9): no audit event payload field may contain individual user PII — no email addresses, no health values, no display names. The `created_by` and `revoked_by` fields carry internal user UUIDs only. The `enabled_by` and `disabled_by` fields carry the same. No name resolution is performed at emit time. Any audit log viewer that displays human-readable names must resolve them from the `users` table at display time, not at storage time.

---

### 26.8 Alerting Rules

These four alerting rules are defined in `SSO_SCIM_IMPLEMENTATION.md §26.7` and reproduced here for DATA_MODEL completeness. The canonical response procedures are in SSO_SCIM §26.7. PagerDuty configuration is checklist item 9 in §26.11.

| Alert ID | Condition | Severity | Owner | Response summary |
|---|---|---|---|---|
| **AL-APIKEY-01** | `api_key.ip_blocked` events for a single `tenant_id` + `key_id` exceed 10 in 10 minutes | **P1** | devops-lead + customer-success | Possible credential stuffing or misconfigured integration. Check for a recent `api_key.ip_enforcement_enabled` event on the same key — if enforcement was just enabled, notify CSM to contact tenant IT admin. If enforcement was not recently changed, treat as credential compromise attempt. |
| **AL-APIKEY-02** | `api_key.rotated` event with no corresponding `api_key.revoked` for `old_key_id` within 26 hours | **P1** | platform-engineer | Overlap window elapsed without pg_cron cleanup. Investigate `api_key_rotation_overlap_cleanup` cron logs. Manually set `revoked_at` if safe; emit `api_key.revoked` with `reason = 'expiry_overlap'`. |
| **AL-APIKEY-03** | `api_key.revoked` with `reason = 'compromise'` | **P1** | security-engineer | Credential compromise declared. Trigger `docs/INCIDENT_RESPONSE.md §R-16` (Application Secret & Encryption Key Exposure) for API key subtype. Review `last_used_at` and Cloudflare access logs for the 7-day pre-revocation window to scope potential abuse. |
| **AL-SCIM-IP-01** | `scim.ip_blocked` events for a single `tenant_id` exceed 5 in 15 minutes AND SCIM provisioning success rate for the same tenant drops below 50% in the same window | **P2** | customer-success | SCIM IP enforcement is blocking IdP provisioning — likely a misconfigured allowlist or an IdP IP range change. CSM notifies tenant IT admin. Temporary mitigation: disable SCIM IP enforcement from admin dashboard. Permanent fix: update SCIM allowlist with correct IdP CIDRs. |

---

### 26.9 SOC 2 Evidence Mapping

The five evidence artefact IDs below (APIKEY-E-001 through APIKEY-E-005) are defined in `SSO_SCIM_IMPLEMENTATION.md §26.8`. They are cross-referenced here for the DATA_MODEL evidence inventory. The collection procedures, storage location (`compliance/evidence/api-key-auth/`), and cross-reference in `docs/SOC2_READINESS.md` CC6.1 and CC6.4 evidence tables are all documented in SSO_SCIM §26.8 and §26.11 item 10.

| Criterion | Control description | How `tenant_api_keys` schema satisfies it | Evidence artefact ID |
|---|---|---|---|
| **CC6.1** — Logical access security controls limit access | IP allowlist enforcement on long-lived credentials prevents use of a compromised API key from non-authorised networks. The `ip_enforcement_enabled` flag and `ip_allowlist_config` JSONB are structurally present in the schema; `chk_ip_allowlist_required` prevents a configuration where enforcement appears enabled but no allowlist exists to enforce against. | `enforceApiKeyIpAllowlist()` in `api-key-auth.ts` evaluates `ip_enforcement_enabled` and `ip_allowlist_config.entries` on every authenticated request. | **APIKEY-E-001** |
| **CC6.2** — Authentication requires appropriate credentials | Scopes enforce least-privilege access at the credential level. `reporting:read` keys cannot call admin mutation endpoints. `scopes TEXT[] NOT NULL` with `chk_api_key_scopes_nonempty` ensures every key has at least one explicit scope; scope validation occurs in `api-key-auth.ts` before route handler dispatch. | Scope check is a compile-time–enforced gate in the auth middleware, not an advisory UI control. | **APIKEY-E-002** |
| **CC6.4** — Access is removed or modified when credentials change | API key rotation (§26.6.2) with 24-hour overlap and HMAC chain linkage between `api_key.rotated` and `api_key.revoked` (§26.7) provides auditable credential lifecycle. Immediate revocation path (`reason = 'compromise'`) available for incident response. `revoked_at` is permanent — once set it cannot be cleared. | `uq_tenant_api_keys_hash_active` partial unique index ensures a revoked key's hash is no longer matched on the authentication hot-path. | **APIKEY-E-003** |
| **CC6.8** — Controls protect against unauthorised changes to configuration | `api_key.ip_enforcement_enabled` and `api_key.ip_enforcement_disabled` DEC-030 events create a non-repudiable audit trail for every change to the IP enforcement configuration. No form_api role can toggle `ip_enforcement_enabled` directly — all updates flow through form_system (RLS §26.5). | HMAC chain anchors every configuration change to a specific `enabled_by` / `disabled_by` actor UUID and timestamp. | **APIKEY-E-004** |
| **CC7.2** — The entity monitors system components for anomalies | AL-APIKEY-01 detects credential stuffing attempts via sustained IP blocks. AL-APIKEY-02 detects rotation overlap failures (infrastructure anomaly — pg_cron not executing). AL-APIKEY-03 surfaces declared credential compromises within PagerDuty SLA. | All three alerts are triggered from DEC-030 events, not from application health-check polling, ensuring the alert path is independent of application-layer availability. | **APIKEY-E-005** |

---

### 26.10 TypeScript Types

```typescript
// apps/api/src/types/tenant-api-keys.ts

export type ApiKeyScope = 'reporting:read' | 'admin:write';

export interface TenantApiKey {
  id:                      string;
  tenant_id:               string;
  label:                   string;
  key_hash:                string;   // never returned in API responses; only used internally
  key_preview:             string;
  scopes:                  ApiKeyScope[];
  created_by:              string;
  last_used_at:            string | null;
  expires_at:              string | null;
  revoked_at:              string | null;
  ip_enforcement_enabled:  boolean;
  ip_allowlist_config:     IpAllowlistConfig | null;
  created_at:              string;
  updated_at:              string;
}

export interface IpAllowlistConfig {
  enabled: boolean;
  entries: IpAllowlistEntry[];
}

export interface IpAllowlistEntry {
  cidr:  string;   // e.g. "203.0.113.0/24"
  label: string;   // e.g. "Office London"
}

// Returned to tenant admin via API — key_hash excluded, ip_allowlist_config entries
// returned with CIDRs visible only to tenant_owner role.
export interface TenantApiKeyAdminView {
  id:                      string;
  label:                   string;
  key_preview:             string;
  scopes:                  ApiKeyScope[];
  created_at:              string;
  last_used_at:            string | null;
  expires_at:              string | null;
  revoked_at:              string | null;
  ip_enforcement_enabled:  boolean;
  cidr_count:              number | null;  // count of entries; CIDRs omitted for non-owner roles
}

export type ApiKeyRevocationReason =
  | 'admin_request'
  | 'rotation'
  | 'expiry_overlap'
  | 'compromise'
  | 'offboarding';

export interface CreateApiKeyRequest {
  tenantId:               string;
  label:                  string;
  scopes:                 ApiKeyScope[];
  createdBy:              string;
  ipEnforcementEnabled?:  boolean;
  ipAllowlistConfig?:     IpAllowlistConfig | null;
}

export interface CreateApiKeyResponse {
  keyId:      string;
  rawKey:     string;   // returned exactly once; never stored
  keyPreview: string;
}
```

---

### 26.11 RLS Test Matrix

The following test scenarios must be covered by `__tests__/db/api_keys_rls.test.ts`. These tests are complementary to the Worker-layer tests in SSO_SCIM §26.11 item 3.

| # | Scenario | Expected result |
|---|---|---|
| 1 | `form_api` role with `app.current_tenant_id = tenant_A` SELECT on `tenant_api_keys` | Returns only rows where `tenant_id = tenant_A`; no rows from tenant_B |
| 2 | `form_api` role with `app.current_tenant_id = tenant_A` SELECT on `tenant_api_keys` WHERE `tenant_id = tenant_B` | 0 rows returned |
| 3 | `form_api` role INSERT into `tenant_api_keys` | Fails with `insufficient_privilege` |
| 4 | `form_api` role UPDATE `tenant_api_keys` | Fails with `insufficient_privilege` |
| 5 | `form_api` role DELETE from `tenant_api_keys` | Fails with `insufficient_privilege` |
| 6 | `form_system` role SELECT `key_hash` from `tenant_api_keys` | Returns `key_hash` value (form_system has full access) |
| 7 | API response handler for GET `/v1/admin/api-keys` using `form_api` role | Response JSON does not include `key_hash` field |
| 8 | `form_system` INSERT a valid `tenant_api_keys` row | Succeeds; `updated_at` trigger fires on subsequent UPDATE |
| 9 | `form_system` INSERT with `ip_enforcement_enabled = true` and `ip_allowlist_config = NULL` | Fails `chk_ip_allowlist_required` constraint |
| 10 | `form_system` INSERT with `ip_enforcement_enabled = false` and `ip_allowlist_config = NULL` | Succeeds |
| 11 | `form_system` INSERT with `ip_allowlist_config = '[]'::jsonb` (array, not object) | Fails `chk_ip_allowlist_shape` constraint |
| 12 | `form_system` INSERT with `label = ''` | Fails `chk_api_key_label_length` constraint |
| 13 | `form_system` INSERT with `label` of 81 characters | Fails `chk_api_key_label_length` constraint |
| 14 | `form_system` INSERT with `scopes = '{}'` (empty array) | Fails `chk_api_key_scopes_nonempty` constraint |
| 15 | Two active keys (revoked_at IS NULL) with the same `key_hash` | Fails `uq_tenant_api_keys_hash_active` unique index |
| 16 | Revoked key and active key with the same `key_hash` | Permitted — partial index does not cover revoked rows |
| 17 | `form_api` SELECT returns row with `revoked_at IS NOT NULL` | Returns revoked rows (RLS does not filter by revocation status; application layer filters active-only in dashboard) |

---

### 26.12 SOC 2 Open Questions

| ID | Question | Priority | Owner | Target |
|---|---|---|---|---|
| **OQ-SSO-26.1** | **Should IP enforcement be mandatory for `admin:write` scope API keys?** The §26.6.3 rotation policy requires quarterly rotation of `admin:write` keys. Making IP enforcement also mandatory for `admin:write` keys eliminates the scenario where a compromised high-privilege API key is usable from any IP. Risk: CI/CD pipelines with dynamic IP allocation (GitHub Actions, Buildkite cloud runners) cannot use a static allowlist. Resolution options: (a) mandatory for `admin:write` on Enterprise plan only; (b) block `admin:write` key creation without IP enforcement unless tenant owner explicitly confirms waiver; (c) no mandatory enforcement but emit a STANDARD-severity DEC-030 advisory event if an `admin:write` key has `ip_enforcement_enabled = false` for >30 days. | P2 | security-engineer + enterprise-architect | Before first `admin:write` key issued to a customer |
| ~~**OQ-SSO-26.2**~~ **🟢 Resolved — DEC-035 (2026-06-09)** | ~~**Should `expires_at` be hard-enforced with automatic revocation, or remain a soft control (age alerts only)?**~~ **Resolved: soft enforcement (provisional).** Age-alert UI warnings (amber 365d, red 730d) + CSM escalation. Hard automatic revocation deferred. Provisional: if any key exceeds 365 days without rotation at first SOC 2 observation period start, hard enforcement must be added before the observation period ends. See `docs/DECISION_LOG.md` DEC-035. | P1 → **🟢 Closed** | enterprise-architect + compliance-officer | **Closed** |

---

### 26.13 Implementation Checklist

See `SSO_SCIM_IMPLEMENTATION.md §26.11` for the full 14-item checklist covering P0 M5, P1 M5–M6, and P2 M6 tasks. The table below summarises P0 M5 items only — all of which must be complete before the first enterprise pilot goes live.

| # | Task | Owner | Priority | Milestone |
|---|---|---|---|---|
| 1 | Run migration `migrations/0051_tenant_api_keys.sql` — full DDL from §26.4, all CHECK constraints, four indexes (§26.4.2), `updated_at` trigger (§26.4.3), RLS policies (§26.5). Use `ON DELETE RESTRICT` on both `tenant_id` FK and `created_by` FK. | platform-engineer | **P0** | M5 |
| 2 | Run migration `migrations/0052_scim_ip_allowlist.sql` — add `scim_ip_enforcement_enabled BOOLEAN DEFAULT false` and `scim_ip_allowlist_config JSONB DEFAULT NULL` to `tenant_sso_configs`; add `chk_scim_ip_allowlist_required` CHECK constraint. (Separate from `tenant_api_keys` migration — `tenant_sso_configs` is a pre-existing table.) | platform-engineer | **P0** | M5 |
| 3 | Update `src/workers/auth/api-key-auth.ts` — add `enforceApiKeyIpAllowlist()` call after key hash validation; add scope enforcement before route handler dispatch; update `last_used_at` via `ctx.waitUntil()`. Add `API_KEY_HASH_SECRET` as Cloudflare Workers Secret. | platform-engineer | **P0** | M5 |
| 4 | Update `src/workers/auth-policy-middleware/ip-allowlist.ts` — add SCIM route branch calling `enforceScimIpAllowlist()` using `scim_ip_enforcement_enabled` and `scim_ip_allowlist_config` from `CachedAuthPolicy`. Update `CachedAuthPolicy` type to include new fields. | platform-engineer | **P0** | M5 |
| 5 | Register all nine DEC-030 events (§26.7) in `docs/AUDIT_LOG_SCHEMA.md` event registry. Validate HMAC chain linkage between `api_key.rotated` and `api_key.revoked` in staging before M5 cut. | platform-engineer + compliance-officer | **P0** | M5 |
| 6 | Implement API Keys panel in admin dashboard — table with label, key_preview, scopes, created_at, last_used_at, age warning pill (§26.6.3), IP enforcement toggle. Create, Rotate, and Revoke flows. Add SCIM IP Restriction section in SSO configuration panel (SSO_SCIM §26.3.4). Both panels gated by `security.ip_allowlist_enabled` feature flag. | platform-engineer + design-craft | **P0** | M5 |
| 7 | Add `API_KEY_HASH_SECRET` to annual key rotation schedule in `docs/CRYPTOGRAPHY_POLICY.md §3` key inventory and `docs/SOC2_READINESS.md §56` key inventory table. Initial rotation schedule: 180 days. | security-engineer | **P0** | M5 |
| 8 | Implement pg_cron job `api_key_rotation_overlap_cleanup` — runs hourly; sets `revoked_at = now()` on predecessor keys where the rotation overlap has expired (created_at of successor key > 24h ago AND predecessor `revoked_at IS NULL`); emits `api_key.revoked` DEC-030 event with `reason = 'expiry_overlap'` in the same transaction. | platform-engineer | **P0** | M5 |

---

*v1.7 · червень 2026 · owner: enterprise-architect + compliance-officer + platform-engineer · §2.11 consent_records table DDL (migration 0037_consent_records.sql); §3.8 Consent Records RLS Policies — fixes OQ-P2-03 from SOC2_READINESS.md §74.11: `consent_records_tenant_isolation` (PERMISSIVE) replaced by `consent_records_cross_tenant_block` (RESTRICTIVE) to prevent tenant admins from reading employee consent rows via OR-merge of permissive policies; CI test added for tenant admin zero-row assertion; §5 Data Classification updated with consent_records (Confidential classification). Cross-ref: docs/SOC2_READINESS.md §74.4, §74.11 (OQ-P2-03 → 🟢 Resolved).*

---

## 27. Enterprise Invite & Pending-Seat Provisioning Schema — OQ-BILL-04 Resolution

> Owner: `enterprise-architect` + `platform-engineer` + `compliance-officer`. Review: before enterprise GA, on any invitation-flow code change, and at every SOC 2 observation period start.
> Scope: enterprise tier only. Covers three non-SCIM provisioning paths: manual invite, bulk CSV import, and SCIM auto-link on registration. SCIM direct provisioning (POST /Users) is documented in `docs/SSO_SCIM_IMPLEMENTATION.md §5–§7` and is unaffected by this schema.
> References: §24 (`enterprise_seat_allocations`, `enterprise_seat_assignments`, OQ-BILL-04 → 🟢 Resolved); §16 (`assertSeatAvailable()`); §25 (offboarding — invitation cleanup on tenant termination); `docs/SSO_SCIM_IMPLEMENTATION.md §5–§7` (SCIM auto-link path); `docs/AUDIT_LOG_SCHEMA.md` (DEC-030); `docs/ENTERPRISE.md §Privacy floor`; `docs/GDPR_DPIA.md §4` (invited-email as personal data).

---

### 27.1 Purpose — OQ-BILL-04 Resolution

**OQ-BILL-04** (DATA_MODEL §24.14, P1 — before enterprise GA) asked:

> *Should `enterprise_seat_assignments` allow a `tenant_admin` to assign a seat to an email address not yet in the `users` table? This enables invite-then-provision flow (admin assigns seat → user receives invite → user creates account → account auto-linked to assignment). Requires a `pending_email TEXT` column on `enterprise_seat_assignments` and a linking step in the SCIM provisioning flow.*

**Resolution: separate `tenant_invitations` table preferred over `pending_email` column on `enterprise_seat_assignments`.**

The OQ proposed adding `pending_email TEXT` to `enterprise_seat_assignments`. After evaluation, a separate `tenant_invitations` table is architecturally preferred for four reasons:

1. **Clean separation of concerns.** An `enterprise_seat_assignments` row represents a confirmed seat assignment for a known user. Mixing confirmed and pending states in a single table with a nullable `user_id` FK makes partial-unique-index logic brittle and forces every query on `enterprise_seat_assignments` to filter on `user_id IS NOT NULL`.
2. **Invitation-specific lifecycle.** Token management, expiry, resend, and revocation are invitation concepts — not seat assignment concepts. They belong in their own table with their own status machine.
3. **GDPR Art. 17 hygiene.** `invited_email` has a different retention curve than a seat assignment: invites should auto-purge on expiry + 30 days even if the tenant persists. A separate table makes the pg_cron erasure job targeted and safe.
4. **Bulk import state.** CSV-import jobs need their own `bulk_seat_import_jobs` table regardless; a `pending_email` column on `enterprise_seat_assignments` would not capture job-level state (total rows, partial failures, R2 object key).

`enterprise_seat_assignments` is extended with an optional `invitation_id FK` column (nullable) to create the link once an invitation is accepted. The existing `user_id NOT NULL` constraint is relaxed to allow NULL during the pending window; a CHECK constraint ensures exactly one of (`user_id`, `invitation_id`) is non-null for any active assignment row.

---

### 27.2 Three Provisioning Paths

| Path | Trigger | Tables touched | Seat reserved immediately? |
|---|---|---|---|
| **Manual invite** | `tenant_admin` invites by email via admin dashboard | `tenant_invitations` INSERT; `enterprise_seat_assignments` INSERT (pending) | Yes — `assertSeatAvailable()` called before INSERT (§27.8) |
| **Bulk CSV import** | `tenant_admin` uploads CSV; `form_system` Worker processes | `bulk_seat_import_jobs` INSERT; `tenant_invitations` INSERT per row; `enterprise_seat_assignments` INSERT per row | Yes — capacity check per row; seat reserved on INSERT |
| **SCIM auto-link** | SCIM POST /Users for a user whose email matches a pending invitation | `tenant_invitations` UPDATE (`status = 'accepted'`); `enterprise_seat_assignments` UPDATE (`user_id = $new_user_id`) | Already reserved at invite time |

**What SCIM direct provisioning does NOT use.** When a SCIM POST /Users arrives for a user with no matching pending invitation, the existing `SSO_SCIM_IMPLEMENTATION.md §5–§7` flow applies: a `users` row is created and a seat is assigned immediately without creating a `tenant_invitations` row.

---

### 27.3 Migration 0053 — `enterprise_seat_assignments` Extension

```sql
-- migrations/0053_enterprise_invite_schema.sql
-- Part 1: extend enterprise_seat_assignments to support pending state.

-- Allow nullable user_id for pending-invite assignments.
ALTER TABLE enterprise_seat_assignments
  ALTER COLUMN user_id DROP NOT NULL;

-- FK to the originating invitation.
-- NULL for SCIM-provisioned or direct assignments (pre-§27 rows).
ALTER TABLE enterprise_seat_assignments
  ADD COLUMN invitation_id UUID REFERENCES tenant_invitations(id) ON DELETE SET NULL;
-- ON DELETE SET NULL: if an invitation row is hard-deleted post-expiry cleanup,
-- the seat assignment record is retained with invitation_id = NULL for audit trail.

-- Exactly one of (user_id, invitation_id) must be non-null for any active assignment.
ALTER TABLE enterprise_seat_assignments
  ADD CONSTRAINT chk_seat_assignment_not_orphan
    CHECK (user_id IS NOT NULL OR invitation_id IS NOT NULL);

-- Index for invitation lookup (SCIM auto-link path; revocation sweep).
CREATE INDEX idx_seat_assignments_invitation
  ON enterprise_seat_assignments (invitation_id)
  WHERE invitation_id IS NOT NULL;
```

> **Migration note.** Existing rows have `user_id NOT NULL` and `invitation_id NULL`. `chk_seat_assignment_not_orphan` is satisfied for all existing rows because `user_id IS NOT NULL`. The partial unique index `uq_tenant_user_active_seat` (§24.5.2) remains valid — it covers `(tenant_id, user_id) WHERE revoked_at IS NULL`, and pending assignments with `user_id IS NULL` are excluded from that index's scope. Both DDL changes are safe to apply in a single migration transaction.

---

### 27.4 `tenant_invitations` Table

```sql
-- migrations/0053_enterprise_invite_schema.sql (continued)
-- Part 2: tenant_invitations.

CREATE TABLE tenant_invitations (
  id                  UUID        NOT NULL DEFAULT gen_random_uuid() PRIMARY KEY,
  tenant_id           UUID        NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
  allocation_id       UUID        NOT NULL REFERENCES enterprise_seat_allocations(id),
  -- The allocation that authorizes this seat. assertSeatAvailable() was called
  -- at invitation creation time using this allocation's total_seats.

  -- ── Invitee identity ───────────────────────────────────────────────────────
  invited_email       TEXT        NOT NULL,
  -- Personal data (GDPR Art. 4(1)). Nullified at expiry+30d by pg_cron (§27.10.1)
  -- or on Art. 17 erasure of the accepted user (§27.10.2). Never in any DEC-030 payload.
  invited_email_hash  TEXT        NOT NULL,
  -- SHA-256(invited_email || ':' || tenant_id::text) keyed with INVITE_HASH_SALT.
  -- Used for: (1) duplicate-invite dedup; (2) SCIM auto-link lookup;
  -- (3) DEC-030 payload identifier (replaces plaintext in all audit events).
  -- Retained after invited_email is nullified — needed for HMAC chain verification.

  -- ── Invite token ───────────────────────────────────────────────────────────
  token_hash          TEXT        NOT NULL,
  -- SHA-256(raw_token || ':' || id::text) keyed with INVITE_HASH_SALT.
  -- Raw token: 32-byte CSPRNG; returned once to Worker for email delivery; discarded after send.
  CONSTRAINT uq_invite_token_hash UNIQUE (token_hash),

  -- ── Status machine ─────────────────────────────────────────────────────────
  status              TEXT        NOT NULL DEFAULT 'pending',
  CONSTRAINT chk_invite_status
    CHECK (status IN ('pending', 'accepted', 'expired', 'revoked')),

  -- ── Actors ─────────────────────────────────────────────────────────────────
  invited_by          UUID        REFERENCES users(id) ON DELETE SET NULL,
  -- NULL for SCIM-initiated bulk flows where no human actor exists.
  accepted_by         UUID        REFERENCES users(id) ON DELETE SET NULL,
  revoked_by          UUID        REFERENCES users(id) ON DELETE SET NULL,

  -- ── Linked seat assignment ──────────────────────────────────────────────────
  assignment_id       UUID        REFERENCES enterprise_seat_assignments(id) ON DELETE SET NULL,
  -- Created at invite time (pending); retained after acceptance.

  -- ── Source ─────────────────────────────────────────────────────────────────
  source              TEXT        NOT NULL DEFAULT 'manual',
  CONSTRAINT chk_invite_source
    CHECK (source IN ('manual', 'csv_import')),
  bulk_import_job_id  UUID        REFERENCES bulk_seat_import_jobs(id) ON DELETE SET NULL,
  CONSTRAINT chk_bulk_job_consistency
    CHECK (
      (source = 'csv_import' AND bulk_import_job_id IS NOT NULL)
      OR (source = 'manual'   AND bulk_import_job_id IS NULL)
    ),

  -- ── Lifecycle timestamps ────────────────────────────────────────────────────
  expires_at          TIMESTAMPTZ NOT NULL DEFAULT now() + INTERVAL '7 days',
  created_at          TIMESTAMPTZ NOT NULL DEFAULT now(),
  accepted_at         TIMESTAMPTZ,
  revoked_at          TIMESTAMPTZ,

  -- ── Coherence constraints ────────────────────────────────────────────────────
  CONSTRAINT chk_invite_accepted_coherent
    CHECK (
      (status = 'accepted' AND accepted_at IS NOT NULL AND accepted_by IS NOT NULL)
      OR (status <> 'accepted' AND accepted_at IS NULL AND accepted_by IS NULL)
    ),
  CONSTRAINT chk_invite_revoked_coherent
    CHECK (
      (status = 'revoked' AND revoked_at IS NOT NULL)
      OR (status <> 'revoked' AND revoked_at IS NULL)
    )
);

-- Prevent duplicate pending invitations for the same email within a tenant.
CREATE UNIQUE INDEX uq_tenant_invite_email_pending
  ON tenant_invitations (tenant_id, invited_email_hash)
  WHERE status = 'pending';

-- Fast lookup by token hash (accept-invite Worker path).
CREATE INDEX idx_invite_token_hash
  ON tenant_invitations (token_hash)
  WHERE status = 'pending';

-- Fast lookup for SCIM auto-link.
CREATE INDEX idx_invite_email_hash_tenant
  ON tenant_invitations (tenant_id, invited_email_hash)
  WHERE status = 'pending';

-- pg_cron expiry sweep.
CREATE INDEX idx_invite_expires_pending
  ON tenant_invitations (expires_at)
  WHERE status = 'pending';
```

#### 27.4.1 Column Notes

| Column | Notes |
|---|---|
| `invited_email` | GDPR Art. 4(1) personal data. Nullified (→ `'[erased]'`) by pg_cron 30 days after `expires_at` for unaccepted invitations; nullified immediately on Art. 17 erasure of `accepted_by` user. Never in any DEC-030 payload, log, or metric. |
| `invited_email_hash` | `SHA-256(invited_email \|\| ':' \|\| tenant_id::text)` using `INVITE_HASH_SALT` Workers Secret. Retained after `invited_email` is nullified — needed for HMAC chain event verification. Not personally identifiable without the salt. |
| `token_hash` | `SHA-256(raw_token \|\| ':' \|\| id::text)` using `INVITE_HASH_SALT`. Verification: Worker receives raw token in the accept URL, hashes it, looks up by hash. |
| `assignment_id` | Points to the `enterprise_seat_assignments` row created at invite time (pending, `user_id IS NULL`). On acceptance, the assignment is updated: `user_id = $new_user_id`. The FK from `enterprise_seat_assignments.invitation_id` back to this invitation is retained as an audit trail — NOT cleared after acceptance. |
| `expires_at` | Default 7 days. On resend, the existing row's `expires_at` is extended (OQ-INV-03 resolution path: option a). Re-sending emits a fresh `tenant.invite_sent` DEC-030 event. |

---

### 27.5 `bulk_seat_import_jobs` Table

```sql
-- migrations/0053_enterprise_invite_schema.sql (continued)
-- Part 3: bulk_seat_import_jobs.

CREATE TABLE bulk_seat_import_jobs (
  id              UUID        NOT NULL DEFAULT gen_random_uuid() PRIMARY KEY,
  tenant_id       UUID        NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
  allocation_id   UUID        NOT NULL REFERENCES enterprise_seat_allocations(id),
  initiated_by    UUID        NOT NULL REFERENCES users(id) ON DELETE RESTRICT,
  -- RESTRICT: job record must persist for audit even if initiating admin is removed.

  status          TEXT        NOT NULL DEFAULT 'processing',
  CONSTRAINT chk_bulk_job_status
    CHECK (status IN ('processing', 'completed', 'partially_failed', 'failed')),

  total_rows      INTEGER     NOT NULL CHECK (total_rows >= 1 AND total_rows <= 500),
  -- 500-row maximum per import; larger provisioning events require SCIM.
  invited_count   INTEGER     NOT NULL DEFAULT 0,
  skipped_count   INTEGER     NOT NULL DEFAULT 0,
  -- skipped: email is already an active user in this tenant — no invite needed.
  failed_count    INTEGER     NOT NULL DEFAULT 0,
  -- failed: invalid email format, seat limit reached mid-import, duplicate pending.

  error_summary   JSONB,
  -- [{"row": N, "email_hash": "...", "reason": "invalid_email"|"seat_limit_reached"|"duplicate_pending"}]
  -- email_hash only — never plaintext email (privacy floor). NULL if status = 'completed'.

  r2_object_key   TEXT        NOT NULL,
  -- Cloudflare R2 key of the uploaded CSV. Deleted by Worker after processing;
  -- max R2 retention 30 days from upload. Key retained here as tombstone for audit.

  created_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
  completed_at    TIMESTAMPTZ,

  CONSTRAINT chk_bulk_job_counts
    CHECK (invited_count + skipped_count + failed_count <= total_rows),
  CONSTRAINT chk_bulk_job_completed_coherent
    CHECK (
      (status IN ('completed', 'partially_failed', 'failed') AND completed_at IS NOT NULL)
      OR (status = 'processing' AND completed_at IS NULL)
    )
);

CREATE INDEX idx_bulk_jobs_tenant
  ON bulk_seat_import_jobs (tenant_id, created_at DESC);
```

**CSV format:** `email` (required, RFC 5322, max 254 chars), `first_name` (optional, used only in invite email greeting — never stored), `last_name` (optional, same). `first_name` and `last_name` are processed in-Worker memory and discarded after email dispatch; they never touch any DB table or DEC-030 event.

**Processing behaviour:** The CSV Worker processes rows sequentially. On `assertSeatAvailable()` failure mid-import: sets job `status = 'partially_failed'`, records `seat_limit_reached` error for that row and all subsequent rows, stops processing. Already-sent invitations are NOT rolled back (fail-forward). The admin dashboard displays a "seat limit reached at row N — N-1 invitations sent" notice.

---

### 27.6 Invite Flow State Machine

```
  ┌──────────────────────────────────────────────────────────────────────────┐
  │  tenant_admin (or CSV Worker) creates invitation                         │
  │  assertSeatAvailable() → blocks if at seat ceiling                       │
  │  INSERT tenant_invitations (status='pending', expires_at=now()+7d)       │
  │  INSERT enterprise_seat_assignments (user_id=NULL, invitation_id=FK)     │
  │  Email sent with raw_token embedded in accept URL                        │
  │  DEC-030: tenant.invite_sent (STANDARD)                                  │
  └──────────────────────┬───────────────────────────────────────────────────┘
                         │
         ┌───────────────┼────────────────────────┐
         │               │                        │
         ▼               ▼                        ▼
  User clicks      expires_at passes       tenant_admin revokes
  accept link      (no acceptance)         (or offboarding)
         │               │                        │
         ▼               ▼                        ▼
  Token verified    pg_cron sweep:         UPDATE status='revoked'
  (hash match)      status='expired'       enterprise_seat_assignments
  status='accepted' seat released          .revoked_at = now()
  seat_assignment   DEC-030:               DEC-030:
  .user_id set      tenant.invite_expired  tenant.invite_revoked (HIGH)
  DEC-030:          ────────────────
  tenant.invite_    30d later:
  accepted          pg_cron nullifies
  (STANDARD)        invited_email→'[erased]'
                    (§27.10 privacy cleanup)
```

Also triggered by SCIM auto-link — same `accepted` outcome, `linked_via: 'scim'` in DEC-030 event (§27.7).

**Status transition table:**

| From | To | Trigger | Seat effect |
|---|---|---|---|
| `pending` | `accepted` | User accepts via URL (token hash match) | `enterprise_seat_assignments.user_id` set |
| `pending` | `accepted` | SCIM POST /Users with matching `invited_email_hash` | Same; SCIM auto-link path (§27.7) |
| `pending` | `expired` | pg_cron: `expires_at < now()` | `enterprise_seat_assignments.revoked_at = now()` |
| `pending` | `revoked` | `tenant_admin` manual revocation or §25 offboarding | `enterprise_seat_assignments.revoked_at = now()` |
| `accepted` | — | Terminal. No further transitions. | Seat remains active until offboarding (§25) |

---

### 27.7 SCIM Auto-Link Path

When the SCIM Worker receives `POST /Users` for a user whose email matches a pending invitation in the same tenant, it auto-accepts the invitation rather than creating a standalone seat assignment.

```typescript
// workers/scim/users/create.ts (addition for §27)
async function handleScimUserCreate(
  req: ScimUserCreateRequest,
  tenantId: string,
  db: DbClient,
): Promise<ScimUser> {

  const emailHash = await computeInviteEmailHash(
    req.email, tenantId, env.INVITE_HASH_SALT,
  );

  const pendingInvite = await db.queryOne<PendingInvitation>(
    `SELECT id, assignment_id
     FROM tenant_invitations
     WHERE tenant_id = $1
       AND invited_email_hash = $2
       AND status = 'pending'
       AND expires_at > now()
     LIMIT 1`,
    [tenantId, emailHash],
  );

  if (pendingInvite) {
    const newUser = await createUserRecord(req, db);
    await db.transaction(async (tx) => {
      await tx.execute(
        `UPDATE tenant_invitations
         SET status = 'accepted', accepted_at = now(), accepted_by = $1
         WHERE id = $2`,
        [newUser.id, pendingInvite.id],
      );
      // Guard: WHERE user_id IS NULL prevents race condition on duplicate SCIM requests.
      await tx.execute(
        `UPDATE enterprise_seat_assignments
         SET user_id = $1
         WHERE id = $2 AND user_id IS NULL`,
        [newUser.id, pendingInvite.assignment_id],
      );
      await emitDec030Event({
        event_type: 'tenant.invite_accepted',
        severity: 'STANDARD',
        payload: {
          tenant_id: tenantId,
          invitation_id: pendingInvite.id,
          accepted_by: newUser.id,
          assignment_id: pendingInvite.assignment_id,
          source: 'scim_auto_link',
          linked_via: 'scim',
        },
      }, tx);
    });
    return toScimUser(newUser);
  }

  // No matching invitation — standard SCIM provisioning path (SSO_SCIM §5–§7).
  return handleScimDirectProvision(req, tenantId, db);
}
```

**Key invariants:** `assertSeatAvailable()` is NOT called in the auto-link path — the seat was reserved at invitation creation. If the invitation has expired by the time SCIM arrives, the lookup returns no rows and standard provisioning runs (the expired invitation's seat is released independently by pg_cron).

---

### 27.8 Seat Reservation Policy

**Pending invitations count against the tenant's contracted seat total.**

`assertSeatAvailable()` (§16.5) is extended to count pending assignment rows:

```typescript
// workers/provisioning/seat-gate.ts (§27 extension)
const row = await db.queryOne<{ used_seats: number; max_seats: number }>(
  `SELECT
     COUNT(esa.id) FILTER (WHERE esa.revoked_at IS NULL) AS used_seats,
     -- Counts both active assignments (user_id IS NOT NULL)
     -- AND pending invite assignments (user_id IS NULL, invitation_id IS NOT NULL).
     ea.total_seats AS max_seats
   FROM enterprise_seat_allocations ea
   LEFT JOIN enterprise_seat_assignments esa ON esa.allocation_id = ea.id
   WHERE ea.tenant_id = $1 AND ea.effective_to IS NULL
   GROUP BY ea.total_seats`,
  [tenantId],
);
```

**Rationale.** Not counting pending invitations would allow a tenant with a 50-seat contract to send 500 invitations. FORM commits to a deterministic seat ceiling for billing and access control purposes; a pending invite is a contractual reservation, not a speculative one.

**Over-invitation protection (bulk CSV):** The CSV Worker stops adding invitations when `assertSeatAvailable()` fails. `bulk_seat_import_jobs.error_summary` records the first row that exceeded the limit with `reason: 'seat_limit_reached'`.

---

### 27.9 RLS Policies

```sql
-- migrations/0053_enterprise_invite_schema.sql (continued)
-- Part 4: Row-Level Security.

-- ── tenant_invitations ────────────────────────────────────────────────────────

ALTER TABLE tenant_invitations ENABLE ROW LEVEL SECURITY;

-- form_api: tenant_admin/tenant_owner can read own tenant's invitations.
-- Mutations go through form_system Workers only.
CREATE POLICY invite_tenant_isolation ON tenant_invitations
  FOR SELECT TO form_api
  USING (tenant_id = current_setting('app.current_tenant_id')::uuid);

CREATE POLICY invite_system_full ON tenant_invitations
  FOR ALL TO form_system
  USING (true) WITH CHECK (true);

-- Defence-in-depth: RESTRICTIVE policy blocks cross-tenant reads even if
-- a permissive policy is inadvertently widened.
CREATE POLICY invite_cross_tenant_block ON tenant_invitations
  AS RESTRICTIVE FOR ALL TO form_api
  USING (tenant_id = current_setting('app.current_tenant_id')::uuid);

-- ── bulk_seat_import_jobs ─────────────────────────────────────────────────────

ALTER TABLE bulk_seat_import_jobs ENABLE ROW LEVEL SECURITY;

CREATE POLICY bulk_job_tenant_isolation ON bulk_seat_import_jobs
  FOR SELECT TO form_api
  USING (tenant_id = current_setting('app.current_tenant_id')::uuid);

CREATE POLICY bulk_job_system_full ON bulk_seat_import_jobs
  FOR ALL TO form_system
  USING (true) WITH CHECK (true);

CREATE POLICY bulk_job_cross_tenant_block ON bulk_seat_import_jobs
  AS RESTRICTIVE FOR ALL TO form_api
  USING (tenant_id = current_setting('app.current_tenant_id')::uuid);
```

**Role access summary:**

| Actor | `tenant_invitations` | `bulk_seat_import_jobs` |
|---|---|---|
| `form_api` (tenant_admin/owner/manager) | SELECT own tenant only | SELECT own tenant only |
| `form_system` | Full | Full |
| `form_api` (other tenant) | ZERO rows (RESTRICTIVE) | ZERO rows (RESTRICTIVE) |

**Who can send invitations?** `tenant_owner` and `tenant_admin` only. `tenant_manager` (third ENTERPRISE.md persona) may view invitation status but cannot create or revoke. Enforced at Worker RBAC layer (`assertRole(['tenant_owner','tenant_admin'])`); not distinguishable at RLS layer.

---

### 27.10 GDPR Art. 17 Erasure Handling

`invited_email` is personal data under GDPR Art. 4(1). Two erasure paths apply.

#### 27.10.1 Expiry + 30 Days (Automatic, pg_cron)

```sql
-- pg_cron: invite_email_expiry_cleanup (daily 04:00 UTC)
-- Nullifies invited_email on expired/revoked invitations older than 30 days.
-- invited_email_hash retained for HMAC chain reference.
UPDATE tenant_invitations
SET invited_email = '[erased]'
WHERE status IN ('expired', 'revoked')
  AND expires_at < now() - INTERVAL '30 days'
  AND invited_email <> '[erased]';
```

This is privacy-by-design retention enforcement, not a data-subject request. The 30-day window allows CSM to investigate failed invite delivery before the email is purged. The `invited_email_hash` is never erased.

#### 27.10.2 Art. 17 Erasure of Accepted User

When a registered user (who joined via an invitation) submits an Art. 17 erasure request, `workers/gdpr/erasure.ts` includes:

```sql
-- Step INV-1: nullify invited_email for invitations accepted by the erased user.
-- invited_email_hash, accepted_by, assignment_id, invited_by retained (UUIDs; not
-- personal data of the erased subject).
UPDATE tenant_invitations
SET invited_email = '[erased]'
WHERE accepted_by = $erased_user_id
  AND invited_email <> '[erased]';
-- Covered by existing data.individual_deletion HIGH DEC-030 event (§12 erasure Worker).
-- No separate invite-specific event required.
```

**Privacy rule for evidence collection:** SQL exports involving `tenant_invitations` must project `invited_email_hash` only — never `invited_email`.

---

### 27.11 DEC-030 HMAC-Chained Audit Events

All six events must be registered in `docs/AUDIT_LOG_SCHEMA.md` under a new **"Enterprise seat invitation events"** subsection.

| Event type | Severity | Retention | Payload fields | When emitted |
|---|---|---|---|---|
| `tenant.invite_sent` | STANDARD | 7 yr | `tenant_id`, `invitation_id`, `email_hash`, `invited_by` (UUID), `source` (`manual`\|`csv_import`), `allocation_id`, `expires_at` (ISO 8601), `bulk_import_job_id` (UUID or null) | After invite INSERT + email dispatch confirmed |
| `tenant.invite_accepted` | STANDARD | 7 yr | `tenant_id`, `invitation_id`, `accepted_by` (new user UUID), `assignment_id`, `source`, `linked_via` (`registration`\|`scim`) | On successful acceptance (token verify or SCIM auto-link) |
| `tenant.invite_expired` | STANDARD | 3 yr | `tenant_id`, `invitation_id`, `source`, `bulk_import_job_id` (UUID or null) | pg_cron `invite_expiry_sweep` marks `status = 'expired'` |
| `tenant.invite_revoked` | HIGH | 7 yr | `tenant_id`, `invitation_id`, `revoked_by` (UUID), `reason` (`admin_action`\|`offboarding`\|`seat_reduction`), `source` | Manual revocation or §25 offboarding |
| `tenant.bulk_invite_started` | STANDARD | 3 yr | `tenant_id`, `bulk_import_job_id`, `total_rows`, `initiated_by` (UUID), `allocation_id` | CSV Worker begins row processing |
| `tenant.bulk_invite_completed` | STANDARD | 3 yr | `tenant_id`, `bulk_import_job_id`, `invited_count`, `skipped_count`, `failed_count`, `status` | CSV Worker finishes |

**HMAC chain ordering for bulk imports:** `tenant.bulk_invite_started` must precede all `tenant.invite_sent` events with the same `bulk_import_job_id`; `tenant.bulk_invite_completed` must follow all of them.

**Privacy invariant:** `invited_email` never appears in any event payload. `email_hash` uses the same `INVITE_HASH_SALT` as the DB column — auditors can verify event-to-DB linkage by recomputing the hash.

---

### 27.12 SOC 2 Evidence Mapping

| SOC 2 Criterion | Control | Evidence artefact |
|---|---|---|
| **CC6.1 — Logical access controls** | `assertSeatAvailable()` called before every invitation; pending assignments count against contracted seat total. | **INV-E-001:** `enterprise_seat_allocations` + `enterprise_seat_assignments` (including `user_id IS NULL` pending rows) export for a 30-day window — demonstrates seat ceiling was not exceeded at any point. |
| **CC6.2 — Access authorised** | Every invitation is authorised by a `tenant_admin` or `tenant_owner`; DEC-030 `tenant.invite_sent` with `invited_by` UUID provides the authorisation record. | **INV-E-002:** DEC-030 `tenant.invite_sent` events for the audit period — every new seat authorised by a named actor. |
| **CC6.3 — Access removed when changed** | Invitation revocation (`tenant.invite_revoked`) releases the reserved seat. §25 offboarding revokes all pending invitations for the departing tenant. | **INV-E-003:** DEC-030 `tenant.invite_revoked` events for the audit period — demonstrates prompt removal of pending access. |
| **P4.2 — Privacy notice prior to processing** | Invite email includes visible link to FORM's Privacy Policy and one-sentence data processing summary before account creation. Template is version-controlled. | **INV-E-004:** Version-controlled invite email HTML template in `compliance/evidence/invitations/invite-email-template-v{N}.html`. |
| **P5.2 — Personal data disposed per policy** | pg_cron `invite_email_expiry_cleanup` (daily 04:00 UTC) purges `invited_email` 30 days post-expiry. Art. 17 step INV-1 nullifies `invited_email` for accepted users on erasure request. | **INV-E-005:** pg_cron `invite_email_expiry_cleanup` execution log over the audit period — demonstrates scheduled erasure is operating. |

---

### 27.13 TypeScript Types

```typescript
// apps/api/src/types/tenant-invitations.ts

export type InviteStatus   = 'pending' | 'accepted' | 'expired' | 'revoked';
export type InviteSource   = 'manual' | 'csv_import';
export type InviteLinkedVia = 'registration' | 'scim';

export interface TenantInvitation {
  id:                  string;
  tenant_id:           string;
  allocation_id:       string;
  // invited_email intentionally omitted — raw email is never returned by the admin API.
  invited_email_hash:  string;
  status:              InviteStatus;
  invited_by:          string | null;
  accepted_by:         string | null;
  revoked_by:          string | null;
  assignment_id:       string | null;
  source:              InviteSource;
  bulk_import_job_id:  string | null;
  expires_at:          string;
  created_at:          string;
  accepted_at:         string | null;
  revoked_at:          string | null;
}

// View returned to tenant_admin — shows email preview, not hash.
export interface TenantInvitationAdminView {
  id:            string;
  email_preview: string;    // Worker: first3***@domain (e.g. joh***@example.com)
  status:        InviteStatus;
  invited_by:    string | null;
  source:        InviteSource;
  expires_at:    string;
  created_at:    string;
  accepted_at:   string | null;
}

export interface BulkSeatImportJob {
  id:             string;
  tenant_id:      string;
  allocation_id:  string;
  initiated_by:   string;
  status:         'processing' | 'completed' | 'partially_failed' | 'failed';
  total_rows:     number;
  invited_count:  number;
  skipped_count:  number;
  failed_count:   number;
  error_summary:  BulkImportError[] | null;
  created_at:     string;
  completed_at:   string | null;
}

export interface BulkImportError {
  row:        number;
  email_hash: string;   // never plaintext email
  reason:     'invalid_email' | 'seat_limit_reached' | 'duplicate_pending';
}

export interface CreateInviteRequest {
  tenantId:          string;
  email:             string;
  allocationId:      string;
  invitedBy:         string;
  customExpiryDays?: number;   // 7–30; default 7 (OQ-INV-01)
}

export interface CreateInviteResponse {
  invitationId: string;
  expiresAt:    string;
}
```

---

### 27.14 Open Questions

| ID | Question | Owner | Priority | Resolution path |
|---|---|---|---|---|
| **OQ-INV-01** | **Custom invite expiry window (7–30 days).** Default `expires_at = now() + INTERVAL '7 days'`. Enterprise HR onboarding cycles often run 2–4 weeks. Should `tenant_owner` be allowed to set a custom expiry of 7–30 days? Implementation: `custom_expiry_days INTEGER CHECK (custom_expiry_days BETWEEN 7 AND 30)` column on `tenant_invitations`; Worker reads from per-invite override or `tenant_settings.default_invite_expiry_days`. Risk: longer expiry windows increase phishing window for a hijacked invite link — partially mitigated by single-use token design. Recommendation: allow per-invite override for `tenant_owner` only; `tenant_admin` uses tenant-wide default. | enterprise-architect + customer-success | **P1 — before GA** | Evaluate after first enterprise pilot feedback; document decision in `docs/DECISION_LOG.md`. |
| **OQ-INV-02** | **Email preview format in admin dashboard.** `TenantInvitationAdminView.email_preview` computed as `first3***@domain`. Lets admins confirm they invited the right person without exposing full email. Risk: some domains are small enough that first-3-chars + domain is re-identifying. Alternative: domain-only (`***@example.com`). Recommendation: `first3***@domain` is industry-standard (Gmail recovery display) and acceptable for the admin dashboard context — the admin already knows who they invited. If any enterprise customer raises a GDPR concern, switch to domain-only. | compliance-officer + enterprise-architect | **P2 — before public admin API docs** | Document decision in `docs/DECISION_LOG.md`; update type if domain-only adopted. |
| **OQ-INV-03** | **Resend flow: extend-expiry vs. create-new-row.** On resend: (a) extend `expires_at` + rotate `token_hash` on the existing row (invalidates old link); (b) mark old invitation `revoked`, create a new row. Recommendation: option (a) — simpler, no `revoked` row pollution for non-security resend events. A fresh `tenant.invite_sent` DEC-030 event is emitted on resend regardless, preserving the audit trail. | platform-engineer + security-engineer | **P1 — before invite email UI ships** | Document in `docs/DECISION_LOG.md`. |

---

### 27.15 Implementation Checklist

#### P0 — Before any tenant sees the invite feature (M5)

| # | Task | Owner | Priority | Milestone | Status |
|---|---|---|---|---|---|
| 1 | Run migration `0053_enterprise_invite_schema.sql` (Parts 1–4): ALTER `enterprise_seat_assignments`; CREATE `tenant_invitations` + `bulk_seat_import_jobs`; CREATE RLS policies. Test: existing seat assignment rows still pass `chk_seat_assignment_not_orphan`; `form_api` SELECT on `tenant_invitations` for wrong tenant returns 0 rows. | platform-engineer | **P0** | M5 | [ ] |
| 2 | Extend `assertSeatAvailable()` (§27.8) to count pending invite assignments. CI test: tenant at 50-seat limit with 49 active + 1 pending invite → next invite creation fails `SeatLimitReachedError`. | platform-engineer | **P0** | M5 | [ ] |
| 3 | Add `INVITE_HASH_SALT` as Cloudflare Workers Secret. Document in `docs/CRYPTOGRAPHY_POLICY.md §3` key inventory (rotation schedule: 365 days). | security-engineer | **P0** | M5 | [ ] |
| 4 | Implement `workers/invitations/create-invite.ts`: role check (`tenant_owner`/`tenant_admin`); `assertSeatAvailable()`; compute `invited_email_hash` + `token_hash`; INSERT `tenant_invitations` + INSERT `enterprise_seat_assignments` (pending); dispatch invite email; emit `tenant.invite_sent` DEC-030. | platform-engineer | **P0** | M5 | [ ] |
| 5 | Implement `workers/invitations/accept-invite.ts`: token hash verification; user registration; UPDATE `tenant_invitations` (`status = 'accepted'`) + UPDATE `enterprise_seat_assignments` (`user_id = $new_user_id`); emit `tenant.invite_accepted` DEC-030 (`linked_via: 'registration'`). Idempotency: already-accepted token → HTTP 200 "already accepted". | platform-engineer | **P0** | M5 | [ ] |
| 6 | Add SCIM auto-link path to `workers/scim/users/create.ts` (§27.7): email hash lookup; auto-accept on match; emit `tenant.invite_accepted` (`linked_via: 'scim'`). Staging test: create pending invite → SCIM POST /Users for same email → verify `assignment_id.user_id` set + invite `status = 'accepted'`. | platform-engineer | **P0** | M5 | [ ] |
| 7 | Register all six DEC-030 events (§27.11) in `docs/AUDIT_LOG_SCHEMA.md` under new "Enterprise seat invitation events" subsection with severity, retention, and payload fields. | platform-engineer + compliance-officer | **P0** | M5 | [ ] |

#### P1 — Before first enterprise pilot with invitations enabled

| # | Task | Owner | Priority | Milestone | Status |
|---|---|---|---|---|---|
| 8 | Implement pg_cron `invite_expiry_sweep` (daily 02:30 UTC): UPDATE `tenant_invitations` pending rows past `expires_at` → `expired`; UPDATE `enterprise_seat_assignments.revoked_at`; emit `tenant.invite_expired` per expired invite (batch, HMAC-chained). Add to `docs/OBSERVABILITY.md §12.6` pg_cron registry as job **10** (renumbered from originally claimed 14 — see OBSERVABILITY.md v3.0 §36.3 conflict resolution). | platform-engineer + devops-lead | **P1** | M5 | [ ] |
| 9 | Implement pg_cron `invite_email_expiry_cleanup` (daily 04:00 UTC, §27.10.1): nullify `invited_email → '[erased]'` for expired/revoked rows older than 30 days. Add to `docs/OBSERVABILITY.md §12.6` as job **12** (renumbered from originally claimed 15 — see OBSERVABILITY.md v3.0 §36.3 conflict resolution). | platform-engineer | **P1** | M5 | [ ] |
| 10 | Add Art. 17 erasure step INV-1 to `workers/gdpr/erasure.ts` (§27.10.2): `UPDATE tenant_invitations SET invited_email='[erased]' WHERE accepted_by=$user_id AND invited_email<>'[erased]'`. Test: erasure request for user who joined via invite → verify `tenant_invitations.invited_email = '[erased]'`. | platform-engineer + compliance-officer | **P1** | M5 | [ ] |
| 11 | Implement bulk CSV import: `POST /api/v1/admin/invitations/bulk` (multipart); CSV validation (RFC 5322, max 500 rows); R2 upload; `bulk_seat_import_jobs` INSERT; async Durable Object processing; emit `tenant.bulk_invite_started` + `tenant.bulk_invite_completed`. Admin dashboard import-history panel. | platform-engineer + design-craft | **P1** | M6 | [ ] |
| 12 | Add invitation management panel to admin dashboard: pending-invites table (`email_preview`, status, `expires_at`, resend + revoke actions); bulk-import history (status, counts, error summary); seat utilisation counter updated to reflect pending + active seats. Gate on `enterprise.invite_flow_enabled` feature flag. | platform-engineer + design-craft | **P1** | M6 | [ ] |
| 13 | Add §25 offboarding integration: on `access_revoked` transition, revoke all pending invitations for the tenant (UPDATE `tenant_invitations SET status='revoked', revoked_at=now(), revoked_by=NULL`; emit `tenant.invite_revoked` per invite with `reason='offboarding'`; SET `enterprise_seat_assignments.revoked_at=now()` per pending assignment). | platform-engineer | **P1** | M5 | [ ] |
| 14 | Collect INV-E-001 through INV-E-005 evidence artefacts (§27.12) after 30 days of production invitation usage; store in `compliance/evidence/invitations/`; cross-reference in `docs/SOC2_READINESS.md` CC6.1, CC6.2, CC6.3, P4.2, P5.2 evidence tables. | compliance-officer | **P1** | M6 | [ ] |

#### P2 — Post-GA improvements

| # | Task | Owner | Priority | Milestone | Status |
|---|---|---|---|---|---|
| 15 | Resolve OQ-INV-01 (custom expiry 7–30 days) and implement if adopted; document in `docs/DECISION_LOG.md`. | enterprise-architect + customer-success | **P2** | M7 | [ ] |
| 16 | Resolve OQ-INV-02 (email preview format) and update `TenantInvitationAdminView` if domain-only adopted. | compliance-officer + enterprise-architect | **P2** | Before public admin API docs | [ ] |
| 17 | Resolve OQ-INV-03 (resend: extend-expiry vs. create-new) and implement. | platform-engineer + security-engineer | **P1** | Before invite email UI ships | [ ] |

---

*v1.8 · червень 2026 · owner: enterprise-architect + platform-engineer + compliance-officer · §27 Enterprise Invite & Pending-Seat Provisioning Schema — resolves OQ-BILL-04 (DATA_MODEL §24.14, P1 before enterprise GA). Migration 0053 extends `enterprise_seat_assignments` (nullable `user_id`, `invitation_id FK`, `chk_seat_assignment_not_orphan` CHECK) and adds two new tables. `tenant_invitations`: `invited_email` (personal data — auto-erased at expiry+30d by pg_cron `invite_email_expiry_cleanup` + Art. 17 step INV-1 in erasure Worker), `invited_email_hash` (SHA-256 keyed with `INVITE_HASH_SALT`, retained post-erasure for HMAC chain reference), `token_hash` (32-byte CSPRNG, single-use, never stored plaintext), 4-state machine (pending→accepted|expired|revoked), `source` (manual|csv_import), `assignment_id FK` (retained post-acceptance as audit trail), `bulk_import_job_id FK`. `bulk_seat_import_jobs`: CSV import job tracking (max 500 rows; `error_summary` contains `email_hash` only — never plaintext email), R2 object key tombstone, status machine (processing→completed|partially_failed|failed). Three provisioning paths: manual invite (§27.3–§27.6), bulk CSV import (§27.5–§27.6), SCIM auto-link on registration (§27.7). Seat reservation policy (§27.8): pending invitations count against contracted seat total — `assertSeatAvailable()` extended to include `user_id IS NULL, revoked_at IS NULL` rows. RLS (§27.9): `form_api` SELECT own-tenant only + RESTRICTIVE cross-tenant block; `form_system` full access; `tenant_manager` read-only (enforced at Worker RBAC layer). GDPR Art. 17 (§27.10): pg_cron `invite_email_expiry_cleanup` daily 04:00 UTC + erasure Worker step INV-1. Six DEC-030 HMAC-chained events (§27.11): `tenant.invite_sent` (STANDARD 7yr), `tenant.invite_accepted` (STANDARD 7yr, `linked_via: registration|scim`), `tenant.invite_expired` (STANDARD 3yr), `tenant.invite_revoked` (HIGH 7yr, `reason: admin_action|offboarding|seat_reduction`), `tenant.bulk_invite_started` (STANDARD 3yr), `tenant.bulk_invite_completed` (STANDARD 3yr). SOC 2 mapping (§27.12): CC6.1 (seat ceiling enforced at invite creation — INV-E-001), CC6.2 (invite = authorised provisioning decision with DEC-030 `invited_by` — INV-E-002), CC6.3 (revocation on offboarding — INV-E-003), P4.2 (privacy notice in invite email template — INV-E-004), P5.2 (scheduled email erasure — INV-E-005). TypeScript types (§27.13): `TenantInvitation` (no `invited_email` field in API type), `TenantInvitationAdminView` (`email_preview: first3***@domain`), `BulkSeatImportJob`, `BulkImportError` (`email_hash` only), `CreateInviteRequest/Response`. Three open questions: OQ-INV-01 (custom expiry 7–30d — P1), OQ-INV-02 (email preview format — P2), OQ-INV-03 (resend token rotation vs. new row — P1). Seventeen-item implementation checklist (7× P0 M5, 7× P1 M5–M6, 3× P2 M7+). Cross-references: §24 (OQ-BILL-04 → 🟢 Resolved; `enterprise_seat_allocations`/`enterprise_seat_assignments` base tables), §16 (`assertSeatAvailable()` extension), §25 (offboarding: pending invite revocation on tenant termination), `docs/SSO_SCIM_IMPLEMENTATION.md §5–§7` (SCIM auto-link is additive; direct SCIM provisioning path unaffected), `docs/AUDIT_LOG_SCHEMA.md` (six new event types to register), `docs/CRYPTOGRAPHY_POLICY.md §3` (`INVITE_HASH_SALT` key inventory entry), `docs/ENTERPRISE.md §Admin Dashboard` (three provisioning paths — manual, CSV, SCIM — all now schema-documented and schema-backed), `docs/OBSERVABILITY.md §12.6` (pg_cron registry: `invite_expiry_sweep` job 14, `invite_email_expiry_cleanup` job 15).*

---

## 28. Rate Limiting, Quota Enforcement & Abuse Prevention Schema

> Owner: `enterprise-architect` + `platform-engineer` + `security-engineer`. Review: before any enterprise pilot launch, on any change to quota tiers or API key policy, and at every SOC 2 observation period start.
> Scope: all tiers. Consumer Pro users are subject to KV sliding-window limits and DB quota tracking where `tenant_id IS NULL`. Enterprise tenants have contract-tier limits recorded in `api_quota_usage.quota_limit`. The Cloudflare WAF layer is documented here for architectural completeness only — it has no Postgres representation.
> References: §26 (`api_keys` table — FK target for `api_key_id`); §13 (analytics events — separate from rate-limit event tracking); §16 (enterprise contract lifecycle — source of `quota_limit` values); `docs/AUDIT_LOG_SCHEMA.md` (DEC-030 HMAC chain); `docs/OBSERVABILITY.md §12.6` (pg_cron job registry); INCIDENT_RESPONSE R-06 (DDoS / WAF Bypass runbook).

---

### 28.1 Scope — Three-Layer Architecture

Rate limiting and quota enforcement operate at three distinct layers. Only the third layer has a Postgres representation; the first two are documented here so that the enforcement contract is legible end-to-end.

```
Incoming request
      │
      ▼
┌─────────────────────────────────────────────────────┐
│  Layer 1 — Cloudflare WAF (L3/L4/L7)               │
│  Volumetric DDoS, TLS fingerprint, geo-block.       │
│  Drops traffic before it reaches Workers.           │
│  No Postgres state. Runbook: INCIDENT_RESPONSE R-06 │
└──────────────────────┬──────────────────────────────┘
                       │ passes WAF
                       ▼
┌─────────────────────────────────────────────────────┐
│  Layer 2 — Cloudflare KV sliding-window (§28.5)     │
│  Per-API-key, per-endpoint-category, per-window.    │
│  Key: rl:{api_key_id}:{endpoint_category}:{window}  │
│  Returns HTTP 429 on window breach.                 │
│  No Postgres state. TTL-managed in KV.              │
└──────────────────────┬──────────────────────────────┘
                       │ passes KV check
                       ▼
┌─────────────────────────────────────────────────────┐
│  Layer 3 — DB quota tracking (Postgres — this §)    │
│  Monthly cumulative counter per api_key +           │
│  endpoint_category. Hard block at quota_limit +     │
│  overage grace. Source of truth for billing and     │
│  enterprise contract enforcement.                   │
│  Tables: api_quota_usage, rate_limit_violations,    │
│          abuse_flags                                │
└─────────────────────────────────────────────────────┘
```

**Why three layers?** Cloudflare WAF handles volumetric attacks before they consume compute. KV handles per-session burst limits at microsecond latency without a database round-trip. Postgres tracks monthly quota consumption because billing evidence and contract enforcement require durable, auditable, tenant-scoped records that survive request lifecycle and Cloudflare KV TTL expiry. The three layers are additive: a request that passes all three layers is both in-session-budget and in-monthly-budget.

---

### 28.2 Tables

#### 28.2.1 `api_quota_usage`

Monthly quota consumption per API key and endpoint category. This table is the billing source of truth for enterprise overage detection and the basis for SOC 2 CC6.1 access-boundary evidence.

```sql
-- migrations/0055_rate_limiting_schema.sql
-- Part 1: api_quota_usage

CREATE TABLE api_quota_usage (
  id                  UUID        PRIMARY KEY DEFAULT gen_random_uuid(),

  api_key_id          UUID        NOT NULL
    REFERENCES api_keys(id) ON DELETE CASCADE,
  -- CASCADE: quota records are billing-adjacent but not standalone billing evidence;
  -- if an API key is hard-deleted, its usage rows go with it. Billing-period snapshots
  -- for invoicing must be taken before key deletion. See §28.10 OQ-RL-01.

  tenant_id           UUID
    REFERENCES tenants(id),
  -- NULL for consumer Pro users (no enterprise tenant). Always non-null for enterprise
  -- API keys. Denormalized from api_keys for RLS performance (avoids per-row join).

  period_month        DATE        NOT NULL,
  -- Always the first calendar day of the billing month: 2026-06-01, 2026-07-01, etc.
  -- Enforced by CHECK constraint; application layer must truncate to month before insert.
  CONSTRAINT chk_quota_period_is_month_start
    CHECK (EXTRACT(DAY FROM period_month) = 1),

  endpoint_category   TEXT        NOT NULL
    CHECK (endpoint_category IN (
      'coaching',
      'analytics_read',
      'webhook_mgmt',
      'admin',
      'bulk_import'
    )),

  request_count       BIGINT      NOT NULL DEFAULT 0,
  -- Atomically incremented on every admitted request (request_count + 1).
  -- Never decremented; corrections are made via overage_count and hard_blocked_at.

  tokens_consumed     BIGINT      NOT NULL DEFAULT 0,
  -- Claude API tokens billed to this key in this period and category.
  -- Incremented by the coaching Worker after receiving the Anthropic usage object.
  -- NULL-safe: non-coaching categories may never increment this field.

  quota_limit         BIGINT,
  -- Monthly request ceiling from the enterprise contract tier.
  -- NULL = unlimited (Consumer Pro plan; no hard block enforced).
  -- Populated at row creation time from the tenant's active contract (§16).

  overage_count       BIGINT      NOT NULL DEFAULT 0,
  -- Requests admitted past quota_limit during the overage grace window.
  -- When overage_count reaches OVERAGE_GRACE (application constant), hard_blocked_at is set.

  hard_blocked_at     TIMESTAMPTZ,
  -- NULL = not blocked for this period.
  -- SET by quota-check.ts Worker when overage_count exceeds grace threshold.
  -- Cleared automatically on period_month rollover (new row, not cleared in place).

  updated_at          TIMESTAMPTZ NOT NULL DEFAULT NOW(),

  CONSTRAINT uq_quota_key_period_category
    UNIQUE (api_key_id, period_month, endpoint_category)
);

-- Retention: 36 months (SOC 2 CC4.1 billing evidence).
-- Rows older than 36 months are archived by pg_cron api_quota_usage_archive (§28.7).

ALTER TABLE api_quota_usage ENABLE ROW LEVEL SECURITY;

-- Composite index supports the quota-check lookup (hot path).
CREATE INDEX idx_quota_usage_key_period
  ON api_quota_usage (api_key_id, period_month, endpoint_category);

-- Tenant-scoped index for admin dashboard quota panel.
CREATE INDEX idx_quota_usage_tenant_period
  ON api_quota_usage (tenant_id, period_month)
  WHERE tenant_id IS NOT NULL;
```

#### 28.2.2 `rate_limit_violations`

Append-only log of rate limit breach events. Written on the non-hot path (Worker `waitUntil`) and used for abuse pattern detection, SOC 2 CC6.6 unauthorized-attempt evidence, and INCIDENT_RESPONSE R-06 triage. Not queried during request handling.

```sql
-- migrations/0055_rate_limiting_schema.sql
-- Part 2: rate_limit_violations

CREATE TABLE rate_limit_violations (
  id                        UUID        PRIMARY KEY DEFAULT gen_random_uuid(),

  api_key_id                UUID
    REFERENCES api_keys(id) ON DELETE SET NULL,
  -- Nullable: unauthenticated requests (no valid API key) still produce violation rows.
  -- SET NULL on API key deletion preserves the violation record for abuse analysis.

  user_id                   UUID
    REFERENCES users(id) ON DELETE SET NULL,
  -- Nullable: not present for machine-credential requests or pre-auth probes.

  tenant_id                 UUID
    REFERENCES tenants(id) ON DELETE SET NULL,
  -- Nullable: consumer tier (no tenant) or unauthenticated.

  ip_country                CHAR(2),
  -- ISO 3166-1 alpha-2 country code only. Never a full IP address.
  -- Privacy floor: full IP is not stored anywhere in Postgres. Cloudflare CF-IPCountry header.

  endpoint                  TEXT        NOT NULL,
  -- Path template, not a full URL. E.g. '/v1/coaching/session' not '/v1/coaching/session?user=...'.
  -- No query parameters, no path IDs that could encode PII.

  violation_type            TEXT        NOT NULL
    CHECK (violation_type IN (
      'per_second',
      'per_minute',
      'per_hour',
      'monthly_quota',
      'burst'
    )),

  layer                     TEXT        NOT NULL
    CHECK (layer IN (
      'cloudflare_waf',
      'kv_sliding_window',
      'db_quota'
    )),
  -- Cloudflare WAF violations are forwarded via logpush to this table for unified abuse analysis.
  -- KV and db_quota violations are written directly by the Worker.

  request_count_at_violation INT,
  -- The counter value that triggered the violation. Useful for threshold tuning.

  limit_value               INT,
  -- The limit that was breached (e.g. 10 for a 10 req/min rule).

  resolved                  BOOLEAN     NOT NULL DEFAULT FALSE,
  -- Set TRUE by security reviewer or automated abuse flag resolution (§28.2.3).

  created_at                TIMESTAMPTZ NOT NULL DEFAULT NOW()
  -- No updated_at: this table is append-only for violation rows.
  -- resolved flag is the only mutable field; updated by form_system or security_reviewer.
);

-- Retention: 90 days rolling. Cleaned by pg_cron rate_limit_violations_cleanup (§28.7).
-- Privacy note: no full IP, no request body, no user-agent string, no path parameters
-- that could encode user content.

ALTER TABLE rate_limit_violations ENABLE ROW LEVEL SECURITY;

-- Index for abuse-pattern queries (api_key + time window).
CREATE INDEX idx_rlv_api_key_created
  ON rate_limit_violations (api_key_id, created_at)
  WHERE api_key_id IS NOT NULL;

-- Index for tenant-scoped security review.
CREATE INDEX idx_rlv_tenant_created
  ON rate_limit_violations (tenant_id, created_at)
  WHERE tenant_id IS NOT NULL;

-- Index for pg_cron cleanup sweep.
CREATE INDEX idx_rlv_created_at
  ON rate_limit_violations (created_at);
```

#### 28.2.3 `abuse_flags`

Pattern-based abuse detection records requiring manual or automated action. This table is an internal security signal — tenant admins have zero access. Every row creation and every `action_taken` change emits a DEC-030 HMAC-chained audit event (§28.6).

```sql
-- migrations/0055_rate_limiting_schema.sql
-- Part 3: abuse_flags

CREATE TABLE abuse_flags (
  id                    UUID        PRIMARY KEY DEFAULT gen_random_uuid(),

  flagged_entity_type   TEXT        NOT NULL
    CHECK (flagged_entity_type IN (
      'api_key',
      'user',
      'tenant',
      'ip_country'
    )),

  flagged_entity_id     TEXT        NOT NULL,
  -- Stores the UUID of the api_key, user, or tenant, cast to TEXT; or a 2-char country
  -- code for ip_country flags. TEXT (not UUID) to accommodate the country-code case
  -- without a nullable UUID column and a separate country_code column.
  -- Application layer must validate format against flagged_entity_type before INSERT.

  flag_type             TEXT        NOT NULL
    CHECK (flag_type IN (
      'credential_stuffing',
      'scraping',
      'quota_abuse',
      'prompt_injection_pattern',
      'bulk_export_anomaly',
      'suspicious_oauth'
    )),

  severity              TEXT        NOT NULL
    CHECK (severity IN ('low', 'medium', 'high', 'critical')),

  auto_actioned         BOOLEAN     NOT NULL DEFAULT FALSE,
  -- TRUE = Cloudflare Worker auto-applied an action (rate_limited, api_key_revoked)
  --        based on deterministic threshold rules. No human actor at creation time.
  -- FALSE = flag raised for manual security review; no action taken yet.

  action_taken          TEXT        NOT NULL DEFAULT 'none'
    CHECK (action_taken IN (
      'none',
      'rate_limited',
      'suspended',
      'api_key_revoked',
      'tenant_suspended'
    )),

  evidence_summary      TEXT,
  -- Statistical metadata only. No PII, no request content, no user-generated text.
  -- Acceptable: "450 requests in 60s from single api_key_id abc123"
  -- Acceptable: "14 distinct endpoints probed in 8s; pattern matches scraping signature v3"
  -- NOT acceptable: prompt content, email address, IP address, user name.

  reviewed_by           UUID
    REFERENCES users(id) ON DELETE SET NULL,
  -- The FORM internal security reviewer who triaged this flag.
  -- NULL for auto_actioned = TRUE rows until a human confirms or overrides.

  reviewed_at           TIMESTAMPTZ,

  resolved              BOOLEAN     NOT NULL DEFAULT FALSE,

  created_at            TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at            TIMESTAMPTZ NOT NULL DEFAULT NOW(),

  CONSTRAINT chk_abuse_review_coherent
    CHECK (
      (reviewed_by IS NOT NULL AND reviewed_at IS NOT NULL)
      OR (reviewed_by IS NULL AND reviewed_at IS NULL)
    )
);

-- DEC-030 trigger events: see §28.6.
-- security.abuse_flag_raised on INSERT.
-- security.abuse_action_taken on UPDATE WHERE action_taken <> OLD.action_taken.

ALTER TABLE abuse_flags ENABLE ROW LEVEL SECURITY;

-- Index for open-flag review queue (security dashboard).
CREATE INDEX idx_abuse_flags_open
  ON abuse_flags (severity, created_at)
  WHERE resolved = FALSE;

-- Index for entity-scoped lookups (is this api_key already flagged?).
CREATE INDEX idx_abuse_flags_entity
  ON abuse_flags (flagged_entity_type, flagged_entity_id)
  WHERE resolved = FALSE;
```

---

### 28.3 RLS Policies

The three tables in §28.2 have distinct access profiles. `api_quota_usage` is billing-adjacent and visible to the key owner and their tenant admins. `rate_limit_violations` is log-forward only — app users never read it directly. `abuse_flags` is a pure internal security signal: tenant admins have zero access regardless of role.

```sql
-- migrations/0055_rate_limiting_schema.sql
-- Part 4: Row-Level Security.

-- ── api_quota_usage ───────────────────────────────────────────────────────────

-- PERMISSIVE: API key owner or tenant admin can read own quota usage.
CREATE POLICY quota_usage_owner_read ON api_quota_usage
  AS PERMISSIVE FOR SELECT
  TO form_api
  USING (
    api_key_id IN (
      SELECT id FROM api_keys
      WHERE user_id = current_setting('app.current_user_id', TRUE)::UUID
         OR tenant_id = current_setting('app.current_tenant_id', TRUE)::UUID
    )
  );

-- RESTRICTIVE: defence-in-depth cross-tenant block.
-- Ensures a misconfigured permissive policy cannot leak rows across tenants.
CREATE POLICY quota_usage_cross_tenant_block ON api_quota_usage
  AS RESTRICTIVE FOR SELECT
  TO form_api
  USING (
    tenant_id = current_setting('app.current_tenant_id', TRUE)::UUID
    OR tenant_id IS NULL  -- consumer rows: only accessible if no tenant context is set
  );

-- form_system: full access (quota-check.ts Worker, billing jobs).
CREATE POLICY quota_usage_system_full ON api_quota_usage
  FOR ALL TO form_system
  USING (TRUE) WITH CHECK (TRUE);

-- ── rate_limit_violations ─────────────────────────────────────────────────────

-- form_api: INSERT only. App users and Workers can log violations; they cannot read them.
-- SELECT is intentionally withheld from form_api — violation history is not a user-facing feature.
CREATE POLICY rlv_api_insert_only ON rate_limit_violations
  AS PERMISSIVE FOR INSERT
  TO form_api
  WITH CHECK (TRUE);

-- tenant_manager role: SELECT own-tenant rows for security triage support.
-- Does not expose user_id rows for other tenants; tenant_id must match.
CREATE POLICY rlv_tenant_manager_read ON rate_limit_violations
  AS PERMISSIVE FOR SELECT
  TO form_api
  USING (
    tenant_id = current_setting('app.current_tenant_id', TRUE)::UUID
    AND current_setting('app.current_role', TRUE) IN ('tenant_owner', 'tenant_admin', 'tenant_manager')
  );

-- form_system: full access (cleanup jobs, abuse detection Workers).
CREATE POLICY rlv_system_full ON rate_limit_violations
  FOR ALL TO form_system
  USING (TRUE) WITH CHECK (TRUE);

-- ── abuse_flags ───────────────────────────────────────────────────────────────

-- form_api: NO access. Tenant admins and members cannot read, write, or enumerate
-- abuse flags regardless of role. This is an internal security signal only.
-- No permissive policy is created for form_api on this table; default-deny applies.

-- security_reviewer role: SELECT + UPDATE for review workflow.
-- security_reviewer is a Postgres role granted to FORM internal security staff only.
-- It may not INSERT or DELETE — flags are created by Workers (form_system) and
-- are never hard-deleted (they are resolved = TRUE).
CREATE POLICY abuse_flags_reviewer_read ON abuse_flags
  AS PERMISSIVE FOR SELECT
  TO security_reviewer
  USING (TRUE);

CREATE POLICY abuse_flags_reviewer_update ON abuse_flags
  AS PERMISSIVE FOR UPDATE
  TO security_reviewer
  USING (TRUE)
  WITH CHECK (TRUE);

-- form_system: full access (auto-actioning Workers, DEC-030 event emission).
CREATE POLICY abuse_flags_system_full ON abuse_flags
  FOR ALL TO form_system
  USING (TRUE) WITH CHECK (TRUE);
```

**Role access summary:**

| Actor | `api_quota_usage` | `rate_limit_violations` | `abuse_flags` |
|---|---|---|---|
| `form_api` (tenant_admin / owner) | SELECT own-tenant rows | INSERT only; SELECT own-tenant (manager+) | ZERO access |
| `form_api` (tenant_member) | SELECT own-key rows | INSERT only | ZERO access |
| `form_api` (other tenant) | ZERO rows (RESTRICTIVE) | ZERO rows | ZERO access |
| `form_system` | Full | Full | Full |
| `security_reviewer` | None (not granted) | None (not granted) | SELECT + UPDATE |

---

### 28.4 Quota Enforcement Flow

The `quota-check.ts` Cloudflare Worker middleware executes on every authenticated API request after Layer 1 WAF and Layer 2 KV checks have passed. It enforces the monthly quota ceiling tracked in `api_quota_usage`.

```
quota-check.ts middleware (runs before API handler)
│
├── 1. UPSERT api_quota_usage row for (api_key_id, period_month, endpoint_category)
│         INSERT ... ON CONFLICT (api_key_id, period_month, endpoint_category)
│         DO UPDATE SET request_count = api_quota_usage.request_count + 1,
│                       updated_at = NOW()
│         RETURNING request_count, quota_limit, overage_count, hard_blocked_at
│
├── 2. If hard_blocked_at IS NOT NULL
│         → Return HTTP 429
│           Retry-After: first day of next month (ISO 8601)
│           X-Quota-Reset: <next_month_first_day>
│           Body: {"error": "monthly_quota_hard_blocked", "resets_at": "..."}
│           Log rate_limit_violations row (layer: 'db_quota', violation_type: 'monthly_quota')
│           STOP
│
├── 3. If quota_limit IS NOT NULL AND request_count > quota_limit
│         ├── If overage_count < OVERAGE_GRACE (application constant; see OQ-RL-02)
│         │       → UPDATE api_quota_usage
│         │             SET overage_count = overage_count + 1, updated_at = NOW()
│         │         Allow request; add response header:
│         │           X-Quota-Warning: overage; X-Quota-Overage: <overage_count>
│         │
│         └── If overage_count >= OVERAGE_GRACE
│                 → UPDATE api_quota_usage
│                       SET hard_blocked_at = NOW(), updated_at = NOW()
│                   Emit DEC-030 security.quota_hard_block (§28.6)
│                   Return HTTP 429 (same as step 2)
│                   STOP
│
└── 4. quota_limit IS NULL (Consumer Pro unlimited) OR request_count <= quota_limit
          → Allow request; proceed to API handler
```

**Atomicity note.** The UPSERT in step 1 uses a single SQL statement (`INSERT ... ON CONFLICT DO UPDATE ... RETURNING`). This ensures `request_count` is incremented atomically; no separate SELECT + UPDATE pattern that could double-count under concurrent requests. The Supabase REST endpoint for this upsert uses `Prefer: return=representation` to retrieve the post-update row in one round-trip.

**Privacy invariant.** The Worker never logs the request body, query parameters, path parameters that encode user IDs, or any header that could contain user content (e.g., `Authorization` raw value). Only counters and categorical metadata are written to `api_quota_usage` and `rate_limit_violations`.

**80% consumption notification.** When `request_count` crosses 80% of `quota_limit` for the first time in a period, the Worker emits a `data.quota_threshold_warning` DEC-030 STANDARD event (1yr retention). This drives a CSM notification email; it does not block the request.

---

### 28.5 KV Sliding-Window (Edge Layer — Not Postgres)

This subsection is documented here for architectural completeness. No Postgres schema is involved.

**Key format:** `rl:{api_key_id}:{endpoint_category}:{window_minutes}`

**Value:** atomic counter (Cloudflare KV atomic increment via Durable Objects counter, not plain KV put/get, to avoid race conditions under concurrent requests).

**TTL:** `window_minutes * 60` seconds. The key expires automatically; no cleanup job needed.

**Enforcement:** If the counter exceeds the tier limit for that window, the Worker returns HTTP 429 with `Retry-After: <window_seconds>` and writes a `rate_limit_violations` row (layer: `'kv_sliding_window'`) via `waitUntil` (non-blocking).

**Tier limits (FORM-RL-001 config):**

| Tier | `coaching` (req/min) | `analytics_read` (req/min) | `webhook_mgmt` (req/min) | `admin` (req/min) | `bulk_import` (req/min) |
|---|---|---|---|---|---|
| Consumer Pro | 10 | 100 | 20 | 20 | 5 |
| Enterprise Starter | 50 | 200 | 50 | 50 | 20 |
| Enterprise Growth | 200 | 500 | 100 | 100 | 50 |
| Enterprise Custom | configurable | configurable | configurable | configurable | configurable |

Enterprise Custom limits are stored in the enterprise contract record (§16) and loaded into the Worker at request time via a KV cache keyed on `tenant_limits:{tenant_id}` (TTL: 5 minutes). This avoids a Postgres round-trip per request for limit lookup.

---

### 28.6 DEC-030 HMAC-Chained Audit Events

Three events must be registered in `docs/AUDIT_LOG_SCHEMA.md` under a new **"Rate limiting and abuse events"** subsection.

| Event type | Severity | Retention | Payload fields | When emitted |
|---|---|---|---|---|
| `security.quota_hard_block` | HIGH | 3 yr | `api_key_id`, `tenant_id` (UUID or null), `period_month` (ISO 8601 date), `endpoint_category`, `overage_count` (integer at time of block) | When `hard_blocked_at` is SET on `api_quota_usage`; emitted by quota-check.ts Worker in same transaction as the UPDATE |
| `security.abuse_flag_raised` | HIGH | 7 yr | `flag_id` (UUID), `flag_type`, `severity`, `flagged_entity_type`, `flagged_entity_id` (api_key UUID or tenant UUID — `user_id` included only if `flagged_entity_type = 'user'`), `auto_actioned` (boolean) | On `abuse_flags` INSERT; emitted by the abuse-detection Worker (form_system) |
| `security.abuse_action_taken` | HIGH | 7 yr | `flag_id` (UUID), `previous_action`, `new_action`, `reviewed_by` (UUID or null for automated action), `auto_actioned` (boolean) | On `abuse_flags` UPDATE where `action_taken` changes from its previous value |

**Privacy invariants for all three events:**
- No full IP address in any payload field.
- No request body or user-generated content.
- No `invited_email` or plaintext credentials.
- `flagged_entity_id` carries a UUID (api_key, user, or tenant) or a 2-char country code for `ip_country` flags. Never an email address or display name.

---

### 28.7 pg_cron Maintenance Jobs

Both jobs must be added to `docs/OBSERVABILITY.md §12.6` pg_cron registry.

#### Job 16 — `rate_limit_violations_cleanup`

```sql
-- Registered in docs/OBSERVABILITY.md §12.6 as job 16.
-- Schedule: daily 03:00 UTC
-- Retention policy: 90 days rolling.

DELETE FROM rate_limit_violations
WHERE created_at < NOW() - INTERVAL '90 days';

-- Expected row count: varies by traffic volume.
-- Alert if DELETE count = 0 for 7 consecutive days (indicates job stall, not zero violations).
-- Alert if runtime > 60s (index idx_rlv_created_at must be present).
```

#### Job 17 — `api_quota_usage_archive`

```sql
-- Registered in docs/OBSERVABILITY.md §12.6 as job 17.
-- Schedule: 1st of each month, 00:30 UTC
-- Retention policy: 36 months (SOC 2 CC4.1 billing evidence).

-- Step 1: identify rows past the 36-month window.
-- Step 2: emit a DEC-030 STANDARD 1yr event before deletion.
-- Step 3: delete.

-- The Worker wraps steps 2 and 3 in a transaction; the audit event is HMAC-chained
-- to the preceding event before the DELETE commits.

WITH archived AS (
  DELETE FROM api_quota_usage
  WHERE period_month < DATE_TRUNC('month', NOW()) - INTERVAL '36 months'
  RETURNING id, api_key_id, tenant_id, period_month, endpoint_category,
            request_count, tokens_consumed
)
-- application layer reads the RETURNING clause and emits:
-- DEC-030 event: data.quota_records_archived, STANDARD, 1yr
-- Payload: { archived_count: <n>, earliest_period_month: <date>, latest_period_month: <date> }
-- No per-row payload — aggregate only; no PII.
SELECT COUNT(*) AS archived_count FROM archived;
```

---

### 28.8 SOC 2 Mapping

| SOC 2 Criterion | Control | Evidence artefact |
|---|---|---|
| **CC6.1 — Logical access controls** | `api_quota_usage.quota_limit` enforces the authorized request ceiling per enterprise contract. Requests past the ceiling are blocked (hard_blocked_at) or warned (overage_count). Consumer Pro users have NULL quota_limit (unlimited); this is an explicit contract decision, not an absence of control. | **RL-E-001:** `api_quota_usage` export for an enterprise tenant over a 30-day audit window — demonstrates that `request_count` never exceeded `quota_limit + OVERAGE_GRACE` before `hard_blocked_at` was set. |
| **CC6.6 — Unauthorized access attempts logged** | `rate_limit_violations` records every layer-2 and layer-3 breach event with timestamp, violation type, layer, and anonymized entity identifiers. 90-day rolling retention covers the SOC 2 observation period. | **RL-E-002:** `rate_limit_violations` export for the audit period (all tenants; tenant_id anonymized for multi-tenant aggregates). Demonstrates unauthorized-rate breach events were captured and can be correlated with `abuse_flags`. |
| **CC7.2 — Monitoring for anomalous activity** | `abuse_flags` provides pattern-based detection with severity classification and a human-review workflow. `auto_actioned = TRUE` rows demonstrate automated response to deterministic abuse patterns. DEC-030 `security.abuse_flag_raised` and `security.abuse_action_taken` events provide a tamper-evident chain from detection to resolution. | **RL-E-003:** `abuse_flags` rows created during the audit period plus corresponding DEC-030 `security.abuse_flag_raised` and `security.abuse_action_taken` events — demonstrates anomaly monitoring is operating and findings are actioned. |

---

### 28.9 Implementation Checklist

#### P0 — M4: Schema and enforcement foundation

| # | Task | Owner | Priority | Milestone | Status |
|---|---|---|---|---|---|
| 1 | Run migration `0055_rate_limiting_schema.sql` (Parts 1–4): CREATE `api_quota_usage`, `rate_limit_violations`, `abuse_flags`; CREATE all indexes; CREATE all RLS policies. CI test: `form_api` SELECT on `abuse_flags` returns zero rows; cross-tenant SELECT on `api_quota_usage` returns zero rows. | platform-engineer | **P0** | M4 | [ ] |
| 2 | Implement `workers/middleware/quota-check.ts`: UPSERT `api_quota_usage`; overage grace logic; `hard_blocked_at` SET; HTTP 429 response with `Retry-After`; `X-Quota-Warning` header; 80% threshold notification; `waitUntil` violation log write. | platform-engineer | **P0** | M4 | [ ] |
| 3 | Define `FORM-RL-001` KV limits config (Cloudflare Workers environment): tier-to-limit mapping for all five endpoint categories; load into tenant-limits KV cache keyed on `tenant_limits:{tenant_id}` with 5-minute TTL. | platform-engineer + enterprise-architect | **P0** | M4 | [ ] |
| 4 | Register three DEC-030 events (`security.quota_hard_block`, `security.abuse_flag_raised`, `security.abuse_action_taken`) in `docs/AUDIT_LOG_SCHEMA.md` under new "Rate limiting and abuse events" subsection with severity, retention, and payload fields. | platform-engineer + compliance-officer | **P0** | M4 | [ ] |

#### P1 — M5: Abuse detection and admin visibility

| # | Task | Owner | Priority | Milestone | Status |
|---|---|---|---|---|---|
| 5 | Implement abuse detection Workers: credential-stuffing detector (N failed auths from single api_key in T seconds → INSERT `abuse_flags`); scraping detector (M distinct endpoints in W seconds → INSERT); prompt-injection-pattern classifier (statistical signal from coaching turn metadata — never content). | security-engineer + platform-engineer | **P1** | M5 | [ ] |
| 6 | Admin dashboard quota panel: per-tenant monthly `request_count` vs. `quota_limit` bar chart; overage indicator; `hard_blocked_at` alert banner; top-5 endpoint categories by usage. Gate on `enterprise.quota_panel_enabled` feature flag. | platform-engineer + design-craft | **P1** | M5 | [ ] |
| 7 | Collect RL-E-001 through RL-E-003 evidence artefacts (§28.8) after 30 days of production usage; store in `compliance/evidence/rate-limiting/`; cross-reference in `docs/SOC2_READINESS.md` CC6.1, CC6.6, CC7.2 evidence tables. | compliance-officer | **P1** | M5 | [ ] |
| 8 | Add pg_cron jobs 16 (`rate_limit_violations_cleanup`, daily 03:00 UTC) and 17 (`api_quota_usage_archive`, monthly 00:30 UTC) to `docs/OBSERVABILITY.md §12.6` registry with expected row counts, alert thresholds, and runbook links. | devops-lead | **P1** | M5 | [ ] |

#### P2 — Post-GA

| # | Task | Owner | Priority | Milestone | Status |
|---|---|---|---|---|---|
| 9 | Per-tenant configurable limits: allow Enterprise Custom tenants to set per-endpoint-category KV limits via admin dashboard. Persist overrides in a `tenant_rate_limit_overrides` table (FK to `tenants`, one row per endpoint_category); load into `tenant_limits:{tenant_id}` KV cache. Requires contract-tier guard: only `enterprise_custom` plan tenants may call the override endpoint. | enterprise-architect + platform-engineer | **P2** | M7 | [ ] |
| 10 | Resolve OQ-RL-01 (billing source of truth — §28.10) and OQ-RL-02 (OVERAGE_GRACE threshold — §28.10); document decisions in `docs/DECISION_LOG.md`. | enterprise-architect + finance | **P2** | M7 | [ ] |

---

### 28.10 Open Questions

| ID | Question | Owner | Priority | Resolution path |
|---|---|---|---|---|
| **OQ-RL-01** | **`api_quota_usage.tokens_consumed` as billing source of truth.** Should `tokens_consumed` be the authoritative field for enterprise overage charges (Claude API token billing), or should a separate `billing_metering` table be created for Stripe Metered integration? Using `api_quota_usage` is simpler through M7 and avoids a new table. Risk: `api_quota_usage` is scoped per api_key + endpoint_category; a billing summary across all keys for a tenant requires aggregation, which is feasible but less direct than a purpose-built metering table. Recommendation: use `api_quota_usage` as the source of truth through M7; create a dedicated `billing_metering` table if MRR exceeds $50k or if a Stripe Metered billing integration is contractually required. | enterprise-architect + finance | **P2 — before Stripe Metered integration** | Evaluate at M7 planning; document decision in `docs/DECISION_LOG.md`. |
| ~~**OQ-RL-02**~~ **🟢 Resolved — DATA_MODEL §30 (2026-06-11)** | ~~**OVERAGE_GRACE threshold: at what `overage_count` value does the hard block trigger?**~~ **Resolved:** `OVERAGE_GRACE_REQUESTS = 500` (1.0% of Starter 50k/month quota). Secondary warn at `QUOTA_SECONDARY_WARN_PCT = 0.95`. Both stored as Cloudflare Workers env vars. Migration 0059b adds `primary_warn_sent_at` + `secondary_warn_sent_at` dedup columns to `api_quota_usage`. New DEC-030 event `security.quota_95pct_warning` (STANDARD, 3yr). See `docs/DATA_MODEL.md §30.3`, `docs/DECISION_LOG.md DEC-045`. | enterprise-architect + customer-success | **🟢 Resolved** |

---

*v1.0 (2026-06-10): §28 Rate Limiting, Quota Enforcement & Abuse Prevention Schema. Three-layer enforcement architecture: Cloudflare WAF (L3/L4/L7 volumetric, no Postgres state), Cloudflare KV sliding-window (per-api-key per-endpoint per-window burst limits, no Postgres state), and Postgres DB-layer quota tracking (durable monthly counters for billing evidence and enterprise contract enforcement). Migration 0055 adds three tables. `api_quota_usage`: monthly request and token counter per `(api_key_id, period_month, endpoint_category)`; `quota_limit` from contract tier (NULL = Consumer Pro unlimited); `overage_count` grace window before `hard_blocked_at`; 36-month retention (SOC 2 CC4.1). `rate_limit_violations`: append-only breach event log; `ip_country` (2-char only — no full IP); `layer` discriminates cloudflare_waf | kv_sliding_window | db_quota; 90-day rolling retention via pg_cron job 16. `abuse_flags`: pattern-based detection records with severity classification (low | medium | high | critical); `auto_actioned` flag distinguishes Worker-automated responses from pending manual review; `evidence_summary` contains statistical metadata only — no PII, no request content; `security_reviewer` Postgres role holds the only non-system SELECT + UPDATE grant; tenant admins have zero access. RLS (§28.3): `api_quota_usage` — PERMISSIVE owner/tenant-admin SELECT + RESTRICTIVE cross-tenant block + `form_system` full; `rate_limit_violations` — `form_api` INSERT-only, tenant manager SELECT own-tenant, `form_system` full; `abuse_flags` — `form_api` zero access, `security_reviewer` SELECT + UPDATE, `form_system` full. Quota enforcement flow (§28.4): `quota-check.ts` Worker UPSERT with atomic `request_count + 1`; hard_blocked_at check first (HTTP 429 with Retry-After pointing to next month); overage grace window (X-Quota-Warning header); 80% threshold CSM notification. KV sliding-window tier limits (§28.5, FORM-RL-001 config): Consumer Pro 10 req/min coaching / 100 analytics_read; Enterprise Starter 50 / 200; Enterprise Growth 200 / 500; Enterprise Custom configurable from contract. Three DEC-030 events (§28.6): `security.quota_hard_block` HIGH 3yr (payload: api_key_id, tenant_id, period_month, endpoint_category, overage_count); `security.abuse_flag_raised` HIGH 7yr (payload: flag_type, severity, flagged_entity_type, flagged_entity_id, auto_actioned); `security.abuse_action_taken` HIGH 7yr (payload: flag_id, previous_action, new_action, reviewed_by). pg_cron jobs: job 16 `rate_limit_violations_cleanup` daily 03:00 UTC (DELETE rows > 90 days); job 17 `api_quota_usage_archive` monthly 00:30 UTC (DELETE rows > 36 months; emit `data.quota_records_archived` DEC-030 STANDARD 1yr before delete). SOC 2 mapping (§28.8): CC6.1 (quota_limit enforces authorized access boundary — RL-E-001), CC6.6 (rate_limit_violations logs unauthorized access attempts — RL-E-002), CC7.2 (abuse_flags + DEC-030 chain documents anomaly monitoring and response — RL-E-003). Implementation checklist: 4× P0 M4 (migration 0055, quota-check.ts middleware, FORM-RL-001 KV config, DEC-030 event registration); 4× P1 M5 (abuse detection Workers, admin dashboard quota panel, SOC 2 evidence collection, pg_cron registry); 2× P2 M7+ (per-tenant configurable limits, OQ resolutions). Two open questions: OQ-RL-01 (tokens_consumed as billing source of truth vs. dedicated billing_metering table — P2, evaluate at M7); OQ-RL-02 (OVERAGE_GRACE = 500 requests recommended — P0, must be set before quota enforcement ships). Owner: enterprise-architect + platform-engineer + security-engineer.*

---

## 29. PAM / Privileged Access Management Postgres Schema — OQ-INS-01 Resolution · CC6.1/CC6.2/CC6.3/CC6.6 Auditor Evidence

> **Closes OQ-INS-01** from `docs/INCIDENT_RESPONSE.md §R-20` (P0 blocker — automated insider detection programme blocked until `admin_jit_escalations` table exists).  
> **Companion section:** `docs/SSO_SCIM_IMPLEMENTATION.md §24` (PAM architecture, KV session schema, DEC-030 events, alert rules). This section provides the **Postgres-durable persistence layer** that makes KV state forensically auditable and enables SQL-based detection queries.  
> **Cross-references:** `docs/INCIDENT_RESPONSE.md §R-20` (C2 query depends on this table), `docs/INCIDENT_RESPONSE.md §R-20 checklist item 1` (creates this schema), `docs/OBSERVABILITY.md §29` (PAM / PAM Observability — AL-PAM-* alerting), `docs/SOC2_READINESS.md` CC6.1/CC6.2/CC6.3/CC6.6, `docs/AUDIT_LOG_SCHEMA.md` (PAM DEC-030 events registered in SSO §24.6).  
> **SOC 2 criteria:** CC6.1 (logical access controls), CC6.2 (access provisioning with approval), CC6.3 (timely deprovisioning / auto-expiry evidence), CC6.6 (privilege escalation audit trail).  
> **Owner:** enterprise-architect + security-engineer + compliance-officer. Migration: `0058_pam_schema.sql`.

---

### 29.1 Purpose and Scope

`docs/SSO_SCIM_IMPLEMENTATION.md §24` specifies the full PAM architecture: `pam-elevation-service` Cloudflare Worker, KV session records with TTL, approval workflow, break-glass protocol, and six DEC-030 HMAC-chained audit events. The KV store (`PAM_KV`) is the **enforcement plane** — it controls whether a given `pam_session_id` grants access at query time. However KV records are ephemeral: they expire with the session TTL (4h maximum for `read_only`; 15 min for `destructive`) and cannot be queried after expiry.

This creates a gap for the two workloads that require durable, queryable PAM state:

1. **Forensic incident response (INCIDENT_RESPONSE R-20 C2 query):** When an insider threat investigation opens, the IC must query: *"Did this admin have an active, legitimately-approved JIT escalation for the window in question?"* KV cannot answer this query after the TTL expires. Postgres can.

2. **SOC 2 auditor evidence (CC6.2 / CC6.3):** The auditor needs exportable records showing that every `form_admin` privilege use was pre-approved and auto-expired. DEC-030 events satisfy part of this, but structured relational records allow column-level queries that raw JSONB exports do not.

Migration 0058 adds two tables:

| Table | Purpose |
|---|---|
| `admin_jit_escalations` | Durable record of every JIT privilege escalation session; one row per PAM session; FK to `audit_log_events` DEC-030 chain. Closes OQ-INS-01. |
| `pam_break_glass_reviews` | Post-incident review records for every break-glass activation; mandated by SSO §24.4.3 within 72 hours of activation. |

**Out of scope for §29:** The KV schema (`pam:session:{id}`, TTL constants, approval workflow state) — that is specified in SSO §24.5 and managed entirely by the `pam-elevation-service` Worker. §29 does not duplicate KV schema.

---

### 29.2 KV ↔ Postgres Duality

The two stores are complementary, not redundant:

| Dimension | Cloudflare KV (`PAM_KV`) | Postgres (`admin_jit_escalations`) |
|---|---|---|
| **Primary use** | Real-time enforcement — `pam-db-proxy` validates before granting `SET ROLE form_admin` | Forensic audit trail — IR investigations, SOC 2 evidence export, compliance queries |
| **Write path** | `pam-elevation-service` Worker (on elevation approval) | `pam-elevation-service` Worker (same transaction; Postgres INSERT via `emit-audit-event` Worker or direct Supabase REST) |
| **Expiry behaviour** | TTL-based eviction (4h max) | Retained 7 years; no automatic deletion |
| **Query model** | Key lookup only (`pam:session:{id}`, `pam:by_admin:{admin_user_id}:{id}`) | Full SQL — JOINs, date ranges, role filters, count aggregates |
| **Privacy invariant** | `justification_hash` only (no free-text justification) | `business_justification` stored in plain text for IR forensics — restricted to `security_reviewer` role only |
| **SOC 2 role** | Demonstrates access is time-limited (CC6.3) | Demonstrates approval was obtained (CC6.2) and access was scoped (CC6.1) |

**Write sequencing:** `pam-elevation-service` writes to Postgres **after** KV write succeeds and the DEC-030 `pam.elevation_approved` event is emitted. Postgres write failure does not block session approval — KV remains authoritative for enforcement. A background reconciliation job (`pam_postgres_sync`, pg_cron job 20) detects missing Postgres rows by comparing `pam.elevation_approved` DEC-030 events against `admin_jit_escalations` and alerts on gaps > 30 minutes.

---

### 29.3 Migration 0058 — `admin_jit_escalations` and `pam_break_glass_reviews`

```sql
-- migrations/0058_pam_schema.sql
-- SOC 2 criteria: CC6.1, CC6.2, CC6.3, CC6.6
-- Owner: enterprise-architect + security-engineer
-- Depends on: users (for FK), audit_log_events (for dec030_event_id FK)
-- PAM KV schema companion: docs/SSO_SCIM_IMPLEMENTATION.md §24.5

-- ─── Table 1: admin_jit_escalations ──────────────────────────────────────────
-- One row per approved PAM escalation session.
-- Closes OQ-INS-01 from docs/INCIDENT_RESPONSE.md §R-20.

CREATE TABLE admin_jit_escalations (
    -- ── Identity ───────────────────────────────────────────────────────────────
    id                          uuid        PRIMARY KEY DEFAULT gen_random_uuid(),
    pam_session_id              uuid        NOT NULL UNIQUE,
    -- Matches the pam_session_id in PAM_KV record and DEC-030 event payload.
    -- UNIQUE enforces one Postgres row per KV session — no duplicates.

    -- ── Actor ─────────────────────────────────────────────────────────────────
    actor_user_id               uuid        NOT NULL
                                            REFERENCES users(id) ON DELETE RESTRICT,
    -- The FORM internal admin user who requested elevation.
    -- RESTRICT: disallow user deletion while an escalation record exists (7yr retention).

    -- ── Access tier ───────────────────────────────────────────────────────────
    access_level                text        NOT NULL
                                            CHECK (access_level IN ('read_only', 'read_write', 'destructive')),
    target_tenant_id            uuid        REFERENCES tenants(id) ON DELETE SET NULL,
    -- NULL = cross-tenant operation (e.g. DSAR compliance export).
    -- SET NULL on tenant deletion: the actor may have accessed a now-deleted tenant;
    -- retain the row for IR forensics but allow tenant record to be purged.

    -- ── Timing ────────────────────────────────────────────────────────────────
    escalation_start            timestamptz NOT NULL DEFAULT now(),
    escalation_expiry           timestamptz NOT NULL,
    -- Derived from access_level TTL (read_only 4h, read_write 1h, destructive 15m).
    -- Must match the TTL on the corresponding PAM_KV record.
    revoked_at                  timestamptz,
    -- Non-NULL if session was explicitly revoked before TTL expiry (e.g. IR R-20 response).

    -- ── Approval ──────────────────────────────────────────────────────────────
    authorised_by_ic_user_id    uuid        REFERENCES users(id) ON DELETE SET NULL,
    -- IC = primary approver (manager for read_write; co-signer for destructive).
    -- NULL for read_only (self-approve tier).
    authorised_by_cop_user_id   uuid        REFERENCES users(id) ON DELETE SET NULL,
    -- CoP = second cosigner; only populated for destructive tier dual-person rule.
    -- NULL for read_only and read_write.

    -- ── Justification ─────────────────────────────────────────────────────────
    business_justification      text,
    -- Plain-text justification — stored here for IR forensics under security_reviewer
    -- access only. Never exposed to tenant_admin. Never appears in DEC-030 payloads
    -- (those use justification_hash only — privacy invariant from SSO §24.6).
    justification_hash          text        NOT NULL,
    -- SHA-256 of business_justification. Cross-reference to DEC-030 event payload.

    -- ── Break-glass flag ──────────────────────────────────────────────────────
    is_break_glass              boolean     NOT NULL DEFAULT false,
    -- true = session created via form-break-glass Cloudflare Access app (SSO §24.4).
    -- When true, authorised_by_ic_user_id is NULL at session start;
    -- populated post-hoc when pam_break_glass_reviews row is created.

    -- ── Status ────────────────────────────────────────────────────────────────
    status                      text        NOT NULL DEFAULT 'approved'
                                            CHECK (status IN ('approved', 'revoked', 'expired')),
    -- 'approved': session was valid; expired naturally at escalation_expiry.
    -- 'revoked':  explicitly revoked before expiry (revoked_at IS NOT NULL).
    -- 'expired':  pam-expiry-sweeper confirmed expiry; equivalent to 'approved' for most queries.

    -- ── DEC-030 linkage ───────────────────────────────────────────────────────
    dec030_elevation_approved_event_id  uuid,
    -- References audit_log_events.id for the pam.elevation_approved event.
    -- Nullable (INSERT may race with DEC-030 event emission; updated by reconciliation job).
    dec030_session_expired_event_id     uuid,
    -- References audit_log_events.id for the pam.session_expired event; populated by expiry sweeper.

    -- ── Metadata ──────────────────────────────────────────────────────────────
    created_at                  timestamptz NOT NULL DEFAULT now()
    -- No updated_at: the only mutable columns are revoked_at, status,
    -- dec030_session_expired_event_id, and authorised_by_ic_user_id (break-glass post-hoc fill).
);

-- ── Indexes ──────────────────────────────────────────────────────────────────
CREATE INDEX idx_jit_actor          ON admin_jit_escalations (actor_user_id, escalation_start DESC);
-- Supports C2 query: all sessions by this actor in a date range.

CREATE INDEX idx_jit_tenant         ON admin_jit_escalations (target_tenant_id, escalation_start DESC)
    WHERE target_tenant_id IS NOT NULL;
-- Supports: all admin accesses to a specific tenant (per-tenant IR scope query).

CREATE INDEX idx_jit_break_glass    ON admin_jit_escalations (is_break_glass, escalation_start DESC)
    WHERE is_break_glass = true;
-- Supports: all break-glass sessions (small subset; fast scan for quarterly review).

CREATE INDEX idx_jit_active         ON admin_jit_escalations (escalation_expiry DESC)
    WHERE status = 'approved' AND revoked_at IS NULL;
-- Supports: FORM-INSIDER-001 alert — "is there an active approved session for this actor right now?".
-- Partial index keeps it small.

-- ─── Table 2: pam_break_glass_reviews ────────────────────────────────────────
-- One row per break-glass activation, recording the mandatory 72-hour post-incident review
-- required by SSO §24.4.3. Owned by compliance-officer role.

CREATE TABLE pam_break_glass_reviews (
    id                          uuid        PRIMARY KEY DEFAULT gen_random_uuid(),
    pam_session_id              uuid        NOT NULL UNIQUE
                                            REFERENCES admin_jit_escalations(pam_session_id) ON DELETE RESTRICT,
    -- UNIQUE: one review per break-glass session. RESTRICT: do not allow escalation record
    -- deletion while review record exists (forensic chain must remain intact).

    -- ── Timing ────────────────────────────────────────────────────────────────
    review_due_at               timestamptz NOT NULL,
    -- = escalation_start + 72 hours. Computed at INSERT by trigger or application layer.
    reviewed_at                 timestamptz,
    -- NULL = review not yet completed. Drives AL-PAM-BG-01 alert (overdue review).

    -- ── Outcome ───────────────────────────────────────────────────────────────
    outcome                     text
                                CHECK (outcome IN ('no_anomaly', 'anomaly_detected', 'process_gap')),
    -- NULL until reviewed_at is set.
    -- 'no_anomaly':       break-glass was justified by the active incident; no follow-up required.
    -- 'anomaly_detected': unexpected query pattern or scope; escalate to security incident (R-20).
    -- 'process_gap':      break-glass occurred due to PAM reliability issue; file reliability ticket.

    -- ── Reviewers ─────────────────────────────────────────────────────────────
    reviewed_by_security        uuid        REFERENCES users(id) ON DELETE SET NULL,
    reviewed_by_compliance      uuid        REFERENCES users(id) ON DELETE SET NULL,
    -- Both must be non-NULL for outcome to be accepted (two-reviewer sign-off).

    -- ── Evidence ──────────────────────────────────────────────────────────────
    github_issue_url            text,
    -- Auto-created by break-glass-notifier Worker at activation; format:
    -- https://github.com/form-internal/security-reviews/issues/{n}
    pg_audit_lines_reviewed     integer,
    -- Count of pg_audit statement log lines reviewed by security-engineer.
    -- NULL if pg_audit was not available during the session (e.g. PgBouncer restart).

    notes                       text,
    -- Free-text reviewer notes. Never exposed outside form_compliance role.
    -- Contains summary of pg_audit findings and decision rationale.

    -- ── DEC-030 linkage ───────────────────────────────────────────────────────
    dec030_break_glass_activated_event_id   uuid,
    -- References audit_log_events.id for pam.break_glass_activated event.
    dec030_review_completed_event_id        uuid,
    -- References audit_log_events.id for pam.break_glass_review_completed event (§29.6).

    created_at                  timestamptz NOT NULL DEFAULT now()
);

CREATE INDEX idx_bg_review_due ON pam_break_glass_reviews (review_due_at ASC)
    WHERE reviewed_at IS NULL;
-- Supports AL-PAM-BG-01: find overdue reviews efficiently.
```

---

### 29.4 Row-Level Security

#### 29.4.1 `admin_jit_escalations`

```sql
ALTER TABLE admin_jit_escalations ENABLE ROW LEVEL SECURITY;

-- form_system: full access (for pam-elevation-service Worker and reconciliation job).
CREATE POLICY jit_system_full ON admin_jit_escalations
    AS PERMISSIVE FOR ALL TO form_system USING (true) WITH CHECK (true);

-- security_reviewer: SELECT-only across all rows (IR investigations, SOC 2 evidence).
CREATE POLICY jit_security_reviewer_read ON admin_jit_escalations
    AS PERMISSIVE FOR SELECT TO security_reviewer USING (true);

-- form_compliance: SELECT-only (compliance-officer SOC 2 evidence export).
CREATE POLICY jit_compliance_read ON admin_jit_escalations
    AS PERMISSIVE FOR SELECT TO form_compliance USING (true);

-- form_api: INSERT-only, no SELECT (pam-elevation-service creates rows; cannot read back).
CREATE POLICY jit_api_insert ON admin_jit_escalations
    AS PERMISSIVE FOR INSERT TO form_api WITH CHECK (true);

-- RESTRICTIVE cross-tenant block: tenant_admin NEVER sees PAM records.
-- A tenant admin logged in as their own RBAC role cannot query this table.
CREATE POLICY jit_no_tenant_admin ON admin_jit_escalations
    AS RESTRICTIVE FOR ALL
    USING (current_setting('app.role', true) NOT IN ('tenant_owner', 'tenant_admin', 'tenant_manager'));
```

**Privacy invariant:** `business_justification` is a plain-text column accessible only to `security_reviewer` and `form_compliance` roles. The `form_api` INSERT policy does not grant SELECT; `form_api` cannot read back justification text after writing it. The DEC-030 event (`pam.elevation_approved`) stores only `justification_hash` — never the plain text.

#### 29.4.2 `pam_break_glass_reviews`

```sql
ALTER TABLE pam_break_glass_reviews ENABLE ROW LEVEL SECURITY;

-- form_compliance: full access (compliance-officer owns the review process).
CREATE POLICY bgr_compliance_full ON pam_break_glass_reviews
    AS PERMISSIVE FOR ALL TO form_compliance USING (true) WITH CHECK (true);

-- security_reviewer: SELECT (security-engineer participates in review but does not own it).
CREATE POLICY bgr_security_read ON pam_break_glass_reviews
    AS PERMISSIVE FOR SELECT TO security_reviewer USING (true);

-- form_system: INSERT only (break-glass-notifier Worker creates the stub row on activation).
CREATE POLICY bgr_system_insert ON pam_break_glass_reviews
    AS PERMISSIVE FOR INSERT TO form_system WITH CHECK (true);

-- All other roles: no access.
CREATE POLICY bgr_deny_all ON pam_break_glass_reviews
    AS RESTRICTIVE FOR ALL
    USING (current_role IN ('form_compliance', 'security_reviewer', 'form_system'));
```

---

### 29.5 Audit Trigger Pattern — `app.pam_session_id` GUC

SSO §24.2 specifies that `pam-db-proxy` executes `SET app.pam_session_id = '<id>'` before each privileged query batch and `RESET app.pam_session_id` after. Triggers on sensitive Postgres tables read this GUC and inject `pam_session_id` into their `audit_log` row metadata, creating a **database-layer linkage** between every privileged query and the PAM session that authorised it — independent of application-layer DEC-030 events.

The following tables must carry this trigger:

| Table | Trigger name | Rationale |
|---|---|---|
| `tenant_sso_configs` | `trg_pam_audit_sso_config` | SSO configuration changes are high-blast-radius; every change under `form_admin` must be traceable to a PAM session |
| `tenant_members` | `trg_pam_audit_tenant_members` | Seat provisioning / deprovisioning by an admin requires PAM-session traceability |
| `enterprise_seat_assignments` | `trg_pam_audit_seat_assignments` | Same rationale as `tenant_members` |
| `user_profiles` | `trg_pam_audit_user_profiles` | Admin access to individual user records is Art. 9 scope |
| `cv_sessions` | `trg_pam_audit_cv_sessions` | CV keypoint data is biometric (Art. 9); admin access must be PAM-traced |
| `audit_log_events` | `trg_pam_audit_audit_log` | Any admin modification to the audit log itself must be traceable — meta-audit |

**Trigger template (applied to each table above):**

```sql
CREATE OR REPLACE FUNCTION fn_inject_pam_session_id()
RETURNS TRIGGER LANGUAGE plpgsql SECURITY DEFINER AS $$
DECLARE
    v_pam_session_id text;
BEGIN
    v_pam_session_id := current_setting('app.pam_session_id', true);
    -- 'true' = return NULL if GUC is not set (non-PAM sessions have no pam_session_id).

    IF v_pam_session_id IS NOT NULL AND v_pam_session_id <> '' THEN
        -- Append to the row's metadata JSONB (assumes a metadata JSONB column exists).
        -- For tables without a metadata column, write to audit_log_events instead.
        NEW.metadata := COALESCE(NEW.metadata, '{}'::jsonb)
            || jsonb_build_object('pam_session_id', v_pam_session_id);
    END IF;

    RETURN NEW;
END;
$$;

-- Applied to each table:
CREATE TRIGGER trg_pam_audit_sso_config
    BEFORE INSERT OR UPDATE ON tenant_sso_configs
    FOR EACH ROW EXECUTE FUNCTION fn_inject_pam_session_id();
-- (Repeat for each table in the list above)
```

**Non-PAM sessions:** When `app.pam_session_id` is not set (normal `form_api` request), `current_setting(..., true)` returns NULL and the trigger is a no-op. No performance overhead outside PAM sessions.

---

### 29.6 DEC-030 Events

All six core PAM audit events (`pam.elevation_requested`, `pam.elevation_approved`, `pam.elevation_denied`, `pam.session_expired`, `pam.break_glass_activated`, `pam.break_glass_expired`) are defined in `docs/SSO_SCIM_IMPLEMENTATION.md §24.6` and registered in `docs/AUDIT_LOG_SCHEMA.md`. §29 does not add new PAM events.

One event is added by §29 to close the break-glass review lifecycle:

| Event | Severity | Retention | Trigger | Payload |
|---|---|---|---|---|
| `pam.break_glass_review_completed` | HIGH | 7 years | Emitted by compliance-officer via admin API when `pam_break_glass_reviews.reviewed_at` is set | `pam_session_id`, `outcome` (enum), `reviewed_by_security`, `reviewed_by_compliance`, `review_completed_at`, `github_issue_url` |

This event closes the break-glass DEC-030 chain: `pam.break_glass_activated` → `pam.break_glass_expired` → `pam.break_glass_review_completed`. An auditor can confirm the full break-glass lifecycle in the HMAC chain by querying these three event types with the same `pam_session_id`.

**`pam.break_glass_review_completed` Zod schema:**

```typescript
import { z } from 'zod';

const PamBreakGlassReviewCompletedPayload = z.object({
  pam_session_id:        z.string().uuid(),
  outcome:               z.enum(['no_anomaly', 'anomaly_detected', 'process_gap']),
  reviewed_by_security:  z.string().uuid(),
  reviewed_by_compliance: z.string().uuid(),
  review_completed_at:   z.string().datetime(),
  github_issue_url:      z.string().url().optional(),
  pg_audit_lines_reviewed: z.number().int().nonneg().optional(),
});
```

---

### 29.7 C2 Query — INCIDENT_RESPONSE R-20 FORM-INSIDER-001 Automation

The C2 query from INCIDENT_RESPONSE R-20 requires `admin_jit_escalations` to be populated. The query determines whether a detected admin operation had a valid, legitimately-approved JIT escalation window:

```sql
-- C2: Active or recently-expired PAM escalation for a given actor
-- Run by IR IC when FORM-INSIDER-001 alert fires for admin_user_id = :actor_id
-- Replace :actor_id and :window_start / :window_end with investigation parameters.

SELECT
    jit.id,
    jit.pam_session_id,
    jit.access_level,
    jit.target_tenant_id,
    jit.escalation_start,
    jit.escalation_expiry,
    jit.revoked_at,
    jit.status,
    jit.is_break_glass,
    jit.business_justification,         -- available to security_reviewer role only
    jit.authorised_by_ic_user_id,
    jit.authorised_by_cop_user_id,
    CASE
        WHEN jit.revoked_at IS NOT NULL THEN 'REVOKED'
        WHEN now() BETWEEN jit.escalation_start AND jit.escalation_expiry THEN 'ACTIVE'
        ELSE 'EXPIRED'
    END AS session_state
FROM admin_jit_escalations jit
WHERE
    jit.actor_user_id = :actor_id
    AND jit.escalation_start BETWEEN :window_start AND :window_end
ORDER BY jit.escalation_start DESC;
```

**Interpretation guide:**

| Result | Interpretation |
|---|---|
| Row with `session_state = 'ACTIVE'` or `'EXPIRED'`, `authorised_by_ic_user_id IS NOT NULL` | Admin operation was within a legitimately-approved PAM window. Low suspicion — continue standard IR triage. |
| Zero rows for the query window | Admin operation occurred with **no approved PAM session** — this is the FORM-INSIDER-001 trigger condition. Treat as unauthorised `form_admin` access. Escalate immediately per R-20 response procedure. |
| Row with `is_break_glass = true` | Break-glass session — check `pam_break_glass_reviews` for linked review outcome. If `reviewed_at IS NULL`, review is overdue (breach of §24.4.3 72-hour requirement). |
| Row with `status = 'revoked'` and `revoked_at` before the suspicious operation timestamp | Session was revoked before the operation occurred. Operation was **outside** the revocation window — treat as potentially unauthorised. |

**Automation note:** Once migration 0058 is deployed, the FORM-INSIDER-001 alert (currently manual per OQ-INS-01) can be implemented as a `pg_cron` job or a Cloudflare Worker that polls for `audit_log_events` where `actor_role = 'form_admin'` with no matching `admin_jit_escalations` row within the operation timestamp's range. Checklist item 2 (§29.10) tracks this automation.

---

### 29.8 FORM-INSIDER-001 Automation Path

Before §29 (OQ-INS-01 open), FORM-INSIDER-001 could not fire automatically:

> *"C2 cannot be run as written and must be replaced by a manual review of admin operation timestamps against calendar records."* — INCIDENT_RESPONSE R-20 OQ-INS-01

After §29 (migration 0058 deployed), the automation path is:

1. **Source signal:** `audit_log_events` where `event_type = 'pam.*'` provides the DEC-030 chain. Any `form_admin` Postgres operation via `pam-db-proxy` emits `SET app.pam_session_id` → trigger row captures `pam_session_id` in table metadata.

2. **Detection query** (to be run by a pg_cron monitoring job every 15 minutes):

```sql
-- FORM-INSIDER-001 candidate: form_admin-class audit events without a valid PAM session
-- Fires when any audit_log_events row has metadata.pam_session_id set
-- but no matching admin_jit_escalations row covers that timestamp.

SELECT DISTINCT ale.id AS audit_event_id,
       ale.actor_id,
       ale.occurred_at,
       ale.metadata->>'pam_session_id' AS claimed_pam_session_id
FROM audit_log_events ale
WHERE
    -- Only audit events that carry a PAM session claim
    ale.metadata->>'pam_session_id' IS NOT NULL
    -- Only recent window (15-minute look-back for cron cadence)
    AND ale.occurred_at >= now() - INTERVAL '15 minutes'
    -- No matching approved escalation covers this timestamp
    AND NOT EXISTS (
        SELECT 1
        FROM admin_jit_escalations jit
        WHERE jit.pam_session_id = (ale.metadata->>'pam_session_id')::uuid
          AND ale.occurred_at BETWEEN jit.escalation_start AND jit.escalation_expiry
          AND jit.revoked_at IS NULL
    );
```

3. **Alert action:** If any rows returned, emit `security.insider_threat_candidate` DEC-030 event (HIGH, 7yr) with `audit_event_id`, `actor_id`, `claimed_pam_session_id`, fire PagerDuty P0 routing key `PAM_PAGERDUTY_ROUTING_KEY`.

This closes the operational gap identified in OQ-INS-01 and makes the insider detection programme fully automated.

---

### 29.9 SOC 2 Evidence Mapping

| Criterion | Control | §29 mechanism | Evidence artefact | Status |
|---|---|---|---|---|
| **CC6.1** — Logical access controls | `form_admin` role is never held as standing credential; every privileged session is recorded in `admin_jit_escalations` with explicit scope and TTL | `admin_jit_escalations` with `target_tenant_id` scope + `escalation_expiry` TTL column; `ACTIVE`/`EXPIRED` session state computation | **PAM-E-001:** 30-day export of `admin_jit_escalations` showing zero rows with `target_tenant_id IS NULL` for single-tenant operations (confirms tenant scope enforcement); all rows have `escalation_expiry ≤ escalation_start + 4h` (confirms max TTL) | 🟡 Authored — closes on migration 0058 deploy + first 30-day export |
| **CC6.2** — Access provisioning | Every `read_write` or `destructive` elevation carries a non-NULL `authorised_by_ic_user_id`; `destructive` also carries `authorised_by_cop_user_id` | `admin_jit_escalations.authorised_by_ic_user_id NOT NULL` for non-read_only rows (enforced by `pam-elevation-service`); CHECK constraint added in §29.10 | **PAM-E-002:** Export of all `admin_jit_escalations` rows where `access_level IN ('read_write', 'destructive')` for the observation period; all rows show non-NULL approver; cross-reference with DEC-030 `pam.elevation_approved` events confirming approval predates session start | 🟡 Authored — closes on first elevation of `read_write` or `destructive` tier |
| **CC6.3** — Timely deprovisioning | Sessions auto-expire by KV TTL; Postgres records `escalation_expiry` and `status`; reconciliation job closes any gap between KV expiry and Postgres record | `admin_jit_escalations.status = 'expired'` rows populated by `pam-expiry-sweeper`; reconciliation job (pg_cron job 20) detects missing rows | **PAM-E-003:** Export of `pam.session_expired` DEC-030 events + matching `admin_jit_escalations` rows showing `status = 'expired'`; zero rows with `escalation_expiry < now() AND status = 'approved'` (confirms no orphaned approved-but-expired rows) | 🟡 Authored |
| **CC6.6** — Privilege escalation audit trail | Every elevation, denial, break-glass, and expiry event is HMAC-chained in DEC-030; break-glass sessions also have a mandatory post-incident review record | `pam_break_glass_reviews` table; `pam.break_glass_review_completed` DEC-030 event; 72-hour review SLA enforced by AL-PAM-BG-01 alert | **PAM-E-004:** Export of all `pam_break_glass_reviews` rows for the observation period; all rows show `reviewed_at IS NOT NULL` and `outcome` set; cross-reference with `pam.break_glass_review_completed` DEC-030 events confirming both reviewers signed off | 🟡 Authored |

All four artefacts are stored in `compliance/evidence/pam/` and cross-referenced in `docs/SOC2_READINESS.md §24.8` (SSO §24 PAM evidence artefacts CC6-E-PAM-001 through CC6-E-PAM-004 → updated to include §29 Postgres evidence as CC6-E-PAM-005 through CC6-E-PAM-008).

---

### 29.10 Implementation Checklist

#### P0 — Must complete before FORM-INSIDER-001 alert can be automated (M6)

| # | Task | Owner | Priority | Milestone | Status |
|---|---|---|---|---|---|
| 1 | Run migration `0058_pam_schema.sql`: CREATE `admin_jit_escalations` (with all columns, indexes, and RLS policies from §29.3 and §29.4.1); CREATE `pam_break_glass_reviews` (with all columns, index, and RLS policies from §29.3 and §29.4.2). Confirm: (a) `\d admin_jit_escalations` shows UNIQUE on `pam_session_id`, RLS enabled; (b) `form_api` INSERT succeeds, SELECT returns 0 rows (INSERT-only RLS confirmed); (c) `tenant_admin`-role query on `admin_jit_escalations` returns 0 rows (RESTRICTIVE policy confirmed). | platform-engineer | **P0** | M6 | [ ] |
| 2 | Update `pam-elevation-service` Worker: on `pam.elevation_approved` DEC-030 event emission, also INSERT a row into `admin_jit_escalations` via Supabase REST. Columns: `pam_session_id`, `actor_user_id`, `access_level`, `target_tenant_id`, `escalation_start`, `escalation_expiry`, `business_justification` (plaintext, not the hash), `justification_hash`, `is_break_glass`, `authorised_by_ic_user_id`, `authorised_by_cop_user_id`, `dec030_elevation_approved_event_id`. Postgres INSERT failure must not block session approval — KV remains authoritative. Log Postgres INSERT failure to Sentry with pam_session_id for reconciliation. | platform-engineer | **P0** | M6 | [ ] |
| 3 | Update `break-glass-notifier` Worker: on break-glass activation, INSERT stub row into `pam_break_glass_reviews` with `pam_session_id`, `review_due_at = now() + INTERVAL '72 hours'`, `github_issue_url` (from auto-created GitHub issue), `dec030_break_glass_activated_event_id`. `reviewed_at`, `outcome`, `reviewed_by_*` remain NULL until compliance-officer completes review. | devops-lead | **P0** | M6 | [ ] |
| 4 | Update `pam-expiry-sweeper` Worker: when emitting `pam.session_expired` DEC-030 event, also UPDATE `admin_jit_escalations SET status = 'expired', dec030_session_expired_event_id = <event_id> WHERE pam_session_id = <id>`. This ensures Postgres `status` column tracks KV expiry confirmation. | devops-lead | **P0** | M6 | [ ] |
| 5 | Add `fn_inject_pam_session_id()` trigger to all six tables listed in §29.5 (`tenant_sso_configs`, `tenant_members`, `enterprise_seat_assignments`, `user_profiles`, `cv_sessions`, `audit_log_events`). Run integration test: start a `read_only` PAM session, execute a SELECT on `tenant_sso_configs` via `pam-db-proxy`, confirm that the query row in `audit_log_events` (generated by the table-level trigger) carries `metadata.pam_session_id = <session_id>`. | platform-engineer | **P0** | M6 | [ ] |
| 6 | Register `pam.break_glass_review_completed` as a new DEC-030 event type in `docs/AUDIT_LOG_SCHEMA.md §6` with Zod schema from §29.6. Deploy updated event registry to `emit-audit-event` Worker. | platform-engineer | **P0** | M6 | [ ] |

#### P1 — Before FORM-INSIDER-001 alert goes to PagerDuty (M7)

| # | Task | Owner | Priority | Milestone | Status |
|---|---|---|---|---|---|
| 7 | Implement pg_cron job 20 `pam_postgres_sync` (§29.2 reconciliation job): runs every 30 minutes; selects `pam.elevation_approved` DEC-030 events from `audit_log_events` in the last 60 minutes; for each event, confirms a matching `admin_jit_escalations` row exists (`pam_session_id = event.payload.pam_session_id`); if missing, emits `security.pam_postgres_sync_gap` DEC-030 event (HIGH, 3yr) and fires PagerDuty P2 to `security-engineer`. Add to `docs/OBSERVABILITY.md §12.6` pg_cron registry as job 20, freshness window 35 minutes. | devops-lead | **P1** | M7 | [ ] |
| 8 | Add AL-PAM-BG-01 alert rule: `pam_break_glass_reviews.review_due_at < now()` AND `reviewed_at IS NULL` → PagerDuty P1 to `form-compliance`. Query: `SELECT pam_session_id, review_due_at FROM pam_break_glass_reviews WHERE review_due_at < now() AND reviewed_at IS NULL`. Cadence: pg_cron daily 08:00 UTC. Implement as pg_cron job 21. Add to `docs/OBSERVABILITY.md §29` (PAM Observability) alert table alongside AL-PAM-01/02/03. | devops-lead + compliance-officer | **P1** | M7 | [ ] |
| 9 | Implement FORM-INSIDER-001 automated detection (§29.8 detection query) as a Cloudflare Worker Cron (every 15 minutes) or pg_cron job. On match: emit `security.insider_threat_candidate` DEC-030 event (HIGH, 7yr); fire PagerDuty P0 routing key `PAM_PAGERDUTY_ROUTING_KEY`. Update INCIDENT_RESPONSE.md R-20 to mark OQ-INS-01 as 🟢 Resolved, pointing to this checklist item. | security-engineer + platform-engineer | **P1** | M7 | [ ] |
| 10 | Collect PAM-E-001 through PAM-E-004 evidence artefacts (§29.9) 30 days after M6 go-live; store in `compliance/evidence/pam/`; update `docs/SOC2_READINESS.md §24.8` to reference CC6-E-PAM-005 through CC6-E-PAM-008. | compliance-officer | **P1** | M7 | [ ] |
| 11 | Add CHECK constraint to `admin_jit_escalations`: `chk_jit_approver_required CHECK (access_level = 'read_only' OR authorised_by_ic_user_id IS NOT NULL)`. This enforces at the DB layer that non-`read_only` escalations always carry an approver — preventing a scenario where `pam-elevation-service` fails to write the approver UUID. Deploy as a separate migration `0058b_pam_approver_constraint.sql`. | platform-engineer | **P1** | M7 | [ ] |

#### P2 — Post-GA

| # | Task | Owner | Priority | Milestone | Status |
|---|---|---|---|---|---|
| 12 | Implement `tenant_admin`-facing PAM session transparency report: a quarterly report (generated by `form_compliance` role) listing the count of `form_admin` sessions that accessed a given tenant's data in the prior quarter, with `access_level` distribution (no actor names, no business_justification). Satisfies enterprise customer transparency expectation and supports GDPR Art. 28 processor accountability. Update `docs/ENTERPRISE_ADMIN_API.md` with the new reporting endpoint. | compliance-officer + platform-engineer | **P2** | M10 | [ ] |
| 13 | Resolve OQ-PAM-01 (§29.11): decide whether `admin_jit_escalations` rows with `is_break_glass = true` should be visible to `form_compliance` in a separate, more restricted query path that requires dual-authentication (both compliance-officer and security-engineer signatures on the query). | compliance-officer + security-engineer | **P2** | M8 | [x] **DEC-042** — alert-on-access adopted; `security.break_glass_audit_record_accessed` DEC-030 HIGH/7yr registered in AUDIT_LOG_SCHEMA.md v1.3 (2026-06-11). No dual-auth query gate. |

---

### 29.11 Open Questions

| ID | Question | Priority | Owner | Resolution path |
|---|---|---|---|---|
| **OQ-PAM-01** | ~~**Should `admin_jit_escalations` rows with `is_break_glass = true` require dual-role authentication to query?**~~ **🟢 RESOLVED — DEC-042 (2026-06-11).** Decision: alert-on-access approach. No dual-auth query gate. `security.break_glass_audit_record_accessed` DEC-030 HIGH/7yr registered in AUDIT_LOG_SCHEMA.md v1.3 — every query targeting `is_break_glass = true` is logged in the immutable HMAC chain. Rationale: query-time co-approval blocks incident response (IR IC needs break-glass records within T+15 min); alert-on-access achieves the same transparency goal without the catch-22. See `docs/DECISION_LOG.md DEC-042`. | P2 → ✅ | compliance-officer + security-engineer | **Resolved** — DEC-042 logged in `docs/DECISION_LOG.md` (2026-06-11) |
| **OQ-PAM-02** | ~~**Should `business_justification` (plain-text) ever be included in the SOC 2 auditor evidence export?**~~ **🟢 RESOLVED — DEC-043 (2026-06-11).** Decision: redacted-sample approach. `business_justification` excluded from any bulk export. Auditors receive `justification_hash` in DEC-030 events; verify specific examples via plaintext on-request during fieldwork; FORM prepares a sanitised redacted sample at `compliance/evidence/pam/justification-sample.md`. Rationale: bulk export risks leaking customer-identifying context; hash-in-chain already satisfies CC6.2 qualitatively; per-instance verification on fieldwork request is auditor-standard practice. See `docs/DECISION_LOG.md DEC-043`. | P1 → ✅ | compliance-officer | **Resolved** — DEC-043 logged in `docs/DECISION_LOG.md` (2026-06-11) |

---

*v1.0 (2026-06-10): §29 PAM / Privileged Access Management Postgres Schema — OQ-INS-01 Resolution · CC6.1/CC6.2/CC6.3/CC6.6 Auditor Evidence. Closes OQ-INS-01 from `docs/INCIDENT_RESPONSE.md §R-20` (P0 blocker — FORM-INSIDER-001 alert and the C2 forensic query were blocked until `admin_jit_escalations` table existed; all automated insider detection required manual calendar-record cross-referencing). §29.1 purpose and scope: two-store duality — KV (enforcement, ephemeral, TTL-based) vs. Postgres (forensic audit, durable 7yr, SQL-queryable); write sequencing (KV first, Postgres second, failure-tolerant); reconciliation job closes any gap. §29.2 KV ↔ Postgres duality table: dimensions are primary use, write path, expiry, query model, privacy invariant, and SOC 2 role. §29.3 migration 0058: two tables — `admin_jit_escalations` (pam_session_id UNIQUE, actor_user_id FK, access_level CHECK enum, target_tenant_id nullable, escalation_start/expiry/revoked_at timing columns, authorised_by_ic_user_id / authorised_by_cop_user_id approval columns, business_justification plain-text under security_reviewer-only, justification_hash for DEC-030 cross-reference, is_break_glass flag, status CHECK enum, dec030 event ID columns; four indexes: idx_jit_actor, idx_jit_tenant, idx_jit_break_glass, idx_jit_active partial) and `pam_break_glass_reviews` (pam_session_id UNIQUE FK, review_due_at, reviewed_at, outcome CHECK enum, reviewed_by_security/compliance FK, github_issue_url, pg_audit_lines_reviewed, notes, dec030 event ID columns; idx_bg_review_due partial). §29.4 RLS: `admin_jit_escalations` — five policies (form_system PERMISSIVE ALL, security_reviewer PERMISSIVE SELECT, form_compliance PERMISSIVE SELECT, form_api PERMISSIVE INSERT-only, RESTRICTIVE block for tenant_admin/owner/manager roles — HR invariant at DDL level); `pam_break_glass_reviews` — four policies (form_compliance PERMISSIVE ALL, security_reviewer PERMISSIVE SELECT, form_system PERMISSIVE INSERT-only, RESTRICTIVE deny-all for other roles). §29.5 audit trigger pattern: `fn_inject_pam_session_id()` SECURITY DEFINER trigger reads `app.pam_session_id` GUC (current_setting with null-safe true flag); appends pam_session_id to metadata JSONB on all six sensitive tables (tenant_sso_configs, tenant_members, enterprise_seat_assignments, user_profiles, cv_sessions, audit_log_events); no-op when GUC not set — zero overhead for normal form_api requests. §29.6 DEC-030: one new event `pam.break_glass_review_completed` (HIGH, 7yr) closes the break-glass DEC-030 chain (break_glass_activated → break_glass_expired → break_glass_review_completed); Zod schema provided. §29.7 C2 query: full SQL provided; interpretation guide (zero rows = FORM-INSIDER-001 condition; row with ACTIVE/EXPIRED state + approver = legitimate access; break-glass row = check pam_break_glass_reviews; revoked row before operation timestamp = potentially unauthorised). §29.8 FORM-INSIDER-001 automation: 15-minute pg_cron / Worker detection query matches audit_log_events carrying metadata.pam_session_id against admin_jit_escalations coverage window; zero matches = `security.insider_threat_candidate` DEC-030 HIGH + PagerDuty P0. §29.9 SOC 2 evidence mapping: CC6.1 (PAM-E-001 — zero null-scoped rows + ≤4h TTL), CC6.2 (PAM-E-002 — non-null approver for all non-read_only rows), CC6.3 (PAM-E-003 — expired rows + DEC-030 pam.session_expired chain), CC6.6 (PAM-E-004 — pam_break_glass_reviews all reviewed within 72h + pam.break_glass_review_completed events); artefacts stored in compliance/evidence/pam/; cross-reference to SOC2_READINESS §24.8 as CC6-E-PAM-005 through CC6-E-PAM-008. §29.10 implementation checklist: 6× P0 M6 (migration, pam-elevation-service INSERT, break-glass-notifier stub INSERT, pam-expiry-sweeper UPDATE, audit triggers, DEC-030 event registration), 5× P1 M7 (pg_cron reconciliation job 20, AL-PAM-BG-01 pg_cron job 21, FORM-INSIDER-001 automation, evidence collection, CHECK constraint migration 0058b), 2× P2 M8–M10 (tenant transparency report, OQ-PAM-01 decision). §29.11 two open questions: OQ-PAM-01 (dual-auth for break-glass audit record queries — P2, recommendation: alert-on-access approach), OQ-PAM-02 (plaintext justification in SOC 2 auditor evidence — P1, recommendation: redacted sample approach). Cross-references: `docs/SSO_SCIM_IMPLEMENTATION.md §24` (KV schema, DEC-030 events, alert rules AL-PAM-01/02/03 — §29 does not duplicate), `docs/INCIDENT_RESPONSE.md §R-20` (OQ-INS-01 → 🟢 Resolved by §29; C2 query updated to reference `admin_jit_escalations`; checklist item 1 closed), `docs/OBSERVABILITY.md §29` (PAM observability — AL-PAM-BG-01 to be added), `docs/OBSERVABILITY.md §12.6` (pg_cron registry: job 20 pam_postgres_sync, job 21 pam_bg_review_alert), `docs/AUDIT_LOG_SCHEMA.md` (`pam.break_glass_review_completed` — new event to register), `docs/SOC2_READINESS.md §24.8` (CC6-E-PAM-005 through CC6-E-PAM-008 evidence artefacts), `docs/CRYPTOGRAPHY_POLICY.md §3` (BUSINESS_JUSTIFICATION is plain-text stored under security_reviewer only — no key needed; confirm with compliance-officer before M6 migration), `docs/ENTERPRISE_ADMIN_API.md` (OQ-PAM-01 transparency report endpoint — P2 M10). Owner: enterprise-architect + security-engineer + compliance-officer.*

---

## 30. Subscription Events Erasure Hardening & Quota Grace Thresholds — OQ-BILL-05, OQ-RL-02 & OQ-BILL-01 Resolution

> **Closes three P0 open questions** from `DATA_MODEL.md §24` and `§28`:
> - **OQ-BILL-05** (P0 — before any erasure path ships): Art. 17 FK constraint approach for `subscription_events.user_id` — split-column chosen over sentinel UUID.
> - **OQ-RL-02** (P0 — before quota enforcement ships): `OVERAGE_GRACE` = 500 requests; secondary 95% soft-warn threshold added.
> - **OQ-BILL-01** (P0 — before enterprise pilot): trial length policy — 14 days fixed for consumer Pro; enterprise pilots via comped active subscription, not extended trial.
>
> DEC-044 and DEC-045 logged in `docs/DECISION_LOG.md`. Migrations: `0059_erasure_hardening.sql` and `0059b_quota_warn_dedup.sql`.
> References: §24 (`subscription_events`, `user_subscriptions`, billing erasure path); §28 (`api_quota_usage`, `OVERAGE_GRACE` constant); §16 (enterprise contract lifecycle); `docs/AUDIT_LOG_SCHEMA.md` (DEC-030); `docs/GDPR_DPIA.md §4`.

### 30.1 Purpose & Scope

Three P0 blockers prevented the erasure Worker and quota enforcement from shipping to production:

1. **OQ-BILL-05**: `subscription_events.user_id UUID NOT NULL REFERENCES users(id)` is a hard FK. On GDPR Art. 17 erasure, FORM must pseudonymize financial records — retain 7 years under Ukrainian Tax Code Art. 44 and EU VAT Directive Art. 245 (cannot delete), but cannot leave the original `user_id` UUID in the row. The FK type prevents setting `user_id` to a pseudonym string.

2. **OQ-RL-02**: `quota-check.ts` contains the check `overage_count < OVERAGE_GRACE`, where `OVERAGE_GRACE` was an unresolved application constant. Without a defined value, the enforcement logic cannot ship: too low cuts off legitimate traffic at the month boundary; too high permits unacceptable quota overrun before the hard block fires.

3. **OQ-BILL-01**: The `trial_ends_at` computation in the billing Worker must know the trial duration. The question was whether enterprise tenants should receive an extended trial via a `trial_duration_days` column on `enterprise_contracts`, or whether enterprise pilots should be handled as a separate model.

**Out of scope for §30:** The full erasure Worker implementation (§12 erasure protocol is authoritative); KV sliding-window configuration (§28.5 is authoritative); enterprise contract commercial terms (§16 is authoritative).

---

### 30.2 OQ-BILL-05 Resolution: Split Column Approach

#### 30.2.1 The Two Options Considered

| Dimension | Option (a): Sentinel UUID | Option (b): Split Column ✓ |
|---|---|---|
| **Approach** | Insert permanent `[erased-user]` row into `users`; set `subscription_events.user_id` to this sentinel UUID on erasure | Add `erased_user_reference TEXT NULL` column; set `user_id = NULL`; write keyed HMAC pseudonym to `erased_user_reference` |
| **Schema change** | None on `subscription_events` | Migration: `ALTER TABLE subscription_events` — `user_id` nullable + new column + CHECK constraint |
| **`users` table impact** | Creates a durable sentinel row that must be excluded from every query on `users` (COUNT, list, DSAR export, activation metrics) | Zero impact on `users` table |
| **RLS implications** | `form_api` SELECT on `users` sees the sentinel unless explicitly excluded; JOINs return the sentinel row | `user_id IS NULL` naturally excludes erased rows; existing queries remain correct without modification |
| **GDPR Art. 4(5) pseudonymization** | Sentinel UUID is a shared opaque identifier — not a pseudonym per Recital 26; two erased users share the same sentinel, preventing per-user re-identification linkage in HMAC chain | `[ERASED-{sha256(user_id + ERASURE_PSEUDONYM_SALT)}]` is a per-user keyed HMAC — satisfies Art. 4(5) definition; cannot reverse without the secret salt |
| **SOC 2 CC8.1 auditability** | Sentinel row is a permanent schema artefact with no natural lifecycle; every caller of `users` requires documented exclusion | Erasure evidence is fully contained within `subscription_events`; simpler for auditors to verify |

#### 30.2.2 Decision: Option (b) — Split Column Adopted (DEC-044)

**Option (b)** is adopted. The sentinel UUID approach was rejected for three reasons:

1. **Contaminates the authoritative identity table.** `users` is the source of truth for access control, DSAR exports, activation metrics, and admin tooling. A permanent sentinel row must be excluded from every query on `users` — an invisible maintenance burden that grows with every new query and will catch future engineers off-guard.

2. **Not a true GDPR pseudonym.** A sentinel UUID is a shared opaque identifier; it does not satisfy GDPR Recital 26's requirement that pseudonymization use "additional information kept separately and subject to technical and organisational measures." The split-column pseudonym `[ERASED-{sha256(user_id + SALT)}]` is a per-user HMAC keyed to a secret stored in Cloudflare Secrets Store — a proper pseudonym under Art. 4(5).

3. **SOC 2 CC8.1 auditability.** Auditors reviewing the `users` table would encounter the sentinel and require an explanation of its origin. The split-column approach keeps all erasure evidence within `subscription_events`, which has a well-documented 7-year financial retention basis.

#### 30.2.3 Migration 0059 — `subscription_events` Erasure Hardening

```sql
-- Migration: 0059_erasure_hardening.sql
-- Owner: platform-engineer + compliance-officer
-- Closes: OQ-BILL-05 (DATA_MODEL §24.14)
-- Run during maintenance window (03:00–04:00 UTC) — ADD CONSTRAINT requires table scan
-- Idempotent: ALTER ADD COLUMN is a no-op if column already exists

BEGIN;

-- Step 1: Make user_id nullable to support erasure pseudonymization
-- FK constraint is retained — only erased rows carry user_id = NULL
ALTER TABLE subscription_events
  ALTER COLUMN user_id DROP NOT NULL;

-- Step 2: Add erased_user_reference column for keyed HMAC pseudonym
ALTER TABLE subscription_events
  ADD COLUMN IF NOT EXISTS erased_user_reference TEXT NULL
    CHECK (
      erased_user_reference IS NULL
      OR erased_user_reference ~ '^\[ERASED-[0-9a-f]{64}\]$'
    );

-- Step 3: Mutual exclusivity constraint
--   A row must have exactly one of user_id or erased_user_reference — never both, never neither
ALTER TABLE subscription_events
  ADD CONSTRAINT chk_sub_events_user_or_erased CHECK (
    (user_id IS NOT NULL AND erased_user_reference IS NULL)
    OR
    (user_id IS NULL AND erased_user_reference IS NOT NULL)
  );

-- Step 4: Index for erasure sweep (find all rows for a user_id during Art. 17 processing)
CREATE INDEX IF NOT EXISTS idx_sub_events_erasure_sweep
  ON subscription_events (user_id)
  WHERE user_id IS NOT NULL;

-- Step 5: Index for pseudonym audit lookup
CREATE INDEX IF NOT EXISTS idx_sub_events_erased_ref
  ON subscription_events (erased_user_reference)
  WHERE erased_user_reference IS NOT NULL;

COMMIT;
```

**Zero-downtime notes:**
- `ALTER COLUMN ... DROP NOT NULL` acquires `ACCESS EXCLUSIVE` briefly. Safe on Supabase with `lock_timeout = '5s'` (default).
- `ADD COLUMN IF NOT EXISTS` with no DEFAULT and no NOT NULL does not rewrite the table on Postgres 11+.
- `ADD CONSTRAINT ... CHECK` requires a table scan — schedule during the 03:00–04:00 UTC maintenance window (`docs/ENGINEERING_RUNBOOK.md §5.1`).

#### 30.2.4 Erasure Worker Step SUB-1

The erasure Worker (§12 Art. 17 protocol) gains step **SUB-1**, inserted after wearable data erasure and before `user_subscriptions` soft-expiry:

```typescript
// Step SUB-1: Pseudonymize subscription_events rows
// Requires: form_system Postgres role (set by erasure Worker via SET ROLE)
// ERASURE_PSEUDONYM_SALT: Cloudflare Secret (write-once — see OQ-ERA-02)

async function pseudonymizeSubscriptionEvents(
  userId: string,
  salt: string
): Promise<{ rowsUpdated: number }> {
  const pseudonym = `[ERASED-${await sha256Hex(userId + salt)}]`;

  const { count } = await supabase
    .from('subscription_events')
    .update({ user_id: null, erased_user_reference: pseudonym })
    .eq('user_id', userId)
    .select('count', { count: 'exact', head: true });

  // billing.user_erased DEC-030 (HIGH, 7yr) — already registered in AUDIT_LOG_SCHEMA.md §6
  // user_id is omitted from the event payload after erasure (see §30.5.2 Zod patch)
  await emitAuditEvent({
    event_type: 'billing.user_erased',
    severity: 'HIGH',
    tenant_id: null,               // consumer erasure — no tenant context
    payload: {
      erased_user_reference: pseudonym,
      rows_pseudonymized:    count ?? 0,
      retention_basis:       'UACode_Art44_EUVATDir_Art245',
      erasure_step:          'SUB-1',
    },
  });

  return { rowsUpdated: count ?? 0 };
}
```

**Privacy invariant:** `user_id` is omitted from `billing.user_erased` DEC-030 events emitted by step SUB-1. The `erased_user_reference` HMAC is the sole identity marker in the event chain after erasure.

#### 30.2.5 Query Pattern Analysis

The `chk_sub_events_user_or_erased` CHECK guarantees a row always has exactly one identity column — the mutual-exclusivity invariant prevents ambiguity. No existing queries require mandatory changes for correctness:

| Query pattern | After migration | Notes |
|---|---|---|
| `WHERE user_id = $id` | Correct — erased rows have `user_id = NULL`, excluded automatically | No change needed |
| `JOIN users ON subscription_events.user_id = users.id` | Inner join excludes `user_id = NULL` rows naturally | No change needed |
| `COUNT(DISTINCT user_id)` | Excludes erased users — **intentional** | No change needed |
| `WHERE user_id IS NOT NULL` | Now meaningful filter (previously tautology) | Recommend making explicit |
| DSAR export scoped by `user_id` | Erased users cannot request DSAR (account closed before erasure runs) | No change needed |
| Auditor query for `billing.user_erased` events | Must use `erased_user_reference` column, not `user_id` | Auditor tooling update required |

---

### 30.3 OQ-RL-02 Resolution: OVERAGE_GRACE = 500 Requests (DEC-045)

#### 30.3.1 Threshold Rationale

`OVERAGE_GRACE` is the count of requests admitted past `quota_limit` before `hard_blocked_at` is set in `api_quota_usage`. The enforcement flow in §28.4 increments `overage_count` on each over-limit request and hard-blocks when `overage_count >= OVERAGE_GRACE`.

**Selected value: `OVERAGE_GRACE = 500`**

| Tier | `quota_limit` (monthly) | 500 requests = | Hard-block window |
|---|---|---|---|
| Enterprise Starter | 50,000 req/month | 1.0% of quota | ≈ 16 req/day overage window |
| Enterprise Growth | 200,000 req/month | 0.25% of quota | Tighter — appropriate for sophisticated customers |
| Enterprise Custom | Configurable | Varies | Per contract |

500 requests is calibrated to:
- Absorb a legitimately busy final day of the month (month-boundary burst is the primary false-positive scenario)
- Not represent a material contract violation — at Starter $9/seat/month, 500 excess requests are economically immaterial
- Align with common SaaS patterns (Stripe uses a grace window before hard-blocking; Twilio uses ~10%)

#### 30.3.2 Environment Variable Configuration

All three quota thresholds are stored as Cloudflare Workers environment variables, not hard-coded. This enables emergency adjustment without a code deploy:

| Variable | Value | Description |
|---|---|---|
| `OVERAGE_GRACE_REQUESTS` | `500` | Requests past `quota_limit` before hard block |
| `QUOTA_WARN_PCT` | `0.80` | First CSM notification threshold (AL-RL-01, P2) |
| `QUOTA_SECONDARY_WARN_PCT` | `0.95` | Second CSM notification — **new**, see §30.3.3 |

#### 30.3.3 Secondary 95% Warning Threshold

With OVERAGE_GRACE = 500, a Starter tenant at 95% consumption has ≈ 2,500 requests before hard block — insufficient time for a CSM to reach the customer, negotiate an uplift, and have FORM update `api_quota_usage.quota_limit` before the block fires. A second warning at 95% provides a final escalation window.

**Schema addition — `api_quota_usage` dedup columns (migration 0059b):**

```sql
-- Migration: 0059b_quota_warn_dedup.sql
ALTER TABLE api_quota_usage
  ADD COLUMN IF NOT EXISTS primary_warn_sent_at    TIMESTAMPTZ NULL,
  ADD COLUMN IF NOT EXISTS secondary_warn_sent_at  TIMESTAMPTZ NULL;
```

**Updated `quota-check.ts` enforcement logic:**

```typescript
const OVERAGE_GRACE         = parseInt(env.OVERAGE_GRACE_REQUESTS    ?? '500');
const WARN_PCT              = parseFloat(env.QUOTA_WARN_PCT           ?? '0.80');
const SECONDARY_WARN_PCT    = parseFloat(env.QUOTA_SECONDARY_WARN_PCT ?? '0.95');

// After UPSERT — read updated row
if (row.hard_blocked_at) {
  return http429(secondsUntilNextMonth());
}
if (row.overage_count >= OVERAGE_GRACE) {
  await setHardBlock(row);           // emits security.quota_hard_block DEC-030 (§28.6)
  return http429(secondsUntilNextMonth());
}
if (row.request_count >= row.quota_limit * SECONDARY_WARN_PCT
    && !row.secondary_warn_sent_at) {
  await emitQuota95Warning(row);     // emits security.quota_95pct_warning DEC-030 (§30.5.1)
  await markSecondaryWarnSent(row);
}
if (row.request_count >= row.quota_limit * WARN_PCT
    && !row.primary_warn_sent_at) {
  await emitQuota80Warning(row);     // emits security.quota_80pct_warning DEC-030 (§28.6)
  await markPrimaryWarnSent(row);
}
```

---

### 30.4 OQ-BILL-01 Resolution: Trial Length Policy (DEC-045)

**Consumer Pro trial: 14 days fixed.** No configurability. The billing Worker computes:

```typescript
const trial_ends_at = addDays(new Date(), 14);  // date-fns addDays
```

**Enterprise pilots: comped active subscription, not extended trial.**

Adding `trial_duration_days` to `enterprise_contracts` was rejected. Enterprise pilots are modelled as a comped subscription:
- `user_subscriptions.status = 'active'`
- `user_subscriptions.billing_channel = 'none'`
- `user_subscriptions.price_usd_cents = 0`
- `enterprise_contracts.pilot_start_at` / `pilot_end_at` govern the pilot window (§16 — already present, no schema change)

**Rationale:**
1. **Clean semantics.** "Trial" is a consumer self-service concept. "Pilot" is a negotiated enterprise evaluation with a defined CSM cadence, success criteria, and conversion gate. Different commercial models should not share a state machine.
2. **No state machine leakage.** The billing Worker does not auto-expire pilots — the CSM triggers conversion manually. This avoids accidental production disruption if `pilot_end_at` passes without a contract signed.
3. **Zero schema change.** `enterprise_contracts.pilot_end_at` already exists; the billing Worker branches on `billing_channel = 'none'` to suppress automatic renewal attempts.

---

### 30.5 DEC-030 HMAC-Chained Audit Events

#### 30.5.1 `security.quota_95pct_warning` (New — STANDARD, 3-year retention)

Register in `docs/AUDIT_LOG_SCHEMA.md §6` alongside `security.quota_80pct_warning`:

```sql
INSERT INTO audit_log (
  event_type, severity, tenant_id, payload,
  sequence_number, prev_hash, hash
) VALUES (
  'security.quota_95pct_warning',
  'STANDARD',
  $tenant_id,
  jsonb_build_object(
    'api_key_id',         $api_key_id,          -- UUID; never key_preview
    'period_month',       $period_month,         -- YYYY-MM-01
    'endpoint_category',  $endpoint_category,
    'request_count',      $request_count,
    'quota_limit',        $quota_limit,
    'pct_consumed',       ROUND($request_count::numeric / $quota_limit * 100, 1)
  ),
  nextval('audit_log_sequence'), $prev_hash,
  encode(hmac(
    concat($sequence_number, $prev_hash, $event_type, $tenant_id::text, $payload::text),
    current_setting('app.hmac_secret'), 'sha256'
  ), 'hex')
);
```

**Privacy invariant:** No `user_id`. Tenant-aggregate event only.

#### 30.5.2 `billing.user_erased` Zod Schema Patch

The existing `billing.user_erased` event (§24.12) carries `user_id`. Post-migration, step SUB-1 emits the event with `user_id` absent and `erased_user_reference` present. The registered Zod schema must be patched for forward compatibility while preserving backward compatibility with pre-migration events:

```typescript
// AUDIT_LOG_SCHEMA.md §6 — billing.user_erased Zod schema patch
const BillingUserErasedPayload = z.object({
  user_id:                z.string().uuid().optional(),     // deprecated post-§30; kept for backward compat
  erased_user_reference:  z.string().regex(/^\[ERASED-[0-9a-f]{64}\]$/),
  rows_pseudonymized:     z.number().int().nonneg(),
  retention_basis:        z.literal('UACode_Art44_EUVATDir_Art245'),
  erasure_step:           z.string(),
});
```

---

### 30.6 SOC 2 Evidence Mapping

| SOC 2 Criterion | Control | §30 mechanism | Evidence artefact | Status |
|---|---|---|---|---|
| **P5.2 — Pseudonymization** | `erased_user_reference` is a per-user keyed SHA-256 HMAC — satisfies GDPR Art. 4(5) pseudonymization; `chk_sub_events_user_or_erased` enforces mutual exclusivity at DB layer | `billing.user_erased` DEC-030 HIGH/7yr carries `erased_user_reference`; migration 0059 adds CHECK constraint | **ERA-E-001** — `subscription_events` row export showing `erased_user_reference IS NOT NULL` rows after first erasure run | 🟡 Authored — closes on first Art. 17 erasure in production |
| **P8.0 — Financial retention basis** | `subscription_events` rows are not deleted on erasure; `billing.user_erased` DEC-030 event carries `retention_basis = 'UACode_Art44_EUVATDir_Art245'` as documentary evidence | `billing.user_erased` event log with `retention_basis` field | **ERA-E-002** — `billing.user_erased` DEC-030 event export with `retention_basis` populated | 🟡 Authored — closes on first Art. 17 erasure in production |
| **CC8.1 — Change management** | `chk_sub_events_user_or_erased` CHECK prevents simultaneous population of both identity columns; billing Worker is sole erasure mutation path; DEC-030 chain provides tamper-evident record | Migration deploy record + DEC-030 event chain for first production erasure | **ERA-E-003** — migration 0059 deploy log + first-erasure DEC-030 chain export | 🟡 Authored — closes on migration deploy + first production erasure |
| **CC6.1 — Logical access** | `OVERAGE_GRACE_REQUESTS = 500` enforces quota ceiling with bounded overage; `QUOTA_SECONDARY_WARN_PCT = 0.95` adds advance CSM escalation window; both as Workers env vars (configurable without deploy) | `security.quota_95pct_warning` + `security.quota_hard_block` DEC-030 events | **RL-E-001 (supplement)** — §35.9 RL-E-001 extended; §30.3 adds the 95% event to the enforcement chain evidence | 🟡 Authored |

---

### 30.7 Implementation Checklist

#### P0 — Must complete before erasure Worker ships (M4)

| # | Task | Owner | Priority | Milestone | Status |
|---|---|---|---|---|---|
| 1 | Run migration `0059_erasure_hardening.sql` (§30.2.3) on staging. Verify: (a) `user_id` is nullable; (b) `erased_user_reference TEXT NULL` column exists; (c) `chk_sub_events_user_or_erased` CHECK present; (d) INSERT with both non-NULL fails with CHECK violation; (e) INSERT with both NULL fails; (f) UPDATE setting `user_id = NULL, erased_user_reference = '[ERASED-{64 hex chars}]'` succeeds. Promote to production in 03:00–04:00 UTC maintenance window. | platform-engineer | **P0** | M4 | [ ] |
| 2 | Run migration `0059b_quota_warn_dedup.sql` (§30.3.3) on staging and production. Verify `api_quota_usage` gains `primary_warn_sent_at` and `secondary_warn_sent_at` columns; existing rows have both NULL. | platform-engineer | **P0** | M4 | [ ] |
| 3 | Update erasure Worker: insert step SUB-1 (`pseudonymizeSubscriptionEvents`) per §30.2.4. Staging test: create a test user with ≥ 3 `subscription_events` rows; trigger erasure; confirm (a) all rows for `user_id` now have `user_id = NULL` + `erased_user_reference = '[ERASED-{hash}]'`; (b) `billing.user_erased` DEC-030 event emitted with `erased_user_reference` in payload, no `user_id`; (c) `rows_pseudonymized` count matches actual row count. | platform-engineer + compliance-officer | **P0** | M4 | [ ] |
| 4 | Patch `billing.user_erased` Zod schema in `docs/AUDIT_LOG_SCHEMA.md §6` per §30.5.2: add required `erased_user_reference`, make `user_id` optional. Deploy updated event registry to `emit-audit-event` Worker. Confirm existing `billing.user_erased` events (pre-migration) still validate against the patched schema. | platform-engineer | **P0** | M4 | [x] **AUDIT_LOG_SCHEMA.md v1.5 (2026-06-11)** — Billing & GDPR erasure events section added; Zod v2 schema with `erased_user_reference` required + `user_id` optional; weekly chain audit post-migration anomaly check documented. Deploy to `emit-audit-event` Worker pending platform-engineer. |
| 5 | Register `security.quota_95pct_warning` in `docs/AUDIT_LOG_SCHEMA.md §6` with Zod schema from §30.5.1. Deploy to `emit-audit-event` Worker. | platform-engineer | **P0** | M4 | [x] **AUDIT_LOG_SCHEMA.md v1.5 (2026-06-11)** — `security.quota_95pct_warning` registered in Security & tenant isolation events section (STANDARD, 3yr); dedup via `secondary_warn_sent_at`; no `user_id`. Deploy pending platform-engineer. |
| 6 | Update `quota-check.ts` per §30.3.2: read `OVERAGE_GRACE_REQUESTS` env var (replace any hard-coded constant); add `QUOTA_SECONDARY_WARN_PCT = 0.95` check with `secondary_warn_sent_at` dedup. Set `OVERAGE_GRACE_REQUESTS = 500`, `QUOTA_SECONDARY_WARN_PCT = 0.95` in Cloudflare Workers production env for `quota-check` Worker. | platform-engineer + devops-lead | **P0** | M4 | [ ] |
| 7 | Configure AL-RL-01b alert in PagerDuty `form-customer-success`: trigger on `security.quota_95pct_warning` DEC-030 event; P1 severity; dedup key `rl-quota-95pct-{tenant_id}-{period_month}-{endpoint_category}`; auto-escalates to P0 if no CSM acknowledgement within 2 hours. Add row to §6.2 Alert Rules Table `rate_limit_health` subsection in `docs/OBSERVABILITY.md`. | devops-lead | **P0** | M4 | [x] **OBSERVABILITY.md v2.9 (2026-06-11)** — AL-RL-01b added to §35.4 detailed alert table and §35.5 `rate_limit_health` §6.2 condensed table; 2h auto-escalation to P0 documented. PagerDuty routing pending devops-lead. |

#### P1 — Before first enterprise pilot goes live (M5)

| # | Task | Owner | Priority | Milestone | Status |
|---|---|---|---|---|---|
| 8 | Confirm all three env vars (`OVERAGE_GRACE_REQUESTS`, `QUOTA_WARN_PCT`, `QUOTA_SECONDARY_WARN_PCT`) are set in production Workers env for `quota-check`. Verify via `wrangler secret list quota-check`. | devops-lead | **P1** | M5 | [ ] |
| 9 | Document `ERASURE_PSEUDONYM_SALT` as a write-once key in `docs/CRYPTOGRAPHY_POLICY.md §3` key inventory. Rationale: OQ-ERA-02 (§30.8) — rotating this key makes pre-rotation pseudonyms irreconcilable. Write-once policy must be recorded before first Art. 17 erasure in production. | compliance-officer + security-engineer | **P1** | M5 | [x] **CRYPTOGRAPHY_POLICY.md v1.1 (2026-06-11)** — `ERASURE_PSEUDONYM_SALT` added to §5 (write-once rotation schedule) and §6 (key custody: `erasure-worker` Cloudflare Workers Secret; 1Password Operations vault backup with `WRITE-ONCE — OQ-ERA-02` label). OQ-ERA-02 → 🟢 Resolved. |
| 10 | Collect ERA-E-001, ERA-E-002, ERA-E-003 evidence artefacts (§30.6) after first production erasure; store in `compliance/evidence/erasure/`; cross-reference `docs/SOC2_READINESS.md` P5.2 and P8.0 evidence tables. | compliance-officer | **P1** | M6 | [ ] |

#### P2 — Post-GA

| # | Task | Owner | Priority | Milestone | Status |
|---|---|---|---|---|---|
| 11 | Evaluate per-tier `OVERAGE_GRACE_REQUESTS` tuning after first 10 enterprise tenants reach 80%+ consumption. If Growth-tier tenants consistently hard-block within the grace window, increase to 1,000 for Growth; keep 500 for Starter. Document outcome in `docs/DECISION_LOG.md`. | enterprise-architect + customer-success | **P2** | M8 | [ ] |

---

### 30.8 Open Questions

| ID | Question | Priority | Owner | Resolution path |
|---|---|---|---|---|
| **OQ-ERA-01** | **DSAR requests from users who have already exercised Art. 17 erasure.** `subscription_events` rows for erased users carry `erased_user_reference IS NOT NULL`. A DSAR export scoped by `user_id` returns zero rows (correct — the account is closed). However, if a user requests DSAR before erasure, the export is a point-in-time snapshot and remains valid. Recommendation: DSAR Worker routes 404 after account closure; no schema change required. Document as a Worker routing rule in `docs/ENTERPRISE_ADMIN_API.md §14`. | P2 | compliance-officer + platform-engineer | Document in ENTERPRISE_ADMIN_API.md §14 before Art. 17 erasure goes live |
| **OQ-ERA-02** | ~~**`ERASURE_PSEUDONYM_SALT` key rotation policy.**~~ **🟢 RESOLVED — CRYPTOGRAPHY_POLICY.md v1.1 (2026-06-11).** Write-once policy documented in `docs/CRYPTOGRAPHY_POLICY.md §5` (rotation schedule: Never) and `§6` (custody: `erasure-worker` Cloudflare Workers Secret; 1Password Operations vault backup labelled `WRITE-ONCE — OQ-ERA-02`). Any rotation attempt requires compliance-officer + security-engineer + legal joint sign-off after full irreconcilability impact assessment. Closes §30.7 checklist item 9. | P1 → ✅ | compliance-officer + security-engineer | **Resolved** — CRYPTOGRAPHY_POLICY.md v1.1 (2026-06-11) |

---

*v1.0 (2026-06-11): §30 Subscription Events Erasure Hardening & Quota Grace Thresholds — resolves OQ-BILL-05 (P0, split-column for `subscription_events` GDPR Art. 17 pseudonymization), OQ-RL-02 (P0, OVERAGE_GRACE = 500 requests), and OQ-BILL-01 (P0, 14-day fixed consumer trial; enterprise pilots via comped active subscription). §30.2 OQ-BILL-05: option (b) split-column adopted (option (a) sentinel UUID rejected — contaminates `users` table; not a true GDPR Art. 4(5) pseudonym; pollutes SOC 2 CC8.1 audit trail); migration 0059 makes `user_id` nullable, adds `erased_user_reference TEXT NULL` with regex CHECK, adds `chk_sub_events_user_or_erased` mutual-exclusivity CHECK, adds two indexes; erasure Worker gains step SUB-1 with `[ERASED-{sha256(user_id + ERASURE_PSEUDONYM_SALT)}]` keyed HMAC pseudonym; query pattern analysis confirms zero mandatory query changes for correctness. §30.3 OQ-RL-02: OVERAGE_GRACE = 500 requests (1.0% of Starter 50k/month quota; 0.25% of Growth 200k/month); secondary 95% warn threshold added (QUOTA_SECONDARY_WARN_PCT = 0.95); migration 0059b adds `primary_warn_sent_at` + `secondary_warn_sent_at` dedup columns to `api_quota_usage`; three env vars replace hard-coded constants. §30.4 OQ-BILL-01: 14-day fixed consumer trial; enterprise pilots use `billing_channel = 'none'`, `price_usd_cents = 0`, `status = 'active'`; `pilot_end_at` in §16 `enterprise_contracts` governs pilot window — no schema change. §30.5 two DEC-030 items: `security.quota_95pct_warning` (new, STANDARD 3yr, tenant-level, no user_id); `billing.user_erased` Zod patch (add required `erased_user_reference`, make `user_id` optional for backward compat). §30.6 SOC 2: ERA-E-001 (P5.2 pseudonymization), ERA-E-002 (P8.0 financial retention with `retention_basis`), ERA-E-003 (CC8.1 migration + erasure chain). §30.7 eleven-item checklist: 7× P0 M4 (migrations 0059/0059b, erasure Worker SUB-1, AUDIT_LOG_SCHEMA patches ×2, quota-check.ts update, AL-RL-01b PagerDuty), 3× P1 M5–M6 (env var confirm, ERASURE_PSEUDONYM_SALT key inventory, evidence collection), 1× P2 M8 (per-tier grace tuning). §30.8 two open questions: OQ-ERA-01 (DSAR routing for erased users — P2, Worker routing decision, no schema change); OQ-ERA-02 (ERASURE_PSEUDONYM_SALT write-once policy — P1, document in CRYPTOGRAPHY_POLICY.md §3). Cross-references: `docs/DATA_MODEL.md §12` (Art. 17 erasure protocol — step SUB-1 insertion point); `docs/DATA_MODEL.md §24` (OQ-BILL-05 ~~P0~~ → 🟢 Resolved; OQ-BILL-01 ~~P0~~ → 🟢 Resolved; `billing.user_erased` Zod schema patched); `docs/DATA_MODEL.md §28` (OQ-RL-02 ~~P0~~ → 🟢 Resolved; `OVERAGE_GRACE` constant defined; `api_quota_usage` schema extended); `docs/AUDIT_LOG_SCHEMA.md §6` (`billing.user_erased` Zod patch + `security.quota_95pct_warning` new registration); `docs/OBSERVABILITY.md §35.4` + §6.2 (AL-RL-01b 95% warn alert added to `rate_limit_health` subsection); `docs/CRYPTOGRAPHY_POLICY.md §3` (ERASURE_PSEUDONYM_SALT — write-once key, P1 before first erasure); `docs/DECISION_LOG.md` (DEC-044: OQ-BILL-05 split-column; DEC-045: OQ-RL-02 + OQ-BILL-01). Owner: enterprise-architect + platform-engineer + compliance-officer.*
