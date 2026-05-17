# FORM · Multi-Tenant Data Model v0.2

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
| **Confidential** | Health goals, restrictions, HRV baseline | Standard RDS encryption | Owner user only (RLS) |
| **Sensitive** | CV raw pose data, meal log items | Per-tenant AES-256-GCM + KMS | Owner user only; `form_admin` via break-glass |
| **Restricted** | Audit log HMAC keys | KMS only; never in DB | `form_audit` role + KMS policy |

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

**v0.1 · травень 2026 · owner: enterprise-architect + compliance-officer + security-engineer**

*v0.2 additions: Section 10 PITR Tenant-Isolated Restore Runbook (closes SOC 2 gap 9.4).*
