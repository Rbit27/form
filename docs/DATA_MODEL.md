# FORM · Multi-Tenant Data Model v0.5

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
| 9.2 | `raw_cv_data` column size: JSONB for pose keypoints can grow large. Column to separate blob store (S3)? | platform-engineer | High — evaluate at 10k workouts/day |
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

> Owner: `platform-engineer` + `enterprise-architect`. Review: on any new wearable source addition or quarterly.
> References: §5 (Data Classification), §6 (Privacy Floor), §8.3 (Art. 17 Erasure), docs/SOC2_READINESS.md §35.6.3, docs/SSO_SCIM_IMPLEMENTATION.md §14.

Wearable data — HRV, resting heart rate, sleep, recovery scores — is GDPR Art. 9 special category health data. This section defines the storage schema, RLS policies, consent gates, GDPR constraints, cross-source normalization, and sync architecture for wearable integrations. **Wearable data is classified as Sensitive (§5) and may never appear in PostHog analytics events, ClickHouse warehouse tables, or any tenant-visible aggregate other than the privacy-preserving view defined in §14.4.**

---

### 14.1 Supported Sources and Data Types

| Source | Platform | Auth mechanism | Data types exposed | HRV metric | Notes |
|---|---|---|---|---|---|
| **HealthKit** | iOS only | HealthKit Background Delivery (no OAuth) | HRV (RMSSD), Resting HR, Sleep analysis (stages), Active energy (kcal), Steps | RMSSD in ms — gold standard | Background delivery pushes new data to FORM backend without polling; requires HealthKit entitlement + usage strings |
| **Health Connect** | Android only | Health Connect Permission API (no OAuth) | HRV (RMSSD), Resting HR, Sleep stages, Active energy, Steps | RMSSD in ms — comparable to HealthKit | WorkManager background sync; 15-min minimum interval per Android policy; requires `READ_HEALTH_DATA` permission |
| **Whoop** | iOS + Android | OAuth 2.0 (PKCE) | Recovery score (0–100), Strain (0–21), HRV (ms), Resting HR, Sleep performance % | RMSSD in ms — computed from overnight 5-minute windows; Whoop's HRV algorithm differs from HealthKit; do not cross-compare | API polling every 2h via Cloudflare Worker cron; Whoop webhook not available on standard API tier |
| **Oura Ring** | iOS + Android | OAuth 2.0 (PKCE) | Readiness score (0–100), HRV (RMSSD ms), Resting HR, Sleep stages, Temperature deviation (°C) | RMSSD in ms — comparable to HealthKit for trend analysis; absolute values may differ | API polling every 2h; Oura personal API token (v2); EU data residency available |
| **Garmin** | iOS + Android | OAuth 1.0a (Garmin Connect API) | Body Battery (0–100), HRV Status (weekly aggregate — not daily RMSSD), Resting HR, Sleep duration, VO2max estimate | Weekly HRV Status only — not daily RMSSD; cannot be compared to Whoop/Oura/HealthKit daily readings | API polling every 2h; Garmin Health API requires partner agreement; HRV Status is a 7-day rolling aggregate, not a single-night reading |

**Cross-device HRV comparison is unreliable.** Whoop and Oura RMSSD are nominally comparable for trend analysis within each source. Garmin HRV Status is a weekly aggregate and is not comparable to nightly RMSSD values. Victor's coaching context never aggregates HRV across sources or compares user readings to population norms.

---

### 14.2 `wearable_readings` Table Schema

