# FORM · Change Management Policy

> **Owner:** `compliance-officer` + `security-engineer` + `enterprise-architect`
> **Effective:** June 2026 · **Review:** annually, or on any material change to deployment architecture or team composition
> **SOC 2 controls:** CC8.1 — authorises, tests, and documents changes before implementation; classifies changes by type; maintains rollback capability
> **References:** `docs/SOC2_READINESS.md §21`, `docs/ENGINEERING_RUNBOOK.md §2 §5`, `docs/AUDIT_LOG_SCHEMA.md`, `docs/DATA_MODEL.md §10–11`, DEC-010 (version-controlled repo), DEC-030 (HMAC-chained audit log)

---

## TL;DR

All production changes to FORM systems must be classified, authorised, and documented before deployment. Emergency changes (P0) follow a 2-hour single-approver fast path with mandatory post-hoc review within 24 hours. Standard changes require a PR with documented rationale. All deployed changes generate HMAC-chained audit events. No direct pushes to `main` without a PR; no production SQL run outside versioned migration files.

---

## 1. Purpose and Scope

CC8.1 requires that changes to infrastructure and software are authorised, tested, and documented before they enter production. For FORM, unauthorized or untested changes carry concrete risks:

- An unauthorized schema migration can alter RLS policies, triggering a Confidentiality (C1) failure
- An untested change to the coaching pipeline can produce incorrect outputs, triggering a Processing Integrity (PI1) failure
- A change deployed without a rollback path can extend an outage beyond the enterprise RTO commitment of 4 hours (A1.1)

**In scope — all production changes to:**

| Surface | Technology |
|---|---|
| API layer | Cloudflare Workers (scripts and routes) |
| Database schema | Supabase PostgreSQL (DDL via `/supabase/migrations/`) |
| Frontend | Cloudflare Pages (web app builds) |
| ML pipeline | Model files in Cloudflare R2; version pointer in Workers KV |
| Auth configuration | Supabase Auth settings, SAML/OIDC config, SCIM endpoints |

**Out of scope:** changes to internal tooling (Linear, 1Password, Notion) with no production data path; open-source dependency updates gated separately by Dependabot + CI.

---

## 2. Change Classification

Every production change is classified before it begins. Classification determines authorization requirements, maximum deployment SLA, and evidence obligations. When a change is ambiguous, it defaults to the next tier up.

| Type | Definition | Authorization | Max Deploy SLA |
|---|---|---|---|
| **Emergency (P0)** | Critical security patch, active breach response, production-down event — time to deploy measured in hours | Single-approver fast path; post-hoc review within 24 h; incident ticket opened simultaneously | 2 hours from decision to deploy |
| **Standard** | Feature work, refactor, schema migration, dependency update, auth configuration change | PR with documented rationale; solo phase: founder self-review with written justification; post-hire: ≥ 1 independent reviewer distinct from PR author | 24–72 hours from PR open to merge |
| **Minor** | Copy/content changes, documentation, configuration with no security impact and no data model effect | PR merge + auto-deploy; no separate approval step | Immediate on merge |

**Ambiguity rule:** If unclear whether Minor or Standard — especially if the change touches any file handling Restricted or Confidential data (per `docs/SOC2_READINESS.md §13`) — classify as Standard and document the rationale in the PR body.

**Change type examples:**

| Change | Type |
|---|---|
| SEV-0 CVE patch, RLS bypass hotfix, prod-down config restore | Emergency (P0) |
| New API endpoint, new schema migration, model version update, SCIM endpoint change | Standard |
| New `docs/` file, marketing copy update, non-sensitive env var tuning | Minor |

---

## 3. Change Authorisation Workflow

### 3.1 Emergency Change (P0)

```
[Incident detected]
       │
       ▼
Open incident ticket (FORM-INC-YYYY-NNN) ← simultaneous, not after
       │
       ▼
Create branch from main: hotfix/INC-NNN-short-description
       │
       ▼
Author fix, open PR with mandatory fields:
  - Incident ticket reference
  - Root cause (preliminary)
  - What the change does
  - Rollback plan
       │
       ▼
Single approver merges (founder solo phase; any senior engineer post-hire)
       │
       ▼
Deploy via wrangler or Cloudflare dashboard
       │
       ▼
DEC-030 audit event: system.deploy_completed (metadata.change_type: "emergency")
       │
       ▼
Post-hoc review within 24 h → close or follow-up issue filed
```

