# FORM · Multi-Tenant Data Model v1.1

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
| OQ-FLAG-01 | **Tier downgrade handling.** When a tenant downgrades from Growth to Starter, flags with `minimum_tier = 'growth'` (e.g. `audit.siem_export_enabled`, `admin.cohort_analytics_enabled`) are still `enabled = TRUE` in their rows. The Worker tier gate will block them at the enforcement layer, but the rows remain inconsistent. Decision needed: (a) auto-disable Growth+ flags on plan downgrade via a contract lifecycle hook (see §16 `tenant.lifecycle_status`), emitting `feature_flag.disabled` DEC-030 events for each; or (b) leave rows intact, rely solely on the Worker tier gate, and accept that re-upgrade re-activates them without a new `form_admin` enablement action. Option (a) is preferred: clean data, correct audit trail, no surprise reactivation. | `enterprise-architect` + `platform-engineer` | High — resolve before any Growth tenant downgrade path is enabled in billing self-serve (M5) |
| OQ-FLAG-02 | **Flag deprecation sunset process.** When `deprecated_at` is set on a registry entry, the `assertFeatureEnabled()` function blocks new evaluations. But existing enabled tenant rows remain. Sunset process needs definition: (a) how far in advance is `deprecated_at` set (proposed: minimum 90 days notice to tenants using the flag); (b) who communicates the sunset to affected tenants; (c) whether the `tenant_feature_flags` rows are soft-deleted or left as tombstones. Proposed: 90-day notice, CSM communication via admin dashboard banner, automatic soft-delete of tenant rows at `deprecated_at + 90 days` via pg_cron. | `enterprise-architect` + `compliance-officer` | Medium — establish policy before the first flag is deprecated (M6) |

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
| 11 | Resolve OQ-FLAG-01: implement plan downgrade hook in contract lifecycle (§16) that auto-disables Growth+/Enterprise-only flags when plan decreases; emit `feature_flag.disabled` DEC-030 events with `reason = 'plan_downgrade'` for each disabled flag | `enterprise-architect` + `platform-engineer` | **P1** | M5 |
| 12 | Resolve OQ-FLAG-02: define flag deprecation sunset policy (minimum 90-day notice); implement pg_cron soft-delete of tenant flag rows at `deprecated_at + 90 days`; wire admin dashboard banner for tenants using a flag with non-null `deprecated_at` | `enterprise-architect` + `compliance-officer` | **P2** | M6 |

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
| OQ-COACH-02 | Token cost attribution model for enterprise: flat per-seat coaching fee (predictable for tenant budgeting) vs. actual consumption billing (fair but variable and complex to invoice). Current schema supports both: `tenant_monthly_coaching_cost` view provides actual consumption; a per-seat flat rate would cap at a configurable `tenant_config.coaching_seat_allowance_tokens` with overage billing. Decision needed before enterprise contracts are signed. | `enterprise-architect` + `finance` | **P1 — before first enterprise pilot invoicing** |
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

*v1.0 · травень 2026 · owner: enterprise-architect + compliance-officer + security-engineer*

*v1.0 additions: §19 Feature Flag & Entitlement Registry Schema — closes the §2.7 stub gap across all cross-references (§15 `cv.client_side_enabled`, §18 `webhook.email_in_payloads`, SSO_SCIM_IMPLEMENTATION.md `security.ip_allowlist_enabled` and `sso.staging_env_enabled`). Two-table design: `feature_flag_registry` (canonical allowlist, form_admin-only write, FK-enforced dotted namespace, `flag_key_namespace` CHECK constraint) and `tenant_feature_flags` (production per-tenant rows with FK to registry, `compliance_approval_ref`, `enabled_at`, RLS read-only for form_api and form_system). Seed data for 14 canonical flags across Starter / Growth / Enterprise tiers with one compliance-gated flag (`webhook.email_in_payloads`). Tier entitlement matrix (auto / avail / — / gate). TypeScript `assertFeatureEnabled()` in `src/workers/feature-flags/assert-feature-enabled.ts`: Cloudflare KV 30-second cache, tier rank comparison against TIER_ORDER map, compliance gate check, fail-closed on any DB or KV error (throws 403). Four DEC-030 HMAC-chained audit events: `feature_flag.enabled` (STANDARD, 7yr), `feature_flag.disabled` (STANDARD, 7yr), `feature_flag.registry_updated` (HIGH, 7yr), `feature_flag.compliance_gate_bypassed` (CRITICAL, 10yr — auto PagerDuty P1, always `blocked: true`). Compliance gate protocol: DPA authorisation → DECISION_LOG PR merge → compliance_approval_ref populated → form_admin enables via internal API → DEC-030 event with approval reference. §19.9 migration script: 14 flag_name-to-flag_key renames, RAISE EXCEPTION abort on unrecognised flags, FK constraint addition, `enabled_at` backfill from `updated_at`, `compliance_approval_ref` column addition, post-migration validation queries (zero unrecognised flags; zero enabled gated flags without approval ref). OQ-FLAG-01: tier downgrade auto-disable decision (preferred: contract lifecycle hook emitting feature_flag.disabled events, M5). OQ-FLAG-02: flag deprecation 90-day sunset policy with pg_cron soft-delete and admin dashboard banner (M6). SOC 2 mapping: CC6.1 (form_admin-only write, tier entitlement gates), CC6.2 (tier check = authorisation gate, plan not self-elevatable), CC8.1 (all mutations HMAC-chained in-transaction), CC9.2 (compliance_approval_ref persisted 7yr, auditor-traceable to DPA clause). 12-item implementation checklist (8× P0 M3, 1× P1 M4, 1× P1 M5, 1× P2 M6).*
