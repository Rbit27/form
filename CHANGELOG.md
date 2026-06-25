# Changelog · FORM

## [8.88.0] — 2026-06-25

### Added
- `content/post-2531-warmup-movement-quality-readiness-diagnostic.md` — Серія 2526–2535 «Сигнали готовності без wearables», пост 6/10. Розминка як живий тест готовності нервово-м'язової системи. Три маркери: (1) RPE першого субмаксимального підходу — особиста базова лінія ±1 норма, +2 — знижений об'єм, +3 і вище — технічна сесія; (2) технічна плавність і координація — «течія» руху vs. «розклеєність», когнітивне зусилля на підтримку форми; (3) динамічний тренд — патерн А (покращення до 3-го підходу) проти патерну Б (немає покращення після 4–5 підходів). Enoka & Duchateau (2008): ЦНС-стомленість деградує координацію раніше суб'єктивного відчуття. Таблиця рішень по тренду. Протокол 15–20 хв діагностичної розминки. clinical-safety: NOT_REQUIRED. sports-scientist review pending. Пост 2530 (DOMS/болісність) BLOCKED — clinical-safety HARD VETO, тема «біль», очікує review.
- `blog.html` — post-2531 card added at top of feed.
- `README.md` — post-2531 → draft; post-2530 → ⚠️ clinical-safety REVIEW REQUIRED; Editorial series 2556–2565 «Тренувальний щоденник як інструмент» proposed (10 posts: навіщо вести правильно, що записувати, ретроспективне читання, паттерни в нелінійному прогресі, RPE-калібратор, технічні нотатки, когнітивні упередження в самооцінці, місячний огляд, рішення про зміну програми, синтез).

### Changed
- `VERSION` — 8.87.0 → 8.88.0.

---

## [8.87.0] — 2026-06-25

### Added
- `content/post-2529-motivation-vs-physical-readiness.md` — Серія 2526–2535 «Сигнали готовності без wearables», пост 4/10. Мотивація і фізіологічна готовність — незалежні змінні: перша не підтверджує і не заперечує другу. Marcora (2009) psychobiological model: perceived exertion — рішення мозку, модульоване мотиваційним фоном. Boksem & Tops (2008): ментальна стомленість змінює effort allocation без зміни периферійної спроможності. Дві системні помилки (тренування при мотивації + поганому відновленні; пропуск при апатії + нормальному відновленні). Оперативна діагностика: стан «А» vs. «Б» за реакцією на розминку. 2×2-таблиця рішень: мотивація × готовність. Ключове правило: рішення після першого субмаксимального підходу, не до входу в зал. clinical-safety: NOT_REQUIRED. sports-scientist review pending.
- `blog.html` — post-2529 card added at top of feed.

### Changed
- `VERSION` — 8.86.1 → 8.87.0.

---

## [8.86.1] — 2026-06-25

### Added
- `docs/SOC2_READINESS.md §114` — Cross-reference patch: PRICE-OBS-E-001 and PRICE-OBS-E-002 registered in §79.4 master consolidated evidence table (2 new rows after LITH-STALE-E-001; §79.4 count 73 → 75). R2 subfolder `compliance/evidence/pricing-exceptions/` entry added to §80.3 (`form_api` NO ACCESS). Both artefacts added to §80.4 Vanta mirror protocol (PRICE-OBS-E-001 quarterly from M9; PRICE-OBS-E-002 annual from M9). Closes `docs/OBSERVABILITY.md §55.10 item 5` (P1/M9). SOC2_READINESS.md v3.38.0 → v3.39.0.

### Changed
- `docs/OBSERVABILITY.md §55.10 item 5` — Status updated: `[ ]` → `[x] Done — 2026-06-25 (SOC2_READINESS.md §114 (v3.39.0), this commit)`.
- `VERSION` — 8.86.0 → 8.86.1.

---

## [8.86.0] — 2026-06-25

### Added
- `content/post-2527-three-types-of-fatigue.md` — Block 2526–2535 «Сигнали готовності» post 2/10. Три типи стомленості, що відчуваються однаково, але вимагають різних рішень. Нейром'язова (центральна) стомленість: знижена ЦНС-рекрутація рухових одиниць (Gandevia, 2001 — twitch interpolation; РФD знижується раніше за максимальну силу); вимагає повного відпочинку або субмаксимальної технічної сесії ≤60% 1RM без відмови. Периферійна (м'язова): EIMD і метаболічне виснаження локально (Sahlin et al., 1998); вимагає перерозподілу сесії, а не скасування. Системна (автономна): хронічне алостатичне перевантаження (McEwen & Stellar, 1993); симпатичний зсув, порушений сон, анергія поза залом; вимагає deload-тижня 7–14 діб, не одного дня відпочинку. Ключовий механізм: суб'єктивне сприйняте зусилля — рішення мозку (Noakes, 2012; Marcora, 2009), тому всі три типи «відчуваються однаково». Операційний 4-кроковий протокол диференціації: локальність → відповідь на розминку → якість першого підходу → загальний контекст. Таблиці диференціації і рішень для кожного типу. clinical-safety: NOT_REQUIRED. sports-scientist review pending.
- `blog.html` — post-2527 card added at top of feed.

### Changed
- `STATUS.md` — Series 2526–2535 progress updated to 2/10; next: post-2528 (Суб'єктивний RPE як сигнал готовності).
- `README.md` — post-2527 → draft.
- `VERSION` — 8.85.1 → 8.86.0.

---

## [8.85.1] — 2026-06-25

### Added
- `docs/AUDIT_LOG_SCHEMA.md §Enterprise Pricing Exception Monitoring events` — New subsection (v2.47): three DEC-030 HMAC-chained events covering the pg_cron job 46 (`pricing_exception_compliance_monitor`) monitoring-layer lifecycle. `enterprise.reentry_chain_integrity_violation` CRITICAL/7yr: daily monitoring sentinel that detects REENTRY-CHAIN-01 bypasses (loyalty re-entry contract renewed without prior `enterprise.pricing_exception_approved`); REENTRY-MONITOR-CHAIN-01 ordering invariant (DEC-030 HTTP 200 confirmed before AL-PRICE-01 PagerDuty P0); Zod `ReentryChainIntegrityViolationPayload` (`tenant_id` UUID, `renewed_event_id` UUID, `renewal_date` date, `check_run_at`). `system.pricing_exception_quarterly_audit_triggered` STANDARD/3yr: quarterly audit trigger emitted by sweep 2 on first 7 days of each calendar quarter; triggers compliance-officer PRICE-OBS-E-001 filing obligation; Zod `PricingExceptionQuarterlyAuditTriggeredPayload` (aggregate `exception_type` breakdown; no `tenant_id`). `system.pricing_exception_check_passed` LOW/1yr: all-clear baseline for AL-PRICE-03 26h dead-man's switch; Zod `PricingExceptionCheckPassedPayload` (aggregate counts only; no `tenant_id`). REENTRY-MONITOR-CHAIN-01 invariant block, emitter assignments, alert routing, CC5.2/CC1.4/CC4.1 SOC 2 auditor narratives included. Retention table extended (+3 rows). Closes `docs/OBSERVABILITY.md §55.10` item 1 (P0/M9).

### Changed
- `docs/AUDIT_LOG_SCHEMA.md` — v2.46 → v2.47. New `§Enterprise Pricing Exception Monitoring events` subsection added. Retention table: +3 rows for `enterprise.reentry_chain_integrity_violation` (7yr CC5.2/CC1.4), `system.pricing_exception_quarterly_audit_triggered` (3yr CC1.4/CC4.1), `system.pricing_exception_check_passed` (1yr CC4.1).
- `docs/OBSERVABILITY.md` — v5.2.4 → v5.2.5. §55.10 item 1 status: `[ ]` → `[x] Done — 2026-06-25 (AUDIT_LOG_SCHEMA.md v2.47, this commit)`. v5.2.5 patch note added.
- `VERSION` — 8.85.0 → 8.85.1.

## [8.85.0] — 2026-06-25

### Added
- `content/post-2526-training-readiness-morning-feeling.md` — Editorial series «Сигнали готовності · 2526–2535» post 1/10. Що таке готовність до тренування фізіологічно: три незалежні виміри (нейром'язовий, периферійний, автономний). Чому ранкове відчуття систематично занижує готовність: sleep inertia (Naëgelé et al., 2003), Cortisol Awakening Response (нормальна фізіологія, не стрес-маркер), незалежність мотивації від фізичного стану (Marcora et al., 2009). Broussard et al. (2015): суб'єктивна готовність після 5.5 год сну знижується помітніше за об'єктивну силову продуктивність. Pilcher & Huffcutt (1996): ми переоцінюємо, наскільки погано нам насправді. Три надійніші маркери: накопичений сон за 48 год, специфічна болісність цільових м'язів, перші підходи в залі як діагностика. Базове правило серії. clinical-safety: NOT_REQUIRED. sports-scientist review pending. Blog card added.

### Changed
- `blog.html` — post-2526 card added (top of grid).
- `STATUS.md` — v8.84.1 → v8.85.0; content library row updated (series 2526–2535 progress 1/10; series 2536–2545 proposed).
- `VERSION` — 8.84.1 → 8.85.0.
- `README.md` — Editorial series 2536–2545 «Прийняття рішень без тренера: алгоритми замість відчуттів» proposed.

## [8.84.1] — 2026-06-25

### Added
- `docs/INCIDENT_RESPONSE.md R-46` — Pricing Exception Compliance Monitor Stale (`pricing_exception_compliance_monitor` · job 46). Forty-sixth runbook. Closes `docs/OBSERVABILITY.md §55.10` item 6. Trigger: AL-PRICE-03 PagerDuty P1 `form-devops` → devops-lead; dedup `pricing-exception-check-stale` 26h; auto-resolves on `system.pricing_exception_check_passed`. Two escalation paths: §R-46.5 REENTRY-CHAIN-01 blind spot (P0 — manual `enterprise.reentry_chain_integrity_violation` CRITICAL/7yr emission per violation preserving REENTRY-MONITOR-CHAIN-01 ordering invariant; corrective pricing review per COST_MODEL §44.7); §R-46.6 quarterly trigger miss (P1 — manual sweep 2 SQL + `system.pricing_exception_quarterly_audit_triggered` emission + audit initiation). Five scope queries (R-46-C1 staleness; R-46-C2 REENTRY-CHAIN-01 P0 gate; R-46-C3 quarterly trigger miss gate; R-46-C4 H4 peer discriminator; R-46-C5 H1 job registration). Four root cause hypotheses H1–H4. Six-step recovery procedure. Two DEC-030 HMAC-chained events: `system.pricing_exception_monitor_stale_declared` HIGH/7yr + `system.pricing_exception_monitor_restored` STANDARD/3yr; PRICING-MONITOR-STALE-CHAIN-01 ordering invariant (HTTP 422 `PRICING_MONITOR_STALE_CHAIN_01_VIOLATION` on breach → R-05). SOC 2: CC5.2 (REENTRY-CHAIN-01 monitoring blind spot coverage; `reentry_violations_found_during_stale = 0` attestation); CC1.4 (pricing exception approval control completeness; §R-46.5 compensating control); CC4.1 (quarterly audit trigger COST_MODEL §44.7; `quarterly_trigger_manually_fired` attestation); CC7.1/CC7.2. Privacy floor: both DEC-030 event payloads carry integer counts, booleans, and timestamps only — no `tenant_id`, employee PII, or GDPR Art. 9 data. PRICE-STALE-E-001 annual evidence artefact defined (§R-46.13 item 4, pending). Four-item implementation checklist (items 1–2 pending P0/M9; item 3 `[x] Done — this commit`; item 4 pending P1/M11). Three Slack communication templates.

### Changed
- `docs/INCIDENT_RESPONSE.md` — v3.10 → v3.11. R-46 appended (forty-sixth runbook).
- `docs/OBSERVABILITY.md` — v5.2.3 → v5.2.4. §6.2 AL-PRICE-03 stale-consequence cross-ref updated to `INCIDENT_RESPONSE R-46 (§R-46.5/R-46.6; v1.0, 2026-06-25)` (was "to be authored"). §12.6 job 46 stale-consequence cross-ref updated to same. §55.10 item 6 marked `[x] Done — 2026-06-25 (INCIDENT_RESPONSE.md v3.11, this commit)`. v5.2.4 patch note added.
- `VERSION` — 8.84.0 → 8.84.1.

## [8.84.0] — 2026-06-25

### Added
- `content/post-2525-between-sessions-synthesis.md` — Editorial series «Між сесіями · 2516–2525» post 10/10 · СЕРІЯ ЗАВЕРШЕНА. Синтез: десять принципів між-сесійного контексту — від фізіології адаптації до між-сесійної системи. Ієрархія впливу (сон → об'єм/інтервал → когнітивний стрес → NEAT → ВСР → активне відновлення → cool-down → mobility). Мінімальна між-сесійна операційна система (пост-тренувальний протокол, дні відпочинку, тижневе планування). Що серія не охопила: харчування, психологія мотивації, специфіка видів тренувань, реабілітаційний контекст. clinical-safety: NOT_REQUIRED. sports-scientist review pending. Blog card added.
- `content/post-2524-neat-daily-activity-between-sessions.md` — Editorial series «Між сесіями · 2516–2525» post 9/10. NEAT (Non-Exercise Activity Thermogenesis) як між-сесійна змінна відновлення і адаптації. Levine et al. (2006): різниця між людьми до 2000 ккал/день. Три механізми впливу: кровотік і доставка субстратів, інсулінова чутливість (Duvivier et al., 2017: розподілена ходьба > 1 год тренування + 13 год сидіння), АНС-тонус і ВСР. Парадокс компенсаторного зниження: King et al. (2008, 2012) — введення тренувальної програми систематично знижує NEAT протягом решти дня. Biddle & Asare (2011): ходьба прискорює когнітивне відновлення. Операційні орієнтири по ситуаціях. Нерухомість між тренуваннями ≠ відновлення. clinical-safety: NOT_REQUIRED. sports-scientist review pending. Blog card added.
- `content/post-2523-cool-down-transition-to-recovery.md` — Editorial series «Між сесіями · 2516–2525» post 8/10. Фізіологія перших 30 хвилин після тренування: ЧСС-відновлення (Farah et al., 2016), лактатний кліренс (Menzies et al., 2010: активне охолодження −30% лактату за 20 хв), парасимпатична реактивація (Buchheit et al., 2007: ВСР-відновлення 30 хв – кілька годин). Активне vs. пасивне охолодження: перевага при ацидозі, мінімальна різниця при стандартному силовому. Cool-down не знижує DOMS (Law et al., 2009). Чотири операційних ситуації з логікою рішення. Що не є cool-down (стретчинг, сауна, CWI). Van Someren (2006): прохолодний душ при пізньому тренуванні. clinical-safety: NOT_REQUIRED. sports-scientist review pending. Blog card added.

### Changed
- `blog.html` — картки post-2523, post-2524, post-2525 додані на початок стрічки (над post-2522).
- `README.md` — серія 2516–2525 header: proposed → COMPLETE · v8.84.0; posts 2523–2525 → draft.
- `STATUS.md` — series 2516–2525 progress 7/10 → 10/10 COMPLETE; post-2523, post-2524, post-2525 summaries appended; next series: 2526–2535.
- `VERSION` — 8.81.3 → 8.84.0.

## [8.81.3] — 2026-06-25

### Added
- `docs/OBSERVABILITY.md §55` — Pricing Exception Compliance Observability (v5.2.3). Closes three monitoring gaps: (1) no daily sentinel for REENTRY-CHAIN-01 chain integrity (Worker-layer HTTP 422 enforcement per COST_MODEL §44.5 existed but had no monitoring-layer verification); (2) no automated trigger for the quarterly pricing exception audit (COST_MODEL §44.7 obligation); (3) no PRICE-SLO-* or PRICE-OBS-E-* evidence artefacts. Adds: pg_cron job 46 `pricing_exception_compliance_monitor` (daily 09:00 UTC; two sweeps — REENTRY-CHAIN-01 correlated subquery + quarterly aggregate trigger; REENTRY-MONITOR-CHAIN-01 ordering invariant); three new DEC-030 monitoring events (`enterprise.reentry_chain_integrity_violation` CRITICAL/7yr; `system.pricing_exception_quarterly_audit_triggered` STANDARD/3yr; `system.pricing_exception_check_passed` LOW/1yr); three SLOs (PRICE-SLO-01 zero-tolerance; PRICE-SLO-02 quarterly cadence; PRICE-SLO-03 1-business-day floor override review); three alert rules (AL-PRICE-01/02/03); two evidence artefacts (PRICE-OBS-E-001 quarterly outcome CC5.2/CC1.4/CC4.1 7yr; PRICE-OBS-E-002 annual monitoring run history CC4.1/A1.1 3yr); `pricing_exception_health` subsection inserted into §6.2; job 46 registered in §12.6 (v1.9 patch); Metabase `Pricing Exception Compliance` dashboard (7 panels). Privacy floor: aggregate counts only in all monitoring events and artefacts; `tenant_id` UUID only in violation events; no `approver_user_id`, employee PII, or GDPR Art. 9 data anywhere in §55. `tenant_manager` (HR) and `enterprise_admin` excluded from dashboard.

### Changed
- `docs/OBSERVABILITY.md` — v5.2.2 → v5.2.3. §55 appended; §6.2 `pricing_exception_health` subsection inserted after `litigation_hold_health`; §12.6 job 46 row added; §12.6 freshness window note extended; §12.6 v1.9 patch note added.
- `VERSION` — 8.81.2 → 8.81.3.

## [8.81.2] — 2026-06-25

### Added
- `content/post-2522-mobility-tissue-work-between-sessions.md` — editorial series «Між сесіями · 2516–2525» post 7/10. Тема: мобільна та тканинна робота між тренуваннями — що підтверджує доказова база (foam rolling, статичний стретчинг, structured mobility), де межа ефективного і ритуального, операційний алгоритм. clinical-safety: NOT_REQUIRED. sports-scientist review pending.

### Changed
- `README.md` — topics for posts 2519–2521 corrected to match actual written content (HRV monitoring, training frequency, cognitive load); post-2522 added as draft.
- `STATUS.md` — content engine range 11–50: post-2522 added; series 2516–2525 progress 6/10 → 7/10.
- `VERSION` — 8.81.1 → 8.81.2.

## [8.81.1] — 2026-06-25

### Added
- `docs/SOC2_READINESS.md §113` — §79.4 Evidence Table Artefact Registration + §112 Omission Fix (v3.38.0). Inserts four missing rows into §79.4 Consolidated Evidence Collection Table (count 69 → 71 net, correcting §112's overclaim): **LITH-E-001** (Annual Litigation Hold Compliance Report — C1.2/CC5.3/CC4.1, annual, 7yr, `compliance/evidence/litigation-hold/LITH-E-001_<YYYY>.md`); **LITH-OBS-E-001** (Annual `litigation_hold_compliance_monitor` pg_cron run history + AL-LITH-01/02/03 — CC5.3/CC4.1/C1.2, annual, 3yr); **CHN-STALE-E-001** (Quarterly `offboard_chain_monitor` job 44 stale-monitoring health report — CC6.1/CC7.2, quarterly from M11, 3yr, closes R-44.11 item 4); **LITH-STALE-E-001** (Quarterly `litigation_hold_compliance_monitor` job 45 stale-monitoring health report — CC5.3/CC7.2/C1.1/C1.2, quarterly from M8, 3yr, closes R-45.14 item 4). §112 omission: §112 (v3.37.0) claimed LITH-E-001 and LITH-OBS-E-001 rows were inserted (count 67→69) but they were described in prose only and never written into the markdown table — §113 corrects this. Privacy floor confirmed: all four artefacts carry aggregate integers and `tenant_id` UUID only; no employee `user_id`, name, email, health value, or GDPR Art. 9 data. HMAC-chain cross-references: LITH-MONITOR-STALE-CHAIN-01 (R-45) and OFFBOARD-CHAIN-MONITOR-STALE-CHAIN-01 (R-44) ordering invariants documented.

### Changed
- `docs/SOC2_READINESS.md` — v3.37.0 → v3.38.0. §113 appended; four §79.4 rows inserted.
- `docs/INCIDENT_RESPONSE.md` — v3.9 → v3.10. R-44.11 item 4 marked `[x] Done — 2026-06-25`; R-45.14 item 4 marked `[x] Done — 2026-06-25`; v1.2 note added to R-44; v1.1 note added to R-45.
- `VERSION` — 8.81.0 → 8.81.1.

## [8.81.0] — 2026-06-25

### Added
- `docs/AUDIT_LOG_SCHEMA.md §Enterprise Litigation Hold Monitoring events` — new section with two DEC-030 HMAC-chained events: `system.litigation_hold_monitor_stale_declared` (HIGH/7yr) and `system.litigation_hold_monitor_restored` (STANDARD/3yr). Full event table, Zod v2 schemas (`LitigationHoldMonitorStaleDeclaredPayload` + `LitigationHoldMonitorRestoredPayload`), emitter assignments, alert routing, LITH-MONITOR-STALE-CHAIN-01 ordering invariant, and SOC 2 auditor narratives (CC5.3/C1.1/C1.2/CC7.2). Closes `docs/INCIDENT_RESPONSE.md R-45.14` item 1 (P0/M6).

### Changed
- `docs/AUDIT_LOG_SCHEMA.md` — header v2.45 → v2.46; v2.46 changelog entry added.
- `docs/INCIDENT_RESPONSE.md R-45.14` — item 1 status `[ ]` → `[x] Done — 2026-06-25 (AUDIT_LOG_SCHEMA.md v2.46)`.
- `VERSION` — 8.80.1 → 8.81.0.

## [8.80.1] — 2026-06-25

### Added
- `docs/INCIDENT_RESPONSE.md R-45` — Litigation Hold Compliance Monitor Stale runbook (v3.9). Closes `docs/OBSERVABILITY.md §54.10` item 6 (P1/M7). Forty-fifth runbook; first with three-type obligation breach coverage. R-45 is the IC protocol for `litigation_hold_compliance_monitor` (job 45, `0 8 * * *`, 26h freshness window) staleness. Three independent breach gates: R-45-C2a (LITH-SLO-01 review-overdue), R-45-C2b (LITH-SLO-02 max-duration cap — P0 trigger), R-45-C2c (LITH-SLO-03 deletion-overdue). Three escalation paths: §R-45.5 (review-overdue — manual `enterprise.litigation_hold_review_overdue` HIGH/7yr emit + stale-window note), §R-45.6 (max-duration breach — P0 founder page + CSM + legal waiver-or-release + manual `enterprise.litigation_hold_max_duration_breached` CRITICAL/7yr emit), §R-45.7 (deletion-overdue — manual deletion workflow + `deletion_completed_date` population + manual `enterprise.litigation_hold_deletion_overdue` HIGH/7yr emit). Six-step recovery (H1–H4 root cause). Two DEC-030 HMAC-chained events: `system.litigation_hold_monitor_stale_declared` HIGH/7yr + `system.litigation_hold_monitor_restored` STANDARD/3yr; LITH-MONITOR-STALE-CHAIN-01 ordering invariant. Privacy floor: both payloads carry integer counts and timestamps only — no `tenant_id`, no employee PII, no GDPR Art. 9 data. SOC 2: CC5.3 + C1.1 + C1.2 + CC7.1 + CC7.2.

### Changed
- `docs/INCIDENT_RESPONSE.md` — v3.8 → v3.9. R-45 appended.
- `docs/OBSERVABILITY.md` — v5.2.0 → v5.2.2. §54.10 item 6 marked `[x] Done — 2026-06-25 (INCIDENT_RESPONSE.md v3.9)`. §12.6 job 45 stale-consequence cross-ref updated: "to be authored" → `§R-45.5/R-45.6/R-45.7; v1.0, 2026-06-25`. §54.4 AL-LITH-01/02/03 runbook rows updated. §54.5 cross-ref updated.
- `VERSION` — 8.80.0 → 8.80.1.

## [8.80.0] — 2026-06-25

### Added
- `docs/OBSERVABILITY.md §54` — Litigation Hold Compliance Observability (v5.2.0). Closes DATA_MODEL §46.8 item 3 (P1/M6). Three SLOs (LITH-SLO-01: 6-month review; LITH-SLO-02: 36-month max duration; LITH-SLO-03: 10-business-day post-release deletion), three alert rules (AL-LITH-01/02/03), pg_cron job 45 `litigation_hold_compliance_monitor` (`0 8 * * *`, 3 enforcement SQL sweeps via covering indexes, 3 DEC-030 chain invariants), LITH-OBS-E-001 monitoring-layer evidence artefact (annual, 3yr, CC5.3/CC4.1/C1.2). §6.2 `litigation_hold_health` subsection inserted; §12.6 job 45 registered.
- `docs/SOC2_READINESS.md §112` — Cross-Reference Patch (v3.37.0). Closes DATA_MODEL §46.8 item 4 + OBSERVABILITY §54.10 item 5. Registers LITH-E-001 (C1.2/CC5.3/CC4.1, annual 7yr) + LITH-OBS-E-001 (CC5.3/CC4.1/C1.2, annual 3yr) in §79.4 master consolidated evidence table (count 67 → 69); creates R2 subfolder `compliance/evidence/litigation-hold/` (EU-region, form_api NO ACCESS); adds 2 entries to §80.4 Vanta mirror.
- `content/post-2521-cognitive-load-between-sessions.md` — Editorial series 2516–2525 · пост 6/10. «Когнітивне навантаження між сесіями: як розумова стомленість впливає на тренувальну готовність». Marcora et al. (2009): 90 хв когнітивного завдання → −15,1% часу до відмови при ідентичних VO₂, ЧСС, лактаті. Психобіологічна модель стомленості: RPE підвищений до початку роботи, без периферійних змін. Нейробіологія: виснаження серотоніну/дофаміну (Tanaka & Watanabe, 2012), накопичення аденозину (Van Cutsem et al., 2017), зниження метаболічної активності PFC (Boksem & Tops, 2008). Наслідки для силового тренування: технічна деградація настає раніше (PFC-залежна функція), RPE першого підходу вищий, виконавчий контроль зв'язку розум-м'яз знижений. Чотири типи когнітивного навантаження: стійка увага, робоча пам'ять, емоційний самоконтроль, непередбачуваність — різне ступені виснаження. Взаємодія: розумова стомленість + дефіцит сну → мультиплікативний ефект; нормальний HRV при значній розумовій стомленості — валідний сценарій. Три операційних інструменти оцінки: ретроспективна 3-питання шкала (3–7/8–11/12–15), тест реакції, суб'єктивна «ментальна свіжість». Алгоритм коригування: помірна стомленість → −20–30% об'єму; значна → −40–50% об'єму, −10–15% інтенсивності. Паттерни розумової стомленості у тренувальному журналі. Pageaux & Lepers (2016); Lorist et al. (2009). clinical-safety: NOT_REQUIRED. sports-scientist review pending. Blog card added.

### Changed
- `docs/DATA_MODEL.md` — v1.31 → v1.32. §46.9 cross-reference table updated: OBSERVABILITY §54 🟡 → 🟢 (v5.2.0); SOC2_READINESS §79.4 LITH-E-001+LITH-OBS-E-001 🟡 → 🟢 (§112 v3.37.0). §46.8 checklist items 3+4 marked Done.
- `docs/OBSERVABILITY.md` — v5.1.5 → v5.2.0/v5.2.1. §54 appended; §6.2 `litigation_hold_health` subsection inserted; §12.6 job 45 `litigation_hold_compliance_monitor` registered; freshness window note extended; v1.8 patch note added.
- `docs/SOC2_READINESS.md` — v3.36.0 → v3.37.0. §112 appended.
- `blog.html` — картка post-2521 додана на початок стрічки (над post-2520).
- `STATUS.md` — series 2516–2525 progress 5/10 → 6/10; post-2521 summary added; current version v8.76.0 → v8.80.0.
- `VERSION` — 8.79.1 → 8.80.0.

## [8.79.1] — 2026-06-25

### Added
- `docs/DATA_MODEL.md §46` — `litigation_hold_records` Schema · Migration 0087. Closes the DDL gap from DEC-080 (2026-06-23, OQ-WIN-04): the litigation hold procedure (MSA §11.6) and two DEC-030 events (`enterprise.litigation_hold_declared` HIGH/7yr, `enterprise.litigation_hold_released` HIGH/7yr) were registered without a Postgres state table to support automated compliance monitoring. §46 defines migration 0087: new table with three ENUMs (status: declared/released/deletion_completed; hold_reason_category: 4 values; hold_release_reason: 5 values); 18 columns including `held_data_categories TEXT[]`, `target_review_date`, `max_expiry_date`, `deletion_target_date`, `deletion_completed_date`; six CHECK constraints — `lhr_held_data_categories_valid` structurally excludes GDPR Art. 9 health data via `@<` operator (implements DEC-036 zero-grace invariant at DDL layer), `lhr_max_expiry_equals_36_months` (MSA §11.6.4 cap), `lhr_release_date_within_max_duration`, `lhr_release_fields_complete`, `lhr_deletion_completed_requires_release`; five indexes including three covering indexes for pg_cron job 45 monitoring sweeps (review overdue, max-duration approach, post-release deletion deadline); RLS: form_admin ALL, compliance_officer SELECT+UPDATE, form_system SELECT, form_api REVOKED, no tenant role access. New evidence artefact LITH-E-001 (annual, 7yr, C1.2/CC5.3/CC4.1). Cross-reference obligation: OBSERVABILITY §54 (pg_cron job 45, LITH-SLO-01/02/03, AL-LITH-01/02/03) pending as §46.8 item 3.

### Changed
- `VERSION` — 8.79.0 → 8.79.1.
- `docs/DATA_MODEL.md` — v1.30 → v1.31; TOC updated (§46 added).

## [8.79.0] — 2026-06-25

### Added
- `content/post-2520-training-frequency-inter-session-interval.md` — Editorial series 2516–2525 · пост 5/10. «Частота тренувань і між-сесійний інтервал: скільки часу насправді потрібно для відновлення». Два незалежні процеси відновлення: периферійне (м'язова тканина, EIMD) і центральне нейральне (ЦНС, нейром'язова передача) — часто не синхронізовані. Три фактори між-сесійного інтервалу: інтенсивність відносно 1RM, обсяг, ексцентрична складова. Таблиця вікон відновлення по типах роботи: технічна/субмакс (24–36 год нейрально), гіпертрофійна помірна (36–48 год периферійно), гіпертрофійна EIMD (72–96+ год периферійно), силова/інтенсивна (48–72 год нейрально), накопичувальна. Розрізнення між частотою тренувань і між-сесійним інтервалом як різними питаннями. Помилка: плутати частоту з обсягом — Ralston et al. (2017): при однаковому тижневому об'ємі частота має мінімальний вплив на гіпертрофію. Емпіричний 3-кроковий протокол визначення власного між-сесійного інтервалу. Moran-Navarro et al. (2017); Schoenfeld & Grgic (2018); Ralston et al. (2017); Häkkinen et al. (2003); Clarkson & Hubal (2002); Meeusen et al. (2013); McLester et al. (2000). clinical-safety: NOT_REQUIRED. sports-scientist review pending. Blog card added.

### Changed
- `blog.html` — картка post-2520 додана на початок стрічки (над post-2519).
- `STATUS.md` — series 2516–2525 progress 4/10 → 5/10; post-2520 summary added.
- `VERSION` — 8.78.0 → 8.79.0.

## [8.78.0] — 2026-06-25

### Added
- `content/post-2519-hrv-between-sessions-monitoring.md` — Editorial series 2516–2525 · пост 4/10. «HRV між сесіями: варіабельність серцевого ритму як індикатор готовності атлета». HRV як відображення балансу симпатики/парасимпатики АНС; RMSSD — стандарт спортивного моніторингу. П'ять умов коректного вимірювання: ранок, одна позиція (супінація або стоячи — завжди однакова), 5 хв, постійний час, нічого до виміру. Головне правило інтерпретації: rolling average 7–14 днів, не абсолютні цифри; коефіцієнт готовності (RMSSD сьогодні / rolling avg). Три практичні сценарії: перед важкою сесією (знижуй а не скасовуй), після накопичувального тижня (deload signal), між-сесійний моніторинг груп. Що HRV не показує: локальна м'язова готовність vs системний стрес. П'ять поширених помилок. Мінімальний 4-кроковий протокол з валідацією. Plews et al. (2013); Buchheit (2014); Kiviniemi et al. (2007); Stanley et al. (2013); Flatt & Esco (2016); Meeusen et al. (2013). clinical-safety: NOT_REQUIRED. sports-scientist review pending. Blog card added.

### Changed
- `blog.html` — картка post-2519 додана на початок стрічки (над post-2518).
- `STATUS.md` — series 2516–2525 progress 3/10 → 4/10; post-2519 summary added.
- `VERSION` — 8.77.1 → 8.78.0.

## [8.77.1] — 2026-06-25

### Changed
- `enterprise.html` — Sub-processor list compliance badge added to the security posture section (after Shared responsibility model badge). Links to `/subprocessors.html`; label: "Full vendor registry · 30-day advance notice of changes · DPA/SCC status per processor". Closes `docs/SOC2_READINESS.md` P1 obligation: add sub-processor list link to `enterprise.html` security section.
- `pricing-enterprise.html` — Sub-processor list row added to the Compliance & audit section of the feature comparison table. Row: "Sub-processor list (30-day notice)" · ✓ all tiers · links to `/subprocessors.html`. Closes `docs/SOC2_READINESS.md` P1 obligation: add sub-processor list link to `pricing-enterprise.html` compliance table.

## [8.77.0] — 2026-06-25

### Added
- `content/post-2518-active-recovery-vs-passive-rest.md` — Editorial series 2516–2525 · пост 3/10. «Активне відновлення vs. повний відпочинок: операційне рішення без бро-науки». Три механізми реального активного відновлення: кліренс лактату (Signorile et al. 1993; Dupont et al. 2004), підвищений кровообіг при EIMD (Mika et al. 2007), парасимпатична активація. Чотири сценарії де пасивний відпочинок виграє: накопичувальний тиждень, ЦНС-стомленість як первинний фактор, обмежений сон, неможливість утримати низьку інтенсивність. Три умови де активне відновлення виграє: гостра периферійна стомленість при короткому між-сесійному інтервалі, загальна гіподинамія протягом дня, ексцентрично-навантажувальна сесія + ≥48 год до наступної. Операційний алгоритм: три запитання для рішення. Що не є активним відновленням: сауна, foam rolling, «легкий» зал, стретчинг. Практичний протокол для двох сценаріїв. clinical-safety: NOT_REQUIRED. sports-scientist review pending. Blog card added.
- `README.md` — Editorial series 2526–2535 «Сигнали готовності: як self-coached атлет читає власне тіло без wearables» додана як новий proposed блок. post-2518 → draft, post-2517 → draft (retrofix).

### Changed
- `blog.html` — картка post-2518 додана на початок стрічки (над post-2517).
- `STATUS.md` — content engine row 11–50: post-2518 added v8.77.0, series 2526–2535 proposed.
- `VERSION` — 8.76.0 → 8.77.0.

## [8.76.0] — 2026-06-25

### Added
- `content/post-2351-block-transition-planning.md` — Editorial series 2346–2355 · пост 6/10. «Планування переходу між блоками: операційний протокол без тренера». П'ятикроковий протокол: (1) ретроспектива блоку (compliance, e1RM тренд, RPE-тренд, технічна якість, обмеження); (2) benchmark session під час деловантажного тижня — стандартизований протокол для міжблокового порівняння; (3) вибір теми наступного блоку за трьома сигналами даних (технічна деградація → технічний блок; RPE не знижується → деакумуляція; compliance <75% → реструктурування); (4) структурні рішення (вправи/обсяг/інтенсивність) на основі Issurin (2010) блокова модель, Israetel et al. (2019) прогресія обсягу, Schoenfeld (2016) спеціалізація; (5) фіксація плану з метрикою успіху. Деловантажний тиждень: три рівні ролі (відновлення + діагностика + когнітивний простір). Три типові помилки переходу. Fitts (1994), Meeusen et al. (2013), Helms et al. (2014), Halson (2014), Wulf et al. (2010), Rippetoe (2009). clinical-safety: NOT_REQUIRED. sports-scientist review pending. Blog card added.
- `content/post-346-invisible-adaptation.md` — перейменований з `post-291-invisible-adaptation.md` (усунено конфлікт нумерації з post-291-technical-drift-hidden-plateau.md, доданим v8.74.0). «Ти платиш за адаптацію, яку не бачиш». Blog card added. clinical-safety: NOT_REQUIRED.

### Changed
- `content/post-291-invisible-adaptation.md` → `content/post-346-invisible-adaptation.md` — git mv для усунення конфлікту нумерації post-291. Технічний дрейф залишається канонічним post-291 (blog card, v8.74.0).
- `blog.html` — картка post-2351 додана на початок стрічки (над post-2350); картка post-346 додана після post-291.
- `VERSION` — 8.75.0 → 8.76.0.

## [8.75.0] — 2026-06-25

### Added
- `docs/DATA_MODEL.md` — v1.30: §45 `enterprise_contracts` Loyalty Discount Schema Extension — Migration 0086. Closes `docs/COST_MODEL.md §44.8 item 1` (P0, M9). Triggered by DEC-081 (2026-06-23, OQ-WIN-02 resolution). New additive migration `0086_enterprise_contracts_discount_type.sql`: `ALTER TABLE enterprise_contracts ADD COLUMN IF NOT EXISTS contract_discount_type VARCHAR(30) NOT NULL DEFAULT 'none' CHECK (IN 'none','multi_year','upfront','loyalty_reentry')`. No DDL mutual-exclusivity constraint (billing_period vs contract_years discrepancy; REENTRY-CHAIN-01 in Worker is the real gate per §45.1 principle 3). Six-item staging validation checklist (§45.3.2). RLS unchanged from §16 (§45.5). REENTRY-CHAIN-01 Worker enforcement documented with four integration test specs (§45.6). SOC 2 REN-E-001 CC5.2/CC1.4 auditor narratives (§45.7). Two-item P0/M9 implementation checklist + one P1/M11 item (§45.8). Two cross-reference obligations closed: COST_MODEL §44.8 item 1 🟢 and AUDIT_LOG_SCHEMA v2.45 exception_type 🟢 (§45.9). TOC extended §42–§45. Header corrected v1.28 → v1.30 (v1.29 omitted title update; bundled here).

### Changed
- `docs/AUDIT_LOG_SCHEMA.md` — v2.45: `exception_type` field (`standard_discount`\|`loyalty_reentry`\|`pilot_credit`\|`floor_exception`) added to `enterprise.pricing_exception_approved` payload. Required and must equal `loyalty_reentry` when `contract_discount_type = 'loyalty_reentry'` on the associated `enterprise_contracts` row, per REENTRY-CHAIN-01 (DEC-081, COST_MODEL §44.5). Optional for pre-DEC-081 events (back-fills to `standard_discount`). Closes `docs/COST_MODEL.md §44.8 item 2` (P0, M9).
- `docs/COST_MODEL.md` — §44.8 item 1 `[ ]` → `[x] Done` (DATA_MODEL §45 authored, migration 0086 DDL canonical registration complete). §44.8 item 2 `[ ]` → `[x] Done` (AUDIT_LOG_SCHEMA.md v2.45 `exception_type` registered).
- `VERSION` — 8.74.0 → 8.75.0.

## [8.74.0] — 2026-06-25

### Added
- `content/post-291-technical-drift-hidden-plateau.md` — Editorial series 291–300 · пост 1/10. «Технічний дрейф: прихована причина плато, яку не видно без камери». Три механізми дрейфу: (1) накопичена мікровтома сполучної тканини → компенсаторна зміна біомеханіки; (2) підсвідоме уникання uncomfortable позицій; (3) нейральна оптимізація за зусиллям замість ефекту. Три форми: структурний дрейф (суглобові кути), темповий дрейф (краш ексцентрики), дрейф діапазону (ROM скорочення). Фізіологічний механізм невидимості: Fitts & Posner (1967) автономна стадія + forward models (Miall & Wolpert 1996) + пропріоцептивна похибка 8–12% (Proske & Gandevia 2012). Вплив на прогрес: зміна стимулу при «тій самій» програмі + захисна компенсація. Протокол виявлення: два відео з інтервалом ≥3 тижні, три питання для порівняння. Корекція: технічна сесія (дрейф <10%) або деload з технічним акцентом (>10%). Зв'язок з FORM CV: системне відстеження кінематики замість епізодичного. clinical-safety: NOT_REQUIRED. sports-scientist review required before publish. Blog card added.
- `README.md` — Editorial series 291–300 «Технічна якість і прихований прогрес» додана як новий proposed блок.

### Changed
- `blog.html` — картка post-291 додана на початок стрічки.
- `VERSION` — 8.73.0 → 8.74.0.

## [8.73.0] — 2026-06-25

### Added
- `content/post-2517-sleep-as-training-resource.md` — Серія «Між сесіями · 2516–2525» пост 2/10. «Сон як тренувальний ресурс: що ти втрачаєш, коли недосипаєш». Три механізми прив'язки адаптації до сну: (1) GH — 70–80% добової секреції під час SWS (Van Cauter et al. 2000); (2) тестостерон — тиждень сну по 5 год → -10–15% (Leproult & Van Cauter 2011); (3) консолідація моторних паттернів під час REM (Walker et al. 2002). Гострий дефіцит: сила -7–8%, RPE-дрейф вгору (Reilly & Piercy 1994). Хронічний дефіцит: порушення балансу MPS/MPB (Dattilo et al. 2011), втрата сигналу про втому (Van Dongen et al. 2003). Підвищена потреба: фази накопичення навантаження + перші 48 год після максимальних зусиль. Практичний мінімум: сталий час підйому (Vitale et al. 2019), 18–20°C + темрява, ≥2 год між інтенсивним тренуванням і сном, тижневий лог тривалості. Межі: сон не компенсує неправильний стимул. clinical-safety: NOT_REQUIRED. sports-scientist review pending. Blog card added.

### Changed
- `blog.html` — картка post-2517 додана на початок стрічки.
- `VERSION` — 8.72.1 → 8.73.0.

---

## [8.72.1] — 2026-06-25

### Changed
- `docs/DATA_ROOM.md` — v0.3: §Enterprise Schema added. Two enterprise-tier Postgres tables registered: `enterprise_renewals` (migration 0084, DATA_MODEL §43, SOC 2 artefacts REN-E-001/002/003) and `enterprise_churn_events` (migration 0085, DATA_MODEL §44, SOC 2 artefacts DEL-E-001/WBK-E-001/CHN-E-001). Closes DATA_MODEL §43.8 item 5 and §44.10 item 5.
- `docs/DATA_MODEL.md` — v1.29: §43.8 item 5 + §44.10 items 4–5 checklist sync. (1) §43.8 item 5 → `[x] Done` (DATA_ROOM §Enterprise Schema created). (2) §44.10 item 4 → `[x] Done` (DEL-E-001/WBK-E-001/CHN-E-001 confirmed registered in SOC2_READINESS v3.27.0 §102; R2 subfolders and Vanta mirror list confirmed). (3) §44.10 item 5 → `[x] Done` (DATA_ROOM §Enterprise Schema created).
- `VERSION` — 8.72.0 → 8.72.1.

---

## [8.72.0] — 2026-06-25

### Added
- `content/post-2516-between-sessions-adaptation.md` — Серія «Між сесіями · 2516–2525» пост 1/10. Зал — стимул; адаптація — між сесіями. MPS пік 3–5 год після тренування, вікно 24–48 год (Phillips et al. 2005). Нейральне відновлення 48–72 год для важких протоколів (Howatson & van Someren 2008). Три вектори-перешкоди: дефіцит сну (Leproult & Van Cauter 2011), системний стрес / кортизол → пригнічення mTORC1 (Drummond & Rasmussen 2008), між-сесійне фізичне навантаження поза обліком. Практичний фреймворк: трекати між-сесійний контекст (сон/стрес/активність) і корелювати з RPE наступного тренування. clinical-safety: NOT_REQUIRED. review_pending: sports-scientist.
- `README.md` — Editorial series 2516–2525 «Між сесіями» додано до roadmap (proposed · v8.72.0). 10 тем: між-сесійна адаптація, дні відпочинку, активне відновлення vs. повний відпочинок, когнітивне навантаження, сон, мобільна робота, уявна практика, cool-down, NEAT, синтез.
- `blog.html` — post-2516 card added at top of feed.

### Changed
- `VERSION` — 8.71.0 → 8.72.0.

---

## [8.71.0] — 2026-06-25

### Added
- `content/post-2449-reading-body-signals-without-listen-to-your-body.md` — Серія «Якість тренувального рішення · 2446–2455» пост 4/10. «Слухай своє тіло» без операційного визначення — реакція, а не рішення (Kahneman System 1). Чотири сигнали, що конвертуються в числа: (1) ΔRPE першого робочого підходу vs. журнальний базис (Gonzalez-Badillo & Sánchez-Medina 2010); (2) суб'єктивна швидкість перших повторень за шкалою 1–3 — проксі нейром'язового рекрутування (Gonzalez-Badillo et al. 2016); (3) час суб'єктивної готовності між підходами vs. особистий патерн (Buchheit & Laursen 2013); (4) технічна стабільність перших підходів — три маркери: постановка, ініціація першої фази, sticking point. Протокол рішення: три+ сигнали норма → повний план; два жовтих → -10%; три+ червоних → 50–60% об'єму. clinical-safety: NOT_REQUIRED. sports-scientist review pending.
- `blog.html` — post-2449 card added at top of feed.

### Changed
- `VERSION` — 8.70.1 → 8.71.0.

---

## [8.70.1] — 2026-06-25

### Changed
- `docs/OBSERVABILITY.md` — v5.1.5: R-44.11 item 3 cross-reference patch. Three stale "to be authored" R-44 references updated to `§R-44.5; v1.0, 2026-06-25` (§6.2 alert table, §12.6 pg_cron registry, §53.5 job spec cross-ref). v5.1.5 patch note added. Header v5.1.4 → v5.1.5.
- `docs/INCIDENT_RESPONSE.md` — v3.8: R-44.11 items 1 and 3 closed. Item 1 `[x] Done — AUDIT_LOG_SCHEMA.md v2.44`; item 3 `[x] Done — OBSERVABILITY.md v5.1.5`. R-44 v1.1 patch note added. Header v3.7 → v3.8.
- `VERSION` — 8.70.0 → 8.70.1.

---

## [8.70.0] — 2026-06-25

### Added
- `content/post-2448-when-to-change-exercise-algorithm.md` — Block 2446–2455 cluster «Якість тренувального рішення» пост 3/10. «Коли змінювати вправу, а коли — витримати: алгоритм без прив'язки до «втоми від рутини»». SAID принцип і learning curve (4–6 тижнів координаційної фази). Schoenfeld et al. (2016): адаптаційні відповіді накопичуються 12–16 тижнів. Чотири хибні причини зміни: нудьга (System 1, Kahneman 2011; Rebar et al. 2016 — автоматизм ≠ емоційне ставлення), нова «ефективніша» вправа, плато у вазі (запізнілий індикатор), страх травми без фактичних сигналів. Дві реальні причини: підтверджена механічна невідповідність і справжнє адаптаційне плато (всі 5 метрик стоять 4–6 тижнів + нормальне відновлення + деловантаж без ефекту). Алгоритм 3 питань. Правило двох тижнів. clinical-safety: NOT_REQUIRED. sports-scientist review pending. Blog card added.
- `blog.html` — post-2448 card added at top of feed.

### Changed
- `STATUS.md` — Block 2446–2455 progress updated to 3/10; v8.69.0 → v8.70.0.
- `VERSION` — 8.69.0 → 8.70.0.

---

## [8.69.0] — 2026-06-25

### Added
- `content/post-2447-tracking-progress-without-barbell-weight.md` — Block 2446–2455 cluster «Якість тренувального рішення» пост 2/10. «Як відстежувати прогрес без ваги на штанзі: п'ять метрик, що говорять більше за 1RM». Вага на штанзі як запізнілий індикатор адаптацій. П'ять метрик для self-coached атлета: (1) RPE при заданому навантаженні — зниження RPE = адаптація без нової ваги; (2) Volume Load (сума вага × повт. × підходи) — тижневий і блоковий тренд; (3) швидкість міжсетового відновлення — скорочення без падіння якості (Gonzalez-Badillo et al. 2016); (4) технічна стабільність під втомою — однорідність техніки між підходами; (5) суб'єктивна швидкість перших повторень (шкала 1–3; González-Badillo & Sánchez-Medina 2010). Три сигнали інтерпретації: прогрес без змін у вазі (продовжуй), стагнація по всіх фронтах (аналіз відновлення/контексту), регрес (деловантаж). Мінімальна послідовність введення метрик: 1–4 тижні RPE+VL → 5–8 + відновлення → 8+ + технічна стабільність + швидкість. clinical-safety: NOT_REQUIRED. sports-scientist review pending. Blog card added.
- `blog.html` — post-2447 card added at top of feed.

### Changed
- `README.md` — post-2446 → draft (retrofix) · post-2447 → draft · v8.69.0.
- `STATUS.md` — Block 2446–2455 progress updated to 2/10; current version v8.68.0 → v8.69.0.
- `VERSION` — 8.68.1 → 8.69.0.

---

## [8.68.1] — 2026-06-25

### Added
- `docs/SOC2_READINESS.md §111` — Cross-Reference Patch: CHN-OBS-E-001 (CC6.1/CC7.1/CC7.2) and DEL-OBS-E-001 (C1.2/CC7.2/A1.1) registered in §79.4 master consolidated evidence table; §80.3 `offboarding/` and `deletions/` folder annotations updated; §80.4 Vanta mirror list extended with both artefacts. Closes `docs/OBSERVABILITY.md §53.10 item 5` (P1/M11). §79.4 count: 65 → 67. SOC2_READINESS.md v3.31.0 → v3.36.0 (header corrected to reflect v3.32–v3.35 patches applied 2026-06-25). compliance-officer + enterprise-architect + devops-lead.

### Changed
- `docs/OBSERVABILITY.md §53.10` — item 5 marked `[x] Done — 2026-06-25 (SOC2_READINESS.md §111 v3.36.0)`. v5.1.3 → v5.1.4.
- `VERSION` — 8.68.0 → 8.68.1.

---

## [8.68.0] — 2026-06-25

### Added
- `content/post-2446-training-decision-vs-training-reaction.md` — Series 2446–2455 «Якість тренувального рішення» post 1/10. Тренувальне рішення vs. тренувальна реакція: де закінчується логіка і починається вгадування. Система 1 vs. Система 2 (Kahneman 2011) у тренувальному контексті. Чотири патерни реакції: коригування навантаження за відчуттям (Zourdos et al. 2016 — RPE систематично завищується на перших підходах), пропуск через настрій (Lally et al. 2010 — звичка незалежна від настрою; Rebar et al. 2016 — автоматизм кращий предиктор за мотивацію), спонтанний PR (Helms et al. 2016 — нейром'язова готовність до максимуму невідчутна суб'єктивно), зміна програми після поганого блоку. Ретроспективне виправдання як механізм ілюзії рішення (Libet et al. 1983). Pre-commitment: implementation intentions (Gollwitzer 1999, Gollwitzer & Sheeran 2006 мета-аналіз — подвоєння виконання). Три питання аудиту рішення. Три контексти де реакція виграє. Анонс серії 2447–2455. clinical-safety: NOT_REQUIRED. sports-scientist review pending. Blog card added.
- `README.md §2506–2515` — нова editorial series: «Програмна логіка без тренера: як self-coached атлет будує і переглядає структуру» (10 постів, proposed). Теми: ідеальна програма, MEV, лінійна прогресія, periodization без сезону, тренувальний тиждень, кількість вправ, обсяг vs. інтенсивність, умови перегляду програми, оцінка чужої програми, синтез.

### Changed
- `STATUS.md` — content engine table: серія 2446–2455 розпочата (пост 1/10 written); series 2506–2515 added to roadmap.
- `VERSION` — 8.67.0 → 8.68.0.

---

## [8.67.0] — 2026-06-25

### Added
- `content/post-2445-training-resilience-synthesis.md` — Series 2436–2445 «Тренування в реальних умовах» post 10/10 **SYNTHESIS**. Тренувальна стійкість self-coached атлета: синтез серії. Три рівні перешкод: фізіологічні (хвороба/детренінг — Mujika & Padilla 2000, Bruusgaard et al. 2010), логістичні (час/місце — Bickel et al. 2011 MEV), системні (паралельні стресори — Stults-Kolehmainen & Sinha 2014, McEwen алостатичне навантаження). П'ятикроковий класифікаційний протокол: тип перешкоди → сигнали тіла (RPE/HRV/sleep) → мінімальна доза (повна/50–70%/символічна/зупинка) → ціна пропуску (Mujika & Padilla: -7–12% сили за 2–4 тижні, відновлення 6–8 тижнів) → паттерн чи епізод. Три компоненти стійкості: правильна оцінка реальних втрат; системне мислення (Deci & Ryan 2000 ідентифікаційна мотивація: «я тренуюсь у вівторок», не бажання); оперативна гнучкість (символічна сесія — законне рішення). Анонс серії 2446–2455. **EDITORIAL SERIES 2436–2445 COMPLETE.** clinical-safety: NOT_REQUIRED. sports-scientist review pending. Blog card added.
- `docs/INCIDENT_RESPONSE.md §R-44` — R-44: Offboard Chain Monitor Stale (`offboard_chain_monitor` · job 44) — forty-fourth runbook. Closes `docs/OBSERVABILITY.md §53.10` item 6 (P1/M10). Job 44 (`0 * * * *`, 2h freshness) OFFBOARD-CHAIN-01 fleet-sweep stale recovery: P1 base → P0 on R-44-C2 confirming active breach (`offboarding_initiated_at IS NULL` past 24h). Four scope queries (R-44-C1 staleness, R-44-C2 P0 gate, R-44-C3 peer health, R-44-C4 job registration). Four root cause hypotheses (H1 job deleted/disabled, H2 form_system permission revoked, H3 pg_net degraded, H4 Supabase outage). Six-step recovery including Step 5 manual breach sweep and backfill (force offboarding initiation for affected tenants; manually emit `enterprise.offboard_chain_sla_breach` for stale-window record). Two DEC-030 HMAC-chained companion events: `system.offboard_chain_monitor_stale_declared` (HIGH/7yr — OffboardChainMonitorStaleDeclaredPayload) and `system.offboard_chain_monitor_restored` (STANDARD/3yr — OffboardChainMonitorRestoredPayload). OFFBOARD-CHAIN-MONITOR-STALE-CHAIN-01 ordering invariant. SOC 2 CC6.1 (OFFBOARD-CHAIN-01 MSA contractual SLA blind-spot coverage) + CC7.1 + CC7.2. R-44.11 four-item implementation checklist: 2× P0/M10 (AUDIT_LOG_SCHEMA registration + PagerDuty routing), 2× P1/M10–M11 (§12.6 cross-ref, SOC2_READINESS registration). Owner: devops-lead + compliance-officer.

### Changed
- `docs/AUDIT_LOG_SCHEMA.md` — v2.43 → v2.44: +2 R-44 companion DEC-030 events in `§Enterprise Post-Churn & Deletion SLA events`, closes R-44.11 item 1 (P0/M10). (1) `system.offboard_chain_monitor_stale_declared` (HIGH, 7yr): IC-emitted at R-44 T+0; OFFBOARD-CHAIN-MONITOR-STALE-CHAIN-01 prerequisite; `OffboardChainMonitorStaleDeclaredPayload` Zod schema. (2) `system.offboard_chain_monitor_restored` (STANDARD, 3yr): IC-emitted at R-44 Step 6; OFFBOARD-CHAIN-MONITOR-STALE-CHAIN-01 terminal event; `OffboardChainMonitorRestoredPayload` Zod schema. OFFBOARD-CHAIN-MONITOR-STALE-CHAIN-01 ordering invariant added. Emitter assignments: +2 rows. Alert routing: +2 rows. OFFL-CHAIN-01 note updated ("to be authored" → "authored 2026-06-25"). Privacy floor: integer counts only — no tenant_id, no employee data.
- `docs/OBSERVABILITY.md` — v5.1.2 → v5.1.3: §53.10 item 6 marked `[x] Done — 2026-06-25 (INCIDENT_RESPONSE.md R-44 v1.0; AUDIT_LOG_SCHEMA.md v2.44)`.
- `blog.html` — post-2445 card added at top of feed.
- `README.md` — post-2445 → draft · v8.67.0.
- `STATUS.md` — Current version v8.66.1 → v8.67.0; Series 2436–2445 marked **COMPLETE 10/10**; next: Editorial 2446–2455.
- `VERSION` — 8.66.1 → 8.67.0.

---

## [8.66.1] — 2026-06-25

### Changed
- `docs/AUDIT_LOG_SCHEMA.md` — v2.42 → v2.43: +2 DEC-030 events in `§Enterprise Post-Churn & Deletion SLA events` closing `docs/OBSERVABILITY.md §53.10` item 1 (P0/M10). (1) `enterprise.offboard_chain_sla_breach` (HIGH, 7yr): pg_cron job 44 `offboard_chain_monitor` emits per flagged tenant (`offboarding_initiated_at IS NULL` past 24h); `OffboardChainSlaBreachPayload` Zod schema: `tenant_id`, `churn_date`, `hours_since_churn`, `check_run_at`, `alert_dedup_key`; routes AL-OFFL-01 P1 PagerDuty `form-enterprise`; OFFL-CHAIN-01 ordering invariant (emit + HTTP 200 before PagerDuty). (2) `system.offboard_chain_check_passed` (LOW, 1yr): all-clear path, no `tenant_id` (privacy floor); `OffboardChainCheckPassedPayload`: `tenants_checked`, `check_run_at`; no alert. OFFL-CHAIN-01 invariant block, CC7.2 auditor narrative, and emitter/alert routing entries added. Section header updated to include `+ §53` and `CC7.2`.
- `docs/OBSERVABILITY.md` — v5.1.0 → v5.1.2: §53.10 item 1 marked `[x] Done — 2026-06-25 (AUDIT_LOG_SCHEMA.md v2.43)`; document header corrected (v5.1.1 was applied 2026-06-25 but title line not updated; corrected to v5.1.2).
- `STATUS.md` — Current version v8.66.0 → v8.66.1.
- `VERSION` — 8.66.0 → 8.66.1.

---

## [8.66.0] — 2026-06-25

### Added
- `content/post-2444-from-motivation-to-system.md` — Series 2436–2445 «Тренування в реальних умовах» post 9/10. Від «тренуватись коли хочеться» до тренувальної системи: операційний перехід. Мотивація — стан; система — структура незалежна від стану. Три ознаки мотиваційної моделі: рішення «іти чи ні» відкривається щоразу; пік активності після натхнення; відсутність мінімального протоколу. Операційні компоненти системи: фіксовані слоти; мінімальний протокол для поганих днів; тригер замість намічного часу. Послідовність переходу: фаза 1 (4 тижні, один слот без виключень) → фаза 2 (два слоти) → фаза 3 (повна структура). Психологія ідентифікації: «я тренуюсь у середу» vs. «хочу більше тренуватись». Тактика при нестабільному розкладі, дітях, проектних ривках. Victor: системна, а не мотиваційна логіка нотифікацій. clinical-safety: NOT_REQUIRED. sports-scientist review pending. Blog card added.
- `blog.html` — post-2444 card added at top of feed.
- `README.md` — Editorial series 2496–2505 «Числа і рішення: self-coached аналітика» proposed (мінімальна ефективна доза, читання журналу, e1RM vs. суб'єктивне, RPE як тренд, точка насичення обсягом, synthesis).

### Changed
- `STATUS.md` — Current version v8.65.1 → v8.66.0; Series 2436–2445 progress updated to 9/10; next: post-2445 (синтез серії).
- `VERSION` — 8.65.1 → 8.66.0.

---

## [8.65.1] — 2026-06-25

### Changed
- `docs/OBSERVABILITY.md` — §6.2 Consolidated Alert Rules: new `enterprise_post_churn_health` subsection inserted after `contract_renewal_health` (§51.6). Two alert rules: AL-OFFL-01 (job 44 `offboard_chain_monitor` hourly fleet-sweep; P1 `form-enterprise`; 4h dedup; CC6.1/CC7.1) and AL-DEL-01 (job 43 `deletion_sla_monitor` P1→P0 day-25/29 escalation; 6h dedup; C1.2/CC4.1/P8). SOC 2 mapping block for both criteria sets and monitoring-layer evidence artefacts (CHN-OBS-E-001, DEL-OBS-E-001). §53.10 item 4 marked `[x] Done`. OBSERVABILITY.md v5.1.0 → v5.1.1. Closes P0/M10 obligation from §53.10 item 4.
- `STATUS.md` — version v8.65.0 → v8.65.1.
- `VERSION` — 8.65.0 → 8.65.1.

---

## [8.65.0] — 2026-06-25

### Added
- `content/post-2443-detraining-recovery.md` — Block 2436–2445 «Тренування в реальних умовах» post 8/10. Детренінг — не тотальна втрата: три рівні стійкості адаптацій. Bruusgaard et al. (2010): міонуклеуси зберігаються після детренінгу — структурна основа м'язової пам'яті. Mujika & Padilla (2000): 2-4 тижні детренінгу → VO2max -4-14%, сила -7-12%, переважно нейронний механізм. Bickel et al. (2011): 1/3 від нормального об'єму підтримує силові адаптації 32 тижні. Три рівні стійкості: міонуклеарна база (роки), рухові патерни (місяці), аеробна база (тижні–місяці). Що детренується першим: плазматичний об'єм крові, нейром'язова синхронізація. Практичні темпи повернення: тиждень 1-2 нейрональна реактивація, 3-4 60-70% сили, 6-8 80-90%, 12-16 повне повернення. Критичний принцип: сполучна тканина відстає від м'язової сили — найчастіша причина травм при агресивному поверненні. Алгоритм повернення: тиждень 1 — 50-60% інтенсивності, RIR 4-5; поступова прогресія +5-10%/тиждень. Häkkinen & Komi (1983): реактивація нейром'язових адаптацій швидша за первинне формування. Стаган et al. (1991): зміни розподілу волокон IIa/IIx при детренінгу. clinical-safety: NOT_REQUIRED. sports-scientist review pending. Blog card added (series 2436–2445, пост 8/10).
- `blog.html` — post-2443 card added at top of feed.

### Changed
- `STATUS.md` — Current version v8.62.0 → v8.65.0; Series 2436–2445 progress updated to 8/10; next: post-2444.
- `VERSION` — 8.64.0 → 8.65.0.

---

## [8.64.0] — 2026-06-25

### Added
- `docs/OBSERVABILITY.md` — §53 Enterprise Post-Churn Lifecycle & GDPR Deletion SLA Observability. Closes the monitoring gap where COST_MODEL §43 (OFFBOARD-CHAIN-01/WINBACK-CHAIN-01/DELETION-CHAIN-01 invariants) and DATA_MODEL §44 (`enterprise_churn_events`) had no dedicated observability section. Three monitoring obligations addressed: (1) OFFBOARD-CHAIN-01 fleet-sweep gap — pg_cron job 44 `offboard_chain_monitor` (hourly; 2h freshness; AL-OFFL-01 P1 PagerDuty `form-enterprise`; emits `enterprise.offboard_chain_sla_breach` HIGH/7yr per breach, `system.offboard_chain_check_passed` LOW/1yr on all-clear; OFFL-CHAIN-01 ordering invariant); (2) DELETION-SLA-01 formalisation — AL-DEL-01 formally defined (formalises existing §12.6 job 43 alert); (3) WBKO-SLO-01 winback coverage target (≥ 90% Green/Amber churns with outreach within 90d; QBR governance, no automated P1). Twelve RED metrics, three SLOs (OFFL-SLO-01 zero-tolerance / OFFL-SLO-02 zero-tolerance / WBKO-SLO-01 commercial), two new evidence artefacts (CHN-OBS-E-001 CC6.1/CC7.1/CC7.2 quarterly; DEL-OBS-E-001 CC7.2/C1.2/A1.1 quarterly), Metabase `Enterprise Post-Churn Lifecycle Health` dashboard spec (10 panels; compliance_officer + enterprise_admin; tenant_manager HR excluded). §12.6 registry: job 44 row added; freshness window note extended; v1.7 patch note. v5.1.0.

### Changed
- `VERSION` — 8.63.0 → 8.64.0.

---

## [8.63.0] — 2026-06-25

### Added
- `content/post-2442-volume-reduction-vs-full-stop-algorithm.md` — Series 2436–2445 «Тренування в реальних умовах» post 7/10. «Зниження обсягу або повна зупинка: алгоритм вибору». Чотири функціональних стани тренування: повноцінне (100%) / знижений об'єм (50–70%) / символічне (1–2 підходи, RPE 7–8) / повна зупинка (0%). Bickel et al. (2011): 1/3 об'єму підтримує адаптацію 32 тижні. Ralston et al. (2017): MEV — 1–5 підходів/м'язова група для підтримки. Ogasawara et al. (2013) + Bruusgaard et al. (2010): міонуклеарна пам'ять — фізіологічна основа швидкого повернення. Алгоритм 4 запитань: хвороба/температура → нефункціональне перевтомлення → логістика/деловантаження → функціональна втома. Таблиця рішень: 7 контекстів × 4 параметри (стан / об'єм / інтенсивність / очікуваний результат). Конкретні числа: 60%, 30% від нормального об'єму в числах підходів. Moran et al. (2009) + Birrer & Sheon (2015): правило «нижче шиї» при хворобі. Barbalho et al. (2020): продовження тренування при нефункціональному перевтомленні подовжує відновлення. Blumert et al. (2007): екстремальне недосипання → зниження сили + ризик техніки. 4 типові помилки: зниження інтенсивності замість об'єму; деловантаження без підстав; зупинка без кінцевого критерію; «трохи знижу» при перевтомленні. Вплив тренувального стажу на вибір між зниженням і зупинкою. clinical-safety: NOT_REQUIRED.
- `blog.html` — post-2442 card added at top of feed.

### Changed
- `README.md` — post-2441 retromarked as **draft · v8.62.0 → post-2441-three-types-bad-training.md**; post-2442 → **draft · v8.63.0 → post-2442-volume-reduction-vs-full-stop-algorithm.md**.
- `STATUS.md` — Series 2436–2445 progress updated 6/10 → 7/10; next: post-2443 (Відновлення після детренінгу).
- `VERSION` — 8.62.0 → 8.63.0.

---

## [8.62.0] — 2026-06-25

### Added
- `content/post-2441-three-types-bad-training.md` — Series 2436–2445 "Тренування в реальних умовах" post 6/10. Три типи «поганого тренування»: (1) мотиваційний — психологічна флуктуація, нейтральний фізіологічно, протокол: повна розминка → оцінка першого підходу → виконати за планом (Deci & Ryan 2000; Hallam et al. 2018; Kilpatrick et al. 2015); (2) фізіологічний — реальний сигнал системи, RPE +2 на звичних вагах, нефункціональне перевтомлення (Halson 2014; Saw et al. 2016; Barbalho et al. 2020), протокол: скоротити або зупинитись; (3) структурний — зовнішня логістична перешкода, власний стан нормальний (Raastad et al. 2000), протокол: адаптувати сесію. Критична помилка: атрибутувати тип 2 як тип 1. Класифікаційна таблиця 3×5. Двохвилинний протокол класифікації. clinical-safety: NOT_REQUIRED. sports-scientist review pending.
- `blog.html` — post-2441 card added at top of feed.

### Changed
- `VERSION` — 8.61.2 → 8.62.0.

## [8.61.2] — 2026-06-25

### Changed
- `docs/OBSERVABILITY.md` — §26.8 `fleet_sso_health` subsection added (AL-SSO-FLEET-01 alert rule: pg_cron job 38, ≥ 3 tenants breaching SSO-SLO-01, PagerDuty `form-platform`, dedup `al-sso-fleet-01-{15min_epoch}` 30-min cooldown, `sla_credit_impact: 'none'` DEC-070 hard invariant); §6.2 consolidated alert rules table extended with `fleet_sso_health` subsection; §26.11 CC7.2 named-rule count twenty-one → twenty-two + AL-SSO-FLEET-01 added; SSO-OBS-E-007 and SSO-FLEET-E-001 artefact rows added to §26.11 evidence table; §48.8 item 4 and §49.9 items 4–5 marked done. Closes three overdue P1 documentation obligations (§48.8 item 4, §49.9 items 4–5). v5.0.2.
- `VERSION` — 8.61.1 → 8.61.2.

## [8.61.1] — 2026-06-25

### Changed
- `docs/SOC2_READINESS.md` — §110 Cross-Reference Patch: registered IR-AUTO-E-001 (CC7.4/CC7.3, quarterly `incident.opened` T+0 latency sample) and IR-AUTO-E-002 (CC7.4, quarterly linkage latency distribution) in §79.4 master evidence table; added both to §80.4 Vanta mirror list; appended §110 section. §79.4 count: 63 → 65 rows. Closes INCIDENT_RESPONSE §17.11 item 11 and R-27.11 item 6.
- `docs/INCIDENT_RESPONSE.md` — §17.11 item 11 marked `[x] Done — 2026-06-25, SOC2_READINESS.md §110 (v8.61.1)`; R-27.11 item 6 marked `[x] Done — 2026-06-25, §81.12 OQ-EVD-02 confirmed 🟢 Resolved (R-27 authoring 2026-06-14); §110 (v8.61.1)`.
- `VERSION` — 8.61.0 → 8.61.1.

## [8.61.0] — 2026-06-25

## [8.60.1] — 2026-06-25

### Changed
- `docs/SOC2_READINESS.md` — §109 Cross-Reference Patch: registered CAEP-E-001 (CC6.3/A1.1/CC7.4), CC6-E-CAEP-001/002/003/004 (CC6.3/CC6.6/CC7.2/CC7.3), SSO-FLEET-E-001 (CC7.2/CC7.3) in §79.4 master evidence table; closes §94.6 items 7-8, §95.6 items 4-5, §97.6 items 1-3. §79.4 count: 57 → 63 rows.
- `VERSION` — 8.60.0 → 8.60.1.

## [8.60.0] — 2026-06-25

### Added
- `content/post-44-training-age-identity.md` — Editorial series post 44. «Тренувальний вік: ким ти є як атлет — насправді». Тренувальний вік як функціональна змінна (Zatsiorsky & Kraemer 1995/2006): рівень накопиченої структурної адаптації і здатність тіла відповідати на прогресуючий стимул. Три рівні (beginner/intermediate/advanced) з операційними маркерами розпізнавання: реакція на лінійну прогресію, швидкість прогресії, відновлення між сесіями, технічна стабільність під навантаженням, здатність до авторегуляції (Zourdos et al. 2016). Пастка «просунутого початківця»: термінологічна грамотність без відповідного адаптивного базису. Нейром'язова адаптація у початківців — Sale (1988): 20–40% сили за 6–8 тижнів без структурних змін. Meeusen et al. (2013) ECSS/ACSM: механізм overreaching при невідповідності стимулу рівню адаптованості. Hubal et al. (2005): варіабельність відгуку -2% → +59% в однорідній групі. Schoenfeld, Ogborn & Krieger (2016): частота vs. об'єм і залежність від рівня. Halson (2014): деловантаження як рівнезалежна змінна. Практичний 3-крок для чесної оцінки рівня + зв'язок з програмною логікою FORM/Victor. clinical-safety: PASS. sports-scientist review pending. Blog card date updated.
- `README.md` — Editorial series header updated: 11–42 → 11–44 drafted. Post-44 → draft · v8.60.0. Editorial series 2476–2485 (Рішення self-coached атлета: коли і як змінювати курс) proposed.

### Changed
- `STATUS.md` — post-44 written noted in content library row 11–50.
- `blog.html` — post-44 card date updated 2026-06-15 → 2026-06-25.
- `VERSION` — 8.59.0 → 8.60.0.

---

## [8.59.0] — 2026-06-25

### Added
- `content/post-2439-non-standard-week-minimal-stimulus.md` — Editorial series post 2439 (пост 4/10 серії 2436–2445 «Тренування в реальних умовах»). «Нестандартний тиждень: як зберегти мінімальний тренувальний стимул без повноцінного розкладу». Три реакції на збій розкладу і чому компенсаційне перевантаження та повна відмова — хибні рішення. MEV (мінімальний ефективний об'єм): Bickel et al. (2011, MSSE) — 1/3 від нормального об'єму підтримує силові адаптації 32 тижні у тренованих атлетів. Ієрархія при обмеженому ресурсі: інтенсивність → частота → об'єм. Ogasawara et al. (2013) + Bruusgaard et al. (2010): м'язова пам'ять і збереження міонуклеарного ядра. Операційна рамка: аудит доступних вікон (0/1/2/3) → вибір стратегії → виконання без компенсації. Мінімальна сесія 20–30 хв: 2 підходи складних рухів RPE 8–9. Що «не робити»: не надолужувати після, не змінювати програму через збої, не жертвувати інтенсивністю заради об'єму. clinical-safety: NOT_REQUIRED. sports-scientist review pending. Blog card added.
- `blog.html` — post-2439 card added at top of feed.

### Changed
- `VERSION` — 8.58.1 → 8.59.0.

---

## [8.58.1] — 2026-06-25

### Changed
- `docs/SOC2_READINESS.md` — §108 Cross-Reference Patch: registered `SCA-STALE-E-001` (quarterly `sca_sla_monitor` stale-incident health report, CC7.2/CC6.8/CC7.1) in §79.4 master evidence table (row 57); added `security/sca-sla-monitor-stale/` to §80.3 R2 folder map; added SCA-STALE-E-001 to §80.4 Vanta mirror list. Closes INCIDENT_RESPONSE R-42.11 item 4 (P1/M10).
- `docs/INCIDENT_RESPONSE.md` — R-42.11 item 4 marked `[x] Done — 2026-06-25, SOC2_READINESS.md §108 (v3.33.0)`.
- `VERSION` — 8.58.0 → 8.58.1.

---

## [8.58.0] — 2026-06-25

### Added
- `content/post-2438-travel-training-hotel-park-minimal-context.md` — Editorial series 2436–2445 «Тренування в реальних умовах» post 3/10. «Тренування в дорозі: готельний номер, парк, мінімальний контекст — операційна рамка». Категоріальна помилка тревел-тренінгу: готельна кімната не є деградованою версією залу — це інший контекст. Правильна ціль — maintenance, не progress. Mujika & Padilla (2000): при зниженні обсягу на 60–90% тренована людина підтримує адаптації до 15 тижнів при збереженні інтенсивності і частоти. Graves et al. (1988): 1–2 силові сесії/тиждень з правильним RPE утримують силові адаптації 12 тижнів. Чотири сценарії контексту: (1) готельний номер без залу — болгарські присідання, відіжмання на підвищенні, hip thrust; (2) готельний зал — адаптовані вправи, full-body формат; (3) парк — турнік, брусья, біг; (4) поїздка до 3 днів — часто раціональніше не тренуватись. П'ять операційних принципів: maintenance не progress; інтенсивність вища за обсяг; 35–45 хв достатньо; планувати до, не під час; RPE як єдиний реалістичний орієнтир. Алгоритм «коли не тренуватись»: джетлаг 5+год, пікове когнітивне навантаження, нездужання, 1–2-денна поїздка. Конкретний bodyweight протокол для номера без залу. clinical-safety: NOT_REQUIRED. Blog card added.

### Changed
- `blog.html` — post-2438 card added at top of feed; series 2436–2445 progress 3/10.
- `README.md` — Editorial series 2466–2475 (Самокоригування в тренінгу) proposed · v8.58.0.
- `STATUS.md` — content engine updated: post-2438 draft · series 2436–2445 progress 3/10 · next: post-2439.
- `VERSION` — 8.57.0 → 8.58.0.

---

## [8.57.0] — 2026-06-25

### Added
- `content/post-2437-missed-week-training-impact.md` — Editorial series 2436–2445 «Тренування в реальних умовах» post 2/10. «Пропущений тиждень: скільки шкоди завдає одна перерва — і чому ти переоцінюєш втрату». Один пропущений тиждень = 2% від 48-тижневого тренувального року. Mujika & Padilla (2000): 7 днів — нижня межа короткострокового детренінгу. Три компоненти за 7 днів: нейром'язова ефективність знижується (відновлення 2–5 сесій); VO₂max -1–3% (відновлення 1–2 тижні); м'язова маса — практично без змін. Fleck & Kraemer (2014): атрофія волокон за 1–2 тижні у тренованих атлетів відсутня. Bruusgaard et al. (2010): міоядра зберігаються місяцями — основа м'язової пам'яті. Психологічні механізми переоцінки втрати: зниження пампу, зміна відчуття ваги, ефект контрасту з піком. Протокол повернення (без хвороби): 1-ша сесія 70–80% об'єму RIR 3–4; 2-га 85–90%; 3-тя — повне повернення. Реальний ризик паузи — розрив ритму, не детренінг. Рішення: конкретна дата наступної сесії до кінця паузи. clinical-safety: NOT_REQUIRED. Blog card added.

### Changed
- `blog.html` — post-2437 card added at top of feed; series 2436–2445 progress 2/10.
- `STATUS.md` — content engine updated: post-2437 draft · series 2436–2445 progress 2/10 · next: post-2438.
- `VERSION` — 8.56.1 → 8.57.0.

---

## [8.56.1] — 2026-06-25

### Changed
- `docs/SOC2_READINESS.md` — §107 Cross-Reference Patch: registered 5 PAM evidence artefacts (CC6-E-PAM-001 through CC6-E-PAM-004 + CC7-E-PAM-001) from OBSERVABILITY §29.9 into §79.4 master evidence table (rows 52–56), added `pam/` subfolder to §80.3 R2 folder map with `form_api NO ACCESS`, updated §80.4 Vanta mirror protocol, and closed open obligations OQ-PAM-OBS-01 + OQ-PAM-OBS-02. Criteria covered: CC6.1 / CC6.2 / CC6.3 / CC6.6 / CC6.7 / CC7.2 / CC7.3. Doc version v3.31.0 → v3.32.0.
- `VERSION` — 8.56.0 → 8.56.1.

---

## [8.56.0] — 2026-06-25

### Added
- `content/post-2436-training-after-illness.md` — Editorial series 2436–2445 «Тренування в реальних умовах» post 1/10. «Тренування після хвороби: скільки реально втрачається і як повертатись без зайвого агресивного рампу». Mujika & Padilla (2000): деtренінг до 4 тижнів — переважно нейром'язовий, не структурний. Coyle et al. (1984): після 2 тижнів сила -10–15%, поперечний переріз м'яза без змін. Häkkinen & Komi (1983): вибухово-силові адаптації чутливіші до паузи за ізометричну силу. Три фази повернення: (0) критерій готовності; (1) перша сесія 40–50% об'єму RIR 4–5; (2) 2–4 сесії 60–80% об'єму; (3) повне повернення за 1–3 тижні. Додатково: респіраторна хвороба +2–3 дні до фази 0. Таблиця очікуваних термінів відновлення. clinical-safety: NOT_REQUIRED. Blog card added.

### Changed
- `blog.html` — post-2436 card added at top of feed; Editorial series 2436–2445 started.
- `README.md` — post-2436 → draft · v8.56.0; Editorial series 2456–2465 (Тренувальна самодіагностика) proposed.
- `STATUS.md` — content engine row updated: post-2436 draft · series 2436–2445 progress 1/10 · next: post-2437.
- `VERSION` — 8.55.0 → 8.56.0.

---

## [8.55.0] — 2026-06-25

### Added
- `content/post-2435-self-coached-athlete-minimal-infrastructure-synthesis.md` — Block 2426–2435 series post 10/10 · **СЕРІЯ ЗАВЕРШЕНА**. «Синтез 2426–2435: мінімальна інфраструктура self-coached атлета». Чотири шари інфраструктури: (1) Логування — мінімальний запис вага×повторення×RPE + суб'єктивна оцінка техніки 1–3; (2) Планування блоку — первинна ціль у метриці + maintenance dose + порядок кардіо/сили + розклад; (3) Операційні рішення тижня — авторегуляція при каліброваному RPE, страховка для press RPE 8+, відкладений відеоперегляд; (4) Ревізія блоку — стандарти як карта, переоцінка, коригування. Мінімальна 4-крокова версія для початківців системної практики. clinical-safety: NOT_REQUIRED. Blog card added.

### Changed
- `blog.html` — post-2435 added at top of feed; СЕРІЯ 2426–2435 COMPLETE.
- `STATUS.md` — series 2426–2435 COMPLETE 10/10; next: Editorial 2436–2445 «Тренування в реальних умовах».
- `VERSION` — 8.54.0 → 8.55.0.

---

## [8.54.0] — 2026-06-25

### Added
- `content/post-2433-video-mirror-or-nothing-feedback-without-coach.md` — Block 2426–2435 series post 8/10. «Відео, дзеркало або нічого: три підходи до зворотного зв'язку без тренера». Три рівні зворотного зв'язку (cueing / feedback / evaluation) та їх відображення на доступні інструменти. Ste-Marie et al. (2012): одночасний відеозворотний зв'язок погіршує моторне навчання, відкладений — ефективний. Peh et al. (2011): дзеркало під час складних рухів порушує моторне програмування. Tran et al. (2010): точність RPE формується систематичним логуванням. Таблиця кутів камери per ліфт; операційна карта 5 ситуацій. clinical-safety: NOT_REQUIRED. Blog card added.
- `content/post-2434-conflicting-training-goals-priority-algorithm.md` — Block 2426–2435 series post 9/10. «Коли дві тренувальні цілі суперечать одна одній: алгоритм пріоритету». Wilson et al. (2012): конкурентне тренування знижує гіпертрофію ~31%, силу ~18%. Atherton et al. (2005): AMPK-mTOR молекулярний перемикач. Три рівні конфліктів (сумісні / частковий / жорсткий) + Schumann et al. (2014). П'ятикроковий алгоритм пріоритету: назвати ціль у метриці → класифікувати конфлікт → структурне рішення → розподіл 80/20 → переоцінка в кінці блоку. Maintenance dose: Ralston et al. (2017). clinical-safety: NOT_REQUIRED. Blog cards added.

### Changed
- `blog.html` — posts 2433–2434 added at top of feed.
- `STATUS.md` — series 2426–2435 progress updated to 9/10; next: post-2435 synthesis (Синтез 2426–2435: мінімальна інфраструктура self-coached атлета).
- `VERSION` — 8.53.1 → 8.54.0.

---

## [8.53.1] — 2026-06-25

### Added
- `docs/SOC2_READINESS.md §106` — Cross-Reference Patch: OBSERVABILITY §52 (`SCA-OBS-E-001/002` · CC6.8 / CC7.1 / CC7.2 / CC9.2 / A1.1) + §80.3 R2 folder retroactive additions (`api-key-auth/`, `victor-safety/`, `sca/`). Closes gap where OBSERVABILITY §52.8 (v5.0.0, 2026-06-23) defined SCA-OBS-E-001 and SCA-OBS-E-002 with §54.9 registration but §79.4 master consolidated evidence table was never updated. §79.4: +2 rows (count 49 → 51). §80.3: +3 folders (`api-key-auth/` and `victor-safety/` retroactive from §105; `sca/` new). §80.4: SCA-OBS-E-001/002 added to Vanta mirror list. SCA-OBS-E-001 (CC6.8/CC7.1/CC9.2 — quarterly SCA SLO report with SCA-SLO-01 zero-tolerance breach count, SCA-SLO-02 High CVE 7d SLA rate, SCA-SLO-03 CI pass rate, AL-SCA-01 activation log; zero-breach quarters = positive zero-tolerance attestation). SCA-OBS-E-002 (CC7.2/A1.1 — quarterly job 42 `sca_sla_monitor` pg_cron run history; zero-stale-event quarters confirm no SCA detection blind-spot). Document header v3.30.0 → v3.31.0.

### Changed
- `docs/OBSERVABILITY.md §52.10 item 4` — updated to note §79.4 registration via §106 (2026-06-25) in addition to §54.9 registration (2026-06-23).
- `VERSION` — 8.53.0 → 8.53.1.

---

## [8.53.0] — 2026-06-25

### Added
- `content/post-2432-training-time-of-day.md` — Editorial series 2426–2435 «Інструментарій self-coached атлета» post 7/10. «Час тренування протягом дня і його вплив на результат: що каже наука». Циркадний ритм і фізична продуктивність: пік між 14:00–20:00 для більшості метрик сили (Atkinson & Reilly, 1996) — ефект 3–8% для сили, до 10% для вибухових рухів. Механізм: ендогенний ріст температури тіла (Racinais & Oksa, 2010). Хронотип «сова» — gap до 26% між ранком і вечором (Facer-Childs & Brandstaetter, 2015). Ключова знахідка: послідовність часу тренування скорочує ефект циркадного дефіциту — циркадна адаптація (Kübler et al., 2019). Спростування трьох мітів: голодний ранок і жироспалювання, ранковий тестостероновий пік як аргумент для гіпертрофії, вечірнє тренування і сон (Stutz et al., 2019 мета-аналіз). Операційна рамка: послідовність у вибраному часі > «правильний» час без послідовності. clinical-safety: NOT_REQUIRED. sports-scientist review pending.
- `blog.html` — post-2432 card added at top of feed.

### Changed
- `STATUS.md` — content engine row updated: post-2432 draft · series 2426–2435 progress 7/10 · next: post-2433.
- `VERSION` — 8.52.0 → 8.53.0.

---

## [8.52.0] — 2026-06-25

### Added
- `docs/AUDIT_LOG_SCHEMA.md §SCA stale-monitor incident events` — new section with `system.sca_monitor_stale_declared` (HIGH/7yr) and `system.sca_monitor_stale_restored` (STANDARD/3yr); Zod v2 schemas (ScaMonitorStaleDeclaredSchema, ScaMonitorStaleRestoredSchema); SCA-STALE-CHAIN-01 ordering invariant (HTTP 422 `SCA_STALE_CHAIN_01_VIOLATION` on inversion → R-05); retention table +2 rows. Closes `docs/INCIDENT_RESPONSE.md R-42.11` item 1 (P0/M9). Document header v2.40 → v2.42 (corrects v2.41 header gap).
- `content/post-2431-solo-vs-training-partner.md` — Editorial series 2426–2435 post 6/10. «Соло проти партнера: що насправді впливає на результат self-coached атлета». Три механізми де партнер реально важливий: (1) підстраховка у важких підходах зі штангою — Barbalho et al. (2017, JSCR): атлети без страхувальника залишають 1.3–1.8 зайвих повторень «в запасі»; (2) accountability для атлетів з проблемою послідовності — Annesi (1999): +16 відсоткових пунктів retention через 6 місяців; (3) зворотний зв'язок по техніці, але лише від технічно грамотного партнера. Підтверджуючі дослідження: Zajonc (1965) соціальна фасилітація; Rhea et al. (2003) +8% обсяг у групі з партнером, але різниця у наближенні до відмови, не базовому навантаженні; Strother et al. (2022) змагальний контекст значимий при RPE 8–9, незначний при RPE 6–7. Операційна карта рішення: два питання (пресовані штангові рухи + RPE 9+? послідовність — реальна проблема?). Логістичні trade-offs: узгодження розкладу, темп тренування, ризик залежності. clinical-safety: NOT_REQUIRED. sports-scientist review pending.
- `blog.html` — post-2431 card added at top of feed.

### Changed
- `docs/INCIDENT_RESPONSE.md R-42.11` — item 1 status updated [ ] → [x] *(AUDIT_LOG_SCHEMA.md v2.42 patch, 2026-06-25)*.
- `STATUS.md` — Editorial series 2426–2435 progress updated; post-2431 → draft.
- `VERSION` — 8.51.0 → 8.52.0.

---

## [8.51.0] — 2026-06-25

### Added
- `docs/INCIDENT_RESPONSE.md` R-42 — SCA SLA Monitor Stale (`sca_sla_monitor` · job 42 · `*/15 * * * *` · 20-min freshness). Forty-second IC runbook. Closes the §12.6 documentation gap: job 42 was the only pg_cron compliance job registered in OBSERVABILITY §12.6 (v1.3 patch, 2026-06-23) without a corresponding INCIDENT_RESPONSE stale runbook. R-42 covers the blind-spot scenario where job 42 exceeds its 20-min freshness threshold, disabling AL-SCA-01 detection and allowing open Critical CVEs to exceed the 24h SCA-SLO-01 remediation SLA without automated alert. P1 base severity; P0 escalation if R-42-C1 (`open_critical_sla_breached ≥ 1`) at declaration. Five root cause hypotheses (H1 deleted/disabled, H2 SQL exception, H3 pg_net degraded, H4 `form_system` permission revoked, H5 Supabase outage). Four scope queries (R-42-C1 P0 gate, R-42-C2 stale window, R-42-C3 peer health, R-42-C4 registration). DEC-030 HMAC-chained events: `system.sca_monitor_stale_declared` HIGH/7yr and `system.sca_monitor_stale_restored` STANDARD/3yr; SCA-STALE-CHAIN-01 ordering invariant (HTTP 422 `SCA_STALE_CHAIN_01_VIOLATION` on breach → R-05). Evidence: SCA-STALE-E-001 quarterly pg_cron job 42 health report (CC7.2/CC6.8, 3yr). SOC 2: CC6.8 / CC7.1 / CC7.2. Owner: security-engineer + compliance-officer.

### Changed
- `docs/INCIDENT_RESPONSE.md` — header v3.6 → v3.7.
- `docs/OBSERVABILITY.md` — §12.6 job 42 `sca_sla_monitor` stale-consequence cross-ref column extended with `INCIDENT_RESPONSE R-42 (job 42 stale recovery runbook — §R-42.5)`; v1.6 patch note added. Closes R-42.10 post-incident control and R-42.11 item 3.
- `VERSION` — 8.50.0 → 8.51.0.

---

## [8.50.0] — 2026-06-25

### Added
- `content/post-2430-autoregulation-when-premature-when-necessary.md` — Editorial series 2426–2435 «Інструментарій self-coached атлета» post 5/10. Авторегуляція (RPE/RIR): не краща версія відсоткового тренінгу, а інший інструмент. Чому для новачка передчасна: суб'єктивне сприйняття зусилля — навичка, яка напрацьовується (Ludbrook & Bhatt 2022: значна розбіжність RPE vs. об'єктивних показників у атлетів < 1 року). Де стає необхідною: intermediate-рівень де лінійна прогресія вичерпується, висока добова варіабельність відновлення (Hamacher et al. 2019), перехід до блокової моделі. Алгоритм поступової інтеграції: фіксовані числа (рік 1) → hybrid ±5% (intermediate рано) → RPE з журналом калібрування (deep intermediate) → повна авторегуляція (advanced, блокова модель). Colquhoun et al. (2017), Zourdos et al. (2016). clinical-safety: NOT_REQUIRED. sports-scientist review pending. Blog card added.
- `blog.html` — post-2430 card added at top of feed.
- `README.md` — Editorial series 2446–2455 «Якість тренувального рішення: як self-coached атлет перестає угадувати» proposed (10 posts). README: post-2429 → draft, post-2430 → draft.

### Changed
- `STATUS.md` — content engine row updated: post-2430 draft · series 2446–2455 proposed · next: post-2431.
- `VERSION` — 8.49.0 → 8.50.0.

---

## [8.49.0] — 2026-06-25

### Added
- `content/post-2429-strength-standards-as-reference.md` — Editorial series 2426–2435 «Інструментарій self-coached атлета» post 4/10. Strength standards як орієнтир: Symmetry Strength, ExRx, кратні BW — грубий контекст відносної сили. Три типові помилки: стандарт як ціль, порівняння між різними системами, ігнорування антропометрії і тренувального контексту. Де реально корисні: фільтр фази прогресії (novice → лінійна прогресія; intermediate → мезоцикли; advanced → блокова модель); виявлення диспропорцій між вправами. Практичний алгоритм: одна система, перевірка раз на блок, профіль диспропорцій, не окремий ліфт. Обмеження: selection bias у даних, відсутність контролю техніки та антропометрії, тренувальний вік не врахований. clinical-safety: NOT_REQUIRED. sports-scientist review pending. Blog card added.
- `blog.html` — post-2429 card added at top of feed.

### Changed
- `VERSION` — 8.48.2 → 8.49.0.

---

## [8.48.2] — 2026-06-25

### Changed
- `docs/SOC2_READINESS.md §105` — Cross-Reference Patch: OBSERVABILITY §31 + §32 (APIKEY-E-001/002/003/004/005 + VSAFETY-E-001/002/003/004 · CC6.1 / CC6.2 / CC6.4 / CC6.8 / CC7.2 / CC7.3 / CC7.4 / A1.2). Closes two P1/M6 cross-reference obligations unfulfilled since OBSERVABILITY v2.1/v2.2 (2026-06-10). §105.1 source: OBSERVABILITY §31.9 (APIKEY-E) and §32.9 (VSAFETY-E). §105.2 APIKEY-E artefact table: CC6.2 scope enforcement code review (E-001), CC6.4 DEC-030 rotation chain (E-002), CC6.8 IP enforcement change audit (E-003), CC7.2 PagerDuty `api_key_health` incident history (E-004), CC6.1 IP block events export (E-005). §105.3 VSAFETY-E artefact table: CC7.2 FORM-VICTOR-001 PagerDuty config + incident history (E-001), CC7.3 `VICTOR_SAFETY_TELEMETRY` observation export (E-002), CC7.4 VSAFETY-CHAIN attestation (E-003), A1.2 KV + `ai.victor_*` DEC-030 export (E-004). §105.4 §79.4 additions: 9 rows; pre-§105 count 40 → post-§105 count 49. §105.5 cross-reference obligations closed: §31.10 item 12 🟢; §32.10 item 11 🟢; VSAFETY-E absent from MASTER-INDEX 🟢. §105.6 four-item P1 checklist (M6 calendar wiring + annual chain attestation). SOC2_READINESS v3.29.0 → v3.30.0.
- `docs/OBSERVABILITY.md §31.10 item 12` — Status `[ ]` → `[x] Done` (SOC2_READINESS §105, 2026-06-25).
- `docs/OBSERVABILITY.md §32.10 item 11` — Status `[ ]` → `[x] Done` (SOC2_READINESS §105, 2026-06-25).
- `VERSION` — 8.48.1 → 8.48.2.

---

## [8.48.1] — 2026-06-25

### Changed
- `docs/SOC2_READINESS.md §104` — Cross-Reference Patch: INCIDENT_RESPONSE R-41 (`deletion_sla_monitor` job 43 · DEL-MON-E-001 CC7.2 Extension · C1.2 / CC4.1 / CC7.2). Closes the CC7.2 criterion gap in the DEL-MON-E-001 §79.4 registration (v8.45.0, 2026-06-25 registered C1.2/CC4.1 only; INCIDENT_RESPONSE R-41.9 maps CC7.2 via `pg-cron-health-monitor` §12.7 detection + DEC-030 detection-to-restoration timeline + HMAC-VERIFY-ALGO-001). §104.2 full artefact table with auditor narratives for all three criteria. §104.3 criteria mapping: CC7.2 added. §104.4 §79.4 row amendment: `C1/CC4 | C1.2/CC4.1` → `C1/CC4/CC7 | C1.2/CC4.1/CC7.2`. §104.5 cross-reference obligations closed (CC7.2 gap 🟢; R-41.11 item 6 🟢). §104.6 one-item P1 checklist (M11 first quarterly filing). SOC2_READINESS v3.28.0 → v3.29.0.
- `docs/INCIDENT_RESPONSE.md R-41` — R-41.10 §104 cross-reference row added. R-41.11 item 6 added [x] Done (SOC2_READINESS §104 v3.29.0). Footer v1.0 → v1.1. Document header v3.5 → v3.6.
- `VERSION` — 8.48.0 → 8.48.1.

---

## [8.48.0] — 2026-06-25

### Added
- `content/post-2428-cardio-and-strength-same-week.md` — Серія 2426–2435 «Інструментарій self-coached атлета» пост 3/10. Кардіо і силові на одному тижні: interference effect. Hickson (1980): оригінальне дослідження при 6 аеробних сесіях/тиждень — конфлікт реальний. Механізм: AMPK (аеробний сигнал) пригнічує mTOR (силовий). Wilson et al. (2012) мета-аналіз: interference effect значущий лише при великих обсягах і неправильній організації. Де конфлікт реальний: 4+ кардіо-сесії/тиждень, відсутність буфера, бігове кардіо при важкому нижньому тілі. Де синергія: аеробна база покращує відновлення між підходами; Zone 2 між силовими — активне відновлення. Де байдуже: рекреаційні обсяги, 24-48 год буфер, помірна інтенсивність. Три операційних питання: пріоритет адаптації, модальність кардіо (велосипед vs. біг при важких ногах), послідовність і часовий буфер. Zone 2 vs HIIT — чому HIIT більш interferent. Halson (2014) — аеробна підготовленість як фактор відновлювальної ємності. clinical-safety: NOT_REQUIRED. sports-scientist review pending.
- `README.md` — Editorial series 2436–2445 (Тренування в реальних умовах: коли план зустрічає дійсність) додано до роадмапу · proposed.
- `blog.html` — post-2428 card додано на початок фіду.

### Changed
- `README.md` — post-2428 → draft · v8.48.0 у серії 2426–2435.
- `STATUS.md` — content engine row оновлено: post-2428 draft · v8.48.0; series 2436–2445 proposed. Next: post-2429 (strength standards).
- `VERSION` — 8.47.0 → 8.48.0.

---

## [8.47.0] — 2026-06-25

### Added
- `content/post-2427-barbell-dumbbell-bodyweight-choice.md` — Серія 2426–2435 «Інструментарій self-coached атлета» пост 2/10. Вибір між штангою, гантелями і власною вагою для рекреаційного атлета без конкретної спеціалізації. Чому питання сформульоване неправильно: Schoenfeld et al. (2020) — гіпертрофічні адаптації між вільними вагами і тренажерами статистично незначущі при контрольованому об'ємі. Де формати справді відрізняються: точність прогресії (штанга — крок 2.5 кг; гантелі — кроки 2–4 кг; власна вага — нелінійна); білатеральний дефіцит (Maffiuletti et al., 2011: 10–15% у нетренованих); технічний поріг; доступність. Де кожен формат виграє. Практичний алгоритм вибору: де ти реально тренуватимешся × що реально виконуватимеш послідовно. Fritz et al. (2022) — найбільший предиктор довгострокової адаптації — дотримуваність, не оптимальність протоколу. Типові помилки у змішаних підходах. clinical-safety: NOT_REQUIRED. sports-scientist review pending.
- `blog.html` — post-2427 card додано на початок фіду.

### Changed
- `README.md` — post-2427 → draft · v8.47.0 у серії 2426–2435.
- `STATUS.md` — content engine row оновлено: post-2427 draft · v8.47.0; next: post-2428 (кардіо і силові на одному тижні).
- `VERSION` — 8.46.1 → 8.47.0.

---

## [8.46.1] — 2026-06-25

### Changed
- `docs/SSO_SCIM_IMPLEMENTATION.md` v2.8.2 — job-number correction and IR cross-reference backfill. (1) `bdg_override_expiry_sweep` job number corrected "job 33" → **"job 34"** in §34.3 (SQL comment + COMMENT ON COLUMN) and §34.11 (checklist items 1 and 6), per `docs/OBSERVABILITY.md §12.6` v0.4 canonical registry obligation (2026-06-19): "authors should update those references at next authoring pass." (2) §29.10 item 9 added: R-34 cross-reference as operational recovery runbook for `turh_retention_purge` (job 32) stale — P1/M9 [x] Done (OBSERVABILITY §12.6 v4.9.3 fulfils). (3) §34.11 item 10 added: R-33 cross-reference as operational recovery runbook for `bdg_override_expiry_sweep` (job 34) stale — P1/M13 [x] Done (OBSERVABILITY §12.6 v0.8 fulfils). §34.11 P2 items renumbered 10–11 → 11–12.
- `VERSION` — 8.46.0 → 8.46.1.

---

## [8.46.0] — 2026-06-25

### Added
- `content/post-2426-training-log-minimal-version.md` — Серія 2426–2435 «Інструментарій self-coached атлета» пост 1/10. Тренувальний журнал як інструмент прийняття рішень, не архів: чому більшість журналів кидають; три функції журналу (точка відліку, тренди, якість рішень); мінімальний набір 4 полів (вправа, вага, підходи/повторення, RPE); що не логувати (розминочні підходи, настрій, щоденна вага); Kreider et al. (2017) — функція зовнішнього спостереження; Halson (2014) — RPE-дрейф 2-3 тижні до виснаження; ритуал огляду (1 хв / 5 хв / 30 хв); відновлення після пропуску. clinical-safety: NOT_REQUIRED. sports-scientist review pending.
- `README.md` — Editorial series 2426–2435 (Інструментарій self-coached атлета) додано до роадмапу; post-2426 → draft.
- `blog.html` — post-2426 card додано на початок фіду.

### Changed
- `STATUS.md` — content engine row оновлено: post-2426 draft · v8.46.0.
- `VERSION` — 8.45.0 → 8.46.0.

---

## [8.45.0] — 2026-06-25

### Added
- `content/post-2417-program-abandonment-self-coached.md` — Серія 2416–2425 «Тренувальна самостійність» пост 2/10. Program hopping як системна пастка self-coached атлета: хронологія адаптації (нейром'язовий мінімум 4–6 тижнів — Häkkinen et al. 1985; структурний 8–12 тижнів — Hubal et al. 2005); три тригери зміни та їх справжня природа; різниця між авторегуляцією в межах програми і program hopping; мінімальні часові горизонти (4/8/12–16 тижнів); три елементи системи захисту (визначений горизонт блоку, контрольні питання, журнал циклів); умови, за яких зміна виправдана. clinical-safety: NOT_REQUIRED. sports-scientist review pending. Blog card added.
- `docs/SOC2_READINESS.md §79.4` — DEL-MON-E-001 evidence artefact registered. Quarterly R-41 activation aggregate report: `activation_count`, `stale_window_hours`, `danger_window_tenants_affected`, `art17_sla_breached` per quarter; zero-activation quarters filed as affirmative attestation; `form_api` REVOKED from evidence path; C1.2/CC4.1; 7yr retention; path `compliance/evidence/deletions/deletion-sla-stale/del-mon-e-001-YYYY-QN.csv`. Closes R-41 implementation checklist item 4 (P1/M11).

### Changed
- `docs/INCIDENT_RESPONSE.md §R-41.11` — item 4 marked [x] Done (2026-06-25). Footer updated: DEL-MON-E-001 §79.4 cross-reference updated from "pending" to registered.
- `blog.html` — post-2417 card added at top of feed.
- `VERSION` — 8.44.1 → 8.45.0.

---

## [8.44.1] — 2026-06-25

### Added
- `docs/INCIDENT_RESPONSE.md` R-41 — GDPR Deletion SLA Monitor Stale (`deletion_sla_monitor` · job 43). Forty-first runbook; closes the gap where job 43 was the only §12.6 compliance pg_cron job without an INCIDENT_RESPONSE stale runbook. Covers P1 base / P0 escalation on active danger-window tenants, four root-cause hypotheses (H1 unauthorized deletion, H2 shared pg_cron failure, H3 `enterprise_churn_events` query failure, H4 Supabase outage), six-step recovery including mandatory backfill-emission and stale-window exception note for `danger_window_tenants_affected ≥ 1`, and `art17_sla_breached` boolean → R-09 activation path. SOC 2: C1.2 / CC4.1 / CC7.2.

### Changed
- `docs/AUDIT_LOG_SCHEMA.md` v2.41 — +2 R-41 monitoring-control events: `system.deletion_sla_monitor_stale_declared` (HIGH/7yr) and `system.deletion_sla_monitor_restored` (STANDARD/3yr). DELETION-SLA-MONITOR-STALE-CHAIN-01 ordering invariant, Zod schemas, emitter assignments, and alert routing added to §Enterprise Post-Churn & Deletion SLA events.
- `docs/OBSERVABILITY.md` §12.6 v1.5 patch — job 43 `deletion_sla_monitor` stale-consequence cross-ref extended with `INCIDENT_RESPONSE R-41 (job 43 stale recovery runbook — §R-41.5)`.
- `docs/INCIDENT_RESPONSE.md` — version bump v3.4 → v3.5.
- `VERSION` — 8.44.0 → 8.44.1.

---

## [8.44.0] — 2026-06-25

### Added
- `content/post-2416-information-overload-self-coached.md` — Editorial series 2416–2425 «Тренувальна самостійність: де self-coached атлет системно зупиняється» post 1/10. «Більше інформації не зробить тебе кращим атлетом». Аналіз-параліч у self-coaching: чому накопичення знань без системи рішень створює більше невизначеності, а не менше. Schwartz (2004) «парадокс вибору». Умовна природа тренувальних рекомендацій. Три рівні рішень (перед блоком/перед сесією/під час сесії). Три системні помилки без протоколу: зміна підходу в середині блоку; надмірна варіативність вправ (Häkkinen et al. 1985 — 4–6 тижнів); реакція на одиничну точку даних (Hubal et al. 2005 — 8–12 тижнів мінімум). Мінімальна архітектура рішень: журнал блоку / протокол перед сесією (3 питання) / тригери зупинки. Межа між «часом навчання» і «часом виконання». clinical-safety: NOT_REQUIRED. sports-scientist review pending. Blog card added.

### Changed
- `README.md` — Editorial series 2416–2425 «Тренувальна самостійність» added to roadmap (10 proposed topics); post-2416 → draft (v8.44.0).
- `STATUS.md` — post-2416 added; Total posts 2073 → 2074; current version v8.44.0.
- `VERSION` — 8.43.0 → 8.44.0.

---

## [8.43.0] — 2026-06-25

### Added
- `content/post-2403-progress-without-visible-changes.md` — Editorial series 2396–2405 post 8/10. «Прогрес без видимих змін: коли адаптація невидима — і як переконатись, що вона є». Ієрархія адаптацій з різними часовими константами: нейральна (Sale 1988 — рекрутування, синхронізація, 20–40% приросту без гіпертрофії, 2–8 тижнів), структурно-сполучна (сухожилля, Magnusson et al. 2010, 6–18 міс.), метаболічна. Вага на штанзі як запізнілий індикатор; Häkkinen et al. (1985): фази стагнації сили при реальному ЕМГ-прогресі. П'ять ранніх маркерів доступних без лабораторії: RPE-тренд при стабільному навантаженні, внутрішньосесійна втомна крива, технічна стабільність під навантаженням, толерантність суглобів і м'яких тканин, відновлення між сесіями. «Дзеркальний суд» як систематична помилка оцінки. Часові горизонти оцінки адаптацій. Ознаки реальної стагнації (не «невидимого прогресу»). Протокол підтвердження прогресу наприкінці блоку (чотири показники). clinical-safety: NOT_REQUIRED. sports-scientist review pending. Blog card added.
- `content/post-2404-decision-quality-under-pressure.md` — Editorial series 2396–2405 post 9/10. «Як зберегти якість тренувальних рішень під тиском». Decision fatigue і ego depletion (Baumeister 1998); уточнення Hagger et al. 2016 — ефект стомлення рішень підтверджується у прикладних контекстах. Три контексти тиску: часовий, мотиваційний, внутрішньосесійна невизначеність. If-then протоколи (Gollwitzer & Sheeran 2006): конкретні приклади для кожного контексту тиску. Скорочення кількості рішень у сесії через записану сесію. Kahneman 2011 Система 1 vs. Система 2 — перепрограмувати автоматичну реакцію, а не боротися з нею. Вплив сну на рішення: Harrison & Horne (2000) когнітивна деградація при 5 год; Walker (2017) дофамінові рецептори і мотивація. Ескалаційні тригери для відступу від if-then. Практична система 4 рівні (перед блоком/перед тижнем/перед сесією/під час). clinical-safety: NOT_REQUIRED. sports-scientist review pending. Blog card added.
- `blog.html` — blog cards added for posts 2400, 2401, 2402, 2403, 2404 (posts 2400–2402 cards were missing from previous iterations; all five added at top of feed).

### Changed
- `README.md` — posts 2403 → draft (v8.43.0); 2404 → draft (v8.43.0).
- `STATUS.md` — posts 2403/2404 added; Total posts 2071 → 2073; current version v8.43.0.
- `VERSION` — 8.42.0 → 8.43.0.

---

## [8.41.1] — 2026-06-23

### Changed
- `docs/COST_MODEL.md` — §44 «Loyalty Re-Entry Discount: OQ-WIN-02 Resolution (DEC-081)» added. Closes OQ-WIN-02 (P1, M9 deadline). Loyalty re-entry discount adopted with five eligibility controls: `wau_band_at_churn IN ('green','amber')` only; 2–12 month post-churn window; 10% cap on Year 1 ACV at then-current list price; WINBACK-CHAIN-01 `enterprise.winback_initiated` prerequisite; tiered approval (CS lead < $50k ACV, founder ≥ $50k ACV). Non-stackable with multi-year discount. §31.5 price floors absolute. New DEC-030 chain invariant REENTRY-CHAIN-01 — HTTP 422 `REENTRY_CHAIN_01_VIOLATION` if `enterprise.contract_renewed` with `contract_discount_type = 'loyalty_reentry'` lacks a prior `enterprise.pricing_exception_approved` within 30 days. New DDL column `contract_discount_type` on `enterprise_contracts`. Communication policy (binding): MUST NEVER appear in enterprise.html, pricing-enterprise.html, or any public materials. §43.11 OQ-WIN-02 row updated: ~~P1 open~~ → 🟢 Resolved → DEC-081 (2026-06-23). COST_MODEL v2.12 → v2.13.
- `docs/DECISION_LOG.md` — DEC-081 added: loyalty re-entry discount decision documented with full rationale and reverse cost.
- `VERSION` — 8.41.0 → 8.41.1.

---

## [8.41.0] — 2026-06-23

### Added
- `content/post-2401-training-time-deficit.md` — Серія 2396–2405 «Тренувальне мислення» пост 6/10. «Тренувальний дефіцит часу: реальний і удаваний». Три ситуації, що маскуються під «немає часу»: (1) реальний дефіцит вікон (новий батько/мати, гострий дедлайн, форс-мажор); (2) структурна неефективність програми (розминка-ритуал → 20 хв замість 8, відпочинок без таймера → +30–80 хв за сесію, допоміжні вправи перед основними — Raastad et al. 2000 передвтома знижує силовий вихід); (3) невідповідність формату ритму (ранок для несовиного, зал далеко від маршруту, програма 5 днів при стабільно вільних 3). Мінімально ефективний підтримуючий стимул: 120–135 хвилин/тиждень — Krieger (2010) мета-аналіз: навіть один підхід дає значущий приріст; Schoenfeld, Ogborn & Krieger (2017) 5–9 підходів/тиждень → значуща гіпертрофія; Ralston et al. (2017) 2 дні = 4+ дні при вирівняному тижневому об'ємі. Структура для реального часового обмеження: compound-first, 2–3 основних рухи, full-body або upper/lower при 2–3 сесіях, RPE-авторегуляція. Консистентний мінімум — кращий предиктор довгострокового результату, ніж непослідовний максимум (Mazzetti et al. 2000). Три діагностичних питання для розрізнення ситуацій. editorial-brutalist. clinical-safety: NOT_REQUIRED. sports-scientist review pending.

### Changed
- `blog.html` — Blog cards added for posts 2400 (backfill) and 2401 (two нові картки на початку grid).
- `README.md` — post-2400 → draft · v8.40.0; post-2401 → draft · v8.41.0; Editorial series 2406–2415 «Тренувальна адаптація в реальних умовах» added (10 proposed topics).
- `STATUS.md` — Total posts 2068 → 2070; серія 2396–2405 оновлена до 6/10; current version → v8.41.0.
- `VERSION` — 8.40.0 → 8.41.0.

---

## [8.40.0] — 2026-06-23

### Added
- `content/post-2400-first-six-weeks-new-program.md` — Серія 2396–2405 «Тренувальне мислення» пост 5/10. «Перші 6 тижнів нової програми: що відбувається і чого не варто чекати». Нейральна адаптація домінує в тижнях 1–8: рекрутмент моторних одиниць, синхронізація, rate coding, між-м'язова координація (Moritani & deVries 1979; Sale 1988). Структурна гіпертрофія — мінімум 8–12 тижнів (Schoenfeld 2010; Damas et al. 2016). Три відчуття, що спотворюють оцінку в ранній фазі: DOMS як сигнал прогресу (Armstrong 1984 — сигнал новизни, Nosaka & Clarkson 1997 repeated bout effect); ранній нейральний стрибок і «тиша» після (Stone et al. 2007); «рухи ще не мої» (Fitts & Posner 1967 три фази моторного навчання). Три помилки: агресивний об'ємний рамп у тижні 1–2 (Kraemer & Ratamess 2004); оцінка програми в тижні 4–5 (мертва зона нейральної адаптації); порівняння з чужим стартом (Hubal et al. 2005: розкид гіпертрофічної відповіді 0–59%). Що трекати: консистентність виконання, технічна стабільність, тривалість DOMS, загальний енергетичний тон. Benchmark session у тижні 6 — реальна базова лінія (Bompa & Buzzichelli 2015). editorial-brutalist. clinical-safety: NOT_REQUIRED. sports-scientist review pending.

### Changed
- `VERSION` — 8.39.1 → 8.40.0.

---

## [8.39.1] — 2026-06-23

### Changed
- `docs/DATA_MODEL.md §44.11` — Cross-reference sync: AUDIT_LOG_SCHEMA obligation row updated 🟡 → 🟢. `enterprise.winback_initiated` / `enterprise.winback_converted` / `enterprise.deletion_certificate_issued` / `enterprise.deletion_sla_warning` / `enterprise.deletion_sla_critical` were registered in `docs/AUDIT_LOG_SCHEMA.md` v2.39/v2.40 (2026-06-23) simultaneously with the §44 authoring and DEC-079 amendment; the §44.11 cross-reference table row was not updated in the same commit. §44.11 is now fully 🟢. DATA_MODEL v1.27 → v1.28.
- `VERSION` — 8.39.0 → 8.39.1.

---

## [8.39.0] — 2026-06-23

### Added
- `content/post-2399-when-self-coaching-breaks.md` — Серія 2396–2405 «Тренувальне мислення» пост 4/10. «Коли self-coaching достатньо — і де він системно ламається». Умови де self-coaching раціональний вибір: мета — загальна фізична підготовка (Krieger 2010, Schoenfeld et al. 2017 маргінальний ефект «оптимізації» відносно ключових змінних), стабільні рухові патерни, структура рішень, здатність до об'єктивної самооцінки. Чотири структурні точки відмови: (1) технічна корекція під навантаженням — Baraki & Kilgore (2022) технічні дефекти фіксуються в руховій пам'яті без корекції; (2) рішення під стресом — відсутність зовнішнього якоря; (3) сліпа пляма прогресу — Hubal (2005) вага росте, техніка деградує невидимо; (4) складні рухи на старті — перевчати важче ніж вчити. AI-тренер vs PT — чесний trade-off: PT незамінний у реалтаймовій корекції і нелінгвістичних сигналах, PT незамінний для змагань; AI достатній у адаптації на основі даних (HRV-тренд, накопичений обсяг, ACWR), консистентності і доступності. П'ять сигналів потреби у зовнішній петлі: технічний (відчуваєш «щось не так» але не можеш назвати), прогресний (стагнація 3+ місяці без пояснення), рішення на відчутті а не структурі. FORM позиціонування: конкретна ніша self-coached атлета де суб'єктивна оцінка системно збоїть. editorial-brutalist. clinical-safety: NOT_REQUIRED. sports-scientist review pending.

### Changed
- `blog.html` — Blog cards added for posts 2397, 2398, 2399 (три нові картки на початку grid).
- `README.md` — Серія 2396–2405: posts 2397, 2398 → draft; post-2399 → draft.
- `STATUS.md` — Total posts 2067 → 2068; серія 2396–2405 оновлена до 4/10; next priorities оновлені.
- `VERSION` — 8.38.0 → 8.39.0.

---

## [8.38.0] — 2026-06-23

### Added
- `content/post-2389-mrv-signs-of-exceeding.md` — Серія 2386–2395 «Навантажувальний менеджмент» пост 4/10. «MRV і ознаки перевищення: як розпізнати «система сказала досить»». Різниця між функціональним (FOR: відновлення 1–2 тижні, суперкомпенсація після деловантаження) і нефункціональним перевантаженням (NFOR: 2–6 тижнів, без суперкомпенсації) — Meeusen et al. (2013) консенсус ECSS/ACSM. Три категорії сигналів MRV-перевищення: (1) продуктивність — зниження або стагнація при стабільних умовах, прогресивне зростання RPE при незмінних вагах, деградація підходів із «поширенням назад»; (2) відновлення — неповне відновлення між сесіями, зростання часу розминки, систематично погана якість сну (Kreher & Schwartz 2012: один із найраніших маркерів); (3) суб'єктивні — небажаність тренувань не мотиваційного типу, стійке відчуття важкості м'язів у спокої (Halson & Jeukendrup 2004). Поріг для висновку: 2+ сигнали з різних категорій, 2+ тижні при постійних умовах. П'ять факторів, що змінюють MRV: тренувальний стаж, фаза блоку, зовнішній стрес, сон, час після паузи. Три помилки: «це просто важкий тиждень», ігнорування суб'єктивних сигналів, «більше-тренуватись-петля» (Kreher & Schwartz 2012). Операційна карта MEV→MAV→MRV→вище MRV. editorial-brutalist. clinical-safety: NOT_REQUIRED. sports-scientist review pending.
- `content/post-2398-when-to-change-program.md` — Серія 2396–2405 «Тренувальне мислення» пост 3/10. «Зміна програми: коли це рішення, а коли — паніка». Три причини передчасних змін: нетерпіння до «видимих» результатів (перші 4–6 тижнів — переважно нейральні адаптації), соціальний шум, дискомфорт від стагнації. Мінімальне вікно оцінки: 8–12 тижнів при стабільних умовах (Bompa & Buzzichelli 2015 — структурна гіпертрофія; Rhea et al. 2002 — розходження між стратегіями після 12 тижнів). Три легітимних критерії зміни: (1) підтверджена стагнація — конкретний вимірюваний показник не зростає 8+ тижнів при стабільних умовах; (2) системна невідповідність цілям — структура не відповідає новій меті чи контексту; (3) структурна несумісність — технічна невідповідність між програмою і реальними умовами. Три паттерни паніки: «результатів немає» за 4–6 тижнів, «бачив кращу програму», «нудно». Три перевіряльних питання. Ієрархія змін: локальна корекція → реструктуризація блоку → повна зміна підходу. Stone et al. (2007) «синдром пошуку оптимальної програми». editorial-brutalist. clinical-safety: NOT_REQUIRED. sports-scientist review pending.

### Changed
- `STATUS.md` — Total posts 2065 → 2067; серія 2386–2395 оновлена до 4/10; серія 2396–2405 оновлена до 3/10; next priorities оновлені.
- `VERSION` — 8.37.0 → 8.38.0.

---

## [8.37.0] — 2026-06-23

### Added
- `content/post-2397-bad-session-as-data.md` — Серія 2396–2405 «Тренувальне мислення» пост 2/10. «Провальна сесія як дані: як не перетворити одне погане тренування на системний висновок». Чотири типи «поганих» сесій: контекстуальна (умови, не тренованість), втомна (накопичена стомленість у кінці блоку), технічна (деадаптація після паузи або нова вправа), мотиваційна (вольові ресурси, не фізіологія). Чому одна сесія — шум: Issurin (2010) нормальна варіативність 8–12% навіть у контрольованих умовах; Hecksteden et al. (2017) до 15% між ідентичними сесіями. Три найчастіші помилки після провальної сесії: термінова зміна програми, аварійне деловантаження, самодіагностика відсутності прогресу; Stone et al. (2007) «реакція на шум» як типова помилка self-coached. Алгоритм чотири кроки: завершити за планом з мінімальними корекціями → зафіксувати контекст → класифікувати тип → рішення тільки про наступну сесію. Системний сигнал: три поспіль сесії в схожих умовах зі схожим відхиленням. editorial-brutalist. clinical-safety: NOT_REQUIRED. sports-scientist review pending.
- `content/post-2388-mav-productive-zone.md` — Серія 2386–2395 «Навантажувальний менеджмент» пост 3/10. «MAV і продуктивна зона: між достатнім і забагато». Термінологічна карта MEV/MAV/MRV: чому тренування ближче до MRV дає більше втоми ніж адаптації (Israetel et al. 2019). Механізм продуктивної зони: Schoenfeld et al. (2017) перевернута U-крива залежності гіпертрофії від обсягу. Два методи пошуку MAV: прогресивне нарощування з двома індикаторами (якість відновлення між сесіями + якість підходів у другій половині сесії) і ретроспективний аналіз через журнал. MAV у різних фазах: акумуляція (зона ширша), реалізація (зона вужча, вища інтенсивність), деловантаження (нижче MEV навмисно). Damas et al. (2016) ранні нейральні адаптації — після паузи MAV тимчасово нижчий. Таблиця орієнтовних MAV по м'язових групах. Colquhoun et al. (2018): помірний обсяг = зрівнянний приріст сили, краще відновлення. Різниця між «більше» і «продуктивно більше»: перерахунок сирих підходів в ефективні часто показує менший реальний обсяг. editorial-brutalist. clinical-safety: NOT_REQUIRED. sports-scientist review pending.

### Changed
- `STATUS.md` — Total posts 2063 → 2065; серія 2396–2405 оновлена до 2/10; серія 2386–2395 оновлена до 3/10; next priorities оновлені.
- `VERSION` — 8.36.1 → 8.37.0.

---

## [8.36.1] — 2026-06-23

### Added
- `docs/MSA_TEMPLATE.md §11.6` — Litigation Hold and Legal Claim Retention clause (GDPR Art. 17(3)(e)). Closes OQ-WIN-04 (`docs/COST_MODEL.md §43.11`; DEC-080). Activation: compliance-officer written instruction + outside counsel countersign. Scope: non-health data only (billing records, contract records, relevant DEC-030 audit log entries, SLA performance data). Art. 9 health data categorically excluded (§12.2 zero-grace-period per DEC-036 is absolute regardless of any hold). Duration: 36-month maximum; 6-month re-confirmation reviews; 5-business-day Customer notice. DPA Art. 7 processing basis: GDPR Art. 6(1)(f) (legitimate interests) or Art. 6(1)(c) (legal obligation). Two DEC-030 events: `enterprise.litigation_hold_declared` HIGH/7yr and `enterprise.litigation_hold_released` HIGH/7yr — registered in `docs/AUDIT_LOG_SCHEMA.md §Enterprise Post-Churn`. Outside counsel review required before first MSA execution (§11.6.9). MSA_TEMPLATE.md v0.6 → v0.7.
- `docs/AUDIT_LOG_SCHEMA.md` — Two new events: `enterprise.litigation_hold_declared` (HIGH/7yr) and `enterprise.litigation_hold_released` (HIGH/7yr). Zod v2 schemas: `LitigationHoldDeclaredPayload` and `LitigationHoldReleasedPayload`. Emitter: compliance-officer (manual, PAM-elevated); no automated path. Enums: `LitigationHoldReasonEnum` (billing_dispute/legal_claim/regulatory_direction/anticipated_litigation), `HeldDataCategoryEnum` (billing_records/contract_records/audit_log_entries/sla_performance_data), `release_reason` (five values). Privacy floor: no user_id, no health values, no Art. 9 data in any hold event payload.
- `docs/DECISION_LOG.md §DEC-080` — OQ-WIN-04 resolved: litigation hold procedure documented in MSA §11.6; DPA Art. 7 amendment mechanism; health data absolute exclusion; DEC-030 events registered.

### Changed
- `docs/MSA_TEMPLATE.md §12.2` — Zero-grace-period text updated: replaced ambiguous parenthetical "(except for active litigation hold on non-health data)" with explicit cross-reference to §11.6.2 confirming Art. 9 health data is categorically excluded from all holds.
- `docs/MSA_TEMPLATE.md` (Internal Notes) — Added litigation hold guidance row: scope, activation requirements, health data exclusion, DEC-030 events, outside counsel gate, OQ-WIN-04 reference.
- `docs/COST_MODEL.md §43.11` — OQ-WIN-04 marked 🟢 Resolved → DEC-080 (2026-06-23).
- `VERSION` — 8.36.0 → 8.36.1.

---

## [8.36.0] — 2026-06-23

### Added
- `content/post-2396-subjective-signals-training.md` — Серія 2396–2405 «Тренувальне мислення: рішення, помилки, автономія» 1/10. Суб'єктивні сигнали тренування: RPE вимірює інтерпретацію ЦНС, а не механічне навантаження (Borg 1982). П'ять систематичних ситуацій помилки: дефіцит сну (Hobson & Levy 2019 мета-аналіз: +1.3–1.7 пункти RPE при тому ж навантаженні), ефект новизни (Henan & Barrientos 2017: −0.8–1.2 пункти за 3–4 тижні без зміни навантаження), накопичена тренувальна втома (зона неточного RPE починається раніше, Helms et al. 2016), перший тиждень після перерви (нейром'язова рекалібровка + психологічний компонент), зміна середовища (спека +8°C → +1.5 пункти RPE, Gonzalez-Alonso & Calbet 2003). Феномен «центрального губернатора» Noakes 2012: RPE 10 — рідко справжня абсолютна відмова. Три протоколи калібровки: anchor-тест, контекстний лог, AMRAP-верифікація раз на блок. editorial-brutalist. clinical-safety: NOT_REQUIRED. Blog card added. sports-scientist review pending.

### Changed
- `README.md` — Editorial series 2396–2405 «Тренувальне мислення» додано як нова серія; post-2396 → draft.
- `STATUS.md` — total posts 2062 → 2063; серія 2396–2405 відкрита.
- `blog.html` — blog card added for post-2396.
- `VERSION` — 8.35.0 → 8.36.0.

---

## [8.35.0] — 2026-06-23

### Added
- `content/post-2387-mev-in-practice.md` — Серія 2386–2395 «Навантажувальний менеджмент» 2/10. MEV на практиці: чому популяційні норми є стартовою гіпотезою, а не ціллю; чотирикроковий протокол визначення власного MEV без лабораторії (базова лінія ефективних підходів → сигнальний тест адаптації → редукційний експеримент → валідація в журнал); таблиця стартових MEV по м'язових групах (Israetel 2019, Krieger 2010, Schoenfeld 2017, Hubal 2005); MEV як структура для важких тижнів і реалізаційних блоків; чотири типові помилки. clinical-safety: NOT_REQUIRED. sports-scientist review pending. Blog card pending.

### Changed
- `README.md` — post-2386, post-2387 статус: proposed → draft.

---

## [8.34.1] — 2026-06-23

### Changed
- `docs/DATA_MODEL.md §44.6` — WINBACK-CHAIN-01 definition patched per DEC-079: Worker lookup is now by `winback_initiated_event_id` (event UUID), not current `tenant_id`; `prior_tenant_id` on `WinbackConvertedPayload` carries the churned tenant UUID. Aligns DATA_MODEL with COST_MODEL §43.7 which was updated at DEC-079 adoption. Also fixes missed document header bump (v1.25 → v1.27). No DDL or RLS changes. Document header v1.25 → v1.27.

---

## [8.34.0] — 2026-06-23

### Added
- `content/post-2386-effective-sets-mechanism-not-count.md` — Серія 2386–2395 «Навантажувальний менеджмент: операційна рамка управління обсягом» 1/10. Що таке «ефективний підхід» і чому не всі підходи рівні: механізм, а не рахунок. Stimulating reps як механізм (Size Principle, рекрутмент волокон типу II, Henneman 1965). Механічна напруга як первинний драйвер гіпертрофії (Schoenfeld 2010, Wackerhage et al. 2019). Чому перші повторення при субмаксимальних навантаженнях є субпороговими і коли підключаються швидкі волокна. Таблиця RPE/RIR vs ефективні повторення. Чому відмова — не обов'язкова умова: RPE 8–9 як оптимальна зона (Schoenfeld & Grgic 2019). Внутрішньосесійна деградація ефективності (Krieger 2010). Junk volume: визначення і ознаки у логу. Що FORM і Victor відстежують замість сирого рахунку підходів. Lasevicius et al. (2019), Baz-Valle et al. (2022). clinical-safety: NOT_REQUIRED. Blog card added.

### Changed
- `blog.html` — blog card added for post-2386 (ефективний підхід · навантажувальний менеджмент 1/10 · 13 хв · draft).
- `STATUS.md` — content table updated: post-2386 added; Серія 2386–2395 відкрита; total posts 2060 → 2061; next priorities updated.

---

## [8.33.0] — 2026-06-23

### Added
- `content/post-2384-long-term-loop-self-coached.md` — Серія 2376–2385 «Зворотний зв'язок без тренера» 9/10. Довгострокова петля self-coached атлета: ретроспектива блоку і системні висновки. Три типи ретроспективної інформації: результативна (e1RM-динаміка, RPE-тренд, compliance), процесна (якість виконання, RPE-калібрація, технічні девіації), гіпотезна (попередньо визначена зміна + критерій підтвердження). Між-блокова порівняльна таблиця. Індивідуальні паттерни: відповідь на об'єм (особистий MAV/MEV/MRV), часова характеристика адаптації, контекстна вразливість, сезонна варіативність. Три помилки ретроспективного аналізу: survivorship bias, overfitting на одному прикладі, ігнорування контексту. Мінімальний шаблон ретроспективи (30 хвилин). Stone et al. (2007) «тренувальна пам'ять»; Issurin (2010) блокова модель та часові характеристики адаптації; Hopkins et al. (1999) мінімально значуща зміна; Zourdos et al. (2016) RPE-надійність; Helms et al. (2016) self-coached як серія мікроекспериментів; Israetel et al. (2019) MEV/MAV/MRV; Hooper et al. (1995) нелінійна навантажувальна толерантність. clinical-safety: NOT_REQUIRED. Blog card added.
- `content/post-2385-information-loop-complete-system.md` — Серія 2376–2385 «Зворотний зв'язок без тренера» 10/10 · СЕРІЯ ЗАВЕРШЕНА. Повна система self-coached атлета: синтез усіх дев'яти постів серії в єдину зв'язну практику. Чотири рівні петлі: рівень 1 мікро (сесія→сесія, 5–7 хв, мікрокорекції), рівень 2 мезо-1 (тижневий огляд, 15–20 хв, флагування аномалій), рівень 3 мезо-2 (блоковий аналіз, 30–45 хв, одна змінна + гіпотеза), рівень 4 макро (6–12 місяців, 1–2 год, структурні зміни та індивідуальні паттерни). Мінімальна версія: трьохетапне нарощення (місяць 1–2 / 3–4 / 5+). Інтеграція класифікаційної рамки (пост 2383) у п'ятикроковий протокол перед кожним рішенням. Три критичні межі системи: технічний зворотний зв'язок, клінічна оцінка, зміна тренувального статусу. Halperin et al. (2015); Stone et al. (2007); Zourdos et al. (2016); Wulf et al. (2010); Meeusen et al. (2013). clinical-safety: NOT_REQUIRED. Blog card added.

### Changed
- `blog.html` — blog cards added for post-2384 (довгострокова петля, 14 хв) і post-2385 (повна система, 15 хв). Обидва — draft, series: 2376–2385 positions 9/10 і 10/10.
- `STATUS.md` — content table updated: post-2384 і post-2385 added; Серія 2376–2385 позначена COMPLETE 10/10; total posts 2058 → 2060; next priorities updated.

---

## [8.32.0] — 2026-06-23

### Added
- `docs/DECISION_LOG.md §DEC-079` — OQ-WIN-03 resolved: new `tenant_id` UUID (Option B) always provisioned at winback; WINBACK-CHAIN-01 invariant amended to event-UUID lookup. Decision: `enterprise.winback_converted.tenant_id` = new provisioned UUID; `prior_tenant_id` (new field) = churned tenant UUID. Rationale: Option A (reuse prior `tenant_id`) violates DEL-E-001 deletion certificate legal representation and SOC 2 C1.2 HMAC chain integrity. Event-UUID predecessor lookup via `winback_initiated_event_id` is stricter and more tamper-resistant than `tenant_id`-based lookup. Pattern consistent with `deletion_request_event_id` FK in `DeletionCertificateIssuedPayload`. No NRR denominator risk (new UUID = clean ARR lifecycle per §43.6.3). Reverse cost: medium (DPA re-execution with EU enterprise customers if reverted). Owner: enterprise-architect + compliance-officer.

### Changed
- `docs/AUDIT_LOG_SCHEMA.md` — v2.39 → v2.40. `WinbackConvertedPayload` gains `prior_tenant_id: z.string().uuid()` (churned tenant UUID; enables HMAC chain self-audit without joining `enterprise_churn_events`); `tenant_id` clarified as new provisioned UUID per DEC-079. WINBACK-CHAIN-01 invariant description amended in event table row and introductory block: lookup by `winback_initiated_event_id` event UUID + cross-check `tenant_id` of predecessor = `prior_tenant_id` of converted event.
- `docs/COST_MODEL.md` — §43.7.3 WINBACK-CHAIN-01 row updated to reflect DEC-079 event-UUID lookup; §43.11 OQ-WIN-03 row updated: `P0 (before first winback)` → 🟢 Resolved → DEC-079 (2026-06-23).
- `VERSION` — 8.31.0 → 8.32.0.

---

## [8.31.0] — 2026-06-23

### Added
- `content/post-2383-four-training-error-types.md` — Серія 2376–2385 «Зворотний зв'язок без тренера», пост 8/10. Класифікація тренувальних помилок до корекції: чому неправильна класифікація — дорожча помилка за саму помилку. Чотири типи: (1) Програмування — змінні неоптимальні, симптом відтворюваний між блоками, compliance ≥ 90%; (2) Виконання — compliance < 85%, RPE-калібрація дрейфує, технічна якість деградує швидше очікуваного (Zourdos et al. 2016; Cook 2010: суб'єктивна vs. об'єктивна оцінка −30–40%); (3) Інтерпретація — дані хороші, рішення без попередньо визначених критеріїв; три механізми (confirmation bias, рецентність, помилкова атрибуція — Halperin et al. 2015); (4) Контекстна — зовнішній стресор не задокументований, симптом не відтворюється в наступному блоці (Kraemer & Ratamess 2004 алостатичне навантаження). Типові каскади помилок: контекстна→інтерпретація→програмування («спіраль консерватизму» Helms et al. 2016); виконання→інтерпретація→програмування. Практичне правило: перевіряти типи від базових до складних (контекст → виконання → інтерпретація → програма). П'ятикроковий діагностичний протокол (15 хв). Meeusen et al. (2013): overreaching vs. мотиваційна нехіть — операційний критерій: результат після 48–72 год відпочинку. Wulf et al. (2010): межі self-діагностики технічних помилок. Мінімальна таблиця класифікації (6 рядків). clinical-safety: NOT_REQUIRED. sports-scientist review pending. Blog card added.

### Changed
- `blog.html` — blog card added for post-2383; total visible cards: 2383+.
- `STATUS.md` — content table updated: post-2383 added, total 2058; серія 2376–2385 8/10; next: post-2384.
- `VERSION` — 8.30.0 → 8.31.0.

---

## [8.30.0] — 2026-06-23

### Added
- `content/post-2382-correcting-program-from-data.md` — Серія 2376–2385 «Зворотний зв'язок без тренера», пост 7/10. Принцип мінімальної ефективної зміни: чому змінювати одну змінну між блоками — не обережність, а умова читабельності власної відповіді на тренування (Schoenfeld 2016). Три рівні корекції: мікро (між сесіями, авторегуляція), мезо (між тижнями, тижневий тренд), блокова (між блоками, системна зміна). Ієрархія змінних за Israetel et al. (2019): об'єм → інтенсивність → вибір вправ → частота. Корекція об'єму: сигнали «занизький» (e1RM нейтральний, RPE стабільний, compliance ≥ 90%) і «завищений» (RPE ріс, e1RM стагнував); пороги MAV/MRV для intermediate. Корекція інтенсивності: сигнали «занизька» (RPE 5–6, повільне e1RM) і «завишена» (технічна деградація, RPE 8–9 на початку). Корекція вправи: тільки при хронічній деградації при коректному навантаженні. Когда НЕ коригувати: низький compliance, відстрочений ефект (Issurin 2010), нейтральні сигнали. Confirmation bias (Halperin et al. 2015): два практичних захисти — попередньо визначені критерії рішення і окремий «аналітичний день» після деловантажу. Межа між корекцією і нестабільністю: мінімум 2 блоки на одній конфігурації. Виняток: сигнал non-functional overreaching (Meeusen et al. 2013) — негайне зниження об'єму. Мінімальна таблиця корекцій (6 рядків). clinical-safety: NOT_REQUIRED. sports-scientist review pending. Blog card added.

### Added
- `content/post-2381-block-analysis-without-coach.md` — Серія 2376–2385 «Зворотний зв'язок без тренера», пост 6/10. Блоковий аналіз (4–8 тижнів) як інструмент окремий від тижневого тренду — відповідає на питання «чи спрацювала ця конфігурація навантаження?». Issurin (2010): мезоцикл — мінімальна одиниця для концентрованих адаптаційних ефектів; Stone et al. (2007): мінімальний горизонт 4–6 тижнів для системної тенденції. Чотири виміри: e1RM-динаміка (Hopkins et al. 1999: практичний поріг ≥ 2,5%; умови порівняння), RPE-тренд (Zourdos et al. 2016 ICC 0.89–0.94; Meeusen et al. 2013 — стійкий ріст RPE при незмінному навантаженні = маркер functional overreaching), compliance (Helms et al. 2016; порогові значення ≥90%/75–89%/<75%), технічна якість (Wulf et al. 2010; Cook 2010 — деградація раніше RPE і e1RM змін). Методологія міжблокового порівняння: контроль позиції у блоці, наявності деловантажу, об'єму навантаження (відстрочений ефект Issurin 2010), зовнішніх конфундерів (Kraemer & Ratamess 2004 алостатичне навантаження). Протокол 45–60 хв: крок 1 збір даних (15 хв), крок 2 матриця оцінок +/0/- (10 хв), крок 3 дерево рішень (5 результатів), крок 4 планування наступного блоку (15 хв). П'ять типових помилок: порівняння піку з провалом; ігнорування compliance як коефіцієнта; висновки на основі одного блоку (Halperin et al. 2015); зміна більше однієї змінної; плутання відстроченого ефекту з відсутністю ефекту. Межа: блоковий аналіз дає коригувальні рішення всередині системи, не заміну системи. Мінімальний шаблон блокового аудиту. clinical-safety: NOT_REQUIRED. sports-scientist review pending. Blog card added.

### Changed
- `blog.html` — blog cards added for post-2381 і post-2382; total visible cards: 2382+.
- `STATUS.md` — content table updated: posts 2381 і 2382 added, total 2057; серія 2376–2385 7/10; next priorities updated.
- `VERSION` — 8.28.0 → 8.30.0.

---

## [8.28.0] — 2026-06-23

### Added
- `docs/AUDIT_LOG_SCHEMA.md §Enterprise Post-Churn & Deletion SLA` — New section `### Enterprise Post-Churn & Deletion SLA events` (DEC-030 HMAC-chained · COST_MODEL §43.7 + OBSERVABILITY §12.6 · C1.2/CC4.1/CC5.2/CC6.1). Five events registered: `enterprise.winback_initiated` (HIGH, 7yr — WINBACK-CHAIN-01 prerequisite: CSM outreach anchor; payload: tenant_id, prior_contract_id, churn_date, months_since_churn 1–60, churn_reason ChurnReasonEnum, wau_band_at_churn, prior_acv_usd, initiated_by CSM UUID, outreach_sequence enum); `enterprise.winback_converted` (CRITICAL, 7yr — WINBACK-CHAIN-01 terminal: blocked HTTP 422 WINBACK_CHAIN_01_VIOLATION without prior `winback_initiated` within 365d; payload: tenant_id, winback_contract_id new UUID, prior_contract_id, winback_initiated_event_id, months_since_churn, prior_acv_usd, new_acv_usd, acv_premium_pct 0–200, new_seats, new_contract_years 1|2|3, confirmed_by compliance-officer UUID); `enterprise.deletion_certificate_issued` (CRITICAL, 7yr — DELETION-CHAIN-01 terminal: blocked HTTP 422 DELETION_CHAIN_01_VIOLATION if days_since_trigger > 35; P0 escalation if gdpr_art17_request and > 30d; payload: tenant_id, contract_id, deletion_request_event_id optional, trigger_type enum, deletion_date, classes_deleted array, classes_retained array, retained_legal_basis literal 'gdpr_art17_3e_legal_obligation', certificate_r2_path regex, compliance_officer_id, days_since_trigger 0–35); `enterprise.deletion_sla_warning` (HIGH, 7yr — DELETION-SLA-01 day-25 pre-breach warning from pg_cron job 43; AL-DEL-01 P1 PagerDuty form-enterprise; payload: tenant_id, days_since_deletion_requested, alert_tier 'warning', check_run_at, alert_dedup_key); `enterprise.deletion_sla_critical` (CRITICAL, 7yr — DELETION-SLA-01 day-29 P0 escalation; payload: same as warning, alert_tier 'critical'). Full Zod v2 schemas (ChurnReasonEnum; WinbackInitiatedPayload; WinbackConvertedPayload; DeletionCertificateIssuedPayload; DeletionSlaWarningPayload; DeletionSlaCriticalPayload). Chain invariants: WINBACK-CHAIN-01 (365d predecessor window; HTTP 422 WINBACK_CHAIN_01_VIOLATION); DELETION-CHAIN-01 (35d max; P0 at 30d for gdpr_art17_request; HTTP 422 DELETION_CHAIN_01_VIOLATION); DELETION-SLA-01 monitoring (job 43, 6h cadence, 7h freshness). Emitter assignments, alert routing (AL-DEL-01 P1/P0 PagerDuty, DEL-CERT-01 advisory Slack), SOC 2 evidence artefacts (DEL-E-001 C1.2/CC4.1 annual 7yr; WBK-E-001 CC4.1/CC5.2 annual 3yr; CHN-E-001 CC6.1/CC7.1 quarterly 3yr), and auditor narratives for C1.2/CC4.1/CC5.2/CC6.1. Privacy floor (all five events): no employee user_id, name, email, health value, or Art. 9 data; tenant_id is FORM-internal UUID only. Closes `docs/COST_MODEL.md §43.10` item 1 (P0/M10 — 🟢) and item 3 remaining AUDIT_LOG_SCHEMA obligation (🟢). AUDIT_LOG_SCHEMA v2.38 → v2.39.

### Changed
- `docs/AUDIT_LOG_SCHEMA.md` — header v2.38 → v2.39; new §Enterprise Post-Churn & Deletion SLA events section; v2.39 version history entry added.
- `docs/COST_MODEL.md` — §43.10 item 1 status `[ ]` → `[x]` Done (documentation — AUDIT_LOG_SCHEMA v2.39, 2026-06-23); §43.10 item 3 "Remaining" note updated: AUDIT_LOG_SCHEMA obligation struck through and marked 🟢 Done.
- `VERSION` — 8.27.0 → 8.28.0 (MINOR: new enterprise DEC-030 event registration section in AUDIT_LOG_SCHEMA).

## [8.27.0] — 2026-06-23

### Added
- `content/post-2380-weekly-trend-reading-protocol.md` — Серія 2376–2385 «Зворотний зв'язок без тренера» пост 5/10. Протокол читання тижневого тренду для self-coached атлета. Тижневий горизонт як перший горизонт видимої адаптаційної тенденції (Stone et al., 2007: мінімум 7–14 днів). Три змінні аудиту: RPE-тренд на еталонних вправах (Zourdos et al., 2016: ICC 0.89–0.94 при порівнянних умовах), e1RM-динаміка (Hopkins et al., 1999: варіація ±2–4%), compliance (виконано/заплановано). Протокол 15 хв у неділю: Крок 1 — збір фактів (три рядки даних); Крок 2 — порівняння без інтерпретації (таблиця краще/стабільно/гірше); Крок 3 — дерево рішень (продовжуй / спостерігай / діагностуй / корекція); Крок 4 — три питання при 2+ «гірше». П'ять типових помилок: порівняння тижнів різного блокового положення, суб'єктивне «відчуття тижня» замість даних, ігнорування compliance як коригувального коефіцієнта, рішення на підставі одного «поганого» тижня (Halperin et al., 2015), аудит без фіксації. Мінімальний шаблон запису. Відмінність тижневого аудиту від блокової оцінки. clinical-safety: NOT_REQUIRED. sports-scientist review pending.

### Changed
- `blog.html` — blog card added for post-2380 (перед post-2379).
- `STATUS.md` — Total posts 2054 → 2055; Серія 2376–2385 progress 4/10 → 5/10; post-2380 summary added; Next оновлено на post-2381.
- `VERSION` — 8.26.0 → 8.27.0 (MINOR: новий content post серії 2376–2385).

## [8.26.0] — 2026-06-23

### Added
- `docs/SOC2_READINESS.md §103` — Evidence Table Patch: ENGAGE-E-001/002/003 / SSO-OBS-E-005/006 / WS-E-001/002/003 / CONC-E-001 Registration (SOC2_READINESS §84 · C1.1 / C1.2 / CC2.2 / CC6.1 / CC7.2 / CC4.1 / CC7.4 / A1.1 / P3.2). Registers nine evidence artefacts defined in §84 (OBSERVABILITY cross-refs: §33.11 engagement, §26.7a/b SSO/SCIM, §41.9 wearable, INCIDENT_RESPONSE §18.6 concurrent-chain) into the §79.4 master evidence table. Nine new §79.4 rows: ENGAGE-E-001 (C1.1/C1.2 — annual `tenant_engagement_snapshots` DDL/RLS/negative-privilege export, 3yr, `engagement/`); ENGAGE-E-002 (CC2.2/CC4.1 — quarterly anonymised QBR package, 3yr, `engagement/`); ENGAGE-E-003 (CC7.2/A1.1 — PagerDuty AL-ENGAGE-01..06 alert history + quarterly `tenant.churn_risk_flagged` DEC-030 export, 3yr, `engagement/`); SSO-OBS-E-005 (A1.1/CC7.2 — quarterly AL-SCIM-MASS-01 PagerDuty log; zero-count = positive non-occurrence attestation, 3yr, `sso/`); SSO-OBS-E-006 (CC7.2/CC6.1 — quarterly AL-SCIM-01..04 log; AL-SCIM-04 zero-count = SCIM-CHAIN-01 integrity attestation, 3yr, `sso/`); WS-E-001 (A1.1 — quarterly `wearable.sync_completed` DEC-030 HMAC-chained export per source, 5yr, `wearable/`); WS-E-002 (P3.2 — quarterly fleet freshness report with K-anonymity gate N<5, 5yr, `wearable/`); WS-E-003 (CC7.2 — annual AL-WS-01..07 alert rule configuration screenshots, 3yr, `wearable/`); CONC-E-001 (CC7.4/CC4.1 — quarterly §18.3.1 skip-and-verify SQL + CONC-CHAIN-01 zero-row check; zero-incident quarters filed as affirmative attestation, 7yr, `ir-chain/`). Four new §80.3 R2 subfolders: `sso/`, `engagement/`, `wearable/`, `ir-chain/` (all `form-api NO ACCESS`). §80.4 Vanta mirror list extended with all nine artefacts. Closes §84.8 items 9 (R2 folders) and 10 (§79.4 registration), and §92.6 item 1 (`sso/` subfolder documentation).

### Changed
- `docs/SOC2_READINESS.md` — §79.4 nine new evidence rows; §80.3 four new R2 subfolders (`sso/`, `engagement/`, `wearable/`, `ir-chain/`); §80.4 Vanta list extended; §84.8 items 9/10 marked `[x] Done`; §92.6 item 1 marked `[x] Done`; §103 section appended. v3.27.0 → v3.28.0.
- `VERSION` — 8.25.0 → 8.26.0 (MINOR: new §103 evidence registration section in SOC2_READINESS).

## [8.25.0] — 2026-06-23

### Added
- `content/post-2379-short-term-vs-long-term-signals.md` — Серія 2376–2385 «Зворотний зв'язок без тренера» пост 4/10. Шум і сигнал: як self-coached атлет відрізняє короткострокові коливання від реальних тенденцій адаптації. Центральна теза: одна точка даних — шум; тенденція — сигнал. Чотири часових горизонти і їх значення: горизонт 1 (всередині сесії) — корекція поточного тренування; горизонт 2 (2–5 днів) — рішення про готовність до наступного тренування; горизонт 3 (7–14 днів) — мікрорішення про прогресію; горизонт 4 (4–8 тижнів) — блокова оцінка і стратегічні рішення. Три фільтруючих питання перед будь-яким висновком: кількість точок даних, контрольованість умов, відповідність горизонту до питання. Hopkins et al. (1999): типова варіація перформансу ±2–4% без реальної зміни форми. Plews et al. (2013): 7-денний rolling average HRV значно надійніший за одиночне значення. Hellard et al. (2006): відношення сигнал-шум у короткострокових вимірюваннях надто низьке для розрізнення «хорошого дня» і «реального покращення форми» без послідовного збору. Stone et al. (2007): мінімальний горизонт 7–14 днів для ідентифікації тенденцій адаптації. Zourdos et al. (2016): надійність RPE вища при зіставних умовах між сесіями. Hooper et al. (1995): суб'єктивні wellbeing шкали при послідовному зборі точніше виявляють початок overreaching. «Правило трьох»: 1 спостереження → запис; 2 → моніторинг; 3 підряд → дія. Таблиця мінімальних точок даних за типом питання (додати підхід сьогодні / підвищити наступного тижня / оцінити програму). Ієрархія горизонтів: рішення рухаються вниз (довгостроковий контекст інформує короткострокові) але не вгору. Чотири типові діагностичні помилки: важка сесія = проблема; хороша сесія = готовий прогресувати; 2 тижні без прогресу = стагнація; суб'єктивне відчуття прогресу = реальний прогрес. clinical-safety: NOT_REQUIRED. sports-scientist review pending.

### Changed
- `blog.html` — blog card added for post-2379 (після post-2378, перед post-12).
- `STATUS.md` — Total posts 2053 → 2054; Editorial 2376–2385 progress 3/10 → 4/10; post-2379 summary added; Next оновлено на post-2380.
- `VERSION` — 8.24.0 → 8.25.0 (MINOR: новий content post серії 2376–2385).

## [8.24.0] — 2026-06-23

### Added
- `docs/SOC2_READINESS.md §102` — Evidence Table Patch: DEL-E-001 / WBK-E-001 / CHN-E-001 Registration (COST_MODEL §43.9 · C1.2 / CC4.1 / CC5.2 / CC6.1 / CC7.1). Registers three post-churn lifecycle SOC 2 evidence artefacts defined in COST_MODEL §43.9 (v1.0, 2026-06-23) into the §79.4 master evidence table. Resolves WIN-E-001 ID collision: COST_MODEL §43.9 reused "WIN-E-001" for the winback programme report, colliding with the existing §87.2 deal-close WIN-E-001 (`winloss/` path); disambiguated in §79.4 by registering the winback artefact as **WBK-E-001** (alias documented in §102.2). Three new rows in §79.4 (after REN-OBS-E-001): DEL-E-001 (C1.2/CC4.1 — annual deletion certificate archive, 7yr, `deletions/`); WBK-E-001 (CC4.1/CC5.2 — annual winback programme aggregate report, 3yr, `winback/`); CHN-E-001 (CC6.1/CC7.1 — quarterly OFFBOARD-CHAIN-01 deprovisioning compliance report, 3yr, `offboarding/`). Three §80.3 R2 subfolders added (`deletions/`, `winback/`, `offboarding/`) with `form-api NO ACCESS` invariants; `offboarding/` also retroactively formalises the folder referenced by OFB-E-005/006 since §67.8 v3.9.1 (2026-06-15). §80.4 Vanta mirror list extended: DEL-E-001/WBK-E-001/CHN-E-001/OFB-E-005/OFB-E-006. Closes COST_MODEL §43.10 item 5 (P1/M11) and §101.6 item 3 (P1/M11).

### Changed
- `docs/SOC2_READINESS.md` — §79.4 three new evidence rows (DEL-E-001/WBK-E-001/CHN-E-001); §80.3 three new R2 subfolders; §80.4 Vanta list extended; §101.6 item 3 marked `[x] Done`. v3.26.0 → v3.27.0.
- `docs/COST_MODEL.md` — §43.10 item 5 marked `[x] Done` with WBK-E-001 disambiguation note. v1.0 unchanged (patch to checklist status only).
- `VERSION` — 8.23.0 → 8.24.0 (MINOR: new §102 evidence registration section in SOC2_READINESS).

## [8.23.0] — 2026-06-23

### Added
- `content/post-2378-when-to-increase-load.md` — Серія 2376–2385 «Зворотний зв'язок без тренера» пост 3/10. Алгоритм прогресії навантаження для self-coached атлета. Два типи рішення: мікрорішення між сесіями і блокове рішення після 4–6 тижнів. Три-перевірочний мікроалгоритм: RPE стабільний або знижений 2+ сесії підряд; технічна якість підтримана; відновлення адекватне. Таблиця рішень: всі три позитивні → підвищити; 2/3 → зберегти; 1/0 → знизити або адаптована сесія. Блоковий алгоритм — три маркери: e1RM зріс, RPE-тренд знизився, технічна якість підтримана наприкінці блоку. Чотири типові помилки: одна хороша сесія як сигнал; підвищення без перевірки техніки; ігнорування відновлення при позитивному плані; сесійна прогресія після 2+ років стажу. Межа консерватизм vs. бездіяльність: якщо всі три позитивні а атлет не підвищує — не обережність. Zourdos et al. (2016) ICC 0.89–0.94; Helms et al. (2016) мінімум 2–4 тижні калібровки; Kraemer & Ratamess (2004) алостатичне навантаження; Rippetoe (2009) LP вікно застосування; Wulf et al. (2010); Hooper et al. (1995); Plews et al. (2013); Еплі (1985) e1RM формула. clinical-safety: NOT_REQUIRED. Blog card added.

### Changed
- `blog.html` — blog card added for post-2378 (after post-2377).
- `README.md` — post-2377 → draft · v8.22.0; post-2378 → draft · v8.23.0; серія 2386–2395 «Навантажувальний менеджмент: операційна рамка управління обсягом» added as proposed (10 topics).
- `STATUS.md` — content table updated: post-2378 added (Total posts 2052 → 2053); Editorial 2376–2385 progress updated to 3/10 · v8.23.0; next updated to post-2379.
- `VERSION` — 8.22.0 → 8.23.0 (MINOR: new editorial post, new series roadmap).

---

## [8.22.0] — 2026-06-23

### Added
- `content/post-2377-what-to-measure-and-what-not-to.md` — Серія 2376–2385 «Зворотний зв'язок без тренера» пост 2/10. Ієрархія метрик: три рівні — первинні (вага/підходи/повт/RPE), контекстуальні (сон, загальний стан, HRV-тренд), прогресні (e1RM, обсяг, технічний аудит). Saw et al. (2016): суб'єктивні самооцінки узгоджуються з об'єктивними маркерами при регулярному зборі. Zourdos et al. (2016) RPE ICC 0.89–0.94. Plews et al. (2013) 7-денний ковзний середній HRV. Leproult & Van Cauter (2011) сон і тестостерон. DOMS, «відчуття помпи» — не маркери прогресу. Damas et al. (2016) DOMS як запальна реакція. Hubal et al. (2005) n-of-1. Правило корисності: якщо метрика не змінює рішення — шум. Мінімальна інформаційна система (первинне, контекстуальне, прогресне). clinical-safety: NOT_REQUIRED. Blog card added.
- `content/post-2350-self-assessment-training-progress.md` — Серія 2346–2355 «Операційні інструменти self-coached атлета» пост 5/10. Чотири виміри прогресу: силовий вихід, обсягова ємність, технічна якість, відновлення. Горизонт оцінки мінімум 4–8 тижнів. Damas et al. (2016): перші 3–4 тижні переважно нейральна адаптація. Meeusen et al. (2013) мінімальний горизонт 4 тижні для діагностики. Чотири практичні методи: e1RM початок vs. кінець блоку (Reynolds 2006, Chapman 1998, Еплі формула), тижневий обсяг (Israetel et al. 2019 MRV), benchmark session (Zourdos et al. 2016), технічний аудит відео (Wulf et al. 2010, Miall & Wolpert 1996). Три систематичних помилки: відчуття замість даних (Helms 2016), чужий прогрес замість n-of-1 (Hubal 2005 розкид 0–250%), оцінка до деловантажу (Halson 2014 суперкомпенсаційне вікно). П'ять запитань протоколу блокової самооцінки. clinical-safety: NOT_REQUIRED. Blog card added.

### Changed
- `blog.html` — blog cards added for post-2377 (after post-2376) and post-2350 (prepended to grid, above post-2349).
- `STATUS.md` — content table updated: post-2377 and post-2350 added; series progress updated; next priorities updated.
- `VERSION` — 8.21.1 → 8.22.0 (MINOR: two new editorial posts, two series advanced).

---

## [8.21.1] — 2026-06-23

### Changed
- `docs/OBSERVABILITY.md` — §12.6 v1.4 patch: pg_cron **job 43** `deletion_sla_monitor` registered in canonical registry. Schedule `0 */6 * * *`; 7h freshness window (1-run tolerance). P1 alert at day 25 (5-day GDPR Art. 17 warning), P0 escalation at day 29 (1-day pre-breach). Emits `enterprise.deletion_sla_warning` HIGH/7yr and `enterprise.deletion_sla_critical` CRITICAL/7yr. Privacy invariant: payload contains only `tenant_id`, `days_since_deletion_requested`, `alert_tier` (no individual user data). Freshness window note extended. Document version v5.0.0 → v5.0.1.
- `docs/DATA_MODEL.md` — §44.11 cross-reference obligation for OBSERVABILITY §12.6 pg_cron job 43 closed (🟡 → 🟢, v1.26 → minor patch).
- `docs/COST_MODEL.md` — §43.10 item 3 status marked `[x] Done (documentation)`: job 43 registered in OBSERVABILITY §12.6; remaining physical deploy obligation noted (platform-engineer, M10).
- `VERSION` — 8.21.0 → 8.21.1 (patch: extension of existing OBSERVABILITY §12.6 registry).

---

## [8.21.0] — 2026-06-23

### Added
- `content/post-2376-training-log-as-decision-tool.md` — Серія 2376–2385: Зворотний зв'язок без тренера (пост 1/10). «Тренувальний журнал як інструмент рішень, а не архів». Три рівні тренувальних рішень (сесія / між сесіями / блоковий горизонт); мінімально достатній запис (вага + RPE + рядок контексту); тижневий огляд RPE-дрейфу і стану (5 хв); блоковий огляд (20 хв, раз на 4–6 тижнів); три систематичні помилки (вибіркова пам'ять, конфлейт «виконано» і «прогрес», confirmation bias при читанні); практичний розбір блокового аналізу Upper/Lower; алгоритм прийняття рішення «збільшити навантаження чи ні» на основі RPE-трендів у журналі. clinical-safety: NOT_REQUIRED. sports-scientist review pending.

### Changed
- `README.md` — додано нову серію 2376–2385 «Зворотний зв'язок без тренера» (10 пропозицій); post-2376 → draft.
- `blog.html` — додано card для post-2376 (перед post-12 у хронологічному порядку).
- `STATUS.md` — Total posts 2049 → 2050; post-2376 added; version header v8.20.0 → v8.21.0; Editorial 2376–2385 started 1/10; Next: post-2377.
- `VERSION` — 8.20.0 → 8.21.0 (minor: новий пост серії 2376–2385).

---

## [8.20.0] — 2026-06-23

### Added
- `content/post-2349-undulating-load-without-program.md` — Серія 2346–2355: Операційні інструменти self-coached атлета (пост 4/10). «Хвилеподібне навантаження без програми: мінімальна структура для реального тижня». Verkhoshansky (1986) принцип акомодації: повторний однаковий стимул → знижений адаптаційний відгук; варіація — фізіологічна необхідність. Rhea et al. (2002): DUP vs. LP за 12 тижнів — +28.8% vs. +14.4% 1ПМ при ідентичному об'ємі; різниця в архітектурі. Colquhoun et al. (2017): DUP у конкурентних пауерліфтерів. Kenttä & Hassmén (1998) total stress load: жорстка програма ігнорує сукупний стрес. Три характери навантаження: Н (85–92% / RPE 8–9.5 / 1–4 повт — нейральна адаптація), С (70–80% / RPE 7–8 / 6–10 повт — гіпертрофія), Л (55–65% / RPE 5–6 / 12–15 повт — технічна точність і активне відновлення). Ротаційна схема Н/С/Л з правилом: Н→Н заборонено. Zourdos et al. (2016) — калібрувальний підхід як RPE-вимірювання (ICC 0.89–0.94): інтеграція з протоколом A/B/C (post-2347). Israetel et al. (2019) MEV/MAV: об'ємний внесок трьох характерів. Три мінімальні інформаційні вимоги перед сесією. Чотиритижневий мікроцикловий огляд (зв'язок із журналом post-2346). clinical-safety: NOT_REQUIRED. sports-scientist review pending.

### Changed
- `blog.html` — додано card для post-2349 (перед post-2348).
- `STATUS.md` — Total posts 2048 → 2049; post-2349 added; version header v8.19.0 → v8.20.0; Editorial 2346–2355 прогрес 3/10 → 4/10; Next: post-2350.
- `VERSION` — 8.19.1 → 8.20.0 (minor: новий пост серії 2346–2355).

---

## [8.19.1] — 2026-06-23

### Changed
- `docs/AUDIT_LOG_SCHEMA.md` — v2.37 → v2.38: registered two DEC-030 events defined in INCIDENT_RESPONSE.md §17 but absent from the schema registry. `incident.linear_ticket_linked` (LOW, 7 yr): emitted by `siem-incident-automator` Worker within 62 s of `incident.opened` (auto) or by IC via PAM-elevated `POST /internal/v1/incident/link-ticket` (manual); payload `{ incident_id, linear_ticket_url, linked_by: "auto" | "manual_ic" }`; SOC 2 CC7.4 evidence source IR-AUTO-E-002. `incident.pii_risk_detected` (MEDIUM, 7 yr): advisory-only; emitted when runtime blocklist detects potential PII pattern in IC `reason` free-text field; payload `{ incident_id, matched_pattern_category: "email"|"national_id"|"uuid_dump"|"art9_term", false_positive_likely: bool }`; SOC 2 P3.2 evidence source IR-AUTO-E-003. Zod schemas (`IncidentLinearTicketLinkedSchema`, `IncidentPiiRiskDetectedSchema`) added. IRCHAIN-01 updated: every `incident.opened` must pair with exactly one `incident.linear_ticket_linked`. IR-PII-CHAIN-01 invariant added: `incident.pii_risk_detected` must immediately follow `incident.severity_changed` for same `incident_id`. Retention table updated. Intro blurb: "Ten events" → "Twelve events". Closes §17.6 P0/M4 checklist items 2 and 5.
- `docs/INCIDENT_RESPONSE.md` — §17.6 checklist items 2 and 5 marked `[x] Done` (schema registration complete 2026-06-23).
- `VERSION` — 8.19.0 → 8.19.1 (patch: AUDIT_LOG_SCHEMA.md v2.38 extension).

---

## [8.19.0] — 2026-06-23

### Added
- `content/post-2348-technical-error-vs-weak-muscle.md` — Серія 2346–2355: Операційні інструменти self-coached атлета (пост 3/10). «Технічна помилка vs. слабкий м'яз: як розрізнити і що з цим робити». Два класи причин технічного збою: Клас 1 — моторна помилка (патерн некоректний); Клас 2 — силовий дефіцит (патерн є, м'яз не витягує під навантаженням). Wulf et al. (2010) — зовнішній фокус ефективніший за внутрішній у моторному навчанні. Wulf & Lewthwaite (2016) OPTIMAL theory — перевантаження уваги знижує моторне навчання. Cook (2010) FMS — паттерн-обмеження видимі без навантаження. Boren et al. (2011) — EMG-профіль глютеус медіус: clamshell і side-lying abduction вищі ніж стандартні варіанти. McGill (2007) — втручання ефективне лише при правильно визначеній причині. Comerford & Mottram (2001) — secondary motor control impairment. Hodges & Franks (2002) — augmented feedback (відео) прискорює моторне навчання. Три ознаки кожного класу. Трикроковий діагностичний протокол: тест навантаження → тест підказу → тест ізоляції. Шарований сценарій (обидва класи одночасно). Обмеження протоколу. clinical-safety: NOT_REQUIRED. sports-scientist review pending.

### Changed
- `blog.html` — додано cards для post-2347 (був відсутній) і post-2348; обидва до post-2346.
- `README.md` — post-2348 у таблиці серії 2346–2355 змінено з `proposed` на `draft · v8.19.0`; додано Editorial series 2366–2375 «Силові адаптації і стелі: діагностика без тренера» (10 proposed).
- `STATUS.md` — Total posts 2047 → 2048; post-2348 added; version header v8.18.0 → v8.19.0; Editorial 2346–2355 прогрес 2/10 → 3/10.
- `VERSION` — 8.18.0 → 8.19.0.

---

## [8.18.0] — 2026-06-23

### Added
- `content/post-2347-low-readiness-protocol.md` — Серія 2346–2355: Операційні інструменти self-coached атлета (пост 2/10). «Дні з низькою готовністю: операційний протокол замість „тренуватись чи ні"». Центральний аргумент: питання «тренуватись чи ні» — хибна рамка; правильна рамка — «який протокол застосовується сьогодні». Три операційних режими: A (повна сесія — RPE калібрувального підходу ≤6, ранок ≥9/15), B (адаптована — RPE 7–8 або ранок 5–8/15: −10–15% ваги, той самий об'єм підходів і повторень), C (мінімальна або активне відновлення — RPE ≥8.5 або ранок ≤4/15). Калібрувальний підхід: 70–75% робочого навантаження як щоденна діагностика готовності. Autoregulation: Tuchscherer (2008) RTS — структурна адаптація навантаження, а не «слухай тіло». Zourdos et al. (2016) — ICC 0.89–0.94 надійність RPE-шкали. Buchheit (2014) — HRV і нейром'язова готовність r>0.7. Hooper et al. (1995) — суб'єктивні шкали як еквівалент об'єктивних маркерів. Israetel et al. (2019) — MEV vs. MRV: режим B зберігає MEV. Зв'язок із журналом (post-2346): режими A/B/C фіксуються щосесійно → через місяць видно розподіл → через рік патерн готовності. Термінальний принцип: 200 B-режимів за 5 років = прихований тренувальний об'єм, який вирізняє прогрес від стагнації. clinical-safety: NOT_REQUIRED. sports-scientist review pending.

### Changed
- `README.md` — post-2347 у таблиці серії 2346–2355 змінено з `proposed` на `draft · v8.18.0`.
- `STATUS.md` — Total posts 2046 → 2047; post-2347 added; version header v8.17.0 → v8.18.0; Editorial 2346–2355 прогрес 1/10 → 2/10.
- `VERSION` — 8.17.1 → 8.18.0.

---

## [8.17.1] — 2026-06-23

### Changed
- `docs/SOC2_READINESS.md` — §101 added: Cross-Reference Patch for DATA_MODEL §44 (`enterprise_churn_events` · Migration 0085 · C1.2 / CC6.1 / CC4.1 / CC5.2). Closes the bidirectional three-document chain (COST_MODEL §43 → DATA_MODEL §44 → SOC2_READINESS §101) following the §84–§100 pattern. §101.1 three-document chain table. §101.2 twelve-row canonical DDL source registration: three ENUMs, `ece_deletion_sla_respected` CHECK (C1.2 DDL backstop — dual control layer), `ece_winback_acv_requires_converted` CHECK (CC5.2 WINBACK-CHAIN-01 backstop), `ece_offboarding_before_deletion` CHECK (CC6.1 process sequencing), UNIQUE INDEX `idx_ece_tenant_churn_date` (CC6.1 completeness), two non-unique indexes (CC4.1 performance guarantee), REVOKE form_api (CC6.1 access control), `fehs_at_churn` column (k≥10 aggregate), `offboarding_initiated_at` column (CC7.1 elapsed-time anchor). §101.3 four privacy floor supplements: FORM-internal only, tenant_manager exclusion (DDL proof query returns 0), fehs_at_churn tenant-aggregate, offboarding timestamps FORM-process-operational. §101.4 five-row SOC 2 criteria mapping supplement (C1.2 dual evidence paths, CC6.1 three DDL layers, CC4.1 O(log n) guarantee, CC5.2 WINBACK-CHAIN-01 DDL backstop, CC7.1 bounded-table compliance query). §101.5 four obligations closed (§84–§100 pattern, COST_MODEL §43.10 item 10, DATA_MODEL §44.11, DATA_MODEL §44.10 item 6 — all 🟢). §101.6 three-item checklist (P0 migration 0085, P1 DEC-030 events, P1 §79.4 registration). Header v3.25.7 → v3.26.0.
- `docs/COST_MODEL.md` — §43.10 item 10 marked `[x] Done` (SOC2_READINESS §101 authored 2026-06-23 v3.26.0).
- `docs/DATA_MODEL.md` — §44.10 item 6 marked `[x] Done`; §44.11 SOC2_READINESS §101 cross-ref obligation updated 🟡 Pending → 🟢 Closed.
- `VERSION` — 8.17.0 → 8.17.1 (patch: SOC2_READINESS §101 extension).

---

## [8.17.0] — 2026-06-23

### Added
- `content/post-2346-training-log-retrospective-analysis.md` — Серія 2346–2355: Операційні інструменти self-coached атлета (пост 1/10). «Як читати власний журнал тренувань: структурований підхід до ретроспективного аналізу». Центральний аргумент: більшість атлетів ведуть журнал як квитанцію (підтвердження факту тренування), а не як діагностичний інструмент (відповідь на питання «чи працює тренування»). Три рівні ретроспективи: сесійна (5 хв, одразу після — відхилення від плану?) / мікроциклова (10–15 хв, початок тижня — тренд за тиждень?) / блокова (30–45 хв, кінець блоку — блок спрацював?). Три RPE-патерни: прогресивний (нормальний) / стабільний при зростаючому навантаженні (позитивна адаптація) / зростаючий при стабільному навантаженні (негативний сигнал — три тижні поспіль = системний). e1RM: корисний трендовий інструмент при стандартизованих умовах, ненадійний при порівнянні між вправами або різними діапазонами повторень. П'ять антипатернів: прокручування журналу замість аналізу; суб'єктивна оцінка без числових прив'язок; пікові значення замість трендових; читання успіхів з ігноруванням регресів; відсутність тимчасової рамки для висновку. FORM/Victor integration: AI-коуч читає числові тренди, але тільки якщо журнал має структуровані дані. Meeusen et al. (2013) — зростаючий RPE при стабільному навантаженні — ранній маркер overreaching. Saw et al. (2016) — суб'єктивний моніторинг ефективний як об'єктивний. Foster et al. (2001) — sRPE. Zourdos et al. (2016) і Helms et al. (2016) — RPE calibration. Issurin (2010) — блокова ретроспектива. Reynolds et al. (2006) — e1RM точність. Hubal et al. (2005) — суб'єктивна оцінка прогресу ненадійна. clinical-safety: NOT_REQUIRED. sports-scientist review pending. Blog card added.

### Changed
- `blog.html` — card added for post-2346 (placed above post-2345).
- `STATUS.md` — Total posts 2045 → 2046; post-2346 added; version header v8.16.0 → v8.17.0.
- `VERSION` — 8.16.0 → 8.17.0 (minor: series 2346–2355 started, post-2346 added).

---

## [8.16.0] — 2026-06-23

### Added
- `content/post-2345-recovery-synthesis.md` — Серія 2336–2345: Відновлення як тренувальна змінна (пост 10/10 · СИНТЕЗ). Концептуальний синтез: адаптація є продуктом відновлення, а не тренування. Selye H (1950) ЗАС: alarm → resistance → exhaustion — адаптація відбувається під час фази резистентності. Zatsiorsky & Kraemer (2006): тренувальний ефект = навантаження × якість відновлення. Banister (1991) fitness-fatigue модель: деловантаж знижує втому, не торкаючись фітнесу — «виявляє» накопичену адаптацію. Kenttä & Hassmén (1998) «Overtraining and Recovery»: total stress load (не тільки тренувальне навантаження) визначає стан системи. Issurin (2010): адаптаційний потенціал концентрованих стимулів залежить від якості відновного вікна. Дев'ять механізмів серії синтезовані через єдину рамку: кожен пост = окремий важіль того самого процесу. Чотирирівнева ієрархія важелів: сон (1) ≫ управління загальним стресом (2) ≫ харчування (3) ≫ модальності (4). Проблема індивідуалізації: відновлення більш індивідуальне, ніж тренування — популяційні протоколи є відправними точками. Термінальний принцип: якість відновлення ≥ обсяг тренувань для intermediate/recreational. Bridge → серія 2346–2355 (операційні інструменти self-coached атлета). clinical-safety: NOT_REQUIRED. sports-scientist review pending. Blog card added. **Editorial 2336–2345 COMPLETE 10/10.**

### Changed
- `blog.html` — card added for post-2345 (placed above post-2344).
- `STATUS.md` — Editorial 2336–2345: IN PROGRESS 8/10 → **COMPLETE 10/10**; Total posts 2044 → 2045; version header v8.14.0 → v8.16.0.
- `VERSION` — 8.15.0 → 8.16.0 (minor: series 2336–2345 complete, new synthesis post).

---

## [8.15.0] — 2026-06-23

### Added
- `content/post-2344-recovery-operating-system.md` — Серія 2336–2345: Відновлення як тренувальна змінна (пост 9/10). Практична операційна система відновлення для self-coached атлета. Saw et al. (2016) систематичний огляд: суб'єктивні показники (втома, сон, хворобливість) предиктивні настільки ж як об'єктивні маркери (HRV, кортизол) — надійний сенсор доступний щодня без обладнання. Hooper et al. (1995): суб'єктивний моніторинг у плавців виявляв перевантаження до змін у крові. Buchheit (2014): traffic light підхід — протокол має бути простим для щоденного виконання. Foster et al. (2001): sRPE × хвилини = тижневий тренувальний об'єм, кращий предиктор адаптації ніж пікова інтенсивність. Meeusen et al. (2013) ECSS/ACSM: зростаючий RPE при стабільному навантаженні — маркер functional overreaching. Сигнальна ієрархія: рівень 1 (структурні/тижневі) → рівень 2 (добові агрегати: сон/енергія/хворобливість/HRV) → рівень 3 (сигнали сесії). Ранковий протокол 3 хвилини: три питання по 1–5, сума →рівень дня. Дерево рішень 4 рівні: зелений (за планом) / жовтий (−5–10% інтенсивність, об'єм зберегти) / помаранчевий (технічна або @60–70%) / червоний (активне відновлення). HRV як коригуючий фактор поверх балу. Тижнева калібровка: середній бал тижня (норма ≥10), кількість помаранчевих/червоних (норма ≤1), RPE-динаміка ключових вправ, деловантаж до того як відчуєш потребу. П'ять антипатернів: «важлива сесія», надмірний трекінг без рішень, бінарне мислення, мотивація як сигнал готовності, деловантаж «коли відчую». clinical-safety: PASS. sports-scientist review pending. Blog card added.
- `README.md` — roadmap: Editorial series 2346–2355 «Self-coaching як система рішень» proposed (10 тем).

### Changed
- `blog.html` — card added for post-2344.
- `STATUS.md` — recovery block 2336–2345 progress updated: 7/10 → 8/10; post-2343 SKIPPED (clinical-safety HARD VETO: «ментальна втома» категорія «ментальне» — потребує founder review); post-2344 added to content library entry.
- `VERSION` — 8.14.0 → 8.15.0 (minor: new recovery content post + roadmap series 2346–2355).

### Skipped
- `post-2343` (psychological-fatigue-training) — clinical-safety HARD VETO. Тема «психологічна/ментальна втома» потрапляє в категорію «ментальне» згідно зі скоупом content-engine. Потребує founder review та clinical-safety deep review перед написанням.

---

## [8.14.0] — 2026-06-23

### Added
- `content/post-2341-stress-allostatic-load.md` — Серія 2336–2345: Відновлення як тренувальна змінна (пост 6/10). Стрес як тренувальна змінна: чому алостатичне навантаження об'єднує всі стресори в один рахунок. McEwen (1998, 2007): алостаз vs. гомеостаз — стабільність через зміну, а не повернення до фіксованої точки. McEwen & Seeman (1999): хронічна гіперкортизолемія пригнічує пульсативний GH (зв'язок із постом 2336), знижує чутливість до інсуліну, конкурує за прегненолон (прекурсор кортизолу і статевих гормонів), порушує mTORC1-сигналінг. Sapolsky (2004): гострий кортизол — адаптивний; хронічний кортизол від постійної фонової тривоги — руйнівний. Marcora et al. (2009): психобіологічна модель втоми — 90 хвилин когнітивного завдання знизили витривалість до відмови на ~15% при незмінних фізіологічних маркерах; RPE підвищений при тій самій зовнішній роботі. Anderson & Strickland (2018): підвищений психологічний стрес корелює з вищим ризиком травм. Практична матриця трьох рівнів алостатичного навантаження: нормальний (тренуйся за планом), підвищений (−10–15% інтенсивності, −1–2 підходи), різко підвищений (технічна робота або субмаксимальне @60–65% без провини). Відмінність: тренінг як інструмент стрес-менеджменту ≠ тренінг як уникання ≠ тренінг попри явні сигнали виснаження. clinical-safety: NOT_REQUIRED. sports-scientist review pending. Blog card added.
- `content/post-2342-nutrition-recovery-timing.md` — Серія 2336–2345: Відновлення як тренувальна змінна (пост 7/10). Харчування і відновлення: анаболічне вікно, лейциновий поріг, розподіл білка, роль вуглеводів. Schoenfeld & Aragon (2013) систематичний огляд: при адекватному добовому споживанні білка тайминг відносно тренування має мінімальний вплив на гіпертрофію — «вікно» значно ширше 30 хв. MPS-механізм: тренування активує mTORC1, підвищений MPS 24–48 год (Phillips et al., 1997) — не «вікно 30 хвилин». Norton & Layman (2006): лейциновий поріг 2–3 г/прийом для запуску mTORC1; у 20–40 г якісного тваринного білка ~1.5–3.5 г лейцину. Moore et al. (2009): дозозалежна відповідь MPS до ~20 г (нетреновані), Witard et al. (2014): ~40 г оптимум для тренованих. Areta et al. (2013): рівномірний розподіл 0.4 г/кг × 3–4 прийоми перевищує «блочний» за сумарним MPS. MacDougall et al. (1999): глікоген −30–40% після гіпертрофійної сесії; Burke et al. (2011): повний ресинтез за 24–48 год при нормальному харчуванні — швидкий ресинтез критичний лише при 2-х тренуваннях/день. Ієрархія пріоритетів: загальний добовий об'єм > якість джерел > розподіл по прийомах > тайминг > пост-тренувальні вуглеводи. clinical-safety: CONDITIONAL PASS (inline review виконано у frontmatter). sports-scientist review pending. Blog card added.
- `blog.html` — cards added for post-2341 and post-2342.

### Changed
- `STATUS.md` — recovery block 2336–2345 progress updated: 5/10 → 7/10; posts 2341 and 2342 added to content library entry.
- `VERSION` — 8.13.1 → 8.14.0 (minor: two new recovery content posts).

---

## [8.13.1] — 2026-06-23

### Added
- `docs/DATA_MODEL.md §44` — `enterprise_churn_events` canonical DDL registration for migration 0085. Closes `docs/COST_MODEL.md §43.10 item 2` (v1.0, 2026-06-23). Full DDL: three ENUMs (`churn_trigger_enum` 4 values, `churn_reason_enum` 6 values, `winback_status_enum` 7 values); 22-column table; three CHECK constraints (`ece_offboarding_before_deletion`, `ece_winback_acv_requires_converted`, `ece_deletion_sla_respected` — DDL-layer GDPR Art. 17 backstop); three indexes (UNIQUE `(tenant_id, churn_date)`, `churn_date DESC`, partial `winback_status NOT IN (converted, abandoned)`); four RLS policies (form_admin ALL, compliance_officer SELECT, form_system ALL; tenant_manager no policy, form_api REVOKED); four COMMENT ON annotations. §44.2 migration dependency chain 0083 → 0084 → 0085 with FK strategy rationale (RESTRICT on tenant_id/contract_id; SET NULL on winback_contract_id). §44.3.2 six-item staging checklist including day 36 ece_deletion_sla_respected violation test. §44.4 22-row column summary + three CHECKs table. §44.5 RLS with three DDL auditor proof queries. §44.6 three chain invariants: OFFBOARD-CHAIN-01 (24h elapsed query), WINBACK-CHAIN-01 (DDL consistency check), DELETION-CHAIN-01 (SLA monitoring queries at day 25/29/35). §44.7 FEHS sourcing from `tenant_engagement_snapshots` with NULL handling and privacy invariant. §44.8 EXPLAIN ANALYZE for all three indexes at 500-row fleet scale. §44.9 SOC 2 evidence cross-references: DEL-E-001 (C1.2/CC4.1), WIN-E-001 (CC4.1/CC5.2), CHN-E-001 (CC6.1/CC7.1) with DDL-layer invariant column and auditor narratives. §44.10 six-item implementation checklist (3× P0/M10, 2× P1/M11, 1× P2). §44.11 four cross-reference obligations (COST_MODEL §43.10 item 2 🟢 Closed; SOC2_READINESS §101 patch, AUDIT_LOG_SCHEMA DEC-030 events, OBSERVABILITY §12.6 job 43 all 🟡 Pending). DATA_MODEL v1.25 → v1.26. Owner: enterprise-architect + compliance-officer.

### Changed
- `docs/COST_MODEL.md §43.10 item 2` — status updated from `[ ]` to `[x] Done (documentation)`: `docs/DATA_MODEL.md §44` authored 2026-06-23 (v1.26). Remaining: apply migration 0085 to staging + production. COST_MODEL header unchanged (checklist sync patch). Owner: enterprise-architect + compliance-officer.
- `VERSION` — 8.13.0 → 8.13.1 (patch: §44 extension of existing DATA_MODEL.md).

---

## [8.13.0] — 2026-06-23

### Added
- `content/post-2340-hrv-recovery-signal.md` — Серія 2336–2345: Відновлення як тренувальна змінна (пост 5/10). HRV як сигнал відновлення: чому одноразовий вимір — шум, а rolling average — сигнал. Flatt & Esco (2016): 7-денний rolling average значно перевершує одноразовий вимір як предиктор функціонального стану. RMSSD і парасимпатичний тонус — механізм. 12+ конфундерів одноразового виміру: час пробудження, алкоголь, температура, поза, гідратація, кофеїн, менструальна фаза та ін. Meeusen et al. (2013) ECSS/ACSM: стійке пригнічення HRV-тренду 4+ днів — маркер non-functional overreaching. Чому популяційні норми марні — тільки особистий базовий рівень. Nóbrega et al. (2018): Apple Watch Series 3 vs. Polar H7 у стані спокою r=0.95. Специфіка силових атлетів: HRV-відповідь варіабельніша і менш передбачувана, ніж у видах витривалості (Schoenfeld 2022, Nuzzo 2023). Altini / HRV4Training підхід. Практичний протокол: 21-денна калібровка, 7-денний rolling average, контекст важливіший за число, не оптимізувати HRV як самоціль. Коли довіряти (стійке пригнічення 4+ днів, після хвороби, після важкого блоку). Коли ігнорувати (перший місяць нового блоку, домінуючий ментальний стрес, < 3 тижнів базового рівня). clinical-safety: NOT_REQUIRED. sports-scientist review pending. Blog card added.
- `blog.html` — card added for post-2340.
- `README.md` — Editorial series 2356–2365 «Програмування під реальне життя» added as proposed (10 topics: мінімальна частота для збереження адаптації, два тренування на тиждень, відрядження, повернення після перерви, форс-мажорний тиждень, один підхід vs. три, суперсети, тренування до відмови, домашнє тренування, синтез); README: 2339 → draft · v8.12.0, 2340 → draft · v8.13.0.
- `VERSION` — 8.12.0 → 8.13.0 (minor: new content post + roadmap extension).

---

## [8.12.0] — 2026-06-23

### Added
- `content/post-2339-ice-baths-cryotherapy.md` — Серія 2336–2345: Відновлення як тренувальна змінна (пост 4/10). CWI і кріотерапія: фізіологічний механізм (вазоконстрикція → реактивна вазодилатація, зниження температури тканин, симпатична активація, протизапальний ефект). Roberts et al. (2015): 12 тижнів систематичного CWI після силових знизили приріст м'язової маси, активацію сателітних клітин (Pax7+, MyoD+) і кількість міоядер — гостре запалення після тренування є адаптаційним сигналом. Bleakley et al. (2012) Cochrane: CWI знижує DOMS і суб'єктивну втому (17 РКД, 366 учасників). Fyfe et al. (2019): CWI пригнічує mTORC1-залежний синтез м'язового білка. Yamane et al. (2006): холодове занурення після тренувань притупляло приріст сили за 4 тижні. Hohenauer et al. (2015) мета-аналіз 36 досліджень: CWI перевищує пасивне відновлення за зниженням DOMS на 24–96 год. CWI vs WBC: механізми і доказова база. Протокол: 10–15°C, 10–15 хвилин, перші 60 хвилин після навантаження. Операційна матриця: виправдано (мультиденні змагання, великий об'єм endurance) vs. контрпродуктивно (силовий або гіпертрофійний мезоцикл 3–5 сесій/тиждень). Суб'єктивне відновлення ≠ фізіологічне відновлення. clinical-safety: NOT_REQUIRED. sports-scientist review pending. Blog card added.
- `blog.html` — card added for post-2339.
- `VERSION` — 8.11.1 → 8.12.0 (minor: new content post).

---

## [8.11.1] — 2026-06-23

### Changed
- `docs/COST_MODEL.md` v2.13 — §43 Enterprise Post-Churn Economics, Offboarding Cost Model & Logo Winback Analytics. Closes the enterprise lifecycle chain §40→§41→§42→§43. Offboarding cost model per tier (Starter $1,000 / Growth $1,750 / Enterprise $2,900 [ESTIMATE]); GDPR Art. 17 deletion economics and non-compliance risk cost; logo winback probability matrix by churn_reason × wau_band (Green champion_left 40–50% at 12m; Red low_utilization 3–8%); winback vs. new-logo break-even analysis (5.8× cheaper for Green band); three new DEC-030 HMAC-chained events (`enterprise.winback_initiated` HIGH/7yr, `enterprise.winback_converted` CRITICAL/7yr, `enterprise.deletion_certificate_issued` CRITICAL/7yr); three chain invariants (OFFBOARD-CHAIN-01 24h, WINBACK-CHAIN-01 365d, DELETION-CHAIN-01 35d/30d GDPR); migration 0085 `enterprise_churn_events` DDL summary (cross-ref to DATA_MODEL §44 — pending); three SOC 2 evidence artefacts (DEL-E-001 C1.2/CC4.1, WIN-E-001 CC4.1/CC5.2, CHN-E-001 CC6.1/CC7.1); pg_cron job 43 `deletion_sla_monitor` (every 6h) to register in OBSERVABILITY §12.6; ten-item implementation checklist; four open questions including OQ-WIN-03 (P0 — tenant_id UUID reuse policy for winback).
- `VERSION` — 8.11.0 → 8.11.1 (patch: extension of existing COST_MODEL doc).

---

## [8.11.0] — 2026-06-23

### Added
- `content/post-2338-foam-roller-massage-stretching.md` — Editorial 2336–2345 «Відновлення як тренувальна змінна», пост 3/10. Піно-ролик, масаж і стретчинг: що каже наука — і де вона мовчить. Schleip (2003): фасція не «звільняється» роликом — структурний тиск у 3–4 рази перевищує масу тіла; нейросенсорний механізм через механорецептори; gate control theory. Pearcey et al. (2015): foam rolling знижує DOMS і performance-маркери через 24–72 год, але n малий. MacDonald et al. (2013, 2014): тимчасовий ROM через ЦНС, не структурні зміни; ефект 30–60 хв. Simic et al. (2013): meta-аналіз 104 досліджень — статичний стретчинг > 30 сек знижує максимальну силу на 5,1–8,3%. Behm & Chaouachi (2011): дефіцит сили до 60 хв після статичного стретчингу. Feland et al. (2001): 60-секундні утримання значно перевершують 15- і 30-секундні для довгострокового ROM (n=62). Guo et al. (2017): масаж — помірна доказова база DOMS зниження, плацебо-ефект неізольований. McMillian et al. (2006): динамічна розминка перевершує статичну за атлетичними показниками. Операційна матриця 5 інструментів. clinical-safety: NOT_REQUIRED.

### Changed
- `blog.html` — blog cards додані для post-2337 (активне vs пасивне відновлення, відсутня з v8.10.0) і post-2338 (піно-ролик, масаж і стретчинг). Обидві перед post-2336.
- `README.md` — Editorial series 2346–2355 «Операційні інструменти self-coached атлета» додана до roadmap; post-2338 → draft.
- `STATUS.md` — Editorial 2336–2345 прогрес 2/10 → 3/10; Total posts 2037 → 2038; Next: post-2339 (ice-baths-cryotherapy).
- `VERSION` — 8.10.0 → 8.11.0 (minor: new editorial post + new roadmap series).

---

## [8.10.0] — 2026-06-23

### Added
- `content/post-2337-active-vs-passive-recovery.md` — Editorial 2336–2345 «Відновлення як тренувальна змінна», пост 2/10. Активне vs пасивне відновлення: фізіологічна рамка вибору між двома стратегіями. Кліренс лактату і оптимальна інтенсивність активного відновлення (Menzies et al. 2010, <40% VO₂max); ЦНС-домінантна vs. метаболічна втома як ключовий критерій вибору; венозний і лімфатичний дренаж; DOMS — суб'єктивний vs. структурний вимір; крижані ванни і притуплення гіпертрофічного сигналу (Roberts et al. 2015); матриця прийняття рішень для 5 типових ситуацій; операційний протокол (15–30 хв, <50% ЧСС.макс, RPE 1–3); функціональні маркери відновлення. clinical-safety: NOT_REQUIRED.

### Changed
- `STATUS.md` — Editorial 2336–2345 прогрес 2/10; Total posts 2036 → 2037; Next: post-2338 (nutrition-recovery-window).
- `VERSION` — 8.9.1 → 8.10.0 (minor: new editorial post).

## [8.9.1] — 2026-06-23

### Changed
- `docs/AUDIT_LOG_SCHEMA.md` v2.37 — SCA monitoring sentinel events registered: `security.sca_sla_breach` (HIGH/7yr, SCA-CHAIN-01 anchor) and `system.sca_sla_check_passed` (LOW/1yr, advisory). Retention table +2 rows. Document header corrected v2.35 → v2.37 (v2.36 was body-only, 2026-06-22). Closes OBSERVABILITY §52.10 item 1 (P0/M4).
- `docs/OBSERVABILITY.md` v5.0.0 — §6.2 `sca_vulnerability_monitoring` subsection added: AL-SCA-01 through AL-SCA-05 consolidated alert rules with SOC 2 CC6.8/CC7.1/CC9.2/CC8.1 mapping. §52.10 checklist items 1, 3, 4 marked done.
- `docs/SOC2_READINESS.md` — §54.9 evidence table: SCA-OBS-E-001 (quarterly SCA SLO performance report) and SCA-OBS-E-002 (quarterly pg_cron run history for job 42) registered. §54.11 item 11 annotated: FORM-SCA-001 documented as AL-SCA-01 per OBSERVABILITY §52, pending operational deployment.
- `VERSION` — 8.9.0 → 8.9.1 (patch: extending existing enterprise docs).

---

## [8.9.0] — 2026-06-23

### Added
- `content/post-2336-sleep-primary-recovery-mechanism.md` — Editorial 2336–2345 «Відновлення як тренувальна змінна» post 1/10. Сон як першочерговий механізм відновлення. Van Cauter et al. (2000): 70–75% GH під час сну, основний пульс у першій SWS-фазі. Leproult & Van Cauter (2011): 7 ночей × 5 год — мінус 10–15% тестостерону. Skein et al. (2011): одна ніч < 6 год — мінус 8–11% продуктивності. Ієрархія відновлення 9 рівнів. MPS механізм: GH → IGF-1 → mTORC1. REM і консолідація рухових навичок. Що реально покращує сон (і що ні). Практичний чекліст для self-coached атлета.

### Changed
- `blog.html` — blog card додана для post-2336 (перед post-2326).
- `README.md` — Editorial series 2336–2345 «Відновлення як тренувальна змінна» додана до roadmap; post-2336 → draft.
- `STATUS.md` — Total posts: 2035 → 2036; Editorial 2336–2345 IN PROGRESS 1/10; Next priorities оновлено.
- `VERSION` — 8.8.0 → 8.9.0.

---

## [8.8.0] — 2026-06-23

### Added
- `content/post-2301-winter-intensity-block.md` — Editorial 2296–2305 post 6/10. Зимовий інтенсифікаційний блок: як конвертувати осіннє накопичення. Механізм нейром'язової реалізації (Behm et al. 2002); чотири параметри переходу (об'єм −30%, зона RM 80–92%, звуження варіацій, відпочинок 3–5 хв); шеститижнева структура з перехідним і деловим тижнями; типові помилки (залишатись у накопиченні, різкий перехід, тоннаж як метрика); реалістичний приріст 1RM 5–12%.
- `content/post-2302-spring-peaking.md` — Editorial 2296–2305 post 7/10. Весняний пік: як реалізувати зимові результати і закрити сезон. Тейпер 1–2 тижні (Mujika & Padilla 2003: 40–60% зниження об'єму, інтенсивність ≥ 85%); покроковий протокол тестування 1RM від 50% до 97–100%; документація як відправна точка нового циклу; що робити після піку.
- `content/post-2303-deload-planning.md` — Editorial 2296–2305 post 8/10. Деловий тиждень: плановий vs. реактивний. Фізіологічне обґрунтування (периферійна vs. центральна втома; суперкомпенсація при зниженні об'єму); плановий протокол (40–50% об'єму, 70–80% RM, RPE ≤ 7); реактивний протокол (5 конкретних сигналів); скільки делових тижнів на рік; помилки виконання.
- `content/post-2304-annual-planning.md` — Editorial 2296–2305 post 9/10. Річний план: як скласти, а не написати і забути. Три рівні деталізації (річний, мезоцикл, тиждень); два макроцикли A+B; три кроки складання (якорі, блоки, метрика); принцип буфера 20–25%; три типові помилки (переоцінка стабільності, відсутність проміжних точок, лінійна екстраполяція приросту).
- `content/post-2305-year-review-calibration.md` — Editorial 2296–2305 post 10/10, завершення блоку. Оцінка тренувального року. П'ять питань ретроспективи; Meehan et al. (2011) — ретроспективна самооцінка об'єму завищена на 15–30%; калібрування трьох відправних точок (1RM, прогноз, реальна частота); підсумок серії editorial-2296–2305.

### Changed
- `blog.html` — blog cards додані для постів 2301–2305 (п'ять нових карток перед post-2300).
- `STATUS.md` — Editorial 2296–2305 позначено COMPLETE 10/10; next priorities оновлено.
- `VERSION` — 8.7.0 → 8.8.0.

---

## [8.7.0] — 2026-06-23

### Added
- `docs/OBSERVABILITY.md §52` — Software Composition Analysis (SCA) & Dependency Vulnerability Monitoring Observability. Companion section to SOC2_READINESS §54 (CC6.8/CC7.1/CC9.2/CC8.1). Closes the documented gap where §38 (CI/CD Observability) explicitly excluded SCA and SOC2_READINESS §54.10 item 11 (FORM-SCA-001 alert) remained `[ ]` open with no pg_cron job, no §6.2 alert IDs, no SLOs, and no evidence artefacts registered in OBSERVABILITY.md. §52.1 scope & architecture (three-layer SCA pipeline: Dependabot + npm audit + Snyk; DEC-030 as signal bus; privacy floor). §52.2 RED metrics (Dependabot PR pipeline, npm audit CI gate, Snyk scheduled scan, CVE remediation pipeline). §52.3 three SLOs: SCA-SLO-01 (zero open Critical CVEs beyond 24h — zero-tolerance), SCA-SLO-02 (≥ 95% High CVEs resolved within 7d, quarterly), SCA-SLO-03 (CI scan pass rate ≥ 99.5%/month). §52.4 five alert rules AL-SCA-01–AL-SCA-05 (P1 Critical SLA breach, P2 High SLA breach, P1 CI scan failure, P2 licence violation, P2 risk acceptance expiry). §52.5 pg_cron job 42 `sca_sla_monitor` (`*/15 * * * *`; 20-min freshness; SCA-SLO-01 enforcement SQL; pg_net AL-SCA-01; two new DEC-030 events: `security.sca_sla_breach` HIGH/7yr + `system.sca_sla_check_passed` LOW/1yr). §52.6 §6.2 Consolidated Alert Rules `sca_vulnerability_monitoring` subsection (FORM-SCA-001 → AL-SCA-01 mapping; closes §54.10 item 11). §52.7 DEC-030 chain SCA-CHAIN-01 invariant. §52.8 two evidence artefacts: SCA-OBS-E-001 (CC6.8/CC7.1/CC9.2, quarterly, 3yr), SCA-OBS-E-002 (CC7.2/A1.1, quarterly, 3yr). §52.9 Metabase `SCA & Dependency Vulnerability Health` dashboard. §52.10 seven-item implementation checklist. §52.11 OQ gap tracker (no open questions).

### Changed
- `docs/OBSERVABILITY.md §12.6` — Job 42 `sca_sla_monitor` registered in pg_cron canonical registry (v1.3 patch); freshness note extended to include job 42 (20-min/15-min cadence pattern consistent with job 34). Document header v4.9.6 → v5.0.0.
- `VERSION` — 8.6.0 → 8.7.0.

## [8.6.0] — 2026-06-23

### Added
- `content/post-2326-strength-base-vs-specialization.md` — Editorial 2326–2335, post 1/10. Силова база vs. спеціалізація: чому порядок визначає стелю прогресу. Концептуальна рамка GPP/SPP для recreational атлета без академічного словника. Stone et al. (2002): рівень SPP обмежений рівнем GPP через адаптаційний потенціал. Bompa & Haff (2009): специфічні адаптації вимагають загальної підготовки як попередньої умови. Woo et al. (1987): структурна адаптація (сухожилля) відстає від функціональної (м'яз). Issurin (2010): concentrated adaptational potential — конечний ресурс. Три стани системи: GPP-дефіцит (< 12 міс., техніка при RPE 7 нестабільна), GPP-перехід (12–24 міс.), SPP-готовність (> 18–24 міс., технічна стабільність при RPE 8+). Silові орієнтири (Rippetoe & Bradford 2011) як довідкові точки без нормативного забарвлення. Чотири типових помилки: передчасна спеціалізація, хронічна базова фаза без переходу, підміна «бази» і «базових вправ», повернення після перерви без відновлювального блоку. Мінімальний чекліст переходу (технічний/структурний/силовий аспекти, 10 пунктів). clinical-safety: NOT_REQUIRED. sports-scientist review pending. Blog card added.

### Changed
- `README.md` — Editorial 2326–2335 «Силова база для recreational атлета» added as proposed (10 topics: 2326 strength base vs. specialization; 2327 three movement patterns; 2328 choosing core exercises; 2329 relative strength; 2330 muscle balance and asymmetry; 2331 weak point vs. weak muscle; 2332 technique as strength limit; 2333 when to add exercise; 2334 GPP to SPP transition; 2335 synthesis). post-2326 → draft.
- `blog.html` — blog card for post-2326 (strength-base-vs-specialization) added at top of grid (before post-2220).
- `STATUS.md` — Total posts 2029 → 2030; Editorial 2326–2335 registered as IN PROGRESS 1/10.
- `VERSION` — 8.5.0 → 8.6.0.

## [8.5.0] — 2026-06-23

### Added
- `content/post-2300-force-majeure-programming.md` — Editorial 2296–2305, post 5/10. Форс-мажор і тренування: що зберігати в першу чергу. Три типи форс-мажору — три різних алгоритми: (1) Хвороба: above/below-neck евристика для тренувальних рішень (не медичний діагноз, явний disclaimer); тренування при системній хворобі подовжує одужання; чотирифазна структура повернення (дні 1–3 рух без навантаження; дні 4–7 50–60%; тижні 2–3 вихід на попередні ваги; тиждень 4+ нормалізація); три маркери неповного відновлення (ЧСС спокою, RPE при стандартному навантаженні, суб'єктивне відновлення вранці). (2) Переїзд: підтримувальний bodyweight протокол push/pull/hinge/legs, 2×30–40 хв, RPE ≥ 7 — Calatayud et al. (2015, 2016): при рівній відносній інтенсивності обладнання не визначає гіпертрофійну адаптацію; Mujika & Padilla (2000): 33–66% obj. при збереженій інтенсивності = адаптація 8–16 тижнів. (3) Life disruption (зміна роботи / серйозні зміни в житті): Baumeister et al. (1998) ego depletion — структурне скорочення ресурсу рішень; Holmes & Rahe (1967): переїзд — одна з найстресовіших побутових подій; Wood & Rünger (2016): поведінковий ритм найважче відновити; ієрархія збереження: звичка появи > інтенсивність > об'єм; матриця рішень 4×4 (гострий/тривалий × час є/немає). Загальний принцип: пріоритизуй те, що важче відновити — поведінковий ритм вразливіший за силові адаптації. П'ятикроковий чекліст. clinical-safety: NOT_REQUIRED. sports-scientist review pending. Blog card added.
- `content/post-2140-real-life-training-synthesis.md` — Editorial 2131–2140, post 10/10 (SYNTHESIS — closes block). Синтез: тренування в умовах реального дорослого атлета. Ключова ідея блоку: умови реального атлета — не перешкоди, це і є тренувальне середовище. Десять постів покроково: 2131 (безсонна ніч — RPE-адаптація, не скасування), 2132 (нерегулярний графік — адаптивна структура vs. жорсткий план), 2133 (hotel gym — 4 рухових патерни при RPE ≥ 7), 2134 (2 сесії/тиждень — Ralston et al. 2017: ≥2 є мінімальним для гіпертрофії), 2135 (сезонні перерви — 3 хронології деадаптації, міонуклеарна пам'ять Bruusgaard 2010), 2136 (паралельне навантаження — ego depletion, матриця рішень), 2137 (мінімальний ефективний тиждень — фізіологічний і поведінковий мінімум), 2138 (хронічна втома — Meeusen 2013, три стани, алгоритм), 2139 (зал vs. вдома — де порівнянні, де структурно різні). Три наскрізних принципи: (1) адаптація не вимагає ідеальних умов — первинні детермінанти не залежать від контексту; (2) поведінковий ритм — найвразливіший ресурс (Wood & Rünger 2016); (3) контекст визначає параметри роботи, не її можливість. Операційна система: мінімум 2 сесії/тиждень RPE ≥ 7; ієрархія скорочень (об'єм → допоміжна → частота → інтенсивність); три стани і три відповіді; три маркери паузи; чотирифазна структура після паузи. Чого блок не стверджує: умови мають значення; не все оптимізовується; 2 сесії — не ідеал. Кандидати на блок 2141–2150: програмування для одночасних цілей сила/гіпертрофія; або тренування 35+. clinical-safety: NOT_REQUIRED. sports-scientist review pending. Blog card added. **Editorial 2131–2140 COMPLETE.**
- `blog.html` — blog cards for post-2300 (force-majeure-programming) і post-2140 (real-life-training-synthesis) added in correct chronological positions.

### Changed
- `STATUS.md` — Editorial 2296–2305: IN PROGRESS 4/10 → 5/10; Editorial 2131–2140: IN PROGRESS 9/10 → COMPLETE 10/10; next priorities updated.
- `VERSION` — 8.4.1 → 8.5.0.

## [8.4.1] — 2026-06-23

### Changed
- `docs/OBSERVABILITY.md` — AL-CI-07 (`ci_telemetry_daily_sync` stale > 26h) formally registered in §38.5 alert rules table: P2, PagerDuty `form-devops` + Slack `#devops`, dedup `ci-telemetry-sync-stale`, runbook R-40. Closes INCIDENT_RESPONSE R-40.11 item 4 (P1/M8) and R-40.10 "AL-CI-07 formal registration" post-incident control. Document header v4.9.5 → v4.9.6.
- `docs/INCIDENT_RESPONSE.md` — R-40.11 item 4 marked `[x] Done` (OBSERVABILITY.md v4.9.6, 2026-06-23); R-40.10 "AL-CI-07 formal registration" control updated to Done.
- `VERSION` — 8.4.0 → 8.4.1.

## [8.4.0] — 2026-06-23

### Added
- `content/post-2299-autumn-accumulation-block.md` — Editorial 2296–2305, post 4/10. Осінній накопичувальний блок: як використати найпродуктивніший тренувальний сезон. Три чинники осінньої ефективності: термічне середовище (Nielsen et al. 1993 — зникнення cardiovascular drift і теплового RPE-штрафу), стабільність розкладу (SRA-консистентність), нейром'язова свіжість після вересневого відновлення. MEV/MAV/MRV рамка (Israetel et al. 2019): точні визначення і відмінність від популярних помилок. Schoenfeld et al. (2019): об'єм з дозозалежним ефектом до MRV — прогресія між сесіями vs. постійне підвищення ваги. 6-тижнева структура: MEV-старт (тиждень 1, RPE 6–7, нижче звичного об'єму) → +1 підхід/1–2 тижні (тижні 2–4) → пік об'єму (тиждень 5) → запланований деловантаж −40–60% (тиждень 6). Krieger (2010): різниця 1 vs. множинні підходи потребує коректного відновлення. Hackett et al. (2013): систематично плановані відновні фази → кращі довгострокові адаптації vs. реактивний деловантаж після перевтоми. Диференціація між м'язовими групами за MRV-відповіддю. Чотири помилки: реактивний деловантаж, підвищення інтенсивності замість об'єму, однакові параметри для всіх груп, «надолужити» після пропуску. FORM/Victor: ефективний об'єм (RPE ≥ 7), MRV-сигнал session RPE +1.5 за 2 тижні, деловантаж запланований від дня 1. clinical-safety: NOT_REQUIRED. sports-scientist review pending. Blog card added. README: Editorial 2316–2325 proposed.

### Changed
- `blog.html` — картка для post-2299 додана (Editorial 2296–2305 4/10).
- `README.md` — Editorial 2316–2325 «Тренування в умовах реального життя» додано як proposed (10 тем: хронічний стрес, мінімальна ефективна одиниця, тренування без залу, хронобіологія, shift work, вік 40+, стать і адаптація, хронічна нестача сну, self-coaching навички, синтез).
- `STATUS.md` — total posts 2028 → 2029; Editorial 2296–2305 progress 3/10 → 4/10; Next: post-2300.
- `VERSION` — 8.3.0 → 8.4.0.

---

## [8.3.0] — 2026-06-23

### Added
- `content/post-2297-summer-training-heat-vacation.md` — Editorial 2296–2305, post 2/10. Літній режим: тренування у спеку і навколо відпусток. Cardiovascular drift та конкуренція кровотоку при тепловому стресі (Nielsen et al. 1993): +8–10 уд/хв при +1°C core temp, що еквівалентно підвищенню RPE на 0.5–1 одиниці без змін навантаження. Ftaiti et al. (2010): гострий тепловий стрес знижує м'язову силу незалежно від гідратації. Теплова акліматизація: 10–14 днів дають збільшення об'єму плазми і ефективності потовиділення (Sawka et al. 2011). Mujika & Padilla (2000): при збереженні інтенсивності за зниженого об'єму (−50%) сила не знижується до 3–4 тижнів зупинки. Graves et al. (1988): 1×/тиж з підтриманою інтенсивністю — достатньо для збереження сили 12 тижнів. Три літніх режими (підтримка, легке накопичення, повне тренування). Мінімальна структура під час відпустки. Armstrong et al. (2007): 2% дегідратація знижує силові і когнітивні показники. FORM/Victor: temperature-adjusted RPE, vacation mode 2×/тиж. clinical-safety: NOT_REQUIRED. sports-scientist review pending. Blog card added.
- `content/post-2298-september-return-after-summer.md` — Editorial 2296–2305, post 3/10. Вересень: як повернутись після літа без «старту з нуля». Bruusgaard et al. (2010): міонуклеї зберігаються 3+ місяців після повної зупинки тренувань — м'язова пам'ять як фізіологічний механізм. Ogasawara et al. (2013): реакапіталізація м'язового об'єму після деренування займає 1/3–1/2 часу першого будівництва. Mujika & Padilla (2000): короткострокове (до 4 тижнів) vs. довгострокове деренування — різна картина для сили і витривалості. Graves et al. (1988): нейром'язовий компонент деренується швидше структурного. Структура вересневого повернення: 2 тижні відновлення нейром'язового контролю (RPE 6–7, −20% об'єму), тижні 3–4 повернення до попередніх ваг, тиждень 5+ накопичувальний блок. Два патерни, що руйнують повернення: синдром «я так далеко відпав» і синдром компенсації. FORM/Victor: retention score, transition week, градуальне повернення. clinical-safety: NOT_REQUIRED. sports-scientist review pending. Blog card added.

### Changed
- `blog.html` — картки для post-2297 і post-2298 додані (Editorial 2296–2305 2/10 і 3/10).
- `STATUS.md` — total posts 2026 → 2028; Editorial 2296–2305 progress 1/10 → 3/10.
- `VERSION` — 8.2.1 → 8.3.0.

---

## [8.2.1] — 2026-06-22

### Added
- `docs/INCIDENT_RESPONSE.md R-40` — Recovery runbook for `ci_telemetry_daily_sync` (pg_cron job 28) staleness. Covers P2 classification, five root-cause hypotheses (H1 job deregistered, H2 peer daily jobs stale, H3 pg_cron schema lock, H4 `ci_telemetry_daily` table access, H5 GitHub Actions relay Worker down), five SQL scope queries (R-40-C1 through R-40-C5), DEC-030 HMAC-chained events (`system.ci_telemetry_stale_declared` HIGH/7yr, `system.ci_telemetry_restored` STANDARD/3yr), CI-TELEMETRY-STALE-CHAIN-01 ordering invariant, evidence artefacts CI-E-001 and CI-E-003, and SOC 2 CC8.1 mapping. Closes the sole gap in the §12.6 pg_cron job registry: jobs 26, 27, 29, 32–37 had R-XX cross-references; job 28 did not.
- `docs/AUDIT_LOG_SCHEMA.md` R-40 monitoring-control events subsection — `system.ci_telemetry_stale_declared` and `system.ci_telemetry_restored` registered with full Zod schemas and CI-TELEMETRY-STALE-CHAIN-01 ordering invariant note. Version v2.35 → v2.36.

### Changed
- `docs/INCIDENT_RESPONSE.md` — v3.3 → v3.4 (R-40 appended).
- `docs/OBSERVABILITY.md §12.6` — job 28 cross-ref column updated to include `INCIDENT_RESPONSE R-40 (job 28 stale recovery runbook — §R-40.5)`. Version v1.2 patch → v1.3 patch notation.
- `VERSION` — 8.2.0 → 8.2.1.

---

## [8.2.0] — 2026-06-22

### Added
- `content/post-2306-linear-vs-nonlinear-progression.md` — Editorial 2306–2315 series, post 1/10. Лінійна vs. нелінійна прогресія для рекреаційного атлета. Rhea et al. (2002): DUP +28% vs LP +14% у силі за 12 тижнів у проміжних атлетів. Механіка LP: чому працює для новачків, де зупиняється. DUP (щоденна варіація інтенсивності/об'єму): дані Zourdos et al. (2016). WUP (тижнева варіація): практичний варіант для атлетів з обмеженнями відновлення між сесіями. Блокова переодизація (Issurin 2008): accumulation → transmutation → realization; для просунутих (3+ роки стажу). Colquhoun et al. (2017): BP vs DUP у пауерліфтерів — схожі результати, системна структура важливіша за вибір між ними. Три маркери переходу від LP до нелінійної: зупинка 3 тижні, RPE-тренд вгору, стаж 12–18 міс. Практична рамка по тренувальному віку. FORM/Victor: зміна моделі за даними, не за календарем. clinical-safety: NOT_REQUIRED. sports-scientist review pending. Blog card added.
- `README.md` — Editorial 2306–2315 series added: «Переодизація для рекреаційного атлета» (10 posts). post-2306 → draft.

### Changed
- `blog.html` — card for post-2306 added above post-2270.
- `VERSION` — 8.1.0 → 8.2.0.

---

## [8.1.0] — 2026-06-22

### Added
- `content/post-2269-strength-and-cardio-conflict.md` — Editorial 2261–2270 series, post 9/10. Concurrent training interference effect: Hickson (1980) original finding, Wilson et al. (2012) meta-analysis (–31% hypertrophy, –18% strength). Molecular mechanism: AMPK vs mTOR conflict, Atherton et al. (2005) FASEB Journal. Three variables controlling interference magnitude: cardio volume/frequency, modality (muscle group overlap), temporal interval between sessions. Practical weekly structure for strength-priority, endurance-priority, and equal-concurrent athletes. Block periodization (Issurin 2008) as alternative to simultaneous development. HIIT caveats. clinical-safety: NOT_REQUIRED. sports-scientist review pending.
- `content/post-2270-program-selection-series-synthesis.md` — Editorial 2261–2270 series, post 10/10. Block synthesis: 10 principles distilled from all 10 posts, each with a single action item. Four operational checklists: new program selection (9 items), current program evaluation (5 items), transition/adaptation decision (4 items), concurrent training (4 items). Full series reference table. Proposed topic candidates for Editorial 2271–2280. clinical-safety: NOT_REQUIRED. sports-scientist review pending.
- `blog.html` — Cards added for post-2269 and post-2270. **Editorial 2261–2270 COMPLETE 10/10.**

### Changed
- `VERSION` — 8.0.1 → 8.1.0.

---

## [8.0.1] — 2026-06-22

### Changed
- `enterprise.html` — Contract clarity section added between pricing tiers and ROI/business-case section. Four editorial fact-cards cover the renewal mechanics sourced from `docs/COST_MODEL.md §42` and `docs/ENTERPRISE.md`: (1) 90-day renewal notice guarantee — MSA breach on FORM's side if missed; (2) Year 1 rate locked — no mid-term adjustments; (3) Multi-year Year 2+ escalation cap — CPI-U + 1%, hard ceiling 5% per year, methodology shared 90 days in advance; (4) Price floors enforced at billing-system layer — $6/seat Starter, $4.50/seat Growth, $4/seat Enterprise, no exception path can waive. New FAQ item efaq11: "What happens to pricing when we renew — will the rate increase?" — covers single-year reset vs. multi-year formula cap, BLS reference, floor logic, 10% seat-reduction threshold, MSA grounding. Fraunces editorial style; num/display/label/feature-card classes per established design system; no fabricated logos or case studies; all facts traceable to `docs/COST_MODEL.md §42.2–§42.5` and `docs/ENTERPRISE.md §Pricing`. compliance-officer: APPROVED (facts sourced from §42 and ENTERPRISE.md). privacy-floor: NOT_AFFECTED (no individual user data).
- `VERSION` — 8.0.0 → 8.0.1.

---

## [8.0.0] — 2026-06-22

### Added
- `content/post-2268-no-perfect-program.md` — Editorial 2261–2270, post 8/10. Чому «кращої програми» не існує — і як це використовувати. Hubal et al. (2005): 585 учасників, однакова 12-тижнева програма — від –2% до +59% зміни поперечника м'яза, від –32% до +250% приросту сили. Bamman et al. (2007): high/moderate/low responders розрізняються на рівні транскрипційної відповіді м'язової тканини, а не старанності. Damas et al. (2019): групові середні в РКД маскують величезний діапазон індивідуальних відповідей. Три рівні відповідності програми: (1) відповідність цілі — West et al. (2010) специфічність стимулу; (2) відповідність контексту відновлення — Kivistö & Häkkinen (2007) вплив позатренувального стресу; (3) відповідність тренувальному віку — Erskine et al. (2010). Чотири критерії «хорошої» програми: прогресивне перевантаження, адекватний об'єм (MEV 4–6 підходів/групу/тиждень, Israetel 2019), правильна частота, структура deload (Schoenfeld 2010). Адгеренс перевершує оптимізацію — Schoenfeld et al. (2016). Синдром вічного пошуку програми. Нейральна адаптація 3–4 тижні Grosprêtre et al. (2016) як аргумент проти ранньої зміни. Precommitment як стратегія — Baumeister & Tierney (2011). 5-кроковий алгоритм вибору для self-coached атлета. clinical-safety: NOT_REQUIRED. sports-scientist review pending. Blog card added.

### Changed
- `blog.html` — blog card for post-2268 added above post-2267.
- `README.md` — post-2268 → draft; Editorial 2261–2270 progress updated to 8/10.
- `STATUS.md` — total posts updated 2022 → 2023; Editorial 2261–2270 progress updated to 8/10; version header v7.81.0 → v8.0.0.
- `VERSION` — 7.99.0 → 8.0.0.

---

## [7.99.0] — 2026-06-22

### Added
- `content/post-2266-switching-between-programs.md` — Editorial 2261–2270, post 6/10. Перехід між програмами: як уникнути «вічного онбордингу». Синдром «вічного онбордингу»: атлет завжди у першому місяці нової програми, ніколи в четвертому. Чотири ситуації де перехід виправданий: програма виконана повністю (12–16 тижнів), структурна невідповідність, зміна тренувальної мети, відсутність прогресу 8+ тижнів при ретельному виконанні. Нейральна адаптація перших 3–4 тижнів як найважливіший аргумент проти ранньої зміни (Grosprêtre et al., 2016). Протокол переходу: deload-тиждень перед переходом, базова оцінка перед новою програмою, мінімальне зобов'язання 6–8 тижнів (precommitment — Baumeister & Tierney, 2011), перший тиждень нової програми як розвідка. Матриця рішень 7 сценаріїв. clinical-safety: NOT_REQUIRED. sports-scientist review pending.
- `content/post-2267-limited-time-program.md` — Editorial 2261–2270, post 7/10. Програма для обмеженого часу: що виключити без втрати ефекту. Ієрархія тренувального ефекту — 4 рівні: основний рух з прогресивним перевантаженням → допоміжні рухи → ізоляція → фінішери/кондиціонування. MEV для intermediate — 4–6 підходів на м'язову групу на тиждень (Israetel et al., 2019); Krieger (2010) — різниця 1 vs. 2–3 підходів більша ніж 3 vs. 5; Ralston et al. (2017) — перша «доза» непропорційно велика. Структура 45-хвилинного тренування; суперсерії на антагоністах як ущільнення без скорочення пауз між важкими підходами; скорочення частоти vs. об'єму (Schoenfeld, 2016); ієрархія скорочень 8 пунктів у порядку зростання впливу; тимчасове vs. постійне обмеження — різні відповіді. clinical-safety: NOT_REQUIRED. sports-scientist review pending.

### Changed
- `blog.html` — blog cards for posts 2266, 2267 added above post-2265.
- `README.md` — posts 2266, 2267 → draft; Editorial 2261–2270 progress updated to 7/10.
- `STATUS.md` — total posts updated 2020 → 2022; Editorial 2261–2270 progress updated to 7/10.
- `VERSION` — 7.98.0 → 7.99.0.

---

## [7.98.0] — 2026-06-22

### Added
- `content/post-2296-training-year-seasonal-thinking.md` — Editorial 2296–2305, post 1/10. Тренувальний рік: чому 12 місяців не рівні і як це враховувати у програмуванні. Bompa & Haff (2009): макроцикл як послідовність фаз накопичення, реалізації, переходу і відновлення. Kiely (2012): атлет — нелінійна система зі змінними зовнішніми умовами, periodizація має враховувати контекст, а не ігнорувати його. Чотири фази тренувального року для recreational athlete: накопичення (стабільний контекст, нарощування об'єму), реалізація (зниження об'єму, підвищення інтенсивності), перехід (відпустки/свята/зміна розкладу — мінімальна частота зі збереженою інтенсивністю), відновлення (деловантаж). Mujika & Padilla (2000): при зниженні об'єму до 60–70% зі збереженням інтенсивності силові характеристики не знижуються до 3–4 тижнів — підтримка можлива при значно менших витратах. Kenttä & Hassmén (1998): структуроване відновлення як обов'язкова умова суперкомпенсації. Карта сезонних перешкод: літо (червень–серпень), грудень, березень–квітень. Три типові помилки: «надолужу влітку», однакові цілі на кожен блок, рахунок «пропущених тижнів» замість «тижнів, що були». Plisk & Stone (2003): periodizація — планування стресу відповідно до стрес-абсорбційного потенціалу атлета. clinical-safety: NOT_REQUIRED. sports-scientist review pending. Blog card added.

### Changed
- `blog.html` — blog card for post-2296 added above post-2265.
- `README.md` — Editorial 2296–2305 (Тренувальний цикл: сезонне мислення для recreational athlete) added to roadmap; post-2296 → draft.
- `VERSION` — 7.97.0 → 7.98.0.

---

## [7.97.0] — 2026-06-22

### Added
- `content/post-2138-training-with-chronic-fatigue.md` — Editorial 2131–2140, post 8/10. Тренування при хронічній втомі: не мотивація, а алгоритм. Таксономія Meeusen et al. (2013): гостра втома (нормальна, 24–72 год) / функціональне перенавантаження FOR (запланована фаза, кілька днів — 2 тижні) / нефункціональне перенавантаження NFOR (ненавмисне, тижні–місяці) / синдром перетренованості OTS (клінічний). Kreher & Schwartz (2012): FOR vs NFOR розрізняються ретроспективно — в реальному часі потрібні маркери. Halson & Jeukendrup (2004): одне навантаження може бути функціональним або нефункціональним залежно від контексту відновлення. Три вимірювані маркери: RPE-дрейф (+1–1.5 за 2–3 тижні), стагнація/регрес e1RM без зовнішніх причин, суб'єктивне відновлення < 2.5/5. Plews et al. (2013): тижневий HRV-тренд як чутливий маркер. Алгоритм трьох станів: зелений → план; жовтий → тактичний деловантаж 7–10 днів (–30–40% обсягу, збережена інтенсивність, Schoenfeld et al. 2017); червоний → повний скид 14–21 день (–50–60%, RPE ≤ 6). Три типові помилки. Критерії готовності до повернення. clinical-safety: NOT_REQUIRED. sports-scientist review pending. Blog card added.
- `content/post-2139-gym-vs-home-training.md` — Editorial 2131–2140, post 9/10. Зал vs. тренування вдома: що реально порівнюється, а що — структурно різне. Calatayud et al. (2015) Journal of Human Kinetics: при однаковому RPE (7–8) і обсязі — різниці в активації м'язів і силових показниках немає. Calatayud et al. (2016): гіпертрофійна адаптація порівнянна при рівній відносній інтенсивності. Schoenfeld & Grgic (2020): первинні детермінанти гіпертрофії — механічне напруження, м'язовий збиток, метаболічний стрес; жоден не вимагає конкретного обладнання, але вимагає достатнього відносного навантаження. Де вони структурно різні: діапазон прогресії (дрібний інкремент vs. прогресія варіантом вправи), розвиток максимальної сили при ≥ 85% 1ПМ, середовищний контекст (Zajonc 1965, Michaels et al. 1982), логістика і часові витрати. Матриця рішень: ціль блоку × адгеренс × доступність. Schoenfeld et al. (2016): адгеренс — первинний предиктор, вище за метод. Гібридна стратегія: основна (зал) + backup (вдома), backup активується за об'єктивним критерієм. Mujika & Padilla (2000) для backup. Три типові помилки. clinical-safety: NOT_REQUIRED. sports-scientist review pending. Blog card added.

### Changed
- `blog.html` — blog cards for post-2138 and post-2139 added above post-2137.
- `STATUS.md` — Editorial 2131–2140 progress updated 7/10 → 9/10; next: post-2140 (синтез).
- `VERSION` — 7.96.1 → 7.97.0.

---

## [7.96.1] — 2026-06-22

### Changed
- `docs/OBSERVABILITY.md` v4.9.4 → v4.9.5 — §39.10 BC-E-001 evidence artefact description extended: R-39 (Backup Age Monitor Stale) activation addendum clause and `r39_activations: 0` zero-incident attestation note added. Closes INCIDENT_RESPONSE R-39.11 item 3 (P1/M11) for the OBSERVABILITY.md component.
- `docs/SOC2_READINESS.md` v3.25.6 → v3.25.7 — §71.7 A1.2 primary evidence artefacts column updated: BC-E-001 (quarterly job 29 run history; OBSERVABILITY.md §39.10) registered as primary A1.2 evidence artefact. Closes INCIDENT_RESPONSE R-39.11 item 3 (P1/M11) for the SOC2_READINESS.md component.
- `docs/INCIDENT_RESPONSE.md` R-39 v1.0 → v1.1 — R-39.11 item 3 marked [x] Done (2026-06-22). No procedural changes to runbook steps.
- `VERSION` — 7.96.0 → 7.96.1.

---

## [7.96.0] — 2026-06-22

### Added
- `content/post-2265-safe-program-modifications.md` — Editorial 2261–2270, post 5/10. Самостійна модифікація програми: що змінювати безпечно, що руйнує логіку. Чотири рівні архітектури програми: структура навантаження (рівень 1), інтенсивність і прогресія (рівень 2), вибір вправ (рівень 3), операційні деталі (рівень 4). Рівні 1–2 — несучі стіни: деавтоматизаційний тиждень, модель прогресії, загальний тижневий об'єм. Рівні 3–4 — оздоблення: заміна вправи всередині патерну, порядок другорядних вправ, паузи в діапазоні +/-30%. Тест «несуча стіна»: п'ять питань. Чому «безпечні» зміни накопичуються в небезпечний результат — програмний дрейф (Crane, 2020). Israetel MEV/MAV (2019), Krieger (2010), Schoenfeld et al. (2016) ×2. clinical-safety: NOT_REQUIRED. sports-scientist review pending. Blog card added.

### Changed
- `blog.html` — blog card for post-2265 added.
- `README.md` — post-2265 → draft.
- `STATUS.md` — post-2265 entry added to 11–50 block row.
- `VERSION` — 7.95.0 → 7.96.0.

## [7.95.0] — 2026-06-22

### Added
- `content/post-2137-minimum-effective-week.md` — Editorial 2131–2140, post 7/10. Мінімальний ефективний тиждень: що насправді підтримує адаптацію при стиснутому часі. Два різних мінімуми: фізіологічний (поріг початку детренінгу) і поведінковий (поріг переривання звички) — в обмежених умовах вони розходяться. Для збереження сили: Mujika & Padilla (2000) — 33–66% обсягу при збереженій інтенсивності = збереження 8–16 тижнів; 1 якісна сесія/тиждень достатньо. Для м'язової маси: Ralston et al. (2017) — ≥2 сесії/тиждень, Bickel et al. (2011) — 1/9 вихідного об'єму зберігало силу на 32 тижні (застереження: 60–75 р.). Ключове: скорочуй обсяг, зберігай інтенсивність (Schoenfeld et al. 2017 — механічне напруження є первинним сигналом; Häkkinen et al. 2001 — нейром'язова деградація починається через 1–2 тижні без навантаження). Зведена таблиця мінімумів за ціллю (сила/м'яза/нейром'язові/звичка/аеробна). Три кроки до мінімального тижня. П'ять помилок. Алгоритм. clinical-safety: NOT_REQUIRED. sports-scientist review pending. Blog card added.

### Changed
- `blog.html` — blog card for post-2137 added.
- `STATUS.md` — Editorial 2131–2140 progress updated 6/10 → 7/10; next: post-2138.
- `VERSION` — 7.94.1 → 7.95.0.

---

## [7.94.1] — 2026-06-22

### Changed
- `docs/AUDIT_LOG_SCHEMA.md` v2.34 → v2.35 — +2 DEC-030 HMAC-chained events closing INCIDENT_RESPONSE R-35.11 item 1 (P0/M10). New entries added to `### System` section and retention table: (1) `system.wl_cert_check_failure_declared` (HIGH/7yr): devops-lead (IC) declares `white_label_cert_check` (job 40, `0 2 * * *`, 26h freshness window) stale incident at T+0 of every R-35 activation; WL-CERT-CHAIN-01 anchor; gap leaves WL-SLO-02/WL-SLO-03 in detection blind spot; Zod v2 payload: `incident_id`, `confirmed_stale_since`, `stale_hours`, `missed_runs`, `at_risk_domain_count`, `earliest_expiry_at` (nullable), `trigger` enum, `initial_severity` enum; HTTP 422 `WL_CERT_CHAIN_01_VIOLATION` on ordering inversion → R-05; CC7.2/A1.2. (2) `system.wl_cert_check_restored` (STANDARD/3yr): devops-lead confirms job 40 re-enabled and first successful post-fix run confirmed; Zod v2 payload: `incident_id`, `restored_at`, `root_cause` enum H1–H5, `fix_deployed_at`, `domains_verified` (aggregate count); WL-CERT-CHAIN-01 terminal event; CC7.2/A1.2. Privacy floor (both events): no `user_id`, no `tenant_id`, no domain names, no hostname values — `at_risk_domain_count` and `domains_verified` are aggregate integers. Owner: devops-lead + compliance-officer + security-engineer.
- `docs/SOC2_READINESS.md` v3.25.5 → v3.25.6 — WL-CERT-E-001 registered in §79.4 consolidated evidence table: quarterly export of job 40 (`white_label_cert_check`) run history from `pg_cron.job_run_details`; CC7.2/A1.2; Auto; quarterly from M10; devops-lead; `compliance/evidence/wl/wl-cert-e-001-YYYY-QN.md`; 3yr retention. Closes R-35.11 item 3 (P1/M11). Privacy floor: `pg_cron.job_run_details` rows contain only `jobname`, `runid`, `started`, `ended`, `status`, `return_message` — no domain names, `tenant_id`, `user_id`, or GDPR Art. 9 special-category data. Owner: compliance-officer.
- `docs/INCIDENT_RESPONSE.md` — R-35.11 items 1 and 3 marked done: item 1 `[x] Done — 2026-06-22, AUDIT_LOG_SCHEMA.md v2.35`; item 3 `[x] Done — 2026-06-22, SOC2_READINESS.md v3.25.6`. Owner: devops-lead + compliance-officer.
- `VERSION` — 7.94.0 → 7.94.1.

---

## [7.94.0] — 2026-06-22

### Added
- `content/post-2264-when-to-adapt-vs-follow-program.md` — Editorial 2261–2270 block, post 4/10. Коли адаптувати програму, а коли дотримуватись оригіналу. Три класи причин для модифікації: клас А (структурні невідповідності — обладнання, частота, тренувальний вік), клас Б (тактичні форс-мажори — накопичена втома, разова аномалія, технічний обмежувач), клас В (уникання дискомфорту — найпоширеніший, найрідше визнаний). Тест трьох питань перед модифікацією. Механізм акумуляції «помірних змін» у руйнівну суму. Розмежування авторегуляції всередині програми vs. модифікації програми: Helms et al. (2014) — RPE як інструмент авторегуляції у заданих рамках vs. підстава для довільних змін. Правило 30% параметрів — межа між адаптацією і новою програмою. Три ознаки, коли правильна відповідь — замінити програму повністю. Peterson et al. (2011) — adherence як найсильніший предиктор результату. Schoenfeld (2010) — структурні адаптації 6–12 тижнів. Kraemer & Ratamess (2004). Чотири кроки перед системною модифікацією. clinical-safety: NOT_REQUIRED. sports-scientist review required before publish. Blog card added.

### Changed
- `blog.html` — card added for post-2264 (prepended before post-2263).
- `STATUS.md` — content table updated: post-2264 added, total 2017 → 2018; Editorial 2261–2270 progress 3/10 → 4/10; next priorities updated.
- `README.md` — post-2264 → draft in table; Editorial 2286–2295 proposed series added.
- `VERSION` — 7.93.0 → 7.94.0.

---

## [7.93.0] — 2026-06-22

### Added
- `content/post-2262-three-types-of-programs.md` — Editorial 2261–2270 block, post 2/10. Три типи програм: лінійна прогресія, блокова, хвильова — і коли кожна підходить. LP: нейральна адаптація за 48–72h SRA-цикл, Rippetoe logic, дві умови застосування (справжній початківець, повернення після перерви), сигнал провалу LP. Блокова: Issurin (2010) концентроване vs. розподілене навантаження, суперімпозиція ефектів, Stone et al. (1999), три фази (accumulation → transmutation → realization), три помилки. DUP/хвильова: Verkhoshansky accommodation principle, Rhea et al. (2002) — DUP подвоїв приріст сили LP при рівному обсязі за 12 тижнів у тренованих суб'єктів, різниця DUP vs. wave loading. Таблиця рішень 3×4 (тренувальний вік / ціль / днів на тиждень / гнучкість). Три типові помилки вибору. clinical-safety: NOT_REQUIRED. sports-scientist review required before publish. Blog card added.
- `content/post-2263-how-to-read-a-program.md` — Editorial 2261–2270 block, post 3/10. Як читати готову програму: що між рядками важливіше за цифри. Програма — стиснутий аргумент про адаптацію, а не інструкція. §Обсяг: 3×5 vs. 5×3 механістично, working vs. all sets, MEV/MAV/MRV Israetel et al. (2019). §Інтенсивність: gap між тестованим і розрахованим 1RM, RPE калібрація у початківців і просунутих (Zourdos 2016, Helms 2016), RIR vs. RPE дивергенція, LP «add 2.5 kg» пастка в проміжному контексті. §Вибір вправ: ціль адаптації vs. заповнення слабкого місця, правила заміни, чому paused squat в Calgary Barbell не довільний. §Паузи: Schoenfeld et al. (2016) — різниця закривається при рівному обсязі, але падіння якості підходів залишається. §Червоні прапорці: 6 структурних дефектів. §П'ять питань перед стартом програми. clinical-safety: NOT_REQUIRED. sports-scientist review required before publish. Blog card added.

### Changed
- `blog.html` — cards added for post-2263 and post-2262 (prepended before post-2261).
- `STATUS.md` — content table updated: posts 2262–2263 added, total 2015 → 2017; Editorial 2261–2270 progress 1/10 → 3/10; next priorities updated.
- `VERSION` — 7.92.1 → 7.93.0.

---

## [7.92.1] — 2026-06-22

### Changed
- `docs/AUDIT_LOG_SCHEMA.md` v2.33 → v2.34 — +7 DEC-030 HMAC-chained events closing SOC2_READINESS.md PT-P1-02 (M6), PT-P1-06 (M6), and 62.8-01 (M5). New section `### Penetration Testing events (DEC-030 HMAC-chained · SOC2_READINESS §61 + §62 · CC4.1/CC7.1/CC7.2/CC6.1/CC6.6/CC9.2/A1.2)` inserted before `## Export & delivery`. (1) `admin.pentest_initiated` (HIGH/7yr): engagement anchor — dual sign-off literals (`authorised_by_compliance_officer: true` + `authorised_by_founder: true`), `staging_only_flag`, `supabase_notification_filed` + `supabase_notification_date` PT-E-007 chain-of-custody fields (§62.2 OQ-PT-02 resolution); PENTEST-CHAIN-02 anchor. (2) `admin.pentest_finding_logged` (HIGH/7yr): per-finding immutable record; `severity_tier` P0–P3 enum; `affected_surface` 12-value enum; `cwe_id` CWE-NNN regex; `cvss_score` 0–10 float; `sla_deadline` computed per §61.5 SLA table; PENTEST-CHAIN-01 anchor. (3) `admin.pentest_finding_remediated` (HIGH/7yr): remediation closure; `remediation_pr` URL; PENTEST-CHAIN-01 successor (HTTP 422 `PENTEST_CHAIN_01_VIOLATION` on inversion → R-05). (4) `admin.pentest_report_filed` (HIGH/7yr): `report_sha256` hex-64 tamper-evident PT-E-001 hash; `finding_count_p0/p1/p2/p3` aggregate; PENTEST-CHAIN-02 terminal (HTTP 422 `PENTEST_CHAIN_02_VIOLATION` on inversion → R-05). (5) `admin.pentest_retest_completed` (MEDIUM/7yr): `retest_method` enum (vendor / self — self only for P2/P3); PENTEST-CHAIN-01 successor. (6) `admin.pentest_risk_accepted` (HIGH/7yr): `approved_by_compliance_officer: true` + `approved_by_founder: true` dual literals; `decision_log_ref` required; Worker HTTP 422 `PENTEST_RISK_ACCEPTANCE_P0_P1_VIOLATION` if `severity_tier ∈ {P0,P1}`. (7) `admin.pentest_chain_enumeration_authorised` (HIGH/7yr) — NEW from §62.4.3 OQ-PT-04 resolution: Layer B production chain enumeration gate; `enumeration_scope: z.literal('audit_log_events SELECT')` hard invariant; `production_flag: z.literal(true)` + `staging_only_destructive_tests_confirmed: z.literal(true)` dual invariants; `credential_expiry` 48h max; must precede any vendor Layer B query (enforced by `emit-audit-event` Worker); CC6.1/CC9.2. Retention table: +7 rows. PENTEST-CHAIN-01/02/03 invariant specs. PENTEST_RISK_ACCEPTANCE_P0_P1_VIOLATION severity gate. Full Zod v2 schemas. SOC 2 auditor narratives CC4.1/CC7.1/CC7.2/CC6.1/CC6.6/CC9.2/A1.2. PT-E-001 through PT-E-007 evidence artefact register. Aggregate PT-E-006 SQL (privacy-safe). Owner: security-engineer + compliance-officer.
- `docs/SOC2_READINESS.md` — PT-P1-02 marked `[x] Done — 2026-06-22, AUDIT_LOG_SCHEMA.md v2.34`; PT-P1-06 marked `[x] Done — 2026-06-22, AUDIT_LOG_SCHEMA.md v2.34`; 62.8-01 marked `[x] Done — 2026-06-22, AUDIT_LOG_SCHEMA.md v2.34`. Three compliance checklist items closed: PT-P1-02 (register all §61.8 DEC-030 event types, M6), PT-P1-06 (register `admin.pentest_chain_enumeration_authorised`, M6), 62.8-01 (register `admin.pentest_chain_enumeration_authorised` with full spec, M5 deadline — overdue resolved). Owner: security-engineer + compliance-officer.
- `VERSION` — 7.92.0 → 7.92.1.

---

## [7.92.0] — 2026-06-22

### Added
- `content/post-2261-program-selection-framework.md` — Editorial 2261–2270 «Від чужої програми до власного протоколу», post 1/10. Як вибрати програму серед сотень варіантів: рамка оцінки без аналізу паралічу. Два жорстких фільтри: адгеренс (Peterson et al. 2011 — найсильніший предиктор результату) і вбудований механізм прогресії. П'ять координат оцінки: тренувальний вік (новачок / проміжний / просунутий і логіка адаптаційного статусу), ціль блоку (одна, не дві), структура і частота (відповідність реальному розкладу), якість документації (чотири явні питання), сумісність із відновленням (сон, зовнішній стрес, Stults-Kolehmainen & Sinha 2014). Алгоритм 6 кроків. Три типові помилки: бренд замість параметрів, зміна програми до мінімального горизонту (Ralston et al. 2017: 6–8 тижнів), адаптація без розуміння логіки. Paradox of choice (Iyengar & Lepper 2000). Schoenfeld & Grgic (2019) мета-аналіз частоти. clinical-safety: NOT_REQUIRED. sports-scientist review required before publish. Blog card added.

### Changed
- `blog.html` — card added for post-2261 (position before post-2260).
- `README.md` — post-2261 updated proposed → draft · v7.92.0; Editorial 2281–2285 «Самопрограмування: від реакції до системи» added as proposed (5 теми).
- `STATUS.md` — total posts 2014 → 2015; Editorial 2261–2270 progress started 1/10.
- `VERSION` — 7.91.0 → 7.92.0.

---

## [7.91.0] — 2026-06-22

### Added
- `content/post-2259-planned-vs-autoregulated-deload.md` — Editorial 2251–2260, post 9/10. Плановий або авторегуляторний deload: як вирішити, який тип потрібен саме тобі. Два підходи з чіткими визначеннями і трейдофами. Confirmation bias як центральна проблема self-coached авторегуляції. П'ять конкретних вимірюваних тригерів: RPE-дрейф (+0.5–1 пункт × 3 сесії), e1RM-тренд (знижується 2 тижні), якість сну (3+ ночі нижче норми), ранкова ЧСС (+5 уд/хв × 3 дні), суб'єктивна готовність (≤5/10 × 3 дні). Правило «2 з 4» для активації деловантажу. Гібридний conditional extension protocol. Критерії вибору підходу. П'ять типових помилок. clinical-safety: NOT_REQUIRED. sports-scientist review required before publish. Blog card added.
- `content/post-2260-mesocycle-synthesis-checklist.md` — Editorial 2251–2260, post 10/10. Синтез серії: перший самостійний блок. Одно-речне резюме кожного з постів 2251–2259. Довідкова таблиця параметрів: накопичувальний vs. інтенсифікаційний блок (ціль / тривалість / інтенсивність / обсяг / частота / деловантаж). 19-пунктовий операційний чекліст трьох фаз (планування: 9 пунктів; виконання: 5 пунктів; ретроспектива: 5 пунктів). Дев'ять типових помилок першого самостійного блоку. Victor integration point. Анонс Editorial 2261–2270 «Від чужої програми до власного протоколу». clinical-safety: NOT_REQUIRED. sports-scientist review required before publish. Blog cards added.

### Changed
- `blog.html` — cards added for post-2259 and post-2260 (positions before post-2258).
- `README.md` — posts 2259 and 2260 updated from proposed → draft · v7.91.0.
- `STATUS.md` — total posts 2012 → 2014; Editorial 2251–2260 progress 8/10 → COMPLETE 10/10.
- `VERSION` — 7.90.0 → 7.91.0.

---

## [7.90.0] — 2026-06-22

### Added
- `content/post-2258-weekly-undulation.md` — Editorial 2251–2260, post 8/10. Тижнева хвиля всередині блоку: чому однакові тижні дають гірший результат, ніж варіативні. Акомодаційний принцип Верхошанського (1988): передбачуваний паттерн знижує адаптаційний сигнал. Два типи ундуляції: лінійна хвиля (volume wave: MEV → +10% → +10% → deload) і нелінійна/справжня ундуляція (DUP). Rhea et al. (2002): DUP перевищила LP після 12 тижнів у тренованих суб'єктів. Три практичні моделі: хвиля обсягу (найпростіша, рекомендована для першого блоку), хвиля інтенсивності (RPE), комбінована хвиля (з застереженням про складність). Чітке розмежування: незапланована варіація = шум, запланована = сигнал. П'ять типових помилок. clinical-safety: NOT_REQUIRED. sports-scientist review required before publish. Blog card added.

### Changed
- `blog.html` — card added for post-2258 (position before post-2257).
- `README.md` — post-2258 updated from proposed → draft · v7.90.0.
- `STATUS.md` — total posts 2011 → 2012; Editorial 2251–2260 progress 7/10 → 8/10.
- `VERSION` — 7.89.1 → 7.90.0.

---

## [7.89.1] — 2026-06-22

### Added
- `docs/INCIDENT_RESPONSE.md` — R-39 Backup Age Monitor Stale (`backup_age_monitor`, job 29): thirty-ninth runbook. Full IC protocol for when job 29 exceeds its 5-hour freshness window, covering the BC-SLO-01/BC-SLO-02 monitoring blind spot. Critical P0/P1/P2 classification gate (R-39-C1 backup freshness scope query determines whether a stale monitor coincides with an actual backup staleness event). Four root cause hypotheses (H1 deleted/disabled, H2 shared pg_cron, H3 query timeout, H4 platform outage). Two DEC-030 HMAC-chained events: `system.backup_monitor_stale_declared` (HIGH/7yr) and `system.backup_monitor_restored` (STANDARD/3yr); BAM-STALE-CHAIN-01 ordering invariant. Internal communication template BAM-INT-01 (P0 only). Evidence path `compliance/evidence/backup/bam-stale/r39-<incident_id>/`; BC-E-001 quarterly addendum format. SOC 2 criteria A1.2 + CC7.2. Owner: devops-lead + compliance-officer.

### Changed
- `docs/AUDIT_LOG_SCHEMA.md` v2.32 → v2.33 — two new R-39 events registered in §Backup & DR Observability events: `system.backup_monitor_stale_declared` (HIGH, 7yr) and `system.backup_monitor_restored` (STANDARD, 3yr); BAM-STALE-CHAIN-01 ordering invariant (HTTP 422 `BAM_STALE_CHAIN_01_VIOLATION`); Zod schemas for both events. Retention table +2 rows (HIGH 7yr A1.2, STANDARD 3yr A1.2).
- `docs/OBSERVABILITY.md` v4.9.3 → v4.9.4 — §12.6 job 29 `backup_age_monitor` cross-ref column extended to include `INCIDENT_RESPONSE R-39 (job 29 stale recovery runbook — §R-39.5)`. Closes the sole §12.6 entry without an INCIDENT_RESPONSE stale runbook cross-reference among jobs with defined stale consequences. v1.2 patch note added.
- `VERSION` — 7.89.0 → 7.89.1.

---

## [7.89.0] — 2026-06-22

### Added
- `content/post-2257-training-frequency-stimulus-recovery.md` — Editorial 2251–2260, post 7/10. Частота тренувань: чому це питання про розподіл стимулу, а не про кількість сесій. SRA-крива (Stimulus→Recovery→Adaptation) як теоретична основа. Schoenfeld et al. 2016 мета-аналіз: 2x краще 1x при рівному об'ємі; 3x не дає значущої переваги над 2x (Colquhoun et al. 2018). Взаємодія частоти і обсягу: частота — механізм розподілу, обсяг — первинна змінна. Різні SRA-вікна для різних м'язових груп і типів рухів. Технічне засвоєння як додатковий аргумент вищої частоти для компаунд-рухів. Практичний алгоритм 5 кроків для вибору частоти в мезоциклі. 5 типових помилок. clinical-safety: NOT_REQUIRED. sports-scientist review required before publish. Blog card added.

### Changed
- `blog.html` — card added for post-2257 (position before post-2256).
- `README.md` — posts 2255 and 2256 updated from proposed → draft; post-2257 added as draft · v7.89.0; Editorial 2271–2280 «Моніторинг без тренера: мінімальний стек даних» added as new proposed block.
- `STATUS.md` — total posts 2010 → 2011; Editorial 2251–2260 progress 6/10 → 7/10.
- `VERSION` — 7.88.0 → 7.89.0.

---

## [7.88.0] — 2026-06-22

### Added
- `content/post-2255-intensity-rpe-progression-within-block.md` — Editorial 2251–2260, post 5/10. Інтенсивність у блоці: %1ПМ і RPE як взаємодоповнювальні інструменти прогресії. Лінійна прогресія (таблиця по тижнях для накопичувального блоку). Wave loading — механізм хвилі, коли підходить, коли ні. Авторегуляція трьох рівнів: мікро (підхід), сесія, тиждень. Три симптоми некерованої прогресії інтенсивності. Практичний алгоритм: стартова інтенсивність і крок для накопичення vs. інтенсифікації. clinical-safety: NOT_REQUIRED. sports-scientist review required before publish. Blog card added.
- `content/post-2256-deload-when-and-how.md` — Editorial 2251–2260, post 6/10. Deload: фізіологічне обґрунтування (нейром'язова втома, структурна мікротравматизація, психологічне виснаження, суперкомпенсація). Два типи: scheduled vs. autoregulated (симптоми-тригери). Три моделі: volume deload / intensity deload / combined deload з конкретними таблицями параметрів. Що не є deload (повна пауза, хаотичне «легеньке», тиждень нових вправ). Тривалість і частота за типом блоку. Маркери успішного deload. clinical-safety: NOT_REQUIRED. sports-scientist review required before publish. Blog card added.

### Changed
- `blog.html` — cards added for post-2255 and post-2256 (positions before post-2254).
- `STATUS.md` — total posts 2008 → 2010; Editorial 2251–2260 progress 4/10 → 6/10.
- `VERSION` — 7.87.1 → 7.88.0.

---

## [7.87.1] — 2026-06-22

### Added
- `docs/INCIDENT_RESPONSE.md` — R-38: Google Directory Alert Check Stale (`google_directory_alert_check`, job 35). Thirty-eighth runbook. Closes the sole remaining gap in 5-min cadence pg_cron stale runbook coverage: job 35 had no INCIDENT_RESPONSE runbook despite being explicitly cross-referenced in R-37 (job 38 peer). R-38 covers the AL-SSO-GDIR-01 / AL-SSO-GDIR-02 detection blind-spot protocol with two-severity matrix (P1/P0), R-38-C1 GDIR-01 scope query, R-38-C2 pg_cron run history, four root cause hypotheses (H1–H4), four recovery steps, GDIR-INT-01 internal communication template (P0 only), DEC-030 GDIR-ALERT-CHAIN-01 chain, evidence preservation via SSO-OBS-E-007 addendum, SOC 2 CC7.2/CC7.3 mapping, and five post-incident controls.

### Changed
- `docs/AUDIT_LOG_SCHEMA.md` — v2.31 → v2.32: registered `system.gdir_alert_stale_declared` (HIGH, 7yr) and `system.gdir_alert_restored` (STANDARD, 3yr) in `§System` section with full Zod v2 schemas and GDIR-ALERT-CHAIN-01 ordering invariant (HTTP 422 `GDIR_ALERT_CHAIN_01_VIOLATION` → R-05). Closes R-38.11 item 1 (P0/M4).
- `docs/OBSERVABILITY.md` — §12.6 v1.1 patch: added `INCIDENT_RESPONSE R-38 (job 35 stale recovery runbook — §R-38.5)` to the `google_directory_alert_check` stale-consequence cross-ref column. Closes R-38.10 `§12.6 cross-reference` post-incident control and R-38.11 item 4.
- `docs/SOC2_READINESS.md` — v3.25.4 → v3.25.5: §92.2 Part D (R-38 activation cross-check SQL + addendum format); §92.3 `r38_activations: 0` zero-event field; §92.4 CC7.3 updated with R-38 response protocol; §92.5 R-38.11 item 3 closure row; §92.6 item 2 status updated to `[x] Done`. Closes R-38.11 item 3 (P1/M11).
- `VERSION` — 7.87.0 → 7.87.1.

---

## [7.87.0] — 2026-06-22

### Added
- `content/post-2254-volume-progression-within-block.md` — Editorial 2251–2260, post 4/10. Об'єм у блоці: MEV/MAV/MRV як три маркери дозування, а не цілі. Прогресія об'єму тиждень за тижнем (MEV → MAV → деловантаження). Чому об'єм і вага — різні механізми прогресії. Індивідуальна варіабельність: тренувальний вік, якість сну, зовнішній стрес. Операційні маркери перетину MRV. 4-крокий алгоритм для конкретного блоку. Krieger (2010), Schoenfeld, Ogborn & Krieger (2016), Hubal et al. (2005), Leproult & Van Cauter (2011), Halson (2014). clinical-safety: NOT_REQUIRED. sports-scientist review required before publish.
- `blog.html` — Blog card added for post-2254.
- `README.md` — Editorial 2261–2270 «Від чужої програми до власного протоколу» added as proposed (10 topics: program selection, linear/block/wave types, reading programs, adapting vs. following, safe modification, transitions, time-constrained programming, "no best program", aerobic/strength interference, synthesis).

### Changed
- `README.md` — rows 2252 і 2253 updated from `proposed` to `draft · v7.86.0` with filenames.
- `STATUS.md` — total posts 2007 → 2008; Editorial 2251–2260 progress 3/10 → 4/10.
- `VERSION` — 7.86.0 → 7.87.0.

---

## [7.86.0] — 2026-06-22

### Added
- `content/post-2252-block-goal-setting.md` — Editorial 2251–2260, post 2/10. Операційна ціль тренувального блоку: чим відрізняється від напряму (ціль-декорація vs. ціль-інструмент). Два рівні цілі: адаптаційна (один пріоритет на блок) і структурна (вимірювана метрика). Ціль-результат vs. ціль-процес. Три обмеження: базовий рівень, тривалість блоку, зовнішній контекст. Перевірочний список з 4 запитань. clinical-safety: NOT_REQUIRED. sports-scientist review required before publish.
- `content/post-2253-accumulation-vs-intensification.md` — Editorial 2251–2260, post 3/10. Два фундаментальних адаптаційних режими силового тренінгу: накопичення (60–80% 1ПМ, 6–15 повт., RIR 2–4, структурна база) і інтенсифікація (80–95% 1ПМ, 1–5 повт., RIR 0–2, нейром'язова конверсія). Чому застрягання в одному режимі блокує прогрес. Логіка чергування і 24-тижневий практичний приклад. Один блок — один тип. clinical-safety: NOT_REQUIRED. sports-scientist review required before publish.
- `blog.html` — Blog cards added for post-2252 і post-2253.

### Changed
- `STATUS.md` — total posts 2005 → 2007; Editorial 2251–2260 progress 1/10 → 3/10.
- `VERSION` — 7.85.1 → 7.86.0.

---

## [7.85.1] — 2026-06-22

### Changed
- `docs/SOC2_READINESS.md` — v3.25.3 → v3.25.4: §79.4 WH-ESCALATION-E-001 row added (CC7/CC9, CC7.2/CC9.2, quarterly `pg_cron.job_run_details` export for job 41, Auto, quarterly from M10, platform-engineer, `webhooks/`, 3yr). §80.3 `webhooks/` R2 subfolder added with `form-api NO ACCESS`. §80.4 Vanta mirror list updated. Closes INCIDENT_RESPONSE R-36.11 item 3 (P1/M11).
- `docs/INCIDENT_RESPONSE.md` — v1.1: R-36.11 checklist item 3 marked [x] Done (2026-06-22, SOC2_READINESS.md v3.25.4). v1.1 version note added.

---

## [7.85.0] — 2026-06-22

### Added
- `content/post-2251-what-is-a-mesocycle.md` — Editorial 2251–2260 post 1/10. Opens new block «Структура тренувального блоку: від наміру до мезоциклу». Central argument: weekly thinking gives tactics without strategy; the mesocycle is the minimum planning unit that lets a self-coached athlete control adaptation rather than just accumulate fatigue. Covers: why post-beginner athletes fail with week-to-week linear thinking (three failure modes: no culmination, no context for decisions, incorrect progress assessment); mesocycle as a block with a single dominant adaptive focus (accumulation / intensification / peaking / deload); duration calibration (Zourdos et al. 2016; Helms et al. 2018: 3–6 weeks by training status); block as unit of analysis for self-coached athlete without external observer; minimal two-phase structure (4-week accumulation → 1-week deload); first-block mistake: multiple equal priorities = no priority; post-block analysis as the input for the next block. clinical-safety: NOT_REQUIRED. sports-scientist review pending. Blog card added.

### Changed
- `blog.html` — post-2251 card added above post-2250.
- `README.md` — Editorial 2251–2260 block added to roadmap with all 10 proposed topics.
- `STATUS.md` — total posts 2004 → 2005; Editorial 2251–2260 started 1/10.
- `VERSION` — 7.84.0 → 7.85.0.

---

## [7.84.0] — 2026-06-22

### Added
- `content/post-2248-constraining-conditions-technical-learning.md` — Editorial 2241–2250 post 8/10. Contextual interference effect (Shea & Morgan, 1979; Magill & Hall, 1990): random practice yields lower current performance but superior retention. Mechanisms: elaborative processing (Shea & Zimny, 1983) and action plan reconstruction (Lee & Magill, 1983). Variability of Practice Hypothesis (Schmidt, 1975): variability builds a generalizable motor schema. Differential Learning (Schöllhorn et al., 2006). Four practical methods for strength training: weight variability within technical phase, tempo variability, movement variation for sub-components, reduced video-feedback frequency (Schmidt et al., 1989; Winstein & Schmidt, 1990). Clear limits: variability is contraindicated in early learning, sub-threshold only, never when relearning is unstable. clinical-safety: NOT_REQUIRED. sports-scientist review pending. Blog card added.
- `content/post-2249-mental-representation-movement-mastery.md` — Editorial 2241–2250 post 9/10. Ericsson's expert mental representations concept from «Peak»: two components (reference model + deviation-detection system). Limits of proprioception: relative vs absolute accuracy, signal competition under load, normalization (Proske & Gandevia, 2012). External vs internal attentional focus: CONSTRAINED ACTION HYPOTHESIS (Wulf, 2013; Wulf & Lewthwaite, 2016 meta-analysis) — external focus favours acquisition; Schoenfeld & Contreras (2016) caveat for isolation/activation in trained athletes. Forward model and backward model (Wolpert, Jordan, Miall, 1995): why experts «feel» deviations before a rep ends. Stepwise calibration cycle for self-coached athletes. Monthly protocol: weeks 1–2 active calibration, weeks 3–4 model testing without video. clinical-safety: NOT_REQUIRED. sports-scientist review pending. Blog card added.
- `content/post-2250-technical-learning-series-synthesis.md` — Editorial 2241–2250 post 10/10, block synthesis. Three-level map of the series: nature of learning (2241–2244), feedback instruments and threshold (2245–2246), practice structure and mental model (2247–2249). Central thesis: skill acquisition = accurate mental model built through structured practice with external feedback, within the technical threshold, at the correct stage of the neuromuscular learning cycle. Minimum-week and full-week protocol templates. Monthly structure (calibration → structured practice → model testing → deload/assessment). Skill-stage table: new movement / stabilisation / development / relearning / automatism × correct tool. Three systematically ignored principles. Open questions for subsequent series: transfer, extreme-fatigue learning, variability individual response, FORM product integration. Fitts & Posner three-stage model as closing framework. clinical-safety: NOT_REQUIRED. sports-scientist review pending. Blog card added.

### Changed
- `blog.html` — three new cards added above post-2247: posts 2248, 2249, 2250.
- `STATUS.md` — Editorial 2241–2250 COMPLETE 10/10; total posts 2001 → 2004; next priorities updated.

---

## [7.83.1] — 2026-06-22

### Added
- `docs/AUDIT_LOG_SCHEMA.md` v2.31 — +2 DEC-030 HMAC-chained events closing OBSERVABILITY §49.9 item 3 (P0/M5) and SOC2_READINESS §95.6 item 3 (P0/M5): (1) `siem.sso_fleet_health_breach` (HIGH, 3yr) — emitted by pg_cron job 38 (`sso_fleet_health_check`) when ≥ 3 enterprise tenants simultaneously breach SSO-SLO-01; new `### SSO Fleet Health Monitoring events` section; Zod v2 schema `SiemSsoFleetHealthBreachPayload` with `sla_credit_impact: z.literal('none')` DEC-070 hard invariant; SOC 2 CC7.2/CC7.3 auditor narratives; SSO-FLEET-E-001 Part B evidence mapping. (2) `system.fleet_sso_check_stale` (LOW, 1yr) — `pg-cron-health-monitor` advisory when job 38 exceeds 6-minute freshness window; `### System` section; Zod v2 payload with `job_name: z.literal('sso_fleet_health_check')`, `stale_since`, `stale_minutes`, `missed_runs`; consumed by SSO-FLEET-E-001 Part A Step 2 freshness gap cross-check (§95.2). Retention table +2 rows. Privacy floor: no `tenant_id` (group-by internal only), no user identity, no GDPR Art. 9 data; `failing_tenant_count` and `fleet_success_rate_pct` are aggregate-only fields.

### Changed
- `docs/OBSERVABILITY.md` — §49.9 item 3 status: `[ ]` → `[x] Done — 2026-06-22, AUDIT_LOG_SCHEMA.md v2.31`.
- `docs/SOC2_READINESS.md` — §95.6 item 3 status: `[ ]` → `[x] Done — 2026-06-22, AUDIT_LOG_SCHEMA.md v2.31`.
- `VERSION` — 7.83.0 → 7.83.1.

---

## [7.83.0] — 2026-06-22

### Added
- `content/post-2247-deliberate-practice-strength-training.md` — Editorial 2241–2250, post 7/10. Навмисна практика у силовому тренінгу: принципи Ericsson адаптовані для залу. П'ять умов deliberate practice і чому звичайне тренування не відповідає жодній. Фізична втома як принципове обмеження (на відміну від шахів і музики): технічна фаза має відбуватись на початку сесії, до накопичення втоми, що знижує технічний поріг. Зона навмисної практики — між автоматичним виконанням і технічним порогом. Декомпозиція руху: один параметр на сесію. Зворотний зв'язок між підходами (відео, паузовані повторення, RPE-якісний журнал). Структура технічної фази: 20–25 хвилин, 4–6 підходів, петля ціль-виконання-корекція-верифікація. Часові реалії і що рахується як навмисна практика. Типові помилки впровадження. Ericsson, Krampe & Tesch-Römer (1993); Ericsson & Pool (2016); Schmidt & Lee (2011). clinical-safety: NOT_REQUIRED. sports-scientist review pending. Blog card added.

### Changed
- `blog.html` — blog cards added for posts 2245 (відеоаналіз), 2246 (технічний поріг), 2247 (deliberate practice); inserted before post-2244 card. Cards for 2245–2246 were missing from v7.82.0.
- `README.md` — post-2247: proposed → draft · v7.83.0; filename added.
- `STATUS.md` — total posts 2000 → 2001; Editorial 2241–2250 in progress 7/10; blog.html cards note updated.
- `VERSION` — 7.82.0 → 7.83.0.

---

## [7.82.0] — 2026-06-22

### Added
- `content/post-2245-video-analysis-self-coached-protocol.md` — Editorial 2241–2250, post 5/10. Відеоаналіз для self-coached атлета: мінімальний протокол без зайвого. Що відео показує і чого не показує: положення суглобів, траєкторія, симетрія vs. внутрішній розподіл навантаження, ЕМГ, глибинні стабілізатори. Мінімальна технічна база: 60+ fps, трипод, висота камери. Кути зйомки для основних рухів: присідання (бокова — рівень тазостегнового суглоба в нижній точці, фронтальна — рівень коліна), станова тяга (бокова — рівень стегна, фронтальна — рівень пояса), жим лежачи (бокова — рівень грифу, фронтальна зверху), жим стоячи (бокова — рівень плечей). Що аналізувати по кожному руху: нахил тулубу, knee tracking, траєкторія грифу, вальгус/варус, нейтральність хребта, sticking point. Протокол перегляду: поточковий аналіз ключових позицій (стоп-кадр), не реальний час; порівняння з базовим записом; запис конкретних спостережень а не естетичних вражень. Частота: раз на 2–3 тижні регулярно + за тригером (RPE-дрейф / «щось не так 3 сесії поспіль» / після перерви / при новій вазі). Типові пастки: аналіз між підходами, порівняння з чужою технікою, фокус на естетику замість механіки, надлишок даних. Коли відео не додає цінності. Мінімальний протокол 7 пунктів. clinical-safety: NOT_REQUIRED. sports-scientist review pending.
- `content/post-2246-technical-degradation-under-load-threshold.md` — Editorial 2241–2250, post 6/10. Технічна деградація під навантаженням: де починається критичний поріг. Два типи технічної зміни під навантаженням: варіативна зміна (нормальна адаптивна варіативність, Stergiou & Decker 2011) vs. якісна зміна (порогова деградація). Механіка переходу: компенсаторний рекрутинг, синергетичні замісники, зміна кінематики для зменшення пікового моменту. Параметри порогу: ступінь засвоєння (Kulpa et al. 2018 — досвідчені атлети стабільні до 85–90% 1RM), специфіка руху, втомний стан (Gonzalez-Izal et al. 2012 — прогресуюча деградація з 6–7-го сету). Три методи визначення особистого порогу: пряме відеоспостереження при 70/80/85/90%, RPE-кінематична кореляція (паралельний запис 4–6 тижнів), паузовані повторення як стрес-тест. Що тренується вище порогу: нервова система кодує паттерн, що реально виконується, а не той, що планується (Schmidt & Lee 2011). Програмування відносно порогу: субпорогова (технічне засвоєння) vs. надпорогова (максимальна сила) робота — обидві мають місце, але вирішують різні задачі. Типові помилки: ігнорування позиції повторення всередині підходу, статичне розуміння порогу (втома знижує поріг), однаковий поріг для всіх вправ, внутрішньоблоковий дрейф. Переучування і поріг: новий паттерн має нижчий поріг, тому переучування вимагає значного зниження ваги (50–60% 1RM). clinical-safety: NOT_REQUIRED. sports-scientist review pending.

### Changed
- `README.md` — posts 2245–2246 updated: proposed → draft · v7.82.0; filenames added.
- `STATUS.md` — total posts 1998 → 2000; Editorial 2241–2250 in progress 4/10 → 6/10; next priorities updated.
- `VERSION` — 7.81.1 → 7.82.0.

---

## [7.81.1] — 2026-06-22

### Changed
- `docs/AUDIT_LOG_SCHEMA.md` — v2.29 → v2.30. Registered 4 new DEC-030 HMAC-chained system events for R-36 (Webhook Escalation Sweep Stale, job 41) and R-37 (SSO Fleet Health Check Stale, job 38) incident chains. Inserted in §System after `system.turh_purge_restored`: `system.webhook_escalation_stale_declared` (HIGH, 7yr, WH-ESCALATION-CHAIN-01 anchor), `system.webhook_escalation_restored` (STANDARD, 3yr, WH-ESCALATION-CHAIN-01 terminal), `system.sso_fleet_check_failure_declared` (HIGH, 7yr, SSO-FLEET-CHAIN-01 anchor, includes `peer_jobs_stale` boolean), `system.sso_fleet_check_restored` (STANDARD, 3yr, SSO-FLEET-CHAIN-01 terminal). Retention table +6 rows (includes 2 rows missed from v2.29: `system.turh_purge_failure_declared` HIGH 7yr and `system.turh_purge_restored` STANDARD 3yr). Both chain ordering invariants registered: `emit-audit-event` Worker returns HTTP 422 on inversion (WH-ESCALATION-CHAIN-01 / SSO-FLEET-CHAIN-01) → R-05.
- `docs/INCIDENT_RESPONSE.md` — Marked R-36.11 item 1 [x] Done (P0/M10 — AUDIT_LOG_SCHEMA.md v2.30, 2026-06-22). Marked R-37.11 item 1 [x] Done (P0/M5 — AUDIT_LOG_SCHEMA.md v2.30, 2026-06-22).
- `VERSION` — 7.81.0 → 7.81.1.

---

## [7.81.0] — 2026-06-22

### Added
- `content/post-2244-feedback-without-coach-technical-degradation.md` — Editorial 2241–2250, post 4/10. Зворотній зв'язок без тренера: як атлет самостійно виявляє технічну деградацію. Два механізми невидимості деградації зсередини: нормалізація відхилення (adaptive motor control, Schmidt & Lee 2011) і конкуренція за увагу під навантаженням (dual-task interference, Logan 1988). Зовнішні сигнали: відео (порівняльний аналіз з базовим записом, ключові точки, два кути), RPE-дрейф при сталій вазі, e1RM-стагнація, асиметрія, дискомфорт у нетипових місцях. Внутрішні сигнали: зміна точки зусилля, зміна відчуття рівноваги і стійкості, «механічне» відчуття руху. Типові пастки: хибні позитиви (деградація vs. адаптація vs. варіація), reinvestment effect (Masters 2000), хибні негативи. Чотирикроковий протокол: базовий відеозапис, паузовані повторення як діагностика (60–65%, пауза 3–4 сек у ключовій точці), щотижневий перегляд RPE-трендів, контрольні технічні підходи раз на два тижні. Ліміти самостійної діагностики і роль епізодичного зовнішнього огляду. clinical-safety: NOT_REQUIRED. sports-scientist review pending. Blog card added.

### Changed
- `blog.html` — blog card for post-2244 added (inserted above post-2243).
- `README.md` — post-2244 → draft · v7.81.0.
- `STATUS.md` — v7.80.0 → v7.81.0; Total posts: 1997 → 1998; Editorial 2241–2250 in progress 3/10 → 4/10; NEXT: post-2245.
- `VERSION` — 7.80.0 → 7.81.0.

---

## [7.80.0] — 2026-06-22

### Added
- `content/post-2136-parallel-load-training-last-on-list.md` — Editorial 2131–2140, post 6/10. Паралельне навантаження: де тренування стає останнім у списку і що з цим робити. Відмінність «нема часу» від структурного паралельного навантаження (конкуруючі реальні пріоритети одночасно). Когнітивний механізм: ego depletion і виснаження ресурсу прийняття рішень (Baumeister et al. 1998). Три типові помилки (тренування ще важливіше при стресі; повна програма раз на тиждень; бінарне мислення). Матриця рішень: інтенсивність навантаження × тривалість → 4 конкретних виходи. Паралельна програма: 2 × тиждень, 30–45 хв, full-body, 3–4 компаундні вправи, збережена інтенсивність при скороченому об'ємі (Schoenfeld et al. 2017; Mujika & Padilla 2000: 33–66% об'єму + збережена інтенсивність = збереження силових показників 8–16 тижнів). Збереження поведінкового патерну важливіше за об'єм (Wood & Rünger, 2016). Коли вибрати усвідомлену паузу. П'ять помилок. Алгоритм re-entry: перший тиждень 70–75% об'єму. clinical-safety: NOT_REQUIRED. sports-scientist review pending. Blog card added.

### Changed
- `blog.html` — blog card for post-2136 added (inserted above post-2135).
- `README.md` — post-2136 → draft · v7.80.0.
- `STATUS.md` — v7.78.0 → v7.80.0; Total posts: 1996 → 1997; Editorial 2131–2140 in progress 5/10 → 6/10; NEXT: post-2137.
- `VERSION` — 7.79.1 → 7.80.0.

---

## [7.79.1] — 2026-06-22

### Changed
- `admin-dashboard.html` — Contract Renewal screen added (`data-screen="renewal"`). Closes COST_MODEL §42.10 items 3 & 7 (documentation phase). Two sections: (1) Renewal Economics fleet panel (`form_admin` read-only) — 4 metric cards (accounts renewing next 12mo, ARR at renewal, fleet health WAU-band distribution, escalation-eligible count + estimated ARR uplift), 12-month ARR-by-month bar chart (Jul 2026–Jun 2027), privacy floor callout (no individual tenant names in fleet view). (2) 3-step Renewal Workflow (`compliance-officer` role) — account selector + stepper: Step 1 Send Notice (`enterprise.renewal_notice_sent`, STANDARD, 7yr), Step 2 Calculate Escalation (locked for Year 1; unlocks for Year 2+ multi-year — CPI+1% formula, 5% hard cap, `bls_report_date` date-only), Step 3 Confirm Renewal (`enterprise.contract_renewed`, atomic with UPDATE `enterprise_contracts` + INSERT `enterprise_renewals`). RENEW-CHAIN-01 and ESCALATION-CHAIN-01 invariant callout with HTTP 422 behaviour documented. Privacy floor panel: fleet view shows tenant count only; workflow view shows account names for compliance-officer; no `user_id`, health data, or GDPR Art. 9 data. Sidebar nav item "Renewal" (↻) added between Billing and Audit log. VERSION 7.79.0 → 7.79.1. Cross-references: COST_MODEL §42 (renewal economics + 3-step workflow spec); COST_MODEL §31.5 (price floor); COST_MODEL §40 (privacy floor); DATA_MODEL §43 (`enterprise_renewals` DDL); AUDIT_LOG_SCHEMA §Enterprise (3 DEC-030 events); MSA_TEMPLATE §4.6 (escalation clause). compliance-officer + enterprise-architect.

## [7.79.0] — 2026-06-22

### Added
- `content/post-2243-practice-vs-training-repetition-learning.md` — Editorial 2241–2250, post 3/10. Практика проти тренування: чому повторення не завжди рівне навчанню. Продуктивність vs. навчання (Soderstrom & Bjork 2015): два різних виміри одного процесу. Desirable difficulties (Bjork 1994): умови, що знижують продуктивність у сесії, підвищують засвоєння. Контекстуальна інтерференція (Shea & Morgan 1979): перемежована практика краща за блокову для довгострокового навчання. Spacing effect і розподілена практика (Ebbinghaus; Donovan & Radosevich 1999 мета-аналіз): час між сесіями як механізм консолідації. Парадокс зворотного зв'язку (Schmidt et al. 1990; Winstein & Schmidt 1990): частий feedback погіршує автономне засвоєння. Variability of practice (Schmidt 1975 schema theory): варіативність зміцнює рухову схему. Алгоритм 4 кроки для self-coached атлета: визнач мету сесії, визнач фазу засвоєння, вибудуй умови практики, відрізняй результат практики від тренування. clinical-safety: NOT_REQUIRED. sports-scientist review pending. Blog card added.

### Changed
- `blog.html` — blog card for post-2243 added (inserted above post-2242).
- `README.md` — post-2243 → draft · v7.79.0.
- `STATUS.md` — Total posts: 1995 → 1996; Editorial 2241–2250 in progress 2/10 → 3/10.
- `VERSION` — 7.78.0 → 7.79.0.

---

## [7.78.0] — 2026-06-22

### Added
- `content/post-2242-neuromuscular-learning-cycle.md` — Editorial 2241–2250, post 2/10. Нейром'язовий цикл засвоєння руху: три фази від новизни до автоматизму. Проблема ступенів свободи (Bernstein 1967): нервова система звужує простір рішень, а не знаходить єдино правильний рух. Фаза 1 (когнітивна): активність префронтальної кори і M1, висока варіабельність між повторами як норма, а не помилка — нервова система «досліджує». Фаза 2 (асоціативна): базальні ганглії формують моторні програми; внутрішньом'язова координація (Sale 1988 — ранній нейральний приріст) і міжм'язова координація (Psek & Cafarelli 1993 — зайва коактивація антагоністів як маркер); паттерн є, але вразливий до стресу. Фаза 3 (автономна): мозочок, предикативний контроль (Wolpert & Ghahramani 2000) — нервова система передбачає корекції до виконання. Офлайн-консолідація: Walker et al. (2002) — REM-сон як інтегральна частина моторного навчання. Закон степеневої прогресії (Newell & Rosenbloom 1981) — чому ранній прогрес швидкий, а пізній — повільний. Три практичних тести для визначення фази. Протоколи тренування для кожної фази. clinical-safety: NOT_REQUIRED. sports-scientist review pending. Blog card added.

### Changed
- `blog.html` — blog card for post-2242 added (inserted above post-2241).
- `VERSION` — 7.76.1 → 7.78.0 (reconciles STATUS.md v7.77.0 discrepancy from prior iteration version bump error; v7.77.x skipped intentionally).

---

## [7.76.1] — 2026-06-22

### Changed
- `docs/COST_MODEL.md` v2.11 → v2.12 — §42.10 item 2 extended with canonical DDL reference: `docs/DATA_MODEL.md §43` (v1.22) cited as authoritative DATA_MODEL registration for migration 0084; platform-engineer directed to mark `docs/DATA_MODEL.md §43.8` item 1 done simultaneously. Closes `docs/DATA_MODEL.md §43.9` pending cross-reference obligation.
- `docs/DATA_MODEL.md` v1.24 → v1.25 — §43.9 cross-reference obligation row updated from 🟡 Pending to 🟢 Closed; all three §43.9 obligations now 🟢.
- `VERSION` — 7.76.0 → 7.76.1.

---

## [7.76.0] — 2026-06-22

### Added
- `content/post-2218-planning-next-block-from-retrospective.md` — Editorial 2211–2220, post 8/10. Планування наступного блоку: як завершений мезоцикл задає параметри наступного. Три типи даних із ретроспективи: перформанс-дані, дані про толерантність до обсягу, якісні дані. Три рішення для наступного блоку: ціль (Issurin 2010, концентрований адаптаційний потенціал), обсяг (Israetel et al. 2019, MEV/MRV — не підвищуй обсяг і інтенсивність одночасно), тижнева структура (де і коли виникав RPE-дрейф). Принцип «відповіді на запит» — три практичних сценарії. Що не змінюється між блоками: рухова база, частота, принципи прогресії. Мінімальний документ планування: ціль, параметри, гіпотеза, сигнали перегляду. clinical-safety: NOT_REQUIRED. sports-scientist review pending. Blog card added.
- `content/post-2219-systematic-errors-mesocycle-programming.md` — Editorial 2211–2220, post 9/10. П'ять системних помилок у програмуванні мезоциклів: шаблони, що руйнують блокову прогресію. Помилка 1 (ідентичні блоки без структурної прогресії): адаптаційна стеля — Issurin (2010). Помилка 2 (хронічний монотип): потенціювання між типами адаптацій — Zatsiorsky & Kraemer (2006). Помилка 3 (реактивний деловантаж): fitness-fatigue модель (Banister 1991, Chiu & Barnes 2003) — деловантаж після шкоди vs. превентивний. Помилка 4 (розрив адаптивного ланцюга між блоками): специфічність адаптацій — Haff & Triplett (2016). Помилка 5 (занадто багато змінних): концентрований блоковий підхід — Issurin (2010). Спільний знаменник: план без явної гіпотези. Таблиця шаблон → сигнал-попередження. clinical-safety: NOT_REQUIRED. sports-scientist review pending. Blog card added.
- `content/post-2220-mesocycle-programming-synthesis.md` — Editorial 2211–2220, post 10/10 (SYNTHESIS). Операційний протокол self-coached мезоциклового програмування. Модель: гіпотеза → тест → аналіз → нова гіпотеза. П'ять фаз: Фаза 1 планування (ретроспектива, вибір цілі, параметри, документ блоку); Фаза 2 виконання (RPE-моніторинг, авторегуляція: три виправдані внутрішньоблокові корекції, технічний моніторинг); Фаза 3 деловантаж (протокол −40–60%, ретроспектива після, не до); Фаза 4 перехідний тиждень (60–75% обсягу, збереження тонусу, ≠ продовжений деловантаж); Фаза 5 наступний блок (три рішення). Довідкова таблиця параметрів за типом блоку (силовий/гіпертрофійний/технічний/відновний). Два чеклисти: початок нового блоку і завершення блоку. clinical-safety: NOT_REQUIRED. sports-scientist review pending. Blog card added. **Editorial 2211–2220 COMPLETE 10/10.**

### Changed
- `blog.html` — blog cards for posts 2218, 2219, 2220 added (inserted above post-2217).
- `VERSION` — 7.75.1 → 7.76.0.

---

## [7.75.1] — 2026-06-22

### Changed
- `docs/SOC2_READINESS.md` v3.25.3 — §95 R-37 addendum patch. Closes `docs/INCIDENT_RESPONSE.md R-37.11` item 3 (P1/M11): §95.1 SSO-FLEET-E-001 description updated to reference R-37 activation addenda + new `R-37 addendum` table row; §95.2 Part D added (SQL query over `system.sso_fleet_check_failure_declared` / `system.sso_fleet_check_restored` DEC-030 pairs, privacy floor, CC7.3 SOC 2 mapping); §95.3 zero-incident attestation JSON updated with `r37_activations: 0` field; §95.4 CC7.3 criteria row updated to reference Part D and INCIDENT_RESPONSE R-37; §95.5 R-37.11 item 3 closure row added. Job 38 numbering confirmed correct throughout §95. Owner: compliance-officer + security-engineer.
- `docs/INCIDENT_RESPONSE.md` v1.1 — R-37.11 item 3 marked `[x] Done — 2026-06-22, SOC2_READINESS.md v3.25.3`; v1.1 version note added. Owner: compliance-officer.
- `VERSION` — 7.75.0 → 7.75.1.

---

## [7.75.0] — 2026-06-22

### Added
- `content/post-2217-how-to-end-a-block.md` — Editorial 2211–2220 post 7/10. Як завершити блок правильно: пікова сесія, тест або плавний перехід. Три підходи структурного завершення мезоциклу: пікова сесія (силовий блок — після тижня зниженого обсягу зі збереженою інтенсивністю, Haff & Triplett 2016), ретроспективний тест (гіпертрофійний/технічний блок — RPE-калібрування: стартова вага з RPE 8 → RPE 6.5 після блоку; проводити після деловантажу, не до — у стані втоми оцінка зміщена), плавний перехід (відновний блок, серії накопичення, перший блок після перерви — без формального тесту, але обов'язкова ретроспектива журналу). Таблиця вибору підходу по цілі блоку. Мінімальний набір даних для фіксації: робочі ваги + RPE на початку/кінці, тижневий обсяг на фіналі, суб'єктивні маркери, оцінка цілі блоку. П'ять помилок: тест у стані максимальної втоми, 1RM після гіпертрофійного блоку без тейпер-підготовки, плавний перехід без ретроспективи, некоректно відкалібрований початковий RPE, пікова сесія як ритуал після кожного блоку. Issurin (2010) — концентрований адаптаційний потенціал блоку; Zatsiorsky & Kraemer (2006); Haff & Triplett (2016). clinical-safety: NOT_REQUIRED. sports-scientist review pending.

### Changed
- `blog.html` — card for post-2217 added (перед post-2216); card for post-2216 added retroactively (перед post-2215) · v7.75.0.
- `STATUS.md` — Editorial 2211–2220: IN PROGRESS 7/10 · v7.75.0. NEXT: post-2218.
- `VERSION` — 7.74.0 → 7.75.0.

---

## [7.74.0] — 2026-06-22

### Added
- `content/post-2216-transition-between-blocks.md` — Editorial 2211–2220 post 6/10. Перехід між блоками: чому два тижні без структури — помилка. Детренінг: нейром'язова деградація через 7–14 днів без структурованого тренування (Mujika & Padilla 2001) — −5–13% сили через зниження нейром'язової активації, ще без морфологічних змін. Послідовність детренінгу: нейром'язова активація (дні 7–10) → метаболічні зміни (дні 10–28) → атрофія міофібрил (28+ днів). Перехідний тиждень як третій стан між деловантажем і новим блоком — відрізняється від обох: обсяг 60–75% від нормального тижня, помірна інтенсивність, діагностичний акцент (re-testing робочих ваг, калібрування RPE). Зміна мети блоку: технічний аудит нових рухів у перехідному тижні. П'ять помилок переходу: нульовий перехід, прямий стрибок з деловантажу в максимум, відсутність re-testing, ігнорування технічного аудиту, умовне планування перехідного тижня. Граничний ефект відпочинку після повного зняття втоми — від'ємний (Zatsiorsky & Kraemer 2006). Issurin (2010): block periodization — кожен блок будується на адаптаційному фундаменті попереднього. Fleck & Kraemer (2014). clinical-safety: NOT_REQUIRED. sports-scientist review pending.

### Changed
- `README.md` — post-2216 marked as draft · v7.74.0.
- `STATUS.md` — Editorial 2211–2220: IN PROGRESS 6/10 · v7.74.0. NEXT: post-2217.
- `VERSION` — 7.73.0 → 7.74.0.

---

## [7.73.0] — 2026-06-22

### Added
- `content/post-2215-deload-week-in-mesocycle.md` — Editorial 2211–2220 post 5/10. Деловантажний тиждень у структурі блоку: де ставити і чому. Fitness-fatigue модель (Banister 1991, Chiu & Barnes 2003): втома маскує фітнес — реальна продуктивність нижча за потенціал атлета в умовах накопиченої втоми. Israetel, Hoffmann & Case (2019): MEV/MAV/MRV і деловантаж як структурне зниження від MRV до MEV. Програмований деловантаж (3+1, 4+1, 5+1 шаблони) vs. реактивний (RPE-дрейф +1.5, технічна деградація у первинних рухах, ЧСС спокою +5–7 bpm, мотиваційна апатія). Протокол деловантажу: −40–60% обсягу, збережена інтенсивність або −5–10%, 3–4 RIR у всіх підходах. П'ять помилок: пропуск при «нормальному самопочутті», зниження ваги замість обсягу, повний відпочинок замість деловантажу, занадто часті деловантажі (кожні 2 тижні), відсутність протоколу. Що відбувається після: RPE −0.5–1.5 при поверненні = «розкриття фітнесу». Mujika & Padilla (2001) — деловантаж ≠ детренінг. Zatsiorsky & Kraemer (2006), Haff & Triplett (2016). clinical-safety: NOT_REQUIRED. sports-scientist review pending.

### Changed
- `blog.html` — card for post-2215 added (перед post-2214).
- `README.md` — Editorial 2231–2240 (Пікування і тестування) added as proposed series.
- `STATUS.md` — Editorial 2211–2220: IN PROGRESS 5/10 · v7.73.0. NEXT: post-2216.
- `VERSION` — 7.72.0 → 7.73.0.

---

## [7.72.0] — 2026-06-22

### Added
- `content/post-2135-seasonal-breaks-detraining.md` — Editorial 2131–2140 post 5/10. Сезонні перерви без паніки: скільки реально втрачається і що повертається першим. Детренінг vs. тейпер: чому це різні фізіологічні ситуації. Три системи, три хронології: нейром'язові адаптації (перші деградують — 1–2 тижні у тренованих, Mujika & Padilla 2001), структурна гіпертрофія (значно стійкіша — після 3 тижнів детренінгу втрата CSA мінімальна, Ogasawara et al. 2013), кардіореспіраторна (4–14% VO₂max за 2–4 тижні — Coyle et al. 1984, але для силових нерелевантно). Міонуклеарна пам'ять — Bruusgaard et al. (2010, PNAS): міонуклеї зберігаються ≥3 місяців після детренінгу; біологічний субстрат м'язової пам'яті. Практична хронологія по тижнях (1–2/3–4/5–8/8–16/>16 тижнів). Що повертається першим: нейром'язова адаптація (3–5 сесій), силові показники (50–70% від тривалості детренінгу), гіпертрофія (прискорена через збережений міонуклеарний потенціал). Три фази повернення: реактивація (50–60% обсягу, RPE 6–7, тижні 1–2), калібрування (70–80%, пошук стартових точок, тижні 3–4), повноцінний мезоцикл (тиждень 5+). П'ять помилок при поверненні (доперервані ваги у першій сесії; повний обсяг відразу; ігнорування DOMS; порівняння з собою до перерви; панічні компенсаторні надобсяги). Рамка «без паніки»: три операційних принципи. clinical-safety: NOT_REQUIRED. sports-scientist review pending.

### Changed
- `blog.html` — card for post-2135 added (перед post-2134).
- `STATUS.md` — Editorial 2131–2140: IN PROGRESS 5/10 · v7.72.0. NEXT: post-2136.
- `VERSION` — 7.71.0 → 7.72.0.

---

## [7.71.0] — 2026-06-22

### Added
- `content/post-2214-weekly-structure-within-mesocycle.md` — Editorial 2211–2220 post 4/10. Побудова тижня всередині мезоциклу: ієрархія рухів і розподіл стресу. Чому частота — не структура: два атлети з однаковою частотою можуть отримати принципово різний результат. Schoenfeld et al. (2016): при однаковому тижневому обсязі вища частота дає незначний гіпертрофійний ефект (3.1% vs 2.7%) — справжня цінність частоти в MEV per session vs. MEV per week (Israetel et al. 2019). Ієрархія рухів: первинні (змагальні або близькі варіанти), вторинні (підтримуючі патерни), аксесуарні (ізоляція, корективна). Бюджет стресу: первинний рух завжди першим — cost of sequencing error. SRA-крива і розподіл стресу: кластеризація важкої роботи на суміжних днях накопичує втому без пропорційної адаптації (48–72 год відновлення після RPE 9 на великих групах). HLM-хвиля інтенсивності (важкий-легкий-середній): знижує залишкову втому без втрати тижневого обсягу. Три практичних шаблони: Upper/Lower (4 дні), PPL (3 і 6 днів), Full-Body (2–3 дні) — для кожного ієрархія рухів і розподіл інтенсивності. П'ять помилок тижневої структури. Zatsiorsky & Kraemer (2006), Haff & Triplett (2016), Kramer & Ratamess (2004). clinical-safety: NOT_REQUIRED. sports-scientist review pending.

### Changed
- `blog.html` — card for post-2214 added (перед post-2213).
- `STATUS.md` — Editorial 2211–2220: IN PROGRESS 4/10 · v7.71.0. NEXT: post-2215.
- `VERSION` — 7.70.1 → 7.71.0.

---

## [7.70.1] — 2026-06-22

### Added
- `docs/INCIDENT_RESPONSE.md` — R-37 SSO Fleet Health Check Stale: thirty-seventh runbook, closes the documentation gap for `sso_fleet_health_check` pg_cron job 38 (`*/5 * * * *`, 6-min freshness window). Full IC protocol for SSO-SLO-01b detection gap: trigger matrix (PagerDuty `form-platform` P1 → IC + security-engineer, dedup `sso-fleet-health-check-stale`); two-severity matrix (P1: stale with < 3 tenants failing — detection gap only; P0: stale AND R-37-C1 ≥ 3 tenants — per-tenant SSO-SLO-01 assessment + R-04 co-activation); T+0–T+30 immediate actions; two scope queries (R-37-C1 fleet SSO failure scope, R-37-C2 pg_cron run history); four root cause hypotheses (H1 job deleted, H2 shared pg_cron failure, H3 `audit_log_events` query failure, H4 Supabase outage); four-step recovery; SSO-FLEET-INT-01 internal P0 template; DEC-030 HMAC chain (`system.sso_fleet_check_failure_declared` HIGH/7yr + `system.sso_fleet_check_restored` STANDARD/3yr + SSO-FLEET-CHAIN-01 ordering invariant); SSO-FLEET-E-001 evidence via R-37 addendum; CC7.2/CC7.3 SOC 2 mapping; four-item implementation checklist. Critical distinction: purely advisory detection-gap runbook — no data mutation, no external API polling required. DEC-070 hard invariant: `sla_credit_impact: 'none'` for SSO-SLO-01b; per-tenant SSO-SLO-01 remains the SLA credit basis.

### Changed
- `docs/OBSERVABILITY.md` — §12.6 pg_cron job 38 registry entry: stale-consequence cross-ref updated to include `INCIDENT_RESPONSE R-37 (job 38 stale recovery runbook — §R-37.5)` · v4.8.5.
- `VERSION` — 7.70.0 → 7.70.1.

---

## [7.70.0] — 2026-06-22

### Added
- `content/post-2213-block-goal-selection.md` — Editorial 2211–2220 post 3/10. Як обрати ціль блоку: силовий, гіпертрофійний, технічний, відновний. Чому один блок — одна пріоритетна ціль (Issurin 2010 — block periodization, концентроване навантаження). Чотири типи блоків із параметрами: силовий (>80%, 1–5 повт., нейром'язова адаптація — Zatsiorsky & Kraemer 2006), гіпертрофійний (60–80%, 6–15 повт., механічна напруга — Schoenfeld 2010; MAV — Israetel et al. 2019), технічний (60–75%, якість первинна), відновний (MED — Bickel et al. 2011, 30–50% обсягу від попереднього блоку). Три чинники вибору типу блоку: поточний тренувальний стан, попередній блок (conjugate sequencing), горизонт планування. Практичні схеми послідовності блоків для базового року і для підготовки до тестування (12-тижнева структура). П'ять помилок у виборі цілі блоку. Як формулювати вимірювану ціль для кожного типу. clinical-safety: NOT_REQUIRED. sports-scientist review pending.

### Changed
- `blog.html` — card for post-2213 added (перед post-2211).
- `README.md` — post-2213 → draft · v7.70.0; Editorial 2221–2230 (Читання адаптацій у реальному часі) added as proposed · v7.70.0.
- `STATUS.md` — Editorial 2211–2220: IN PROGRESS 3/10 · v7.70.0. NEXT: post-2214.
- `VERSION` — 7.69.0 → 7.70.0.

---

## [7.69.0] — 2026-06-22

### Added
- `content/post-2212-progressive-overload-within-mesocycle.md` — Editorial 2211–2220 post 2/10. Прогресивне навантаження всередині мезоциклу: що підвищувати, коли і на скільки. Шість змінних прогресії (вага, обсяг, щільність, частота, варіація, RPE-тренд). SRA-крива (Selye 1950) і фазова логіка прогресії (накопичення vs. інтенсифікація). Чому лінійна прогресія «+2.5 кг щотижня» не є стратегією для intermediate. Принцип SAID (Zatsiorsky & Kraemer 2006). Подвійна прогресія (Double Progression): рухатись у повтореннях до верхнього порогу діапазону, потім підвищувати вагу. Потрійна прогресія (Triple Progression): три осі — повторення → підходи → вага, для гіпертрофійних блоків. MAV (Israetel et al. 2019): поріг максимального адаптивного обсягу і симптоми перевищення. Відсотковий підхід проти RPE-авторегуляції: переваги, обмеження, гібридна стратегія. RPE-дрейф: три послідовні сесії з наростаючим RPE без приросту продуктивності — сигнал корекції або деловантаження. Kiely (2012): деловантаження як фаза завершення адаптаційних процесів. Правило одного провідного стресора за фазу. Практичний шаблон 6-тижневого блоку з відсотками і RPE-цілями. clinical-safety: NOT_REQUIRED. sports-scientist review pending.

### Changed
- `blog.html` — card for post-2212 added (перед post-2134).
- `README.md` — post-2212 → draft · v7.69.0; тема уточнена (progressive overload within mesocycle); post-2213 → proposed «Як обрати ціль блоку».
- `STATUS.md` — Editorial 2211–2220: IN PROGRESS 2/10 · v7.69.0. NEXT: post-2213.
- `VERSION` — 7.68.1 → 7.69.0.

---

## [7.68.1] — 2026-06-22

### Changed
- `docs/INCIDENT_RESPONSE.md` — R-36 Webhook Escalation Sweep Stale: thirty-sixth runbook. IC protocol for when `webhook_degraded_escalation_check` pg_cron job 41 (`*/30 * * * *`, 35-min freshness) exceeds its stale window. Two-severity matrix with dual-P0 sub-cases: P1 (stale, no active SLO breach); P0 (notification breach — R-36-C1 > 0: degraded webhook past WH-SLO-04 2 h window without slo_breach event; 30-min SLA) and/or P0 (suspension deadline miss — R-36-C2 > 0: degraded webhook past 48 h without suspended event; 15-min SLA). Critical distinction: unlike R-35 (detection-gap only), R-36 requires both customer notification AND database state mutation as compensating controls. H3 (Resend API failure) is a novel vendor-dependency risk not present in peer stale runbooks. Five root causes (H1 job deleted, H2 DDL change on `tenant_webhooks`, H3 Resend API, H4 lock contention from `webhook-dispatcher` high-write load, H5 Supabase outage → R-03). Three scope queries (R-36-C1 notification gate, R-36-C2 suspension gate, R-36-C3 pg_cron run history). Four-step recovery with Step 2c (PAM-elevated suspension UPDATE + integration.webhook_suspended HIGH/7yr + CSM WH-NOTIF-02 equivalent) before Step 2b (integration.webhook_delivery_slo_breach STANDARD/3yr + CSM WH-NOTIF-01 equivalent or direct email H3 fallback). DEC-030 events: `system.webhook_escalation_stale_declared` HIGH/7yr + `system.webhook_escalation_restored` STANDARD/3yr; WH-ESCALATION-CHAIN-01 ordering invariant. Evidence: WH-ESCALATION-E-001 (quarterly pg_cron.job_run_details export, CC7.2/CC9.2, 3yr). Four-item implementation checklist (2× P0/M10, 2× P1/M10–M11). Closes documentation gap left by OBSERVABILITY.md §12.6 v0.9 patch (2026-06-22) which registered job 41 without a stale runbook cross-reference.
- `docs/OBSERVABILITY.md` — §12.6 v1.0 patch: job 41 `webhook_degraded_escalation_check` stale-consequence cross-ref extended with `INCIDENT_RESPONSE R-36 (job 41 stale recovery runbook — §R-36.5)`. Closes R-36.10 post-incident control and R-36.11 checklist item 4. DEC-030 registration obligation (R-36.11 item 1) and SOC2_READINESS evidence registration (R-36.11 item 3) remain pending.
- `VERSION` — 7.68.0 → 7.68.1.

---

## [7.68.0] — 2026-06-22

### Added
- `content/post-2211-mesocycle-practical-definition.md` — Editorial 2211–2220 post 1/10. Що таке мезоцикл для практичних цілей. Часові константи адаптацій (Moritani & DeVries 1979; Schoenfeld 2010; Magnusson 2010). Три фази всередині блоку: накопичення / інтенсифікація / деловантаження. Чотири типи мезоциклу (силовий / гіпертрофійний / технічний / відновний). Перехід між блоками — conjugate sequencing (Issurin 2010). Три маркери хорошого блоку (e1RM або RPE-тренд, технічна якість, реакція на деловантаження). П'ять помилок управління мезоциклом. Peterson et al. (2011): adherence як сильніший предиктор результату за схему. Kiely (2012): деловантаження як фаза завершення адаптаційних процесів. clinical-safety NOT_REQUIRED. sports-scientist review pending.

### Changed
- `blog.html` — card for post-2211 added (перед post-2134).
- `STATUS.md` — Editorial 2211–2220: IN PROGRESS 1/10 · v7.68.0. NEXT: post-2212.
- `README.md` — Editorial 2211–2220 block added to roadmap (10 proposed topics).
- `VERSION` — 7.67.1 → 7.68.0.

---

## [7.67.0] — 2026-06-22

### Added
- `content/post-2134-two-sessions-per-week.md` — Editorial 2131–2140 post 4/10. Два тренування на тиждень замість чотирьох: не деградація, а перемонтаж. Мета-аналіз Schoenfeld et al. (2016): при рівному тижневому об'ємі різниця між 1–3 тренуваннями на тиждень статистично незначима для гіпертрофії. Ralston et al. (2017): аналогічний висновок для сили. Vikberg et al. (2019): одна сесія/тиждень при достатньому об'ємі дає значущий ефект. Математика перемонтажу: три варіанти структури (Full-body × 2 / Upper + Lower / Push-Pull). Ієрархічний фільтр: рухові патерни → якірна вправа → аксесуари в кінці. Відновлення: мінімум 72 год між сесіями; Dattilo (2011) — якість сну критичніша при концентрованих навантаженнях. «Стеля vs. підлога» як різна психологічна рамка. Що не робити: A/B варіації в одній й тій самій сесії, хаотична третя сесія, компенсація об'єму через інтенсивність. Алгоритм перемонтажу 6 кроків: режим → структура → патерни → тижневий об'єм-ціль → дні та відновлення → оцінка через 3–4 тижні. clinical-safety NOT_REQUIRED. sports-scientist review pending.

### Changed
- `blog.html` — card for post-2134 added (перед post-2133).
- `STATUS.md` — Editorial 2131–2140 updated: IN PROGRESS 4/10 · v7.67.0. NEXT: post-2135.
- `VERSION` — 7.66.0 → 7.67.0.

---

## [7.67.1] — 2026-06-22

### Changed
- `docs/OBSERVABILITY.md` — §12.6 pg_cron canonical registry v0.9 patch: registered jobs 40 (`white_label_cert_check`, daily 02:00 UTC, 26h freshness, CC7.2/A1.2) and 41 (`webhook_degraded_escalation_check`, every 30 min, 35-min freshness, CC9.2/CC7.2). Resolves two job-number conflicts: §42.7 (v3.9, 2026-06-14) had claimed "job 32" for `white_label_cert_check` and §43.7 (v4.0, 2026-06-14) had claimed "job 34" for `webhook_degraded_escalation_check` — both numbers were already canonical in §12.6 since the v0.4 patch (job 32 = `turh_retention_purge`; job 34 = `bdg_override_expiry_sweep`). In-text citations corrected throughout §42.7, §42.14 item 3, §43.7, §43.15 items 6–8. Conflict resolution note v0.9 added. Freshness window note extended to include jobs 40 and 41.
- `docs/INCIDENT_RESPONSE.md` — R-35 stale recovery runbook added for `white_label_cert_check` (job 40): P1 default (stale, no at-risk certs) / P0 escalation (stale + ≥ 1 active cert expiring within 7 days); scope query R-35-C1; DEC-030 events `system.wl_cert_check_failure_declared` (HIGH/7yr) and `system.wl_cert_check_restored` (STANDARD/3yr); chain invariant WL-CERT-CHAIN-01; SOC 2 evidence WL-CERT-E-001 (CC7.2/A1.2, quarterly 3yr); internal communication template WL-CERT-INT-01 (P0 only). Completes the stale-runbook obligation introduced by OBSERVABILITY.md §42.7 (v3.9). Closes §12.6 job 40 cross-ref to INCIDENT_RESPONSE R-35.
- `VERSION` — 7.67.0 → 7.67.1.

---

## [7.66.0] — 2026-06-22

### Added
- `content/post-2133-hotel-gym-travel-protocol.md` — Editorial 2131–2140 post 3/10. Hotel gym: мінімально-ефективний протокол для відрядження. Тривалість поїздки як перше рішення (1–2 дні / 3–5 днів / 7+). Мінімальний ефективний стимул: 1/9 об'єму → силова підтримка, 1/3 → гіпертрофійна (Bickel et al. 2011). Механізм eccentric tempo і mechanical tension без зовнішнього навантаження (Schoenfeld 2010). Протокол А (гантелі 15–30 кг, 5 патернів × 3 сети, темп 3-1-2, 35–45 хв). Протокол Б (bodyweight only, 4 патерни, 2–3 сети, 30–40 хв). Що пропустити: PR-спроби, ізоляція, максимальний об'єм, важкі технічні підйоми без стійки. RPE-калібрація у відрядженні: Waterhouse (2007) jet lag + Judelson (2007) hydration → суб'єктивне навантаження природно вище, стеля RPE 7–8. Повернення: -15–20% об'єму перший тиждень, без компенсаційної логіки. Дерево рішень 4 кроки. clinical-safety NOT_REQUIRED. sports-scientist review pending.
- `README.md` — Editorial block 2201–2210 "Практика самоконтролю" додано до roadmap (proposed · v7.66.0).

### Changed
- `blog.html` — card for post-2133 added (перед post-2132).
- `STATUS.md` — Editorial 2131–2140 updated: IN PROGRESS 3/10 · v7.66.0. NEXT: post-2134 (два тренування на тиждень замість чотирьох).

---

## [7.65.0] — 2026-06-22

### Added
- `content/post-2132-irregular-schedule-training-structure.md` — Editorial 2131–2140 post 2/10. Нерегулярний графік і тренування: мінімальна структура для максимального утримання. Три типи нерегулярності: хаотичний тиждень / змінна робота / сезонна нестабільність. Концепція якірної сесії (один гарантований слот) + опортуністичних сесій (заповнюєш вікно, не плануєш). Математика утримання: 100% реалістичного плану > 50% амбітного. Мінімальна ефективна доза: 1 сесія/тиждень достатня для підтримки адаптації (maintenance), 2+ — для прогресу. Full-body або мінімальний спліт — кожна сесія самодостатня. Правило «не компенсуй»: максимум +1 опортуністична у хороший тиждень. Горизонт планування: 4–6 тижнів, а не тижень. Автоматичні deload-тижні при нерегулярному графіку — особливість, а не баг. Три операційні схеми + шість операційних правил. clinical-safety NOT_REQUIRED. sports-scientist review pending.

### Changed
- `blog.html` — card for post-2132 added (перед post-2131).
- `STATUS.md` — Editorial 2131–2140 updated: IN PROGRESS 2/10 · v7.65.0. NEXT: post-2133 (hotel gym протокол для відрядження).

---

## [7.64.1] — 2026-06-22

### Changed
- `enterprise.html` — Quarterly compliance attestation badge added to the Compliance &amp; security posture section (8th badge, green-teal accent, after Annual penetration tests). Closes the gap between `docs/ENTERPRISE.md §Hard commitments` ("Quarterly compliance attestation") and the enterprise landing page — this hard commitment was documented in ENTERPRISE.md but not surfaced to procurement audiences. Badge copy: "Written attestation each quarter — SOC 2 controls status, audit chain integrity, and privacy floor verification." Owners: compliance-officer + enterprise-architect. Cross-reference: `docs/ENTERPRISE.md` (hard commitments list). Clinical-safety: NOT_REQUIRED.

---

## [7.64.0] — 2026-06-22

### Added
- `content/post-2131-sleepless-night-training-algorithm.md` — Editorial 2131–2140, post 1/10. Тренування після безсонної ночі: триступеневий алгоритм рішення (класифікація контексту → режим сесії → параметри модифікації). Відмінність ситуативної і хронічної нестачі сну. Три умови для повного скасування. Пострестораційний принцип. Мотиваційний парадокс: алгоритм замінює мотивацію критеріями. clinical-safety: NOT_REQUIRED.
- `blog.html` — card for post-2131 added.
- `README.md` — Editorial 2191–2200 block (Психологія і поведінка self-coached атлета) added to roadmap.

### Changed
- `STATUS.md` — Editorial 2131–2140 marked IN PROGRESS 1/10; post-2131 noted.

---

## [7.63.0] — 2026-06-22

### Added
- `content/post-2130-self-coached-operational-thinking-synthesis.md` — Editorial 2121–2130 block synthesis, post 10/10. Синтез операційного мислення self-coached атлета: десять постів блоку зведені в єдину операційну петлю (спостерігай → інтерпретуй → вирішуй → виконуй). Чотири ключові принципи: явність моделі, pre-commitment, рівнева відповідність рішень, пропорційність порогу. Відкриті питання блоку, зв'язок з блоком 2131–2140.
- `blog.html` — cards for posts 2128, 2129, 2130 added.

### Changed
- `STATUS.md` — Editorial 2121–2130 marked COMPLETE 10/10; next priorities updated to Editorial 2131–2140.
- `README.md` — posts 2128–2130 status updated to draft.

---

## [7.62.0] — 2026-06-22

### Added
- `content/post-2129-decisions-with-incomplete-data.md` — Editorial 2121–2130, post 9/10. Рішення при неповних даних: алеаторна vs. епістемічна невизначеність, принцип зворотності, асиметрія помилок типу I і II у тренуванні. Техніка pre-commitment («що б мене переконало») — до досвіду, не після. Чотирикрокова структура прийняття рішень при невизначеності. clinical-safety: NOT_REQUIRED.

---

## [7.61.0] — 2026-06-22

### Added
- `content/post-2128-systemic-error-recognition.md` — Editorial 2121–2130, post 8/10. Сім ознак системної помилки в тренувальному підході. Різниця між тактичною помилкою (корекція параметра) і системною (перегляд моделі). Психологічні механізми, що утримують від перегляду: sunk cost, ідентифікація з підходом, невизначеність. П'ятикроковий протокол перегляду. clinical-safety: CONDITIONAL_PASS — guard rail for pain/injury embedded.

---

## [7.60.1] — 2026-06-22

### Changed
- `docs/INCIDENT_RESPONSE.md` — v3.4: Scenario O (Linear API unavailable during active P0) added to §9.3 tabletop catalog. Scenario covers AL-IR-LINEAR-01 (P2 Slack after 5× exponential-backoff, 62 s), `incident.opened` DEC-030 HIGH/7yr chain lock with `linear_ticket_url: "PENDING"` before Linear failure, `POST /internal/v1/incident/link-ticket` PAM-elevated amendment endpoint (§17.2.5), `incident.linear_ticket_linked` LOW/7yr `linked_by: "manual_ic"`. Five discussion points: chain lock at T+0; amendment endpoint PAM at 02:15 UTC; PENDING gap in IR-AUTO-E-001; R-03 co-trigger threshold; manual vs auto path in IR-AUTO-E-002 P95. §9.5 Month 22 updated: lightweight 30-min tabletop (security-engineer + platform-engineer; staging drill disabling Linear API, triggering synthetic SIEM alert, running amendment path, confirming chain record). §17.6 item 8 marked [x] Done. Closes the §17 `siem-incident-automator` P1 documentation obligation.
- `VERSION` → 7.60.1

---

## [7.60.0] — 2026-06-22

### Added
- `content/post-2127-confirmation-bias-training.md` — Editorial 2121–2130 post 7/10. Confirmation bias у тренуванні: як атлет ненавмисно шукає підтвердження, а не дані. Три форми прояву: (1) селективний запис — детальний по хороших сесіях, скорочений по поганих, що системно спотворює ретроспективну оцінку блоку; (2) ретроспективне перекалібрування RPE — автоматична нормалізація реального числа до очікуваного плану; (3) вибіркова інтерпретація аномалій — стагнація «складної вправи» отримує пояснення, стагнація «простої» стає проблемою. Чотири точки найбільшого ризику: рішення залишитись на програмі / рішення про деловантаження / рішення про зміну вправи або техніки / блокова ретроспективна оцінка. Сім ознак самодіагностики. П'ять операційних контрзаходів: запис RPE до перегляду плану; стандарт мінімального запису для всіх сесій незалежно від якості; систематичний пошук дисконфірмуючих даних (3 конкретні спостереження раз на два тижні); фіксована шкала для аномалій без обов'язкової інтерпретації; розділення факту і пояснення у записі. Парадокс тривалого self-coaching: чим більше аналізуєш самостійно — тим більше дані проходять крізь ту саму підтверджувальну призму. Мета — не нейтральний атлет, а система записів, що зберігає дані, які суперечать поточній моделі, у тому самому форматі, що й підтверджувальні. clinical-safety: NOT_REQUIRED.

### Changed
- `blog.html` — post-2127 card додана вище post-2126; post-2126 card додана (пропущена у v7.59.0).
- `README.md` — post-2127 → draft · v7.60.0; Editorial 2181–2190 «Невизначеність і рішення» (10 topics) proposed · v7.60.0.
- `STATUS.md` — Editorial 2121–2130 IN PROGRESS 7/10; NEXT: post-2128. Editorial 2181–2190 proposed noted.
- `VERSION` → 7.60.0

---

## [7.59.0] — 2026-06-22

### Added
- `content/post-2126-does-this-program-fit-me.md` — Editorial 2121–2130 post 6/10. Як визначити, чи програма підходить тобі — без тренера поруч. Центральна діагностична проблема self-coached атлета: відрізнити «ще не адаптувався» (норма для перших 4 тижнів нового блоку) від «програма структурно не відповідає». Помилка першого роду: зміна програми на тижні 3 (перерваний адаптаційний процес). Помилка другого роду: тримання невідповідної програми 6 місяців (накопичені збитки). Мінімальне вікно оцінки: 4 тижні (виключення адаптаційного шуму) і 8 тижнів (оцінка кумулятивного ефекту). Три виміри відповідності: обсяг (чи можеш відновитись), інтенсивність (чи відповідають % 1RM рівню підготовки), частота (чи вписується в реальний тиждень і відновлення). Сигнали відповідності: прогресія після 8 тижнів, стабілізоване відновлення після 4 тижнів, плановий обсяг виконується на ≥90%, техніка стабільна або покращується, RPE відповідає плановому. Сигнали невідповідності: систематичне скорочення обсягу у >25% підходів через відновлення, RPE-overrun у >25% сесій, деградація техніки після деловантаження, відсутність прогресії після 8 тижнів при достатньому відновленні і техніці. Критичне розрізнення: «програма не підходить» vs. «я не виконую програму» — чотири типи невиконання (пропуск сесій, систематичне заниження навантаження, довільна заміна вправ, пропуск допоміжної роботи). Практична схема: 10 питань після тижня 4 і тижня 8, шкала 8–10/5–7/<5. Коли підтверджена невідповідність — коригуй конкретний вимір (обсяг → знизь підходи; інтенсивність → перерахуй ваги; частота → інша структура; технічний рівень → технічна робота спочатку), а не шукай нову програму. Застереження: програма, що «підходить», не є синонімом «комфортної». Реальна відповідність = достатньо стресово для адаптації AND достатньо відновлювано для послідовного виконання. clinical-safety: NOT_REQUIRED.

### Changed
- `README.md` — post-2126 → draft · v7.59.0.
- `STATUS.md` — Editorial 2121–2130 IN PROGRESS 6/10; NEXT: post-2127.
- `VERSION` → 7.59.0

---

## [7.58.1] — 2026-06-22

### Changed
- `docs/INCIDENT_RESPONSE.md` — v3.2 → v3.3. §9.3 Tabletop Scenario Catalog extended: six new scenarios (G, H, I, J, K, X) consolidated from their respective runbook sections (R-12.12, R-13.14, R-14.11, R-17, R-16, R-34) into the canonical drill catalog. Scenario G: insider data exfiltration across 14 enterprise tenants, Art. 33 clock question, legal observer required. Scenario H: US DOJ criminal subpoena covering EU-resident enterprise users, Art. 48 GDPR analysis, scope minimisation against 'all records', external counsel required. Scenario I: DSAR export cross-tenant contamination (420-user tenant, Art. 9 data in wrong exports, ICO 72-h partial filing). Scenario J: Anthropic API 90-min outage during live enterprise sessions, fallback SLA validation, clinical_safety_bypass DEC-030 emission check. Scenario K: API key committed to public GitHub fork with junior IC + CO on PTO, R-16 rotation order, Art. 33 assessment matrix. Scenario X: turh_retention_purge job 32 stale, R-34-C1 P1/P0 gate query, staging drill procedure including optional P0 pseudonymisation path. §9.5 Year 2 schedule: Month 21 → Scenario G (insider exfiltration); Month 24 tabletop pool expanded to include H and I. §9.6 Year 3 Testing Schedule added (M25–M36): H at M27, I+Fire drill at M30, J+K dual tabletop at M33, X+pen test at M36. R-16 Scenario K checklist item [x] Done. R-34.11 item 6 Scenario X checklist item [x] Done. · v7.58.1
- `VERSION` → 7.58.1

## [7.58.0] — 2026-06-22

### Added
- `content/post-2125-three-types-of-plateau.md` — Editorial 2121–2130 post 5/10. Три типи плато — і три різних рішення для self-coached атлета. Тип 1: накопичена втома маскує реальний прогрес (рішення — деловантаження, не більше роботи). Тип 2: технічна стеля — сила є, механіка не дозволяє її реалізувати (рішення — навмисна регресія навантаження на 15–25%, 4–8 тижнів технічного освоєння). Тип 3: програмна стеля — організм адаптувався до поточного стимулу (рішення — зміна rep range, частоти, головної вправи або схеми прогресії). Схема діагностики з трьох кроків: перевір відновлення → перевір техніку → перевір тривалість плато. Дві систематичні помилки self-coached атлета: «пробити плато» збільшенням об'єму при типі першому; «продовжувати програму» без змін при типі третьому. Правило накладання типів: завжди усуни тип перший першим. Мінімальний набір записів для майбутньої діагностики: RPE по підходах, відеозапис кожен 2–3 тижні, оцінка відновлення перед сесією, тривалість плато в числах. clinical-safety: NOT_REQUIRED.
- `README.md` — додано Editorial 2171–2180 «Зовнішній погляд: що бачить тренер, чого не бачить атлет» до роадмапу (posts 2171–2180, 10 topics proposed) · v7.58.0.

### Changed
- `blog.html` — post-2125 card додана вище post-2124.
- `README.md` — post-2125 → draft · v7.58.0.
- `STATUS.md` — Editorial 2121–2130 IN PROGRESS 5/10; NEXT: post-2126.
- `VERSION` → 7.58.0

---

## [7.57.0] — 2026-06-22

### Added
- `content/post-2124-autoregulation-vs-planned-progression.md` — Editorial 2121–2130 post 4/10. Авторегуляція vs. планова прогресія: коли відходити від програми, а коли — ні. Термінологічне розмежування: планова прогресія — рішення прийняте до сесії, авторегуляція — рішення в реальному часі за попередньо визначеними критеріями. Три патерни self-coached помилок: хронічний escape hatch (авторегуляція як виправдання), авторегуляція без критеріїв («слухаю тіло» без визначених порогів), планова прогресія як dogma (ігнорування реальних сигналів). Реальні сигнали для відступу: технічний колапс (не дрейф — колапс), RPE-відхилення ≥1.5 балів від цільового після першого робочого підходу, об'єктивний маркер зниженої готовності ідентифікований до сесії, гострий больовий сигнал. Псевдосигнали: перший підхід важкий суб'єктивно, «не відчуваю сьогодні», залишковий DOMS, харчування попереднього дня, вага «виглядає важкою». Протокол «зупинись на один підхід»: Крок 0 (за годину) → Крок 1 (перший підхід за планом) → Крок 2 (RPE-оцінка після першого підходу, корекція -5–7.5% при відхиленні ≥1.5) → Крок 3 (оцінка після другого підходу, скорочення обсягу при повторному відхиленні) → Крок 4 (пост-сесійний запис). Критерій системності: корекції у >20–25% сесій блоку = проблема в плані, не в днях. Авторегуляція в ранній фазі освоєння нового рухового патерну: технічний стандарт, а не RPE-навантаження. Мінімальний набір записів для виходу за межі пам'яті. Принцип стійкості: послідовна середня програма > оптимальна програма з частими відступами. clinical-safety: NOT_REQUIRED.

### Changed
- `blog.html` — post-2124 card додана вище post-2123.
- `README.md` — post-2123 і post-2124 → draft · v7.57.0.
- `STATUS.md` — Editorial 2121–2130 IN PROGRESS 4/10; NEXT: post-2125.
- `VERSION` → 7.57.0

---

## [7.56.0] — 2026-06-22

### Added
- `content/post-2123-uncomfortable-question-protocol.md` — Editorial 2121–2130 post 3/10. Протокол «незручного питання»: як поставити собі питання тренера раз на блок. Чотири типи питань тренера, яких атлет уникає: правильність задачі, правильність метрики, альтернативні пояснення, ігноровані дані. Механізм confirmation bias у self-coaching: психологічна інвестиція в план створює системний фільтр підтвердження. Протокол: 25–35 хв, раз на блок, 7 питань із вимогою специфічних відповідей (мінімум 3 речення). Питання 1: чи правильну проблему вирішував блок. Питання 2: чи «поліпшення» є реальним або артефактом вимірювання. Питання 3: які аномалії пояснені зовнішнім фактором — і правомірно чи ні. Питання 4: що пропустив або систематично уникав. Питання 5: техніка поліпшилась чи деградувала паралельно зі зростанням навантаження. Питання 6: чи план відповідав реальному, а не ідеальному тижню. Питання 7: що б ти змінив, дивлячись на цей журнал як тренер чужого атлета. Критерії якості відповідей: специфічність, falsifiability, наявність дискомфорту. Три типи висновків і відповідні дії. Мінімальна 3-питальна версія. clinical-safety: NOT_REQUIRED.
- `README.md` — додано Editorial 2161–2170 «Довгостроковий розвиток self-coached атлета: від блоку до року» до роадмапу · v7.56.0.

### Changed
- `blog.html` — post-2123 card додана вище post-2122.
- `VERSION` → 7.56.0

---

## [7.55.0] — 2026-06-22

### Added
- `content/post-2122-feeling-vs-measuring-systematic-error.md` — Editorial 2121–2130 post 2/10. Системна помилка #1 self-coached атлета: плутати «відчуваю» і «вимірюю». Три типи даних у тренувальному записі: об'єктивне навантаження (вага×підходи×повторення — єдине, що порівнюється напряму між сесіями), структурована суб'єктивна оцінка (RPE/RIR — вимірює сприйняття зусилля в поточних умовах, не навантаження як таке), глобальна нарація стану (контекст для аномалій, не критерій рішення). Три патерни помилки: «здається легко» як хибний сигнал до збільшення, «важко» як хибний сигнал до зниження, ретроспективна реінтерпретація нерозмежованих записів. Механіка RPE: кумулятивний ефект втоми, калібрування досвідом, обмеженість міжсесійного порівняння без контексту. Коли суб'єктивне є валідним: гострий незвичний дискомфорт, системне зниження мотивації ≥7 днів, тенденція RPE вищій від планового при незмінному навантаженні. Практична триблочна структура запису (A: навантаження; B: RPE/RIR/техніка; C: контекст). Тест «вимірюєш чи відчуваєш?» перед кожним рішенням. clinical-safety: NOT_REQUIRED.

### Changed
- `blog.html` — post-2122 card додана вище post-2121.
- `VERSION` → 7.55.0

---

## [7.54.0] — 2026-06-22

### Added
- `content/post-2121-training-decisions-without-coach.md` — Editorial 2121–2130 post 1/10. Як приймати тренувальні рішення без тренера: структура замість інтуїції. Чотири типи рішень: сесійні (усуваємо через правила до сесії), мікроциклові (тріколор-протокол за HRV і суб'єктивною готовністю), блокові (п'ять структурованих питань огляду), стратегічні (квартальний горизонт і зворотний шлях). Чому інтуїція ненадійна: ефект поточного стану, confirmation bias, narrative fallacy, короткий горизонт. Що тренер реально робить (три функції) і як відтворити без зовнішньої людини. Мінімальна система (80% ефекту): план до сесії — 3 хв; нотатка після — 2 хв; тижневий огляд — 10 хв; блоковий огляд — 30 хв. Де структура не допомагає: технічні рішення 3D-проблема, діагностика нових патернів, два блоки невирішеної проблеми. clinical-safety: NOT_REQUIRED.

### Changed
- `blog.html` — post-2121 card додана вище post-2120.
- `README.md` — post-2121 → draft · v7.54.0; Editorial 2151–2160 (методологічна грамотність self-coached атлета) proposed · v7.54.0.
- `STATUS.md` — post-2121 details added; Editorial 2151–2160 proposed.
- `VERSION` → 7.54.0

---

## [7.53.0] — 2026-06-22

### Added
- `docs/INCIDENT_RESPONSE.md R-34` — Runbook 34: `turh_retention_purge` (job 32) stale recovery. Closes the documentation gap identified in OBSERVABILITY.md §12.6 — job 32 was the sole pg_cron job with PagerDuty P1 routing but no INCIDENT_RESPONSE cross-reference. R-34 covers: trigger matrix (AL-CRON-STALE-032 P1 → P0 escalation), two-stage severity classification (P1 = monitoring gap / no personal data breach due to safety gate; P0 = both purge AND Erasure Worker failed), immediate actions, five root cause hypotheses (H1 job deleted, H2 schema change, H3 permission revoked, H4 lock contention, H5 Supabase outage), scope queries R-34-C1 and R-34-C1b, PAM-elevated manual pseudonymisation fallback (Step 2c), DEC-030 chain spec (TURH-PURGE-CHAIN-01 invariant: `system.turh_purge_failure_declared` must precede `system.turh_purge_restored`), evidence preservation (TURH-CRON-E-001/002/003), SOC 2 TSC mapping (CC6.3/CC4.1/CC7.2/P4.1), and post-incident controls. Privacy floor enforced throughout: all DEC-030 events carry only operational metadata — no `user_id`, no `tenant_id`, no GDPR Art. 9 data.

### Changed
- `docs/INCIDENT_RESPONSE.md` — v3.1 → v3.2; R-34 appended after R-33.
- `docs/OBSERVABILITY.md` — v4.9.2 → v4.9.3; §12.6 job 32 cross-ref column updated with `INCIDENT_RESPONSE R-34 (job 32 stale recovery runbook — §R-34.5)`.
- `docs/AUDIT_LOG_SCHEMA.md` — v2.27 → v2.29; two new DEC-030 events registered: `system.turh_purge_failure_declared` (HIGH/7yr, TURH-PURGE-CHAIN-01 anchor) and `system.turh_purge_restored` (STANDARD/3yr).
- `docs/SOC2_READINESS.md` — v3.25.1 → v3.25.2; §79.4 three new evidence rows (TURH-CRON-E-001/002/003), §80.3 `turh-purge/` subfolder added, §80.4 Vanta mirror list updated.
- `VERSION` → 7.53.0

---

## [7.52.0] — 2026-06-22

### Added
- `content/post-2119-when-self-coaching-is-not-enough.md` — Editorial 2111–2120 post 9/10. Коли self-coaching вже недостатньо: сигнали і що з ними робити. Core argument: self-coaching має структурні межі (не дисциплінарні), їх розпізнавання — частина компетентності. Чотири переваги self-coaching: контекстуальна повнота, відсутність соціально-бажаних фільтрів, безперервність спостереження, швидкість рішень. Проблема single-observer системи: confirmation bias знижується структурними інструментами але не до нуля. Чотири сигнали структурного ліміту: (1) технічна деградація у ключовому русі, діагностована але не вирішена після 3+ підходів за один блок — причини: 3D-проблема при 2D-спостереженні, proprioceptive miss, межа методологічного знання; (2) стагнація основного показника при підтвердженій adherence і трьох виключених стандартних причинах — 3+ блоки; (3) змагальна підготовка з критичним горизонтом (піковка, вибір спроб), де ціна помилки суттєво вища; (4) нова складна рухова навичка (важка атлетика, спеціалізовані техніки) без прогресу в якості після 3–4 тижнів цілеспрямованої роботи. Форми зовнішньої допомоги — не лише «постійний тренер»: одиночна технічна консультація (для сигналу 1), програмний огляд раз на блок/рік (для сигналу 2), піковий блок 8–12 тижнів (для сигналу 3), peer review як перший крок. Три елементи ефективного питання до консультанта: конкретна проблема + що виключено + що потрібно від консультанта. clinical-safety: NOT_REQUIRED.
- `content/post-2120-self-coaching-architecture-synthesis.md` — Editorial 2111–2120 post 10/10 · BLOCK COMPLETE. Синтез блоку 2111–2120: архітектура self-coaching як практика. Вихідна рамка: self-coaching = виконання чотирьох конкретних функцій (спостереження / діагностика / рішення / підзвітність) без зовнішньої людини. Шість компонентів архітектури: (1) система спостереження — журнал первинний, числові маркери, контекстний маркер, систематичне відео; (2) структуровані питання трьох рівнів — сесія (1 хв) / тиждень (5 хв) / блок (30 хв) + «незручне питання» щоблоку; (3) захист від п'яти когнітивних спотворень через структурні рішення, не дисципліну: гіпотезний формат знижує confirmation bias і narrative fallacy; числовий критерій — outcome bias і present bias; фіксований горизонт — sunk cost; (4) контекстне коригування = об'єктивний сигнал (tricolor protocol), ≠ систематичне пом'якшення; (5) гіпотезний формат для кожної зміни: одна змінна / числовий аутком / фіксований горизонт / умови валідності; (6) межі системи і модульне зовнішнє залучення. Взаємодія компонентів: спостереження → питання → захист від спотворень; гіпотеза → спостереження; контекстне коригування → гіпотеза (компрометація); межі → питання. Victor = інфраструктура: пам'ять без ретроспективної редакції, структурні питання у визначений момент, патерн-розпізнавання у часових рядах, гіпотезний каркас без підміни рішень. Десять операційних принципів: журнал первинний / питання за розкладом / «незручне» обов'язкове / спотворення знижуються структурою / коригування ≠ пом'якшення / кожна зміна — тест / межі = компетентність / допомога модульна / архітектура спрощена. clinical-safety: NOT_REQUIRED.

### Changed
- `blog.html` — post-2119 і post-2120 cards додані (вище post-2111).
- `README.md` — post-2119 → draft · v7.52.0; post-2120 → draft · v7.52.0.
- `STATUS.md` — Editorial 2111–2120 COMPLETE 10/10 · v7.52.0; post-2119 і post-2120 details added.
- `VERSION` → 7.52.0

---

## [7.51.0] — 2026-06-21

### Added
- `content/post-2118-training-hypothesis-self-testing.md` — Editorial 2111–2120 post 8/10. Як формулювати тренувальну гіпотезу і перевіряти її самостійно. Core argument: self-coached атлет постійно проводить тести, більшість без явного формулювання — немає базового рівня, визначеного кінця, контрольних умов; Kahneman & Lovallo (1993) planning fallacy: плануємо в ізольованих умовах, оцінюємо в шумному контексті. Чотири компоненти валідної гіпотези: (1) одна змінна явно сформульована до зміни — «збільшую кількість робочих сетів у жимі з 3 до 5», не «спробую більш інтенсивно»; (2) числовий аутком з мінімально значущим порогом — «e1RM +5 кг за 6 тижнів», не «подивлюсь на прогрес»; (3) фіксований часовий горизонт — кінець тесту записаний до початку; (4) умови валідності — що залишається константним і критерій скомпрометованості. Скомпрометований тест — попередні дані, не провал: записуєш причину і повторюєш за чистих умов. Різниця між тестом і програмною логікою: загальновизнаний принцип = логіка; твердження «для мене краще» або вибір між альтернативами = кандидат на гіпотезний формат. Типові помилки: ретроспективна гіпотеза (confirmation bias вже виконав роботу) / занадто великий аутком / паралельні зміни / відсутність запису умов. Мінімальний 5-рядковий формат: зміна / очікую / базовий рівень / зафіксовані умови / кінець тесту. Victor: методологічний каркас без підміни рішень. clinical-safety: NOT_REQUIRED.

### Changed
- `blog.html` — post-2118 card додана (вище post-2117).
- `README.md` — post-2118 → draft · v7.51.0; Editorial 2141–2150 proposed (прогрес без тренера: вимірювання, інтерпретація, захист здобутків — 10 тем).
- `STATUS.md` — Editorial 2111–2120 updated: 8/10 · v7.51.0; post-2118 details added; Editorial 2141–2150 proposed.
- `VERSION` → 7.51.0

---

## [7.50.0] — 2026-06-21

### Added
- `content/post-2115-contextual-adjustment-without-feeling-autoregulation.md` — Editorial 2111–2120 post 6/10. Контекстне коригування: як врахувати стан без авторегуляції «на відчуття». Дві системні відмови: жорстке дотримання плану ігнорує відновлювальний ресурс; авторегуляція «на відчуття» = ненадійний сигнал залежний від мотивації (Day et al. 2004: авторегуляція перевершує фіксований план лише при структурованих RPE-протоколах). П'ять вимірюваних контекстних змінних: тривалість сну (год), відновлення 1–5, зовнішнє навантаження (бінарно), попередня сесія, час від останньої. Триколірна класифікація до початку тренування: зелена (план без змін), жовта (одна змінна знижена — допоміжна робота -20–30%, RPE -0.5), червона (дві+ знижені — 60–70% навантаження або 30 хв RPE ≤ 6). RPE у системі: Zourdos et al. (2016) надійність при каліброваній шкалі; для self-coached — порівняння при стандартному навантаженні в динаміці. Три помилки впровадження: класифікація постфактум, надто часті «червоні», три «жовті» поспіль як нерозпізнаний патерн. Відмінність між коригуванням і відмовою від роботи: документований критерій vs. постфактум обґрунтування. clinical-safety: NOT_REQUIRED.
- `content/post-2116-post-block-reflection-without-coach.md` — Editorial 2111–2120 post 7/10. Рефлексія після блоку: що і як аналізувати, коли немає тренера. Чому блоковий огляд не відбувається сам по собі: момент невизначений (немає чіткого кінця блоку), огляд не запланований, немає шаблону. Сім точок огляду: прогресія e1RM на ключових вправах (число), adherence (відсоток виконаних сесій і підходів), тренд RPE при стандартному навантаженні, що прогресувало / стагнувало, контекстні відхилення блоку, суб'єктивна оцінка 1–10, відкрите питання «що не змінилось, хоча мало б». Обов'язковий вихід: три рішення до кінця сесії (що змінюється в програмі; що робиться з відкритим питанням; контекстна ціль на блок). Тривалість: 25–35 хвилин письмово. Три помилки: огляд без числових даних, відповіді без рішень, перехід в пошук нової програми. Системний ефект 12 оглядів на рік: часовий ряд з патернами, невидимими в рамках одного блоку. clinical-safety: NOT_REQUIRED.

### Changed
- `blog.html` — post-2116 і post-2115 cards додані між post-2117 і post-2114.
- `README.md` — post-2115 і post-2116 → draft · v7.50.0.
- `STATUS.md` — Editorial 2111–2120 updated: 7/10 · v7.50.0.
- `VERSION` → 7.50.0

---

## [7.49.0] — 2026-06-21

### Added
- `content/post-2117-five-cognitive-biases-self-coached-training.md` — Editorial 2111–2120 post 5/10. П'ять когнітивних спотворень у тренуванні: де логіка self-coached атлета ламається передбачувано. Core argument: self-coached атлет не робить нераціональних рішень — він застосовує нормальні еволюційно обґрунтовані механізми мислення до ситуацій, для яких вони не оптимізовані; тренер позбавлений не спотворень, а причетності. П'ять механізмів: (1) Confirmation bias (Wason 1960) — після вибору програми пошук доказів спрямований на підтвердження; маніфестація: пам'ятаєш хороші сесії, ігноруєш стагнацію; структурне рішення — числовий критерій до початку блоку, не наратив. (2) Present bias (Kahneman peak-end rule) — поточний стан у момент оцінки домінує над трендом; маніфестація: оцінка блоку різна в залежності від дня тижня, коли її робиш; рішення — фіксований день огляду за числами журналу. (3) Sunk cost fallacy (Arkes & Blumer 1985) — минулий вклад корумпує рішення про майбутнє; маніфестація: «8 тижнів на програмі — не можна міняти»; рішення — явне питання «я б обрав це зараз з нуля?». (4) Asymmetric attribution (Ross 1977 self-serving bias) — успіх → програма; невдача → зовнішні умови; маніфестація: прогрес приписується плану, стагнація — сну/стресу; рішення — перевірка симетрії: ті самі умови → однакова логіка атрибуції. (5) Planning fallacy (Kahneman & Lovallo 1993) — план уявляє ідеальні умови, оцінка відбувається по реально виконаному; маніфестація: 19 з 24 запланованих сесій = «програма не спрацювала»; рішення — мінімальний і оптимальний варіант плану, оцінка по обом. Взаємодія: спотворення підсилюють одне одного — confirmation bias + sunk cost = захисний патерн, що тримає на неефективній програмі. Wilson & Brekke (1994): знання про механізм не відключає його в реальному часі; допомагають структури, не усвідомленість. Victor-angle: зовнішній агент без причетності обходить всі п'ять структурно — e1RM-тренд замість наративу, фіксований огляд, симетричне питання, відсоток виконання плану. clinical-safety: NOT_REQUIRED. sports-scientist review: not required (cognitive psychology + decision science).

### Changed
- `blog.html` — post-2117 card added (above post-2114); post-2114 card added (missing from v7.48.0, above post-2113).
- `README.md` — post-2117 → draft · v7.49.0; Editorial 2111–2120 progress 4/10 → 5/10; Editorial 2131–2140 block proposed (10 теми: адаптація тренування до реального життя) · v7.49.0.
- `STATUS.md` — Editorial 2111–2120 updated: 5/10 · v7.49.0; post-2117 details added.
- `VERSION` → 7.49.0

---

## [7.48.0] — 2026-06-21

### Added
- `content/post-2114-uncomfortable-question-self-coached.md` — Editorial 2111–2120 post 4/10. Незручне питання: як self-coached атлет ставить сам собі питання тренера. Core argument: структурна проблема самопитання — людина є одночасно прокурором і підзахисним, тому мотивоване міркування (Kunda 1990) систематично спрямовує пошук доказів на захист уже прийнятих рішень. Тренер ставить незручне питання без конфлікту інтересів — self-coached атлет не може бути «ззовні», але може будувати процедуру, що відтворює цей стан. Три характеристики ефективного питання: конкретний критерій (не «є прогрес», а «e1RM зріс на Х за блок»), фіксований розклад (не «коли відчую потребу»), відповідь вимагає рішення (не «продовжу спостерігати»). Три категорії: (1) діагностика прогресу — числові критерії per lift; (2) виклик плану — «якщо б починав зараз з нуля — обрав би той самий план?»; (3) системна чесність — «яке рішення я уникаю і чому?». Процедура: розклад + письмова відповідь + обов'язкове рішення по кожній відповіді. Прийом «зовнішнього ревізора»: Liberman & Trope (1998) construal level theory — психологічна дистанція змінює тип мислення від конкретного/реактивного до абстрактного/принципового; практично: писати про власне тренування в третій особі при першому читанні. Межа: відповіді без наслідків = ритуал; питання без даних журналу = підтверджувальний наратив; рефлексія без виходу в рішення = румінація. Victor: класифікуючі питання з числами в ключових точках (кінець блоку, стагнація N днів, відхилення від плану) — не мотиваційний тон, а запит на класифікацію ситуації і вибір дії. clinical-safety: NOT_REQUIRED. sports-scientist review: pending.

### Changed
- `README.md` — post-2114 → draft · v7.48.0; Editorial 2111–2120 progress 3/10 → 4/10.
- `STATUS.md` — Editorial 2111–2120 updated: 4/10 · v7.48.0; post-2114 details added.
- `VERSION` → 7.48.0

---

## [7.47.1] — 2026-06-21

### Added
- `docs/AUDIT_LOG_SCHEMA.md` — `### Enterprise Contract Renewal Monitoring events (DEC-030 HMAC-chained · OBSERVABILITY §51 · CC4.1/CC2.2/A1.1)`: new section registering two DEC-030 events for the RENEW-NOTICE-01 pg_cron job 39 monitoring control. (1) `enterprise.renewal_notice_overdue` HIGH/7yr: per-tenant alert event emitted when RENEW-NOTICE-01 SQL detects an active contract in the 85–95-day notice window with no `enterprise.renewal_notice_sent` on file; fires AL-RENEW-01 P1 PagerDuty `form-enterprise`; Zod v2 `RenewalNoticeOverduePayload`; CC4.1/CC2.2 auditor narratives; REN-E-002 SOC 2 mapping. (2) `system.renewal_notice_check_passed` LOW/1yr: operational all-clear health signal emitted when zero gaps detected; no `tenant_id` — privacy floor; 26h absence triggers `system.cron_job_stale` + INCIDENT_RESPONSE R-28; Zod v2 `RenewalNoticeCheckPassedPayload`; REN-OBS-E-001 A1.1 mapping. Two retention table rows added. **Closes `docs/OBSERVABILITY.md §51.7` item 1 (P0/M11).** AUDIT_LOG_SCHEMA.md v2.27 → v2.28.

### Changed
- `docs/OBSERVABILITY.md` — §51.7 item 1 marked `[x] Done (2026-06-21, AUDIT_LOG_SCHEMA.md v2.28)`; §51.4.1 + §51.4.2 registration obligation notes updated to 🟢 Done. Header v4.9.1 → v4.9.2 (v4.8.4 patch note added to version history).
- `VERSION` → 7.47.1

---

## [7.47.0] — 2026-06-21

### Added
- `content/post-2113-training-log-accurate-memory.md` — Editorial 2111–2120 post 3/10. Точна пам'ять vs. спотворена: чому тренувальний журнал вирішує більше ніж здається. Core argument: людська пам'ять систематично спотворює тренувальні дані — тренер не пам'ятає «відчуття» підйому, він тримає числа; Meehl (1954): статистичне рішення перевершує клінічне в 19/20 досліджень, тому що числа не редагуються ретроспективно. Три механізми спотворення: ретроспективне згладжування зусиль (Redelmeier & Kahneman 1996 — пікові моменти і кінцівка, не інтегральна інтенсивність), усереднення по поточному стану, confirmation bias у самооцінці прогресу. Три практичних наслідки: переоцінка готовності до навантаження, помилкова діагностика плато, помилкова атрибуція результатів програми. Три шари мінімально функціонального журналу: (1) числовий запис — 90 сек: вага×повт×підходи + RPE + одне речення контексту; (2) тижневий огляд — 10 хв: тренд e1RM, RPE drift, відсоток виконання, одне конкретне рішення; (3) блоковий підсумок — 30 хв: порівняльна таблиця старт/фінал, три питання, стратегія. Ключова дистинкція: журнал без огляду — архів; огляд без рішення — ритуал. FORM angle: Victor агрегує дані і ставить діагностичні питання при відхиленнях — не мотивація, а системний аналіз. clinical-safety: NOT_REQUIRED. sports-scientist review: pending.

### Changed
- `blog.html` — post-2113 card added (above post-2112).
- `README.md` — post-2113 → draft · v7.47.0; Editorial 2121–2130 block proposed (10 тем: прийняття рішень self-coached атлета — методи, помилки, системи).
- `STATUS.md` — Editorial 2111–2120 updated 3/10 · v7.47.0.
- `VERSION` → 7.47.0

---

## [7.46.0] — 2026-06-21

### Added
- `content/post-2112-external-observability-system.md` — Editorial 2111–2120 post 2/10. Як побудувати систему зовнішньої спостережливості без тренера поруч. Core argument: self-observation is structurally limited — during a set, cognitive bandwidth is occupied by execution, leaving the observational channel overloaded or closed; Moran et al. (2012) show even trained athletes cannot accurately reproduce their kinematic parameters from internal sensation, and error grows with fatigue. Four-level observability system: (1) video — external eye for technique and movement pattern; shoot primary lifts first+last set, review same day with a specific question, store by date+exercise; (2) numerical log — external memory for progress dynamics; minimum: weight×reps×sets + RPE + one-sentence note; (3) structured self-review — weekly 10 min (e1RM trend, RPE drift, adherence %, one decision) + block 30 min (start/end table, three questions); (4) external check — at least once per block, partner or specialist. System vs. ritual: a system generates decisions — if a weekly review produces no concrete decision it's a ritual. Minimum viable version: 60s entry → 10 min weekly review → 30 min block summary. FORM angle: Victor treats video as pattern-over-time (not good/bad), auto-aggregates all log data, triggers structured questions at the right moment. Fitzsimons & Finkel (2011) on minimal public accountability raising self-assessment accuracy. clinical-safety: NOT_REQUIRED. sports-scientist review: pending.

### Changed
- `blog.html` — post-2112 card added (above post-2060, below post-2111).
- `README.md` — post-2112 → draft.
- `STATUS.md` — Editorial 2111–2120 updated 2/10 · v7.46.0; next priorities updated.
- `VERSION` → 7.46.0

---

## [7.45.0] — 2026-06-21

### Added
- `content/post-2111-coach-functions-self-coached-athlete.md` — Editorial 2111–2120 post 1/10 · NEW BLOCK. Чотири функції тренера, які self-coached атлет виконує сам. Core argument: a coach performs four specific cognitive functions — external observation (sees patterns outside your subjectivity), accurate memory (numbers not narrative — Meehl 1954: statistical prediction outperforms clinical judgment), the uncomfortable question (without emotional self-protection), contextual adjustment (plan adapted to real life, not ideal conditions). Without a coach these functions don't disappear — they go unperformed or performed poorly with no system. Replacements: video analysis for external eye; training log + monthly review + block summary for accurate memory; quarterly uncomfortable-question protocol (e1RM delta, RPE drift, adherence %, distanced-perspective question); three-zone system (green/yellow/red) for contextual adjustment. Fourth function: external accountability (Gollwitzer & Brandstätter 1997 — public commitment mechanism changes behavior). FORM angle: Victor designed around all four functions with explanation, not silent switching. clinical-safety: NOT_REQUIRED. sports-scientist review: pending.

### Changed
- `blog.html` — post-2111 card added (above post-2060).
- `README.md` — roadmap block 2111–2120 (Self-coaching як практика: структура, функції, рішення) added; post-2111 → draft.
- `STATUS.md` — Editorial 2111–2120 entry added (1/10 · v7.45.0); Next priorities updated; version header bumped to v7.45.0.
- `VERSION` — 7.44.0 → 7.45.0.

---

## [7.44.0] — 2026-06-21

### Added
- `content/post-2059-training-documentation-vs-analytics.md` — Editorial 2051–2060 post 9/10. Тренувальна документація vs. тренувальна аналітика: де проходить межа корисного. Core argument: documentation (recording what happened) and analytics (extracting meaning) are two distinct processes with different goals — confusing them or neglecting either creates systematic decision-making failure. Three failure modes: over-documentation (archive without use, control illusion, signal buried in noise); under-documentation (decisions based on systematically distorted memory — Meehl 1954 clinical vs. statistical prediction: even a simple structured log outperforms «overall impression»); tool without system (changing apps without changing review habit). Minimum effective log: weight × reps × sets + RPE + one technical note = 5–8 lines/session. Three analytics levels: weekly (5–10 min, tactical — did session match plan?); monthly/inter-block (20–30 min, tactical+strategic — e1RM trend, RPE drift, technical drift); quarterly (45–60 min, strategic — quarterly review, block planning). Data hierarchy: Level 1 = e1RM, RPE at standardized loads, technical quality (highest diagnostic value); Level 2 = weekly volume, session rating, rest duration; Level 3 = sleep, external stress, context (explains anomalies, doesn't build trends). Usefulness test: three concrete decisions influenced by the log in the last month. clinical-safety: NOT_REQUIRED. sports-scientist review: pending.
- `content/post-2060-editorial-block-synthesis-critical-thinking.md` — Editorial 2051–2060 post 10/10 · BLOCK SYNTHESIS. Синтез блоку 2051–2060: критичне мислення self-coached атлета. Block map table: all 10 posts with topic and key principle. The unifying thread: critical thinking in training = the ability to ask the right question before making a decision — each post contains one fundamental question the self-coached athlete often skips. Five synthesis principles: (1) diagnosis precedes decision — define what before deciding what to do; (2) time horizon determines visible reality — different decisions require different data windows; (3) progression is a system of variables — know which variable you're manipulating; (4) minimum effective decision beats complex correct one; (5) self-coaching requires explicit substitution of three coach functions (external eye, accurate memory, uncomfortable question). Three systemic traps: confirmation thinking (seeing progress where wanted); single-data-point decisions (one session ≠ trend); system management hypertrophy (infrastructure heavier than training). Decision reference table mapping 8 questions to the relevant posts. Preview: Editorial 2061–2070 — «Тренування в умовах реального дорослого атлета». clinical-safety: NOT_REQUIRED. sports-scientist review: pending.

### Changed
- `blog.html` — post-2060 and post-2059 cards added (above post-2058).
- `STATUS.md` — Editorial 2051–2060 updated: 8/10 · v7.43.0 → COMPLETE 10/10 · v7.44.0; post-2059 and post-2060 entries added; Next priorities updated.
- `VERSION` — 7.43.1 → 7.44.0.

---

## [7.43.1] — 2026-06-21

### Added
- `docs/AUDIT_LOG_SCHEMA.md` v2.26 → v2.27 — `system.scim_guard_cfg_cache_stale` (LOW/1yr) registered in §System. Closes `docs/SOC2_READINESS.md §98.5` item 1 (P1/M13). Advisory event emitted by `billing-event-relay` Cloudflare Worker when `SCIM_KV.delete` throws during BDG KV cache invalidation; 3600s TTL safety-net self-clears; compliance-officer reviews in GUARD-E-001 quarterly stale-advisory cross-check.

### Changed
- `docs/SOC2_READINESS.md` — §98.5 item 1 status updated to `[x] Done — AUDIT_LOG_SCHEMA.md v2.27, 2026-06-21`.
- `VERSION` — 7.43.0 → 7.43.1.

## [7.43.0] — 2026-06-21

### Added
- `content/post-2058-neuromuscular-adaptation-year-one.md` — Editorial 2051–2060 post 8/10. Нейром'язова адаптація в перший рік тренувань: чому приріст сили — не те, що думаєш. Core argument: strength gains in the first 6–12 weeks of training are primarily neural, not hypertrophic — and understanding this changes how to build a first-year program. Covers: Moritani & deVries (1979) seminal EMG study — force gains outpace muscle mass; Sale (1988) — neural adaptation dominant in first 6–12 weeks in untrained. Motor unit recruitment mechanics: Henneman size principle (1965), rate coding, intermuscular coordination, motor unit synchronization. Hakkinen et al. (1985): voluntary activation in untrained below potential max. Folland & Williams (2007) review: 60–90% of early strength gains attributed to neural mechanisms (4–8 wk). Beginner gains mechanism: neural learning from zero + motor learning rapid coordination gains + low baseline. Four mistakes that stall neural adaptation: (1) no intent for maximal velocity (Behm & Sale 1993 — intent matters); (2) excessive exercise variety — neural adaptation is movement-specific; (3) training to failure every set (Enoka 1988 — quality motor commands, not exhaustion); (4) ignoring movement pattern quality. Neural → hypertrophic transition at 6–12 weeks. Practical principles: frequency > volume early; linear progression = neural learning structure; RIR 2–3 for base movements; muscle memory is largely neural memory. Operational answer. clinical-safety: NOT_REQUIRED. sports-scientist review: pending.

### Changed
- `blog.html` — post-2058 card added (above post-2056).
- `STATUS.md` — Editorial 2051–2060 updated: 7/10 → 8/10 · v7.43.0; post-2058 entry added.
- `README.md` — roadmap 2101–2110 (від нейральної адаптації до системного мислення self-coached атлета) added as proposed.
- `VERSION` — 7.42.1 → 7.43.0.

## [7.42.1] — 2026-06-21

### Added
- `docs/INCIDENT_RESPONSE.md R-33` — BDG Override Expiry Sweep Failure runbook (job 34 `bdg_override_expiry_sweep`, `*/15 * * * *`, 20-min freshness window, DEC-066/migration 0079 SCIM guard safety-net). P1/P0 severity split, H1–H4 root cause hypotheses, PAM-elevated manual UPDATE procedure (P0), three DEC-030 events (BDG-SWEEP-CHAIN-01 anchor), four evidence artefacts (BDG-EXP-E-001–004), SOC 2 CC6.3/CC7.2/A1.2 mapping.

### Changed
- `docs/AUDIT_LOG_SCHEMA.md` v2.25 → v2.26 — Three BDG-SWEEP DEC-030 events added: `system.bdg_sweep_failure_declared` (HIGH/7yr, BDG-SWEEP-CHAIN-01 anchor), `system.bdg_sweep_override_cleared` (STANDARD/7yr, P0 only), `system.bdg_sweep_restored` (STANDARD/3yr).
- `docs/OBSERVABILITY.md` v4.8.5 → v4.9.1 — §12.6 job 34 `bdg_override_expiry_sweep` stale-consequence cross-ref extended with `INCIDENT_RESPONSE R-33 (job 34 stale recovery runbook — §R-33.5)`; v0.8 patch note added.
- `docs/SOC2_READINESS.md` v3.24.8 → v3.25.1 — §79.4 four BDG-EXP-E artefacts registered (BDG-EXP-E-001–004); §80.3 `bdg-sweep/` R2 subfolder added; §80.4 Vanta mirror list updated.

---

## [7.42.0] — 2026-06-21

### Added
- `content/post-2057-block-structure-recreational-athlete.md` — Editorial 2051–2060 post 7/10. Блокова структура для звичайного атлета: чи потрібна вона без змагань. Core argument: periodization structure is not a competitive-athlete privilege — it is a tool for managing interference effects and accumulating targeted adaptation. Covers: Issurin block periodization framework (2008) — accumulation / transmutation / realization phases; recreational athlete interpretation (realization block optional, first two universally applicable). Four concrete advantages for non-competitive athletes: psychological horizon, real accumulation, differentiated data, built-in recovery rhythm. Four situations where blocks don't add value: novice stage, very low volume, high schedule unpredictability. Practical two-phase cycle: accumulation (4–6 wk, 65–80% 1RM, 8–15 rep range) → intensification (3–4 wk, 80–90%+ 1RM, 3–6 rep range) → transition week. Three early-block-exit signals: volume-no-longer-feels-like-load; accumulated fatigue markers; external disruption > 10–14 days. DUP alternative: Rhea et al. (2002) LP vs. DUP comparison (28% vs. 14% bench, 36% vs. 18% squat over 12 wk); three-day DUP schema (strength / hypertrophy / endurance). Operational decision table: novice → linear; early intermediate → DUP or wave; intermediate+ → block; advanced → mandatory. clinical-safety: NOT_REQUIRED. sports-scientist review: pending.

### Changed
- `docs/INCIDENT_RESPONSE.md` — R-32.11 item 6 marked [x] Done (2026-06-21): `docs/SSO_SCIM_IMPLEMENTATION.md §36.6` item 9 added; post-incident controls row updated.
- `docs/SSO_SCIM_IMPLEMENTATION.md` — §36.6 item 9 added: cross-reference to INCIDENT_RESPONSE R-32 as operational recovery runbook for `caep_reregister_sweep` stale condition; two-severity matrix and §R-32.5 Steps 1–5 cited. Closes R-32.11 item 6.

## [7.41.0] — 2026-06-21

### Added
- `content/post-2056-exercise-change-when-and-why.md` — Editorial 2051–2060 post 6/10. Коли треба змінити вправу — і чому більшість робить це надто рано або надто пізно. Two failure modes: too early (novelty myth, boredom) vs. too late (real accommodation). SAID principle — Behm & Sale (1993), Morrissey et al. (1995): strength adaptations are movement-specific. Motor learning stages (Schmidt & Lee 2011): cognitive → associative → autonomous — hundreds of reps to reach autonomous. Repeated bout effect (Damas et al. 2018): DOMS reduction ≠ adaptation reduction. Regional hypertrophy and variety (Fonseca et al. 2014, Kassiano et al. 2022): argument for accessory selection, not constant rotation of primary lifts. Exercise variation ≠ exercise change — critical distinction. Three scenarios to change: adaptation ceiling (8–10+ weeks stagnation, all variables controlled), structural mismatch to current goal, persistent discomfort at specific anatomical position (not pain — that → doctor). Three questions before swapping. Journal-based decision table: load trend × RPE drift × technique stability × context. Practical algorithm: 6-step process from log review to decision. README: block 2091–2100 (Психологія прогресу та довгострокова мотивація) added as proposed. clinical-safety: NOT_REQUIRED. sports-scientist review: pending.

### Changed
- `blog.html` — post-2056 card added (above post-2055).
- `STATUS.md` — Editorial 2051–2060 updated to 6/10 · v7.41.0.
- `README.md` — post-2056 → draft · v7.41.0; block 2091–2100 proposed.

---

## [7.40.0] — 2026-06-21

### Added
- `content/post-2055-rest-intervals-timer-vs-readiness.md` — Editorial 2051–2060 post 5/10. Час відпочинку між сетами: таймер чи суб'єктивна готовність — що працює. PCr resynthesis kinetics: Hultman et al. (1967) + Greenhaff et al. (1994) — 50% at 30s, 87% at 90s, 97–99% at 3–4 min. Schoenfeld et al. (2016): 3 min vs. 1 min — significant superiority for strength and hypertrophy outcomes in trained men. de Salles et al. (2009) meta-analysis: 2–5 min rest outperforms <1 min for strength/hypertrophy. Subjective readiness failure modes: habituation to discomfort, psychological pressure to move, day-to-day recovery variability. Timer failure modes: ignores session-to-session variability, doesn't adapt to exercise demand, promotes mechanical execution. Operational filter: timer mandatory for strength (1–5RM, 85%+), standardization, and athletes who tend to rush; subjective readiness appropriate for technical work, experienced athletes, atypical conditions. Hybrid model: minimum timer floor (2 min) + maximum ceiling window (30–60 s post-beep). RPE delta test: first to third set at standardized 3-min rest — >2-point jump flags systemic recovery issue. clinical-safety: NOT_REQUIRED. sports-scientist review: pending.

### Changed
- `blog.html` — post-2055 card added (above post-2054).
- `STATUS.md` — Block 2051–2060 updated to 5/10.

---

## [7.39.0] — 2026-06-21

### Added
- `content/post-2054-training-journal-mirror-three-months.md` — Editorial 2051–2060 post 4/10. Тренувальний журнал як дзеркало: що він говорить через 3 місяці. Core distinction: real-time journal vs. retrospective journal at 90 days — different objects with different diagnostic value. Three patterns visible only at 3-month scale: (1) unconscious load wave — stochastic deload vs. discipline failure require 3-month view to distinguish; (2) RPE calibration drift — same number, different relative intensity across the block, detectable via anchor-point comparison; (3) technical drift under load — visible in retrospective video or technique-note comparison. Common retrospective errors: confirmation reading, single-point reaction vs. pattern, absolute vs. relative comparison. 5-step retrospective protocol: aggregate weekly data, three key questions (frequency, volume trend, wave), RPE anchor check, technical review, one specific conclusion. What the journal cannot tell you: pattern vs. cause — retrospective requires second step of hypothesis about context. Minimum logging system: weight/sets/reps + RPE per last working set + one-liner on session feel. clinical-safety: NOT_REQUIRED. sports-scientist review: pending.
- README.md — Roadmap 2081–2090 proposed (Editorial series: тренувальна система і довгострокове мислення).

### Changed
- `blog.html` — post-2054 card added (above post-2053).
- `STATUS.md` — Editorial 2051–2060 → 4/10 · v7.39.0.

## [7.38.0] — 2026-06-21

### Added
- `content/post-2052-progression-without-weight-increase-overload.md` — Editorial 2051–2060 post 2/10. Прогресія без числового збільшення: що ще рахується як overload. Three adaptation mechanisms (Schoenfeld 2010): mechanical tension (primary, mTOR pathway), metabolic stress, muscle damage. Seven forms of non-weight progression: double progression (reps→weight), density (shorter rest), ROM expansion (Schoenfeld & Grgic 2020 lengthened partials), tempo manipulation (TUT, eccentric), exercise variation, reduced external stabilisation, bilateral→unilateral transition. Self-coached error: logging only weight/reps misses volume, density, ROM progress. Overload 3×3 matrix: dimensions (volume/intensity/density) × actions (increase/decrease/shift) — change one dimension per mesocycle. Decision algorithm: when weight IS the right variable vs. when another dimension is more appropriate (RPE, technique stability, cycle phase, recovery state). clinical-safety: NOT_REQUIRED. sports-scientist review: pending.
- `content/post-2053-technical-plateau-technique-replaces-weight.md` — Editorial 2051–2060 post 3/10. Технічне плато — коли поліпшення техніки заміщує підняття ваги. Distinction: strength plateau (exhausted adaptive potential, solution = program change) vs technique plateau (motor pattern change, temporary number drop, solution = time + quality reps). Three concrete mechanics examples: quarter-squat→full ROM, restricted ROM bench→full, lumbar-flexion deadlift→hip hinge — all reassessments of baseline at correct mechanics. Motor learning curve (Schmidt & Lee, 2011): cognitive → associative → autonomous stage; 300–500 quality reps at 60–75% 1RM minimum. Practical calculation: 2 sessions/week × 3×6 reps = 36 reps/week → minimum 8–16 weeks to recover previous numbers at new mechanics. Common failure loop: numbers drop → change program → cancel technique work → old number returns via old pattern → false confirmation. When technique improvement genuinely costs strength (rare): anatomical variation (anteversion, limb proportion ratio) — rule: deviation identical at 40% and 90% = anatomy; deviation appears under load = compensation. Decision tree: three branches with diagnostic criteria and specific solutions. Four journal additions when changing technique: date, motor-sensation quality score (1–5), video every 2–3 weeks, RPE deviation tracking. clinical-safety: NOT_REQUIRED. sports-scientist review: pending.

### Changed
- `blog.html` — post-2052 and post-2053 cards added (above post-2051).
- `STATUS.md` — Editorial 2051–2060 → 3/10 · v7.38.0.

## [7.37.1] — 2026-06-21

### Added
- `docs/INCIDENT_RESPONSE.md` — R-32: CAEP Stream Re-Registration Sweep Failure recovery runbook for pg_cron job 37 (`caep_reregister_sweep`, `*/5 * * * *`, 6-min freshness window). Two-severity matrix: P1 (stale, `caep_status_error_count = 0` — monitoring gap only; CAEP streams in-flight but re-registration sweep halted), P0 (stale + `caep_status_error_count > 0` — active CAEP stream integrity breach; manual `POST /internal/v1/caep/reregister` PAM-elevated call required). Five root cause hypotheses H1–H5 (job deleted, pg_net degraded, Worker down, IdP rate-limit, Supabase outage). Three DEC-030 events: `system.caep_sweep_failure_declared` (HIGH/7yr, CAEP-SWEEP-CHAIN-01 anchor), `system.caep_sweep_manual_reregister_completed` (STANDARD/7yr, P0 only, PAM-elevated API call), `system.caep_sweep_restored` (STANDARD/3yr, root cause H1–H5). Four evidence artefacts: CAEP-SWEEP-E-001/002/003/004. Template CAEP-INT-01 internal memo (no customer-facing comms). SOC 2 mapping: CC6.3, CC7.2, CC7.3, A1.1.

### Changed
- `docs/AUDIT_LOG_SCHEMA.md` — v2.25: three new DEC-030 System events registered after `system.audit_log_purge_restored`: `system.caep_sweep_failure_declared` (HIGH/7yr, CAEP-SWEEP-CHAIN-01 anchor — IC emits at T+0), `system.caep_sweep_manual_reregister_completed` (STANDARD/7yr, P0 only — PAM-elevated `POST /internal/v1/caep/reregister`), `system.caep_sweep_restored` (STANDARD/3yr — devops-lead restoration confirmation, H1–H5 root_cause_category). CAEP-SWEEP-CHAIN-01 ordering invariant noted; HTTP 422 on violation.
- `docs/OBSERVABILITY.md` — v4.8.5: §12.6 job 37 `caep_reregister_sweep` registry entry updated — stale-consequence cross-ref column appended with `INCIDENT_RESPONSE R-32 (job 37 stale recovery runbook — §R-32.5)`. v0.7 patch note added.
- `docs/SOC2_READINESS.md` — v3.24.8: §79.4 four CAEP-SWEEP artefacts registered (CAEP-SWEEP-E-001/002/003/004) after AUDIT-PURGE-COMP-E-001; §80.3 `caep-sweep/` subfolder added after `audit-log-purge/`; §80.4 Vanta mirror list updated with CAEP-SWEEP-E-001–004 (R-32.8 incident-triggered).
- `docs/INCIDENT_RESPONSE.md` — v3.1: R-32 appended; header v3.0 → v3.1.
- `STATUS.md` — VERSION → 7.37.1.

## [7.37.0] — 2026-06-21

### Added
- `content/post-2051-cost-of-mistakes-self-coached.md` — Editorial block 2051–2060 post 1/10. Ціна помилки у self-coached тренуванні: які помилки зворотні, а які — ні. Three-class taxonomy: Class A (cheap/reversible, 1–4 weeks); Class B (conditionally reversible, depends on reaction time — 4 weeks to 18 months); Class C (hard/irreversible — entrenched motor patterns, structural injuries from chronic incorrect loading). Transition mechanism A→B→C: ignoring signals turns cheap mistakes expensive. Three most common self-coached transitions: volume without reaction, technical drift under fatigue, pain-signal without response. Asymmetry of risk: excessive caution is also a Class B error. Three-question framework before major decisions: cost if right / cost if wrong / how fast will you know. clinical-safety: NOT_REQUIRED. sports-scientist review: pending.

### Changed
- `blog.html` — post-2051 card added (top of grid).
- `README.md` — post-2051 → draft · v7.37.0; roadmap 2071–2080 (діагностичне мислення self-coached атлета) added as proposed.
- `STATUS.md` — Editorial 2051–2060 → 1/10 · v7.37.0; current version → v7.37.0.

## [7.36.1] — 2026-06-21

### Added
- `docs/INCIDENT_RESPONSE.md` — R-31: GDPR Audit Log Retention Purge Failure recovery runbook for pg_cron job 27 (`audit_log_retention_purge`, `0 3 1 * *`, 48h freshness window). Three-severity matrix: P1 (stale, zero eligible rows — monitoring gap; expected until ~2033), P0 (stale + eligible rows — GDPR Art. 5(1)(e) overcollection breach; manual purge required), P0-chain (H6 chain verification failure — `emit-audit-event` HTTP 422 → DELETE intentionally blocked; R-05 co-activation required). Six root cause hypotheses H1–H6. Unique H6: self-referential DEC-030 ordering constraint requires `data.audit_log_purge_completed` emitted HTTP 200 confirmed BEFORE DELETE. Three DEC-030 events: `system.audit_log_purge_failure_declared` (HIGH/7yr), `system.audit_log_purge_manual_run_completed` (STANDARD/7yr, `dec030_ordering_respected: z.literal(true)` mandatory), `system.audit_log_purge_restored` (STANDARD/3yr, `r05_activated`). Five evidence artefacts: AUDIT-PURGE-E-001/002/003/004/AUDIT-PURGE-COMP-E-001. Template AUDIT-INT-01 internal memo.

### Changed
- `docs/AUDIT_LOG_SCHEMA.md` — v2.24: three new DEC-030 events registered in §System after `system.purge_cron_restored`: `system.audit_log_purge_failure_declared` (HIGH/7yr), `system.audit_log_purge_manual_run_completed` (STANDARD/7yr, `dec030_ordering_respected: z.literal(true)`), `system.audit_log_purge_restored` (STANDARD/3yr, H1–H6 root_cause_category, `r05_activated`). AUDIT-PURGE-CHAIN-01 ordering invariant noted.
- `docs/OBSERVABILITY.md` — v4.8.4: §12.6 job 27 `audit_log_retention_purge` registry entry updated — stale consequence column includes `INCIDENT_RESPONSE R-31 (job 27 stale recovery runbook — §R-31.5)`.
- `docs/SOC2_READINESS.md` — v3.24.7: §79.4 five AUDIT-PURGE artefacts registered (AUDIT-PURGE-E-001/002/003/004/AUDIT-PURGE-COMP-E-001); §80.3 `audit-log-purge/` subfolder added; §80.4 Vanta mirror list updated.
- `docs/INCIDENT_RESPONSE.md` — v3.0: R-31 appended; header v2.9 → v3.0.
- `STATUS.md` — VERSION → 7.36.1.

## [7.36.0] — 2026-06-21

### Added
- `content/post-2035-ai-coach-vs-gpt-wrapper.md` — Editorial block 2031–2040 post 5/10. AI-coach і GPT-wrapper: як відрізнити продукт від красиво упакованого автозаповнення. Three requirements for a real AI-coach: persistent athlete state, longitudinal response model, differentiated context reaction. Three practical tests: memory test, context test, prediction test. Why GPT-wrappers answer context-independent questions well but context-dependent questions dangerously. What to expect and not expect from AI training tools. clinical-safety: NOT_REQUIRED. sports-scientist review: NOT_REQUIRED.
- `content/post-2036-readiness-scores-dashboard-dependency.md` — Editorial block 2031–2040 post 6/10. Readiness score і dashboard dependency: коли цифра починає керувати тобою. What readiness scores actually measure (proxies: HRV, RHR, skin temperature). Four dashboard-dependency mechanisms: external locus of control, post-rationalisation, anxiety without data, over-caution at low scores. Where readiness is genuinely useful: systemic trends, personal calibration, sharp deviation detection. Protocol: assess own state before app; score as Bayesian prior, subjective state as update; one tracker minimum 3 months; quarterly week without device. Hammert et al. (2024) HRV-guided training. clinical-safety: NOT_REQUIRED. sports-scientist review: pending.
- `content/post-2037-tracking-fatigue-vs-output.md` — Editorial block 2031–2040 post 7/10. Відстеження втоми і відстеження результату: чому це різні речі і чому треба обидві. Output tracking answers "is the system progressing?"; fatigue tracking answers "what state is the system in?". Six-scenario divergence matrix. Three metrics each for output and fatigue. Banister et al. (1975) fitness–fatigue model. clinical-safety: NOT_REQUIRED. sports-scientist review: pending.
- `content/post-2038-when-technology-is-the-obstacle.md` — Editorial block 2031–2040 post 8/10. Коли технологія заважає: ознаки, що інструмент став милицею. Four signs of tech dependency. Mechanisms forming dependency: engagement design, uncertainty, social confirmation, gradual authority shift. Five-question dependency test. Four-step dismantling protocol. clinical-safety: NOT_REQUIRED. sports-scientist review: NOT_REQUIRED.
- `content/post-2039-vo2max-estimate-longitudinal-tracking.md` — Editorial block 2031–2040 post 9/10. VO2max estimate: що цифра говорить у динаміці. Consumer devices use sub-maximal inference (RMSE ±3–6 ml/kg/min — Shcherbina et al. 2017, Kinnunen et al. 2020). Strength-athlete specifics: body mass increase lowers relative VO2max. Stable-conditions longitudinal trend at 3+ months horizon meaningful; single measurements and comparisons with population norms are not. clinical-safety: NOT_REQUIRED. sports-scientist review: pending.
- `content/post-2040-technology-stack-synthesis.md` — Editorial block 2031–2040 post 10/10 · BLOCK SYNTHESIS. Синтез: технологічний стек self-coached атлета у 2026. Fundamental position: technology supports decisions, does not replace them. Minimum effective stack: training log, timer, camera 1–2×/week. Justified expansion criteria for HRV, full tracker, AI tools. Four questions before any new tool. FORM position on coaching autonomy. clinical-safety: NOT_REQUIRED. sports-scientist review: NOT_REQUIRED.

### Changed
- `blog.html` — six blog cards added for posts 2035–2040.
- `STATUS.md` — Editorial 2031–2040 → **COMPLETE 10/10 · v7.36.0**; current version → v7.36.0.

## [7.35.0] — 2026-06-21

### Added
- `content/post-2034-home-video-analysis-camera-blind-spots.md` — Editorial block 2031–2040 post 4/10. Відеоаналіз техніки вдома: що вчить камера і де її сліпа зона. Key arguments: (1) camera captures 2D projection of 3D movement from one angle — positional markers, bar path, rep drift, rhythm; (2) systematic blind spots: third plane geometry, muscle activation vs. kinematics, internal state/RPE, load context; (3) confirmation bias in self-analysis — Miall & Wolpert (1996) internal model bias; (4) three high-value scenarios: new movement learning, suspected technical regression, technique drift under fatigue; (5) operational protocol: two angles (sagittal + frontal), one question per session, every 1–2 weeks, own-reference comparison only. clinical-safety: NOT_REQUIRED. sports-scientist review: pending.

### Changed
- `blog.html` — post-2034 card added (bottom of grid, after post-12).
- `README.md` — post-2033 → draft · v7.34.0; post-2034 → draft · v7.35.0; roadmap 2061–2070 (тренування в умовах реального дорослого атлета) added as proposed.
- `STATUS.md` — Editorial 2031–2040 → 4/10 · v7.35.0; VERSION → 7.35.0.

## [7.34.0] — 2026-06-21

### Added
- `content/post-2033-training-log-signal-noise-decision.md` — Editorial block 2031–2040 post 3/10. Тренувальний щоденник як інструмент аналізу: сигнал, шум і рішення. Key arguments: (1) logs answer "what happened," not "why" — conflating the two is the most common analytical error; (2) signal vs. noise taxonomy for strength athletes — high-signal: weekly tonnage by movement pattern, 5-week trend on primary lift, one subjective recovery metric; low-signal: single-session result, daily mood, week-over-week delta; (3) false precision trap — 5% adjustments on 20% noisy data; (4) three-metric discipline — track only what drives a decision; (5) three-data-point rule before any programming change. clinical-safety: NOT_REQUIRED. sports-scientist review: pending.

### Changed
- `blog.html` — post-2033 card added (top of grid, above post-2032).
- `STATUS.md` — Editorial 2031–2040 → 3/10; VERSION → 7.34.0.

## [7.33.0] — 2026-06-21

### Added
- `docs/SOC2_READINESS.md` — §79.4: five PURGE-CRON evidence artefacts registered: **PURGE-CRON-E-001** (`system.purge_cron_failure_declared` HMAC chain, CC7.2/PI1.2, 7yr), **PURGE-CRON-E-002** (R-30-C1/C3 aggregate scope queries, CC6.5/PI1.2/P4.1, 7yr), **PURGE-CRON-E-003** (`pg_cron.job_run_details` stale window, CC7.2/A1.1, 3yr), **PURGE-CRON-E-004** (`system.purge_cron_manual_run_completed` HMAC chain, CC6.5/PI1.2, 7yr), **PURGE-CRON-COMP-E-001** (Template PURGE-INT-01 signed memo + Art. 33 assessment, CC6.5/PI1.2, 7yr). All incident-triggered on R-30 P0 activation or when Step 2 executed.

### Changed
- `docs/SOC2_READINESS.md` — §80.3 R2 folder structure: `purge-cron/` subfolder added after `etf-cron/` (`form-api NO ACCESS`; stores per-activation `r30-<incident_id>/` subdirectories).
- `docs/SOC2_READINESS.md` — §80.4 Vanta mirror list: PURGE-CRON-E-001/002/003/004 + PURGE-CRON-COMP-E-001 appended (INCIDENT_RESPONSE R-30.8).
- `docs/INCIDENT_RESPONSE.md` — R-30.11 item 5 marked `[x] Done` (SOC2_READINESS.md v3.25.0, 2026-06-21).
- `STATUS.md` — VERSION → 7.33.0.

## [7.32.1] — 2026-06-21

### Added
- `docs/INCIDENT_RESPONSE.md` — R-30: GDPR Workout Data Retention Purge Failure — `workout_data_purge` pg_cron stale. Full runbook covering severity classification (P1 → P0 → P0 escalated with Art. 33 DPA assessment threshold), T+0–T+30 immediate actions, five root-cause hypotheses with scope queries, PAM-gated manual purge recovery, PURGE-CRON-CHAIN-01 DEC-030 HMAC-chained audit trail (`system.purge_cron_failure_declared` HIGH/7yr, `system.purge_cron_manual_run_completed` STANDARD/7yr, `system.purge_cron_restored` STANDARD/3yr), five evidence artefacts (PURGE-CRON-E-001 → PURGE-CRON-COMP-E-001), SOC 2 CC6.5/CC7.2/A1.2 mapping, and post-incident controls. Owner: compliance-officer + devops-lead + platform-engineer.

### Changed
- `docs/AUDIT_LOG_SCHEMA.md` — §System: three new PURGE-CRON events registered after `system.gdir_alert_check_stale`; PURGE-CRON-CHAIN-01 ordering invariant documented for all three.
- `docs/OBSERVABILITY.md` — §12.6 pg_cron registry: job 26 (`workout_data_purge`) stale-consequence cross-reference updated to include `INCIDENT_RESPONSE R-30 (§R-30.5)`.
- `STATUS.md` — VERSION → 7.32.1.

## [7.32.0] — 2026-06-21

### Added
- `content/post-2032-wearables-what-measured-where-interpretation-fails.md` — Editorial 2031–2040 · Технологія і self-coached практика · Post 2/10. «Носимі пристрої і тренування: що реально вимірюється, де інтерпретація провалюється» — три сенсори (PPG, акселерометр, гіроскоп); де ланцюжок надійний (ЧСС у спокої і рівномірний кардіо: Gillinov et al. 2017, похибка 1.8–5.0%); HRV: Schäfer & Vagedes (2013) умови надійності; стадії сну: Chinoy et al. (2021) точність класифікації 55–78%; калорії: Evenson et al. (2015) похибка 15–50%+; пропрієтарні «оцінки готовності»: Sargent et al. (2020); таблиця трьох категорій (надійні / ситуативно корисні / не для рішень); операційне правило: тренди, не точкові значення; пристрій як друга точка зору, не замінник. clinical-safety: NOT_REQUIRED. sports-scientist review required.
- `blog.html` — картка для post-2032 додана.

### Changed
- `STATUS.md` — Editorial 2031–2040: 1/10 → 2/10. VERSION → 7.32.0.

## [7.31.0] — 2026-06-21

### Added
- `content/post-1921-longterm-stability-cluster5-opener.md` — Block 1881–1930 · Cluster 5 · Post 1/10 · CLUSTER OPENER. «Довгострокова стабільність: від кризового управління до автоматичної адаптації» — п'ять операційних ознак зрілої системи; таблиця кластерів 1–5; Woodman & Hardy (2001) якість системи управління стресом. clinical-safety: NOT_REQUIRED. sports-scientist review required.
- `content/post-1922-automatic-triggers-training-system.md` — Block 1881–1930 · Cluster 5 · Post 2/10. «Автоматичні тригери: як система сама приймає рішення» — Gollwitzer (1999) implementation intentions; три рівні тригерів (часові, IF-THEN, сигнальні HRV/RPE); алгоритм проектування власних тригерів; таблиця шаблонних тригерів. Wood & Neal (2007): мотивація менш релевантна при правильних тригерах. clinical-safety: NOT_REQUIRED. sports-scientist review required.
- `content/post-1923-weekly-training-decision-algorithm.md` — Block 1881–1930 · Cluster 5 · Post 3/10. «Тижневий алгоритм тренувальних рішень: від ситуативного до системного» — три питання (тижнева ємність / режим / конкретні слоти); алгоритм з урахуванням мезоциклу; три приклади застосування; таблиця ситуативний vs. системний. Lally et al. (2010). clinical-safety: NOT_REQUIRED. sports-scientist review required.
- `content/post-1924-longterm-progress-12month-view.md` — Block 1881–1930 · Cluster 5 · Post 4/10. «Довгостроковий прогрес: чому 12-місячна картина важливіша за 4-тижневу» — три ефекти спотворення 4-тижневого горизонту; таблиця хронологічних горизонтів адаптацій; Krieger (2010); структура макроциклу; метрики квартального і річного відстеження. clinical-safety: NOT_REQUIRED. sports-scientist review required.
- `content/post-1925-monthly-review-feedback-loop.md` — Block 1881–1930 · Cluster 5 · Post 5/10. «Мінімальна петля зворотного зв'язку: щомісячний огляд тренувань» — чотири блоки огляду (адгерентність, метрики, суб'єктивна якість 1–5, рішення); шаблон запису; Carver & Scheier (1982) теорія контролю і само-регуляції. clinical-safety: NOT_REQUIRED. sports-scientist review required.
- `content/post-1926-training-log-autonomous-adaptation.md` — Block 1881–1930 · Cluster 5 · Post 6/10. «Тренувальний журнал як інструмент автономної адаптації» — архів vs. зворотний зв'язок; мінімальний запис (5 елементів); читання до і після сесії; патерни мезоциклу; Stamatis et al. (2020) метакогніція. clinical-safety: NOT_REQUIRED. sports-scientist review required.
- `content/post-1927-proactive-training-adaptation.md` — Block 1881–1930 · Cluster 5 · Post 7/10. «Від реакції до прогнозування: проактивна адаптація тренувань» — реактивна vs. проактивна логіка; Baumeister et al. (1998) виснаження его; два рівні проекції; принцип «навмисного деловантаження»; карта ризиків; квартальне вікно (15–20 хв). clinical-safety: NOT_REQUIRED. sports-scientist review required.
- `content/post-1928-self-regulated-athlete-autonomy.md` — Block 1881–1930 · Cluster 5 · Post 8/10. «Self-regulated athlete: операційне визначення тренувальної автономії» — Zimmerman (2000): само-моніторинг → само-оцінка → само-корекція; таблиця self-coached vs. self-regulated; Bandura (1997) самоефективність; п'ять операційних ознак; три базові практики. clinical-safety: CONDITIONAL PASS. sports-scientist review required.
- `content/post-1929-from-rules-to-principles-training-maturity.md` — Block 1881–1930 · Cluster 5 · Post 9/10. «Від правил до принципів: тренувальна зрілість і її ознаки» — чотири принципи силового тренування; п'ять ознак переходу; таблиця стадій зрілості; де правила залишаються цінними; практичний тест. clinical-safety: NOT_REQUIRED. sports-scientist review required.
- `content/post-1930-cluster5-block-synthesis.md` — Block 1881–1930 · Cluster 5 · Post 10/10 · CLUSTER SYNTHESIS · BLOCK SYNTHESIS. «Від виживання до автономії: синтез кластеру 5 і блоку 1881–1930» — операційна карта 10 постів кластеру 5; п'ять принципів блоку; вісь прогресу від виживання до автономії; preview блоків 1931–2000. clinical-safety: NOT_REQUIRED. sports-scientist review required.
- `content/post-2047-universal-principles-dont-apply-to-you.md` — Editorial 2041–2050 · Post 7/10. «Чому тренувальні принципи «для всіх» не для тебе» — Hubal et al. (2005): діапазон відповіді на тренування −2% до +59%; чотири домени варіативності (частота, діапазон, обсяг, відновлення); три помилки від ставлення до «середніх» як до особистих; алгоритм персоналізації 5 кроків. clinical-safety: NOT_REQUIRED. sports-scientist review required.
- `content/post-2048-dunning-kruger-strength-training.md` — Editorial 2041–2050 · Post 8/10. «Ефект Даннінга-Крюгера у силовому тренуванні: карта некомпетентності» — чому тренування ідеальне середовище для DK-ефекту; чотири стадії; де ефект болісний (техніка, відновлення, оцінка програми); п'ять практик зниження. clinical-safety: NOT_REQUIRED. sports-scientist review required.
- `content/post-2049-evaluate-training-advice-criteria.md` — Editorial 2041–2050 · Post 9/10. «Як оцінити якість тренувальної поради: операційні критерії» — четири причини труднощів оцінки; п'ять критеріїв (компетенція, обґрунтування, специфічність, джерело, власні дані); швидкий фільтр; де поради найненадійніші. clinical-safety: NOT_REQUIRED. sports-scientist review required.
- `content/post-2050-editorial-block-synthesis-operational-thinking.md` — Editorial 2041–2050 · Post 10/10 · BLOCK SYNTHESIS. «Синтез блоку 2041–2050: операційне мислення self-coached атлета» — операційна карта 10 постів; п'ять принципів блоку; хибне спрощення як маркетинговий механізм; «До/Після» фрейм операційного мислення; preview Editorial 2051–2060. clinical-safety: NOT_REQUIRED. sports-scientist review required.
- `blog.html` — 14 карток додано (posts 1921–1930, 2047–2050).

### Changed
- `STATUS.md` — Block 1881–1930: Cluster 5 COMPLETE 50/50 (BLOCK COMPLETE). Editorial 2041–2050: 6/10 → 10/10 (COMPLETE). Version updated to v7.31.0.
- `VERSION` — 7.30.0 → 7.31.0.

---

## [7.30.0] — 2026-06-21

### Added
- `content/post-1911-extended-pause-return-fundamentals.md` — Block 1881–1930 · Cluster 4 · Post 1/10 · CLUSTER OPENER. «Після тривалої паузи: чому просто «повернутись» не працює» — два режими провалу (надто агресивний / надто пасивний); пауза як передбачувана характеристика довгострокової практики; карта кластеру 4. clinical-safety: NOT_REQUIRED. sports-scientist review required.
- `content/post-1912-detraining-what-is-lost.md` — Block 1881–1930 · Cluster 4 · Post 2/10. «Деадаптація: що реально втрачається за 2–8 тижнів без тренувань» — три якості по-різному: нейральна сила (Hortobágyi 2000), гіпертрофія (Bickel 2011), VO₂max (Mitchell 1992); таблиця хронології по тижнях; міонуклеарна ретенція і м'язова пам'ять (Gundersen 2016); пастка нейральної готовності. clinical-safety: NOT_REQUIRED. sports-scientist review required.
- `content/post-1913-athlete-identity-during-pause.md` — Block 1881–1930 · Cluster 4 · Post 3/10. «Ідентичність атлета під час паузи: не «колишній», а «на паузі»» — Stambulova 2009 (ідентичність і кар'єрні переходи); Brewer et al. 1993 (Athletic Identity); три тактики: мінімальний фізичний зв'язок, мова/журнал, конкретна дата повернення (Gollwitzer 1999). clinical-safety: CONDITIONAL PASS. sports-scientist review required.
- `content/post-1914-ramp-up-short-pause.md` — Block 1881–1930 · Cluster 4 · Post 4/10. «Рамп-ап протокол: повернення після паузи 2–6 тижнів» — 3-тижневий протокол (60–70% → 75–85% → 100% обсягу, RPE cap поетапний); DOMS після повернення (Clarkson & Hubal 2002); DOMS як операційний сигнал. clinical-safety: NOT_REQUIRED. sports-scientist review required.
- `content/post-1915-ramp-up-extended-pause.md` — Block 1881–1930 · Cluster 4 · Post 5/10. «Рамп-ап протокол: повернення після паузи 2–6 місяців і більше» — 3-фазний протокол (стабілізація тж. 1–2 → акумуляція тж. 3–5 → нормалізація тж. 6+); помилка «надолужити»; рамка «новий підготовчий блок зі стартовою перевагою». clinical-safety: NOT_REQUIRED. sports-scientist review required.
- `content/post-1916-minimum-effective-dose-insurance.md` — Block 1881–1930 · Cluster 4 · Post 6/10. «Мінімальна підтримуюча доза: страховка від паузи» — Bickel 2011 (1/9 обсягу зберігає силу); Androulakis-Korakakis 2020 (1–2 важких сети/тиж підтримують гіпертрофію); шаблон МПД 2×/тиж; превентивне розгортання. clinical-safety: NOT_REQUIRED. sports-scientist review required.
- `content/post-1917-antifragile-training-architecture.md` — Block 1881–1930 · Cluster 4 · Post 7/10. «Антикрихкість у тренуванні: система, яка не ламається» — 3 принципи: надлишковість (місце/час/вправи), масштабованість (3 режими з критеріями), МПД як якір; аудит крихкості (5 запитань); Wood & Neal 2007 (структура > воля). clinical-safety: NOT_REQUIRED. sports-scientist review required.
- `content/post-1918-emergency-training-protocol.md` — Block 1881–1930 · Cluster 4 · Post 8/10. «Аварійний план тренування: протокол форс-мажору» — decision fatigue (Baumeister 2011); шаблон 2×/тиж 30 хв (сесія А: нижнє+тяг, сесія Б: верхнє+задня); критерії активації і деактивації; документування плану. clinical-safety: NOT_REQUIRED. sports-scientist review required.
- `content/post-1919-training-identity-long-term-resilience.md` — Block 1881–1930 · Cluster 4 · Post 9/10. «Тренувальна ідентичність як фундамент стійкості: довгострокова перспектива» — неперервність як власний тренувальний фактор (Krieger 2010); лог + мова + PR + авторство; ідентичність прив'язана до практики, не до результатів (Stambulova 2009). clinical-safety: CONDITIONAL PASS. sports-scientist review required.
- `content/post-1920-cluster4-synthesis-resilience-map.md` — Block 1881–1930 · Cluster 4 · Post 10/10 · CLUSTER SYNTHESIS. «Від виживання до стійкості: операційна карта кластеру 4» — таблиця 9 постів з інструментами; 3 алгоритми (ПАУЗА ПОЧАЛАСЬ / ПРОГНОЗУЄТЬСЯ / ПРОФІЛАКТИКА); 6 принципів кластеру; preview Cluster 5 «Довгострокова стабільність». clinical-safety: NOT_REQUIRED. sports-scientist review required.
- `content/post-2046-training-perfectionism-help-hinder.md` — Editorial 2041–2050 · Post 6/10. «Тренувальний перфекціонізм: де він допомагає, де руйнує» — адаптивний vs. дезадаптивний перфекціонізм (Frost et al. 1990; Hewitt & Flett 1991); корисний у технічному домені (точність виконання, журнал, режим сну); деструктивний у домені дотримання (критерій входу, катастрофізація, novel program bias); операційна таблиця; «стандарт якості» vs. «критерій входу». clinical-safety: CONDITIONAL PASS. sports-scientist review required.
- `blog.html` — 11 карток додано (posts 1911–1920, 2046).

### Changed
- `STATUS.md` — Block 1881–1930: 30/50 → 40/50 · Cluster 4 COMPLETE. Editorial 2041–2050: 5/10 → 6/10.
- `VERSION` — 7.29.0 → 7.30.0.

---

## [7.29.0] — 2026-06-21

### Added
- `content/post-16-underrecovery-vs-overtraining.md` — Editorial series · Post 16/∞. «Недовідновлення або перетренованість: як self-coached атлет читає сигнали тіла» — чотири точки спектру втоми (гостра втома, функціональне перевантаження, нефункціональне перевантаження, OTS); надійні маркери (HRV-тренд, RPE-дрейф, технічна деградація, сон); ненадійні (мотивація, DOMS, суб'єктивне «відчуття»); практичний протокол розрізнення; таблиця відповідей за станом. clinical-safety: NOT_REQUIRED. sports-scientist review pending.
- `blog.html` — картка для post-16 додана перед post-15.
- `README.md` — Editorial series 281–290 «Self-coached athlete decision systems» (10 тем, proposed) · v7.29.0.

### Changed
- `STATUS.md` — content library row 11–50: post-16 зафіксовано.
- `VERSION` — 7.28.1 → 7.29.0.

---

## [7.28.1] — 2026-06-21

### Changed
- `docs/OBSERVABILITY.md` — v4.8.3: §12.6 job 25 cross-ref column updated to include `INCIDENT_RESPONSE R-29 (job 25 stale recovery runbook — §R-29.5)`; §36.5 implementation checklist item 6 added (P1/M10, [x] Done). Closes R-29.11 item 4 and R-29.10 post-incident control.
- `docs/SOC2_READINESS.md` — v3.24.6: §79.4 master evidence table — five ETF-CRON artefacts registered (ETF-CRON-E-001/CC4.1+CC7.2, ETF-CRON-E-002/CC5.2+CC7.2, ETF-CRON-E-003/CC4.1+A1.1, ETF-CRON-E-004/CC4.1+CC5.2, ETF-CRON-COMP-E-001/CC4.1+CC5.2); §80.3 `etf-cron/` R2 subfolder added; §80.4 Vanta mirror list updated. Closes R-29.11 item 5.
- `docs/INCIDENT_RESPONSE.md` — R-29.11 items 4 and 5 marked [x] Done.
- `VERSION` — 7.28.0 → 7.28.1.

---

## [7.28.0] — 2026-06-21

### Added
- `content/post-2045-program-boredom-diagnosis.md` — Block 2041–2050 · Post 5/10. «Нудьга з програмою: діагноз чи сигнал до зміни» — операційна діагностика між психологічною монотонією і функціональною стагнацією; Lally et al. 2010 (повторюваність як механізм автоматизації); Fiogbee 2019 (long-term adherence); три рівні відповіді (прогрес є + дотримання є → продовжуй; дотримання падає → мінімальна косметична зміна; прогрес зупинився + деловантаження не допомогло → плановий перехід); novel program bias як контент-маркетинговий механізм; Victor: рішення на основі даних, не нудьги. clinical-safety: NOT_REQUIRED. sports-scientist review required before publish.
- `README.md` — roadmap: posts 2051–2060 додано (proposed): ціна помилки, прогресія без числа, технічне плато, журнал як дзеркало, відпочинок між сетами, зміна вправи, блокова структура без змагань, нейром'язова адаптація рік 1, документація vs. аналітика, синтез блоку.

### Changed
- `VERSION` — 7.27.0 → 7.28.0.

---

## [7.27.0] — 2026-06-21

### Added
- `content/post-1901-internal-transition-opener.md` — Block 1881–1930 · Cluster 3 · Post 1/10. Внутрішній перехід як окрема категорія: три типи (мотиваційний, цільовий, перфекціоністська стагнація), розрізнення від перетренованості. clinical-safety: NOT_REQUIRED. sports-scientist review required.
- `content/post-1902-motivational-transition-training.md` — Block 1881–1930 · Cluster 3 · Post 2/10. Системний мотиваційний спад: чотири причини (вичерпана ціль, плато, зміна контексту, накопичена втома), діагностична послідовність, що реально допомагає. clinical-safety: NOT_REQUIRED. sports-scientist review required.
- `content/post-1903-all-or-nothing-training-pattern.md` — Block 1881–1930 · Cluster 3 · Post 3/10. «Все або нічого»: механіка, три джерела, практичні прояви, обхідний протокол (офіційний мінімальний формат, перейменування, відокремлення явки від якості). Bickel et al. 2011. clinical-safety: NOT_REQUIRED. sports-scientist review required.
- `content/post-1904-perfectionism-training-regularity.md` — Block 1881–1930 · Cluster 3 · Post 4/10. Перфекціонізм: де функціональний (технічний, програмний частково), де дисфункціональний (умов, виконання, прогресу, помилки). Тест трьох рівнів. clinical-safety: NOT_REQUIRED. sports-scientist review required.
- `content/post-1905-training-goal-change-adaptation.md` — Block 1881–1930 · Cluster 3 · Post 5/10. Зміна цілей: три типи, п'ятикроковий протокол, ідентичність у практиці а не в цілі, поступова адаптація vs. перезапуск. clinical-safety: NOT_REQUIRED. sports-scientist review required.
- `content/post-1906-priority-shift-strength-to-other.md` — Block 1881–1930 · Cluster 3 · Post 6/10. Зміна пріоритету від сили: interference effect (Wilson 2012), три сценарії (→витривалість, →мобільність, →специфіка), три запитання перед зміною. clinical-safety: NOT_REQUIRED. sports-scientist review required.
- `content/post-1907-compound-internal-external-transition.md` — Block 1881–1930 · Cluster 3 · Post 7/10. Складений перехід: три приклади взаємопідсилення, ознаки, хибна тактика «по черзі», протокол паралельної обробки. clinical-safety: NOT_REQUIRED. sports-scientist review required.
- `content/post-1908-simultaneous-changes-hierarchy-protocol.md` — Block 1881–1930 · Cluster 3 · Post 8/10. Ієрархія рішень при «ідеальному шторм»: три рівні, 4-тижневий протокол, принцип одного рішення на тиждень. clinical-safety: NOT_REQUIRED. sports-scientist review required.
- `content/post-1909-exit-prolonged-internal-transition.md` — Block 1881–1930 · Cluster 3 · Post 9/10. Вихід з тривалого внутрішнього переходу: чотири фази, правило першого кроку (20–30 хвилин, 50–60% об'єму), психологічний борг, відновлення vs. перебудова. clinical-safety: NOT_REQUIRED. sports-scientist review required.
- `content/post-1910-cluster3-synthesis-internal-transitions.md` — Block 1881–1930 · Cluster 3 · Post 10/10 · CLUSTER SYNTHESIS. Операційна карта внутрішніх переходів: всі типи у таблицях, шість принципів, спільний алгоритм-блок-схема, анонс Cluster 4. clinical-safety: NOT_REQUIRED. sports-scientist review required.
- `content/post-2044-training-to-failure-when-right-when-wrong.md` — Editorial 2041–2050 · Post 4/10 (Хибні спрощення в тренувальній культурі). Відмова у тренуванні: технічна vs. механічна vs. конкурентна. Giessing 2016, Sampson & Groeller 2016, Lasevicius 2022 — зона 0–4 RIR оптимальна, 0 vs. 2–3 RIR мінімальна різниця при суттєво нижчій системній вартості. Де відмова виправдана (ізольовані, останній підхід, тренажери). Де надлишкова (вільні рухи без страховки, ранні підходи, нова техніка, поганий відновлення). Операційна таблиця RIR-таргетингу. clinical-safety: NOT_REQUIRED. sports-scientist review pending.

### Changed
- `blog.html` — blog cards added: post-1901 through post-1910 (Training Behavior), post-2044 (Sports Science). blog.html updated to post-1910 and post-2044.
- `VERSION` — 7.26.1 → 7.27.0.

---

## [7.26.1] — 2026-06-21

### Added
- `docs/INCIDENT_RESPONSE.md R-29` — Enterprise Mid-Contract Termination Risk Monitoring Failure. Twenty-ninth runbook. Closes the documentation gap in `docs/OBSERVABILITY.md §12.6` for pg_cron job 25 (`mid_contract_termination_risk_check`, schedule `0 9 * * 1`, 8-day freshness): §12.6 defines PagerDuty P1 `form-customer-success` routing on staleness but no IC recovery runbook existed. R-29 provides: three-severity matrix (P1 stale / P0 high-ACV at-risk tenant undetected / P0 escalated contract terminated during stale); T+0–T+30 immediate actions with `system.etf_cron_failure_declared` HIGH/7yr at T+0; three scope queries (R-29-C1 at-risk tenant assessment, R-29-C2 pg_cron.job_run_details stale duration, R-29-C3 tenants recovered without intervention); five root cause hypotheses H1–H5; four-step recovery (restore job 25, manual §36.2 backfill per missed Monday, CSM intervention, confirm restoration); internal Template ETF-INT-01 (no customer-facing template — CSM outreach is normal AL-ETF-01 protocol); three DEC-030 HMAC-chained events with ETF-CRON-CHAIN-01 ordering invariant; five SOC 2 evidence artefacts (CC4.1/CC5.2/CC7.2/A1.1); seven-item implementation checklist.

### Changed
- `docs/INCIDENT_RESPONSE.md` — version footer v2.9 → v3.0.
- `VERSION` — 7.26.0 → 7.26.1.

---

## [7.26.0] — 2026-06-21

### Added
- `content/post-2043-sets-quality-vs-quantity.md` — Block 2041–2050 · Post 3/10. «Кількість vs. якість підходів: де реальне тренувальне навантаження» — концепція ефективного повторення (Morton et al. 2019: близькість до відмови, а не абсолютна вага); ефективний підхід (Baz-Valle et al. 2022: 0–4 RIR); деградація якості всередині сесії (Fink et al. 2018: 1 хв vs. 3 хв відпочинку); деградація між сесіями при недовідновленні; алгоритм «коли додавати підходи, коли підвищувати якість»; калібрування RIR для self-coached атлета; техніка як компонент якості підходу; Victor RPE/RIR логіка. Schoenfeld et al. 2017, Helms et al. 2016, Krieger 2010. clinical-safety: NOT_REQUIRED. sports-scientist review required before publish.

### Changed
- `blog.html` — post-2043 картка додана (Sports Science · 13 хв · draft).
- `STATUS.md` — Editorial 2041–2050 IN PROGRESS 3/10 · v7.26.0. Version footer оновлений до v7.26.0.
- `README.md` — post-2042 → draft · v7.24.0; post-2043 → draft · v7.26.0.
- `VERSION` — 7.25.0 → 7.26.0.

---

## [7.25.0] — 2026-06-21

### Added
- `content/post-1891-new-job-schedule-disruption.md` — Block 1881–1930 · Cluster 2 · Post 1/10. Нова робота і радикальна зміна графіку: мета першого місяця = явка не якість, три варіанти часового вікна (до/після/обід) з критеріями вибору, адаптація маршруту, когнітивне навантаження онбордингу як алостатичний стресор, соціальні зобов'язання vs. тренування в перші 4–6 тижнів, покрокований чекліст. clinical-safety: NOT_REQUIRED. sports-scientist review required before publish.
- `content/post-1892-business-travel-training-system.md` — Block 1881–1930 · Cluster 2 · Post 2/10. Ділові поїздки: три рівні польового протоколу (A — зал з вагами, B — гантелі/гумки, C — власна вага), чекліст перед поїздкою, правило 48 годин (перша сесія встановлює прецедент), часова зона і тренування, протокол конфлікту з діловою вечерею, довге відрядження 2+ тижні як тимчасова нова база. clinical-safety: NOT_REQUIRED. sports-scientist review required before publish.
- `content/post-1893-relocation-new-city-training-setup.md` — Block 1881–1930 · Cluster 2 · Post 3/10. Переїзд у нове місто або країну: Wood et al. (2005) — вікно відкритості 2–4 тижні, правило першого тижня (зал і перша сесія до нових рутин), три пріоритети (зал / слот / маршрут), специфіка міжнародного переїзду, переїзд як перезавантаження системи. clinical-safety: NOT_REQUIRED. sports-scientist review required before publish.
- `content/post-1894-parenthood-training-restructure.md` — Block 1881–1930 · Cluster 2 · Post 4/10. Народження дитини: реальність перших 3 місяців (розбитий сон, когнітивне навантаження, алостатичний стрес), нові тренувальні вікна (ранній ранок / денний сон / після партнера / вихідні), зміна очікувань «оптимальне» → «достатнє», формат 2 сесій A/B 45 хв, вплив дефіциту сну на нейром'язову координацію, DEC-013 grace. clinical-safety: NOT_REQUIRED. sports-scientist review required before publish.
- `content/post-1895-remote-office-transition.md` — Block 1881–1930 · Cluster 2 · Post 5/10. Офіс↔дистанційно: два напрямки переходу з різними ризиками, офіс→дистанційно (втрата маршруту-тригера, розмиття меж, зниження локомоції), дистанційно→офіс (втрата часового ресурсу), гібрид (прив'язка до дня/часу не формату), примусовий перехід і behavioural spillover, матриця рішень 4×3. clinical-safety: NOT_REQUIRED. sports-scientist review required before publish.
- `content/post-1896-gym-loss-home-training-bridge.md` — Block 1881–1930 · Cluster 2 · Post 6/10. Втрата залу: що домашнє тренування може і не може, три рівні (0/1/2) з конкретними протоколами, психологічна помилка «домашнє не рахується», організація простору, паралельний пошук нового залу, межі ефективності (до 4–8 тижнів / 8–16 / після 16), правило першої домашньої сесії за 48h. clinical-safety: NOT_REQUIRED. sports-scientist review required before publish.
- `content/post-1897-high-cognitive-load-training.md` — Block 1881–1930 · Cluster 2 · Post 7/10. Когнітивне навантаження: Marcora et al. (2009) — когнітивна стомленість підвищує суб'єктивний RPE, три режими (кризовий тиждень / розтягнутий стрес / пік з відомою датою), тренування як інструмент управління когнітивним станом (Chang et al. 2012 — 2–4h покращення), уникати нових вправ і RPE 10, правило «не відпрацьовувати» після кризи. clinical-safety: NOT_REQUIRED. sports-scientist review required before publish.
- `content/post-1898-financial-constraint-training.md` — Block 1881–1930 · Cluster 2 · Post 8/10. Фінансові обмеження: ієрархія витрат на тренування (6 рівнів), мінімальні вимоги до бюджетного залу, де шукати (муніципальні / університети / гаражні зали), домашній зал — коли має сенс, реальний потенціал гантелей для досвідченого атлета, фінансова ситуація і тренувальна ідентичність. clinical-safety: NOT_REQUIRED. sports-scientist review required before publish.
- `content/post-1899-vacation-seasonal-transition.md` — Block 1881–1930 · Cluster 2 · Post 9/10. Відпустка і сезони: тривалість як головний фактор (4 рівні), три рівні польового протоколу без залу, психологія рішення до початку, сезонні переходи (зима→весна форсований ріст, літня спека і нейром'язова координація, зимова зниженість енергії), протокол повернення після 1–2 і 3–4 тижнів. clinical-safety: NOT_REQUIRED. sports-scientist review required before publish.
- `content/post-1900-cluster2-synthesis-life-transition-matrix.md` — Block 1881–1930 · Cluster 2 · Post 10/10 · CLUSTER SYNTHESIS. Матриця 9 life-transition сценаріїв: головний ризик, ключове рішення, мінімум підтримки. П'ять спільних принципів: рішення до переходу, мінімальна присутність важливіша за якість, тимчасове зниження ≠ деградація, перший крок критичний, після переходу — не відпрацьовувати. Операційна блок-схема. Victor і контекстні мітки. Коли life-transition потребує більшого. clinical-safety: NOT_REQUIRED. sports-scientist review required before publish.

### Changed
- `blog.html` — cards added for posts 1891–1900 (Block 1881–1930 Cluster 2 · Конкретні life-transition сценарії). blog.html updated to post-1900.
- `STATUS.md` — Block 1881–1930 updated: Cluster 2 COMPLETE 10/10 (posts 1891–1900); next: Cluster 3 (posts 1901–1910).
- `VERSION` — 7.24.1 → 7.25.0.

---

## [7.24.1] — 2026-06-21

### Changed
- `docs/SOC2_READINESS.md` — v3.24.5: §79.4 master evidence table — six RENEW-CRON incident-triggered artefacts registered: RENEW-CRON-E-001 (CC4.1/CC7.2, 7yr — `system.renew_cron_failure_declared` HMAC chain export), RENEW-CRON-E-002 (CC4.1/CC2.2, 7yr — R-28-C1/C3 SQL outputs), RENEW-CRON-E-003 (CC4.1/A1.1, 3yr — `pg_cron.job_run_details` stale window), RENEW-CRON-E-004 (CC4.1/CC2.2, 7yr — `system.renew_cron_manual_check_completed` chain exports), RENEW-CRON-E-005 (CC2.2/CC4.1, 7yr — `enterprise.renewal_notice_sent` chain exports), RENEW-CRON-COMP-E-001 (CC4.1/CC2.2, 7yr — Template REN-01 compensating control memo). §80.3 `renewals/` comment updated. §80.4 Vanta mirror list updated. Closes INCIDENT_RESPONSE R-28.11 item 5 (P1/M11).
- `docs/OBSERVABILITY.md` — v4.8.2: §12.6 pg_cron registry job 39 stale-condition cross-ref column updated with `INCIDENT_RESPONSE R-28 (job 39 stale recovery runbook — §R-28.5)`; §51.7 item 8 added ([x] Done) documenting R-28 cross-reference closure. Closes INCIDENT_RESPONSE R-28.11 item 4 (P1/M11) and R-28.10 post-incident control.
- `docs/INCIDENT_RESPONSE.md` — R-28.11 items 4 and 5 marked `[x] Done` with closure notes referencing OBSERVABILITY.md v4.8.2 and SOC2_READINESS.md v3.24.5 respectively.
- `VERSION` — 7.24.0 → 7.24.1.

---

## [7.24.0] — 2026-06-21

### Added
- `content/post-2042-progressive-overload-three-failures.md` — Block 2041–2050 · Post 2/10. «"Більше = краще": три ситуації, де принцип прогресивного перевантаження перетворюється на помилку» — три сценарії: (1) об'єм до відновлювальної адаптації (Schoenfeld et al. 2017 + overreach pattern); (2) вага до технічної консолідації (Comfort et al. 2012); (3) частота до стабілізації відновлення (Ralston et al. 2017 + volume-not-adjusted error). Спільна механіка — перевантаження здатності, а не м'яза. Операційний протокол трьох питань. Victor логіка: вимагає метрики деградації перед рекомендацією прогресу. clinical-safety: NOT_REQUIRED. sports-scientist: review pending.

### Changed
- `blog.html` — post-2042 картка додана (Sports Science · 13 хв · draft).
- `STATUS.md` — Block 2041–2050 IN PROGRESS 2/10 · v7.24.0. Version header оновлений до v7.24.0.
- `VERSION` — 7.23.0 → 7.24.0.

---

## [7.23.0] — 2026-06-21

### Added
- `content/post-1882-spatial-transition-training.md` — Block 1881–1930 · Cluster 1 · Post 2/10. «Просторовий перехід: що відбувається з тренуванням при переїзді або зміні залу» — що саме руйнується при зміні простору (маршрут, роздягальня, обладнання, соціальне мікросередовище); три сценарії (зміна залу у тому ж місті 3–6 тижнів, переїзд у нове місто, відрядження 2–6 тижнів з протоколом першої сесії в перші 2 дні); чому чекати «ідеальний зал» = пастка паузи; перекалібрування обладнання (-5–10% до вагів, 2 тижні); чекліст. clinical-safety: NOT_REQUIRED. sports-scientist: review pending.
- `content/post-1883-temporal-transition-training.md` — Block 1881–1930 · Cluster 1 · Post 3/10. «Часовий перехід: тренування при зміні графіка роботи і розпорядку дня» — чотири причини, чому «просто перенести тренування» не спрацьовує (рівень втоми, ментальний стан, соціальна структура, втрата тригера); чотири сценарії (нова робота, дистанційка, повернення в офіс, сімейний розпорядок); ранок vs. вечір (Chtourou & Souissi 2012): 4–6 тижнів адаптації; карта нормалізації (тиж. 1–6); чекліст. clinical-safety: NOT_REQUIRED. sports-scientist: review pending.
- `content/post-1884-social-transition-training.md` — Block 1881–1930 · Cluster 1 · Post 4/10. «Соціальний перехід: тренування після втрати партнера, нового середовища, зміни підзвітності» — чотири функції тренувального партнера (commitment device, структурний час, мікросоціальний тонус, спільний наратив); від тренера до self-coached; три рівні замінної підзвітності (соціальна/структурна/системна); група→індивідуально (peer performance effect, Feltz 2011); соціальна адаптація у новому залі 4–6 тижнів; чекліст. clinical-safety: NOT_REQUIRED. sports-scientist: review pending.
- `content/post-1885-compound-transition-training.md` — Block 1881–1930 · Cluster 1 · Post 5/10. «Складений перехід: коли змінюється все одночасно» — синергетичний ефект множинних переходів; три сценарії (переїзд+нова робота, народження дитини, зміна роботи+особистий контекст); горизонти 8–12 тижнів (подвійний) і 3–6 місяців (повний); три пастки (все/нічого, надолуження, порівняння з «дового»); Victor як зовнішня пам'ять при хаотичному переході. clinical-safety: NOT_REQUIRED. sports-scientist: review pending.
- `content/post-1886-short-vs-long-disruption.md` — Block 1881–1930 · Cluster 1 · Post 6/10. «Коротке порушення vs. довгострокова зміна контексту: різні відповіді» — критерій розмежування: ≤4 тижні = «утримання мінімуму» / >4 тижні = «побудова нової системи»; мінімальна підтримуюча доза (Bickel 2011); сіра зона 4–8 тижнів; паралельна стратегія при невизначеності; таблиця тривалість паузи vs. зміни адаптацій. clinical-safety: NOT_REQUIRED. sports-scientist: review pending.
- `content/post-1887-building-new-triggers.md` — Block 1881–1930 · Cluster 1 · Post 7/10. «Побудова нових тригерів: практичний протокол» — п'ять критеріїв ефективного тригера (конкретність, habit stacking Fogg/Clear, мінімальний поріг, повторюваність, відчутність у новому контексті); три типи тригерів (часовий, подієвий, середовищний); п'ятикроковий протокол; Lally et al. (2010) медіана 66 днів; чотири помилки; Victor і implementation intentions (Gollwitzer 1999). clinical-safety: NOT_REQUIRED. sports-scientist: review pending.
- `content/post-1888-mev-context-shift.md` — Block 1881–1930 · Cluster 1 · Post 8/10. «Мінімальна ефективна доза при контекстному переході: що зберігати в першу чергу» — Bickel et al. (2011) 1/9 об'єму при збереженій інтенсивності підтримує адаптації 16 тижнів; параметри MEV (1–2 сесії/тиж, 2–3 підходи на м'язову групу, 80–85% 1RM); ієрархія скорочень; два практичні формати; помилка «легкі ваги при зниженому об'ємі». clinical-safety: NOT_REQUIRED. sports-scientist: review pending.
- `content/post-1889-return-after-gap-within-transition.md` — Block 1881–1930 · Cluster 1 · Post 9/10. «Повернення після паузи всередині переходу: що робити, якщо вже пропустив» — myonuclear retention (Bruusgaard 2010, Egner 2013); таблиця тривалість паузи vs. горизонт відновлення; протокол 60→75→100% за 4–6 тижнів; три типи повернення (той самий контекст / новий контекст / 3+міс пауза); особлива увага до сполучної тканини при тривалій паузі; три хибні психологічні посилання; перша сесія — розвідка, не демонстрація. clinical-safety: NOT_REQUIRED. sports-scientist: review pending.
- `content/post-1890-cluster1-synthesis-context-shift.md` — Block 1881–1930 · Cluster 1 · Post 10/10 · CLUSTER SYNTHESIS. «Операційна система контекстної адаптації: синтез кластеру 1» — карта рішень (виявити → тип → тривалість → тригер vs. повернення → MEV → checkpoint); п'ять принципів; зведені протоколи для трьох типів переходів; таблиця горизонтів адаптації; десять помилок; Victor як контекстна інфраструктура. **CLUSTER 1 COMPLETE 10/10.** clinical-safety: NOT_REQUIRED. sports-scientist: review pending.

### Changed
- `blog.html` — posts 1882–1890 додані (9 карток, Block 1881–1930 Cluster 1); blog.html updated to post-1890.
- `VERSION` — 7.22.2 → 7.23.0.

---

## [7.22.2] — 2026-06-21

### Changed
- `docs/INCIDENT_RESPONSE.md` — v2.8 → v2.9: §19 / R-28 Enterprise Contract Renewal Notice Monitoring Failure — twenty-eighth runbook. Covers `renewal_notice_check` pg_cron job 39 (OBSERVABILITY §51) going stale beyond the 26h freshness window, taking the RENEW-NOTICE-01 detection control offline. Closes documentation gap in OBSERVABILITY §12.6 (job 39 stale alert defined but no recovery runbook existed). Three-severity matrix: P1 (stale, no tenant in 85–95-day detection window), P0 (stale + tenant in detection window, notice not yet sent — 2h SLA to dispatch manually), P0 escalated (detection window fully closed with no notice — customer notification + outside counsel within 1h). R-28-C1/C2/C3 scope queries, five root cause hypotheses (H1: job deleted, H2: SQL exception, H3: pg_net degraded, H4: service_role permission revoked, H5: platform outage), four-step recovery (restore job 39, manual RENEW-NOTICE-01 backfill per missed window, manual notice dispatch via Admin Console, confirm restoration). Two communication templates: REN-01 (internal compensating control memo) and REN-02 (customer notification for P0 escalated — outside counsel review required if days_remaining < 60). Three DEC-030 HMAC-chained events: `system.renew_cron_failure_declared` HIGH/7yr, `system.renew_cron_manual_check_completed` STANDARD/7yr, `system.renew_cron_restored` STANDARD/3yr; RENEW-CRON-CHAIN-01 ordering invariant. Six evidence artefacts RENEW-CRON-E-001–E-005 + RENEW-CRON-COMP-E-001 mapped to CC4.1/CC2.2/CC7.2/A1.1. Seven-item implementation checklist: 2× P0/M11, 3× P1/M11, 2× P2/M8–M12 (incl. Scenario Q tabletop and job 40 weekly dry-run). Cross-references: OBSERVABILITY §51/§12.6/§12.7, COST_MODEL §42.3.2/§42.7, DATA_MODEL §43, R-05, R-03, R-27. compliance-officer + devops-lead + platform-engineer.
- `VERSION` — 7.22.1 → 7.22.2.

---

## [7.22.1] — 2026-06-21

### Added
- `content/post-1881-context-shift-training-behavior.md` — Block 1881–1930 · Cluster 1 · Post 1/10 (Поведінкова адаптація при зміні контексту). «Контекстний перехід і тренувальна система: чому рутини ламаються і що з цим робити» — habit discontinuity hypothesis (Verplanken & Roy 2016): звичні поведінкові патерни прив'язані до контекстних тригерів (Wood & Neal 2007), а не до характеру або мотивації; три типи переходів (просторовий: переїзд/зміна залу, часовий: нова робота/дитина/дистанційка, соціальний: зміна партнера/мережі); чому «просто тренуйся» не працює при переході (cognitive load, Neal et al. 2013); протокол 4 кроки (визнати перехід як поведінкову подію, новий anchor через habit stacking, мінімальна версія 30–40 хв зі збереженням інтенсивності Bickel 2011, двотижневий checkpoint); чотири помилки (чекати стабілізації, чекати «відчуття», відтворювати старий формат, компенсувати об'ємом); особливий випадок народження дитини; Victor як зовнішня пам'ять з контекстом. clinical-safety: NOT_REQUIRED.

### Changed
- `blog.html` — картка post-1881 додана вгорі.
- `STATUS.md` — Block 1881–1930 IN PROGRESS 1/50 · v7.22.1 додано; версія оновлена.
- `VERSION` — 7.22.0 → 7.22.1.

---

## [7.22.0] — 2026-06-21

### Added
- `content/post-1871-longterm-athlete-expectations-vs-reality.md` — Block 1831–1880 · Cluster 5 · Post 1/10. Що відбувається через 5–10 років тренування: нелінійне сповільнення адаптаційного темпу (Schoenfeld 2016), чотири розриви між очікуванням і реальністю, що реально накопичується за 10 років (технічна зрілість, знання власного відгуку, стійкість системи, калібровка зусиль), три типи довгострокових атлетів. clinical-safety: NOT_REQUIRED · sports-scientist review required.
- `content/post-1872-behavioral-crises-in-training.md` — Block 1831–1880 · Cluster 5 · Post 2/10. Поведінкові кризи: чотири типи (криза цілей, криза програми, криза ідентичності, накопичена стомленість), діагностичний протокол 4 кроки, головна помилка — ігнорування. clinical-safety: NOT_REQUIRED · sports-scientist review required.
- `content/post-1873-goals-change-over-years.md` — Block 1831–1880 · Cluster 5 · Post 3/10. Зміна цілей з роками: чотири природних переходи (від результату до процесу, від «більше» до «краще», від досягнення до підтримання, від цілей до стилю практики), три помилки переоцінки цілей. clinical-safety: NOT_REQUIRED · sports-scientist review required.
- `content/post-1874-athletic-identity-35-55.md` — Block 1831–1880 · Cluster 5 · Post 4/10. Тренувальна ідентичність 35–55: три зміни зовнішнього контексту, чотири специфічні поведінкові ризики, тренувальні адаптації, поведінкові переваги зрілого атлета. clinical-safety: NOT_REQUIRED · sports-scientist review required.
- `content/post-1875-from-program-to-principles.md` — Block 1831–1880 · Cluster 5 · Post 5/10. Від програми до принципів: чотири рівні особистої методології (незмінні принципи, персональні параметри, операційні правила, відкриті гіпотези), як методологія формується, чотири помилки при переході. clinical-safety: NOT_REQUIRED · sports-scientist review required.
- `content/post-1876-training-narrative.md` — Block 1831–1880 · Cluster 5 · Post 6/10. Тренувальний наратив: наратив як інтерпретаційна структура, три джерела деструктивних наративів, три найпоширеніші деструктивні наративи і альтернативні фрейми, змінюється через досвід і дані. clinical-safety: NOT_REQUIRED · sports-scientist review required.
- `content/post-1877-progress-comparison-without-external-benchmarks.md` — Block 1831–1880 · Cluster 5 · Post 7/10. Порівняння і прогрес: чотири проблеми соціального порівняння, три рівні внутрішньої системи оцінки, два законних використання зовнішнього порівняння, протокол квартальної самооцінки. clinical-safety: NOT_REQUIRED · sports-scientist review required.
- `content/post-1878-behavioral-maturity-8-signs.md` — Block 1831–1880 · Cluster 5 · Post 8/10. Поведінкова зрілість: 8 ознак зрілого атлета, три спільних принципи (дія на основі даних, системне мислення, реалізм і самосвідомість). clinical-safety: NOT_REQUIRED · sports-scientist review required.
- `content/post-1879-cluster5-synthesis-behavioral-maturity.md` — Block 1831–1880 · Cluster 5 · Post 9/10 · CLUSTER SYNTHESIS. Операційна модель: три рівні зрілої практики (зрілий зв'язок з цілями, функціональний наратив і система оцінки, системна стійкість). Місце Victor у довгостроковій практиці. Практичний чекліст. clinical-safety: NOT_REQUIRED · sports-scientist review required.
- `content/post-1880-block-synthesis-training-behavior-identity.md` — Block 1831–1880 · Block Synthesis · Post 10/10 · BLOCK COMPLETE 50/50. Зведена операційна модель чотирьох рівнів архітектури зрілої практики: ідентичність і наратив, система зворотного зв'язку, операційна система регулярності, довгострокова адаптивність. Місце Victor у кожному рівні. Квартальний чекліст для self-coached атлета. clinical-safety: NOT_REQUIRED · sports-scientist review required.

### Changed
- `blog.html` — 10 нових карток: posts 1871–1880 (Block 1831–1880 · Cluster 5 · BLOCK COMPLETE). Оновлено до post-1880.
- `STATUS.md` — Block 1831–1880 COMPLETE 50/50 · v7.22.0; наступний пріоритет: Block 1881–1930 (тема TBD · product-strategist + sports-scientist).
- `VERSION` — 7.21.1 → 7.22.0.

## [7.21.1] — 2026-06-21

### Changed
- `docs/OBSERVABILITY.md` — v4.8.0 → v4.8.1: §6.2 Consolidated Alert Rules: `enterprise_mid_contract_termination_risk` subsection (AL-ETF-01, P1, `form-customer-success`) and `contract_renewal_health` subsection (AL-RENEW-01, P1, `form-enterprise`) inserted between `sso_browser_security` and `pam_session_health`. §51.7 items 3/4/5 marked `[x] Done`. Closes three P0/M11 obligations from OBSERVABILITY §51.7.
- `docs/SOC2_READINESS.md` — v3.24.3 → v3.24.4: §79.4 master evidence table: `REN-OBS-E-001` row added (CC4.1/A1.1 — quarterly `renewal_notice_check` pg_cron job 39 run history; `renewals/`). §80.4 Vanta mirror list updated to include `REN-OBS-E-001 (OBSERVABILITY §51.5)`. Closes OBSERVABILITY §51.7 item 4.
- `docs/COST_MODEL.md` — v2.10 → v2.11: §42.10 item 4 documentation closure — RENEW-NOTICE-01 monitoring cron marked `[x] Done (documentation)`; implementation decision: pg_cron job 39 (OBSERVABILITY §51.2.1) over original Cloudflare Worker extension spec. Closes OBSERVABILITY §51.7 item 5.
- `VERSION` — 7.21.0 → 7.21.1.

---

## [7.21.0] — 2026-06-21

### Added
- `content/post-2041-program-flexibility-vs-consistency.md` — Editorial 2041–2050 · Block opener. Гнучкість програми vs. послідовність: чотири патерни псевдо-гнучкості (нова інформація, нудьга, чужий прогрес, три тижні плато), нейром'язова фаза адаптації 4–6 тижнів, операційна перевірка з чотирьох питань до будь-якої зміни програми. Heaselgrave et al. 2019, Peterson 2011. clinical-safety: NOT_REQUIRED.

### Changed
- `blog.html` — нова картка post-2041 (Editorial 2041–2050 · Self-Coaching). blog.html оновлено до post-2041.
- `README.md` — roadmap: Editorial 2041–2050 «Хибні спрощення в тренувальній культурі» додано; post-2041 → draft; posts 2042–2050 proposed.
- `STATUS.md` — Editorial 2041–2050 IN PROGRESS 1/10 · v7.21.0; version header оновлено до v7.21.0.
- `VERSION` — 7.20.0 → 7.21.0.

---

## [7.20.0] — 2026-06-21

### Added
- `content/post-1861-regularity-as-system-not-motivation.md` — Block 1831–1880 · Cluster 4 · Post 1/10. Регулярність як системна характеристика: чому мотивація — ненадійна основа. Чотири елементи системи: фіксований розклад, мінімальна версія, порогові умови відпочинку, протокол після пропуску. Кумулятивний ефект регулярності vs. інтенсивності. Deci & Ryan SDT. clinical-safety: NOT_REQUIRED.
- `content/post-1862-training-habit-loop-mechanics.md` — Block 1831–1880 · Cluster 4 · Post 2/10. Механіка habit loop у силовому тренінгу. Duhigg (2012) сигнал→рутина→нагорода. Habit stacking (Clear, 2018). Lally et al. 2010: медіана 66 днів для формування звички. Перші 8–12 тижнів — критична фаза. Ефективні нагороди для тренування. clinical-safety: NOT_REQUIRED.
- `content/post-1863-missed-training-response-protocol.md` — Block 1831–1880 · Cluster 4 · Post 3/10. Пропущене тренування: три типи пропуску і правило «наступного тренування». What-the-hell effect (Polivy & Herman, 1985). Bandura self-efficacy. Серія пропусків як сигнал для перегляду системи. clinical-safety: NOT_REQUIRED.
- `content/post-1864-motivational-cycles-training.md` — Block 1831–1880 · Cluster 4 · Post 4/10. Мотиваційні цикли: три фази (активація, нормалізація, спад). Дофамін і новизна (Schultz, 1998). Три хибних реакції на спад. Нормалізація як місце реального прогресу. clinical-safety: NOT_REQUIRED.
- `content/post-1865-fatigue-vs-demotivation.md` — Block 1831–1880 · Cluster 4 · Post 5/10. Стомленість vs. демотивація: три рівні стомленості (периферійна, ЦНС, системна). Діагностичний протокол: маркери відновлення + тест 10 хвилин + контекст. Протоколи для кожного стану. clinical-safety: NOT_REQUIRED.
- `content/post-1866-minimum-effective-regularity.md` — Block 1831–1880 · Cluster 4 · Post 6/10. Мінімальна ефективна регулярність. Наукова база (Ralston 2017, Schoenfeld 2017): 1–2 сесії з 50–60% об'єму при збереженні інтенсивності підтримують адаптацію. Мінімальна версія тренування як заздалегідь визначений формат. clinical-safety: NOT_REQUIRED.
- `content/post-1867-training-on-a-bad-day.md` — Block 1831–1880 · Cluster 4 · Post 7/10. Тренування у «поганий день»: три сценарії (емоційно важкий / дефіцит сну / хворий стан) і три рішення. Neck check rule (Fitzgerald, 1991). Тест 10 хвилин. clinical-safety: NOT_REQUIRED.
- `content/post-1868-streak-perfectionism-trap.md` — Block 1831–1880 · Cluster 4 · Post 8/10. Стрік і перфекціонізм: де стрік корисний, де стає пасткою. FORM DEC-013: grace після двох пропусків, не одного. Консистентність (відсоток виконаних сесій) vs. неперервна послідовність. Три пастки тренувального перфекціонізму. clinical-safety: NOT_REQUIRED.
- `content/post-1869-long-term-consistency-vs-short-term-optimization.md` — Block 1831–1880 · Cluster 4 · Post 9/10. Довгострокова стійкість vs. короткострокова оптимізація. Кумулятивні адаптаційні горизонти. Три деструктори: травми, часті зміни програми, тривалі паузи. Local vs. global optimum у програмуванні. clinical-safety: NOT_REQUIRED.
- `content/post-1870-cluster4-synthesis-regularity-system.md` — Block 1831–1880 · Cluster 4 · Post 10/10 · CLUSTER COMPLETE. Операційна система регулярності: чотири рівні (архітектура розкладу, протоколи збоїв, управління мотивацією, довгострокова метрика). Роль Victor у системі регулярності. clinical-safety: NOT_REQUIRED.

### Changed
- `blog.html` — 10 нових карток: posts 1861–1870 (Block 1831–1880 · Cluster 4 · Регулярність, звичка та стійкість практики). blog.html оновлено до post-1870.
- `STATUS.md` — Block 1831–1880 Cluster 4 COMPLETE 10/10 · v7.20.0; next: Cluster 5 (posts 1871–1880, тема TBD).
- `VERSION` — 7.19.1 → 7.20.0.

---

## [7.19.1] — 2026-06-21

### Added
- `admin-dashboard.html` — "Privileged Access Activity" card added to the Security & Compliance tab (OBSERVABILITY.md §29.13 checklist item 2 · P1/M5). Card elements: heading + sub-heading ("time-limited, two-person approved, fully audited"); primary stat `privilegedOperationsCount` (large numeral); secondary stat break-glass activations with green dot + "None" at zero, amber dot + count + tooltip at >0; period selector (current month, previous 3 calendar months); explanatory PAM mechanics note (two-person approval, 4 h session cap, DEC-030 HMAC chain, 72 h post-hoc review obligation); footer link to `/security.html#privileged-access`. Mock data covers June–March 2026 including one break-glass event in April to exercise the amber state. Period-selector JS added: `updatePamActivity()` re-renders both stats on change. Privacy floor enforced by design: no `admin_user_id`, no session ID, no query content — only aggregate counts per period. Positioned above the existing "Break-glass & support access · last 90 days" card. Ref: OBSERVABILITY.md §29.13.3 (backend SQL), §29.13.4 (privacy invariants), §29.13.5 (placement + copy), §29.13.7 (SOC 2 CC6.1 / C1.2 mapping). Owner: design-craft + platform-engineer + compliance-officer.

### Changed
- `docs/OBSERVABILITY.md §29.13.8` — checklist item 2 marked `[x] Done — 2026-06-21, admin-dashboard.html v0.48.0`.
- `docs/DATA_MODEL.md §31` — checklist item 10 (DSAR status widget) marked `[x] Done — 2026-06-21, admin-dashboard.html v0.48.0`. Widget was present in dashboard but checklist not updated; cross-reference closure only — no HTML change required.
- `VERSION` — 7.19.0 → 7.19.1.

---

## [7.19.0] — 2026-06-21

### Added
- `content/post-2031-technology-vs-decision.md` — Editorial 2031–2040 · Post 1/10. Технологія не замінює рішення: де AI-додатки допомагають self-coached атлету — і де безсилі. Три категорії тренувальних рішень (структурні / тактичні / оперативні) і роль алгоритму в кожній. Чому «персоналізація» у більшості додатків — шаблон з параметрами. Фундаментальна межа технології: якість руху, контекст поза даними, нейром'язовий стан у реальному часі. Що FORM будує і де чесні про обмеження. clinical-safety: NOT_REQUIRED. sports-scientist review: not required.

### Changed
- `blog.html` — post-2031 card added (Editorial 2031–2040 · Post 1/10). Category: Training Methodology.
- `README.md` — Editorial roadmap extended: posts 2031–2040 proposed (cluster: Technology & self-coached practice). post-2031 → draft.
- `STATUS.md` — Editorial 2031–2040 row added: IN PROGRESS 1/10 · v7.19.0.
- `VERSION` — 7.18.0 → 7.19.0.

---

## [7.18.0] — 2026-06-21

### Added
- `content/post-1852-social-comparison-in-gym.md` — Block 1831–1880 · Cluster 3 · Post 2/10. Соціальне порівняння в залі: чому зал конструктивно провокує порівняння (схожість задачі + видимість результату). Upward vs. downward comparison — механіка за Festinger (1954). Де порівняння замінює аналіз власних даних (ego-lifting, ваговий стрибок, знецінення прогресу). Протокол ізоляції від порівняльного шуму через конкретний план і журнал. clinical-safety: NOT_REQUIRED.
- `content/post-1853-social-media-training-noise.md` — Block 1831–1880 · Cluster 3 · Post 3/10. Positive selection bias і illusion of consensus у фітнес-контенті. Три категорії медіа-впливу: методологічний шум, обсяговий тиск, тімінговий тиск. Операційний критерій «корисна інформація vs. шум»: правило 24 годин без підтверджуючих власних даних. Куратура замість пасивного споживання. clinical-safety: NOT_REQUIRED.
- `content/post-1854-training-partner-structure-vs-dependency.md` — Block 1831–1880 · Cluster 3 · Post 4/10. Що тренувальний партнер реально дає (регулярність за Carron et al. 2003, страховка, соціальний контекст). Де партнерство стає проблемою: розклад-залежність, компроміс програми, ефект темпу, соціальна механіка за рахунок тренувальної. Чотири принципи структури без залежності. clinical-safety: NOT_REQUIRED.
- `content/post-1855-group-training-individual-plan.md` — Block 1831–1880 · Cluster 3 · Post 5/10. Три механіки групового тиску на індивідуальну програму (соціальний темп, конкуренція, зміна пріоритетів). Три ролі групового формату: основа / допоміжний / ситуативний. Практика збереження власних координат у WOD і командному тренінгу. clinical-safety: NOT_REQUIRED.
- `content/post-1856-competitive-context-without-competitive-goal.md` — Block 1831–1880 · Cluster 3 · Post 6/10. Коли конкуренція є корисним інструментом (social facilitation, Zajonc 1965) і коли стає проблемою: ego-lifting, перебудова пріоритетів, хибний прогрес. Перевірка перед ваговим рішенням. Конкуренція з собою як базова форма для self-coached атлета. clinical-safety: NOT_REQUIRED.
- `content/post-1857-social-accountability-without-external-pressure.md` — Block 1831–1880 · Cluster 3 · Post 7/10. Три типи підзвітності і стійкість кожного. Мінімальний ефективний протокол (Локвуд і Кролич 2008): одна людина, одна метрика, раз на тиждень. Помилки: без метрики, без наслідків, як соціальна гра, як замінник програми. Альтернативи без живого партнера. clinical-safety: NOT_REQUIRED.
- `content/post-1858-when-social-environment-overrides-plan.md` — Block 1831–1880 · Cluster 3 · Post 8/10. Тренувальний дрейф: визначення, чотири сигнали, три діагностичних питання. Відновлення без «починати спочатку»: фіксація стану → порівняння з планом → рішення → нова точка старту. Превенція через конкретний план і тижневий огляд. clinical-safety: NOT_REQUIRED.
- `content/post-1859-designing-training-environment.md` — Block 1831–1880 · Cluster 3 · Post 9/10. П'ять рівнів тренувального середовища за Thaler & Sunstein (2008) choice architecture: фізичний простір, часова структура, соціальний контекст, цифрове середовище, підготовка. Практичний quarterly аудит. Чому дизайн середовища надійніший за мотивацію. clinical-safety: NOT_REQUIRED.
- `content/post-1860-cluster3-synthesis-individual-practice-social-context.md` — Block 1831–1880 · Cluster 3 · Post 10/10 · CLUSTER SYNTHESIS. Синтез постів 1851–1860. Операційна модель трьох рівнів: фільтрація шуму, свідоме використання соціальних факторів, дизайн середовища. Реактивний vs. проактивний режим. Тижневий дрейф-чек, середовищний аудит Q, мінімальна підзвітність. Місце Victor як антишумового референсу. clinical-safety: NOT_REQUIRED.

### Changed
- `blog.html` — posts 1852–1860 added (Cluster 3 · Block 1831–1880). Total posts displayed: 1860.
- `STATUS.md` — Block 1831–1880 updated: IN PROGRESS 21/50 → 30/50 · Cluster 3 COMPLETE · v7.18.0. Next: Cluster 4 posts 1861–1870.
- `VERSION` — 7.17.1 → 7.18.0.

---

## [7.17.1] — 2026-06-21

### Changed
- `docs/OBSERVABILITY.md` — v4.7.3 → v4.8.0. §51 Enterprise Contract Renewal Monitoring Observability (RENEW-NOTICE-01): pg_cron job 39 `renewal_notice_check` (daily 09:00 UTC, 26h freshness); alert AL-RENEW-01 (P1, PagerDuty `form-enterprise`, dedup 72h); DEC-030 events `enterprise.renewal_notice_overdue` HIGH/7yr and `system.renewal_notice_check_passed` LOW/1yr; SOC 2 evidence REN-OBS-E-001 (CC4.1/A1.1) and REN-E-002 (CC4.1/CC2.2); §6.2 `contract_renewal_health` subsection; seven-item checklist. §12.6 pg_cron registry updated: job 39 row added; freshness note updated; v0.6 patch note. Closes COST_MODEL §42.10 item 4 documentation obligation (RENEW-NOTICE-01 not previously registered in OBSERVABILITY.md).
- `VERSION` — 7.17.0 → 7.17.1.

---

## [7.17.0] — 2026-06-21

### Added
- `content/post-1851-social-influence-on-training-behavior.md` — Block 1831–1880 · Cluster 3 · Post 1/10. Три механізми соціального впливу на тренувальну поведінку: соціальні норми (Cialdini 2003), соціальне порівняння (Festinger 1954), поведінкова контагія (Christakis & Fowler 2009). Чому self-coached атлет вразливіший до соціального шуму без зовнішньої структури тренера. Протокол щомісячного аудиту середовища — 4 питання. clinical-safety: NOT_REQUIRED.

### Changed
- `blog.html` — post-1851 card added; now current to post-1851.
- `README.md` — post-1851 status: `proposed` → `draft · v7.17.0`; title updated.
- `STATUS.md` — Block 1831–1880: IN PROGRESS 20/50 → 21/50; Cluster 3 IN PROGRESS 1/10 noted; Next updated to post-1852.
- `VERSION` — 7.16.0 → 7.17.0.

---

## [7.16.0] — 2026-06-21

### Added
- `content/post-1841-reading-own-training-feedback.md` — Block 1831–1880 · Cluster 2 · Post 1/10. Три джерела тренувального відгуку (суб'єктивне / об'єктивне / поведінковий патерн), ієрархія надійності, три типи помилок читання, чотири питання після тренування, Victor як зовнішня пам'ять. clinical-safety: NOT_REQUIRED.
- `content/post-1842-when-looks-wrong-is-functionally-acceptable.md` — Block 1831–1880 · Cluster 2 · Post 2/10. Звідки береться «ідеальна модель» техніки і чому вона не враховує морфологію (McGill, Frost). Три категорії «виглядає не так»: анатомічна варіація / компенсаторний патерн / функціональний збій. Чотирикроковий алгоритм оцінки. clinical-safety: NOT_REQUIRED.
- `content/post-1843-typical-self-assessment-mistakes-strength.md` — Block 1831–1880 · Cluster 2 · Post 3/10. Шість системних помилок самооцінки: recency bias, подвійний стандарт, конфлікт «хотілось / підняв», неправильна референтна точка, переоцінка суб'єктивного при накопиченій втомі, ігнорування поведінкового патерну. clinical-safety: NOT_REQUIRED.
- `content/post-1844-video-analysis-own-movement-methodology.md` — Block 1831–1880 · Cluster 2 · Post 4/10. Мінімальний протокол запису (що і коли знімати), кути камери, контрольні маркери для присяду / станового / жиму, чотирикроковий алгоритм аналізу, помилки відеоаналізу. clinical-safety: NOT_REQUIRED.
- `content/post-1845-cognitive-bias-training-confirmation-status-quo.md` — Block 1831–1880 · Cluster 2 · Post 5/10. Confirmation bias (Nickerson 1998) і status quo bias (Samuelson & Zeckhauser 1988, Kahneman & Tversky 1979) у тренуванні: прояви, чому небезпечні без тренера, протоколи протидії. clinical-safety: NOT_REQUIRED.
- `content/post-1846-when-self-reflection-helps-vs-hinders.md` — Block 1831–1880 · Cluster 2 · Post 6/10. Де саморефлексія продуктивна, де заважає (internal focus під час руху — Wulf et al.), руминація vs аналіз, чотирирівневий ритм рефлексії. clinical-safety: NOT_REQUIRED.
- `content/post-1847-feedback-loop-self-coached-athlete.md` — Block 1831–1880 · Cluster 2 · Post 7/10. Чотири компоненти замкненої feedback loop: збір даних → аналіз → рішення → виконання. Де петля найчастіше розривається. Часові горизонти і типи рішень (таблиця). Victor як структура петлі. clinical-safety: NOT_REQUIRED.
- `content/post-1848-training-journal-as-temporal-feedback.md` — Block 1831–1880 · Cluster 2 · Post 8/10. Три функції журналу, мінімум vs оптимум запису, формати, як читати (тижневий / мезоцикловий / «рік тому» перегляд). clinical-safety: NOT_REQUIRED.
- `content/post-1849-five-questions-after-training-block.md` — Block 1831–1880 · Cluster 2 · Post 9/10. П'ять питань мезоциклової ретроспективи: що прогресувало / що не прогресувало і чому / оптимальне навантаження / що зробив би інакше / одна зміна для наступного блоку. Коли проводити. clinical-safety: NOT_REQUIRED.
- `content/post-1850-cluster2-synthesis-self-assessment-system.md` — Block 1831–1880 · Cluster 2 · Post 10/10 · CLUSTER SYNTHESIS. П'ять рівнів операційної системи самооцінки: запис після тренування → тижневий аналіз → відеоаналіз → когнітивна корекція → мезоциклова ретроспектива. Карта системи. Що система не замінює. Victor в архітектурі системи. clinical-safety: NOT_REQUIRED.

### Changed
- `blog.html` — posts 1841–1850 added (Cluster 2 · Самооцінка без тренера); now current to post-1850.
- `README.md` — posts 1841–1850 status: `proposed` → `draft · v7.16.0`.
- `STATUS.md` — Block 1831–1880: IN PROGRESS 10/50 → 20/50; Cluster 2 COMPLETE noted; Next updated to Cluster 3.
- `VERSION` — 7.15.2 → 7.16.0.

---

## [7.15.2] — 2026-06-21

### Changed
- `docs/SOC2_READINESS.md` — §98.5 item 3 cross-reference patch: `compliance/fieldwork/scim-bulk-deprovision-guard.md` fieldwork guide (authored 2026-06-20, indexed in `compliance/evidence/auditor-onboarding/README.md` v1.1) was reflected in `docs/SSO_SCIM_IMPLEMENTATION.md §35.9` item 5 ([x] Done — 2026-06-20, v2.8.1) but not in SOC2_READINESS §98.5. This patch closes the one-document gap: §98.5 item 3 `[ ]` → `[x] Done — 2026-06-20`. Document header v3.24.2 → v3.24.3 + v3.24.3 version footer added. No new DDL, DEC-030 events, or evidence artefact IDs.
- `VERSION` — 7.15.1 → 7.15.2.

---

## [7.15.1] — 2026-06-21

### Added
- `content/post-45-progress-beyond-barbell-weight.md` — Editorial · training-methodology. Прогрес без ваги штанги: що ще рахується і чому. Шість паралельних метрик адаптації (повтори при тому ж RPE, технічна стабільність, відновлення між підходами, швидкість штанги, об'ємна ємність, консистентність). Алгоритм діагнозу для self-coached атлета, коли штанга «стоїть». Refs: Schoenfeld 2010, Atherton & Rennie 2009, Bazuelo-Ruiz et al. 2015, Barker et al. 2010. clinical-safety: NOT_REQUIRED.

### Changed
- `blog.html` — картка post-45 додана (Training Methodology · 13 хв · draft · 2026-06-21). blog.html актуальний до post-45.
- `README.md` — post-45 → draft · v7.15.1.
- `STATUS.md` — 11–50 row: post-45 entry added · v7.15.1. Version header updated to v7.15.1.
- `VERSION` — 7.15.0 → 7.15.1.

---

## [7.15.0] — 2026-06-21

### Added
- `content/post-1832-habit-vs-decision-training-behavior.md` — Block 1831–1880 · Cluster 1 · Post 2/10. Звичка проти рішення: поведінкова механіка регулярного тренування. Habit loop і базальні ганглії (Graybiel 2008), decision fatigue (Baumeister 1998), SRHI automaticity index (Verplanken & Orbell 2003), контекстна специфічність звички (Wood et al. 2002, 2005). Victor як зовнішня cue-структура. clinical-safety: NOT_REQUIRED.
- `content/post-1833-environment-behavioral-design-training.md` — Block 1831–1880 · Cluster 1 · Post 3/10. Середовище і поведінковий дизайн. Nudge theory (Thaler & Sunstein 2008), context-dependent habits (Wood, Neal & Drolet 2013), friction reduction vs willpower (Fogg 2019). Victor як структурований prompt замість відкритого питання. clinical-safety: NOT_REQUIRED.
- `content/post-1834-self-talk-internal-narrative-athlete.md` — Block 1831–1880 · Cluster 1 · Post 4/10. Внутрішній монолог атлета. Instructional vs motivational self-talk meta-analysis (Hatzigeorgiadis et al. 2011, ESr=0.48). «Possible selves» і поведінка (Markus & Nurius 1986). Три мовні пастки. Журнал як поведінковий архів ідентичності. clinical-safety: NOT_REQUIRED.
- `content/post-1835-social-accountability-vs-autonomous-motivation.md` — Block 1831–1880 · Cluster 1 · Post 5/10. Зовнішня підзвітність vs внутрішня мотивація. SDT (Ryan & Deci 2000), Deci et al. 1999 (128 studies), Kohler effect (Feltz et al. 2011). Victor як інфраструктура, не джерело мотивації. clinical-safety: NOT_REQUIRED.
- `content/post-1836-perfectionism-training-trap.md` — Block 1831–1880 · Cluster 1 · Post 6/10. Перфекціонізм і тренування. Hewitt & Flett 1991 (три виміри), adaptive vs maladaptive perfectionism in sport (Gotwals et al. 2012). All-or-nothing thinking. Умовна ідентичність = крихка конструкція. clinical-safety: NOT_REQUIRED.
- `content/post-1837-return-after-break-identity-recovery.md` — Block 1831–1880 · Cluster 1 · Post 7/10. Повернення після перерви. Abstinence violation effect (Marlatt & Gordon 1985). Детренованість: сила тримається 3–4 тижні (Mujika & Padilla 2000), м'язова пам'ять (Bruusgaard et al. 2010). DEC-013 (streak grace). Протокол повернення. clinical-safety: NOT_REQUIRED.
- `content/post-1838-goal-vs-process-orientation-training.md` — Block 1831–1880 · Cluster 1 · Post 8/10. Ціль проти процесу. Task vs ego orientation (Nicholls 1984, 1989), три рівні цілей (Hardy, Jones & Gould 1996). Victor відстежує процесні метрики, не тільки 1RM. clinical-safety: NOT_REQUIRED.
- `content/post-1839-athletic-identity-age-35-55.md` — Block 1831–1880 · Cluster 1 · Post 9/10. Ідентичність і вік 35–55. Stambulova Career Transition Model, Houle et al. 2010, Dionigi 2006 (masters athletes). Криза vs трансформація ідентичності. Відновлення як компетентність — не управління спадом. clinical-safety: NOT_REQUIRED.
- `content/post-1840-cluster1-synthesis-athletic-identity-architecture.md` — Block 1831–1880 · Cluster 1 · Post 10/10 · CLUSTER SYNTHESIS. Поведінковий стек 5 рівнів. Аудит кожного рівня (5 діагностичних питань). Чотири захисні рейки. Роль Victor. Анонс Cluster 2. clinical-safety: NOT_REQUIRED.

### Changed
- `blog.html` — 9 нових карток постів 1832–1840 додано (category: Training Behavior). blog.html актуальний до post-1840.
- `STATUS.md` — Block 1831–1880: IN PROGRESS 1/50 → IN PROGRESS 10/50; Cluster 1 COMPLETE 10/10 · v7.15.0. Наступний: Cluster 2 posts 1841–1850.
- `VERSION` — 7.14.1 → 7.15.0.

---

## [7.14.1] — 2026-06-21

### Changed
- `docs/DATA_MODEL.md` — §40.9 cross-reference patch: OQ-SIEM-03 updated from 🟡 Open P2 to 🟢 Resolved DEC-071 (2026-06-19). DEC-071 resolved this in `docs/OBSERVABILITY.md §50` (v4.7.0) but §40.9 was not included in that patch's cross-update sweep. Document header v1.23 → v1.24.
- `VERSION` — 7.14.0 → 7.14.1.

---

## [7.14.0] — 2026-06-21

### Added
- `content/post-1831-athletic-identity-vs-motivation.md` — Block 1831–1880 · Cluster 1 · Post 1/10. Атлетична ідентичність vs. мотивація: Brewer, Van Raalte & Linder 1993 (AIMS); ego depletion Baumeister et al. 1998; implementation intentions Gollwitzer 1999; habit formation 66-day median Lally et al. 2010; self-efficacy Bandura 1997; мінімальна доза як якір ідентичності Bickel et al. 2011; три помилки ідентичності; 4 практичних кроки. clinical-safety NOT_REQUIRED.

### Changed
- `blog.html` — додано картку для post-1831 (Block 1831–1880 Cluster 1).
- `README.md` — post-1831 → draft · v7.14.0; Block 1981–2030 «Реальний атлет у реальному житті: тренування поза оптимальними умовами» (50 proposed posts) added.
- `STATUS.md` — Block 1831–1880 IN PROGRESS 1/50; version header + footer updated to v7.14.0.
- `VERSION` — 7.13.0 → 7.14.0.

---

## [7.13.0] — 2026-06-21

### Added
- `content/post-1822-two-weeks-poor-recovery-diagnosis.md` — Cluster 5 Post 2/10. Два тижні поганого відновлення поспіль. Переоцінка через три запитання. Чотири рівні інтервенції: деловантажний тиждень → тиждень без тренувань → медична оцінка → структурний перегляд. Маркери безпечного виходу. Протокол поступового повернення (тижні 1–4). clinical-safety PASS.
- `content/post-1823-chronic-recovery-impairment-new-baseline.md` — Cluster 5 Post 3/10. Хронічне порушення відновлення: нова базова лінія. Відмінність «тривала яма» vs «нова реальність». Протокол формування нової бази (21-денна стабілізація, медіана). Перерахунок зон. Адаптація об'єму, частоти, деловантажу. Коли до лікаря. clinical-safety PASS.
- `content/post-1824-training-during-illness-return-protocol.md` — Cluster 5 Post 4/10. Тренування при захворюванні та повернення. Правило «вище/нижче шиї». Три стадії хвороби. Протоколи повернення за тривалістю: 3–5 днів / 1–2 тижні / 3+ тижні. Особливості COVID і кишкових інфекцій. Реальні втрати результатів. clinical-safety PASS.
- `content/post-1825-recovery-reduced-energy-availability.md` — Cluster 5 Post 5/10. Відновлення при зниженій ЕД. Механізми: синтез білка, глікоген, гормональний фон, HRV при дефіциті. Адаптація об'єму (-15–25%), пріоритет інтенсивності. RED-S — попереджувальні сигнали та ескалація до лікаря. clinical-safety CONDITIONAL PASS.
- `content/post-1826-recovery-after-competition-cycle.md` — Cluster 5 Post 6/10. Відновлення після змагального циклу. Три фази: пасивне (дні 1–7), реактивація (тижні 2–3), повернення до об'єму (тиждень 4+). Що відбувається з ЦНС, м'язами, психологічним станом. Таймлайн-таблиця. clinical-safety PASS.
- `content/post-1827-recovery-with-structurally-limited-sleep.md` — Cluster 5 Post 7/10. Відновлення при структурно обмеженому сні. MRV знижується на 20–30%. Три адаптації: знизити об'єм, зберегти інтенсивність, збільшити відновлення між сесіями. Оптимізація якості сну. Протоколи при новонародженій дитині і нічних змінах. clinical-safety PASS.
- `content/post-1828-recovery-35-55-age-related-changes.md` — Cluster 5 Post 8/10. Відновлення 35–55: вікові зміни. Гормональний фон (тестостерон, ГР, кортизол), нейром'язова система, сполучна тканина. Практичні адаптації: частіший деловантаж (3–4 тижні), 72–96 год між важкими сесіями, пріоритет сну. DEC-068. clinical-safety PASS.
- `content/post-1829-atypical-recovery-markers-your-norm.md` — Cluster 5 Post 9/10. Атипові маркери відновлення. Три типи атипового атлета: хронічно низький HRV, високий HRV при поганому стані, непередбачувані коливання. Чому популяційна норма ≠ ваша норма. Протокол персональної бази (21 день, медіана). Відносні зони. clinical-safety PASS.
- `content/post-1830-block-synthesis-recovery-management.md` — Cluster 5 Post 10/10 · BLOCK SYNTHESIS 1781–1830. Операційна система відновлення: 6 принципів, зведена таблиця маркерів, алгоритм швидкого рішення для 9 нестандартних ситуацій, синтез 5 кластерів. Коли виходити за межі самоуправління. Victor як інтегрована система. **BLOCK 1781–1830 COMPLETE 50/50.** clinical-safety PASS.

### Changed
- `blog.html` — додано картки для posts 1822–1830.
- `STATUS.md` — Block 1781–1830 оновлено до COMPLETE 50/50 · v7.13.0; Cluster 5 COMPLETE 10/10; version footer оновлено до v7.13.0.
- `VERSION` — 7.12.0 → 7.13.0.

---

## [7.12.0] — 2026-06-21

### Added
- `content/post-1821-one-week-poor-recovery-algorithm.md` — Cluster 5 Post 1/10. Один тиждень поганого відновлення: алгоритм без паніки. Операційне визначення «поганого тижня» (≥3 з 6 маркерів протягом 4+ днів). Три типи причин: Тип А (тренувальне перевантаження -25–35%), Тип Б (зовнішній стрес -30–40%), Тип В (комбінований -40–50%). Алгоритм-дерево рішень. Моніторинг: HRV-тренд, RPE-дрейф, SRQ. Що НЕ робити. Коли передати до пост-1822 (two-week protocol). clinical-safety PASS.

### Changed
- `blog.html` — додано картку для post-1821.
- `README.md` — додано Block 1931–1980 «Програмування для перехідного атлета: від intermediate до advanced» (50 proposed posts) · v7.12.0.
- `VERSION` — 7.11.0 → 7.12.0.

---

## [7.11.0] — 2026-06-21

### Added
- `content/post-1812-recovery-travel-jet-lag.md` — Cluster 4 Post 2/10. Циркадна біологія і джетлаг: SCN, Zeitgebers, нейром'язовий добовий пік. Вплив на SWS і HRV (Atkinson & Reilly). Схід→захід vs. захід→схід. Протокол до/під час/після перельоту. Тренування в готелі параметрами таблицею. HRV-реакліматизація. clinical-safety PASS.
- `content/post-1813-recovery-pre-competition-peak-phase.md` — Cluster 4 Post 3/10. Передзмагальний тейпер: нейром'язова відповідь, метааналіз Bosquet et al. (2007) +3%. Тривалість 7–14 днів. Формула: -40–50% об'єму, збережена інтенсивність. Типові помилки тейпера. HRV як навігатор. Активаційна сесія за 1–2 дні. clinical-safety PASS.
- `content/post-1814-recovery-irregular-schedule-shift-work.md` — Cluster 4 Post 4/10. Хронічний соціальний джетлаг і ротаційний графік. ГР і SWS при денному сні. Типи графіків: фіксована ніч vs. ротація. Базові принципи: стабільність > оптимальність. Таблиця корекцій по типу тижня. Динамічна базова лінія HRV. clinical-safety PASS.
- `content/post-1815-recovery-concurrent-training.md` — Cluster 4 Post 5/10. Interference effect: AMPK/mTORC1 (Hickson 1980, Wilson et al. 2012). Часовий інтервал 6+ год. Порядок сесій. Тип кардіо і рівень interference. Об'єм аеробного ≤90 хв/тиж. clinical-safety PASS.
- `content/post-1816-recovery-heat-environment.md` — Cluster 4 Post 6/10. Терморегуляція і конкуренція за серцевий викид. Нейром'язова функція при гіпертермії. Дегідратація −2% і сила (Judelson et al.). Теплова акліматизація 10–14 днів. Pre-cooling, сон при спеці, HRV перекалібрування. clinical-safety PASS.
- `content/post-1817-recovery-minimum-effective-volume.md` — Cluster 4 Post 7/10. MEV: 1 сесія 2–3 підходи ≥70% 1ПМ зберігає адаптацію 8–12 тижнів (Bickel et al., 2011). Структура MEV-блоків А і Б. Повернення до повного об'єму поетапно. Інтенсивність важливіша за об'єм у MEV. clinical-safety PASS.
- `content/post-1818-recovery-functional-overreaching.md` — Cluster 4 Post 8/10. Спектр FOR/NFOR/OTS. Маркери FOR. Поетапний протокол: Фаза 1 (MEV/2 тижні) → Фаза 2 (50%) → Фаза 3 (70–80%). Ранні сигнали попередження. OTS виходить за межі самостійного управління — зазначено явно. clinical-safety PASS.
- `content/post-1819-recovery-knowledge-worker-cognitive-load.md` — Cluster 4 Post 9/10. Когнітивна стомленість: глутамат у dlPFC (Wiehler et al., 2022). RPE при когнітивній стомленості (Marcora et al., 2009). Технічна якість і центральна відмова. Стратегії knowledge worker: ранкові сесії, пауза перед тренуванням, тижнева структура. clinical-safety PASS.
- `content/post-1820-cluster4-synthesis-context-specific-recovery.md` — Cluster 4 Post 10/10 SYNTHESIS. Операційна карта 9 контекстів (таблиця). Ієрархія корекцій. Що ніколи не знижується. Таблиця інтерпретації HRV/RPE за контекстом. Тимчасові vs. структурні корекції. Cluster 4 одним реченням на контекст. Анонс Cluster 5. clinical-safety PASS.

### Changed
- `blog.html` — додано пости 1812–1820.
- `VERSION` — 7.10.1 → 7.11.0.

---

## [7.10.1] — 2026-06-21

### Changed
- `docs/SOC2_READINESS.md` — v3.24.1 → v3.24.2: §79.4 master evidence table — added REN-E-001 (CC5.2/CC1.4 · annual `enterprise.contract_renewed` chain export + RENEW-CHAIN-01 zero-row query), REN-E-002 (CC4.1/CC2.2 · quarterly `enterprise.renewal_notice_sent` export), REN-E-003 (CC4.1/A1.1 · annual renewal conversion rate cross-tab). §80.3 R2 folder structure — `renewals/` subfolder added with `form-api NO ACCESS`. §80.4 Vanta mirror list — `REN-E-001/002/003 (§42.9)` appended. §100.6 item 2 documentation portion marked done. Closes three cross-reference obligations: COST_MODEL §42.10 item 5 (documentation portion), DATA_MODEL §43.8 item 4 (documentation portion), SOC2_READINESS §100.6 item 2 (documentation portion).
- `docs/COST_MODEL.md` — v2.9 → v2.10: §42.10 item 5 status updated (§79.4 registration done; first evidence filing pending first renewal cycle est. M12).
- `docs/DATA_MODEL.md` — v1.22 → v1.23: §43.8 item 4 status updated (§79.4 registration done; first evidence filing pending first renewal cycle est. M12).
- `VERSION` — 7.10.0 → 7.10.1.

---

## [7.10.0] — 2026-06-21

### Added
- `content/post-1811-recovery-under-external-stress-allostatic-load.md` — Cluster 4 · Post 1/10. Відновлення під зовнішнім стресом: аллостатичне навантаження (McEwen 1998, 2008) і спільна HPA-вісь для тренувального і позатренувального стресу. Що змінюється в даних при підвищеному life stress: HRV-тренд, RPE-дрейф, технічна деградація. Алгоритм корекції: три рівні стресу → три глибини зниження об'єму (20–25% / 35–40% / 50%+). Часові горизонти різних стресорів. Що не скорочувати (частота, інтенсивність основних рухів). Victor і контекстне читання навантаження через HRV + RPE + SRQ. clinical-safety: PASS.

### Changed
- `blog.html` — додано картку для post-1811. Оновлено до post-1811.
- `VERSION` — 7.9.0 → 7.10.0.
- `README.md` — додано редакційну серію 271–280 (Exercise physiology deep dives, vol. 2) у roadmap.
- `STATUS.md` — оновлено рядок Block 1781–1830: Cluster 4 розпочато (post-1811 · Відновлення в специфічних контекстах).

---

## [7.9.0] — 2026-06-21

### Added
- `content/post-1802-deload-training-journal.md` — Cluster 3 · Post 2/10. Як виглядає повноцінний деловантаж у тренувальному журналі: три змінні (об'єм, інтенсивність, частота), числа 40–60% від пікового тижня, конкретний 4-денний спліт пік vs. деловантаж із вагами і RPE, що фіксувати, три помилки, коли подовжувати деловантаж. clinical-safety: PASS.
- `content/post-1803-transition-block-between-mesocycles.md` — Cluster 3 · Post 3/10. Перехідний блок між мезоциклами: три сценарії, дві функції (відновлення + підготовка), тривалість по типу блоку (7–21 день), структура, конкретний 12-денний приклад після інтенсифікаційного мезоциклу, відмінність від деловантажного тижня. clinical-safety: PASS.
- `content/post-1804-active-recovery-within-week.md` — Cluster 3 · Post 4/10. Активне відновлення всередині тижня: механізми (кровоток, нейром'язовий тонус, суб'єктивний стан), коли виправдане і коли шкодить, що рахується активним відновленням (ходьба пульс 90–110), практична структура тижня. clinical-safety: PASS.
- `content/post-1805-mobility-tissue-work-recovery.md` — Cluster 3 · Post 5/10. Мобільність і тканинна робота: foam rolling підтверджені ефекти (DOMS, ROM тимчасово) і спростовані (фасціальні спайки), статичний стретчинг перед важкими сесіями — ризик зниження показників, чотири ніші застосування, мінімальний протокол, коли до фахівця. clinical-safety: PASS.
- `content/post-1806-cold-heat-therapy-recovery.md` — Cluster 3 · Post 6/10. Холодна і теплова терапія: CWI підтверджені ефекти і критичне застереження (пригнічення mTOR-адаптації при гіпертрофії), контрастний душ як компроміс, сауна (HSP, взаємодія з гіпертрофією), порівняльна таблиця, практичний вибір. clinical-safety: PASS.
- `content/post-1807-sleep-recovery-protocols.md` — Cluster 3 · Post 7/10. Сон як інструмент відновлення: цільовий діапазон 8–9 годин, три ключові фізіологічні ефекти (ГР-пік, моторна консолідація REM, нейром'язовий відпочинок), чотири поведінкові стратегії (розклад, температура 18–20°C, світло, кофеїн до 14:00), вечірні тренування і буфер 60–90 хв, борг сону. clinical-safety: PASS.
- `content/post-1808-recovery-after-maximal-effort.md` — Cluster 3 · Post 8/10. Відновлення після максимальних зусиль: нейром'язовий борг vs. метаболічне стомлення, часовий горизонт по типу зусиль (таблиця), протокол дні 1–3/4–7/тиждень 2, маркери готовності, різниця тестування і змагань, підготовка до тестування. clinical-safety: PASS.
- `content/post-1809-forced-training-pause-recovery.md` — Cluster 3 · Post 9/10. Вимушена пауза: фізіологія детренованості (нейром'язова, м'язова маса, 1ПМ — часові горизонти), три типи пауз і протоколи повернення, специфіка хвороби (48 годин безсимптомного), чотири помилки повернення, таблиця повернення по тривалості паузи. clinical-safety: PASS.
- `content/post-1810-cluster3-synthesis-recovery-protocols.md` — Cluster 3 · Post 10/10 · **CLUSTER 3 COMPLETE**. Синтез: ієрархія інструментів відновлення (рівні 1–3), матриця рішень (ситуація → протокол), три основні помилки Cluster 3, мінімальна операційна система, ключові числа-довідник. clinical-safety: PASS.

### Changed
- `blog.html` — додано записи для постів 1802–1810. Оновлено до post-1810.
- `VERSION` — 7.8.1 → 7.9.0.

---

## [7.8.1] — 2026-06-21

### Changed
- `docs/COST_MODEL.md` — v2.8 → v2.9: marked §35.10 item 1, §37.10 item 1, §38.10 item 1, and §39.10 item 1 as `[x] Done`, syncing checklist state with AUDIT_LOG_SCHEMA.md registrations already completed in v1.7/v2.3/v2.7/v2.9 (2026-06-12 through 2026-06-15). No model or schema changes.
- `VERSION` — 7.8.0 → 7.8.1.

---

## [7.8.0] — 2026-06-21

### Added
- `content/post-1801-planned-vs-reactive-deload.md` — Cluster 3 · Post 1/10 · Block 1781–1830. Плановий деловантаж vs. реактивний деловантаж: фізіологічна логіка кожного, практичні відмінності (тривалість блоку, що знижується, хто ухвалює рішення), три типові помилки кожного підходу, комбінований підхід, зв'язок із системою маркерів Cluster 2. clinical-safety: PASS.

### Changed
- `blog.html` — post-1801 card додано в каталог.
- `README.md` — post-1801 → draft · v7.8.0.
- `VERSION` — 7.7.0 → 7.8.0.

---

## [7.7.0] — 2026-06-21

### Added
- `content/post-1792-resting-hr-recovery-marker.md` — Cluster 2 · Post 2/10 · Block 1781–1830. ЧСС у спокої як маркер відновлення: фізіологічна логіка (АНС/парасимпатична активність), протокол вимірювання (лежачи, до кофеїну), чотири типи сигналів тренду, амплітуда відхилень (+5–7 уд/хв / 3+ дні), ЧСС у спокої vs HRV. clinical-safety: NOT_REQUIRED.
- `content/post-1793-subjective-readiness-scale-srq.md` — Cluster 2 · Post 3/10 · Block 1781–1830. SRQ: чотири запитання (Kellmann & Kallus 2001) і спрощена версія (3×10). Чому суб'єктивне випереджає HRV (Saw et al. 2016): інтеграція всіх стресорів, специфічна м'язова стомленість. Коли ненадійне: звикання до навантаження, когнітивний дисонанс. Патерни: системно низька мотивація, розрив між фізичним і ментальним. clinical-safety: NOT_REQUIRED.
- `content/post-1794-sleep-recovery-marker.md` — Cluster 2 · Post 4/10 · Block 1781–1830. Сон як маркер: SWS і гормон росту, REM і консолідація навичок. Кількість vs якість як окремі параметри. Точність трекерів (тривалість надійна, стадії — ні). Сон як діагностичний інструмент при перевтомі і психологічному стресі. Помилки: борг сну, алкоголь, orthosomnia. clinical-safety: NOT_REQUIRED.
- `content/post-1795-morning-readiness-checklist.md` — Cluster 2 · Post 5/10 · Block 1781–1830. Ранковий чекліст 90 сек: рішення A/B/C. Три блоки — об'єктивні маркери (HRV, ЧСС у спокої), суб'єктивна оцінка (3 числа), контекст (фаза мезоциклу, зовнішній стресор). Матриця рішень. Правило розминки для жовтих ситуацій. Варіанти B1/B2/B3. clinical-safety: NOT_REQUIRED.
- `content/post-1796-rpe-trend-accumulated-fatigue.md` — Cluster 2 · Post 6/10 · Block 1781–1830. RPE-тренд: центральна і периферична компонента стомленості. Контрольні підйоми — вибір і протокол. Реакція на паттерн: +0.5–1 / 2 тижні (норма) vs +1.5–2 / 2–3 тижні (сигнал). Калібровка шкали. Відмінності між гіпертрофійним і силовим блоком. clinical-safety: NOT_REQUIRED.
- `content/post-1797-e1rm-strength-tracking-recovery.md` — Cluster 2 · Post 7/10 · Block 1781–1830. e1RM: формули Brzycki і Epley, оптимальне вікно RIR 0–2. Чому відображає відновлення, а не тільки адаптацію. Протокол контрольного підйому. Інтерпретація тренду: −3–5%+ за 2–3 тижні. Коли хибний сигнал (техніка, кількість повторень поза діапазоном). clinical-safety: NOT_REQUIRED.
- `content/post-1798-conflicting-recovery-signals.md` — Cluster 2 · Post 8/10 · Block 1781–1830. Чому маркери суперечать (кожен вимірює різне). П'ять типових конфліктів і інтерпретація. Ієрархія: рівні 1–4 (хвороба → правило розминки → RPE-тренд → HRV-тренд → суб'єктивне). Правило трьох маркерів: 2+ в один бік = реагуй. clinical-safety: NOT_REQUIRED.
- `content/post-1799-monitoring-without-tracker.md` — Cluster 2 · Post 9/10 · Block 1781–1830. Мінімальна система без трекера: 3 числа вранці (90 сек) + RPE/e1RM при підходах + тижневий огляд (5 хв). Мінімальний тижневий шаблон. Три практичні приклади. Типові помилки. Перехід до трекера: паралельне ведення 4–6 тижнів. clinical-safety: NOT_REQUIRED.
- `content/post-1800-cluster2-synthesis-monitoring-system.md` — Cluster 2 · Post 10/10 · Block 1781–1830 · **CLUSTER 2 COMPLETE**. Карта шести маркерів (що/коли/навіщо/горизонт). Два рівні системи (мінімум без трекера / розширений). Ієрархія сигналів при конфлікті. Тижневий аудит відновлення (6 запитань). Межі системи. Огляд Cluster 3–5 Block 1781–1830. clinical-safety: NOT_REQUIRED.

### Changed
- `blog.html` — оновлено: posts 1792–1800 додано в каталог.
- `VERSION` — 7.6.2 → 7.7.0.

---

## [7.6.2] — 2026-06-21

### Changed
- `docs/MSA_TEMPLATE.md` — v0.5 → v0.6. New §4.6 Multi-Year Price Escalation clause (seven sub-clauses) added to Article 4 (Order Forms, Fees, and Payment). Closes `docs/COST_MODEL.md §42.10` checklist item 8 (P1/M5). §4.6.1 Applicability: Year 2 and Year 3 of 2- or 3-year Multi-Year Commitments only. §4.6.2 Escalation Formula: `min(Prior Rate × (1 + CPI + 0.01), Prior Rate × 1.05)` — CPI-U (All Urban Consumers), fixed +1% service-improvement component, 5% annual cap non-waivable by agreement. §4.6.3 Price Floor: COGS-anchored minimum per `docs/COST_MODEL.md §31.5`; floor applies if escalated rate would fall below it. §4.6.4 Notice Requirement: 90-day written notice to billing contact; content includes Escalated Rate, BLS CPI-U reference month (date only, no URL), escalation percentage, cap-confirmation, and right to request full methodology within 15 business days. §4.6.5 Escalation Transparency: FORM provides complete calculation on 15-day written request; no BLS URL in any notice or audit payload. §4.6.6 Audit Record: each Annual Escalation recorded as `enterprise.renewal_escalation_calculated` (HIGH, 7-year) per DEC-030 HMAC chain; payload: BLS reference month (date-only), CPI percentage, cap trigger flag, Prior Rate, Escalated Rate. §4.6.7 Outside Counsel Checkpoint: required before first §4.6 execution (OQ-REN-02) — three confirmation points (CPI-U jurisdiction appropriateness, 5% cap enforceability, 90-day notice statutory compliance); EU customers may require HICP index substitution; filing path `compliance/contracts/{CUSTOMER_SLUG}/msa-s46-counsel-signoff-v{N}.pdf`. §4.5 updated to defer multi-year escalation governance to §4.6. Internal Notes: §4.6 row added. Privacy floor: BLS reference month date-only in all notices and DEC-030 payloads; no individual employee user_id, health data, or Art. 9 category in any §4.6 artefact. Cross-references: `docs/COST_MODEL.md §42.5` (formula source, `bls_report_date` date-only constraint), `docs/COST_MODEL.md §42.10 item 8` (closed 🟢), `docs/COST_MODEL.md §42.11 OQ-REN-02` (counsel path), `docs/COST_MODEL.md §31.5` (price floor), `docs/AUDIT_LOG_SCHEMA.md §Enterprise` (`enterprise.renewal_escalation_calculated` DEC-030 event). Owner: compliance-officer + enterprise-architect + founder.
- `docs/COST_MODEL.md` — §42.10 item 8 marked `[x] Done (2026-06-21, MSA_TEMPLATE.md v0.6 §4.6)`.
- `VERSION` — 7.6.1 → 7.6.2.

---

## [7.6.1] — 2026-06-21

### Added
- `content/post-1791-hrv-recovery-marker-practical.md` — Cluster 2 · Post 1/10 · Block 1781–1830. HRV як маркер відновлення — практичне застосування: чому абсолютний RMSSD не порівнюється між атлетами, особиста базова лінія і ковзне середнє. Ранковий протокол: лежачи, до кофеїну, 5 хв, один пристрій. Зони інтерпретації (±10% норма, −10–20% помірне відхилення, −20% значне). Що HRV не каже: тип стресу, м'язова специфічність, діагноз перетренованості. Топ-5 помилок (реакція на одну точку, порівняння з іншими, непослідовне вимірювання, ігнорування контексту, поганий пристрій). Ієрархія при конфлікті маркерів. Victor-інтеграція: тижневий тренд + контекст тренувальних даних. clinical-safety: NOT_REQUIRED.

### Changed
- `blog.html` — картка post-1791 додана; total entries updated to post-1791.
- `STATUS.md` — Block 1781–1830 IN PROGRESS 11/50; Cluster 2 IN PROGRESS 1/10 · v7.6.1.
- `VERSION` — 7.6.0 → 7.6.1.

---

## [7.6.0] — 2026-06-21

### Added
- `content/post-1782-sleep-primary-recovery-vector.md` — Cluster 1 · Post 2/10 · Block 1781–1830. Сон як первинний вектор відновлення: SWS і секреція GH (70–80% в перших 2 циклах), REM і консолідація рухових навичок, синтез білка під час сну. Наслідки депривації: RPE зростає, силовий вихід падає 2–8%. Тривалість vs. якість — sleep efficiency як операційний маркер. Victor-інтеграція з трекерами. clinical-safety: NOT_REQUIRED.
- `content/post-1783-ans-hrv-what-is-measured.md` — Cluster 1 · Post 3/10 · Block 1781–1830. АНС і HRV: RMSSD і LF/HF, симпатично-парасимпатичний баланс, ранковий протокол (лежачи, до кофеїну, 5 хв). Міжіндивідуальна варіація: числа непорівнянні між атлетами, тільки власний тренд. Гострий спад після тренування vs. хронічна супресія. Коли HRV вводить в оману: хвороба, алкоголь, дорога. Порівняння пристроїв: Polar H10 vs. оптичні. clinical-safety: NOT_REQUIRED.
- `content/post-1784-central-peripheral-neuromuscular-fatigue.md` — Cluster 1 · Post 4/10 · Block 1781–1830. Центральна і периферійна нейром'язова стомленість: метаболіти і crossbridge cycling (периферійна), суправертебральний drive і рекрутування рухових одиниць (центральна). Чому «ЦНС тижнями відновлюється» — перебільшення (24–72год). Практичне розрізнення: локально vs. глобально. Victor: паттерни через RPE кількох вправ. clinical-safety: NOT_REQUIRED.
- `content/post-1785-inflammation-repair-after-training.md` — Cluster 1 · Post 5/10 · Block 1781–1830. Запалення і репарація: EIMD механізм (Z-disk, sarcomere strain), IL-6/TNF-α у репарації. DOMS — не надійний маркер гіпертрофії (дані після 2018). Repeated bout effect. НПЗП і лід: дані про притуплення адаптації. Хронічне запалення при систематичному недовідновленні. clinical-safety: NOT_REQUIRED.
- `content/post-1786-protein-carbs-recovery-physiology.md` — Cluster 1 · Post 6/10 · Block 1781–1830. Білок і вуглеводи у відновленні — фізіологія, не дієтологія. mTORC1-шлях, leucine-поріг (~2–3г), тимчасове вікно MPS (Aragon & Schoenfeld 2013). Ресинтез глікогену: 5–7%/год, GLUT4-транслокація. Коли timing критичний (< 8год між сесіями) vs. коли ні. Явний дисклеймер: не дієтологічна консультація. clinical-safety: CONDITIONAL_PASS (guard rails вбудовані).
- `content/post-1787-stress-allostatic-load-recovery.md` — Cluster 1 · Post 7/10 · Block 1781–1830. Стрес і алостатичне навантаження: модель McEwen, HPA вісь і кортизол при тренувальному і психологічному стресі. Тіло не розрізняє стрес-джерело. Life load audit — 3 запитання перед мезоциклом. Victor: HRV-тренд як інтегратор всіх стресорів. clinical-safety: NOT_REQUIRED.
- `content/post-1788-recovery-35-plus-age-factors.md` — Cluster 1 · Post 8/10 · Block 1781–1830. Відновлення після 35: GH/IGF-1/T зниження, сповільнений ресинтез колагену сухожиль/зв'язок, зменшення SWS. Що НЕ змінюється: максимальний силовий потенціал, гіпертрофічна відповідь. Masters athletes meta-analyses. Практичні корекції програмування. clinical-safety: NOT_REQUIRED.
- `content/post-1789-active-recovery-when-helps-when-hurts.md` — Cluster 1 · Post 9/10 · Block 1781–1830. Активне відновлення: визначення (30–50% VO2max), механізми кровообігу і лімфатики. Дані: DOMS знижується на 10–20%, але структурна репарація не прискорюється. Коли допомагає (прогулянка 20–30 хв після максимального зусилля) vs. заважає (прихований середній інтенсив). Практичний протокол: ЧСС < 130. clinical-safety: NOT_REQUIRED.

- `content/post-1790-cluster1-synthesis-recovery-architecture.md` — Cluster 1 · Post 10/10 · Block 1781–1830 · **CLUSTER 1 COMPLETE**. Операційна карта відновлення: п'ять векторів, часова матриця відновлення (вектор × вікно × маркери × деградація), ієрархія пріоритетів (сон → алостатичне навантаження → нейром'язовий стан → справжній відпочинок → нутрієнтний тайминг), протокол моніторингу (Victor-сторона + атлет-сторона), три режими помилок, огляд Clusters 2–5. clinical-safety: NOT_REQUIRED.

### Changed
- `blog.html` — картки post-1782 через post-1790 додані; total entries updated to post-1790.
- `STATUS.md` — Block 1781–1830 Cluster 1 COMPLETE 10/10; IN PROGRESS 10/50.
- `VERSION` — 7.5.2 → 7.6.0.

---

## [7.5.2] — 2026-06-20

### Changed
- `docs/AUDIT_LOG_SCHEMA.md` — v2.21 → v2.23. Новий розділ `### Enterprise Contract Renewal events (DEC-030 HMAC-chained · COST_MODEL §42.7 · CC5.2/CC1.4/CC4.1/CC2.2/A1.1)` вставлено перед `## Export & delivery`. Зареєстровано три DEC-030 HMAC-chained події з `docs/COST_MODEL.md §42.7`: `enterprise.renewal_notice_sent` (STANDARD, 7yr — RENEW-CHAIN-01 anchor, `days_until_renewal` 85–100, `notice_type` enum, `sent_by` FORM UUID), `enterprise.renewal_escalation_calculated` (HIGH, 7yr — ESCALATION-CHAIN-01 prerequisite, `bls_report_date` date-only, `floor_respected: z.literal(true)` HTTP 422), `enterprise.contract_renewed` (STANDARD, 7yr — RENEW-CHAIN-01 + ESCALATION-CHAIN-01 terminal event, `notice_event_id` FK, `pricing_exception_event_id` для retention_discount path, `floor_respected: z.literal(true)` HTTP 422). Три Zod v2 схеми. RENEW-NOTICE-01 моніторинг invariant. Три SOC 2 evidence artefacts (REN-E-001 CC5.2/CC1.4, REN-E-002 CC4.1/CC2.2, REN-E-003 CC4.1/A1.1). Retention table +3 rows. Closes `docs/COST_MODEL.md §42.10` checklist item 1 (P0/M12 — 🟢).
- `docs/COST_MODEL.md` — §42.10 checklist item 1 позначено `[x] Done (2026-06-20, AUDIT_LOG_SCHEMA.md v2.23)`.
- `docs/SOC2_READINESS.md` — §100.1 three-document chain table extended: додано 4-й рядок для `docs/AUDIT_LOG_SCHEMA.md §Enterprise Contract Renewal` v2.23 (DEC-030 event registration); sequencing note оновлено.
- `VERSION` — 7.5.1 → 7.5.2.

## [7.5.1] — 2026-06-20

### Added
- `content/post-1781-recovery-mechanisms-vs-subjective-feel.md` — Cluster 1 · Post 1/10 · Block 1781–1830. Відновлення: чотири паралельних процеси (ресинтез глікогену 24–48 год, м'язова репарація 48–96 год, нейром'язове відновлення 72–120+ год, гормональний баланс). Чому суб'єктивне «відчуваю добре» — запізнілий сигнал із низькою специфічністю. Що корелює краще: HRV-тренд, RPE при контрольному навантаженні, технічний дрейф, e1RM динаміка. Часова архітектура відновлення — таблиця. clinical-safety: NOT_REQUIRED.
- `README.md` — Block 1881–1930 «Прийняття рішень у тренуванні: сесійний і тижневий рівень» додано в roadmap (50 тем, 5 кластерів). Теми: сесійний протокол готовності, авторегуляція, рішення в нестандартних ситуаціях, читання сигналів тіла без self-help риторики, тижневий і мезоцикловий ретро.

### Changed
- `blog.html` — картка post-1781 додана; total entries updated to post-1781.
- `STATUS.md` — Block 1781–1830 IN PROGRESS 1/50; content engine row updated v7.5.1.
- `VERSION` — 7.5.0 → 7.5.1.

## [7.5.0] — 2026-06-20

### Added
- `content/post-1773-volume-progression-mesocycles.md` — Cluster 5 · Post 3/10 · Block 1731–1780. Прогресія об'єму між мезоциклами: loading block → deload → новий блок (не з нуля). Israetel et al. (2019): правило +2–4 ефективних підходи за мезоцикл. Таблиця чотирьох послідовних блоків. Коли об'єм плато і прогресують інтенсивністю. Міжмезоциклова карта Victor. clinical-safety: NOT_REQUIRED.
- `content/post-1774-deload-when-how-much-protocol.md` — Cluster 5 · Post 4/10 · Block 1731–1780. Деловантаж: плановий vs реактивний, протокол -40–60%/RPE ≤ 6/1 тиждень, суперкомпенсація. Bosquet et al. (2007): 2–3% приріст після планового зниження. Bickel et al. (2011): інтенсивність як збережувальний сигнал. Дві типові помилки (не робити / занадто глибоко). clinical-safety: NOT_REQUIRED.
- `content/post-1775-weekly-volume-audit-protocol.md` — Cluster 5 · Post 5/10 · Block 1731–1780. 15-хвилинний протокол тижневого аудиту: ефективні підходи vs MEV/MAV, RPE-тренд, суб'єктивна свіжість, технічні нотатки. Чотири запитання → три виходи (продовжити/скоригувати/деловантаж). Автоматична агрегація в Victor. clinical-safety: NOT_REQUIRED.
- `content/post-1777-volume-needs-experience-progression.md` — Cluster 5 · Post 7/10 · Block 1731–1780. Потреби в об'ємі по стадіях: початківець 6–10, інтермедіат 10–16, досвідчений 16–22+. Ralston et al. (2017), Colquhoun et al. (2018). Чому менший стартовий об'єм — ефективніший підхід. Як Victor адаптує рекомендації до тренувального віку. clinical-safety: NOT_REQUIRED.
- `content/post-1778-victor-volume-management-partnership.md` — Cluster 5 · Post 8/10 · Block 1731–1780. Межі автоматизації Victor: що система бачить (ефективні підходи, RPE-тренди, MEV/MAV/MRV, флаги) і що не бачить (суб'єктивний стан, сон без трекера, контекст). Модель партнерства: Victor = пам'ять + аналіз + флаги; Атлет = контекст + рішення + логування. clinical-safety: NOT_REQUIRED.
- `content/post-1779-pre-mesocycle-checklist.md` — Cluster 5 · Post 9/10 · Block 1731–1780. Чекліст перед новим мезоциклом: 4 блоки (оцінка попереднього, стан зараз, цілі нового, фіксація в Victor). 8 запитань + 3 кроки Victor-онбординг. clinical-safety: NOT_REQUIRED.
- `content/post-1780-block-synthesis-volume-management-system.md` — Cluster 5 · Post 10/10 · Block 1731–1780. BLOCK SYNTHESIS: синтез 50 постів кластерів 1–5, три поведінкові зрушення, архітектура операційної системи управління об'ємом. Анонс Block 1781–1830 (інтенсивність і управління навантаженням). **BLOCK 1731–1780 COMPLETE 50/50.** clinical-safety: NOT_REQUIRED.

### Changed
- `blog.html` — картки posts 1772–1780 додані; total entries updated to post-1780. Block 1731–1780 Cluster 5 COMPLETE.
- `STATUS.md` — Block 1731–1780 позначено COMPLETE 50/50; Cluster 5 COMPLETE v7.5.0; наступні пріоритети: Block 1781–1830 (тема TBD — product-strategist + sports-scientist).

## [7.4.2] — 2026-06-20

### Added
- `content/post-1772-mrv-breach-signals-accumulated-fatigue.md` — Cluster 5 · Post 2/10 · Block 1731–1780. Сигнали перевищення MRV: маркери накопиченої втоми (performance decline, RPE drift, мотиваційні флаги). Meeusen et al. (2013) ECSS/ACSM overtraining consensus, Halson & Jeukendrup (2004) overreaching vs overtraining. Різниця між важким тижнем і системним перевтомленням. Три кроки відповіді. Як Victor фіксує і флажить MRV-порушення. clinical-safety: NOT_REQUIRED.
- `content/post-1776-five-volume-management-mistakes.md` — Cluster 5 · Post 6/10 · Block 1731–1780. П'ять помилок управління об'ємом: volume inflation без відновлення, один об'єм для всіх м'язів, ігнорування сигналів втоми, рахунок всіх підходів як рівних, зміна занадто багатьох змінних одночасно. Для кожної: причина, вигляд у даних Victor, корекція. clinical-safety: NOT_REQUIRED.

## [7.4.0] — 2026-06-20

### Added
- `content/post-1771-victor-effective-sets-tracking.md` — Cluster 5 · Post 1/10 · Block 1731–1780 · Синтез і операційна система управління об'ємом. Ефективний підхід vs. розминочний vs. сміттєвий об'єм. Schoenfeld 2010/2017: гіпертрофічний сигнал і наближення до відмови. Steele et al. 2017: підходи в межах 0–4 RIR. Helms et al. 2016: RIR-калібровка і систематична недооцінка резерву. Класифікація Victor: RPE ≥ 7 + технічна цілісність = ефективний підхід. Дроп-сети (0,5–1 підхід), суперсети антагоністів (обидва рахуються), суперсети агоністів (знижка 0,7), передвтомлення (знижка 0,1–0,2). Junk volume: низький RPE або понад MAV. Тижневий підрахунок MEV/MAV/MRV тільки по ефективних підходах. clinical-safety: NOT_REQUIRED.

### Changed
- `blog.html` — post-1771 card added; total entries updated to post-1771.
- `README.md` — Block 1831–1880 roadmap added (50 posts · Тренувальна поведінка та атлетична ідентичність): 5 clusters (1831–1840 атлетична ідентичність і звичка; 1841–1850 самооцінка без тренера; 1851–1860 соціальний контекст; 1861–1870 управління очікуваннями; 1871–1880 синтез операційної системи атлетичної практики).
- `STATUS.md` — Block 1731–1780 Cluster 5 IN PROGRESS · v7.4.0; post-1771 added.
- `VERSION` — 7.3.0 → 7.4.0.

---

## [7.3.0] — 2026-06-20

### Added
- `content/post-1762-illness-recovery-volume-protocol.md` — Cluster 4 · Post 2/10. Повернення після хвороби. Хвороба ≠ деловантаж: запальні цитокіни, post-illness recovery mode (Nieman et al., 2011). Правило «вище/нижче шиї». П'ятиденний протокол: дні 1–2 рух, день 3 — 50% RPE 5–6, дні 4–5 — 70% RPE 6–7, дні 6–7 — норма. Colquhoun et al. (2018) по детренінгу тренованих атлетів. Спеціальний випадок COVID-19 (Elliott et al., 2020; мінімум 7 днів до старту). clinical-safety: NOT_REQUIRED.
- `content/post-1763-travel-training-volume-management.md` — Cluster 4 · Post 3/10. Тренування в подорожі. Три сценарії: готельний зал / без залу / повноцінний зал. Ієрархія рухових патернів замість відтворення вправ (нижня частина тіла / тяга / жим / хіп-хінж). Прогресія складності вправи замість ваги при тренуванні з власною вагою (pistol squat, archer push-up, nordic curl). Об'єм 50–70%, RPE без змін, -1 RPE пункт перші дні після перельоту. Логістичний мінімум (resistance band, TRX). clinical-safety: NOT_REQUIRED.
- `content/post-1764-stress-volume-hpa-axis.md` — Cluster 4 · Post 4/10. Об'єм під час психологічного стресу. ГПА-вісь не розрізняє тип стресу: кортизол, ГКС, алостатичне навантаження (McEwen & Stellar, 1993). Marcora et al. (2009): когнітивна втома підвищує RPE без зміни фізіологічної вартості. Dhabhar (2014): гострий стрес — адаптивний, хронічний — руйнівний. Три рівні: помірний (-20–30%), значний (-40–50%, RPE 7, -1 сесія), гострий (мінімальний рух). Наратив «тренуватись щоб скинути стрес» — правдивий лише при рівні 1. clinical-safety: NOT_REQUIRED.
- `content/post-1765-post-deload-volume-return.md` — Cluster 4 · Post 5/10. Перший тиждень після деловантажу. Деловантаж ≠ деловантаж після хвороби. Fitness-fatigue модель (Banister, 1975; Calvert et al., 1976) замість спрощеної суперкомпенсації. Три помилки: відразу до MRV, тест 1RM в перший день, старт з MEV нового блоку. Правильна структура: сесія 1 — 80–90% MAV, RPE 7–7.5; сесія 2 — 90–100%, RPE 8. «Свіжіший, ніж зазвичай» — пік стабільніше розподілити по блоку. clinical-safety: NOT_REQUIRED.
- `content/post-1766-post-competition-volume-management.md` — Cluster 4 · Post 6/10. Після змагань або максимальних зусиль. Три специфіки змагального дня: максимальні зусилля кілька разів, симпатична активація, логістичний стрес. Дуга: дні 1–3 рух; дні 4–5 — 40–50% RPE 5–6; дні 6–10 — 60–70% RPE 6–7; дні 11–14 — 80–90%; тиждень 2+ — 100% MAV. Chycki et al. (2018): water cut — регідратація за 24 год не відновлює нейром'язову функцію повністю. Таблиця об'єм/RPE/фокус. Коли починати новий мезоцикл. clinical-safety: NOT_REQUIRED.
- `content/post-1767-volume-perception-trap-feeling-undertrained.md` — Cluster 4 · Post 7/10. Перцептивна пастка «занадто мало». Чотири джерела: відсутність DOMS (McHugh et al., 1999 — DOMS ≠ стимул); переоцінка RIR (Helms et al., 2016 — середня похибка 1–2 повторення в бік «більше в запасі»); порівняння без контексту; незадоволення темпом. Практична чотирьохкрокова перевірка через журнал Victor. Зворотній ефект: накопичена втома ховає потребу в зниженні. clinical-safety: NOT_REQUIRED.
- `content/post-1768-sleep-deprivation-volume-compromise.md` — Cluster 4 · Post 8/10. Хронічний дефіцит сну і об'єм. GH-секреція і SWS (Ukichterle et al., 2018); mTOR і протеїновий синтез (Dattilo et al., 2011); нейром'язове відновлення (Skein et al., 2011). RPE-зсув без суб'єктивної сонливості (Van Dongen et al., 2003). Три рівні: ситуативний (1–2 ночі) -10–15%; накопичений (10+ ночей) -25–30%, RPE 7.5; хронічний (2+ тижні < 5.5 год) — мінімум, вирішуємо причину. Ієрархія: лишаємо основні рухи з інтенсивністю, прибираємо ізоляцію і накачувальні блоки. Фіксований час пробудження як простий буфер. clinical-safety: NOT_REQUIRED.
- `content/post-1769-extended-absence-return-protocol.md` — Cluster 4 · Post 9/10. Повернення після 2–6 тижнів. М'язова пам'ять: міоядра зберігаються місяцями після детренінгу (Bruusgaard et al., 2010) — повернення за 30–50% початкового часу. Протокол: 2–3 тижні — легкий старт 60–70% MAV тиждень 1; 4–6 тижнів — поетапно 50 → 75 → 90 → 100% MAV по тижнях. Стартові ваги: 2 тижні — 85–90%; 3–4 тижні — 75–80%; 5–6 тижнів — 65–70%. Ризик надмірної DOMS і втрата регулярності. clinical-safety: NOT_REQUIRED.
- `content/post-1770-cluster-synthesis-volume-nonstandard-situations.md` — Cluster 4 · Post 10/10 · CLUSTER 4 COMPLETE. Синтез: зведена таблиця 14 нестандартних ситуацій (ситуація → об'єм % / RPE-ліміт / частота / джерело). Три принципи: інтосивність захищається першою (Bickel et al., 2011); одна точка — не тренд; усвідомлена адаптація > реактивне рішення. Мітки тижня у Victor (busy_week / post_illness / travel / high_stress / post_deload / post_competition / return). Що кластер не вирішує (overtraining syndrome, травми, хронічні хвороби). Перехід до Cluster 5. clinical-safety: NOT_REQUIRED.

### Changed
- `blog.html` — posts 1762–1770 cards added (9 нових карток, total entries updated to post-1770). Cluster 4 complete in blog.
- `STATUS.md` — Cluster 4 marked COMPLETE 10/10 · v7.3.0; next: Cluster 5 (posts 1771–1780 · Синтез блоку та операційна система управління об'ємом).

---

## [7.2.0] — 2026-06-20

### Added
- `content/post-1761-busy-week-minimal-protocol.md` — Block 1731–1780 · Cluster 4 · Post 1/10 (Об'єм у нестандартних ситуаціях). Зайнятий тиждень: мінімальний протокол без втрати адаптаційного сигналу. Bickel et al. (2011): 1/3 об'єму при збереженій інтенсивності зберігає силу 16 тижнів. Fisher et al. (2013): інтенсивність — первинний збережувальний сигнал, об'єм різати можна агресивно. Два протоколи: 60–75 хвилин один раз (full body 8–12 підходів RPE 7–8) і 40 хвилин двічі (верх/низ або push/pull/legs). Чотири помилки: надолуження в кінці тижня, зниження лише інтенсивності, збереження повного об'єму у стислий час, ігнорування нестандартного тижня. Ogasawara et al. (2013) про м'язову пам'ять. clinical-safety: NOT_REQUIRED.

### Changed
- `blog.html` — post-1761 card added; total entries updated to 1761.
- `README.md` — Block 1781–1830 roadmap added (50 posts · Система відновлення self-coached атлета): 5 clusters (1781–1790 принципи відновлення; 1791–1800 моніторинг; 1801–1810 активне відновлення і деловантаж; 1811–1820 Victor-інтеграція; 1821–1830 практична система і нестандартні ситуації).
- `VERSION` — 7.1.0 → 7.2.0.
- `STATUS.md` — Block 1731–1780 Cluster 4 IN PROGRESS · post-1761 logged; next: post-1762.

---

## [7.1.0] — 2026-06-20

### Added
- `content/post-1752-allostatic-load-and-mrv.md` — Block 1731–1780 · Cluster 3 · Post 2/10 (Індивідуалізація об'єму). Алостатичне навантаження і MRV: McEwen (1993, 1998), стрес-фізіологія, кортизол як «загальна валюта стресу». Компоненти: когнітивний стрес, сон, соціальні стресори, харчовий статус, хвороба, джетлаг. Griffith et al. (2015): відновлення на 40% довше при підвищеному психосоціальному стресі. Правило 70%, деловантаж за потребою. clinical-safety: NOT_REQUIRED.
- `content/post-1753-sex-differences-volume-tolerance.md` — Block 1731–1780 · Cluster 3 · Post 3/10. Гендерні відмінності і переносимість об'єму: склад волокон (Glenmark 1994), захисний ефект естрогену (Tiidus 2011), Hunter (2009) — вища стійкість до втоми. Потенційно вищий відносний MRV у жінок. Менструальний цикл як тренувальна змінна: фолікулярна vs. лютеїнова фаза (Bambaeichi 2004, Sung 2014). Практичні адаптації. Застереження: варіабельність усередині статей &gt; між ними. clinical-safety: NOT_REQUIRED.
- `content/post-1754-sleep-and-recovery-volume.md` — Block 1731–1780 · Cluster 3 · Post 4/10. Сон як тренувальна змінна: GH секреція і SWS, IGF-1 дефіцит −15–25% при скороченні сну (Mullington 2009), REM і рухова консолідація (Stickgold 2005). Blumert et al. (2007): −2.5 кг жиму після 24 год без сну. Mah et al. (2011): 10 год сну +5% швидкості. Механізм MRV-стискання через дефіцит сну ~20–30%. Оптимум 8–10 год для атлета. Практичне управління. clinical-safety: NOT_REQUIRED.
- `content/post-1755-caloric-status-and-mrv.md` — Block 1731–1780 · Cluster 3 · Post 5/10. Харчовий статус і MRV: Murphy et al. (2015) — MPS −19% при помірному дефіциті. AMPK vs. mTOR конкуренція (Mounier 2015). Знижений глікогенний ресинтез (Hargreaves 2004). Підвищений кортизол при дефіциті. Кількісні орієнтири: помірний дефіцит MRV −10–20%, агресивний −25–40%. Правило: зберегти інтенсивність, знизити об'єм до MEV. Helms et al. (2014). clinical-safety: NOT_REQUIRED.
- `content/post-1756-chronological-age-and-volume.md` — Block 1731–1780 · Cluster 3 · Post 6/10. Хронологічний вік і об'єм після 35: саркопенія, анаболічна резистентність (Moore 2015), зниження GH/IGF-1/тестостерону. Повільніше відновлення, зміни сполучної тканини. Що зберігається: гіпертрофічний потенціал (Peterson 2011), нейронна адаптація, перевага досвіду. Адаптації: більше відновлення між сесіями, деловантаж кожні 3 тижні, уникнення стрибків об'єму. Сильні сторони атлета 35+. clinical-safety: NOT_REQUIRED.
- `content/post-1757-genetic-variability-and-volume.md` — Block 1731–1780 · Cluster 3 · Post 7/10. Генетична варіабельність відповіді: Hubal et al. (2005) — від −2% до +59% гіпертрофії при однаковій програмі. Спадковість типу волокон 40–50% (Bouchard 1986). IGF-1, міостатин (MSTN), ACTN3 як генетичні модулятори. Спадковість відповіді 40–60%. Чому комерційні ДНК-тести неефективні (Webborn 2015). High/low responder і наслідки для об'єму. n-of-1 як відповідь. clinical-safety: NOT_REQUIRED.
- `content/post-1758-nof1-volume-calibration.md` — Block 1731–1780 · Cluster 3 · Post 8/10. N-of-1 калібровка MEV/MRV: три показники (e1RM, SRQ, тривалість DOMS). Протокол MEV-пошуку знизу вгору. Протокол MRV-пошуку зверху вниз. Правило однієї змінної. Мінімальний горизонт 4 тижні. Сезонний перегляд кожні 12–16 тижнів. Мінімальний журнал. Victor як аналітичний шар. clinical-safety: NOT_REQUIRED.
- `content/post-1759-five-mistakes-volume-individualization.md` — Block 1731–1780 · Cluster 3 · Post 9/10. П'ять помилок: таблиця як відповідь замість гіпотези; зміна кількох змінних; оцінка до деловантажу; ігнорування контекстуальних змінних; порівняння з іншими. Спільне коріння: нетерпіння і дискомфорт невизначеності. Виправлення для кожної помилки. clinical-safety: NOT_REQUIRED.
- `content/post-1760-cluster-synthesis-volume-individualization.md` — Block 1731–1780 · Cluster 3 · Post 10/10 · Cluster Synthesis. Карта 9 змінних з кластеру. Шестикрокова операційна система: стартова гіпотеза → контекстний чекліст → 4-тижній блок + 3 показники → деловантаж і оцінка → одна зміна → сезонне оновлення. Ієрархія рішень: сон → стрес → харчовий статус → об'єм. Практичне порівняння двох атлетів. 7 ключових принципів. Перехід до Cluster 4. clinical-safety: NOT_REQUIRED.

### Changed
- `blog.html` — posts 1752–1760 added; total posts displayed updated to 1760.
- `VERSION` — 7.0.1 → 7.1.0.
- `STATUS.md` — Block 1731–1780 Cluster 3 COMPLETE 10/10; next priorities updated.

---

## [7.0.1] — 2026-06-20

### Added
- `docs/DATA_MODEL.md §43` — `enterprise_renewals` Schema: Canonical DDL Registration (Migration 0084). Closes cross-reference obligation from `docs/COST_MODEL.md §42.6` (v2.8, 2026-06-20). Two ENUMs (`renewal_type_enum` 5-value, `rate_basis_enum` 4-value); CREATE TABLE (21 columns — UUID PK, two FK RESTRICT with ON DELETE RESTRICT boundary to §67 tenant deletion pipeline, `renewal_type_enum`, DATE, two INTEGER CHECK > 0, two NUMERIC(10,4) CHECK > 0, `rate_basis_enum`, BOOLEAN, two NULLable NUMERIC for escalation, NULLable NUMERIC for multi-year discount, `floor_respected` BOOLEAN DEFAULT true, two GENERATED ALWAYS AS STORED NUMERIC for `new_acv_usd`/`new_tcv_usd`, SMALLINT CHECK IN (1,2,3), three soft-ref UUID NULLable, TIMESTAMPTZ); four CHECKs (`chk_floor_always_respected CHECK (floor_respected = true)` DDL pricing floor backstop, `chk_escalation_fields` all-or-nothing escalation, `chk_multi_year_discount` iff multi_year type, `chk_non_renewal_seats`); three indexes (UNIQUE `(tenant_id, renewal_date)`, `renewal_date DESC`, `renewal_type`); four COMMENT ON annotations. `enterprise_contracts` renewal UPDATE in single SERIALIZABLE transaction with `enterprise_renewals` INSERT and `enterprise.contract_renewed` event. Five-item staging validation checklist. Full column summary table (21 rows) + CHECK constraints table + GENERATED columns semantics note. RLS: `form_api` REVOKED; `compliance_reviewer` SELECT all; `form_admin` SELECT all; `form_system` SELECT + INSERT only (no UPDATE — append-only); zero tenant-role grants; DDL auditor proof queries. RENEW-CHAIN-01/ESCALATION-CHAIN-01 DDL relationship (`notice_event_id` soft-ref as per-row chain anchor; `chk_escalation_fields` as ESCALATION-CHAIN-01 DDL backstop); two SOC 2 evidence queries. SOC 2 evidence cross-references: REN-E-001 (CC5.2/CC1.4), REN-E-002 (CC4.1/CC2.2), REN-E-003 (CC4.1/A1.1) with DDL-layer invariant column and auditor narratives. Five-item implementation checklist (3× P0/M12, 2× P1/M13). DATA_MODEL header v1.21 → v1.22. Owner: enterprise-architect + compliance-officer + customer-success.
- `docs/SOC2_READINESS.md §100` — Cross-Reference Patch: DATA_MODEL §43 (`enterprise_renewals` · Migration 0084 · CC5.2/CC1.4/CC4.1). Three-document chain table (COST_MODEL §42 economic spec → DATA_MODEL §43 canonical DDL → SOC2_READINESS §100 cross-reference closure); first §9X-§100 patch added simultaneously with DATA_MODEL section (not retroactively). Nine-row canonical DDL source registration table. Three privacy floor supplements: no tenant role accesses `enterprise_renewals` (financial data — zero exceptions); `tenant_manager` exclusion confirmed; `cpi_reference_month` date-only never URL. Four-row SOC 2 criteria mapping supplement (DDL layer only, §79.4 unchanged): CC5.2 (`chk_floor_always_respected` visible HTTP 500 on bypass — not silent failure), CC1.4 (UNIQUE INDEX prevents duplicate renewals; `notice_event_id` enables one-query RENEW-CHAIN-01 check), CC4.1 (two indexes guarantee REN-E-002/003 evidence cadence at fleet scale), CC2.2 (`chk_escalation_fields` ensures escalation notice internally consistent at INSERT). Four obligations closed (pattern ✓, COST_MODEL §42.9 one-way reference ✓, §42.10 item 5 §79.4 registration 🟡 P1/M13, header discrepancy v3.22.1 → v3.24.1 ✓). Three-item implementation checklist (1× P0/M12, 2× P1/M13). SOC2_READINESS header v3.22.1 → v3.24.1.

### Changed
- `VERSION` — 7.0.0 → 7.0.1.

---

## [7.0.0] — 2026-06-20

### Added
- `content/post-1751-training-age-and-volume.md` — Тренувальний вік і об'єм: чому новачки не потребують більше. Cluster 3 · Блок 1751–1760 · Індивідуалізація об'єму. Тренувальний вік vs. хронологічний стаж. Новачковий ефект і нейронні адаптації (Moritani & deVries, 1979). MEV новачка: 4–6 підходів на групу. Три сценарії, де надлишковий об'єм шкодить початківцю. Практичний самодіагноз рівня атлета. Carpinelli & Otto (1998), Krieger (2010). Прогресія об'єму з тренувальним віком. Victor як трекер функціонального рівня. clinical-safety: NOT_REQUIRED.

### Changed
- `blog.html` — post-1751 картка додана (blog updated to post-1751).
- `VERSION` — 6.99.0 → 7.0.0.

---

## [6.99.0] — 2026-06-20

### Added
- `content/post-1744-deload-signals-planning.md` — Деловантаж зсередини: сигнали потреби і планування тижня зниження. Fitness-fatigue model (Zatsiorsky & Kraemer, 2006). Плановий vs. реактивний деловантаж. Три категорії сигналів (продуктивність, суб'єктивні, об'єктивні). Структура: -40–60% об'єму, RPE 5–7. Об'ємний vs. інтенсивний деловантаж. Victor-інфраструктура.
- `content/post-1745-training-frequency-muscle-groups.md` — М'язова частота: скільки разів тренувати м'язову групу. Krieger (2010) мета-аналіз. McLester et al. (2000). Вікно MPS 24–48 год. Практичні схеми 3/4/6 днів. Обмежувачі частоти. Взаємодія з High/Low.
- `content/post-1746-rpe-rir-autoregulation-protocol.md` — RPE/RIR як мова навантаження: протокол авторегуляції. Tuchscherer RPE шкала. % 1ПМ vs. RPE — де кожен корисний. Калібровка RPE. RPE-дрейф як сигнал втоми. Протокол першого підходу. Між-тижнева авторегуляція. Helms et al. (2018), Zourdos et al. (2016).
- `content/post-1747-tracking-training-volume.md` — Відстеження тренувального об'єму: мінімально необхідні дані. Мінімальний ввід per підхід і per сесія. Тоннаж vs. робочі підходи (Schoenfeld et al., 2019). Тижневий аудит 5 питань. Типові помилки. Victor як мінімальна достатня система.
- `content/post-1748-muscle-groups-recovery-differentiation.md` — М'язові групи і диференційований підхід до об'єму. Системна vs. локальна втома (Kreher & Schwartz, 2012; Fleck & Kraemer, 2014). Класифікація: прямий і непрямий об'єм. Пріоритетні vs. підтримуючі групи. Практичні правила диференціації.
- `content/post-1749-five-volume-management-mistakes.md` — П'ять помилок в управлінні об'ємом: занадто швидке нарощування; хронічно нижче MAV; пропущені деловантажі; однаковий об'єм для всіх груп; відстеження тільки ваги штанги. Таблиця помилок за рівнем атлета.
- `content/post-1750-cluster-synthesis-volume-management-system.md` — Операційна система управління об'ємом: синтез Cluster 2. Ієрархія концепцій (MEV/MAV/MRV → мезоцикл → H/L → деловантаж → частота → RPE → відстеження). Операційний протокол 3 рівнів: сесія, тиждень, мезоцикл. Практичний приклад 4-тижневого мезоциклу. Victor-роль. 9 ключових принципів. **CLUSTER 2 COMPLETE 10/10.**

### Changed
- `blog.html` — posts 1744–1750 додані (blog updated to post-1750).
- `VERSION` — 6.98.1 → 6.99.0.

## [6.98.1] — 2026-06-20

### Added
- `docs/COST_MODEL.md §42` — Enterprise Contract Renewal Economics: Multi-Year Commitment Pricing, Price Escalation & Renewal ARR Waterfall. Covers renewal pricing mechanics (standard annual at current list, multi-year Year 2+ at CPI+1% capped 5%, retention discount per §32), seat reduction thresholds (≤10% no renegotiation; >10% triggers reset), 90-day renewal notice protocol (RENEW-NOTICE-01 SQL cron, 3 notice states), price escalation formula, Migration 0084 (`enterprise_renewals` table with RLS + REVOKE ALL), 3 DEC-030 HMAC-chained audit events with Zod schemas (`enterprise.renewal_notice_sent`, `enterprise.renewal_escalation_calculated`, `enterprise.contract_renewed`), 2 chain invariants (RENEW-CHAIN-01 / ESCALATION-CHAIN-01 enforced via HTTP 422), Renewal ARR waterfall decomposing into NRR §23.1, blended conversion rate model (~74% raw / ~83% with intervention [ESTIMATE]), 3 SOC 2 evidence artefacts (REN-E-001/002/003 covering CC5.2/CC4.1/A1.1), and a 10-item implementation checklist (P0 M11-12, P1 M12-13, P2 M24-30). enterprise-architect + compliance-officer + security-engineer.

### Changed
- `docs/COST_MODEL.md` — version bump v2.7 → v2.8; TOC extended with §42 entries.
- `VERSION` — 6.98.0 → 6.98.1.

---

## [6.98.0] — 2026-06-20

### Added
- `content/post-1743-high-low-training-principle.md` — Block 1731–1780 · Cluster 2 · Post 3/10. High/Low принцип і хвилеподібне навантаження всередині тижня. Чому рівномірне «середнє» навантаження щодня поступається поляризації важко/легко (Issurin 2010; Seiler & Tønnessen 2009). Daily Undulating Periodization: Rhea et al. (2002) — DUP vs. лінійна прогресія за 12 тижнів, Zourdos et al. (2016). Charlie Francis H/L принцип, адаптований для силового тренінгу. High-сесія: RPE 8–9, 40–60% тижневого об'єму. Low-сесія: RPE 5–7, 10–20% тижневого об'єму, не конкурує з High за відновлення. Практичні структури для 3 і 4 днів/тиждень (H–L–H, Upper/Lower з H/L, Push/Pull/Legs). Розподіл підходів між сесіями: поляризована vs. рівномірна стратегія. Таблиця прикладу двох рівнів хвиль: між тижнями і всередині тижня. Відновлення між High-сесіями: Damas et al. (2016) — 72 год для великих м'язових груп. П'ять помилок при впровадженні H/L. Victor: session RPE, розподіл об'єму між сесіями, суб'єктивне відновлення перед High. clinical-safety: NOT_REQUIRED. sports-scientist review required before publish.

### Changed
- `blog.html` — post-1743 додано вгорі списку.
- `STATUS.md` — Block 1731–1780 Cluster 2 оновлено: post-1743 COMPLETE, наступний post-1744.
- `VERSION` — 6.97.0 → 6.98.0.

---

## [6.97.0] — 2026-06-20

### Added
- `content/post-1742-mesocycle-volume-progression.md` — Block 1731–1780 · Cluster 2 · Post 2/10. Мезоцикл і прогресія об'єму: як будувати хвилю накопичення від MEV до MRV. Чому стабільний об'єм зупиняється: Krieger (2009, 2010) дозозалежна залежність, але без прогресії — адаптаційне плато. Механізм суперкомпенсації на рівні мезоциклу (Yakovlev 1967). Практична структура хвилі тиждень за тижнем: MEV → +2 підходи/тиждень → MRV → деловантаж (конкретний числовий приклад MEV=8, MRV=16, 5+1 тижнів). Формула ставки прогресії: (MRV−MEV)/кількість тижнів. Три форми прогресії об'єму: кількість підходів, кількість повторень, навантаження (тоннаж). Коли хвиля ламається: неочікуваний RPE-стрибок, системне погіршення відновлення, зупинка сили, технічний збій. Деловантаж: 40–60% від пікового тижня, RIR 4+, ваги зберігаються — механіка суперкомпенсації, не «відпочинок». Адаптація для силового піку (хвиля інтенсивності) vs. гіпертрофія (хвиля об'єму). Victor: тижневий об'єм, RPE-тренд, порівняння мезоциклів. clinical-safety: NOT_REQUIRED. sports-scientist review required before publish.

### Changed
- `blog.html` — post-1742 додано вгорі списку.
- `STATUS.md` — Block 1731–1780 Cluster 2 оновлено: post-1742 COMPLETE, наступний post-1743.
- `VERSION` — 6.96.0 → 6.97.0.

---

## [6.96.0] — 2026-06-20

### Added
- `content/post-1741-personal-mev-mav-mrv-calibration.md` — Block 1731–1780 · Cluster 2 · Post 1/10. Персональна калібровка MEV/MAV/MRV: чому таблиця з інтернету — не ваші числа. Hubal et al. (2005): 585 учасників, ідентична програма — від -2% до +250% варіативності відповіді. Що визначає особисті MEV/MAV/MRV: тренувальний вік, м'язово-специфічна відновлюваність, зовнішній стрес (Stults-Kolehmainen & Sinha 2014), сон і харчовий статус (Leproult & Van Cauter 1999). Протокол знаходження MEV (блок 4–6 тижнів, старт від 4–6 підходів, прогресія +2/тиждень до появи адаптаційного сигналу). Протокол виявлення MRV (прогресивне збільшення об'єму + моніторинг RPE-дрейфу і відновлення). MAV-зона як операційний об'єм. Скільки часу займає калібровка і коли повторювати. Шість помилок при калібровці (некалібрований RPE, змішані умови, короткий горизонт, зміна кількох змінних одночасно, ігнорування відновлення). Специфіка по групах: великі vs. малі м'язи. Victor: RPE-тренд, тижневий об'єм, порівняння сесій. clinical-safety: NOT_REQUIRED. sports-scientist review required before publish.

### Changed
- `blog.html` — post-1741 додано вгорі списку.
- `STATUS.md` — Block 1731–1780 оновлено: Cluster 2 розпочато, post-1741 COMPLETE, наступний post-1742.
- `VERSION` — 6.95.0 → 6.96.0.

---

## [6.95.0] — 2026-06-20

### Added
- `content/post-1739-autoregulation-in-practice.md` — Block 1731–1780 · Cluster 1 · Post 9/10. Авторегуляція: як читати власне тіло в реальному часі і коригувати тренування. Визначення авторегуляції (RTS, Тачер). Три інструменти: RPE/RIR (Helms, Zourdos et al., 2016), VBT/bar speed (González-Badillo & Sánchez-Medina, 2010), якість руху. Розминкові підходи як діагностика стану дня. Таблиця авторегуляційних рішень за першим підходом (RPE ±2 від цільового → конкретні дії). Авторегуляція між тижнями: RPE-тренд, блокована прогресія, несподівано легкий тиждень. Жорсткий план vs. авторегуляція: коли що краще; гібридна стратегія. Обмеження: непотренована психологічна готовність, некалібрований RPE, перевантаженість рішеннями, відсутність журналу. Victor: базова лінія RPE, відхилення як сигнал. clinical-safety: NOT_REQUIRED. sports-scientist review required before publish.
- `content/post-1740-cluster-synthesis-volume-load-principles.md` — Block 1731–1780 · Cluster 1 · Post 10/10 · Cluster Synthesis. Операційна рамка управління об'ємом і навантаженням. Чотирирівнева ієрархія: адаптаційний сигнал (MEV) → якість стимулу (MAV/MRV) → організація (частота/deload/прогресія/авторегуляція) → вимірювання (ефективні підходи/RPE-тренд). Практична рамка накопичувального блоку: тиждень 1 (60–65% MRV) → тижні 2–4 (+2 підх/тиждень) → тиждень 5 опційно (95–100% MRV) → deload (40–60%). Операційний діагностичний протокол для чотирьох сценаріїв: прогрес зупинився, RPE-дрейф, невдалий deload, знижена мотивація. Орієнтовна таблиця MEV/MAV/MRV/частоти/тривалості блоку для трьох рівнів. П'ять питань тижневого аудиту. Victor-інтеграція: об'ємний профіль, RPE-базова лінія, тип прогресії, персональний deload-тригер. Синтез 10 постів. clinical-safety: NOT_REQUIRED. sports-scientist review required before publish. **CLUSTER 1 COMPLETE 10/10.**

### Changed
- `blog.html` — posts 1739 і 1740 додано вгорі списку.
- `STATUS.md` — Block 1731–1780 Cluster 1 COMPLETE 10/10. Наступний: Cluster 2 (posts 1741–1750).
- `VERSION` — 6.94.0 → 6.95.0.

---

## [6.94.0] — 2026-06-20

### Added
- `content/post-1738-progressive-overload-mechanisms.md` — Block 1731–1780 · Cluster 1 · Post 8/10. Прогресивне перевантаження: механізми, типи прогресії і чому «завжди додавати вагу» — неповна картина. Принцип SAID (Specific Adaptation to Imposed Demands). Нейром'язова vs. структурна фаза адаптації (Moritani & DeVries, 1979). Сім типів прогресії: 1) лінійна (LP) — початківці, щотижневе збільшення ваги; 2) подвійна (DP) — повторення у діапазоні, потім вага; 3) через об'єм — додавання підходів; 4) RPE-прогресія — зниження RPE при незмінному навантаженні; 5) щільнісна — скорочення відпочинку; 6) технічна — якість руху без зміни параметрів; 7) варіативна — перехід до складнішої варіації. Математика адаптаційної кривої: початківець +2.5–5 кг/тиждень → середній +2.5–5 кг/місяць → досвідчений +2.5–5 кг/3–6 місяців. Вибір механізму за тренувальним рівнем. Прогресивне перевантаження у MEV/MAV/MRV рамці: стопор прогресу часто є проблемою об'єму, не вибором важелів. Victor-відстеження: e1RM (Epley/Brzycki), RPE-тренд, об'єм, силові показники. clinical-safety: NOT_REQUIRED. sports-scientist review required before publish.

### Changed
- `blog.html` — post-1738 додано вгорі списку.
- `STATUS.md` — Block 1731–1780 оновлено: post-1738 COMPLETE, наступний post-1739.
- `VERSION` — 6.93.0 → 6.94.0.

---

## [6.93.0] — 2026-06-20

### Added
- `content/post-1737-deload-structure-principles.md` — Block 1731–1780 · Cluster 1 · Post 7/10. Deload: що це насправді і чому більшість роблять його неправильно. Фізіологія ресинтезу адаптацій: тренувальний стрес як тригер, відновлення як фаза реалізації. Чому повне припинення тренувань ≠ deload: втрата нейром'язового тонусу і моторних патернів після 3–5 днів. Параметри deload: об'єм 40–60% від пікового (основний важіль), інтенсивність збережена або незначно знижена, частота та сама. Два типи: deload за об'ємом (основний) і повний deload (при overreaching). Нижня межа: ~2–3 ефективних підходи/м'яз (підтримувальний рівень). Три тригери: плановий (після 4–5 тижнів), реактивний (при RPE-дрейфі/регресі), контекстуальний (знижені умови відновлення). П'ять поширених помилок: підвищення інтенсивності замість зниження об'єму; надто рідко; надто часто без достатнього накопичення; без планування; кардіо-заміна при незвичній тривалості. Суперкомпенсація: силові показники через 1–2 тижні після deload. Хвиляста альтернатива без явного deload-тижня. Victor: відстеження RPE-тренду і динаміки після deload. clinical-safety: NOT_REQUIRED. sports-scientist review required before publish.

### Changed
- `blog.html` — post-1737 додано вгорі списку.
- `STATUS.md` — Block 1731–1780 оновлено: post-1737 COMPLETE, наступний post-1738.
- `VERSION` — 6.92.0 → 6.93.0.

---

## [6.92.0] — 2026-06-20

### Added
- `content/post-1736-training-frequency-volume-interaction.md` — Block 1731–1780 · Cluster 1 · Post 6/10. Частота тренувань: скільки сесій на тиждень насправді потрібно. Частота як функція розподілу тижневого об'єму, а не самостійна змінна. Schoenfeld, Ogborn & Krieger (2016): 2×/тиждень > 1× при рівному об'ємі (d=0.73). Ralston et al. (2017): для сили різниця менш виражена. MPS-механізм: піки синтезу м'язового білка 24–48 год після сесії (Burd et al., 2010). Пороговий ефект за сесію: ~3–4 ефективних підходи для значущого MPS-сигналу (Baz-Valle et al., 2019). Схеми частоти: 1× (підтримка/обмежений час), 2× (оптимум для більшості), 3× (пріоритетні групи/рухи). Частота для гіпертрофії (MPS-логіка) vs. сили (моторне навчання і нейром'язова адаптація). Вплив тренувального віку на оптимальну частоту. Взаємодія частоти з MEV/MAV/MRV: частота розподіляє, а не множить об'єм. Компроміс реальний розклад vs. теоретичний оптимум. clinical-safety: NOT_REQUIRED. sports-scientist review required before publish.

### Changed
- `blog.html` — post-1736 додано вгорі списку.
- `STATUS.md` — Block 1731–1780 оновлено: post-1736 COMPLETE, наступний post-1737.
- `VERSION` — 6.91.0 → 6.92.0.

---

## [6.91.0] — 2026-06-20

### Added
- `content/post-1735-mrv-maximum-recoverable-volume.md` — Block 1731–1780 · Cluster 1 · Post 5/10. MRV: верхня межа об'єму — де більше вже шкодить. Визначення MRV (maximum recoverable volume): максимальний об'єм, від якого організм може відновитись зі збереженням адаптаційного ефекту. Чому атлети системно заходять за MRV: позитивна петля «більше підходів → кращий результат» і запізнений зворотній зв'язок. П'ять факторів, що зміщують MRV: тренувальний вік, м'язова група, інтенсивність (% 1RM), зовнішній стрес (Stults-Kolehmainen & Sinha, 2014), фаза мезоциклу. Шість ранніх сигналів перевищення: RPE-дрейф при незмінному навантаженні, регрес при зростаючому об'ємі, сповільнене відновлення, деградація якості, зниження мотивації (нейрохімічний механізм, Meeusen et al., 2013), порушення сну. Орієнтовна таблиця MRV по основних м'язових групах. Порівняльна таблиця: функціональне overreaching vs. нефункціональне (5 характеристик). Протокол виходу з перевищення MRV: deload 40–60%, повернення з MEV-рівня. MRV у структурі мезоциклу: 60%→85–95%→100%→deload. Victor-відстеження: RPE-тренд, динаміка об'єму, комбінація «більше підходів + немає прогресу + RPE зростає». clinical-safety: NOT_REQUIRED. sports-scientist review required before publish.

### Changed
- `blog.html` — post-1735 додано вгорі списку.
- `STATUS.md` — Block 1731–1780 оновлено: post-1735 COMPLETE, наступний post-1736.
- `VERSION` — 6.90.0 → 6.91.0.

---

## [6.90.0] — 2026-06-20

### Added
- `content/post-43-when-rpe-lies.md` — Editorial series · post 43. "Коли RPE бреше: ситуації, де суб'єктивна шкала не калібрована." Сім ситуацій де RPE систематично дає хибний сигнал: нова вправа (занижений 1–2 одиниці через нейром'язову неефективність), мотиваційний anchoring (очікування замість вимірювання), накопичена втома (1RM знижується, RPE не реагує; Meeusen et al. 2013), емоційний фон (Rolph et al. 2019), недосип-парадокс (Knowles et al. 2018), ефект першого підходу, зона 9–10 (похибка вдвічі більша; Colbert et al. 2021). Алгоритм калібрування з 5 кроків. Порівняльна таблиця ефектів. Тон: editorial-brutalist. Оцінка 13 хв. clinical-safety: NOT_REQUIRED. sports-scientist review pending.

### Changed
- `blog.html` — post-43-when-rpe-lies додано вгорі списку.
- `STATUS.md` — content engine table row 11–50 оновлено: post-43-when-rpe-lies → draft v6.90.0.
- `VERSION` — 6.89.0 → 6.90.0.

---

## [6.89.0] — 2026-06-20

### Added
- `content/post-1732-tonnage-real-role.md` — Block 1731–1780 · Cluster 1 · Post 2/10. Тоннаж: що він вимірює, де корисний і де обманює. Три способи збільшити тоннаж без прогресу. Де тоннаж справді корисний: архівна перевірка при незмінній структурі, виявлення регресії, розподіл навантаження між групами. Порівняльна таблиця тоннаж / кількість підходів / ефективні підходи — три рівні точності. Як Victor читає тоннаж як допоміжний, а не основний сигнал. clinical-safety: NOT_REQUIRED. sports-scientist review required before publish.
- `content/post-1733-mev-minimum-effective-volume.md` — Block 1731–1780 · Cluster 1 · Post 3/10. MEV: мінімальний ефективний об'єм. Різниця між «тренуватись замало» і «бути нижче MEV». Дозо-відповідний зв'язок (Schoenfeld 2017) і нижній поріг. MEV для гіпертрофії vs. сили (Krieger 2010, Ralston 2017). Чотири фактори, що зміщують MEV: тренувальний вік, м'язова група, якість підходів, стан відновлення. Орієнтовна таблиця MEV по основних м'язових групах. Підтримувальний об'єм vs. MEV (Bickel 2011). MEV і частота. clinical-safety: NOT_REQUIRED. sports-scientist review required before publish.
- `content/post-1734-mav-maximum-adaptive-volume.md` — Block 1731–1780 · Cluster 1 · Post 4/10. MAV: максимальний адаптивний об'єм — оптимальна зона між MEV і MRV. MAV як діапазон, а не фіксоване число. Чотири фактори, що змінюють MAV: тренувальний вік, м'язова група, фаза мезоциклу, зовнішній стрес (Stults-Kolehmainen 2014). MAV для гіпертрофії (12–20 підх/тиждень) vs. сили (4–8 важких підходів). Шість практичних сигналів: у MAV-зоні і за нею. Структура накопичувального блоку: MEV → MAV за 3–5 тижнів. clinical-safety: NOT_REQUIRED. sports-scientist review required before publish.

### Changed
- `blog.html` — posts 1732, 1733, 1734 додані вгорі списку.
- `VERSION` — 6.88.0 → 6.89.0.

---

## [6.88.0] — 2026-06-20

### Added
- `docs/SOC2_READINESS.md §99` — Cross-Reference Patch: DATA_MODEL §42 (`enterprise_contracts` Expansion Tracking Schema · migration 0083 · CC5.2 / CC1.4 / CC4.1 / A1.2). Closes the implicit §90–§98 pattern obligation: every DATA_MODEL section introducing canonical DDL whose SOC 2 evidence artefacts are registered in SOC2_READINESS receives a bidirectional cross-reference patch. §99.1 three-document chain (COST_MODEL §41 → SOC2_READINESS §79.4 v3.23.0 → DATA_MODEL §42). §99.2 seven-row canonical DDL invariant table: `CHECK (current_seats >= initial_seats)` (CC5.2 backstop), `CHECK (expansion_count >= 0)` (CC1.4 counter integrity), `initial_seats` back-fill UPDATE (CC1.4 baseline), `idx_ec_expansion_count` partial index (A1.2 evidence-collection performance), `idx_ec_last_expansion_date` partial index (A1.2 date-range performance), `tenant_contract_portal` view patch excluding `initial_seats` (CC1.4/privacy floor), `tenant_manager` exclusion confirmed (CC4.2). §42.3 staging validation registered as migration artefact. §99.3 two privacy floor supplements: `initial_seats` exclusion from `tenant_contract_portal` (FORM-internal expansion benchmark not exposed to tenant_admin); `tenant_manager` HR exclusion confirmed for all four new columns — zero-query DDL proof for CC4.2. §99.4 four-row criteria mapping supplement (DDL reinforcement): CC5.2 (two independent control layers), CC1.4 (back-fill + counter constraint), CC4.1 (partial index as prerequisite for sustained quarterly EXP-E-002 cadence), A1.2 (§42.6 BDG cache invalidation + `system.scim_guard_cfg_cache_stale` monitoring signal). §99.5 closes two obligations (pattern + §42 one-way reference). §99.6 four-item checklist: 3× P0/M10 (migration validation + artefact, KV invalidation hook, view patch), 1× P1/M12 (first EXP-E-001 with `expansion_count` cross-reference assertion). SOC 2 doc v3.23.0 → v3.24.0.

### Changed
- `VERSION` — 6.87.0 → 6.88.0.

---

## [6.87.0] — 2026-06-20

### Added
- `content/post-1731-training-volume-what-it-measures.md` — Block 1731–1780 Cluster 1, post 1/10. Об'єм тренування: що він справді вимірює і чому це важливо. Три визначення об'єму (тоннаж / підходи / ефективні підходи). Schoenfeld et al. 2017 дозо-реакційний ефект (≥10 підходів/тиждень → superior hypertrophy). MEV/MAV/MRV — введення в рамку. Стеля за сесію (4–6 ефективних підходів). Зовнішній стрес як прихована змінна MRV (Stults-Kolehmainen & Sinha 2014). Victor і відстеження об'єму. clinical_safety: NOT_REQUIRED.

### Changed
- `README.md` — Block 1681–1730: статус оновлено до COMPLETE · v6.86.0. Cluster 5 (posts 1722–1730) → all done. Block 1731–1780 «Управління об'ємом і навантаженням для силового атлета» added as proposed (50 topics, 5 clusters). Post-1731 → draft.
- `blog.html` — card для post-1731 додано перед post-1730.
- `STATUS.md` — Block 1731–1780 IN PROGRESS; post-1731 added; наступний post-1732.
- `VERSION` — 6.86.0 → 6.87.0.

---

## [6.86.0] — 2026-06-20

### Added
- `content/post-1722-movement-condition-matrix.md` — Block 1681–1730 Cluster 5, post 2/10. Матриця рух × умова: таблиця 5 рухів × 6 умов (стандарт / 10–15 хв / 5 хв / змагальний день / без залу / поганий стан) з конкретними протоколами і джерельними постами. 5-крокова схема рішення до тренування. clinical_safety: NOT_REQUIRED.
- `content/post-1723-athlete-level-warmup.md` — Block 1681–1730 Cluster 5, post 3/10. Рівень атлета і розминка: початківець (патерн учиться у розминці), середній (активація, не навчання), досвідчений (протокол ефективності + PAP критичний). Таблиця рівень × пріоритет × навантажені підходи × RPE × PAP × мінімальний час. clinical_safety: NOT_REQUIRED.
- `content/post-1724-bench-full-protocol.md` — Block 1681–1730 Cluster 5, post 4/10. Жим лежачи: повна схема у 5 умовах (стандарт / 10–15 хв / 5 хв / змагальний день / поганий стан). PAP-підхід 92–95% за 8–12 хв до опенера. Три зони підготовки (грудний відділ / плечовий комплекс / зап'ясток). Victor-логування. clinical_safety: NOT_REQUIRED.
- `content/post-1725-squat-full-protocol.md` — Block 1681–1730 Cluster 5, post 5/10. Присідання: повна схема у 5 умовах. Hip CARs + ankle dorsiflexion + bird dog + goblet squat + 5 навантажених підходів (стандарт). PAP 90–95% на змагальний день. Діагностичний RPE при поганому стані. clinical_safety: NOT_REQUIRED.
- `content/post-1726-deadlift-full-protocol.md` — Block 1681–1730 Cluster 5, post 6/10. Становий підйом: специфіка без ексцентрики, задній ланцюг стомлюється швидко. Hip hinge drills (RDL без штанги + hip airplane). PAP для змагань: 85–90% (не 90–95%). Sumo vs. conventional: різна підготовка кульшового. clinical_safety: NOT_REQUIRED.
- `content/post-1727-pullups-rows-protocol.md` — Block 1681–1730 Cluster 5, post 7/10. Підтягування і горизонтальна тяга: протокол без безкоштовного розігріву від попередніх вправ. Scapular CARs, band pull-apart, dead hang. Другорядний рух: 2 підходи. Grip fatigue після станового — окремий параметр. clinical_safety: NOT_REQUIRED.
- `content/post-1728-victor-warmup-profile.md` — Block 1681–1730 Cluster 5, post 8/10. Victor-профіль розминки: що логується (тип, тривалість, флаги, RPE навантажених підходів, RPE першого робочого), що обчислює (RPE-baseline, диференціал за типом), що НЕ робить. Три вимоги для побудови profіlу: ≥30 сесій, однакові умови, мін 3 сесії на рух × тип. clinical_safety: NOT_REQUIRED.
- `content/post-1729-top7-warmup-mistakes.md` — Block 1681–1730 Cluster 5, post 9/10. Топ-7 помилок: рівномірне скорочення, мобільність до загальної, PAP на тренуванні, ритуальна розминка без адаптації, пропуск навантажених підходів, хибна інтерпретація болю (Victor ≠ медична оцінка), відсутність логування. Паттерн → data-сигнал → фікс. clinical_safety: NOT_REQUIRED.
- `content/post-1730-block-synthesis-operational-guide.md` — Block 1681–1730 Cluster 5, post 10/10 · BLOCK COMPLETE. Синтез блоку 1681–1730: операційний довідник силового атлета. Таблиця рух × умова з посиланнями на протоколи (пости 1724–1727, 1719), три принципи, матриця рівня атлета, 5-кроковий текстовий decision tree, Victor-профіль (3 bullet). Довідник, не огляд. clinical_safety: NOT_REQUIRED.

### Changed
- `blog.html` — cards для постів 1722–1730 додано перед post-1721. Статус post-1721 → Cluster 5 COMPLETE. blog.html updated to post-1730.
- `VERSION` — 6.85.0 → 6.86.0.

---

## [6.85.0] — 2026-06-20

### Added
- `content/post-1721-block1681-1730-synthesis-intro.md` — Block 1681–1730 Cluster 5, post 1/10. Синтез блоку 1681–1730: огляд чотирьох кластерів (принципи, нижня частина тіла, верхня частина тіла, особливі умови), три точки перетину між кластерами, три принципи що управляють блоком, дорожня карта Cluster 5 (posts 1722–1730). clinical_safety: NOT_REQUIRED.

### Changed
- `blog.html` — post-1721 card додано перед post-1720. blog.html updated to post-1721.
- `README.md` — Block 1681–1730: додано Cluster 3 (1701–1710), Cluster 4 (1711–1720) і Cluster 5 (1721–1730) з повним переліком тем та статусами. Cluster 2 статус → done. Block статус оновлено до IN PROGRESS v6.85.0.
- `STATUS.md` — Block 1681–1730: Cluster 5 IN PROGRESS; post-1721 added; наступний post-1722.
- `VERSION` — 6.84.0 → 6.85.0.

---

## [6.84.0] — 2026-06-20

### Added
- `content/post-1711-warm-up-time-limited-hierarchy.md` — Cluster 4, post 1/10. Розминка за обмеженого часу: ієрархія скорочення. Три рівні: незнімний мінімум (загальна + навантажені підходи), пріоритет за поточним рухом, що вирізати першим. Правило МЕД при браку часу: три елементи повністю > сім наполовину. clinical_safety: NOT_REQUIRED.
- `content/post-1712-5min-warm-up-protocol.md` — Cluster 4, post 2/10. 5-хвилинний протокол: абсолютний мінімум. Схема: 2 хв загальна (ЧСС 110–130) + 1 хв пріоритетний елемент + 2 хв три підходи штанги. Реальний вплив на RPE порівняно з 0 хв і повною розминкою. clinical_safety: NOT_REQUIRED.
- `content/post-1713-10min-warm-up-protocol.md` — Cluster 4, post 3/10. 10-хвилинний протокол: де знаходиться розрив ефективності. Нелінійна залежність 5 vs. 10 vs. 20+ хв. Схема: загальна (3 хв) + мобілізація+активація (3 хв) + 4 навантажені підходи (4 хв). Порівняльна таблиця за RPE першого підходу. clinical_safety: NOT_REQUIRED.
- `content/post-1714-competition-day-warm-up-strategy.md` — Cluster 4, post 4/10. Змагальний день: логіка підготовки до максимального зусилля. Три принципи: не виснажуйся, калібруй під чергу, PAP. Таблиця часу до виходу × протокол. Ритуальна функція розминки. clinical_safety: NOT_REQUIRED.
- `content/post-1715-competition-bench-press-warm-up.md` — Cluster 4, post 5/10. Змагальна розминка жиму лежачи: повна схема 6 підходів (опенер 120 кг), скорочені за 20 і 15 хв. Специфіка: setup і arch у кожному підході, пауза від 70%+, команди суддів. PAP за 8–12 хв. clinical_safety: NOT_REQUIRED.
- `content/post-1716-competition-squat-deadlift-warm-up.md` — Cluster 4, post 6/10. Змагальна розминка присідання і станового: схеми підходів для обох рухів, рухова підготовка перед підходами, скорочені варіанти. Специфіка: глибина, IAP, страп, відстань від штанги. clinical_safety: NOT_REQUIRED.
- `content/post-1717-travel-warm-up-protocol.md` — Cluster 4, post 7/10. Розминка в подорожі: тіловаговий протокол 10–15 хв. Загальна розминка без обладнання, мобілізація (Lizard Pose, Thread the Needle), активація. Адаптація під нижню/верхню частину тіла. 5-хвилинний мінімум для готелю. clinical_safety: NOT_REQUIRED.
- `content/post-1718-home-warm-up-no-equipment.md` — Cluster 4, post 8/10. Розминка вдома: 12-хвилинний протокол без залу. World's Greatest Stretch, Thoracic Extension на стільці. Адаптація під тіловаговий, пресовий і еспандерний тренінг. Психологічна функція розминки як ритуалу запуску. clinical_safety: NOT_REQUIRED.
- `content/post-1719-warm-up-fatigue-poor-condition.md` — Cluster 4, post 9/10. Розминка при поганому стані (втома/недосип): фізіологічні зміни, діагностичний RPE у навантажених підходах, матриця рішень (RPE 65% → корекція плану). Три помилки. clinical_safety: NOT_REQUIRED.
- `content/post-1720-cluster4-synthesis-special-conditions.md` — Cluster 4, post 10/10 (синтез). Матриця 7 умов × протокол × пріоритет. Дерево рішень. Три спільні принципи. Що Victor відстежує в особливих умовах. **Block 1681–1730 Cluster 4 COMPLETE.** clinical_safety: NOT_REQUIRED.

### Changed
- `blog.html` — posts 1711–1720 додані (10 нових записів перед post-1710). blog.html updated to post-1720.
- `VERSION` — 6.83.1 → 6.84.0.
- `STATUS.md` — Block 1681–1730 Cluster 4 COMPLETE; наступний: Cluster 5 (синтез блоку 1681–1730).

---

## [6.83.1] — 2026-06-20

### Changed
- `docs/OBSERVABILITY.md §6.2` — four SCIM / SSO security alert subsections integrated from §26.8 into the consolidated §6.2 Alert Rules table: `mass_deprovision` (AL-SCIM-MASS-01 — SCIM mass deprovisioning ≥ 10% of seats, P0/P1 dual-level, `docs/INCIDENT_RESPONSE.md R-24`), `scim_endpoint_operations` (AL-SCIM-01 sensitive attribute burst P1, AL-SCIM-02 5xx spike P2, AL-SCIM-03 sync stall P2, AL-SCIM-04 chain invariant violation P1 + R-05 co-activate), `scim_role_history` (AL-SCIM-05 nightly reconciliation gap P2), `sso_browser_security` (AL-SSO-WEB-01 mobile WebView blocked P1 — zero-fire expected state). These subsections were defined in §26.7a/b/c and §26.8 with explicit "(new subsection in §6.2, insert after …)" markers but had not been physically integrated into the §6.2 consolidated table. SOC 2 CC7.2: §26.11's claim of "twenty-one alert rules" covering AL-SCIM-MASS-01, AL-SCIM-01..04, AL-SCIM-05 is now accurate in §6.2. Document header v4.7.2 → v4.7.3.
- `docs/SOC2_READINESS.md §73` — checklist item 10 (P1/M6: add AL-C1-01 to §6.2 `c1_erasure_sla` subsection) marked `[x] Done — 2026-06-09, OBSERVABILITY.md v2.0` — the subsection existed since OBSERVABILITY v2.0 but the SOC2 checklist had not been updated.
- `VERSION` — 6.83.0 → 6.83.1.

## [6.83.0] — 2026-06-20

### Added
- `content/post-42-video-technique-analysis.md` — Editorial post-42 (founder voice · George). Відеоаналіз власної техніки: проприоцептивна ілюзія і розрив між наміром і реалізацією. Ракурси зйомки та що кожен показує (сагітальний, фронтальний, 45°). Технічні маркери для чотирьох основних рухів (присідання, жим, станова, тяга). Де камера принципово сліпа: м'язова активація, IAP, суб'єктивне RPE, структурна морфологія. Типові помилки при перегляді відео (порівняння з елітою, вибір «найкращого» кадру, аналіз без фокусу). Практичний протокол: одне питання → один важкий підхід → одна корекція → цикл. clinical-safety: NOT_REQUIRED.

### Changed
- `blog.html` — added card for post-42-video-technique-analysis (Training Methodology · 13 хв · editorial founder voice).
- `STATUS.md` — 11–50 row updated: post-42-video-technique-analysis added v6.83.0.
- `README.md` — Editorial series header updated: 11–42 drafted. post-42 → draft · v6.83.0. Posts 891–895 proposed (нові теми self-coached атлет).
- `VERSION` — 6.82.0 → 6.83.0.

## [6.82.0] — 2026-06-20

### Added
- `content/post-1701-upper-body-prep-intro.md` — Block 1681–1730 Cluster 3 opener, post 1/10. Рухова підготовка верхньої частини тіла: три зони обмежень (грудний відділ, плечовий комплекс, зап'ястковий комплекс). Self-assessment для кожної зони: Wall Angel + Thoracic Rotation (T-spine), Supine ER + Cross-Body Stretch (плечо), Wrist Extension Test (зап'ясток). Таблиця зон × рухів: що лімітує bench press, OHP та підтягування/ряди. Анонс 1702–1710. clinical-safety: NOT_REQUIRED.
- `content/post-1702-thoracic-spine-mobility.md` — T-spine як «суглоб у середині»: анатомія T1–T12, критичні сегменти для силового атлета (T4–T8 для OHP, T6–T10 для арки, T10–T12 ротація). Три доказові вправи: Foam Roller Extension (сегментна екстензія), Open Book Rotation (ізольована ротація), Cat-Cow з T-spine акцентом. Зв'язок T-spine з жимом лежачи (арка), OHP (вертикальне вирівнювання), підтягуванням (ротаційна рухливість у лопатковій ретракції). Правило двох вправ для обмеженого часу. clinical-safety: NOT_REQUIRED.
- `content/post-1703-shoulder-external-rotation.md` — Зовнішня ротація плеча: механіка субакроміального простору при дефіциті ER, роль infraspinatus + teres minor. Два тести: Supine ER Test (норма ≥90°), Cross-Body Stretch (задня капсула). Side-Lying ER, Sleeper Stretch, Wall ER Drill — три цільові вправи з протоколами. Зв'язок ER з шириною хвата у bench press: <70° = механізм ризику. Диференціація м'якої тканини vs. задньої капсули як джерела обмеження. clinical-safety: NOT_REQUIRED.
- `content/post-1704-lat-shoulder-flexion-overhead.md` — Latissimus dorsi як ключовий обмежувач OHP: анатомія через торако-люмбальну фасцію. Два тести: Doorway Lat Stretch, Overhead Reach (лежачи). Три вправи: Lat Stretch на стійці, Quadruped Lat Stretch, Active Lat Mobilization (вис). Зв'язок з підтягуванням (dead hang = той самий обмежувач). Таблиця дозування за рівнем обмеження (норма / помірне / виражене). clinical-safety: NOT_REQUIRED.
- `content/post-1705-rotator-cuff-activation.md` — Чотири м'язи RotC і їх роль у bench та OHP (таблиця). Принципи активаційної роботи перед тренуванням: малий об'єм, мала вага, після прогріву, повільний темп. Band Pull-Apart (straight + Y), Face Pull, Cuban Press, Empty/Full Can — чотири вправи з протоколами. Дві готові схеми: активація перед bench (5–6 хв) і перед OHP (6–7 хв). clinical-safety: NOT_REQUIRED.
- `content/post-1706-scapular-control-activation.md` — Лопатка як операційний центр тягових рухів. Три м'язові системи: serratus anterior, lower trapezius, middle trap + rhomboids. Тести: Wall Slide (lower trap / SA слабкість), Dead Hang (winging scapula). Чотири вправи: Scapular Pull-Up (ізольована депресія без ліктьового згинання), Band Pull-Apart, Prone YTW, Serratus Push-Up. Лопаткова дисфункція як механізм травматизму — не реабілітація, а стандартна профілактика. clinical-safety: NOT_REQUIRED.
- `content/post-1707-bench-press-warm-up-protocol.md` — Специфічний протокол для bench press як пріоритетного руху дня (~24–28 хв). П'ять кроків: загальна розминка → T-spine → плечова мобілізація → RotC активація → навантажені підходи. Таблиця підйому ваги (приклад 100 кг: 6 проміжних підходів). Шість технічних точок перевірки у підході з порожнім грифом. Скорочений протокол 12–15 хв. Дерево рішень за симптомами. Victor: плечо до/після розминки (1–5). clinical-safety: NOT_REQUIRED.
- `content/post-1708-ohp-warm-up-protocol.md` — Специфічний протокол для OHP (~28–32 хв). Таблиця відмінностей bench vs. OHP (T-spine, плечова флексія, lat, supraspinatus, кор). П'ять кроків: загальна розминка → T-spine + lat мобілізація (7–9 хв, пріоритет) → плечова активація → кор-активація → навантажені підходи. Таблиця підйому ваги (приклад 70 кг). Шість технічних точок. Скорочений протокол 15 хв. Дерево рішень. DB OHP як тимчасова альтернатива при обмеженій ER. clinical-safety: NOT_REQUIRED.
- `content/post-1709-pull-up-row-warm-up-protocol.md` — Специфічний протокол для підтягувань і рядів (~22–26 хв). Специфіка вертикальної vs. горизонтальної тяги. П'ять кроків: гребний ергометр → Active Lat Hang → лопаткова активація (пріоритет) → хват/передпліччя → навантажені підходи. Таблиці навантажених підходів для pull-up і row. Скорочений протокол 12 хв. Дерево рішень: winging, weak grip, неповний ROM. Хват як окремий обмежувальний фактор. clinical-safety: NOT_REQUIRED.
- `content/post-1710-cluster3-synthesis-upper-body-prep.md` — Синтез кластеру 3 (posts 1701–1710). Зведена таблиця: зона × обмеження × інструмент × час × рух (9 рядків). Три дерева рішень: bench, OHP, pull-up/row. Мінімальний протокол 15 хв для трьох рухів. Принцип двошарової підготовки для верхньої частини (мобілізаційний + активаційний, обидва обов'язкові). Victor: 5 точок даних і як використовуються. Анонс Cluster 4 (особливі умови). clinical-safety: NOT_REQUIRED.

### Changed
- `blog.html` — updated to post-1710: added blog cards for posts 1692–1710 (19 нових карток).
- `STATUS.md` — Cluster 2 COMPLETE, Cluster 3 COMPLETE (posts 1701–1710) · v6.82.0. Наступний: Cluster 4.
- `VERSION` — 6.81.1 → 6.82.0.

---

## [6.81.1] — 2026-06-20

### Changed
- `docs/DATA_MODEL.md §42` — `enterprise_contracts` Expansion Tracking Schema Extension (Migration 0083) added as new §42. Closes cross-reference obligation from `docs/COST_MODEL.md §41.6` (v2.7, 2026-06-20). §42.1 four design principles (additive-only migration, `contracted_seats` billing authority unchanged, no new user data, BDG KV invalidation on expansion). §42.2 migration dependency chain (0083 follows 0082; safe zero-downtime ADD COLUMN on Postgres 11+). §42.3 full DDL from COST_MODEL §41.6.1: four ADD COLUMNs with CHECKs, back-fill UPDATE, two partial indexes (`idx_ec_expansion_count`, `idx_ec_last_expansion_date`), five-item staging validation checklist. §42.4 column semantics table (`contracted_seats` billing-authoritative, `initial_seats` immutable post-expansion, `current_seats` mirrors `contracted_seats`, `expansion_count` increment invariant, `last_expansion_date` DATE not TIMESTAMPTZ); `tenant_contract_portal` view patch to exclude `initial_seats` from `tenant_admin` reads. §42.5 RLS behaviour: unchanged policies; `tenant_contract_portal` patched DDL; `tenant_manager` (HR) exclusion rationale. §42.6 BDG cache invalidation sequence: three-step protocol (emit `enterprise.expansion_initiated` → UPDATE `enterprise_contracts` → `billing-event-relay` `SCIM_KV.delete`); failure mode (`system.scim_guard_cfg_cache_stale` LOW/1yr advisory). §42.7 four DEC-030 HMAC-chained events: `enterprise.expansion_initiated` (STANDARD, 7yr, `floor_respected: z.literal(true)` HTTP 422 invariant), `billing.seats_expanded` (STANDARD, 7yr, financial extension), `enterprise.tier_upgraded` (STANDARD, 7yr, `amendment_id` correlation), `enterprise.expansion_floor_enforced` (HIGH, 7yr, `floor_source: 'COST_MODEL_§31.5'` literal required). §42.8 two SOC 2 evidence artefacts: EXP-E-001 (CC5.2/CC1.4, annual `billing.seats_expanded` chain export with `floor_respected: true` attestation + `expansion_count` reconciliation query, 7yr, `compliance/evidence/expansion/`; already registered SOC2_READINESS v3.23.0 §79.4); EXP-E-002 (CC4.1, quarterly adoption health band vs. expansion pipeline cross-reference, 3yr); collection SQL for both artefacts including zero-count attestation protocol. §42.9 eight-item implementation checklist: 3× P0/M10–M11 (migration 0083 deploy + staging validation, BDG KV cache invalidation hook + integration test, Admin Console expansion amendment workflow), 4× P1/M11–M12 (CSM expansion playbook from `qbr_completed`, Admin Console expansion history panel, EXP-E-001 first zero-count filing, OQ-EXP-03 outside counsel engagement), 1× P2/M18 (OQ-EXP-01 expansion probability calibration after 5 events). §42.10 one open question (OQ-EXP-03 — pure tier upgrade: amendment vs. new contract; blocks all pure tier upgrades until resolved; outside counsel P1/M12). DATA_MODEL.md header v1.20 → v1.21.
- `VERSION` — 6.81.0 → 6.81.1.

---

## [6.81.0] — 2026-06-20

### Added
- `content/post-1691-lower-body-movement-prep-intro.md` — Block 1681–1730 Cluster 2 opener, post 1/10. Три зони обмежень нижньої частини тіла (кульшовий комплекс, гомілковостопний суглоб + коліно, люмбопельвічний контроль), механізм і симптоми в присіданні та становій тязі (таблиця: зона × обмеження × симптом у присіданні × симптом у становій). Self-assessment протоколи: кульшовий (присядь без штанги, роторна компенсація тазом), дорсифлексія (kneel-to-wall test, норма 10–15 см), люмбопельвічний контроль (bird-dog, нейтральний таз при мінімальному навантаженні). Рухова підготовка vs. мобілізація: механістична різниця (goblet squat — нейром'язова репетиція, не мобілізація). Правило максимум 2–3 вправи ≤8 хв для специфічного обмеження. Що Victor відстежує: асиметрія в розминці, RPE першого навантаженого підходу, технічний feedback. Анонс 1692–1700. clinical-safety: NOT_REQUIRED. sports-scientist review required before publish.

### Changed
- `blog.html` — картка для post-1691 додана першою позицією.
- `README.md` — Block 1681–1730 доданий у roadmap: Cluster 1 COMPLETE (1681–1690) + Cluster 2 IN PROGRESS (1691–1700) з темами.
- `STATUS.md` — Block 1681–1730 Cluster 2 IN PROGRESS · post-1691 tracked · v6.81.0.
- `VERSION` — 6.80.1 → 6.81.0.

---

## [6.80.1] — 2026-06-20

### Changed
- `docs/SOC2_READINESS.md §79.4` — EXP-E-001 (CC5.2/CC1.4, annual `billing.seats_expanded` chain export with `floor_respected: true` attestation) and EXP-E-002 (CC4.1, quarterly adoption health band vs. expansion pipeline cross-reference) added to master evidence table. Closes COST_MODEL §41.10 item 7 (P1/M11). SOC 2 doc v3.22.1 → v3.23.0.
- `docs/SOC2_READINESS.md §80.3` — `expansion/` subfolder added to R2 `form-soc2-evidence` folder structure with `form-api NO ACCESS` invariant.
- `docs/SOC2_READINESS.md §80.4` — EXP-E-001/EXP-E-002 (§41.9) added to Vanta mirror list.
- `docs/COST_MODEL.md §41.10` — checklist item 7 (P1/M11) marked `[x] Done (2026-06-20, SOC2_READINESS v3.23.0 §79.4/§80.3/§80.4)`.
- `VERSION` — 6.80.0 → 6.80.1.

---

## [6.80.0] — 2026-06-20

### Added
- `content/post-1681-warm-up-function-not-ritual.md` — Block 1681–1730 Cluster 1, post 1/10. Навіщо розминка: функція, не ритуал. Чотири механістичні обґрунтування: термічні ефекти (в'язкість, ферменти, нервова провідність), синовіальна рідина та сполучна тканина, нейром'язовий PAP-праймінг, дві функції (загальна + специфічна). Що розминка НЕ є (не кардіо, не стретчинг, не відновлення). Victor RPE-сигнал при першому робочому підході.
- `content/post-1682-warm-up-structure-general-to-specific.md` — post 2/10. Структура розминки: загальне → специфічне. Чотири рівні: General Elevation (5–8 хв, 110–130 уд/хв), Dynamic Mobility (CARs, ротації), Movement-Specific Rehearsal (порожній гриф), Loaded Warm-up Sets. Типова помилка — пропуск рівня 3. Часовий розподіл для 60 та 90 хв сесій. Victor: моніторинг часу старту першого робочого підходу.
- `content/post-1683-static-stretch-pre-training-science.md` — post 3/10. Статична розтяжка перед тренуванням: дані. Мета-аналіз Kay & Blazevich (2012): >60 с знижує силу на 5–8%. Механізми: механічний (актин-міозин), нейральний (GTO гальмування), жорсткість сухожилка. Безпечне вікно: <30 с. Два виправдані сценарії. Правильний час: після тренування або в дні відпочинку.
- `content/post-1684-dynamic-warm-up-principles.md` — post 4/10. Динамічна розминка: принципи побудови. Порівняння з статичною (таблиця). Ключові категорії: CARs кульшового суглоба, ротація грудного відділу, дорсифлексія, махи ногами, мобілізація плечового пояса. Параметри: 8–10 повторень, контрольований темп, без балістики. Підбір під тренувальний день.
- `content/post-1685-loaded-warm-up-sets-protocol.md` — post 5/10. Розминочні підходи до робочої ваги: протокол. Функція: нейром'язова калібровка під навантаженням. Стандартний протокол (гриф, 40%, 60%, 75–80%, опціонально 90%). Чому відсоткова система, а не фіксовані стрибки. Діагностика через швидкість грифа. Критерій завершення розминки. Паузи між підходами.
- `content/post-1686-pap-potentiation-maximal-effort.md` — post 6/10. Пост-активаційна потенціація (PAP): механізм (MLC phosphorylation + збудливість мотонейронів), вікно 5–10 хв. Контекст застосування: змагання, тестування 1ПМ, блок пікування. Протокол: 85–95% × 1 → 5–8 хв пауза → цільове зусилля. Мета-аналіз Seitz & Haff (2016): ефект тільки у тренованих атлетів (стаж 2+ роки). Відмінність від стандартного розминочного підходу.
- `content/post-1687-warm-up-duration-med.md` — post 7/10. Тривалість розминки: мінімальна ефективна доза. Дані: м'язова температура стабілізується за 10–15 хв; надмірна розминка знижує PCr-пул (Gaitanos et al., 1993). Змінні: температура середовища, початковий стан атлета, вік, тип тренування. Практичний розподіл для 60/75/90 хв. Три типові помилки надмірної розминки.
- `content/post-1688-warm-up-time-constraint-poor-recovery.md` — post 8/10. Компресований протокол при обмеженому часі: що скорочується (загальна, мобілізація, репетиція) і що ніколи не скорочується (навантажені підходи). Поганий стан відновлення: розминочний підхід як тест готовності. RPE 60–70% як кількісна відмітка. Victor запит при аномальному RPE в розминці. Три заборони при поганому відновленні.
- `content/post-1689-warm-up-top5-mistakes.md` — post 9/10. Топ-5 помилок self-coached атлета: (1) статична розтяжка перед важкими підходами, (2) перехід від загальної частини одразу до навантажених, (3) занадто багато розминочних підходів, (4) розминка невідповідного руху, (5) реальне зусилля в розминочних підходах. Спільний знаменник: відсутність розуміння функції кожного елемента.
- `content/post-1690-cluster1-synthesis-warm-up-protocol.md` — post 10/10. Синтез кластеру 1: референсна таблиця (ціль × інструмент × час × позиція), дерево рішень для 60/90 хв / змагальний день / обмежений час. Victor і розминка: чотири параметри моніторингу. Вісім ключових висновків кластеру. Анонс Cluster 2 (1691–1700 · рухова підготовка нижньої частини тіла).
- `content/post-42-velocity-based-training.md` — Editorial series post 42. «Velocity-Based Training — швидкість грифа як ціна повторення» — MCV як об'єктивний сигнал зусилля і готовності; load-velocity профіль і velocity zones; VL% як критерій зупинки підходу (10–15% силовий, 20–25% гіпертрофійний); побудова власного профілю; обладнання від GymAware до суб'єктивної оцінки без датчика. González-Badillo et al. (2014, 2017); Weakley et al. (2021). clinical-safety: PASS. sports-scientist review required before publish.
- `content/post-43-post-activation-potentiation.md` — Editorial series post 43. «Post-Activation Potentiation і PAPE — як важкий підхід робить наступний вибуховіший» — PAP vs PAPE термінологія; три механізми (MLC-фосфорилювання, мотонейронна збудливість via PICs, пеннаційний кут); парадокс потенціація+втома одночасно; оптимальне вікно 5–12 хвилин; практичний complex training і contrast sets; обмеження для новачків. Tillin & Bishop (2003); Seitz & Haff (2016); Cuenca-Fernández et al. (2017). clinical-safety: PASS. sports-scientist review required before publish.

### Changed
- `docs/DECISION_LOG.md` — DEC-078 додано: Block 1681–1730 тема «Розминка та рухова підготовка для силового атлета» (5 кластерів, 50 постів). clinical-safety: NOT_REQUIRED. sports-scientist review required.
- `blog.html` — post-42 і post-43 картки додано; post-1681–1690 картки додано.
- `STATUS.md` — Block 1681–1730 Cluster 1 COMPLETE (10/10 · v6.80.0) додано; editorial series 11–43 (оновлено з 11–40); block 11–50 row оновлено: posts 42, 43 tracked v6.80.0. Blog cards added.
- `VERSION` — 6.79.1 → 6.80.0.

---

## [6.79.1] — 2026-06-20

### Changed
- `docs/DECISION_LOG.md` — DEC-078 зареєстровано (Block 1681–1730 тема).
- `VERSION` — 6.79.0 → 6.79.1.

---

## [6.79.0] — 2026-06-20

### Added
- `docs/AUDIT_LOG_SCHEMA.md §"Enterprise Seat Expansion & Tier Upgrade events"` (v2.21 → v2.22) — Registers four DEC-030 HMAC-chained events from `docs/COST_MODEL.md §41.7` (P0/M10): (1) `enterprise.expansion_initiated` (STANDARD, 7yr) — anchor event on DocuSign countersignature; `amendment_id` UUID correlates the expansion cluster; `floor_respected: z.literal(true)` chain invariant (HTTP 422 if absent); `trigger_type` (csm_initiated / tenant_initiated / milestone_driven); `approved_by` authority enum; `decision_log_ref` required when `discount_pct > 15`. (2) `billing.seats_expanded` (STANDARD, 7yr) — first full AUDIT_LOG_SCHEMA registration; DATA_MODEL §24.5.1 baseline fields (`tenant_id`, `previous_seats`, `new_seats`, `price_per_seat_usd_cents`) + COST_MODEL §41.7.2 financial extension (`expansion_arr_usd`, `net_invoice_usd`, `billing_variant`, `amendment_id`, `floor_respected: z.literal(true)`); EXP-CHAIN-01 successor (HTTP 422 if `expansion_initiated` absent); BDG KV cache-invalidation trigger (SSO_SCIM §35.7). (3) `enterprise.tier_upgraded` (STANDARD, 7yr) — emitted only when `tier_crossing: true` (EXP-CHAIN-02); `old_rate_per_seat_usd`, `new_rate_per_seat_usd`, `tier_crossing_credit_usd`, `multi_year_commitment`. (4) `enterprise.expansion_floor_enforced` (HIGH, 7yr) — emitted synchronously when Admin Console detects `proposed_rate < floor`; FLOOR-CHAIN-01 predecessor to `expansion_initiated`; `requested_rate_usd`, `floor_rate_usd`, `applied_rate_usd`, `floor_source: literal('COST_MODEL_§31.5')`, `founder_approval_required`. Three chain invariants: EXP-CHAIN-01 (`billing.seats_expanded` follows `expansion_initiated`, HTTP 422); EXP-CHAIN-02 (`tier_upgraded` when `tier_crossing: true`, HTTP 422); FLOOR-CHAIN-01 (`expansion_floor_enforced` precedes `expansion_initiated`). Four Zod v2 schemas registered (`ExpansionInitiatedPayload`, `SeatsExpandedPayload`, `TierUpgradedPayload`, `ExpansionFloorEnforcedPayload`). SOC 2 artefacts: EXP-E-001 (CC5.2/CC1.4 annual; 7yr; `compliance/evidence/expansion/EXP-E-001_<YYYY>.csv`) and EXP-E-002 (CC4.1 quarterly; 3yr; `compliance/evidence/expansion/EXP-E-002_<YYYY>-Q<N>.csv`); registration in SOC2_READINESS §79.4/§80.3 pending P1/M11. Retention table: +4 rows. Privacy floor: no `user_id`, no employee name/email, no health data; aggregate contract-level only; `enterprise_contracts` financial rate columns `form_api` REVOKED.

### Changed
- `docs/AUDIT_LOG_SCHEMA.md` — version v2.21 → v2.22; retention table +4 rows; v2.22 footer entry added.
- `docs/COST_MODEL.md §41.10` — checklist item 1 (P0/M10) marked `[x] Done (2026-06-20, AUDIT_LOG_SCHEMA v2.22)`. Document header v2.7 unchanged (patch-level status update).
- `VERSION` — 6.78.0 → 6.79.0.

---

## [6.78.0] — 2026-06-20

### Added
- `content/post-41-discipline-vs-programming-error.md` — Editorial series post 41. «Де закінчується дисципліна — і починається помилка у програмуванні» — п'ять маркерів розмежування: RPE-дрейф, нульовий прогрес 4+ тижнів, технічна регресія у свіжому стані, зникнення варіативності відповіді, відчуття «відбуваю номер»; три рішення (деловантаження+переоцінка / мікроадаптація / перебудова блоку) з алгоритмом вибору; чотирикроковий діагностичний алгоритм; Peterson 2011 (adherence), Hubal 2005 (варіабельність відгуку), Schoenfeld 2010 (progressive overload), Ralston 2017 (1 ефективний підхід), Meeusen et al. 2013 (NFOR у мотивованих атлетів), Zatsiorsky & Kraemer 2006 (принцип варіативності). clinical-safety: NOT_REQUIRED. sports-scientist review required before publish.

### Changed
- `blog.html` — post-41 card додано перед post-1680.
- `STATUS.md` — post-41 → draft v6.78.0; blog.html updated; README 266–270 proposed.
- `README.md` — post-41 → draft · v6.78.0; posts 266–270 added as proposed.
- `VERSION` — 6.77.0 → 6.78.0.

---

## [6.77.0] — 2026-06-20

### Added
- `content/post-1672-pullup-latissimus-accessory.md` — Block 1631–1680 Cluster 5, post 2/10. Широкий спинний у підтягуванні: анатомія у зонах 1 і 2, два типи дефіциту (не відривається від висяння, зупиняється в середній фазі), специфічна допоміжна (straight-arm pulldown, lat pulldown з супінацією, single-arm cable pulldown, негативи), програмування, три типові помилки. clinical-safety: NOT_REQUIRED.
- `content/post-1673-pullup-scapular-stabilizers-accessory.md` — Block 1631–1680 Cluster 5, post 3/10. Лопаткові стабілізатори у тягових рухах: нижній трапець (депресія), зубчастий передній (контроль протракції), ромбоподібні (ретракція). Два режими відмови. Специфічна допоміжна (band pull-apart, face pull, Y/T/W raises, scapular pull-up), порядок активації. clinical-safety: NOT_REQUIRED.
- `content/post-1674-pullup-rear-delt-rhomboid-accessory.md` — Block 1631–1680 Cluster 5, post 4/10. Задній дельт і ромбоподібні: горизонтальне відведення у верхній точці підтягування і фінішна ретракція горизонтальної тяги. Діагностика по зонах підтягування (зона 3) і горизонтальної тяги (зона 3). Специфічна допоміжна (face pull, cable rear delt fly, chest-supported row з паузою, prone Y-T). clinical-safety: NOT_REQUIRED.
- `content/post-1675-pullup-bicep-accessory.md` — Block 1631–1680 Cluster 5, post 5/10. Біцепс у тягових рухах: вторинний рушій у верхній третині, chin-up vs pull-up активація, чотири ознаки дефіциту, диференціація від латисімус-дефіциту і заднього дельта, специфічна допоміжна (supinated lat pulldown, incline DB curl, Zottman curl, hammer curl), пастка «більше curl-ів». clinical-safety: NOT_REQUIRED.
- `content/post-1676-pullup-grip-forearm-accessory.md` — Block 1631–1680 Cluster 5, post 6/10. Хват і передпліччя: три компоненти (crush, pinch, support grip), чотири ознаки ізольованого хват-обмежувача, питання лямок (volume з лямками, grip-тренування окремо), п'ять методів тренування хвату, програмування по частоті. clinical-safety: NOT_REQUIRED.
- `content/post-1677-pullup-core-accessory.md` — Block 1631–1680 Cluster 5, post 7/10. Кор у тяговому русі: антиекстензія у підтягуванні (hollow body у специфічному контексті), антиротація при горизонтальній тязі. Діагностика (розгойдування тіла, ротація тулуба при важкій тязі), п'ять специфічних вправ (RKC plank, Pallof press, hollow hold, anti-rotation cable, dead bug), різниця між корисним і некорисним core-work. clinical-safety: NOT_REQUIRED.
- `content/post-1678-row-specific-accessory.md` — Block 1631–1680 Cluster 5, post 8/10. Горизонтальна тяга: специфічна допоміжна для BB bent-over row, cable seated row і chest-supported row. Чому три варіанти мають різні обмежувачі. Таблиці діагнозу, первинна і вторинна допоміжна, правило лямок у рядах. clinical-safety: NOT_REQUIRED.
- `content/post-1679-pullup-row-accessory-selection-protocol.md` — Block 1631–1680 Cluster 5, post 9/10. Семикроковий протокол добору допоміжних: рух → відео → зона → тип обмежувача → 1-2 вправи → 4–6 тижнів → переоцінка. Дві таблиці Zone × Limiter Type → Accessory (вертикальна і горизонтальна тяга). П'ять помилок у доборі. clinical-safety: NOT_REQUIRED.
- `content/post-1680-cluster5-synthesis-pullup-row-accessory.md` — Block 1631–1680 Cluster 5, post 10/10 (синтез). Зведена карта 13 обмежувачів, чотири кроки діагностики, п'ять типових помилок тягових рухів, push/pull баланс з перспективи тяги, «Що Victor відстежує» (RPE-тренд по зонах, grip failure прапори, чекбокси лопаткової ретракції), адаптаційні горизонти. BLOCK 1631–1680 COMPLETE. clinical-safety: NOT_REQUIRED.

### Changed
- `blog.html` — post cards 1672–1680 додано перед post-1671.
- `STATUS.md` — Block 1631–1680 COMPLETE 50/50 · v6.77.0; blog.html updated to post-1680; total posts 1200.
- `VERSION` — 6.76.0 → 6.77.0.

---

## [6.76.0] — 2026-06-20

### Added
- `docs/COST_MODEL.md §41` — Enterprise Seat Expansion Economics: Mid-Contract Upsell, Tier Upgrade Pricing & Expansion Governance. Closes documentation gap between §40 (adoption health signal, Green-band 35% expansion probability) and §23 (NRR engine, Seat Expansion ARR). Covers: three expansion trigger types (T1 CSM-initiated, T2 tenant-initiated, T3 milestone-driven); minimum increment 25 seats; pro-rata billing formula; tier-crossing credit mechanics; two billing variants (BV-1 immediate, BV-2 anniversary rollup); tier upgrade pricing and commercial conditions; discount authority matrix with six tiers; migration 0083 (`enterprise_contracts` + `initial_seats`, `current_seats`, `expansion_count`, `last_expansion_date`); four DEC-030 HMAC-chained events (`enterprise.expansion_initiated` STANDARD 7yr, `billing.seats_expanded` financial extension STANDARD 7yr, `enterprise.tier_upgraded` STANDARD 7yr, `enterprise.expansion_floor_enforced` HIGH 7yr) — all with `floor_respected: true` chain invariant and no individual user data; adoption-weighted NRR decomposition ($1,112 expected expansion ARR/yr per Starter account at 40/45/15 band distribution); two SOC 2 evidence artefacts (EXP-E-001 CC5.2/CC1.4 annual, EXP-E-002 CC4.1 quarterly); nine-item implementation checklist; three open questions (OQ-EXP-01 probability calibration, OQ-EXP-02 self-serve threshold, OQ-EXP-03 tier upgrade as amendment vs. new contract). Document header updated v2.6 → v2.7.

### Changed
- `docs/COST_MODEL.md` — header v2.6 → v2.7; TOC entry §41 added.
- `VERSION` — 6.75.0 → 6.76.0.

---

## [6.75.0] — 2026-06-20

### Added
- `content/post-1671-pullup-row-accessory-logic.md` — Block 1631–1680 Cluster 5, post 1/10. Логіка допоміжної роботи для підтягувань і горизонтальної тяги: три типи обмежувачів (м'язовий, позиційний, стабілізаційний), три зони підтягування (депресія/відрив, середня фаза, верхня точка) і горизонтальної тяги (розтягнення, середина, фінішна ретракція), відео- і відчуттєва діагностика, відмінність тягових рухів від жимових у стабілізаційному контексті. clinical-safety: NOT_REQUIRED.

### Changed
- `blog.html` — post-1671 card додано перед post-1670.
- `STATUS.md` — Block 1631–1680 Cluster 5 START 1/10 · v6.75.0; total posts 1191.
- `README.md` — Block 1661–1670 додано як draft; Block 1671–1680 додано як proposed.
- `VERSION` — 6.74.0 → 6.75.0.

---

## [6.74.0] — 2026-06-20

### Added
- `content/post-1661-bench-accessory-logic.md` — Block 1631–1680 Cluster 4, post 1/10. Логіка допоміжної роботи для жиму лежачи: три типи обмежувачів (м'язовий, позиційний, стабілізаційний), три зони підйому, відео- і відчуттєва діагностика, принцип вибору допоміжки. clinical-safety: NOT_REQUIRED.
- `content/post-1662-bench-triceps-accessory.md` — Cluster 4 post 2/10. Трицепси у жимі: анатомія трьох головок, ознаки локаут-дефіциту, close grip bench, JM press, board press, overhead extension. Диференціація м'язового vs. позиційного дефіциту. clinical-safety: NOT_REQUIRED.
- `content/post-1663-bench-chest-accessory.md` — Cluster 4 post 3/10. Грудні у нижній точці ROM: чому нижня точка найважча, як bounce маскує дефіцит, pause bench, Larsen press, dumbbell bench як повний-ROM варіант, зв'язок leg drive з нижньою точкою. clinical-safety: NOT_REQUIRED.
- `content/post-1664-bench-front-delts-accessory.md` — Cluster 4 post 4/10. Передні дельти і середня фаза: роль дельти у горизонтальному жимі, incline bench, overhead press, front raise, ризик перетренованості дельти при великому обсязі жиму. clinical-safety: NOT_REQUIRED.
- `content/post-1665-bench-upper-back-arch-accessory.md` — Cluster 4 post 5/10. Верхня спина і arch: функція ретракції лопаток, barbell row, chest-supported row, face pull, lat pulldown, правило 1:1 тяг і жимів, arch у грудному vs. поперековому відділі. clinical-safety: NOT_REQUIRED.
- `content/post-1666-bench-core-leg-drive-accessory.md` — Cluster 4 post 6/10. Корс і leg drive: ланцюг ноги→корс→arch→гриф, ознаки відсутнього leg drive, Larsen press як діагностика, farmer's carry, технічна відпрацювання включення ніг, arch і безпека попереку. clinical-safety: NOT_REQUIRED.
- `content/post-1667-bench-grip-wrist-accessory.md` — Cluster 4 post 7/10. Хват і зап'ясток: правильне положення грифа у долоні, нейтральне зап'ясток, «закрутка» грифа, wrist extension, farmer's carry для хвату, ширина хвату і зміна навантаження, коли бинти маскують проблему. clinical-safety: NOT_REQUIRED.
- `content/post-1668-bench-rotator-cuff-accessory.md` — Cluster 4 post 8/10. Ротаторна манжета: чотири м'язи і їх роль у стабілізації суглоба, зовнішня ротація, face pull, band pull-apart, Y/T/W/L raises, кут опускання і навантаження на манжету, дисбаланс ротаторів при великому обсязі жиму. clinical-safety: NOT_REQUIRED.
- `content/post-1669-bench-accessory-selection-protocol.md` — Cluster 4 post 9/10. Протокол добору: симптом → тип дефіциту → вправа → цикл оцінки. Таблиця рішень, типові помилки, мінімальний стек без діагнозу, частота оцінки результату. clinical-safety: NOT_REQUIRED.
- `content/post-1670-cluster4-synthesis-bench-accessory.md` — Cluster 4 post 10/10 (синтез). Карта обмежувачів жиму лежачи, кроки діагностики, типові помилки, базовий стек, горизонти адаптації, зв'язок із Cluster 5. clinical-safety: NOT_REQUIRED.

### Changed
- `blog.html` — posts 1661–1670 додані перед post-1660 (Cluster 4 COMPLETE).
- `STATUS.md` — Block 1631–1680 Cluster 4 COMPLETE 40/50 · v6.74.0; наступний Cluster 5.
- `VERSION` — 6.73.1 → 6.74.0.

---

## [6.73.1] — 2026-06-20

### Changed
- `docs/SOC2_READINESS.md` — §91.2 GUARD-E-001 collection SQL extended with two §98-directed SQL blocks: (1) §98.2 contracted-seats traceability assertion (DEC-069 CC6.3) — LATERAL JOIN verifying `scim.bulk_deprovision_blocked` `payload.contracted_seats` matches `enterprise_contracts` at block time; (2) §98.3 stale-advisory cross-check (CC7.2) — counts `system.scim_guard_cfg_cache_stale` advisory events per tenant per quarter. §98.5 item 2 marked `[x] Done — 2026-06-20`. SOC 2 doc v3.22.0 → v3.22.1.
- `docs/SSO_SCIM_IMPLEMENTATION.md` — §35.9 item 4 marked `[x] Done (2026-06-20, SOC2_READINESS §98)`. Docs-side obligation fulfilled; `AUDIT_LOG_SCHEMA.md §System` physical registration of `system.scim_guard_cfg_cache_stale` (LOW/1yr) remains pending per §98.5 item 1. Document header v2.7 → v2.8.1 (v2.8 header update omitted at §36 authoring time; corrected). SSO/SCIM doc v2.7 → v2.8.1.
- `VERSION` — 6.73.0 → 6.73.1.

---

## [6.73.0] — 2026-06-20

### Added
- `content/post-261-program-vs-intuition-when-to-deviate.md` — Editorial series post 261. «Програма або інтуїція: коли відступ від плану — правильне рішення, а коли — виправдання». Чотири критерії аутрегуляції (технічна деградація, RPE-відхилення, варіабельність між підходами, тест «аргументу для тренера»), протокол рішення в залі, розбір чому «слухати тіло» без калібрування — не методологія. Zourdos 2016; Helms 2017; Schoenfeld & Grgic 2019; Meeusen et al. 2013. clinical-safety: NOT_REQUIRED. 13 хв read.

### Changed
- `blog.html` — post-261 card added before post-238.
- `STATUS.md` — Editorial 211–265 row updated: post-261 draft · v6.73.0; README roadmap 261–265 proposed.
- `README.md` — Editorial roadmap extended: posts 261–265 added (261 draft, 262–265 proposed).
- `VERSION` — 6.72.0 → 6.73.0.

---

## [6.72.0] — 2026-06-20

### Added
- `content/post-1652-deadlift-erectors-accessory.md` — Cluster 3 post 2/10. Розгиначі спини як обмежувач середньої фази станового підйому. Ознаки дефіциту, back extension зі штангою, RDL з паузою, паузний DL нижче колін, горизонт адаптації 8–12 тижнів.
- `content/post-1653-deadlift-hamstrings-accessory.md` — Cluster 3 post 3/10. Підколінні сухожилля: двосуглобова функція, RDL з паузою, Nordic curl, ексцентричне зміцнення. Суmo-специфіка.
- `content/post-1654-deadlift-glutes-lockout-accessory.md` — Cluster 3 post 4/10. Сідничні м'язи як обмежувач локауту. Hip thrust з ізометрією у верхній точці, паузний DL у верхній третині, pull-through. Суmo-специфіка.
- `content/post-1655-deadlift-quads-liftoff-accessory.md` — Cluster 3 post 5/10. Квадрицепси у відриві від підлоги. Механіка першої фази, deficit deadlift, паузний DL у перших 2–3 см, front squat як загальний об'єм.
- `content/post-1656-deadlift-upper-back-lats-accessory.md` — Cluster 3 post 6/10. Верхня спина і широка спина: контроль грифа і стабілізація. Straight arm pulldown для latissimus, barbell row для верхньої спини. Різниця між двома функціями.
- `content/post-1657-deadlift-core-iap-accessory.md` — Cluster 3 post 7/10. Корс і IAP як фундамент передачі зусилля. Техніка брейсингу як передумова. Farmer's carry, weighted plank, pallof press, deadbug.
- `content/post-1658-deadlift-adductors-hip-sumo-accessory.md` — Cluster 3 post 8/10. Аддуктори і кульшовий суглоб: специфіка суmo. Copenhagen plank, sumo RDL, adductor machine, lateral lunge. Мобільність кульшового суглоба як окрема змінна.
- `content/post-1659-deadlift-accessory-selection-protocol.md` — Cluster 3 post 9/10. Покроковий протокол добору допоміжних вправ: від симптому до зони обмежувача, від типу дефіциту до конкретного інструменту і розміщення у програмі.
- `content/post-1660-cluster3-synthesis-deadlift-accessory.md` — Cluster 3 post 10/10 (синтез). Карта обмежувачів станового підйому, кроки діагностики, типові помилки, базовий стек без діагнозу, горизонти адаптації. **CLUSTER 3 COMPLETE 10/10.**

### Changed
- `blog.html` — posts 1652–1660 added (9 нових записів перед post-1651).
- `STATUS.md` — Block 1631–1680 Cluster 3 updated: COMPLETE 10/10 · v6.72.0; наступний: Cluster 4 (posts 1661–1670 · допоміжна для жиму лежачи).
- `VERSION` — 6.71.1 → 6.72.0.

---

## [6.71.1] — 2026-06-20

### Changed
- `docs/SOC2_READINESS.md` — §98 Cross-Reference Patch: SSO_SCIM §35 (DEC-069 · OQ-SSO-34.2) · BDG `getGuardConfig()` seat source audit traceability (CC6.3 / A1.2 / CC7.2). Supplements §91 (GUARD-E-001) with §35.7 three-step reconstruction protocol, `contracted_seats` assertion SQL, A1.2 KV decoupling rationale, and `system.scim_guard_cfg_cache_stale` CC7.2 advisory monitoring signal. SOC 2 doc v3.21.2 → v3.22.0.
- `VERSION` — 6.71.0 → 6.71.1.

---

## [6.71.0] — 2026-06-20

### Added
- `content/post-1651-deadlift-accessory-logic.md` — Block 1631–1680 Cluster 3, post 1/10. Логіка допоміжної роботи для станового підйому: три типи обмежувачів (м'язовий, позиційний, стабілізаційний), три зони підйому (відрив, середина, локаут), діагностика за відео і відчуттями, принцип точкового вибору 1–2 вправ, антишаблон «допоміжки для всього». clinical_safety: NOT_REQUIRED.
- `blog.html` — картка post-1651 додана перед post-1650 (Programming, draft, 8 хв).
- `README.md` — Blocks 1606–1660 додані до roadmap: pullup/row completion (1606–1610), overhead press (1611–1620), movement system synthesis (1621–1630), accessory principles cluster 1 (1631–1640), squat accessory cluster 2 (1641–1650), deadlift accessory cluster 3 proposed (1651–1660).

### Changed
- `STATUS.md` — Block 1631–1680: Cluster 2 COMPLETE → Cluster 3 started (post-1651, 1/10); README roadmap updated to v6.71.0.
- `VERSION` — 6.70.0 → 6.71.0.

---

## [6.70.0] — 2026-06-20

### Added
- `content/post-1641-squat-accessory-logic.md` — Block 1631–1680 Cluster 2, post 1/10. Логіка допоміжної роботи для присідання: три типи обмежувачів (м'язовий, позиційний, стабілізаційний), діагностика за відео і відчуттями, принцип «1–2 вправи на обмежувач».
- `content/post-1642-squat-quad-accessory.md` — Cluster 2, post 2/10. Квадрицепси як обмежувач у присіданні: ознаки дефіциту, паузне присідання / front squat / goblet squat як специфічна допоміжна, обсяг і програмування.
- `content/post-1643-squat-posterior-chain-accessory.md` — Cluster 2, post 3/10. Задній ланцюг у присіданні: обмежувач верхньої третини, RDL / good morning / hip thrust / box squat, розведення по тренувальних днях.
- `content/post-1644-squat-bottom-position-accessory.md` — Cluster 2, post 4/10. Паузне присідання і box squat як технічні інструменти: функціональна різниця, правила ваги і тривалості паузи, коли обидва зайві.
- `content/post-1645-squat-core-stability-accessory.md` — Cluster 2, post 5/10. Кор і стабілізація у присіданні: IAP-механізм, farmer's carry / overhead squat / pallof press / dead bug, відмінність від скручувань.
- `content/post-1646-squat-knee-stability-accessory.md` — Cluster 2, post 6/10. Колінна стабільність і відведення стегна: три причини вальгусу, clamshell / lateral band walk / BSS / banded squat, роль стопи.
- `content/post-1647-squat-hip-mobility-accessory.md` — Cluster 2, post 7/10. Кульшовий суглоб і присідання: чотири мобільнісних компоненти, тести на обмеження, banded ankle mobilization / deep squat hold / 90/90 / cossack squat; пасивна vs. активна мобільність.
- `content/post-1648-squat-upper-back-accessory.md` — Cluster 2, post 8/10. Верхня спина у присіданні: ознаки слабкості, face pull / chest-supported row / yoke carry як специфічна допоміжка для вертикального стиснення.
- `content/post-1649-squat-accessory-volume-frequency.md` — Cluster 2, post 9/10. Обсяг і частота допоміжної роботи для присідання: орієнтири за категорією, правило «не більше 2–3 вправ одночасно», тижнева рамка для 3–4 тренувань.
- `content/post-1650-cluster-synthesis-squat-accessory.md` — Cluster 2, post 10/10 (синтез). Операційний протокол: п'ять кроків від симптому до оцінки ефекту, таблиця симптом→вправа, антишаблони.
- `blog.html` — posts 1641–1650 додані до каталогу (Programming, draft).

### Changed
- `STATUS.md` — Block 1631–1680: Cluster 1 → Cluster 2 COMPLETE 10/50; next: Cluster 3 (posts 1651–1660 · допоміжна для станового підйому).
- `VERSION` — 6.69.1 → 6.70.0.

---

## [6.69.1] — 2026-06-20

### Added
- `docs/SOC2_READINESS.md §97` — Cross-reference patch: formally registers CC6-E-CAEP-001/002/003/004 (SSO_SCIM §23 CAEP/SSF real-time session revocation architecture — CC6.3/CC6.6/CC7.2/CC7.3) in SOC2_READINESS §79.4. These four evidence artefacts were defined in SSO_SCIM §23.10 (v1.5, 2026-06-01) with SOC 2 criteria mapping but never cross-referenced here; §94 covered DEC-072 controls (§36) only. Artefacts registered: CC6-E-CAEP-001 (caep_events distribution, CC6.3, quarterly/3yr), CC6-E-CAEP-002 (audit_log sso.caep_* HMAC chain, CC6.3/CC7.3, quarterly/7yr — referenced in MSA Addendum 6.4 as primary SLA evidence), CC6-E-CAEP-003 (tenant CAEP stream registration snapshot sanitized, CC6.3, quarterly/3yr), CC6-E-CAEP-004 (AL-CAEP-01/03 PagerDuty incident history, CC7.2/CC7.3, quarterly/3yr). SOC2_READINESS v3.21.1 → v3.21.2.

### Changed
- `docs/SOC2_READINESS.md §94` — Stale reference patch (§97.4): corrected `docs/MSA_TEMPLATE.md §Addendum 3` → `§Addendum 6` and `docs/ENTERPRISE_ONBOARDING.md §2.4` → `§2.3` in §94 intro, §94.4 CC6.3 criteria row, and §94 version footer (4 occurrences). Root cause: §94 authored 2026-06-19 before MSA v0.5 + ENTERPRISE_ONBOARDING §2.3 published on 2026-06-20 (v6.67.0).
- `VERSION` — 6.69.0 → 6.69.1.

## [6.69.0] — 2026-06-20

### Added
- `content/post-238-does-program-fit-you.md` — Editorial series 231–250, post 9/10. «Як оцінити, чи програма підходить саме тобі — без тренера і без n=1 гадань». П'ять сигналів відповідності: патерн відновлення, технічна якість під навантаженням, тренд сили/RPE, поведінка між сесіями, накопичення мікротравматизму. Структурований self-assessment після 4-тижневого блоку. clinical-safety NOT_REQUIRED.
- `blog.html` — картка post-238 додана перед post-237.
- `README.md` — post-238 → draft; нові теми 257–260 proposed.

### Changed
- `STATUS.md` — рядок Editorial 211–250: post-238 додано, count 8→9, roadmap 257–260 proposed.
- `VERSION` — 6.68.0 → 6.69.0.

## [6.68.0] — 2026-06-20

### Added
- `content/post-1631-accessory-work-role-self-coached.md` — Block 1631–1680 Cluster 1, post 1/10. Що таке допоміжна вправа і для чого вона потрібна self-coached атлету: визначення, дві ролі (усунення слабкої ланки і загальний руховий резервуар), практичний тест на корисність вправи. clinical-safety NOT_REQUIRED.
- `content/post-1632-accessory-work-classification-gpp-spp.md` — Block 1631–1680 Cluster 1, post 2/10. Класифікація допоміжної роботи: GPP (загально-підготовча), SPP (специфічно-підготовча), виправляючі вправи. Три категорії з різною логікою програмування, інтенсивності і місця в тижні. clinical-safety NOT_REQUIRED.
- `content/post-1633-accessory-work-minimum-effective-dose.md` — Block 1631–1680 Cluster 1, post 3/10. Принцип мінімальної ефективної дози для допоміжної роботи. Чому self-coached атлети переходять MED, алгоритм визначення мінімуму, довідкові значення за стажем. clinical-safety NOT_REQUIRED.
- `content/post-1634-main-lifts-vs-accessory-priority.md` — Block 1631–1680 Cluster 1, post 4/10. Ієрархія пріоритетів: основні підйоми завжди першими. Ієрархія скорочення при нестачі часу, єдиний сценарій для тимчасової зміни пріоритету. clinical-safety NOT_REQUIRED.
- `content/post-1635-choosing-accessory-exercises-criteria.md` — Block 1631–1680 Cluster 1, post 5/10. Чотири критерії вибору допоміжних вправ: локалізація обмеження, рівень специфічності, доповнення vs. дублювання, технічна доступність. Алгоритм фільтрації. clinical-safety NOT_REQUIRED.
- `content/post-1636-accessory-intensity-to-failure.md` — Block 1631–1680 Cluster 1, post 6/10. Інтенсивність допоміжної роботи: чи варто доходити до відмови. Технічна vs. м'язова відмова, RIR-таблиця для кожної категорії вправ, виняток для гіпертрофії. clinical-safety NOT_REQUIRED.
- `content/post-1637-deload-accessory-work-strategy.md` — Block 1631–1680 Cluster 1, post 7/10. Деload для допоміжної роботи: volume deload vs. intensity deload, пропорції зниження, що залишати і що прибирати, чеклист деload-тижня. clinical-safety NOT_REQUIRED.
- `content/post-1638-tracking-accessory-work-logs.md` — Block 1631–1680 Cluster 1, post 8/10. Відстеження допоміжної роботи: різна логіка для SPP, GPP і виправляючих вправ. Як перевіряти кореляцію між прогресом допоміжки і основного підйому. clinical-safety NOT_REQUIRED.
- `content/post-1639-top5-accessory-programming-mistakes.md` — Block 1631–1680 Cluster 1, post 9/10. Топ-5 помилок у програмуванні допоміжної роботи для self-coached атлетів. Самодіагностична таблиця. clinical-safety NOT_REQUIRED.
- `content/post-1640-cluster-synthesis-accessory-principles.md` — Block 1631–1680 Cluster 1, post 10/10. Синтез: шестикрокова операційна карта допоміжної роботи self-coached атлета. Чеклист одного тренування. Анонс Cluster 2 (допоміжна для присідання). clinical-safety NOT_REQUIRED.
- `docs/DECISION_LOG.md §DEC-077` — Block 1631–1680 тема: Допоміжна робота і спеціалізовані вправи. 5 кластерів: принципи (1631–1640), присідання (1641–1650), тяга (1651–1660), жим (1661–1670), інтеграція + синтез (1671–1680).
- `blog.html` — 10 нових карток (post-1631–1640, категорія Programming) додано на початок стрічки.

### Changed
- `VERSION` — 6.67.0 → 6.68.0.

---

## [6.67.0] — 2026-06-20

### Added
- `docs/MSA_TEMPLATE.md §Addendum 6` — CAEP/SSF Real-Time Session Control SLA Addendum v1.0. Closes `docs/SSO_SCIM_IMPLEMENTATION.md §36.6` item 6 (P1/M5 · DEC-072). Six sections: §6.1 purpose/scope (baseline JWT TTL vs. CAEP path), §6.2 real-time SLA ("less than 60 seconds in normal operation", PUSH prerequisite, polling excluded, SLA credit mechanism), §6.3 Customer Obligations (maintain PUSH; notify CSM on change; confirm `active` stream), §6.4 SOC 2 evidence (CC6-E-CAEP-002; privacy floor: user_id_hash + tenant_id only), §6.5 Governing law, §6.6 Outside counsel requirement. Order Form: CAEP SLA checkbox; Exhibits: Addendum 6 entry; Internal Notes: Addendum 6 guidance row. Applies to Okta / Entra ID / Google Workspace OIDC only; non-PUSH IdPs → JWT TTL baseline.
- `docs/ENTERPRISE_ONBOARDING.md §2.3` — CAEP/SSF IdP prerequisite verification. Closes `docs/SSO_SCIM_IMPLEMENTATION.md §36.6` item 7 (P1/M5 · DEC-072). Five-row IdP compatibility table. Two SLA paths: CAEP Addendum 6 (< 60 s) vs. JWT TTL baseline (≤ 15 min). Five-item CSM checklist including IdP confirmation script, Linear tracker field, Addendum 6 gate, and post-go-live CAEP stream status check.

### Changed
- `docs/MSA_TEMPLATE.md` — v0.4 → v0.5 (Addendum 6 + Order Form + Exhibits + Internal Notes).
- `docs/ENTERPRISE_ONBOARDING.md` — v0.2 → v0.3 (§2.3 added).
- `docs/SSO_SCIM_IMPLEMENTATION.md` — §36.6 items 6 and 7 marked `[x] Done`.
- `VERSION` — 6.66.0 → 6.67.0.

---

## [6.66.0] — 2026-06-20

### Added
- `content/post-237-self-progress-reading.md` — Editorial 211–250 series post 8/10. «Як читати власний прогрес без зовнішнього тренера: п'ять запитань, що замінюють погляд збоку». П'ять структурованих запитань, що відтворюють зовнішній аналітичний погляд для self-coached атлета: читати журнал від третьої особи, валідність метрики, тренд а не точка, gap між записаним і реальним, поведінкові патерни. clinical-safety NOT_REQUIRED. Dunning & Kruger 1999; Kross et al. 2014; Helms 2018; Schoenfeld 2017. 13 хв.

### Changed
- `blog.html` — додано картку post-237 перед post-236.
- `STATUS.md` — Editorial 211–250: 7 → 8 постів; post-237 (self-progress-reading) зафіксовано; README roadmap updated: posts 254–256 proposed.
- `VERSION` — 6.65.0 → 6.66.0.

---

## [6.65.0] — 2026-06-20

### Added
- `content/post-1621-movement-system-five-patterns.md` — Block 1571–1630 Cluster 6 post 1/10. Рух як система: posterior chain і core як спільний фундамент, CNS як спільний ресурс (48–72 год відновлення після субмаксимального навантаження), push/pull і squat/hinge баланс, три класичних приклади переносу дефіциту між патернами.
- `content/post-1622-movement-transfer-and-conflicts.md` — Cluster 6 post 2/10. Перенос і конфлікти між рухами: мобільність стегна (присідання ↔ становий), lat tightness (тяга ↔ жим над головою), серрат (підтягування ↔ overhead), торакальна мобільність як shared resource. Алгоритм читання симптому в русі B як дефіциту в русі A.
- `content/post-1623-weekly-technical-programming.md` — Cluster 6 post 3/10. Технічне програмування тижня: принцип технічної свіжості, присідання перед становим, жим над головою не після важкого bench, push/pull баланс у розкладі. Шаблони 3-денного і 4-денного тижня.
- `content/post-1624-systemic-weak-point-diagnosis.md` — Cluster 6 post 4/10. Системна діагностика: протокол 5-рухового скринінгу при 50–60% RM, матриця обмежувачів по типу (мобільність / сила / координація), логіка ранжування пріоритетів.
- `content/post-1625-push-pull-balance.md` — Cluster 6 post 5/10. Push/pull баланс: анатомія дисбалансу, правило 1:1 і 1:1,5, три методи виявлення, чотири способи корекції без повного перепрограмування.
- `content/post-1626-recovery-between-patterns.md` — Cluster 6 post 6/10. Відновлення між патернами: м'язове vs. нейром'язове відновлення, таблиця overlap, правила 48–72 год, коли суб'єктивне погіршення другого руху є сигналом до перегляду розкладу.
- `content/post-1627-cross-movement-video-audit.md` — Cluster 6 post 7/10. Відеодіагностика між рухами: протокол два ракурси × 5 рухів при 50–60% RM, повторювані постуральні патерни, формат таблиці з колонкою «повторюється в», архівування і порівняння між мезоциклами.
- `content/post-1628-systemic-technical-plateau.md` — Cluster 6 post 8/10. Системне технічне плато: три причини (CNS-перевтома, обсяг > засвоєність, брак технічної роботи), протокол виходу (технічний deload → скринінг → один фокус → поступове повернення).
- `content/post-1629-self-coached-technical-audit.md` — Cluster 6 post 9/10. Самостійний технічний аудит: покроковий протокол 6 кроків (підготовка, виконання, перегляд P0–P3, таблиця результатів, ранжування, один фокус на мезоцикл), порівняння між мезоциклами.
- `content/post-1630-block-synthesis-1571-1630.md` — Cluster 6 post 10/10. **Block synthesis 1571–1630**: три наскрізних принципи (трирівнева правильність, варіативність поверх обмежень, тип обмежувача → рішення), чотири принципи Cluster 6, операційна система self-coached атлета. **BLOCK 1571–1630 COMPLETE 60/60.**
- `docs/DECISION_LOG.md` — DEC-076: Cluster 6 (posts 1621–1630) — розширення блоку до 60 постів, тема «Інтеграція рухових патернів».

### Changed
- `blog.html` — додано картки post-1621 через post-1630 перед post-1620.
- `STATUS.md` — Block 1571–1630 Cluster 6 COMPLETE позначено; block 1571–1630 COMPLETE 60/60.
- `VERSION` — 6.64.0 → 6.65.0.

---

## [6.64.0] — 2026-06-20

### Added
- `content/post-236-real-week-programming.md` — Editorial 211–250 series post 7/10. «Програма під реальний тиждень: чому більшість планів тренувань припускають те, що не існує». П'ять принципів: якірні сесії, мінімальна ефективна сесія, ієрархія рухів, тижневий перегляд, облік нестабільних тижнів. clinical-safety NOT_REQUIRED. 13 хв.

### Changed
- `blog.html` — додано картку post-236 перед post-235.
- `STATUS.md` — Editorial 211–250 оновлено: post-236 added v6.64.0; posts count 5→7; серія оновлена.
- `VERSION` — 6.63.0 → 6.64.0.

---

## [6.63.0] — 2026-06-20

### Added
- `content/post-1611-overhead-press-individual-variation.md` — Block 1571–1620 Cluster 5 post 1/10. Що таке «правильний» жим над головою. Три рівні правильності (безпека / механічна ефективність / індивідуальна відповідність). Що варіюється між атлетами: хват, кут ліктів, нахил тулуба, зап'ястки, лопатка. Що є універсальним: upward rotation лопатки, відсутність цервікальної гіперекстензії, гриф над вухами. clinical_safety: NOT_REQUIRED.
- `content/post-1612-overhead-press-anthropometry-grip.md` — Cluster 5 post 2/10. Антропометрія і вибір ширини хвату. Biacromial distance як відправна точка: 1,0–1,5×. Довжина передпліч і кут ліктів — elbow flare як кінематична адаптація, не помилка. Стіновий тест як скринінг грудної мобільності. Saeterbakken & Fimland (2013), Schwanbeck et al. (2009). Практична матриця хвату за антропометрією.
- `content/post-1613-overhead-press-emg-muscle-activation.md` — Cluster 5 post 3/10. EMG-дані для жиму над головою та варіацій. Передній дельтоїд як первинний рушій. Standing vs. seated: вища активація стабілізаторів стоячи, вище абсолютне навантаження сидячи. Штанга vs. гантелі: гантелі виявляють асиметрії. Variation-specific EMG таблиця.
- `content/post-1614-scapular-mechanics-overhead-press.md` — Cluster 5 post 4/10. Механіка лопатки у вертикальному жимі. Чотири фази: задній нахил → верхня ротація → зовнішня ротація → елевація. Роль serratus anterior і трапеції. Три дисфункції: шраг-патерн, передній нахил, крилоподібна лопатка. Три скринінги: wall slide, band pull-apart, overhead reach.
- `content/post-1615-overhead-press-variations-matrix.md` — Cluster 5 post 5/10. Варіації жиму над головою і матриця вибору. Шість варіацій з біомеханічними профілями: strict barbell, seated, dumbbell, push press, Arnold, landmine. Матриця вибору за метою. Принцип комбінування: основна + 1–2 допоміжних.
- `content/post-1616-overhead-press-rom-lockout.md` — Cluster 5 post 6/10. ROM у жимі над головою. Повний ROM: від ключиці до локауту над вухами. McMahon et al. (2014) і Pedrosa et al. (2022) — розтягнута позиція і гіпертрофічний стимул. Bar path як крива навколо обличчя. Стіновий тест для самодіагностики. Часткові повтори: інструмент, не стратегія за замовчуванням.
- `content/post-1617-overhead-press-video-diagnosis.md` — Cluster 5 post 7/10. Відеодіагностика двох ракурсів. Бічний: bar path, поперековий прогин, локаут. Фронтальний: асиметрія, колапс зап'ясток. P0–P3 таксономія для OHP. Протокол перегляду 0,25×. Режим: бічний кожні 2–3 тренування, фронтальний раз на мезоцикл.
- `content/post-1618-overhead-press-technical-limiters-matrix.md` — Cluster 5 post 8/10. Три типи обмежувачів (мобільність / сила / моторний контроль) і матриця корекції. Діагностичний алгоритм. Матриця корекції: грудна мобільність, зовнішня ротація, serratus, нижня трапеція, патерн. Правило одного обмежувача за мезоцикл.
- `content/post-1619-overhead-press-progression-zero-to-weighted.md` — Cluster 5 post 9/10. П'ять ступенів прогресії: активаційний фундамент (wall slides, shoulder CARs) → технічна база (порожній гриф, темп 2-1-2) → накопичення об'єму (30–60%) → силовий розвиток (60–80%, 4×5–6) → навантажений жим + push press. Критерії переходу між ступенями.
- `content/post-1620-overhead-press-cluster-synthesis.md` — Cluster 5 post 10/10 (block synthesis). Три рівні операційної системи: що визначити один раз (overhead assessment, обмежувач, варіація, хват), що перевіряти щотренування (активація, ROM, відео), що переглядати на мезоциклі (P0–P3 аналіз, переоцінка обмежувача, ротація). Анонс Cluster 6. sports-scientist review required.

### Changed
- `blog.html` — додано картки posts 1611–1620 у верхній секції списку.
- `VERSION` — 6.62.1 → 6.63.0.

---

## [6.62.1] — 2026-06-20

### Fixed
- `docs/OBSERVABILITY.md` v4.7.2 — pg_cron job-number conflict resolution: `sso_fleet_health_check` renumbered from job 36 to job 38 throughout §49 and §12.6. Job 36 was already assigned to `dsar_slo_miss_counter_reset` in the §12.6 v0.4 patch (canonical authority). Job 38 row added to §12.6 registry table. §12.6 freshness window note and conflict resolution note (v0.5) updated.
- `docs/SOC2_READINESS.md` v3.21.1 — §95 pg_cron job citations corrected to job 38 (§95.1, §95.2, §95.3 attestation JSON fields `job38_gaps_over_6min`/`job38_run_details_`, §95.3 pre-M10 note, §95.4 CC7.2/CC7.3, §95.5, §95.6 checklist items 1–3, version footer).

### Changed
- `VERSION` — 6.62.0 → 6.62.1.

---

## [6.62.0] — 2026-06-20

### Added
- `content/post-235-discipline-vs-programming-error.md` — Editorial 211–250 series post 6/10. «Де закінчується дисципліна — і починається помилка у програмуванні». П'ять сигналів кожного сценарію + тест одного запитання для розрізнення. clinical-safety NOT_REQUIRED. 13 хв.

### Changed
- `blog.html` — додано картку post-235 перед post-234.
- `README.md` — post-235 → draft · v6.62.0; додано proposed posts 251–253.
- `STATUS.md` — Editorial 211–250 оновлено: post-235 added v6.62.0.
- `VERSION` — 6.61.0 → 6.62.0.

---

## [6.61.0] — 2026-06-20

### Added
- `content/post-1601-pullup-individual-variation.md` — Block 1571–1620 Cluster 4 post 1/10. Що таке «правильне» підтягування і чому відповідь залежить від структури тіла. Три рівні правильності, чотири змінні (хват, тип, глибина, кут тулуба), чотири універсальних параметри.
- `content/post-1602-pullup-anthropometry-grip.md` — Cluster 4 post 2/10. Антропометрія і вибір хвату: ширина (вузький/плечовий/широкий), тип пронація/супінація/нейтраль, кут. Таблиця антропометрія → вибір хвату. Lehman et al. (2004) ЕМГ chin-up vs. pull-up.
- `content/post-1603-pullup-row-emg-muscle-activation.md` — Cluster 4 post 3/10. EMG-дані для підтягувань і горизонтальної тяги. Latissimus, біцепс, нижня трапеція, ромбоподібні, задня дельта. Signorile (2002), Fenwick (2009). Вертикальна і горизонтальна тяга — взаємодоповнення, не дублювання.
- `content/post-1604-scapular-mechanics-pulling.md` — Cluster 4 post 4/10. Механіка лопатки у тягових рухах: чотири фази у підтягуванні, три найчастіші порушення (scapular shrug, winging, тяга без ретракції), три тести для self-coached атлета.
- `content/post-1605-horizontal-row-variations.md` — Cluster 4 post 5/10. Горизонтальна тяга: варіації та їх механіка. Bent-over row, seated cable row, T-bar, inverted row. Матриця вибору за метою.
- `content/post-1606-pullup-rom-lockout.md` — Cluster 4 post 6/10. ROM у підтягуваннях: dead hang vs. «майже розгинання», підборіддя vs. груди, часткові повтори. McMahon et al. (2014), Pedrosa et al. (2022) — тренування у розтягнутій позиції і гіпертрофічний стимул.
- `content/post-1607-pullup-row-video-diagnosis.md` — Cluster 4 post 7/10. Відеодіагностика двох ракурсів для підтягування і bent-over row. Пріоритети P0–P3: таблиці по чотири рівні. Протокол перегляду на 0.25× швидкості.
- `content/post-1608-pullup-row-technical-limiters-matrix.md` — Cluster 4 post 8/10. Три типи технічних обмежувачів (мобільність / сила / моторний контроль) і матриця корекції для підтягування і горизонтальної тяги.
- `content/post-1609-pullup-progression-zero-to-weighted.md` — Cluster 4 post 9/10. П'ять ступенів прогресії підтягувань: активаційний фундамент → ексцентрична прогресія → перші концентричні → накопичення об'єму → навантажені підтягування. Критерії переходу між ступенями.
- `content/post-1610-pullup-row-cluster-synthesis.md` — Cluster 4 post 10/10 (block synthesis). Три рівні системи self-коучингу для підтягувань і горизонтальної тяги: що визначити один раз, що перевіряти щотренування, що переглядати на мезоциклі. Анонс Cluster 5 (вертикальний жим).

### Changed
- `blog.html` — додано posts 1601–1610 у верхній секції списку.
- `STATUS.md` — Block 1571–1620 Cluster 4 COMPLETE 10/10; наступний пріоритет Cluster 5 (posts 1611–1620).
- `VERSION` — 6.60.1 → 6.61.0.

---

## [6.60.1] — 2026-06-20

### Changed
- `docs/CRYPTOGRAPHY_POLICY.md` — v1.2 → v1.3. Registered `HMAC_KDF_SALT` (§5 Key Rotation + §6 Key Custody) and `ADOPTION_NOTES_SALT` (§5 + §6). Closes `docs/SOC2_READINESS.md §93.5` item 2 (P0/M9 — `HMAC_KDF_SALT` prerequisite for HMAC-VERIFY-ALGO-001 test vector and Admin Dashboard per-tenant key derivation panel), `docs/SOC2_READINESS.md §96.6` item 2 (P0/M10 — `ADOPTION_NOTES_SALT`), and `docs/DATA_MODEL.md §41.9` item 1 (P0/M10 — `ADOPTION_NOTES_SALT` registration obligation).
- `VERSION` — 6.60.0 → 6.60.1.

---

## [6.60.0] — 2026-06-20

### Added
- `content/post-234-plan-deviations-self-sabotage.md` — Editorial series post. «Відхилення від плану: коли це правильне рішення, а коли — самосаботаж». Таксономія трьох типів відхилень (контекстне / авторегулятивне / превентивне). Сім ознак правильного рішення і п'ять ознак самосаботажу. Механізм афективного прогнозування (Wilson & Gilbert 2005) і його вплив на тренувальні рішення. Операційний алгоритм із п'яти запитань. clinical-safety: NOT_REQUIRED.

### Changed
- `blog.html` — картка post-234 (plan-deviations-self-sabotage) додана вище post-233.
- `README.md` — post-234 позначено як `draft · v6.60.0`. Новий блок 241–250 «Рішення без тренера» додано до роадмапу.
- `STATUS.md` — рядок Editorial 211–240 оновлено до Editorial 211–250 (post-234 · 5/10). Версія v6.55.0 → v6.60.0.
- `VERSION` — 6.59.0 → 6.60.0.

---

## [6.59.0] — 2026-06-20

### Added
- `content/post-1591-bench-technique-individual-variation.md` — Block 1571–1620 Cluster 3 post 1/10. Що таке «правильна техніка жиму лежачи» і чому відповідь завжди індивідуальна. Три рівні «правильності» (безпека, адаптаційна ефективність, індивідуальна відповідність). Чому довжина рук, ширина плечей і глибина грудної клітки визначають механіку більше, ніж інструкція. Константи — зап'ястя, ретракція лопаток, контрольований ексцентрик, траєкторія до нижньої третини грудини. clinical-safety: NOT_REQUIRED.
- `content/post-1592-bench-anthropometry-arms-shoulders-chest.md` — Cluster 3 post 2/10. Три анатомічних параметри жиму: wingspan (довжина рук і ROM), bioacromial distance (ширина плечей і оптимальний хват — 1.5×bioacromial як правило + тест перпендикулярного передпліччя), глибина грудної клітки (природне скорочення ROM). Три антропометричних профілі і їх наслідки.
- `content/post-1593-bench-grip-width-emg-mechanics.md` — Cluster 3 post 3/10. EMG Clemons & Aaron (1997) і Barnett et al. (1995): широкий хват — вища активація грудного, вкорочений ROM; вузький — +20–30% трицепс. Навантаження на ротаторну манжету при широкому хваті (Fiebert et al., 1995). Матриця трьох варіантів за метою і умовами.
- `content/post-1594-bench-arch-thoracic-mobility-limits.md` — Cluster 3 post 4/10. Анатомічна дуга (з Th4–Th8, грудний відділ) vs. компенсаторна (поперек). Три функції дуги: скорочення ROM, стабілізація лопатки, зниження навантаження на передню капсулу. Межі: сідниці на лаві, відсутність болю в поперековому відділі, відтворюваність. Foam roller і thoracic extension — підготовча робота.
- `content/post-1595-bench-elbow-angle-shoulder-load.md` — Cluster 3 post 5/10. 90° — максимальна абдукція плеча, ризик субакроміального імпінджменту (Kolber et al., 2017). 45–60° — tucked, вищий внесок трицепса, менше навантаження на манжету. 60–75° — робочий компроміс. Зв'язок ширини хвату і кута ліктів: зміна хвату на 3–5 см автоматично змінює кут.
- `content/post-1596-bench-scapular-retraction-leg-drive.md` — Cluster 3 post 6/10. Ретракція лопаток: функція 1 — стабільна основа плечового суглоба; функція 2 — замок кінетичного ланцюга (ноги → таз → хребет → лопатки → плечі). Типова помилка: ретракція на старті + протракція при виштовхуванні. Leg drive — упор ніг у підлогу, не поштовх тазом. Ефект іррадіації (Siff, 2003).
- `content/post-1597-bench-variations-close-grip-dumbbells-incline.md` — Cluster 3 post 7/10. Жим вузьким хватом: +20–30% трицепс (Barnett et al.). Гантелі: вищий ROM + вища активація грудного (Saeterbakken et al., 2011). Похила лавка 30–45°: ключична голівка грудного. Спадна лавка: стернальна голівка, знижене навантаження на плечо. Матриця варіацій за метою.
- `content/post-1598-bench-video-diagnosis-two-angles.md` — Cluster 3 post 8/10. Бічний ракурс: траєкторія (діагональ, не вертикаль), точка контакту (нижня третина грудини), лікті під грифом, джерело дуги. Фронтальний: симетрія (<5° норма), кут абдукції, трекінг зап'ясток. Коли і з якою вагою знімати. Пріоритети P0–P3 з таблицею.
- `content/post-1599-bench-technical-limiters-correction-matrix.md` — Cluster 3 post 9/10. Три типи: рухливість (проблема без навантаження — foam roller, doorway stretch), сила (проблема >80% — аксесуарна робота: serratus push-up, CG bench, face pull), патерн (розуміння є, відчуття немає — паузовий і темповий жим при 60–70%). Матриця симптом → тип → перший крок.
- `content/post-1600-bench-cluster-synthesis.md` — Cluster 3 post 10/10. Синтез 1591–1599: операційна система self-коучингу. Рівень 1 (один раз): антропометрична карта — довжина рук, bioacromial, глибина грудної клітки, базова ширина хвату, кут ліктів, лопаткова ретракція. Рівень 2 (щомісяця): відеосесія при 75–80%. Рівень 3 (щотижня): замок перед важким сетом, нотатка після. Чотири принципи системи. **Cluster 3 COMPLETE 10/10.**

### Changed
- `blog.html` — posts 1591–1600 blog cards inserted above post-1590. blog.html updated to post-1600.
- `VERSION` — 6.58.1 → 6.59.0.

---

## [6.58.1] — 2026-06-20

### Changed
- `docs/AUDIT_LOG_SCHEMA.md` — v2.21: +7 CAEP / SSF events (`sso.caep_event_received`, `sso.caep_session_revoked`, `sso.caep_user_suspended`, `sso.caep_user_purged`, `sso.caep_stream_error`, `sso.caep_stream_registered` from §23; `sso.caep_reregistration_queued` from §36 / DEC-072). New `### CAEP / SSF events` section with ordering constraint §23.8, 4 SOC 2 evidence artefacts (CC6-E-CAEP-001..004), and CC6.3 auditor narrative. `sso.google_directory_sync_error` controlled vocabulary extended with `'cache_eviction_failed_on_risc_hijacking'` (DEC-072 §36.4). Closes SSO_SCIM §23.13 item 9 (P0/M4) and §36.6 item 4 (P0/M5).
- `docs/SSO_SCIM_IMPLEMENTATION.md` — §23.13 checklist item 9 marked `[x] Done`; §36.6 checklist item 4 marked `[x] Done`.
- `VERSION` — 6.58.0 → 6.58.1.

---

## [6.58.0] — 2026-06-20

### Added
- `content/post-40-evaluate-own-program.md` — «Як оцінити власну програму: 5 ознак, що вона більше не працює для тебе». Editorial series post 40. Три сценарії відмови програми (фізіологічний/структурний/виконавський); п'ять операційних маркерів: нульова прогресія 4+ тижнів (Ralston 2017), RPE-дрейф вгору (Helms 2016), технічна деградація у свіжому стані, структурний конфлікт з розкладом (Peterson 2011 — adherence як найсильніший предиктор), сповільнення відновлення (Halson 2014); три питання до рішення; алгоритм відповіді для кожної ознаки. clinical-safety: NOT_REQUIRED. sports-scientist review required before publish.

### Changed
- `blog.html` — post-40 blog card inserted above post-39.
- `README.md` — post-40 → draft · v6.58.0; Block 1591–1605 «Жим лежачи та варіації» added as proposed (15 topics, Cluster 3).
- `STATUS.md` — editorial series updated to 11–40; post-40 entry prepended.
- `VERSION` — 6.57.0 → 6.58.0.

---

## [6.57.0] — 2026-06-20

### Added
- `content/post-1581-deadlift-individual-variation.md` — Block 1571–1620 Cluster 2 post 1/10. Що таке «правильний» становий підйом і чому відповідь залежить від структури тіла. Три рівні «правильності» (безпека, механічна ефективність, індивідуальна відповідність). Вплив довжини стегна, кута западини і довжини рук. Що є дійсно універсальним (штанга над серединою стопи, нейтральний хребет, вертикальна траєкторія).
- `content/post-1582-deadlift-anthropometry-proportions.md` — Cluster 2 post 2/10. Антропометрія і вибір варіації. Три ключові пропорції: стегно/гомілка (тест і таблиця профілів), довжина рук (тест), анатомія кульшового суглоба (тест Craig). Практичний 4-кроковий протокол вибору між конвенційним і сумо.
- `content/post-1583-deadlift-conventional-vs-sumo.md` — Cluster 2 post 3/10. Конвенційний vs. сумо: EMG Escamilla et al. (2000, 2002) — активація задньої поверхні стегна (+25–43%), квадрицепсів (+20–30%), привідних (+20–40%). Різниця у відстані підйому (−20–30% у сумо), навантаженні на поперек і кульшові суглоби. Матриця вибору за анатомією і метою.
- `content/post-1584-deadlift-variations-rdl-block-deficit.md` — Cluster 2 post 4/10. Варіації: RDL (ексцентрик задньої поверхні, 60–80% від макс.), блок-тяга (lockout, вище навантаження), дефіцитний становий (відрив, +2.5–5 см). Матриця «вузьке місце → варіація».
- `content/post-1585-deadlift-neutral-spine-flexion.md` — Cluster 2 post 5/10. Нейтральний хребет у становому. Flexion-relaxation phenomenon: при повній флексії параспінальні м'язи вимикаються. Різниця між грудним і поперековим відділом під навантаженням. Практичний критерій для кожної фази підйому на базі McGill (2007, 2016) і Cholewicki et al. (2000).
- `content/post-1586-deadlift-bar-position-first-pull.md` — Cluster 2 post 6/10. Стартова позиція і перший пул. Три правила геометрії: штанга над серединою стопи (~2.5 см від гомілки), лопатки над штангою, нейтральний хребет до відриву. Механіка першого і другого пулу. Tension before lift. Таблиця частих помилок.
- `content/post-1587-deadlift-lockout-hyperextension.md` — Cluster 2 post 7/10. Lockout і гіперекстензія. Що таке правильний lockout (тіло вертикальне, сідничні стиснуті). Механічні наслідки гіперекстензії (компресія фасеткових суглобів). Три критерії розрізнення правильної і надмірної позиції. Слабкий розгинач стегна як прихована причина.
- `content/post-1588-deadlift-video-diagnosis.md` — Cluster 2 post 8/10. Відеодіагностика. Протокол двох ракурсів (збоку і спереду/ззаду), чеклист точок контролю для кожної фази, пріоритети P0–P3, практичний протокол зйомки при 80–85%, перегляд 0.25x/0.5x.
- `content/post-1589-deadlift-technical-limiters-matrix.md` — Cluster 2 post 9/10. Технічні обмежувачі. Три типи: рухливість (є без навантаження), сила (лише >80%), патерн (є розуміння але немає відчуття). Матриця аксесуарних вправ для кожного типу і кожного дефекту.
- `content/post-1590-deadlift-cluster-synthesis.md` — Cluster 2 post 10/10. Синтез кластеру 2. Три рівні операційної системи — антропометрична карта (один раз), щомісячна відеосесія, щотижневий технічний фокус. Чотири принципи самокоучингу. Block synthesis.

### Changed
- `blog.html` — posts 1581–1590 додані на початок списку (Technique / deadlift cluster).
- `STATUS.md` — Block 1571–1620 Cluster 2 COMPLETE; next: Cluster 3 (posts 1591–1600 · Жим лежачи та варіації).

---

## [6.56.0] — 2026-06-20

### Added
- `compliance/docs/hmac-chain-verification-algorithm.md` v1.0 — HMAC-VERIFY-ALGO-001: FORM DEC-030 HMAC Chain Verification Algorithm for enterprise SIEM consumers (closes `docs/OBSERVABILITY.md §50.10` items 1 and 2, P0/M9, DEC-071). §2 chain structure (DEC-030 fields, HMAC-SHA256 input format `{event_id}:{event_type}:{canonical_payload}:{created_at_unix_ms}:{previous_signature}`, chain-start sentinel = 64 zero chars, `canonical_payload = json.dumps(sort_keys=True, separators=(",", ":"))`, `created_at_unix_ms` = integer epoch ms). §3 Python 3.10+ verification pseudocode: `verify_chain(events, hmac_secret)` — sorts by `created_at`, accumulates `prev_signature`, detects `SIGNATURE_MISMATCH` and `CHAIN_POINTER_MISMATCH`; helper functions `compute_signature()`, `canonical_payload()`, `created_at_to_ms()`. §4 four implementation notes: per-tenant isolation (one chain per `tenant_id`), cursor continuity (accumulate-then-verify or incremental carry-forward), SIEM delivery gaps vs. chain breaks (AL-SIEM-05 P0 fires on FORM-side break; gap-period events verifiable by timestamp-range pull request), HMAC secret rotation boundary handling (two keys for cross-rotation exports; 30-day advance notice; rotation est. M24). §5 test vector with all computed hex signatures: Sig1 = `6926f92c8a76a5986bf1cac4d2de8ff0056fc6cd49b9212d528a77c2af0d56f5` (event `sso.login_succeeded`, `created_at_ms = 1781863200000`, `previous_signature = 64 zeros`); Sig2 = `af8620250edfd83b994723921973b2e339a9534259b9f7015d2f8ec2567efaaf` (event `auth.session_created`, `created_at_ms = 1781863205000`); Sig3 = `ba05639de41cb1fadecb47931f2b5e2be884848834a0f664c5c6bc35e69a8a0a` (event `sso.login_failed`, `created_at_ms = 1781863260000`); `expected_result = {valid: true, errors: [], events_checked: 3}`. §6 security questionnaire standard response verbatim (per-tenant key via HKDF-SHA256, Admin Dashboard display-once, AL-SIEM-05 P0 monitoring, library on CSM request). §7 privacy floor (key not retained post-display, no PII/Art.9 data in chain artefacts, synthetic test vector). §8 cross-reference table (8 entries: OBSERVABILITY §50, §27.4, §27.7; AUDIT_LOG_SCHEMA DEC-030; CRYPTOGRAPHY_POLICY §5; ENTERPRISE_ONBOARDING §3.5; SECURITY_QUESTIONNAIRE LOG-04; DATA_ROOM §Technical Security; DECISION_LOG DEC-071). Per-tenant key derivation note: `HKDF-SHA256(IKM=FORM_AUDIT_HMAC_SECRET, info=tenant_id, salt=HMAC_KDF_SALT, len=32)`. Owner: security-engineer + devops-lead + compliance-officer.

### Changed
- `docs/DATA_ROOM.md` — §Technical Security section added (v0.1 → v0.2): HMAC-VERIFY-ALGO-001 row with ID, title, file path (`compliance/docs/hmac-chain-verification-algorithm.md`), audience (enterprise security teams, SIEM engineers, SOC 2 auditors), SOC 2 mapping (CC1.1/C1.1), sharing trigger (security questionnaire package + M10 pilot onboarding §3.5). Closes `docs/OBSERVABILITY.md §50.10` item 2 (P0/M9 — DEC-071).
- `docs/OBSERVABILITY.md §50.10` — items 1 and 2 marked `[x] Done` (2026-06-20). Document version v4.7.0 unchanged (patch-level status update).
- `VERSION` — v6.55.0 → v6.56.0.

---

## [6.55.0] — 2026-06-20

### Added
- `content/post-233-program-adaptation-when-it-undermines-progress.md` — Editorial series post 233, block 231–240. «Що таке «адаптація програми» і де вона починає підривати прогрес». Таксономія трьох типів адаптації (ситуативна / структурна / дрейф); когнітивний механізм раціоналізації post-hoc (Teixeira et al. 2012; Crabtree & Holloway 2022); три операційних сигнали дрейфу (обсяг нижче 70%, інтенсивність нижче порогу, принципи замінились преференціями); парадокс авторегуляції vs. дрейфу (Helms et al. 2014; Zourdos et al. 2016; Mattocks et al. 2017); протокол 15-хвилинного аудиту раз на чотири тижні; чому явне переписування програми ефективніше за непомічений дрейф. Посилання на post-232 (принцип vs. параметр). clinical-safety: NOT_REQUIRED. sports-scientist review required before publish.

### Changed
- `blog.html` — картка post-233 додана (перед post-232).
- `STATUS.md` — editorial series оновлено до 211–240, post-233 записано.
- `VERSION` — v6.54.1 → v6.55.0.

---

## [6.54.1] — 2026-06-20

### Added
- `docs/ENTERPRISE_ONBOARDING.md §3.5` — "Audit log — HMAC chain verification" onboarding step (v0.1 → v0.2). Three-step CSM protocol: (1) share HMAC-VERIFY-ALGO-001 Data Room artefact (`compliance/docs/hmac-chain-verification-algorithm.md`) via §1.2 NDA-gated security package; (2) direct customer security team to `tenant_hmac_verify_key` at Admin Dashboard → Security → Audit Export (display-once; FORM does not retain copy); (3) record `[library-request: HMAC-VERIFY-ALGO-001]` tag in `enterprise_contracts.notes` if customer requests verification library. Standard customer Q&A response block. Privacy floor note (HKDF-SHA256 derived key, no Art. 9 data in verification output). Closes `docs/OBSERVABILITY.md §50.10` item 4 (P1/M10 — DEC-071).
- `docs/SECURITY_QUESTIONNAIRE.md LOG-04` — "Can customers independently verify the integrity of their exported audit log events?" (v1.0 → v1.1). Full HMAC-VERIFY-ALGO-001 response: algorithm specification reference (`docs/OBSERVABILITY.md §50`), Python pseudocode with `verify_chain()`, test vector, per-tenant key (`HKDF-SHA256(IKM=FORM_AUDIT_HMAC_SECRET, info=tenant_id, …)` at Admin Dashboard), FORM-side AL-SIEM-05 P0 chain-break monitoring, library availability on CSM request. §50.8 standard security questionnaire response language included verbatim. SOC 2 mapping CC1.1/C1.1. Closes `docs/OBSERVABILITY.md §50.10` item 5 (P1/M10 — DEC-071). Last reviewed updated 2026-06-06 → 2026-06-20.

### Changed
- `docs/OBSERVABILITY.md §50.10` — items 4 and 5 marked `[x] Done` (2026-06-20). Document version v4.7.0 unchanged (patch-level status update).
- `VERSION` — v6.54.0 → v6.54.1.

---

## [6.54.0] — 2026-06-20

### Added
- `content/post-39-training-partner.md` — editorial № 39: «Тренування з партнером: де соціальна фасилітація підсилює прогрес, де вона його блокує». Zajonc (1965) social facilitation: arousal підсилює домінантні реакції і заважає новим патернам; Kerrigan et al. (2017) meta-analysis adherence; Hausenblas et al. (2001) social presence і зусилля; Baron (1986) distraction-conflict theory; Latané et al. (1979) social loafing; Carron et al. (1996) group cohesion; Feltz et al. (2011) Köhler effect; три типи партнерства (accountability / parallel / shared program); два операційних питання; три правила протоколу. clinical-safety: NOT_REQUIRED. sports-scientist review required before publish.

### Changed
- `blog.html` — картка post-39 додана (перед post-38).
- `STATUS.md` — editorial series оновлено до 11–39, 44–50.
- `VERSION` — v6.53.0 → v6.54.0.

---

## [6.53.0] — 2026-06-20

### Added
- `content/post-1571-squat-technique-individual-variation.md` — Block 1571–1620 Cluster 1 post 1/10. Opener: три рівні «правильності» техніки (безпека, адаптаційна ефективність, індивідуальна відповідність); чому антропометрія визначає механіку більше ніж тренер; константи незалежно від будови тіла (нейтральний хребет, колінно-стопна відповідність, контроль ROM). Типові помилки тренера і атлета-аматора. clinical-safety: NOT_REQUIRED. sports-scientist review required before publish.
- `content/post-1572-squat-anthropometry-proportions.md` — Cluster 1 post 2/10. Антропометрія і присідання: три ключових пропорції (стегно/гомілка, тулуб/ноги, дорсифлексія); тест knee-to-wall, тест Faber/Craig; практичні висновки для чотирьох антропометричних профілів. clinical-safety: NOT_REQUIRED. sports-scientist review required.
- `content/post-1573-squat-highbar-vs-lowbar.md` — Cluster 1 post 3/10. Хай-бар vs. лоу-бар: механічні відмінності, порівняльна таблиця EMG (Escamilla 2001), навантаження на суглоби, антропометричні сценарії вибору. Різниця в абсолютній силі ~8–10% (Glassbrook 2019). clinical-safety: NOT_REQUIRED. sports-scientist review required.
- `content/post-1574-squat-stance-width-toe-angle.md` — Cluster 1 post 4/10. Ширина стійки і кут носків: тест Fan, п'ятикроковий протокол налаштування, типові помилки («вузько», «носки вперед»). Кісткова структура вертлюжної западини і антеверсія/ретроверсія визначають природній кут. clinical-safety: NOT_REQUIRED. sports-scientist review required.
- `content/post-1575-squat-depth-mobility-anatomy-goals.md` — Cluster 1 post 5/10. Глибина присідання: чотири рівні ROM, порівняльна EMG (Caterisano 2002), навантаження на ПС і ПКЗ, структурні vs. функціональні обмежувачі, butt wink причини і рішення, таблиця рекомендацій за контекстом. clinical-safety: NOT_REQUIRED. sports-scientist review required.
- `content/post-1576-squat-forward-lean-causes-norms.md` — Cluster 1 post 6/10. Нахил тулуба вперед: п'ять причин (антропометрія, дорсифлексія, кор, квадрицепс, позиція штанги) і діагностична матриця для розрізнення. Протокол корекції для кожної причини. Нормальний діапазон нахилу за Escamilla (2001). clinical-safety: NOT_REQUIRED. sports-scientist review required.
- `content/post-1577-squat-knee-valgus-research.md` — Cluster 1 post 7/10. Вальгус коліна: різниця між компенсаторним і колапсуючим вальгусом; п'ять причин; критичний розбір Fry et al. (2003) і правила «коліна за носком»; діагностичний протокол для self-coached атлета (bodyweight vs. важкі ваги). clinical-safety: NOT_REQUIRED. sports-scientist review required.
- `content/post-1578-squat-self-diagnosis-video.md` — Cluster 1 post 8/10. Відеодіагностика присідання: два обов'язкових ракурси, протоколи читання сагітальної і фронтальної площини, коли відзнімати, 15-хвилинний протокол аналізу, confirmation bias (Miall & Wolpert 1996). clinical-safety: NOT_REQUIRED. sports-scientist review required.
- `content/post-1579-squat-technical-limiters-strength-mobility-pattern.md` — Cluster 1 post 9/10. Технічні обмежувачі: три типи (мобільний, силовий, патерновий), діагностична матриця з п'яти критеріїв, комбіновані обмежувачі і порядок корекції. clinical-safety: NOT_REQUIRED. sports-scientist review required.
- `content/post-1580-squat-cluster-synthesis.md` — Cluster 1 post 10/10. Синтез операційна система самокоучингу: 4 рівні (антропометрична карта один раз → відеодіагностика раз на 4–6 тижнів → матриця обмежувачів при регресії → один фокус на сесію). Checklist для двох ракурсів. Анонс Cluster 2 (становий підйом). Block 1571–1620 Cluster 1 COMPLETE 10/10. clinical-safety: NOT_REQUIRED. sports-scientist review required.
- `docs/DECISION_LOG.md` — DEC-075 (Block 1571–1620 тема: Техніка великих рухів для self-coached атлета, 5 кластерів × 10 постів) + DEC-074 (нумерація: пояснення пропуску 1501–1555, зарезервовані для заплановних але ненаписаних блоків). Append-only.

### Changed
- `blog.html` — десять нових карток (posts 1571–1580, тег «Technique») додані на початок списку. blog.html updated to post-1580.
- `STATUS.md` — Block 1571–1620 Cluster 1 COMPLETE 10/10; next: Cluster 2 (1581–1590, становий підйом та варіації).
- `VERSION` — v6.52.1 → v6.53.0.

---

## [6.52.1] — 2026-06-19

### Changed
- `docs/SOC2_READINESS.md` — §96 Cross-Reference Patch: DATA_MODEL §41 (enterprise_adoption_snapshots · §90 DDL Cross-Reference Update · CC4.1 / CC2.2 / CC7.3 / A1.1 / CC4.2). Closes the implicit pattern obligation established by §89–§95: bidirectional cross-reference between DATA_MODEL §41 (canonical DDL source for `enterprise_adoption_snapshots`, v1.20 2026-06-19) and SOC2_READINESS §90 (ADO-E-001/002/003 evidence artefacts). §90 (v3.15.0) pre-dates §41 and cited DATA_MODEL §17 as k-anonymity pattern precedent; §96 registers §41 as the specific DDL reference and supplements the §90 privacy floor record with two DDL-level invariants: (1) `tenant_manager` RLS exclusion — HR role explicitly barred from all four `enterprise_adoption_snapshots` RLS policies, preventing adoption health from being captured as an employee disciplinary control (wellness-as-punishment no-go alignment); (2) `notes_hash` SHA-256 + ADOPTION_NOTES_SALT — plaintext never stored, artefact rows contain no recoverable CSM notes. §96.4 maps five DDL invariants to TSC criteria reinforcement: CC4.1 (GENERATED ALWAYS AS STORED band + UNIQUE dedup), CC2.2 (z.literal(true) write-time gate vs. post-export assert), CC7.3 (system.csm_followup_overdue in chain makes missed responses detectable), A1.1 (adoption_health_band ENUM churn-probability table is DDL definition of ARR-at-band), CC4.2 (tenant_manager exclusion confirms IT/People Ops ownership of deficiency evaluation). Four-item P0–P1 implementation checklist: migration 0078 ON DELETE RESTRICT staging validation, ADOPTION_NOTES_SALT Cryptography Policy registration, §80.3 R2 adoption/ subfolder cross-tracking, §41.9 item 5 / §90.6 item 3 parallel closure at first ADO-E-001 filing. SOC 2 internal version v3.20.0 → v3.21.0. Owner: compliance-officer + enterprise-architect + platform-engineer.
- `VERSION` — v6.52.0 → v6.52.1.

---

## [6.52.0] — 2026-06-19

### Added
- `content/post-232-how-to-read-a-program-principle-vs-parameter.md` — Editorial series post 232, block 231–240. «Як читати програму — і відрізнити принцип від параметра». Розмежування двох шарів будь-якої програми: принципи (прогресивне перевантаження, специфічність, відновлення, механічна напруга) та параметри (дні тижня, конкретні вправи в межах патерну, точний час відпочинку, розбиття обсягу між сесіями). Три питання для кожного елементу програми. Найчастіші помилки при адаптації. Практичний алгоритм читання будь-якої нової програми. Rhea et al. 2003; Krieger 2010; Schoenfeld 2010; Helms et al. 2014; Grgic et al. 2018. clinical-safety: NOT_REQUIRED. sports-scientist review required.

### Changed
- `blog.html` — картка post-232 додана вище post-231 (Training Science · 2026-06-19).
- `STATUS.md` — рядок «Editorial 211–231» оновлено до «Editorial 211–232»: post-232 додано v6.52.0.
- `VERSION` — v6.51.0 → v6.52.0.

---

## [6.51.0] — 2026-06-19

### Added
- `content/post-1566-training-stagnation-exit-strategy.md` — Block 1556–1570 Cluster 3, post 1/5. Вихід з тренувальної стагнації: чотири важелі (об'єм, інтенсивність, частота, варіація), принцип «один важіль за раз», таймлайн 4-тижневої перевірки, мезоциклічна реструктуризація коли окремий важіль не достатній. clinical-safety: NOT_REQUIRED. sports-scientist review required.
- `content/post-1567-recovery-stagnation-exit-strategy.md` — Block 1556–1570 Cluster 3, post 2/5. Вихід з відновлювальної стагнації: деловантаж-протокол (-40–60% об'єму, збережена інтенсивність, 1–2 тижні), аудит факторів відновлення (сон, зовнішній стрес, тривалий дефіцит), протокол поступового повернення. clinical-safety: NOT_REQUIRED. sports-scientist review required.
- `content/post-1568-technical-stagnation-exit-strategy.md` — Block 1556–1570 Cluster 3, post 3/5. Вихід з технічної стагнації: локалізація обмежувача через відеоаналіз, три типи (нейром'язова неефективність / мобільнісний дефіцит / м'язовий дисбаланс), корекційний блок (-10–20% навантаження, допоміжна робота, 6–8 тижнів). clinical-safety: NOT_REQUIRED. sports-scientist review required.
- `content/post-1569-self-coached-protocol-limits.md` — Block 1556–1570 Cluster 3, post 4/5. Межі self-coached протоколу: три граничних сигнали (тривалість 4+ місяці без відповіді, суперечливі діагнози, технічні обмежувачі поза відеоаналізом). Визначення «зовнішня допомога» як цільова консультація, а не постійний тренер. clinical-safety: NOT_REQUIRED. sports-scientist review required.
- `content/post-1570-block-synthesis-progress-evaluation.md` — Block 1556–1570 Cluster 3 synthesis + block synthesis. Повна операційна система трьох рівнів: рівень 1 (тижневий огляд, щотижня), рівень 2 (діагностика плато, аномальний стан 3+ тижні), рівень 3А/Б/В (стратегія виходу за типом). Швидкий довідник в таблиці. BLOCK COMPLETE 15/15. clinical-safety: NOT_REQUIRED. sports-scientist review required.

### Changed
- `blog.html` — додані картки posts 1566–1570 вище post-1565. blog.html updated to post-1570.
- `VERSION` → 6.51.0

---

## [6.50.0] — 2026-06-19

### Added
- `content/post-231-programs-for-athletes-that-dont-exist.md` — Editorial series post 231, block 231–240 opener. «Чому більшість тренувальних програм написано для атлета, якого не існує». П'ять прихованих припущень у кожній програмі (стабільний сон, однорідне відновлення, психологічна нейтральність, стабільний мікроцикл, повний adherence) і чому вони рідко відповідають дійсності. Ключовий аргумент: різниця між принципом (прогресивне перевантаження, специфічність, відновлення) і параметром (конкретна вага, кількість підходів, частота). Чотири практичні сценарії адаптації без порушення принципу. Blumert et al. 2007; Kraemer & Ratamess 2004; Kiely 2012; Meeusen et al. 2013; Halson 2014. clinical-safety: NOT_REQUIRED. sports-scientist review required.

### Changed
- `blog.html` — картка post-231 додана вище post-211 (Training Science · 2026-06-19).
- `STATUS.md` — рядок «Editorial 211» розширено до «Editorial 211–231»: post-231 додано v6.50.0; README roadmap updated posts 231–240.
- `README.md` — roadmap: posts 231–240 proposed (block theme: «Self-coached атлет як перекладач принципів у контекст»). Post-231 позначено draft · v6.50.0.
- `VERSION` — v6.49.0 → v6.50.0.

---

## [6.49.0] — 2026-06-19

### Added
- `content/post-1561-plateau-vs-noise-definition.md` — Block 1556–1570, Cluster 2 (post 1/5). «Плато vs. шум: як визначити справжню стагнацію через дані». Визначення плато через три ознаки (e1RM нейтральна зона, RPE-тренд без покращення, відсутня технічна деградація) за три+ тижні після виключення конфаундерів. Диференціація нейтральної зони від справжнього плато: фаза акумуляції, передпікова стагнація, підтримка. Таймлайн «коли починати хвилюватись» (1–2 тижні: ігноруй; 3 тижні: виключи конфаундери; 4+ тижні: дій). Чотири найпоширеніші помилки ідентифікації. Zourdos et al. 2016. clinical-safety: NOT_REQUIRED. sports-scientist review required.
- `content/post-1562-noise-sources-training-data.md` — Block 1556–1570, Cluster 2 (post 2/5). «Шум у тренувальних даних: джерела, масштаб і фільтрація». Сім джерел шуму з масштабом ефекту: сон (−3–6% e1RM за ніч: Reilly & Piercy 1994, Blumert et al. 2007), гідратація (−2–5%: Judelson et al. 2007), час доби (до 8%: Atkinson & Reilly 1996), психологічний стрес (до 10%: Stults-Kolehmainen & Sinha 2014), варіабельність AMRAP-протоколу, температура, харчування до тренування. Зведена таблиця очікуваної тижневої варіабельності e1RM за рівнем атлета (початківець ±3–5%, середній ±2–4%, просунутий ±1–3%). Контрольний список семи конфаундерів перед діагнозом «плато». Метод ковзного середнього (3 тижні). П'ять мінімальних стандартів вимірювання. clinical-safety: NOT_REQUIRED.
- `content/post-1563-three-types-stagnation.md` — Block 1556–1570, Cluster 2 (post 3/5). «Три типи стагнації: тренувальна, відновлювальна, технічна». Тренувальна: RPE знижується, самопочуття норма, програма >10 тижнів без змін — причини: застаріла програма, недостатній об'єм, неправильний тип прогресії, відсутність специфічності. Відновлювальна: RPE зростає, загальна енергія знижена, сон/стрес/дефіцит — механізм хронічного перевантаження без суперкомпенсації (Schoenfeld et al. 2017). Технічна: дрейф у першому повторенні, специфічна слабкість або ненасштабований моторний патерн. Таблиця диференційної діагностики. Дві найпоширеніші комбіновані стагнації. clinical-safety: NOT_REQUIRED.
- `content/post-1564-plateau-diagnostic-protocol.md` — Block 1556–1570, Cluster 2 (post 4/5). «Протокол діагностики плато: 2-тижневий алгоритм». Умова запуску: e1RM нейтрально 3+ тижні + конфаундери виключені. Тиждень 1 — чотири параметри: щоденний суб'єктивний моніторинг (сон/енергія/тонус 1–5), RPE-аудит по ключових рухах, технічний аудит (відеозйомка першого і останнього повторення), контекстний аудит (програма, калорії, позатренувальне навантаження). Тиждень 2 — тест-втручання: тренувальна гіпотеза (+5% або +1 підхід), відновлювальна (−35% об'єму + сон), технічна (−15% навантаження + технічний фокус). Зведена таблиця «сигнали → гіпотеза → тест → висновок». Алгоритм при змішаних сигналах. Три ситуації поза межами self-coached протоколу. clinical-safety: NOT_REQUIRED.
- `content/post-1565-plateau-noise-decision-matrix.md` — Block 1556–1570, Cluster 2 synthesis (post 5/5). «Синтез кластеру 2: матриця рішень «плато vs. шум»». Трирівнева матриця: Рівень 1 — визначення (шум vs. потенційне плато), Рівень 2 — тип стагнації (чотири ключові сигнали → чотири реакції), Рівень 3 — перевірочний тест. Швидкий алгоритм трьох питань (три хвилини). Таймлайн дій за тривалістю стагнації. Чотири найчастіші помилки при реакції на плато. Інтеграція з тижневим оглядом (кластер 1): стандартний моніторинг vs. аномальний стан. **Cluster 2 COMPLETE (posts 1561–1565 · Плато vs. шум — діагностика справжньої стагнації).** clinical-safety: NOT_REQUIRED. sports-scientist review required.

### Changed
- `blog.html` — картки post-1561 через post-1565 додані вище post-1560 (Progress Evaluation · 2026-06-19).
- `STATUS.md` — Block 1556–1570 оновлено: 5/15 → 10/15; Cluster 2 COMPLETE; next = Cluster 3 (тема TBD).
- `VERSION` — v6.48.1 → v6.49.0.

---

## [6.48.1] — 2026-06-19

### Added
- `docs/SOC2_READINESS.md §95` — Evidence Artefact Cross-Reference Patch: OBSERVABILITY §49 (DEC-070 · OQ-SSO-OBS-01 · CC7.2 / CC7.3). Registers SSO-FLEET-E-001 — quarterly export of AL-SSO-FLEET-01 PagerDuty incidents cross-referenced with pg_cron job 36 (`sso_fleet_health_check`) run-history; 3yr retention; `compliance/evidence/sso/SSO-FLEET-E-001_<YYYY-QN>.csv`; first filing M11. Closes OBSERVABILITY §49.9 item 4 (P1/M11). Three-part collection methodology: Part A (job 36 full run-history + freshness gap cross-check — LAG window, zero gaps expected; `system.fleet_sso_check_stale` advisory cross-check on non-zero gaps); Part B (`siem.sso_fleet_health_breach` DEC-030 HIGH/3yr export — aggregate-only payload; `sla_credit_impact: 'none'` hard DEC-070 invariant asserted on every row); Part C (manual PagerDuty AL-SSO-FLEET-01 incident export; each incident cross-referenced with Part B DEC-030 event within ±5 minutes). Zero-incident attestation JSON template for pre-M10 quarters (fields: artefact, quarter, query_run_at, incident count, breach event count, job36 gap count, monitoring_pipeline_reference, attestation text). Two-row CC7.2/CC7.3 criteria mapping with DEC-070 signal-source coherence rationale. Cross-reference closure table closes §49.9 item 4 (🟢) and pattern obligation (🟢); §49.9 item 5 §26.8 alert table addition tracked (🟡 Open P1/M5, devops-lead). Seven-item implementation checklist (3× P0/M5, 4× P1/M11). §80.4 Vanta mirror list updated: SSO-FLEET-E-001 added; backfills GUARD-E-001 (§91), SSO-OBS-E-007 (§92), HMAC-VERIFY-ALGO-001 (§93), CAEP-E-001 (§94) which were declared added in prior version notes but not applied to the §80.4 text. SOC2_READINESS v3.19.0 → v3.20.0.

### Changed
- `VERSION` — v6.48.0 → v6.48.1.

---

## [6.48.0] — 2026-06-19

### Added
- `content/post-211-think-in-systems.md` — Editorial series post 211. «Як думати системами, а не тренуваннями: чому підхід «сесія за сесією» обмежує розвиток». Чотири часові горизонти тренування (сесія / мікроцикл / мезоцикл / макроцикл) і чому більшість тренувальних рішень приймається на неправильному рівні. Ключові концепції: відстрочена трансформація (суперкомпенсація видна тільки після деловантажу, не в середині блоку); RPE-дрейф як сигнал блоку а не сесії; технічний дрейф під навантаженням видно тільки в ретроспективі; патерн реакції на деловантаж як діагностичний індикатор. Три практики переходу на системне мислення: відстрочений аналіз (мінімум 2 тижні тренду для структурних рішень); блочна ретроспектива (5 питань після кожного мезоциклу); ієрархія рішень по рівнях (сесія/тиждень/блок/рік). Hubal et al. 2005; Bompa & Haff 2009; Meeusen et al. 2013; Halson 2014; Zatsiorsky & Kraemer 2006; Stone, Stone & Sands 2007. clinical-safety: NOT_REQUIRED. sports-scientist review required before publish.

### Changed
- `blog.html` — post-211 card added above post-190 (Training Science · 13 хв · 2026-06-19).
- `README.md` — post-211 status updated: proposed → draft · v6.48.0. Posts 221–230 added to editorial roadmap: пасивне vs. активне відновлення (221); нечітка «середина» інтенсивності (222); дисципліна як навик (223); «я втомився» — дані чи виправдання (224); «читати тіло» як вміння (225); базовий словник спортивної науки (226); тренувальна інерція (227); мотивація vs. структура (228); «невдала» сесія без самосаботажу (229); синтез 221–230 (230).
- `STATUS.md` — Editorial 211 row added; version bump v6.45.1 → v6.48.0.
- `VERSION` — v6.47.0 → v6.48.0.

---

## [6.47.0] — 2026-06-19

### Added
- `content/post-1560-weekly-progress-review-integration-protocol.md` — Block 1556–1570, Cluster 1 synthesis (post 5/5). «Тижневий огляд прогресу: як інтегрувати e1RM, RPE-тренд і відеоаналіз в одну систему». Протокол 20 хв/тиждень для self-coached атлета. Три шари адаптації з різними часовими горизонтами: силова (e1RM, 3–4 тижні), нейром'язова (RPE-тренд, 1–2 тижні), технічна (відеоаналіз, 2–6 тижнів). Крок 1 — e1RM вектор (дельта за 4 тижні, три зони: прогрес/нейтраль/регресія); Крок 2 — RPE-тренд (±0.5 threshold, матриця 4 узгоджень з e1RM); Крок 3 — відеоаналіз (дрейф першого vs. останнього повторення, 3 сценарії і дії); Крок 4 — інтегральна матриця 6 сценаріїв (e1RM↑/нейтр/↓ × RPE↓/↑ × техніка стабільна/дрейф → конкретна дія). Шаблон журнального запису. 4 типові помилки. Алгоритм дій при конфліктних сигналах. clinical-safety: NOT_REQUIRED. sports-scientist review required before publish. **Cluster 1 COMPLETE (posts 1556–1560).**

### Changed
- `blog.html` — post-1560 card added at top (Progress Evaluation · 14 хв · 2026-06-19).
- `STATUS.md` — Block 1556–1570 updated: 4/15 → 5/15; Cluster 1 marked COMPLETE; next = Cluster 2 (плато vs. шум).
- `VERSION` — v6.46.0 → v6.47.0.

---

## [6.46.0] — 2026-06-19

### Added
- `docs/SOC2_READINESS.md §94` — Evidence Artefact Cross-Reference Patch: SSO_SCIM §36 (DEC-072 · OQ-SSO-23.1/23.3/23.4 · CC6.3 / A1.1 / CC7.4). Registers CAEP-E-001 — two-part quarterly export covering (1) CAEP cert-rotation re-registration event pair (`sso.caep_reregistration_queued` → `sso.caep_stream_registered`) as CC6.3/A1.1 evidence; (2) RISC hijacking group cache eviction zero-tolerance check (`sso.google_directory_sync_error` `error_type: 'cache_eviction_failed_on_risc_hijacking'`) as CC7.4 evidence. Includes collection SQL with three-step Part A and complementary Part B queries, zero-event attestation JSON templates, three-criterion SOC 2 mapping (CC6.3: automated re-registration pipeline closes post-cert-rotation CAEP blind spot; A1.1: pg_cron job 37 as SSO integrity monitoring infrastructure; CC7.4: RISC hijacking → stale group-membership eviction closes 24-hour auth-state retention window), and nine-item implementation checklist (5× P0/M5, 4× P1/M10). Closes SSO_SCIM §36.6 item 1 (OBSERVABILITY §12.6 job 37 registration 🟢) and the pattern obligation from §85/§91. §80.3 `caep/` subfolder added; §80.4 Vanta mirror updated. SOC2_READINESS v3.18.0 → v3.19.0.

### Changed
- `VERSION` — v6.45.1 → v6.46.0.

---

## [6.45.1] — 2026-06-19

### Added
- `content/post-1559-video-analysis-without-coach-protocol.md` — Block 1556–1570 post 4/15. Відеоаналіз без тренера: операційний протокол. Два обов'язкові ракурси (сагітальний і фронтальний) і коли потрібен третій. Маркери по рухах (присідання / станова тяга / жим лежачи / тяга в нахилі) у вигляді таблиць фаза–ракурс–маркер. Дрейф техніки між першим і останнім повторенням як прогностичний сигнал технічної стійкості. Мінімальний журнал відеоаналізу (6 рядків). Частота зйомки за контекстом. Confirmation bias і два практичних рішення (Miall & Wolpert 1996). Три ситуації де відео недостатнє. Інтеграція з e1RM і RPE-тренд. 5-тижневий онбординг-протокол. clinical-safety: NOT_REQUIRED.
- `blog.html` — картка post-1559 додана першою у grid.

### Changed
- `STATUS.md` — Block 1556–1570 → IN PROGRESS 4/15 · v6.45.1; editorial series рядок оновлено до 1556–1559. Version: v6.45.0 → v6.45.1.

---

## [6.45.0] — 2026-06-19

### Added
- `content/post-1491-autoregulation-long-term-programming.md` — Block 1451–1500 Cluster 5 post 1/10. Авторегуляція в довгостроковому програмуванні: RPE vs. фіксований % 1RM, де RPE потрібний (топ-сети, накопичення) і де ні (фаза піку, деавантаж, перший тиждень блоку). Три помилки: авторегуляція вниз як уникнення, відсутність калібрування, зміна фази під виглядом авторегуляції. Практична схема RPE по фазах мезоциклу. clinical-safety: NOT_REQUIRED.
- `content/post-1492-undulating-periodization-dup.md` — Block 1451–1500 Cluster 5 post 2/10. DUP і WUP: Rhea et al. (2002), Zourdos et al. (2016). Механізм: паралельні адаптації різних діапазонів повторень. DUP (3+ сесії/тиждень) vs. WUP (2–3 сесії/тиждень). Три помилки впровадження. Сумісність DUP і блокової переодизації. clinical-safety: NOT_REQUIRED.
- `content/post-1493-when-to-change-program.md` — Block 1451–1500 Cluster 5 post 3/10. Чотири обґрунтовані причини змінити програму (заплановане завершення, технічне стелово, стагнація 3+ тижні, зміна цілі) vs. чотири хибні (нудно, «не відчуваю прогрес», нова методика, дискомфорт). Мінімальний горизонт оцінки — 6–8 тижнів. Рамка трьох питань. clinical-safety: NOT_REQUIRED.
- `content/post-1494-block-transition-protocol.md` — Block 1451–1500 Cluster 5 post 4/10. Горизонти розпаду адаптацій (нейральні 2–4 тижні, гіпертрофія >3 тижнів, кардіореспіраторна найшвидше; Ogasawara et al. 2011). Деактивація, трансфер, «міст». Деавантаж-відновлення vs. деавантаж-перехід. Пастка першого тижня нового блоку. clinical-safety: NOT_REQUIRED.
- `content/post-1495-micro-meso-macrocycle-architecture.md` — Block 1451–1500 Cluster 5 post 5/10. Три рівні планування: розподіл (мікро), акцент (мезо), траєкторія (макро). Типова тривалість мезоциклів за типом, ефективна кількість тренувальних тижнів у рік (~28–36), покрокова схема побудови «з нуля», помилки на кожному рівні. clinical-safety: NOT_REQUIRED.
- `content/post-1496-five-programming-mistakes.md` — Block 1451–1500 Cluster 5 post 6/10. П'ять помилок довгострокового програмування: надмірний об'єм, відсутність фазування, часта зміна програм, ігнорування деавантажу, уникнення слабких сторін. Механізм, діагностика за журналом, вирішення для кожної. clinical-safety: NOT_REQUIRED.
- `content/post-1497-reading-program-between-blocks.md` — Block 1451–1500 Cluster 5 post 7/10. П'ять метрик міжблокового аналізу: e1RM, RPE при фіксованому навантаженні, тоннаж, стабільність техніки, RPE-тренд по тижнях. Правильна методологія порівняння блоків. Шаблон 15-хвилинної ретроспективи. clinical-safety: NOT_REQUIRED.
- `content/post-1498-from-template-to-personalized-program.md` — Block 1451–1500 Cluster 5 post 8/10. Що в шаблоні не міняти (логіка прогресії, частота, деавантаж) vs. що можна (вправи, розподіл, допоміжні). П'ятикроковий цикл адаптації. Межа персоналізації (>50% змін = нова програма). clinical-safety: NOT_REQUIRED.
- `content/post-1499-programming-without-coach.md` — Block 1451–1500 Cluster 5 post 9/10. П'ять джерел зворотного зв'язку для self-coached атлета: журнал, відеоаналіз, пір-рев'ю, разові консультації, структуровані самооцінки. Пастка онлайн-контенту як псевдо-зворотного зв'язку. clinical-safety: NOT_REQUIRED.
- `content/post-1500-block-synthesis-periodization-programming.md` — **Block 1451–1500 SYNTHESIS** (post 10/10 · Cluster 5 post 10/10). Операційна рамка self-coached атлета: алгоритм від рівня макроциклу до тижневих 5-хвилинних ретроспектив. Синтез п'яти кластерів. Три принципи: специфічність, прогресивне перевантаження, відновлення як частина плану. **BLOCK COMPLETE 50/50.** clinical-safety: NOT_REQUIRED.
- `content/post-1558-rpe-trend-objective-readiness-indicator.md` — Block 1556–1570 post 3/15. RPE-тренд: чому серія вимірів у стандартних умовах об'єктивна, попри суб'єктивність одиничного RPE. Стандартизація умов виміру. Три типи тренду (знижувальний / плоский / зростаючий) і що кожен означає. Ранній детектор накопиченої нейром'язової втоми. Порівняння з e1RM. clinical-safety: NOT_REQUIRED.
- `blog.html` — posts 1491–1500 і 1558 додані.

### Changed
- `STATUS.md` — Block 1451–1500 → **COMPLETE 50/50 · v6.45.0**; Block 1556–1570 → IN PROGRESS 3/15.
- `VERSION` — 6.44.0 → 6.45.0.

---

## [6.44.0] — 2026-06-19

### Added
- `docs/SOC2_READINESS.md §93` — Evidence Artefact Cross-Reference Patch for OBSERVABILITY §50 (HMAC Chain Verification Algorithm · DEC-071 · OQ-SIEM-03 · CC1.1 / C1.1 / CC7.4). Closes the gap where OBSERVABILITY §50.8 (v4.7.0) assigned CC1.1/C1.1 SOC 2 criteria to artefact HMAC-VERIFY-ALGO-001 without any formal registration in SOC2_READINESS. §93.1: evidence artefact registration (documentation artefact, one-time authoring, permanent retention, storage `compliance/docs/hmac-chain-verification-algorithm.md`, Vanta mirror via SHA-256 hash). §93.2: three-audience verification protocol — Audience A (7-step SOC 2 auditor independent `verify_chain()` run on 90-day SIEM export + AL-SIEM-05 cross-check), Audience B (3-step enterprise customer: obtain per-tenant key from Admin Dashboard → run `verify_chain()` on 7-day SIEM export → confirm zero errors), Audience C (4-step FORM security-engineer on DEC-030 schema change: identify field change → recompute test vectors → bump API header version → notify tenant SIEM integrators). §93.3: three-criterion SOC 2 mapping — CC1.1 (artefact existence as independently-testable commitment to control environment integrity), C1.1 (test vector proving `SIGNATURE_MISMATCH` + `CHAIN_POINTER_MISMATCH` propagation for confidentiality tamper-detection), CC7.4 (Audience A independent auditor evaluation without FORM operational involvement). §93.4: cross-reference obligations — OBSERVABILITY §50.8 🟢 Closed; §50.10 item 2 (DATA_ROOM exhibit entry) 🟡 Open P1/M10. §93.5: 8-item implementation checklist (3×P0/M9–M10, 5×P1/M10). SOC2_READINESS header v3.15.0 → v3.18.0.

### Changed
- `docs/SOC2_READINESS.md` — header v3.15.0 → v3.18.0; §93 appended.
- `VERSION` — 6.43.0 → 6.44.0.

---

## [6.43.0] — 2026-06-19

### Added
- `content/post-1481-training-age-as-programming-variable.md` — Block 1451–1500 Cluster 4 post 1/10. Тренувальний вік як змінна програмування: хронологічний vs. ефективний тренувальний вік, поріг адаптаційного стимулу, відновлювальна ємність, чому лінійна прогресія руйнується (Kraemer & Ratamess 2004, Fleck & Kraemer 2004, Stone et al. 2007). clinical-safety: NOT_REQUIRED.
- `content/post-1482-novice-intermediate-advanced-adaptation-differences.md` — Block 1451–1500 Cluster 4 post 2/10. Нейральна фаза → структурна фаза (Moritani & deVries 1979, Häkkinen et al. 1985, 1988, Enoka 1988). Якісні відмінності між тренажними рівнями і їх програмні наслідки. clinical-safety: NOT_REQUIRED.
- `content/post-1483-individual-variability-hypertrophy-response.md` — Block 1451–1500 Cluster 4 post 3/10. Hubal et al. 2005 (n=585, 5–59% CSA gain). ACTN3, IGF-1, андрогенні рецептори, міостатин, сателітні клітини (Bamman et al. 2007). N-of-1 підхід для калібрування власної кривої доза-відповідь. clinical-safety: NOT_REQUIRED.
- `content/post-1484-individual-variability-strength-response.md` — Block 1451–1500 Cluster 4 post 4/10. Склад м'язових волокон, жорсткість сухожиль (Magnusson et al. 2008), нейронний стеля (Häkkinen et al. 2003), ричажна механіка. Hubal et al. 2005 силова варіантність 0–250%. Практична рамка: власні дані понад популяційні норми. clinical-safety: NOT_REQUIRED.
- `content/post-1485-recovery-integration-in-programming.md` — Block 1451–1500 Cluster 4 post 5/10. Реактивне vs. структурне відновлення. Суперкомпенсаційна модель (Selye 1950, Zatsiorsky & Kraemer 2006). Типи deload: об'ємний, інтенсивний, частотний. Блокова переодизація (Issurin 2008) як вбудована структура відновлення. clinical-safety: NOT_REQUIRED.
- `content/post-1486-sleep-and-training-adaptation.md` — Block 1451–1500 Cluster 4 post 6/10. GH під час SWS (Van Cauter et al. 2000), T:C-ratio при депривації (Leproult & Van Cauter 2011), ризик травми (Milewski et al. 2014 — 1.7× при <8h), Mah et al. 2011 sleep extension. Регулярність > тривалість. clinical-safety: NOT_REQUIRED (performance framing only).
- `content/post-1487-load-management-acwr.md` — Block 1451–1500 Cluster 4 post 7/10. ACWR (Gabbett 2016), Foster session RPE (Foster 1998), монотонія і Strain (Foster et al. 2001), суб'єктивна готовність (Hooper et al. 1995). Мінімальна система моніторингу для self-coached атлета. clinical-safety: NOT_REQUIRED.
- `content/post-1488-deload-supercompensation-timing.md` — Block 1451–1500 Cluster 4 post 8/10. Суперкомпенсаційна модель (Яковлєв / Zatsiorsky & Kraemer 2006). Диференційований час відновлення за системами. Три типи deload. Тайминг за тренажним віком (3+1, 4+1). Помилка «deload по відчуттях». clinical-safety: NOT_REQUIRED.
- `content/post-1489-individualizing-program-from-template.md` — Block 1451–1500 Cluster 4 post 9/10. Шаблони (5/3/1, Sheiko) як популяційний оптимум (Dankel et al. 2017). Три причини відхилення від шаблону. Трикроковий процес індивідуалізації. RPE-based авторегуляція (Helms et al. 2018, Zourdos et al. 2016). clinical-safety: NOT_REQUIRED.
- `content/post-1490-cluster-4-synthesis-training-age-variability-recovery.md` — Block 1451–1500 Cluster 4 post 10/10. Синтез: три осі (тренавальний вік / варіативність / відновлення) в єдиній матриці рішень. Таблиця всіх 10 постів кластеру з ключовим висновком. Прикладна рамка для self-coached intermediate. clinical-safety: NOT_REQUIRED.
- `content/post-1557-e1rm-diagnostic-tool-amrap-protocol.md` — Block 1556–1570 · Post 2 of 15 · «e1RM як діагностичний інструмент: AMRAP-протокол і обмеження формул» — чотири формули e1RM (Еплі 1985, Брзіцкі 1993, Ландер 1985, О'Конер 1989) і числові приклади; систематичне падіння точності при R>10 (Reynolds et al. 2006); вплив профілю м'язових волокон (Chapman et al. 1998, ±10–15% між індивідами); AMRAP-протокол: стандартизація умов (час доби, 48-год відновлення, тестова вага 75–85% e1RM, постійна схема розминки, RPE 9–9.5 після сету); читати e1RM як тренд у часі (три точки = сигнал, одна точка = шум); типові помилок (зміна формули між блоками, тест після важкого тижня, R>10, нестандартизовані умови); де e1RM принципово не підходить (перші 4–6 тижнів нової програми, після перерви, при нестабільній техніці, між рухами). clinical-safety: NOT_REQUIRED. sports-scientist review required before publish.

### Changed
- `blog.html` — оновлено до post-1490 (Cluster 4 posts 1481–1490) + додано картку post-1557.
- `STATUS.md` — Block 1451–1500 Cluster 4 COMPLETE; Total posts: 1169 → 1170; content engine row updated with post-1557 entry.
- `VERSION` — 6.42.0 → 6.43.0.

---

## [6.42.0] — 2026-06-19

### Added
- `docs/DATA_MODEL.md §41` — Enterprise Customer Adoption Snapshots Schema: canonical DATA_MODEL section for `enterprise_adoption_snapshots` (migration 0078). Closes cross-reference obligation from `docs/COST_MODEL.md §40.7` (v2.6, 2026-06-18). Three design invariants: no `user_id` structural guarantee, `form_api` REVOKED, k-anon gate at API layer. `adoption_health_band` ENUM (green/amber/red) GENERATED ALWAYS AS STORED from `wau_count / contracted_seats`. Full DDL with four GENERATED columns, `uq_tenant_snapshot_month` UNIQUE, `chk_adoption_coherent` CHECK, three indexes. RLS: `tenant_admin/owner` own-tenant SELECT; `tenant_manager` (HR) explicitly excluded; `compliance_reviewer` all; `form_system` write; `form_api` REVOKED. Four DEC-030 events: `enterprise.adoption_snapshot_filed` (STANDARD, 3yr, ADO-CHAIN-01), `enterprise.adoption_milestone_reached` (STANDARD, 3yr), `enterprise.adoption_health_downgraded` (HIGH, 3yr, Linear task + `system.csm_followup_overdue` advisory), `enterprise.qbr_completed` (STANDARD, 3yr, `privacy_floor_verified: z.literal(true)` HTTP 422 invariant). SOC 2 artefacts ADO-E-001/E-002/E-003 (CC4.1/A1.1/CC2.2/CC7.3/CC4.2). Ten-item implementation checklist (4× P0/M10, 4× P1/M10–M11, 2× P2). Three open questions OQ-ADO-01/02/03. TOC updated to add §40 and §41. DATA_MODEL header v1.19 → v1.20.

### Changed
- `docs/DATA_MODEL.md` — header v1.19 → v1.20; TOC extended with §40 (`tenant_siem_configs`) and §41 (`enterprise_adoption_snapshots`) entries.
- `VERSION` — 6.41.0 → 6.42.0.

---

## [6.41.0] — 2026-06-19

### Added
- `content/post-190-synthesis-ten-quarterly-questions.md` — Editorial series Block 181–190 synthesis · «Синтез 181–190: десять питань, які self-coached атлет має ставити щоквартально» — квартальна рамка для self-coached атлета: десять питань за чотирма рівнями (тренувальний вік і горизонт адаптації, відповідність програми реальному тижню і зовнішньому стресу, розвиток навичок і вибір вправ, ретроспективний аналіз трендів); Stults-Kolehmainen & Sinha 2014 (психологічний стрес і силова адаптація); McEwen (HPA-вісь і алостатичне навантаження); горизонти тренувального віку (сесія/мезоцикл/рік); мінімально ефективна кількість вправ; п'ять питань перед власним блоком. clinical-safety: NOT_REQUIRED. sports-scientist review required before publish.

### Changed
- `blog.html` — додано картку post-190 (Editorial Block 181–190 synthesis); blog.html updated to post-190.
- `STATUS.md` — Editorial series оновлено: post-190 written v6.41.0; Block 181–190 synthesis COMPLETE.
- `VERSION` — 6.40.0 → 6.41.0.

---

## [6.40.0] — 2026-06-19

### Added
- `content/post-1471-goal-specific-programming-framework.md` — Block 1451–1500 · Cluster 3 opener · пост 1/10. «Програмування під ціль: чому "тренуватись" і "тренуватись для чогось конкретного" — різні речі» — три принципові цілі (сила/гіпертрофія/кондиція), принцип специфічності цілі (Sale, 1988), проблема «зробити все одночасно» і interference (Wilson et al., 2012), ланцюжок ціль→провідна адаптація→параметри→структура, горизонти цілі (блок/мезоцикл/рік). clinical-safety: NOT_REQUIRED. sports-scientist review required before publish.
- `content/post-1472-maximal-strength-programming.md` — пост 2/10. «Програмування для максимальної сили: параметри, логіка, типові помилки» — три механізми нейром'язової адаптації (рекрутування, синхронізація, пригнічення Гольджі; Zatsiorsky & Kraemer), ключові параметри (≥85% 1RM, знижений обсяг, 3–5 хв відпочинок, максимальна специфічність), структура силового мезоциклу (4×4→4×3→4×2→2×1 прогресія або RPE-based), допоміжна/акцесорна мінімальна, силовий блок у контексті річного плану. clinical-safety: NOT_REQUIRED. sports-scientist review required before publish.
- `content/post-1473-hypertrophy-programming.md` — пост 3/10. «Програмування для гіпертрофії: обсяг, механіка, межі ефективного тренування» — три механізми (Schoenfeld, 2010): механічна напруга, метаболічний стрес, пошкодження м'язів; MEV/MAV/MRV концепція (RP Strength); широкий діапазон повторень (6–30+; Schoenfeld et al., 2017); відпочинок 1.5–3 хв (Schoenfeld et al., 2016); частота 2× краще за 1×; структура мезоциклу «хвиля підйомів»; гіпертрофія ≠ сила автоматично. clinical-safety: NOT_REQUIRED. sports-scientist review required before publish.
- `content/post-1474-conditioning-strength-endurance-programming.md` — пост 4/10. «Програмування для кондиції та силової витривалості» — кондиція як інфраструктура (Deweese et al., 2013); три типи: аеробна база (Зона 2), MetCon (AMRAP/EMOM/tabata), силова витривалість (висока щільність); провідна змінна — щільність; прогресія — більша робота за менший час; аеробна база як довгостроковий актив. clinical-safety: NOT_REQUIRED. sports-scientist review required before publish.
- `content/post-1475-strength-hypertrophy-combined-programming.md` — пост 5/10. «Поєднання сили та гіпертрофії в одному блоці» — природний зв'язок (структурний потенціал × нейром'язова ефективність); DUP і powerbuilding як стратегії (Rhea & Alderman, 2004); обмеження: ресурс відновлення, конкуруючі провідні параметри; чотири правила поєднання; для кого ефективно, для кого секвенування краще. clinical-safety: NOT_REQUIRED. sports-scientist review required before publish.
- `content/post-1476-concurrent-training-interference-effect.md` — пост 6/10. «Concurrent training та interference effect» — Wilson et al. (2012) мета-аналіз: сила −11%, гіпертрофія −31% vs. ізольоване силове; AMPK-mTOR конфлікт; тип кардіо (велосипед/гребля < бігу); частота (<3×/тиж мінімальний interference); часовий проміжок ≥6 год (Robineau et al., 2016); таблиця рекомендацій за сценарієм. clinical-safety: NOT_REQUIRED. sports-scientist review required before publish.
- `content/post-1477-goal-prioritization-annual-plan.md` — пост 7/10. «Пріоритизація цілей у річному плані» — секвенування vs. паралельність; різні часові рамки адаптацій (нейром'язова 4–6 тижнів, гіпертрофія 8–16+ тижнів, аеробна місяці); класична послідовність 5 фаз; тривалість кожної; варіанти А/Б/В для різних профілів; типові помилки. clinical-safety: NOT_REQUIRED. sports-scientist review required before publish.
- `content/post-1478-transitioning-between-training-goals.md` — пост 8/10. «Перехід між цілями тренування» — що відбувається наприкінці блоку (накопичена втома, адаптаційна стеля, специфічні патерни); роль перехідного тижня (deload); специфіка переходу гіпертрофія→сила, сила→гіпертрофія, сила→кондиція; ознаки поганого переходу; maintenance volume (Jäger et al., 2011). clinical-safety: NOT_REQUIRED. sports-scientist review required before publish.
- `content/post-1479-goal-specific-exercise-selection.md` — пост 9/10. «Ціль-специфічний вибір вправ» — три осі впливу цілі на вибір (специфічність/варіативність, ROM і навантажувальний профіль, вільне навантаження vs. машини); силовий блок (конкурентний рух незмінний, слабкі ланки, акцесорне мінімальне); гіпертрофічний (широкий арсенал, ізоляція дозволена); кондиційний (безпека при втомі, функціональні патерни); практична таблиця ціль→вправи. clinical-safety: NOT_REQUIRED. sports-scientist review required before publish.
- `content/post-1480-cluster-3-synthesis-programming-by-goal.md` — пост 10/10 · Cluster 3 synthesis. «Синтез кластеру 3: рамка програмування під ціль» — зведена таблиця трьох адаптацій і параметрів; п'ять питань для вибору провідної цілі; рамка річного плану (flowchart); підсумок interference і переходів; як Victor підтримує goal-specific програмування; що кластер не вирішує (advanced теми — Cluster 4). clinical-safety: NOT_REQUIRED. sports-scientist review required before publish.

### Changed
- `blog.html` — додано картки posts 1471–1480 (Cluster 3 · Програмування під конкретні цілі); blog.html updated to post-1480.
- `STATUS.md` — Block 1451–1500 оновлено: Cluster 3 COMPLETE (posts 1471–1480); footer v6.40.0.
- `VERSION` — 6.39.1 → 6.40.0.

---

## [6.39.1] — 2026-06-19

### Changed
- `docs/AUDIT_LOG_SCHEMA.md` — registered `system.gdir_alert_check_stale` DEC-030 HMAC-chained event (LOW severity, 1yr retention) in `### System` section (v2.20). Event emitted by `pg-cron-health-monitor` health supervisor when pg_cron job 35 (`google_directory_alert_check`, `*/5 * * * *`) exceeds its 6-min freshness window — leaves AL-SSO-GDIR-01/AL-SSO-GDIR-02 in a CC7.2 detection blind spot. Payload: `job_name`, `schedule`, `last_successful_run` (ISO 8601), `gap_minutes`, `freshness_window_minutes: 6`. Privacy invariant: no `user_id`, no `tenant_id`, no health data — infrastructure monitoring signal. PagerDuty P1 `form-devops`; dedup `gdir-alert-check-stale`. Retention table +1 row. SOC 2 quarterly evidence: SSO-OBS-E-007 Part B. Closes `docs/OBSERVABILITY.md §48.8` item 3 (P1/M4 — registration obligation flagged in CHANGELOG v6.37.1 — 🟢). Cross-ref: OBSERVABILITY §12.6 (job 35), OBSERVABILITY §48 (AL-SSO-GDIR-01/02), SOC2_READINESS §92 (SSO-OBS-E-007), DEC-067, DEC-030.
- `VERSION` — 6.39.0 → 6.39.1.

---

## [6.39.0] — 2026-06-19

### Added
- `content/post-1556-progress-evaluation-framework.md` — Block 1556–1570 · Cluster 1 opener · пост 1/15. «Як читати власний тренувальний прогрес: рамка без зовнішнього тренера» — п'ять каналів оцінки (e1RM/об'єм, RPE-тренд, технічна якість, відновлення між сесіями, суб'єктивна готовність); baseline-протокол на початку блоку; горизонти аналізу (сесія/тиждень/блок/рік); три систематичні помилки (короткий горизонт, один вимір, confirmation bias); різниця між «прогресу немає» і «прогресу не видно»; протокол 5-крокової ретроспективи блоку. Kenttä & Hassmén 1998, Moritani & deVries 1979, Hubal et al. 2005. clinical-safety: NOT_REQUIRED. sports-scientist review required before publish.

### Changed
- `README.md` — додано Block 1556–1570 «Практична оцінка тренувального прогресу без тренера» як нову proposed секцію (15 тем).
- `blog.html` — додано картку post-1556; blog.html updated to post-1556.
- `STATUS.md` — Block 1556–1570 додано: IN PROGRESS 1/15; v6.39.0 footer.
- `VERSION` — 6.38.0 → 6.39.0.

---

## [6.38.0] — 2026-06-19

### Added
- `content/post-1462-load-progression-models.md` — Block 1451–1500 · Cluster 2 · пост 2/10. Прогресія навантаження: лінійна (початківець), хвильова DUP (intermediate), блокова (advanced). Феномен відстроченої трансформації (Verkoshansky/Siff). Прогресія за об'ємом, щільністю і специфічністю — не тільки за вагою. clinical-safety: NOT_REQUIRED. sports-scientist review required before publish.
- `content/post-1463-volume-intensity-balance.md` — Block 1451–1500 · Cluster 2 · пост 3/10. MEV/MAV/MRV як рамка планування (Israetel et al.). Компроміс об'єму і інтенсивності: акумуляція — об'єм, реалізація — інтенсивність. Як MRV змінюється залежно від сну, стресу і тижня у блоці. Персональний пошук MRV через практику. clinical-safety: NOT_REQUIRED. sports-scientist review required before publish.
- `content/post-1464-training-frequency-distribution.md` — Block 1451–1500 · Cluster 2 · пост 4/10. 2× на м'яз на тиждень — стандарт (Schoenfeld 2016 мета-аналіз). PPL×2, Upper/Lower, Full Body: критерії вибору. Шість рухових патернів замість м'язових груп. Правило 1:1 жим–тяга. clinical-safety: NOT_REQUIRED. sports-scientist review required before publish.
- `content/post-1465-tempo-tut-evidence.md` — Block 1451–1500 · Cluster 2 · пост 5/10. TUT переоцінений як самостійний параметр — при контрольованому об'ємі різниця мінімальна (Schoenfeld & Grgic 2020). Ексцентрик важливий через механічний стимул, а не TUT. Концентрик: максимальне зусилля у реалізаційних фазах. Паузи і їх функція. clinical-safety: NOT_REQUIRED. sports-scientist review required before publish.
- `content/post-1466-deload-structure-planning.md` — Block 1451–1500 · Cluster 2 · пост 6/10. Механізм deload і відстрочена трансформація. Volume deload (40–60% підходів при збереженій вазі) — стандарт. Типи: volume/intensity/full. Ознаки overreaching vs. ознаки передчасного deload. Запланований (кожні 4–6 тижнів) і реактивний. clinical-safety: NOT_REQUIRED. sports-scientist review required before publish.
- `content/post-1467-mesocycle-transition-mechanics.md` — Block 1451–1500 · Cluster 2 · пост 7/10. Перехід між мезоциклами: аналіз блоку → тестування (якщо потрібне) → діагностика слабких ланок → вибір вправ → планування прогресії → фіксація. Тестування AMRAP vs. 1RM. Документаційна практика. clinical-safety: NOT_REQUIRED. sports-scientist review required before publish.
- `content/post-1468-weak-links-diagnosis-programming.md` — Block 1451–1500 · Cluster 2 · пост 8/10. Стicking point, технічна деградація, асиметрія. Діагностика через журнал і відео. Типові слабкі ланки в присіданні (upper back, quad), становій (sticking point старт, upper back локаут), жимі (трицепс). Адресація через допоміжні вправи. Асиметрія > 15% → unilateral. clinical-safety: NOT_REQUIRED. sports-scientist review required before publish.
- `content/post-1469-self-coached-programming-challenges.md` — Block 1451–1500 · Cluster 2 · пост 9/10. Специфіка self-coached: confirmation bias, уникання слабкості, RPE-деградація, послідовність, ego-driven рішення. Практичні контрмеханізми: відео, upfront-рішення, мінімальна/стандартна/максимальна версія сесії, AMRAP-калібровка. clinical-safety: NOT_REQUIRED. sports-scientist review required before publish.
- `content/post-1470-cluster2-synthesis.md` — Block 1451–1500 · Cluster 2 · пост 10/10 · **CLUSTER 2 COMPLETE**. Synthesis дев'яти постів у єдину операційну рамку: що робити до, під час і після мезоциклу. Місце Victor у рамці. Перехід до Cluster 3. clinical-safety: NOT_REQUIRED. sports-scientist review required before publish.

### Changed
- `blog.html` — додано записи post-1462 through post-1470; blog.html updated to post-1470.
- `STATUS.md` — Block 1451–1500 оновлено: Cluster 2 COMPLETE 20/50; next: Cluster 3.
- `VERSION` — 6.37.1 → 6.38.0.

---

## [6.37.1] — 2026-06-19

### Changed
- `docs/OBSERVABILITY.md` — §12.6 pg_cron health monitoring job registry: jobs 28–37 registered (v4.7.0 → v4.7.1). Closes ten cross-reference obligations across SSO_SCIM_IMPLEMENTATION.md, DATA_MODEL.md, SOC2_READINESS.md, and OBSERVABILITY.md itself. Job 28: `ci_telemetry_daily_sync` (CC8.1, §38.9). Job 29: `backup_age_monitor` (A1.2/CC7.2, §39.7, every 4h). Job 30: `quarterly_perf_regression_check` (A1.2/CC7.2, §40.6, PERF-SLO-06). Job 31: `wearable_sync_freshness_check` (CC7.2/WS-SLO-06, §41). Job 32: `turh_retention_purge` (CC6.3/GDPR Art. 17(3)(b), SSO_SCIM §29/DEC-051). Job 34: `bdg_override_expiry_sweep` (CC6.3/A1.2, SSO_SCIM §34.3/DEC-066, every 15min). Job 35: `google_directory_alert_check` (CC7.2/CC7.3, §48/SOC2_READINESS §92/DEC-067, every 5min — closes §48.8 item 3 P1/M4 obligation). Job 36: `dsar_slo_miss_counter_reset` (P5.1/CC7.2, DATA_MODEL §35.4/DEC-052, quarterly). Job 37: `caep_reregister_sweep` (CC6.3, SSO_SCIM §36/DEC-072, every 5min — closes SSO_SCIM §36.6 item 1 P0/M5 obligation). Conflict resolution: SSO_SCIM §34.3 and DATA_MODEL §35.4 both claimed "job 33" for newly authored jobs after `evidence_cron_freshness_check` (job 33) was already canonical; resolved as `bdg_override_expiry_sweep` → job 34, `dsar_slo_miss_counter_reset` → job 36. Freshness window note updated to cover 5-min, 15-min, and quarterly cadences. `system.gdir_alert_check_stale` LOW/1yr DEC-030 event noted for AUDIT_LOG_SCHEMA.md §System registration.
- `VERSION` — 6.37.0 → 6.37.1.

---

## [6.37.0] — 2026-06-19

### Added
- `content/post-1461-exercise-selection-as-programming-variable.md` — Block 1451–1500, Cluster 2 opener (post 1/10). Вибір вправ як програмна змінна: SAID-принцип (Zatsiorsky & Kraemer), ієрархія ролей (основна/допоміжна/акцесорна, Helms et al. 2015), вибір вправ у фазах переодизації (акумуляція/трансмутація/реалізація), DUP і специфічність руху, п'ять питань для вибору вправи, типові помилки self-coached програмування. clinical-safety: NOT_REQUIRED.

### Changed
- `blog.html` — blog card для post-1461 додано на початку списку.
- `STATUS.md` — Block 1451–1500 оновлено (11/50 · Cluster 2 IN PROGRESS); Current version → v6.37.0.
- `README.md` — Block 1451–1500 Cluster 2 теми (posts 1461–1470) додано до roadmap.
- `VERSION` — 6.36.0 → 6.37.0.

---

## [6.36.0] — 2026-06-19

### Added
- `content/post-1451-periodization-what-it-is-and-is-not.md` — Block 1451–1500 opener, Cluster 1 post 1/10. Що таке переодизація і що вона точно не є: Матвєєв (1965) і джерело поняття, робоче визначення (систематична зміна тренувальних змінних у часі), чим переодизація не є (ротація вправ, muscle confusion, надмірна складність), три рівні планування (макро/мезо/мікро), мінімальний поріг для recreational атлета. clinical-safety: NOT_REQUIRED.
- `content/post-1452-linear-periodization-mechanics-limits.md` — Cluster 1 post 2/10. Лінійна переодизація: механіка LP (об'єм→інтенсивність), чому ефективна для початківців, три точки де LP ламається (акомодація, стеля об'єму, однодиректаційність), варіації що продовжують корисність (хвильова LP, 3-тижнева хвиля, розгалужена), коли LP залишається першим вибором навіть для advanced. clinical-safety: NOT_REQUIRED.
- `content/post-1453-nonlinear-periodization-dup-wup.md` — Cluster 1 post 3/10. DUP vs. WUP: що робить переодизацію нелінійною, Daily Undulating Periodization (Rhea 2002, Miranda 2011, Colquhoun 2017), Weekly Undulating Periodization, порівняльна таблиця, переваги для intermediate, де нелінійність не вирішує проблему. clinical-safety: NOT_REQUIRED.
- `content/post-1454-block-periodization-accumulation-transmutation-realization.md` — Cluster 1 post 4/10. Блокова переодизація: модель Ісуріна (2008, 2010), принцип концентрації якостей, три типи блоків (акумуляція/трансмутація/реалізація), залишкові тренувальні ефекти, для кого актуально, спрощена 2-блокова версія для self-coached. clinical-safety: NOT_REQUIRED.
- `content/post-1455-conjugate-method-self-coached-athlete.md` — Cluster 1 post 5/10. Спряжений метод: походження у Верхошанського, адаптація Simmons (Westside), ME+DE дні, критичне питання перенесення без accommodating resistance і поза equipped пауерліфтингом, що реально запозичити принцип а не шаблон. clinical-safety: NOT_REQUIRED.
- `content/post-1456-autoregulation-rpe-vbt-overlay.md` — Cluster 1 post 6/10. Авторегуляція як overlay: RPE (Zourdos 2016) і VBT (González-Badillo 2010) як дві форми, чому «за відчуттями» ≠ авторегуляція, RPE-DUP практичний приклад, калібровка RPE, авторегуляція над системою а не замість. clinical-safety: NOT_REQUIRED.
- `content/post-1457-macrocycle-long-term-planning.md` — Cluster 1 post 7/10. Макроцикл: чому горизонт необхідний (фізіологічно і психологічно), штучний горизонт для non-competitive атлета (тест/бенчмарк), структура макроциклу (акумуляція→пікування→перехід→оцінка), 2 макроцикли/рік, backward planning. clinical-safety: NOT_REQUIRED.
- `content/post-1458-mesocycle-3-8-weeks-working-unit.md` — Cluster 1 post 8/10. Мезоцикл: логіка акумуляції втоми і суперкомпенсації, хвиля 4-тижневого мезоциклу (70–80/85–90/95–100/40–60%), чому атлети завершують мезоцикл занадто рано (стагнація = нормальна фаза), деловантаження: що є і що не є, мезоцикл і вік 35+. clinical-safety: NOT_REQUIRED.
- `content/post-1459-microcycle-weekly-structure.md` — Cluster 1 post 9/10. Мікроцикл: частота тренувань (2–3x/тиждень на групу — Schoenfeld et al. 2016), ієрархія Heavy/Medium/Light, секвенсування синергістів і антагоністів, інтерференційний ефект (Hickson 1980, Wilson et al. 2012), практичний 3-денний full body мікроцикл. clinical-safety: NOT_REQUIRED.
- `content/post-1460-synthesis-choosing-periodization-system.md` — Cluster 1 post 10/10, synthesis. П'ять запитань для вибору системи: тренувальний статус/ціль/дні/горизонт/фаза. Матриця рішень LP/DUP/блокова. Авторегуляція як overlay завжди. Прагматичний принцип FORM: найкраща система — та яку можна зрозуміти, виконувати і підтримувати. clinical-safety: NOT_REQUIRED.
- `docs/DECISION_LOG.md DEC-073` — Block 1451–1500 тема: Переодизація та довгострокове програмування. П'ять кластерів. Клінічна безпека: NOT_REQUIRED all. sports-scientist review required перед publish.

### Changed
- `blog.html` — 10 нових blog cards додано (posts 1451–1460 · Periodization block Cluster 1) на початку списку.
- `STATUS.md` — Block 1451–1500 рядок додано в content table (10/50 · IN PROGRESS · v6.36.0); Next priorities оновлено.
- `VERSION` — 6.35.0 → 6.36.0.

---

## [6.35.0] — 2026-06-19

### Added
- `docs/SSO_SCIM_IMPLEMENTATION.md §36` — OQ-SSO-23.1 · OQ-SSO-23.3 · OQ-SSO-23.4 Resolution (DEC-072): CAEP cert-rotation re-registration hook, SSF polling fallback SLA framing, RISC hijacking group cache eviction. Migration 0082 spec (`caep_reregistration_required`, `caep_reregistration_trigger`, `caep_last_reregistered_at`), `caep_reregister_sweep` pg_cron (job 37), `sso.caep_reregistration_queued` DEC-030 event, extended `sso.google_directory_sync_error` vocabulary. Closes three P2 open questions from §23.11 (all due M5). v2.7 → v2.8.
- `docs/DECISION_LOG.md DEC-072` — CAEP cert-rotation re-registration + SSF polling SLA + RISC hijacking cache eviction: three grounds each; reverse cost assessment; cross-references to §36.

### Changed
- `docs/SSO_SCIM_IMPLEMENTATION.md §23.11` — OQ-SSO-23.1, OQ-SSO-23.3, OQ-SSO-23.4 tracker rows updated from 🟡 P2 Open → 🟢 Resolved DEC-072 (2026-06-19).
- `VERSION` — 6.34.0 → 6.35.0.

---

## [6.34.0] — 2026-06-19

### Added
- `content/post-1442-athletic-identity-35-plus-build-break-rebuild.md` — Тренувальна ідентичність після 35: AIMS і логіка підтвердження (Brewer et al. 1993), три механізми руйнування (накопичення ролей, порівняння з молодим собою, травми), три кроки відновлення після перерви. Cluster 5, post 2/10. clinical-safety: NOT_REQUIRED.
- `content/post-1443-minimum-training-standard-35-plus-threshold.md` — Мінімальний тренувальний стандарт: фізіологія підтримуючого тренування (Bickel 2011, Ralston 2017, Androulakis-Korakakis 2020), аудит найгіршого тижня, різниця між МТС і ситуативним «лінивим тижнем». Cluster 5, post 3/10. clinical-safety: NOT_REQUIRED.
- `content/post-1444-structure-over-decisions-35-plus-training.md` — Структура замість рішень: decision fatigue (Baumeister 2007), implementation intentions (Gollwitzer 1999, 2006 meta-analysis), контекстуальні тригери і звичка (Wood & Neal 2007, Neal et al. 2012). Cluster 5, post 4/10. clinical-safety: NOT_REQUIRED.
- `content/post-1445-plastic-program-three-week-versions-35-plus.md` — Пластична програма: стандартний/скорочений/форс-мажорний тиждень, MEV/MAV/MRV рамка (Israetel et al.), клас A/B/C вправ, алгоритм вибору версії тижня. Cluster 5, post 5/10. clinical-safety: NOT_REQUIRED.
- `content/post-1446-visible-progress-training-log-motivation-35-plus.md` — Видимий прогрес: SDT і потреба в компетентності (Deci & Ryan 2000), що відстежувати для 35+ (e1RM, RPE, між-підходне відновлення), місячний тренд vs. тижневий шум. Cluster 5, post 6/10. clinical-safety: NOT_REQUIRED.
- `content/post-1447-return-after-break-protocol-35-plus.md` — Повернення після перерви: міонуклеарна ретенція (Bruusgaard et al. 2010; Egner et al. 2016), асиметрична крива детренування, психологічні бар'єри, двотижневий реінтеграційний блок 60–70%. Cluster 5, post 7/10. clinical-safety: NOT_REQUIRED.
- `content/post-1448-multiple-identities-35-plus-protect-athletic.md` — Множинні ідентичності після 35: ієрархія значущості ролей (Stets & Burke 2000), намір vs. зобов'язання (Grove et al. 1997), небезпека over-identification, п'ятикрокова рамка ранжування. Cluster 5, post 8/10. clinical-safety: NOT_REQUIRED.
- `content/post-1449-long-term-athletic-identity-10-20-30-years.md` — Довгострокова атлетична ідентичність: masters athletes (Tanaka & Seals 2008, Pollock et al. 1997), чотири фази мотивації (performance → health → mastery → legacy), deliberate practice (Ericsson et al. 1993). Cluster 5, post 9/10. clinical-safety: NOT_REQUIRED.
- `content/post-1450-cluster-5-synthesis-adherence-athletic-identity-35-plus.md` — Синтез кластеру 5: взаємодія п'яти механізмів адгеренсу, аудит-чекліст, failure modes за механізмом, триступеневий протокол повернення, завершення блоку 1401–1450. Cluster 5, post 10/10 · Block synthesis. clinical-safety: NOT_REQUIRED.
- `content/post-181-training-age-progress-horizons.md` — Тренувальний вік і очікування прогресу: три горизонти, де логіка роботи різна. Тренувальний вік vs. час у залі. Три горизонти адаптації: горизонт 1 (0–18 міс., сесійний рівень, лінійна прогресія), горизонт 2 (18 міс.–3 роки, мезоцикловий рівень, хвилеподібна прогресія), горизонт 3 (3+ роки, річний рівень, блокова логіка). Чотири евристики самодіагностики. Порівняльна таблиця параметрів по горизонтах. clinical-safety PASS. Pending: sports-scientist review.
- `README.md` — Розширення roadmap editorial series: posts 211–220 «Системне мислення атлета» — 10 нових тем (think in systems, ієрархія сигналів від тіла, «добра» vs. «популярна» програма, MED, консистентність, «слухати тіло» vs. виправдання, структура vs. відчуття, когнітивний ресурс і планування, довгострокова тренованість як інвестиція, синтез).

### Changed
- `blog.html` — картки posts 1442–1450 додані. Картки додані: post-181 (Training Science · 13 хв · 2026-06-19); post-15, post-14, post-13, post-12 (Training Science · 2026-05-16). blog.html updated to post-1450 · post-181.
- `STATUS.md` — Block 1401–1450: BLOCK COMPLETE 50/50. 11–50: blog.html cards added for posts 12–15. 100–200: 101 → 102 posts, post-181 added. Version header updated to v6.34.0.
- `VERSION` — 6.33.1 → 6.34.0.

---


## [6.33.1] — 2026-06-19

### Added
- `docs/OBSERVABILITY.md §50` — OQ-SIEM-03 Resolution: HMAC Chain Verification for Tenant SIEM Consumers — Documentation-First Algorithm Specification with Concrete Library Trigger (DEC-071, v4.7.0). Adopts Option B (documentation-only): publishes `compliance/docs/hmac-chain-verification-algorithm.md` as Data Room artefact HMAC-VERIFY-ALGO-001 (DEC-030 chain structure, Python verification pseudocode, 3-event test vector, per-tenant isolation note, cursor-continuity guidance). No open-source library at GA. Library trigger: ≥ 2 distinct pilot CSM requests (M10–M13) → Python reference implementation before GA; Go deferred to post-GA. Per-tenant HMAC verification key via HKDF-SHA256. Seven-item implementation checklist (3× P0/M9–M10, 2× P1/M10, 1× P1/M12, 1× P2/M13-if-triggered). Marketing claim "HMAC-chained audit log" confirmed accurate without library (§50.8). Closes OQ-SIEM-03 (open since docs/OBSERVABILITY.md v1.4, 2026-06-01).
- `docs/DECISION_LOG.md DEC-071` — HMAC chain verification documentation-first decision: five grounds (claim accuracy, adequate capability match, asymmetric maintenance burden, stronger due-diligence signal, pattern consistency with DEC-043/051/053/065/067). Reverse cost: low (library is additive; spec unchanged).

### Changed
- `docs/OBSERVABILITY.md` — version header v4.6.0 → v4.7.0. OQ-SIEM-03 row in §27.12, §34.10, and §47.10 updated from 🟡 Open P2 → 🟢 Resolved DEC-071 (§50).
- `VERSION` — 6.33.0 → 6.33.1.

## [6.33.0] — 2026-06-19

### Added
- `content/post-1441-adherents-35-plus-cluster5-opener.md` — П'ять механізмів, які утримують атлета 35+ у тренуванні: ідентичність як якір, мінімальний тренувальний стандарт (Bickel 2011; Ralston 2017; Androulakis-Korakakis 2020), структура замість рішень (Wood & Neal 2007), пластична програма з трьома рівнями, зворотний зв'язок і автономна мотивація (Teixeira et al. 2012). Cluster 5 opener · Block 1401–1450. clinical-safety: NOT_REQUIRED.
- `README.md` — Block 1541–1555 «Тренування як система прийняття рішень» додано до роадмапу (15 тем: евристика рішень, пріоритети при обмеженому часі, три рівні рішень, чеклісти, confirmation bias, межа між аналізом і infomania).

### Changed
- `blog.html` — картка post-1441 додана (Training 35–55 · 13 хв · 2026-06-19). blog.html updated to post-1441.
- `STATUS.md` — Block 1401–1450: 40/50 → 41/50, Cluster 5 IN PROGRESS (тема: Адгеренс і довгострокова атлетична ідентичність для 35+). Version header updated to v6.33.0.
- `VERSION` — 6.32.0 → 6.33.0.

## [6.32.0] — 2026-06-19

### Added
- `content/post-1432-son-yak-trenuvalʼna-zminna.md` — Сон як тренувальна змінна: операційний протокол для атлета з реальним графіком. Сон як активна фаза адаптації (MPS, GH-секреція, моторна пам'ять, ЦНС). Архітектура сну N3/REM. Протокол трьох рівнів (постійний час пробудження, буфер перед сном, управління розкладом). Тактика при кумулятивному дефіциті. clinical-safety: NOT_REQUIRED.
- `content/post-1433-nakopychena-vtoma-35-plus.md` — Накопичена втома 35+: як відрізнити тренувальну втому від системного перевантаження. Нормальна втома vs. FO vs. NFO. Операційний чекліст 14 пунктів. Об'єктивні маркери: HRV, ЧСС спокою, перфоманс RPE-трекінг. Реакція по зонах. clinical-safety: NOT_REQUIRED.
- `content/post-1434-delovantazhennia-35-plus.md` — Деловантаження 35+: структура, частота, протоколи. Фізіологічне обґрунтування (сполучна тканина, ЦНС, алостатичне навантаження). Три типи деловантажу (плановий, технічний, активне відновлення при NFO). Частота 3–4 тижні. Детальний протокол. Психологія деловантажу. clinical-safety: NOT_REQUIRED.
- `content/post-1435-hrv-praktyka-35-plus.md` — HRV у практиці атлета 35+: від вимірювання до рішення. Стандартний протокол вимірювання. 14–21-денний базовий рівень. Операційна схема рішення (чотири сценарії HRV × суб'єктивний стан). Де HRV корисний і де обмежений. clinical-safety: NOT_REQUIRED.
- `content/post-1436-systemnyy-stres-trenuvalʼna-yemnistʼ.md` — Системний стрес і тренувальна ємність. Алостатичне навантаження і «бюджет відновлення». Три категорії нетренувального стресу. Три версії тижня (ідеальний, напружений, кризовий). Тактика адаптації для специфічних ситуацій. clinical-safety: NOT_REQUIRED.
- `content/post-1437-metody-vidnovlennia-dokazova-baza.md` — Методи відновлення: ієрархія доказовості. Рівень 1 (сон, харчування, програмування). Рівень 2 (CWI і нюанс гіпертрофії, сауна, активне відновлення, масаж). Рівень 3 (компресія, розтяжка, кріокамери). Рівень 4 (флоатинг, BCAA, EMS, відновлювальні добавки). clinical-safety: NOT_REQUIRED.
- `content/post-1438-vidnovlennia-kharchovyy-rezhym.md` — Відновлення і харчовий режим: ≥1,6–2,2 г/кг білка, leucine threshold, розподіл протягом дня, тайміг (добова кількість > «вікно»), нічний казеїн. Калорійний баланс і MPS. Гідратація. clinical-safety: REVIEW_REQUIRED (очікується CONDITIONAL PASS).
- `content/post-1439-dovhostrokova-stiykistʼ-atleta.md` — Довгострокова стійкість атлета. Типові причини циклів «почав–зупинився». П'ять ознак атлетів з 20-річною практикою. Мінімальний тренувальний стандарт. Адгеренс як системний результат. Зв'язок управління відновленням і стійкості. clinical-safety: NOT_REQUIRED.
- `content/post-1440-syntez-klastera-4.md` — Синтез кластеру 4: операційний протокол відновлення для 35+. Ієрархія чотирьох рівнів. Аудит-чекліст відновлення. Мінімальна операційна система (щоранковий ритуал + щотижневий мінімум). CLUSTER 4 COMPLETE 10/10. clinical-safety: NOT_REQUIRED.

### Changed
- `blog.html` — картки post-1432–1440 додані (Training 35–55 · 2026-06-19). blog.html updated to post-1440.
- `STATUS.md` — Block 1401–1450: 31/50 → 40/50, Cluster 4 COMPLETE. Next: Cluster 5 (posts 1441–1450, тема TBD). Version header updated to v6.32.0.
- `VERSION` — 6.31.0 → 6.32.0.

## [6.31.0] — 2026-06-19

### Added
- `content/post-1431-vidnovlennia-35-plus-fiziolohiia.md` — Відновлення після 35: фізіологія, якої не вистачало у вашій програмі. Cluster 4 opener (posts 1431–1440 · Відновлення і довгострокова стійкість для 35+). Що змінюється біологічно (синтез м'язового білка, запальна відповідь, гормональний фон, ЦНС відновлення). Чотири компоненти втоми. Три ключові змінні відновлення: сон, баланс навантаження, системний стрес. HRV як операційний інструмент. Системна накопичена втома: ознаки і логіка реакції. Деловантаження як структурний елемент, не покарання. clinical-safety: NOT_REQUIRED.
- `README.md` — Block 1526–1540 «Тренувальна культура і середовище для self-coached атлета» додано до роадмапу (15 тем: вибір залу, тренувальний партнер, онлайн-спільноти, підзвітність, адгеренс і середовище).

### Changed
- `blog.html` — картка post-1431 додана (Training 35–55 · 13 хв · 2026-06-19). blog.html updated to post-1431.
- `STATUS.md` — Block 1401–1450 row: 30/50 → 31/50, Cluster 4 IN PROGRESS. Version header updated to v6.31.0.
- `VERSION` — 6.30.0 → 6.31.0.

## [6.30.0] — 2026-06-19

### Added
- `content/post-1422-tyah-poshtovkh-balans-35.md` — Баланс тяг-поштовх для атлета 35+: симетрія як перший принцип. Рахунок підходів як практична діагностика (ціль ≥1:1). Горизонтальний і вертикальний тяг — вибір варіацій і суглобовий профіль. Face pull як структурна вправа. Зовнішні ротатори і середня трапеція. Зв'язок дисбалансу з хронічною шийною напругою. clinical-safety: NOT_REQUIRED.
- `content/post-1423-vilni-vahi-vs-trenajery-35.md` — Вільні ваги vs тренажери для 35+: механічний вибір замість культурного. Що вільна вага робить добре (стабілізатори, адаптивна траєкторія). Що тренажер робить добре (ізоляція, нижчий суглобовий стрес, безпека без рятівника). Практична карта замінників за рухомими патернами. Оптимальна стратегія — інтеграція. clinical-safety: NOT_REQUIRED.
- `content/post-1424-rukhova-kompetentnist-35.md` — Рухова компетентність для 35+: спочатку рух, потім вага. Три компоненти: діапазон, стабільність, навантажена компетентність. Чотири практичних рухових тести. Два підходи до корекції (зупинка vs замінник + паралельна робота). Технічна деградація як ранній сигнал. Протокол повернення після перерви. clinical-safety: NOT_REQUIRED.
- `content/post-1425-temp-eksentryk-35.md` — Темп і ексцентрик для 35+: контроль як механічна перевага. Три механізми ексцентрика: механічна напруга, адаптація сухожиль, гіпертрофія. Темп нотація і три базові режими (3-0-1-0, 4-2-X-0, 2-3-1-0). Де ексцентрик виправданий, де — ні. Zниження ваги при введенні темпу — зміна механічного профілю, не регрес. clinical-safety: NOT_REQUIRED.
- `content/post-1426-unilateral-asimetriya-35.md` — Унілатеральний тренінг для 35+: виявити, виміряти, виправити. Чому bilateral маскує асиметрії. Протокол діагностики (split squat, single-arm row, single-leg RDL). Порогові значення: < 10% норма, 10–15% корекція, > 15% пріоритет. Стратегія «слабша сторона диктує». Унілатеральне навантаження і нижчий осьовий стрес. clinical-safety: NOT_REQUIRED.
- `content/post-1427-zamina-vprav-bez-vtraty-stymulu.md` — Заміна вправ без втрати стимулу: практичний фреймворк для 35+. Чотири рівні еквівалентності (рухова задача, цільова група, профіль навантаження, прогресія). Критичне питання: що конкретно не підходить. Карта замінників для всіх основних рухомих патернів. Тимчасова vs постійна заміна. clinical-safety: NOT_REQUIRED.
- `content/post-1428-rukhovi-priorytety-35.md` — Рухові пріоритети для 35+: розвивати, підтримувати, обмежити. Активно розвивати: грудна рухливість, задній ланцюг, лопаткові стабілізатори, рухливість стегон. Підтримувати: базова сила, аеробна база. Обмежити: вправи з хронічним дискомфортом, регулярний > 90% 1RM без змагальної мети. Практичний аудит 5 кроків. clinical-safety: NOT_REQUIRED.
- `content/post-1429-core-stabilizatsiya-35.md` — Core і стабілізація для 35+: за межами планки і кранчів. Core як стабілізаційна система, не «вправи для живота». Чотири напрямки: антифлексія (dead bug, RKC plank), антиекстензія (ab wheel, hollow body), антиротація (Pallof press, suitcase carry), ротаційна сила (cable woodchop). IAP і дихання. Інтеграція у розминку і між підходами. clinical-safety: NOT_REQUIRED.
- `content/post-1430-syntez-klastera-3.md` — Синтез кластера 3: вибір вправ і рухові пріоритети для 35+ — операційний протокол. Ієрархія 5 рівнів: механічна логіка → структура (баланс + асиметрія) → інструментарій → якість виконання → стратегія. Чекліст аудиту програми. 4-тижневий протокол введення змін. Що не робити. Cluster 3 COMPLETE (posts 1421–1430 · Вибір вправ і рухові пріоритети для 35+). clinical-safety: NOT_REQUIRED.

### Changed
- `blog.html` — 9 карток додано для постів 1422–1430 (Training 35–55 · 13 хв · 2026-06-19). blog.html updated to post-1430.
- `VERSION` — 6.29.1 → 6.30.0.

## [6.29.1] — 2026-06-19

### Added
- `docs/OBSERVABILITY.md §49` — OQ-SSO-OBS-01 Resolution: per-tenant SSO-SLO-01 evaluation confirmed (ENTERPRISE_SLA §3.1); fleet-wide SSO-SLO-01b companion signal added (AL-SSO-FLEET-01, P1, pg_cron job 36 `sso_fleet_health_check`); DEC-070. Closes P1 open question open since v1.3.
- `docs/DECISION_LOG.md DEC-070` — OQ-SSO-OBS-01 decision: per-tenant confirmed, SSO-SLO-01b operational companion, `sla_credit_impact: 'none'`.

### Changed
- `docs/OBSERVABILITY.md` — header v4.5.0 → v4.6.0; §26.13 OQ-SSO-OBS-01 row → 🟢 Resolved DEC-070; §48.7 OQ gap tracker OQ-SSO-OBS-01 → 🟢 Resolved DEC-070.
- `VERSION` — 6.29.0 → 6.29.1.

## [6.29.0] — 2026-06-19

### Added
- `content/post-1421-vybit-vprav-35-mekhanichna-lohika.md` — Вибір вправ для атлета 35+: механічна логіка замість традиції. Cluster 3 opener (posts 1421–1430 · Вибір вправ і рухові пріоритети для 35+). П'ять питань перед будь-якою вправою (рухова задача, рухливість, суглобовий профіль, тренувальний статус, прогресивне навантаження). Сім рухових патернів з варіаціями і суглобовим профілем. Унілатеральний тренінг як детектор асиметрій. Операційний алгоритм 5 кроків. Практичний розбір: жим лежачи, присідання, підтягування, OHP. clinical-safety: NOT_REQUIRED. sports-scientist review required.
- `README.md` — Block 1511–1525 (AI і технологічні інструменти в самостійному тренінгу) added to content roadmap · v6.29.0.

### Changed
- `blog.html` — card added for post-1421 (Training 35–55 · 13 хв · 2026-06-19).
- `STATUS.md` — Block 1401–1450: 20/50 → 21/50; Cluster 3 STARTED v6.29.0.
- `VERSION` — 6.28.0 → 6.29.0.

## [6.28.0] — 2026-06-19

### Added
- `content/post-1412-obsiah-35-plus-chastyna-1.md` — Об'єм для атлета 35+: як визначити ефективний коридор між МЕД і МАВ. МЕД vs. МАВ визначення, чому після 35 МАВ нижчий (Bamman 2007: -35–50% сателітноклітинна відповідь), операційні способи знайти МЕД і МАВ, орієнтовні числа по м'язових групах, три типові помилки обліку об'єму.
- `content/post-1413-obsiah-35-plus-chastyna-2.md` — Об'єм для 35+: розподіл між м'язовими групами при обмеженому часі. Три рівні пріоритизації, «вкрадений об'єм» компаундних вправ, ієрархія захисту від атрофії (квадрицепс, сідниці, верхня спина першочергово), практичний приклад 50 підходів/тиждень.
- `content/post-1414-chastota-35-plus-mikrotsykl.md` — Частота для 35+: як будувати мікроцикл для зрілого атлета. Доказова база частоти (Schoenfeld 2016, McLester 2000). Відновне вікно після важкої сесії 35+: 72–96 год (Ahtiainen 2003). Три конфігурації: full-body 3x, upper-lower 4x, PPL 3x. Важкі рухові патерни: станова 1x/10 днів.
- `content/post-1415-chastota-35-plus-rozmishchennia.md` — Частота для 35+: розміщення важких сесій і нестабільний розклад. Ієрархія tier 1–3. Чотири сценарії: пропуск, 48 год між важкими, відсутність залу 5–7 днів, хаотичний тиждень. «Ковзний тиждень» як системне рішення.
- `content/post-1416-intensyvnist-35-plus-diapazony.md` — Інтенсивність для 35+: діапазони RPE і % RM. Чому важка робота необхідна: fast-twitch і саркопенія. Три зони з ефектами і відновною вартістю. Чому RPE точніший за % RM після 35. Три типові помилки інтенсивності.
- `content/post-1417-intensyvnist-35-plus-avtorehuliatsiya.md` — Авторегуляція після 35: від теорії до операційного протоколу. RPE і RIR як інструменти. Три рівні авторегуляції. Zourdos 2016: точність RPE ±1. Mann 2010: авторегулятивні vs. фіксовані протоколи. Авторегуляція в структурі periodization.
- `content/post-1418-periodyzatsiia-35-plus-blok-struktura.md` — Periodization для 35+: блоки і мезоцикли. Три рівні periodization. Блочна модель: акумуляція (3–4 тижні), інтенсифікація (2–3 тижні), деловантаження. Адаптації для нестабільного розкладу. Rhea & Alderman 2004: +10–15% сили при periodized.
- `content/post-1419-periodyzatsiia-35-plus-delovantazhennia.md` — Деловантаження після 35: структурний елемент, не ситуативний. Два типи накопиченого навантаження. Правило 3–4 тижні + 1 деловантаження. Тип А (знижений об'єм, збережена інтенсивність) vs. Тип Б. Bickel 2011: підтримання адаптації при 1/3 об'єму.
- `content/post-1420-syntez-klastera-2.md` — Синтез кластера 2: 13 операційних правил зведеною таблицею. Два шаблони: «Зайнятий батько» (full-body 3x) і «Self-coached intermediate» (upper-lower 4x) з числовою прогресією об'єму і RPE. П'ять незмінних принципів. Cluster 2 COMPLETE.

### Changed
- `blog.html` — cards added for posts 1412–1420 (9 cards · Training 35–55 · 2026-06-19).
- `STATUS.md` — Block 1401–1450: 11/50 → 20/50; Cluster 2 COMPLETE v6.28.0.
- `VERSION` — 6.27.2 → 6.28.0.

---

## [6.27.2] — 2026-06-19

### Added
- `docs/SSO_SCIM_IMPLEMENTATION.md §35` — OQ-SSO-34.2 Resolution: BDG `getGuardConfig()` Seat Source (DEC-069). Decision: `enterprise_contracts.contracted_seats` with 3600s KV safety-net TTL + active invalidation on `billing.seats_expanded`/`billing.seats_reduced` DEC-030 events. Option B (real-time `COUNT(active tenant_users)`) rejected on anti-DoS, audit-traceability, and pattern-consistency grounds. Six-dimension option matrix, updated `getGuardConfig()` TypeScript spec, four edge cases, §35.7 SOC 2 CC6.3 three-step audit reconstruction protocol, five-item implementation checklist (3× P0/M13 + 2× P1/M13).
- `docs/DECISION_LOG.md DEC-069` — formal adoption record for OQ-SSO-34.2.

### Changed
- `docs/SSO_SCIM_IMPLEMENTATION.md` — §34.12 OQ-SSO-34.2 row → 🟢 Resolved (DEC-069); document header v2.6 → v2.7.
- `VERSION` — 6.27.1 → 6.27.2.

---

## [6.27.1] — 2026-06-19

### Added
- `content/post-1411-prohramuvannia-35-plus-zahalni-pryntsypy.md` — Block 1401–1450, Cluster 2 opener, post 1411. «Програмування після 35: як фізіологія першого кластера перекладається у структуру тренувань» — три змінні три коригування: об'єм (вужчий МЕД–МАВ коридор), частота (2x/тиждень стандарт, розширене відновне вікно), інтенсивність (70–85% 1RM оптимум); чому «щадний режим» — неправильна відповідь; рамка кластера 2 (пости 1412–1420). Peterson et al. 2011, Androulakis-Korakakis 2020, Bickel 2011, Ahtiainen 2003, Schoenfeld 2016, Harridge 1999. clinical-safety: PASS.

### Changed
- `blog.html` — post-1411 card added (Training 35–55 · 13 хв · 2026-06-19).
- `STATUS.md` — Block 1401–1450 updated: 10/50 → 11/50; Cluster 2 in progress; post-1411 note added.
- `VERSION` — 6.27.0 → 6.27.1.

---

## [6.27.0] — 2026-06-19

### Added
- `content/post-1401-trenuvannia-35-55-vstup.md` — Block 1401–1450 opener. «Тренування 35–55: чому це окрема тема, а не той самий протокол для старших». Дві помилки: ігнорувати зміни або надмірно консервувати. Шість систем, що змінюються. Що не змінюється: принципи адаптації, здатність до гіпертрофії. DEC-068. clinical-safety: NOT_REQUIRED. sports-scientist review required.
- `content/post-1402-hormonalnyy-fon-pislia-35.md` — «Гормональний фон після 35». Тестостерон −1–2%/рік (Harman et al., Baltimore Longitudinal Study); SHBG зростає → вільний тестостерон відносно менший. GH і IGF-1 — паралельне зниження. Кортизол/анаболіки: менш буферне відношення. Практика: сон як фізіологічний пріоритет; 72-год мінімум; MEV замість MAV. clinical-safety: NOT_REQUIRED.
- `content/post-1403-sarkopeniya-mekhanizmy.md` — «Саркопенія: коли починається, яким темпом іде». ~0.5–1%/рік після 35. Fast-twitch волокна типу II атрофуються непропорційно (Lexell et al.). Механізми: денервація, сателітна активність, inflammaging, mTOR інтерференція, синтез білка. Frontera 1988, Fiatarone 1994. clinical-safety: CONDITIONAL PASS (фокус на функції, не зовнішньому вигляді).
- `content/post-1404-vidnovna-zdatnist-pislia-35.md` — «Відновна здатність після 35». Inflammaging, знижена сателітна активність, нейром'язова готовність як окрема система. Маркери: ВСР, ЧСС спокою, суб'єктивна готовність, технічна якість першого підходу. 72-год мінімум між важкими сесіями; деловантаження 3–4 тижні. clinical-safety: NOT_REQUIRED.
- `content/post-1405-spoluchna-tkanyma-vik.md` — «Сполучна тканина і вік». Сухожилля адаптуються 3–6 місяців. Знижений синтез колагену, зміна механічних властивостей (Wang et al.), нижча васкуляризація. Механічне навантаження стимулює синтез колагену (Kjaer et al.). Ізометричні і ексцентричні протоколи. clinical-safety: CONDITIONAL PASS (немає стверджень про «запобігання травмам»).
- `content/post-1406-nervova-systema-rukhovist-35.md` — «Нервова система і рухова якість після 35». RFD знижується раніше за максимальну силу (Häkkinen). Fast-twitch рухові одиниці — Kamen & Knight. Мієлінізація і провідність. Рухова пам'ять зберігається. Що тренується: 85–95% RM, вибухові рухи. clinical-safety: NOT_REQUIRED.
- `content/post-1407-vo2max-kardio-adaptatsiya-35.md` — «VO2max і серцево-судинна адаптація після 35». Зниження ~1%/рік у нетренованих; різниця тренований/нетренований того ж віку — 20–40%. Відповідь на аеробний тренінг зберігається. HIIT ефективний (Wisloff et al.). Аеробна база і відновлення між підходами. clinical-safety: NOT_REQUIRED.
- `content/post-1408-son-pislia-35.md` — «Сон після 35». SWS знижується (Van Cauter et al. 2000). GH секреція і SWS: замкнений цикл. Деградація не детермінована: тренований стан збільшує SWS. Постійний розклад підйому, температура, відсутність важкого тренінгу пізно ввечері. clinical-safety: NOT_REQUIRED.
- `content/post-1409-sylova-vidpovid-35.md` — «Силова відповідь після 35». Гіпертрофія реальна але повільніша. Максимальна сила зберігається краще за потужність. Hubal et al., Verdijk et al., Peterson et al. (+30% сили у 50+). Два профілі: нижча початкова відповідь і повільніша реалізація. Горизонт оцінки 12–16 тижнів. clinical-safety: NOT_REQUIRED.
- `content/post-1410-syntez-klastera-1.md` — «Синтез кластера 1: дев'ять змін після 35». Зведення 9 тем у 5 практичних принципів: MEV як точка відліку; інтенсивність не нижча але паузи більші; fast-twitch потребує прямого стимулу; деловантаження структурне; горизонт оцінки 12–16 тижнів. Cluster 1 COMPLETE. clinical-safety: NOT_REQUIRED.
- `docs/DECISION_LOG.md` — DEC-068: Block 1401–1450 тема — Тренування 35–55.

### Changed
- `blog.html` — posts 1401–1410 cards added (Training 35–55 · 2026-06-19). blog.html updated to post-1410.
- `STATUS.md` — Block 1401–1450 updated: Cluster 1 COMPLETE (posts 1401–1410 · фізіологічна основа), 10/50.

---

## [6.26.1] — 2026-06-19

### Added
- `content/post-171-video-self-analysis.md` — Editorial post 171. «Відеоаналіз власного тренування: що бачить камера, що бачить тренер, що не бачить жоден» — KR vs. KP (Schmidt & Lee 2011); concurrent vs. terminal feedback (Salmoni 1984); confirmation bias при самоаналізі (Miall & Wolpert 1996); що камера бачить добре і де її сліпі плями; що бачить тренер vs. що не бачить; практичний протокол двох ракурсів і одного конкретного питання на сесію. clinical-safety: NOT_REQUIRED. sports-scientist review required before publish.

### Changed
- `blog.html` — post-171 card added (Training science · 13 хв · 2026-06-19).
- `STATUS.md` — Editorial series updated to 11–37, 44–50, 106–122, 170–171, 1361; post-171 note added.
- `VERSION` — 6.26.0 → 6.26.1.

---

## [6.26.0] — 2026-06-19

### Added
- `content/post-1392-hnuchka-arkhitektura-trenuvalnoho-roku.md` — Cluster 5, post 2/10. Гнучка архітектура тренувального року: прив'язані фази (якорі) + гнучке наповнення. Три горизонти планування (рік/блок/тиждень). Ретроспективний аудит для ідентифікації сезонних якорів. Практичний приклад атлета з двома «важкими сезонами». Issurin 2010, Plisk & Stone 2003, Haff & Triplett 2016. clinical-safety: PASS.
- `content/post-1393-perekhidni-tochky-mizh-blokamy.md` — Cluster 5, post 3/10. Перехідний тиждень між блоками як самостійна фаза з чотирма функціями: фізичне відновлення, оцінка результатів, планування наступного, психологічна декомпресія. Три сценарії між блоками і чому перехідна фаза виграє. Mujika & Padilla 2000, Bickel 2011. clinical-safety: PASS.
- `content/post-1394-trenuValna-identychnist-na-dovhomu-horyzonti.md` — Cluster 5, post 4/10. Тренувальна ідентичність: через результат vs. через процес. Три сценарії тиску (примусова пауза, хронічна зміна умов, вікова зміна показників). Що підтримує ідентичність через процес. Brewer, Van Raalte & Linder 1993, Androulakis-Korakakis 2020. clinical-safety: PASS.
- `content/post-1395-piat-meta-navychok-trenovanogo-atleta.md` — Cluster 5, post 5/10. П'ять мета-навичок тренованого атлета: темпування адаптацій, читання власних сигналів та autoregulation, управління програмним переходом, планування на трьох горизонтах, оновлення переконань від власних даних. Hubal 2005, Androulakis-Korakakis 2021, Bompa & Haff 2009. clinical-safety: PASS.
- `content/post-1396-upravlinnia-prohresom-cherez-roky.md` — Cluster 5, post 6/10. Управління прогресом через роки: поняття «розширення ємності» vs. «збільшення піку». П'ять альтернативних метрик: субмаксимальна ефективність, стабільність якості, час відновлення, adherence rate, порівняння з собою рік тому. Schoenfeld 2010, Zourdos 2016. clinical-safety: PASS.
- `content/post-1397-chytannia-bahatokhrichnykh-danykh.md` — Cluster 5, post 7/10. Читання власних багаторічних даних: мінімальний набір для ретроспективного аналізу, п'ять сигналів видимих лише на горизонті кількох блоків (сезонні патерни, тренд RPE, тренд відновлення, adherence тренд, циклічні провали), квартальний і річний ретро-протокол. Halson 2014. clinical-safety: PASS.
- `content/post-1398-koly-perebudovuvaty-systemu.md` — Cluster 5, post 8/10. Коли перебудовувати систему: різниця між блоковою корекцією і системною перебудовою. Чотири ознаки системної зміни. Що НЕ є підставою. Чотири кроки системної перебудови. Issurin 2010, Kiely 2012. clinical-safety: PASS.
- `content/post-1399-stiykist-i-poslidovnist-yak-holovna-zminna.md` — Cluster 5, post 9/10. Adherence як дизайн системи, не мотивація. П'ять елементів дизайну: фіксований час/місце, мінімальне зобов'язання як підлога, реалістичний план, зниження когнітивних витрат, вбудована варіабельність. Принцип «не пропускати двічі поспіль» як структурний. Wood & Neal 2007, Fogg 2020. clinical-safety: PASS.
- `content/post-1400-syntez-bloku-1351-1400.md` — **Block synthesis** · Cluster 5, post 10/10. Синтез блоку 1351–1400: огляд п'яти кластерів + десять принципів атлета що тренується роками в реальних умовах. Відкрите питання для наступного блоку (product-strategist + sports-scientist). clinical-safety: PASS.

### Changed
- `blog.html` — updated to post-1400; nine new entries added (posts 1392–1400) in Long-term Progress category.
- `STATUS.md` — Block 1351–1400 marked COMPLETE 50/50; Cluster 5 COMPLETE; Block 1401–1450 TBD added to next priorities.
- `VERSION` — 6.25.1 → 6.26.0.

---

## [6.25.1] — 2026-06-19

### Changed
- **`docs/SOC2_READINESS.md §92`** — Evidence Artefact Cross-Reference Patch: OBSERVABILITY §48 (Google Directory Sync Alert Signal Source · DEC-067 · OQ-SSO-OBS-02). Registers **SSO-OBS-E-007** (CC7.2/CC7.3 — quarterly AL-SSO-GDIR-01 PagerDuty incidents + AL-SSO-GDIR-02 Slack alerts + pg_cron job 35 freshness cross-check; 3yr; `compliance/evidence/sso/`). Closes OBSERVABILITY §48.8 item 4 cross-reference obligation. Adds `form-soc2-evidence/sso/` to §80.3 R2 folder structure. SOC 2 version bumped v3.16.0 → v3.17.0.
- `VERSION` — 6.25.0 → 6.25.1.

---

## [6.25.0] — 2026-06-19

### Added
- **Block 1351–1400 Cluster 5 OPEN** (posts 1391–1400 · Від тактики до стратегії — багаторічна перспектива): post-1391 написано у `content/`:
  - `post-1391-vid-taktyky-do-stratehii-bahatolichne-planuvannia.md` — Cluster 5 opener: чому більшість атлетів думає тактично; чим стратегічне мислення відрізняється від тактичного (чотири конкретних зміщення); дані Hubal et al. 2005 (індивідуальна варіабельність), Peterson, Rhea & Alvar 2004 (хвилеподібна варіація обсягу), Bompa & Haff 2009 (послідовні фази), Androulakis-Korakakis et al. 2020 (мінімальна доза збереження сили); як чотири кластери блоку 1351–1400 формують єдину стратегічну рамку; основна помилка тактичного атлета (між блоками немає осмисленого переходу); як виглядає стратегічний цикл у практиці; чотири зміщення після блоку; структура Кластера 5 (posts 1392–1400). clinical-safety PASS.
- **`blog.html`** — додано картку post-1391, updated to post-1391.

### Changed
- `STATUS.md` — Block 1351–1400: 40/50 → 41/50; Cluster 5 IN PROGRESS · v6.25.0.
- `VERSION` — 6.24.0 → 6.25.0.

---

## [6.24.0] — 2026-06-19

### Added
- **Block 1351–1400 Cluster 4 COMPLETE** (posts 1381–1390 · Програмування при структурних обмеженнях): 10 нових постів у `content/`:
  - `post-1381-strukturni-obmezhennia-ramka.md` — opener: структурні обмеження vs. тимчасові труднощі; три ознаки структурного обмеження; психологічна пастка очікування; нова рамка (від тимчасового режиму до оптимального рішення в межах реального контексту).
  - `post-1382-audyt-vlasnyh-resursiv.md` — чотири вектори аудиту (час фактичний, обладнання стабільно доступне, відновні ресурси, когнітивна ємність); практичний аудит за останні 4 тижні.
  - `post-1383-minimalnyy-efektyvnyy-obsiah.md` — MEV при системних обмеженнях; Bickel (2011): 1/9 об'єму при збереженні інтенсивності достатньо; Ralston (2017): 1x/тижень = 70–80% ефекту 2x/тижень; три рівні при обмеженнях.
  - `post-1384-dvi-sesii-na-tyzhden.md` — 2x/week як самостійна архітектура (не компроміс); McLester (2003) і Schoenfeld (2015) доказова база; full-body vs. upper/lower; збереження інтенсивності.
  - `post-1385-obmezhenyi-dostup-do-obladnannia.md` — ієрархія обладнання за адаптаційним потенціалом; що можна і не можна відтворити без штанги; Calatayud (2015); найбільш проблемна зона — нижня частина тіла.
  - `post-1386-pereoblashtuvannia-prohramy.md` — проєктування від обмежень проти масштабування старої програми; принцип ядро+буфер (ядро 25–35 хв — в кожній сесії завжди); реальний приклад редизайну.
  - `post-1387-shcho-zakhyshchaty-pershym.md` — чотирирівнева ієрархія: якість руху → базова сила → частота стимулу → об'єм і аксесуари; що не можна компромісувати за будь-яких умов.
  - `post-1388-sezony-obmezhen.md` — ретроспективний аудит рокового ритму; прив'язка фаз (прогрес/підтримка/мінімум) до передбачуваних сезонів; proactive перехід до мінімуму; Plisk & Stone (2003).
  - `post-1389-pereosmslennia-prohresu.md` — нові метрики (стабільність ±5–10%, послідовність, якість виконання, структурне здоров'я); переосмислення від «чи б'ю PR» до «чи ефективно використовую реальний ресурс».
  - `post-1390-syntez-klastera-4.md` — синтез кластера: три принципи; перехід до Cluster 5.
- **`blog.html`** — додано 10 нових карток (posts 1381–1390), updated to post-1390.

### Changed
- `STATUS.md` — Block 1351–1400: 30/50 → 40/50; Cluster 4 COMPLETE · v6.24.0; Cluster 5 TBD (синтез блоку).

---

## [6.23.1] — 2026-06-19

### Added
- **`docs/OBSERVABILITY.md §48`** (v4.4.0 → v4.5.0) — OQ-SSO-OBS-02 resolution: Google Directory sync alert signal source adopted as DEC-030 Supabase `audit_log_events` queries (DEC-067). New pg_cron job 35 (`google_directory_alert_check`, `*/5 * * * *`, role `form_audit`) implements AL-SSO-GDIR-01 (error rate >20% per tenant → PagerDuty P2 via form-alert-relay, 30-min dedup cooldown) and AL-SSO-GDIR-02 (P95 latency >3000ms per tenant → Slack #alerts-sso P3). Cloudflare Analytics Engine migration deferred to >50 active GD tenants or job 35 P95 >500ms for 7 consecutive days (four-step review protocol). New SOC 2 evidence artefact SSO-OBS-E-007 (CC7.2/CC7.3, quarterly, `compliance/evidence/sso/SSO-OBS-E-007_<YYYY-QN>.csv`, first filing M11). §48.7 OQ gap tracker: OQ-SSO-OBS-02 🟢 Resolved, OQ-SSO-OBS-01 🟡 Open P1 (unchanged). §48.8 implementation checklist: 5 items (P0/M4 pg_cron deploy, P0/M4 PagerDuty route, P1/M4 AUDIT_LOG_SCHEMA advisory, P1/M11 SOC2_READINESS §79.4 cross-ref, P1/M10 calendar entry).
- **`docs/DECISION_LOG.md` DEC-067** (2026-06-19) — OQ-SSO-OBS-02 resolution: DEC-030 Supabase queries for AL-SSO-GDIR-01/02 at M4; Analytics Engine deferred (trigger: >50 active GD tenants OR job 35 P95 >500ms for 7 days). Owner: devops-lead + platform-engineer + compliance-officer. Five grounds: zero marginal cost at current scale, adequate scale math, audit trail coherence, pattern consistency with DEC-043/044/051/053/065, concrete measurable migration trigger.

### Changed
- `docs/OBSERVABILITY.md §26.13` — OQ-SSO-OBS-02 row marked 🟢 Resolved (DEC-067, 2026-06-19); Analytics Engine migration trigger appended inline.

---

## [6.23.0] — 2026-06-19

### Added
- `content/post-170-five-thinking-traps-self-coached-athlete.md` — Editorial 170 (series 161–180, synthesis 166–170). «П'ять пасток мислення, які зупиняють атлета більше, ніж погана програма»: (1) пошук оптимальної програми як заміна тренуванню; (2) DOMS/помп/тривалість як метрика якості сесії; (3) оцінка блоку за першими двома тижнями; (4) чужий прогрес як точка відліку; (5) програма як священний текст замість гіпотези. Механізм і вихід для кожної. 5-крокий мінімальний протокол (базова лінія → зобов'язання → журнал → ретроспектива → повтор). Ralston 2017; Schoenfeld 2016; Damas 2016; Hubal 2005; Zatsiorsky 2006. clinical-safety: NOT_REQUIRED. sports-scientist review required before publish.

### Changed
- `blog.html` — post-170 card додано (першим у списку)
- `STATUS.md` — Editorial series розширено до 11–37, 44–50, 106–122, 170, 1361; post-170 written v6.23.0

---

## [6.22.0] — 2026-06-19

### Added
- `content/post-1372-syhnaly-khronichnoho-navantazhennia.md` — Block 1351–1400 Cluster 3, post 2/10. Сигнали хронічного навантаження: розмежування functional overreaching / non-functional overreaching / OTS (Meeusen et al. 2013). Три рівні сигналів — тренувальні (RPE дрейф, втрата мотивації, скутість), відновні і системні (HRV, сон, дратівливість), та ті що вимагають медичної консультації. Таблиця: нормальна адаптаційна втома vs. накопичене перевантаження. Чому тренований атлет пропускає сигнали (нормалізація, атрибуція зовнішнім факторам, конфлікт ідентичності). Протокол моніторингу — три запитання щотижня. clinical-safety: PASS.
- `content/post-1373-sukhozyllia-zviazky-na-rokakh.md` — Block 1351–1400 Cluster 3, post 3/10. Сухожилля і зв'язки на роках: асиметрія адаптації м'язів (2–4 тижні) vs. сухожилль (3–6 місяців). Що відбувається при тривалому тренуванні (жорсткість, переріз, колагеновий оборот, регіональна специфіка). Модель сухожильного континууму Cook & Purdam (2009). Вікно синтезу колагену і практичні наслідки (ексцентрика, частота 2–3×/тиждень). Три рівні управління: профілактика, реакція на перші ознаки, хронічний дискомфорт. clinical-safety: PASS.
- `content/post-1374-stratehichni-delovantazhi-trenovanyi-atlet.md` — Block 1351–1400 Cluster 3, post 4/10. Стратегічні деловантажі: чому стандартна модель 3+1 не масштабується на пізні роки. Три типи деловантажу: Тип 1 об'ємний (5–7 днів), Тип 2 структурний (10–14 днів), Тип 3 підтримуюча фаза (2–6 тижнів). Проактивне vs. реактивне планування. Сигнали для позачергового деловантажу. П'ять поширених помилок. Базова річна структура деловантажів. clinical-safety: PASS.
- `content/post-1375-koryvuvannia-priorytetiv-trenuvalnyy-vik-1.md` — Block 1351–1400 Cluster 3, post 5/10. Коригування пріоритетів частина 1: що залишається незмінним (прогресивне навантаження, базові рухи, техніка, специфічність). Шість зон коригування: від максимальної до оптимальної гіпертрофії (MEV/MRV, Israetel 2019); хвилеподібна інтенсивність для управління сухожильним накопиченням; оптимальна частота (2× на тиждень, Schoenfeld 2016); структурний баланс замість довільних допоміжних рухів; проактивне відновлення; якісні метрики. clinical-safety: PASS.
- `content/post-1376-novi-metryky-uspikhu-treningova-yakist.md` — Block 1351–1400 Cluster 3, post 6/10. Нові метрики успіху: чому PR — слабка метрика для пізніх років. Шість категорій метрик: якість відновлення (HRV, готовність), технічна стабільність під втомою, стабільність RPE у мезоциклі, суперкомпенсаційний ефект після деловантажу, структурне здоров'я (щотижневий самозвіт), багаторічна тенденція. Мінімальний практичний протокол відстеження. clinical-safety: PASS.
- `content/post-1377-chytaty-dovhostrokovu-dynamiku-dani-1.md` — Block 1351–1400 Cluster 3, post 7/10. Довгострокові дані частина 1: чому власний журнал — інструмент прогнозування, а не архів. Три рівні аналізу: трендовий (3–6 місяців), циклічний (6–18 місяців), відновний профіль (12–36 місяців). Сезонна динаміка і повторювані патерни. Типові помилки аналізу. Мінімальний журнал для значущого аналізу. clinical-safety: PASS.
- `content/post-1378-chytaty-dovhostrokovu-dynamiku-dani-2.md` — Block 1351–1400 Cluster 3, post 8/10. Довгострокові дані частина 2: патерн накопичення — три фази в даних. П'ять причин плато (накопичення, недонавантаження, стеля руху, технічний дрейф, зовнішній фактор) і їхні ознаки в даних. Прогнозування на основі особистого профілю. Щоквартальний алгоритм аналізу (30–45 хвилин, 5 кроків). Межі самостійного аналізу. clinical-safety: PASS.
- `content/post-1379-balans-prohres-zberezhennia.md` — Block 1351–1400 Cluster 3, post 9/10. Баланс прогресу і збереження: дві цілі не конкурують у просторі, а розподілені в часі. Наука збереження: Bickel et al. (2011) — 1/9 від об'єму при збереженні інтенсивності достатньо для підтримки адаптацій. П'ять конкретних контекстів для фази збереження. Практична структура фази (об'єм, інтенсивність, частота, тривалість). Різниця між деловантажем і фазою збереження (таблиця). Психологія «менше» і когнітивна реструктуризація. clinical-safety: PASS.
- `content/post-1380-syntez-klastera-3-atletychne-dovholittia.md` — Block 1351–1400 Cluster 3, post 10/10 (синтез). Огляд усіх 10 постів кластера. Три головних принципи: (1) горизонт оцінки масштабується з досвідом; (2) відновний ресурс є первинною змінною; (3) структурне здоров'я — непремінна умова. Одна дія для запуску проактивного управління. Пропозиція теми Cluster 4 (1381–1390): програмування при структурних обмеженнях — на затвердження product-strategist + sports-scientist. clinical-safety: PASS.

### Changed
- `blog.html` — posts 1372–1380 додано; загальна кількість постів оновлена до 1380
- `STATUS.md` — Block 1351–1400 Cluster 3: **COMPLETE 10/10 · v6.22.0**; наступний пріоритет: Cluster 4 (1381–1390, тема на затвердження)

---

## [6.21.0] — 2026-06-19

### Added
- **`docs/SSO_SCIM_IMPLEMENTATION.md §34` — SCIM Bulk Deprovision Guard** (v2.5 → v2.6): Preventive per-tenant configurable SCIM bulk deprovision threshold guard (default 20%, range 5–100%, 5-minute rolling KV window). Closes OQ-R24-01 (P1, INCIDENT_RESPONSE.md R-24.14). DEC-066 adopted. Key additions: `enforceDeprovisionGuard()` TypeScript Worker function; `getGuardConfig()` with 60s KV cache; `revokeActiveOverride()` auto-revocation after first override use; migration `0079_bulk_deprovision_guard.sql` (3 columns, 2 constraints, 1 index, pg_cron job 33); `update_bdg_threshold()` Supabase SECURITY DEFINER RPC; `POST /internal/v1/admin/scim/bulk-override` CSM PAM endpoint; Admin Dashboard "Bulk Deprovision Guard" panel; GUARD-E-001 SOC 2 evidence artefact (CC6.3/A1.2/CC7.2/CC9.2); R-24/guard interaction table; §34.11 implementation checklist.
- **`docs/AUDIT_LOG_SCHEMA.md`** (v2.18 → v2.19): +5 DEC-030 HMAC-chained events — `scim.bulk_deprovision_blocked` (HIGH 7yr), `scim.bulk_deprovision_override_issued` (HIGH 7yr), `scim.bulk_deprovision_override_used` (HIGH 7yr), `scim.bulk_deprovision_threshold_updated` (STANDARD 7yr), `system.scim_guard_repeated_trigger` (STANDARD 1yr advisory, GUARD-CHAIN-01). New section inserted before `### Security & tenant isolation events`.
- **`docs/DECISION_LOG.md` DEC-066** (2026-06-19): OQ-R24-01 resolution — per-tenant SCIM bulk deprovision threshold (default 20%, range 5–100%), 5-minute rolling window KV counter, CSM-countersigned time-limited override (1h TTL, auto-revoked after first use). Owner: enterprise-architect + platform-engineer + compliance-officer + customer-success.
- **`docs/INCIDENT_RESPONSE.md` R-24.15 item 10** (2026-06-19): Marked `[x] Done` — OQ-R24-01 resolved via DEC-066; implementation spec in `docs/SSO_SCIM_IMPLEMENTATION.md §34`.
- **`docs/SOC2_READINESS.md §91`** (v3.15.0 → v3.16.0): Evidence Artefact Cross-Reference Patch for SSO_SCIM §34. Registers GUARD-E-001 (CC6.3/A1.2/CC7.2/CC9.2 — quarterly SCIM Bulk Deprovision Guard event export, `compliance/evidence/scim-guard/GUARD-E-001_<YYYY-QN>.csv`, first filing M14). Collection SQL, SOC 2 criteria mapping, cross-reference obligations closure table, six-item implementation checklist.

## [6.20.0] — 2026-06-19

### Added
- **Block 1351–1400 Cluster 3 opener** — пост 1371: «Що змінюється після п'яти років тренування — і чому рамка «більше, важче, далі» перестає працювати» — Кластер 3 (Атлетичне довголіття і управління накопиченим навантаженням). Три компоненти атлетичного капіталу (структурний, нейром'язовий, психологічний); накопичене навантаження на горизонті років (хронічна тканинна втома — Maganaris et al. 2004; компенсаційні патерни; психологічна втома — Meeusen et al. 2013); чому рамка «більше, важче, далі» деградує у тренованого атлета (Zatsiorsky & Kraemer 2006; Schoenfeld & Grgic 2019); нова операційна рамка — управління накопиченим навантаженням; анонс постів 1372–1380. clinical-safety: PASS. sports-scientist review required before publish.
- `blog.html` — картка post-1371 додана (Cluster 3 opener).
- `STATUS.md` — Block 1351–1400: Cluster 3 IN PROGRESS (1/10), post-1371 додано.
- `VERSION` — 6.19.0 → 6.20.0.

## [6.19.0] — 2026-06-19

### Added
- **Block 1351–1400 Cluster 2 COMPLETE 10/10** — тема: Реальні умови і багаторічна послідовність (posts 1361–1370). clinical-safety: PASS all. sports-scientist review required before publish.
  - post-1362 — «Стрес поза залом і тренувальний об'єм: чому життєве навантаження — тренувально-релевантний вхід»: загальний адаптаційний синдром (Сельє); алостатичне навантаження (McEwen 1998); спільна адаптаційна ємність для тренувального і позатренувального стресу; три режими масштабування (A стандартний / B знижений / C мінімальний); як читати HRV, суб'єктивну готовність і сон; проактивне vs. реактивне масштабування.
  - post-1363 — «Мінімально необхідний тренувальний тиждень: що реально зберігає прогрес, коли часу мало»: дослідження Krieger (2010), Ralston (2017), Androulakis-Korakakis (2020) — одна якісна сесія на тиждень зберігає силу 16 тижнів; чому інтенсивність важливіша за об'єм при компресії; два мінімальних форматів (один великий день / два скорочених дні); ієрархія виключень; перефреймування мінімального тижня як стратегічної консервації.
  - post-1364 — «Адаптація програми до доступного інвентарю: як зберегти стимул без ідеальних умов»: ціль вправи vs. сама вправа; ієрархія замін: повна → часткова → інший механічний профіль → альтернатива; матриця замін для squat/hinge/horizontal press/vertical press/horizontal pull/vertical pull; чого не робити при адаптації; тимчасова заміна не порушує довгострокової специфічності.
  - post-1365 — «Детренування і повернення: що насправді втрачається, за який час, і як правильно відновлювати»: нейром'язові vs. структурні адаптації — різний часовий профіль; конкретні терміни (Mujika & Padilla 2001, 2000); до 2 тижнів — мінімальні втрати; 4–8 тижнів — 8–12% від піку сили; м'язова пам'ять: ядра міосателітних клітин і нейронні патерни; структура блоку повернення за тривалістю паузи; таблиця рекомендацій.
  - post-1366 — «Фазова логіка тренувальних пріоритетів: як адаптувати напрямок під зміну обставин»: три категорії (первинний / вторинний / підтримка); класичні тригери — підвищення зайнятості, батьківство, переїзд, сімейні обставини; три запитання для налаштування; помилки перфекціонізму, накопиченого «боргу», ігнорування змін; фазова логіка як частина атлетичного шляху, не відхилення від нього.
  - post-1367 — «Ідентичність атлета на роках: як залишатись у тренуванні, коли зовнішня мотивація зникає»: теорія самодетермінації Deci & Ryan; від зовнішньої до внутрішньої мотивації; goal-based vs. process-based орієнтація (Ntoumanis & Biddle 1999); ідентичність як структурний рівень (Clear 2018); чому послідовність — не про силу волі; практичний аудит мотивації — три запитання.
  - post-1368 — «Дизайн середовища для систематичності: видалення тертя і формування стандарту»: що таке «тертя» і чому воно важливіше за ентузіазм; часова точка, логістика, рішення «тренуватись за замовчуванням»; формування стандарту: 66 днів за Lally et al. (2010); три середовищні рівні — мікросередовище, розклад, соціальний контекст; адаптація при зміні середовища.
  - post-1369 — «Жива програма: як вносити зміни без втрати послідовності»: що таке program hopping і чому він шкідливий; три категорії змін — операційні (вільно), структурні (після аналізу), заміна програми (тільки між блоками); критерії для кожної категорії; різниця між «зміна» і «ескейп» — перевірочне питання; архітектура живої програми з фіксованими і гнучкими елементами.
  - post-1370 — «Синтез Кластера 2: реальні умови і багаторічна послідовність — зведена рамка»: дев'ять принципів кластера (adherence, масштабування, мінімальний тиждень, патерн-заміни, м'язова пам'ять, фазова логіка, ідентичність, середовище, жива програма); центральна ідея: реальні умови є умовами тренування; зв'язок з Кластером 1 (архітектура + виконання); анонс Кластера 3 (атлетичне довголіття і управління накопиченим навантаженням на роках).
- `blog.html` — картки post-1362–1370 додані (Block 1351–1400 Cluster 2 COMPLETE · Real conditions & Long-term Consistency).
- `STATUS.md` — Block 1351–1400 оновлено: Cluster 2 COMPLETE 20/50; Cluster 3 TBD.
- `VERSION` — 6.18.1 → 6.19.0.

## [6.18.1] — 2026-06-19

### Changed
- `docs/INCIDENT_RESPONSE.md` — v2.7 → v2.8. Two cross-reference patch closures: (1) §1 IC Quick-Start addendum — concurrent incident coordination line added per §18.7 item 3 (P1/M7, overdue): "Two active incidents? → Confirm distinct IDs in #incidents. Separate channels, separate ICs for ≥ P1. (§18.4)"; closes DEC-053 documentation obligation. (2) R-20 §C3 PAM_TELEMETRY procedural note — blockquote added after Art. 9 exposure SQL per `docs/OBSERVABILITY.md §29.12.4` item 4 (P1/M4, overdue): documents §29.12.2 investigation procedure control (UUID queries against `PAM_TELEMETRY` require active PagerDuty incident; findings added to incident timeline; `cf-ae-pam-read` token access restriction). §18.7 item 3 marked [x] Done.
- `docs/OBSERVABILITY.md` — §29.12.4 item 4 marked [x] Done: PAM_TELEMETRY investigation procedure note confirmed added to `docs/INCIDENT_RESPONSE.md` R-20 §C3. Document version v4.4.0 unchanged (patch-level status update).
- `VERSION` — 6.18.0 → 6.18.1.

## [6.18.0] — 2026-06-19

### Added
- **post-1361** — «Систематична послідовність як стратегічна перевага: чому adherence перевищує оптимальність на п'ятирічному горизонті» — відкриває Cluster 2 (posts 1361–1370 · Реальні умови і багаторічна послідовність) у Block 1351–1400. Efficacy vs. effectiveness на довгому горизонті; компаундинг послідовності; три причини втрати послідовності; МЕД як стратегічний інструмент; програмування від поверху; протокол форс-мажору. Bickel 2011, Ralston 2017, Androulakis-Korakakis 2020, Peterson 2011. clinical-safety: PASS. sports-scientist review required before publish.
- `blog.html` — картка post-1361 додана (Block 1351–1400 Cluster 2 opener · Long-term Progress).
- `STATUS.md` — Block 1351–1400 Cluster 2 IN PROGRESS; editorial series розширено до post-1361.
- `VERSION` — 6.17.0 → 6.18.0.

## [6.17.0] — 2026-06-19

### Added
- **Block 1351–1400 Cluster 1 COMPLETE 10/10** — тема блоку: Довгострокова стратегія прогресу та атлетичне довголіття (Long-term Progress Strategy & Athletic Longevity). Cluster 1: Фундамент багаторічного планування (posts 1351–1360). sports-scientist review required before publish. clinical-safety: NOT_REQUIRED (чиста методологія програмування).
  - post-1351 — «Чому 12-тижневий блок — це тактика, а не стратегія»: розмежування тактики (блок) і стратегії (напрямок на роки); що змінюється у виборі вправ, інтерпретації плато, накопиченні об'єму при мисленні роками.
  - post-1352 — «Атлетична кар'єра як структура: фази розвитку і переходи між ними»: три фази — загальний розвиток (0–2 тр. роки), спеціалізація (2–6), майстерність (6+); маркери переходів; чому більшість пропускає фазу 1 надто швидко. Refs: Bompa (2009), Zatsiorsky (2006), LTAD.
  - post-1353 — «Модель довгострокового розвитку сили: від загального до специфічного»: принцип зміщення відношення загальна/специфічна робота з тренувальним віком; що це означає для вибору вправ на роки. Refs: Bompa, Zatsiorsky.
  - post-1354 — «Накопичена адаптація проти накопиченої втоми: як читати довгострокову динаміку»: паралельне накопичення адаптацій і втоми; ознаки домінування втоми; роль офф-сезону на масштабі більшому, ніж один блок.
  - post-1355 — «Як змінюються тренувальні пріоритети зі зростанням тренувального віку»: конкретні зрушення пріоритетів по роках — рік 1 (техніка/звичка/об'єм), рік 3 (інтенсивність/варіативність), рік 5+ (специфічність/кумулятивне навантаження).
  - post-1356 — «Структура тренувального року: макроцикл для self-coached intermediate»: три моделі (змагальна/однопікова/безстартова); офф-сезон — функція і тривалість; розподіл фаз (накопичення 45–55%, інтенсифікація 25–35%, реалізація 10–15%). Refs: Bompa & Haff (2009).
  - post-1357 — «Адаптаційний ліміт: механіка уповільнення прогресу і тактика відповіді»: чотири механізми уповільнення; що «плато» означає на різних тренувальних роках; три стратегії виходу (варіація, блокова periodization, маніпуляція об'ємом).
  - post-1358 — «Планування із зворотнього відліку: 2–3 роки вперед без жорсткого скрипту»: техніка backward planning; карта фаз і критерії переходів; парадокс довгострокового планування (чим далі горизонт — тим менше деталей).
  - post-1359 — «Маркери, що сигналізують про необхідність стратегічних змін»: п'ять стратегічних сигналів (три блоки без прогресу, хронічний RPE-дрейф, вправи без стимулу, goal-тип мismatch, незадоволеність незалежно від результатів); чим відрізняються від тактичних.
  - post-1360 — «Синтез кластера 1: фундамент багаторічного планування»: шість ідей; таблиця «блокове мислення vs. багаторічна перспектива»; preview Cluster 2 (TBD product-strategist + sports-scientist).
- `blog.html` — updated to post-1360; 10 нових записів категорії Long-term Progress.
- `README.md` — editorial posts 51 (reading-training-data) і 52 (RPE calibration) оновлено зі статусу `proposed` → `draft · v6.14.0 / v6.15.0`.

## [6.16.1] — 2026-06-19

### Changed
- `docs/ENTERPRISE_ONBOARDING.md` — §3.3 SCIM provisioning: added SCIM IP enforcement note (self-hosted proxy customers only) after the audit events table, per `docs/SSO_SCIM_IMPLEMENTATION.md §33.2.4` (DEC-062). Covers `scim_ip_enforcement_enabled` toggle path, `scim.ip_allowlist_blocked` DEC-030 events, enable-only-after-first-sync gate, and explicit cloud-IdP exclusion. Closes SSO §33.7 item 5 (P1/M9).
- `docs/SSO_SCIM_IMPLEMENTATION.md` — §17.2 Q-05 row extended with SCIM IP enforcement discovery step per §33.2.2: self-hosted SCIM proxy identification, `scim_ip_enforcement_enabled` Admin Dashboard path, enablement-deferred-until-post-sync gate. §33.7 items 5 and 6 marked [x] Done (2026-06-19). Document header v2.5 unchanged (patch-level cross-reference closure).
- `docs/OBSERVABILITY.md` — §47.11 item 7 marked [x] Done: SIEM-CONSENT-E-001 was registered in `docs/SOC2_READINESS.md §88` (v3.13.0, 2026-06-18) and §79.4/§80.3 (v3.13.1, 2026-06-18); checklist status now reflects the completed work. Document version v4.4.0 unchanged (patch-level status update).
- `VERSION` — 6.16.0 → 6.16.1.

## [6.16.0] — 2026-06-19

### Added
- **Block 1301–1350 COMPLETE 50/50** — тема: Змагальна підготовка та пікування (Competition Preparation & Peaking). Повний блок з 5 кластерів, 50 постів. sports-scientist review required before publish.
  - **Cluster 1 COMPLETE** (posts 1301–1310 · Що таке пікування і навіщо воно потрібне): фізіологія пікування (адаптація vs. вираження сили), чому hard до старту не працює, часовий горизонт 12/8/4 тижні, переключення з накопичення на реалізацію, обсяг vs. інтенсивність у піковому мезоциклі, специфічність (одяг/команди/пауза), відбір спроб, тейпер-парадокс, помилки першого старту, синтез.
  - **Cluster 2 COMPLETE** (posts 1311–1320 · Пікування: протоколи та маніпуляції): три моделі тейперу (лінійна/ступінчата/зворотна), тривалість 1-2-3 тижні, збереження частоти, інтенсивність RPE 8-9, управління втомою (периферична vs. нейронна), допоміжні вправи у піковому блоці, остання сесія перед стартом, тиждень змагань, три помилки тейперу, синтез.
  - **Cluster 3 COMPLETE** (posts 1321–1330 · День змагань — виконання та управління): структура розминки на змаганнях, розрахунок розминочних спроб відносно opener, відкривальна спроба як ключове рішення, між спробами (30 хв), другі і треті спроби, тайм-менеджмент, харчування і гідратація дня (clinical-safety: CONDITIONAL PASS), суддівські вимоги, адаптація коли план зривається, чеклист дня.
  - **Cluster 4 COMPLETE** (posts 1331–1340 · Ментальна підготовка до виступу): тривога vs. збудження (Brooks 2014), передзмагальний ритуал, process focus vs. outcome focus, 3-кроковий протокол після невдалої спроби, ефект аудиторії (соціальна фасилітація), self-talk (мотиваційний/інструкційний), перший старт як збір даних, конкурентна частота, аналіз після невдачі без самозасудження (clinical-safety: CONDITIONAL PASS), синтез.
  - **Cluster 5 COMPLETE** (posts 1341–1350 · Після змагань: аналіз, відновлення та наступний цикл): фізичне відновлення 72 год, суб'єктивне vs. об'єктивне навантаження після старту, пост-змагальна дезорієнтація (CONDITIONAL PASS), технічний аналіз відео, стратегічний аналіз рішень, перехід до офф-сезону, перший мезоцикл після старту, постановка нових цілей, сезонна структура A/B/C-старти, синтез блоку.
- `blog.html` — updated to post-1350; 50 нових записів категорії Competition Prep.
- `STATUS.md` — Block 1301–1350 COMPLETE, next: Block 1351–1400 (тема TBD).
- `VERSION` — 6.15.0 → 6.16.0.

### Clinical safety notes
- post-1327 (харчування дня змагань): CONDITIONAL PASS — без вагових маніпуляцій, загальні принципи підтримки ефективності.
- post-1339 (аналіз після невдачі): CONDITIONAL PASS — спортивний framing, не клінічний; рекомендація спортивного психолога при тривалому дистресі.
- post-1343 (пост-змагальна дезорієнтація): CONDITIONAL PASS — post-competition blues у спортивному контексті, не клінічна депресія.
- posts 1331–1340 загалом: PASS — конкурентна тривога у спортивному контексті, без клінічних термінів.


## [6.15.0] — 2026-06-19

### Added
- `content/post-52-rpe-calibration.md` — Editorial series Block 51–100, post 2/50. «RPE як мова: чому «8 з 10» значить різне для кожного — і що з цим робити»: три типи калібраційних помилок (занижена, завищена, непослідовна RPE), чому досвід/страх відмови/маскування втоми спотворюють шкалу, протокол рекалібрування (підхід до відмови раз на блок, дебрефінг, якірні точки), коли RPE-авторегуляція перевершує %1RM і коли ні, як Victor відстежує RPE-вагові траєкторії в часі. Посилання: Zourdos et al. 2016 (RIR-based RPE), Colado et al. 2012 (RPE overestimation у новачків). clinical-safety PASS.

### Changed
- `blog.html` — post-52 card added (Editorial series · Training science · before post-51).
- `README.md` — posts 886–890 added to Block 851–900 roadmap table (proposed · v6.15.0): RPE-calibration, mobility as training variable, exercise selection without coach, when difficulty = adaptation vs. programming error, athlete maturity markers.
- `STATUS.md` — row 51–99 updated: post-52 added v6.15.0, clinical-safety PASS, sports-scientist review pending; count 49 → 50.
- `VERSION` — 6.14.1 → 6.15.0.


## [6.14.1] — 2026-06-19

### Changed
- `docs/AUDIT_LOG_SCHEMA.md` — v2.18. Five new DEC-030 HMAC-chained events registered in new `### Enterprise Adoption Monitoring events (DEC-030 HMAC-chained · COST_MODEL §40.8 · CC4.1/CC2.2/CC7.3/CC4.2/A1.1)` section inserted before `## Export & delivery`. (1) `enterprise.adoption_snapshot_filed` (STANDARD, 3yr): monthly `evidence-collection-cron` INSERT record for `enterprise_adoption_snapshots`; Zod v2 `AdoptionSnapshotFiledPayload`; no `user_id`, `coaching_engaged_seats < 5` suppressed at API layer. (2) `enterprise.adoption_milestone_reached` (STANDARD, 3yr): once-per-milestone-per-tenant threshold crossing record; milestone enum `s1_activation_40pct`/`s2_wau_30pct`/`s3_habitual_20pct`/`s2_wau_40pct_green`; idempotency enforced. (3) `enterprise.adoption_health_downgraded` (HIGH, 3yr): emitted on band worsening (Green→Amber, Green→Red, Amber→Red); ADO-CHAIN-01 monitoring invariant specified — `system.csm_followup_overdue` advisory emitted by weekly cron if no CSM follow-up within 10 business days. (4) `enterprise.qbr_completed` (STANDARD, 3yr): CSM manual trigger via Admin Console; QBR-PRIV-01 chain invariant: `privacy_floor_verified: z.literal(true)` required — HTTP 422 if absent or false; `renewal_date` date-only, no attendee names. (5) `system.csm_followup_overdue` (LOW, 1yr): ADO-CHAIN-01 advisory; `downgrade_event_id` FK to triggering chain record; non-blocking; compliance-officer SIEM review queue. Retention table: +6 rows (adoption_snapshot_filed 3yr CC4.1/A1.1; adoption_milestone_reached 3yr CC4.1; adoption_health_downgraded 3yr CC7.3/CC4.2; qbr_completed 3yr CC2.2/CC4.1; csm_followup_overdue 1yr advisory). SOC 2 auditor narratives for CC4.1, CC2.2, CC7.3, CC4.2, A1.1. Evidence artefacts ADO-E-001/002/003 cross-referenced (already registered in `docs/SOC2_READINESS.md §90`). Document header v2.17 → v2.18. compliance-officer + security-engineer + enterprise-architect + customer-success.
- `docs/COST_MODEL.md` — §40.10 checklist item 1 marked [x] Done (documentation — AUDIT_LOG_SCHEMA.md v2.18, 2026-06-19); implementation gates (migration 0078, evidence-collection-cron extension, Admin Console modal, integration test) tracked in items 2–4.
- `VERSION` — 6.14.0 → 6.14.1.

## [6.14.0] — 2026-06-19

### Added
- `content/post-51-reading-training-data.md` — Editorial series Block 51–100, post 1/50. «Як читати власні тренувальні дані — і не робити хибних висновків»: три структурні помилки self-coached атлета (рішення за точкою замість тренду; порівняння з учора замість 8-тижневого горизонту; плутанина кореляції і причини), що насправді треба читати в даних, і як Victor це робить без ручного аналізу. Посилання: Lukas et al. 2021 (HRV trend vs. single measure), Helms et al. 2014 (4–8 тижнів мінімум для оцінки прогресу). clinical-safety PASS.

### Changed
- `blog.html` — post-51 card added (Editorial series · Training science · between post-106 and post-50).
- `README.md` — posts 881–885 added to editorial roadmap table (draft · v6.14.0): concurrent strength+hypertrophy, testing 1RM, exercise order, HRV as programming tool, volume tolerance across training career.
- `STATUS.md` — row 51–99 updated: post-51 added v6.14.0, clinical-safety PASS, sports-scientist review pending.
- `VERSION` — 6.13.0 → 6.14.0.

## [6.13.0] — 2026-06-19

### Added
- `content/post-1291-why-single-metric-decisions-fail.md` — Block 1251–1300, Cluster 5, post 1/10. Чому одна метрика не може бути підставою для тренувального рішення: будь-яка метрика вимірює один вимір у багатовимірному просторі; приклад-розбір (низький HRV при зеленому суб'єктивному, нормальному сні і недобраному тижневому об'ємі); HRV чутливий до нетренувальних чинників, ЧСС реагує з затримкою, Sleep Score — агрегат, суб'єктивне притуплюється при хронічному перевантаженні; мінімальна система — чотири потоки. clinical-safety: NOT_REQUIRED.
- `content/post-1292-four-data-streams-roles.md` — Block 1251–1300, Cluster 5, post 2/10. Чотири потоки даних і що кожен вимірює насправді: HRV (стан ВНС, горизонт 1–2 доби, чутливий до протоколу і нетренувальних стресорів); сон (тривалість + суб'єктивна якість, тижневий кумулятивний тренд); суб'єктивне (найбільш інформаційно насичений сигнал, записується до трекера, притуплюється при хронічному перевантаженні); тренувальний об'єм (причина, а не наслідок, тижневий горизонт). Зведена таблиця тимчасових горизонтів і слабких місць. clinical-safety: NOT_REQUIRED.
- `content/post-1293-training-volume-fifth-stream.md` — Block 1251–1300, Cluster 5, post 3/10. Тренувальний об'єм як окремий потік: два формати — тижневий тоннаж (кг × повтори) і ACWR (acute:chronic ratio, коридор 0.8–1.3; > 1.5 = spike-обмеження незалежно від HRV); чому без контексту об'єму дані трекера втрачають інтерпретаційну рамку; мінімальний підхід без складних таблиць; два правила-сигнали (тиждень > попереднього на > 20%, два тижні росту підряд). clinical-safety: NOT_REQUIRED.
- `content/post-1294-morning-decision-protocol.md` — Block 1251–1300, Cluster 5, post 4/10. Протокол ранкового рішення: 4 кроки за 3 хвилини — суб'єктивна оцінка до відкриття трекера (ефект якоря реальний), дані трекера (rolling HRV, тривалість сну, Recovery Score), контекст об'єму раз на тиждень, рішення за таблицею зон. Таблиця рішень (зелено/жовто/червоно) і правила ієрархії при конфліктах. Мінімальний денний і тижневий формат запису. clinical-safety: NOT_REQUIRED.
- `content/post-1295-when-signals-conflict.md` — Block 1251–1300, Cluster 5, post 5/10. Коли сигнали суперечать один одному: чотири типи конфліктів — низький HRV при хорошому суб'єктивному (найпоширеніший; можливі нетренувальні причини; правило помірного тренінгу); нормальний HRV при поганому суб'єктивному (суб'єктивне ≤4 завжди перемагає); хороший сон при поганому суб'єктивному (соціальний джетлаг, інерція сну); зелені дані при важкому тренінгу (нерозпізнаний сигнал або конкретний день). Ієрархія пріоритетів — 5 правил. Важливість запису конфліктних рішень для калібровки. clinical-safety: NOT_REQUIRED.
- `content/post-1296-green-yellow-red-zones.md` — Block 1251–1300, Cluster 5, post 6/10. Трирівнева зонова система щоденних рішень: зона визначається за найнижчим сигналом (не середнім); зелена — повний план, не пушити на рекорд; жовта — -20–30% об'єму, планова або -5–10% інтенсивність; червона (будь-який критичний сигнал або два жовтих одночасно) — відпочинок або легка активність; спеціальний кейс «два жовтих = червона»; таблиця порогів по кожній метриці; калібровка після 4–6 тижнів. clinical-safety: NOT_REQUIRED.
- `content/post-1297-weekly-trend-audit.md` — Block 1251–1300, Cluster 5, post 7/10. Тижневий аудит — стратегія (на відміну від щоденного протоколу — тактики): 5 запитань (HRV тренд тижня, тренд сну, середнє суб'єктивне, об'єм і ACWR, кореляція зон і якості тренінгу); типові помилки по об'єму (недобір → spike → кумуляція); формат запису 5 хвилин; правило двох/трьох жовтих тижнів → deload. clinical-safety: NOT_REQUIRED.
- `content/post-1298-common-integration-mistakes.md` — Block 1251–1300, Cluster 5, post 8/10. Шість поширених помилок: HRV-детермінізм (один провал скасовує план); ігнорування суб'єктивного при зеленому HRV; порівняння свого HRV з чужим (абсолютні значення непорівнянні між людьми); ігнорування об'єму як контексту; дата-паралізація (аналіз замість дії); відсутність особистої калібровки. Захист — письмовий протокол рішення. clinical-safety: NOT_REQUIRED.
- `content/post-1299-individual-calibration.md` — Block 1251–1300, Cluster 5, post 9/10. Індивідуальна калібровка системи: три фази — базове спостереження тижні 1–4 (без змін рішень, лише збір даних), зіставлення з якістю тренінгу тижні 5–12 (три ключових питання для пошуку особистих порогів), оновлення таблиці особистих порогів; три ситуації що вимагають повторної калібровки (зміна програми, пристрою, режиму); що не треба оптимізувати навіть при калібровці (ACWR > 1.5, суб'єктивне ≤4). clinical-safety: NOT_REQUIRED.
- `content/post-1300-block-synthesis-1251-1300.md` — Block 1251–1300, Cluster 5, post 10/10. Синтез блоку 1251–1300 «Wearables, data & the self-coached athlete»: підсумки всіх 5 кластерів (HRV, ЧСС, некардіо метрики, сон, інтеграція); зведені таблиці «що трекер вимірює добре / погано / не вимірює взагалі»; мінімальний набір 5 вимірювань для self-coached атлета; головний принцип — «дані — карта, не територія». Блок COMPLETE 50/50. clinical-safety: NOT_REQUIRED.

### Changed
- `blog.html` — posts 1291–1300 added (Cluster 5 · Block 1251–1300 · Wearables & Data). Total posts in blog: 1300.
- `VERSION` — 6.12.1 → 6.13.0.

---

## [6.12.1] — 2026-06-19

### Changed
- `docs/SOC2_READINESS.md` — §90 Evidence Artefact Cross-Reference Patch: COST_MODEL §40 (Enterprise Customer Adoption Economics, v2.6, 2026-06-18). Closes two explicit cross-reference obligations from `docs/COST_MODEL.md §40.10` item 5 and §40.9 (P1/M11). Registers three SOC 2 evidence artefacts: **ADO-E-001** (CC4.1/A1.1 — quarterly `enterprise_adoption_snapshots` fleet health summary export; Q1 fleet-health + Q3 board-ARR-distribution query pair; k-anonymity attestation; 3yr; `compliance/evidence/adoption/ADO-E-001_<YYYY-QN>.csv`); **ADO-E-002** (CC2.2/CC4.1 — quarterly `enterprise.qbr_completed` DEC-030 chain export; `privacy_floor_verified: true` invariant asserted every row; technology control, not procedural; 3yr; `compliance/evidence/adoption/ADO-E-002_<YYYY-QN>.csv`); **ADO-E-003** (CC7.3/CC4.2 — quarterly `enterprise.adoption_health_downgraded` HIGH event export; `system.csm_followup_overdue` advisory cross-check must = 0; zero-event attestation JSON per §90.3; 3yr; `compliance/evidence/adoption/ADO-E-003_<YYYY-QN>.csv`). §80.4 Vanta mirror list updated: ADO-E-001/002/003 added. Document header v3.14.0 → v3.15.0. compliance-officer + customer-success + data-engineer + enterprise-architect.
- `VERSION` — 6.12.0 → 6.12.1.

## [6.12.0] — 2026-06-19

### Changed
- `content/post-11-deload-weeks.md` — rewritten (v2) to match updated blog card title. «Діловий тиждень: чому деловантаження — це тренування, а не капітуляція» — модель суперкомпенсації і fitness–fatigue (Meeusen et al. 2013 ECSS/ACSM; Zatsiorsky & Kraemer 2006); ритміка 4–6 тижнів (Halson 2014); 6 ознак потреби в деловантаженні; чому знижувати обсяг а не інтенсивність (Bickel et al. 2011); Upper/Lower деловантажний тиждень — конкретика; taper vs. регулярне деловантаження; психологія. clinical-safety: NOT_REQUIRED. sports-scientist review required before publish.
- `STATUS.md` — post-11 editorial entry updated v6.12.0.
- `VERSION` — 6.11.0 → 6.12.0.

## [6.11.0] — 2026-06-19

### Added
- `content/post-1281-sleep-staging-psg-vs-ppg.md` — Block 1251–1300, Cluster 4, post 1/10. Стадії сну PSG vs. PPG: що вимірює полісомнографія (ЕЕГ + ЕОГ + ЕМГ) і що вимірює споживчий трекер (PPG + акселерометр). Точність стадіальної класифікації ~58–69% (Chinoy et al., Sleep 2021). N1 систематично занижується, N2 завищується. Тренд корисний, абсолютні хвилини по стадіях — умовні. clinical-safety: NOT_REQUIRED.
- `content/post-1282-sleep-duration-most-reliable-metric.md` — Block 1251–1300, Cluster 4, post 2/10. Тривалість сну — найнадійніша метрика (85–92% точності vs. PSG). Систематичне завищення ~20–40 хв через проблеми з SOL і WASO. Тижневий тренд надійний, щоденна цифра — умовна. clinical-safety: NOT_REQUIRED.
- `content/post-1283-overnight-hrv-vs-morning-hrv.md` — Block 1251–1300, Cluster 4, post 3/10. Нічний HRV vs. ранковий HRV: динаміка протягом ночі, різні вікна у Oura / Whoop / Garmin / Polar, чому між пристроями порівняння неможливе. Rolling average за 7 ночей — практичний сигнал. clinical-safety: NOT_REQUIRED.
- `content/post-1284-rem-detection-consumer-devices.md` — Block 1251–1300, Cluster 4, post 4/10. REM-виявлення: PSG потребує ЕЕГ + ЕОГ + ЕМГ — трекер не має жодного. Точність ~50–65%, систематична плутанина N1 і REM. Чому REM важливий для атлетів і чому конкретна цифра від трекера некорисна. clinical-safety: NOT_REQUIRED.
- `content/post-1285-sleep-score-composite-anatomy.md` — Block 1251–1300, Cluster 4, post 5/10. Анатомія Sleep Score: компоненти і ваги для Oura, Whoop Recovery %, Garmin, Fitbit. Алгоритми тренуються на загальній популяції. «83 від Oura» ≠ «83 від Whoop». Корисно: відхилення від власного базового рівня. clinical-safety: NOT_REQUIRED.
- `content/post-1286-chronotype-training-timing.md` — Block 1251–1300, Cluster 4, post 6/10. Хронотип (~21% генетичний), пік силової продуктивності через 5–7 год після пробудження, соціальний джетлаг як фоновий стрес. Тривалість і стабільність сну важливіші за оптимізацію часу тренування для більшості атлетів. clinical-safety: NOT_REQUIRED.
- `content/post-1287-sleep-restriction-training-performance.md` — Block 1251–1300, Cluster 4, post 7/10. Недосип: одна ніч → -2–8% максимальної сили; хронічне обмеження → гормональний ефект і знижений MPS. Суб'єктивна адаптація небезпечна. Три рівні відповіді: одна ніч / 2–3 ночі / системний недосип. clinical-safety: NOT_REQUIRED.
- `content/post-1288-orthosomnia-sleep-tracking-nocebo.md` — Block 1251–1300, Cluster 4, post 8/10. Ортосомнія (Baron et al., J Clin Sleep Med 2017): тривога через дані трекера як джерело погіршення сну. Поведінкові патерни. Правило 30 хвилин, тижневий огляд. clinical-safety: REQUIRED (review before publish).
- `content/post-1289-sleep-data-training-decisions.md` — Block 1251–1300, Cluster 4, post 9/10. Протокол: три надійні сигнали (rolling HRV × 7, тижнева тривалість, суб'єктивне при пробудженні). Дворежимна система: конгруентність і розбіжність. Суб'єктивне при розбіжності — пріоритет. Victor як контекстний шар. clinical-safety: NOT_REQUIRED.
- `content/post-1290-sleep-tracking-cluster-4-synthesis.md` — Block 1251–1300, Cluster 4, post 10/10 (синтез). Десять висновків кластеру. Що вимірює добре, погано, не вимірює зовсім. Мінімальний набір метрик для силовика. clinical-safety: NOT_REQUIRED.
- `blog.html` — posts 1281–1290 added to blog index (Cluster 4 · Сон — трекер vs. реальність). post-1288 flagged «draft · clinical-safety required» in badge.

### Changed
- `VERSION` — 6.10.2 → 6.11.0.

## [6.10.2] — 2026-06-19

### Changed
- `admin-dashboard.html` — Adoption Health screen added (COST_MODEL §40.10 P1 item 7). New sidebar item «Adoption Health» between Engagement and Reports. Screen implements: S1 Activation / S2 WAU / S3 Habitual stage cards with GREEN/AMBER/RED health band badge; 6-month WAU + Habitual trend chart; coaching engagement panel with k-anonymity gate (coaching_engaged_pct suppressed if cohort < 5 seats); QBR & renewal block (last QBR date, next QBR date, renewal date, privacy_floor_verified attestation note); privacy floor note referencing enterprise_adoption_snapshots. All values aggregated at tenant level — no user_id, no individual data. Acme Corp demo: S1 70% (threshold ≥60% Day 60 ✓), S2 WAU 47% (threshold ≥30% M3 ✓), S3 Habitual 38% (threshold ≥20% M6 ✓) → GREEN band.
- `VERSION` — 6.10.1 → 6.10.2.

## [6.10.1] — 2026-06-19

### Added
- `content/post-38-overreaching-vs-overtraining.md` — Editorial series post-38. «Overreaching vs overtraining: чому ти не зможеш відрізнити одне від одного без даних» — FOR/NFOR/OTS феноменологія однакова в перший тиждень; час і дані перформансу як єдині надійні маркери розрізнення; RPE-дрейф після deload; HRV rolling average (Halson & Jeukendrup 2004); 72-годинний тест; дві помилки self-coached атлета (тренуватись через NFOR; деловантажуватись при FOR); алгоритм для кожного рівня; MD-referral при підозрі на OTS (Meeusen et al. 2013). clinical-safety CONDITIONAL PASS. sports-scientist review required before publish.
- `blog.html` — карта post-38 додана вгорі каталогу.

### Changed
- `README.md` — post-38 → draft · v6.10.1.
- `STATUS.md` — рядок «11–50» оновлено: post-38 editorial додано v6.10.1.
- `VERSION` — 6.10.0 → 6.10.1.

## [6.10.0] — 2026-06-19

### Added
- `content/post-1271-step-count-not-training.md` — Block 1251–1300 Cluster 3, post 1/10. Крокомір: що він рахує і чому це не тренування. Акселерометр і класифікація кроків, точність ±1–4% при ходьбі vs. нульове відображення силових, походження «10 000 кроків» (Hatano 1965), NEAT-фрейм, практичні орієнтири для атлета.
- `content/post-1272-active-calories-accuracy.md` — Cluster 3, post 2/10. Активні калорії: похибка 30–50% — норма. ЧСС→VO2→ккал алгоритм, Shcherbina et al. 2017 (27–93% похибка), EPOC як пропущений витрат, силові vs. кардіо точність, відносне порівняння замість абсолютних чисел.
- `content/post-1273-training-load-algorithms.md` — Cluster 3, post 3/10. Training Load у Garmin/Polar/Whoop. Firstbeat EPOC, Polar Cardio+Muscle Load, Whoop Strain 24/7. Спільна межа: ЧСС не відображає нейром'язовий стрес. ACWR для силовиків — перенесена концепція. Тонни (кг×повторення) надійніший показник.
- `content/post-1274-readiness-score-algorithms.md` — Cluster 3, post 4/10. Body Battery, Oura Readiness, Whoop Recovery: алгоритми і обмеження. Непрозора методологія, відображення минулого, 2–4 тижні для baseline. Де корисні (тренд, конгруентність), де шкодять (тривога, автоматичне скасування тренувань).
- `content/post-1275-vo2max-from-wearable.md` — Cluster 3, post 5/10. VO2max від трекера: submax ЧСС-VO2 проекція. Точність ±3.5 мл/кг/хв (~7%) для бігунів (Roos et al. 2013). Ненадійно: силовики, спека, нерівний рельєф, початківці, бета-блокатори. Тренд за місяці — корисний. Абсолютна цифра — умовна.
- `content/post-1276-neat-and-step-counter.md` — Cluster 3, post 6/10. NEAT і крокомір: різниця до 2 000 ккал/день між людьми (Levine et al. 2005). Феномен «активного атлета, що сидить», LPL і тривале сидіння (Hamilton et al. 2007), легка ходьба як компонент відновлення (Menzies et al. 2010). Межа між NEAT і компульсивним рухом.
- `content/post-1277-intensity-minutes-active-minutes.md` — Cluster 3, post 7/10. Intensity Minutes, Active Minutes, Apple кільця: метрики для загальної популяції. Чому силове тренування майже не рахується в ЧСС-based метриках. Stand Minutes — єдина з реальним LPL-бекграундом. Психологія закриття кілець.
- `content/post-1278-respiratory-rate-wearable.md` — Cluster 3, post 8/10. Дихальна частота від трекера: PPG-based RR, похибка ±1–2 вдихи/хв. Whoop включає в Recovery. Ранній детектор хвороби (Quer et al. 2021 COVID-19 detection). Для здорового атлета — тихий маркер з низькою щоденною варіабельністю.
- `content/post-1279-combining-wearable-metrics.md` — Cluster 3, post 9/10. Як об'єднати метрики трекера. Мінімальний набір: HRV rolling average + RHR + суб'єктив + тренувальний об'єм. Фрейм конгруентності. Saw et al. 2015: суб'єктивні маркери часто чутливіші за об'єктивні. Типові помилки інтерпретації.
- `content/post-1280-activity-load-cluster-3-synthesis.md` — Cluster 3 synthesis, post 10/10. Десять висновків блоку: крокомір, калорії, Training Load, Score-метрики, VO2max, NEAT, Intensity Minutes, RR, інтеграція метрик, конгруентність. Загальна межа корисності трекерів. Наступний кластер: сон.
- `blog.html` — оновлено: posts 1271–1280 додано в каталог.

### Changed
- `VERSION` — 6.9.2 → 6.10.0.

---

## [6.9.2] — 2026-06-19

### Changed
- `docs/SOC2_READINESS.md` — document header corrected v3.13.1 → v3.14.0. §89 (Evidence Artefact Cross-Reference Patch: MSA_TEMPLATE Addendum 5, Partner Agreement) was authored and its version note appended in a previous commit but the H1 title line was not updated at that time. Header now matches the v3.14.0 body note.
- `docs/OBSERVABILITY.md` — document header corrected v4.3.0 → v4.4.0. §47 (OQ-SIEM-02 Resolution — SIEM-CONSENT-01 Chain Invariant, DEC-065) body note correctly reads v4.4.0 since 2026-06-18; H1 title line now matches.
- `docs/INCIDENT_RESPONSE.md` — review footer block corrected v2.6 · 2026-06-14 → v2.7 · 2026-06-15. The v2.7 version note (R-05 OFB-REGION-01 cross-reference patch) was appended to the body but the static footer review block retained the prior version stamp.
- `VERSION` — 6.9.1 → 6.9.2.

## [6.9.1] — 2026-06-19

### Added
- `content/post-1261-resting-hr-recovery-marker.md` — Block 1251–1300 Cluster 2 post 1/10. «ЧСС у спокої: перший показник, який атлет ігнорує» — RHR як маркер відновлення і тренованості; три рівні читання (хронічний тренд, тижневий rolling average, гострий підйом); що підвищує і знижує RHR; точність PPG-трекерів для RHR vs. HRV; персональний baseline через медіану; Pedlar et al. 2018 — 76% атлетів: RHR підвищується до клінічних симптомів хвороби; практичний алгоритм ранок/зал/тижневий огляд. clinical-safety: NOT_REQUIRED.
- `content/post-1262-resting-hr-trend-vs-daily-point.md` — Block 1251–1300 Cluster 2 post 2/10. «ЧСС у спокої: тижневий тренд проти щоденного числа» — добова варіабельність RHR 3–8 уд/хв; три типи відхилень (гострий одноденний стрибок — шум; прогресивний тренд — попередження; тривала висока rolling average — сигнал); персональний baseline через медіану 14–21 дня; threshold rolling average на 5+ уд/хв 3+ дні; сезонне і блокове переоцінювання; як трекери плутають тебе алгоритмічним чорним ящиком. clinical-safety: NOT_REQUIRED.
- `content/post-1263-max-hr-220-minus-age-myth.md` — Block 1251–1300 Cluster 2 post 3/10. «220 мінус вік: чому формула неточна і як знайти справжній HRmax» — Fox et al. 1971 — формула ніколи не публікувалась як клінічна норма; стандартне відхилення ±10–12 уд/хв; Tanaka et al. 2001 (JACC, 18 712 осіб) — краще рівняння 208 − 0.7×вік, але та сама індивідуальна похибка; методи визначення реального HRmax: прямий тест, hill sprint, 30-хвилинний польовий тест, емпіричне відстеження; альтернативні підходи до зон без HRmax (AT, RPE). clinical-safety: NOT_REQUIRED.
- `content/post-1264-hr-zones-individual-setup.md` — Block 1251–1300 Cluster 2 post 4/10. «Зони ЧСС: як встановити індивідуальні межі без лабораторії» — системи зонування (3-зонна, 5-зонна, 7-зонна Seiler, 2-зонна норвезька); VT1 і VT2 — два порогових рівні; talk test (Persinger et al. 2004); 30-хвилинний AT-тест як польовий стандарт; зони від AT vs. зони від % HRmax; практичний приклад; оновлення кожні 8–12 тижнів; для силового атлета RPE достатньо. clinical-safety: NOT_REQUIRED.
- `content/post-1265-hr-in-strength-training.md` — Block 1251–1300 Cluster 2 post 5/10. «ЧСС під час силового тренування: чому 140 уд/хв не означає «важко»» — exercise pressor reflex (Group III/IV аферентні нейрони, метаборецептори м'язів); pressor response пропорційний % від 1RM і тривалості підходу; ЧСС при силовому ≠ аеробна інтенсивність; де ЧСС корисна при силовому (між підходами, загальне метаболічне навантаження, HRR); bradycardia у тренованих силовиків — нормально; RPE і %1RM — надійніші для управління інтенсивністю. clinical-safety: NOT_REQUIRED.
- `content/post-1266-cardiac-drift-during-cardio.md` — Block 1251–1300 Cluster 2 post 6/10. «Серцевий дрейф: чому ЧСС зростає при постійному темпі і що це говорить» — механіка cardiac drift (перерозподіл кровотоку на терморегуляцію, зниження ударного об'єму, компенсаторне підвищення ЧСС); більш виражений у спеку, при дегідратації, нетепловій адаптації; González-Alonso et al. 1997: 4% дегідратація → cardiac output −25%; Convertino et al. 1980: теплова адаптація 7–14 днів знижує drift; практичне управління при кардіо за ЧСС-зонами. clinical-safety: NOT_REQUIRED.
- `content/post-1267-hr-recovery-post-exercise.md` — Block 1251–1300 Cluster 2 post 7/10. «Відновлення ЧСС після тренування: що HRR говорить про серцево-судинну адаптацію» — фізіологія HRR: перша хвилина — парасимпатична реактивація; HRR1 норми (нетреновані 12–18, треновані 30–40+ уд/хв); Cole et al. 1999 (NEJM): HRR1 <12 уд/хв — підвищений ризик; HRR при силовому (між підходами як mini-HRR); Bangsbo et al. 2010: HRR знижується при overreaching; практичний протокол вимірювання і відстеження. clinical-safety: NOT_REQUIRED.
- `content/post-1268-stress-hr-vs-training-hr.md` — Block 1251–1300 Cluster 2 post 8/10. «ЧСС стресу і ЧСС навантаження: як відрізнити два різних сигнали» — спільний симпатичний механізм (HPA-axis + pressor reflex); відмінності у патерні: навантажувальна ЧСС залежить від потужності, стрес-ЧСС варіабельна і тривала; нічна ЧСС — що пояснює підйом; конгруентність для інтерпретації; Salmon 2001 (Br J Sports Med): тренованість знижує ЧСС-відповідь на психологічний стрес; змагальний контекст і pre-competition anxiety. clinical-safety: NOT_REQUIRED.
- `content/post-1269-hr-vs-hrv-what-to-read-first.md` — Block 1251–1300 Cluster 2 post 9/10. «ЧСС vs. HRV: яку метрику читати першою і в якому контексті» — RHR повільніша, стабільніша, краща для хронічних трендів; HRV чутливіша до гострих ефектів, вимогливіша до протоколу; таблиця: яке питання → яка метрика; конгруентність (RHR↑ + HRV↓) — надійний сигнал; суперечність — конфаундер або шум; якщо тільки одна: RHR для довгострокового, HRV для тижневого; послідовність читання: RHR → HRV → суб'єктивна оцінка. clinical-safety: NOT_REQUIRED.
- `content/post-1270-hr-cluster-2-synthesis.md` — Block 1251–1300 Cluster 2 post 10/10 (синтез). Десять ключових висновків кластеру. Трирівнева практична модель: ранок (RHR rolling + HRV rolling) → зал (перші підходи + ЧСС між підходами) → тижневий огляд. Чим ЧСС є і чим не є. Наступний кластер: крокомір, об'єм руху і activity load. clinical-safety: NOT_REQUIRED.

### Changed
- `blog.html` — posts 1261–1270 added (Cluster 2 complete, Block 1251–1300 «Wearables, data & the self-coached athlete»). Total: 1270 posts in blog.
- `STATUS.md` — Block 1251–1300 Cluster 2 COMPLETE (posts 1261–1270, тема: ЧСС у спокої та під час тренування). Next: Cluster 3 (posts 1271–1280, тема: крокомір, об'єм руху і activity load).
- `VERSION` — 6.9.0 → 6.9.1.

## [6.9.0] — 2026-06-19

### Added
- `content/post-1252-hrv-7day-baseline-protocol.md` — Block 1251–1300 Cluster 1 post 2/10. 7-денний rolling baseline HRV: операційний посібник. Формула rolling average, механіка ковзного вікна, мінімальний горизонт калібрування 4 тижні (Nordin et al. 2019). П'ять ситуацій для перерахунку baseline. Три умови оптимального протоколу. Readiness score vs. прямий rolling average. Практична таблиця рішень. clinical-safety: NOT_REQUIRED.
- `content/post-1253-hrv-accumulation-block-normal.md` — Block 1251–1300 Cluster 1 post 3/10. HRV під час накопичувального блоку: функціональне перенакопичення vs. нефункціональне перетренування. U-curve HRV накопичення → deload → суперкомпенсація. Три стани таблицею. Коли «жовтий» стає справжнім сигналом. Buchheit 2014; Plews et al. 2013; Meeusen et al. 2013. clinical-safety: NOT_REQUIRED.
- `content/post-1254-sleep-score-vs-hrv.md` — Block 1251–1300 Cluster 1 post 4/10. Sleep score і HRV: різні виміри різних речей. PSG-порівняння точності (Van den Berg et al. 2019). Три сценарії дисоціації. Ієрархія метрик сну для спортивних рішень: тривалість → стабільність → суб'єктивна оцінка → HRV → sleep score. clinical-safety: NOT_REQUIRED.
- `content/post-1255-hrv-measurement-protocol.md` — Block 1251–1300 Cluster 1 post 5/10. Протокол вимірювання HRV: шість джерел артефактів, п'ятикроковий оптимальний протокол, ієрархія точності пристроїв (ECG → пальцевий PPG → Oura Ring → зап'ясний PPG). Esco & Flatt 2014. clinical-safety: NOT_REQUIRED.
- `content/post-1256-hrv-strength-vs-metabolic.md` — Block 1251–1300 Cluster 1 post 6/10. HRV у силових тренуваннях vs. метаболічна робота: периферійна vs. центральна втома, специфіка нейром'язового сигналу, де HRV корисний для силовика (системний моніторинг, deload). Flatt et al. 2017; Bosquet et al. 2008. clinical-safety: NOT_REQUIRED.
- `content/post-1257-when-to-ignore-hrv.md` — Block 1251–1300 Cluster 1 post 7/10. Коли ігнорувати HRV і тренуватись: 4 ситуації «можна ігнорувати», 2 ситуації серйозного сигналу. Матриця HRV × суб'єктивна готовність. HRV-тривога як реальний ризик. Hooper et al. 1995; Meeusen et al. 2013. clinical-safety: NOT_REQUIRED.
- `content/post-1258-hrv-women-menstrual-cycle.md` — Block 1251–1300 Cluster 1 post 8/10. HRV у жінок: менструальний цикл як вбудована змінна. Фолікулярна (вищий HRV) vs. лютеїнова (10–20% нижче) фаза. Окремі baseline по фазах. Гормональні контрацептиви і зміна профілю. Межі доказової бази. Nakamura et al. 2019; Wikström-Frisén et al. 2017. clinical-safety: CONDITIONAL PASS — освітній нейтральний тон.
- `content/post-1259-hrv-confounders-alcohol-stress-travel.md` — Block 1251–1300 Cluster 1 post 9/10. Конфаундери HRV: алкоголь (-10–25%, 24–48 год), хронічний стрес, гострий стрес перед виміром, хвороба (ранній маркер -15–30%), jet lag (3–7 днів), дегідратація, кофеїн до виміру. Таблиця конфаундерів з тривалістю і діями. Kreutzer et al. 2019; Thayer et al. 2012. clinical-safety: NOT_REQUIRED.
- `content/post-1260-hrv-cluster-1-synthesis.md` — Block 1251–1300 Cluster 1 post 10/10 (синтез). Десять ключових висновків кластеру. Трирівнева практична модель: ранок (rolling average + конфаундери) → зал (перші підходи) → тижневий огляд. Чим HRV є і чим не є. clinical-safety: NOT_REQUIRED.

### Changed
- `blog.html` — posts 1252–1260 added (Cluster 1 complete, Block 1251–1300 «Wearables, data & the self-coached athlete»). Total: 1260 posts in blog.
- `STATUS.md` — Block 1251–1300 Cluster 1 COMPLETE (posts 1251–1260, HRV deep dive). Next: Cluster 2 (posts 1261–1270, тема: ЧСС у спокої та під час тренування).

## [6.8.2] — 2026-06-19

### Changed
- `docs/AUDIT_LOG_SCHEMA.md` — v2.17. Two new DEC-030 HMAC-chained events registered in new `§SIEM Consent & Export Governance events` section: `siem.consent_addendum_signed` (HIGH, 7yr) and `siem.consent_addendum_revoked` (HIGH, 7yr). Closes `docs/OBSERVABILITY.md §47.11` checklist item 2 (P0/M5 — DEC-065). Includes Zod v2 schemas, SIEM-CONSENT-01 chain invariant spec, SIEM-CONSENT-E-001 SOC 2 evidence mapping (CC9.2/CC1.1), CC9.2/CC1.1 auditor narratives, retention table (+2 rows), and fail-closed emitter assignment. Privacy floor: endpoint URL and email hashed only; no user_id or Art. 9 data in chain.
- `docs/OBSERVABILITY.md` — §47.11 checklist item 2 marked [x] Done (2026-06-19).

## [6.8.1] — 2026-06-19

### Added
- `content/post-1251-hrv-rolling-average-vs-daily-point.md` — Block 1251–1300 post 1/50. «HRV як тренувальний сигнал: чому одна точка нічого не означає» — RMSSD як парасимпатичний маркер; добова варіабельність 20–30% (Plews 2012/2013); три типи відхилень (гострий одноденний спад, прогресивний спад тренду, обвал тренду); 7-денне rolling average vs. щоденна точка; персональний baseline (28 вимірювань — мінімум, Nordin et al. 2019); PPG vs. ECG accuracy gap (Hernando et al. 2018); HRV-guided training переваги і обмеження (Buchheit 2014); практичний алгоритм застосування без readiness кружечків. clinical-safety: NOT_REQUIRED. sports-scientist review required before publish.
- `README.md` — Block 1496–1510 «Програмування для атлета без ідеальних умов» added as proposed · v6.8.1. 15 тем: непередбачуваний графік, мінімальний ефективний тиждень, батьківство і тренування, перельоти, хвороба, готель, зміна залу, перехідний тиждень, стресовий місяць, 45-хвилинний блок, паралельний хобі-спорт, сезонна адаптація, системний стрес, 6-місячне планування з перервами, синтез.

### Changed
- `blog.html` — card for post-1251 added (Wearables & Data · 13 хв).
- `STATUS.md` — Block 1251–1300 IN PROGRESS 1/50 · content-engine routine 2026-06-19.

## [6.8.0] — 2026-06-19

### Added
- `content/post-1241-victor-centralized-feedback-memory.md` — Cluster 5 post 1/10. Victor як централізована пам'ять feedback: чотири систематичні проблеми пам'яті (ефект рецентності, підтверджувальне спотворення, стискання, відсутність міток), що Victor робить інакше, практичний старт.
- `content/post-1242-victor-context-tagging-system.md` — Cluster 5 post 2/10. Система контекстних міток у Victor: чотири виміри (джерело, умови тренування, тренувальна фаза, тип спостереження), практичний приклад запису, мінімальний набір.
- `content/post-1243-victor-feedback-logging-protocol.md` — Cluster 5 post 3/10. Протокол внесення feedback у Victor: три рівні запису (базовий 2 хв / стандартний 5 хв / розширений 10–15 хв), таймінг, що не записувати, позначка підтверджує/суперечить.
- `content/post-1244-victor-delayed-analysis-protocol.md` — Cluster 5 post 4/10. Відстрочений аналіз через Victor: 48-годинне правило на практиці — два різних моменти запису, чому 48 год, Victor як «якір» між моментами.
- `content/post-1245-victor-pattern-recognition.md` — Cluster 5 post 5/10. Патерни через Victor: три типи (частотний, часовий, крос-джерельний), чому пам'ять не бачить патернів, патерни як аргумент у розмові з тренером, патерн відсутності.
- `content/post-1246-victor-coach-integration.md` — Cluster 5 post 6/10. Victor і тренер: два формати запису тренерського feedback (реальний час vs. стратегічний), три ефекти Victor, що Victor не робить, варіант без постійного тренера.
- `content/post-1247-victor-training-partner-integration.md` — Cluster 5 post 7/10. Victor і тренувальний партнер: верифікація та суперечності — два сценарії (збіг незалежних джерел / суперечність), рівні підтвердження (гіпотеза → кандидат → підтверджений патерн).
- `content/post-1248-victor-competitive-context-protocol.md` — Cluster 5 post 8/10. Victor і конкурентний контекст: повна послідовність від дня змагань до програмних рішень, ізоляція feedback від конкурентів, змагання як дані.
- `content/post-1249-victor-self-assessment-loop.md` — Cluster 5 post 9/10. Victor і самооцінка: замикання петлі — три практики (калібрування, виявлення розбіжностей, зворотна верифікація), де самооцінка перевищує точність зовнішніх джерел.
- `content/post-1250-cluster-5-block-synthesis.md` — Cluster 5 post 10/10 + Block synthesis. Синтез кластера 5 (6 принципів Victor) і повний синтез блоку 1201–1250 «Coaching intelligence»: пять кластерів у зв'язку, три центральних принципи, що блок не вирішив.

### Changed
- `blog.html` — posts 1241–1250 додані (Cluster 5: Victor як feedback-інфраструктура). blog.html тепер відображає пости до post-1250.
- `STATUS.md` — Block 1201–1250 COMPLETE 50/50 · v6.8.0; наступні пріоритети оновлені.

## [6.7.2] — 2026-06-18

### Changed
- `docs/COST_MODEL.md` — §40 Enterprise Customer Adoption Economics & Seat Utilization Health Model appended (v2.5 → v2.6). Closes the documentation gap between §39 (deal close) and §34 (renewal risk). Covers: three-stage adoption funnel (S1 Activation / S2 Weekly-Active / S3 Habitual); health bands (Green/Amber/Red) with ARR-at-risk and expansion probability per tier; QBR aggregate-only framework (privacy floor: no `user_id` in any artefact); `enterprise_adoption_snapshots` Postgres DDL (migration 0078, RLS, k-anon n ≥ 5); four DEC-030 HMAC-chained events; three SOC 2 evidence artefacts (ADO-E-001/002/003); 10-item implementation checklist; three open questions (OQ-ADO-01/02/03).

## [6.7.1] — 2026-06-18

### Added
- `content/post-122-progression-algorithm.md` — Editorial №122. «Коли додавати вагу, а коли — ні: алгоритм прогресії без шаблонів і без прогресофобії». Три сигнали готовності (RPE-дрейф, RIR-зростання, технічна стабільність); чотири сценарії з алгоритмом рішень; операційний чеклист на сесію; форми прогресії крім ваги. clinical-safety NOT_REQUIRED. sports-scientist review required before publish.
- Blog card for post-122 added to `blog.html`.
- `README.md`: Block 1481–1495 «Вимірювання і діагностика тренованості без лабораторії» added as proposed.
- `STATUS.md`: editorial series updated to 11–122; total posts 1159.

## [6.7.0] — 2026-06-18

### Added
- `content/post-1231-competitive-environment-feedback-context.md` — Block 1201–1250 Cluster 4, post 1/10. Вступ: чому конкурентне середовище — окремий контекст feedback. Три механізми спотворення: соціальне порівняння, захист его, ефект аудиторії. Груповий клас, змагання, лідерборди — всі вони змінюють структуру зворотного зв'язку принципово, а не просто «додають тиск». clinical-safety PASS.
- `content/post-1232-public-training-audience-effect-feedback.md` — Cluster 4 post 2/10. Ефект аудиторії у тренуванні: два напрями спотворення (ті хто дають feedback пом'якшують критику; ті хто отримують стають захисними). Заяц соціальна фасилітація як теоретичний контекст. Протокол: технічні розмови — після тренування, приватно. clinical-safety PASS.
- `content/post-1233-judge-feedback-powerlifting-weightlifting.md` — Cluster 4 post 3/10. Суддівський feedback у пауерліфтингу (IPF/USAPL/WRPF): специфіка білих/червоних прапорів як вбудованого технічного сигналу. Як витягнути корисний сигнал: позиція судді, порівняння між спробами, пост-міт розмова. Три типові помилки при читанні суддівського feedback. clinical-safety PASS.
- `content/post-1234-group-training-hidden-competition-feedback.md` — Cluster 4 post 4/10. Прихована конкуренція у груповому тренуванні (CrossFit, командні програми, семінари): два феномени — інфляція позитивного feedback і порівняльне якоріння. Протокол відокремлення пост-сесійного самоаналізу від групової динаміки. clinical-safety PASS.
- `content/post-1235-coach-competition-vs-training-functions.md` — Cluster 4 post 5/10. Тренер на змаганнях vs. тренер на тренуванні — принципово різні функції. Тренувальна: розвиток, методологія. Змагальна: тактичні рішення, психологічне заземлення, реальночасові cue. Правило реконтекстуалізації: не повертати змагальний coaching feedback у тренувальний блок без переосмислення. clinical-safety PASS.
- `content/post-1236-video-analysis-competition-attempts.md` — Cluster 4 post 6/10. Відеоаналіз змагальних спроб ≠ відеоаналіз тренувального процесу. Технічний розпад на максимальних зусиллях — не системна проблема техніки, а перевищення поточної технічної ємності. Чотирикроковий протокол перегляду. Правило 48 годин перед технічними висновками. clinical-safety PASS.
- `content/post-1237-competitor-feedback-conflict-of-interest.md` — Cluster 4 post 7/10. Feedback від конкурента: конфлікт інтересів що майже ніколи не визнається. Чотирикатегорійний фільтр. Ключове запитання: чия вигода від цієї поради? clinical-safety PASS.
- `content/post-1238-ego-protection-feedback-reception.md` — Cluster 4 post 8/10. Три прояви захисту его у конкурентному контексті: екстерналізація, селективна пам'ять, відторгнення feedback. Адаптивні короткостроково — контрпродуктивні довгостроково. Правило 48 годин: жодних технічних висновків у день змагань. Поведінкова психологія продуктивності, без клінічного контенту. clinical-safety PASS.
- `content/post-1239-competitive-environment-feedback-degradation.md` — Cluster 4 post 9/10. П'ять сигналів деградації feedback-якості у конкурентному середовищі. Рішення: розділити конкурентні виступи і цикли технічного розвитку. Пост-змагальна сесія 48–72 год як основний механізм технічного оновлення. clinical-safety PASS.
- `content/post-1240-cluster-4-synthesis-competitive-feedback.md` — Cluster 4 post 10/10. Синтез: 7 принципів кластера. Контекстна карта блоку 1201–1250 (Cluster 1–4). Прев'ю Cluster 5: Victor як feedback-інфраструктура що зв'язує всі чотири джерела. clinical-safety PASS.

### Changed
- `blog.html` — 10 нових карток постів 1231–1240 додано. Загальне покриття: пости 1–1240 + editorial серія.
- `STATUS.md` — Block 1201–1250: IN PROGRESS 30/50 → 40/50; Cluster 4 COMPLETE v6.7.0; next: Cluster 5 (1241–1250).
- `VERSION` — 6.6.3 → 6.7.0.

## [6.6.3] — 2026-06-18

### Added
- `content/post-1231-competitive-environment-feedback-context.md` — partial (committed separately during session)
- `content/post-1232-public-training-audience-effect-feedback.md` — partial
- `content/post-1236-video-analysis-competition-attempts.md` — partial
- `content/post-1237-competitor-feedback-conflict-of-interest.md` — partial

## [6.6.2] — 2026-06-18

### Changed
- `enterprise.html` — SIEM integration section updated: Sumo Logic removed (not in `siem_destination_type_enum`); Microsoft Sentinel and IBM QRadar added to match v6.5.0 schema (`tenant_siem_configs`). Addendum 4 (SIEM Data Processing) consent requirement noted in integration card description. New FAQ item `efaq10` added: SIEM setup, Addendum 4 workflow, endpoint URL hashing privacy floor.
- `pricing-enterprise.html` — SIEM feature label updated in both the calculator add-ons panel and the feature comparison table: (Datadog, Splunk, Sumo Logic) → (Datadog, Splunk, Sentinel, QRadar). Cross-references: `docs/DATA_MODEL.md §40`, `docs/OBSERVABILITY.md §47`, DEC-065.
- `VERSION` — 6.6.1 → 6.6.2.

## [6.6.1] — 2026-06-18

### Added
- `content/post-37-clean-technique-goal-specific.md` — Editorial series post 37. «Що таке «чиста техніка» — і чому вона залежить від цілі, а не від естетики». Три різні «чисті техніки» для одного руху (пауерліфтер / бодібілдер / recreational атлет); техніка як принцип (рух виконаний + адаптаційний ефект + прийнятний ризик), а не позиція; анатомія як вето — вертлюжні западини і дорсифлексія; демонтаж правила «коліна за носком» (Fry et al. 2003); технічний мінімум vs. ідеальне виконання; техніка під втомою — чотири сигнали критичного дрейфу; три кроки: мета руху → анатомічне вікно → встановлення мінімуму. clinical-safety PASS. sports-scientist review required before publish. 13 хв.

### Changed
- `blog.html` — post-37 card added. Total coverage includes posts 1–1230 + editorial post-37.
- `README.md` — post-37 status: proposed → draft · v6.6.1. Block 1466–1480 «Технічна компетентність і рухова якість» added as proposed (15 topics).
- `STATUS.md` — Editorial series updated: post-37 written v6.6.1.
- `VERSION` — 6.6.0 → 6.6.1.

## [6.6.0] — 2026-06-18

### Added
- `content/post-1221-training-partner-third-feedback-type.md` — Block 1201–1250 Cluster 3 post 1/10. Тренувальний партнер як третій тип feedback: відмінність від тренера (структура відносин, тип уваги), що партнер може що не може (продовжене спостереження vs. технічний аналіз), три функції (реальний час, продовжене спостереження, поведінковий дзеркало), структурування запитань. clinical-safety PASS.
- `content/post-1222-external-view-without-authority.md` — Cluster 3 post 2/10. Що просити партнера спостерігати: чотири категорії надійного спостереження (симетрія, темп, точки відхилення, порівняння підходів), три категорії поза компетенцією (м'язова активація, субтильна техніка, причини), список корисних і марних запитань. clinical-safety PASS.
- `content/post-1223-giving-feedback-to-partner.md` — Cluster 3 post 3/10. Протокол надання feedback партнеру: коли не давати (під час підходу, перед важким підходом, якщо не просив), коли давати (після підходу, при систематичному патерні), чотири правила (опис не оцінка, конкретно, один пункт, «бачу» vs. «знаю чому»), формат [що+де+коли]. clinical-safety PASS.
- `content/post-1224-training-pace-synchronization.md` — Cluster 3 post 4/10. Синхронізація темпу тренування: три механізми синхронізації (соціальний тиск, природна синхронізація ритму, «не затримувати»), де підтягує (хронічно довгі паузи, відволікання, пізні підходи), де стримує (повільний партнер, соціальні розмови), три варіанти рішення (незалежний темп, погоджений, спільний таймер), п'ять сигналів впливу на результати. clinical-safety PASS.
- `content/post-1225-ego-and-training-partner.md` — Cluster 3 post 5/10. Конкурентна динаміка і его: три сценарії несвідомого відхилення від плану, механізм (соціальне порівняння без явного змагання), де корисна (фінальні підходи, хронічне уникання навантаження, свідомий вибір), де шкідлива (несвідомий дрейф), чотири сигнали контролю, інструмент (план до тренування). clinical-safety PASS.
- `content/post-1226-partner-different-level.md` — Cluster 3 post 6/10. Партнер з різним рівнем і стилем: переваги різниці (досвідченіший бачить більше, ефект пояснення, менша конкурентна динаміка), проблеми (різний оптимальний об'єм і темп, ненадійний feedback від менш досвідченого, ризик методологічних суперечок), рішення (роздільні плани, явна домовленість про роль). clinical-safety PASS.
- `content/post-1227-spotting-as-feedback.md` — Cluster 3 post 7/10. Спот як спостережницька позиція: що страхувальник відчуває недоступного на відео (асиметрія зусилля, момент застрявання, якість дотику), техніка хорошого споту у жимі і присяді, типові помилки (постійний дотик, мікро-допомога, підказки без прохання, занадто ранній спот), домовленість перед підходом. clinical-safety PASS.
- `content/post-1228-video-analysis-in-pairs.md` — Cluster 3 post 8/10. Відеоаналіз у парі: переваги (інша точка уваги, розмова під час перегляду, соціальна відповідальність за зйомку), три типові помилки (немає розподілу ролей, аналіз причин замість опису, забагато пунктів), практична схема чотирьох кроків, кути зйомки для присяду/жиму/тяги. clinical-safety PASS.
- `content/post-1229-when-partner-degrades-training.md` — Cluster 3 post 9/10. Коли тренувальний партнер знижує якість: п'ять сигналів деградації, механізм кожного, дії в порядку (структура → чесна розмова → окреме тренування). clinical-safety PASS.
- `content/post-1230-cluster-synthesis-training-partner.md` — Cluster 3 post 10/10 · синтез. Сім принципів кластера: партнер — третій тип feedback; надійний у спостереженні ненадійний у причинному аналізі; формат [що+де+коли]; темп і конкурентна динаміка ненейтральні; спот як спостережницька позиція; різниця рівнів — перевага і ризик; партнерство може деградувати якість. Кластер 3 у контексті блоку 1201–1250. clinical-safety PASS.

### Changed
- `blog.html` — posts 1221–1230 added (Cluster 3 · тренувальний партнер як feedback-ресурс). Total coverage: posts 1–1230.
- `STATUS.md` — Block 1201–1250 updated: Cluster 3 COMPLETE 30/50; next: Cluster 4 posts 1231–1240.
- `VERSION` — 6.5.0 → 6.6.0.

---

## [6.5.0] — 2026-06-18

### Added
- `docs/DATA_MODEL.md §40` — `tenant_siem_configs` SIEM Integration Configuration Schema (v1.19). Full DDL in two-migration design: migration 0063 (base table with `siem_destination_type_enum`, `export_enabled`, `endpoint_url_hash` SHA-256[:32], `filter_rules`, `filter_compliance_approved`, `chk_siem_export_requires_endpoint` CHECK) and migration 0076 (additive: `addendum_signed_at`, `addendum_version`, `signed_by_email_hash`, `chk_siem_addendum_consistency` CHECK, `idx_tsc_addendum_signed` partial index). Includes RLS matrix (`form_api` REVOKED, `form_system` tenant-isolated, `security_reviewer` unrestricted read), four DEC-030 events, export activation state machine, SOC 2 CC9.2/C1.1/CC1.1/CC7.2 cross-references, and SIEM-CONSENT-E-001 cross-check query.

### Changed
- `docs/OBSERVABILITY.md §47.11 item 3` — status updated from `[ ]` to `[x] Closed by DATA_MODEL §40 (v1.19, 2026-06-18)`.
- `VERSION` — 6.4.0 → 6.5.0.

---

## [6.4.0] — 2026-06-18

### Added
- `content/post-121-exercise-order-neural-fatigue.md` — Editorial series post 121. «Порядок вправ у тренуванні: не звичка, а рішення» — нейром'язова специфічність втоми, технічна деградація під навантаженням, адаптаційний пріоритет першої вправи, pre-exhaust як навмисний інструмент, три питання для визначення порядку в сесії. clinical-safety: PASS. sports-scientist review required before publish.
- `README.md` — Block 1451–1500 «Athlete decision-making under uncertainty» додано як нові proposed теми (15 топіків · v6.4.0). Включає: ієрархія тренувальних доказів, помилка надмірної оптимізації, рішення при неповних сигналах, асиметрія ризику, порядок вправ, RPE-калібрування через час, confirmation bias, та синтез.

### Changed
- `blog.html` — картка post-121 додана перед post-120.
- `STATUS.md` — Editorial series 106–120 → 106–121; Total posts: 1157 → 1158; Block 1451–1500 в roadmap.
- `VERSION` — 6.3.0 → 6.4.0.

---

## [6.3.0] — 2026-06-18

### Added
- `content/post-1212-feedback-to-coach.md` — Block 1201–1250 Cluster 2, post 2/10. Feedback тренеру: три типи (операційний, технічний, контекстний), що не є корисним feedback, мінімальний формат тижневого звіту (6 рядків), частота комунікації, feedback як навичка. clinical-safety: PASS.
- `content/post-1213-remote-coaching.md` — Block 1201–1250 Cluster 2, post 3/10. Дистанційний коучинг: що можна і не можна дистанційно, відеопротокол (кути для кожного основного руху, навантаження, технічні параметри), комунікаційний протокол, обмеження відеоаналізу як інтерпретації, коли формат оптимальний. clinical-safety: PASS.
- `content/post-1214-when-to-change-coach.md` — Block 1201–1250 Cluster 2, post 4/10. Коли змінювати тренера: сигнали що не є причиною, реальні червоні прапори (шаблон без індивідуалізації, відсутність реакції на feedback, відсутність пояснення логіки, зникнення довіри), пряма розмова як перший крок, процес зміни. clinical-safety: PASS.
- `content/post-1215-critical-reading-coach-advice.md` — Block 1201–1250 Cluster 2, post 5/10. Критичне читання рекомендацій тренера: чотири питання (чому саме це, для кого спрацьовує, як дізнаємось, що очікувати), коли слідувати попри сумніви, коли сумнів обґрунтований, як ставити питання без підриву відносин. clinical-safety: PASS.
- `content/post-1216-autonomy-vs-compliance.md` — Block 1201–1250 Cluster 2, post 6/10. Автономія vs. compliance: де автономія доречна (мікрорішення в сесії, адаптація до обладнання, feedback), де заважає (зміна структури без погодження, видалення вправ, перевищення інтенсивності), структура зон відповідальності. clinical-safety: PASS.
- `content/post-1217-integrating-coach-feedback.md` — Block 1201–1250 Cluster 2, post 7/10. Інтеграція тренерського feedback: причини невпровадження (одночасність, відсутність ізоляції, пам'ять), протокол (одна рекомендація, конкретна фіксація, горизонт і критерій), технічні зміни (шари від порожньої штанги), коли повертатись з даними. clinical-safety: PASS.
- `content/post-1218-coaching-between-sessions.md` — Block 1201–1250 Cluster 2, post 8/10. Між-сесійний коучинг: п'ять навичок (точне виконання, фіксація відхилень, шум vs. сигнал, знати що вирішувати самостійно, рутина без зовнішнього тиску), рутини до/після сесії і тижнева ревізія. clinical-safety: PASS.
- `content/post-1219-athlete-coach-conflict.md` — Block 1201–1250 Cluster 2, post 9/10. Конфлікт атлет-тренер: три типи розбіжностей (технічна, пріоритети, самооцінка), що не робити, п'ятикроковий алгоритм вирішення, особливий випадок розбіжності у самооцінці руху. clinical-safety: PASS.
- `content/post-1220-cluster-synthesis-coaching.md` — Block 1201–1250 Cluster 2, post 10/10. Синтез кластеру 2: п'ять принципів (тренер як інструмент, якість входу, ітеративний процес, compliance і критичне мислення, між-сесійний час). Відкриті питання: частота контакту, груповий коучинг, межа Victor vs. живий тренер. clinical-safety: PASS.

### Changed
- `blog.html` — posts 1212–1220 added; blog.html updated to post-1220.
- `STATUS.md` — Block 1201–1250 updated: Cluster 2 COMPLETE 20/50; next: Cluster 3 (posts 1221–1230).
- `VERSION` — 6.2.2 → 6.3.0.

---

## [6.2.2] — 2026-06-18

### Added
- `docs/MSA_TEMPLATE.md` Addendum 5 — PARTNER AGREEMENT ADDENDUM (v1.0). Closes `docs/DATA_MODEL.md §38.8` checklist item 3 (P0 — partner privacy floor in MSA_TEMPLATE). Eight-clause addendum: §5.1 four partner categories (referral/reseller/integration/white_label) with DPA requirement matrix; §5.2 four non-waivable privacy floor clauses (individual health data prohibition, no misrepresentation of data access, no-go criteria compliance, Partner DPA requirement); §5.3 three explicit no-go criteria (insurance risk scoring, government backdoors, wellness-as-punishment) with enforcement specificity; §5.4 revenue share governance (quarterly, privacy_floor_check_passed gate, Bronze/Silver/Gold tier benefits, 12-month inactive policy); §5.5 enforcement and PART-CHAIN-01 (four termination grounds, R-22 escalation, revenue share forfeiture and set-off); §5.6 three SOC 2 evidence artefacts (PART-E-001/002/003; paths `compliance/evidence/partners/`); §5.7 four DEC-030 events (`enterprise.partner_agreement_signed/deal_attributed/revenue_share_paid/offboarded`, all 7yr, PAM session required); §5.8 eight-point outside counsel review requirement. MSA_TEMPLATE version bumped v0.3 → v0.4.
- `docs/SOC2_READINESS.md §89` — Evidence Artefact Cross-Reference Patch: MSA_TEMPLATE Addendum 5 (Partner Agreement). Formally registers PART-E-001 (CC9.2 — annual partner agreement execution, `dpa_signed: true` gate), PART-E-002 (CC9.2/CC4.1 — annual revenue share payment governance, `privacy_floor_check_passed: true` gate), PART-E-003 (CC9.2/CC7.4 — quarterly partner offboarding, PART-CHAIN-01 predecessor verification). §89.3 zero-event attestation JSON template. §89.4 CC9.2/CC4.1/CC7.4 criterion rows. §89.5 closes two cross-reference obligations (DATA_MODEL §38.8 item 3, MSA_TEMPLATE Addendum 5 §5.6). §89.6 four-item P1 checklist. SOC2_READINESS version bumped v3.13.1 → v3.14.0.

### Changed
- `VERSION` — 6.2.1 → 6.2.2.

---

## [6.2.1] — 2026-06-18

### Added
- `content/post-1211-first-session-with-coach.md` — Block 1201–1250 Cluster 2 post 1/10 (Cluster 2: робота з тренером). «Перша сесія з тренером: що взяти, що показати, що запитати» — що тренер будує в перші 20 хвилин (модель атлета); тренувальний журнал для тренера (4–6 тижнів, ключові ваги + RPE, де прогрес зупинився); відео техніки (бічний кут, робочий підхід, 4–5 повторів, стабільна камера); 3 питання для оцінки тренера (програмування, відстеження прогресу, основне обмеження); технічна консультація vs. постійний коучинг; 4 критерії якості першої сесії; коли не варто звертатись до тренера зараз. clinical-safety: PASS. sports-scientist review required before publish.
- `README.md` — Block 1401–1450 «Programme design principles: structure, logic, decision-making» додано як нові proposed теми (15 топіків · v6.2.1).

### Changed
- `blog.html` — додано картку для post-1211 (Coaching Intelligence · 13 хв) перед post-1210.
- `STATUS.md` — block 1201–1250: 10/50 → 11/50, Cluster 2 STARTED. Total posts: 1156 → 1157. Версія → v6.2.1.
- `VERSION` — 6.2.0 → 6.2.1.

---

## [6.2.0] — 2026-06-18

### Added
- `content/post-1201-self-assessment-system.md` — Block 1201–1250 Cluster 1 post 1/10. Coaching intelligence & external feedback. «Самооцінка без тренера: чому атлету потрібна система, а не інтуїція» — чотири систематичні помилки самооцінки без структури (рецентність, підтвердження, відсутність baseline, компресія пам'яті); три рівні системи (сесія 2–3 хв, тиждень 10–15 хв, мезоцикл 30–45 хв); мінімальний робочий набір: дата+навантаження+RPE+одне речення+стан на вході. clinical-safety: NOT_REQUIRED.
- `content/post-1202-video-analysis-technique.md` — Block 1201–1250 Cluster 1 post 2/10. «Відеоаналіз власної техніки: що ловить камера і де вона сліпа» — що камера фіксує добре (фронтальна/сагітальна площина, темп, симетрія) і де сліпа (м'язове зусилля, активація, 3D-кінематика, контекст відновлення); оптимальний кут зйомки і fps; практичний протокол відеосесії. clinical-safety: NOT_REQUIRED.
- `content/post-1203-overanalysis-trap.md` — Block 1201–1250 Cluster 1 post 3/10. «Коли відеоаналіз заважає: пастка overanalysis у проміжного атлета» — п'ять ознак що аналіз вийшов за межу; paralysis by analysis — нейронний механізм (конкуруючі когнітивні режими); принцип одного технічного фокусу; практичний орієнтир частоти відеоаналізу. clinical-safety: NOT_REQUIRED.
- `content/post-1204-kinematic-markers.md` — Block 1201–1250 Cluster 1 post 4/10. «Що дивитись на відео: ключові кінематичні маркери для п'яти базових рухів» — мінімальний набір маркерів для присіду, станної тяги, жиму лежачи, горизонтальної і вертикальної тяги; кут зйомки для кожної вправи; практичний протокол використання маркерів. clinical-safety: NOT_REQUIRED.
- `content/post-1205-training-log-feedback.md` — Block 1201–1250 Cluster 1 post 5/10. «Тренувальний журнал як feedback-інструмент: що записувати, щоб навчитись читати» — обов'язковий мінімум (дата, навантаження, RPE, стан на вході, одне речення про техніку); три рівні читання (після сесії, після тижня, після мезоциклу); журнал без читання — марна витрата часу. clinical-safety: NOT_REQUIRED.
- `content/post-1206-rpe-calibration.md` — Block 1201–1250 Cluster 1 post 6/10. «RPE-калібрування: як перетворити суб'єктивне відчуття на надійний сигнал» — шкала RPE 1–10 і RIR; три причини неточності RPE без калібрування; чотири методи калібрування (ретроспективне порівняння, anchor sets, прогресивний набір, кросс-вправне порівняння); поширені помилки. clinical-safety: NOT_REQUIRED.
- `content/post-1207-session-comparison.md` — Block 1201–1250 Cluster 1 post 7/10. «Порівняльний аналіз власних тренувань: як знайти сигнал у власній статистиці» — три типи порівнянь (навантаження×RPE, навантаження при однаковому RPE, об'єм між блоками); правило трьох точок для відфільтрування шуму; виявлення паттернів відновлення. clinical-safety: NOT_REQUIRED.
- `content/post-1208-weekly-self-audit.md` — Block 1201–1250 Cluster 1 post 8/10. «Протокол щотижневого самоаудиту для self-coached атлета» — чотири блоки: виконання плану, відновлення, техніка, рішення на наступний тиждень; мінімальна форма запису; зв'язок з мезоцикловим аналізом. clinical-safety: NOT_REQUIRED.
- `content/post-1209-technique-progress.md` — Block 1201–1250 Cluster 1 post 9/10. «Прогрес у техніці: як він виглядає і коли його можна побачити» — три рівні технічного прогресу (усунення грубих помилок → кінематична оптимізація → автоматизація); ознаки реального прогресу (стабільність під навантаженням, збереження при втомі); що не є технічним прогресом. clinical-safety: NOT_REQUIRED.
- `content/post-1210-cluster-synthesis.md` — Block 1201–1250 Cluster 1 post 10/10 (synthesis). «Кластер-синтез 1: самооцінка без тренера — інструменти, а не ілюзія» — п'ять ключових ідей кластера; мінімальна операційна система самооцінки; що залишилось за межами і де починається кластер 2. clinical-safety: NOT_REQUIRED.

### Changed
- `blog.html` — 10 нових карток додано (posts 1201–1210), вставлені перед post-1200.
- `STATUS.md` — версія v6.1.1 → v6.2.0; Block 1201–1250 рядок додано до content library; Next priorities оновлено.
- `VERSION` — 6.1.2 → 6.2.0.

---

## [6.1.2] — 2026-06-18

### Changed
- `docs/DATA_MODEL.md` — extended v1.15 → v1.18: added §37 Enterprise Pipeline Tracking Schema (`enterprise_pipeline_stages` migration 0072 + `enterprise_impl_time_log` migration 0060), §38 Enterprise Partner Channel Schema (`enterprise_partners` migration 0073 with `partner_category_enum` + `partner_status_enum`), §39 Enterprise Deal Outcomes Schema (`enterprise_deal_outcomes` migration 0074 — closes forward reference `docs/DATA_MODEL.md §39.5` cited in `docs/COST_MODEL.md §39`). Three new TOC entries. Full DDL, RLS, DEC-030 event registry cross-references, SOC 2 evidence artefact tables, implementation checklists, and open questions for each section. Covers: `enterprise_pipeline_stages` (immutable audit pattern, eight-value stage enum, PIPE-CHAIN-01/PIPE-CHAIN-02 invariants, three pipeline DEC-030 events + three implementation DEC-030 events, PIPE-E-001/002/003 artefacts); `enterprise_impl_time_log` (OQ-08 cost tracking, twelve activity codes, five role codes); `enterprise_partners` (PART-CHAIN-01 invariant, DPA requirement matrix, no-go partner criteria, four partner DEC-030 events, PART-E-001/002/003 artefacts); `enterprise_deal_outcomes` (eight win reason codes, ten loss reason codes, six competitor_category values, four CHECK constraints, WIN-CHAIN-01/PIPE-CHAIN-02 DEC-030 invariants, `privacy.no_go_criteria_applied` auto-companion, WIN-E-001/002/003 artefacts). Privacy floor enforced across all sections: no prospect company names, no individual `user_id`, `form_api` REVOKED on all three tables.
- `VERSION` — 6.1.1 → 6.1.2.

---

## [6.1.1] — 2026-06-18

### Added
- `content/post-120-synthesis-nine-frameworks.md` — Editorial series, post 120. «Синтез 112–120: дев'ять рамок для self-coached атлета другого і третього року» — синтез дев'яти постів 112–119 у зв'язану систему рамок: (1) зовнішній стрес як параметр тренування (HPA-вісь, Stults-Kolehmainen & Sinha 2014, Bartholomew 2008); (2) ситуативний шум vs. системний сигнал (RPE-тренд, HRV rolling average, горизонт); (3) обсяг первинний, частота — механізм розподілу (Schoenfeld et al. 2016, Brigatto et al. 2019, 4–6 підходів/сесія стеля); (4) техніка під втомою як маркер якості підходу (Bradford 2023, Refalo 2022, 4 маркери breakdown); (5) зовнішній зворотний зв'язок — партнер/дзеркало/камера (Miall & Wolpert 1996, Zajonc 1965, сліпі зони); (6) мікроцикл як відновна логіка (High-Low принцип, Charlie Francis 1992, 3 типові помилки розміщення); (7) дихальна механіка — IAP і Valsalva як техніка (McGill et al. 1990, Harman 1988, Kavcic et al. 2004); (8) річний горизонт — periodization для аматора (Bickel et al. 2011, 4 блоки, 130–140 сесій/рік); (9) програма має термін придатності — 5 операційних сигналів (Hubal 2005, Meeusen 2013). Мета-секція: діагностичний сценарій з послідовним застосуванням 9 рамок. clinical-safety: NOT_REQUIRED. sports-scientist review required before publish.

### Changed
- `blog.html` — додано картку для post-120 перед post-119.
- `README.md` — post-120 статус: proposed → draft · v6.1.1.
- `STATUS.md` — Editorial series оновлено: 11–119 → 11–120; рядок Editorial 120 додано до таблиці. Версія → v6.1.1.
- `VERSION` — 6.1.0 → 6.1.1.

---

## [6.1.0] — 2026-06-18

### Added
- `content/post-1191-individual-variability-neural-adaptation.md` — Block 1151–1200 Cluster 5, post 1/10. Індивідуальна варіативність нейронної адаптації: Hubal et al. (2005) дані, базовий VA, архітектура MU, нейронна пластичність, катехоламінова система. Non-responders як mismatched responders. Поведінкові ознаки нейронного профілю.
- `content/post-1192-genetic-factors-neural-adaptation.md` — Cluster 5, post 2/10. Генетичні фактори: ACTN3 R577X, BDNF Val66Met, MSTN, IGF-1. Доказова база і реальний ефект-розмір. Що генетика не визначає для аматорів.
- `content/post-1193-fiber-type-neural-adaptation.md` — Cluster 5, post 3/10. Тип волокон і нейронна адаптація: тип I/IIA/IIX, принцип Ханнемана, rate coding, конверсія IIX→IIA при силовому тренуванні. Практичний тест без біопсії. Програмні висновки.
- `content/post-1194-training-age-neural-adaptation.md` — Cluster 5, post 4/10. Тренувальний вік: три стадії нейронного розвитку (0–12 міс / 1–3 роки / 3+ роки). Нейронний «кредит». Діагностика: плато від нейронної стелі vs. помилки програмування.
- `content/post-1195-neural-plateau-mechanisms-strategies.md` — Cluster 5, post 5/10. Нейронне плато: три механізми (habituation, VA-максимум, накопичена втома). Діагностичне деловантаження як діагностичний протокол. Чотири стратегії виходу.
- `content/post-1196-long-term-motor-development.md` — Cluster 5, post 6/10. Довгострокова рухова еволюція: когнітивна → асоціативна → автономна стадія (Fitts і Posner, 1967). Мієлінізація і кортикальна реорганізація. Переучування автоматизованих паттернів: механізм і протокол.
- `content/post-1197-sex-differences-neural-adaptation.md` — Cluster 5, post 7/10. Статеві відмінності: VA, нейронна стійкість до втоми (Hunter 2009 мета-аналіз), циклічність нейронної готовності. Що однакове. Програмні висновки.
- `content/post-1198-age-neural-dynamics.md` — Cluster 5, post 8/10. Вікова нейронна динаміка 35–55: втрата MU, уповільнення провідності, зниження corticospinal excitability. Тренований 55-річний vs. нетренований. MED-стратегія. Чому підтримка інтенсивності важливіша за обсяг з віком.
- `content/post-1199-genetic-testing-training.md` — Cluster 5, post 9/10. Генетичне тестування: що вимірюється, де докази є (ACTN3, ACE), де ринок перебільшує (відсутність RCT «ген → програма», полігенна природа). BASEM і ACSM позиція. Порівняння ДНК-тесту і поведінкового моніторингу.
- `content/post-1200-block-1151-1200-synthesis.md` — Cluster 5, post 10/10 + Block 1151–1200 synthesis. П'ять кластерів, п'ять операційних принципів блоку. Що змінилося після блоку. Можливі напрямки Блоку 1201–1250.
- `blog.html` — posts 1191–1200 added to the top of the feed.

### Changed
- `VERSION` — 6.0.2 → 6.1.0 (Block 1151–1200 COMPLETE).
- `STATUS.md` — Block 1151–1200 status updated to COMPLETE 50/50.

---

## [6.0.2] — 2026-06-18

### Changed
- `docs/SOC2_READINESS.md` v3.13.0 → v3.13.1 — §79.4 master evidence table patch: four artefact rows added (WIN-E-001 CC5.2/CC1.4, WIN-E-002 CC5.2/CC4.1, WIN-E-003 CC1.4/CC9.2, SIEM-CONSENT-E-001 CC9.2/CC1.1). §80.3 R2 folder structure extended with `winloss/` (WIN-E-001/002/003; form-api NO ACCESS) and `siem-consent/` (SIEM-CONSENT-E-001; form-api NO ACCESS). Closes §87.7 item 4 (P1/M10) and §88.5 item 1 (P1/M8). Document header corrected v3.10.0 → v3.13.1.
- `VERSION` — 6.0.1 → 6.0.2.

---

## [6.0.1] — 2026-06-18

### Added
- `content/post-119-program-signs-stopped-working.md` — Editorial series, post 119. «Ознаки того, що твоя програма перестала підходити — і що з цим робити» — три причини зупинки прогресу (шум / функціональне перевантаження / програмна невідповідність); 5 операційних сигналів: RPE-дрейф на незмінних навантаженнях, нульовий прогрес 4+ тижнів, змінений патерн відновлення, технічна регресія на свіжому стані, системне зниження включення; що НЕ є сигналом; три відповіді (мікроадаптація / деловантаження+переоцінка / перебудова програми) і 5-кроковий алгоритм діагностики. Джерела: Hubal et al. 2005 (variability in muscle response); Meeusen et al. 2013 (ECSS/ACSM overreaching consensus); Halson & Jeukendrup 2004 (overtraining analysis); Krieger 2010 (volume meta-analysis); McEwen 2007 (allostatic load). clinical-safety: PASS. sports-scientist review required before publish.

### Changed
- `blog.html` — додано картку для post-119 перед post-118.
- `STATUS.md` — Editorial series оновлено: 11–118 → 11–119; рядок контент-таблиці Editorial 119 додано. Версія → v6.0.1.
- `VERSION` — 6.0.0 → 6.0.1.

---

## [6.0.0] — 2026-06-18

### Added
- `content/post-1181-periodization-models-neural-adaptations.md` — Cluster 4, post 1/10. Три моделі прогресії: лінійна (0–12 міс, нереалізований нейронний потенціал, нейронні адаптації 60–80% gainів у перші 6–12 тижнів), хвильова weekly undulating (6–24 міс, різні зони навантаження тижня), DUP Daily Undulating Periodization (12+ міс, денна варіативність). Rhea et al. (2002): DUP +28.8% vs. лінійна +14.4% 1RM squat за 12 тижнів. Таблиця: стаж, частота, нейронна варіативність, складність управління. Вибір моделі за рівнем тренованості нейронної системи. clinical-safety: NOT_REQUIRED. sports-scientist review required before publish.
- `content/post-1182-rpe-autoregulation-calibration.md` — Cluster 4, post 2/10. RPE як авторегуляція: чому фіксований відсоток — наближення. Шкала Zourdos RPE 1–10 прив'язана до RIR (reps in reserve). Різниця між Боргом 6–20 і силовою шкалою. Протокол калібрування: запис → порівняння реального запасу → рекалібрування до відмови 1–2 рази на місяць. Helms et al. (2016): r=0.75–0.85 у тренованих ≥2 роки. Гібридна модель: плановий відсоток як вихідна точка, RPE-корекція як механізм адаптації. clinical-safety: NOT_REQUIRED.
- `content/post-1183-velocity-based-training-neural.md` — Cluster 4, post 3/10. VBT: швидкість штанги як об'єктивний нейронний маркер. Load-velocity profile: побудова (5 зон ваги, лінійний енкодер), використання в день тренування. Velocity loss: <20% = помірна стимуляція (70–75% адаптивного ефекту при мінімальній втомі), ~40% ≈ відмова. González-Badillo et al. (2016). Обмеження: потрібен пристрій, не всі вправи підходять. Альтернатива без пристрою: RPE + темп. clinical-safety: NOT_REQUIRED.
- `content/post-1184-hrv-training-load-decision-protocol.md` — Cluster 4, post 4/10. HRV і тренувальні рішення: baseline формування (28 вимірювань, 7-денне ковзаюче середнє, фіксована поза і пристрій). Три зони (+/-0.5 SD зелена, -0.5/-1.5 SD жовта, >-1.5 SD червона) з конкретними операційними відповідями. Матриця HRV × RPE як подвійна валідація. Чотири типові помилки. clinical-safety: NOT_REQUIRED.
- `content/post-1185-functional-overreaching-supercompensation.md` — Cluster 4, post 5/10. FOR vs. NFOR: критична різниця. Нейронний механізм суперкомпенсації: центральна втома → зниження VA → правильне деловантаження → VA повертається → реалізація накопиченої адаптації. Структура важкого блоку: +20–40% обсяг, 80–90% 1RM, 1–2 тижні, критерії зупинки. Деловантаження після FOR: параметри і ознаки готовності до піку. 2–3 FOR-цикли на рік для аматора. clinical-safety: NOT_REQUIRED.
- `content/post-1186-neural-mesocycle-structure.md` — Cluster 4, post 6/10. Нейронно-орієнтований мезоцикл 4–6 тижнів: чотири фази (Накопичення 70–80% → Інтенсифікація 82–90% знижений обсяг → FOR/Пік 88–93% → Деловантаження ≥80% -50% обсяг). Таблиця 6-тижневого мезоциклу по фазах/RPE/обсягу. Варіанти 4 і 5-тижневої схеми. Авторегуляція фаз при відхиленні HRV. Зміна акценту між мезоциклами. clinical-safety: NOT_REQUIRED.
- `content/post-1187-peak-protocol-taper-math.md` — Cluster 4, post 7/10. Протокол піку і тапер-математика. Різниця між таперингом і деловантаженням. Три паралельних процеси: усунення центральної втоми, збереження нейронної збудливості, оптимізація м'язового стану. Bosquet et al. (2007): 182 протоколи, -41–60% обсягу, 8–14 днів, незмінна інтенсивність = +2.2% середній ефект. 10-денний протокол піку для аматора. Контроль DOMS: жодних нових вправ за 5 днів до тесту. Психологічна підготовка: одна зовнішня підказка. clinical-safety: NOT_REQUIRED.
- `content/post-1188-intra-session-autoregulation.md` — Cluster 4, post 8/10. Авторегуляція всередині сесії: розминочний підхід при 70–75% як нейронний тест. Матриця рішень (RPE +/-1–2 від норми → дії). Три сценарії: сесія «здулась», іде краще за план, прогресивна втома. Відпочинок між підходами як авторегуляційний параметр (нейронна сесія: 3–7 хв). Відмінність авторегуляції від ігнорування плану. clinical-safety: NOT_REQUIRED.
- `content/post-1189-macrocycle-neural-programming.md` — Cluster 4, post 9/10. Макроцикл і блокова модель (Issurin, 2008): Accumulation → Transmutation → Realization. Ієрархічна залежність між блоками. Практичний річний розклад аматора: 2–3 повних цикли (лютий–квітень, травень–серпень, вересень–грудень) з конкретними датами тест-подій. Таблиця залишкових тренувальних ефектів: нейронна сила (початок зниження 2–3 тижні), гіпертрофія (4–6 тижнів), технічний паттерн (4–6 тижнів). Довгострокове нейронне плато: ознаки і рішення. clinical-safety: NOT_REQUIRED.
- `content/post-1190-cluster-1181-1190-neural-programming-synthesis.md` — Cluster 4, post 10/10 (synthesis). По одному ключовому пункту з кожного з постів 1181–1190. Три операційні принципи: авторегуляція як ієрархічний процес (макро→мезо→сесія), нейронний стимул і нейронна вартість як окремі параметри, дата тест-події організовує програму зворотнім відліком. Наступний — Cluster 5 (1191–1200): індивідуальна варіативність і довгостроковий нейронний розвиток. clinical-safety: NOT_REQUIRED.

### Changed
- `blog.html` — додано картки для постів 1181–1190 перед post-1180.
- `STATUS.md` — Block 1151–1200: 30/50 → 40/50, Cluster 4 COMPLETE, next → Cluster 5. Version header → v6.0.0.
- `VERSION` — 5.99.2 → 6.0.0.

---

## [5.99.2] — 2026-06-18

### Changed
- `docs/MSA_TEMPLATE.md` v0.3 → v0.4 — Addendum 4 (SIEM Data Processing Addendum v1.0) authored. Five-clause structure (Scope & Activation with SIEM-CONSENT-01 invariant reference; Documented Instructions per GDPR Art. 28(3)(a); Customer Obligations incl. no-reidentification warranty; FORM Obligations incl. privacy floor warranty + 72h breach notification + DEC-030 chain record; Revocation via Admin Dashboard). §4.6 SOC 2 evidence cross-reference (SIEM-CONSENT-E-001, CC9.2/CC1.1). §4.7 governing law. §4.8 five-point outside-counsel review requirement. Order Form updated with SIEM Addendum checkbox. Internal Notes updated with SIEM Addendum 4 guidance row. Closes `docs/OBSERVABILITY.md §47.11` checklist item 5 (P1/M5). References: DEC-065, `docs/OBSERVABILITY.md §47`.
- `docs/OBSERVABILITY.md` — §47.11 checklist item 5 marked [x] Done (2026-06-18): Addendum 4 authored.
- `VERSION` — 5.99.1 → 5.99.2.

---

## [5.99.1] — 2026-06-18

### Added
- `content/post-11-deload-weeks.md` — Post 11 (content-engine): «Діловий тиждень: чому деловантаження — це тренування, а не капітуляція». Supercompensation механізм, fitness-fatigue model, ритміка делоадів (4–6 тижнів), шість ознак потреби, практика для Upper/Lower split. Clinical-safety PASS (training science); sports-scientist review pending.
- `blog.html` — картка post-11 додана у топ grid.

### Changed
- `STATUS.md` — 11–50 row: post-11 (deload weeks) відмічено як додано; version → v5.99.1.
- `VERSION` — 5.99.0 → 5.99.1.

---

## [5.99.0] — 2026-06-18

### Added
- `content/post-1171-pap-mechanism-myosin-phosphorylation.md` — Block 1151–1200 Cluster 3 post 1/10. Постактиваційна потенціація: механізм фосфорилювання регуляторних легких ланцюгів міозину (MLCK), підвищена Ca²⁺-чутливість міофібрил, H-рефлекторна потенціація. Двофазна відповідь: втома (1–3 хв) → PAP домінує (4–10 хв). Sale (2002), Hamada et al. (2000).
- `content/post-1172-pap-complex-training-protocol.md` — Cluster 3 post 2/10. PAP на практиці: complex training (85–90% 1RM → 4–8 хв → вибухова робота), contrast sets. Залежність від тренувального статусу: +3–8% у тренованих. Wilson et al. (2013) мета-аналіз, Seitz & Haff (2016). Ключова змінна — інтервал відпочинку.
- `content/post-1173-deload-neural-mechanism-central-fatigue.md` — Cluster 3 post 3/10. Деловантаження як нейронний інструмент: VA знижується до 91–95% після мезоциклу, нейромедіаторний дисбаланс, підвищена кортикальна витрата. Відновлення VA за 5–7 днів при -40–60% об'єму + збережена інтенсивність ≥80% 1RM. Meeusen et al. (2010), Gandevia (2001).
- `content/post-1174-stress-hpa-axis-strength-neural-readiness.md` — Cluster 3 post 4/10. Стрес і HPA-вісь: кортизол змінює збудливість моторних нейронів. Marcora (2009): ментальна втома знижує час до відмови на 15.1% при ідентичних периферичних маркерах. Хронічний стрес і overtraining syndrome мають перекриваючі нейроендокринні маркери.
- `content/post-1175-sleep-strength-performance-neural-readiness.md` — Cluster 3 post 5/10. Сон і максимальна сила: ≤6 год → grip strength -5–8%, час до відмови -10–11%. Знижена кортикоспінальна збудливість, підвищений аденозин. Blumert et al. (2007) — важкоатлети після депривації сну. Mah et al. (2011) — подовжений сон +5%.
- `content/post-1176-rate-coding-strength-adaptation-firing-rate.md` — Cluster 3 post 6/10. Rate coding: після повного рекрутування сила зростає через частоту розряду. Carroll et al. (2002): тренована група — вищі пікові частоти. Специфічний стимул — 85–97% 1RM. Cluster sets, балістична робота. Межі GtG для пікового rate coding.
- `content/post-1177-deload-supercompensation-voluntary-activation.md` — Cluster 3 post 7/10. Суперкомпенсація VA: twitch interpolation, VA 96–99% після тапера. Bosquet et al. (2007): 8–14-денний тапер -41–60% об'єму → +3–6% результату. Пік нейронного виразу через 7–14 днів. Атлети без деловантажень — хронічно пригнічена VA.
- `content/post-1178-neural-plateau-strength-development-diagnosis.md` — Cluster 3 post 8/10. Нейронне плато: два типи (структурне vs нейронне VA/rate coding). Діагностика через тапер-тест і RPE-динаміку. Рішення: важка робота 90%+, contrast sets, систематичний тапер. Помилка — додавати об'єм при нейронному плато.
- `content/post-1179-day-of-neural-readiness-heavy-attempt.md` — Cluster 3 post 9/10. П'ять керованих факторів готовності в день підходу: циркадний ритм (Drust 2005, пік 16–19:00, +3–8%), PAP-розминка (90–95% 1RM → 4–8 хв), кофеїн 3–6 мг/кг (Grgic et al. 2020 мета-аналіз), збудження (Єркес–Додсон), зовнішній фокус (Wulf & Lewthwaite 2016).
- `content/post-1180-cluster-1171-1180-neural-strength-regulation-synthesis.md` — Cluster 3 post 10/10. Synthesis: 9 ключових знахідок по одній на пост, три операційні принципи (нейронна система — керований обмежувач; структура деловантаження має значення; максимальна сила потребує нейронних умов), п'ять помилок, програмні наслідки, preview Кластера 4.
- `blog.html` — оновлено: додано 10 записів Кластера 3 (пости 1171–1180) у верхній частині списку.

### Changed
- `STATUS.md` — Block 1151–1200 оновлено до 30/50; Cluster 3 COMPLETE; Next: Cluster 4 (1181–1190).

## [5.98.1] — 2026-06-18

### Added
- `docs/OBSERVABILITY.md §47` — OQ-SIEM-02 Resolution: Tenant SIEM Export Consent, Per-Tenant DPA Addendum Design & SIEM-CONSENT-01 Chain Invariant (DEC-065). Decision: per-tenant signed MSA Addendum 4 required before `siem.tenant_export_enabled`; Admin Dashboard consent capture UI gate; SIEM-CONSENT-01 chain invariant (HTTP 422 `SIEM_CONSENT_01_NO_ADDENDUM` if no prior `siem.consent_addendum_signed` in chain); two new DEC-030 events (`siem.consent_addendum_signed` HIGH 7yr, `siem.consent_addendum_revoked` HIGH 7yr); MSA Addendum 4 template (five clauses); `tenant_siem_configs` DDL extension (addendum_signed_at, addendum_version); one SOC 2 artefact SIEM-CONSENT-E-001 (CC9.2/CC1.1 — quarterly signed-addendum roster); eight-item implementation checklist (4× P0/M5, 2× P1/M5, 2× P1/M8). Closes OQ-SIEM-02 from §27.12 (P1 — before M5). Cross-ref: `docs/DECISION_LOG.md DEC-065`.
- `docs/SOC2_READINESS.md §88` — Evidence Artefact Cross-Reference Patch: OBSERVABILITY §47 (DEC-065). Registers SIEM-CONSENT-E-001 (CC9.2/CC1.1) in SOC2_READINESS evidence tables; extends CC9.2 and CC1.1 criterion rows; closes OBSERVABILITY §47.9 cross-reference obligation.
- `docs/DECISION_LOG.md DEC-065` — OQ-SIEM-02: Per-tenant MSA Addendum 4 (SIEM Data Processing Addendum) required before `siem.tenant_export_enabled`. Four grounds: GDPR Art. 28(3)(a) written instruction gap; enterprise IT expectation; reverse cost of non-compliance higher; SIEM-CONSENT-01 technology enforcement consistent with FORM's data-layer compliance principle.

### Changed
- `VERSION` — 5.98.0 → 5.98.1.

---

## [5.98.0] — 2026-06-18

### Added
- `content/post-1161-motor-programs-schemas.md` — Block 1151–1200 Cluster 2, post 1/10. Generalized Motor Program і Schema Theory (Schmidt, 1975). Де зберігаються моторні програми: M1, SMA, мозочок, базальні ганглії. Три процеси при «виучуванні» важкого руху. Парадокс Бека: надмірна увага до техніки у досвідчених атлетів знижує якість виконання.
- `content/post-1162-feedback-feedforward-motor-control.md` — Block 1151–1200 Cluster 2, post 2/10. Feedback (реактивний) vs feedforward (прогностичний) управління рухом. Forward і inverse models мозочка (Wolpert & Kawato, 1998). Тренований атлет = точніші внутрішні моделі. Помилки передбачення як навчальний сигнал. Ексцентрик — feedback-інтенсивна фаза.
- `content/post-1163-stages-motor-learning-fitts-posner.md` — Block 1151–1200 Cluster 2, post 3/10. Три стадії моторного навчання (Fitts & Posner, 1967): когнітивна → асоціативна → автономна. fMRI-відбиток кожної стадії. Reinvestment phenomenon. Регрес при технічній переробці. Різний тип підказок для кожної стадії.
- `content/post-1164-focus-attention-motor-learning-wulf.md` — Block 1151–1200 Cluster 2, post 4/10. Constrained Action Hypothesis (Wulf). Зовнішній фокус → нижча ЕМГ при рівному або вищому силовому виході. OPTIMAL Theory (Wulf & Lewthwaite, 2016). Практичний фреймінг підказок. Відеоаналіз: ефективний лише за конкретного питання-фокусу.
- `content/post-1165-variability-practice-contextual-interference.md` — Block 1151–1200 Cluster 2, post 5/10. Contextual Interference Effect (Battig & Shea). Рандомна практика гірша під час навчання, краща у ретенції. Schema Theory у дії. Desirable Difficulties (Bjork). Модель прогресії варіативності від нової вправи до освоєної.
- `content/post-1166-motor-learning-plateau.md` — Block 1151–1200 Cluster 2, post 6/10. Нейронна динаміка плато. Deliberate Practice (Ericsson, 1993): завдання трохи за межею, зворотний зв'язок, когнітивна залученість. Offline consolidation через сон (Walker et al., 2002). Механізми «застрявання» і стратегії виходу.
- `content/post-1167-skill-transfer-specificity.md` — Block 1151–1200 Cluster 2, post 7/10. Трансфер визначається координаційним паттерном і вектором зусилля, а не м'язами. Чотири рівні перенесення. Жим лежачи і жим стоячи — різні нейронні навички. Нейронне обґрунтування вибору допоміжних вправ.
- `content/post-1168-fatigue-motor-learning.md` — Block 1151–1200 Cluster 2, post 8/10. Втома змінює нейромоторний паттерн: компенсаторна рекрутація «записується» у схему. Чому нові рухи — на початку сесії. State-dependent learning. Виняток: специфічна підготовка до виконання навички під втомою.
- `content/post-1169-motor-imagery-mental-rehearsal.md` — Block 1151–1200 Cluster 2, post 9/10. fMRI і TMS: уявне виконання активує ті ж нейронні мережі. Yue & Cole (1992): +22% сили від уявних скорочень. PETTLEP-модель. Кінестетична > зорової уяви для моторного навчання. Clark et al. (2014): уявне тренування уповільнює атрофію при іммобілізації.
- `content/post-1170-cluster-synthesis-motor-adaptation.md` — Block 1151–1200 Cluster 2, post 10/10. Синтез Кластера 2. Три операційні принципи: навчання vs тренінг — різні режими; помилки передбачення є навчальним сигналом; час між сесіями є частиною навчання. П'ять поширених помилок. Анонс Кластера 3: нейронна регуляція силового розвитку (PAP, деловантаження, нейронна готовність).

### Changed
- `blog.html` — posts 1161–1170 added (Cluster 2 complete); total blog entries updated to post-1170.
- `VERSION` — 5.97.0 → 5.98.0.

---

## [5.97.0] — 2026-06-18

### Added
- `compliance/fieldwork/incident-reason-verification.md` v1.0 — Auditor fieldwork protocol for `incident.severity_changed` reason hash retrieval (DEC-044). Five-step procedure: (1) locate `incident.severity_changed` chain record via `compliance_reviewer` SQL; (2) retrieve `incident.linear_ticket_linked` URL; (3) access Linear ticket via read-only guest credentials; (4) read `reason_plaintext` from ticket description/log; (5) independent SHA-256 hash verification against stored `reason_hash` using `INCIDENT_REASON_HASH_SALT` from 1Password Operations vault. Hash formula: `SHA-256(incident_id + '|' + reason_plaintext + '|' + INCIDENT_REASON_HASH_SALT)`. Privacy constraints on fieldwork artefacts documented. SOC 2: P3.2, CC2.2. Closes `docs/INCIDENT_RESPONSE.md §17.7 checklist item 6` (P1/M5).
- `compliance/fieldwork/concurrent-incident-chain-verification.md` v1.0 — Auditor fieldwork protocol for concurrent incident sub-chain ordering (DEC-053). All five §18.5 spec items: (1) timestamp-interleaved master chain design rationale (single-list HMAC invariant; sub-chain fork rejected on five grounds); (2) §18.3.1 canonical SQL with annotated column table (`sub_chain_seq`, `master_chain_prev_event_id`, non-INC predecessor interpretation); (3) §18.3.2 four-step HMAC spot-verification (primary: SOC2_READINESS §79.7 master check; optional per-incident path with 1Password salt lookup); (4) §18.3.3 CONC-CHAIN-01 detection query with zero-row/any-row interpretation guide; (5) Tabletop Scenario P worked example — INC-20260618-a1b2c3 (P0 breach) + INC-20260618-d4e5f6 (P1 SIEM false-positive) — showing §18.3.1 outputs for each incident, non-INC predecessor entries annotated as correct/expected, §18.3.3 zero-row CONC-E-001 attestation. SOC 2: CC7.4, CC4.1. Closes `docs/INCIDENT_RESPONSE.md §18.7 checklist item 1` (P1/M7).
- `compliance/evidence/auditor-onboarding/README.md` v1.0 — Auditor onboarding package index. Lists access grants (compliance_reviewer DB role, Linear read-only credentials, 1Password vault share, R2 pre-signed URLs), indexes both fieldwork protocol documents with SOC 2 TSC mapping, links to MASTER-INDEX (`docs/SOC2_READINESS.md §79`). Satisfies `docs/INCIDENT_RESPONSE.md §18.7 item 1` requirement to include the concurrent-incident fieldwork doc in the onboarding package index.

### Changed
- `docs/INCIDENT_RESPONSE.md` — §17.7 checklist item 6 marked [x] Done (2026-06-18): `compliance/fieldwork/incident-reason-verification.md` v1.0 authored. §18.7 checklist item 1 marked [x] Done (2026-06-18): `compliance/fieldwork/concurrent-incident-chain-verification.md` v1.0 authored with worked example; `compliance/evidence/auditor-onboarding/README.md` v1.0 created.
- `VERSION` — 5.96.0 → 5.97.0.

## [5.96.0] — 2026-06-18

### Added
- `content/post-118-valsalva-iap-core-bracing.md` — Editorial series post 118. «Valsalva і внутрішньочеревний тиск: чому «тримай повітря» — це не забобони». IAP як активна кінетична ланка (McGill et al. 1990); bracing vs. hollowing (Kavcic et al. 2004); Valsalva до 200 мм рт. ст. (Harman et al. 1988); чотири стінки IAP-системи; серцево-судинна безпека у здорових атлетів (Lentini et al. 1999); коли Valsalva, коли дихання через повторення; п'ять помилок. clinical-safety: PASS. 13-min read.
- `content/post-118-longterm-self-coached-planning.md` — Editorial supplementary (concurrent with 118). «Довгострокова програма без тренера: що треба мати у голові на рік вперед» — річний горизонт як системна рамка; часові константи адаптацій (нейром'язова/структурна/сполучна тканина); periodization для аматора — три фази; базова річна структура 4 блоків; математика послідовності 130-140 сесій; Bickel 2011, Hubal 2005; 4 питання перед річним горизонтом. clinical-safety: PASS. 13-min read.
- `content/post-1151-rukhovi-odynytsi-fundamentalna-odynytsia-syly.md` — Block 1151–1200 Cluster 1, post 1 of 10. Рухові одиниці: альфа-мотонейрон і закон «все або нічого»; типи I/IIa/IIx (Burke 1967); гіпертрофія змінює потужність «пострілу», а не структуру іннервації (Narici et al. 1989); чому IIx потребує proximity to failure (Schoenfeld et al. 2017). clinical-safety: NOT_REQUIRED. review_pending: sports-scientist.
- `content/post-1152-pryntsyp-rozmiru-henneman-rekrutuvannia.md` — Block 1151–1200 Cluster 1, post 2 of 10. Принцип розміру Геннемана (1965): порядок рекрутування детермінований біофізикою мотонейрону; компенсаторне рекрутування при стомленні; Del Vecchio et al. 2019 (intramuscular EMG); proximity to failure як ключовий параметр гіпертрофії; Schoenfeld et al. 2017 meta-analysis. clinical-safety: NOT_REQUIRED. review_pending: sports-scientist.
- `content/post-1153-rate-coding-chastota-rozriadiv-moduliatsiia-syly.md` — Block 1151–1200 Cluster 1, post 3 of 10. Rate coding: від twitch до тетанусу; дублетні розряди (Burke et al. 1976; Van Cutsem et al. 1998); "намір рухатися швидко" як нейральний стимул (Behm & Sale 1993); адаптація rate coding через 2–4 тижні тренування. clinical-safety: NOT_REQUIRED. review_pending: sports-scientist.
- `content/post-1154-neuralnyi-draiv-maksymalna-syla-1pm.md` — Block 1151–1200 Cluster 1, post 4 of 10. Нейральний драйв і voluntary activation: twitch interpolation (Merton 1954; Belanger & McComas 1981); VA 85–95% у нетренованих vs. 95–99% у тренованих (Kent-Braun & Le Blanc 1996); кортикоспінальний шлях і TMS (Todd et al. 2003); клітини Реншоу і Ia гальмування; психологічна мобілізація +5–12% сили (Behm 2004). clinical-safety: NOT_REQUIRED. review_pending: sports-scientist.
- `content/post-1155-ranni-sylovi-adaptatsii-neuralni-mekhanizmy.md` — Block 1151–1200 Cluster 1, post 5 of 10. Ранні нейральні адаптації: Moritani & deVries 1979; Häkkinen 1985 (квадрицепс, ЕМГ vs. біопсія); п'ять механізмів (рекрутування IIx, rate coding, зниження ко-контракції антагоніста, зменшення білатерального дефіциту, міжм'язова координація); гіпертрофія за МРТ не раніше 6–8 тижнів. clinical-safety: NOT_REQUIRED. review_pending: sports-scientist.
- `content/post-1156-intermuscular-coordination.md` — Block 1151–1200 Cluster 1, post 6 of 10. Міжм'язова координація: внутрішньом'язова vs. міжм'язова; синхронізація рухових одиниць (Milner-Brown et al. 1975); зниження ко-контракції антагоніста (Carolan & Cafarelli 1992); специфічність нейральної адаптації (Sale & MacDougall 1981). clinical-safety: NOT_REQUIRED. review_pending: sports-scientist.
- `content/post-1157-muscle-spindles-gto.md` — Block 1151–1200 Cluster 1, post 7 of 10. М'язові веретена і GTO: Ia аференти і стретч-рефлекс; γ-мотонейронна регуляція і PAP; SSC нейральний внесок до 30–40% (Komi 2000); GTO і Ib гальмування; адаптація порогу GTO у тренованих (Aagaard et al. 2000). clinical-safety: NOT_REQUIRED. review_pending: sports-scientist.
- `content/post-1158-inhibition-disinhibition.md` — Block 1151–1200 Cluster 1, post 8 of 10. Гальмування і розгальмування: клітини Реншоу, Ia реципрокне, Ib автогенне; тренування знижує Ia інгібіцію (Aagaard et al. 2002); VA 85–92% нетреновані → 95–99% тренованих; "надлюдська сила" при адреналіновому розгальмуванні — частковий ефект, не повний. clinical-safety: NOT_REQUIRED. review_pending: sports-scientist.
- `content/post-1159-central-peripheral-fatigue.md` — Block 1151–1200 Cluster 1, post 9 of 10. Центральна і периферична втома: периферична — Pi, pH, збій Ca²⁺ трансієнту (Allen, Lamb & Westerblad 2008); центральна — знижений VA, серотонін/допамін (Newsholme), аференти III/IV групи; RPE як активний регулятор (Marcora et al. 2009); асиметричні часові шкали відновлення і їх програмувальні наслідки. clinical-safety: NOT_REQUIRED. review_pending: sports-scientist.
- `content/post-1160-cluster-synthesis.md` — Block 1151–1200 Cluster 1, post 10 of 10. Синтез кластеру 1151–1160: дистиляція кожного з 9 постів, три інтегруючих принципи (нейральна адаптація як передумова гіпертрофії; специфічність нейрального навчання; відновлення як нейральний процес), п'ять розвінчаних міфів про "нейральне тренування". Teaser: Cluster 2 — рухова адаптація і моторна пам'ять. clinical-safety: NOT_REQUIRED. review_pending: sports-scientist.

### Changed
- `blog.html` — 12 нових карток: post-1160 (synthesis) → post-1151 (motor units) + post-118 Valsalva/IAP + post-118 Longterm planning (content-engine). blog.html updated to post-1160.
- `docs/DECISION_LOG.md` — DEC-064: Block 1151–1200 тема «Нейром'язовий контроль та рухова адаптація» затверджена.
- `STATUS.md` — Block 1151–1200 row додано; total posts 1145 → 1146; editorial 201–210 proposed; README roadmap оновлено.
- `README.md` — post-118 status updated to draft · v5.96.0; editorial topics 201–210 added to roadmap.
- `VERSION` — 5.95.1 → 5.96.0.

## [5.95.1] — 2026-06-18

### Changed
- `docs/SOC2_READINESS.md` — §76.7/§76.8 CALIB-E cross-reference patch (v3.12.1): registers CALIB-E-001 and CALIB-E-002 (defined in §83.5, v3.9.0, 2026-06-15) in the §76 canonical CC7.2/CC4.1 evidence artefact table. Closes OBSERVABILITY.md §44.6 item 7 (P0/M5).
- `docs/OBSERVABILITY.md` — §44.6 item 7 marked [x] Done (2026-06-18).
- `VERSION` — 5.95.0 → 5.95.1.

## [5.95.0] — 2026-06-18

### Added
- `content/post-117-microcycle-load-placement.md` — Editorial series post 117. «Як ставити вагу у мікроцикл: не «коли вільно», а за логікою відновлення» — мікроцикл як функціональна одиниця (Bompa & Haff 2009); ЦНС-відновлення 48-72 год, MPS-вікно 24-48 год, сполучна тканина; High-Low принцип (Charlie Francis 1992); три помилки розміщення; алгоритм 5 кроків для 3-4 днів/тиждень. clinical-safety: PASS. 13-min read.

### Changed
- `blog.html` — blog card added for post-117 (inserted before post-116).
- `STATUS.md` — editorial series 11–34, 44–50, 106–117; Total posts 1133 → 1134; editorial 191–200 proposed.
- `README.md` — post-117 status updated to draft; editorial topics 191–200 added to roadmap.
- `VERSION` — 5.94.0 → 5.95.0.

## [5.94.0] — 2026-06-18

### Added
- `content/post-1141-kinetic-chain-force-transmission-intro.md` — Block 1101–1150 Cluster 5, post 1/10. Кінетичний ланцюг: вступ. Відкритий і закритий кінетичний ланцюг. Проксимо-дистальний принцип послідовності активації. Тулуб як активна кінетична ланка передачі між нижньою і верхньою кінцівкою. Ground reaction force і вектор зусилля. Слабка ланка, компенсація і системна адаптація. clinical-safety: NOT_REQUIRED.
- `content/post-1142-thoracolumbar-fascia-posterior-oblique-sling.md` — Block 1101–1150 Cluster 5, post 2/10. TLF і posterior oblique sling: анатомія TLF (posterior/middle/anterior layers). Posterior oblique sling: lat + TLF + contralateral glute max. Механізм при deadlift: lat натягує TLF, а не тягне гриф. EMG і cadaveric дані. «Lat в штані» як механічний ефект. Contralateral training і задіяння POS. clinical-safety: NOT_REQUIRED.
- `content/post-1143-anterior-oblique-sling-push-pattern.md` — Block 1101–1150 Cluster 5, post 3/10. Anterior oblique sling і push: pec major + contralateral external oblique + hip adductors. Leg drive в bench press через AOS. Ротаційні рухи і AOS (удар, кидок, swing). AOS vs POS реципрокний стосунок. Асиметрія AOS і rotation bias. Програмування однобічних push і ротаційних вправ. clinical-safety: NOT_REQUIRED.
- `content/post-1144-intra-abdominal-pressure-core-kinetic-link.md` — Block 1101–1150 Cluster 5, post 4/10. IAP і core як кінетична ланка. IAP при силовому навантаженні (5–8 → 200+ mmHg). Bracing vs. hollowing: Grenier & McGill 2007, Vera-Garcia 2007. Valsalva при максимальних підходах. Core як активний передавач зусилля між кінцівками. Чотирибічна IAP система: діафрагма, тазове дно, черевна стінка, TLF. Практичний протокол sub-max і max підходів. clinical-safety: NOT_REQUIRED.
- `content/post-1145-lumbopelvic-hip-complex-force-transmission.md` — Block 1101–1150 Cluster 5, post 5/10. LPHC і glute max. Lumbopelvic-hip complex: рух в одній ланці змінює дві інших. Glute max як передавач через hip extension і TLF. SIJ force closure (form closure vs. force closure, Snijders 1993). «Gluteal amnesia» і компенсаторні ланки. APT і PPT під навантаженням. Glute max activation перед важкими підходами. clinical-safety: NOT_REQUIRED.
- `content/post-1146-foot-ground-reaction-force-kinetic-chain.md` — Block 1101–1150 Cluster 5, post 6/10. Стопа і GRF. Tripod foot: три точки контакту і наслідки відхилення. GRF три компоненти (vertical/AP/ML). Pronation/supination: механіка і наслідки надлишку. Дорсифлексія як обмежувач кінетичного ланцюга. Стопа як proprioceptive система. Взуття і transfer GRF. clinical-safety: NOT_REQUIRED.
- `content/post-1147-muscle-stiffness-stretch-shortening-cycle.md` — Block 1101–1150 Cluster 5, post 7/10. М'язова жорсткість і SSC. Stiffness як механічна властивість (k = F/Δx). Сухожилля як пружний резервуар. SSC три механізми: пружна енергія, рефлекторна активація, моторна готовність. Fast vs. slow SSC. Rate of force development (RFD) і tendon stiffness (Bojsen-Møller 2005). Тренування жорсткості і SSC: важкий тренінг, plyometrics, isometric при довгих довжинах. clinical-safety: NOT_REQUIRED.
- `content/post-1148-bilateral-deficit-asymmetry-kinetic-chain.md` — Block 1101–1150 Cluster 5, post 8/10. Bilateral deficit і асиметрії. BLD визначення і типова величина (5–15% у нетренованих). Нейром'язові механізми: cortical inhibition, symmetry constraint. Symmetry constraint маскує асиметрії при двобічних рухах. Cross-education effect (Carroll 2006, ~7–8% transfer). Унілатеральне тестування: split squat, single-leg press, hop test. Програмування при значних асиметріях (>15%). clinical-safety: NOT_REQUIRED.
- `content/post-1149-programming-kinetic-chain-awareness.md` — Block 1101–1150 Cluster 5, post 9/10. Програмування з урахуванням кінетичного ланцюга. Warm-up як послідовна активація: стопа → glute → core → pattern → progressive loading. Pattern sequencing у сесії: hinge перед squat, pull перед push. Transfer training через виявлення слабкої ланки. Підрахунок об'єму через патерни замість «м'язів». Блокова модель через призму кінетичного ланцюга. Щоденний checklist перед важким підходом. clinical-safety: NOT_REQUIRED.
- `content/post-1150-kinetic-chain-cluster-synthesis-1141-1150.md` — Block 1101–1150 Cluster 5, post 10/10. **BLOCK COMPLETE.** Cluster synthesis: ключові положення постів 1141–1149. Три інтегруючих принципи: (1) зусилля передається через систему, (2) жорсткість кожної ланки визначає ефективність, (3) асиметрії компенсуються і маскуються двобічним навантаженням. П'ять myth-busters. Огляд усіх п'яти кластерів блоку 1101–1150. clinical-safety: NOT_REQUIRED.

### Changed
- `blog.html` — blog cards added for posts 1141–1150 (inserted before post-1140).
- `STATUS.md` — Block 1101–1150 COMPLETE 50/50; Cluster 5 COMPLETE; blog.html updated to post-1150; Total posts 1123 → 1133; Next: Block 1151–1200 тема TBD (product-strategist + sports-scientist).
- `VERSION` — 5.93.0 → 5.94.0.

---

## [5.93.0] — 2026-06-18

### Added
- `docs/SOC2_READINESS.md §87` — Evidence Artefact Cross-Reference Patch: COST_MODEL §39 (Deal Close Governance, Win/Loss Analytics & Competitive Intelligence). Registers three SOC 2 artefacts from COST_MODEL §39.9 (v2.5, 2026-06-15) into SOC2_READINESS: WIN-E-001 (CC5.2/CC1.4 — annual `enterprise.deal_closed_won` export with `floor_respected: true` attestation; 7yr; `compliance/evidence/winloss/WIN-E-001_<YYYY>.csv`), WIN-E-002 (CC5.2/CC4.1 — `enterprise.win_loss_analysis_recalibrated` chain export with PIPE-CHAIN-02 verification; 7yr; `compliance/evidence/winloss/WIN-E-002_deal<N>.json`), WIN-E-003 (CC1.4/CC9.2 — quarterly `privacy.no_go_criteria_applied` export; zero-count quarters filed as affirmative ethics attestation; 3yr; `compliance/evidence/winloss/WIN-E-003_<YYYY-QN>.csv`). Six SOC 2 criterion rows extended (CC5.2 × 2, CC1.4 × 2, CC4.1 × 1, CC9.2 × 1). Closes five cross-reference obligations from COST_MODEL §39.9 and §39.10 items 5 and 6 (both now [x] Done). Five-item implementation checklist: 1× P0/M9 (Admin Console modal prerequisite), 3× P1/M10 (WIN-E-001 first annual filing, WIN-E-003 quarterly cadence, §79.4 master evidence table), 1× P2/Deal 10 (WIN-E-002 at OQ-PIPE-01 threshold). SOC2_READINESS.md v3.11.1 → v3.12.0.

### Changed
- `docs/COST_MODEL.md §39.10` — checklist items 5 (WIN-E-001) and 6 (WIN-E-003) marked [x] Done (documentation); implementation gates tracked in SOC2_READINESS.md §87.7.
- `VERSION` — 5.92.0 → 5.93.0.

---

## [5.92.0] — 2026-06-18

### Added
- `content/post-116-training-feedback-partner-mirror-camera.md` — Editorial series post 116. «Тренувальний партнер, дзеркало, камера: три типи зворотного зв'язку і що кожен не бачить». proprioception як система з похибкою (Proske & Gandevia 2012); KR vs. KP (Schmidt & Lee 2011); concurrent vs. terminal feedback (Salmoni 1984); партнер: Zajonc 1965, bias, підтверджуюча хвала; дзеркало: Miall & Wolpert 1996, зниження proprioception accuracy, 2D один кут; камера: terminal KP, порівняння в часі, confirmation bias; інтегральна таблиця трьох інструментів; мінімальна система feedback для self-coached атлета. clinical-safety: PASS.

### Changed
- `blog.html` — blog card added for post-116 (inserted before post-115).
- `README.md` — post-115 status → draft · v5.90.0; post-116 status → draft · v5.92.0; editorial 181–190 proposed.
- `STATUS.md` — Editorial series updated to 11–34, 44–50, 106–116; post-116 entry added; Total posts: 1122 → 1123.
- `VERSION` — 5.91.0 → 5.92.0.

---

## [5.91.0] — 2026-06-18

### Added
- `content/post-1131-pull-pattern-horizontal-vertical-system.md` — Block 1101–1150 Cluster 4, post 1/10. Pull pattern як система: дві осі (горизонтальна і вертикальна), кінематичний ланцюг від стопи/тазу до хвату. Три рухи лопатки в тязі: ретракція, депресія, downward rotation у фінальній фазі. Роль SC/AC суглобів. Чому «тягни ліктями» — корисна, але неповна кю. Вхідна точка до кластеру. clinical-safety: NOT_REQUIRED.
- `content/post-1132-scapular-retraction-depression-trapezius-rhomboids.md` — Cluster 4, post 2/10. Ретракція vs депресія лопатки — різні м'язи, різні функції. Середня трапеція (горизонтальна ретракція), нижня трапеція (депресія + ретракція — ключова для обох осей тяги), rhomboids (ретракція + легка елевація). Serratus anterior як ексцентричний контролер у тязі. «Pack your scapulae» механічно визначено. Scapular SET перед тягою vs рух під час тяги. clinical-safety: NOT_REQUIRED.
- `content/post-1133-latissimus-dorsi-mechanics-shoulder-extension-adduction.md` — Cluster 4, post 3/10. Анатомія lat: origin (thoracolumbar fascia, iliac crest, нижні 6 ребер), insertion (intertubercular groove). Три дії: розгинання, приведення, внутрішня ротація плеча. Thoracolumbar fascia і чому lat є «м'язом стегна». Умови повного розтягнення широчайшого. Tight lat і overhead mobility. Пронований vs супінований хват і активація. clinical-safety: NOT_REQUIRED.
- `content/post-1134-vertical-pull-mechanics-pullup-lat-pulldown-grip-torso.md` — Cluster 4, post 4/10. Pull-up vs lat pulldown: одна механіка, різна стабілізаційна вимога. Траєкторія в pull-up — не вертикальна. Нахил тулуба в lat pulldown як нормальна механіка. Вплив ширини хвату: horizontal adduction vs extension vs ROM. Overhand/underhand/neutral grip механічні відмінності. Behind-the-neck pulldown ризики. Три чинники кінцевого діапазону. clinical-safety: NOT_REQUIRED.
- `content/post-1135-horizontal-pull-mechanics-row-variations-lumbar.md` — Cluster 4, post 5/10. Горизонтальна тяга як противага push-домінантним програмам. Bent-over row: hip hinge. Кут тулуба 45° vs 90° (Pendlay). Момент сили на хребті. Iniціація лопаткою vs ліктем. Seated cable row і chest-supported row — коли. Як далеко тягнути. Програмні рекомендації. clinical-safety: NOT_REQUIRED.
- `content/post-1136-elbow-flexors-biceps-brachialis-grip-pulling.md` — Cluster 4, post 6/10. Три згиначі ліктя: biceps (залежний від супінації moment arm — chin-up сильніший за pull-up), brachialis (grip-independent pure flexor), brachioradialis (neutral grip specialist). «Elbow dominance» пастка в lat pulldown. Fatigue sequencing. Практичне програмування ротації хватів. clinical-safety: NOT_REQUIRED.
- `content/post-1137-grip-width-angle-muscle-recruitment-pulldown-rows.md` — Cluster 4, post 7/10. EMG-аналіз ширини хвату у lat pulldown: wide pronated vs medium neutral vs close supinated. ROM і розтягнення lat при вузькому хваті. Overhand vs underhand у bent-over row. Neutral grip cable row як benchmark лопаткової механіки. Хват як інструмент рекрутменту, не стилістичний вибір. clinical-safety: NOT_REQUIRED.
- `content/post-1138-unilateral-pull-single-arm-row-anti-rotation.md` — Cluster 4, post 8/10. Чому однобічна тяга ≠ половина двобічної. Anti-rotation вимога тулуба. Three-point DB row, standing single-arm cable row, Meadows row, single-arm lat pulldown. Bilateral deficit у тязі (менш виражений ніж у push). Асиметрії як діагностика. Програмування як акцесорного або діагностичного інструменту. clinical-safety: NOT_REQUIRED.
- `content/post-1139-rear-deltoid-external-rotation-face-pull-horizontal-abduction.md` — Cluster 4, post 9/10. Задня дельта як prime mover у горизонтальному відведенні, систематично недотренована при push-домінантних програмах. Механіка face pull: ретракція + horizontal abduction + external rotation. Infraspinatus і teres minor як центрататори glenohumeral head. Чому «більше рядків» не замінює face pull повністю. Модель push:pull + external rotation. clinical-safety: NOT_REQUIRED.
- `content/post-1140-pull-pattern-cluster-synthesis-1131-1140.md` — Cluster 4, post 10/10. Синтез: дев'ять механічних висновків з постів 1131–1139, три інтегруючих принципи (кінетичний ланцюг у тязі, ретракція/депресія лопатки як основа, grip + кут + тулуб як важелі рекрутменту), чотири myth-busters. Наступний кластер: силова передача і кінетичний ланцюг (1141–1150). clinical-safety: NOT_REQUIRED.

### Changed
- `blog.html` — blog cards added for posts 1131–1140 (10 cards, inserted before post-1130).
- `STATUS.md` — Block 1101–1150 Cluster 4 COMPLETE 10/10; Cluster 5 next noted.
- `VERSION` — 5.90.0 → 5.91.0.

---

## [5.90.0] — 2026-06-18

### Added
- `content/post-115-technique-under-fatigue.md` — Editorial series post 115. «Техніка під втомою: що змінюється, чому це важливо і коли зупинятись». Диференційна втома стабілізаторів і прайм-муверів; компенсаторний рекрутмент у трьох ключових рухах (squat/press/hinge); чотири маркери technical breakdown; три сценарії зупинки; failure vs. non-failure (Bradford 2023, Refalo 2022); програмні рішення для стійкості техніки. clinical-safety: PASS.

### Changed
- `blog.html` — blog card added for post-115 (inserted before post-114).
- `STATUS.md` — Editorial series оновлено до 106–115; total posts 1121 → 1122.
- `README.md` — editorial roadmap розширено: теми 171–180 додано.
- `VERSION` — 5.89.0 → 5.90.0.

---

## [5.89.0] — 2026-06-18

### Added
- `content/post-1123-glenohumeral-joint-flexion-abduction-impingement.md` — Block 1101–1150 Cluster 3, post 3/10. Glenohumeral joint: ball-and-socket з найбільшим діапазоном і найменшою кістковою стабільністю. Subacromial простір: що проходить і що відбувається при внутрішній ротації вище 90°. Ротаторна манжета як центрователь головки. Кут відведення 75–80° при bench. clinical-safety: NOT_REQUIRED.
- `content/post-1124-bench-press-bar-path-touch-point-mechanics.md` — Cluster 3, post 4/10. Bench press: J-curve як фізичний наслідок моменту сили, не стилістичний вибір. Оптимальна точка торкання і антропометрія. Ширина хвату і кут відведення ліктя. Arch: пауерліфтинг vs. гіпертрофія — різна логіка. Leg drive і лопатки. clinical-safety: NOT_REQUIRED.
- `content/post-1125-shoulder-impingement-pressing-grip-elbow-mechanics.md` — Cluster 3, post 5/10. Shoulder impingement як механічний феномен: три параметри ризику (кут відведення, внутрішня ротація, upward rotation якість). Ширина хвату і elbows flared — контекст важливіший за правило. Neutral grip vs. straight bar vs. Swiss bar. clinical-safety: NOT_REQUIRED.
- `content/post-1127-horizontal-vs-incline-press-pec-fibers-anterior-deltoid.md` — Cluster 3, post 7/10. Pectoralis major: clavicular/sternal/costal волокна. Як кут нахилу лави змінює силові лінії. EMG-дані (Rodríguez-Ridao 2020): clavicular більше при incline, але anterior deltoid також суттєво зростає. Практична шкала 30°/45°/60°. clinical-safety: NOT_REQUIRED.
- `content/post-1128-unilateral-push-pattern-dumbbell-press-stabilization.md` — Cluster 3, post 8/10. Dumbbell press ≠ barbell з більшим ROM. Bilateral deficit і контрлатеральне інгібування. One-arm press: anti-rotation вимога. Landmine press. Serratus anterior при широкій амплітуді протракції. Floor press і ексцентрична амплітуда. clinical-safety: NOT_REQUIRED.
- `content/post-1129-triceps-pressing-elbow-extension-long-head-lockout.md` — Cluster 3, post 9/10. Трицепс як primary driver верхньої третини жиму. Три голівки: lateral, medial, long — різні умови роботи. Long head: двосуглобова механіка і залежність від позиції плеча. Sticking point в bench: де і чому. Close-grip bench нюанс. clinical-safety: NOT_REQUIRED.
- `content/post-1130-push-pattern-cluster-synthesis-1121-1130.md` — Cluster 3, post 10/10. Синтез: дев'ять механічних висновків з постів 1121–1129, три інтегруючих принципи (кінематичний ланцюг, mobility/stability sequencing, context-specific selection), чотири myth-busters. Наступний кластер: pull pattern (1131–1140). clinical-safety: NOT_REQUIRED.

### Changed
- `blog.html` — blog cards added for posts 1121–1130 (10 cards, inserted before post-1120).
- `STATUS.md` — Block 1101–1150 Cluster 3 COMPLETE 10/10.
- `VERSION` — 5.88.1 → 5.89.0.

---

## [5.88.1] — 2026-06-18

### Added
- `content/post-1121-push-pattern-horizontal-vertical-mechanics.md` — Block 1101–1150 Cluster 3, post 1/10. Push pattern як система: горизонтальна і вертикальна вісь, кінематичний ланцюг плечового поясу, три ступені свободи лопатки (протракція/ретракція, elevation/depression, upward/downward rotation), sternoclavicular joint, практична логіка системного погляду на push.
- `content/post-1122-scapulohumeral-rhythm-serratus-trapezius.md` — Cluster 3, post 2/10. Scapulohumeral rhythm: класичне співвідношення 2:1, serratus anterior (upward rotation через нижній кут лопатки), три частини трапецієподібного (верхня — elevation, нижня — depression + upward rotation, середня — ретракція), force couple serratus + lower trap, winging лопатки механічно, різниця ритму між горизонтальним і вертикальним жимом.
- `content/post-1126-overhead-press-thoracic-spine-rib-pelvis.md` — Cluster 3, post 6/10. Overhead press як тест всього тіла: thoracic extension як необхідна умова, rib flare механічно, нейтральний таз під навантаженням вгору, взаємозв'язок грудного відділу, поперека і тазу, push press і force transfer, практичний чек-лист перед OHP сесією.

### Changed
- `VERSION` — 5.88.0 → 5.88.1 (Cluster 3 posts 1121, 1122, 1126 — partial; 1123–1125, 1127–1130 in progress)

---

## [5.88.0] — 2026-06-18

### Added
- `content/post-114-frequency-vs-volume-self-coached.md` — Editorial series post 114. «Частота vs. обсяг: практичне рішення для self-coached атлета без нескінченного часу». Обсяг як первинна змінна, частота як механізм розподілу. Schoenfeld et al. 2016 мета-аналіз; Brigatto et al. 2019 equated volume; MPS вікна; стеля обсягу за сесію; три сценарії; алгоритм: планувати обсяг → виводити частоту. clinical-safety: NOT_REQUIRED. sports-scientist review required.

### Changed
- `blog.html` — blog cards added for post-113 (відсутня) та post-114 (нова); вставлені перед post-112.
- `STATUS.md` — Editorial series рядок оновлено: 106–113 → 106–114; post-114 і blog card пост-113 задокументовані.
- `VERSION` — 5.87.0 → 5.88.0.

---

## [5.87.0] — 2026-06-18

### Added
- `content/post-1111-squat-pattern-three-joints-mechanics.md` — Block 1101–1150 Cluster 2, post 1/10. Squat as a closed kinetic chain across three joints: ankle dorsiflexion, knee flexion, hip flexion + external rotation. Kinematic chain logic: why a problem in one joint presents as a symptom in another. Frontal and transverse plane control. Neutral spine in squat. Why squat is not the «opposite» of hip hinge.
- `content/post-1112-squat-depth-anatomy-atg-parallel.md` — post 2/10. Three different constraints on squat depth: ankle dorsiflexion, hip mobility, and bone morphology of the hip socket. Butt wink — three causes, three different solutions. Why ATG is a ceiling for some and simply unavailable for others. Activation research: depth and glute EMG.
- `content/post-1113-squat-torso-angle-trunk-inclination.md` — post 3/10. Why trunk forward lean in squat is physics, not a fault. How femur length and bar position determine «natural» trunk angle. High-bar vs. low-bar torso mechanics. When excessive forward lean is an actual problem vs. a morphology consequence.
- `content/post-1114-squat-load-distribution-quads-glutes.md` — post 4/10. How variant, depth and trunk angle shift load between quadriceps, glutes and hamstrings. Quadriceps peak in front squat and high-bar. Glutes peak with low-bar and depth. Hamstrings as stabilisers, not prime movers, in squat. Comparative table of variants.
- `content/post-1115-ankle-mobility-dorsiflexion-squat.md` — post 5/10. Dorsiflexion as one of the most common hidden constraints in squat. Gastrocnemius vs. soleus vs. capsular restriction — how to differentiate. Compensatory patterns: heel rise, excessive forward lean, valgus as three outputs of one limitation. Short-term (heel elevation) and long-term (stretching, mobilisation) solutions.
- `content/post-1116-knee-valgus-squat-structural-vs-motor.md` — post 6/10. Valgus in squat as a symptom, not a diagnosis. Four causes: gluteus medius weakness, limited hip external rotation, foot pronation, quadriceps failure under load. Why the same cue «push your knees out» does not fix different internal problems. Diagnostic algorithm.
- `content/post-1117-high-bar-low-bar-squat-mechanics-comparison.md` — post 7/10. How bar position changes moment arm and load distribution. High-bar: more knee-dominant, suits olympic lifting and shorter-femur athletes. Low-bar: greater posterior chain contribution, mechanical advantage for maximum load, standard in powerlifting. Mobility requirements differ. Neither is objectively «better».
- `content/post-1118-front-squat-goblet-squat-vertical-torso-mechanics.md` — post 8/10. Anterior load placement forces vertical torso through physics, not coaching cue. Maximum quadriceps stimulus of all barbell squat variants. Stricter dorsiflexion and thoracic extension demands. Goblet squat as a teaching tool and standalone hypertrophy exercise. Front squat and knee health: not a universal claim.
- `content/post-1119-unilateral-squat-bulgarian-lunge-step-up.md` — post 9/10. Why unilateral squat variants are a different motor task, not simplified bilateral squat. Frontal plane stabilisation (gluteus medius), anti-rotation control, asymmetry detection. Bulgarian split squat mechanics: torso angle shifts emphasis between quad and hip. Lunge variants. Step-up for load dosing in rehab. Why bilateral and unilateral cannot fully replace each other.
- `content/post-1120-squat-cluster-synthesis-1111-1120.md` — post 10/10. Cluster synthesis: five applied principles from squat pattern biomechanics. Kinematic chain as diagnostic key. «Correct technique» is a function of the body, not a standard. Depth and variant under goal. Mobility as investment. Bilateral and unilateral as different adaptations.

### Changed
- `blog.html` — 10 new post entries added (posts 1111–1120); updated to post-1120.
- `STATUS.md` — Block 1101–1150 row: Cluster 2 COMPLETE 10/10; total posts 1111 → 1121; next priorities updated to Cluster 3 push pattern.
- `VERSION` — 5.86.0 → 5.87.0.

---

## [5.86.0] — 2026-06-17

### Added
- `docs/AUDIT_LOG_SCHEMA.md §SCIM IP KV Cache Invalidation Error events` — v2.16. New advisory STANDARD/3yr event `scim.ip_kv_invalidation_error`: emitted by Admin Dashboard API when `SSO_KV.delete('scim_ip_cfg:{tenant_id}')` throws after a successful `update_scim_ip_enforcement_flag()` RPC write; non-blocking (HTTP 200 returned); payload: `tenant_id`, `kv_error_class`, `new_enforcement_value` (bool), `fallback_propagation_ttl_s: 300`. No PagerDuty — compliance-officer weekly SIEM review. Retention table +1 row (3yr, CC7.2). Closes SSO_SCIM_IMPLEMENTATION.md §33.7 item 3 (P0/M8).

### Changed
- `docs/SSO_SCIM_IMPLEMENTATION.md` — §33.7 item 3 marked [x] Done; cross-reference to AUDIT_LOG_SCHEMA.md v2.16 added.
- `VERSION` — 5.85.0 → 5.86.0.

---

## [5.85.0] — 2026-06-17

### Added
- `content/post-1101-hip-hinge-pelvis-spine-mechanics.md` — Block 1101–1150 Biomechanics, Cluster 1 post 1/10. Hip hinge fundamentals: lumbopelvic rhythm, pelvis-spine relationship, moment arm mechanics, neutral lumbar range, three assessment tests. Opens Block 1101–1150 Біомеханіка рухових патернів і силова передача.
- `content/post-1102-hamstring-length-vs-hip-mobility.md` — Cluster 1 post 2/10. Two distinct hip hinge limitations: passive SLR vs. knee-to-chest protocol to distinguish hamstring length from hip joint restriction. Motor control as third scenario. Practical 4-step assessment.
- `content/post-1103-deadlift-bar-path-analysis.md` — Cluster 1 post 3/10. Deadlift bar path: vertical projection as the goal, contact with shin as geometric necessity (not style), knee bypass mechanics, conventional vs. sumo bar path difference, three-line video analysis protocol.
- `content/post-1104-posterior-chain-glutes-hamstrings-hip-angle.md` — Cluster 1 post 4/10. Posterior chain load distribution by hip angle: gluteus maximus as upper-phase driver, hamstrings dual role (hip extensor + knee stabilizer), why "tuck under" in lockout = incomplete hip extension. Identifying weak link by failure phase.
- `content/post-1105-conventional-vs-sumo-deadlift-anatomy.md` — Cluster 1 post 5/10. Conventional vs. sumo as anatomical decision: femur length, acetabular morphology, femoral anteversion. Load distribution table. Practical selection algorithm. Debunking "sumo is cheating."
- `content/post-1106-rdl-mechanics-eccentric-hamstring.md` — Cluster 1 post 6/10. RDL eccentric phase mechanics: fascicle length adaptation, 3–4s eccentric tempo rationale, technical criteria for bottom position, RDL vs. leg curl comparison, programming guidelines (40–60% 1RM deadlift, 3–4×6–10).
- `content/post-1107-sldl-vs-rdl-tissue-loading.md` — Cluster 1 post 7/10. SLDL vs. RDL tissue loading comparison: knee angle determines hamstring tension; SLDL increases erector demand via lift-from-floor; RDL provides constant eccentric tension. Programming placement and prerequisites for SLDL.
- `content/post-1108-single-leg-hip-hinge-stability-mechanics.md` — Cluster 1 post 8/10. Single-leg hip hinge as a distinct skill: Trendelenburg and gluteus medius, anti-rotation demand, three-step learning progression (assisted → contralateral → ipsilateral), common errors (pelvic rotation, valgus), rationale for unilateral work in bilateral sport.
- `content/post-1109-hip-hinge-olympic-lifting-pull-mechanics.md` — Cluster 1 post 9/10. Hip hinge in Olympic lifting: first pull as controlled deadlift, transition ("lag") position, second pull as explosive hip extension, triple extension sequencing, rate of force development (RFD), derivatives (power clean, KB swing) for strength athletes.
- `content/post-1110-hip-hinge-cluster-synthesis.md` — Cluster 1 post 10/10. Cluster synthesis 1101–1110: five applied principles from the cluster (pattern over exercise, angle determines load, anatomy as factor not excuse, undervalued eccentric, stability ≠ strength), gaps identified, preview of Cluster 2 (squat pattern).

### Changed
- `blog.html` — 10 new post cards added (posts 1101–1110, category Biomechanics/Synthesis), inserted before post-1100.
- `STATUS.md` — Block 1101–1150 Cluster 1 COMPLETE (10/50); next priorities updated; total posts 1101 → 1111; version header 5.84.0 → 5.85.0.
- `VERSION` — 5.84.1 → 5.85.0.

---

## [5.84.1] — 2026-06-17

### Changed
- `docs/AUDIT_LOG_SCHEMA.md` v2.15 — `### Mobile Session Replay events` section added: registers `mobile.replay_config_updated` (STANDARD, 3yr) and `mobile.replay_tier_violation` (HIGH, 7yr) DEC-030 HMAC-chained events per OBSERVABILITY §46.6 (DEC-063). Full Zod schemas, REPLAY-CHAIN-01 ordering invariant, SOC 2 evidence table (REPLAY-E-001 CC6.7; REPLAY-E-003 CC7.2/P1.1). Retention table +2 rows. Closes OBSERVABILITY §46.11 item 3 (P0/M5) and SOC2_READINESS §86.7 item 3 (P0/M5) — documentation portion complete; Worker registry + integration test remain implementation tasks.
- `docs/OBSERVABILITY.md` — §46.11 item 3 marked [x] Done (docs); implementation remainder noted.
- `docs/SOC2_READINESS.md` — §86.7 item 3 marked [x] Done; cross-reference to AUDIT_LOG_SCHEMA v2.15.
- `VERSION` — 5.84.0 → 5.84.1.

---

## [5.84.0] — 2026-06-17

### Added
- `content/post-113-reading-training-variability.md` — Editorial series post-113. «Як атлет читає власну варіабельність: коли «не той день» — норма, а коли — сигнал». Шум vs. системний сигнал; RPE-тренд rolling 2–3 тижні; HRV rolling average (Nordin 2019); горизонт 48–72 год; алгоритм 3 кроки; правило трьох поганих сесій. clinical-safety: PASS. sports-scientist review required.
- README.md — editorial roadmap розширено: теми 151–165 proposed (content-engine routine · 2026-06-17).

### Changed
- `STATUS.md` — editorial series оновлено до 106–113; v5.84.0.
- `VERSION` — 5.83.0 → 5.84.0.

---

## [5.83.0] — 2026-06-17

### Added
- `content/post-1097-epl-third-compartment-listers-tubercle.md` — Block 1051–1100 М'язова специфіка, cluster 10 post 2/5. Extensor pollicis longus (EPL): початок — пост. ulna + IOM (середня третина), прикріплення — тильна основа дистальної фаланги I. Іннервація PIN (C7–C8). Третій дорзальний компартмент: розташування медіально від горбика Lister, горбик Lister як анатомічний шків ~45°. Функція: єдиний розгинач IP суглоба великого пальця; ретропульсія. Tabletop test (скринінг розриву). Відстрочений атриційний розрив EPL після перелому дистального radius: атриція об нерівну поверхню кісткового мозолю через 4–12 тижнів, незалежно від ступеня складності перелому. Розрив при РА. EIP-to-EPL трансфер як золотий стандарт лікування. PIN palsy і диференціація від ECRL/ECRB-збереженої клінічної картини. Практика: моніторинг IP-розгинання 12 тижнів після перелому, hook grip не знижує ризик. clinical-safety: NOT_REQUIRED. sports-scientist review pending.
- `content/post-1098-fpl-anterior-interosseous-nerve-thumb-flexion.md` — Block 1051–1100 М'язова специфіка, cluster 10 post 3/5. Flexor pollicis longus (FPL): початок — передня поверхня radius + IOM (середня–нижня третина), прикріплення — долонна основа дистальної фаланги I. Іннервація: anterior interosseous nerve (AIN) — гілка серединного нерва C8–T1. AIN іннервує тріаду: FPL + FDP-2/3 + pronator quadratus. AIN syndrome: ізольована нейропатія з FPL + FDP-2/3 слабкістю без сенсорного дефіциту. OK sign (pinch sign failure) — скринінговий тест. Parsonage-Turner brachial neuritis — найпоширеніша причина у спортсменів. Mannerfelt lesion — атриційний розрив FPL при РА на рівні карпального тунелю. Accessory head FPL (Gantzer's muscle) у 45–67% — фактор AIN ентраппінгу. FPL у тренуванні: IP замикання при стандартному хваті, ізометричний hook grip, tip pinch. Pulley система великого пальця: oblique pulley як функціональний еквівалент A2 у пальцях. clinical-safety: NOT_REQUIRED. sports-scientist review pending.
- `content/post-1099-tfcc-ulnar-wrist-load-transfer.md` — Block 1051–1100 М'язова специфіка, cluster 10 post 4/5. TFCC (triangular fibrocartilage complex): компоненти — articular disc, meniscus homolog, dorsal/palmar radioulnar ліґаменти (DRUL/PRUL), ulnocarpal ліґаменти (UL, UT). Функція: передача ~20% осьового навантаження кисті (positive ulnar variance підвищує), стабілізація DRUJ (DRUJ ballottement тест), карпальна стабілізація. Forearm rotation і dynamic ulnar variance. Palmer класифікація: Type 1A (центральний розрив — найчастіший у спортсменів), 1B (foveal tear — DRUJ нестабільність), 1C, 1D; Type 2 (дегенеративний). Клінічні тести: fovea sign, TFCC compression test, DRUJ ballottement, press test. Диференціальна діагностика: ECU tendinopathy, LT ligament tear, ulnar impaction, hamate fracture. МРТ 3T з артрографічним контрастом — gold standard; артроскопія — референтний стандарт. Лікування: іммобілізація (1A часткові), arthroscopic debridement, peripheral repair, foveal reattachment, ulnar shortening osteotomy. Ризики у силовиків: clean/snatch, overhead press, heavy rows. Positive ulnar variance як базовий фактор ризику дегенеративних змін. clinical-safety: NOT_REQUIRED. sports-scientist review pending.
- `content/post-1100-block-synthesis-1051-1100.md` — Block 1051–1100 synthesis (post 50/50). Підсумок 10 кластерів: лопатковий контроль, ротаційна манжета, механіка ліктя, флексори/пронатори передпліччя, lumbricales і interossei, hypothenar і ульнарний полюс, дорзальні розгиначі, sagittal band і retinacular system, довгі м'язи великого пальця (I і III компартменти), FPL/AIN і TFCC. Три головні висновки: іннервація як клінічна карта, анатомічні варіанти як популяційна реальність, функціональна анатомія як простір адаптацій. Практичні наслідки: hook grip vs. стандартний — різні навантаження на компартменти; моніторинг EPL після перелому radius; TFCC сигнали при clean/snatch. Вперед: Block 1101–1150 — Біомеханіка рухових патернів і силова передача (tentative). clinical-safety: PASS. sports-scientist review pending.

### Changed
- `blog.html` — posts 1097–1100 додані у верх списку.
- `STATUS.md` — Block 1051–1100 COMPLETE 50/50; next priorities updated до Block 1101–1150.
- `VERSION` — 5.82.1 → 5.83.0.

---

## [5.82.1] — 2026-06-17

### Changed
- `docs/SSO_SCIM_IMPLEMENTATION.md` — bumped to v2.5; appended §33 (OQ-SSO-32.1 & OQ-SSO-32.2 Resolution). OQ-SSO-32.1: documents CSM onboarding protocol for `scim_ip_enforcement_enabled` toggle, restricted to self-hosted SCIM proxy customers; cloud-hosted IdP tenants explicitly excluded due to dynamic IP range incompatibility. OQ-SSO-32.2: adopts active push KV cache invalidation (`SSO_KV.delete('scim_ip_cfg:{tenantId}')`) on every toggle, mirroring §25.4 pattern; includes TypeScript handler (`handleScimIpEnforcementToggle`), Postgres RPC (`update_scim_ip_enforcement_flag` — migration `0077_scim_ip_enforcement_toggle_rpc.sql`), advisory-only audit event on KV error (`scim.ip_kv_invalidation_error`, STANDARD severity, non-blocking), and SOC 2 evidence mapping (CC6.1, CC6.3).
- `VERSION` — 5.82.0 → 5.82.1.

---

## [5.82.0] — 2026-06-17

### Added
- `content/post-1096-apl-epb-first-compartment-dequervain.md` — Block 1051–1100 М'язова специфіка, cluster 10 (тема: великий палець і зап'ясткова стабільність), post 1/5. Abductor pollicis longus (APL) і extensor pollicis brevis (EPB) — перший дорзальний компартмент зап'ястка. APL: початок — пост. radius + ulna + IOM, прикріплення — основа 1-ї п'ясткової, функція — радіальна абдукція CMC-1 + розгинання CMC-1 + радіальна девіація зап'ястка. EPB: початок — пост. radius + IOM, прикріплення — проксимальна фаланга I, функція — розгинання MCP великого пальця. Обидва: іннервація PIN (C7–C8). Septa у першому компартменті у 34–67% — критично для хірургічного лікування. De Quervain's tenosynovitis: механізм стенозуючого тендовагініту, провокуючі рухи у силовому тренуванні (стандартний хват + ульнарна девіація). Finkelstein test (оригінальний 1930) vs. Eichhoff modification: різна специфічність. Intersection syndrome — диференціація. Hook grip як практична зміна хвату для зниження навантаження на 1-й компартмент. Консервативне (thumb spica, ін'єкція) і хірургічне лікування. clinical-safety: NOT_REQUIRED. sports-scientist review pending.

### Changed
- `blog.html` — додана картка post-1096 вгорі блоку.
- `STATUS.md` — Block 1051–1100 оновлено: 45→46/50; cluster 10 IN PROGRESS; поточна версія v5.82.0.
- `VERSION` — 5.81.0 → 5.82.0.

---

## [5.81.0] — 2026-06-17

### Added
- `content/post-1092-extensor-digiti-minimi-anatomy.md` — Block 1051–1100 М'язова специфіка, cluster 9, post 2/5. Extensor digiti minimi (EDM): єдиний власний розгинач мізинця у поверхневому шарі posterior compartment, спільний common extensor origin з EDC, два сухожилки у більшості людей, іннервація PIN (C7–C8). Функція: ізольоване розгинання 5-го пальця при зігнутих 2–4 — механізм обходу juncturae tendinum аналогічний до EIP і вказівного. Варіабельність: відсутність EDC до мізинця у частини людей → EDM є єдиним розгиначем. PIN palsy дає повну втрату функції EDM разом із EDC. Тест ізольованої функції мізинця. clinical-safety: NOT_REQUIRED. sports-scientist review pending.
- `content/post-1093-extensor-digitorum-communis-architecture.md` — Block 1051–1100, cluster 9, post 3/5. Extensor digitorum communis (EDC): найбільший м'яз поверхневого шару posterior compartment, чотири сухожилки до пальців 2–5 з'єднані juncturae tendinum. Архітектура extensor expansion: sagittal bands, central slip (PIP), lateral bands → terminal tendon (DIP). Функційні межі EDC без intrinsic м'язів (intrinsic minus hand, claw). Клінічна роль juncturae tendinum у маскуванні розривів. Зони ушкодження розгинача I–VIII з клінічною прив'язкою. Lateral epicondylitis: Maudsley test специфічний для EDC-компоненту. clinical-safety: NOT_REQUIRED. sports-scientist review pending.
- `content/post-1094-sagittal-band-anatomy.md` — Block 1051–1100, cluster 9, post 4/5. Sagittal band: парна фіброзна структура над MCP, що утримує EDC по центру при всіх положеннях пальця. Класифікація Rayan-Murray (I–III). Boxer's knuckle: механізм, клінічна картина, тест нестабільності EDC при флексії. Ульнарна сльоза 3-го пальця як найпоширеніший варіант. Ревматоїдний артрит: ерозія sagittal band → прогресуюча ульнарна девіація. Консервативне (4–6 тижнів іммобілізації) і хірургічне лікування. Протокол повернення атлета до навантажень. clinical-safety: NOT_REQUIRED. sports-scientist review pending.
- `content/post-1095-retinacular-system-digits-anatomy.md` — Block 1051–1100, cluster 9, post 5/5. Retinacular system of digits: ORL (зв'язок Landsmeer), TRL і lateral bands. PIP–DIP coupling: розгинання PIP → натяг ORL → пасивне розгинання DIP. Boutonnière деформація: розрив central slip → lateral bands вентральніше від осі PIP → PIP згинання + DIP гіперекстензія. Swan-neck деформація: ослаблення volar plate → lateral bands дорзально → PIP гіперекстензія + DIP згинання. Bunnell test: intrinsic tightness vs joint capsule contracture. Mallet finger → нелікований → swan-neck cascade. clinical-safety: NOT_REQUIRED. sports-scientist review pending.

### Changed
- `blog.html` — додані картки post-1092, post-1093, post-1094, post-1095 вгорі блоку.
- `STATUS.md` — Block 1051–1100 cluster 9 оновлено: 41→45/50; наступний: posts 1096–1100 (cluster 10, тема TBD).
- `VERSION` — 5.80.2 → 5.81.0.

---

## [5.80.2] — 2026-06-17

### Changed
- `docs/SOC2_READINESS.md §86` — Evidence Artefact Cross-Reference Patch: OBSERVABILITY §46 (DEC-063). Registers three SOC 2 evidence artefacts defined in OBSERVABILITY §46.9 (v4.3.0, DEC-063) that were flagged for addition here in §46.11 item 9 (P1/M8). REPLAY-E-001 (CC6.7 — on-demand enterprise debug session authorisation record: Sentry session list + `mobile.replay_config_updated` DEC-030 chain + zero-violation confirmation; 3yr; per debug window). REPLAY-E-002 (CC6.7 — `replay-allowlist.test.ts` CI pass log: asserts prohibited route-name patterns, `maskAllText`/`maskAllInputs` invariants, consumer-tier zero sample rates; 3yr; per `sentry-replay-config.ts` change; clinical-safety sign-off required for allowlist changes). REPLAY-E-003 (CC7.2 + P1.1 — quarterly `mobile.replay_tier_violation` DEC-030 export; zero-event quarters filed as affirmative attestation; route_name SHA-256 hashed; 7yr). §86.5 extends CC6.7 × 2, CC7.2 × 1, P1.1 × 1 criterion rows. §86.6 closes four cross-reference obligations from OBSERVABILITY §46.9/§46.11. §86.7 seven-item checklist: 2× P0/M5, 5× P1/M5–M8. SOC2_READINESS.md v3.11.0 → v3.11.1.
- `docs/OBSERVABILITY.md §46.11` — Item 9 marked [x] Done (2026-06-17): SOC2_READINESS.md §86 closes the obligation.
- `VERSION` — 5.80.1 → 5.80.2.

---

## [5.80.1] — 2026-06-17

### Changed
- `docs/OBSERVABILITY.md §46` — OQ-MOBILE-02 Resolution: Sentry Session Replay Screen Allowlist & Clinical-Safety Review Protocol (DEC-063). Closes OQ-MOBILE-02 (P1 — before M4 deploy, from §28.13). Tier S screen allowlist adopted (14 routes: SsoLogin, SsoCallback, OnboardingStep1–4, Subscription, BillingConfirmation, AdminDashboard, AdminDashboardTeamActivity, AppError, NetworkError, Settings, SsoSettings). Session replay remains disabled by default; enterprise 5% error-session sample on Tier S only; on-demand 10% CSM-initiated protocol (§46.5). `maskAllText: true` + `maskAllInputs: true` mandatory. Four-layer privacy model: Tier S architectural exclusion → `isReplayPermitted()` runtime gate in `beforeSend` → text masking → §28.3 breadcrumb deny-list. Two new DEC-030 events: `mobile.replay_config_updated` (STANDARD, 3yr) + `mobile.replay_tier_violation` (HIGH, 7yr; expected zero in steady state). REPLAY-CHAIN-01 ordering invariant. Three SOC 2 evidence artefacts: REPLAY-E-001 (CC6.7, on-demand), REPLAY-E-002 (CC6.7, per allowlist change), REPLAY-E-003 (CC7.2/P1.1, quarterly; zero-event attestation). Clinical-safety gate: sign-off + compliance-officer approval + new DEC entry required for any allowlist change; CI enforcement via `replay-allowlist.test.ts`. Two open questions: OQ-REPLAY-01 (P2 — error sample rate 5% → 20% after 90 days M5 data), OQ-REPLAY-02 (P2 — Admin Dashboard conditional classification on k-anonymity floor CI enforcement). §28.13 OQ-MOBILE-02 row updated to 🟢 Resolved DEC-063. OBSERVABILITY.md header v4.2.1 → v4.3.0.
- `docs/DECISION_LOG.md` — DEC-063 (OQ-MOBILE-02 — Sentry session replay Tier S allowlist, 2026-06-17) and DEC-062 (OQ-SSO-23.2/25.1/25.3 — magic-link CAEP session coverage, SCIM IP flag, API key IP enforcement, 2026-06-16) added. DEC-062 was referenced in SSO_SCIM §32 (v5.79.1) but not yet registered in the DECISION_LOG; both entries added in this commit.
- `VERSION` — 5.80.0 → 5.80.1.

---

## [5.80.0] — 2026-06-17

### Added
- `content/post-1091-extensor-indicis-proprius-anatomy.md` — Block 1051–1100 cluster 9 post 1/5. Extensor indicis proprius (EIP): глибокий posterior compartment forearm, походить від дистальної третини ulna і posterior interosseous membrane, прикріплюється до extensor expansion вказівного (ульнарніше EDC). Іннервація: posterior interosseous nerve (PIN, C7–C8). Єдина функція — незалежне розгинання вказівного пальця (pointing sign). EIP → EPL tendon transfer як найпоширеніша реконструкція при ревматоїдному або посттравматичному розриві EPL. PIN palsy: слабкість розгиначів пальців і великого пальця при збереженні зап'ястного розгинання і відсутньому сенсорному дефіциті. clinical-safety: NOT_REQUIRED. sports-scientist review required before publish.
- `blog.html` — post-1091 card added at top of feed.

### Changed
- `STATUS.md` — Block 1051–1100 updated: 40/50 → 41/50; cluster 9 IN PROGRESS; blog.html updated to post-1091; Total posts 1096 → 1097; version v5.80.0.
- `VERSION` — 5.79.1 → 5.80.0.

---

## [5.79.1] — 2026-06-16

### Changed
- `docs/SOC2_READINESS.md §85` — Evidence Artefact Cross-Reference Patch: SSO_SCIM §32 (DEC-062). Registers CC6-E-ML-001 (CC6.3 — magic-link CAEP revocation integration test, M4 deploy gate, 7yr), CC6-E-SCIM-IP-001 (CC6.1 — migration 0076 `scim_ip_enforcement_enabled DEFAULT FALSE` confirmation, M5, 7yr), CC6-E-APIKEY-IP-001 (CC6.1/CC6.3 — monthly `sso.ip_allowlist_blocked` export filtered to `auth_path = 'api_key'`, from M5, 3yr). Closes three cross-reference obligations from `docs/SSO_SCIM_IMPLEMENTATION.md §32.6` (v2.4, DEC-062). SOC2_READINESS.md header v3.10.0 → v3.11.0.

---

## [5.79.0] — 2026-06-16

### Added
- `content/post-16-overtraining-underrecovery.md` — Editorial series post 16. «Перетренованість або недовідновлення: як self-coached атлет читає сигнали тіла». Спектр accumulated fatigue → functional overreaching → non-functional overreaching → OTS (Meeusen et al. 2013 ECSS/ACSM consensus; Halson & Jeukendrup 2004); time-to-recovery як діагностичний ключ; диференційний чекліст 4 рівні; HRV rolling average; RPE-дрейф. clinical-safety CONDITIONAL PASS (guard rail: OTS clinical symptoms redirected to physician). sports-scientist review required before publish. Blog card exists at blog.html.

### Changed
- `README.md` — editorial roadmap extended: proposed topics 141–150 added.
- `STATUS.md` — post-16 added to editorial series note; Total posts 1095 → 1096; current version v5.79.0.
- `VERSION` — 5.77.0 → 5.79.0 (5.78.0 was logged in CHANGELOG but VERSION was not updated in prior commit; this catch-up bump resolves the gap).

---

## [5.78.0] — 2026-06-16

### Added
- `content/post-1086-dorsal-interossei-anatomy.md` — Block 1051–1100 cluster 8 post 1/5. Dorsal interossei: four bipennate intrinsic hand muscles (DAB = Dorsal ABduct). Bipennate origin from adjacent metacarpals, deep branch of ulnar nerve innervation. 1st DI as largest and pinch stabiliser; clinical marker for ulnar neuropathy atrophy. Fat grip training and DI loading in compound pulls.
- `content/post-1087-palmar-interossei-anatomy.md` — Block 1051–1100 cluster 8 post 2/5. Palmar interossei: three unipennate intrinsic muscles (PAD = Palmar ADduct). Co-activation with DI during cylinder grip — not antagonists but a stiffness system. Card test for adduction weakness; ulnar neuropathy clinical picture. Pinch and climbing as specific stressors.
- `content/post-1088-thenar-complex-anatomy.md` — Block 1051–1100 cluster 8 post 3/5. Thenar eminence complex: APB (palmar abduction), opponens pollicis (opposition/metacarpal rotation), FPB (MCP flexion, dual innervation median + ulnar). Opposition mechanics: 3D movement of 1st metacarpal. APB atrophy as classic CTS marker; recurrent branch of median nerve anatomy. Pinch grip and climbing as primary stressors.
- `content/post-1089-hypothenar-complex-anatomy.md` — Block 1051–1100 cluster 8 post 4/5. Hypothenar eminence: ADM, FDMB, ODM — all deep branch of ulnar nerve. ODM and opponens pollicis as the "cupping" system of the palm. Guyon's canal (ulnar tunnel syndrome) with zone 1/2/3 differentiation; cyclist's palsy; hook of hamate fracture risk in grip training. Cubital tunnel vs. Guyon's canal differential.
- `content/post-1090-adductor-pollicis-anatomy.md` — Block 1051–1100 cluster 8 post 5/5. Adductor pollicis: oblique + transverse heads, deep branch of ulnar nerve, largest intrinsic thumb muscle. Primary force generator for lateral (key) pinch. Froment's sign: IP flexion compensation via FPL when AP is weak — clinical test for ulnar neuropathy. Jeanne's sign. 1st web space atrophy as visible marker. Plate pinch, blob lifting, climbing pinch holds as AP-specific training.
- `blog.html` — posts 1086–1090 added at top of feed.

### Changed
- `STATUS.md` — Block 1051–1100 progress updated to 40/50; next: posts 1091–1095 (extensor indicis proprius, extensor digiti minimi, extensor digitorum communis architecture, sagittal band, retinacular system of digits).

---

## [5.77.0] — 2026-06-16

### Added
- `docs/SSO_SCIM_IMPLEMENTATION.md §32` — OQ-SSO-23.2 · OQ-SSO-25.1 · OQ-SSO-25.3 Resolution (DEC-062). Three P1 blockers resolved together: (1) §32.2 — `isRevoked()` call-graph confirmed to cover magic-link sessions at middleware layer; integration test `CC6-E-ML-001` added as M4 deploy closure gate. (2) §32.3 — SCIM IP allowlist scope: option (c) adopted (`scim_ip_enforcement_enabled BOOLEAN NOT NULL DEFAULT FALSE`, migration `0076`); Admin Dashboard toggle with Okta/Azure AD warning banner; advisory audit event `scim.ip_enforcement_misconfigured`. (3) §32.4 — API key IP enforcement: `enforceIpAllowlist()` extended with `authPath: 'sso' | 'api_key'`; wired into `api-key-auth.ts`; `sso.ip_blocked` DEC-030 payload extended with `auth_path` field. Document header v2.3 → v2.4.
- `docs/AUDIT_LOG_SCHEMA.md §SCIM` — new `scim.ip_enforcement_misconfigured` event (STANDARD, 3yr, advisory). Emitted by `enforceScimIpAllowlist()` when enforcement flag is enabled but allowlist is empty; non-blocking; no PagerDuty page; compliance-officer SIEM review. Closes SSO §32.8 checklist item 3 (P0, before SCIM go-live).

### Changed
- `docs/AUDIT_LOG_SCHEMA.md §SSO authentication policy events` — `sso.ip_blocked` payload extended with `auth_path` (`'sso' | 'api_key'`, optional, default `'sso'`); backwards-compatible. Closes SSO §32.8 checklist item 1 (P0/M4).
- `docs/AUDIT_LOG_SCHEMA.md` — header updated v2.13 → v2.14.
- `docs/SSO_SCIM_IMPLEMENTATION.md §23.9` — OQ-SSO-23.2 row updated 🟡 P1 → 🟢 Resolved (DEC-062; §32.2).
- `docs/SSO_SCIM_IMPLEMENTATION.md §25.13` — OQ-SSO-25.1 and OQ-SSO-25.3 rows updated 🟡 P1 → 🟢 Resolved (DEC-062; §32.3, §32.4).
- `VERSION` — 5.76.0 → 5.77.0.

---

## [5.76.0] — 2026-06-16

### Added
- `content/post-112-training-and-life-stress.md` — Editorial series post 112. «Тренування і стрес поза залом: як системне навантаження складається і чому програма це не враховує». HPA-вісь і алостатичне навантаження (McEwen); Stults-Kolehmainen & Sinha 2014 (психологічний стрес → знижені силові адаптації); Bartholomew 2008 (гострий стрес → підвищений RPE); Martin 2009 (ментальна втома → зниження витривалості); Leproult & Van Cauter 1999 (сон → кортизол); практичний алгоритм check-in + таблиця рішень; HRV як інтегральний маркер. clinical-safety: CONDITIONAL PASS. sports-scientist review pending.

### Changed
- `blog.html` 