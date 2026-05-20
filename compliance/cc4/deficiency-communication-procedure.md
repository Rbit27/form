# Deficiency Communication Procedure

**File reference:** CC4-E-003
**SOC 2 criterion:** CC4.2 — Evaluates and Communicates Deficiencies
**Remediates:** CC4-GAP-003 (P0) — no board or audit committee exists to receive deficiency communications
**Owner:** founder
**Effective date:** 2026-05-20
**Next review:** Quarterly (aligned with advisory board call) or upon Series A close — whichever is first

---

## Purpose and Scope

CC4.2 of the AICPA Trust Services Criteria requires that an entity evaluates and communicates identified control deficiencies to those parties responsible for taking corrective action, including senior management and the board of directors or its equivalent.

FORM is a pre-seed, solo-founder company. No formal board of directors exists. This is a structural gap acknowledged explicitly as CC4-GAP-003 with P0 severity. This document defines the compensating control accepted in lieu of a formal board: a three-tier communication structure that routes deficiency information to every oversight stakeholder that exists at the current company stage.

The three tiers are:

1. **Tier 1 — Founder self-review:** Monthly SOC 2 evidence memo, filed as a git-committed artifact
2. **Tier 2 — Advisory board notification:** Quarterly P0/P1 deficiency communication at the formal advisory call
3. **Tier 3 — Investor observer:** Quarterly deficiency summary in the investor update

All three tiers must operate simultaneously. Failure to execute any tier within its defined period is itself a control deficiency that must be entered into `compliance/cc4/control-deficiency-log.csv`.

---

## Series A Trigger

> **This procedure expires and must be replaced by a formal audit committee charter within 90 days of Series A close.**

At Series A close, a formal board of directors with fiduciary accountability is constituted. Within 90 days of close, the compliance-officer must:

1. Draft an audit committee charter (or assign board oversight of the audit function to the full board if an audit committee is not yet formed).
2. File the charter as a board-approved artifact in `compliance/board/audit-committee-charter.md`.
3. Update `compliance/cc4/control-deficiency-log.csv` to close CC4-GAP-003 with `closed_date` = date of board approval and `auditor_exhibit_ref` = the charter file path.
4. Archive this procedure as superseded; do not delete it (it forms part of the SOC 2 Type I/II historical audit trail).

If 90 days pass after Series A close without a compliant audit committee charter, a new P0 deficiency must be opened immediately.

---

## Tier 1 — Founder Self-Review

### Purpose

The founder, acting as both management and the oversight body at this company stage, performs a structured monthly review of all open control deficiencies and evidence collection status. The review is documented as a Markdown memo and committed to the compliance repository, where the git commit signature serves as the attestation record.

### Cadence

Monthly, on the last business day of each calendar month. The first review covers the month of launch.

### Storage

`compliance/evidence/soc2-monthly-YYYY-MM.md`

Where `YYYY-MM` is the month being reviewed (not the date of filing). File within 3 business days of month end.

### Template

The monthly memo must use the following template exactly. Fields in `[brackets]` must be populated; do not leave them blank.

---

