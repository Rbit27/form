# FORM · Penetration Testing Program v1.0

> Authoritative program document for FORM's annual external penetration test.
> Owner: `security-engineer` + `compliance-officer`. Reviewed annually (February, post-test-cycle close).
> SOC 2 criteria: **CC7.1** (threat identification), **CC7.2** (vulnerability management), **CC9.2** (vendor risk — testing firm).
> References: `docs/AUDIT_LOG_SCHEMA.md` (DEC-030), `docs/INCIDENT_RESPONSE.md`, `docs/VULNERABILITY_MANAGEMENT.md`, `docs/ENTERPRISE.md §7`, `docs/SOC2_READINESS.md §16`.

---

## TL;DR

FORM conducts an external penetration test at minimum once per year (Q1). An independent CREST-accredited firm tests the production API surface, authentication flows, SSO/SCIM endpoints, admin dashboard, mobile apps, and Cloudflare edge configuration against the OWASP WSTG v4.2, ASVS Level 2, MASVS L1/L2, and CWE Top 25 frameworks. Critical and High findings must be remediated within 24 hours and 7 calendar days respectively. An executive summary is shared with enterprise customers (ACV ≥ $50k) under mutual NDA. Full evidence artifacts are filed to `compliance/pentest/YYYY/` in the private compliance repository.

This document supersedes the embedded description in `docs/SOC2_READINESS.md §16`. §16 now cross-references this document and is authoritative only for gap-status tracking.

---

## 1. Purpose and Mandate

### 1.1 Why this program exists

The penetration testing program formally identifies security vulnerabilities in FORM's production systems before an attacker finds them. It serves three simultaneous purposes:

1. **Technical assurance** — confirms that FORM's data isolation controls (RLS, RBAC, tenant scoping) hold against adversarial manipulation of the API surface.
2. **Enterprise sales enablement** — `docs/ENTERPRISE.md §7` lists "Annual external penetration test; executive summary on request" as a hard commitment to enterprise customers. This program is what fulfils that commitment.
3. **SOC 2 evidence** — closes CC7.1 (system operations: threat and vulnerability identification) and CC7.2 (vulnerability management: remediation tracking), both of which were 🔴 Gap pre-program.

### 1.2 Relationship to other security controls

| Control | Relationship |
|---|---|
| `docs/VULNERABILITY_MANAGEMENT.md` | Defines patching SLAs for dependency CVEs (Dependabot / Snyk); pentest findings follow the same severity nomenclature but have distinct SLAs defined in §8 |
| `docs/INCIDENT_RESPONSE.md` | Critical findings discovered during a test are treated as P1 incidents; P0 classification if RLS bypass is confirmed |
| `docs/RESPONSIBLE_DISCLOSURE.md §14` | External researchers disclose bugs via coordinated disclosure; pentest is an *invited* assessment — different safe-harbour terms |
| `docs/AUDIT_LOG_SCHEMA.md` (DEC-030) | Every finding intake, remediation, and closure emits a DEC-030 HMAC-chained event (§12) |
| `docs/SOC2_READINESS.md §16` | Gap-status view and cross-reference only; operational detail lives here |

---

## 2. Program Structure

| Phase | Timing | Duration |
|---|---|---|
| Firm selection and SOW sign | Q4 (November–December) | 3–4 weeks |
| Pre-engagement setup | January | 1 week |
| Active testing window | Late January – early February (Q1) | 2–3 weeks |
| Report delivery | Within 7 days of test close | — |
| Remediation (Critical + High) | 24h / 7d from report delivery | — |
| Re-test of Critical + High | Within 30 days of remediation | 3–5 days |
| Evidence filing + SOC 2 package | February | 1 week |
| Executive summary distribution | March (post-remediation-close) | — |

The annual cycle anchors to Q1 so that a clean attestation package is available before the SOC 2 auditor fieldwork window (target: Q2–Q3 of each year).

---

## 3. Scope

### 3.1 In-scope surfaces