```sql
-- Classification: SENSITIVE — GDPR Art. 9 (health data: HR, HRV, sleep, biometric recovery scores)
-- RLS: enforced — users see only their own rows; tenant aggregates via §14.4 view only
-- Retention: 2 years from recorded_at (§35.6.3 proposed schedule; Supabase TTL migration required)
CREATE TABLE wearable_readings (
  id                  UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id             UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  tenant_id           UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,

  source              TEXT NOT NULL
                        CHECK (source IN (
                          'healthkit',
                          'health_connect',
                          'whoop',
                          'oura',
                          'garmin'
                        )),

  reading_type        TEXT NOT NULL
                        CHECK (reading_type IN (
                          'hrv_rmssd',             -- RMSSD in ms (HealthKit, Health Connect, Whoop, Oura)
                          'hrv_weekly_status',     -- Garmin weekly HRV aggregate (ordinal: 'balanced'|'low'|'unbalanced'|'poor')
                          'resting_hr',            -- beats per minute
                          'sleep_duration_min',    -- total sleep in minutes
                          'sleep_efficiency_pct',  -- sleep efficiency 0–100 (Whoop, Oura)
                          'sleep_stage_deep_min',  -- deep sleep minutes (HealthKit, Oura)
                          'sleep_stage_rem_min',   -- REM sleep minutes (HealthKit, Oura)
                          'recovery_score',        -- 0–100 (Whoop Recovery, Oura Readiness)
                          'strain',                -- 0–21 (Whoop only)
                          'body_battery',          -- 0–100 (Garmin only)
                          'temperature_deviation_c', -- skin temp deviation °C (Oura only)
                          'steps',                 -- daily step count
                          'active_energy_kcal',    -- active calories burned
                          'vo2max_estimate'        -- ml/kg/min (Garmin only)
                        )),

  value_numeric       NUMERIC,                     -- normalized SI-unit value; NULL for ordinal types
  value_text          TEXT,                        -- ordinal values (e.g. Garmin HRV Status: 'balanced')
  value_unit          TEXT NOT NULL,               -- 'ms', 'bpm', 'min', 'pct', 'score', 'kcal', 'ml/kg/min', 'C', 'ordinal'

  recorded_at         TIMESTAMPTZ NOT NULL,        -- when the device recorded the reading (not sync time)
  synced_at           TIMESTAMPTZ NOT NULL DEFAULT now(),

  source_reading_id   TEXT,                        -- external ID from source API; used for idempotency
  metadata            JSONB,                       -- source-specific extras (sleep stage breakdown, Whoop cycle ID, etc.)

  deleted_at          TIMESTAMPTZ,                 -- soft delete during 30-day Art. 17 window; hard delete by nightly job

  CONSTRAINT wearable_readings_idempotency
    UNIQUE (user_id, source, reading_type, recorded_at)
);

-- Composite index: RLS filter + coaching context query (last 7 days per user)
CREATE INDEX idx_wearable_user_type_time
  ON wearable_readings (user_id, reading_type, recorded_at DESC)
  WHERE deleted_at IS NULL;

-- Tenant-level aggregate queries (§14.4 view uses this)
CREATE INDEX idx_wearable_tenant_type_time
  ON wearable_readings (tenant_id, reading_type, recorded_at DESC)
  WHERE deleted_at IS NULL;
```

**`source_reading_id` + the UNIQUE constraint on `(user_id, source, reading_type, recorded_at)` ensure idempotent upserts.** A sync that re-delivers the same reading (common on Health Connect and during polling retries) produces no duplicate rows.

---

### 14.3 RLS Policies for `wearable_readings`

```sql
-- Enable RLS
ALTER TABLE wearable_readings ENABLE ROW LEVEL SECURITY;
ALTER TABLE wearable_readings FORCE ROW LEVEL SECURITY;

-- Users: read and write own rows only
CREATE POLICY wearable_user_select ON wearable_readings
  FOR SELECT TO form_api
  USING (
    user_id = current_setting('app.current_user_id')::UUID
    AND deleted_at IS NULL
  );

CREATE POLICY wearable_user_insert ON wearable_readings
  FOR INSERT TO form_api
  WITH CHECK (
    user_id = current_setting('app.current_user_id')::UUID
    AND tenant_id = current_setting('app.current_tenant_id')::UUID
  );

CREATE POLICY wearable_user_update ON wearable_readings
  FOR UPDATE TO form_api
  USING (user_id = current_setting('app.current_user_id')::UUID)
  WITH CHECK (user_id = current_setting('app.current_user_id')::UUID);

-- Soft-delete (sets deleted_at); hard delete is form_admin only
CREATE POLICY wearable_user_soft_delete ON wearable_readings
  FOR UPDATE TO form_api
  USING (user_id = current_setting('app.current_user_id')::UUID);

-- Coach role: read readings for consented users within the same tenant
-- Consent check: user must have 'wearable' in consent.categories_consented (§35.4.2)
CREATE POLICY wearable_coach_select ON wearable_readings
  FOR SELECT TO form_api
  USING (
    tenant_id = current_setting('app.current_tenant_id')::UUID
    AND current_setting('app.current_role') IN ('coach', 'tenant_admin', 'tenant_owner')
    AND deleted_at IS NULL
    AND EXISTS (
      SELECT 1 FROM user_consents uc
      WHERE uc.user_id = wearable_readings.user_id
        AND 'wearable' = ANY(uc.categories_consented)
        AND uc.revoked_at IS NULL
    )
  );

-- Tenant admins: no raw reading access; use §14.4 aggregate view only
-- (No SELECT policy for tenant_admin on raw table — aggregate view uses form_readonly role)

-- form_admin: BYPASSRLS for nightly hard-delete erasure job and DSAR export
-- (No policy needed — BYPASSRLS is set on the role)
```

