# FORM · CC8 Quarterly Change Review

**Document ID:** CC8-E-006  
**Template version:** v1.0 (2026-06-07)  
**File naming:** `quarterly-change-review-YYYYQQ.md` — replace `YYYY` with the four-digit year and `QQ` with `Q1`, `Q2`, `Q3`, or `Q4` before saving  
**Quarter under review:** [FILL: e.g., Q1 2026 (2026-01-01 – 2026-03-31)]  
**Review date:** [FILL: YYYY-MM-DD]  
**Reviewer:** [FILL: Name, Title]  
**Founder approval:** [FILL: Name — required for every quarterly review]  
**SOC 2 criteria:** CC8.1 — Changes authorized, designed, developed, configured, documented, tested, approved, and implemented  
**Policy reference:** `compliance/cc8/change-management-policy.md` (POL-011)  
**Owner:** devops-lead

---

> **Auditor note.** This document is the quarterly retrospective required by POL-011 §11. It covers every production change deployed during the quarter, demonstrates that change management controls operated effectively, and surfaces any deficiencies for remediation. Supporting evidence is cross-referenced throughout. Retain for a minimum of three years.

---

## 1. Change Volume Summary

> **Auditor note.** The figures below are derived from `compliance/cc8/change-log.csv` (CC8-E-004), which is populated from DEC-030 `system.deploy` audit events. Totals must reconcile with the GitHub commit/merge history for `main` and the Cloudflare Pages deploy log. Any discrepancy must be investigated and explained in Section 8 (Open Findings).

| Metric | Count | Notes |
|---|---|---|
| Total commits merged to `main` | [FILL: integer] | Source: GitHub — Insights → Network, filtered to quarter |
| Production deploys (Cloudflare Workers / Pages) | [FILL: integer] | Source: Cloudflare dashboard deploy log; must match DEC-030 `system.deploy` event count |
| Production deploys (Supabase Edge Functions) | [FILL: integer] | Source: Supabase CLI deploy history |
| Schema migrations executed in production | [FILL: integer] | Source: Supabase migration history; cross-ref `change-log.csv` |
| Emergency changes | [FILL: integer] | Must match row count in Section 3 and in `emergency-change-log.csv` |
| Rollback events | [FILL: integer] | Must match row count in Section 6 |
| DEC-030 `system.deploy` events in audit log | [FILL: integer] | Source: audit log export; must equal total production deploys above |
| DEC-030 `system.config_changed` events in audit log | [FILL: integer] | Source: audit log export; covers WAF, KV, env, routes, DNS changes |
| Commits with CI gate bypassed | [FILL: integer — target: 0] | Any non-zero value requires explanation; see Section 4 |
| Direct commits to `main` (branch protection violations) | [FILL: integer — target: 0] | Any non-zero value is a control failure; see Section 5 |

**Quarter-over-quarter trend (prior quarter):**

| Metric | This quarter | Prior quarter | Delta |
|---|---|---|---|
| Total production deploys | [FILL] | [FILL — or N/A if first quarter] | [FILL] |
| Emergency changes | [FILL] | [FILL] | [FILL] |
| Rollbacks | [FILL] | [FILL] | [FILL] |

---

## 2. Change Classification Breakdown

> **Auditor note.** Every merged PR must carry a change classification (Standard / Normal / Emergency) in accordance with POL-011 §3. The figures below should reconcile with the PR list export from GitHub.

| Classification | Count | % of total |
|---|---|---|
| Standard (pre-approved routine) | [FILL] | [FILL]% |
| Normal (modified application behaviour, schema, or infrastructure) | [FILL] | [FILL]% |
| Emergency | [FILL] | [FILL]% |
| **Total** | [FILL] | 100% |

**PR list source:** GitHub repository → Pull Requests → Merged, filtered to [FILL: quarter date range]. Export available via GitHub Admin API or `gh pr list --state merged --json number,title,mergedAt,labels`.

---

## 3. Emergency Change Log Summary

> **Auditor note.** Each row below must have a corresponding entry in `compliance/cc8/emergency-change-log.csv` (CC8-E-005). Retroactive written approval must be completed within 24 hours of the emergency deploy per POL-011 §7.3. Missing or late retroactive approvals are control failures and must be logged in `compliance/cc4/control-deficiency-log.csv`. If there were no emergency changes this quarter, replace the placeholder rows with "None — no emergency changes occurred during this quarter."

