# FORM · Multi-Tenant Data Model v0.6

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

