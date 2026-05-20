# CC4 Evidence Collection Plan

**File reference:** CC4-E-002
**SOC 2 criterion:** CC4.1 — Ongoing and Separate Evaluations
**Owner:** compliance-officer
**Status:** Active — collection begins at launch
**First evidence checkpoint:** T+30 days post-launch
**Observation period end (Type II):** Launch + 6 months

This document converts the CC4.1 design intent into an executable evidence collection schedule. It is the remediation artifact for CC4-GAP-002. Auditors will use this plan to verify that ongoing evaluation controls operated continuously across the SOC 2 observation window.

---

## 1. Ongoing Evaluation Controls

Ongoing evaluations are built-in monitoring activities that run automatically or on a fixed recurring schedule without requiring a discrete annual decision to perform them.

### 1.1 Monthly SOC 2 Dashboard Review

| Field | Value |
|---|---|
| **Control reference** | docs/SOC2_READINESS.md §15 — Compliance Calendar |
| **SOC 2 criterion** | CC4.1, CC4.2 |
| **Evidence type** | Structured Markdown memo — signed by founder via git commit |
| **Collection frequency** | Monthly, last business day of each calendar month |
| **Storage path** | `compliance/evidence/soc2-monthly-YYYY-MM.md` |
| **Responsible owner** | compliance-officer (founder) |
| **Start date** | Launch month (Month 0) |
| **Retention** | 7 years per §15.1 evidence retention schedule |

**What is captured:** Deficiencies opened this month, deficiencies closed this month, outstanding P0/P1 count, sub-processor changes, access review due dates, next review date. Template is defined in `compliance/cc4/deficiency-communication-procedure.md` §2 (Tier 1 founder self-review).

**Auditor traceability:** Each monthly memo git commit SHA is logged in the HMAC audit log as `event_type: compliance.monthly_review` so auditors can verify no months were skipped across the observation window.

---

### 1.2 Daily HMAC Chain Integrity Check

| Field | Value |
|---|---|
| **Control reference** | DEC-030; docs/AUDIT_LOG_SCHEMA.md; docs/OBSERVABILITY.md §16.3 probe S-007 |
| **SOC 2 criterion** | CC4.1, CC7.2 |
| **Evidence type** | Automated probe pass/fail event log — Cloudflare Worker Scheduled Task reporting to Better Stack via Heartbeat API |
| **Collection frequency** | Every 10 minutes (probe S-007); daily roll-up log retained for compliance purposes |
| **Storage path** | `compliance/evidence/chain-check-YYYY-MM/` (monthly directory, one JSON file per day: `YYYY-MM-DD.json`) |
| **Responsible owner** | security-engineer (automated); compliance-officer reviews monthly roll-up |
| **Start date** | Launch (probe S-007 deployed at M3 per §15.2) |
| **Retention** | 7 years |

**What is captured:** Probe S-007 calls `GET /internal/audit/health` and asserts `chain_valid: true` and last entry age ≤ 10 minutes. Each check result (timestamp, region, pass/fail, chain hash snapshot) is written to Better Stack and mirrored to the monthly directory. Any failure triggers a P0 PagerDuty alert to security-engineer within 5 minutes per docs/OBSERVABILITY.md §16.5.

**Auditor traceability:** An unbroken daily run of `chain_valid: true` results across the 6-month observation window is the primary evidence for DEC-030 operating effectiveness. A single gap or chain break is escalated immediately and documented in `control-deficiency-log.csv`.

---

### 1.3 Quarterly Access Review

| Field | Value |
|---|---|
| **Control reference** | docs/SOC2_READINESS.md §23 |
| **SOC 2 criterion** | CC4.1, CC6.2, CC6.3 |
| **Evidence type** | Structured Markdown artifact — §23.6 template, signed by reviewer via git commit |
| **Collection frequency** | Quarterly (Q1: January, Q2: April, Q3: July, Q4: October) |
| **Storage path** | `compliance/evidence/access-review-YYYY-QN.md` |
| **Responsible owner** | security-engineer + compliance-officer |
| **Start date** | PRE-23 (Month O-1 per §15.2 pre-observation checklist); first execution before observation period start |
| **Retention** | 7 years |