| EC ID | Date (UTC) | Incident ref | Change summary | CI bypassed? | Bypass reason | Retroactive approval by | Retroactive approval date | Within 24 h? | DEC-030 event emitted? |
|---|---|---|---|---|---|---|---|---|---|
| EC-[FILL]-001 | [FILL: YYYY-MM-DD] | [FILL: INC-XXX] | [FILL: one-line description] | [FILL: Y / N] | [FILL: reason or N/A] | [FILL: name] | [FILL: YYYY-MM-DD] | [FILL: Y / N] | [FILL: Y / N] |
| EC-[FILL]-002 | [FILL] | [FILL] | [FILL] | [FILL] | [FILL] | [FILL] | [FILL] | [FILL] | [FILL] |

**Add rows as needed. Remove unused placeholder rows before filing.**

**Emergency change rate this quarter:** [FILL: integer] emergency change(s) out of [FILL: integer] total deploys ([FILL: percentage]%)

**Observations:** [FILL: narrative commentary — e.g., "All emergency changes received retroactive approval within 24 hours. CI was not bypassed in any instance." or describe exceptions.]

---

## 4. CI/CD Gate Compliance

> **Auditor note.** POL-011 §5.1 requires all seven CI checks (type check, lint, unit tests, integration tests, dependency audit, build, bundle size gate) to pass before any PR is merged to `main`. This section confirms that requirement was met for every deploy during the quarter. Evidence: GitHub Actions run logs, accessible via the commit SHA or PR number.

### 4.1 CI gate summary

| Question | Answer |
|---|---|
| Did all non-emergency production deploys pass all required CI checks before merge? | [FILL: Yes / No] |
| Total CI runs triggered on PRs targeting `main` | [FILL: integer] |
| CI runs that passed all required checks | [FILL: integer] |
| CI runs that failed (PR blocked — never merged with failing CI) | [FILL: integer] |
| PRs merged with one or more failing CI checks (exceptions) | [FILL: integer — target: 0] |

### 4.2 CI check coverage confirmation

Confirm each required check was active and enforced throughout the quarter (POL-011 §5.1):

| Required check | Tool | Status throughout quarter | Notes |
|---|---|---|---|
| Type check | TypeScript `tsc --noEmit` | [FILL: Active / Temporarily disabled — date and reason] | |
| Lint | ESLint | [FILL: Active / Temporarily disabled] | |
| Unit tests | Vitest / Jest | [FILL: Active / Temporarily disabled] | |
| Integration tests | Supabase Edge Function tests | [FILL: Active / Temporarily disabled] | |
| Dependency audit | `npm audit --audit-level=high` | [FILL: Active / Temporarily disabled] | |
| Build | `npm run build` | [FILL: Active / Temporarily disabled] | |
| Bundle size gate | `size-limit` or Cloudflare Pages size report | [FILL: Active / Temporarily disabled] | |

### 4.3 CI exceptions (non-zero entries require explanation)

> If no exceptions occurred, state "None." For each exception, provide the PR number, date, reason the exception was granted, who authorized it, and how the risk was mitigated.

