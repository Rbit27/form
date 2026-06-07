# FORM · Change Management & Software Development Lifecycle Policy

**Policy ID:** POL-011
**Version:** 1.0
**Effective date:** 2026-06-07
**Owner:** devops-lead
**Reviewers:** security-engineer · compliance-officer
**Review cadence:** Annual; update on architecture change
**SOC 2 criteria:** CC8.1

---

## 1. Purpose

This policy defines how FORM authorizes, develops, tests, approves, and deploys changes to production infrastructure, application code, data schemas, and configuration. It establishes controls that prevent unauthorized changes, ensure quality gates before deployment, and create an auditable record of every change to the production environment.

---

## 2. Scope

This policy applies to:

- All changes to production application code (Cloudflare Workers, Cloudflare Pages, Supabase Edge Functions)
- All changes to production database schema, RLS policies, and seed data (Supabase PostgreSQL)
- All changes to infrastructure configuration (Cloudflare WAF rules, KV namespaces, DNS, Worker routes)
- All changes to environment secrets and API keys (beyond key rotation, which is governed by `docs/KEY_ROTATION.md`)
- All changes to CI/CD pipeline configuration (GitHub Actions workflows)
- All changes to third-party integration configuration (WorkOS, PostHog, ElevenLabs, Anthropic)

This policy does **not** cover:
- Local development environment changes
- Content updates to `content/` (blog posts, copy) that do not touch application logic
- Documentation-only changes in `docs/` that carry no infrastructure consequence

---

## 3. Change Classification

Every change is classified before the merge decision is made.

| Class | Definition | Examples | Approval required |
|---|---|---|---|
| **Standard** | Pre-approved routine change with well-understood risk; follows normal SDLC | Dependency version bump (patch-level), CI config tweak, Worker performance optimization, adding a new blog post file | 1 reviewer + CI pass |
| **Normal** | Non-emergency change that modifies application behaviour, schema, or infrastructure | New feature, schema migration, new Worker route, updated WAF rule, secrets rotation | 1 reviewer + CI pass + devops-lead sign-off |
| **Emergency** | Critical production issue requiring deploy without full review cycle (P0/P1 incident in progress) | Security patch for actively exploited CVE, rollback of broken deploy causing data loss | Founder or devops-lead verbal/Slack approval; retroactive written approval within 24 h |

When in doubt, classify **higher** (more restrictive).

---

## 4. Source Control & Branching

### 4.1 Branch model

- `main` is the single production branch. Direct commits to `main` are prohibited.
- All changes must be made on a feature or fix branch: `feature/<slug>`, `fix/<slug>`, `hotfix/<slug>`, `chore/<slug>`.
- Feature branches must be created from and merged into `main` only.

### 4.2 Branch protection rules (main)

The following rules must be enforced on `main` at all times:

| Rule | Requirement |
|---|---|
| Require pull request | All changes via PR — no direct push |
| Require CI to pass | All required status checks must pass before merge |
| Require code review | Minimum 1 approved review from a team member |
| No force push | `--force` and `--force-with-lease` prohibited |
| No branch deletion | `main` cannot be deleted |
| No bypass for administrators | Protection applies to all users including owner |

**Evidence artefact:** Screenshot of GitHub branch protection settings — `compliance/cc8/branch-protection-screenshot.png` — collected at initial setup and at each rule change.

### 4.3 Prohibited git operations

The following operations are explicitly prohibited on production branches:

```
git push --force          # prohibited
git push --force-with-lease  # prohibited on main
git commit --no-verify    # prohibited (bypasses pre-commit hooks)
git tag -d                # deleting existing tags prohibited
git rebase -i <published> # rewriting published history prohibited
```

---

## 5. CI/CD Pipeline Gates

### 5.1 Required checks before merge to main

All of the following status checks must pass before a PR can be merged. No exceptions outside the emergency change process (§7).

| Check | Tool | Failure action |
|---|---|---|
| Type check | TypeScript `tsc --noEmit` | Block merge |
| Lint | ESLint (zero errors, warnings-as-info) | Block merge |
| Unit tests | Vitest / Jest — 100% of existing tests must pass | Block merge |
| Integration tests | Relevant Edge Function tests | Block merge |
| Dependency audit | `npm audit --audit-level=high` — no HIGH or CRITICAL unpatched CVEs | Block merge |
| Build | `npm run build` exits 0 | Block merge |
| Bundle size gate | No unreviewed bundle increase > 20 kB gzipped | Block merge (flag for manual review) |

