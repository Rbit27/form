# RB-ENT-WIN-LOSS-01 · Enterprise Win/Loss Analytics — Quarterly Review & OQ-PIPE-01 Calibration

> **Runbook ID:** RB-ENT-WIN-LOSS-01
> **Type:** Recurring analytics · quarterly cadence
> **Owner:** data-engineer · reviewed by: enterprise-architect · customer-success · compliance-officer
> **Last reviewed:** 2026-06-15
> **SOC 2 TSC:** CC5.2, CC1.4, CC4.1, CC9.2
> **Cross-refs:**
> - `docs/COST_MODEL.md §39` — authoritative source for all SQL queries and event schemas in this runbook
> - `docs/COST_MODEL.md §37.3` — stage conversion rate benchmark table (OQ-PIPE-01 target)
> - `docs/COST_MODEL.md §37.6` — ARR bridge; `enterprise.arr_bridge_closed` monthly event
> - `docs/AUDIT_LOG_SCHEMA.md §Enterprise + §Privacy` — four DEC-030 events used here
> - `docs/DATA_MODEL.md §39.5` — `enterprise_deal_outcomes` DDL (RLS: `form_api` REVOKED)
> - `docs/SOC2_READINESS.md` — WIN-E-001 (CC5.2/CC1.4), WIN-E-002 (CC5.2/CC4.1), WIN-E-003 (CC1.4/CC9.2)
> - `docs/DECISION_LOG.md` — DEC-06X (OQ-PIPE-01 closure entry, created at Deal 10)

---

## 1. Purpose and Scope

This runbook governs two recurring activities:

| Activity | Cadence | Owner | Section |
|---|---|---|---|
| **Quarterly win/loss review** | After each `enterprise.arr_bridge_closed` DEC-030 event (monthly) — full narrative review quarterly, before investor updates | data-engineer + founder | §4 |
| **OQ-PIPE-01 calibration** | When `COUNT(*) ≥ 10` terminal outcomes; estimated M18 (Deal 10) | founder + data-engineer | §5 |

**Out of scope:** Deal-close mechanics (Admin Console "Close Deal" modal — `docs/COST_MODEL.md §39.10` P0 checklist); ARR bridge event emission (`docs/COST_MODEL.md §37.6`); partner attribution reporting (`docs/COST_MODEL.md §38`).

**Privacy floor (inherited from `docs/COST_MODEL.md §39`):** No prospect company names, contact email, or individual employee `user_id` in any query result, DEC-030 event, or evidence file. `competitor_category` is a structured enum only — verbatim loss-call notes stay in CRM. `deal_id` is a FORM-internal UUID never shared externally.

---

## 2. Pre-Flight Checklist

Before running any query:

- [ ] Confirm you are connecting as `compliance_reviewer` or `form_system` role (read-only; `form_api` is REVOKED from `enterprise_deal_outcomes`)
- [ ] Confirm Supabase project is the **production** project (not staging)
- [ ] Verify the `enterprise_deal_outcomes` table exists: `SELECT 1 FROM enterprise_deal_outcomes LIMIT 1;`
- [ ] Note the current terminal outcome count before starting:

```sql
SELECT
  COUNT(*) FILTER (WHERE outcome = 'closed_won')                              AS won,
  COUNT(*) FILTER (WHERE outcome = 'closed_lost' AND NOT no_go_criteria_triggered) AS lost_excl_nogo,
  COUNT(*) FILTER (WHERE no_go_criteria_triggered)                            AS no_go_rejections,
  COUNT(*)                                                                    AS total
FROM enterprise_deal_outcomes;
```

- [ ] If `lost_excl_nogo + won < 10`, **do not run §39.6.1** — the conversion rate is not statistically meaningful. Run §39.6.2–39.6.5 for directional signals only; skip OQ-PIPE-01 calibration (§5)

---

## 3. Analytics Queries

All five queries are the canonical versions from `docs/COST_MODEL.md §39.6`. Run as `compliance_reviewer` role.

### 3.1 Stage conversion rate actuals — §39.6.1

**Gate:** Run only when terminal outcome count ≥ 10 (see §2 pre-flight). This is the OQ-PIPE-01 resolution query.

