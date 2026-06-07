# Vulnerability Management & Patching SLA Policy

**Policy ID:** POL-010  
**Version:** 1.0  
**Status:** IN_FORCE  
**Effective date:** 2026-06-07  
**Owner:** `security-engineer` + `compliance-officer`  
**Review cadence:** Quarterly per compliance calendar (§15.1 of `docs/SOC2_READINESS.md`); also triggered by any P0/P1 security incident or CVE-triggered emergency deployment  
**Next review:** 2026-09-07  
**SOC 2 criteria:** CC7.1 · CC7.2 · CC7.3 · CC7.4 · CC5.3  
**Source document:** `docs/SOC2_READINESS.md §41` (authoritative; this file is the standalone auditor exhibit)  
**Artefact ID:** CC5-E-003 (CC5.3 standalone Vulnerability Management Policy)

---

## 1. Purpose and Scope

This policy establishes FORM's Vulnerability Management & Patching SLA Programme. It defines how FORM discovers, classifies, remediates, and documents software vulnerabilities across its production environment, and discloses material findings to enterprise customers.

This document is the binding standalone policy for SOC 2 criteria CC7.1 and CC7.2. It closes PRE-15 (patching SLA enforced and tracked) and CC5.3 (standalone Vulnerability Management Policy document) as identified in the FORM SOC 2 readiness programme.

### 1.1 Components in Scope

| Layer | Components | Patching authority |
|---|---|---|
| Application dependencies | npm packages in `workers/`, `supabase/functions/`, React Native app | FORM — security-engineer + devops-lead |
| Application runtime | Node.js version (`.nvmrc`); Cloudflare Workers runtime | FORM controls Node.js target; CF runtime is vendor-managed (§8) |
| Database engine | Supabase Postgres major/minor version | Supabase-managed; FORM monitors advisories and requests upgrades (§8) |
| Mobile client | React Native version, Expo SDK, third-party RN packages | FORM — coordinated via `expo upgrade` and Dependabot |
| CI/CD toolchain | GitHub Actions runners, EAS build workers, Wrangler CLI | GitHub/EAS managed for runners; FORM-pinned for CLI versions |
| **Out of scope** | Cloudflare global network, Supabase PostgreSQL kernel, Apple/Google OS | Vendor responsibility; monitored via vendor security advisories (§4.4) |

### 1.2 Privacy Constraint

No CVE triage discussion, patch PR description, or Linear ticket shall include health data content, user identifiers, or coaching transcript excerpts. Vulnerability context must be expressed in structural and architectural terms only. This constraint applies to all five DEC-030 `vuln.*` audit events (§11).

---

## 2. Vulnerability Discovery Sources

FORM maintains four independent discovery channels:

### 2.1 Dependabot Automated Dependency Scanning

GitHub Dependabot is configured via `.github/dependabot.yml` to scan npm ecosystems weekly (Monday 06:00 UTC). Alerts route to `security-engineer` as reviewer. Critical-severity findings generate Dependabot PRs; `security-engineer` opens a Linear `[SEC]` ticket for any finding with CVSS ≥ 7.0.

Required Dependabot configuration baseline:

| Setting | Required value | Rationale |
|---|---|---|
| `schedule.interval` | `weekly` or tighter | Ensures ≤ 7-day discovery lag for new CVEs |
| `open-pull-requests-limit` | ≥ 10 per ecosystem | Prevents silent halt when limit is hit |
| `reviewers` | `["security-engineer"]` | Security-engineer sees every dependency PR |
| `labels` | `["security", "dependencies"]` | Enables Linear automation to detect Dependabot PRs |
| `groups.patch-updates` | enabled | Reduces merge friction for patch-only updates |

### 2.2 `npm audit` CI Gate

The CI workflow runs `npm audit --audit-level=critical` as a **required blocking check** on every pull request to `main`. Any Critical finding (CVSS ≥ 9.0) fails the build and blocks merge. A separate `npm audit --audit-level=high` non-blocking warning step runs until promotion to blocking gate 30 days before SOC 2 observation period start.

### 2.3 Annual External Penetration Test