### 5.2 Post-merge / deploy checks

After successful merge and automated deploy:

| Check | Tool | SLA |
|---|---|---|
| Smoke test | Automated synthetic: login flow + workout creation | Within 5 min of deploy |
| Error rate check | Sentry — error rate < 1% on affected routes for 15 min | Monitored for 15 min post-deploy |
| Rollback trigger | Error rate ≥ 1% or smoke test failure → auto or manual rollback | Within 10 min of detection |

### 5.3 Pipeline configuration as evidence

The CI pipeline configuration file (`/.github/workflows/ci.yml`) is the authoritative definition of required checks. Any change to this file must itself pass through the PR + review process. A human-readable snapshot is maintained in `compliance/cc8/ci-pipeline-config.md` and updated on each pipeline change.

---

## 6. Code Review Requirements

### 6.1 Minimum review standard

Every PR must receive at least one approval from a team member who did not author the change. During the solo-founder phase (pre-first-hire), the founder may self-merge after CI passes for Standard changes; Normal changes require documented self-review checklist completion (see §6.2). Emergency changes follow §7.

This is a known compensating control for separation-of-duties gap (recorded in `docs/SOC2_READINESS.md`). It is re-evaluated upon first hire and must be replaced by true two-person review within 30 days of adding a second engineer.

### 6.2 Self-review checklist (solo-founder phase only)

For Normal changes during solo-founder phase, the author must complete the following in the PR description before merging:

- [ ] Change classification confirmed (Standard / Normal / Emergency)
- [ ] Schema migrations are backward-compatible or migration plan documented
- [ ] RLS policies reviewed — no tenant isolation regression
- [ ] No secrets committed (pre-commit hook confirms; manual scan performed)
- [ ] DEC-030 events emitted for relevant state changes
- [ ] Rollback plan documented for schema changes
- [ ] CHANGELOG.md updated
- [ ] VERSION bumped

### 6.3 Review focus areas

Reviewers must evaluate:

1. **Correctness** — does the change do what it claims?
2. **Security** — no SQL injection, no unauthorized data access, no secrets in code
3. **Tenant isolation** — no RLS bypass, no cross-tenant data leakage
4. **DEC-030 compliance** — audit events emitted for all state changes required by `docs/AUDIT_LOG_SCHEMA.md`
5. **Clinical safety** — any change touching user-facing content about food, body, pain, or mental health must have `clinical_safety_status: APPROVED` in PR description

---

## 7. Emergency Change Process

Emergency changes bypass the normal PR review cycle to address a critical production incident. The following constraints apply:

### 7.1 Authorization

An emergency change may be initiated only by:
- The founder, or
- The devops-lead (once hired), with founder notification within 30 minutes

Authorization must be recorded in Slack `#incidents` channel before or simultaneously with the deploy.

### 7.2 Emergency deploy procedure

1. Create `hotfix/<slug>` branch from `main`
2. Make the minimal change required to resolve the incident
3. Run CI locally: `npm run typecheck && npm run test && npm audit --audit-level=high`
4. Push branch and deploy directly (CI gate may be bypassed only if CI itself is unavailable or blocking resolution of the incident — document reason)
5. Notify `#incidents` with: branch name, change summary, incident reference, authorization
6. Open PR immediately after deploy for retroactive review
7. Complete retroactive written approval within 24 hours (see §7.3)

If CI is bypassed, note this explicitly in the emergency change log.

### 7.3 Retroactive approval (within 24 hours)

Within 24 hours of every emergency deploy, the following must be completed:

1. PR merged with documented self-review (§6.2) or peer review if second engineer available
2. Emergency change log entry added to `compliance/cc8/emergency-change-log.csv`:
   ```
   date, incident_ref, change_summary, authorized_by, ci_bypassed (Y/N), bypass_reason, retroactive_approval_by, retroactive_approval_date
   ```
3. Incident entry updated in `docs/INCIDENT_RESPONSE.md` with change reference
4. DEC-030 `system.deploy` event emitted with `emergency: true` flag

Failure to complete retroactive approval within 24 hours is a control failure and must be logged in `compliance/cc4/control-deficiency-log.csv`.

### 7.4 Emergency change DEC-030 event