**Tenant admins cannot query raw `wearable_readings` rows.** Attempting a SELECT as `tenant_admin` returns zero rows (no matching policy). All tenant-level wearable visibility is through the privacy-preserving aggregate view in §14.4.

---

### 14.4 Tenant-Visible Aggregate View

Tenant admins and HR dashboards may see aggregated, bucketed wearable indicators — not raw readings.

```sql
-- Classification: Internal (aggregated, bucketed; no raw Art. 9 values)
-- Visible to: tenant_admin, tenant_owner via form_readonly role
-- k-anonymity: groups with < 5 members are suppressed (NULL returned)
CREATE VIEW tenant_wearable_summary AS
WITH weekly_hrv AS (
  SELECT
    tenant_id,
    user_id,
    DATE_TRUNC('week', recorded_at) AS week_start,
    AVG(value_numeric) AS avg_hrv_rmssd,
    LAG(AVG(value_numeric)) OVER (
      PARTITION BY tenant_id, user_id ORDER BY DATE_TRUNC('week', recorded_at)
    ) AS prev_week_avg_hrv
  FROM wearable_readings
  WHERE reading_type = 'hrv_rmssd'
    AND deleted_at IS NULL
    AND recorded_at >= NOW() - INTERVAL '4 weeks'
  GROUP BY tenant_id, user_id, DATE_TRUNC('week', recorded_at)
),
weekly_sleep AS (
  SELECT
    tenant_id,
    user_id,
    DATE_TRUNC('week', recorded_at) AS week_start,
    AVG(value_numeric) AS avg_sleep_min
  FROM wearable_readings
  WHERE reading_type = 'sleep_duration_min'
    AND deleted_at IS NULL
    AND recorded_at >= NOW() - INTERVAL '4 weeks'
  GROUP BY tenant_id, user_id, DATE_TRUNC('week', recorded_at)
)
SELECT
  h.tenant_id,
  h.week_start,
  COUNT(DISTINCT h.user_id) AS user_count,

  -- HRV trend direction only — no raw RMSSD values
  SUM(CASE WHEN h.avg_hrv_rmssd > h.prev_week_avg_hrv * 1.05 THEN 1 ELSE 0 END) AS hrv_improving_count,
  SUM(CASE WHEN h.avg_hrv_rmssd < h.prev_week_avg_hrv * 0.95 THEN 1 ELSE 0 END) AS hrv_declining_count,
  SUM(CASE WHEN h.avg_hrv_rmssd BETWEEN h.prev_week_avg_hrv * 0.95
                                     AND h.prev_week_avg_hrv * 1.05 THEN 1 ELSE 0 END) AS hrv_stable_count,

  -- Sleep duration buckets only — no exact minutes
  SUM(CASE WHEN s.avg_sleep_min < 360 THEN 1 ELSE 0 END) AS sleep_under_6h_count,
  SUM(CASE WHEN s.avg_sleep_min BETWEEN 360 AND 419 THEN 1 ELSE 0 END) AS sleep_6_7h_count,
  SUM(CASE WHEN s.avg_sleep_min BETWEEN 420 AND 479 THEN 1 ELSE 0 END) AS sleep_7_8h_count,
  SUM(CASE WHEN s.avg_sleep_min >= 480 THEN 1 ELSE 0 END) AS sleep_over_8h_count

FROM weekly_hrv h
LEFT JOIN weekly_sleep s
  ON h.tenant_id = s.tenant_id
  AND h.user_id = s.user_id
  AND h.week_start = s.week_start
WHERE h.prev_week_avg_hrv IS NOT NULL  -- exclude first week (no baseline)
GROUP BY h.tenant_id, h.week_start
HAVING COUNT(DISTINCT h.user_id) >= 5;  -- k-anonymity floor: suppress groups with < 5 users
```