Per the penetration test programme (`docs/SOC2_READINESS.md §16`), FORM commissions an annual external penetration test (OWASP WSTG + ASVS L2 + MASVS L1). All findings with CVSS ≥ 7.0 trigger the patching SLA (§3) from the engagement close date (report delivery).

### 2.4 Vendor Security Advisory Monitoring

| Vendor | Advisory channel | Owner | Action threshold |
|---|---|---|---|
| Cloudflare | `blog.cloudflare.com/tag/security`, Trust Hub | devops-lead | Any advisory referencing Workers, D1, R2, or WAF |
| Supabase | GitHub `supabase/supabase` releases, security@supabase.io | devops-lead | Any advisory referencing Postgres, Auth, Storage, or Edge Functions |
| Anthropic | `docs.anthropic.com/changelog`, Anthropic Trust Center | security-engineer | Any advisory referencing API authentication, data retention, or model security |
| ElevenLabs | ElevenLabs changelog, security@elevenlabs.io | security-engineer | Any advisory referencing API key security or TTS data handling |
| Node.js | `nodejs.org/en/blog/vulnerability` | devops-lead | All published CVEs; applies to `.nvmrc`-pinned runtime version |
| Expo / EAS | `expo.dev/changelog`, GitHub `expo/expo` security advisories | devops-lead | Any advisory referencing OTA update signing, EAS token security, or app credential handling |
| GitHub | GitHub Security Blog | security-engineer | Advisories affecting Actions runners, GHCR, or secret scanning |

Response SLA: within 24 hours of a Critical advisory, `security-engineer` must assess applicability and open a Linear ticket if FORM's stack is affected. Within 72 hours for High advisories.

### 2.5 Responsible Disclosure

A formal bug bounty programme is not currently active. The responsible disclosure path is `security@form.coach` with a 72-hour acknowledgment SLA. All received reports enter the §5 triage workflow. Target: launch a formal HackerOne programme at Series A or 10k MAU, whichever comes first.

---

## 3. Severity Classification

FORM uses CVSS v3.1 Base Score as the primary severity classifier, with three FORM-specific uplift rules.

### 3.1 Base Severity Tiers

| CVSS v3.1 Base Score | FORM tier | Linear priority |
|---|---|---|
| 9.0 – 10.0 | **Critical** | P0 |
| 7.0 – 8.9 | **High** | P1 |
| 4.0 – 6.9 | **Medium** | P2 |
| 0.1 – 3.9 | **Low** | P3 |
| 0.0 / informational | **Informational** | Backlog |

### 3.2 FORM-Specific Uplift Rules

The following conditions elevate a finding **one tier** above its CVSS Base Score:

| Rule | Condition | Example |
|---|---|---|
| **U-01 — Health data exposure path** | Vulnerable component has a code path processing GDPR Art. 9 special-category data (`health_profiles`, `coaching_turns`, `wearable_data`, `cv_sessions.keypoints_enc`) | CVSS 5.5 Medium npm package in a coaching Worker → elevated to High |
| **U-02 — RLS bypass potential** | Vulnerability could construct a SQL injection or auth bypass circumventing Supabase Row-Level Security, enabling cross-tenant data access | Any SQL injection < 7.0 in the API layer is auto-elevated to **Critical** — cross-tenant exposure is always P0 |
| **U-03 — HMAC chain integrity** | Vulnerability could allow forging or modifying `audit_log_events` entries, breaking the DEC-030 HMAC chain | A cryptographic library vulnerability affecting HMAC-SHA256 is elevated one tier regardless of base score |

When uplift applies, the elevated tier governs the patching SLA. The uplift rationale must be documented in the Linear ticket `severity_rationale` field.

---

## 4. Patching SLA

These SLAs are FORM's binding internal commitments and are disclosed to enterprise customers on request (Critical/High details under NDA per §10).