```json
{
  "event": "system.deploy",
  "actor": "founder|devops-lead",
  "deploy_type": "emergency",
  "branch": "hotfix/<slug>",
  "incident_ref": "<incident-id>",
  "ci_bypassed": true | false,
  "bypass_reason": "<if bypassed>",
  "timestamp": "<ISO-8601>"
}
```

---

## 8. Rollback Procedures

### 8.1 When to rollback

Rollback is triggered by any of the following within 15 minutes of deploy:

- Smoke test failure (§5.2)
- Error rate ≥ 1% on affected routes (§5.2)
- P0 alert firing on a metric that was green pre-deploy
- User-reported functional regression confirmed within 30 minutes

### 8.2 Rollback methods by component

| Component | Rollback method | Expected time to recovery |
|---|---|---|
| Cloudflare Workers / Pages | Revert commit + push to trigger deploy, or Cloudflare dashboard rollback to previous deployment | < 5 minutes |
| Supabase Edge Functions | Deploy previous function version via Supabase CLI: `supabase functions deploy <name> --version <prev>` | < 10 minutes |
| Database schema migration (non-destructive) | Deploy migration rollback script; note: destructive migrations (column drop) require point-in-time recovery | < 15 minutes for reversible; 30-60 min for PITR |
| Database schema migration (destructive) | Supabase PITR restore to pre-migration snapshot; see `docs/ENGINEERING_RUNBOOK.md §recovery` | 30-60 minutes; RTO ≤ 4 h |
| Cloudflare WAF rule | Revert via Cloudflare dashboard or Terraform; rule change is near-instantaneous | < 2 minutes |
| KV namespace data | Restore from KV backup snapshot or re-run seed; see `docs/ENGINEERING_RUNBOOK.md §kv-recovery` | 15-30 minutes |

### 8.3 Rollback decision authority

- devops-lead (or founder) initiates rollback without committee approval
- Rollback decision must be recorded in `#incidents` Slack channel
- DEC-030 `system.deploy` event emitted with `rollback: true` flag

### 8.4 Post-rollback

Within 24 hours of rollback:
1. Root cause identified and documented in incident ticket
2. Fix developed and tested against the failure scenario before re-deploy
3. Post-incident review scheduled within 5 business days

---

## 9. Change Documentation (DEC-030 Audit Events)

Every production deploy must emit a `system.deploy` event to the HMAC-chained audit log (`docs/AUDIT_LOG_SCHEMA.md`).

### 9.1 Standard deploy event

```json
{
  "event": "system.deploy",
  "actor": "ci-system | <developer-id>",
  "deploy_type": "standard | normal",
  "commit_sha": "<SHA>",
  "branch": "<branch>",
  "pr_number": "<PR number>",
  "components_changed": ["workers", "pages", "edge-functions", "schema", "waf"],
  "version": "<new VERSION value>",
  "timestamp": "<ISO-8601>"
}
```

### 9.2 Configuration change event

Any change to infrastructure configuration (WAF rules, KV bindings, environment variables, Worker routes) must emit:

```json
{
  "event": "system.config_changed",
  "actor": "<developer-id>",
  "config_scope": "waf | kv | env | routes | dns",
  "description": "<human-readable summary>",
  "pr_number": "<PR number>",
  "timestamp": "<ISO-8601>"
}
```

### 9.3 Audit chain requirement

Both event types are subject to DEC-030 HMAC chaining. Any gap in the chain for `system.deploy` events constitutes a control failure and must be investigated per `docs/INCIDENT_RESPONSE.md R-05`.

---

## 10. Prohibited Changes

The following are explicitly prohibited and constitute policy violations:

| Prohibited action | Rationale |
|---|---|
| Direct commit to `main` without PR | Bypasses review and CI gate |
| Merge with failing CI | Bypasses quality gate |
| `git push --force` on `main` | Rewrites published history; destroys audit trail |
| `git commit --no-verify` | Bypasses pre-commit hooks including secret scanning |
| Deleting git tags | Tags are immutable release markers; deletion destroys audit trail |
| Committing secrets, API keys, or passwords | Key rotation required; secret exposure triggers R-05 |
| Modifying past DEC-030 audit log entries | Audit log is append-only and HMAC-chained; modification = integrity violation |
| Deploying schema migration without rollback plan for destructive operations | Risk of unrecoverable data loss |
| Bypassing `clinical_safety_status` gate for user-facing content | Required by DEC-018; cannot be waived |

Policy violations must be logged in `compliance/cc4/control-deficiency-log.csv` regardless of severity.