**What is captured:** Full §23.6 artifact — inventory snapshot of all credentials across GitHub, Supabase, Cloudflare, 1Password, and PostHog; authorization comparison against §23.5 authorized roster; deprovisioning actions taken; CC6 control effectiveness assessment (Step 5); reviewer attestation. See docs/SOC2_READINESS.md §23.4 for the five-step procedure.

**Auditor traceability:** Four artifacts per 12-month observation window demonstrates continuous quarterly cadence. Each artifact's SHA-256 hash is written into the HMAC audit log as `event_type: compliance.access_review`.

---

### 1.4 Supabase Weekly Health Export

| Field | Value |
|---|---|
| **Control reference** | docs/OBSERVABILITY.md §6 (database health metrics) |
| **SOC 2 criterion** | CC4.1, A1.1 (Availability) |
| **Evidence type** | JSON export — Supabase database health metrics snapshot |
| **Collection frequency** | Weekly, every Monday 09:00 UTC |
| **Storage path** | `compliance/evidence/supabase-YYYY-MM-DD.json` (date = Monday of the week) |
| **Responsible owner** | devops-lead (automated export via Supabase API + GitHub Action) |
| **Start date** | Launch |
| **Retention** | 7 years |

**What is captured:** Database connection count, replication lag, WAL size, active row-level security policy version, backup status (PITR last success timestamp), storage volume utilization. Provides evidence that the Supabase-hosted primary database remained healthy and continuously backed up across the observation window.

**Auditor traceability:** 26 weekly exports per 6-month window. Gaps indicate a monitoring failure and must be documented in `control-deficiency-log.csv` within 24 hours.

---

### 1.5 Cloudflare Monthly Analytics Snapshot

| Field | Value |
|---|---|
| **Control reference** | docs/OBSERVABILITY.md §16 (CDN and WAF monitoring) |
| **SOC 2 criterion** | CC4.1, CC7.2, A1.2 |
| **Evidence type** | CSV export — Cloudflare Analytics API output |
| **Collection frequency** | Monthly, first business day of the following month |
| **Storage path** | `compliance/evidence/cloudflare-YYYY-MM.csv` (YYYY-MM = month being reported) |
| **Responsible owner** | devops-lead (automated export via Cloudflare Analytics API + GitHub Action) |
| **Start date** | Launch |
| **Retention** | 7 years |

**What is captured:** Total requests, WAF rule triggers by rule ID, bot score distribution, DDoS mitigation events, 4xx/5xx error rate by endpoint, bandwidth. Provides monthly evidence of WAF operating effectiveness and availability status. Anomalous WAF spikes are cross-referenced against `control-deficiency-log.csv` entries.

**Auditor traceability:** 6 monthly exports per 6-month observation window. Each CSV is SHA-256 hashed and the hash logged in the HMAC audit log as `event_type: compliance.cloudflare_export`.

---

## 2. Separate Evaluation Controls

Separate evaluations are periodic, discrete activities — each produces a bounded evidence package rather than a continuous stream of records.

### 2.1 Annual External Penetration Test

| Field | Value |
|---|---|
| **Control reference** | docs/SOC2_READINESS.md §16 |
| **SOC 2 criterion** | CC4.1 (separate evaluation), CC7.1, CC7.2 |
| **Evidence type** | Full penetration test report + remediation evidence package |
| **Collection frequency** | Annual (Q1, January — per §15.1 compliance calendar) |
| **Storage path** | `compliance/evidence/pentest-YYYY/` (contains: executive-summary.pdf, remediation-evidence.md, hmac-chain-verification.txt, scope-statement.pdf, retest-confirmation.pdf) |
| **Responsible owner** | security-engineer (engagement management); compliance-officer (SOC 2 package assembly) |
| **Start date** | First engagement: January of the first full calendar year post-launch (Year 1 Q1) |
| **Retention** | 7 years |

**What is captured:** Five mandatory artifacts defined in docs/SOC2_READINESS.md §16.6: executive summary with finding counts by severity, remediation evidence closing all critical and high findings, HMAC chain verification confirming audit log integrity across the test window, signed scope statement, and retest confirmation for any critical findings. Full report available to enterprise customers under NDA for contracts >$50k ACV.

