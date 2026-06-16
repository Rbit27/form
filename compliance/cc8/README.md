# FORM · CC8 Change Management — Evidence Index

> SOC 2 Type II · Trust Service Criteria CC8 (Change Management).
> Owner: devops-lead + security-engineer. Review: quarterly.

---

## CC8 Criteria Coverage

| Criterion | Description | Status |
|---|---|---|
| CC8.1 | Changes authorized, designed, developed, configured, documented, tested, approved, implemented | 🟢 Policy defined — `change-management-policy.md` |

---

## Evidence Files

| File | Description | SOC 2 Criteria | Collection Cadence |
|---|---|---|---|
| `change-management-policy.md` | Change Management & SDLC Policy v1.0 (POL-011) | CC8.1 | Annual review; update on architecture change |
| `staging-data-anonymisation-procedure.md` | Staging Data Anonymisation Procedure v1.0 (POL-CC8-ANON-01) — four-step quarterly staging refresh before k6 Cloud EU-West load-test runs; ANON-01–ANON-05 invariants; clinical-safety sign-off gate; PERF-STAGING-E-001 evidence artefact | CC4.1, A1.1, CC8.1 | Quarterly (before each PERF-SLO-06 run); annual policy review |
| `change-log.csv` | _(create at M4 deploy)_ Log of all production changes tagged `system.deploy` per DEC-030 | CC8.1 | Continuous; export quarterly |
| `emergency-change-log.csv` | _(create at M4 deploy)_ Log of all emergency changes with retroactive approval records | CC8.1 | On each emergency change; export quarterly |
| `branch-protection-screenshot.png` | _(collect at M3 hardening)_ Screenshot of GitHub branch protection rules for `main` | CC8.1 | At each rule change |
| `ci-pipeline-config.md` | _(create at M3 hardening)_ Snapshot of required CI checks; updated when pipeline changes | CC8.1 | At each pipeline change |
| `quarterly-change-review.md` | _(template)_ Quarterly review: change volume, emergency changes, CI failure rate, rollback count | CC8.1 | Quarterly |

---

## Gap Status

| Item | Gap | Closed by |
|---|---|---|
| Change management process (code review, CI gates) | 🟡 → 🟢 | `change-management-policy.md` |
| Production deploy requires CI pass | 🟡 → 🟢 | `change-management-policy.md` §5 |
| Emergency change process | 🟡 → 🟢 | `change-management-policy.md` §7 |
| Rollback capability documented | 🟡 → 🟢 | `change-management-policy.md` §8 |

---

## Compliance Calendar

| Quarter | Task | Artefact |
|---|---|---|
| Q1/Q3 | Quarterly change review — volume, emergency count, CI failure rate, rollbacks | `quarterly-change-review-YYYYQQ.md` |
| Annual | Policy review + approval re-sign | Updated `policy-approval-log.csv` row |
| On each emergency change | Complete emergency change log entry within 24 h | `emergency-change-log.csv` row |

---

*v1.0 · 2026-06-07 · Owner: devops-lead*