If "Include administrators" override is used to bypass CI gates: create a GitHub Issue tagged `admin-override` within 30 minutes. See §7.

### 3.2 Standard Change

```
[Work starts on feature branch]
       │
       ▼
PR opened against main
  PR body must include:
  - What is changing and why
  - Which SOC 2 surface is affected (if any)
  - Rollback plan
  - Schema migration included? (down migration mandatory)
       │
       ▼
CI gates (target state — CC8-GAP-001):
  ci/build · ci/test · ci/lint must pass
       │
       ▼
Review (solo phase: founder self-review; post-hire: ≥ 1 independent reviewer)
       │
       ▼
Merge to main → auto-deploy to production
       │
       ▼
DEC-030 audit event: system.deploy_completed (metadata.change_type: "standard")
```

### 3.3 Minor Change

PR opens, CI gates run (when live), merge triggers auto-deploy. No separate approval step. Minor change PRs that touch security-adjacent files are automatically reclassified to Standard by the reviewer who catches it.

---

## 4. Branch Protection Requirements

The following settings must be configured on the `main` branch of `github.com/Rbit27/form`. The screenshot of this configuration is SOC 2 evidence artefact **CC8-E-001**, filed to `/evidence/cc8/branch-protection-YYYY-MM-DD.png` in the private compliance repository.

| Setting | Required value | Gap |
|---|---|---|
| Require pull request before merging | Enabled | — |
| Required approvals | 1 | — |
| Dismiss stale reviews on new commits | Yes | — |
| Require status checks to pass before merging | Yes (when CI live) | CC8-GAP-001 |
| Require branches to be up to date before merging | Yes | — |
| Require linear history | Yes | — |
| Include administrators | No (solo phase — see §7) | CC8-GAP-002 |

**Evidence artefact CC8-E-001:** Screenshot of the branch protection settings page, filed quarterly. Re-file immediately on any settings change.

---

## 5. CI/CD Pipeline Gates

Automated CI is a primary CC8.1 control — it provides an objective, repeatable test gate that cannot be bypassed by a single human approving their own code.

| Dimension | Current state | Target state |
|---|---|---|
| Build gate | Manual | GitHub Actions `ci/build` on every PR |
| Test gate | No automated execution | GitHub Actions `ci/test`; failing test blocks merge |
| Lint gate | No automated enforcement | GitHub Actions `ci/lint` (ESLint + TypeScript); failure blocks merge |
| Deploy on merge | Manual `wrangler deploy` | GitHub Actions deploy job on merge to `main` |
| Deployment log | No structured archive | Actions run log archived to `r2://form-backups/ci-logs/YYYY/MM/` |

**Gap: CC8-GAP-001.** No CI pipeline is configured as of June 2026. This is a documented gap. Until CI is live, the compensating control is the manual pre-deploy checklist in `docs/ENGINEERING_RUNBOOK.md §2`. Every deploy executed under the manual path must confirm checklist completion in a PR comment before merge.

**Evidence artefact CC8-E-002** (target — once CI is live): GitHub Actions run log per production deploy, capturing: commit SHA, branch, PR number, test results, lint results, deploy timestamp, Cloudflare deploy hash. Auto-archived to R2 per deploy.

---

## 6. Database Schema Change Controls

Schema changes carry the highest risk of any change type. A bad migration can corrupt RLS policies, destroy data, or break tenant isolation in ways that are hard to reverse quickly.

**Mandatory rules — no exceptions:**

1. All schema changes must be delivered via versioned migration files in `/supabase/migrations/`. Direct `ALTER` or `CREATE` statements run from the Supabase dashboard console are prohibited in production.
2. Every migration file is reviewed under Standard change authorization (§3.2), regardless of apparent simplicity.
3. Every `up` migration must have a corresponding `down` migration in the same PR. A PR that adds an up migration without a down migration fails review.

**Migration reviewer checklist:**

| Check | Requirement |
|---|---|
| RLS policies | New or altered tables must have `ENABLE ROW LEVEL SECURITY` and an appropriate policy. Confirm in the DDL. |
| Tenant isolation | Every new table storing user or tenant data must have `tenant_id` FK referencing `tenants(id)`. Confirm in the migration DDL. |
| Index strategy | New tables with expected query volume must have indexes documented in the migration comment or in `docs/DATA_MODEL.md §11`. |
| Backward compatibility | If the migration is not backward-compatible (e.g., column drop that application code still reads), confirm a two-phase deploy plan is included in the PR. |
| Down migration tested | Reviewer confirms the down migration was executed in staging, not just authored. |