**Note:** First pentest engagement budget: $15,000–$30,000. Shortlist: Bishop Fox, NCC Group, Cobalt.io, Cure53. Provider selection to be documented and filed to `compliance/evidence/pentest-YYYY/provider-selection-rationale.md`.

---

### 2.2 Quarterly Access Review (as Separate Evaluation)

The quarterly access review (§1.3 above) also constitutes a separate evaluation under CC4.1 because each execution is a discrete, bounded review performed by a named reviewer who attests to completeness. It is listed here to reflect both its ongoing and separate evaluation character under the AICPA framework.

| Field | Value |
|---|---|
| **Storage path** | Same as §1.3: `compliance/evidence/access-review-YYYY-QN.md` |
| **Separate evaluation scope** | CC6 control effectiveness assessment (§23.4 Step 5) is a distinct evaluative judgment, not a continuous monitoring output |
| **Frequency** | 4 per year (Q1–Q4) |

---

### 2.3 Year-over-Year SOC 2 Readiness Review

| Field | Value |
|---|---|
| **Control reference** | docs/SOC2_READINESS.md §15.1 (Q4 annual compliance review) |
| **SOC 2 criterion** | CC4.1 (separate evaluation across all criteria), CC4.2 |
| **Evidence type** | Annual review memo — structured assessment of all TSC criteria against current control state |
| **Collection frequency** | Annual (Q4, October–November, aligned with audit preparation) |
| **Storage path** | `compliance/evidence/soc2-annual-YYYY.md` |
| **Responsible owner** | compliance-officer + security-engineer |
| **Start date** | Year 1 Q4 (first annual review covers the full first observation period) |
| **Retention** | 7 years |

**What is captured:** Year-over-year delta across all five TSC (CC, A, PI, C, P), control effectiveness assessment for every control mapped in docs/SOC2_READINESS.md, updated deficiency log review, gap closure rate, and updated SOC 2 readiness percentage. This document is the primary input to the annual audit firm readiness assessment meeting.

---

## 3. Evidence Collection Timeline

| Month post-launch | Collection event |
|---|---|
| T+0 (launch) | Chain integrity probe S-007 begins; Supabase weekly exports begin; Cloudflare monthly export begins |
| T+30 days | **First evidence checkpoint** — verify all automated collection pipelines are running; file first monthly SOC2 dashboard memo (`soc2-monthly-YYYY-MM.md`); confirm chain-check directory has 30 days of passing results |
| T+90 days (Q1 post-launch) | First quarterly access review (`access-review-YYYY-Q1.md` or Q2 depending on launch month) |
| T+180 days | SOC 2 Type II observation period minimum satisfied — evidence package complete; fieldwork ready |
| T+270 days | Second quarterly access review; Q3 monthly memos and cloudflare exports reviewed |
| T+365 days | First annual review (`soc2-annual-YYYY.md`); first annual penetration test (if launch was Q1) |

---

## 4. Evidence Integrity Controls

All evidence artifacts are:

1. Stored in the private `form-compliance` git repository with branch protection (`required_reviewers: 1`; no force-push).
2. SHA-256 hashed on commit; hash written to the HMAC audit log as an event so the chain of custody is tamper-evident.
3. Named with the date or period they cover — not the date they were filed — so auditors can verify coverage without ambiguity.
4. Never retroactively modified. If an artifact contains an error, a corrected version is filed as `YYYY-MM-v2.md` with a note explaining the correction; the original is retained.

---

## 5. Gap Status

This document remediates **CC4-GAP-002** (docs/SOC2_READINESS.md §32.3). The gap — that monitoring evidence is not yet in steady state because the product is pre-launch — resolves at **launch + 6 months** when the first complete observation-period evidence package exists. This plan converts the design intent into an executable schedule so auditors can verify, at the time of fieldwork, that collection was planned before the observation period began.

**Auditor exhibit:** CC4-E-002

---

*Owner: compliance-officer. Effective: 2026-05-20. Next review: first evidence checkpoint T+30 days post-launch, then annually.*
