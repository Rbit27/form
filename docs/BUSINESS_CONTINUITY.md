# FORM · Business Continuity & Disaster Recovery Plan v1.0

> **Classification:** Internal — Enterprise Confidential. Shareable under NDA with enterprise customers.
> **Owner:** `devops-lead` + `compliance-officer`. Review: after every DR drill, after any infrastructure change affecting RTO/RPO, and annually (see §15.1 of `docs/SOC2_READINESS.md`).
> **SOC 2 controls satisfied:** CC7.4, CC7.5, A1.1, A1.2, A1.3, CC9.2.
> **Related documents:** `docs/INCIDENT_RESPONSE.md`, `docs/ENTERPRISE_SLA.md`, `docs/DATA_MODEL.md §8–10`, `docs/OBSERVABILITY.md`, `docs/SOC2_READINESS.md §18`.
> **Decision log:** DEC-030 (HMAC-chained audit log integrity during recovery).

---

## Table of Contents

1. [Purpose and Scope](#1-purpose-and-scope)
2. [Business Impact Analysis](#2-business-impact-analysis)
3. [Recovery Objectives by Tier](#3-recovery-objectives-by-tier)
4. [Service Dependency Map](#4-service-dependency-map)
5. [Backup Architecture](#5-backup-architecture)
6. [Recovery Scenarios and Runbooks](#6-recovery-scenarios-and-runbooks)
7. [DR Drill Procedure and Evidence](#7-dr-drill-procedure-and-evidence)
8. [Incident Communication Plan](#8-incident-communication-plan)
9. [Roles and Responsibilities](#9-roles-and-responsibilities)
10. [Continuity for Non-Infrastructure Failures](#10-continuity-for-non-infrastructure-failures)
11. [Known Gaps and Roadmap](#11-known-gaps-and-roadmap)
12. [Document Control](#12-document-control)

---

## 1. Purpose and Scope

### 1.1 Purpose

This Business Continuity Plan (BCP) and Disaster Recovery Plan (DRP) is the authoritative document defining how FORM restores operations following an unplanned disruption. It serves three audiences:

1. **Engineering team** — executable runbooks for on-call engineers without reliance on tribal knowledge.
2. **Enterprise customers** — evidence that contractual SLA commitments (docs/ENTERPRISE_SLA.md) are backed by tested procedures.
3. **SOC 2 auditors** — documentary evidence for criteria A1.1 (availability commitments and system components), A1.2 (capacity and environmental threats), A1.3 (recovery plan and procedures), CC7.5 (recovery from incidents), and CC9.2 (business continuity and vendor management).

This document is part of FORM's formal Policy Pack alongside the Acceptable Use Policy (`docs/ACCEPTABLE_USE_POLICY.md`) and Incident Response Runbook (`docs/INCIDENT_RESPONSE.md`).

### 1.2 Scope

**In scope:**

| Component | Service | Criticality |
|---|---|---|
| API runtime | Cloudflare Workers (global edge) | Critical |
| Primary database | Supabase Postgres (Multi-AZ, us-east-1 or eu-central-1) | Critical |
| Object storage | Supabase Storage (video reps, profile images) | High |
| Authentication | Supabase Auth + custom SAML/OIDC bridge | Critical |
| AI coaching | Anthropic Claude API | High |
| AI voice | ElevenLabs API | Medium |
| Audit log | Supabase Postgres — `audit_events` table (HMAC-chained, DEC-030) | Critical |
| Analytics | PostHog Cloud | Low |
| Error tracking | Sentry | Medium |
| Uptime monitoring | Better Uptime | Medium |

**Out of scope:**

- iOS/Android app distribution (Apple App Store / Google Play — operated by Apple/Google).
- Third-party provider internal infrastructure (Supabase, Cloudflare, Anthropic, ElevenLabs handle their own BCP).
- Developer laptops and local environments.

### 1.3 Activation Criteria

This BCP is activated whenever an incident is declared at **P0 or P1** severity under `docs/INCIDENT_RESPONSE.md §1.1`, OR when any of the following conditions are met independently of a declared incident:

- Production database unreachable for > 5 consecutive minutes.
- API error rate > 25% sustained for > 10 minutes.
- Authentication service failure affecting any enterprise tenant login.
- Confirmed or suspected data corruption in any tenant partition.
- Complete loss of access to production environment credentials.

---

## 2. Business Impact Analysis

### 2.1 Service Criticality and Maximum Tolerable Downtime

The Maximum Tolerable Downtime (MTD) is the outer bound — beyond this, the business impact (customer SLA breach, regulatory obligation, revenue loss) becomes severe enough to trigger executive escalation.

| Service | MTD (Enterprise) | MTD (Consumer Pro) | Business Impact if Exceeded |
|---|---|---|---|
| **API + Authentication** | 4 hours | 8 hours | SLA credit obligation; GDPR Art. 33 breach risk if data involved; customer churn risk |
| **AI Coaching** | 4 hours | 8 hours | Core product unavailable; user-facing degradation; NPS impact |
| **Database (read)** | 1 hour | 4 hours | All authenticated operations fail; no coaching sessions possible |
| **Database (write)** | 30 minutes | 2 hours | Progress data loss; workout history gaps; user trust damage |
| **SSO / SAML** | 1 hour | N/A | Enterprise users locked out; admin dashboard inaccessible; P1 SLA breach |
| **SCIM Provisioning** | 24 hours | N/A | New-hire onboarding delayed; de-provisioning lag (security risk) |
| **Audit Log (write)** | 15 minutes | N/A | SOC 2 evidence gap; HMAC chain integrity break (DEC-030 trigger) |
| **Admin Dashboard** | 4 hours | N/A | HR/People leads lose visibility; no management actions possible |
| **AI Voice** | 8 hours | 24 hours | Coaching quality degraded; fallback to text-only mode |
| **Analytics (PostHog)** | 48 hours | 48 hours | Operational dashboards blind; no product impact |
| **Error Tracking (Sentry)** | 24 hours | 24 hours | Reduced incident detection fidelity; compensate with uptime checks |

### 2.2 Revenue Impact Estimate

At target ARR of $500k (consumer PMF gate) and initial enterprise ARR, a 4-hour P0 outage carries the following estimated impact:

| Impact Category | Estimate |
|---|---|
| Enterprise SLA credit (99.9% tier, 1 P0) | ~$2,000–$8,000 per affected tenant/month |
| Consumer subscription churn risk (4h outage) | < 0.5% monthly churn uplift (industry benchmark) |
| GDPR supervisory authority fine (if breach involved) | Up to €10M / 2% global turnover |
| Reputational cost in enterprise sales | High — reference accounts require zero unmitigated P0s |

### 2.3 Privacy Floor During Recovery

The Privacy Floor is non-negotiable and persists through all recovery states:

> **HR never sees individual user data** — not during normal operations, not during disaster recovery, not in any restored backup state. Aggregate-only admin access is enforced at the Row-Level Security (RLS) layer and in the admin API, not in application code (docs/DATA_MODEL.md §6, docs/ENTERPRISE.md §Privacy Floor).

Any PITR restore must be verified against this invariant before lifting maintenance mode. The verification check is Step 7 of the Scenario D runbook (§6.4).

---

## 3. Recovery Objectives by Tier

These targets govern contractual commitments to enterprise customers (`docs/ENTERPRISE_SLA.md §3`) and the engineering team's design targets.

### 3.1 RTO and RPO Targets

| Tier / Component | Recovery Time Objective (RTO) | Recovery Point Objective (RPO) | Notes |
|---|---|---|---|
| **Enterprise tier (1,000+ seats)** | **4 hours** | **1 hour** | Contractual. Breach triggers SLA credit (ENTERPRISE_SLA.md §4). |
| **Growth tier (200–1,000 seats)** | **4 hours** | **1 hour** | Contractual. Same Standard SLA as Enterprise. |
| **Starter tier (50–200 seats)** | **4 hours** | **1 hour** | Contractual. Standard SLA. |
| **Consumer Pro** | 8 hours | 4 hours | Best-effort. Disclosed in ToS. No credit obligation. |
| **Consumer Free** | 24 hours | 24 hours | No commitment. |
| **Auth / SSO** | **1 hour** | 15 minutes | SSO outage blocks enterprise users. Priority restore above general API. |
| **Audit Log (write path)** | 15 minutes | 0 minutes | HMAC chain must be intact. Read access during recovery is acceptable; write must resume before chain gap occurs. |
| **Cold start (Scenario D — nuke)** | 8 hours | Depends on last cold backup | Worst-case, non-contractual. See §6.4. |

### 3.2 RPO Implementation Basis

The 1-hour RPO for enterprise and 4-hour RPO for Pro are **maximum tolerable data loss windows**, not expected loss windows:

- **Supabase Postgres** uses Point-in-Time Recovery (PITR) via WAL streaming. WAL lag to the warm standby is < 5 minutes in Multi-AZ configuration (Supabase contractual guarantee on paid plans). PITR retention window: 7 days.
- **Supabase Storage** replicates across Availability Zones within the selected region (us-east-1 or eu-central-1). Effective RPO for media: 0 minutes under normal AZ conditions.
- **Expected data loss** in a typical Supabase platform incident: < 5 minutes. The 1-hour RPO is the engineering commitment buffer, accounting for detection delay, runbook execution time, and PITR restore latency.

### 3.3 Data Region Commitments

Enterprise customers may elect EU data residency at contract execution (eu-central-1 Frankfurt or eu-west-1 Ireland). Region selection is contractually bound; post-deployment migration requires a formal migration engagement. All RTO/RPO targets apply equally across regions (Supabase Multi-AZ exists in both regions).

---

## 4. Service Dependency Map

Understanding dependencies prevents circular confusion during incidents. This map shows which service failures cascade.

```
User (mobile app / admin dashboard)
    │
    ▼
Cloudflare Workers (API edge)  ◄─── Cloudflare DNS (api.form.coach)
    │
    ├──► Supabase Auth  ◄─── SAML/OIDC bridge (enterprise SSO)
    │        │
    │        └──► SCIM Provisioning (Okta / Azure AD)
    │
    ├──► Supabase Postgres (primary database)
    │        │
    │        ├──► Row-Level Security (tenant isolation)
    │        ├──► audit_events (HMAC-chained, DEC-030)
    │        └──► PITR / WAL replica
    │
    ├──► Supabase Storage (video, images)
    │
    ├──► Anthropic Claude API (AI coaching engine)
    │
    ├──► ElevenLabs API (AI voice coaching)
    │
    ├──► PostHog (analytics, non-critical path)
    └──► Sentry (error tracking, non-critical path)

Monitoring:
    Better Uptime ──► PagerDuty ──► On-call engineer
    Cloudflare Analytics ──► Sentry ──► On-call engineer
```

**Critical-path dependencies (failure = full P0):**
1. Cloudflare Workers — all API traffic runs through it; no secondary edge.
2. Supabase Postgres — all user data, sessions, and tenant configuration live here.
3. Supabase Auth — all authentication (consumer + enterprise) depends on it.

**Single-provider risk acknowledgment:**
FORM currently has no secondary edge provider for Cloudflare Workers and no secondary database provider for Supabase. These are documented risks (R-003, R-004 in `docs/SOC2_READINESS.md §14`). Mitigation: both providers carry SLAs > 99.9% and global redundancy in their own infrastructure. Formal secondary-provider DR is Post-Series-A scope.

---

## 5. Backup Architecture

### 5.1 Primary Backup Mechanism — Supabase PITR

| Parameter | Configuration |
|---|---|
| **Backup type** | Continuous WAL streaming (Point-in-Time Recovery) |
| **RPO** | < 5 minutes (WAL lag) |
| **Retention window** | 7 days |
| **Restore granularity** | 1-minute intervals within the retention window |
| **Geographic scope** | Primary region + warm standby in same Supabase AZ group |
| **Plan requirement** | Supabase Pro or Enterprise (PITR not available on Free plan) |
| **Verification** | Quarterly PITR restore test in staging; annual DR drill in staging (§7) |

PITR restore procedure: `docs/DATA_MODEL.md §10 — PITR Tenant-Isolated Restore Runbook`.

### 5.2 Cold Storage Backup — Backblaze B2

| Parameter | Configuration |
|---|---|
| **Backup type** | Nightly export via Supabase CLI (`supabase db dump`), encrypted, uploaded to Backblaze B2 |
| **Bucket name** | `form-dr-backups` (B2, EU region for EU-residency customers) |
| **Encryption** | AES-256 at rest; encrypted before upload using GPG key (key stored in 1Password "DR / Cold Storage" vault) |
| **Retention** | 90 days rolling |
| **Access** | 1Password team vault → "DR / Cold Storage" → B2 credentials |
| **Status** | **Partial gap** — automated nightly export not yet implemented. Manual export procedure exists (see §6.4, Scenario D). Automated pipeline scheduled Q3. |

### 5.3 Audit Log Backup

The `audit_events` table is subject to special backup handling due to DEC-030:

- Audit logs are **append-only** and **HMAC-chained**. Any restore that alters historical audit records breaks the chain and must be treated as a potential integrity breach.
- During a PITR restore, audit logs from after the restore point are **not recoverable**. The gap is documented in the post-incident report and disclosed to affected enterprise tenants under the GDPR Art. 33 assessment (if the gap is material).
- SOC 2 audit log retention: 7 years. Even after user deletion, audit records are anonymised (user_id replaced; action and timestamp retained per DEC-030 / `docs/AUDIT_LOG_SCHEMA.md §3`).

### 5.4 Cloudflare Workers Configuration Backup

Cloudflare Workers configuration is stored as Infrastructure-as-Code in the Git repository (`wrangler.toml`, `src/`). The Git repository is the backup. In a complete environment loss, Workers are redeployable via `wrangler deploy` from the latest committed state in `main`.

- **Secrets** (API keys, Supabase service role key) are NOT stored in Git. They are stored in 1Password and re-entered manually during a cold start.
- **DNS configuration** is documented in `docs/DEPLOYMENT.md` and recoverable from Cloudflare dashboard or DNS export.

---

## 6. Recovery Scenarios and Runbooks

Each scenario maps to a failure mode from the Business Impact Analysis (§2). Runbooks are executable step-by-step procedures, not narrative descriptions.

### 6.1 Scenario A — Supabase Database Unavailability

**Definition:** Supabase Postgres primary is unreachable or returning errors on all queries for > 5 minutes (not operator-caused).

**Detection sources:**
- Better Uptime health check on `GET /api/health` returns non-2xx for 3 consecutive 1-minute checks → PagerDuty P1 alert.
- Sentry: database connection error spike > 50 errors/minute.
- Cloudflare Analytics: 5xx rate on database-dependent endpoints > 20%.

**Response:**

| Step | Action | Owner | Time Budget |
|---|---|---|---|
| 1 | Acknowledge PagerDuty alert. Check Supabase Status Page (`status.supabase.com`) and Supabase dashboard for active incidents. | On-call engineer | < 5 min |
| 2 | If **Supabase platform incident**: update `form.coach/status` page. Switch API to maintenance mode (return HTTP 503 with `Retry-After: 1800` header). Post in enterprise customer Slack channels (template §8.2). | On-call engineer | < 10 min |
| 3 | If **FORM-side issue** (migration failure, misconfiguration): invoke `docs/INCIDENT_RESPONSE.md §5.2` (Database Incident runbook). | On-call engineer | < 15 min |
| 4 | If Supabase platform incident persists > 2 hours: contact Supabase Enterprise Support. Escalate to founder. | On-call engineer | 2 hours |
| 5 | If total outage > 4 hours and RPO breach is imminent: initiate PITR restore to the latest clean checkpoint. Follow `docs/DATA_MODEL.md §10` exactly. | Founder + on-call | By 4-hour mark |
| 6 | Notify enterprise tenants via DPA-specified contact channel within 1 hour of confirmed P1. Use template §8.3. | On-call engineer | < 1 hour of P1 declaration |
| 7 | Write post-incident report using `docs/INCIDENT_RESPONSE.md §8` template. File within 5 business days. | On-call engineer | D+5 |

**RTO target: 4 hours** (from first alert to service restored). If Supabase recovers within 4 hours internally, no PITR restore is required. If outage exceeds 4 hours, PITR is initiated regardless of Supabase's own timeline.

**Known risk:** FORM is on Supabase's shared infrastructure. Supabase does not provide a formal SLA on PITR restore time. Compensating control: annual PITR drill validates the engineering team's restore capability independently of Supabase's timeline.

---

### 6.2 Scenario B — Cloudflare Workers API Unavailability

**Definition:** All API endpoints globally unreachable for > 5 minutes, confirmed as Cloudflare-side (not Supabase or application error).

**Detection sources:**
- Better Uptime health checks fail from multiple global probe locations simultaneously.
- Cloudflare Status Page (`cloudflarestatus.com`) shows active incident.

**Response:**

| Step | Action | Owner | Time Budget |
|---|---|---|---|
| 1 | Confirm it is a Cloudflare platform incident (not a FORM Worker bug). Check Cloudflare Status Page + `wrangler tail` for errors. | On-call engineer | < 5 min |
| 2 | Update `form.coach/status`. Switch to maintenance messaging (HTTP 503). Post enterprise Slack notification (template §8.2). | On-call engineer | < 10 min |
| 3 | If it is a FORM Worker bug (not Cloudflare platform): rollback to previous Worker version via `wrangler rollback`. | On-call engineer | < 15 min |
| 4 | After Cloudflare recovery: run smoke test suite (F1–F8 flows per qa-walker protocol) before lifting maintenance mode. | On-call engineer | < 30 min post-recovery |
| 5 | Verify all Worker deployments serve the expected version (`wrangler tail`). Replay any queued background jobs if applicable. | On-call engineer | < 1 hour post-recovery |

**RTO target: Cloudflare-dependent.** FORM has no secondary edge provider. Cloudflare's historical uptime is > 99.99%; global anycast redundancy is the compensating control.

**Known gap:** No secondary CDN/edge failover exists. This is documented risk R-004 in `docs/SOC2_READINESS.md §14`. Formal secondary-provider DR is Post-Series-A scope.

---

### 6.3 Scenario C — Data Corruption (Accidental or Malicious)

**Definition:** Production data found to be corrupted, partially deleted, or tampered with. Includes: accidental mass-delete via admin API, failed migration with data loss, insider threat, or external attack.

**Detection sources:**
- `pg_stat_user_tables` row count monitoring: > 5% drop in any key table within 1 hour → PagerDuty P1.
- Audit log review triggered by any P1 incident or anomalous admin API call.
- HMAC audit log chain break detected (DEC-030 trigger → immediate P0).

**Response:**

| Step | Action | Owner | Time Budget |
|---|---|---|---|
| 1 | Immediately **freeze the affected table(s)**: revoke `INSERT`, `UPDATE`, `DELETE` from `form_api` role. Stop further corruption while preserving read access. | On-call engineer | < 5 min |
| 2 | **Snapshot current state** before any restore: `pg_dump --table=<affected_table> --format=custom > pre-restore-snapshot.dump`. | On-call engineer | < 10 min |
| 3 | **Identify corruption timestamp** from audit log (`docs/AUDIT_LOG_SCHEMA.md`) and Cloudflare Worker logs (Sentry + `wrangler tail` replay). | On-call engineer | < 20 min |
| 4 | **Invoke PITR restore** to a timestamp 5 minutes before the identified corruption event. Follow `docs/DATA_MODEL.md §10` (PITR Tenant-Isolated Restore Runbook) step-by-step. | On-call engineer | < 2 hours |
| 5 | **Validate restored data** against the pre-corruption snapshot: compare row counts, checksums on key columns. Document any unrecoverable delta. | On-call engineer | < 30 min post-restore |
| 6 | **Restore RLS policies** and verify Privacy Floor: confirm HR-visible aggregates are unaffected; confirm no individual user data is exposed in admin views. | On-call engineer | < 15 min |
| 7 | **Re-enable table permissions** and lift maintenance mode only after validation is complete. | On-call engineer | After step 6 |
| 8 | If malicious actor suspected: invoke `docs/INCIDENT_RESPONSE.md §5.5` (Security Breach). Notify `compliance-officer`. Start 72-hour GDPR breach notification clock from confirmed detection time. | compliance-officer | Immediately |
| 9 | Write post-incident report (`docs/INCIDENT_RESPONSE.md §8` template). | On-call engineer | D+5 |

**RTO target: 4 hours** (enterprise). **RPO target: 1 hour** (enterprise). With PITR at < 5-minute WAL granularity, the limiting factor is detection time, not restore granularity.

**Note on HMAC audit log integrity:** If the corruption event affected the `audit_events` table itself, a chain break must be assessed per DEC-030. A chain break is a P0 security incident. Do not restore audit records from a pre-corruption backup without reviewing the integrity implications with `compliance-officer` first — overwriting a tampered audit log with an earlier clean backup may destroy forensic evidence.

---

### 6.4 Scenario D — Complete Environment Loss ("Nuke Scenario")

**Definition:** Loss of all production Supabase project data, Cloudflare Workers configuration, and access credentials simultaneously. Plausible causes: compromised Supabase account, supplier-side catastrophic data loss, extreme operator error at provider level.

**Pre-condition for this runbook:** Supabase PITR restore is not possible (project deleted or inaccessible). Cold storage backup must be used.

**Response:**

| Step | Action | Dependency |
|---|---|---|
| 1 | Access 1Password team vault → "DR / Cold Storage" → retrieve Backblaze B2 credentials and GPG decryption key. | 1Password accessible |
| 2 | Download latest nightly backup from B2 bucket `form-dr-backups`. Verify GPG signature before decryption. | Backblaze B2 accessible |
| 3 | Create a new Supabase project in the target region (us-east-1 or eu-central-1 per affected customer contracts). | Supabase dashboard accessible |
| 4 | Restore from the decrypted dump: `supabase db restore < backup.sql` (or Supabase CLI import). | Step 3 complete |
| 5 | Re-run all migrations from `supabase/migrations/` in sequence to bring schema to HEAD. | Git repository accessible |
| 6 | Restore Cloudflare Workers: `wrangler deploy` from Git `main`. Re-enter all environment secrets from 1Password into Cloudflare Workers dashboard. | Git + 1Password accessible |
| 7 | Re-point DNS: update `api.form.coach` CNAME in Cloudflare DNS panel to the new Worker route. | Cloudflare DNS accessible |
| 8 | **Privacy Floor verification:** Run RLS test suite (`docs/DATA_MODEL.md §4 — Tenant Isolation Guarantees`). Confirm no cross-tenant data access. Confirm admin aggregates do not expose individual user data. | Steps 4–7 complete |
| 9 | Run smoke test (F1–F8 core flows). Do not lift maintenance mode until all critical flows pass. | Step 8 complete |
| 10 | Notify all enterprise tenants of restoration, data gap period (last cold backup to restoration point), and any data loss. Issue formal incident communication (§8.4). | After step 9 |
| 11 | Write post-incident report. Assess whether cold backup gap constitutes a GDPR breach (data created after last backup and before restoration is lost). | D+5 |

**RTO target: 8 hours** (worst-case cold start; estimated 4–6 hours of execution, 2-hour buffer).

**RPO target:** Determined by last cold backup timestamp. If cold backup is from T-18h, data loss = 18 hours. This exceeds the 1-hour contractual RPO — enterprise customers must be notified of the breach and SLA credit applied automatically.

**Known gap:** Automated nightly cold backup export to Backblaze B2 is not yet implemented. Current state: Supabase PITR is the sole backup mechanism, which is unavailable in the nuke scenario. Automated pipeline is on the Q3 implementation roadmap. Until then, this scenario's RTO is Supabase's own DR capability (contractually undefined on the Pro plan).

---

### 6.5 Scenario E — SSO / Authentication Service Failure

**Definition:** Enterprise tenant users cannot authenticate via SAML 2.0 or OIDC. Consumer users may be able to log in via email/password while enterprise SSO is degraded.

**Detection sources:**
- Enterprise tenant admin reports login failures.
- Supabase Auth dashboard shows SAML assertion errors.
- Better Uptime check on `/auth/saml/callback` returns errors.

**Response:**

| Step | Action | Owner | Time Budget |
|---|---|---|---|
| 1 | Determine scope: single tenant (IdP-side issue) vs all enterprise tenants (FORM-side SAML bridge issue). Check Supabase Auth logs and SAML assertion error details. | On-call engineer | < 10 min |
| 2 | If single-tenant IdP issue: notify the tenant admin; provide `docs/SSO_CLIENT_CONFIG.md` self-service guide. Most IdP misconfigurations are resolvable by the tenant admin. | On-call engineer | < 30 min |
| 3 | If FORM-side SAML bridge issue: check Cloudflare Worker SAML handler logs. Roll back if a recent deployment introduced the regression. | On-call engineer | < 20 min |
| 4 | If Supabase Auth platform issue: check `status.supabase.com`. Contact Supabase Enterprise Support. | On-call engineer | < 30 min |
| 5 | Emergency bypass (P0 only, explicit compliance-officer approval required): temporarily enable email/password login for the affected enterprise tenant while SSO is restored. Log the bypass in the audit trail. Disable bypass within 24 hours. | compliance-officer approval | Only if RTO breach imminent |
| 6 | Notify tenant admin and CS lead via DPA contact. Log in audit events: `sso_outage_start`, `sso_restored`. | On-call engineer | Within 1 hour of confirmed P1 |

**RTO target: 1 hour** (SSO outage is enterprise-blocking; priority over general API recovery).

---

## 7. DR Drill Procedure and Evidence

### 7.1 Schedule

Per `docs/SOC2_READINESS.md §15.1` (Annual Compliance Calendar):

| Drill | Frequency | Window | Scenario | Evidence Artifact |
|---|---|---|---|---|
| Primary PITR Drill | Annual (Q1 January) | Scheduled maintenance window; staging environment | Scenario C (data corruption simulation) | `evidence/dr-drills/YYYY-MM-drill-report.md` |
| Cold Start Verification | Annual (Q3, after cold backup automation is implemented) | Staging | Scenario D cold restore | `evidence/dr-drills/YYYY-MM-cold-start-report.md` |
| SSO Failover Test | Annual (Q2) | Staging IdP environment | Scenario E (SAML bypass + restore) | `evidence/dr-drills/YYYY-MM-sso-drill-report.md` |

### 7.2 Drill Execution — Primary PITR Drill

The January drill exercises **Scenario C (data corruption)** in a staging environment. Staging must be a recent schema copy of production (migrations applied to HEAD; data can be synthetic).

**Pre-drill (D-7):**

```
[ ] Notify all engineers that the drill will occur this week.
[ ] Confirm staging environment is up-to-date (run migrations to HEAD).
[ ] Confirm PITR is enabled on the staging Supabase project.
[ ] Assign DR Lead — must NOT be the author of this runbook (independence requirement).
[ ] Assign Observer — compliance-officer or devops-lead (independent sign-off required).
[ ] Create a pre-drill snapshot of staging for baseline comparison.
```

**Drill execution (2–4 hours):**

```
[ ] DR Lead simulates data corruption: delete 10% of rows from the workouts table in staging.
[ ] DR Lead follows Scenario C runbook (§6.3) step-by-step, without assistance from runbook author.
[ ] Observer timestamps each step in the drill report.
[ ] Record:
    - T0: corruption event timestamp
    - T1: detection timestamp (from monitoring alert)
    - T2: containment timestamp (table freeze)
    - T3: restore-complete timestamp
    - T4: validation-complete timestamp
[ ] Measure: time-to-detect (T1–T0), time-to-contain (T2–T0), time-to-restore (T3–T0), data loss in rows.
[ ] Compare actuals against targets in §3.1.
```

**Post-drill (D+1):**

```
[ ] DR Lead writes drill report using template §7.3.
[ ] Observer signs report (physical or electronic signature acceptable).
[ ] File report at: evidence/dr-drills/YYYY-MM-drill-report.md
[ ] Update this BCP document if runbook gaps were discovered.
[ ] Update SOC 2 evidence registry (docs/SOC2_READINESS.md §12).
[ ] If RTO or RPO targets were not met: create remediation task in Linear; re-drill within 90 days.
```

### 7.3 Drill Report Template

```markdown
# DR Drill Report — [YYYY-MM]

**Date:** YYYY-MM-DD
**Scenario exercised:** C — Data Corruption Simulation
**DR Lead:** [name / role]
**Observer:** [name / role]
**Environment:** Staging (Supabase project ID: [id])

## Timing Results

| Metric | Target | Actual | Pass/Fail |
|---|---|---|---|
| Time-to-detect (T1 – T0) | < 10 min | X min | |
| Time-to-contain (T2 – T0) | < 15 min | X min | |
| Time-to-restore (T3 – T0) | < 120 min (staging) | X min | |
| Data loss (unrecovered rows) | 0 | X | |
| Runbook gaps identified | 0 | X | |

## Observations

[Narrative of what happened, deviations from the runbook, tools that didn't work as expected.]

## RLS / Privacy Floor Verification

[ ] Confirmed: no cross-tenant data exposure in restored state.
[ ] Confirmed: HR-visible aggregates do not expose individual user data post-restore.

## Audit Log Integrity Check

[ ] HMAC chain verified intact (no break detected post-restore).
[ ] Audit gap period documented: [T0] to [T3].

## Gaps and Remediation

| Gap | Severity | Remediation | Owner | Due |
|---|---|---|---|---|
| [gap description] | High/Medium/Low | [action] | [role] | YYYY-MM-DD |

## Sign-off

**DR Lead:** _______________________ Date: YYYY-MM-DD
**Observer:** _______________________ Date: YYYY-MM-DD
**compliance-officer:** ______________ Date: YYYY-MM-DD

SOC 2 evidence: CC7.5, A1.3. Retain for 7 years per docs/SOC2_READINESS.md §15.1.
```

---

## 8. Incident Communication Plan

Communication during an outage must be consistent, timely, and non-speculative. Over-communication is preferred to silence.

### 8.1 Communication Channels and Ownership

| Channel | Audience | Owner | Trigger |
|---|---|---|---|
| `form.coach/status` | All users, public | On-call engineer | Any P0 or P1 declared |
| Enterprise customer Slack Connect | Enterprise tenant admins | CS lead / on-call | P0 or P1 affecting enterprise features |
| Direct email to DPA contact | Enterprise tenants (per contract) | compliance-officer | Any potential data breach; P0 affecting enterprise |
| Internal Slack `#incidents` | Engineering team | On-call engineer | All P0, P1 |
| PagerDuty escalation | On-call engineer → founder | Automated (PagerDuty) | P0 unacknowledged for > 5 min |

### 8.2 Initial Notification Template (< 30 min from P1 declaration)

> **Subject:** [FORM Status] Service Degradation — [YYYY-MM-DD HH:MM UTC]
>
> We are currently investigating an issue affecting [component]. Enterprise users may experience [specific impact].
>
> We have activated our incident response procedure. Our team is actively working on resolution.
>
> **Status page:** form.coach/status
> **Next update:** [HH:MM UTC, within 60 minutes]
>
> We will post updates every 60 minutes until resolved.

### 8.3 Hourly Update Template

> **Subject:** [FORM Status] Update #[N] — [YYYY-MM-DD HH:MM UTC]
>
> **Current status:** [Investigating / Identified / Mitigating / Resolved]
> **Impact:** [What is affected and for which tenant/user group]
> **What we know:** [Factual summary, no speculation]
> **What we're doing:** [Current action]
> **Estimated resolution:** [Time estimate, if known; otherwise "under investigation"]
>
> Next update: [HH:MM UTC]

### 8.4 Resolution and Post-Incident Summary Template

> **Subject:** [FORM Status] Resolved — [YYYY-MM-DD HH:MM UTC]
>
> **Incident summary:** [One paragraph: what happened, when, duration, user impact]
> **Root cause:** [Technical root cause, written for a non-technical audience]
> **Resolution:** [What was done to resolve it]
> **Data impact:** [None / Specify any data loss or exposure — be precise]
> **SLA credit:** [Whether a credit applies and approximately when it will appear on the next invoice]
> **What we're doing to prevent recurrence:** [Top 2–3 actions, with owners and timelines]
>
> We apologise for the disruption. If you have questions, contact your dedicated CSM or support@form.coach.

### 8.5 GDPR 72-Hour Breach Notification (Supervisory Authority)

If an incident involves confirmed or probable unauthorized access to health data or PII, the 72-hour GDPR Art. 33 clock starts at the moment of first suspicion — not at confirmed breach scope.

The `compliance-officer` agent is responsible for:
1. Starting and tracking the notification clock from first detection.
2. Preparing the Art. 33 notification using the template in `docs/INCIDENT_RESPONSE.md §7.2`.
3. Filing with the lead supervisory authority (Garante Privacy for Italy; ICO for UK; DPA for DE/FR/NL as applicable based on affected data subjects).
4. Logging all communication in the incident record.

---

## 9. Roles and Responsibilities

### 9.1 BCP Role Definitions

| Role | BCP Responsibility | Primary | Backup |
|---|---|---|---|
| **Incident Commander (IC)** | Declares severity, owns recovery timeline, makes go/no-go decisions | On-call engineer | Founder |
| **DR Lead** | Executes recovery runbooks; does not make severity decisions | On-call engineer | Founder |
| **Observer** | Independent witness to DR drill; signs drill report | compliance-officer | devops-lead |
| **Enterprise Comms** | Customer notifications, DPA contact, SLA credit | CS lead / compliance-officer | Founder |
| **GDPR Lead** | Art. 33 assessment, supervisory authority notification | compliance-officer | Founder |
| **Audit Log Custodian** | Verifies HMAC chain integrity during and after recovery | security-engineer | devops-lead |
| **Privacy Floor Verifier** | Confirms RLS invariants hold post-recovery | enterprise-architect | security-engineer |

### 9.2 On-Call Structure (Pre-Series-A)

Until the engineering team grows beyond a sole operator:

- **On-call engineer** = Founder (all roles above default to founder pre-hire).
- **Observer independence requirement:** During the annual DR drill, an independent observer (advisor, early employee, or compliance-officer agent) must witness and sign off. A solo drill without witness is a compensating measure accepted only during the pre-observation phase of SOC 2 (before Month O-1).
- PagerDuty is configured with the founder's phone as the primary escalation target. All monitoring alerts (Better Uptime, Sentry, Cloudflare) route through PagerDuty.

---

## 10. Continuity for Non-Infrastructure Failures

### 10.1 Key Person Dependency

FORM is currently operated by a single founder. This is a documented continuity risk.

**Mitigation:**
- All runbooks in this document are written for execution by a competent engineer without tribal knowledge from the author.
- All credentials are in 1Password team vault (not in the founder's personal password manager).
- Git repository on GitHub is the source of truth for all code and configuration.
- Recovery procedures are tested annually to validate that documentation is sufficient.

**Escalation path:** In the event of founder unavailability, the designated emergency contact (documented in the 1Password "Emergency" vault entry) has access to: 1Password team vault, GitHub repo, Cloudflare dashboard, Supabase dashboard, Backblaze B2.

**Gap:** No secondary operator exists pre-Series A. This is the highest continuity risk. Mitigation: first engineering hire will be granted full production access and trained on these runbooks within 30 days of start.

### 10.2 Third-Party Vendor Failure

For each critical third-party dependency, FORM has reviewed the vendor's own BCP/DR documentation:

| Vendor | SLA | FORM's Fallback |
|---|---|---|
| **Supabase** | 99.9% (Pro plan); DR architecture documented at supabase.com/docs/guides/platform/ha | Waiting + PITR restore; Scenario A/D runbooks |
| **Cloudflare** | 99.99% (Enterprise); global anycast | Waiting; no secondary; Scenario B runbook |
| **Anthropic** | No formal SLA on API availability | Fallback: disable AI coaching features; return static program templates; resume when API available |
| **ElevenLabs** | No formal SLA | Fallback: text-only coaching mode (pre-built UI flag `voice_coaching_enabled = false`) |
| **PostHog** | 99.9% | Fallback: none needed; analytics-only; no user-facing impact |
| **Sentry** | 99.9% | Fallback: Cloudflare Analytics + manual log review |

Vendor security and BCP assessments are reviewed annually by `compliance-officer` and documented in `docs/SUBPROCESSORS.md`.

### 10.3 Natural Disaster / Regional Infrastructure Outage

FORM uses AWS us-east-1 (N. Virginia) as the default region, with EU residency option in eu-central-1 (Frankfurt). A regional AWS outage would affect Supabase (hosted on AWS).

- Supabase has Multi-AZ within the selected region. A single AZ failure is handled by Supabase's own failover.
- A full regional AWS outage (extremely low probability) would be treated as Scenario A. If outage exceeds 4 hours, a cross-region migration would be evaluated — this is not currently scripted and would be a best-effort recovery.
- Cloudflare Workers run on Cloudflare's own global network (not AWS) and are not regionally vulnerable to AWS outages.

---

## 11. Known Gaps and Roadmap

All gaps are tracked in `docs/SOC2_READINESS.md §14` (Risk Register). This section summarises BCP-specific gaps.

| Gap ID | Description | Risk Level | Mitigation in Place | Target Resolution |
|---|---|---|---|---|
| BCP-01 | Automated nightly cold backup to Backblaze B2 not yet implemented. | High | Supabase PITR (7-day retention) covers most scenarios. | Q3 (automated pipeline) |
| BCP-02 | No secondary edge provider for Cloudflare Workers. | Medium | Cloudflare's own global redundancy (> 99.99% historical uptime). | Post-Series-A |
| BCP-03 | Sole operator (single founder) pre-Series A. | High | Written runbooks; 1Password team vault; annual DR drill with external observer. | First engineering hire (Month 6–9) |
| BCP-04 | Cold start (Scenario D) RTO of 8 hours exceeds enterprise 4-hour contractual RTO. | High | Scenario D requires complete Supabase account loss — extremely low probability. Scenario A and C runbooks achieve 4-hour RTO. | When automated cold backup + cross-region restore is scripted (Post-Series-A). |
| BCP-05 | DR drill requires external independent observer; unavailable solo. | Medium | Compensating: founder documents drill with timestamped screenshots. Accepted pre-observation period only. | First engineering hire acts as observer. |
| BCP-06 | PITR restore for Scenario D unavailable if Supabase project deleted. | High | See BCP-01. | Q3 cold backup automation. |

---

## 12. Document Control

| Field | Value |
|---|---|
| **Document version** | 1.0 |
| **Effective date** | 2026-06-06 |
| **Next review** | 2027-06-06 (annual) or after any DR drill or infrastructure change affecting RTO/RPO |
| **Owner** | `devops-lead` + `compliance-officer` |
| **Approver** | Founder |
| **Distribution** | Internal; shareable under NDA with enterprise customers upon request |
| **SOC 2 evidence** | CC7.4, CC7.5, A1.1, A1.2, A1.3, CC9.2 |
| **Related policies** | `docs/ACCEPTABLE_USE_POLICY.md`, `docs/INCIDENT_RESPONSE.md` |
| **Related technical docs** | `docs/ENTERPRISE_SLA.md`, `docs/DATA_MODEL.md §8–10`, `docs/OBSERVABILITY.md`, `docs/SOC2_READINESS.md §18` |
| **Audit log reference** | DEC-030 (HMAC-chained audit log integrity during recovery) |

### Change Log

| Version | Date | Author | Summary |
|---|---|---|---|
| 1.0 | 2026-06-06 | enterprise-builder (compliance-officer + devops-lead) | Initial document. Formalises BCP/DRP content from SOC2_READINESS.md §18 into standalone enterprise Policy Pack artifact. Closes SOC2 CC1.2 gap: AUP ✓, IRP ✓, BCP ✓. |