**Rollback for schema changes:**

- Within 30 minutes of deploy: run the down migration script from the PR. Confirm row counts. Emit `system.config_changed` audit event with `metadata.reason: "migration_rollback"`.
- Beyond 30 minutes / data already written: use Supabase PITR per `docs/DATA_MODEL.md §10`. RTO: 15 minutes to rollback decision; 4 hours to complete PITR restore.

**Evidence artefact CC8-E-003:** Migration git history (`git log supabase/migrations/`) — complete, tamper-evident record of every DDL change with author, timestamp, and PR link. Automated; no manual filing required.

**Evidence artefact CC8-E-004:** PITR restore test record. Quarterly test of Supabase PITR targeting the schema migration recovery path. Filed to `/evidence/cc8/pitr-test-YYYY-MM.md`. Owner: `devops-lead`.

---

## 7. Rollback Procedures

All production surfaces must have a documented, tested rollback path. Target: 15 minutes to rollback decision, surface-specific RTO to execution.

| Surface | Rollback method | RTO |
|---|---|---|
| **Cloudflare Workers** | Dashboard → Workers → select deployment → "Rollback to this version"; or CLI `wrangler rollback` | < 5 min |
| **Cloudflare Pages** | Dashboard → Pages → Deployments → select any prior deployment → "Rollback" | < 5 min |
| **Supabase schema** | Run down migration script from the PR; PITR as fallback | < 15 min (down migration); < 4 h (PITR) |
| **ML model** | Update `model_version` KV pointer to prior version's R2 key — model files are never overwritten | < 5 min |

**Quarterly rollback drill.** Each path above must be tested in a staging environment quarterly. Results filed alongside the DR drill evidence (`docs/BUSINESS_CONTINUITY.md`). Any RTO exceedance is a gap finding requiring remediation within 30 days.

---

## 8. Separation of Duties — Solo-Founder Phase

This section documents the structural gap honestly. SOC 2 auditors reviewing a pre-revenue solo-founder company will not fail a Type I readiness assessment solely on this basis — provided the gap is disclosed in the Management Assertion Letter with compensating controls documented.

**The structural gap:** One person (the founder) can author code, approve the PR, and execute the production deploy. No technical control presently prevents this.

**Compensating controls:**

1. All production changes go through a PR, even from the sole author. Every change has an immutable git history entry. There is no mechanism to deploy code that was never in a PR.
2. Written rationale is mandatory in the PR body for all Standard and Emergency changes (what is changing, why, rollback plan, security impact assessment).
3. Automated test gate (CC8-GAP-001 target) will reduce the human error surface — a failing test blocks deploy regardless of who authored the change.
4. The audit log is append-only and HMAC-chained (DEC-030). No deployment event can be edited after the fact. An auditor can independently verify the complete deployment history.

**Auditor disclosure language.** The Management Assertion Letter must include a paragraph disclosing the separation of duties limitation and listing these four compensating controls. Auditors at firms experienced with early-stage SaaS (Prescient Assurance, Johanson Group) routinely accept this pattern as a finding-with-compensating-controls rather than a qualified opinion, provided the controls are demonstrably operational throughout the observation period.

**Post-hire target.** The moment a second engineer joins: branch protection is updated to require review from a code owner distinct from the PR author; `CODEOWNERS` is created mapping production-path directories to the first engineering hire as required reviewer; this gap moves from 🟡 Partial to 🟢 Done.

---

## 9. Include-Administrators Override (Emergency Exception)

GitHub's "Include administrators" setting is deliberately set to **off** during the solo-founder phase. If the branch protection required a reviewer other than the author, a P0 production outage could not be remediated. This is a documented exception, not an oversight.

**Compensating controls governing every admin override use:**

1. Within 30 minutes: create a GitHub Issue tagged `admin-override`. The issue must reference: PR number, incident ticket that justified the override, planned post-hoc review date.
2. Post-hoc review within 24 hours: founder reviews the merged change as an independent reviewer. Any concerns tracked as follow-up issues. The `admin-override` issue is updated with the review outcome.
3. Every `admin-override` issue is reviewed at the next quarterly access review to confirm the post-hoc review occurred within the 24-hour SLA.