[FILL: None. / Or list exceptions in the format: PR #NNN (YYYY-MM-DD) — reason — authorized by — mitigation.]

### 4.4 Post-deploy smoke test results

| Deploy event | Smoke test result | Error rate within 15 min post-deploy | Action taken |
|---|---|---|---|
| [FILL: PR #NNN / commit SHA] | [FILL: Pass / Fail] | [FILL: < 1% / ≥ 1%] | [FILL: None / Rollback triggered] |

**Add rows for each deploy. If no failures occurred, a single summary row is acceptable: "All [N] deploys passed smoke tests; error rate remained < 1% post-deploy."**

---

## 5. Branch Protection Violations

> **Auditor note.** POL-011 §4.2 prohibits direct commits to `main` without a PR. Branch protection rules enforce this technically on GitHub; this section confirms no violations occurred. A violation means the branch protection configuration failed or was temporarily disabled — both are control failures. Evidence: GitHub audit log (`Organization settings → Audit log` or API endpoint `GET /repos/{owner}/{repo}/events`).

**Zero-tolerance policy:** Any direct commit to `main`, force-push to `main`, or temporary disabling of branch protection rules constitutes a control failure that must be logged in `compliance/cc4/control-deficiency-log.csv` regardless of intent or outcome.

| Question | Answer |
|---|---|
| Were any direct commits pushed to `main` without a PR during the quarter? | [FILL: No / Yes — see below] |
| Were branch protection rules disabled at any point during the quarter? | [FILL: No / Yes — see below] |
| Were any force pushes made to `main` during the quarter? | [FILL: No / Yes — see below] |
| Were any `git commit --no-verify` commits merged to `main`? | [FILL: No / Yes — see below] |

**Violations (if any):**

[FILL: None detected. Branch protection rules were active and enforced throughout the quarter, confirmed via GitHub audit log export dated [FILL: YYYY-MM-DD] / Or describe each violation: date, committer, nature of violation, detection method, remediation action, deficiency log reference.]

**Evidence:** GitHub audit log covering [FILL: quarter date range] reviewed on [FILL: YYYY-MM-DD]. Audit log export stored at: [FILL: path or storage location].

---

## 6. Rollback Events

> **Auditor note.** POL-011 §8 defines rollback triggers (smoke test failure, error rate ≥ 1%, P0 alert, user-reported regression confirmed within 30 minutes) and rollback methods by component. Each rollback must emit a DEC-030 `system.deploy` event with `rollback: true`. Post-rollback root cause analysis must be completed within 24 hours and a post-incident review held within 5 business days. If no rollbacks occurred, state "None" below.

| Rollback ID | Date (UTC) | Trigger | PR / commit ref rolled back | Component(s) affected | Rollback method | Time to recovery | Root cause summary | Post-incident review date | DEC-030 event emitted? |
|---|---|---|---|---|---|---|---|---|---|
| RB-[FILL]-001 | [FILL: YYYY-MM-DD] | [FILL: Smoke test / Error rate / P0 alert / User report] | [FILL: PR #NNN or SHA] | [FILL: Workers / Pages / Edge Functions / Schema] | [FILL: Revert commit / CF dashboard / Supabase CLI / PITR] | [FILL: N min] | [FILL: one-line RCA] | [FILL: YYYY-MM-DD] | [FILL: Y / N] |

**Add rows as needed. If no rollbacks occurred this quarter:** None — no rollback events occurred during [FILL: quarter].

**Total rollbacks this quarter:** [FILL: integer]  
**Rollback-to-deploy ratio:** [FILL: e.g., 1:47 or 0%]

---

## 7. Control Effectiveness Rating

> **Auditor note.** Rate each CC8.1 sub-criterion as **Effective** (control operated as designed throughout the quarter with no unmitigated failures) or **Deficient** (one or more control failures occurred). Any Deficient rating requires a corresponding open finding in Section 8 and a logged entry in `compliance/cc4/control-deficiency-log.csv`. Provide a concise rationale for each rating based on the evidence reviewed in Sections 1–6.

| Sub-criterion | CC8.1 requirement | Rating | Rationale |
|---|---|---|---|
| CC8.1-a | Changes are authorized before implementation | [FILL: Effective / Deficient] | [FILL: e.g., "All [N] Normal changes approved via PR review; [N] Emergency changes received verbal + retroactive written approval within 24 h per POL-011 §7." or describe deficiency.] |
| CC8.1-b | Changes are designed and developed following documented procedures | [FILL: Effective / Deficient] | [FILL: e.g., "All changes followed branching model (POL-011 §4.1); self-review checklist completed on all Normal changes during solo-founder phase."] |
| CC8.1-c | Changes are configured and tested before implementation | [FILL: Effective / Deficient] | [FILL: e.g., "All non-emergency deploys passed all 7 required CI checks (POL-011 §5.1); no merge-with-failing-CI exceptions. Post-deploy smoke tests passed for all deploys."] |
| CC8.1-d | Changes are documented | [FILL: Effective / Deficient] | [FILL: e.g., "CHANGELOG.md and VERSION updated on all Normal changes; DEC-030 system.deploy events emitted for all [N] deploys with no gaps in HMAC chain."] |
| CC8.1-e | Changes are approved before implementation | [FILL: Effective / Deficient] | [FILL: e.g., "PR approval recorded in GitHub for all Standard and Normal changes; emergency changes received documented founder authorization. Solo-founder compensating control (POL-011 §6.1) applied and disclosed."] |
| CC8.1-f | Changes are implemented in an authorized manner | [FILL: Effective / Deficient] | [FILL: e.g., "No direct commits to main; branch protection rules active throughout the quarter; no force pushes detected per GitHub audit log."] |
| CC8.1-g | Emergency changes are controlled and documented | [FILL: Effective / Deficient] | [FILL: e.g., "No emergency changes this quarter." or "N emergency changes; all received retroactive approval within 24 h; all logged in emergency-change-log.csv; DEC-030 events emitted."] |
| CC8.1-h | Rollbacks can be executed and are documented when used | [FILL: Effective / Deficient] | [FILL: e.g., "Rollback procedures documented in POL-011 §8 and confirmed exercisable. No rollbacks required this quarter." or summarize rollback events.] |

**Overall CC8.1 rating this quarter:** [FILL: Effective / Deficient]

[FILL: One-sentence overall summary — e.g., "CC8.1 controls operated effectively throughout Q1 2026 with no unmitigated deficiencies." or describe any exceptions.]

---

## 8. Open Findings

> **Auditor note.** List all control deficiencies identified during this review. A deficiency is any instance where a control failed to operate as designed, was not present, or produced an exception that was not fully mitigated. Each finding must be logged in `compliance/cc4/control-deficiency-log.csv` before this review document is finalized. Reference the CSV row ID here. If no deficiencies were identified, state "None."

**Deficiency log:** `compliance/cc4/control-deficiency-log.csv`

| Finding ID | Section ref | Description | Severity (Low / Med / High / Critical) | Control deficiency log ref | Remediation owner | Target remediation date | Status |
|---|---|---|---|---|---|---|---|
| [FILL: F-YYYYQQ-001] | [FILL: §N] | [FILL: Description of the deficiency — what failed, when, what evidence supports the finding] | [FILL] | [FILL: DEF-NNN in cc4/control-deficiency-log.csv] | [FILL: name] | [FILL: YYYY-MM-DD] | [FILL: Open / In progress / Closed] |

**If no deficiencies:** None — no control deficiencies were identified during the [FILL: quarter] review.

**Carryover findings from prior quarter:**

| Finding ID | Prior quarter ref | Description | Status update |
|---|---|---|---|
| [FILL: F-YYYYQQ-001] | [FILL: prior quarter doc] | [FILL: description] | [FILL: Remediated YYYY-MM-DD / Still open — updated target date] |

**If no carryover findings:** None.

---

## 9. Sign-Off

> **Auditor note.** Both fields are required. The reviewer confirms that the evidence cited in this document was examined and that the ratings in Section 7 reflect the evidence. The founder's signature constitutes management's acceptance of the review findings. For the audit period, this document — combined with the cited evidence artefacts — should be presented to the auditor.

| Role | Name | Date | Signature / Confirmation |
|---|---|---|---|
| Reviewer (devops-lead) | [FILL: Full name] | [FILL: YYYY-MM-DD] | [FILL: "I confirm the evidence cited in this document was reviewed and the ratings reflect my professional assessment." — or wet / electronic signature] |
| Founder approval | [FILL: Full name] | [FILL: YYYY-MM-DD] | [FILL: "I have reviewed and approve this quarterly change review." — or wet / electronic signature] |

**Evidence artefacts reviewed during this quarterly review:**

| Artefact ID | File / Source | Version / Export date reviewed |
|---|---|---|
| CC8-E-004 | `compliance/cc8/change-log.csv` | [FILL: export date] |
| CC8-E-005 | `compliance/cc8/emergency-change-log.csv` | [FILL: export date] |
| CC8-E-002 | `compliance/cc8/branch-protection-screenshot.png` | [FILL: screenshot date — confirm rules unchanged or note changes] |
| CC8-E-003 | `compliance/cc8/ci-pipeline-config.md` | [FILL: version/date — confirm pipeline unchanged or note changes] |
| CC8-E-007 | GitHub PR history (merged PRs for quarter) | [FILL: export date / API call date] |
| — | GitHub audit log (branch protection, force push events) | [FILL: export date] |
| — | Cloudflare Pages deploy log | [FILL: date range reviewed] |
| — | DEC-030 audit log export (`system.deploy` + `system.config_changed`) | [FILL: export date] |

---

## 10. Footer

**Policy reference:** `compliance/cc8/change-management-policy.md` (POL-011 v1.0)  
**SOC 2 criteria:** CC8.1  
**Evidence index:** `compliance/cc8/README.md`  
**Deficiency log:** `compliance/cc4/control-deficiency-log.csv`  
**Next quarterly review due:** [FILL: YYYY-MM-DD — first business day of the month following quarter end]  
**Retention:** Minimum 3 years from review date  
**Document owner:** devops-lead  
**Template version:** v1.0 · 2026-06-07
