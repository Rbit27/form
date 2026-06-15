# FORM · Project Status v5.26.0

> Live status board. Updated every iteration. Власник: `process-keeper`.

---

## Current version

**v5.26.0** — 2026-06-15

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
| 801–850 | Evidence-based deep dives | 50 | **BLOCK COMPLETE v5.14.0** — all 50 posts written. Posts 801–840 (prev batches). **Batch 841–850 COMPLETE v5.14.0:** post-841 electrolytes performance science (PASS), post-842 nitrate/beetroot (PASS — guard rail: drug interactions with nitroglycerin), post-843 sodium bicarbonate buffering (PASS), post-844 adaptogen comparison (PASS), post-845 rhodiola rosea (CONDITIONAL PASS — guard rail: SSRI/SNRI/MAOI interactions, serotonin syndrome risk), post-846 tart cherry/recovery (PASS), post-847 glutathione/antioxidant paradox (PASS), post-848 NAC (CONDITIONAL PASS — guard rail: nitroglycerin interaction + antioxidant paradox for trained athletes), post-849 phosphatidylserine/cortisol (PASS), post-850 cordyceps/C.militaris (PASS — block synthesis). All sports-scientist review required before publish. |

| 851–900 | Applied programming deep dives | 50 | **BLOCK COMPLETE v5.24.0** — all 50 posts written; blog.html caught up to post-900; sports-scientist review required for all 50 before publish |
| 901–950 | Advanced programming systems | 40/50 | **IN PROGRESS v5.31.0** — posts 901–940 written. 901–930: (see v5.29.0). 931–940: advanced deload variations, block vs. DUP periodization, autoregulation tools (RPE/RIR/VBT/APRE), training around soreness (DOMS), exercise selection advanced (3-level hierarchy), progressive overload beyond weight (7 vectors), accumulation vs. intensification, taper for non-competitive athletes, advanced warm-up protocols (PAP), training log analysis. clinical-safety NOT REQUIRED. sports-scientist review required before publish. blog.html updated to post-940. |

**Total posts: 940**

**Editorial series 11–34, 44–46** — post-46 written v5.26.0: «Форс-мажор у тренуванні: адаптація без паніки і без нульового місяця» — мінімальний ефективний доз при порушенні режиму; деталізований розбір по 4 сценаріях (робочий дедлайн, відрядження, легке нездоров'я, виснаженість); таймлайн детренування без паніки; протокол повернення після паузи 60-70-80-90-100%; clinical-safety: PASS. Blog card added. Open question: OQ-EDITORIAL-01 — filename collision with May 2026 short posts (post-22-periodization.md etc.) — status: 🟡 Unresolved.

---

## Enterprise documentation

| Document | Status | Last updated |
|---|---|---|
| SOC2_READINESS.md | Complete — 25 674 lines | v4.x |
| SSO_SCIM_IMPLEMENTATION.md | Complete — §31 added (OQ-SSO-24.1+24.2 resolved DEC-060, pam-db-proxy architecture) · v2.3 | v5.13.1 |
| INCIDENT_RESPONSE.md | Complete — 9 575 lines | v4.x |
| DATA_MODEL.md | Complete — 12 618 lines | v4.x |
| OBSERVABILITY.md | Complete — v4.0, §43 Webhook Delivery Observability added (WH-SLO-01–04, AL-WH-01–05, DDL 0074/0074b, pg_cron job 34) · OQ-WL-OBS-01 resolved DEC-054 | v5.1.1 |
| COST_MODEL.md | Complete — 8 341 lines | v4.x |
| SHARED_RESPONSIBILITY_MODEL.md | Complete — v1.0 | v4.18.0 |

---

## Open decisions (recent)

| ID | Summary | Status |
|---|---|---|
| DEC-060 | OQ-SSO-24.1+24.2: pam-db-proxy — Supabase Edge Function + port 5432 direct session connection | Closed |
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
- **Block 801–850: COMPLETE v5.14.0** — all 50/50 written. Batch 841–850 complete: electrolytes, nitrate/beetroot, sodium bicarb, adaptogen comparison, rhodiola (CONDITIONAL PASS), tart cherry, glutathione, NAC (CONDITIONAL PASS), phosphatidylserine, cordyceps. Sports-scientist review required for all 50 before publish
- Review posts 586–632 (sports-scientist pass before publish — blocks research literacy, training with tech, programming edge cases)
- newsletter-04.md — **READY TO SEND** (brand-voice editorial pass complete · v4.36.0)
- newsletter-05.md — **READY TO SEND** (brand-voice editorial pass complete · v4.65.2 · pending subscription link substitution only)
- PostHog DPA review (compliance-officer) — **REVIEW COMPLETE**; click-through execution pending (founder action — PostHog Dashboard → Organization → Privacy & DPA); PIA-2026-001 filed
- Block 601–650: **COMPLETE** — всі пости написані (639 blocked HARD VETO); sports-scientist review needed перед publish
- Block 651–700: **COMPLETE** — all 50 posts written; 659–660 clinical-safety CONDITIONAL PASS; sports-scientist review required before publish
- Block 851–900: **COMPLETE 50/50** — all posts written (v5.24.0); blog.html caught up to post-900; sports-scientist review pending for all 50 posts before publish
- Block 901–950: **IN PROGRESS 40/50 · v5.31.0** — posts 901–940 written. blog.html updated to post-940. Next: posts 941–950 (proposed: neural adaptations to strength training, tendon and connective tissue adaptation, training age and response rate, advanced RPE programming, fatigue management across a training year, training for longevity, hormonal responses to training, sleep and training quality, cognitive fatigue and training decisions, the minimum effective dose)
- Block 1001–1050 proposed: Training literacy: reading your own data (added to README roadmap v4.66.0)

---

**v0.1 · червень 2026 · process-keeper · update every iteration · v5.29.0**
