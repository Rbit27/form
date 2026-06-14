# FORM · Project Status v4.95.0

> Live status board. Updated every iteration. Власник: `process-keeper`.

---

## Current version

**v4.99.0** — 2026-06-14

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
| 651–700 | Biomechanics & injury prevention | 50 | **BLOCK COMPLETE** — all 50 posts written; posts 659–660 clinical-safety CONDITIONAL PASS; sports-scientist review required before publish |
| 701–750 | Strength sports specifics | 50 | **BLOCK COMPLETE** — posts 705–706 clinical-safety CONDITIONAL PASS/PASS; posts 743–750 completed this iteration (landmine exercises, paused bench press, TnG vs. dead stop deadlift, seasonal programming, weightlifting for powerlifters, cross-discipline base, competition day execution, block synthesis); all sports-scientist review required before publish |
| 751–800 | Athlete lifecycle & context | 33 | In progress — posts 751–783 written except gaps noted: 751–780 written (desk-worker, shift-worker, masters 40+, return after break, travel training, novice phase, intermediate transition, female athlete considerations, time-constrained training, high-volume vs. life stress, new parent training, training after 50, student during exams, return after viral illness, hybrid athlete concurrent training, entrepreneur-athlete irregular schedule, strength for runners, hotel training, training without gym, youth strength training [clinical-safety CONDITIONAL PASS], when self-coaching ends [clinical-safety PASS], where motivation ends — programming vs psychology [clinical-safety PASS after required edits], chronic dehydration training, team sport + strength training, recreational athlete & season, powerlifting competition return, chronic sleep restriction training [clinical-safety PASS], academic stress training — allostatic load [clinical-safety PASS], entrepreneur-athlete at business scale — allostatic load [clinical-safety PASS], ME/CFS і пост-вірусний стан — clinical-safety PASS); post-781 written (тренінг після 60 — sarcopenia/type-II-fiber atrophy, motor unit loss/RFD, anabolic resistance 40g threshold, orthopaedic adaptations/OA, programming 3–4×/week full-body, RIR 1–2, MD-referral gate); post-782 written (тренінг під час перименопаузи — estrogen anabolic role, collagen/tendon injury risk, bone density axial loading ≥70% 1RM, vasomotor thermoregulation, sleep disruption, programming higher frequency + eccentric tempo 3–4s, MD-referral gate); post-783 written (strength for amateur triathletes — concurrent training, AMPK/mTOR interference, seasonal programming); posts 784–800 pending; sports-scientist review required before publish |

**Total posts: 778**

**Editorial series 11–16** — post-16 written (overtraining vs underrecovery · clinical-safety PASS); blog.html cards added for posts 11–16; post-17 (AI-coach vs PT) proposed in README.

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
| DEC-051 | OQ-SSO-28.2: tenant_users_role_history retention 7yr | Closed |
| DEC-050 | OQ-TURH-01+02: form_role_enum & role_change_source_enum ENUM types | Closed |
| DEC-049 | OQ-SSO-27.2: role change audit trail — separate table approach | Closed |
| DEC-048 | OQ-SSO-27.3: SCIM KV failure fallback — option (c) + AL-REVOKE-01 | Closed |
| DEC-047 | Vanta selected over Drata (PRE-25 pathway) | Closed |
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

- **Block 701–750: COMPLETE** — всі 50 постів написані; sports-scientist review required before publish
- **Block 751–800: IN PROGRESS** — posts 751–783 written (33/50); next: post-784 (функціональний фітнес vs. силовий тренінг: методологічна різниця); sports-scientist review required before publish
- Review posts 586–632 (sports-scientist pass before publish — blocks research literacy, training with tech, programming edge cases)
- newsletter-04.md — **READY TO SEND** (brand-voice editorial pass complete · v4.36.0)
- newsletter-05.md — **READY TO SEND** (brand-voice editorial pass complete · v4.65.2 · pending subscription link substitution only)
- PostHog DPA review (compliance-officer) — **REVIEW COMPLETE** (scope analysis + Art.9 ruling documented · `compliance/dpa/dpa-collection-checklist-posthog-2026-06-13.md`); click-through execution pending (founder action — PostHog Dashboard → Organization → Privacy & DPA); PIA-2026-001 filed v4.x
- Block 601–650: **COMPLETE** — всі пости написані (639 blocked HARD VETO); sports-scientist review needed перед publish
- Block 651–700: **COMPLETE** — all 50 posts written; 659–660 clinical-safety CONDITIONAL PASS; sports-scientist review required before publish
- Block 851–900 proposed: Applied programming deep dives (added to README roadmap v4.52.0)
- Block 1001–1050 proposed: Training literacy: reading your own data (added to README roadmap v4.66.0)

---

**v0.1 · червень 2026 · process-keeper · update every iteration · v4.96.0**