| Surface | Test mode | Rationale |
|---|---|---|
| **API layer** (Cloudflare Workers — all routes under `/api/*`) | Black-box (unauthenticated) + grey-box (valid JWT) | Primary attack surface; handles all health data |
| **Authentication flows** (OIDC magic link, MFA, session management) | Grey-box | Auth bypass is FORM's highest-consequence vulnerability class |
| **SAML 2.0 / OIDC SSO flows** (SP-initiated and IdP-initiated) | Grey-box with test IdP config | Enterprise blocker; certificate replay and assertion injection are known-pattern attacks |
| **SCIM v2 provisioning endpoints** | Grey-box | Provisioning injection, group-mapping bypass, unauthorized deprovisioning |
| **Admin dashboard** (`/admin/*`, all three persona tiers: `tenant_owner`, `tenant_admin`, `tenant_manager`) | Grey-box | Role escalation, IDOR, stored XSS |
| **RLS tenant isolation** (via crafted API requests — no direct Postgres port access) | Grey-box | Any cross-tenant row read is an automatic Critical; hardest control to test via code review alone |
| **iOS app** (release build, certificate pinning in effect) | Binary analysis + MiTM-attempt | OWASP MASVS L1 + L2; local storage secrets; certificate pinning bypass |
| **Android app** (release build) | Binary analysis + MiTM-attempt | Same as iOS |
| **Cloudflare edge configuration** (WAF, rate limits, CORS, CSP, TLS configuration) | Black-box | WAF bypass, header injection, rate-limit evasion, TLS downgrade |

### 3.2 Out-of-scope surfaces

| Surface | Reason |
|---|---|
| Supabase internal infrastructure (Postgres port, internal APIs) | Third-party; Supabase holds its own SOC 2 Type II; direct Postgres not internet-reachable |
| Anthropic API | Third-party sub-processor; FORM's threat model assumes Anthropic's perimeter is intact |
| ElevenLabs, PostHog, Sentry, Stripe | Third-party sub-processors; covered by their own security programs |
| Denial-of-service or availability testing | Destructive; Cloudflare mitigates L3/L4; would trigger SLA breach for active enterprise tenants |
| Social engineering of FORM personnel | Out of scope at current company size; revisit when headcount exceeds 10 |
| Physical access testing | Remote-first organisation; no owned physical infrastructure |
| Direct Postgres connection (`db.<project>.supabase.co`) | Only `form_api` role is exposed via the API; Postgres port is firewall-restricted to Cloudflare egress IPs |

### 3.3 Scope change process

If a new product surface launches before or during the test (e.g., browser extension, ClickHouse analytics cluster, webhook delivery endpoint), scope expansion requires a signed addendum before the testing firm accesses the new surface. Scope changes discovered mid-test that reveal an *adjacent* attack surface are immediately escalated to `security-engineer`, who decides within 4 hours whether to (a) extend scope by addendum, (b) document as an out-of-scope observation for the next cycle, or (c) treat as a responsible disclosure observation.

---

## 4. Methodology

FORM specifies the following frameworks as the minimum baseline. The testing firm may supplement with proprietary tooling; all supplemental coverage must be documented in the methodology appendix of the final report.

| Framework | Version | Application |
|---|---|---|
| OWASP WSTG (Web Security Testing Guide) | v4.2 | API and web surface — required checklist coverage for every in-scope web/API surface |
| OWASP ASVS (Application Security Verification Standard) | Level 2 | Authentication, session management, data protection, cryptography |
| OWASP MASVS (Mobile Application Security Verification Standard) | L1 + L2 | iOS and Android apps |
| PTES (Penetration Testing Execution Standard) | Current | Engagement structure, scoping, reporting, evidence preservation |
| CWE Top 25 | Current year | Weakness categories — all Top 25 must be checked; results documented per CWE ID |

**CVSS version:** CVSS v3.1 base score is the primary severity anchor. FORM applies context adjusters (§8.2, §8.3) on top of the base score.

**Automated vs manual balance:** Automated scanner output (Burp Suite, Semgrep, Trivy, Nuclei) is acceptable as a signal source but every finding submitted in the report must have manually-confirmed exploitability. Scanner noise submitted as findings without manual confirmation is grounds for rejection of the finding and a quality note to the firm.

---

## 5. Test Cadence

| Test type | Frequency | Trigger | Owner |
|---|---|---|---|
| **Annual external penetration test** | Once per year (Q1) | Compliance calendar (`docs/SOC2_READINESS.md §15.1`) | `security-engineer` (procurement) + external firm (execution) |
| **Re-test of prior Critical / High findings** | Within 30 days of remediation closure | After any Critical or High from the annual test is remediated | `security-engineer` + original testing firm |
| **Architecture-change triggered test** | Within 60 days of major change | SSO/SCIM launch, new database region, new AI provider, significant API restructure (>10 routes changed) | `security-engineer` + `enterprise-architect` scoping |
| **Pre-SOC 2 observation readiness test** | One-time | Before the observation period starts (PRE-21 in `docs/SOC2_READINESS.md §15.2`) | `security-engineer`; `compliance-officer` countersigns engagement letter |
| **Post-P0-incident targeted test** | Within 90 days of P0 closure | After any P0 security incident; scope is limited to the affected surface area | `security-engineer` + incident commander; scope defined in the P0 PIR |