```sql
WITH stage_entry AS (
  SELECT
    eps.id                           AS deal_id,
    CASE WHEN edo.outcome = 'closed_won' THEN 1 ELSE 0 END AS closed_won,
    eps.stage_s0_entered_at IS NOT NULL AS entered_s0,
    eps.stage_s1_entered_at IS NOT NULL AS entered_s1,
    eps.stage_s2_entered_at IS NOT NULL AS entered_s2,
    eps.stage_s3_entered_at IS NOT NULL AS entered_s3,
    eps.stage_s4_entered_at IS NOT NULL AS entered_s4,
    eps.stage_s5_entered_at IS NOT NULL AS entered_s5
  FROM enterprise_pipeline_stages eps
  JOIN enterprise_deal_outcomes edo ON edo.deal_id = eps.id
  WHERE eps.stage IN ('S5', 'closed_lost')
    AND NOT edo.no_go_criteria_triggered
),
totals AS (
  SELECT
    COUNT(*) FILTER (WHERE entered_s0)   AS n_s0,
    COUNT(*) FILTER (WHERE entered_s1)   AS n_s1,
    COUNT(*) FILTER (WHERE entered_s2)   AS n_s2,
    COUNT(*) FILTER (WHERE entered_s3)   AS n_s3,
    COUNT(*) FILTER (WHERE entered_s4)   AS n_s4,
    COUNT(*) FILTER (WHERE closed_won=1) AS n_won
  FROM stage_entry
)
SELECT
  n_s0                                                AS deals_entered_pipeline,
  ROUND(n_s1::NUMERIC/NULLIF(n_s0,0)*100,1)          AS s0_to_s1_pct,
  ROUND(n_s2::NUMERIC/NULLIF(n_s1,0)*100,1)          AS s1_to_s2_pct,
  ROUND(n_s3::NUMERIC/NULLIF(n_s2,0)*100,1)          AS s2_to_s3_pct,
  ROUND(n_s4::NUMERIC/NULLIF(n_s3,0)*100,1)          AS s3_to_s4_pct,
  ROUND(n_won::NUMERIC/NULLIF(n_s4,0)*100,1)         AS s4_to_s5_pct,
  ROUND(n_won::NUMERIC/NULLIF(n_s0,0)*100,1)         AS overall_win_rate_pct
FROM totals;
```

**Interpretation:** Compare each `*_pct` column to the corresponding row in `docs/COST_MODEL.md §37.3`. A deviation > 10 pp in any stage is material and triggers the OQ-PIPE-01 protocol (§5).

### 3.2 Loss reason frequency distribution — §39.6.2

```sql
SELECT
  loss_primary_reason,
  COUNT(*)                                                   AS deal_count,
  ROUND(COUNT(*)::NUMERIC/SUM(COUNT(*)) OVER ()*100,1)      AS share_pct,
  SUM(CASE WHEN attributed_to_partner_id IS NOT NULL
        THEN 1 ELSE 0 END)                                  AS partner_sourced
FROM enterprise_deal_outcomes
WHERE outcome = 'closed_lost' AND NOT no_go_criteria_triggered
GROUP BY loss_primary_reason
ORDER BY deal_count DESC;
```

**Interpretation:** Top-loss reasons drive product and pricing feedback. Feed into roadmap prioritization if `share_pct ≥ 20%` for any single reason. `partner_sourced` column highlights whether partner-channel deals have different loss profiles.

### 3.3 Win rate by tier — §39.6.3

```sql
SELECT
  eps.tier_forecast                                           AS tier,
  COUNT(*) FILTER (WHERE edo.outcome = 'closed_won')         AS won,
  COUNT(*) FILTER (WHERE edo.outcome = 'closed_lost'
    AND NOT edo.no_go_criteria_triggered)                    AS lost,
  ROUND(
    COUNT(*) FILTER (WHERE edo.outcome = 'closed_won')::NUMERIC
    / NULLIF(COUNT(*) FILTER (
        WHERE edo.outcome IN ('closed_won','closed_lost')
          AND NOT edo.no_go_criteria_triggered
      ), 0) * 100, 1
  )                                                          AS win_rate_pct,
  ROUND(AVG(edo.sales_cycle_days)
    FILTER (WHERE edo.outcome = 'closed_won'), 0)            AS avg_cycle_days_won
FROM enterprise_pipeline_stages eps
JOIN enterprise_deal_outcomes edo ON edo.deal_id = eps.id
WHERE eps.stage IN ('S5', 'closed_lost')
GROUP BY eps.tier_forecast
ORDER BY tier;
```