| Severity | CVSS | Patching SLA | Clock start | Max extension | Escalation if missed |
|---|---|---|---|---|---|
| **Critical** | 9.0–10.0 | **24 hours** | First notification (Dependabot alert, advisory publication, or pentest report delivery — whichever earliest) | +24 h with compliance-officer approval; engineering lead must personally approve deploy | Page founder immediately; open `#inc-YYYYMMDD-crit-cve`; treat as P0 per `docs/INCIDENT_RESPONSE.md` |
| **High** | 7.0–8.9 | **7 calendar days** | First notification | +7 d with security-engineer + compliance-officer approval; risk acceptance memo required | Page security-engineer at T+5 d if patch not in staging; Linear ticket auto-escalated to P0 |
| **Medium** | 4.0–6.9 | **30 calendar days** | Linear ticket creation date (triage complete) | +30 d with compliance-officer sign-off; documented risk assessment required | Weekly automated reminder; escalated to High at T+25 d if no patch PR |
| **Low** | 0.1–3.9 | **90 calendar days** | Linear ticket creation date | +90 d acceptable with documented rationale | Quarterly bulk review; close as Won't Fix only with compliance-officer approval |
| **Informational** | 0.0 | Next scheduled sprint | Backlog grooming | N/A | Monthly backlog review |

**SLA applies to:** having a patch merged to `main` **and deployed to production** (or a fully documented risk acceptance on file). A PR open but not merged does not satisfy the SLA.

**No upstream patch:** If no patch is available from the upstream maintainer, FORM must either (a) implement a compensating control (WAF rule, code-level sanitisation, feature flag disable) within the SLA window, or (b) initiate a Risk Acceptance (§7). "No upstream patch available" does not pause the SLA clock.

---

## 5. SLA Clock Mechanics

### 5.1 Clock Start

The SLA clock starts at the **earliest** of:

1. Timestamp of first GitHub Dependabot alert notification email received by `security-engineer`
2. Timestamp of CVE publication in GHSA or NVD, if FORM's stack is observably affected
3. Engagement close date on a penetration test report containing the finding
4. Timestamp of `security@form.coach` receipt for a responsible disclosure report
5. Timestamp when any FORM team member becomes aware of a vulnerability through any channel

The clock start event must be recorded in the Linear ticket `vuln_discovered_at` field within 2 hours of awareness.

### 5.2 Clock Stop

The SLA clock stops when **both** of the following are true:
- The patch PR has been merged to `main` (GitHub merge timestamp)
- The change has been deployed to production (Wrangler deploy log timestamp, or EAS Submit confirmation)

If a compensating control is applied instead of a patch, the clock stops when the control is confirmed active in production and documented in the Linear ticket.

### 5.3 Clock Pause (Exceptional Cases Only)

| Pause condition | Approver | Maximum pause |
|---|---|---|
| Awaiting upstream maintainer patch (patch PR submitted) | security-engineer | 14 calendar days; after that, Risk Acceptance required |
| Supabase/Cloudflare managed-service advisory — patch is vendor-side, no FORM action required | devops-lead | Until vendor confirms patch deployed; FORM must verify within 24 h of vendor announcement |
| CI pipeline broken due to unrelated issue blocking the security PR from merging | devops-lead | 8 hours maximum; break-glass deploy allowed if CI failure is confirmed unrelated |

---

## 6. CVE Triage & Remediation Workflow

