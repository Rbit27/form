# RB-ENT-QBR-01 · Enterprise Quarterly Business Review — Operational Runbook

> **Runbook ID:** RB-ENT-QBR-01
> **Type:** Recurring · quarterly cadence (Growth); monthly cadence (Enterprise dedicated)
> **Owner:** customer-success · reviewed by: compliance-officer · enterprise-architect
> **Last reviewed:** 2026-07-04
> **SOC 2 TSC:** CC2.2, CC4.1, CC9.2
> **Cross-refs:**
> - `docs/COST_MODEL.md §26.8` — QBR economics ($262.50/QBR, 7h fully-loaded)
> - `docs/COST_MODEL.md §26.9` — FEHS score bands and CS action SLAs
> - `docs/COST_MODEL.md §40.4` — permissible QBR aggregate metrics (Tier 1 k ≥ 5)
> - `docs/COST_MODEL.md §51` — tiered k-anonymity framework (QBR-K-ANON-01)
> - `docs/COST_MODEL.md §52` — `fehs_breakdown` schema (DEC-089)
> - `docs/ENTERPRISE_ONBOARDING.md §6.3` — 90-day QBR agenda and attendees
> - `docs/ENTERPRISE_ONBOARDING.md §A.2` — Admin Dashboard data pull queries
> - `docs/ENTERPRISE_SLA.md` — SLA tier definitions and QBR cadence per tier
> - `docs/AUDIT_LOG_SCHEMA.md §Enterprise Adoption Monitoring events` — `enterprise.qbr_completed` Zod schema
> - `docs/ENTERPRISE.md §Privacy floor` — seven non-negotiable constraints

---

## 1. Purpose and Scope

This runbook governs how FORM's Customer Success team prepares, conducts, and closes a Quarterly Business Review (QBR) with an enterprise tenant. It is the operational counterpart to the strategic framework in `docs/COST_MODEL.md §26.8` and the 90-day QBR agenda in `docs/ENTERPRISE_ONBOARDING.md §6.3`.

A QBR has three purposes:
1. **Performance review** — demonstrate programme health against the activation and engagement targets agreed at contract signature.
2. **Relationship cadence** — maintain executive sponsor engagement and surface blockers before they become churn signals.
3. **Audit evidence** — emit the `enterprise.qbr_completed` DEC-030 event with `privacy_floor_verified: true`, producing SOC 2 evidence artefact ADO-E-002 that confirms FORM communicated aggregate product performance to enterprise tenants on the contracted schedule (CC2.2, CC4.1).

**Out of scope:** this runbook does not cover pilot QBRs (Day 90 checkpoint — see `docs/ENTERPRISE_ONBOARDING.md §6.3`), CSM save protocol activations (see `docs/COST_MODEL.md §46`), or renewal negotiations (see `docs/COST_MODEL.md §42`).

---

## 2. QBR Cadence by Tier

| Tier | Cadence | CSM model | Notes |
|---|---|---|---|
| **Starter** | Not included | Shared inbox | No QBR SLA; founder conducts ad hoc check-ins at D90 per `ENTERPRISE_ONBOARDING.md §6.3` |
| **Growth** | Quarterly | Named CSM, shared 12–15 accounts | QBR is a contractual commitment per `docs/ENTERPRISE_SLA.md` |
| **Enterprise** | Monthly health check + quarterly full QBR | Dedicated CSM, max 6 accounts | Monthly = 30 min health sync (no `qbr_completed` event required); quarterly = this runbook |

**Scheduling rule:** schedule the first QBR at contract Day 90 (§ 2.1 below), then quarterly from that anchor date. For Enterprise dedicated accounts, schedule monthly health syncs at Day 30, Day 60, and Day 120 in between quarterly QBRs.

---

## 3. Pre-QBR Checklist (T−5 business days)

Complete all items before the QBR call.

```
□ 1. QBR type confirmed: quarterly / monthly / offboarding / kickoff
□ 2. Attendees invited: CSM + tenant_owner + tenant_admin + (optional) People Ops / Benefits lead
□ 3. Renewal date looked up: if renewal < 90 days away, flag to CS Director and prepare §10 renewal transition
□ 4. FEHS score retrieved: Admin Dashboard internal view → tenant FEHS panel → note fehs_score and fehs_status
□ 5. FEHS band reviewed: Green (75–100) / Yellow (50–74) / Red (25–49) / Critical (0–24)
□ 6. Adoption snapshot pulled: see §4 (data pull procedure)
□ 7. Support cases reviewed: any open P0/P1 tickets? any SLA breaches in last 90 days? (ENTERPRISE_SLA.md §6)
□ 8. Action items from previous QBR: confirm each item status (complete / in-progress / overdue)
□ 9. Expansion signal assessed: is Green-band + WAU ≥ 40%? if yes, prepare §11 expansion slide
□ 10. Privacy floor attestation rehearsed: CSM must be ready to attest QBR-PRIV-01 (§7)
□ 11. One-page data snapshot PDF generated: Admin Dashboard → Export → QBR Snapshot
```

