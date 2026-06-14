# FORM · Project Status v5.8.3

> Live status board. Updated every iteration. Власник: `process-keeper`.

---

## Current version

**v5.8.3** — 2026-06-14

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
| 751–800 | Athlete lifecycle & context | 50 | **BLOCK COMPLETE** — all 50 posts written v5.1.0: posts 791 (athletes with movement limitations · clinical-safety CONDITIONAL PASS), 793 (sustained high stress training · clinical-safety CONDITIONAL PASS), 794 (medically-restricted diet · clinical-safety CONDITIONAL PASS) now written with embedded guard rails. Blog cards added for posts 785–800. sports-scientist review required for all 50 before publish |
| 801–850 | Evidence-based deep dives | 31 | **31/50 written** — posts 801–810: creatine, protein, caffeine, periodization vs. autoregulation, RPE/RIR, hypertrophy mechanisms, myofibrillar vs. sarcoplasmic, steroids/research interpretation, genetic ceiling, fat as endocrine organ. Posts 811–820: sleep & performance, deload science, training age, volume landmarks (MEV/MAV/MRV), inter-set rest, rate of force development, concurrent training interference, heat/cold exposure, altitude training, evidence-based stretching. Posts 821–830 (v5.8.0): protein timing, creatine loading revisited, beta-alanine, citrulline/nitrates, magnesium, omega-3 + training, Vitamin D + athletes, ashwagandha (CONDITIONAL PASS), sleep supplements melatonin/glycine (CONDITIONAL PASS), CBD & recovery (CONDITIONAL PASS). **Post 831 (v5.8.1):** antioxidants & training adaptation paradox — ROS signaling, AMPK/PGC-1α, Gomez-Cabrera 2008, Ristow 2009, Paulsen 2014; clinical-safety: PASS. All sports-scientist review required before publish. |

**Total posts: 827**

**Editorial series 11–30** — posts 26–29 blog cards added v5.8.3 (content existed since v5.8.2, no blog cards); post-30 written v5.8.3: «Що таке «довгострокові результати»» — training horizon, myofibrillar adaptation, tendon timelines, planning horizon problem; clinical-safety: PASS. Proposed next: posts 31–35 (mobility vs flexibility, reading research, self-programming vs coach, dropout mechanics, breathing mechanics). Open question: OQ-EDITORIAL-01 — filename collision with May 2026 short posts (post-22-periodization.md etc.) — status: 🟡 Unresolved.

---

## Enterprise documentation

| Document | Status | Last updated |
|---|---|---|
| SOC2_READINESS.md | Complete — 25 674 lines | v4.x |
| SSO_SCIM_IMPLEMENTATION.md | Complete — 9 283 lines | v4.x |
| INCIDENT_RESPONSE.md | Complete — 9 575 lines | v4.x |
| DATA_MODEL.md | Complete — 12 618 lines | v4.x |
| OBSERVABILITY.md | Complete — v4.0, §43 Webhook Delivery Observability added (WH-SLO-01–04, AL-WH-01–05, DDL 0074/0074b, pg_cron job 34) · OQ-WL-OBS-01 resolved DEC-054 | v5.1.1 |
| COST_MODEL.md | Complete — 8 341 lines | v4.x |
| SHARED_RESPONSIBILITY_MODEL.md | Complete — v1.0 | v4.18.0 |

---

## Open decisions (recent)

| ID | Summary | Status |
|---|---|---|
| DEC-054 | OQ-WL-OBS-01: `custom_domain_hash` SHA-256 for white_label DEC-030 payloads | Closed |
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

- **Block 751–800: COMPLETE** — all 50/50 posts written; posts 791, 793, 794 clinical-safety CONDITIONAL PASS (guard rails embedded); sports-scientist review required for all 50 before publish
- **Block 801–850 ACTIVE** — Evidence-based deep dives: 30/50 written. Batch 1 (801–810): creatine, protein, caffeine, periodization vs. autoregulation, RPE/RIR, hypertrophy mechanisms, myofibrillar vs. sarcoplasmic, steroids/research interpretation, genetic ceiling, fat as endocrine organ. Batch 2 (811–820): sleep & performance, deload science, training age, volume landmarks (MEV/MAV/MRV), inter-set rest, RFD, concurrent training interference, heat/cold exposure, altitude training, evidence-based stretching. **Batch 3 (821–830) COMPLETE v5.8.0:** protein timing, creatine loading revisited, beta-alanine, citrulline/nitrates, magnesium, omega-3+training, Vitamin D+athletes, ashwagandha (CONDITIONAL PASS), sleep supplements (CONDITIONAL PASS), CBD+recovery (CONDITIONAL PASS). **Next batch (831–840):** HMB evidence, caffeine tolerance and cycling, glutamine, zinc+magnesium (ZMA), collagen peptides, sodium bicarbonate buffering, nitrate timing, adaptogen comparison, rhodiola rosea, electrolytes and performance
- Review posts 586–632 (sports-scientist pass before publish — blocks research literacy, training with tech, programming edge cases)
- newsletter-04.md — **READY TO SEND** (brand-voice editorial pass complete · v4.36.0)
- newsletter-05.md — **READY TO SEND** (brand-voice editorial pass complete · v4.65.2 · pending subscription link substitution only)
- PostHog DPA review (compliance-officer) — **REVIEW COMPLETE**; click-through execution pending (founder action — PostHog Dashboard → Organization → Privacy & DPA); PIA-2026-001 filed
- Block 601–650: **COMPLETE** — всі пости написані (639 blocked HARD VETO); sports-scientist review needed перед publish
- Block 651–700: **COMPLETE** — all 50 posts written; 659–660 clinical-safety CONDITIONAL PASS; sports-scientist review required before publish
- Block 851–900 proposed: Applied programming deep dives (added to README roadmap v4.52.0)
- Block 1001–1050 proposed: Training literacy: reading your own data (added to README roadmap v4.66.0)

---

**v0.1 · червень 2026 · process-keeper · update every iteration · v5.3.0**
