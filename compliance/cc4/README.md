# FORM · CC4 Monitoring Activities — Evidence Index

> SOC 2 Type II · Trust Service Criteria CC4 (Monitoring Activities).
> Owner: compliance-officer + security-engineer. Review: quarterly or on any change to monitoring tool stack, incident response structure, or board/advisory composition.
> Reference: `docs/SOC2_READINESS.md §32`, `docs/INCIDENT_RESPONSE.md §2`, `docs/OBSERVABILITY.md §16`, `docs/AUDIT_LOG_SCHEMA.md`, DEC-030.

---

## CC4 Criteria Coverage

| Criterion | Description | Status |
|---|---|---|
| CC4.1 — Ongoing evaluations | Entity selects, develops, and performs ongoing evaluations to ascertain whether components of internal control are present and functioning | 🟡 Partial — five ongoing evaluation mechanisms designed and configured (§15 compliance calendar, §25.5 daily HMAC chain check, §23 quarterly access review, weekly Supabase health export, monthly Cloudflare analytics); pre-launch: observation-period evidence not yet accumulated (CC4-GAP-002) |
| CC4.1 — Separate evaluations | Entity selects, develops, and performs separate (periodic) evaluations to ascertain whether components of internal control are present and functioning | 🟡 Partial — three separate evaluation mechanisms defined: §16 annual external pentest, §23 quarterly access review, §15 year-over-year SOC 2 effectiveness review; first pentest execution pending (CC4-GAP-002) |
| CC4.2 — Deficiency communication | Entity evaluates and communicates internal control deficiencies in a timely manner to those responsible for taking corrective action, including senior management and the board | 🟡 Partial — structured deficiency log exists (CC4-E-001); three-tier compensating control designed for board-less governance (CC4-E-003, CC4-GAP-003 P0); formal board communication not yet operational at pre-seed stage |

---

## Evidence Files