**If FEHS = Critical (0–24):** do not proceed with standard QBR. Escalate to Founder + CS Director immediately. Follow `docs/INCIDENT_RESPONSE.md §12` (enterprise tenant SLA breach) and schedule a joint Founder + CSM call within 24h per `docs/COST_MODEL.md §26.9`. Downgrade this call type to a recovery call, not a QBR.

---

## 4. Admin Dashboard Data Pull

Pull these metrics no more than 24 hours before the QBR call. All queries run against the Admin Dashboard internal view (`form_admin` role). **No individual user data is accessed at any step.**

| Metric | Admin Dashboard path | k-anon floor | Tier |
|---|---|---|---|
| Total activated seats | Analytics → Activation → Last 90 days | k ≥ 5 (Tier 1) | |
| Activation rate % | Activated ÷ contracted seats × 100 | k ≥ 5 (Tier 1) | |
| Weekly Active User rate % | WAU ÷ contracted seats × 100 | k ≥ 5 (Tier 1) | |
| Coaching engagement rate | Sessions/week with a Victor interaction ÷ activated seats | k ≥ 5 (Tier 1) | |
| Workout log rate | Unique loggers ÷ activated seats × 100 | k ≥ 5 (Tier 1) | |
| Streak cohort size | Active streak holders (cohort must be ≥ 5 to report) | k ≥ 5 (Tier 1) | |
| Victor session volume | Total Victor coaching sessions last 30 days | k ≥ 5 (Tier 1) | |
| D30 retention cohort | Analytics → Cohorts → D30 (most recent) | k ≥ 5 (Tier 1) | |
| In-app NPS score | Analytics → NPS → Last 30 days | aggregate only | |
| Open support tickets | Support → Open Cases | — | |
| SLA uptime last 90 days | SLA Reports → Monthly | — | |

**Prohibited metrics (never present at QBR):**

| Prohibited item | Why |
|---|---|
| Individual employee names, emails, or user IDs | ENTERPRISE.md privacy floor rule 1 |
| Department-level breakdown | Enables re-identification by org chart context |
| Manager vs. direct-report split | Privacy floor rule 3 |
| Body composition aggregates | Privacy floor rule 6 |
| Mental health signal aggregates | Privacy floor rule 7 |
| ED-screening aggregate | Privacy floor rule 5 |
| CV session count | GDPR Art. 9 biometric processing; excluded per DEC-095 until DPIA supplement complete |
| Workout frequency distributions | Tier 2 health-adjacent; requires k ≥ 10 and new DEC before inclusion |
| Recovery score averages | Same as above |

If the customer requests any prohibited metric during the QBR, use the no-go escalation script from `docs/ENTERPRISE_ONBOARDING.md §A.3`:

> "I want to flag that what you're describing falls outside FORM's permitted use cases, which are defined in our MSA and are a product principle we don't negotiate on. I'll need to escalate this to our founder before we can continue. I'll follow up within one business day."

Do not offer workarounds. Escalate to Founder only.

---

## 5. QBR Call — Agenda (60 minutes)

| # | Agenda item | Duration | CSM notes |
|---|---|---|---|
| **1** | Results vs. success criteria | 15 min | Compare activation rate, WAU rate, NPS against the targets agreed in `docs/ENTERPRISE_ONBOARDING.md §6` (30/60/90-day milestones by tier). Show YoY or quarter-on-quarter trend if data available. |
| **2** | Employee feedback themes | 10 min | Share aggregated in-app NPS comments (never attribute individual quotes to employees or departments). If NPS comment set < 5 responses, report as "insufficient responses for reporting this period." |
| **3** | Support case review | 10 min | Review open tickets and any SLA adherence issues. If there were SLA breaches, reference the SLA credit calculation per `docs/ENTERPRISE_SLA.md §6` and confirm whether a credit applies. |
| **4** | Roadmap preview | 10 min | Share one or two upcoming features relevant to this tenant's use case. Do not make binding feature commitments. |
| **5** | Expansion discussion (if eligible) | 10 min | Deliver if: WAU health band = Green AND `fehs_score > 85`. Walk through seat expansion economics or white-label upgrade. Record `expansion_discussed: true` when filing the DEC-030 event. |
| **6** | Renewal timeline | 5 min | If renewal < 180 days: confirm renewal date, review process, and offer to schedule renewal call with CS Director. If renewal < 90 days: connect to `docs/COST_MODEL.md §42` renewal runbook immediately after this call. |