**Architecture-change threshold:** The following changes automatically trigger a scoped test within 60 days:
- New SSO provider integration (SAML or OIDC)
- New database region (data residency expansion)
- New AI vendor or model type that receives health data
- New third-party API integration with write-access to `users`, `health_profiles`, `workouts`, or `meal_logs`
- Mobile app binary that introduces a new certificate or key management scheme

---

## 6. Testing Firm Selection

### 6.1 Minimum requirements

| Criterion | Requirement |
|---|---|
| **Certification** | CREST-accredited firm, OR the lead tester holds OSCP + one of: OSCE3, OSWE, GWAPT (GIAC), BSCP (PortSwigger) |
| **Independence** | No current or former employment with FORM; no financial relationship with any FORM sub-processor; no conflict with FORM's investors or advisors |
| **Health-data experience** | Demonstrated prior engagements testing systems that handle health or biometric data (HIPAA-adjacent environment preferred); reference available |
| **NDA** | Mutual NDA executed before sharing any test credentials, architecture diagrams, API specs, or environment access details |
| **Insurance** | Professional indemnity (PI) ≥ $2 million USD; cyber liability ≥ $1 million USD; FORM named as additional insured during engagement window |
| **Report format** | Structured report required: (a) executive summary, (b) per-finding table with ID, title, CVSS v3.1 score + vector string, description, evidence screenshots, step-by-step reproduction, remediation guidance, (c) methodology appendix, (d) OWASP WSTG / ASVS checklist coverage matrix |
| **Data handling** | Report and all work artifacts stored in an AES-256-encrypted repository; destroyed or returned within 30 days of final report delivery; never used for marketing or case studies without FORM written consent |

### 6.2 Evaluation shortlist (for first engagement)

The following firms meet the minimum bar and are shortlisted for RFP:

| Firm | Notes |
|---|---|
| **Bishop Fox** | Deep SAML/OIDC experience; Continuous Attack Surface Management (CAASM) option for Year 2 |
| **NCC Group** | SOC 2 + GDPR dual-expertise; EU-based testers available for EU data residency contexts |
| **Cure53** | Strong MASVS and browser/extension track record; boutique team, good for mobile-heavy scope |
| **Cobalt.io** | Continuous pentesting platform — evaluate for Year 2 once finding volume is understood |

The shortlist is reviewed annually. `security-engineer` may add firms; removal requires documented justification and `compliance-officer` approval.

### 6.3 RFP and selection process

| Step | Action | Owner | Timing |
|---|---|---|---|
| 1 | Issue RFP to shortlisted firms with scope document (§3), framework requirements (§4), and evidence requirements (§10) | `security-engineer` | November 1 |
| 2 | Receive proposals (pricing, tester CVs, sample reports, references) | `security-engineer` | November 15 |
| 3 | Reference check — one call per firm finalist with a client who ran a similar scope | `security-engineer` | November 22 |
| 4 | Firm selection decision memo filed to `compliance/pentest/YYYY/00-firm-selection-memo.md` | `security-engineer` + `compliance-officer` | November 30 |
| 5 | Statement of Work (SOW) + mutual NDA executed | `security-engineer` (counterpart); `compliance-officer` (FORM signatory) | December 15 |

---

## 7. Pre-Engagement Preparation

The following tasks must be completed before the test window opens.

| Task | Owner | Deadline |
|---|---|---|
| Create isolated test tenant (separate `tenant_id`, no production user data) | `platform-engineer` | 7 days before test start |
| Create test user accounts at all privilege tiers: `tenant_owner`, `tenant_admin`, `tenant_manager`, standard `user` | `security-engineer` | 7 days before test start |
| Provision test IdP configuration for SAML 2.0 (test Okta sandbox or equivalent) | `enterprise-architect` | 7 days before test start |
| Provide tester with: API documentation, Postman collection, architecture diagram, deployment topology | `security-engineer` | 5 days before test start |
| Confirm Cloudflare WAF is NOT set to block mode for tester's IP range (whitelist tester egress IPs) | `devops-lead` | 1 day before test start |
| Confirm audit log chain integrity cron is running (will produce HMAC verification artifact at close — §10) | `security-engineer` | 1 day before test start |
| Notify enterprise tenant admins: "We are conducting a security test this week; you may see unusual traffic patterns in audit logs" | `customer-success` | 1 day before test start |
| Alert `#security-alerts` Slack channel with test window dates to suppress false-positive paging | `security-engineer` | Morning of day 1 |

**No production user data in test environment.** The test tenant must not contain real health profiles, workout history, or meal logs. Synthetic data generation (realistic structure, no real PII) is acceptable.