```
[DISCOVERY]
  Dependabot alert / npm audit / pentest finding / advisory / responsible disclosure
       │
       ▼
[TRIAGE]  security-engineer  (≤ 4 h for Critical/High; ≤ 24 h for Medium)
  1. Read CVE description and affected version range
  2. Confirm FORM is on an affected version (package.json / .nvmrc)
  3. Apply uplift rules §3.2 (U-01, U-02, U-03) — document rationale if uplift applied
  4. Assign final severity tier
  5. Open Linear ticket: [SEC] CVE-YYYY-NNNN — <package> (<severity>)
     Required fields: vuln_discovered_at, cvss_base, final_severity,
                      sla_deadline, affected_components[], uplift_applied, uplift_rationale
       │
       ▼
[ASSESSMENT]  security-engineer + devops-lead
  · Patched version available?       → Yes: proceed to Patch PR
  · No patched version?              → Compensating control OR Risk Acceptance §7
  · Health data exposure (U-01)?     → Notify compliance-officer
  · RLS bypass potential (U-02)?     → Page security-engineer immediately (P0)
       │
       ▼
[PATCH PR]
  1. Create branch: fix/cve-YYYY-NNNN
  2. Update package.json / package-lock.json to patched version
  3. Run full test suite locally: npm test, npm run build
  4. Run npm audit to confirm CVE resolved
  5. PR description: severity, CVSS score, SLA deadline, affected component, brief impact
     NOTE: no health data content in PR description (§1.2 privacy constraint)
  6. CI must pass npm audit --audit-level=critical
  7. Required reviewers: security-engineer (always) + second reviewer for Critical/High
       │
       ▼
[STAGING DEPLOY + VERIFY]
  · Deploy to staging environment
  · Confirm affected functionality still works (smoke test)
  · For U-02 uplift: run __tests__/db/rls_isolation.test.ts against staging
  · For U-03 uplift: run HMAC chain integrity verification against staging audit log
       │
       ▼
[PRODUCTION DEPLOY]
  · Merge PR to main (GitHub merge timestamp = SLA clock stop candidate)
  · CI/CD auto-deploys via Wrangler / EAS (deploy timestamp = SLA clock stop)
  · Record deploy timestamp in Linear ticket patched_at field
  · For Critical: post confirmation in #security-alerts channel
       │
       ▼
[EVIDENCE FILING]  compliance-officer files to compliance/evidence/vuln-management/YYYY/
  · Linear ticket export (JSON) — artefact CC7-E-001
  · Git commit hash of patch merge
  · npm audit post-patch output (zero Critical/High findings)
  · Production deploy log excerpt (Wrangler deploy timestamp)
  · For U-01/U-02/U-03: uplift rationale memo
```

---

## 7. Risk Acceptance & SLA Exception Protocol

Risk Acceptance formally acknowledges an extended timeline and requires compensating controls. It does **not** excuse FORM from eventual remediation.

### 7.1 Eligibility Criteria

Risk Acceptance is permitted **only** when all of the following are true:

1. No upstream patch exists, or the patch introduces a breaking change requiring material refactoring
2. At least one compensating control is deployed or formally described
3. The vulnerability does **not** trigger U-02 (RLS bypass potential — these are never eligible for Risk Acceptance; the feature must be disabled if no patch is available)
4. The risk acceptance has a defined expiry date (no indefinite acceptances)

### 7.2 Approval Matrix

| Severity | Approver | Max duration | Compensating control | Evidence |
|---|---|---|---|---|
| **Critical** | Founder + compliance-officer (joint) | 7 days | Mandatory — WAF block, feature flag disable, or equivalent | `compliance/evidence/vuln-management/risk-acceptance/YYYY-NNNN.md` |
| **High** | security-engineer + compliance-officer | 30 days | Mandatory | Same as Critical |
| **Medium** | security-engineer | 60 days | Recommended; document why not applied if absent | Linear ticket |
| **Low** | security-engineer | 90 days | Optional | Linear ticket or close as Won't Fix |

### 7.3 Won't Fix Policy

A vulnerability may be closed as Won't Fix only when:
- CVSS Base Score ≤ 3.9 (Low or Informational)
- The vulnerable component is not reachable from production traffic (confirmed dead code or disabled feature)
- Approved by security-engineer with rationale documented in Linear

Won't Fix decisions are reviewed annually during the Q4 vulnerability management audit.

---

## 8. Platform & Runtime Patching (Vendor-Managed Components)

| Component | Patch authority | FORM monitoring action | FORM response SLA |
|---|---|---|---|
| **Cloudflare Workers runtime** | Cloudflare — automatic | Monitor Cloudflare Trust Hub; subscribe to `cf-announce` email list | No action for automatic patches; verify functionality within 24 h of deployment announcement |
| **Cloudflare WAF / OWASP CRS** | Cloudflare — managed ruleset auto-updated | Monitor Cloudflare WAF changelog quarterly | Confirm OWASP CRS version active in dashboard; export evidence as CC7-E-004 |
| **Supabase Postgres** | Supabase — maintenance-window upgrades | Monitor Supabase changelog; review `nvd.nist.gov` for Postgres CVEs | For Critical Postgres CVEs: contact Supabase support within 24 h to confirm patch status |
| **Expo / EAS OTA runtime** | Expo — Hermes engine auto-versioned | Monitor `expo.dev/changelog` and GitHub security advisories | Run `expo upgrade` within 30 days of a security-tagged SDK release; Critical findings: 7-day SLA |
| **GitHub Actions runners** | GitHub — `ubuntu-latest` auto-patched | Monitor GitHub Changelog for runner security updates | Confirm CI workflows do not pin to a runner version with a known CVE |
| **Apple iOS / Google Android OS** | Apple / Google — end-user update | Not in FORM's patch scope | BYOD policy (§40.6 of SOC2_READINESS.md) requires OS updates within 30 days of Critical OS patch |