---

## 6. FEHS Band Response — Post-QBR Action

During the QBR or immediately after, assess FEHS band and take the corresponding action.

| FEHS band | Score | Post-QBR action | SLA |
|---|---|---|---|
| **Green** | 75–100 | Routine follow-up. If score > 85 and WAU ≥ 40%: initiate expansion discussion within 5 business days. Create Linear task referencing `enterprise.qbr_completed` event. | 5 business days |
| **Yellow** | 50–74 | Root cause identification within 5 business days. Schedule follow-up call to present remediation plan. Do not wait until next quarterly QBR. | 5 business days |
| **Red** | 25–49 | CSM + CS Director joint outreach within 2 business days. Identify primary risk signal from `fehs_breakdown`. Prepare recovery plan referencing `docs/COST_MODEL.md §40.6` CSM intervention playbook. | 2 business days |
| **Critical** | 0–24 | Do not hold standard QBR (see §3 pre-check). Founder + CSM joint call within 24h. Activate `docs/INCIDENT_RESPONSE.md §12`. | 24 hours |

**ADO-CHAIN-01 monitoring invariant:** If `enterprise.adoption_health_downgraded` was emitted for this tenant since the last QBR, the Linear task created post-QBR must be closed with `qbr_completed` referenced. The `evidence-collection-cron` Worker monitors for downgrade events with no corresponding QBR or CSM check-in within 10 business days and emits `system.csm_followup_overdue` (LOW, 1yr). This is an operational control, not a compliance gate.

---

## 7. DEC-030 Event — Mandatory Post-QBR Filing

**File within 60 minutes of QBR call end.** Use the Admin Console "Complete QBR" modal (authenticated via Cloudflare Access — `form_admin` role required).

### 7.1 QBR-PRIV-01 Attestation

Before submitting, the CSM must attest `privacy_floor_verified: true`. This attestation covers:
- Only Tier 1 (k ≥ 5) permissible metrics were presented (§4 table).
- No individual employee user_id, name, email, health value, coaching content, or GDPR Art. 9 special-category data was shared with or accessed by the customer.
- No department-level breakdown was provided.
- k-anonymity floor was enforced: any aggregate with n < 5 was suppressed and reported as "insufficient data."
- QBR-K-ANON-01 (DEC-088): Tier 1 k ≥ 5 floor was respected for all six §4 permissible metrics.

The `emit-audit-event` Worker returns **HTTP 422 `QBR_PRIV_01_ATTESTATION_MISSING`** if `privacy_floor_verified` is absent. The event is not emitted and must be re-submitted after attestation.

### 7.2 `enterprise.qbr_completed` Event Schema

```typescript
// Source: docs/AUDIT_LOG_SCHEMA.md §Enterprise Adoption Monitoring events
// DEC-030 class: STANDARD · retention: 7 years · privacy: aggregate only
{
  tenant_id:                string,          // FORM-internal org slug
  qbr_date:                 string,          // ISO 8601 date of QBR call (not filing date)
  csm_user_id:              string,          // CSM's FORM-internal admin UUID (not customer user)
  qbr_type:                 'quarterly' | 'monthly' | 'kickoff' | 'offboarding',
  activation_rate_pct:      number,          // aggregate — no user_id
  weekly_active_rate_pct:   number,          // aggregate — no user_id
  action_items_count:       number,          // integer
  customer_exec_attended:   boolean,
  expansion_discussed:      boolean,         // true if §11 expansion slide was delivered
  privacy_floor_verified:   true,            // QBR-PRIV-01 literal — HTTP 422 if absent or false

  // FEHS fields (optional pre-M10; required at M10 Admin Console modal launch — DEC-089)
  fehs_score_at_qbr?:       number,          // 0–100 composite
  fehs_breakdown?: {
    activation_pct:         number,          // signal 1 — weight 25%
    wau_pct:                number,          // signal 2 — weight 25%
    seat_utilisation_pct:   number,          // signal 3 — weight 20%
    exec_engagement_score:  number,          // signal 4 — weight 15%
    support_volume_inverse: number,          // signal 5 — weight 10%
    renewal_distance_score: number,          // signal 6 — weight 5%
  }
}
```