```
# SOC 2 Monthly Evidence Memo — [YYYY-MM]

**Date filed:** [YYYY-MM-DD]
**Reviewer:** [Founder name]
**Git commit serves as signature.**

---

## 1. Deficiencies Opened This Month

| Deficiency ID | Criterion | Severity | Description (one line) | Owner | Target Date |
|---|---|---|---|---|---|
| [ID or "None"] | | | | | |

## 2. Deficiencies Closed This Month

| Deficiency ID | Criterion | Severity | Closure Date | Auditor Exhibit |
|---|---|---|---|---|
| [ID or "None"] | | | | |

## 3. Outstanding Deficiency Summary

| Severity | Count |
|---|---|
| P0 | [n] |
| P1 | [n] |
| P2 | [n] |
| **Total open** | **[n]** |

## 4. Evidence Collection Status

| Control | Last Evidence Filed | Status |
|---|---|---|
| Monthly SOC2 memo (prior month) | [path or "N/A — first memo"] | [On schedule / Late / Not yet started] |
| Daily chain integrity (S-007) | [YYYY-MM-DD last passing run] | [On schedule / Gap detected] |
| Quarterly access review | [YYYY-QN or "Not yet due"] | [Complete / Due in [n] days / Overdue] |
| Supabase weekly export | [YYYY-MM-DD last export] | [On schedule / Gap detected] |
| Cloudflare monthly export | [YYYY-MM or "Not yet due"] | [Complete / Due in [n] days / Overdue] |
| Annual pentest | [YYYY or "Not yet due"] | [Complete / Scheduled / Not yet due] |

## 5. Sub-Processor Changes This Month

[None — or list any new or removed sub-processors with 30-day notice sent date]

## 6. Next Review Date

[YYYY-MM-DD — last business day of next month]

---

*Committed by: [founder name]. Git commit SHA serves as attestation. Filed: [YYYY-MM-DD].*
```

---

### Execution Notes

- File the memo to `compliance/evidence/soc2-monthly-YYYY-MM.md` and commit. The commit SHA is logged in the HMAC audit log as `event_type: compliance.monthly_review` automatically.
- If a month is missed, open a new deficiency in `control-deficiency-log.csv` immediately upon discovery. Do not backfill a memo with a false date.
- The monthly memo is the primary input for Tier 2 (advisory board) and Tier 3 (investor update) deficiency summaries.

---

## Tier 2 — Advisory Board Notification

### Purpose

FORM's advisory board (documented in `docs/ADVISORY_BOARD.md`) does not carry fiduciary duty, but it constitutes the most qualified external oversight available at pre-seed stage. Any P0 or P1 deficiency is communicated to the advisory board at the next quarterly advisory call as a standing agenda item. This satisfies the spirit of CC4.2's board communication requirement at pre-Series-A stage.

### Cadence

Quarterly advisory calls — schedule defined in `docs/ADVISORY_BOARD.md`. P0 deficiencies that arise between calls must be communicated via email to all advisors within 5 business days of the deficiency being opened, rather than waiting for the next quarterly call.

### Quorum

At least 1 advisor must be present (in-person, video, or confirmed by email) for the deficiency communication to count as executed. If quorum is not achieved, reschedule within 10 business days and document the rescheduling in the written summary.

### Agenda Item Template

The following agenda item must appear in every quarterly advisory call agenda that has open P0 or P1 deficiencies. If there are no open P0/P1 deficiencies, a brief statement to that effect must still be included.

---

```
Agenda Item: SOC 2 Control Deficiency Review (Standing Item)

Time allocation: 10 minutes

Content:
1. Open P0 deficiencies: [list IDs and one-line descriptions, or "None"]
2. Open P1 deficiencies: [list IDs and one-line descriptions, or "None"]
3. Deficiencies closed since last call: [list IDs, or "None"]
4. Compensating controls accepted at current stage: [reference this document]
5. Any deficiency requiring advisor input or escalation: [describe or "None"]

Outcome: Advisors acknowledge receipt and note any concerns.
```

---

### Written Summary

Within 5 business days of the quarterly advisory call, the founder must file a written summary of the deficiency communication to `compliance/evidence/advisory-brief-YYYY-QN.md`. The summary must include:

- Date and attendees of the call (first name + role sufficient; no last names required)
- Open P0/P1 deficiency list as presented
- Any advisor questions, concerns, or action items raised
- Advisor acknowledgement (email confirmation from at least 1 advisor that they received and reviewed the summary)

The summary is filed as a git commit. The commit SHA is the attestation record.

### P0 Out-of-Cycle Notification

If a P0 deficiency is opened between quarterly calls, the founder must:

1. Email all advisors within 5 business days. Subject line: `[FORM SOC2] P0 Control Deficiency Opened — [Deficiency ID]`.
2. Include: deficiency description, root cause, compensating control in place, target remediation date.
3. File the email (or a summary of it) as `compliance/evidence/p0-notification-YYYY-MM-DD-[ID].md`.
4. Reference the notification file in `control-deficiency-log.csv` under `auditor_exhibit_ref` for the relevant row.

---

## Tier 3 — Investor Observer

### Purpose

FORM's seed-round investor observer (rights to be confirmed in term sheet per CC1-GAP-002 / CC2-GAP-002 remediation plan) receives a deficiency summary in every quarterly investor update. This creates a third, independent documentary record that control deficiencies are being communicated upward and tracked.

### Cadence

Quarterly investor update — cadence defined in `docs/INVESTOR_UPDATE_TEMPLATE.md`. Deficiency section must appear in every update, including updates where there are no open deficiencies (which is itself a positive signal for investors).

### Template Section

The following section must be included in every quarterly investor update. It slots into the investor update template immediately before or after the "Risks and Mitigations" section.

---

```
## SOC 2 Compliance Status

**Open P0 deficiencies:** [n] ([list IDs] or "None")
**Open P1 deficiencies:** [n] ([list IDs] or "None")
**Deficiencies closed this quarter:** [n] ([list IDs] or "None")
**SOC 2 readiness (self-assessed):** [X]%
**Observation period status:** [Not yet started / In progress — started YYYY-MM-DD / Complete]
**Audit firm:** [Not yet engaged / [Firm name] engaged — fieldwork begins [date]]

Key compliance events this quarter:
- [Bullet: e.g., "CC4-GAP-001 closed — control deficiency log artifact created and filed"]
- [Bullet: e.g., "First quarterly access review completed — no critical findings"]
- [Or: "No material compliance events this quarter."]

Next compliance milestone: [description and target date]
```

---

### Investor Observer Acknowledgement

The investor update is sent to the investor observer via email at the address designated in the term sheet. The sent email (or a forwarded confirmation) is filed as `compliance/evidence/investor-update-YYYY-QN-sent.txt` (plain text or PDF). This file is the auditor exhibit confirming Tier 3 execution.

---

## Execution Log

The following table tracks execution of all three tiers. The compliance-officer updates this table whenever a tier action is completed. This log is the primary auditor exhibit for CC4.2 compensating control operation.

| Period | Tier 1 Filed | Tier 1 Path | Tier 2 Call Date | Tier 2 Summary Path | Tier 3 Update Sent | Tier 3 Path |
|---|---|---|---|---|---|---|
| Pre-launch | 2026-05-20 | N/A — procedure created | N/A | N/A | N/A | N/A |
| [Month 1] | | | | | | |
| [Month 2] | | | | | | |
| [Q1] | | | | | | |

*Add rows as each period completes. This table must never be backdated.*

---

## SOC 2 CC4.2 Criterion Reference

CC4.2 of the AICPA Trust Services Criteria (2017, as updated) states:

> *The entity evaluates and communicates internal control deficiencies in a timely manner to those parties responsible for taking corrective action, including senior management and the board of directors or its equivalent.*

This procedure is FORM's documented compensating control for the absence of a formal board, accepted for the pre-Series-A stage. The three-tier structure ensures that:

- Every deficiency is reviewed by the founder (management) monthly (Tier 1)
- Every P0/P1 deficiency is communicated to external advisors quarterly or within 5 business days for P0s (Tier 2)
- All deficiencies are reported to investors quarterly (Tier 3)

The auditor will evaluate whether this compensating control is reasonably designed and whether execution evidence demonstrates it operated continuously across the observation period. The execution log above, the monthly memo artifacts, the advisory brief artifacts, and the investor update artifacts together constitute the complete evidence package for CC4.2.

---

**Auditor exhibit:** CC4-E-003

*Owner: founder. Effective: 2026-05-20. Expires: 90 days after Series A close. Reviewed: quarterly at advisory call.*