---

## 8. Finding Severity Classification

### 8.1 Base classification (CVSS v3.1)

| Severity | CVSS v3.1 Score | Remediation SLA | Escalation path |
|---|---|---|---|
| **Critical** | 9.0 – 10.0 | **24 hours** from report delivery | Page Incident Commander immediately; treat as P1 per `docs/INCIDENT_RESPONSE.md §1.2`; notify `compliance-officer` |
| **High** | 7.0 – 8.9 | **7 calendar days** | `security-engineer` + `devops-lead` assigned; status update daily to `#security-alerts` |
| **Medium** | 4.0 – 6.9 | **30 calendar days** | Linear ticket; `security-engineer` triage within 48h; batched into sprint |
| **Low** | 0.1 – 3.9 | **90 calendar days** or next quarterly access review | Batched-triaged at quarterly review |
| **Informational** | N/A | Next quarterly review | Documented; no SLA; inform relevant code owner |

### 8.2 Health-data severity uplift

Any finding that could expose fields classified as `data_class = 'restricted'` in the `docs/AUDIT_LOG_SCHEMA.md` taxonomy — including but not limited to: `health_profiles.*`, `meal_logs.*`, `workouts.*`, `body_composition.*`, `ed_screening.*`, or any field containing GDPR Art. 9 special-category data — is automatically upgraded **one severity tier**:

| Finding base severity | Post-uplift severity |
|---|---|
| Low | Medium |
| Medium | High |
| High | Critical |
| Critical | Critical (no change; maximum already) |

**Rationale:** Health data carries heightened regulatory and ethical risk. A CVSS Medium finding that leaks a user's workout history or body composition data triggers GDPR Art. 33 notification obligations; treating it as High aligns the remediation timeline with that regulatory clock.

### 8.3 Tenant isolation uplift (absolute override)

Any finding that demonstrates or could plausibly enable one tenant's `form_api` session to read, write, or delete another tenant's rows — regardless of CVSS base score — is automatically classified **Critical**. This maps to Risk SR-02 in `docs/SOC2_READINESS.md §14` (highest residual risk in the register).

**Immediate action on isolation finding:** Even during the active testing window, a confirmed RLS bypass must be reported to `security-engineer` by phone/Slack within 1 hour of confirmation (do not wait for the full report). Incident Commander is paged within 30 minutes of `security-engineer` confirmation, per `docs/INCIDENT_RESPONSE.md §3.1`.

### 8.4 Accepted risk

Medium and Low findings may be accepted as residual risk when the estimated remediation cost exceeds the expected loss in a realistic threat model (FAIR methodology). Acceptance requires:
1. Written justification from the finding's risk owner
2. Corresponding entry in the risk register (`docs/SOC2_READINESS.md §14`) with accepting-risk-owner and review date
3. `compliance-officer` countersignature
4. Filed in `compliance/pentest/YYYY/open-accepted-risks.md`

Critical and High findings **cannot be accepted as residual risk** — they must be remediated within SLA.

---

## 9. Remediation Workflow

### 9.1 Step-by-step process

| Step | Action | Owner | Timing |
|---|---|---|---|
| **1 — Intake** | Read full report within 24h of delivery; create one Linear ticket per finding with labels `pentest-finding` + `severity-{critical\|high\|medium\|low\|informational}` + `pentest-YYYY` | `security-engineer` | Within 24h of report |
| **2 — Triage** | Assign ticket to responsible engineer; set due date per §8.1 SLA; link to affected code path or infra component; apply health-data and tenant-isolation uplifts (§8.2, §8.3) | `security-engineer` | Within 48h of report |
| **3 — Remediation** | Assigned engineer implements fix in a dedicated branch; PR description must include finding ID (e.g., `PT-2026-001-007`), CVSS score, and test-plan confirming the finding is no longer reproducible | Assigned engineer | Per SLA |
| **4 — Review** | PR reviewed by `security-engineer`; cannot self-review (if assignee = `security-engineer`, `compliance-officer` or `enterprise-architect` reviews instead) | `security-engineer` | Within 4h of PR opening for Critical; 24h for High |
| **5 — Re-test** | For Critical and High findings: provide exact reproduction steps to testing firm; receive written confirmation that the fix holds before closing the ticket | `security-engineer` + testing firm | Within 30 days of remediation close |
| **6 — Evidence filing** | Close Linear ticket with links to: merged PR, re-test confirmation (email or signed addendum from testing firm), deployment evidence (EAS build ID or Cloudflare deploy hash + timestamp) | `security-engineer` | At ticket close |
| **7 — SOC 2 package** | Compile all closed-ticket evidence into `compliance/pentest/YYYY/remediation-evidence.md`; write attestation sign-off (§10) | `compliance-officer` | Before end of remediation window |