**The view exposes only trend direction (improving/declining/stable) and sleep duration buckets.** No individual user identifiers, no raw RMSSD values, no recovery scores. A tenant admin cannot infer any individual employee's health status from this view.

**k-anonymity floor:** weeks with fewer than 5 participating users return no rows — the `HAVING COUNT(...) >= 5` clause implements this. Consistent with the analytics floor in §13.6 and SOC 2 §35 privacy floor (§6).

---

### 14.5 Cross-Source HRV Normalization

| Source | Metric stored | Unit | Comparability | Victor coaching context |
|---|---|---|---|---|
| HealthKit | `hrv_rmssd` | ms | ✅ Comparable to Oura for same-source trends | "HRV (HealthKit) trending down for 3 days" |
| Health Connect | `hrv_rmssd` | ms | ✅ Comparable to HealthKit | "HRV (Health Connect) stable this week" |
| Oura Ring | `hrv_rmssd` | ms | ✅ Comparable to HealthKit for trend analysis; absolute value may differ by ±10–20ms | "HRV (Oura) below your 30-day baseline" |
| Whoop | `hrv_rmssd` | ms | 🟡 Same metric name; Whoop uses overnight 5-min windows, algorithm differs; trend direction reliable, absolute values not cross-comparable | "HRV (Whoop) trending down — 5-day decline" |
| Garmin | `hrv_weekly_status` | ordinal | 🔴 Weekly aggregate — not daily RMSSD; cannot be compared to other sources | "HRV status (Garmin): low this week" |

**Implementation rule:** the coaching context builder tags each HRV reading with its source. Victor's prompt template uses `{{hrv_source}}` + `{{hrv_trend_label}}` — it never outputs raw RMSSD numbers to the user and never aggregates HRV across sources.

---

### 14.6 Coaching Context Integration

The coaching context builder queries `wearable_readings` for the authenticated user over the prior 14 days. It applies `stripPersonalProperties()` (documented in §35 PRV-14) before assembling the LLM context object.

**Forbidden in LLM context (never passed to Anthropic API):**
- Raw RMSSD values (e.g. "47ms")
- Exact resting HR values
- Exact sleep duration in minutes
- Temperature deviation values
- VO2max estimates
- Body battery or strain raw scores

**Permitted in LLM context (bucketed or directional only):**

| Wearable signal | Permitted representation |
|---|---|
| HRV trend | "improving" / "declining" / "stable" (7-day vs. prior 7-day baseline) |
| Resting HR trend | "elevated vs. baseline" / "normal" (directional only) |
| Sleep duration | "< 6h" / "6–7h" / "7–8h" / "> 8h" bucket (3-night average) |
| Recovery / Readiness | "low" (0–33) / "medium" (34–66) / "high" (67–100) bucket |
| Strain | "low" / "moderate" / "high" (Whoop only, bucketed) |
| Body battery | "depleted" (0–25) / "low" (26–50) / "charged" (51–75) / "full" (76–100) |

**Victor never tells the user their specific HRV number.** If a user asks "what's my HRV?", Victor directs them to their wearable app for raw data and explains that FORM uses trend direction, not the raw value, for coaching context.

---

### 14.7 GDPR and Privacy Constraints

| Constraint | Detail | Regulatory basis |
|---|---|---|
| **Data classification** | Sensitive — GDPR Art. 9 special category | §5 classification table |
| **Legal basis** | Explicit consent — `consent.categories_consented` must include `'wearable'` | GDPR Art. 9(2)(a) |
| **Consent gate** | Wearable sync is blocked at the API layer if `'wearable'` not in user's `categories_consented`; attempting sync without consent logs `wearable.sync_blocked_no_consent` DEC-030 event | §35 PRV-10 |
| **Retention** | 2 years from `recorded_at` (proposed in §35.6.3; Supabase row-level TTL migration required before launch) | GDPR Art. 5(1)(e) |
| **Art. 17 erasure** | `deleted_at` set within 30 days of erasure request (soft delete); nightly `form_admin` job hard-DELETEs rows where `deleted_at < NOW() - INTERVAL '30 days'`; includes OAuth tokens in `wearable_oauth_tokens` | GDPR Art. 17 |
| **Analytics prohibition** | Never in PostHog events, ClickHouse tables, dbt models, or Metabase dashboards (§14.10) | Art. 9; §5 Sensitive classification |
| **LLM prohibition** | Raw values never in Anthropic API requests; bucketed/directional representations only (§14.6) | Art. 9; §35 PRV-14 |
| **Sub-processor disclosure** | Whoop, Oura, Garmin listed in sub-processor register as data processors; DPAs required (P-GAP-002) | GDPR Art. 28 |

