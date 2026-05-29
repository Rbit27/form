# Changelog · FORM

Усі помітні зміни в цьому проєкті документуються тут.
Формат базується на [Keep a Changelog 1.1.0](https://keepachangelog.com/en/1.1.0/).
Версіювання: **semver-light** — `MAJOR.MINOR.PATCH`.

---

## [1.17.3] — 2026-05-29

### Added
- `content/post-141-autoregulation-rpe-rir-vbt.md` — Авторегуляція: RPE 1–10 у силовому тренінгу (Zourdos et al. 2016); RIR як контролер навантаження (Helms et al. 2016); VBT — MPV/% 1RM кореляція (González-Badillo 2010), мінімальна швидкість відмови, індивідуальний load-velocity профіль; порівняльна таблиця трьох методів; комбінована стратегія; обмеження для новачків. clinical-safety NOT REQUIRED.
- `blog.html` — картки post-139 (PAP), post-140 (eccentric / titin), post-141 (autoregulation RPE/RIR/VBT)

### Changed
- `VERSION` → 1.17.3

---

## [1.17.2] — 2026-05-29

### Added
- `content/post-140-eccentric-training-mechanisms.md` — Ексцентричний тренінг: тітін як активний пружний елемент при Ca²⁺-залежному розтягуванні (Herzog et al. 2012, Nishikawa et al. 2020); ексцентрична сила ≥ 120–150% концентричного максимуму; stretch-mediated hypertrophy і саркомерогенез у серію (Oranchuk et al. 2019); Nordic hamstring curl −51% травм підколінних сухожилків (Petersen et al. 2011); акцентоване ексцентрическое (AEL) і протокол 4-секундного опускання; переважна гіпертрофія Type II волокон; фазування в мезоциклі. clinical-safety NOT REQUIRED.

### Changed
- `VERSION` → 1.17.2

---

## [1.17.1] — 2026-05-29

### Added
- `content/post-139-post-activation-potentiation.md` — Постактиваційна потенціація: MLC-фосфорилювання як молекулярна основа PAP; PAP vs PAPE (Blazevich & Babault 2019); конкурентні ефекти потенціації і втоми, вікно 4–12 хвилин; залежність від тренувального рівня і відсотка Type II волокон; комплексне тренування (таблиця пар: присідання → стрибок, жим → метання); мета-аналіз Hodgson et al. (2005); responders vs non-responders; критерії включення в мезоцикл. clinical-safety NOT REQUIRED.

### Changed
- `VERSION` → 1.17.1

---

## [1.17.0] — 2026-05-29

### Added
- `docs/DATA_MODEL.md §22` — Gamification, Streak & Achievement Schema. Closes the data-layer gap for DEC-013 (streak grace after 2 consecutive misses, not 1) and formalises personal record and achievement storage. Five-table schema: `user_streaks` (current_streak, longest_streak, streak_start_date, last_activity_date, grace_used_this_cycle, total_grace_uses; `chk_streak_longest` invariant constraint), `streak_grace_events` (append-only grace history; no UPDATE/DELETE RLS for non-admin), `personal_records` (PR series per user × exercise × metric; `pr_metric` ENUM with 5 metrics; current-best via `DISTINCT ON`; e1RM estimates with formula attribution in context JSONB), `achievement_definitions` (form_admin-only write; dotted-namespace key; soft-archive via `is_active`; 19-item seed catalog across 5 categories and 4 tiers), `user_achievements` (form_system INSERT only; UNIQUE index preventing same achievement twice in one calendar day). DEC-013 state machine: 4 transition paths fully specified; `grace_used_this_cycle` flag enforces at-most-one grace per cycle; DEC-013 invariant table with enforcement locations; CI regression test specification. RLS: `form_api` own-row read/update on streaks; no direct form_api insert on grace_events or user_achievements; `tenant_admin` zero access (DEC-002 compliance). DEC-030: 6 HMAC-chained event types (gamification.streak_started, streak_grace_applied, streak_reset, streak_milestone, pr_set, achievement_earned); STANDARD severity, 3-year retention; emitted in-transaction. GDPR Art. 17: explicit 4-step erasure sequence with per-table DEC-030 events; ON DELETE CASCADE safety net. Privacy floor: zero aggregate gamification views; any future admin reporting requires §17 aggregate layer + k-anonymity ≥ 5 + clinical-safety gate (DEC-029). SOC 2: CC6.1, CC6.2, CC7.2, CC8.1, P4.0/P5.0. 5 open questions (OQ-GAM-01 through OQ-GAM-05). 12-item implementation checklist (8× P0 M3, 3× P1 M4, 1× P0 M3 erasure). Data model version bump v1.1 → v1.2. ToC updated.

### Changed
- `VERSION` → 1.17.0

---

## [1.16.6] — 2026-05-29

### Changed
- `docs/SOC2_READINESS.md §39` — Security Trust Center Architecture (`security.form.coach`). Full portal design for the enterprise-facing trust page. Seven pages specified across public and NDA-gated tiers: `/` security overview, `/sub-processors` (closes CC9-GAP-007 design phase 🔴→🟡: Cloudflare Worker spec, `SECURITY_PORTAL_KV` schema, JSON API, SHA-256 `X-List-Hash` header for programmatic polling by procurement tools), `/dpa` DPA template download with SCC Module 2 note, `/disclosure` responsible disclosure policy (90-day window, PGP key, CVSS 3.1 severity table), `/questionnaire` CAIQ Lite PDF hub, `/soc2` NDA-gated SOC 2 report access (HTML request form → Resend notification → CSM DocuSign NDA → Vanta/Drata link, 7-day expiry), `/pentest` executive summary NDA-gate. `trust_center_requests` Supabase table DDL (form_admin-only RLS; form_api and form_system excluded; no tenant exposure). Eight DEC-030 HMAC-chained audit events covering full trust center lifecycle (`trust_center.sub_processor_list_updated`, `trust_center.soc2_report_requested/approved/declined/shared`, `trust_center.pentest_summary_shared`, `trust_center.nda_signed`, `trust_center.disclosure_report_submitted`). Gap status: CC9-GAP-007 🔴→🟡 (sub-processor Worker fully specified; deployment remaining), CC2-GAP-003 🔴→🟡 (publication architecture defined; deployment closes to 🟢). SOC 2 evidence mapping: CC9.2 (vendor list transparency), CC2.3 (change notification pledge), CC6.1 (NDA-gate on restricted docs), CC6.2 (access issuance log), CC7.1 (responsible disclosure), CC8.1 (HMAC-chained mutations), P6.1 (DPA public availability). 14-item implementation checklist (8× P0 M4, 5× P1 M4/M5, 1× P2 M5). SOC 2 readiness: ~95% → ~95% (design complete; advances when Worker deployed and screenshot filed as CC9 evidence artifact).
- `VERSION` → 1.16.6

---

## [1.16.5] — 2026-05-29

### Added
- `content/post-138-satellite-cells-myonuclear-domain.md` — Satellite cells і міоядерний домен: PAX7+/MYOD1+ активаційний каскад (HGF, FGF-2, MGF), міоядерний домен ~1500–2000 μm³ (Cheek 1985, Kadi & Thornell 2000), responders vs non-responders (Bamman 2003, Petrella 2006/2008), Bruusgaard et al. (2010 PNAS) — міоядра персистують після атрофії, McCarthy et al. (2011 PNAS) — satellite cell-незалежна гіпертрофія до певної межі, Seaborne et al. (2018 Sci Rep) — епігенетичний відбиток тренінгу, 5 практичних наслідків для ексцентрики, close-to-failure, повернення після паузи. clinical-safety NOT REQUIRED.
- `blog.html` — картка post-138 (Satellite cells і міоядерний домен)
- `README.md` — post-138 позначений [x]; нові roadmap-записи post-139–142 додані
- `STATUS.md` — рядки post-132–138 додані до content engine table; blog.html рахунок оновлено до 99 карток / 138 постів
- `VERSION` → 1.16.5

---

## [1.16.4] — 2026-05-29

### Added
- `content/post-137-mtorc1-signaling-cascade.md` — Сигнальний каскад mTORC1: PI3K→Akt→TSC2→Rheb→mTORC1 повний ланцюг, phosphatidic acid (Hornberger 2006 PNAS), S6K1 і 4E-BP1 як ефектори, SESN2–GATOR система амінокислотного сенсингу (Wolfson et al. 2016 Science), AMPK два механізми інгібування mTOR (Hardie et al. 2012), часовий профіль активації, 5 практичних наслідків для програмування (без дієтичних рекомендацій). clinical-safety PASS.
- `README.md` — post-137 позначений [x]
- `VERSION` → 1.16.4

---

## [1.16.3] — 2026-05-29

### Added
- `content/post-136-spinal-reflex-chains-inhibition.md` — Спінальні рефлекторні ланцюги і гальмування: GTO і Ib-автогенне гальмування, Ia-інгібіторні інтернейрони і реципрокне гальмування (Sherrington 1907), H-рефлекс і адаптації H/M у силових vs витривалісних атлетів, нейральне розгальмування як механізм ранніх приростів сили, клітини Реншоу, co-activation trade-off; clinical-safety PASS
- `README.md` — post-136 позначений [x]
- `VERSION` → 1.16.3

---

## [1.16.2] — 2026-05-29

### Added
- `blog.html` — blog cards for post-136 (Спінальні рефлекторні ланцюги і гальмування) та post-137 (Сигнальний каскад mTORC1)
- `README.md` — roadmap items post-132/133 перенумеровані як post-136/137 відповідно до реального стану content/ (пости 132–135 вже written з іншими темами)
- `VERSION` → 1.16.2

---

## [1.16.1] — 2026-05-29

### Changed
- `docs/DATA_MODEL.md §16.3` — `tenant_pilots` table extended with three new columns required to track commercial pilot structure (COST_MODEL §21.9 P1 checklist item): `pilot_type` TEXT ('free' | 'extended_free' | 'paid') with CHECK constraint and inline comments mapping to COST_MODEL §21.2 definitions; `monthly_fee_cents` INTEGER (NULL for free types; 100 = $1.00/seat/month for paid type); `founder_approved_at` TIMESTAMPTZ (NULL for standard free pilots; application-enforced before provisioning extended_free or paid pilots). `pilot_seats` CHECK ceiling raised from 500 → 10000 to accommodate paid and extended pilots targeting ≥ 500 seats. `CONSTRAINT pilot_fee_consistency` ensures fee/type coherence at DB layer. Migration block added for live-database ALTER TABLE. Implementation checklist row 2a added. Column notes table documents ASC 606 and audit-event linkage.
- `pricing-enterprise.html` FAQ — two new items added (closes COST_MODEL §21.9 P1 item "Update pricing-enterprise.html FAQ with pilot terms, discount floor note, and multi-year discount schedule"): FAQ #5 "What pilot options are available before we commit?" covers all three pilot structures (Free 90d / Extended Free 180d / Paid $1/seat/month) with eligibility, approval requirements, credit-on-conversion terms, and DPA timing; FAQ #6 "What is the multi-year discount schedule and are there price floors?" states 10%/20% annual discount for 2yr/3yr contracts, upfront stacking, multiplicative application, and hard floor prices (Starter $6/seat, Growth $8/seat, Enterprise $7/seat reference floor) with waiver requirements.
- `VERSION` → 1.16.1

---

## [1.16.0] — 2026-05-29

### Added
- `content/post-135-lateral-strength-asymmetry.md` — Латеральна асиметрія сили: LSI (Limb Symmetry Index), нормативна асиметрія у здорових атлетів (5–15%), структурна vs функціональна асиметрія, зв'язок з ризиком травми (Bishop et al. 2018 systematic review — немає консистентного доказу), коли виправляти (LSI <85%), спортивно-специфічна асиметрія, cross-education ефект, практичне вимірювання без лабораторії. clinical-safety PASS.
- `blog.html` — Картка для post-135.
- `VERSION` → 1.16.0

---

## [1.15.0] — 2026-05-29

### Added
- `content/post-132-bilateral-deficit.md` — Bilateral deficit phenomenon: why bilateral leg press produces 5–15% less force than sum of unilateral reps. Neural architecture: commissural interneurons, corticospinal drive reduction (Zijdewind & Kernell 2001), reciprocal inhibition, Ia interneurons. Population differences (Häkkinen et al. 1996). Practical programming: unilateral trains neural drive unavailable in bilateral movements — combine, not replace. clinical-safety PASS.
- `content/post-133-foam-rolling-myofascial-release.md` — Foam rolling: what the science actually shows. Fascial release mechanism is mechanically implausible at body-weight pressures (Chaudhry et al. 2008: >1000 N required). Actual mechanisms: gate control (Melzack & Wall 1965), autogenic inhibition via GTOs, mechanoreceptors. Real effects: +5–15° ROM lasting 10–20 min (Wiewelhove et al. 2019 meta-analysis), neutral acute performance impact at ≤120 s/muscle. Evidence-based protocol. clinical-safety PASS.
- `content/post-134-training-age-adaptation-ceiling.md` — Training age & adaptation ceiling: novice/intermediate/advanced response differences. Neural adaptation phase (weeks 1–12, Häkkinen 1988). Structural hypertrophy phase (months 3–18, Schoenfeld 2010). Advanced ceiling: 1–3% annual strength gain (Zatsiorsky & Kraemer), block periodization (Issurin 2010), INOL. RPA self-classification heuristic. Programming matrix by training age. clinical-safety PASS.
- `docs/COST_MODEL.md §21` — Pilot Economics, Conversion Governance & Discount Authority Matrix. Three pilot structures (Free 90d standard, Extended 180d with founder approval, Paid $1/seat/month). ASC 606 variable consideration treatment for each (closes OQ-15): free pilots zero revenue; paid pilot ratable + contingent credit liability. Conversion targets: 35%/55%/70% by type (OQ-20 calibration after 10 pilots). Discount authority matrix: Founder ≤40%, AE ≤20%, CSM ≤10% renewal, SDR 0%; floor prices ($6/seat Starter, $4.50/seat Growth). Contract minimums ($600/year, 12-month minimum term, 25-seat increment). Pilot KPIs: Day-14 activation rate as P0 leading indicator. Multi-year economics (2yr/3yr break-even analysis). Six-item implementation checklist. OQ-20, OQ-21, OQ-22 added.
- `blog.html` — Cards for posts 132, 133, 134.

---

## [1.14.1] — 2026-05-29

### Added
- `docs/OBSERVABILITY.md §23` — Enterprise SLA Reporting & Credit Calculation. Closes `docs/SOC2_READINESS.md §2 A1.1` gap ("SLA credit calculation and process"). Covers: 99.9% monthly uptime SLA definition (43.8-minute covered-downtime budget); dual-source measurement methodology (Better Stack synthetic probes primary, Cloudflare Analytics Engine corroborating); five-tier credit schedule (5/15/25/50% of monthly MRR); automated credit application without tenant claim; three-table Postgres schema (`sla_monthly_reports`, `sla_incidents`, `maintenance_windows`) with RLS tenant isolation; `GET /api/v1/admin/sla/current` and `/history` API specs; monthly PDF delivery; ten DEC-030 HMAC-chained `sla.*` event types; SOC 2 A1.1 and CC7.2 evidence mapping with four compliance artefact paths; twelve-item implementation checklist (M4/M5). Doc bumped v0.9 → v1.0.

### Changed
- `docs/OBSERVABILITY.md` — version header v0.9 → v1.0.
- `docs/SOC2_READINESS.md §2 A1.1` — "SLA credit calculation and process" status 🟡 Gap → ✅ Done; evidence updated to reference `OBSERVABILITY.md §23`.
- `VERSION` → 1.14.1

---

## [1.14.0] — 2026-05-29

### Added
- `content/post-131-circadian-training-timing.md` — Циркадна біологія і тренувальний тайминг. CLOCK/BMAL1-петля і Clock-генова регуляція мітохондрій (PGC-1α, Peek et al. 2013 Cell); добова варіація сили ~3–8% (Atkinson & Reilly 1996); mTORC1 і циркадна трансляційна активність рибосом (Jouffe et al. 2013, Cell Metabolism); хронотип MSFsc (Roenneberg 2003–2006); соціальний джетлаг і здоров'я; Clock-зсув від тренування (Wolff & Esser 2012 PNAS); ранкові компроміси (спінальна декомпресія McGill 1990); вечірні тренування і сон (Myllymäki 2011); матриця вибору часу тренування за хронотипом і типом роботи. clinical-safety PASS.

### Changed
- `README.md` — content roadmap: post-130 і post-131 відмічені як виконані.
- `blog.html` — додано картку post-131 «Циркадна біологія і тренувальний тайминг».
- `STATUS.md` — content engine table: додано рядки post-130, post-131.
- `VERSION` → 1.14.0

---

## [1.13.0] — 2026-05-28

### Added
- `docs/DATA_MODEL.md §21` — AI Coaching Session & Memory Schema. П'ять таблиць: `coaching_sessions` (session_token, model_id, context_window_used_pct, session_cost_usd, per-model token counts), `coaching_turns` (column-level AES-256-GCM шифрування content, clinical_safety_flagged per turn, append-only via RLS), `coaching_memory` (dotted-namespace memory keys, confidence score, GDPR soft-delete), `coaching_feedback` (thumbs_up/down/flagged_unsafe/flagged_inaccurate), `coaching_session_context` (hash-only immutable context snapshot). Стратегія context window: 80% threshold → auto-complete + fresh session з memory re-injection. GDPR Art. 17 erasure: content_encrypted → NULL, memory soft-delete з per-key audit events. Cost attribution: session_cost_usd at turn-write time + `tenant_monthly_coaching_cost` materialized view (pg_cron CONCURRENTLY). 7 DEC-030 HMAC-chained audit events. SOC 2 mapping: CC6.1/CC6.7/CC7.2/CC9.2. 15-item implementation checklist (M3/M4). 3 open questions (OQ-COACH-01/02/03).

### Changed
- `docs/DATA_MODEL.md` — v1.1 additions footer оновлений зі §20 і §21 описами.
- `VERSION` → 1.13.0

---

## [1.12.25] — 2026-05-28

### Added
- `content/post-130-tendon-adaptation-loading.md` — Сухожилля не встигають: адаптація сполучної тканини до навантаження. Колагенова ієрархія (тропоколаген → фібрила → фасцикула); механотрансдукція через інтегрини/FAK і Piezo1 Ca²⁺ канали; Kjaer 2009 — синтез колагену 24–72 год, повна інтеграція 70–100+ днів; жорсткість (N/mm) vs CSA як два різних часових вікна адаптації (Bohm et al. 2015); сайт-специфічність (пателярне, ахіллове, ротаторна манжета, Magnusson 2010); вікно небезпеки і механізм тендинопатії; Rio et al. 2015 (ізометрика 5×45s при 70% MVC); Beyer et al. 2015 RCT (HSR = ексцентрика для ахіллового). clinical-safety PASS.

### Changed
- `blog.html` — картка post-130 «Сухожилля не встигають» додана.
- `VERSION` → 1.12.25

---

## [1.12.24] — 2026-05-28

### Changed
- `docs/DATA_MODEL.md` — ToC entry for §21 AI Coaching Session & Memory Schema added; document header version v1.0 → v1.1.
- `VERSION` → 1.12.24

---

## [1.12.23] — 2026-05-28

### Changed
- `docs/DATA_MODEL.md` — §20 White-Label Tenant Branding Schema added. Closes the gap where `SSO_SCIM_IMPLEMENTATION.md §4.5` referenced `tenants.custom_domain` but no column or dedicated table existed. New `tenant_branding` table: 1:1 with `tenants` (lazy instantiation), columns for `logo_url` / `logo_dark_url` / `favicon_url` / `logo_alt_text`, `primary_color` + auto-calculated `primary_color_contrast`, `custom_domain` with DNS-verification state machine (`not_configured` → `pending_dns` → `active` / `error`), `powered_by_removal` (form_admin BYPASSRLS only; form_system WITH CHECK rejects TRUE; ARR gate ≥ $50k + DECISION_LOG ref required), and `email_sender_name` / `email_reply_to`. R2 asset storage at `form-tenant-assets/tenant-assets/{tenant_id}/`; PNG/WebP only (SVG rejected — XSS risk). WCAG AA colour floor: `validatePrimaryColor()` TypeScript implementation using IEC 61966-2-1 relative luminance; ≥ 4.5:1 contrast ratio with `#ffffff` required; `<3.0:1` returns 422 unconditionally; `primary_color_contrast` auto-set to `#ffffff` or `#000000`. "Powered by FORM" enforcement: `powered_by_removal = FALSE` always unless `contract_arr ≥ $50,000` + signed contract amendment + DECISION_LOG ref; attempts without ref emit CRITICAL DEC-030 event + auto PagerDuty P1. Custom domain DNS management: Cloudflare CNAME proxying, CNAME + preflight verification, KV-cached Cloudflare Worker tenant resolution at 60s TTL. RLS: `form_api` tenant-scoped SELECT; `form_system` full read + insert/update with WITH CHECK rejecting `powered_by_removal = TRUE`; `form_api` zero write access. 12 DEC-030 HMAC-chained audit events (STANDARD/HIGH/CRITICAL). Nightly pg_cron consistency check at 04:00 UTC detects white_label/tenant_branding invariant violations → CRITICAL DEC-030 + PagerDuty P1. Endpoint map for 9 API paths (form_api read, form_system writes, form_admin powered-by-removal). SOC 2 mapping: CC7.2 (HMAC audit chain), CC8.1 (change management + ARR gate), CC6.1 (logo access), CC6.2 (DNS verification before activation), CC6.8 (SVG rejection). 3 open questions (OQ-BRAND-01 email_reply_to domain constraint; OQ-BRAND-02 secondary_color scope; OQ-BRAND-03 domain re-verification automation). 13-item implementation checklist (9× P0 M5, 3× P1 M5/M6, 1× P2 M6). ToC updated with §20 entry. owner: enterprise-architect + compliance-officer + security-engineer.
- `VERSION` → 1.12.23

---

## [1.12.22] — 2026-05-28

### Added
- `content/post-129-bone-adaptation-strength-training.md` — Закон Вольфа і кісткова адаптація: механотрансдукція через остеоцити (Wnt/β-catenin, RANKL/OPG — Robling et al. 2008); BMU-цикл 3–6 місяців (Parfitt 2001); теорія механостату Фроста (1987) з порогами мікродеформації; вікно небезпеки між м'язовою і кістковою адаптацією; Guadalupe-Grau et al. (2009) мета-аналіз BMD; Troy et al. (2018); сайт-специфічність; вік і BMD; практичні правила програмування. clinical-safety PASS.

### Changed
- `README.md` — content roadmap: post-129 відмічений як виконаний; додано теми post-130–133 (м'язово-сухожильна жорсткість, циркадна біологія, спінальні рефлекси, mTORC1 каскад)
- `blog.html` — додано картку post-129 «Закон Вольфа і залізо»
- `STATUS.md` — content engine table: додано рядки post-126–129
- `VERSION` → 1.12.22

---

## [1.12.21] — 2026-05-28

### Added
- `content/post-126-central-neuromuscular-fatigue.md` — Центральна нейром'язова втома: twitch interpolation і voluntary activation (Gandevia 1996); Central Governor Model (Noakes 2004); аферентна сигналізація Group III/IV; серотонін/дофамін-гіпотези; кумулятивна втома (Meeusen 2013 ECSS consensus); практика: RPE-тренд, deload, HRV. clinical-safety PASS.
- `content/post-127-stretch-shortening-cycle-titin.md` — SSC і тітін: три фази (ексцентрична/аморизаційна/концентрична); сухожилля як пружний резервуар; тітін Ca²⁺-залежна взаємодія з актином (Nishikawa 2012); CMJ vs SJ ~8–18% SSC-внесок; паузний vs звичайний присід; complex training і PAP. clinical-safety PASS.
- `content/post-128-motor-unit-synchronization-rate-coding.md` — Синхронізація мотонейронів: Henneman Size Principle (1965); рекрутування vs rate coding; МЕ-синхронізація і пікова сила (Semmler & Enoka 2000); нейронні адаптації (Carroll et al. 2011); bilateral deficit; важкий тренінг для IIb-рекрутування; ізометрії для rate coding. clinical-safety PASS.

### Changed
- `README.md` — content roadmap оновлено: додано записи post-125 через post-128 з анотаціями і clinical-safety статусом
- `VERSION` → 1.12.21

---

## [1.12.18] — 2026-05-28

### Changed
- `enterprise.html` — ROI / business case section added between Pricing tiers and Implementation timeline. Five value levers (utilisation arbitrage, absenteeism reduction at 50% of RAND meta-analysis, productivity uplift at 0.5% of Pronk published effect, turnover avoidance, direct cost comparison) with sourcing notes. 3-year ROI table by tier (Starter 100-seat: 255%, Growth 300-seat: 318%, Enterprise 1,000-seat: 434%) drawn from `docs/COST_MODEL.md §20`. Payback stat pills (2.3–3.4 months, $0 pilot risk, 65% activation SHRM benchmark). Anti-fabrication note linking to custom business case session. Floor-case disclosure (40% activation + 5% absenteeism reduction → 161-175% ROI). Owner: enterprise-architect + compliance-officer.
- `VERSION` → 1.12.18

---

## [1.12.17] — 2026-05-28

### Added
- [`content/post-125-cardiac-adaptations-strength-training.md`](content/post-125-cardiac-adaptations-strength-training.md) — «Серце атлета: чому ЕКГ силовика виглядає "аномально" і чому це норма». Morganroth et al. (1975) Ann Intern Med: ексцентрична (аеробний тренінг) vs концентрична (силовий тренінг) гіпертрофія ЛШ; MacDougall et al. (1985) Circ Res: пік АТ 370/360 мм рт. ст. під час підходу до відмови — транзиторний феномен; Haykowsky et al. (2000) Chest метааналіз: ~2 мм приріст товщини стінки ЛШ при довгостроковому силовому тренінгу; критерії ESC 2010 / Pelliccia: нормальні ЕКГ-варіанти у атлетів vs знахідки, що потребують обстеження; Maron BJ (2009) Circulation: диференційна діагностика серця атлета vs ГКМП — 7 критеріїв (симетричність, розмір порожнини, діастолічна функція, зворотність при детренуванні, LVOT, сімейний анамнез, генетика); паралельне тренування як оптимальний кардіальний профіль; клінічна примітка для симптоматичних атлетів. clinical-safety PASS.
- Картка `blog.html` для post-125 («Серце атлета»).
- Картка `blog.html` для post-124 («Відпочинок між підходами») — картка була пропущена у v1.12.16.
- `STATUS.md` — рядки content engine table для post-124 і post-125.

### Changed
- `VERSION` → 1.12.17

---

## [1.12.15] — 2026-05-28

### Changed
- `docs/SOC2_READINESS.md` v2.9 — three P-series gap advances reflecting compliance artifacts authored 2026-05-21: P-GAP-004 🔴→🟡 (`compliance/p1/retention-decisions.md` P1-RET-001 authored; PRV-23/PRV-26 status updated); P-GAP-007 🔴→🟡 (`compliance/p1/complaint-intake-procedure.md` P1-CIP-001 authored; PRV-52 status updated); P-GAP-002 partial advance (`compliance/p1/sub-processor-register.md` P1-SUB-001 added to §38.5 document list and data room structure). §35.11 gap analysis remediation steps marked ✅ partial. §38.3 Privacy TSC readiness ~82% → ~84%. §38.4 master gap registry updated. §38.5.2 data room p1/ structure annotated with filing status for all five authored artifacts.
- `VERSION` → 1.12.15

---

## [1.12.14] — 2026-05-28

### Added
- [`content/post-123-cross-education.md`](content/post-123-cross-education.md) — «Крос-едукація: тренуєш одну кінцівку — адаптується й інша». Scripture et al. (1894): 130 років феномена; Manca et al. (2017) мета-аналіз 96 РКД, n=1841 — weighted mean CE = 16.1%, d=0.57; Lee & Carroll (2007): три паралельні механізми — двостороння активація M1, спинномозкові перехресні інтернейрони, когнітивна/ментальна компонента (Yue & Cole 1992 — 22% приріст сили від mental imagery); Frazer et al. (2018): ~33% збереження сили у іммобілізованій кінцівці при CE-тренінгу; топографічна специфічність ефекту (Farthing & Zehr 2014); гіпертрофічна CE: Narici et al. (1989) — +8% нетренована vs +13% тренована, Munn et al. (2005) — +2.8% vs +5.4%; модератори ефекту — інтенсивність, проксимальні vs дистальні м'язи, ексцентричний > концентричний, стать (Farthing et al. 2007); 5 практичних наслідків для програмування: травма/іммобілізація, корекція асиметрій, новачки (перший місяць), відновний тренінг, вибір унілатеральних вправ; межі — CE не замінює тренінг нетренованої кінцівки; зв'язок з FORM CV-трекінгом асиметрії. clinical_safety: NOT REQUIRED.
- Картка `blog.html` для post-123 («Крос-едукація»).
- `STATUS.md` — рядок content engine table для post-123.

### Changed
- `VERSION` → 1.12.14

---

## [1.12.13] — 2026-05-28

### Added
- [`content/post-122-pennation-angle-force-production.md`](content/post-122-pennation-angle-force-production.md) — «Кут пір'їстості і сила: що PCSA говорить про твій м'яз і чого не каже». Kawakami et al. (1993, 1995, 1998): θ = 14.6° у тренованих vs 8.4° у нетренованих (vastus lateralis), in vivo ультразвук, поздовжні дані; PCSA = (об'єм × cosθ) / довжина фасцикул — формула і кожен компонент; Wickiewicz et al. (1983) і Friedrich & Brand (1990) архітектурні дані; Azizi et al. (2008): architectural gear ratio і dynamic gearing — чому вищий θ одночасно і збільшує силу, і знижує передачу швидкості до сухожилля; порівняльна таблиця: VL, gastrocnemius medialis, soleus, biceps brachii, biceps femoris longus, rectus femoris (θ, FL, відносний PCSA, функція); чому gastrocnemius важко гіпертрофувати — 4 механізми: базово високий θ, двосуглобова природа, еластичне накопичення енергії сухожилля (Ishikawa et al. 2005), вузьке вікно оптимальної довжини; Aagaard et al. (2001): 14 тижнів → +22% θ, +10% товщина, +18% ізометрична сила; Reeves et al. (2004) у літніх; Franchi et al. (2014): ексцентричне (FL +11.1%) vs концентричне тренування (θ +12.7%) — різні архітектурні адаптації; питома сила (~22.5–30 N/cm²), обмеження PCSA-моделі (нейральний внесок, жорсткість сухожилля, плечо важеля, динаміка vs ізометрія); практичні наслідки для вибору вправ і програмування; відкриті питання: індивідуальна варіативність θ, стеля адаптації, зворотність при детренінгу, статеві відмінності. clinical_safety: NOT REQUIRED.
- Картка `blog.html` для post-122 («Кут пір'їстості і сила»).
- `STATUS.md` — рядок content engine table для post-122.
- `README.md` — post-122 позначено як виконаний у roadmap.

### Changed
- `VERSION` → 1.12.13

---

## [1.12.12] — 2026-05-28

### Added
- [`content/post-121-sarcomerogenesis-muscle-length-adaptation.md`](content/post-121-sarcomerogenesis-muscle-length-adaptation.md) — «Саркомерогенез: як м'яз змінює свою довжину у відповідь на тренінг». Lynn & Morgan (1994): послідовне додавання саркомерів при ексцентричному навантаженні і зміщення θ_opt у бік довших довжин; Gordon et al. (1966) length-tension relationship і L_opt як механістична основа; Morgan (1990) теоретична модель протекції від DOMS через SSN; Butterfield (2010) підтвердження; Brughelli & Cronin (2007) огляд: порівняльна таблиця ексцентричного vs концентричного впливу на SSN, довжину фасцикул, θ_opt, кут пір'їстості і PCSA; Nordic Hamstring Curl як найкраще задокументований прикладний кейс — Mjøsund et al. (2021) мета-аналіз: +11% довжина фасцикул, −51% повторних пошкоджень підколінних; Guex et al. (2016) пряме порівняння NHC vs conventional; практичний вибір вправ: RDL для підколінних vs leg curl (довге vs коротке положення), incline dumbbell curl для біцепса; практичні наслідки для тестування (кут при піковій силі зміщується з SSN — ізокінетичний тест при фіксованому куті може занижувати адаптацію); програмування: ексцентрична фаза і повна амплітуда як архітектурні стимули; відкриті питання (ceiling для адаптації SSN, м'язова специфічність, роль тайтину, зворотність при детренінгу). clinical_safety: NOT REQUIRED.
- Картка `blog.html` для post-121 («Саркомерогенез: як м'яз змінює свою довжину у відповідь на тренінг»).
- `STATUS.md` — рядок content engine table для post-121.
- `README.md` — post-121 позначено як виконаний у roadmap.

### Changed
- `VERSION` → 1.12.12

---

## [1.12.11] — 2026-05-28

### Changed
- [`docs/INCIDENT_RESPONSE.md`](docs/INCIDENT_RESPONSE.md) — v0.5 → v0.8: додано R-14 DSAR / Data Subject Rights Incident (чотирнадцятий runbook). Closes 🔴 Gap "Response time SLA for DSAR" і 🟡 Gap "DSAR process" із SOC2_READINESS.md; виконує §10.1.5 GDPR_DPIA.md ("DSAR handling runbook"). R-14 покриває 7 сценаріїв (P0–P2): cross-tenant export contamination (+ R-01 паралельно), Art. 9 erasure failure (+ R-11 для keypoints_enc), missed 30-day deadline, export worker failure, bulk/abuse DSAR, enterprise employee DSAR (joint-controller), DPA complaint received. Scope assessment SQL для in-flight DSAR register + erasure completeness. 14 DEC-030 HMAC-chained audit events (`dsar.request_received` → `dsar.export_link_revoked` CRITICAL → `dsar.erasure_manual_supplement` CRITICAL). 5 communication templates (D-01 DPA voluntary notification, D-02 user delay notice, D-03 Art. 12(5) refusal, D-04 employee employer redirect, E-DSAR-01 enterprise tenant). Privacy floor: FORM не може виконати employer instruction що гасить Art. 15 right of employee. SOC 2 mapping: P4.0, P5.0, P5.1, P8.0, CC2.2, CC6.5. Evidence package: `compliance/evidence/dsar/YYYY-MM/<dsar_id>/` 7-year retention. 6 preventive controls. Tabletop Scenario I: cross-tenant contamination 420-user Growth tenant (UK/Ireland), 10 discussion points. Виправлено header version v0.5 → v0.8 (проміжні v0.6, v0.7 були внесені не по порядку і не оновлювали header). Виправлено owner в header: `devops-lead` → `compliance-officer` (відповідно до footer з v0.5). Appendix A quick reference: додано R-14 quick-ref рядки.
- [`docs/SOC2_READINESS.md`](docs/SOC2_READINESS.md) — DSAR process 🟡 Gap → 🟡 Partial (ref: R-14); Response time SLA 🔴 Gap → 🟡 Partial (ref: R-14.4; технічний тест P-GAP-005 ще pending).
- `VERSION` → 1.12.11

---

## [1.12.10] — 2026-05-28

### Added
- [`content/post-120-myofibrillar-vs-sarcoplasmic-hypertrophy.md`](content/post-120-myofibrillar-vs-sarcoplasmic-hypertrophy.md) — «Міофібрилярна vs саркоплазматична гіпертрофія: де наука, а де міф». Schoenfeld (2010): відсутність прямих доказів вибіркової гіпертрофії; Haun et al. (2019) PLOS ONE: перше пряме дослідження фракцій (глікоген і саркоплазматичні білки при великому обсязі), методологічні обмеження; що реально доведено — кілька компонентів гіпертрофії, накопичення глікогену, індивідуальна варіативність складу; що залишається гіпотезою — вибіркова активація типу гіпертрофії залежно від зони повторень; порівняльна таблиця реальних відмінностей важкого vs легкого тренінгу; відкриті питання (controlled trial з биопсіями, функціональне значення різниці, роль тайтину); clinical_safety: NOT REQUIRED.
- Картка `blog.html` для post-120 («Міофібрилярна vs саркоплазматична гіпертрофія»).
- `STATUS.md` — рядок content engine table для post-120.
- `README.md` — post-120 позначено як виконаний у roadmap.

### Changed
- `VERSION` → 1.12.10

---

## [1.12.9] — 2026-05-28

### Added
- [`content/post-119-rep-range-continuum.md`](content/post-119-rep-range-continuum.md) — «Континуум сила-гіпертрофія: що насправді визначає адаптацію». Спростування зон-міту (1–5=сила, 8–12=гіпертрофія, 15+=витривалість); Schoenfeld et al. (2017) — еквівалентна гіпертрофія при низькому (25–35 повт.) і помірному (8–12 повт.) навантаженні за умови близькості до відмови; Lasevicius et al. (2018) і Schoenfeld & Grgic (2019) meta-аналіз — гіпертрофія в діапазоні 6–30 повторень при RIR 0–4; три механізми гіпертрофії: механічна напруга (primary, mTOR/MAPKs, Schoenfeld 2010), метаболічний стрес, пошкодження м'язів (переглядається, DOMS ≠ якість); нейральні адаптації при важких навантаженнях (1–5RM) — рекрутування мотонейронів і синхронізація; практичний приклад (присідання варіант А vs Б); рекомендації для проміжного ліфтера: 4–6 повт. базові / 8–15 допоміжні / 15–30 ізоляція; RIR 1–3 як орієнтир авторегуляції; Schoenfeld & Krieger (2017) обсяг 10–20 підходів/тиждень. clinical_safety: PASS.

### Changed
- `VERSION` → 1.12.9

---

## [1.12.8] — 2026-05-28

### Added
- [`content/post-118-deload-strategies.md`](content/post-118-deload-strategies.md) — «Підводка і розвантаження: доказова база деловдів». Три типи деловду: зниження об'єму (−40–50% сетів, збережена інтенсивність 90–95%) vs зниження інтенсивності vs повний відпочинок; доказова база Schoenfeld et al. про збереження нервово-м'язових адаптацій при підтримці інтенсивності; Ogasawara et al. (2013) — перерва 3 тижні не знижує гіпертрофію відносно безперервного тренінгу; проактивний деловд (кожні 4–6 тижнів для просунутих, 6–8 для проміжних) vs реактивний (симптоматичний); практичні маркери для реактивного деловду (grip strength −5–10%, HRV нижче базової 7+ днів, RPE +1–2 одиниці без причини, хронічно низька мотивація, деградація техніки); помилка "деловд-як-стиль-життя"; ліміт 7–10 днів. Посилання на post-117 (модель Банністера). clinical_safety: PASS.

### Changed
- `VERSION` → 1.12.8

---

## [1.12.7] — 2026-05-28

### Changed
- [`docs/COST_MODEL.md`](docs/COST_MODEL.md) — §20 Customer ROI Model & Business Case Template added (patch extension). Buyer-facing financial justification tool for enterprise sales: status quo cost baseline (Willis Towers Watson / Mercer benchmarks, $43–90/seat/month at 20% utilisation); five quantifiable value levers (utilisation arbitrage 3.25×, absenteeism reduction 14% conservative vs RAND 28%, productivity uplift 0.5%, turnover avoidance, direct cost comparison); three-year ROI tables by tier (Starter 100-seat 255% ROI 3.4-month payback / Growth 300-seat 318% 2.9-month / Enterprise 1,000-seat 434% 2.3-month); three sensitivity analyses showing floor case (40% activation + 5% absenteeism reduction) still yields 161–175% ROI; non-financial value drivers (CSRD/ESG, talent acquisition, EEOC/HIPAA/ADA risk avoidance, culture signal); copy-paste business case template for internal champion; sales champion usage notes (lead with absenteeism, disclose conservative basis, no fabricated case studies). OQ-18 added: pilot activation instrumentation requirement. OQ-19 added: ICP salary band discovery question.
- `VERSION` → 1.12.7

---

## [1.12.6] — 2026-05-28

### Added
- [`content/post-117-fitness-fatigue-model.md`](content/post-117-fitness-fatigue-model.md) — «Модель Баністера: чому після важкого блоку форма росте — але потім». Banister et al. (1975) dual-factor impulse-response model: fitness component τ₁ ≈ 42 дні і fatigue component τ₂ ≈ 13 днів; Morton et al. (1990) Mathematical Modeling of Cardiovascular Fitness — верифікація на плавцях, k₂ > k₁ (втома пригнічує більше, ніж адаптація піднімає короткостроково); Busso (2003) нелінійне розширення і індивідуальна варіативність τ₁/τ₂; Zatsiorsky & Kraemer — блокова структура як практичне втілення dual-factor; Bosquet et al. (2007) meta-analysis (27 досліджень, ~2000 атлетів): оптимальний tapering +2–3% performance, –41–60% обсягу, 8–14 днів, збережена інтенсивність; Plews et al. (2013) HRV як proxy для h(t) fatigue component; практична структура мезоциклу accumulation/intensification/deload/peak; обмеження моделі. clinical_safety: PASS (виключно тренувальна наука і фізіологія адаптації).
- `blog.html` — картка Post 117 (data-cat="science").
- `STATUS.md` — рядок post-117 у content engine table.
- `README.md` — пост post-117 відмічено [x]; додано 5 нових тем до роадмапу (post-118–122): відпочинок між підходами, drop sets, міофібрилярна vs саркоплазматична гіпертрофія, sarcomerogenesis, кут пір'їстості.

### Changed
- `VERSION` → 1.12.6

---

## [1.12.5] — 2026-05-24

### Added
- [`content/post-116-hypertrophy-responder-variability.md`](content/post-116-hypertrophy-responder-variability.md) — «Варіативність гіпертрофічної відповіді: чому одна програма дає різні результати». Hubal et al. (2005) — 585 учасників, розкид CSA 0–+59% і сили 0–+250% при однаковому протоколі; Bamman et al. (2007) кластерний аналіз: extreme responders (~26%), moderate (~51%), non-responders (~23%); Petrella et al. (2008) — базальний рівень сателітних клітин передбачає гіпертрофічну відповідь; генетика: ACTN3 R577X і волокна Type II, міостатин як негативний регулятор (Schuelke 2004), полігенна природа потенціалу; Wakahara et al. (2013) — регіональна неоднорідна гіпертрофія; Churchward-Venne et al. (2015) — non-responders на низькому обсязі стають responders при вищому; успадковуваність 50–80% (HERITAGE Family Study); практичні наслідки: горизонт оцінки 12–16 тижнів, обсяг до MEV/MAV порогу, сила ≠ гіпертрофія. clinical_safety: PASS (виключно тренувальна фізіологія і генетика).
- `blog.html` — картка Post 116 (data-cat="science").
- `STATUS.md` — рядки post-114, post-115, post-116 у content engine table.

### Changed
- `VERSION` → 1.12.5

---

## [1.12.4] — 2026-05-24

### Added
- [`content/post-114-training-volume-hypertrophy.md`](content/post-114-training-volume-hypertrophy.md) — «Обсяг тренувань і гіпертрофія: крива доза-відповідь». Krieger (2010) мета-аналіз (multiple vs single sets), Schoenfeld/Ogborn/Krieger (2017) доза-відповідь (10+ сетів/м'яз/тиждень), Radaelli et al. (2015) 6-місячне RCT (5 > 3 > 1 сет), MEV/MAV/MRV операційна модель (Israetel), ознаки перевищення MRV, зворотній зв'язок інтенсивності і обсягу. clinical_safety: PASS (виключно тренувальна наука).
- [`content/post-115-training-frequency-hypertrophy.md`](content/post-115-training-frequency-hypertrophy.md) — «Частота тренувань і гіпертрофія: нюанси важливіші за правило». Schoenfeld/Ogborn/Krieger (2016) 2x > 1x (ES 0.37 vs 0.14, але обсяг неурівняний), Ralston et al. (2017) при рівному обсязі ефект частоти зникає, McLester et al. (2000) деградація якості пізніх сетів, Colquhoun et al. (2018) 6x = 3x, вікно MPS (~24–48h), частота як механізм розподілу обсягу, full-body vs upper/lower vs bro split аналіз. clinical_safety: PASS (виключно тренувальна наука).
- `blog.html` — картки Post 114 і Post 115 додані до сітки блогу (data-cat="science").

### Changed
- `VERSION` → 1.12.4

---

## [1.12.3] — 2026-05-24

### Added
- [`docs/DATA_MODEL.md`](docs/DATA_MODEL.md) §19 — Feature Flag & Entitlement Registry Schema (doc bumped v0.9 → v1.0): closes the §2.7 stub gap that left `tenant_feature_flags` without registry enforcement, tier gating, compliance gates, or DEC-030 audit trail. Two-table design: `feature_flag_registry` (canonical allowlist, `form_admin`-only write, FK-enforced dotted namespace via `flag_key_namespace` CHECK constraint) + `tenant_feature_flags` (production per-tenant rows with FK to registry, `compliance_approval_ref`, `enabled_at`, RLS read-only for `form_api` and `form_system`). Seed data for 14 canonical flags across Starter / Growth / Enterprise tiers (`sso.saml_enabled`, `sso.oidc_enabled`, `scim.provisioning_enabled`, `admin.dashboard_enabled`, `audit.rest_export_enabled`, `webhook.outbound_enabled`, `cv.client_side_enabled`, `audit.siem_export_enabled`, `audit.s3_sync_enabled`, `admin.cohort_analytics_enabled`, `branding.white_label_enabled`, `sso.staging_env_enabled`, `security.ip_allowlist_enabled`, `webhook.email_in_payloads`). Tier entitlement matrix: auto / avail / — / 🔒 gate per flag × tier. TypeScript `assertFeatureEnabled()` in `src/workers/feature-flags/assert-feature-enabled.ts`: Cloudflare KV 30-second cache, `TIER_ORDER` rank map, fail-closed on any error (throws 403). Four DEC-030 HMAC-chained audit events: `feature_flag.enabled` (STANDARD, 7yr), `feature_flag.disabled` (STANDARD, 7yr), `feature_flag.registry_updated` (HIGH, 7yr), `feature_flag.compliance_gate_bypassed` (CRITICAL, 10yr — auto PagerDuty P1). Compliance gate protocol: DPA authorisation → DECISION_LOG PR → `compliance_approval_ref` populated → `form_admin` enables → DEC-030 event. §19.9 migration script from §2.7 stub: 14 flag_name → flag_key renames, `RAISE EXCEPTION` abort on unrecognised flags, FK constraint addition, `enabled_at` backfill. OQ-FLAG-01 tier-downgrade auto-disable hook (M5). OQ-FLAG-02 deprecation sunset (M6). SOC 2: CC6.1/CC6.2/CC8.1/CC9.2. 12-item implementation checklist (8× P0 M3).

### Changed
- `VERSION` → 1.12.3

---

## [1.12.2] — 2026-05-24

### Added
- [`content/post-113-eccentric-overload.md`](content/post-113-eccentric-overload.md) — Ексцентрична сила: Katz (1939) ієрархія F_eccentric > F_isometric > F_concentric і актоміозинова кінетика; ексцентричний max +20–50% vs концентричний (Dvir 2004; Grange & Houston 1991); Enoka (1996) — менший EMG при вищій ексцентричній силі; Roig et al. (2009) мета-аналіз: ексцентричний акцент → більша гіпертрофія (ES 0.79 vs 0.52) без втрати концентричної сили; Franchi et al. (2014) — дистальна гіпертрофія і довші фасцикули при ексцентричному акценті; Nordic Hamstring Exercise: Timmins et al. (2016) оптимальний кут і FL; Har-Nir et al. (2022) −51% травм підколінних (OR 0.49); supramaximal loading механіка; flywheel: Vicens-Bordas et al. (2018) ексцентрична сила +10.2% vs +3.8%; Duchateau & Enoka (2008) нейральна специфічність ексцентрики; Repeated Bout Effect і дозування; clinical-safety PASS
- `blog.html` — картки для post-110, post-111, post-112, post-113
- `README.md` — post-113 [x] у roadmap
- `STATUS.md` — рядки post-110–post-113

### Changed
- `VERSION` → 1.12.2

---

## [1.12.1] — 2026-05-24

### Added
- [`content/post-110-muscle-architecture.md`](content/post-110-muscle-architecture.md) — Архітектура м'яза: кут пір'їстості, довжина пучка, PCSA; clinical-safety PASS
- [`content/post-111-post-activation-potentiation.md`](content/post-111-post-activation-potentiation.md) — PAP механізм (RLC фосфорилювання), вікно 4–12 хв, complex/contrast training; clinical-safety PASS
- [`content/post-112-rpe-vs-percentage-autoregulation.md`](content/post-112-rpe-vs-percentage-autoregulation.md) — RPE vs %1RM, авторегуляція навантаження, Zourdos/Helms валідація; clinical-safety PASS

### Changed
- `README.md` — content roadmap +posts 110–112
- `VERSION` → 1.12.1

---

## [1.12.0] — 2026-05-24

### Added
- [`docs/OBSERVABILITY.md`](docs/OBSERVABILITY.md) §22 — AI Coaching Quality Observability (doc bumped v0.8 → v0.9): closes the gap between operational observability (§3/§4.5 — tokens, latency) and quality observability (is Victor coaching well?). Privacy-first design: GDPR Art. 9 constraint enforced — no coaching content in any observability signal; all metrics are behavioral proxies computed at session or weekly cohort level via pg_cron. Ten-metric quality proxy taxonomy: plan adherence rate, session depth (turns), session abandonment rate, post-coaching workout start rate, workout completion rate (post-coaching), retry rate within session, plan regeneration rate, explicit negative feedback (thumbs-down), no-engagement rate (7d), streak grace trigger rate — all computed in `victor_quality_daily` table with k-anonymity floor (sample_size ≥ 5 before row is written). Prompt version tracking: `victor_prompt_version` + `prompt_ab_bucket` columns on `coaching_turns`; 14-day baseline window for regression detection. Six QA-COACH-SLOs: plan adherence ≥ 65%, session abandonment ≤ 30%, post-coaching workout start ≥ 40%, explicit negative feedback ≤ 8%, plan regen ≤ 15%/user/week, no-engagement ≤ 40%. Five clinical safety monitoring signals (session duration spike, retry spike, no-action spike, thumbs-down P0 at 25%, explicit harm report) with clinical-safety VETO authority path via INCIDENT_RESPONSE.md R-10. Six alerting rules AL-COACH-01 through AL-COACH-06 with 72h suppression window after prompt deploys (prevents small-sample false positives). A/B testing observability: `prompt_ab_bucket` tenant-locked random split, 14-day / 500-session minimum, two-proportion z-test α=0.05, `victor_ab_results` table DDL. Four DEC-030 HMAC-chained events for prompt lifecycle. Metabase "Victor Coaching Quality" dashboard spec (8 panels). `victor_quality_daily` DDL with UNIQUE constraint + RLS (founder + ml-engineer + clinical-safety; tenant_admin scoped; form_system write) + 24-month retention cleanup. SOC 2 mapping: CC7.2 (quality monitoring), CC7.4 (prompt regression via victor_ab_results), CC1.2 (jailbreak CI gate AL-COACH-05), CC5.2 (clinical safety escalation), C1.1 (no coaching content in observability). Twelve-item implementation checklist across M3/M4/M6.

### Changed
- `docs/OBSERVABILITY.md` header — v0.8 → v0.9
- `VERSION` → 1.12.0

---

## [1.11.0] — 2026-05-24

### Added
- [`docs/SSO_SCIM_IMPLEMENTATION.md`](docs/SSO_SCIM_IMPLEMENTATION.md) §18 — Tenant IdP Migration Runbook (doc bumped v0.9 → v1.0): end-to-end procedure for migrating an active enterprise tenant between identity providers without fragmenting user history or roles. Migration trigger table (6 scenarios: M&A vendor replacement, IdP vendor switch, SAML → OIDC protocol upgrade, SCIM-only migration, custom domain change, on-premises ADFS → cloud). Pre-migration assessment: 6 FORM-side checks (F-01 staging prerequisite → F-06 founder authorisation) and 6 customer IT checks (C-01 new app in staging → C-06 maintenance window). Two new tables: `sso_idp_migrations` (state machine planning→staging→cutover_pending→committed/rolled_back, exclusion constraint preventing concurrent active migrations per tenant, BYPASSRLS form_admin) and `sso_external_id_mappings` (old↔new external_id linkage, form_admin only, never API-exposed). Three migration type runbooks: Type A same-protocol different-vendor (SAML field swap, two-person SQL remap gate), Type B SAML→OIDC protocol upgrade (full SP rebuild, NameID→sub resolution, SAML SLO cleanup), Type C SCIM-only (dual-token 24h overlap, group external_id update). SCIM identity preservation: email-match strategy (default; staging SCIM sync keyed by email, SQL with 0-unmatched gate before cutover) and manual ID mapping (M&A email-change scenario; CSV import with two-person review; 20% manual-rate escalation threshold). Rollback: 4 trigger conditions, reverse-remap SQL, post-incident review gate. Post-migration validation: 8 checks (V-01 SSO error rate, V-02 login coverage, V-03 no duplicates, V-04 role spot-check, V-05 SCIM deprovision smoke test, V-06 group mapping, V-07 HMAC chain integrity, V-08 RLS isolation CI). Seven new DEC-030 HMAC-chained events: `sso.migration_initiated` (HIGH), `sso.migration_staging_validated` (MEDIUM), `sso.migration_production_committed` (HIGH), `sso.migration_rolled_back` (HIGH), `sso.external_id_remapped` (HIGH — id values excluded from log payload by design), `sso.migration_old_config_decommissioned` (MEDIUM), `sso.scim_token_rotated_for_migration` (HIGH). SOC 2 mapping: CC8.1 (state machine + founder auth + two-person SQL gate), CC6.1 (remap prevents duplicate-account access), CC6.2 (is_active gate during cutover window), CC7.2 (OBSERVABILITY §13 rollback signal), CC9.2 (48h rollback window + decommission event), C1.1 (external_id values excluded from audit log). G-004 status 🟡 Partial → 🟡 Partial (§18 covers certificate-replacement scenarios; closes to 🟢 on expiry cron + admin UI). Implementation checklist: 12 tasks (7× P0, 4× P1, 1× P2), M5/M6.

### Changed
- `docs/SSO_SCIM_IMPLEMENTATION.md` header — v0.8 → v1.0
- `VERSION` → 1.11.0

---

## [1.10.2] — 2026-05-24

### Added
- [`content/post-109-force-velocity-curve.md`](content/post-109-force-velocity-curve.md) — Крива сила-швидкість: Hill (1938) гіперболічне рівняння і чому максимальна потужність досягається при 30–45% 1RM а не при максимальній силі; explosive strength deficit (ESD, Zatsiorsky); F-V профілювання за Samozino et al. (2012, 2014) і Morin & Samozino (2016); діагностика силового vs швидкісного дефіциту; Cormie et al. (2011) оптимальне навантаження для потужності; Haff & Nimphius (2012); Skelton et al. (1994) RFD і вікові зміни; блокова логіка для аматора; 13-хв читання; clinical-safety PASS
- `blog.html` — картка для post-109
- `README.md` — post-109 позначений [x] у roadmap

### Changed
- `VERSION` → 1.10.2

---

## [1.10.1] — 2026-05-24

### Added
- [`content/post-107-hypertrophy-mechanisms-mechanical-tension.md`](content/post-107-hypertrophy-mechanisms-mechanical-tension.md) — Три механізми гіпертрофії: ієрархія між mechanical tension (первинний, mTORC1 → Wackerhage et al. 2019), metabolic stress (допоміжний, Schoenfeld 2013), та muscle damage (побічний ефект, не обов'язковий тригер); repeated bout effect як ключовий контраргумент; Burd et al. (2012) і Mitchell et al. (2012) дані MPS; практичні висновки — повний ROM, RIR 0–2, без гонитви за DOMS; 9-хв читання; clinical-safety PASS
- [`content/post-108-lengthened-partials-stretch-hypertrophy.md`](content/post-108-lengthened-partials-stretch-hypertrophy.md) — Гіпертрофія при розтягнутих положеннях: Maeo et al. (2021) підколінні lengthened vs shortened; Pedrosa et al. (2022) квадрицепс; Kassiano et al. (2023) мета-аналіз — lengthened position > shortened partials; titin механізм (Ca²⁺ sensitization, sarcomerogenesis); Schoenfeld & Grgic (2020) ROM review; exercise selection по групах (RDL для підколінних, incline curl для біцепса, overhead extension для трицепса, повні присідання для квадрицепса); 13-хв читання; clinical-safety PASS
- `blog.html` — картки для post-107 і post-108
- `README.md` — post-107 і post-108 позначені [x] у roadmap

### Changed
- `VERSION` → 1.10.1

---

## [1.10.0] — 2026-05-24

### Added
- [`docs/COST_MODEL.md`](docs/COST_MODEL.md) §19 — Enterprise Go-to-Market Financial Model (doc bumped v0.9 → v1.0): sales velocity equation for founder-led and AE-led phases; pipeline stage conversion model with hard exit gates and BANT triggers; founder time-budget per deal (37 hrs); first AE hire economics ($120k OTE, 3.8× quota multiple, $179k net GP/yr, Month-5 ramp recovery); SDR layer economics ($70k OTE, 2.1× ROI); CSM cost absorption bridge (< 5% of revenue at Growth 700+ seats, Enterprise 1,500+ seats); three-scenario Month 1–24 enterprise ARR forecast (Base $188k, Bull $312k, Bear $86k); Series A quality-signal framing. OQ-16 (Growth minimum seat floor) and OQ-17 (win rate calibration) added.

### Changed
- `docs/COST_MODEL.md` header — v0.9 → v1.0
- `VERSION` → 1.10.0

---

## [1.9.24] — 2026-05-24

### Added
- [`content/post-106-women-strength-anatomy.md`](content/post-106-women-strength-anatomy.md) — Жіночий підхід до силового: Q-angle myth (Collins et al. 2009), estrogen і tendon laxity (Lowe 2017), ACL-ризик з base-rate контекстом (Myklebust 2003, NCAA 2016), recovery differences (Hicks et al. 2001), cycle-phase periodization як contested area; 13-хв читання; clinical-safety PASS WITH CONDITIONS
- `blog.html` — картка для post-106
- `STATUS.md` — рядки post-104, post-105, post-106 у content engine table
- `README.md` — post-106 позначено [x] у roadmap

### Changed
- `VERSION` → 1.9.24

---

## [1.9.23] — 2026-05-24

### Added
- [`content/post-104-dehydration-performance.md`](content/post-104-dehydration-performance.md) — Дегідрація і продуктивність: Savoie et al. (2015), González-Alonso et al. (1997), ACSM urine-color protocol, hyponatremia warning; clinical-safety PASS WITH CONDITIONS
- [`content/post-105-grip-strength-longevity.md`](content/post-105-grip-strength-longevity.md) — Grip strength як маркер здоров'я: Bohannon (2019) мета-аналіз, Leong et al. (2015) PURE, Celis-Morales (2018) UK Biobank, NHANES нормативи, 4 методи тренування
- `blog.html` — картки для post-104 і post-105

### Fixed
- `README.md` — roadmap: виправлено розрив post-103 (Жіночий підхід → post-106); post-104 і post-105 позначені [x]

---

## [1.9.22] — 2026-05-24

### Changed
- `docs/OBSERVABILITY.md` v0.7 → v0.8 — added §21 ClickHouse Analytics Observability (M9): closes the documented gap from §10 ("ClickHouse for analytics — observability strategy not yet designed"). Covers migration trigger thresholds (PostHog cost P0, query latency P0, MAU/tenant/volume advisory signals); ClickHouse Cloud EU-Central architecture with GDPR DPA; RED metrics table + eight alerting rules (AL-CH-01 through AL-CH-09) for ClickHouse itself; OpenTelemetry Collector pipeline (Fly.io EU) with PII filter processor; `traces` / `events` / `sessions` MergeTree DDL; BLOCKED_PROPERTY_KEYS deny-list preventing health values at ingest; six analytics SLOs (CH-SLO-01 through CH-SLO-06) with error budget burn rates; per-tenant ClickHouse Row Policies with break-glass cross-tenant access logging to DEC-030 HMAC chain; three-phase PostHog migration runbook; PostHog data deletion for GDPR Art. 5(1)(e) compliance; privacy constraints matrix (six constraints); nightly PII scan (AL-CH-09, P0); SOC 2 CC7.2/CC6.1/CC6.2/C1.2/PI1.2/A1.1/CC9.2 evidence mapping; six auditor artefact files; sixteen-item M7-prep + M9 implementation checklist.
- `VERSION` → 1.9.22

---

## [1.9.21] — 2026-05-24

### Added
- `content/post-103-readiness-diagnostics.md` — four-dimensional readiness framework: neuromuscular (VBT/RPE deviation during warm-up sets), systemic-autonomic (HRV RMSSD vs 7-day rolling baseline, Plews et al. 2013: >20% drop = suboptimal recovery), local/structural (eccentric DOMS peak at 24–48h, Clarkson & Hubal 2002: strength reduction 10–40% for up to 7 days), cognitive-neural (Pageaux & Lepers 2018: cognitive fatigue → reduced voluntary neural drive ~85–90% of max). Decision matrix: full session / volume reduction (20–30%) / intensity reduction / active recovery. Meeusen et al. (2013): both NFO risk from training through suppressed readiness AND undertraining risk from over-cautious avoidance framed. Flatt et al. (2017): 7-day rolling mean eliminates single-day noise. Pre-session protocol: 2-min HRV + 5-min warm-up diagnostics. FORM wearable integration angle: automated baseline, rolling comparisons, per-session load adaptation. clinical-safety PASS.
- `blog.html` — card for post-103 (readiness diagnostics) added
- `STATUS.md` — entries for post-101, post-102, post-103 added to content engine table; blog.html count updated (97 cards, 103 posts authored)
- `VERSION` → 1.9.21

---

## [1.9.20] — 2026-05-24

### Added
- `content/post-101-sleep-recovery-mechanisms.md` — sleep physiology for strength athletes: GH pulsatile secretion (70% during SWS via Veldhuis & Bowers 2010), GH → IGF-1 → PI3K/Akt/mTORC1 signaling cascade, Leproult & Van Cauter (2011) JAMA: 1-week 5h sleep → −10–15% daytime testosterone; cortisol elevation via HPA axis → ubiquitin-proteasome activation + mTORC1 suppression; evidence-graded sleep hygiene table (Level A/B/C: circadian entrainment, darkness, 16–19°C temperature, caffeine cutoff, alcohol SWS fragmentation, pre-sleep training cutoff); "one bad night" section preventing anxiety framing using Van Dongen et al. 2003 cumulative debt data; FORM wearable proxy signals (HRV rMSSD trend, RPE anomaly detection, sleep duration not sleep stages from consumer devices); clinical-safety PASS
- `content/post-102-heat-cold-training-performance.md` — temperature as performance modifier: Q10 enzymatic kinetics (10°C drop → 2× slower reactions), cross-bridge cycling temperature dependence; heat mechanisms — CNS fatigue acceleration above 38°C core temp (Nielsen et al. 1993) + cardiovascular output competition; Jensen (2013) strength reduction in heat; cold mechanisms — Bergh & Ekblom (1979): 30°C vs 39°C → ~30% peak power reduction; increased muscle viscosity + injury risk (McGovern 2009); temperature × strength output table (optimal 15–25°C); warm-up duration table by ambient temperature (8–25+ min); heat adaptation (Périard et al. 2015: 10–14 days → plasma volume expansion); FORM RPE vs temperature load correction; no hyperthermia/hypothermia triggers; clinical-safety PASS
- `blog.html` — cards for post-101 (sleep/recovery mechanisms) and post-102 (heat/cold performance modifiers) added before empty-state
- `README.md` — post-102 marked [x] complete
- `VERSION` → 1.9.20

---

## [1.9.19] — 2026-05-24

### Changed
- `docs/OBSERVABILITY.md` §20 — Wearable Data Integration Observability (v0.6→v0.7). Five-source wearable pipeline (HealthKit, Health Connect, Whoop, Oura Ring, Garmin) added to observability scope. RED metrics table per provider with Workers Analytics Engine counter names (`wearable_sync_success_total`, `wearable_sync_failures_total`, `wearable_cron_heartbeat`, etc.). Eight freshness and operational SLOs (WEAR-SLO-01 through WEAR-SLO-08) with differentiated targets per sync mechanism: HealthKit ≥ 95% freshness-within-25h, Health Connect ≥ 90% (WorkManager 15-min OS floor), Whoop/Oura ≥ 92%, Garmin ≥ 88% (500 req/day quota constraint); OAuth token validity ≥ 99.5% (WEAR-SLO-06); cron run completion ≥ 99% (WEAR-SLO-07); HRV normalization valid-output rate ≥ 98% (WEAR-SLO-08). OAuth token health state machine (five states: healthy/expiry-approaching/expired/refresh-failed/revoked-by-user). `wearable_sync_health` Postgres table DDL — operational metadata only (timestamps, failure counts, `sync_status` enum, `error_code_last`; no health values); RLS blocking tenant_admin direct access; 90-day pg_cron retention. Eight alerting rules AL-WEAR-01 through AL-WEAR-08: fleet failure spikes → P1 PagerDuty (HealthKit >12%, Health Connect >18%, third-party API >20%, token refresh systemic >5%); quota risk and individual expiry → P2 Slack `#platform-alerts`; cron heartbeat absent → P1 PagerDuty (AL-WEAR-07); per-enterprise-tenant stale rate >30% → P2 Slack `#csm-alerts` with privacy-safe payload (no user_id). `wearable_fleet_health` materialized view with k-anonymity HAVING n ≥ 5 and tenant_admin RLS; daily 05:30 UTC pg_cron refresh. Admin dashboard "Wearable Integration Health" widget spec (five panels: source counts, sync status stacked bars, median freshness gauge, 14-day P90 trend, attention banner with error bucketing — no individual user exposure). Six DEC-030 HMAC-chained audit events (`wearable.source_connected/disconnected`, `sync_completed` count-only, `sync_blocked`, `token_refresh_failed`, `permission_revoked`) aligned with DATA_MODEL.md §14.7. Privacy constraints matrix (five constraints with enforcement locations: no raw values in observability, no GROUP BY user_id, k-anonymity n ≥ 5, tenant_admin blocked from raw rows, count-only DEC-030 payloads). SOC 2 CC7.2/PI1.2/C1.2/A1.1/CC9.2 evidence mapping with three compliance artefact export paths (`wearable-fleet-YYYY-MM-DD.csv`, `wearable-slo-YYYY-MM.csv`, `wearable-sync-blocks/YYYY-MM.csv`). Thirteen-item implementation checklist across M4/M5. Header corrected v0.5 → v0.7 (§19 had bumped to v0.6 without updating the document header).
- `VERSION` → 1.9.19

---

## [1.9.18] — 2026-05-23

### Added
- `content/post-98-accommodating-resistance.md` — variable resistance mechanics: force curve matching, band/chain loading (25–35% of total load), Baker & Newton (2005), Rhea et al. (2009), Westside context, sticking point overload protocol; clinical-safety PASS
- `content/post-99-sticking-points.md` — sticking point mechanics in squat (parallel — hip extensors, McLaughlin 1978), bench (two-phase van den Tillaar & Ettema 2013), deadlift (below/above knee); weak chain vs technique vs load diagnosis; targeted accessory selection; clinical-safety PASS
- `content/post-100-e1rm-vs-rep-ranges.md` — E1RM milestone: Epley/Brzycki/Lombardi formulas; accuracy degradation past 5 reps (Reynolds et al. 2006); velocity-based estimation MPV (Jidovtseff et al. 2011); practical case for rep-range programming; clinical-safety PASS

### Changed
- `blog.html` — added cards for post-97, post-98, post-99, post-100 (four new Science entries)
- `docs/DECISION_LOG.md` — DEC-032 (no-backdoor law enforcement policy formalized), DEC-033 (post-100 milestone: quality over velocity)
- `README.md` — post-98, post-99, post-100 marked `[x]` done
- `VERSION` → 1.9.18

---

## [1.9.17] — 2026-05-23

### Changed
- `docs/INCIDENT_RESPONSE.md` §R-13 (v0.6→v0.7). R-13 Law Enforcement / Government Data Request: 9-type request taxonomy (criminal subpoena through FISA order, EU DPA supervisory authority demand, informal voluntary); 5-tier severity matrix (P0 NSL/FISA/Art.9≥10-users/enterprise-scope; P1 criminal order/EU DPA; P2 civil subpoena; P3 informal); 8 unique response constraints including founder-only authority, counsel-before-acknowledgment rule, Art. 9 no-voluntary-production floor, emergency voluntary exception, GDPR Art. 48 gate for non-EU court orders, no-backdoor reaffirmation, restricted `#inc-YYYYMMDD-legal` channel; 11-step legal review process; challenge protocol for overbroad/Art.48-defective requests; scope minimization table with permitted/higher-standard/never-produce categories (CV keypoints architecturally impossible, coaching_turns requires Art. 9 explicit authorization); user notification template with gag-order exception path; enterprise tenant Template T-01; GDPR Art. 48 MLAT analysis table (US, UK, others) with DPA notification protocol; 10 DEC-030 HMAC-chained events (`legal.request_received` HIGH through `legal.disclosure_executed` CRITICAL); annual transparency reporting protocol with NSL "0 to 499" range method; SOC 2 CC6.3/CC6.1/C1.1/CC9.1/P6 mapping; Tabletop Scenario H (DOJ subpoena, EU-resident Growth-tier tenant, 14 discussion points). Appendix A updated with R-13 quick reference. Footer v0.6→v0.7.
- `VERSION` → 1.9.17

---

## [1.9.16] — 2026-05-23

### Added
- `content/post-97-loaded-carries.md` — loaded carries: farmer's walk, suitcase carry, yoke; механізми (grip під динамічним навантаженням, anti-lateral flexion, ізометрична стабілізація під час локомоції), McGill et al. (2013), Androulakis-Korakakis et al. (2018), Bohannon (2019), протоколи і місце у тижні; clinical-safety PASS

### Changed
- `README.md` — post-97 відмічений як [x], roadmap post-100–104 додані
- `VERSION` → 1.9.16

---

## [1.9.15] — 2026-05-23

### Added
- `content/post-94-tendon-vs-muscle-adaptation.md` — сухожилля vs м'язи: structural lag (2–4 тижні vs 8–12 тижнів), Bohm et al. (2015) tendon stiffness і RFD, ексцентрика і HSR для сухожильної адаптації, правило 10%, deload-логіка; clinical-safety PASS
- `content/post-95-peak-strength-vs-strength-endurance.md` — максимальна сила vs силова витривалість: mTOR vs AMPK молекулярний конфлікт, Hickson (1980) і Wilson et al. (2012) interference effect, блокова логіка для аматорів; clinical-safety PASS
- `content/post-96-warmup-vs-activation.md` — розминка vs активація: температурний і нейронний механізми розминки, Fradkin et al. (2010) review, PAP (myosin RLC фосфорилювання, 3–7 хв вікно, тільки для досвідчених), RAMP-протокол; clinical-safety PASS

### Changed
- `README.md` — post-94, 95, 96 відмічені як [x] з clinical-safety PASS

---

## [1.9.14] — 2026-05-23

### Changed
- `docs/INCIDENT_RESPONSE.md` §R-12 + §14 (v0.5→v0.6). R-12 Insider Threat / Privileged Access Abuse: evidence-before-containment constraint; private restricted investigation channel (founder + security-engineer + compliance-officer only); 9-trigger severity matrix; graduated containment options C-1 through C-5 ordered by stealth; legal-before-HR rule with jurisdiction notes; blast-radius SQL (Art. 9 classification, multi-tenant exposure, enterprise tenant identification); GDPR Art. 33/34 table by data category; 11-item evidence package with SHA-256 manifest; 7 post-incident preventive controls; SOC 2 CC6.2/CC6.3/CC7.2/CC7.3/CC7.4/CC9.1/CC1.2 mapping; Tabletop Scenario G added to §9 drill catalog (insider exfiltration before resignation, 14 enterprise tenants, 14 discussion points). §14 Continuous Improvement Program: PIR Action Item Registry (10-field Linear schema, 5-tier SLA table, monthly CSV export); 4 escalation thresholds; quarterly control effectiveness review (60-min agenda, MTTD/MTTR/false-positive metrics, output template); runbook update protocol (6 mandatory triggers, dual-approval gate, tagged archive); annual control self-assessment (Q4, CC4.1/CC4.2/CC7.2/CC7.4/CC7.5, audit firm handoff); SOC 2 CC4.1/CC4.2/CC7.5/CC2.1 mapping. Appendix A updated with R-12 and §14 quick references. Footer v0.5→v0.6.
- `VERSION` → 1.9.14

---

## [1.9.13] — 2026-05-23

### Added
- `content/post-93-mobility-vs-flexibility.md` — «Мобільність vs гнучкість: чому пасивне розтягування не переносить у силу». Sports-science post on the distinction between passive flexibility and active mobility under load: Behm & Chaouachi (2011) meta-analysis showing static stretching reduces force −5.5% and power −2.8% with no transfer to performance; Magnusson et al. (1996) demonstrating that repeated passive stretching changes viscoelastic resistance neurally, not structurally; Weppler & Magnusson (2010) stretch-tolerance model; the mobility deficit concept (passive vs active ROM gap); four limiting factors (neural inhibition, joint capsule, fascia, muscle length); end-range isometrics and loaded end-range work as evidence-based transfer mechanisms; three high-priority joints (ankle dorsiflexion → squat, hip internal rotation → deep squat, thoracic + shoulder → overhead press); pre-training vs post-training vs embedded mobility work protocols. Clinical-safety reviewed: PASS.
- `blog.html` — added card for post-93 (Мобільність vs гнучкість) in sports-science category.
- `STATUS.md` — content engine table row for post-93.

### Changed
- `VERSION` → 1.9.13

---

## [1.9.12] — 2026-05-23

### Added
- `docs/SSO_SCIM_IMPLEMENTATION.md` §17 — Enterprise Pilot Program: 90-Day Runbook (v0.8→v0.9). Operational protocol for the full pilot lifecycle from contract signature through conversion or termination. Pre-Pilot Technical Qualification: 9-item customer checklist (compatible IdP, SCIM 2.0, pilot users, IT admin contact, network access, DPA, contract, billing, data region) + 5-item FORM platform prerequisite check. Pilot tenant provisioning: SQL walkthrough for staging + production tenant records, `pilot_programs` INSERT, IT admin dashboard access. 90-day milestone timeline: 12 milestones (Day 0 provisioning → Day 14 SSO go-live → Day 21 SCIM first-sync → Day 30/60 reviews → Day 90 decision) with pass criteria and escalation paths. Pilot success metrics: 6 named metrics (SSO adoption rate ≥80% at Day 90, SCIM sync error rate <0.5%, P0 incidents = 0, admin satisfaction, NPS, SLA compliance) with binary conversion gate logic. Mid-pilot health monitoring: weekly digest Worker (Monday 06:00 UTC), 5 alert thresholds, OBSERVABILITY.md §18 dashboard integration. Pilot-to-production conversion: 7-step protocol including production tenant activation, "Promote to Production" (§16.11), SCIM Option A re-provision vs Option B user-ID mapping migration with two-person SQL review gate. Termination data handling: 24h DSAR-format export, 30-day window, Art. 17 erasure, DPA termination, sub-processor notification. Database: `pilot_programs` table (state machine FSM, generated `pilot_deadline_at`, terminal state exclusivity constraint, RLS: form_admin/form_system only) + `pilot_milestone_events` (typed enum, `metric_snapshot` JSONB, TypeScript interface). 9 new DEC-030 HMAC-chained audit events: `pilot.tenant_provisioned`, `pilot.sso_setup_started`, `pilot.started`, `pilot.milestone_review`, `pilot.paused`, `pilot.resumed`, `pilot.converted`, `pilot.terminated`, `pilot.data_deleted`. SOC 2 mapping: CC3.2, CC6.2, CC7.2, CC8.1, CC9.2, A1.1. Implementation checklist: 13 tasks (7× P0, 5× P1, 1× P0 QA), M4/M5. G-011: 🟡 Partial (operational runbook layer added; infrastructure provisioning remains blocker).

### Changed
- `docs/SSO_SCIM_IMPLEMENTATION.md` — ToC updated §17; v0.8→v0.9; 4223→4746 lines.
- `VERSION` → 1.9.12

---

## [1.9.11] — 2026-05-23

### Added
- `content/post-91-time-under-tension.md` — «Час під навантаженням (TUT): де правда, де міф». Sports-science post covering the TUT debate: mechanical tension vs. metabolic stress vs. muscle damage as hypertrophy drivers; Burd et al. (2012) JPhysiol evidence that intensity + proximity to failure outweigh time-under-load; Roig et al. (2009) on eccentric training superiority; practical tempo table by goal (strength / hypertrophy / endurance / BFR); three common TUT mistakes. Clinical-safety reviewed: PASS.
- `content/post-92-concurrent-training-interference.md` — «Конкурентне тренування: чому сила і витривалість конфліктують». Sports-science post on the interference effect: Hickson (1980) original finding; AMPK/mTOR molecular conflict; Wilson et al. (2012) meta-analysis (−31% strength, −23% power, −19% hypertrophy); three moderating variables (modality, order, inter-session interval); practical recommendations by priority (hypertrophy-first, endurance-first, GPP); HIIT vs. Zone 2 comparison for concurrent contexts. Clinical-safety reviewed: PASS.

### Changed
- `VERSION` → 1.9.11

---

## [1.9.10] — 2026-05-23

### Added
- `docs/SSO_SCIM_IMPLEMENTATION.md` §16 — Admin Dashboard SSO & SCIM Configuration UI (v0.7→v0.8). Closes G-007 design phase (highest-severity unresolved gap: admin UI required before first enterprise pilot). Full product spec for the FORM admin dashboard identity & access configuration surface. OIDC setup wizard (5-step: IdP selection with pre-populated Okta/Entra ID/Google Workspace endpoints → discovery URL fetch + client credential entry → attribute mapping table → sandboxed test login → go-live toggle with enforce option). SAML 2.0 setup wizard (6-step: IdP selection including ADFS + OneLogin + Ping → SP metadata display with ACS/SLO/Entity ID → IdP metadata XML upload or manual entry → sandboxed test login → attribute mapping with IdP-specific defaults → go-live). SP metadata viewer: connection status banner (last login timestamp + active session count), certificate expiry countdown (color-coded green/yellow/red), PEM and metadata.xml download, dual-cert rotation button wiring to §8.1 procedure, ADFS manual-upload warning. SCIM token management: masked display (last 8 chars), one-click rotation with 15-minute overlap window (`scim_token_hash_previous` + `scim_token_previous_expires_at`), 90-day age alert (SOC 2 CC6.1). Group-to-role mapping UI: add/remove/edit table, Entra ID GUID display with Resolve action, fallback role selector, strict-mode toggle; all writes to `tenant_sso_configs.group_role_mappings` JSONB (§16.7). Azure AD GUID resolver closes G-008 design: opt-in Microsoft Graph API (client credentials per-tenant app registration, admin consent required), batch resolve up to 20 GUIDs/request, `resolved_display_name` stored as annotation, GUID remains authoritative, graceful fallback to manual alias; new columns: `graph_app_id`, `graph_app_secret_key_ref`, `graph_tenant_id`, `graph_consent_granted_at`, `graph_last_group_sync_at` (§16.8). Session policy panel: session timeout (1–720 h default 168), max concurrent sessions, force-SSO toggle with lockout warning + slug confirmation, re-auth gate for sensitive actions (§16.9). IP allowlist UI closes G-006 design: CIDR add/remove with optional label, enable toggle with lockout warning + slug confirmation, max 50 entries per tenant; Cloudflare Worker enforcement spec with `CF-Connecting-IP`, 60-second TTL cache, `sso.ip_blocked` audit event (§16.10). SSO staging environment closes G-011 design: `{slug}-staging.form.coach` endpoint, new `tenants.staging_tenant_id` and `is_staging` columns, 5 synthetic test users auto-created, independent `tenant_sso_configs` row, one-button "Promote to Production" copies protocol/OIDC/SAML/mapping fields (not SCIM token / IP allowlist / session policy), `sso_staging.config_promoted` audit event (§16.11). 16 new audit events on DEC-030 HMAC chain: `sso_config.wizard_started`, `sso_config.test_login_initiated`, `sso_config.test_login_passed`, `sso_config.test_login_failed`, `sso_config.activated`, `sso_config.deactivated`, `sso_config.deleted`, `sso_cert.rotation_initiated`, `sso_cert.rotation_completed`, `scim_token.generated`, `scim_token.revoked`, `sso_config.ip_allowlist_updated`, `sso_config.session_policy_updated`, `sso_config.group_mapping_updated`, `graph_api.group_resolved`, `sso_staging.config_promoted`. SOC 2 mapping: CC6.1 (SCIM token rotation evidence), CC6.2 (group-role mapping UI), CC6.3 (IP allowlist), CC8.1 (all changes HMAC-chained). Implementation checklist: 15 tasks (8× P0, 5× P1, 2× P2), M4/M5. Gap status: G-006 🔴→🟡, G-007 🔴→🟡, G-008 🔴→🟡, G-011 🔴→🟡.

### Changed
- `docs/SSO_SCIM_IMPLEMENTATION.md` — ToC updated §16; header bumped v0.7→v0.8; §9 gap table updated: G-006, G-007, G-008, G-011 marked as design-resolved.
- `VERSION` → 1.9.10

---

## [1.9.9] — 2026-05-23

### Added
- `content/post-90-calisthenics-vs-weighted-training.md` — Калістеніка проти обтяжень: чесне порівняння без фан-клубів. Covers EMG comparison (Calatayud et al. 2015: push-up variations vs bench press ~70% 1RM equivalence), progressive overload mechanisms in bodyweight training (leverage, unilateral variants, tempo, ROM, stability base), structural limitations of calisthenics (maximal strength, lower-body hypertrophy), structural limitations of weighted training (kinesthetic control, accessibility), partial transfer via shared muscles and vectors (Ralston et al. 2018), hybrid programming strategy (barbell compounds + calisthenics auxiliary), Victor 3-question filter: progression / goal match / consistency. clinical_safety_reviewed: PASS.

### Changed
- `blog.html` — card added for post-90
- `STATUS.md` — post-90 row added to content engine table
- `README.md` — post-90 marked done; post-94/95/96 added to roadmap
- `VERSION` → 1.9.9

---

## [1.9.8] — 2026-05-23

### Added
- `docs/DATA_MODEL.md` §18 — Notification & Webhook Event Queue Schema (v0.8→v0.9). 23-value `webhook_event_type` enum covering training lifecycle, streak events (DEC-013 two-miss grace), auth/SSO/SCIM, enterprise lifecycle, wearable sync, system events. `notifications` table: in-app/push/email with soft-delete, RLS (user self-read + tenant admin history + form_system). `webhook_endpoints` table: HTTPS-enforced, AES-256-GCM encrypted signing secret, auto-deactivation at 10 consecutive failures, GIN index on events[]. `webhook_deliveries` table: idempotency via UNIQUE(endpoint_id, idempotency_key), response_body capped 1 KB, exponential backoff (1m→5m→30m→2h→8h→24h, max 6 attempts), HMAC-SHA256 signing with timing-safe verification. GDPR Art.17 erasure: notifications hard-deleted, webhook_deliveries payload-scrubbed in-place within 24h, `erasure.webhook_payload_scrubbed` DEC-030 event. 9 indexes aligned with §11 strategy. 12 DEC-030 audit events. 3 pg_cron maintenance jobs (03:15/03:20/03:25 UTC). Privacy: email excluded from payloads by default; `email_in_webhooks` feature flag requires DPA. 14-item migration checklist, 18-item implementation checklist (M3–M5).

### Changed
- `docs/DATA_MODEL.md` — ToC updated §18; header bumped v0.8→v0.9.
- `VERSION` → 1.9.8

---

## [1.9.7] — 2026-05-23

### Added
- `content/post-87-supercompensation-recovery-windows.md` — Суперкомпенсація: теорія, реальні обмеження та практичні інтервали відновлення. Covers GAS theory, fitness-fatigue two-factor model (Banister 1975–1982), functional vs non-functional overreaching (Meeusen et al. 2013 ECSS consensus), recovery time windows by stimulus type, and practical loading-timing implications for intermediate athletes. clinical_safety_reviewed: PASS.
- `content/post-88-velocity-based-training-vbt.md` — Velocity-Based Training: авторегуляція через швидкість штанги. Covers load-velocity relationship (González-Badillo & Sánchez-Medina 2010), MPV vs peak velocity, velocity zones by adaptation goal (Pareja-Blanco et al. 2017), velocity-loss thresholds for volume autoregulation (Sánchez-Medina & González-Badillo 2011), daily L-V profile for 1RM estimation, VBT as daily readiness indicator, equipment comparison (GymAware/PUSH/apps), and FORM CV angle. clinical_safety_reviewed: PASS.

### Changed
- `VERSION` → 1.9.7

---

## [1.9.6] — 2026-05-23

### Added
- `docs/DATA_MODEL.md` §17 — Enterprise Admin Reporting Schema — Aggregate-Only Data Model & Privacy-Floor Enforcement (v0.7→v0.8). Closes the schema gap between the §2.6 prototype `tenant_wellness_summary` and the full production admin dashboard data model. Permitted/Prohibited Metric Registry (§17.2): 12 permitted aggregate metrics (participation, WAU/MAU, session duration, form score, D30 retention, CV/voice adoption, opt-in NPS, SCIM-group engagement) and 9 strictly prohibited categories (individual workout rows, coaching content, health goals, body composition, meal logs, mental health signals, HRV, user-identity linkage, opted-out users) — 5 categories carry clinical-safety veto. Four production materialized views: `tenant_wellness_summary_v2` (replaces §2.6 prototype — adds `user_aggregate_consent` join, 13-week rolling window, k-anonymity CASE guard on `avg_form_score`, `REFRESH CONCURRENTLY` unique index, RLS policy); `tenant_engagement_summary` (activation_rate_pct, WAU/MAU, D30 retention with cohort-size k-gate via CTEs — no `user_health_profiles` join); `tenant_feature_adoption` (CV adoption % and voice coaching adoption % — session count permitted, content structurally absent); `tenant_cohort_breakdown` (SCIM-group engagement using §15 tables — Art. 9 sensitive-name gate, `meets_anonymity_floor` drives API suppression). Three-layer k-anonymity enforcement: `assert_k_anonymity()` Postgres STABLE function with `p_k_floor DEFAULT 5`; `applyKAnonymityGuard` + `buildSuppressedResponse` TypeScript in `workers/admin-reporting/k-anonymity-guard.ts`; Metabase defence-in-depth filter; `tenants.reporting_k_floor INTEGER CHECK IN (5,10,15)` per-tenant override. pg_cron refresh schedule 02:15/02:30/02:45/03:00 UTC CONCURRENTLY; stale banner at > 26h; P2 alert on cron failure. 5 admin reporting endpoints under `/v1/admin/reporting/`; no `/users` endpoint; `Last-Refreshed` header; EU 403 gate for `dpa_ref IS NULL`. `user_aggregate_consent` table: SCIM-provisioned default `consent_version = NULL` until onboarding; Settings → Privacy opt-out effective at next refresh; `consent_version` drives re-solicitation on registry expansion. GDPR Art. 89 safeguards table. 5 DEC-030 HMAC-chained audit events including `admin.consent_override_attempted` (HIGH, 7yr — auto-P1 incident). Privacy floor test suite: 6 Jest tests (no-user_id assertion, k-anonymity null/pass, opted-out exclusion, no-individual-in-response, cohort-suppression); CI gate blocks deploy on failure. SOC 2 mapping: CC6.1, CC6.3, CC7.2, C1.1, C1.2, P3.2, P4.1, P6.4. 18-item implementation checklist (12× P0, 4× P1, 2× P2). OQ-ENT-02: k-floor sufficiency under GDPR Recital 26 for EU enterprise pilots. OQ-ENT-03: consent re-solicitation mechanism on metric registry expansion.

### Changed
- `docs/DATA_MODEL.md` — ToC updated to include §16 and §17; header bumped v0.7→v0.8.
- `VERSION` → 1.9.6

---

## [1.9.5] — 2026-05-23

### Added
- `content/post-89-acwr-training-load.md` — ACWR і моніторинг навантаження: інструмент, який складніший за формулу. Gabbett (2016) BJSM: acute:chronic workload ratio, U-подібна крива ризику і «безпечна» зона 0.8–1.3; session-RPE (Foster et al. 2001) як метод розрахунку internal load (RPE × хвилини = AU); математична критика rolling-average ACWR (Impellizzeri et al. 2019, Murray et al. 2017): mathematical coupling чисельника і знаменника, поведінка при низькому chronic load; EWMA (Murray et al. 2017) з λ-параметрами для 7-денного і 28-денного вікна як статистично точніша альтернатива; external vs internal load: чому session-RPE перевершує зовнішні метрики для моніторингу накопиченої втоми; Bowen et al. (2017): absolute load як модератор; Victor multi-layer readiness model — 4 рівні: absolute load trend, RPE-drift, суб'єктивна готовність, відносний spike; practical implementation та логіка повернення після паузи. clinical-safety PASS.
- `blog.html` — новий post-89 card: «ACWR і моніторинг навантаження — інструмент, який складніший за формулу» (sports-science, 13 хв читання, Victor · FORM).

### Changed
- `STATUS.md` — додані рядки: post-85 (RFD, PASS), post-86 (periodization models, PASS), post-87 (BLOCKED · clinical-safety VETO · food), post-88 (BLOCKED · clinical-safety VETO · food/supplement), post-89 (ACWR, PASS).
- `README.md` — roadmap оновлено: post-85 і post-86 позначено як [x] з реальними назвами написаних постів; post-87 і post-88 позначено як VETO BLOCKED; post-89 позначено як [x]; нові теми post-91–93 додані до roadmap (loaded carries, accommodating resistance, sticking points).
- `VERSION` → 1.9.5

---

## [1.9.4] — 2026-05-23

### Added
- `docs/OBSERVABILITY.md` §19 — SLO Error Budget Management (v0.5→v0.6). Error budget concept (1 − SLO target = permissible unreliability); per-SLO budget tables: 8 platform SLOs (SLO-01–08), 8 CV pipeline SLOs (CV-SLO-01–08), 12 synthetic probes (S-001–S-012) з monthly budget minutes; Google SRE two-window burn rate alerting: fast burn 14.4×/1h → P0 PagerDuty, slow burn 6.0×/6h → P1, trend 1.0×/3d → Slack; три Postgres queries (slo_burn_rate_fast.sql, slo_burn_rate_slow.sql, slo_burn_rate_trend.sql) + PagerDuty YAML з dedup keys; п'ятирівнева deploy policy (Green/Yellow/Orange/Red/Critical) з UTC 18–22 peak-hour freeze і GitHub Actions gate з override audit trail; `slo_budget_tracking` DDL з GENERATED `budget_consumed_pct`, RLS roles, 90-day pg_cron retention; `slo_budget_weekly` materialized view з Monday 06:00 UTC refresh; Metabase "SLO Error Budget Dashboard" (6 панелей: budget gauges, burn rate trend, top-5 consumers, 30-day rolling, enterprise tenant breakdown, exhaustion audit trail); SOC 2 CC4.1/CC7.2/A1.1/A1.2/CC5.2 mapping; 13-item checklist M3/M4.

### Changed
- `VERSION` → 1.9.4
- `CHANGELOG.md` → [1.9.4] entry added

---

## [1.9.3] — 2026-05-23

### Added
- `content/post-85-rate-of-force-development.md` — Rate of Force Development: чому швидкість приросту сили важливіша за саму силу. RFD як dF/dt у вікні 0–200 мс (Aagaard et al. 2002); нейральна база — motor unit recruitment rate, firing rate (Van Cutsem et al. 1998); чому абсолютна сила не конвертується в потужність без velocity training; методи розвитку RFD: intent-based training (максимальна намір швидкості при субмаксимальних вагах), ballistic/plyometric, accommodating resistance (ланцюги/гуми), contrast training (PAP — Tillin & Bishop 2009); практичне програмування: 2–4 підходи × 3–6 повторів × 30–70% 1RM з максимальним intent, позиція у тижні перед важкою силовою роботою. clinical-safety PASS.
- `content/post-86-periodization-models.md` — Моделі програмування: лінійна vs DUP vs блокова. GAS (Selye 1950) + SAID принцип як фізіологічна база periodization; LP (Zatsiorsky) — максимальна простота, ефективна для new-intermediate, вичерпується через 12–16 тижнів через монотонний стимул; DUP (Rhea et al. 2002, JSCR) — варіація у межах тижня (сила/гіпертрофія/потужність-фокус по днях), усуває monotony без зміни обсягу; Block (Issurin 2008) — концентрація адаптацій і заплановані піки, Wilson et al. 2012 про competing adaptations; Harries et al. 2015 мета-аналіз — DUP перевершує LP для intermediate у більшості метрик; Victor-фільтр: три питання (стаж, ціль, частота) визначають модель. clinical-safety PASS.

### Changed
- `VERSION` → 1.9.3
- `CHANGELOG.md` → [1.9.3] entry added

---

## [1.9.2] — 2026-05-23

### Added
- `docs/COST_MODEL.md` §18 — Revenue Recognition, ARR/MRR Bridge & SaaS Metrics Taxonomy. ASC 606 / IFRS 15 ratable recognition rule for annual and multi-year contracts; billings vs. revenue vs. cash three-waterfall model; monthly MRR bridge template (New/Expansion/Contraction/Churned ARR); free 90-day pilot timing and first-recognition date (zero ARR during pilot, recognition on Day 91); deferred revenue schedule for 2-year and 3-year deals (short-term vs. long-term balance sheet split); NRR/GRR model with target table (≥110% growth-stage, ≥120% scale-stage) and churn-rate sensitivity matrix; investor-grade SaaS metrics taxonomy (ARR, MRR, ACV, TCV, Bookings, Billings, Revenue, NRR, GRR, CAC, LTV, Magic Number, Rule of 40) with FORM-specific notes and reporting cadence. OQ-14: revenue recognition for mid-contract seat additions. OQ-15: ASC 606 variable-consideration treatment for conditional pilot discounts (15-month vs. 12-month recognition period depending on legal structure of pilot agreement). Doc bumped v0.8 → v0.9.

### Changed
- `VERSION` → 1.9.2
- `CHANGELOG.md` → [1.9.2] entry added

---

## [1.9.1] — 2026-05-23

### Added
- `content/post-84-motor-learning.md` — Мотор лернінг: як насправді засвоюється техніка підйомів. Fitts & Posner (1967) три стадії: когнітивна (prefrontal контроль, висока варіабельність), асоціативна (striatum/cerebellum, звуження помилок), автономна (моторна кора, мінімальний prefrontal control — «паралич від аналізу» при надмірних інструкціях). Shea & Morgan (1979) contextual interference effect: рандомна практика гірша короткостроково, але дає кращу довгострокову ретенцію — мозок будує моторну програму заново кожен раз, а не «кешує». Schmidt (1975) schema theory: variability of practice будує ширші моторні схеми і кращий перенос на нові умови (протилежно ідеї «однієї правильної форми»). Wulf & Prinz (2001) і мета-аналіз Wulf (2007): зовнішній фокус уваги («штовхни підлогу») стабільно перевершує внутрішній («напружи квадрицепс») — знижує prefrontal навантаження і дозволяє автоматизованим механізмам керувати рухом. Salmoni et al. (1984) guidance effect: надмірний feedback формує залежність і гальмує розвиток внутрішньої системи детекції помилок — оптимум кожні 3–5 повторів. Walker et al. (2002) Neuron: сон консолідує моторні сліди +20% без додаткової практики (SWS реактивує моторні мережі). Victor-практика: зовнішні кіу за замовчуванням, прогресивне зменшення частоти feedback, більше зворотного зв'язку на когнітивній стадії. clinical-safety PASS.

### Changed
- `blog.html` → картка для post-84 (Motor learning) додана після post-83
- `STATUS.md` → рядок post-84-motor-learning.md у content engine table
- `README.md` → post-84 позначений [x]; roadmap розширений до post-90 (ACWR, Calisthenics vs weighted); ⚠️ нотатки про перетин для post-85/86

---

## [1.9.0] — 2026-05-22

### Added
- `content/post-82-deload-protocols.md` — Розвантаження: коли, як і чому Victor дає тобі тиждень назад. Banister двофакторна модель fitness-fatigue (1975): fitness-ефект (повільний, тривалий) vs fatigue (швидкий, минущий); продуктивність = Fitness − Fatigue. Три рівні стомлення: периферичне (м'язовий рівень, Laurent et al. 1992 — маркери пошкодження до 7 діб після ексцентричного тренінгу), центральне (ЦНС-збудливість, Gandevia 2001 — зберігається довше периферичного), сполучнотканинне (кlag синтезу колагену у сухожиллях, Kjaer 2004). Розмежування: нормальне стомлення vs функціональне перевантаження (FOR, оборотне, навмисне — Meeusen et al. 2013) vs нефункціональне (NFOR, тижні відновлення) vs OTS. Чотири типи деload: volume deload (−40–60% об'єму, зберігає моторний патерн — Zourdos et al. 2021), intensity deload (−10–20% 1RM, для ЦНС-стомлення після 85–95%+), комбінований, skill deload. Реактивний vs проактивний підхід: Victor = reactive-first, proactive як страховка. Сигнали для деload (RPE drift 1.5+ балів 2+ тижні, техніка, відновлення). Тривалість: 5–7 днів (декондиціонування у тренованих не раніше 2–3 тижнів — Bosquet et al. 2013). Victor-практика: без жорсткого календаря; після деload — суперкомпенсація виводить на пік вище попереднього. clinical-safety PASS.
- `content/post-83-muscle-fiber-types.md` — Типи м'язових волокон: чому «повільні» і «швидкі» — це не вирок. Тричастинна класифікація: Type I (окислювальні, низький поріг, висока витривалість до стомлення), Type IIa (гібридні, «робочий кінь» силового тренінгу), Type IIx (гліколітичні, максимальна потужність; у людини — IIx, не IIb — Schiaffino & Reggiani 2011 PhysRev). Henneman Size Principle (1965): рекрутування I → IIa → IIx у суворому порядку; IIx рекрутуються тільки при ≥80–85% 1RM або при глибокому стомленні нижчих одиниць (Morton et al. 2016 JAP — EMG-підтвердження). Розподіл (Gollnick et al. 1972): середній ~50% Type I, але варіабельність 20–80%. Конвертація: IIx→IIa реальна і значуща після 8 тижнів силового тренінгу (Staron et al. 1991); I→IIa практично не відбувається. Гіпертрофійний відгук: Type II мають вищий потенціал, але Type I гіпертрофуються при достатньому об'ємі і наближенні до відмови (Mitchell et al. 2012 JAP — сумірна гіпертрофія при 12 тижнях). Міф «правильних rep ranges»: діапазон повторень не критичний — критичні рекрутування, механічне навантаження і наближення до відмови (Schoenfeld et al. 2017 JSCR мета-аналіз). Victor-практика: профілювання через RPE-drift між сетами (швидкий drift = вищий Type II), корекція пауз і об'єму, spectrum-тренінг без прив'язки до уявного «ідеального типу». clinical-safety PASS.

---

## [1.8.5] — 2026-05-22

### Changed
- `docs/COST_MODEL.md` → v0.8: §8 (Enterprise Economics) and §14.6 (Enterprise LTV model) recalculated to match the current three-tier rate card from `docs/ENTERPRISE.md` and `enterprise.html`. Previous §8 used stale placeholder pricing ($25–35/seat, 20/100/500-seat deal sizes); now aligned to Starter $12/seat (50–200 seats), Growth $9/seat (200–1,000 seats), Enterprise $6–8/seat (1,000+). §8.1 ACV table, §8.3 minimum deal floor (50 seats), §8.4 margin comparison (Growth tier representative), §8.5 CAC table (Starter/Growth/Enterprise tiers), §8.6 LTV/CAC (four representative deals), §8.7 NRR model (Growth cohort baseline), §8.8 Y1/Y2+ table (500-seat Growth deal) all updated. §14.6 LTV model uses 50-seat Starter minimum and Growth/Enterprise deal sizes; 3-yr GM-adj LTV and LTV:CAC ratios recalculated. §16 was already correct and is unchanged.

---

## [1.8.4] — 2026-05-22

### Added
- `content/post-81-neural-vs-structural-adaptation.md` — Нейральна vs структурна адаптація: чому перші місяці тренінгу — це не гіпертрофія. Moritani & deVries (1979) класичний EMG-експеримент (EMG без змін окружності перші 3–4 тижні), три нейральні механізми (рекрутування рухових одиниць — Henneman Size Principle, rate coding — Patten et al. 2001, міжм'язова координація), Sale (1988) MSSEx (максимальне рекрутування зростає до структурних змін), Folland & Williams (2007) SportsMed (нейральні фактори пояснюють 20–100% початкового приросту сили), Aagaard et al. (2001) JAP (архітектурні зміни — кут пеннації +30%), часова шкала нейральної/структурної фази (тижні 1–4/4–8/8–16/16+), м'язова пам'ять: нейральна і epigenetic — Seaborne et al. (2018) Sci Rep, практичні наслідки для новачків (техніка = нейральна інвестиція), повернення після паузи, першого місяця нової вправи, зв'язок з авторегуляцією. clinical-safety PASS.

### Changed
- `blog.html` → додані картки для post-80 (RPE/RIR авторегуляція) і post-81 (нейральна vs структурна адаптація)
- `STATUS.md` → content engine table: додані рядки post-80 і post-81
- `README.md` → roadmap: post-80 і post-81 позначені [x]; food/supplement теми (натщесерце, Omega-3) переміщені до post-87–88 з флагом clinical-safety required; roadmap розширений до post-88 (motor learning і техніка, bilateral deficit, stretch-mediated hypertrophy)

---

## [1.8.3] — 2026-05-22

### Added
- `content/post-80-rpe-rir-autoregulation.md` — Авторегуляція: RPE та RIR замість відсотків від 1RM. 10-бальна шкала Tuchscherer, RIR як зворотній бік RPE, три фундаментальні проблеми відсоткового програмування (варіація 1RM ±5–8% Meeusen et al. 2013, контекстна невідповідність навантаження, застарілий 1RM), RPE у блоках accumulation/intensification/peaking/deload, Colquhoun et al. 2017 JSCR (порівнянний приріст 1RM, менше провальних сесій у RPE-групі), Helms et al. 2016 IJSPP (точність RIR-оцінки ±1 у досвідчених), обмеження (новачки, пік-блок, командне тренування), гібридний підхід, Victor integration. clinical-safety PASS.

### Fixed
- `docs/SSO_SCIM_IMPLEMENTATION.md` — заголовок виправлено з v0.6 → v0.7 (§15 SCIM Groups вже був у тілі doc). ToC доповнено пунктом 15 (SCIM 2.0 Groups Provisioning).

---

## [1.8.2] — 2026-05-22

### Changed
- `docs/INCIDENT_RESPONSE.md` → v0.5 — R-11 CV / Pose Estimation Biometric Data Privacy Incident + §13 Board & Investor Communication Protocol. R-11: GDPR Art. 9 biometric-specific runbook for CV pipeline incidents (distinct from R-01); two-track architecture (Track A on-device, Track B `cv_sessions.keypoints_enc`); scope assessment SQL; Art. 9 heightened notification rules; clinical-safety gate; 12-item evidence package; post-incident five-way re-enable gate. §13: Board & Investor Communication Protocol; six-trigger threshold table; founder-exclusive authority; Templates B-01 through B-04 with explicit constraints; 7-year archive in `compliance/evidence/incident-comms/<slug>/board/`; SOC 2 CC2.2/CC9.1/CC7.3/A1.1 mapping. Appendix A updated.

---

## [1.8.1] — 2026-05-22

### Added
- `content/post-79-said-principle-specificity.md` — Принцип SAID: три виміри специфічності адаптації (нейром'язова — кутова специфічність, швидкість скорочення, Sale & MacDougall 1981, Blazevich & Jenkins 2002; метаболічна — ATP-PCr/glycolytic/oxidative специфічність; морфологічна — ROM і архітектура м'яза, Schoenfeld & Grgic 2020, McMahon 2014), перенос тренування (де є, де мінімальний, cross-education ~8–15%), чому початківці — виняток із правила (Häkkinen 2001), exercise selection logic, техніка як нейром'язова інвестиція. clinical-safety PASS.

### Changed
- `blog.html` → додані картки для post-77 (BFR), post-78 (Concurrent Training), post-79 (SAID principle)
- `STATUS.md` → content engine table: додані рядки post-77, post-78, post-79
- `README.md` → post-79 позначений [x] (нова тема замість food-topic з VETO); roadmap розширений до post-86 (neural adaptation, training load monitoring, calisthenics vs weights); food/supplement теми переміщені до post-80–83 з флагом clinical-safety required

---

## [1.8.0] — 2026-05-22

### Added
- `compliance/cc6/offboarding-procedure.md` (CC6-E-001, v0.1) — Personnel Offboarding Procedure. CC6.3 control: 24h credential revocation SLA across 13 systems (GitHub, Supabase, Cloudflare, 1Password, PostHog, Sentry, Google Workspace, Slack, Linear, ElevenLabs, Anthropic, AWS, Stripe). Device/data handling, NDA reminder templates, secrets rotation decision matrix (HMAC key always rotated per DEC-030), DEC-030 audit events, solo-founder compensating controls, founder succession/emergency access protocol (§8), exception log, SOC 2 evidence package spec. PRE-05 🔴 Open → 🟡 Authored; CC6-GAP-001 🔴 → 🟡 Authored. Closes to 🟢 upon first execution with evidence filing.
- `content/post-77-blood-flow-restriction.md` — BFR Training: venous occlusion mechanism, AMPK-driven Type II recruitment, AOP pressure protocol, Abe/Loenneke classic sets (30–15–15–15, 30–60s rest), evidence review (Slysz 2016, Lixandrão 2018), scenarios where BFR > standard loading (rehab, joint-limited, age-related), contraindications (DVT, coagulopathy, uncontrolled hypertension). clinical-safety PASS.
- `content/post-78-concurrent-training.md` — Concurrent Training / interference effect: Hickson 1980 origin, AMPK vs mTOR molecular antagonism (TSC2 / RAPTOR phosphorylation), Wilson et al. 2012 meta-analysis (hypertrophy −, strength −, VO₂max =), mode effect (running > cycling), timing rules (strength → aerobic; ≥ 6h gap), volume thresholds (≤ 3 × 30–40 min = minimal interference), sample weekly template, aerobic training benefits for lifters (PCr resynthesis, mitochondrial density). clinical-safety PASS.

### Changed
- `docs/SOC2_READINESS.md` → PRE-05 🔴 Open → 🟡 Authored; fast-track item 🔴 → 🟡 in priority registry.
- `README.md` → post-77/78 marked [x]; former planned topics renumbered post-79–82.

---

## [1.7.2] — 2026-05-22

### Changed
- `docs/OBSERVABILITY.md` → v0.5 — §18 CV Pose Estimation Model Observability. Privacy-first observability design for GDPR Art. 9 biometric data (raw frames never leave device; `keypoints_enc` inaccessible to observability stack). RED signal taxonomy for three CV layers: on-device SDK telemetry, `cv_sessions` server-side aggregates, fleet views. `cv_sdk_telemetry` table DDL: cold_start_ms, inference_latency P50/P95, frame_count, frames_dropped, frames_skipped, kalman_resets, memory_warning_count, device_tier — no keypoints or form scores. `cv_session_fleet_stats` materialized view: daily × model_name × model_version × tenant_id; pct_completed, pct_tracking_lost, confidence_p50/P25, avg_below_threshold_ratio, three defect prevalence columns (knee_cave, back_arch, depth_short); pg_cron 04:00 UTC. Eight SLOs: completion ≥ 85%, tracking_lost ≤ 8%, iOS inference P95 < 30 ms, Android P95 < 50 ms, cold start P95 < 800 ms, confidence P50 ≥ 0.80, high-confidence rate ≥ 70%, below-threshold ratio ≤ 0.12; ≥ 50-session window guard. Eight alerting rules AL-CV-01 through AL-CV-08: fleet spike → PagerDuty P1, confidence drop → PagerDuty P1, new model version regression → P1 with rollback trigger, latency/cold-start/frame-drop/Kalman → Slack P2, per-tenant anomaly → Slack #csm-alerts P2. Alert dedup (`dedup_key`) + rollout suppression (2h post 5% fleet). Blast-radius query aligned with INCIDENT_RESPONSE.md R-10 and DATA_MODEL.md §15.2 `idx_cv_sessions_model_version`. Six-constraint privacy matrix (no GROUP BY user_id, k-anonymity n ≥ 5, keypoints_enc inaccessible, no per-user form scores, CI lint gate). Four-model variant table (blazepose_lite/full, movenet_lightning/thunder × Apple Vision / ML Kit). Metabase "CV Model Health" (9 panels) and "CV Rollout Progress" dashboard specs. SOC 2 CC7.2/CC7.3/PI1.2/PI1.4/C1.2 evidence mapping; monthly CSV to `compliance/evidence/cv-fleet-stats-YYYY-MM.csv`. Thirteen-item implementation checklist (4× P0, 5× P1, 4× P2) across M4/M5.

---

## [1.7.1] — 2026-05-22

### Changed
- `docs/DATA_MODEL.md` → v0.7 — §16 Enterprise Contract & Pilot Lifecycle Schema. `tenants.lifecycle_status` 5-state machine (pilot/active/expired/suspended/churned) with Worker access gate. `tenant_pilots` table: 90-day pilot tracking with fixed-schema JSONB `success_criteria`, CSM owner field, `internal_notes` PII prohibition + weekly content-scan. `tenant_contracts` table: `arr_cents` FORM-internal (excluded from `form_api` RLS and `tenant_contract_portal` view); partial unique index enforcing one active contract per tenant. `assertSeatAvailable()` TypeScript in Cloudflare Worker provisioning layer: `SELECT … FOR UPDATE SKIP LOCKED` TOCTOU protection; 402 on seat-limit breach. `tenant_seat_utilization` materialized view: aggregate seat counts, 7d/30d utilization, k-anonymity NULL suppression below n=15, nightly 02:00 UTC pg_cron refresh; `arr_cents` absent. Renewal notice Worker (09:05 UTC): Slack to `#csm-renewals` at `renewal_notice_days`, PagerDuty P2 at ≤ 14 days. 14 DEC-030 HMAC-chained audit events (including HIGH-severity `tenant.contract_terminated` and `tenant.access_suspended`; `attempted_email: null` invariant CI-enforced). `dpa_ref TEXT` column required before EU pilot activation (closes P-GAP-002 DPA tracking). SOC 2: CC6.1 (seat enforcement), CC6.2 (contract-as-authorisation), CC9.1 (renewal notice), CC9.2 (contract record), A1.1 (SLA tier from plan), PI1.4 (processing integrity), C1.2 (`arr_cents` confidential). 14-item implementation checklist (7× P0, 4× P1, 2× P2, 1× Pre-EU-pilot). OQ-ENT-01: minimum seat floor validation at M6.

---

## [1.7.0] — 2026-05-22

### Added
- `content/post-76-lactate-myth.md` — «Лактат: не отрута, а паливо — чому "молочна кислота" зробила силовий тренінг дурнішим» (~2700 слів, 13 хв читання). Охоплює: Brooks (1984/2009) клітинний лактатний шатл, H⁺ vs лактат як причина ацидозу (Robergs 2004, Westerblad 2002), MCT-транспортери, LT1/LT2/MLSS, чому DOMS не пов'язаний з лактатом, практичні наслідки для відпочинку між підходами, аеробної бази силового атлета, β-аланіну. Editorial-brutalist тон. · **clinical-safety: PASS**
- `blog.html` — додано картки Post 75 (cluster sets) і Post 76 (lactate myth)
- `STATUS.md` — додано рядки для post-75 і post-76 у content engine таблицю

---

## [1.6.1] — 2026-05-22

### Added
- `content/post-75-cluster-sets.md` — «Кластерні підходи: наука інтра-сетного відпочинку у силовому тренінгу та гіпертрофії» (~2800 слів, 15 хв читання). Охоплює: PCr-кінетику деградації традиційного підходу (Gaitanos 1993, Fitts 2008), velocity loss як проксі-маркер (Pareja-Blanco / Sánchez-Medina 2017), збереження HTMU через принцип Хеннемана, три варіанти (традиційний кластер / rest-pause / undulating cluster), ключові дослідження (Oliver 2013, Haff 2003/2008, Tufano 2016/2017), кінетична таблиця відновлення PCr за часом паузи, контраіндикації (гіпертрофія, новачки, <70% 1ПМ, ізоляційні вправи). Editorial-brutalist тон. · **clinical-safety: PASS**

---

## [1.6.0] — 2026-05-22

### Added
- `compliance/cc1/security-training-log.md` — CC1-E-001, v0.1 · Annual Security Awareness Training Completion Log (CC1-GAP-002 🔴→🟡 AUTHORED). Eight-topic curriculum: OWASP Top 10, phishing and social engineering, GDPR Art. 9 data classification, incident response basics, credential hygiene / secrets management, OWASP ASVS secure code review, privacy and GDPR obligations, vendor risk awareness. Evidence filing instructions with git commit SHA workflow. Founder self-attestation template. Solo-phase auditor disclosure (CC1-E-002 and CC1-E-005 intentionally absent). Gap closes 🟢 upon founder self-attestation executed and committed.

### Changed
- `compliance/cc1/README.md` — CC1-E-001 row 🔴 Pending → 🟡 AUTHORED; CC1-GAP-002 row 🔴 Open → 🟡 AUTHORED with artifact reference.
- `docs/SOC2_READINESS.md` → v2.8 — CC1-GAP-002 updated in §22 gap table, §38.4 master gap registry, and §38.6 evidence artifact index. Version note appended. Open gap count: 49 (no change — gap advances from 🔴 to 🟡, P0 count holds until self-attestation executed).

---

## [1.5.0] — 2026-05-22

### Added
- `compliance/c1/data-asset-inventory.md` — PRE-34-E-001, v0.1 · Confidential Data Asset Inventory (C1-GAP-002 → 🟡 Authored, pending founder signature for 🟢). Covers 14 Supabase table entries (DC-01 through DC-07 + audit_log + tenants + tenant_users + programs + subscriptions + analytics queue) with classification, retention period (P1-RET-001), column-level encryption status, and backup tier coverage. Cloudflare Workers / R2 / KV. Backblaze B2 WORM Tier 3 (7-year immutable). 8 sub-processor data holdings cross-referenced to `compliance/p1/sub-processor-register.md`. GitHub, 1Password Teams, Cloudflare Workers Secrets. PostHog EU / ClickHouse / Metabase analytics layer. Four-tier classification (Public/Internal/Confidential/Sensitive/Restricted) + Art. 9 health-data overlay applied throughout. GDPR Art. 17 erasure sequence (§8.1). Interim device disposal rule (§8.3, pending C1-GAP-004). RLS access control matrix cross-referenced from DATA_MODEL.md §8. Founder signatory block with C1-INV-001 signing procedure in compliance/c1/README.md.
- `compliance/c1/README.md` — C1 evidence directory index; PRE-34-E-001 through PRE-34-E-008 artifact tracking; gap status for C1-GAP-001 through C1-GAP-004; signatory procedure for PRE-34-E-001.

### Changed
- `docs/SOC2_READINESS.md` v2.6→v2.7 — CC7-GAP-004 🔴→🟢 CLOSED (R-05 runbook confirmed in INCIDENT_RESPONSE.md §R-05 v0.4; gap was stale); C1-GAP-002 🔴→🟡 AUTHORED (PRE-34-E-001 v0.1 filed); PRE-34-E-001 evidence status updated; §38.4 master gap registry updated; open gap count 50→49; P0 count 14→13.

### SOC 2 Gap Progress
- CC7-GAP-004 🔴→🟢 CLOSED (R-05 runbook existed in INCIDENT_RESPONSE.md v0.4; gap was authored before runbook was merged)
- C1-GAP-002 🔴→🟡 AUTHORED (`compliance/c1/data-asset-inventory.md` v0.1; pending founder signature)
- SOC 2 open gaps: 50→49 · P0: 14→13 · Readiness: ~95% (unchanged — no new control implementations; documentation artifacts advanced)

---

## [1.4.1] — 2026-05-22

### Added
- `content/post-74-bilateral-deficit.md` — Білатеральний дефіцит: нейром'язовий феномен де білатеральна сила < суми унілатеральних. Комісуральне гальмування через corpus callosum (Howard & Enoka 1991), клітини Реншоу, дефіцит рекрутування Type IIx рухових одиниць. Білатеральна фасилітація у нетренованих і балістичних рухах. Модуляція дефіциту тренувальним досвідом. Cross-education (Carroll et al. 2006). Програмні наслідки: тестування асиметрії, вибір вправ, перенос у спортивний контекст. 13 хв читання · clinical-safety PASS.
- `blog.html` — картка Post 74 (Білатеральний дефіцит) у секції блогу.
- `STATUS.md` — рядок post-74-bilateral-deficit.md у content engine table.

---

## [1.4.0] — 2026-05-21

### Added
- `docs/DATA_MODEL.md §15` — Computer Vision (CV) Pose Estimation Data Schema (v0.5 → v0.6); closes long-standing schema gap for `cv_sessions` table referenced in SOC2_READINESS.md PRE-34-E-006, §35.6.3, C1.2, and INCIDENT_RESPONSE.md §R-10. On-device inference architecture rationale (GDPR Art. 9 avoidance for raw video, < 100 ms latency, competitive privacy posture). `cv_sessions` DDL: BlazePose 33-keypoint / MoveNet 17-keypoint schema, 7-slug `cv_flags` defect taxonomy, `keypoints_enc` AES-256-GCM column-level encryption in Cloudflare Workers Secret (closes PRE-34-E-006), status state machine (active/completed/partial/tracking_lost), model identity columns for R-10 blast-radius scoping, 6 supporting indexes. RLS: user self-access; form_coach zero-row at Postgres layer (application-layer view only); form_tenant_admin zero-row privacy floor; form_system full access; form_admin break-glass with mandatory incident_id + separate decryption endpoint. Three CI RLS assertions added to `rls_isolation.test.ts`. `tenant_cv_summary` materialized view: weekly aggregate with k-anonymity N ≥ 5 suppression, form_quality averages, status percentages, three defect-prevalence columns (no individual data). Keypoint storage decision resolves §9 gap 9.2: JSONB TOAST with R2 migration gate at 20 GB / 1,000 sessions/day; `workouts.raw_cv_data` deprecated. `stripCVData()` TypeScript: 4-bucket form_quality mapping, flag-presence-only extraction, keypoints categorically excluded from Anthropic API; CI grep gate. GDPR: Art. 9 biometric classification; `'cv'` consent gate; two-phase deletion (1-year keypoints nullification, 3-year row hard delete); Art. 17 immediate keypoints nullification without grace period. Analytics restrictions: keypoints_enc + form_score/cv_flags excluded from all PostHog/ClickHouse/Metabase/Sentry/Anthropic layers. Seven DEC-030 HMAC-chained audit events (cv.session_started, cv.session_completed, cv.session_blocked, cv.keypoints_nullified, cv.session_hard_deleted, cv.keypoints_admin_accessed HIGH, cv.model_version_changed). 17-item implementation checklist (9× P0, 5× P1, 2× P2, 1× Pre-launch) across M3/M4/M5. OQ-CV-01: R2 migration gate.

### Gap Closures
- §9 gap 9.2 (`raw_cv_data` storage): 🔴→✅ RESOLVED — `cv_sessions.keypoints_enc` JSONB TOAST with R2 gate condition
- SOC2 PRE-34-E-006 (`cv_sessions.keypoints` column-level encryption): 🟡→🟢 DDL authored (implementation commit pending)

---

## [1.3.2] — 2026-05-21

### Added
- `content/post-73-stretch-mediated-hypertrophy.md` — 13-min sports-science post: titin, sarcomerogenesis, stretch-mediated hypertrophy; Wolf et al. 2023, Kassiano et al. 2023, Maeo et al. 2024; clinical-safety PASS; blog.html card added

---

## [1.3.1] — 2026-05-21

### Added
- `docs/ACCEPTABLE_USE_POLICY.md` — v1.0 AUTHORED (status: pending external counsel review + founder signature before taking legal effect); promotes `docs/SOC2_READINESS.md` §27 Annex A v0.1-draft to standalone enterprise-grade policy; closes CC1-GAP-001 + CC5-GAP-001 (both P0 SOC 2 gaps); filed as CC1-E-004; 13 sections: Purpose, Scope (all FORM systems + MDM devices), Definitions (6 terms incl. DEC-030 Audit Chain + Break-Glass Access), Acceptable Uses (10 items incl. consent-gated health data access + break-glass with incident_id), Prohibited Uses (12 items incl. health data curiosity access, git secrets, security control bypass, SHA-1/MD5, audit log tampering, force-push to main), Health Data Special Category Rules (GDPR Art. 9 consent gate enforcement, analytics exclusion, Sentry scrubber, coaching context bucketed only, DEC-030 on every access), Device Security Requirements (FileVault 2 / BitLocker, MDM, ≤5min screen lock, no public Wi-Fi without WARP), Incident Reporting Obligations (same-day mandatory, #security-alerts path), Enforcement (access revocation, GDPR Art. 83 trigger on health data violations = P0 incident), Acknowledgment path (compliance/cc1/aup/ + policy-approval-log.csv), SOC 2 mapping (CC1.1, CC5.3, CC6.3 + CC1-E-004), Signatory block (founder + compliance-officer; TBD on counsel approval)

### SOC 2 Gap Progress
- CC1-GAP-001 🔴→🟡 (AUP v1.0 authored; pending counsel review + founder signature for P0-11 gate PASS)
- CC5-GAP-001 🔴→🟡 (AUP authored — same document; counsel + signature pending)
- SOC 2 P0 gate: 0/14 → 0/14 (criteria met when counsel review + signing complete; authoring prerequisite fulfilled)

---

## [1.3.0] — 2026-05-21

### Added
- `docs/CRYPTOGRAPHY_POLICY.md` — v1.0 IN FORCE; closes CC5-GAP-002 (P0 SOC 2 gap); approved algorithms (TLS 1.3, AES-256-GCM, HMAC-SHA256, Argon2id), prohibited algorithms (MD5, SHA-1, DES/3DES, RC4, ECB, static IVs), minimum key lengths (RSA 2048, ECC P-256, AES-256, HMAC 256-bit, JWT 512-bit), key rotation schedule (JWT 90d, API keys 180d, DB encryption annual, HMAC chain annual with chain-continuity procedure), key custody matrix (1Password, Cloudflare Workers Secrets, Supabase Vault, offline escrow for B2 WORM tier), certificate management (Cloudflare-managed auto-renew, mobile SHA-256 pinning with 90d advance rotation notice), key compromise runbook (detect → isolate → rotate → audit → notify → post-mortem), algorithm deprecation procedure; SOC 2 evidence mapping CC5.2/CC5.3/C1.1/C1.2; 14-item implementation checklist M3/M4
- `compliance/policy-approval-log.csv` — CC5-P1-004; policy registry with 9 policies (POL-001 through POL-009) covering AUP, Cryptography Policy, Security Policy, Privacy Policy, DPIA, Incident Response, Sub-processor Register, Retention Decisions, Complaint Intake Procedure; columns: policy_id, policy_name, version, file_path, status, effective_date, approved_by, approval_method, next_review_date, gap_ids_closed, soc2_criteria, gdpr_basis, notes
- `compliance/cc1/README.md` — CC1 evidence directory index; CC1-E-001 through CC1-E-005 artifact tracking; AUP acknowledgment procedure (signing path → git tag → PDF to cc1/aup/ → policy-approval-log.csv update → SOC 2 P0-11 gate closure); CC1 gap status tracker (CC1-GAP-001 🟡 AUTHORED pending counsel; CC1-GAP-002/003/004 🔴 Open)

### SOC 2 Gap Progress
- CC5-GAP-002 🔴→🟢 CLOSED (Cryptography Policy authored and in force)
- CC1-GAP-001 🔴→🟡 PARTIAL (AUP document authored; pending counsel review + founder signature for full P0-11 gate closure)
- CC5-P1-004 🔴→🟡 PARTIAL (policy-approval-log.csv created; complete population after AUP and Cryptography Policy formally approved)

---

## [1.2.0] — 2026-05-21

### Added
- `compliance/p1/complaint-intake-procedure.md` — enterprise: GDPR privacy complaint intake procedure (P1-CIP-001); closes P-GAP-007 (P8.1) — no formal complaint intake process; intake channel `privacy@form.coach`; 30-day response SLA with 90-day extension path per GDPR Art. 12(3); 8 triage categories (CAT-01 DSAR fallback through CAT-08 Art. 22 automated decision-making); per-category response procedures including Art. 9 flag requirement and compliance-officer sign-off; 6 new DEC-030 HMAC-chained audit events (`privacy.complaint_received`, `privacy.complaint_resolved`, `privacy.complaint_escalated`, `privacy.processing_restricted`, `privacy.objection_received`, `privacy.automated_decision_review_requested`) — all 7-year retention; complaint log schema (user_id hashed SHA-256, no PII in CSV); supervisory authority escalation paths for UK (ICO), EU (DPC/lead DPA), Ukraine (Ombudsman); 11 open items (P0: mailbox creation, mailbox address in privacy policy; P1: `processing_restricted` column migration, 6 DEC-030 events in AUDIT_LOG_SCHEMA.md, Settings deep-link, 23-day PagerDuty alert, PRE-35-E-011 evidence registration); annual Q1 review schedule with transparency report commitment; gap closure: P-GAP-007 🔴→🟡 (procedure defined — mailbox + DEC-030 wiring + privacy policy pending); P-GAP-001 dependency documented
- `compliance/p1/complaint-log.csv` — enterprise: empty complaint log (schema header only); no PII; `user_id_hashed` field uses SHA-256; feeds annual transparency report

---

## [1.1.2] — 2026-05-21

### Added
- `docs/COST_MODEL.md §17` — Downside Scenario Analysis & Unit Economics Stress Tests (v0.7): six adverse scenarios stress-tested against v0.6 baseline; S1 Anthropic API doubles (break-even +1.3%, survivable), S2 App Store 30% commission (break-even +22.3%, success-triggered — web billing path mitigates), S3 churn doubles 5%→10% (LTV −50%, product intervention required — D7→D30 retention primary signal), S4 20% price cut (break-even +26.0%, hold price), S5 enterprise Year 1 miss (not existential), S6 S1+S3 combined (freeze paid acquisition protocol); survivability verdicts, mitigation priority lists, per-scenario monitoring thresholds; OQ-12 (Anthropic volume pricing threshold ~25k Pro subs), OQ-13 (Victor token budget — instrument in beta)
- `content/post-72-total-stress-training-capacity.md` — sports-science: як загальний стрес (сон-депривація, когнітивний стрес, ситуативні стресори, імунне навантаження) знижує реальну MRV; механізми IGF-1, кортизолемії, прозапальних цитокінів; практичний стрес-аудит і модель корекції об'єму; clinical-safety PASS (physiological framing, no food/body-image/mental-health content)
- `blog.html` — картка post-72 (Загальний стрес і тренувальна ємність)
- `STATUS.md` — content engine table: post-72 row added
- `README.md` — roadmap: post-72 marked [x]; protein timing renumbered to post-73 with clinical-safety warning; removed duplicate entries (old post-72/73 and post-75 allostatic duplicate)

---

## [1.1.1] — 2026-05-21

### Added
- `compliance/p1/sub-processor-register.md` — enterprise: GDPR Art. 28 / SOC 2 CC9.2 sub-processor register (P1-SUB-001); closes P-GAP-005 (standalone auditor exhibit); 8 sub-processors (SP-01 Anthropic, SP-02 Supabase, SP-03 Cloudflare, SP-04 ElevenLabs, SP-05 PostHog, SP-06 Sentry, SP-07 Apple, SP-08 Google); DPA & SCC status matrix; Art. 9 exposure per sub-processor (SP-01 coaching turns, SP-02 all health data — others excluded by design); Sentry compensating control documented with full TypeScript `beforeSend` PII scrubber + CI test requirement; tier classification (Tier 1: Art. 9 / Tier 2: personal / Tier 3: encrypted passthrough); sub-processor addition/removal workflows; customer DPA notice language (30-day advance, 10-day objection window); annual Q1 review procedure; gap status: PRE-02 🟡, PRE-08 🟡, PRE-09 🟡, CC9.2 🟢

---

## [1.1.0] — 2026-05-21

### Added
- `compliance/p1/retention-decisions.md` — enterprise: formal health-data retention decision record (P1-RET-001); closes P-GAP-004 (decision phase); per-category decisions for `workout_sessions` (3yr), `coaching_turns` (2yr), `cv_sessions` (1yr — biometric-adjacent, confirmed), `meal_log` (2yr), `wearable_readings` (2yr), `user_profile` health fields (until deletion/consent withdrawal); GDPR Art. 5(1)(e) legal basis per category; Supabase `pg_cron` TTL migration SQL; `data.retention_sweep_completed` DEC-030 event schema; privacy policy retention table template; gap-closure checklist (decision recorded — legal sign-off and implementation pending); annual review procedure; creates `compliance/p1/` directory

---

## [1.0.1] — 2026-05-21

### Added
- `content/post-71-allostatic-load.md` — sports-science: allostatic load framework (McEwen & Stellar 1993; McEwen 1998 NEJM), HPA axis physiology, shared-budget problem for training and life stress, HRV as allostatic proxy, 4-stage performance degradation sequence, load adjustment decision framework; clinical_safety_reviewed: PASS (physiological framing throughout, no mental health pathologising, no food content)
- `blog.html` — added cards for post-70 (MEV/MAV/MRV) and post-71 (allostatic load)
- `STATUS.md` — content engine table updated with post-70 and post-71 rows
- `README.md` — roadmap updated: post-70 and post-71 marked [x]; protein timing (formerly post-71) renumbered post-72 with clinical-safety warning flag

---

## [1.0.0] — 2026-05-21

### Added
- `docs/SOC2_READINESS.md §38` — Pre-Audit Readiness Assessment & Management Assertion (capstone): AICPA AT-C 205 management assertion draft, consolidated TSC scorecard (~95% overall), master open gap registry (50 gaps: 14 P0 / 28 P1 / 8 P2), auditor briefing package, evidence artifact master index (62 artifacts), 11-phase audit engagement timeline, binary readiness decision gate (14 P0 criteria + 10 P1 criteria). Document now auditor-presentable for kickoff call.
- `content/post-70-training-volume-mev-mav-mrv.md` — sports-science: MEV/MAV/MRV volume landmarks; Schoenfeld 2017 dose-response meta-analysis; individual variation as central thesis; clinical_safety_reviewed: PASS

### Milestone
- **v1.0.0** — перший "audit-ready" мілстоун: SOC 2 документ охоплює всі 5 TSC з §1–§38, має management assertion draft, 62 evidence artifacts, та binary decision gate для аудиторського engagement.

---

## [0.99.2] — 2026-05-21

### Added
- `content/post-70-training-volume-mev-mav-mrv.md` — sports-science: MEV/MAV/MRV volume landmarks framework; dose-response biology, Schoenfeld 2017 meta-analysis, individual variation as central thesis; clinical_safety_reviewed: PASS (MRV framed as ceiling, not target; no competitive volume framing)

---

| Bump | Коли |
|---|---|
| **PATCH** (`x.y.Z`) | Кожна cloud-ітерація. Один концепт = один bump. |
| **MINOR** (`x.Y.z`) | Нова фіча, новий розділ документації, помітна зміна. |

---

## [0.99.1] — 2026-05-21

### Added
- **`docs/SOC2_READINESS.md` §37** — **PI1 — Processing Integrity Deep-Dive** (compliance-officer + platform-engineer + ml-engineer). Extends the high-level §3 table into a full deep-dive matching the structure of §26–§35 (CC, A1, C1, P1–P8). Five processing surfaces mapped against PI1.1/PI1.2/PI1.3: (1) Victor AI coaching pipeline — `coaching_turns.status` state machine, clinical safety output filter, P95 latency SLO, three-gate authorization; (2) CV pose estimation — `REP_COUNT_CONFIDENCE_MIN = 0.65` threshold, `cv_partial` session status; (3) wearable data ingestion — anchor-based idempotency, cross-source HRV normalization, Garmin categorical exclusion; (4) nutrition/macro calculation — deterministic arithmetic constants, implicit-zero gap (PI-GAP-003), CI test gap (PI-GAP-004); (5) analytics ETL — PostHog→ClickHouse row-count cross-check gap (PI-GAP-005). DEC-030 HMAC chain established as cross-surface PI1 evidence layer with canonical three-event audit pattern and stuck-turn detection SQL. 6 control IDs (PIC-01–PIC-06). 5 output surfaces mapped to PI1.3. 7 gaps opened: PI-GAP-001 (stuck-row escalation P1), PI-GAP-002 (accuracy observability P1), PI-GAP-003 (meal log NOT NULL P2), PI-GAP-004 (nutrition CI test P2), PI-GAP-005 (ETL cross-check P1), PI-GAP-006 (SCIM RFC 7643 conformance P1 blocked on G-001), PI-GAP-007 (DSAR completeness count P2). 8 evidence artifacts (PRE-37-E-001–PRE-37-E-008). 9-item implementation checklist across M3/M4/post-launch. SOC 2 readiness: ~94% → ~95%.
- **`VERSION`** — 0.99.0 → 0.99.1

---

## [0.99.0] — 2026-05-21

### Added
- **`content/post-69-creatine.md`** — «Creatine: The Mechanism, the Evidence, and the Cases Where It Does Not Matter» (sports-scientist, clinical-safety PASS). Covers: PCr resynthesis mechanism and creatine kinase reaction; intramuscular PCr capacity and depletion timeline; Rawson & Volek (2003) meta-analysis (+8% strength, +14% reps-to-failure); Lanhers et al. (2017) upper body strength meta-analysis; Kreider et al. (2017) ISSN position stand; hypertrophy pathway (volume-mediated vs. direct cellular); non-responder phenotype (baseline saturation, vegetarian response); loading (20 g/day × 5–7 days) vs. maintenance (3–5 g/day × 28 days) equivalence; timing evidence (Antonio & Ciccone 2013, Candow 2014) with honest interpretation of small sample sizes; monohydrate vs. HCl/buffered/ethyl ester — no superiority evidence for alternatives (Spillane et al. 2009); intramuscular vs. subcutaneous water retention distinction; renal safety (Bizzarini & De Angelis 2004) and creatinine measurement confound; four scenarios where creatine is unlikely to matter: submaximal technique phases, aerobic-dominant training, near-saturated dietary intake, first 4 weeks for beginners. ~2,800 words. Clinical safety PASS: no dietary restriction framing, water retention contextualised as intramuscular, weight discussed only in training-load and lean-mass context.
- **`blog.html`** — added post-68 card (heat-cold-training, Незабаром, cat-science) and post-69 card (creatine, Незабаром, cat-science); blog now shows 69 posts
- **`STATUS.md`** — added post-68 and post-69 rows to content engine table
- **`README.md`** — post-69 marked complete with key references
- **`VERSION`** — 0.98.1 → 0.99.0

---

## [0.98.1] — 2026-05-21

### Added
- **`docs/SOC2_READINESS.md`** §36 — **Penetration Testing & Red Team Protocol** (security-engineer + compliance-officer). 504 lines. Covers seven attack-surface zones (external perimeter, web app, mobile MASVS L2, API, tenant isolation/lateral movement, social engineering, supply chain). Eight-phase PTES methodology with OWASP OTG v4, OWASP MASVS v2, OWASP API Security Top 10, NIST SP 800-115. Four engagement types with formal trigger conditions (annual black-box, pre-launch gray-box, post-major-change triggered, continuous DAST). CVSS 3.1-anchored severity/SLA table: Critical 24h, High 7d, Medium 30d, Low 90d. Five TC-RLS test cases: horizontal tenant access, JWT manipulation (alg:none + RS256→HS256 confusion), SECURITY DEFINER leak, SCIM over-provisioning wrong-tenant admin, DEC-030 HMAC chain tamper. Five Victor AI red team scenarios: prompt injection, medical advice extraction, PII exfiltration, context window poisoning, system prompt extraction — all cross-referenced to INCIDENT_RESPONSE.md R-10. Automated DAST vs. manual red team non-substitutability documented. 5 gaps opened: PEN-GAP-001/005 (no pentest or attestation letter — P0 enterprise gates), PEN-GAP-002/003/004 (CI DAST, RLS test execution, Victor AI test — P1). SOC 2 readiness: ~93% → ~94%.
- **`VERSION`** — 0.98.0 → 0.98.1

---

## [0.98.0] — 2026-05-21

### Added
- **`content/post-68-heat-cold-training.md`** — «Training in Hot and Cold Environments: Thermoadaptation, CWI, and the Adaptation Tradeoff» (sports-scientist, clinical-safety PASS). Covers: heat effects on contractile force and central fatigue (Racinais & Oksa 2010, Nybo et al. 2010); heat acclimation protocol (Nielsen 1993, 10–14 days, plasma volume expansion, HR reduction, relevance for strength athletes); CWI adaptation tradeoff — Roberts et al. (2015, *J Physiol*) CWI blunts satellite cell activation and mTOR signalling with attenuated hypertrophy/strength over 12-week block; Yamane et al. (2006) 4-week attenuation; Peake et al. (2017) systematic review; phase-based CWI decision rule (competition window vs. accumulation block); cold environment warm-up implications; sauna recovery evidence (Laukkanen). ~2,700 words. Clinical safety: no food content, no body image, heat illness addressed educationally with medical referral direction.
- **`README.md`** — post-68 marked complete; blog post count updated to 68
- **`VERSION`** — 0.97.1 → 0.98.0

---

## [0.97.1] — 2026-05-21

### Added
- **`docs/INCIDENT_RESPONSE.md`** v0.1 → v0.4 — **R-10 AI Coach Safety Incident (Victor Harmful Guidance)** (security-engineer + clinical-safety, enterprise-architect). First AI-specific runbook in FORM's incident taxonomy. Eight-level severity matrix: isolated quality issue (P2) → systematic harmful pattern (P1) → ED-adjacent content / injury report (P0) → prompt injection (P0 security reclassify). Three incident sub-types with distinct response branches: prompt regression (system prompt rollback, clinical-safety review gate, hotfix PR with clinical-safety as mandatory approver), model behavior drift (Anthropic safety team notification, R-07 parallel activation), prompt injection (P0 security incident, input sanitization, Victor Jailbreak Test Suite). Immediate actions: `coaching_turns` read-only query protocol — `content_hash` references only in incident channel; ED-adjacent raw text routed to restricted compliance folder (clinical-safety + compliance-officer + security-engineer + founder access only). Blast radius query by `victor_prompt_version`. Containment: feature-flag pathway scoping (not full-product disable), edge Worker content filter middleware, clinical-safety veto on re-enable decisions. Eradication: prompt diff methodology, `VICTOR_PROMPT_GUIDE.md` update gate, PostHog `feedback_negative` rate alert + Sentry `coaching_response_flagged` alert. Evidence package: SOC 2 CC7.4 / CC5.2 / CC1.2. Regulatory notes: GDPR Art. 34 physical injury path, Art. 33 prompt-injection health-data branch. No-go customer escalation: wellness-as-punishment detection → founder escalation. Appendix A updated with R-10 / prompt-injection / model-drift / wellness-as-punishment quick references. Header version corrected v0.1 → v0.4.
- **`VERSION`** — 0.97.0 → 0.97.1

---

## [0.97.0] — 2026-05-21

### Added
- **`content/post-67-foam-rolling.md`** — "Foam Rolling and Myofascial Release: What Works, What Is Ritual" (sports-scientist, clinical-safety PASS). Covers: why SMR cannot mechanically deform fascia (Behm & Wilke 2019 — forces required are 100× bodyweight); actual mechanisms (mechanoreceptor stimulation, gate control modulation, transient viscoelastic changes); ROM evidence (Cheatham et al. 2015 systematic review — acute ROM gain without performance decrease); DOMS evidence (Macdonald et al. 2013 RCT, Wiewelhove et al. 2019 meta-analysis — SMD −0.3–0.5); performance: no significant effect (Healey et al. 2014); pressure intensity vs. outcomes (tolerable pressure ≥ sharp pain for mechanoreceptor loading); 5-minute protocol for pre-training, 5–10 min post-training for DOMS management; what foam rolling cannot do (fix structural dysfunction, treat injury, replace mobility training); the ritual time-trap.
- **`blog.html`** — Post 66 (HRV for Strength Athletes) and Post 67 (Foam Rolling) cards added → 67 posts total
- **`README.md`** — roadmap: post-66 and post-67 checked [x]; post-75–80 new topics added
- **`STATUS.md`** — content engine table: post-66 and post-67 rows added; blog.html row updated to 67 posts / v0.97.0
- **`VERSION`** — 0.96.0 → 0.97.0

---

## [0.96.0] — 2026-05-21

### Added
- **`docs/DATA_MODEL.md`** v0.3 → v0.5 — **§14 Wearable Data Integration Schema** (platform-engineer + enterprise-architect). Five source integrations: HealthKit (iOS Background Delivery), Health Connect (Android WorkManager), Whoop, Oura Ring, Garmin. `wearable_readings` table DDL with 14 `reading_type` values, `(user_id, source, reading_type, recorded_at)` UNIQUE idempotency constraint, two composite indexes, and `deleted_at` soft-delete. Full RLS policies: user self-access, coach access behind consent gate, `form_admin` BYPASSRLS for erasure. `tenant_wearable_summary` aggregate view: k-anonymity floor (N ≥ 5), HRV trend direction only (no raw RMSSD), sleep duration buckets. Cross-source HRV normalization table (Garmin weekly HRV Status not comparable to daily RMSSD). Coaching context builder: `stripPersonalProperties()` extension, all wearable signals bucketed/directional — raw values never reach Anthropic API. GDPR Art. 9: explicit consent gate, 2-year retention, 30-day Art. 17 erasure window. Seven DEC-030 HMAC-chained audit events. `wearable_oauth_tokens` table with Supabase Vault encryption. Analytics restrictions table: wearable data excluded from all PostHog / ClickHouse / Metabase layers. 18-item implementation checklist (11× P0, 5× P1, 2× P2).
- **`content/post-66-hrv-strength-athletes.md`** — "HRV for Strength Athletes: How to Actually Use It" (sports-scientist + brand-voice, clinical-safety PASS). Covers: RMSSD mechanism (adenosine / autonomic NS); why strength athletes specifically benefit (CNS fatigue, Flatt & Esco 2016, Nuuttila et al. 2022); 7–14 day trend vs. single-day reading; personal baseline vs. population norms; morning measurement protocol; HRV-guided volume auto-regulation (Kiviniemi et al. 2010 RCT); high/mildly-suppressed/markedly-suppressed response heuristics; measurement confounds (alcohol, illness, cross-device incompatibility); when to ignore HRV (peaking, new block baseline period, illness); the anxiety trap (weekly trend-checking vs. daily obsession); FORM wearable data transparency statement.
- **`VERSION`** — 0.95.1 → 0.96.0

---

## [0.95.1] — 2026-05-20

### Changed
- **`docs/SOC2_READINESS.md`** v2.2 → v2.3 — **§35 P1–P8 — Privacy: Deep-Dive Control Implementation** (compliance-officer). Completes the SOC 2 deep-dive series: §33 A1 Availability (v2.1), §34 C1 Confidentiality (v2.2), §35 P-series Privacy (v2.3). Full mapping of all 16 AICPA P-series sub-criteria against GDPR article-level obligations. GDPR ↔ P-series mapping table (§35.2) establishes dual-compliance framework: DEC-030 HMAC-chained audit events serve simultaneously as SOC 2 evidence and GDPR Art. 5(2) accountability artifacts. 53 controls (PRV-01–PRV-53): Art. 13 notice delivery architecture with scroll-completion gate and `consent_version` stamping; five-category granular consent model with DEC-030 `privacy.consent_granted` / `privacy.consent_withdrawn` event schema; data minimization (CV on-device inference, LLM prompt PII stripping, PostHog Art. 9 field blocklist); §35.6.3 health data retention schedule (workout_sessions 3yr, coaching_turns 2yr, cv_sessions 1yr, wearable_readings 2yr, meal_log 2yr); §35.7.2 authoritative DSAR five-step process flow with 30-day GDPR Art. 15 SLA; §35.8.2 four-principle government request handling policy; 72h/1h breach notification SLAs. 8 gaps: P-GAP-001 (privacy policy not live — P0), P-GAP-002 (sub-processor list not published — P0), P-GAP-003 (cookie banner — P1), P-GAP-004 (retention periods TBD — P1), P-GAP-005 (DSAR untested — P1), P-GAP-006 (gov request policy — P1), P-GAP-007 (no complaint intake — P1), P-GAP-008 (annual review — P2). 10 evidence artifacts (PRE-35-E-001–010). **Readiness: ~92% → ~93%**.
- **`VERSION`** — 0.95.0 → 0.95.1

---

## [0.95.0] — 2026-05-20

### Added
- **`content/post-65-caffeine-strength-training.md`** — editorial-brutalist пост «Caffeine and Strength Training: What the Evidence Actually Shows». Adenosine receptor antagonism як центральний механізм (не «енергія» — блокування сигналу втоми). Доказова база: Warren et al. (2010) +3.5% MVC, Grgic et al. (2018) Hedges' g ≈ 0.20 для сили, (2019) g ≈ 0.35–0.48 для потужності і wingate. Dose-response: 3–6 мг/кг за 45–60 хв. Tolernce dynamics: receptor upregulation, часткове збереження performance benefit, 5–7-day washout. CYP1A2 polymorphism: Cornelis et al. (2006) — slow metabolizers можуть не отримати користь або деградувати. Сон: half-life 4–6 год (fast) до 9–10 год (slow) — interference зі SWS недооцінена. Contraindications: anxiety, arrhythmia, pregnancy, drug interactions. Victor/FORM секція: індивідуальне dose-response tracking, tolerance creep detection, sleep trade-off signal. Clinical-safety PASS.
- **`blog.html`** — додано 5 відсутніх карток: post-33 (Energy Systems), post-34 (Supersets/Antagonist Pairs), post-35 (Tempo Training), post-64 (Plyometrics/RFD), post-65 (Caffeine). Grid тепер має 65 карток.

### Changed
- **`README.md`** — roadmap оновлено: post-64 і post-65 позначено [x]; додано нові теми post-66–74 (allostatic load, foam rolling, heat/cold, creatine, sleep/recovery, protein timing, lifestyle capacity, grip longevity, unilateral training).
- **`STATUS.md`** — blog.html рядок оновлено (16 → 65 posts, v0.95.0); додано рядки для post-64 і post-65.
- **`VERSION`** — 0.94.1 → 0.95.0

---

## [0.94.1] — 2026-05-20

### Changed
- `docs/SOC2_READINESS.md` v2.1 → v2.2 — **§34 C1 — Confidentiality: Deep-Dive Control Implementation** (compliance-officer + security-engineer). Full C1.1 and C1.2 control mapping. C1.1 (8 sub-criteria): confidential data classification (✅ Done — §5 five-tier policy cross-referenced); data inventory (🟡 Partial — DATA_MODEL.md + §5 mapped as de facto inventory; standalone signed exhibit C1-GAP-002 surfaced); access controls (✅ Done — Supabase RLS row-level isolation, RBAC four roles, break-glass dual-auth); AES-256 at rest + TLS 1.3 in transit (✅ Done — Supabase default + column-level `pgp_sym_encrypt`); workforce NDAs (🔴 Gap → C1-GAP-001 P0 pre-hire gate); third-party DPAs (🟡 Partial — C1-GAP-003 P1, 7 sub-processors with filing paths); data minimization in LLM prompts (✅ Done — no PII in Anthropic API calls, `stripForbiddenProperties` middleware); DEC-030 HMAC access monitoring (✅ Done). C1.2 (5 sub-criteria): GDPR Art. 17 erasure 30-day grace + hard delete + 60-day backup purge (✅ Done); ClickHouse erasure SQL template (✅ Done); audit log anonymization on delete (✅ Done — `[DELETED-{hash}]` replacement, HMAC chain preserved); media/device disposal (🔴 Gap → C1-GAP-004 P1 NIST SP 800-88); end-of-employment offboarding (🟡 Partial — 4-hour access revocation SLA, not yet executed). 4 gaps (C1-GAP-001 P0, C1-GAP-002/003/004 P1). 8 evidence artifacts PRE-34-E-001 through PRE-34-E-008. 10-item checklist (2× P0, 6× P1, 2× P2). **Readiness: ~91% → ~92%**.

---

## [0.94.0] — 2026-05-20

### Added
- **`content/post-64-plyometrics-strength-athletes.md`** — editorial-brutalist пост «Plyometrics for Strength Athletes: Rate of Force Development and the Power Transfer Question». SSC механіка (slow-SSC GCT >250ms vs fast-SSC <250ms), RFD як gap силового тренінгу (0–200ms вікно), дослідження: Rimmer & Sleivert (2000), Arabatzi & Kellis (2012), Saez de Villarreal et al. (2012 JSCR meta-analysis), tendon adaptation lag Magnusson/Bohm. Програмування в foot contacts (80–100 FC beginner → 200+ advanced), intensity categories (low/medium/high), session placement rules (до силової, не після hypertrophy volume), хвилеподібна прогресія замість лінійної. Типові помилки: занадто швидке збільшення об'єму (10% правило → tendon lag constraint), soft surfaces only, ignoring landing quality, лінійний overload. FORM/Victor секція: три умови перед додаванням плайометрики (training age + strength base + recovery capacity). Clinical-safety PASS.

---

## [0.93.1] — 2026-05-20

### Changed
- `docs/SOC2_READINESS.md` v2.0 → v2.1 — **§33 A1 — Availability: Deep-Dive Control Implementation** (compliance-officer + security-engineer + devops-lead). Closes the final three open gaps in the §2 Availability overview table that had no corresponding implementation section. Three-tier capacity architecture formally documented: Cloudflare Workers (no capacity risk pre-$500k ARR), Supabase PgBouncer (DB connection-wait p99 > 10ms / 48h = upgrade trigger to Supabase Enterprise), AI inference (ElevenLabs character budget risk formally quantified — ~84× overrun at 10k DAU on Business plan 500k chars/month; Scale/Enterprise upgrade required pre-launch; upgrade path documented). Capacity upgrade triggers table for 6 metrics with thresholds, actions, and owners. **Load testing program** (closes 🟡 Gap): k6 suite, 5 scenarios with pass criteria — baseline (100 VUs, 5 min), enterprise SSO login burst (500 VUs, 30s ramp), coaching session load (1,000 VUs), DB connection saturation (450 concurrent connections), SCIM provisioning burst (500 ops/60s); CI gate integration; evidence filed to `compliance/evidence/load-tests/`. **SLA Framework** (closes 🟡 Gap): four-tier availability commitments (Starter 99.5% API / Growth 99.9% / Enterprise 99.9% API + SSO); downtime definition anchored to probe S-001 (OBSERVABILITY §16.2) with 90-second detection window (3 consecutive 30s-probe failures); credit schedule 10%/25%/50%/100% by availability band; automatic application within 5 business days; Template E-04 (INCIDENT_RESPONSE §12.5) as notification mechanism; monthly SLA report Worker spec (Better Stack → JSON → R2 `compliance/evidence/sla-reports/YYYY-MM.json`); privacy floor enforcement (credit log stores `{tenant_id, credit_amount}` only — no individual data); two new DEC-030 HMAC-chained events (`billing.sla_credit_applied`, `system.dr_drill_completed`). **A1.2 cross-reference**: RTO/RPO scenario map for 8 failure scenarios (Worker rollback <5 min / Pages reactivation <5 min / PITR 2–4h / region failover 1–4h / R2 nightly 4–8h / B2 WORM 24–48h / AI outage N/A / TTS outage <1 min); health-data anonymisation script requirement on non-prod restores (PRE-33-E-005). **DR Drill procedure** (closes 🔴 Gap): 5-step scope with quantitative pass criteria (≤1% row count discrepancy, zero RLS violations, RTO < 4h wall-clock, two-person rule); 6-artifact evidence package (PRE-33-E-006 drill log / PRE-33-E-007 row counts / PRE-33-E-008 RLS test / PRE-33-E-009 cluster destroy / PRE-33-E-010 sign-off / PRE-33-E-011 DEC-030 event); FAIL protocol (P1 Linear ticket, 14-day remediation, full re-drill if RLS fails); annual Q1 schedule requirement in §15 compliance calendar. 11-artifact evidence catalog (PRE-33-E-001 through PRE-33-E-011). 10-item implementation checklist (4× P0 pre-launch: PgBouncer config, ElevenLabs upgrade, DEC-030 events, k6 suite; 4× P1: SLA report Worker, anonymisation script, §15 calendar, first DR drill; 2× P2 trigger-based). Gap closures: "DR test performed annually" 🔴 → 🟡 Partial; "Load testing before launches" 🟡 Gap → 🟡 Partial; "SLA credit calculation and process" 🟡 Gap → 🟡 Partial. SOC 2 header version corrected v1.6 → v2.1 (header was not bumped during §27–§32 additions). **Readiness: ~90% → ~91%**.

---

## [0.93.0] — 2026-05-20

### Added
- **`content/post-63-women-strength-menstrual-cycle.md`** — 13-хв editorial-brutalist пост про менструальний цикл і силовий тренінг; McNulty et al. (2020) мета-аналіз як основа; гормональна контрацепція addressed directly; effect sizes reported accurately (~1.8% follicular/luteal); clinical-safety PASS WITH CONDITIONS; autoregulation framing ("listen and adjust", не "track to optimize")
- **`blog.html`** — картка post-63 додана до articles grid
- **`STATUS.md`** — рядок content engine table для post-63

---

## [0.92.0] — 2026-05-20

### Added
- **`compliance/`** — новий каталог аудиторських артефактів SOC 2 Type II
  - [`compliance/README.md`](compliance/README.md) — індекс каталогу, посилання на docs/SOC2_READINESS.md, таблиця `evidence/` path-patterns
  - [`compliance/cc4/control-deficiency-log.csv`](compliance/cc4/control-deficiency-log.csv) — **CC4-E-001** (CC4-GAP-001, P1): структурований CSV-реєстр усіх відомих дефектів контролю (8 pre-launch записів: CC1-GAP-001, CC2-GAP-002, CC3-GAP-001/002/003, CC4-GAP-001/002/003); обов'язкові поля: `deficiency_id`, `criterion`, `description`, `severity`, `root_cause`, `linear_ticket_url`, `owner`, `target_date`, `closed_date`, `auditor_exhibit_ref`; CC4-GAP-001 позначений closed (remediated цим же CSV)
  - [`compliance/cc4/evidence-collection-plan.md`](compliance/cc4/evidence-collection-plan.md) — **CC4-E-002** (CC4-GAP-002, P1): план збору доказів для 5 ongoing + 3 separate evaluation controls (CC4.1); для кожного control: тип доказів, частота, storage path, відповідальний owner, start date (launch); перша точка перевірки — T+30 post-launch
  - [`compliance/cc4/deficiency-communication-procedure.md`](compliance/cc4/deficiency-communication-procedure.md) — **CC4-E-003** (CC4-GAP-003, **P0**): трирівнева компенсуюча процедура для CC4.2 (відсутня рада директорів); Tier 1: щомісячний memo founder-а (git-commit підпис, шаблон), Tier 2: щоквартальна нотифікація advisory board (кворум, шаблон agenda, 5-day written summary), Tier 3: quarterly investor observer update (шаблон секції); P0 exception: 24h notification для P0 дефектів; Series A trigger — процедура expire-ує і замінюється audit committee charter впродовж 90 днів після закриття
  - `compliance/evidence/` — порожній каталог (заповнюється після launch)
- [`content/post-62-velocity-based-training.md`](content/post-62-velocity-based-training.md) — **Velocity-Based Training: What the Research Actually Says**; clinical-safety: not-required
  - §1 What VBT Is: MPV vs MV distinction, device accuracy spectrum (GymAware LPT > Push/BEAST accelerometers > smartphone apps), practical stakes of device choice
  - §2 Load-Velocity Profile: González-Badillo & Sánchez-Medina (2010) — every %1RM maps to a narrow velocity window; squat benchmarks (~1.0 m/s at ~40%, ~0.3 m/s near max); 1RM estimation without maxing
  - §3 Velocity Loss Thresholds: Pareja-Blanco et al. (2017, Eur J Sport Sci) — 20% vs 40% VL in squat (neuromuscular efficiency vs hypertrophy stimulus tradeoff); Sanchez-Medina & González-Badillo (2011) on fatigue-velocity relationship; programmable variable framing, not universal prescription
  - §4 1RM Estimation Accuracy: Jidovtseff et al. (2011) bench r=0.97; 5% day-to-day velocity variation propagates error; readiness monitoring application
  - §5 What VBT Doesn't Solve: program design, exercise selection, equipment accuracy degradation at high loads
  - §6 FORM Without a Transducer: maximal intent cues — Behm & Sale (1993), Sakamoto & Sinclair (2006); CV pipeline tempo change as velocity loss proxy
  - §7 Bottom Line: gear vs principle, anti-hype close

### Changed
- [`blog.html`](blog.html) — картка post-62 (Velocity-Based Training) додана до grid
- [`README.md`](README.md) — post-62 позначено [x] (VBT); жіноче тренування зсунуто на post-63; foam rolling → post-64; кофеїн → post-65
- [`STATUS.md`](STATUS.md) — рядок post-62 доданий до таблиці content engine (clinical-safety: not-required)

---

## [0.91.1] — 2026-05-20

### Changed
- [`docs/OBSERVABILITY.md`](docs/OBSERVABILITY.md) → **v0.4**: added §17 Cost Monitoring & Anomaly Alerting; doc header version bumped from v0.2 to v0.4; §10 "Cost monitor Worker" gap entry updated to "Design complete — see §17."
  - **§17.1 Architecture**: daily Cloudflare scheduled Worker (Cron Trigger `0 6 * * *`) fetching Anthropic Admin API (input/output token counts), ElevenLabs monthly character delta via KV namespace, Cloudflare GraphQL Analytics (Workers requests). Supabase Postgres as the cost store; separate PagerDuty service "FORM Cost" with low-urgency routing to avoid paging on-call for non-user-impacting cost spikes.
  - **§17.2 Schema**: `cost_daily` table with generated `variance_pct` column (actual vs. COST_MODEL.md projection); `cost_anomalies` table for immutable anomaly log; `form_cost_monitor` Postgres role with write-only grants and no RLS bypass.
  - **§17.3 Thresholds**: per-service P1 (1.5× 7d rolling avg) and P0 (3× 7d rolling avg) anomaly thresholds derived from COST_MODEL.md §13.1; externalised to `COST_THRESHOLDS` Workers Secret for hot-update without redeploy; Supabase absolute thresholds (6 GB / 7.5 GB DB size) aligned to the $25 Pro → $599 Team plan cliff.
  - **§17.4 Worker implementation**: TypeScript pseudocode for `fetchAnthropicUsage` (Anthropic Admin API `/v1/usage`), `fetchElevenLabsUsage` (cumulative-delta via KV, 7-day TTL), `fetchCloudflareUsage` (GraphQL Analytics); `detectAnomalies` against externalized thresholds; `Promise.allSettled` for partial-failure tolerance; KV delta logic for ElevenLabs character accounting.
  - **§17.5 Per-tenant COGS**: `cost_daily_per_tenant` materialized view from `coaching_sessions` (token + voice char telemetry per tenant); `pg_cron` refresh at 07:00 UTC; per-tenant anomaly thresholds (margin-negative single-day = P0; 5× spike = P1); privacy constraint: view restricted to `form_admin` only — not exposed to enterprise admin dashboard.
  - **§17.6 Dashboards**: three Metabase specs — Infra Cost Overview (six panels: daily COGS by service, 7d trend, COGS/Pro MAU, Anthropic split, ElevenLabs %, MTD run rate); Enterprise Tenant Cost panel (restricted, four panels including COGS vs. contracted revenue scatter); Cost Anomaly History (open anomalies, last-30d resolved, P0 frequency trend).
  - **§17.7 Alerting**: PagerDuty event payload schema with `dedup_key` on `service + metric + date`; Slack `#metrics` daily digest format (06:10 UTC) with and without anomaly section; urgency routing table distinguishing cost-only P0 (low urgency, business-hours response) from service-impacting cost anomaly (routes to FORM Production, high urgency).
  - **§17.8 SOC 2 evidence**: CC7.2 (continuous monitoring), CC7.3 (anomaly evaluation with immutable `cost_anomalies` log), CC5.2 (per-tenant resource accounting as technology control), A1.2 (cost spike as leading capacity indicator before rate-limit availability event); monthly CSV export to `compliance/evidence/cost-daily-YYYY-MM.csv`.
  - **§17.9 Checklist**: 16 tasks across M3/M4/M5 — P0: migrations, Worker code, cron trigger, four secrets, KV namespace, Postgres role; P1: PagerDuty service, Slack webhook, pg_cron refresh, three Metabase dashboards; P2: est_cost_usd seed, compliance CSV export, ElevenLabs history endpoint upgrade.

---

## [0.91.0] — 2026-05-20

### Added
- [`content/post-61-overreaching-vs-overtraining.md`](content/post-61-overreaching-vs-overtraining.md) — operational diagnostic framework for FO/NFO/OTS: Meeusen et al. 2013 ECSS/ACSM consensus, HRV trend, RPE drift, bar speed decline, decision tree by state, recovery protocols; clinical-safety reviewed ✓
- [`blog.html`](blog.html) — cards for post-60 (deload protocols) and post-61 (overreaching vs overtraining)
- [`README.md`](README.md) — roadmap extended to post-69; post-60 and post-61 marked done; topics post-62–69 queued
- [`STATUS.md`](STATUS.md) — content engine table: entries added for post-60 and post-61

---

## [0.90.0] — 2026-05-20

### Added
- [`docs/SOC2_READINESS.md`](docs/SOC2_READINESS.md) §31 — CC3 Risk Assessment: усі 4 sub-criteria mapped (CC3.1–CC3.4); §14 Risk Register cross-referenced як CC3.2 control; DEC-030 HMAC log + clinical-safety VETO as CC3.3 anti-fraud controls; 3 gaps documented (CC3-GAP-001 risk appetite statement, CC3-GAP-002 fraud risk table, CC3-GAP-003 org-change checklist)
- [`docs/SOC2_READINESS.md`](docs/SOC2_READINESS.md) §32 — CC4 Monitoring Activities: CC4.1 mapped (5 ongoing + 3 separate evaluation mechanisms); CC4.2 mapped (4-layer deficiency communication); CC4-GAP-003 (P0) board-less compensating control documented with Series A upgrade trigger

### Changed
- SOC 2 readiness: **~86% → ~90%** — всі 9 Common Criteria (CC1–CC9) тепер формально mapped

---

## [0.89.2] — 2026-05-20

### Added
- [`content/post-60-deload-protocols.md`](content/post-60-deload-protocols.md) — deload protocols: supercompensation theory, 4 deload types, reactive vs proactive triggers, practical examples; clinical-safety reviewed ✓

---

## [0.89.1] — 2026-05-20

### Changed
- `docs/COST_MODEL.md` → **v0.6**: added §15 Enterprise Infrastructure COGS Deep-Dive and §16 Enterprise Tier Break-Even by Plan.
  - **§15**: per-feature marginal cost quantification for every enterprise-specific capability — SSO/SAML token validation ($0.0000065/seat/month), SCIM provisioning ($0.0000002), admin dashboard queries ($0.0000002), 7-year audit log storage hot + cold tier ($0.0000001 each), SIEM webhook delivery ($0.0000075), white-label DNS/SSL/logo (~$0.001/tenant), multi-tenant RLS ($0). Consolidated summary: all enterprise features together add < $0.00001/seat/month — less than 0.003% of the $0.34 baseline COGS. Explains why enterprise infrastructure gross margin reaches 97%+.
  - **§16**: plan-level break-even table across Starter (50–200 seats, $12/seat), Growth (200–1,000 seats, $9/seat), Enterprise (1,000+, $6–8/seat) incorporating infra COGS + implementation amortization + CS labor. Key finding: 50-seat Starter Y1 GM = 47.9%; 20-seat deal is margin-negative in Y1 (−26.1%). Y1 vs Y2+ improvement table (Starter 50-seat: +15.9 pp when implementation drops to zero). Multi-year contract economics: 25% 3-year discount gives up $13,800 GP but eliminates ~$32,400 of churn-exposure ARR on a 200-seat Growth deal.
  - **OQ-11** added to §11: minimum seat threshold for acceptable Y1 Starter margin — track actual CS hours on first 5 Starter deals before committing to sub-50-seat pilots.
  - ToC updated for §15 and §16. Header and version footer bumped to v0.6.

---

## [0.89.0] — 2026-05-20

### Added
- `content/post-59-zone2-cardio.md` — **Zone 2 Hype vs. Zone 2 Evidence**. LT1 physiology: mitochondrial biogenesis via PGC-1α (Holloszy 1967, Hood et al. 2000). Seiler 80/20 polarized model (Seiler & Kjerland 2006, Seiler 2010): elite endurance athletes at 10–15 h/week — structural context that created the distribution, not a universal prescription. HIIT equivalence: Gibala et al. (2006), Burgomaster et al. (2008) — equivalent mitochondrial stimulus in less time. Concurrent training interference: Wilson et al. (2012) meta-analysis — interference is dose-, mode-, and frequency-dependent; cycling > running for minimizing attenuation. Legitimate Zone 2 applications: CVD risk reduction, work capacity for long sessions, active recovery (true LT1 only), GPP for advanced athletes. What it is not: fat-loss mechanism, program substitute. FORM tracking proxy: resting HR trend, HRV baseline, workout HR response. clinical-safety: not-required. ~1,100 words.
- `blog.html` — картки post-58 (Mind-Muscle Connection) і post-59 (Zone 2) додані до grid.
- `STATUS.md` — рядки post-58 і post-59 додані до content engine таблиці.
- `README.md` — content engine roadmap: post-56–59 позначено [x] з актуальними темами; додані future topics post-60–65.

---

## [0.88.0] — 2026-05-20

### Added
- `docs/SOC2_READINESS.md §29` — **CC1 Control Environment**: COSO Principles 1–5 mapped to solo-founder compensating-control framework. CC1.1 (integrity/ethics — AUP gap CC1-GAP-001 P0, founder CoC in git as compensating control); CC1.2 (board oversight — advisory board charter + compliance calendar self-review cadence + counsel escalation path; CC1-GAP-002 accepted at pre-seed, expires at Seed close); CC1.3 (org structure — ENTERPRISE.md role taxonomy + .claude/agents/ functional RACI + ICS structure; CC1-GAP-005 org chart artifact); CC1.4 (competence — background check provider CC1-GAP-003 P0, self-training log CC1-GAP-007); CC1.5 (accountability — 🟢 confirmed via HMAC-chained audit log DEC-030, daily chain check §25.5, break-glass post-hoc justification). 5 gaps CC1-GAP-001 through -005; implementation checklist; evidence mapping table. SOC 2 readiness: ~82% → ~84%.
- `docs/SOC2_READINESS.md §30` — **CC2 Communication and Information**: COSO Principles 12–14. CC2.1 (internal comms — git-as-policy-store, compliance calendar, `#inc-YYYYMMDD-slug` incident channel convention; policy approval log CSV gap CC2-GAP-001 P1); CC2.2 (🟡 — hiring/ JDs, ENTERPRISE.md SLA, INCIDENT_RESPONSE.md §2.2 on-call mapped; solo-founder permanent-on-call compensating with expiry trigger CC2-GAP-005); CC2.3 (external — privacy policy at docs/PRIVACY_POLICY.md P0 gate CC2-GAP-002; sub-processor list CC2-GAP-003 P0 blocks enterprise DPA and GDPR Art. 28; Art. 33 dry-run gap CC2-GAP-006). 6 gaps including 2 🔴 blockers. 17-item implementation checklist. Cross-section dependency table (CC5-P1-004, CC9-GAP-007, CC1-GAP-005). SOC 2 readiness: ~84% → ~86%.
- `content/post-58-mind-muscle-connection.md` — **Mind-Muscle Connection Is Real — With Conditions**. Wulf et al. (2001) constrained action hypothesis: internal focus disrupts motor automaticity in skill/velocity tasks → external focus wins for compounds. Calatayud et al. (2016): EMG amplitude significantly higher with internal focus for curls and chest flies — load matched. Snyder & Fry (2012): biceps EMG ↑ with internal focus instruction. Practical split: compound/velocity lifts → external cues (outcome-based: "push the floor away"); isolation/single-joint → internal focus (target tissue activation). Training age dimension: beginners lack motor automation for internal focus on compounds. FORM application: cue delivery is exercise-specific, not interchangeable filler. ~850 words.

### Changed
- `docs/COST_MODEL.md` — header corrected v0.4 → v0.5; ToC updated to include §13 (Infrastructure Cost Breakdown by Service) and §14 (Cohort LTV Model & CAC Payback Period) which were added in v0.5 but missing from the ToC.

---

## [0.87.1] — 2026-05-20

### Added
- `docs/INCIDENT_RESPONSE.md §§R-08, R-09, 9.3–9.5, 12` — v0.3 extension: **R-08 Supply Chain Attack / Compromised Dependency** (blast radius matrix by package location and runtime access surface; deploy-freeze protocol; secrets rotation; CI preventive controls: Dependabot + npm audit + Socket.dev; SOC 2 CC6.1/CC6.8/CC7.4 evidence package). **R-09 Ransomware / Destructive Payload** (always-P0; isolation-first protocol overriding standard evidence-first rule; no-pay policy with decision authority constraints; Supabase PITR recovery to new project; full secrets rotation sequence; enterprise tenant validation before re-enabling traffic; law enforcement preservation note; GDPR Art. 33 applicability for destruction). **§9.3 Scenarios E & F** — tabletop scenarios: supply chain (backdoored npm package in Workers env) and ransomware (staging-to-production lateral movement); both require customer-success participation. **§9.5 Year 2 Testing Schedule** — 6-scenario rotation across all runbook types. **§12 Enterprise Tenant SLA Breach & Incident Communication Protocol** — SLA breach thresholds (99.9% API, 99.5% SSO); credit schedule (10/25/50% by uptime band); contact ownership matrix with per-severity timing SLAs; privacy floor enforcement for tenant communications; Templates E-01 (initial notification), E-02 (60-min status update), E-03 (resolution), E-04 (SLA credit); SOC 2 evidence mapping (CC9.2, A1.1, A1.2, CC7.3, CC2.2). Appendix A updated to reference R-08, R-09, and §12 enterprise comms protocol. INCIDENT_RESPONSE doc → v0.3.

---

## [0.87.0] — 2026-05-20

### Added
- `content/post-57-training-after-40.md` — **Тренінг після 40: саркопенія, анаболічна резистентність і програмування під реальну фізіологію** (sports-scientist pending, clinical-safety not-required). Саркопенія: Frontera et al. (2000) — 14% втрати сили/десятиліття після 50; Evans (2010) — 1–2%/рік; Fragala et al. (2019) NSCA — диспропорційна втрата волокон типу II. Анаболічна резистентність: Damas et al. (2021), Kumar et al. (2009) — знижений МПС відгук; Wilkinson et al. (2015) — mTORC1-залежний МПС на ~30% нижчий у 60+. Гормони: Bhasin et al. (2001) — тестостерон -1–2%/рік після 35; Iranmanesh et al. (1991) — GH -14%/десятиліття. Відновлення: Zbinden-Foncea et al. (2020) — EIMD-вікно +24–48 год. Що не змінюється: Peterson et al. (2011) — приріст 1.1 кг у 60–70; нейральна адаптація. Практичні модифікації: інтенсивність > об'єм, 4-тижневий деловк, розминка обов'язкова, авторегуляція критична. Victor: вік як вхідний параметр для деловк-логіки і recovery window. ~2,700 words.
- `blog.html` — картки post-56 (Periodization Models) і post-57 (Training after 40) додані до grid.
- `STATUS.md` — рядки post-56 і post-57 додані до content engine таблиці.

---

## [0.86.0] — 2026-05-20

### Added
- `docs/OBSERVABILITY.md §16` — Synthetic Monitoring & Uptime Verification: twelve-probe taxonomy S-001–S-012 (API health, auth, workout logging, AI coaching turn, SSO SAML metadata, SCIM, HMAC audit chain, backup freshness, CDN, status page, Stripe webhook, PostHog ingest); Better Stack declarative YAML config; Cloudflare scheduled Worker implementations for S-007 (HMAC chain) and S-008 (backup freshness) with TypeScript; multi-region strategy (us-east-1, eu-central-1, ap-southeast-1), 2-of-3 failure threshold; regional latency SLO table; post-deployment smoke gate (GitHub Actions, 120s, P1 alert); synthetic user data hygiene across 7 pipelines; PagerDuty payload schema; SOC 2 CC7.2/CC7.3/A1.2/CC4.1 evidence mapping; 13-item implementation checklist (M3/M4/M5). Observability doc → v0.3.
- `content/post-56-periodization-models.md` — **Periodization Models: Linear, Undulating, and Block** (sports-scientist, clinical-safety-reviewed: no food/injury/pain content). LP structure with 8-week example table; DUP with Rhea et al. (2002) and Prestes et al. (2009) research basis; block periodization (accumulation → transmutation → realization), residual training effects (strength ~30d, aerobic ~15–25d), conjugate variant; model-selection matrix (training age × schedule × goal); RPE/RIR autoregulation integration note. ~1,100 words.
- `docs/SOC2_READINESS.md §28` — CC9 Risk Mitigation & Vendor Risk Management: CC9.1 risk mitigation register (10 risk categories, explicit transfer/reduce/accept/avoid strategy, residual risk levels, evidence artefacts); insurance program roadmap (cyber liability $2M, D&O $2M, E&O $1M, pre-Series A timeline); CC9.2 full vendor risk assessment table covering 10 vendors (Anthropic, ElevenLabs, Supabase, Cloudflare, PostHog EU, Better Stack, Expo/EAS, Stripe, 1Password, GitHub) with criticality, SOC 2 status, DPA status, SLA, contingency; Cloudflare concentration risk formally documented with acceptance statement; Expo/EAS gap (no SOC 2, no DPA) surfaced as CC9-GAP-003; 8-item implementation checklist CC9-GAP-001–008; CC9 gap analysis 14 sub-criteria; SOC 2 readiness ~77% → ~82%. SOC2 doc → v1.8.
- `docs/DATA_MODEL.md §13` — Analytics Event Tracking Schema: PostHog event taxonomy (9 categories, 29 named events with typed properties, universal enriched properties); Cloudflare Worker enrichment middleware with Zod validation and forbidden-property scrubbing (`stripForbiddenProperties` — 16 prohibited fields); ClickHouse warehouse: 4 tables (fact_events ReplacingMergeTree 2yr TTL, dim_users, fact_cohort_retention AggregatingMergeTree with k-anon floor N<5, fact_session_metrics); hourly + daily ETL pipeline via PostHog S3 export → ClickHouse → dbt; GDPR Art. 9 analytics prohibition table (10 categories); Art. 17 erasure job; 19-item checklist (M3/M4/M5). DATA_MODEL doc → v0.4.

---

## [0.85.1] — 2026-05-19

### Added
- `content/post-55-training-splits.md` — **Тренувальні сплити: доказова матриця вибору** (sports-scientist pending, clinical-safety not-required). Full body vs Upper/Lower vs PPL vs body-part спліти: чотири архітектури з доказовою базою. Schoenfeld et al. (2016): вища частота переважає при рівному об'ємі. Ralston et al. (2017): Cohen's d 0.25 при рівному об'ємі. McLester et al. (2000): 3 vs 1 сесія при рівному об'ємі. Colquhoun et al. (2018): full body vs upper/lower — adherence вища. Krieger (2010): 2–3 якісних сети > 5 suboptimal. Матриця вибору: тренувальний стаж × стабільність розкладу × мета. Три конкретних сценарії: новачок 3 дні → full body; intermediate 4 стабільних дні → верх/низ; intermediate нестабільний → full body з акцентами. FORM Victor: спліт — похідна від реального розкладу, не навпаки. ~2,600 words.
- `blog.html` — картки post-54 (MEV/MAV/MRV volume landmarks) і post-55 (Training splits) додані до grid.
- `STATUS.md` — рядки post-54 і post-55 додані до content engine таблиці.
- `README.md` — roadmap оновлено: post-54 [x] і post-55 [x] відмічені як виконані; post-56–59 додані як наступні теми (жінки і тренінг, overreaching, після 40, AI-coach vs PT).

## [0.85.0] — 2026-05-19

### Added
- `docs/SOC2_READINESS.md §27` — CC5 Control Activities: risk-to-control mapping (10 categories), IaC baseline table, configuration standards, policy inventory (12 policies, AUP + Cryptography Policy as 🔴 gaps), solo-founder compensating control narrative, AUP draft in Annex A; 5 open items CC5-GAP-001–005; readiness ~77% (no regression)
- `docs/SSO_SCIM_IMPLEMENTATION.md §15` — SCIM 2.0 Groups Provisioning: RFC 7643 §8 schema, `scim_groups`/`scim_group_members` tables with RLS, 6 SCIM endpoints, group-to-role `highest_privilege` algorithm, Okta/Entra/Google Workspace IdP specifics, nested group flattening (§15.6.2a), Art. 9 group name scanner, 9 audit events (DEC-030 HMAC chain), G-014 gap (Google Workspace by-design limitation); 15-item implementation checklist

---

## [0.84.2] — 2026-05-19

### Added
- `content/post-54-training-volume-landmarks.md` — MEV/MAV/MRV: training volume landmarks explainer; Schoenfeld 2017 dose-response, Israetel/RP landmark estimates, Victor voice, clinical-safety clean

## [0.84.1] — 2026-05-19

### Changed
- `docs/SOC2_READINESS.md` v1.5 → v1.6 — **§26 CC6 Logical and Physical Access Controls** (compliance-officer + security-engineer + enterprise-architect). Full CC6.1–CC6.8 criteria mapping with FORM-specific implementation per sub-criterion. Identity and authentication architecture diagram (IdP/SAML/OIDC → Supabase Auth → API; internal Cloudflare Access layer). MFA enforcement matrix for all six admin surfaces (Cloudflare, Supabase, GitHub, 1Password, AWS, PostHog) with compensating-control narrative for solo-founder separation-of-duties. Break-glass PAM procedure formally mapped to CC6.1 with auditor narrative (compensating controls for pre-team entity accepted by AICPA). User lifecycle: three provisioning flows (SCIM auto, manual invite, consumer self-serve); role modification with atomic SQL + audit_log write in same transaction; deprovisioning SLA ≤24h; full internal engineer offboarding checklist (11 checklist items covering all credential surfaces). Physical access and device policy for remote-work context: FileVault 2, MDM enrolment, VPN policy (Cloudflare WARP), media disposal via macOS EACS + MDM remote wipe with step-by-step disposal procedure. CC6.6 external threat boundary: Cloudflare WAF + DDoS, CORS `form.coach`-only, tenant ID injection at API gateway (DEC-030 session variable), API key 90-day rotation SLA, zero raw secrets in VCS (`git-secrets` pre-commit). CC6.7 information transmission controls: all API responses RLS-scoped; audit log bulk export requires admin role + signed URL (1h expiry) + approval event; S3 bucket policy JSON (PRE-26-E-003); privacy floor transmission table (HR never receives individual data — data-layer, not config). CC6.8 malicious software: Dependabot + `npm audit --audit-level=critical` CI gate; CSP header design (`default-src 'self'`; Connect restricted to Supabase); ESLint `no-eval` rule; Cloudflare Page Shield (P2 upgrade path). Ten new DEC-030 audit events: `access.mfa_enabled`, `access.mfa_disabled`, `access.mfa_recovery_initiated`, `access.session_revoked`, `access.break_glass_initiated`, `access.break_glass_expired`, `access.bulk_export_approved`, `access.device_registered`, `access.device_wiped`, `access.api_key_rotated` — all HMAC-chained per DEC-030. Eight evidence artefacts PRE-26-E-001 through PRE-26-E-008. Implementation checklist: 14 items (6 P0, 5 P1, 3 P2). Gap closures: MFA enforcement 🟡 Gap → 🟡 Partial (→🟢 on impl); separation of duties 🟡 Partial → compensating controls formally documented; media disposal 🔴 Gap → 🟡 Partial; offboarding 🔴 Gap → 🟡 Partial; dependency scanning CC6.8 🟡 Gap → 🟡 Partial; external boundary CC6.6 🟡 Gap → 🟡 Partial. **Readiness: ~75% → ~77%**.

## [0.84.0] — 2026-05-19

### Added
- `content/post-53-programming-for-busy.md` — **Програмування для занятого: MEV-режим, 3-денний тиждень і нестабільний розклад** (sports-scientist pending, clinical-safety not-required). MEV/MAV/MRV рамка (Israetel): MEV як підлога прогресу, не компроміс. Структурна крихкість сплітів при нестабільному розкладі vs structural resilience full body. 3-денна структура з ротаційними акцентами: таблиця А/B/С з сетами на м'яз. Schoenfeld et al. (2016): частота перевершує при рівному об'ємі — ключова умовна залежність. Ralston et al. (2017): Cohen's d 0.25 — незначущий ефект частоти при контрольованому об'ємі. Режими «прогрес» vs «обслуговування»: Bickel et al. (2011) — 1/3 об'єму зберігає гіпертрофію 8 тижнів при збереженні інтенсивності. Krieger (2010): 2–3 якісні сети при RIR 1–2 > 5 неякісних при RIR 5+. Правило скасування (не перенесення) пропущеної сесії. FORM Victor: оперативний контекст між сесіями — корекція очікуваної форми за днями між тренуваннями. ~2,200 words.
- `blog.html` — картки post-51 (Дихання і IAP), post-52 (Типи волокон), post-53 (Програмування для занятого) додані до grid.
- `STATUS.md` — рядки post-51, post-52, post-53 додані до content engine таблиці.
- `README.md` — roadmap оновлено: post-51 [x] (breathing/IAP, виправлено з плановою темою), post-52 [x] (muscle fiber types, виправлено), post-53 [x] (programming for busy), post-54–57 додані як наступні теми.

## [0.83.0] — 2026-05-19

### Added
- `content/post-51-breathing-iap-strength.md` — **Дихання і внутрішньочеревний тиск у силовому тренінгу: механіка bracing** (sports-scientist pending, clinical-safety ✅ Conditional Approved). Фізіологія IAP: діафрагма + поперечний м'яз живота + тазове дно = закрита порожнина під тиском; McGill & Cholewicki (2001) — IAP як значущий чинник стабілізації поперекового відділу; Harman et al. (1988) — IAP 150–200 mmHg при максимальних присіданнях. Маневр Вальсальви (VM): технічна анатомія (bracing vs full VM); чотири гемодинамічні фази VM; відмінність між патологічним VM і силовим VM тривалістю ≤3с у здорових атлетів (Monroe et al. 2014). 360° bracing vs abdominal hollowing: Grenier & McGill (2007) — bracing перевершує hollowing за стабілізацією хребта у всіх умовах. Пояс: Zink et al. (2001) — ефективний тільки при активному брейсингу. Протоколи дихання за інтенсивністю: VM для >85% 1RM, варіанти A/B для гіпертрофійних сетів (6–12 reps). Розвиток брейсингу як навичка: dead bug, Pallof press, farmers walk. Між підходами: фізіологічне зітхання для ВНС-відновлення. Обов'язковий медичний disclaimer на початку статті (гіпертонія, ССЗ, глаукома, вагітність). ~2,200 words.
- `content/post-52-muscle-fiber-types.md` — **Типи м'язових волокон і тренінг: що ти можеш і чого не можеш змінити** (sports-scientist pending, clinical-safety ✅ Approved). MHC-ізоформи: Type I (MHC-I, β-MHC) — oxidative, висока стійкість до втоми; Type IIa (MHC-IIa) — hybrid, найбільш пластичний; Type IIx (MHC-IIx) — glycolytic, максимальна пікова сила. Генетичний компонент: Komi & Karlsson (1979), Simoneau & Bouchard (1995) — ~45% генетика / ~45% тренінг; детерміністичний фрейм відкинуто на користь описового. Henneman's Size Principle (1957): рекрутування у порядку розміру; наслідки для rep-range і рекрутування Type IIx. IIx → IIa зміщення при силовому тренінгу; detrained individuals мають більший IIx-відсоток. Гіпертрофія Type I: Mitchell et al. (2012) — 30% vs 80% 1RM до відмови — подібна загальна гіпертрофія, але різний розподіл між типами. «Rep max test» при 80% 1RM як практичний польовий інструмент (з обмовками). Schoenfeld et al. (2017) — важкі vs легкі протоколи при рівному об'ємі. Програмування за діапазонами: 1–5 (нейром'язова адаптація, IIx), 6–12 (IIa hybrid), 15–30 до відмови (Type I + метаболічний стрес). FORM velocity tracking і RPE-каліброва як proxy для функціонального fiber-type профілю. ~2,000 words.

### Changed
- `docs/SOC2_READINESS.md` v1.4 → v1.5 — **§25 CC7 System Monitoring & Anomaly Detection** (compliance-officer + security-engineer). Повне CC7.1–CC7.5 criteria mapping з FORM-специфічними реалізаціями. П'ять шарів моніторингу: Cloudflare WAF (auth rate limits + threat rules), Supabase Auth webhook (`auth-monitor` Edge Function), HMAC audit chain daily check (`audit-chain-daily-check` cron 06:00 UTC), Better Stack (uptime + log anomalies), Sentry (error rates + health data leak detection). Auth brute force: WAF правила `FORM-AUTH-RATELIMIT-001/002` (5 req/60s per IP+JA3); `auth-monitor` anomaly query (>5 failed logins per user/60хв; >20 distinct targets per IP/30хв). `form-alert-relay` Cloudflare Worker — єдина точка маршрутизації алертів (Cloudflare → Slack `#security-alerts` + PagerDuty за severity). 10 нових DEC-030 audit events: `auth.login_failed`, `auth.brute_force_detected`, `auth.account_enumeration_detected`, `auth.login_from_new_country`, `system.row_count_anomaly_detected`, `system.audit_chain_verified`, `system.audit_chain_break_detected`, `system.audit_chain_check_failed`, `system.audit_chain_reanchored`, `system.security_alert_fired`. `row-count-monitor` Edge Function (15-хв cron, >5%/>20% deviation thresholds). R-05 runbook (Audit Log Chain Break) специфіковано для `docs/INCIDENT_RESPONSE.md`. Sentry alert rules: `FORM-FATAL-001`, `FORM-SPIKE-001`, `FORM-AUTH-ERR-001`, `FORM-HEALTH-LEAK-001` — з health-data field leak detection і PagerDuty routing. Evidence artifacts PRE-25-E-001 through PRE-25-E-004. Implementation checklist: 20 tasks (13 P0, 4 P1, 3 P2). Gap closure: "Anomaly alerting" 🟡 Gap → 🟡 Partial (design complete); "Continuous monitoring" + "System health monitoring" — architecture fully specified. **Readiness: ~73% → ~75%**.

## [0.82.1] — 2026-05-19

### Changed
- `docs/INCIDENT_RESPONSE.md` v0.1 → v0.2 — **R-07 Sub-processor Breach Notification + §11 Sub-processor Incident Management** (security-engineer + compliance-officer + enterprise-architect). Closes the gap where INCIDENT_RESPONSE.md had no protocol for the scenario where Anthropic, Supabase, ElevenLabs, or Cloudflare notifies FORM of a breach on their end — the most operationally ambiguous P0 scenario because FORM cannot contain the breach at source and must derive its Art. 33 assessment from second-hand information. **R-07 runbook:** trigger conditions with per-processor severity floors (Supabase/Anthropic → always P0; ElevenLabs/Cloudflare → P1+; PostHog/Sentry → P2 with audit caveats); immediate actions including GDPR Art. 33 clock start documentation; scope assessment checklist (data categories, enterprise tenant overlap, Art. 9 determination); GDPR 72-hour clock management under sub-processor context — key point: clock starts at FORM's awareness of notification, not at processor's discovery; Art. 33 partial filing target at T+48h maximum; enterprise tenant notification (phone within 4h, DPA SLA check); full evidence package spec; post-incident vendor management review triggering formal written inquiry on SLA misses. **§11 framework:** §11.1 Purpose (why sub-processor incidents are operationally distinct from R-01–R-06 — FORM cannot control investigation pace); §11.2 Sub-processor Registry with data categories, Art. 9 classification, DPA status, notification contacts, and severity floors for 9 processors; documented gaps — Anthropic DPA notification SLA not enumerated (risk accepted, renewal target), ElevenLabs DPA unsigned before production (P1 action item), Sentry scrubbing unverified; annual review checklist with audit_log entry spec. §11.3 Art. 28(3)(f) contractual requirements and DPA signing checklist — notification SLA, cooperation scope, sub-processor restrictions, data deletion obligations. §11.4 Processor notification SLA tracker (live table updated per incident). §11.5 SOC 2 CC9.2 evidence integration — evidence types, storage locations, auditor package spec. Appendix A updated: R-07 quick reference line, sub-processor GDPR clock context note, §11.2 registry reference.

## [0.82.0] — 2026-05-19

### Added
- `content/post-50-proprioception-balance.md` — **Пропріоцепція і баланс у силовому тренінгу: навіщо це intermediate-атлету** (sports-scientist pending, clinical-safety not-required). М'язові веретена (Ia/II аферентні волокна), апарат Гольджі (сучасний перегляд: тонка регуляція зусилля, не тільки захисне гальмування), суглобові механорецептори і їх специфічність до крайніх кутів. Joint position sense у тренованих vs нетренованих (Fidler et al. 2022). Нестабільні поверхні: Anderson & Behm (2005) — 60% втрата сили без підвищення активації стабілізаторів; Behm & Colado (2012) систематичний огляд — незначний benefit для сили у тренованих. Pogіршення JPS при м'язовій втомі (Strojnik et al. 2000) як механізм технічної відмови у важких сетах. Де пропріоцепція реально тренується: унілатеральна робота (Bulgarian split squat, single-leg RDL), темпові повторення, важкий базовий тренінг сам по собі. FORM CV як external recalibration loop — доповнення, а не заміна внутрішнього сигналу. ~1,900 words.
- `blog.html` — картки post-49 (Rate of Force Development) і post-50 (Пропріоцепція і баланс) додані до grid.
- `STATUS.md` — рядки post-49 і post-50 додані до content engine таблиці.
- `README.md` — roadmap оновлено: post-49 [x] (RFD, виправлено опис), post-50 [x] (proprioception), post-51 і post-52 додані як наступні теми.

## [0.81.0] — 2026-05-19

### Added
- `docs/SOC2_READINESS.md` v1.3 → v1.4 — **§24 Cookie Consent & Consent Management Platform** (compliance-officer + security-engineer). Closes PRE-16 design phase (🔴 Open → 🟡 Partial; **critical gaps: 1 → 0** at design level). Full cookie & tracker inventory: 7 items (5 necessary: Supabase auth, PKCE verifier, Stripe mid/sid, Cloudflare; 2 analytics: PostHog ph_*, ph_opt_in_out_*). CMP selection: Cookiebot (Usercentrics) Essential — CNIL + ICO certified, Cloudflare Zaraz native integration, consent log API for SOC 2 evidence chain, €14/month. Technical spec: Zaraz consent gate with PostHog deferred until `consent.statistics = true` callback; banner config requirements (default-deny, equal button prominence, granular categories). Mobile consent architecture: ATT (`requestTrackingAuthorization()`) for iOS, in-app ConsentScreen (analytics toggle OFF by default, `posthog.optIn/Out()`) for GDPR EU users on both platforms — deferred to mobile phase, documented in MOBILE_ROADMAP.md. HMAC audit chain: 6 new DEC-030 events (`privacy.consent_granted`, `consent_declined`, `consent_withdrawn`, `consent_updated`, `att_consent_granted`, `att_consent_denied`) — satisfies GDPR Art. 7(1) record-keeping without storing plain-text consent receipts. Enterprise B2B: individual consent required despite SCIM provisioning; `analytics_consent_default: false` tenant-level toggle. Evidence package PRE-16-E-001 through PRE-16-E-005. Implementation checklist: 11 items (6× P0, 3× P1, 2× P2). Control updates: Cookie banner/CMP 🔴→🟡 Partial; P2.1 🔴→🟡 Partial; Privacy Policy cookies section 🔴→🟡 Partial. Readiness: **~71% → ~73%**.
- `content/post-49-rate-of-force-development.md` — **Rate of Force Development: The Training Quality Most Athletes Ignore** (sports-scientist, clinical-safety ✅ Pass — pure neuromuscular science). RFD defined (N/s, slope of force-time curve); time constraint analysis: CMJ 200–250ms, striking <100ms, squat propulsive 400–600ms. Force plate measurement: 0–50ms (neural, early RFD), 0–100ms, 0–200ms (CSA-dependent, late RFD); bar acceleration as field proxy. Why hypertrophy training doesn't maximize RFD: neural drivers (motor unit recruitment rate, rate coding, high-frequency doublets at 80–120Hz in first 5–10ms, synchronization). Four training methods: (1) maximal intent at 55–70% 1RM (4–6 sets × 3–4 reps, 3+ min rest); (2) ballistic/plyometric (SSC elastic energy, PAP cross-reference post-43); (3) contrast training (85–95% compound → 30–40% ballistic, peaking block only); (4) explosive isometrics (±15–20° angle specificity). Programming: early in session and week, low volume (accumulation 10–15 reps, peaking 6–10 reps), stop on velocity decline. FORM tracking: within-session >20–25% velocity loss = acute fatigue; cross-session velocity decline at identical load = chronic indicator. Cite: Aagaard et al. (2002) JAP; Tillin & Bishop (2009) Sports Medicine; Suchomel et al. (2016) Sports Medicine. ~2,070 words.

## [0.80.1] — 2026-05-19

### Added
- `docs/SOC2_READINESS.md` v1.2 → v1.3 — **§23 Quarterly Access Review Procedure** (compliance-officer + security-engineer + enterprise-architect). Closes PRE-23 design phase (🔴 Open → 🟡 Partial). Full scope definition: eleven production systems (GitHub, Cloudflare, Supabase, 1Password, PostHog, Sentry, Stripe, ElevenLabs, Anthropic, Apple Developer, Google Play) plus service-account / API-token inventory and enterprise tenant admin role sweep. Six-step procedure: (1) enumerate active access, (2) compare against authorized roster, (3) deprovision within 24h SLA, (4) enterprise tenant inactive-account sweep with 5-day tenant-admin acknowledgment gate, (5) CC4.2 control effectiveness table for six highest-risk CC6 controls, (6) sign-off and SHA-256 artifact filing into HMAC audit chain (`system.access_review_completed` DEC-030 event). Authorized role roster table for solo-founder phase plus five authorized service accounts with rotation SLAs. Evidence artifact template (`access-review-YYYY-QN.md`) for private compliance repository. Solo-founder compensating control: git commit timestamp + HMAC chain publication replaces independence requirement; expires at first hire. Privacy floor enforcement: reviewer scope limited to role/permission metadata — individual health data queries prohibited. Three new gap items (CC6-GAP-001 through CC6-GAP-003). Control table updates: CC4.2 "Quarterly control effectiveness review" 🔴 → 🟡 Partial; CC6.3 "Quarterly access review" new row added 🟡 Partial. Gap Analysis Summary updated: critical gaps 2 → 1 (cookie consent sole remaining blocker); controls in-place 28 → 31; partial 30 → 33; readiness ~69% → ~71%.

## [0.80.0] — 2026-05-19

### Added
- `content/post-48-training-volume-adaptation.md` — **Тренувальний об'єм: коли більше стає гіршим** (sports-scientist, clinical-safety not-required). SRA-крива як фундамент управління об'ємом. MEV/MAV/MRV фреймворк (Israetel et al.) з типовими значеннями для великих/середніх/малих груп і чинниками, що зміщують всі числа (тренувальний вік, сон, загальний стрес, харчовий баланс). Функціональне vs нефункціональне надперевантаження: чіткі ознаки кожного, часові рамки відновлення. Хвиляста надорганізація — шеститижнева таблиця (MEV+ → MAV → функціональний overreach → деавантаж). Мінімальна ефективна доза: Schoenfeld et al. (2017) — 5–10 сетів/м'яз/тиждень дають значущу гіпертрофію; Krieger (2010) — 20–30% перевага вищого об'єму при вдвічі більшій кількості. Сигнали наближення і перевищення MRV. Як FORM управляє об'ємом динамічно через швидкість прогресу, суб'єктивне відновлення і пульс спокою. Clinical-safety PASS (чиста тренувальна фізіологія). ~1,400 words.
- `blog.html` — картки post-47 (concurrent training) і post-48 (training volume) додані до grid.
- `STATUS.md` — рядки post-47 і post-48 додані до content engine таблиці.
- `README.md` — roadmap оновлено: post-47 [x] з правильним топіком (concurrent training), post-48 [x], post-49 (proprioception) і post-50 (busy scheduling) додані як наступні.

## [0.79.0] — 2026-05-19

### Added
- `docs/SOC2_READINESS.md` v1.1 → v1.2 — **§22 Security Awareness & Training Program** (compliance-officer + security-engineer). Closes CC1.4 (commitment to competence) and CC2.2 (internal communication) gaps. Eight-topic required curriculum (OWASP Top 10, phishing, data classification, incident response, credential hygiene, secure code review, privacy/GDPR, vendor risk awareness) with SOC 2 criteria mapping per topic. Solo-founder annual training plan: four specific courses (OWASP WebGoat, Cloudflare Security track, Supabase security docs, GDPR self-assessment) with evidence artefact naming convention. New hire security onboarding checklist: eight sequential steps, MFA hard gate at step 3 before credentials granted. Annual refresher cadence: Q1 February (all topics), Q3 August (OWASP + CVE review), ad-hoc post-incident (14-day SLA). Phishing simulation programme: post-hire, quarterly, KnowBe4/GoPhish, no-punitive-action failure protocol, four outcome tiers. Evidence package: CC1-E-001 through CC1-E-005. Four new gap items (CC1-GAP-001 through CC1-GAP-004): AUP document, founder annual training execution, phishing platform procurement, background check provider. Gap closure: "Security awareness programme" 🟡→🟢 Done; "New hire onboarding" 🔴→🟢 Done; "Annual refresher" 🟡→🟢 Done; "Phishing simulation" 🔴→🟡 Partial. Readiness: ~67% → **~69%**.
- `content/post-47-concurrent-training.md` — **Кардіо вбиває силу: що насправді каже наука про concurrent training** (sports-scientist). Hickson (1980) interference experiment: 6+6 protocol, 7-week inflection point where concurrent group stalled. Molecular mechanism: AMPK activation by endurance suppresses mTORC1 — signal is transient and time-dependent. Interference hypothesis with three proposed mechanisms (molecular conflict, chronic fatigue, motor competition) and explicit boundary conditions. Modern evidence: Wilson et al. (2012) meta-analysis, Murach & Bagley (2016) — interference is volume-dependent not inevitable. Key modifiers: exercise order (strength before cardio), session separation (6h minimum same-day, 24h ideal), modality (cycling vs. running and lower-body hypertrophy), cardio volume as primary driver. Practical 4-strength + 2-cardio weekly structure table. FORM differentiators: modality selection per training block, volume modulation by phase, order enforced by molecular logic. Clinical-safety PASS (pure training physiology, no nutrition/body-image content). ~1,350 words.

## [0.78.1] — 2026-05-19

### Added
- `docs/SSO_SCIM_IMPLEMENTATION.md` v0.5 → v0.6 — **§14 SCIM Data Processing: GDPR Article 28 Compliance Framework** (compliance-officer + enterprise-architect + security-engineer). Closes G-013 design phase (previously 🔴 hard block "do not enable SCIM for any customer until DPA is updated" → 🟡 design complete, outside counsel review pending). Personal data inventory for all SCIM attribute classes (identity, account state, organisational, reporting structure, groups, sync log, audit trail) with retention periods and controller legal basis chain. DPA Annex B template clause language ready for outside counsel: subject-matter, duration, nature/purpose, types of personal data, categories of data subjects, controller obligations (Art. 14 transparency notice duty, SCIM token rotation, IdP schema hygiene), sub-processor disclosure (no new processors), and 6-path Art. 15–20 data subject rights assistance matrix. Privacy floor enforcement: `users.scim_attributes` excluded from `tenant_manager` RLS and all HR-tier API responses; role mapping opacity (admin sees resulting role, not source group attribute); k-anonymity floor of 5 applies to SCIM-dimension aggregate reports. Data minimisation: unknown core attributes silently dropped (RFC 7644 §3.3); enterprise extension allowlist (5 approved attributes); Art. 9 special category attribute scanner with `BLOCKED_ATTRIBUTE_PATTERNS` blocklist → `HTTP 400` + `scim.rejected_sensitive_attribute` audit event on match. Art. 17 dual-path erasure: Path A (employer SCIM DELETE/active=false — access revocation, 30-day soft delete, then anonymisation) vs. Path B (user-initiated GDPR erasure — health data deleted, tenant admin notified via webhook, HMAC chain preserved via anonymised `actor_id` hash in separate `gdpr_erasure_log` table). 10 new audit events added to DEC-030 taxonomy (`scim.user.provisioned`, `scim.user.updated`, `scim.user.deprovisioned`, `scim.group.created`, `scim.group.updated`, `scim.group.deleted`, `scim.rejected_sensitive_attribute`, `scim.token.rotated`, `scim.token.revoked`, `system.gdpr_erasure_request`). Sub-processor disclosure: no new processors; PostHog `distinctId` must be FORM UUID, never email, for SCIM-provisioned users — constraint documented. EU residency: Supabase `eu-central-1` for EU tenants; SCC Module 2 (controller-to-processor) for US-region tenants with EU employees. 12-item implementation checklist (P0/P1/P2 priority). G-013: 🔴 → 🟡 Partial. SOC 2 CC6.1 and CC6.2 impact documented.

## [0.78.0] — 2026-05-19

### Added
- `content/post-46-hormones-training.md` — Гормони і тренінг: тестостерон, кортизол, GH. Гостра реакція (15–25% T-підвищення під час тренування — реальне, але короткочасне і слабко прогнозує гіпертрофію) проти хронічної адаптації. Чому один аналіз крові нічого не доводить: добова варіація T до 35%, вільний vs загальний (SHBG). Що знижує базовий T: недосип (1 тиждень 5h = -10–15%, Leproult & Van Cauter 2011), хронічне RED-S, а не помірне коригування маси. Кортизол: гострий пік необхідний (глюконеогенез, мобілізація ЖК) — проблема тільки при хронічному стані. T:C ratio — ненадійна одиночна метрика. GH: пульсаційна секреція, нічний NREM-пік; IGF-1 як справжній медіатор анаболізму; пероральні «GH-релізери» руйнуються в ШКТ. «Тестостеронові бустери»: цинк/D3 коригують дефіцити, ашваганда — скромний ефект при стресі (5–15%, Nasimi Doost Azgomi 2019), D-аспарагінова — немає ефекту у тренованих, трибулус/мака — слабка база. Сон — єдиний нефармакологічний важіль із реальним ефект-розміром. Physician note додана. Clinical-safety: CONDITIONAL PASS (severe/prolonged deficit framing; no dosing guidance; no body-comp targets). Sports-scientist review pending. 14 хв читання.
- `blog.html` — картка post-46 (Гормони і тренінг) додана до articles index
- `README.md` — post-46 позначений [x]; pipeline: post-47 (пропріоцепція), post-48 (адаптація до об'єму)
- `STATUS.md` — post-46 у content engine таблиці

## [0.77.0] — 2026-05-19

### Added
- `docs/SOC2_READINESS.md` v1.0 → v1.1 — **§21 CC8 Change Management Controls: Formal Policy & Evidence Framework** (compliance-officer + security-engineer + devops-lead). Three-tier change classification: Emergency (P0, 2h deploy SLA, single-approver + mandatory post-hoc review + simultaneous incident ticket), Standard (PR + written rationale, 24–72h SLA), Minor (PR + auto-deploy). Branch protection spec for `main`: 7 exact settings with current/target/gap table (CC8-E-001). CI/CD gap (CC8-GAP-001) formally documented — compensating control: manual pre-deploy checklist in ENGINEERING_RUNBOOK.md. Database schema change controls: 3 hard rules (migrations only, always Standard authorization, mandatory down-migration), 5-item migration review checklist (RLS, tenant_id FK, index strategy, backward compat, down-migration test), rollback paths (down-migration <15 min, PITR <4h). Rollback procedures: Cloudflare Workers (`wrangler rollback`, <5 min), Pages (permanent history, <5 min), Supabase schema (<15 min / <4h), ML model KV pointer (<5 min). Separation of duties: structural gap documented without softening — 4 compensating controls (PR-always, written rationale, test gate pending, HMAC audit log), Management Assertion Letter disclosure guidance, post-hire CODEOWNERS path. Admin-override emergency exception: `admin-override` GitHub Issue within 30 min + 24h post-hoc review (CC8-E-005). Evidence package: 6 artifacts (CC8-E-001 through CC8-E-006). Control status: PR review policy 🆕 🟢; Rollback procedure 🟡 → 🟢; Emergency change process 🟡 → 🟢; Separation of duties 🔴 → 🟡. Critical gaps: 2 → 1. Readiness: ~65% → ~67%.

## [0.76.0] — 2026-05-19

### Added
- `content/post-45-cns-vs-peripheral-fatigue.md` — ЦНС-втома проти периферійної: два механізми з різною кінетикою відновлення і різними наслідками для програмування. Периферійна: субстратне виснаження (PCr 30-180s, глікоген 24-48h), Pi і H⁺ накопичення (pH-опосередкована інгібіція актин-міозин містка), кальцієва трансієнтність. Центральна: падіння central drive (interpolated twitch technique), серотонін/дофамін нейротрансмітерний зсув, захисне гальмування. Порівняльна таблиця симптомів. Програмні наслідки: обсяг vs інтенсивність різно накопичують втому, ATF ≠ тренування до RIR-1-3 по ЦНС-вартості, нейронно-насичені вправи вищий центральний привід. Деавантаж: периферійна → знижуй обсяг; центральна → знижуй інтенсивність + обсяг. HRV як інтегральний проксі (не прямий ЦНС-маркер). Що НЕ є ЦНС-втомою: демотивація, кофеїновий відкат, DOMS. Clinical-safety не вимагається. Sports-scientist review pending. 14 хв читання.
- `blog.html` — картка post-45 (ЦНС-втома проти периферійної) додана до articles index
- `README.md` — post-45 позначений [x]; pipeline зсунутий (post-46 = гормони, post-47 = пропріоцепція, post-48 = адаптація до об'єму)
- `STATUS.md` — post-45 у content engine таблиці

## [0.75.0] — 2026-05-19

### Added
- `docs/SSO_SCIM_IMPLEMENTATION.md` v0.4 → v0.5 — **§13 Federated Logout: SAML SLO & OIDC Back-Channel Logout** (enterprise-architect + security-engineer). Closes G-002 design phase (SAML 2.0 Single Logout): SP-initiated SLO flow (LogoutRequest → IdP propagation → LogoutResponse, HTTP-Redirect binding with deflate+base64+URL-encode, RS256 signature), IdP-initiated SLO (unsolicited LogoutRequest to `/auth/saml/slo`, NameID+SessionIndex lookup), 6-state SLO lifecycle (Idle → LogoutInitiated → WaitingIdPResponse → PropagatingToOtherSPs → SessionsRevoked → Complete/Failed), 4 failure modes with graceful degradation (local session cleared regardless of IdP response — never block local logout on partner SP timeout). Closes G-003 design phase (OIDC Back-Channel Logout): `back_channel_logout_uri` registration at `https://api.form.coach/auth/oidc/backchannel-logout`, logout_token JWT validation (iss, aud, iat within 5 min, nonce MUST NOT be present, events claim required), session revocation via `enterprise_sessions.idp_session_id`, 200 OK synchronous response with 2s revocation SLA. Front-channel logout explicitly rejected (4 reasons: SameSite=Strict blocks cross-origin, session state URL leak, third-party cookie deprecation, back-channel strictly superior). Session revocation mechanics: `revocation_reason='federated_logout'` unified with SCIM/timeout paths (§12), JTI blocklist interaction documented. SAML NameID format handling: persistent (required), transient fallback, email-format fallback chain. New `idp_session_id` column requirement on `enterprise_sessions` — prerequisite for both SLO and back-channel. 8 new audit events (`slo.sp_initiated`, `slo.idp_initiated`, `slo.completed`, `slo.failed`, `slo.fallback_local_only`, `backchannel_logout.received`, `backchannel_logout.validated`, `backchannel_logout.revoked`). 8-item implementation sequencing with dependencies. G-002: 🔴 → 🟡 Partial (design complete, implementation pending). G-003: 🔴 → 🟡 Partial. SOC 2 CC6.1 partial closure.

## [0.74.0] — 2026-05-19

### Added
- `docs/SOC2_READINESS.md` v0.9 → v1.0 — **§20 Status Page Architecture & Availability Communication** (compliance-officer + security-engineer + enterprise-architect). Architecture decision: Better Stack Statuspage with CNAME `status.form.coach`; rejected Atlassian Statuspage (cost/lock-in) and custom Worker (eng burden). 6 monitored components with check intervals and SLO targets: API (30 s, 99.9%), Auth synthetic probe (60 s, 99.9%), Realtime WebSocket (60 s, 99.5%), CV health (120 s, 99.5%), Admin dashboard (60 s, 99.9%), Audit log delivery metric (5 min, P95 < 5 s). 5 incident states (Operational / Degraded / Partial Outage / Major Outage / Under Maintenance) with auto-trigger rules and human confirmation SLAs (P0: human update ≤5 min; P1: ≤15 min; P2: ≤1 h). Status update protocol linked to `INCIDENT_RESPONSE.md §6` CC-01 template. Subscriber notifications: email (per-component subscription, `status@form.coach`), RSS/Atom feed, enterprise Slack webhook via `form-status-relay` Worker fan-out with delivery audit log. R2 archival cron (monthly, 7-year retention, HMAC-SHA256 signed) for SOC 2 evidence. Sub-processor list publication: `security.form.coach/sub-processors` Worker (HTML + JSON, change notifications 30 days advance). SOC 2 evidence mapping: A1.1, A1.2, CC7.4, CC6.8. 17-item implementation checklist (PRE-10 and PRE-11 gated on 30/90-day runtime). Gap closures: "Status page" 🔴 → 🟡 Partial; "Uptime monitoring" 🔴 → 🟡 Partial; "Sub-processor list published" 🔴 → 🟡 Partial. Critical gaps: 3 → 2. Readiness: ~63% → ~65%.

## [0.73.0] — 2026-05-18

### Added
- `content/post-44-heat-cold-recovery.md` — Heat і cold exposure у відновленні: CWI блокує mTORC1 і знижує гіпертрофійний сигнал якщо застосована одразу після силового (Roberts et al., 2015: знижена mTORC1 і p70S6K активність, менша сателітна клітинна активність). Сауна нейтральна для mTOR, HSP70-адаптація, GH-пік, серцево-судинні дані Laukkanen et al. (2315 учасників, 20.7 років). Contrast therapy: Cochrane review (Versey 2013) — перевага над пасивним відпочинком, але не над CWI. Тайминг-таблиця (0-30 хв / 4-6 год / 24+ год після силового). Практичний протокол (температури, тривалість, частота). Розбір 5 хайп-тверджень. Застереження для серцево-судинних патологій (ACSM standard). 13 хв читання. clinical-safety PASS.
- `blog.html` — картки post-42, post-43, post-44 додані до articles index
- `STATUS.md` — рядки post-42/43/44 у content engine таблиці
- `README.md` — roadmap: post-42/43/44 позначені [x]; pipeline розширено до post-47

## [0.72.0] — 2026-05-18

### Added
- `docs/SSO_SCIM_IMPLEMENTATION.md` v0.3 → v0.4 — **§12 Session Token Lifecycle & Refresh Management** (enterprise-architect + security-engineer). Dual-token model: RS256 JWT access token (15 min, memory-only, no localStorage) + opaque UUID refresh token (httpOnly Secure SameSite=Lax cookie, SHA-256 hashed in DB). `enterprise_sessions` table schema with `family_id` + `generation` columns enabling token family attack detection. Token reuse detection: stale-token lookup → bulk `family_id` revoke with `revoked_reason='token_reuse_detected'`, 30-second clock-skew grace for parallel requests. Enterprise session timeout config via `tenant_sso_configs.session_policy` JSONB (`session_timeout_hours` 8–720h, `idle_timeout_minutes`, `max_concurrent_sessions`, `reauth_on_sensitive_ops`). SCIM deprovisioning → synchronous session revocation in same transaction; 15-min access token residual documented; opt-in JTI Redis blocklist for zero-latency deprovisioning requirement. SSO re-auth for sensitive ops (OIDC `prompt=login`, SAML `ForceAuthn=true`, signed re-auth assertion JWT, 5-min TTL). 13 audit events (security-critical: `session.token_reuse_detected`). Cloudflare Worker middleware boundary vs Supabase Auth scope. RS256 monthly key rotation + emergency path. Per-tenant migration flag with 48h advance notice pattern. 9 implementation dependencies sequenced.

## [0.71.0] — 2026-05-18

### Added
- `content/post-42-velocity-based-training.md` — Velocity-Based Training: load-velocity профіль і зони MCV (0.15–1.0+ м/с), VL% як критерій зупинки сету (10–15% сила / 20–25% гіпертрофія / 30–40% силова витривалість), протокол побудови власного профілю (6 вправ × 3–5 повт від 50% до ~90%). Обладнання від GymAware ($2800) до Coach's Eye (безкоштовно). Практичний протокол без пристроїв. Застереження про специфічність вправи і атлета. Де VBT не підходить. ~14 хв читання. clinical-safety PASS.
- `content/post-43-post-activation-potentiation.md` — PAP vs PAPE: молекулярний механізм (MLC-фосфорилювання на Ser19/Thr18, persistent inward currents, пеннаційний кут) vs спостережуваний приріст результату. PAP-втома парадокс: чому оптимальне вікно 4-8 хв (не 30 сек, не 15 хв). Три комплексні протоколи з числами (присідання 85-93% → CMJ, жим 85% → медбол, RDL → фермерська хода). Варіабельність відповіді між атлетами. Чіткі протипоказання (гіпертрофійна фаза, початківці, недовідновлення). 9 хв читання. clinical-safety PASS.

## [0.70.1] — 2026-05-18

### Added
- `docs/SOC2_READINESS.md` v0.8 → v0.9 — **§19 Cold Storage Backup — Architecture, Implementation & SOC 2 Evidence** (compliance-officer + security-engineer + devops-lead). Closes the sole remaining 🔴 critical gap ("cold storage backup") documented in §18.6. Three-tier backup architecture (Supabase PITR / Cloudflare R2 90-day rolling / Backblaze B2 7-year WORM): nightly logical dump Worker spec with pg_dump + gzip + SHA-256 manifest, R2 Terraform lifecycle rule; monthly cold-archive Worker spec with AES-256-GCM client-side encryption, per-month key rotation, B2 Object Lock WORM. Integrity verification: monthly spot-check procedure + annual full restore test mapped to SOC 2 A1.3 evidence artifact. Tenant data isolation constraints: dual-custody access model for B2, HMAC chain re-anchoring on restore (DEC-030). Privacy constraints: health-adjacent field anonymisation required on any non-production restore. SOC 2 control closure table (A1.2, A1.3, C1.2, CC7.5). Gap closure: cold storage backup 🔴 → 🟡 Partial. Critical gaps: 1 → 0 (upon implementation). Readiness: ~60% → ~63%. 16-item implementation checklist for devops-lead (M4 enterprise launch gate).

## [0.70.0] — 2026-05-18

### Added
- `content/post-41-grip-strength.md` — Хват як обмежувальний фактор: анатомія відмови forearms (FDS/FDP, ішемічний механізм при >30–40% MVC), три типи хвату (crush/pinch/support) з поясненням відсутності трансферу, чесний розбір стропів без dogma, dead hang / plate pinch тест базового рівня, протокол 3×/тиждень наприкінці сесій, дерево рішень. 13 хв читання. clinical-safety PASS.
- `STATUS.md` — рядок post-41 у content engine таблиці
- `README.md` — post-41 позначено [x] у roadmap

## [0.69.0] — 2026-05-18

### Added
- `content/post-40-hypertrophy-mechanisms.md` — Механічна напруга, метаболічний стрес і пошкодження м'язів: модель Шенфельда 2010 у 2026; stretch-mediated hypertrophy, тітін-сигналінг, перегляд ролі метаболічного стресу і EIMD. clinical-safety PASS.
- `blog.html` — картка post-40 додана до article index; placeholder-картка post-41 (grip strength) готова
- `STATUS.md` — content engine таблиця оновлена (post-40)
- `README.md` — roadmap оновлено: post-40 позначено [x]; розширено pipeline post-42–45

## [0.68.0] — 2026-05-18

### Added
- `docs/DATA_MODEL.md` v0.3 → v0.4 — **§12 Soft Delete, Data Retention & GDPR Art. 17 Erasure** (enterprise-architect + compliance-officer)
  - Soft-delete strategy table (14 tables; `audit_log` immutable per DEC-030; `tenant_scim_tokens` uses `revoked_at` not `deleted_at`)
  - Retention windows by data classification (Art. 9 special category, consumer vs. enterprise DPA)
  - Full Art. 17 erasure flow: re-auth gate → soft-delete → async erasure job → users anonymization (not hard-delete; FK integrity reasoning) → confirmation email
  - `pg_cron` schedule: media-upload 90-day purge, 30-day hard-delete of user data, 2-year consumer audit log purge
  - Art. 20 data portability export spec: JSON zip, signed Supabase Storage URL, 72h SLA target, 3-request/30-day rate limit
  - Data minimisation register per Art. 5(1)(c): 10 data points with collection justification or exclusion reason
- `docs/SOC2_READINESS.md` v0.7 → v0.8 — **§18 Business Continuity & Disaster Recovery** (devops-lead + compliance-officer)
  - RTO/RPO commitments: Enterprise 4h/1h, Pro 8h/4h, Auth/SSO 1h/15min
  - 4 failure scenarios with step-by-step response: Supabase unavailability, Cloudflare outage, data corruption (PITR path), nuke scenario (cold start from B2)
  - Annual DR drill procedure (staging-env Scenario C), drill report template, evidence filing spec for CC7.5
  - Communication tree: 5 incident phases × 3 channels (internal, enterprise, consumer)
  - SOC 2 gap closure: CC7.5 🔴 → 🟡 Partial; DR runbook 🟡 → 🟢 Done; readiness ~58% → ~60%
  - Cold storage backup gap newly documented (B2 nightly export not yet implemented)
- `docs/SSO_SCIM_IMPLEMENTATION.md` v0.2 → v0.3 — **§11 Just-in-Time (JIT) Provisioning Design** (enterprise-architect + security-engineer)
  - JIT vs. SCIM decision matrix (4 scenarios); JIT is the default for SSO-enabled, SCIM-disabled tenants
  - Full provisioning flow: domain gate → seat-limit check (transactional FOR UPDATE) → INSERT users → audit log → admin notification
  - SAML and OIDC claim extraction tables with per-field fallback behavior; per-tenant `claim_mapping JSONB` config
  - Domain verification gate: `allowed_domains TEXT[]`, exact-match only (no subdomain passthrough), implementation note
  - Seat limit race-condition fix: `SELECT ... FOR UPDATE` inside transaction prevents concurrent JIT overshooting `max_seats`
  - JIT-to-SCIM reconciliation: 409 Conflict → PATCH adopt path; role not overwritten unless SCIM payload includes `roles`
  - 6 JIT audit events per DEC-030; admin controls (jit_provisioning_enabled, default_role, notify_on_jit_provision)
  - Security note: domain verification is mandatory; JIT without it is a tenant isolation bypass
- `docs/COST_MODEL.md` v0.4 → v0.5 — **§§13-14: Infrastructure Cost Breakdown & Cohort LTV Model** (data-engineer)
  - §13: per-service cost table at 1k/10k/50k/100k MAU (Anthropic, ElevenLabs, Supabase, Cloudflare, Sentry, PostHog, Redis); fixed vs. variable classification; upgrade inflection points; ElevenLabs Creator-plan quota concern flagged (exhausted at ~14 Pro voice users); Anthropic prompt caching lever (up to 90% reduction on cached context); blended per-user cost stabilizes at $0.32–0.34 at scale
  - §14: 24-month cohort survival tables (monthly vs. annual Pro plans; 5% vs. 1.5% monthly churn); M24 GM-adjusted LTV $194 (monthly) and $176 (annual); CAC payback periods ($30→M3, $50→M4, $80→M7, $120→M12); LTV:CAC targets by channel; enterprise LTV model (40-seat min, 120% NRR, 3-year GM-adj $18,660); cohort survival requirements; deferred revenue recognition note
- `content/post-40-hypertrophy-mechanisms.md` — **sports-science: механізми гіпертрофії** (sports-scientist, clinical-safety: PASS)
  - Schoenfeld 2010 three-mechanism framework updated with 2020s evidence
  - Mechanical tension as primary driver; metabolic stress and muscle damage re-examined
  - Stretch-mediated hypertrophy (Pedrosa et al. 2022 cited); practical training implications
  - Victor callout: editorial-brutalist, Ukrainian main body, bilingual key terms

---

## [0.67.1] — 2026-05-18

### Added
- `docs/OBSERVABILITY.md` v0.1 → v0.2 — **§§13-15: per-tenant observability, trace correlation, audit log export pipeline** (devops-lead + platform-engineer + security-engineer)
  - **§13 Per-Tenant Observability Implementation**: tenant context injection pattern in Cloudflare Workers, Analytics Engine data point schema with `tenant_id` as `indexes[0]`, per-tenant SLO breach detection Worker (Standard 99.9% / Premium 99.95% tiers), admin dashboard SLA API schema (`GET /api/admin/tenants/:id/sla`) with privacy floor (no per-user data in SLA reports), weekly isolation audit query (cross-tenant leak detection → P0 incident). Closes M4 gap "Per-tenant SLA reporting".
  - **§14 Cross-Backend Trace Correlation (Sentry ↔ Cloudflare)**: shared W3C `traceparent` as single `trace_id` namespace, Worker implementation (`extractTraceId`, `buildTraceparent`, upstream propagation), `X-Trace-Id` echo header, React Native `apiFetch` wrapper attaching `cf.trace_id` tag to Sentry transactions, Analytics Engine correlation query, Sentry→Cloudflare navigation runbook (6-step), verification checklist. Closes M3 gap "Trace correlation between Sentry and Cloudflare".
  - **§15 Audit Log Export Pipeline**: two-mode delivery architecture (real-time webhook + hourly batch); CloudEvents 1.0 webhook payload with HMAC-SHA256 signature (`X-Form-Signature`), 5-attempt exponential retry policy, delivery failure audit event; hourly NDJSON batch to R2 with optional S3/GCS customer sync; HMAC chain customer verification algorithm (language-agnostic, uses shared onboarding key per DEC-030); privacy constraints (no raw `user_id`, no health content, no prompt/response); delivery SLAs (webhook P95 < 30 s, batch < 5 min post-hour); 10-item implementation checklist for platform-engineer. Closes M4 gap "Audit log export pipeline".
  - `§10` gap table updated: 3 gaps now reference their design sections (§13, §14, §15)

---

## [0.67.0] — 2026-05-18

### Added
- `content/post-39-motor-unit-henneman.md` — рухова одиниця і принцип Хенемана: size principle (рекрутинг від малих мотонейронів Type I до великих Type IIx), рекрутинг vs rate coding як два механізми регуляції сили, таблиця характеристик Type I/IIa/IIx, чому підходи до відмови рекрутують швидкі волокна через примусовий рекрутинг (не тільки інтенсивність), нейральна адаптація передує гіпертрофічній (rate coding + синхронізація перші 4–8 тижнів), обмеження CV-аналізу для нейром'язового профілю (зв'язок з FORM CV pipeline). (sports-scientist · clinical-safety PASS)
- `blog.html` — картки post-38 і post-39
- `STATUS.md` — рядки post-38 і post-39 у content engine table
- `README.md` — post-37–39 відмічені [x], roadmap post-40–44 додано

---

## [0.66.0] — 2026-05-18

### Added
- `docs/SOC2_READINESS.md` v0.6 → v0.7 — **§17 Vendor Security Review Process** (compliance-officer + security-engineer)
  - Closes two documented 🔴 Gaps: "Vendor security review process" and "Annual vendor security review"
  - Three-tier vendor classification (Critical / High / Standard) with proportionate review cadence
  - Vendor Risk Registry: 11 vendors (8 sub-processors + operational tools) with DPA status, cert level, risk score, owner
  - 5-step Initial Vendor Assessment with DPA gate (non-waivable) and compliance-officer approval veto
  - January Annual Review process with SOC 2 CC9.2 evidence package definition
  - 6-factor risk scoring matrix (data sensitivity, cert level, DPA, incident history, data residency, subprocessor chain depth)
  - New sub-processor addition workflow (7 steps + emergency exception clause)
  - Termination & offboarding: 24h credential revocation, written deletion confirmation, 7-year retention
  - SOC 2 control mapping: CC9.1, CC9.2, CC9.3, P8.1
  - Critical gaps: 3 → 1. Readiness: ~56% → ~58%
- `content/post-38-isometric-training.md` — sports science deep-dive on isometric training (sports-scientist, clinical_safety_status: PASS)
  - RFD (Rate of Force Development) and motor unit synchronization
  - ±15° angle-specificity principle and sticking point exploitation
  - Yielding vs. overcoming isometric mechanics and use cases
  - Supramaximal isometric loading via power rack (>1RM force production)
  - Tendon collagen synthesis physiology under isometric load
  - Three concrete protocols: weak point overcoming isometric, yielding paused reps, intra-set PAP potentiation
  - Honest limitations section (inferior to dynamic training for hypertrophy)

---

## [0.65.1] — 2026-05-18

### Changed
- `docs/DATA_MODEL.md` v0.2 → v0.3 — four new sections:
  - **§2.8 workout_sets** — table definition (closes inconsistency: table was referenced in PITR runbook §10.3 but never defined in core schema); CV flags JSONB + per-tenant KMS encryption note; composite unique index on `(workout_id, set_number)`.
  - **§2.9 coaching_sessions + coaching_turns** — Victor interaction storage with privacy floor: `coaching_turns.content` classified Confidential; coaching turns excluded from `tenant_wellness_summary` MV; token/voice-char instrumentation columns for COST_MODEL reconciliation (OQ-01/OQ-02).
  - **§2.10 tenant_sso_configs + tenant_scim_tokens + scim_provisioning_log** — SSO/SCIM auxiliary tables; `oidc_client_secret_enc` AES-256-GCM via KMS; `token_hash` write-only Restricted classification; `scim_provisioning_log` for IdP sync debugging; cross-reference to `docs/SSO_SCIM_IMPLEMENTATION.md`.
  - **§3.6–3.7** — RLS policies for all six new tables; `form_system` role on SSO config read path; partial index for non-revoked SCIM tokens.
  - **§5** — data classification table updated for new tables (`coaching_turns.content` Confidential, `oidc_client_secret_enc` Sensitive, `token_hash` Restricted).
  - **§11 Index Strategy for RLS Performance** — isolation indexes `(tenant_id)` and `(tenant_id, user_id)` on all tables; relationship-traversal indexes for RLS subqueries (`workout_id`, `session_id`); SSO/SCIM hot-path indexes; BRIN for time-series aggregate scans; partial index on recent workouts (90-day MV window); index maintenance policy (CONCURRENTLY, quarterly `pg_stat_user_indexes` review). (enterprise-architect · compliance-officer · security-engineer)

---

## [0.65.0] — 2026-05-18

### Added
- `content/post-37-blood-flow-restriction.md` — BFR-тренінг: механізм оклюзії (венозна, не артеріальна), три гіпертрофічні шляхи (метаболічний стрес + рекрутинг Type II волокон, клітинне набухання, GH-викид), порівняння з традиційним HL-тренінгом (гіпертрофія порівнянна, сила — ні), три контексти де BFR виправданий (реаб після операцій, детренованість, finisher), практичні протоколи (20–40% 1RM, 30+3×15, 30–45 сек відпочинок), протипоказання (DVT, серцева недостатність), матриця застосування 8 сценаріїв. (sports-scientist · clinical-safety PASS)
- `blog.html` — картка post-37 BFR-тренінг
- `STATUS.md` — рядок post-37 у content engine table

---

## [0.64.1] — 2026-05-18

### Changed
- `docs/SOC2_READINESS.md` — Section 16: Penetration Test Program. Scope definition (API, auth flows, SSO/SCIM endpoints, RLS-via-API, iOS/Android apps, Cloudflare edge); methodology (OWASP WSTG + ASVS L2 + MASVS L1/L2 + PTES + CWE Top 25); finding severity SLAs (Critical 24h, High 7d, Medium 30d, Low 90d) with health-data and tenant-isolation severity-uplift rules; remediation tracking via Linear (intake → triage → fix → re-test → evidence filing); SOC 2 evidence package (engagement letter + full report + HMAC chain verification per DEC-030 + remediation sign-off); customer disclosure policy (executive summary under NDA for >$50k ACV). CC7 control table updated with "External penetration test" row (🟡 Partial). PRE-21 moved 🔴 Open → 🟡 Partial. Readiness score updated to ~55% (catching up from earlier v0.5 changes). Gap delta: CC7.1, CC7.2 🔴 → 🟡; critical gaps 4 → 3. (compliance-officer · security-engineer · enterprise-architect)

---

## [0.64.0] — 2026-05-18

### Added
- `content/post-36-detraining.md` — Детренованість: нейром'язові vs структурні адаптації та їх різні строки деградації, міонуклеарна гіпотеза м'язової пам'яті (Bruusgaard 2010), мінімальна ефективна доза для maintenance при збереженій інтенсивності (Bickel 2011), матриця повернення за тривалістю паузи 1–12+ тижнів, асиметрія реадаптації м'язів vs сполучної тканини. (sports-scientist · clinical-safety PASS)

---

## [0.63.0] — 2026-05-18

### Added
- `content/post-33-energy-systems-for-strength.md` — Energy systems для силовика: ATP-PCr, гліколіз, окислювальна система. Практика відновлення між підходами, HRV, aerobic capacity у силовому контексті. (sports-scientist · clinical-safety ✓)
- `content/post-34-supersets-antagonist-pairs.md` — Supersets і antagonist pairs: таксономія, механізм (reciprocal inhibition + antagonist potentiation via PAP), дані щодо збереження сили, практичне програмування. (sports-scientist · clinical-safety ✓)
- `content/post-35-tempo-training.md` — Tempo prescription: TUT myth vs механічна напруга, ексцентрична механіка (titin, stretched-position), intentional velocity концентрика, isometric holds, CV-feedback. (sports-scientist · clinical-safety ✓)

---

## [0.61.1] — 2026-05-18

### Changed
- `docs/COST_MODEL.md` — v0.3 → v0.4. Recalculated all ARPU-dependent sections to reflect Pro consumer price of **$19/month** (Western markets), per `pricing.html`. Changes span §4 (unit economics table), §5 (break-even: 6,530 → 5,112 Pro subscribers), §6 (gross margin path: ~67–78% → ~79–87%; FORM benchmark: 65–75% → 75–85%), §8.4 (enterprise vs. consumer ARPU comparison), §9.2 (App Store tax table), §10.1–10.3 (sensitivity margins: SBP baseline 81.7% → 82.4%), §12.2 (freemium break-even: 1.05% → 0.82%). Geo-pricing note added in §4 ($9/month UA/EE/PL; enterprise direct-billed regardless of geo).
- `VERSION` → 0.61.0 → 0.61.1

---

## [0.61.0] — 2026-05-17

### Added
- `content/post-32-cluster-sets.md` — «Cluster sets — як накопичувати об'єм без кумулятивної втоми» (~2 300 слів, 13 хв). Охоплює: PCr кінетику в стандартному підході — виснаження за 6–8 с (Gaitanos et al., 1993, *J Appl Physiol*), часткове відновлення 15–60 с (Greenhaff et al., 1994; Hultman et al., 1967); velocity loss (VL%) як проксі-показник нейром'язової втоми — поріг 20–25% (Pareja-Blanco & Sánchez-Medina, 2017, *Sports Med*), механізм H⁺/тропонін (Fitts, 2008, *J Physiol*); таксономія методів: inter-repetition clusters (класичні), rest-pause, myo-reps (Borge Fagerli, 2010) — визначення, мета, умови застосування; дані досліджень: Oliver et al. (2013, *J Human Kinetics*) — VL cluster ~8% vs traditional ~28% при 80% 1ПМ; Haff et al. (2003, *JSCR*) — пікова сила і потужність у важкоатлетів при 90% 1ПМ; Denton & Cronin (2006, *JSCR*) — 15–20 с достатньо для часткового PCr-буфера; таблиця PCr-відновлення (10/15–20/30/60 с → % відновлення → ефект); умови застосування: 80–95% 1ПМ силовий і потужнісний акцент, volume accumulation myo-reps, коли НЕ потрібно (< 80%, новачки, часовий дефіцит); практичний приклад програмування: 4×(2+2) vs 4×4 при 87,5%, 5×(1+1+1) при 90% для потужнісних вправ; 4 типові помилки (занадто короткі паузи, застосування для всього, myo-reps до відмови кілька разів, одночасне нарощення об'єму); Victor-контекст: real-time VL tracking через CV bar velocity estimation, валідація техніки між мікроблоками. Clinical-safety: PASS (суто методологія тренування, без харчового/естетичного/ментального контенту).

### Changed
- `blog.html` — картка Post 32 (cluster sets) додана до card-grid перед Post 31. Загальна кількість карток: 31 → 32.
- `STATUS.md` — content engine: рядок post-32 доданий.
- `README.md` — content engine roadmap: post-32 позначений `[x]`.
- `VERSION` → 0.60.0 → 0.61.0

---

## [0.60.0] — 2026-05-17

### Added
- `content/post-31-unilateral-training.md` — «Унілатеральний тренінг: коли і чому (асиметрія сили, стабільність кору, трансфер у спорт)» (~2 400 слів, 14 хв). Охоплює: bilateral deficit (BLD) — феномен і нейронний механізм (іпсілатеральне інгібування, crossed extensor reflex suppression) з даними Secher et al. (1988, *J Appl Physiol*) і Koh et al. (2009, *Eur J Appl Physiol*); поширеність асиметрії сили ніг у спортсменів — Meylan et al. (2010, *JSCR*), поріг 10–15% як маркер продуктивності та ризику травм (не естетики); чому bilateral тренування не виправляє асиметрію автоматично — компенсаційні патерни і обґрунтування примусового унілатерального навантаження; McCurdy et al. (2005, *JSCR*) — результати 8-тижневого порівняльного дослідження single-leg squat vs bilateral squat; cross-education effect (контралатеральний перенос сили) — Munn et al. (2004, *J Appl Physiol*) meta-analysis ~8% трансфер; спортивний трансфер — Speirs et al. (2016, *JSCR*) і Jones et al. (2009) спринт і зміна напрямку; стабільність кору: Behm et al. (2010, *Sports Med*) — реальна різниця активації стабілізаторів vs prime movers у унілатеральних умовах; Bulgarian split squat — два окремих бар'єри для початківців (баланс/пропріоцепція і мобільність hip flexor), рекомендація: перші 2–3 тижні = моторне навчання, не силове тренування; порівняльна таблиця bilateral vs. unilateral (7 критеріїв); розподіл об'єму між bilateral і unilateral: базовий (60–70%/30–40%), при асиметрії або спортивному акценті (50/50 або 40/60), практичний приклад 4-денного тижня; Victor-контекст: CV pose estimation для виявлення pelvic drop, knee valgus і асиметрії глибини в унілатеральних вправах. Цитати: Secher et al. (1988), Koh et al. (2009), Meylan et al. (2010), McCurdy et al. (2005), Munn et al. (2004), Speirs et al. (2016), Jones et al. (2009), Behm et al. (2010). Clinical-safety: PASS.

### Changed
- `blog.html` — картки Post 30 і Post 31 (eccentric training і unilateral) додані до card-grid перед Post 29. Загальна кількість карток: 29 → 31.
- `STATUS.md` — content engine: рядки post-30 і post-31 додані.
- `README.md` — content engine roadmap: post-30 і post-31 позначені `[x]`.
- `VERSION` → 0.59.0 → 0.60.0

---

## [0.59.0] — 2026-05-17

### Added
- `content/post-30-eccentric-training.md` — «Ексцентричне тренування — чому фаза опускання важливіша за підйом» (~2 500 слів, 15 хв). Охоплює: крива сила-швидкість і чому ми можемо опускати більше, ніж підіймаємо (~20–50% ексцентрична перевага — Enoka, 1996, *J Appl Physiol*); titin як молекулярна пружина і чому активація mTORC1 вища в подовженому стані — Schoenfeld (2010, *JSCR*), Lindstedt et al. (2001, *Exerc Sport Sci Rev*); EIMD: Z-disk streaming, sarcolemmal disruption, repeated bout effect — Proske & Morgan (2001, *J Physiol*), обговорення як фізіологія ремоделювання (не прославлення болю), посилання на post-20 для деталей DOMS; нейром'язова специфіка: переважне рекрутування тип-II волокон при ексцентричному навантаженні, відмінності alpha-motoneuron inhibition — Enoka (1996); мета-аналіз Roig et al. (2009, *Br J Sports Med*): eccentric d=0.88 vs. concentric d=0.49 для сили; d=0.54 vs. d=0.27 для гіпертрофії; саркомерогенез і довжина фасцикул: Franchi et al. (2014, *J Appl Physiol*) — +11% довжина фасцикул у ексцентричній групі, зменшений кут пернатості; AEL-протоколи: 4 практичні рівні (supramaximal, weight releasers, tempo control, unilateral eccentric/bilateral concentric) — Hody et al. (2019, *Front Physiol*); Nordic hamstring curl: 51% зниження ризику травм (van Dyk et al. 2019), механізм через збільшення довжини фасцикул; темп-нотація 3-1-0: що знищується при скиданні ваги; 4 типові помилки (bounce, непослідовний темп, прискорення у термінальній фазі, різкий об'ємний стрибок); практична матриця програмування за контекстом; Victor-контекст: детекція швидкості грифа, аналіз кутів суглобів, відстеження послідовності глибини. Цитати: Enoka (1996), Schoenfeld (2010), Lindstedt et al. (2001), Proske & Morgan (2001), Roig et al. (2009), Franchi et al. (2014), Hody et al. (2019). Clinical-safety: PASS.

### Changed
- `VERSION` → 0.58.1 → 0.59.0

---

## [0.58.1] — 2026-05-17

### Changed
- `docs/COST_MODEL.md` v0.2 → v0.3 — §12 Free Tier Subsidy Model & Freemium Funnel Economics added. Covers: total infrastructure subsidy at scale (500–100k free MAU); minimum conversion rate for cost neutrality (break-even algebraically derived at **1.05% lifetime conversion**); per-phase conversion rate targets; freemium CAC vs. paid UA comparison (infra-only CAC ≤ $0.78 at 6-month conversion window vs. typical paid UA $30–80); free tier cost control levers (voice quota, session depth, context truncation); free pool governance triggers ($500/$2k/$5k monthly subsidy thresholds). Closes the gap flagged in §3.4. Audience: founder, investors, growth-lead.
- `VERSION` → 0.58.0 → 0.58.1

---

## [0.58.0] — 2026-05-17

### Added
- `content/post-29-mind-muscle-connection.md` — «Зв'язок розум-м'яз — наука, бро-наука, або щось посередині?» (~2 600 слів, 13 хв). Охоплює: internal vs. external focus of attention (IFA/EFA); OPTIMAL theory Wulf — обмеження екстраполяції з моторного навчання на силові тренування; EMG-дані Schoenfeld & Contreras (2018, *Eur J Sport Sci*) — +12–15% активація bicep при IFA в ізоляційних вправах; Calatayud et al. (2016, 2017) — ефект зникає при >75–80% 1RM; чому навантаження змінює рівняння (рекрутування рухових одиниць); матриця застосування IFA/EFA за типом вправи і навантаженням; методологічні обмеження: EMG ≠ гіпертрофія, проблема зовнішньої валідності, неоднорідність визначень IFA у дослідженнях; practical frame «intent vs. execution»; CV-контекст: що pose estimation може відстежити опосередковано. Цитати: Wulf & Lewthwaite (2016 *Psychonomic B&R*), Schoenfeld & Contreras (2016 *SCJ*; 2018 *Eur J Sport Sci*), Calatayud et al. (2016 *Eur J Appl Physiol*; 2017 *J Hum Kinet*), Wulf (2013 *IRSEP*). Clinical-safety: PASS.

### Changed
- `blog.html` — картка Post 29 (mind-muscle connection) додана до card-grid перед Post 28. Загальна кількість карток: 28 → 29.
- `STATUS.md` — content engine: рядок post-29 додано (draft · clinical-safety PASS · sports-scientist pending).
- `README.md` — content engine roadmap: post-28 позначено [x], post-29 позначено [x], додано нові теми post-30–35 (eccentric training, unilateral, cluster sets, energy systems, supersets, tempo).
- `VERSION` → 0.57.0 → 0.58.0

---

## [0.57.0] — 2026-05-17

### Added
- `content/post-28-progressive-overload-mechanics.md` — «Прогресивне перевантаження — механізм адаптації, а не мотиваційна мантра» (~2 700 слів, 14 хв). Охоплює: mechanical tension як головний драйвер гіпертрофії, mTORC1-шлях, repeated bout effect, SRA-крива (Stimulus → Recovery → Adaptation) з поясненням різних часових рамок для м'язів vs. сполучної тканини. Шість форм прогресії в ієрархії (load → volume → density → frequency → ROM → технічна якість). Чому лінійна прогресія ламається у intermediate-атлетів: нейронний vs. структурний резерв, накопичення втоми приховує fitness. Практичне дерево рішень: RPE-gate → volume threshold → load bump (double progression). Шість найпоширеніших помилок: PR кожного тижня, деградація техніки, плутанина втома/адаптація, часта зміна вправ, агресивне нарощування об'єму, ігнорування не-тренувальних змінних. Контекст AI-тренера: детекція стагнації, deload на основі реального RPE vs. запланованого. Цитати: Schoenfeld (2010 *JSCR*; 2016 *Sports Med*; 2017 *J Sports Sci*), Helms et al. (2016 *SCJ*), Krieger (2010 *JSCR*), Ralston et al. (2017 *Sports Med*), Plisk & Stone (2003 *SCJ*), Zourdos et al. (2016 *JSCR*), Dankel et al. (2017 *Eur J Appl Physiol*). Clinical-safety: PASS.
- `docs/SOC2_READINESS.md` Section 15 — Annual Compliance Calendar. Три підрозділи: (15.1) Master Compliance Calendar — 16 recurring activities mapped to SOC 2 controls, named owners, and evidence artifacts across monthly/quarterly/annual cadences; (15.2) Pre-Observation Period Readiness Checklist — 27 PRE items across Legal/Policy Foundations (PRE-01–09), Technical Controls (PRE-10–19), Process Controls (PRE-20–27), plus Audit Firm Engagement milestone timeline (Month O-6 to O+9); (15.3) First-Year Implementation Priority matrix — Before First Hire (12 items P1–P3 with effort estimates) vs. After First Hire (10 items), plus documented Compensating Controls acceptance for 5 solo-founder-specific gaps. Moves 7 items from 🔴 Gap → 🟡 Partial (security training Q1-Feb, vendor review Q1-Jan, DR drill Q1-Jan, privacy review Q1-Jan, offboarding quarterly, media disposal Q2-Jun, control effectiveness review quarterly). Critical gaps: 6 → 4. Partial: 21 → 28. Readiness: ~51% → ~55%. Open Items updated: draft privacy policy ✅, DR drill scheduled ✅.

### Changed
- `blog.html` — картка Post 28 (прогресивне перевантаження — механізм адаптації) додана до card-grid після Post 27. Загальна кількість карток: 27 → 28.
- `STATUS.md` — content engine: рядок post-28 додано (draft · clinical-safety PASS · sports-scientist DONE).
- `docs/SOC2_READINESS.md` — version footer v0.4 → v0.5; Open Items оновлено: 5 нових відкритих пунктів (Sentry DPA, uptime monitoring, pentest, security training first cohort, first access review); DR drill date тепер `[x]` (Section 15.1 Q1-Jan calendar entry).
- `VERSION` → 0.56.1 → 0.57.0

---

## [0.56.1] — 2026-05-17

### Changed
- `docs/OBSERVABILITY.md` — +300 lines, three new sections:
  - **§0 RED Methodology** — explicit Rate/Error/Duration service map for all 10 production services (Workers API, Supabase Postgres, Auth, Anthropic, ElevenLabs, R2 cache, iOS/Android mobile, EAS, enterprise SSO); triage cascade description for cross-tier failure attribution.
  - **§11 SLI Calculation Expressions** — PromQL formulas for all 9 SLIs in §2.1 (availability, P95 latency, crash-free rate, enterprise SSO per-tenant), plus error-budget consumption formula with worked example.
  - **§12 Developer Instrumentation Guide** — Sentry Performance span API for Workers (TypeScript), Analytics Engine `writeDataPoint()` with CFA SQL query example, OTel OTLP-via-fetch pattern for the Series A ClickHouse path, React Native screen instrumentation, and a 10-point instrumentation checklist (tenant_id, user_id_hash, GDPR Art. 9 health data exclusions, alert review).

## [0.56.0] — 2026-05-17

### Added
- `content/post-27-training-age-vs-chronological-age.md` — «Тренувальний вік проти хронологічного: чому "пізно починати" — не та проблема» (~2 200 слів, 13 хв). Охоплює: тренувальний вік як окрема змінна, нейром'язові vs морфологічні адаптації (Unhjem 2015), порівняльна матриця трьох профілів, дані про початківців 40–70 (Reeves 2004, Fragala et al. 2019 NSCA position statement, Peterson et al. 2011), inflammaging і відновлення, практичні імплікації для програмування на трьох рівнях тренувального віку, Victor-специфічні рекомендації для Дмитра/Миколи/новачка 40+. Цитати: Unhjem et al. (2015, *J Electromyogr Kinesiol*), Peterson et al. (2011, *Ageing Res Rev*), Bhasin et al. (2001, *AJP Endocrinol Metab*), Reeves et al. (2004, *Exp Physiol*), Fragala et al. (2019, *JSCR*), Narici & Maffulli (2010, *Br Med Bull*). Clinical-safety: PASS.
- `blog.html` — картки Post 26 (сухожилля) і Post 27 (тренувальний вік) додані до card-grid після Post 25. Загальна кількість карток: 25 → 27.
- `STATUS.md` — content engine table: рядки post-26 і post-27 додані.

### Changed
- `README.md` — post-27 відмічено `[x]` у content engine roadmap.
- `VERSION` → 0.55.0 → 0.56.0

## [0.55.0] — 2026-05-17

### Added
- `content/post-26-tendon-ligament-adaptation.md` — «Адаптація сухожиль і зв'язок: чому сполучна тканина не встигає за м'язами» (~2 000 слів). Охоплює: аваскулярність і колагенові цикли, синдром «сильний м'яз — слабке сухожилля», overuse tendinopathy механізм, HSR vs ексцентрика, 90% isometric threshold (Arampatzis), практичні імплікації для програмування і Victor. Цитати: Kjaer (2004, *Physiol Rev*), Magnusson et al. (2010, *Nat Rev Rheum*), Arampatzis et al. (2007, *J Exp Biol*; 2010, *J Biomech*), Beyer et al. (2015, *AJSM*), Rio et al. (2015, *BJSM*), Alfredson et al. (1998, *AJSM*). Clinical-safety: CONDITIONAL PASS → PASS після 2 mandatory edits.
- `docs/PRIVACY_POLICY.md` — повна GDPR-сумісна Privacy Policy (807 рядків, v0.1.0-draft, pre-legal-review). 9 категорій даних; legal basis Art. 6 + Art. 9(2)(a); 9 sub-processors із DPA-статусом і SCCs Module 2; 8 GDPR rights; DSAR SLA 30 днів; Appendix A SOC2 mapping (17 annotations); Appendix B Apple App Store disclosure. Закриває 3 critical SOC2 gaps: P1.1, P1.2, CC9.2+P6.1. Critical gaps: 9 → 6. SOC2 readiness: 45% → ~51%.

### Changed
- `README.md` — post-26 відмічено `[x]`.
- `VERSION` → 0.54.1 → 0.55.0

---

## [0.54.1] — 2026-05-17

### Added
- `docs/SOC2_READINESS.md` Section 14 — Formal Risk Register (CC3). 18 risks across 6 categories: Security (SR-01…SR-05), Availability (AR-01…AR-04), Processing Integrity (IR-01…IR-02), Confidentiality (CR-01…CR-02), Privacy (PR-01…PR-02), Vendor/Operational (VR-01…VR-02, OR-01). Each risk carries: description, inherent Likelihood × Impact score, existing controls, residual score, named owner, and mitigation status. Risk appetite statement: FORM tolerates MEDIUM residual risk; zero tolerance on health data confidentiality and HMAC audit chain integrity. Heatmap summary included. Closes CC3 gaps for SOC 2.

### Changed
- `docs/SOC2_READINESS.md` CC3 controls — `Formal risk assessment documented` updated 🟡 Partial → ✅ Done (Section 14); `Risk register maintained, reviewed annually` updated 🔴 Gap → 🟡 Partial (Section 14 exists; first annual review scheduled August 2026).
- `docs/SOC2_READINESS.md` P3 control — `Purpose limitation documented` updated 🟡 Gap → ✅ Done (docs/GDPR_DPIA.md v0.1, May 2026 — DPIA was shipped at v0.50.0 but SOC2 doc was not updated).
- `docs/SOC2_READINESS.md` Gap Analysis Summary — critical gaps: 10 → 9; partial: 22 → 21; controls in place: 23 → 25; readiness score: 42% → 45%.
- `docs/SOC2_READINESS.md` Open Items — DPIA and risk register marked `[x]` done.
- `docs/SOC2_READINESS.md` version footer — v0.3 → v0.4.
- `VERSION` → 0.54.0 → 0.54.1

---

## [0.54.0] — 2026-05-17

### Added
- `content/post-25-breathing-mechanics.md` — sports-science post «Як дихати під штангою: Valsalva, bracing і внутрішньочеревний тиск без містики». ~1750 слів, 13 хв читання. Охоплює: IAP як механізм стабілізації хребта (McGill 2002/2010, Harman 1988); Valsalva maneuver — механіка, пікові значення тиску, реальні протипоказання; bracing vs. hollowing — чому bracing перевищує hollowing для компаундних підйомів (McGill, Grenier & Norman 2003); практична модель застосування за контекстом (важкі компаунди / субмаксимальні ваги / ізоляція / overhead); топ помилок. clinical_safety_status: PASS.
- `blog.html` → картка post-25 («Як дихати під штангою») додана до card-grid. Загальна кількість карток: 24 → 25.
- `STATUS.md` → рядки post-24 і post-25 додані до content engine table.
- `README.md` → roadmap оновлено: post-24 і post-25 позначені як done; post-28 (Mobility work) додано як наступний.

## [0.53.0] — 2026-05-17

### Added
- `content/post-24-training-to-failure.md` — sports-science post «До відмови чи ні: коли остання повторення має значення, а коли руйнує наступну». ~1800 слів, 12 хв читання. Охоплює: три типи відмови (concentric muscular / technical / volitional); огляд Schoenfeld & Grgic 2019 — proximity-to-failure як основний предиктор гіпертрофії; Refalo et al. 2022 мета-аналіз — failure vs 1–2 RIR при зіставному об'ємі дає однакову гіпертрофію; Morton et al. 2019 — широкий RIR-діапазон і зіставний ріст; Sampson & Groeller 2016 — нейром'язова стомленість після failure sets; контексти де відмова виправдана (ізоляція, 1RM-тест, кінець акумуляційного блоку); практична таблиця RIR за контекстом. clinical_safety_status: PASS.
- `blog.html` → картка post-24 («До відмови чи ні») додана до card-grid. Загальна кількість карток: 23 → 24.
- `docs/COST_MODEL.md` §8.5 Enterprise CAC Model — трирівнева модель (20/100/500 seats) з розбивкою по: AE/SDR outreach, technical POC, legal review, amortized content. Cash CAC range: $1,000–$14,000 залежно від розміру deal-у.
- `docs/COST_MODEL.md` §8.6 LTV/CAC Analysis — LTV = ACV × 3yr × 89% GM. Всі три розміри deal-у повертають LTV/CAC вище 10×: Small 11.9×, Mid 22.9×, Large 52.2×. Payback period: 3 → 0.6 місяців.
- `docs/COST_MODEL.md` §8.7 Expansion and Churn Economics — NRR модель для когорти 10 mid-deal ($360k ARR): 3 сценарії (80% / 105% / 117% NRR); expansion triggers (SSO-based frictionless rollout, pilot→company-wide); churn signals (champion departure, <30% utilization by M3, Q4 budget reset). Benchmark: Series A target >110% NRR.
- `docs/COST_MODEL.md` §8.8 Year 1 vs Year 2+ Deal Economics — таблиця waterfall: імплементаційна вартість amortized → +4.1 pp margin у Year 2. Multi-year contract strategy: 5–8% discount виправданий.
- `docs/COST_MODEL.md` §10.5 Enterprise Revenue Mix Sensitivity — 4-сценарна таблиця (5% → 80% enterprise mix): blended GM від ~82% до ~88%. Висновок: mix — вторинний lever, стратегічний кейс для enterprise — NRR і predictability.

### Changed
- `docs/COST_MODEL.md` Table of Contents — додані посилання на §8.5–8.8 та §10.5.
- `docs/COST_MODEL.md` version footer — v0.1 → v0.2.
- `VERSION` → 0.52.1 → 0.53.0

---

## [0.52.1] — 2026-05-17

### Added
- `docs/SOC2_READINESS.md` Section 13 — Data Classification Policy. Four-tier taxonomy: **Public** (marketing, open docs) / **Internal** (team docs, roadmap) / **Confidential** (user PII, tenant metadata, API keys, audit logs) / **Restricted** (GDPR Art. 9 health data: biometrics, CV keypoints, ED-screening responses, mental health self-reports, body composition). Each tier defines encryption requirements, access controls, logging rules, breach notification SLA, HR visibility rules, LLM prompt inclusion rules, and code label (`data_class` field). Full FORM data inventory mapped to tiers. Audit log `data_class` field defined with alerting rule (any `class:restricted` non-system access → `#security-alerts` within 30s). Quarterly compliance SQL query included. Closes SOC 2 criteria C1.1.

### Changed
- `docs/SOC2_READINESS.md` C1.1 control row — `Confidential data classification policy` updated from 🔴 Gap → ✅ Done.
- `docs/SOC2_READINESS.md` Gap Analysis Summary — critical gaps: 11 → 10; controls in place: 22 → 23. Readiness score: 40% → 42%.
- `docs/SOC2_READINESS.md` Open Items — `Define data classification policy tiers` marked `[x]`.
- `docs/SOC2_READINESS.md` version footer — v0.2 → v0.3.
- `VERSION` → 0.52.0 → 0.52.1

---

## [0.52.0] — 2026-05-17

### Added
- `content/post-23-velocity-based-training.md` — sports-science post «Швидкість штанги — кращий датчик, ніж твій відсоток від максимуму». ~2500 слів, 13 хв читання. Охоплює: проблема %1RM і денні коливання ±7-11%; load-velocity relationship і таблиця MPV → % 1RM для присідань; Velocity Loss Threshold (VLT) і Minimum Velocity Threshold (MVT); Pareja-Blanco et al. 2017 — 20% vs 40% VLT і різний адаптаційний відгук (сила vs метаболічний стрес); побудова індивідуального L-V профілю; практична реалізація без Gymaware (iPhone slow-mo, myLift, Open Barbell, суб'єктивна RPV); три схеми впровадження за рівнем. Всі 5 джерел вбудовані в текст. clinical_safety_status: PASS.
- `blog.html` → картки post-22 (periodization, 15 хв) і post-23 (VBT, 13 хв) додані до card-grid після post-21.

### Changed
- `README.md` → roadmap: post-22/23 позначені `[x]`; нові теми post-24..27 (mobility, breathing mechanics, tendon adaptation, training age) додані до roadmap
- `STATUS.md` → записи post-21, post-22, post-23 додані до content engine таблиці
- `VERSION` → 0.51.0 → 0.52.0

---

## [0.51.0] — 2026-05-17

### Added
- `content/post-22-periodization.md` — sports-science post «Лінійна, хвиляста, блочна: як обрати модель periodization під свій рівень». ~2200 слів. Охоплює лінійну periodization (для початківців та піків), DUP з практичним прикладом тижня (сила/гіпертрофія/потужність), блочну periodization (акумуляція → інтенсифікація → реалізація), матрицю рішень за стажем/частотою/складністю, ACSM position, Issurins block model і DUP мета-аналітичну базу. clinical_safety_status: PASS. Голос Віктора в фіналі.
- `docs/DATA_MODEL.md` Section 10 — PITR Tenant-Isolated Restore Runbook (SOC 2 criteria A1.2/A1.3). 7-кроковий runbook: PITR-вікно, ізольований RDS-кластер, pg_dump з tenant_id фільтром, re-import через INSERT … ON CONFLICT, teardown, пост-дрил запис. Закриває SOC 2 gap 9.4.

### Changed
- `docs/DATA_MODEL.md` — v0.1 → v0.2; gap 9.4 позначений RESOLVED; ToC оновлено.
- `VERSION` → 0.50.0 → 0.51.0

---

## [0.50.0] — 2026-05-17

### Added
- `docs/GDPR_DPIA.md` v0.1 — повний Data Protection Impact Assessment під GDPR Art. 35 для FORM's health data processing. Охоплює: обов'язковість DPIA (5 з 9 EDPB-критеріїв виконані — special category, large-scale, systematic profiling, innovative tech, vulnerable subjects); inventory 12 processing activities (PA-01…PA-12) з legal basis для кожної; data flow diagrams для consumer та enterprise tier; data residency (us-east-1 / eu-central-1 / eu-west-1) і SCC transfer mechanism; necessity & proportionality analysis — доведено, чому CV frames ніколи не залишають device, чому meal logs excluded з enterprise aggregates, чому k-anon floor = 5; risk register 12 ризиків (R-01…R-12) з Likelihood × Severity scoring, inherent та residual scores; технічні та організаційні заходи (RLS fail-closed, per-tenant KMS, HMAC audit chain DEC-030, ED-screening isolation DEC-018, clinical-safety VETO); residual risk statement — overall MEDIUM-HIGH, accepted; supervisory authority consultation assessment (voluntary DPC engagement recommended); blocking open items перед EU enterprise launch (privacy policy, consent granularity, Sentry DPA, DSAR runbook). Закриває 🔴 critical gap з docs/SOC2_READINESS.md і відкриває шлях до EU enterprise sales.

### Changed
- `VERSION` → 0.49.0 → 0.50.0

---

## [0.49.0] — 2026-05-16

### Added
- `content/post-21-cv-vs-wearables.md` — «Чому FORM обрав комп'ютерний зір замість wearable-only підходу». 14-хв читання. Охоплює: що wearables вимірюють добре (HRV, ЧСС, сон) і де їхня принципова межа (якість руху); як pracює pose estimation (MediaPipe BlazePose, MoveNet); on-device inference на Neural Engine (<10ms/frame); що аналізує CV — кути суглобів, bar path, симетрія, velocity; чесні обмеження (освітлення, occlusion, кут камери); CV + wearables як ортогональні виміри (готовність vs якість руху). clinical-safety: PASS — технічна тема без харчових, дієтичних або тілесних тригерів. sports-scientist + platform-engineer pending.
- `blog.html` → картка post-21 (category: product) додана до card-grid після post-20.

### Changed
- `README.md` → синхронізовано з поточним станом: версія v0.31.0 → v0.49.0, «17 blog post drafts» → «21», «10 blog posts» у content/ → «21», blog.html «9 posts» → «21 posts», content/ directory listing розширено (post-11…post-21), метрики оновлено (releases 45+ → 60+, docs 58+ → 65+, total lines 30k+ → 38k+, decisions 30 → 31), content roadmap: post-18/19/20 відмічені [x], post-21 доданий [x], post-22/23 плановані.
- `VERSION` → 0.48.2 → 0.49.0

---

## [0.48.2] — 2026-05-16

### Changed
- `pricing-enterprise.html` → v0.48.2: added **"What do you need?"** requirement-toggles block to the enterprise pricing calculator. Three interactive checkboxes (SSO / SCIM, Dedicated CSM, SOC 2 Type II) enforce minimum tier seat counts — CSM bumps minimum to 201 seats (Growth+), SOC 2 bumps to 1,001 seats (Enterprise). Auto-adjusts seat slider with a visible enforcement notice. Wired into calculator `render()` loop so discounts, tier badge, and feature matrix all respond live. CSS: new `.req-check`, `.req-label`, `.req-tier-pill`, `.req-notice` classes.
- `VERSION` → 0.48.1 → 0.48.2

---

## [0.48.1] — 2026-05-16

### Added
- `content/post-20-doms-muscle-soreness.md` — «Крепатура — це не індикатор тренування». 13-хв читання. Розбір механізмів DOMS (EIMD, запальний каскад, нейросенсорна гіперчутливість), міф «без болю немає росту», repeated bout effect, практичні висновки для self-coached атлетів. clinical-safety PASS.
- `blog.html` → Post 20 card додано до card-grid
- `STATUS.md` → рядок post-20 у content engine table
- `VERSION` → 0.48.0 → 0.48.1

---

## [0.48.0] — 2026-05-16

### Added
- `content/post-19-warmup-protocols.md` — «Warm-up протоколи — чому більшість не відповідає реальним потребам атлета». Спортивна наука: термічна розминка, статичний vs. динамічний стретчинг (meta-analysis Simic 2013, Behm & Chaouachi 2011), PAP vs. рампові підходи, індивідуальна варіабельність. clinical-safety: PASS WITH MINOR EDITS — 4 правки застосовані (stop-cue, pain-signal, Gołaś citation removed, PAP translation fixed).
- `docs/MOBILE_ROADMAP.md` — канонічний документ мобільного продукту: v1.0/v1.1/v2.0 scope, технічні constraints (CV battery budget, latency SLOs, on-device/cloud boundary), deferred items table, 11 open questions для Founding Engineer, integration touchpoints з цим репо. Правило для cloud agents: mobile code не в цьому репо.
- `blog.html` — картка post-19 у гриді блогу.

### Fixed (clinical-safety edits on post-19)
- «Виживання» passage: додано stop-and-assess cue замість чистого warm-up diagnosis
- Ramp sets section: додано одне речення про гострий/нетиповий біль як сигнал зупинитись
- Видалено непідтверджений Gołaś et al. citation з body text
- Виправлено «пострецепторна потенціація» → «постактиваційна потенціація»

---

## [0.47.2] — 2026-05-16

### Changed
- `admin-dashboard.html` → v0.47.2: expanded from single static Overview screen to fully interactive multi-screen mockup. Added JavaScript tab-switching nav. New screens:
  - **Users** — anonymised user table with search/filter by status and department.
  - **Engagement** — 8-week W-ACSU trend chart (Acme Corp vs consumer average), D30 retention and session-frequency stats, privacy-floor notice.
  - **Reports** — downloadable SOC 2 evidence package, SLA uptime report, seat-utilization CSV, GDPR data-processing summary; scheduled delivery table.
  - **Branding** — Enterprise-plan gate with CSM CTA.
  - **SSO · SCIM** — complete SAML 2.0 SP config panel (ACS URL, Entity ID, IdP metadata), SCIM v2 provisioning status (endpoint, token rotation, last-sync stats, attribute mapping table), JIT provisioning toggle, Force SSO toggle, MFA policy config. Functional copy-to-clipboard on all read-only fields.
  - **Roles · RBAC** — permission matrix (8 capabilities × 4 roles) with hard-never rows for individual health data; role distribution chart for Acme Corp.
  - **Billing** — contract summary (Growth plan, 412 seats, annual ACV), days-to-renewal countdown, invoice history.
  - **Audit log** — HMAC chain status banner (intact, 14,832 events), event feed with action badges (`auth.*`, `tenant.*`, `data.*`, `support.break_glass`), HMAC prefix column, filter by action type and time range, NDJSON/SIEM export buttons. Append-only notice. Implements DEC-030 (HMAC audit chain) at UI level.
- `VERSION` — 0.47.1 → 0.47.2

---

## [0.47.1] — 2026-05-16

### Added
- `content/post-18-rest-intervals.md` — blog post «Скільки відпочивати між підходами — і чому твій таймер на 60 секунд неправильний» (13 хв читання). Охоплює: PCr-ресинтез кінетика (Harris et al. 1976; Greenhaff 2001; Casey et al. 1996), Schoenfeld et al. 2016 (3 хв vs 1 хв — гіпертрофія і сила), таблиця інтервалів за ціллю (сила/гіпертрофія/витривалість/потужність), походження «60-секундного міф», активний vs пасивний відпочинок, суперсети і м'язова специфічність, ЦНС-відновлення для важкоатлетів. clinical-safety PASS (чисто фізіологічна тема, без харчових або тілесних тригерів). sports-scientist pending. Owner: content-engine.
- `blog.html` — картка post-18 (category: science) у grid після post-17.
- `STATUS.md` — рядки post-17 і post-18 у content engine table (post-17 був пропущений у попередньому оновленні).

### Changed
- `VERSION` — 0.47.0 → 0.47.1
- `STATUS.md` — версія заголовку і footer v0.47.0 → v0.47.1

---

## [0.47.0] — 2026-05-16

### Added
- `content/post-17-overtraining-vs-laziness.md` — blog post «Перетренованість vs лінощі: як не переплутати одне з одним» (11 хв читання). Охоплює: спектр FO/NFO/OTS (Meeusen et al. 2013), вимірювані маркери (HRV-тренд, ранковий ЧСС, RPE-тренд 2+ тижні, якість сну, mood як lagging indicator), ЦНС-захисний механізм (Noakes 2012, з відповідним застереженням що модель дискутується), 4-рівнева операційна вилка (один поганий день → 5-7 днів → 2+ тижні → 4+ тижнів), три питання перед тим, як назвати себе лінивим. clinical-safety PASS (два FIX застосовані: redirect до лікаря при mood-dominant NFO в level-3 блоці; carve-outs для травми/хвороби/ментального здоров'я в «не пропускай два поспіль»). Owner: sports-scientist + content-engine.
- `blog.html` — картка post-17 (category: science) у grid між post-16 і post-09.
- `docs/DECISION_LOG.md` — DEC-031: документує розширення команди агентів з 14 → 24 (10 нових ролей: enterprise-architect, compliance-officer, devops-lead, data-engineer, security-engineer, ml-engineer, qa-lead, qa-walker, customer-success, product-manager).

### Changed
- `README.md` — виправлено 7 застарілих метрик: HTML pages 13→16, markdown docs 47+→58+, blog posts 9→17, AI agents 14→24 (3 місця), decisions 28→30, total lines 22k→30k+. Секція «The team» переписана для всіх 24 агентів у 6 групах (Product+Safety, Design+Voice, Growth+Content, Engineering+Infra, Research+Process, Quality+Compliance).
- `STATUS.md` — синхронізовано версію заголовку (v0.31.0 → v0.46.1→v0.47.0 після цього bump-у), дата, TL;DR paragraph, кількість постів, агентів, footer.

---

## [0.46.1] — 2026-05-16

### Changed
- `docs/SOC2_READINESS.md` → v0.2: додано три нові розділи, які були відсутні і є обов'язковими для реального SOC 2 Type II звіту та enterprise-procurement:
  - **Sub-Processor Register** — 8 sub-processors з DPA-статусом, регіоном, механізмом GDPR-transfer (SCC Module 2); відповідає CC9 та GDPR Art. 28; 30-денний notice period перед додаванням нового.
  - **Complementary User Entity Controls (CUECs)** — 12 контролів (CUEC-01..12) що enterprise-customers мають впровадити на своєму боці: IAM (IdP security, MFA at IdP, SCIM deprovisioning SLA), data responsibility (employee consent disclosure, k-anonymity policy), incident cooperation (security contact, forensic cooperation).
  - **Common Security Questionnaire Responses** — pre-approved відповіді на CAIQ/SIG Lite питання: data residency, encryption (at-rest AES-256 + per-tenant KMS CMK, in-transit TLS 1.3 + cert pinning), penetration testing, incident notification SLAs, offboarding/deletion, BCP (RTO 4h / RPO 1h), AI training prohibition, sub-processor change notification.

---

## [0.46.0] — 2026-05-16

### Added
- `content/post-16-ai-coach-vs-pt.md` — blog post «AI-тренер проти персонального тренера: чесний trade-off» (13 хв читання). Охоплює: що PT насправді дає (real-time оцінка, ситуативна корекція, соціальна відповідальність, стосунки) і де ламається (нерівна якість, ціна, відсутність між-сесійного контексту); що AI дає (агрегація даних, постійна доступність, відсутність упередження, консистентність) і де ламається (немає фізичних очей, неможливість фізичної оцінки, контекстна сліпота); матриця вибору по сценаріях (новачок, intermediate, реабілітація, tight budget, непередбачуваний графік); гібридні варіанти; чесна позиція FORM. Засновник задекларував conflict of interest у вступі. clinical-safety PASS (без харчування/тіла/ментального). Owner: content-engine.
- `blog.html` — картка post-16 (category: industry) у grid між post-15 і post-09.
- `STATUS.md` — рядок post-16 у content engine table.
- `README.md` — розділ «Content engine roadmap (post-16+)» з темами post-17..post-20.

---

## [0.45.0] — 2026-05-16

### Added
- `pricing-enterprise.html` — interactive enterprise pricing calculator. Inputs: seat count (50–10,000, synced slider + number input), contract length (1yr / 2yr –15% / 3yr –25%), annual upfront payment toggle (+10%). Live output panel: tier auto-detect (Starter $12 / Growth $9 / Enterprise $6–8), per-seat effective rate with discounts, annual contract total, multi-year total, savings callout. Feature add-ons panel updates per tier (SSO/SCIM, Admin, Audit, CSM, SIEM, SOC 2 report, White-label, custom DPA). Feature comparison table (16 rows × 3 tiers). FAQ (billing, seats, SOC 2 timeline, DPA). CTA mailto pre-fills subject with tier + seat count. Brand tokens exact match to enterprise.html. Owner: design-craft + enterprise-architect.
- `content/post-15-training-frequency.md` — blog post «Скільки разів на тиждень качати м'яз: що каже наука і що з цим робити» (13 хв читання). Охоплює: MPS window (0–72h фазовий розклад, досвідчені vs початківці), Schoenfeld et al. 2016 (ES ~0.36 на користь 2x/тиж при рівному об'ємі), Ralston et al. 2017 (сила + обмеження при 20+ сетах), Krieger 2010 (якість підходів vs концентрація), McLester 2000 (1x vs 3x). Volume-distributor фреймворк з таблицею 1x/2x/3x. Практичні рекомендації: 3 дні = full body, 4 дні = upper/lower, 5 днів = конфігурації. «Що FORM робить»: Victor адаптує частоту по RPE-тренду, HRV-тренду, DOMS-маркеру. clinical-safety PASS (без харчування/тіла/ментального). Owner: sports-scientist + content-engine.
- `blog.html` — картка post-15 (training frequency) у grid (category: science).
- `STATUS.md` — рядки post-15, enterprise.html, pricing-enterprise.html у таблицях; оновлено agent count до 24.

---

## [0.44.0] — 2026-05-16

### Added
- `enterprise.html` — enterprise landing page (Fraunces display hero, editorial layout, "For 50+ employees" positioning). Sections: three pillars (privacy/identity/compliance), features grid with compliance badges and SLA table, privacy floor with non-negotiable HR data rules, pricing tiers (Starter $12 / Growth $9 / Enterprise $6–8 per seat), 90-day implementation timeline, pilot CTA. References ENTERPRISE.md, AUDIT_LOG_SCHEMA.md, DEC-030. Owner: enterprise-architect + compliance-officer + security-engineer.

---

## [0.43.0] — 2026-05-16

### Added
- `content/post-14-sleep-deprived-training.md` — blog post «Тренування на 4 годинах сну: що дійсно відбувається і що робити» (13 хв читання). Охоплює: тестостерон -10-15% при 5h сну (Leproult & Van Cauter 2011), кортизол і катаболічне середовище, MPS impairment (Dattilo 2011), CNS reaction time деградація, GH і SWS архітектура; фреймворк MED: -40-50% об'єму, RPE-стеля 8, 2-3 сесії/тиж, 45-хв ліміт, 3h поріг пропуску; 3-місячний горизонт повернення до прогресу. Без контенту про харчування при ГВ, постпологові зміни тіла або ментальне здоров'я батьків. clinical-safety PENDING. Owner: content-engine + sports-scientist.
- `blog.html` — картка post-14 у grid (category: science).
- `STATUS.md` — рядок post-14 у content engine table.

---

## [0.42.0] — 2026-05-16

### Added
- `blog.html` — картка post-13 (RIR/RPE autoregulation) у grid (category: science).
- `STATUS.md` — рядок post-13 у content engine table.

---

## [0.41.0] — 2026-05-16

### Added
- `docs/OBSERVABILITY.md` — metrics/logs/traces taxonomy (629 рядків). SLIs/SLOs per service, metrics taxonomy (infrastructure/product/business), structured log schemas з GDPR Art. 9 compliance (AI inference logs: tokens+latency, no content), distributed trace schema з tenant_id propagation, 20+ alerting rules P0–P3, 7 dashboard definitions, tooling decisions (Better Stack vs Datadog), error budget policy. SOC 2 evidence: CC7.2. Owner: devops-lead.

---

## [0.40.0] — 2026-05-16

### Added
- `docs/COST_MODEL.md` — unit economics і cost model (416 рядків). COGS breakdown по тирах (Free/Pro/Enterprise), Anthropic API cost derivation, ElevenLabs voice cost, App Store tax impact (15% vs 30%), break-even analysis, gross margin targets, scaling economics, sensitivity analysis (2× API price, 80% vs 20% voice adoption). Owner: data-engineer + founder.
- `content/post-13-rir-rpe-autoregulation.md` — blog post «RIR і RPE: як навчитись вимірювати зусилля, а не тільки вагу» (12 хв читання). Охоплює: Zourdos RPE scale, proximity-to-failure та гіпертрофія (Schoenfeld 2017, Refalo 2022), miscalibration patterns у novices (Zourdos 2016), авторегуляція vs фіксовані % схеми (Helms 2016), роль Victor у калібрації RPE через velocity/timing data. clinical-safety PASS. Owner: content-engine.

---

## [0.38.0] — 2026-05-16

### Added
- `content/post-12-technique-over-intensity.md` — blog post «Техніка або інтенсивність: чому форма — це не безпека, а продуктивність» (13 хв читання). Охоплює: нейром'язову ефективність, механічний стимул vs зовнішнє навантаження, аналіз технічних помилок у squat/bench/deadlift, практичний фреймворк для self-coached атлетів. clinical-safety PASS. Owner: content-engine.
- `blog.html` — картка post-12 у grid (category: science).
- `STATUS.md` — рядок post-12 у content engine table.

## [0.37.0] — 2026-05-16

### Added
- `docs/INCIDENT_RESPONSE.md` — production-grade IR runbook (1486 рядків). P0–P3 severity matrix, GDPR 72h clock anchoring, 6 runbooks (data breach, outage, account takeover, SSO compromise, HMAC chain break, DDoS), communication templates, PIR template, drill schedule. SOC 2 evidence: CC7.2–CC7.5, CC9.2. Owner: security-engineer + devops-lead.

## [0.36.0] — 2026-05-16

### Added
- `docs/SSO_SCIM_IMPLEMENTATION.md` — повний технічний runbook для SAML 2.0 / OIDC + SCIM 2.0 provisioning (1154 рядки). Покриває: вибір протоколу, Okta/Azure AD/Google Workspace setup, multi-tenant isolation, role mapping, security controls (PKCE, SLO, cert rotation), customer onboarding checklist, операційний runbook, 13 audit event types, 13 GAP-items для engineering. Owner: enterprise-architect + security-engineer. SOC 2 evidence: CC6.1–CC6.3.
| **MAJOR** (`X.y.z`) | Брейкінг-чейндж, pivot бренду/скоупу. Не до релізу. |

---

## [0.39.0] — 2026-05-16

### Added
- `docs/DATA_MODEL.md` — multi-tenant Postgres data model (RLS-based). Covers: RLS vs schema-per-tenant decision with trade-off matrix, connection architecture (PgBouncer + SET LOCAL GUC pattern), core schema (tenants, users, user_health_profiles, workouts, meal_logs, tenant_wellness_summary MV, feature flags), RLS policies with fail-closed design, tenant isolation guarantees per layer, data classification (5 tiers with per-tenant AES-256-GCM + KMS for Sensitive class), privacy floor belt-and-suspenders enforcement (RLS + application), migration checklist + dangerous patterns, backup/restore isolation, GDPR DSAR + Right to Erasure flows. SOC 2 evidence: CC6.1, CC6.7, P series. Owner: enterprise-architect + compliance-officer + security-engineer.

---

## [Unreleased]

---

## [0.35.0] — 2026-05-16

### Why MINOR
- New enterprise compliance document: `docs/SOC2_READINESS.md` — full SOC 2 Type II readiness assessment across all 5 Trust Service Criteria

### Added
- [`docs/SOC2_READINESS.md`](docs/SOC2_READINESS.md) — SOC 2 Type II readiness: 5 TSC coverage (Security/Availability/Processing Integrity/Confidentiality/Privacy), 55-control gap analysis (~40% in place, 11 critical gaps, remediation roadmap), evidence collection process, audit firm engagement timeline, continuous compliance tooling. References DEC-030, AUDIT_LOG_SCHEMA.md, ENTERPRISE.md, SECURITY.md.

---

## [0.34.0] — 2026-05-16

### Why MINOR
- New blog post: `content/post-11-deload-weeks.md` — 13-min read on deload week science, supercompensation, and periodization
- Blog card added to `blog.html`

### Added
- [`content/post-11-deload-weeks.md`](content/post-11-deload-weeks.md) — deload weeks: накопичена втома, суперкомпенсація, ознаки потреби, плановий vs реактивний деload, що конкретно робити. clinical-safety PASS. sports-scientist review pending.
- `blog.html` — картка post-11 в grid (Спортивна наука, 13 хв)

---

## [0.33.0] — 2026-05-16

### Why MINOR
- New: enterprise admin dashboard mockup (`admin-dashboard.html`) — full tenant control panel for HR/People leads with privacy floor
- New: `docs/AUDIT_LOG_SCHEMA.md` — append-only HMAC-chained audit log architecture (SOC 2 / GDPR foundation)
- New: DEC-030 in DECISION_LOG codifies the audit log architecture
- iPhone app shipped multiple EAS updates: v0.20.0 → v0.21.0 (PR detection, streak grace per DEC-013, active meal slot highlight, rest timer ±30s controls)

### Added
- [`admin-dashboard.html`](admin-dashboard.html) — enterprise admin panel mockup: tenant switcher, 4 stat cards (Активовано 287/412, Тижнева 68%, W-ACSU 3.4, NPS 52), 30-day activity bar chart, department adoption breakdown, anonymous users table, privacy floor card emphasizing HR cannot see individual data
- [`docs/AUDIT_LOG_SCHEMA.md`](docs/AUDIT_LOG_SCHEMA.md) — full schema (HMAC chain, partition strategy), action taxonomy (auth/tenant/data/privacy/integration/system/support), retention policies (7y / 3y / 30d), access controls + break-glass procedure, export options (webhook, S3 sync, REST, SIEM)
- [`docs/DECISION_LOG.md`](docs/DECISION_LOG.md) — DEC-030 audit log architecture

### iPhone app (EAS Update preview branch)
- v0.20.0 — PR detection in Workout (heavier set → 🏆 alert + success haptic)
- v0.20.1 — PR within-session fix (Math.max(historical, currentSession), only alert if historical > 0); active meal slot highlight ("· зараз") based on current hour
- v0.20.2 — Streak grace per DEC-013: tolerates 1 missed day, breaks on 2 consecutive misses (Progress.tsx)
- v0.20.3 — Today.tsx streak fix to match Progress.tsx (consistency across screens)
- v0.21.0 — Rest timer ±30s adjustment buttons

### QA walks (qa-walker)
- P2 found and fixed: within-session double-PR re-trigger
- P1 found and fixed: streak contradicted DEC-013 (broke after 1 miss, not 2)

---

## [0.31.0] — 2026-05-15

### Why MINOR
- New blog post: `content/post-10-why-programs-assume-youre-always-ready.md`
- Fills a genuine content gap: posts 01/04/07 cover HRV, sleep mechanics, and progressive overload individually; no post addresses the core conceptual problem — why standardized programs can't account for variable recovery capacity — and how adaptive training solves it
- This is the differentiating argument behind FORM: Victor adapts because it reads the day's context, not only the week's schedule
- `blog.html` updated: 10th card added, hero counters updated
- Pending sports-scientist + clinical-safety review before publish (topic: training load management — no body image, no dietary claims)

### Added
- [`content/post-10-why-programs-assume-youre-always-ready.md`](content/post-10-why-programs-assume-youre-always-ready.md) — 13-min read, training-science; covers: the hidden assumption in every pre-written program, training load concept (acute:chronic workload ratio), autoregulation and RPE, what "adaptive" actually means vs. marketing buzzword (3 levels), practical self-coaching tips, FORM's approach via Victor, clinical-safety frontmatter note

### Changed
- [`blog.html`](blog.html) — post-10 card added (Спортивна наука, "Незабаром"); hero: 9→10 статей, 108→121 хв читання; card placed between post-04 and post-09
- [`STATUS.md`](STATUS.md) — post-10 added to content engine table; version → v0.31.0
- [`README.md`](README.md) — post-10 added to content tree; blog posts metric 9→10; version → v0.31.0
- [`VERSION`](VERSION) — 0.30.0 → 0.31.0

---

## [0.30.0] — 2026-05-15

### Why MINOR
- New HTML surface: `blog.html` — articles index for 9 existing blog posts
- Genuine gap: 9 blog posts existed as markdown drafts with no discoverable HTML entry point; journalists, investors, and beta candidates directed to the marketing site couldn't find the content engine
- Adds category filter (all / training science / industry analysis / founder notes / product), newsletter CTA with email capture, and honest status badges ("Незабаром" vs "Після Q3 interviews" for post-06 which requires real interview data)
- clinical-safety considerations: page shows titles + short neutral excerpts only; no body-image claims; post-04 framed around sleep/recovery mechanics without prescriptive language; no "never skip" phrasing — uses brand principle language

### Added
- [`blog.html`](blog.html) — articles index: 9 cards with category badges (Спортивна наука / Аналіз ринку / Нотатки засновника / Продукт / Дослідження), JS category filter, newsletter CTA with email capture, post-06 marked "Після Q3 interviews" (honest — requires real user interview data to complete), consistent editorial-brutalist design with grain overlay + acid lime + Fraunces/Manrope/JBMono

### Fixed
- [`STATUS.md`](STATUS.md) — "28 total documented decisions" → "29" (DEC-029 was added in v0.29.0 but counter not updated)
- [`README.md`](README.md) — VERSION comment in repo tree was hardcoded "0.22.0" → "(current)"; HTML page count 12 → 13; surface description updated

### Changed
- [`sitemap.xml`](sitemap.xml) — `blog.html` added (priority 0.8, changefreq weekly)
- [`VERSION`](VERSION) — 0.29.0 → 0.30.0
- [`STATUS.md`](STATUS.md) — blog.html added to HTML pages table; version → v0.30.0
- [`README.md`](README.md) — blog.html added to repo tree; metrics updated

---

## [0.29.0] — 2026-05-15

### Why MINOR
- New document: `docs/GROWTH_LOOPS.md` — viral/organic growth mechanics
- Fills a genuine gap: BETA_RECRUITING.md covers how to get 200 beta users; RETENTION_PLAYBOOK.md covers D0-D90 retention; but no document defines the specific viral mechanics (share cards, referral system, streak milestones) the Founding Engineer needs to build
- 5 loops designed with DEC-002 (no in-app social) and DEC-016 (no free tier) constraints baked in
- DEC-029 logged: growth loops policy — external sharing only, clinical-safety gate on all share content

### Added
- [`docs/GROWTH_LOOPS.md`](docs/GROWTH_LOOPS.md) — 5 growth loops: Loop 1 Form Check Share (CV result → share card → organic acquisition), Loop 2 Streak Share (milestone → share), Loop 3 Victor WOM (coaching quality drives word of mouth), Loop 4 Referral Program (formal, post-launch M4-M5), Loop 5 Beta FOMO (waitlist referral mechanics); clinical-safety allowed/blocked content tables; FE implementation priority P0-P4; PostHog events for tracking; K-factor targets; 5 open questions for future decisions
- `docs/DECISION_LOG.md` — DEC-029: growth loops sharing policy (external only, clinical-safety gate, allowed vs. blocked share content)

### Changed
- [`VERSION`](VERSION) — 0.28.0 → 0.29.0
- [`STATUS.md`](STATUS.md) — GROWTH_LOOPS.md added to docs table; doc count 55+→56+; decisions 28→29; version → v0.29.0
- [`README.md`](README.md) — GROWTH_LOOPS.md added to docs tree; doc count 55+→56+; decisions 28→29; version → v0.29.0

---

## [0.28.0] — 2026-05-15

### Why MINOR
- New document: `docs/VICTOR_PROMPT_GUIDE.md` — LLM implementation guide for Victor
- Fills a genuine gap: BRAND.md has tone rules, post-09 explains the challenge, PRODUCT_SPEC.md lists Victor chat as P1 — but no document tells the FE HOW to actually build Victor's LLM layer
- Critical for FE hire: demonstrates technical depth to candidates and lets them start building Victor correctly from day 1
- 9 sections, 22 scenario examples with concrete UA/EN responses, full clinical-safety checklist embedded
- clinical-safety: reviewed — hard blocks defined in Layer 2, ED-flag mode documented in §7, distress response placeholder explicitly marked TBD (not fabricated), jurisdiction-appropriate mental health resources flagged as pending before ship

### Added
- [`docs/VICTOR_PROMPT_GUIDE.md`](docs/VICTOR_PROMPT_GUIDE.md) — LLM implementation guide: 5-layer system prompt architecture (role + safety constraints + context injection + format + user message), TypeScript context schema with all runtime fields (HRV, sleep, streak, ED-flag, form score, etc.), 4 tone modes with UA/EN phrase examples (smart_direct/mindful/cold/coach), response format rules (length per context, forbidden constructions), 22 scenario library with exact Victor phrasing (Day 1 / PR / mediocre session / 1-missed / 3-missed / week-missed / injury / plateau / food no-flag / food ED-flag / low HRV / high HRV / unmotivated / 30-streak / comparison / faster results / deload start / post-deload / supplements / 2-weeks-away / pre-event / out-of-scope), ED-flag restricted mode (what changes, invisible flag rationale, distress response), Victor silence rules (when NOT to speak, 1/day notification cap), 7 open questions for FE, clinical-safety embedded checklist

### Changed
- [`VERSION`](VERSION) — 0.27.0 → 0.28.0
- [`STATUS.md`](STATUS.md) — VICTOR_PROMPT_GUIDE.md added to docs table; doc count 54+→55+; version → v0.28.0
- [`README.md`](README.md) — VICTOR_PROMPT_GUIDE.md added to docs tree; doc count 54+→55+; version → v0.28.0

---

## [0.27.0] — 2026-05-15

### Why MINOR
- New document: `docs/RETENTION_PLAYBOOK.md` — Day 0/7/30/90 retention mechanics for FORM beta and beyond
- Fills genuine gap: ANALYTICS_SETUP.md tracks what to measure, BETA_PLAYBOOK.md handles TestFlight ops, COMMUNITY.md handles Slack/Discord, but nothing defined what happens inside the product when users show churn signals
- Key decisions: weekly streaks (not daily — respects programmed rest days), 3-attempt re-engagement cap then silence, Pause always before Cancel (3-click max), Victor memory arc (Week 1 → Month 3), clinical-safety checklist for all retention copy
- 11 sections, 4 open questions for FE + design hire, explicit clinical-safety constraints embedded throughout
- clinical-safety: reviewed — retention copy constraints documented as hard blocks/approved/gray-zone checklist

### Added
- [`docs/RETENTION_PLAYBOOK.md`](docs/RETENTION_PLAYBOOK.md) — 560-line retention strategy: critical windows (D0/D1-3/D7/D14/D30/D90) with Victor example phrases, notification types + cadence (1/day cap, user-controlled windows), weekly streak mechanics (session-based, grace week, "не пропускай два поспіль" embedded), Victor personalization arc, 3-stage re-engagement protocol (passive/at-risk/churned, max 3 attempts), Pause>Cancel design (3-click, 2Q exit survey), clinical-safety constraints as hard-blocks/approved/gray-zone checklist, metrics framework (no fabricated targets), integration map, 4 open questions

### Changed
- [`VERSION`](VERSION) — 0.26.0 → 0.27.0
- [`STATUS.md`](STATUS.md) — RETENTION_PLAYBOOK.md added to docs table; doc count 53+→54+; version → v0.27.0
- [`README.md`](README.md) — RETENTION_PLAYBOOK.md added to docs tree; doc count 53+→54+; version → v0.27.0

---

## [0.26.0] — 2026-05-15

### Why MINOR
- New content: `content/newsletter-03.md` — Issue 03 "Building Victor: What it takes to design an AI coaching voice"
- Extends newsletter content engine to 3 issues; natural third chapter after "Why we're building FORM" and "What FORM does/doesn't do"
- Victor voice demonstrated via side-by-side example (competitor vs Victor on skipped workout) — clinical-safety PASS
- brand-voice + clinical-safety reviewed: no food/body-image/injury/mental health content

### Added
- [`content/newsletter-03.md`](content/newsletter-03.md) — Issue 03 "Building Victor" (~720 words): problem statement (app praising on rest days), two bad coaching defaults (drill sergeant vs cheerleader), "не пропускай два поспіль" philosophy explained, side-by-side skipped-workout example (competitor vs Victor), clinical-safety as design constraint framed honestly, beta CTA · clinical-safety PASS

### Changed
- [`VERSION`](VERSION) — 0.25.0 → 0.26.0
- [`STATUS.md`](STATUS.md) — newsletter-03.md added to content table; version → v0.26.0
- [`README.md`](README.md) — newsletter-03.md added to content tree; version → v0.26.0

---

## [0.25.0] — 2026-05-15

### Why MINOR
- New document: `docs/LINKEDIN_PLAYBOOK.md` — LinkedIn strategy for FORM's pre-launch phase
- Fills genuine gap: `content/linkedin-posts.md` has 12 posts ready but no strategic framework for deploying them; `docs/TWITTER_PLAYBOOK.md` exists for X but LinkedIn is a distinct platform (different audience, algorithm, format, value proposition for FORM)
- LinkedIn is FORM's primary hiring and investor-visibility channel — engineers/investors are on LinkedIn, not necessarily on Fitness Twitter
- 15 sections: LinkedIn vs X comparison table, profile setup (headline/about/featured/Creator Mode), company page recommendation (defer pre-beta), 4 content pillars, 3-tier connection strategy, LinkedIn for hiring (InMail templates, FE+DL cadence), LinkedIn for investor relationships (what VCs check, warm intros), build-in-public translation guide (thread vs single post, repurposing blogs), 2-3x/week cadence with day/time table, voice rules comparison table, metrics (SSI, engagement rate, hiring funnel UTMs), honest limitations (not a consumer growth channel), 10-item quick-start checklist, integration map to 9 docs, 4 open questions
- clinical-safety: not required (no food, body image, mental health, or injury content)

### Added
- [`docs/LINKEDIN_PLAYBOOK.md`](docs/LINKEDIN_PLAYBOOK.md) — 15-section LinkedIn strategy: X vs LinkedIn comparison table, profile optimization (headline variants, About section draft, Creator Mode setup), company-page recommendation (defer until post-beta), 4 content pillars with post assignments from linkedin-posts.md, 3-tier connection strategy (Tier 1: 50 connections week 1, Tier 2: pre-outreach warm-up, Tier 3: follow only), hiring InMail templates + cadence for FE + Design Lead, investor visibility playbook (what VCs check before first meeting), build-in-public translation guide (what works/doesn't from X, repurposing blog content), 2-3x/week cadence with day/time table, voice rules table (LinkedIn vs X vs newsletter), metrics framework (SSI, engagement rate, profile views, hiring funnel UTMs), honest limitations (LinkedIn won't drive App Store installs), 10-item quick-start checklist (doable today without domain or live product), integration map to 9 companion docs, 4 open questions (network size, language split UA/EN, LinkedIn newsletter vs Beehiiv, existing network segmentation)

### Changed
- [`VERSION`](VERSION) — 0.24.0 → 0.25.0
- [`STATUS.md`](STATUS.md) — LINKEDIN_PLAYBOOK.md added to docs table; TL;DR → v0.25.0; doc count 51+→52+; releases 40+→41+; version header + footer → v0.25.0
- [`README.md`](README.md) — LINKEDIN_PLAYBOOK.md added to docs tree; doc count 51+→52+; version refs → v0.25.0

---

## [0.24.0] — 2026-05-15

### Why MINOR
- New document: `docs/INVESTOR_PIPELINE.md` — investor CRM and relationship management framework
- Fills genuine gap: INVESTOR_OUTREACH.md (target lists + cold templates), SEED_NARRATIVE.md (verbal pitch), DD_FAQ.md (due diligence Q&A), and DATA_ROOM.md (materials) all exist — but no doc defines HOW to manage the pipeline itself: stage definitions, CRM schema, follow-up cadence, signals of interest, post-meeting ritual, close mechanics
- 15 sections: 7-stage pipeline model (S0–S6), Google Sheets CRM schema (15 columns + 3 tabs), cadence rules per stage transition, warm intro templates (request + forwardable blurb), 30-min pre-meeting prep ritual, post-meeting recap template, green/yellow/red signal taxonomy, anti-pattern table, realistic raise timeline, integration map to 7 other docs
- clinical-safety not required (no food/body/injury/mental health content)

### Added
- [`docs/INVESTOR_PIPELINE.md`](docs/INVESTOR_PIPELINE.md) — investor relationship management: 7-stage pipeline (S0 Listed → S6 Closed/Declined), CRM spreadsheet schema (15 columns, color logic, 3 tabs), stage-by-stage cadence rules, warm intro request + forwardable blurb templates, 30-min pre-meeting prep ritual, post-meeting recap email template + log checklist, green/yellow/red interest signals, park vs. hard-decline logic, monthly investor updates strategy (links INVESTOR_UPDATE_TEMPLATE.md), SAFE mechanics + round close checklist, 12-item anti-pattern table, realistic M0→M5 timeline, 14-doc integration map, 4 open questions for counsel + product-strategist

### Changed
- [`VERSION`](VERSION) — 0.23.0 → 0.24.0
- [`STATUS.md`](STATUS.md) — INVESTOR_PIPELINE.md added to docs table; TL;DR → v0.24.0; doc count 50+→51+; releases 39+→40+; version header + footer → v0.24.0
- [`README.md`](README.md) — INVESTOR_PIPELINE.md added to docs tree; doc count 50+→51+; version refs → v0.24.0

---

## [0.23.0] — 2026-05-15

### Why MINOR
- New document: `docs/TWITTER_PLAYBOOK.md` — pre-launch X/Twitter growth strategy
- Fills genuine gap: `content/twitter-launch-thread.md` covers launch day only; nothing covered the 3-4 month pre-launch build-in-public window
- George can start today: account setup, handle rationale, 3 bio variants, pinned tweet concept, 5 content pillars with real example drafts, weekly cadence, engagement strategy, beta recruiting via X (connects to RESEARCH_SCREENER.md), voice rules for X, metrics framework, honest limitations, quick-start checklist
- clinical-safety not required (no food/body/injury/mental health content)

### Added
- [`docs/TWITTER_PLAYBOOK.md`](docs/TWITTER_PLAYBOOK.md) — 555-line pre-launch X strategy: account setup (handle, 3 bio variants, pinned tweet), Phase 1/2/3 content phases, 5 content pillars with example tweets, weekly cadence (5-7 posts/week), build-in-public formula (what to share vs not), engagement map (5 account categories), beta recruiting via X → RESEARCH_SCREENER.md, voice rules for X vs LinkedIn vs newsletter, launch-week coordination → twitter-launch-thread.md, metrics (engagement rate, UTM attribution, beta recruits), honest limitations (X won't scale installs, build-in-public shelf life), quick-start checklist (8 tasks for today)

### Changed
- [`VERSION`](VERSION) — 0.22.0 → 0.23.0
- [`STATUS.md`](STATUS.md) — TWITTER_PLAYBOOK.md added to docs table; TL;DR → v0.23.0; doc count 49+→50+; releases 38+→39+; version header + footer → v0.23.0
- [`README.md`](README.md) — TWITTER_PLAYBOOK.md added to docs tree; doc count 49+→50+; version refs → v0.23.0

---

## [0.22.0] — 2026-05-15

### Why MINOR
- New document: `docs/BETA_FEEDBACK_PROTOCOL.md` — operational protocol for processing beta user feedback
- Closes genuine gap: BETA_PLAYBOOK.md handles TestFlight operations, BETA_RECRUITING.md handles acquisition, COMMUNITY.md handles Slack/Discord — but no doc defines what happens AFTER feedback arrives
- 12 sections: channels, taxonomy (7 categories + P0–P3 severity), daily intake process, safety escalation, weekly synthesis, monthly report template, feedback→decision filter, Victor voice tracker, privacy rules, integration map, open questions
- clinical-safety not required (no food/body/injury/mental health content — safety escalation paths reference clinical-safety but the doc itself is operational procedure)

### Added
- [`docs/BETA_FEEDBACK_PROTOCOL.md`](docs/BETA_FEEDBACK_PROTOCOL.md) — intake, taxonomy (7 categories, P0–P3 severity), daily triage process, safety escalation path, weekly synthesis rhythm, monthly report template, feedback→decision filter with 5-step logic gate, Victor voice tracker, privacy rules, integration with COMMUNITY/RESEARCH/ANALYTICS_SETUP/DECISION_LOG/PRODUCT_SPEC/SUPPORT, 3 open questions for Founding Engineer + LEGAL.md followup

### Changed
- [`VERSION`](VERSION) — 0.21.0 → 0.22.0
- [`STATUS.md`](STATUS.md) — BETA_FEEDBACK_PROTOCOL.md added to docs table; TL;DR → v0.22.0; doc count 48+→49+; releases 37+→38+; version header + footer → v0.22.0
- [`README.md`](README.md) — BETA_FEEDBACK_PROTOCOL.md added to docs tree; doc count 48+→49+; version refs → v0.22.0

---

## [0.21.0] — 2026-05-15

### Why MINOR
- New document: `docs/RESEARCH_SCREENER.md` — practical recruiting toolkit for 30 pre-product user interviews
- Fills genuine operational gap: RESEARCH.md has the 45-min interview script and synthesis framework, but no screener survey, no recruiting messages, no tracker template, no payment flow
- George can use this TODAY to start recruiting — does not require site to be live
- 12 sections: screener questions with pass/fail criteria, channel-specific messages (Reddit, X, Telegram, LinkedIn), email templates, Google Sheets tracker columns, gift card payment flow (Amazon/Apple/Monobank/PayPal), anti-bias checklist, weekly velocity targets
- clinical-safety not required (no food/body/injury/mental health content; ethics reference deferred to RESEARCH.md §9 which already has the safeguards)

### Added
- [`docs/RESEARCH_SCREENER.md`](docs/RESEARCH_SCREENER.md) — user research recruiting toolkit: 7-question screener (Q1–Q7 with pass/reject/maybe criteria), recruiting messages EN+UA for Reddit/X/Telegram/LinkedIn, Reddit post templates for r/fitness + r/xxfitness, X DM flow with search operators, interview tracker (17-column Google Sheets spec), scheduling email sequence (confirmation + 24h + 1h reminders), gift card payment flow (Amazon/Apple/Monobank/PayPal), rejection template, anti-bias pre-call checklist, weekly velocity targets (8→20→30 calls), 3 open questions for LEGAL.md followup

### Changed
- [`VERSION`](VERSION) — 0.20.0 → 0.21.0
- [`STATUS.md`](STATUS.md) — RESEARCH_SCREENER.md added to docs table; TL;DR → v0.21.0; doc count 47+→48+; releases 36+→37+; version header + footer → v0.21.0; timeline extended to v0.21.0
- [`README.md`](README.md) — RESEARCH_SCREENER.md added to docs tree; doc count 47+→48+; version refs → v0.21.0; date → 15 травня 2026

---

## [0.20.0] — 2026-05-15

### Why MINOR
- New document: `docs/DD_FAQ.md` — investor due diligence FAQ prep
- Completes the investor preparation toolkit: PITCH.md (deck) + SEED_NARRATIVE.md (call script) + INVESTOR_OUTREACH.md (outreach) + FINANCIALS.md (model) + DATA_ROOM.md (materials) were all present, but no DD Q&A prep
- Covers 7 blocks: Team, Market, Product/Tech, Financials, Execution risks, Round terms, Data Room
- All answers follow Principle #1: no fabricated numbers; honest about unknowns; confident on known facts
- Fix: STATUS.md table had stale decision count (27 → 28) — DEC-028 was logged in v0.18.0 but table row wasn't updated
- Fix: STATUS.md and README.md dates updated to 15 травня 2026

### Added
- [`docs/DD_FAQ.md`](docs/DD_FAQ.md) — 7-block investor DD FAQ: Team (co-founder risk, Ukraine risk), Market (TAM, competition, Apple/Google risk), Product/Tech (CV accuracy, privacy, LLM costs, iOS-only rationale), Financials (subscription model, pricing, break-even, fund use), Execution risks (3 biggest risks, kill criteria, Ukraine), Round (amount, terms, what beyond money), Data Room index. Companion to SEED_NARRATIVE.md, PITCH.md, FINANCIALS.md, DATA_ROOM.md.

### Changed
- [`VERSION`](VERSION) — 0.19.0 → 0.20.0
- [`STATUS.md`](STATUS.md) — DD_FAQ.md added to docs table; DECISION_LOG.md row fixed 27→28 decisions; date → 15 травня 2026; TL;DR → v0.20.0; doc count 46+→47+; releases 35+→36+; version header + footer → v0.20.0; timeline extended to v0.20.0
- [`README.md`](README.md) — DD_FAQ.md added to docs tree; doc count 46+→47+; version refs → v0.20.0; date → 15 травня 2026

---

## [0.19.0] — 2026-05-14

### Why MINOR
- New document: `docs/SEED_NARRATIVE.md` — verbal pre-seed pitch script
- Complements PITCH.md (slide structure) and INVESTOR_OUTREACH.md (outreach templates)
- Fills genuine gap: George needs a spoken narrative for angel calls, not just deck slides
- Covers the specific challenge of raising pre-product pre-revenue: 6 narrative blocks, objection playbook, audience variants (angel/seed fund/accelerator/strategic)
- Fix: STATUS.md and README.md had stale decision count (27 → 28, DEC-028 was added in v0.18.0)
- Fix: STATUS.md timeline was missing v0.16.0–v0.19.0 milestones (stopped at v0.15.0)

### Added
- [`docs/SEED_NARRATIVE.md`](docs/SEED_NARRATIVE.md) — 6-block verbal pitch script for angel/pre-seed investor calls: Block 0 (opening line), Block 1 (personal story), Block 2 (product — 3 things), Block 3 (timing / tech window), Block 4 (market + model), Block 5 (honest state + what's next), Block 6 (ask). Objection playbook: 6 common objections with verbatim responses. Tonal rules table (do/don't). Audience variants table (angel-tech / angel-fitness / seed fund / strategic / accelerator). Companion references to PITCH.md, INVESTOR_OUTREACH.md, FINANCIALS.md.

### Changed
- [`VERSION`](VERSION) — 0.18.0 → 0.19.0
- [`STATUS.md`](STATUS.md) — SEED_NARRATIVE.md added to docs table; TL;DR → v0.19.0; doc count 45+→46+; releases 34+→35+; version header + footer → v0.19.0; decision count 27→28; timeline extended to v0.19.0
- [`README.md`](README.md) — SEED_NARRATIVE.md added to docs tree; doc count 45+→46+; version refs → v0.19.0; decision count 27→28

---

## [0.18.0] — 2026-05-14

### Why MINOR
- New document: `docs/HIRING_GUIDE.md` — platform-by-platform JD posting guide
- Closes the "Apply Founding Engineer + Design Lead JDs on platforms" gap from STATUS.md "What's next (immediate)"
- Prior art: JD markdown files in `hiring/` + designed careers page (`jobs.html`) — but no concrete posting instructions for George
- New: DEC-028 (platform priority for hiring) in DECISION_LOG.md
- Fix: STATUS.md had wrong version reference (v0.15.0) — corrected to v0.18.0

### Added
- [`docs/HIRING_GUIDE.md`](docs/HIRING_GUIDE.md) — 8-section platform-by-platform hiring guide: role/platform priority table (FE: Djinni > DOU > Wellfound > LinkedIn; Design Lead: LinkedIn > Wellfound > Djinni > Behance); per-platform step-by-step setup (Djinni account, DOU jobs + forum, Wellfound company profile, LinkedIn personal post vs. paid job post); CV screening red/green flags by role; 48-hour response protocol; rejection email template; interview logistics (45-min structure); 5-week timeline; $0-150 budget breakdown; pre-posting checklist; tracking sheet columns

### Changed
- [`VERSION`](VERSION) — 0.17.0 → 0.18.0
- [`STATUS.md`](STATUS.md) — HIRING_GUIDE.md у docs table; TL;DR → v0.18.0; doc count 44+→45+; releases 33+→34+; version header + footer → v0.18.0
- [`README.md`](README.md) — HIRING_GUIDE.md у docs tree; doc count 44+→45+; version refs → v0.18.0
- [`docs/DECISION_LOG.md`](docs/DECISION_LOG.md) — DEC-028 added (platform priority: Djinni-first для FE, LinkedIn-first для Design Lead; $0 start, boost as fallback)

---

## [0.17.0] — 2026-05-14

### Why MINOR
- New document: `docs/ADVISORY_BOARD.md` — full advisory board recruitment playbook
- Closes the "Get sport-science advisor signed" gap in the roadmap: prior art was jd-sport-science-advisor.md (what the role is) but no plan for how to find and sign advisors
- First document that defines the full advisory strategy: 4 profiles, compensation structure, outreach templates, trial period protocol, agreement terms, timeline

### Added
- [`docs/ADVISORY_BOARD.md`](docs/ADVISORY_BOARD.md) — 8-section advisory board playbook: why we need advisors (3 risk categories: science drift, clinical liability, credibility signals); 4 advisor profiles (sport science HIGH priority, clinical/psychology HIGH priority, CV/ML MEDIUM, industry LOW/post-seed) each with profile criteria, anti-profile, where to find, what we ask, why they'd say yes; compensation table (0.10–0.25% equity per slot, 2yr monthly vesting no cliff, no cash pre-seed); warm intro + cold outreach templates with rules; 20-minute first call structure with red flags; 30-day trial protocol; advisory agreement key terms (FAST Agreement reference); timeline with hard deadline (Slot 1 + 2 signed before any food/body features ship — clinical-safety requirement)

### Changed
- [`VERSION`](VERSION) — 0.16.0 → 0.17.0
- [`README.md`](README.md) — ADVISORY_BOARD.md у docs tree; doc count 43+→44+; version refs → v0.17.0
- [`STATUS.md`](STATUS.md) — ADVISORY_BOARD.md у docs table; version → v0.17.0; TL;DR → v0.17.0

---

## [0.16.0] — 2026-05-14

### Why MINOR
- Two new content assets: `content/newsletter-01.md` + `content/newsletter-02.md`
- First full newsletter issues ready to publish on Beehiiv — closes the "перші 2 пости" gap from NEWSLETTER.md
- Closes the last "Newsletter setup плюс перші 2 пости" item from the roadmap; prior work (NEWSLETTER.md) was platform setup only
- Both passed clinical-safety review (PASS WITH NOTES: food logging feature needs ED screening gate; Victor correction loop needs ease-off exit path — product-level items logged, not newsletter blockers)

### Added
- [`content/newsletter-01.md`](content/newsletter-01.md) — Issue 01 "Why we're building FORM": personal founder letter (400–500 words), Victor off-duty tone, opens with in-person coach moment, draws category gap between tracker and trainer, lists what newsletters will and won't contain, beta CTA. clinical-safety: PASS.
- [`content/newsletter-02.md`](content/newsletter-02.md) — Issue 02 "What FORM actually does (and what it doesn't)": structured breakdown of 3 core features (CV form on 8 lifts, HRV-adjusted programming, Victor voice) + 5 explicit won't-dos (no medical diagnoses, no calorie surveillance, no before/after, no guarantees, no data monetization) + honest CV accuracy uncertainty section. clinical-safety: PASS.

### Changed
- [`VERSION`](VERSION) — 0.15.0 → 0.16.0

### Notes (product action items from clinical-safety)
- Food logging feature requires SCOFF or equivalent ED screening gate before a user can enable it
- Victor correction loop needs a documented ease-off exit path for users who re-attempt compulsively

---

## [0.15.0] — 2026-05-14

### Why MINOR
- New HTML surface: `jobs.html` — dedicated careers page for FE + Design Lead hiring
- Translates "JDs ready to post" into a shareable URL the founder can send to candidates and link from LinkedIn
- Closes the gap between having JDs in markdown and having a designed hiring entry point

### Added
- [`jobs.html`](jobs.html) — 5-section careers page: Why FORM (4 values: mission, craft, equity, timing) / Open roles (FE iOS + Design Lead with requirements, hard-noes, compensation pills) / What We Value (6 principles from HIRING.md) / Interview Process (6 steps, honest timeline) / Compensation tables (salary + equity + benefits). Links to full JDs in `hiring/`. CTA: hiring@form.coach. Added to sitemap.xml with priority 0.8.

### Changed
- [`VERSION`](VERSION) — 0.14.0 → 0.15.0
- [`sitemap.xml`](sitemap.xml) — jobs.html added (weekly, priority 0.8)
- [`README.md`](README.md) — jobs.html in repo tree; HTML count 11→12; version refs → v0.15.0
- [`STATUS.md`](STATUS.md) — jobs.html in pages table; version → v0.15.0; TL;DR → v0.15.0; "Apply JDs publicly" gap addressed

---

## [0.14.0] — 2026-05-14

### Why MINOR
- New HTML surface: `investors.html` — standalone investor one-pager page
- Closes the "one-pager here: [link]" gap referenced throughout INVESTOR_OUTREACH.md and FIRST_30_DAYS.md
- Gives investors a single shareable URL with the full pitch summary, unit economics, honest traction, ask breakdown, and links to all data-room docs

### Added
- [`investors.html`](investors.html) — 9-section investor overview page: The ask / The gap (problem + competition table) / Product (4 layers: CV + HRV + Victor + nutrition) / Market (SAM 12M + SOM $25M ARR + why-now) / Traction (honest: what exists vs. committed milestones) / Business model + unit economics ($9/$19, LTV/CAC 4–7×, gross margin 78–84%) / Team (George + AI-agent model) / Risks (honest table) / Next steps (invest@form.coach + links to PITCH/FINANCIALS/DATA_ROOM/TECHNICAL/GitHub). Marked `noindex` — investor-direct link only.

### Changed
- [`VERSION`](VERSION) — 0.13.0 → 0.14.0
- [`README.md`](README.md) — investors.html in repo tree; HTML count 10→11; releases 29+→30+; version refs → v0.14.0
- [`STATUS.md`](STATUS.md) — investors.html in pages table; version → v0.14.0; TL;DR → v0.14.0

---

## [0.13.0] — 2026-05-14

### Why MINOR
- Новий документ: `docs/ASSUMPTIONS_REGISTER.md` — реєстр 30 product/market/tech передумов до research sprint
- Перший документ що явно фіксує що ми вважаємо правдивим ДО 30 інтерв'ю, з confidence levels і pivot criteria
- Закриває gap між RESEARCH.md (як проводити) і DECISION_LOG.md (що вирішено): тепер є місце де відстежується що ще невідомо

### Added
- [`docs/ASSUMPTIONS_REGISTER.md`](docs/ASSUMPTIONS_REGISTER.md) — 30 numbered assumptions у 6 категоріях (User Pain Points, Product Beliefs, Pricing, Technical Feasibility, Market & Competition, Go-to-Market); confidence levels High/Medium/Low; status ● Untested → ✓/✗/~/? після research; test column → конкретні interview questions (Q12, Q15, Q17, Q19, Q23); wrong-if + pivot-if для кожної; review schedule (10/30 interviews, Beta W1/W4); cross-reference таблиця до RESEARCH, PERSONAS, OKRS, ANALYTICS_SETUP, FINANCIALS

### Changed
- [`docs/DECISION_LOG.md`](docs/DECISION_LOG.md) — додано DEC-025 (FIRST_30_DAYS execution framework), DEC-026 (PostHog EU Cloud analytics), DEC-027 (Assumptions Register як research gate); decision count 24 → 27
- [`VERSION`](VERSION) — 0.12.0 → 0.13.0
- [`README.md`](README.md) — ASSUMPTIONS_REGISTER.md у repo tree; doc count 41+→42+; decision count 24→27; version refs → v0.13.0
- [`STATUS.md`](STATUS.md) — ASSUMPTIONS_REGISTER.md у docs-таблиці; decision count → 27; TL;DR → v0.13.0

---

## [0.12.0] — 2026-05-14

### Why MINOR
- Новий документ: `docs/FIRST_30_DAYS.md` — 4-тижневий execution playbook
- Перший документ що перетворює 40+ стратегічних доків в операційний план дій
- Синтезує RESEARCH.md + INVESTOR_OUTREACH.md + DEPLOYMENT.md + CONTENT_CALENDAR.md + HIRING.md в єдину послідовність

### Added
- [`docs/FIRST_30_DAYS.md`](docs/FIRST_30_DAYS.md) — Week 0 (інфраструктура: domain, email, tools), W1 (Twitter, Show HN, перший newsletter), W2 (research sprint — recruiting DM templates, interview notes template), W3 (investor CRM setup, warm intro scripts, FE JD distribution channels + tech assessment), W4 (синтез, synthesis checklist, 8-gate checkpoint), troubleshooting таблиця

### Changed
- [`VERSION`](VERSION) — 0.11.0 → 0.12.0
- [`README.md`](README.md) — FIRST_30_DAYS.md у repo tree; doc count 40+→41+; version refs → v0.12.0
- [`STATUS.md`](STATUS.md) — FIRST_30_DAYS.md у docs-таблиці; версія → v0.12.0

---

## [0.11.0] — 2026-05-14

### Why MINOR
- Новий документ: `docs/BETA_RECRUITING.md` — повний playbook рекрутингу 200 бета-юзерів
- Закриває явний gap: `BETA_PLAYBOOK.md` покриває операції, але не пошук учасників
- 10 розділів: target profile, канали (3 tier), scoring, invite-sequence, incentives, timeline, anti-patterns

### Added
- [`docs/BETA_RECRUITING.md`](docs/BETA_RECRUITING.md) — Beta user recruiting: мета 200 qualified users, Tier 1-3 канали (особиста мережа / спільноти / контент), scoring rubric (15 балів), форма заявки (+3 нових питання), invite email sequence, incentive structure, timeline Wave 0→2, метрики тижня, anti-patterns, messaging quick-ref (UA+EN)

### Changed
- [`VERSION`](VERSION) — 0.10.0 → 0.11.0
- [`STATUS.md`](STATUS.md) — BETA_RECRUITING.md у docs-таблиці; doc count 39+→40+; версія → v0.11.0
- [`README.md`](README.md) — BETA_RECRUITING.md у repo tree; doc count оновлено; version refs → v0.11.0

---

## [0.10.0] — 2026-05-14

### Why MINOR
- Новий документ: `docs/ANALYTICS_SETUP.md` — повний instrumentation guide для Founding Engineer
- Companion до `PRODUCT_SPEC.md`: що будувати → як вимірювати
- 65 typed events, W-ACSU computation, PostHog iOS SDK setup, privacy implementation, A/B test guide

### Added
- [`docs/ANALYTICS_SETUP.md`](docs/ANALYTICS_SETUP.md) — PostHog setup (iOS SDK + Cloudflare EU), 65-event taxonomy з typed enums (no PII), W-ACSU SQL + PostHog formula, AARRR funnel setup, user super-properties, Core dashboard blueprint (11 panels), A/B test flag setup, privacy opt-out implementation, QA checklist per sprint, 3-environment separation (dev/beta/prod)

### Changed
- [`VERSION`](VERSION) — 0.9.0 → 0.10.0
- [`STATUS.md`](STATUS.md) — ANALYTICS_SETUP.md у docs-таблиці; doc count 38+→39+; версія → v0.10.0
- [`README.md`](README.md) — ANALYTICS_SETUP.md у repo tree; doc count оновлено; version refs → v0.10.0

---

## [0.9.0] — 2026-05-14

### Why MINOR
- Новий документ: `docs/PRODUCT_SPEC.md` — повна технічна специфікація iOS-продукту для Founding Engineer
- Перший документ що синтезує FLOWS + TECHNICAL + API в єдиний "що будуємо" артефакт
- Інключає 12-тижневий sprint plan, acceptance criteria, clinical-safety checklist, CV pipeline spec

### Added
- [`docs/PRODUCT_SPEC.md`](docs/PRODUCT_SPEC.md) — iOS product specification: MVP scope (P0/P1/P2), 7 screen specs з acceptance criteria, Victor AI implementation guide, CV pipeline для 8 ліфтів з accuracy targets, HealthKit integration, StoreKit 2 config, 12-тижневий sprint plan (Sprint 0–6 → TestFlight), NFR (performance/battery/privacy/a11y), pre-merge checklist, clinical-safety checklist per sprint

### Changed
- [`VERSION`](VERSION) — 0.8.0 → 0.9.0
- [`STATUS.md`](STATUS.md) — PRODUCT_SPEC.md додано до docs-таблиці; doc count 37+→38+; версія → v0.9.0
- [`README.md`](README.md) — PRODUCT_SPEC.md у repo tree; doc count оновлено; version refs → v0.9.0

---

## [0.8.0] — 2026-05-14

### Why MINOR
- Новий операційний документ: `docs/COMMUNITY.md` — beta community setup
- Закриває явний gap у STATUS.md ("Beta community (Discord/Telegram)" — not built)
- Підтримує BETA_PLAYBOOK.md (Slack-invite згадувався без деталей реалізації)
- Модераційні правила (секція 8) відповідають clinical-safety defaults

### Added
- [`docs/COMMUNITY.md`](docs/COMMUNITY.md) — Slack (closed beta ≤200) + Discord (post-launch); 11 каналів; ролі; welcome flow; тижневий ритм; feedback збір; moderation guidelines з ED/mental-health defaults; метрики здоров'я спільноти; pre-launch checklist; план переходу Slack→Discord з OG Beta privileges

### Changed
- [`STATUS.md`](STATUS.md) — beta community gap позначений addressed; COMMUNITY.md у docs-таблиці; timeline оновлено; версія → v0.8.0
- [`README.md`](README.md) — COMMUNITY.md у repo tree; doc count 36+→37+; releases 23+→24+; version refs → v0.8.0

---

## [0.7.0] — 2026-05-14

### Why MINOR
- New content asset: `content/linkedin-posts.md` — 12 LinkedIn posts across 4 categories
- Closes social media gap (Twitter + Reddit existed; LinkedIn was missing)
- LinkedIn is the primary channel for hiring, investor outreach, and user interviews

### Added
- [`content/linkedin-posts.md`](content/linkedin-posts.md) — 12 ready-to-post LinkedIn posts: 3 founder-story, 4 product/insight, 3 hiring, 2 build-in-public templates. EN/UA mix. 12-week posting calendar + hashtag bank.

### Changed
- [`STATUS.md`](STATUS.md) — post-09 added to content table (was missing); linkedin-posts.md added to content table; version refs bumped to v0.7.0; TL;DR updated (23+ releases)

---

## [0.6.0] — 2026-05-14

### Why MINOR
- New HTML surface: `press.html` — full press kit page
- New doc: `docs/NEWSLETTER.md` — Beehiiv setup guide, first 2 post concepts

### Added
- [`press.html`](press.html) — 9-section HTML press kit.
- [`docs/NEWSLETTER.md`](docs/NEWSLETTER.md) — newsletter setup guide.

### Changed
- [`README.md`](README.md) — press.html added to quick-nav; HTML count 9→10; doc count 35+→36+
- [`sitemap.xml`](sitemap.xml) — press.html added
- [`STATUS.md`](STATUS.md) — updated to v0.6.0

---

## [0.5.0] — 2026-05-14

### Why MINOR
- README перероблений як справжній project entry point
- Marketing site deployment-ready

### Changed
- [`README.md`](README.md) — full overhaul. Quick-navigation, full repo tree, team, operating model, metrics.

---

## [0.4.9] — 2026-05-14

### Added
- [`docs/DEPLOYMENT.md`](docs/DEPLOYMENT.md) — Cloudflare Pages deployment runbook
- [`sitemap.xml`](sitemap.xml) — SEO sitemap
- [`robots.txt`](robots.txt) — blocks AI training scrapers

---

## [0.4.8] — 2026-05-14

### Added
- [`workout-live.html`](workout-live.html) — high-fidelity workout mockup з CV
- `content/post-09-building-victor-voice.md`

---

## [0.4.7] — 2026-05-14

### Added
- [`docs/CONTENT_CALENDAR.md`](docs/CONTENT_CALENDAR.md)
- [`STATUS.md`](STATUS.md) — comprehensive project status snapshot
- `content/post-08-why-no-free-tier.md`
- [`docs/B2B_PLAYBOOK.md`](docs/B2B_PLAYBOOK.md)
- [`docs/ENGINEERING_RUNBOOK.md`](docs/ENGINEERING_RUNBOOK.md)
- `programs/sample-cycle-aware-week.md`
- [`docs/INVESTOR_UPDATE_TEMPLATE.md`](docs/INVESTOR_UPDATE_TEMPLATE.md)
- [`faq.html`](faq.html) — 30+ questions у 7 категоріях
- `content/post-07-progressive-overload-myths.md`
- [`docs/DECISION_LOG.md`](docs/DECISION_LOG.md) — 24 documented decisions
- `programs/sample-deload-week.md`
- `nutrition/sample-cutting-plan.md`
- [`docs/SUPPORT.md`](docs/SUPPORT.md)
- [`docs/I18N.md`](docs/I18N.md)
- `content/press-pitch-templates.md`
- `content/reddit-launch-posts.md`
- `hiring/jd-founding-engineer-ios.md`
- `hiring/jd-design-lead.md`
- `hiring/jd-sport-science-advisor.md`
- [`docs/LAUNCH_CHECKLIST.md`](docs/LAUNCH_CHECKLIST.md)
- `content/post-06-30-user-interviews.md`
- `content/twitter-launch-thread.md`

### Changed
- [`index.html`](index.html) — nav оновлений

---

## [0.3.9] — 2026-05-14

### Added
- [`settings.html`](settings.html) — Settings screen mockup
- [`docs/OKRS_2026.md`](docs/OKRS_2026.md)
- [`docs/DATA_ROOM.md`](docs/DATA_ROOM.md)
- [`about.html`](about.html)
- [`docs/SECURITY.md`](docs/SECURITY.md)
- `content/post-05-building-fitness-from-ukraine.md`
- [`pricing.html`](pricing.html)
- [`docs/EMAIL_SEQUENCE.md`](docs/EMAIL_SEQUENCE.md)
- [`docs/BETA_PLAYBOOK.md`](docs/BETA_PLAYBOOK.md)
- [`screens.html`](screens.html)

---

## [0.3.5] — 2026-05-14

### Added
- [`nutrition/sample-3day-plan.md`](nutrition/sample-3day-plan.md)
- [`docs/API.md`](docs/API.md)
- `content/post-03-we-tested-8-ai-trainers.md`

---

## [1.12.16] — 2026-05-28

### Added
- `content/post-124-rest-intervals-hypertrophy.md` — Відпочинок між підходами і гіпертрофія; Schoenfeld et al. (2016) JSCR 3 хв vs 1 хв; PCr кінетика (Harris 1976, Greenhaff 1994); спростування гормональної гіпотези (West 2010, Ahtiainen 2005); практичні орієнтири для компаундних і ізолюючих вправ; clinical-safety NOT REQUIRED

### Fixed
- `README.md` — виправлено записи post-118 і post-119 (відображали старі планові теми замість реально відправлених файлів: deload-strategies і rep-range-continuum); додано post-123 (cross-education) і post-124 до content roadmap

---

## [0.3.0] — 2026-05-14

### Added
- [`docs/PITCH.md`](docs/PITCH.md)
- [`docs/PERSONAS.md`](docs/PERSONAS.md)
- [`docs/TECHNICAL.md`](docs/TECHNICAL.md)

---

## [0.2.1] — 2026-05-14

### Added
- `VERSION` — single source of truth
- `CHANGELOG.md`
- `CONTRIBUTING.md`

---

## [0.2.0] — 2026-05-14

### Added
- [`index.html`](index.html) — marketing site
- 14 субагентів
- [`docs/BRAND.md`](docs/BRAND.md)
- [`docs/FLOWS.md`](docs/FLOWS.md)
- [`docs/MARKETING.md`](docs/MARKETING.md)

### Changed
- Victor: «Не пропускай. Ніколи.» → «Не пропускай два поспіль.»
- Гасло: «Тренер. У кишені.»

---

## [0.1.0] — 2026-05-13

### Added
- Перша версія marketing-сайту
- 7 початкових субагентів
- Базова естетика: warm-black + cream + acid lime + Fraunces/Manrope/JBMono