### 9.2 Linear ticket conventions

```
Title: [PT-YYYY-NNN] {SEVERITY} — {Brief finding description}
Labels: pentest-finding, severity-critical, pentest-2026
Priority: Urgent (Critical/High) | Medium (Medium) | Low (Low/Informational)
Body (required fields):
  - Finding ID: PT-YYYY-NNN
  - CVSS v3.1 Score + Vector String
  - Affected surface: (e.g., "SCIM /Users endpoint")
  - Reproduction steps: [link to finding in full report — internal only]
  - Remediation guidance from testing firm: [brief]
  - Target remediation date: YYYY-MM-DD
  - Re-test required: Yes (Critical/High) | No (Medium/Low)
```

Linear query for all open pentest findings: `label:pentest-finding AND NOT state:done`

This query is reviewed at every quarterly access review and monthly during active remediation windows.

### 9.3 Emergency patch procedure (Critical findings)

If a Critical finding is discovered and `security-engineer` determines it is exploitable in the production environment during the test window:

1. `security-engineer` immediately suspends the relevant surface (feature flag off, route disabled, or WAF block rule activated) — within 1 hour of confirmation.
2. Incident Commander declares P1, follows `docs/INCIDENT_RESPONSE.md §3`.
3. Emergency patch bypasses sprint process; PR is reviewed and deployed same day.
4. Testing firm confirms fix; test window continues on other surfaces.
5. Complete incident timeline documented in PIR (post-incident review).

---

## 10. SOC 2 Evidence Package

All artifacts are filed to `compliance/pentest/YYYY/` in the private `form-compliance` repository immediately after the remediation window closes. Access is restricted to `security-engineer`, `compliance-officer`, and the founder.

| Artifact | Filename | Description | Auditor required |
|---|---|---|---|
| Firm selection memo | `00-firm-selection-memo.md` | Explains firm chosen, criteria met, reference-check summary | CC9.2 |
| Statement of Work + NDA | `01-engagement-letter.pdf` | Signed SOW and mutual NDA with testing firm | CC7.1, CC9.2 |
| Full penetration test report | `02-report-full.pdf` | Complete findings report from firm (not shared externally) | CC7.1 |
| Executive summary (redacted) | `03-report-exec-summary.pdf` | Customer-shareable version — reproduction steps removed | CC7.1 |
| Remediation evidence | `04-remediation-evidence.md` | Linear ticket IDs, PR links, re-test confirmations for all Critical + High findings | CC7.2 |
| Attestation sign-off | `05-attestation-sign-off.md` | `security-engineer` written statement that all Critical + High findings are remediated; signed and dated | CC7.1 |
| Accepted risks (if any) | `06-open-accepted-risks.md` | Medium / Low findings accepted as residual risk with risk-owner sign-off and risk register cross-reference | CC3.2 |
| HMAC chain verification | `07-hmac-chain-verification.txt` | Output of DEC-030 HMAC chain integrity cron run on the date the test window closes | CC7.1 |

**HMAC chain verification rationale:** This artifact confirms that if a tester had discovered a log-tampering vector during the engagement window, FORM's automated HMAC cron would have detected the chain break. A clean verification output on the day the test closes means the audit log is intact evidence for the auditor — no tampering occurred while the testing firm had access.

**Report storage policy:** The full report (`02-report-full.pdf`) contains architectural details and working exploit techniques. It is:
- Stored only in the private `form-compliance` repo with branch protection
- Never committed to the public `rbit27/form` repo
- Never used as LLM training data or shared with AI vendors
- Never shared unredacted with enterprise customers — only the executive summary (§11)

---

## 11. Enterprise Customer Disclosure

### 11.1 Executive summary (ACV ≥ $50k)

Enterprise customers with a signed MSA and ACV ≥ $50,000 may request the executive summary (`03-report-exec-summary.pdf`) under the following conditions:

| Condition | Detail |
|---|---|
| Request channel | `security@form.coach` or dedicated CSM Slack Connect channel |
| Required: mutual NDA | Standalone mutual NDA executed, or NDA clause within existing MSA confirmed active |
| Authorisation | `compliance-officer` must approve each disclosure (not delegatable) |
| What is shared | Finding counts by severity tier, SLA compliance confirmation, methodology summary, attestation letter stating all Critical/High findings remediated |
| What is never shared | Step-by-step reproduction steps, working exploit code, specific vulnerable parameter names, internal architecture diagrams used during the test |
| Timing | Earliest release: after all Critical and High findings are remediated and re-tested (§9) |