**Evidence artefact CC8-E-005:** All GitHub Issues tagged `admin-override` at `github.com/Rbit27/form/issues?label=admin-override`. Refresh cadence: per occurrence.

**Post-hire target.** After first engineering hire: set "Include administrators" to **on**. Emergency changes use the ≥ 1 independent reviewer fast path. Admin-override exception is retired.

---

## 10. DEC-030 Audit Events

All production deploys and significant change lifecycle events generate HMAC-chained audit events per `docs/AUDIT_LOG_SCHEMA.md`. These cannot be edited or deleted after creation.

| Event name | Severity | Retention | Trigger | Key metadata |
|---|---|---|---|---|
| `system.deploy_completed` | STANDARD | 3 years | Every production deploy (Workers, Pages, ML model pointer) | `surface`, `change_type` (emergency / standard / minor), `commit_sha`, `pr_number`, `deployed_by`, `rollback_available` |
| `system.config_changed` | HIGH | 7 years | Auth configuration change; Supabase Auth settings; SAML/OIDC config update; branch protection modification | `surface`, `change_description`, `changed_by`, `reason` |
| `system.schema_migration_applied` | HIGH | 7 years | Every `supabase db push` to production | `migration_file`, `author`, `pr_number`, `rls_verified` (bool), `down_migration_included` (bool) |
| `system.schema_migration_rolledback` | HIGH | 7 years | Down migration execution or PITR restore | `migration_file`, `rollback_method` (down_migration / pitr), `reason` |
| `security.admin_override_used` | HIGH | 7 years | PR merged via "Include administrators" override | `pr_number`, `incident_ticket`, `merged_by`, `post_hoc_review_due` |
| `system.ci_gate_bypassed` | HIGH | 7 years | Deploy executed under CC8-GAP-001 manual compensating control (pre-CI era) | `pr_number`, `checklist_confirmed_by`, `deploy_hash` |

All events carry: `event_id` (UUIDv4), `tenant_id: "form_internal"`, `timestamp` (ISO 8601 UTC), `hmac_chain_sequence`, `prev_event_hmac`. Chain integrity is verified by weekly cron per `docs/SOC2_READINESS.md §25.5`.

---

## 11. Evidence Package for SOC 2 Auditors

| Evidence ID | Description | Location | Refresh cadence | Owner |
|---|---|---|---|---|
| **CC8-E-001** | Branch protection screenshot — `main` branch settings | `/evidence/cc8/branch-protection-YYYY-MM-DD.png` | Quarterly; immediately on settings change | founder |
| **CC8-E-002** | CI/CD run log per production deploy — test results, lint results, deploy hash, commit SHA | GitHub Actions history + `r2://form-backups/ci-logs/YYYY/MM/` | Per deploy (automated once CC8-GAP-001 closed) | automated |
| **CC8-E-003** | Migration file git history — complete DDL change record | `git log supabase/migrations/` | Per deploy (automated) | automated |
| **CC8-E-004** | Quarterly PITR restore test record | `/evidence/cc8/pitr-test-YYYY-MM.md` | Quarterly | devops-lead |
| **CC8-E-005** | Admin-override issue log — all GitHub Issues tagged `admin-override` | `github.com/Rbit27/form/issues?label=admin-override` | Per occurrence | founder |
| **CC8-E-006** | PR merge history — all changes merged to `main` with author, reviewer, CI status | `github.com/Rbit27/form/pulls?state=closed` + GitHub audit log export | Per change (automated) | automated |

**Auditor note on CC8-E-002:** Unavailable until CC8-GAP-001 (CI pipeline) is closed. Compensating control during the gap: manual pre-deploy checklist (`docs/ENGINEERING_RUNBOOK.md §2`) confirmed in a PR comment before every merge. This evidence lives in the PR comment history (CC8-E-006).

---

## 12. Open Gaps and Remediation Plan

| Gap ID | Description | Severity | Owner | Target |
|---|---|---|---|---|
| **CC8-GAP-001** | No CI pipeline configured. Automated test / lint / build gate does not exist. Compensating control: manual pre-deploy checklist. | 🟡 Partial | platform-engineer | Before enterprise GA (M10) |
| **CC8-GAP-002** | "Include administrators" branch protection is off (solo-founder phase). Admin-override logging procedure compensates. | 🟡 Partial | founder | Expires at first engineering hire |

