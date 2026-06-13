# FORM · Project Status v4.41.0

> Live status board. Updated every iteration. Власник: `process-keeper`.

---

## Current version

**v4.51.0** — 2026-06-13

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
| 601–650 | Programming edge cases | 38 | Draft — posts 637–638 added (block continues, 38/50) |

**Total posts: 638**

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
- Block 601–650 continues: posts 639–650 remaining (12 posts); **next: тренування при хронічному болю (639) — clinical-safety HARD VETO required before write; do not write without explicit review**
- Block 651–700 planning: biomechanics & injury prevention (first topic: chronic pain — **clinical-safety review required before writing**)

---

**v0.1 · червень 2026 · process-keeper · update every iteration · v4.51.0**