Template: see §15.3 (Executive Summary Cover Memo).

### 11.2 Attestation letter (ACV < $50k)

Standard-tier enterprise customers (ACV < $50k) receive an attestation letter rather than the executive summary:

> "FORM completed an annual penetration test in [Month YYYY] conducted by [Firm name], a CREST-accredited security testing firm. The test covered FORM's API, authentication flows, SSO/SCIM, admin dashboard, mobile applications, and edge configuration. All Critical and High severity findings from the test were remediated within contractual SLAs. A full executive summary is available to qualified enterprise customers under mutual NDA."

`compliance-officer` countersigns all attestation letters. Template: §15.2.

### 11.3 Enterprise buyer timeline coordination

Enterprise customers may request that FORM schedule its annual penetration test to align with their internal vendor security review cycle. FORM accommodates this subject to:
- ≥60 days advance written notice to `security@form.coach`
- Availability of the selected testing firm in the requested window
- FORM's right to decline if the requested window conflicts with a major product launch or enterprise customer pilot onboarding

Requested schedule changes are logged as `admin.pentest_window_adjusted` DEC-030 events (§12).

---

## 12. DEC-030 Audit Events

All penetration test lifecycle events emit HMAC-chained audit log entries per `docs/AUDIT_LOG_SCHEMA.md`. These events are filed as `actor_type = 'system'` or `'support'` depending on the action.

| Event | Trigger | Severity tier | Retention | DEC-030 fields |
|---|---|---|---|---|
| `security.pentest_engagement_signed` | SOW + NDA executed with testing firm | STANDARD | 7 years | `actor_id`: compliance-officer UUID; `metadata`: `{firm, sow_ref, test_window_start, test_window_end}` |
| `security.pentest_window_opened` | First day of active testing | STANDARD | 7 years | `metadata`: `{firm, scope_surfaces[], tester_ips[]}` |
| `security.pentest_finding_intake` | Linear ticket created for each finding | STANDARD | 7 years | `metadata`: `{finding_id, severity, cvss_score, surface, linear_ticket_id}` |
| `security.pentest_finding_remediated` | Linear ticket closed with evidence | STANDARD | 7 years | `metadata`: `{finding_id, severity, linear_ticket_id, pr_url, deploy_ref, retest_required}` |
| `security.pentest_retest_confirmed` | Re-test confirmation received from firm | HIGH | 7 years | `metadata`: `{finding_id, severity, firm_confirmation_ref}` |
| `security.pentest_finding_accepted` | Medium/Low finding accepted as residual risk | HIGH | 7 years | `metadata`: `{finding_id, severity, risk_owner, risk_register_entry, review_date}` |
| `security.pentest_window_closed` | Final day of active testing | STANDARD | 7 years | `metadata`: `{firm, total_findings_by_severity, all_critical_high_remediated: bool}` |
| `security.pentest_evidence_filed` | SOC 2 evidence package filed to compliance repo | HIGH | 7 years | `metadata`: `{year, artifacts_filed[], hmac_chain_verified: bool, compliance_officer_id}` |
| `security.pentest_summary_disclosed` | Executive summary shared with enterprise customer | HIGH | 7 years | `metadata`: `{tenant_id, acv_tier, recipient_email_hash, nda_ref, compliance_officer_approval_ref}` |
| `admin.pentest_window_adjusted` | Test window rescheduled at enterprise customer request | STANDARD | 3 years | `metadata`: `{requesting_tenant_id, original_window, adjusted_window, reason}` |

All events carry `hmac_prev` and `hmac_self` fields per the HMAC chain specification in `docs/AUDIT_LOG_SCHEMA.md §2`. The `security.pentest_evidence_filed` event's `hmac_self` value is recorded in `compliance/pentest/YYYY/07-hmac-chain-verification.txt` as the closing chain seal.

---

## 13. Roles and Responsibilities

| Role | Responsibilities |
|---|---|
| **`security-engineer`** | Primary owner; firm selection; scope definition; pre-engagement setup; finding intake and triage; remediation oversight; re-test coordination; evidence filing |
| **`compliance-officer`** | SOW countersignature; attestation letter countersignature; executive summary disclosure approval; risk acceptance countersignature; SOC 2 package completeness sign-off |
| **`devops-lead`** | WAF whitelist for tester IPs; deployment evidence (Cloudflare deploy hashes); infrastructure change freeze during test window |
| **`enterprise-architect`** | SSO/SCIM test configuration; RLS verification support; post-finding architecture review for isolation-class findings |
| **`platform-engineer`** | Test environment provisioning; synthetic data generation; Mobile app build provision |
| **`customer-success`** | Enterprise tenant notification before test window opens; executive summary distribution coordination |
| **founder** | Final sign-off on any risk acceptance for Critical/High findings (if `compliance-officer` escalates) |