**DEC-030 audit events for wearable data:**

| Event | Trigger | Logged fields |
|---|---|---|
| `wearable.source_connected` | User connects a wearable source | `user_id`, `tenant_id`, `source`, `scopes[]` |
| `wearable.source_disconnected` | User disconnects a source | `user_id`, `tenant_id`, `source`, `revoked_by` |
| `wearable.sync_completed` | Background sync job completes | `user_id`, `tenant_id`, `source`, `reading_count` (no values) |
| `wearable.sync_blocked_no_consent` | Sync attempted without `'wearable'` consent | `user_id`, `tenant_id`, `source` |
| `wearable.data_accessed_by_coach` | Coach role queries readings (§14.3 policy) | `coach_user_id`, `athlete_user_id`, `tenant_id`, `reading_types[]` |
| `wearable.erasure_soft_deleted` | Art. 17 request sets `deleted_at` | `user_id`, `tenant_id`, `reading_count` |
| `wearable.erasure_hard_deleted` | Nightly job permanently deletes soft-deleted rows | `user_id`, `tenant_id`, `reading_count` |

---

### 14.8 OAuth and Token Storage

```sql
-- Classification: RESTRICTED (OAuth tokens = access credentials)
-- Tokens encrypted at rest via Supabase Vault (AES-256-GCM, per-tenant key)
CREATE TABLE wearable_oauth_tokens (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id         UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  tenant_id       UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
  source          TEXT NOT NULL
                    CHECK (source IN ('whoop', 'oura', 'garmin')),
                    -- HealthKit and Health Connect use platform grants, not OAuth tokens
  access_token    TEXT NOT NULL,    -- encrypted via Supabase Vault; plaintext never logged
  refresh_token   TEXT,             -- encrypted via Supabase Vault
  expires_at      TIMESTAMPTZ NOT NULL,
  scopes          TEXT[] NOT NULL,  -- granted OAuth scopes
  revoked_at      TIMESTAMPTZ,      -- set on disconnect; token purged from Vault

  created_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at      TIMESTAMPTZ NOT NULL DEFAULT now(),

  UNIQUE (user_id, source)          -- one active token set per user per source
);

ALTER TABLE wearable_oauth_tokens ENABLE ROW LEVEL SECURITY;
ALTER TABLE wearable_oauth_tokens FORCE ROW LEVEL SECURITY;

-- Users: read own tokens (to check connection status); cannot read token values
CREATE POLICY oauth_tokens_user_select ON wearable_oauth_tokens
  FOR SELECT TO form_api
  USING (
    user_id = current_setting('app.current_user_id')::UUID
    AND revoked_at IS NULL
  );

-- form_admin only for actual token reads (used by Cloudflare Worker sync job via service role)
-- (Service role for sync workers uses form_admin — BYPASSRLS)
```

**Token refresh architecture:**
- A Cloudflare Worker cron job runs every 30 minutes and refreshes tokens expiring within 60 minutes.
- Token refresh is not in the request path — the mobile app never handles OAuth token exchange after initial connect.
- On revocation (user disconnects): `revoked_at` set, Supabase Vault key material for that token rotated, `wearable.source_disconnected` DEC-030 event emitted.

---

### 14.9 Sync Architecture