---

## 11. Compliance Calendar

| Cadence | Task | Artefact | SOC 2 Criteria |
|---|---|---|---|
| Quarterly | Change management review — volume, emergency changes, CI failure rate, rollback count, open deficiencies | `compliance/cc8/quarterly-change-review-YYYYQQ.md` | CC8.1 |
| Quarterly | Review emergency change log for unapproved entries | `compliance/cc8/emergency-change-log.csv` | CC8.1 |
| Annual | Policy review and re-approval | Updated row in `compliance/policy-approval-log.csv` | CC8.1 |
| On each pipeline change | Update `compliance/cc8/ci-pipeline-config.md` snapshot | `ci-pipeline-config.md` | CC8.1 |
| On each branch protection rule change | Re-capture `branch-protection-screenshot.png` | Screenshot | CC8.1 |
| At M3 hardening | Enforce all branch protection rules listed in §4.2; confirm CI pipeline implements all §5.1 checks | `branch-protection-screenshot.png` + `ci-pipeline-config.md` | CC8.1 |

---

## 12. SOC 2 Evidence Artefacts

| Artefact ID | Artefact | Description | Collection method | Criteria |
|---|---|---|---|---|
| CC8-E-001 | `compliance/cc8/change-management-policy.md` | This policy document (POL-011) | Git history; version at audit | CC8.1 |
| CC8-E-002 | `compliance/cc8/branch-protection-screenshot.png` | GitHub branch protection settings screenshot | Manual capture at setup and on rule change | CC8.1 |
| CC8-E-003 | `compliance/cc8/ci-pipeline-config.md` | CI pipeline required checks snapshot | Manual update on each pipeline change | CC8.1 |
| CC8-E-004 | `compliance/cc8/change-log.csv` | All production deploy DEC-030 events — commit SHA, PR, components, version, timestamp | Automated via DEC-030 export quarterly | CC8.1 |
| CC8-E-005 | `compliance/cc8/emergency-change-log.csv` | All emergency changes with retroactive approval records | Manual entry within 24 h of emergency deploy | CC8.1 |
| CC8-E-006 | `compliance/cc8/quarterly-change-review-YYYYQQ.md` | Quarterly review memo | Quarterly; compliance-officer sign-off | CC8.1 |
| CC8-E-007 | Pull request history (GitHub) | All merged PRs with review approvals and CI status | GitHub audit log export (admin API) | CC8.1 |
| CC8-E-008 | `compliance/policy-approval-log.csv` | Annual policy approval record for POL-011 | Annual review | CC8.1 |

---

## 13. Open Questions

| ID | Question | Owner | Priority | Target |
|---|---|---|---|---|
| OQ-CC8-01 | **Should the bundle size gate (§5.1) be enforced via a custom GitHub Action or Cloudflare Pages built-in size reporting?** Cloudflare Pages provides compressed asset size in the deploy log but does not fail the build on overage. A custom Action (e.g., `bundlesize` or `size-limit`) can set hard limits and fail CI. Recommendation: custom Action with `size-limit` for `npm run build` output. | devops-lead | P2 | M4 hardening |
| OQ-CC8-02 | **Once a second engineer joins, what is the required review turnaround SLA?** Current policy has no time-bound for review response. Recommendation: 1 business day for Standard; 4 hours for Normal during incident windows. Document in a future policy v1.1. | devops-lead + compliance-officer | P3 | On first hire |
| OQ-CC8-03 | **Should schema migrations require a dedicated review by enterprise-architect in addition to standard code review?** Migrations touching RLS policies or multi-tenant isolation are higher risk than code changes. Recommendation: yes — tag migrations with `area: schema` label; enterprise-architect must approve before merge. | enterprise-architect + devops-lead | P1 | M3 hardening |

---

*v1.0 (2026-06-07): Initial policy — POL-011. Covers change classification (Standard/Normal/Emergency), branch protection (§4), CI/CD gates (§5), code review requirements including solo-founder compensating control (§6), emergency change process with 24-h retroactive approval (§7), rollback procedures by component (§8), DEC-030 audit events (§9), prohibited changes (§10), compliance calendar (§11), and SOC 2 evidence artefacts CC8-E-001 through CC8-E-008 (§12). Closes four SOC2_READINESS.md gaps: "Change management (code review, CI gates)", "Production deploy requires CI pass", "Emergency change process", "Rollback capability documented". Owner: devops-lead.*