---

## 14. Prohibited Activities

The following are explicitly prohibited during any FORM-commissioned penetration test engagement:

| Prohibited action | Consequence |
|---|---|
| Access or download production user personal data or health data beyond minimum-necessary proof-of-concept | Automatic scope suspension; engagement terminated; incident declared |
| Share any personal data with third parties | Immediate engagement termination; NDA breach proceedings |
| Perform denial-of-service or disrupt production services | Engagement terminated; legal action may follow |
| Test against production user accounts (outside the provisioned test tenant) | Engagement terminated |
| Modify or delete production data | Engagement terminated; data recovery initiated |
| Conduct cross-tenant attacks against real enterprise tenant data (use the test tenant) | Engagement terminated; P0 declared |
| Publish findings publicly before coordinated close of remediation window | NDA breach; legal action |
| Use FORM architecture details or findings for unrelated client work | NDA breach |

These prohibitions are contractually binding in the SOW and NDA. Violation terminates the safe harbour and triggers legal escalation per FORM's legal team.

---

## 15. Templates

### 15.1 Engagement scope letter (to testing firm)

```
FORM · Penetration Test Scope Letter · [YYYY]

This scope letter accompanies the Statement of Work signed [DATE] between
FORM (rbit27/form) and [FIRM NAME] and governs the [YEAR] annual penetration test.

Test window: [START DATE] – [END DATE]
Primary contact (FORM): security-engineer <security@form.coach>
Primary contact (Firm): [NAME, EMAIL]

INCLUDED SURFACES (see docs/PENETRATION_TESTING.md §3.1):
  [list all in-scope surfaces]

EXCLUDED SURFACES (see §3.2):
  [list all out-of-scope surfaces]

MANDATORY FRAMEWORKS:
  OWASP WSTG v4.2 | OWASP ASVS Level 2 | OWASP MASVS L1+L2 | CWE Top 25 | PTES

TEST ENVIRONMENT:
  Base URL: https://[staging domain]
  Test tenant ID: [UUID]
  Tester credentials: [shared via 1Password shared vault — link]
  Tester egress IPs to whitelist: [list]

SPECIAL HANDLING REQUIREMENTS:
  - Any finding that may bypass Row-Level Security (tenant isolation) must be
    reported to security@form.coach by phone/Slack within 1 hour of confirmation.
    Do not wait for the final report.
  - Health-data fields (health_profiles, meal_logs, workouts, body_composition,
    ed_screening) carry GDPR Art. 9 special-category status. Accessing more
    data than necessary for PoC is prohibited.

REPORT DELIVERY:
  Format: PDF (executive summary + full findings table + methodology appendix
          + OWASP WSTG/ASVS checklist coverage matrix)
  Delivery method: Encrypted email or secure file-share to security@form.coach
  Deadline: Within 7 calendar days of test close

Signed: _________________________ (FORM, compliance-officer) [DATE]
```

### 15.2 Attestation letter (standard tier, ACV < $50k)

```
FORM Security Attestation — Annual Penetration Test · [YEAR]

Date: [DATE]
Issued to: [CUSTOMER NAME]
Issued by: compliance-officer, FORM

FORM completed an annual penetration test in [MONTH YEAR], conducted by
[FIRM NAME], a CREST-accredited security testing firm.

Scope: FORM's production API, authentication flows, SSO/SCIM provisioning
endpoints, admin dashboard, iOS and Android mobile applications, and
Cloudflare edge configuration.

Framework: OWASP WSTG v4.2, OWASP ASVS Level 2, OWASP MASVS L1/L2,
CWE Top 25.

Result: All Critical and High severity findings identified during the test
were remediated within FORM's published SLAs (Critical: 24 hours; High:
7 calendar days). Re-testing by the external firm confirmed all Critical and
High remediations hold.

A full executive summary is available to enterprise customers (ACV ≥ $50k)
under a signed mutual NDA. Contact security@form.coach to request access.

Signed: _________________________ (compliance-officer, FORM) [DATE]
```

### 15.3 Executive summary cover memo (ACV ≥ $50k)