| File | Exhibit ID | Description | SOC 2 Criteria | Status |
|---|---|---|---|---|
| `compliance/cc4/control-deficiency-log.csv` | CC4-E-001 | Structured deficiency register — mandatory columns: `deficiency_id`, `criterion`, `description`, `severity`, `root_cause`, `linear_ticket_url`, `owner`, `target_date`, `closed_date`, `auditor_exhibit_ref`; seeded with all known pre-launch gaps | CC4.2 | 🟢 In Force · 2026-05-20 |
| `compliance/cc4/evidence-collection-plan.md` | CC4-E-002 | Ongoing and separate evaluation evidence collection schedule — 5 ongoing controls + 3 separate evaluations; collection frequency, storage path, responsible owner, start date, auditor traceability for each; timeline from T+0 (launch) to T+365 | CC4.1, CC4.2 | 🟢 Exists · pre-launch; collection begins at launch |
| `compliance/cc4/deficiency-communication-procedure.md` | CC4-E-003 | Three-tier compensating control for CC4.2 board communication requirement — Tier 1: monthly founder self-review memo (`soc2-monthly-YYYY-MM.md`); Tier 2: advisory board quarterly notification with P0 out-of-cycle 5-business-day rule; Tier 3: investor observer quarterly deficiency summary; execution log; Series A expiry trigger | CC4.2 | 🟢 Exists · compensating control active; expires 90 days post-Series-A |
| _(pending launch)_ `compliance/evidence/soc2-monthly-YYYY-MM.md` | Tier 1 memos | Monthly founder self-review memo — deficiencies opened/closed, outstanding count, evidence collection status, sub-processor changes, next review date; filed last business day of each month; git commit SHA serves as attestation | CC4.1, CC4.2 | 🔴 Not yet started — collection begins at launch |
| _(pending launch)_ `compliance/evidence/chain-check-YYYY-MM/YYYY-MM-DD.json` | S-007 logs | Daily HMAC chain integrity probe results — probe S-007 (`GET /internal/audit/health`), 10-minute cadence; `chain_valid`, last-entry age, region, hash snapshot; Better Stack + monthly directory mirror | CC4.1, CC7.3 | 🔴 Not yet started — probe S-007 deployed at M3 per §15.2 |
| _(pending launch)_ `compliance/evidence/supabase-YYYY-MM-DD.json` | Weekly exports | Weekly Supabase health snapshot — connection count, replication lag, WAL size, RLS policy version, PITR last success, storage utilization; every Monday 09:00 UTC | CC4.1, A1.1 | 🔴 Not yet started — begins at launch |
| _(pending launch)_ `compliance/evidence/cloudflare-YYYY-MM.csv` | Monthly exports | Monthly Cloudflare analytics export — request totals, WAF rule triggers by rule ID, bot score distribution, DDoS events, 4xx/5xx rate by endpoint; SHA-256 hash logged to HMAC audit log | CC4.1, CC7.2, A1.2 | 🔴 Not yet started — begins at launch |
| _(pending launch)_ `compliance/evidence/access-review-YYYY-QN.md` | Quarterly access review | Quarterly access review artifact (§23.6 template) — inventory snapshot, authorization comparison, deprovisioning actions, CC6 control effectiveness assessment (Step 5); also serves as CC4.2 quarterly control effectiveness check | CC4.1, CC4.2, CC6.2, CC6.3 | 🔴 Not yet started — first execution PRE-23 |
| _(pending)_ `compliance/evidence/pentest-YYYY/` | Annual pentest | Annual external penetration test package — executive summary, remediation evidence (all critical/high findings closed), HMAC chain verification, signed scope statement, retest confirmation; budget $15–30k; shortlist: Bishop Fox, NCC Group, Cobalt.io, Cure53 | CC4.1, CC7.1, CC7.2 | 🔴 Not yet started — first engagement Year 1 Q1 |
| _(pending launch)_ `compliance/evidence/advisory-brief-YYYY-QN.md` | Tier 2 summaries | Quarterly advisory board deficiency communication summary — attendees, P0/P1 deficiency list, advisor questions, acknowledgement confirmation; filed within 5 business days of call | CC4.2 | 🔴 Not yet started |
| _(pending launch)_ `compliance/evidence/investor-update-YYYY-QN-sent.txt` | Tier 3 notifications | Quarterly investor observer deficiency summary confirmation — sent email or forwarded receipt; corresponds to deficiency section in `docs/INVESTOR_UPDATE_TEMPLATE.md` | CC4.2 | 🔴 Not yet started |
| _(pending)_ `compliance/evidence/soc2-annual-YYYY.md` | Annual review memo | Year-over-year SOC 2 readiness review — all TSC criteria assessed; control effectiveness ratings; gap closure rate; updated readiness percentage; input to audit firm fieldwork | CC4.1, CC4.2 | 🔴 Not yet started — first review Year 1 Q4 |

---

## Gap Status

| Gap ID | Item | Priority | Status | Closed by |
|---|---|---|---|---|
| **CC4-GAP-001** | No formal control deficiency log artifact | P1 — blocks CC4.2 from 🟡 → 🟢 | 🟢 Closed · 2026-05-20 | `compliance/cc4/control-deficiency-log.csv` (CC4-E-001) |
| **CC4-GAP-002** | Monitoring evidence not yet in steady state — pre-launch; observation-period evidence not accumulated | P1 — accepted; resolves automatically at launch + 6 months | 🔴 Open — evidence collection begins at launch per CC4-E-002 schedule | `compliance/cc4/evidence-collection-plan.md` (CC4-E-002); gap closes at launch + 6 months when full observation window is covered |
| **CC4-GAP-003** | No board or audit committee to receive CC4.2 deficiency communications | P0 — compensating control accepted; structural gap | 🟡 Compensating control active — three-tier structure (CC4-E-003) operating; must be upgraded to formal audit committee within 90 days of Series A close | `compliance/cc4/deficiency-communication-procedure.md` (CC4-E-003); execution log tracks Tier 1/2/3 per quarter |

---

## Compliance Calendar