---

## 9. Enterprise Customer Disclosure Policy

Per `docs/ENTERPRISE.md §What we promise customers`, FORM commits to sharing an **annual penetration test summary** and **quarterly compliance attestations**. This section extends that commitment to in-year CVE disclosures.

### 9.1 Disclosure Tiers

| Severity | Disclosure | Timing | Format |
|---|---|---|---|
| **Critical** (U-02 — cross-tenant risk) | Mandatory — all active enterprise tenants | Within 2 hours of confirming cross-tenant exposure scope | Template E-VULN-01 |
| **Critical** (no cross-tenant risk) | Optional — notify tenant admin if their data type is in affected component | Within 24 hours of patch deploy | Template E-VULN-02 |
| **High** | Included in quarterly compliance attestation | Next scheduled quarterly attestation | Aggregate: "N High CVEs remediated; all within SLA" |
| **Medium / Low** | Included in annual pentest summary | Annual | Aggregate count + SLA adherence rate |

Customers with ACV ≥ $50k/year may request the full pentest executive summary under mutual NDA. Available within 30 days of the annual engagement close.

### 9.2 Disclosure Templates

**Template E-VULN-01 — Critical CVE with cross-tenant exposure potential**

```
Subject: FORM Security Notice — Critical CVE [CVE-YYYY-NNNN] — Action required

Dear [Tenant Admin Name],

We are writing to notify you of a Critical security vulnerability (CVE-YYYY-NNNN,
CVSS [score]) identified in [component] on [date]. This vulnerability affected
[affected data types in structural terms — no individual user data].

Current status: [Patch deployed to production at HH:MM UTC / Compensating control active]

Impact to your account: [Based on our investigation, your tenant data was not
accessed / We have no evidence of exploitation during the exposure window
[start]–[end] / We are continuing to investigate and will update you within [N] hours]

Actions required from you: [None at this time / Please [specific action if any]]

For questions, contact your Customer Success Manager or security@form.coach.

FORM Security Team
```

**Template E-VULN-02 — Critical CVE, no cross-tenant risk**

```
Subject: FORM Security Notice — Critical CVE Remediated [CVE-YYYY-NNNN]

Dear [Tenant Admin Name],

This notice confirms that FORM has identified and remediated a Critical security
vulnerability (CVE-YYYY-NNNN) affecting [component]. The patch was deployed on
[date] within our 24-hour Critical patching SLA.

No cross-tenant data exposure was possible through this vulnerability. Your
organisation's data was not at risk.

No action is required on your part. This notification is provided as part of
FORM's quarterly compliance attestation programme.

FORM Security Team
```

---

## 10. DEC-030 Audit Events

All vulnerability management lifecycle events are HMAC-chained per `docs/AUDIT_LOG_SCHEMA.md` (DEC-030). A chain break on any `vuln.*` event is a P0 incident per `docs/INCIDENT_RESPONSE.md §R-05`.