**Interpretation:** Low win rate on a tier is a pricing signal (too expensive) or a fit signal (wrong ICP). High `avg_cycle_days_won` on Mid-Market vs. Enterprise may indicate the mid-market segment needs a faster evaluation path.

### 3.4 Competitor category impact — §39.6.4

```sql
SELECT
  competitor_category,
  COUNT(*)                                                   AS times_appeared,
  COUNT(*) FILTER (WHERE outcome = 'closed_lost')           AS losses_to_category,
  COUNT(*) FILTER (WHERE outcome = 'closed_won')            AS wins_against_category,
  ROUND(
    COUNT(*) FILTER (WHERE outcome = 'closed_won')::NUMERIC
    / NULLIF(COUNT(*),0) * 100, 1
  )                                                         AS win_rate_vs_category_pct,
  ROUND(AVG(acv_usd) FILTER (WHERE outcome = 'closed_won'),0) AS avg_won_acv_usd
FROM enterprise_deal_outcomes
WHERE competitor_category IS NOT NULL
GROUP BY competitor_category
ORDER BY times_appeared DESC;
```

**Interpretation:** Categories where `win_rate_vs_category_pct < 30%` are competitive vulnerabilities. If `unknown` share > 15% after Deal 20, evaluate OQ-WIN-02 (`docs/COST_MODEL.md §39.11`). No company names appear — `competitor_category` is a structured enum.

### 3.5 Sales cycle days by tier (p50 / p90) — §39.6.5

```sql
SELECT
  tier,
  COUNT(*)                                                         AS n,
  PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY sales_cycle_days)   AS median_days,
  PERCENTILE_CONT(0.9) WITHIN GROUP (ORDER BY sales_cycle_days)   AS p90_days,
  ROUND(AVG(sales_cycle_days), 0)                                  AS avg_days
FROM enterprise_deal_outcomes
WHERE outcome = 'closed_won'
GROUP BY tier
ORDER BY tier;
```

**Interpretation:** p90 > 120 days suggests the evaluation/legal review phase needs acceleration tooling (MSA pre-signature, security questionnaire automation). Track trend over successive quarters.

---

## 4. Quarterly Review Procedure

Run once per quarter, after the monthly `enterprise.arr_bridge_closed` DEC-030 event is emitted (confirm via the Admin Console audit log or Supabase `audit_log_events`).

| Step | Action | Owner | Output |
|---|---|---|---|
| 1 | Run §2 pre-flight counter query. Record `won`, `lost_excl_nogo`, `no_go_rejections`. | data-engineer | Running note |
| 2 | Run §3.2 (loss reasons) + §3.3 (win rate by tier) + §3.5 (sales cycle p50/p90). Export results to CSV. | data-engineer | `win-loss-review-YYYY-QN.csv` |
| 3 | If `won + lost_excl_nogo ≥ 10`: run §3.1 (stage conversion rates) and compare to `docs/COST_MODEL.md §37.3`. Document delta. | data-engineer | See §5 if deviation > 10 pp |
| 4 | If `competitor_category` populated on ≥ 3 deals: run §3.4. Otherwise skip (insufficient sample). | data-engineer | Optional `competitor-YYYY-QN.csv` |
| 5 | Write a 3–5 sentence narrative summarising: overall win rate, top loss reason, cycle time trend, any material competitor pattern. | founder + data-engineer | `win-loss-narrative-YYYY-QN.md` |
| 6 | Attach narrative and CSV exports to the quarterly investor update draft. The ARR bridge (`enterprise.arr_bridge_closed`) provides the revenue number; the win/loss narrative provides the qualitative context. | founder | Investor update |
| 7 | File WIN-E-003 (if quarter contains any `privacy.no_go_criteria_applied` events, or file zero-count attestation): export DEC-030 events for the quarter; save to `compliance/evidence/winloss/WIN-E-003_YYYY-QN.csv`. | compliance-officer | WIN-E-003 |

---