```
iOS — HealthKit
  ├── HealthKit Background Delivery registration (per data type)
  ├── HKObserverQuery fires on new data → FORM app calls backend
  └── Backend: upsert into wearable_readings via (user_id, source, reading_type, recorded_at) UNIQUE

Android — Health Connect
  ├── WorkManager periodic task (15-min minimum; actual interval: 30 min to preserve battery)
  ├── Health Connect Read request for delta since last sync (stored as SharedPreferences token)
  └── Backend: batch upsert

Whoop / Oura / Garmin
  ├── Cloudflare Worker Cron Trigger: every 2 hours (*/120 * * * *)
  ├── For each source:
  │     1. Load access token from Supabase (via service role)
  │     2. If token expired → refresh and update wearable_oauth_tokens
  │     3. Fetch readings since last_synced_at (stored per user per source in wearable_oauth_tokens.updated_at)
  │     4. Validate reading schema (Zod) — reject unknown fields
  │     5. Upsert into wearable_readings
  │     6. Emit wearable.sync_completed DEC-030 event (reading_count only, no values)
  └── Rate limits: Whoop 100 req/min; Oura 5,000 req/day; Garmin 1,500 req/day — Worker enforces per-source limits

Idempotency:
  All paths use INSERT ... ON CONFLICT (user_id, source, reading_type, recorded_at) DO NOTHING
  Duplicate deliveries from HealthKit Background Delivery (common after app restart) are silently ignored
```

---

### 14.10 Analytics Restrictions

| Layer | `wearable_readings` rows allowed? | `tenant_wearable_summary` view allowed? | Reason |
|---|---|---|---|
| PostHog events | 🔴 No | 🔴 No | Art. 9 Sensitive — §13.6 prohibition |
| ClickHouse `fact_events` | 🔴 No | 🔴 No | Art. 9 Sensitive |
| ClickHouse `fact_session_metrics` | 🔴 No | 🔴 No | Art. 9 Sensitive |
| Metabase dashboards (raw) | 🔴 No | 🟡 Yes — aggregate view only, tenant_admin role | k-anonymity ≥ 5; trend/bucket only |
| LLM coaching context (Victor) | 🔴 Raw values No | 🔴 No | Bucketed/directional only — §14.6 |
| DEC-030 audit log | 🟡 Sync counts only | N/A | No individual values ever logged |
| DSAR export (user requests own data) | ✅ Yes — full export | N/A | GDPR Art. 15 right of access |

The `stripForbiddenProperties()` function (§35 PRV-14) is extended to include all `wearable_readings` numeric field names in its blocklist. A CI lint rule enforces this — any analytics event fixture containing a wearable field name fails the build.

---

### 14.11 Implementation Checklist