**FEHS-CHAIN-01 invariant** (enforced by `emit-audit-event` Worker):

| Violation | HTTP | Error code |
|---|---|---|
| `fehs_score_at_qbr` present without `fehs_breakdown` | 422 | `FEHS_CHAIN_01_BREAKDOWN_REQUIRED` |
| `fehs_breakdown` present without `fehs_score_at_qbr` | 422 | `FEHS_CHAIN_01_SCORE_REQUIRED` |
| `abs(weighted_sum(fehs_breakdown) − fehs_score_at_qbr) > 0.5` | 422 | `FEHS_CHAIN_01_SCORE_MISMATCH` |

Weighted sum formula: `(activation_pct × 0.25) + (wau_pct × 0.25) + (seat_utilisation_pct × 0.20) + (exec_engagement_score × 0.15) + (support_volume_inverse × 0.10) + (renewal_distance_score × 0.05)`

### 7.3 Privacy Constraints on Event Payload

The following are **never** included in any field of `enterprise.qbr_completed` or any companion event:
- Individual employee `user_id`, name, email, or health value
- GDPR Art. 9 special-category data (biometric, mental health, body composition)
- Verbatim NPS comments or any quotable employee statement
- Department-level sub-aggregates
- `form_api` does not have access to `enterprise_adoption_snapshots` — event is emitted by CSM via admin tooling only

---

## 8. QBR Written Summary (T+48h deadline)

CSM sends a written summary within 48 hours of the QBR call. The summary must include:

1. Attendees and date.
2. Metrics presented (copy from the one-page PDF snapshot).
3. Action items with owner and due date.
4. Expansion: discussed / not discussed / proposal pending.
5. Renewal: date confirmed / timeline discussed / CSM Director loop-in scheduled.
6. DEC-030 event ID (confirmation the event was filed).

**File the summary in the account CRM.** Do not include individual employee data in the CRM record. Any reference to employee engagement must use the same aggregate metrics presented in the call.

---

## 9. SOC 2 Evidence Artefact — ADO-E-002

**ADO-E-002** is the SOC 2 evidence artefact that confirms every active Growth/Enterprise account received a QBR within the contracted cadence.

| Field | Value |
|---|---|
| **Artefact ID** | ADO-E-002 |
| **SOC 2 criteria** | CC2.2 (external communication — demonstrates FORM communicated product status to enterprise tenants on schedule); CC4.1 (QBR as performance monitoring activity) |
| **Collection cadence** | Quarterly |
| **Content** | Export of all `enterprise.qbr_completed` DEC-030 events for the observation quarter; confirms every active Growth/Enterprise account has at least one `qbr_completed` event within the contracted interval with `privacy_floor_verified: true` in every row |
| **Collection path** | `compliance/evidence/adoption/ADO-E-002_<YYYY-QN>.csv` |
| **Retention** | 3 years |
| **First collection** | From M11 (when first Growth/Enterprise account completes first post-go-live QBR) |

**Quarterly ADO-E-002 collection query (compliance-officer runs before Vanta upload):**

```sql
-- No individual user data in any column; tenant_id is FORM-internal UUID
SELECT
  tenant_id,
  qbr_date,
  qbr_type,
  fehs_score_at_qbr,
  activation_rate_pct,
  weekly_active_rate_pct,
  privacy_floor_verified,
  occurred_at
FROM enterprise_events
WHERE
  event_type = 'enterprise.qbr_completed'
  AND occurred_at BETWEEN :quarter_start AND :quarter_end
ORDER BY tenant_id, occurred_at;
```

**Coverage check** — run immediately after the quarterly export:

```sql
-- Active tenants with no QBR in the quarter (should be zero for Growth/Enterprise)
SELECT ec.tenant_id, ec.sla_tier
FROM enterprise_contracts ec
WHERE
  ec.lifecycle_status = 'active'
  AND ec.sla_tier IN ('growth', 'enterprise')
  AND NOT EXISTS (
    SELECT 1 FROM enterprise_events ev
    WHERE
      ev.tenant_id = ec.tenant_id
      AND ev.event_type = 'enterprise.qbr_completed'
      AND ev.occurred_at BETWEEN :quarter_start AND :quarter_end
  );
```