| Event type | Trigger | Logged by | Severity | Retention | Key payload fields |
|---|---|---|---|---|---|
| `vuln.cve_discovered` | Linear ticket created for CVSS ≥ 4.0 finding | security-engineer (manual via admin API) | MEDIUM | 7 years | `cve_id`, `cvss_base`, `final_severity`, `uplift_applied`, `uplift_rules[]`, `component`, `affected_version`, `sla_deadline` |
| `vuln.patch_deployed` | Production deploy of the patching PR confirmed | devops-lead (automated via CI post-deploy hook) | MEDIUM | 7 years | `cve_id`, `patch_pr_url`, `merged_at`, `deployed_at`, `sla_met` (bool), `elapsed_hours` |
| `vuln.sla_exception_granted` | Risk acceptance approved per §7.2 | compliance-officer (manual via admin API) | HIGH | 7 years | `cve_id`, `original_severity`, `approved_by[]`, `new_deadline`, `compensating_control`, `rationale_ref` |
| `vuln.wont_fix_closed` | Linear ticket closed as Won't Fix per §7.3 | security-engineer | MEDIUM | 7 years | `cve_id`, `cvss_base`, `rationale`, `approved_by` |
| `vuln.enterprise_notified` | Template E-VULN-01 or E-VULN-02 sent to enterprise tenant | customer-success (manual) | MEDIUM | 7 years | `cve_id`, `tenant_id`, `template_used`, `sent_by`, `notified_at` |

**Privacy note:** No `vuln.*` payload may contain `user_id`, `session_id`, health data, or coaching content. `cve_id`, `component`, and `affected_version` are infrastructure identifiers with no user data linkage.

---

## 11. SOC 2 Evidence Mapping

### 11.1 Control Criteria

| SOC 2 criterion | How this policy satisfies it |
|---|---|
| **CC7.1 — Threat identification** | §2 documents four discovery sources; §6 CI gate; §10 DEC-030 `vuln.*` events create a continuous audit trail of every discovered vulnerability |
| **CC7.2 — Vulnerability monitoring** | §6 triage workflow with Linear ticket per CVSS ≥ 4.0; §4 SLA table; §5 clock mechanics |
| **CC7.3 — Security event evaluation** | §3.2 uplift rules (U-01 Art. 9 health data, U-02 RLS bypass, U-03 HMAC chain) constitute the security event evaluation step |
| **CC7.4 — Response to identified events** | §4 SLA table; §6 remediation workflow; §7 risk acceptance; cross-reference to `docs/INCIDENT_RESPONSE.md` R-08 |
| **CC5.3 — Vulnerability Management Policy** | This document is the standalone Vulnerability Management Policy. Reference from `compliance/policy-approval-log.csv` POL-010 |
| **CC6.8 — Prevent malicious software** | §2.2 Dependabot + `npm audit --audit-level=critical` CI gate; TruffleHog scan (CC5-P1-003) |

### 11.2 Evidence Artefacts

| Artefact ID | Description | Location | Cadence | Criteria |
|---|---|---|---|---|
| CC7-E-001 | Linear vulnerability ticket export — all `[SEC]` tickets during observation period | `compliance/evidence/vuln-management/YYYY/linear-export-YYYY-QN.json` | Quarterly + at audit fieldwork | CC7.1, CC7.2 |
| CC7-E-002 | `npm audit` CI gate pass record — GitHub Actions workflow run logs | `compliance/evidence/vuln-management/YYYY/ci-audit-gate-YYYY-QN.txt` | Per audit period | CC7.1, CC7.2 |
| CC7-E-003 | Dependabot alert closure log — all alerts opened and closed during observation period | GitHub Security tab → CSV export | Quarterly | CC7.1 |
| CC7-E-004 | Cloudflare WAF rule configuration export | `compliance/evidence/vuln-management/YYYY/cloudflare-waf-YYYY-QN.json` | At any rule change + quarterly | CC7.1 |
| CC7-E-005 | `audit_log_events` query filtered on `event_type LIKE 'vuln.%'`, HMAC chain verified | `compliance/evidence/vuln-management/YYYY/dec030-vuln-chain-YYYY.json` | At audit fieldwork | CC7.1, CC7.2 |
| CC7-E-006 | Risk acceptance memos for any SLA exceptions during observation period | `compliance/evidence/vuln-management/risk-acceptance/YYYY/` | Per exception | CC7.2 |
| CC7-E-007 | Responsible disclosure inbox log | `compliance/evidence/vuln-management/YYYY/responsible-disclosure-log.csv` | Quarterly | CC7.1 |
| CC7-E-008 | Vendor advisory monitoring log | `compliance/evidence/vuln-management/YYYY/vendor-advisory-log.csv` | Monthly | CC7.1 |
| CC7-E-009 | SLA adherence summary — per-severity tier totals, SLA met rate, exceptions | `compliance/evidence/vuln-management/YYYY/sla-adherence-YYYY.md` | Quarterly + annual | CC7.1, CC7.2 |

