# FORM · Project Status v4.41.0

> Live status board. Updated every iteration. Власник: `process-keeper`.

---

## Current version

**v4.58.0** — 2026-06-13

---

## Content library

| Range | Theme | Posts | Status |
|---|---|---|---|
| 01–10 | Foundational / brand | 10 | Published |
| 11–50 | Core sport science | 40 | Published |
| 51–99 | Applied training science | 49 | Published |
| 100–200 | Exercise physiology deep dives | 101 | Published |
| 201–350 | Training methodology & programming | 150 | Published |
| 351–500 | Self-coached athlete practical | 150 | Published |
| 501–559 | Self-programming mastery | 59 | Published |
| 560–585 | Sport psychology block | 26 | Published |
| 586–590 | Research literacy block | 5 | Draft — review_pending: sports-scientist |
| 591–600 | Training with tech block | 10 | Draft — review_pending: sports-scientist · **BLOCK COMPLETE** |
| 601–650 | Programming edge cases | 50 | **BLOCK COMPLETE** — 49/50 written (post-639 clinical-safety HARD VETO; all others done); sports-scientist review required before publish |
| 651–700 | Biomechanics & injury prevention | 4 | In progress — posts 651–654 written; sports-scientist review required before publish |

**Total posts: 654**

---

## Enterprise documentation

| Document | Status | Last updated |
|---|---|---|
| SOC2_READINESS.md | Complete — 25 674 lines | v4.x |
| SSO_SCIM_IMPLEMENTATION.md | Complete — 9 283 lines | v4.x |
| INCIDENT_RESPONSE.md | Complete — 9 575 lines | v4.x |
| DATA_MODEL.md | Complete — 12 618 lines | v4.x |
| OBSERVABILITY.md | Complete — v3.6, §39 Backup Integrity/DR Readiness added, pg_cron job 29 | v4.24.2 |
| COST_MODEL.md | Complete — 8 341 lines | v4.x |
| SHARED_RESPONSIBILITY_MODEL.md | Complete — v1.0 | v4.18.0 |

---

## Open decisions (recent)

| ID | Summary | Status |
|---|---|---|
| DEC-046 | OQ-33: readiness_bucket CONDITIONAL PASS; mood_bucket VETO | Closed |
| DEC-045 | OQ-RL-02 + OQ-BILL-01 resolved | Closed |
| DEC-044 | OQ-BILL-05: split-column GDPR pseudonymization | Closed |
| DEC-043 | OQ-PAM-02: business_justification SOC2 handling | Closed |
| DEC-042 | OQ-PAM-01: break-glass alert-on-access | Closed |
| DEC-041 | Enterprise AI coaching flat per-seat billing | Closed |
| DEC-040 | Feature flag lifecycle: downgrade auto-disable + 90-day deprecation | Closed |

---

## Site pages

| Page | Status |
|---|---|
| index.html | Live |
| enterprise.html | Live |
| pricing-enterprise.html | Live |
| pricing.html | Live |
| security.html | Live |
| admin-dashboard.html | Live |
| blog.html | Live |

---

## Active constraints (HARD)

- `clinical-safety` HARD VETO on all food/body/pain/mental health content
- Audit log: append-only HMAC-chained (DEC-030)
- No social features in app (DEC-002)
- Streak grace after 2 misses, not 1 (DEC-013)
- Editorial-brutalist tone, no «дисципліна — свобода» (DEC-007, DEC-014)
- External sharing: clinical-safety gate (DEC-029)

---

## Next priorities

- Review posts 586–632 (sports-scientist pass before publish — blocks research literacy, training with tech, programming edge cases)
- newsletter-04.md — **READY TO SEND** (brand-voice editorial pass complete · v4.36.0)
- newsletter-05.md — **DRAFT COMPLETE** (training-with-tech synthesis, posts 591–600; brand-voice editorial pass pending)
- PIA filing for readiness_bucket PostHog (compliance-officer — DEC-046 pre-condition)
- PostHog DPA review (compliance-officer — DEC-046 pre-condition)
- Block 601–650: **COMPLETE** — всі пости написані (639 blocked HARD VETO); sports-scientist review needed перед publish
- Block 651–700: posts 651–654 written (kinetic chain, mobility/stability framework, ankle mobility, squat anatomy). Next: posts 655–660 — bench biomechanics, deadlift mechanics, core stability, breathing patterns, shoulder complex (659 — clinical-safety review required before writing), rehab-to-performance (660 — clinical-safety review required before writing)
- newsletter-05.md — brand-voice editorial pass pending before send
- Block 851–900 proposed: Applied programming deep dives (added to README roadmap v4.52.0)

---

**v0.1 · червень 2026 · process-keeper · update every iteration · v4.58.0**