**Neither gap is a blocker for SOC 2 Type I.** Both are disclosed in the Management Assertion Letter with the compensating controls documented in this policy. Both resolve automatically at specific hiring/engineering milestones.

---

## 13. Policy Compliance and Exceptions

**Enforcement:** `security-engineer` reviews PRs for classification compliance during quarterly access reviews. Any deploy that appears to have bypassed the classification workflow is investigated as a potential CC8 finding.

**Exception request:** Any exception to this policy (e.g., bypassing the down migration requirement for an additive-only migration) must be documented in the PR body, approved by `compliance-officer`, and logged as a `system.config_changed` DEC-030 event with `metadata.reason: "change_management_exception"`.

**Policy review:** This document is reviewed annually. Material changes to the deployment architecture (e.g., introduction of Kubernetes, multi-region deploy, self-hosted CI) trigger an out-of-cycle review. Reviews are logged in `docs/DECISION_LOG.md`.

---

## Appendix A — Pre-Deploy Checklist (CC8-GAP-001 Compensating Control)

Until CC8-GAP-001 is closed, every Standard and Emergency deploy must confirm these items in a PR comment before merge. Minor changes are exempt.

```
FORM · Pre-Deploy Checklist — [PR #NNN] — [YYYY-MM-DD]

Change type: [ ] Emergency  [ ] Standard
Deploy surface: [ ] Workers  [ ] Pages  [ ] Schema  [ ] ML model  [ ] Auth config

□ PR body describes: what changes, why, which SOC 2 surface is affected
□ Rollback plan documented in PR body
□ Schema migration: down migration included and tested in staging (if applicable)
□ RLS policies verified for any new or altered tables (if applicable)
□ No direct SQL run from Supabase dashboard
□ No secrets or PII committed in plaintext
□ Monitoring alerts reviewed post-deploy (30-min watch period)
□ DEC-030 deploy event will be emitted on completion

Confirmed by: [name] — [timestamp UTC]
```

This comment must be present and unflagged before merge. Its absence during the CC8-GAP-001 era is a finding in the next quarterly review.

---

## Appendix B — Post-Hire Transition Checklist

When the first engineering hire joins, the following change management controls must be updated within 14 days:

```
□ Update GitHub branch protection: Required approvals → 1 (from non-author reviewer)
□ Create CODEOWNERS file mapping production-path directories to the new engineer
□ Set "Include administrators" to ON
□ Retire the admin-override exception process (§9) — issue closed with note "post-hire SoD satisfied"
□ Enable CC8-GAP-002 closure: file CC8-E-001 screenshot with new settings
□ Verify first-hire has read and signed this policy (email confirmation → /evidence/cc8/policy-acknowledgement-YYYY.md)
□ Log DEC-030 event: system.config_changed (metadata.reason: "post_hire_sod_controls_activated")
□ Update docs/SOC2_READINESS.md §21 gap table: CC8-GAP-002 → 🟢 Done
```

---

*v1.0 (2026-06-15): Initial standalone policy. Extracts and formalises CC8.1 change management controls from `docs/SOC2_READINESS.md §21`. Covers: purpose and scope, change classification (Emergency / Standard / Minor), authorisation workflows for all three types, GitHub branch protection requirements (CC8-E-001), CI/CD pipeline state and CC8-GAP-001 documentation, database schema change controls with mandatory reviewer checklist, rollback procedures for all four production surfaces (Workers, Pages, schema, ML model), separation of duties analysis with solo-founder compensating controls and auditor disclosure guidance, include-administrators override exception procedure (CC8-E-005), six DEC-030 HMAC-chained audit events for the change lifecycle, evidence package (CC8-E-001 through CC8-E-006), open gaps register (CC8-GAP-001, CC8-GAP-002), exception procedure, Appendix A pre-deploy checklist (CC8-GAP-001 compensating control), Appendix B post-hire transition checklist. Both open gaps are 🟡 Partial — non-blocking for SOC 2 Type I; resolve at CI-pipeline milestone and first engineering hire respectively. Cross-references: `docs/SOC2_READINESS.md §21`, `docs/ENGINEERING_RUNBOOK.md §2 §5`, `docs/AUDIT_LOG_SCHEMA.md`, `docs/DATA_MODEL.md §10–11`, `docs/BUSINESS_CONTINUITY.md`, DEC-010, DEC-030. Owner: compliance-officer + security-engineer + enterprise-architect.*