## 5. OQ-PIPE-01 Calibration Protocol (Deal 10, 20, 50)

**Trigger:** `COUNT(*) FROM enterprise_deal_outcomes WHERE outcome IN ('closed_won','closed_lost') AND NOT no_go_criteria_triggered` ≥ 10.

This is a **founder-gated** protocol — data-engineer prepares Steps 1–2; founder signs off on Steps 3–4.

### Step 1 — Run §3.1 query and document delta

Compute the delta for each stage vs. the base case in `docs/COST_MODEL.md §37.3`:

| Stage | §37.3 Benchmark | Actual (Deal 10) | Delta | Material? (> 10 pp) |
|---|---|---|---|---|
| S0 → S1 | — | — | — | — |
| S1 → S2 | — | — | — | — |
| S2 → S3 | — | — | — | — |
| S3 → S4 | — | — | — | — |
| S4 → S5 (close) | — | — | — | — |
| Overall win rate | — | — | — | — |

### Step 2 — Create DECISION_LOG entry

Create `docs/DECISION_LOG.md` entry **DEC-06X** (next available DEC- number). Required fields:

- **Title:** OQ-PIPE-01 resolution — pipeline conversion rate calibration (Deal N)
- **Date:** today
- **Owner:** founder + data-engineer
- **Deals analysed:** n (excluding no-go)
- **Recalibrated rates:** paste §37.3 delta table from Step 1
- **Implications for:** §19.1 sales velocity, §37.5 ARR build table
- **§37.3 update required:** yes / no

### Step 3 — Update §37.3 in COST_MODEL.md

If any stage deviation is material (> 10 pp), update `docs/COST_MODEL.md §37.3`. Mark the updated row **"Actuals (Deal N, YYYY-MM-DD)"** and retain the original benchmark row for reference. Commit as a minor patch (`enterprise: COST_MODEL §37.3 OQ-PIPE-01 calibration · vX.Y.Z`).

### Step 4 — Emit `enterprise.win_loss_analysis_recalibrated` DEC-030 event

Via Admin Console (founder role only). Required payload fields:

| Field | Value |
|---|---|
| `deals_analysed` | n (≥ 10, no-go excluded) |
| `no_go_excluded_count` | count of `no_go_criteria_triggered = true` rows |
| `s0_to_s1_pct` | from §3.1 output |
| `s1_to_s2_pct` | from §3.1 output |
| `s2_to_s3_pct` | from §3.1 output |
| `s3_to_s4_pct` | from §3.1 output |
| `s4_to_s5_pct` | from §3.1 output |
| `overall_win_rate_pct` | from §3.1 output |
| `decision_log_ref` | DEC-06X (slug from Step 2) — **required; PIPE-CHAIN-02 returns HTTP 422 if absent** |
| `cost_model_section_updated` | `'§37.3'` (literal string) |
| `calibration_date` | YYYY-MM-DD |

Repeat Steps 1–4 at Deal 20 and Deal 50 (stability check per `docs/COST_MODEL.md §39.10`).

---

## 6. SOC 2 Evidence Actions

| Artefact | Criteria | When to file | Path | Retention |
|---|---|---|---|---|
| **WIN-E-001** | CC5.2, CC1.4 | Annually — export `enterprise.deal_closed_won` DEC-030 chain events; verify `floor_respected: true` on all rows | `compliance/evidence/winloss/WIN-E-001_YYYY.csv` | 7 yr |
| **WIN-E-002** | CC5.2, CC4.1 | At OQ-PIPE-01 calibration (Deal 10/20/50) — export `enterprise.win_loss_analysis_recalibrated` DEC-030 chain event; include `decision_log_ref` | `compliance/evidence/winloss/WIN-E-002_YYYY-DealN.csv` | 7 yr |
| **WIN-E-003** | CC1.4, CC9.2 | Quarterly — export `privacy.no_go_criteria_applied` chain events; file zero-count attestation if none occurred | `compliance/evidence/winloss/WIN-E-003_YYYY-QN.csv` | 3 yr |

**Zero-count WIN-E-003 procedure:** If `SELECT COUNT(*) FROM audit_log_events WHERE action = 'privacy.no_go_criteria_applied' AND created_at BETWEEN '<quarter_start>' AND '<quarter_end>'` returns 0, file a signed attestation:

