# Changelog · FORM

## [4.41.0] — 2026-06-13

### Added
- `content/post-624-paused-reps-dead-stop-technique.md` — Programming edge cases block, post 24/50. Паузні повторення і dead-stop техніка: механічна логіка (SSC removal, TUT at sticking point, motor control), per-lift sections (паузний присід, паузний жим, dead-stop deadlift) з load ranges (%1RM + RPE anchors), типові помилки програмування (занадто важко, занадто багато об'єму, indefinite replacement), три практичні шаблони інтеграції з week-by-week прогресією. Sources: Sorensen 2016, Schoenfeld 2017, Helms 2016, Flanagan & Comyns 2008.

### Changed
- `STATUS.md` — content table updated: post-624 added, total posts 622 → 623; block 601–650 progress 22/50 → 23/50.
- `VERSION` — 4.40.1 → 4.41.0.

---

## [4.40.1] — 2026-06-13

### Added
- `docs/COST_MODEL.md §37` — Enterprise Pipeline Health & ARR Forecasting Model. Six-stage pipeline (S0 Inbound → S5 Closed-Won) with entry/exit criteria, maximum stage age, and required escalation actions. Stage conversion rate table (conservative/base/optimistic; benchmarked against OpenView Partners 2024 and Gartner wellness 2023). Three pipeline health metrics: PCR (3.0× healthy threshold, stage weights 5–100%), sales velocity formula (Y1 $1,944/mo; Y2 $9,164/mo), and aging triggers. Y1–Y3 ARR build table (base case + downside) with quarterly granularity. Monthly ARR bridge reconciliation (new + expansion − contraction − churn + indexation) with NRR floor thresholds. Forecasting governance cadence (weekly / monthly / quarterly) with forecast override policy and audit trail requirement. `enterprise_pipeline_stages` Postgres table DDL with RLS; four SQL queries (PCR, conversion actuals, aging alert, bridge). Four DEC-030 HMAC-chained events: `enterprise.pipeline_reviewed` (STANDARD, 3yr), `enterprise.arr_bridge_closed` (STANDARD, 7yr — Ukrainian Tax Code Art. 44 basis), `enterprise.deal_aged_out` (STANDARD, 3yr — pg_cron job 31), `enterprise.pipeline_conversion_model_recalibrated` (STANDARD, 7yr). 10-item checklist (4× P0/M7–M8, 3× P1/M9–M10, 3× P2/M18–M22). 3 open questions: OQ-PIPE-01 (actual conversion rates after Deal 10), OQ-PIPE-02 (CRM adoption decision), OQ-PIPE-03 (partner channel attribution). Privacy floor: no individual employee user_id in any §37 DEC-030 event; deal_id is FORM-internal UUID only.

### Changed
- `docs/COST_MODEL.md` — version header v2.2 → v2.3; TOC updated to include §37.
- `VERSION` — 4.40.0 → 4.40.1.

---

## [4.40.0] — 2026-06-13

### Added
- `content/post-622-return-to-accumulation-after-peak.md` — Programming edge cases block, post 22/50. "Піковий блок закінчено. Що тепер?" — post-peak transition and return-to-accumulation planning. Covers: transition week protocol (50–65% intensity, 2–3 working sets, psychological reset — not a deload, not training); residual training effects by Issurin (2008): strength quality persists ~30 ± 5 days after ending specific stimulus; why new tested 1RM is the only correct percentage base, with two sub-scenarios for below-expectation tests; volume cycling logic — why accumulation restarts at MEV not MAV (fitness-fatigue model, Banister et al. 1975); 8–10-week mesocycle structure post-peak (transition week + phase 1 MEV→MAV + phase 2 peak-accumulation + deload); two mistakes after successful peak ("riding the wave" at high intensity without volume foundation; staying in peak mode 2–3 extra weeks while fitness decays); one mistake after failed peak (immediate retest); transfer journal format (what to document about test-day and peak block mechanics while memory is fresh). Citations: Issurin 2008 (residual training effects, block periodization), Banister et al. 1975 (fitness-fatigue model), Hakkinen et al. 1985 (neural detraining), Plisk & Stone 2003 (accumulation-realization structure). ~1900 words, Ukrainian, editorial-brutalist. Clinical safety: PASS (no food/body-image/injury/mental health content).

### Changed
- `blog.html` — added cards for posts 619, 620, 621, 622 above post-618 card.
- `STATUS.md` — block 601–650: 21 → 22 posts; total 621 → 622; next-priorities line updated; version v4.39.0 → v4.40.0.
- `VERSION` — 4.39.0 → 4.40.0.
- `README.md` — Block 601–650 progress updated; new continued topics added for posts 622–650.

---

## [4.39.0] — 2026-06-13

### Added
- `content/post-619-autoregulation-anchor-points.md` — Programming edge cases block, post 19/50. "Калібрування RPE: якорні сесії і як підтримувати точність внутрішньої шкали." Direct continuation of post-616 (autoregulation basics). Covers calibration drift mechanics (systematic under-call in progressing athletes, over-call under fatigue); three anchor-point session types (AMRAP-anchor, near-max/1RM test, external-confirmation set with video review); four signals of drift (AMRAP doesn't match logged RPE, 1RM test doesn't match training trajectory, identical loads RPE trend without external cause, regular failure at predicted RIR 2–3); practical calibration frequency (AMRAP every 4–6 weeks, near-max every 10–16 weeks, mandatory after break or block transition); calibration log format; per-block workflow (first week post-deload = anchor, final week = comparison). Citations: Helms et al. 2016 (±1 RIR accuracy under calibrated conditions), Zourdos et al. 2016 (modified RPE scale, JSCR), Hackett et al. 2012 (systematic overestimation of proximity to failure), Helms et al. 2018 (RPE vs VBT accuracy), Tuchscherer/RTS (AMRAP as diagnostic unit, practitioner manual 2008). ~1900 words, Ukrainian, editorial-brutalist. Clinical safety: not_required.
- `content/post-621-peaking-for-1rm.md` — Programming edge cases block, post 21/50. "Пікування на максимум: структура блоку, вибір спроб і виконання в день тесту." Distinguishes peaking (block structure to maximize strength expression on target date) from tapering (fatigue management in final weeks). Fitness-fatigue model framing: fitness t½ ~42 days, fatigue t½ ~13 days (Banister et al. 1975; Haff & Triplett 2016 Essentials S&C). Three peak block lengths with use cases: 3-week (experienced peakers, fast fatigue responders), 4-week (standard for intermediate/advanced after 8–10-week accumulation block, most universal), 6-week (high-volume blocks, embedded mini-deload in week 3). Volume/intensity trajectory with week-by-week specifics. Opener selection: opener = comfortable 97%, second = PR target, third = stretch if going well — with solo-day correction (+5% not +3% due to absent crowd energy). 72-hour rule (last heavy session ≥72h before test, Grgic et al. 2018), 48-hour activation rule (lighter potentiation session), PAP timing 3–7 minutes (Tillin & Bishop 2009, Sports Medicine). Warm-up ramp protocol (bar → 50% → 65% → 75% → 85% → 93% → opener). Four systematic errors: too-long peak (neural detraining begins ~2–3w without stimulus, Hakkinen et al. 1985), too-short peak, heavy session in final week, intensity reduction instead of volume reduction (Murach & Bagley 2015, S&C Journal). Solo vs. competition differences (rack safeties, conservative opener, warm-up time budget). ~1950 words, Ukrainian, editorial-brutalist. Clinical safety: not_required.

### Changed
- `content/post-620-high-low-responder-identification.md` — revised by sports-scientist pass: stronger editorial-brutalist H1 ("Ти не знаєш, як твоє тіло реагує на тренування. І це проблема"), cleaner opening paragraph that skips the already-known Hubal baseline and jumps to the practical application gap. Slug updated to `high-low-responder-identification-n-of-1`. Content structure and citations unchanged.
- `STATUS.md` — block 601–650: 20 → 21 posts; total 620 → 621; version v4.38.0 → v4.39.0.
- `VERSION` — 4.38.0 → 4.39.0.

---

## [4.38.0] — 2026-06-13

### Added
- `content/post-620-high-low-responder-identification.md` — Programming edge cases block, post 20/50. "Ідентифікація high/low responder — практичний n-of-1 підхід для самокерованого атлета." Builds on post-589 (Hubal et al. 2005 variability data) and post-618 (myonuclear memory). Covers: responder status as dose-response curve position (not fixed ceiling), citing Bamman et al. 2007 on differential gene regulation (IGF-1/myostatin pathway) in high/middle/low tertiles; three low-responder signals (stagnation at consistent progressive overload, strength PR without hypertrophy = neurally-dominant profile, no size change after 12+ weeks); three high-responder signals (fast strength trajectory, visible hypertrophy 8–12wk with new stimulus, fast detraining recovery); structured 8-week n-of-1 test protocol (single variable change, pre/post circumference + strength + rep-max progression); programming implications (low: 16–20+ sets/week, high frequency 3–4x; high: 6–10 sets, standard-to-high frequency with overreaching risk); why this is not "genetics determines everything" — it's dose calibration. Summary table (6 dimensions). ~2000 words, Ukrainian, editorial-brutalist. Clinical safety: not_required.

### Changed
- `STATUS.md` — block 601–650: 18 → 20 posts (post-619 placeholder noted); total 618 → 620; version bump.
- `VERSION` — 4.37.1 → 4.38.0.

---

## [4.37.1] — 2026-06-13

### Changed
- `docs/DATA_MODEL.md` — v1.9 → v1.10. Added §31 DSAR Request Registry Schema (Art. 15/17/20 lifecycle tracking). Supplies the DDL for the `dsar_requests` table referenced by INCIDENT_RESPONSE.md R-14 SQL queries and SOC2_READINESS.md P4.0 evidence — closes a genuine cross-reference gap that existed since R-14 was written. §31 includes: `dsar_request_type` + `dsar_status` enums; full column DDL (SLA timeline with `deadline_at` + `extended_deadline_at` GENERATED columns, export lifecycle, erasure lifecycle, refusal, DPA notification, HMAC chain); 5 indexes (open-deadline, tenant, user, export-pending, retention); 2 triggers (updated_at, closed_at auto-set on terminal transition); RLS policies for form_system (full), form_user (self-read), form_tenant_admin (id/request_type/status/deadline only — privacy floor enforced via column-level REVOKE/GRANT), form_admin (break-glass, DEC-042); 14 DEC-030 event registrations with Zod schema skeletons aligned to INCIDENT_RESPONSE.md R-14 §14.7; 3 alerting rules (AL-DSAR-01 5-day P1 PagerDuty, AL-DSAR-02 overdue P0, AL-DSAR-03 undelivered export P2 Slack); 6 SOC 2 evidence artefacts (DSAR-E-001 through DSAR-E-006) mapped to P4.0/P5.0/P5.1/P8.0/CC2.2/CC6.5; 11-item implementation checklist (7× P0 M4, 3× P1 M5, 1× P2 M8); 2 open questions (OQ-DSAR-01 CCPA 45-day deadline, OQ-DSAR-02 retain_until GENERATED column behaviour post-erasure). Also updated TOC to include §28–§31 entries (was previously truncated at §27).
- `VERSION` — 4.37.0 → 4.37.1.

---

## [4.37.0] — 2026-06-13

### Added
- `content/post-618-detraining-rates-muscle-memory.md` — Programming edge cases block, post 18/50. "Розтренованість — як швидко зникає те, що ти будував, і що це означає для програмування." Covers detraining rates across three adaptation categories (neural: begins 1–2 weeks; structural/hypertrophy: resilient through 2–3 weeks per Ogasawara et al. 2013; aerobic: starts declining at 10–12 days). Core mechanism: myonuclear retention (Bruusgaard et al. 2010) — nuclei persist after atrophy, enabling faster retraining than original timeline. Maintenance dose: 1/3 of volume at maintained intensity preserves most adaptations (Bickel et al. 2011). Practical return-from-break map: 1–2 weeks (minor volume reduction), 2–4 weeks (reintegration week at 60–70%), 4–8 weeks (return block 3–6 weeks), 12+ weeks (full return meso). Two common errors: starting from zero and returning to full load immediately. Clinical safety: not_required — no food/body-image/injury/mental health content. Sports-scientist reviewed. ~1750 words, Ukrainian, editorial-brutalist tone.

### Changed
- `blog.html` — картка post-618 додана на початок grid.
- `STATUS.md` — block 601–650 updated: 17 → 18 posts; total 617 → 618; version v4.36.0 → v4.37.0.
- `VERSION` — 4.36.0 → 4.37.0.

---

## [4.36.0] — 2026-06-13

### Added
- `content/post-616-autoregulation-in-practice.md` — Programming edge cases block, post 16/50. "Авторегуляція на практиці — коли RPE і RIR кращі за фіксовані відсотки, і коли ця система ламається." Covers daily strength variability (±5–10% is normal), RPE/RIR definitions and practical calibration, when autoregulation outperforms fixed percentages (intermediate+, trained athletes), when it fails (beginners can't rate RPE accurately, motivated athletes systematically underestimate, no anchor points without 1RM history), and practical log-based implementation. References Connell & Wilson 2021 on RPE-based vs fixed-% groups. ~1533 words, Ukrainian, editorial-brutalist. Clinical safety: not_required.
- `content/post-617-reactive-vs-scheduled-deload.md` — Programming edge cases block, post 17/50. "Реактивний vs запланований деload — чим вони різняться і чому більшість атлетів ламають обидва." Defines two distinct types (scheduled: predetermined structural check; reactive: symptom-triggered), argues for scheduled for beginners (can't self-diagnose fatigue accurately), reactive awareness for intermediate+, and primarily reactive for advanced with structural checkpoints. Decision criteria: 3 of 4 triggers (RPE trend +0.5–1, motivation drop, sleep quality, resting HR +2–3 bpm). Covers the "I don't feel tired" fallacy as hidden cumulative fatigue. References Faigenbaum & Kraemer 2000. ~1500 words, Ukrainian, editorial-brutalist. Clinical safety: not_required.

### Changed
- `content/newsletter-04.md` — editorial review finalized. brand-voice pass complete: `ready_to_send`. Recommended subject: "Everything you read about training is right. None of it is about you." Recommended preview: "Research tells you what works on average. Your log tells you what works for you." Tone check: pass (clean editorial-brutalist, no motivational clichés, no дисципліна=свобода framing).
- `VERSION` — 4.35.1 → 4.36.0.

---

## [4.35.1] — 2026-06-13

### Changed
- `enterprise.html` — three additions to the enterprise landing page (v1.0 → v1.1, created at v4.8.1, first update since). (1) **Shared responsibility model badge** added to the compliance posture grid (7th badge, purple accent, after Break-glass access control) — surfaces `docs/SHARED_RESPONSIBILITY_MODEL.md` (added at v4.18.0) as a visible signal to enterprise procurement teams. (2) **"Due diligence package" section** inserted between Data portability and FAQ — four resource cards linking to `/security.html`, `/privacy.html`, `/subprocessors.html`, and a "Shared responsibility model · Available on request" card. All three pages existed before but were unreachable from the enterprise landing page. Procurement and legal teams doing diligence need these links within one click. (3) **Footer navigation expanded** from 3 items to 6 — adds Security, Privacy, Subprocessors alongside existing Pricing, Admin demo, Contact sales; `flex-wrap` added for responsive overflow. No new features invented — all content references `docs/ENTERPRISE.md` commitments and existing pages/docs already in the repo. Owners: enterprise-architect + compliance-officer. Clinical-safety: not_required (no food/body/injury/mental-health content).
- `VERSION` — 4.35.0 → 4.35.1.

---

## [4.35.0] — 2026-06-13

### Added
- `content/post-615-broken-mesocycle-mid-block-disruption.md` — Programming edge cases block, post 15/50. "Розламаний мезоцикл — що робити, коли блок перервано на середині." Covers two disruption scenarios (stress-without-recovery vs. rest-without-training), block-phase-dependent decision algorithm (early break → restart; mid-block → reintegration week; late-block → deload + new cycle), sunk cost bias in periodization, and a practical reintegration protocol. References: Schmidt & Lee motor learning (procedural memory retention), Ogasawara et al. 2013 (structural adaptation preservation). Clinical safety: not_required — pure periodization/programming content, no food/body/injury/mental health framing. ~1900 words, Ukrainian, editorial-brutalist tone.

### Changed
- `blog.html` — картка post-615 додана на початок grid.
- `STATUS.md` — block 601–650 updated: 14 → 15 posts; total 614 → 615; version v4.33.0 → v4.35.0.
- `VERSION` — 4.34.0 → 4.35.0.

---

## [4.34.0] — 2026-06-13

### Added
- `content/post-613-first-goal-achieved-whats-next.md` — Programming edge cases block, post 13/50. Що робити після закриття першої силової цілі: три питання замість рефлекторної наступної цифри; транзитний 4-тижневий блок; три архітектурні варіанти наступного вектора (кількісний поріг, якісний стрибок, зміна модальності). Persona: Олексій, щойно закрив першу значиму ціль.
- `content/post-614-natural-athlete-enhanced-research.md` — Programming edge cases block, post 14/50. Як натуральному атлету читати дослідження з enhanced популяцій: механізм MPS, об'ємна толерантність, частота тренувань, фільтр для ідентифікації enhanced-вибірок. Що переноситься (proximity to failure, progressive overload, rep ranges), що не переноситься напряму (об'єм 20+, висока частота). Практичний чотирикроковий фільтр для читача.

### Changed
- `STATUS.md` — block 601–650 updated: 12 → 14 posts; total 612 → 614; next priorities reference updated to post-614+.
- `VERSION` — 4.33.1 → 4.34.0.

---

## [4.33.1] — 2026-06-13

### Changed
- `docs/SOC2_READINESS.md` v3.5.5 → v3.7.0. New §77 P8.2 Privacy Incident Classification & Operating Evidence: PI-P0–PI-P3 severity taxonomy, three new DEC-030 events (`privacy.incident_opened`, `privacy.incident_reviewed`, `privacy.incident_closed`) with Zod v2 schemas, PRIV-INC-CHAIN-01 ordering invariant, runbook mapping (R-11, R-22, §15), evidence artefacts PRIV-PI-E-001–E-004, and SOC 2 criteria mapping (P8.1/P8.2/P8.3/CC7.4). Closes §2 P8 row "Privacy incident tracking" 🟡 Gap → 🟡 Partial. §2 gap table corrections: "DR test performed annually" 🔴 Gap → 🟡 Partial (BCP §7 + BC-SLO-04 already documented), "Annual privacy review" 🔴 Gap → 🟡 Partial (§60 already documented). Gap Analysis Summary: 34 → 36 Partial; readiness score ~98.3% → ~98.4%. Owners: compliance-officer + security-engineer.
- `VERSION` — 4.33.0 → 4.33.1.

---

## [4.33.0] — 2026-06-13

### Added
- `content/post-612-invisible-progress-adaptation-without-load-increase.md` — Programming edge cases block, 12th post. "Невидимий прогрес — коли адаптація є, але вага на штанзі не росте." Covers four adaptation types that don't show as load increases: neural (Moritani & deVries 1979, Folland & Williams 2007), technical efficiency (force transmission, bar path), connective tissue lag (Wiesinger et al. 2015), and training-age-specific progress markers. Four practical non-load metrics: RPE trend at fixed weight, technique quality under fatigue, inter-set recovery, and volume at stable quality. Reference: Lasevicius et al. (2019) on stimulus vs. load in intermediate athletes. Clinical safety: not_required — no food/body/injury/mental health content. Sports-scientist reviewed. ~1900 words, Ukrainian, editorial-brutalist tone.

### Changed
- `blog.html` — картки post-611 і post-612 додані на початок grid (post-611 була пропущена в попередній ітерації).
- `STATUS.md` — версія v4.31.0 → v4.33.0; блок 601–650 оновлено: 11 → 12 постів; Total posts: 611 → 612.
- `VERSION` — 4.32.0 → 4.33.0.
- `README.md` — Block 601–650 оновлено: initial 10 topics позначено як Done (posts 601–611), post-612 додано, 9 нових тем для 613–650 додано до roadmap.

---

## [4.32.0] — 2026-06-13

### Added
- `content/post-611-when-self-coaching-hits-its-limit.md` — Programming edge cases block, 11th post. "Де закінчується самокоучинг — коли потрібен зовнішній тренер." Covers four structural limits of self-coaching: technical blind spots (external vs. internal feedback, motor learning principle), diagnostic dead-end (two-plateau signal), programming complexity overflow, and competitive context. Clarifies what an external coach actually provides (external observation, cross-client pattern recognition, explicit independent programming) vs. what they don't (motivation, accountability as primary function). Three-question decision framework in prose. Clinical safety PASS — all indicators are programming/performance markers only; no food/body/injury/mental health content. Sports-scientist reviewed. ~1900 words, Ukrainian, editorial-brutalist tone. Completes the initial 10-topic plan for Block 601–650 (programming edge cases).

### Changed
- `STATUS.md` — content table updated: post-611 added, total 610 → 611; next priorities updated to reflect block 601–650 initial plan complete, Block 651–700 clinical-safety gate flagged for chronic pain topic.

---

## [4.31.2] — 2026-06-13

### Changed
- `docs/MOBILE_ROADMAP.md` v0.1 → v0.2 (platform-engineer pass). Three updates: (1) Q-M-011 closed — StoreKit 2 directly confirmed, no RevenueCat at v1.0, per Founding Engineer decision already recorded in Platform Decisions table. (2) Q-M-012 added — `readiness_bucket` PostHog event on mobile is a compliance hold: DEC-046 CONDITIONAL PASS requires PIA filing + PostHog DPA review before mobile instrumentation activates. (3) Q-M-013 added — enterprise mobile shared-responsibility boundary documented; FORM owns mobile app security, customer owns MDM/corporate device management; reference `docs/SHARED_RESPONSIBILITY_MODEL.md §6`. §8.7 Enterprise section updated to reference SHARED_RESPONSIBILITY_MODEL.md and clarify FORM vs. customer boundary.

---

## [4.31.1] — 2026-06-13

### Changed
- `docs/AUDIT_LOG_SCHEMA.md` — v2.1 → v2.2. +10 `sla.*` DEC-030 HMAC-chained event types registered (closes OBSERVABILITY.md §23.11 checklist item 8, P0). New section: `### Enterprise SLA events (OBSERVABILITY §23 · A1.1 / CC7.2)`. Events: `sla.incident_opened/closed` (HIGH, 7yr), `sla.measurement_reconciled` (STANDARD, 7yr — dual-source Better Stack / Cloudflare Analytics reconciliation), `sla.credit_calculated/approved/adjusted` (HIGH, 7yr — financial audit chain), `sla.dispute_opened` (STANDARD, 3yr), `sla.dispute_resolved` (HIGH, 7yr), `sla.maintenance_window_registered` (STANDARD, 3yr), `sla.exclusion_reclassified` (HIGH, 7yr — security-engineer approval + second-approver gate for windows > 30 min). Three ordering invariants (SLA-CHAIN-01/02/03) enforced by `emit-audit-event` Worker (HTTP 422 on violation). Ten Zod schemas provided. Retention table: +8 rows. Privacy floor confirmed: no user_id, no employee PII, no health values; `dispute_reason` hashed if contains `@`; financial fields (`credit_amount_usd`, `final_credit_usd`) tenant-RLS protected. SOC 2: A1.1 (availability commitment audit chain) + CC7.2 (SLA anomaly monitoring). Cross-ref: OBSERVABILITY.md §23.9, ENTERPRISE_SLA.md, SOC2_READINESS.md §2 A1.1.
- `VERSION` — 4.31.0 → 4.31.1.

---

## [4.31.0] — 2026-06-13

### Added
- `content/post-610-flexible-microcycle-variable-schedule.md` — Programming edge cases block, post 10/50. Flexible microcycle for irregular schedules: replacing the fixed weekly template with a time-window + session-count model. Three models (2/7, 3/7, 4/7) with session priority rules (heavy compound always first). Minimum effective frequency anchor: Schoenfeld et al. 2016 (2×/week threshold for hypertrophy). Emergency single-session protocol. Bickel et al. 2011 on maintenance under reduced frequency. Diagnostic log-based schedule variance analysis. Sports-scientist reviewed, clinical-safety: not_required.
- `blog.html` — картки post-608, post-609, post-610 додані на початок grid.

### Changed
- `STATUS.md` — блок 601–650 оновлено до 10 постів; total 610; версія → 4.31.0.
- `VERSION` — 4.30.0 → 4.31.0.

---

## [4.30.0] — 2026-06-13

### Added
- `content/post-608-masters-athletes-programming.md` — Programming edge cases block, post 8/10. Masters athletes (40+): what actually changes (testosterone ~1–2%/yr decline post-30, type II fiber atrophy, connective tissue turnover), what doesn't (progressive overload, exercise selection, capacity for progress), and practical programming adjustments (inter-session recovery +24–48h, MRV ~15–20% lower default, RIR 1–2 default on compound lifts, deload every 3–4 weeks). Concrete 35→47 year-old program comparison. Argues for maintaining heavy work — not reducing intensity — as the stronger case against accelerated aging. Sports-scientist reviewed, clinical-safety: not_required.
- `content/post-609-taper-protocols-competition.md` — Programming edge cases block, post 9/10. Competition taper protocols with evidence base: fitness-fatigue model mechanics (τ₁ ≈ 42 days fitness, τ₂ ≈ 13 days fatigue), Bosquet et al. 2007 meta-analysis (27 studies, ~2000 athletes, 2–3% performance gain), Murach & Bagley 2015 (intensity preservation requirement for strength). Three taper types (step/exponential/linear) with exponential having strongest evidence. Powerlifting-specific: 2-week taper template, last heavy session timing, opener selection (92–93% of working max), second-attempt strategy. Five common mistakes (reducing intensity not volume, tapering too long, adding new exercises, full rest week, ignoring external stress). Sports-scientist reviewed, clinical-safety: not_required.

### Changed
- `VERSION` — 4.29.1 → 4.30.0.

---

## [4.29.1] — 2026-06-13

### Changed
- `docs/SOC2_READINESS.md` — §76 CC7.2/CC4.1 Security Monitoring & Anomaly Detection Operating Evidence added (v3.6.3). Formally closes §2 CC7 gap "Anomaly alerting (auth failures, spike detection)" 🟡 Gap → 🟡 Partial. Packages the alert corpus already defined in OBSERVABILITY.md §7, §27, and §34 as an auditor-ready exhibit: six primary security alert rules (auth failure spike P1, HMAC chain break P0, tenant_id missing P1, traffic spike P2, audit log write failure P0, Enterprise SSO outage P1), four SIEM correlation rules CR-01–CR-04 with detection paths (Analytics Engine real-time for CR-01/CR-04; pg_cron 5-min bridge for CR-02/CR-03 until M9 ClickHouse migration), CC4.1 meta-monitoring via AL-SIEM-06 dead-man's switch + pg_cron freshness gates (jobs 22–23) + S-007 chain integrity probe, six evidence artefacts ANOM-E-001 through ANOM-E-006, twelve-item implementation checklist (4× P0/M5, 5× P1/quarterly, 2× P1/M9, 1× P2/Q1-2027), two open questions (OQ-ANOM-01 per-tenant CR-01 blind-spot risk P1, OQ-ANOM-02 AL-SIEM-06 off-peak false-positive threshold P1). Privacy floor confirmed: no user_id, no health data, no plaintext IP in any CC7.2 signal. Overall SOC 2 readiness ~98.6% → ~98.7% (authored).
- `VERSION` — 4.29.0 → 4.29.1.

---

## [4.29.0] — 2026-06-13

### Added
- `content/post-607-concurrent-training-interference-resolved.md` — Programming edge cases block, post 7/50. Deep dive into concurrent training interference: AMPK-mTOR molecular mechanism, four controlling parameters (session order, time gap, cardio mode, volume), three practical scheduling architectures. Evidence from Hickson 1980, Wilson et al. 2012, Murach & Bagley 2016, Coffey & Hawley 2017, Küüsmaa et al. 2016. Clinical safety: PASS — physiology only, no food/body-image/appearance framing.
- `blog.html` — картки post-605, post-606, post-607 додані на початок grid.
- `README.md` — Block 751–800 (Athlete lifecycle & context) і Block 801–850 (Evidence-based deep dives) додані до roadmap.
- `STATUS.md` — блок 601–650 оновлено до 7 постів; total 607; версія → 4.29.0.

### Changed
- `VERSION` — 4.28.0 → 4.29.0.

---

## [4.28.0] — 2026-06-13

### Added
- `content/post-605-block-wave-linear-periodization.md` — Programming edge cases block, post 5/50. Practical comparison of linear, daily undulating (DUP), and block periodization models. Decision table by training age, session frequency, and planning horizon. Self-coached intermediate case study: why DUP over block for athletes without a competition date. Prestes et al. 2009 and Harries et al. 2015 cited. Clinical safety: PASS — no food/body-image/injury/mental health content.
- `content/post-606-shift-work-travel-training.md` — Programming edge cases block, post 6/50. Detraining timelines by quality (neural strength, hypertrophy, VO2max, motor skill). Three-mode switching architecture: Mode A (full, 3–4 sessions/week), Mode B (maintenance, 1–2 sessions/week), Mode C (minimal, hotel/no-gym). Specific scenarios: night shift, 3–5 day travel, unpredictable weekly schedule with planless sequencing. Bickel et al. 2011 cited. Clinical safety: PASS — circadian disruption at physiological level only, no mental health framing.
- `STATUS.md` — post count updated to 606; block 601–650 row updated to 6 posts; next priorities updated.

### Changed
- `VERSION` — 4.27.3 → 4.28.0.

---

## [4.27.3] — 2026-06-13

### Changed
- `docs/OBSERVABILITY.md` — §40 Pre-Launch Load Testing & Performance Capacity Observability added (v3.7). Closes SOC 2 observability gap *"Load testing before launches"* by adding the DEC-030 HMAC-chain layer to the k6 program already defined in `docs/SOC2_READINESS.md §33.3`. Five DEC-030 events (`system.load_test_initiated`, `_completed`, `_failed`, `_gate_bypassed` CRITICAL, `system.perf_regression_detected`); five production alert rules AL-PERF-01 through AL-PERF-05; six SLOs PERF-SLO-01–06 aligned with §33.3 and §33.4.1 production targets; pg_cron job 30 quarterly regression check; six-panel Metabase dashboard; three SOC 2 evidence artefacts LT-E-001–LT-E-003. Auditor narrative closes the policy→gate→performance→SLA evidence chain via `commit_sha` linkage to §38 CI-E-001.
- `docs/SOC2_READINESS.md` — §2 gap table: *"Load testing before launches"* advanced from 🟡 Gap → 🟡 Partial (program defined in §33.3; DEC-030 gate events now in OBSERVABILITY.md §40; first gate execution pending M5).
- `VERSION` — 4.27.2 → 4.27.3.

## [4.27.2] — 2026-06-13

### Added
- `content/post-604-when-to-switch-program.md` — Programming edge cases block 601–650, post 4. Коли змінювати програму, а коли залишатися: чотири реальних сигнали зміни vs. чотири хибних (DOMS-адаптація, синдром новизни, нетерпіння, соціальне порівняння), матриця рішення по шести критеріях, часові горизонти оцінки за рівнем (початківець 8 тижнів / проміжний 10–12 / просунутий 8+), часті помилки (міняти при збереженій проблемі, виходити на піку акумуляції), різниця між «скоригувати» і «замінити», практичний 5-крокові алгоритм. Тон: editorial-brutalist. Clinical-safety: PASS.
- `blog.html` — картка post-604 додана на початок grid.

### Changed
- `STATUS.md` — блок 601–650 оновлено до 4 постів; total 604; версія → 4.27.2.
- `VERSION` — 4.27.1 → 4.27.2.

---

## [4.27.1] — 2026-06-13

### Added
- `content/post-603-minimum-effective-dose.md` — Programming edge cases block 601–650, post 3. Мінімальна ефективна доза: що не можна скоротити, а що можна. Три різних MED (підтримка/прогрес/форс-мажор), конкретні числа з дослідження (Ralston 2017, Schoenfeld 2016), ієрархія жертвування змінними при скороченні часу (інтенсивність → компаунд → близькість до відмови → об'єм), двохсесійний і трьохсесійний тижневі шаблони, типові помилки. Тон: editorial-brutalist, практично орієнтований.

### Changed
- `content/post-602-dup-vs-single-session.md` — sports-scientist corrections applied: Rhea 2002 протокол уточнено як 4RM/6RM/8RM (не strength/hypertrophy/power трійця з пізніших робіт); Miranda 2011 результат виправлено (немає статистично значущої між-групової різниці, DUP тренд кращий за effect sizes); MPS вікно уточнено (коротше у тренованих ~24h, ніж у нетренованих 48-72h); natural vs. enhanced сформульовано як механістичний inference; додано Schoenfeld et al. 2016 + Helms et al. 2014.
- `STATUS.md` — блок 601–650 оновлено до 3 постів; next priorities оновлено.
- `VERSION` — 4.27.0 → 4.27.1.

---

## [4.27.0] — 2026-06-13

### Added
- `content/post-602-dup-vs-single-session.md` — Programming edge cases block 601–650, post 2. Single-session vs. щоденне хвильове програмування (DUP) для натурального атлета: що реально означає DUP (vs. WUP і блокова periodization), огляд дослідження (Rhea et al. 2002, Miranda et al. 2011) із чесним коментарем щодо обмежень, фізіологічне обґрунтування для натурала (вікно MPS 24-36h, системна і периферична втома), практична матриця «DUP vs. single-session focus» за 6 параметрами, трьохденний тижневий шаблон DUP із відсотками 1RM, типові помилки застосування. Тон: editorial-brutalist, без оверклеймів.

### Changed
- `STATUS.md` — блок 601–650 оновлено до 2 постів (draft); next priorities оновлено.
- `VERSION` — 4.26.1 → 4.27.0.

---

## [4.26.1] — 2026-06-13

### Changed
- `docs/AUDIT_LOG_SCHEMA.md` v2.0 → v2.1 — +11 DEC-030 HMAC-chained events across two new families: **CI/CD Pipeline** (5 events: `system.deployment_completed` formalised, `ci.migration_applied` HIGH 7yr, `ci.migration_failed` CRITICAL 7yr, `ci.deployment_rolled_back` HIGH 7yr, `ci.pipeline_failed` STANDARD 3yr — Closes OBSERVABILITY §38.10 checklist item 1, P0 M7) and **Backup & DR Observability** (6 events: `system.backup_completed` STANDARD 5yr, `system.backup_failed` CRITICAL 7yr, `system.restore_test_initiated` STANDARD 7yr, `system.restore_test_completed` STANDARD 7yr, `system.restore_test_failed` CRITICAL 7yr, `system.backup_staleness_detected` HIGH 7yr — Closes OBSERVABILITY §39.11 checklist item 1, P0 M13 enterprise GA). Full Zod schemas provided. BC-CHAIN-01 and CI-CHAIN-01 ordering invariants documented. Retention table +11 rows. Generic `### System` section updated to reference dedicated schema sections.
- `docs/AUP.md` — created as alias redirect to `docs/ACCEPTABLE_USE_POLICY.md`. Fixes dangling reference in `docs/SOC2_READINESS.md` (3 occurrences).
- `VERSION` — 4.26.0 → 4.26.1.

---

## [4.26.0] — 2026-06-13

### Added
- `content/post-601-return-after-long-break.md` — Programming edge cases block 601–650, post 1. Повернення після тривалої перерви: ієрархія втрат при детренованості (аеробна база → нейральна сила → структурна маса), міонуклеарна пам'ять (Bruusgaard et al., 2010) як механістична основа м'язової пам'яті, чотиритижневий протокол повернення (40→55→70→90% об'єму, 60→70→80→90% навантажень, RIR ≥ 4→3→2→нормально), розрахунок стартових ваг за тривалістю паузи (80% / 70% / 60% / 50–60% від до-паузових), типи пауз і специфіка кожної, типові помилки (1RM тест у перший день, компенсація об'ємом при легких вагах), рекомендації щодо ведення журналу при поверненні. Посилання: Mujika & Padilla (2001) systematic review.

### Changed
- `blog.html` — додано картку post-601 (Programming edge cases · № 601 · draft).
- `STATUS.md` — блок 601–650 додано до content library table (1 пост, draft); next priorities оновлено.
- `VERSION` — 4.25.0 → 4.26.0.

---

## [4.25.0] — 2026-06-13

### Added
- `content/post-599-vo2max-longitudinal.md` — Training with tech block, post 9/10. VO2max як довгостроковий маркер: різниця між snapshot і трендом, систематичний дрейф wearable-алгоритмів після зміни пристрою, протокол зіставного вимірювання (30-денна ковзна середня, anchor субмакс-тест), реалістичні темпи зростання за рівнем тренованості (нетренований +5–10, тренований аматор +1–3, advanced +0.5–1.5 мл/кг/хв за блок), особливості silove атлетів (маса у знаменнику, cardiac output vs. peripheral utilization, алгоритм calibrated on running data). Посилання на HERITAGE Family Study та Aerobics Center Longitudinal Study.
- `content/post-600-training-tech-stack-2026.md` — Training with tech block, post 10/10. Синтез блоку: чотирирівнева архітектура стеку (Tier 0 — без технологій, Tier 1 — варто інвестувати, Tier 2 — корисно при правильному використанні, Tier 3 — ситуативно або skip). Принцип однонаправленого потоку даних: реальний світ → пристрій → аналіз → рішення. Ознаки over-stack, протокол спрощення. Scorecard 2026: що реально покращилось (AI coaching assistance, video analysis, HRV consumer accuracy), що не покращилось (sleep staging, readiness algorithm, training load estimate для силових). Closes Training with tech block (posts 591–600).

### Changed
- `STATUS.md` — posts 599–600 marked complete; Training with tech block closed; break-glass audit event priority removed (already registered in AUDIT_LOG_SCHEMA.md v0.4); next priorities updated.
- `VERSION` — 4.24.2 → 4.25.0.

---

## [4.24.2] — 2026-06-13

### Changed
- `docs/OBSERVABILITY.md` v3.6 — §39 Backup Integrity, DR Readiness & Business Continuity Observability added. Closes the observability gap explicitly noted in §37.1 ("Supabase internal backup retention — monitored via S-008 synthetic probe in §16"). §39 is the compliance-grade companion to `docs/BUSINESS_CONTINUITY.md`: five BC-SLOs (BC-SLO-01 Postgres freshness ≤ 26h, BC-SLO-02 R2 freshness ≤ 26h, BC-SLO-02b B2 EU ≤ 30h, BC-SLO-03 restore RTO ≤ 4h, BC-SLO-04 quarterly drill cadence); six alert rules AL-BC-01 through AL-BC-06 including two no-auto-resolve P0s (AL-BC-02 Postgres critical, AL-BC-05 backup failed); pg_cron job 29 `backup_age_monitor` (every 4h, complements rather than replaces S-008 synthetic probe); six DEC-030 HMAC-chained events (`system.backup_completed` STANDARD/5yr, `system.backup_failed` CRITICAL/7yr, `system.restore_test_initiated` STANDARD/7yr, `system.restore_test_completed` STANDARD/7yr with `privacy_floor_verified` boolean, `system.restore_test_failed` CRITICAL/7yr, `system.backup_staleness_detected` HIGH/7yr); BC-CHAIN-01 ordering invariant; five SOC 2 evidence artefacts BC-E-001 through BC-E-005 (A1.2, A1.3, CC7.2, CC6.5); seven-panel Metabase dashboard; eleven-item implementation checklist; OQ-BC-OBS-01 (Supabase Management API cross-validation, P1 M8) and OQ-BC-OBS-02 (quarterly drill scope, P2 M5).
- `STATUS.md` — OBSERVABILITY.md updated to v3.6; VERSION 4.24.1 → 4.24.2.
- `VERSION` — 4.24.1 → 4.24.2.

---

## [4.24.1] — 2026-06-13

### Added
- `content/post-598-tech-dependency-training.md` — Training with tech block, post 8/10. Коли технологія заважає: ознаки tech-залежності в тренуванні. Структура: 8 патернів tech-залежності (неможливість тренуватися без пристрою, decision paralysis навколо readiness score, оптимізація застосунку як ціль, ретроактивне редагування журналу, вибіркові записи, апаратні ритуали як обов'язкові умови, пристрій vs тіло — пристрій перемагає, порівняння через метрики між сезонами), практичний скид (analog week, рішення до даних, фіксована аналітика, мінімальний журнал). clinical-safety: PASS — поведінка в тренуванні, не клінічна діагностика.

### Changed
- `README.md` — Додано Block 651–700 (Biomechanics & injury prevention, 10 тем) та Block 701–750 (Strength sports specifics, 8 тем) у roadmap. Block 601–650 розширено з 5 до 10 тем.
- `blog.html` — Додано картку post-598.
- `STATUS.md` — Training with tech: 7→8 posts; total posts 597→598; next priorities: posts 599–600.
- `VERSION` — 4.24.0 → 4.24.1.

---

## [4.24.0] — 2026-06-13

### Added
- `content/post-594-ai-coach-vs-gpt-wrapper.md` — Training with tech block, post 4/10. AI-coach проти GPT-wrapper: як відрізнити продукт від автозаповнення. Структура: архітектура wrapper vs coaching-продукту, 5 ознак wrapper (однакова відповідь при однаковому запиті, ігнорування даних, відмова від конкретності при специфічному запиті, впевненість без застережень), що відрізняє справжній coaching-продукт (збережений контекст, детектування патернів, детермінована логіка vs генерація), 5 практичних питань перед вибором. Секція про FORM: Victor — голос, логіка — детермінована. clinical-safety: PASS.
- `content/post-595-strength-apps-progress-data.md` — Training with tech block, post 5/10. Силові блоки в застосунках: що вважати «прогресом» у даних проти у штанзі. Структура: що справді вимірюється (виконання, не здатність), метрики з фізіологічним змістом (1ПМ, оцінний 1ПМ, RPE-тренд, технічна стабільність), метрики без змісту (тоннаж, streak, PR без контексту), патерни для відстеження, пастка «оптимізація даних як ціль», практичні налаштування застосунку. clinical-safety: PASS.
- `content/post-596-readiness-score-dashboard-trap.md` — Training with tech block, post 6/10. Readiness score і dashboard-trap: ієрархія сигналів для прийняття рішень. Структура: що вимірює readiness (HRV, RHR, сон), проблема агрегації (різні алгоритми, різні числа, відсутність тренувального контексту), коли скор корисний (ненаправлений відпочинок, тригер для аудиту, нетипові відхилення), коли заважає (суперечить плану без причини, anxiety навколо числа, піковий блок — норма низького скору), ієрархія: програма → суб'єктивне відчуття → об'єктивні дані → readiness score. clinical-safety: PASS.
- `content/post-597-training-log-primary-data-source.md` — Training with tech block, post 7/10. Тренувальний журнал як первинний data source. Структура: що записувати (навантаження, RPE, суб'єктивний стан, технічні нотатки, відхилення від плану), що не записувати, чому журнал важливіший за wearable-дані (контекст vs факт), формати від простого до аналітичного, аналіз по закінченню блоку, журнал як арбітр у самокоучингу, типові помилки (ретроспективне редагування, запис тільки хороших тренувань, ігнорування RPE). clinical-safety: PASS.

### Changed
- `STATUS.md` — Training with tech: 3→7 posts; total posts 593→597; next priorities updated: posts 598–600 (3 remaining).
- `VERSION` — 4.23.1 → 4.24.0.

---

## [4.23.1] — 2026-06-13

### Added
- `docs/AUDIT_LOG_SCHEMA.md §Privacy Impact Assessment events` — six `privacy.pia_*` DEC-030 HMAC-chained events registered: `privacy.pia_filed` (STANDARD, 7yr), `privacy.pia_completed` (HIGH, 7yr), `privacy.pia_veto_issued` (CRITICAL, 7yr — triggers PRIV-VETO-001 PagerDuty alert), `privacy.constraint_relaxation_rejected` (HIGH, 7yr), `privacy.annual_review_scope_drafted` (STANDARD, 3yr), `privacy.annual_review_completed` (HIGH, 7yr). Zod schemas, PIA-CHAIN-01 ordering invariant, retention table rows (+3). Closes `docs/PRIVACY_IMPACT.md §11` P0/M4 checklist item 1.
- `docs/PRIVACY_IMPACT.md §10` — PIA-2026-001 filed: `readiness_bucket` PostHog conditional pass (T-6, Medium risk, clinical-safety CONDITIONAL PASS + VETO for mood_bucket). First entry in the living PIA Register. OQ-PIA-03 closed (🟢 DEC-046 + PIA-2026-001).

### Changed
- `docs/AUDIT_LOG_SCHEMA.md` — v1.9 → v2.0; retention table +3 rows for `privacy.pia_*` events.
- `docs/PRIVACY_IMPACT.md` — v1.0 → v1.1; §10 PIA Register populated; §11 item 1 marked documented; OQ-PIA-03 resolved.
- `VERSION` — 4.23.0 → 4.23.1.

---

## [4.23.0] — 2026-06-13

### Added
- `content/post-593-video-technique-analysis.md` — Training with tech block, post 3/10. Відеоаналіз техніки: що смартфон показує, що ні, де лабораторія принципова. Структура: кінематика vs механіка, частота кадрів (30/60/240 fps), позиція камери і вибір площини, різниця між симптомом і діагнозом у відео, протокол зйомки для self-coached атлета, огляд інструментів (Kinovea/Coach's Eye), коли потрібна 3D motion capture або силова платформа, самоаналіз vs тренерський погляд. Практична таблиця «питання → площина → позиція камери». clinical-safety: PASS.

### Changed
- `blog.html` — додана картка post-593 вгорі блоку Training with tech.
- `STATUS.md` — оновлено: Training with tech block 2→3 posts, total 592→593, next priorities 594–600.
- `VERSION` — 4.22.0 → 4.23.0.

---

## [4.22.0] — 2026-06-13

### Added
- `content/newsletter-04.md` — Issue 04 editorial: "Everything you read about training is right. None of it is about you." Synthesises the research literacy block (posts 586–590) for the broader beta audience. Core argument: population-level research generates priors, not prescriptions; individual response variability is large; the training log is the update mechanism. Practical filter: two questions to ask before applying any study finding (subject population, individual variation). Bridges research literacy block to the upcoming training-with-tech block. Subject line options and preview text included. clinical-safety: PASS.

---

## [4.21.0] — 2026-06-13

### Added
- `content/post-592-hrv-guided-training.md` — Training with tech block, post 2/10. HRV-guided training: what HRV actually measures (autonomic nervous system state, not muscle fatigue directly), valid measurement protocol (same time, same device, same position), personal baseline formation (minimum 14 days, reliable at 4 weeks). Three-zone decision framework (green/yellow/red relative to personal baseline). Decision algorithm per zone. Core topic: two types of score-vs-feeling discrepancy and different responses for each (device lower than felt → train but don't exceed plan; device higher than felt → trust subjective state). HRV-as-weather-forecast framing. Five common errors. When HRV monitoring adds no value (< 4 sessions/week low intensity). sports_scientist_review: required before publish. clinical-safety: PASS.

### Changed
- `STATUS.md` — content table updated: post-592 added, total posts 591 → 592; next priorities updated to reflect newsletter-04 drafted status and posts 593–600 as next content horizon.

---

## [4.20.1] — 2026-06-12

### Changed
- `docs/OBSERVABILITY.md` v3.5 — §38 CI/CD & Deployment Pipeline Observability added. Defines DORA metrics (deployment frequency, lead time, change failure rate, MTTR) with Postgres queries against `audit_log_events`. RED metrics via `CI_TELEMETRY` Analytics Engine (eight-column schema, no PII). Four SLOs: CI-SLO-01 test pass rate ≥ 98%, CI-SLO-02 smoke test ≥ 99.5%, CI-SLO-03 migration failure rate = 0% (hard floor), CI-SLO-04 rollback MTTR P95 < 30 min. Six alert rules AL-CI-01 through AL-CI-06 including P0 dual-page for production migration failure. Five DEC-030 HMAC-chained events: `system.deployment_completed` (formalised Zod schema, STANDARD 5yr), `ci.migration_applied` (HIGH 7yr), `ci.migration_failed` (CRITICAL 7yr), `ci.deployment_rolled_back` (HIGH 7yr), `ci.pipeline_failed` (STANDARD 3yr). Eight-panel Metabase dashboard. Five SOC 2 evidence artefacts CI-E-001–CI-E-005 (CC8.1, CC7.2). pg_cron job 28 `ci_telemetry_daily_sync`. Twelve-item implementation checklist through M13. Cross-references: SOC2_READINESS §21 (CC8.1), INCIDENT_RESPONSE R-02, ENTERPRISE_SLA §4.3 RTO, AUDIT_LOG_SCHEMA §System, DEC-030.

---

## [4.20.0] — 2026-06-12

### Added
- `content/post-591-wearables-accuracy.md` — Training with tech block, post 1/10. Consumer wearable accuracy: PPG limitations for HRV vs. ECG gold standard, sleep staging PSG gap (58% stage accuracy), VO2max submax estimation with ±3–5 ml/kg/min range, TRIMP-based training load blind spots. Practical framework: trust trend not point, device as one data source not authority.
- `README.md` — content roadmap added: published block table, block 591–600 topic list (Training with tech), proposed block 601–650 (Programming edge cases). Replaces placeholder.

### Changed
- `STATUS.md` — new content block row 591 added; next priorities updated to include post-591 review.

---

## [4.19.0] — 2026-06-12

### Added
- `content/post-586-effect-size-sports-science.md` — Research literacy block post 1/5. Cohen's d in practical training terms: what 0.2/0.5/0.8 actually means for programming decisions. Statistical vs. practical significance, confidence intervals, winner's curse in underpowered studies.
- `content/post-587-study-duration-vs-adaptation.md` — Research literacy block post 2/5. Why 6-week studies cannot answer 6-month questions. Neural (4–8 wk), structural hypertrophy (8–16+ wk), connective tissue (months–years) adaptation timelines. Residual training effects as an uncontrolled confound. Wash-in period as a study quality signal.
- `content/post-588-population-bias-research.md` — Research literacy block post 3/5. Why untrained-subject results don't transfer directly to trained athletes: ceiling effects, prior neural adaptation, connective tissue state, supplement response profiles. Practical filter for evaluating study populations.
- `content/post-589-individual-vs-population-recommendations.md` — Research literacy block post 4/5. Individual response variability in hypertrophy (Hubal et al. range: 5–59%). High/low responder continuum. n-of-1 approach for calibrating personal dose-response. When population guidelines are priors vs. when your training log overrides them.
- `content/post-590-evidence-hierarchy-sports-science.md` — Research literacy block post 5/5. Full evidence hierarchy from anecdote to meta-analysis. RCT within-subject vs. between-group design. Meta-analysis limitations: heterogeneity (I²), publication bias (funnel plots), pooling problem. Expert consensus lag. Five practical questions for evaluating any training claim.
- `STATUS.md` — full project status board replacing placeholder: content library table, enterprise doc inventory, open decisions summary, active constraints, next priorities.

## [4.18.0] — 2026-06-12

### Added
- `docs/SHARED_RESPONSIBILITY_MODEL.md` v1.0 — enterprise shared responsibility model covering infrastructure, application, identity, data, audit, operations, compliance, onboarding/offboarding, and liability boundaries. Consolidates responsibilities previously scattered across DATA_MODEL.md, SSO_SCIM_IMPLEMENTATION.md §GDPR, and ENTERPRISE.md §commitments. Primary audience: enterprise security teams, procurement, and legal. References DEC-030, ENTERPRISE_SLA.md, SOC2_READINESS.md.

## [4.17.1] — prior