```
FORM Executive Pentest Summary Cover Memo · [YEAR]

Date: [DATE]
Issued to: [CUSTOMER NAME] — [SECURITY CONTACT NAME, EMAIL]
Issued by: compliance-officer, FORM
NDA reference: [MSA section or standalone NDA date]

Enclosed: [YEAR] Annual Penetration Test Executive Summary
          (docs/PENETRATION_TESTING.md §11.1 conditions apply)

SUMMARY OF FINDINGS:

  Critical:        [N] found · [N] remediated · 0 open
  High:            [N] found · [N] remediated · 0 open
  Medium:          [N] found · [N] remediated · [N] accepted
  Low:             [N] found · [N] remediated · [N] accepted
  Informational:   [N] noted

All Critical and High findings were remediated within SLA and confirmed
by re-test. The enclosed executive summary contains finding counts,
severity distribution, and a methodology summary. Reproduction steps
and technical exploitation details are not included in the shared version.

HANDLING REQUIREMENTS (per NDA):
  - Do not share with parties outside your security team without FORM
    written consent.
  - Do not upload to AI systems, document stores with third-party access,
    or open-source bug trackers.
  - Return or certifiably destroy all copies within 12 months or upon
    contract termination, whichever is earlier.

For questions: security@form.coach | Your dedicated CSM.

Signed: _________________________ (compliance-officer, FORM) [DATE]
```

---

## 16. SOC 2 Control Mapping

| SOC 2 Criterion | How this program satisfies it | Evidence artifact |
|---|---|---|
| **CC7.1** — Entity selects, develops, and performs ongoing evaluations to ascertain whether components of internal control are present and functioning | Annual external penetration test by independent CREST firm; scope covers all in-production surfaces; findings triaged and remediated per §9 | `02-report-full.pdf` + `03-report-exec-summary.pdf` + `05-attestation-sign-off.md` |
| **CC7.2** — Entity evaluates and communicates internal control deficiencies in a timely manner | Per-finding Linear tickets with SLA deadlines; re-test confirmations for Critical/High; `compliance-officer` sign-off before evidence filing | `04-remediation-evidence.md` + `05-attestation-sign-off.md` + Linear audit trail |
| **CC4.1** — Entity selects, develops, and performs separate evaluations to ascertain whether components of internal control are present and functioning | Annual external pentest is the primary "separate evaluation" mechanism for operational controls not covered by §23 quarterly access review | `01-engagement-letter.pdf` (independence of testing firm) |
| **CC9.2** — Entity assesses and monitors risks associated with third parties who have access to systems | Testing firm is a third-party with temporary authenticated access to test environment; firm selection criteria (§6.1) and SOW (§10) govern this relationship | `00-firm-selection-memo.md` + `01-engagement-letter.pdf` |
| **CC3.2** — Entity identifies and assesses risks to achievement of its objectives | Post-test risk register update: all Medium/Low accepted risks are entered in `docs/SOC2_READINESS.md §14` | `06-open-accepted-risks.md` cross-referenced to §14 risk register |
| **P5.2** — Privacy: personal information is protected from unauthorized access | Pentest scope includes authentication bypass and health-data exposure scenarios; health-data severity uplift (§8.2) ensures these findings receive top-tier remediation priority | §8.2 + `04-remediation-evidence.md` |

---

## 17. Open Items and Gap Status

| Item | Status | Owner | Target |
|---|---|---|---|
| PRE-21: First penetration test executed (pre-SOC 2 observation period) | 🟡 Partial — program defined; firm selection + execution pending | `security-engineer` | Month O-3 (3 months before observation start) |
| Testing firm SOW + NDA executed for Year 1 | 🔴 Open | `security-engineer` | December [YEAR-1] |
| Test environment (isolated test tenant) provisioned | 🔴 Open | `platform-engineer` | January [YEAR] |
| DEC-030 events implemented in `audit_log` (§12) | 🟡 Partial — schema defined; implementation pending | `platform-engineer` | Month O-3 |
| `compliance/pentest/YYYY/` directory structure created in private repo | 🔴 Open | `compliance-officer` | January [YEAR] |

**Gap tracking in SOC2_READINESS.md:** `docs/SOC2_READINESS.md §16.11` tracks CC7.1 and CC7.2 gap status and references this document for operational detail. Status is updated in §16.11 after each annual test cycle closes.

---

## 18. Version History

| Version | Date | Changes | Author |
|---|---|---|---|
| v1.0 | 2026-06-12 | Initial standalone document. Extracted from `docs/SOC2_READINESS.md §16`; expanded with RFP process, pre-engagement preparation, DEC-030 event registry, three disclosure templates, full SOC 2 control mapping. | `security-engineer` + `compliance-officer` |

---

*Reviewed annually in February following close of the prior year's penetration test cycle. Any change to in-scope surfaces, remediation SLAs, or customer disclosure tiers requires `compliance-officer` countersignature and a version bump.*

**security-engineer + compliance-officer + enterprise-architect**