```
WIN-E-003 · YYYY-QN · Zero-count attestation
Period: YYYY-MM-DD to YYYY-MM-DD
no_go_criteria_applied events: 0
Attested by: compliance-officer · <date>
Affirms: No prospect was presented with a no-go use case during this quarter.
SOC 2 CC1.4: Ethical rejection policy was operational; zero-count is an affirmative attestation, not an omission.
```

Save to `compliance/evidence/winloss/WIN-E-003_YYYY-QN_zero-count.md`.

---

## 7. Founder Calendar — Quarterly Scheduling

After the monthly `enterprise.arr_bridge_closed` event is emitted, add the following to the founder calendar for the end-of-quarter investor update preparation:

| Task | Timing | Owner |
|---|---|---|
| Run §3.2 + §3.3 + §3.5 queries; export CSVs | T+0 (ARR bridge close day) | data-engineer |
| Draft win/loss narrative (3–5 sentences) | T+2 days | founder + data-engineer |
| File WIN-E-003 (or zero-count attestation) | T+3 days | compliance-officer |
| Attach narrative to investor update draft | T+5 days | founder |
| If ≥ 10 terminal outcomes: trigger OQ-PIPE-01 §5 protocol | As triggered | founder + data-engineer |

**ARR bridge trigger query** — confirm `arr_bridge_closed` event was emitted this month:

```sql
SELECT id, created_at, payload->>'period' AS period
FROM audit_log_events
WHERE action = 'enterprise.arr_bridge_closed'
ORDER BY created_at DESC
LIMIT 3;
```

---

## 8. Troubleshooting

| Symptom | Cause | Remedy |
|---|---|---|
| `enterprise_deal_outcomes` table not found | Migration `0074` not yet applied | Apply `docs/COST_MODEL.md §39.10` P0 checklist item 2; do not run queries until migration is confirmed |
| `tier_forecast` column missing from §3.3 query | `enterprise_pipeline_stages` DDL differs from §37.8.1 spec | Check `\d enterprise_pipeline_stages` in psql; align DDL before running |
| §3.1 `NULL` in conversion rate columns | Insufficient data in a stage (NULLIF guard) | Normal at low deal volumes; do not interpret NULL as 0% — report as "insufficient data" |
| PIPE-CHAIN-02 HTTP 422 on OQ-PIPE-01 emission | `decision_log_ref` field missing from payload | Create DEC-030 entry (§5 Step 2) before emitting event; `decision_log_ref` is required |
| WIN-E-003 `audit_log_events` query returns 0 rows | Correct — file zero-count attestation per §6 | File the attestation; zero-count is affirmative evidence per CC1.4 |

---

## 9. Open Questions

| ID | Question | Priority | Owner | Resolution path |
|---|---|---|---|---|
| **OQ-WIN-01** | Manual vs. automated `enterprise.deal_closed_won` emission | P2 | platform-engineer | Evaluate after Deal 5; automate if > 1 miss or > 48h delay. See `docs/COST_MODEL.md §39.11` |
| **OQ-WIN-02** | `competitor_category` unknown% threshold for subcategory enum | P2 | data-engineer | Run §3.4 at Deal 20; if `unknown` share > 15%, see `docs/COST_MODEL.md §39.11` |

---

*v1.0 (2026-06-15): Initial runbook — closes `docs/COST_MODEL.md §39.10` P1 checklist item 7 (M10: add §39.6 queries to data-engineer analytics runbook; schedule quarterly win/loss review in founder calendar). Five canonical analytics queries from `docs/COST_MODEL.md §39.6` reproduced verbatim (§3.1–§3.5); OQ-PIPE-01 four-step calibration protocol from `docs/COST_MODEL.md §39.7` (§5); quarterly review cadence with ARR bridge trigger (§4); WIN-E-001/002/003 SOC 2 evidence actions with zero-count WIN-E-003 attestation procedure (§6); founder calendar scheduling (§7). Privacy floor: no company names, contact email, or employee `user_id` in any query result, DEC-030 event, or evidence file; `competitor_category` is a structured enum only; `form_api` REVOKED from `enterprise_deal_outcomes`. Owner: data-engineer + enterprise-architect + customer-success + compliance-officer.*