| # | Task | Owner | Priority | Milestone | Notes |
|---|---|---|---|---|---|
| 1 | Create `wearable_readings` table with UNIQUE constraint and RLS policies per §14.3 | `platform-engineer` | **P0** | M3 | Migration must pass RLS lint check (§3.5); include `deleted_at` column and both composite indexes |
| 2 | Create `wearable_oauth_tokens` table with Supabase Vault encryption for `access_token` and `refresh_token` | `platform-engineer` | **P0** | M3 | Never log token values in any DEC-030 event or debug log |
| 3 | Implement HealthKit integration: Background Delivery registration + `HKObserverQuery` handler + backend upsert endpoint | `platform-engineer` | **P0** | M3 | iOS only; test with all supported data types (HRV, HR, sleep, energy, steps) |
| 4 | Implement Health Connect integration: WorkManager periodic task + delta sync + batch upsert | `platform-engineer` | **P0** | M3 | Android only; test Health Connect permission flow on Android 13+ and 14+ |
| 5 | Add `'wearable'` consent gate enforcement in sync endpoint: check `user_consents.categories_consented` before accepting any wearable data | `platform-engineer` | **P0** | M3 | Log `wearable.sync_blocked_no_consent` DEC-030 event on rejection |
| 6 | Wire all seven DEC-030 `wearable.*` audit events to HMAC chain (§14.7) | `platform-engineer` | **P0** | M3 | Audit events for: source_connected, source_disconnected, sync_completed, sync_blocked_no_consent, data_accessed_by_coach, erasure_soft_deleted, erasure_hard_deleted |
| 7 | Implement Whoop API client: OAuth PKCE flow + token refresh + data fetch + rate-limit enforcement | `platform-engineer` | **P0** | M4 | Cloudflare Worker cron every 2h; never exceed 100 req/min rate limit |
| 8 | Implement Oura API client (v2): OAuth PKCE flow + token refresh + daily aggregate fetch | `platform-engineer` | **P0** | M4 | EU data residency endpoint; 5,000 req/day limit |
| 9 | Implement Garmin API client: OAuth 1.0a + Health API data fetch + HRV Status (weekly) parsing | `platform-engineer` | **P1** | M4 | Requires Garmin Health API partner agreement; lower priority than Whoop/Oura |
| 10 | Build coaching context builder: 14-day wearable query + `stripPersonalProperties()` + bucket mapping (§14.6) | `platform-engineer` | **P0** | M3 | Unit tests: verify no raw RMSSD/HR values pass through to LLM context; test all source × reading_type combinations |
| 11 | Create `tenant_wearable_summary` view (§14.4): k-anonymity floor, trend buckets, sleep buckets | `platform-engineer` | **P1** | M4 | Confirm view is accessible by `form_readonly` role for Metabase; not accessible by `form_api` |
| 12 | Implement Art. 17 hard-delete nightly job: hard-DELETE `wearable_readings` where `deleted_at < NOW() - INTERVAL '30 days'`; include `wearable_oauth_tokens` on revocation | `platform-engineer` | **P0** | M3 | Runs under `form_admin` BYPASSRLS; emit `wearable.erasure_hard_deleted` DEC-030 event with reading_count |
| 13 | Add wearable field blocklist to `stripForbiddenProperties()` and PostHog CI lint rule | `data-engineer` | **P0** | M3 | CI build must fail if any analytics event fixture contains a wearable reading field name |
| 14 | Implement Supabase row-level TTL migration: delete `wearable_readings` where `recorded_at < NOW() - INTERVAL '2 years'` (nightly cron) | `platform-engineer` | **P1** | M4 | Pending formal retention decision per P-GAP-004; use proposed 2-year period from §35.6.3 as placeholder |
| 15 | DSAR export: include `wearable_readings` full export (all non-deleted rows) in Art. 15 data package | `platform-engineer` | **P0** | M3 | Per §14.10 — users are entitled to their own raw readings; export as JSON per reading_type per source |
| 16 | Integration test: wearable consent revocation → sync blocked → existing readings soft-deleted → DEC-030 chain | `qa-lead` | **P1** | M4 | Full state machine test across consent → sync → revocation → erasure |
| 17 | Load test: Cloudflare Worker cron at 10,000 connected users (Whoop + Oura simultaneously) | `devops-lead` | **P2** | M5 | Verify rate limits honored; verify no cross-user token leak |
| 18 | Sub-processor DPA collection: obtain DPAs from Whoop (US), Oura (EU), Garmin (US); execute SCC Module 2 for US processors | `compliance-officer` | **P1** | Pre-launch | Required before any enterprise pilot customer whose employees connect wearables; closes P-GAP-002 partial scope |

---

*v0.5 additions: §14 Wearable Data Integration Schema — five source integrations (HealthKit, Health Connect, Whoop, Oura Ring, Garmin) with platform-specific auth mechanisms, data types, and HRV metric comparability notes; `wearable_readings` table DDL with 14 reading_type values, `(user_id, source, reading_type, recorded_at)` UNIQUE idempotency constraint, two composite indexes, and `deleted_at` soft-delete; RLS policies for user self-access, coach access with consent gate, and form_admin erasure path; `tenant_wearable_summary` aggregate view with k-anonymity floor (N ≥ 5), HRV trend direction only (no raw RMSSD), and sleep duration buckets (no exact minutes); cross-source HRV normalization table (Garmin weekly HRV Status not comparable to daily RMSSD; Whoop algorithm differs from HealthKit); coaching context builder with `stripPersonalProperties()` extension and bucket mapping for all wearable signals (bucketed/directional only — raw values never in Anthropic API requests); GDPR Art. 9 constraint table (explicit consent gate, 2-year retention, 30-day Art. 17 erasure window, analytics prohibition); seven DEC-030 HMAC-chained audit events (`wearable.source_connected`, `source_disconnected`, `sync_completed`, `sync_blocked_no_consent`, `data_accessed_by_coach`, `erasure_soft_deleted`, `erasure_hard_deleted`); `wearable_oauth_tokens` table with Supabase Vault encryption for tokens (RESTRICTED classification); sync architecture: HealthKit Background Delivery (iOS), Health Connect WorkManager (Android, 30-min interval), Cloudflare Worker cron every 2h (Whoop/Oura/Garmin) with per-source rate limits; analytics restrictions table confirming wearable data excluded from all PostHog/ClickHouse/Metabase layers; 18-item implementation checklist (11× P0, 5× P1, 2× P2) across M3/M4/M5.*
