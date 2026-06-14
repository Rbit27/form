# FORM · Project Status v5.0.0

> Live status board. Updated every iteration. Власник: `process-keeper`.

---

## Current version

**v5.0.0** — 2026-06-14

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
| 751–800 | Athlete lifecycle & context | 47 | **47/50 written** — posts 784–790, 792, 795–800 added v5.0.0: functional fitness vs strength training (SAID principle · clinical-safety PASS), shoulder mobility training adaptations, hip mobility training adaptations, chronic recovery deficit strategy, first year post-university schedule, irregular work schedule training, self-coaching year two, Masters 40–70 planning spectrum, CrossFit-to-strength-training transition, martial arts concurrent scheduling, frequent travel + jet lag strategy, budget training without gym, self-coached vs coached synthesis (post-799), block synthesis (post-800). **BLOCKED (3 posts, clinical-safety required before write):** post-791 (athletes with disabilities), post-793 (training under chronic work stress), post-794 (medically-restricted diet). sports-scientist review required for all written posts before publish |

**Total posts: 792**

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

- **Block 751–800: 47/50 written** — posts 791, 793, 794 BLOCKED pending clinical-safety (disability training, chronic work stress, medically-restricted diet); sports-scientist review required for all 47 before publish
- **Block 801–850 NEXT** — Evidence-based deep dives: creatine, protein, caffeine, fat, periodization vs. autoregulation, RPE/RIR, hypertrophy mechanisms, steroids and natural training, genetic ceiling, myofibrillar vs. sarcoplasmic hypertrophy
- Clinical-safety gate needed: posts 791, 793, 794 — clinical-safety agent review required before writing
- Review posts 586–632 (sports-scientist pass before publish — blocks research literacy, training with tech, programming edge cases)
- newsletter-04.md — **READY TO SEND** (brand-voice editorial pass complete · v4.36.0)
- newsletter-05.md — **READY TO SEND** (brand-voice editorial pass complete · v4.65.2 · pending subscription link substitution only)
- PostHog DPA review (compliance-officer) — **REVIEW COMPLETE**; click-through execution pending (founder action — PostHog Dashboard → Organization → Privacy & DPA); PIA-2026-001 filed
- Block 601–650: **COMPLETE** — всі пости написані (639 blocked HARD VETO); sports-scientist review needed перед publish
- Block 651–700: **COMPLETE** — all 50 posts written; 659–660 clinical-safety CONDITIONAL PASS; sports-scientist review required before publish
- Block 851–900 proposed: Applied programming deep dives (added to README roadmap v4.52.0)
- Block 1001–1050 proposed: Training literacy: reading your own data (added to README roadmap v4.66.0)

---

**v0.1 · червень 2026 · process-keeper · update every iteration · v5.0.0**