---

## 12. Gap Closure Record

| Gap | Before this policy | After this policy |
|---|---|---|
| **PRE-15** — Patching SLA enforced and tracked | 🔴 Open — thresholds mentioned in §16 but no standalone policy or evidence path | 🟢 **Done** — §4 SLA table is the binding policy; §6 workflow with Linear ticket produces CC7-E-001 |
| **CC7.1** — Vulnerability identification | 🟡 Partial — pentest programme defined; dependency scanning not documented as policy | 🟡 **Advancing** — four discovery sources (§2), severity classification (§3), triage workflow (§6); advances to 🟢 after first annual pentest and first full quarter of CC7-E-001 |
| **CC7.2** — Remediation tracking | 🟡 Partial — "first test + Linear tickets" required | 🟡 **Advancing** — Linear ticket schema (§6), SLA clock mechanics (§5), DEC-030 events (§10); advances to 🟢 after first quarter of `vuln.*` events in audit log |
| **CC5.3** — Vulnerability Management Policy (standalone) | 🟡 Partial — programme defined in SOC2_READINESS.md but not separated as standalone policy | 🟢 **Done** — this document is the standalone policy; `compliance/policy-approval-log.csv` POL-010 is the registration record |

---

## 13. Implementation Checklist

| # | Task | Owner | Priority | Status |
|---|---|---|---|---|
| 1 | Add `.github/dependabot.yml` per §2.1 baseline; confirm security-engineer is assigned reviewer | devops-lead | **P0** | ☐ Pre-launch |
| 2 | Merge `npm audit --audit-level=critical` CI gate (closes CC6-GAP-005); validate it fails on a known-vulnerable package in a test branch | devops-lead | **P0** | ☐ Pre-launch |
| 3 | Add `npm audit --audit-level=high` as non-blocking warning step; promote to blocking 30 days before SOC 2 observation start | devops-lead | **P1** | ☐ Pre-SOC 2 |
| 4 | Create Linear `[SEC]` ticket template with all required fields from §6 triage step | security-engineer | **P0** | ☐ Pre-launch |
| 5 | Subscribe security-engineer and devops-lead to all vendor advisory channels (§2.4); document confirmations in `compliance/evidence/vuln-management/advisory-subscriptions.md` | security-engineer | **P0** | ☐ Pre-launch |
| 6 | Register five `vuln.*` DEC-030 event types from §10 in `docs/AUDIT_LOG_SCHEMA.md` event registry | security-engineer | **P1** | ☐ M3 |
| 7 | Implement `vuln.cve_discovered` and `vuln.patch_deployed` DEC-030 event emission | platform-engineer + security-engineer | **P1** | ☐ M4 |
| 8 | Create `compliance/evidence/vuln-management/` directory structure with README | compliance-officer | **P0** | ☑ Done — `compliance/evidence/vuln-management/README.md` |
| 9 | File this policy in `compliance/cc7/vuln-management-policy.md`; record in `compliance/policy-approval-log.csv` | compliance-officer | **P0** | ☑ Done — this document |
| 10 | Add TruffleHog CI scan (CC5-P1-003) per §2 complement; closes CC5.1 credential-leakage control gap | security-engineer | **P1** | ☐ Pre-SOC 2 |
| 11 | Implement `vuln.enterprise_notified` DEC-030 event and wire Templates E-VULN-01/E-VULN-02 to `#enterprise-security` Slack | platform-engineer + customer-success | **P1** | ☐ M4 |
| 12 | Produce first CC7-E-009 SLA adherence summary after first full quarter of operation | security-engineer | **P2** | ☐ Q3 2026 |

---

*v1.0 · 2026-06-07 · security-engineer + compliance-officer*  
*Filed per `docs/SOC2_READINESS.md §41.14` checklist item 9. Closes PRE-15 🔴 → 🟢. Advances CC5.3 🟡 → 🟢. CC7.1 and CC7.2 advance toward 🟢 on first quarter of operational evidence.*