| Cadence | Task | Artefact |
|---|---|---|
| Monthly (last business day) | **Tier 1:** File `soc2-monthly-YYYY-MM.md` founder self-review memo — deficiencies opened/closed, outstanding counts, evidence collection status, sub-processor changes | `compliance/evidence/soc2-monthly-YYYY-MM.md` |
| Monthly (1st of following month) | Cloudflare analytics CSV export for prior month; SHA-256 hash logged to HMAC audit log | `compliance/evidence/cloudflare-YYYY-MM.csv` |
| Weekly (Monday 09:00 UTC) | Supabase health snapshot export | `compliance/evidence/supabase-YYYY-MM-DD.json` |
| Continuous (10-min cadence) | HMAC chain integrity probe S-007 (`GET /internal/audit/health`); daily roll-up filed monthly | `compliance/evidence/chain-check-YYYY-MM/YYYY-MM-DD.json` |
| Quarterly (Jan / Apr / Jul / Oct) | Quarterly access review (§23) — enumerate, compare, deprovision, CC4.2 control effectiveness assessment (Step 5) | `compliance/evidence/access-review-YYYY-QN.md` |
| Quarterly (next advisory call; P0 within 5 business days) | **Tier 2:** Advisory board deficiency communication — standing agenda item; written summary within 5 business days | `compliance/evidence/advisory-brief-YYYY-QN.md` |
| Quarterly | **Tier 3:** Investor observer deficiency summary in quarterly investor update | `compliance/evidence/investor-update-YYYY-QN-sent.txt` |
| Annual (Q1, January) | External penetration test engagement; remediation evidence package assembled | `compliance/evidence/pentest-YYYY/` |
| Annual (Q4) | Year-over-year SOC 2 readiness review — all TSC criteria, control effectiveness, gap closure rate | `compliance/evidence/soc2-annual-YYYY.md` |
| At Series A close (within 90 days) | Replace CC4-E-003 compensating control with formal audit committee charter | `compliance/board/audit-committee-charter.md`; update CC4-GAP-003 closed_date in `control-deficiency-log.csv` |
| On any new P0 deficiency | Out-of-cycle Tier 2 notification to all advisors within 5 business days; file to `compliance/evidence/p0-notification-YYYY-MM-DD-[ID].md` | `compliance/evidence/p0-notification-*.md` |
| On any new deficiency (any severity) | Add row to `compliance/cc4/control-deficiency-log.csv` within 24 hours | `compliance/cc4/control-deficiency-log.csv` |

---

## Open Action Items

| ID | Action | Owner | Priority |
|---|---|---|---|
| CC4-GAP-002-A | At launch: verify all five automated collection pipelines are running (S-007 probe, Supabase weekly export, Cloudflare monthly export, Better Stack heartbeats); file first T+30 evidence checkpoint memo | devops-lead + compliance-officer | P1 — at launch |
| CC4-GAP-002-B | Wire `compliance.monthly_review`, `compliance.access_review`, and `compliance.cloudflare_export` HMAC audit log events to automated GitHub Action that creates monthly directory in `compliance/evidence/chain-check-YYYY-MM/` | security-engineer | P1 — before observation period begins |
| CC4-GAP-003-A | Execute first Tier 2 advisory board deficiency communication (standing agenda item); file `advisory-brief-YYYY-QN.md` within 5 business days of call | founder | P0 — next advisory call |
| CC4-GAP-003-B | Confirm investor observer rights in term sheet at Seed close; begin Tier 3 investor update deficiency section from first post-close quarterly update | founder | P0 — Seed close |
| CC4-GAP-003-C | At Series A close: draft audit committee charter within 90 days; file to `compliance/board/audit-committee-charter.md`; close CC4-GAP-003 in deficiency log | compliance-officer | P0 — 90 days post-Series-A |

---

*v1.0 · 2026-06-07 · Owner: compliance-officer + security-engineer*
*Gaps closed this version: CC4-GAP-001 (control-deficiency-log.csv in force · 2026-05-20)*
*Gaps open: CC4-GAP-002 (resolves launch + 6 months), CC4-GAP-003 (P0 compensating control active; expires Series A + 90 days)*