If this query returns any rows: the CSM must schedule and complete a catch-up QBR before the ADO-E-002 filing deadline, or compliance-officer must document the gap with a written justification note appended to the artefact file. A gap in QBR cadence for a contracted Growth or Enterprise account is a CC2.2 control failure.

---

## 10. Renewal Transition (< 180 days to renewal)

If the renewal date is within 180 days, append the following to the standard QBR agenda:

1. Confirm renewal date and customer's internal budget cycle.
2. Identify renewal decision-maker (may differ from day-to-day admin contact).
3. Ask if there are concerns likely to affect renewal. Document in CRM — do not collect individual health data.
4. If FEHS = Yellow or below: loop in CS Director before renewal call.
5. If customer signals non-renewal or flags concerns: emit `enterprise.renewal_risk_flagged` (STANDARD, 3yr) and activate `docs/COST_MODEL.md §34` (Renewal Risk Register).

For detailed renewal mechanics, pricing, and discount authority, see `docs/COST_MODEL.md §42`.

---

## 11. Expansion Discussion Protocol

Expansion is appropriate when:
- `wau_health_band = 'green'` (WAU ≥ 40% of contracted seats), **and**
- `fehs_score > 85`, **and**
- Fewer than 2 open P1/P0 support tickets.

If eligible, include one expansion slide covering:
- Current seat utilisation trend (aggregate, not individual).
- Pro-rata cost for 25-seat increment at current rate (minimum expansion per `docs/COST_MODEL.md §41.2`).
- Tier upgrade option if currently on Starter or Growth and usage pattern suggests Enterprise features would add value.

Record `expansion_discussed: true` in the `qbr_completed` DEC-030 event. This triggers a Linear task creation per `docs/COST_MODEL.md §41.10 item 5` (proposal to be delivered within 5 business days).

**Do not apply verbal discounts during the QBR call.** Any discount beyond standard rates requires the discount authority matrix from `docs/COST_MODEL.md §41.5`. Expansion discounts below the `COST_MODEL.md §31.5` price floor are blocked by `emit-audit-event` Worker with HTTP 422.

---

## 12. When NOT to Hold a Standard QBR

| Condition | Correct action |
|---|---|
| FEHS = Critical (0–24) | Founder + CSM call within 24h; activate `INCIDENT_RESPONSE.md §12` |
| Active P0 incident on FORM infrastructure | Postpone QBR until incident resolved; notify customer via incident comms |
| Tenant is in contract termination proceedings | Switch to offboarding QBR (`qbr_type: 'offboarding'`); route to `ENTERPRISE_ONBOARDING.md §11` |
| Account is in pilot phase (< Day 90) | Use pilot QBR format from `ENTERPRISE_ONBOARDING.md §6.3`; emit `pilot.milestone_review` event instead |
| CSM has been on this account < 2 weeks (handoff) | Shadow previous CSM or CS Director on first QBR; do not file `qbr_completed` solo until handoff documentation is complete |

---

## 13. Cost Reference

From `docs/COST_MODEL.md §26.8`:

| Tier | QBR cost (fully-loaded) | Cost as % ACV | Notes |
|---|---|---|---|
| Starter (ad hoc) | $262.50 per call | ~3.7% at minimum ACV | Not in contract; discretionary |
| Growth (quarterly) | $262.50 per QBR | ~2.2% of $72k ACV | 4 QBRs/year = $1,050 |
| Enterprise (monthly health + quarterly full) | $262.50 per full QBR | ~1.0% of $96k+ ACV | 4 full + 8 health syncs/year |

QBR ROI: 3.4× in Year 1 if one logo churn is prevented per year (source: `docs/COST_MODEL.md §26.8`). The binding constraint is CSM capacity, not ROI.

---

## 14. Privacy Floor — Restatement

The following are **non-negotiable** at every QBR, regardless of customer request:

1. HR never sees individual user data. Aggregates only.
2. Aggregates suppressed when cohort n < 5 (Tier 1 k-anonymity, QBR-K-ANON-01).
3. Manager-on-direct-report data is never available.
4. Employee can revoke company-association at any time; their data leaves all aggregates immediately.
5. ED-screening data: never aggregated to employer at any cohort size.
6. Body composition data: never aggregated to employer.
7. Mental health data: never aggregated to employer.
8. CV session count: excluded until DPIA supplement + DPA Schedule A amendment complete (DEC-095).

These constraints are enforced at three layers: Admin Dashboard RLS (k ≥ 5 suppression at query layer), `privacy_floor_verified` chain invariant in `enterprise.qbr_completed`, and CSM training. **Customer cannot override any of these.** If a customer demands individual data, escalate to Founder; lose the deal if necessary (see `docs/ENTERPRISE.md §When we say no`).

---

## 15. Implementation Checklist

| # | Task | Owner | Priority | Milestone | Status |
|---|---|---|---|---|---|
| 1 | Build Admin Console "Complete QBR" modal with `privacy_floor_verified` attestation checkbox, FEHS auto-population from latest `health_score_updated`, and `qbr_completed` event emission. | platform-engineer | **P0** | M10 | [ ] |
| 2 | Ensure FEHS-CHAIN-01 invariant is implemented in `emit-audit-event` Worker (co-presence and tolerance check for `fehs_score_at_qbr` + `fehs_breakdown`). | security-engineer | **P1** | M9 | [ ] |
| 3 | Train CSM on this runbook before first enterprise go-live; specifically: QBR-PRIV-01 attestation scope, k-anonymity Tier 1 floor, no-go escalation script, and `fehs_breakdown` interpretation. | customer-success | **P0** | Before first enterprise pilot QBR | [ ] |
| 4 | Add quarterly ADO-E-002 collection to compliance calendar (`compliance/evidence/adoption/ADO-E-002_<YYYY-QN>.csv`). | compliance-officer | **P1** | M11 | [ ] |
| 5 | Register QBR cadence breach alert: if `enterprise_contracts` row with `sla_tier IN ('growth', 'enterprise')` has no `qbr_completed` event in the last 95 days, emit `system.qbr_cadence_breach` (HIGH, 3yr) to PagerDuty `#alerts-cs`. | devops-lead | **P1** | M10 | [ ] |

---

*v1.0 · 2026-07-04 · enterprise-builder cloud worker*

*New operational runbook. Provides step-by-step QBR preparation, call execution, DEC-030 filing, and SOC 2 evidence artefact collection procedure for enterprise Customer Success. Consolidates the QBR framework scattered across `docs/COST_MODEL.md §26.8/§40.4/§51/§52` and `docs/ENTERPRISE_ONBOARDING.md §6.3/§A.2` into a single operator-facing document. Privacy floor: no individual employee `user_id`, name, email, health value, coaching content, or GDPR Art. 9 special-category data appears in any QBR metric, DEC-030 event, or CRM record governed by this runbook; aggregate k ≥ 5 floor (QBR-K-ANON-01, DEC-088) enforced at all stages; `privacy_floor_verified: true` literal required in `enterprise.qbr_completed` (QBR-PRIV-01). Cross-references: `docs/COST_MODEL.md §26.8` (QBR economics); `docs/COST_MODEL.md §26.9` (FEHS bands 75–100 Green / 50–74 Yellow / 25–49 Red / 0–24 Critical); `docs/COST_MODEL.md §40.4` (permissible QBR metrics); `docs/COST_MODEL.md §40.9` (ADO-E-002 evidence artefact); `docs/COST_MODEL.md §51` (QBR-K-ANON-01); `docs/COST_MODEL.md §52` (fehs_breakdown schema, DEC-089); `docs/ENTERPRISE_ONBOARDING.md §6.3` (90-day QBR agenda); `docs/ENTERPRISE_ONBOARDING.md §A.2` (Admin Dashboard data pull); `docs/ENTERPRISE_ONBOARDING.md §A.3` (no-go escalation script); `docs/ENTERPRISE_SLA.md §3` (QBR cadence per tier); `docs/ENTERPRISE.md §Privacy floor` (seven non-negotiable constraints); `docs/AUDIT_LOG_SCHEMA.md §Enterprise Adoption Monitoring events` (`enterprise.qbr_completed` QBR-PRIV-01 + FEHS-CHAIN-01 schemas); `docs/COST_MODEL.md §34` (Renewal Risk Register — triggered at FEHS Yellow/Red + renewal < 180d); `docs/COST_MODEL.md §41` (Expansion Economics — `expansion_discussed: true` trigger); `docs/COST_MODEL.md §46` (CSM Save Protocol — Critical-band escalation path); `docs/INCIDENT_RESPONSE.md §12` (enterprise tenant SLA breach — Critical-band activation). SOC 2: CC2.2 (external communication — `qbr_completed` chain invariant proves communication occurred); CC4.1 (performance monitoring — QBR cadence + ADO-E-002 quarterly export); CC9.2 (customer commitments — QBR is a contractual deliverable for Growth/Enterprise). Owner: customer-success + compliance-officer + enterprise-architect.*
